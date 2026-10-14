<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 29
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
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
