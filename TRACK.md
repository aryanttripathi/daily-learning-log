<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 8
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 08 shipped 2026-10-27.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 09**.

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
  *** ANSWERED BY LESSON 08 (2026-10-27): RelationAddBlocks() in hio.c multiplies the pages it
  needs by (1 + the number of extension-lock waiters) and takes Max() with
  bistate->already_extended_by, which ACCUMULATES, so a COPY ramps 10, 10, 20, 40, 64, 64, ...
  up to MAX_BUFFERS_TO_EXTEND_BY = 64 and the surplus goes to the FSM or to bistate->next_free.
  Measured cost of the overshoot on a 20,000-row COPY: 2,896 blocks against 2,858. ***
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

-- HANDOVER TO LESSON 09 -----------------------------------------------------------------------
Scope for lesson 09, "The buffer manager: BufferDesc, the clock-sweep victim search, and the
strategy ring". LESSON 08 STOPPED EXACTLY AT 09'S DOOR AND NAMED THE HANDLES IT LEFT THERE, so 09
should open from them rather than from first principles.
THE INHERITED QUESTIONS, in descending order of value:
 (1) *** GetVictimBuffer() WAS THE BLACK BOX AT THE CENTRE OF LESSON 08'S BEST MEASUREMENT. ***
     ExtendBufferedRelShared() calls it extend_by times BEFORE taking the extension lock, zeroes
     each buffer with MemSet, and the comment says why ("Writing out victim buffers is the most
     expensive part of extending the relation, particularly when doing so requires WAL flushes").
     Lesson 09 owns the clock sweep that answers it, and should price that claim.
 (2) THE bulkwrite RING. Lesson 08 measured, via pg_stat_io, that a COPY of 14,288 blocks reported
     extends=14288 AND writes=12240 in the `bulkwrite` context -- i.e. the ring evicted 86% of what
     it had just created, inside one statement. BufferAccessStrategy / BAS_BULKWRITE /
     IOContextForStrategy are 09's, and that pair of numbers is a ready-made experiment.
 (3) LimitAdditionalPins()'s arithmetic (NBuffers / (MaxBackends + NUM_AUXILIARY_PROCS), minus
     PrivateRefCountOverflowed + REFCOUNT_ARRAY_ENTRIES, floored at 1) reduced a 64-block
     extension to ONE BLOCK at shared_buffers=128kB -- measured. The pin accounting itself is
     lesson 10's, but 09 should at least say what a pin costs and why a backend can only hold a
     proportional share.
