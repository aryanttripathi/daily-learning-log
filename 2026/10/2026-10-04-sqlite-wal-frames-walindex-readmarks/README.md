<!--
entry-meta
date: 2026-10-04
type: lesson
track: SQLite
lesson: 19
category: Database Internals
title: The WAL and Its Index — Frame Append, a Hash That Is a Permutation, and Five Read Marks
slug: sqlite-wal-frames-walindex-readmarks
-->

# The WAL and Its Index — Frame Append, a Hash That Is a Permutation, and Five Read Marks

**2026-10-04 · SQLite Track · Lesson 19 of 33**

## Where This Fits

- **Previous lesson:** [Lesson 18](../2026-10-03-sqlite-vfs-locking-styles-inode-emulation/README.md) opened the two dispatch tables and established the `xShmMap`/`xShmLock`/`xShmBarrier`/`xShmUnmap` slots, which VFSes leave them null, and the exact line ordering inside `sqlite3PagerWalSupported()`. It deliberately did **not** touch the `-shm` file's contents. This lesson is that content.
- **Lessons 15–17** established the five lock levels and their byte offsets, the rollback-journal commit sequence, and the `aMJNeeded[]` matrix that silently disables cross-file atomicity in WAL mode. **This lesson does not re-teach those.** It takes as given that in WAL mode the database file never rises above `SHARED` and that write exclusion has moved into the `-shm` file.
- **What this lesson adds:** the 32-byte WAL header and 24-byte frame header with the running checksum chain that links every frame to the one before it; `walFrames()` end to end, including the in-place frame overwrite and the `iReCksum`/`walRewriteChecksums()` repair that follows it; the wal-index's 32 KiB page layout, `HASHTABLE_NPAGE_ONE`, and the two header copies; why `walHash()` is a *permutation* of the 8192 slots rather than a hash in any useful sense, and what therefore actually collides; `walIndexAppend()` and `walCleanupHash()`; why `walFindFrame()` keeps scanning after it finds a match; the five `aReadMark[]` slots and the `WAL_READ_LOCK(i)` bytes that protect them; `walTryBeginRead()`'s mark selection, the `aReadMark[0]` shortcut and its two re-validation `memcmp`s; the one `memcmp` in `sqlite3WalBeginWriteTransaction()` that *is* `SQLITE_BUSY_SNAPSHOT`; and `walRestartHdr()`, which rewinds the WAL and increments a checkpoint counter that turns out not to be global.
- **Next:** [Lesson 20](../../../TRACK.md) takes checkpointing — `walCheckpoint()`, the `WalIterator` and its two algorithms, `wal_autocheckpoint`, and the `FULL`/`RESTART`/`TRUNCATE` modes. This lesson supplies `nBackfill`, `nBackfillAttempted` and the read-mark floor that bounds how far a checkpoint may go; it measures that floor from the outside but does not open `walCheckpoint()`.

Source read by file and line through a code index during this run, at commit `9696acb0` and re-read at `ccbdec84` (identical line numbering for every region cited). Measurements are against the **system `libsqlite3` 3.45.1** on x86-64 Linux 6.18, ext4, `page_size=1024` unless stated. Where source and measurement disagree, §13 says so.

---

## 1. Two Files, and Only One of Them Is Durable

WAL mode adds two files beside the database. They are not two halves of one thing; they have opposite durability contracts.

| | `-wal` | `-shm` |
|---|---|---|
| Holds | page images, as frames | an index over those frames, plus lock bytes |
| Written by | `xWrite` on `Wal.pWalFd` | stores into a shared memory mapping |
| Synced | yes, at commit when `synchronous=FULL` | **never** |
| Survives a crash | yes | treated as garbage; rebuilt by `walIndexRecover()` |
| Size | `32 + mxFrame × (szPage + 24)` bytes | a multiple of `WALINDEX_PGSZ` = 32768 |
| Can be absent | no | **yes** — see §14 |

The `-shm` file is a cache with a lock table embedded in it. Nothing in it is authoritative: every number it holds can be recomputed by reading the `-wal` file from the front. That is what makes the next section's checksum chain load-bearing — it is the only thing standing between a torn write and a recovery that accepts garbage.

## 2. The WAL File: Two Headers and One Chain

`WAL_HDRSIZE` is 32 and `WAL_FRAME_HDRSIZE` is 24 (`wal.c:477-480`). Frame *i* (1-based) begins at

```c
#define walFrameOffset(iFrame, szPage) (                               \
  WAL_HDRSIZE + ((iFrame)-1)*(i64)((szPage)+WAL_FRAME_HDRSIZE)         \
)
```
— `wal.c:498-500`. There is no padding and no alignment slack: the file is a dense array of frames after a 32-byte prologue, so the frame count is a pure function of the file size.

**WAL header, 32 bytes, all big-endian:**

| Off | Size | Field | Notes |
|----:|-----:|-------|-------|
| 0 | 4 | magic | `0x377f0682` (little-endian frame checksums) or `0x377f0683` (big-endian). `#define WAL_MAGIC 0x377f0682`, `wal.c:491`; the LSB is `SQLITE_BIGENDIAN`, set at `wal.c:4178` |
| 4 | 4 | format version | `3007000` |
| 8 | 4 | page size | |
| 12 | 4 | checkpoint sequence | `pWal->nCkpt` — see §12, it is not what it looks like |
| 16 | 4 | salt-1 | incremented by 1 on every WAL reset |
| 20 | 4 | salt-2 | re-randomized on every WAL reset |
| 24 | 8 | checksum | over bytes 0–23, computed with `nativeCksum=1` (`wal.c:4184`) |

**Frame header, 24 bytes, all big-endian** (`walEncodeFrame`, `wal.c:1001-1025`):

| Off | Size | Field | Notes |
|----:|-----:|-------|-------|
| 0 | 4 | page number | |
| 4 | 4 | `nTruncate` | **non-zero means this frame commits a transaction**, and the value is the database size in pages after that commit. Zero otherwise. This single field is the commit record |
| 8 | 8 | salt-1, salt-2 | copied verbatim from the WAL header |
| 16 | 8 | checksum | cumulative, see below |

### The chain

`walChecksumBytes()` (`wal.c:888-944`) is a two-word Fibonacci-weighted sum over 32-bit words:

