<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 16
-->

# Current Track: SQLite Internals

Sequential deep dive. One lesson per day, each building on the last.

The order follows SQLite's actual layering (see sqlite.org/arch.html and fileformat2.html): the on-disk format and b-tree structures first, then the pager, journals, and VFS, then concurrency and WAL, then the SQL compiler and bytecode engine, then operational and advanced topics.

## Syllabus

### Part I — On-disk format and the b-tree layer (`btree.c`)
- [x] 01 — The database file header and how lockBtree() bootstraps page 1
- [x] 02 — Varints, serial types, and the record format
- [x] 03 — The b-tree page header, cell pointer array, and the four cell layouts
- [x] 04 — In-page space management: freeblocks, fragmented bytes, and defragmentation
- [x] 05 — Cell payload overflow: maxLocal/minLocal/maxLeaf and overflow page chains
- [x] 06 — The freelist: trunk and leaf pages, and how allocateBtreePage() and freePage2() recycle pages
- [x] 07 — The sqlite_schema table, root page numbers, and schema loading (sqlite3InitOne)
- [x] 08 — Table b-trees: rowids, INTEGER PRIMARY KEY aliasing, and WITHOUT ROWID tables
- [x] 09 — Index b-trees: key records, record sort order, and covering-index lookups
- [x] 10 — BtCursor navigation: moveToChild, table and index seeks, and cursor save/restore
- [x] 11 — Insertion and page splits: balance(), balance_nonroot() and balance_deeper()
- [x] 12 — Deletion, underflow, and rebalancing sibling pages
- [x] 13 — Auto-vacuum: pointer-map (ptrmap) pages, page relocation, and incremental_vacuum

### Part II — The pager, journals, and the OS interface (`pager.c`, `pcache*.c`, `os_unix.c`)
- [x] 14 — The page cache: PgHdr, pcache.c and pcache1.c, and dirty-page lists
- [x] 15 — The pager state machine and the five lock states (SHARED/RESERVED/PENDING/EXCLUSIVE)
- [x] 16 — The rollback journal file format and the single-file atomic commit sequence
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
- [ ] 33 — VACUUM and VACUUM INTO: the temp-database rewrite in vacuum.c, the backup API it shares, and why it is the only way to change auto_vacuum mode

<!--
syllabus revision 2026-09-28 (lesson 13): lesson 33 added, lessons-total 32 -> 33.
Lesson 13 had to refer to VACUUM repeatedly as the mechanism that does what auto-vacuum
cannot (repacking partially filled pages, changing auto_vacuum mode, forensic erasure),
and no lesson covered vacuum.c. Placed after lesson 32 because both are whole-file
operations. No lesson was reordered, split, or removed.

no revision 2026-09-29 (lesson 14): the ordering held. Lesson 14 needed only btree-layer
material already covered and stopped exactly where lesson 15 (pager state machine) begins:
at the xStress -> pagerStress() call, whose legality depends on the pager state and the
doNotSpill flags that lesson 15 covers. Lesson 31 (memory allocation) remains the right
home for SQLITE_CONFIG_PAGECACHE bulk allocation and soft-heap-limit behaviour, which
lesson 14 deliberately left alone.

scope note 2026-09-30 (lesson 15): no lesson added, removed, reordered or split;
lessons-total stays 33. Lesson 15 necessarily consumed some os_unix.c material that
lesson 18 nominally owns: the PENDING_BYTE / RESERVED_BYTE / SHARED_FIRST offsets from
os.h:159-166 and the unixLock() escalation sequence, because the five lock levels cannot
be explained without saying which bytes they are. Lesson 18 should therefore NOT re-teach
those offsets and should instead take: the sqlite3_vfs / sqlite3_io_methods dispatch
tables and VFS registration; unixInodeInfo and the intra-process lock emulation that makes
two connections in one process behave correctly despite per-process POSIX record locks;
the alternative locking styles in os_unix.c (dot-file, flock, AFP, named-semaphore, nolock)
and when each is selected; xCheckReservedLock and its interaction with UNKNOWN_LOCK;
xSectorSize / xDeviceCharacteristics and the IOCAP flags (SAFE_APPEND, SEQUENTIAL,
BATCH_ATOMIC) that lessons 15 and 16 both reference but neither explains; and the lock-byte
PAGE as it appears in the b-tree address space (PENDING_BYTE_PAGE, btreeInt.h:612) rather
than the lock byte ranges themselves.

Lesson 15 also pulled forward one WAL observation (a WAL write transaction never raises the
database file above SHARED; write exclusion moves to the -shm file) purely to bound the
rollback-mode claims. Lessons 19-20 keep the -shm file, read-marks and checkpointing intact.

scope note 2026-10-01 (lesson 16): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held — lesson 16 needed only the pager states and lock
levels from lesson 15 and the dirty-list/spill mechanics from lesson 14.

Two scope adjustments to record:

(a) Lesson 16 took more of the IOCAP flags than lesson 15's note anticipated, because the
journal header format cannot be explained without them: SQLITE_IOCAP_SAFE_APPEND selects
the nRec=0xffffffff path in writeJournalHdr (pager.c:1532-1539) and suppresses the extra
journal header after a spill (pager.c:4431), and SQLITE_IOCAP_SEQUENTIAL gates both journal
syncs in syncJournal (pager.c:4409, 4421). SQLITE_IOCAP_POWERSAFE_OVERWRITE was also taken,
since it is why the measured sectorSize is 512 rather than 4096 (setSectorSize,
pager.c:2799-2813). Lesson 18 should therefore take only BATCH_ATOMIC and the
xSectorSize / xDeviceCharacteristics dispatch itself, and may reference the three flags
above as already covered.

(b) Lesson 16 produced a real hot journal (SIGKILL of a spilling writer), measured the
rollback end to end (md5 identical, file truncated to dbOrigSize x pageSize, 14 ms), and
established why a zeroed magic number makes a journal safe to ignore. Lesson 17 should NOT
re-teach the playback walk or re-run a basic recovery. It should take: hasHotJournal()'s
five conditions and the xCheckReservedLock interaction; the pagerSyncHotJournal step and
the cache-reset distinction between a genuinely hot journal and a merely persistent one
(the isHot parameter, pager.c:2865-2871); SQLITE_READONLY_ROLLBACK; the super-journal
format, writeSuperJournal, readSuperJournal (pager.c:1340-1382) and pagerIsSuperJrnlName;
and the multi-database commit protocol where the super-journal's deletion is the commit
point instead of the per-database journal's.
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
