<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 6
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 06 shipped 2026-10-25.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 07**.

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

- [x] 01 — The cluster on disk: `base/<dboid>/<relfilenode>`, the three forks, and 1 GB segment files
- [x] 02 — The page: `PageHeaderData`'s 24 bytes, `pd_lower`/`pd_upper`, the line pointer array, and special space
- [x] 03 — `HeapTupleHeaderData`: `t_xmin`, `t_xmax`, `t_ctid`, `t_infomask`/`t_infomask2`, and `t_hoff`
- [x] 04 — Attribute storage: MAXALIGN padding between attributes, 1-byte vs 4-byte varlena headers, `atthasmissing`, and TOAST
- [x] 05 — The Free Space Map: the FSM tree pages and how a page with room is found
- [x] 06 — The Visibility Map: `PD_ALL_VISIBLE`, all-frozen, and the index-only scan it enables
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
The SQLite track's lessons 31-33 notes were archived on that run; see TRACK-ARCHIVE.md, row
"31 - 33". The capstone document (2026/10/2026-10-19-sqlite-capstone/README.md) closes the track
and records which items were deliberately left unread. None of these carry forward into the
PostgreSQL track. Do not hand them to a PostgreSQL lesson.

-- LESSON 04 (2026-10-23) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-23-postgres-attribute-storage-varlena-toast/README.md (single README,
not split -- the two halves were held together by one thread, see below). Source commits read:
master @f52d4051 AND REL_16_STABLE (varatt.h, heaptoast.h) because master has DIVERGED HARD.
Trees so far: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051. Keep stating the commit.

*** THIS RUN HAD A CLUSTER. Same recipe as lesson 03, reproduced without trouble. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
data_checksums OFF, default_toast_compression = pglz, pageinspect 1.12. initdb as user 'pg' into
/var/tmp/pgdata --no-sync, pg_ctl -o '-p 5455 -k /var/tmp -c listen_addresses='. The two traps
from lesson 03 are real and still apply (not root; not under the scratchpad path).

THE UNIFYING THESIS, worth reusing: heap_compute_data_size() is BOTH the layout oracle and the
TOAST loop's measuring stick. It decides where each attribute lands, and every one of the four
TOAST passes is a `while (heap_compute_data_size(...) > maxDataLen)`. Layout and TOAST are one
mechanism consulted from two directions. Later lessons may cite this framing.

LESSON 02'S TWO DEBTS ARE CLOSED, both MEASURED:
- Heap page bytes 12-19 = "1c00 e01f 0020 0420" => pd_lower 28, pd_upper 8160, pd_special 8192,
  pd_pagesize_version 0x2004. BYTES 16-19 = "00 20 04 20" EXACTLY as lesson 02 predicted. nbtree
  leaf = "f0 1f 04 20". Both fingerprints are now observations, not arithmetic. Use freely.
- MaxHeapTupleSize 8160: one `text STORAGE PLAIN` column, repeat('x',n). n<=8132 OK (lp_len 8156
  at n=8128 = 24+4+8128); n=8133..8140 ALL fail with the SAME text "row is too big: size 8168,
  maximum size 8160", because the reported size is MAXALIGN(28+n) and is constant across those
  eight values. 8160 and the MAXALIGN-8 granularity are now measured.

TOOK, and must not be re-taught:
- The data area starts at MAXALIGN(t_hoff), and fill_val() aligns the RAW POINTER not an offset
  ("XXX we use the att_nominal_alignby macro on the pointer value itself ... This is a bit of a
  hack"). That hack is sound ONLY because t_hoff is MAXALIGNed. Consequence taught: the null
  bitmap can never disturb attribute alignment, it only moves the base by a multiple of 8.
- THE FOUR (really six) ALIGNMENT MACROS AND WHY: two axes -- nominal vs conditional, datum vs
  pointer. att_align_nominal / att_nominal_alignby (unconditional), att_align_datum /
  att_datum_alignby (building, skip align if attlen==-1 && VARATT_IS_SHORT),
  att_align_pointer / att_pointer_alignby (walking, skip align if attlen==-1 &&
  VARATT_NOT_PAD_BYTE). The *_alignby variants are master's CompactAttribute fast path.
  att_align_nominal's third use case carries "XXX this should be revisited, probably" about
  unconditionally aligning varlenas inside arrays and multiranges.
- typalign_to_alignby()'s four cases (TYPALIGN_CHAR->1, SHORT->ALIGNOF_SHORT, INT->ALIGNOF_INT,
  DOUBLE->ALIGNOF_DOUBLE) WITH the rationale quoted: codes not real values are stored "mainly so
  that initial catalog contents can be machine-independent".
- att_addlength_pointer's three cases: attlen>0 -> attlen; -1 -> VARSIZE_ANY; -2 -> strlen+1, with
  AssertMacro((attlen) == -2) proving there is no fourth.
- COLUMN ORDER, MEASURED: (int2,int8,int2,int8,int2,int8) lp_len 72 vs (int8,int8,int8,int2,int2,
  int2) lp_len 54 -- 18 pad bytes in a 54-byte payload. Full offset reconstruction is in the
  document. ESTABLISHED AND REUSABLE; do not re-derive.
- THE VARLENA BIT LAYOUT COMMENT from varatt.h, quoted whole (both endian tables). Three things
  extracted and reusable: (1) the flag bits are in the PHYSICALLY FIRST byte in both endiannesses,
  which is why a one-byte peek is portable; (2) a 1-byte length word can never be zero because the
  length includes itself, hence 0x00 is unambiguously pad -- "(We now *require* pad bytes to be
  filled with zero!)"; (3) the length ALWAYS includes the length word.
- Header sizes, all offsetof not literals: VARHDRSZ 4 (in c.h, NOT varatt.h), VARHDRSZ_SHORT 1,
  VARHDRSZ_EXTERNAL 2, VARHDRSZ_COMPRESSED 8, VARATT_SHORT_MAX 0x7F = 127.
- Every VARATT_IS_* reads through `varattrib_1b` (first member a uint8) deliberately, so the
  compiler cannot assume alignment. VARATT_NOT_PAD_BYTE is BYTE-IDENTICAL in both endian arms.
- THE 126-BYTE BOUNDARY, derived and MEASURED. VARATT_CAN_MAKE_SHORT requires VARATT_IS_4B_U (so
  already-short and compressed datums are excluded) and VARSIZE-4+1 <= 127 => data <= 126.
  pg_column_size: 124->125, 125->126, 126->127, 127->131, 128->132.
  Row-width cost of crossing it DEPENDS ON THE OFFSET: on (int,text) lp_len 155->159 (+4); on
  (int2,text) 153->159 (+6 -- 3 for the header plus 2 pad bytes that did not exist).
  CENTRAL FRAMING TAUGHT: a varlena's alignment is a property of its VALUE, not its column.
- THE PAD-BYTE PEEK, fully taught (lesson 03's handover named this lesson's owner of it) with
  att_align_pointer's comment quoted whole, plus the two-line proof: zero => cannot be a 1-byte
  header => pad or already-aligned 4B word => aligning is safe; non-zero => cannot be pad =>
  1-byte header or already-aligned 4B word => not aligning is safe. One comparison resolves four
  possibilities into two safe actions. heaptuple.c's file-header comment quoted for the
  "always zeroed / now required" history.
- THE MEASURED PEEK PAIR, the best measurement of the lesson. pg4_peek3(a int2, b text, c text),
  both rows a=1 b='AB' so byte-identical through offset 28:
    row1 c='CD'           -> t_data \x0100074142074344, lp_len 32. Byte at 29 = 0x07 -> NO align.
    row2 c=repeat('Q',127)-> t_data \x01000741420000000c020000..., lp_len 163. Byte at 29 = 0x00
                             -> align to 32, THREE pad bytes, then 0x0000020c>>2 = 131 = 4+127.
    lp_len 163 = 24+2+3+3+131 exactly.
  A later lesson may cite this pair; it does not need re-deriving.
- STORAGE PLAIN DOES NOT STOP HEADER PACKING -- MEASURED, and it is the ATT_IS_PACKABLE comment's
  own example ("This can be verified with pageinspect"). text column, 'AB', attstorage='p':
  lp_len 27 and t_data \x074142 (1-byte header) for BOTH create-time `STORAGE PLAIN` and
  alter-time `SET STORAGE PLAIN`. A 4-byte header would be lp_len 30. Cause: the relation's
  TupleDesc comes through TupleDescInitEntry(), which copies the TYPE's typstorage
  (TYPSTORAGE_EXTENDED), not the COLUMN's attstorage. The three predicates were quoted:
  ATT_IS_PACKABLE, VARLENA_ATT_IS_PACKABLE, COMPACT_ATTR_IS_PACKABLE.
