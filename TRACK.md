<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 32
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
- [x] 32 — Verifying a database: how PRAGMA integrity_check walks the file, and SQLite's test strategy
- [ ] 33 — VACUUM and VACUUM INTO: the temp-database rewrite in vacuum.c, the backup API it shares, and why it is the only way to change auto_vacuum mode

<!--
NOTE BLOCK POLICY (set 2026-10-12, lesson 27). This block holds the scope and handover notes for
the three most recent lessons only. Older notes are not deleted: they are preserved verbatim in
this file's own git history, and TRACK-ARCHIVE.md records exactly which commit and blob to read
for which lessons. When this block grows past three lessons, move the oldest out and update
TRACK-ARCHIVE.md. The lesson documents themselves are the real record; these notes exist only so
that a fresh session knows what a previous run deliberately left alone.

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

scope note 2026-10-17 (lesson 32): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Per the NOTE BLOCK POLICY above, lesson 29's
note was moved out this run and TRACK-ARCHIVE.md gained its row (commit
aea7ae911690ab140bb7036d5588d06369ac10f7, TRACK.md blob
1cb2703f12208654808b5bebde2299d80fddb95a). This block now holds lessons 30, 31 and 32.

Build used: SQLite 3.53.4 amalgamation from better-sqlite3@13.0.3 (deps/sqlite3/sqlite3.c,
269,668 lines -- ALL line numbers in the lesson are into that file), gcc 13.3.0 -O2,
-DSQLITE_THREADSAFE=0 -DSQLITE_ENABLE_DBSTAT_VTAB -DSQLITE_ENABLE_DBPAGE_VTAB, 2-core container.
NEW TEST DATABASE, DIFFERENT FROM LESSONS 28-31: t(a INTEGER PRIMARY KEY, b TEXT, c INTEGER) with
200,000 rows + index tc ON t(c), page_size 4096, rollback-journal mode -- 2,254 pages /
9,232,384 bytes (lessons 28-31 used a wider-row db at 3,836 pages / 15,712,256 bytes). Every
figure here was re-measured on the narrow db; do NOT mix the two page counts. Also built
s25000/s50000/s100000/s200000/s400000.db (275 / 557 / 1,122 / 2,254 / 4,524 pages) for scaling.
Harness: a pass-through VFS (iVersion 1) counting xRead, plus a 60-line sqlite3_module named
ckvtab with iVersion 4 and a logging xIntegrity. No patched SQLite build was needed this run.

LESSON 31'S NAMED HANDOFF IS CLOSED. xIntegrity IS NOW EXERCISED AND MEASURED: mFlags IS EXACTLY
isQuick -- 0 under integrity_check, 1 under quick_check, confirmed by a module that prints its
arguments. OP_VCheck (codegen 146984, impl 105016) is emitted in a SECOND pass over the table
hash (146965-146991) gated on five conditions, the binding one being
pVTab->pModule->iVersion>=4 && xIntegrity!=0. A non-NULL *pzErr with SQLITE_OK is a FINDING, not
an error: it becomes a result row and the check continues. MEASURED AND IMPORTANT: AN EPONYMOUS
VIRTUAL TABLE IS NEVER CHECKED -- the pass walks db->aDb[i].pSchema->tblHash and eponymous tables
are not in it, so dbstat, sqlite_dbpage, pragma_* and generate_series are structurally out of
reach. integrity_check returned plain "ok" with xIntegrity never called. Second trap, mirror of
lesson 30's: a module with xConnect but NO xCreate cannot be used with CREATE VIRTUAL TABLE and
the failure is SILENT -- the statement prepares, steps with no error, and the table does not
exist afterwards. Use designated initialisers for sqlite3_module; positional ones put the
function pointer in xShadowName and only -Wincompatible-pointer-types catches it.

THE CENTRAL STRUCTURAL FACTS OF THIS LESSON (later lessons and any re-read should start here):
- integrity_check is TWO checkers, not one. (a) OP_IntegrityCk (103870) -> C walk
  sqlite3BtreeIntegrityCheck (84370). (b) a GENERATED VDBE PROGRAM (146494 onward) that scans
  every table with every index cursor open. They catch DISJOINT fault sets.
