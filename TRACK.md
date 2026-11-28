<!--
track-state
subject: PostgreSQL
mode: lesson
started: 2026-10-19
paused: 2026-10-30 (after lesson 10, to make JanusGraph the active track)
resumed: 2026-11-27 (JanusGraph track completed; capstone 2026-11-27)
lessons-total: 35
lessons-done: 11
resume-at: 12
paused-tracks: none
source-pin: Ubuntu source package postgresql-16 16.15-0ubuntu0.24.04.1, re-fetched and
  patch-checked per file on 2026-11-28 by the lesson 11 run. Lessons 07-11 all read this same
  release. Lessons 01-06 read upstream git trees: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd,
  04 @f52d4051, 05 @126976a8, 06 @0019a1ec. RE-VERIFY THE PATCH SERIES AT EACH PART BOUNDARY:
  as of 2026-11-28 there are 14 patches in debian/patches/series and the backend C files they
  touch are bufmgr.c, procsignal.c, postgres.c, pgstat_database.c, extension.c, dfmgr.c,
  guc_tables.c and llvmjit.c -- extract the +++/--- paths per file, never in bulk
previous-track: JanusGraph, 28 lessons 2026-10-30 .. 2026-11-26, capstone 2026-11-27; PR-candidate
  registers live on at tracks/janusgraph-registers.md (1-64, FROZEN) and
  tracks/janusgraph-registers-2.md (65-83, next free 84) and are NOT part of this track
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: active. Lesson 11 shipped 2026-11-28, the first entry after the 29-day pause, and the
next run takes lesson 12.**
The syllabus was written on 2026-10-19, the day the SQLite track's capstone shipped. The track was
paused on 2026-10-30 at Aryant's request so that JanusGraph could be the active subject; that track
ran 28 lessons and closed with its capstone on 2026-11-27, and **this file was restored from commit
`b15be247946f61b279b6fc0491fd9db5b8228b0b` (blob `334867d68ca1a206a738886ba23de59d1d8d38be`,
83,498 bytes) by that run, byte-for-byte verified before editing.** `tracks/postgresql.md` was
deleted in the same commit, as its own "How to resume" instructions required. Nothing in the
syllabus or the NOTE BLOCK below changed while the track was paused.

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

## Notes For The Next Run — READ THIS FIRST

**The next item is lesson 12 — tuple locks: the four strengths, the conflict table, and the
`LockTuple`/`XactLockTableWait` protocol.** It is the first unchecked item and nothing was skipped:
lessons 01-10 shipped 2026-10-20 to 2026-10-29, the track was paused for 29 days, and lesson 11
shipped 2026-11-28. All eleven are indexed in the root [`README.md`](README.md).

**Lesson 11 handed lesson 12 a loaded gun, and the NOTE BLOCK below says so in detail.** Three
things from it are operational:

- **Lesson 11 measured `Lock | transactionid` and `Lock | tuple` and deliberately did not explain
  them.** It proved that the `wait_event` string for a heavyweight lock *is* `LockTagTypeNames[]`
  indexed by the `LOCKTAG` type byte, which means lesson 10's workload A (823 samples on
  `Lock | transactionid`, 111 on `Lock | tuple`) was reading `locktag_type == 5` and `== 4`.
  **Lesson 12 owes the mechanism.** Lesson 11's exercise 4 produced a two-edge deadlock whose both
  edges were `LOCKTAG_TRANSACTION` from a pair of plain `UPDATE`s that never named a tuple lock —
  that is the thing to open.
- **`LOCKTAG_TUPLE` is a physical address** `(dboid, reloid, blocknum, offnum)`, so it stops naming
  the same logical row the instant the tuple moves. Lesson 11 stated this as lesson 12's problem
  statement and did not resolve it.
- **Do not re-teach `LOCKTAG`, the conflict table, the 16 partitions, the fast path or
  `deadlock.c`.** Lesson 11 took all five in full. Lesson 12's conflict table is the *tuple-lock*
  one (`HEAP_LOCK_*` / `MultiXactStatus`), which is a different table at a different granularity.
- **Use the `pg12_*` prefix for every throwaway table.** Lesson 11 used `pg11_*` throughout and the
  convention is back on track after lessons 08, 09 and 10 all lapsed.

**Source provenance.** The commit is now pinned in the front matter (`source-pin`) and was
re-fetched and patch-checked per file on 2026-11-28. `apt-get source postgresql-16` remains the
route; the recipe is in the NOTE BLOCK. Re-verify the patch series at each Part boundary — Part III
begins at lesson 13, so the next verification is due then.

**Entry size.** Lesson 11 shipped at 46 KB with a 14 KB digest. The real constraint on a run is
**output**, not input: every file pushed must be emitted in full as a tool argument, so large
pre-existing files (this one, the root `README.md`) should be edited **in place on local disk with
small scripted replacements**. The root `README.md` index is one row per line — **edit the row, do
not rebuild the table.** Lesson 11's run did exactly that and it works.

