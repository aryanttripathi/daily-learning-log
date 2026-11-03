<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 5
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open.

**Mode: `lesson`, switched 2026-11-03.** Sessions 01–04 ran as `contributor-reading`: a short
reading guide per day, with four to six questions deliberately left unanswered and the `## My
Notes` half written by Aryant. All four shipped with that half as the untouched template — 24
questions posed, none answered, no evidence a file was opened. Session 04's entry stated the
problem plainly, named the two honest options, and recorded a decision rule here for session 05 to
execute rather than ask a fifth time. **Session 05 executed it.** From lesson 05 on, entries answer
their own questions and carry `Where This Breaks Down`, `Further Study`, `Next Steps` and
`Takeaways` — same syllabus, same pinned commit, same depth bar. The carried questions are tracked
under each entry's `Next Steps` and settled as the syllabus reaches them. Flipping back is one line
of the front-matter above; nothing else in this file depends on the mode.

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
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

## Session Log

Newest first.

- **05 — 2026-11-03** · [`2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md`](2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md) · **first entry in `lesson` mode.** Read `OrderedKeyValueStoreManagerAdapter` (whole), `OrderedKeyValueStoreAdapter` (whole, with exact lines for `concatenate` L250-264, `convertQuery` L231-240, the decode block L215-297, `KeyIteratorImpl` L299+), `BerkeleyJEKeyValueStore` (whole; `getSlice` L212-281, locking L146-208), `BerkeleyJEStoreManager` L120-145 / L200-262 / L265-300, `Backend` L518-546 and L258-263, `KVQuery`, `KeySelector`, `BaseKeyColumnValueAdapter`, `StandardStoreFeatures` L183-187, `Query` L24, plus both `BerkeleyFixedLengthKCVSTest` and `BerkeleyVariableLengthKCVSTest`. Taught the fixed-vs-variable key-length split, why fixed needs no post-filter and variable does, the three byte layouts, the cursor walk, the batch that isn't, and BerkeleyJE's own two-tier locking. Hands-on: the two KCVS tests, breakpoints on `OrderedKeyValueStoreAdapter.java:254`/`:289` and `BerkeleyJEKeyValueStore.java:258`, plus a scratch test proving the `getLength`-throws vs `adjustToLength`-coerces asymmetry. Daily Diff: the **2026-11-02** edition, 49 stories, fully readable this run — deep dive on Neumann's *The Benchmarking Crisis*.
- **04 — 2026-11-02** · [`2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md`](2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md) · read `InMemoryKeyColumnValueStore` L85–149, `InMemoryColumnValueStore` (whole), `SinglePageEntryBuffer` L36–101, `BufferPage` L39–175, plus `MemoryEntryList` and `BufferPageTest`. Six questions posed, none answered in the entry. Hands-on: `BufferPageTest#testGetIndex` and `#testLargeColumnKey`, breakpoints on `BufferPage.java:87` and `:121`, and a throwaway `dumpLayout()`. Radar: PR #5018, #3864, #1128. **`My Notes` never filled — the fourth consecutive empty session, which triggered the mode switch.**
- **03 — 2026-11-01** · [`2026/11/2026-11-01-janusgraph-store-manager-features/README.md`](2026/11/2026-11-01-janusgraph-store-manager-features/README.md) · read `KeyColumnValueStoreManager` (whole), `StoreManager` (whole), `StoreFeatures` L31–257, `StandardStoreFeatures`, `KeyColumnValueStore` L185–227, `Backend` L249–287 + L339–343, `InMemoryStoreManager` L64–72 + L134–135, plus `InMemoryStoreManagerTest` and `FeatureRequirementCondition`. Six questions posed, none answered. Radar: #2268, #3390, #1128. **`My Notes` never filled.**
- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery` (whole), `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, plus `KeyColumnValueStoreTest` L862–938 and `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered. Radar: #3864, #3816, PR #4086. **`My Notes` never filled.**
- **01 — 2026-10-30** · [`2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md`](2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md) · read `data-model.md`, `BUILDING.md`, `CONTRIBUTING.md`, `InMemoryStoreManager`, `StandardStoreManager` L28–33, `InMemoryGraphTest`. Six questions posed, none answered. Radar: #3013, #225, #1573. **`My Notes` left empty.**

## Syllabus

### Part I — Orientation and the storage contract (`diskstorage/`, `janusgraph-inmemory`)

- [x] 01 — Setup and the data model: fork, build `janusgraph-inmemory`, module map, and `docs/advanced-topics/data-model.md` (edgestore rows, relation columns)
- [x] 02 — `KeyColumnValueStore`: `getSlice`, `mutate`, `acquireLock`, plus `StaticBuffer`, `Entry`, `SliceQuery`, `KeySliceQuery`
- [x] 03 — `KeyColumnValueStoreManager` and `StoreFeatures`: how a backend advertises ordered scans, locking, batch mutation, and where core branches on them
- [x] 04 — `InMemoryKeyColumnValueStore`: answering a slice query over a sorted in-memory structure
- [x] 05 — `OrderedKeyValueStoreManagerAdapter` and `BerkeleyJEKeyValueStore`: faking key-column-value on an ordered key-value store
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

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter, exactly as session 04's rule specified. Lesson 06
  and onward should write in `lesson` mode without re-litigating the decision, without a notice at the
  top, and without nagging. If Aryant asks for reading sessions back, flip `mode:` and the entries
  resume the other shape from the next run; nothing else in this file changes.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01–04 keep their templates. They are his
  to fill at any time, and a filled one is still worth reading before writing — if a future run finds
  content in any of them, answer those questions in that run's entry regardless of mode.
- **Verified at the pinned commit, for later lessons to cite rather than re-derive.**
  `janusgraph-core/.../keycolumnvalue/keyvalue/` has **12 files**; `janusgraph-berkeleyje/src/main/.../berkeleyje/`
  has **4** (`BerkeleyJEKeyValueStore` 14,249 B, `BerkeleyJEStoreManager` 17,460 B, `BerkeleyJETx` 6,524 B,
  `BerkeleyJELocks` 2,856 B) and its test package **6**. The encoding, exactly: `concatenate(front, end,
  appendLength)` at `OrderedKeyValueStoreAdapter.java:250-264`, with `addKeyLength = !hasFixedKeyLength()
  && appendLength` at `L251`. `concatenate(front,end)` (`L242-244`, `appendLength=true`) builds **stored
  record keys and lock keys**; `concatenatePrefix` (`L246-248`, `false`) builds **query bounds** — the
  asymmetry is required, because a bound carrying the length suffix would stop being a prefix. Fixed:
  `key[L] ‖ column`. Variable: `key[k] ‖ column ‖ be16(k)`, suffix written at `L259-262`, read back by
  `getShort(len-2)` at `L278`, `variableKeyLengthSize = 2` (`L68`), cap `maxVariableKeyLength =
  Short.MAX_VALUE` (`L67`) enforced with a strict `<` at `L221`.
- **The theorem the adapter rests on, for lesson 08 and lesson 27 to reuse.** With every key exactly
  the same width, lexicographic order on `key ‖ column` **is** lexicographic order on the pair
  `(key, column)`, so a prefix range scan is exact and `convertQuery` installs `Predicates.alwaysTrue()`
  (`L232`). With variable-width keys that collapses — a key that is a byte-prefix of another key leaks
  into its range — so the filter becomes `equalKey` (`L234`, defined `L287-292`), which recovers the true
  width from the trailing short and rejects on length mismatch first (`L289`).
  `BerkeleyVariableLengthKCVSTest.retrieveBackendAndGetKeyValue` is that leak written down as a test
  (`"hello"` vs `"hel"`, both inside the byte range `68 65 6C 00 .. 68 65 6C FF`, one discarded by
  `equalKey`); its `//Errors out here without fix` comment is the fossil. `BerkeleyFixedLengthKCVSTest`
  differs by one expression: `ImmutableMap.of(storeName, 8)` at its `L29`.
