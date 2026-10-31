<!--
track-state
subject: JanusGraph
mode: contributor-reading
started: 2026-10-30
lessons-total: 28
lessons-done: 2
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase (Contributor Reading)

Sequential reading of the JanusGraph source, one session per day, each building on the last.
**This track is different from SQLite and PostgreSQL.** Those were lessons *about* a system. This
one is a guided **reading session** in a codebase Aryant intends to contribute to before the
January 2027 LFX Mentorship applications open. The generated part of each entry is a short guide;
the other half — `## My Notes` — is written by Aryant after reading. A session whose `My Notes`
stays empty did not count.

**Why bottom-up.** JanusGraph is a translation layer: a property graph mapped onto a
BigTable-style key → sorted-columns store, and Gremlin traversals turned into slice queries over
those rows. The syllabus follows that layering upward: storage contract → graph ⇄ bytes → write
path → read path → Gremlin integration → schema, indexes and production backends. Classes that
are reference material rather than reading material (`GraphDatabaseConfiguration`, most of
`StandardJanusGraphTx`) are visited only for the slices a lesson needs.

**Pinned source.** All line numbers and permalinks use commit `a59ae8d8` (2026-10-28) on
`JanusGraph/janusgraph` `master`. Re-pin only at a Part boundary, and record the new commit here.

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.

## Session Log

Newest first. The next run reads the most recent entry's `## My Notes` before writing.

- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery` (whole), `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, plus `KeyColumnValueStoreTest` L862–938 and `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered in the entry. Hands-on: `InMemoryKeyColumnValueStoreTest#getSliceRespectsAllBoundsInclusionArguments` and `#getSliceRespectsColumnLimit`. Radar: #3864, #3816, PR #4086. **Session 01's `My Notes` were empty — one consecutive empty session; today's entry carries the one-line notice.** **`My Notes` awaiting Aryant.**
- **01 — 2026-10-30** · [`2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md`](2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md) · read `data-model.md`, `BUILDING.md`, `CONTRIBUTING.md`, `InMemoryStoreManager`, `StandardStoreManager` L28–33, `InMemoryGraphTest`. Six questions posed, none answered in the entry. Hands-on: `WholeRowDeletionOptimizationTest`. Radar: #3013, #225, #1573. **`My Notes` left empty.**

## Syllabus

### Part I — Orientation and the storage contract (`diskstorage/`, `janusgraph-inmemory`)

- [x] 01 — Setup and the data model: fork, build `janusgraph-inmemory`, module map, and `docs/advanced-topics/data-model.md` (edgestore rows, relation columns)
- [x] 02 — `KeyColumnValueStore`: `getSlice`, `mutate`, `acquireLock`, plus `StaticBuffer`, `Entry`, `SliceQuery`, `KeySliceQuery`
- [ ] 03 — `KeyColumnValueStoreManager` and `StoreFeatures`: how a backend advertises ordered scans, locking, batch mutation, and where core branches on them
- [ ] 04 — `InMemoryKeyColumnValueStore`: answering a slice query over a sorted in-memory structure
- [ ] 05 — `OrderedKeyValueStoreManagerAdapter` and `BerkeleyJEKeyValueStore`: faking key-column-value on an ordered key-value store
- [ ] 06 — The test harness: `JanusGraphTest`, `InMemoryGraphTest`, backend test suites, and how a PR's test is expected to look
- [ ] 07 — `Backend` and `BackendTransaction`: the stores that get opened (edgestore, graphindex, system_properties, ids, logs) and the cache/locker wrappers

### Part II — Graph ⇄ bytes (`graphdb/idmanagement`, `graphdb/database`)

- [ ] 08 — `IDManager`: vertex ID bit layout, partition bits, vertex-type suffixes, and how row keys are derived
- [ ] 09 — `VertexIDAssigner` and the ID authority: block allocation from the ids store without per-insert coordination
- [ ] 10 — `EdgeSerializer.writeRelation`: the column layout for edges and properties, direction, sort keys
- [ ] 11 — `EdgeSerializer.parseRelation` and `RelationCache`: decoding columns back into relations
- [ ] 12 — `StandardSerializer`: attribute encodings, order-preserving serializers, and custom types
- [ ] 13 — System types and schema storage: schema vertices, `BaseKey`/`BaseLabel`, and how a property-key definition is persisted

### Part III — The write path (`core`, `graphdb/transaction`, `graphdb/database`, `diskstorage/locking`)