**Reading this file at all is now a two-step job.** At 92 KB, `get_file_contents` on `TRACK.md`
**exceeds the MCP tool-result token limit and fails**, saving the content to a local file instead.
Lesson 11's run recovered it with `python3 -I -c "json.load(...)"` on the saved path, extracting
`d[0]['text']` and stripping the `[Resource from github at ...] ` prefix, then read it with `sed -n`.
The root `README.md` (52 KB) fails the same way. **Budget for this: two tool calls plus a script,
not one call.**

**Daily Diff procedure — the 2026-11-27 rule held and was used in anger on 2026-11-28.** The front
page `https://tdd.cat/` is a **floor, never a ceiling**. Try `https://tdd.cat/{today}/`; then fetch
**both** the front page and `https://tdd.cat/archive/`; take the **maximum fetchable** edition
across all three, probing the slash form for any date either advertises. On 2026-11-28 both
`/2026-11-28/` and `/2026-11-27/` returned `client_error`, the front page named 11-26 (46 stories)
and `/archive/` agreed, marking 11-26 as Latest across 151 editions — so the maximum fetchable
edition was one the previous run had already digested, and the **fallback-pool rule** fired as
written: digest a whole uncovered edition before falling back to a single spent item.
**Editions covered to date: 11-08 through 11-26 inclusive — 11-23 was closed on 2026-11-28 and the
fallback pool is now EMPTY of whole editions.** 11-27 onward are uncovered and had not published as
of 03:4x UTC on 2026-11-28. The remaining unspent single items are from the 2026-11-06 edition (two)
and the 2026-11-18 edition (two). There is **no `WebFetch` tool in this environment**; pages are
fetched through a sub-agent, which must be asked for the structured item list (title, source,
outbound URL, one-line gist) rather than for page text, and whose summarised tail portion is not
reliable enough to quote — on 2026-11-28 the 11-23 tail summary claimed 31 rows, listed 29, and
duplicated one verbatim item. Always cross-check the item count against the page's own stated figure
and state how many items were actually recovered verbatim.

**Mermaid.** Render every diagram with `mmdc` before pushing and then inspect the rendered SVG: grep
it for a literal backslash-n (must be zero) and read the node labels back. Accumulated rules: quote
every node label containing parentheses, commas, `->`, `|` or angle brackets; **do not use numeric
HTML entities such as `&#40;` for parentheses — they render as literal `&(`** (found 2026-11-27);
use named entities such as `&lt;`, `&gt;` and `&amp;` for comparisons and ampersands, which
rendered correctly on 2026-11-28 in both diagram 1 (`MyProc-&gt;fpInfoLock`, `nLocks &gt; 0`) and
diagram 2 (`nWaitOrders &gt; 0 ?`); never escape a quote inside a label, drop the inner quotes
instead; a semicolon inside a `sequenceDiagram` message is a statement separator, so use a comma;
and a `flowchart` edge label takes a quoted string with `<br/>` in it and renders multi-line
correctly. A `subgraph` whose node labels contain `[16]` array subscripts renders fine as long as
the whole label is quoted.

**Sub-agents.** State the prohibition verbatim in every sub-agent prompt — naming `curl`, `wget`,
`gh`, `git` and `raw.githubusercontent.com`, and instructing the sub-agent to report failure rather
than find another route. It was obeyed and reported on 2026-11-26, 2026-11-27 and again on
2026-11-28 (three sub-agents, all three reported WebFetch-only compliance explicitly).

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
- [x] 11 — The heavyweight lock manager: `LOCKTAG`, lock partitions, and the deadlock detector
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

NOTE-BLOCK ROTATION DONE THIS RUN (2026-11-28, the lesson 11 run): lesson 08's note was dropped and
lesson 11's was added. TRACK-ARCHIVE.md gained the row "PG 08" pointing at commit
0fe7e863679c97fa83c2cc7ce35226b40c87ff6c / blob 458173fa9a1255ce9f3ce6232eec4ae5cffa6da2, which is
the last state of TRACK.md carrying it, and that row was added FIRST, as the policy above requires.
This block now holds 09, 10 and 11 -- exactly the three the policy allows. WHEN LESSON 12 SHIPS IT
MUST DROP THE LESSON 09 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

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
  (C) LESSON 09'S, AND TRY THIS FIRST -- CONFIRMED AGAIN BY LESSON 11 ON 2026-11-28, so this is
      now four runs in a row and should be treated as the default: postgresql-16 was ALREADY
      INSTALLED in the sandbox image,
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
  pg7_* (07), 08's unprefixed set (seg, ext_*, tiny_copy, q1..q4, dedup, b, io_t, iostat_t),
  09's unprefixed set (t_small, t_mid, t_big, t_copy, t_tiny, in database `bufdemo`), 10's
  unprefixed set (pintab1..pintab12, t_freeze, t_prune, t_std, t_std2, onepage, big1, rightmost)
  -- three lapses in a row -- and then pg11_* (11, in database `pg11lab`), which put the
  convention back. *** CONVENTION RESTORED AS OF LESSON 11. KEEP IT: pg12_*, pg13_*, ... ***
