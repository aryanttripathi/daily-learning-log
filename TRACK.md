<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 24
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
- [x] 21 — From SQL text to parse tree: tokenize.c and the Lemon-generated parse.y
- [x] 22 — Name resolution and the Expr/Select trees (resolve.c)
- [x] 23 — The VDBE: bytecode programs, registers, and Mem cells (vdbe.c, vdbemem.c)
- [x] 24 — Type affinity, collating sequences, and value comparison
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

scope note 2026-10-06 (lesson 21): no lesson added, removed, reordered or split;
lessons-total stays 33. Part IV opens. The ordering held: lesson 21 needed nothing from
Parts I-III except the general shape of a prepared statement, and it stops exactly where
lesson 22 begins — at the point where a reduce action is about to build an Expr/Select tree.
Source read at trunk commit 466e0851 (NOT the ccbdec84 used by lessons 19-20): tokenize.c
(all 899 lines; aiClass 61, keywordhash.h include 148, sqlite3IsIdChar 190, getToken 197,
analyzeWindowKeyword/OverKeyword/FilterKeyword 246/254/261, sqlite3GetToken 273 with switch
arms 279-589, ID tail 592-594, sqlite3RunParser 600), parse.y (1-70, 255-345, 588-606 and the
%fallback/%token_class/precedence blocks), lempar.c (193-197, 292-333, 549-608, 643, 701, 951,
1090-1092), mkkeywordhash.c (all 722 lines), sqliteLimit.h (88-118), sqliteInt.h (SQLITE_N_LIMIT
1577), main.c (2985-3079).

Five things to record for the lessons that follow:

(a) Lesson 21 deliberately stopped at the grammar-symbol boundary. It did NOT read resolve.c,
did NOT explain any reduce action's body, and did NOT touch the Expr/ExprList/Select/SrcList
structures beyond naming them as what the actions build. Lesson 22 (name resolution and the
Expr/Select trees) keeps all of that. The one thing lesson 21 took that lesson 22 would
otherwise want: the double-quoted-string misfeature was measured at the TOKENIZER level only
("zzz" is TK_ID out of sqlite3GetToken; SELECT "zzz" FROM q returns the TEXT 'zzz'), and
the lesson explicitly defers the resolution step that reinterprets it. Lesson 22 should take
sqlite3VdbeUsesDoubleQuotedString, the DQS_DDL/DQS_DML db_config flags and the -DSQLITE_DQS
build options, and may cite lesson 21's measurement rather than redoing it.

(b) Lesson 21 read vdbeaux.c not at all and vdbe.c not at all, so lesson 23 (the VDBE) is
untouched. It did, however, measure three VDBE-adjacent limits from the outside and those
should be referenced rather than re-measured: SQLITE_LIMIT_COLUMN caps a result set at 2000
terms (SELECT 1,1,... fails at 2001 with "too many columns in result set"),
SQLITE_LIMIT_COMPOUND_SELECT at 500 arms (fails at 501), and SQLITE_LIMIT_EXPR_DEPTH at 1000
("Expression tree is too large (maximum depth 1000)", reached by the left-recursive
SELECT 1+1+1+... chain at 1001 terms). All three are enforced in reduce actions or later,
NOT by the automaton. Lesson 23 or 25 owns sqlite3ExprCheckHeight() itself.

(c) The measured list of all twelve settable limits on 3.45.1, for anyone who needs it without
re-probing: LENGTH 1000000000, SQL_LENGTH 1000000000, COLUMN 2000, EXPR_DEPTH 1000,
COMPOUND_SELECT 500, VDBE_OP 250000000, FUNCTION_ARG 127, ATTACHED 10,
LIKE_PATTERN_LENGTH 50000, VARIABLE_NUMBER 250000, TRIGGER_DEPTH 1000, WORKER_THREADS 0.
sqlite3_limit(db, 12, -1) returns -1 on 3.45.1, i.e. SQLITE_N_LIMIT is 12 there.