```
s1 += x[i]   + s2;
s2 += x[i+1] + s1;
```

Two things make it a *chain* rather than a per-frame checksum. First, `walEncodeFrame` seeds it with `pWal->hdr.aFrameCksum` — the previous frame's output — and runs it over two disjoint regions in order:

```c
walChecksumBytes(nativeCksum, aFrame, 8, aCksum, aCksum);          /* frame hdr bytes 0..7  */
walChecksumBytes(nativeCksum, aData, pWal->szPage, aCksum, aCksum); /* then the page image   */
```
— `wal.c:1017-1018`. Only the first 8 bytes of the frame header are covered: page number and `nTruncate`. The salts and the checksum itself are not. Second, the chain's initial value is the *WAL header's own* checksum (`wal.c:4190-4191`), so frame 1 is cryptographically — well, arithmetically — bound to the header that declares the page size and salts.

The consequence is the one that matters for recovery: a frame is valid only if every frame before it is valid. Recovery stops at the first frame whose checksum does not continue the chain, and that is the end of the WAL. There is no need for a "valid frames" count on disk, which is exactly why there is no such field in the header.

The salts are the second line of defence, and they catch a different failure: a WAL that was reset and partially overwritten. Old frames past the new write point still carry the *old* salts, so `walDecodeFrame` rejects them on a salt mismatch before the checksum is even consulted.

**Measured** (Hands-On A): the 32-byte header checksum, and the full 383-frame chain of a spilling transaction, recomputed from the file bytes in Python with no reference to SQLite's own code, matched at every frame.

## 3. `walFrames()`: the Append Path

`sqlite3WalFrames()` (`wal.c:4361`) is a thin SEH wrapper around `walFrames()` (`wal.c:4124`). The caller is the pager, with a dirty-page list, and `isCommit` set only when this frame set finishes a transaction. The sequence:

1. **Snapshot the live header** (`wal.c:4157-4160`). If the shared wal-index header differs from this connection's cached copy, set `iFirst = pLive->mxFrame + 1`. Since this connection holds the WRITE lock, the only way the two can differ is that *this* connection already spilled frames in this transaction — `walIndexWriteHdr()` runs only on commit. So `iFirst` means "the first frame belonging to my own uncommitted transaction".
2. **Maybe rewind** — `walRestartLog()` (§12).
3. **Write the WAL header if `mxFrame == 0`** (`wal.c:4174-4211`). Salts are randomized only when `pWal->nCkpt == 0` (`wal.c:4182`); otherwise the salts already in `pWal->hdr.aSalt` are reused, which is how `walRestartHdr()`'s increment survives into the file. `truncateOnCommit` is set here, and the header is `fsync`ed if `syncHeader` (§13).
4. **Write each dirty page as a frame**, in dirty-list order (`wal.c:4226-4259`). `nDbSize` is `nTruncate` only for the *last* page of a committing list (`wal.c:4253`) — one commit record per transaction, on the final frame.
5. **Repair checksums** if anything was overwritten in place (§4).
6. **Pad and sync** if committing and `synchronous=FULL` (§13).
7. **Truncate** to `journal_size_limit` if this completed the WAL's first transaction (`wal.c:4306-4313`).
8. **Append to the wal-index** (`wal.c:4320-4331`), one `walIndexAppend()` per frame that actually got appended — marked by the `PGHDR_WAL_APPEND` flag set in step 4.
9. **Publish** (`wal.c:4333-4348`). `pWal->hdr.mxFrame = iFrame` always; `iChange++` and `nPage = nTruncate` only on commit; and `walIndexWriteHdr()` — the step that makes the frames visible to every other connection — **only on commit**.

Step 8 is worth dwelling on. The comment at `wal.c:4315-4318` justifies doing it without any extra lock: the WRITE lock excludes other writers, and appending only ever writes to `aPgno[]`/`aHash[]` entries above every reader's `mxFrame`. Readers are reading the same memory concurrently, unsynchronized, and that is deliberate — §8 explains what `walFindFrame()` does about it.

```mermaid
flowchart LR
    subgraph FILE["-wal file: dense frame array after a 32-byte prologue"]
        direction TB
        WH["offset 0, 32 bytes: WAL header<br/>magic 0x377f0682 | ver 3007000 | szPage<br/>ckptSeq = Wal.nCkpt (per-CONNECTION, see §12)<br/>salt-1, salt-2 | cksum over bytes 0..23"]
        F1["frame 1 @ 32<br/>hdr 24B: pgno | nTruncate=0 | salt1,salt2 | cksum<br/>then szPage bytes of page image"]
        F2["frame 2 @ 32+(szPage+24)<br/>nTruncate=0 -> not a commit"]
        F3["frame 3<br/>nTruncate = N  <<< COMMIT RECORD<br/>db is N pages after this transaction"]
        WH --> F1 --> F2 --> F3
    end

    subgraph CHAIN["the cumulative checksum, walEncodeFrame wal.c:1013-1021"]
        direction TB
        C0["seed = the WAL HEADER's own checksum<br/>(wal.c:4190-4191)"]
        C1["frame i: s = cksum(frame_hdr bytes 0..7, s)<br/>then  s = cksum(page image, s)<br/>bytes 8..23 of the header are NOT covered"]
        C2["store s at frame_hdr offset 16<br/>carry s into frame i+1"]
        C3["recovery walks forward and STOPS at the first<br/>frame whose stored s != recomputed s,<br/>or whose salts differ from the header's.<br/>That is why no valid-frame COUNT is stored."]
        C0 --> C1 --> C2 --> C3
    end

    subgraph APPEND["walFrames() wal.c:4124-4352 — ordering IS the commit protocol"]
        direction TB
        A1["1. iFirst = pLive->mxFrame+1 if shared hdr != my cached hdr<br/>(true only if I already spilled in THIS txn)"]
        A2["2. walRestartLog(): rewind to frame 1 if readLock==0<br/>and no reader holds READ_LOCK(1..4)"]
        A3["3. if mxFrame==0: write the 32-byte WAL header;<br/>randomize salts only when nCkpt==0"]
        A4{"4. page already in MY uncommitted range?<br/>walFindFrame(pgno) >= iFirst"}
        A5["4a. OVERWRITE payload in place at offset+24;<br/>set iReCksum = min(iReCksum, iWrite);<br/>clear PGHDR_WAL_APPEND. No new frame."]
        A6["4b. APPEND a frame. nTruncate = nTruncate only on<br/>the LAST page of a committing list; set PGHDR_WAL_APPEND."]
        A7["5. if committing and iReCksum: walRewriteChecksums()<br/>rebuilds every frame header from iReCksum onward"]
        A8["6. pad to sector boundary + fsync, only if committing,<br/>synchronous=FULL and padToSectorBoundary (OFF on ext4, §13)"]
        A9["7. truncate to journal_size_limit if this finished<br/>the WAL's FIRST transaction"]
        A10["8. walIndexAppend() per PGHDR_WAL_APPEND frame:<br/>aPgno plainly, then aHash with AtomicStore"]
        A11["9. mxFrame = iFrame always;<br/>iChange++ and nPage only on commit;<br/>walIndexWriteHdr() ONLY ON COMMIT"]
        A1 --> A2 --> A3 --> A4
        A4 -->|yes| A5 --> A7
        A4 -->|no| A6 --> A7
        A7 --> A8 --> A9 --> A10 --> A11
    end

    A6 -.->|"writes"| F3
    A5 -.->|"invalidates the chain from iWrite on"| C2
    A7 -.->|"repairs it"| C1
    A11 ==>|"THE commit point: frames on disk,<br/>then index entries, then the header.<br/>A reader sampling mxFrame is guaranteed<br/>every frame and index entry below it exists."| C3

    style F3 fill:#fff3cd,stroke:#856404
    style A5 fill:#f8d7da,stroke:#721c24
    style A11 fill:#d4edda,stroke:#155724
    style C3 fill:#cfe2ff,stroke:#084298
```

