<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 10
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 10 shipped 2026-10-29.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 11**.

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
- [x] 05 — The Free Space Map: the FSM tree pages and how a page with room is found
- [x] 06 — The Visibility Map: `PD_ALL_VISIBLE`, all-frozen, and the index-only scan it enables
- [x] 07 — HOT updates and page pruning: redirect line pointers, `heap_page_prune()`, and the HOT chain
- [x] 08 — `smgr` and `md.c`: relation extension, the segment map, and the fsync request queue

### Part II — Shared memory: buffers and locks (`storage/buffer`, `storage/lmgr`)

- [x] 09 — The buffer manager: `BufferDesc`, the clock-sweep victim search, and the strategy ring
- [x] 10 — Buffer pins versus content locks, and the LWLock implementation
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

-- LESSON 08 (2026-10-27) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-27-postgres-smgr-md-segments-fsync-queue/README.md (single README, not
split -- the segment map, the extension batching and the fsync queue are three sections of one
vertical slice, and splitting would have hidden that they are the same layer).

SOURCE PROVENANCE: `apt-get source postgresql-16` again, version 16.15-0ubuntu0.24.04.1, the same
build the cluster runs. LESSON 07'S ROUTE WORKS AND IS NOW THE DEFAULT -- see the environment notes
below for the exact recipe (deb-src on noble, then apt-get source into a world-readable dir).
PATCH CHECK, AND IT MATTERS MORE THIS RUN THAN LAST: there are 15 patches in debian/patches/ now,
and ONE OF THEM TOUCHES bufmgr.c (v8-0001-Fix-recovery-conflict-SIGUSR1-handling.patch). Its only
bufmgr.c hunk is at LockBufferForCleanup(), nowhere near ExtendBufferedRel*, so the extension code
quoted is upstream verbatim -- but DO NOT repeat lesson 07's blanket "no patch touches anything we
read"; extract the +++/--- paths and check per file. md.c, smgr.c, sync.c, hio.c, checkpointer.c
and fd.c are untouched. Trees/releases so far: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd,
04 @f52d4051, 05 @126976a8, 06 @0019a1ec, 07 = release 16.15, 08 = release 16.15.
Files read IN FULL: storage/smgr/README (52 lines), include/storage/smgr.h, include/storage/sync.h.
Read substantially: storage/smgr/md.c (1649 lines -- every function named in the lesson),
storage/sync/sync.c (624 lines). Read in part: postmaster/checkpointer.c (ForwardSyncRequest,
CompactCheckpointerRequestQueue, the CheckpointerShmemStruct declaration, CheckpointerShmemSize),
access/heap/hio.c (RelationAddBlocks), storage/buffer/bufmgr.c (ExtendBufferedRelBy/To/Common/
Shared, LimitAdditionalPins), common/file_utils.c (pg_pwrite_zeros), utils/misc/guc_tables.c
(the file_extend_method entry), include/storage/fd.h (enum FileExtendMethod),
include/port/pg_iovec.h (PG_IOV_MAX), catalog/storage.c (AddPendingSync, smgrDoPendingSyncs),
access/nbtree/nbtsort.c (the smgrimmedsync call site).

*** THIS RUN HAD A CLUSTER, EVERY number is measured, AND THE PRIMARY INSTRUMENT WAS strace, NOT
SQL. *** PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, Linux 6.18, ext4,
2 vCPU / 8 GB RAM, ~30 GB free. block_size 8192, segment_size 131072 blocks, wal_segment_size
16 MB, wal_level replica, max_wal_size 1 GB, checkpoint_timeout 300 s, fsync on, data_sync_retry
off, max_files_per_process 1000, file_extend_method posix_fallocate, shared_buffers 128 MB except
where a measurement says otherwise, data_checksums OFF.
*** A DIFFERENT CLUSTER RECIPE WORKED THIS RUN AND IS SIMPLER: `apt-get install postgresql
postgresql-contrib` then `pg_ctlcluster 16 main start`, running psql as the postgres user via
`su postgres -c`. No initdb, no /var/tmp dance, ALTER SYSTEM + pg_ctlcluster restart works for
PGC_POSTMASTER settings, and the log is at /var/log/postgresql/postgresql-16-main.log which is
where log_checkpoints and DEBUG1 output land. Both recipes work; this one is fewer steps. The
world-readable-script trap still applies: put psql -f scripts in /tmp/<dir> with chmod 777. ***

