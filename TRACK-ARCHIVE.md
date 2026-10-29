# TRACK-ARCHIVE.md

Where to find the per-lesson scope and handover notes that have rolled off the end of
[`TRACK.md`](TRACK.md).

## Why this file exists

`TRACK.md` carries a comment block of per-lesson notes: what each lesson took, what it
deliberately left for a later lesson, which source it read, which measurements reproduced and
which did not. On 2026-10-12 that block had reached **68 KB**, and because the GitHub API can
only replace a file whole, every daily run had to rewrite all 68 KB in order to tick one
checkbox and append one note.

So `TRACK.md` now keeps the notes for the **three most recent lessons** only. Older notes are
**not deleted and not duplicated** — they are already stored permanently and immutably in this
repository's git history, which is what git is for. This file is the index into that history.

## How to read an archived note

Fetch `TRACK.md` at the commit or blob listed below and read its comment block. With the GitHub
MCP tools:

```
get_file_contents(owner="aryanttripathi", repo="daily-learning-log",
                  path="TRACK.md", sha="<commit sha from the table>")
```

The `sha` argument accepts a commit SHA and returns the file as it stood at that commit. The blob
SHA in the last column is the stronger pointer: it identifies the exact bytes regardless of which
commit you reach them through.

## Archived ranges

| Lessons | Subject | Where the notes live | `TRACK.md` blob SHA at that commit |
|---|---|---|---|
| 13 – 24 | SQLite Internals | commit `724c164b9a3edfddd6993c458cbfa87c0c521805` (2026-10-12, the last commit before the first trim) | `e3aa5a6715fee568a4dfdead028d3d7624ca883f` |
| 25 | SQLite Internals | commit `5429a52018a46573b419ac99bf80d8fb3b0777b9` (2026-10-13, the last commit before the second trim) | `29cf5381c8b006917033a062727d9e6ce68b2cfb` |
| 26 | SQLite Internals | commit `ad3839f8be496cef3cf2db47fd4866cc7733b04d` (2026-10-14, the last commit before the third trim) | `68c8bcb936ccc25efac2076b649494b22a755cc2` |
| 27 | SQLite Internals | commit `8d6880a7dace31a374afc865d1556b9e3593caa4` (2026-10-15, the last commit before the fourth trim) | `e5e40ca598fb43fd2f26d090bc2bb10f576f4d5b` |
| 28 | SQLite Internals | commit `815ea964987e5764d01f917e944ca8a7eb5e2851` (2026-10-16, the last commit before the fifth trim) | `b11a76fdd03a5ee9ede69de38ec2585b029d0efd` |
| 29 | SQLite Internals | commit `aea7ae911690ab140bb7036d5588d06369ac10f7` (2026-10-17, the last commit before the sixth trim) | `1cb2703f12208654808b5bebde2299d80fddb95a` |
| 30 | SQLite Internals | commit `99aeb34381108985d51952987e71f9be978f235f` (2026-10-18, the last commit before the seventh trim) | `6c348998957d33c83ab6a3bc80755743eeec223a` |
| 31 – 33 | SQLite Internals | commit `5c5c0d88e5f5caf0c4b43c579dd76edeb64be644` (2026-10-19, the last commit before the syllabus was replaced by the PostgreSQL track) | `a21eb5940c9fd0dd7a3d47bce2e141b2628cd550` |
| **PG 01** | **PostgreSQL Internals** | commit `2fd269184b17e59f9091b82cb8934672fd32c8f7` (2026-10-23, the last commit before the eighth trim — the first of the PostgreSQL track) | `1feb2d21947ee646bf23f350c8ac84424fe87251` |
| **PG 02** | **PostgreSQL Internals** | commit `4b7ea85dcfcc830f71ca8ff16d7f26d84344b7a5` (2026-10-24, the last commit before the ninth trim) | `f47b8260ba0cab419ce2f4fc59a3e9c439ef6650` |
| **PG 03** | **PostgreSQL Internals** | commit `610357515b8e64cde4cb22cfaa3a34dd019456c4` (2026-10-25, the last commit before the tenth trim) | `91f09a63f3a86af94a3d519f09c62e5d69272be5` |
| **PG 04** | **PostgreSQL Internals** | commit `fffcd6bafd24c05eb83f0e2e46d0efab0597d880` (2026-10-26, the last commit before the eleventh trim) | `9fb0a858ff439a121aa72abf94e642c6fb06bc4b` |
| **PG 05** | **PostgreSQL Internals** | commit `5d3a81e3c7b92c6679f964fb9354092862b4c6a3` (2026-10-26, the last commit before the twelfth trim) | `54c92e672c6081eec9935d5f6f8514ad8520a7ed` |
| **PG 06** | **PostgreSQL Internals** | commit `01878f9bb5d3979a81c03925c2ae8219292b7dc6` (2026-10-28, the last commit before the thirteenth trim) | `df40aec01bc9a115a92e1d609bce79b1b8901e25` |
| **PG 07** | **PostgreSQL Internals** | commit `99a7b3b7eab710319907cbd11d5c311ebe61d4fa` (2026-10-29, the last commit before the fourteenth trim) | `937bfd19f68fdc42eebe5c83f1f636869629a59e` |

