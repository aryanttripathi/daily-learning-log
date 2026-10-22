<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 3
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 03 shipped 2026-10-22.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 04**.

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
- [ ] 04 — Attribute storage: MAXALIGN padding between attributes, 1-byte vs 4-byte varlena headers, `atthasmissing`, and TOAST
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

-- LESSON 01 (2026-10-20) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-20-postgres-cluster-on-disk-forks-segments/README.md (single README,
not split). Source commit read: postgres/postgres@ddd468ca (note: the syllabus was built against
@45fc51fc; the tree moved between those two runs, so later lessons should state their own commit
rather than assume either).

TOOK, and must not be re-taught:
- RelFileLocator {spcOid, dbOid, relNumber}; RelFileNumber is a *typedef of Oid*, so the compiler
  cannot catch an OID/filenode mix-up. relfilenode != OID; rewrites that change it: VACUUM FULL,
  CLUSTER, TRUNCATE, REINDEX, some ALTER TABLE. pg_class.relfilenode is 0 for pg_class itself;
  pg_relation_filenode() is the escape hatch. This was framed as the price of moving the catalog
  into real tables -- comparison axis 1 from the SQLite capstone. Do not re-argue it.
- GetRelationPath() in src/common/relpath.c: three tablespace shapes (global/, base/<dbOid>/,
  pg_tblspc/<spcOid>/<TABLESPACE_VERSION_DIRECTORY>/<dbOid>/) x temp/permanent x main/non-main =
  nine sprintf branches. GetDatabasePath() duplicates the three-way branch with an
  "XXX this must agree with GetRelationPath()!" comment. Returns RelPathStr BY VALUE, deliberately,
  so the pointer cannot decay and the string is safe in a critical section.
- forkNames[] = {"main","fsm","vm","init"} -- NO underscores in the array; the underscore comes
  from the "%u_%s" format string. That is why pg_relation_size(rel,'fsm') takes 'fsm' not '_fsm'.
  forkname_to_number()'s errhint enumerates the four. forkname_chars() loops from forkNum=1 and
  therefore cannot recognise "main", and documents the assumption that no fork name is a prefix of
  another. MAX_FORKNUM = INIT_FORKNUM = 3, FORKNAMECHARS = 4 with zero slack, StaticAssertDecl ties
  array length to the enum.
- Temp relations: "t%d_%u" where %d is procNumber (a backend SLOT, not getpid()); PROCNUMBER_CHARS
  is 6 because relpath.h states MAX_BACKENDS = 2^18-1. Shared catalogs assert dbOid == 0 and
  procNumber == INVALID_PROC_NUMBER, so a shared catalog can never be temporary.
- The init fork, at the level of docs 66.5 only: it is an empty relation of the right type, and
  crash recovery resets an unlogged relation by COPYING it over the main fork and erasing the rest.
- Segments, from md.c's header comment: the on-disk invariant (zero or more segments of exactly
  RELSEG_SIZE, exactly one partial, then optionally any number of ZERO-LENGTH inactive segments),
  blocknum / RELSEG_SIZE and blocknum % RELSEG_SIZE, md_seg_fds[forknum]/md_num_open_segs,
  mdmaxcombine() returning RELSEG_SIZE - segoff, the "read/write crosses segment boundary" elogs,
  mdtruncate() keeping a 0-length K+1st segment when nblocks is an exact multiple (and WHY: other
  backends' open descriptors), _mdfd_getseg()'s zero-padding of skipped segments with the
  "can happen in hash indexes" comment, _mdnblocks()'s walk and its elog(FATAL,"segment too big"),
  and the StaticAssertDecl on RELSEG_SIZE <= INT_MAX with the --with-segsize rationale.
- Numbers established and reusable: BLCKSZ 8192 (max 32768); RELSEG_SIZE 131072 blocks;
  MaxBlockNumber 0xFFFFFFFE = 4,294,967,294; max segments per fork = 32,767; max bytes per fork =
  35,184,372,072,448 = 32 TiB MINUS 16 KiB (two BlockNumber values are reserved, not one).

MEASUREMENTS: the hands-on was written and reasoned from source, NOT executed -- this run had no
PostgreSQL cluster. The document does not claim any figure as observed. If a later lesson needs a
measured number from Part I, it must run it. The install step (pageinspect, pg_freespacemap,
pg_visibility) is stated in lesson 01's hands-on; lessons 05-07 should not repeat it.

LEFT FOR LATER, deliberately: TOAST naming and reltoastrelid (04); FSM/VM *contents* (05, 06);
smgr's fsync request queue, register_dirty_segment, relation extension locking and
mdzeroextend/FileFallocate (08); fd.c's virtual FD mechanism beyond naming it (08);
ItemPointerData's on-disk width (03). (PageHeaderData and the 15-bit lp_off/lp_len cap, also on
this list, were taken by lesson 02 and are now closed.)

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