- *** THE COMPILED-PROBE TRAP IS WORSE THAN LESSON 10's NOTE SAYS. postgres.h redefines
  fprintf TO pg_fprintf AS WELL AS printf, and pg_fprintf is in no library you link, so a probe
  that only does `#undef printf` fails at LINK time with `undefined reference to pg_fprintf`.
  Lesson 11's working preamble: #include "postgres.h", then #undef printf, #undef fprintf,
  #undef sprintf, #undef snprintf, #undef vsnprintf, then #include <stdio.h>. Build with
  `gcc -I $(pg_config --includedir-server) -o probe probe.c`. No -l flags needed for pure
  sizeof/offsetof/macro values. ***
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

-- LESSON 11 (2026-11-28) AS TAUGHT ------------------------------------------------------------
Document: 2026/11/2026-11-28-postgres-heavyweight-lock-manager/README.md (single README, 46 KB, not
split -- LOCKTAG, the conflict table, the partitions, the fast path and deadlock.c are one vertical
slice and the fast path only makes sense as an escape from the partitions, which only make sense as
an escape from a single LockMgrLock).

SOURCE PROVENANCE: `apt-get source postgresql-16` for the fifth run in a row, version
16.15-0ubuntu0.24.04.1, the same build the cluster runs; deb-src had to be re-enabled again
(`sed -i 's/^Types: deb$/Types: deb deb-src/' /etc/apt/sources.list.d/ubuntu.sources`, then
apt-get update). PATCH CHECK DONE PER FILE AND THE SERIES IS NOW 14, NOT 15: storage/lmgr/lock.c,
deadlock.c, lmgr.c, proc.c, include/storage/{lock,proc,lwlock,lockdefs}.h, storage/lmgr/README and
utils/adt/lockfuncs.c are ALL UNTOUCHED, so every line quoted is upstream verbatim. The backend C
files the series does touch: bufmgr.c + procsignal.c + tcop/postgres.c + include/storage/procsignal.h
+ include/tcop/tcopprot.h (v8-0001-Fix-recovery-conflict-SIGUSR1-handling.patch),
utils/activity/pgstat_database.c (pgstat-report-conflicts-immediately.patch), and
commands/extension.c + utils/fmgr/dfmgr.c + utils/misc/guc_tables.c + postgresql.conf.sample +
include/utils/guc.h (extension_destdir), plus jit/llvm/llvmjit.c (jit-s390x). MD5s of the files read
in full, for a future byte-level re-verification: lock.c a9671b2431ba1b14538f33a22e276623,
deadlock.c 341dcbeec83c590ae2f7c53175445609, lock.h 6556dada0739c8dc394a74a8f58d22a8,
lmgr/README bc707a5b4f3dffd56395da2088ac6870.
Files read IN FULL: storage/lmgr/README (731 lines), include/storage/lock.h (625 lines),
include/storage/lockdefs.h (61 lines). Read substantially: storage/lmgr/deadlock.c (1159 lines --
every function named in the lesson). Read in part: storage/lmgr/lock.c (4695 lines: LockConflicts,
lock_mode_names, the lock-method tables, NLOCKENTS, LockShmemSize, the FAST_PATH_* macros,
EligibleForRelationFastPath / ConflictsWithRelationFastPath, LockAcquireExtended's fast-path branch
at 925-1010, FastPathGrantRelationLock, FastPathTransferRelationLocks, proclock_hash at 541),
storage/lmgr/proc.c (1930 lines: ProcSleep's rule-2 loop at 1100-1175, the timer setup at 1209-1241,
the wait loop at 1280-1345, CheckDeadLock at 1726-1800, CheckDeadLockAlert),
utils/adt/lockfuncs.c (LockTagTypeNames and the pg_locks column mapping at 236-261),
utils/activity/wait_event.c (the PG_WAIT_LOCK cases at 90 and 147), storage/lmgr/lmgr.c:1307,
include/utils/wait_event.h (PG_WAIT_LOCK = 0x03000000).

