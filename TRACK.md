<!--
track-state
subject: JanusGraph
mode: contributor-reading
started: 2026-10-30
lessons-total: 28
lessons-done: 4
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

- **04 — 2026-11-02** · [`2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md`](2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md) · read `InMemoryKeyColumnValueStore` L85–149, `InMemoryColumnValueStore` (whole), `SinglePageEntryBuffer` L36–101, `BufferPage` L39–175, plus `MemoryEntryList` and `BufferPageTest`. Four files, down from six, per session 03's size note. Six questions posed, none answered in the entry. Hands-on: `BufferPageTest#testGetIndex` and `#testLargeColumnKey`, breakpoints on `BufferPage.java:87` (the negated compare) and `:121` (the valPos sign switch), and a throwaway `dumpLayout()` with `offsetIndex` predicted first. Radar: PR #5018, #3864, #1128. **Session 03's `My Notes` were empty — streak 3; the entry drops the one-line notice for a plain statement of the problem and names the two options (fill the notes once, or switch the track to `lesson` mode).** **`My Notes` awaiting Aryant.**
- **03 — 2026-11-01** · [`2026/11/2026-11-01-janusgraph-store-manager-features/README.md`](2026/11/2026-11-01-janusgraph-store-manager-features/README.md) · read `KeyColumnValueStoreManager` (whole), `StoreManager` (whole), `StoreFeatures` L31–257, `StandardStoreFeatures` (`hasScan` L54, `Builder` from L180, the private ctor), `KeyColumnValueStore` L185–227 (the four `getKeys` overloads session 02 deferred), `Backend` L249–287 + L339–343, `InMemoryStoreManager` L64–72 + L134–135, plus `InMemoryStoreManagerTest` and `FeatureRequirementCondition`. Six questions posed, none answered in the entry. Hands-on: `InMemoryStoreManagerTest#testStoreCycle`, a throwaway `dumpFeatures()` with every value predicted first, and breakpoints on `Backend.java:261`/`:271` under `InMemoryGraphTest`. Radar: #2268, #3390, #1128. **Sessions 01 and 02 both left `My Notes` empty — streak 2; today's entry carries the one-line notice and has no `From Your Last Notes`.** **`My Notes` never filled.**
- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery` (whole), `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, plus `KeyColumnValueStoreTest` L862–938 and `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered in the entry. Hands-on: `InMemoryKeyColumnValueStoreTest#getSliceRespectsAllBoundsInclusionArguments` and `#getSliceRespectsColumnLimit`. Radar: #3864, #3816, PR #4086. **Session 01's `My Notes` were empty — one consecutive empty session; today's entry carries the one-line notice.** **`My Notes` never filled.**
- **01 — 2026-10-30** · [`2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md`](2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md) · read `data-model.md`, `BUILDING.md`, `CONTRIBUTING.md`, `InMemoryStoreManager`, `StandardStoreManager` L28–33, `InMemoryGraphTest`. Six questions posed, none answered in the entry. Hands-on: `WholeRowDeletionOptimizationTest`. Radar: #3013, #225, #1573. **`My Notes` left empty.**

## Syllabus

### Part I — Orientation and the storage contract (`diskstorage/`, `janusgraph-inmemory`)

- [x] 01 — Setup and the data model: fork, build `janusgraph-inmemory`, module map, and `docs/advanced-topics/data-model.md` (edgestore rows, relation columns)
- [x] 02 — `KeyColumnValueStore`: `getSlice`, `mutate`, `acquireLock`, plus `StaticBuffer`, `Entry`, `SliceQuery`, `KeySliceQuery`
- [x] 03 — `KeyColumnValueStoreManager` and `StoreFeatures`: how a backend advertises ordered scans, locking, batch mutation, and where core branches on them
- [x] 04 — `InMemoryKeyColumnValueStore`: answering a slice query over a sorted in-memory structure
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

- **Empty-notes streak: 3.** Sessions 01, 02 *and* 03 left `My Notes` as the untouched template. Session 04's entry
  stopped repeating the one-line notice and said the thing plainly: the mode withholds answers on purpose, so
  three empty sessions mean eighteen open questions and no evidence a file was opened. It named two options — fill the
  notes once, or switch this track to `lesson` mode — and committed session 05 to **settling** the question rather
  than asking a fourth time. **So: session 05 checks 04's notes. If they are filled, reset the streak to 0 and resume
  the mode as designed. If they are empty, that is four, and session 05 should flip `mode:` in this file's
  front-matter to `lesson`, say so in its first paragraph, and teach item 05 as a lesson** — same syllabus, same
  pinned commit, same depth bar, but answering its own questions and adding back `Where This Breaks Down`,
  `Further Study`, `Next Steps` and `Takeaways`. Do not ask Aryant first; the decision rule is recorded here and in
  session 04's entry, and an unattended run has no one to ask.
