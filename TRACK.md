<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 31
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
- [x] 25 — Code generation for INSERT/UPDATE/DELETE: OP_MakeRecord, OP_Insert, and index maintenance
- [x] 26 — The query planner: WhereLoop objects, cost estimation, and the path solver (where.c)
- [x] 27 — ANALYZE statistics: sqlite_stat1 and sqlite_stat4 and how the planner uses them
- [x] 28 — Sorting and temporary storage: the VDBE sorter (vdbesort.c) and external merge sort

### Part V — Operational and advanced topics

- [x] 29 — Transactions at the SQL level: autocommit, savepoints, and statement journals
- [x] 30 — Virtual tables: the xBestIndex protocol, plus sqlite_dbpage and dbstat introspection
- [x] 31 — Memory-mapped I/O and memory allocation (mmap_size, memsys, lookaside)
- [ ] 32 — Verifying a database: how PRAGMA integrity_check walks the file, and SQLite's test strategy
- [ ] 33 — VACUUM and VACUUM INTO: the temp-database rewrite in vacuum.c, the backup API it shares, and why it is the only way to change auto_vacuum mode

<!--
NOTE BLOCK POLICY (set 2026-10-12, lesson 27). This block holds the scope and handover notes for
the three most recent lessons only. Older notes are not deleted: they are preserved verbatim in
this file's own git history, and TRACK-ARCHIVE.md records exactly which commit and blob to read
for which lessons. When this block grows past three lessons, move the oldest out and update
TRACK-ARCHIVE.md. The lesson documents themselves are the real record; these notes exist only so
that a fresh session knows what a previous run deliberately left alone.

scope note 2026-10-14 (lesson 29): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 29 needed lesson 16's rollback-journal
record format as a contrast, lesson 15's pager states as the frame, lesson 03's page-type byte to
read a captured page, and lesson 28's VFS-shim technique; nothing else. Also trimmed per the policy
above: lesson 26's note moved out and indexed in TRACK-ARCHIVE.md (commit
ad3839f8be496cef3cf2db47fd4866cc7733b04d, TRACK.md blob 68c8bcb936ccc25efac2076b649494b22a755cc2).
Notes for 27, 28, 29 retained. Build: SQLite 3.53.4 from better-sqlite3@13.0.3's
deps/sqlite3/sqlite3.c, gcc 13.3.0 -O2, 2-core container, page_size 4096 unless stated.

BOTH of lessons 25/26's named handoffs are CLOSED, and the answer inverts lesson 26's premise.
WHERE_ONEPASS_DESIRED / WHERE_ONEPASS_MULTIROW / WHERE_DUPLICATES_OK and sqlite3MultiWrite() are
IRRELEVANT to statement journalling. The gate is
  p->usesStmtJournal = (u8)(pParse->isMultiWrite && pParse->mayAbort);   sqlite3.c:90569
and for a DELETE, mayAbort is never set (nothing in a DELETE can raise a constraint), so
where.c's "if( eOnePass!=ONEPASS_SINGLE ) sqlite3MultiWrite(pParse)" sets a flag that the && then
discards. MEASURED: "DELETE FROM t WHERE a>5" on a table with a UNIQUE index -> usesStmtJournal=0.
A later lesson must not repeat lesson 26's expectation that ONEPASS controls this.

THE OBSERVABLE, for any future lesson that needs it. sqlite3FinishCoding (sqlite3.c:126684) attaches
VdbeComment((v, "usesStmtJournal=%d", pParse->mayAbort && pParse->isMultiWrite)) to the very
OP_Transaction that consults the flag (sqlite3.c:126761). So on a -DSQLITE_DEBUG build (which turns
on SQLITE_ENABLE_EXPLAIN_COMMENTS) the flag is readable off the "comment" column of EXPLAIN's
Transaction row -- no patched build needed. The amalgamation in that npm package ships NO shell.c,
so this needs a 20-line harness that looks the column up by name.

Measured usesStmtJournal table (all on 3.53.4, schema-only decision -- data is never consulted):
  INSERT..SELECT into a table with UNIQUE                      1
  same with OR IGNORE / OR REPLACE / OR FAIL / OR ROLLBACK     0  (all four)
  INSERT..SELECT into a bare "a INTEGER PRIMARY KEY" table     1  (rowid can collide)
  INSERT..SELECT into a table with no PK and no constraints    0
  INSERT..SELECT where target has NOT NULL, or CHECK           1
  UPDATE of an indexed/UNIQUE column                           1
  UPDATE of a non-indexed column, no constraint                0
  DELETE, one row or many, with or without indexes             0
  CREATE INDEX and CREATE UNIQUE INDEX                         1
  single-row INSERT..VALUES                                    0
  ... with an AFTER INSERT trigger whose body is RAISE(ABORT)  1
  ... with an AFTER INSERT trigger that only writes            0
  INSERT..SELECT into an FK child: foreign_keys=OFF 0, ON      1
  single-row DELETE by rowid on the FK PARENT, foreign_keys=ON 1