Take: src/backend/storage/buffer/README IN FULL (it is the design document for the whole part and
nothing has read it yet); buf_internals.h (BufferDesc, the packed buf_state and its BM_* flags,
BufferTag, the buffer mapping table's partitions); freelist.c (StrategyGetBuffer, the clock sweep,
nextVictimBuffer, StrategyNotifyBgWriter, and the BufferAccessStrategy ring types); and the
bufmgr.c core -- ReadBuffer_common, BufferAlloc, GetVictimBuffer, FlushBuffer, and the
StartBufferIO/TerminateBufferIO protocol. pg_buffercache IS THE OBSERVATION TOOL and has not been
installed in any run yet -- it is present in /usr/share/postgresql/16/extension/.
MEASURE IT. Good candidates: (a) watch nextVictimBuffer sweep by sampling pg_buffercache's
usagecount histogram under a scan; (b) prove the ring is a ring -- a seqscan of a table larger than
shared_buffers should leave only ~32 blocks of it cached (BAS_BULKREAD's 256 kB), and a COPY should
show the eviction lesson 08 measured; (c) find the usagecount ceiling (BM_MAX_USAGE_COUNT = 5) by
re-reading one block repeatedly; (d) measure what a cache-miss read costs against a hit with
EXPLAIN (BUFFERS) and pg_stat_io's reads/hits; (e) pg_buffercache_summary / _usage_counts if the
version in 16 has them.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, the hole (02); the tuple header,
t_ctid's four meanings, the infomask catalogues (03); alignment, varlena, the 126-byte boundary,
TOAST (04); the FSM's categories, its on-page max-heap, fsm_logical_to_physical, the 3-block floor
(05); ANY of the visibility map (06); LP_REDIRECT and the lp_off unit change, the line-pointer state
machine, HEAP_HOT_UPDATED/HEAP_ONLY_TUPLE, pd_prune_xid, PD_PAGE_FULL, the Max(fillfactor, BLCKSZ/10)
prune gate, heap_page_prune's plan/execute split, the HOT-blocking matrix, the prune/VACUUM dividing
line (07); AND NOW, FROM 08: the smgr switch and smgrsw[], the MdfdVec segment map, _mdfd_getseg's
targetseg arithmetic, mdnblocks's walk, the RELSEG_SIZE invariant and inactive zero-length segments,
mdzeroextend's fallocate-vs-FileZero branch and the >8 cutoff, PG_IOV_MAX 32, the
MAX_BUFFERS_TO_EXTEND_BY 64 ramp, skipFsync's meaning, FileTag's four fields, and the whole fsync
request queue (register_dirty_segment -> ForwardSyncRequest -> AbsorbSyncRequests -> pendingOps ->
ProcessSyncRequests).
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 126, 4069, 4095, FSM_CAT_STEP 32,
the FSM leaf offset 4123+k, the fingerprint "00 20 04 20" at offsets 16-19 (= zero special space,
NOT "heap"), MAPSIZE 8168, HEAPBLOCKS_PER_PAGE 32672, the VM offset formula 24 + (b % 32672)/4,
VISIBLE_MASK8 0x55 / FROZEN_MASK8 0xaa, the 59-byte XLOG_HEAP2_VISIBLE record, the 59/61-byte
XLOG_HEAP2_PRUNE record, the 71-byte HOT_UPDATE record, 166 vs 247 WAL bytes per update (HOT vs
cold), the 140-bytes-per-row geometry, lesson 07's correction that lesson 06's "59 heap fetches" is
rows-on-the-old-block + 1, and now: RELSEG_SIZE observed as 1,073,741,824 bytes of segment 0,
MAX_BUFFERS_TO_EXTEND_BY 64, PG_IOV_MAX 32, max_requests = Min(NBuffers, 10000000),
FSYNCS_PER_ABSORB 10, and the ~1,940:1 writes-to-fsyncs dedup ratio.
Do NOT take: pins vs content locks and the LWLock implementation (10) -- 09 may name
LockBufHdr/UnlockBufHdr but must not teach LWLocks; the heavyweight lock manager (11); tuple lock
strengths (12); hint bits and their FPI machinery (15, 22); snapshots (14); the checkpointer's
spread writes and the REDO pointer (24) -- lesson 08 touched ProcessSyncRequests only, and
BufferSync belongs to 24; crash recovery (25).
OPEN ITEMS carried forward:
- FOR LESSON 09: the three inherited questions above.
- FOR LESSON 10: LimitAdditionalPins's REFCOUNT_ARRAY_ENTRIES / PrivateRefCountOverflowed.
- FOR LESSON 14: unchanged from 07 -- GlobalVisState / vistest and heap_prune_satisfies_vacuum were
  black boxes, and heap_page_prune_opt carries a live XXX about old_snapshot_threshold citing
  postgresql.org/message-id/20200403001235.e6jfdll3gh2ygbuc%40alap3.anarazel.de.
- FOR LESSON 19 (unchanged from 05/06/07, still not closed): fsm_vacuum_page() and
  FreeSpaceMapVacuumRange() have STILL not been read -- they are the only code that makes the upper
  FSM levels true, AND they are the answer to lesson 07's open question about whether a cold update
  can be routed onto a prune-recovered page. Plus lesson 06's 18,400 WAL bytes over 118 blocks
  breakdown, and VACUUM's two heap passes. NEW FROM 08: register_unlink_segment, pendingUnlinks and
  SyncPostCheckpoint() were read but not exercised -- the one-checkpoint delay before a dropped
  relation's file is actually unlinked is observable with DROP TABLE + two CHECKPOINTs.
- FOR LESSON 24: lesson 08 read ProcessSyncRequests and left BufferSync alone. The race the comment
  describes ("a buffer that must be written and fsync'd for the checkpoint could have been dumped by
  a backend just before it was visited by BufferSync()") and the Absorb-after-BufferSync ordering
  that closes it are 24's to teach. Also ckpt_longest_sync: measured spread 0.095-82.894 ms per
  file, and nothing bounds it.
- FOR LESSON 29: unchanged -- lesson 07's cold-update WAL run emitted 74 Btree/DELETE records.
- FOR LESSON 31: unchanged -- pg_class.relallvisible's cost formula, and whether costsize.c models
  HOT at all.
- UNOBSERVED, for whoever gets there first: MaxHeapTuplesPerPage = 291 has been DERIVED three times
  (05, 07) and never reached.
- UNOBSERVED BRANCH, for whichever lesson runs with checksums on first: every measurement in
  lessons 03-08 was taken with data_checksums OFF. Still unexercised:
  visibilitymap_prepare_truncate's XLogHintBitIsNeeded() -> log_newpage_buffer (06), and
  heap_page_prune's MarkBufferDirtyHint branch under XLogHintBitIsNeeded() (07).
- UNEXERCISED CODE PATHS: heap_prune_chain's outcome (4), the orphaned-redirect self-heal (07); and
  NEW FROM 08, register_dirty_segment's queue-full local-fsync fallback, plus
  ExtendBufferedRelShared's `existing_id >= 0` branch with its
  errhint("This has been seen to occur with buggy kernels") -- both read, neither reached.
- NOT MEASURED, carried from 07: the SUMMARIZING-INDEX (BRIN) HOT path.
- NEW FROM 08, worth one cheap run: truncate a relation to an exact multiple of RELSEG_SIZE and
  check whether segment K+1 is kept at length 0, as mdtruncate's comment promises.
- A DOCUMENTATION PATCH WORTH PROPOSING UPSTREAM, unchanged from 06: visibilitymap.c's LOCKING
  section still says "only VACUUM currently sets visibility map bits", which heap_multi_insert's
  all_frozen_set contradicts.

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 05's note was dropped and TRACK-ARCHIVE.md gained the row
"PG 05" pointing at commit 5d3a81e3c7b92c6679f964fb9354092862b4c6a3 / blob
54c92e672c6081eec9935d5f6f8514ad8520a7ed, which is the last state of TRACK.md carrying it. This
block now holds 06, 07 and 08 -- exactly the three the policy allows. WHEN LESSON 09 SHIPS IT MUST
DROP THE LESSON 06 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap covered lesson 05; pg_visibility covers 06.
  Lesson 01's hands-on installs all three; lesson 02's needs only pageinspect; lesson 05's needs
  pageinspect and pg_freespacemap; lesson 07's needs pageinspect and pgstattuple. Lesson 08 needed
  only pageinspect (and barely that -- its instrument was strace).
- CONFIRMED 2026-10-22 THROUGH 2026-10-27 (six runs): the cloud sandbox CAN run a cluster.
  *** TWO RECIPES, BOTH WORKING, AND THE SECOND IS SIMPLER. ***
  (A) Lessons 03-07's: create a non-root user, initdb -D /var/tmp/pgdata --no-sync as that user,
      pg_ctl with -o '-p 5455 -k /var/tmp'. TWO TRAPS: initdb refuses to run as root, and it
      refuses a data directory under the scratchpad path -- /var/tmp works.
  (B) Lesson 08's, fewer steps: `apt-get install postgresql postgresql-contrib`, then
      `pg_ctlcluster 16 main start`, then run everything as `su postgres -c "psql ..."`. The data
      dir is /var/lib/postgresql/16/main, the log is /var/log/postgresql/postgresql-16-main.log,
      ALTER SYSTEM + `pg_ctlcluster 16 main restart` applies PGC_POSTMASTER settings, and
      `ALTER SYSTEM SET log_min_messages='debug1'; SELECT pg_reload_conf();` is how you get
      checkpoint-sync and other DEBUG1 lines into that log. Remember to set it back to 'warning'.
  BOTH recipes keep the world-readable-script trap: psql -f cannot read a file under the scratchpad
  path as the pg user, so put scratch SQL in /tmp/<dir> with chmod 777 (or /var/tmp).
  The cluster is per-run and does not persist, so table names from earlier runs are gone.
- *** strace IS AVAILABLE AND IT IS THE RIGHT INSTRUMENT FOR ANYTHING BELOW THE BUFFER MANAGER.
  gdb is installed too (no debug symbols, so it cannot read shared-memory structs). THE HARNESS
  LESSON 08 BUILT AND LATER LESSONS SHOULD COPY: mkfifo a pipe, start
  `su postgres -c "psql -d DB -At -q < fifo > out"` in the background, write
  "SELECT pg_backend_pid();" into the fifo, read the pid out of the output file, then
  `strace -f -tt -p <pid> -e trace=<syscalls> -o trace.txt &`, then feed the real SQL into the fifo
  and poll for a DONE marker. That gives a clean per-statement syscall trace of a NAMED backend.
  Note that parallel workers are NOT children of that backend, so -f does not follow them; disable
  parallelism or trace the postmaster if you need them. ***
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04), pg5_* (05), pg6_* (06),
  pg7_* (07), and 08's unprefixed set (seg, ext_*, tiny_copy, q1..q4, dedup, b, io_t, iostat_t --
  a lapse; go back to pgNN_*). Keep the pgNN_ prefix convention.