Notes on the rows:

- The **13 – 24** blob also contains the notes for lessons 25 and 26, which were retained in
  `TRACK.md` at the time of the first trim.
- The **25** blob contains the notes for lessons 25, 26 and 27. It is the more convenient place to
  read lesson 25's note, because the 13–24 blob is 68 KB.
- The **26** blob contains the notes for lessons 26, 27 and 28, and is the most convenient place to
  read lesson 26's note — the one on the `WhereLoop` cost model, the closed-form `SEARCH`/`SCAN`
  crossover, and the tie rule.
- The **27** blob contains the notes for lessons 27, 28 and 29, and is the most convenient place to
  read lesson 27's note — the one on `sqlite_stat1` arithmetic, the `sqlite_stat4` sample
  selection, and the `szTabRow` boundary where the scan-versus-index decision flips because of a
  column the query never mentions. It also carries the standing reminder that `WHERETRACE` needs
  **both** `SQLITE_DEBUG` and `SQLITE_ENABLE_WHERETRACE`, and that `sqlite3WhereTrace` is
  `SQLITE_PRIVATE`, so a harness must `#include "sqlite3.c"` to set it.
- The **28** blob contains the notes for lessons 28, 29 and 30, and is the most convenient place to
  read lesson 28's note — the one on the VDBE sorter: the PMA on-disk format decoded by hexdump, the
  `mxPmaSize` memory threshold and the spill boundary predicted to the single row, the 16-PMA cliff
  reachable by one page of `cache_size`, the `typeMask` fast-path gates, and `LIMIT -1` measuring
  2.74× slower than no `LIMIT` at all. Its unresolved sorter-mmap item was **closed by lesson 31**,
  whose note records the answer.
- The **29** blob contains the notes for lessons 29, 30 and 31, and is the most convenient place to
  read lesson 29's note — the one on savepoints and statement journals: a statement transaction
  *is* a savepoint at `db->nSavepoint + db->nStatement`; `WHERE_ONEPASS` shown to have **no
  effect** on statement journalling because `mayAbort` is never set for a `DELETE`; and the 64 KB
  spill threshold predicted in closed form and then hit to the single input row, 65,600 bytes in
  exactly two `xWrite` calls.
- The **30** blob contains the notes for lessons 30, 31 and 32, and is the most convenient place to
  read lesson 30's note — the one on virtual tables and `xBestIndex`: every output field's entry
  value read from inside the callback (`estimatedCost = 5e+98`, `estimatedRows = 25`, both reset on
  every call), the cost channel's `10·log₂` quantisation and the point above 2×10⁹ where
  `sqlite3LogEstFromDouble` stops computing a logarithm and reads the IEEE-754 exponent instead, the
  K+4 call-schedule bound, the `xDestroy == 0` trap that yields a misleading `no such module`, and
  the `sqlite_dbpage` truncation finding that **contradicts the documented** "any `ROLLBACK TO` will
  cancel the null-INSERT" rule, with the two differing statement sequences measured by file size.
