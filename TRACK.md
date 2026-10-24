<!--
track-state
subject: PostgreSQL
started: 2026-10-19
lessons-total: 35
lessons-done: 5
-->

# Current Track: PostgreSQL Internals

Sequential deep dive. One lesson per day, each building on the last.

**STATUS: in progress — lesson 05 shipped 2026-10-24.** The syllabus was written on 2026-10-19, the
day the SQLite track's capstone shipped. The next run takes **lesson 06**.

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
- [ ] 06 — The Visibility Map: `PD_ALL_VISIBLE`, all-frozen, and the index-only scan it enables
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

-- LESSON 03 (2026-10-22) AS TAUGHT ------------------------------------------------------------
Document: 2026/10/2026-10-22-postgres-heap-tuple-header/README.md (single README, not split).
Source commit read: postgres/postgres@f2edf5dd. Lesson 01 @ddd468ca, 02 @3cde90ec, 03 @f2edf5dd --
the tree moved all three times. Keep stating the commit.

*** THIS RUN HAD A CLUSTER. Part I is no longer unmeasured. ***
PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64, gcc 13.3.0, block_size 8192,
segment_size 1GB, data_checksums OFF, pageinspect 1.12, running as a throwaway cluster in the
sandbox (initdb into /var/tmp, port 5455, unix socket only). A LATER RUN CAN DO THE SAME: psql and
/usr/lib/postgresql/16/bin/* are present in the cloud sandbox image, and pageinspect,
pg_freespacemap, pg_visibility, pgstattuple and amcheck are all in
/usr/share/postgresql/16/extension/. initdb must run as a NON-ROOT user (a 'pg' user was created)
and the data directory must NOT be under the scratchpad path (permission denied); /var/tmp works.
Version gap must be stated: struct and flag constants from master, observations from 16.

TOOK, and must not be re-taught:
- HeapTupleHeaderData BY BYTE OFFSET: 0 t_xmin(4), 4 t_xmax(4), 8 t_field3(4), 12 t_ctid(6),
  18 t_infomask2(2), 20 t_infomask(2), 22 t_hoff(1), 23 t_bits[]. SizeofHeapTupleHeader =
  offsetof(t_bits) = 23 (not sizeof, same reason as SizeOfPageHeaderData in lesson 02).
  MinHeapTupleSize = MAXALIGN(23) = 24. MEASURED: a zero-column table's tuple has lp_len 24,
  t_hoff 24, natts 0, lp_off 8168.
- THE TWO NESTED UNIONS, and the 27-vs-23 resolution: docs Table 66.4 lists t_cid and t_xvac as
  separate 4-byte rows (sum 27) but they are arms of t_field3. Outer union t_choice overlays
  HeapTupleFields with DatumTupleFields {int32 datum_len_, int32 datum_typmod, Oid datum_typeid};
  heap_form_tuple fills the Datum arm for EVERY tuple ("even though this tuple may never become a
  Datum") and the insert path overwrites it. On disk t_choice is always t_heap. t_xvac only via
  pg_upgrade; HeapTupleHeaderGetXvac returns InvalidTransactionId unless HEAP_MOVED.
  Combo CIDs were NAMED AND DEFERRED to lesson 16.
- MEASURED: t_cid observable as 0/1/2 for three inserts in one transaction, same t_xmin.
  heap_page_items names the column t_field3 and comments it "/* shared with xvac */" -- nothing in
  the tuple says which arm you are reading except HEAP_MOVED.
- ItemPointerData fully: 6 bytes, {BlockIdData ip_blkid; OffsetNumber ip_posid}, pg_attribute_packed
  + pg_attribute_aligned(2), with the header's "you'll waste lots of space" warning quoted.
  BlockIdData {bi_hi, bi_lo} exists ONLY so a block number can be SHORTALIGN'd (block.h rationale
  quoted). bi_hi = blkno>>16, bi_lo = blkno & 0xffff. MEASURED on block 299 (0x12b):
  ip_blkid bytes "00 00 2b 01", ip_posid "01 00" -- high half first, each half LE, THE SAME SHAPE
  AS pd_lsn from lesson 02. Lesson 01 deferred ItemPointerData's width; it is now CLOSED.