The ordering of steps 8 and 9 is the commit point. Frames are on disk, then the index entries, then the header. A reader that samples the header gets a value of `mxFrame` for which all index entries and all frames already exist.

## 4. Overwriting a Frame, and the Checksum Debt It Creates

A long transaction that spills, then dirties the same page again, does **not** append a second frame for it. `walFrames()` looks the page up in its own uncommitted range and overwrites the existing frame's payload in place:

```c
if( iFirst && (p->pDirty || isCommit==0) ){
  u32 iWrite = 0;
  VVA_ONLY(rc =) walFindFrame(pWal, p->pgno, &iWrite);
  if( iWrite>=iFirst ){
    i64 iOff = walFrameOffset(iWrite, szPage) + WAL_FRAME_HDRSIZE;
    if( pWal->iReCksum==0 || iWrite<pWal->iReCksum ) pWal->iReCksum = iWrite;
    rc = sqlite3OsWrite(pWal->pWalFd, pData, szPage, iOff);
    p->flags &= ~PGHDR_WAL_APPEND;
    continue;
  }
}
```
— `wal.c:4233-4249`. Note the offset: `+ WAL_FRAME_HDRSIZE`. Only the page image is rewritten; the 24-byte header is left alone. The frame keeps its page number and its `nTruncate`, both of which are still correct.

But its checksum is now wrong, and so is every checksum after it, because the chain is cumulative. That debt is recorded as `pWal->iReCksum` — the lowest overwritten frame — and paid at commit by `walRewriteChecksums()` (`wal.c:4075`), which re-reads and rewrites frame headers from `iReCksum` through the last frame. While the debt is outstanding, `walEncodeFrame` stops even pretending: `if (pWal->iReCksum == 0) { ...compute... } else { memset(&aFrame[8], 0, 16); }` (`wal.c:1013-1024`). Frames written during a transaction that has already overwritten something get **zeroed salts and zeroed checksums** — unmistakably invalid, so a crash mid-transaction leaves a WAL that recovery truncates at exactly the right place rather than one with plausible-looking garbage.

**Measured** (Hands-On C), `cache_size=10`, one transaction inserting 1499 rows then updating 59 of them:

```
mxFrame = 383
file size = 401416    expected 32 + 383*1048 = 401416
frames in file: 383   chain failures: 0   commit frames: 2
24-byte pwrite64 to -wal : 764      (383 appends + 381 checksum rewrites)
1024-byte pwrite64 to -wal: 407     (383 appends + 24 in-place overwrites)
integrity_check: ok
```

407 page writes for 383 frames: 24 pages were rewritten in place rather than appended. 764 frame-header writes for 383 frames: `walRewriteChecksums()` rebuilt 381 of them at commit. And **2** commit frames for the whole run — `CREATE TABLE` and the one big transaction — confirming that `nTruncate` marks transactions, not frames.

## 5. The Wal-Index Layout

The `-shm` file is a sequence of `WALINDEX_PGSZ` blocks:

```c
#define HASHTABLE_NPAGE      4096                 /* Must be power of 2 */
#define HASHTABLE_HASH_1     383                  /* Should be prime */
#define HASHTABLE_NSLOT      (HASHTABLE_NPAGE*2)  /* Must be a power of 2 */
#define HASHTABLE_NPAGE_ONE  (HASHTABLE_NPAGE - (WALINDEX_HDR_SIZE/sizeof(u32)))
#define WALINDEX_PGSZ   (                                         \
    sizeof(ht_slot)*HASHTABLE_NSLOT + HASHTABLE_NPAGE*sizeof(u32) \
)
```
— `wal.c:647-661`. With `WALINDEX_HDR_SIZE` = 136 (`wal.c:474`), `HASHTABLE_NPAGE_ONE` = 4096 − 34 = **4062**, and `WALINDEX_PGSZ` = 2×8192 + 4×4096 = **32768**. The whole point of the odd 4062 is in the comment at `wal.c:651-655`: shrinking the first block's page-number array by exactly the header size keeps every hash table on an aligned 32 KiB boundary.

Note the naming discrepancy: the published format document calls this constant `HASHTABLE_NPAGE_FIRST`; the source calls it `HASHTABLE_NPAGE_ONE`. Same 4062.

Block 0:

```
   0 .. 47    WalIndexHdr   copy 0
  48 .. 95    WalIndexHdr   copy 1
  96 .. 135   WalCkptInfo   (nBackfill, 5 read marks, 8 lock bytes, nBackfillAttempted, pad)
 136 .. 16383 u32 aPgno[4062]      frame -> page number
16384 .. 32767 u16 aHash[8192]     slot  -> frame index within this block
```

