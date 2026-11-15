<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 17
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-11 at the Part II/III boundary (lesson 13) — a59ae8d8 is still the tip of
  JanusGraph/janusgraph master, so there is nothing to re-pin to; staying. Not re-raised by lessons 14,
  15, 16 or 17 (all four mid-Part, by design; list_commits was deliberately not run). DUE NEXT RUN: the
  Part III/IV boundary is lesson 18, which must re-run list_commits on master rather than assuming
  either answer.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (PR candidates 1-32; "Still open, and who owns it")
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

**The registers live in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md), new on
2026-11-14.** PR candidates 1–32 and the "Still open, and who owns it" ledger were split out of this
file at lesson 16: this file had reached 116 KB, it is read **in full** at the start of every run, and
those two sections were ~55 KB of append-only material that matters only when choosing a contribution
or settling a debt. Nothing was dropped in the move. A run reads this file always; it reads the
registers file when its lesson is named there as an owner, or when it needs the next free candidate
number. The split is the only structural change of the 2026-11-14 run.

## Session Log

Newest first. **Session-log roll-off:** the full `**Findings.**` paragraph is kept for the three most
recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons 09-12 were
compressed on 2026-11-13, lesson 13 on 2026-11-14, and **lesson 14's paragraph was compressed this
run**; the full paragraphs remain verbatim in git history at the commit each line names, and their
substance is carried forward either in the surviving `Verified by` blocks, in the registers file, or in
the root `README.md`'s per-lesson narrative. Next run, lesson 15's paragraph is the one that goes.

- **17 — 2026-11-15** · [`2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md`](2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md) · **Lesson 16's "complete frame and one unopened box" held without correction; the box is open.** Read `IndexSerializer` (whole, 614 lines; written up for `:224-408` and `:593-612`), `IndexRecordUtil` (whole), `IndexUpdate`, `IndexUpdateContainer`, `IndexMutationType`, `IndexRecords`, `IndexRecordEntry` (all whole), `StandardJanusGraph` again (whole, 1490; cited at `:267-305`, `:518-533`, `:761-771`, `:1014-1025`, `:1080-1097`, `:1158-1206`, `:1295`), `ManagementSystem` (whole, 1815; cited at `:431` and `:831-879`), `BackendTransaction` (whole), `HashingUtil` (whole), `StaticBuffer` (whole), `StandardJanusGraphTx` again (whole, 2278; cited at `:436` only), `IndexSerializerRemoveElementTest`, `WholeRowDeletionOptimizationTest` and `WholeRowDeletionCapturingStoreManager` (all whole), plus four directory listings. One doc page: [Index Performance](https://docs.janusgraph.org/schema/index-management/index-performance/).
  **Findings.** **THE finding: the composite-index row key is deliberately unorderable and the column width *is* the uniqueness level.** `getIndexKey` is `varint(indexId)` ++ per-field `writeObjectNotNull` — the **non**-order-preserving writer lesson 12 dissected — optionally prefixed with 4 bytes of murmur, and `hashKeys` **is not a config option**: `StandardJanusGraph:304-305` passes `storeFeatures.isDistributed() && storeFeatures.isKeyOrdered()`. So the backends that could scan the graphindex store in order are exactly the ones whose keys are hashed, which is the mechanism behind the documented "composite indexes are equality-only" — a limitation the docs assert without explaining and that no planner change can lift. The column is `0x00` alone for `SINGLE`, `0x00` ++ element id for `SET`, and `0x00` ++ element id ++ one varint per field for `LIST`; uniqueness is enforced by lesson 02's one-value-per-`(key, column)` contract with **no index-side check**, and the cardinality itself is **derived** in `ManagementSystem:848-850` from `unique()` and the keys' own cardinalities, never declared. `acquireLock(CompositeIndexType, …)` excludes exactly `LIST`, the one cardinality whose column cannot collide. **`IndexMutationType.UPDATE` exists only for inline (covering) keys, is reachable from one of the three generation sites, and is written as an addition** — `IndexUpdateContainer` is a two-field state machine that keeps it only while no `ADD`/`DELETE` touches the same index key. **Refinement to lesson 16, not a correction:** "phases 1-4 touch storage only to take locks" is false of phase 3 — `indexMatches` goes through `VertexCentricQueryBuilder` for every removal (`onlyLoaded = rel.isRemoved()`) and every partitioned vertex, from a `parallelStream`, with phase-1/2 locks held; `loadLabelsCheckedByIndexes` batches only the label reads. Also: `mutateIndex` calls the **four**-argument `mutateEntries`, so lesson 16's whole-row-deletion optimisation is edgestore-only **by construction**; a deletion's value bytes exist for the lock, not the mutation; `getCompleteDocument` is a second unbatched read inside phase 6; `indexMatches` emits a full Cartesian product with no guard; and three TTL paths disagree, the composite one taking the minimum over **every field key of the record**. **One conjecture, derived and explicitly not run:** `UPDATE` reaches the addition-lock pass with expected value `null` against a cell that by construction exists — predicted failure for a `LOCK`-consistency, non-`LIST`, inline-key composite index on an inline-only property change. Handed to lesson 18 with a named confirming scenario. No new PR candidates logged (the conjecture becomes one only if lesson 18 confirms it). No syllabus revisions. Daily Diff: the **2026-11-14** edition (39 stories) — **RFC 10008, the HTTP QUERY method**, after XRAID's primaries proved unfetchable.

