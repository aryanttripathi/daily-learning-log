<!--
track-state
subject: SQLite
started: 2026-09-16
lessons-total: 33
lessons-done: 27
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
- [ ] 28 — Sorting and temporary storage: the VDBE sorter (vdbesort.c) and external merge sort

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

scope note 2026-10-10 (lesson 25): no lesson added, removed, reordered or split;
lessons-total stays 33. The ordering held exactly. Lesson 25 needed lesson 24's affinity
machinery as input and lesson 23's register model as vocabulary, and nothing else, and it
stopped at sqlite3BtreeInsert()/sqlite3BtreeDelete() without re-entering lessons 11-12.

Taken, and NOT to be re-taught:

(a) The register contract. regNewData holds the rowid at regNewData and column i at
regNewData+1+i; aRegIdx[i] holds index i's key record with its scratch range at
aRegIdx[i]+1..; and aRegIdx[nIdx] -- one past the last index -- holds the TABLE record.
aRegIdx[i]==0 means "this index does not change, skip it entirely", and isUpdate is
literally regOldData!=0 (insert.c:1930). Measured on a 4-column table with 3 indexes:
regNewData=1, columns 2-5, aRegIdx=[6,10,13,16]. pTab->pIndex is newest-index-first and
cursor numbers follow, with iIdxCur == iDataCur+1 for a rowid table.

(b) THE CENTRAL FINDING, and the one later lessons are most likely to trip over. The table
affinity string lives in one of two completely different operand slots, and a single u8
local (bAffinityDone, insert.c:1918) decides which. If the statement touches any index, or
the table has a CHECK constraint, sqlite3TableAffinity(v,pTab,regNewData+1) emits a
standalone OP_Affinity that converts the registers IN PLACE before any index key is copied
out of them (insert.c:2075, 2424). Otherwise sqlite3TableAffinity(v,pTab,0) RETROFITS the
string into the p4 of the OP_MakeRecord already emitted (insert.c:2734). Measured both ways
on the same three-column declaration: without an index, "MakeRecord 2 3 5 'DBE'" and no
Affinity opcode; with one index on b, "Affinity 2 3 0 'DBE'" and BOTH MakeRecords with an
empty p4. Lesson 26 and later should not assume OP_MakeRecord.p4 is where affinity lives.

(c) Index key records get NO affinity string at all (insert.c:2464 uses AddOp3, no p4),
because (b) already converted the registers. The key is built with OP_SCopy per ordinary
column and OP_IntCopy for the rowid. This sources lesson 23's observation that all its
OP_SCopy sites were on the INSERT/upsert path: the two emission sites are this key-build
loop and the upsert DO UPDATE arm. A six-statement census found 0 OP_SCopy on SELECT and
on DELETE, 1 on UPDATE, 2 on INSERT, 3 on upsert. That SUPPORTS but does not close
lesson 23's Next Steps item 2, which remains open as lesson 25's Next Steps item 3.

(d) The table record is encoded inside sqlite3GenerateConstraintChecks, not in
sqlite3CompleteInsertion, and the header comment dates the reason to 2019-05-07: building
it later would let constraint-time affinity conversion change the values the record is
built from, after the index keys had already been copied. "Generate constraint checks"
encodes the row; sqlite3CompleteInsertion only emits writes and flags.

(e) OPFLAG_APPEND is decided from SYNTAX, at compile time. appendFlag is set only when
pIpk->op==TK_NULL -- a literal NULL in the VALUES list -- or when the IPK column is omitted
(insert.c:1510, 1536). Measured OP_Insert.p5: 57 (0x39, with APPEND) for VALUES(NULL,'x')
and for INSERT INTO t(b) VALUES('x'); 49 (0x31, no APPEND) for VALUES(?,'x'), VALUES(7,'x')
and VALUES(1+1,'x'). The parameter form also costs three extra instructions per row
(NotNull/NewRowid/MustBeInt). Combined measured cost on 200k inserts: 1.24x. The two
effects were NOT separated -- that is lesson 25's Next Steps item 2 and would need a patched
build.

