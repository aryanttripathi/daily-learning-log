<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 6
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

- **06 — 2026-11-04** · [`2026/11/2026-11-04-janusgraph-test-harness/README.md`](2026/11/2026-11-04-janusgraph-test-harness/README.md) · read the harness: `JanusGraphBaseTest` (whole; lifecycle L151-199, `clearGraph`/`getBackend` L132-145, `setupConfig`/`clopen` L213-242, `evaluateQuery` ×7 L589-750, `executeScanJob` L752-762), `JanusGraphTest` (whole file analysed — 12,258 lines, class at L312, 222 `@Test` + 7 `@ParameterizedTest` + 5 `@Tag(BRITTLE_TESTS)` at L3098/L3179/L10156/L10192/L10272, ten banner sections, `simpleLogTest` L8270-8282), `KeyColumnValueStoreTest` (whole; lifecycle L94-141, abstract hook L102, fixture constants L82-92, the seven `@FeatureFlag` sites, `scanTest` L445-462, `testOrderedGetKeysRespectsKeyLimit` L491-513), `AbstractKCVSTest`, `JanusGraphBaseStoreFeaturesTest`, `TestCategory`, `StorageSetup`, `TestGraphConfigs`, `FeatureFlag`, `FeatureRequirementCondition`, `JanusGraphFeature`, `RetryingTestExecutionExtension`, `InMemoryGraphTest`, `InMemoryStoreManager` L64-70, `BerkeleyFixedLengthKCVSTest`, `BerkeleyVariableLengthKCVSTest`, `BerkeleyKeyValueTest`, `BerkeleyGraphConcurrentTest`, `CQLGraphTest` L239-290/L390/L481, root `pom.xml` (surefire L404-461, `test.excluded.groups` L102, profiles L1900-1972), `janusgraph-test/pom.xml`, `CONTRIBUTING.md`, `.github/PULL_REQUEST_TEMPLATE.md`. Taught the src/main-not-src/test inversion, the one-abstract-hook-per-level contract, the **four** opt-out idioms and their four different reporting semantics, `@FeatureFlag`'s 4-of-24 coverage and its post-`@BeforeEach` ordering, `evaluateQuery`'s profiler assertions, and the build-level exclusions. Hands-on: read the Surefire XML rather than the console — 31 bare `<skipped/>` on `InMemoryGraphTest`, zero trace of the 5 `BRITTLE` methods, `testConcurrentGetSlice` as a 0 ms **pass**, and `<skipped message="Database doesn't support unordered scan."/>` on BerkeleyJE. Daily Diff: the **2026-11-03** edition, 52 stories, front page and archive agreed and were current — deep dive on Spiral's *Statistics are aggregates* plus the Vortex `FileStatistics` layout.
- **05 — 2026-11-03** · [`2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md`](2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md) · **first entry in `lesson` mode.** Read `OrderedKeyValueStoreManagerAdapter` (whole), `OrderedKeyValueStoreAdapter` (whole, with exact lines for `concatenate` L250-264, `convertQuery` L231-240, the decode block L215-297, `KeyIteratorImpl` L299+), `BerkeleyJEKeyValueStore` (whole; `getSlice` L212-281, locking L146-208), `BerkeleyJEStoreManager` L120-145 / L200-262 / L265-300, `Backend` L518-546 and L258-263, `KVQuery`, `KeySelector`, `BaseKeyColumnValueAdapter`, `StandardStoreFeatures` L183-187, `Query` L24, plus both `BerkeleyFixedLengthKCVSTest` and `BerkeleyVariableLengthKCVSTest`. Taught the fixed-vs-variable key-length split, why fixed needs no post-filter and variable does, the three byte layouts, the cursor walk, the batch that isn't, and BerkeleyJE's own two-tier locking. Daily Diff: the 2026-11-02 edition, 49 stories — deep dive on Neumann's *The Benchmarking Crisis*.
- **04 — 2026-11-02** · [`2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md`](2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md) · read `InMemoryKeyColumnValueStore` L85–149, `InMemoryColumnValueStore` (whole), `SinglePageEntryBuffer` L36–101, `BufferPage` L39–175, plus `MemoryEntryList` and `BufferPageTest`. Six questions posed, none answered in the entry. Radar: PR #5018, #3864, #1128. **`My Notes` never filled — the fourth consecutive empty session, which triggered the mode switch.**
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
- [x] 06 — The test harness: `JanusGraphTest`, `InMemoryGraphTest`, backend test suites, and how a PR's test is expected to look
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
Lesson 06 found no reason to move anything: the harness turned out to be exactly where the Part I
findings needed it to be, and the three questions it would answer are already owned by lesson 07.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Lesson 07 and onward write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01–04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.