- The **31 – 33** blob is the **final state of the SQLite `TRACK.md`**: it carries the notes for
  lessons 31, 32 and 33 *and* the complete 33-item syllabus with every lesson title as taught,
  all 33 boxes checked. It is therefore the right place to read lesson 31's mmap/allocator note
  (the 256-byte `unixFetch` end-of-file gate, the 4,368-byte cost of a cached 4,096-byte page, and
  `setupLookaside`'s three-branch arithmetic matched at nine configurations), lesson 32's verifier
  note (the two checkers, the min-heap byte-accounting proof, and the `quick_check` gap), and
  lesson 33's `VACUUM` note (the randomly-named `ATTACH`, the transfer optimisation, and the
  post-commit `xTruncate`). Read it before writing anything that refers back to the SQLite track.
- The **PG 01** blob is the first PostgreSQL-track trim, and it carries the notes for PostgreSQL
  lessons **01, 02 and 03** together with the handover that was written *to* lesson 04. It is
  therefore the right place to read lesson 01's note — the one on `RelFileLocator` and
  `RelFileNumber` being a typedef of `Oid` (so the compiler cannot catch an OID/filenode mix-up),
  `GetRelationPath()`'s nine `sprintf` branches and its deliberate return-by-value, the
  `forkNames[]` array with **no underscores in it** (the underscore comes from the `"%u_%s"`
  format string, which is why `pg_relation_size(rel,'fsm')` takes `'fsm'` and not `'_fsm'`),
  `md.c`'s on-disk segment invariant and the zero-length K+1st segment that `mdtruncate()` keeps
  on purpose, and the reusable ceilings: `BLCKSZ` 8192, `RELSEG_SIZE` 131072 blocks, 32,767
  segments per fork, and 32 TiB **minus 16 KiB** per fork because *two* `BlockNumber` values are
  reserved rather than one. Note that lessons 01 and 02 were written **without a cluster** and
  claim no observed figures; lesson 04 later measured and confirmed lesson 02's two outstanding
  predictions.
  **Read this one with lesson 08's note in hand:** lesson 08 (2026-10-27) turned the segment
  invariant and the zero-length K+1st segment from paper into measurements — a 142,858-block
  relation with segment 0 at exactly 1,073,741,824 bytes and a `.1` of 11,786 blocks, and a
  `VACUUM` truncation that left `.1` present at **0 bytes** and invisible to `mdnblocks`.
