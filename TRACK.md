<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 20
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
- [x] 17 — Hot journals, crash recovery, and super-journals for multi-database commits
- [x] 18 — The VFS: sqlite3_vfs and sqlite3_io_methods, POSIX advisory locks in os_unix.c, and the lock-byte page

### Part III — Write-ahead logging, revisited in depth (`wal.c`)
- [x] 19 — WAL frame append, wal-index hash tables, and read-mark slots
- [x] 20 — Checkpoint algorithm, wal_autocheckpoint, and WAL reset/restart

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

scope note 2026-10-02 (lesson 17): no lesson added, removed, reordered or split;
lessons-total stays 33. Lesson 17 took exactly the scope lesson 16's note (b) assigned it
and did not re-teach the playback walk. It read source at commit fde3a84d: pager.c
(1265-1316, 1318-1386, 1725-1816, 2570-2704, 2875-2935, 3010-3060, 4090-4111, 5180-5300,
5330-5470), vdbeaux.c (2918-3170) and sqlite.h.in (556-561).

Three scope adjustments to record:

(a) Lesson 17 necessarily took part of xCheckReservedLock that lesson 15's note assigned to
lesson 18 — specifically that sqlite3OsCheckReservedLock() is hasHotJournal()'s condition 5
and the ticket #3883 race around it (pager.c:5217-5225). It treated the call as a black box
and did NOT touch unixCheckReservedLock's implementation, the UNKNOWN_LOCK interaction, or
unixInodeInfo. Lesson 18 keeps all of those and should open the box lesson 17 only named.

(b) The multi-file commit protocol lives in the VDBE layer, not the pager, so lesson 17
necessarily read vdbeCommit() (vdbeaux.c:2930-3180) ahead of Part IV. What it took is narrow:
the aMJNeeded[] matrix and the safety_level/memdb test that decide nTrans (2962-2983), the
super-journal name generation (3056-3079), the per-journal CommitPhaseOne loop (3139-3144),
and the sqlite3OsDelete(..., 1) commit point (3156). Lessons 21-28 keep everything else about
the VDBE; lesson 29 (SQL-level transactions, savepoints, statement journals) keeps autocommit,
sqlite3VdbeHalt's surrounding logic, and the statement journal entirely — lesson 17 mentioned
the statement journal only as a forward reference and measured nothing about it.

(c) Lesson 17 established two things later lessons should build on rather than repeat: that
SQLITE_READONLY_ROLLBACK (776) makes a crashed database unreadable to a read-only connection,
and that aMJNeeded[] silently disables cross-file atomicity when any participant is in WAL,
MEMORY or OFF journal mode or at synchronous=OFF (measured as a torn delete+WAL transaction).
Lessons 19-20 should reference the WAL row of that matrix as already measured when discussing
what WAL gives up, and lesson 32 (integrity_check) should reference the measured finding that
a torn multi-file transaction leaves every participating file reporting integrity_check = ok,
since that bounds what integrity_check can be claimed to verify.

scope note 2026-10-03 (lesson 18): no lesson added, removed, reordered or split;
lessons-total stays 33. Lesson 18 took exactly the scope that lessons 15, 16 and 17's notes
assigned it: the sqlite3_vfs / sqlite3_io_methods dispatch tables and VFS registration;
unixInodeInfo and the intra-process lock emulation; the alternative locking styles
(dot-file, flock, named-semaphore, nolock, afp/proxy/nfs) and autolockIoFinder's selection
logic; xCheckReservedLock opened up, with UNKNOWN_LOCK; xSectorSize / xDeviceCharacteristics
dispatch plus BATCH_ATOMIC only; and the lock-byte PAGE (PENDING_BYTE_PAGE). It did NOT
re-teach the PENDING_BYTE / RESERVED_BYTE / SHARED_FIRST offsets (lesson 15) or the
SAFE_APPEND / SEQUENTIAL / POWERSAFE_OVERWRITE flags (lesson 16), referencing both as
already covered. Source read at commit fde3a84d: os_unix.c (423-427, 236-276, 521-528,
584-586, 1282-1365, 1471-1512, 1527-1618, 1660-1716, 1780-1840, 2099-2109, 2341-2400,
2443-2571, 4144-4157, 4330-4400, 4461-4482, 5800-6016, 6128-6207, 8460-8526), os.c (355-447),
pager.c (396-407, 627-634, 906-964, 1129-1173, 1187-1212, 2384-2390, 2764-2806, 3682-3689,
5018-5074, 5330-5395, 5631-5648, 6583-6731, 7622-7733), btreeInt.h (592-614), btree.c and
backup.c (PENDING_BYTE_PAGE sites only).