### Verified this run, for lesson 07 and the query lessons to cite rather than re-derive

- **Where the test code lives.** The abstract suites are in **`janusgraph-backend-testutils/src/main/java`**
  — `src/main`, not `src/test` — and reach backend modules as an ordinary **compile-scope** jar, with no
  `<type>test-jar</type>` anywhere. `janusgraph-test/pom.xml` declares exactly three compile-scope
  JanusGraph deps (`janusgraph-core`, `janusgraph-inmemory`, `janusgraph-backend-testutils`) and sets
  `forkCount=4`. `janusgraph-test/src/main/java/org/janusgraph/` contains **only** `diskstorage/cache/`;
  everything else is under `src/test`. Consequence worth remembering: the protected surface of
  `KeyColumnValueStoreTest` is public API for out-of-tree backends.
- **Two base classes, one abstract hook each.** `JanusGraphBaseTest` (`L108`,
  `implements JanusGraphBaseStoreFeaturesTest`) requires `getConfiguration()` (`L126`);
  `KeyColumnValueStoreTest` (`L80`, `extends AbstractKCVSTest implements JanusGraphBaseStoreFeaturesTest`)
  requires `openStorageManager()` (`L102`). `JanusGraphBaseStoreFeaturesTest` is a one-method interface
  (`StoreFeatures getStoreFeatures()`) and exists solely so a JUnit extension can reach features without
  knowing the base class. Both `setUp`s use the same throwaway-manager-then-real-manager pattern so
  `clearStorage()` runs against a manager that is immediately discarded.
- **Two constants pinned by the harness that lesson 07 will care about.** `JanusGraphBaseTest.getBackend`
  (`L136-145`) rewrites `LOCK_LOCAL_MEDIATOR_GROUP` to `"tmp"` and `UNIQUE_INSTANCE_ID` to `"inst"` on
  every graph-level test, so **no test in the tree exercises two mediator groups against one store**
  unless it builds its own `Backend`. And `StorageSetup.getInMemoryConfiguration()` sets
  `IDAUTHORITY_WAIT = Duration.ZERO` and `DROP_ON_CLEAR = false`, so everything that passes in memory
  passes with ID-authority coordination latency removed (relevant to lesson 09).
- **`AbstractKCVSTest`** is 37 lines and supplies one decision: `times = TimestampProviders.MICRO`.
  Every store-level test runs at microsecond resolution. `getConsistentTxConfig(manager)` reading
  `manager.getFeatures().getKeyConsistentTxConfig()` is the **only** place the harness consults the
  feature flags mechanically outside `@FeatureFlag`.
- **`KeyColumnValueStoreTest` fixture, exactly:** `TRIALS = 5000` (`L82`), `numKeys = 500` (`L85`),
  `numColumns = 50` (`L86`) → **25,000 cells** per `loadValues()`, `storeName = "testStore1"`
  (`L88`, `protected` so the Berkeley fixed-length suite can key a length map on it). 1,240 lines.
- **`JanusGraphTest`, measured:** 649,177 B, **12,258 lines**, class declared `L312`, **222 `@Test`**,
  **7 `@ParameterizedTest`**, **5 `@Tag`** (all `BRITTLE_TESTS`, at `L3098`, `L3179`, `L10156`, `L10192`,
  `L10272`) → **229 test methods**, no `@Nested`, ten banner comments (`L321` update-then-remove, `L629`
  indexing, `L894` schema, `L3387` advanced, `L4300` consistency, `L4718` vertex-centric, `L8266`
  logging, `L8601` global graph queries, `L10066` TTL, `L10582` IO). `JanusGraphIndexTest` is 490,586 B.
