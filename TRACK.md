<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 8
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

- **08 — 2026-11-06** · [`2026/11/2026-11-06-janusgraph-id-bit-layout/README.md`](2026/11/2026-11-06-janusgraph-id-bit-layout/README.md) · **Part II opens; stayed on `a59ae8d8` deliberately** so Part II's byte offsets stay comparable with lessons 02-07. Read `IDManager` (whole, 754 lines), `BufferUtil` (whole), `NumberUtil` (whole), `VertexIDAssigner` (whole; cited L104-107/L112/L123/L130-133), `ManagementSystem` (whole; L561-568 used), `IDManagementTest` (whole), `JanusGraphPartitionGraphTest` (whole; L87/L97-99/L113-119/L391/L396/L493), `InMemoryPartitionGraphTest` (whole, 51 lines), `docs/advanced-topics/data-model.md` and `docs/advanced-topics/custom-vertex-id.md`; plus the complete caller sets of `new IDManager(`, `IDManager()` and `fromVertexId` by code search. **THE finding: the vertex id and the edgestore row key are two different 64-bit words.** `constructId:450` builds `[0 | count | partition | suffix]`; `getKey:502` rebuilds it as `(partition << partitionOffset) | addPadding(count)` with `partitionOffset = Long.SIZE - partitionBits` — the partition rotates from the middle of the word to the **top**, count stays in the low bits and is never shifted by `partitionBits`. So at the production default id `104` has row key `0xa000000000000008`, and with `partitionBits > 0` the key's **sign bit can be set** (partition 17 at pb=5 → `0x8800000000000008`), which is why `getKeyID` is `>>>` throughout. Second finding: **every default graph runs at `partitionBits = 5`, not 0** — `cluster.max-partitions` is `FIXED`, default **32**, validator demands `>1` and a power of 2, `NumberUtil.getPowerOf2(32) = 5`; `DEFAULT_PARTITION_BITS = 0` and the no-arg `IDManager()` have **no caller in the tree**. Third: **`fromVertexId` accepts a temporary (negative) id and returns garbage** — clause 1 of its precondition uses `>>>`, clause 2 compares signed against a positive bound, the body uses `>>`; verified at pb ∈ {0,3,5,16}, now PR candidate 8. **Closed lesson 07's open question:** `EDGESTORE_MAX_KEY` (eight `0xFF`) is unreachable because no proper `VertexIDType` suffix is `111` and no schema id ends in `11`; `EDGESTORE_MIN_KEY` needs `count == 0`, which `addPadding:347` forbids. Also: the one always-zero gap bit at position `63-partitionBits` of every row key; the XOR fold in `getPartitionHashForId:566-577` is the **identity** for counts below `2^partitionBits`; `getPartitionedVertexRepresentatives:601-609` allocates 512 KB per call at pb=16. Hands-on: `ManagementSystem.getVertexKey`/`getVertexId` as a public window onto the rotation, a jshell reproduction of the `fromVertexId` gap, and `InMemoryPartitionGraphTest` — which **empties three inherited tests** (`testPartitionSpreadFlushBatch`, `testPartitionSpreadFlushNoBatch`, `testKeyBasedGraphPartitioning`) and reports them passing, a third site for the empty-override anti-pattern. Daily Diff: **no 2026-11-06 edition**; front page served 2026-11-04 while `/archive/` listed **2026-11-05** as latest, so the 2026-11-05 edition (62 stories) was digested and said so — deep dive on GitHub's new Git storage architecture ("Rubicon") and its immutable-objects/mutable-refs split.
- **07 — 2026-11-05** · [`2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md`](2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md) · **Part I complete.** Read `Backend` (whole, 775 lines), `BackendTransaction` (whole, 568), `BackendOperation` (whole, 148), `CacheTransaction` (whole, 248), `KCVSCache` (whole, 96), `NoKCVSCache` (whole, 44), `KCVSProxy` (whole, 94), `ExpectedValueCheckingStoreManager` (whole, 108), `ExpirationKCVSCache` (L40-190 + L215-359 of 359), `ExpirationCacheTest` (L41-160), and `GraphDatabaseConfiguration` L628-656 / L780-814 / L982-984 / L1054-1057 / L1607-1609 / L1946-1962. Taught the constructor-decides / `initialize`-opens split, the six store names and which of the two managers opens each (three of six through the locking manager, nine physical stores on a locking-emulated backend), the four-level transaction stack and the one level `unwrapTx` strips, `CacheTransaction`'s buffer and its monotonic `numMutations`, the invalidation-does-not-evict semantics and the `CountDownLatch(5)` back-pressure cleaner, and the `hasLocking()`-means-two-things consequence of reading features at `L249`. **THE finding: the retry backoff is a coin flip, not a ±20% jitter** — `(int)(2.0 / (1 + (rand*2-1)*0.2))` truncates to 1 or 2, so `pertubTime(d)` ∈ {d, d/2}; reproduced verbatim under OpenJDK 21 (`{1=500803, 2=499197}` over 1,000,000 draws) and measured at a median of **14** attempts (max 38) against the 10 s read budget and **21** (max 48) against the 100 s write budget, against documentation that says "backoff exponentially". Second finding: **`KCVSProxy.java:67` omits `unwrapTx`** where its seven siblings apply it — latent, one token, now PR candidate 1. Hands-on: a jshell reproduction of the divisor distribution, and `ExpirationCacheTest#testGracePeriod`, which asserts the cache-bypass semantics directly (`getSliceCalls` 0 → 2 → 4 on two identical reads) and needs exactly `PENALTY_THRESHOLD` = 5 reads to wake the cleanup thread. Daily Diff: the **2026-11-04** edition, 56 stories, front page one ahead of `/archive/` — deep dive on PostgreSQL 19's `REPACK CONCURRENTLY` and its not-MVCC-safe final swap.
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
- [x] 07 — `Backend` and `BackendTransaction`: the stores that get opened (edgestore, graphindex, system_properties, ids, logs) and the cache/locker wrappers