TOOK, and must not be re-taught:
- THE SWITCH: smgrsw[] has ONE element, 16 md* function pointers, and the README's own defence of
  the indirection quoted ("Removing the switch layer would save nothing noticeable anyway, since
  storage-access operations are surely far more expensive than one extra layer of C function
  calls"). smgropen() does NO I/O -- hash entry only, keyed on RelFileLocatorBackend.
- THE SEGMENT MAP, structurally: SMgrRelationData's md_num_open_segs[fork] + md_seg_fds[fork] ->
  MdfdVec { File mdfd_vfd; BlockNumber mdfd_segno; } -> fd.c VFD -> file. _fdvec_resize() NEVER
  SHRINKS, deliberately, "because we want mdtruncate() to be able to promise that it won't allocate
  memory, so that it is allowed in a critical section". _mdfd_openseg asserts segments open in
  strict order -- NO SPARSE MAP. md_num_open_segs IS A CACHE, NOT A CENSUS (header comment quoted).
- *** THE 1 GB SEGMENT BOUNDARY, CROSSED AND MEASURED -- lesson 01's arithmetic is now an
  observation, which is what its handover asked for. *** 1,000,000 rows x 1000-byte pad ->
  base/16388/16438, 142,858 blocks, 1,170,292,736 bytes. On disk: segment 0 = EXACTLY
  1,073,741,824 bytes = 131,072 blocks = RELSEG_SIZE; `.1` = 96,550,912 = 11,786 blocks;
  `_fsm` = 311,296 (38 blocks); no `_vm` until the first VACUUM. 131072 + 11786 = 142858 exactly.
  SIDE DETAIL: stat reports segment 0 as 2,097,160 x 512 B = 1,073,745,920 allocated, 4,096 bytes
  MORE than apparent -- ext4 extent metadata, not PostgreSQL.
- *** mdnblocks() IS TWO SYSCALLS AND THE TRACE SHOWS THE LOOP CONDITION:
    openat("base/16388/16438") = 14 ; lseek(14,0,SEEK_END) = 1073741824   <- == RELSEG_SIZE, advance
    openat("base/16388/16438.1") = 15 ; lseek(15,0,SEEK_END) = 96550912   <- < RELSEG_SIZE, return
  1.45 ms for the pair. _mdfd_openseg deliberately passes NO O_CREAT here ("might create a segment
  which has vanished through some operating system misadventure"). ***
- *** RELATION SIZE IS A SYSCALL, EVERY TIME, OUTSIDE RECOVERY -- MEASURED: the same backend did
  lseek(15,0,SEEK_END) again 3 s later for the same unchanged relation. smgrnblocks_cached()
  returns InvalidBlockNumber when not InRecovery, reason quoted from smgr.h ("lack of a shared
  invalidation mechanism for changes in file size"). THIS IS THE REMOVED-REDUNDANCY THREAD'S
  STRUCTURAL LIMIT: PostgreSQL stores a relation's size NOWHERE, where SQLite keeps a page count
  in the header at offset 28 and validates it against the change counter. ***
- *** RELATION EXTENSION, THE LESSON'S BEST EXPERIMENT: THE SAME 20,000 ROWS, FOUR SYSCALL SHAPES.
    (1) INSERT..SELECT (no bistate):        2,860 pwritev, iovcnt=1, 0 fallocate; 2,858 blocks
    (2) COPY (bistate):                     48 fallocate, 1 pwritev;              2,896 blocks
    (3) COPY, file_extend_method=write_zeros: 0 fallocate, 97 pwritev;            2,896 blocks
    (4) COPY, shared_buffers=128kB:         2,861 pwritev, iovcnt=1;              2,858 blocks
  (2)'s fallocate lengths IN ORDER: 81920, 81920, 163840, 327680, then 524288 x 44 = 10, 10, 20,
  40, 64, 64, ... That is hio.c arithmetic exactly: extend_by = num_pages*(1+waitcount), then
  Max(.., bistate->already_extended_by) which ACCUMULATES (0->10->20->40->80), then
  Min(.., MAX_BUFFERS_TO_EXTEND_BY = 64). 48 syscalls instead of 2,860 = 60x. ***
- *** THE CLAMP NOBODY EXPECTS, and the most operationally surprising thing in the lesson:
  LimitAdditionalPins() computes NBuffers / (MaxBackends + NUM_AUXILIARY_PROCS), subtracts
  PrivateRefCountOverflowed + REFCOUNT_ARRAY_ENTRIES, floors at 1. At shared_buffers=128kB
  (16 buffers) with max_connections=100 the quotient is 0, so EVERY extension went back to ONE
  BLOCK -- measured, row (4). EXTENSION BATCHING IS A FUNCTION OF shared_buffers /
  max_connections, documented nowhere a DBA would look. ***
- THE COST OF BATCHING, measured: 2,896 vs 2,858 blocks = +38 blocks = +1.3% permanent slack for
  the same 20,000 rows. Bulk extension over-extends by design (not_in_fsm_pages, then
  FreeSpaceMapVacuumRange, or bistate->next_free).
- file_extend_method IS A REAL GUC IN THIS BUILD (PGC_SIGHUP, RESOURCES_DISK, default
  posix_fallocate, alternative write_zeros, the fallocate option compiled in only
  #ifdef HAVE_POSIX_FALLOCATE). *** IT CHANGES THE SYSCALL, NOT THE CLOCK -- MEASURED: five COPY
  runs of a 97 MB CSV each way, medians 638 ms (fallocate) vs 670 ms (write_zeros), no difference
  outside noise. What DOES move: backend write_bytes in /proc/<pid>/io, 197,615,616 vs 219,070,464
  for a 117,047,296-byte relation (+21.5 MB, +10.9%). HONEST LIMIT STATED IN THE LESSON:
  write_bytes cannot separate WAL from heap writeback, and the nominal 117 MB of zero writes mostly
  never reach the device because the real data overwrites the same page-cache pages. A cold-cache
  or blktrace run would be needed for a precise figure. ***
- pg_pwrite_zeros / FileZero: PG_IOV_MAX = Min(IOV_MAX, 32), and EVERY iovec points at the SAME
  `static const PGIOAlignedBlock zbuffer` of BLCKSZ. MEASURED: a 40-block extension splits 32 + 8,
  a 64-block extension splits 32 + 32.
- *** pg_stat_io SEES BLOCKS, NOT SYSCALLS -- worth remembering before trusting it for I/O shape.
  MEASURED after pg_stat_reset_shared('io'): client backend / relation / bulkwrite extends = 14288,
  exactly the relation's block count, writes = 12240, fsyncs NULL; client backend / relation /
  normal extends = 6 (the FSM fork, extended without a strategy). Those 14,288 blocks were ~224
  fallocate calls. IOOP_EXTEND is counted with extend_by as its count argument. ***
- *** skipFsync IS A TRANSFER OF OBLIGATION, NOT A WAIVER, and the proof is a measurement that
  looks like a bug: four plain `CREATE TABLE q (id int, pad text)` produced FOUR fsync() syscalls
  in the CLIENT BACKEND, on relfilenodes that pg_class identifies as pg_toast_*_index. nbtsort.c
  builds btrees OUTSIDE shared buffers with smgrextend(..., skipFsync=true) and pays the debt in
  _bt_leafbuild's `if (wstate->btws_use_wal) smgrimmedsync(...)`, comment quoted. THE TELL IN THE
  TRACE is openat("<relfilenode>.1") = -1 ENOENT immediately before the fsync -- that is
  mdimmedsync's deliberate probe past the last active segment, which mdnblocks would never do. ***
  NOT taken here: smgrDoPendingSyncs / wal_skip_threshold is DEAD CODE at wal_level=replica
  because AddPendingSync is guarded by `relpersistence == PERMANENT && !XLogIsNeeded()`. Verified
  by reading the guard, not measured. A LESSON THAT RUNS WITH wal_level=minimal CAN TAKE IT.
- THE FSYNC REQUEST QUEUE, the part nobody reads, now read: FileTag { int16 handler; int16 forknum;
  RelFileLocator rlocator; uint32 segno; } is the WHOLE design. Constants that are code, not
  folklore: max_requests = Min(NBuffers, MAX_CHECKPOINT_REQUESTS=10000000) -> 16,384 at
  shared_buffers=128MB and 16 at 128kB (DERIVED from checkpointer.c:912, nothing exposes it);
  FSYNCS_PER_ABSORB = UNLINKS_PER_ABSORB = 10; too_full = (num_requests >= max_requests/2) with the
  SetLatch deliberately AFTER LWLockRelease. FOUR request types (SYNC/UNLINK/FORGET/FILTER) and SIX
  handlers (MD, CLOG, COMMIT_TS, MULTIXACT_OFFSET, MULTIXACT_MEMBER, NONE) share one queue.
  retryOnError DIFFERS BY CALLER and the difference is a durability argument: register_dirty_segment
  passes FALSE (fall back to a local fsync), register_unlink_segment and register_forget_request
  pass TRUE (spin on WaitLatch 10 ms forever), because dropping an unlink leaks a file.
- *** THE OBSERVATION TECHNIQUE WORTH REUSING: log_min_messages = debug1 MAKES
  ProcessSyncRequests() PRINT ONE LINE PER pendingOps ENTRY. That is a direct dump of the
  checkpointer's hash table, with real paths. MEASURED after CHECKPOINT (drain), then two UPDATEs
  by ctid '(0,1)' and '(131072,1)' on the two-segment relation, then CHECKPOINT:
    number=1 file=base/16388/16438   time=0.421 ms
    number=2 file=base/16388/16438.1 time=0.125 ms
    number=3 file=pg_xact/0000       time=0.138 ms
  "wrote 4 buffers ... sync files=3". THREE FACTS: segno is part of the key so ONE RELATION COSTS
  TWO ENTRIES; pg_xact rides the same queue via SYNC_HANDLER_CLOG; and `sync files=N` in
  checkpoint complete IS ckpt_sync_rels, i.e. literally the number of hash entries processed. ***
- *** THE DEDUP RATIO, MEASURED: ~40,800 block writes (buffers_backend 24,490 + buffers_clean 100 +
  3 + the checkpoint's own 16,214) collapsed into sync files=21, longest=0.083 s. The 22,500-block
  heap written and rewritten three times is ONE entry fsynced once in 59.153 ms; its `_fsm` fork is
  a SEPARATE entry (forknum is in the key -- this is the mechanical justification for "forks grow
  and truncate independently", which lessons 05/06 asserted from the README). ~1,940:1. ***
- buffers_backend_fsync STAYED 0 IN EVERY CONFIGURATION, including 57,987 backend writes against a
  16-SLOT queue. That is the design working, not a weak test: CompactCheckpointerRequestQueue
  hashes {type, ftag} and keeps only the last occurrence, so a queue of near-duplicates always
  compacts and ForwardSyncRequest never returns false. THE FAILURE NEEDS A QUEUE BOTH FULL AND
  DUPLICATE-FREE -- many distinct hot segments, not many writes.
- *** AN EXPERIMENT THAT FAILED, REPORTED AS A FAILURE IN THE DOCUMENT: SIGSTOPping the
  checkpointer to force queue overflow DEADLOCKED -- the inserting backend hit max_wal_size and
  waited for a checkpoint that could not start. Abandoned, checkpointer resumed. THE QUEUE-FULL
  FALLBACK IN register_dirty_segment() WAS READ AND NEVER EXECUTED. DO NOT REPEAT THIS METHOD. ***
- *** THE INACTIVE ZERO-LENGTH SEGMENT, OBSERVED -- lesson 01 derived it, nothing had seen it.
  DELETE FROM seg WHERE ctid >= '(100000,1)' then VACUUM: "truncated 142858 to 100000 pages".
  After: segment 0 = 819,200,000 B (= 100,000 x 8192), `.1` = 0 BYTES AND STILL PRESENT,
  `_fsm` 311,296 -> 221,184, `_vm` created at 32,768 (4 blocks). mdtruncate's reason quoted ("other
  backends and/or the checkpointer might be holding open file references"), plus
  register_dirty_segment being called ON A TRUNCATION (the queue carries "this file got shorter"
  with the same FileTag), and Assert(v != &md_seg_fds[fork][0]) -- "we never drop the 1st segment".
  AND THE ZERO-LENGTH SEGMENT IS INVISIBLE ABOVE: a fresh backend scanning the table did
  openat("base/16388/16438") + lseek = 819200000 and made ZERO references to `.1` in the whole
  trace. NOT REACHED: truncating to an exact multiple of RELSEG_SIZE, which the comment says keeps
  segment K+1 at length 0. ***
- ANSWERS TO THE THREE INHERITED QUESTIONS, recorded so nobody re-asks them:
  (1) LESSON 06'S "why does bulk extension overshoot": RelationAddBlocks's waitcount multiplier and
      the accumulating bistate->already_extended_by, capped at 64. Measured ramp above.
  (2) LESSON 07'S "N cold updates produce exactly N new blocks": consistent with extend_by = 1 on
      the no-bistate path -- heap_update's RelationGetBufferForTuple has no BulkInsertState, so each
      cold update that cannot fit extends by exactly one block. The FSM question (can a cold update
      be routed back onto a prune-recovered page) is STILL OPEN; the answer needs
      FreeSpaceMapVacuumRange, which is lesson 19's.
  (3) LESSON 05'S "is the FSM's 3-block floor a consequence of extension": NO. It is
      fsm_logical_to_physical's interleaving (the bottom page cannot be block 0), which lesson 05
      already derived. smgrextend has nothing to do with it. CLOSED.
- WAL AND EXTENSION: not separately measured this run, but stated with evidence from the code --
  mdextend/mdzeroextend emit no WAL; the new page's contents are logged by whatever writes into it.
  A LESSON THAT WANTS THE EVIDENCE SHOULD pg_waldump an INSERT that extends and show there is no
  extension record. Lesson 08's Next Steps do not claim otherwise.
DELIBERATELY TREATED AS BLACK BOXES: GetVictimBuffer, StrategyFreeBuffer, IOContextForStrategy and
the bulkwrite ring (09); BufferDesc, the clock sweep (09); pin accounting beyond the one line in
LimitAdditionalPins (10); max_safe_fds and the VFD LRU (named, not measured); BufferSync and the
checkpoint's spread writes (24); pendingUnlinks/SyncPostCheckpoint (19).

NO SYLLABUS REVISION THIS RUN. Lesson 08's title was taught exactly as written -- relation
extension, the segment map and the fsync request queue were all three taken in full -- and all
three inherited questions were answered or explicitly closed.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): seg, ext_insert, ext_copy, ext_copy_wz, tiny_copy, q1..q4, dedup, b, io_t,
iostat_t. NOTE: this run did NOT use the pgNN_ prefix convention and should have. Lesson 09 should
go back to pg9_*.

-- LESSON 09 (2026-10-28) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-28-postgres-buffer-manager-clock-sweep/README.md (single README, not
split -- BufferDesc, the sweep and the ring are one decision procedure, and the ring only makes
sense as a modification of the sweep).

SOURCE PROVENANCE: `apt-get source postgresql-16` for the third run in a row, version
16.15-0ubuntu0.24.04.1, the same build the cluster runs. PATCH CHECK DONE PER FILE as lesson 08
insisted: the patch series touches src/Makefile.global.in, bufmgr.c, procsignal.c, postgres.c,
pgstat_database.c, psql/settings.h, fe_utils/print.h, pg_config_manual.h, storage/procsignal.h,
tcop/tcopprot.h and tutorial/README. freelist.c, buf_init.c, buf_table.c and buf_internals.h are
UNTOUCHED; bufmgr.c's only hunk is at LockBufferForCleanup() (line 4923), nowhere near
BufferAlloc / GetVictimBuffer / PinBuffer / ExtendBufferedRel*. Trees/releases so far:
01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051, 05 @126976a8, 06 @0019a1ec,
07 = release 16.15, 08 = release 16.15, 09 = release 16.15.
Files read IN FULL: storage/buffer/README (276 lines -- lesson 08's handover asked for this and it
was worth it), storage/buffer/freelist.c (774 lines), include/storage/buf_internals.h (440 lines),
storage/buffer/buf_init.c (186 lines). Read in part: storage/buffer/bufmgr.c (5676 lines --
BufferAlloc, GetVictimBuffer, PinBuffer, PinBuffer_Locked, UnpinBuffer, InvalidateVictimBuffer,
BgBufferSync), include/storage/lwlock.h (NUM_BUFFER_PARTITIONS), access/heap/heapam.c (initscan's
`rs_nblocks > NBuffers / 4`), and the pg_buffercache 1.4 extension SQL in
/usr/share/postgresql/16/extension/.

*** THIS RUN HAD A CLUSTER, EVERY number is measured, AND THE INSTRUMENT WAS pg_buffercache +
pg_stat_io + pg_shmem_allocations -- NOT strace. *** PostgreSQL 16.15 (Ubuntu
16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, Linux 6.18, ext4, 2 vCPU / 8 GB RAM. block_size 8192,
shared_buffers 128 MB = 16,384 buffers except where a measurement says otherwise,
vacuum_buffer_usage_limit 256 kB, bgwriter_lru_maxpages 100, bgwriter_lru_multiplier 2.0,
data_checksums OFF, fsync on. pg_buffercache 1.4 AND pg_prewarm 1.2 INSTALLED FOR THE FIRST TIME IN
THIS TRACK -- both were already present in /usr/share/postgresql/16/extension/, `CREATE EXTENSION`
is all it took.
*** RECIPE NOTE, SIMPLER AGAIN THAN LESSON 08'S: postgresql-16 was ALREADY INSTALLED in the sandbox
image this run (psql 16.15 on PATH, cluster "16 main" present but down). `pg_ctlcluster 16 main
start` was the only step, and `sudo -u postgres psql -X -q` works directly -- no `su postgres -c`
quoting, no initdb, no /var/tmp dance. CHECK FOR AN EXISTING CLUSTER WITH pg_lsclusters BEFORE
INSTALLING ANYTHING. Two traps hit this run: (a) ALTER SYSTEM cannot run inside an implicit
transaction block, so psql -c with TWO statements fails -- use one -c per ALTER SYSTEM; (b) VACUUM
likewise cannot share a -c with other statements. ***
*** NO DEBUG SYMBOLS: postgresql-16-dbgsym IS NOT IN THE UBUNTU 24.04 ARCHIVE (apt-cache policy
returns nothing, apt-get install says "Unable to locate package"). gdb is installed but cannot
resolve StrategyControl, which is a static in freelist.c. ANY LESSON PLANNING TO READ A
SHARED-MEMORY STRUCT DIRECTLY MUST FIND ANOTHER WAY -- see the arc trick below, which is the way. ***

TOOK, and must not be re-taught:
- THE FOUR SHMEM ARRAYS and their per-buffer costs, MEASURED out of pg_shmem_allocations at
  shared_buffers=128MB and divided by 16,384: "Buffer Descriptors" 1,048,576 = EXACTLY 64.0 B
  (BUFFERDESC_PAD_TO_SIZE = (SIZEOF_VOID_P == 8 ? 64 : 1), pure cache-line padding against false
  sharing); "Buffer Blocks" 134,221,824 = 16384*8192 + 4096, the remainder being PG_IO_ALIGN_SIZE
  slack so the pool start can be TYPEALIGNed; "Buffer IO Condition Variables" 262,144 = 16.0 B
  (deliberately OUTSIDE BufferDesc to keep it under 64 B -- header comment says they would fit
  "but for now that is not done"); "Checkpoint BufferIds" 327,680 = 20.0 B = sizeof(CkptSortItem),
  preallocated so a checkpoint never mallocs (LESSON 24'S). *** AND "Buffer Strategy Status" = 28
  BYTES TOTAL FOR THE WHOLE CLUSTER -- that is the entire replacement policy's shared state. ***
  TRAP RECORDED: "Shared Buffer Lookup Table" is only 2,896 B; that is the HTAB directory, the
  elements come from a separate ShmemInitHash freelist and do not carry the name.
- THE 32-BIT STATE WORD, which is the heart of the whole file: 18-bit refcount (mask (1U<<18)-1),
  4-bit usagecount (mask 0x003C0000, shift 18), 10 flag bits (mask 0xFFC00000). ALL TEN FLAGS
  tabulated with their bit numbers: BM_LOCKED 22, BM_DIRTY 23, BM_VALID 24, BM_TAG_VALID 25,
  BM_IO_IN_PROGRESS 26, BM_IO_ERROR 27, BM_JUST_DIRTIED 28, BM_PIN_COUNT_WAITER 29,
  BM_CHECKPOINT_NEEDED 30, BM_PERMANENT 31. *** THE TRICK IS THAT BM_LOCKED IS THE BUFFER HEADER
  SPINLOCK AND IT LIVES IN THE WORD IT PROTECTS, so pin/unpin/usagecount are a CAS with ZERO
  lock-acquire pairs. The asymmetry from the header comment: a lock HOLDER may do arbitrary updates
  in one store (clearing BM_LOCKED in the same write); a non-holder is restricted to CAS, and plain
  atomic increment/OR/AND are FORBIDDEN because they cannot check BM_LOCKED. ***
- BM_MAX_USAGE_COUNT = 5, with the tradeoff comment quoted ("a large value (comparable to NBuffers)
  would approximate LRU semantics. But it can take as many as BM_MAX_USAGE_COUNT+1 complete cycles
  of clock sweeps to find a free buffer"). Four bits would allow 15; the policy chooses 5.
- BufferTag's five 4-byte fields (spcOid, dbOid, relNumber, forkNum, blockNum) and the constraint
  that forces that shape, quoted: the tag "must be sufficient to determine where to write the
  block, without reference to pg_class or pg_tablespace entries". SAME TUPLE as lesson 01's path
  and lesson 08's FileTag -- the identity is stable from filename to buffer header.
- NUM_BUFFER_PARTITIONS = 128 (lwlock.h), partition = hashcode % NUM_BUFFER_PARTITIONS, "NB:
  NUM_BUFFER_PARTITIONS must be a power of 2". Share lock to look up, exclusive to change a tag.
  Two rules: pin the found buffer BEFORE releasing the partition lock; lock multiple partitions in
  partition-number order. README's history recorded: one cluster-wide BufMgrLock before 8.1, split
  128 ways in 8.2.
- StrategyGetBuffer's TWO-STAGE SEARCH, in full, with four details that are easy to miss:
  (a) the freelist peek `firstFreeBuffer >= 0` is DELIBERATELY RACY and the comment defends it
      ("it'd be sad to acquire it here -- uselessly in most cases ... that's pretty harmless");
      the freelist is only never-used buffers plus InvalidateBuffer/tag-race returns, so once the
      pool is full it stays empty forever and every allocation is a sweep.
  (b) *** trycounter RESETS ON EVERY DECREMENT, NOT EVERY TICK. *** Only the PINNED branch does
      --trycounter; the usagecount-decrement branch sets trycounter = NBuffers. So
      "no unpinned buffers available" fires ONLY when NBuffers consecutive buffers were all
      pinned, never because the sweep is doing work. The sweep may run arbitrarily long while
      making progress.
  (c) ClockSweepTick is LOCK-FREE EXCEPT ON WRAP: nextVictimBuffer is a pg_atomic_uint32 only ever
      INCREMENTED, modulo applied at read time; the spinlock is taken on exactly one tick in
      NBuffers, purely to keep completePasses consistent for StrategySyncStart. Comment accepts the
      consequence ("buffers being returned slightly out of apparent order").
  (d) THE VICTIM IS RETURNED WITH ITS HEADER SPINLOCK STILL HELD, and the comment says why ("To
      ensure that no one else can pin the buffer before we do"). GetVictimBuffer immediately calls
      PinBuffer_Locked. There is no window.
- *** THE LESSON'S BEST TRICK, AND IT IS REUSABLE WHENEVER A SHARED-MEMORY COUNTER IS NOT EXPOSED:
  LOCATE THE CLOCK HAND FROM pg_buffercache.bufferid. Fill the pool with one generation of blocks,
  allocate exactly N fresh ones with pg_prewarm(...,'buffer',...) (which passes a NULL strategy, so
  every block goes through the normal sweep), and ask which bufferids the new generation occupies.
  MEASURED: 2,000 blocks landed in bufferid 373..2372 -- EXACTLY 2,000 CONSECUTIVE IDS, an arc, not
  a scatter. Next batch predicted 2373..4372, measured 2396..4395 (23 buffers skipped because
  catalog queries between the two calls re-pinned them); third batch predicted 4396..6395,
  MEASURED 4396..6395 EXACTLY, with a lead()-based gap check returning holes = 0. SO: THE HAND
  ADVANCES ONE BUFFER PER ALLOCATION PLUS ONE PER SKIP, and EVICTION ORDER IS POSITION ORDER, NOT
  AGE ORDER -- 14,000 buffers at usagecount 0 sat untouched because the hand was elsewhere. ***
- A SECOND SIGNATURE OF THE SWEEP, measured: after ONE full pass of allocations (16,384 prewarmed
  blocks into a 16,384-buffer pool) the usagecount histogram was 0=16027, 1=211, 2=18, 3=99, 4=8,
  5=21. BufferAlloc stamps each new tag with BUF_USAGECOUNT_ONE and the hand comes round and
  decrements it back to 0, so A POOL JUST FILLED ONCE IS, BEHIND THE HAND, A POOL OF
  USAGECOUNT-0 BUFFERS.
- PinBuffer's CAS loop, and THE PART THAT MATTERS MOST: the whole usagecount block is guarded by
  `ref == NULL` from GetPrivateRefCountEntry. If this backend already holds a pin it takes the else
  branch and TOUCHES THE SHARED state WORD NOT AT ALL. *** THE SHARED COUNTER MEASURES DISTINCT
  BACKEND PIN EPISODES, NOT ACCESSES. *** And with a strategy the increment is capped at 1 ("Ring
  buffers shouldn't evict others from pool. Thus we don't make usagecount more than 1").
  MEASURED saturation on a 3,449-block cached table, one full scan per iteration:
  1, 2, 3, 4, 5, 5, 5, 5 -- exactly BM_MAX_USAGE_COUNT, so a 5-touch and a 500-touch page are
  INDISTINGUISHABLE to the policy.
- GetVictimBuffer's THREE `goto again` PATHS, each a real race: (1) the dirty victim's content lock
  is held, so the acquisition is LWLockConditionalAcquire(LW_SHARED) and the comment names the bug
  that forced it ("two backends are both trying to split btree index pages, and the second one just
  happens to be trying to split the page the first one got from StrategyGetBuffer"); (2) a strategy
  would need a WAL flush, so StrategyRejectBuffer votes -- BAS_BULKREAD only, from_ring only, and it
  sets that slot to InvalidBuffer "necessary to prevent infinite loop if all ring members are
  dirty"; (3) InvalidateVictimBuffer failed. EXIT CONTRACT asserted: refcount == 1 and NONE of
  BM_TAG_VALID / BM_VALID / BM_DIRTY -- a victim is ANONYMOUS before BufferAlloc stamps the new tag.
- *** reuses AND evictions IN pg_stat_io ARE THE SAME EVENT, labelled by from_ring:
  pgstat_count_io_op(..., from_ring ? IOOP_REUSE : IOOP_EVICT). That is the measuring instrument
  for everything about rings. ***
- THE RING, structurally: GetAccessStrategy is a four-case switch -- BAS_NORMAL returns NULL (no
  object at all), BAS_BULKREAD 256 kB, BAS_BULKWRITE 16*1024 kB, BAS_VACUUM 256 kB overridable by
  vacuum_buffer_usage_limit. Then GetAccessStrategyWithSize does ring_buffers = kb / (BLCKSZ/1024),
  returns NULL if 0 (*** 0 kB MEANS NO RING, NOT A TINY RING ***), and clamps
  ring_buffers = Min(NBuffers / 8, ring_buffers). The object is BACKEND-PRIVATE, palloc'd, no lock:
  btype, nbuffers, a `current` cursor, a flat Buffer[]. GetBufferFromRing accepts a slot only if
  refcount == 0 AND usagecount <= 1 (the counterpart of PinBuffer's capped increment: 1 means
  "only I touched this"). AND THE RING IS NOT A RESERVATION -- slots are filled by AddBufferToRing
  from inside StrategyGetBuffer, i.e. by an ordinary sweep/freelist allocation.
- *** THE SINGLE MOST REUSABLE FACT IN THE LESSON: reads - reuses = THE RING SIZE IN BUFFERS,
  readable straight out of pg_stat_io with no source access. MEASURED FOUR WAYS:
    seq scan t_mid (7,242 blk), bulkread:        7,242 - 7,210 = 32
    VACUUM t_big, BUFFER_USAGE_LIMIT '256kB':   76,376 - 76,344 = 32
    VACUUM t_big, BUFFER_USAGE_LIMIT '2MB':     49,775 - 49,519 = 256   (2MB/8kB, no rounding)
    COPY FROM (15,560 extends), bulkwrite:      15,560 - 13,512 = 2,048
    seq scan t_mid at shared_buffers=128kB:      7,242 - 7,240 = 2      (= Min(16/8, 32)) ***
- THE SEQ-SCAN THRESHOLD, measured on both sides: initscan's `rs_nblocks > NBuffers / 4` = 4,096
  blocks at 16,384 buffers. t_small (3,449 blk, BELOW) -> context `normal`, 3,449 buffers resident,
  THE WHOLE TABLE CACHED. t_mid (7,242 blk, ABOVE) -> context `bulkread`, 32 buffers resident.
  NOTHING GRADUAL. And the threshold MOVES WITH shared_buffers, so the same table and query change
  caching behaviour completely when the pool is resized, with nothing in EXPLAIN to say so.
- *** THE RING IS PER PARTICIPANT, NOT PER QUERY -- caught by accident. The first t_mid run was
  taken with max_parallel_workers_per_gather=2 and showed 96 resident buffers, not 32, with the
  leader reading only 2,392 of 7,242 blocks. leader + 2 workers x 32 = 96. BufferAccessStrategy is
  palloc'd backend-local, so parallelism MULTIPLIES the footprint the ring is meant to bound. SET
  max_parallel_workers_per_gather=0 FOR ANY BUFFER MEASUREMENT. ***
- *** THE HEADLINE EXPERIMENT, AND THE ONE A LATER LESSON SHOULD CITE: THE SAME 242 MB READ, TWO
  PLAN NODES, OPPOSITE OUTCOMES. Warm t_small (3,449 blk) to usagecount 2, then read t_big
  (31,035 blk = 242 MB) through a 128 MB pool.
    A) SEQ SCAN (BAS_BULKREAD): bulkread reads=31035 reuses=31003 EVICTIONS=0; t_small still 3,449
       buffers at avg usagecount 2.00, UNTOUCHED; t_big occupies 32 buffers; 12,647 buffers unused.
    B) INDEX SCAN (no strategy object): normal reads=36013 EVICTIONS=23320; t_small ZERO BUFFERS,
       GONE; t_big 13,944 + index 2,210 buffers at avg usagecount 0.44; pool unused = 0.
  initscan() builds a ring; index_beginscan and the bitmap heap scan DO NOT. An index scan that
  touches most of a large table is the pathology the ring prevents AND the case it does not cover. ***
- BUFFER_USAGE_LIMIT 0 MEASURED AS THE CONTROL: context flips from `vacuum` to `normal`, reuses
  becomes NULL, and one VACUUM of t_big took 16,255 of 16,384 buffers with 60,309 evictions and
  zero unused. That single row is the whole justification for the ring's existence.
- BgBufferSync's ONE call into the policy: StrategySyncStart returns nextVictimBuffer % NBuffers,
  completePasses PLUS nextVictimBuffer / NBuffers (correcting for an unlanded wrap), and
  numBufferAllocs read with pg_atomic_exchange_u32(...,0) -- *** READING IT RESETS IT ***. The
  bgwriter scans forward from that position "which it does not change!", keeps smoothed_density
  (init 10.0) and multiplies by bgwriter_lru_multiplier, stopping at bgwriter_lru_maxpages.
  MEASURED on the dirtied cluster: buffers_clean 10448, maxwritten_clean 95, buffers_alloc 804317,
  buffers_backend 498979. NOTE: StrategyGetBuffer increments numBufferAllocs AFTER the ring check
  ("buffers recycled by a strategy object are intentionally not counted here"), so buffers_alloc
  UNDERCOUNTS ring-served accesses by design.
- ANSWERS TO LESSON 08'S THREE INHERITED QUESTIONS, all three closed:
  (1) GetVictimBuffer: opened in full (the three retry paths, the exit contract, the
      IOOP_REUSE/IOOP_EVICT split). Lesson 08's quoted claim that "writing out victim buffers is
      the most expensive part of extending the relation" is consistent with the code path
      (FlushBuffer + ScheduleBufferTagForWriteback inside the BM_DIRTY branch) but was NOT priced
      independently this run -- the extension path was not re-measured. Still open as a price.
  (2) The bulkwrite ring: ANSWERED AND BETTER THAN ASKED. Lesson 08's "extends=14288, writes=12240,
      the ring evicted 86% of what it created" is exactly reads/extends - reuses = ring size. This
      run measured extends 15,560 and reuses 13,512 on a fresh COPY: the difference is 2,048, and
      2,048 = Min(NBuffers/8, 16MB/8kB) = Min(2048, 2048), which at shared_buffers=128MB is a
      coincidence of the two constants. The ring did not "waste" 86%; it reused 2,048 slots 13,512
      times, writing out each previous occupant.
  (3) What a pin costs: stated as the state-word CAS and the backend-private refcount short-circuit;
      REFCOUNT_ARRAY_ENTRIES / PrivateRefCountOverflowed deliberately left to lesson 10.
- *** UPSTREAM HAS ALREADY REMOVED THE FREELIST, and the lesson says so: commit
  2c789405275928ce0d2ceb7c4add91d27df92502, "bufmgr: Remove freelist, always use clock-sweep",
  Greg Burd, 2025-09-05, on master; reviewed by Tomas Vondra and Andres Freund; 6 files,
  +31/-209, INCLUDING 40 LINES OF THE README THIS LESSON QUOTES. Rationale quoted from the commit
  message: "While on the surface this appears to be removing an optimization it is in fact
  eliminating code that induces overhead in the form of synchronization that is problematic for
  multi-core systems", and the main motive was that "having the freelist would require dedicated
  complexity in several upcoming patches". have_free_buffer() goes too, and pg_autoprewarm is
  capped at NBuffers. A FOLLOW-ON PATCH REMOVES buffer_strategy_lock ENTIRELY. So the two-stage
  search measured here is a snapshot of a structure mid-retirement; 16.15 still has both paths. ***
- HONEST NULL RESULT, recorded as one: THE SIX-PASS BOUND IMPLIED BY BM_MAX_USAGE_COUNT IS NOT
  VISIBLE IN WALL CLOCK on this hardware. A 16,035-block eviction storm against a pool saturated
  at usagecount 5 measured 65.7/73.3/65.7 ms and at usagecount 1 measured 86.5/74.1/86.5 ms --
  no separation above the noise, because the per-block 8 KB memcpy out of the OS page cache
  dominates the header examinations. The bound is real in the code; this box cannot see it.
  pg_prewarm's usagecount-K setup DOES work and is in the lesson (prewarm the same range K times).
DELIBERATELY TREATED AS BLACK BOXES: the LWLock implementation and pins vs content locks (10) --
LockBufHdr/UnlockBufHdr and content_lock are NAMED and counted, never explained; StartBufferIO /
TerminateBufferIO and the BM_IO_IN_PROGRESS condition variable (named only); FlushBuffer's internals
and the writeback context (24); BufferSync and BM_CHECKPOINT_NEEDED (24); hint bits and
MarkBufferDirtyHint (15); localbuf.c and temp-table buffers (not scheduled -- a candidate for 35).

NO SYLLABUS REVISION THIS RUN. Lesson 09's title was taught exactly as written -- BufferDesc, the
clock-sweep victim search and the strategy ring were all three taken in full -- and all three of
lesson 08's inherited questions were answered or explicitly narrowed. Lesson 08's handover asked
for five measurements by name and all five were taken: (a) the usagecount histogram under a scan
(and better, the hand's position); (b) the ring proved to be a ring (32 buffers of a 7,242-block
table); (c) the BM_MAX_USAGE_COUNT = 5 ceiling by repeated reads; (d) reads/hits/evictions out of
pg_stat_io across hit and miss; (e) pg_buffercache_summary and pg_buffercache_usage_counts, which
DO exist in the 1.4 extension shipped with 16.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): t_small, t_mid, t_big, t_copy, t_tiny, in database `bufdemo`. NOTE: LIKE
LESSON 08, THIS RUN DID NOT USE THE pgNN_ PREFIX AND SHOULD HAVE. Lesson 10, please use pg10_*.

-- HANDOVER TO LESSON 10 -----------------------------------------------------------------------
Scope for lesson 10, "Buffer pins versus content locks, and the LWLock implementation". LESSON 09
NAMED BOTH MECHANISMS AND REFUSED TO EXPLAIN EITHER, so 10 has a clean surface to open.
THE INHERITED QUESTIONS, in descending order of value:
 (1) *** THE buffer/README's "Notes About Shared Buffer Access Rules" SECTION IS LESSON 10'S
     PRIMARY TEXT AND LESSON 09 DID NOT TEACH IT. *** Lesson 09 read all 276 lines but taught only
     the "Buffer Manager's Internal Locking", "Normal Buffer Replacement Strategy", "Buffer Ring
     Replacement Strategy" and "Background Writer's Processing" sections. THE FIVE NUMBERED ACCESS
     RULES AND THE CLEANUP-LOCK PROTOCOL ARE UNTOUCHED AND ARE 10'S CORE: rule 4 (hint bits may be
     OR'd in under a SHARED lock because a conflicting update only loses a hint) and rule 5 (to
     remove a tuple you must observe refcount == 1 while holding the exclusive lock) are the two
     worth measuring. LockBufferForCleanup's single-waiter limitation ("The current implementation
     only supports a single waiter for pin-count-1 on any particular shared buffer") and
     BM_PIN_COUNT_WAITER / wait_backend_pgprocno are the mechanism; note that the ONE Debian patch
     touching bufmgr.c patches exactly this function, so READ THE PATCH, not just the file.
 (2) THE PRIVATE REFCOUNT MACHINERY, which lesson 09 used as an explanation and never opened:
     GetPrivateRefCountEntry / ReservePrivateRefCountEntry / NewPrivateRefCountEntry,
     REFCOUNT_ARRAY_ENTRIES, PrivateRefCountOverflowed, and the ResourceOwner integration
     (ResourceOwnerRememberBuffer / ResourceOwnerEnlargeBuffers). *** LESSON 08 LEFT THIS AS A LIVE
     OPEN ITEM BECAUSE LimitAdditionalPins() SUBTRACTS BOTH OF THEM AND THAT ARITHMETIC REDUCED A
     64-BLOCK EXTENSION TO ONE BLOCK AT shared_buffers=128kB. *** Closing it closes a two-lesson
     thread. Also: README says pins may be held for long intervals and across a whole seqscan page,
     but NOT across transaction boundaries -- ask what enforces that.
 (3) THE LWLock ITSELF: lwlock.c's state word, the shared/exclusive/wait-queue encoding,
     LWLockAcquire / LWLockConditionalAcquire / LWLockAcquireOrWait, the tranches
     (LWTRANCHE_BUFFER_CONTENT is assigned in buf_init.c and lesson 09 quoted that line), and the
     MainLWLockArray layout with BUFFER_MAPPING_LWLOCK_OFFSET. Lesson 09 measured that
     GetVictimBuffer MUST use LWLockConditionalAcquire or deadlock; 10 owns why.
Take: storage/buffer/README's access-rules half IN FULL; storage/lmgr/lwlock.c and
include/storage/lwlock.h (the state word, the wait queue, the tranche registry); bufmgr.c's
LockBuffer / LockBufferForCleanup / ConditionalLockBufferForCleanup / CheckBufferIsPinnedOnce /
PinBuffer / UnpinBuffer and the PrivateRefCount block at the top of the file; and
include/storage/buf_internals.h's BufferDescriptorGetContentLock / BufferDescriptorGetIOCV.
MEASURE IT. Good candidates: (a) pg_buffercache.pinning_backends and pg_buffercache_summary()'s
buffers_pinned are THE ONLY SQL-VISIBLE PIN COUNTS -- find something that holds a pin long enough
to see (a cursor left open mid-seqscan is the obvious one); (b) make LockBufferForCleanup WAIT, by
running VACUUM against a table with an open cursor pinning a page, and watch pg_stat_activity's
wait_event / wait_event_type (BufferPin); (c) pg_stat_activity and pg_wait_events (16 has the
latter) give the LWLock wait_event names -- BufferMapping, BufferContent, BufferIO are all
reachable; (d) prove rule 4 by watching a SELECT dirty a page via hint bits under a shared lock
(pg_buffercache's isdirty before/after, with lesson 09's measurement method); (e) if a
contention measurement is wanted, pgbench -c N with a small shared_buffers makes BufferMapping
waits appear -- but note lesson 09's finding that this box is 2 vCPU, so contention results will be
weak and should be reported as such.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, the hole (02); the tuple header,
t_ctid's four meanings, the infomask catalogues (03); alignment, varlena, the 126-byte boundary,
TOAST (04); the FSM (05); ANY of the visibility map (06); LP_REDIRECT, the line-pointer state
machine, pd_prune_xid, PD_PAGE_FULL, the Max(fillfactor, BLCKSZ/10) prune gate,
heap_page_prune's plan/execute split, the HOT-blocking matrix (07); the smgr switch, the MdfdVec
segment map, mdnblocks's walk, the RELSEG_SIZE invariant, mdzeroextend's branch, PG_IOV_MAX 32, the
MAX_BUFFERS_TO_EXTEND_BY 64 ramp, skipFsync, FileTag, the whole fsync request queue (08); AND NOW,
FROM 09: the four shmem arrays and their per-buffer sizes, the 32-bit state word and all ten BM_*
flags, BM_MAX_USAGE_COUNT 5, BufferTag's five fields, NUM_BUFFER_PARTITIONS 128,
StrategyGetBuffer's two-stage search and ClockSweepTick, trycounter's reset semantics,
GetVictimBuffer's three retry paths, the IOOP_REUSE/IOOP_EVICT split, all four
BufferAccessStrategyType ring sizes and the Min(NBuffers/8) cap, the initscan NBuffers/4 threshold,
and BgBufferSync's StrategySyncStart call.
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 126, 4069, 4095, FSM_CAT_STEP 32,
the FSM leaf offset 4123+k, the fingerprint "00 20 04 20" at offsets 16-19 (= zero special space,
NOT "heap"), MAPSIZE 8168, HEAPBLOCKS_PER_PAGE 32672, the VM offset formula 24 + (b % 32672)/4,
VISIBLE_MASK8 0x55 / FROZEN_MASK8 0xaa, the 59-byte XLOG_HEAP2_VISIBLE record, the 59/61-byte
XLOG_HEAP2_PRUNE record, the 71-byte HOT_UPDATE record, 166 vs 247 WAL bytes per update (HOT vs
cold), the 140-bytes-per-row geometry, lesson 07's correction that lesson 06's "59 heap fetches" is
rows-on-the-old-block + 1, RELSEG_SIZE observed as 1,073,741,824 bytes of segment 0,
MAX_BUFFERS_TO_EXTEND_BY 64, PG_IOV_MAX 32, max_requests = Min(NBuffers, 10000000),
FSYNCS_PER_ABSORB 10, the ~1,940:1 writes-to-fsyncs dedup ratio, and now: 64 bytes per BufferDesc,
28 bytes of Buffer Strategy Status, 16,384 buffers at shared_buffers=128MB, the ring sizes
32/2048/32 buffers and the Min(NBuffers/8) cap, reads-minus-reuses = ring size, the 4,096-block
seq-scan threshold, and the 0-evictions-vs-23,320-evictions pair for the same 242 MB read.
Do NOT take: the heavyweight lock manager, LOCKTAG and the deadlock detector (11); tuple lock
strengths and the conflict table (12); hint bits' FPI machinery (15, 22); snapshots and
GlobalVisState (14); the checkpointer's spread writes, BufferSync and BM_CHECKPOINT_NEEDED (24);
crash recovery (25).
OPEN ITEMS carried forward:
- FOR LESSON 10: the three inherited questions above, and lesson 08's REFCOUNT_ARRAY_ENTRIES /
  PrivateRefCountOverflowed item, which is now (2).
- NEW FROM 09, FOR WHOEVER WANTS IT: write a tiny extension exposing StrategySyncStart()'s three
  outputs as a SQL function. nextVictimBuffer and completePasses have NO SQL exposure, there are no
  debug symbols in this sandbox, and the arc trick is an inference that only works on a quiet
  cluster. That extension would turn it into a direct reading and make the six-pass bound
  measurable.
- NEW FROM 09: GetVictimBuffer's cost in the EXTENSION path was not priced (question (1) above is
  only half closed). A run that re-does lesson 08's COPY measurement while watching pg_stat_io's
  bulkwrite writes and the bgwriter's buffers_clean would close it.
- FOR LESSON 14: unchanged -- GlobalVisState / vistest and heap_prune_satisfies_vacuum were black
  boxes, and heap_page_prune_opt carries a live XXX about old_snapshot_threshold citing
  postgresql.org/message-id/20200403001235.e6jfdll3gh2ygbuc%40alap3.anarazel.de.
- FOR LESSON 19 (unchanged from 05/06/07/08, still not closed): fsm_vacuum_page() and
  FreeSpaceMapVacuumRange() have STILL not been read -- they are the only code that makes the upper
  FSM levels true, AND they are the answer to lesson 07's open question about whether a cold update
  can be routed onto a prune-recovered page. Plus lesson 06's 18,400 WAL bytes over 118 blocks
  breakdown, VACUUM's two heap passes, and 08's register_unlink_segment / pendingUnlinks /
  SyncPostCheckpoint (the one-checkpoint delay before a dropped relation's file is unlinked is
  observable with DROP TABLE + two CHECKPOINTs). NEW FROM 09: vacuum_buffer_usage_limit's full
  range and VACUUM's ring behaviour are now measured (32 and 256 buffers at '256kB' and '2MB',
  and the normal strategy at 0) -- cite those rather than re-measuring.
- FOR LESSON 24: unchanged from 08 -- ProcessSyncRequests was read and BufferSync left alone; the
  race the comment describes and the Absorb-after-BufferSync ordering are 24's. NEW FROM 09: the
  "Checkpoint BufferIds" array is 20 bytes per buffer of preallocated shared memory and exists
  precisely so a checkpoint never allocates; BM_CHECKPOINT_NEEDED is bit 30; and
  ScheduleBufferTagForWriteback / WritebackContext / WRITEBACK_MAX_PENDING_FLUSHES were named in 09
  and never explained.
- FOR LESSON 29: unchanged -- lesson 07's cold-update WAL run emitted 74 Btree/DELETE records.
- FOR LESSON 31: unchanged -- pg_class.relallvisible's cost formula, and whether costsize.c models
  HOT at all. NEW FROM 09: whether the planner knows anything about shared_buffers at all (the
  effective_cache_size / seq_page_cost relationship) is a fair question for 31, given that lesson
  09 showed the SAME query's cache effect depends entirely on which node reads the heap.
- UNOBSERVED, for whoever gets there first: MaxHeapTuplesPerPage = 291 has been DERIVED three times
  (05, 07) and never reached.
- UNOBSERVED BRANCH, for whichever lesson runs with checksums on first: every measurement in
  lessons 03-09 was taken with data_checksums OFF. Still unexercised:
  visibilitymap_prepare_truncate's XLogHintBitIsNeeded() -> log_newpage_buffer (06), and
  heap_page_prune's MarkBufferDirtyHint branch under XLogHintBitIsNeeded() (07).
- UNEXERCISED CODE PATHS: heap_prune_chain's outcome (4), the orphaned-redirect self-heal (07);
  register_dirty_segment's queue-full local-fsync fallback and ExtendBufferedRelShared's
  `existing_id >= 0` branch with its errhint("This has been seen to occur with buggy kernels")
  (08); and NEW FROM 09, StrategyGetBuffer's `elog(ERROR, "no unpinned buffers available")` -- an
  attempt to force it at shared_buffers=128kB (16 buffers) with a nested-loop join and a large
  prewarm did NOT trigger it, because trycounter resets on every decrement. It needs NBuffers
  SIMULTANEOUS pins, which means a query whose concurrent pin set exceeds the whole pool.
- NOT MEASURED, carried from 07: the SUMMARIZING-INDEX (BRIN) HOT path.
- NEW FROM 08, still worth one cheap run: truncate a relation to an exact multiple of RELSEG_SIZE
  and check whether segment K+1 is kept at length 0, as mdtruncate's comment promises.
- A DOCUMENTATION PATCH WORTH PROPOSING UPSTREAM, unchanged from 06: visibilitymap.c's LOCKING
  section still says "only VACUUM currently sets visibility map bits", which heap_multi_insert's
  all_frozen_set contradicts.

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 06's note was dropped and TRACK-ARCHIVE.md gained the row
"PG 06" pointing at commit 01878f9bb5d3979a81c03925c2ae8219292b7dc6 / blob
df40aec01bc9a115a92e1d609bce79b1b8901e25, which is the last state of TRACK.md carrying it. This
block now holds 07, 08 and 09 -- exactly the three the policy allows. WHEN LESSON 10 SHIPS IT MUST
DROP THE LESSON 07 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap covered lesson 05; pg_visibility covers 06.
  Lesson 01's hands-on installs all three; lesson 02's needs only pageinspect; lesson 05's needs
  pageinspect and pg_freespacemap; lesson 07's needs pageinspect and pgstattuple; lesson 08 needed
  only pageinspect (and barely that -- its instrument was strace); LESSON 09 NEEDED NEITHER, only
  pg_buffercache 1.4 and pg_prewarm 1.2, both of which are now CONFIRMED PRESENT AND WORKING.
- CONFIRMED 2026-10-22 THROUGH 2026-10-28 (seven runs): the cloud sandbox CAN run a cluster.
  *** THREE RECIPES NOW, AND THE THIRD IS THE SHORTEST. CHECK pg_lsclusters FIRST. ***
  (A) Lessons 03-07's: create a non-root user, initdb -D /var/tmp/pgdata --no-sync as that user,
      pg_ctl with -o '-p 5455 -k /var/tmp'. TWO TRAPS: initdb refuses to run as root, and it
      refuses a data directory under the scratchpad path -- /var/tmp works.
  (B) Lesson 08's: `apt-get install postgresql postgresql-contrib`, then
      `pg_ctlcluster 16 main start`, then run everything as `su postgres -c "psql ..."`.
  (C) LESSON 09'S, AND TRY THIS FIRST: postgresql-16 was ALREADY INSTALLED in the sandbox image,
      with psql 16.15 on PATH and the cluster "16 main" present but down. `pg_lsclusters` shows it.
      Then `pg_ctlcluster 16 main start` and `sudo -u postgres psql -X -q -d DB -c "..."` -- no
      su quoting, no initdb, no world-readable-script dance for -c work. ALTER SYSTEM +
      `pg_ctlcluster 16 main restart` applies PGC_POSTMASTER settings (shared_buffers,
      max_connections). The contrib extensions were already in
      /usr/share/postgresql/16/extension/ even though postgresql-contrib reported as not installed
      -- CHECK THE DIRECTORY BEFORE INSTALLING.
      TWO TRAPS NEW THIS RUN: ALTER SYSTEM cannot run inside an implicit transaction block, so
      `psql -c "ALTER SYSTEM SET a; ALTER SYSTEM SET b;"` FAILS -- one -c per statement. Same for
      VACUUM. And `/usr/bin/time` does NOT exist; use psql's `\timing on` instead.
  ALL recipes keep the world-readable-script trap for psql -f: put scratch SQL in /tmp/<dir> with
  chmod 777 (or /var/tmp). The cluster is per-run and does not persist, so table names from earlier
  runs are gone.
- *** strace IS AVAILABLE AND IT IS THE RIGHT INSTRUMENT FOR ANYTHING BELOW THE BUFFER MANAGER.
  gdb is installed BUT THERE ARE NO DEBUG SYMBOLS -- postgresql-16-dbgsym IS NOT IN THE UBUNTU
  24.04 ARCHIVE (confirmed lesson 09: apt-cache policy returns nothing). SO gdb CANNOT READ
  SHARED-MEMORY STRUCTS OR FILE-STATIC VARIABLES. THE HARNESS LESSON 08 BUILT AND LATER LESSONS
  SHOULD COPY: mkfifo a pipe, start
  `su postgres -c "psql -d DB -At -q < fifo > out"` in the background, write
  "SELECT pg_backend_pid();" into the fifo, read the pid out of the output file, then
  `strace -f -tt -p <pid> -e trace=<syscalls> -o trace.txt &`, then feed the real SQL into the fifo
  and poll for a DONE marker. That gives a clean per-statement syscall trace of a NAMED backend.
  Note that parallel workers are NOT children of that backend, so -f does not follow them; disable
  parallelism or trace the postmaster if you need them. ***
- *** WHEN A SHARED-MEMORY COUNTER IS NOT EXPOSED IN SQL, LESSON 09'S TECHNIQUE IS THE FALLBACK:
  drive the structure to a known state, apply an exactly-counted number of operations, and read the
  CONSEQUENCE out of a view that does exist. Locating nextVictimBuffer as a contiguous arc of
  pg_buffercache.bufferid values is the worked example, and it predicted the next arc's start to
  the buffer. pg_shmem_allocations is the other under-used view: every ShmemInitStruct name is
  queryable with its size, so a per-element struct size can be read back by division. ***
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04), pg5_* (05), pg6_* (06),
  pg7_* (07), 08's unprefixed set (seg, ext_*, tiny_copy, q1..q4, dedup, b, io_t, iostat_t) and
  09's unprefixed set (t_small, t_mid, t_big, t_copy, t_tiny, in database `bufdemo`) -- two lapses
  in a row. GO BACK TO THE pgNN_ PREFIX CONVENTION.
- pg_waldump lives at /usr/lib/postgresql/16/bin/pg_waldump, works on the live cluster's segments,
  and `--stats=record` prints a per-record-type record/FPI byte breakdown. Lesson 06 used it to
  price XLOG_HEAP2_VISIBLE and lesson 07 to price XLOG_HEAP2_PRUNE against Btree/INSERT_LEAF.
  Lessons 21-25 should use it rather than reasoning about WAL on paper. METHOD THAT WORKS WELL,
  from lesson 07: pg_switch_wal() first, record pg_current_wal_lsn(), run the workload, record the
  LSN again, then `pg_waldump -s <lsn1> -e <lsn2> --stats=record` for the table and
  `-r <rmgr>` for verbatim records. That isolates one workload's WAL exactly.
- *** GETTING THE SOURCE: `apt-get source postgresql-16` IS THE RIGHT ROUTE IN THIS SANDBOX AND
  LESSONS 07, 08 AND 09 ALL USED IT. *** Ubuntu 24.04 ships deb-src DISABLED in
  /etc/apt/sources.list.d/ubuntu.sources (a deb822 file with `Types: deb`); the fix is
  `sed -i 's/^Types: deb$/Types: deb deb-src/'` on that file, then `apt-get update`, then
  `apt-get source postgresql-16` into a world-readable dir. ~33 MB, a few seconds, and it unpacks
  postgresql-16-16.15/ WITH debian/patches/ applied -- so CHECK WHICH FILES THE PATCH SERIES
  TOUCHES BEFORE QUOTING ANYTHING, per file and not in bulk: as of 16.15 there are 15 patches and
  one of them DOES touch src/backend/storage/buffer/bufmgr.c (at LockBufferForCleanup only).
  Extract the +++/--- paths and check the hunk offsets. This is strictly better than WebFetching
  files one at a time: it is the exact build the cluster runs, it allows reading a file IN FULL, and
  WebFetch on postgresql.org summarises rather than reproducing. The Sources section must still
  cite only URLs actually seen that run; the local tree is evidence, not a citation.
- pg_buffercache 1.4 (pg_buffercache_pages, pg_buffercache_summary, pg_buffercache_usage_counts)
  and pg_prewarm 1.2 are INSTALLED AND MEASURED AS OF LESSON 09 and are the right tools for
  anything about the pool; pgstattuple was used by lesson 07 and is needed again by 19; amcheck for
  lessons 28-29 (it is the closest thing PostgreSQL has to SQLite's integrity_check, and the
  comparison is worth making explicitly when lesson 28 arrives).
- *** MEASUREMENT HYGIENE ESTABLISHED BY LESSON 09 AND WORTH REPEATING: set
  max_parallel_workers_per_gather = 0 for anything that counts buffers or I/O, because
  backend-local structures (BufferAccessStrategy rings, and presumably more) are per participant
  and parallel workers are a separate backend_type in pg_stat_io. And `pg_stat_reset_shared('io')`
  immediately before each measurement -- pg_stat_io accumulates across the whole cluster lifetime
  and its column is `context`, not `io_context`. ***

-- LESSON 10 (2026-10-29) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-29-postgres-pins-content-locks-lwlock/README.md (single README, not
split -- pins, content locks and the cleanup lock are one access-control story, and the LWLock is
the implementation of the middle one; splitting would have hidden that the cleanup lock is built
out of the other two).

SOURCE PROVENANCE: `apt-get source postgresql-16` for the fourth run in a row, version
16.15-0ubuntu0.24.04.1, the same build the cluster runs. PATCH CHECK DONE PER FILE: lwlock.c,
lwlock.h, lwlocknames.txt, buf_internals.h, pruneheap.c, vacuumlazy.c and proclist_types.h are
UNTOUCHED. *** bufmgr.c's single hunk IS INSIDE THIS LESSON'S SUBJECT THIS TIME and was read
directly: v8-0001-Fix-recovery-conflict-SIGUSR1-handling.patch changes TWO COMMENT LINES above
HoldingBufferPinThatDelaysRecovery() at line 4923 ("Check called from RecoveryConflictInterrupt
handler" -> "Check called from ProcessRecoveryConflictInterrupts()"). NO EXECUTABLE LINE of the pin
or lock path differs from upstream. *** Trees/releases so far: 01 @ddd468ca, 02 @3cde90ec,
03 @f2edf5dd, 04 @f52d4051, 05 @126976a8, 06 @0019a1ec, 07 = release 16.15, 08 = release 16.15,
09 = release 16.15, 10 = release 16.15.
Files read IN FULL: storage/lmgr/lwlock.c (1994 lines), include/storage/lwlock.h (222 lines),
storage/lmgr/lwlocknames.txt (54 lines), storage/buffer/README (276 lines, re-read for the
access-rules section). Read in part: storage/buffer/bufmgr.c (PinBuffer, UnpinBuffer,
ReservePrivateRefCountEntry, NewPrivateRefCountEntry, GetPrivateRefCountEntry, LockBuffer,
LockBufferForCleanup, ConditionalLockBufferForCleanup, CheckBufferIsPinnedOnce,
HoldingBufferPinThatDelaysRecovery, the PrivateRefCount comment block at 175-205),
access/heap/pruneheap.c (heap_page_prune_opt), access/heap/vacuumlazy.c (the cleanup-lock branch of
lazy_scan_heap at 958-1001), include/postmaster/postmaster.h (MAX_BACKENDS and its note),
include/storage/proclist_types.h, and the XLogReadBufferForRedoExtended(..., true, ...) call sites
across src/backend/access/.

*** THIS RUN HAD TWO CLUSTERS -- A PRIMARY AND A STREAMING STANDBY -- AND EVERY number is measured.
*** PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, Linux 6.18, ext4, 2 vCPU / 8 GB RAM.
shared_buffers 128 MB = 16,384 buffers, max_connections 100, block_size 8192, data_checksums OFF.
pg_buffercache 1.4, pageinspect 1.12, pg_prewarm 1.2, pgbench 16.15.
*** THREE NEW ENVIRONMENT FACTS, ALL REUSABLE:
(a) postgresql-16 WAS ALREADY INSTALLED AGAIN (lesson 09's note holds); `pg_ctlcluster 16 main
    start` was the only step. deb-src had to be re-enabled: `sed -i 's/^Types: deb$/Types: deb
    deb-src/' /etc/apt/sources.list.d/ubuntu.sources` then `apt-get update`.
(b) *** postgresql-server-dev-16 INSTALLS AND IS THE ANSWER TO LESSON 09'S "NO DEBUG SYMBOLS"
    PROBLEM FOR ANYTHING THAT IS A COMPILE-TIME CONSTANT. A 30-line C program compiled with
    `gcc -I $(pg_config --includedir-server)` prints sizeof/offsetof/macro values directly. TRAP:
    postgres.h #defines printf to pg_printf, which is not in any library you link, so `#undef
    printf` and use fprintf(stdout, ...). This does NOT help with runtime shared-memory statics
    (StrategyControl, nextVictimBuffer) -- lesson 09's arc trick is still the way for those. ***
(c) A STANDBY IS CHEAP AND WORTH BUILDING: `pg_basebackup -D /var/lib/postgresql/16/standby -R -C
    -S lab_standby -X stream -c fast -h /var/run/postgresql -U postgres`, then start it with
    pg_ctl and -o '-c port=5433 -c data_directory=... -c hot_standby=on -c
    max_standby_streaming_delay=8s -c log_recovery_conflict_waits=on', pointing config_file/hba_file
    at /etc/postgresql/16/main/. chmod 700 the data dir first. wal_level was already replica.
    LESSONS 24-27 SHOULD DO THIS. ***
Traps hit: VACUUM and ALTER SYSTEM still cannot share a `psql -c` with other statements -- put GUCs
in PGOPTIONS instead (`PGOPTIONS='-c vacuum_freeze_min_age=0' psql -c 'VACUUM (FREEZE) t'`). And
psql -f scripts AND any file a background psql writes must live somewhere world-readable
(/var/tmp/lab, chmod a+w) -- the scratchpad path is not readable by the postgres user.

TOOK, and must not be re-taught:
- THE TWO MECHANISMS AND THEIR DIFFERENT PROPOSITIONS, from storage/buffer/README: a pin says THE
  PAGE IS STILL HERE, a content lock says NOBODY IS EDITING IT. All five access rules tabulated.
  *** RULE 2 IS WHY PINS EXIST AT ALL (drop the content lock, keep the pin, keep reading tuple
  data) and it is the licence under which lesson 03 read a HeapTuple pointing into shared memory.
  RULE 4 (hint bits need only a shared lock) IS LESSON 15'S and was deliberately left there. ***
  Ordering is absolute: LockBuffer() opens with Assert(BufferIsPinned(buffer)), and UnpinBuffer
  closes with Assert(!LWLockHeldByMe(content_lock)). Lock inside pin, always.
- *** BUF_REFCOUNT_MASK == MAX_BACKENDS == 262143 == 2^18-1, AND THAT IS CAUSAL, NOT A COINCIDENCE.
  postmaster.h: "MAX_BACKENDS is limited to 2^18-1 because that's the width reserved for buffer
  references in buf_internals.h." The server's process ceiling IS the refcount field width.
  Lesson 09 measured the field; this closes why it is that wide. ***
- THE BACKEND-LOCAL PIN HALF: REFCOUNT_ARRAY_ENTRIES = 8, PrivateRefCountArray + PrivateRefCountHash
  overflow, round-robin displacement via PrivateRefCountClock++ % 8, and the Reserve/New split whose
  reason is quoted ("called with a spinlock held"). A repeat pin of a buffer you already hold is a
  linear scan of 8 slots plus ref->refcount++ -- NO ATOMICS AT ALL on that path.
  MEASURED: 12 cursors in one transaction on 12 one-page tables -> 12 pinned buffers,
  pinning_backends=1 each, buffers_pinned=12, so FOUR entries were in the overflow hash.
  PrivateRefCountOverflowed is not exposed in SQL -- the overflow is inferred from 12 > 8.
- *** THE SAME BACKEND SHOWED TWELVE PINS AND ZERO pg_locks ROWS ABOUT THEM (only relation
  AccessShareLocks). PINS ARE NOT LOCKS AND NO LOCK-MONITORING QUERY WILL EVER SEE THEM. ***
- MEASURED: a 37,500-page seq scan held ~5 pins at the sampled instant, 0 before and after. A scan
  pins one page at a time; that is what makes "pins may be held for long intervals" survivable.
- *** THE STRUCT NUMBERS, PRINTED BY COMPILED PROBE, NOT DERIVED: sizeof(LWLock) = 16,
  LWLOCK_PADDED_SIZE = PG_CACHE_LINE_SIZE = 128, sizeof(LWLockPadded) = 128;
  sizeof(BufferDesc) = 52, BufferDescPadded = 64, offsetof(state) = 24,
  offsetof(wait_backend_pgprocno) = 28, offsetof(content_lock) = 36.
  NUM_INDIVIDUAL_LWLOCKS = 48 (45 named locks, 3 retired slots 0/10/45), NUM_BUFFER_PARTITIONS 128,
  NUM_LOCK_PARTITIONS 16, NUM_PREDICATELOCK_PARTITIONS 16, NUM_FIXED_LWLOCKS = 208.
  OFFSETS: individual 0, buffer-mapping 48, lock-manager 176, predicate-lock 192.
  LWTRANCHE_BUFFER_CONTENT = 56, LWTRANCHE_FIRST_USER_DEFINED = 77, 29 builtin tranche names. ***
- *** THE SIZE ASYMMETRY, THE LESSON'S BEST STRUCTURAL POINT: 208 named LWLocks = 26,624 bytes
  (LWLockShmemSize adds sizeof(int)+128 = 26,756) for the WHOLE SERVER -- ProcArrayLock,
  WALWriteLock, all 128 buffer-mapping partitions -- while 16,384 buffer content locks cost
  16384*16 = 262,144 bytes, A QUARTER of the 1,048,576-byte "Buffer Descriptors" allocation lesson
  09 measured. AND THE PADDING GOES THE "WRONG" WAY: the 208 rare locks are each padded to a full
  128-byte cache line (112 bytes of padding apiece, lwlock.h's reason quoted), the 16,384 hot ones
  are packed TWO PER CACHE LINE at 64 bytes per descriptor, so buffers i and i^1 false-share.
  Padding them would cost 2 MB instead of 262 KB. NOT MEASURED -- see open items. ***
  TRAP: the main array is allocated with ShmemAlloc(), not ShmemInitStruct(), so it has NO ROW in
  pg_shmem_allocations -- it is inside the "<anonymous>" entry (4,730,240 B here).
- THE 32-BIT LWLock STATE WORD, all bits: LW_FLAG_HAS_WAITERS 1<<30, LW_FLAG_RELEASE_OK 1<<29,
  LW_FLAG_LOCKED 1<<28, LW_VAL_EXCLUSIVE 1<<24 = 16777216, LW_VAL_SHARED 1,
  LW_SHARED_MASK (1<<24)-1, LW_LOCK_MASK (1<<25)-1. The exclusive sentinel is ADDED TO THE SAME
  COUNTER the shared holders increment, guarded by StaticAssertDecl(LW_VAL_EXCLUSIVE >
  MAX_BACKENDS).
- *** A STALE COMMENT FOUND AND REPORTED: lwlock.c's comment above LW_SHARED_MASK says "Must be
  greater than MAX_BACKENDS - which is 2^23-1, so we're fine." MAX_BACKENDS IS 0x3FFFF = 2^18-1 IN
  THIS TREE. The assertion still holds (64x headroom, not 2x); postmaster.h mentions 2^23-1 as a
  DIFFERENT ceiling (inval.c's 3-byte backend id). THE CODE IS RIGHT AND THE COMMENT IS STALE BY
  ONE REFACTOR -- say so if any later lesson quotes it. ***
- LWLockAttemptLock: ONE CAS LOOP, NO SPINLOCK, never blocks, returns true meaning "you must wait".
  *** THE CAS IS ISSUED EVEN WHEN THE LOCK IS SEEN AS BUSY, swapping in the unchanged value,
  because "this doubles as a memory barrier" (quoted). *** Shared acquisition of an uncontended
  lock is a single successful CAS. HISTORY: commit ab5194e6f617a9a9e7aadb3dd1cee948a42d0755
  "Improve LWLock scalability", Andres Freund, 2014-12-25, reviewed by Amit Kapila and Robert Haas,
  +653/-302 across lwlock.c and lwlock.h, fetched from the pgsql-committers archive and quoted.
- THE FOUR-PHASE ACQUIRE and the race it closes (header quoted: "by the time we have finished
  queuing, the former locker very well might have already finished it's work"). Attempt, queue,
  attempt again, sleep, loop back to attempt. *** BEING WOKEN DOES NOT MEAN OWNING THE LOCK, and
  the refusal to hand off is argued from a 29-Dec-2001 discussion, quoted ("a process swap for
  every lock acquisition"). *** LWLockDequeueSelf is the cost of winning phase 3. extraWaits
  re-posts unrelated semaphore signals. *** LWLockReportWaitStart() IN PHASE 4 IS THE ONLY REASON
  ANY OF THIS IS SQL-VISIBLE: a lock taken in phase 1 or 3 is reported NOWHERE. ***
- THE WAIT-LIST MUTEX IS BIT 28 OF THE WORD IT GUARDS. pg_atomic_fetch_or_u32 to take it, then
  PLAIN NON-ATOMIC READS in the perform_spin_delay loop, then fetch_and to release. The fast paths
  never touch it; comment quoted ("even though the wait list is locked, non-conflicting lock
  operations may still happen concurrently").
- LWLockWakeup's policy: EITHER one exclusive waiter OR a contiguous run of shared waiters up to the
  first exclusive one, never both. LW_WS_PENDING_WAKEUP is set before the semaphore is posted so
  LWLockDequeueSelf needs no rescan. LW_FLAG_RELEASE_OK is cleared to stop wakeup storms
  ("Prevent additional wakeups until retryer gets to run"). LWLockQueueSelf puts
  LW_WAIT_UNTIL_FREE waiters on the HEAD (proclist_push_head) and everyone else on the TAIL.
  proclist_head is two ints holding head/tail PGPROCNO -- INDEXES, NOT POINTERS, which is why the
  list works across processes mapping shmem at different addresses.
- *** LWLOCKS HAVE NO DEADLOCK DETECTION, NO TIMEOUT, NO FAIRNESS COUNTER, NO ORDERING
  ENFORCEMENT. That absence is lesson 11's entire reason to exist. The only protection is
  convention in comments, e.g. the README's "they must be locked in partition-number order". ***
- held_lwlocks[MAX_SIMUL_LWLOCKS=200] (3,200 B), searched BACKWARDS on release ("Usually, but not
  always, it will be the latest-acquired lock"), overflow is elog(ERROR, "too many LWLocks taken")
  checked BEFORE HOLD_INTERRUPTS(). *** HOLD_INTERRUPTS() SPANS THE WHOLE HELD PERIOD, SO A BACKEND
  HOLDING AN LWLOCK CANNOT BE CANCELLED -- statement_timeout, pg_cancel_backend() and recovery
  conflicts all queue behind it. Reason quoted. This is the premise the §13 standby measurement
  depends on: what gets cancelled is the PIN, not the lock. *** LWLockReleaseAll() on the error path.
- CALL-SITE COUNTS, grepped: LWLockAcquire 505, LWLockConditionalAcquire 14, LockBuffer 365,
  LockBufferForCleanup 15, ConditionalLockBufferForCleanup 7, and *** LWLockAcquireOrWait = 3
  (ONLY xlog.c:2593, WALWriteLock) and LWLockWaitForVar = 5 (ONLY xlog.c:1503, WALInsertLocks).
  TWO ENTRY POINTS EXIST FOR ONE SUBSYSTEM EACH -- BOTH ARE LESSONS 22-23 AND WERE LEFT THERE. ***
- THE CLEANUP LOCK AS A PROTOCOL, not a mode: exclusive content lock, then LockBufHdr, then
  refcount == 1. *** THE WAITER QUEUE IS ONE int (wait_backend_pgprocno, offset 28) AND ONE FLAG
  (BM_PIN_COUNT_WAITER, bit 29), with elog(ERROR, "multiple backends attempting to wait for
  pincount 1") as the overflow handler. ITS SAFETY IS BORROWED FROM LESSON 11: the README argues
  from the relation-level ShareUpdateExclusiveLock that stops two VACUUMs overlapping. ***
  The handoff is in UnpinBuffer, signalled by THE LAST OTHER PIN-HOLDER, never by a timer, and the
  waiter keeps its own pin throughout -- hence CheckBufferIsPinnedOnce's elog(ERROR, "incorrect
  local pin count") if you hold two.
- heap_page_prune_opt's ladder RE-READ and the gate NAMED: step 1 RecoveryInProgress() -> return
  (SO A STANDBY NEVER PRUNES ON ITS OWN -- every cleanup lock there comes from replay), step 5 is
  ConditionalLockBufferForCleanup. *** THIS CLOSES LESSON 07'S "OPPORTUNISTIC": one cursor on the
  page and pruning is skipped with NO retry, NO queue and NO counter anywhere. *** Lesson 07's
  819-byte minfree arithmetic was cited, not re-derived.
- *** THE PRIMARY-SIDE HEADLINE. An aggressive VACUUM (FREEZE) of a ONE-PAGE, 40-TUPLE table spent
  22.03 OF ITS 22.03 SECONDS blocked on one cursor's pin. pg_stat_activity: wait_event_type =
  BufferPin, wait_event = BufferPin. pg_locks for that pid: TWO ROWS, BOTH GRANTED (virtualxid
  ExclusiveLock, relation ShareUpdateExclusiveLock) -- NO ungranted row, NOTHING for
  pg_blocking_pids() to find. VACUUM VERBOSE reported "1 page, 40 tuples, 30 hits, 0 misses,
  elapsed: 22.03 s" and HAS NO LINE FOR A CLEANUP-LOCK WAIT. Recipe: vacuum_freeze_min_age=0 +
  vacuum_freeze_table_age=0 forces aggressive, so lazy_scan_noprune returns false and
  lazy_scan_heap falls through to LockBufferForCleanup under Assert(vacrel->aggressive).
  NON-AGGRESSIVE VACUUM NEVER WAITS -- it takes a share lock, runs lazy_scan_noprune and leaves
  relfrozenxid alone. The pin costs you latency OR freezing progress, decided by a GUC. ***
- *** THE STANDBY HEADLINE, AND THE REASON THIS LESSON NEEDED TWO CLUSTERS. Only heap_xlog_prune
  (heapam.c:9345), btree_xlog_vacuum (nbtxlog.c:614) and 8 hash_xlog.c sites pass
  get_cleanup_lock = true to XLogReadBufferForRedoExtended. One FETCH 1 on the standby held
  Heap2/PRUNE redo at 0/3C059830 for 8000.427 ms -- within HALF A MILLISECOND of
  max_standby_streaming_delay = 8s -- logged verbatim as "recovery still waiting after 1000.161 ms"
  (deadlock_timeout, LOG ONLY, no cancel) then "recovery finished waiting after 8000.427 ms", with
  the CONTEXT line showing nredirected 20 / redirected [1->41 ... 20->60] / blk 0. The query died
  with 'canceling statement due to conflict with recovery / User was holding shared buffer pin for
  too long' and confl_bufferpin went 0 -> 1.
  SETUP THAT MAKES IT WORK, AND THE CONFOUND TO AVOID: the pin holder must take its snapshot AFTER
  the updates commit, or ResolveRecoveryConflictWithSnapshot fires first and you measure the
  horizon instead of the pin. Recipe: open a long-lived snapshot ON THE PRIMARY, do the UPDATEs
  under it (so nothing prunes early), COMMIT it, open the standby cursor, THEN VACUUM the primary.
  ALSO: the primary's own reads prune opportunistically, so do not SELECT the table between the
  commit and the VACUUM or there will be no prune record to replay. ***
  set_ps_display_suffix("waiting") and LogRecoveryConflict are BOTH inside if (InHotStandby), so the
  primary case has NO log line and NO ps suffix at any log_* setting -- confirmed by ps.
- *** WHAT ACTUALLY WAITS, 8,785 pg_stat_activity SAMPLES AT ~50 ms OVER THREE WORKLOADS, AND THE
  RESULT IS A NULL RESULT FOR THE LESSON'S OWN SUBJECT. 1,697 (19.3%) were on an LWLock, in only 7
  of the 45+29 available tranches.
    A) 16 clients, HOT UPDATEs on ONE heap page, 8,190 tps, 227,882 updates, 3,424 samples:
       WALWrite 1525 (44.5%), Lock|transactionid 823, ClientRead 501, CPU 245, IO|WALSync 189,
       Lock|tuple 111, LockManager 27, ProcArray 1. *** ZERO BufferContent. ***
    B) 293 MB table / 128 MB shared_buffers, random UPDATE, synchronous_commit=off, 21,258 tps,
       1,584 samples, 430,279 client-backend evictions + 403,185 writes in pg_stat_io:
       ClientRead 1357, CPU 185, IO|DataFileWrite 27, ProcArrayGroupUpdate 10, BufferMapping 2,
       IO|DataFileRead 2, ProcArray 1. *** STILL ZERO BufferContent. ***
    C) 32 clients, monotonic INSERT into a bigserial PK (rightmost btree leaf), 31,744 tps,
       3,777 samples: ClientRead 3444, CPU 185, *** BufferContent 100 *** (30% of non-idle,
       non-CPU), XactBuffer 32, XidGen 6, XactGroupUpdate 6, ProcArray 2, LockManager 1,
       IO|DataFileExtend 1.
  CONCLUSION, AND IT IS THE LESSON'S THESIS: content locks are held for tens of instructions, so a
  50 ms sampler cannot see them even with 16-way serialization on one page. THE LWLOCKS YOU CAN SEE
  WAITING ARE THE ONES HELD ACROSS AN I/O (WALWriteLock, or FlushBuffer's SHARED content lock
  during a write). A rightmost-page btree split is the one cheap way to make BufferContent appear.
  SAMPLER: a bash loop of `psql -At -c "select wait_event_type, wait_event from pg_stat_activity
  where backend_type='client backend'"` every 0.05 s into a world-readable file. ***
DELIBERATELY TREATED AS BLACK BOXES, and the next lessons own them: the heavyweight lock manager,
LOCKTAG, lock partitions and the deadlock detector (11); tuple locks and Lock|tuple /
Lock|transactionid, which DOMINATED workload A and were left unexplained on purpose (12);
pg_xact and XactBuffer (13); hint bits and rule 4 (15); freezing, relfrozenxid and why
lazy_scan_noprune cannot finish an aggressive page (18); VACUUM's two passes (19); WALInsertLocks,
XLogInsert, XLogFlush, group commit and the two exotic LWLock entry points (22-23);
BufferSync and BM_CHECKPOINT_NEEDED (24); the startup process, redo and
ResolveRecoveryConflictWithBufferPin's internals (25); walsender/walreceiver and the slot (26);
btree leaf-page pins and xs_want_itup (28-30).

NO SYLLABUS REVISION THIS RUN. Lesson 10's title was taught exactly as written -- buffer pins,
content locks and the LWLock implementation were all three taken in full -- plus the two deferrals
lesson 09's handover assigned (BM_PIN_COUNT_WAITER / wait_backend_pgprocno, and
pg_buffercache.pinning_backends / buffers_pinned as pin instruments, which lesson 09's Next Steps
item 6 asked for by name). Lesson 11's title needs no change: this lesson established exactly the
gap it fills (no deadlock detection, no timeout, no ordering enforcement in LWLocks) and already
borrowed ShareUpdateExclusiveLock from it, so the partitions/LOCKTAG/detector framing is right.

OPEN ITEMS HANDED FORWARD, in priority order:
(1) *** THE CONTENT-LOCK FALSE SHARING IS CLAIMED BUT NOT MEASURED. *** Two BufferDescs per
    128-byte cache line is arithmetic, not an experiment. Pin two backends to different cores and
    compare exclusive-content-lock throughput on buffers i / i^1 versus i / i+2.
(2) The 50 ms sampling floor. -DLWLOCK_STATS (sh_acquire_count, ex_acquire_count, block_count,
    dequeue_self_count, spin_delay_count, dumped at backend exit) or eBPF on the
    TRACE_POSTGRESQL_LWLOCK_* tracepoints would turn workload A's zero into a real block count and
    give the phase-3 win rate. Both need a rebuild; neither was attempted.
(3) A SKIPPED CLEANUP LOCK IS COUNTED NOWHERE -- same gap lesson 09 found in GetVictimBuffer's
    retry paths. Auditing all 14 LWLockConditionalAcquire sites for silent-failure accounting is a
    bounded task with a patch at the end of it.
(4) max_standby_streaming_delay = -1 plus hot_standby_feedback = on was NOT run: replay should
    stall unboundedly with confl_bufferpin staying 0. A June 2026 pgsql-hackers thread ("[PATCH]
    Prevent repeated deadlock-check signals in standby buffer pin waits") identifies that regime as
    untested upstream and is worth reading before lessons 25-26.
(5) Whether a btree INDEX-ONLY scan's leaf pin (kept when xs_want_itup is set) can stall
    btree_xlog_vacuum replay the same way. Bring this to lesson 30.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pintab1..pintab12, t_freeze, t_prune, t_std, t_std2, onepage, big1,
rightmost. NOTE: like lesson 08, this run did NOT use the pgNN_ prefix convention. Lesson 11 should
use pg11_*.
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