- The **PG 02** blob carries the notes for PostgreSQL lessons **02, 03 and 04** together with the
  handover that was written *to* lesson 05. It is therefore the right place to read lesson 02's
  note — the one on `PageHeaderData` by byte offset, the `SizeOfPageHeaderData = offsetof` trick,
  the page fingerprint `00 20 04 20` at offsets 16–19, `pd_flags`' four defined bits, `ItemIdData`'s
  three bitfields and why they cap `BLCKSZ` at 32 KB, the four `lp_flags` states,
  `PageAddItemExtended`'s four-way `PANIC` check and its three `WARNING` refusals, the 8160-vs-8164
  `MAXALIGN` derivation, `MaxHeapTuplesPerPage` = 291, the four free-space functions and why there
  are four, `compactify_tuples`' two modes, and the modified FNV-1a checksum in full — including
  the **checksum validity window** ("many or even most pages in shared buffers have invalid page
  checksums"), which is the fact most likely to be needed again.
  **Read this one before citing the fingerprint:** lesson 05 later showed by measurement that
  bytes 16–19 are identical on an FSM page, so the fingerprint identifies *zero special space*,
  not *heap*. Lesson 02's framing of it is narrower than it reads.
- The **PG 03** blob carries the notes for PostgreSQL lessons **03, 04 and 05** together with the
  handover that was written *to* lesson 06. It is therefore the right place to read lesson 03's
  note — the one on `HeapTupleHeaderData` by byte offset (0 `t_xmin`, 4 `t_xmax`, 8 `t_field3`,
  12 `t_ctid`, 18 `t_infomask2`, 20 `t_infomask`, 22 `t_hoff`, 23 `t_bits[]`, with
  `SizeofHeapTupleHeader` = 23 and `MinHeapTupleSize` = 24), the two nested unions and the
  27-versus-23 resolution of the documentation's Table 66.4, `ItemPointerData`'s six bytes and why
  `BlockIdData` is split into `bi_hi`/`bi_lo` at all, `t_ctid`'s **four** meanings with the
  chain-following rule (`xmin(next) == xmax(this)`), all sixteen `t_infomask` bits and all of
  `t_infomask2` tabulated against their owning lesson, the composite constants that are not bits
  (`HEAP_XMIN_FROZEN` = `0x0300`, `HEAP_MOVED` = `0xC000`), the `BITMAPLEN`/`t_hoff` step function
  measured at natts 8/9/16/72/73, and the null bitmap's LSB-first order with **1 meaning NOT
  NULL**. **Read this one before re-deriving anything about the tuple header**; lesson 06 cites its
  numbers without re-deriving them. It is also the blob that still contains lesson 04's incorrect
  `HEAP_FSM_CREATION_THRESHOLD` claim in the lesson 04 note — see the `PG 02` annotation above and
  the correction recorded in lesson 05's note.
- The **PG 04** blob carries the notes for PostgreSQL lessons **04, 05 and 06** together with the
  handover that was written *to* lesson 07. It is therefore the right place to read lesson 04's
  note — the one on attribute storage: the unifying thesis that `heap_compute_data_size()` is
  **both** the layout oracle and the TOAST loop's measuring stick (every one of the four TOAST
  passes is a `while (heap_compute_data_size(...) > maxDataLen)`), the six alignment macros on two
  axes (nominal vs conditional × datum vs pointer) with the `att_align_nominal` pointer-alignment
  "hack" comment that is sound only because `t_hoff` is `MAXALIGN`ed, `att_addlength_pointer`'s
  three cases proved exhaustive by its own `AssertMacro`, the varlena bit-layout comment quoted
  whole for both endiannesses, the **126-byte boundary** derived from `VARATT_CAN_MAKE_SHORT` and
  then measured with `pg_column_size` (124→125, 125→126, 126→127, 127→**131**), the pad-byte peek
  with its two-line four-case proof, and the measured column-order result (18 pad bytes in a
  54-byte payload). It also carries lesson 04's two **closed** lesson-02 debts: the heap page
  fingerprint `1c00 e01f 0020 0420` at bytes 12–19 and `MaxHeapTupleSize` 8160 pinned by the
  constant "row is too big: size 8168" message across `n = 8133..8140`.
  **Read this one with the `PG 02` and `PG 03` annotations in hand:** it is also where lesson 04's
  incorrect `HEAP_FSM_CREATION_THRESHOLD` / `FSM_LOCAL_MAP` handover claim lives, which lesson 05
  disproved (the patch was reverted in 2019 and never re-landed — no released PostgreSQL has a
  small-table FSM exemption).