- **Where the `8` comes from, for lesson 07 and lesson 08.** `Backend.java:537-542` builds a
  three-entry map by **name**: `EDGESTORE_NAME`, `EDGESTORE_NAME + LOCK_STORE_SUFFIX`, and
  `storageConfig.get(IDS_STORE_NAME)`, each `8` = `sizeof(long)`. **`graphindex`, `system_properties`
  and the logs are variable-length and pay the filter cost.** The same branch (`L525-536`) rejects
  `ALLOW_CUSTOM_VERTEX_ID_TYPES` outright on an ordered-key-value backend, closing the manager first
  because BerkeleyJE has already opened its environment.
- **Three different implementations of one contract, now on record.** `[sliceStart, sliceEnd)` is:
  a pair of binary-search insertion points in `SinglePageEntryBuffer.getSlice` (session 04); a single
  `key.compareTo(keyEnd) >= 0` per record at `BerkeleyJEKeyValueStore.java:258`; and a predicate
  (`columnInRange`, `OrderedKeyValueStoreAdapter.java:294-297`) for the key-range scan. The contract is
  in `KeyColumnValueStore`'s javadoc; **none of the enforcement is.** Lesson 27 should add the CQL
  backend as a fourth data point.
- **Feature flags that are load-bearing and unasserted.** BerkeleyJE's builder
  (`BerkeleyJEStoreManager.java:129-141`) sets `orderedScan`, `transactional`, `keyConsistent`,
  `locking(true)`, `keyOrdered(true)`, `scanTxConfig(READ_UNCOMMITTED)`, `supportsInterruption(false)`,
  `cellTTL(true)`, `optimisticLocking(false)` — and **never** `multiQuery` or `batchMutation`, which are
  plain uninitialized `boolean` fields in `StandardStoreFeatures.Builder` (`L183-187`) and therefore
  `false`. Consequences: `hasBatchMutation()` false makes `Backend.java:261-263` set
  `bufferSize = Integer.MAX_VALUE`, disabling the mutation buffer rather than sizing it; and
  `hasMultiQuery()` false is the **only** thing keeping `BerkeleyJEKeyValueStore.getSlices`'s
  `UnsupportedOperationException` (`L282-285`) out of the read path, reached from
  `OrderedKeyValueStoreAdapter.java:97`. Nothing asserts either flag.