- **16 — 2026-11-14** · [`2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md`](2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md) · **All four things lesson 15 owed this run are settled, and one of them by correcting lesson 15.** Read `StandardJanusGraph` (whole, 1490 lines; written up for `:761-1470` plus the constructor fields it depends on), `StandardJanusGraphTx` again (whole, 2278; cited at `:1020-1048` and `:2115-2230`), `BackendTransaction` (whole), `CacheTransaction` (whole), `Mutation` (whole), `KCVEntryMutation` (whole), `TransactionLogHeader` (whole), `StandardVertexProperty` (whole), `GraphDatabaseConfiguration` again (whole, 2192; cited at `:255-270`, `:388-406`, `:748-800`, `:1912-1925`, `:2088-2110`), `WholeRowDeletionOptimizationTest` (whole), `WholeRowDeletionCapturingStoreManager` (whole), plus the `janusgraph-inmemory` test-package listing. Three `search_code` queries at the pinned ref (`isUpsert`, `getBufferSize`, `wholeRowDeletion` under `janusgraph-inmemory`), one `search_issues` query on this repo, and the [Failure & Recovery](https://docs.janusgraph.org/operations/recovery/) doc page.
  **Findings.** **The emission order is not decided in the commit path.** `prepareCommit` is seven phases over three accumulators — `mutations` (`ListMultimap<vertexId, relation>`), `mutatedProperties` (`ListMultimap<vertex, relation>`, properties only) and `indexUpdates` — and phases 1-4 only *collect* and take locks; phase 5 writes one `mutateEdges` per vertex key with **additions and deletions in the same call**, and the resolution happens in `Mutation.consolidate()` keyed on `KCVEntryMutation.ENTRY2COLUMN_FCT` (the **column**; the value is ignored) under the javadoc'd rule that *"additions apply logically after deletions"* — lesson 02's `KeyColumnValueStore` contract, finally spent. The delete/add **lock asymmetry is the uniqueness constraint**: a delete locks with the full `Entry` (expected value = the serialized relation, `:901`), an add locks with the column alone (expected value `null` = "must not exist", `:1004`), and an add whose endpoint `isNew()` takes **no lock at all** (`:1002`), so bulk-inserting fresh vertices is free of storage locks even under `ConsistencyModifier.LOCK`. `prepareCommit` runs **twice** only when `hasSchemaElements && !hasTxIsolation`, which is what the three filter predicates (`SCHEMA_FILTER`, `NO_SCHEMA_FILTER`, `NO_FILTER`, `:1208-1213`) exist for; `SCHEMA_FILTER` is two conditions ANDed (`BaseRelationType` **and** out-vertex `instanceof JanusGraphSchemaVertex`). **Whole-row deletion skips `writeRelation` entirely** for a fully-removed vertex (`:1136-1138`), is a config AND a feature (`storage.drop-whole-row-on-vertex-removal`, MASKABLE, default **true**, AND `hasOptimizedWholeRowDeletion()`), and costs identity in the change stream — which is the entire reason `cdcCanIdentifyDeletedRelation` (`:955-980`) exists, and it is the clearest statement in the tree of what a tombstone can and cannot carry, read straight off lesson 10's column layout. **Stage 7 swallows every index exception** (collect → log → `SECONDARY_FAILURE` → return success), and the repair path needs `tx.log-tx`, which defaults **false** and is force-disabled under batch loading (`:1232`) — [#833](https://github.com/JanusGraph/janusgraph/issues/833) has been open on exactly this since 2017. **`ChangedSchemaVertices` is cache invalidation, not persistence**: collected at `:1236` before any write because it reads live transaction state, consumed in a `finally` that never escalates, and its broadcast writes the management log **synchronously**, putting max-write-time on the commit's critical path. **Correction to lesson 15:** the `upsert` flag is **not** a commit-path mechanism — `isUpsert` appears in exactly one file and its only read is inside `StandardVertexProperty.remove()`, where it pays back the slice query the blind-overwrite optimisation skipped. Also: the phase-debug trace is gated on `totalMutations >= storage.buffer-size` (default **1024**), so it is silent on ordinary transactions; `txCounter` starts at 0 per JVM so WAL ids collide across restarts and recovery orders by the header timestamp; phase 3's index-update generation runs on `parallelStream()` with **no** threshold. New PR candidates **31** and **32**, both comment/message-level. No syllabus revisions. Structural change: the two registers split to `tracks/janusgraph-registers.md`. Daily Diff: the **2026-11-13** edition (47 stories) — Cloudflare choosing **QuePaxa**, a randomized priority-based leaderless consensus protocol, for its Meerkat control plane.

- **15 — 2026-11-13** · [`2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md`](2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md) · **All three obligations lesson 14 handed this run are discharged.** Read `StandardJanusGraphTx` (whole, 2278 lines; written up for `:193-1135` and `:2137-2278`), `StandardTransactionBuilder` (whole, 759), `StandardJanusGraph` (whole, 1490; cited at `:570-612`, `:750-762`, `:1215-1240`), `GraphDatabaseConfiguration` (whole, 2192; cited at `:115-132`, `:664-700`, `:1060-1070`, `:2055-2070`), `ElementLifeCycle` (whole), `StandardVertex` (whole), `CaffeineVertexCache` (whole), `SimpleAddedRelations` (whole), `ConcurrentAddedRelations` (whole), `AddedRelationsContainer` (whole), `AddedPropertiesSingleValue` / `SetValue` / `ListValue` (all whole), `RemovedVertexTrackingTest` (whole), plus the directory listings for `graphdb/transaction/`, `.../addedrelations/`, `.../vertexcache/`, `graphdb/vertices/` and the in-memory test package. Also read [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) with `issue_read` and the [Transaction Handling](https://docs.janusgraph.org/basics/transactions/) doc page.
  **Findings.** Every data write funnels through `connectRelation:945` and fans out to **three** destinations (each endpoint's own container, the transaction's container, the vertex cache for non-new endpoints only) plus a fourth for properties (`addedPropertyIndex`). The per-transaction and per-vertex relation containers are **the same class with opposite `groupPropertiesByKey`**, which makes `propertiesSize` permanently 0 at transaction level and `propertiesContainer` permanently empty at vertex level, and means `getView(types, filter)` **ignores `types`** at transaction level. `ids.flush` defaults **true**, so a user vertex's temporary id lives for three statements; `makeSchemaVertex:1098` assigns unconditionally, so a schema vertex never has one. `ElementLifeCycle` is a six-state machine (three constants `private`) whose predicates are **range tests on the numeric ordering**, with `New` and `Modified` absorbing and `isModified()` excluding both `New` and `Removed`. `ConcurrentAddedRelations.getAllUnsafe()` is the one method **not** passed through `copyView` — it returns a live view, which is the whole meaning of the name, and `commit:2145` hands it to `graph.commit`. `MIN_VERTEX_CACHE_SIZE = 100` is a probability reduction, not a fix, for a non-atomic tier-2/tier-3 sequence in `CaffeineVertexCache.get` — **and the `#893` it cites is a 2018 Elasticsearch-TTL documentation PR, not a vertex-cache bug** (PR candidate 29). New PR candidates **29 and 30**. Daily Diff: the **2026-11-12** edition (66 stories) — PostgreSQL's 52-CVE security release and CVE-2026-6460, the composite-attribute type-confusion umbrella fix.

- **14 — 2026-11-12** · [`2026/11/2026-11-12-janusgraph-open-path-config-merge/README.md`](2026/11/2026-11-12-janusgraph-open-path-config-merge/README.md) · **Part III opens; all three of lesson 13's obligations discharged and one of its notes corrected.** Read the whole open path — `JanusGraphFactory`, `GraphFactoryUtils`, `SchemaInitializationManager`, `GraphDatabaseConfigurationBuilder`, `ReadConfigurationBuilder`, `ConfigOption`, the five `Configuration` implementations, `KCVSConfiguration`, `MetricInstrumentedSchemaCache`, `ReadConfigurationBuilderTest` — plus slices of `GraphDatabaseConfiguration`, `StandardJanusGraph`, `Backend`, `ManagementSystem` and `StandardSerializer`. **THE finding: the global configuration is one row** in `system_properties`, row key UTF-8 `"configuration"`, columns the dotted option paths, values `writeClassAndObject`; `ConfigOption.ACCEPTED_DATATYPES` is a closed set of 13 classes whose registration ids top out at 65, so **every config value's registration prefix is one byte**. Second: **MASKABLE options leak from the first instance to every later one** (`getGlobalSubset` filters on `isGlobal()`, which includes MASKABLE; never refreshed, never discrepancy-checked) — PR candidate **26**. Third: **two store managers per open**, the first closed before the second exists, so a storage option whose authoritative value is in the row is not applied to the manager that reads the row. Also: six FIXED options, three escapable via `graph.allow-upgrade`, and a successful upgrade writes `allow-upgrade=false` back, disarming itself. New PR candidates **26**, **27**, **28**. The two facts lesson 18 was promised — `KCVSConfiguration.set:153`'s `acquireLock` with an expected value, and the five `ConfigOption` types collapsing to three predicates at open time — are restated in *Notes For The Next Run*. Daily Diff: the **2026-11-11** edition (58 stories). *(Full paragraph in git history at the 2026-11-12 and 2026-11-13 commits; compressed form at the 2026-11-14 commit.)*
- **13 — 2026-11-11** · [`2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md`](2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md) · **Part II complete; re-pin reviewed and declined — `a59ae8d8` is still the tip of `master`.** Read the whole system-type and schema-storage surface: `BaseKey`, `BaseLabel`, `BaseRelationType`, `SystemTypeManager`, `Token`, `JanusGraphSchemaCategory`, `TypeDefinitionCategory`, `TypeDefinitionMap`, `TypeDefinitionDescription`, `StandardRelationTypeMaker`, `JanusGraphSchemaVertex`, `PropertyKeyVertex`, `StandardSchemaCache`, `TypeDefinitionDescriptionSerializer`, `EnumSerializer`, plus slices of `StandardJanusGraphTx` and `IDManager`. **THE finding: there is no catalog** — the schema is a subgraph in the same `edgestore`, six system property keys and two system edge labels hold all of it, and their ids are *computed* (`(raw << 6) | 37` / `| 53`) so there is no bootstrap read. A property key is **11 edgestore columns + 2 graphindex entries**, its definition **exactly 7** of them by set *equality*, and one ordinal byte (5 `DATATYPE` vs 6 `UNIDIRECTIONAL`) is the whole on-disk difference between a property key and an edge label. New PR candidates **24** and **25**. Daily Diff: the **2026-11-10** edition (63 stories). *(Full entry in git history at the 2026-11-11 commit; the facts later lessons need survive in the registers file and in the root README's narrative.)*
- **12 — 2026-11-10** · [`2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md`](2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md) · **stayed on `a59ae8d8`** · read `StandardSerializer`, the `Serializer`/`OrderPreservingSerializer`/`SupportsNullSerializer` interfaces, all seventeen registered attribute serializers, `NumericUtils`, `StringEncoding`, `VariableLong`, `VariableString`, `StandardRelationTypeMaker` and `SerializerTest`. **THE finding: `StandardSerializer` has four write entry points and `EdgeSerializer` chooses between them by *where in the column the value sits*, not by its type — and the whole difference is one flag byte.** Also: eleven order-preserving datatypes, three in-tree comparators that disagree, and PR candidate 20 rewritten rather than dropped. Daily Diff: the **2026-11-09** edition (50 stories) — Turso implementing Tantivy's `Directory` trait against its own B-tree. *(Full entry in git history at the 2026-11-10 commit.)*
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
- [x] 16 — `StandardJanusGraph.commit` and `prepareCommit`: deletes, additions, mutations, and the order they are emitted in
- [x] 17 — Composite indexes on write: `IndexSerializer` and the graphindex entries
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
2026-10-30.** Lesson 17 was handed "a complete frame and one unopened box" and that is exactly what it
found: everything lesson 16 promised about *when* index updates are generated, locked and written held
without correction, and the whole lesson fitted inside `IndexSerializer` + `IndexRecordUtil` +
`ManagementSystem.createCompositeIndex`. **One refinement, not a correction:** lesson 16's "phases 1-4
touch storage only to take locks" is true of phases 1, 2 and 4 and false of phase 3 —
`IndexRecordUtil.indexMatches` goes through `VertexCentricQueryBuilder` for every *removal*
(`onlyLoaded = rel.isRemoved()`) and for every partitioned vertex, i.e. it issues slice queries from a
`parallelStream` with the phase-1/2 locks already held. `loadLabelsCheckedByIndexes`
(`StandardJanusGraph:1019-1021`) batches only the **label** reads the constraint check needs, not those
property reads.

**Lesson 18 is now the best-prepared item in Part III, and it inherits one falsifiable conjecture.**
Lesson 16 gave it the four graph-level `acquireLock` call sites and the expected-value asymmetry;
lesson 14 gave it `KCVSConfiguration.set`'s own `acquireLock` with an expected value; lesson 17 adds
the two index-lock passes and **the conjecture it must settle first**: `IndexMutationType.UPDATE` is
not a deletion, so it is handled by pass 2 (`StandardJanusGraph:1090-1096`) with expected value
`null` — "this cell must not exist" — against a cell that by construction already exists, because an
`UPDATE` is generated only when an **inline** key changed and the field-key values (hence row key and
column) did not. Predicted failure conditions: composite index, `ConsistencyModifier.LOCK`,
cardinality != `LIST`, at least one inline key, transaction changing only an inline property of an
existing element. **Not verified by running** — the entry says so and puts the confirming scenario in
its Hands-On B (BerkeleyJE, breakpoint on `prepareCommitAcquireIndexLocks`, second pass). Lesson 18
either logs a PR candidate or names the line that makes it a non-issue. Lesson 18's second inherited
item: confirm from `ExpectedValueCheckingStore`/`ConsistentKeyLocker` the **idempotence** claim lesson
17 derived from `BackendTransaction`'s javadoc — that a second lock request for an already-locked
`(key, column)` pair in the same transaction is dropped, which is what makes "deletions first"
load-bearing and lets one transaction hand a unique value from one element to another.

**The re-pin decision is DUE NEXT RUN.** Lesson 13 reviewed it at the Part II/III boundary and found
`a59ae8d8` still the tip of `master`. Lessons 14, 15, 16 and 17 were all mid-Part, so it was **not
re-raised and `list_commits` was not run** in any of them — by design. Lesson 18 closes Part III, so
**lesson 18 must re-run `list_commits` on `JanusGraph/janusgraph` `master`** and decide, rather than
assuming either answer. If it re-pins, note that Part IV's byte offsets stop being comparable with
lessons 02-17.

**The roll-off rule applied again this run.** `TRACK.md` keeps the notes for the three most recent
lessons; older ones roll off into git history. The three most recent are now 15, 16 and 17, so
**lesson 14's `Verified by` block is removed** (compressed form in git history at the 2026-11-14 and
2026-11-15 commits, full form at the 2026-11-12 and 2026-11-13 commits) and lessons 15, 16 and 17 keep
their blocks in full. Next run, lesson 15's block is the one that goes — but **do not drop lesson 14's
two facts that lesson 18 was explicitly promised**, both already restated above: `KCVSConfiguration.set`
calling `store.acquireLock(rowKey, column, expectedValueBuffer, txh)` at `:153`, and the five
`ConfigOption` types collapsing to three predicates at open time. The **PR-candidate and open-question
registers** remain in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md); the roll-off
rule does **not** apply there.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Lesson 18 (`ExpectedValueCheckingStore`, `ConsistentKeyLocker`) owns the locking layer and nothing
  above it.** It must **not** re-derive: lesson 16's four graph-level `acquireLock` call sites or the
  delete/add asymmetry; lesson 17's two index-lock passes, the `acquireLock(CompositeIndexType, …)`
  three-conjunct predicate, or the `SINGLE`/`SET`/`LIST` column widths. What it owns is what happens
  *inside* `acquireLock`: the `ExpectedValueCheckingStore` wrapper, where the expected value is
  remembered and when it is checked, `ConsistentKeyLocker`'s lock rows, the lock-expiry and
  lock-wait options, and how all of that behaves on a store whose `StoreFeatures` offer no native
  locking. Entry point suggested by lesson 14: `KCVSConfiguration.set:153`, the simplest
  compare-and-set in the tree.
- **Lesson 19 should check whether the index store cache is invalidated by these writes.**
  `KCVSCacheInvalidationService` is constructed with both `backend.getEdgeStoreCache()` and
  `backend.getIndexStoreCache()` (`StandardJanusGraph:298-299`), and `mutateIndex` goes through
  `indexStore.mutateEntries`, i.e. the `KCVSCache` wrapper lesson 07 mapped. Whether a composite-index
  row is evicted or written through is lesson 19's question, not lesson 17's.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64 (a large share of each is permalink URLs, which are not prose). The
  old 5-9 KB budget was a `contributor-reading` constraint and does not apply. **Mechanical notes,
  carried and extended this run:** a single `create_or_update_file` call carrying ~87 KB succeeds;
  `get_file_contents` on anything over roughly 55 KB returns an over-limit error that *saves the JSON to
  a local file*, which is the cheap way to read a large source file (`python3`/`sed`/`nl` on the saved
  JSON) **and the only way to get trustworthy line numbers** — a file returned inline has no numbering,
  so lesson 17 cited `IndexRecordUtil` by member name with whole-file permalinks and reserved
  `#Lstart-Lend` fragments for the three files it had numbered locally. That split is honest and should
  be repeated rather than guessed around. `README.md` and `TRACK.md` are both over the limit now, so
  both are read to a local file and edited with a script rather than re-emitted from context.
- **Method note, carried; lesson 17 is the fifth entry where simulation was not needed.** Lessons 10-12
  re-implemented the pinned encoding functions and ran them; lessons 13-17 computed from the read
  sources instead and said so. Lesson 17's derived claims are each labelled where they appear — the
  `UPDATE` lock conjecture, the deletions-first/idempotence argument, and the Cartesian-product cost —
  and each is paired with an exercise that runs the real code (Hands-On A dumps actual graphindex bytes
  for all three cardinalities; B is written to fail if the conjecture holds). **Keep the rule that
  produced them:** compute or re-implement rather than assert, say which, and always pair it with an
  exercise that runs the real code.
- **Daily Diff procedure — the 2026-11-05 one-way cross-check fired for the fourth consecutive run and
  was right again.** At ~03:35 UTC on 2026-11-15 the front page **and** `/archive/` both served
  **2026-11-13 / 47 stories** (archive total **138 editions**) — the edition lesson 16 had digested —
  while `https://tdd.cat/2026-11-14/` served a complete **39-story** edition first try, and
  `https://tdd.cat/2026-11-15/` returned a client error. **The procedure stands as amended on
  2026-11-13:** (a) fetch the front page and `/archive/` and take the maximum; (b) **also probe the
  dated URL for today and yesterday directly** (`https://tdd.cat/YYYY-MM-DD/`), because a published
  edition can be missing from both indexes — proved on 11-13 and again on 11-14; (c) before digesting,
  check this session log for the newest edition already covered — 13 covered 11-10, 14 covered 11-11,
  15 covered 11-12, 16 covered 11-13, 17 covered 11-14, so every run so far has found exactly one
  uncovered edition; (d) the uncovered-item fallback is still the last resort, and the 2026-11-06
  edition still has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). Edition sizes remain
  volatile — 22 on 11-08, 50 on 11-09, 63 on 11-10, 58 on 11-11, 66 on 11-12, 47 on 11-13, 39 on 11-14
  — so a small edition is not evidence of a partial publish. At 39 stories the page fit one WebFetch.
  **New this run:** when the obvious deep-dive candidate's primary sources are unreachable, say so and
  pick another. XRAID (LWN, 2026-11-13) was the most on-topic item in the edition, but both primaries —
  the lore/LWN mailing-list archive URL and Hellwig's `raid-sucks.pdf` slides — are **robots-disallowed**
  to WebFetch, and the LWN report states outright that it does not describe the declustered layout,
  parity distribution or rebuild behaviour. The digest covers XRAID as a notable item with that gap
  named explicitly and deep-dives **RFC 10008** instead, which was fetchable in full. Do not fill an
  unverifiable mechanism with plausible generalities to protect a chosen item.

