<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 15
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-11 at the Part II/III boundary (lesson 13) — a59ae8d8 is still the tip of
  JanusGraph/janusgraph master, so there is nothing to re-pin to; staying. Not re-raised by lessons 14
  or 15 (both mid-Part, by design; list_commits was deliberately not run). Re-check at the Part III/IV
  boundary, after lesson 18, and re-run list_commits there rather than assuming either answer.
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
**Reviewed 2026-11-11 (lesson 13, the last item of Part II, so the Part II/III boundary was available for
the first time since lesson 08): `a59ae8d8` is still the tip of `master` — the three newest commits are
`a59ae8d8` (2026-10-28), `9a30ff14` (2026-10-28) and `5e3e7bc7` *JanusGraph release 1.2.0* (2026-10-24).
There is nothing to re-pin to, so the pin stays and the decision is closed rather than deferred. Next
review: the Part III/IV boundary, after lesson 18.**

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

## Session Log

Newest first. **Session-log roll-off, new this run:** the full `**Findings.**` paragraph is kept for the
three most recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons
09-12 were compressed to their headline finding on 2026-11-13; their full paragraphs remain verbatim in
git history at the commit each line names, and their substance is carried forward either in the surviving
`Verified by` blocks or in the root `README.md`'s per-lesson narrative. Next run, lesson 13's paragraph is
the one that goes.