### Part II — Graph ⇄ bytes (`graphdb/idmanagement`, `graphdb/database`)

- [x] 08 — `IDManager`: vertex ID bit layout, partition bits, vertex-type suffixes, and how row keys are derived
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

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30.** Lesson 08 made one explicit decision the Part boundary allowed and declined the other:
it **stayed pinned on `a59ae8d8`** rather than re-pinning, because every line number in lessons 08-13
is cross-referenced against lessons 02-07 and Part II is the layer where byte offsets matter most.
Re-pin at the **Part II/III boundary** (after lesson 13) instead, and record the new commit here.

Lesson 07 handed the deferred `InMemoryKeyColumnValueStore.RowIterator` /
`OrderedKeyValueStoreAdapter.KeyIteratorImpl` reading on to **lesson 22** (query processing and limits)
rather than keeping it in Part I: the per-key limit at `OrderedKeyValueStoreAdapter.java:381` is the point
of that code, and lesson 07's scan material turned out to be about `hasScan`/`hasOrderedScan` branching in
`BackendTransaction`, not about iterator mechanics. `SimpleScanJob` goes with it.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01–04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Part I is complete; Part II is under way.** Lessons 01–07 cover the storage contract, two store
  implementations, the adapter, the harness, and the wiring; lesson 08 opened Part II. **The re-pin
  decision at the Part I/II boundary was taken and recorded: stay on `a59ae8d8`.** Do not revisit it
  mid-Part. The next boundary where re-pinning is allowed is **Part II/III, after lesson 13**; at that
  point either re-pin and record the new commit in the front-matter and here, or state explicitly that
  it is staying, as lesson 08 did.

### Verified by lesson 08, for lessons 09-13 and the read path to cite rather than re-derive

- **An id is not a row key, and `getKey:502` is the whole difference.** `constructId:450-459` builds
  `[0 | count | partition | suffix]`; `getKey:489-510` rebuilds it as
  `(partition << partitionOffset) | type.addPadding(count)` with `partitionOffset = Long.SIZE - partitionBits`
  (`:431`, **64 not `TOTAL_BITS`**). Count is **not** shifted by `partitionBits` in the key. Verified
  examples at pb=3: `(count=1, part=5)` → id `104` / key `0xa000000000000008`; `(count=2, part=5)` → id
  `168` / key `0xa000000000000010`. At **pb=0 the two words are identical**, which is why the distinction
  is invisible in most tests. Round trip `getKeyID(getKey(id)) == id` re-verified against
  `IDManagementTest.EntityIDTest`'s own five parameter sets, 0 mismatches over ~9,000 counts × 3 proper
  user-vertex types.
- **`partitionBits = 5` is the production default and 0 is not configurable.** `CLUSTER_MAX_PARTITIONS`
  (`cluster.max-partitions`) is `ConfigOption.Type.FIXED`, default **32**, validator
  `integer>1 && NumberUtil.isPowerOf2(integer)`. `VertexIDAssigner.java:104` is
  `NumberUtil.getPowerOf2(config.get(CLUSTER_MAX_PARTITIONS))`, and `getPowerOf2` is
  `Long.SIZE-(numberOfLeadingZeros(value)+1)` → **5**. `ConsistentKeyIDAuthority` re-derives the same
  number independently and re-checks `0..16`. **`DEFAULT_PARTITION_BITS = 0` (`:396`) and the no-arg
  `IDManager()` (`:434-436`) have no caller anywhere in the tree** — code-searched at the pinned commit:
  `new IDManager(` matches exactly `VertexIDAssigner.java:105` and `IDManagementTest`, both with an
  explicit width. Do not build mental models on pb=0.
- **The bounds table, exact** (`vertexCountBound = 1L << (63 - pb - 3)`, `:429`):
  pb=0 → 2^60; pb=3 → 2^57 (the test default, `JanusGraphPartitionGraphTest:87` sets
  `cluster.max-partitions = 8`); **pb=5 → 2^55 = 36,028,797,018,963,968**; pb=8 → 2^52; pb=12 → 2^48;
  pb=16 (`MAX_PARTITION_BITS`, `:392`) → 2^44 = 17,592,186,044,416. One partition bit costs one bit of
  vertex count, and the choice is `FIXED` at graph creation. `relationCountBound` is special-cased to
  `Long.MAX_VALUE` at pb=0 (`:427`) because `1L << 63` would overflow.
  `SCHEMA_COUNT_BOUND = 1 << 54` (`:410`) uses `MAX_PADDING_BITWIDTH = 6` and `TYPE_LEN_RESERVE = 3`
  and does **not** depend on `partitionBits` — schema ids have no partition field (`:678-681`).
- **One always-zero gap bit per row key.** Since `count < 2^(60-pb)`, `count<<3` occupies bits
  `0..62-pb` while the partition starts at bit `64-pb`: **bit `63-pb` of every user-vertex row key is
  always 0**. At pb=0 that bit is the sign bit the `TOTAL_BITS` comment (`:382-387`) is about. At pb>0
  the key's **sign bit belongs to the partition and can be set** — pb=5, partition 17 →
  `0x8800000000000008`. Ordering is still correct because `StaticBuffer` compares unsigned byte-wise;
  arithmetic after `getLong(0)` is not, which is what `getKeyID`'s javadoc (`:523-529`) warns about and
  why it uses `>>>` exclusively. `:543`'s `partitionOffset<Long.SIZE ? ... : 0` ternary is **required**,
  not defensive: Java masks long shift distances to 6 bits, so `value >>> 64` is `value >>> 0`.