PRAGMA foreign_keys therefore changes the I/O profile of statements that never mention an FK.

RUN-TIME gate, on top of the prepare-time flag (OP_Transaction, sqlite3.c:100747):
  p->usesStmtJournal && pOp->p2 && (db->autoCommit==0 || db->nVdbeRead>1)
and then p->iStatement = db->nSavepoint + db->nStatement (100750) -- a statement transaction IS a
savepoint, numbered above every named one. MEASURED both ways with an identical abortable UPDATE
(NOT NULL violation engineered onto the LAST row of 100,000 so the statement does real work first):
inside BEGIN after an earlier UPDATE -> 307 sub-journal records, statement undone, earlier UPDATE
survived, autocommit still 0; in autocommit -> 0 records, everything undone, autocommit 1.
NOT ESTABLISHED: the db->nVdbeRead>1 arm. In autocommit with one writer the external outcome is
identical either way because sqlite3VdbeHalt commits right after, so this harness could not
distinguish it. Needs a build printing p->iStatement. Lesson 29's Next Steps item 1. Do not state
it as measured.

WHY FOUR HARNESSES FOUND NOTHING, and the trap for any future lesson. openSubJournal
(sqlite3.c:64202) calls sqlite3JournalOpen with nSpill = sqlite3Config.nStmtSpill, so the
sub-journal is a MEMJOURNAL first and the real VFS xOpen happens only on spill. Below the
threshold a statement journal has ZERO filesystem footprint -- no xOpen, no name, nothing for a
VFS shim or strace. Count records with PAGERTRACE instead: the "#if 0" above
"int sqlite3PagerTrace=1;" (sqlite3.c:59751) flipped to "#if 1" makes subjournalPage print
"STMT-JOURNAL %d page %d". sqlite3PagerTrace has PLAIN external linkage (not SQLITE_PRIVATE), so a
separate TU can set it -- unlike sqlite3WhereTrace in lesson 27.

THE SPILL BOUNDARY, predicted in closed form then measured to the single input row.
SQLITE_STMTJRNL_SPILL defaults to 64*1024 (sqlite3.c:24052). memjrnlWrite spills on
"(iAmt+iOfst) > p->nSpill". A sub-journal record is 4+page_size, so at 4096: 4100*16 = 65,600 >
65,536 and 4100*15 = 61,500, i.e. the 16th record's PAGE write (not its 4-byte pgno write, which
ends at 61,504) creates the file. Re-dirtying a prefix of a 581-page / 200,000-row table inside an
open SAVEPOINT: K=6146 -> 15 records, xOpen 0. K=6147 -> 16 records, xOpen 1, exactly 2 xWrites
(n=61504 off=0 from memjrnlCreateFile, then n=4096 off=61504), 65,600 bytes = 16*4100. The same
arithmetic predicted the 307-record case to the write: 2 + (307-15-1)*2 = 584 writes and
307*4100 = 1,258,700 bytes. MEASURED 584 and 1,258,700.
PRAGMA journal_mode=memory forces nStmtSpill=-1: 657 records / 2,693,700 bytes became zero bytes.