- EXACTLY TWO heap allocations for the whole C walk: aPgRef = sqlite3MallocZero((nCkPage/8)+1)
  (84417) = 282 BYTES on the 2,254-page db, and heap = sqlite3PageMalloc(pageSize) (84423) =
  4,096 bytes = 1,024 u32 slots, mxHeap = pageSize/4-1 = 1023 under SQLITE_DEBUG. The safety
  invariant: a cell costs >=4 content bytes + 2 pointer bytes, so a 4,096-byte page holds at most
  floor(4096/6) = 682 heap entries. Message buffer is STACK: char zErr[100] (84409).
- aRoot FOR main ON THIS SCHEMA IS [2,3,1], THREE ROOTS NOT TWO. Page 1 is checked as an ordinary
  table b-tree because sqlite_schema is an entry in the schema's table hash. temp contributes [1].
  Registers are fixed: reg1 = errors remaining, reg3 = the string OP_ResultRow emits, reg7 = row
  counter, reg8+i = entry count for the i-th root/index. SQLITE_INTEGRITY_CHECK_ERROR_MAX = 100
  (146466).
- checkTreePage ZEROES pPage->isInit (84133) before btreeInitPage so an already-validated cached
  page is RE-validated. That is the whole difference between a checker and a cache tour, and it is
  lesson 31's handed-forward btreeInitPage error path: page 4 type byte 0x0d -> 0x07 gives exactly
  "Tree 2 page 4: btreeInitPage() returns error code 11" (11 = SQLITE_CORRUPT).
- THE MIN-HEAP COVERAGE PROOF: entries are (start<<16)|end (84249), prev starts at
  contentOffset-1 as an IMPLIED entry covering header + cell pointer array + gap (84313), pulls in
  order, overlap -> "Multiple uses for byte %u of page %u", gaps accumulate into nFrag, and
  nFrag is compared with data[hdr+7] ONLY IF heap[0]==0 (84333) so one fault yields one message.
- INDEX KEY ORDER IS NOT CHECKED BY THE C WALK. The ordering test at 84210 is guarded by
  if( pPage->intKey ). Table b-trees get "Rowid %lld out of order"; index b-trees get NOTHING.
- THE ERROR CAP TRUNCATES THE WALK, NOT JUST THE OUTPUT. integrity_check(1) read 1,619 pages,
  integrity_check(20) read 2,258 on the same file. integrityCheckResultRow (145184) is three
  opcodes: OP_ResultRow, OP_IfPos on reg1, OP_Halt. The b-tree layer's messages arrive as ONE
  newline-joined result row, so row count != message count.
- "database disk image is malformed" IS A RESULT ROW, NOT AN ERROR. The pragma writes
  aOp[5].p4.z = sqlite3ErrStr(SQLITE_CORRUPT) (147015) and stores that address in P3 OF
  INSTRUCTION 0; op_column_corrupt (99867) reads aOp[0].p3 and jumps there instead of aborting.
  Only two sites goto op_column_corrupt (99680, 99757), BOTH inside OP_Column -- so a malformed
  RECORD is survivable (final rc 101, DONE) while SQLITE_CORRUPT from any other opcode still
  aborts and sets SQLITE_CorruptRdOnly if a transaction is open.