Blocks 1, 2, …: `u32 aPgno[4096]` then `u16 aHash[8192]`.

**`WalIndexHdr`** (`wal.c:321-333`) is 48 bytes: `iVersion`, unused padding, `iChange`, `isInit`, `bigEndCksum`, `szPage` (a `u16`, with 1 standing in for 65536), `mxFrame`, `nPage`, `aFrameCksum[2]`, `aSalt[2]`, `aCksum[2]`.

It is stored **twice**, and `walIndexWriteHdr()` (`wal.c:974-986`) writes the copies in a specific order:

```c
memcpy((void*)&aHdr[1], &pWal->hdr, sizeof(WalIndexHdr));   /* copy 1 first  */
walShmBarrier(pWal);
memcpy((void*)&aHdr[0], &pWal->hdr, sizeof(WalIndexHdr));   /* copy 0 second */
```

A reader compares the two and accepts only if they agree and the internal `aCksum` validates. Copy 1 is written first so that a reader seeing a valid copy 0 knows copy 1 was already complete — the pair is a torn-write detector for a region that is never synced and has no locking.

One detail that is visible from outside and easy to get wrong: `aSalt[2]` is `memcpy`'d from the WAL file bytes (`wal.c:1489`), so it stays in **WAL byte order** — big-endian — inside a structure every other field of which is native-endian. **Measured**: the same salt read as `0x26ee4254` from the WAL header appears as `0x5442ee26` when the `-shm` field is read little-endian. Those are the same four bytes.

**`WalCkptInfo`** (`wal.c:394-400`) is the mutable part:

| Off | Field | Who may write it |
|----:|-------|------------------|
| 96 | `nBackfill` | increased only under `WAL_CKPT_LOCK`; reset to 0 by a WRITE-lock holder on reset (`wal.c:343-346`) |
| 100 | `aReadMark[0]` | never used; a placeholder so indexes need no ±1 (`wal.c:361-363`). Its value is always 0 |
| 104–116 | `aReadMark[1..4]` | only by a holder of an **exclusive** `WAL_READ_LOCK(K)` (`wal.c:367-370`) |
| 120–127 | `aLock[8]` | **never read or written** — `fcntl` targets only (`wal.c:355-356`) |
| 128 | `nBackfillAttempted` | set *before* backfilling; may exceed `nBackfill` after a checkpoint crash |
| 132 | unused | |

The eight lock bytes are asserted into place, not merely documented (`wal.c:1713-1723`):

```c
assert(     5 ==  WAL_NREADER          );
assert(   123 ==  WALINDEX_LOCK_OFFSET + WAL_READ_LOCK(0) );
...
assert(   127 ==  WALINDEX_LOCK_OFFSET + WAL_READ_LOCK(4) );
```

So: byte 120 WRITE, 121 CKPT, 122 RECOVER, 123–127 the five read locks, with `WAL_READ_LOCK(I) = 3+I` and `WAL_NREADER = SQLITE_SHM_NLOCK - 3 = 5` (`wal.c:294-299`).

## 6. `walHash()` Is a Permutation, Not a Hash

```c
static int walHash(u32 iPage){
  assert( iPage>0 );
  assert( (HASHTABLE_NSLOT & (HASHTABLE_NSLOT-1))==0 );
  return (iPage*HASHTABLE_HASH_1) & (HASHTABLE_NSLOT-1);
}
static int walNextHash(int iPriorHash){
  return (iPriorHash+1)&(HASHTABLE_NSLOT-1);
}
```
— `wal.c:1170-1177`. Multiply by 383, mask to 13 bits, linear probing forward.

383 is odd, so multiplication by 383 is **invertible modulo 2^13**. That is not a detail about hash quality; it changes what the function is. `p1*383 ≡ p2*383 (mod 8192)` holds if and only if `p1 ≡ p2 (mod 8192)`. The map is a bijection on residues, so:

- **Distinct page numbers never share a slot** unless they differ by a multiple of 8192.
- The table's load factor is capped at exactly **0.5** by construction, because `HASHTABLE_NSLOT = 2 × HASHTABLE_NPAGE` and one table indexes at most `HASHTABLE_NPAGE` frames.
- Therefore the probe sequences that actually occur come from one of two sources: the **same page appearing in several frames** (the common case, and the whole reason the WAL exists), or pages exactly 8192 apart landing together.

**Measured** (Hands-On B): `383^-1 mod 8192 = 7807`; pages 0..8191 hit 8192 distinct slots with zero collisions; distinct pages 1..4096 in one table give a maximum slot occupancy of 1 at load factor 0.5.

It follows that this is not a hash function trying to spread arbitrary keys. It is an indexing scheme for a dense, small, integer key space, where the designer knows the load factor in advance and uses the cheapest injective scramble available. §Where This Breaks Down returns to what that costs.

`walHashGet()` (`wal.c:1205-1230`) resolves a block index to pointers and a frame offset:

```c
pLoc->aHash = (volatile ht_slot *)&pLoc->aPgno[HASHTABLE_NPAGE];
if( iHash==0 ){
  pLoc->aPgno = &pLoc->aPgno[WALINDEX_HDR_SIZE/sizeof(u32)];
  pLoc->iZero = 0;
}else{
  pLoc->iZero = HASHTABLE_NPAGE_ONE + (iHash-1)*HASHTABLE_NPAGE;
}
```

`iZero` is the frame number just before this block's first frame, so `idx = iFrame - iZero` is a block-local 1-based index — and that `idx`, not the frame number, is what goes in a `u16` hash slot. That is why `ht_slot` can be 16 bits at all: it only ever has to hold 1..4096.

`walFramePage()` (`wal.c:1235-1248`) inverts it: `iHash = (iFrame + HASHTABLE_NPAGE - HASHTABLE_NPAGE_ONE - 1) / HASHTABLE_NPAGE`, followed by five `assert`s pinning the boundary cases.

## 7. `walIndexAppend()`