- **THE finding of the lesson: four opt-out idioms, four reporting semantics.** Java does not inherit
  method annotations through an override — the repo documents this itself at `CQLGraphTest.java:250-252`
  (*"overridden without a test annotation (which excludes it from this class)"*). So:
  (1) bare `@Override` → absent from the report entirely; (2) `@Override @Test @Disabled` → reported
  `<skipped/>`; (3) `@Disabled("reason")` → reported with the reason; (4) **`@Test @Override` with an
  empty body and no `@Disabled` → runs, passes, counted as coverage.** All four are in the tree.
- **Idiom 2 at scale:** `InMemoryGraphTest.java` is 5,147 B and carries **exactly 31**
  `@Override @Test @Disabled` + empty body, at `L57, 61, 64, 67, 70, 73, 76, 79, 82, 85, 88, 91, 94, 97,
  100, 103, 106, 109, 112, 115, 118, 121, 124, 127, 130, 133, 136, 139, 142, 145, 148`. **None carries a
  reason string**, unlike `CQLGraphTest.java:390` and `:481` (`@Disabled("Use Parametrized test instead")`).
  The 31 group as: multi-instance 3, global/managed config 6, multi-query/batch 6, logging 2, schema
  maker / direct schema commit 6, drops 2, plus `testDataTypes`, `testForceIndexUsage`,
  `testTransactionConfiguration`, `testIndexUpdatesWithReindexAndRemove`. Two of them
  (`simpleLogTest(boolean)`, `simpleLogTestWithFailure(boolean)`) are `@Test` on a **parameterized**
  signature — the parent declares them `@ParameterizedTest @ValueSource(booleans={true,false})` at
  `JanusGraphTest.java:8270-8282` — latent, harmless only because `@Disabled` precedes resolution.
- **Idiom 4, the false green:** `BerkeleyFixedLengthKCVSTest.java:32-40` and
  `BerkeleyVariableLengthKCVSTest.java:45-53` both override `testConcurrentGetSlice` and
  `testConcurrentGetSliceAndMutate` with `@Test @Override` + empty body. Four reported **passes** across
  the module for concurrency behaviour never exercised. Parents at `KeyColumnValueStoreTest.java:720-728`
  → `testConcurrentStoreOps` (`L730`) → `ConcurrentRandomSliceReader` (`L780`).
- **`@FeatureFlag` covers 4 of the 24 `StoreFeatures` booleans**, and that gap is the structural reason
  sessions 03/04/05 each found an unasserted invariant. `JanusGraphFeature` = `{CellTtl, Scan,
  OrderedScan, UnorderedScan}`. `FeatureRequirementCondition` (`L26`) implements
  **`BeforeTestExecutionCallback`**, not `ExecutionCondition`, because the check needs a live instance to
  call `getStoreFeatures()` on — so an aborted test has **already** opened a throwaway manager, cleared
  storage, opened a real manager, opened a database and begun a transaction. Abort messages are at `L40`
  / `L45` / `L50` / `L55`; `default:` throws `UnsupportedOperationException`. The seven usage sites in
  `KeyColumnValueStoreTest`: `scanTest` `L445` (Scan), `testOrderedGetKeysRespectsKeyLimit` `L491`
  (OrderedScan), `testGetKeysColumnSlicesSimple` `L528` (Scan), `testGetKeysColumnSlicesOnLowerTriangular`
  `L568` (Scan), `testGetKeysWithSliceQuery` `L1002` (UnorderedScan), `testGetKeysWithKeyRange` `L1017`
  (OrderedScan), `testTtl` `L1034` (CellTtl **+** `@Tag(BRITTLE_TESTS)`).