Four scope adjustments to record:

(a) Lesson 18 took sqlite3PagerWalSupported() (pager.c:7625-7629) and measured that its line
ordering is observable from SQL: locking_mode=EXCLUSIVE makes journal_mode=WAL succeed on
unix-dotfile and unix-none (exclusiveMode short-circuits the iVersion>=2 && xShmMap test),
while ?nolock=1 defeats WAL even in exclusive mode because noLock is tested first. It also
measured that a failed journal_mode=WAL returns "delete" with SQLITE_OK and no error.
Lessons 19-20 should treat that gate as already measured and should NOT re-derive it; they
keep the -shm file contents, the wal-index hash tables, read-marks, frame append and
checkpointing entirely. The heap-memory wal-index under exclusiveMode (pager.c:7663-7666)
was named only as the justification for the short-circuit and is still theirs to explain.

(b) Lesson 18 demonstrated PENDING_BYTE_PAGE by relocating PENDING_BYTE with
sqlite3_test_control(SQLITE_TESTCTRL_PENDING_BYTE, 0x2000) at page_size=1024 and showing
page 9 as 1024 zero bytes between 0x0d leaves. It also catalogued the ~two dozen sites that
special-case it. Lesson 32 (integrity_check) should reference the measured fact that
integrity_check marks PENDING_BYTE_PAGE as referenced (btree.c:11263-11264) so it is not
reported as a leak, rather than rediscovering it; lesson 33 (VACUUM / the backup API) keeps
backup.c's page-skipping (254, 265, 419, 479, 507, 519) and pager_truncate's adjustment.

(c) Lesson 18 touched xFetch / xUnfetch ONLY as far as measuring that the unix VFS maps the
database PROT_READ|MAP_SHARED (never writable), that reads then collapse to the pre-mmap
bootstrap preads while every write remains a pwrite64 through xWrite, and that mmap_size
defaults to 0. That measurement exists to support today's Daily Diff deep-dive (the CIDR 2022
mmap paper, which names SQLite's approach "user space copy-on-write"). Lesson 31
(memory-mapped I/O and memory allocation) keeps everything else: mmapSize / mmapSizeActual /
mmapSizeMax / nFetchOut, SQLITE_FCNTL_MMAP_SIZE, SQLITE_MAX_MMAP_SIZE, the xFetch-returns-NULL
fallback, the SIGBUS hazard in detail, and memsys / lookaside / SQLITE_CONFIG_PAGECACHE.

(d) Two findings later lessons should build on rather than repeat. First:
setDeviceCharacteristics on Linux (os_unix.c:4353-4375) is a compile-time constant plus a
single F2FS ioctl, so every SQLITE_IOCAP_ATOMIC*, SAFE_APPEND and SEQUENTIAL flag is UNSET on
ordinary Linux, and the pager discards the reported sector size anyway when
POWERSAFE_OVERWRITE is set (pager.c:2802-2805, measured: VFS says 4096, pager uses 512). That
bounds lesson 16's journal-format claims: those optimization paths are real in the source and
unreachable in ordinary Linux measurements. Second: the BATCH_ATOMIC commit path requires
zSuper==0 (pager.c:6587), so it is mutually exclusive with lesson 17's super-journal — another
row in the same table as aMJNeeded[]. Lesson 29 (SQL-level transactions, savepoints,
statement journals) may reference that exclusion.

Version caveat for anyone re-running lesson 18's measurements: they were taken against system
libsqlite3 3.45.1 while the source was read at trunk fde3a84d. The visible disagreement is
SQLITE_IOCAP_SUBPAGE_READ, which trunk sets unconditionally and 3.45.1 does not have. The
lesson states this rather than reconciling it.

scope note 2026-10-04 (lesson 19): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 19 needed only the -shm method
slots from lesson 18, the lock levels from lesson 15, and the spill/dirty-list mechanics from
lesson 14, and it took precisely the scope lesson 18's note (a) assigned it: the -shm file
contents, the wal-index hash tables, read-marks, frame append, and the heap-memory wal-index.
It did NOT re-derive the sqlite3PagerWalSupported() line ordering — it referenced lesson 18's
measurement and used the same unix-dotfile + locking_mode=EXCLUSIVE setup only to show that
no -shm file is created at all. Source read at commits 9696acb0 and ccbdec84 (identical line
numbering for every region cited), almost entirely src/wal.c; see the lesson's Sources for the
line ranges.