```c
idx = iFrame - sLoc.iZero;
if( idx==1 ){                      /* first entry in this block: zero it all */
  int nByte = (int)((u8*)&sLoc.aHash[HASHTABLE_NSLOT] - (u8*)sLoc.aPgno);
  memset((void*)sLoc.aPgno, 0, nByte);
}
if( sLoc.aPgno[idx-1] ){           /* a previous writer died mid-transaction */
  walCleanupHash(pWal);
}
nCollide = idx;
for(iKey=walHash(iPage); sLoc.aHash[iKey]; iKey=walNextHash(iKey)){
  if( (nCollide--)==0 ) return SQLITE_CORRUPT_BKPT;
}
sLoc.aPgno[(idx-1)&(HASHTABLE_NPAGE-1)] = iPage;
AtomicStore(&sLoc.aHash[iKey], (ht_slot)idx);
```
— `wal.c:1347-1376`. Four things are doing real work here.

- **Lazy zeroing.** A 32 KiB block is zeroed only when its first frame arrives. Nothing pre-initializes the `-shm` file beyond its header.
- **`aPgno[idx-1]` being non-zero is a diagnosis, not an error.** It means a previous writer appended frames and then vanished without committing. The index entries are still there; `walCleanupHash()` (`wal.c:1271-1326`) removes every `aHash[]` slot whose value exceeds `mxFrame - iZero` and `memset`s the tail of `aPgno[]`. Because `mxFrame` only ever advances past those frames when they are legitimately rewritten, at most one block needs cleaning (`wal.c:1266-1269`).
- **`nCollide = idx` bounds the probe by the number of entries already in the block.** A full-table scan is impossible by construction, and a probe run longer than the population is proof of corruption, not bad luck.
- **Write order: `aPgno[]` plainly, then `aHash[]` with `AtomicStore`.** A reader that sees the hash slot is guaranteed to see the page number behind it. The reverse order would hand readers a slot pointing at an unwritten `aPgno[]` entry.

## 8. `walFindFrame()`: Why the Reader Does Not Stop at the First Match

```c
iMinHash = walFramePage(pWal->minFrame);
for(iHash=walFramePage(iLast); iHash>=iMinHash; iHash--){
  nCollide = HASHTABLE_NSLOT;
  iKey = walHash(pgno);
  while( (iH = AtomicLoad(&sLoc.aHash[iKey]))!=0 ){
    u32 iFrame = iH + sLoc.iZero;
    if( iFrame<=iLast
     && iFrame>=pWal->minFrame
     && sLoc.aPgno[(iH-1)&(HASHTABLE_NPAGE-1)]==pgno
    ){
      assert( iFrame>iRead || CORRUPT_DB );
      iRead = iFrame;
    }
    if( (nCollide--)==0 ){ *piRead = 0; return SQLITE_CORRUPT_BKPT; }
    iKey = walNextHash(iKey);
  }
  if( iRead ) break;
}
```
— `wal.c:3656-3687`. Three predicates, each for a different reason, and they are documented as being *deliberately stricter than necessary* (`wal.c:3645-3655`):

- `aPgno[...] == pgno` filters ordinary probe collisions.
- `iFrame <= iLast` discards frames appended **after this reader's snapshot was taken**. This is the snapshot isolation. The reader's `mxFrame` is a private copy; the index it is reading is live and growing underneath it.
- `iFrame >= pWal->minFrame` discards frames already backfilled into the database, which the reader can get more cheaply from the database file itself. `minFrame` is set to `nBackfill+1` in `walTryBeginRead()` (`wal.c:3341`).

The loop does **not** break on a match. It keeps walking the probe run, overwriting `iRead` each time, because `walIndexAppend()` appends later frames to *later* slots in the same run — so the last match in the run is the newest version of the page. The `assert( iFrame>iRead )` states that invariant. Blocks are searched newest-first and the loop exits on the first block that yields anything.

Why unsynchronized reads of a concurrently-mutated hash table are safe: every slot a reader can legitimately accept was written before its snapshot was taken, and the two frame-range predicates reject exactly the slots that might be mid-write. A garbage slot is not merely unlikely — it is rejected by `iFrame <= iLast`.

**Measured** (Hands-On B), a 14-frame WAL in which page 1 appears in frames 1 and 3 and page 2 in frames 2 and 4:

```
aPgno[0..7] (frame -> pgno): [(1,1),(2,2),(3,1),(4,2),(5,3),(6,4),(7,5),(8,6)]
non-zero aHash slots (14): [(383,1),(384,3),(766,2),(767,4),(1149,5),(1532,6),(1915,7),(2298,8)]
  frame 1 pgno 1: walHash=383, aHash[383]=1
  frame 3 pgno 1: walHash=383, aHash[384]=3
```

Slot 383 holds frame 1; the probe lands frame 3 in slot 384. A reader looking for page 1 at `iLast=14` walks 383 → 384, matches both, and returns **3**. A reader whose snapshot is `iLast=2` matches only slot 383 and returns 1. The two readers get different versions of page 1 out of the same live table with no locking between them. That is the whole concurrency story in two slots.

The count also confirms the invariant `walIndexAppend()` asserts under `SQLITE_ENABLE_EXPENSIVE_ASSERT`: 14 non-zero hash slots for `mxFrame = 14`.

## 9. Five Read Marks, and What a Mark Actually Is

A read mark is a published promise: *some reader may be using frames up to this number, so do not overwrite them.* Its enforcement is the pairing of two things — the `u32` value in `aReadMark[K]` and a shared `fcntl` lock on byte `120 + 3 + K`.

The rules (`wal.c:358-382`):

- `aReadMark[K]` may be modified only while holding `WAL_READ_LOCK(K)` **exclusively**. A reader using mark *K* holds it **shared**. So the value cannot change under a reader — the lock mode is the whole synchronization mechanism, and no separate mutex exists.
- `aReadMark[0]` is a placeholder whose value is always 0. A reader holding `WAL_READ_LOCK(0)` ignores the WAL entirely and reads everything from the database file.
- `READMARK_NOT_USED` is `0xffffffff`.
- A checkpointer may backfill only up to the **minimum** `aReadMark[j]` over all *in-use* marks. "In use" means a reader holds `WAL_READ_LOCK(j)`, which the checkpointer discovers by trying to take it exclusively.
- 32-bit loads are assumed atomic, so reading a mark needs no lock (`wal.c:391-392`).

Five marks means **at most four distinct snapshots** can pin the WAL at once (mark 0 pins nothing). A fifth concurrent reader does not fail; it shares the nearest existing mark, which simply gives it a slightly older snapshot than it could have had.

## 10. `walTryBeginRead()`: Picking a Mark

