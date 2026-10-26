<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 7
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 07 shipped 2026-10-26.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 08**.

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

-- LESSON 05 (2026-10-24) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-24-postgres-free-space-map/README.md (single README, not split -- one
mechanism, no sub-topic deserved its own file). Source commit read: postgres/postgres@126976a8
(master) PLUS REL_16_STABLE for src/include/storage/freespace.h. Trees so far: 01 @ddd468ca,
02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051, 05 @126976a8. Keep stating the commit.
Files read IN FULL: storage/freespace/README, fsmpage.c, freespace.c, storage/fsm_internals.h,
storage/freespace.h.

*** THIS RUN HAD A CLUSTER, and EVERY number in the lesson is measured. Same recipe as 03/04. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
data_checksums off, pageinspect 1.12, pg_freespacemap 1.2, pgstattuple 1.5. The two traps still
apply (initdb not as root; data dir not under the scratchpad path -- /var/tmp works).

*** CORRECTION TO LESSON 04'S HANDOVER, important: HEAP_FSM_CREATION_THRESHOLD AND FSM_LOCAL_MAP
DO NOT EXIST. *** Not in master @126976a8, not in REL_16_STABLE (freespace.h is byte-identical
between the two branches), and a repo-wide code search returns zero hits. The feature was
b0eaa4c51b "Avoid creation of the free space map for small heap relations" and it was REVERTED
WHOLESALE by 7db0cde6b58eef2ba0c70437324cbc7622230320 on 2019-05-07 along with six follow-ups
(06c8a5090e, 13e8643bfc, 6f918159a9, 9c32e4c350, 29d108cdec, 08ecdfe7e5). The revert message says
the process-local map "was reset whenever a block with enough free space was found" and that a
per-relation relcache map would be better; that redesign was never landed. SO NO RELEASED
POSTGRESQL HAS A SMALL-TABLE FSM EXEMPTION. Measured confirmation: the FSM fork appears at TWO
heap pages, not five. Do not propagate the 4-page threshold claim.

TOOK, and must not be re-taught:
- THE CATEGORY. FSM_CATEGORIES 256, FSM_CAT_STEP = BLCKSZ/256 = 32, MaxFSMRequestSize =
  MaxHeapTupleSize = 8160. fsm_space_avail_to_cat FLOORS (and clamps to 254 below 8160, returning
  255 only at >= 8160); fsm_space_needed_to_cat CEILS ((needed+31)/32) and returns 1 for needed==0;
  fsm_space_cat_to_avail(255) = 8160 exactly, else cat*32. The reason 255 is reserved is quoted
  from the header: otherwise a page with exactly MaxFSMRequestSize free would not satisfy a
  request for MaxFSMRequestSize. needed > 8160 is an elog(ERROR) "invalid FSM request size", and
  it is UNREACHABLE from SQL because lesson 04's "row is too big" fires first.