- The **PG 05** blob carries the notes for PostgreSQL lessons **05, 06 and 07** together with the
  handover that was written *to* lesson 08. It is therefore the right place to read lesson 05's
  note — the one on the Free Space Map: `FSM_CATEGORIES` 256 and `FSM_CAT_STEP` 32 with the floor/
  ceil asymmetry between `fsm_space_avail_to_cat` and `fsm_space_needed_to_cat` and why category
  255 is reserved, `NonLeafNodesPerPage` 4095 against `SlotsPerFSMPage` **4069** as the ragged
  remainder, the reusable byte offsets (`fp_next_slot` at page offset 24, the page root at 28, leaf
  slot *k* at **4123 + k**), `fsm_logical_to_physical` fully evaluated so the fork is **not**
  level-major and the minimum fork is three blocks, `fsm_search_avail`'s expanding search triangle
  and the two self-correcting rebuild paths, `fp_next_slot` written under a *shared* lock on
  purpose, the measured finding that **the upper levels are stale until `VACUUM`** (the same
  `INSERT` landing on a different block before and after), the 32-byte ceiling trap measured to the
  byte (8 bytes of rounding costing one 8 KB page), `MaxHeapTuplesPerPage` = 291 first *observed*,
  and the honest non-result that the predicted 291-ceiling `PageGetHeapFreeSpace() == 0` trap
  **did not fire**. It is also where the correction to lesson 04's `HEAP_FSM_CREATION_THRESHOLD`
  claim is recorded in full, and where the "bytes 16–19 identify zero special space, not heap"
  correction to lesson 02 lives.
  **Two of its claims were narrowed by later runs:** lesson 06 showed that "a one-page table has no
  FSM fork" holds only *pre-*`VACUUM`, and lesson 08 answered its open question about whether the
  three-block FSM floor is a consequence of extension (it is `fsm_logical_to_physical`'s
  interleaving, not `smgrextend`).