### Verified by lesson 17, for lessons 18, 19, 25, 26, 27 and 28 to cite rather than re-derive

- **`IndexSerializer` has seven `getIndexUpdates`; three do work** (`:240-265` relation, `:267-319`
  vertex, plus `reindexElement:390-408` for lesson 25). The four thin ones only pick between
  `FULL_INDEX_APPLIES_TO_FILTER` and `INDEX_APPLIES_TO_NO_CONSTRAINTS_FILTER` (`:224-238`). The
  preconditions on both real overloads (`:241`, `:278`) are Java `assert`s, **not** the live
  `Preconditions` lesson 16 found in `prepareCommit`.
- **The two commit-path overloads differ in four load-bearing ways:** index list (`getKeyIndexes()` vs
  `getKeyIndexesReferences()`, which carries `isInlined()`); whether `UPDATE` is possible at all
  (`getUpdateType(relation, false)` hard-coded at `:243` vs `indexRef.isInlined()` at `:281`);
  deduplication (`HashSet<IndexUpdate>` on `IndexUpdate.equals` over `(index, type, key, entry)` vs a
  `HashMap<key, IndexUpdateContainer>` keyed on `update.getKey()` **alone**, `:273`, `:293-297`); and
  the guards `updatedProperties.isEmpty()` (`:272`) and `vertex.isRemoved() && indexRef.isInlined()`
  (`:283`). **`IndexMutationType.UPDATE` is reachable from exactly one of the three generation sites
  and only for an inline key.**
