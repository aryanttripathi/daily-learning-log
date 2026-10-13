<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 28
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

- [ ] 29 — Transactions at the SQL level: autocommit, savepoints, and statement journals
- [ ] 30 — Virtual tables: the xBestIndex protocol, plus sqlite_dbpage and dbstat introspection
- [ ] 31 — Memory-mapped I/O and memory allocation (mmap_size, memsys, lookaside)
- [ ] 32 — Verifying a database: how PRAGMA integrity_check walks the file, and SQLite's test strategy
- [ ] 33 — VACUUM and VACUUM INTO: the temp-database rewrite in vacuum.c, the backup API it shares, and why it is the only way to change auto_vacuum mode

<!--
NOTE BLOCK POLICY (set 2026-10-12, lesson 27). This block holds the scope and handover notes for
the three most recent lessons only. Older notes are not deleted: they are preserved verbatim in
this file's own git history, and TRACK-ARCHIVE.md records exactly which commit and blob to read
for which lessons. When this block grows past three lessons, move the oldest out and update
TRACK-ARCHIVE.md. The lesson documents themselves are the real record; these notes exist only so
that a fresh session knows what a previous run deliberately left alone.

scope note 2026-10-11 (lesson 26): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 26 needed aiRowLogEst[] only as an
INPUT and nothing from ANALYZE's machinery, so the 26/27 boundary is clean: lesson 26 owns the
WhereLoop object, the five cost formulas, whereLoopInsert/FindLesser, wherePathSolver and
whereShortCut; lesson 27 owns every question about where the numbers come from.

Both of lesson 25's named handoffs are CLOSED.

(a) sqlite3WhereOkOnePass (where.c:173-183) is a pure getter -- memcpy of aiCurOnePass[] plus
`return pWInfo->eOnePass`. The decision is at where.c:7228-7248, after the solver. ONEPASS_SINGLE
is nothing but WHERE_ONEROW on the chosen loop, which is why a DELETE by rowid gets it: whereShortCut
sets WHERE_COLUMN_EQ|WHERE_IPK|WHERE_ONEROW at where.c:6392-6398. ONEPASS_MULTI additionally
requires WHERE_ONEPASS_MULTIROW from the caller, a non-virtual table, and either no WHERE_MULTI_OR
or WHERE_DUPLICATES_OK (DELETE only) -- the comment at 7217-7227 says the OR exclusion exists
because UPDATE reads aiCurOnePass[1] and that is "not set accurately for scans that use the OR
optimization". One finding not in lesson 25: when ONEPASS is granted on a rowid table whose chosen
loop was COVERING, where.c:7240-7245 STRIPS WHERE_IDX_ONLY from the already-selected WhereLoop and
does not recompute rRun. The plan is costed as index-only and then run as not-index-only.

(b) The affinity-string trimming is codeApplyAffinity(), wherecode.c:446-481, NOT in where.c at all,
and it is NOT the same rule as sqlite3TableAffinityStr's backward loop. It trims BOTH ends on
`zAff[i] <= SQLITE_AFF_BLOB` (so NONE or BLOB), the leading loop ALSO ADVANCES `base`, and the two
guards are asymmetric (leading `n>0`, trailing `n>1`); if nothing survives, no opcode is emitted.
That explains lesson 25's "Affinity p1=2 p2=3 p4='CCB' for a four-column index with four constrained
terms" completely: the leading character was BLOB, so it was stripped AND p1 was advanced by one.
Measured four ways on 3.45.1 with the A=BLOB/B=TEXT/C=NUMERIC coding from lesson 24:
  q(w, x numeric, y numeric, z text)          'ACCB' -> Affinity p1=2 p2=3 p4='CCB'
  q2(w numeric, x numeric, y numeric, z text)  'CCCB' -> Affinity p1=1 p2=4 p4='CCCB'
  q3(w numeric, x numeric, y numeric, z)       'CCCA' -> Affinity p1=1 p2=3 p4='CCC'
  q4(w, x, y, z)                               'AAAA' -> no OP_Affinity emitted
Caller is codeAllEqualityTerms at wherecode.c:941; updateRangeAffinityStr (494-508) separately
rewrites characters to BLOB for range terms.

Lesson 24's open `tcol = CAST(icol AS TEXT)` row is NOT closed. It now has a named place to look --
sqlite3ExprNeedsNoAffinityChange, called from updateRangeAffinityStr -- and that is lesson 26's
Next Steps item 7.

Measured facts later lessons can rely on (all on stock 3.45.1, no debug build, SQLITE_ENABLE_STAT4
confirmed ABSENT via PRAGMA compile_options, so where.c:4172's #else branch is the live one and
full-scan rRun == rSize+16 unconditionally):