*** THIS RUN HAD A CLUSTER AND EVERY number is measured. *** PostgreSQL 16.15
(Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, Linux 6.18, ext4, 2 vCPU / 8 GB RAM, ~31 GB
free. shared_buffers 128 MB = 16,384 buffers, max_connections 100, deadlock_timeout 1000 ms,
max_locks_per_transaction 64, max_prepared_transactions 0, block_size 8192, log_lock_waits turned
ON for the deadlock measurements. Extensions used: pageinspect, pg_buffercache, pgstattuple (only
pg_locks / pg_stat_activity / pg_shmem_allocations were actually needed -- NO extension is required
for this lesson, which is a first in the track). Database `pg11lab`.
ENVIRONMENT FACTS, ALL REUSABLE:
(a) Recipe C again, fourth run in a row: pg_lsclusters showed "16 main" present and DOWN,
    `pg_ctlcluster 16 main start` was the only setup step. ALTER SYSTEM + pg_reload_conf() is
    enough for log_lock_waits (it is PGC_SUPERUSER, not PGC_POSTMASTER -- no restart needed).
(b) *** THE TWO-FIFO HARNESS IS THE RIGHT INSTRUMENT FOR ANYTHING INVOLVING TWO BACKENDS AND AN
    EXACT INTERLEAVING, AND IT IS SIMPLER THAN LESSON 08'S strace VERSION BECAUSE NO PID IS NEEDED:
    mkfifo f1 f2; chmod 777 f1 f2; start `psql -X -q -At -d DB < f1 > o1 &` and the same for f2;
    then `exec 3>f1; exec 4>f2` and write statements into &3 / &4 with sleeps between. Put
    `\timing on` down both pipes FIRST -- psql's own \timing is the measurement instrument and it
    is accurate to the microsecond. Everything (fifos, output files, SQL scripts) must live in
    /var/tmp/lab chmod 777; the scratchpad path is NOT readable by the postgres user. Close with
    `exec 3>&-; exec 4>&-`. ***
(c) `LOCK TABLE` OUTSIDE A TRANSACTION BLOCK IS AN ERROR ("LOCK TABLE can only be used in
    transaction blocks") and a backgrounded `psql -c "LOCK TABLE ..."` therefore exits 1 silently
    if you redirect stderr. Wrap it: BEGIN; LOCK TABLE ...; COMMIT; in a -f script.
(d) /usr/bin/time still does not exist (lesson 09's note holds); psql \timing is the substitute.

TOOK, and must not be re-taught:
- *** LOCKTAG IS 16 BYTES WITH NO PADDING AND THAT IS CAUSAL, NOT COSMETIC: it is the hash key, and
  lmgr/README says a padding byte would make "the hash computation will be random". MEASURED:
  sizeof(LOCKTAG) = 16, field4 at 12, locktag_type at 14, locktag_lockmethodid at 15. The header's
  own phrasing -- "defined with malice aforethought" -- was quoted. ***
- *** THE SINGLE BEST FINDING OF THE LESSON, AND IT CLOSES A LESSON 10 LOOSE END: LockTagTypeNames[]
  (lockfuncs.c:29, 12 entries, StaticAssertDecl'd against LOCKTAG_LAST_TYPE+1) feeds BOTH
  pg_locks.locktype (lockfuncs.c:236) AND pg_stat_activity.wait_event for a Lock wait. proc.c:1328
  waits with event id `PG_WAIT_LOCK | locallock->tag.lock.locktag_type` = 0x03000000 | the tag type
  BYTE; wait_event.c:148 resolves it with GetLockNameFromTagType(); lmgr.c:1307 is
  `return LockTagTypeNames[locktag_type];`. SO THE WAIT EVENT NAME FOR A HEAVYWEIGHT LOCK IS THE
  LOW BYTE OF THE LOCKTAG. Lesson 10's workload A (823 Lock|transactionid, 111 Lock|tuple) was
  reading locktag_type 5 and 4. Confirmed live: a backend blocked on LOCK TABLE ACCESS EXCLUSIVE
  reported wait_event_type=Lock, wait_event=relation. ***
- The full 12-type LOCKTAG table with each type's field1..field4 meaning, tabulated from the
  SET_LOCKTAG_* macros. LOCKTAG_TUPLE is (dboid, reloid, blocknum, offnum) -- A PHYSICAL ADDRESS,
  explicitly flagged as lesson 12's problem statement and NOT resolved.
- *** THE CONFLICT TABLE IS SYMMETRIC BUT NOT MONOTONE, AND THIS IS THE LESSON'S BEST STRUCTURAL
  POINT. All 64 pairs checked programmatically: ZERO asymmetric pairs. Masks:
  AccessShare 0x100, RowShare 0x180, RowExclusive 0x1e0, ShareUpdateExclusive 0x1f0, Share 0x1d8,
  ShareRowExclusive 0x1f8, Exclusive 0x1fc, AccessExclusive 0x1fe. BUT mode 4
  (ShareUpdateExclusive) SELF-CONFLICTS and permits mode 3 (RowExclusive), while mode 5 (Share)
  does NOT self-conflict and BLOCKS mode 3. NEITHER MASK CONTAINS THE OTHER -- modes 4 and 5 are
  genuinely INCOMPARABLE, there is no total order, and that is why conflictTab exists instead of a
  >= comparison. *** THIS DISCHARGES LESSON 10'S IOU: VACUUM's "only one at a time, concurrently
  with writes" guarantee is literally the X at (4,4) in that matrix and nothing else. ***
  COROLLARY WORTH CARRYING: EligibleForRelationFastPath uses `(mode) < ShareUpdateExclusiveLock`,
  a NUMERIC comparison on a non-totally-ordered set, correct only because modes 1-3 happen to be
  the three weak DML modes. Mode 4 is in neither the eligible nor the conflicting set.
- DEFAULT_LOCKMETHOD and USER_LOCKMETHOD point at the SAME LockConflicts array and the same
  lock_mode_names; only the trace flag differs. MAX_LOCKMODES 10, MaxLockMode 8, sizeof(LOCKMASK) 4
  so there is room for 31.
- THE THREE REPRESENTATIONS OF ONE LOCK AS A COST LADDER: LOCALLOCK hit (no shared memory touched
  AT ALL, nLocks++ and return) -> fast path -> LOCK+PROCLOCK in the partitioned hash. LOCALLOCK's
  "when nLocks == 0 they should be considered garbage" comment quoted.
- *** THE STRUCT NUMBERS, PRINTED BY COMPILED PROBE, NOT DERIVED: sizeof(LOCKTAG) 16,
  sizeof(LOCK) 152, sizeof(PROCLOCK) 64, sizeof(PROCLOCKTAG) 16, sizeof(LOCALLOCK) 72,
  sizeof(LOCALLOCKTAG) 20, sizeof(PGPROC) 880. LOCK offsets: tag 0, grantMask 16, waitMask 20,
  procLocks 24, waitProcs 40, requested[] 64, nRequested 104, granted[] 108, nGranted 148.
  PGPROC offsets: myProcLocks[] 184, fpLockBits 760, fpRelId[16] 768, fpVXIDLock 832.
  NUM_LOCK_PARTITIONS 16, LOG2_NUM_LOCK_PARTITIONS 4, LOCK_MANAGER_LWLOCK_OFFSET 176 (matches
  lesson 10's lwlock.h figures exactly), FP_LOCK_SLOTS_PER_BACKEND 16, LOCKTAG_LAST_TYPE 11. ***
- proclock_hash's trick: `lockhash ^= ((uint32) procptr) << LOG2_NUM_LOCK_PARTITIONS;` (lock.c:541)
  -- shifting the proc pointer left by 4 leaves the low 4 bits, i.e. the partition, untouched, so a
  PROCLOCK always lands in its LOCK's partition. PGPROC.myProcLocks is itself per-partition
  (dlist_head[16] at offset 184) and README says why: fast-path transfer requires one backend to
  manipulate another's PROCLOCK lists.
- The partition-number-order rule, and README's refusal to optimise the detector ("trying to make
  this work does not seem a productive use of effort") quoted. CheckDeadLock releases in REVERSE
  order for two stated reasons, the second being "(2) This avoids O(N^2) behavior inside
  LWLockRelease" -- WHICH IS LESSON 10'S held_lwlocks[] BACKWARD SEARCH SURFACING TWO LAYERS UP.
- *** THE FAST PATH, MEASURED THREE WAYS. (1) SLOT EXHAUSTION IS A HARD CLIFF: 20 one-page tables
  read in one transaction gave EXACTLY fastpath=t 16 / fastpath=f 4. (2) PARTITIONING WALKS OFF IT:
  one `SELECT count(*) FROM pg11_part WHERE v = 0` on a 64-partition RANGE table took 66 relation
  AccessShareLocks, 16 fastpath and 50 through the shared hash, plus 1 fastpath virtualxid.
  (3) THE TRANSFER IS OBSERVABLE: backend A (pid 927) held a fastpath AccessShareLock on pg11_fp1
  and read fastpath=t; three seconds after backend B queued for ACCESS EXCLUSIVE, A's SAME ROW read
  fastpath=f WHILE A SAT INSIDE pg_sleep() DOING NOTHING. That is FastPathTransferRelationLocks
  running inside the strong locker and rewriting a stranger's PGPROC -- the only place in the server
  that does this. pg_blocking_pids() correctly returned {927}. ***
- FastPathStrongRelationLocks MEASURED IN SHARED MEMORY: pg_shmem_allocations row "Fast Path Strong
  Relation Lock Data", size 4100, allocated_size 4224 = 1024 uint32 counters + one 4-byte slock_t,
  confirming FAST_PATH_STRONG_LOCK_HASH_PARTITIONS = 1024 WITHOUT a probe. A 1024-way coarse
  partitioning in front of the 16-way fine one, read with NO LOCK. The memory-ordering argument was
  quoted in full ("either the value is truly zero, or if it is a stale value, the strong locker has
  yet to acquire the per-backend LWLock we now hold") -- and it RESTS ON LESSON 10'S OBSERVATION
  that LWLockAcquire doubles as a memory barrier, which there was only a comment justifying a
  redundant CAS and here is load-bearing for a lock-free read.
  TRAP: "LOCK hash" and "PROCLOCK hash" are only 2896 B each in pg_shmem_allocations -- that is the
  dynahash HEADER; the elements live in the 6,651,776-byte "<anonymous>" entry.
- *** TWO DEADLOCK PATHS EXIST AND THEY DIFFER BY ~3,800x. THIS IS THE HEADLINE.
  (A) EARLY DEADLOCK, in ProcSleep, NOT deadlock.c. Two sessions each took ShareLock on pg11_dl,
      then each requested ExclusiveLock. The second one's error came back in 0.265 ms:
      "ERROR: deadlock detected / DETAIL: Process 997 waits for ExclusiveLock on relation 16512 of
      database 16384; blocked by process 998. Process 998 waits for ... blocked by process 997."
      deadlock_timeout is 1000 ms and deadlock.c NEVER RAN. The mechanism is proc.c:1104-1156's
      two-question loop -- "Must he wait for me?" (conflictTab[proc->waitLockMode] & myHeldLocks)
      then "Must I wait for him?" (conflictTab[lockmode] & proc->heldLocks) -- which finds a
      two-node cycle with TWO BITMASK ANDS and calls RememberSimpleDeadLock(), setting
      early_deadlock. THE MOST COMMON REAL-WORLD DEADLOCK (the lock upgrade) NEVER TOUCHES THE
      DETECTOR. The DETAIL prints the relation by RAW OID because RememberSimpleDeadLock fills
      deadlockDetails[] from the tag with no catalog lookup.
  (B) THE TIMEOUT PATH. Two transactions, cross-ordered UPDATEs on pg11_dl. Victim errored at
      1000.736 ms client-side / 1000.182 ms in the log -- within 0.74 ms of deadlock_timeout. ***
- *** AND THE PART THAT MAKES (B) WORTH THE LOG LINES: THE NON-VICTIM RAN A FULL DEADLOCK CHECK AND
  FOUND NOTHING. Verbatim from the log: 03:41:16.546 [970] "process 970 still waiting for ShareLock
  on transaction 769 after 1000.221 ms" / DETAIL "Process holding the lock: 971. Wait queue: 970."
  -- then 03:41:17.550 [971] "process 971 detected deadlock while waiting for ShareLock on
  transaction 768 after 1000.182 ms" and the ERROR. 970's timer fired BEFORE 971 began waiting, so
  the cycle did not exist yet and its check was wasted: 16 exclusive partition LWLock acquisitions
  and a graph walk for nothing. THIS IS README'S MISC NOTE 1 EXECUTING ("the last process in the
  cycle to wait ... is certain to detect and resolve the cycle"). 970 then waited 2004.709 ms total
  -- one second for its own useless check, one for the victim's. BOTH EDGES WERE
  LOCKTAG_TRANSACTION from plain UPDATEs that never named a tuple lock: LESSON 12'S OPENING. ***
- deadlock.c's structure taken in full: DeadLockCheckRecurse pushes one soft-edge reversal onto
  curConstraints[] / recurses / pops; TestConfiguration calls ExpandConstraints then TopoSort per
  queue, then FindLockCycle from every constraint's waiter AND blocker AND startProc LAST ("because
  if it has a soft cycle still to be dealt with, we want to deal with that first"); FindLockCycle
  believes waitOrders[] over reality; hard edges come from proclock->holdMask & conflictMask, soft
  edges from queue position. TopoSort is DELIBERATELY the wrong algorithm and says so ("a lot
  simpler and slower than ... Knuth's Volume 1. However, Knuth's method doesn't try to minimize the
  damage to the existing order") -- it scans BACKWARDS and emits the last proc with
  beforeConstraints == 0. A SORT FAILURE IS INFORMATION, not an error: it means the newest reversal
  contradicts an earlier one, and README says detecting it is what stops the search looping.
- *** THE DETECTOR'S WORKSPACE IS ~31 KB PER BACKEND, palloc'd ONCE AT STARTUP INTO TopMemoryContext,
  for a reason quoted verbatim: "(a) the deadlock checker might be invoked when there's no free
  memory left, and (b) the checker is normally run inside a signal handler, which is a very
  dangerous place to invoke palloc from." With MaxBackends = 122 (100 + 3 autovacuum + 1 + 8
  max_worker_processes + 10 max_wal_senders) and sizeof(EDGE) = 40: visitedProcs/topoProcs 976 B
  (ALIASED -- topoProcs = visitedProcs, "it needn't run concurrently with FindLockCycle"),
  deadlockDetails ~2928, beforeConstraints 488, afterConstraints 488, waitOrders ~1464,
  waitOrderProcs 976, curConstraints 4880, possibleConstraints 4*MaxBackends*40 = 19520. ***
  maxCurConstraints = MaxBackends DOUBLES AS THE RECURSION LIMIT ("Making it really big might
  potentially allow a stack-overflow problem"). AND THE HONEST COMMENT, quoted: "Is this enough?
  In practice it seems it should be, but I don't quite see how to prove it. If we run out, we might
  fail to find a workable wait queue rearrangement even though one exists."
- *** TestConfiguration RETURNS -1 FOR BOTH A HARD DEADLOCK AND A SELF-INCONSISTENT OR
  OUT-OF-WORKSPACE CONSTRAINT SET, SO A RESOURCE SHORTFALL IS REPORTED AS A DEADLOCK AND NOTHING
  COUNTS IT. A transaction can be aborted for a rescuable soft deadlock. Handed forward as open
  item (3). ***
- *** max_locks_per_transaction IS A POOL-SIZING GUC, NOT A PER-TRANSACTION CAP, AND THIS WAS
  MEASURED BY BISECTION. At max_locks_per_transaction = 64, ONE transaction took 5,000 advisory
  locks (pg_advisory_xact_lock over generate_series) and reported 5,000 rows in pg_locks. 7,800 /
  7,808 / 8,000 / 8,500 / 9,000 / 10,000 all succeeded. Bisecting 10,000..40,000 put the wall
  between 12,812 (success) and 12,929 (failure) -- about 1.64x NLOCKENTS() = max_locks_per_xact *
  (MaxBackends + max_prepared_xacts) = 64 * 122 = 7,808, the excess coming from LockShmemSize's
  stated 10% safety margin plus dynahash slack. Failure is "ERROR: out of shared memory / HINT: You
  might need to increase max_locks_per_transaction." ***
- THE RELATION EXTENSION LOCK IS EXCLUDED FROM THE GRAPH BY FIAT: FindLockCycleRecurseMember returns
  false immediately for LOCKTAG_RELATION_EXTEND, justified by an invariant enforced only by an
  Assert in LockAcquireExtended (the file-static IsRelationExtensionLockHeld, which is
  PG_USED_FOR_ASSERTS_ONLY). In a non-assert build a path that took a heavyweight lock while holding
  an extension lock would create a cycle the detector refuses to look at.
- Tom Lane's 2001-01-19 pgsql-hackers message "Deadlock reimplementation notes (kinda long)" was
  FETCHED THIS RUN and is the design document that lmgr/README's deadlock section was committed
  from; the cover text says so outright. Quoted: "it's better to spend some time to guarantee that
  we've checked all possible escape routes".

*** THE NULL RESULT, AND IT CONFIRMS LESSON 10 FROM THE OTHER SIDE. Two workloads on the
64-partition table, max_parallel irrelevant here:
  (A) 16 clients, prepared `SELECT count(*) FROM pg11_part WHERE id = :k` with k in -5..-1 so
      runtime pruning scans NOTHING but all 66 locks are still taken: 61,272 tps, 0.261 ms mean
      latency, 1,225,086 transactions in 20 s, i.e. ~4 MILLION lock acquisitions/sec of which ~3M
      went through the 16 partition LWLocks. A 40 ms pg_stat_activity sampler caught ZERO
      LWLock|LockManager in 36 active samples -- THE SAMPLER IS ~150x SLOWER THAN THE TRANSACTION
      IT IS TRYING TO OBSERVE, and only 36 of 600 sampling attempts caught any active backend.
  (B) 16 clients, `SELECT count(*) FROM pg11_part WHERE v = 0` (real seq scans of all 64
      partitions): 347 tps, 46.1 ms mean latency, and LWLock|LockManager DID appear -- 9 of 159
      non-idle samples, alongside 150 Client|ClientRead.
  CONCLUSION, AND IT IS THE RIGHT ONE TO CARRY FORWARD: partition LWLocks are held for tens of
  instructions, exactly as lesson 10 concluded for content locks, SO THE WORKLOAD WHERE LOCK
  MANAGER CONTENTION MATTERS MOST IS THE ONE WHERE SAMPLING CAN SEE IT LEAST. Anyone sizing
  NUM_LOCK_PARTITIONS from pg_stat_activity is reading an instrument that cannot resolve the
  signal. A LOWER-OVERHEAD SAMPLER IS NOW THE TRACK'S BIGGEST INSTRUMENTATION GAP (see open item 2;
  this is the same gap lesson 10 logged as its own item 2, now independently confirmed). ***

NO SYLLABUS REVISION THIS RUN. Lesson 11's title was taught exactly as written -- LOCKTAG, lock
partitions and the deadlock detector were all three taken in full -- plus the two things lesson 10's
handover assigned by name (the ShareUpdateExclusiveLock argument it had borrowed, and the
LWLock-absence gap it had identified). Lesson 12's title needs no change: this lesson established
exactly the gap it fills (LOCKTAG_TUPLE is a physical address; Lock|transactionid and Lock|tuple
were measured and left unexplained; exercise 4's deadlock ran entirely on LOCKTAG_TRANSACTION
edges), so the strengths/conflict-table/LockTuple framing is right. Part III still begins at lesson
13 and the Part II/III boundary is where the next patch-series re-verification is due.

OPEN ITEMS HANDED FORWARD, in priority order:
(1) *** CONFIRM THE PARTITION ASSIGNMENT IS CONTENT-ADDRESSED. LockHashPartition is the low 4 bits
    of the dynahash hash of a 16-byte tag, so a lock's partition depends on WHAT is locked, not WHO
    locks it. Build 16 relations whose OIDs land in distinct partitions and check whether
    LWLock|LockManager tracks tag distribution rather than client count. Needs a compiled probe
    calling LockTagHashCode -- postgresql-server-dev-16 makes that routine (see the probe-trap note
    in the environment section). ***
(2) THE 40 ms SAMPLING FLOOR, NOW BLOCKING TWO LESSONS. Same gap lesson 10 logged. -DLWLOCK_STATS
    or eBPF on TRACE_POSTGRESQL_LWLOCK_* would turn workload (A)'s zero into a real block count;
    both need a rebuild and neither was attempted. THIS IS THE SINGLE HIGHEST-VALUE
    INFRASTRUCTURE INVESTMENT AVAILABLE TO THIS TRACK and lessons 22-23 will need it too.
(3) A WASTED DeadLockCheck IS COUNTED NOWHERE -- exercise 4's non-victim took 16 exclusive LWLocks
    and walked the graph for nothing. This is now the THIRD instance of the same accounting gap:
    lesson 09's GetVictimBuffer retries, lesson 10's skipped cleanup locks, and this. THREE IS A
    PATTERN AND ARGUABLY A PATCH -- a pg_stat_database or pg_stat_activity counter for
    deadlock-checks-run vs deadlocks-found would be a bounded, defensible first contribution.
(4) FORCE A GENUINE DS_SOFT_DEADLOCK AND CATCH THE REORDERING. Exercise 3 fired early_deadlock in
    ProcSleep instead, so the wait-queue rearrangement path -- the whole TopoSort/ExpandConstraints
    machine -- WAS NEVER OBSERVED EXECUTING, only read. Needs three or more waiters in one queue
    with a reorderable conflict pattern. log_lock_waits will NOT show it; DEBUG_DEADLOCK and
    PrintLockQueue would, and that needs a rebuild (folds into item 2).
(5) MEASURE FastPathTransferRelationLocks AS A FUNCTION OF max_connections. It scans every backend's
    array under every backend's fpInfoLock, so at max_connections = 1000 one ALTER TABLE should take
    ~1000 LWLocks. Time one LOCK TABLE ACCESS EXCLUSIVE at 100 vs 1000 connections.
(6) IS maxPossibleConstraints EXHAUSTION REACHABLE? The source says the proof is unknown. Either
    construct a configuration that hits it or demonstrate the bound is safe.
(7) For lesson 15: rule 4 of storage/buffer/README (hint bits need only a shared content lock) is
    STILL unspent -- lesson 10 left it, lesson 11 did not touch it.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run), all correctly prefixed: pg11_fp1..pg11_fp20, pg11_dl, pg11_part and
pg11_part_0..pg11_part_63, in database `pg11lab`. Lesson 12 should use pg12_*.
-->

## Completed Subjects

### JanusGraph Codebase — 28 lessons, 2026-10-30 to 2026-11-26 (capstone 2026-11-27)

Bottom-up deep dive through the JanusGraph source at pinned commit `a59ae8d8` (2026-10-28), in six
parts: the storage contract (01-07), graph ⇄ bytes (08-13), the write path (14-18), the read path
(19-22), Gremlin integration (23-24), and schema, mixed indexes and production backends (25-28).
Sessions 01-04 ran in `contributor-reading` mode and shipped with their `## My Notes` half as the
untouched template; the mode switched to `lesson` on 2026-11-03 and the 24 carried questions were
settled as the syllabus reached them. **Sessions 01-04's `## My Notes` remain Aryant's to fill at
any time and must never be edited by a run.** Capstone:
[`2026/11/2026-11-27-janusgraph-capstone/README.md`](2026/11/2026-11-27-janusgraph-capstone/README.md).

The full 28-item syllabus, the session log, and the `Verified by lesson 26/27/28` fact blocks are
preserved in this file's git history, in the commit immediately preceding the 2026-11-27 capstone
commit. The lesson documents themselves are indexed in the root [`README.md`](README.md), which is
the real record of what was taught. **The track's actual deliverable is the PR-candidate register**,
which was deliberately not archived and stays live at
[`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md) (candidates 1-64, frozen, plus
the "Still open, and who owns it" ledger) and
[`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md) (65-83, next free 84).

**Closing summary.** A property graph folded onto one sorted row per vertex, where every graph
concept is an encoding decision and the only distributed-systems machinery exists because a mixed
index commits *after* storage. Atomicity is **borrowed**, once: `PRIMARY_SUCCESS` is written inside
the storage mutation so it can serve as evidence that the storage write landed — and on default CQL
that loan does not clear, because the log is a different store in a different partition written by
a different unlogged batch. Repair is therefore **re-derivation from the edgestore, never a replay
of the log**, which is what makes it idempotent, convergent, order-free and correct even when the
log entry is wrong. The track's best transferable idea is the three-way idempotence taxonomy —
assignment, accumulation, delivery — whose dividing line has nothing to do with distribution. And
its honest qualification: **28 lessons, 20 outstanding experiments, and not one run ever executed
JanusGraph.**

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