- **Session 04 answered two of session 03's six questions incidentally, as predicted** — but did *not* state the
  answers, only pointed at the two lines that carry them (question 1 → `InMemoryKeyColumnValueStore.java:138` vs
  `:143`; question 4 → `InMemoryStoreManager`'s own config). Both are still open on paper. Session 05 owns the
  remaining four (2, 3, 5, 6) only insofar as BerkeleyJE touches them; do not pre-empt.
- **Verified at the pinned commit, for later sessions to cite rather than re-derive.** `janusgraph-inmemory`'s main
  package has **11 files**, its test package **15**. The two-level structure: `InMemoryKeyColumnValueStore` holds a
  `ConcurrentSkipListMap<StaticBuffer, InMemoryColumnValueStore>` (`L94`, built `L99`); each row is one
  `InMemoryColumnValueStore` holding exactly one `SharedEntryBuffer` field (`L47`), starting as
  `SinglePageEntryBuffer` (`L51`) and promoted to `MultiPageEntryBuffer` once `numEntries() > DEF_PAGE_SIZE`
  (`L45` = **500**; promotion `L131-135`; never reversed, `L136-138`). A page is **two arrays** —
  `int[] offsetIndex` and `byte[] rawData` (`BufferPage.java:43-44`) — and each entry packs as
  `[valPos][column][value]` (`BufferPage.java:137-144`), `valPos` stored in **one byte when positive** and otherwise
  as a **negated int32** (`readValPos`, `L120-135`). `byteSize()` is `rawData.length + offsetIndex.length*4 + 16`
  (`L173-175`).
- **The read path, end to end, for sessions 19–22 and 27 to compare against:** `getSlice(KeySliceQuery)`
  `InMemoryKeyColumnValueStore.java:103-107` → `InMemoryColumnValueStore.java:68-76` (lock, then delegate) →
  `SinglePageEntryBuffer.java:53-78`. That last method is the whole thing: **two** `getIndex` binary searches
  (`L55`, `L59`), insertion-point decoding `(-x-1)` (`L56-58`, `L60-62`), `if (start < end)` else `EMPTY_LIST`
  (`L63`, `L76`), allocation sized to the **range** at `L64` while the limit is checked **inside** the loop at `L66`,
  and `getNoCopy(i)` at `L72` which **aliases the live `rawData`** rather than copying. `BufferPage.getIndex`
  (`L71-98`) compares the argument against a `rawData` window and **negates** the result (`L87`), licensed by the
  `checkArgument(column instanceof StaticArrayBuffer)` at `L84`.
- **Locking in this backend is config-gated, and that is load-bearing.** `InMemoryColumnValueStore.getLock(txh)`
  (`L146-160`) returns a lazily built `ReentrantLock` only when `STORAGE_TRANSACTIONAL` is true, and
  `NoLock.INSTANCE` otherwise. `mutate` (`L99`) is additionally `synchronized` on the method. Session 18 should note
  that `acquireLock` on this store throws `UnsupportedOperationException` outright
  (`InMemoryKeyColumnValueStore.java:132-134`), which is the other half of session 03's question 3.
- **Three PR candidates surfaced, none to be opened for him.** (1) `SinglePageEntryBuffer.java:38` builds its logger
  from `BufferPageUtils.class` instead of its own class, so its `L113` debug line is filed under the wrong logger —
  a one-line fix. (2) `InMemoryColumnValueStore.java:78-97` is a `private static class MemoryEntryList` byte-for-byte
  duplicating the top-level `MemoryEntryList.java` and never constructed in that file — dead code. (3) The real one:
  `numPages`, `numEntries`, `createFragmentationReport`, `quickDefragment` and `dumpTo` call `this.lock.lock()`
  **directly** from `InMemoryColumnValueStore.java:162` onward, bypassing `getLock(txh)`; since `lock` is assigned
  only on the transactional path, those are a latent NPE on a non-transactional store. Session 04's question 6 asks
  him to prove it reachable. Also noted without a line number: `createFragmentationReport` in
  `InMemoryKeyColumnValueStore` guards its body with `if (e.getValue() instanceof InMemoryColumnValueStore)`, which
  is unconditionally true given the map's value type. **Each is still unverified by Aryant**, as is session 02's
  `KeyColumnValueStoreTest.java:887-893` dead-loop candidate.
- **Issue search this run:** the in-memory slice path has **nothing open filed against it**. The issue search
  returned exactly two, both already on the radar (#3864, #1128). The one new item is **PR #5018** (draft, opened
  2026-10-05 by maintainer `porunov`, "Read what the step after a lookup by id reads along with the existence") — the
  only open PR matching `inmemory`, and a read-path batching change that lands on session 04's question 3. Repo-wide
  `good first issue` is **still exactly four** (#3127, #3126, #3013, #1573), unchanged for four runs; #3126/#3127 stay
  reserved for session 25. Keep re-running the search; stop re-listing the four.
- **Still deferred from Part I:** `MultiPageEntryBuffer`, `BufferPageUtils`, `BufferPage.merge` (both overloads,
  `L177` onward) and the fragmentation/defragmentation machinery
  (`InMemoryKeyColumnValueStoreFragmentationReport`, `SharedEntryBufferFragmentationReport`). The paging *write* path
  is the natural companion to session 16; the fragmentation report is operational and may never get a session.
  `InMemoryKeyColumnValueStore.RowIterator` goes with session 06.
- **Entry size, third data point.** Session 04 came in at **~15 KB** against the mode's 5–9 KB budget, after one
  deliberate trimming pass, and *with* the file count cut from six to four as session 03 advised. So fewer files was
  **not** the lever. The actual floor is structural: six long pinned permalink definitions and the Sources block are
  ~2.3 KB on their own, and the reading table, Map, six questions, Hands-On and Radar are ~2 KB each by format.
  **Conclusion: the 5–9 KB budget is not achievable with the mode's required ten sections at this depth bar.** Either
  the budget moves to ~12–15 KB or a section goes. Do not spend another run trimming prose to hit it.

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