- **15 — 2026-11-13** · [`2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md`](2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md) · **All three obligations lesson 14 handed this run are discharged.** Read `StandardJanusGraphTx` (whole, 2278 lines; written up for `:193-1135` and `:2137-2278`), `StandardTransactionBuilder` (whole, 759), `StandardJanusGraph` (whole, 1490; cited at `:570-612`, `:750-762`, `:1215-1240`), `GraphDatabaseConfiguration` (whole, 2192; cited at `:115-132`, `:664-700`, `:1060-1070`, `:2055-2070`), `ElementLifeCycle` (whole), `StandardVertex` (whole), `CaffeineVertexCache` (whole), `SimpleAddedRelations` (whole), `ConcurrentAddedRelations` (whole), `AddedRelationsContainer` (whole), `AddedPropertiesSingleValue` / `SetValue` / `ListValue` (all whole), `RemovedVertexTrackingTest` (whole), plus the directory listings for `graphdb/transaction/`, `.../addedrelations/`, `.../vertexcache/`, `graphdb/vertices/` and the in-memory test package. Also read [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) with `issue_read` and the [Transaction Handling](https://docs.janusgraph.org/basics/transactions/) doc page.
  **Findings.** Every data write funnels through `connectRelation:945` and fans out to **three** destinations (each endpoint's own container, the transaction's container, the vertex cache for non-new endpoints only) plus a fourth for properties (`addedPropertyIndex`). The per-transaction and per-vertex relation containers are **the same class with opposite `groupPropertiesByKey`**, which makes `propertiesSize` permanently 0 at transaction level and `propertiesContainer` permanently empty at vertex level, and means `getView(types, filter)` **ignores `types`** at transaction level. `ids.flush` defaults **true**, so a user vertex's temporary id lives for three statements; `makeSchemaVertex:1098` assigns unconditionally, so a schema vertex never has one. `ElementLifeCycle` is a six-state machine (three constants `private`) whose predicates are **range tests on the numeric ordering**, with `New` and `Modified` absorbing and `isModified()` excluding both `New` and `Removed`. `ConcurrentAddedRelations.getAllUnsafe()` is the one method **not** passed through `copyView` — it returns a live view, which is the whole meaning of the name, and `commit:2145` hands it to `graph.commit`. `MIN_VERTEX_CACHE_SIZE = 100` is a probability reduction, not a fix, for a non-atomic tier-2/tier-3 sequence in `CaffeineVertexCache.get` — **and the `#893` it cites is a 2018 Elasticsearch-TTL documentation PR, not a vertex-cache bug** (PR candidate 29). New PR candidates **29 and 30**. Daily Diff: the **2026-11-12** edition (66 stories) — PostgreSQL's 52-CVE security release and CVE-2026-6460, the composite-attribute type-confusion umbrella fix.

- **14 — 2026-11-12** · [`2026/11/2026-11-12-janusgraph-open-path-config-merge/README.md`](2026/11/2026-11-12-janusgraph-open-path-config-merge/README.md) · **Part III opens. All three obligations lesson 13 handed this run are discharged, and one of lesson 13's own notes is corrected.** Read `JanusGraphFactory` (whole), `GraphFactoryUtils` (whole), `SchemaInitializationManager` (whole), `GraphDatabaseConfigurationBuilder` (whole), `ReadConfigurationBuilder` (whole), `ModifiableConfigurationBuilder` (whole), `ConfigOption` (whole), `BasicConfiguration` (whole), `ModifiableConfiguration` (whole), `MixedConfiguration` (whole), `MergedConfiguration` (whole), `KCVSConfiguration` (whole), `KCVSConfigurationBuilder` (whole), `JanusGraphConstants` (whole), `janusgraph.internal.properties` (whole), `MetricInstrumentedSchemaCache` (whole), `ReadConfigurationBuilderTest` (whole), plus `GraphDatabaseConfiguration` (the option declarations 140-189, 1027-1030, 1157-1159, 1363-1375, 417-423; the class body 1604-1680; `getIDAssigner` 1934-1936, `getBackend` 1951-1956, `getSerializer` 1967-1981, `getTypeCache` 1983-1986, `getLocalConfiguration` 1988-1996, `preLoadConfiguration` 2034-2120, `selectExactConfig` 2122-2126), `StandardJanusGraph` (the constructor 264-360, `closeInternal` 422-470, `getSchemaCache` 554-556, `newTransaction` 588-603, `typeCacheRetrieval` 611-644, `getGlobalSystemConfig` 745-749), `Backend` (the constructor 236-280, `initialize` 333-417, `getGlobalSystemConfig` 478-479, `getStorageManager` 522-545), `ManagementSystem` (fields and constructor 145-188, `configVerifier` 190-207, `getOpenInstancesInternal` 209-213, `forceCloseInstance` 225-234) and `StandardSerializer` again (the registration table 113-171, for the 13 config datatypes).
  **Findings.** The global configuration is **one row** in `system_properties`, row key UTF-8 `"configuration"` (13 bytes), columns the dotted option paths, values `writeClassAndObject`; `hidden.frozen` is a column in that same row and is excluded from `getAll()` by a swallowed `ConfigElement.parse` failure logged at debug. `ConfigOption.ACCEPTED_DATATYPES` is a closed set of **13** classes whose registration numbers top out at 65, so **every config value's registration prefix is exactly one byte** — and the set is closed because `KCVSConfiguration:75` builds its own bare `new StandardSerializer()`, which no `attributes.custom.*` registration ever reaches. **Obligation (b), answered:** at open time `FIXED`, `GLOBAL_OFFLINE` and `GLOBAL` are **indistinguishable** — every open-path site uses `isGlobal()`/`isManaged()`/`isLocal()`, and the only two places in the tree that switch on the constant are `ModifiableConfiguration.set:37-44` and `ManagementSystem`'s `configVerifier:190-207`. The resolution order is three rows wide (`MergedConfiguration(overwrite, MixedConfiguration(global, local))`). **The MASKABLE surprise:** `getGlobalSubset` filters on `isGlobal()`, which *includes* `MASKABLE`, so every MASKABLE option in the first instance's properties file is frozen into the global row, never updated, never discrepancy-checked (`getManagedSubset` excludes MASKABLE), and inherited by every later instance that omits it — **PR candidate 26**. **Obligation (a), answered:** `system_properties` needs nothing but a `KeyColumnValueStoreManager`, which is why the open path creates store manager #1, reads and freezes the row through it, and **closes it** (try-with-resources → `KCVSConfiguration.close` → `txProvider.close` → `manager.close()`) before `getBackend()` opens store manager #2 and a *second* `KCVSConfiguration` on the same row; in the constructor, `schemaCache` is wired at `:308` but its retriever cannot run until `isOpen` at `:320`, and that invariant holds by ordering alone with no assertion or test. **Obligation (c), answered:** `MetricInstrumentedSchemaCache` is a `StandardSchemaCache` with a counting retriever — behaviour identical, no HIT counter, `MISS` incremented before the store call, `expireSchemaElement` uncounted, counters under the JVM-global system prefix; lesson 13's `isCacheable` asymmetry means a permanent 100% miss rate on repeated lookups of a non-existent type is correct rather than a misconfiguration. Also: six FIXED options exactly, three of them escapable via `graph.allow-upgrade` from a list parsed out of a jar resource (`storage-version,janusgraph-version,titan-version`; `STORAGE_VERSION=2`), and a successful upgrade writes `allow-upgrade=false` back into the database, disarming itself. **Correction to lesson 13's note:** `attributes.custom.*` is `GLOBAL_OFFLINE`, not FIXED. New PR candidates **26**, **27**, **28**. No syllabus revisions.

- **13 — 2026-11-11** · [`2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md`](2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md) · **Part II complete; re-pin reviewed and declined — `a59ae8d8` is still the tip of `master`, so there is nothing to re-pin to.** Read `BaseKey` (whole), `BaseLabel` (whole), `BaseRelationType` (whole), `EmptyRelationType` (whole), `SystemTypeManager` (whole), `Token` (whole), `JanusGraphSchemaCategory` (whole), `TypeDefinitionCategory` (whole), `TypeDefinitionMap` (whole), `TypeDefinitionDescription` (whole), `SchemaSource` (whole), `StandardRelationTypeMaker` (whole), `StandardPropertyKeyMaker` (whole), `JanusGraphSchemaVertex` (whole), `PropertyKeyVertex` (whole), `StandardSchemaCache` (whole), `TypeDefinitionDescriptionSerializer` (whole), `EnumSerializer` (whole), `StandardSerializer` again (the registration table, slots 41-57 and 63), `StandardSchemaCacheTest` (whole), plus `StandardJanusGraphTx` (`makeSchemaVertex` 1080-1112, `updateSchemaVertex` 1114-1116, `getSchemaVertex` 1216-1224, `getRelationType` 1231-1239, `getExistingRelationType` 1252-1262, the vertex constructor 641-664, `getOrLoadRelationTypeById` 488-497, `expireSchemaElement` 2253-2275, `newTypeCache` 282/337/341/1109/1217/2209), `StandardJanusGraph` (`typeCacheRetrieval` 611-643, `vertexExistenceQuery` 306, `getSchemaCache` 554-556, `edgeQuery` 682-685) and `IDManager` again (the bit-mask doc 36-62, the `VertexIDType` suffixes, `addPadding` 345-353, `getSchemaId` 691-694, `isSystemRelationTypeId` 717-719).
  **Findings.** The schema is a subgraph in the same `edgestore`: six system property keys and two system edge labels, whose ids are computed arithmetically (`(raw << 6) | 37` for system property keys, `| 53` for system edge labels) so there is no bootstrap read — `VertexExists`=101, `SchemaName`=2085, `SchemaDefinitionProperty`=2149, `SchemaCategory`=2213, `SchemaDefinitionDesc`=2277, `SchemaUpdateTime`=2341, `VertexLabelEdge`=181, `SchemaDefinitionEdge`=2357 (the last of those and `SchemaUpdateTime` share raw id 36 and are separated only by the suffix). A property key is 11 edgestore columns plus 2 graphindex entries; its definition is **exactly 7** `SchemaDefinitionProperty` columns, enforced by set *equality* in `TypeDefinitionMap.isValidDefinition`, and one ordinal byte (5 `DATATYPE` vs 6 `UNIDIRECTIONAL`) is the whole on-disk difference between a property key and an edge label. Property keys and edge labels deliberately share the `rt` namespace, which is why `getRelationType` hard-codes `EDGELABEL.getSchemaName(name)` and why the two kinds can never share a name. **Lesson 11's debt settled**: `getExistingRelationType` bottoms out at `SystemTypeManager`'s static 17-entry map for system ids and otherwise at the vertex constructor, which picks `PropertyKeyVertex`/`EdgeLabelVertex`/`VertexLabelVertex`/`JanusGraphSchemaVertex`/`CacheVertex` **from the id's suffix bits alone**, before any byte of the row is read. **Lesson 12's debt settled**: every `TypeDefinitionCategory` is persisted as its declaration-order ordinal via `EnumSerializer` (`VariableLong.writePositive`, one byte for all 25), with no version marker and only a disabled-by-default `assert` for type safety — the "Don't change the order" comment is the entire mitigation, and it applies to all seventeen registered enums. Also: `StandardSchemaCache`'s `isCacheable` asymmetry (an empty `SchemaDefinitionEdge` is cached, an empty definition is not, because the latter means "not visible here yet"); its two-bit `systemTypeId` field admitting only four cacheable system types; one-way primary-map overflow to the Caffeine backup; every cache miss costing a whole key-consistent transaction with the KCVS cache disabled; and a dead disjunct in `SystemTypeManager.throwIfSystemName` (the lowercased argument can never match the mixed-case system-name map; `Token.isSystemName` is what actually protects the namespace). New PR candidates **24** (fix that dead disjunct) and **25** (pin the ordinals of all seventeen persisted enums in a test). No syllabus revisions.
- **12 — 2026-11-10** · [`2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md`](2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md) · **stayed on `a59ae8d8`** · read `StandardSerializer`, the `Serializer`/`OrderPreservingSerializer`/`SupportsNullSerializer` interfaces, all seventeen registered attribute serializers, `NumericUtils`, `StringEncoding`, `VariableLong`, `VariableString`, `StandardRelationTypeMaker` and `SerializerTest`. **THE finding: `StandardSerializer` has four write entry points and `EdgeSerializer` chooses between them by *where in the column the value sits*, not by its type — and the whole difference is one flag byte.** Also: eleven order-preserving datatypes, three in-tree comparators that disagree, and PR candidate 20 rewritten rather than dropped. Daily Diff: the **2026-11-09** edition (50 stories) — Turso implementing Tantivy's `Directory` trait against its own B-tree. *(Full entry in git history at the 2026-11-10 commit; the facts later lessons need survive in the lesson-13 block below and in the root README's narrative.)*
- **11 — 2026-11-09** · [`2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md`](2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md) · **stayed on `a59ae8d8`** · read `EdgeSerializer` from the read side, `RelationCache`, `RelationReader`, `VariableLong`'s backward section, `IDHandler`, the `StaticArrayBuffer`/`ReadArrayBuffer` pair, `CacheEdge`, `RelationConstructor`, `ModificationDeserializer`, `EdgeSerializerTest` and `RelationReparseBenchmark`, plus commit `29c0b18a`. **THE finding: the backward encoding is the only genuinely order-preserving varint in the tree, and that — not readability — is why the tail ids are written backward.** `parseRelation` walks one cursor back and forth across a cell rather than streaming it. Daily Diff: the **2026-11-08** edition (22 stories) — a Postgres replication slot that cannot advance while its captured tables are idle. *(Full entry in git history at the 2026-11-09 commit.)*
- **10 — 2026-11-08** · [`2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md`](2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md) · **stayed on `a59ae8d8`** · read `EdgeSerializer` (whole, 591 lines), `IDHandler`, `VariableLong`, `EdgeDirection`, slices of `IDManager`, `Multiplicity`, `BufferUtil`, `StandardJanusGraph`, `ManagementSystem`, `RelationTypeVertex`, `BasicVertexCentricQueryBuilder` and the data-model doc. **THE finding: lesson 09's four conflict-avoidance bits cost one byte in every single cell of the database** — the relation-type prefix gets a 1-byte encoding only for a schema count of 7 or less, and the smallest count a default graph can assign is 16. `writeRelation` writes one buffer and returns one integer, `valuePosition`. Daily Diff: **no 2026-11-08 or 2026-11-07 edition existed** — one uncovered item from 2026-11-06 instead (Postgres `SKIP LOCKED` queues). *(Full entry in git history at the 2026-11-08 commit.)*
- **09 — 2026-11-07** · [`2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md`](2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md) · **stayed on `a59ae8d8`** · read `VertexIDAssigner` (whole, 545 lines), `StandardIDPool`, `IDBlockSizer`, `ConsistentKeyIDAuthority` (whole, 500 lines), `AbstractIDAuthority`, `ConflictAvoidanceMode`, `IDBlock`, `SimpleBulkPlacementStrategy`, `IDAuthorityTest` and the bulk-loading docs. **THE finding: `ids.authority.conflict-avoidance-tag-bits` defaults to 4, is `FIXED`, and its tag defaults to 0 with mode `NONE` — so every count JanusGraph assigns is a multiple of 16**, and the first `NormalVertex` id in a fresh default graph is **4096**, not 256. Coordination is amortised, not eliminated. Daily Diff: the **2026-11-06** edition (52 stories) — Infino's coordinator-free 128-bit ids. *(Full entry in git history at the 2026-11-07 commit.)*
- **08 — 2026-11-06** · [`2026/11/2026-11-06-janusgraph-id-bit-layout/README.md`](2026/11/2026-11-06-janusgraph-id-bit-layout/README.md) · **Part II opens; stayed on `a59ae8d8` deliberately** so Part II's byte offsets stay comparable with lessons 02-07. Read `IDManager` (whole, 754 lines), `BufferUtil`, `NumberUtil`, `VertexIDAssigner`, `ManagementSystem`, `IDManagementTest`, `JanusGraphPartitionGraphTest`, `InMemoryPartitionGraphTest`, `docs/advanced-topics/data-model.md`, `docs/advanced-topics/custom-vertex-id.md`. **THE finding: the vertex id and the edgestore row key are two different 64-bit words** — `constructId:450` builds `[0 | count | partition | suffix]`, `getKey:502` rebuilds it as `(partition << partitionOffset) | addPadding(count)`, rotating the partition to the **top**. Second: **every default graph runs at `partitionBits = 5`, not 0.** Third: **`fromVertexId` accepts a temporary (negative) id and returns garbage** (PR candidate 8). Closed lesson 07's `EDGESTORE_MAX_KEY` question (PR candidate 9). Daily Diff: no 2026-11-06 edition at the time; digested **2026-11-05** (62 stories) — GitHub's "Rubicon" Git storage architecture.
- **07 — 2026-11-05** · [`2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md`](2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md) · **Part I complete.** Read `Backend` (775 lines), `BackendTransaction` (568), `BackendOperation`, `CacheTransaction`, `KCVSCache`, `NoKCVSCache`, `KCVSProxy`, `ExpectedValueCheckingStoreManager`, `ExpirationKCVSCache`, `ExpirationCacheTest`, and slices of `GraphDatabaseConfiguration`. **THE finding: the retry backoff is a coin flip, not a ±20% jitter** — `(int)(2.0 / (1 + (rand*2-1)*0.2))` truncates to 1 or 2, measured median **14** attempts (max 38) against the 10 s read budget and **21** (max 48) against the 100 s write budget. Second: **`KCVSProxy.java:67` omits `unwrapTx`** (PR candidate 1). Daily Diff: the **2026-11-04** edition, 56 stories — PostgreSQL 19's `REPACK CONCURRENTLY`.
- **06 — 2026-11-04** · [`2026/11/2026-11-04-janusgraph-test-harness/README.md`](2026/11/2026-11-04-janusgraph-test-harness/README.md) · read the harness: `JanusGraphBaseTest`, `JanusGraphTest` (12,258 lines), `KeyColumnValueStoreTest`, the `@FeatureFlag` machinery, the BerkeleyJE suites, root `pom.xml`, `CONTRIBUTING.md`. Taught the src/main-not-src/test inversion, the one-abstract-hook-per-level contract, the **four** opt-out idioms and their four different reporting semantics, `@FeatureFlag`'s 4-of-24 coverage and its post-`@BeforeEach` ordering. Hands-on: read the Surefire XML rather than the console — 31 bare `<skipped/>` on `InMemoryGraphTest`, `testConcurrentGetSlice` as a 0 ms **pass**. Daily Diff: the **2026-11-03** edition, 52 stories — Spiral's *Statistics are aggregates*.
- **05 — 2026-11-03** · [`2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md`](2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md) · **first entry in `lesson` mode.** Read `OrderedKeyValueStoreManagerAdapter`, `OrderedKeyValueStoreAdapter`, `BerkeleyJEKeyValueStore`, `BerkeleyJEStoreManager`, `KVQuery`, `KeySelector`, `BaseKeyColumnValueAdapter`, both BerkeleyJE KCVS suites. Taught the fixed-vs-variable key-length split, why fixed needs no post-filter and variable does, the three byte layouts, the cursor walk, the batch that isn't, and BerkeleyJE's own two-tier locking. Daily Diff: the 2026-11-02 edition, 49 stories — Neumann's *The Benchmarking Crisis*.
- **04 — 2026-11-02** · [`2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md`](2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md) · read `InMemoryKeyColumnValueStore` L85–149, `InMemoryColumnValueStore` (whole), `SinglePageEntryBuffer` L36–101, `BufferPage` L39–175, plus `MemoryEntryList` and `BufferPageTest`. Six questions posed, none answered in the entry. Radar: PR #5018, #3864, #1128. **`My Notes` never filled — the fourth consecutive empty session, which triggered the mode switch.**
- **03 — 2026-11-01** · [`2026/11/2026-11-01-janusgraph-store-manager-features/README.md`](2026/11/2026-11-01-janusgraph-store-manager-features/README.md) · read `KeyColumnValueStoreManager`, `StoreManager`, `StoreFeatures` L31–257, `StandardStoreFeatures`, `KeyColumnValueStore` L185–227, `Backend` L249–287 + L339–343, `InMemoryStoreManager`, `InMemoryStoreManagerTest`, `FeatureRequirementCondition`. Six questions posed, none answered. Radar: #2268, #3390, #1128. **`My Notes` never filled.**
- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery`, `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, `KeyColumnValueStoreTest` L862–938, `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered. Radar: #3864, #3816, PR #4086. **`My Notes` never filled.**
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
- [x] 09 — `VertexIDAssigner` and the ID authority: block allocation from the ids store without per-insert coordination
- [x] 10 — `EdgeSerializer.writeRelation`: the column layout for edges and properties, direction, sort keys
- [x] 11 — `EdgeSerializer.parseRelation` and `RelationCache`: decoding columns back into relations
- [x] 12 — `StandardSerializer`: attribute encodings, order-preserving serializers, and custom types
- [x] 13 — System types and schema storage: schema vertices, `BaseKey`/`BaseLabel`, and how a property-key definition is persisted

### Part III — The write path (`core`, `graphdb/transaction`, `graphdb/database`, `diskstorage/locking`)

- [x] 14 — `JanusGraphFactory.open` → `GraphDatabaseConfiguration` → `StandardJanusGraph` constructor: local vs global (fixed/masked) config
- [x] 15 — `StandardJanusGraphTx`: `addVertex`, `addEdge`, property writes, added/deleted relation containers, and the vertex cache
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
2026-10-30.** Lesson 15 was handed three obligations by lesson 14 and discharged all three, so nothing
needed re-scoping. Part III's remaining three items (16, 17, 18) are mechanism, and lesson 15's
three-destination fan-out plus its two container tables are the input all three need.

**The re-pin decision stays closed until after lesson 18.** Lesson 13 reviewed it at the Part II/III
boundary and found `a59ae8d8` still the tip of `master`. Lessons 14 and 15 were both mid-Part, so it was
**not re-raised and `list_commits` was not run** in either — by design. The next review is the **Part
III/IV boundary, after lesson 18**, and it must re-run `list_commits` on `JanusGraph/janusgraph` `master`
rather than assuming either answer.

**The roll-off rule applied again this run.** `TRACK.md` keeps the notes for the three most recent
lessons; older ones roll off into git history. The three most recent are now 13, 14 and 15, so:
**lesson 12's compressed block is removed** (it remains verbatim in git history at the 2026-11-10 commit
and in compressed form at the 2026-11-11 and 2026-11-12 commits), **lesson 13's block is now compressed**
to the facts lessons 17, 18, 20 and 25 were promised — the full block is preserved verbatim in git
history at the 2026-11-11 commit — and lessons 14 and 15 keep their blocks in full. Next run, lesson 13's
block is the one that goes.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Lesson 16 (`StandardJanusGraph.commit` / `prepareCommit`) inherits three collections, not a blank
  page.** (a) `addedRelations.getAllUnsafe()` is a **live, uncopied view** over the transaction-level
  `SimpleAddedRelations(false)` — `ConcurrentAddedRelations` overrides it as `synchronized` but
  deliberately does **not** route it through `copyView` as it does every other view method; the
  interface javadoc says it "may only be invoked at the end of the transaction after there are no
  additional changes". `hppc`'s iterators are not fail-fast, so a violation is silent. (b)
  `deletedRelations` is a `Map<Long, InternalRelation>` keyed by `relation.longId()`, lazily allocated
  on first `removeRelation` of a **loaded** relation; a new relation removed in the same transaction is
  pulled out of `addedRelations` instead and never appears here. (c) `removedVertexCanonicalIds` is a
  `Set<Object>` of **canonical** ids (partition representatives collapsed), skipping new vertices
  entirely, and is what `wholeRowDeletionEnabled` consumes. Lesson 16 also owns the `upsert` flag set at
  `StandardJanusGraphTx:1035` and what `ChangedSchemaVertices.of(addedRelations, deletedRelations)`
  (`StandardJanusGraph:1236`) extracts.
- **Entry size.** Lesson 05 ≈35 KB, 06 ≈63 KB, 07 ≈78 KB, 08 ≈67 KB, 09 ≈61 KB, 10 ≈70 KB, 11 ≈56 KB,
  12 ≈91 KB, 13 ≈72 KB, 14 ≈75 KB, 15 ≈86 KB (a large share of each is permalink URLs, which are not
  prose). In line with the SQLite and PostgreSQL lessons. The old 5-9 KB budget was a
  `contributor-reading` constraint and does not apply.
- **Method note, carried, and lesson 15 is the third entry where simulation was not needed.** Lessons
  10-12 re-implemented the pinned encoding functions and ran them; lessons 13, 14 and 15 computed from
  the read sources instead and said so. Lesson 15's derived claims — that the transaction-level
  container's `propertiesSize` is **identically 0** (only `addProperty`/`removeProperty` mutate it and
  both are guarded by `groupPropertiesByKey`), that `edgesByLabel` is **never** built at transaction
  level (same guard), that `getView(types, filter)` ignores `types` at transaction level, and that
  `isModified()` is the range `3..5` and therefore excludes both `New` and `Removed` — are each read
  directly off the pinned sources with the derivation shown. **Keep the rule that produced them:**
  compute or re-implement rather than assert, say which, and always pair it with an exercise that runs
  the real code. Lesson 15's is `RemovedVertexTrackingTest`, the only test in the tree that casts to
  `StandardJanusGraphTx` and asserts on `isVertexFullyRemoved` directly, and whose second method is a
  regression test for the exact `it()`-resolution ordering in `StandardVertex.remove()`. One trap
  carried from lesson 12: model the layer above, not just the unit.
- **Daily Diff procedure — the max-of-two rule is no longer sufficient, and this run found out the hard
  way.** At ~03:40 UTC on 2026-11-13 the front page and `/archive/` **agreed**: both newest at
  **2026-11-11** (58 stories), archive reporting **136 editions**. But `https://tdd.cat/2026-11-12/`
  **exists and is complete** — 66 stories — so the front page and the archive can agree with each other
  and still both be a day behind. **The procedure is now:** (a) fetch the front page and `/archive/` and
  take the maximum of the two; (b) **also probe the dated URL for today and yesterday directly**
  (`https://tdd.cat/YYYY-MM-DD/`), because a published edition can be missing from both indexes; (c)
  before digesting, check this session log for the newest edition already covered — lesson 13 covered
  2026-11-10, lesson 14 covered 2026-11-11, lesson 15 covered 2026-11-12, so every run so far has found
  exactly one uncovered edition; (d) the uncovered-item fallback is still the last resort, and the
  2026-11-06 edition still has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). Edition sizes
  remain volatile — 22 stories on 2026-11-08, 50 on 11-09, 63 on 11-10, 58 on 11-11, 66 on 11-12 — so a
  small edition is not evidence of a partial publish. **The run fires at 03:30 UTC and the edition for
  that calendar day is not up yet**; every run so far has digested the previous day's. Say the edition
  date explicitly in the digest. One practical note carried from lesson 13 and confirmed again: a large
  edition's page can exceed 100 KB, which is about what one WebFetch returns, so the tail of the item
  list needs a second call with an offset (the 2026-11-12 page was 109,608 characters and needed
  `offset: 100000` for stories 61-66).