QUICK_CHECK: THE FOLK SUMMARY IS WRONG AND THE DOCS ARE EASY TO OVER-READ. There is exactly ONE
consequential if( !isQuick ) block (146857) plus the WITHOUT ROWID PK-order test (146822).
quick_check STILL DOES, measured and confirmed in source: the right-most-column fetch that forces
the whole record header to be parsed (146800); "wrong # of entries in index <name>" via OP_Eq on
reg 8+cnt vs 8+iTab, PARTIAL INDEXES EXCLUDED by pIdx->pPartIdxWhere==0 (146591-146618); "NULL
value in <tab>.<col>" with an OP_Column reload because OP_IsType cannot see NaN (146753-146771);
"non-<TYPE> value in" for STRICT via aStdTypeMask[] (146785); the NUMERIC/TEXT affinity checks
(146803-146820); "CHECK constraint failed in <tab>" (146832); and xIntegrity with mFlags=1.
quick_check SKIPS ONLY: the per-row OP_Found index probe, OP_IFindKey, "rowid not at
end-of-record", the non-BINARY collation exact-text re-compare, "non-unique entry in index", and
the WITHOUT ROWID PK-order test.

MEASURED CORRUPTION MATRIX (base.db copied, one targeted byte/field edit, cap 20, both pragmas).
THE TWO ROWS THAT MATTER: (a) ONE PAYLOAD BYTE flipped inside index leaf page 7's cell[0]
(0x9d->0x9e at file offset 28671) -> quick_check "ok", integrity_check "row 96157 missing from
index tc". (b) cell[0] and cell[1] POINTERS SWAPPED on index leaf page 7 (keys now out of order,
coverage and counts untouched) -> quick_check "ok", integrity_check "row 192314 missing from index
tc" -- ONE row reported, not two, because the binary search over the mis-ordered page still finds
one of the swapped pair. Other rows: appended page + header pagecount 2254->2255 -> "Page 2255:
never used" (BOTH pragmas); header freelist count 0->5 -> "Freelist: size is 0 but should be 5"
(BOTH); page 4 type byte -> btreeInitPage 11 + "wrong # of entries in index tc" (BOTH MESSAGES
under quick_check too); page 4 cell[0] ptr -> 0 -> "Offset 0 out of range 297..4092" + "row 1
missing from index tc" (quick_check gets only the first); page 4 cell[0] ptr -> 320 -> "Rowid 111
out of order" + "Multiple uses for byte 320 of page 4" + malformed row (BOTH); frag byte 0->9 ->
"Fragmentation of 0 bytes reported as 9 on page 4" (BOTH); nCell 131->130 -> "Fragmentation of 30
bytes reported as 0" + wrong # of entries (BOTH); page 3 right-child 1870->4 -> "2nd reference to
page 4" + "Child page depth differs" + 18 x "Page N: never used" before the cap.

COST, MEASURED TWO WAYS -- THE HEADLINE IS THAT THE EXTRA I/O IS CACHE PRESSURE, NOT EXTRA PASSES:
- cache_size=-1000000 (whole file fits): quick_check 19.8 ms / 2,254 xRead; integrity_check
  94.1 ms / 2,254 xRead. SAME READ COUNT = the page count. 4.8x CPU, 1.0x I/O.
- default cache (-2000 KiB ~ 488 pages): quick_check 23.0 ms / 3,874 xRead; integrity_check
  121.6 ms / 35,380 xRead = 15.7 READS PER PAGE OF THE FILE, 9.1 extra per page, all from the
  per-row index probes thrashing against the table scan's working set.
- SCALING, second run of each, cache holding the file: quick_check 2.5 / 4.8 / 9.5 / 19.8 / 40.3 ms
  at 25k/50k/100k/200k/400k rows (factors 1.92, 1.98, 2.08, 2.03 -- FLATLY LINEAR);
  integrity_check 9.3 / 24.7 / 45.5 / 96.6 / 213.7 ms (factors 2.66, 1.84, 2.12, 2.21, avg ~2.1).
  Consistent with the documented O(N log N) BUT the log term is only ~5-10% per doubling over a
  16x range and the 2.66 is cache-warmth noise. DO NOT quote the asymptotics as the reason
  integrity_check is slow; the 4.8x constant factor is.
- PROGRESS HANDLER: nOps=1000 -> integrity_check 2,202 callbacks, quick_check 1,002, because
  checkProgress (83817) ticks its own nStep on top of the VDBE instruction counter. nOps=1 ->
  1,002,322 callbacks for 200k rows (~5 VDBE ops/row). A handler returning 1 after 50 calls gave
  rc=9 SQLITE_INTERRUPT at 17.8 ms into a 123 ms check -- the abort lands MID-WALK.
