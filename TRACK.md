<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 1
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 01 shipped 2026-10-20.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 02**.

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
- [ ] 02 — The page: `PageHeaderData`'s 24 bytes, `pd_lower`/`pd_upper`, the line pointer array, and special space
- [ ] 03 — `HeapTupleHeaderData`: `t_xmin`, `t_xmax`, `t_ctid`, `t_infomask`/`t_infomask2`, and `t_hoff`
- [ ] 04 — Attribute storage: MAXALIGN padding, the null bitmap, 1-byte vs 4-byte varlena headers, and TOAST
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

LEFT FOR LATER, deliberately: PageHeaderData and the 15-bit lp_off/lp_len that caps BLCKSZ at
2^15 (lesson 02 -- lesson 01 mentions the cap in one line and explicitly defers); TOAST naming and
reltoastrelid (04); FSM/VM *contents* (05, 06); smgr's fsync request queue, register_dirty_segment,
relation extension locking and mdzeroextend/FileFallocate (08); fd.c's virtual FD mechanism beyond
naming it (08); ItemPointerData's on-disk width (03).

-- HANDOVER TO LESSON 02 -----------------------------------------------------------------------
Scope for lesson 02 (the page). Take: PageHeaderData field by field -- pd_lsn, pd_checksum,
pd_flags, pd_lower, pd_upper, pd_special, pd_pagesize_version, pd_prune_xid -- the 24-byte figure
and whether this build agrees; ItemIdData (lp_off:15, lp_flags:2, lp_len:15) and the four
lp_flags states; the line pointer array growing up from pd_lower while tuples grow down from
pd_upper; special space at pd_special and what index AMs put there; PageIsEmpty/PageGetFreeSpace;
page checksums (data_checksums, pd_checksum, and the fact that it is verified on read into the
buffer pool, not on every access -- the CMU mmap paper in this run's digest is the reason that
distinction matters). Primary sources: docs 66.6 Database Page Layout
(https://www.postgresql.org/docs/current/storage-page-layout.html),
src/include/storage/bufpage.h, src/include/storage/itemid.h,
src/backend/storage/page/README and bufpage.c.
Entry point from lesson 01: lesson 01 ends by telling the reader to hexdump the first 64 bytes of
block 0 WITHOUT interpreting them. Lesson 02 should open by interpreting exactly those bytes.
Do NOT re-explain: what a fork is, how a path is built, what a segment is, or that relfilenode
differs from OID. Reference lesson 01 by relative path instead.
Do NOT re-teach from the SQLite track: what a page is in the abstract, slotted-page layout as a
concept, or cell pointer arrays. The contrast worth drawing is specific: SQLite's page header is
8 or 12 bytes and carries a cell count; PostgreSQL's is 24 and carries an LSN. Say why.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, heap_page_items, bt_page_items,
  bt_metap, fsm_page_contents). pg_freespacemap and pg_visibility cover lessons 05 and 06.
  Lesson 01's hands-on installs all three.
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