(d) A trunk-versus-3.45.1 discrepancy that is NOT the usual benign kind, recorded in full
because it bit this lesson and will bite lesson 29 (SQL-level transactions) if it reuses the
limit list. sqliteLimit.h:112-114 at trunk says "Prior to version 3.45.0 (2024-01-15), the
parser stack was hard-coded to 100 entries", implying 3.45.0+ has the growable stack with
SQLITE_MAX_PARSER_DEPTH 2500 and the "Recursion limit" message. Measured on 3.45.1 the
opposite holds: the message is "parser stack overflow" (the pre-rework text), the boundary is
93/94 nested parens (consistent with a 100-entry hard cap), 3.45.1's own parse.y has no
%stack_size directive, its lempar.c has only the YYSTACKDEPTH<=0 form with no YYGROWABLESTACK,
and there is no settable SQLITE_LIMIT_PARSER_DEPTH. So the comment's version attribution is
wrong, or the rework landed after 3.45.1. The lesson reports both and resolves neither; a Next
Step is to bisect for the commit that introduced %stack_size / %stack_size_limit /
SQLITE_MAX_PARSER_DEPTH and, if the comment is wrong, report it upstream. Do not cite
"2500" or "Recursion limit" as observable behaviour on a 3.45.x build.

(e) Three findings later lessons should build on rather than repeat. First: the %fallback set
is derivable from outside, and the arithmetic closes exactly — 73 token names in parse.y's
%fallback block (default build) expand to 78 keyword spellings (COLUMNKW->COLUMN,
LIKE_KW->LIKE/GLOB/REGEXP, CTIME_KW->the three CURRENT_*, TEMP->TEMP/TEMPORARY), plus the 7
JOIN_KW spellings and INDEXED from %token_class idj = 86 predicted usable as a bare table
name; 88 measured. The two differences are both mechanism: IF is in the fallback list but
fails as a table name because ifnotexists ::= IF NOT EXISTS gives TK_IF an action in that
state (fallback fires only on NO action), and WINDOW/OVER/FILTER succeed despite being absent
from the list because the tokenizer decides them. Second: zKWText[] is readable out of a
loaded libsqlite3 by taking the minimum of the 147 sqlite3_keyword_name() pointers as a base —
measured 666 bytes holding 860 bytes of keyword text, 341 bytes saved over NUL-terminated
storage. Third: sqlite3_error_offset() is exactly pParse->sLastToken.z - zSql, so it points at
the offending token's first byte for a tokenizer error, at the lookahead token for a syntax
error, and returns -1 for anything raised after parsing. Lesson 32 (SQLite's test strategy)
may want the fact that PRAGMA parser_trace and lemon's FALLBACK/WILDCARD/stack-growth traces
all require -DSQLITE_DEBUG and are therefore unobservable on a release build — every automaton
claim in lesson 21 is source-read, not measured.

scope note 2026-10-07 (lesson 22): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 22 took precisely the scope lesson
21's note (a) assigned it — resolve.c in full, the Expr/ExprList/Select/SrcList/NameContext
structures, and the DQS resolution step — and it cited lesson 21's tokenizer-level DQS
measurement rather than redoing it. It stops where lesson 23 begins: at the point where every
TK_ID has become TK_COLUMN with an iTable cursor number and an iColumn index, and emits no
bytecode of its own. Source read at trunk commit 9f05c6e6 (NOT lesson 21's 466e0851; trunk
moved): resolve.c (all 2367 lines — see the lesson's Sources for the per-function ranges),
sqliteInt.h (1897-1898, 2584, 3071-3133, 3141-3172, 3271-3311, 3393-3445, 3457, 3519-3586,
3629-3691), select.c (6493, 6554, 6578-6592).

Five things to record for the lessons that follow:

(a) ONE DEVIATION from lesson 21's note (a), recorded explicitly: that note assigned lesson 22
"sqlite3VdbeUsesDoubleQuotedString, the DQS_DDL/DQS_DML db_config flags and the -DSQLITE_DQS
build options". Lesson 22 took the flags and the policy function in full
(areDoubleQuotedStringsEnabled, resolve.c:161-177; SQLITE_DqsDDL/DqsDML at sqliteInt.h:
1897-1898; SQLITE_DBCONFIG_DQS_DML=1013 / DQS_DDL=1014) and measured the complete eight-row
truth table including the anomalous writable_schema && DqsDML row. It did NOT take
sqlite3VdbeUsesDoubleQuotedString / sqlite3VdbeAddDblquoteStr, because those sit behind
SQLITE_ENABLE_NORMALIZE (resolve.c:733-735) which the system 3.45.1 build does not define,
making them unmeasurable here. Lesson 23 (the VDBE) should take that pair if it covers
sqlite3_normalized_sql at all; otherwise lesson 32 (test strategy) is the right home, since
SQLITE_ENABLE_NORMALIZE is a test-build option.

(b) Lesson 22 read NO vdbe.c and NO vdbeaux.c, so lesson 23 remains untouched. It did use
EXPLAIN output as a measuring instrument in four places (the NOT NULL fold, the inlined
coalesce, the IS-TRUE equality, the nullable-column NotNull test) but treated opcodes purely
as evidence and explained no opcode semantics. Lesson 23 owns OP_Column's p5 (which is
Expr.op2 for a TK_COLUMN node — lesson 22 named the field and its meaning without following
it into codegen), register allocation, and Mem cells. Lesson 25 (code generation for
INSERT/UPDATE/DELETE) keeps pParse->oldmask / newmask, which lesson 22 only observed being
SET in lookupName:570-580.

(c) Three findings later lessons should build on rather than rediscover. First: lesson 22
REFINES lesson 21's note (e) third item. Lesson 21 stated that sqlite3_error_offset() "returns
-1 for anything raised after parsing". That is too strong. Name-resolution errors DO carry an
offset, by a different mechanism — sqlite3RecordErrorOffsetOfExpr() reading Expr.w.iOfst, not
pParse->sLastToken. Measured: "no such column: zzz" in SELECT a, zzz FROM t gives offset 10;
"ambiguous column name: a" gives 7; "no such function: nosuchfn" gives 7. The offset is -1
only where the raising call site passes no Expr, which is exactly sqlite3ResolveOrderGroupBy:
1770 passing pError=0. Lessons 23-29 should use the corrected rule: the offset is present iff
the error site has an Expr to point at. Second: three distinct code paths raise the identical
"Nth ORDER BY term out of range" text, and sqlite3_error_offset() is the ONLY externally
visible way to tell them apart (-1 from pass 2, the term's offset from pass 1 when iCol>0xffff,
the term's offset from resolveCompoundOrderBy). The 65535/65536 boundary is measured. Third:
sqlite3ExprColUsed (resolve.c:179-200) returns ALLBITS for ANY reference to a generated column,
so one generated-column reference marks every column of the table as used. Lesson 26 (the
query planner) should reference that rather than rediscovering why a covering index stops
covering; lesson 9 established what colUsed is for.

(d) A trunk-versus-3.45.1 discrepancy that is a WRONG ANSWER, not a cosmetic difference, and
the most important thing in this lesson to carry forward. Trunk's TK_ISNULL/TK_NOTNULL arm
(resolve.c:1061-1098) requires NC_Where on every NameContext before folding
"<NOT NULL column> IS NOT NULL" to TRUE, under a comment dated 2024-03-28 explaining that an
aggregated table's bare column can be NULL despite NOT NULL. 3.45.1 (released 2024-01-30) does
NOT have that guard, and the consequence is reproducible: on an EMPTY table with a INT NOT
NULL, "SELECT a, a IS NULL, a IS NOT NULL, count(*) FROM e" returns (NULL, 0, 1, 0) where
(NULL, 1, 0, 0) is correct. Measured, with EXPLAIN confirming the fold (three instructions,
Integer 1 / ResultRow, no NULL test at all). The null-extended LEFT JOIN case IS guarded in
3.45.1, by a different mechanism — EP_CanBeNull set in lookupName:507-509 — and returns the
correct (NULL, 1, 0). Anyone re-running lessons 23-29 on a 3.45.x build must not treat
IS NULL on a NOT NULL column in an aggregate result set as trustworthy. A Next Step is to
bisect 3.45.1..3.46.0 for the commit that adds the NC_Where loop.

(e) Two unfinished threads, recorded rather than guessed. First: the error string
"misuse of aliased aggregate" IS present in the shipped 3.45.1 library (confirmed with
strings) but NO construction reached it — WHERE, HAVING, GROUP BY, ORDER BY and a scalar
subquery all produced different errors from different layers ("misuse of aggregate: count()",
"aggregate functions are not allowed in the GROUP BY clause", or success). The structural
reason is that NC_AllowAgg is cleared only at resolveSelectStep:2014, i.e. only when the
result set contains no aggregate and there is no GROUP BY, in which case no alias in that
result set can be an aggregate either. The sibling branch
"misuse of aliased window function" IS reachable (NC_AllowWin is cleared at 2002, right after
the result set) and was measured. Whether the aggregate branch is dead or merely needs
UPDATE...FROM / a trigger / a converted compound is unresolved; lesson 25 or 29 may settle it.
Second: anRef[8] in the TK_NOTNULL arm bounds the nRef save AND restore at eight contexts
while the NC_Where test loop between them is unbounded, so past eight nesting levels the fold
can fire with only a partial nRef restore. Predicted consequence (a subquery wrongly marked
CORRELATED past level 8) was NOT measured; it is listed as a Next Step.

scope note 2026-10-08 (lesson 23): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 23 took the scope lesson 22's note
(b) assigned it — OP_Column's p5, register allocation, and Mem cells — and it began where
lesson 22 stopped, turning iTable/iColumn into OP_Column.p1/p2. Source read at trunk commit
74675a9e (NOT lesson 22's 9f05c6e6; trunk moved again): vdbe.h (all 449 lines), vdbeInt.h
(all, with per-struct ranges in the lesson's Sources), vdbe.c (9580 lines; allocateCursor
253-320, sqlite3VdbeExec 903-1010, the jump sites, OP_Move/Copy/SCopy/IntCopy 1662-1780,
OP_ResultRow 1807-1846, OP_RealAffinity 2195-2199, OP_Column 3014-3390, OP_TypeCheck
3427-3500, OP_MakeRecord 3578-3640), vdbeaux.c (5766 lines; sqlite3VdbeChangeP5 1303-1306,
sqlite3VdbeTypeofColumn 1308-1321, allocSpace 2560-2599, sqlite3VdbeRewind 2601-2637,
sqlite3VdbeMakeReady 2639-2756), expr.c (the OP_SCopy site 6016-6035, ExprCodeExprList 6060-
6115, the TK_ISNULL/TK_NOTNULL arms 6336-6352 and 6534-6550, the temp-register allocator
7710-7830), sqliteInt.h (OPFLAG_* 4109-4129, Parse register fields 3928/3954-3958/3990),
vdbemem.c (2280 lines, read for the Mem helpers named in the lesson). vdbeInt.h was ALSO
fetched at tag version-3.45.1 (tree 189e44df) specifically to license the ctypes probes.

Six things to record for the lessons that follow:

(a) Lesson 23 did NOT take sqlite3VdbeUsesDoubleQuotedString / sqlite3VdbeAddDblquoteStr,
which lesson 22's note (a) offered it conditionally. It does not cover sqlite3_normalized_sql
at all, and this build has no SQLITE_ENABLE_NORMALIZE. Per that note, lesson 32 (test
strategy) is now the owner of that pair.

(b) Lesson 23 deliberately stopped at the register file's edge. It did NOT explain
applyAffinity, sqlite3MemCompare, any collating-sequence lookup, or any OP_Eq-family
semantics — lesson 24 keeps all of that, and will need the MEM_* flag word this lesson
establishes. It did NOT follow a value into a record: OP_MakeRecord was read ONLY for the two
lines that set MEM_IntReal (vdbe.c:3630-3633), and OP_TypeCheck ONLY for the 6-byte
MEM_IntReal branch (3472-3491). Lesson 25 keeps OP_MakeRecord, OP_Insert, OP_IdxInsert and all
index maintenance, including the OPFLAG_NCHANGE / APPEND / USESEEKRESULT / PREFORMAT meanings
of p5 — lesson 23 took only OPFLAG_LENGTHARG / TYPEOFARG / BYTELENARG, which are OP_Column's.
Lesson 28 keeps vdbesort.c entirely; lesson 23 named CURTYPE_SORTER and nothing more.
Lesson 30 keeps CURTYPE_VTAB, xBestIndex and OPFLAG_NOCHNG; lesson 31 keeps lookaside and
memsys, which bear on the measured szMalloc=128 minimum but were not opened.

(c) Validated struct offsets for x86-64 on the measured build, so later lessons need not
re-derive them. Vdbe: nMem 36, nCursor 40, pc 48, aMem 104, apArg 112, apCsr 120, aVar 128,
aOp 136, nOp 144, nOpAlloc 148, eVdbeState 199, pFree 256. Mem (56 bytes): u 0, z 8, n 16,
flags 20, enc 22, eSubtype 23, db 24, szMalloc 32, uTemp 36, zMalloc 40, xDel 48; MEMCELLSIZE
= 24. sizeof(Op) = 24. The validations used, both of which must be re-run if the build changes:
nOp read from the struct equals the EXPLAIN row count (checked over 25 statements, no
mismatch), and aMem - aOp equals ROUND8(24*nOp) + ROUNDDOWN8(24*(nOpAlloc-nOp)) - 56*nMem
(exact on SELECT 1 -> 1088, SELECT a,b,c,d FROM t -> 920, SELECT * FROM t,u -> 752). The
Vdbe prefix through aVar, the whole sqlite3_value struct and all sixteen MEM_* constants are
byte-identical between trunk 74675a9e and the version-3.45.1 tag; the two differences found
are VdbeCursor.aType (u32 aType[1] at 3.45.1, FLEXARRAY at trunk) and sqlite3_context.argc
(u8 at 3.45.1, u16 at trunk), neither of which any measurement touched.

(d) Four findings later lessons should cite rather than rediscover. First: the register file
IS the cursor table — cursor 0 at aMem[0], cursor i at aMem[nMem-i], the VdbeCursor plus its
BtCursor living in that register's zMalloc, proven as nine exact apCsr[i] == aMem[idx].zMalloc
matches across four statements with flags == MEM_Undefined on every one. nMem - nCursor is
therefore exactly pParse->nMem, which is a free way to read the code generator's register
high-water mark from outside. Second: pParse->nMem is a monotone high-water mark and each
DISTINCT literal costs one permanent register via constant factoring, measured as 4+k for
k = 1,2,4,8,16 distinct constants against a flat 6 for the same count of identical terms; the
aTempReg[8] LIFO fully recycles sequential temps and shows NO discontinuity at eight (nested
live temps grow a dead-straight +3 per level through depth 12). Third: constant subexpressions
are evaluated once in a prologue block that lives AFTER OP_Halt, reached by OP_Init's forward
jump and left by a trailing OP_Goto; OP_Transaction lives in that same block, which is why
table-touching programs end Halt / Transaction / Goto. Lesson 29 may want that when discussing
where a transaction actually starts. Fourth: MEM_Zero makes a 100 MB zeroblob cost 3 us to
typeof() or length(), 227 ms once || forces ExpandBlob(), and the compact form does NOT
survive a round trip through a table (read back: MEM_Blob|MEM_Dyn, n = 100000).

(e) A documentation-versus-codegen gap worth carrying, and a candidate patch. opcode.html says
OPFLAG_TYPEOFARG is set when the result "will only be used by the typeof() function or the
IS NULL or IS NOT NULL operators or the equivalent". True of the flag; NOT true of the code
generator. sqlite3VdbeTypeofColumn() (vdbeaux.c:1313-1321) is a last-opcode peephole called
ONLY from expr.c:6346 (sqlite3ExprIfTrue) and expr.c:6544 (sqlite3ExprIfFalse) — i.e. only on
the jump paths. sqlite3ExprCodeTarget's TK_ISNULL/TK_NOTNULL arm never calls it. Measured on
ten 1.2 MB blobs: "SELECT id FROM big WHERE payload IS NULL" takes 0.0025 ms with Column
p5=0x80, while "SELECT payload IS NULL FROM big" takes 2.2764 ms with p5=0x00 — the same
predicate, ~900x apart, differing only in where it sits in the statement. Lesson 23 lists the
two-line fix as a Next Step. Do not repeat the claim that IS NULL always gets the flag.

(f) Two unfinished threads, recorded rather than guessed. First: MEM_IntReal was NOT verified
by measurement. It is set in exactly two places on trunk, both write paths (OP_MakeRecord
3630-3633 and OP_TypeCheck 3472-3491), and the read path's OP_RealAffinity produces MEM_Real
instead, so no UDF argument in 19 cases showed 0x0020. Lesson 24 (affinity) or lesson 25
(OP_MakeRecord) should settle it, ideally by inserting integers on both sides of the 6-byte
boundary (140737488355327 / 140737488355328) into a STRICT REAL column and decoding the
serial type on disk with lesson 2's decoder. Second: OP_SCopy appeared only 3 times in 61
statements, all on the INSERT/upsert path, and ZERO times in roughly 45 read-only statements
including every ORDER BY shape tried; OP_Copy carried 44. The emission sites are expr.c:6031
(sqlite3ExprCode when inReg != target and the expr is neither a subquery nor TK_REGISTER) and
expr.c:6092 (ExprCodeExprList without SQLITE_ECEL_DUP). Whether a SELECT can reach either is
unresolved. This matters because OP_SCopy's correctness invariant is enforced only by
Mem.pScopyFrom / mScopyFlags / bScopy and memAboutToChange(), all inside #ifdef SQLITE_DEBUG,
so the dangerous opcode's safety net is absent from every shipping build. Lesson 25 owns the
INSERT path and should say which construct emits it and why.

scope note 2026-10-09 (lesson 24): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held. Lesson 24 took exactly what lesson 23's note
handed it — applyAffinity, sqlite3MemCompare and the OP_Eq family — and it stopped at the
register file's edge on the write side, leaving OP_MakeRecord's serialisation loop, OP_Insert
and index maintenance to lesson 25. Source read at trunk commit 9cba4634 (lesson 23 read
74675a9e; trunk moved again): vdbe.c (9562 lines; alsoAnInt 329-336, applyNumericAffinity
339-371, applyAffinity 373-426, sqlite3_value_numeric_type 436-451, sqlite3ValueApplyAffinity
455-461, computeNumericType 469-491, numericType 500-512, the OP_Eq/Ne/Lt doc blocks 2242-2333,
case OP_Eq..OP_Ge 2334-2478, OP_TypeCheck 3427-3500, OP_Affinity 3526-3550), vdbeaux.c (5766
lines; sqlite3VdbeSerialType 3904-3998 behind #if 0, vdbeCompareMemStringWithEncodingChange
4423-4447, vdbeCompareMemString 4449-4462, sqlite3BlobCompare 4481-4506, sqlite3IntFloatCompare
4524-4540, sqlite3MemCompare 4552-4641), expr.c (sqlite3ExprAffinity 29-94, sqlite3ExprCollSeq
240-320, sqlite3ExprNNCollSeq 327-333, sqlite3ExprCollSeqMatch 337-343, sqlite3CompareAffinity
350-366, comparisonAffinity 372-388, sqlite3IndexAffinityOk 395-403, binaryCompareP5 409-418,
sqlite3BinaryCompareCollSeq 432-452, sqlite3ExprCompareCollSeq 460-466, codeCompare 471-495,
the TK_EQ arms 5296-5330 and 6306-6340), build.c (sqlite3AffinityType 1710-1776), sqliteInt.h
(SQLITE_AFF_* 2370-2385, struct CollSeq 2341-2347, the p5 flag bits 2396-2398), main.c
(sqlite3BinaryCompare 1061-1078, rtrimCollFunc 1084-1094, sqlite3IsBinary 1100-1105,
nocaseCollatingFunc 1116-1128, createCollation 2907-2975, the five registrations 3579-3586),
callback.c (in full), util.c (sqlite3StrICmp 423-441, sqlite3_strnicmp 442-453).

Five things lesson 24 established that later lessons should NOT re-derive:

(a) Affinity is seven consecutive ASCII characters 0x40-0x46 masked by SQLITE_AFF_MASK = 0x47,
sharing the p5 byte with SQLITE_JUMPIFNULL (0x10) and SQLITE_NULLEQ (0x80). "Is it numeric" is
one >= compare. EXPLAIN exposes p4 and p5 as ordinary result columns, so the affinity character
and the NULL-semantics flags are readable with NO debug build — p5 & 0x47 read as ASCII gives
'D'/'B'/'A'/'C' for INTEGER/TEXT/BLOB/NUMERIC, measured. Later lessons can use that instrument.

(b) Column affinity is ONE left-to-right pass with a rolling u32 4-byte window
(h = (h<<8) + sqlite3UpperToLower[x]). The documented five-rule precedence is encoded as guard
conditions: CHAR/CLOB/TEXT unguarded, BLOB guarded on aff in {NUMERIC,REAL}, REAL/FLOA/DOUB
guarded on aff == NUMERIC, and INT breaks out of the loop. Measured on 29 declared type names:
BLOBTEXT and TEXTBLOB are both TEXT, REALCHAR and CHARREAL are both TEXT, FLOATING POINT is
INTEGER (the 'oint' window). Do not re-explain this; cite section 2.

(c) The OP_Eq family UNDOES its affinity conversion at the end of the opcode (pIn1->flags =
flags1; pIn3->flags = flags3), so the conversion is correct but uncacheable. Verified
behaviourally: hex(t) is still 316533 after a numeric comparison, and both conjunct orders of
"t = n AND t = '1e3'" return the row. Measured cost of the per-row re-parse on 300k rows:
1.61x for '299999', 1.82x for '2.99999e5', 2.10x for a 20-char zero-padded literal, versus
0.97x for CAST('299999' AS INTEGER) — which is constant-folded into lesson 23's run-once
prologue. Lesson 26 may want this when discussing why a sargable term should be a bound
parameter or a CAST rather than a string literal.

(d) Lesson 23's note (f) first unfinished thread is now SETTLED, and the answer is on disk.
A REAL column keeps an integer serial type up to six bytes and reverts to serial type 7 past
that, because sqlite3VdbeSerialType's MEM_IntReal branch converts to MEM_Real exactly when the
integer would need 8 bytes. Measured, one row per value, serial type decoded from the page
bytes: 1 -> serial 9 (ZERO payload bytes), 2 and 127 -> serial 1, 128 and 32767 -> serial 2,
8388607 -> 3, 2147483647 -> 4, 2^40 -> 5 (6 bytes), 2^50 -> 7 (8-byte double), 1.5 -> 7. All
ten report typeof() = 'real'. MAX_6BYTE is 140737488355327, so lesson 23's guessed boundary
pair was exactly right. Lesson 25 does not need to re-measure this; it should instead explain
the serialisation loop that consumes the flags.

(e) Two scope notes for later lessons. Lesson 26 (where.c) inherits one UNRESOLVED row: with
an index on a TEXT column, "tcol = CAST(icol AS TEXT)" is NOT blocked by sqlite3IndexAffinityOk
(both sides TEXT -> the two-column branch returns SQLITE_AFF_BLOB -> aff < SQLITE_AFF_TEXT ->
return 1), yet the plan is a SCAN. Lesson 24 recorded this as unverified rather than guessing;
where.c must be read to answer it. Measured plans that ARE explained: tcol = 'x' and tcol = 5
both SEARCH (TEXT affinity, TEXT index); tcol = icol and tcol = ncol both SCAN (NUMERIC
affinity vs TEXT index); tcol = 5 COLLATE NOCASE degrades to a covering-index SCAN on a
COLLATION mismatch, not an affinity one — so the plan's verb alone does not identify which
gate closed. Lesson 28 (the sorter) inherits the collation call counts: a user collation on a
500-row table was invoked 500 times for an equality scan, 499 for ORDER BY ... LIMIT 1, and
~3730 for count(DISTINCT), so the sorter's merge is where collation cost actually lives.
Lesson 29 (savepoints) inherits a cross-engine framing already written into that day's digest:
SQLite needs no per-row lock-holder encoding because it has a single writer, which is why it
has nothing like PostgreSQL's MultiXact subsystem.

Also recorded: NOCASE's comparison stops at an embedded NUL byte (the *a!=0 guard in
sqlite3_strnicmp, util.c:451), and this reaches constraint enforcement — a UNIQUE ... COLLATE
NOCASE index rejects 'a'||char(0)||'c' as a duplicate of 'a'||char(0)||'b', both three stored
bytes. And NOCASE/RTRIM are registered for UTF-8 only while BINARY is registered three times,
so on a UTF-16 database every NOCASE comparison takes
vdbeCompareMemStringWithEncodingChange() — two ephemeral Mem cells and two transcodes per
comparison. That penalty is UNMEASURED and is in lesson 24's Next Steps.
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