- **`VertexIDType` is a variable-length suffix code, not a stored tag.** 17 members,
  `offset()`/`suffix()`/`isProper()`. Proper user vertices: `NormalVertex` `000`, `PartitionedVertex`
  `010`, `UnmodifiableVertex` `100` — **all three have `offset() == 3`**, asserted at `getKey:498` and
  guarded for the whole enum by `IDManagementTest.testUserVertexBitWidth` (whose first clause is a bare
  `assert`, so a no-op without `-ea`). Schema side: `Schema` `01`, `SchemaType` `101`, `RelationType`
  `0101`, `PropertyKey` `00101`, `UserPropertyKey` `000101`, `SystemPropertyKey` `100101`, `EdgeLabel`
  `10101`, `UserEdgeLabel` `010101`, `SystemEdgeLabel` `110101`, `VertexLabel` `01101`,
  `GenericSchemaType` `1001`; `Invisible` `1`, `InvisibleVertex` `11`. `is()` (`:355-362`) is a masked
  compare, so the hierarchy costs nothing — and **on a non-`Number` it returns `suffix() == 0`**, which
  is the custom-string-id path: a string id is a `NormalVertex` by definition and can never be schema or
  partitioned. `UserPropertyKey` count=1 → id `69` = `0b1000101`; `UserEdgeLabel` count=1 → `85`.
- **`EDGESTORE_MAX_KEY` and `EDGESTORE_MIN_KEY` are both unreachable — lesson 07's question, closed.**
  `oneBuffer(8)` ends in `111`; no proper `VertexIDType` suffix is `111` (only `000`/`010`/`100`), and
  `getUserVertexIDType:461-470` throws `InvalidIDException` for anything else. The schema branch has
  key == id and requires low 2 bits `01`, but `0xFF & 3 == 3`. String ids can't collide: `getStringIdBuffer`
  forces `length != 8`, and the ordered-scan branch is gated on `!allowCustomVertexIdType` anyway.
  `zeroBuffer(8)` needs `count == 0`, forbidden by `addPadding:347` and asserted at `getKey:501`. The
  half-open interval is therefore safe here — **by accident of the suffix encoding, documented nowhere**.
  See PR candidate 9.
- **Partitioned vertices: one id, up to `2^pb` row keys, and the hash is the identity for small counts.**
  `getVertexID:555-564` checks `partition == PARTITIONED_VERTEX_PARTITION (1)` then **discards it**,
  calling `getCanonicalVertexIdFromCount`. Placement is the XOR fold `getPartitionHashForId:566-577`,
  guarded by `checkState(partitionBits>0)` because the loop never advances at pb=0. **Every count below
  `2^pb` folds to itself** (verified: at pb=16, counts 1..10 → partitions 1..10 in order), so the first
  handful of partitioned vertices get exactly the sequential placement the fold exists to prevent;
  `testPartitionHashes` (`JanusGraphPartitionGraphTest:113-119`) only asserts `> bound/2` distinct over
  `1..2*bound`. `getPartitionedVertexRepresentatives:601-609` allocates `long[getPartitionBound()]`
  fresh per call — 32 longs at pb=5, **65,536 longs / 512 KB at pb=16**, with only a bare `assert` at
  `:603` between that and an `int` overflow (the real guard is `VertexIDAssigner:106`).
- **Temporary ids preserve the bit pattern.** `makeTemporary:665-668` is `(1L<<63) | id` with
  `checkArgument(id>0)`; `isTemporary` is `id<0`. Every `VertexIDType.is()` check still works on a
  temporary, which `IDManagementTest` asserts for `NormalVertex`, `UserEdgeLabel` and `Invisible`.
  Temporaries carry no partition (`:650-654`). This is what PR candidate 8 trips over.
- **Three invariants are `assert`, not `Preconditions`:** `getKey:498`, `getKey:501`,
  `getPartitionedVertexRepresentatives:603`. No-ops under a normal JVM, all on cold paths.
- **`relationCountBound` and `constructId` disagree at pb=0.** `getRelationID:549-552` admits
  `count < Long.MAX_VALUE`; `constructId:453-454` then requires
  `unsignedBitLength(count)+pb+offset <= 63`. Unreachable in practice because pb=0 is not configurable.

### Verified by lesson 07, for Part II and the write/read path lessons to cite rather than re-derive

- **`Backend` is two-phase and the phases are different kinds of work.** Constructor
  (`Backend.java:239-306`) turns capability into configuration; `initialize` (`L333-417`) opens stores.
  `storeFeatures = storeManager.getFeatures()` is read at **`L249`**, *before* the locking wrapper is
  built at `L271-276`, and nothing re-reads it. So `Backend.getStoreFeatures().hasLocking()` is **false**
  on CQL/HBase/in-memory for the life of the graph while `storeManagerLocking.getFeatures().hasLocking()`
  is **true** (`ExpectedValueCheckingStoreManager.java:60`, `:104-106`). `BackendTransaction` gets the
  first one (`Backend.java:628`). Lesson 18 will need this distinction; do not re-derive it.
- **The six store names and which manager opens each.** `janusgraph_ids` (`IDS_STORE_NAME`, GLOBAL_OFFLINE,
  default `JanusGraphConstants.JANUSGRAPH_ID_STORE_NAME`) via `storeManager` → straight into
  `ConsistentKeyIDAuthority`, **no cache, no lock wrapper** (`L336`, `L340`); `edgestore` (`L345`) and
  `graphindex` (`L346`) via **`storeManagerLocking`** → `ExpirationKCVSCache` or `NoKCVSCache`;
  `txlog` opened **twice** — once as a log (`txLogManager.openLog`, `L379`) and once raw into a
  `NoKCVSCache` for `getTxLogPersistor()` (`L381`); `systemlog` via `KCVSLogManager` (`L380`);
  `system_properties` via **`storeManagerLocking`**, opened **once** and shared by both the global and the
  user `KCVSConfiguration` (`L385-412`). Three of six go through the locking manager, so on a
  locking-emulated backend the real store count is **nine**: plus `edgestore_lock_`, `graphindex_lock_`
  and `system_properties_lock_` (`LOCK_STORE_SUFFIX = "_lock_"` at `L157`, appended at
  `ExpectedValueCheckingStoreManager.java:68`). On BerkeleyJE (`hasLocking()` true,
  `storeManagerLocking == storeManager`) there are **no** `_lock_` stores at all.