`wal.c:3102-3354`. After header validation and possible recovery, the body is three stages.

**Stage 1 — the shortcut** (`wal.c:3216-3249`). If `nBackfill == pWal->hdr.mxFrame`, the entire WAL is already in the database file, so take `WAL_READ_LOCK(0)` and read nothing from the WAL. Then immediately re-check:

```c
if( memcmp((void *)walIndexHdr(pWal), &pWal->hdr, sizeof(WalIndexHdr)) ){
  walUnlockShared(pWal, WAL_READ_LOCK(0));
  return WAL_RETRY;
}
```

The comment (`wal.c:3228-3240`) gives the precise hazard: holding `READ_LOCK(0)` asserts the *database file* is a trustworthy snapshot. If frames were appended before the lock was acquired, a checkpointer could have started backfilling them and crashed, leaving a torn image in the file. Taking the lock prevents future checkpoints; it says nothing about one already in flight.

**Stage 2 — select or allocate** (`wal.c:3256-3291`):

```c
for(i=1; i<WAL_NREADER; i++){
  u32 thisMark = AtomicLoad(pInfo->aReadMark+i);
  if( mxReadMark<=thisMark && thisMark<=mxFrame ){ mxReadMark = thisMark; mxI = i; }
}
if( (pWal->readOnly & WAL_SHM_RDONLY)==0 && (mxReadMark<mxFrame || mxI==0) ){
  for(i=1; i<WAL_NREADER; i++){
    rc = walLockExclusive(pWal, WAL_READ_LOCK(i), 1);
    if( rc==SQLITE_OK ){
      AtomicStore(pInfo->aReadMark+i, mxFrame);
      mxReadMark = mxFrame; mxI = i;
      walUnlockExclusive(pWal, WAL_READ_LOCK(i), 1);
      break;
    }else if( rc!=SQLITE_BUSY ) return rc;
  }
}
if( mxI==0 ) return rc==SQLITE_BUSY ? WAL_RETRY : SQLITE_READONLY_CANTINIT;
```

The best existing mark not exceeding this reader's `mxFrame` wins. If none equals `mxFrame`, try to claim a *free* mark and set it — "free" being decided by whether the exclusive lock succeeds, since an in-use mark is held shared by its reader. Failure to claim one is not an error; the reader proceeds with the best mark it found. A read-only connection on a read-only `-shm` cannot write marks at all, which is where `SQLITE_READONLY_CANTINIT` comes from.

**Stage 3 — lock, then verify twice** (`wal.c:3293-3351`):

```c
pWal->minFrame = AtomicLoad(&pInfo->nBackfill)+1;
walShmBarrier(pWal);
if( AtomicLoad(pInfo->aReadMark+mxI)!=mxReadMark
 || memcmp((void *)walIndexHdr(pWal), &pWal->hdr, sizeof(WalIndexHdr))
){
  walUnlockShared(pWal, WAL_READ_LOCK(mxI));
  return WAL_RETRY;
}
```

Two checks because there are two ways the ground can move between reading the header and locking the mark: a writer may have wrapped the log, or a checkpointer may have backfilled past `pWal->hdr.mxFrame`. Either makes the cached header describe a WAL that no longer exists. `WAL_RETRY` goes back to the top; `WAL_RETRY_PROTOCOL_LIMIT` (`wal.c:3033-3037`) eventually converts persistent retries into `SQLITE_PROTOCOL` rather than spinning forever.

The barrier between sampling `nBackfill` and the `memcmp` is load-bearing, and `wal.c:3327-3339` spells out the race it closes: it guarantees the checkpointer that set `nBackfill` could not have been working from a header newer than the one cached here, which is what makes "frames below `minFrame` are safe to read from the database file" sound.

**Measured** (Hands-On D), four readers opened at four successive commits, `wal_autocheckpoint=0`:

```
reader 1 open   mxFrame=3  nBackfill=0  aReadMark=['0','3','NOT_USED','NOT_USED','NOT_USED']
reader 2 open   mxFrame=4  nBackfill=0  aReadMark=['0','3','4','NOT_USED','NOT_USED']
reader 3 open   mxFrame=5  nBackfill=0  aReadMark=['0','3','4','5','NOT_USED']
reader 4 open   mxFrame=6  nBackfill=0  aReadMark=['0','3','4','5','6']
reader 5 open   mxFrame=7  nBackfill=0  aReadMark=['0','3','4','5','6']
```

Marks fill 1→4 with distinct snapshots. Reader 5 arrives at `mxFrame=7`, finds no free mark, and shares mark 4 at frame 6 — one commit stale, by design, silently.

And the floor on backfill, measured directly (Hands-On E):

```
old reader pinned                  mxFrame=3   nBackfill=0   aReadMark=['0','3','NU','NU','NU']
4 more commits                     mxFrame=10  nBackfill=0   aReadMark=['0','3','6','NU','NU']
  pragma wal_checkpoint(PASSIVE) -> (0, 10, 3)
after PASSIVE ckpt WITH old reader mxFrame=10  nBackfill=3   nBackfillAttempted=3
  old reader still sees count = 1
  pragma wal_checkpoint(PASSIVE) -> (0, 10, 10)
after PASSIVE ckpt, reader gone    mxFrame=10  nBackfill=10  nBackfillAttempted=10
```

`PASSIVE` returned `(busy=0, nLog=10, nCkpt=3)`: not busy, 10 frames in the log, **3 backfilled**. The checkpoint stopped exactly at `aReadMark[1] = 3`. One reader holding a seven-frame-old snapshot capped the checkpoint at 30% completion — and reported success while doing it. Releasing the reader let the next identical call reach 10.

After the checkpoint the marks collapse to `['0','10','NU','NU','NU']`, and a reader arriving when `nBackfill == mxFrame` takes mark 0:

```
after TRUNCATE checkpoint         mxFrame=0  nBackfill=0  aReadMark=['0','0','NOT_USED',...]
new reader, WAL fully backfilled  mxFrame=0  nBackfill=0  aReadMark=['0','0','NOT_USED',...]
```

No new mark appears, because stage 1 short-circuited.