- A STALE COMMENT, flagged and resolved by measurement: freespace.c's worked example says
  "MaxFSMRequestSize is 8164 bytes" and tabulates 8164-8192 as category 255. The code says 8160.
  MEASURED: pg_freespace on the emptiest possible heap page returns 8160. The comment's example is
  4 bytes wrong (8164 is lesson 02's PageGetFreeSpace on a fresh page). Lesson 05's Next Step 6
  asks whether it was ever true -- a git log question, not yet answered.
- THE ON-PAGE STRUCTURE, derived and then MEASURED. FSMPageData { int fp_next_slot; uint8
  fp_nodes[] }, offsetof(fp_nodes) = 4. NodesPerPage = 8192 - MAXALIGN(24) - 4 = 8164.
  NonLeafNodesPerPage = BLCKSZ/2 - 1 = 4095 = 2^12 - 1. LeafNodesPerPage = SlotsPerFSMPage =
  8164 - 4095 = 4069. The non-leaf count is a round power of two minus one and the leaf count is
  the RAGGED REMAINDER -- which is why SlotsPerFSMPage is not a power of two, why 4096-4069 = 27
  leaf positions do not exist on the page, and why the README says "a few right-most leaf nodes
  are missing". One bottom FSM page covers 4069 heap blocks = 31.79 MiB of heap.
- BYTE OFFSETS, ESTABLISHED AND MEASURED, reusable without re-deriving: PageGetContents = page+24,
  so fp_next_slot is at page offset 24 (int32 LE), fp_nodes[0] (the page root) at offset 28, and
  LEAF SLOT k AT OFFSET 4123 + k. Node 8163 lands on byte 8191 -- the array exactly fills the page.
  MEASURED: get_byte(...,28) = get_byte(...,4123) = 12 on a page whose pg_freespace avail was 384
  = 12*32; bytes 24..31 = "00000000 0c 0c 00 0c" decoded field by field against fsm_page_contents.
- THE TREE OF PAGES. FSM_TREE_DEPTH = ((SlotsPerFSMPage >= 1626) ? 3 : 4) = 3; FSM_ROOT_LEVEL 2,
  FSM_BOTTOM_LEVEL 0; constant height by design. 4069^3 = 67,369,460,509 >> 2^32-1. Logical
  addressing {level, logpageno} + slot; fsm_get_location = heapblk/4069, heapblk%4069;
  fsm_get_parent/fsm_get_child divide and multiply by SlotsPerFSMPage.
- fsm_logical_to_physical FULLY EVALUATED AND MEASURED. root {2,0} -> block 0; {1,0} -> 1;
  {0,0} -> 2; {0,1} -> 3; {0,4068} -> 4070; {1,1} -> 4071; {0,4069} -> 4072. THE FORK IS NOT
  LEVEL-MAJOR: upper-level pages are interleaved into the stream where they become necessary.
  CONSEQUENCE, MEASURED: the minimum FSM fork is THREE BLOCKS = 24576 bytes, because the bottom
  page cannot be block 0. Observed: 1 heap page -> fsm 0 bytes; 2..4069 heap pages -> 24576;
  4071 heap pages -> 32768, and the level-1 page (block 1) gains its second populated leaf
  (node 4096). SlotsPerFSMPage = 4069 is now an OBSERVATION.
- fsm_set_avail: bubble up via parentof, stop as soon as a parent's recomputed max is unchanged;
  the no-op fast path has TWO conditions (oldvalue == value AND value <= fp_nodes[0]) because an
  equal leaf under a too-low root must still proceed so the trailing sanity check fires; and that
  check (value > fp_nodes[0] -> fsm_rebuild_page) is self-correcting measure #1.
- fsm_search_avail: the EXPANDING SEARCH TRIANGLE. Root byte < minvalue -> return -1 (one unlocked
  byte answers "nothing anywhere"). Start at fp_next_slot, ALWAYS clamped. Loop
  parentof(rightneighbor(nodeno)); rightneighbor detects stepping onto a level's leftmost node
  (2^level - 1) with ((x+1) & x) == 0 and corrects with parentof. Triangle doubles each step =>
  log2(N). Result is THE FIRST SUITABLE SLOT AT OR RIGHT OF fp_next_slot WITH WRAPAROUND, not the
  leftmost. Descent prefers the left child; if neither child qualifies the page is TORN ->
  DEBUG1 "fixing corrupt FSM block %u, relation %u/%u/%u", upgrade to exclusive, fsm_rebuild_page,
  MarkBufferDirtyHint, goto restart (self-correcting measure #2).
- fp_next_slot IS WRITTEN UNDER A SHARED LOCK, deliberately. Typed int not uint16 because "int is
  more likely to be atomically fetchable/storable"; README: "fp_next_slot is just a hint and we can
  easily reset it if it gets corrupted". MEASURED: after a search returning slot 0 at the bottom
  level, bytes 24..27 = 01000000 -> fp_next_slot = 1, i.e. advancenext. fsm_search passes
  advancenext = (addr.level == FSM_BOTTOM_LEVEL), so only leaf pages advance.
  NOTE for master: this is now gated on BufferBeginSetHintBits()/BufferFinishSetHintBits() in
  @126976a8, which PG 16 does not have. Flagged in the lesson's divergence note.
- NOT WAL-LOGGED, and master's README argues the nuance HARD -- four new paragraphs. Writes use
  MarkBufferDirtyHint(), reads use RBM_ZERO_ON_ERROR, so a bad FSM page becomes "nothing free",
  which costs space not correctness. BUT the README insists RelationCopyStorage() (ALTER TABLE SET
  TABLESPACE), RelationCopyStorageUsingBuffer() (CREATE DATABASE STRATEGY = wal_log), base backups
  and pg_checksums ALL verify checksums on *_fsm files, so the hint-bit FPIs are load-bearing and
  must not be downgraded to MarkBufferDirty(). Quote it if a later lesson claims the FSM is
  simply unchecked. (Lesson 24/25 may want this when FPIs come up.)
- fsm_does_block_exist: cached smgr_cached_nblocks[MAIN_FORKNUM] fast path, else
  RelationGetNumberOfBlocks -- the comment admits the lseek() cost and says the alternative "would
  cause FSM to set zero space available for blocks that main fork extension just recorded". A hit
  past the end zeroes the slot and RESTARTS FROM THE ROOT, with the comment calling advancenext
  "pessimal for this rare scenario". fsm_search has an emergency valve: restarts > 10000 ->
  InvalidBlockNumber, "Looping indefinitely is nevertheless scary".
- AN FSM PAGE IS AN "EMPTY" PAGE TO bufpage.c -- MEASURED. PageInit(page, BLCKSZ, 0): pd_lower 24
  forever, pd_upper = pd_special = 8192, so PageIsEmpty() is ALWAYS true, PageGetExactFreeSpace()
  is ALWAYS 8168, and lesson 02's four free-space functions are meaningless on this fork. The whole
  payload lives in lesson 02's HOLE, which is why PageInit's full-page MemSet and the checksum's
  coverage of the hole matter here.
- THE FINGERPRINT DOES NOT IDENTIFY THE FORK -- MEASURED, and it corrects a tempting misreading of
  lessons 02/04. FSM page bytes 12-19 = "1800 0020 0020 0420"; heap page = "2800 c001 0020 0420".
  BYTES 16-19 ARE "00 20 04 20" FOR BOTH. The fingerprint identifies the SPECIAL-SPACE SIZE (zero),
  which heap, sequence, TOAST and FSM pages all share. State it that way from now on.
- MEASURED, the FSM-vs-reality chain on an empty heap page: exact hole 8164 -> PageGetHeapFreeSpace
  / PageGetFreeSpace 8160 -> cat 255 -> reported 8160. Every step cites an earlier lesson's number.
  pd_lower was 28 (one LP_UNUSED pointer survived PageTruncateLinePointerArray).
- MEASURED, rounding loss: a page with exact hole 2080 / PageGetFreeSpace 2076 is recorded as 2048.
  28 bytes invisible; maximum possible loss 31 bytes per block.
- *** THE BEST MEASUREMENT OF THE LESSON, and the one a later lesson should cite: THE UPPER LEVELS
  ARE STALE UNTIL VACUUM. *** fsm_set_and_search bubbles up WITHIN ONE PAGE only; nothing
  propagates a bottom page's root into its parent's leaf except FreeSpaceMapVacuum[Range](), which
  VACUUM calls. RecordPageWithFreeSpace's own comment says so. MEASURED on an 8-page table built
  by plain inserts: leaves all 384, ROOT PAGE AND LEVEL-1 PAGE ENTIRELY ZERO (fsm_page_contents
  printed only "fp_next_slot: 0"). After VACUUM both upper pages populated. THEN, with each step in
  a SEPARATE psql process so no backend reused another's RelationGetTargetBlock relcache hint:
  the same tiny INSERT landed at ctid (2,5) before VACUUM (FSM search failed at the root ->
  RelationGetBufferForTuple's LAST-BLOCK FALLBACK) and at (0,5) after VACUUM (a real FSM hit).
  Identical statement, identical table, different target block, decided by ONE BYTE at page 0
  offset 28. The last-block fallback is why this failure mode is usually invisible.
- *** THE 32-BYTE CEILING TRAP, MEASURED TO THE BYTE. *** pg5_cons: block 0 left with
  PageGetFreeSpace = 8136 (one 24-byte tuple on an otherwise empty page) -> avail_to_cat floors to
  254 -> the category promises 8128. A row whose MAXALIGNed footprint is EXACTLY 8136 asks
  needed_to_cat = ceil(8136/32) = 255 > 254, so block 0 is NOT OFFERED; the last-block fallback
  (block 2, 32 bytes) had no room; THE RELATION EXTENDED TO BLOCK 3 and the row landed at (3,1).
  8 bytes of rounding cost one 8 KB page. Generalisation taught: a block recorded at category C is
  never offered to a request above 32*C, so any block whose usable free space is not an exact
  multiple of 32 has a dead band of up to 31 bytes at the top of its range.
- MaxHeapTuplesPerPage = 291, FIRST MEASURED HERE (lesson 02 derived it). 291 single-NULL-column
  tuples of lp_len 24 fit on page 0; the 292nd went to (1,1); pd_lower 1188 = 24 + 291*4, pd_upper
  1208, exact hole 20 -- the LINE POINTER ARRAY is the binding constraint, not the tuple bytes.
- HONEST NON-RESULT, recorded deliberately: the predicted "PageGetHeapFreeSpace returns 0 at the
  291 ceiling" trap DID NOT FIRE. Delete 290 of 291 and vacuum: with 290 LP_UNUSED pointers present
  PD_HAS_FREE_LINES is set (page flags 5 = PD_HAS_FREE_LINES|PD_ALL_VISIBLE), the function finds a
  reusable pointer and returns 6976, not 0. DO NOT cite the 291 ceiling as a cause of FSM
  under-reporting. What it DOES cost, measured: keeping the LAST row leaves all 291 line pointers
  (pd_lower 1188, exact hole 6980, FSM 6976); keeping the FIRST row truncates the array to 1
  (pd_lower 28, exact hole 8140, FSM 8128). SAME TABLE, SAME ONE SURVIVOR, 1152 BYTES OF DIFFERENCE
  decided only by WHERE ON THE PAGE the survivor sits, because PageTruncateLinePointerArray can
  only reclaim a trailing contiguous unused run (lesson 02).
- Also measured: a ONE-PAGE table has NO FSM fork (0 bytes) and pg_freespace reports 0 for its only
  block, because fsm_readbuf(..., extend=false) returns InvalidBuffer and GetRecordedFreeSpace
  returns 0.
- Index FSM semantics NAMED AND NOT TAKEN: for indexes the FSM tracks entirely-unused pages rather
  than free space within pages, so the byte is a boolean; indexfsm.c is the separate wrapper.
  Lesson 05's Next Step 5 asks a later reader to settle which AMs use it. Lessons 28-30 may want it.
- The README's TODO quoted and left open: "fastroot to avoid traversing upper nodes with just 1
  child" and "use a different system for tables that fit into one FSM page" -- the second is
  exactly what the reverted b0eaa4c51b tried to do.

VERSION DIVERGENCE: smaller than lesson 04's. storage/freespace.h is BYTE-IDENTICAL between master
@126976a8 and REL_16_STABLE. The differences found: (1) master's fsm_search_avail writes
fp_next_slot inside `if (exclusive_lock_held || BufferBeginSetHintBits(buf))` with a matching
BufferFinishSetHintBits(buf, false, false) -- that hint-bit API does not exist in 16, which simply
writes the field; (2) master's README carries four paragraphs on RBM_ZERO_ON_ERROR vs external
checksum verifiers that older trees do not; (3) FreeSpaceMapPrepareTruncateRel's
log_newpage_buffer() path carries an "XXX consider XLogSaveBufferForHint()" note. IDENTICAL in both:
every constant, every conversion function, the addressing arithmetic, the tree depth rule, and the
on-page layout. EVERY MEASURED NUMBER IS PG 16.

NO SYLLABUS REVISION THIS RUN. Lesson 05's title was taught exactly as written and nothing was left
over. The only correction made was to lesson 04's HANDOVER (the nonexistent
HEAP_FSM_CREATION_THRESHOLD), not to the syllabus.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg5_grow, pg5_empty, pg5_tiny, pg5_tiny2, pg5_one, pg5_gran, pg5_round,
pg5_stale, pg5_big, pg5_cons. Keep the pgNN_ prefix convention.

-- LESSON 06 (2026-10-25) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-25-postgres-visibility-map/README.md (single README, not split -- one
fork, one mechanism). Source commits read: master @0019a1ecf4c7615a2570d21eeb4b32ae3b371f18
(visibilitymap.c, visibilitymap.h, visibilitymapdefs.h) PLUS REL_16_STABLE @774c47de (the same
three files + the five VM call sites in heapam.c) and REL_16_STABLE @0cf2c7e3 (heapam_xlog.h).
Trees so far: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051, 05 @126976a8,
06 @0019a1ec (+ @774c47de / @0cf2c7e3). Keep stating the commit.
Files read IN FULL: access/heap/visibilitymap.c (master), access/visibilitymap.h,
access/visibilitymapdefs.h, access/heapam_xlog.h (16). Read in part: heapam.c (16) -- only the
all_visible_cleared / all_frozen_set sites.

*** THIS RUN HAD A CLUSTER and crossed a VM PAGE BOUNDARY (35,000 heap blocks, ~273 MB). Same
recipe as 03/04/05, both traps still apply (initdb not as root; data dir not under the scratchpad
path -- /var/tmp works; psql -f scripts must also live somewhere world-readable). ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
segment_size 1GB, data_checksums OFF, pageinspect 1.12, pg_visibility 1.2, pg_freespacemap 1.2.
Disk headroom at the time: ~30 GB on /, which is what made the boundary table possible. pg_waldump
is at /usr/lib/postgresql/16/bin/pg_waldump and WORKS on the live cluster's segments; --stats=record
gives a per-record-type byte breakdown. A LATER LESSON THAT WANTS WAL EVIDENCE SHOULD USE IT (22,
23, 24, 25 all will).

TOOK, and must not be re-taught:
- THE CONSTANTS, all of visibilitymapdefs.h (25 lines, the whole public vocabulary):
  BITS_PER_HEAPBLOCK 2, VISIBILITYMAP_ALL_VISIBLE 0x01, VISIBILITYMAP_ALL_FROZEN 0x02,
  VISIBILITYMAP_VALID_BITS 0x03. THE 0b10 CODE POINT IS UNREACHABLE BY ASSERTION at both ends:
  Assert(flags != VISIBILITYMAP_ALL_FROZEN) in _set, Assert(flags != VISIBILITYMAP_ALL_VISIBLE)
  in _clear. Two bits, three states. Taught as a deliberate waste, not an accident.
- THE GEOMETRY, derived and then MEASURED. MAPSIZE = BLCKSZ - MAXALIGN(SizeOfPageHeaderData)
  = 8168. HEAPBLOCKS_PER_BYTE = BITS_PER_BYTE/BITS_PER_HEAPBLOCK = 4. HEAPBLOCKS_PER_PAGE
  = 8168*4 = 32672 = 255.25 MiB of heap per VM page = 8.03x the FSM's 4069. NO TREE, flat array,
  so no FSM_TREE_DEPTH analogue and no ragged remainder: 8168*4 is exact, where lesson 05's
  8164-4095 = 4069 stranded 27 leaf positions. Quote the header's "There's no extra headers".
- BYTE OFFSETS, ESTABLISHED AND MEASURED, reusable without re-deriving (this is lesson 06's
  equivalent of lesson 05's "leaf slot k at offset 4123+k"):
      VM page   = b / 32672
      offset    = 24 + (b % 32672)/4      <- PageGetContents is page+24, same base as lesson 05
      bit shift = (b % 4) * 2             <- low bit ALL_VISIBLE, high bit ALL_FROZEN
  MEASURED at the boundary: heap block 32671 -> VM page 0 byte index 8167 -> PAGE OFFSET 8191,
  the last byte of the page; heap block 32672 -> VM page 1 offset 24. Byte 23 is page header, 0.
- *** THE FIRST VM PAGE BOUNDARY CROSSED IN THIS TRACK. *** Unlogged pg6_bound, 140,000 rows x
  1800-byte pad -> 35,000 heap blocks. vm = 16384 (2 blocks, because 35000 > 32672);
  fsm = 90112 (11 blocks = ceil(35000/4069)=9 bottom + 1 level-1 + 1 root, LESSON 05'S ARITHMETIC
  CONFIRMED INDEPENDENTLY). Byte 8191 of VM page 0 = 0xff, byte 24 of VM page 1 = 0xff.
  pg_visibility_map_summary: 35000 / 35000.
- THE FORK FLOOR, side by side with lesson 05: a ONE-BLOCK heap after its first VACUUM has
  vm = 8192 (ONE block) and fsm = 24576 (THREE blocks). The VM has no tree so nothing stops a
  map page being block 0. *** AND A REFINEMENT OF LESSON 05: lesson 05 measured "a one-page table
  has NO FSM fork (0 bytes)". That is the PRE-VACUUM state only -- VACUUM creates the FSM fork
  even for a one-block heap (measured 0 -> 24576). Do not propagate the unqualified claim. ***
- AN FSM PAGE IS AN EMPTY PAGE / so is a VM page: PageInit(page, BLCKSZ, 0) in vm_readbuf, so
  MEASURED pd_lower 24, pd_upper = pd_special = 8192, pd_flags 0, prune_xid 0, PageIsEmpty()
  always true, payload entirely in lesson 02's HOLE. ONE DIFFERENCE FROM THE FSM WORTH KEEPING:
  pd_lsn on a VM page is NON-ZERO and set deliberately (PageSetLSN in visibilitymap_set);
  measured 0/15E9740. On an FSM page it is incidental.
- PD_ALL_VISIBLE, WHICH LESSON 02 DEFERRED, IS NOW CLOSED. Taught as the two-places-one-truth
  design with a reason: the VM bit is what a READER consults (cheap, stays cached); the page flag
  is what a WRITER consults (it already holds the page; pinning the VM page might need I/O). That
  asymmetry is the ONLY reason the pin-before-lock dance exists.
- THE LOCKING DANCE, quoted from the header and taught: examine the heap page UNLOCKED, pin the VM
  page if PD_ALL_VISIBLE is set, then lock the heap page; if the flag got set in between, unlock,
  pin, relock. visibilitymap_pin_ok() is the cheap re-check (pure block-number compare, no I/O).
  The header's "This shouldn't happen often, because only VACUUM currently sets visibility map
  bits" was FLAGGED AS ARGUABLY STALE -- see COPY FREEZE below. Lesson 06's Next Step 3 proposes
  it as a documentation patch.
- THE FIVE CLEAR CALL SITES IN heapam.c (16), tabulated: heap_insert, heap_multi_insert (guarded
  `&& !(options & HEAP_INSERT_FROZEN)`), heap_delete, heap_update old page, heap_update new page
  (`newbuf != buffer &&`). All five do PageClearAllVisible + visibilitymap_clear(VALID_BITS) --
  ALWAYS BOTH BITS, which is why the Assert(flags != ALL_VISIBLE) in _clear can exist. All five
  raise a flag in the HOST record: XLH_INSERT_ALL_VISIBLE_CLEARED, XLH_DELETE_ALL_VISIBLE_CLEARED,
  XLH_UPDATE_OLD_ALL_VISIBLE_CLEARED, XLH_UPDATE_NEW_ALL_VISIBLE_CLEARED.
  visibilitymap_clear uses MarkBufferDirty, *NOT* MarkBufferDirtyHint -- the contrast with lesson
  05 is literally one function name deep. Taught that way.
- *** WHY THE VM IS WAL-LOGGED AND THE FSM IS NOT, with PG 16's own argument quoted in full.
  The record does NOT protect the bit; it keeps the TWO COPIES from diverging in the unsafe
  direction (map says visible, page has forgotten -> no writer will ever clear it). Clearing needs
  no record because the clearing is always INSIDE another record. ***
  xl_heap_visible = { TransactionId snapshotConflictHorizon; uint8 flags; }, SizeOfHeapVisible = 5,
  XLOG_HEAP2_VISIBLE = 0x40, "Backup blk 0: visibility map buffer / Backup blk 1: heap buffer".
- MEASURED WITH pg_waldump, the price of all that: 980 Heap2/VISIBLE records in one 16 MB segment;
  57,830 record bytes + 16,384 FPI bytes = 74,214 total = 5.65% of the segment. Histogram: 978
  records at EXACTLY 59 BYTES, 2 at 64/8256 (the FPI of each VM page on its first touch after a
  checkpoint). flags 0x01 observed for VACUUM with freeze off, 0x03 for VACUUM FREEZE. Verbatim
  record shape recorded in the lesson, with the two blkrefs and `fork vm blk 0 FPW`.
  SO: 59 BYTES OF WAL PER HEAP PAGE MARKED, plus one FPI per VM page per checkpoint. The FSM pays
  zero for the same VACUUM. That pair of numbers is the whole 05-vs-06 tradeoff and a later lesson
  should cite it rather than re-measure.
  NOT a VM-only figure, recorded separately so nobody confuses them: a VACUUM of a 118-block table
  emitted 18,400 WAL bytes total across all record types = 155.9 bytes/block. Lesson 19 owns that
  breakdown.
- COPY FREEZE SETS THE BITS AT LOAD TIME, measured, NO VACUUM: heapam.c's
  `starting_with_empty_page = PageGetMaxOffsetNumber(page) == 0; if (starting_with_empty_page &&
  (options & HEAP_INSERT_FROZEN)) all_frozen_set = true;` with
  Assert(!(all_visible_cleared && all_frozen_set)) and XLH_INSERT_ALL_FROZEN_SET.
  MEASURED: 5000 rows, 80 heap blocks, 47 blocks (0..46) with BOTH bits, byte 24 = 0xff.
  THE 33 UNSET BLOCKS ARE TRAILING *EMPTY* PAGES left by bulk relation extension -- max block
  actually containing a row was 46, so no heap_multi_insert ever ran on 47..79. Autovacuum later
  truncated the relation to 47 blocks and all 47 were all-visible+all-frozen. Lesson 06's Next
  Step 4 hands the "why does extension overshoot" question to lesson 08.
- *** THE HEADLINE MEASUREMENT, and the one a later lesson should cite: PAGE GRANULARITY COSTS 59
  HEAP FETCHES FOR ONE UPDATED ROW. *** pg6_ios, 50,000 rows / 863 heap blocks / PK on id, query
  `select count(id) from pg6_ios where id between 1 and 20000`, seqscan+bitmapscan off:
    after INSERT (no vm fork):          Heap Fetches 20000, Buffers shared hit=401
    after VACUUM:                        Heap Fetches     0, Buffers shared hit=57   (7.0x)
    after ONE update of id=7:            Heap Fetches    59, Buffers shared hit=60
    after pg_truncate_visibility_map():  Heap Fetches 20000, Buffers shared hit=403
  THE PLAN NODE WAS `Index Only Scan using pg6_ios_pkey` IN ALL FOUR -- taught as "index-only is a
  plan shape, not a promise; Heap Fetches is the only place EXPLAIN tells you which happened".
  THE 59 IS FULLY ACCOUNTED FOR: id=7 lived on block 0, the new version landed at (862,5), and
  pg_visibility showed EXACTLY blocks 0 and 862 cleared; block 0 holds 57 live rows + 1 dead index
  entry still pointing at (0,7) = 58, plus 1 on block 862 = 59.
  *** CORRECTED BY LESSON 07 (2026-10-26): THE 59 IS NOT A CONSTANT. *** Lesson 07 rebuilt this
  experiment, reproduced the geometry EXACTLY (863 blocks, shared hit=57, new version on block
  862, exactly blocks 0 and 862 cleared) and measured 58, because its table holds 57 rows on
  block 0 where this one held 58. The figure is (index entries in range pointing at the dirtied
  block) + 1. Cite the formula and give the measured instance; do not carry 59 forward as a
  constant.
- MEASURED, the byte sequence that proves the whole addressing scheme with three get_byte calls,
  on a 12-block table: (no fork) -> 85 -> 255 -> 243.
  85 = 0x55 = 01010101 = four blocks all-visible, none frozen -- LITERALLY VISIBLE_MASK8.
  255 = 0xff after VACUUM FREEZE. 243 = 0xf3 after updating one row on BLOCK 1
  = 0xff & ~(0x03 << 2), i.e. HEAPBLK_TO_OFFSET(1) = 2 confirmed at the bit level.
  Byte 27 = 0 because the heap stops at block 11 -- which is what makes visibilitymap_count's
  "we assume that any extra bytes in the last page are zeroed" safe.
- visibilitymap_count: VISIBLE_MASK8 0x55 "the lower bit of each bit pair", FROZEN_MASK8 0xaa
  "the upper bit of each bit pair", pg_popcount_masked, NO LOCK ("immediately stale anyway... we
  only really need an approximate value"). This is what feeds pg_class.relallvisible. Lesson 06's
  Next Step 6 leaves the planner's cost formula to lesson 31.
- visibilitymap_get_status: ONE UNLOCKED BYTE READ, "A single byte read is atomic", "It is the
  caller's responsibility to deal with all concurrency issues!". THE TWO-PARAGRAPH SAFETY PROOF
  WAS TAUGHT because it is specific and non-obvious: INSERTS are serialised for free (the VM bit
  is cleared BEFORE the index is touched, and index buffer lock/unlock is a full memory barrier);
  DELETES may be arbitrarily stale but the tuple stays visible until the deleter commits and
  ProcArrayLock acts as the read barrier. FLAGGED AS A CONSTRAINT: any NEW consumer of the VM has
  to re-derive that argument for itself.
- vm_readbuf's RBM_ZERO_ON_ERROR, same justification as the FSM ("It's always safe to clear bits,
  so it's better to clear corrupt pages than error out"), and the extend=false -> InvalidBuffer ->
  `return (uint8) 0` path, which is why a table with NO vm fork behaves as "nothing is visible"
  rather than erroring. Matches the docs' "queries will treat the visibility map as containing all
  zeroes". vm_extend = ExtendBufferedRelTo(EB_CREATE_FORK_IF_NEEDED | EB_CLEAR_SIZE_CACHE) +
  CacheInvalidateSmgr; relation extension itself left to lesson 08 as a black box, as lesson 05
  did.
- visibilitymap_prepare_truncate: MemSet(&map[truncByte+1], 0, MAPSIZE-(truncByte+1)) and
  map[truncByte] &= (1 << truncOffset) - 1, with the reason quoted -- the tail bits are cleared
  "because we don't get a chance to clear the bits if the heap is extended again", i.e. a
  re-extended heap would inherit STALE SET BITS, the unsafe direction. This is the one VM write
  gated on XLogHintBitIsNeeded() -> log_newpage_buffer, with "We cannot use MarkBufferDirtyHint
  here, because that will not dirty the page during recovery". UNEXERCISED: data_checksums was OFF
  this run, so that branch was not observed. Lesson 06's Next Step 5.
- pg_visibility 1.2 as the observation tool, all signatures taught from the docs: pg_visibility_map
  (2 overloads), pg_visibility (2 overloads, adds pd_all_visible), pg_visibility_map_summary,
  pg_check_frozen, pg_check_visible, pg_truncate_visibility_map. Permissions recorded: superuser
  or pg_stat_scan_tables, except pg_truncate_visibility_map which is superuser-only.
- *** MEASURED, A FORCED TWO-PLACES DISAGREEMENT, and the blind spot it exposes. ***
  pg_truncate_visibility_map('pg6_ios') then pg_visibility: vm all_visible = 0 of 863, heap pages
  pd_all_visible = 861 (861 not 863 because two blocks were already cleared by the UPDATE).
  Nothing corrupt; the optimisation simply stopped. AND pg_check_visible STILL RETURNED 0 ROWS --
  it is defined as "non-all-visible tuples stored in pages marked all-visible IN THE VISIBILITY
  MAP", so a map that marks nothing can hide nothing. THE CHECKS DETECT OVER-CLAIMING ONLY. A
  clean pg_check_visible is NOT evidence the map is right. After the repairing VACUUM: 862 / 862.
- The docs' "four orders of magnitude smaller" claim was CHECKED rather than repeated: the
  asymptote is 32672:1 (3.27e4, so the claim is right) but THE FORK HAS A ONE-PAGE FLOOR, so
  MEASURED ratios were 1:1 for a one-block table, 863:1 for pg6_ios, 17500:1 for pg6_bound. The
  advertised ratio only arrives past 255.25 MiB of heap.

VERSION DIVERGENCE: LARGER THAN LESSON 05'S, and it moves RESPONSIBILITY rather than geometry.
(1) visibilitymap_set in 16 is
    `(Relation rel, BlockNumber heapBlk, Buffer heapBuf, XLogRecPtr recptr, Buffer vmBuf,
      TransactionId cutoff_xid, uint8 flags) -> void`;
    in master it is `(BlockNumber heapBlk, Buffer vmBuf, uint8 flags, RelFileLocator rlocator)
      -> uint8` and RETURNS THE PREVIOUS BITS.
(2) WHO LOCKS: 16's function does LockBuffer(vmBuf, BUFFER_LOCK_EXCLUSIVE) itself; master asserts
    the CALLER holds it, via BufferIsLockedByMeInMode (a predicate 16 does not have).
(3) WHO LOGS: 16's function calls log_heap_visible() and PageSetLSN, and sets the HEAP page's LSN
    only `if (XLogHintBitIsNeeded())` with a four-line comment on why it must NOT otherwise;
    master does NO WAL at all and asserts `InRecovery || CritSectionCount > 0`, leaving the caller
    to log the VM change together with the heap change.
(4) THE HEADER COMMENT CHANGED WITH IT, and 16's is the clearer statement of WHY: 16 says
    "Clearing visibility map bits is not separately WAL-logged" + "When we *set* a visibility map
    during VACUUM, we must write WAL. This may seem counterintuitive, since the bit is basically a
    hint..."; master generalises to "Changes to the visibility map bits are not separately
    WAL-logged" because setting no longer logs either. QUOTE 16'S VERSION even when reading master.
(5) visibilitymap_count: 16 uses VISIBLE_MASK64/FROZEN_MASK64 + pg_popcount64 over uint64 words
    with StaticAssertStmt(MAPSIZE % sizeof(uint64) == 0, "unsupported MAPSIZE"); master uses
    VISIBLE_MASK8 0x55 / FROZEN_MASK8 0xaa + pg_popcount_masked over bytes.
(6) master adds HEAPBLK_TO_MAPBLOCK_LIMIT and visibilitymap_truncation_length(); 16 has neither.
(7) master's visibilitymap_clear gained the same BufferIsLockedByMeInMode assertion.
IDENTICAL IN BOTH: BITS_PER_HEAPBLOCK, all three flag constants, MAPSIZE, HEAPBLOCKS_PER_BYTE,
HEAPBLOCKS_PER_PAGE, all three address macros, PageInit(page, BLCKSZ, 0), RBM_ZERO_ON_ERROR,
vm_extend's ExtendBufferedRelTo + CacheInvalidateSmgr, and prepare_truncate's masking arithmetic.
EVERY MEASURED NUMBER IS PG 16.15.

NO SYLLABUS REVISION THIS RUN. Lesson 06's title was taught exactly as written -- PD_ALL_VISIBLE,
all-frozen and the index-only scan it enables were all three taken in full, and nothing was left
over. Lesson 05's handover scope was executed completely, including the two measurements it asked
for by name (decode a VM page by hexdump and recover the bits for a known block; compare
pg_relation_size(rel,'vm') against (rel,'fsm') on the same table).

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg6_one, pg6_move, pg6_ios, pg6_freeze, pg6_bound, pg6_walcost, pg6_wal.
Keep the pgNN_ prefix convention.

-- LESSON 07 (2026-10-26) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-26-postgres-hot-updates-page-pruning/README.md (single README, not split
-- one mechanism; the prune heuristic and the chain walk are the same story told from two ends).

*** SOURCE PROVENANCE IS DIFFERENT THIS RUN AND THE NEXT RUN SHOULD KNOW WHY. *** No git tree was
fetched. The source read is the Debian/Ubuntu source package postgresql-16 version
16.15-0ubuntu0.24.04.1, obtained with `apt-get source postgresql-16` after adding deb-src lines for
noble to /etc/apt/sources.list.d/. THAT IS THE SAME BUILD THE CLUSTER RUNS, so code and
measurements cannot drift. Verified rather than assumed: none of the 14 patches in
debian/patches/ touches pruneheap.c, heapam.c, itemid.h, bufpage.h, htup_details.h or
heapam_xlog.h (checked by extracting every +++/--- path out of the patch series), so the quoted
files are upstream REL_16_15 verbatim. Trees/releases so far: 01 @ddd468ca, 02 @3cde90ec,
03 @f2edf5dd, 04 @f52d4051, 05 @126976a8, 06 @0019a1ec, 07 = release 16.15 (no SHA; said so in the
document rather than inventing one). THE apt-get source ROUTE IS FASTER AND MORE FAITHFUL THAN
WEB-FETCHING FILES ONE AT A TIME AND LATER LESSONS SHOULD USE IT; WebFetch on
postgresql.org/docs summarises rather than reproducing, and cannot be quoted verbatim.
Files read IN FULL: access/heap/README.HOT (520 lines), access/heap/pruneheap.c (1238 lines),
storage/itemid.h (184 lines). Read in part: heapam.c (the use_hot_update block ~3800-3885 and
HeapDetermineColumnsInfo), bufpage.h (PD_PAGE_FULL, PageSetPrunable/PageClearPrunable,
PageIsFull/SetFull/ClearFull), htup_details.h (the HOT t_infomask2 macros), heapam_xlog.h
(xl_heap_prune, SizeOfHeapPrune, the XLH_UPDATE_* flags).

*** THIS RUN HAD A CLUSTER and EVERY number in the lesson is measured. Same recipe as 03/04/05/06,
both traps still apply (initdb not as root; data dir not under the scratchpad path -- /var/tmp
works; psql -f scripts must live somewhere world-readable such as /var/tmp). ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
segment_size 1GB, data_checksums OFF, pageinspect 1.12, pgstattuple 1.5, pg_visibility 1.2,
pg_freespacemap 1.2. pg_waldump at /usr/lib/postgresql/16/bin/pg_waldump, used again, works.

TOOK, and must not be re-taught:
- LP_REDIRECT CLOSED (lesson 02's deferred fourth state). The whole itemid.h enum with its
  parenthetical invariants, the no-storage convention quoted ("By convention, lp_len == 0 in every
  line pointer that does not have storage, independently of its lp_flags state"), and
  ItemIdGetRedirect's comment quoted. *** THE CENTRAL FACT, TAUGHT AND MEASURED: lp_off CHANGES
  UNITS. Byte offset in LP_NORMAL, OffsetNumber in LP_REDIRECT, with lp_flags the only
  discriminator. MEASURED lp_off = 19 on a redirect, and 19 is inside the 24-byte page header, so
  it CANNOT be a byte offset -- that impossibility is the proof and it is reusable. ***
- THE FULL LINE-POINTER STATE MACHINE, every transition MEASURED on one page (pg7_chain, 18 rows,
  396-byte tuples -> MAXALIGN 400, lower 96 = 24+18*4, upper 992 = 8192-18*400, both exact):
    NORMAL -> REDIRECT  root dead, later member live    (heap_prune_record_redirect)
    NORMAL -> DEAD      tuple dead, no surviving chain  (heap_prune_record_dead)
    NORMAL -> UNUSED    heap-only tuple dead            (heap_prune_record_unused)
    REDIRECT -> REDIRECT retarget; REDIRECT -> DEAD whole chain dead
    DEAD -> UNUSED      VACUUM ONLY, after the index entry goes
  THE ASYMMETRY IS THE DESIGN and heap_page_prune_execute says it: "A heap-only tuple can never
  become LP_DEAD; an LP_REDIRECT item or a regular heap tuple can." Keep that sentence.
- THE INFOMASK2 BITS CLOSED (lesson 03 assigned them here). HEAP_HOT_UPDATED 0x4000,
  HEAP_ONLY_TUPLE 0x8000, and MEASURED t_infomask2 values that decode exactly:
  16387 = 0x4003 (root, HOT_UPDATED + 3 natts), 32771 = 0x8003 (heap-only tail),
  *** 49155 = 0xC003 = BOTH FLAGS on a mid-chain tuple ***, which is the observation that makes the
  two flags visibly orthogonal. heap_tuple_infomask_flags() agreed. ALSO: HeapTupleHeaderIsHotUpdated
  is NOT a bare bit test -- it is the bit AND !XMAX_INVALID AND !XminInvalid.
- pd_prune_xid AND PD_PAGE_FULL CLOSED (both lesson 02 deferrals). PageSetPrunable quoted whole and
  taught as a MONOTONE MINIMUM, not an assignment. heap_update sets it on the OLD tuple's page only,
  with its XXX quoted ("Should we set hint on newbuf as well? ... for now we choose not to optimize
  for aborts"). PD_PAGE_FULL 0x0002 is set by heap_update's `else` branch (newbuf != buffer), i.e.
  it is a record that an UPDATE already failed, and bufpage.h's "Again, this is just a hint."
- *** THE PRUNE HEURISTIC, LOCATED EXPERIMENTALLY TO WITHIN ONE ROW. minfree =
  Max(RelationGetTargetPageFreeSpace(rel, HEAP_DEFAULT_FILLFACTOR), BLCKSZ/10) = 819 at
  fillfactor 100. MEASURED BOTH SIDES on pg7_dead: 888 bytes free (10.84%) with a known-prunable
  dead tuple and pd_prune_xid set survived THREE consecutive full table scans unchanged; one extra
  INSERT took free space to 484 (5.9%) and the VERY NEXT scan pruned. PRUNING IS A SPACE-PRESSURE
  RESPONSE, NOT CLEANUP. That pair of numbers is the lesson's best single experiment. ***
- heap_page_prune_opt's early-exit ladder in cost order (RecoveryInProgress -> pd_prune_xid==0 ->
  GlobalVisTestIsRemovableXid -> free-space gate -> ConditionalLockBufferForCleanup -> RECHECK
  under the lock). THE UNLOCKED FREE-SPACE READ IS DELIBERATE and its comment is quoted
  ("Avoiding taking a lock seems more important than sometimes getting a wrong answer in what is
  after all just a heuristic estimate"). IT NEVER BLOCKS; README.HOT prices the failure
  ("The worst-case consequence of this is only that an UPDATE cannot be made HOT").
- THE THREE CALL SITES, tabulated: heapam_handler.c:138 (heapam_index_fetch_tuple),
  heapam_handler.c:2157 (bitmap heap scan), heapam.c:465 (heapgetpage / seqscan). NO INSERT PATH,
  with README.HOT's reason quoted. So a plain SELECT is a writer. MEASURED: the prune in the lesson
  was triggered by `select count(*)`, and separately pg7_hot's block 0 was already pruned BY THE
  INDEX-ONLY SCAN ITSELF before heap_page_items was ever called.
- THE FSM IS DELIBERATELY NOT UPDATED AFTER A PRUNE, comment quoted ("The free space should be
  reused by UPDATEs to *this* page"). THIS IS A DIRECT POLICY CONSTRAINT ON LESSON 05's FSM and
  lesson 19 will meet the other half of it (VACUUM does update the FSM).
- heap_page_prune's PLAN-THEN-EXECUTE structure with its reason quoted ("ensures that WAL replay
  will work the same as the normal case"), AND PASS 0: HTSV is computed for every tuple up front
  into prstate.htsv[] because "running HTSV twice could result in different results" -- THE HORIZON
  CAN MOVE DURING THE SCAN. The backwards loop (maxoff down to FirstOffsetNumber) and its
  microarchitectural reason quoted (line pointers ascend, tuples descend, so descending the array
  ascends through memory -> "increases prefetching efficiency significantly").
- *** MarkBufferDirty vs MarkBufferDirtyHint APPEAR IN ONE FUNCTION, selected by whether real work
  was done: the prune branch uses MarkBufferDirty + XLogInsert, the no-op-but-hint-changed branch
  uses MarkBufferDirtyHint(buffer, true) with "This is treated as a non-WAL-logged hint." Lesson 06
  drew this contrast ACROSS two forks; 07 found it inside one function. Lessons 15 and 22 own the
  machinery; do not re-teach it, but this is the clearest example in the tree. ***
- heap_prune_chain's SEVEN termination conditions listed, and lesson 03's chain rule
  (xmin(next) == xmax(this)) shown load-bearing, with README.HOT's abort race quoted IN FULL
  including the parenthetical history: "(Early versions of the HOT code assumed that holding pin on
  the page buffer while following a HOT link would prevent this type of problem, but checking
  XMIN/XMAX matching is a much more robust solution.)" The walk advances past DEAD *and*
  RECENTLY_DEAD "just in case there's a DEAD one after them" and stops at anything else.
- THE FOUR OUTCOMES of heap_prune_chain, quoted as code: (1) intermediates -> unused, (2) i>=nchain
  -> record_dead, (3) else -> record_redirect, (4) the SELF-HEALING orphaned-redirect branch
  (nchain < 2 && ItemIdIsRedirected) whose comment admits the forward scan can reach a chain out of
  order. Plus the PRE-LOOP special case: a DEAD heap-only tuple that is not HotUpdated is reaped on
  sight, for aborted HOT updates (XMIN_INVALID heap-only tuples with no chain pointing at them).
  ALSO the counting subtlety: "changing a redirect (even to DEAD state) doesn't count" toward
  ndeleted, and prune_opt reports ndeleted - nnewlpdead to pgstats.
- THE HOT-ELIGIBILITY TEST, both gates in order: (1) newbuf == buffer, already settled by
  RelationGetBufferForTuple; (2) !bms_overlap(modified_attrs, hot_attrs) where hot_attrs is
  RelationGetIndexAttrBitmap(rel, INDEX_ATTR_BITMAP_HOT_BLOCKING). summarized_update via sum_attrs.
  HeapDetermineColumnsInfo + heap_attr_equals = BITWISE equality, with README.HOT's reason quoted
  ("there might be multiple notions of equality for a datatype"), plus its two unconditional
  "not equal" shortcuts (whole-row reference attrnum==0; any system attribute except tableoid).
- *** THE HOT-BLOCKING MATRIX, MEASURED at fillfactor=50 so space was NEVER the constraint, using
  the DELTA of n_tup_hot_upd per transaction: v unindexed -> HOT. btree(v) -> COLD.
  btree(id) WHERE v > 1000000 (v only in a PREDICATE) -> COLD. btree((v % 7)) (v only inside an
  EXPRESSION) -> COLD. pad = pad -> HOT. So README.HOT's glossary definition of "indexed column"
  is not pedantry, it is measurable. ESTABLISHED; do not re-derive. ***
- *** ctid DOES NOT DISTINGUISH HOT FROM COLD, measured: a cold update landed at (0,30) -> (0,31),
  same page. The reliable signals are n_tup_hot_upd and the HEAP_ONLY_TUPLE bit
  (t_infomask2 & 32768). ALSO MEASURED, the chain-ending shape README.HOT describes: the old
  version keeps HEAP_ONLY_TUPLE but LOSES HEAP_HOT_UPDATED, and the new version has neither. ***
- *** LESSON 06'S 59 RE-DERIVED, AND IT IS 58 ON THIS TABLE. pg7_ios (fillfactor=100, 50,000 rows
  of pad=repeat('x',100)) reproduced lesson 06's GEOMETRY EXACTLY -- 863 heap blocks, shared hit=57
  on the clean index-only scan, Heap Fetches 0 after VACUUM, new version at (862,5) on a different
  block, exactly blocks 0 and 862 cleared in the VM. BUT Heap Fetches was 58, not 59, because this
  table has 57 rows on block 0 (measured directly) where lesson 06's had 58. 57 + 1 = 58. NOT a
  contradiction, a one-row-per-page width difference. *** DO NOT CITE 59 AS A CONSTANT. *** Cite
  "one cold update costs (rows on the old block) + 1 fetches, two VM bits, one index entry and one
  dead tuple" and give the measured instance. ***
- THE HOT SIDE, pg7_hot (fillfactor=85, same rows): 1021 heap blocks (18.3% more disk for the same
  data -- the permanent fillfactor tax, measured), new version at (0,50) SAME BLOCK, ONE VM bit
  cleared (block 0 only), n_tup_hot_upd 1, n_dead_tup 0, Heap Fetches 49 = exactly the 49 index
  entries pointing at block 0 (counted). lp 7 was ALREADY LP_REDIRECT, max(lp) 50, 49 LP_NORMAL.
- *** THE STEADY STATE, THE LESSON'S HEADLINE. pg7_steady, fillfactor=70, 5000 rows, 125 heap
  blocks, 16 index blocks, 40 rows/page. 2000 AUTOCOMMITTED single-row updates:
  2000/2000 HOT (100%), heap 125 -> 125 blocks, index 16 -> 16 blocks, pgstattuple
  dead_tuple_count 0, pd_prune_xid 0, row at (0,42), block 0 = 40 LP_NORMAL + 1 LP_REDIRECT +
  1 LP_UNUSED, max_lp 42, AND NO VACUUM EVER RAN. ZERO BYTES OF GROWTH IN EITHER FORK. ***
- THE ROW GEOMETRY ARITHMETIC, derived and then confirmed twice, reusable: pad=repeat('x',100) gives
  lp_len 133 = 24 header + 4 + 4 + 101, where 101 is a ONE-BYTE varlena header + 100 data bytes
  because 100 <= 126 (LESSON 04'S BOUNDARY STILL PAYING OFF). MAXALIGN(133) = 136, + 4 line pointer
  = 140/row. 8160/140 = 58.3 rows/page at fillfactor 100 -- MEASURED 58. 8160*0.70/140 = 40.8 at
  fillfactor 70 -- MEASURED 40. Both exact. Use this to predict HOT headroom: fillfactor F gives
  (8160*(1-F)/rowbytes) free updates per page before the chain must leave.
- *** WAL, MEASURED WITH pg_waldump --stats=record OVER A DEDICATED SEGMENT RANGE (pg_switch_wal
  first, then record the LSN -- this worked cleanly and later lessons should copy the method).
  3000 autocommitted updates each, fillfactor=70, zero FPI bytes in both runs:
    HOT  (pg7_steady, v unindexed): TOTAL 497,948 B = 166 B/update.
         Heap/HOT_UPDATE 3000 rec / 213,011 B (71.0 ea); Heap2/PRUNE 2999 rec / 182,937 B
         (61.0 ea) = 36.74%; Transaction/COMMIT 3000 / 102,000 B (34.0 ea). NO btree records.
    COLD (pg7_cold, btree(v)):      TOTAL 741,213 B = 247 B/update.
         Heap/UPDATE 2990 / 212,301 B; Heap/UPDATE+INIT 10 / 1,830 B; Btree/INSERT_LEAF
         *** 5992 rec / 383,488 B = 51.74% *** (~2 per update: PK + v); Btree/INSERT_UPPER 8,
         SPLIT_R 8, DELETE 74 (16,598 B total); Heap/LOCK 10; Heap2/PRUNE only 336; COMMIT 3000.
         Heap grew 125 -> 135 blocks.
    SO HOT SAVES 1.49x THE WAL, NOT 10x, AND ITS OWN PRUNE RECORDS ARE 36.7% OF ITS BILL.
    Btree/DELETE in the cold run is nbtree deletion keeping its leaves from bloating -> LESSON 29. ***
- THE VERBATIM STEADY-STATE WAL CYCLE, recorded in the lesson and worth reusing:
  HOT_UPDATE (71 B, old_off 11, new_off 41, flags 0x61) / COMMIT (34 B) /
  PRUNE (59 B, tx: 0, nredirected 1, redirected [11->41], nunused 0) /
  HOT_UPDATE (71 B, old_off 41, new_off 43, flags 0x60) / COMMIT /
  PRUNE (61 B, redirected [11->43], unused [41]).
  FOUR FACTS EXTRACTED: (1) *** tx: 0 ON EVERY PRUNE RECORD -- pruning is not part of any
  transaction ***; (2) *** THE ROW OSCILLATES BETWEEN OFFSETS 41 AND 43 FOREVER, root lp 11 never
  moves, two line pointers recycled indefinitely ***; (3) 59 vs 61 bytes is EXACTLY one
  OffsetNumber, and xl_heap_prune is 9 B (SizeOfHeapPrune = offsetof(isCatalogRel)+sizeof(bool)
  over {TransactionId, uint16, uint16, bool}) with the arrays riding in blkref 0's data via
  XLogRegisterBufData, 2 B/offset and 4 B/redirect pair, comment quoted ("not actually in the
  buffer, but we pretend that they are"); (4) flags 0x61 on the FIRST update and 0x60 after -- 0x60
  is PREFIX_FROM_OLD|SUFFIX_FROM_OLD and the 0x01 is XLH_UPDATE_OLD_ALL_VISIBLE_CLEARED, LESSON
  06'S FLAG, set only once because after that the VM bit is already clear. XLH_UPDATE_* values all
  tabulated from heapam_xlog.h.
- *** THE OPERATIONAL HEADLINE, AND THE BEST "WHERE THIS BREAKS DOWN" MEASUREMENT IN THE TRACK SO
  FAR. ONE IDLE TRANSACTION TURNS HOT OFF. pg7_hold, fillfactor=70, 5000 rows, 125 blocks, with a
  single `BEGIN ISOLATION LEVEL REPEATABLE READ; select count(*);` left sitting idle in another
  session, then 2000 autocommitted updates:
      1965/2000 HOT = 98.25%  BUT  heap 125 -> 160 blocks (+35), n_dead_tup 2000,
      row ended at (159,10), block 0 = 58 LP_NORMAL / 0 LP_REDIRECT / max_lp 58.
  THE ARITHMETIC CLOSES EXACTLY: block 0 started with 40 rows, holds 58 at 140 B/row, 40+18 = 58
  MEASURED; each later page starts near-empty and absorbs ~57 (8160/140), so 18 + 35*57 = 2013 ~
  2000. *** 35 COLD UPDATES PRODUCED EXACTLY 35 NEW BLOCKS. *** So a 98.25% HOT ratio coexisted
  with a 28% larger table: THE HOT RATIO IS A MISLEADING HEALTH METRIC, TRUST THE TABLE SIZE.
  SAME EFFECT WITHOUT A SECOND SESSION: 10,000 updates inside ONE `DO $$ ... $$` block on pg7_hot
  gave 9828 HOT (98.3%), 173 cold, heap 1021 -> 1193 (173 cold updates, 172 new blocks). YOUR OWN
  TRANSACTION IS THE OLD SNAPSHOT. ***
- README.HOT's framing of the ORIGINAL problem, quoted and taught as the thesis: single-page
  reclamation is blocked not by locking or WAL but by not being able to SAFELY RE-FIND the index
  entries, because "an allegedly immutable function that in fact is not immutable might prevent us
  from re-finding an index entry". HOT sidesteps it rather than solving it. And the one-way door
  from Limitations: "It'll be tough to recompute the index keys for a root line pointer you don't
  have data for anymore ..." QUOTE THESE TWO IF ANY LATER LESSON TOUCHES VACUUM STRATEGY.
- The MaxHeapTuplesPerPage = 291 cap (lesson 05's number) CITED, with README.HOT's reason quoted:
  the feature is a resurrection of an idea "proposed and rejected before because of fear of line
  pointer bloat", and the cap limits the damage. NOT OBSERVED -- see open items.

DELIBERATELY TREATED AS BLACK BOXES, as the handover asked: GlobalVisState / vistest (14),
relation extension and RelationGetBufferForTuple (08), the buffer cleanup lock's implementation
(09/10), HeapTupleSatisfiesVacuum's internals (14), VACUUM's two passes and index bulk deletion
(19), nbtree's INSERT_LEAF/DELETE/SPLIT records (28/29).

NO SYLLABUS REVISION THIS RUN. Lesson 07's title was taught exactly as written: redirect line
pointers, heap_page_prune(), and the HOT chain were all three taken in full, plus the four
lesson-02/03 deferrals the handover assigned (LP_REDIRECT, pd_prune_xid, PD_PAGE_FULL,
HEAP_HOT_UPDATED/HEAP_ONLY_TUPLE). Lesson 06's handover asked for five measurements by name and all
five were taken: (a) the fillfactor re-run of the pg6_ios experiment, (b) heap_page_items mid-chain
showing lp_flags = 2 and lp_off as an OffsetNumber, (c) pd_prune_xid set and then cleared by a
plain SELECT, (d) index tuple count across HOT updates (unchanged at 18, and 16 index blocks across
2000 updates), (e) pgstattuple's dead_tuple_percent across prune and vacuum (4.83% -> 0).

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg7_chain, pg7_dead, pg7_ios, pg7_hot, pg7_steady, pg7_cold, pg7_hold,
pg7_idx, pg7_x. Keep the pgNN_ prefix convention.

-- HANDOVER TO LESSON 08 -----------------------------------------------------------------------
Scope for lesson 08, "smgr and md.c: relation extension, the segment map, and the fsync request
queue". LESSONS 05, 06 AND 07 HAVE ALL NOW TREATED "EXTEND THE RELATION" AS A BLACK BOX AND EACH
LEFT 08 A DIFFERENT CONCRETE QUESTION. Lesson 08 is where they get answered, and it should open by
naming all three rather than starting from the file layout, because lesson 01 already taught the
file layout (GetRelationPath, the three forks, the 1 GB segments, mdtruncate's zero-length K+1st
segment) and 08 must NOT re-teach it.
THE THREE INHERITED QUESTIONS, in descending order of value:
 (1) *** FROM LESSON 06, STILL OPEN: WHY DOES BULK RELATION EXTENSION OVERSHOOT? *** Lesson 06
     measured COPY FREEZE leaving 33 trailing EMPTY pages out of 80, which is also why only 47 of
     80 blocks got VM bits, and autovacuum later truncated the relation to 47. The answer is in
     RelationGetBufferForTuple's extension batching (and in PG 16, ExtendBufferedRelBy /
     EB_* flags, which lesson 06 saw named in vm_extend but did not read).
 (2) NEW FROM LESSON 07: lesson 07 measured, repeatedly, that N cold updates produce EXACTLY N new
     blocks when the old page is full (35 cold -> 35 blocks; 173 cold -> 172 blocks). That is a
     statement about RelationGetBufferForTuple's page-choice policy and its interaction with
     LESSON 05'S FSM, and it sits exactly on 08's boundary. Lesson 07 also found that
     heap_page_prune deliberately does NOT update the FSM after recovering space ("The free space
     should be reused by UPDATEs to *this* page") -- so 08 should say which path DOES tell the FSM
     about a prune-recovered page, and whether a cold update can ever be routed back onto it.
 (3) FROM LESSON 05, unchanged: smgrextend / mdextend and whether the FSM fork's 3-block floor is a
     consequence of extension or of fsm_vacuum.
Take: src/backend/storage/smgr/README and smgr.c (the f_smgr indirection table and why it exists
with exactly one implementation); md.c in full -- mdopenfork/mdextend/mdzeroextend/mdread/mdwrite/
mdnblocks/mdtruncate/mdimmedsync, the _mdfd_getseg segment map and _fdvec_resize; the SYNC REQUEST
QUEUE in storage/sync/sync.c (RegisterSyncRequest, ForgetRelationSyncRequests, the checkpointer's
AbsorbSyncRequests) which is the part nobody reads and the part that makes checkpoints correct;
RelationGetBufferForTuple in hio.c WITH its extension batching; and ExtendBufferedRelBy /
ExtendBufferedRelTo in bufmgr.c (the EB_CREATE_FORK_IF_NEEDED / EB_CLEAR_SIZE_CACHE flags lesson 06
quoted without reading). bufmgr.c is also lesson 09's territory: 08 should read ONLY the extension
entry points and leave BufferDesc, the clock sweep and the strategy ring alone.
MEASURE IT. The recipe below works and has now worked five runs running. Good candidates, in rough
order of value: (a) close question (1) by instrumenting extension -- watch pg_relation_size grow in
steps larger than one block under concurrent inserters, and find the batch size; (b) cross a 1 GB
SEGMENT BOUNDARY for the first time in this track -- lesson 01 derived RELSEG_SIZE 131072 and the
32,767-segment ceiling on paper and NOTHING has observed a `.1` file yet, and ~31 GB of disk
headroom was available this run, so a 1 GB+ unlogged table is feasible and would turn lesson 01's
arithmetic into an observation the way lesson 06 did for the VM boundary; (c) mdtruncate's
zero-length K+1st segment, also still unobserved; (d) pg_waldump the extension path to see whether
extension is WAL-logged at all (it is not, for the heap -- say so with evidence); (e) watch the
sync request queue with a forced checkpoint and pg_stat_bgwriter.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, the hole (02); the tuple header,
t_ctid's four meanings, the infomask catalogues (03); alignment, varlena, the 126-byte boundary,
TOAST (04); the FSM's categories, its on-page max-heap, fsm_logical_to_physical, the 3-block floor,
MaxHeapTuplesPerPage = 291 (05); ANY of the visibility map (06); AND NOW, FROM 07: LP_REDIRECT and
the lp_off unit change, the full line-pointer state machine, HEAP_HOT_UPDATED/HEAP_ONLY_TUPLE,
pd_prune_xid and PageSetPrunable, PD_PAGE_FULL, the Max(fillfactor, BLCKSZ/10) prune gate,
heap_page_prune's plan/execute split, heap_prune_chain's four outcomes, the HOT-eligibility test and
the bitwise-equality rule, the HOT-blocking matrix, and the prune/VACUUM dividing line.
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 2032, 1996, 8136, 126, 18, 4069,
4095, FSM_CAT_STEP 32, the FSM leaf offset 4123+k, the page fingerprint "00 20 04 20" at offsets
16-19 (NOT fork-specific -- say "special space is zero"), MAPSIZE 8168, HEAPBLOCKS_PER_PAGE 32672,
the VM offset formula 24 + (b % 32672)/4, bit shift (b % 4)*2, VISIBLE_MASK8 0x55 /
FROZEN_MASK8 0xaa, the 59-byte XLOG_HEAP2_VISIBLE record, the VM's one-block fork floor against the
FSM's three, and now: the 59/61-byte XLOG_HEAP2_PRUNE record, SizeOfHeapPrune 9, the 71-byte
HOT_UPDATE record, 34-byte COMMIT, 166 vs 247 WAL bytes per update (HOT vs cold), the 140-bytes-per
-row geometry (133 lp_len + MAXALIGN + 4) and the 58-at-ff100 / 40-at-ff70 rows-per-page it
predicts, and LESSON 07'S CORRECTION THAT LESSON 06'S "59 HEAP FETCHES" IS NOT A CONSTANT (it is
rows-on-the-old-block + 1; measured 58 on lesson 07's table).
Do NOT take: the buffer manager, BufferDesc, clock sweep, the strategy ring (09); pins vs content
locks and LWLocks (10); the heavyweight lock manager (11); tuple lock strengths (12); hint bits and
their FPI machinery (15, 22) -- lesson 07 found MarkBufferDirty and MarkBufferDirtyHint side by
side in heap_page_prune and taught the CONTRAST only; snapshots and HeapTupleSatisfiesMVCC (14);
freezing and relfrozenxid (18); VACUUM's two heap passes, the dead-TID store, index bulk deletion
(19); nbtree (28/29).
OPEN ITEMS carried forward:
- FOR LESSON 08: the three inherited questions above.
- FOR LESSON 14: lesson 07 used GlobalVisState / vistest and heap_prune_satisfies_vacuum as named
  black boxes. NOTE A LIVE XXX IN THE SOURCE worth chasing there: heap_page_prune_opt opens with
  "XXX: Magic to keep old_snapshot_threshold tests appear 'working'. They currently are broken, and
  discussion of what to do about them is ongoing", citing
  postgresql.org/message-id/20200403001235.e6jfdll3gh2ygbuc%40alap3.anarazel.de, and calls
  SnapshotTooOldMagicForTest() when old_snapshot_threshold == 0. The whole old_snap_xmin /
  old_snap_ts / old_snap_used plumbing threaded through PruneState exists for that feature.
- FOR LESSON 19 (unchanged from 05/06, still not closed): fsm_vacuum_page() and
  FreeSpaceMapVacuumRange() have STILL not been read -- they are the only code that makes the upper
  FSM levels true. Plus lesson 06's 18,400 WAL bytes over 118 blocks (155.9/block) breakdown. AND
  NEW FROM 07: VACUUM is the ONLY path that takes LP_DEAD -> LP_UNUSED and the only path that
  removes an index entry; lesson 07 measured the index going 19 -> 18 entries at VACUUM and 0
  change at prune, and the two heap passes are what 19 owes the reader.
- FOR LESSON 29: lesson 07's cold-update WAL run emitted 74 Btree/DELETE records unprompted --
  nbtree deletion defending its own leaves. 29 owns that; 07 only noted it.
- FOR LESSON 31: pg_class.relallvisible comes from visibilitymap_count (06); the index-only-scan
  cost formula that reads it is still not located. AND NEW: the planner has no notion of HOT, so
  lesson 07's 58-vs-49 Heap Fetches difference is invisible to cost estimation -- worth checking
  whether anything in costsize.c models it.
- UNOBSERVED, for whoever gets there first: MaxHeapTuplesPerPage = 291 has been DERIVED three times
  (05, 07) and never reached. Build a page with hundreds of LP_DEAD stubs and find where PageAddItem
  refuses.
- UNOBSERVED BRANCH, for whichever lesson runs with checksums on first: every measurement in
  lessons 03-07 was taken with data_checksums OFF. Two paths are still unexercised:
  visibilitymap_prepare_truncate's XLogHintBitIsNeeded() -> log_newpage_buffer (06), and
  heap_page_prune's MarkBufferDirtyHint branch under XLogHintBitIsNeeded() (07).
- UNEXERCISED CODE PATH: heap_prune_chain's outcome (4), the orphaned-redirect self-heal
  (nchain < 2 && ItemIdIsRedirected(rootlp)). Reachable only via a specific visit order.
  coverage.postgresql.org/src/backend/access/heap/pruneheap.c.func.html will say whether the
  regression suite reaches it.
- NOT MEASURED THIS RUN, and lesson 07 said so in the document: the SUMMARIZING-INDEX path. Every
  HOT/cold verdict used btree. A BRIN index on the updated column should give n_tup_hot_upd + 1
  AND a revised summary; README.HOT admits the propagation is pessimistic ("Realistically, we only
  need to propagate the update to the indexes that contain the updated values, but that is yet to
  be implemented"). Any later lesson that wants a cheap, high-value measurement can take this.
- A DOCUMENTATION PATCH WORTH PROPOSING UPSTREAM, unchanged from 06: visibilitymap.c's LOCKING
  section still says "only VACUUM currently sets visibility map bits", which heap_multi_insert's
  all_frozen_set contradicts.

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 04's note was dropped and TRACK-ARCHIVE.md gained the row
"PG 04" pointing at commit fffcd6bafd24c05eb83f0e2e46d0efab0597d880 / blob
9fb0a858ff439a121aa72abf94e642c6fb06bc4b, which is the last state of TRACK.md carrying it. This
block now holds 05, 06 and 07 -- exactly the three the policy allows. WHEN LESSON 08 SHIPS IT MUST
DROP THE LESSON 05 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap covered lesson 05; pg_visibility covers 06.
  Lesson 01's hands-on installs all three; lesson 02's needs only pageinspect; lesson 05's needs
  pageinspect and pg_freespacemap; lesson 07's needs pageinspect and pgstattuple.
- CONFIRMED 2026-10-22, 2026-10-23, 2026-10-24, 2026-10-25 AND AGAIN 2026-10-26: the cloud sandbox
  CAN run a cluster.
  psql, pg_config and /usr/lib/postgresql/16/bin/{initdb,postgres,pg_ctl,...} are in the image
  (PostgreSQL 16.15 on Ubuntu 24.04), and pageinspect, pg_freespacemap, pg_visibility, pgstattuple
  and amcheck are all present in /usr/share/postgresql/16/extension/. Recipe that worked: create a
  non-root user, initdb -D /var/tmp/pgdata --no-sync as that user, pg_ctl with
  -o '-p 5455 -k /var/tmp'. TWO TRAPS: initdb refuses to run as root, and it refuses a data
  directory under the scratchpad path (permission denied) -- /var/tmp works. A third, minor, found
  this run: psql cannot read a -f script file living under the scratchpad path either (permission
  denied as the pg user), so put scratch SQL somewhere world-readable such as /var/tmp. Every
  lesson from here on SHOULD measure; say so when it cannot. The cluster is per-run and does not
  persist, so table names from earlier runs are gone.
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04), pg5_* (05), pg6_* (06),
  pg7_* (07 -- the full lists are in the lesson 05, 06 and 07 notes above). Keep the pgNN_ prefix
  convention.
- pg_freespacemap was used by lesson 05 and pg_visibility by lesson 06 (both present in
  /usr/share/postgresql/16/extension/, both installed cleanly).
- pg_waldump lives at /usr/lib/postgresql/16/bin/pg_waldump, works on the live cluster's segments,
  and `--stats=record` prints a per-record-type record/FPI byte breakdown. Lesson 06 used it to
  price XLOG_HEAP2_VISIBLE and lesson 07 to price XLOG_HEAP2_PRUNE against Btree/INSERT_LEAF.
  Lessons 21-25 should use it rather than reasoning about WAL on paper. METHOD THAT WORKS WELL,
  from lesson 07: pg_switch_wal() first, record pg_current_wal_lsn(), run the workload, record the
  LSN again, then `pg_waldump -s <lsn1> -e <lsn2> --stats=record` for the table and
  `-r <rmgr>` for verbatim records. That isolates one workload's WAL exactly.
- *** GETTING THE SOURCE: `apt-get source postgresql-16` IS THE RIGHT ROUTE IN THIS SANDBOX AND
  LESSON 07 ESTABLISHED IT. *** /etc/apt/sources.list has no deb-src lines by default; write
  `deb-src http://archive.ubuntu.com/ubuntu/ noble main universe` (plus noble-updates) to
  /etc/apt/sources.list.d/, `apt-get update`, then `apt-get source postgresql-16` into a
  world-readable dir. ~33 MB, a few seconds, and it unpacks postgresql-16-16.15/ WITH debian/patches/
  applied -- so CHECK which files the patch series touches before quoting anything (extract the
  +++/--- paths; as of 16.15 none of them touches access/heap, storage/ or include/access). This is
  strictly better than WebFetching files one at a time: it is the exact build the cluster runs, it
  allows reading a file IN FULL, and WebFetch on postgresql.org summarises rather than reproducing.
  The Sources section must still cite only URLs actually seen that run; the local tree is evidence,
  not a citation.
- pg_buffercache is needed from lesson 09; pgstattuple was used by
  lesson 07 and is needed again by 19; amcheck for lessons 28-29 (it is the closest thing PostgreSQL has to SQLite's
  integrity_check, and the comparison is worth making explicitly when lesson 28 arrives).
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