- **Lockers are lazy, keyed, and only ever exercised with one group.** `Backend.getLocker` (`L310-325`) is
  a `ConcurrentHashMap` + `putIfAbsent`, fed by a two-entry registry (`consistentkey` → a
  `ConsistentKeyLocker.Builder` on a store opened from the **unwrapped** `storeManager` at `L190`;
  `test` → `TestLockerManager` by reflection, `L183`, `L726-745`). `LOCK_BACKEND` is `GLOBAL_OFFLINE`,
  default `"consistentkey"` (`GraphDatabaseConfiguration.java:982-984`). Combined with lesson 06's finding
  that `JanusGraphBaseTest.getBackend` pins `LOCK_LOCAL_MEDIATOR_GROUP` to `"tmp"`, **no test in the tree
  exercises two mediator groups against one store.** Lesson 18 should take that as given.
- **The fixed-key-length map, exactly** (`Backend.java:537-542`): `edgestore` → 8, `edgestore_lock_` → 8,
  `<ids store name>` → 8. Everything else (`graphindex`, `system_properties`, `txlog`, `systemlog`) is
  variable-length on an `OrderedKeyValueStoreManager` and pays lesson 05's length prefix and post-filter.
  The `edgestore_lock_` entry is reachable only by an out-of-tree ordered-KV backend without innate
  locking — forward-looking, not dead — and such a backend would get fixed keys for `edgestore_lock_` and
  variable for `graphindex_lock_`. `ALLOW_CUSTOM_VERTEX_ID_TYPES` is rejected outright for ordered-KV
  (`L526-536`), closing the manager first with a comment about BerkeleyJE's leaked environment.
- **The transaction stack is four levels deep, and `unwrapTx` strips exactly one.**
  `BackendTransaction` → `CacheTransaction` → `ExpectedValueCheckingTransaction` → {`inconsistentTx`,
  `strongConsistentTx`}. `KCVSCache.unwrapTx` is `protected final` (`KCVSCache.java:80-83`) and returns
  `((CacheTransaction) txh).getWrappedTransaction()`, so the store below a cache receives the
  `ExpectedValueCheckingTransaction`. `ExpectedValueCheckingStoreManager.beginTransaction` (`:89-101`)
  opens **two** real backend transactions per graph transaction **unconditionally**, and `mutateMany`
  (`:75-86`) sends the whole flush to the consistent one if `prepareForMutations()` reports any lock held
  — all-or-nothing per flush, never per key.
- **`KCVSProxy` BUG, verified, latent, one token.** `KCVSProxy.java:66-68` —
  `getKeys(MultiSlicesQuery queries, StoreTransaction txh)` returns `store.getKeys(queries, txh)` **without
  `unwrapTx`**, while all seven other delegating methods in the file (`:52, :57, :62, :77, :82, :87, :92`)
  apply it. Through a `KCVSCache` the backend store would receive a `CacheTransaction` and
  `ClassCastException` on its own cast. Grepped the whole tree: the method is declared at
  `KeyColumnValueStore.java:227`, implemented by `OrderedKeyValueStoreAdapter:157`,
  `CQLKeyColumnValueStore:816`, `HBaseKeyColumnValueStore:155`, `InMemoryKeyColumnValueStore:147`,
  `MetricInstrumentedStore:234` (not a `KCVSProxy`, so nothing to unwrap there), referenced in a
  `StoreFeatures:49` javadoc — and **called by nothing** above the storage layer. The test double at
  `KCVSCacheTest.java:241-242` reproduces the same omission, so no test would catch it either.
  See PR candidate 1 below.
- **`NoKCVSCache` is 44 lines of empty overrides** and passes `metricsName = null`, which `incActionBy`
  checks (`KCVSCache.java:52-57`). "Cache off" is a subclass, not a read-path flag — `getSlice` is
  `KCVSProxy`'s plain delegation. Cache metrics need **both** a non-null name from
  `Backend.getMetricsCacheName` (`L494-497`: `null` without `basic-metrics`, `"stores"` with
  `metrics.merge-stores`, else `edgestore.cache` / `graphindex.cache`) **and** a group name on the
  transaction.
- **Invalidation does not evict.** `ExpirationKCVSCache.invalidate` (`:266-270`) only does
  `expiredKeys.put(key, now + cacheTimeMS)` plus a 1/1000 latch countdown; it never touches `cache`.
  `getSlice`'s first branch (`:93-98`) sees `isExpired` and goes to the store **without caching the
  result**, so between a mutation and the sweep every read of that row is a full round trip while the
  stale entry still costs `maximumWeight`. The sweep is triggered by a **`CountDownLatch(PENALTY_THRESHOLD
  = 5)`** counted down by every bypassing read (`:311`) and, with probability
  `1/INVALIDATE_KEY_FRACTION_PENALTY = 1/1000`, by an invalidation (`:269`); the daemon `CleanupThread`
  (`:329-357`) does nothing but `await()` then `clearExpiredCache(true)`. `clearExpiredCache` (`:277-295`)
  is `synchronized`, only sweeps keys with `getAge >= invalidationGracePeriodMS`, and does a **full scan of
  `cache.asMap().keySet()`** per sweep. `clearCache()` carries an eleven-line comment (`:253-259`)
  explaining two orderings: sweep `expiredKeys` before `invalidateAll`, and only the cleanup thread may
  reassign `penaltyCountdown` — which is what the `withNewPenaltyCountdown` boolean encodes.
  **The cache key is the whole `KeySliceQuery`**, so one row read at limit 10 and limit 100 caches twice.