- **Locking on this backend, for lesson 18 to contrast against `ConsistentKeyLocker`.** Because
  `locking(true)` is declared, session 03's inverted `Backend` branch does **not** wrap this store in
  `ExpectedValueCheckingStore`. BerkeleyJE brings two tiers of its own: a JVM-local
  `ConcurrentMap<StaticBuffer, BerkeleyJETx>` (`BerkeleyJEKeyValueStore.java:71`, from
  `getLockHolders` at `BerkeleyJEStoreManager.java:297-299`) whose `putIfAbsent` throws
  `PermanentLockingException` **immediately** on contention rather than waiting (`L171-179`, with the
  comment at `L175-176` explaining it must never block while holding JE record locks); and
  `checkExpectedValue` (`L191-208`) re-reading under `LockMode.RMW` (`L196`) on the transaction's first
  mutation. With `storage.transactions=false`, `acquireLock` degrades to a `log.warn` and a `return`
  (`L159-162`) — **silent loss of locking** — and `beginTransaction` refuses a non-single-threaded config
  outright (`BerkeleyJEStoreManager.java:217-221`).
- **Why the duplication is affordable**, and the single best fact from this lesson:
  `dbConfig.setKeyPrefixing(true)` at `BerkeleyJEStoreManager.java:252`. All columns of a row share the
  row-key prefix and land adjacently in the B-tree, so JE's leaf-node prefix compression absorbs the
  repetition the adapter creates. `setTransactional(transactional)` is `L251`; `batchLoading` adds
  `setDeferredWrite(true)` at `L254-256`.
- **One real open issue sits exactly on this code, and it is a contained first contribution.**
  [#4781](https://github.com/JanusGraph/janusgraph/issues/4781) — "storage.transactions=false with
  berkeleyje backend prevents any write operation", `kind/bug/possible`, **unassigned**, opened
  2025-03-21, untouched since. It lands on `BerkeleyJEKeyValueStore.java:159-162` and
  `BerkeleyJEStoreManager.java:217-221`. The BerkeleyJE/ordered-key-value issue search this run returned
  **exactly two** open issues: #4781 and [#1670](https://github.com/JanusGraph/janusgraph/issues/1670)
  ("Can the storage backends be BerkeleyDB not BerkeleyDB JE?", stale since 2021). Lesson 06 reads the
  test harness and is the right place to decide what a reproduction test for #4781 should look like.
- **Carried open questions from sessions 01–04 (24 total, none answered by Aryant).** Lesson 05 closed
  several by contrast rather than by assertion: the half-open interval (02's and 04's Q2) now has three
  distinct implementations on record; `getKeys(SliceQuery)` vs `getKeys(KeyRangeQuery)` (03's Q1, 04's Q1)
  diverge far harder here than in the in-memory store — the ordered-key backend **throws** on both the
  `SliceQuery` and `MultiSlicesQuery` overloads (`OrderedKeyValueStoreAdapter.java:151-159`), supporting
  strictly *fewer* scan shapes than the unordered in-memory store, which is the opposite of the intuition.
  Still genuinely open and owned by **lesson 07**: `MemoryEntryList` sized to the range while the limit is
  checked inside the loop; whether `getNoCopy`'s aliasing of live `rawData` is observable across a
  `mutate`; and the `this.lock.lock()`-bypasses-`getLock(txh)` NPE candidate in
  `InMemoryColumnValueStore.java:162` onward.
- **Still deferred from Part I:** `MultiPageEntryBuffer`, `BufferPageUtils`, `BufferPage.merge` (both
  overloads) and the fragmentation/defragmentation machinery; `InMemoryKeyColumnValueStore.RowIterator`
  goes with lesson 06. New to the deferred list from lesson 05: `BerkeleyJETx` (6,524 B — cursor
  bookkeeping, `prepareForMutations`, cache/lock modes) and `BerkeleyJELocks`, both of which belong with
  lesson 18 rather than here; and `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299` onward), the
  row-grouping iterator with the shared mutable `current` field and the per-key limit at `L381`, which
  goes with lesson 06's scan tests.
- **Entry size.** The 5–9 KB budget was a `contributor-reading` constraint and no longer applies.
  Lesson 05 came in at ~35 KB, in line with the SQLite and PostgreSQL lessons, which is the right
  comparison set now. Do not trim toward the old number.
- **tdd.cat endpoints: stop looking for the leader.** Three runs' worth of conclusions inverted again on
  2026-11-03 — the front page's "always one day behind the maximum" offset broke (front page and archive
  both at 11-02, which *was* the maximum), and RSS went from leading to lagging by one edition. The
  correct procedure is the one the task already states: fetch the front page, digest whatever edition it
  serves, say which one it is, cross-check `/archive/` for the maximum, treat RSS as a third opinion.
  Future digests should not spend space re-deriving an ordering.

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