```mermaid
flowchart TB
    subgraph SHM["-shm block 0 — 32768 bytes, never synced"]
        direction TB
        H0["bytes 0-47: WalIndexHdr copy 0<br/>iVersion, iChange, isInit, bigEndCksum,<br/>szPage, mxFrame, nPage,<br/>aFrameCksum[2], aSalt[2] (WAL byte order), aCksum[2]"]
        H1["bytes 48-95: WalIndexHdr copy 1<br/>(written FIRST, barrier, then copy 0)"]
        CK["bytes 96-135: WalCkptInfo<br/>96 nBackfill | 100-116 aReadMark[0..4]<br/>120-127 aLock[8] (fcntl only, never read)<br/>128 nBackfillAttempted | 132 pad"]
        PG["bytes 136-16383: u32 aPgno[4062]<br/>aPgno[idx-1] = page number of frame idx<br/>HASHTABLE_NPAGE_ONE = 4096 - 136/4 = 4062"]
        HT["bytes 16384-32767: u16 aHash[8192]<br/>aHash[slot] = idx (block-local, 1..4096)<br/>load factor capped at 0.5 by NSLOT = 2*NPAGE"]
        H0 --> H1 --> CK --> PG --> HT
    end

    subgraph LOCKS["the 8 lock bytes, asserted at wal.c:1713-1723"]
        L0["120 WAL_WRITE_LOCK — one writer"]
        L1["121 WAL_CKPT_LOCK — one checkpointer"]
        L2["122 WAL_RECOVER_LOCK — recovery in progress"]
        L3["123 READ_LOCK(0) — holder IGNORES the WAL"]
        L4["124-127 READ_LOCK(1..4) — shared by readers,<br/>exclusive to change aReadMark[K]"]
    end

    subgraph LOOKUP["walFindFrame(pgno) — wal.c:3656-3687"]
        direction TB
        Q1["iKey = (pgno * 383) & 8191<br/>383 is odd, so this is a PERMUTATION of the slots"]
        Q2{"aHash[iKey] != 0 ?"}
        Q3["iFrame = aHash[iKey] + iZero"]
        Q4{"aPgno[iH-1] == pgno<br/>AND iFrame <= iLast<br/>AND iFrame >= minFrame"}
        Q5["iRead = iFrame<br/>(do NOT break — a later slot in the<br/>same probe run holds a NEWER frame)"]
        Q6["iKey = (iKey+1) & 8191<br/>bounded by nCollide, else SQLITE_CORRUPT"]
        Q7["return iRead: newest frame <= this<br/>reader's snapshot, or 0 -> read the db file"]
        Q1 --> Q2
        Q2 -->|yes| Q3 --> Q4
        Q4 -->|match| Q5 --> Q6
        Q4 -->|no match| Q6
        Q6 --> Q2
        Q2 -->|"slot == 0: end of probe run"| Q7
    end

    HT -.->|"read unsynchronized,<br/>AtomicLoad per slot"| Q2
    PG -.->|"read unsynchronized"| Q4
    H0 -.->|"mxFrame snapshotted<br/>into pWal->hdr at BEGIN;<br/>becomes iLast"| Q4
    CK -.->|"minFrame = nBackfill + 1<br/>(wal.c:3341)"| Q4
    L4 -.->|"shared lock pins aReadMark[K];<br/>checkpoint may not backfill past<br/>min(in-use marks)"| CK

    style H0 fill:#d4edda,stroke:#155724
    style HT fill:#cfe2ff,stroke:#084298
    style Q5 fill:#fff3cd,stroke:#856404
    style L4 fill:#f8d7da,stroke:#721c24
```

## 11. The One `memcmp` That Is `SQLITE_BUSY_SNAPSHOT`

`sqlite3WalBeginWriteTransaction()` (`wal.c:3785-3832`) is short, and almost all of it is one test:

```c
rc = walLockExclusive(pWal, WAL_WRITE_LOCK, 1);
if( rc ) return rc;                      /* -> SQLITE_BUSY: another writer      */
pWal->writeLock = 1;
if( memcmp(&pWal->hdr, (void *)walIndexHdr(pWal), sizeof(WalIndexHdr))!=0 ){
  rc = SQLITE_BUSY_SNAPSHOT;             /* someone committed since my BEGIN     */
}
```

Two distinct failures with two distinct codes, and the difference matters operationally. `SQLITE_BUSY` on the lock means *a writer is active now* — retrying later can succeed. `SQLITE_BUSY_SNAPSHOT` means *this connection's read snapshot is stale* — retrying is futile until the read transaction is ended, because the snapshot is pinned to the one the connection already observed. A busy handler cannot resolve it; only `ROLLBACK` can. This is WAL mode's entire concurrency control for writes: no row locks, no predicate locks, just "has the wal-index header changed since I started reading".

Note also the precondition `assert( pWal->readLock>=0 )` (`wal.c:3800`): a write transaction is always nested inside a read transaction. There is no path that takes the WRITE lock without a snapshot first.

**Measured** (Hands-On E): a connection that opened a read transaction, then had another connection commit, then tried to insert:

```
stale reader upgrading to writer -> OperationalError 'database is locked'
                                    sqlite_errorname= SQLITE_BUSY_SNAPSHOT
```

The extended code is the only way to tell this apart from ordinary contention; the primary code and message are identical to plain `SQLITE_BUSY`. An application that retries on `"database is locked"` without checking the extended code will spin forever on this one.

## 12. WAL Reset — and a Checkpoint Counter That Is Not Global

`walRestartLog()` (`wal.c:3961-4001`) runs at the top of every `walFrames()` call. It does nothing unless `pWal->readLock == 0`, i.e. this writer's own read transaction took mark 0, which (per §10 stage 1) can only happen when `nBackfill == mxFrame`. So: everything is backfilled. If additionally no reader holds marks 1–4 — tested by grabbing all four exclusively in one call, `walLockExclusive(pWal, WAL_READ_LOCK(1), WAL_NREADER-1)` — the log is rewound.