- attcacheoff AND THE CLIFF, from TupleDescFinalize() (master: computed ONCE there, NOT lazily in
  nocachegetattr as in older trees). Three break conditions = three cache killers: attlen <= 0
  (any varlena or cstring -- EVERYTHING AFTER THE FIRST VARLENA IS UNCACHED, not just that
  column), attgenerated == ATTRIBUTE_GENERATED_VIRTUAL, off > PG_INT16_MAX. Inside the cached
  prefix it uses att_nominal_alignby, correct precisely because the prefix has no varlenas.
  CompactAttribute taught as the companion: 8 bytes, with "Any code changes which enlarge this
  struct should be considered very carefully", holding attcacheoff/attlen/attbyval/attalignby
  (a BYTE COUNT, not the attalign char code) plus four :1 flags incl. attispackable and
  atthasmissing. Also named: firstNonCachedOffsetAttr, firstNonGuaranteedAttr.
- atthasmissing, THE THIRD NULL ENCODING (lesson 03 named and deferred it). heap_attisnull()'s
  first branch quoted: attnum > natts AND atthasmissing => return FALSE, i.e. a column with no
  bytes at all reports NOT NULL. getmissingattr() taught: AttrMissing {am_present, am_value} off
  tupleDesc->constr->missing + (attnum-1); by-value returned directly ("/* no  need to cache
  by-value attributes */" -- double space is in the source); by-reference interned in a
  BACKEND-LIFETIME hash table keyed {len,value} and hashed over the BYTES, so the returned pointer
  is the only value heap_deform_tuple can produce that does not point into a tuple.
  THE asymmetry FLAGGED, not resolved: heap_attisnull gates on atthasmissing, getmissingattr on
  am_present. Lesson 04's Next Step 5 asks whether they can ever disagree.
- MEASURED, atthasmissing: `add column b text default 'hello'` -> filenode UNCHANGED, natts 1,
  lp_len 28, t_data \x01000000 (four bytes, the int, nothing else), t_infomask 2048 =
  HEAP_XMAX_INVALID only with HEAP_HASNULL CLEAR, atthasmissing t, attmissingval {hello}, and
  SELECT returns 'hello'. Contrast cases also measured: a VOLATILE default
  (clock_timestamp()::text) CHANGED the filenode and reset atthasmissing to FALSE FOR EVERY
  COLUMN (the rewrite undoes the optimisation); `add column b text` with no default leaves
  atthasmissing false, attmissingval null, natts 1 -- i.e. lesson 03's encoding #2 unchanged.
- ALL FOUR TOAST CONSTANTS DERIVED IN CLOSED FORM FROM MaximumBytesPerTuple(n) =
  MAXALIGN_DOWN((BLCKSZ - MAXALIGN(SizeOfPageHeaderData + n*sizeof(ItemIdData))) / n), using only
  lesson 01-03 constants, AND THEN MEASURED:
    TOAST_TUPLES_PER_PAGE 4;  TOAST_TUPLE_THRESHOLD = TOAST_TUPLE_TARGET = 2032
    TOAST_TUPLES_PER_PAGE_MAIN 1; TOAST_TUPLE_TARGET_MAIN = 8160 == MaxHeapTupleSize (NOT a
      coincidence: "the largest tuple that will fit on a heap page" describes both)
    EXTERN_TUPLES_PER_PAGE 4; EXTERN_TUPLE_MAX_SIZE 2032
    TOAST_MAX_CHUNK_SIZE = 2032 - MAXALIGN(23) - sizeof(Oid) - sizeof(int32) - VARHDRSZ
                         = 2032 - 24 - 4 - 4 - 4 = 1996
  MEASURED: the threshold boundary is t_len > 2032 STRICTLY -- datalen 2004 (t_len 2032) stays
  inline with 0 chunks, datalen 2005 (t_len 2033) produces 2 chunks. Chunk lengths for
  repeat('y',10000): 1996 x5 + 20 = 10000. Chunk TUPLES: lp_len 2032 each, t_hoff 24, exactly
  FOUR on page 0 -- i.e. EXTERN_TUPLE_MAX_SIZE and EXTERN_TUPLES_PER_PAGE both observed.
  Chunk tuple reconstruction: 24 + 4 (chunk_id Oid) + 4 (chunk_seq int4) + 4 (varlena hdr) + 1996.
  The header's two warnings quoted: the needs_toast_table()/LOBLKSIZE XXX, and "NB: Changing
  TOAST_MAX_CHUNK_SIZE requires an initdb."
- THE FOUR-PASS ALGORITHM, with heap_toast_insert_or_update()'s own enumeration comment quoted
  verbatim, plus the hoff/maxDataLen computation ("this should match heap_form_tuple()") and
  RelationGetToastTupleTarget(rel, TOAST_TUPLE_TARGET) - hoff. Taught: each pass is a while loop
  re-calling heap_compute_data_size; selection is STRICTLY GREEDY biggest-first, one per
  iteration; per-pass filters are EXTENDED (compress, pass 1), EXTENDED|EXTERNAL (externalise,
  pass 2), MAIN (compress, pass 3), MAIN (externalise, pass 4); pass 1's "one long field and
  several short ones" shortcut quoted; and PASS 4 REASSIGNS maxDataLen = TOAST_TUPLE_TARGET_MAIN -
  hoff, i.e. 2008 bytes of budget for passes 1-3 and 8136 for pass 4. That single reassignment IS
  what MAIN means.
- toast_tuple_find_biggest_attribute()'s FLOOR: biggest_size STARTS at MAXALIGN(pointer size) =
  MAXALIGN(18) = 24, not zero. An attribute under 24 bytes is NEVER a candidate, because a
  pointer would not shrink the tuple.
- varatt_external FULLY DECODED (REL_16_STABLE spelling; see divergence). 16 bytes no padding
  "because we sometimes compare these pointers using memcmp", stored UNALIGNED so callers must
  memcpy into a local first. Datum = VARHDRSZ_EXTERNAL(2) + 16 = 18. VARTAG_ONDISK = 18 AND THE
  TAG EQUALLING THE SIZE IS ARCHAEOLOGY, not arithmetic -- the enum comment says the "peculiar
  value" comes from "a previous notion that the tag field was the pointer datum's length".
  VARLENA_EXTSIZE_BITS 30 / MASK 0x3FFFFFFF is where the documented 1 GB limit comes from, with
  2 bits left for ToastCompressionId (PGLZ 0, LZ4 1, INVALID 2) and the header's blunt "we can
  never have more than 4 values in this enum".
  "IS IT COMPRESSED" IS NOT A FLAG -- VARATT_EXTERNAL_IS_COMPRESSED is a SIZE COMPARISON,
  extsize < rawsize - VARHDRSZ, with the comment quoted. So the method bits are meaningless when
  the sizes are equal.
- MEASURED, the 18-byte pointer byte by byte. repeat('y',10000) in a STORAGE EXTERNAL column:
  lp_len 42 = 24 + 18 (no alignment, 1-byte header class); t_infomask 2054 = 0x0806 =
  XMAX_INVALID|HASEXTERNAL|HASVARWIDTH -- lesson 03 catalogued HEAP_HASEXTERNAL 0x0004 and
  deferred it, now OBSERVED. t_data \x01121427000010270000814000007f400000 =>
  01 (va_header, the LE "00000001" TOAST-pointer pattern) | 12 (va_tag = 18 = VARTAG_ONDISK) |
  14270000 (va_rawsize 10004 = 10000+4) | 10270000 (va_extinfo 10000; NOT compressed because
  10000 == 10004-4) | 81400000 (va_valueid 16513) | 7f400000 (va_toastrelid 16511).
  BOTH OIDS CROSS-CHECKED against pg_class.reltoastrelid and the toast table's chunk_id this run.
  TRAP MEASURED: pg_column_size() on that column returns 10000, NOT 18 -- it reports the external
  datum's size, not the in-tuple footprint. For footprint, read lp_len.
- THE FOUR STORAGE STRATEGIES, MEASURED SIDE BY SIDE on one table, one value repeat('ab',2500):
  ext (e) pg_column_size 5000, externalised uncompressed, 3 chunks 1996/1996/1008;
  extd (x) 66, COMPRESSED THEN EXTERNALISED, 1 chunk of 66;
  main (m) 70, compressed INLINE;
  plainc (p) 5004, untouched inline with a 4-byte header.
  THE 66-vs-70 EXPLANATION, established and reusable: the pglz payload is 62 bytes in both.
  Inline a compressed datum carries VARHDRSZ_COMPRESSED = 8 (length word + va_tcinfo): 8+62 = 70.
  Externally the length word is redundant (the pointer's va_rawsize/va_extinfo carry it), so the
  chunk stores va_tcinfo + payload: 4+62 = 66.
  THE WHOLE TUPLE RECONSTRUCTED BYTE-EXACTLY: measured lp_len 5136 = 24 + 18 (ext ptr @24) +
  18 (extd ptr @42) + 70 (main compressed @60) + 2 PAD (@130-131) + 5004 (plainc @132).
  The 2 pad bytes were verified by substring: tuple offsets 126-137 = "ff 0f 02 42 00 00 30 4e
  00 00 61 62", so 130-131 = 00 00 and the word at 132 is 0x00004e30>>2 = 5004 with low bits 00.
  main's word at 60 is 0x0000011a, low bits 10 => compressed, >>2 = 70; its va_tcinfo = 0x1388 =
  5000 = decompressed size, method 0; then the pglz stream "fc 61 62 61 0f 02 ff ...".
- PASS 4 MEASURED, and it is the best MAIN demonstration available: two MAIN columns of 5120
  INCOMPRESSIBLE bytes each (concatenated distinct md5s) -> lp_len 5168, HASEXTERNAL set, 3 chunks
  totalling 5120, pg_column_size 5120 / 5124 -> EXACTLY ONE of the two was externalised, because
  after that the data size 18+2+5124 = 5144 <= 8136 and pass 4 stopped. 5168 = 24+18+2+5124.
  THE SAME DATA with default EXTENDED: lp_len 60 = 24+18+18, 6 chunks, 10240 bytes.
  So: same bytes, one attstorage apart => a 5168-byte row with 3 chunks, or a 60-byte row with 6.
- toast_tuple_target IS A TARGET, NOT A THRESHOLD -- MEASURED, and the single best operational
  trap in the lesson. WITH (toast_tuple_target = 512): datalen 2004 stays at lp_len 2032, FOUR
  TIMES the configured target, because the gate into the toaster is `t_len >
  TOAST_TUPLE_THRESHOLD` (a compile-time 2032) and the reloption only reaches maxDataLen INSIDE
  the function. datalen 2005 drops to lp_len 42. Lowering the target below 2032 does NOTHING;
  raising it above 2032 works as expected.

HONEST NON-RESULT, recorded deliberately: the attcacheoff cliff was NOT reproduced as a
measurable cost. Two 400k-row tables, ten int columns and one text, text first vs text last,
warm cache, `select sum(c10)`: late 23.9/18.0/18.3 ms, early 22.6/21.4/19.4 ms -- overlapping
distributions. At ten columns the uncached walk is nine macro steps per row and is lost in
per-row executor overhead. The lesson says so explicitly and tells the reader NOT to cite the
cliff as a measured cost. Lesson 04's Next Step 2 specifies the 200-column redo with
EXPLAIN (ANALYZE, TIMING OFF). A later lesson that wants this number must earn it.

VERSION DIVERGENCE, the largest this track has hit -- a full table is in the lesson. Summary:
master @f52d4051 renamed varatt_external -> varatt_external_oid and added varatt_external_oid8
(20 bytes, va_valueid_lo/_hi) for 64-bit TOAST value IDs; VARTAG_ONDISK -> VARTAG_ONDISK_OID (18)
with a new VARTAG_ONDISK_OID8 (4); VARATT_EXTERNAL_GET_EXTSIZE/GET_COMPRESS_METHOD/IS_COMPRESSED
-> VARATT_EXTINFO_* taking a bare uint32; TOAST_MAX_CHUNK_SIZE is no longer a scalar but is
parameterised by the TOAST table's chunk-id type (TOAST_OID_MAX_CHUNK_SIZE /
TOAST_OID8_MAX_CHUNK_SIZE); every externally visible accessor became a static inline function
("All of these were originally macros, accounting for the upper-case naming"); the `slow` flag in
heap_deform_tuple/nocachegetattr is GONE, replaced by firstNonCachedOffsetAttr and
first_null_attr(); attcacheoff moved from lazy fill-in to TupleDescFinalize(); TOAST_INDEX_HACK
lives in access/common/indextuple.c, not heaptoast.*. IDENTICAL in both trees: the bit layouts,
VARHDRSZ_*, VARATT_SHORT_MAX, VARLENA_EXTSIZE_BITS, the endian internals, the four-pass algorithm
and the MaximumBytesPerTuple derivation. EVERY MEASURED NUMBER IS PG 16.

NO SYLLABUS REVISION THIS RUN. Lesson 03 had already retitled 04 to drop the null bitmap and add
atthasmissing; that title was taught as written, in full, and nothing was left over.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg4_fp, pg4_ceil, pg4_bad, pg4_good, pg4_var, pg4_var2, pg4_peek, pg4_peek2,
pg4_peek3, pg4_plain, pg4_plain2, pg4_miss, pg4_miss2, pg4_toast, pg4_t2, pg4_strat, pg4_main,
pg4_main2, pg4_ext2, pg4_ttt, pg4_late, pg4_early. Keep the pgNN_ prefix convention.

-- LESSON 05 (2026-10-24) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-24-postgres-free-space-map/README.md (single README, not split -- one
mechanism, no sub-topic deserved its own file). Source commit read: postgres/postgres@126976a8
(master) PLUS REL_16_STABLE for src/include/storage/freespace.h. Trees so far: 01 @ddd468ca,
02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051, 05 @126976a8. Keep stating the commit.
Files read IN FULL: storage/freespace/README, fsmpage.c, freespace.c, storage/fsm_internals.h,
storage/freespace.h.

*** THIS RUN HAD A CLUSTER, and EVERY number in the lesson is measured. Same recipe as 03/04. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
data_checksums off, pageinspect 1.12, pg_freespacemap 1.2, pgstattuple 1.5. The two traps still
apply (initdb not as root; data dir not under the scratchpad path -- /var/tmp works).

*** CORRECTION TO LESSON 04'S HANDOVER, important: HEAP_FSM_CREATION_THRESHOLD AND FSM_LOCAL_MAP
DO NOT EXIST. *** Not in master @126976a8, not in REL_16_STABLE (freespace.h is byte-identical
between the two branches), and a repo-wide code search returns zero hits. The feature was
b0eaa4c51b "Avoid creation of the free space map for small heap relations" and it was REVERTED
WHOLESALE by 7db0cde6b58eef2ba0c70437324cbc7622230320 on 2019-05-07 along with six follow-ups
(06c8a5090e, 13e8643bfc, 6f918159a9, 9c32e4c350, 29d108cdec, 08ecdfe7e5). The revert message says
the process-local map "was reset whenever a block with enough free space was found" and that a
per-relation relcache map would be better; that redesign was never landed. SO NO RELEASED
POSTGRESQL HAS A SMALL-TABLE FSM EXEMPTION. Measured confirmation: the FSM fork appears at TWO
heap pages, not five. Do not propagate the 4-page threshold claim.

TOOK, and must not be re-taught:
- THE CATEGORY. FSM_CATEGORIES 256, FSM_CAT_STEP = BLCKSZ/256 = 32, MaxFSMRequestSize =
  MaxHeapTupleSize = 8160. fsm_space_avail_to_cat FLOORS (and clamps to 254 below 8160, returning
  255 only at >= 8160); fsm_space_needed_to_cat CEILS ((needed+31)/32) and returns 1 for needed==0;
  fsm_space_cat_to_avail(255) = 8160 exactly, else cat*32. The reason 255 is reserved is quoted
  from the header: otherwise a page with exactly MaxFSMRequestSize free would not satisfy a
  request for MaxFSMRequestSize. needed > 8160 is an elog(ERROR) "invalid FSM request size", and
  it is UNREACHABLE from SQL because lesson 04's "row is too big" fires first.
- A STALE COMMENT, flagged and resolved by measurement: freespace.c's worked example says
  "MaxFSMRequestSize is 8164 bytes" and tabulates 8164-8192 as category 255. The code says 8160.
  MEASURED: pg_freespace on the emptiest possible heap page returns 8160. The comment's example is
  4 bytes wrong (8164 is lesson 02's PageGetFreeSpace on a fresh page). Lesson 05's Next Step 6
  asks whether it was ever true -- a git log question, not yet answered.
- THE ON-PAGE STRUCTURE, derived and then MEASURED. FSMPageData { int fp_next_slot; uint8
  fp_nodes[] }, offsetof(fp_nodes) = 4. NodesPerPage = 8192 - MAXALIGN(24) - 4 = 8164.
  NonLeafNodesPerPage = BLCKSZ/2 - 1 = 4095 = 2^12 - 1. LeafNodesPerPage = SlotsPerFSMPage =
  8164 - 4095 = 4069. The non-leaf count is a round power of two minus one and the leaf count is
  the RAGGED REMAINDER -- which is why SlotsPerFSMPage is not a power of two, why 4096-4069 = 27
  leaf positions do not exist on the page, and why the README says "a few right-most leaf nodes
  are missing". One bottom FSM page covers 4069 heap blocks = 31.79 MiB of heap.
- BYTE OFFSETS, ESTABLISHED AND MEASURED, reusable without re-deriving: PageGetContents = page+24,
  so fp_next_slot is at page offset 24 (int32 LE), fp_nodes[0] (the page root) at offset 28, and
  LEAF SLOT k AT OFFSET 4123 + k. Node 8163 lands on byte 8191 -- the array exactly fills the page.
  MEASURED: get_byte(...,28) = get_byte(...,4123) = 12 on a page whose pg_freespace avail was 384
  = 12*32; bytes 24..31 = "00000000 0c 0c 00 0c" decoded field by field against fsm_page_contents.
- THE TREE OF PAGES. FSM_TREE_DEPTH = ((SlotsPerFSMPage >= 1626) ? 3 : 4) = 3; FSM_ROOT_LEVEL 2,
  FSM_BOTTOM_LEVEL 0; constant height by design. 4069^3 = 67,369,460,509 >> 2^32-1. Logical
  addressing {level, logpageno} + slot; fsm_get_location = heapblk/4069, heapblk%4069;
  fsm_get_parent/fsm_get_child divide and multiply by SlotsPerFSMPage.
- fsm_logical_to_physical FULLY EVALUATED AND MEASURED. root {2,0} -> block 0; {1,0} -> 1;
  {0,0} -> 2; {0,1} -> 3; {0,4068} -> 4070; {1,1} -> 4071; {0,4069} -> 4072. THE FORK IS NOT
  LEVEL-MAJOR: upper-level pages are interleaved into the stream where they become necessary.
  CONSEQUENCE, MEASURED: the minimum FSM fork is THREE BLOCKS = 24576 bytes, because the bottom
  page cannot be block 0. Observed: 1 heap page -> fsm 0 bytes; 2..4069 heap pages -> 24576;
  4071 heap pages -> 32768, and the level-1 page (block 1) gains its second populated leaf
  (node 4096). SlotsPerFSMPage = 4069 is now an OBSERVATION.
- fsm_set_avail: bubble up via parentof, stop as soon as a parent's recomputed max is unchanged;
  the no-op fast path has TWO conditions (oldvalue == value AND value <= fp_nodes[0]) because an
  equal leaf under a too-low root must still proceed so the trailing sanity check fires; and that
  check (value > fp_nodes[0] -> fsm_rebuild_page) is self-correcting measure #1.
- fsm_search_avail: the EXPANDING SEARCH TRIANGLE. Root byte < minvalue -> return -1 (one unlocked
  byte answers "nothing anywhere"). Start at fp_next_slot, ALWAYS clamped. Loop
  parentof(rightneighbor(nodeno)); rightneighbor detects stepping onto a level's leftmost node
  (2^level - 1) with ((x+1) & x) == 0 and corrects with parentof. Triangle doubles each step =>
  log2(N). Result is THE FIRST SUITABLE SLOT AT OR RIGHT OF fp_next_slot WITH WRAPAROUND, not the
  leftmost. Descent prefers the left child; if neither child qualifies the page is TORN ->
  DEBUG1 "fixing corrupt FSM block %u, relation %u/%u/%u", upgrade to exclusive, fsm_rebuild_page,
  MarkBufferDirtyHint, goto restart (self-correcting measure #2).
- fp_next_slot IS WRITTEN UNDER A SHARED LOCK, deliberately. Typed int not uint16 because "int is
  more likely to be atomically fetchable/storable"; README: "fp_next_slot is just a hint and we can
  easily reset it if it gets corrupted". MEASURED: after a search returning slot 0 at the bottom
  level, bytes 24..27 = 01000000 -> fp_next_slot = 1, i.e. advancenext. fsm_search passes
  advancenext = (addr.level == FSM_BOTTOM_LEVEL), so only leaf pages advance.
  NOTE for master: this is now gated on BufferBeginSetHintBits()/BufferFinishSetHintBits() in
  @126976a8, which PG 16 does not have. Flagged in the lesson's divergence note.
- NOT WAL-LOGGED, and master's README argues the nuance HARD -- four new paragraphs. Writes use
  MarkBufferDirtyHint(), reads use RBM_ZERO_ON_ERROR, so a bad FSM page becomes "nothing free",
  which costs space not correctness. BUT the README insists RelationCopyStorage() (ALTER TABLE SET
  TABLESPACE), RelationCopyStorageUsingBuffer() (CREATE DATABASE STRATEGY = wal_log), base backups
  and pg_checksums ALL verify checksums on *_fsm files, so the hint-bit FPIs are load-bearing and
  must not be downgraded to MarkBufferDirty(). Quote it if a later lesson claims the FSM is
  simply unchecked. (Lesson 24/25 may want this when FPIs come up.)
- fsm_does_block_exist: cached smgr_cached_nblocks[MAIN_FORKNUM] fast path, else
  RelationGetNumberOfBlocks -- the comment admits the lseek() cost and says the alternative "would
  cause FSM to set zero space available for blocks that main fork extension just recorded". A hit
  past the end zeroes the slot and RESTARTS FROM THE ROOT, with the comment calling advancenext
  "pessimal for this rare scenario". fsm_search has an emergency valve: restarts > 10000 ->
  InvalidBlockNumber, "Looping indefinitely is nevertheless scary".
- AN FSM PAGE IS AN "EMPTY" PAGE TO bufpage.c -- MEASURED. PageInit(page, BLCKSZ, 0): pd_lower 24
  forever, pd_upper = pd_special = 8192, so PageIsEmpty() is ALWAYS true, PageGetExactFreeSpace()
  is ALWAYS 8168, and lesson 02's four free-space functions are meaningless on this fork. The whole
  payload lives in lesson 02's HOLE, which is why PageInit's full-page MemSet and the checksum's
  coverage of the hole matter here.
- THE FINGERPRINT DOES NOT IDENTIFY THE FORK -- MEASURED, and it corrects a tempting misreading of
  lessons 02/04. FSM page bytes 12-19 = "1800 0020 0020 0420"; heap page = "2800 c001 0020 0420".
  BYTES 16-19 ARE "00 20 04 20" FOR BOTH. The fingerprint identifies the SPECIAL-SPACE SIZE (zero),
  which heap, sequence, TOAST and FSM pages all share. State it that way from now on.
- MEASURED, the FSM-vs-reality chain on an empty heap page: exact hole 8164 -> PageGetHeapFreeSpace
  / PageGetFreeSpace 8160 -> cat 255 -> reported 8160. Every step cites an earlier lesson's number.
  pd_lower was 28 (one LP_UNUSED pointer survived PageTruncateLinePointerArray).
- MEASURED, rounding loss: a page with exact hole 2080 / PageGetFreeSpace 2076 is recorded as 2048.
  28 bytes invisible; maximum possible loss 31 bytes per block.
- *** THE BEST MEASUREMENT OF THE LESSON, and the one a later lesson should cite: THE UPPER LEVELS
  ARE STALE UNTIL VACUUM. *** fsm_set_and_search bubbles up WITHIN ONE PAGE only; nothing
  propagates a bottom page's root into its parent's leaf except FreeSpaceMapVacuum[Range](), which
  VACUUM calls. RecordPageWithFreeSpace's own comment says so. MEASURED on an 8-page table built
  by plain inserts: leaves all 384, ROOT PAGE AND LEVEL-1 PAGE ENTIRELY ZERO (fsm_page_contents
  printed only "fp_next_slot: 0"). After VACUUM both upper pages populated. THEN, with each step in
  a SEPARATE psql process so no backend reused another's RelationGetTargetBlock relcache hint:
  the same tiny INSERT landed at ctid (2,5) before VACUUM (FSM search failed at the root ->
  RelationGetBufferForTuple's LAST-BLOCK FALLBACK) and at (0,5) after VACUUM (a real FSM hit).
  Identical statement, identical table, different target block, decided by ONE BYTE at page 0
  offset 28. The last-block fallback is why this failure mode is usually invisible.
- *** THE 32-BYTE CEILING TRAP, MEASURED TO THE BYTE. *** pg5_cons: block 0 left with
  PageGetFreeSpace = 8136 (one 24-byte tuple on an otherwise empty page) -> avail_to_cat floors to
  254 -> the category promises 8128. A row whose MAXALIGNed footprint is EXACTLY 8136 asks
  needed_to_cat = ceil(8136/32) = 255 > 254, so block 0 is NOT OFFERED; the last-block fallback
  (block 2, 32 bytes) had no room; THE RELATION EXTENDED TO BLOCK 3 and the row landed at (3,1).
  8 bytes of rounding cost one 8 KB page. Generalisation taught: a block recorded at category C is
  never offered to a request above 32*C, so any block whose usable free space is not an exact
  multiple of 32 has a dead band of up to 31 bytes at the top of its range.
- MaxHeapTuplesPerPage = 291, FIRST MEASURED HERE (lesson 02 derived it). 291 single-NULL-column
  tuples of lp_len 24 fit on page 0; the 292nd went to (1,1); pd_lower 1188 = 24 + 291*4, pd_upper
  1208, exact hole 20 -- the LINE POINTER ARRAY is the binding constraint, not the tuple bytes.
- HONEST NON-RESULT, recorded deliberately: the predicted "PageGetHeapFreeSpace returns 0 at the
  291 ceiling" trap DID NOT FIRE. Delete 290 of 291 and vacuum: with 290 LP_UNUSED pointers present
  PD_HAS_FREE_LINES is set (page flags 5 = PD_HAS_FREE_LINES|PD_ALL_VISIBLE), the function finds a
  reusable pointer and returns 6976, not 0. DO NOT cite the 291 ceiling as a cause of FSM
  under-reporting. What it DOES cost, measured: keeping the LAST row leaves all 291 line pointers
  (pd_lower 1188, exact hole 6980, FSM 6976); keeping the FIRST row truncates the array to 1
  (pd_lower 28, exact hole 8140, FSM 8128). SAME TABLE, SAME ONE SURVIVOR, 1152 BYTES OF DIFFERENCE
  decided only by WHERE ON THE PAGE the survivor sits, because PageTruncateLinePointerArray can
  only reclaim a trailing contiguous unused run (lesson 02).
- Also measured: a ONE-PAGE table has NO FSM fork (0 bytes) and pg_freespace reports 0 for its only
  block, because fsm_readbuf(..., extend=false) returns InvalidBuffer and GetRecordedFreeSpace
  returns 0.
- Index FSM semantics NAMED AND NOT TAKEN: for indexes the FSM tracks entirely-unused pages rather
  than free space within pages, so the byte is a boolean; indexfsm.c is the separate wrapper.
  Lesson 05's Next Step 5 asks a later reader to settle which AMs use it. Lessons 28-30 may want it.
- The README's TODO quoted and left open: "fastroot to avoid traversing upper nodes with just 1
  child" and "use a different system for tables that fit into one FSM page" -- the second is
  exactly what the reverted b0eaa4c51b tried to do.

VERSION DIVERGENCE: smaller than lesson 04's. storage/freespace.h is BYTE-IDENTICAL between master
@126976a8 and REL_16_STABLE. The differences found: (1) master's fsm_search_avail writes
fp_next_slot inside `if (exclusive_lock_held || BufferBeginSetHintBits(buf))` with a matching
BufferFinishSetHintBits(buf, false, false) -- that hint-bit API does not exist in 16, which simply
writes the field; (2) master's README carries four paragraphs on RBM_ZERO_ON_ERROR vs external
checksum verifiers that older trees do not; (3) FreeSpaceMapPrepareTruncateRel's
log_newpage_buffer() path carries an "XXX consider XLogSaveBufferForHint()" note. IDENTICAL in both:
every constant, every conversion function, the addressing arithmetic, the tree depth rule, and the
on-page layout. EVERY MEASURED NUMBER IS PG 16.

NO SYLLABUS REVISION THIS RUN. Lesson 05's title was taught exactly as written and nothing was left
over. The only correction made was to lesson 04's HANDOVER (the nonexistent
HEAP_FSM_CREATION_THRESHOLD), not to the syllabus.

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg5_grow, pg5_empty, pg5_tiny, pg5_tiny2, pg5_one, pg5_gran, pg5_round,
pg5_stale, pg5_big, pg5_cons. Keep the pgNN_ prefix convention.

-- LESSON 06 (2026-10-25) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-25-postgres-visibility-map/README.md (single README, not split -- one
fork, one mechanism). Source commits read: master @0019a1ecf4c7615a2570d21eeb4b32ae3b371f18
(visibilitymap.c, visibilitymap.h, visibilitymapdefs.h) PLUS REL_16_STABLE @774c47de (the same
three files + the five VM call sites in heapam.c) and REL_16_STABLE @0cf2c7e3 (heapam_xlog.h).
Trees so far: 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd, 04 @f52d4051, 05 @126976a8,
06 @0019a1ec (+ @774c47de / @0cf2c7e3). Keep stating the commit.
Files read IN FULL: access/heap/visibilitymap.c (master), access/visibilitymap.h,
access/visibilitymapdefs.h, access/heapam_xlog.h (16). Read in part: heapam.c (16) -- only the
all_visible_cleared / all_frozen_set sites.

*** THIS RUN HAD A CLUSTER and crossed a VM PAGE BOUNDARY (35,000 heap blocks, ~273 MB). Same
recipe as 03/04/05, both traps still apply (initdb not as root; data dir not under the scratchpad
path -- /var/tmp works; psql -f scripts must also live somewhere world-readable). ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
segment_size 1GB, data_checksums OFF, pageinspect 1.12, pg_visibility 1.2, pg_freespacemap 1.2.
Disk headroom at the time: ~30 GB on /, which is what made the boundary table possible. pg_waldump
is at /usr/lib/postgresql/16/bin/pg_waldump and WORKS on the live cluster's segments; --stats=record
gives a per-record-type byte breakdown. A LATER LESSON THAT WANTS WAL EVIDENCE SHOULD USE IT (22,
23, 24, 25 all will).

TOOK, and must not be re-taught:
- THE CONSTANTS, all of visibilitymapdefs.h (25 lines, the whole public vocabulary):
  BITS_PER_HEAPBLOCK 2, VISIBILITYMAP_ALL_VISIBLE 0x01, VISIBILITYMAP_ALL_FROZEN 0x02,
  VISIBILITYMAP_VALID_BITS 0x03. THE 0b10 CODE POINT IS UNREACHABLE BY ASSERTION at both ends:
  Assert(flags != VISIBILITYMAP_ALL_FROZEN) in _set, Assert(flags != VISIBILITYMAP_ALL_VISIBLE)
  in _clear. Two bits, three states. Taught as a deliberate waste, not an accident.
- THE GEOMETRY, derived and then MEASURED. MAPSIZE = BLCKSZ - MAXALIGN(SizeOfPageHeaderData)
  = 8168. HEAPBLOCKS_PER_BYTE = BITS_PER_BYTE/BITS_PER_HEAPBLOCK = 4. HEAPBLOCKS_PER_PAGE
  = 8168*4 = 32672 = 255.25 MiB of heap per VM page = 8.03x the FSM's 4069. NO TREE, flat array,
  so no FSM_TREE_DEPTH analogue and no ragged remainder: 8168*4 is exact, where lesson 05's
  8164-4095 = 4069 stranded 27 leaf positions. Quote the header's "There's no extra headers".
- BYTE OFFSETS, ESTABLISHED AND MEASURED, reusable without re-deriving (this is lesson 06's
  equivalent of lesson 05's "leaf slot k at offset 4123+k"):
      VM page   = b / 32672
      offset    = 24 + (b % 32672)/4      <- PageGetContents is page+24, same base as lesson 05
      bit shift = (b % 4) * 2             <- low bit ALL_VISIBLE, high bit ALL_FROZEN
  MEASURED at the boundary: heap block 32671 -> VM page 0 byte index 8167 -> PAGE OFFSET 8191,
  the last byte of the page; heap block 32672 -> VM page 1 offset 24. Byte 23 is page header, 0.
- *** THE FIRST VM PAGE BOUNDARY CROSSED IN THIS TRACK. *** Unlogged pg6_bound, 140,000 rows x
  1800-byte pad -> 35,000 heap blocks. vm = 16384 (2 blocks, because 35000 > 32672);
  fsm = 90112 (11 blocks = ceil(35000/4069)=9 bottom + 1 level-1 + 1 root, LESSON 05'S ARITHMETIC
  CONFIRMED INDEPENDENTLY). Byte 8191 of VM page 0 = 0xff, byte 24 of VM page 1 = 0xff.
  pg_visibility_map_summary: 35000 / 35000.
- THE FORK FLOOR, side by side with lesson 05: a ONE-BLOCK heap after its first VACUUM has
  vm = 8192 (ONE block) and fsm = 24576 (THREE blocks). The VM has no tree so nothing stops a
  map page being block 0. *** AND A REFINEMENT OF LESSON 05: lesson 05 measured "a one-page table
  has NO FSM fork (0 bytes)". That is the PRE-VACUUM state only -- VACUUM creates the FSM fork
  even for a one-block heap (measured 0 -> 24576). Do not propagate the unqualified claim. ***
- AN FSM PAGE IS AN EMPTY PAGE / so is a VM page: PageInit(page, BLCKSZ, 0) in vm_readbuf, so
  MEASURED pd_lower 24, pd_upper = pd_special = 8192, pd_flags 0, prune_xid 0, PageIsEmpty()
  always true, payload entirely in lesson 02's HOLE. ONE DIFFERENCE FROM THE FSM WORTH KEEPING:
  pd_lsn on a VM page is NON-ZERO and set deliberately (PageSetLSN in visibilitymap_set);
  measured 0/15E9740. On an FSM page it is incidental.
- PD_ALL_VISIBLE, WHICH LESSON 02 DEFERRED, IS NOW CLOSED. Taught as the two-places-one-truth
  design with a reason: the VM bit is what a READER consults (cheap, stays cached); the page flag
  is what a WRITER consults (it already holds the page; pinning the VM page might need I/O). That
  asymmetry is the ONLY reason the pin-before-lock dance exists.
- THE LOCKING DANCE, quoted from the header and taught: examine the heap page UNLOCKED, pin the VM
  page if PD_ALL_VISIBLE is set, then lock the heap page; if the flag got set in between, unlock,
  pin, relock. visibilitymap_pin_ok() is the cheap re-check (pure block-number compare, no I/O).
  The header's "This shouldn't happen often, because only VACUUM currently sets visibility map
  bits" was FLAGGED AS ARGUABLY STALE -- see COPY FREEZE below. Lesson 06's Next Step 3 proposes
  it as a documentation patch.
- THE FIVE CLEAR CALL SITES IN heapam.c (16), tabulated: heap_insert, heap_multi_insert (guarded
  `&& !(options & HEAP_INSERT_FROZEN)`), heap_delete, heap_update old page, heap_update new page
  (`newbuf != buffer &&`). All five do PageClearAllVisible + visibilitymap_clear(VALID_BITS) --
  ALWAYS BOTH BITS, which is why the Assert(flags != ALL_VISIBLE) in _clear can exist. All five
  raise a flag in the HOST record: XLH_INSERT_ALL_VISIBLE_CLEARED, XLH_DELETE_ALL_VISIBLE_CLEARED,
  XLH_UPDATE_OLD_ALL_VISIBLE_CLEARED, XLH_UPDATE_NEW_ALL_VISIBLE_CLEARED.
  visibilitymap_clear uses MarkBufferDirty, *NOT* MarkBufferDirtyHint -- the contrast with lesson
  05 is literally one function name deep. Taught that way.
- *** WHY THE VM IS WAL-LOGGED AND THE FSM IS NOT, with PG 16's own argument quoted in full.
  The record does NOT protect the bit; it keeps the TWO COPIES from diverging in the unsafe
  direction (map says visible, page has forgotten -> no writer will ever clear it). Clearing needs
  no record because the clearing is always INSIDE another record. ***
  xl_heap_visible = { TransactionId snapshotConflictHorizon; uint8 flags; }, SizeOfHeapVisible = 5,
  XLOG_HEAP2_VISIBLE = 0x40, "Backup blk 0: visibility map buffer / Backup blk 1: heap buffer".
- MEASURED WITH pg_waldump, the price of all that: 980 Heap2/VISIBLE records in one 16 MB segment;
  57,830 record bytes + 16,384 FPI bytes = 74,214 total = 5.65% of the segment. Histogram: 978
  records at EXACTLY 59 BYTES, 2 at 64/8256 (the FPI of each VM page on its first touch after a
  checkpoint). flags 0x01 observed for VACUUM with freeze off, 0x03 for VACUUM FREEZE. Verbatim
  record shape recorded in the lesson, with the two blkrefs and `fork vm blk 0 FPW`.
  SO: 59 BYTES OF WAL PER HEAP PAGE MARKED, plus one FPI per VM page per checkpoint. The FSM pays
  zero for the same VACUUM. That pair of numbers is the whole 05-vs-06 tradeoff and a later lesson
  should cite it rather than re-measure.
  NOT a VM-only figure, recorded separately so nobody confuses them: a VACUUM of a 118-block table
  emitted 18,400 WAL bytes total across all record types = 155.9 bytes/block. Lesson 19 owns that
  breakdown.
- COPY FREEZE SETS THE BITS AT LOAD TIME, measured, NO VACUUM: heapam.c's
  `starting_with_empty_page = PageGetMaxOffsetNumber(page) == 0; if (starting_with_empty_page &&
  (options & HEAP_INSERT_FROZEN)) all_frozen_set = true;` with
  Assert(!(all_visible_cleared && all_frozen_set)) and XLH_INSERT_ALL_FROZEN_SET.
  MEASURED: 5000 rows, 80 heap blocks, 47 blocks (0..46) with BOTH bits, byte 24 = 0xff.
  THE 33 UNSET BLOCKS ARE TRAILING *EMPTY* PAGES left by bulk relation extension -- max block
  actually containing a row was 46, so no heap_multi_insert ever ran on 47..79. Autovacuum later
  truncated the relation to 47 blocks and all 47 were all-visible+all-frozen. Lesson 06's Next
  Step 4 hands the "why does extension overshoot" question to lesson 08.
- *** THE HEADLINE MEASUREMENT, and the one a later lesson should cite: PAGE GRANULARITY COSTS 59
  HEAP FETCHES FOR ONE UPDATED ROW. *** pg6_ios, 50,000 rows / 863 heap blocks / PK on id, query
  `select count(id) from pg6_ios where id between 1 and 20000`, seqscan+bitmapscan off:
    after INSERT (no vm fork):          Heap Fetches 20000, Buffers shared hit=401
    after VACUUM:                        Heap Fetches     0, Buffers shared hit=57   (7.0x)
    after ONE update of id=7:            Heap Fetches    59, Buffers shared hit=60
    after pg_truncate_visibility_map():  Heap Fetches 20000, Buffers shared hit=403
  THE PLAN NODE WAS `Index Only Scan using pg6_ios_pkey` IN ALL FOUR -- taught as "index-only is a
  plan shape, not a promise; Heap Fetches is the only place EXPLAIN tells you which happened".
  THE 59 IS FULLY ACCOUNTED FOR: id=7 lived on block 0, the new version landed at (862,5), and
  pg_visibility showed EXACTLY blocks 0 and 862 cleared; block 0 holds 57 live rows + 1 dead index
  entry still pointing at (0,7) = 58, plus 1 on block 862 = 59.
- MEASURED, the byte sequence that proves the whole addressing scheme with three get_byte calls,
  on a 12-block table: (no fork) -> 85 -> 255 -> 243.
  85 = 0x55 = 01010101 = four blocks all-visible, none frozen -- LITERALLY VISIBLE_MASK8.
  255 = 0xff after VACUUM FREEZE. 243 = 0xf3 after updating one row on BLOCK 1
  = 0xff & ~(0x03 << 2), i.e. HEAPBLK_TO_OFFSET(1) = 2 confirmed at the bit level.
  Byte 27 = 0 because the heap stops at block 11 -- which is what makes visibilitymap_count's
  "we assume that any extra bytes in the last page are zeroed" safe.
- visibilitymap_count: VISIBLE_MASK8 0x55 "the lower bit of each bit pair", FROZEN_MASK8 0xaa
  "the upper bit of each bit pair", pg_popcount_masked, NO LOCK ("immediately stale anyway... we
  only really need an approximate value"). This is what feeds pg_class.relallvisible. Lesson 06's
  Next Step 6 leaves the planner's cost formula to lesson 31.
- visibilitymap_get_status: ONE UNLOCKED BYTE READ, "A single byte read is atomic", "It is the
  caller's responsibility to deal with all concurrency issues!". THE TWO-PARAGRAPH SAFETY PROOF
  WAS TAUGHT because it is specific and non-obvious: INSERTS are serialised for free (the VM bit
  is cleared BEFORE the index is touched, and index buffer lock/unlock is a full memory barrier);
  DELETES may be arbitrarily stale but the tuple stays visible until the deleter commits and
  ProcArrayLock acts as the read barrier. FLAGGED AS A CONSTRAINT: any NEW consumer of the VM has
  to re-derive that argument for itself.
- vm_readbuf's RBM_ZERO_ON_ERROR, same justification as the FSM ("It's always safe to clear bits,
  so it's better to clear corrupt pages than error out"), and the extend=false -> InvalidBuffer ->
  `return (uint8) 0` path, which is why a table with NO vm fork behaves as "nothing is visible"
  rather than erroring. Matches the docs' "queries will treat the visibility map as containing all
  zeroes". vm_extend = ExtendBufferedRelTo(EB_CREATE_FORK_IF_NEEDED | EB_CLEAR_SIZE_CACHE) +
  CacheInvalidateSmgr; relation extension itself left to lesson 08 as a black box, as lesson 05
  did.
- visibilitymap_prepare_truncate: MemSet(&map[truncByte+1], 0, MAPSIZE-(truncByte+1)) and
  map[truncByte] &= (1 << truncOffset) - 1, with the reason quoted -- the tail bits are cleared
  "because we don't get a chance to clear the bits if the heap is extended again", i.e. a
  re-extended heap would inherit STALE SET BITS, the unsafe direction. This is the one VM write
  gated on XLogHintBitIsNeeded() -> log_newpage_buffer, with "We cannot use MarkBufferDirtyHint
  here, because that will not dirty the page during recovery". UNEXERCISED: data_checksums was OFF
  this run, so that branch was not observed. Lesson 06's Next Step 5.
- pg_visibility 1.2 as the observation tool, all signatures taught from the docs: pg_visibility_map
  (2 overloads), pg_visibility (2 overloads, adds pd_all_visible), pg_visibility_map_summary,
  pg_check_frozen, pg_check_visible, pg_truncate_visibility_map. Permissions recorded: superuser
  or pg_stat_scan_tables, except pg_truncate_visibility_map which is superuser-only.
- *** MEASURED, A FORCED TWO-PLACES DISAGREEMENT, and the blind spot it exposes. ***
  pg_truncate_visibility_map('pg6_ios') then pg_visibility: vm all_visible = 0 of 863, heap pages
  pd_all_visible = 861 (861 not 863 because two blocks were already cleared by the UPDATE).
  Nothing corrupt; the optimisation simply stopped. AND pg_check_visible STILL RETURNED 0 ROWS --
  it is defined as "non-all-visible tuples stored in pages marked all-visible IN THE VISIBILITY
  MAP", so a map that marks nothing can hide nothing. THE CHECKS DETECT OVER-CLAIMING ONLY. A
  clean pg_check_visible is NOT evidence the map is right. After the repairing VACUUM: 862 / 862.
- The docs' "four orders of magnitude smaller" claim was CHECKED rather than repeated: the
  asymptote is 32672:1 (3.27e4, so the claim is right) but THE FORK HAS A ONE-PAGE FLOOR, so
  MEASURED ratios were 1:1 for a one-block table, 863:1 for pg6_ios, 17500:1 for pg6_bound. The
  advertised ratio only arrives past 255.25 MiB of heap.

VERSION DIVERGENCE: LARGER THAN LESSON 05'S, and it moves RESPONSIBILITY rather than geometry.
(1) visibilitymap_set in 16 is
    `(Relation rel, BlockNumber heapBlk, Buffer heapBuf, XLogRecPtr recptr, Buffer vmBuf,
      TransactionId cutoff_xid, uint8 flags) -> void`;
    in master it is `(BlockNumber heapBlk, Buffer vmBuf, uint8 flags, RelFileLocator rlocator)
      -> uint8` and RETURNS THE PREVIOUS BITS.
(2) WHO LOCKS: 16's function does LockBuffer(vmBuf, BUFFER_LOCK_EXCLUSIVE) itself; master asserts
    the CALLER holds it, via BufferIsLockedByMeInMode (a predicate 16 does not have).
(3) WHO LOGS: 16's function calls log_heap_visible() and PageSetLSN, and sets the HEAP page's LSN
    only `if (XLogHintBitIsNeeded())` with a four-line comment on why it must NOT otherwise;
    master does NO WAL at all and asserts `InRecovery || CritSectionCount > 0`, leaving the caller
    to log the VM change together with the heap change.
(4) THE HEADER COMMENT CHANGED WITH IT, and 16's is the clearer statement of WHY: 16 says
    "Clearing visibility map bits is not separately WAL-logged" + "When we *set* a visibility map
    during VACUUM, we must write WAL. This may seem counterintuitive, since the bit is basically a
    hint..."; master generalises to "Changes to the visibility map bits are not separately
    WAL-logged" because setting no longer logs either. QUOTE 16'S VERSION even when reading master.
(5) visibilitymap_count: 16 uses VISIBLE_MASK64/FROZEN_MASK64 + pg_popcount64 over uint64 words
    with StaticAssertStmt(MAPSIZE % sizeof(uint64) == 0, "unsupported MAPSIZE"); master uses
    VISIBLE_MASK8 0x55 / FROZEN_MASK8 0xaa + pg_popcount_masked over bytes.
(6) master adds HEAPBLK_TO_MAPBLOCK_LIMIT and visibilitymap_truncation_length(); 16 has neither.
(7) master's visibilitymap_clear gained the same BufferIsLockedByMeInMode assertion.
IDENTICAL IN BOTH: BITS_PER_HEAPBLOCK, all three flag constants, MAPSIZE, HEAPBLOCKS_PER_BYTE,
HEAPBLOCKS_PER_PAGE, all three address macros, PageInit(page, BLCKSZ, 0), RBM_ZERO_ON_ERROR,
vm_extend's ExtendBufferedRelTo + CacheInvalidateSmgr, and prepare_truncate's masking arithmetic.
EVERY MEASURED NUMBER IS PG 16.15.

NO SYLLABUS REVISION THIS RUN. Lesson 06's title was taught exactly as written -- PD_ALL_VISIBLE,
all-frozen and the index-only scan it enables were all three taken in full, and nothing was left
over. Lesson 05's handover scope was executed completely, including the two measurements it asked
for by name (decode a VM page by hexdump and recover the bits for a known block; compare
pg_relation_size(rel,'vm') against (rel,'fsm') on the same table).

Throwaway tables left by this run (cluster does not persist; names listed only to avoid collisions
within a future run): pg6_one, pg6_move, pg6_ios, pg6_freeze, pg6_bound, pg6_walcost, pg6_wal.
Keep the pgNN_ prefix convention.

-- HANDOVER TO LESSON 07 -----------------------------------------------------------------------
Scope for lesson 07, "HOT updates and page pruning: redirect line pointers, heap_page_prune(), and
the HOT chain". Lesson 06 ends on exactly the right cliff for it: the headline measurement is that
one UPDATE cost 59 heap fetches because the new version landed on a DIFFERENT block (862) and so
cleared TWO VM bits and added a NEW index entry. HOT is the mechanism that makes that update not
happen that way -- same page, no new index entry, one VM bit cleared instead of two. LEAD WITH
THAT. Lesson 06 has already paid for the setup; 07 should open by re-deriving the 59 under HOT and
saying what it becomes.
Take: src/backend/access/heap/README.HOT (THE primary source, in-tree, not yet read in this
track -- it is the design document the whole feature is specified in); heap_page_prune(),
heap_page_prune_opt() and heap_prune_chain() in src/backend/access/heap/pruneheap.c;
LP_REDIRECT as the fourth lp_flags value (lesson 02 catalogued LP_UNUSED/LP_NORMAL/LP_DEAD and
*named* LP_REDIRECT -- 07 owns it); pd_prune_xid and PD_PAGE_FULL, both of which lesson 02
catalogued and deferred; HEAP_HOT_UPDATED and HEAP_ONLY_TUPLE, which lesson 03 tabulated in
t_infomask2 and assigned to 07; the HOT-eligibility test in heap_update (what makes an update
non-HOT: an indexed column changed, or no room on the page); and the chain-following rule lesson 03
taught (validate with xmin(next) == xmax(this)) applied to a redirect.
MEASURE IT. The cluster recipe works and is below; pgstattuple is present and lesson 07 is one of
the two lessons the environment note already earmarks it for. Good candidates, in rough order of
value: (a) RE-RUN LESSON 06'S pg6_ios EXPERIMENT with fillfactor low enough that the new version
stays on the same page, and show Heap Fetches going to ~57 instead of 59 -- one VM bit instead of
two; (b) heap_page_items on a page mid-HOT-chain, showing lp_flags = 2 (LP_REDIRECT) and an
lp_off that is an OFFSET NUMBER rather than a byte offset, which is the single most confusing
thing about the line pointer array and lesson 02 did not cover it; (c) watch pd_prune_xid get set
and then cleared by an opportunistic prune triggered by a plain SELECT, which is the fact most
people do not believe; (d) count index tuples before and after a run of HOT updates to show the
index did not grow; (e) pgstattuple's dead_tuple_percent across prune and vacuum.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, PageInit, the four free-space
functions, the hole (02); the tuple header, t_ctid's four meanings, the infomask catalogues,
BITMAPLEN, t_hoff, the null bitmap (03); alignment, varlena, the 126-byte boundary, attcacheoff,
atthasmissing, TOAST (04); the FSM's categories, its on-page max-heap, fp_next_slot,
fsm_logical_to_physical, the 3-block floor, the 32-byte rounding and its ceiling trap,
MaxHeapTuplesPerPage = 291 (05); ANY of the visibility map -- the constants, MAPSIZE, the 32672
geometry, the byte/bit offsets, PD_ALL_VISIBLE's two-places design, the pin-before-lock dance, the
five clear call sites, why it is WAL-logged, xl_heap_visible, pg_visibility's functions, the
59-fetch measurement (06). CITE the 59 and re-derive what HOT does to it; do not re-teach the map.
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 2032, 1996, 8136, 126, 18, 4069,
4095, the 32-byte FSM_CAT_STEP, the FSM leaf offset 4123+k, the page fingerprint "00 20 04 20" at
offsets 16-19 (WHICH IS NOT FORK-SPECIFIC -- say "special space is zero"), and now: MAPSIZE 8168,
HEAPBLOCKS_PER_PAGE 32672, the VM offset formula 24 + (b % 32672)/4, bit shift (b % 4)*2,
VISIBLE_MASK8 0x55 / FROZEN_MASK8 0xaa, the 59-byte XLOG_HEAP2_VISIBLE record, and the VM's
one-block fork floor against the FSM's three.
Do NOT take: smgr, md.c, relation extension, the fsync queue (08) -- lesson 06 treated "extend the
relation" as a black box and 07 should too, EXCEPT that lesson 06 left 08 a concrete question (see
OPEN ITEMS); the buffer manager, BufferDesc, clock sweep, the strategy ring (09); the heavyweight
lock manager (11); tuple lock strengths (12); hint bits and their FPI machinery (15, 22) -- lesson
06 quoted MarkBufferDirty vs MarkBufferDirtyHint as a CONTRAST and did not teach either; snapshots
and HeapTupleSatisfiesMVCC (14) -- 07 will be tempted, because "is this tuple dead" is exactly a
snapshot question; use GlobalVisState/vistest as a named black box and defer; freezing and
relfrozenxid (18); VACUUM's two heap passes, the dead-TID store, index bulk deletion (19) -- the
distinction 07 must hold is PRUNING (opportunistic, single page, no index touched) versus VACUUMING
(index-aware, two passes), and 07 owns only the first; amgettuple/amgetbitmap and the bitmap heap
scan (30).
OPEN ITEMS carried forward:
- FOR LESSON 08: why does bulk relation extension overshoot? Lesson 06 measured COPY FREEZE
  leaving 33 trailing EMPTY pages out of 80, which is also why only 47 blocks got VM bits. The
  answer is in RelationGetBufferForTuple's extension batching, which is 08's territory.
- FOR LESSON 19 (unchanged from lesson 05, still not closed): fsm_vacuum_page() and
  FreeSpaceMapVacuumRange() have still not been read. They are the only code that makes the upper
  FSM levels true. ALSO NEW for 19: lesson 06 measured a VACUUM emitting 18,400 WAL bytes over 118
  blocks (155.9/block) of which the VISIBLE records are a minority; 19 owns that breakdown.
- FOR LESSON 22/24/25: pg_waldump works on this cluster and --stats=record gives per-record-type
  byte and FPI counts. Lesson 06 used it for the first time in this track. Use it.
- FOR LESSON 31: pg_class.relallvisible comes from visibilitymap_count; the index-only-scan cost
  formula that reads it was not located this run.
- UNEXERCISED BRANCH, for whichever lesson runs with checksums on first:
  visibilitymap_prepare_truncate's `XLogHintBitIsNeeded() -> log_newpage_buffer` path. Every
  measurement in lessons 03-06 was taken with data_checksums OFF.
- A DOCUMENTATION PATCH WORTH PROPOSING UPSTREAM, if anyone wants it: visibilitymap.c's LOCKING
  section still says "only VACUUM currently sets visibility map bits", which heap_multi_insert's
  all_frozen_set contradicts. Lesson 06 flagged it rather than fixing it.

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 03's note was dropped and TRACK-ARCHIVE.md gained the row
"PG 03" pointing at commit 610357515b8e64cde4cb22cfaa3a34dd019456c4 / blob
91f09a63f3a86af94a3d519f09c62e5d69272be5, which is the last state of TRACK.md carrying it. This
block now holds 04, 05 and 06 -- exactly the three the policy allows. WHEN LESSON 07 SHIPS IT MUST
DROP THE LESSON 04 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap covered lesson 05; pg_visibility covers 06.
  Lesson 01's hands-on installs all three; lesson 02's needs only pageinspect; lesson 05's needs
  pageinspect and pg_freespacemap.
- CONFIRMED 2026-10-22, 2026-10-23, 2026-10-24 AND AGAIN 2026-10-25: the cloud sandbox CAN run a
  cluster.
  psql, pg_config and /usr/lib/postgresql/16/bin/{initdb,postgres,pg_ctl,...} are in the image
  (PostgreSQL 16.15 on Ubuntu 24.04), and pageinspect, pg_freespacemap, pg_visibility, pgstattuple
  and amcheck are all present in /usr/share/postgresql/16/extension/. Recipe that worked: create a
  non-root user, initdb -D /var/tmp/pgdata --no-sync as that user, pg_ctl with
  -o '-p 5455 -k /var/tmp'. TWO TRAPS: initdb refuses to run as root, and it refuses a data
  directory under the scratchpad path (permission denied) -- /var/tmp works. A third, minor, found
  this run: psql cannot read a -f script file living under the scratchpad path either (permission
  denied as the pg user), so put scratch SQL somewhere world-readable such as /var/tmp. Every
  lesson from here on SHOULD measure; say so when it cannot. The cluster is per-run and does not
  persist, so table names from earlier runs are gone.
- Table names used so far, for avoiding collisions within a run: seg_demo (01), pg2_fill /
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04), pg5_* (05), pg6_* (06 --
  the full lists are in the lesson 04, 05 and 06 notes above). Keep the pgNN_ prefix convention.
- pg_freespacemap was used by lesson 05 and pg_visibility by lesson 06 (both present in
  /usr/share/postgresql/16/extension/, both installed cleanly).
- pg_waldump lives at /usr/lib/postgresql/16/bin/pg_waldump, works on the live cluster's segments,
  and `--stats=record` prints a per-record-type record/FPI byte breakdown. Lesson 06 used it to
  price XLOG_HEAP2_VISIBLE. Lessons 21-25 should use it rather than reasoning about WAL on paper.
- pg_buffercache is needed from lesson 09; pgstattuple for
  lessons 07 and 19; amcheck for lessons 28-29 (it is the closest thing PostgreSQL has to SQLite's
  integrity_check, and the comparison is worth making explicitly when lesson 28 arrives).
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