- **Cache sizing, exact.** `cache.db-cache-size` default **0.3**
  (`GraphDatabaseConfiguration.java:628-631`); below 1.0 it is a fraction of *currently free* heap,
  `(maxMemory - (totalMemory - freeMemory)) * cacheSize` (`Backend.java:359-360`), so the budget depends on
  when the graph is opened; above 1.0 it is absolute bytes and must exceed 1000. Split is fixed at
  `EDGESTORE_CACHE_PERCENT = 0.8` / `INDEXSTORE_CACHE_PERCENT = 0.2` with a sum check (`L162-163`, `L367`).
  `cache.db-cache-time` default **10000 ms** (`:651-656`), and **0 is remapped** to
  `ETERNAL_CACHE_EXPIRATION` = 200 years in ms (`Backend.java:165`, `:352`) because the cache constructor
  rejects zero. `cache.db-cache-clean-wait` default **50 ms** (`:639-643`).
- **`CacheTransaction`, the write buffer.** `Map<KCVSCache, Map<StaticBuffer, KCVEntryMutation>>` keyed by
  the **cache object**, name resolved only at flush (`:164`). `numMutations += m.getTotalMutations()` at
  `:102` is added **before and regardless of the merge** at `:96-97` and is never reduced by `consolidate`,
  so it is an upper bound and **both** thresholds it feeds fire early on repeated writes to the same keys.
  The only automatic flush is `if (batchLoading && numMutations >= persistChunkSize)` (`:104`) — without
  batch loading nothing flushes until `commit()` (`:232-235` = `flushInternal(); tx.commit();`).
  `flushInternal` (`:148-201`): consolidate serially below `numMutationsParallelThreshold` else via
  `parallelStream` (`:151-157`); chunk and `persist` (`:160-176`), where `private int persist(...)` always
  `return 0` (`:124`) and the assignment `numSubMutations = persist(subMutations)` is a counter reset, and
  a chunk boundary **can fall mid-store**; invalidate **after** persistence, per key (`:179-198`) — and
  `hasValidateKeysOnly()` is `true` in the base class with **no override in the tree**, so the entry-list
  branch at `:185-195` is **dead** and `ExpirationKCVSCache.invalidate:267` asserts it stays so; `clear()`
  (`:203-208`) empties the inner maps but keeps the outer store keys. `logMutations` (`:211-224`) opens
  with `checkArgument(!batchLoading, ...)` — that one precondition is why transaction logging and batch
  loading are mutually exclusive (lesson 28).
- **`storage.buffer-size` is a durability knob, and it is ignored on some backends.** Default **1024**
  (`GraphDatabaseConfiguration.java:780-782`), but `Backend.java:261-263` replaces it with
  `Integer.MAX_VALUE` when `!storeFeatures.hasBatchMutation()` — an *unbounded* buffer, not a buffer of
  one. So a 5,000-mutation transaction becomes five `mutateMany` calls on CQL and one on a non-batching
  backend, and multi-chunk atomicity is the backend transaction's problem.
  `storage.num-mutations-parallel-threshold` default **100** (`:789-794`).
- **`BackendOperation`, and the backoff that is not exponential — THE finding of the lesson.**
  `BASE_REATTEMPT_TIME = 50 ms`, `PERTURBATION_PERCENTAGE = 0.2` (`BackendOperation.java:40-41`).
  `pertubTime` (`:43-49`) computes `duration.dividedBy((int)(2.0 / (1 + (rand*2-1.0)*0.2)))`. The inner
  expression lands in `(1.667, 2.5]` and `(int)` **truncates**, so the divisor is only ever **1 or 2** and
  `pertubTime(d)` returns exactly `d` or `d/2`, a fair coin flip at `rand = 0.5` — not a ±20% jitter.
  Verified under OpenJDK 21 with real `java.time.Duration`: `{1=500803, 2=499197}` over 1,000,000 draws;
  `pertubTime(50ms)` yields exactly `{25 ms, 50 ms}` over 100,000 draws. Since `waitBase =
  waitTime.multipliedBy(2)` (`:100`), the base either doubles or stands still, geometric mean **√2**.
  Measured over 200,000 simulated runs: **10 s budget (`storage.read-time`, `:801-804`) → min 8, p50 14,
  p99 24, max 38, mean 14.52 attempts; 100 s budget (`storage.write-time`, `:796-799`) → min 11, p50 21,
  p99 33, max 48, mean 21.11.** Both options' own documentation says "backoff exponentially". Every retry
  logs at **INFO with a full stack trace** (`:89`). The `assert !duration.isZero()` at `:47` sits **after**
  the division it guards; the divisor's floor of 1 is what actually prevents `ArithmeticException`, and
  nothing documents that. Exception triage (`:70-84`) keeps the **innermost** `BackendException`: temporary
  → retry; a `BackendException` thrown directly → rethrown unretried; anything else → wrapped in
  `PermanentBackendException("Permanent exception while executing backend operation " + exe.toString())`,
  which is why every `Callable` overrides `toString()` with a bare label (`EdgeStoreQuery`,
  `MultiEdgeStoreQuery`, `VertexIndexQuery`, `EdgeStoreKeys`, `IndexQuery`, `indexQueryAggregation`,
  `RawQuery`, `Totals`, `CacheMutation`). Those strings are the only identification a permanent failure
  carries.
- **`BackendTransaction`'s five feature branches** (all against the raw features from `Backend.java:249`):
  `hasMultiQuery()` at `:303` and `:322`; `hasScan()` at `:395` (else
  `UnsupportedOperationException("… use Faunus instead")`); `isKeyOrdered() && !allowCustomVertexIdType` at
  `:401`; `hasOrderedScan()` at `:415`. **`EDGESTORE_MIN_KEY = zeroBuffer(8)` / `EDGESTORE_MAX_KEY =
  oneBuffer(8)`** (`:69-71`, comment *"Assumes 64 bit key length as specified in IDManager"*) — eight
  `0xFF` bytes, and lesson 02's half-open interval means the maximal key is **excluded** from a full
  ordered scan. Lesson 08 owns whether that key is assignable. `MIN_TASKS_TO_PARALLELIZE = 2` (`:67`), so a
  single-key multi-query never touches the pool (`:357`). The parallel fan-out (`:355-392`) counts failures
  in an `AtomicInteger`, `log.warn`s each, and throws `JanusGraphException("… N individual queries
  failed.")` **with no cause attached**; `doneSignal.await()` has **no timeout**, and only `Exception` is
  caught, so a task that throws an `Error` never reaches `countDown()` and hangs the caller.