- **The composite row key is `varint(index.longId())` ++ per-field `writeObjectNotNull` (or
  `writeClassAndObject` for a generic data type), optionally prefixed by 4 bytes of murmur.** Two facts
  follow and lessons 25-27 need both. (1) The values use the **non**-order-preserving writer lesson 12
  dissected, so graphindex row keys have no value order. (2) `hashKeys` **is not a config option**:
  `StandardJanusGraph:304-305` passes `storeFeatures.isDistributed() && storeFeatures.isKeyOrdered()`,
  and `hashLength` is a `private final` `HashLength.SHORT` = **4 bytes** with no setter
  (`IndexSerializer:133-134`, `HashingUtil`). So the backends that could scan the index store in order
  are exactly the ones whose keys are hashed. **That is the mechanism behind the documented
  "composite indexes are equality-only"**, and it cannot be lifted in the planner.
- **The column width IS the uniqueness level** (`IndexRecordUtil.getIndexEntry`): `SINGLE` → the single
  byte `0x00` (`FIRST_INDEX_COLUMN_BYTE`); `SET` → `0x00` ++ element id; `LIST` → `0x00` ++ element id
  ++ one `varint(record[i].relationId)` per field. The value is the element identity (`writeVertexId`,
  or `relationId ++ outVertexId ++ typeId [++ inVertexId]` for a relation) followed by inline
  properties, split by the same single `valuePosition` integer lesson 10 found in `writeRelation`.
  Uniqueness is enforced by lesson 02's one-value-per-`(key, column)` contract, not by any index-side
  check.