### Verified by lesson 15, for lessons 16, 17, 18, 19 and 25 to cite rather than re-derive

- **`connectRelation` (`StandardJanusGraphTx:945-960`) is the only door, and it opens onto four
  destinations:** (1) `r.getVertex(i).addRelation(r)` for every endpoint — the per-vertex container;
  (2) `addedRelations.add(r)` — the transaction container; (3) `vertexCache.add(vertex, vertex.id())`
  for endpoints where `!vertex.isNew()`; (4) `addedPropertyIndex.add(prop)` for properties only.
  `addVertex`, `addEdge` and `addProperty` are its only callers; `makeSchemaVertex` reaches it through
  `addProperty`.
- **One class, two configurations.** Transaction level is `SimpleAddedRelations(false)` /
  `ConcurrentAddedRelations(false)` (`:336`/`:340`); per-vertex is the same classes with `true`
  (`StandardVertex.addRelation`). All derived from that single `groupPropertiesByKey` guard: at
  transaction level `propertiesSize` is **identically 0**, `edgesByLabel` is **never built**, and
  `getView(Set<RelationType>, Predicate)` **ignores the type set**; at vertex level
  `propertiesContainer` is allocated with capacity 0 and **never written**, properties live in a
  `HashMap<String keyName, AddedPropertiesValue>`, and `edgesByLabel` is built once
  `edgesContainer.size() > MAX_EDGES_WITHOUT_LABEL_INDEX` (**32**, package-private, not a config
  option) with the existing 32 back-filled in that call. `isEmpty()` tests all three stores, so each
  configuration only ever exercises two clauses.
- **Properties are grouped by key *name* (a `String`)**; the upstream javadoc says a transaction that
  renames the key "changes [the grouping] under this one". Three strategies by `Cardinality`:
  `AddedPropertiesSingleValue` (one field; `addValue` returns **1 unconditionally**, `removeValue`
  nulls the slot with **no identity check**), `AddedPropertiesSetValue` (`HashMap<value, relation>`,
  true delta), `AddedPropertiesListValue` (`LinkedHashSet<>(20)`, true delta). **`isNull()` is a
  hardcoded `false` in `SetValue` and `ListValue`**, so `removeProperty`'s `propertiesMap.remove(key)`
  is dead code for `SET`/`LIST` — **PR candidate 30**. The `SINGLE` counter balances only because
  `addProperty` always removes the previous value first.
- **`ConcurrentAddedRelations` copies every view except the one that matters.** All methods
  `synchronized`; `getView`, `getViewOfProperties(Predicate)` and `getViewOfPreviousRelations` go
  through `copyView`; **`getAllUnsafe()` does not**, and the superclass returns an `AbstractCollection`
  delegating to the live `iterator()`. `commit:2145` passes it straight to `graph.commit`, and `hppc`
  iterators are not fail-fast. **Lesson 16 must not assume a snapshot.**
- **Two different "empty" containers.** `AddedRelationsContainer.EMPTY` (interface constant) **throws
  `UnsupportedOperationException`** from `add`/`remove` and is the per-vertex sentinel;
  `EmptyAddedRelations.getInstance()` is what `releaseTransaction:2205` swaps into the transaction.
- **`isSingleThreaded` is the only concurrency knob**, set by `threadBound()` alone
  (`StandardTransactionBuilder:146-150`). It decides six things: the `addedRelations` class, the
  `addedPropertyIndex` class, the `newTypeCache` map class (`HashMap` vs `NonBlockingHashMap`), whether
  `getLock` returns `FakeLock.INSTANCE` (`:831`) without touching `uniqueLocks`, and the map/set types
  of the three lazily-initialised fields (`deletedRelations` `:784-792`,
  `removedVertexCanonicalIds` `:808-816`, `uniqueLocks` `:832-840`). `graph.tx()` is thread-bound;
  `graph.newTransaction()` and `createThreadedTx()` are not.
- **`ids.flush` defaults to `true`** (`GraphDatabaseConfiguration:1064-1067`, MASKABLE), so a user
  vertex's temporary id is alive only between `StandardJanusGraphTx:686` and `:690`;
  `label.isPartitioned()` also forces immediate assignment. With `ids.flush=false` it survives to
  commit, where `idAssigner.assignIDs(addedRelations)` (`StandardJanusGraph:1226`) is handed **the
  relations, not the vertices** — which is why `addVertex:692` unconditionally adds `VertexExists`.
  **Lesson 13 confirmed:** `makeSchemaVertex:1098` assigns unconditionally and asserts `longId() > 0`.
- **`vertexCache.add` is the last statement of `addVertex` (`:697`), deliberately.** `connectRelation`
  skips new endpoints, so `:692`/`:695` never publish the new vertex; `:697` is the single publication
  point and keys the cache by **whatever id the vertex has then**.
- **`CaffeineVertexCache` is a bounded Caffeine cache plus an UNBOUNDED dirty map.**
  `volatileVertices` is a `NonBlockingHashMap` sized by `cache.tx-dirty-size` as an **initial capacity
  hint only**; `cache.tx-cache-size` (MASKABLE, **default 20000**) bounds only the clean cache. The
  removal listener rescues an evicted vertex iff `isTxOpen() && (isModified() || isRemoved())`; `add`
  puts into the dirty map iff `isNew() || hasAddedRelations()`. **Three predicates for three disjoint
  reasons to be dirty**, which is why `removeRelation:770-773` must call `vertexCache.add` explicitly
  for remove-only vertices. `get` is tier 1 (cache) → tier 2 (dirty map) → tier 3 (construct), and
  tier 2 before tier 3 is correctness-critical.
- **`cache.tx-dirty-size` has no declared default**; the fallback is batch-loading-dependent —
  `TX_DIRTY_SIZE_DEFAULT_WITH_BATCH = 4096`, `..._WITHOUT_BATCH = 32`
  (`GraphDatabaseConfiguration:691`, `:698`, resolved `:2060-2066`). Separately,
  `vertexCacheSize(int)` also sets `indexCacheWeight = size / 2`
  (`StandardTransactionBuilder:197-201`).
- **`ElementLifeCycle` is six states in one byte, three `private`**: `New` 1, `Loaded` 2,
  `AddedRelations` 3, `RemovedRelations` 4, `Modified` 5, `Removed` 6. The predicates are **range tests
  on the numeric ordering** — `isModified()` is `3..5`, so it excludes both `New` and `Removed`. In
  `update(byte, Event)`: `REMOVED` wins from **any** valid state including `Removed` (double-remove is
  a silent no-op, which is what makes `StandardVertex.remove()` idempotent), and `New` and `Modified`
  are **absorbing** for relation events. `StandardVertex` holds the byte `volatile` behind a
  **private** `lifecycleMutex`, not `this`.
- **The transaction-local locking vocabulary is complete and coordinates nothing beyond this
  transaction: `getLock` / `LockTuple` / `ReentrantTransactionLock` / `CombinerLock` / `FakeLock`, with
  a hardcoded `LOCK_TIMEOUT = Duration.ofMillis(5000L)` (`:199`) and no `ConfigOption`.** Tuples:
  `(out, type, in)` for `SIMPLE`; `(vertex, type, direction)` per constrained direction otherwise,
  combined by `CombinerLock` — `ONE2ONE` takes two locks and issues three storage queries.
  `uniqueLocks` **never shrinks** (upstream `TODO` at `:841-842`). A property write adds one
  `getLock(IndexLockTuple(index, match))` per unique composite index **record**, from
  `IndexRecordUtil.indexMatches(...)` (`:997-1000`). **Lesson 18 starts at `ConsistencyModifier.LOCK`
  — read at exactly one place, `:1030` — and `ExpectedValueCheckingStore`.**
- **`enableBatchLoading()` removes storage work, not just a flag check.** It calls
  `consistencyChecks(false)`, setting **both** `verifyUniqueness` and `acquireLocks` false
  (`StandardTransactionBuilder:169-173`, `:222-227`), so up to three uniqueness queries per edge
  disappear *and* the storage lock is never requested.
- **The blind-overwrite path, and the `upsert` flag lesson 16 inherits** (`:1024-1042`). For `single`
  or `set`, three conditions must **all** hold to skip reading the old value: (1)
  `!hasVerifyUniqueness()` **or** the key's `ConsistencyModifier != LOCK`; (2)
  `!TypeUtil.hasAnyIndex(key)`; (3) requested cardinality equals the key's declared one. Then only
  in-memory duplicates are removed (`getDuplicatedAddedRelation`, not a query) and
  `prop.setUpsert(true)` promises the commit path that a later delete must read the database.
  Otherwise `vertex.properties(key.name())` runs — a real slice query for a loaded vertex.
  **Adding any index to a key silently moves every write of that key onto the expensive path, with no
  log line and no metric. Lesson 17 owns what a "record" is.**
- **Auto-schema creation happens inside a data write.** `checkPropertyConstraintForVertexOrCreate...`
  (`:870-877`), `...ForEdgeOrCreate...` (`:879-886`) and
  `checkConnectionConstraintOrCreate...` (`:888-903`) return immediately on
  `hasDisabledSchemaConstraints()` and otherwise call `config.getAutoSchemaMaker()` mid-write. Two
  concurrent transactions auto-creating the same key both succeed locally; the `SchemaName` UNIQUE
  graphindex entry (lesson 13) rejects one at the storage layer at commit. **Lesson 18's territory,
  and why `schema.default=none` is the production recommendation.**
- **Obligation (a) from lesson 14, settled: nothing in `StandardJanusGraphTx`'s constructor can reach
  the backend.** All seven values read off `graph` (`:308-317`) are pre-built objects; `txHandle` is
  the only backend-facing field, is non-final, and `setBackendTransaction:381-384` asserts the one-shot
  contract. The constructor does **no I/O** — four containers, three `VertexConstructor`s, one
  anonymous `IDPool` (an `AtomicLong` starting at **1**, shared by vertices and relations, never
  released), conditionally two metrics wrappers. Its only guard is
  `Preconditions.checkArgument(graph.isOpen())` at `:306`, which with lesson 14's `isOpen = true` at
  `StandardJanusGraph:320` means no transaction can exist before the graph finishes wiring.
  `StandardTransactionBuilder`'s constructor (`:107-133`) reads **fifteen pre-resolved fields** off the
  `GraphDatabaseConfiguration`, no `Configuration` ladder, so building a transaction cannot fail on a
  missing option; the only live `Configuration` it keeps is `customOptions`, passed to
  `StandardBaseTransactionConfig` for the storage layer. **A transaction whose backend transaction
  fails to open is never registered** — `openTransactions.add(tx)` is at `StandardJanusGraph:593`,
  after `setBackendTransaction`; same shape as lesson 14's `Backend` leak, one level down.
- **Obligation (c), settled: `newTypeCache` resolves name → id, `vertexCache` resolves id → object**,
  and they meet only in `getSchemaVertex:1216-1225` (`newTypeCache` first, then
  `graph.getSchemaCache().getSchemaId`, then `vertexCache.get(schemaId, existingVertexRetriever)` —
  `verifyExistence == false`, `:347`, because the schema vertex may not be in storage yet).
  `newTypeCache` is written only at `makeSchemaVertex:1109` and emptied at `:2206`.