```c
static void walRestartHdr(Wal *pWal, u32 salt1){
  u32 *aSalt = pWal->hdr.aSalt;
  pWal->nCkpt++;
  pWal->hdr.mxFrame = 0;
  sqlite3Put4byte((u8*)&aSalt[0], 1 + sqlite3Get4byte((u8*)&aSalt[0]));
  memcpy(&pWal->hdr.aSalt[1], &salt1, 4);
  walIndexWriteHdr(pWal);
  AtomicStore(&pInfo->nBackfill, 0);
  pInfo->nBackfillAttempted = 0;
  pInfo->aReadMark[1] = 0;
  for(i=2; i<WAL_NREADER; i++) pInfo->aReadMark[i] = READMARK_NOT_USED;
}
```
— `wal.c:2234-2248`. Salt-1 is **incremented by one** in big-endian; salt-2 is replaced with four fresh random bytes, which the caller generated (`sqlite3_randomness(4, &salt1)`, `wal.c:3970` — the local is called `salt1` but lands in `aSalt[1]`, which is salt-**2** on disk). Together they guarantee stale frames past the new write point can never be mistaken for live ones. The WAL file itself is **not truncated**; writing restarts at offset 32.

Then comes the part that is not obvious. `pWal->nCkpt` is assigned in exactly one place outside `walRestartHdr()`: `wal.c:1488`, inside `walIndexRecover()`, from the WAL header's bytes 12–15. A connection that opened a healthy WAL and never ran recovery therefore keeps a **private** counter starting at whatever it read once. `walFrames()` writes that private value into the file header on every restart (`wal.c:4181`).

**Measured** (Hands-On E): two different connections each triggered a reset. Salt-1 advanced `0xcec8a878 → 0xcec8a879` and salt-2 changed completely (`0xcaff79c3 → 0x620da983`), the file size stayed at 10512 bytes — rewound, not truncated — and `nBackfill` and `aReadMark[1]` reset to 0. But the on-disk checkpoint sequence stayed at **1** across both resets, because the second reset was performed by a connection whose own `nCkpt` was still 0.

So bytes 12–15 of the WAL header are not a global generation count and should not be read as one. Their only consumer is savepoint bookkeeping: `sqlite3WalSavepoint()` stores `pWal->nCkpt` in `aWalData[3]` (`wal.c:3909`) and `sqlite3WalSavepointUndo()` compares it to detect "the log wrapped after this savepoint was opened", in which case it rolls the savepoint's frame position back to 0 (`wal.c:3924-3931`). For that purpose a per-connection counter is sufficient, because only that connection's savepoints are being compared. It is adequate for its one job and misleading for any other.

## 13. Padding, Header Syncs, and Two Flags That Are Off Here

`sqlite3WalOpen()` initializes both flags optimistically and then clears them from the device characteristics (`wal.c:1752-1772`):

```c
pRet->syncHeader = 1;
pRet->padToSectorBoundary = 1;
...
int iDC = sqlite3OsDeviceCharacteristics(pDbFd);
if( iDC & SQLITE_IOCAP_SEQUENTIAL ){ pRet->syncHeader = 0; }
if( iDC & SQLITE_IOCAP_POWERSAFE_OVERWRITE ){ pRet->padToSectorBoundary = 0; }
```

[Lesson 16](../2026-10-01-sqlite-rollback-journal-atomic-commit/README.md) established what these two flags are, and [lesson 18](../2026-10-03-sqlite-vfs-locking-styles-inode-emulation/README.md) measured that on ordinary Linux ext4 `POWERSAFE_OVERWRITE` is **set** and `SEQUENTIAL` is **not**. Applying those facts here: `padToSectorBoundary` is 0 and `syncHeader` is 1 on this system. So the padding block at `wal.c:4283-4295` — which repeats the final commit frame until the next sector boundary so that only whole sectors are synced — is dead code on an ordinary Linux filesystem. It exists for devices that can scramble a partially-overwritten sector.

**Measured** (Hands-On F), four commits at `page_size=1024`, `strace -e trace=openat,fdatasync`:

| | frames in `-wal` | `fdatasync` on the `-wal` fd |
|---|---:|---:|
| `synchronous=NORMAL` | 5 | 4 |
| `synchronous=FULL` | 5 | 8 |

Two readings. **The frame count is identical** — five frames for four commits (one page-1 frame plus four commit frames on page 2), with no duplicated tail frames in either mode. That is `padToSectorBoundary == 0` confirmed from outside. And **`FULL` costs exactly one extra `fdatasync` of the `-wal` file per commit**: four commits, four extra syncs, nothing else different. That is `WAL_SYNC_FLAGS(sync_flags)` being zero at `NORMAL`, so the sync at `wal.c:4298` is skipped while everything else in `walFrames()` runs unchanged.

This is the concrete content of the WAL durability tradeoff. At `synchronous=NORMAL` a commit is `pwrite`s with no barrier: atomic and visible, but a power cut can lose recent transactions. The remaining four syncs in both columns belong to the one-time journal-mode conversion and the WAL header write.

## 14. The Wal-Index Without a File

The `-shm` file is not required. `walIndexPage()` (`wal.c:810-818`) has two paths:

```c
if( pWal->exclusiveMode==WAL_HEAPMEMORY_MODE ){
  pWal->apWiData[iPage] = (u32 volatile *)sqlite3MallocZero(WALINDEX_PGSZ);
}else{
  rc = sqlite3OsShmMap(pWal->pDbFd, iPage, WALINDEX_PGSZ, ...);
}
```

`WAL_HEAPMEMORY_MODE` is selected at open when `bNoShm` (`wal.c:1754`), and in that mode `walShmBarrier()` becomes a no-op (`wal.c:950-953`) and `walIndexClose()` frees the pages instead of unmapping them (`wal.c:1651-1661`). The wal-index becomes ordinary heap memory: identical layout, identical hash tables, zero sharing. The lock bytes are never touched because there is nobody to exclude.

This is the mechanism behind the `sqlite3PagerWalSupported()` short-circuit [lesson 18 measured](../2026-10-03-sqlite-vfs-locking-styles-inode-emulation/README.md): `locking_mode=EXCLUSIVE` makes WAL work on a VFS with no `xShmMap` at all.

**Measured** (Hands-On G), `file:h.db?vfs=unix-dotfile` with `locking_mode=EXCLUSIVE`:

```
locking_mode -> ('exclusive',)
journal_mode -> ('wal',)
files present: ['h.db', 'h.db-wal', 'h.db.lock']
7 frame slots of 4120 bytes ... all chain_ok=True
```

WAL mode, five committed transactions, a fully valid checksum chain — and **no `-shm` file in the directory at all**. (`h.db.lock` is the dotfile VFS's lock *directory*, nothing to do with the wal-index.) Every number the wal-index would have held existed only in this process's heap and vanished with it.