- **Cardinality is derived in `ManagementSystem.createCompositeIndex:848-850`, never declared:**
  `unique()` → `SINGLE`; else all-`SINGLE` property keys → `SET`; else → `LIST`. Also there:
  `unique` is rejected for non-`VERTEX` indexes (`:838`), and `STATUS` is `ENABLED` immediately when
  `oneNewKey || constraint.isNew()`, otherwise `INSTALLED` + `REGISTER_INDEX` (`:852-860`, `:877`).
  **Lesson 25 starts at this method.**
- **`acquireLock(CompositeIndexType, boolean)` (`StandardJanusGraph:768-771`) excludes exactly
  `Cardinality.LIST`** — the one cardinality whose column already contains the property ids and
  therefore cannot collide. `SET` **does** take storage locks under `ConsistencyModifier.LOCK`. Three
  conjuncts: config flag (false under `enableBatchLoading()`), modifier `== LOCK`, cardinality
  `!= LIST`.
- **`prepareCommitAcquireIndexLocks:1080-1097` is two full passes over the same list, deletions first**,
  mixed-index updates skipped in both. Deletion: `acquireIndexLock(key, (Entry) entry)` → expected value
  = the entry's value bytes. Addition (and therefore `UPDATE`): `acquireIndexLock(key,
  entry.getColumn())` → expected value `null`. For a `SINGLE` index that addition lock is literally
  "nobody currently owns this value", which is the **entire** uniqueness mechanism for `unique()`.
- **`prepareCommitIndexUpdatesAndCheckIfAnyMixedIndexUsed:1163-1206`**: one `mutateIndex` per update
  with a `Collections.singletonList` and the `NO_ADDITIONS`/`NO_DELETIONS` sentinel on the other side;
  an `UPDATE` takes the **addition** path (the branch tests `isDeletion()` only); a deletion passes the
  whole `Entry`, whose value bytes exist for the **lock**, not the mutation (lesson 16:
  `KCVEntryMutation` compares both sides by column). **`BackendTransaction.mutateIndex` calls the
  four-argument `indexStore.mutateEntries` — there is no `wholeRowDeletion` parameter for the index
  store at all**, so lesson 16's `storage.drop-whole-row-on-vertex-removal` optimisation is
  edgestore-only by construction.
- **The mixed branch sets `has2iMods = true` unconditionally**, and `has2iMods` is read in exactly one
  place: `hasSecondaryPersistence = logTxIdentifier != null || commitSummary.has2iModifications`
  (`:1295`) — whether the transaction needs the WAL. **Lesson 28.** `itx.needsCompleteDocument` /
  `registerCompleteDocument` (`:1198-1201`) call `getCompleteDocument` (`IndexSerializer:332-357`),
  which reads every value of every non-`DISABLED` field key through `documentProperties:364-370` — one
  slice with `query.fast-property`, otherwise one per key — and the in-tree comment states why it must
  happen here: *"Read here, while the transaction can still read: the index commit runs after the
  storage commit."* **Lesson 26** needs the rest.
- **cdc-only mixed indexes:** `isCdcOnlyMixedIndex:1158-1161`; the two constructor filters
  (`:267-288`) differ by one clause — the deletion filter keeps a cdc-only index when
  `index.getElement().isRelation()`, so relation-document deletions stay synchronous. The
  constructor's `log.warn` is the only signal, fires once at open, and says itself that the WAL
  records these transactions as successful, so recovery will not repair them. **Lessons 26 and 28.**
- **`SchemaStatus` on the index-write path, index side:** `indexAppliesToWithoutConstraints` rejects a
  `DISABLED` **composite index**, and `IndexSerializer:257` / `:304` skip a `DISABLED` **field** of a
  mixed index. `INSTALLED` and `REGISTERED` still receive writes — which, with lesson 16's `:1125`
  relation-type check, is what lets a reindex job converge. **Lesson 25.**
- **TTL reaches index entries three ways** (`IndexSerializer:244`, `:291-292`, `:306-308`), and
  `IndexRecordUtil.getIndexTTL(vertex, keys...)` takes the **minimum** over the vertex's TTL and every
  named key's, treating 0 as unset. For the composite path the keys are **every field key of the
  record**, not the property that triggered the write. `IndexUpdate.setTTL` itself asserts
  `ttl > 0 && mutationType != DELETE`.
- **Cost facts worth not re-deriving.** `indexMatches` is a recursion over field-key positions that
  emits **∏ kᵢ** records for a vertex with kᵢ values on field key i (1 for all-`SINGLE` keys);
  `IndexRecords.add` overrides `ArrayList.add` to `Arrays.copyOf` the shared buffer, without which
  every record would alias it. `onlyLoaded` **is** `rel.isRemoved()` (`IndexSerializer:288`), so a
  deletion's record is computed from pre-transaction values and an addition's from post-change values,
  each with the changed property substituted via `replaceKey`/`replaceValue`. Inline keys are a
  covering index whose bytes are a full `EdgeSerializer.writeRelation` cell nested inside the index
  cell, framed as `int length`, `int valuePosition` per property.

### Verified by lesson 16, for lessons 17, 18, 19, 25, 27 and 28 to cite rather than re-derive

- **`StandardJanusGraphTx.commit:2137-2164` is the only caller**, it is `synchronized` on the
  transaction, and it branches on `hasModifications()` (`:2223-2225` = `!addedRelations.isEmpty() ||
  !deletedRelations.isEmpty()`). A read-only transaction goes straight to `txHandle.commit()` and
  `graph.commit` is never entered. On failure it rolls back the **backend** handle and rethrows wrapped,
  and `releaseTransaction()` runs in the `finally` regardless — there is no retry-this-transaction path.
- **`prepareCommit:814-874` is seven phases over three accumulators.** `mutations`
  (`ListMultimap<Object vertexId, InternalRelation>`, written by phases 1-2, read by phase 5),
  `mutatedProperties` (`ListMultimap<InternalVertex, InternalRelation>`, **properties only**, read by
  phase 3), `indexUpdates` (`List<IndexUpdate>`, written by phases 1-3, read by phases 4 and 6).
  **Phases 1-4 touch storage only to take locks.** `ModificationSummary` (`:803-812`) carries
  `hasModifications = !mutations.isEmpty()` and `has2iModifications`.
- **The phase-debug trace is double-gated**, `:827`: `logForPrepareCommit.isDebugEnabled() &&
  totalMutations >= backend.getBufferSize()`. The logger is a child logger,
  `org.janusgraph.graphdb.database.StandardJanusGraph.prepareCommit` (`:162-163`), and
  `getBufferSize()` is `storage.buffer-size`, MASKABLE, default **1024**
  (`GraphDatabaseConfiguration:780-782`). So the trace is silent on every ordinary transaction, and
  `totalMutations` counts **relations**, not storage operations.
- **Deletes (phase 1, `:879-921`) and additions (phase 2, `:983-1010`) differ in exactly three places.**
  Precondition (`isRemoved()` vs `isNew()`, both live `Preconditions`, not asserts); lock guard
  (additions add `!vertex.isNew() &&`, `:1002`, so a relation on a brand-new endpoint takes **no** lock
  and skips the `writeRelation` behind it); lock argument (`acquireEdgeLock(key, entry)` at `:901`
  forwards column **and value**, `acquireEdgeLock(key, entry.getColumn())` at `:1004` forwards the column
  with expected value `null`). **That asymmetry is the uniqueness constraint**: a delete asserts "this
  cell exists with this content", an add asserts "this cell does not exist". It is also why lesson 10's
  remaining-properties sort in `EdgeSerializer` is a correctness requirement — the delete-side expected
  value is a byte comparison.
- **Both loops iterate `getLen()`, not `getArity()`**, and guard the two `put`s with
  `pos == 0 || !isLoop()`, so a self-loop files one mutation entry. The `acquireLock` block sits
  **outside** that guard, so a self-loop under `ConsistencyModifier.LOCK` requests the same lock twice —
  idempotent per `BackendTransaction`'s javadoc.
- **`acquireLock(relation, pos, acquireLocksConfig)` (`:761-766`) is the second of exactly two reads of
  `ConsistencyModifier.LOCK` in the tree** (the first is `StandardJanusGraphTx:1030`). Three conjuncts:
  the config flag (false under `enableBatchLoading()`), modifier `== LOCK`, and the multiplicity
  constrains *this position* or the type is `SIMPLE` and `pos == 0`. So a `MULTI` edge never takes a
  storage lock. The composite-index sibling (`:768-771`) is the same shape with
  `getCardinality() != Cardinality.LIST`.
- **Phase 5 (`:1102-1156`) is one `mutateEdges` per distinct vertex key**, and the two entry lists are
  presized asymmetrically — `additions` at `edges.size()`, `deletions` at `max(10, edges.size()/10)`,
  both at **0** under whole-row deletion, with a comment naming super-node removal. `canonicalId`
  collapses a partitioned vertex before asking `tx.isVertexFullyRemoved`, which is lesson 15's
  `removedVertexCanonicalIds` being consumed.
- **`wholeRowDeletionEnabled` (`:295-296`) is a config AND a feature**:
  `storage.drop-whole-row-on-vertex-removal` (MASKABLE, default **true**,
  `GraphDatabaseConfiguration:755-761`) AND `backend.getStoreFeatures().hasOptimizedWholeRowDeletion()`.
  When true for a fully-removed vertex, `writeRelation` is **not called at all** for its deletions
  (`:1136-1138`) — a CPU/allocation saving, explicitly not a correctness one.
- **The inner loop is lesson 10's "2 + N columns per edge" in code.** `baseType.getRelationIndexes()`
  yields the base type plus one invisible relation type per vertex-centric index, each a full independent
  `writeRelation`. **`SchemaStatus.DISABLED` is checked at `:1125` and nowhere else on the write path,
  and it checks only `== DISABLED`** — `INSTALLED` and `REGISTERED` index types **do** receive writes,
  which is what lets a reindex job finish. **Lesson 25 owns the rest of that lifecycle.** `getTTL`
  (`:781-800`) is the **minimum** of the type's TTL and every endpoint label's, applied as
  `EntryMetaData.TTL` on additions only, and non-zero only for an `UnmodifiableVertex` id.
- **THE finding: the emission order is a store contract, not a code path.** Phase 5 hands additions and
  deletions to the same `mutateEdges`; `BackendTransaction` forwards to `KCVSCache.mutateEntries`;
  `CacheTransaction.mutate` builds **one** `KCVEntryMutation` per (store, key) and `Mutation.merge`s
  repeats. `flushInternal` then calls `consolidate()` on every mutation, with the comment *"to ensure
  that no addition accidentally gets swallowed by a delete"*, and `Mutation.consolidate`'s javadoc states
  the rule: a deletion identical to an addition is redundant *"since we consider additions to apply
  logically after deletions"*. `KCVEntryMutation` supplies `ENTRY2COLUMN_FCT = Entry::getColumn` for
  **both** sides, so "identical" means **same column, value ignored** — which is lesson 10's "the column
  is the uniqueness constraint", re-derived one layer down. `wholeRowDeletion` merges as a sticky OR, and
  `Mutation.hasWholeRowDeletion`'s javadoc promises the backend applies it **before** any additions in
  the same mutation, which is what makes the sticky merge safe. **Lesson 27 must check whether CQL
  honours both promises** — `graph.assign-timestamp` (LOCAL, default **true**) is the real mechanism, per
  its own description: "one timestamp to all insertions and another slightly earlier timestamp to all
  deletions in the same batch".
- **Mutations reach storage before `commit()` only under batch loading.** `CacheTransaction.mutate`'s
  flush is guarded `batchLoading && numMutations >= persistChunkSize`, which is what every `[FAILURE]`
  comment means by "nothing has been persisted unless batch-loading". `numMutationsParallelThreshold`
  (`storage.num-mutations-parallel-threshold`, MASKABLE, default **100**,
  `GraphDatabaseConfiguration:789-794`) switches consolidation to a `parallelStream`.
- **`commit:1215-1409` numbers its stages twice**, `1,2,3,3.1,3.2` then `1a,1,2,3` inside the
  `hasModifications` branch, then `4,5` in the `finally` — PR candidate 32. Linearised: empty-check
  `:1217`; commit time + `transactionId = txCounter.incrementAndGet()` `:1220-1222`;
  `idAssigner.assignIDs(addedRelations)` `:1226` **unless** `hasAssignIDsImmediately()` (**this is the
  caller lesson 09's note placed in `prepareCommit` — it is in `commit`**); flags and
  `ChangedSchemaVertices.of` `:1229-1236`; WAL PRECOMMIT `:1244`; schema sub-transaction `:1254-1287`;
  the real `prepareCommit` `:1292`; WAL PRIMARY `:1302`; `commitStorage()` `:1308`; `commitIndexes()`
  `:1322`; user log `:1337`; WAL SECONDARY `:1353`; `finally` schema expiry `:1377-1406`.
- **`commitStorage()` is `storeTx.commit()` only.** The full `BackendTransaction.commit()` (storage
  **then** indexes) is used solely on the no-modifications path at `:1366`. So index writes always
  happen strictly after a successful storage write, and **stage 7 swallows every index exception** —
  collected into a `Map<String,Throwable>`, logged, status `SECONDARY_FAILURE`, return success. Repair
  needs `tx.log-tx`, GLOBAL, default **false** (`GraphDatabaseConfiguration:258-260`), and
  `logTransaction` is additionally `&& !hasEnabledBatchLoading()` (`:1232`).
  [#833](https://github.com/JanusGraph/janusgraph/issues/833) is open on exactly this.
- **`prepareCommit` runs twice only for schema, and only without transaction isolation.** The three
  predicates are at `:1208-1213`; `SCHEMA_FILTER` is **two** conditions ANDed (`BaseRelationType` **and**
  out-vertex `instanceof JanusGraphSchemaVertex`), `:1292` passes `hasTxIsolation ? NO_FILTER :
  NO_SCHEMA_FILTER`. `assert commitSummary.hasModifications && !commitSummary.has2iModifications`
  (`:1267`) encodes that schema writes never touch a mixed index — lesson 13's `SchemaName` UNIQUE index
  is `Token.INTERNAL_INDEX_NAME`, a *composite* index. A failed schema commit **keeps its locks** until
  they expire, in the local mediator and in the lock store (`:1278-1280`) — **lesson 18.**
  `Preconditions.checkArgument(!hasSchemaElements || (!hasEnabledBatchLoading() && acquireLocks))`
  (`:1251-1252`) makes schema creation inside a batch-loading transaction a hard failure, which is the
  mechanical reason for `schema.default=none` in production.
- **`ChangedSchemaVertices` (`:1443-1470`) is cache invalidation, not persistence.** Filters on
  reference identity against `BaseLabel.SchemaDefinitionEdge` (lesson 13's id **2357**); keeps `all`
  (local graph-cache expiry) and `kept` (not-removed → other open transactions **and** the cross-instance
  broadcast). Collected at `:1236` **before** any write, because it reads live transaction state that
  `releaseTransaction` will swap away. The `finally`'s ordering is commented as deliberate: graph-level
  entries first, so a transaction opened later cannot hold the stale definition. **Nothing escalates** —
  `expireSchemaElements:1412-1421` and `tellInstancesToExpire:1428-1437` both log at warn — and the
  management log writes **synchronously**, so a struggling backend holds the commit for up to its
  max-write-time. It also runs after a *failed* commit, because a non-isolated backend may already have
  committed the schema half. **Lesson 25 owns the acknowledged-vs-unacknowledged eviction pair.**
- **Phase 3's index-update generation is the one genuinely parallel step of a commit** (`:1022-1024`,
  `mutatedProperties.keySet().parallelStream()`), with **no** threshold, and `getIndexUpdates(vertex, …)`
  can read. `loadLabelsCheckedByIndexes:1035-1080` is a prefetch, not a correctness step: it batches the
  label reads for existing, non-removed vertices whose changed keys reference a `hasSchemaTypeConstraint`
  index (`checksLabel:1063-1075`) into multi-queries of up to `query.batch.max-keys-per-read` (MASKABLE,
  default **1000**, `GraphDatabaseConfiguration:393-401`), returns early below 2 vertices, and only runs
  when `useMultiQuery() && hasMultiQuery()`.
- **The WAL has four record shapes and two positional enums.** Header is
  `[8-byte timestamp][VariableLong transactionId][writeObjectNotNull(LogTxStatus)][unsigned byte
  metaSize][(unsigned byte LogTxMeta ordinal)(value)]…`, row key
  `HashingUtil.hashPrefixKey(SHORT, getLongBuffer(transactionId))`. `serializeModifications` writes added
  then deleted, each length-prefixed, each relation as `[vertexId(pos 0)][writeRelation(rel, 0, tx)]` —
  **only position 0**, the mirror being derivable. Capacity hint `256 + (added + deleted) * 40`.
  `txCounter` starts at **0** per JVM (`:321`), so ids collide across restarts and recovery orders by the
  header timestamp. **`LogTxMeta`'s ordinal is written as a raw byte and read back as
  `values()[ordinal]` with no range check** — a stronger version of PR candidate 25's concern, and
  **lesson 28's**.
- **`isUpsert` has exactly one read in the tree**, inside `StandardVertexProperty.remove()`: after
  `tx().removeRelation(this)` and the lifecycle update, it runs
  `element().query().types(type.name()).properties().forEach(propertyRemover)` — the slice query the
  blind-overwrite path skipped. The ordering matters: the property is already `Removed` and already out
  of `addedRelations` when the query runs. **So lesson 15's description of `upsert` as a promise to the
  commit path was wrong about the layer**, and by the time `prepareCommit` sees anything, an
  upsert-then-remove is an ordinary `deletedRelations` entry for the old DB-resident property.

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
  iterators are not fail-fast. **Lesson 16 confirmed it is iterated two or three times with no copy.**
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
  — read at exactly one place in this file, `:1030` — and `ExpectedValueCheckingStore`.**
- **`enableBatchLoading()` removes storage work, not just a flag check.** It calls
  `consistencyChecks(false)`, setting **both** `verifyUniqueness` and `acquireLocks` false
  (`StandardTransactionBuilder:169-173`, `:222-227`), so up to three uniqueness queries per edge
  disappear *and* the storage lock is never requested.
- **The blind-overwrite path, and the `upsert` flag** (`:1024-1042`). For `single` or `set`, three
  conditions must **all** hold to skip reading the old value: (1) `!hasVerifyUniqueness()` **or** the
  key's `ConsistencyModifier != LOCK`; (2) `!TypeUtil.hasAnyIndex(key)`; (3) requested cardinality
  equals the key's declared one. Then only in-memory duplicates are removed
  (`getDuplicatedAddedRelation`, not a query) and `prop.setUpsert(true)`. Otherwise
  `vertex.properties(key.name())` runs — a real slice query for a loaded vertex. **Adding any index to
  a key silently moves every write of that key onto the expensive path, with no log line and no
  metric. Lesson 17 owns what a "record" is.** **Lesson 16 corrected where the flag is consumed:**
  `StandardVertexProperty.remove()`, not the commit path.
- **Auto-schema creation happens inside a data write.** `checkPropertyConstraintForVertexOrCreate...`
  (`:870-877`), `...ForEdgeOrCreate...` (`:879-886`) and
  `checkConnectionConstraintOrCreate...` (`:888-903`) return immediately on
  `hasDisabledSchemaConstraints()` and otherwise call `config.getAutoSchemaMaker()` mid-write. Two
  concurrent transactions auto-creating the same key both succeed locally; the `SchemaName` UNIQUE
  graphindex entry (lesson 13) rejects one at the storage layer at commit. **Lesson 18's territory,
  and why `schema.default=none` is the production recommendation** — and lesson 16 adds the mechanical
  reason: `commit:1251-1252` makes schema creation in a batch-loading transaction a hard failure.
- **`releaseTransaction:2200-2211` sets `isOpen = false` first, then swaps eight fields to empty
  singletons.** `txHandle`, `config`, `graph`, `edgeSerializer`, `temporaryIds` and `relTypeCache` are
  **not** reset, which is why `rollbackIfOpen()` still works. `getVertices` defends against the swap
  racing a reader with `result.removeIf(v -> v == null || v.isRemoved())`. **Lesson 16's consequence:**
  anything the commit path needs off live transaction state must be collected before the swap, which
  is exactly why `ChangedSchemaVertices.of` is called at `:1236`.
- **`recordRemovedVertex:804-818` keys by CANONICAL id** (partition representatives collapsed) and
  returns early for new vertices; `isVertexFullyRemoved:824-826` is what commit queries to choose a
  whole-row deletion — consumed at `StandardJanusGraph:1112`.
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
  transaction affinity with one exception: `v instanceof SystemRelationType`.
  **`verifyAttribute:735-759`** is where lesson 12's registry is consulted at write time, once:
  `Object.class` keys check only `validDataType`; otherwise `attributeHandler.convert(datatype,
  attribute)` and a null result is a `SchemaViolationException`.
- **The `#893` in `MIN_VERTEX_CACHE_SIZE`'s comment (`:201-206`) points at the wrong thing.** Verified
  with `issue_read`: [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) is
  a PR opened 2018-01-17 by `davidclement90`, "Remove TTL support for Elasticsearch in
  documentation.", closed 2018-01-18, milestone `Release v0.2.1` — almost certainly a carried-over
  **Titan** id. **PR candidate 29**, and the real report must be found before substituting a number.
  The hazard is real: `CaffeineVertexCache.get`'s tier-2/tier-3 sequence is not atomic, so two threads
  of one multi-threaded transaction can both construct a vertex for the same id; the floor of **100**
  lowers the probability and does not serialise the sequence.

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