SUB-JOURNAL RECORD FORMAT, decoded off the shim at page_size 4096: 4-byte BIG-ENDIAN pgno then the
raw page, 4100 per record, records at 0 / 4100 / 8200 with pgno 3 / 4 / 5 and first page byte 0x0d
(lesson 03's table-leaf code). NO header, NO checksum, NO padding -- against the main journal's
512-byte header and 4104 per record (pgno 4 / page 4096 / checksum 4), both visible in the same
run. It can drop them because it is SQLITE_OPEN_SUBJOURNAL|DELETEONCLOSE with zName=NULL
(measured flags 0x201e) and is never read after a crash.

WHY THE SUB-JOURNAL IS USUALLY EMPTY (the thing that cost this run four harnesses). A savepoint
stores a POSITION, not a snapshot: pagerOpenSavepoint (66601) records nOrig = dbSize,
iOffset = journalOff, iSubRec = nSubRec and an EMPTY bitvec. pagerPlaybackSavepoint (63088) replays
the MAIN journal from iOffset to journalOff into a pDone bitvec FIRST, then the sub-journal records
iSubRec..nSubRec skipping anything in pDone. So a page first dirtied AFTER the savepoint opened is
covered by the main journal and needs no sub-journal record at all. subjRequiresPage (60702) fires
only for nOrig>=pgno with the bit clear -- i.e. a page dirtied EARLIER in the transaction and
written again inside the savepoint. ONE record per page per savepoint. Its inner loop also clears
bTruncateOnRelease on every LATER savepoint. To produce records at all the recipe is:
BEGIN; <write pages>; SAVEPOINT (or an abortable multi-write statement); <write the same pages>.

Record counts are explained by page layout, not row counts: 100,000-row p(a INTEGER PRIMARY KEY,
b INTEGER UNIQUE, c INTEGER NOT NULL) has page_count 628 = ~307 table leaves + ~321 index pages,
and an UPDATE of only the non-indexed c produced exactly 307 records.

bTruncateOnRelease ONLY TRUNCATES IN MEMORY (sqlite3PagerSavepoint, 66689): nSubRec is rewound
either way, but the sqlite3OsTruncate is gated on sqlite3JournalIsInMemory(sjfd). Once spilled, a
sub-journal file never shrinks on RELEASE.

sqlite3VdbeHalt (91190) decides RELEASE vs ROLLBACK: rc==OK or errorAction==OE_Fail -> RELEASE;
OE_Abort -> ROLLBACK; anything else -> full sqlite3RollbackAll + CloseSavepoints + autoCommit=1.
Of the four "special" errors only NOMEM and FULL are eligible for statement rollback, and only
when usesStmtJournal; IOERR and INTERRUPT ALWAYS take the whole transaction down. vdbeCloseStatement
(91094) runs ROLLBACK then RELEASE on the rollback path (the anonymous savepoint must vanish either
way), and the commit branch sets db->nStatement = 0 wholesale rather than decrementing.

SQL-LEVEL SEMANTICS, all measured:
- SAVEPOINT outside BEGIN sets autoCommit=0 and isTransactionSavepoint=1 and is NOT counted in
  nSavepoint; RELEASE of it sets autoCommit=1 and calls sqlite3VdbeHalt -- it IS the commit.
  Observable oddity: right after "SAVEPOINT outer", sqlite3_get_autocommit()==0 while
  sqlite3_txn_state()==SQLITE_TXN_NONE. Two APIs, two answers, same instant (deferred).
- BEGIN with a non-empty stack -> SQLITE_ERROR "cannot start a transaction within a transaction".
- ROLLBACK TO is idempotent and repeatable (3x ok), does NOT pop the named savepoint; only RELEASE
  does, after which ROLLBACK TO gives "no such savepoint: k".
- Duplicate names are a stack, not a dict: with two savepoints named s, RELEASE s pops the inner
  one and a following ROLLBACK TO s finds the OUTER one and undoes pre-inner work (row count went
  to 0, not 1).
- A plain ROLLBACK empties the whole stack.
- OP_Savepoint refuses SAVEPOINT and RELEASE while db->nVdbeWrite>0 ("SQL statements in progress",
  SQLITE_BUSY); ROLLBACK TO is NOT on that list.
- ROLLBACK TO restores nDeferredCons/nDeferredImmCons from the savepoint; RELEASE does not.
- ROLLBACK TO across a schema change calls sqlite3ExpirePreparedStatements +
  sqlite3ResetAllSchemasOfConnection and then re-sets DBFLAG_SchemaChange.

COST OF AN OPEN SAVEPOINT, measured (300,000 rows, second full UPDATE pass over the same pages,
best of 5): no savepoint 48.3 ms / 0 extra bytes; one savepoint open 53.3 ms / 657 records /
2,693,700 bytes. Do NOT quote the ~10% wall-clock as the cost -- the temp file stayed in the host
page cache. The reproducible number is the 2.69 MB of writes that did not exist before, on
unchanged SQL. Also note subjRequiresPage is a LINEAR SCAN of aSavepoint[] on every write to an
already-writeable page, so depth costs CPU as well as records.

Handed forward: lesson 30 gets the whole xBestIndex protocol (whereLoopAddVirtual 4685,
whereLoopAddVirtualOne 4361, allocateIndexInfo 1408, vtabBestIndex 1669, the
sqlite3_vtab_collation/in/rhs_value/distinct accessors 4557-4646), sqlite_stat4-as-a-table, and now
ALSO sqlite3VtabSavepoint: EVERY branch of OP_Savepoint and of vdbeCloseStatement calls it with the
same index, so xSavepoint/xRelease/xRollbackTo is the savepoint stack re-exported to extensions --
including the "db->autoCommit==0 || db->nVTrans==0" assert at sqlite3.c:100466. Lesson 31 gets
sqlite3StackAllocRawNN, the loadStatTbl allocation cost, the unresolved sorter-mmap question from
lesson 28, vdbesort.c's bulk-allocation mode, and now ALSO memjournal's FileChunk chain as a third
allocator consumer (nChunkSize == nSpill == 64 KB for a sub-journal, so chunks are large) plus
sqlite3_config(SQLITE_CONFIG_STMTJRNL_SPILL) as a tunable it should price. Lesson 32 gets
OptimizationEnabled(), the SQLITE_TESTCTRL_OPTIMIZATION bit table, SQLITE_Stat4,
SQLITE_TESTCTRL_SORTER_MMAP, and now the observation that PRAGMA integrity_check is read-only and
so never reaches any of this -- but that sqlite3BtreeTripAllCursors(SQLITE_ABORT_ROLLBACK,
isSchemaChange==0) on the ROLLBACK TO path is the mechanism behind SQLITE_ABORT_ROLLBACK, which a
test-strategy lesson should cover. Lesson 33 (VACUUM) should note that VACUUM does not recompute
statistics, that it reruns the CREATE INDEX sorter path once per index, and that CREATE INDEX
itself has usesStmtJournal=1 (measured above), so a VACUUM inside a transaction is a statement-
journal workload.

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (effect measured in lesson
28, function still not read -- carried over from lessons 27 and 28), sqlite3Stat4ProbeSetValue and
the stat4 probe-record machinery in vdbemem.c, whereRangeSkipScanEst, the vdbeSorterCompareReal
arm of the sorter comparators, and -- new this run -- the WAL savepoint path
(PagerSavepoint.aWalData, sqlite3WalSavepoint, sqlite3WalSavepointUndo) and sqlite3PagerMovepage's
subjournal call site, neither of which any measurement here exercised.

scope note 2026-10-15 (lesson 30): no lesson added, removed, reordered or split;
lessons-total stays 33. Per the NOTE BLOCK POLICY above, lesson 27's note was moved out this run
and TRACK-ARCHIVE.md gained its row (commit 8d6880a7dace31a374afc865d1556b9e3593caa4, TRACK.md
blob e5e40ca598fb43fd2f26d090bc2bb10f576f4d5b). This block now holds lessons 28, 29 and 30.

Build used: the same SQLite 3.53.4 amalgamation from better-sqlite3@13.0.3
(deps/sqlite3/sqlite3.c). NO SQLITE_DEBUG build was needed this run -- everything measured came
out of ordinary builds plus EXPLAIN QUERY PLAN and stat(1). Three harnesses, all small and all
rebuildable from the lesson's Hands-On section: an instrumented vtab printing the whole
xBestIndex conversation; a cost-sweep variant; a malfunction-mode variant. Also
-DSQLITE_ENABLE_DBSTAT_VTAB -DSQLITE_ENABLE_DBPAGE_VTAB for a small exec-and-print tool, since
the amalgamation ships both modules compiled out by default and no shell.c is vendored.

TRAP THAT COST TIME, FOR FUTURE RUNS. sqlite3_create_module returns SQLITE_OK even when the
module is unusable: createModule (sqlite3.c:162533) does (void)sqlite3VtabCreateModule(...) and
returns the rc it was passed. Worse, sqlite3VtabCallCreate (sqlite3.c:162681) reports
"no such module: X" when pMod==0 OR xCreate==0 OR xDestroy==0. A module with a good xCreate and
a NULL xDestroy is registered, works eponymously, and still fails CREATE VIRTUAL TABLE with
"no such module". Fill in xDestroy.

MEASUREMENTS THIS LESSON OWNS, all original:
- Entry values of every xBestIndex output field, read from inside the callback:
  estimatedCost = 5e+98 (SQLITE_BIG_DBL/2, SQLITE_BIG_DBL = 1e99 at sqlite3.c:15708),
  estimatedRows = 25, idxNum = 0, idxFlags = 0, orderByConsumed = 0, aConstraintUsage zeroed.
  Reset per call, not once (whereLoopAddVirtualOne, sqlite3.c:173212).
- THE COST CLIFF. EXPLAIN QUERY PLAN's third column IS the loop's rRun. Swept one module's
  estimatedCost: 1->0, 2->10, 10->33, 100->66, 1e6->199 (all = 10*log2), then 1e12->400 where
  10*log2 = 399, and 5e98->3280 where 10*log2 = 3279. Above 2e9 sqlite3LogEstFromDouble
  (sqlite3.c:38519) stops computing a log and reads the IEEE exponent field:
  e = (a>>52)-1022; return e*10. So cost is quantised to one value per power of two above 2e9,
  and both big values are a full unit off. rSetup is hard-wired to 0 for every vtab loop.
- CALL SCHEDULE: 1 (all usable, +1 if bRetryLimit) + 1 (no IN) + K (distinct prereq masks)
  + 1 (all disabled) + 1 (all disabled, no IN) = K+4 upper bound. Measured: WHERE a=5 -> 1 call
  (the mBest==0 && !bIn short circuit); FROM vt,t1 WHERE a=t1.x -> 2; WHERE a IN (1,2,3) -> 2
  calls and THREE xFilter calls.
- WO_IN is folded to SQLITE_INDEX_CONSTRAINT_EQ in allocateIndexInfo; there is no
  SQLITE_INDEX_CONSTRAINT_IN. LIMIT(73)/OFFSET(74) arrive with iColumn=0 and OFFSET before LIMIT.
- colUsed verified as a used-columns mask: 0x1 / 0x7 / 0x5 / 0x3 for four different select lists.
- MALFUNCTION: gap in argvIndex -> "vt.xBestIndex malfunction"; duplicate argvIndex -> same.
  NOT TRIGGERED: the pIdxCons->usable==0 arm (the winning plan came from an all-usable call).
  That is lesson 30's Next Steps item 2, left open deliberately.
- DIRECTONLY is enforced at USE, not definition: CREATE VIEW v AS SELECT * FROM dbstat succeeds,
  SELECT * FROM v then fails with 'unsafe use of virtual table "dbstat"'.
- statBestIndex's idxNum bitmask confirmed through EQP text: WHERE name=? -> 0x2,
  + ORDER BY name,path -> 0xa, bare -> 0x0, all in HEX because of SQLITE_INDEX_SCAN_HEX;
  sqlite_dbpage prints decimal because it does not set that bit.
- dbstat/dbpage CROSS-CHECK on a 120-page, 4096-byte-page file with users(3000 rows) +
  users_name: root page 2 interior, ncell 103 = header bytes 3-4 0x0067, rightmost child
  0x00000078 = 120 = count(*) FROM sqlite_dbpage. Leaf page 4: 0x0D, ncell 0x1E=30, content
  start 0x0055=85; 8 + 2*30 + (4096-85) = 4079, so unused = 17, which is exactly dbstat's
  unused for that page. Aggregate mode: users 105 pages 97.06% packed, users_name 14 pages
  81.84%.
- THE REAL FIND, and it closes lesson 29's sqlite3VtabSavepoint handoff. sqlite_dbpage's whole
  truncation mechanism is DbpageTable.pgnoTrunc (one Pgno) + iDbTrunc, which explains three of
  the four documented limitations mechanically (dbpageUpdate sqlite3.c:233046, dbpageSync,
  dbpageRollbackTo). The FOURTH documented rule -- "any ROLLBACK TO will cancel the null-INSERT"
  -- IS WRONG as stated. Measured on fresh 120-page copies, by file size:
    BEGIN; INSERT(100,NULL); COMMIT;                                      -> 99 pages
    BEGIN; INSERT(100,NULL); UPDATE ... pgno=3; COMMIT;                   -> 120 (no-op, as doc'd)
    BEGIN; SAVEPOINT s1; INSERT(100,NULL); ROLLBACK TO s1; COMMIT;        -> 99  (NOT cancelled)
    BEGIN; UPDATE ... pgno=3; SAVEPOINT s1; INSERT(100,NULL); ROLLBACK TO s1; COMMIT; -> 120
    BEGIN; UPDATE ... pgno=3; SAVEPOINT s1; INSERT(100,NULL); RELEASE s1; COMMIT;     -> 99
    BEGIN; INSERT(100,NULL); ROLLBACK;                                    -> 120
  Cause: sqlite3VtabSavepoint (sqlite3.c:162995) only calls xRollbackTo
  "if( xMethod && pVTab->iSavepoint>iSavepoint )", and pVTab->iSavepoint is set ONLY at
  SAVEPOINT_BEGIN, and only for vtabs already in db->aVTrans[]. A vtab that joins the
  transaction after the savepoint was taken has iSavepoint==0 and never hears about it.
  NOT DONE: bisecting which version this changed in, and not reported upstream. Lesson 30's
  Next Steps item 4. Do not re-derive this; extend it.
  Also noted: sqlite3VtabSavepoint temporarily CLEARS SQLITE_Defensive around the callback.
- Other exact error strings, each measured on a clean copy: DELETE FROM sqlite_dbpage ->
  "cannot delete" (argc==1 arm); INSERT(1,NULL) -> "bad page value" (no page-1-specific
  message); with SQLITE_DBCONFIG_DEFENSIVE set, UPDATE -> rc=11 and the "read-only" arm;
  null-INSERT at pgno=121 on a 120-page header -> no-op (the
  pgnoTrunc < sqlite3BtreeLastPage guard in dbpageSync, visible from outside).
- Zeroing a live page then PRAGMA integrity_check -> "Tree 2 page 99: btreeInitPage() returns
  error code 11" + "wrong # of entries in index users_name". Hand that to lesson 32.

DELIBERATELY NOT TAKEN: sqlite_stat4-as-a-table, which lessons 27 and 29 both handed to lesson 30.
The vtab/xBestIndex material filled the day on its own and the stat4-table angle belongs with
whichever later lesson revisits the planner's inputs. It is NOT closed -- re-hand it forward.

Handed forward: lesson 31 still gets sqlite3StackAllocRawNN, the loadStatTbl allocation cost,
the unresolved sorter-mmap question from lesson 28, vdbesort.c's bulk-allocation mode,
memjournal's FileChunk chain, and now ALSO allocateIndexInfo's one-malloc-for-four-arrays plus
SZ_HIDDENINDEXINFO as a flexible-array-member sibling of the lookaside question. Lesson 32 gets
xIntegrity (the iVersion 4 method, never exercised here), the corrupted databases this run
produced, and btreeInitPage's error path. Lesson 33 (VACUUM) gets the backup API and should note
that sqlite_dbpage's xSync calls sqlite3PagerTruncateImage, which is NOT how VACUUM shrinks.
sqlite_stat4-as-a-table goes to whichever of 31-33 revisits the planner's inputs.

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (carried over from
lessons 27, 28 and 29 -- still not read), sqlite3Stat4ProbeSetValue, whereRangeSkipScanEst, the
WAL savepoint path (PagerSavepoint.aWalData, sqlite3WalSavepoint, sqlite3WalSavepointUndo) and
sqlite3PagerMovepage's subjournal call site (both carried from lesson 29), and -- new this run --
sqlite3_vtab_in/distinct/rhs_value's implementations beyond the struct fields they read,
xShadowName and SQLITE_VTAB_INNOCUOUS/CONSTRAINT_SUPPORT, xFindFunction and
sqlite3VtabOverloadFunction, and dbstat's cursor walk (statNext/statDecodePage), whose output I
verified against raw bytes without reading its code.


scope note 2026-10-16 (lesson 31): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Per the NOTE BLOCK POLICY above, lesson 28's
note was moved out this run and TRACK-ARCHIVE.md gained its row (commit
815ea964987e5764d01f917e944ca8a7eb5e2851, TRACK.md blob
b11a76fdd03a5ee9ede69de38ec2585b029d0efd). This block now holds lessons 29, 30 and 31.

Build used: the same SQLite 3.53.4 amalgamation from better-sqlite3@13.0.3
(deps/sqlite3/sqlite3.c), gcc 13.3.0 -O2, -DSQLITE_THREADSAFE=0, 2-core container, system page
size 4096. Test db: t(a INTEGER PRIMARY KEY, b TEXT, c INTEGER) 200,000 rows + index tc ON t(c),
page_size 4096, 3,836 pages / 15,712,256 bytes. Five variant builds this run: default,
-DSQLITE_ENABLE_SORTER_MMAP, -DSQLITE_ENABLE_MEMSYS5, -DSQLITE_ENABLE_MEMORY_MANAGEMENT, and one
12-line patched build adding "int sqlite3FetchTrace" + an fprintf inside unixFetch. That symbol
has PLAIN external linkage by construction (I added it), unlike sqlite3WhereTrace in lesson 27 --
so a separate TU can flip it; it is NOT in stock SQLite, do not look for it.

LESSON 28'S NAMED HANDOFF IS CLOSED, and lesson 28's three candidate causes were ALL WRONG.
Sorter mmap fails for two stacked reasons:
(a) db->nMaxSorterMmap is set to 0x7FFFFFFF ONLY inside #ifdef SQLITE_ENABLE_SORTER_MMAP
    (sqlite3.c:190846-190853), which this amalgamation does not define, so the field is 0 and
    BOTH gates -- vdbeSorterMapFile's "pFile->iEof <= nMaxSorterMmap" (107150) and
    vdbeSorterExtendFile's "nByte <= nMaxSorterMmap" (107838) -- are false for any real PMA.
    MEASURED: default build, 200k-row ORDER BY spilling 11,088,919 bytes over 8 PMAs,
    cache_size=-2000, threads=0 -> ZERO xFetch on temp files. With the compile flag, or with
    sqlite3_test_control(SQLITE_TESTCTRL_SORTER_MMAP, db, 0x7FFFFFFF) on a default build -> 16
    xFetch, STILL ZERO mapped. Timing is identical: 46.5 ms default vs 47.0 ms SORTER_MMAP,
    best-of-5 (46.5 is byte-identical to lesson 28's number on the same db).
(b) EVEN WHEN ENABLED IT CANNOT SUCCEED. Traced from the patched build:
      FETCH (temp) iOff=0 nAmt=11088919 need=11089175 mmapSize=11088925
                   mmapSizeMax=2147418112 st_size=11091968 -> null
    mmapSizeMax is the FULL SQLITE_MAX_MMAP_SIZE, so the unixFileControl clamp lesson 28
    suspected DID NOT FIRE and the VFS declined nothing -- the mapping exists and is exactly
    mmapSize = the SIZE_HINT value, because SQLITE_FCNTL_SIZE_HINT (44292-44303) calls
    unixMapfile(pFile, nByte). vdbeSorterMapFile asks for the ENTIRE file at offset 0 and
    unixFetch demands mmapSize >= iOff+nAmt+256. The only slack the sorter ever creates is
    vdbeSorterExtendFile(db, fd, iEof + szPMA + 9) at 108132 -- NINE bytes. Final gap: 250 bytes.
    Not fixable through any API-exposed knob. NOT DONE: trying a VFS whose xFetch has no EOF
    buffer, and bisecting the release where this stopped working. Lesson 31 Next Steps item 1.

THE 256-BYTE EOF BUFFER IS THE CENTRAL FACT OF THIS LESSON and later lessons will need it.
unixFetch (45912) returns NULL unless mmapSize >= iOff+nAmt+256; the mapping can never exceed the
file length; therefore THE LAST PAGE OF EVERY DATABASE IS UNMAPPABLE AT EVERY mmap_size. Measured
to the byte on the 15,712,256-byte db: mmap_size 15,708,160 -> 579 of 581 pages mapped;
15,708,416 -> 580; 15,712,256 -> 580. One page per 256 bytes, predicted first try from
floor((min(mmap_size,fileSize)-256)/page_size). The comment says the buffer exists because a
CORRUPT page can make SQLite overread "a few bytes" -- hand that to lesson 32.

mmap FACTS MEASURED (index-only scan touching 581 pages, cache_size=-2000):
- mmap_size is a PREFIX WINDOW, not a hot-page cache. unixMapfile always maps from offset 0,
  min(st_size, mmapSizeMax). 262144 / 1048576 / 4194304 ALL gave 581 xFetch and ZERO mapped and
  582 xRead, because index tc lives past the 4 MB mark. No partial credit, no warning.
- mmap_size=20971520: 2 xRead (page 1 + last page), 581 xFetch, 580 mapped,
  sqlite3_memory_used 146,864 vs 2,166,280 (14.8x), CACHE_USED 14,136 vs 2,103,096,
  CACHE_MISS 2 vs 582, heap growth during the scan 656 B vs 2,020,072 B.
- WALL CLOCK IS THE SMALL PART: best-of-7 6.4 -> 5.6 ms (1.14x) at cache_size=-2000 and
  6.0 -> 5.7 at cache_size=-10, on a host-page-cache-resident file. Do NOT quote 1.14x as the
  value of mmap; the 14.8x memory figure is the reproducible one.
- A READ INSIDE A WRITE TRANSACTION IS STILL MAPPED (PAGER_GET_READONLY satisfies the
  eState==PAGER_READER || flags test). BEGIN IMMEDIATE; SELECT -> 580 mapped. The UPDATE itself
  -> ZERO xFetch. Rule is "a page you will WRITE is never mapped", not "no mmap during writes".
- WAL EATS THE MAPPING. Same db in wal: scan before an UPDATE of 200 rows -> 581 xFetch/580
  mapped; scan after -> 399 xFetch/398 mapped. 182 pages had moved into the WAL and
  sqlite3WalFindFrame()!=0 routes them to getPageNormal. Decays per commit, restored by
  checkpoint. NOT MEASURED: the recovery after wal_checkpoint(TRUNCATE) -- predicted to return
  to 581, left as Hands-On exercise 3.
- Page 1 is never mapped (pgno>1 is unconditional; the assert at 65155 depends on it).
- A mapped page is sizeof(PgHdr)+nExtra ONLY, flags=PGHDR_MMAP, recycled via
  pPager->pMmapFreelist, and NOT in the pcache hash -- sqlite3PagerLookup cannot find it.
- One mmap() failure sets pFd->mmapSizeMax=0 permanently for that file (45853).

ALLOCATOR FACTS MEASURED:
- pcache1 szAlloc = szPage + szExtra + ROUND8(sizeof(PgHdr1)) = 4368 AT page_size 4096, i.e.
  272 bytes / 6.6% overhead per page. Clean staircase of +4368 and one malloc per page; a
  5000-row scan step was +266,448 = 61 x 4368 exactly. cache_size=-2000 therefore holds 468
  pages, not 500.
- pcache1InitBulk: first fetch into an empty cache allocates szAlloc * SQLITE_DEFAULT_PCACHE_INITSZ
  = 20 * 4368 = 87,360 in ONE malloc (measured as the +88,424 at schema-read time and as the
  "largest allocation" in every sqlite3_status run). Bulk pages have isBulkLocal=1 and go back on
  pCache->pFree, never to sqlite3_free. Disabled entirely when SQLITE_CONFIG_PAGECACHE is set
  (58416-58422) or nMax<3.
- MEMSYS5 COSTS 1.81x. Same scan: system malloc 2,165,680 vs memsys5 (32 MB CONFIG_HEAP,
  mnReq=32) 3,917,376; mnReq=512 -> 3,936,768; mnReq=4096 -> 4,104,192. Cause is power-of-two
  rounding on the DOMINANT allocation: 4368 -> 8192. Unavoidable at any legal page size because
  szAlloc is always 2^k + 272. NOT TESTED: whether SQLITE_CONFIG_PAGECACHE + CONFIG_HEAP removes
  it by keeping page buffers out of memsys5. Lesson 31 Next Steps item 5.
- THE SOFT HEAP LIMIT WORKS WITHOUT SQLITE_ENABLE_MEMORY_MANAGEMENT, and not by reclaiming.
  sqlite3_release_memory is literally "return 0" without that flag, yet measured heap tracked the
  limit within 1.2% in BOTH builds (limit 200,000 -> 202,400 default / 197,952 with the flag;
  500,000 -> 503,792 / 499,344; 2,000,000 -> 2,004,064 / 1,999,616). The two builds differ by
  4,448 bytes = one page. Mechanism is mem0.nearlyFull -> pcache1UnderMemoryPressure (58204),
  consulted TWICE in pcache1FetchStage2: to refuse a createFlag==1 allocation and to make step 4
  recycle an LRU page. THE TELL: CACHE_MISS was identical at 582 in every row, so no page was
  evicted and re-read -- the cache stopped GROWING, it was not SHRUNK.
  sqlite3HeapNearlyFull() has exactly one other caller in the amalgamation: vdbesort.c:108375,
  the spill arm lesson 28 could not trigger. The soft heap limit is the knob that triggers it.

LOOKASIDE, PREDICTED IN CLOSED FORM THEN MATCHED AT EVERY CONFIGURATION:
setupLookaside (188162): sz = ROUNDDOWN8(sz); if sz <= sizeof(LookasideSlot*) (8) then sz = 0;
szAlloc = sz*cnt; then three branches with LOOKASIDE_SMALL = 128 --
  sz>=384          : nBig = szAlloc/(384+sz), nSm = (szAlloc - sz*nBig)/128
  384 > sz >= 256  : nBig = szAlloc/(128+sz), nSm = (szAlloc - sz*nBig)/128
  sz < 256         : nBig = szAlloc/sz,       nSm = 0
Measured high-water of SQLITE_DBSTATUS_LOOKASIDE_USED, holding statements open until MISS_FULL
climbs: 1200,40 -> 123 (30+93, THE DEFAULT); 1200,100 -> 309 (75+234); 1201,100 -> 309 and a
120,000-byte buffer (ROUNDDOWN8); 384,1000 -> 2000; 256,1000 -> 1334; 248,1000 -> 1000;
128,500 -> 500; 16,100 -> 100; 8,100 -> 0 slots and NO buffer. Every one correct first try.
- THE DEFAULT IS 1200,40 = 48,000 BYTES, not 1200,100 = 120,000, because
  SQLITE_OMIT_TWOSIZE_LOOKASIDE is not defined (24069-24075). Confirmed independently:
  sqlite3_memory_used right after sqlite3_open is 9,184 with lookaside off and 57,184 with the
  default -- exactly 48,000 apart. cnt IS A BYTE BUDGET FACTOR, NOT A SLOT COUNT, since 3.31.0.
- Speed, 300 x prepare+finalize of a 2-way self-join with GROUP BY/HAVING/ORDER BY/LIMIT:
  1200,100 -> 10.4 us; 128,500 -> 9.7 us; lookaside off -> 13.6 us. So 1.31x, and 64,000 bytes of
  small slots BEAT 120,000 bytes of 1200-byte slots on this statement. Treat 1.31x as an UPPER
  bound (prepare is the lookaside-heaviest operation), not a typical figure.
- The ceiling is hard: 400 held statements against 1200,100 gave used=307, hi=309 and 42,818
  MISS_FULL. sqlite3_db_config(SQLITE_DBCONFIG_LOOKASIDE) returns SQLITE_BUSY while any slot is
  out, so it cannot be resized under load.
- TRAP: a request <=128 bytes that finds no small slot FALLS THROUGH and consumes a full-size
  slot (32134-32160). And when bDisable>0 (sz forced to 0) MISS_SIZE is NOT incremented, so
  anStat[1] undercounts by whatever ran inside DisableLookaside regions.

Handed forward: lesson 32 gets xIntegrity (iVersion 4, never exercised), the corrupted databases
lesson 30 produced, btreeInitPage's error path, OptimizationEnabled() and the
SQLITE_TESTCTRL_OPTIMIZATION bit table, SQLITE_TESTCTRL_SORTER_MMAP as a testctrl that changes
measured behaviour, and NOW ALSO the 256-byte overread allowance as the read-path counterpart of
what integrity_check looks for. Lesson 33 (VACUUM) gets the backup API, the fact that VACUUM does
not recompute statistics, that it reruns the CREATE INDEX sorter path once per index, that
CREATE INDEX has usesStmtJournal=1, that sqlite_dbpage's xSync calls sqlite3PagerTruncateImage
which is NOT how VACUUM shrinks, and NOW ALSO that a memory-mapped file cannot be truncated on
Windows so VACUUM and auto-vacuum silently fail to shrink it there (sqlite.org/mmap.html).
STILL OPEN AND RE-HANDED to 32/33: sqlite_stat4-as-a-table (handed to 30 by both 27 and 29,
deliberately not taken there, and not taken here either), and loadStatTbl's allocation cost
(handed here by lesson 28 and NOT done -- no measurement of it this run).

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (carried from lessons 27,
28, 29 and 30 -- STILL NOT READ, now four lessons running), sqlite3Stat4ProbeSetValue,
whereRangeSkipScanEst, the WAL savepoint path (PagerSavepoint.aWalData, sqlite3WalSavepoint,
sqlite3WalSavepointUndo), sqlite3PagerMovepage's subjournal call site, and -- new this run --
memsys3 and memsys5's own internals (only their external cost was measured), the
SQLITE_CONFIG_PAGECACHE fixed-slot arm of pcache1Alloc and pcache1.bUnderPressure, the Win32
mmap implementation at 53541-53660 (read only to confirm it mirrors the unix one), and
sqlite3MemdebugSetType / the MEMTYPE_LOOKASIDE tagging, which only exists under SQLITE_MEMDEBUG.
NOT ATTRIBUTED: the 2 MISS_SIZE events per prepare measured above -- wherePathSolver's
sqlite3StackAllocRawNN (174748) is the obvious candidate but was not confirmed. Lesson 31 Next
Steps item 4.
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