- t_ctid's FOUR meanings: self (latest), forward pointer (superseded), MovedPartitionsOffsetNumber
  0xfffd + InvalidBlockNumber, SpecTokenOffsetNumber 0xfffe with the token in ip_blkid. Both magics
  exceed MaxOffsetNumber and there is a StaticAssertDecl(MaxOffsetNumber < SpecTokenOffsetNumber).
  THE CHAIN-FOLLOWING RULE, quoted and then measured: validate with xmin(next) == xmax(this),
  because VACUUM may erase the pointed-to tuple first and the slot may be reused.
- MEASURED: insert-then-update pair, lp1 t_ctid (0,2) / xmax 747, lp2 t_ctid self / xmin 747.
  That update happened to be HOT; HOT, redirect line pointers and pruning remain lesson 07's
  entirely and were explicitly deferred.
- t_infomask2: HEAP_NATTS_MASK 0x07FF (11 bits => 2047), "bits 0x1800 are available",
  HEAP_KEYS_UPDATED 0x2000, HEAP_HOT_UPDATED 0x4000, HEAP_ONLY_TUPLE 0x8000, HEAP2_XACT_MASK 0xE000.
  HEAP_TUPLE_HAS_MATCH IS HEAP_ONLY_TUPLE under another name (hash-join overlay, rationale quoted).
  ESTABLISHED AND REUSABLE: the natts mask is NOT the binding constraint on column count --
  t_hoff being uint8 is, per the MaxTupleAttributeNumber header comment ("a little over 1700");
  1664 = 8*208 and 1600 = 8*200 are round numbers so layout changes do not move a user-visible
  limit. MEASURED: "ERROR: tables can have at most 1600 columns" at 1601 columns.
- NATTS IS PER-TUPLE, NOT PER-TABLE, and this was MEASURED: ALTER TABLE ADD COLUMN with no DEFAULT
  does not rewrite, so one page held tuples with natts 2 and natts 3, and the natts-2 tuple reads
  its third column as NULL with HEAP_HASNULL CLEAR and no bitmap at all. pageinspect's
  tuple_data_split_internal comment quoted as the authority. CONSEQUENCE TAUGHT: a NULL has TWO
  on-disk encodings (0 bit in bitmap, or attnum > natts) and heap_attisnull adds a third,
  atthasmissing -- which was NAMED AND DEFERRED to lesson 04.
- t_infomask: ALL SIXTEEN BITS tabulated with their owning lesson (0x0001 HASNULL, 0x0002
  HASVARWIDTH, 0x0004 HASEXTERNAL, 0x0008 HASOID_OLD, 0x0010 XMAX_KEYSHR_LOCK, 0x0020 COMBOCID,
  0x0040 XMAX_EXCL_LOCK, 0x0080 XMAX_LOCK_ONLY, 0x0100 XMIN_COMMITTED, 0x0200 XMIN_INVALID,
  0x0400 XMAX_COMMITTED, 0x0800 XMAX_INVALID, 0x1000 XMAX_IS_MULTI, 0x2000 UPDATED,
  0x4000 MOVED_OFF, 0x8000 MOVED_IN). THE FLAG WORD IS FULL; three bits are held purely for
  pre-9.0 pg_upgrade compatibility. HEAP_XACT_MASK 0xFFF0.