- sqlite3LogEst/LogEstAdd/LogEstToInt/estLog were reimplemented from util.c:2055-2128 and verified
  against all nine assert()s in where.c and build.c: LogEst(28)=48, LogEst(2)=10, LogEst(20)=43,
  LogEst(18)=42, LogEst(10)=33, LogEst(15)=39, LogEst(1000)=99, LogEst(5)=23, LogEst(1)=0.
- estLog(LogEst N) applies sqlite3LogEst to a LogEst, giving LogEst(log2(N)) -- the b-tree depth.
  Measured: estLog(99)=33 (10 units), estLog(166)=40 (16), estLog(199)=43 (20), estLog(265)=47 (26).
  Any later lesson quoting rLogSize must use this, not rSize-33.
- THE CENTRAL RESULT. For a single table with one == term, non-covering index, the SEARCH/SCAN
  crossover is closed-form and was predicted to the single row at four table sizes with ZERO
  mismatches. Formulas: scan rRun = rSize+16; index rRun = LogEstAdd(LogEstAdd(rLogSize,
  nOut+1+(15*szIdxRow)/szTabRow), nOut+16). Measured boundaries for t(a integer, b text) index (a),
  szTabRow=LogEst(28)=48, szIdxRow=LogEst(8)=30, ratio term = 9:
    N=10000     flip between k=5631 (rRun 146) and 5632 (148), scan 148
    N=100000    flip between k=61439 (181) and 61440 (182), scan 182
    N=1000000   flip between k=589823 (213) and 589824 (215), scan 215
    N=10000000  flip between k=5767167 (246) and 5767168 (248), scan 248
  TIE RULE, measured four times: equal cost goes to the FULL TABLE SCAN, because whereLoopAddBtree
  inserts the IPK loop before looping over real indexes and every comparison in wherePathSolver
  (where.c:6071-6085) is strict `<`. The crossover ratio k/N is 0.56-0.61, NOT the 10-20% folklore.
- szTabRow and szIdxRow are LogEst(width*4) from build.c:2305-2330, fully determined by the DDL.
  Column.szEst is 1 for INTEGER, 5 for a bare TEXT/BLOB/CLOB (v=16, then v/4+1 at build.c:1771),
  k/4+1 for VARCHAR(k). estimateIndexWidth iterates nColumn, so a rowid table's index includes the
  trailing rowid at weight 1. Changing a column's DECLARED TYPE moves the crossover with no change
  to any stored byte.
- Skip-scan fires at exactly 18 duplicates, never 17 (where.c:3633, aiRowLogEst[nEq+1]>=42).
  Measured k=14..23 with stat1 "1000000 k 1": SCAN at 14-17, "ANY(a) AND b=?" at 18+. LogEst(16)=
  LogEst(17)=40 and LogEst(18)=42 -- there is NO integer with LogEst 41 in that range, because of
  the 3-bit mantissa in sqlite3LogEst's a[] table. Several of where.c's integer thresholds may sit
  in such gaps; enumerating them is lesson 26's Next Steps item 5.
- Automatic-index boundary derived from rSetup = rLogSize+rSize+28 (where.c:4091-4104) composed
  through the solver's cost rule (where.c:5970-5979), then measured: for a 1,000,000-row inner table
  the auto-index appears at exactly 9 outer rows. Path A (scan inner, auto-index outer) is flat at
  246 across 6..12 rows because rSetup is >31 LogEst units below the inner scan and LogEstAdd
  discards it; path B (scan outer, scan inner) moves 241/243/245/247/248/251. Predicted = measured
  at every point. The solver also REVERSED the join order and added a Bloom filter.
- The solver runs TWICE when there is an ORDER BY: wherePathSolver(pWInfo,0) to get nRowOut, then
  whereInterstageHeuristic (6313-6356, sets prereq=ALLBITS on unconstrained loops of tables the
  first pass searched), then wherePathSolver(pWInfo,nRowOut+1). where.c:7115-7121.
- mxChoice = 1 / 5 / 12 / 18 (where.c:5871-5888, computeMxChoice 5663-5810 returns
  bStarUsed?18:12). This matches queryplanner-ng.html's published N values exactly.
- whereLoopOutputAdjust RETURNS EARLY for a single table with no ORDER BY (where.c:3074-3079).
  That is why the crossover predictions above needed no adjustment term, and it means the same
  table+index+WHERE gets a different nOut once it appears in a join.
- whereShortCut cannot use a UNIQUE index with 4+ key columns: aLTermSpace[3] (whereInt.h:174,
  tested at where.c:6407). Verified that 3-column and 4-column cases produce IDENTICAL
  EXPLAIN QUERY PLAN output, so the shortcut is invisible from outside and must be measured at
  prepare() time, not query time. That is lesson 26's Next Steps item 4.

Two comment-versus-code disagreements recorded, both in where.c:4158-4176:

- The comment says the STAT4 full-scan penalty is "2.75". The code is `rSize + 16 - 2`, and
  LogEst 14 is 2**1.4 = 2.639, while 10*log2(2.75)=14.59 would round to 15. The code is 2.64x,
  one unit more aggressive than its own comment. ("3.0" for LogEst 16 = 3.031 is honest.)
  The same rounding slack appears at where.c:3644, where a "1.375 fudge factor" is coded as +5,
  i.e. 1.414.
- UNRESOLVED, handed to lesson 27. The comment claims that at 2.75 "a full table scan is preferred
  over using an index on a column with just two distinct values where each value has about an equal
  number of appearances." Substituting nOut = LogEst(N/2) gives index rRun = rSize+13 at N = 1e4,
  4e5, 1e6, 1e7 and 1e8 -- below rSize+14 as well as rSize+16, so the 2-unit reduction does NOT
  flip the decision; the index still wins by one unit (~7%). The claim can therefore only hold if
  STAT4 also replaces nOut via whereEqualScanEst. That is a derivation, not a measurement, because
  this build has STAT4 off. Lesson 27 should settle it with a -DSQLITE_ENABLE_STAT4 build and
  WHERETRACE 0x2.

Cost-model accuracy, measured with a stopwatch (file-backed, cache large enough to hold the table,
best of 5, forced full scan via the unary-+ trick):

- Uniform distribution, 400,000 rows, stat1's average IS the truth. Real crossover is near 10
  distinct values (index/scan = 0.90); the model's is at k/N ~ 0.58. In between the planner keeps
  the index and loses: 1.53x at 4 distinct values, 2.41x at 3, 3.20x at 2. At 1 distinct value the
  model finally says SCAN and the two are equal (1.02x).
- Skewed distribution, same table, matching fraction varied 5%..95% over 8 total values: stat1's k
  stayed at exactly 50000 (= N/8) in EVERY run, because stat1 stores only the average. The correct
  answer moved by 12x while the planner's input did not move at all. At 95% the index costs 1.64x.
  This is the motivating measurement for lesson 27 and for sqlite_stat4's existence.