Four things to record for the lessons that follow:

(a) Lesson 20 (checkpointing) keeps walCheckpoint() (wal.c:2281), the WalIterator and its two
algorithms WALITER-1 / WALITER-2 (wal.c:576-636, 1796-2108 — note WALITER-2 was added in
SQLite 3.54.0 and the system 3.45.1 used for measurements does not have it), walLimitSize(),
sqlite3WalCheckpoint() (4387) and the PASSIVE/FULL/RESTART/TRUNCATE modes. Lesson 19 took
nBackfill / nBackfillAttempted / minFrame and the read-mark FLOOR only as far as measuring it
from outside: PRAGMA wal_checkpoint(PASSIVE) returned (0, 10, 3) with one reader pinned at
aReadMark[1]=3, and (0, 10, 10) once released. Lesson 20 should treat that floor as already
measured and explain the algorithm that respects it, not re-demonstrate the effect. It also
keeps the sqlite3_wal_checkpoint_v2 C interface and wal_autocheckpoint's 1000-page default
entirely — lesson 19 only ever set wal_autocheckpoint=0 to get deterministic measurements.

(b) Lesson 19 necessarily took walRestartLog() / walRestartHdr() (wal.c:2234-2248, 3961-4001)
ahead of lesson 20's "WAL reset/restart" line item, because walRestartLog() is the FIRST thing
walFrames() calls and the append path cannot be described without it. What it took: the
readLock==0 precondition, the WAL_READ_LOCK(1..WAL_NREADER-1) exclusive grab, the salt-1
increment and salt-2 re-randomization, the fact that the file is rewound and NOT truncated
(measured: size unchanged at 10512 bytes), and the reset of nBackfill and the read marks.
Lesson 20 should reference that as covered and take instead: reset as initiated by a
CHECKPOINT rather than a writer, journal_size_limit / walLimitSize interaction, and the
TRUNCATE mode. Lesson 19 measured TRUNCATE's effect (mxFrame -> 0) without opening it.

(c) A finding lesson 20 and lesson 29 should build on rather than rediscover: Wal.nCkpt is
assigned in exactly ONE place outside walRestartHdr(), namely wal.c:1488 inside
walIndexRecover(). A connection that opened a healthy WAL and never ran recovery therefore
carries a PRIVATE counter, and walFrames() writes that private value into WAL header bytes
12-15 on every restart (wal.c:4181). Measured: ckptSeq stayed 1 across two resets performed by
two different connections. So the on-disk checkpoint sequence number is NOT a global generation
count. Its only consumer is savepoint bookkeeping (sqlite3WalSavepoint, wal.c:3909;
sqlite3WalSavepointUndo, 3924-3931), for which per-connection is sufficient — lesson 29
(savepoints) should take that pair of functions, which lesson 19 only named.

(d) Lesson 19 applied lesson 18's note (d) rather than re-measuring it: POWERSAFE_OVERWRITE is
set on ext4, so sqlite3WalOpen() clears padToSectorBoundary (wal.c:1770-1772) and the padding
loop at wal.c:4283-4295 is unreachable on an ordinary Linux filesystem. Measured confirmation:
identical frame counts (5) at synchronous=NORMAL and FULL, with FULL costing exactly one extra
fdatasync of the -wal per commit. Lesson 31 (mmap and memory allocation) still owns everything
about xFetch beyond what lesson 18 took; lesson 19 did not touch mmap at all.

One sub-measurement that did NOT reproduce, recorded so nobody builds on it: reading the
-shm file's POSIX lock records out of /proc/locks was intermittent. Structurally identical
scripts sometimes showed the expected records at bytes 124-127 (plus the unix VFS's DMS byte at
128) and sometimes showed none for that inode, while the database file's SHARED lock was always
visible. The byte offsets observed agree with wal.c:1713-1723 and the read-mark VALUES were
reproducible every time, so the lesson presents the values and effects as the evidence and
flags the /proc/locks view as corroborating only. The cause was not isolated; a plausible
hypothesis, stated as such in the lesson, is lock coalescing interacting with the
single-process lock emulation lesson 18 covered. Worth settling by re-running the readers in
separate processes.

