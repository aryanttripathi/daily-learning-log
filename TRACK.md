<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 24
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-22 at the Part V/VI boundary (lesson 24) — PERFORMED, and the pin stays
  for the FIFTH review in a row. list_commits on JanusGraph/janusgraph master returns a59ae8d8
  (2026-10-28, Restore 1.3.0-SNAPSHOT version CTR), 9a30ff14 (2026-10-28, Restore 1.2.1-SNAPSHOT) and
  5e3e7bc7 JanusGraph release 1.2.0 (2026-10-24), then 0f578b3f (2026-10-22, fixes #5124), b7c14c53
  (2026-10-23, fixes #5126), c29f6f36 (fixes #5122), 18594751 (fixes #5120) and 297ce21f (fixes #5118).
  The pinned commit is still the tip; there is nothing to re-pin to.
  Method: every file cited by line number is transcribed to local disk and its git blob SHA computed
  and compared against what get_file_contents reported. Lesson 24 did this for FOURTEEN files, all
  matching. Keep it.
  NEXT REVIEW: the Part VI boundary — after lesson 28, or at the capstone. Lessons 25, 26 and 27 are
  mid-Part and must NOT re-raise it.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (candidates 1-64, FROZEN, plus the "Still open, and who owns
  it" ledger) and tracks/janusgraph-registers-2.md (candidates 65-67, next free 68) — new 2026-11-22
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open. **Part V closed on 2026-11-22 with lesson 24. Four lessons remain, all of Part VI.**

**Mode: `lesson`, switched 2026-11-03.** Sessions 01–04 ran as `contributor-reading`: a short
reading guide per day, with four to six questions deliberately left unanswered and the `## My
Notes` half written by Aryant. All four shipped with that half as the untouched template — 24
questions posed, none answered, no evidence a file was opened. Session 04's entry stated the
problem plainly, named the two honest options, and recorded a decision rule here for session 05 to
execute rather than ask a fifth time. **Session 05 executed it.** From lesson 05 on, entries answer
their own questions and carry `Where This Breaks Down`, `Further Study`, `Next Steps` and
`Takeaways` — same syllabus, same pinned commit, same depth bar. The carried questions are tracked
under each entry's `Next Steps` and settled as the syllabus reaches them. Flipping back is one line
of the front-matter above; nothing else in this file depends on the mode. **The mode question is
closed. Do not reopen it unprompted.**

**Why bottom-up.** JanusGraph is a translation layer: a property graph mapped onto a
BigTable-style key → sorted-columns store, and Gremlin traversals turned into slice queries over
those rows. The syllabus follows that layering upward: storage contract → graph ⇄ bytes → write
path → read path → Gremlin integration → schema, indexes and production backends. Classes that
are reference material rather than reading material (`GraphDatabaseConfiguration`, most of
`StandardJanusGraphTx`) are visited only for the slices a lesson needs.

**Pinned source.** All line numbers and permalinks use commit `a59ae8d8` (2026-10-28) on
`JanusGraph/janusgraph` `master`. Re-pin only at a Part boundary, and record the new commit here.
**Reviewed 2026-11-22 at the Part V/VI boundary (lesson 24): PERFORMED, and the pin stays for the fifth
time.** `list_commits` on `master` returns the same three newest commits as at lessons 13, 18 and 22 —
`a59ae8d8`, `9a30ff14` (both 2026-10-28) and `5e3e7bc7` *JanusGraph release 1.2.0* (2026-10-24) — so the
repository has not moved in 25 days and Part VI's line numbers will stay comparable with lessons 02–24.
**The verification method is now blob-SHA identity and nothing weaker.** Lesson 22 introduced it, lesson
23 used it on four files, and lesson 24 used it on **fourteen**, including a 137,928-byte
`GraphDatabaseConfiguration.java`. A matching blob SHA makes a local copy byte-identical to the blob at
the pinned commit *regardless of who transcribed it*, which is what lets a run delegate the
transcription of a large file to a sub-agent and still cite it by line number. Lesson 23's weaker
"delegated read, member-name citations only" grade is now reserved for files that were **not**
transcribed at all.

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

**The registers are now two files, split again on 2026-11-22.**
[`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md) holds candidates **1–64** and the
*"Still open, and who owns it"* ledger and is **frozen** — do not rewrite it.
[`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md) holds candidates **65 on** (next
free **68**). Numbering is continuous across the two volumes. The reason for the second split is the same
as the first, one level down: volume 1 had reached **134 KB**, and every file this log writes has to be
re-emitted whole to be pushed, so appending three candidates to it costs a full 134 KB re-emission. A run
that logs candidates must write the register **in the same push** as this file, per the rule the
2026-11-19 desync produced; lesson 24 did.

## Session Log

Newest first. **Session-log roll-off:** the full `**Findings.**` paragraph is kept for the three most
recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons 09-12 were
compressed on 2026-11-13, lesson 13 on 2026-11-14, lesson 14 on 2026-11-15, lesson 15 on 2026-11-16,
lesson 16 on 2026-11-17, lesson 17 on 2026-11-20, lesson 20 on 2026-11-21, and **lesson 21's paragraph
was compressed this run with its `Verified by` block removed**. The full paragraphs remain verbatim in
git history at the commit each line names. Next run, lesson 22's block and paragraph are the ones that
go.

**New on 2026-11-22: the one-line entries for lessons 01–19 were dropped from this log.** They carried
nothing the rest of the repository does not carry better — the **Syllabus** below lists all 28 lesson
titles as taught, and the root [`README.md`](README.md) index links every entry and every digest by date
and number — while costing ~23 KB of a file that is read in full at the start of every run and
re-emitted in full on every push. Their full text is in git history at the commit immediately preceding
the 2026-11-22 entry. Entries roll off this log one per run from here; the window is the five most
recent lessons.

- **24 — 2026-11-22** · [`2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md`](2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md) · **Part V complete. The Part V/VI re-pin review was PERFORMED and the pin stays for the fifth time. All four debts lesson 23 handed over are discharged.** Read the whole TinkerPop step and multi-query surface at the pinned commit, **fourteen files transcribed to disk and blob-SHA verified locally**, all matching on the first attempt: `JanusGraphStep` (374 lines, 17,030 bytes = `315640d2`; cited at `:61`, `:63-70`, `:73-123`, `:77-122`, `:78-83`, `:86`, `:88`, `:89-90`, `:94-99`, `:101-104`, `:105`, `:107`, `:109-121`, `:110`, `:116-118`, `:119-120`, `:125-142`, `:130-132`, `:134-138`, `:144-163`, `:153-157`, `:158`, `:161`, `:165-172`, `:174-181`, `:179`, `:183-196`, `:185-186`, `:188-190`, `:192`, `:193`, `:198-203`, `:205-241`, `:214-218`, `:254`, `:265`, `:270-278`, `:285-289`, `:322-334`, `:336-357`, `:344-355`, `:359-370`), `JanusGraphVertexStep` (197, 8,152 = `d328dff0`; `:49`, `:51-54`, `:56-69`, `:60-68`, `:71-77`, `:74-76`, `:79-98`, `:100-115`, `:109`, `:112`, `:113`, `:117-129`, `:131-137`, `:174`, `:193-196`), `MultiQueriable` (65, 3,310 = `8496dfe1`), `JanusGraphMultiQueryStep` (182, 7,262 = `1f5e9a63`; `:40`, `:48-50`, `:89-104`, `:93-103`, `:106-114`, `:116-124`, `:117`), `JanusGraphMultiQueryStrategy` (260, 13,779 = `60fa1caf`; `:45`, `:47`, `:58-107`, `:59-61`, `:68-72`, `:74-77`, `:80-94`, `:96-104`, `:101-103`, `:118-128`, `:113-114`, `:130-147`, `:133-142`, `:138`, `:146`, `:149-157`, `:165-210`, `:175-177`, `:179`, `:181-201`, `:199-201`, `:200`, `:203-208`, `:207`, `:212-233`, `:218`, `:219-222`, `:224-227`), `JanusGraphUnusedMultiQueryRemovalStrategy` (94, 4,348 = `225ead59`; `:41`, `:47-71`, `:63-70`, `:73-84`), `JanusGraphNoOpBarrierVertexOnlyStep` (64, 2,746 = `2810c6f3`; `:30`, `:44-57`, `:46-48`, `:49`, `:53-55`), `JanusGraphTraversalUtil` (431, 22,204 = `8290888d`; `:254-312`, `:259-263`, `:319-321`, `:331-345`, `:334-343`, `:350-365`, `:361`, `:373-380`, `:382-388`, `:390-403`, `:414-420`, `:428-430`), `JanusGraphRepeatStep` (148, 7,289 = `3a6f1b34`; `:42`, `:55`), `MultiQueryPositions` (28, 1,003 = `5b47c194`; `:24-26`), `MultiQueriableStepBatchFetcher` (146, 7,250 = `e4a25136`; `:32`, `:36-42`, `:55-69`, `:71-81`, `:83-91`, `:93-97`, `:99-109`, `:104`, `:115-125`, `:117-118`, `:127-129`, `:131-138`, `:140-142`), `VertexStepBatchFetcher` (41, 1,700 = `d226dcef`; `:24-40`), `BatchProcessingQueue` (94, 2,693 = `141dcb7e`; `:29-33`, `:40-42`, `:44-53`, `:48`, `:55-65`, `:67-77`, `:87-89`), `GraphDatabaseConfiguration` (2,191, 137,928 = `4d5a9167`; `:365-366`, `:368-377`, `:379-384`, `:386-391`, `:401`, `:403-415`, `:668-670`, `:1702-1704`, `:1778-1796`, `:2058`, `:2069-2073`). **Read inline, member-name citations only:** `JanusGraphMultiQueryStrategyTest`, `OptimizerStrategyTest`. **Seen only as `search_code` fragments:** `MultiVertexCentricQueryBuilder`, `QueryProfiler`, `JanusGraphAssert`. Three directory listings; three `search_code` queries, all returning results; `list_commits` on `master`. Three PR candidates logged (**65-67**) in the **new** [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file. Daily Diff: the **2026-11-21** edition (47 stories), deep dive on **Arctic**, the OSDI '26 practical lock-free adaptive radix tree.

  **Findings.** **THE finding: the batch size of a `JanusGraphVertexStep` is written by two strategies, and on the shipped default the second write always wins.** `JanusGraphVertexStep.batchSize` defaults to **`Integer.MAX_VALUE`** (`:54`), not to zero or one. `JanusGraphLocalQueryOptimizerStrategy` pass 1 never calls `setBatchSize` (lesson 23) and pass 5 calls `setBatchSize(txVertexCacheSize)` (`:170`, default **20000**). Then `JanusGraphMultiQueryStrategy:179` executes `boolean barrierSizeSet = !limitedBatch;` — and `query.batch.limited` defaults to **`true`** (`GraphDatabaseConfiguration:379-384`), so `barrierSizeSet` starts `false` and the strategy writes a size over whatever pass 5 left: a user `barrier(n)` if there is one (`:219-222`), else the generated barrier's size (`:224-227`) which is `query.batch.limited-size`, default **2500** (`:386-391`), else `limitedBatchSize` directly (`:199-201`). **So on a default graph `g.V().out()` and `g.V().local(out())` both batch at 2500 and `cache.tx-cache-size` has no effect on batching at all.** The asymmetry lesson 23 asked about only becomes observable with `query.batch.limited=false`, where `barrierSizeSet` starts `true`, no `setBatchSize` call happens anywhere in the strategy, and the top-level step keeps `Integer.MAX_VALUE` — one multi-query over every start vertex — while the local one keeps 20000. The two halves agree: at `limited=false` `JanusGraphMultiQueryStep.initialize()` (`:93-103`) also drains its **entire** upstream into an `ArrayList` before passing it on. **Nobody has run the experiment; Hands-On A is it.** Second: **the two steps are not a matched pair.** `JanusGraphStep` implements `HasContainerHolder` and compiles a `GraphCentricQuery` inside itself; `JanusGraphVertexStep` implements `MultiQueriable` and only forwards arguments to a `BaseVertexQuery` builder. Only `JanusGraphStep` supports an offset (`setLimit`'s low bound, consumed by `MultiDistinct*Iterator` at `:110`/`:119`, never passed to a query); `JanusGraphVertexStep.setLimit` asserts `low == 0` (`:174`). Only `JanusGraphVertexStep` batches. Third: **both of `JanusGraphStep`'s execution arms converge on the same merge**, so the de-duplicating offset-applying `MultiDistinctUnorderedIterator`/`MultiDistinctOrderedIterator` is *the* execution path, not a special case for split `or()`s — and the split arm gives each of N branch queries the **full** `highLimit` (`:179`), so a `limit(10)` over a three-branch `or()` can read 30 elements. The `ids != null && ids.length > 0` arm consults **no index at all**: `iteratorList`/`holdsConditions` (`:322-357`) re-check every folded `has()` and every `or()` in Java. Fourth, **debt (b) is fully settled: all four multi-query annotations are set in one class, `MultiVertexCentricQueryBuilder`** — `multi=true` and `vertices=N` in `constructQuery`, `partitioned=true` in the partitioned-vertex branch, `multiPreFetch=true` in `preFetch()`. **`vertices` is the only one carrying a number, and it is the batch size actually used**, which makes it the observable for Hands-On A. Fifth, the **insert-then-remove** design is deliberate and javadoc'd (`JanusGraphMultiQueryStrategy:113-114`): a `JanusGraphMultiQueryStep` is inserted before every `MultiQueriable` *and* every `TraversalParent`, and `JanusGraphUnusedMultiQueryRemovalStrategy` deletes the ones with three empty client lists **together with the barrier they generated**, identified by reference equality on `getGeneratedBarrierStep()` — the same identity test that distinguishes a generated barrier from a user's `barrier(n)` when choosing a batch size (`JanusGraphTraversalUtil:361`). Sixth: `JanusGraphNoOpBarrierVertexOnlyStep` is a four-line subclass whose entire content is `break`ing out of the fill loop on a non-vertex traverser (`:53-55`) plus refusing to top up a non-empty barrier (`:46-48`). Three PR candidates: **65** (`isLegalMultiQueryPosition` walks `currentStep` and tests `step`, so the walk is a no-op and the answer depends only on the position step's own labels — the one reportable defect, and **not yet confirmed by a run**), **66** (`BatchProcessingQueue.setBatchSize` drops the constructor's `Math.max(batchSize,1)`, and neither `query.batch.limited-size` nor `cache.tx-cache-size` has a `positiveInt()` verifier where `MAX_KEYS_PER_READ` at `:401` does), **67** (`JanusGraphVertexStep`'s copy constructor does not copy `batchSize`; latent, no reachable path found).

- **23 — 2026-11-21** · [`2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md`](2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md) · **Part V opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the strategy layer at the pinned commit. **Blob-SHA verified and cited by line number:** `JanusGraphLocalQueryOptimizerStrategy` (whole, 245 lines, 13,515 bytes = `01de0bed`), `HasStepFolder` (whole, 532 lines, 27,466 bytes = `0232fc9b`), `StandardJanusGraph` (whole, 1,489 lines, 80,408 bytes = `f650acb2`) and `JanusGraphStepStrategyTest` (whole, 546 lines, 46,982 bytes = `abd23a81`). **Read inline, cited by member name:** `JanusGraphStepStrategy`, `JanusGraphStep`, `JanusGraphMixedIndexCountStrategy`, `AbstractJanusGraphMixedIndexAggStrategy`, `OptimizerStrategyTest`. **Read by a delegated reader** (text not locally transcribed, so member-name citations only): the four `AdjacentVertex*` strategies and their abstract base, `JanusGraphHasStepStrategy`, `JanusGraphMultiQueryStrategy`, `JanusGraphUnusedMultiQueryRemovalStrategy`, `JanusGraphMixedIndexAggStrategy`, `JanusGraphIoRegistrationStrategy`. Four PR candidates logged (**61-64**). Daily Diff: the 2026-11-19 / 61-story edition; deep dive on PostgreSQL 19's `REPACK CONCURRENTLY` and the `pg_plan_advice` / `pg_stash_advice` extensions.

  **Findings.** **THE finding: the content of the strategy layer is its dependency graph, and two of the twelve strategies declare no ordering at all.** `StandardJanusGraph`'s one `static` block (`:165-185`) registers twelve strategies, **all** of them `ProviderOptimizationStrategy` — including `JanusGraphIoRegistrationStrategy`, whose entire `apply()` configures an `IoStep` — and registers the same object for `StandardJanusGraph.class` and `StandardJanusGraphTx.class`. OLAP gets **one** of the twelve (`FulgoraUtil.FULGORA_STRATEGIES` is `JanusGraphLocalQueryOptimizerStrategy` alone). Reading the full `applyPrior`/`applyPost` graph produces the fact Part V turns on: **`JanusGraphStepStrategy` and `JanusGraphLocalQueryOptimizerStrategy` have no direct ordering edge**, and the only paths between them run through `JanusGraphMixedIndexCountStrategy` and `JanusGraphMixedIndexAggStrategy`, which a user disables with one `withoutStrategies(...)` call. No traversal was found where the order is observable, so it is filed as a missing declaration (candidate 61), not a defect. Second: **`local()` is the only construct that gets an order or a range to the storage layer.** Pass 1 folds a `has()` only for an edge-returning step, folds no order and no range (`:106`: *"We cannot fold in orders or ranges since they are not local"*) and folds only a *high* limit from a following `RangeGlobalStep` **without removing it**; pass 5 calls `foldInHasContainer`, `foldInOrder` **and** `foldInRange`, sets `setBatchSize(txVertexCacheSize)`, and `unfoldLocalTraversal` deletes the `LocalStep` entirely when its child collapses to one step. A vertex-centric query folds **at most one** order key (`:423-425`); a graph-centric one folds any number. **Pass 1 never calls `setBatchSize` at all** — handed to lesson 24, and answered there. Third, lesson 22's profiler debt: **the attachment order on the normal path is safe**, but `buildGlobalGraphCentricQuery()` **memoizes**, and `JanusGraphMixedIndexAggStep`'s constructor calls it during strategy application, before TinkerPop's `setMetrics`, so a traversal ending in `count()`/`min()`/`max()`/`sum()`/`mean()` is **compiled under `QueryProfiler.NO_OP`, has its index selection done at strategy time, and shows no `constructGraphCentricQuery` span in `.profile()`** when the push-down is declined — the common case with no mixed index. The probe also mutates the step it declined to replace (`hasLocalContainers.put(null, {[]})`); `toString()`, `hashCode()` and the or()-fallback switch were traced and none changes behaviour (candidate 62). Fourth, the second debt is answered **no**: the TinkerPop side never produces a multi-sub-query `ElementQuery`; `or()` is folded only for a `JanusGraphStep` (`:185`), each branch flattened into a `List<HasContainer>` — with an `order().by(k)` inside a branch becoming `has(k, neq(null))` — and then executed either as **one** disjunctive `GraphCentricQuery` or as **N** single-sub-query ones merged **in the step**. So candidates 55 and 56 stand and the right PR is a **comment**, not a deletion. Fifth, the third debt: `search_code` found **nothing** in the repository asserting profile-tree shape, so the `AND-query`/`OR-query` flattening is not a tested contract and candidate 60 must be filed before any new-span patch. Also: `validFoldInOrBranch` is **all-or-nothing across a whole `or()`** (candidate 64), and `AdjacentVertexHasUniquePropertyOptimizerStrategy` is the one strategy permitted to touch the backend, gated on `optimizerBackendAccess()`, which it uses to **run a side traversal during strategy application**.

- **22 — 2026-11-20** · [`2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md`](2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md) · **Part IV complete. The Part IV/V re-pin review was PERFORMED and the pin stays for the fourth time.** Read the whole `graphdb/query` top level and the profile package at the pinned commit: `QueryProcessor` (whole, 320 lines, **blob-SHA verified** — 16,048 bytes = `b28e6887`), `LimitAdjustingIterator` (whole, 131 lines, **blob-SHA verified** — 5,961 bytes = `96a7e3ae`), plus `ResultSetIterator`, `ResultMergeSortIterator`, `BackendQueryHolder`, `QueryExecutor`, `MetricsQueryExecutor`, `ElementQuery`, `BackendLimitAware`, `QueryProfiler`, `SimpleQueryProfiler`, `TP3ProfileWrapper`, `GraphCentricQuery`, `BaseVertexCentricQuery`, `VertexCentricQuery`, `CloseableIteratorUtils`, `MultiIterator`, `SubqueryIterator` and `QueryProcessorTest` — all whole, all cited by member name. **THE finding: the in-memory filter sits between the backend limit and the element count**, so `limit(10)` bounds what the caller receives and not what is read, and the backend limit **doubles** — `L, 2L, 4L …` with a full re-read each time, no cursor — until enough survive. Second: **`QueryProcessor` creates no profiler span at all**, so merge, sort and filter time is invisible in `.profile()` and in Metrics. Third: **half the class is dead** — `VertexCentricQuery.isSorted()` is hard-coded `true` and `GraphCentricQuery.numSubQueries()` hard-coded `1`. Fourth: **the single-slice fitted sorted vertex-centric case does not use `QueryProcessor` at all** — `useSimpleQueryProcessor` diverts it to `SimpleVertexQueryProcessor`. Nine PR candidates logged (**52-60**). Daily Diff: no new edition existed, so the uncovered-item fallback was used: #15, Percona on Galera `pc.weight`.

- **21 — 2026-11-19** · [`2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md`](2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md) · **Findings.** Paragraph compressed 2026-11-22 under the roll-off rule, and the `Verified by lesson 21` block removed in the same run; the full versions are in git history at the 2026-11-19 through 2026-11-21 commits, and the facts lessons 25–28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the two planners are the same kind of function and a different shape** — `GraphCentricQueryBuilder` consults no statistics either, but solves a **weighted set cover** with a lexicographic objective (more clauses covered, then *fewer* indexes, then higher total score), may choose several indexes, and may choose none, which is the full scan. Index selection sits behind a swappable SPI (`IndexSelectionStrategy`), the default `threshold-based` strategy admits `2^10 = 1024` candidate groups per compiled query, a composite index is all-or-nothing and excluded outright under a top-level `Or`, and execution is stream-one/intersect-the-rest with the non-first sub-queries materialised **eagerly and unbounded**. Six PR candidates, **46-51**.

- **20 — 2026-11-18** · [`2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md`](2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md) · **Findings.** Paragraph compressed 2026-11-21 under the roll-off rule, and the `Verified by lesson 20` block removed in the same run; the full versions are in git history at the 2026-11-18 through 2026-11-20 commits. The load-bearing facts: **the number of `SliceQuery` shapes a vertex-centric traversal makes per vertex is a product, not a constant** — one ordinary traversal compiles to three slices, each its own cache entry on the same edgestore row; **the planner that chooses between a label's vertex-centric indexes is one `double` and no statistics at all**, whose strict `>` makes every tie resolve to the plain label slice; and direction is the low bit of the varint inside the relation type, so a prefix slice cannot select a direction. Lessons 25 and 27 still owe the `bestCandidate != null` reachability question and the `setDirectColumnByStartOnlyAllowed` flag.

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
- [x] 24 — `JanusGraphStep` and `JanusGraphVertexStep`: folding `has`/`limit`/`order` into storage queries, and multi-query batching

### Part VI — Schema, indexes, and production backends

- [ ] 25 — `ManagementSystem` and the index lifecycle: INSTALLED → REGISTERED → ENABLED, and reindex jobs
- [ ] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30. Part V is complete and lesson 25 opens Part VI.**

**THE fact to carry forward, and the one a reader of this track will misremember: the batch size of a
`JanusGraphVertexStep` is written twice and on the default configuration the second write wins.** Default
field value `Integer.MAX_VALUE`; pass 5 of `JanusGraphLocalQueryOptimizerStrategy` writes
`txVertexCacheSize`; `JanusGraphMultiQueryStrategy` then writes 2500 over it, because
`boolean barrierSizeSet = !limitedBatch` (`:179`) and `query.batch.limited` defaults to `true`. **So
`cache.tx-cache-size` has no effect on batching on a default graph**, and the pass-1/pass-5 asymmetry
lesson 23 found is only observable at `query.batch.limited=false`. Lessons 25–28 should not re-derive
this, and should not state the asymmetry without the `limited` qualifier.

**Second, and the piece of unfinished business lesson 25 should pick up if it has five spare minutes:
none of lesson 24's three Hands-On experiments has been run.** Hands-On A (the `vertices` annotation at
default vs `limited=false`, top-level vs `local()`) is the only direct evidence for the paragraph above —
everything there is derived from reading. Hands-On B is what turns candidate 65 from a reading into a
reportable defect. **Candidate 65 must not be filed upstream until B has been run.**

**Third, the multi-query annotations, settled and not to be re-searched.** All four of
`QueryProfiler`'s `multi` / `multiPreFetch` / `vertices` / `partitioned` are set in
`MultiVertexCentricQueryBuilder` and nowhere else in main sources: `multi` and `vertices` in
`constructQuery`, `partitioned` in the partitioned-vertex branch, `multiPreFetch` in `preFetch()`.
**`vertices` is the batch size actually used**, and it is the only one of the four carrying a number —
which also means the test helper `JanusGraphAssert.queryProfilerAnnotationIsPresent`, which matches the
literal string `"<annotation>=true"` in `TraversalMetrics.toString()`, **cannot** assert on it. That is
why nothing in the repository asserts a numeric batch size, and it is the gap candidate 67's test should
fill.

**Fourth, carried from lesson 22's block, which rolls off next run — the three facts lessons 25–28 were
promised.** (a) **Deletion semantics, for lesson 28:** `result.isRemoved()` elements are dropped
**eagerly** inside `PreSortingIterator`; everything else the transaction deleted is filtered **lazily as
reached** by `getFilterIterator`, deliberately, *"so that a change which the caller makes while it reads
the results is seen."* (b) **For lesson 27:** `PreSortingIterator`'s `MAX_SORT_ITERATION = 1_000_000`
budget is **per sub-query**, it reads the whole result before throwing rather than bailing out
incrementally, and the boundary rejects a complete 1,000,000-element result (candidate 53); combined with
`LimitAdjustingIterator`'s cursor-less doubling re-read, an unfitted ordered query on CQL is the worst
case in the tree — put a number on it against a real backend. (c) **For lesson 26:** `b7c14c53` added
`index.[X].solr.max-page-size` (default 10,000) and a page-doubling read in `SolrIndex`; compare it with
`LimitAdjustingIterator`'s doubling and say whether `ElasticSearchIndex` has an equivalent. The full
lesson-22 block is in git history at the 2026-11-20 through 2026-11-22 commits.

**Fifth: Part VI is four lessons and three of them touch code this track has only seen from above.**
Lessons 26 and 27 are the first to read a non-in-memory backend. Budget for that — a CQL or Elasticsearch
file is large, and the blob-SHA-verified transcription method (above) is what makes a large file citable
by line number, so plan on delegating the transcription and verifying the SHA locally rather than trying
to read 100 KB inline.

- **Lesson 25 (`ManagementSystem` and the index lifecycle) inherits four.** (a) whether any shipped
  relation type has `isInvisibleType() == true` without being a system type; (b)
  `mgmt.buildEdgeIndex`/`buildPropertyIndex` with the `TypeDefinitionCategory.RELATIONTYPE_INDEX` schema
  edge; (c) reads skip a **composite** index unless `index.getStatus() == SchemaStatus.ENABLED`, with no
  base-type exemption, while a **mixed** index is filtered **per `ParameterIndexField` status** inside
  `coversAll` — lesson 25 owns the state machine that moves those statuses and should state which of the
  two a reindex relies on; (d) from lesson 23,
  `AdjacentVertexHasUniquePropertyOptimizerStrategy` decides validity with
  `IndexSelectionUtil.existsMatchingIndex(...)` requiring a **composite, `Cardinality.SINGLE`** index and
  is gated on `GraphDatabaseConfiguration#optimizerBackendAccess()` — the one strategy allowed to touch
  the backend, which it does by running a side traversal during strategy application. Say what it does
  with an index whose status is not `ENABLED`. Still starts at `ManagementSystem.createCompositeIndex:848-850`.
- **Lesson 26 (`ElasticSearchIndex`) inherits four feature flags:** `supportNotQueryNormalForm()`,
  `supportsOrderingListProperty()`, `supportsExistsQuery()` and `supportsOrdering(KeyInformation)` (added
  by `18594751`, fixes #5120). Lesson 23 added the second's caller:
  `HasStepFolder.validJanusGraphOrder` (`:102-143`) consults
  `graph.getIndexSerializer().allMixedIndexBackendSupportsOrderingListProperty()` to decide whether a
  **vertex** order on a non-`SINGLE` key is foldable at all, so that flag changes query *compilation*,
  not just execution. Plus the Solr page-doubling comparison in (c) above.
- **Lesson 27 (CQL) inherits three, unchanged.** The `setDirectColumnByStartOnlyAllowed` flag from lesson
  20; the feature-builder classification lesson 18 owed (whether `CQLStoreManager` and `HBaseStoreManager`
  set `.locking(...)`, and so whether `ConsistentKeyLocker` runs on them); and the worst-case read cost in
  (b) above. Also from lesson 16: whether the CQL `BatchStatement` ordering is explicit or implicit in
  client-side timestamps (`graph.assign-timestamp`, LOCAL, default **true**).
- **Lesson 28 (transaction log, CDC, stale-index repair) inherits three.** The deletion semantics in (a)
  above; lesson 23's `MultiDistinctOrderedIterator` cross-query ordering invariant, which is an `assert`;
  and **new from lesson 24**, the same invariant one layer up — `JanusGraphStep:118`'s
  `assert queries.stream().allMatch(q -> q.getOrder().isEmpty() || q.getOrder().equals(order))` is the
  only thing standing between two differently-ordered branch queries and a silently mis-ordered merged
  result, and assertions are off in production. A stale mixed index is exactly the condition that
  produces it.
- **Open and unresolved, deliberately not filled in.** (a) Lesson 20's
  `BasicVertexCentricQueryBuilder:711` `Preconditions.checkArgument(bestCandidate != null, …)`
  reachability question. (b) Whether any production path passes a real limit to
  `QueryUtil.processIntersectingRetrievals` (candidate 51 is a reachability claim, not a confirmed
  defect). (c) Whether `BackendQueryHolder`'s duplicated `isFitted` in `hashCode` (candidate 54) can
  degrade anything. (d) `IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel
  colliding with `IndexCandidate`'s default score (inside candidate 50). (e) Lesson 23's §2.2 ordering
  question — is the `JanusGraphStepStrategy` / `JanusGraphLocalQueryOptimizerStrategy` order observable by
  any traversal? Hands-On B of lesson 23 is the experiment; nobody has run it. (f) **New from lesson 24:**
  whether `BatchProcessingQueue.removeLast()` has any caller at all — `HasStepBatchFetcher` and
  `DropStepBatchFetcher` were not read, so it must not be called dead code.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, 23 ~59, **24 ~59** (digest
  ~21). A large share of each is permalink URLs, which are not prose.

**Mechanical notes, carried and extended this run. Read this section before planning a run's pushes.**
`get_file_contents` on a file whose doubled output exceeds roughly 40 KB returns an over-limit error that
saves the JSON to a local file — the cheap way to read a large file, costing nothing in context. The
correct extraction is `json.loads(longest_text)["content"][*]["resource"]["text"]`, **not** the plain
`text` value (which has a `[Resource from github at repo://…]` banner glued onto line 1). A file under
roughly 20 KB comes back inline with no line numbers and must be written to local disk to be numbered.
**The transcription can be delegated to a sub-agent and the blob SHA verified locally afterwards** —
lesson 24 did this for fourteen files, including a 137,928-byte one, all matching first time, and that is
now the standard method: it moves the transcription cost out of the run's own context entirely while
*strengthening* the citation grade, because byte-identity is a proof and a delegated read without it is
only testimony.

**The real constraint on a run is OUTPUT, not input, and it is now the binding one.** Every file pushed
must be emitted in full as a tool argument; there is no server-side copy or patch. Reading a 134 KB file
costs almost nothing; *rewriting* it costs ~37 K output tokens. That is why lesson 24 made two structural
changes rather than one: the session-log entries for lessons 01–19 were dropped (−23 KB, and they
duplicated the syllabus and the root README index), and the register was split a second time into
`tracks/janusgraph-registers-2.md`, freezing volume 1 at 134 KB rather than re-emitting it to append three
candidates. This file went from **126 KB to ~90 KB** as a result. **Watch the trend: it still grows by
roughly 8 KB per lesson**, and the root `README.md` (63 KB, regenerated in full every run) grows too. If
this file passes ~110 KB again, the next structural move is to split the `Verified by lesson NN` blocks
into `tracks/janusgraph-verified.md` — they are ~45 KB of the total, they are append-and-roll-off
material, and a run would then emit two medium files instead of one large one. **That is a decision for
Aryant**, and it is flagged rather than taken, because it changes where a run looks for the facts it is
told to cite rather than re-derive.

**Mermaid.** Render every diagram with `mmdc` before pushing. Lesson 24's three (two in the lesson, one
in the digest) rendered clean on the first attempt with the lesson-19 rule observed: quote every node
label containing parentheses, commas, `->`, `|` or angle brackets, use HTML entities such as `&lt;` for
comparisons, and never escape a quote inside a label — drop the inner quotes instead.

**Daily Diff procedure — a FOURTH disagreement shape, and the archive was the endpoint that was right.**
At ~03:40 UTC the front page served **2026-11-20 / 50 stories**, while `/archive/` reported *146
Editions* with `Latest >` pointing at **`/2026-11-21/`** and calendar links for 11-19 (*61*), 11-20
(*50*) and 11-21 (*47*), with 11-22 unlinked. `/2026-11-21/` returned **HTTP 200** and served the
47-story edition; `/2026-11-22/` returned `client_error`. This is the exact inverse of yesterday, two
runs apart: on 2026-11-21 the archive advertised an edition the site would not serve, and today it
advertised one the site served cleanly while the front page lagged. **The rule stands and was applied:
take the maximum *fetchable* edition.** So 2026-11-21 was digested. **2026-11-20 is uncovered and was
deliberately skipped** — it is now in the **fallback pool** alongside the unspent items from 2026-11-06
(two) and 2026-11-18 (two), and a future run with no new edition should take the whole uncovered 11-20
edition before falling back to a single spent item. The procedure is otherwise unchanged: (a) read the
front page and the archive and take the maximum; (b) probe the dated URL for today and the previous two
days; (c) check this session log in both directions, as a floor and a ceiling; (d) only then the
uncovered-item fallback. **Constraint still in force: there is no `WebFetch` tool in this environment** —
pages are fetched through a sub-agent which flags which parts are verbatim and which are an extracted
summary, and the summary portion is not reliable enough to quote. This run's fetch of `/2026-11-21/` was
verbatim through item 21 of 47, so the digest lists items 1–21 and says so. Edition sizes remain
volatile: 22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61, 61, 50, 47 across 11-08 to 11-21. **A PDF fetched
for a deep dive splits the same way:** the Arctic OSDI paper came back verbatim through §3.8.2 and
summary-only after, and the digest labels every claim from the summary half as untrusted. Expect that
split on any paper over ~40 K characters and plan the deep dive around the readable half.

### Verified by lesson 24, for lessons 25, 26, 27 and 28 to cite rather than re-derive

- **`JanusGraphStep` is a `GraphStep` + `HasStepFolder` + `Profiling` + `HasContainerHolder` (`:61`) and
  is NOT a `MultiQueriable`.** `JanusGraphVertexStep` is a `VertexStep` + `HasStepFolder` + `Profiling` +
  **`MultiQueriable`** (`:49`). Everything about batching is the second step; everything about compiling a
  `GraphCentricQuery` inside a step is the first. Only `JanusGraphStep` accepts an offset (`setLimit`
  `:285-289`, consumed at `:110`/`:119`); `JanusGraphVertexStep.setLimit` has
  `Preconditions.checkArgument(low == 0, …)` (`:174`) and `getLowLimit()` throws (`:178-181`).
- **`JanusGraphStep`'s iterator supplier has four arms** (`:77-122`): `ids == null` → empty (`:78-79`);
  `ids.length > 0` → `graph.vertices(ids)`/`edges(ids)` filtered **in memory** by `iteratorList`
  (`:322-334`) and `holdsConditions` (`:336-357`), which requires at least one branch of every folded
  `or()` to hold — **no index is consulted on this arm at all**; otherwise one disjunctive
  `GraphCentricQuery` (`:88-90`) or N per-branch ones (`:91-99`). The arm switch is
  `if (!globalQuery.getSubQuery(0).getBackendQuery().isEmpty() || hasLocalContainers.size() != 1)`
  (`:88`), and `hasLocalContainers` is keyed by **`or()` step id** (`:254`, `:265`), so `size()` counts
  `or()` **steps, not branches**.
- **Both arms converge on the same merge (`:109-121`), so `MultiDistinctUnorderedIterator` /
  `MultiDistinctOrderedIterator` is the execution path of every non-id `JanusGraphStep`, not a special
  case.** The offset lives only there. `highLimit` is applied **twice** — `query.limit(highLimit)` at
  `:161`/`:179` *and* the merge bound — so N branch queries can read `N × highLimit` elements to emit
  `highLimit`. The cross-query order invariant at `:118` is an `assert`.
- **`buildGraphCentricQuery(query, profiler)` (`:183-196`) is where a `JanusGraphStep` compiles:**
  `addNested(CONSTRUCT_GRAPH_CENTRIC_QUERY)` + `startTimer` (`:185-186`) → `disableSmartLimit()` iff the
  end step is a `CountGlobalStep` (`:188-190`) → `requireOrderValues()` (`:192`) →
  `constructQueryWithoutProfile(VERTEX|EDGE)` (`:193`) → `stopTimer`. An `or()` with no passable branch
  returns `GraphCentricQuery.emptyQuery(...)` at `:153-157`.
- **`JanusGraphVertexStep.makeQuery` (`:100-115`) forwards; it does not compile.** A folded
  `order().by(k)` emits **two** calls — `query.has(order.key)` then `query.orderBy(...)` (`:106-111`) —
  the vertex-centric twin of `requireOrderValues()`, and the in-source comment says the compiler takes
  that `has(key)` as an unbounded range which another condition on the key narrows. Only a high limit is
  passed (`:112`). The profiler is attached at `:113` by an unconditional cast to
  `BasicVertexCentricQueryBuilder`, at execution time, so unlike the aggregation path it always gets the
  live `TP3ProfileWrapper`.
- **The batch-size decision, complete and in order.** Field default `Integer.MAX_VALUE`
  (`JanusGraphVertexStep:54`). `JanusGraphLocalQueryOptimizerStrategy` pass 1: no `setBatchSize`; pass 5:
  `setBatchSize(txVertexCacheSize)` (`:170`), `cache.tx-cache-size` default **20000**
  (`GraphDatabaseConfiguration:668-670`). Then `JanusGraphMultiQueryStrategy.configureMultiQueriables`
  (`:165-210`): `boolean barrierSizeSet = !limitedBatch` (`:179`) where `query.batch.limited` defaults
  **true** (`:379-384`); if false-started, `applyPreviousMultiQueryAndReturnIfBarrierSizeSet`
  (`:212-233`) writes a user `barrier(n)` size (`:219-222`) else the generated barrier's size
  (`:224-227`) = `query.batch.limited-size` default **2500** (`:386-391`), else `:199-201` writes
  `limitedBatchSize`; finally `setUseMultiQuery(true)` (`:207`), which builds the
  `VertexStepBatchFetcher` with whatever `batchSize` then is (`JanusGraphVertexStep:74-76`). At
  `query.batch.limited=false` **no `setBatchSize` call happens anywhere in the strategy**.
  `query.batch.enabled` defaults **true** (`:368-377`); `query.batch.repeat-step-mode` defaults
  `ALL_REPEAT_PARENTS` (`:403-415`). **Neither `LIMITED_BATCH_SIZE` nor `TX_CACHE_SIZE` has a
  `positiveInt()` verifier**, where `MAX_KEYS_PER_READ` (`:401`) does.
- **`JanusGraphMultiQueryStep` is a tee, not a buffer** (`:40`): three client lists (`:48-50`),
  `processNextStart` passes the traverser through and calls `registerTraverser` (`:106-124`), which is
  guarded by `traverser.get() instanceof Vertex` (`:117`). Its one piece of control flow is
  `initialize()` (`:89-104`): **when `limitBatchSize` is false it drains the entire upstream into an
  `ArrayList`**, registers every vertex and pushes the list back as its own starts (`:93-103`).
- **Three batch queues, and the loop arithmetic.** `MultiQueriableStepBatchFetcher` (`:32`) holds
  first/current/next-loop `BatchProcessingQueue`s (`:39-41`). `fetchData` (`:99-109`) **replaces**
  `multiQueryResults` wholesale (`:104`) — no merge, no LRU. `prefetchNextBatch` (`:115-125`) always does
  `multiQuery.addVertex(requiredFetchVertex)` (`:118`), so **a batch is a hint, never a correctness
  constraint**. `ensureCorrectLoopQueues` (`:131-138`) promotes next→current only on `loops ==
  currentLoops + 1` and otherwise discards both queues, so a loop jump silently degrades batching to one
  vertex per query. `BatchProcessingQueue` de-duplicates across all pending batches via
  `allBatchedElements` (`:31`) and `removeFirst` flushes the partial last batch rather than waiting
  (`:55-65`).
- **Insert-then-remove is deliberate and javadoc'd** (`JanusGraphMultiQueryStrategy:113-114`). A tee goes
  before every `MultiQueriable` *and* every `TraversalParent` *and* at the end of every repeat traversal
  (`:118-128`); `JanusGraphUnusedMultiQueryRemovalStrategy` removes the ones with three empty client
  lists **and their generated barrier** (`:63-70`, `:73-84`). The generated barrier is identified by
  **reference equality** on `getGeneratedBarrierStep()`, both there and in
  `JanusGraphTraversalUtil.getLocalNonMultiQueryProvidedBatchSize` (`:361`) — that one identity test is
  what separates a user's `barrier(n)` from the strategy's own.
- **`JanusGraphNoOpBarrierVertexOnlyStep` (`:30`) differs from TinkerPop's `NoOpBarrierStep` in exactly
  two lines:** it returns early if the barrier is non-empty (`:46-48`) and it `break`s out of the fill
  loop on the first non-vertex traverser (`:53-55`), so for non-vertices it behaves as if its size were 1.
- **`repeat()` is replaced by exact class** (`:101-103`):
  `getStepsOfAssignableClassRecursively(RepeatStep.class, …)` then `getClass() == RepeatStep.class`, then
  `JanusGraphRepeatStep.replace` (`JanusGraphRepeatStep:55`). The in-source reason (`:97-98`): the
  inserted barriers would otherwise switch TinkerPop's `RepeatStep` into its barrier mode, which feeds
  *all* start traversers into the repeat traversal at once — the barrier added to bound the batch would
  destroy the batching.
- **Position rules.** `getLocalMultiQueryPositionForStep` (`:331-345`) walks back over `SideEffectStep`
  and `ProfileStep`, puts the tee **before** a preceding `NoOpBarrierStep` (`:338-340`), and returns
  empty at the traversal start (`:341-343`).
  `getAllMultiQueryPositionsForMultiQueriable` (`:254-312`) short-circuits to a single same-loop position
  for any step that is not its traversal's start (`:259-263`); the outward walk through repeat parents
  exists only for steps that *are* a start, and is gated by the two booleans derived from
  `repeatStepMode` (`JanusGraphMultiQueryStrategy:80-94`).
- **`isLegalMultiQueryPosition` (`:390-403`) is broken — candidate 65.** The loop advances `currentStep`
  and the body tests the parameter `step`, so the walk is a no-op and the answer reduces to
  `step.getLabels().isEmpty()` (or unconditional `true` when `step` is the start step). The javadoc
  (`:382-388`) says it should match TinkerPop's `LazyBarrierStrategy`, which accumulates over every step
  up to the position. Not yet confirmed by a run.
- **`MULTIQUERY` / `NUMVERTICES` / `PARTITIONED_VERTEX` / `MULTIPREFETCH` are all set in
  `MultiVertexCentricQueryBuilder`** and nowhere else in main sources (`search_code` fragments; the file
  was not transcribed). `vertices=N` is the batch size **actually used**.
- **The test surface, for lesson 25 onward.** `JanusGraphMultiQueryStrategyTest` is **concrete** and
  extends the abstract `OptimizerStrategyTest`, whose `@BeforeEach open()` is
  `graph = StorageSetup.getInMemoryGraph()`; every test calls `clopen(option(USE_MULTIQUERY), true, …)`
  which rebuilds the graph with `STORAGE_BACKEND=inmemory`. Six `@Test` methods; the two that matter here
  are `testNoOpBarrierStepInsertedIfNotPresentAndLimitBatchSize` (asserts **1**
  `JanusGraphNoOpBarrierVertexOnlyStep` with `LIMITED_BATCH` true, and **1** plain `NoOpBarrierStep` when
  the user wrote `barrier(1)`) and `testNoOpBarrierStepNotInsertedLimitBatchSizeDisabled` (asserts **0**
  of either with `LIMITED_BATCH` false). **Neither asserts a numeric batch size, and nothing else in the
  repository does either.** Run one with
  `mvn test -pl janusgraph-test -am -Dtest=JanusGraphMultiQueryStrategyTest#<method> -DfailIfNoTests=false -Dsurefire.failIfNoSpecifiedTests=false`.
  The only dedicated test for the queue is
  `janusgraph-core/src/test/java/.../fetcher/BatchProcessingQueueTest.java`.

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
  `new HashSet<>(Arrays.asList(JanusGraphLocalQueryOptimizerStrategy, JanusGraphHasStepStrategy, JanusGraphStepStrategy))`
  — **re-verified by lesson 24 against the transcribed file at `:47`.**
  `JanusGraphUnusedMultiQueryRemovalStrategy.applyPrior()` = `new HashSet<>(Arrays.asList(JanusGraphMultiQueryStrategy))`
  — **re-verified at `:41`.**
  **`AdjacentVertexFilterOptimizerStrategy` and `JanusGraphIoRegistrationStrategy` declare neither** — the
  first is nevertheless ordered because `JanusGraphLocalQueryOptimizerStrategy` names it; the second has no
  edge in either direction.
- **Consequence, and the single most useful fact in Part V: the `JanusGraphStepStrategy` ↔
  `JanusGraphLocalQueryOptimizerStrategy` order is transitive-only**, via
  `JanusGraphStepStrategy` → `JanusGraphMixedIndexCountStrategy`/`JanusGraphMixedIndexAggStrategy` →
  `JanusGraphLocalQueryOptimizerStrategy`. No traversal was found where the order is observable; filed as a
  missing declaration (candidate 61), not a defect.
- **`JanusGraphLocalQueryOptimizerStrategy` is five passes in a fixed order** (`:92-96`):
  `applyJanusGraphVertexSteps` → `applyJanusGraphPropertiesSteps` → `applyJanusGraphLabelSteps` →
  `applyJanusGraphDropSteps` → `inspectLocalTraversals`. Guards: `!traversal.getGraph().isPresent()`
  (`:64-65`) and `janusGraph == null` (`:68-70`); **no `onGraphComputer` guard**, because this is the one
  strategy OLAP installs. Config comes from `TransactionConfiguration` when
  `JanusGraphTraversalUtil.getJanusGraphTx(traversal)` is present (`:78-84`) and from
  `GraphDatabaseConfiguration` otherwise (`:85-90`) — so **a per-transaction mode override only affects
  traversals started from the transaction**.
- **Pass 1 vs pass 5, the whole point of the strategy.** Pass 1 (`:99-117`): every `VertexStep` becomes a
  `JanusGraphVertexStep`; `foldInHasContainer` **only** if `isEdgeReturnStep` (`:105`); the comment at
  `:106` is *"We cannot fold in orders or ranges since they are not local"*; the only fold is a **high**
  limit from a following `RangeGlobalStep`, and the range step is **not removed** (`:110-114`). Pass 5
  (`:163-197`), for a `LocalStep` whose child starts with a `VertexStep` or `PropertiesStep`:
  `setBatchSize(txVertexCacheSize)` (`:170`, `:185`), `foldInHasContainer`,
  **`foldInOrder(..., isVertexOrder=false)`** (`:175`, `:191`) and **`foldInRange`** (`:177`, `:193`),
  then `unfoldLocalTraversal` (`:225-235`) which replaces the entire `LocalStep` with the single step in
  the **outer** traversal when the child collapsed to one step. **Pass 1 never calls `setBatchSize`** —
  answered by lesson 24, see above.
- **Pass 2 (`:118-161`) matches three unrelated TinkerPop classes in one predicate** (`:119`):
  `PropertiesStep` → `JanusGraphPropertiesStep`, `PropertyMapStep` → `JanusGraphPropertyMapStep`,
  `ElementMapStep` → `JanusGraphElementMapStep`; idempotence guard
  `if (originalStep instanceof MultiQueriable) return;` (`:120-122`); `setBatchSize(txVertexCacheSize)`
  at `:158` — **and, per lesson 24, that value is also overwritten by `JanusGraphMultiQueryStrategy`
  under the default `query.batch.limited=true`.** Passes 3 and 4 (`:199-223`) fold nothing and exist only
  to give the multi-query layer a `MultiQueriable`.
- **`HasStepFolder` is an interface with 8 abstract and 14 `static` methods** (blob-SHA verified, 27,466
  bytes = `0232fc9b`, 532 lines). `foldInIds` `:145-178`, `foldInHasContainer` `:181-214`,
  `localFoldInHasContainer` `:225-245`, `validFoldInOrBranch` `:247-268`, `branchPassesElements` `:270-281`,
  `validFoldInHasContainer` `:327-343`, `isExistsStep` `:351-363`, `foldInHasFilter` `:367-378`,
  `foldInOrder` `:380-434`, `splitAndP` `:437-445`, `foldInRange` `:484-502`,
  `foldableHasContainerNoLimit` `:507-531`, `validJanusGraphOrder` `:102-143`.
- **`or()` folding, exactly.** The `OrStep` arm (`:185`) is guarded by
  `janusgraphStep instanceof JanusGraphStep`, so **a vertex-centric step never folds an `or()`**.
  `validFoldInOrBranch` is **all-or-nothing across the whole `or()`** — one unrecognised step `return`s and
  abandons every branch — and requires at least one `has()` or exists step. `branchPassesElements` rejects
  a branch whose `RangeGlobalStep` has `getLowRange() > 0 || getHighRange() == 0`, because **a branch is
  evaluated per element**. `localFoldInHasContainer` converts a `has(key)` exists step and **every key of
  an `order()`** into `HasContainer(key, P.neq(null))`. `addLocalHasContainersOfNoBranch` (`:195-197`)
  records an `or()` that no element can pass as a parent id with an **empty** branch set.
  `JanusGraphStepStrategyTest.java:342-380` is the specification table.
- **`foldInOrder` collects consecutive `OrderGlobalStep`s and iterates them BACKWARDS**, in-source reason:
  *"TinkerPop sorts by each order() in turn, keeping the order of ties … so the elements come in the order
  of the last one."* The gate at `:423-425` means **a vertex-centric query folds at most one order key; a
  graph-centric one folds any number.** `validJanusGraphOrder` (`:102-143`) rejects a `ValueTraversal`
  with a non-null `getBypassTraversal()`, `Order.shuffle`, a key with no `PropertyKey`, a
  non-`Comparable` `dataType()`, and a **vertex** order on a non-`Cardinality.SINGLE` key unless
  `graph.getIndexSerializer().allMixedIndexBackendSupportsOrderingListProperty()` — so that mixed-index
  feature flag changes query **compilation**, not just execution (lesson 26).
- **`foldInRange` (`:484-502`) is asymmetric.** `low` is folded **only** when
  `janusgraphStep instanceof JanusGraphStep`; otherwise it is hard `0`. The `RangeGlobalStep` is removed
  only when the folder is a `JanusGraphStep` **or** `range.getLowRange() == 0`.
- **`JanusGraphStepStrategy` checks the ids twice.** If `ArrayUtils.isEmpty(getIds())`: create
  `JanusGraphStep`, `foldInIds`, `foldInHasContainer`, then **re-check** before `foldInOrder` and
  `foldInRange` — in-source: *"if there are any ids folded … we shouldn't fold in order and range steps so
  that they can be handled by TinkerPop."* The `else` arm creates **no** `JanusGraphStep`. It also
  early-returns on `TraversalHelper.onGraphComputer(traversal)`.
- **`buildGlobalGraphCentricQuery()` memoizes, and `JanusGraphMixedIndexAggStep`'s constructor calls it at
  strategy time.** `if (globalQuery != null) return globalQuery;`, and when `hasLocalContainers.isEmpty()`
  it inserts `hasLocalContainers.put(null, {[]})` as a side effect (lesson 24 confirms both verbatim at
  `JanusGraphStep:130-138`). `AbstractJanusGraphMixedIndexAggStrategy.buildMixedIndexAggStep` constructs
  `new JanusGraphMixedIndexAggStep<>(...)` — whose constructor calls that method — and **discards the new
  step** when `getMixedIndexAggQuery() == null`, leaving the original `JanusGraphStep` with a query
  compiled under `NO_OP`. `getAggregation` returns non-null only for a trailing `CountGlobalStep` or a
  single-key `PropertiesStep` followed by `Max`/`Min`/`Mean`/`SumGlobalStep`, both requiring
  `originalGraphStep.isStartStep()`.
- **Nothing in the repository asserts profile-tree shape.** `search_code` for
  `AND_QUERY OR_QUERY TP3ProfileWrapper assert metrics` → 0 results; `"backend-query" profile test
  assertion` → 0 results. Suggestive, not proof, but enough that **candidate 60 must be filed before any
  new-span patch**.
- **The test surface for this layer.** `janusgraph-test/.../optimize/` holds ten files:
  `AdjacentVertexFilterOptimizerStrategyTest` (5,194 bytes), `AdjacentVertexHasIdOptimizerStrategyTest`
  (4,146), `AdjacentVertexHasUniquePropertyOptimizerStrategyTest` (5,476),
  `AdjacentVertexIsOptimizerStrategyTest` (2,729), `JanusGraphIoRegistrationStrategyTest` (1,482),
  `JanusGraphLocalQueryOptimizerStrategyTest` (9,045), `JanusGraphMultiQueryStepTest` (4,705),
  `JanusGraphMultiQueryStrategyTest` (11,886), `JanusGraphStepStrategyTest` (46,982) and the abstract
  `OptimizerStrategyTest` (6,085). **They all run on the in-memory backend.** `makeSampleGraph()` builds
  composite indexes `byId`, `byUniqueId` (unique), `byWeight`, `byIdWeight`, edge indexes
  `byWeightDesc`/`byWeightAsc` on `knows`, a `names` LIST key with a `namesByWeight` property index, then
  `numV = 100` vertices and `superV = 10` super-vertices each with 100 `knows` edges — **no mixed index**,
  which is exactly the condition that makes the aggregation push-down decline.
  `JanusGraphStepStrategyTest` (blob-SHA verified, 46,982 bytes = `abd23a81`, 546 lines) is the
  specification: `doTest` (`:111-122`), `doMultiQueryTest` (`:124-142`), the `or()` rows at `:342-380`,
  and three non-parameterized tests at `:144-170` pinning `splitAndP`'s `AndP` expansion.

### Verified by lesson 22, for lessons 23, 24, 25, 26, 27 and 28 to cite rather than re-derive

*(This block rolls off next run. The three facts lessons 25–28 were promised are restated in
**Notes For The Next Run** above so nothing is lost when it goes; the full text stays in git history.)*

- **The stack, outward from the backend, built at `QueryProcessor:101-107`.**
  `executor.execute(...)` → `LimitAdjustingIterator` (or `PreSortingIterator`) →
  `filter !allNewSet.contains` (graph-centric only, `:103-105`) →
  `getFilterIterator(hasDeletions, !subquery.isFitted())` (`:107`) → the merge fold (`:109-111`) →
  `ResultSetIterator` (`:69-74`). **The in-memory filter is below the element count and above the backend
  limit.** `getUnfoldedIterator` runs **eagerly inside `iterator()`**.
- **Three limits, and only the middle one moves.** `query.getLimit()` counts **elements emitted** and is
  enforced by `ResultSetIterator`, which reads one element ahead and calls `close()` the instant the count
  is reached. `backendQueryHolder.getBackendQuery().getLimit()` counts **raw backend results** and is the
  one that **doubles**. `MAX_SORT_ITERATION = 1_000_000` (`:56`) bounds a pre-sort and **throws** rather
  than truncating.
- **The branch map.** `VertexCentricQuery.isSorted()` → **`true`** unconditionally; `numSubQueries()` →
  `queries.size()`; `hasDuplicateResults()` → `false`. `GraphCentricQuery.isSorted()` →
  `!orders.isEmpty()`; `numSubQueries()` → literal **`1`**; `isEmpty()` → `getLimit() <= 0`.
  `search_code` for `implements ElementQuery` returns **exactly these two classes**. Consequences, all
  dead code in the shipped tree: the unsorted `concat` + `seenResults` arm (`:139-150`) is unreachable;
  `ResultMergeSortIterator`'s `filterDuplicates` is always `false`; `lastFirstOrderValueWithinLimit`
  returns `null` for every vertex-centric query.
- **The merge fold is binary, right-leaning and left-biased.** The loop runs backwards (`:93`) so
  sub-query 0 ends up the outermost **left** operand; ties break with `if (c <= 0)`. No *k*-way heap:
  *n* sub-queries cost up to *n−1* comparator calls per emitted element.
- **`sorted()` (`:187-224`) is a bounded top-*k* heap with an explicit stability tie-break.** The
  `position` tie-break is a **correctness** property: a `PriorityQueue` is not stable, so without it
  `limit(10)` and `limit(20)` in one transaction could disagree on the first ten.
- **`withoutResultsAfter` signals success by reference inequality** — `QueryExecutor.withoutResultsAfter`
  is a `default` method returning its argument, so **identity is the negative answer**.
- **`LimitAdjustingIterator`: the doubling and the two resumption units.** Growth is
  `currentLimit = (int) Math.min(maxLimit, Math.round(currentLimit * 2.0))`, then a **fresh**
  `getNewIterator(currentLimit)` — there is no cursor, the backend is re-read from the start — then either
  `skipBackendResults(...)` (graph-centric) or replaying `count` `next()` calls (vertex-centric).
  **Growth is the price of an unfitted query and nothing else.**
- **`PreSortingIterator` (`:247-296`)** reads under `updateLimit(MAX_SORT_ITERATION)`, so it **overrides
  `query.hard-max-limit` in both directions**, drops `isRemoved()` elements eagerly, throws
  `QueryException` on `fetched >= MAX_SORT_ITERATION` **after** the full read, and is constructed inside
  the per-sub-query loop, so the million is **per sub-query** (candidate 53).
- **Two profiler implementations produce different tree shapes from identical instrumentation.**
  `TP3ProfileWrapper.addNested` **flattens** a single-child `AND-query`/`OR-query` into its parent and
  **accumulates** result size; `SimpleQueryProfiler.addNested` **ignores `hasSiblings`** and **assigns**
  `resultSize`. `GraphCentricQuery.observeWith` passes `hasSiblings == false`, so a graph query's
  `OR-query` span is **always** flattened in a TinkerPop profile (candidate 60).
- **`BackendQueryHolder.observeWith` is the sole annotator of a sub-query** and the origin of
  `isFitted=`/`isOrdered=`. A holder never observed still carries `QueryProfiler.NO_OP`, and
  `QueryProcessor` reads `getProfiler()` at **construction**. Its `hashCode()` is
  `Objects.hash(backendQuery, isFitted, isFitted)` — `isFitted` twice, `isSorted` never — while `equals`
  compares `isSorted` (candidate 54).
- **`QueryProcessor` creates no profiler span.** A growing `LimitAdjustingIterator` emits **several
  sibling `backend-query` spans with doubling `limit` annotations**, which is the only external signature
  of the doubling.
- **Construction sites and the bypass.** `useSimpleQueryProcessor` begins `if (!query.isSimple()) return
  false;` where `isSimple()` is `queries.size() == 1 && isFitted() && isSorted()`, **so the single-slice
  fitted sorted case does not use `QueryProcessor` at all.**
- **The test surface.** `QueryProcessorTest` has 7 `@Test` methods and hard-codes
  `when(query.numSubQueries()).thenReturn(1)`, so the backwards fold, `ResultMergeSortIterator`, the `<=`
  tie-break and the `concat` arm have **no coverage** (candidate 57).
- **Two upstream commits that belong in later lessons' citations.** `0f578b3f` (fixes #5124) re-selects a
  **sorting mixed index** when the first selection covered nothing. `b7c14c53` (fixes #5126) adds
  `index.[X].solr.max-page-size` (default **10,000**) and reads Solr results in **doubling pages**:
  31,846 hits in **5.9 s** at page size 50 versus **0.52 s** at 1,000. **The growing-batch read appears
  independently at three levels of this system.**

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
