<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 32
lessons-done: 2
-->

# Current Track: SQLite Internals

Sequential deep dive. One lesson per day, each building on the last.

The order follows SQLite's actual layering (see sqlite.org/arch.html and fileformat2.html): the on-disk format and b-tree structures first, then the pager, journals, and VFS, then concurrency and WAL, then the SQL compiler and bytecode engine, then operational and advanced topics.

## Syllabus

### Part I — On-disk format and the b-tree layer (`btree.c`)
- [x] 01 — The database file header and how lockBtree() bootstraps page 1
- [x] 02 — Varints, serial types, and the record format
- [ ] 03 — The b-tree page header, cell pointer array, and the four cell layouts
- [ ] 04 — In-page space management: freeblocks, fragmented bytes, and defragmentation
- [ ] 05 — Cell payload overflow: maxLocal/minLocal/maxLeaf and overflow page chains
- [ ] 06 — The freelist: trunk and leaf pages, and how allocateBtreePage() and freePage2() recycle pages
- [ ] 07 — The sqlite_schema table, root page numbers, and schema loading (sqlite3InitOne)
- [ ] 08 — Table b-trees: rowids, INTEGER PRIMARY KEY aliasing, and WITHOUT ROWID tables
- [ ] 09 — Index b-trees: key records, record sort order, and covering-index lookups
- [ ] 10 — BtCursor navigation: moveToChild, table and index seeks, and cursor save/restore
- [ ] 11 — Insertion and page splits: balance(), balance_nonroot() and balance_deeper()
- [ ] 12 — Deletion, underflow, and rebalancing sibling pages
- [ ] 13 — Auto-vacuum: pointer-map (ptrmap) pages, page relocation, and incremental_vacuum

### Part II — The pager, journals, and the OS interface (`pager.c`, `pcache*.c`, `os_unix.c`)
- [ ] 14 — The page cache: PgHdr, pcache.c and pcache1.c, and dirty-page lists
- [ ] 15 — The pager state machine and the five lock states (SHARED/RESERVED/PENDING/EXCLUSIVE)
- [ ] 16 — The rollback journal file format and the single-file atomic commit sequence
- [ ] 17 — Hot journals, crash recovery, and super-journals for multi-database commits
- [ ] 18 — The VFS: sqlite3_vfs and sqlite3_io_methods, POSIX advisory locks in os_unix.c, and the lock-byte page

### Part III — Write-ahead logging, revisited in depth (`wal.c`)
- [ ] 19 — WAL frame append, wal-index hash tables, and read-mark slots
- [ ] 20 — Checkpoint algorithm, wal_autocheckpoint, and WAL reset/restart

### Part IV — The SQL compiler and bytecode engine
- [ ] 21 — From SQL text to parse tree: tokenize.c and the Lemon-generated parse.y
- [ ] 22 — Name resolution and the Expr/Select trees (resolve.c)
- [ ] 23 — The VDBE: bytecode programs, registers, and Mem cells (vdbe.c, vdbemem.c)
- [ ] 24 — Type affinity, collating sequences, and value comparison
- [ ] 25 — Code generation for INSERT/UPDATE/DELETE: OP_MakeRecord, OP_Insert, and index maintenance
- [ ] 26 — The query planner: WhereLoop objects, cost estimation, and the path solver (where.c)
- [ ] 27 — ANALYZE statistics: sqlite_stat1 and sqlite_stat4 and how the planner uses them
- [ ] 28 — Sorting and temporary storage: the VDBE sorter (vdbesort.c) and external merge sort

### Part V — Operational and advanced topics
- [ ] 29 — Transactions at the SQL level: autocommit, savepoints, and statement journals
- [ ] 30 — Virtual tables: the xBestIndex protocol, plus sqlite_dbpage and dbstat introspection
- [ ] 31 — Memory-mapped I/O and memory allocation (mmap_size, memsys, lookaside)
- [ ] 32 — Verifying a database: how PRAGMA integrity_check walks the file, and SQLite's test strategy

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
