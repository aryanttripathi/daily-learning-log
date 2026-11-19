<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 21
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-16 at the Part III/IV boundary (lesson 18); NOT re-run since, as lessons
  19, 20 and 21 are all mid-Part. Each got a free cross-check instead and all four agree: lesson 19 measured
  Backend at 775 lines and GraphDatabaseConfiguration at 2191; lesson 20 measured EdgeSerializer at 591,
  matching lessons 10 and 11; lesson 21 measured GraphDatabaseConfiguration at 2191 again (exact match with
  18 and 19) and found QueryUtil.processIntersectingRetrievals at :463, exactly where lesson 20 cited it
  (lesson 20 reported QueryUtil's total as 521 lines against lesson 21's 520 — the cited member line agrees
  to the line, so the difference is in the reported total, not a moved pin). The pin has demonstrably not
  moved. NEXT REVIEW: DUE NOW — lesson 22 is the Part IV/V boundary and owes it; re-run list_commits on
  JanusGraph/janusgraph master and either re-pin and record the new commit here, or record the decline.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (PR candidates 1-51, next free 52; "Still open, and who owns it")
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
**Reviewed again 2026-11-16 (lesson 18, the last item of Part III): `list_commits` on `master` returns
the same three newest commits as at lesson 13 — `a59ae8d8` (2026-10-28), `9a30ff14` (2026-10-28) and
`5e3e7bc7` *JanusGraph release 1.2.0* (2026-10-24). There is nothing to re-pin to, so the pin stays and
Part IV's byte offsets stay comparable with lessons 02-18. Lessons 19, 20 and 21 were all mid-Part and
correctly did not re-run `list_commits`; each cross-checked a line count against an earlier lesson's
independent count instead, and all three matched exactly. **The review is now DUE: lesson 22 is the
Part IV/V boundary and owes it.**

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

**The registers live in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md), new on
2026-11-14, and now carrying candidates 1–51 (next free number **52**; 46–51 were added by lesson 21,
which also repaired the file: lesson 20's 43–45 were recorded in this file but never written to the
register, and were recovered this run from lesson 20's own entry).**
PR candidates and the "Still open, and who owns it" ledger were split out of this
file at lesson 16: this file had reached 116 KB, it is read **in full** at the start of every run, and
those two sections were ~55 KB of append-only material that matters only when choosing a contribution
or settling a debt. Nothing was dropped in the move. A run reads this file always; it reads the
registers file when its lesson is named there as an owner, or when it needs the next free candidate
number. The split is the only structural change of the 2026-11-14 run.

## Session Log

Newest first. **Session-log roll-off:** the full `**Findings.**` paragraph is kept for the three most
recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons 09-12 were
compressed on 2026-11-13, lesson 13 on 2026-11-14, lesson 14 on 2026-11-15, lesson 15 on 2026-11-16,
lesson 16 on 2026-11-17, and **lesson 17's paragraph was compressed this run**; the full paragraphs
remain verbatim in git history at the commit each line names, and their substance is carried forward
either in the surviving `Verified by` blocks, in the registers file, or in the root `README.md`'s
per-lesson narrative. Next run, lesson 18's paragraph is the one that goes.

- **21 — 2026-11-19** · [`2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md`](2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md) · **stayed on `a59ae8d8`. All three debts lesson 20 handed over are discharged, and the answer to the headline question is "same kind of function, different shape": the graph-centric planner is also statistics-free, but it solves a weighted set cover over the query's clauses, may choose several indexes, and may choose none — which is the full scan.** Read the whole graph-query and index-selection surface at the pinned commit: `GraphCentricQueryBuilder` (whole, 612 lines, numbered locally; cited at `:177-249`, `:236-238`, `:271-274`, `:282-283`, `:302-330`, `:332-436`, `:448-465`, `:471-495`, `:503-524`, `:532-551`, `:561-597`, `:603-611`), `GraphCentricQuery`, `JointIndexQuery`, `Query`, `IndexSelectionStrategy`, `AbstractIndexSelectionStrategy`, `BruteForceIndexSelectionStrategy`, `ApproximateIndexSelectionStrategy`, `ThresholdBasedIndexSelectionStrategy`, `IndexCandidate`, `IndexCandidateGroup`, `IndexSelectionUtil`, `SubqueryIterator` (all whole), plus `QueryUtil` again (whole, **520** lines by this run's count — lesson 20 reported 521, but `processIntersectingRetrievals` sits at **`:463` in both counts**, so the one-line difference is in the reported total, not a moved pin), `StandardJanusGraphTx` (whole, 2277; cited at `:373-376`, `:1589-1592`, `:1596-1631`, `:1640-1698`, `:1713-1755`, `:1807-1836`, `:1838-1857`, `:1868-1891`, `:1894-1942`, `:1956-1990`), `IndexSerializer` (whole, 613; `:414-446`, `:454-462`, `:464-485`), `GraphDatabaseConfiguration` (whole, **2191** — identical to lessons 18 and 19's independent counts), `StandardJanusGraph` (whole, 1489) and `QueryTest` (whole, 1105 — the dedicated test class the vertex-centric compiler does not have). `search_code` located `JanusGraphStep` and `SubqueryIteratorTest`; `issue_read` fetched issue #4931 and its closing PR #4941. Daily Diff: the 2026-11-18 edition (61 stories), deep dive on Postgres `CREATE TEMP TABLE` catalog churn and the instance-wide shared-invalidation ring.
  **Findings.** **THE finding: the graph-centric decision is a weighted set cover with a lexicographic objective, and it lives behind a swappable SPI rather than in the builder.** `GraphCentricQueryBuilder.constructQueryWithoutProfile` (105 lines) calls `indexSelector.selectIndices(...)`, which mutates a caller-owned `Set<Condition> coveredClauses` as an out-parameter and returns `SelectedIndexQuery(JointIndexQuery, isSorted)`. `IndexCandidateGroup.compareTo` ranks groups by **more clauses covered, then fewer indexes, then higher total score** — so a mixed index scoring `5` beats a composite scoring `1004` when it covers one more clause, and the `+1000`-per-clause composite bias only ever breaks ties. Scoring is four constants (`EQUAL_CONDITION_SCORE = 4`, `OTHER_CONDITION_SCORE = 1`, `CARDINALITY_SINGE_SCORE = CARDINALITY_OTHER_SCORE = 1000`) plus two on the approximate strategy (`ORDER_MATCH = 1`, `ALREADY_MATCHED_ADJUSTOR = -1.1`); no statistics anywhere, and `ThresholdBasedIndexSelectionStrategy` carries the in-tree `TODO` asking for selectivity histograms. Default is `threshold-based` with `query.index-select-threshold = 10`, counted against **raw** candidates, so the default admits `2^10 = 1024` `IndexCandidateGroup` allocations per compiled query inside the `OPTIMIZATION` span. A composite index is all-or-nothing (`constructIndexCover` needs an equality at every field position, and an `IN` multiplies out to one `Object[]` tuple per combination → one `KeySliceQuery` each inside one `MultiKeySliceQuery`), and it is excluded outright when the top-level condition is an `Or`. Execution is **stream-one/intersect-the-rest**: `StandardJanusGraphTx.elementProcessorImpl.execute` streams subquery 0 through `SubqueryIterator` under the joint limit and materialises subqueries 1..n−1 **eagerly and unbounded** via `QueryUtil.processIntersectingRetrievals(retrievals, Query.NO_LIMIT)` — which, because `NO_LIMIT == Integer.MAX_VALUE`, makes that function's adaptive retry loop, its `min(16, 2^(n-1))` multiplier and its `subLimit^1.5` growth unreachable from its only caller (candidate 51). The full scan is literally an empty `JointIndexQuery`; `query.force-index` turns it into a `JanusGraphException`. `isSorted` survives three independent gates (`IndexSelectionUtil.indexCoversOrder`, the same check inside `IndexSerializer.getQuery` when building `OrderEntry`s, and the builder's own `takesTheOrder` re-derivation from the serialized query, which demands position 0, equal length and matching field and direction at every position). **Lesson 20's three debts are closed:** the second scoring function is above; `processIntersectingRetrievals` is an *intersect*, not a merge, because graph-centric sub-queries are conjuncts of one `And`; and `JanusGraphStep` proves the strategy layer **cannot** fold a `has()` into a compiled query — it calls `requireOrderValues()` and then `constructQueryWithoutProfile(...)` on a fresh builder, and the only post-compilation rewrite in the tree is `JointIndexQuery.narrowFirst`, used by `withoutResultsAfter`. Six PR candidates logged, **46-51**; next free **52**.