- **`releaseTransaction:2200-2211` sets `isOpen = false` first, then swaps eight fields to empty
  singletons.** `txHandle`, `config`, `graph`, `edgeSerializer`, `temporaryIds` and `relTypeCache` are
  **not** reset, which is why `rollbackIfOpen()` still works. `getVertices` defends against the swap
  racing a reader with `result.removeIf(v -> v == null || v.isRemoved())`, with an upstream comment
  naming `graph.close()` on another thread — `verifyOpen()` is a check, not a guarantee.
- **`recordRemovedVertex:804-818` keys by CANONICAL id** (partition representatives collapsed) and
  returns early for new vertices; `isVertexFullyRemoved:824-826` is what commit queries to choose a
  whole-row deletion — the consumer of lesson 14's `wholeRowDeletionEnabled` at
  `StandardJanusGraph:295`.
- **`StandardVertex` shortcuts every read for a new vertex.** `properties(String...)` answers from its
  own container with no query, filtering invisible properties when no keys are named — so
  `v.properties()` on a brand-new vertex does **not** show the `VertexExists` property
  `addVertex:692` just added. `loadRelations` returns `EntryList.EMPTY_LIST` when `isNew()`, and
  `hasLoadedRelations` is a flat `false`. `relTypeCache` (`:239`) is a `ConcurrentHashMap<>(30)`
  allocated only when `isLazyLoadRelations()`, populated by `getOrLoadRelationTypeById:486-496`.
  **Lesson 19 owes the `CacheVertex` side of all of these.**
- **`verifyWriteAccess`'s unmodifiable-vertex guard is three clauses:**
  `hasId() && isUnmodifiableVertex(id) && !isNew()` (`:390-398`). The `!isNew()` is what lets
  `makeSchemaVertex` write onto a vertex whose id already carries schema suffix bits; once committed,
  schema changes must go through `ManagementSystem` — **lesson 25.** `verifyAccess:400-409` enforces
  transaction affinity with one exception: `v instanceof SystemRelationType`, because system types are
  `SystemTypeManager` singletons belonging to no transaction. **`verifyAttribute:735-759`** is where
  lesson 12's registry is consulted at write time, once: `Object.class` keys check only
  `validDataType`; otherwise `attributeHandler.convert(datatype, attribute)` and a null result is a
  `SchemaViolationException`. Everything downstream sees the normalised value.
- **The `#893` in `MIN_VERTEX_CACHE_SIZE`'s comment (`:201-206`) points at the wrong thing.** Verified
  with `issue_read`: [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) is
  a PR opened 2018-01-17 by `davidclement90`, "Remove TTL support for Elasticsearch in
  documentation.", closed 2018-01-18, milestone `Release v0.2.1` — almost certainly a carried-over
  **Titan** id. **PR candidate 29**, and the real report must be found before substituting a number.
  The hazard is real: `CaffeineVertexCache.get`'s tier-2/tier-3 sequence is not atomic, so two threads
  of one multi-threaded transaction can both construct a vertex for the same id; the floor of **100**
  lowers the probability and does not serialise the sequence.

### Verified by lesson 14, for lessons 16, 17, 18 and 25 to cite rather than re-derive

- **The open path is nine frames and three of them do work:** `JanusGraphFactory.open` →
  `GraphFactoryUtils.defineSchemaAndStart` → **`GraphDatabaseConfigurationBuilder.build`** (the merge) →
  **`ReadConfigurationBuilder.buildGlobalConfiguration`** (the freeze and the checks) →
  `GraphDatabaseConfiguration` ctor + `preLoadConfiguration` (~45 fields resolved eagerly) →
  `SchemaInitializationManager.initializeSchemaAndStart` → **`new StandardJanusGraph`**.
- **One row, not a config file.** Store `system_properties`, row key UTF-8 `"configuration"` (13 bytes,
  no id encoding), columns the full dotted option paths in UTF-8, values `writeClassAndObject`. A second
  row `"userconfig"` in the same store holds `Graph.Variables` (`Backend:412`). `hidden.frozen`
  (`BasicConfiguration:37`) is a column in the configuration row; it fails `ConfigElement.parse`, so
  `getAll()` swallows the exception at debug and the marker never appears as an option.
- **Every config value's registration prefix is one byte.** `ConfigOption.ACCEPTED_DATATYPES` is 13
  classes — `Byte` 10, `Short` 11, `Integer` 12, `Long` 13, `Boolean` 15, `String` 18, `Float` 19,
  `Double` 20, `String[]` 30, `TimestampProviders` 54, `ConflictAvoidanceMode` 57, `Duration` 64,
  `Instant` 65 — all below 128. The set is closed by a bootstrap constraint, not taste:
  `KCVSConfiguration:75` builds its own bare `new StandardSerializer()`, and `ConfigOption`'s static
  block (`:96-98`) refuses to let the class load unless all 13 are `validDataType` on one. **Enums not
  in that set must be declared `ConfigOption<String>` and converted by
  `GraphDatabaseConfiguration.selectExactConfig:2122-2126`** — `has-step-mode`, `repeat-step-mode`,
  `schema.default` and the index-selection strategy all take that route.
- **Five types, three predicates, and that is the whole type system.** `isGlobal()` = everything but
  `LOCAL`; `isManaged()` = `{FIXED, GLOBAL_OFFLINE, GLOBAL}`; `isLocal()` = `{MASKABLE, LOCAL}`.
  Counts in `GraphDatabaseConfiguration.java`: FIXED **6**, GLOBAL_OFFLINE **15**, GLOBAL **8**,
  MASKABLE **88**, LOCAL **34**.
- **At open time FIXED, GLOBAL_OFFLINE and GLOBAL are indistinguishable.** Only two sites in the tree
  switch on the constant, both post-open: `ModifiableConfiguration.set:37-44`
  (`hasUpgradeableFixed(name) || !isFixed() || !isFrozen()`) and `ManagementSystem`'s
  `configVerifier:190-207`, which rejects FIXED (unless upgradeable) and LOCAL outright and, for
  `GLOBAL_OFFLINE`, requires exactly one open instance *which must be this one* and then sets
  `graphShutdownRequired = true` at `:204`. **Lesson 18 and lesson 25 both need this.**
- **Resolution order, three rows wide**, from `MergedConfiguration.get:39-47` over
  `MixedConfiguration.get:49-58`: managed → overwrite, global row, default (**the local file is never
  consulted**); MASKABLE → overwrite, local file, global row, default; LOCAL → overwrite, local file,
  default. `overwrite` is not user-settable — only `LOCK_LOCAL_MEDIATOR_GROUP`, `UNIQUE_INSTANCE_ID`,
  the tx/mgmt log forcings and the Titan id-store fallback write to it.
- **`localConfiguration` (`GraphDatabaseConfigurationBuilder:72-73`) is NOT in the lookup path.** It is
  an `isLocal()`-filtered projection kept only so `getLocalConfiguration()` can hand TinkerPop a
  `Configuration` with `gremlin.graph` set. Do not read it as "the local config" when debugging.
- **The MASKABLE leak.** `getGlobalSubset` (`ReadConfigurationBuilder:172-178`) filters on `isGlobal()`,
  which includes MASKABLE, so first-open persists every MASKABLE option from the properties file into
  the global row; it is never refreshed (that branch runs only when not frozen) and never
  discrepancy-checked (`getManagedSubset:180-186` filters on `isManaged()`). A later instance that omits
  the option gets the **first instance's value**, not the code default, with no log line. **PR
  candidate 26.**
- **Six FIXED options:** `graph.timestamps`, `graph.janusgraph-version` (hidden),
  `graph.storage-version`, `graph.titan-version` (hidden), `cluster.max-partitions` (default 32, power
  of 2 > 1), `ids.authority.conflict-avoidance-tag-bits` (default 4, range 0-16).
  `JanusGraphConstants.UPGRADEABLE_FIXED` is parsed at class load from the jar resource
  `janusgraph.internal.properties`: `storage-version,janusgraph-version,titan-version`, and
  `janusgraph.storage-version=2`. **The other three can never be changed by anything.** Matching is on
  `option.getName()` — the leaf name, not the path.
- **A successful upgrade disarms itself:** `janusGraphVersionsWithDisallowedUpgrade:132-137` writes
  `allow-upgrade=false` into the global row along with the three version values, and the equal-version
  path emits a `log.warn` telling you to turn it off locally too.
- **The discrepancy check compares against the DEFAULT when the row has no entry**
  (`getOptionsWithDiscrepancies:262-267`), so setting a managed option locally to a non-default value on
  a graph where it was never written fails the open even with no real conflict. `overwrite` wins the
  comparison (`:270-273`), which is why `LOCK_LOCAL_MEDIATOR_GROUP` does not trip it.
  `graph.allow-stale-config` downgrades **all** such errors to warnings; there is no per-option escape.
  Both `allow-stale-config` and `allow-upgrade` are read by a hand-rolled local → global → default
  ladder (`:236-251`), because `MixedConfiguration` does not exist yet at that point.
- **Two store managers per open, and the first is closed before the second exists.**
  `GraphDatabaseConfigurationBuilder:64` creates manager #1 from the **local file alone**;
  `KCVSConfigurationBuilder:38-55` opens `system_properties` on it with a `TransactionalProvider` whose
  `close()` is `manager.close()` (`:49`); `ReadConfigurationBuilder:70` holds it in try-with-resources,
  so `KCVSConfiguration.close():225-234` closes the store **and** the manager. `Backend.initialize:384-399`
  then opens `system_properties` again through `storeManagerLocking` and builds a **second**
  `KCVSConfiguration` on the same row, this one with a deliberately empty `close()`. Consequence: any
  MASKABLE or GLOBAL storage option whose authoritative value is in the row is **not applied to the
  manager that reads the row**.
- **Reads of the row are whole-row snapshots.** `asReadConfiguration()` calls `toMap()` **once**
  (`:175-195`, slicing `[zeroBuffer(1), oneBuffer(128))`) and closes over the resulting `HashMap`.
  Nothing re-reads it for the life of the graph — which is precisely why GLOBAL_OFFLINE must force a
  shutdown and FIXED must be refused.
- **`KCVSConfiguration.set(key, value, expectedValue, true)` calls `store.acquireLock(rowKey, column,
  expectedValueBuffer, txh)` at `:153` before `store.mutate`.** The config row participates in the same
  compare-and-set primitive as a composite index entry. **Lesson 18 should start here.**
- **`getOpenInstancesInternal` (`ManagementSystem:209-213`) reads the open-instance set out of the same
  config row** — `getContainedNamespaces(REGISTRATION_NS)`, i.e. the `graph.instance.[uid].startup-time`
  entries that `StandardJanusGraph:341` writes. There is no separate registry, so the GLOBAL_OFFLINE
  "no other instance is open" guard is only as accurate as the last clean shutdown.
- **`StandardJanusGraph` construction order** (`:264-360`): config `:266` → cdc filters `:267-288` →
  **`backend = getBackend()` `:289`** → `name` `:291` → `idAssigner`/`idManager` `:293-294` →
  `wholeRowDeletionEnabled` `:295` → `cacheInvalidationService` `:297` → `serializer` `:301`
  (**where `attributes.custom.*` is applied**) → `edgeSerializer` `:302` → `storeFeatures` `:303` →
  `indexSerializer` `:304` → `vertexExistenceQuery` `:306` → `queryCache` `:307` →
  **`schemaCache` `:308`** → `times`/`indexSelector` `:309-310` → `scriptEngine` `:312-318` →
  **`isOpen = true` `:320`** → `computerJobs` `:326` → instance registration `:332-341` →
  `managementLogger` + `registerReader` `:343-346` → metrics/shutdown hook `:351-358`.
- **`:308` before `:320` is safe by ordering alone.** `typeCacheRetrieval` is a field initializer
  (`:611-644`) and so exists before the constructor body, but both its methods call `newTransaction`,
  which throws `graphShutdown` while `isOpen` is false. Nothing between `:308` and `:320` touches the
  cache; the first code that can is `managementLog.registerReader` at `:346`. **There is no assertion
  and no test pinning this.**
- **A constructor failure after `:289` leaks the `Backend`.** Only `:351-358` is inside a
  `try`/`catch (RuntimeException | Error)` that calls `closeInternal()` and rethrows.
- **`getBackend()` (`GraphDatabaseConfiguration:1951-1956`) has no memoisation and no guard**, and is
  reachable from three call sites, one of them configuration-driven: `SchemaInitializationManager:45-53`
  constructs a whole `StandardJanusGraph` and hands it to `JanusGraphFactory.drop`, which calls
  `getBackend()` again at `:228`. **PR candidate 5's experiment is now worth more than it was.**
- **`MetricInstrumentedSchemaCache` counts and nothing else** — a `StandardSchemaCache` wrapping a
  retriever that increments MISS before delegating (`:36-53`). No HIT counter (hits are
  RETRIEVAL − MISS); a failed retrieval still counts as a miss; `expireSchemaElement` is uncounted;
  counters live under `METRICS_SYSTEM_PREFIX_DEFAULT`, so **two graphs in one JVM share them**. Lesson
  13's `isCacheable` asymmetry means a permanent 100% miss rate on repeated lookups of a type that does
  not exist is the correct reading, not a misconfiguration.