-- HANDOVER TO LESSON 04 -----------------------------------------------------------------------
Scope for lesson 04 (attribute storage). Take: how the bytes AFTER t_hoff are laid out and walked
-- att_align_nominal / att_nominal_alignby / att_pointer_alignby / att_datum_alignby and why there
are four, the TYPEALIGN macros and typalign_to_alignby's four cases (TYPALIGN_CHAR/SHORT/INT/
DOUBLE -> 1/ALIGNOF_SHORT/ALIGNOF_INT/ALIGNOF_DOUBLE); att_addlength_pointer's three cases
(attlen>0, -1 varlena, -2 cstring); the 1-byte vs 4-byte varlena headers and VARATT_IS_SHORT /
VARATT_CAN_MAKE_SHORT / SET_VARSIZE_SHORT / attispackable, including the PAD-BYTE PEEKING TRICK in
att_align_pointer's comment ("A zero byte must be either a pad byte, or the first byte of a
correctly aligned 4-byte length word") -- that comment is the single best thing in tupmacs.h and
lesson 04 owns it; why short varlenas are NOT aligned at all; attlen -2 cstrings never needing
alignment; heap_compute_data_size and the fact that it runs AFTER t_hoff is fixed, so attribute
padding is measured from an already-MAXALIGNed base (lesson 03's Next Step 5 points the reader at
this); column-order effects on row width as a measured experiment; atthasmissing / ADD COLUMN with
DEFAULT as the third NULL encoding (lesson 03 named and deferred it); then TOAST -- the four
storage strategies, the toast threshold and MaxHeapTupleSize/4, reltoastrelid and the
pg_toast.pg_toast_<oid> naming (lesson 01 deferred the naming and reltoastrelid explicitly and
lesson 04 owns them), varatt_external's fields, HEAP_HASEXTERNAL, and SET STORAGE PLAIN/EXTERNAL/
EXTENDED/MAIN.
Primary sources: src/include/access/tupmacs.h (READ THIS RUN for att_isnull only -- the align/
addlength macro family and typalign_to_alignby were read but NOT taught and must be),
src/include/varatt.h (NOT read yet), src/backend/access/common/heaptuple.c
(heap_compute_data_size and fill_val's per-attribute branches were read this run; heap_deform_tuple
and nocachegetattr were NOT), src/backend/access/common/detoast.c and
src/backend/access/heap/heaptoast.c (NOT read), docs 66.6 and the TOAST section of 66.2,
src/include/access/tupdesc.h for CompactAttribute (attbyval, attlen, attalignby, attispackable,
atthasmissing) -- note master uses CompactAttribute where older trees use Form_pg_attribute, which
is a real version difference worth stating.
Entry point from lesson 03: lesson 03's Next Step 1 tells the reader to re-run the hexdump
exercise on a table whose first column is text and try to find the attribute boundaries in the
data bytes -- and says they will fail, because a varlena header is 1 or 4 bytes depending on
length. Lesson 04 should answer exactly that, and the pad-byte peeking comment is the answer.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len packing, the 15-bit cap,
free-space arithmetic, checksums (lesson 02); the heap tuple header's fields, the two unions,
ItemPointerData's width or byte order, t_ctid's four meanings, the infomask/infomask2 bit
catalogues, natts being per-tuple, BITMAPLEN, t_hoff's MAXALIGN arithmetic or its measured step
function, the null bitmap's bit order or polarity (lesson 03). Cite lesson 03's numbers
(23, 24, the 24/32/40 steps, 8160, 8164, 291) -- do not re-derive them. Reference both lessons by
relative path.
Do NOT take: FSM/VM contents (05, 06); HOT, t_ctid-as-chain, pruning, LP_REDIRECT (07); smgr's
fsync queue and relation extension (08); visibility rules (14); hint bits (15); combo CIDs (16);
MultiXacts (17); freezing (18).
Lesson 04 SHOULD measure, as the first thing it does: the heap fingerprint bytes at page offsets
16-19 and the 8160 insert ceiling, both still unverified from lesson 02. Setup recipe is in the
LESSON 03 block above.

NOTE-BLOCK ROTATION DUE NEXT RUN: this block now holds 01, 02 and 03 -- exactly the three the
policy allows. When lesson 04 ships it MUST drop the LESSON 01 block, and before dropping it MUST
add a TRACK-ARCHIVE.md row recording the commit and blob that still carry lesson 01's note.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap and pg_visibility cover lessons 05 and 06.
  Lesson 01's hands-on installs all three; lesson 02's hands-on needs only pageinspect.
- CONFIRMED 2026-10-22: the cloud sandbox CAN run a cluster. psql, pg_config and
  /usr/lib/postgresql/16/bin/{initdb,postgres,pg_ctl,...} are in the image (PostgreSQL 16.15 on
  Ubuntu 24.04), and pageinspect, pg_freespacemap, pg_visibility, pgstattuple and amcheck are all
  present in /usr/share/postgresql/16/extension/. Recipe that worked: create a non-root user,
  initdb -D /var/tmp/pgdata --no-sync as that user, pg_ctl with -o '-p 5455 -k /var/tmp'. TWO
  TRAPS: initdb refuses to run as root, and it refuses a data directory under the scratchpad path
  (permission denied) -- /var/tmp works. Every lesson from here on SHOULD measure; say so when it
  cannot. The cluster is per-run and does not persist, so table names from earlier runs are gone.
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_fixed / pg3_null / pg3_upd / pg3_alter / pg3_cid / pg3_empty / pg3_blk /
  w8 / w9 / w16 / w72 / w73 (03). Keep the pgNN_ prefix convention.
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