- **In-memory features, re-read exactly** (`InMemoryStoreManager.java:64-70`): `orderedScan(true)`,
  `unorderedScan(true)`, `keyOrdered(true)`, `persists(false)`, `optimisticLocking(true)`,
  `optimizedWholeRowDeletion(true)`, `keyConsistent(buildGraphConfiguration())`. **It declares BOTH scan
  shapes** — so all four scan-flagged tests run in memory and only `testTtl` aborts (`cellTTL` unset).
  On BerkeleyJE (`orderedScan` but no `unorderedScan`, lesson 05's `L129-141`) `testGetKeysWithSliceQuery`
  aborts with `"Database doesn't support unordered scan."`. Note for lesson 27: this corrects the loose
  phrasing in lesson 05's notes that called the in-memory store "unordered".
- **`evaluateQuery`, for lessons 20–22.** Seven overloads at `JanusGraphBaseTest.java:589-750`. They
  install a `SimpleQueryProfiler` via a **hard cast** to `BasicVertexCentricQueryBuilder` (`L605`) or
  `GraphCentricQueryBuilder` (`L689`) and assert on profiler annotations: result count, pairwise order via
  `RelationComparator.compareOnOrders(...) <= 0` (`L630`) or `orders.compare(...) <= 0` (`L714`),
  `NUMVERTICES_ANNOTATION == 1` (`L646`), sub-query count, and per sub-query the `FITTED` and `ORDERED`
  annotations (`L652-653`, `L732-733`). The graph-centric form additionally asserts the **exact set of
  intersecting index names** and that `FULLSCAN` and index queries are mutually exclusive (`L738-749`).
  **The hole: both forms guard the fitted/ordered assertions with `if (subQuerySpecs.length == 2)`** and
  nothing asserts the arity — pass a `boolean[]` of any other length and the two strongest assertions
  vanish silently. `executeScanJob` (`L752-762`) pins every scan-job test to
  `setNumProcessingThreads(2).setWorkBlockSize(100)`.
- **Build-level selection decides more than annotations do.** `TestCategory` declares
  `BRITTLE_TESTS`, `SERIAL_TESTS`, `MEMORY_TESTS`, `PERFORMANCE_TESTS`; root `pom.xml:102` sets
  `test.excluded.groups = MEMORY_TESTS,PERFORMANCE_TESTS,BRITTLE_TESTS`, wired at `:423`.
  **`@Tag`-excluded methods are dropped at discovery and do not appear in the Surefire XML at all**,
  unlike `@Disabled` skips. Root surefire (`L404-425`): `argLine` = `-Xms256m -Xmx768m
  -XX:+HeapDumpOnOutOfMemoryError -ea` (`L90`), `runOrder=alphabetical` (`L409`), `failIfNoTests=false`
  (`L410`), `forkedProcessTimeoutInSeconds=21600` — **six hours** (`L412`), `reuseForks=false` (`L413`).
  With `forkCount=4` that is a fresh JVM per class, four at a time.
- **The exclusion that matters most: `**/*ConcurrentTest.java`** at root `pom.xml:416`. The
  `tinkerpop-test` execution (`L427-459`, `combine.self="override"`, `skipTests=${test.skip.tp}` which
  defaults **true** at `L96`) re-includes five of the seven filename patterns — **not**
  `*PerformanceTest.java` and **not** `*ConcurrentTest.java`. So `JanusGraphConcurrentTest` (20,004 B) and
  every `*GraphConcurrentTest` subclass are **not part of `mvn test`**, which means JanusGraph's
  concurrency coverage is off by two independent mechanisms at once (this, plus the four empty overrides
  above). Lesson 18 should take this as given when it reads `ConsistentKeyLocker`.
- **`SERIAL_TESTS` is the odd one out**: not in the default exclusion list, applied at **class** level
  (`JanusGraphEventualGraphTest`, `JanusGraphOperationCountingTest`, `CQLLogTest`), and acted on **only**
  by `janusgraph-cql/pom.xml` and `janusgraph-scylla/pom.xml`, which run those classes in a dedicated
  `<groups>SERIAL_TESTS</groups> <parallel>none</parallel>` execution and exclude them from the parallel
  pass via `<test.cql.excludes>${test.excluded.groups},SERIAL_TESTS</test.cql.excludes>`. In
  `janusgraph-inmemory` and `janusgraph-berkeleyje` the tag has no effect. `reuseForks=false` is what
  actually protects `JanusGraphOperationCountingTest`'s process-global `MetricManager`, not the tag.
- **Opt-in profiles:** `memory-test` (`L1900-1935`, activated by `-Dtest.skip.mem=false`,
  `<groups>MEMORY_TESTS</groups>`, `<excludedGroups/>`, `forkCount=1`, `argLine=${mem.jvm.opts}` =
  `-Xms256m -Xmx256m -ea …` at `L91` — the heap clamp is the point) and `performance-test`
  (`L1937-1972`, `-Dtest.skip.perf=false`). `jdk17-plus-tests` (`L1656-1680`) sets `test.jdk.jvm.opts`.
- **Global test-config override:** `TestGraphConfigs` reads `$JANUSGRAPH_CONFIG` in a static initialiser
  and `applyOverrides` rewrites **every matching key** on every test's config (called from
  `JanusGraphBaseTest.java:155`). Hard constants in the same class: `getTTL` = **10 s**,
  `getSchemaConvergenceTime` = **60 s** — the real time budget of every TTL and schema-status test.