- **Correction to lesson 13's note:** `attributes.custom.[X].attribute-class` and `serializer-class`
  (`GraphDatabaseConfiguration:1369-1375`) are **`GLOBAL_OFFLINE`, not FIXED**. They can be changed
  through `ManagementSystem` with every other instance shut down, which then forces this instance's
  shutdown.
- **The freeze is not atomic.** `getGlobalSubset` + three `setup*` calls + `freezeConfiguration()` is
  N+4 independent `BackendOperation.execute` calls. A crash between them leaves version entries with no
  `hidden.frozen`, and the next open dies on
  `Preconditions.checkArgument(!globalWrite.has(...), "Database has already been initialized but not
  frozen")` with no repair path. The only fix is clearing the store.

### Verified by lesson 13 — compressed; full block in git history at the 2026-11-11 commit

- **There is no catalog.** The schema is a subgraph in the same `edgestore`, written by the same
  `EdgeSerializer` and read back by the same `parseRelation`. `BaseKey` declares **six** system property
  keys and `BaseLabel` **two** system edge labels, and that is the entire vocabulary. System type ids
  are **computed, never stored**, by `IDManager.getSchemaId(type, raw)` = `(raw << offset()) | suffix()`:
  `VertexExists` **101**, `VertexLabelEdge` **181**, `SchemaName` **2085**, `SchemaDefinitionProperty`
  **2149**, `SchemaCategory` **2213**, `SchemaDefinitionDesc` **2277**, `SchemaUpdateTime` **2341**,
  `SchemaDefinitionEdge` **2357**. `SystemTypeManager` asserts `SYSTEM_TYPES_BY_ID.size()==17`.
- **`TypeDefinitionCategory` has 25 constants, ordinals 0-24**, property categories 0-16 and edge
  categories 17-24 (`RELATIONTYPE_INDEX`, `TYPE_MODIFIER`, `INDEX_FIELD`, `INDEX_SCHEMA_CONSTRAINT`,
  `PROPERTY_KEY_EDGE`, `CONNECTION_EDGE`, `UPDATE_CONNECTION_EDGE`, `INDEX_INLINE_KEY`). **Lesson 17
  owns the eight edge categories.** Definition sets are checked by set **EQUALITY**, not containment
  (`TypeDefinitionMap.isValidDefinition` asserts `keys.size() == requiredTypes.size()`): `PROPERTYKEY`
  = 7, `EDGELABEL` = 7, `GRAPHINDEX` = 6, `VERTEXLABEL` = 2 (**no `STATUS`**), `TYPE_MODIFIER` = 1.
  `CONSISTENCY_LEVEL` and `TTL` are in **no** list — they live on separate `TYPE_MODIFIER` vertices
  reached by a definition edge, which is **lesson 18's** concern.
- **One ordinal byte is the whole on-disk difference between a property key and an edge label**: 5
  (`DATATYPE`) against 6 (`UNIDIRECTIONAL`). **A property key's row is 11 edgestore columns** —
  `VertexExists`, `SchemaName`, `SchemaCategory`, `SchemaUpdateTime`, and 7 ×
  `SchemaDefinitionProperty` — **plus 2 graphindex entries** (UNIQUE on `SchemaName`, STANDARD/LIST on
  `SchemaCategory`). **Lesson 17 can cite this as the cost baseline.**
- **`TypeDefinitionDescription` is registration 63 and its payload is `[category ordinal][modifier]`.**
  For a property category the modifier must be null, so every tag on a property key's definition is
  **exactly two bytes: `[ordinal][0x00]`**. Only `INDEX_FIELD` (`Parameter[]`) and `CONNECTION_EDGE`
  (`String`) carry a real modifier — **lesson 17 again.**
- **Every persisted enum is positional and unversioned.** `EnumSerializer.write` is
  `VariableLong.writePositive(ordinal)`, one byte below 128, and `getValue` validates only the RANGE;
  `TypeDefinitionMap.setValue`'s type check is an `assert`, disabled in production. Inserting a constant
  mid-list silently reinterprets stored schema. The comment at `TypeDefinitionCategory:41` is the entire
  mitigation. **PR candidate 25.**
- **`BaseKey.SchemaName`'s internal composite index declares `Index.UNIQUE` with
  `ConsistencyModifier.LOCK` and `getBackingIndexName() == Token.INTERNAL_INDEX_NAME`, so schema-name
  uniqueness rests on `ConsistentKeyLocker` — lesson 18.** Enumeration by category goes through
  `QueryUtil.getVertices(tx, BaseKey.SchemaCategory, <category>)`.
- **`StandardSchemaCache`: two maps, both one-way.** `typeNames` (name → id) and `schemaRelations`
  ((schema id, system type, direction) → raw `EntryList`), each a `volatile` primary plus a bounded
  Caffeine backup; exceeding `MAX_CACHED_TYPES_DEFAULT = 10000` (or `×3` for relations) sets the primary
  to `null` **permanently**. The relation key packs `systemTypeId` into **two bits**, so only four system
  types are cacheable (`SchemaDefinitionEdge` 0, `SchemaName` 1, `SchemaCategory` 2,
  `SchemaDefinitionProperty` 3). **Lesson 25 owes `JanusGraphSchemaVertex.resetCache()` and
  `expireSchemaElement`'s interaction with it.**
- **The Java class of every vertex comes from the id's suffix bits**, in the vertex constructor at
  `StandardJanusGraphTx:641-664` — `PropertyKeyVertex`, `EdgeLabelVertex`, `VertexLabelVertex`,
  `JanusGraphSchemaVertex`, `CacheVertex`/`PreloadedVertex` — before a byte of the row is read.
  `PreloadedVertex` is chosen by the `createStubVertex` flag on the same branch. **Lesson 19.**
- **Cardinality is a view; multiplicity is the stored concept.** There is no `CARDINALITY` category;
  `StandardPropertyKeyMaker.cardinality` converts in and `PropertyKeyVertex.cardinality()` converts out.
  Only two defaults exist in `TypeDefinitionCategory.defaultValue`: `SORT_ORDER → Order.ASC` (dead for
  relation types) and `STATUS → SchemaStatus.ENABLED` (live; vertex labels store no `STATUS`).
- **A reachability bug in the only name guard:** `throwIfSystemName` lowercases its argument before
  `isSystemType(name)`, whose first disjunct checks a map keyed by mixed-case names (`~T$VertexExists`),
  so that disjunct can never match from this call site. `Token.isSystemName`'s `~` check is what actually
  protects the namespace. **PR candidate 24.**

### PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 29-30 are new this run** (lesson 15); 26-28 came from lesson 14,
24-25 from lesson 13 and 21-23 from lesson 12, which also **rewrote** candidate 20 rather than dropping
it. **18 is still the most substantial the track has found. The best first contributions are now 29, 28,
21 and 24** — 29 is a one-line comment correction, 28 is a one-line error message with no behavioural
risk, 21 is doc-only and 24 is a two-line guard fix, and all four force a maintainer to confirm a
reading. Candidates 1-28 carry forward; 29-30 were verified at this pinned commit on 2026-11-13, 26-28 on
2026-11-12, 24-25 on 2026-11-11, 21-23 on 2026-11-10, 18-20 on 2026-11-09, 15-17 on 2026-11-08, 11-14 on
2026-11-07, 8-10 on 2026-11-06, 1 and 7 on 2026-11-05, 2-4 and 6 on 2026-11-04, and **5 still needs its
experiment run — and is now worth more**, because lesson 14 found a second, configuration-driven call
site for `getBackend()`.

1. **`KCVSProxy.java:67`** passes `txh` where every sibling method passes `unwrapTx(txh)`. One token.
   Fix the same omission in the test double at `KCVSCacheTest.java:242` and add a `KCVSCacheTest` case
   that calls `getKeys(MultiSlicesQuery, …)` through a cache and asserts the store received the
   unwrapped transaction — nothing currently does. Latent today (no caller above the storage layer), a
   `ClassCastException` the first time anything calls it.
2. **`RetryingTestExecutionExtension.java:77`** prints the wrong variable:
   `"Aborting test #" + minSuccess + " of " + maxInvocations` — `minSuccess` is the *required success
   count*, not the invocation index; `invocation` is in scope and used correctly in
   `evaluateExecutionCondition`'s messages. Also missing a space before `- still have retries left`.
3. **Replace the four empty concurrency overrides with `@Disabled("…")`** —
   `BerkeleyFixedLengthKCVSTest.java:32-40` and `BerkeleyVariableLengthKCVSTest.java:45-53`. Turns four
   reported passes into four reported skips with a stated reason. Fold in the third site lesson 08
   found: `InMemoryPartitionGraphTest` overrides `testPartitionSpreadFlushBatch`,
   `testPartitionSpreadFlushNoBatch` and `testKeyBasedGraphPartitioning` with empty bodies and reports
   them **passing** — the three tests that check partition spread are the three that don't run on the
   in-memory backend.
4. **`BerkeleyVariableLengthKCVSTest.retrieveBackendAndGetKeyValue` cannot fail.** Body `L56-120`, outer
   `catch (BackendException e) { //generic failure }` at **`L113`** with no `fail`, no rethrow and no
   assertion; the inner try `L90-102` wraps the call its own comment at `L93` marks *"Errors out here
   without fix"* and catches the same exception with only `txh.rollback()`. Also imports JUnit 4
   `assertEquals` in a Jupiter suite and opens its own transactions instead of using `getTxConfig()`.
5. **Needs one experiment first: `GraphDatabaseConfiguration.getBackend()`.** Run lesson 07's
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
8. **`IDManager.fromVertexId`, `IDManager.java:632-636`** is missing `toVertexId`'s positivity check, so
   it accepts a temporary (negative) id and returns a negative "user id" instead of throwing.
   Precondition clause 1 uses `>>>`, clause 2 compares the **signed** negative id against a positive
   bound, the body uses `>>`. Verified at pb ∈ {0, 3, 5, 16}. At the production pb=5,
   `fromVertexId(getTemporaryVertexID(NormalVertex, 1))` returns **-36,028,797,018,963,968**; the
   legitimate round trip `fromVertexId(toVertexId(7)) == 7` is unaffected. `fromVertexId` is public,
   user-facing API (`docs/advanced-topics/custom-vertex-id.md`). The fix is one
   `checkArgument(id > 0, ...)` plus a `VertexIDAssignerTest` case — that test currently probes
   out-of-range with exactly `new long[] {0, idManager.getVertexCountBound()}`, never a negative.
   **File as an issue first**, not a PR: tightening a public method's input validation can break a
   caller relying on the current leniency. Attach lesson 08 §8's table.
9. **A comment, not code: `BackendTransaction`'s `EDGESTORE_MIN_KEY`/`EDGESTORE_MAX_KEY`.** Lesson 08 §9
   proves both bounds lie strictly outside the assignable key space, and lesson 09 completes the
   `MIN_KEY` half with `BASE_ID = 1`. That proof is what makes the half-open scan interval safe and it
   exists nowhere in the tree. Smallest possible contribution; pairs naturally with candidate 10.
10. **Two documentation bugs, one trivial PR each.** (a) `docs/advanced-topics/data-model.md` states the
    64-bit vertex id *"is the key which points to the row containing the vertex's adjacency list"* and
    describes no transformation — true only at `partitionBits = 0`, which **no real graph uses**. One
    paragraph, plus a sentence about the partition rotation. (b)
    `docs/advanced-topics/custom-vertex-id.md`'s two groovy samples call
    `graph.getIDManager().toVertexID(123L)` and `graph.getIDManager().fromVertexID(long)` — capital-D
    `ID`, while the actual methods are `toVertexId` and `fromVertexId`. The samples as written do not
    compile.
11. **NEW, and the smallest the track has found: `ConsistentKeyIDAuthority.java:420-421`** formats the
    timeout with `timeout.getNano()`, the nanosecond-of-second component, not the total. The timeout
    reaching `claimIDBlock` is `ids.renew-timeout`, default `Duration.ofMillis(120000)`, whose
    `getNano()` is **0** — measured under OpenJDK 21, along with `PT2S` → 0, `PT0.3S` → 300000000,
    `PT1.5S` → 500000000. So the one message emitted when ID allocation finally gives up, which per the
    `IDS_RENEW_TIMEOUT` javadoc fails the entire database, reads *"Reached timeout 0"* on every
    whole-second configuration including every default deployment. Fix is `timeout` or
    `timeout.toMillis()` with the format specifier adjusted. Zero behavioural risk. **Send this one
    first.**
12. **`ConsistentKeyIDAuthority.java:392-399`: a comment that describes unimplemented intent.** The
    rollback path builds an anonymous `BackendOperation.TransactionalProvider` commented *"Use normal
    consistency level for these non-critical delete operations"*, whose `openTx()` body is
    `manager.beginTransaction(storeTxConfigBuilder.build())` — character-for-character `openTx()` at
    `:223-225`. `storeTxConfigBuilder` had the key-consistent custom options baked in at construction
    (`:189`, `:196`, `:198`) and is never re-derived, so the "normal" consistency level is the critical
    path's consistency level. The anonymous class differs from `this` only in a no-op `close()` that
    `BackendOperation.execute` never calls. Either implement the intent (derive a non-key-consistent
    config) or delete the comment and the anonymous class and pass `this`. **File as an issue** — which
    of the two is right is a maintainer's call.
