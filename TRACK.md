<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 23
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-20 at the Part IV/V boundary (lesson 22) — PERFORMED, and the pin stays
  for the fourth review in a row. list_commits on JanusGraph/janusgraph master returns a59ae8d8 (2026-10-28,
  Restore 1.3.0-SNAPSHOT version CTR), 9a30ff14 (2026-10-28) and 5e3e7bc7 JanusGraph release 1.2.0
  (2026-10-24), then 0f578b3f (2026-10-22, fixes #5124) and b7c14c53 (2026-10-23, fixes #5126). There is
  nothing to re-pin to. Lesson 22 also replaced the line-count cross-check with a STRONGER one: it
  reconstructed the two files it cites by line number and verified their git blob SHAs —
  QueryProcessor.java 16,048 bytes = b28e68871c9a8a0f655373deaa4185cdf4da9a44 and
  LimitAdjustingIterator.java 5,961 bytes = 96a7e3ae38ff3c0138879c5688f8a62053b66220, both exactly what
  get_file_contents reported at the pinned commit. Use the blob-SHA check from here on.
  NEXT REVIEW: the Part V/VI boundary, after lesson 24 — which is the NEXT run and MUST perform it.
  Lesson 23 was mid-Part and correctly did not re-raise it.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (PR candidates 1-64, next free 65; "Still open, and who owns it")
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
**Reviewed again 2026-11-20 (lesson 22, the last item of Part IV, and the Part IV/V boundary): the review
was PERFORMED and the pin stays for the fourth time.** `list_commits` on `master` returns the same newest
commits as at lessons 13 and 18 — `a59ae8d8` (2026-10-28), `9a30ff14` (2026-10-28) and `5e3e7bc7`
*JanusGraph release 1.2.0* (2026-10-24), then `0f578b3f` (2026-10-22, fixes #5124) and `b7c14c53`
(2026-10-23, fixes #5126). There is nothing to re-pin to, so Part V's line numbers stay comparable with
lessons 02-22. Lessons 19, 20 and 21 were all mid-Part and correctly did not re-run `list_commits`; the
next review is the **Part V/VI boundary, after lesson 24**, and lessons 23-24 must not re-raise it.
**The cross-check method changed and improved this run:** instead of comparing line-count totals, lesson 22
reconstructed the two files it cites by line number and verified their **git blob SHAs** against what
`get_file_contents` reported — `QueryProcessor.java` 16,048 bytes = `b28e6887…`,
`LimitAdjustingIterator.java` 5,961 bytes = `96a7e3ae…`. Byte-identity is a proof where a line count is
only evidence; use it from here on for every file cited by line number.

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

**The registers live in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md), new on
2026-11-14, and now carrying candidates 1–60 (next free number **61**; 52–60 were added by lesson 22 —
nine, the largest batch the track has filed — and written in the **same push** as this file, per the rule
the 2026-11-19 desync produced).**
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
lesson 16 on 2026-11-17, lesson 17 on 2026-11-20, and **lesson 20's paragraph was compressed this run**;
the full paragraphs remain verbatim in git history at the commit each line names, and their substance is
carried forward either in the surviving `Verified by` blocks, in the registers file, or in the root
`README.md`'s per-lesson narrative. **Lesson 20's `Verified by` block was removed this run** and its
paragraph compressed. Next run, lesson 21's block and paragraph are the ones that go.

- **23 — 2026-11-21** · [`2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md`](2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md) · **Part V opened. Mid-Part, so the re-pin question was correctly NOT re-raised — the Part V/VI review is due after lesson 24. All three debts lesson 22 handed over are discharged.** Read the strategy layer at the pinned commit. **Blob-SHA verified and cited by line number:** `JanusGraphLocalQueryOptimizerStrategy` (whole, 245 lines, 13,515 bytes = `01de0bed`; cited at `:57`, `:63-90`, `:64-65`, `:68-70`, `:78-84`, `:85-90`, `:92-96`, `:99-117`, `:105`, `:106`, `:109`, `:110-114`, `:118-161`, `:119`, `:120-122`, `:134-137`, `:158`, `:163-197`, `:170`, `:174-177`, `:185`, `:191-193`, `:199-223`, `:225-235`, `:228`, `:231`, `:238-239`), `HasStepFolder` (whole, 532 lines, 27,466 bytes = `0232fc9b`; `:102-143`, `:145-178`, `:181-214`, `:185`, `:195-197`, `:200`, `:217-223`, `:225-245`, `:247-268`, `:269-281`, `:282-288`, `:327-343`, `:351-363`, `:367-378`, `:380-434`, `:418-420`, `:423-425`, `:437-445`, `:484-502`, `:507-531`, `:513`, `:519`), `StandardJanusGraph` (whole, 1,489 lines, 80,408 bytes = `f650acb2`; `:165-185`, `:183-184`, `:377`) and `JanusGraphStepStrategyTest` (whole, 546 lines, 46,982 bytes = `abd23a81`; `:111-122`, `:124-142`, `:144-170`, `:342-380`, `:415-416`). **Read inline, cited by member name:** `JanusGraphStepStrategy`, `JanusGraphStep`, `JanusGraphMixedIndexCountStrategy`, `AbstractJanusGraphMixedIndexAggStrategy`, `OptimizerStrategyTest`. **Read by a delegated reader this run** (paths and blob SHAs from a directly fetched directory listing; text not locally transcribed, so member-name citations only, and the entry says so): the four `AdjacentVertex*` strategies and their abstract base, `JanusGraphHasStepStrategy`, `JanusGraphMultiQueryStrategy`, `JanusGraphUnusedMultiQueryRemovalStrategy`, `JanusGraphMixedIndexAggStrategy`, `JanusGraphIoRegistrationStrategy`. Three directory listings (the strategy package, the step package, the optimize test package); four `search_code` queries, of which **two returned 0 results** — `AND_QUERY OR_QUERY TP3ProfileWrapper assert metrics` and `"backend-query" profile test assertion`. Four PR candidates logged (**61-64**), register now 1-64, next free 65; `tracks/janusgraph-registers.md` written in the **same push** as this file, per the 2026-11-19 rule. Daily Diff: **a new edition was available and the fallback was NOT needed** — front page and `/2026-11-19/` both serve the 2026-11-19 / 61-story edition while `/archive/` advertises a 50-story 2026-11-20 whose URL returns `client_error`; deep dive on PostgreSQL 19's `REPACK CONCURRENTLY` and the `pg_plan_advice` / `pg_stash_advice` extensions.

  **Findings.** **THE finding: the content of the strategy layer is its dependency graph, and two of the twelve strategies declare no ordering at all.** `StandardJanusGraph`'s one `static` block (`:165-185`) registers twelve strategies, **all** of them `ProviderOptimizationStrategy` — including `JanusGraphIoRegistrationStrategy`, whose entire `apply()` configures an `IoStep` — and registers the same object for `StandardJanusGraph.class` and `StandardJanusGraphTx.class`. OLAP gets **one** of the twelve (`FulgoraUtil.FULGORA_STRATEGIES` is `JanusGraphLocalQueryOptimizerStrategy` alone). Reading the full `applyPrior`/`applyPost` graph produces the fact Part V turns on: **`JanusGraphStepStrategy` and `JanusGraphLocalQueryOptimizerStrategy` have no direct ordering edge**, and the only paths between them run through `JanusGraphMixedIndexCountStrategy` and `JanusGraphMixedIndexAggStrategy`, which a user disables with one `withoutStrategies(...)` call. No traversal was found where the order is observable — the two target disjoint step types and `HasStepFolder`'s walks break on any unrecognised step — so it is filed as a missing declaration (candidate 61), not a defect. Second: **`local()` is the only construct that gets an order or a range to the storage layer.** Pass 1 folds a `has()` only for an edge-returning step, folds no order and no range (`:106`: *"We cannot fold in orders or ranges since they are not local"*) and folds only a *high* limit from a following `RangeGlobalStep` **without removing it**; pass 5 calls `foldInHasContainer`, `foldInOrder` **and** `foldInRange`, sets `setBatchSize(txVertexCacheSize)`, and `unfoldLocalTraversal` deletes the `LocalStep` entirely when its child collapses to one step. A vertex-centric query folds **at most one** order key (`:423-425`); a graph-centric one folds any number. **Pass 1 never calls `setBatchSize` at all** — handed to lesson 24 as an explicit open question rather than guessed. Third, lesson 22's profiler debt: **the attachment order on the normal path is safe** — `addNested(constructGraphCentricQuery)` → `requireOrderValues()` → `constructQueryWithoutProfile` → `observeWith(addNested(GraphCentricQuery))` → `ProfiledIterator` → `iterables` → `new QueryProcessor` — but there is a hole in the strategy layer: `buildGlobalGraphCentricQuery()` **memoizes**, and `JanusGraphMixedIndexAggStep`'s constructor calls it during strategy application, before TinkerPop's `setMetrics`, so a traversal ending in `count()`/`min()`/`max()`/`sum()`/`mean()` is **compiled under `QueryProfiler.NO_OP`, has its index selection done at strategy time, and shows no `constructGraphCentricQuery` span in `.profile()`** when the push-down is declined — which is the common case with no mixed index. The probe also mutates the step it declined to replace (`hasLocalContainers.put(null, {[]})`); `toString()`, `hashCode()` and the or()-fallback switch were all traced and none changes behaviour (candidate 62). Fourth, the second debt is answered **no**: the TinkerPop side never produces a multi-sub-query `ElementQuery`. `or()` is folded only for a `JanusGraphStep` (`:185`), each branch flattened into a `List<HasContainer>` — with an `order().by(k)` inside a branch becoming `has(k, neq(null))`, because TinkerPop drops elements a `by()` cannot value — and then executed either as **one** disjunctive `GraphCentricQuery` or as **N** single-sub-query ones merged by `MultiDistinctUnorderedIterator`/`MultiDistinctOrderedIterator` **in the step**, where the step's low limit can also be applied. So candidates 55 and 56 stand and the right PR is a **comment**, not a deletion. Fifth, the third debt: `search_code` found **nothing** in the repository asserting profile-tree shape, so the `AND-query`/`OR-query` flattening is not a tested contract and candidate 60 must be filed before any new-span patch. Also worth keeping: `validFoldInOrBranch` is **all-or-nothing across a whole `or()`** — one unrecognised step in one branch abandons the fold for every branch (candidate 64) — and `AdjacentVertexHasUniquePropertyOptimizerStrategy` is the one strategy permitted to touch the backend, gated on `optimizerBackendAccess()`, which it uses to **run a side traversal during strategy application** and resolve a unique key to a concrete vertex id, substituting a `DiscardStep` when there is no match.

- **22 — 2026-11-20** · [`2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md`](2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md) · **Part IV complete. The Part IV/V re-pin review was PERFORMED and the pin stays for the fourth time. All three debts lesson 21 handed over are discharged.** Read the whole `graphdb/query` top level and the profile package at the pinned commit: `QueryProcessor` (whole, 320 lines, **blob-SHA verified** — 16,048 bytes = `b28e6887`; cited at `:56`, `:69-74`, `:76-162`, `:80-119`, `:87-88`, `:93`, `:95-100`, `:101-107`, `:109-111`, `:114`, `:120-160`, `:139-151`, `:170-176`, `:187-224`, `:227-237`, `:239-245`, `:247-296`, `:257-275`, `:298-318`), `LimitAdjustingIterator` (whole, 131 lines, **blob-SHA verified** — 5,961 bytes = `96a7e3ae`; `:59-65`, `:76-113`, `:78-86`, `:87`, `:91-96`, `:99-108`), plus `ResultSetIterator`, `ResultMergeSortIterator`, `BackendQueryHolder`, `QueryExecutor`, `MetricsQueryExecutor`, `ElementQuery`, `BackendLimitAware`, `QueryProfiler`, `SimpleQueryProfiler`, `TP3ProfileWrapper`, `GraphCentricQuery`, `BaseVertexCentricQuery`, `VertexCentricQuery`, `CloseableIteratorUtils`, `MultiIterator`, `SubqueryIterator` (re-read) and `QueryProcessorTest` — all whole, all cited by member name. `search_code` for `new QueryProcessor`, `hasSiblings addNested`, `implements ElementQuery`, `useSimpleQueryProcessor` and `BackendLimitAware`; `list_commits` on `master`. **THE finding: the in-memory filter sits between the backend limit and the element count**, so `limit(10)` bounds what the caller receives and not what is read, and the backend limit **doubles** — `L, 2L, 4L …` with a full re-read each time, no cursor — until enough survive. Second: **`QueryProcessor` creates no profiler span at all** (three pass-through uses, no `addNested`/`startTimer`/`setResultSize`), so merge, sort and filter time is invisible in `.profile()` and in Metrics; the only external traces are the gap between a `backend-query` span's `readCount` and the elements returned, and repeated sibling `backend-query` spans with doubling `limit` annotations. Third: **half the class is dead** — `VertexCentricQuery.isSorted()` is hard-coded `true` and `GraphCentricQuery.numSubQueries()` hard-coded `1`, so the unsorted branch's `concat` + de-dup arm (`:139-150`) is unreachable and `ResultMergeSortIterator`'s `filterDuplicates` is dead in both implementations. Fourth: **the single-slice fitted sorted vertex-centric case does not use `QueryProcessor` at all** — `useSimpleQueryProcessor` diverts it to `SimpleVertexQueryProcessor`. Nine PR candidates logged (**52-60**, the largest batch so far), register now 1-60, next free 61; `tracks/janusgraph-registers.md` written in the **same push** as this file, per the 2026-11-19 rule. Daily Diff: **no 2026-11-19 or 2026-11-20 edition exists** (front page and `/archive/` agree on 2026-11-18, both dated URLs `client_error`) and 2026-11-18 was already digested by lesson 21, so the uncovered-item fallback was used: #15, Percona on Galera `pc.weight` — whose printed quorum formula does not match the `have_quorum` code it quotes.

- **21 — 2026-11-19** · [`2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md`](2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md) · **stayed on `a59ae8d8`. All three debts lesson 20 handed over are discharged, and the answer to the headline question is "same kind of function, different shape": the graph-centric planner is also statistics-free, but it solves a weighted set cover over the query's clauses, may choose several indexes, and may choose none — which is the full scan.** Read the whole graph-query and index-selection surface at the pinned commit: `GraphCentricQueryBuilder` (whole, 612 lines, numbered locally; cited at `:177-249`, `:236-238`, `:271-274`, `:282-283`, `:302-330`, `:332-436`, `:448-465`, `:471-495`, `:503-524`, `:532-551`, `:561-597`, `:603-611`), `GraphCentricQuery`, `JointIndexQuery`, `Query`, `IndexSelectionStrategy`, `AbstractIndexSelectionStrategy`, `BruteForceIndexSelectionStrategy`, `ApproximateIndexSelectionStrategy`, `ThresholdBasedIndexSelectionStrategy`, `IndexCandidate`, `IndexCandidateGroup`, `IndexSelectionUtil`, `SubqueryIterator` (all whole), plus `QueryUtil` again (whole, **520** lines by this run's count — lesson 20 reported 521, but `processIntersectingRetrievals` sits at **`:463` in both counts**, so the one-line difference is in the reported total, not a moved pin), `StandardJanusGraphTx` (whole, 2277; cited at `:373-376`, `:1589-1592`, `:1596-1631`, `:1640-1698`, `:1713-1755`, `:1807-1836`, `:1838-1857`, `:1868-1891`, `:1894-1942`, `:1956-1990`), `IndexSerializer` (whole, 613; `:414-446`, `:454-462`, `:464-485`), `GraphDatabaseConfiguration` (whole, **2191** — identical to lessons 18 and 19's independent counts), `StandardJanusGraph` (whole, 1489) and `QueryTest` (whole, 1105 — the dedicated test class the vertex-centric compiler does not have). `search_code` located `JanusGraphStep` and `SubqueryIteratorTest`; `issue_read` fetched issue #4931 and its closing PR #4941. Daily Diff: the 2026-11-18 edition (61 stories), deep dive on Postgres `CREATE TEMP TABLE` catalog churn and the instance-wide shared-invalidation ring.
  **Findings.** **THE finding: the graph-centric decision is a weighted set cover with a lexicographic objective, and it lives behind a swappable SPI rather than in the builder.** `GraphCentricQueryBuilder.constructQueryWithoutProfile` (105 lines) calls `indexSelector.selectIndices(...)`, which mutates a caller-owned `Set<Condition> coveredClauses` as an out-parameter and returns `SelectedIndexQuery(JointIndexQuery, isSorted)`. `IndexCandidateGroup.compareTo` ranks groups by **more clauses covered, then fewer indexes, then higher total score** — so a mixed index scoring `5` beats a composite scoring `1004` when it covers one more clause, and the `+1000`-per-clause composite bias only ever breaks ties. Scoring is four constants (`EQUAL_CONDITION_SCORE = 4`, `OTHER_CONDITION_SCORE = 1`, `CARDINALITY_SINGE_SCORE = CARDINALITY_OTHER_SCORE = 1000`) plus two on the approximate strategy (`ORDER_MATCH = 1`, `ALREADY_MATCHED_ADJUSTOR = -1.1`); no statistics anywhere, and `ThresholdBasedIndexSelectionStrategy` carries the in-tree `TODO` asking for selectivity histograms. Default is `threshold-based` with `query.index-select-threshold = 10`, counted against **raw** candidates, so the default admits `2^10 = 1024` `IndexCandidateGroup` allocations per compiled query inside the `OPTIMIZATION` span. A composite index is all-or-nothing (`constructIndexCover` needs an equality at every field position, and an `IN` multiplies out to one `Object[]` tuple per combination → one `KeySliceQuery` each inside one `MultiKeySliceQuery`), and it is excluded outright when the top-level condition is an `Or`. Execution is **stream-one/intersect-the-rest**: `StandardJanusGraphTx.elementProcessorImpl.execute` streams subquery 0 through `SubqueryIterator` under the joint limit and materialises subqueries 1..n−1 **eagerly and unbounded** via `QueryUtil.processIntersectingRetrievals(retrievals, Query.NO_LIMIT)` — which, because `NO_LIMIT == Integer.MAX_VALUE`, makes that function's adaptive retry loop, its `min(16, 2^(n-1))` multiplier and its `subLimit^1.5` growth unreachable from its only caller (candidate 51). The full scan is literally an empty `JointIndexQuery`; `query.force-index` turns it into a `JanusGraphException`. `isSorted` survives three independent gates (`IndexSelectionUtil.indexCoversOrder`, the same check inside `IndexSerializer.getQuery` when building `OrderEntry`s, and the builder's own `takesTheOrder` re-derivation from the serialized query, which demands position 0, equal length and matching field and direction at every position). **Lesson 20's three debts are closed:** the second scoring function is above; `processIntersectingRetrievals` is an *intersect*, not a merge, because graph-centric sub-queries are conjuncts of one `And`; and `JanusGraphStep` proves the strategy layer **cannot** fold a `has()` into a compiled query — it calls `requireOrderValues()` and then `constructQueryWithoutProfile(...)` on a fresh builder, and the only post-compilation rewrite in the tree is `JointIndexQuery.narrowFirst`, used by `withoutResultsAfter`. Six PR candidates logged, **46-51**; next free **52**.

- **20 — 2026-11-18** · [`2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md`](2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md) · **stayed on `a59ae8d8`. The count question lesson 19 handed over is answered: the number of `SliceQuery` shapes a traversal makes per vertex is a product, not a constant. Lesson 08's partitioned-read debt is fully discharged; lesson 19's `MultiSliceQueriesGroupingUtil` hand-off is traced to its single entry condition.** Read the whole vertex-query package at the pinned commit: `BasicVertexCentricQueryBuilder` (whole, 973 lines, numbered locally; cited at `:205-212`, `:325-334`, `:336-342`, `:344-367`, `:376-409`, `:498-517`, `:537-544`, `:546-731`, `:733-774`, `:787-801`, `:816-847`, `:849-929`, `:960-969`), `BaseVertexCentricQueryBuilder`, `VertexCentricQueryBuilder`, `MultiVertexCentricQueryBuilder`, `SimpleVertexQueryProcessor`, `BaseVertexCentricQuery`, `VertexCentricQuery`, `BasicVertexCentricQueryUtil`, `BackendQueryHolder` (all whole), plus `EdgeSerializer` again (whole, 591, numbered; cited at `:430-434`, `:436-552`, `:554-577`, `:579-588` — the line count matches lessons 10 and 11, a free confirmation the pin has not moved), `QueryUtil` (whole, 521, numbered; cited at `:57-71`, `:89-95`), `IDHandler` (whole), `InternalRelationType` (whole), `RelationTypeVertex` (whole). Tests read: `JanusGraphBaseTest` (whole, 764, numbered; `evaluateQuery` at `:589-659`), `JanusGraphTest` (whole, 12,258, numbered; cited at `:2120-2262`, `:4722-4761`, `:4818-4834`, `:5086-5106`, `:5265-5293`), `InMemoryGraphTest` (whole, for its `@Disabled` list). Three directory listings. Four `search_code` queries, of which `VertexCentricQueryTest` returned **0 hits** — there is no dedicated unit test for this compiler at all; its entire coverage lives inside `JanusGraphTest`. One doc page: [Index Performance](https://docs.janusgraph.org/schema/index-management/index-performance/).
  **Findings.** Paragraph compressed 2026-11-21 under the roll-off rule, and the `Verified by lesson 20` block removed in the same run; the full versions are in git history at the 2026-11-18 through 2026-11-20 commits. The load-bearing facts: **the number of `SliceQuery` shapes a vertex-centric traversal makes per vertex is a product, not a constant** — one ordinary traversal compiles to three slices, each its own cache entry on the same edgestore row; **the planner that chooses between a label's vertex-centric indexes is one `double` and no statistics at all**, whose strict `>` makes every tie resolve to the plain label slice; and direction is the low bit of the varint inside the relation type, so a prefix slice cannot select a direction. Lessons 21, 22 and 23 were the stated consumers and all three used it; lessons 25 and 27 still owe the `bestCandidate != null` reachability question and the `setDirectColumnByStartOnlyAllowed` flag.

- **19 — 2026-11-17** · [`2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md`](2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md) · **Part IV opened.** **Findings.** Paragraph compressed 2026-11-20 under the roll-off rule, and the `Verified by lesson 19` block removed in the same run; the full versions are in git history at the 2026-11-17 through 2026-11-19 commits. The load-bearing facts: a read passes through **four** cache tiers (per-transaction `CaffeineVertexCache`, the vertex's own four-entry `queryCache` with its linear `subsumes` scan, the per-graph `ExpirationKCVSCache`, the backend); **what the documentation calls invalidation is a mark, not an eviction, and its release is a `CountDownLatch(5)` counter rather than a clock**, which is why `testGracePeriod` and its sibling are both `@RepeatedIfExceptionsTest` and flaky by issue number; and every user vertex in an ordinary transaction is a `CacheVertex`. Lessons 22 and 27 were the stated consumers — lesson 22 used the tier model to place `SubqueryIterator`'s cache interaction, and lesson 27 still owes the CQL comparison.

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
- [x] 22 — `QueryProcessor`, limits, sub-query merging, and query profiling

### Part V — Gremlin integration (`graphdb/tinkerpop`)

- [x] 23 — The strategy layer: `JanusGraphLocalQueryOptimizerStrategy` and the other optimizer strategies
- [ ] 24 — `JanusGraphStep` and `JanusGraphVertexStep`: folding `has`/`limit`/`order` into storage queries, and multi-query batching

### Part VI — Schema, indexes, and production backends

- [ ] 25 — `ManagementSystem` and the index lifecycle: INSTALLED → REGISTERED → ENABLED, and reindex jobs
- [ ] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30.** Lesson 23 opened Part V at the strategy layer, and the headline answer is that **the
interesting content of that layer is not any single rewrite but the declared dependency graph between
twelve strategies — and two of the twelve declare no ordering of their own.** All three debts lesson 22
handed over are discharged. The re-pin question was **not** re-raised: lesson 23 is mid-Part and the
next review is the Part V/VI boundary, after lesson 24.

**THE fact to carry forward: `JanusGraphStepStrategy`'s position relative to
`JanusGraphLocalQueryOptimizerStrategy` is pinned only transitively, through the two mixed-index
aggregation strategies.** There is no direct edge. The only paths are
`JanusGraphStepStrategy` → `JanusGraphMixedIndexCountStrategy` → `JanusGraphLocalQueryOptimizerStrategy`
and the same through `JanusGraphMixedIndexAggStrategy`, both of which are `applyPrior` on the count/agg
side and an inherited `applyPost` on the abstract base. `withoutStrategies(JanusGraphMixedIndexCountStrategy.class,
JanusGraphMixedIndexAggStrategy.class)` — one line any user can write — leaves the order of the two main
rewriters undeclared. **I did not find a traversal where the order is observable** (the two target
disjoint step types and `HasStepFolder`'s walks break on any unrecognised step, so a `JanusGraphVertexStep`
terminates a walk exactly as a `VertexStep` would), so this is filed as a missing-declaration candidate,
not a defect. Lesson 24 should not restate it as a bug.

**Second fact, and the one lesson 24 needs most: a traversal that ends in an aggregation is compiled
during strategy application, under `QueryProfiler.NO_OP`, and the result is memoized on the step.**
`JanusGraphMixedIndexAggStep`'s constructor calls the public no-arg `janusGraphStep.buildGlobalGraphCentricQuery()`
from inside `AbstractJanusGraphMixedIndexAggStrategy.buildMixedIndexAggStep`. That method memoizes
(`if (globalQuery != null) return globalQuery;`) and TinkerPop has not yet called `setMetrics`, so the
`addNested(constructGraphCentricQuery)` span is created against `NO_OP` and discarded. When the
push-down is then **declined** (`getMixedIndexAggQuery() == null`, the common case with no mixed index),
the original `JanusGraphStep` stays in the traversal carrying the memoized query and `.profile()` shows
**no `constructGraphCentricQuery` span** — while `GraphCentricQuery` and `backend-query` are still there,
because `observeWith` runs later with the live `TP3ProfileWrapper`. **Index selection therefore happens
earlier than the lazy iterator supplier suggests** for `count()`/`min()`/`max()`/`sum()`/`mean()`. The
probe also mutates the step it declined to replace: `hasLocalContainers.put(null, {[]})`. I traced
`toString()`, `hashCode()` and the or()-fallback switch and found no behaviour change, but do not assume
that for a shape I did not try.

**Third, the profiler-attachment ordering, settled and not to be re-derived.** On the normal path, inside
the lambda `setIteratorSupplier` installs: `buildGlobalGraphCentricQuery()` →
`addNested(CONSTRUCT_GRAPH_CENTRIC_QUERY)` + `startTimer` → `disableSmartLimit()` if the end step is a
`CountGlobalStep` → `requireOrderValues()` → `constructQueryWithoutProfile(VERTEX|EDGE)` → `stopTimer` →
`globalQuery.observeWith(addNested(GRAPH_CENTRIC_QUERY))` → `executeGraphCentricQuery` reads
`query.getProfiler()` and wraps `builder.iterables(...)` in a `ProfiledIterator` → `new QueryProcessor`
reads `getProfiler()` at construction. **The holders are observed strictly before anything reads the
profiler**, so lesson 22's "observed too late" worry does not materialise on this path. Only the
memoized path above loses a span.

**Fourth: the TinkerPop side never produces a multi-sub-query `ElementQuery`, sorted or unsorted, so
candidates 55 and 56 stand — and the right PR is a comment, not a deletion.** `JanusGraphStep`'s
iterator supplier has two arms:
`if (!globalQuery.getSubQuery(0).getBackendQuery().isEmpty() || hasLocalContainers.size() != 1)` takes one
disjunctive `GraphCentricQuery` (still `numSubQueries() == 1`); otherwise it builds **N separate**
`GraphCentricQuery` objects, one per folded `or()` branch, each with one sub-query, and merges them **in
the step** with `MultiDistinctUnorderedIterator<>(lowLimit, highLimit, responses)` or
`MultiDistinctOrderedIterator<>(lowLimit, highLimit, responses, comparator)`. So the de-duplicating
multi-query merge JanusGraph actually needs exists one layer **above** `QueryProcessor`, where it can
also apply the step's low limit — which is why the generic `concat` + `seenResults` arm inside
`QueryProcessor` is unreachable. Note the fallback condition counts **`or()` steps, not branches**
(`hasLocalContainers.size()`), so with two folded `or()`s the per-branch split never happens. The
ordered merge's soundness rests on an `assert`
(`queries.stream().allMatch(q -> q.getOrder().isEmpty() || q.getOrder().equals(order))`), disabled in
production — carry that to lesson 28.

**Fifth: the `AND-query`/`OR-query` flattening is not a tested contract.** `search_code` for
`AND_QUERY OR_QUERY TP3ProfileWrapper assert metrics` returned **0 results**, and for
`"backend-query" profile test assertion` also **0 results**, repository-wide. Treat that as suggestive
rather than proof — the index demonstrably does not return every file containing a string (lesson 22 hit
the same limit on `BackendLimitAware`) — but combined with lesson 22's reading of both profiler
implementations it means **nothing in the repository asserts profile-tree shape**. Consequence for any
new-span proposal: a span added as an only child of an `AND-query`/`OR-query` group is silently
reparented in a TinkerPop profile and nested in a `SimpleQueryProfiler` tree. **Candidate 60 must be
filed before any new-span patch**, and this is now recorded so lesson 24 does not re-open the question.

**Sixth, the fold rules, which lesson 24 will need line by line and should cite rather than re-read.**
`local()` is the only construct that gets an order or a range to the storage layer: pass 1
(`applyJanusGraphVertexSteps`, `:99-117`) folds a `has()` only when `isEdgeReturnStep`, folds **no** order
or range (comment at `:106`: *"We cannot fold in orders or ranges since they are not local"*), and folds
only a *high* limit from a following `RangeGlobalStep` **without removing it** (`:110-114`); pass 5
(`inspectLocalTraversals`, `:163-197`) calls `foldInHasContainer`, `foldInOrder(..., false)` **and**
`foldInRange`, sets `setBatchSize(txVertexCacheSize)`, and `unfoldLocalTraversal` (`:225-235`) deletes the
`LocalStep` entirely when its child has collapsed to one step. A **vertex-centric query folds at most one
order key** (`HasStepFolder:423-425`); a graph-centric one folds any number. `foldInRange` (`:484-502`)
folds a *low* limit only for a `JanusGraphStep`, and removes the `RangeGlobalStep` only for a
`JanusGraphStep` or when `getLowRange() == 0`. `JanusGraphStepStrategy` checks `ArrayUtils.isEmpty(getIds())`
**twice** — before and after `foldInIds` — and skips order/range folding if ids were folded, so TinkerPop
handles them.

**Seventh, `or()` folding, verified against the test table.** `foldInHasContainer`'s `OrStep` arm
(`HasStepFolder:185`) is guarded by `janusgraphStep instanceof JanusGraphStep`, so `or()` is never folded
for a vertex-centric step. Per branch: `validFoldInOrBranch` (`:247-268`) is **all-or-nothing across the
whole `or()`** — one unrecognised step abandons every branch — and requires at least one condition;
`branchPassesElements` (`:270-281`) drops a branch containing a `RangeGlobalStep` with `lowRange > 0` or
`highRange == 0`, because a branch runs **per element**; `localFoldInHasContainer` (`:225-245`) converts a
`has(key)` exists step and **every key of an `order().by(key)`** into `HasContainer(key, P.neq(null))`.
`JanusGraphStepStrategyTest.java:342-380` (blob-SHA verified this run) is the specification; cite it
rather than re-deriving.

- **The mode question is closed. Do not reopen it unprompted.** Entries from lesson 05 on write in
  `lesson` mode without re-litigating the decision, without a notice at the top, and without nagging.
  If Aryant asks for reading sessions back, flip `mode:` and the entries resume the other shape from
  the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Lesson 24 (`JanusGraphStep`/`JanusGraphVertexStep`, multi-query batching) has a fully specified
  input and owes four things.** (a) The **batch-size asymmetry lesson 23 could not settle**:
  `inspectLocalTraversals` calls `setBatchSize(txVertexCacheSize)` on the `JanusGraphVertexStep` it
  creates (`:170`) and `applyJanusGraphVertexSteps` **never calls `setBatchSize` at all**, so lesson 24
  must say whether a top-level `out()` gets `txVertexCacheSize` by another route — most likely
  `JanusGraphMultiQueryStrategy`, which runs after and attaches clients to every `MultiQueriable` — or
  keeps a different default than `local(out())`. **Do not file anything on this until `JanusGraphVertexStep`
  and `JanusGraphMultiQueryStep` are actually read.** (b) The **multi-query annotations**:
  `QueryProfiler`'s `MULTIQUERY_ANNOTATION = "multi"`, `MULTIPREFETCH_ANNOTATION = "multiPreFetch"`,
  `NUMVERTICES_ANNOTATION = "vertices"` and `PARTITIONED_VERTEX_ANNOTATION = "partitioned"` are declared
  and set nowhere lessons 22 or 23 read; lesson 24 owns whoever writes them and should say which a real
  `.profile()` of a batched traversal shows. (c) `JanusGraphMultiQueryStrategy`'s own shape, for which
  lesson 23 established only the frame: `applyPrior` is all three of
  `{JanusGraphLocalQueryOptimizerStrategy, JanusGraphHasStepStrategy, JanusGraphStepStrategy}`, it reads
  `useMultiQuery()`, `limitedBatch()`, `limitedBatchSize()` and `repeatStepMode()`, inserts
  `JanusGraphMultiQueryStep` (plus `JanusGraphNoOpBarrierVertexOnlyStep` when `limitedBatch`) at positions
  from `JanusGraphTraversalUtil.getLocalMultiQueryPositionForStep`, replaces exact-class `RepeatStep`s via
  `JanusGraphRepeatStep.replace`, and pairs with `JanusGraphUnusedMultiQueryRemovalStrategy`
  (`applyPrior = {JanusGraphMultiQueryStrategy}`), which deletes any `JanusGraphMultiQueryStep` with all
  three client lists empty **and** its generated barrier — the insert-then-remove design is deliberate and
  stated in `insertMultiQuerySteps`' javadoc. (d) Lesson 20's `executeMultiQuery` /
  `executeMultiSliceMultiQuery` switch is the other half of the batching story. Start from
  `JanusGraphVertexStep` and `JanusGraphMultiQueryStep`.
- **Lesson 24 is the Part V/VI boundary. The re-pin review is DUE after it, and lesson 24 must perform
  it.** Lessons 23 correctly did not re-raise it. The procedure is unchanged: re-run `list_commits` on
  `JanusGraph/janusgraph` `master` rather than assuming either answer, and state the result.
- **Lesson 25 (`ManagementSystem`) inherits three, unchanged from lesson 21.** (a) whether any shipped
  relation type has `isInvisibleType() == true` without being a system type; (b)
  `mgmt.buildEdgeIndex`/`buildPropertyIndex` with the `TypeDefinitionCategory.RELATIONTYPE_INDEX` schema
  edge; (c) reads skip a **composite** index unless `index.getStatus() == SchemaStatus.ENABLED`, with no
  base-type exemption, while a **mixed** index is filtered **per `ParameterIndexField` status** inside
  `coversAll` — lesson 25 owns the state machine that moves those statuses and should state which of the
  two a reindex relies on. Still starts at `ManagementSystem.createCompositeIndex:848-850`. **New from
  lesson 23:** `AdjacentVertexHasUniquePropertyOptimizerStrategy` decides validity with
  `IndexSelectionUtil.existsMatchingIndex(...)` requiring a **composite, `Cardinality.SINGLE`** index and
  is gated on `GraphDatabaseConfiguration#optimizerBackendAccess()` — the one strategy allowed to touch
  the backend, and it does so by running a side traversal during strategy application. Lesson 25 should
  say what that strategy does with an index whose status is not `ENABLED`.
- **Lesson 26 (`ElasticSearchIndex`) inherits three feature flags, unchanged:**
  `supportNotQueryNormalForm()`, `supportsOrderingListProperty()` and `supportsExistsQuery()` — and
  lesson 23 adds the third's caller: `HasStepFolder.validJanusGraphOrder` (`:102-143`) consults
  `graph.getIndexSerializer().allMixedIndexBackendSupportsOrderingListProperty()` to decide whether a
  **vertex** order on a non-`SINGLE` key is foldable at all, so that flag changes query *compilation*, not
  just execution. Also carried: commit `b7c14c53`'s `index.[X].solr.max-page-size` (default 10,000) and
  the page-doubling read in `SolrIndex` — compare the two doubling loops and say whether
  `ElasticSearchIndex` has an equivalent.
- **Lesson 27 (CQL) inherits two, unchanged.** The `setDirectColumnByStartOnlyAllowed` flag from lesson
  20, and the feature-builder classification lesson 18 owed: whether `CQLStoreManager` and
  `HBaseStoreManager` set `.locking(...)` and so whether `ConsistentKeyLocker` runs on them. Also carried
  from lesson 22: the per-sub-query million-element pre-sort budget and the no-cursor re-read make an
  unfitted ordered query on CQL the worst case in the tree — put a number on the re-read cost against a
  real backend rather than the in-memory store.
- **Lesson 28 (transaction log, CDC, stale-index repair) inherits the deletion semantics** from lesson 22
  — `result.isRemoved()` dropped **eagerly** in `PreSortingIterator`, everything else the transaction
  deleted filtered **lazily as reached**, deliberately — **plus, new from lesson 23, the multi-query
  merge**: when the edgestore and a mixed index disagree, the thing that sees both is
  `MultiDistinctOrderedIterator`, whose cross-query ordering invariant is an `assert`.
- **Open and unresolved, deliberately not filled in.** (a) Lesson 20's
  `BasicVertexCentricQueryBuilder:711` `Preconditions.checkArgument(bestCandidate != null, …)`
  reachability question is still open. (b) Whether any production path passes a real limit to
  `QueryUtil.processIntersectingRetrievals` (candidate 51 is a reachability claim, not a confirmed
  defect). (c) Whether `BackendQueryHolder`'s duplicated `isFitted` in `hashCode` (candidate 54) can
  degrade anything. (d) `IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel
  colliding with `IndexCandidate`'s default score, carried from lesson 21 inside candidate 50. (e) **New:
  the §2.2 ordering question — is the `JanusGraphStepStrategy` / `JanusGraphLocalQueryOptimizerStrategy`
  order observable by any traversal?** Hands-On B of lesson 23 is the experiment; nobody has run it.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, **23 ~59** (digest ~16). A
  large share of each is permalink URLs, which are not prose. The old 5-9 KB budget was a
  `contributor-reading` constraint and does not apply.
  **Mechanical notes, carried and confirmed this run.** A single `create_or_update_file` call carrying
  ~100 KB succeeds. `get_file_contents` on a file whose *doubled* output exceeds roughly 40 KB returns an
  over-limit error that saves the JSON to a local file — the cheap way to read a large file, and it costs
  nothing in context. The correct extraction from that payload is
  `json.loads(longest_text)["content"][*]["resource"]["text"]`, **not** the plain `text` value (which has a
  `[Resource from github at repo://…]` banner glued onto line 1) and **not** the nested JSON string itself.
  Verified this run by blob SHA on `TRACK.md`, `README.md`, `tracks/janusgraph-registers.md`,
  `HasStepFolder.java`, `StandardJanusGraph.java` and `JanusGraphStepStrategyTest.java`. **A file under
  roughly 20 KB comes back inline with no line numbers and must be typed out to local disk to be numbered
  at all** — lesson 23 did that for `JanusGraphLocalQueryOptimizerStrategy.java` (13,515 bytes) and the
  blob SHA matched `01de0bed` exactly on the first attempt. **Do the blob-SHA check on every file read to
  disk and on every push over ~50 KB.** Files read inline and not transcribed are cited by member name
  only, per lesson 20's split.
  **New this run, and worth keeping: delegating a bulk survey works, but it changes the citation grade.**
  Ten strategy files were read at the pinned commit by a delegated reader, which returned
  `applyPrior()`/`applyPost()` initialisers and `apply()` bodies verbatim. The paths and blob SHAs came
  from a directory listing fetched directly, so the permalinks are sound, but the text never passed
  through a local transcription. Lesson 23 cited those by member name and **said so in the entry**. Keep
  that discipline: a delegated read is a legitimate source and a weaker one, and the entry must say which
  claims rest on it.
- **Method note; lesson 23 is the eleventh entry where simulation was not needed.** Lessons 10-12
  re-implemented the pinned encoding functions and ran them; lessons 13-23 computed from the read sources
  instead and said so. This run's derived claims — the transitive-only ordering edge, the memoized
  `NO_OP` compile, the two-arm `or()` execution, and the absence of any profile-shape test — are each
  labelled where they appear and each paired with an exercise that runs the real code: Hands-On A runs
  `JanusGraphStepStrategyTest#doTest` and has the reader break `branchPassesElements` to prove a specific
  row fails, B prints the live sorted strategy list with each strategy's `applyPrior`/`applyPost` and
  re-prints it with the two aggregation strategies removed, C reproduces the missing
  `constructGraphCentricQuery` span by profiling the same query with and without a trailing `count()`,
  and D shows `unfoldLocalTraversal` deleting a `LocalStep`. **Keep the rule that produced them:** compute
  or re-implement rather than assert, say which, and always pair it with an exercise that runs the real
  code. **Render every Mermaid diagram with `mmdc` before pushing** — this run's three diagrams (two in
  the lesson, one in the digest) were extracted from the finished Markdown and rendered, all clean on the
  first attempt, with the lesson-19 rule observed throughout (quote every node label containing
  parentheses, commas, `->`, `|` or angle brackets, use HTML entities such as `&lt;` for comparisons, and
  never escape a quote inside a label — drop the inner quotes instead).
- **Daily Diff procedure — a NEW edition was available and the fallback was NOT needed, ending a
  two-run streak.** At ~03:40 UTC the front page served **2026-11-19 / 61 stories**, `/archive/` reported
  *145 Editions* with its `Latest >` link pointing at **`/2026-11-20/`** and calendar links for both
  11-19 (*"61 stories"*) and 11-20 (*"50 stories"*), **2026-11-21 unlinked**, and the dated URLs for both
  11-21 and 11-20 returned `client_error` while **`/2026-11-19/` returned HTTP 200**. So the archive is
  now ahead of what the site will serve: it advertises a 50-story 11-20 edition whose URL 404s. The
  maximum *fetchable* edition was 11-19, the newest already digested was 11-18 (lesson 21), so 11-19 was
  taken whole. **Next run must re-probe `/2026-11-20/`** — if it resolves, that edition is uncovered and
  is the one to digest; if it still 404s while the archive still lists it, record the disagreement as
  standing. The procedure is otherwise unchanged: (a) read the front page and the archive and take the
  maximum; (b) probe the dated URL for today and the previous two days; (c) check this session log for
  the newest edition already covered; (d) only then the uncovered-item fallback. Running tally of spent
  fallback items: 2026-11-06 has two (Infino ids, Postgres `SKIP LOCKED`), 2026-11-18 has two (the
  temp-table item, and #15 Galera `pc.weight`). **Constraint still in force: there is no `WebFetch` tool
  in this environment** — pages must be fetched through the `web-fetch` subagent, which returns a report
  flagging which parts are verbatim and which are an extracted summary, and **the summary portion is not
  reliable enough to quote**. This run's fetch of the front page and of `/2026-11-19/` was truncated at
  item 22, so the digest named only items 1-22 and said so. Edition sizes remain volatile: 22, 50, 63,
  58, 66, 47, 39, 49, 37, 57, 61, 61 across 11-08 to 11-19.

### Verified by lesson 23, for lessons 24, 25, 26, 27 and 28 to cite rather than re-derive

- **The registry, verbatim, at [`StandardJanusGraph.java:165-185`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/StandardJanusGraph.java#L165-L185)**
  (blob-SHA verified, 80,408 bytes = `f650acb2`). One `static` block clones
  `TraversalStrategies.GlobalCache.getStrategies(Graph.class)` and `addStrategies` **twelve** instances, in
  source order: `AdjacentVertexFilterOptimizerStrategy`, `AdjacentVertexHasIdOptimizerStrategy`,
  `AdjacentVertexIsOptimizerStrategy`, `AdjacentVertexHasUniquePropertyOptimizerStrategy`,
  `JanusGraphLocalQueryOptimizerStrategy`, `JanusGraphHasStepStrategy`, `JanusGraphMultiQueryStrategy`,
  `JanusGraphUnusedMultiQueryRemovalStrategy`, `JanusGraphMixedIndexAggStrategy`,
  `JanusGraphMixedIndexCountStrategy`, `JanusGraphStepStrategy`, `JanusGraphIoRegistrationStrategy`. The
  same object is registered for **both** `StandardJanusGraph.class` and `StandardJanusGraphTx.class`
  (`:183-184`). **All twelve are `TraversalStrategy.ProviderOptimizationStrategy`** — none is a
  `DecorationStrategy` — so TinkerPop's category sort puts the whole JanusGraph layer in one block and
  every ordering question is an edge *inside* it. `ReadOnlyStrategy` is added separately and
  per-traversal, at `:377`. **OLAP gets one strategy, not twelve:** `FulgoraUtil`'s
  `FULGORA_STRATEGIES` is `…getStrategies(Graph.class).clone().addStrategies(JanusGraphLocalQueryOptimizerStrategy.instance())`.
- **The complete `applyPrior`/`applyPost` graph.** `JanusGraphLocalQueryOptimizerStrategy.applyPrior()` =
  `Collections.singleton(AdjacentVertexFilterOptimizerStrategy.class)` (`:57`, `:238-239`), no `applyPost`.
  `AdjacentVertexOptimizerStrategy` (abstract base of HasId, Is and HasUniqueProperty) `applyPost()` =
  `singleton(JanusGraphLocalQueryOptimizerStrategy)`; `AdjacentVertexHasUniquePropertyOptimizerStrategy`
  **overrides** it to `new HashSet<>(Arrays.asList(JanusGraphStepStrategy.class, JanusGraphLocalQueryOptimizerStrategy.class))`.
  `AbstractJanusGraphMixedIndexAggStrategy.applyPost()` = `singleton(JanusGraphLocalQueryOptimizerStrategy)`;
  both `JanusGraphMixedIndexCountStrategy` and `JanusGraphMixedIndexAggStrategy` add
  `applyPrior() = singleton(JanusGraphStepStrategy)`. `JanusGraphHasStepStrategy.applyPrior()` =
  `singleton(JanusGraphLocalQueryOptimizerStrategy)`. `JanusGraphMultiQueryStrategy.applyPrior()` =
  `new HashSet<>(Arrays.asList(JanusGraphLocalQueryOptimizerStrategy, JanusGraphHasStepStrategy, JanusGraphStepStrategy))`.
  `JanusGraphUnusedMultiQueryRemovalStrategy.applyPrior()` = `new HashSet<>(Arrays.asList(JanusGraphMultiQueryStrategy))`.
  **`AdjacentVertexFilterOptimizerStrategy` and `JanusGraphIoRegistrationStrategy` declare neither** — the
  first is nevertheless ordered because `JanusGraphLocalQueryOptimizerStrategy` names it; the second has no
  edge in either direction. (The ten files other than `JanusGraphLocalQueryOptimizerStrategy`,
  `JanusGraphStepStrategy`, `JanusGraphMixedIndexCountStrategy` and
  `AbstractJanusGraphMixedIndexAggStrategy` were read by a **delegated reader** this run, which returned
  these initialisers verbatim; the paths and blob SHAs are from a directly fetched directory listing.)
- **Consequence, and the single most useful fact in Part V: the `JanusGraphStepStrategy` ↔
  `JanusGraphLocalQueryOptimizerStrategy` order is transitive-only**, via
  `JanusGraphStepStrategy` → `JanusGraphMixedIndexCountStrategy`/`JanusGraphMixedIndexAggStrategy` →
  `JanusGraphLocalQueryOptimizerStrategy`. No traversal was found where the order is observable; filed as a
  missing declaration, not a defect.
- **`JanusGraphLocalQueryOptimizerStrategy` is five passes in a fixed order** (`:92-96`):
  `applyJanusGraphVertexSteps` → `applyJanusGraphPropertiesSteps` → `applyJanusGraphLabelSteps` →
  `applyJanusGraphDropSteps` → `inspectLocalTraversals`. Guards: `!traversal.getGraph().isPresent()`
  (`:64-65`) and `janusGraph == null` (`:68-70`); **no `onGraphComputer` guard**, because this is the one
  strategy OLAP installs. Config comes from `TransactionConfiguration` when
  `JanusGraphTraversalUtil.getJanusGraphTx(traversal)` is present (`:78-84`) and from
  `GraphDatabaseConfiguration` otherwise (`:85-90`) — four values, eight accessor names, so **a
  per-transaction mode override only affects traversals started from the transaction**.
- **Pass 1 vs pass 5, the whole point of the strategy.** Pass 1 (`:99-117`): every `VertexStep` becomes a
  `JanusGraphVertexStep`; `foldInHasContainer` **only** if `isEdgeReturnStep` (`:105`); the comment at
  `:106` is *"We cannot fold in orders or ranges since they are not local"*; the only fold is a **high**
  limit from a following `RangeGlobalStep`, `setLimit(0, mergeHighLimits(limit, getHighLimit()))`, and the
  range step is **not removed** (`:110-114`); `:109` is an `assert`. Pass 5 (`:163-197`), for a `LocalStep`
  whose child starts with a `VertexStep` or `PropertiesStep`: `setBatchSize(txVertexCacheSize)` (`:170`,
  `:185`), `foldInHasContainer`, **`foldInOrder(..., isVertexOrder=false)`** (`:175`, `:191`) and
  **`foldInRange`** (`:177`, `:193`), then `unfoldLocalTraversal` (`:225-235`) which replaces the entire
  `LocalStep` with the single step in the **outer** traversal when the child collapsed to one step (both
  preconditions are `assert`s, `:228` and `:231`). **Pass 1 never calls `setBatchSize`** — open question
  for lesson 24.
- **Pass 2 (`:118-161`) matches three unrelated TinkerPop classes in one predicate** (`:119`):
  `PropertiesStep` → `JanusGraphPropertiesStep` (folds `has()` if `getReturnType().forProperties()`,
  `:134-137`), `PropertyMapStep` → `JanusGraphPropertyMapStep`, `ElementMapStep` →
  `JanusGraphElementMapStep`; idempotence guard `if (originalStep instanceof MultiQueriable) return;`
  (`:120-122`); a four-valued `MultiQueryPropertiesStrategyMode` collapses to two booleans
  (`ALL_PROPERTIES.equals(...)` and `!NONE.equals(...)`); `setBatchSize(txVertexCacheSize)` at `:158`.
  **The property prefetch batch is the transaction vertex cache size**, not a setting of its own.
  Passes 3 and 4 (`:199-223`) are structurally identical twelve-line methods that only substitute
  `JanusGraphLabelStep`/`JanusGraphDropStep` and fold nothing; both early-return when their mode is `NONE`,
  so their only purpose is giving the multi-query layer a `MultiQueriable`.
- **`HasStepFolder` is an interface with 8 abstract and 14 `static` methods** (blob-SHA verified, 27,466
  bytes = `0232fc9b`, 532 lines). `foldInIds` `:145-178`, `foldInHasContainer` `:181-214`,
  `localFoldInHasContainer` `:225-245`, `validFoldInOrBranch` `:247-268`, `branchPassesElements` `:270-281`,
  `keysWithValues` `:289-…`, `validFoldInHasContainer` `:327-343`, `isExistsStep` `:351-363`,
  `foldInHasFilter` `:367-378`, `foldInOrder` `:380-434`, `splitAndP` `:437-445`, `foldInRange` `:484-502`,
  `foldableHasContainerNoLimit` `:507-531`, `validJanusGraphOrder` `:102-143`.
- **`or()` folding, exactly.** The `OrStep` arm (`:185`) is guarded by
  `janusgraphStep instanceof JanusGraphStep`, so **a vertex-centric step never folds an `or()`**.
  `validFoldInOrBranch` is **all-or-nothing across the whole `or()`** — one unrecognised step `return`s and
  abandons every branch — and requires `hasCondition` true (at least one `has()` or exists step); allowed
  steps are `HasContainerHolder` (with `validJanusGraphHas`), exists, `OrderGlobalStep` with non-null
  `keysWithValues`, `RangeGlobalStep`, `IdentityStep`, `NoOpBarrierStep`. `branchPassesElements` rejects a
  branch whose `RangeGlobalStep` has `getLowRange() > 0 || getHighRange() == 0`, because **a branch is
  evaluated per element**. `localFoldInHasContainer` converts a `has(key)` exists step into
  `HasContainer(key, P.neq(null))` and **every key returned by `keysWithValues` of an `order()`** into the
  same, because TinkerPop drops an element whose `by()` yields nothing. `addLocalHasContainersOfNoBranch`
  (`:195-197`) records an `or()` that no element can pass as a parent id with an **empty** branch set.
  `isExistsStep` recognises TinkerPop's `has(key)` as `TraversalFilterStep(PropertiesStep)` with exactly
  one key; `foldInHasFilter` rewrites it in place into a real `HasStep` with `P.neq(null)` and returns it so
  the loop re-examines it. `validFoldInHasContainer` is a three-valued `Boolean` accumulator whose
  `defaultValue` is `true` from `foldInHasContainer` (`:200`) and `foldableHasContainerNoLimit`'s
  `HasContainerHolder` arm (`:519`), `false` from that method's `OrStep` arm (`:513`).
- **`foldInOrder` collects consecutive `OrderGlobalStep`s and iterates them BACKWARDS**, in-source reason:
  *"TinkerPop sorts by each order() in turn, keeping the order of ties … so the elements come in the order
  of the last one, and where that ties, of those before it."* A key already present is skipped. The gate at
  `:423-425` is `if (orders.isEmpty() || orders.size() > 1 && !(janusgraphStep instanceof JanusGraphStep))
  return;` — **a vertex-centric query folds at most one order key; a graph-centric one folds any number.**
  `validJanusGraphOrder` (`:102-143`) rejects: a `ValueTraversal` with a non-null `getBypassTraversal()`
  (a `ProductiveByStrategy` rewrite), `Order.shuffle`, a key with no `PropertyKey`, a non-`Comparable`
  `dataType()`, and a **vertex** order on a non-`Cardinality.SINGLE` key unless
  `graph.getIndexSerializer().allMixedIndexBackendSupportsOrderingListProperty()` — so that mixed-index
  feature flag changes query **compilation**, not just execution (lesson 26).
- **`foldInRange` (`:484-502`) is asymmetric.** `low` is folded **only** when
  `janusgraphStep instanceof JanusGraphStep`; otherwise it is hard `0`. The `RangeGlobalStep` is removed
  only when the folder is a `JanusGraphStep` **or** `range.getLowRange() == 0` — so a vertex-centric step
  with a non-zero offset keeps the range step and folds only the high bound.
- **`JanusGraphStepStrategy` checks the ids twice.** `apply` is one `getStepsOfClass(GraphStep.class, …)`
  loop. If `ArrayUtils.isEmpty(getIds())`: create `JanusGraphStep`, `foldInIds`, `foldInHasContainer`, then
  **re-check** `ArrayUtils.isEmpty(janusGraphStep.getIds())` before `foldInOrder(..., returnsVertex())` and
  `foldInRange` — in-source: *"if there are any ids folded … we shouldn't fold in order and range steps so
  that they can be handled by TinkerPop."* The `else` arm creates **no** `JanusGraphStep`: it unwraps
  `Element` arguments to `id()` and installs an iterator supplier over `graph.vertices(...)`/`edges(...)`,
  *"Make sure that any provided 'start' elements are instantiated in the current transaction"*. It also
  early-returns on `TraversalHelper.onGraphComputer(traversal)`.
- **`JanusGraphStep`'s execution is lazy and has two arms.** Inside `setIteratorSupplier`:
  `buildGlobalGraphCentricQuery()` → `buildGraphCentricQuery(query, queryProfiler)` which does
  `addNested(CONSTRUCT_GRAPH_CENTRIC_QUERY)` + `startTimer` → `disableSmartLimit()` **iff**
  `traversal.getEndStep() instanceof CountGlobalStep` → `requireOrderValues()` →
  `constructQueryWithoutProfile(VERTEX|EDGE)` → `stopTimer`; **then**
  `globalQuery.observeWith(queryProfiler.addNested(GRAPH_CENTRIC_QUERY))`; **then**
  `executeGraphCentricQuery` reads `query.getProfiler()` and wraps `builder.iterables(query, graphClass)`
  in a `ProfiledIterator`, which is where `new QueryProcessor` happens. **Holders are observed before
  anything reads the profiler.** The arm switch is
  `if (!globalQuery.getSubQuery(0).getBackendQuery().isEmpty() || hasLocalContainers.size() != 1)`:
  one disjunctive query (built with `query.or(localQueries)`), else **N** `GraphCentricQuery` objects one
  per `or()` branch, merged by `MultiDistinctUnorderedIterator<>(lowLimit, highLimit, responses)` or
  `MultiDistinctOrderedIterator<>(lowLimit, highLimit, responses, comparator)`. `hasLocalContainers.size()`
  counts **`or()` steps, not branches**. `setMetrics(MutableMetrics)` is the only thing that replaces
  `queryProfiler = QueryProfiler.NO_OP` with `new TP3ProfileWrapper(metrics)`.
- **`buildGlobalGraphCentricQuery()` memoizes, and `JanusGraphMixedIndexAggStep`'s constructor calls it at
  strategy time.** `if (globalQuery != null) return globalQuery;`, and when `hasLocalContainers.isEmpty()`
  it inserts `hasLocalContainers.put(null, {[]})` as a side effect.
  `AbstractJanusGraphMixedIndexAggStrategy.buildMixedIndexAggStep` constructs
  `new JanusGraphMixedIndexAggStep<>(originalGraphStep, traversal, agg)` — whose constructor calls that
  method — and **discards the new step** when `getMixedIndexAggQuery() == null`, leaving the original
  `JanusGraphStep` with a query compiled under `NO_OP`. `getAggregation` returns non-null only for a
  trailing `CountGlobalStep` (`JanusGraphMixedIndexCountStrategy`) or a single-key `PropertiesStep` followed
  by `Max`/`Min`/`Mean`/`SumGlobalStep` (`JanusGraphMixedIndexAggStrategy`), both skipping
  `IdentityStep`/`NoOpBarrierStep`, and both require `originalGraphStep.isStartStep()`.
- **Nothing in the repository asserts profile-tree shape.** `search_code` for
  `AND_QUERY OR_QUERY TP3ProfileWrapper assert metrics` → 0 results; `"backend-query" profile test
  assertion` → 0 results. Suggestive, not proof (the index does not reliably return every file containing a
  string), but enough that **candidate 60 must be filed before any new-span patch**.
- **The test surface for this layer.** `janusgraph-test/.../optimize/` holds ten files:
  `AdjacentVertexFilterOptimizerStrategyTest` (5,194 bytes), `AdjacentVertexHasIdOptimizerStrategyTest`
  (4,146), `AdjacentVertexHasUniquePropertyOptimizerStrategyTest` (5,476),
  `AdjacentVertexIsOptimizerStrategyTest` (2,729), `JanusGraphIoRegistrationStrategyTest` (1,482),
  `JanusGraphLocalQueryOptimizerStrategyTest` (9,045), `JanusGraphMultiQueryStepTest` (4,705),
  `JanusGraphMultiQueryStrategyTest` (11,886), `JanusGraphStepStrategyTest` (46,982) and the abstract
  `OptimizerStrategyTest` (6,085). **They all run on the in-memory backend** —
  `OptimizerStrategyTest.open()` is `graph = StorageSetup.getInMemoryGraph()` — so
  `mvn -pl janusgraph-test -Dtest=<Class> test` needs nothing external. `makeSampleGraph()` builds composite
  indexes `byId`, `byUniqueId` (unique), `byWeight`, `byIdWeight`, edge indexes `byWeightDesc`/`byWeightAsc`
  on `knows`, a `names` LIST key with a `namesByWeight` property index, then `numV = 100` vertices and
  `superV = 10` super-vertices each with 100 `knows` edges and 100 `names` properties — **no mixed index**,
  which is exactly the condition that makes the aggregation push-down decline.
  `JanusGraphStepStrategyTest` (blob-SHA verified, 46,982 bytes = `abd23a81`, 546 lines) is the
  specification: `doTest` (`:111-122`) builds a `DefaultTraversalStrategies`, adds
  `JanusGraphStepStrategy.instance()` plus the row's extra strategies, calls `applyStrategies()` and
  asserts `assertEquals(optimized, original)`; `doMultiQueryTest` (`:124-142`) compares `toString()`
  instead, *"if the traversal has any steps of type SideEffectStep that fails"*. The `or()` rows are
  `:342-380`. Three non-parameterized tests: `shouldTriggerHasContainerSplit`,
  `shouldTriggerLocalHasContainerSplit`, `shouldTriggerLocalHasContainerConvert` (`:144-170`) — they pin
  `splitAndP`'s `AndP` expansion, 2 and 4 and 2 containers respectively. The only method name verified
  verbatim in `JanusGraphLocalQueryOptimizerStrategyTest` is `testDefaultConfiguration`.

### Verified by lesson 22, for lessons 23, 24, 25, 26, 27 and 28 to cite rather than re-derive

- **The stack, outward from the backend, built at `QueryProcessor:101-107`.**
  `executor.execute(query, backendQuery, executionInfo, profiler)` → `LimitAdjustingIterator` (or
  `PreSortingIterator`) → `filter !allNewSet.contains` (graph-centric only, `:103-105`) →
  `getFilterIterator(hasDeletions, !subquery.isFitted())` (`:107`) → the merge fold (`:109-111`) →
  `ResultSetIterator` (`:69-74`). **The in-memory filter is below the element count and above the backend
  limit.** `getUnfoldedIterator` runs **eagerly inside `iterator()`**: `hasDeletions` and `getNew` are
  both called, and in the sorted branch the new elements are drained to a list and sorted, before any
  caller asks for an element.
- **Three limits, and only the middle one moves.** `query.getLimit()` counts **elements emitted** and is
  enforced by `ResultSetIterator`, which reads one element ahead and calls `close()` the instant the
  count is reached — which is what releases backend resources such as an Elasticsearch scroll context.
  `backendQueryHolder.getBackendQuery().getLimit()` counts **raw backend results** and is the one that
  **doubles**. `MAX_SORT_ITERATION = 1_000_000` (`:56`) bounds a pre-sort and **throws** rather than
  truncating. `ResultSetIterator.remove()` is the only `remove()` in the stack that is not an
  `UnsupportedOperationException`; it calls `current.remove()` on the previously returned element.
- **`getFilterIterator` is absent when there is nothing to filter** (`:239-245`): `if (filterDeletions ||
  filterMatches)` wraps, `else return iterator;`. A fitted sub-query in a transaction with no deletions
  gets no wrapper at all, so a typed fitted vertex-centric `limit(10)` is three objects:
  `ResultSetIterator` → `LimitAdjustingIterator` → entry iterator.
- **The branch map, from four one-line bodies.** `VertexCentricQuery.isSorted()` → **`true`**
  unconditionally; `getSortOrder()` → `new RelationComparator(vertex, getOrders())`, **not** an
  `OrderList`; `numSubQueries()` → `queries.size()`; `hasDuplicateResults()` → `false` (comment: *"We
  wanna count self-loops twice"*). `GraphCentricQuery.isSorted()` → `!orders.isEmpty()`;
  `numSubQueries()` → literal **`1`**; `hasDuplicateResults()` → `false`; `isEmpty()` → `getLimit() <= 0`,
  identical to `BaseVertexCentricQuery.isEmpty()`. `search_code` for `implements ElementQuery` returns
  **exactly these two classes**.
- **Three consequences of that map, all dead code in the shipped tree.** (1) The unsorted branch's
  `iterators.size() > 1` arm — `CloseableIteratorUtils.concat(iterators)` plus the `seenResults`
  de-duplication block, `:139-150` — is **unreachable**, because the only unsorted family has
  `numSubQueries() == 1`; every unsorted query falls to `else iterator = iterators.get(0);` (`:151`).
  (2) `ResultMergeSortIterator`'s `filterDuplicates` is always `false`, so its
  `do { … } while (next != null && filterDuplicates && comp.compare(current, next) == 0)` always runs one
  iteration. (3) `lastFirstOrderValueWithinLimit` returns `null` for every vertex-centric query, via the
  `!(query.getSortOrder() instanceof OrderList)` guard. The class javadoc (`:48`) still documents (2) as
  live.
- **The merge fold is binary, right-leaning, and left-biased.** The loop runs **backwards**
  (`for (int i = numSubQueries() - 1; i >= 0; i--)`, `:93`) so that sub-query 0 ends up the outermost
  **left** operand. `ResultMergeSortIterator` holds one lookahead per side (`nextFirst`, `nextSecond`) and
  breaks ties with **`if (c <= 0)`** — left wins — so equal-ordering results emerge in sub-query order,
  deterministically. There is no *k*-way heap: *n* sub-queries cost up to *n−1* comparator calls per
  emitted element. `Preconditions.checkArgument(iterator != null)` (`:114`) is the backstop for
  `numSubQueries() == 0`, unreachable because such a query is compiled to a limit-0 `emptyQuery`.
- **`sorted()` (`:187-224`) is a bounded top-*k* heap with an explicit stability tie-break.** Fast path
  (`!(sortOrder instanceof OrderList) || size < 2`) is a plain `List.sort` — every vertex-centric query
  lands there. Graph-centric path boxes each element into `Ranked<R>{element, Object[] values =
  orders.values(element), int position}`, compares with `orders.compareValues` then
  `Integer.compare(position, position)`, and when `kept = min(limit, size) < size` fills a
  `PriorityQueue<>(kept + 1, byValues.reversed())`, polling the head so the heap holds the best *kept*:
  Θ(*m* log *k*) comparisons, Θ(*k*) memory, *m* order-value reads instead of Θ(*m* log *m*). `first =
  new ArrayList<>(heap)` is then **re-sorted** because heap order is not sorted order. The `position`
  tie-break is a **correctness** property, not a nicety: a `PriorityQueue` is not stable, so without it
  `limit(10)` and `limit(20)` in one transaction could disagree on the first ten.
- **`withoutResultsAfter` is the only downward flow in the stack, and it signals success by reference
  inequality.** Guard (`:170-176`): there is a limit, `!hasDuplicateResults()`, the new elements **fill**
  the limit, and the sort order **is** an `OrderList`; then the value is
  `((OrderList) getSortOrder()).value(sortedNew.get(getLimit() - 1), 0)` — the first order key's value on
  the last new element inside the limit — and may still be `null` if that element has no value, since a
  missing value sorts last. Per sub-query (`:95-100`) it is offered only when `subquery.isSorted()`, and
  acceptance is `narrowed != subquery.getBackendQuery()`. `QueryExecutor.withoutResultsAfter` is a
  `default` method returning its argument, so **identity is the negative answer** — the same idiom as
  lesson 21's `requireValues`. `BackendQueryHolder.withBackendQuery` exists for this and copies the
  profiler across (`holder.profiler = profiler;`) so the narrowed query is timed under the original span.
- **`LimitAdjustingIterator`: the doubling and the two resumption units.** Concrete subclass is
  `QueryProcessor:298-318`; `super(Integer.MAX_VALUE-1, backendQueryHolder.getBackendQuery().getLimit())`.
  `hasNext` (`LimitAdjustingIterator:76-113`): grow only when `currentLimit < maxLimit` **and** either the
  limit is used up (`count >= currentLimit`) or the wrapped iterator ended while being `BackendLimitAware`
  with `hasBackendReachedLimit()`. Growth is
  `currentLimit = (int) Math.min(maxLimit, Math.round(currentLimit * 2.0))`, then a **fresh**
  `getNewIterator(currentLimit)` — there is no cursor, the backend is re-read from the start — then either
  `skipBackendResults(getBackendResultsRead())` (backend results, no element built) or
  `for (int i = 0; i < count; i++) iterator.next()` (elements, rebuilt). `count` is incremented in
  `next()`, so it counts results handed **upward to the filters**, not to the user: **growth is the price
  of an unfitted query and nothing else.** `Query.NO_LIMIT == Integer.MAX_VALUE > maxLimit`, so an
  unbounded read never grows; and `checkArgument(currentLimit > 0 && maxLimit > 0)` means a limit-0
  backend query throws here, kept unreachable by `query.isEmpty()`.
- **`BackendLimitAware` exists because element-count resumption is wrong for a filtering iterator.**
  `hasBackendReachedLimit()` is `subQuery.hasLimit() && position >= subQuery.getLimit()`;
  `getBackendResultsRead()` is `position`, advanced only when an element is actually handed out and
  deliberately **not** by `close()`'s lookahead probe; `skipBackendResults(long)` is guarded by
  `Preconditions.checkState(!started, "Ids can only be skipped before the iterator looks for its first
  element")`. `search_code` returned one implementing class, **`SubqueryIterator`** (treat as suggestive:
  the index did not return the interface's own file or `LimitAdjustingIterator`). Practical split:
  **graph-centric skips, vertex-centric replays.** The comment at `LimitAdjustingIterator:93-94` —
  *"Close old iterator. This is needed otherwise it would not be timed properly by profiler."* — is
  load-bearing: `SubqueryIterator` starts its timer in its constructor and stops it in `close()`, and
  `SimpleQueryProfiler.startTimer` opens with `checkArgument(!runningTimer, "A timer is already
  running")`, so a skipped `close()` would **throw** on the next read of that span, not merely lose a
  measurement.
- **`PreSortingIterator` (`:247-296`, constructor `:257-275`).** Taken when
  `query.isSorted() && !subquery.isSorted()`. Reads under `updateLimit(MAX_SORT_ITERATION)`, so it
  **overrides `query.hard-max-limit` in both directions**; counts every result in `fetched` but adds only
  `!result.isRemoved()` ones to the list, because a removed vertex's order values cannot be read (and
  `QueryProcessorTest#leavesOutTheRemovedElementsBeforeSortingWhatTheBackendReturns` pins it with a mock
  whose `valueOrNull` throws); throws `QueryException` on `fetched >= MAX_SORT_ITERATION` **after** the
  full read, with no incremental bail-out; and is constructed **inside the per-sub-query loop**, so the
  million is per sub-query. The boundary **rejects a complete 1,000,000-element result** (candidate 53).
  This is the exception named in `0f578b3f`'s message for `g.V().order().by("name").limit(10)`.
- **Deletion semantics, two different timings, both deliberate.** `result.isRemoved()` elements are
  dropped **eagerly** inside `PreSortingIterator`. Everything else the transaction deleted — including
  elements changed so they no longer match — is filtered **lazily as reached** by `getFilterIterator`, in
  both branches, with the in-tree reason: *"so that a change which the caller makes while it reads the
  results is seen."* `isDeleted` is therefore called once per backend result on an unfitted query with
  deletions, which is visible in Metrics as a high-count timer.
- **The profiler, complete.** `QueryProfiler` is five instance methods (`addNested(String, boolean)`,
  `setAnnotation`, `startTimer`, `stopTimer`, `setResultSize`), a `NO_OP` whose `addNested` returns
  `this`, and four `static profile(...)` helpers that wrap a `Function` in start/stop/`setResultSize`.
  Group names: `OR-query`, `AND-query`, `backend-query`, `optimization`,
  `constructGraphCentricQuery`, `GraphCentricQuery`, `MixedIndexCountQuery` (constant name is
  `MIXED_INEX_COUNT_QUERY` — typo in the identifier, value correct). Annotations: `condition`, `orders`,
  `limit`, `multi`, `multiPreFetch`, `vertices`, `partitioned`, `isFitted`, `isOrdered`, `query`,
  `fullscan`, `index`, `queries`, `queriesAmount`, `multiSlices`, `limitsPerQuery`.
- **Two profiler implementations produce different tree shapes from identical instrumentation.**
  `TP3ProfileWrapper.addNested` **flattens** a single-child `AND-query`/`OR-query` into its parent
  (`if (!hasSiblings && (groupName.equals(AND_QUERY) || groupName.equals(OR_QUERY))) return this;`), ids
  children `parentId + "." + groupName + "_" + subMetricCounter++`, `toString()`s any non-`String`/
  `Number` annotation value, and **accumulates** result size via
  `metrics.incrementCount(TraversalMetrics.ELEMENT_COUNT_ID, size)`. `SimpleQueryProfiler.addNested`
  **ignores `hasSiblings`** and always allocates a child, stores annotations as-is (`convert` is a no-op
  with its type-narrowing lines commented out in-tree), and **assigns** `resultSize`. `hasSiblings` is
  computed in exactly one place: `BaseVertexCentricQuery.observeWith` →
  `boolean consistsOfMultipleQueries = queries.size() > 1;`. `GraphCentricQuery.observeWith` calls the
  one-argument `indexQuery.observeWith(profiler)`, i.e. `hasSiblings == false`, so a graph query's
  `OR-query` span is **always** flattened in a TinkerPop profile and **always** present in a
  `SimpleQueryProfiler` tree (candidate 60).
- **`BackendQueryHolder.observeWith` is the sole annotator of a sub-query**, and the origin of
  `isFitted=`/`isOrdered=` in a profile: `addNested(OR_QUERY, hasSiblings)` then `FITTED_ANNOTATION`,
  `ORDERED_ANNOTATION`, `QUERY_ANNOTATION`, then `if (backendQuery instanceof ProfileObservable)
  ((ProfileObservable) backendQuery).observeWith(profiler)`. A holder never observed still carries
  `QueryProfiler.NO_OP`, and `QueryProcessor` reads `getProfiler()` at **construction** — so `observeWith`
  must precede the stack build or the read is silently unprofiled. Its `hashCode()` is
  `Objects.hash(backendQuery, isFitted, isFitted)` — `isFitted` twice, `isSorted` never — while `equals`
  compares `isSorted` (candidate 54).
- **`QueryProcessor` creates no profiler span.** Three `profiler` uses, all pass-through. No `addNested`,
  `startTimer`, `stopTimer` or `setResultSize` anywhere in the file. So merge, `sorted()`, `allNewSet` and
  `matches` time is absent from `.profile()` **and** from Metrics; a `backend-query` span's result size is
  `SubqueryIterator`'s `readCount`, i.e. **ids read** including skipped and null-converting ones; and a
  growing `LimitAdjustingIterator` emits **several sibling `backend-query` spans with doubling `limit`
  annotations**, which is the only external signature of the doubling.
- **`GraphCentricQuery.toString()` prints the limit where it means the orders**:
  `if (!orders.isEmpty()) b.append(getLimit());` followed by
  `if (hasLimit()) b.append("(").append(getLimit()).append(")");`, so an ordered limited graph query
  renders `[condition]10(10):vertex`. Because `TP3ProfileWrapper.setAnnotation` `toString()`s it, that is
  what lands in a real profile's `query` annotation (candidate 52).
- **`MetricsQueryExecutor` is unrelated to the profiler** and wraps `getNew`, `hasDeletions`, `isDeleted`
  and `execute` with `<prefix>.query.<name>.<op>.{calls,time,exceptions}` on `MetricManager.INSTANCE`.
  **`withoutResultsAfter` is the one method it does not wrap** — straight delegation, no counter — so the
  narrowing path is invisible to Metrics as well as to the profiler.
- **Iterator plumbing.** `CloseableIteratorUtils.filter` is a `CloseableAbstractIterator` that calls
  `close()` on its delegate when the source is exhausted; `concat` is `MultiIterator`, which closes each
  inner iterator as it is exhausted and, on `close()`, closes the current one and drains and closes the
  rest. `emptyIterator()` is a shared singleton (`EMPTY_CLOSABLE_ITERATOR`) cast per call.
- **Construction sites and the bypass.** `search_code` for `new QueryProcessor` returns **three**:
  `GraphCentricQueryBuilder.iterables` (wrapped in a `CloseableIteratorUtils.filter` on
  `aClass::isInstance`), `BasicVertexCentricQueryBuilder.executeIndividualRelations`, and
  `QueryProcessorTest.run`. The vertex-centric site is guarded —
  `if (useSimpleQueryProcessor(query, vertex)) return new SimpleVertexQueryProcessor(query, tx).relations(); else return new QueryProcessor<>(query, tx.edgeProcessor);`
  — and `useSimpleQueryProcessor` begins `if (!query.isSimple()) return false;`, where
  `BaseVertexCentricQuery.isSimple()` is `queries.size() == 1 && queries.get(0).isFitted() &&
  queries.get(0).isSorted()`. **So the single-slice fitted sorted case, the common one, does not use
  `QueryProcessor` at all** — it goes to `SimpleVertexQueryProcessor`, which lessons 23/24 own.
- **The test surface.** `QueryProcessorTest`
  (`janusgraph-core/src/test/java/org/janusgraph/graphdb/query/QueryProcessorTest.java`) is the dedicated
  unit test: 7 `@Test` methods, a Mockito `ElementQuery`, a `FakeBackendQuery extends BaseQuery` whose
  `updateLimit` returns a copy, and a fake `Executor` that counts `newTaken`, records every executed
  query and marks narrowing by returning a **new** instance. Its fixture hard-codes
  `when(query.numSubQueries()).thenReturn(1)`, so the backwards fold, `ResultMergeSortIterator`, the
  `<=` tie-break and the `concat` arm have **no coverage here** (candidate 57). The seven names are
  worth knowing verbatim: `takesTheNewElementsOfAnUnorderedQueryOnlyAsFarAsItsLimit`,
  `readsTheBackendOfAnUnorderedQueryOnceItsNewElementsRunOut`,
  `asksTheBackendOfAnOrderedQueryForTheResultsUpToItsLastNewValueWithinTheLimit`,
  `asksNothingWhereTheNewElementsDontFillTheLimit`,
  `asksNothingWhereTheLastNewElementWithinTheLimitHasNoValue`,
  `asksNothingOfABackendWhichDoesntSort`,
  `leavesOutTheRemovedElementsBeforeSortingWhatTheBackendReturns`.
- **Two upstream commits read this run that belong in later lessons' citations.** `0f578b3f` (2026-10-22,
  fixes #5124) re-selects a **sorting mixed index** when the first index selection covered nothing, and
  its message is the clearest in-tree English on why a joint query reads all but its first sub-query
  whole; it also names the `PreSortingIterator` `QueryException` as the failure it avoids. `b7c14c53`
  (2026-10-23, fixes #5126) adds `index.[X].solr.max-page-size` (default **10,000**) and reads Solr
  results in **doubling pages** after a first page of `max-result-set-size`, with measurements: 31,846
  hits in **5.9 s** at page size 50 versus **0.52 s** at 1,000, because Solr re-collects and re-sorts
  every hit before each offset. **The growing-batch read therefore appears independently at three levels
  of this system** — `LimitAdjustingIterator`, `SolrIndex`, and the compile-time `2^uncovered`
  multiplier.

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