scope note 2026-10-05 (lesson 20): no lesson added, removed, reordered or split;
lessons-total stays 33. Part III is now complete. Lesson 20 took exactly the scope lesson 19's
note (a) assigned it and treated the backfill floor as already measured, explaining the
algorithm instead: walCheckpoint() (2250-2477), the mxSafeFrame loop, the salt re-check under
WAL_READ_LOCK(0), the WalIterator with both algorithms, the two fsyncs, the four modes,
walLimitSize() and the autocheckpoint hook. It referenced lesson 19's walRestartLog() rather
than re-teaching it, and took reset-as-initiated-by-a-checkpoint (TRUNCATE calling
walRestartHdr at 2465) plus the journal_size_limit path, as note (b) directed. Source read at
commit ccbdec84, same as lesson 19: wal.c (394-400, 404-466, 511-535, 576-636, 1796-1838,
1863-1901, 1920-1978, 2000-2085, 2189-2207, 2234-2248, 2250-2477, 2483-2491, 2627-2637,
1782-1784, 3368-3403, 4192, 4306-4313, 4346, 4388-4518, 4525-4532), pager.c (7550-7553),
main.c (2498-2539, 3721) and pragma.c (2398-2435).

Four things to record for the lessons that follow:

(a) Two measured corrections to common belief, both worth citing rather than re-deriving.
First: SQLITE_CHECKPOINT_RESTART does NOT reset the wal-index header. Measured on 3.45.1 with
no readers, RESTART returned (0, 32, 32) and left mxFrame=32, nBackfill=32 and the 131872-byte
-wal file exactly as they were; only TRUNCATE calls walRestartHdr() from the checkpointer
(2465), after which mxFrame/nBackfill/nBackfillAttempted/aReadMark[1] are all 0, slots 2-4 are
0xffffffff and the file is 0 bytes. Second: a non-PASSIVE checkpoint that cannot take
WAL_WRITE_LOCK is silently downgraded (eMode2 = PASSIVE, 4448) and the SQLITE_BUSY the caller
sees comes from the final line (4517), NOT from a failure to do work — measured as
(1, 29, 29), i.e. busy with the backfill 100% complete. Lesson 29 (SQL-level transactions)
may want that distinction when discussing what SQLITE_BUSY means to an application.

(b) Documentation discrepancies verified against source AND measurement, recorded because
they will come up again: pragma.html states "By default, the checkpoint is RESTART", but
pragma.c:2400 initializes eMode = SQLITE_CHECKPOINT_PASSIVE (measured: plain
PRAGMA wal_checkpoint returned (0, 29, 21) with a reader pinned, where explicit RESTART
returned (1, 29, 21)). Also, pragma.html documents -1 in the first result column for busy;
3.45.1 returns 1. Treat "nonzero" as busy.

(c) Lesson 20 measured the checkpoint's I/O shape with strace on a 50-frame WAL under
TRUNCATE: 52 pread64, 52 pwrite64, exactly 2 fdatasync (the -wal before the copy at 2359, the
database after a COMPLETE copy at 2415) and 2 ftruncate (the database down to nPage*szPage at
2413, the -wal to zero at 2466). Lesson 31 (mmap) and lesson 32 (integrity_check) can
reference the database-truncate as the only mechanism that shrinks a WAL-mode database file,
and the fact that a PARTIAL checkpoint neither syncs nor truncates the database.

(d) journal_size_limit in WAL mode is lazy and that is now measured: sqlite3WalLimit() ->
Wal.mxWalSize is consumed only at the first commit after a reset (truncateOnCommit, set at
4192 when a fresh WAL header is written, consumed at 4306-4313) and at close in persistent-WAL
mode (2629-2637, truncating to zero, not to the limit). Measured: limit 32768 with a 2476152-
byte -wal, RESTART checkpoint left the file at 2476152, and the NEXT single-row commit cut it
to exactly 32768. Lesson 29 may reference this when covering what a COMMIT can be charged for.
Also recorded: walLimitSize() (2483-2491) wraps its ftruncate in BeginBenignMalloc and ignores
errors by design, so an over-limit WAL has no diagnostic.

Unfinished thread, recorded rather than guessed: lesson 20 did not manage to produce
nBackfillAttempted > nBackfill, which requires killing a checkpointer between 2356 and 2419.
The lesson states the condition from the source comment (348-353) and lists the experiment as
a Next Step. Whoever gets to lesson 32 (integrity_check) should find out whether a database
left in that state reports anything.
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
