<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 0
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: starting.** The syllabus was written on 2026-10-19, the day the SQLite track's capstone
shipped. The first run after this one takes **lesson 01**.

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

- [ ] 01 — The cluster on disk: `base/<dboid>/<relfilenode>`, the three forks, and 1 GB segment files
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
The SQLite track's lessons 31-33 notes were archived on this run; see TRACK-ARCHIVE.md, row
"31 - 33". The capstone document (2026/10/2026-10-19-sqlite-capstone/README.md) closes the track
and records which items were deliberately left unread: FTS5, R-Tree, session/changesets,
sqlite3_recover, the WAL savepoint path (PagerSavepoint.aWalData, sqlite3WalSavepoint,
sqlite3WalSavepointUndo), sqlite3PagerMovepage's subjournal call site, sqlite3VdbeFindIndexKey,
sqlite_stat4-as-a-table, loadStatTbl's allocation cost, and sqlite3DefaultRowEst. None of these
carry forward into the PostgreSQL track. Do not hand them to a PostgreSQL lesson.

-- POSTGRESQL TRACK: HANDOVER TO LESSON 01 -----------------------------------------------------
Scope for lesson 01 (the cluster on disk). Take: the PGDATA directory layout, base/<dboid>/,
relfilenode versus OID and when they diverge (rewrites: VACUUM FULL, CLUSTER, TRUNCATE), the
three forks (main, _fsm, _vm) and the init fork, 1 GB segmentation and the .1/.2 suffixes,
pg_relation_filepath()/pg_relation_size()/pg_table_size(), and the fact that a 32-bit block
number times 8 KB caps a single relation fork at 32 TB. Measure: create a table, read its path
from pg_relation_filepath, verify the file size against pg_relation_size, force a rewrite and
show relfilenode changing while the OID does not, and push one relation past one segment to see
the .1 file appear. Primary sources: docs chapter 66.1 Database File Layout
(https://www.postgresql.org/docs/current/storage-file-layout.html) and
src/backend/storage/smgr/README.
Leave for later: TOAST tables and their indexes (lesson 04), the FSM and VM file *contents*
(lessons 05, 06), smgr's fsync queue and relation extension locking (lesson 08), and anything
about tablespaces beyond the symlink in pg_tblspc.
Do NOT re-teach from the SQLite track: what a page is, what a b-tree is, why a cost model
quantises, or how a journal works. Reference them by contrast instead -- the capstone's four
tradeoffs are the intended comparison axes.

Known environment facts for a fresh session (verify, do not assume):
- Default block size is 8 KB (compile-time BLCKSZ); segment size 1 GB (compile-time
  RELSEG_SIZE). Both are build options, so state the build you measured.
- pageinspect is the extension that makes Part I observable (page_header, heap_page_items,
  bt_page_items, bt_metap, fsm_page_contents). pg_freespacemap and pg_visibility cover lessons
  05 and 06. Install them in lesson 01 and say so, so later lessons do not repeat it.
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