(f) The four flag words, all decodable from EXPLAIN with no debug build, against
sqliteInt.h:4109-4129. OP_Insert.p5: 57 = NCHANGE|LASTROWID|APPEND|USESEEKRESULT (plain
INSERT), 49 = same minus APPEND, 5 = ISUPDATE|NCHANGE (the UPDATE path, because update.c:1109
passes appendBias=0 and useSeekResult=0), 64 = ISNOOP alone (codeWithoutRowidPreupdate).
OP_IdxInsert.p5: 16 = USESEEKRESULT on INSERT, 17 = |NCHANGE for a WITHOUT ROWID PK index,
0 on UPDATE. OP_Delete.p2: 68 = ISNOOP|ISUPDATE (hook only, rowid unchanged), 4 = ISUPDATE
(real delete, rowid changed), 1 = NCHANGE, 0 on both REPLACE deletes. OP_Delete.p5: 2 =
SAVEPOSITION for ONEPASS_MULTI, 4 = AUXDELETE. OP_OpenWrite.p5 = 8 = FORDELETE on a DELETE.
Extended result codes read out of OP_Halt.p1: 1299 = CONSTRAINT_NOTNULL, 1555 =
CONSTRAINT_PRIMARYKEY, 2067 = CONSTRAINT_UNIQUE.

(g) OP_IdxInsert.p4 is pIdx->uniqNotNull ? nKeyCol : nColumn (insert.c:2843-2845), and this
is directly observable: a UNIQUE index on a NOT NULL column gives p4='1' where a UNIQUE
index on a nullable column and a non-unique index both give p4='2'. OP_NoConflict.p4 is
nKeyCol, never nColumn.

(h) Program size is EXACTLY linear in index count: +5 instructions per single-column index
(OpenWrite, SCopy, IntCopy, MakeRecord, IdxInsert), measured 15/21/26/31/36/41/46
instructions for 0..6 indexes. The first index costs 6 because it forces the OP_Affinity
from (b). NoConflict stays 0 for non-unique indexes -- a non-unique index is pure write
amplification with no validation cost. Measured run cost: 1.00x/1.53x/2.06x/2.50x/2.97x for
0..4 indexes on 20k inserts, about +0.53 us per index per row.

(i) sqlite3IndexAffinityStr / computeIndexAffStr (insert.c:75-114) CLAMPS every affinity to
[BLOB, NUMERIC]: "if( aff>SQLITE_AFF_NUMERIC) aff = SQLITE_AFF_NUMERIC;". INTEGER (0x44),
REAL (0x45) and FLEXNUM (0x46) all collapse to 'C', and so does the XN_ROWID arm which sets
SQLITE_AFF_INTEGER one line above the clamp. An index affinity string can contain only
'A','B','C'. Its own header comment (insert.c:59-73) is WRONG about this in three ways: it
promises 'D' for INTEGER and 'F' for REAL (and 'F' is FLEXNUM's character, not REAL's --
REAL is 'E'), and it claims "An extra 'D' is appended to the end of the string to cover the
rowid", which the clamp prevents. Measured two independent ways on 3.45.1: a WITHOUT ROWID
two-pass DELETE whose PK is (i INTEGER, r REAL, tx TEXT) emits "MakeRecord 1 3 16 'CCB'"
(delete.c:577-579), and an IN-list seek on an INTEGER, a REAL and a NUMERIC column each emit
Affinity p4='C'. This string NEVER reaches an INSERT -- the insert path's index keys have no
p4 at all. Lesson 26 inherits one loose end: for a four-column index with four constrained
terms the emitted opcode is "Affinity p1=1 p2=3 p4='CCB'", three characters for four
columns, and WHICH where.c function does that trimming was NOT traced.

(j) The delete path builds NO index key record. sqlite3GenerateRowIndexDelete calls
sqlite3GenerateIndexKey with regOut=0 and prefixOnly=1 (delete.c:925), and the OP_MakeRecord
inside that function is conditional on regOut (delete.c:1017-1019). OP_IdxDelete assembles
an UnpackedRecord on the C stack from loose registers instead. prefixOnly means a
UNIQUE NOT NULL index is sought on nKeyCol only. The pPrior/regPrior optimization skips
re-loading columns shared with the previous index, measured as exactly one OP_Column saved
for s(x) + s(x,y) versus s(x) + s(y,z) (3 vs 4), and it is defeated by a partial index per
ticket a9efb42811fa41ee (2019-11-02). sqlite3VdbeDeletePriorOpcode(v, OP_RealAffinity)
(delete.c:1008-1016) un-appends the REAL re-promotion that lesson 24 covered, because the
value is headed back into an index key where the compact integer form is stored.