- **Commit/rollback asymmetry, deliberate.** `commit()` (`:165-168`) is storage-then-indexes and does
  **not** catch index exceptions — the first throw leaves later indexes uncommitted. `rollback()`
  (`:176-189`) is indexes-then-storage, catches per index, keeps only the **last** exception (no suppression
  chaining) and rethrows after `storeTx.rollback()`. `commitIndexes()` (`:152-162`) collects into
  `Map<String,Throwable>` and **returns** it; `StandardJanusGraph` uses the split pair for exactly that
  (lesson 16). Storage-first is what makes the edgestore the source of truth and a mixed index a
  repairable artefact — lesson 28's premise.
- **`GraphDatabaseConfiguration.getBackend()` is a factory, not an accessor** (`:1951-1956`): every call
  does `new Backend(configuration); backend.initialize(configuration);` and mutates the config's
  `storeFeatures` field as a side effect. A new store manager, new index providers, new log managers, a new
  `StandardScanner` and — with the cache on — two new `ExpirationKCVSCache`es each starting a new daemon
  `CleanupThread`. `StandardJanusGraph:289` calls it once and exposes the result at `:533`. Three call
  sites build a **second** `Backend` against a live graph: `JanusGraphIndexTest:4401`
  (try-with-resources), `JanusGraphEventualGraphTest:67` (**never closed**, built only to read
  `hasTxIsolation()` inside a `checkArgument`), and `JanusGraphFactory:228` in the drop path.
  **Not verified:** whether the second one succeeds or fails on BerkeleyJE's environment lock. See
  PR candidate 5.
- **Shutdown.** `close()` (`L633-662`) and `clearStorage()` (`L671-704`) share **one** `hasAttemptedClose`
  flag, each set **before** the work, so either makes the other a debug-logged no-op and a mid-way failure
  leaves the `Backend` permanently un-closeable. Both use `ExceptionWrapper` + `executeWithCatching` so
  every step runs and the first exception is rethrown with the rest suppressed. Order: logs → scanner →
  stores → id authority → configs → **index providers** → store manager → thread pool, with a comment at
  `L650-652` explaining it — closing the CQL store manager's executors can restore a swallowed interrupt
  and break the I/O of anything closed after it. `close()` null-checks `edgeStore`/`indexStore`/
  `idAuthority`/`systemConfig`/`userConfig` (`L645-649`); `clearStorage()` **does not** (`L684-688`), so it
  NPEs on a constructed-but-uninitialized `Backend`.
- **`StandardScanner` is built from the unwrapped `storeManager`** (`Backend.java:305`), so scan jobs
  bypass **both** the cache and the locking wrapper. `buildStoreIndexScanJob` (`L452-463`) pins
  `setNumProcessingThreads(1)` and `setWorkBlockSize(PAGE_SIZE)`, which lesson 06 found
  `JanusGraphBaseTest.executeScanJob` overrides to `(2, 100)`. `ManagementSystem` has six call sites
  (`:1034, :1063, :1114, :1116, :1366, :1617`) — lesson 25.
- **No non-key-consistent fallback exists.** `initialize` `L339-343` throws `IllegalStateException` (not a
  `BackendException`) if `!isKeyConsistent()`. The `idAuthority = null` on `L338` is dead. That is the hard
  floor under every backend in the tree, and the reason lesson 03 found `keyConsistent(...)` to be the one
  feature with no `@FeatureFlag` guarding it.
- **`ExpirationCacheTest` is the single concrete subclass of `KCVSCacheTest`**
  (`janusgraph-test/src/test/java/org/janusgraph/diskstorage/cache/ExpirationCacheTest.java:41`), and
  `testGracePeriod` (`:108-148`) **asserts the bypass semantics directly**: after one
  `cache.mutateEntries` + `commit`, two identical `verifyResults` calls take `store.getSliceCalls()`
  0 → 2 → **4** (`:131`, `:134`); then `minCleanupTriggerCalls = 5` — the same number as
  `PENALTY_THRESHOLD` — reads are needed to drive the latch to zero and wake `CleanupThread` (`:138-141`)
  before two more reads finally give `0` store calls (`:143-147`). Both methods there are
  `@RepeatedIfExceptionsTest(repeats = 3)` with comments naming issues **#2934** and **#3352** — lesson
  06's flaky machinery applied to the two tests that depend on wall-clock sleeps and a daemon thread.
  `CACHE_SIZE = 48 MB`, `METRICS_STRING = "metrics"` (`:43-44`).

### PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 8–10 are new this run** (lesson 08) and 8 is now the smallest
the track has found after the `KCVSProxy` one-token fix. Candidates 1–7 carry forward and were **not**
re-verified this run: 1 and 7 were verified at this pinned commit on 2026-11-05, 2–4 and 6 on 2026-11-04,
and 5 still needs its experiment run.

1. **NEW, and the smallest the track has found: `KCVSProxy.java:67`** passes `txh` where every sibling
   method passes `unwrapTx(txh)`. One token. Fix the same omission in the test double at
   `KCVSCacheTest.java:242` and add a `KCVSCacheTest` case that calls `getKeys(MultiSlicesQuery, …)`
   through a cache and asserts the store received the unwrapped transaction — nothing currently does.
   Latent today (no caller above the storage layer), a `ClassCastException` the first time anything calls it.