- **20 — 2026-11-18** · [`2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md`](2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md) · **stayed on `a59ae8d8`. The count question lesson 19 handed over is answered: the number of `SliceQuery` shapes a traversal makes per vertex is a product, not a constant. Lesson 08's partitioned-read debt is fully discharged; lesson 19's `MultiSliceQueriesGroupingUtil` hand-off is traced to its single entry condition.** Read the whole vertex-query package at the pinned commit: `BasicVertexCentricQueryBuilder` (whole, 973 lines, numbered locally; cited at `:205-212`, `:325-334`, `:336-342`, `:344-367`, `:376-409`, `:498-517`, `:537-544`, `:546-731`, `:733-774`, `:787-801`, `:816-847`, `:849-929`, `:960-969`), `BaseVertexCentricQueryBuilder`, `VertexCentricQueryBuilder`, `MultiVertexCentricQueryBuilder`, `SimpleVertexQueryProcessor`, `BaseVertexCentricQuery`, `VertexCentricQuery`, `BasicVertexCentricQueryUtil`, `BackendQueryHolder` (all whole), plus `EdgeSerializer` again (whole, 591, numbered; cited at `:430-434`, `:436-552`, `:554-577`, `:579-588` — the line count matches lessons 10 and 11, a free confirmation the pin has not moved), `QueryUtil` (whole, 521, numbered; cited at `:57-71`, `:89-95`), `IDHandler` (whole), `InternalRelationType` (whole), `RelationTypeVertex` (whole). Tests read: `JanusGraphBaseTest` (whole, 764, numbered; `evaluateQuery` at `:589-659`), `JanusGraphTest` (whole, 12,258, numbered; cited at `:2120-2262`, `:4722-4761`, `:4818-4834`, `:5086-5106`, `:5265-5293`), `InMemoryGraphTest` (whole, for its `@Disabled` list). Three directory listings. Four `search_code` queries, of which `VertexCentricQueryTest` returned **0 hits** — there is no dedicated unit test for this compiler at all; its entire coverage lives inside `JanusGraphTest`. One doc page: [Index Performance](https://docs.janusgraph.org/schema/index-management/index-performance/).
  **Findings.** **THE finding: the vertex-centric planner is one `double` with no statistics, and its arithmetic has a crossover nobody states.** A 1-point equality on a leading sort-key position scores +5.0, a *k*-point `Contain.IN` scores +5.0/k, any range (including the `RangeInterval` that `has(key)` compiles to) scores a flat +1 and then **breaks the loop**, and +3 is added when the candidate's declared sort order and key prefix satisfy the `orderBy` — which, with no `orderBy`, **every** candidate gets, making the ordering bonus a constant offset rather than a tiebreaker. So `5.0/k` crosses the flat `+1` at **k = 5**: a six-value `IN` list is scored *below* an open range on the same key, although it compiles to six tight fitted slices rather than one wide one (**PR candidate 44**, derived, paired with Hands-On B which makes a real planner choose). Second: **`score > bestScore` is strict and `RelationTypeVertex.getRelationIndexes()` yields `Iterables.concat(singletonList(this), …)` — the base relation type is always candidate #0 — so every tie resolves to the plain label slice**, which is the mechanism behind the docs' prose warning that a constraint must land on an index's *first* sort key. Third, and this is the answer lesson 19 asked for: **sub-queries per vertex = Σ over named types of ( dirs × ∏ |points| over the leading run of point-interval sort-key positions )**, with dirs ∈ {1,2} and an `isUnbounded`-BOTH collapse forcing 1; `JanusGraphTest` pins 1, 2 and 3 at `:4829-4834` and 2 at `:5267-5268`. Crossing one is the switch that changes machinery — `bq.getQueries().size() == 1` picks `executeMultiQuery`, anything more picks `executeMultiSliceMultiQuery`, i.e. the grouping tree the db-cache rewrites in place. Fourth: **direction is the low bit of the varint *inside* the relation type**, not a level above it, which is why an untyped `direction(OUT)` query is unfitted (`:5290-5293` asserts the pair three lines apart), why `sliceLimit *= 2` is arithmetic rather than a heuristic, and why a typed BOTH query is two slices. Fifth: **`has(key)` is not "no constraint"** — `intersectConstraints` returns `new RangeInterval<>()` for it, so it occupies a slot in `intervalConstraints` and makes the collapsed BOTH form **unfitted** (`:2247-2248`). Two more candidates: **43** (`getAllRelationsQuery()` allocates a `SliceQuery` per call where the sibling `ALL_PROPERTIES_SLICE` in the same package is a `static final`, with an in-tree comment saying why) and **45** (`BackendQueryHolder.hashCode()` is `Objects.hash(backendQuery, isFitted, isFitted)` — `isFitted` hashed twice, `isSorted` never, while `equals` compares both). Candidates 43-45 logged; next free number **46**.

- **19 — 2026-11-17** · [`2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md`](2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md) · **Part IV opens. Lesson 18's inherited question is answered in full — invalidation *marks* a row key, it does not evict and it does not write through — and the release turns out to be a `CountDownLatch(5)`, not a clock. Three debts from lessons 15 and 16 discharged; lesson 08's partitioned-read debt half-discharged and explicitly handed to lesson 20.** Read the whole vertex and cache layer at the pinned commit: `AbstractVertex`, `StandardVertex`, `CacheVertex` (all whole), `VertexCache`, `CaffeineVertexCache`, `EmptyVertexCache` (all whole), `KCVSCache`, `ExpirationKCVSCache`, `NoKCVSCache`, `CacheTransaction` (all whole), `StandardTransactionBuilder` (whole), plus `StandardJanusGraphTx` again (whole, 2277 lines, numbered locally; cited at `:206`, `:230-268`, `:345-365`, `:486-496`, `:526-607`, `:609-665`, `:1428-1467`, `:1562-1573`, `:2091-2103`, `:2200-2210`, `:2252-2276`), `Backend` again (whole, 775, numbered; cited at `:162-165`, `:257`, `:344-381`, `:768-774`) and `GraphDatabaseConfiguration` again (whole, 2191, numbered; cited at `:610-698`, `:2055-2066`) — both line counts identical to lesson 18's independent counts, a free confirmation that the pin has not moved. Tests read whole: `CacheVertexRefreshTest`, `ExpirationCacheTest`. Four directory listings. Five `search_code` queries, of which `hasValidateKeysOnly` (**3 hits, repo-wide**) is the basis of the dead-branch finding. One doc page: [`docs/operations/cache.md`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/docs/operations/cache.md). **Findings.** (1) **Invalidation granularity is the row key and that is the only implemented path:** `hasValidateKeysOnly()` returns `true` and is overridden nowhere, so the `else` branch of `CacheTransaction.flushInternal` — which builds a `List<CachableStaticBuffer>` from the mutation's entries — is unreachable, and `ExpirationKCVSCache.invalidate`'s precondition is a tautology (**PR candidate 39**). (2) **An invalidated row is uncacheable, not merely uncached:** while the key sits in `expiredKeys`, every `getSlice` takes the `isExpired` branch, reads the store and does **not** repopulate, on all three read paths, enforced by the `key -> null` sentinel from `fillResultAndReturnMisses`; release needs `PENALTY_THRESHOLD = 5` latch countdowns plus `getAge >= db-cache-clean-wait`, and `ExpirationCacheTest.testGracePeriod`'s `minCleanupTriggerCalls = 5` is that constant verbatim. (3) **`db-cache-time = 0`, the setting the docs recommend for a single instance, makes the invalidation *mark* eternal too** — `getExpirationTime()` uses the same `cacheTimeMS`, so `isExpired`'s self-healing branch can never fire and release falls entirely to a 1-in-1000 probabilistic cleanup. (4) **Cleanup is an `O(cache size)` key-set scan** under `synchronized`, with no key → `KeySliceQuery` index (**PR candidate 40**). (5) **Invalidation runs after `persist`**, so a `persist` that throws mid-chunk skips the whole invalidation loop and `clear()`, leaving pre-mutation rows cached for up to `db-cache-time` (**PR candidate 41**, derived from reading, marked *do not file until observed*). (6) **"Pinned in the cache" is implemented on the way out:** `CaffeineVertexCache` lets Caffeine evict by size and rescues modified/removed vertices into an unbounded `volatileVertices` from the removal listener, so `cache.tx-cache-size` bounds clean vertices only. (7) The public two-argument `CacheVertex.addToQueryCache` **skips the refresh-counter guard** and its one caller is the graph-index `vertexIDConversionFct` path (**PR candidate 42**). (8) `refresh()` has exactly one production caller chain — `ManagementSystem`/`ManagementLogger` → `tx.expireSchemaElement` → `sv.refresh()` — so a user-data `CacheVertex` is never refreshed by JanusGraph itself. Candidates 39-42 logged; next free number **43**.

- **18 — 2026-11-16** · [`2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md`](2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md) · **Part III complete. Both debts lesson 17 handed this run are settled — one of them by confirming its conjecture and correcting the backend it named. Re-pin reviewed at the Part III/IV boundary and declined: `a59ae8d8` is still the tip of `master`.** Read the whole locking layer at the pinned commit: `ExpectedValueCheckingStore`, `ExpectedValueCheckingStoreManager`, `ExpectedValueCheckingTransaction`, `ConsistentKeyLocker` (whole, 638 lines, numbered locally), `AbstractLocker` (whole), `LocalLockMediator` (whole), `LockerState` (whole), `ConsistentKeyLockerSerializer` (whole), `ConsistentKeyLockStatus` (whole), `StandardLockCleanerRunnable` (whole), `StandardLockCleanerService` (whole), `PermanentLockingException`, `TemporaryLockingException`, plus `Backend` (whole, 775, numbered; cited at `:157`, `:183-202`, `:271-276`, `:292-299`, `:309-325`, `:345-346`, `:385`, `:537-541`), `BackendTransaction` (whole), `StandardJanusGraph` again (whole, 1489; cited at `:761-771`, `:899-901`, `:1002-1005`, `:1077-1097`, `:1230`), `GraphDatabaseConfiguration` again (whole, 2191; cited at `:148`, `:150-155`, `:796-804`, `:937-996`), `KCVSConfiguration` (whole), `IndexUpdate` (whole), `BufferUtil` (whole), `BaseKeyColumnValueAdapter` (whole), `InMemoryStoreManager` (whole), `BerkeleyJEStoreManager` (whole), `LockKeyColumnValueStoreTest` (whole), `InMemoryLockKeyColumnValueStoreTest` (whole), and both `locking/` directory listings. Five `search_code` queries and one doc page: [Eventual Consistency](https://docs.janusgraph.org/advanced-topics/eventual-consistency/).
  **Findings.** Paragraph compressed 2026-11-19 under the roll-off rule; the full version is in git history at the 2026-11-16 through 2026-11-18 commits. The one-sentence substance: **uniqueness survives an eventually consistent store because of one boolean** — `ExpectedValueCheckingStoreManager`'s constructor rebuilds the backing manager's features with `.locking(true)`, so every `hasLocking()` branch above it sees `true` on a store with no native locking. Two facts this block owed forward are carried in *Notes For The Next Run* rather than dropped: **lesson 27** still owes the CQL/HBase feature-builder classification (`InMemoryStoreManager` is wrapped and runs the full `ConsistentKeyLocker` protocol; `BerkeleyJEStoreManager` sets `.locking(true)` and is native), and **lesson 25** still starts from `ManagementSystem.createCompositeIndex:848-850`.

- **17 — 2026-11-15** · [`2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md`](2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md) · **Lesson 16's "complete frame and one unopened box" held without correction; the box is open.** Paragraph compressed 2026-11-18 under the roll-off rule; the full version is in git history at the 2026-11-15 through 2026-11-17 commits, and its substance survives in the surviving `Verified by` blocks, in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md) and in the root `README.md`'s per-lesson narrative.

- **16 — 2026-11-14** · [`2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md`](2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md) · **All four things lesson 15 owed were settled, one of them by correcting lesson 15; this is also the run that split the registers out of `TRACK.md`.** Paragraph compressed 2026-11-17 under the roll-off rule; the full version is in git history at the 2026-11-14 commit, and its substance survives in the root `README.md`'s per-lesson narrative and in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md).
  **Findings.** **The emission order is not decided in the commit path.** `prepareCommit` is seven phases over three accumulators — `mutations` (`ListMultimap<vertexId, relation>`), `mutatedProperties` (`ListMultimap<vertex, relation>`, properties only) and `indexUpdates` — and phases 1-4 only *collect* and take locks; phase 5 writes one `mutateEdges` per vertex key with **additions and deletions in the same call**, and the resolution happens in `Mutation.consolidate()` keyed on `KCVEntryMutation.ENTRY2COLUMN_FCT` (the **column**; the value is ignored) under the javadoc'd rule that *"additions apply logically after deletions"* — lesson 02's `KeyColumnValueStore` contract, finally spent. The delete/add **lock asymmetry is the uniqueness constraint**: a delete locks with the full `Entry` (expected value = the serialized relation, `:901`), an add locks with the column alone (expected value `null` = "must not exist", `:1004`), and an add whose endpoint `isNew()` takes **no lock at all** (`:1002`), so bulk-inserting fresh vertices is free of storage locks even under `ConsistencyModifier.LOCK`. `prepareCommit` runs **twice** only when `hasSchemaElements && !hasTxIsolation`, which is what the three filter predicates (`SCHEMA_FILTER`, `NO_SCHEMA_FILTER`, `NO_FILTER`, `:1208-1213`) exist for; `SCHEMA_FILTER` is two conditions ANDed (`BaseRelationType` **and** out-vertex `instanceof JanusGraphSchemaVertex`). **Whole-row deletion skips `writeRelation` entirely** for a fully-removed vertex (`:1136-1138`), is a config AND a feature (`storage.drop-whole-row-on-vertex-removal`, MASKABLE, default **true**, AND `hasOptimizedWholeRowDeletion()`), and costs identity in the change stream — which is the entire reason `cdcCanIdentifyDeletedRelation` (`:955-980`) exists, and it is the clearest statement in the tree of what a tombstone can and cannot carry, read straight off lesson 10's column layout. **Stage 7 swallows every index exception** (collect → log → `SECONDARY_FAILURE` → return success), and the repair path needs `tx.log-tx`, which defaults **false** and is force-disabled under batch loading (`:1232`) — [#833](https://github.com/JanusGraph/janusgraph/issues/833) has been open on exactly this since 2017. **`ChangedSchemaVertices` is cache invalidation, not persistence**: collected at `:1236` before any write because it reads live transaction state, consumed in a `finally` that never escalates, and its broadcast writes the management log **synchronously**, putting max-write-time on the commit's critical path. **Correction to lesson 15:** the `upsert` flag is **not** a commit-path mechanism — `isUpsert` appears in exactly one file and its only read is inside `StandardVertexProperty.remove()`, where it pays back the slice query the blind-overwrite optimisation skipped. Also: the phase-debug trace is gated on `totalMutations >= storage.buffer-size` (default **1024**), so it is silent on ordinary transactions; `txCounter` starts at 0 per JVM so WAL ids collide across restarts and recovery orders by the header timestamp; phase 3's index-update generation runs on `parallelStream()` with **no** threshold. New PR candidates **31** and **32**, both comment/message-level. No syllabus revisions. Structural change: the two registers split to `tracks/janusgraph-registers.md`. Daily Diff: the **2026-11-13** edition (47 stories) — Cloudflare choosing **QuePaxa**, a randomized priority-based leaderless consensus protocol, for its Meerkat control plane.

- **15 — 2026-11-13** · [`2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md`](2026/11/2026-11-13-janusgraph-tx-write-path-containers/README.md) · **All three obligations lesson 14 handed this run are discharged.** Read `StandardJanusGraphTx` (whole, 2278 lines; written up for `:193-1135` and `:2137-2278`), `StandardTransactionBuilder` (whole, 759), `StandardJanusGraph` (whole, 1490; cited at `:570-612`, `:750-762`, `:1215-1240`), `GraphDatabaseConfiguration` (whole, 2192; cited at `:115-132`, `:664-700`, `:1060-1070`, `:2055-2070`), `ElementLifeCycle` (whole), `StandardVertex` (whole), `CaffeineVertexCache` (whole), `SimpleAddedRelations` (whole), `ConcurrentAddedRelations` (whole), `AddedRelationsContainer` (whole), `AddedPropertiesSingleValue` / `SetValue` / `ListValue` (all whole), `RemovedVertexTrackingTest` (whole), plus the directory listings for `graphdb/transaction/`, `.../addedrelations/`, `.../vertexcache/`, `graphdb/vertices/` and the in-memory test package. Also read [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) with `issue_read` and the [Transaction Handling](https://docs.janusgraph.org/basics/transactions/) doc page.
  **Findings.** Every data write funnels through `connectRelation:945` to **three** destinations (each
  endpoint's container, the transaction's container, the vertex cache for non-new endpoints) plus
  `addedPropertyIndex` for properties. The per-transaction and per-vertex relation containers are **the
  same class with opposite `groupPropertiesByKey`**. `ids.flush` defaults **true**. `ElementLifeCycle`
  is a six-state machine whose predicates are range tests on the numeric ordering.
  `ConcurrentAddedRelations.getAllUnsafe()` returns a live view and `commit:2145` hands it to
  `graph.commit`. `MIN_VERTEX_CACHE_SIZE = 100` is a probability reduction, not a fix, for a non-atomic
  tier-2/tier-3 sequence in `CaffeineVertexCache.get` — **and the `#893` it cites is a 2018
  Elasticsearch-TTL documentation PR, not a vertex-cache bug** (PR candidate 29). New PR candidates
  **29 and 30**. Daily Diff: the **2026-11-12** edition (66 stories) — PostgreSQL's 52-CVE security
  release and CVE-2026-6460. *(Full paragraph in git history at the 2026-11-13 through 2026-11-15
  commits; compressed form from the 2026-11-16 commit.)*

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
- [x] 18 — Locking: `ExpectedValueCheckingStore` and `ConsistentKeyLocker`, and how uniqueness survives an eventually consistent store

### Part IV — The read path (`graphdb/vertices`, `graphdb/query`)

- [x] 19 — `StandardVertex`/`CacheVertex`, lazy adjacency loading, and the transaction-level and database-level caches
- [x] 20 — `BasicVertexCentricQueryBuilder`: `v.edges(...)` → `SliceQuery`, and vertex-centric indexes
- [x] 21 — `GraphCentricQueryBuilder`: index selection between composite, mixed, and full scan
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
2026-10-30.** Lesson 21 closed the query-compilation half of Part IV by answering, in full, the question
lesson 20 posed: **the graph-centric planner is the same *kind* of function as the vertex-centric one —
a scalar over schema and query text, with no statistics of any sort — but a different *shape*.** Where
lesson 20's picks one relation index per `(type, direction)` by `score > bestScore`, this one solves a
weighted set cover: `IndexSelectionStrategy.selectIndices` returns a `JointIndexQuery` that may hold
several indexes, and `IndexCandidateGroup.compareTo` ranks groups **lexicographically — more clauses
covered, then fewer indexes, then higher score.**

**THE fact to carry forward, because it decides what the read actually costs: only the first subquery
streams.** `StandardJanusGraphTx.elementProcessorImpl.execute:1894-1942` hands subquery 0 to
`SubqueryIterator` with the joint query's limit as its `readLimit`, and materialises subqueries
1..n−1 **eagerly and without any limit** through
`QueryUtil.processIntersectingRetrievals(retrievals, Query.NO_LIMIT)` before the first result exists.
`limit(10)` therefore reaches exactly one index. The unbounded read is deliberate — the in-tree comment
says *"NO_LIMIT is passed to processIntersectingRetrievals to prevent incomplete intersections, which
could lead to missed results"* — and issue **#4931** (closed by PR #4941, milestone 1.2.0) is the other
half of that story: `JointIndexQuery.updateLimit` propagates a limit into a subquery **only when
`queries.size() == 1`**, so a multi-subquery joint query leaves subquery 0 at `NO_LIMIT` and its cache
key does not vary with the outer limit. Lesson 22 should not describe the graph-centric path as
limit-bounded; it is limit-bounded in exactly one place.

**Second fact, and the one to be careful about: "fitted" means something different on this side.**
Lesson 20's `isFitted` was positional — the chosen relation index's extended sort-key prefix consumed
with no gap. Here it is a **count**: `coveredClauses.size() == queryConditions.numChildren() ||
coveredClauses.contains(queryConditions)` (`:430`). The second disjunct exists because the mixed-index
`Or` path adds the **entire** condition to the cover in one go. Do not carry lesson 20's definition
across.

**Third, a layout-level fact that will keep paying: the full scan is not a flag.** It is an **empty
`JointIndexQuery`**, produced whenever `coveredClauses.isEmpty()` (`:432-434`), and the executor
branches on `indexQuery.isEmpty()`. `query.force-index` (default `false`) turns that branch into a
`JanusGraphException`; otherwise it is `log.warn("Query requires iterating over all vertices …")` plus a
profiler annotation `fullscan=true`. Lessons 25-28 can treat "no index covered a single clause" and
"full graph scan" as the same state.

**Fourth, the shipped limit defaults are the opposite of the constants in the file.**
`query.smart-limit` defaults to **`false`** and `query.hard-max-limit` to **`Integer.MAX_VALUE`**, so
`DEFAULT_NO_LIMIT = 1000` and `MAX_BASE_LIMIT = 20000` are opt-in and an unlimited graph query asks its
backend for everything in one request. `QueryTest#testQueryLimitAdjustment` measures the multiplier end
to end: `limit(5)` with 0, 1 and 2 uncovered clauses prints `@5`, `@10`, `@20` — `limit × 2^uncovered`,
exactly, via `QueryUtil.adjustLimitForTxModifications` with
`uncoveredAndConditions = queryConditions.numChildren() - coveredClauses.size()`.

**The re-pin decision stays CLOSED, and it is now DUE NEXT RUN.** Lesson 21 did **not** re-run
`list_commits` — it is mid-Part, as lessons 18, 19 and 20 instructed. It got **two** free cross-checks,
both exact: `GraphDatabaseConfiguration` measured **2191** lines, identical to lessons 18 and 19's
independent counts, and `QueryUtil`'s `processIntersectingRetrievals` sits at **`:463`**, exactly where
lesson 20 cited it (lesson 20 reported the file's total as 521 lines against this run's 520; since the
cited member line agrees to the line, the discrepancy is in the reported total, not a moved pin — worth
knowing when using totals as a pin check). **Lesson 22 is the Part IV/V boundary and owes the review:
re-run `list_commits` on `JanusGraph/janusgraph` `master` and either re-pin and record the new commit
here, or record the decline.**

**The roll-off rule applied again.** The three most recent are now 19, 20 and 21, so **lesson 18's
`Verified by` block is removed** (it survives verbatim in git history at the 2026-11-16 through
2026-11-18 commits) and lesson 18's session-log paragraph was compressed this run. The two facts that
block owed forward are **not** dropped: **lesson 27** still owes the CQL/HBase feature-builder
classification — `InMemoryStoreManager` never calls `.locking(...)` and does call `.keyConsistent(...)`,
so it is wrapped and runs the full `ConsistentKeyLocker` protocol, while `BerkeleyJEStoreManager` sets
`.locking(true)` and is native, so none of the `consistentkey` package runs — and **lesson 25** still
starts at `ManagementSystem.createCompositeIndex:848-850`, per lesson 17. Next run, lesson 19's block
and paragraph are the ones that go; carry forward lesson 19's four-tier cache model and its
`ExpirationKCVSCache` release constants, which lessons 22 and 27 both still need.

**The registers file was out of sync and is now repaired.** `TRACK.md` recorded that lesson 20 appended
candidates **43-45**, but `tracks/janusgraph-registers.md` still ended at **42** with
`last-updated: 2026-11-17 (lesson 19 …)` — lesson 20 updated the pointer here and never wrote the
register. This run recovered 43, 44 and 45 verbatim from lesson 20's own entry (§2, §4 and §7 of
`2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md`), appended them with that
provenance stated, and then appended this run's **46-51**. The register and this file now agree: **1-51
present, next free 52.** *A run that logs candidates must write the register in the same push as
`TRACK.md`; do not trust the pointer alone.*

- **The mode question is closed. Do not reopen it unprompted.** Entries from lesson 05 on write in
  `lesson` mode without re-litigating the decision, without a notice at the top, and without nagging.
  If Aryant asks for reading sessions back, flip `mode:` and the entries resume the other shape from
  the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Lesson 22 (`QueryProcessor`) now has a fully specified input, and owes three things.** (a) What the
  generic processor does with an unfitted or unsorted `BackendQueryHolder` — how `ResultMergeSortIterator`
  and `ResultSetIterator` compose, and whether the in-memory filter runs before or after the limit.
  (b) `LimitAdjustingIterator` proper, noting that `SubqueryIterator` implements `BackendLimitAware` with
  `hasBackendReachedLimit() == subQuery.hasLimit() && position >= subQuery.getLimit()`,
  `getBackendResultsRead()` and a `skipBackendResults(long)` guarded by
  `Preconditions.checkState(!started, …)` — and that with `smart-limit` off by default there is often
  nothing to adjust. (c) The **re-pin review**.
- **Lesson 23/24 (the strategy layer) inherit a settled answer, not a question.** The strategy layer
  **cannot** fold a `has()` into a compiled query. `JanusGraphStep` calls
  `centricQueryBuilder.requireOrderValues()` and then `constructQueryWithoutProfile(…)` on a fresh,
  mutable builder; `GraphCentricQuery` is immutable and exposes no way to add a condition. The only
  post-compilation rewrite in the tree is `JointIndexQuery.narrowFirst`, used by
  `withoutResultsAfter` to add a range on the first order key, and it rewrites the **backend** query,
  not the condition tree. Start from `JanusGraphStep.executeGraphCentricQuery` and
  `JanusGraphLocalQueryOptimizerStrategy`.
- **Lesson 25 (`ManagementSystem`) now inherits three.** (a) and (b) unchanged from lesson 20 — whether
  any shipped relation type has `isInvisibleType() == true` without being a system type, and
  `mgmt.buildEdgeIndex`/`buildPropertyIndex` with the `TypeDefinitionCategory.RELATIONTYPE_INDEX` schema
  edge — plus **(c) new this run:** reads skip a **composite** index unless
  `index.getStatus() == SchemaStatus.ENABLED`, with **no base-type exemption** of the kind lesson 20
  found on the relation-index path, while a **mixed** index is filtered **per `ParameterIndexField`
  status** inside `coversAll`. Lesson 25 owns the state machine that moves those statuses and should
  state which of the two a reindex relies on. Still starts at
  `ManagementSystem.createCompositeIndex:848-850`.
- **Lesson 26 (`ElasticSearchIndex`) inherits three feature flags, each of which changes a planner
  decision this run could only name:** `supportNotQueryNormalForm()` (decides whether a top-level `Or`
  has any candidate at all, since composite indexes are excluded outright),
  `supportsOrderingListProperty()` (decides `indexCoversOrder` for a `LIST`/`SET` key — Elasticsearch,
  OpenSearch and Solr can, Lucene cannot), and `supportsExistsQuery()` (decides whether `has(key)` can
  be answered from the index, and therefore whether `requireValues` can narrow a subquery).
- **Lesson 27 (CQL) inherits two.** The `setDirectColumnByStartOnlyAllowed` flag from lesson 20, and now
  the feature-builder classification lesson 18's block owed: whether `CQLStoreManager` and
  `HBaseStoreManager` set `.locking(...)` and so whether `ConsistentKeyLocker` runs on them.
- **Open and unresolved, deliberately not filled in.** (a) Lesson 20's
  `BasicVertexCentricQueryBuilder:711` `Preconditions.checkArgument(bestCandidate != null, …)`
  reachability question is still open and is **not** the same question as this side's: the graph-centric
  planner has no equivalent assertion, because an empty cover is a legitimate outcome (the full scan).
  (b) `IndexCandidateGroup.getTotalScore()` memoises with a `Double.NEGATIVE_INFINITY` sentinel while
  `IndexCandidate`'s default score **is** `Double.NEGATIVE_INFINITY`, so a candidate whose `setScore`
  was never called makes its group's total `−∞` *and* unmemoisable. Reachable only through a custom
  strategy; worth one sentence inside candidate 50's issue rather than a candidate of its own.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, **21 ~103** (a large share of each is
  permalink URLs, which are not prose; this run's Sources section alone is ~19 KB of them). The old
  5-9 KB budget was a `contributor-reading` constraint and does not apply.
  **Mechanical notes, carried and confirmed this run.** A single `create_or_update_file` call carrying
  ~103 KB succeeds. `get_file_contents` on a file whose *doubled* output exceeds roughly 40 KB returns
  an over-limit error that saves the JSON to a local file — the cheap way to read a large source file
  **and the only way to get trustworthy line numbers**, since a file returned inline has none, and it
  costs nothing in context. **A warning learned the hard way this run:** the defensive parser must take
  the **longest** `text` value in the saved JSON, not concatenate all of them — the saved payload
  contains the file twice (once as `text`, once inside a nested `{"content": …}` resource), and a naive
  join produces a file with the JSON tail appended, which silently shifts every line number past the
  real end of file. Verify by checking the byte count against the size `get_file_contents` reports for
  the path. Files read inline this run were cited by member name only, per lesson 20's split.
  `README.md`, `TRACK.md` and `tracks/janusgraph-registers.md` are all far over the limit, so all three
  are read to local files and edited with a script rather than re-emitted from context.
- **Method note; lesson 21 is the ninth entry where simulation was not needed.** Lessons 10-12
  re-implemented the pinned encoding functions and ran them; lessons 13-21 computed from the read
  sources instead and said so. This run's derived claims — the inert `getIndexTypeScore` branch, the
  `k`-blind `IN` score, the inert `ALREADY_MATCHED_ADJUSTOR` for composite indexes, the non-deterministic
  brute-force tie-break, and the unreachable retry loop in `processIntersectingRetrievals` — are each
  labelled where they appear and each paired with an exercise that runs the real code: Hands-On A makes
  the profiler print a two-subquery joint query and a `multiKSQ[3]`, B breakpoints
  `calculateIndexCandidateScore` and `IndexCandidateGroup.compareTo` so the 2008-versus-8 comparison is
  read off a live planner, and C breakpoints `processIntersectingRetrievals` so `subLimit ==
  Integer.MAX_VALUE` and the single pass are observed rather than argued. **Keep the rule that produced
  them:** compute or re-implement rather than assert, say which, and always pair it with an exercise
  that runs the real code. **Render every Mermaid diagram with `mmdc` before pushing** — both of this
  run's diagrams and the digest's rendered clean on the first attempt, with the lesson-19 rule observed
  throughout (quote every node label containing parentheses, commas, `->`, `|` or angle brackets, use
  HTML entities such as `&lt;` for generics, and never escape a quote inside a label — drop the inner
  quotes instead).
- **Daily Diff procedure — the three-run streak of the front page running ahead ENDED this run.** At
  ~03:40 UTC the front page served **2026-11-18 / 61 stories**, `/archive/` named **2026-11-18** as the
  newest and listed 2026-11-01 through 2026-11-18, and `https://tdd.cat/2026-11-19/` returned a
  `client_error`: the two indexes **agreed**, and today's edition had not published. The procedure
  stands unchanged: (a) read the front page and the archive and take the maximum; (b) probe the dated URL
  for today and yesterday; (c) check this session log for the newest edition already covered — 13 covered
  11-10, 14 covered 11-11, 15 covered 11-12, 16 covered 11-13, 17 covered 11-14, 18 covered 11-15,
  19 covered 11-16, 20 covered 11-17, 21 covered 11-18, so every run so far has found exactly one
  uncovered edition; (d) the uncovered-item fallback is still the last resort, and the 2026-11-06 edition
  still has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). WebFetch caches per URL for 15
  minutes, so a retry must be genuinely later — do other work in between. Edition sizes remain volatile:
  22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61 across 11-08 to 11-18.

### Verified by lesson 21, for lessons 22, 23, 24, 25, 26, 27 and 28 to cite rather than re-derive

- **One compile method, three outcomes, and `numSubQueries()` is hard-coded `1`.**
  `GraphCentricQueryBuilder.constructQueryWithoutProfile(ElementCategory)` is `:332-436` and returns a
  `GraphCentricQuery(resultType, queryConditions, orders, BackendQueryHolder<JointIndexQuery>, limit)`.
  `GraphCentricQuery.numSubQueries()` returns a literal `1` — the fan-out lives **inside**
  `JointIndexQuery`, not beside it, which is the opposite of lesson 20's product of holders. Early exits:
  `limit == 0` → `emptyQuery` (`FixedCondition(false)`, limit 0, `isEmpty()` is `getLimit() <= 0`), and
  `QueryUtil.constraints2QNF` returning `null` → `emptyQuery`.
- **Fluent-layer rewrites, and three differences from the vertex-centric builder.** `has(key)` is
  `has(key, Cmp.NOT_EQUAL, null)`; `hasNot(key)` is `has(key, Cmp.EQUAL, null)`; `interval(k,s,e)` adds
  two constraints; `Contain.NOT_IN` **also** calls `has(key)` first. Differences: `orderBy` accepts
  **several** keys here (the only guard is `!orders.containsKey(key)`; the cardinality check is
  commented out in-tree at `:236-237`), constraints are keyed by `String` and resolved to a
  `RelationType` only inside `constraints2QNF`, and there is no `ADJACENT_ID` diversion.
- **`constraints2QNF(tx, constraints)` is the two-arg overload, i.e. `stringsByCodePoint = true`.** That
  flag is the whole semantic difference between the two families' condition trees: a graph query compares
  strings in `Cmp` ranges **by code point** (as the mixed indexes and `OrderList` do), a vertex-centric
  query **by UTF-16 code unit** (as the edgestore's column bytes do). `QueryTest`'s `BY_CODE_POINT`
  array and `#testGraphQueryConditionsCompareStringsByCodePoint` /
  `#testInMemoryOrderComparesStringsByCodePoint` pin the pair.
- **`constraints2QNF`'s decision table.** `atomKey == null` → `null` (unsatisfiable); unknown key with
  `Cmp.EQUAL null` → `continue` (trivially satisfied), unknown key otherwise → `null`; empty
  `Contain.NOT_IN` → `continue`; empty `Contain.IN` → `null`; 1-value `IN` → a plain `Cmp.EQUAL`;
  *n*-value `IN` → a nested `Or` of equalities; *n*-value `NOT_IN` → *n* flat `Cmp.NOT_EQUAL`s; an
  `OrJanusPredicate` → an `Or` with an in-tree `// FIXME:` admitting it may not be in normal form.
- **Candidate generation is wider than lesson 20's and excludes composite indexes under an `Or`.**
  `IndexSelectionUtil.getMatchingIndexes` traverses the **whole** condition tree and collects
  `((InternalRelationType) key).getKeyIndexes()` for every `PredicateCondition`, at any depth; a condition
  on an edge label contributes nothing. The filter is
  `indexType.getElement() == resultType && !(conditions instanceof Or && (indexType.isCompositeIndex() || !serializer.features((MixedIndexType) indexType).supportNotQueryNormalForm()))`
  — so **a top-level disjunction has no composite candidate at all**, and a mixed candidate only if its
  backend takes non-normal-form conditions. The expression depends on `||` short-circuit for its own
  type safety.
- **`createIndexCandidate` returns `null` — "not a candidate" — in six distinguishable cases:** a
  schema-constrained (`indexOnly`) index with no single-valued label equality (and the label clause, when
  present, is a **free covered clause**); a composite index whose condition is not in query normal form;
  a composite index not `SchemaStatus.ENABLED` (**no base-type exemption**, unlike lesson 20's relation
  indexes); a composite index with **any** field position lacking an equality (`constructIndexCover`
  leaves `indexCovers` empty); a mixed index whose `coversAll` filter accepts nothing; and an empty
  `subCover` for any reason.
- **A composite index's `IN` lists multiply out into tuples, not sub-queries.** `constructIndexCover`
  recurses position by position and emits one `Object[]` per combination of `IN` values across all
  fields, so a 2-field index with a 3-value and a 2-value `IN` yields **six** tuples →
  `IndexSerializer.getQuery(CompositeIndexType, List<Object[]>)` → six `KeySliceQuery`s inside **one**
  `MultiKeySliceQuery`, each with column range `[0x00, 0xFF)` (`BufferUtil.zeroBuffer(1)` to
  `oneBuffer(1)`). The `unique` flag of that `MultiKeySliceQuery` is
  `index.getCardinality() == Cardinality.SINGLE` — the one place cardinality actually matters.
- **`coversAll` is where the backend is consulted.** Per-field `SchemaStatus.ENABLED` (a mixed index is
  filtered field by field, not as a whole); `atom.getValue() == null` is rejected unless the predicate is
  `Cmp.NOT_EQUAL`; `has(key)` is the `existsQuery` branch, gated on
  `IndexSerializer.supportsExistsQuery`; and everything else on
  `IndexSerializer.supports(index, field, predicate)`. The `Or` branch is stricter than the `And` branch:
  it demands **every** sub-clause of **every** branch be covered and then adds the **entire** condition
  to the cover.
- **The score: four constants, two three-line functions, zero statistics.**
  `EQUAL_CONDITION_SCORE = 4`, `OTHER_CONDITION_SCORE = 1`,
  `CARDINALITY_SINGE_SCORE = CARDINALITY_OTHER_SCORE = 1000` (both branches of `getIndexTypeScore`
  return the same number → **candidate 46**); `getConditionBasicScore` gives `4` only to a
  `PredicateCondition` with `Cmp.EQUAL`, so an `Or` (an `IN`) scores `1` **independent of `k`** →
  **candidate 47**, the mirror image of lesson 20's `5.0/k`. Brute-force score is
  `Σ over subCover (basic + typeScore)`, i.e. `|subCover| × typeScore + Σ basic`.
  Approximate adds `ORDER_MATCH = 1` (a constant offset whenever `orders.isEmpty()`, exactly like lesson
  20's `+3`) and multiplies an already-covered clause's basic score by `ALREADY_MATCHED_ADJUSTOR = -1.1`
  **outside** the `+1000`, so a re-covered equality on a composite index still contributes
  `−4.4 + 1000 = +995.6` → **candidate 48**.
- **The objective function is `IndexCandidateGroup.compareTo`, and it is lexicographic:** more
  `coveredClauses` → then **fewer** `indexCandidates` → then higher `getTotalScore()`. A group's clause
  count is the **union** of its members' sub-covers, not the sum. Consequence worth remembering: **a
  mixed index scoring 5 beats a composite index scoring 1004 if it covers one more clause**, so the
  `+1000` bias only ever breaks ties.
- **Worked scores, computed from those functions** (keys `name`, `age`, `status`; composite
  `byName(name)`, `byNameAge(name, age)`, `byStatus(status)`; mixed `search(name, age, status)`; all
  `ENABLED`): `has(name,"alice").has(age,30)` → `byName` 1004 / `byNameAge` **2008** / `search` 8, winner
  `byNameAge`. `has(name,"alice").has(age,gt(30))` → `byNameAge` is **not a candidate**, `byName` 1004,
  `search` **5**, winner `search` on clause count. `has(name, within("a","b","c"))` → `byName` **1001**
  (one `Or` clause), three `KeySliceQuery`s. Two single-key composites on different keys both score 1004
  and are chosen **jointly**.
- **Three shipped strategies; the default is `threshold-based` with a threshold of 10 on RAW candidates.**
  `query.index-select-strategy` default `threshold-based` (shorthands `brute-force`, `approximate`,
  `threshold-based`, or a full class name, resolved via `Backend.getImplementationClass` against
  `REGISTERED_INDEX_SELECTION_STRATEGIES`); `query.index-select-threshold` default **10**.
  `indexCandidates.size() <= threshold` is measured on the `Set<IndexType>` **before**
  `createIndexCandidate` filtering, so the default admits `2^10 = 1024` `IndexCandidateGroup`
  allocations per compiled query, inside the `OPTIMIZATION` profiler span.
  `AbstractIndexSelectionStrategy(Configuration)` has an **empty body**; only the threshold strategy
  reads the config.
- **Brute force calls `createIndexCandidate` once per candidate; approximate calls it `n × (p + 1)`
  times.** The greedy loop rebuilds every candidate on every pass, re-walking the condition tree and
  re-allocating sub-covers and `IN` tuple lists, with nothing memoised → **candidate 49**. Greedy can
  also be strictly worse: on `has(name,"alice").has(age,gt(30))` it picks `byName` (1005) then `search`
  (−2.4) for a **two**-subquery joint query where brute force chose `{search}` alone.
- **The brute-force tie-break is non-deterministic.** `group.compareTo(bestGroup) > 0` is strict, the
  enumeration is `new PowerSet<>(new HashSet<IndexCandidate>(…))`, and `IndexCandidate` declares **no
  `equals` and no `hashCode`** — so bucket order is identity-hash dependent, and which of two
  equal-scoring indexes lands at position 0 (and is therefore **streamed** rather than materialised) can
  change between JVM runs → **candidate 50**. `bestIndexes.sort(…)` is by score **descending** and
  `List.sort` is stable, so equal scores keep that order.
- **`isSorted` survives three independent gates.** (1) The strategy's
  `IndexSelectionUtil.indexCoversOrder(index, orders, serializer)` — every order key held, each
  `supportsOrdering`, and `supportsOrderingListProperty` for a `LIST`/`SET` key; the **deprecated**
  two-arg overload checks only `indexesKey`. (2) `IndexSerializer.getQuery(MixedIndexType, …)` builds
  `OrderEntry`s only under the same predicate, else `IndexQuery.NO_ORDER`. (3) The builder's
  `sortedOnlyWhereTheIndexSorts` → `takesTheOrder(query)`, which demands **position 0**, a **mixed**
  index, `order.size() == orders.size()` (a prefix is **not** enough), and matching
  `IndexRecordUtil.key2Field` name **and** direction at every position. The builder deliberately
  distrusts the strategy, because a custom one is allowed to claim an order the serializer did not grant.
- **`requireOrderValues()` is TinkerPop `order().by()` semantics, in three stages.** `JanusGraphStep`
  calls it before compiling. It appends a `key <> null` condition per order key into a fresh `And`
  (`queryConditions` becomes that `And`, so the in-memory filter enforces it regardless); **only when
  `indexQuery.isEmpty()`** does it re-select over `sortingIndexes(resultType, withValues)` (mixed only,
  `indexCoversOrder`, and `holdsNoOrderKeyWithCustomParameters` — no parameter whose key starts with
  `ParameterType.CUSTOM_PARAMETER_PREFIX`), accepting the result only if it has **exactly one** subquery
  covering **every** clause; otherwise `requireValues(indexQuery, key)` conjoins the exists-condition
  onto each eligible mixed subquery and signals success by **returning a different object**
  (`return anyNarrowed ? narrowed : indexQuery`), which the caller tests with `!=`.
- **Limit arithmetic, and the defaults that invert the constants.** `DEFAULT_NO_LIMIT = 1000` and
  `MAX_BASE_LIMIT = 20000` are file-private at `:282-283`. `useSmartLimit` and `hardMaxLimit` come from
  the config at construction, each overridable per transaction via `customOptions`.
  `query.smart-limit` default **`false`**, `query.hard-max-limit` default **`Integer.MAX_VALUE`**,
  `query.force-index` default **`false`**. So the shipped path is
  `indexLimit = (limit == NO_LIMIT ? hardMaxLimit : limit)`, then
  `min(hardMaxLimit, adjustLimitForTxModifications(tx, queryConditions.numChildren() - coveredClauses.size(), indexLimit))`.
  `QueryTest#testQueryLimitAdjustment` measures `limit(5)` at 0, 1 and 2 uncovered clauses as `@5`,
  `@10`, `@20`.
- **Execution is stream-one/intersect-the-rest, and the rest are unbounded.**
  `StandardJanusGraphTx.elementProcessorImpl.execute:1894-1942`: subqueries `1..n-1` become
  `QueryUtil.IndexCall` lambdas going through `indexCache.get(adjustedQuery, …)` and are passed to
  `QueryUtil.processIntersectingRetrievals(retrievals, Query.NO_LIMIT)` **eagerly, as an argument**;
  subquery 0 goes to `new SubqueryIterator(…, indexQuery.getLimit(), getConversionFunction(resultType), otherResults)`.
  `SubqueryIterator` turns `otherResults` into a `HashSet` (the in-tree comment: scanning the list would
  make the intersection `O(n·m)`).
- **`processIntersectingRetrievals`' adaptive loop is unreachable from its only caller.**
  `multiplier = min(16, 2^(n-1))`; `subLimit = Integer.MAX_VALUE`, replaced by `limit * multiplier` only
  if `Integer.MAX_VALUE / multiplier >= limit`. With `limit == Query.NO_LIMIT == Integer.MAX_VALUE`:
  `n == 1` → multiplier 1 → `subLimit = Integer.MAX_VALUE`; `n >= 2` → the guard fails → `subLimit` stays
  `Integer.MAX_VALUE`. Either way a sub-result would need `2147483647` elements to clear
  `exhaustedResults`, so the `do/while` runs **exactly one pass** and the `subLimit^1.5` growth never
  executes → **candidate 51**. `search_code` found exactly one production caller. **It is an intersect,
  not a merge** — graph-centric sub-queries are conjuncts of one `And`, where lesson 20's were
  alternatives merge-sorted by `ResultMergeSortIterator`.
- **`SubqueryIterator`'s contract, which lesson 22 needs.** `findNext()` is
  `while (emittedCount < readLimit && ids.hasNext())` with membership test
  `otherResultSet == null || otherResultSet.contains(id)`; an id converting to a `null` element is logged
  `"Subquery returned invalid element id: {}"` and skipped. It implements `BackendLimitAware`:
  `hasBackendReachedLimit()` is `subQuery.hasLimit() && position >= subQuery.getLimit()`,
  `getBackendResultsRead()` is `position`, `skipBackendResults(long)` is guarded by
  `Preconditions.checkState(!started, "Ids can only be skipped before the iterator looks for its first element")`.
  `isSafeToCache()` is `emittedCount < readLimit || subQuery.getLimit() <= readLimit`, and `close()`
  probes one element ahead to distinguish "stopped at the limit" from "gave up". Its javadoc names
  `JointIndexQuery.updateLimit(int)` propagating the limit **only when `queries.size() == 1`** as the
  divergence behind **issue #4931** (`kind/bug/confirmed`, `area/index`, milestone 1.2.0, closed
  2026-09-22 by PR #4941). `QueryTest#testIndexQueryCache` is the regression suite, nine cases, using
  `assertBackendHit` / `assertNoBackendHit`.
- **The full scan branch.** Reached by `indexQuery.isEmpty()`. `config.hasForceIndexUsage()` →
  `JanusGraphException("Could not find a suitable index to answer graph query and graph scans are disabled: " + query)`;
  otherwise `log.warn("Query requires iterating over all vertices [{}]. For better performance, use indexes", query.getCondition())`,
  a nested `"scan"` profiler with `FULLSCAN_ANNOTATION = true`, and `getVertices()` / `getEdges()` /
  `new VertexCentricEdgeIterable(getInternalVertices(), RelationCategory.PROPERTY)` by result type, in a
  `ProfiledIterator<>`.
- **The transaction overlay, for lesson 22 and for lesson 28's stale-index work.** `getNew(query)`
  short-circuits to `emptyIterator()` when `numSubQueries() == 1 && getSubQuery(0).getBackendQuery().isEmpty()`
  (a full scan already sees new elements). For vertices it `collectPredicates` the tree tracking negation
  parity, then unions `addNewVertices` — the conjunct with the **fewest** added equality properties,
  counted via `addedPropertyIndex.count(...)`, with `-1` meaning "not an equality conjunct" — and
  `addLoadedVertices`, the pre-existing vertices whose indexed keys this transaction changed, including
  order keys when `isSortedByAnIndex(query)`, plus vertices a removal could newly match
  (`MATCHED_BY_ONE_VALUE` is the predicate set for which a removal can make a condition hold).
  `isDeleted` is `(v.hasAddedRelations() || v.hasRemovedRelations()) && !query.matches(result)` for a
  vertex and `result.isNew() && !query.matches(result)` for a relation.
  `withoutResultsAfter` narrows subquery 0 with a range on the first order key via
  `JointIndexQuery.narrowFirst`, bailing out unless the key is `Cardinality.SINGLE`, the field passes
  `isRangedAsOrdered` (`ENABLED`, no custom parameters; float/double only ascending and not `NaN`;
  `Instant` only on whole milliseconds; `String` only on a `STRING`/`TEXTSTRING` field with no unpaired
  surrogate) and the backend `supports` the range predicate.
- **Reading a composite index back.** `IndexSerializer.query(Subquery, …)`:
  `sq.execute(tx, tx.getConfiguration().useMultiQuery(), maxKeysPerRead())` — batching is the
  transaction's `query.batch` setting, not index-specific — then per entry
  `readBuffer.movePositionTo(entry.getValuePosition())` and `IDHandler.readVertexId(readBuffer, true)`
  for a vertex (wrapped in a `VertexWithInlineProps` carrying the index's inline properties, which is how
  a covering composite index avoids a second edgestore read) or `bytebuffer2RelationId` otherwise. A mixed
  index goes through `tx.indexQuery(index.getBackingIndexName(), query.getMixedQuery())` mapped by
  `IndexRecordUtil::string2ElementId`.
- **Test entry points established this run.** `QueryTest` (`janusgraph-test/src/test/java/org/janusgraph/graphdb/query/QueryTest.java`,
  1105 lines, 22 `@Test` methods) runs entirely on `InMemoryStoreManager` and is the dedicated test class
  for this compiler — `#testQueryLimitAdjustment`, `#testIdLookUpQuery`, `#testMultipleKeysQuery`,
  `#testIndexQueryCache`, `#testMultipleIndexQueryWithLimits`, `#testComplexConditions`,
  `#testOrBranchesCheckEachElementOnItsOwnAsTinkerGraphDoes`,
  `#testOrderLeavesOutTheElementsWithoutAKeyItOrdersByAsTinkerGraphDoes`, the three string-ordering tests,
  and `#testFuzzyMatchWithoutIndex` / `#testTextContainsPhraseWithoutIndex` / `#testTextNegatedWithoutIndex`
  for the full-scan predicate paths. Its `conditionOf(JanusGraphQuery<?>)` helper casts straight to
  `GraphCentricQueryBuilder` and calls `constructQuery(ElementCategory.VERTEX).getCondition()`, which is
  the cheapest way to assert on a compiled condition tree. `SubqueryIteratorTest` exists in
  `janusgraph-core/src/test/java/org/janusgraph/graphdb/util/`.

### Verified by lesson 20, for lessons 21, 22, 23, 24, 25 and 28 to cite rather than re-derive

- **There is one query compiler, `BasicVertexCentricQueryBuilder.constructQueryWithoutProfile`, and two
  shells over it.** `VertexCentricQueryBuilder` (one vertex, `v.query()`) and
  `MultiVertexCentricQueryBuilder` (a `Set<InternalVertex>`, `tx.multiQuery(...)`) both call
  `super.constructQuery(returnType)` and differ only in the pre-load and the result constructor.
- **Fluent-layer rewrites that happen before the compiler sees anything.** `has(key)` **is**
  `has(key, Cmp.NOT_EQUAL, null)`; `hasNot(key)` **is** `has(key, Cmp.EQUAL, null)`;
  `interval(k, s, e)` adds `GREATER_THAN_EQUAL s` **and** `LESS_THAN e`; `has(ADJACENT_ID, …)` is
  diverted into `adjacent(vertex)` inside `addConstraint`; `has(ID, …)` becomes a constraint on
  `ImplicitKey.JANUSGRAPHID` carrying `RelationIdentifier.getRelationId()` values. `orderBy` throws on a
  second key: `Preconditions.checkArgument(orders.isEmpty(), "Only a single sort order is supported on
  vertex queries")`.
- **`compileConstraints` turns conditions into a `Map<RelationType, Interval>` and returns whether all
  of them fitted.** `Cmp.EQUAL` → `PointInterval`; the four comparisons → `range(...)`; an `Or` of
  equalities on one key → a multi-point `PointInterval` via `QueryUtil.extractOrCondition` (this is how
  `Contain.IN` arrives); **`has(key)` → `new RangeInterval<>()`, an entry in the map, not a `null`**;
  anything else → `null`, which sets `isFitted = false`. `adjacentVertex` with an id →
  `PointInterval(id)` under `ImplicitKey.ADJACENT_ID`; without one → unfitted. `intersect` returns
  `null` (unfitted) when two intervals hold values of **different classes**. Any `interval.isEmpty()`
  short-circuits the whole query to `emptyQuery()` before a byte is serialized.
- **The untyped branch is one prefix slice, and its `fitted` expression is a statement about byte
  layout:** `(adjacentVertex == null && dir == BOTH || returnType == PROPERTY && dir == OUT) &&
  !conditions.hasChildren()`. `JanusGraphTest:5290-5293` asserts `v.query()` fitted and
  `v.query().direction(OUT)` unfitted, same result count, three lines apart. `sliceLimit *= 2` applies
  only in this branch, only when `dir != BOTH` and the category is `EDGE` or `RELATION`, and only when
  `sliceLimit < Integer.MAX_VALUE/3`. `computeLimit` is called **before** `conditions.add(returnType)`
  and the `VisibilityFilterCondition`, so those two structural conditions never inflate the limit.
- **The `isUnbounded`-BOTH collapse** (`:645-654`) fires when the type is an edge label, `typeDir ==
  BOTH`, **every** interval is an endless `RangeInterval`, and `orders.isEmpty()`. It emits one slice
  with `sortKey == null`, and its `fitted` is `isIntervalFittedConditions && intervalConstraints
  .isEmpty()` — so `labels("sorted").direction(BOTH).has("weight")` is **unfitted** (`:2247-2248`),
  because `has(weight)` put something in the map. The in-tree `TODO` at `:648-650` admits the condition
  is too restrictive.
- **`getExtendedSortKey(type, dir, tx)`** = declared sort keys, then `ImplicitKey.ADJACENT_ID` if
  `isEdgeLabel() && !multiplicity().isUnique(dir)`, then `ImplicitKey.JANUSGRAPHID` if
  `!multiplicity().isConstrained()`. So `friend` with `sortKey(weight, time)` and `MULTI` plans against
  `[weight, time, ADJACENT_ID, JANUSGRAPHID]`, a plain `MULTI` label against `[ADJACENT_ID,
  JANUSGRAPHID]`, and a `ONE2MANY` label queried `IN` against `[]`. **`adjacent(u)` is therefore planner
  input, not a post-filter** — it lands on a sort-key position and narrows bytes (`:5281-5284` fitted;
  `:5285-5286` unfitted, because a range on `weight` stops the descent before `ADJACENT_ID`).
- **Candidate filters, before scoring:** `if (!candidate.isUnidirected(BOTH) &&
  !candidate.isUnidirected(direction)) continue;` and `if (!candidate.equals(type) &&
  candidate.getStatus() != SchemaStatus.ENABLED) continue;`. **The status check exempts the base type.**
  So reads skip an `INSTALLED`/`REGISTERED` relation index while lesson 17's write path still feeds it —
  which is what lets a reindex converge. **Lesson 25.**
- **`EdgeSerializer.getQuery(InternalRelationType, Direction, TypedInterval[])` keeps two
  `DataOutput`s**, `colStart` and `colEnd`, writes the same relation-type prefix into both and diverges
  only where the interval does. A point writes the same value into both (via
  `VariableLong.writePositiveBackward` for `JANUSGRAPHID`, `IDHandler.writeVertexId(..., false)` for
  `ADJACENT_ID`, `writeInline(..., InlineType.KEY)` otherwise) and the loop continues; a range writes the
  two bounds and **breaks**. `DESC` swaps the two buffers and takes
  `getStaticBufferFlipBytes(keyStartPos, …)` — ones-complemented from `keyStartPos`, which is the
  position right after the relation-type prefix and is itself not flipped — and inverts both inclusivity
  adjustments. `JanusGraphTest:5277-5280` asserts identical `fitted` with `isSorted` flipping between a
  `desc` and an `asc` `orderBy` against the same `DESC` index.
- **The missing-value sentinel.** A relation without a value for a sort key serializes that position as
  `-1`, the largest byte, so it sorts after every real value. An open-ended range therefore writes an
  explicit `colEnd.putByte((byte) -1)` with `endInclusive = false` **only if**
  `leavesOutMissingValues(key)`, which delegates to `serializer.writesMissingValuesLast(key.dataType())`
  — **fitness here is a property of the data type, not of the query**. The builder re-checks the same
  predicate at `:763-770` and clears `isFitted` when it does not hold.
- **`isFitted` is `isIntervalFittedConditions && position == intervalConstraints.size()`**, where
  `position` counts extended-sort-key slots consumed with a non-null interval. One gap in the prefix and
  the equality fails — `has("time",15).has("weight",3.5)` on `connect` (`sortKey(time)`) is `{false,
  true}` at `:4819`; `interval("time",10,20).interval("weight",0.0,5.0)` is `{false, true}` at
  `:5269-5270`.
- **`isSimple()` is `queries.size()==1 && isFitted && isSorted`**, and `useSimpleQueryProcessor` adds
  `queryOnlyLoaded || every vertex isLoaded()`. `SimpleVertexQueryProcessor.vertexIds()` reads adjacent
  ids straight out of the `Entry` with `edgeSerializer.readRelation(entry, true, tx).getOtherVertexId()`
  — header-only parse, **no relation object constructed** — and tracks sortedness as it goes. Its
  `LimitAdjustingIterator` is entered only when `sliceQuery.hasLimit() && sliceQuery.getLimit() !=
  query.getLimit()`, i.e. only when `computeLimit` actually inflated the limit.
  `constructQuery(vertex, baseQuery)` ends with `Preconditions.checkArgument(!queryOnlyLoaded ||
  query.isSimple(), …)`, which is what makes lesson 17's index-delta reads safe.
- **`loadRelationsFromCache` (the `CachedRelationConstructor`, used by `MultiVertexCentricQueryBuilder`
  only) demands an *exact* cache hit, not lesson 19's `subsumes` scan.** Its guard is
  `cacheVertex.getFromCache(q.getBackendQuery()) == null || !q.isFitted() || !q.isSorted()` over **every**
  sub-query; one miss and the whole thing falls back to `executeRelations`. It also requires
  `vertex instanceof CacheVertex && !isPartitionedVertex(vertex) && vertex.isLoaded()`.
- **Limit arithmetic.** `HARD_MAX_LIMIT = 300000` (private to the builder). `computeLimit` is
  `Math.max(baseLimit, Math.min(HARD_MAX_LIMIT, adjustLimitForTxModifications(tx, remaining, baseLimit)))`,
  and `QueryUtil.adjustLimitForTxModifications` multiplies by `2^uncoveredAndConditions` (guarded by
  `Integer.MAX_VALUE / limit`) and then adds `min(MAX_VALUE - limit, 5)` when `tx.hasModifications()`.
  `remaining` is `conditions.size()` in the untyped branch and `intervalConstraints.size() - position` in
  the typed one. Worked, clean tx: `v.query().direction(OUT).limit(10).edges()` → **20**; with
  modifications → **25**; plus one uncovered condition → **40**; a fully fitted
  `.labels("friend").has("weight",0.5).limit(3)` → **3**. The in-tree comment is a design statement:
  *"Don't be smart with query limit adjustments - it just messes up the caching layer"* — every distinct
  limit is a distinct `SliceQuery` and therefore a distinct cache key at both tiers.
- **Partitioned-vertex merge rules differ by result type.** `executeRelations` merge-sorts with
  `RelationComparator.compareOnOrders(r1, r2, baseQuery.getOrders())`; `executeVertices` merge-sorts with
  `VertexArrayList.VERTEX_ID_COMPARATOR` but **first** degrades to the relation path when
  `!orders.isEmpty()`; `executeVertexIds` does **not** merge-sort at all — it concatenates with
  `merge.addAll(...)` and breaks once `merge.size() >= baseQuery.getLimit()`.
- **Property prefetch.** `VertexCentricQueryBuilder.execute` prefetches all properties only when
  `returnType == PROPERTY && hasSingleType() && !hasQueryOnlyLoaded() &&
  tx.getConfiguration().hasPropertyPrefetching()` **and** `!propertiesLoaded(vertex)`, where the guard is
  `holder.hasLoadedRelations(ALL_PROPERTIES_SLICE)` against a `static final` `SliceQuery` built from
  `IDHandler.getBounds(PROPERTY, false)` — shared and never mutated, with an in-tree comment saying the
  point is to allocate no query per access. When prefetching is on, the `size() > 1` multi-slice pre-load
  is **skipped**.
- **`SliceQuery` extras set here.** `getQuery(type, dir, sortKey)` returns `new SliceQuery(sliceStart,
  sliceEnd, type.name())` — the third argument is the type name, carried for profiling — and calls
  `.setDirectColumnByStartOnlyAllowed(isDirectColumnByStartOnlyAllowed(type))`, which is `true` only for
  a `PropertyKeyVertex` whose multiplicity has `Cardinality.SINGLE`. **Lesson 27** should check what the
  CQL store does with that flag.
- **Test entry points established this run.** `JanusGraphBaseTest.evaluateQuery(JanusGraphVertexQuery,
  RelationCategory, int expectedResults, int numSubQueries, boolean[] {fitted, ordered}, …)` at
  `:589-659` is the measuring instrument: it attaches a `SimpleQueryProfiler`, walks the profiler's
  children skipping `OPTIMIZATION` and anything annotated `MULTI_SLICES_ANNOTATION`, and asserts
  `assertEquals(numSubQueries, subQueryCounter)` plus
  `assertEquals(Integer.valueOf(1), profiler.getAnnotation(NUMVERTICES_ANNOTATION))`. The vertex-centric
  suites are `JanusGraphTest#testVertexCentricQuery(int)` (`:4726`, 10,000 vertices by default;
  `BerkeleyGraphTest#testVertexCentricQuerySmall` overrides to 1,450),
  `#testVertexCentricOrderLeavesOutTheRelationsWithoutTheKey` (`:2120-2262`, the `{3, null, 1, 2, null,
  5}` data set and the float/NaN assertions) and `#testRelationTypeIndexes` (`:5086`, which builds six
  relation indexes including two on the same label in opposite orders). **None of the three is in
  `InMemoryGraphTest`'s `@Disabled` list**, so all three run on `janusgraph-inmemory`. A `search_code`
  for `VertexCentricQueryTest` returns **0 hits** — this compiler has no dedicated unit test.

### Verified by lesson 19, for lessons 20, 21, 22, 25, 27 and 28 to cite rather than re-derive

- **Every user vertex in an ordinary transaction is a `CacheVertex`.** `VertexConstructor.get`
  (`StandardJanusGraphTx:624-664`) ends `if (idInspector.isUserVertexId(vertexId)) { if
  (createStubVertex) vertex = new PreloadedVertex(...); else vertex = new CacheVertex(...); }`, and
  `createStubVertex` is `config.hasPreloadedData()`. `StandardVertex` is never instantiated directly for
  user data; it is only the base class. Schema ids branch earlier to `PropertyKeyVertex`,
  `EdgeLabelVertex`, `VertexLabelVertex`, `JanusGraphSchemaVertex`, or `SystemTypeManager.getSystemType`.
- **Three `VertexConstructor`s, built once per transaction at `:346-348`**, differing only in
  `verifyExistence`: `externalVertexRetriever` = `hasVerifyExternalVertexExistence()` (default **true**,
  used by `getVertex`, i.e. `g.V(id)`), `internalVertexRetriever` = `hasVerifyInternalVertexExistence()`
  (default **false**, used by `getInternalVertex`), `existingVertexRetriever` = **false** (schema and
  partitioned lookups). `verifyExistence` costs **one slice query per id** —
  `graph.edgeQuery(canonicalVertexId, graph.vertexExistenceQuery, txHandle)` — batched in
  `getVertices(Object...)` as `graph.edgeMultiQuery(vertexIds, graph.vertexExistenceQuery, txHandle)`
  (`:570-577`) for ids not already cached.
- **Tier 1 is keyed by `SliceQuery`; tier 2 is keyed by `KeySliceQuery`, not by row key** (for lessons
  20 and 22). So one edgestore row holds **one db-cache entry per distinct query shape**, while
  `invalidate` works on the row key — which is the entire reason the `expiredKeys` side table exists:
  there is no index from key to the `KeySliceQuery`s that mention it.
- **`CacheVertex` lookup is two-stage: exact `HashMap` hit, then a linear `subsumes` scan.**
  `loadRelations` does `queryCache.get(query)` under `synchronized (queryCache)`; on a miss, `load`
  calls `getSuperResultSet(query)`, which iterates the map asking `entry.getKey().subsumes(query)`, and
  on a hit returns `query.getSubset(superKey, superVal)` **with no backend read**. The map is
  `new HashMap<>(4)` and its own monitor. `hasLoadedRelations` = `queryCache.get(query) != null ||
  getSuperResultSet(query) != null`.
- **The refresh counter is a lost-update guard with two entry points.** `refresh()` does
  `refreshes++; queryCache.clear();`; `addToQueryCache(q, entries, refreshesBefore)` stores **only if**
  `refreshes == refreshesBefore`. The two-argument `loadRelations` reads the count itself via
  `refreshes()`; the three-argument overload takes it from the caller, because a multi-query reads many
  vertices in one backend call and hands out results afterwards — `executeMultiQuery:1428-1451`
  snapshots `((CacheVertex) v).refreshes()` per vertex before the read, and
  `executeMultiSliceMultiQuery:1453-1467` does the same into a `Map<Object, Long>` keyed by vertex id
  (for **every** `CacheVertex` in the collection, not only the ones it reads).
- **The public two-argument `CacheVertex.addToQueryCache(query, entries)` skips the refresh check**, and
  its only caller is `StandardJanusGraphTx`'s `vertexIDConversionFct` (`:2091-2103`), which stuffs
  `VertexWithInlineProps.getInlineProperties()` into the vertex's query cache after a graph-centric
  index read. **PR candidate 42**, derived from reading.
- **`CaffeineVertexCache` pins reactively, not preventively.** `add` eagerly puts into
  `volatileVertices` when `vertex.isNew() || vertex.hasAddedRelations()`; the **removal listener**
  catches everything else on the way out, with the wider predicate
  `((AbstractVertex) v).isTxOpen() && (v.isModified() || v.isRemoved())` → `putIfAbsent`. `get` consults
  `cache.getIfPresent`, then `volatileVertices`, then the `Retriever`, and **re-admits** via
  `cache.get(id, k -> v)`, which also guarantees one object per id per transaction under concurrency.
  `volatileVertices` is a jctools `NonBlockingHashMap` and is **unbounded**, so `cache.tx-cache-size`
  bounds clean vertices only. `getAllNew()` filters `volatileVertices.values()` by `isNew()` — the
  mechanism `prepareCommit` relies on. The listener's `assert removalCause == SIZE || REPLACED` is a
  real statement about configuration: no `expireAfter*`, no weak/soft values, and `EXPLICIT` only from
  `close()`, which clears `volatileVertices` first.
- **Tier-0 sizing.** `MIN_VERTEX_CACHE_SIZE = 100L` (`:206`) is applied as
  `Math.max(MIN_VERTEX_CACHE_SIZE, config.getVertexCacheSize())` **only when
  `!config.isReadOnly()`** (`:350-355`). `StandardTransactionBuilder.readOnlyOLAP()` sets
  `vertexCacheSize = 0` *and* `dirtyVertexSize = 0`, so an OLAP transaction gets `maximumSize(0)` —
  every `add` is evicted immediately with cause `SIZE`. `indexCacheWeight` is **not** a separate
  option: `StandardTransactionBuilder.vertexCacheSize(int size)` sets `indexCacheWeight = size / 2`,
  which is where the docs' *"half the transaction cache size"* and the 9,998-element ceiling come from.
  `CaffeineVertexCache` uses `.executor(Runnable::run)` and enables `recordStats()` only
  `if (log.isDebugEnabled())` **at construction**.
- **`releaseTransaction()` (`:2200-2210`) swaps in five empty singletons** — `EmptyVertexCache`,
  `EmptySubqueryCache`, `EmptyAddedRelations`, `Collections.emptyMap()` for deleted relations and locks,
  `EmptyIndexCache`, `Collections.emptyMap()` for `newTypeCache`. `EmptyVertexCache.get` returns
  **null** and logs a warning, which is why `getVertices` ends `result.removeIf(v -> v == null ||
  v.isRemoved())` with a comment naming the `graph.close()`-on-another-thread race.
- **Which stores are cached.** `Backend:257`: `cacheEnabled = !configuration.get(STORAGE_BATCH) &&
  configuration.get(DB_CACHE)`. `Backend:344-376`: `edgestore` and `graphindex` get
  `ExpirationKCVSCache` when enabled, `NoKCVSCache` otherwise; split
  `EDGESTORE_CACHE_PERCENT = 0.8` / `INDEXSTORE_CACHE_PERCENT = 0.2` (`:162-163`, `public static final`,
  with a `Preconditions` check that they sum to 1.0 — **not configurable**). `txLogStore` is
  `NoKCVSCache` off **`storeManager`** (`:381`). `DB_CACHE_TIME == 0` becomes
  `ETERNAL_CACHE_EXPIRATION = 1000L*3600*24*365*200` (`:165`, `:352`). A fractional `DB_CACHE_SIZE` is
  resolved against `runtime.maxMemory() - (runtime.totalMemory() - runtime.freeMemory())` — free heap
  **at graph-open time**.
- **Defaults, from `GraphDatabaseConfiguration:610-698`:** `cache.db-cache` **false** (MASKABLE),
  `cache.db-cache-size` **0.3** (Double), `cache.db-cache-time` **10000** ms (Long),
  `cache.db-cache-clean-wait` **50** ms (Integer), `cache.tx-cache-size` **20000** (Integer),
  `cache.tx-dirty-size` **unset** → `32` without batch loading, `4096` with
  (`TX_DIRTY_SIZE_DEFAULT_WITHOUT_BATCH` / `_WITH_BATCH`, `:691`/`:698`; resolved at `:2058-2066`).
- **Invalidation marks, never evicts, and never writes through.**
  `ExpirationKCVSCache.invalidate(key, entries)` is `expiredKeys.put(key, now + cacheTimeMS)` plus a
  1-in-`INVALIDATE_KEY_FRACTION_PENALTY` (1000) `penaltyCountdown.countDown()`. `isExpired(query)`
  looks up `expiredKeys.get(query.getKey())`, self-heals if `until < now`, otherwise counts the latch
  down and returns true. A true `isExpired` means `store.getSlice` **with no `cache.put`**. Release
  needs `PENALTY_THRESHOLD = 5` countdowns **and** `getAge(until) = now - (until - cacheTimeMS) >=
  invalidationGracePeriodMS`; `clearExpiredCache(true)` then scans `cache.asMap().keySet()` and resets
  the latch. Called from `CacheTransaction.flushInternal()` **after** `persist(subMutations)`.
- **The "expired ⇒ do not repopulate" rule is enforced by a sentinel, not a second check.**
  `fillResultAndReturnMisses` returns `Pair<List<StaticBuffer>, Map<StaticBuffer, KeySliceQuery>>` and
  puts `key -> null` for an expired key, `key -> ksqs` for a plain miss; both multi-key paths then do
  `if (ksqs != null) cache.put(ksqs, subresult)`.
- **`ExpirationKCVSCache.getMultiSlices` rewrites the query plan in place** (for lessons 20 and 22):
  per group it drops fully-served queries via `keyQueriesGroup.setQueries(...)`, collects
  `Pair.of(query, remainingCurrentGroupKeys)` for partially-served ones, calls
  `MultiSliceQueriesGroupingUtil.moveQueriesToNewLeafNode(updatedQueryGroups, allKeysArr,
  groupingRootTreeNode, allQueryGroups)`, then `filterEmptyGroups` and
  `multiKeysQueryGroups.setQueryGroups(allQueryGroups)`. The db-cache is **not** a transparent wrapper
  on this path.
- **`KCVSCache.mutate` throws `UnsupportedOperationException("Only supports mutateEntries()")`.**
  Writes go `mutateEntries` → `((CacheTransaction) txh).mutate(this, key, additions, deletions,
  wholeRowDeletion)` and are buffered; `unwrapTx` is `final` and asserts `txh instanceof
  CacheTransaction`. Deletions are kept as `Entry` (with values) until `CacheTransaction.convert`
  strips them to columns via `KCVEntryMutation.ENTRY2COLUMN_FCT`. The three documented escape hatches
  are `getSliceNoCache(KeySliceQuery, txh)`, `getSliceNoCache(List<StaticBuffer>, SliceQuery, txh)` and
  `getMultiSlicesNoCache(...)`; lesson 18's expected-value read reaches past even these via
  `getBackingStore()`.
- **The db-cache's weigher is an estimate, by the authors' admission.**
  `CAFFEINE_CACHE_ENTRY_SIZE + KEY_QUERY_SIZE + entries.getByteSize()` with
  `KEY_QUERY_SIZE = OBJECT_HEADER + 4 + 1 + 3 * (OBJECT_REFERENCE + STATICARRAYBUFFER_RAW_SIZE + 10)`,
  i.e. serialized bytes for objects Caffeine holds deserialized — the known limitation the docs page
  links as [issue 2369](https://github.com/JanusGraph/janusgraph/issues/2369). Only
  `expireAfterWrite(cacheTimeMS)` is configured; there is **no** `expireAfterAccess`, so a hot entry
  still dies at `db-cache-time`.
- **Deserialization is cached at no tier.** All tiers store `EntryList`s, so a tier-1 or tier-2 hit
  still re-runs lesson 11's `parseRelation` per entry. `RelationConstructor.readRelation(v, iterable,
  tx)` sits *after* `loadRelations` in `edgeProcessorImpl.execute` (`:1562-1573`), whose `Retriever` is
  the lambda `query1 -> QueryProfiler.profile(profiler, query1, q -> graph.edgeQuery(v.id(), q,
  txHandle))` — the vertex never holds a store reference, which is what makes `CacheVertexRefreshTest`
  possible against a mocked transaction.
- **Test entry points established this run.**
  `janusgraph-core/src/test/.../graphdb/vertices/CacheVertexRefreshTest.java` —
  `aLoadWhichARefreshOverlapsIsNotKept` (two `CountDownLatch`es, a pausing `Retriever`, asserts
  `loads == 2`) and `aResultLoadedBeforeTheCallIsKeptOnlyIfNoRefreshCameSince`; both build a
  `CacheVertex` over a Mockito-mocked `StandardJanusGraphTx` with `isNew()` overridden, so the
  transaction is never consulted. `janusgraph-test/src/test/.../diskstorage/cache/ExpirationCacheTest.java`
  extends `KCVSCacheTest`, `CACHE_SIZE = 48 MB`, `METRICS_STRING = "metrics"`; `testExpiration` and
  `testGracePeriod` are both `@RepeatedIfExceptionsTest(repeats = 3)` and both flaky by issue number
  ([2934](https://github.com/JanusGraph/janusgraph/issues/2934),
  [3352](https://github.com/JanusGraph/janusgraph/issues/3352)) **for exactly the reason this lesson
  found** — the release is a counter race, not a clock.

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