13. **Two dead assertions in `IDAuthorityTest`, plus the parameterization that would make them live.**
    `checkBlock:207-209` (a block's ids are evenly spaced) and `testSimpleIDAcquisition:247-251`
    (consecutive blocks abut, with `previous` write-only as a result) are both gated on
    `hasEmptyUid == (uidBitWidth == 0)`, and `configs():84-103` produces exactly three configurations
    with tag-bits **4** (default), **9** and **7**. **Neither assertion has ever executed in any run of
    any backend's `IDAuthorityTest`.** Nothing in the suite checks the two properties `getId`'s shift is
    responsible for. The fix is a fourth `configs()` entry with `IDAUTHORITY_CAV_BITS = 0`; the
    assertions then run as written and the commented-out `IDAUTHORITY_RANDOMIZE_UNIQUEID` lines at `:99`
    and `:135` can go at the same time.
14. **Two small things in the id-assignment hot path, one PR.** (a)
    `VertexIDAssigner.PartitionIDPool.accessed()` (`:378`, `:533-535`) writes
    `lastAccess = System.currentTimeMillis()` on a `volatile long` for **every element id assigned**, and
    its only reader `getLastAccess():537-539` has **no caller in the tree** (code-searched at the pinned
    commit). Measured **33.0 ns/op** against 0.30 ns for an empty loop, 50M iterations after warmup under
    OpenJDK 21 — roughly 1.3 s on a 10M-vertex, 3-relations-each load, for a field nothing reads. Delete
    the field, `accessed()` and `getLastAccess()`. (b) `SimpleBulkPlacementStrategy:58` uses one shared
    `java.util.Random` for every placement decision on every thread (a contended CAS on its seed) while
    `ConsistentKeyIDAuthority:246` uses `ThreadLocalRandom` for the same kind of draw; and the class
    javadoc (`:37-45`) claims the strategy "assigns all vertices created in a transaction to the same
    partition id", which is true of `getPartitions(Map):100-105` and **false** of
    `getPartition(element):95-97`, which re-rolls per call and is the path every relation takes. Fix the
    `Random`, fix the javadoc. `currentPartitions` being a non-volatile `int[]` written from any thread
    is worth a sentence in the PR body but not a change — a stale read is still a valid partition.

15. **A test for the relation-type ordering contract.** `IDManagementTest.edgeTypeIDTest:141-201` runs
    1,000,000 trials and asserts the round trip, the `DirectionID`, full buffer consumption, the inline
    round trip, `DataOutput`-vs-`getRelationType` equality, and both `getBounds` bracketings — and never
    compares two **different** type prefixes to each other. So the actual ordering property is unpinned:
    the encoding is monotone *within* a byte-length class and **inverts across one**. Add a case asserting
    exactly that, with the two concrete pairs lesson 10 verified — `count 1023` (`7f fe`) vs `count 1024`
    (`70 10 80`), and the default-stride pair `1008` vs `1024` — so "columns are grouped by type, not
    sorted by type id" is documented in the suite rather than folklore. Phrase the assertions as the
    contract (`compareTo(...) > 0` with a comment naming the STOP-bit cause), not as a bug. The same patch
    can fix `int trails = 1000000` at `:145` (`trails` → `trials`). **Smallest-risk candidate on the list
    after 11** — pure test addition, no production change.
16. **One paragraph in `docs/advanced-topics/data-model.md`.** The doc's "Individual Edge Layout" section
    (L66-88) describes the relation-type id, the direction bit and the sort key correctly, and says nothing
    about the **six prefix bands** — which are the single most useful thing to know when reading a raw
    edgestore row, and the reason a `v.edges()` slice does not return vertex-centric index replicas. Add
    the band table (0-1 system, 2-3 user, 4-5 invisible = index replicas, 6-7 unreachable), one sentence
    on `getBounds`, and one sentence making explicit that columns are **grouped** by type rather than
    ordered by type id. Docs-only, no code risk.
17. **`EdgeSerializer.java:481-483` — the dead empty-interval branch.** `interval.isEmpty()` means
    *unsatisfiable*, and breaking out of the loop leaves `sliceStart == null`, which falls into `:533` and
    yields a slice over the **whole type band** — the maximum over-read, where the correct answer is the
    empty set. It is unreachable today because `BasicVertexCentricQueryBuilder:607-610` returns
    `BaseVertexCentricQuery.emptyQuery()` for any empty interval before `getQuery` is called, so this is
    not a live bug. Either delete the branch as dead or make the empty case produce an empty slice;
    which one is a maintainer's call, so **open as an issue first**, with the trace from `:481` to `:533`
    to `:548`, rather than as a PR.

18. **Hoist `readRelation` onto `RelationReader`, and make the OLAP label scan header-only.** The
    entry cache (`StaticArrayEntry`'s `transient volatile RelationCache cache`) is applied by
    `EdgeSerializer.readRelation`, which is **not** declared on the `RelationReader` interface — that
    interface has exactly one method, `parseRelation`. `JanusGraphVertexDeserializer.readHadoopVertex`
    holds a `RelationReader` from `setup.getRelationReader()`, so it cannot reach the cache, and it
    iterates the same `Iterable<Entry>` **twice** calling `parseRelation(data, false, typeManager)` both
    times: once in the vertex-label loop and once in the decode loop. So **every edgestore column in a
    Hadoop/Spark scan is fully parsed twice and memoised never**, which is precisely the cost `29c0b18a`
    (#4995) removed from the OLTP path. Two independent wins: add `readRelation` as a `default` method on
    `RelationReader` delegating to `parseRelation` (so implementations without an entry cache are
    unaffected) and switch `janusgraph-hadoop` to it; and change the label-finding loop to
    `parseRelation(data, true, typeManager)`, or to `EdgeSerializer.parseTypeId(data)`, since it reads
    only `relation.typeId` and, for the one matching column, `getOtherVertexId()` — today it fully parses
    the sort key, signature and every inline property of every column to find one vertex-label column.
    **Check before filing:** whether a second `RelationReader` implementation exists that a default
    method would surprise, and whether an open issue already covers OLAP scan cost. Land candidate 19
    first — `decodeProperties` guards on `hasProperties()`, which returns `false` for a header-only cache
    *without* throwing, so a careless reuse of a header-only parse there would silently drop properties.
19. **Make a header-only `RelationCache` throw instead of NPE.** `get(long)` (`:67-69`),
    `numProperties()` (`:85-87`) and `propertyIterator()` (`:97-101`) all dereference `properties`
    unguarded, so on a header-only cache (`properties == null`) each is a bare `NullPointerException`
    with no message. The contract is documented only in the `isFullyParsed()` javadoc (`:71-76`), and the
    entire enforcement is three hand-written conditions — `EdgeSerializer:85`,
    `CacheEdge.getPropertyMap`, `CacheVertexProperty.getPropertyMap`. `RelationCacheTest` asserts
    `isFullyParsed()` and `hasProperties()` on a header-only cache and asserts nothing about the other
    three. Throw `IllegalStateException("relation was parsed header-only")` from each, with a
    `RelationCacheTest` case per method. Smallest of the three new candidates, and it is the guard that
    makes 18 safe to write.
20. **Bound the ASC sort-key read as the DESC read is bounded — REWRITTEN by lesson 12; the trigger is
    a third-party serializer, not an in-tree one.** `EdgeSerializer:169-174` sends DESC through
    `in.subrange(keyLength, true)`, whose `limit` is exactly the sort-key window, so
    `StaticArrayBuffer.require` throws past it; ASC reads **in place** out of `in` itself and nothing
    compares the bytes consumed against `keyLength`. **Lesson 12 settled reachability: no.** Ten of the
    eleven in-tree order-preserving datatypes are fixed-width in byte order, and the eleventh (`String`,
    `3 + 2n`) is self-delimiting by a `00 00` terminator that `writeByteOrder`'s
    `Preconditions.checkArgument(c > 0)` refuses to let into the payload — read length equals write
    length over 20,000 strings. **What survives** is that `StandardRelationTypeMaker.checkSortKey`'s only
    gate is `instanceof OrderPreservingSerializer`, with no width or symmetry check, so a **custom**
    serializer registered through `attributes.custom.attribute[X].serializer-class` can be accepted as a
    sort key and read past its window — silently wrong on ASC, a loud `ArrayIndexOutOfBoundsException` on
    DESC. File it as *"bound the ASC sort-key read so a third-party `OrderPreservingSerializer` cannot
    read past the sort-key window"*: one line,
    `in.subrange(keyLength, def.getSortOrder() == Order.DESC)` (the non-inverting `subrange` returns a
    **view** with no byte copy), plus a test that registers a deliberately over-reading serializer and
    asserts the exception rather than garbage. Low priority, trivially reviewable, no behaviour change for
    any in-tree type. **If reviewers prefer, drop it — but record that decision rather than leaving it
    open a fourth lesson.** Note the inverting `subrange` *does* allocate (`new byte[length]` +
    arraycopy + `(byte)~b` loop + two wrappers), so every DESC index replica pays three allocations and a
    copy on its first property read.
21. **Fix `docs/advanced-topics/serializer.md`'s *Custom Object Serialization* section. Doc-only,
    smallest candidate in the list, and the best first contribution this track has found.** The page
    states that *"JanusGraph will test both serialization and deserialization of a custom class before
    persisting data to disk"* and that the attribute class must implement `AttributeSerializer`, have a
    no-argument constructor and implement `equals(Object)`. Reading the whole registration path —
    `RegisteredAttributeClassesConverter.convert` → `RegisteredAttributeClass.registerWith` →
    `StandardSerializer.registerClass` → `registerClassInternal` — **there is no such round trip**: the
    registration validates the number, the non-nullity of the type and serializer and the absence of a
    collision, then puts two map entries. No instance is constructed, nothing is serialized, `equals` is
    never called. The no-arg-constructor requirement is real but applies to the **serializer**
    (`Class.forName(serializerName).newInstance()`), not the attribute class, which the paragraph
    conflates. Four edits: delete the round-trip claim, split the two requirements, drop the stale
    `equals` requirement, and add the two facts the page omits — that a **built-in** type's serializer
    cannot be replaced through configuration (with the hard-coded `HashMap` exception, from #306) and that
    both options are `GLOBAL_OFFLINE`. Zero code risk, and the review forces a maintainer to confirm the
    finding.
22. **Document the sort-key datatype restrictions where they are discoverable, and clear three dead
    spots in the same area.** `StandardRelationTypeMaker.sortKey`'s javadoc gives as its example *"the
    edge label friend has the sort key (since), which is a property key with a timestamp data type"* —
    and `InstantSerializer` is **not** an `OrderPreservingSerializer`, so the canonical example in the API
    documentation is precisely the case `checkSortKey` rejects. Replace it with `Long`/`Date`, list the
    eleven permitted types, and note that a `String` sort key forbids `U+0000` (enforced only at
    serialization time by `StringSerializer.writeByteOrder`; `verifyAttribute` checks only `MAX_LENGTH`).
    Bundle, or split if reviewers prefer one concern per PR: remove `StringSerializer`'s
    `TEXT_COMPRESSION_THRESHOLD`, which is `public static final`, never read by `StringSerializer`, and
    read only by a test to size random strings; remove `normalizeDataType`'s `Instant` branch, which is
    unconditionally dead (`java.time.Instant` is `final`, and the branch returns exactly what the
    fallthrough returns); and collapse `makeDefinition`'s triple `checkSortKey`/`checkSignature` calls to
    one each.
23. **Make `InstantSerializer` and `DurationSerializer` order-preserving.** Largest of the three new
    candidates and the most useful — `Instant` is the type users reach for first for a time-ordered
    vertex-centric index, and today it is rejected outright. The encoding has to be
    `[8-byte biased seconds][4-byte biased nanos]` = 12 bytes in byte order, leaving `read`/`write`
    untouched for on-disk compatibility, because the current nanos field rides `VariableLong`'s
    sign-magnitude zig-zag, which is **not** order-preserving (lesson 10's STOP-bit inversion applies).
    Tests: add both to `SerializerTest`'s `TYPES` (which moves `testSerializedOrder`'s
    `assertEquals(10, sortTypes.size())` to 12 — and that count is itself a fragile assertion worth
    replacing with a set comparison in the same PR) and to
    `testMissingValuesOfTheTypesWhichWriteThemLastComeAfterTheirValues`. **Search for an existing issue
    first**; #948 (UUID) is the precedent for how such a request is framed, and #1248 is the
    user-facing symptom report for the UUID version of the same complaint.

24. **NEW, and tied with 21 for the smallest: the dead disjunct in
    `SystemTypeManager.throwIfSystemName`.** The only validation a user-supplied type name passes
    through is
    `if (SystemTypeManager.isSystemType(name.toLowerCase(Locale.ROOT)) || Token.isSystemName(name))`.
    `isSystemType` is `SYSTEM_TYPES_BY_NAME.containsKey(name) || ADDITIONAL_RESERVED_NAMES.contains(name)`,
    and the keys of `SYSTEM_TYPES_BY_NAME` are mixed-case (`~T$VertexExists`, `~T$SchemaName`, …) while
    the argument has been lowercased — so **the first disjunct can never match from this call site**, the
    only one a repository-wide code search surfaces. Nothing is exploitable: `Token.isSystemName`'s `~`
    prefix check still rejects every system name, and `ADDITIONAL_RESERVED_NAMES` (`key`, `vertex`,
    `edge`, `element`, `property`, `label`) is all lowercase and does fire. But the line reads as though
    two independent checks protect the namespace and only one does. Either drop the `isSystemType` call
    or compare case-insensitively and keep it meaningful; add a test asserting that
    `makePropertyKey("~T$SchemaName")` throws **and** naming which check catches it. Zero behavioural
    risk either way, and the review forces a maintainer to confirm the reading.
25. **Pin the ordinals of every persisted enum in a test, because the ordinal IS the on-disk format.**
    `EnumSerializer.write` is `VariableLong.writePositive(object.ordinal())` and `read` is
    `values[ordinal]` with only a **range** check, so all seventeen enums registered at
    `StandardSerializer` slots 41-57 (plus 69) are persisted by declaration order with no version marker.
    `TypeDefinitionCategory`'s first line is the comment *"Don't change the order because it breaks
    backward compatibility"* — and that comment is the entire mitigation, because
    `TypeDefinitionMap.setValue`'s `assert type.verifyAttribute(value)` is an `assert`, disabled in
    production. Inserting a constant mid-list silently reinterprets stored schema (a `SIGNATURE` reads as
    a `SORT_ORDER`, surfacing much later as a `ClassCastException` in `PropertyKeyVertex.dataType()`);
    deleting one shifts everything above it. The cheap, reviewable version is a new
    `PersistedEnumOrdinalTest` (or a block in `SerializerTest`) holding a golden name → ordinal list for
    all seventeen enums, so that a mid-list insertion fails CI instead of corrupting a graph. The
    expensive version — a version field, or name-based encoding — is an on-disk format change and almost
    certainly out of scope; **propose the test first and say so in the PR body.**

26. **A `MASKABLE` option set once, by the first instance, becomes the cluster default forever — silently.**
    `ReadConfigurationBuilder.getGlobalSubset` (`:172-178`) filters on `ConfigOption.isGlobal()`, which
    **includes `MASKABLE`** (`ConfigOption:182`), so first-open copies every MASKABLE option present in
    that instance's properties file into the `system_properties` `"configuration"` row and then freezes
    it. That copy is never refreshed — `getGlobalSubset` runs only in the not-frozen branch — and never
    discrepancy-checked, because `getManagedSubset` (`:180-186`) filters on `isManaged()`, which
    excludes MASKABLE. But `MixedConfiguration.get:49-58` *does* fall through to it when the option is
    absent locally. Net effect: instance A creates the graph with `cache.db-cache-size=0.5`; instance B,
    months later, omits the line entirely and gets **0.5, not the code default**, with no warning in any
    log and nothing in the configuration reference that says so. Start doc-only plus a test: the
    `@ParameterizedTest` pair in `ReadConfigurationBuilderTest` (`:301`/`:328`) already drives the
    managed/not-managed split, and lesson 14's Hands-On §2 step 3 is the integration-level assertion
    (open a temp BerkeleyJE graph with a MASKABLE option set, reopen without it, assert the value). Then
    ask the maintainers which of the three fixes they want: document it, warn when a frozen MASKABLE
    global value is used with no local value present, or stop persisting MASKABLE in `getGlobalSubset`
    (a behaviour change, and the only one that is genuinely a fix).
27. **`ModifiableConfiguration.set` has an `UPGRADEABLE_FIXED` escape hatch and `remove` does not.**
    `set` (`:37-44`) guards with `hasUpgradeableFixed(option.getName()) || !option.isFixed() ||
    !isFrozen()`; `remove` (`:54-59`) guards with only `!option.isFixed() || !isFrozen()`. So with
    `graph.allow-upgrade=true` you may **set** `graph.storage-version` on a frozen graph but may not
    **remove** it, and the failure is an `IllegalArgumentException` reading "Cannot change configuration
    option" that does not mention the asymmetry. This may well be deliberate — removing a version marker
    is not something an upgrade should do — but nothing says so, and the two guards sit nine lines apart
    looking like a copy-paste divergence. The small, reviewable PR is a one-line comment on `remove`
    stating the intent, plus a test pinning that `remove` of an upgradeable-fixed option throws even
    under `allow-upgrade`. If a maintainer says it is a bug instead, the fix is one token. Either way
    the review settles it.
28. **`KCVSConfiguration.asReadConfiguration()` throws a message-less `IllegalArgumentException` on a
    datatype mismatch.** `:202` is
    `Preconditions.checkArgument(!entries.containsKey(key) || dataType.isAssignableFrom(entries.get(key).getClass()));`
    — no message, no format arguments. This fires at open time, from
    `MixedConfiguration.has`/`get`, whenever a value stored in the `"configuration"` row does not match
    the `ConfigOption`'s current datatype — which is exactly what happens if an option's declared type
    changes across releases, or if a custom backend option collides on a path. The user sees a bare
    `java.lang.IllegalArgumentException` from inside a lambda, with no key, no expected type and no
    actual type. The whole fix is adding a message:
    `"Configuration value for key [%s] is a %s, not the expected %s"`. Zero behavioural risk, improves
    every support thread that ever hits it, and small enough to be a clean first PR. Worth pairing with
    a one-line test that stores a `Long` under a `String`-typed option's path and asserts the message
    names the key.

29. **NEW, and the smallest the track has found: `MIN_VERTEX_CACHE_SIZE`'s comment cites the wrong
    issue.** `StandardJanusGraphTx.java:201-205` reads *"This is a workaround for #893. Cache sizes
    small relative to the level of thread parallelism can lead to JanusGraph generating multiple copies
    of a single vertex in a single transaction."* Read with `issue_read` at the source:
    [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) is a pull request
    opened 2018-01-17 by `davidclement90`, titled *"Remove TTL support for Elasticsearch in
    documentation."*, closed 2018-01-18, milestone `Release v0.2.1`, label `cla: yes`. It has nothing
    to do with vertex caches or thread parallelism; the number is almost certainly a carried-over
    **Titan** issue id from before the fork. Anyone trying to understand why a 100-element floor exists
    follows the comment into unrelated 2018 documentation history. One line, no behaviour change.
    **Do the homework before opening it:** search the upstream tracker (and Titan's archived tracker)
    for the actual duplicate-vertex report, and if it cannot be found, say so in the PR and propose
    describing the mechanism in the comment instead of citing a number — substituting one wrong number
    for another is worse than the status quo. The mechanism is independently verifiable and worth
    stating in the PR body: `CaffeineVertexCache.get`'s tier-2 (`volatileVertices`) → tier-3
    (`retriever.get`) sequence is not atomic, so two threads of one multi-threaded transaction can both
    miss and both construct; only one wins `cache.get(id, k -> v)`, and the loser has already been
    handed to its caller.
30. **NEW: `AddedPropertiesSetValue.isNull()` and `AddedPropertiesListValue.isNull()` are hardcoded
    `false`, which makes a cleanup branch dead code.** `SimpleAddedRelations.removeProperty` ends with
    ```
    if (propertiesValue.isNull()) { propertiesMap.remove(key); }
    ```
    `AddedPropertiesSingleValue.isNull()` returns `internalRelation == null`, so a `SINGLE` key's entry
    is pruned correctly. `AddedPropertiesSetValue.isNull()` and `AddedPropertiesListValue.isNull()` both
    `return false` unconditionally, so for a `SET` or `LIST` key the branch can **never** fire: remove
    every property of that key from a vertex and you are left with an empty `AddedPropertiesSetValue`
    wrapping an empty `HashMap` (or an empty `LinkedHashSet`), keyed by the key's name, for the rest of
    the transaction. `getAllProperties()` then concatenates an empty iterator per dead key on every full
    view of that vertex's properties. Bounded by the number of distinct key names ever written, so this
    is a wart rather than a leak — but it is an inconsistency between three implementations of one
    two-method interface, and the fix is `return internalRelationMap.isEmpty();` /
    `return internalRelationSet.isEmpty();`. **The test is the larger half of the PR:**
    `SimpleAddedRelations` has **no dedicated unit test** anywhere in the tree at the pinned commit, so
    this needs a new test class that adds and removes `SET`- and `LIST`-cardinality properties and
    asserts both `propertiesSize` and the pruning. Worth checking first whether any call site depends
    on the entry surviving — none was found in `StandardVertex` or `StandardJanusGraphTx`, but
    `getView(Set, Predicate)`'s `allProperties` short-circuit should be re-read before claiming that in
    the PR.

### Still open, and who owns it

- **Closed by lesson 14 — all three obligations lesson 13 handed it.** (a) **Construction order:**
  established line by line for `StandardJanusGraph:264-360`; the answer to "what must exist before a
  value can be read out of `system_properties`" is **a `KeyColumnValueStoreManager` and nothing else** —
  the config row is never read through `typeCacheRetrieval`, which is why the open path can create,
  use and close an entire store manager before `getBackend()` runs. `schemaCache` at `:308` is safe
  before `isOpen` at `:320` only because nothing in that window touches it; there is no assertion and
  no test. (b) **FIXED / GLOBAL_OFFLINE / GLOBAL / MASKABLE at open time:** the first three are
  **indistinguishable** — every open-path site uses `isGlobal()`/`isManaged()`/`isLocal()`, and the only
  two sites in the tree that read the constant are `ModifiableConfiguration.set:37-44` and
  `ManagementSystem`'s `configVerifier:190-207`. MASKABLE is the only type whose handling differs at
  open, in `MixedConfiguration.get`'s precedence and in its exclusion from the discrepancy check — and
  that exclusion combined with its inclusion in `getGlobalSubset` is PR candidate 26. (c)
  **`MetricInstrumentedSchemaCache` counts and nothing else**, confirmed by reading all 74 lines: a
  `StandardSchemaCache` with a MISS-incrementing retriever, no HIT counter, no counted eviction, and
  JVM-global counter names. Also corrected lesson 13's note that `attributes.custom.*` is FIXED — it is
  `GLOBAL_OFFLINE`.
- **Closed by lesson 15 — all three obligations lesson 14 handed it.** (a) **The no-backend window is
  harmless by construction:** every value `StandardJanusGraphTx`'s constructor reads off `graph`
  (`:308-317`) is a pre-built object, `txHandle` is the only backend-facing field and is non-final, and
  the constructor does no I/O whatsoever. `StandardTransactionBuilder`'s constructor reads **fifteen
  pre-resolved fields** off the `GraphDatabaseConfiguration` — not off a `Configuration` — so building a
  transaction consults no config row and cannot fail on a missing option. (b) **A user vertex can carry
  a temporary id past creation, but only with `ids.flush=false`, and the default is `true`** — so
  normally the temporary id lives for the three statements between `:686` and `:690`. Lesson 13's
  schema-vertex finding is confirmed: `makeSchemaVertex:1098` assigns unconditionally and asserts
  `longId() > 0`. The containers hold, in the meantime, a `StandardVertex` at lifecycle byte `New` plus
  its `VertexExists` property, which is the relation through which `assignIDs` reaches the vertex at
  commit. (c) **`newTypeCache` resolves name → id and `vertexCache` resolves id → object**, and they meet
  only in `getSchemaVertex:1216-1225`; `newTypeCache` is checked first, written only at
  `makeSchemaVertex:1109`, and emptied at `:2206`.
- **Owed to lesson 16 by lesson 15:** `addedRelations.getAllUnsafe()` is a **live, uncopied view** and
  `commit:2145` hands it straight to `graph.commit` — establish what `prepareCommit` does with it, what
  `ChangedSchemaVertices.of(addedRelations, deletedRelations)` (`StandardJanusGraph:1236`) extracts and
  why schema changes need separate handling, how the `upsert` flag set at `StandardJanusGraphTx:1035`
  changes the delete path, and the exact emission order of deletes versus additions.
- **Owed to lesson 18 by lesson 15:** everything in `StandardJanusGraphTx` is transaction-local —
  `getLock`, `LockTuple`, `ReentrantTransactionLock`, `CombinerLock`, `FakeLock`, a hardcoded 5000 ms
  timeout, and a `uniqueLocks` map that never shrinks. The file reads `ConsistencyModifier.LOCK` at
  exactly one place (`:1030`, to decide whether the blind-overwrite path is allowed). Lesson 18 picks
  up at `ExpectedValueCheckingStore` and `ConsistentKeyLocker`, and owes an answer to how a
  transaction-local lock and a storage lock compose when both are in play on one property write.
- **Owed to lesson 17 by lesson 15:** a property write takes one `getLock(IndexLockTuple(index, match))`
  per unique composite index **record**, built from
  `IndexRecordUtil.indexMatches(vertex, index, key, normalizedValue)` (`:997-1000`). Lesson 17 must say
  what a *record* is there before it explains the graphindex entries, and should note that
  `TypeUtil.hasAnyIndex(key)` is one of the three conditions gating the blind-overwrite path — adding
  any index to a key silently moves every write of that key onto a path that issues a slice query.
- **Owed to lesson 19 by lesson 15:** why `StandardVertex.hasLoadedRelations` is a flat `false` and
  `loadRelations` returns `EntryList.EMPTY_LIST` for a new vertex while `CacheVertex` must answer both
  truthfully; and what `relTypeCache` (`:239`, a `ConcurrentHashMap<>(30)` allocated only when
  `isLazyLoadRelations()`, populated by `getOrLoadRelationTypeById:486-496`) actually caches, given its
  javadoc says "only in `JanusGraphLazyRelation`".
- **Owed to lesson 18 by lesson 14:** `KCVSConfiguration.set(..., checkExpectedValue=true)` calls
  `store.acquireLock` at `:153` on the config row — who calls that overload, and does the config row
  therefore go through `ConsistentKeyLocker` exactly as a `SchemaName` graphindex entry does? Related
  and unanswered: `StandardJanusGraph:341`'s `REGISTRATION_TIME` write goes through
  `ModifiableConfiguration.set`, i.e. the **non**-expected-value overload, so provisionally the instance
  registration is **unlocked** and `ManagementSystem`'s `openInstances.size() < 2` guard for
  GLOBAL_OFFLINE is racing it. Confirm or refute.
- **Owed to lesson 25 by lesson 14:** `ManagementSystem`'s `modifyConfig` is a `ModifiableConfiguration`
  with `Restriction.GLOBAL` over a `TransactionalConfiguration` (`:175-177`), so the whole
  `JanusGraphManagement` config surface is writes to the one `"configuration"` row. `graphShutdownRequired`
  (`:204`) is the GLOBAL_OFFLINE consequence — trace where it is honoured.
- **Closed by lesson 12:** both questions lesson 11 owed it. **No in-tree `readObjectByteOrder` is
  length-ambiguous** — ten of eleven order-preserving datatypes are fixed-width in byte order and
  `String` is self-delimiting by a terminator its own writer refuses to let into the payload — so PR
  candidate 20's silent-corruption framing is unreachable and the candidate was **rewritten**, not
  dropped: the surviving trigger is a custom third-party `OrderPreservingSerializer`, which
  `checkSortKey` accepts on nothing but an `instanceof`. And **`VariableString.writeBackward` does put
  `STRING_ID_MARKER` at the high address**, with the `0x80` stop bit at the low one, so
  `readBackward`'s `getByte(--position)` peek is correct for the string case as it is for the long case.
  Also closed from lesson 10's list: `writeObjectByteOrder` vs `writeObject` vs `writeClassAndObject`
  (three positions selected by `InlineType`, plus a fourth path confined to `Object.class` keys),
  `writesMissingValuesLast` (a question about the flag byte, excluding exactly `ClassSerializer`), and
  **is every `isOrderPreservingDatatype` also fixed-width? — no, `String` is not, but it does not matter,
  because it is self-delimiting**.
- **Closed by lesson 13 — all three obligations lessons 10, 11 and 12 handed it.** (a) **The bands:**
  schema-definition relations do land in bands 0 and 1, because `BaseRelationType.isInvisibleType()` is
  an unconditional `true` and every `BaseKey`/`BaseLabel` id is a system id (suffix 37 or 53); implicit
  keys are user-band as lesson 10 suspected, and `writeRelation:364-365` filters them out of the
  remaining-properties loop regardless. (b) **The recursion:** `getExistingRelationType` bottoms out at
  `SystemTypeManager`'s static 17-entry `HashMap` for system ids, and otherwise at the vertex constructor
  at `StandardJanusGraphTx:641-664`, which picks the Java class **from the id's suffix bits alone**,
  before any byte of the row is read — so the format is not self-describing anywhere; the *ids* are. (c)
  **The ordinals:** every `TypeDefinitionCategory` reaches disk as a one-byte declaration-order ordinal
  through `EnumSerializer`, `getValue` validates only the range, `TypeDefinitionDescriptionSerializer`
  adds **no** versioning of its own, and schema values do use `writeClassAndObject` — `SchemaDefinitionDesc`
  has datatype `Object.class`, so every definition value is self-describing at the *type* level and
  unversioned at the *value* level. New PR candidates 24 and 25.
- **Owed to lesson 17 by lesson 13:** the eight **edge** `TypeDefinitionCategory` values (ordinals 17-24)
  are the whole of what hangs off `SchemaDefinitionEdge`. Establish which carry a real modifier
  (`INDEX_FIELD` with `Parameter[]`, `CONNECTION_EDGE` with a `String` are the two with datatypes) and
  what `CompositeIndexTypeWrapper`/`MixedIndexTypeWrapper` read out of `getRelated`. Cost baseline to
  cite: a property key is 11 edgestore columns plus 2 graphindex entries, and every definition tag is two
  bytes.
- **Owed to lesson 18 by lesson 13:** `BaseKey.SchemaName`'s internal composite index declares
  `ConsistencyModifier.LOCK` and `getBackingIndexName() == Token.INTERNAL_INDEX_NAME`, so "one name, one
  type" rests entirely on `ConsistentKeyLocker`. Establish what `ExpectedValueCheckingStore` does with an
  internal index key, and whether two instances creating the same property key concurrently fail cleanly
  or leave two ids under one name. Also: `CONSISTENCY_LEVEL` and `TTL` live on separate `TYPE_MODIFIER`
  schema vertices reached by a definition edge, which is where the modifier machinery belongs.
- **Owed to lesson 25 by lesson 13:** `JanusGraphSchemaVertex.resetCache()` and
  `StandardJanusGraphTx.expireSchemaElement` are the invalidation half of a schema change, and
  `updateSchemaVertex` (which writes `SchemaUpdateTime`) is the only thing that marks one. Establish how
  `ManagementSystem` and `ManagementLogger` get from a committed schema change to an eviction on every
  other open instance, and whether `SchemaUpdateTime` is read by anything or only written.
- **Owed to lessons 21 and 26 by lesson 12, and it decides whether an inconsistency is a bug:** three
  comparators disagree — the stored bytes (`-0.0 < 0.0` strictly, NaN last), `AttributeUtils.compareValues`
  (same, via `Double.compareTo`), and `AttributeUtils.compare` (returns 0 for `-0.0` vs `0.0`, `null` for
  a one-NaN pair, and compares across numeric types where `compareValues` throws), plus
  `RelationComparator.compareValues` as a third entry point. Establish which one answers a `Cmp` range,
  which one `GraphCentricQueryBuilder` scores an index with, and whether any of the three divergences is
  reachable from a user query. Related: #5120, a mixed index given an order by a TEXT key no backend can
  sort by, is the same shape one layer up. And the string byte order is **UTF-16 code-unit order, not
  code-point order**, which is the UUID bug's twin waiting in `String` — Elasticsearch, OpenSearch and
  Solr sort strings by code point.
- **Owed to lesson 17 by lesson 12:** `IndexRecordUtil` makes the same `hasGenericDataType` →
  `writeClassAndObject` branch that `EdgeSerializer` does, so graphindex field values take the
  **non**-byte-ordered path. Establish whether a composite index key is byte-ordered at all, and if not,
  what gives a composite index its range behaviour. Carry lesson 12's cost table: the sort-key premium is
  zero for fixed-width types and `3 + 2n` vs `1 + n` for strings.
- **Owed to lessons 19-20 by lesson 11:** `RelationCache`'s header-only state is what
  `tx.getConfiguration().isLazyLoadRelations()` and `JanusGraphLazyRelationConstructor` trade on
  (`RelationConstructor`'s entry iterator branches on it). Establish which Gremlin steps settle for a
  header and which force a full parse — **17 of 50 is the static answer; the dynamic one is lesson 20's.**
  And lesson 20's starting point is now sharper: the extended sort key works *because* the backward
  encoding is order-preserving.
- **Closed by lesson 10:** `TYPE_LEN_RESERVE` and `MAX_PADDING_BITWIDTH` (they are `PREFIX_BIT_LEN` and
  the one shift in `stripEntireRelationTypePadding`); the `IDHandler`-vs-`getKeyID` first-bit convention
  (two different address spaces, nothing to reconcile); `BufferUtil.nextBiggerBuffer` (cannot overflow on
  a type prefix) and `padBuffer` (pads row keys, never columns); and where lesson 09's stride of 16 and
  the edge/endpoint co-location actually show up in the column layout — the stride as one byte of prefix
  in every cell, the co-location as the reason a vertex-centric index slice is local.
- **Owed to lesson 17 by lesson 10:** composite indexes go to the **graphindex** store with a different
  key and column scheme. Carry lesson 10's §8 column arithmetic over (2 + N columns per edge for N
  vertex-centric indexes, each a full re-serialization) so the two index kinds can be costed side by side.
- **Owed to lesson 20 by lessons 09 and 10:** `BasicVertexCentricQueryBuilder` properly —
  `constructSliceQueries`' recursion, the candidate scoring at `:683-713`, `getExtendedSortKey`, and the
  `ADJACENT_ID` / `JANUSGRAPHID` position assertions at `EdgeSerializer:466-471` that it has to satisfy.
  Plus, from lesson 09: edges take the partition of an incident vertex (`VertexIDAssigner:206-219`), so
  vertex-centric queries have a locality property that falls out of id assignment, not out of the query
  layer. Worth one paragraph.
- **Owed to lesson 16 by lesson 09:** `assignIDs(Iterable<InternalRelation>)`
  (`VertexIDAssigner:278-333`) is called from `StandardJanusGraph.prepareCommit`, and `ids.flush`
  (default `true`) decides whether ids are assigned at element creation or at commit.
  `VertexIDAssignerTest#testDisableIdsFlush:123` is the entry point. The bulk path's two-pass structure
  and its own 1000-attempt `leftOvers` loop belong there. **Add from lesson 10:** `prepareCommit`'s
  `wholeRowDeletion` short-circuit (`StandardJanusGraph:1112-1138`) skips `writeRelation` entirely for a
  fully-removed vertex when the backend advertises the capability — a serialization cost, not a
  correctness one, and the comment at `:1131-1135` says so.
- **Owed to lesson 18 by lesson 09:** `ConsistentKeyIDAuthority`'s comment at `:90-92` says its slice
  bounds mirror `ConsistentKeyLocker`'s; read the two protocols side by side. Also still owed:
  `ExpectedValueCheckingTransaction.prepareForMutations()`, `ExpectedValueCheckingStore`, `BerkeleyJETx`,
  `BerkeleyJELocks`, `LockKeyColumnValueStoreTest`. **Add from lesson 10:** `acquireEdgeLock` locks on
  the **serialized entry** (`StandardJanusGraph:900-901` passes the whole `Entry`; `:1004` passes only
  `entry.getColumn()`) — which is why the remaining-properties sort at `EdgeSerializer:369-371` is a
  correctness requirement, not tidiness. The asymmetry between those two call sites is worth a look.
- **Owed to lesson 19 by lesson 08:** the partitioned-vertex read cost — one logical vertex, up to
  `2^partitionBits` representative rows, one slice query each. Plus, carried from lesson 04 and sharpened
  by lesson 07: the `getNoCopy` aliasing question belongs in `KeyColumnValueStoreTest` and must mutate
  through a `CacheTransaction`; the `MemoryEntryList` sizing question and the
  `InMemoryColumnValueStore.java:162` NPE candidate also land here. `MultiPageEntryBuffer`,
  `BufferPageUtils` and `BufferPage.merge` remain deferred with no owner; propose lesson 19.
- **Owed to lesson 22:** `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299+`, shared mutable `current`, per-key limit at
  `L381`) plus `SimpleScanJob`.
- **Owed to lesson 25 by lesson 10:** the `SchemaStatus` gate at `StandardJanusGraph:1125`
  (`DISABLED` index types are skipped at write time) and `ManagementSystem:365-390`'s
  `canIndexBeEnabled` / `REGISTER_INDEX` branch are the write-side half of the index lifecycle. Also:
  changing a multiplicity rewrites the column layout and there is **no migration in `writeRelation`** —
  whatever protects a user from that lives in lesson 25's machinery.
- **Owed to lesson 27 by lesson 09:** the ids-store column layout — `[-(end)][timestamp][uid]` with an
  empty value, slice bounds `zeroBuffer(1)` to `oneBuffer(17)` — is the smallest real column layout in
  the codebase and the cleanest thing to compare against `CQLKeyColumnValueStore`. Hands-On §2's byte
  dump is specified for it. Also the count stride of 16, which affects CQL token distribution. **Add from
  lesson 10:** the edgestore column layout is now fully specified, so lesson 27 can compare CQL's
  clustering-key encoding against the six-band prefix directly.
- **Carried from sessions 01-04:** 24 questions posed, none answered by Aryant. The half-open interval now
  has **three** implementations on record (`SinglePageEntryBuffer.getSlice` binary search;
  `BerkeleyJEKeyValueStore.java:258`'s per-record `compareTo`;
  `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange`) and two consumers relying on the upper
  bound being exclusive (`BackendTransaction`'s `EDGESTORE_MAX_KEY`, and `ConsistentKeyIDAuthority`'s
  `UPPER_SLICE = oneBuffer(17)`). **Lesson 10 adds a third consumer:** every `getQuery` slice end is
  `nextBiggerBuffer`-exclusive, and the missing-value marker relies on exclusivity on the ASC upper bound
  and the DESC lower bound. Lesson 27 adds CQL as the fourth implementation. The contract is in
  `KeyColumnValueStore`'s javadoc and **none** of the enforcement is. **Lesson 14 adds a fifth:**
  `KCVSConfiguration.get` builds `KeySliceQuery(rowKey, column, nextBiggerBuffer(column))` for a
  single-column point lookup, which is correct only because the upper bound is exclusive.

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