2. **`RetryingTestExecutionExtension.java:77`** prints the wrong variable:
   `"Aborting test #" + minSuccess + " of " + maxInvocations` — `minSuccess` is the *required success
   count*, not the invocation index; `invocation` is in scope and used correctly in
   `evaluateExecutionCondition`'s messages. Also missing a space before `- still have retries left`.
3. **Replace the four empty concurrency overrides with `@Disabled("…")`** —
   `BerkeleyFixedLengthKCVSTest.java:32-40` and `BerkeleyVariableLengthKCVSTest.java:45-53`. Turns four
   reported passes into four reported skips with a stated reason.
4. **`BerkeleyVariableLengthKCVSTest.retrieveBackendAndGetKeyValue` cannot fail.** Body `L56-120`, outer
   `catch (BackendException e) { //generic failure }` at **`L113`** with no `fail`, no rethrow and no
   assertion; the inner try `L90-102` wraps the call its own comment at `L93` marks *"Errors out here
   without fix"* and catches the same exception with only `txh.rollback()`. Also imports JUnit 4
   `assertEquals` in a Jupiter suite and opens its own transactions instead of using `getTxConfig()`.
5. **NEW, needs one experiment first: `GraphDatabaseConfiguration.getBackend()`.** Run lesson 07's
   Hands-On §3 second half — call it on an already-open BerkeleyJE graph and record whether a second
   `Environment` opens, BDB's lock refuses it, or a silent second `Backend` appears
   (`jcmd <pid> Thread.print | grep ExpirationStoreCache` counts the cleanup threads). If it succeeds,
   that is a resource leak reachable from three call sites and worth an issue plus a fix to
   `JanusGraphEventualGraphTest:67`; if it fails, the fix is a rename.
6. **The reproduction test for [#4781](https://github.com/JanusGraph/janusgraph/issues/4781)**, specified
   by lesson 06. Second test still needed for the silent lock loss at
   `BerkeleyJEKeyValueStore.java:159-162`, which a fix touching only `beginTransaction` would miss.
7. **Issue, not a PR: the backoff formula.** `PERTURBATION_PERCENTAGE = 0.2` can only mean ±20% and the
   implementation cannot express it, but changing it changes retry timing for every deployment. File an
   issue with lesson 07's jshell reproduction and the measured attempt distribution, asking whether the
   intended behaviour is jitter around an exponential base or the current coin flip. **Check
   `git log -p --follow BackendOperation.java` first** — the `(int)` cast may predate `java.time.Duration`
   and have been harmless in a `long`-milliseconds version.
8. **NEW, smallest after candidate 1: `IDManager.fromVertexId`, `IDManager.java:632-636`** is missing
   `toVertexId`'s positivity check, so it accepts a temporary (negative) id and returns a negative
   "user id" instead of throwing. Precondition clause 1 uses `>>>` (a negative id's sign bit becomes
   magnitude, result comfortably positive); clause 2 compares the **signed** negative id against a
   positive bound (trivially true); the body uses `>>`, sign-extending. Verified at pb ∈ {0, 3, 5, 16}.
   At the production pb=5, `fromVertexId(getTemporaryVertexID(NormalVertex, 1))` returns
   **-36,028,797,018,963,968**; the legitimate round trip `fromVertexId(toVertexId(7)) == 7` is
   unaffected. `fromVertexId` is public, user-facing API (`docs/advanced-topics/custom-vertex-id.md`).
   The fix is one `checkArgument(id > 0, ...)` plus a `VertexIDAssignerTest` case — that test currently
   probes out-of-range with exactly `new long[] {0, idManager.getVertexCountBound()}`, never a negative.
   **File as an issue first**, not a PR: tightening a public method's input validation can break a
   caller relying on the current leniency. Attach lesson 08 §8's table.
9. **A comment, not code: `BackendTransaction`'s `EDGESTORE_MIN_KEY`/`EDGESTORE_MAX_KEY`.** Lesson 08 §9
   proves both bounds lie strictly outside the assignable key space (no proper `VertexIDType` suffix is
   `111`, no schema id ends in `11`, `count > 0` is enforced at `addPadding:347`). That proof is what
   makes the half-open scan interval safe and it exists nowhere in the tree. Smallest possible
   contribution; pairs naturally with candidate 10.
10. **Two documentation bugs, one trivial PR each.** (a) `docs/advanced-topics/data-model.md` states the
    64-bit vertex id *"is the key which points to the row containing the vertex's adjacency list"* and
    describes no transformation — true only at `partitionBits = 0`, which **no real graph uses** (see the
    `partitionBits = 5` finding above). One paragraph, plus a sentence about the partition rotation.
    (b) `docs/advanced-topics/custom-vertex-id.md`'s two groovy samples call
    `graph.getIDManager().toVertexID(123L)` and `graph.getIDManager().fromVertexID(long)` — capital-D
    `ID`, while the actual methods are `toVertexId` and `fromVertexId`. The surrounding prose in the same
    file spells `toVertexId(long)` correctly. The samples as written do not compile.

### Still open, and who owns it

- **Closed by lesson 08:** the id/key distinction and the full bit layout every Part II lesson needs;
  lesson 07's `EDGESTORE_MAX_KEY` question (unreachable, PR candidate 9); and the real value of
  `partitionBits` (5, not 0), which lessons 09-13 must assume rather than re-derive.
- **Owed to lesson 10 by lesson 08:** `TYPE_LEN_RESERVE = 3` and `MAX_PADDING_BITWIDTH = 6` are accounted
  for but not spent — `EdgeSerializer.writeRelation` spends them, with `SCHEMA_COUNT_BOUND = 1 << 54` as
  the budget. `IDHandler`'s **different** first-bit convention (per `getKeyID`'s javadoc at `:523-529`)
  must be reconciled there, and `IDManagementTest.edgeTypeIDTest`'s 1,000,000-trial `IDHandler` loop is
  the starting point. `BufferUtil.nextBiggerBuffer` / `padBuffer` also go to lesson 10.