- COMPOSITE CONSTANTS ARE NOT BITS: HEAP_XMIN_FROZEN = COMMITTED|INVALID = 0x0300 ("committed AND
  invalid" is the frozen encoding), HEAP_XMAX_SHR_LOCK = 0x0050, HEAP_MOVED = 0xC000,
  HEAP_LOCK_MASK. Hence GetRawXmin vs GetXmin, and hence heap_tuple_infomask_flags returning TWO
  arrays (raw_flags tests != 0, combined_flags tests (mask & COMBO) == COMBO). Freezing itself was
  DEFERRED to lesson 18; hint bits to 15; MultiXact bits to 17; tuple-lock strengths to 12.
- MEASURED: a fresh insert has t_infomask = 2048 = 0x0800, i.e. HEAP_XMAX_INVALID SET while
  t_xmax = 0 -- the flag is the fast path so visibility need not read the field. With one NULL,
  2049 = HEAP_XMAX_INVALID|HEAP_HASNULL. raw_flags/combined_flags output for the update pair:
  {HASVARWIDTH,XMIN_COMMITTED,HOT_UPDATED} and {HASVARWIDTH,XMAX_INVALID,UPDATED,ONLY_TUPLE}.
- t_hoff AND THE NULL BITMAP, fully taught -- LESSON 04 MUST NOT RE-TEACH THE BITMAP.
  heap_form_tuple's five-line layout computation quoted. BITMAPLEN(NATTS) = (NATTS+7)/8, sized from
  natts NOT from the null count. t_hoff = MAXALIGN(23 + BITMAPLEN(natts)) when hasnull, else
  MAXALIGN(23). ESTABLISHED, MEASURED, REUSABLE STEP FUNCTION: natts 8 -> t_hoff 24 (bitmap FREE,
  it lands in the pad byte that exists anyway); natts 9 -> 32; natts 16 -> 32; natts 72 -> 32
  (bitmap 9 bytes, zero pad); natts 73 -> 40. So the 9th nullable column costs 8 bytes/row forever
  and columns 9-72 cost nothing more in header. lp_len measured at 52/64/92/316/328 respectively.
- BITMAP BIT ORDER AND POLARITY: att_isnull = !(BITS[ATT>>3] & (1 << (ATT & 0x07))) -- LSB-first,
  and 1 MEANS NOT NULL. fill_val's `bitP = &bit[-1]; bitmask = HIGHBIT` is a SENTINEL that forces
  the first iteration down the advance branch, NOT an MSB-first ordering -- taught explicitly
  because it reads like the opposite. bits_to_text prints in att_isnull order.
  MEASURED: 3 int columns with the middle NULL gives t_bits '10100000', lp_len 32 vs 36 for the
  same shape with no nulls -- the NULL costs zero data bytes and the bitmap was free.
  length(t_bits) = BITMAPLEN(natts)*8, so it reveals the bitmap's BYTE count, not the column count.
- palloc0 => pad bytes are zero on disk, tied back to lesson 02's "the checksum covers the hole".
- THE FULL HEXDUMP, measured and decoded field by field (3 ints, no nulls, lp_off 8152, lp_len 36):
  "dc020000 00000000 00000000 00000000 0100 0300 0008 18 00 | 01000000 02000000 03000000"
  => xmin 732, xmax 0, t_field3 0, ctid (0,1), infomask2 3, infomask 0x0800, t_hoff 0x18=24,
  byte 23 = 00 = PURE MAXALIGN PAD (no HASNULL so no bitmap lives there), then three LE int4.
  A later lesson may cite this dump; it does not need re-deriving.
- THE REDUNDANCY THREAD (opened lesson 02) extended as planned: PostgreSQL's record header carries
  NO type information at all, so decoding needs pg_class/pg_attribute -- which is why
  tuple_data_split() takes a relid and calls relation_open(). SQLite's varint record header makes a
  page decodable in isolation. Framed as the forensic consequence: a salvaged page parses to
  t_hoff and no further.
- MinimalTupleData and MINIMAL_TUPLE_OFFSET were mentioned in Further Study ONLY, not taught.
  Lesson 32 still owns the executor's tuple slots.

MEASUREMENTS NOW ON RECORD (first in this track): MinHeapTupleSize 24; the t_hoff step function at
8/9/16/72/73 columns; t_bits polarity; the 1600-column error text; t_cid 0/1/2; the ctid halfword
byte order on block 299; the full 24-byte header hexdump; fresh-insert infomask 0x0800.
STILL NOT MEASURED, and lesson 02 flagged these as the first things to verify once a cluster
exists: the heap page fingerprint bytes "00 20 04 20" at page offsets 16-19, and the 8160-byte
MaxHeapTupleSize insert ceiling. Lesson 03 did NOT get to them and put them in its Next Steps as
item 4. LESSON 04 SHOULD DO THEM -- it will have a cluster and it needs MaxHeapTupleSize anyway
for the TOAST threshold discussion.
Throwaway tables left by this run (drop or reuse, say which): pg3_fixed, pg3_null, pg3_upd,
pg3_alter, pg3_cid, pg3_empty, pg3_blk, w8, w9, w16, w72, w73. Lesson 01's seg_demo and lesson 02's
pg2_fill / pg2_big are from earlier runs' documents, not from a surviving cluster.

SYLLABUS REVISION MADE THIS RUN: lesson 04's title previously read "Attribute storage: MAXALIGN
padding, the null bitmap, 1-byte vs 4-byte varlena headers, and TOAST". Lesson 03 took the null
bitmap completely (BITMAPLEN, the t_hoff arithmetic, bit order, polarity, the measured step
function), so the title now reads "Attribute storage: MAXALIGN padding between attributes, 1-byte
vs 4-byte varlena headers, `atthasmissing`, and TOAST". atthasmissing was added because lesson 03
surfaced it as the third NULL encoding and deferred it. No reordering; no other change.

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

-- HANDOVER TO LESSON 06 -----------------------------------------------------------------------
Scope for lesson 06, "The Visibility Map: PD_ALL_VISIBLE, all-frozen, and the index-only scan it
enables". Lesson 05 answered "which page has room"; lesson 06 answers "which pages can be trusted
without looking at their tuples". The handoff is clean, and lesson 05 has already set up the frame:
the VM is the SECOND auxiliary fork, two BITS per heap block against the FSM's one BYTE, and it is
the fork that is WAL-logged where the FSM is not. Lead with that contrast -- it is the sharpest way
in, and lesson 05's reader has just been shown exactly what "not WAL-logged" buys and costs.
Take: src/backend/access/heap/visibilitymap.c and its long file-header comment (THE primary source,
not yet read in this track); src/include/access/visibilitymap.h; VISIBILITYMAP_ALL_VISIBLE and
VISIBILITYMAP_ALL_FROZEN, BITS_PER_HEAPBLOCK = 2, HEAPBLOCKS_PER_BYTE, HEAPBLOCKS_PER_PAGE and
MAPSIZE -- derive them against lesson 05's SlotsPerFSMPage = 4069 and say which fork is denser and
by how much; the LOCKING RULES in that header (the VM bit and PD_ALL_VISIBLE must be set together,
and the rule about what may be cleared without an exclusive lock); visibilitymap_set /
visibilitymap_clear / visibilitymap_get_status / visibilitymap_count / visibilitymap_prepare_truncate;
WHY THE VM *IS* WAL-LOGGED while the FSM is not, and what XLOG_HEAP2_VISIBLE does; the
index-only-scan path's dependence on it (lesson 30 owns amgetbitmap and the bitmap heap scan, so
reference the IOS, do not teach the index AM interface); pg_visibility's
pg_visibility_map / pg_visibility / pg_check_visible / pg_check_frozen as the observation tools; and
PD_ALL_VISIBLE itself, which lesson 02 catalogued as flag 0x0004 and EXPLICITLY DEFERRED TO 06 --
lesson 06 owns it entirely and must teach the two-places-one-truth invariant.
MEASURE IT. The recipe works and is below; lesson 05's measurements are a template. Good candidates:
decode a VM page by hexdump and recover the two bits for a known block (the offsets are as
derivable as lesson 05's 4123+k and nobody has written them down yet); show PD_ALL_VISIBLE and the
VM bit moving together across INSERT -> VACUUM -> UPDATE; show an index-only scan's Heap Fetches
going to 0 after VACUUM and back up after one UPDATE; and compare pg_relation_size(rel,'vm') against
(rel,'fsm') on the same table -- lesson 05 measured the FSM's 3-block floor and the VM's floor is a
different number, which is worth having side by side.
NOTE: page flags 5 = PD_HAS_FREE_LINES|PD_ALL_VISIBLE was OBSERVED by lesson 05 on a vacuumed page
(see the 291 note above). That is an observation of the bit existing, nothing more -- lesson 06
still owns the whole mechanism.
Do NOT re-explain: the page header, line pointers, lp_off/lp_len, PageInit, the four free-space
functions, checksums, the hole (02); the tuple header, t_ctid, the infomask catalogues, BITMAPLEN,
t_hoff, the null bitmap (03); the alignment macros, varlena layouts, the 126-byte boundary, the
pad-byte peek, attcacheoff, atthasmissing, ANY of TOAST (04); the FSM's categories, its on-page
max-heap, fp_next_slot, fsm_logical_to_physical, the 3-block floor, the 32-byte rounding and its
ceiling trap, MaxHeapTuplesPerPage = 291, or the "an FSM page is an empty page" framing (05).
Cite without re-deriving: 8192, 131072, 8160, 8164, 291, 24, 23, 2032, 1996, 8136, 126, 18, 4069,
4095, 8164 nodes, 32-byte FSM_CAT_STEP, the FSM leaf offset 4123+k, and the page fingerprint
"00 20 04 20" at offsets 16-19 -- WHICH LESSON 05 SHOWED IS NOT FORK-SPECIFIC. Say "special space
is zero", not "this is a heap page".
Do NOT take: HOT, pruning, LP_REDIRECT, pd_prune_xid, heap_page_prune (07); smgr, md.c, relation
extension, the fsync queue, ExtendBufferedRelTo internals (08) -- lesson 05 treated "extend the
relation" as a black box and lesson 06 should too; the buffer manager, BufferDesc, clock sweep (09);
MarkBufferDirtyHint's own implementation and the hint-bit FPI machinery (15, and 22 for FPIs) --
lesson 05 quoted the README's argument about it but did NOT teach the function; snapshots and
HeapTupleSatisfiesMVCC (14); freezing, relfrozenxid, anti-wraparound VACUUM (18) -- ALL-FROZEN as a
VM BIT belongs to 06, but what freezing *is* belongs to 18, so name the bit and defer the mechanism;
VACUUM's two heap passes, the dead-TID store, FreeSpaceMapVacuumRange's role in them (19);
amgettuple/amgetbitmap and the bitmap heap scan (30).
OPEN ITEM lesson 05 could not close and 19 will want: fsm_vacuum_page() and FreeSpaceMapVacuumRange()
were NOT read this run. They are the only code that makes the upper FSM levels true. Lesson 19 must
read them; lesson 06 does not need them.

NOTE-BLOCK ROTATION DONE THIS RUN: lesson 02's note was dropped and TRACK-ARCHIVE.md gained the row
"PG 02" pointing at commit 4b7ea85dcfcc830f71ca8ff16d7f26d84344b7a5 / blob
f47b8260ba0cab419ce2f4fc59a3e9c439ef6650, which is the last state of TRACK.md carrying it. This
block now holds 03, 04 and 05 -- exactly the three the policy allows. WHEN LESSON 06 SHIPS IT MUST
DROP THE LESSON 03 BLOCK, and before dropping it MUST add its TRACK-ARCHIVE.md row first.

Known environment facts for a fresh session (verify, do not assume):
- Block size 8 KB and segment size 1 GB are compile-time (BLCKSZ, RELSEG_SIZE) and exposed as the
  read-only GUCs block_size and segment_size. State the build you measured.
- pageinspect is what makes Part I observable (page_header, page_checksum, heap_page_items,
  heap_tuple_infomask_flags, tuple_data_split, bt_page_items, bt_metap, fsm_page_contents). All
  of it is SUPERUSER-ONLY. pg_freespacemap covered lesson 05; pg_visibility covers 06.
  Lesson 01's hands-on installs all three; lesson 02's needs only pageinspect; lesson 05's needs
  pageinspect and pg_freespacemap.
- CONFIRMED 2026-10-22, 2026-10-23 AND AGAIN 2026-10-24: the cloud sandbox CAN run a cluster.
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
  pg2_big (02), pg3_* and w8 / w9 / w16 / w72 / w73 (03), pg4_* (04), pg5_* (05 -- the full lists
  are in the lesson 04 and 05 notes above). Keep the pgNN_ prefix convention.
- pg_freespacemap was used by lesson 05 and pg_visibility is needed by lesson 06 (both present in
  /usr/share/postgresql/16/extension/). pg_buffercache is needed from lesson 09; pgstattuple for
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