- **Two retry mechanisms coexist.** Home-grown `RetryingTestExecutionExtension` implements
  `ExecutionCondition` + `ParameterResolver` + `TestExecutionExceptionHandler`, constructed per invocation
  with `(invocation, maxInvocations, minSuccess)`, accumulating failures in the **parent** context's store
  keyed by method name. Third-party `@RepeatedIfExceptionsTest(repeats = 3)` (rerunner-jupiter) is used
  eight times in `CQLGraphTest` (`L239, 244, 258, 263, 269, 276, 283, 290`), and its parameterized variant
  **broke on JUnit 5.13+** (comment at `L250-252`), which is what forced the hand-expansion idiom there.
  The `testing/flaky` label has **18 open issues** (#3651, #3931, #3352, #2934, #1908, #1497, #1462,
  #1091, #3356, #4269 among them).

### Four contained PR candidates, all verified at the pinned commit this run

Ordered smallest-first. None of them needs a design discussion, and all four are in files this track
has already read.

1. **`RetryingTestExecutionExtension.java:77`** prints the wrong variable:
   `"Aborting test #" + minSuccess + " of " + maxInvocations` — `minSuccess` is the *required success
   count*, not the invocation index; `invocation` is in scope and used correctly in
   `evaluateExecutionCondition`'s messages. Also missing a space before `- still have retries left`.
   One-token diff with a visible effect on every retry log line.
2. **Replace the four empty concurrency overrides with `@Disabled("…")`** —
   `BerkeleyFixedLengthKCVSTest.java:32-40` and `BerkeleyVariableLengthKCVSTest.java:45-53`. Turns four
   reported passes into four reported skips with a stated reason. Thirty-second review.
3. **`BerkeleyVariableLengthKCVSTest.retrieveBackendAndGetKeyValue` cannot fail.** Its body (`L56-120`)
   is one big `try` whose outer `catch (BackendException e) { //generic failure }` at **`L113`** has no
   `fail`, no rethrow and no assertion; the inner try (`L90-102`) wraps the very call its own comment at
   **`L93`** marks *"Errors out here without fix"* and catches the same exception with only
   `txh.rollback()`. Both `assertEquals` calls sit after the throwing line, so the regression this test
   guards against produces a **pass**. The only surviving failure mode is `getSlice` returning wrong
   contents (an `AssertionError`, which does propagate). It also imports **JUnit 4**
   `org.junit.Assert.assertEquals` while the suite is on `org.junit.jupiter.api`, and opens its own
   transactions with `StandardBaseTransactionConfig.of(TimestampProviders.MICRO)` instead of the
   inherited `getTxConfig()`. Fix: delete both catches (declare `throws BackendException`) or
   `fail(e.getMessage())` in each; switch the import; use `getTxConfig()`.
4. **The reproduction test for [#4781](https://github.com/JanusGraph/janusgraph/issues/4781)**, deferred
   here by lesson 05 and now specified. **Re-verified this run: still open, unassigned, no linked PRs,
   untouched since 2025-03-21, one 👍, label `kind/bug/possible`.** It belongs in
   `janusgraph-berkeleyje/src/test/.../graphdb/berkeleyje/BerkeleyGraphTest.java` (9,971 B) rather than in
   the shared suite, because the behaviour is backend-specific: override `getConfiguration()` with
   `storage.transactions=false`, then `graph.addVertex(T.label, "fake label")` and `assertThrows` the
   `PermanentBackendException`. The issue's stack trace reports
   `BerkeleyJEStoreManager.beginTransaction(BerkeleyJEStoreManager.java:210)` on 1.1.0/1.2.0 where lesson
   05 located the same throw at **`L217-221`** at the pinned commit — a reminder that a reported line
   number is version-specific. Second test needed for the silent lock loss at
   `BerkeleyJEKeyValueStore.java:159-162`, which a fix touching only `beginTransaction` would miss.

### Still open, and who owns it

- **Owned by lesson 07, and now answerable by a test** rather than by reading: `MemoryEntryList` sized to
  the range while the limit is checked inside the loop; whether `getNoCopy`'s aliasing of live `rawData`
  is observable across a `mutate` (this one belongs in `KeyColumnValueStoreTest`, gated by nothing, and
  would then apply to every backend); and the `this.lock.lock()`-bypasses-`getLock(txh)` NPE candidate at
  `InMemoryColumnValueStore.java:162` onward.
- **Carried from sessions 01–04:** 24 questions posed, none answered by Aryant. Lesson 05 closed several
  by contrast; the half-open interval now has three distinct implementations on record
  (`SinglePageEntryBuffer.getSlice` binary searches; `BerkeleyJEKeyValueStore.java:258`'s per-record
  `compareTo`; `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange` predicate). Lesson 27 adds the
  CQL backend as a fourth. The contract is in `KeyColumnValueStore`'s javadoc and **none** of the
  enforcement is.
- **Still deferred from Part I:** `MultiPageEntryBuffer`, `BufferPageUtils`, `BufferPage.merge` (both
  overloads) and the fragmentation machinery; `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299`+, shared mutable `current`, per-key limit at
  `L381`) — both now belong with **lesson 07**'s scan tests rather than with lesson 06, since the harness
  lesson did not need to run them. `BerkeleyJETx` (6,524 B) and `BerkeleyJELocks` go with **lesson 18**.
  From lesson 06: `JanusGraphAssert` (12,563 B), `LockKeyColumnValueStoreTest` (23,021 B),
  `MultiWriteKeyColumnValueStoreTest` (14,237 B), `IDAuthorityTest` (20,575 B) and `SimpleScanJob`
  (14,040 B) were located but not read — they belong with lessons 18, 16, 09 and 07 respectively.
- **Entry size.** The 5–9 KB budget was a `contributor-reading` constraint and no longer applies. Lesson
  05 was ~35 KB, lesson 06 ~63 KB, in line with the SQLite and PostgreSQL lessons. Do not trim toward the
  old number.
- **tdd.cat: stop looking for the leader.** The 2026-11-03 amendment stands and was confirmed on
  2026-11-04: the front page served the maximum date (**2026-11-03**, 52 stories) complete with source
  URLs on the first fetch, and `/archive/` (128 editions) agreed — two consecutive runs where the cheap
  endpoints were simply correct. The procedure is the one the task already states: fetch the front page,
  digest whatever edition it serves, **say which one it is**, cross-check `/archive/` for the maximum,
  treat RSS as a third opinion. Do not spend entry space re-deriving an ordering. One new
  rendering-quality note: the 2026-11-03 edition's item 41 had a title and a summary that belonged to
  different stories, so a mislabelled item is a thing to notice rather than resolve.

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