Handed forward: lesson 27 gets the stat1/stat4 format and all four estimators (whereKeyStats 1713,
whereRangeScanEst 2087, whereEqualScanEst 2269, whereInScanEst 2333), sqlite3AnalysisLoad, the
hasStat1 flag, and the unresolved 2.75 claim above. Lesson 28 gets whereSortingCost's
implementation side -- OP_SorterInsert vs OP_IdxInsert and the "extra factor of 2.0 or 3.0" the
comment at where.c:5562-5564 mentions but does not code here -- plus the external merge itself.
Lesson 29 gets WHERE_ONEPASS_DESIRED / WHERE_ONEPASS_MULTIROW / WHERE_DUPLICATES_OK as a
caller-side contract and sqlite3MultiWrite. Lesson 30 gets whereLoopAddVirtual (4685),
whereLoopAddVirtualOne (4361), allocateIndexInfo (1408), vtabBestIndex (1669), the
sqlite3_vtab_collation/in/rhs_value/distinct accessors (4557-4646) and the whole xBestIndex
protocol, all untouched here. Lesson 31 gets sqlite3StackAllocRawNN, which wherePathSolver uses for
its aTo/aFrom/aSortCost block (where.c:6042-6049). Lesson 32 gets OptimizationEnabled() and the
SQLITE_TESTCTRL_OPTIMIZATION bit table that where.c references in six places (SQLite's own
".testctrl opt -starquery" is named in computeMxChoice's comment).

scope note 2026-10-12 (lesson 27): no lesson added, removed, reordered or split;
lessons-total stays 33. Structural change to THIS FILE instead: the per-lesson scope notes for
lessons 13-24 were moved out, because this comment block had reached 68 KB and every run has to
rewrite the whole file to tick one checkbox. They are preserved verbatim in this file's git
history and TRACK-ARCHIVE.md records exactly where. Nothing was deleted.

Lesson 27 settled the question lesson 26 handed forward, and the answer was "both of you were
right about different tables":
- Built two binaries from ONE SQLite 3.53.4 amalgamation (the copy vendored in the better-sqlite3
  npm package at deps/sqlite3/sqlite3.c), differing only in -DSQLITE_ENABLE_STAT4; a third adds
  -DSQLITE_DEBUG -DSQLITE_ENABLE_WHERETRACE. NOTE FOR FUTURE RUNS: WHERETRACE needs BOTH
  SQLITE_DEBUG and SQLITE_ENABLE_WHERETRACE (sqlite3.c's macro is
  #if defined(SQLITE_DEBUG) && (defined(SQLITE_TEST) || defined(SQLITE_ENABLE_WHERETRACE))).
  Defining only the latter compiles and traces nothing. sqlite3WhereTrace is SQLITE_PRIVATE in the
  amalgamation, so the harness must #include "sqlite3.c" to set it; a separate TU cannot.
- THE RESULT. t(a INTEGER, b <type>), 400,000 rows, a = i%2 (two values, equal counts), index on
  (a), WHERE a=0. rSize = LogEst(400000) = 186. Scan rRun = 202 without STAT4 and 200 with it
  (rSize+16 and rSize+16-2, both read verbatim out of WHERETRACE=0x8). Index rRun depends on the
  DECLARED TYPE OF b, which the query never mentions:
    b INTEGER / VARCHAR(1)  index 201  -> STAT4 picks SCAN (200), no-STAT4 picks the index (201)
    b VARCHAR(4)            index 200  -> STAT4 picks SCAN on the TIE, no-STAT4 picks the index
    b TEXT / VARCHAR(40)    index 199  -> both builds pick the index
    b VARCHAR(400)          index 198  -> both builds pick the index
  So where.c:4160's "at 2.75 a full table scan is preferred over using an index on a column with
  just two distinct values" is TRUE, for narrow tables, and lesson 26's derivation (index wins by
  one unit) was also true, for its own table, which was t(a integer, b text). The flip boundary is
  a szTabRow boundary: Column.szEst 2 for VARCHAR(4) vs 5 for bare TEXT. Not one stored byte
  differs between the two databases. Exact k in VARCHAR(k) where it flips: NOT narrowed; left as
  lesson 27's Next Steps item 2.
- "equality scan regions t_a(0): 200000" -- with two equal-count values STAT4's own estimate EQUALS
  stat1's second integer, so the 2-unit rRun discount is the entire difference between the builds.
  This is why lesson 26's substitution of nOut = LogEst(N/2) was the right substitution.
- TIE RULE, a fifth confirmation: at b VARCHAR(4) the STAT4 build prices both loops at 200 and the
  trace prints "skip:" for the index loop -- it is never inserted, because whereLoopAddBtree adds
  the IPK loop first and whereLoopInsert needs strictly better.
- The arithmetic complaint from lesson 26 STANDS: LogEst 14 = 2.639, the comment says 2.75, the
  code is one unit more aggressive than its own prose. Only the DECISION the comment describes was
  wrong-looking, not the discount.

Other measurements lesson 27 owns, all original, all on the binaries above:
- stat1 arithmetic: I = ceil(nRow/(anDLt[i]+1)) at analyze.c:868-872, with the special case
  "if( iVal==2 && p->nRow*10 <= nDistinct*11 ) iVal = 1". First integer is nSkipAhead ? nEst : nRow.
- WHICH stat1 ROWS EXIST. Measured: noidx|<NULL>|1000, withidx|wi|1000 143, wo|wo|1000 1. The
  idx-IS-NULL row appears ONLY when the table has no non-partial index: needTableCnt is cleared at
  analyze.c:1067 by any index with pPartIdxWhere==0, and the OP_Count row is emitted only at
  analyze.c:1362 under "pOnlyIdx==0 && needTableCnt". For an indexed table the row count comes from
  analysisLoader's pTable->nRowLogEst = pIndex->aiRowLogEst[0] (analyze.c:1636). WITHOUT ROWID uses
  idx == tbl (resolved by the sqlite3_stricmp(argv[0],argv[1]) branch).
- NOTHING IN SQLITE WRITES "sz=", "unordered" OR "noskipscan". grep -c '"sz="' sqlite3.c -> 0.
  decodeIntArray (analyze.c:1556-1575) only READS them. They are a human-only hint channel.
  MEASURED: hand-writing 'unordered' into stat1 then "ANALYZE sqlite_schema" turns
  "SCAN t USING COVERING INDEX t_a" into "SCAN t + USE TEMP B-TREE FOR ORDER BY" and makes a
  BETWEEN range abandon the index. WITHOUT the reload the edit is invisible -- stats are read once
  at schema load. sz=4000 parsed but did NOT move an equality plan (cost 127 either way); what it
  CAN move is lesson 27's Next Steps item 3, still open.
- STAT4 SAMPLES, 400,000 rows, a=0 for 380,000 and a=1..20 for 1,000 each. stat1 "400000 19048".
  sqlite_stat4 holds exactly 24 rows (SQLITE_STAT4_SAMPLES). 8 are PERIODIC with nLt[1] at 44444,
  88889, 133334, ... stride 44445 = nEst/9+1 = nEst/(mxSample/3+1)+1, predicted before looking.
  16 non-periodic, one per surviving distinct value -- and a = 2, 10, 12, 16 HAVE NO SAMPLE: all
  twenty have nEq[0]=1000, so sampleIsBetter fell through sampleIsBetterPost to the iHash LCG
  tiebreak (iPrn = 0x689e962d*nCol ^ 0xd0944565*nEst, advanced by *1103515245+12345). Predicting
  that surviving set from the seed is lesson 27's Next Steps item 5.
- aAvgEq DERIVED AND CONFIRMED TO THE UNIT. Sampled values estimate 1000; UNSAMPLED values (a=2)
  and NONEXISTENT values (a=999) both estimate 1002, via whereKeyStats' aStat[1]=aAvgEq[nField-1].
  initAvgEq: nDist100 = 100*400000/19048 = 2099 (truncating 2099.75); 17 samples end a distinct
  prefix run (the last of the 8 periodics contributes 380000, the 16 singletons 1000 each) so
  sumEq=396000, nSum100=1700; avgEq = 100*(400000-396000)/(2099-1700) = 400000/399 = 1002.5 -> 1002.
  Both integer truncations are load-bearing. A nonexistent value is NEVER estimated at 0.
- whereInScanEst sums: a IN (2,5,9) -> 1002+1000+1000 -> "IN row estimate: est=3002", exact.
- whereRangeScanEst: a>5 -> "STAT4 range scan: 385000..400000 est=138", and 15000 IS the true count
  (both bounds landed on sample boundaries). The unanalysed guess would have been 166 (a quarter of
  the table), wrong by 6.7x. The "if( iLwrIdx==iUprIdx ){ nNew -= 20; }" 4x rule did not fire in
  any measured case; finding a query where it does is NOT done.
- THE STAT4 HEADLINE, same skewed table. stat1 alone prices WHERE a=0 and WHERE a=5 IDENTICALLY at
  165 and picks the index for both -- 380,000 rows reached through an index. With STAT4: a=0 ->
  SCAN (200), a=5 -> SEARCH (122). Timed, best of 4x8 reps, hot cache: no-STAT4 index path 25.1 ms
  vs forced scan 18.3 ms = 1.37x penalty; STAT4 17.9 ms. Modest BECAUSE the table is cached; the
  cold-cache version is lesson 27's Next Steps item 4, not done.
- PRAGMA analysis_limit IS DANGEROUS, measured: limit 0 -> stat1 "400000 19048"; limit 100 ->
  "400000 101", off by 190x, and the resulting plan costs 90 instead of 165 for BOTH a=0 and a=5 --
  the planner became MORE confident while becoming less correct. And mxSample = nLimit==0 ? 24 : 0,
  so sqlite_stat4 came back with 0 rows under the limit, silently. PRAGMA optimize turns a
  temporary limit on BY DEFAULT (bit 0x10), measured writing "50000 96" where the truth is 100.
  PRAGMA optimize(-1) prints the statement it would run, verbatim: 'ANALYZE "main"."t"'. It prints
  nothing on a connection where stats are current and no query touched the table.

Handed forward: lesson 28 gets whereSortingCost's implementation side -- OP_SorterInsert vs
OP_IdxInsert and the "extra factor of 2.0 or 3.0" the comment at where.c:5562-5564 mentions but does
not code there -- plus the external merge itself, and should check whether STAT4's sharper nOut
improves the sorter estimate or is swamped by that factor. Lesson 29 still gets
WHERE_ONEPASS_DESIRED / WHERE_ONEPASS_MULTIROW / WHERE_DUPLICATES_OK and sqlite3MultiWrite.
Lesson 30 still gets the whole xBestIndex protocol (whereLoopAddVirtual 4685,
whereLoopAddVirtualOne 4361, allocateIndexInfo 1408, vtabBestIndex 1669, the
sqlite3_vtab_collation/in/rhs_value/distinct accessors 4557-4646), and now ALSO gets sqlite_stat4
as a table that can be SELECTed and UPDATEd like any other -- the hand-edit technique in lesson 27
section 8.2 is a vtab-adjacent introspection trick worth repeating there. Lesson 31 gets
sqlite3StackAllocRawNN and should price the per-connection cost of loadStatTbl's allocation
(IndexSample[nSample] plus 3*nIdxCol*nSample tRowcnts per index, paid at every schema load).
Lesson 32 gets OptimizationEnabled() and the SQLITE_TESTCTRL_OPTIMIZATION bit table, and now also
SQLITE_Stat4 as one of those bits (it gates both loadStat4 and statInit's mxSample). Lesson 33
(VACUUM) should note that VACUUM does NOT recompute statistics.

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (the unanalysed fallback --
lesson 26 read the szTabRow part of build.c but not this function), sqlite3Stat4ProbeSetValue and
the rest of the stat4 probe-record machinery in vdbemem.c, and whereRangeSkipScanEst
(where.c, called when nEq != nRecValid).

scope note 2026-10-13 (lesson 28): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 28 needed lesson 26's LogEst/estLog
arithmetic and lesson 24's record-comparison semantics as inputs, lesson 02's record format to
decode PMA bytes, and lesson 25's p5 flag table to read OPFLAG_USESEEKRESULT off a CREATE INDEX;
nothing else. Also trimmed: lesson 25's note moved out of this block per the policy above and
indexed in TRACK-ARCHIVE.md (commit 5429a52018a46573b419ac99bf80d8fb3b0777b9, TRACK.md blob
29cf5381c8b006917033a062727d9e6ce68b2cfb). Notes for 26, 27, 28 retained.

Both of lesson 27's named handoffs are CLOSED.

(a) THE 2.0x/3.0x FACTOR IS CODED, in whereSortingCost itself, as two TUNING lines:
"rSortCost += 10" under WHERE_USE_LIMIT (LogEst 10 = 2.00x) and a further "+= 6" when
nSorted!=0 (total +16 = 2**1.6 = 3.03x). The comment that says the factor applies "if the sort
is built using OP_IdxInsert and OP_Sort rather than with OP_SorterInsert" and the TUNING comment
that says "Extra 2.0x if using LIMIT" describe THE SAME TWO LINES, because WHERE_USE_LIMIT ==
SF_FixedLimit and SF_FixedLimit implies p->iLimit!=0, which is the condition that selects
OP_IdxInsert. Lesson 27's reading of the comment was correct.

(b) THE CENTRAL FINDING, and the one later lessons are most likely to need. The switch between
the two sorts is ONE LINE in sqlite3Select at tag-select-0650:
"if( p->iLimit==0 && sSort.addrSortIndex>=0 ){ ChangeOpcode(OP_SorterOpen); sortFlags |=
SORTFLAG_UseSorter; }". An OP_OpenEphemeral is emitted speculatively and REWRITTEN IN PLACE to
OP_SorterOpen only when there is no LIMIT clause of ANY kind. Measured on EXPLAIN: ORDER BY
alone -> SorterOpen/SorterInsert/SorterSort/SorterData/SorterNext + OpenPseudo (a sorter cursor
cannot answer OP_Column); LIMIT 10, LIMIT 10 OFFSET 5, LIMIT ?, LIMIT -1 and LIMIT (SELECT 10)
ALL -> OpenEphemeral/Sequence/IdxInsert/Sort/Next. pushOntoSorter's bSeq = !(UseSorter) adds the
OP_Sequence tie-breaker column, measured as exactly +1 VDBE instruction per row (vmstep
2,000,009 -> 2,200,009 on 200k rows), because a b-tree needs a unique key and the sorter does
not (vdbeSorterMerge takes p1 on res<=0, so the merge is stable by construction).
THE MODEL AND THE CODE GENERATOR DISAGREE: LIMIT ? and LIMIT -1 get useLimit=0, iLimit=320 and
sort cost 254 -- byte-identical to no LIMIT -- while generating the b-tree plan. Measured
best-of-five on 200k rows: sorter 46.5 ms / 3,282,330 temp bytes; LIMIT -1 127.3 ms /
3,465,216 temp bytes = 2.74x SLOWER for 1.06x the bytes. The cost is b-tree insertion
(lessons 11-12), not I/O. LIMIT -1 is semantically unbounded and is a pure 2.7x pessimisation.
LIMIT 10 itself is a BOUNDED HEAP, not a sort: IfNotZero/Last/IdxLE/Delete run before the insert
so the ephemeral index never exceeds LIMIT rows (9.6 ms, zero temp bytes), and non-ORDER-BY
columns are loaded only after the pruning (pSort->pDeferredRowLoad / innerLoopLoadRow:
Column 0,1 at addr 5 but Column 0,0 at addr 11).

Measured facts later lessons can rely on (all on SQLite 3.53.4 built from better-sqlite3@13.0.3's
deps/sqlite3/sqlite3.c, gcc -O2, 2-core container, page_size 4096, threads 0):

- THE FLUSH THRESHOLD IS A MEMORY THRESHOLD. mnPmaSize = SQLITE_SORTER_PMASZ(250) * pgsz;
  mxPmaSize = MAX(mnPmaSize, MIN(cache_size bytes, SQLITE_MAX_PMASZ = 1<<29)). Spill test is
  "iMemory + nReq > mxPmaSize" with iMemory += ROUND8(nVal + sizeof(SorterRecord)) = ROUND8(nVal+16)
  and szPMA += nVal + varintLen(nVal). Every record on the test table was 11-16 bytes, so every one
  cost EXACTLY 32 bytes of iMemory: PMAs held 64000/64000/64000/8000 records at mxPmaSize 2,048,000
  (= 2048000/32, exact four times over). On-disk PMA content was 1,010,793-1,085,898 bytes, about
  49% of the threshold -- a 1.95x memory inflation that is pure SorterRecord header plus rounding.
- SPILL BOUNDARY PREDICTED TO THE SINGLE ROW: 200,000 x 32 = 6,400,000, so cache_size=-6250
  (6,400,000) spills NOTHING and -6249 (6,398,976) spills everything. Confirmed first try.
- mnPmaSize FLOOR CONFIRMED: cache_size -100 and -500 give byte-identical spill (3,282,339,
  7 PMAs) because both clamp UP to 250*4096 = 1,024,000. The docs for SQLITE_CONFIG_PMASZ say
  threads launch when content exceeds page_size times the "MINIMUM of the PRAGMA cache_size
  setting and this value"; the code is MAX and the measurement agrees with the code. mnPmaSize
  appears in exactly two places in vdbesort.c -- that MAX, and the
  "szPMA > mnPmaSize && sqlite3HeapNearlyFull()" arm of the per-record allocation mode -- and in
  NO thread-launch test.
- PMA FORMAT VERIFIED BY HEXDUMP. One leading varint = total content bytes, then varint-length-
  prefixed records packed end to end, no padding/checksum/page structure. First bytes:
  "bd d8 69" = 1,010,793 (content length); "0b" = 11; "03 17 03 72 6f 77 2d 31 06 87 17" =
  header 3, serial types 0x17 (text len 5) and 0x03 (3-byte int), "row-1", 428823. Field order is
  (orderby..., data...) from makeSorterRecord's OP_MakeRecord regBase+nOBSat, nBase-nOBSat.
  A PMA is sorted in sqlite3VdbeRecordCompare order, NOT byte order: a memcmp check reports
  unsorted for every PMA while decoding field 1 reports sorted for every PMA, because the leading
  serial-type byte varies with text length ("row-9" 0x17 sorts before "row-10" 0x19 byte-wise).
- SQLITE_DBSTATUS_TEMPBUF_SPILL (verb 13) IS EXACT FOR THE SORTER AND BLIND TO EPHEMERAL TABLES.
  Matched an independent VFS-shim xWrite byte count to the byte (3,282,330 = sum of four PMA
  content lengths 3,282,318 + four 3-byte varints). But it reported 0 for the LIMIT -1 sort that
  wrote 3,465,216 bytes, because its b-tree arm reads only db->aDb[1].pBt (the TEMP database) and
  an OP_OpenEphemeral table is a separate transient Btree. The documentation claims it covers
  "writes to intermediate tables that are part of complex queries". It also only lands at
  sqlite3VdbeSorterClose, i.e. at finalize/reset -- a read before finalize sees 0.
- THE 16-PMA CLIFF, predicted to the single page of cache_size. SORTER_MAX_MERGE_COUNT = 16 and
  vdbeSorterTreeDepth returns 0 for <=16 PMAs. With page_size=512 (mnPmaSize 128,000) and a
  POSITIVE cache_size: 782 pages -> mxPma 400,384 -> 12,512 rec/PMA -> 16 PMAs -> 1 temp file,
  3,282,366 bytes, 56.5 ms. 781 pages -> 399,872 -> 12,496 rec/PMA -> 17 PMAs -> 2 temp files
  (pTask->file2, used by the single-threaded IncrMergers), 6,564,686 bytes = 2.0001x, 67.4 ms.
  EVERY RECORD IS WRITTEN TWICE above the cliff. The wall-clock ratio was only 1.19x BECAUSE the
  3 MB temp file stayed in the host page cache -- do not quote 1.19x as the cost of the extra
  merge level; the BYTE ratio is the reproducible result. Depth 2 begins at 257 PMAs.
- typeMask: three gates at Init (nAllField<13, aColl[0]==0 or ==db->pDfltColl, and
  KEYINFO_ORDER_BIGNULL clear), then narrowed per Write by the serial type of field 1.
  nAllField = (ORDER BY terms) + (result columns) + 1, measured: 1 orderby + 10 cols = 12 keeps
  0x01; + 11 cols = 13 loses it; 2 orderby + 9 cols = 12 keeps, + 10 cols = 13 loses.
  Timed on a 300k single-key in-memory sort, best of 3: INTEGER fast path 85.4 ms vs 114.7 with
  NULLS LAST (1.34x); TEXT 114.1 vs 159.4 (1.40x); COLLATE NOCASE 191.1. THE CONTROL that makes
  it the BIGNULL BIT and not null semantics: DESC keeps 0x01 (NULLS LAST is DESC's default, bit
  clear, 100.0 ms) while DESC NULLS FIRST loses it (131.0 ms). Verified by printing typeMask at
  Rewind from a patched build. On a 12-FIELD record the effect was unmeasurable (209 vs 212 ms) --
  record decode dominates there.
- whereSortingCost DECOMPOSED EXACTLY from a patched build: nCol = LogEst((nExpr+59)/30) is a
  THREE-STEP STAIRCASE -- 10 for 1-30 result columns, 16 for 31-60, 20 for 61+ -- so "sorting cost
  proportional to the number of output columns" buys nothing for essentially every real query.
  No LIMIT: 201 + 10 + estLog(201)=43 -> 254. LIMIT 10: 201 + 10 + 10 + estLog(33)=17 -> 238,
  i.e. LOWER than the unlimited sort, correctly, because nRow collapses to the limit before
  estLog. LIMIT 100000 -> 261, above 254. pWInfo->iLimit is a LogEst, not a row count (33 =
  LogEst(10)), arriving as sqlite3WhereBegin's iAuxArg from p->nSelectRow (320 when unlimited).
  WHERETRACE 0x002 prints "---- sort cost=N (Y/X) increases cost A to B".
- sqlite3DefaultRowEst MEASURED (lesson 27 listed it as unexamined). An unanalysed table gets
  rSize = LogEst 200 = 2**20 = 1,048,576 assumed rows: nRowIn=201 and sort cost 254 before
  ANALYZE, nRowIn=177 and 229 after (stat1 "t|NULL|200000", LogEst(200000)=176), with rUnsort
  216 -> 192. A 5.2x cardinality over-estimate inflates the sorting cost by 25 LogEst units.
  The FUNCTION ITSELF IS STILL NOT READ -- only its effect. Lesson 33 or any later cost work
  should read it; it sets the baseline for every number in lessons 26-28.
- CREATE INDEX drain loop: SorterSort / SorterData / OP_SeekEnd / IdxInsert p5=16
  (OPFLAG_USESEEKRESULT, lesson 25's flag table) / SorterNext. Sorting first converts random
  b-tree insertion into pure appends to the rightmost leaf -- that is why the sorter exists for
  DDL. CREATE UNIQUE INDEX is the ONLY user of OP_SorterCompare: "SorterCompare 3 22 9 1" then
  "Halt 2067 2 0 t.b" (2067 = SQLITE_CONSTRAINT_UNIQUE), i.e. uniqueness is checked between
  ADJACENT keys in the sorted stream, never by a per-row index probe.
- PRAGMA threads=4 on this 2-core box: nTask=5, 14 temp files, 9,846,966 spill bytes (3.0x the
  single-threaded 3,282,330) and 62 ms against 46.5. Each task writes its own level-0 PMAs and
  each multi-threaded IncrMerger keeps two staging files. threads=0 is the right default on
  small machines; a later lesson quoting multi-threaded sorter numbers must say the core count.
- Build properties: SQLITE_ENABLE_SORTER_REFERENCES is NOT defined in this amalgamation, so every
  measurement carries the full row through the sorter (the documented default for
  SQLITE_CONFIG_SORTERREF_SIZE is "never use this optimization" anyway).
  SQLITE_TESTCTRL_SORTER_MMAP(db, 0) gates BOTH vdbeSorterMapFile AND vdbeSorterExtendFile's
  CHUNK_SIZE/SIZE_HINT preallocation.

UNRESOLVED, handed to lesson 31 (mmap_size). vdbeSorterOpenTempFile explicitly sets
SQLITE_FCNTL_MMAP_SIZE to SQLITE_MAX_MMAP_SIZE on the PMA temp file, so it is MEANT to be mapped
regardless of PRAGMA mmap_size. Instrumented: 8 xFetch calls on the temp file, 0 successful
mappings, and all 3.28 MB came back through buffered xRead. Disabling the path entirely made no
measurable difference (48.3 vs 46.5 ms best-of-five), consistent with the map never having
happened. WHY the unix VFS declined was NOT established -- candidates are the
sqlite3GlobalConfig.mxMmap/szMmap clamp in unixFileControl, the zero-length map attempted at
extend time, or the DELETEONCLOSE locking style. This is also a caveat on every READ count above;
the WRITE byte counts are unaffected. Lesson 28's Next Steps item 1.

Handed forward: lesson 29 still gets WHERE_ONEPASS_DESIRED / WHERE_ONEPASS_MULTIROW /
WHERE_DUPLICATES_OK and sqlite3MultiWrite, and now ALSO gets statement journals as the one kind of
temporary file lesson 28 deliberately did not touch (tempfiles.html lists nine kinds; lesson 28
covered transient indices and the sorter's own temp files only). Lesson 30 still gets the whole
xBestIndex protocol and sqlite_stat4-as-a-table. Lesson 31 gets sqlite3StackAllocRawNN, the
loadStatTbl allocation cost, the unresolved sorter-mmap question above, and should note that
vdbesort.c's bulk-allocation mode (list.aMemory, doubled by sqlite3Realloc, links threaded by
OFFSET because the block moves) is a second large consumer of the allocator alongside the page
cache. Lesson 32 gets OptimizationEnabled(), the SQLITE_TESTCTRL_OPTIMIZATION bit table,
SQLITE_Stat4, and now SQLITE_TESTCTRL_SORTER_MMAP as another testctrl knob that changes measured
behaviour. Lesson 33 (VACUUM) should note that VACUUM does not recompute statistics AND that its
rebuild recreates every index, i.e. it runs the CREATE INDEX sorter path in section 9 once per
index.

NOT read this run, still unexamined: src/build.c's sqlite3DefaultRowEst (effect measured, function
not read -- carried over from lesson 27), sqlite3Stat4ProbeSetValue and the stat4 probe-record
machinery in vdbemem.c, whereRangeSkipScanEst, and the vdbeSorterCompareReal /
vdbeSorterCompareRealInt / vdbeSorterGetReal arm of the comparators (SORTER_TYPE_REAL was never
exercised by a measurement this run).
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