- The **PG 06** blob carries the notes for PostgreSQL lessons **06, 07 and 08** together with the
  handover that was written *to* lesson 09. It is therefore the right place to read lesson 06's
  note — the one on the visibility map: all of `visibilitymapdefs.h` (`BITS_PER_HEAPBLOCK` 2,
  `VISIBILITYMAP_ALL_VISIBLE` `0x01`, `ALL_FROZEN` `0x02`, `VALID_BITS` `0x03`, and the `0b10` code
  point that is **unreachable by assertion at both ends**), the flat-array geometry (`MAPSIZE`
  8168, `HEAPBLOCKS_PER_BYTE` 4, `HEAPBLOCKS_PER_PAGE` **32672** = 255.25 MiB of heap per VM page,
  with no tree and therefore no ragged remainder, unlike lesson 05's 4069), the reusable address
  formula (VM page `b / 32672`, byte `24 + (b % 32672)/4`, bit shift `(b % 4) * 2`) **measured at
  the first VM page boundary crossed in this track** — heap block 32,671 landing on page offset
  **8191**, the last byte of the page, and block 32,672 on byte 24 of the next, on a 35,000-block
  table whose FSM fork independently confirmed lesson 05's arithmetic at 11 blocks; the
  two-places-one-truth argument for `PD_ALL_VISIBLE` and the pin-before-lock dance it forces; the
  five `visibilitymap_clear` call sites in `heapam.c` with their four `XLH_*_ALL_VISIBLE_CLEARED`
  flags and `MarkBufferDirty` rather than `MarkBufferDirtyHint`; **PG 16's own quoted argument for
  why setting a VM bit must be WAL-logged while the FSM pays nothing**, priced with `pg_waldump` at
  **59 bytes per heap page marked** (978 of 980 records at exactly 59 bytes, 5.65% of one 16 MB
  segment); `COPY FREEZE` setting both bits at load time with no `VACUUM`; the byte sequence
  `85 → 255 → 243` (`0x55` = `VISIBLE_MASK8`, then `0xff`, then `0xf3`) that proves the addressing
  scheme in three `get_byte` calls; and the **blind spot**: with the map truncated to nothing and
  861 heap pages still claiming `PD_ALL_VISIBLE`, `pg_check_visible` returned **zero rows**,
  because the checks detect over-claiming only. It also carries the seven-point 16-versus-master
  divergence table for `visibilitymap_set` (who locks, who logs, and the return value).
  **Read this one with lesson 07's note in hand:** lesson 06's headline "one cold update costs
  **59** heap fetches" was corrected by lesson 07 to **58** on a one-row-narrower table — the
  figure is `rows-on-the-dirtied-block + 1`, not a constant. Lesson 06's refinement of lesson 05
  stands: a one-block heap *does* get an FSM fork, but only after its first `VACUUM`.
  **And read it with lesson 09's note in hand** for the layer underneath: lesson 06's `RBM_ZERO_ON_ERROR`
  and `vm_extend` treated the buffer manager as a black box, which lesson 09 (2026-10-28) opened.
- The **PG 07** blob carries the notes for PostgreSQL lessons **07, 08 and 09** together with the
  handover that was written *to* lesson 10, and is the version of `TRACK.md` that still contains
  all three of those note sections. It is therefore the right place to read lesson 07's note — the
  one on HOT updates and page pruning: `LP_REDIRECT` and the line-pointer state machine, the
  819-byte prune heuristic located to within one row, the HOT-blocking matrix, the zero-growth
  steady state, the WAL comparison (166 vs 247 bytes/update), and the one-idle-transaction
  finding. Archived on **2026-10-29 by the lesson 10 run**.
- Lessons 01–12 of the SQLite track predate the note block; those lessons have no handover notes,
  only their documents.

After the 2026-10-19 run, `TRACK.md` describes a **new track — PostgreSQL Internals, 35 lessons** —
and after the 2026-10-28 run its note block holds the notes for PostgreSQL lessons **07, 08 and
09** plus the handover to lesson 10. Every SQLite note, and now PostgreSQL lessons 01–06's, lives
in git history, indexed by the table above.

## Procedure for the next trim

When `TRACK.md`'s note block holds more than three lessons' notes:

1. Note the current `TRACK.md` blob SHA and the current commit SHA **before** editing.
2. Remove the oldest notes from `TRACK.md`, keeping the three most recent.
3. Add a row here with the lesson range, that commit, and that blob SHA.

Never delete a note without adding its row here first.

**When a track ends and a new one begins**, the outgoing track's last three notes are the ones most
worth keeping readable: they carry the handoffs that the capstone has to close. Archive them as a
single range row — done for SQLite lessons 31–33 on 2026-10-19 — on the run that replaces the
syllabus, rather than trimming them one at a time into a syllabus they no longer describe. Note
that this row is also the only pointer to the outgoing track's full syllabus text, so the
**Completed Subjects** entry in `TRACK.md` must name it.

**Label rows by track once more than one track has been archived.** SQLite rows are bare numbers
for historical reasons; every PostgreSQL row is prefixed `PG`, because "lesson 01" is now ambiguous
on its own. A future third track should do the same.

**A note's claims are not automatically true later.** Lesson 04's handover asserted
`HEAP_FSM_CREATION_THRESHOLD = 4` and `FSM_LOCAL_MAP` with specific behaviour; lesson 05 found that
neither identifier exists in any branch — the patch that introduced them was reverted in 2019 and
never re-landed — and said so in the lesson. Lesson 07 did the same thing to a *measurement*:
lesson 06's "59 heap fetches after one cold update" turned out not to be a constant but
`rows-on-the-old-block + 1`, and reproduced as **58** on a table one row per page narrower. So treat
an archived or live note as *what that run believed*, not as established fact: it is working memory,
and one header fetch (or one re-measurement) is usually enough to check a specific claim before
building on it. When a later lesson overturns an earlier note, record the correction in the new note
**and** annotate the archived row, as the `PG 02`, `PG 03`, `PG 04`, `PG 05` and `PG 06` rows above
now are.

## What this file is not

It is not the record of what was taught. **The lesson documents are**, and they are indexed in the
root [`README.md`](README.md). These notes are working memory between runs: a fresh session reads
them to avoid re-teaching something or contradicting an earlier day. If you want to know what
lesson 18 covered, read lesson 18; read its note only if you need to know what lesson 18 decided
*not* to cover and why.
