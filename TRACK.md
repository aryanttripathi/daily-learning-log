<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 4
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 04 shipped 2026-10-23.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 05**.

The order follows PostgreSQL's actual source layout rather than a textbook outline. It was built
from two pieces of primary evidence read on 2026-10-19: the chapter list of
[Part VII Internals](https://www.postgresql.org/docs/current/internals.html) in the official
documentation, and the set of in-tree design documents in `postgres/postgres@45fc51fc`, which is
where the project writes down how each module actually works:

```
src/backend/access/{brin,gin,gist,hash,nbtree,spgist,rmgrdesc}/README
src/backend/access/heap/README.HOT        src/backend/access/heap/README.tuplock
src/backend/access/transam/README         src/backend/access/transam/README.parallel
src/backend/executor/README               src/backend/optimizer/README
src/backend/replication/README            src/backend/storage/aio/README.md
src/backend/storage/buffer/README         src/backend/storage/freespace/README
src/backend/storage/lmgr/README           src/backend/storage/lmgr/README-SSI
src/backend/storage/lmgr/README.barrier   src/backend/storage/page/README
src/backend/storage/smgr/README           src/backend/utils/mmgr/README
src/backend/utils/{fmgr,mb,misc,resowner}/README
```

Those files are the backbone of this track; the documentation and `doxygen.postgresql.org` are the
supplement. The layering runs: on-disk format → shared-memory buffers and locks → MVCC →
isolation → WAL and recovery → access methods → planner and executor.

**Why PostgreSQL after SQLite.** The four tradeoffs the capstone identified — catalog placement,
writer concurrency, cost-model resolution, and the verification oracle — are each resolved the
*opposite* way here: a catalog of real tables, many concurrent writers with per-tuple visibility,
a floating-point cost model, and no `integrity_check` at all. Every lesson in this track should be
readable as the other answer to a question SQLite already answered.

## Syllabus

### Part I — Physical storage and the heap (`access/heap`, `storage/page`, `storage/smgr`)

- [x] 01 — The cluster on disk: `base/<dboid>/<relfilenode>`, the three forks, and 1 GB segment files
- [x] 02 — The page: `PageHeaderData`'s 24 bytes, `pd_lower`/`pd_upper`, the line pointer array, and special space
- [x] 03 — `HeapTupleHeaderData`: `t_xmin`, `t_xmax`, `t_ctid`, `t_infomask`/`t_infomask2`, and `t_hoff`
- [x] 04 — Attribute storage: MAXALIGN padding between attributes, 1-byte vs 4-byte varlena headers, `atthasmissing`, and TOAST
- [ ] 05 — The Free Space Map: the FSM tree pages and how a page with room is found
- [ ] 06 — The Visibility Map: `PD_ALL_VISIBLE`, all-frozen, and the index-only scan it enables
- [ ] 07 — HOT updates and page pruning: redirect line pointers, `heap_page_prune()`, and the HOT chain
- [ ] 08 — `smgr` and `md.c`: relation extension, the segment map, and the fsync request queue

### Part II — Shared memory: buffers and locks (`storage/buffer`, `storage/lmgr`)

- [ ] 09 — The buffer manager: `BufferDesc`, the clock-sweep victim search, and the strategy ring
- [ ] 10 — Buffer pins versus content locks, and the LWLock implementation
- [ ] 11 — The heavyweight lock manager: `LOCKTAG`, lock partitions, and the deadlock detector
- [ ] 12 — Tuple locks: the four strengths, the conflict table, and the `LockTuple`/`XactLockTableWait` protocol

### Part III — MVCC and transaction management (`access/transam`)

- [ ] 13 — Transaction identifiers: xid assignment, `pg_xact`, and the two status bits per transaction
- [ ] 14 — Snapshots: `SnapshotData`'s `xmin`/`xmax`/`xip`, and `HeapTupleSatisfiesMVCC`'s decision order
- [ ] 15 — Hint bits: why the first read after a commit writes, and what that costs
- [ ] 16 — Subtransactions: `pg_subtrans`, the subxid cache, and combo CIDs
- [ ] 17 — MultiXacts: `pg_multixact`'s offsets and members, member flags, and multixact freezing
- [ ] 18 — Freezing and wraparound: `relfrozenxid`, `datfrozenxid`, and the anti-wraparound VACUUM
- [ ] 19 — VACUUM: the two heap passes, the dead-TID store, and index bulk deletion

### Part IV — Isolation (`storage/lmgr/README-SSI`)

- [ ] 20 — Serializable Snapshot Isolation: SIREAD locks, the dangerous-structure test, and false positives

### Part V — WAL, recovery, and replication (`access/transam`, `replication`)

- [ ] 21 — The WAL record: `XLogRecord`, block references, and the resource manager table
- [ ] 22 — `XLogInsert`: WAL insertion locks, registered buffers, and full-page images
- [ ] 23 — Commit: `XLogFlush`, `synchronous_commit` levels, and group commit
- [ ] 24 — Checkpoints: the checkpointer's spread writes and the REDO pointer
- [ ] 25 — Crash recovery: the redo loop and the per-page LSN comparison
- [ ] 26 — Streaming replication: walsender, walreceiver, and physical replication slots
- [ ] 27 — Logical decoding: the reorder buffer and the output plugin callbacks

### Part VI — Access methods (`access/nbtree`, `indexam`)

- [ ] 28 — nbtree: the Lehman-Yao variant, the high key, and right-links
- [ ] 29 — nbtree page splits, deduplication, and the half-dead page deletion dance
- [ ] 30 — The index AM interface: `amgettuple`, `amgetbitmap`, and the bitmap heap scan

### Part VII — Planner and executor (`optimizer`, `executor`)

- [ ] 31 — The planner: `RelOptInfo`, `Path`, the cost constants, and the join-order search
- [ ] 32 — The executor: the `PlanState` tree, `ExecProcNode`, and tuple slots
- [ ] 33 — Row locking in the executor: `LockRows`, `table_tuple_lock`, and EvalPlanQual re-checks
- [ ] 34 — Parallel query: parallel safety, the Gather node, and shared tuple queues
- [ ] 35 — The table AM interface: what `heap` assumes and what a different table AM must supply

<!--
NOTE BLOCK POLICY (set 2026-10-12, lesson 27; carried into the PostgreSQL track 2026-10-19).
This block holds the scope and handover notes for the three most recent lessons only. Older notes
are not deleted: they are preserved verbatim in this file's own git history, and TRACK-ARCHIVE.md
records exactly which commit and blob to read for each archived range. Never drop a note without
adding its archive row first.

-- SQLITE TRACK CLOSED 2026-10-19 (capstone). --------------------------------------------------
The SQLite track's lessons 31-33 notes were archived on that run; see TRACK-ARCHIVE.md, row
"31 - 33". The capstone document (2026/10/2026-10-19-sqlite-capstone/README.md) closes the track
and records which items were deliberately left unread. None of these carry forward into the
PostgreSQL track. Do not hand them to a PostgreSQL lesson.

-- LESSON 02 (2026-10-21) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-21-postgres-page-header-line-pointers/README.md (single README, not
split). Source commit read: postgres/postgres@3cde90ec. Lesson 01 read @ddd468ca; the tree moved
again between the two runs, which lesson 02 states explicitly in its preamble. Keep doing that.

TOOK, and must not be re-taught:
- PageHeaderData BY BYTE OFFSET: 0 pd_lsn (8), 8 pd_checksum (2), 10 pd_flags (2), 12 pd_lower (2),
  14 pd_upper (2), 16 pd_special (2), 18 pd_pagesize_version (2), 20 pd_prune_xid (4), 24 pd_linp[].
  SizeOfPageHeaderData = offsetof(PageHeaderData, pd_linp) = 24 and NOT sizeof, with the
  "line pointer(s) do not count as part of header" comment. MAXALIGN(24) = 24 so no padding
  precedes pd_linp; PageGetContents() still MAXALIGNs and carries the pre-8.3 warning.
- THE FINGERPRINT, established and reusable: bytes 16-19 little-endian identify the access method
  and build. Heap (and sequences, TOAST) = "00 20 04 20" (pd_special 8192, pd_pagesize_version
  0x2004). nbtree = "f0 1f 04 20" (pd_special 8176). A freshly PageInit'ed 8 KB heap page reads
  "18 00 00 20 00 20 04 20" at offsets 12-19. Later lessons may use this without re-deriving it.
- pd_pagesize_version packs size in the high 8 bits and version in the low 8; works only because
  page sizes are constrained to multiples of 256; PG_PAGE_LAYOUT_VERSION = 4 since 8.3;
  PageSetPageSizeAndVersion ASSERTS rather than masks; the size is only a cross-check, docs say
  "no support for having more than one page size in an installation".
- pd_lsn: PageXLogRecPtr is now one uint64 but the accessors swap 32-bit halves on little-endian
  "for historical reasons". CONSEQUENCE TAUGHT: on LE machines bytes 0-3 are the HIGH half and
  bytes 4-7 the LOW half, each LE, so a hexdump reads in the same order as the printed X/Y form.
- pd_flags: PD_HAS_FREE_LINES 0x0001, PD_PAGE_FULL 0x0002, PD_ALL_VISIBLE 0x0004,
  PD_VALID_FLAG_BITS 0x0007. First two are hints and PD_HAS_FREE_LINES is NOT WAL-logged.
  PageIsVerified rejects any undefined bit, so a fourth flag is a compatibility boundary.
  PD_ALL_VISIBLE was NAMED AND DEFERRED to lesson 06 -- lesson 06 still owns it entirely.
- pd_prune_xid: PageSetPrunable only ever lowers it (TransactionIdPrecedes); it is a minimum;
  "currently unused in index pages". Named and DEFERRED to lesson 07.
- ItemIdData {lp_off:15, lp_flags:2, lp_len:15} = 4 bytes. THIS IS WHY BLCKSZ CAPS AT 32768 --
  stated twice in bufpage.h (the LocationIndex comment and "we can only support pages up to 32KB").
  Lesson 01 deferred this; it is now closed. Hand-decode on LE gcc/clang:
  lp_off = u & 0x7FFF, lp_flags = (u>>15)&3, lp_len = (u>>17)&0x7FFF -- taught WITH the caveat
  that C bitfield order is implementation-defined and that this is why heap_page_items() exists.
- The four lp_flags states and the itemid.h convention "lp_len == 0 in every line pointer that
  does not have storage, independently of its lp_flags state". LP_UNUSED 0 / LP_NORMAL 1 /
  LP_REDIRECT 2 / LP_DEAD 3. LP_REDIRECT reuses lp_off as an OFFSET NUMBER (ItemIdGetRedirect is
  an alias of ItemIdGetOffset) -- named and DEFERRED to lesson 07, which owns HOT chains.
  ItemIdHasStorage (lp_len != 0) vs ItemIdIsUsed (lp_flags != LP_UNUSED) are different questions.
  ItemIdSetDead clears storage, ItemIdMarkDead keeps it and carries the "used as if it were a
  hint-bit mechanism ... multiple processors in parallel" comment.
- Offset numbers are 1-based; PageGetItemId subtracts one. The item count is NOT STORED:
  PageGetMaxOffsetNumber = (pd_lower - SizeOfPageHeaderData) / sizeof(ItemIdData). Framed as the
  deliberate removal of SQLite's redundant cell count, and therefore of a cross-check.
- PageInit: MemSets the WHOLE page, sets pd_lower = 24, pd_upper = pd_special = pageSize -
  MAXALIGN(specialSize), sets size|version, and deliberately does NOT compute a checksum.
  PageIsNew is pd_upper == 0; PageIsEmpty is pd_lower <= 24. Different questions.
- PageAddItemExtended: the four-way pointer sanity check as ereport(PANIC); the
  "!!! EREPORT(ERROR) IS DISALLOWED HERE !!!" banner (critical section); first-fit scan from
  FirstOffsetNumber *in order to help PageTruncateLinePointerArray*, which can only reclaim a
  contiguous unused run at the array tail; the hint-is-wrong-so-clear-it path; three WARNING
  refusals ("will not overwrite a used ItemId", "specified item offset is too large", "can't put
  more than MaxHeapTuplesPerPage items in a heap page"); and the SILENT InvalidOffsetNumber when
  lower > upper, because no-room is an expected outcome. Signed-int arithmetic everywhere, with
  the stated reason.
- THE 8160 vs 8164 DERIVATION, established and reusable: MaxHeapTupleSize = BLCKSZ -
  MAXALIGN(SizeOfPageHeaderData + sizeof(ItemIdData)) = 8192 - 32 = 8160. PageGetFreeSpace on a
  fresh heap page = 8192 - 24 - 4 = 8164. Insert needs MAXALIGN(size) <= 8164, and the largest
  multiple of 8 that satisfies that is 8160 -- so the two numbers differ by 4 purely because of
  MAXALIGN. MaxHeapTuplesPerPage = (8192 - 24) / (MAXALIGN(23) + 4) = 8168/28 = 291, where
  SizeofHeapTupleHeader = offsetof(HeapTupleHeaderData, t_bits) = 23 (lesson 03 owns the struct).
- The four free-space functions and why there are four: PageGetExactFreeSpace (raw hole),
  PageGetFreeSpace (minus one ItemIdData, for INDEX pages), PageGetFreeSpaceForMultipleTuples,
  PageGetHeapFreeSpace (returns 0 at the 291 ceiling, because LP_REDIRECT/LP_DEAD slots cost 4
  bytes and zero item space; it exists to protect MaxHeapTuplesPerPage-sized stack arrays such as
  PageRepairFragmentation's itemidbase[]).
- compactify_tuples' two modes: presorted => plain memmove toward the page end, with a skip of the
  already-correct suffix; not presorted => temp buffer, because moving in place could overwrite an
  unmoved tuple, and the copy-back RESTORES the presorted property for next time.
  PageRepairFragmentation detects presortedness while building itemidbase and uses ereport(ERROR)
  (not PANIC) with the "clobber adjacent disk buffers, spreading the data loss further" rationale.
- CHECKSUMS, fully taught: modified FNV-1a, hash = (hash^value)*FNV_PRIME ^ ((hash^value)>>17),
  FNV_PRIME 16777619, shift 17 chosen for no common denominator with FNV_PRIME's set bits;
  N_SUMS = 32 columns of 32-bit words each with its own random base offset from
  checksumBaseOffsets[], 32 chosen as the largest state fitting x86 SSE registers and FIXED
  because changing it changes the result; two extra rounds with value 0; xor-fold to 32 bits;
  then checksum ^= blkno (transposed-page detection) and (checksum % 65535) + 1 so it is never 0.
  pg_checksum_page transiently zeroes pd_checksum and restores it.
- CHECKSUM VALIDITY WINDOW, the central operational fact: valid ONLY on disk. Set by
  PageSetChecksum immediately before flush, verified by PageIsVerified immediately after read.
  "many or even most pages in shared buffers have invalid page checksums". No flag bit says
  whether it is valid -- a deliberate choice to avoid trusting page contents to decide whether to
  verify them. WAL CRC (32-bit) and page checksum (16-bit) are SEPARATE domains; full-page images
  may carry invalid checksums and replay must not test them. The checksum COVERS THE ZERO HOLE,
  which is what PageInit's full-page MemSet buys; FPIs skip the hole and re-zero it on reapply.
- Hint bits + checksums + full_page_writes => MarkBufferDirtyHint must emit a full page image, so
  one bit can cost 8 KB of WAL; during recovery no new WAL can be written, so a standby cannot
  dirty a clean page to set a hint bit. NAMED AND DEFERRED to lesson 15, which owns hint bits.
- PageIsVerified's five header conditions (pd_flags has no undefined bits, lower <= upper <=
  special <= BLCKSZ, pd_special == MAXALIGN(pd_special)) with the honest comment that they
  "don't prove the header is correct"; the all-zero page passes BY DESIGN (extend-then-crash,
  tied back to lesson 01); the all-zeroes test runs AFTER the checksum test so no WARNING fires
  for a legitimately new page; ignore_checksum_failure works only when header_sane; the exact
  log line "page verification failed, calculated checksum %u but expected %u".
- BTPageOpaqueData's five fields (btpo_prev, btpo_next, btpo_level, btpo_flags, btpo_cycleid),
  BTCycleId = uint16, total 16 bytes already MAXALIGNed, reached via BTPageGetOpaque ->
  PageGetSpecialPointer. Taught ONLY as evidence that pd_special points where it says.
  Lessons 28-29 still own nbtree entirely; do not treat the struct as introduced.

VERSION DIVERGENCE, flagged in the lesson and worth carrying: at @3cde90ec (master) bufpage.c
calls DataChecksumsNeedVerify()/DataChecksumsNeedWrite() -- predicates, not a bool -- and wraps
verification in HOLD_INTERRUPTS() because the checksum state can change at runtime; and
storage/page/README names pg_enable_data_checksums()/pg_disable_data_checksums(). The PG 18 docs
describe enable/disable as OFFLINE via pg_checksums with data_checksums read-only, and note that
PG 18 turns checksums ON BY DEFAULT at initdb. The lesson claims the runtime functions exist only
in the tree read, not in any released version. Later lessons touching checksums should re-check.

MEASUREMENTS: again NOT executed -- this run had no PostgreSQL cluster either. Every figure in
lesson 02 (the byte signatures, 8160, 8164, 291, 8176, 24) is arithmetic over constants read this
run, and the document says so in a "Not measured" note above its Takeaways. Part I is now TWO
lessons deep with zero observed numbers. The first lesson that can run a cluster should verify
the fingerprint bytes and the 8160 ceiling first, because everything later in Part I leans on them.

-- LESSON 03 (2026-10-22) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-22-postgres-heap-tuple-header/README.md (single README, not split).
Source commit read: postgres/postgres@f2edf5dd. Lesson 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd --
the tree moved all three times. Keep stating the commit.

*** THIS RUN HAD A CLUSTER. Part I is no longer unmeasured. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
segment_size 1GB, data_checksums OFF, pageinspect 1.12, running as a throwaway cluster in the
sandbox (initdb into /var/tmp, port 5455, unix socket only). A LATER RUN CAN DO THE SAME: psql and
/usr/lib/postgresql/16/bin/* are present in the cloud sandbox image, and pageinspect,
pg_freespacemap, pg_visibility, pgstattuple and amcheck are all in
/usr/share/postgresql/16/extension/. initdb must run as a NON-ROOT user (a 'pg' user was created)
and the data directory must NOT be under the scratchpad path (permission denied); /var/tmp works.
Version gap must be stated: struct and flag constants from master, observations from 16.

TOOK, and must not be re-taught:
- HeapTupleHeaderData BY BYTE OFFSET: 0 t_xmin(4), 4 t_xmax(4), 8 t_field3(4), 12 t_ctid(6),
  18 t_infomask2(2), 20 t_infomask(2), 22 t_hoff(1), 23 t_bits[]. SizeofHeapTupleHeader =
  offsetof(t_bits) = 23 (not sizeof, same reason as SizeOfPageHeaderData in lesson 02).
  MinHeapTupleSize = MAXALIGN(23) = 24. MEASURED: a zero-column table's tuple has lp_len 24,
  t_hoff 24, natts 0, lp_off 8168.
- THE TWO NESTED UNIONS, and the 27-vs-23 resolution: docs Table 66.4 lists t_cid and t_xvac as
  separate 4-byte rows (sum 27) but they are arms of t_field3. Outer union t_choice overlays
  HeapTupleFields with DatumTupleFields {int32 datum_len_, int32 datum_typmod, Oid datum_typeid};
  heap_form_tuple fills the Datum arm for EVERY tuple ("even though this tuple may never become a
  Datum") and the insert path overwrites it. On disk t_choice is always t_heap. t_xvac only via
  pg_upgrade; HeapTupleHeaderGetXvac returns InvalidTransactionId unless HEAP_MOVED.
  Combo CIDs were NAMED AND DEFERRED to lesson 16.
- MEASURED: t_cid observable as 0/1/2 for three inserts in one transaction, same t_xmin.
  heap_page_items names the column t_field3 and comments it "/* shared with xvac */" -- nothing in
  the tuple says which arm you are reading except HEAP_MOVED.
- ItemPointerData fully: 6 bytes, {BlockIdData ip_blkid; OffsetNumber ip_posid}, pg_attribute_packed
  + pg_attribute_aligned(2), with the header's "you'll waste lots of space" warning quoted.
  BlockIdData {bi_hi, bi_lo} exists ONLY so a block number can be SHORTALIGN'd (block.h rationale
  quoted). bi_hi = blkno>>16, bi_lo = blkno & 0xffff. MEASURED on block 299 (0x12b):
  ip_blkid bytes "00 00 2b 01", ip_posid "01 00" -- high half first, each half LE, THE SAME SHAPE
  AS pd_lsn from lesson 02. Lesson 01 deferred ItemPointerData's width; it is now CLOSED.
- t_ctid's FOUR meanings: self (latest), forward pointer (superseded), MovedPartitionsOffsetNumber
  0xfffd + InvalidBlockNumber, SpecTokenOffsetNumber 0xfffe with the token in ip_blkid. Both magics
  exceed MaxOffsetNumber and there is a StaticAssertDecl(MaxOffsetNumber < SpecTokenOffsetNumber).
  THE CHAIN-FOLLOWING RULE, quoted and then measured: validate with xmin(next) == xmax(this),
  because VACUUM may erase the pointed-to tuple first and the slot may be reused.
- MEASURED: insert-then-update pair, lp1 t_ctid (0,2) / xmax 747, lp2 t_ctid self / xmin 747.
  That update happened to be HOT; HOT, redirect line pointers and pruning remain lesson 07's
  entirely and were explicitly deferred.
- t_infomask2: HEAP_NATTS_MASK 0x07FF (11 bits => 2047), "bits 0x1800 are available",
  HEAP_KEYS_UPDATED 0x2000, HEAP_HOT_UPDATED 0x4000, HEAP_ONLY_TUPLE 0x8000, HEAP2_XACT_MASK 0xE000.
  HEAP_TUPLE_HAS_MATCH IS HEAP_ONLY_TUPLE under another name (hash-join overlay, rationale quoted).
  ESTABLISHED AND REUSABLE: the natts mask is NOT the binding constraint on column count --
  t_hoff being uint8 is, per the MaxTupleAttributeNumber header comment ("a little over 1700");
  1664 = 8*208 and 1600 = 8*200 are round numbers so layout changes do not move a user-visible
  limit. MEASURED: "ERROR: tables can have at most 1600 columns" at 1601 columns.
- NATTS IS PER-TUPLE, NOT PER-TABLE, and this was MEASURED: ALTER TABLE ADD COLUMN with no DEFAULT
  does not rewrite, so one page held tuples with natts 2 and natts 3, and the natts-2 tuple reads
  its third column as NULL with HEAP_HASNULL CLEAR and no bitmap at all. pageinspect's
  tuple_data_split_internal comment quoted as the authority. CONSEQUENCE TAUGHT: a NULL has TWO
  on-disk encodings (0 bit in bitmap, or attnum > natts) and heap_attisnull adds a third,
  atthasmissing -- which was NAMED AND DEFERRED to lesson 04.
- t_infomask: ALL SIXTEEN BITS tabulated with their owning lesson (0x0001 HASNULL, 0x0002
  HASVARWIDTH, 0x0004 HASEXTERNAL, 0x0008 HASOID_OLD, 0x0010 XMAX_KEYSHR_LOCK, 0x0020 COMBOCID,
  0x0040 XMAX_EXCL_LOCK, 0x0080 XMAX_LOCK_ONLY, 0x0100 XMIN_COMMITTED, 0x0200 XMIN_INVALID,
  0x0400 XMAX_COMMITTED, 0x0800 XMAX_INVALID, 0x1000 XMAX_IS_MULTI, 0x2000 UPDATED,
  0x4000 MOVED_OFF, 0x8000 MOVED_IN). THE FLAG WORD IS FULL; three bits are held purely for
  pre-9.0 pg_upgrade compatibility. HEAP_XACT_MASK 0xFFF0.
- COMPOSITE CONSTANTS ARE NOT BITS: HEAP_XMIN_FROZEN = COMMITTED|INVALID = 0x0300 ("committed AND
  invalid" is the frozen encoding), HEAP_XMAX_SHR_LOCK = 0x0050, HEAP_MOVED = 0xC000,
  HEAP_LOCK_MASK. Hence GetRawXmin vs GetXmin, and hence heap_tuple_infomask_flags returning TWO
  arrays (raw_flags tests != 0, combined_flags tests (mask & COMBO) == COMBO). Freezing itself was
  DEFERRED to lesson 18; hint bits to 15; MultiXact bits to 17; tuple-lock strengths to 12.
- MEASURED: a fresh insert has t_infomask = 2048 = 0x0800, i.e. HEAP_XMAX_INVALID SET while
  t_xmax = 0 -- the flag is the fast path so visibility need not read the field. With one NULL,
  2049 = HEAP_XMAX_INVALID|HEAP_HASNULL. raw_flags/combined_flags output for the update pair:
  {HASVARWIDTH,XMIN_COMMITTED,HOT_UPDATED} and {HASVARWIDTH,XMAX_INVALID,UPDATED,ONLY_TUPLE}.
- t_hoff AND THE NULL BITMAP, fully taught -- LESSON 04 MUST NOT RE-TEACH THE BITMAP.
  heap_form_tuple's five-line layout computation quoted. BITMAPLEN(NATTS) = (NATTS+7)/8, sized from
  natts NOT from the null count. t_hoff = MAXALIGN(23 + BITMAPLEN(natts)) when hasnull, else
  MAXALIGN(23). ESTABLISHED, MEASURED, REUSABLE STEP FUNCTION: natts 8 -> t_hoff 24 (bitmap FREE,
  it lands in the pad byte that exists anyway); natts 9 -> 32; natts 16 -> 32; natts 72 -> 32
  (bitmap 9 bytes, zero pad); natts 73 -> 40. So the 9th nullable column costs 8 bytes/row forever
  and columns 9-72 cost nothing more in header. lp_len measured at 52/64/92/316/328 respectively.
- BITMAP BIT ORDER AND POLARITY: att_isnull = !(BITS[ATT>>3] & (1 << (ATT & 0x07))) -- LSB-first,
  and 1 MEANS NOT NULL. fill_val's `bitP = &bit[-1]; bitmask = HIGHBIT` is a SENTINEL that forces
  the first iteration down the advance branch, NOT an MSB-first ordering -- taught explicitly
  because it reads like the opposite. bits_to_text prints in att_isnull order.
  MEASURED: 3 int columns with the middle NULL gives t_bits '10100000', lp_len 32 vs 36 for the
  same shape with no nulls -- the NULL costs zero data bytes and the bitmap was free.
  length(t_bits) = BITMAPLEN(natts)*8, so it reveals the bitmap's BYTE count, not the column count.
- palloc0 => pad bytes are zero on disk, tied back to lesson 02's "the checksum covers the hole".
- THE FULL HEXDUMP, measured and decoded field by field (3 ints, no nulls, lp_off 8152, lp_len 36):
  "dc020000 00000000 00000000 00000000 0100 0300 0008 18 00 | 01000000 02000000 03000000"
  => xmin 732, xmax 0, t_field3 0, ctid (0,1), infomask2 3, infomask 0x0800, t_hoff 0x18=24,
  byte 23 = 00 = PURE MAXALIGN PAD (no HASNULL so no bitmap lives there), then three LE int4.
  A later lesson may cite this dump; it does not need re-deriving.
- THE REDUNDANCY THREAD (opened lesson 02) extended as planned: PostgreSQL's record header carries
  NO type information at all, so decoding needs pg_class/pg_attribute -- which is why
  tuple_data_split() takes a relid and calls relation_open(). SQLite's varint record header makes a
  page decodable in isolation. Framed as the forensic consequence: a salvaged page parses to
  t_hoff and no further.
- MinimalTupleData and MINIMAL_TUPLE_OFFSET were mentioned in Further Study ONLY, not taught.
  Lesson 32 still owns the executor's tuple slots.

MEASUREMENTS NOW ON RECORD (first in this track): MinHeapTupleSize 24; the t_hoff step function at
8/9/16/72/73 columns; t_bits polarity; the 1600-column error text; t_cid 0/1/2; the ctid halfword
byte order on block 299; the full 24-byte header hexdump; fresh-insert infomask 0x0800.
STILL NOT MEASURED, and lesson 02 flagged these as the first things to verify once a cluster
exists: the heap page fingerprint bytes "00 20 04 20" at page offsets 16-19, and the 8160-byte
MaxHeapTupleSize insert ceiling. Lesson 03 did NOT get to them and put them in its Next Steps as
item 4. LESSON 04 SHOULD DO THEM -- it will have a cluster and it needs MaxHeapTupleSize anyway
for the TOAST threshold discussion.
Throwaway tables left by this run (drop or reuse, say which): pg3_fixed, pg3_null, pg3_upd,
pg3_alter, pg3_cid, pg3_empty, pg3_blk, w8, w9, w16, w72, w73. Lesson 01's seg_demo and lesson 02's
pg2_fill / pg2_big are from earlier runs' documents, not from a surviving cluster.

SYLLABUS REVISION MADE THIS RUN: lesson 04's title previously read "Attribute storage: MAXALIGN
padding, the null bitmap, 1-byte vs 4-byte varlena headers, and TOAST". Lesson 03 took the null
bitmap completely (BITMAPLEN, the t_hoff arithmetic, bit order, polarity, the measured step
function), so the title now reads "Attribute storage: MAXALIGN padding between attributes, 1-byte
vs 4-byte varlena headers, `atthasmissing`, and TOAST". atthasmissing was added because lesson 03
surfaced it as the third NULL encoding and deferred it. No reordering; no other change.

-- LESSON 04 (2026-10-23) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-23-postgres-attribute-storage-varlena-toast/README.md (single README,
not split -- the two halves were held together by one thread, see below). Source commits read:
master @f52d4051 AND REL_16_STABLE (varatt.h, heaptoast.h) because master has DIVERGED HARD.
Trees so far: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051. Keep stating the commit.

*** THIS RUN HAD A CLUSTER. Same recipe as lesson 03, reproduced without trouble. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
data_checksums OFF, default_toast_compression = pglz, pageinspect 1.12. initdb as user 'pg' into
/var/tmp/pgdata --no-sync, pg_ctl -o '-p 5455 -k /var/tmp -c listen_addresses='. The two traps
from lesson 03 are real and still apply (not root; not under the scratchpad path).

THE UNIFYING THESIS, worth reusing: heap_compute_data_size() is BOTH the layout oracle and the
TOAST loop's measuring stick. It decides where each attribute lands, and every one of the four
TOAST passes is a `while (heap_compute_data_size(...) > maxDataLen)`. Layout and TOAST are one
mechanism consulted from two directions. Later lessons may cite this framing.

LESSON 02'S TWO DEBTS ARE CLOSED, both MEASURED:
- Heap page bytes 12-19 = "1c00 e01f 0020 0420" => pd_lower 28, pd_upper 8160, pd_special 8192,
  pd_pagesize_version 0x2004. BYTES 16-19 = "00 20 04 20" EXACTLY as lesson 02 predicted. nbtree
  leaf = "f0 1f 04 20". Both fingerprints are now observations, not arithmetic. Use freely.
- MaxHeapTupleSize 8160: one `text STORAGE PLAIN` column, repeat('x',n). n<=8132 OK (lp_len 8156
  at n=8128 = 24+4+8128); n=8133..8140 ALL fail with the SAME text "row is too big: size 8168,
  maximum size 8160", because the reported size is MAXALIGN(28+n) and is constant across those
  eight values. 8160 and the MAXALIGN-8 granularity are now measured.

TOOK, and must not be re-taught:
- The data area starts at MAXALIGN(t_hoff), and fill_val() aligns the RAW POINTER not an offset
  ("XXX we use the att_nominal_alignby macro on the pointer value itself ... This is a bit of a
  hack"). That hack is sound ONLY because t_hoff is MAXALIGNed. Consequence taught: the null
  bitmap can never disturb attribute alignment, it only moves the base by a multiple of 8.
- THE FOUR (really six) ALIGNMENT MACROS AND WHY: two axes -- nominal vs conditional, datum vs
  pointer. att_align_nominal / att_nominal_alignby (unconditional), att_align_datum /
  att_datum_alignby (building, skip align if attlen==-1 && VARATT_IS_SHORT),
  att_align_pointer / att_pointer_alignby (walking, skip align if attlen==-1 &&
  VARATT_NOT_PAD_BYTE). The *_alignby variants are master's CompactAttribute fast path.
  att_align_nominal's third use case carries "XXX this should be revisited, probably" about
  unconditionally aligning varlenas inside arrays and multiranges.
- typalign_to_alignby()'s four cases (TYPALIGN_CHAR->1, SHORT->ALIGNOF_SHORT, INT->ALIGNOF_INT,
  DOUBLE->ALIGNOF_DOUBLE) WITH the rationale quoted: codes not real values are stored "mainly so
  that initial catalog contents can be machine-independent".
- att_addlength_pointer's three cases: attlen>0 -> attlen; -1 -> VARSIZE_ANY; -2 -> strlen+1, with
  AssertMacro((attlen) == -2) proving there is no fourth.
- COLUMN ORDER, MEASURED: (int2,int8,int2,int8,int2,int8) lp_len 72 vs (int8,int8,int8,int2,int2,
  int2) lp_len 54 -- 18 pad bytes in a 54-byte payload. Full offset reconstruction is in the
  document. ESTABLISHED AND REUSABLE; do not re-derive.
- THE VARLENA BIT LAYOUT COMMENT from varatt.h, quoted whole (both endian tables). Three things
  extracted and reusable: (1) the flag bits are in the PHYSICALLY FIRST byte in both endiannesses,
  which is why a one-byte peek is portable; (2) a 1-byte length word can never be zero because the
  length includes itself, hence 0x00 is unambiguously pad -- "(We now *require* pad bytes to be
  filled with zero!)"; (3) the length ALWAYS includes the length word.
- Header sizes, all offsetof not literals: VARHDRSZ 4 (in c.h, NOT varatt.h), VARHDRSZ_SHORT 1,
  VARHDRSZ_EXTERNAL 2, VARHDRSZ_COMPRESSED 8, VARATT_SHORT_MAX 0x7F = 127.
- Every VARATT_IS_* reads through `varattrib_1b` (first member a uint8) deliberately, so the
  compiler cannot assume alignment. VARATT_NOT_PAD_BYTE is BYTE-IDENTICAL in both endian arms.
- THE 126-BYTE BOUNDARY, derived and MEASURED. VARATT_CAN_MAKE_SHORT requires VARATT_IS_4B_U (so
  already-short and compressed datums are excluded) and VARSIZE-4+1 <= 127 => data <= 126.
  pg_column_size: 124->125, 125->126, 126->127, 127->131, 128->132.
  Row-width cost of crossing it DEPENDS ON THE OFFSET: on (int,text) lp_len 155->159 (+4); on
  (int2,text) 153->159 (+6 -- 3 for the header plus 2 pad bytes that did not exist).
  CENTRAL FRAMING TAUGHT: a varlena's alignment is a property of its VALUE, not its column.
- THE PAD-BYTE PEEK, fully taught (lesson 03's handover named this lesson's owner of it) with
  att_align_pointer's comment quoted whole, plus the two-line proof: zero => cannot be a 1-byte
  header => pad or already-aligned 4B word => aligning is safe; non-zero => cannot be pad =>
  1-byte header or already-aligned 4B word => not aligning is safe. One comparison resolves four
  possibilities into two safe actions. heaptuple.c's file-header comment quoted for the
  "always zeroed / now required" history.
- THE MEASURED PEEK PAIR, the best measurement of the lesson. pg4_peek3(a int2, b text, c text),
  both rows a=1 b='AB' so byte-identical through offset 28:
    row1 c='CD'           -> t_data \x0100074142074344, lp_len 32. Byte at 29 = 0x07 -> NO align.
    row2 c=repeat('Q',127)-> t_data \x01000741420000000c020000..., lp_len 163. Byte at 29 = 0x00
                             -> align to 32, THREE pad bytes, then 0x0000020c>>2 = 131 = 4+127.
    lp_len 163 = 24+2+3+3+131 exactly.
  A later lesson may cite this pair; it does not need re-deriving.
- STORAGE PLAIN DOES NOT STOP HEADER PACKING -- MEASURED, and it is the ATT_IS_PACKABLE comment's
  own example ("This can be verified with pageinspect"). text column, 'AB', attstorage='p':
  lp_len 27 and t_data \x074142 (1-byte header) for BOTH create-time `STORAGE PLAIN` and
  alter-time `SET STORAGE PLAIN`. A 4-byte header would be lp_len 30. Cause: the relation's
  TupleDesc comes through TupleDescInitEntry(), which copies the TYPE's typstorage
  (TYPSTORAGE_EXTENDED), not the COLUMN's attstorage. The three predicates were quoted:
  ATT_IS_PACKABLE, VARLENA_ATT_IS_PACKABLE, COMPACT_ATTR_IS_PACKABLE.
- attcacheoff AND THE CLIFF, from TupleDescFinalize() (master: computed ONCE there, NOT lazily in
  nocachegetattr as in older trees). Three break conditions = three cache killers: attlen <= 0
  (any varlena or cstring -- EVERYTHING AFTER THE FIRST VARLENA IS UNCACHED, not just that
  column), attgenerated == ATTRIBUTE_GENERATED_VIRTUAL, off > PG_INT16_MAX. Inside the cached
  prefix it uses att_nominal_alignby, correct precisely because the prefix has no varlenas.
  CompactAttribute taught as the companion: 8 bytes, with "Any code changes which enlarge this
  struct should be considered very carefully", holding attcacheoff/attlen/attbyval/attalignby
  (a BYTE COUNT, not the attalign char code) plus four :1 flags incl. attispackable and
  atthasmissing. Also named: firstNonCachedOffsetAttr, firstNonGuaranteedAttr.
- atthasmissing, THE THIRD NULL ENCODING (lesson 03 named and deferred it). heap_attisnull()'s
  first branch quoted: attnum > natts AND atthasmissing => return FALSE, i.e. a column with no
  bytes at all reports NOT NULL. getmissingattr() taught: AttrMissing {am_present, am_value} off
  tupleDesc->constr->missing + (attnum-1); by-value returned directly ("/* no  need to cache
  by-value attributes */" -- double space is in the source); by-reference interned in a
  BACKEND-LIFETIME hash table keyed {len,value} and hashed over the BYTES, so the returned pointer
  is the only value heap_deform_tuple can produce that does not point into a tuple.
  THE asymmetry FLAGGED, not resolved: heap_attisnull gates on atthasmissing, getmissingattr on
  am_present. Lesson 04's Next Step 5 asks whether they can ever disagree.
- MEASURED, atthasmissing: `add column b text default 'hello'` -> filenode UNCHANGED, natts 1,
  lp_len 28, t_data \x01000000 (four bytes, the int, nothing else), t_infomask 2048 =
  HEAP_XMAX_INVALID only with HEAP_HASNULL CLEAR, atthasmissing t, attmissingval {hello}, and
  SELECT returns 'hello'. Contrast cases also measured: a VOLATILE default
  (clock_timestamp()::text) CHANGED the filenode and reset atthasmissing to FALSE FOR EVERY
  COLUMN (the rewrite undoes the optimisation); `add column b text` with no default leaves
  atthasmissing false, attmissingval null, natts 1 -- i.e. lesson 03's encoding #2 unchanged.
- ALL FOUR TOAST CONSTANTS DERIVED IN CLOSED FORM FROM MaximumBytesPerTuple(n) =
  MAXALIGN_DOWN((BLCKSZ - MAXALIGN(SizeOfPageHeaderData + n*sizeof(ItemIdData))) / n), using only
  lesson 01-03 constants, AND THEN MEASURED:
    TOAST_TUPLES_PER_PAGE 4;  TOAST_TUPLE_THRESHOLD = TOAST_TUPLE_TARGET = 2032
    TOAST_TUPLES_PER_PAGE_MAIN 1; TOAST_TUPLE_TARGET_MAIN = 8160 == MaxHeapTupleSize (NOT a
      coincidence: "the largest tuple that will fit on a heap page" describes both)
    EXTERN_TUPLES_PER_PAGE 4; EXTERN_TUPLE_MAX_SIZE 2032
    TOAST_MAX_CHUNK_SIZE = 2032 - MAXALIGN(23) - sizeof(Oid) - sizeof(int32) - VARHDRSZ
                         = 2032 - 24 - 4 - 4 - 4 = 1996
  MEASURED: the threshold boundary is t_len > 2032 STRICTLY -- datalen 2004 (t_len 2032) stays
  inline with 0 chunks, datalen 2005 (t_len 2033) produces 2 chunks. Chunk lengths for
  repeat('y',10000): 1996 x5 + 20 = 10000. Chunk TUPLES: lp_len 2032 each, t_hoff 24, exactly
  FOUR on page 0 -- i.e. EXTERN_TUPLE_MAX_SIZE and EXTERN_TUPLES_PER_PAGE both observed.
  Chunk tuple reconstruction: 24 + 4 (chunk_id Oid) + 4 (chunk_seq int4) + 4 (varlena hdr) + 1996.
  The header's two warnings quoted: the needs_toast_table()/LOBLKSIZE XXX, and "NB: Changing
  TOAST_MAX_CHUNK_SIZE requires an initdb."
- THE FOUR-PASS ALGORITHM, with heap_toast_insert_or_update()'s own enumeration comment quoted
  verbatim, plus the hoff/maxDataLen computation ("this should match heap_form_tuple()") and
  RelationGetToastTupleTarget(rel, TOAST_TUPLE_TARGET) - hoff. Taught: each pass is a while loop
  re-calling heap_compute_data_size; selection is STRICTLY GREEDY biggest-first, one per
  iteration; per-pass filters are EXTENDED (compress, pass 1), EXTENDED|EXTERNAL (externalise,
  pass 2), MAIN (compress, pass 3), MAIN (externalise, pass 4); pass 1's "one long field and
  several short ones" shortcut quoted; and PASS 4 REASSIGNS maxDataLen = TOAST_TUPLE_TARGET_MAIN -
  hoff, i.e. 2008 bytes of budget for passes 1-3 and 8136 for pass 4. That single reassignment IS
  what MAIN means.
- toast_tuple_find_biggest_attribute()'s FLOOR: biggest_size STARTS at MAXALIGN(pointer size) =
  MAXALIGN(18) = 24, not zero. An attribute under 24 bytes is NEVER a candidate, because a
  pointer would not shrink the tuple.
- varatt_external FULLY DECODED (REL_16_STABLE spelling; see divergence). 16 bytes no padding
  "because we sometimes compare these pointers using memcmp", stored UNALIGNED so callers must
  memcpy into a local first. Datum = VARHDRSZ_EXTERNAL(2) + 16 = 18. VARTAG_ONDISK = 18 AND THE
  TAG EQUALLING THE SIZE IS ARCHAEOLOGY, not arithmetic -- the enum comment says the "peculiar
  value" comes from "a previous notion that the tag field was the pointer datum's length".
  VARLENA_EXTSIZE_BITS 30 / MASK 0x3FFFFFFF is where the documented 1 GB limit comes from, with
  2 bits left for ToastCompressionId (PGLZ 0, LZ4 1, INVALID 2) and the header's blunt "we can
  never have more than 4 values in this enum".
  "IS IT COMPRESSED" IS NOT A FLAG -- VARATT_EXTERNAL_IS_COMPRESSED is a SIZE COMPARISON,
  extsize < rawsize - VARHDRSZ, with the comment quoted. So the method bits are meaningless when
  the sizes are equal.
- MEASURED, the 18-byte pointer byte by byte. repeat('y',10000) in a STORAGE EXTERNAL column:
  lp_len 42 = 24 + 18 (no alignment, 1-byte header class); t_infomask 2054 = 0x0806 =
  XMAX_INVALID|HASEXTERNAL|HASVARWIDTH -- lesson 03 catalogued HEAP_HASEXTERNAL 0x0004 and
  deferred it, now OBSERVED. t_data \x01121427000010270000814000007f400000 =>
  01 (va_header, the LE "00000001" TOAST-pointer pattern) | 12 (va_tag = 18 = VARTAG_ONDISK) |
  14270000 (va_rawsize 10004 = 10000+4) | 10270000 (va_extinfo 10000; NOT compressed because
  10000 == 10004-4) | 81400000 (va_valueid 16513) | 7f400000 (va_toastrelid 16511).
  BOTH OIDS CROSS-CHECKED against pg_class.reltoastrelid and the toast table's chunk_id this run.
  TRAP MEASURED: pg_column_size() on that column returns 10000, NOT 18 -- it reports the external
  datum's size, not the in-tuple footprint. For footprint, read lp_len.
- THE FOUR STORAGE STRATEGIES, MEASURED SIDE BY SIDE on one table, one value repeat('ab',2500):
  ext (e) pg_column_size 5000, externalised uncompressed, 3 chunks 1996/1996/1008;
  extd (x) 66, COMPRESSED THEN EXTERNALISED, 1 chunk of 66;
  main (m) 70, compressed INLINE;
  plainc (p) 5004, untouched inline with a 4-byte header.
  THE 66-vs-70 EXPLANATION, established and reusable: the pglz payload is 62 bytes in both.
  Inline a compressed datum carries VARHDRSZ_COMPRESSED = 8 (length word + va_tcinfo): 8+62 = 70.
  Externally the length word is redundant (the pointer's va_rawsize/va_extinfo carry it), so the
  chunk stores va_tcinfo + payload: 4+62 = 66.
  THE WHOLE TUPLE RECONSTRUCTED BYTE-EXACTLY: measured lp_len 5136 = 24 + 18 (ext ptr @24) +
  18 (extd ptr @42) + 70 (main compressed @60) + 2 PAD (@130-131) + 5004 (plainc @132).
  The 2 pad bytes were verified by substring: tuple offsets 126-137 = "ff 0f 02 42 00 00 30 4e
  00 00 61 62", so 130-131 = 00 00 and the word at 132 is 0x00004e30>>2 = 5004 with low bits 00.
  main's word at 60 is 0x0000011a, low bits 10 => compressed, >>2 = 70; its va_tcinfo = 0x1388 =
  5000 = decompressed size, method 0; then the pglz stream "fc 61 62 61 0f 02 ff ...".
- PASS 4 MEASURED, and it is the best MAIN demonstration available: two MAIN columns of 5120
  INCOMPRESSIBLE bytes each (concatenated distinct md5s) -> lp_len 5168, HASEXTERNAL set, 3 chunks
  totalling 5120, pg_column_size 5120 / 5124 -> EXACTLY ONE of the two was externalised, because
  after that the data size 18+2+5124 = 5144 <= 8136 and pass 4 stopped. 5168 = 24+18+2+5124.
  THE SAME DATA with default EXTENDED: lp_len 60 = 24+18+18, 6 chunks, 10240 bytes.
  So: same bytes, one attstorage apart => a 5168-byte row with 3 chunks, or a 60-byte row with 6.
- toast_tuple_target IS A TARGET, NOT A THRESHOLD -- MEASURED, and the single best operational
  trap in the lesson. WITH (toast_tuple_target = 512): datalen 2004 stays at lp_len 2032, FOUR
  TIMES the configured target, because the gate into the toaster is `t_len >
  TOAST_TUPLE_THRESHOLD` (a compile-time 2032) and the reloption only reaches maxDataLen INSIDE
  the function. datalen 2005 drops to lp_len 42. Lowering the target below 2032 does NOTHING;
  raising it above 2032 works as expected.

HONEST NON-RESULT, recorded deliberately: the attcacheoff cliff was NOT reproduced as a
measurable cost. Two 400k-row tables, ten int columns and one text, text first vs text last,
warm cache, `select sum(c10)`: late 23.9/18.0/18.3 ms, early 22.6/21.4/19.4 ms -- overlapping
distributions. At ten columns the uncached walk is nine macro steps per row and is lost in
per-row executor overhead. The lesson says so explicitly and tells the reader NOT to cite the
cliff as a measured cost. Lesson 04's Next Step 2 specifies the 200-column redo with
EXPLAIN (ANALYZE, TIMING OFF). A later lesson that wants this number must earn it.

VERSION DIVERGENCE, the largest this track has hit -- a full table is in the lesson. Summary:
master @f52d4051 renamed varatt_external -> varatt_external_oid and added varatt_external_oid8
(20 bytes, va_valueid_lo/_hi) for 64-bit TOAST value IDs; VARTAG_ONDISK -> VARTAG_ONDISK_OID (18)
with a new VARTAG_ONDISK_OID8 (4); VARATT_EXTERNAL_GET_EXTSIZE/GET_COMPRESS_METHOD/IS_COMPRESSED
-> VARATT_EXTINFO_* taking a bare uint32; TOAST_MAX_CHUNK_SIZE is no longer a scalar but is
parameterised by the TOAST table's chunk-id type (TOAST_OID_MAX_CHUNK_SIZE /
TOAST_OID8_MAX_CHUNK_SIZE); every externally visible accessor became a static inline function
("All of these were originally macros, accounting for the upper-case naming"); the `slow` flag in
heap_deform_tuple/nocachegetattr is GONE, replaced by firstNonCachedOffsetAttr and
first_null_attr(); attcacheoff moved from lazy fill-in to TupleDescFinalize(); TOAST_INDEX_HACK
lives in access/common/indextuple.c, not heaptoast.*. IDENTICAL in both trees: the bit layouts,
VARHDRSZ_*, VARATT_SHORT_MAX, VARLENA_EXTSIZE_BITS, the endian internals, the four-pass algorithm
and the MaximumBytesPerTuple derivation. EVERY MEASURED NUMBER IS PG 16.

NO SYLLABUS REVISION THIS RUN. Lesson 03 had already retitled 04 to drop the null bitmap and add
atthasmissing; that title was taught as written, in full, and nothing was left over.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg4_fp, pg4_ceil, pg4_bad, pg4_good, pg4_var, pg4_var2, pg4_peek, pg4_peek2,
pg4_peek3, pg4_plain, pg4_plain2, pg4_miss, pg4_miss2, pg4_toast, pg4_t2, pg4_strat, pg4_main,
pg4_main2, pg4_ext2, pg4_ttt, pg4_late, pg4_early. Keep the pgNN_ prefix convention.

-- HANDOVER TO LESSON 05 -----------------------------------------------------------------------
Scope for lesson 05, "The Free Space Map: the FSM tree pages and how a page with room is found".
Lesson 04 answered "how big is this tuple"; lesson 05 answers "which page does a tuple of that
size go to". That is the handoff, and it is clean.
Take: src/backend/storage/freespace/README (THE primary source, not yet read in this track);
freespace.c and fsm_internals.h; the FSM's THREE-LEVEL tree of pages and the binary max-heap
INSIDE each page (SlotsPerFSMPage, FSM_TREE_DEPTH, FSM_ROOT_LEVEL, FSM_BOTTOM_LEVEL,
FSM_CATEGORIES and the 32-byte granularity -- a category is one byte, so free space is tracked to
BLCKSZ/256 = 32 bytes, NOT exactly); fsm_space_avail_to_cat / fsm_space_cat_to_avail and the
deliberate ROUNDING DOWN; GetPageWithFreeSpace / RecordAndGetPageWithFreeSpace /
RecordPageWithFreeSpace; the FSM_LOCAL_MAP / local in-memory map for relations too small to have
an FSM fork at all (HEAP_FSM_CREATION_THRESHOLD = 4 pages, and WHY -- an FSM fork for a 1-page
table costs 3 pages); the fact that THE FSM IS NOT WAL-LOGGED and is therefore allowed to be
wrong, with the "it's just a hint" rationale and how VACUUM repairs it; fsm_page_contents() from
pageinspect as the observation tool; and the connection to lesson 02's FOUR free-space functions,
especially PageGetHeapFreeSpace returning 0 at the 291-tuple ceiling -- that is the number the FSM
is NOT aware of, which is a real trap worth measuring.
MEASURE IT. The cluster recipe works and is in this block. Good candidates: hexdump an FSM page
and decode the max-heap by hand; show the 32-byte category granularity by inserting tuples whose
sizes differ by less than 32 bytes and watching the FSM not notice; demonstrate the
HEAP_FSM_CREATION_THRESHOLD by checking pg_relation_size(rel,'fsm') as a table grows past 4 pages;
and find a case where the FSM hands back a page that PageGetHeapFreeSpace says is full.
Lesson 04's Next Step 4 tells the reader to read toast_fetch_datum() before lesson 05 -- lesson 05
does NOT own the TOAST read path and should not take it, but may reference it.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, the 15-bit cap, the four
free-space functions, checksums (02); the tuple header's fields, t_ctid, the infomask catalogues,
natts-per-tuple, BITMAPLEN, t_hoff arithmetic, the null bitmap's order or polarity (03); the
alignment macro family, typalign_to_alignby, the varlena bit layouts, the 126-byte boundary, the
pad-byte peek, attcacheoff/CompactAttribute, atthasmissing, or ANY of TOAST -- thresholds, chunk
size, the four passes, varatt_external, the four storage strategies (04).
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 2032, 1996, 8136, 126, 18, and
the heap page fingerprint "00 20 04 20" at page offsets 16-19 (now MEASURED, lesson 04).
Do NOT take: the Visibility Map and PD_ALL_VISIBLE (06, which still owns them entirely); HOT,
pruning, LP_REDIRECT, pd_prune_xid (07); smgr's fsync queue, relation extension locking,
mdzeroextend/FileFallocate, fd.c's virtual FDs (08); the buffer manager (09); visibility rules
(14); hint bits (15); combo CIDs (16); MultiXacts (17); freezing (18); VACUUM's two heap passes
and the dead-TID store (19).

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 01's note was dropped and TRACK-ARCHIVE.md gained the
row "PG 01" pointing at commit 2fd269184b17e59f9091b82cb8934672fd32c8f7 / blob
1feb2d21947ee646bf23f350c8ac84424fe87251, which is the last state of TRACK.md carrying it. This
block now holds 02, 03 and 04 -- exactly the three the policy allows. WHEN LESSON 05 SHIPS IT MUST
DROP THE LESSON 02 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap and pg_visibility cover lessons 05 and 06.
  Lesson 01's hands-on installs all three; lesson 02's hands-on needs only pageinspect.
- CONFIRMED 2026-10-22 AND AGAIN 2026-10-23: the cloud sandbox CAN run a cluster. psql, pg_config
  and /usr/lib/postgresql/16/bin/{initdb,postgres,pg_ctl,...} are in the image (PostgreSQL 16.15
  on Ubuntu 24.04), and pageinspect, pg_freespacemap, pg_visibility, pgstattuple and amcheck are
  all present in /usr/share/postgresql/16/extension/. Recipe that worked: create a non-root user,
  initdb -D /var/tmp/pgdata --no-sync as that user, pg_ctl with -o '-p 5455 -k /var/tmp'. TWO
  TRAPS: initdb refuses to run as root, and it refuses a data directory under the scratchpad path
  (permission denied) -- /var/tmp works. Every lesson from here on SHOULD measure; say so when it
  cannot. The cluster is per-run and does not persist, so table names from earlier runs are gone.
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04 — the full list is in the
  lesson 04 note above). Keep the pgNN_ prefix convention.
- pg_buffercache is needed from lesson 09; pgstattuple for lessons 07 and 19; amcheck for
  lessons 28-29 (it is the closest thing PostgreSQL has to SQLite's integrity_check, and the
  comparison is worth making explicitly when lesson 28 arrives).
-->

## Completed Subjects

### SQLite Internals — 33 lessons, 2026-09-16 to 2026-10-18 (capstone 2026-10-19)

Sequential deep dive through `btree.c`, `pager.c`, `wal.c`, the compiler and the VDBE, in five
parts: the on-disk format and the b-tree layer (01–13), the pager, journals and VFS (14–18),
write-ahead logging (19–20), the SQL compiler and bytecode engine (21–28), and operational and
advanced topics (29–33). Capstone:
[`2026/10/2026-10-19-sqlite-capstone/README.md`](2026/10/2026-10-19-sqlite-capstone/README.md).

The full 33-item syllabus, with every lesson title as taught, is preserved in this file's git
history — the last commit carrying it is the one recorded in
[`TRACK-ARCHIVE.md`](TRACK-ARCHIVE.md) under row "31 – 33". The lesson documents themselves are
indexed in the root [`README.md`](README.md), which is the real record of what was taught.

**Closing summary.** One file, one writer, a bytecode interpreter in the middle, tests instead of
types. The commit point is a filesystem operation (`unlink("-journal")` in rollback mode, a frame
header with a nonzero commit counter in WAL mode). The planner's cost channel is 10 bits wide.
And `quick_check` is a *weaker* check at nearly the same I/O as `integrity_check`, blind to index
key order — the single most useful operational fact the track produced.