- PARTIAL CHECKS, MEASURED: integrity_check(t) on a db with an orphan page said "ok" (never-used
  sweep skipped) and on a db with a lying freelist count said "ok" (freelist skipped), while
  integrity_check(sqlite_schema) caught the freelist lie. integrity_check(t) took 125.8 ms /
  35,363 reads against the full check's 121.1 ms / 35,380 -- A PARTIAL CHECK IS WEAKER WITHOUT
  BEING FASTER on a single-table schema. integrity_check(sqlite_schema) is 0.1 ms / 1 read and is
  the one genuinely cheap variant. Implementation is bPartial/bCkFreelist at 84390.

TEST-STRATEGY FIGURES TAKEN FROM sqlite.org/testing.html THIS RUN (fetched): TCL 1,390 files /
23.2 MB / 51,445 cases; TH3 50,362 cases / ~2.4 M instances per coverage run / ~248.5 M in a
pre-release soak / 76.9 MB / 1,055.4 KSLOC; SLT 7.2 M queries over 1.12 GB compared against
PostgreSQL, MySQL, SQL Server, Oracle 10g; dbsqlfuzz mutates BOTH the SQL and the database file,
336 seeds, ~1 B mutations/day (~500 M cases/day on 16 cores); library ~155.8 KSLOC vs ~92,053
KSLOC of tests = ~590:1; 100% branch and claimed 100% MC/DC under TH3; 1,184 testcase() and 6,754
assert(). integrity_check is the ORACLE in three places: after I/O-error simulation, in the TCL
crash tests, and in TH3's snapshot-damage tests.

Handed forward to lesson 33 (VACUUM/VACUUM INTO): the never-used-page sweep is exactly the
condition VACUUM exists to eliminate, so a VACUUMed file is the obvious thing to re-check; the
backup API; that VACUUM does NOT recompute statistics; that it reruns the CREATE INDEX sorter
path once per index; that CREATE INDEX has usesStmtJournal=1; that sqlite_dbpage's xSync calls
sqlite3PagerTruncateImage which is NOT how VACUUM shrinks; that a memory-mapped file cannot be
truncated on Windows so VACUUM and auto-vacuum silently fail to shrink it there; and NOW ALSO
that integrity_check reads every page exactly once when the cache fits, which makes it a usable
before/after instrument for a rewrite.
STILL OPEN AND RE-HANDED to 33: sqlite_stat4-as-a-table (handed to 30 by both 27 and 29, then to
31, then here -- NOT taken four times running; it may simply not belong in this track), and
loadStatTbl's allocation cost (handed by 28, not done by 31, not done here).

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (carried from lessons 27,
28, 29, 30 and 31 -- STILL NOT READ, now FIVE lessons running; stop handing it forward unless a
lesson actually needs it), sqlite3Stat4ProbeSetValue, whereRangeSkipScanEst, the WAL savepoint
path (PagerSavepoint.aWalData, sqlite3WalSavepoint, sqlite3WalSavepointUndo),
sqlite3PagerMovepage's subjournal call site, and -- new this run -- sqlite3VdbeFindIndexKey (the
body of OP_IFindKey; only its opcode comment and call site were read, and NO measurement
triggered it, so the 2-ULP tolerance is UNVERIFIED and is lesson 32 Next Steps item 1), every
checkPtrmap call path (compiled in but never exercised -- the test db is not auto-vacuum, so the
"Bad ptr map entry" message was never produced: Next Steps item 3), the WITHOUT ROWID arm of the
generated program (pPk, r2, 146822 -- never run), and sqlite3_recover / the recovery extension,
which was not looked at at all.
NOTE FOR ANY FUTURE RUN: the lesson's 2,254-page figures CANNOT be compared against lessons 28-31's
3,836-page figures. Different schema, different row width, same page_size.
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