- [ ] 14 — `JanusGraphFactory.open` → `GraphDatabaseConfiguration` → `StandardJanusGraph` constructor: local vs global (fixed/masked) config
- [ ] 15 — `StandardJanusGraphTx`: `addVertex`, `addEdge`, property writes, added/deleted relation containers, and the vertex cache
- [ ] 16 — `StandardJanusGraph.commit` and `prepareCommit`: deletes, additions, mutations, and the order they are emitted in
- [ ] 17 — Composite indexes on write: `IndexSerializer` and the graphindex entries
- [ ] 18 — Locking: `ExpectedValueCheckingStore` and `ConsistentKeyLocker`, and how uniqueness survives an eventually consistent store

### Part IV — The read path (`graphdb/vertices`, `graphdb/query`)

- [ ] 19 — `StandardVertex`/`CacheVertex`, lazy adjacency loading, and the transaction-level and database-level caches
- [ ] 20 — `BasicVertexCentricQueryBuilder`: `v.edges(...)` → `SliceQuery`, and vertex-centric indexes
- [ ] 21 — `GraphCentricQueryBuilder`: index selection between composite, mixed, and full scan
- [ ] 22 — `QueryProcessor`, limits, sub-query merging, and query profiling

### Part V — Gremlin integration (`graphdb/tinkerpop`)

- [ ] 23 — The strategy layer: `JanusGraphLocalQueryOptimizerStrategy` and the other optimizer strategies
- [ ] 24 — `JanusGraphStep` and `JanusGraphVertexStep`: folding `has`/`limit`/`order` into storage queries, and multi-query batching

### Part VI — Schema, indexes, and production backends

- [ ] 25 — `ManagementSystem` and the index lifecycle: INSTALLED → REGISTERED → ENABLED, and reindex jobs
- [ ] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run.** No reordering, no splits. The 28 items stand as written on 2026-10-30.

- **Session 02 confirmed, at the pinned commit:** the column interval is half-open, start inclusive
  (`SliceQuery.java:64-68`) and end exclusive (`L73-L77`), encoded in `contains` at `L136-L138`;
  `mutate`'s deletions-before-additions guarantee is `KeyColumnValueStore.java:107-110`; lock
  verification is deferred to the first `mutate` of a transaction, `L139-L144`; `getMultiSlices`'
  unoptimised default body is `L91-L101`. Sessions 04, 05 and 27 should cite these rather than
  re-deriving them.
- **`Entry extends StaticBuffer`** (`Entry.java:27`) with `getValuePosition()` (`L29`) is the fact
  session 10 and 11 need: one entry is one byte buffer, column then value, split at that offset.
  Session 02 left the *physical layout* as question 5 rather than stating it.
- **Session 02 deliberately did not open** the four `getKeys` overloads, `KeyIterator`,
  `KeyRangeQuery`, `MultiSlicesQuery` or `KeySlicesIterator` (`KeyColumnValueStore.java:185` down).
  Session 03 owns those, alongside the `orderedScan`/`unorderedScan` flags that gate them.
- **A real tiny-PR candidate was found and left for Aryant to verify:**
  `janusgraph-backend-testutils/.../KeyColumnValueStoreTest.java:887-893` — a `for` loop over the
  slice result whose entire body is `if (!src.equals(dst)) { int x = 1; }`, a leftover breakpoint
  hook, immediately before the `assertEquals` that does the work. Deleting it is a two-line,
  zero-risk, DCO-signed first commit. **Do not open it for him**; it is in his `My Notes` template
  under *Possible issue / PR idea*.
- **Issue search this run:** the two live issues on today's area are **#3864** (getMultiSlices for
  HBase) and **#3816** (multi-range slice queries for CQL) — both open since June 2023, no assignee,
  both `kind/performance`. Repo-wide `good first issue` is still exactly four: #3013, #3126, #3127,
  #1573; #3126/#3127 stay reserved for session 25. Re-run the search each session.
- `janusgraph-inmemory`'s paging machinery (`MultiPageEntryBuffer`, `BufferPage`, `BufferPageUtils`,
  `SinglePageEntryBuffer`, `InMemoryColumnValueStore`) remains deferred to session 04.
- **Empty-notes streak: 1.** Session 01's `My Notes` were never filled. Session 03 must re-check
  session 02's notes and increment or reset the count.

## Paused Subjects

### PostgreSQL Internals — paused 2026-10-30 after lesson 10 of 35

Paused at Aryant's request to make JanusGraph the active track. **Nothing was dropped.** The full
PostgreSQL track state — syllabus with lessons 01–10 checked, the handover note block, environment
recipes, and archive pointers — is preserved verbatim in
[`tracks/postgresql.md`](tracks/postgresql.md). When the JanusGraph track completes, that file
becomes `TRACK.md` again and the next run resumes at **lesson 11 — the heavyweight lock manager**.

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