- **Owed to lesson 19 by lesson 08:** the partitioned-vertex read cost — one logical vertex, up to
  `2^partitionBits` representative rows, one slice query each.
- **Owed to lesson 09 by lesson 08, in addition to the items below:** `StandardIDPool` per partition plus
  the dedicated schema and partitioned-vertex pools (`VertexIDAssigner:130-133`), `IDPlacementStrategy`
  injection at `:112`, and the `renewalThreads = 3 * partitionIdBound + 2` sizing at `:123` — three id
  kinds per partition plus schema plus partitioned vertices, a direct consequence of the ID layout.
- **A third site for the empty-override anti-pattern (candidate 3's family).**
  `InMemoryPartitionGraphTest` overrides `testPartitionSpreadFlushBatch`,
  `testPartitionSpreadFlushNoBatch` and `testKeyBasedGraphPartitioning` with empty bodies and reports
  them **passing** — the three tests that check partition spread are the three that don't run on the
  in-memory backend. Fold into candidate 3 when it is written, or send separately.
- **Closed by lesson 07:** the `Backend`/`BackendTransaction` wiring, the cache and locker wrapper stacks,
  and the retry semantics every later lesson inherits. Lesson 06's three questions about scan tests are
  answered by `BackendTransaction`'s `hasScan`/`hasOrderedScan` branches rather than by the iterators.
- **Reassigned by lesson 07:** `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299+`, shared mutable `current`, per-key limit at
  `L381`) plus `SimpleScanJob` (14,040 B) now belong with **lesson 22**. `MultiPageEntryBuffer`,
  `BufferPageUtils` and `BufferPage.merge` remain deferred with no owner; propose **lesson 19** when the
  vertex cache needs them.
- **Carried from lesson 04, now sharper:** the `getNoCopy` aliasing question belongs in
  `KeyColumnValueStoreTest` (gated by nothing, applying to every backend) and must mutate through a
  `CacheTransaction`, because `flushInternal` holds the caller's `Entry` objects across the whole flush and
  invalidates only afterwards. The `MemoryEntryList` sizing question and the
  `InMemoryColumnValueStore.java:162` NPE candidate move to **lesson 19**.
- **Carried from sessions 01–04:** 24 questions posed, none answered by Aryant. The half-open interval now
  has **three** implementations on record (`SinglePageEntryBuffer.getSlice` binary search;
  `BerkeleyJEKeyValueStore.java:258`'s per-record `compareTo`;
  `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange`) and lesson 07 adds a **fourth consumer** —
  `BackendTransaction`'s `EDGESTORE_MAX_KEY` relies on the upper bound being exclusive. Lesson 27 adds CQL
  as the fourth implementation. The contract is in `KeyColumnValueStore`'s javadoc and **none** of the
  enforcement is.
- **Owed to lesson 18:** `ExpectedValueCheckingTransaction.prepareForMutations()`, `ExpectedValueCheckingStore`,
  `ConsistentKeyLocker`, `BerkeleyJETx` (6,524 B), `BerkeleyJELocks`, `LockKeyColumnValueStoreTest`
  (23,021 B). Lesson 18 starts already knowing that two backend transactions are open per graph
  transaction and that `*ConcurrentTest.java` is excluded from `mvn test` at root `pom.xml:416`.
- **Owed to lesson 09:** `ConsistentKeyIDAuthority`, `IDAuthorityTest` (20,575 B), and lesson 06's note
  that `StorageSetup.getInMemoryConfiguration()` sets `IDAUTHORITY_WAIT = Duration.ZERO`, so everything
  passing in memory passes with ID-authority coordination latency removed. Add lesson 07's finding that the
  ids store is opened through the **unwrapped** manager with **no cache and no lock wrapper**.
- **Entry size.** Lesson 05 ≈35 KB, lesson 06 ≈63 KB, lesson 07 ≈78 KB (a large share of that is
  permalink URLs, which are not prose). In line with the SQLite and PostgreSQL lessons. The old 5–9 KB
  budget was a `contributor-reading` constraint and does not apply.
- **tdd.cat, 2026-11-06: the archive led the front page, and the resolution rule's exception fired.**
  At 09:00 IST the front page served **2026-11-04** (56 stories) while `/archive/` listed **2026-11-05**
  (62 stories) as latest with the 2026-11-06 cell unlinked — the archive *ahead* by one, the opposite of
  2026-11-05. `https://tdd.cat/2026-11-05/` fetched cleanly from the archive's own href, so lesson 08
  digested 2026-11-05 and said so in a blockquote at the top of the document. **The rule stands as
  written** (front page authoritative for "newest available", glance at `/archive/` only to notice a
  *larger* maximum) — today was exactly the case the exception clause covers, and it cost one extra
  fetch. Keep naming the edition in the document. Also: the 2026-11-05 page's filter bar rendered
  "No stories match the selected filters" while the header counted 62, the same glitch as 2026-11-04, so
  Must-Read/Recommended tags were unavailable again — worth one line and nothing more.
- **tdd.cat: the cheap endpoints are reliable but not always in agreement.** On 2026-11-05 the front page
  served **2026-11-04** (56 stories, source URLs intact on the first fetch) while `/archive/` reported a
  maximum of **2026-11-03** across 128 editions — the archive one *behind* the front page, the reverse of
  2026-11-03 and 2026-11-04 when the two agreed. **Resolution rule, settled: the front page is
  authoritative for "newest available"; the archive is a lagging index, not a tie-breaker.** The procedure
  stands: fetch the front page, digest whatever edition it serves, **say which one it is**, glance at
  `/archive/` only to notice a *larger* maximum, treat RSS as a third opinion. Do not spend entry space
  re-deriving an ordering. Rendering glitches continue to be worth a one-line note and nothing more:
  2026-11-03's item 41 paired a title with another story's summary; 2026-11-04's HN section rendered
  "No stories match the selected filters" while the header counted 56.

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