(k) UPDATE is OP_Delete + OP_Insert and ALWAYS rewrites the whole row -- unchanged columns
are read with OP_Column and written straight back into the new record (measured). Only
indexes whose columns can change are opened at all (aRegIdx[i]==0; update.c:108-153 decides).
The OP_Delete is a no-op unless the rowid changes: p2=68 (ISNOOP|ISUPDATE) when the rowid is
unchanged, p2=4 (ISUPDATE) when it changes. One statement writes one b-tree cell; the other
writes two and deletes one.

(l) ONEPASS. Measured all three shapes: DELETE by rowid = ONEPASS_SINGLE (no loop, no FIFO);
DELETE by a non-unique index = ONEPASS_MULTI (SeekGE/IdxGT/Next with OP_Once-guarded cursor
opening inside the loop, OP_Delete.p5=SAVEPOSITION); a WITHOUT ROWID DELETE with a subquery
= ONEPASS_OFF with the full two-pass shape (OpenEphemeral + a MakeRecord/IdxInsert subroutine,
then reopened write cursors, Rewind, RowData, NotFound). The two-pass UPDATE
(UPDATE ... SET b=? WHERE b=?) uses OP_Insert into an ephemeral cursor as the FIFO push.
wcf withholds WHERE_ONEPASS_MULTIROW whenever bComplex (a subquery) is set. Lesson 26 OWNS
sqlite3WhereOkOnePass; lesson 25 read its answer and did not open it.

Version disagreements recorded, both real and both measured:

- OP_IdxDelete changed operands between 3.45.1 and trunk. 3.45.1 (vdbe-3.45.1 lines
  6487-6501): "Opcode: IdxDelete P1 P2 P3 * P5", synopsis key=r[P2@P3], P3 is the register
  count and P5 is the boolean controlling whether a missing entry raises
  SQLITE_CORRUPT_INDEX. Trunk (vdbe.c:6792-6865): P5 is the register count, P4 is an Index*,
  P3 may name a record to compare against and skip, and a missing entry goes through
  sqlite3VdbeFindIndexKey before erroring. Measurements in this lesson show the 3.45.1 form
  (p3=2, p5=1). Any later lesson citing OP_IdxDelete operands must say which version.

- OP_TypeCheck.p3. Trunk's sqlite3TableAffinity explicitly zeroes it when swapping the
  opcode into an existing MakeRecord slot (insert.c:191-196: "p3 = pPrev->p3; pPrev->p3 = 0;").
  Measured on 3.45.1, TypeCheck.p3 == 4, the same value MakeRecord.p3 carries. 3.45.1's
  insert.c was NOT fetched, so this is recorded as an unresolved disagreement, not a verdict.
  It is lesson 25's Next Steps item 1.

Build properties measured rather than read (useful to later lessons):
SQLITE_ENABLE_PREUPDATE_HOOK is ON in CPython 3.13's libsqlite3 3.45.1 -- the WITHOUT ROWID
insert emits the OP_Integer 0 / OP_Insert p5=OPFLAG_ISNOOP pair that only
codeWithoutRowidPreupdate (inside that #ifdef) produces. SQLITE_ENABLE_NULL_TRIM is OFF --
every measured OP_MakeRecord has p5==0, and a 4-column row with two trailing NULLs still
carries four serial-type bytes on disk (page 2 cell: 07 01 05 00 11 00 00 78 79).

Handed forward: lesson 26 gets sqlite3WhereOkOnePass, the where.c affinity-string trimming
from (i), and lesson 24's still-open tcol = CAST(icol AS TEXT) row. Lesson 28 gets
OP_SorterInsert, which appeared here only as OP_IdxInsert's sibling. Lesson 29 gets
sqlite3MultiWrite() and the OE_Abort/OE_Fail/OE_Rollback distinction that lesson 25's action
table names but does not implement. Lesson 30 gets OP_VUpdate and the virtual-table arms of
sqlite3Insert/DeleteFrom/Update, all of which lesson 25 skipped. Lesson 32
(integrity_check) gets sqlite3GenerateIndexKey, which it reuses to re-derive index keys
from table rows.

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
-->

## Completed Subjects

<!-- moved here when a track finishes -->
_None yet._