- pg_waldump lives at /usr/lib/postgresql/16/bin/pg_waldump, works on the live cluster's segments,
  and `--stats=record` prints a per-record-type record/FPI byte breakdown. Lesson 06 used it to
  price XLOG_HEAP2_VISIBLE and lesson 07 to price XLOG_HEAP2_PRUNE against Btree/INSERT_LEAF.
  Lessons 21-25 should use it rather than reasoning about WAL on paper. METHOD THAT WORKS WELL,
  from lesson 07: pg_switch_wal() first, record pg_current_wal_lsn(), run the workload, record the
  LSN again, then `pg_waldump -s <lsn1> -e <lsn2> --stats=record` for the table and
  `-r <rmgr>` for verbatim records. That isolates one workload's WAL exactly.
- *** GETTING THE SOURCE: `apt-get source postgresql-16` IS THE RIGHT ROUTE IN THIS SANDBOX AND
  LESSONS 07 AND 08 BOTH USED IT. *** Ubuntu 24.04 ships deb-src DISABLED in
  /etc/apt/sources.list.d/ubuntu.sources (a deb822 file with `Types: deb`); the fix that worked in
  lesson 08 is `sed -i 's/^Types: deb$/Types: deb deb-src/'` on that file, then `apt-get update`,
  then `apt-get source postgresql-16` into a world-readable dir. ~33 MB, a few seconds, and it
  unpacks postgresql-16-16.15/ WITH debian/patches/ applied -- so CHECK WHICH FILES THE PATCH SERIES
  TOUCHES BEFORE QUOTING ANYTHING, per file and not in bulk: as of 16.15 there are 15 patches and
  one of them DOES touch src/backend/storage/buffer/bufmgr.c (at LockBufferForCleanup only).
  Extract the +++/--- paths and check the hunk offsets. This is strictly better than WebFetching
  files one at a time: it is the exact build the cluster runs, it allows reading a file IN FULL, and
  WebFetch on postgresql.org summarises rather than reproducing. The Sources section must still
  cite only URLs actually seen that run; the local tree is evidence, not a citation.
- pg_buffercache is needed from lesson 09 and has NOT been installed in any run yet (present in
  /usr/share/postgresql/16/extension/); pgstattuple was used by lesson 07 and is needed again by
  19; amcheck for lessons 28-29 (it is the closest thing PostgreSQL has to SQLite's
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
