<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 25
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-22 at the Part V/VI boundary (lesson 24) — PERFORMED, and the pin stays
  for the FIFTH review in a row. list_commits on JanusGraph/janusgraph master returned a59ae8d8
  (2026-10-28, Restore 1.3.0-SNAPSHOT version CTR), 9a30ff14 (2026-10-28) and 5e3e7bc7 JanusGraph
  release 1.2.0 (2026-10-24). The pinned commit is still the tip; there is nothing to re-pin to.
  Method: every file cited by line number has its git blob SHA computed locally and compared against
  what the API reported. Lesson 25 improved the method: a directory listing with
  fields: ["name","sha","size"] returns every file's blob SHA in ONE cheap call, so a large file read
  through the over-limit path can be verified with no transcription step at all. Lesson 25 verified
  ManagementSystem.java (86,864 bytes = a722a7d9) and ManagementLogger.java (24,707 = 1980d1fb) that way.
  NEXT REVIEW: the Part VI boundary — after lesson 28, or at the capstone. Lessons 26 and 27 are
  mid-Part and must NOT re-raise it; lesson 25 correctly did not.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (candidates 1-64, FROZEN, plus the "Still open, and who owns
  it" ledger) and tracks/janusgraph-registers-2.md (candidates 65-70, next free 71)
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open. **Part VI opened on 2026-11-23 with lesson 25. Three lessons remain: 26, 27 and 28.**

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
free **71**). Numbering is continuous across the two volumes. The reason for the second split is the same
as the first, one level down: volume 1 had reached **134 KB**, and every file this log writes has to be
re-emitted whole to be pushed, so appending three candidates to it costs a full 134 KB re-emission. A run
that logs candidates must write the register in the same run as this file, per the rule the 2026-11-19
desync produced. Lesson 24 did it in one push; **lesson 25 did it as two consecutive pushes**, because
this file plus the register exceeds what one tool call can emit — the invariant that matters is that
both land before the run ends, and both did.

## Session Log

Newest first. **Session-log roll-off:** the full `**Findings.**` paragraph is kept for the three most
recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons 09-12 were
compressed on 2026-11-13, lesson 13 on 2026-11-14, lesson 14 on 2026-11-15, lesson 15 on 2026-11-16,
lesson 16 on 2026-11-17, lesson 17 on 2026-11-20, lesson 20 on 2026-11-21, lesson 21 on 2026-11-22, and
**lesson 22's paragraph was compressed this run with its `Verified by lesson 22` block removed**. The full
paragraphs remain verbatim in git history at the commit each line names. Next run, lesson 23's block and
paragraph are the ones that go.

**On 2026-11-22 the one-line entries for lessons 01–19 were dropped from this log; lesson 20's went on
2026-11-23.** They carried
nothing the rest of the repository does not carry better — the **Syllabus** below lists all 28 lesson
titles as taught, and the root [`README.md`](README.md) index links every entry and every digest by date
and number — while costing ~23 KB of a file that is read in full at the start of every run and
re-emitted in full on every push. Their full text is in git history at the commit immediately preceding
the 2026-11-22 entry. Entries roll off this log one per run from here; the window is the five most
recent lessons.

- **25 — 2026-11-23** · [`2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md`](2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md) · **Part VI opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the management and index-lifecycle surface at the pinned commit. **Blob-SHA verified against the directory listing and cited by line number:** `ManagementSystem` (1,815 lines, 86,864 bytes = `a722a7d9`; cited at `:190-207`, `:209-213`, `:241-268`, `:253`, `:256-257`, `:340-393`, `:365`, `:367`, `:368`, `:387`, `:389-391`, `:622-639`, `:628-639`, `:655-679`, `:704-706`, `:720-724`, `:746-767`, `:758`, `:770-808`, `:785-791`, `:791`, `:802`, `:807`, `:831-879`, `:841-846`, `:852`, `:860`, `:877`, `:967`, `:977`, `:980-1140`, `:987-996`, `:1000-1019`, `:1013-1016`, `:1026-1032`, `:1033-1051`, `:1052-1056`, `:1100-1112`, `:1130-1135`, `:1142-1147`, `:1154-1166`, `:1207-1301`, `:1227-1287`, `:1237-1242`, `:1244-1254`, `:1289-1299`, `:1303-1305`, `:1310-1319`, `:1321-1326`, `:1328-1341`, `:1334`, `:1343-1361`, `:1351`, `:1392-1459`, `:1436-1458`) and `ManagementLogger` (505 lines, 24,707 bytes = `1980d1fb`; `:71`, `:169-230`, `:176-207`, `:208-224`, `:235-237`, `:239-260`, `:275-286`, `:288-296`, `:306-315`, `:317-326`, `:328-339`, `:341-357`, `:382-441`). **Read inline, member-name citations only:** `SchemaStatus` (`7b40611d`), `SchemaAction` (`b2bf7951`), `JanusGraphIndexWrapper` (`94d0458d`), `GraphIndexStatusWatcher` (`3ebe2f6a`), `AbstractIndexStatusWatcher` (`e0ab9913`), `IndexUpdateJob` (`f1b9b521`), `IndexRepairJob` (`434abee9`). Two directory listings, one with `fields: ["name","sha","size"]` — **the cheap SHA-verification method this run introduced**. One `search_code` query, returning nothing. Three PR candidates logged (**68-70**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same run as this file, as the second of two consecutive pushes. Daily Diff: the **2026-11-22** edition (38 items, **read in full**), deep dive on **Apache Kvrocks**, Redis mapped onto RocksDB, with a sidebar on **ParadeDB**'s filter-aware vector search.

  **Findings.** **THE finding: most indexes never enter the lifecycle at all, and one expression decides it.** `canIndexBeEnabled` — `oneNewKey || (constraint != null && constraint.isNew())` at `:852` for a composite index, `type.isNew() || any sortKey isNew` at `:365` for a vertex-centric one — writes `ENABLED` instead of `INSTALLED` (`:860` / `:367`) and skips `updateIndex(index, REGISTER_INDEX)` entirely (`:877` / `:389-391`). It is **sound**, not a shortcut: a composite index is all-or-nothing over its fields, so a key created in this management transaction means no pre-existing element can qualify and the backfill is provably empty. It is also asymmetric in a way the docs do not state — **one** new key out of nine is enough, and a brand-new `indexOnly(...)` constraint is enough on its own. Second: **"the index's status" is two physically different things.** Composite and relation-type keep one `SchemaStatus` as a `BaseKey.SchemaDefinitionProperty` tagged `TypeDefinitionCategory.STATUS` (`setStatusVertex:1328-1341`, one property remove plus one add, idempotent at `:1334`). A **mixed** index's index-level status is set to `ENABLED` **unconditionally** at `:758` and means nothing; the real statuses are **per field**, held as the **last element** of each `INDEX_FIELD` schema edge's `Parameter[]`, written once by `addIndexKey:791` and changed by `setStatusEdges:1343-1361` by **removing and re-adding the edge** — carrying `MAPPED_NAME`, the backend field name, along in the copy. The "STATUS is last" contract is established in one place (`:785-791`) and checked only by an `assert` (`:1351`), off in production. `JanusGraphIndexWrapper.getIndexStatus(key)` hides the whole split by **ignoring its key argument** on a composite index, which makes `GraphIndexStatusWatcher`'s per-key poll report the same fact N times there and makes it essential on a mixed index. So **a mixed index is not one lifecycle but N, one per `addIndexKey` call.** Third: **the INSTALLED → REGISTERED transition is a callback in the issuing JVM's heap and nothing persists it.** `REGISTER_INDEX` sets `INSTALLED` and arms an `UpdateStatusTrigger`; `commit` sends the eviction **after** `transaction.commit()` (`:253`, `:256-257`); `EvictionTrigger`'s constructor snapshots the open-instance roster; each receiver queues a `SendAckOnTxClose` that waits for **every transaction open when the message arrived** to close; `runTriggers()` then opens a **new** management transaction and writes `REGISTERED`, but only for elements still `INSTALLED`, so a racing `DISABLE_INDEX` supersedes it. **What `REGISTERED` actually guarantees is therefore not "every instance has seen the change" but "every instance has seen it and every transaction that predated it has finished"** — which is what makes a reindex safe. Lose the issuing JVM mid-handshake and the index is stranded at `INSTALLED` with no recovery path in the code (re-issuing `REGISTER_INDEX` works, undocumented, because `INSTALLED` is in its applicable set); lose a *receiving* instance and `graph.management-auto-close-stale-instances` plus the ack timeout is the only escape, `mgmt.forceCloseInstance` the manual one. Fourth, **lesson 23's debt (c) is settled**: a reindex of a composite or relation-type index relies on the **vertex** status, a reindex of a mixed index on the **field** statuses, both via `IndexRepairJob.validateIndexStatus` — whose mixed branch explicitly **skips `DISABLED` fields** (`status != SchemaStatus.DISABLED && !acceptable`) and fails only when no field is acceptable, so a partly-disabled mixed index reindexes while a composite one has no partial mode. Its javadoc still says *"ENABLED or REGISTERED"* where the set it uses is `REINDEX`'s four statuses. Debts (a) and (b) also settled: **every** vertex-centric index's backing relation type is invisible without being a system type (`:368` calls `maker.invisible()`), and the `RELATIONTYPE_INDEX` schema edge is written at `:387`. Debt (d) is **not** settled — `IndexSelectionUtil` was not located by the one `search_code` query spent on it, and the lesson declined to guess; it passes to lesson 26. Fifth: **`REINDEX` only adds and `REMOVE_STALE_ENTRIES` only removes, and they scan different stores** — edgestore versus graphindex — so returning a `DISABLED` index to trustworthy is two complete passes over the data, and a delete that lands while an index is disabled is invisible to a reindex forever. `REMOVE_STALE_ENTRIES` is also the only action that changes no status and is unsupported for vertex-centric indexes. Sixth: **the reindex-to-enable step is a best-effort callback** (`:1436-1458`) that catches every `Throwable` and logs at error, so a fully successful scan can silently leave the index `REGISTERED`; and `REINDEX`'s applicable set contains `DISABLED`, so reindexing an index that is receiving no writes is permitted and is almost always wrong. Three PR candidates: **68** (both `return null` branches in `updateIndex` are unreachable because `isApplicableStatus` throws rather than returning `false`), **69** (the mixed-index status is a positional array slot behind an `assert`, in the same array as `MAPPED_NAME`), **70** (the pending INSTALLED → REGISTERED callback is unpersisted and JVM-local — the only one of the three with user-visible consequences, and **not yet confirmed by a run**).

- **24 — 2026-11-22** · [`2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md`](2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md) · **Part V complete. The Part V/VI re-pin review was PERFORMED and the pin stays for the fifth time. All four debts lesson 23 handed over are discharged.** Read the whole TinkerPop step and multi-query surface at the pinned commit, **fourteen files transcribed to disk and blob-SHA verified locally**, all matching on the first attempt: `JanusGraphStep` (374 lines, 17,030 bytes = `315640d2`; cited at `:61`, `:63-70`, `:73-123`, `:77-122`, `:78-83`, `:86`, `:88`, `:89-90`, `:94-99`, `:101-104`, `:105`, `:107`, `:109-121`, `:110`, `:116-118`, `:119-120`, `:125-142`, `:130-132`, `:134-138`, `:144-163`, `:153-157`, `:158`, `:161`, `:165-172`, `:174-181`, `:179`, `:183-196`, `:185-186`, `:188-190`, `:192`, `:193`, `:198-203`, `:205-241`, `:214-218`, `:254`, `:265`, `:270-278`, `:285-289`, `:322-334`, `:336-357`, `:344-355`, `:359-370`), `JanusGraphVertexStep` (197, 8,152 = `d328dff0`; `:49`, `:51-54`, `:56-69`, `:60-68`, `:71-77`, `:74-76`, `:79-98`, `:100-115`, `:109`, `:112`, `:113`, `:117-129`, `:131-137`, `:174`, `:193-196`), `MultiQueriable` (65, 3,310 = `8496dfe1`), `JanusGraphMultiQueryStep` (182, 7,262 = `1f5e9a63`; `:40`, `:48-50`, `:89-104`, `:93-103`, `:106-114`, `:116-124`, `:117`), `JanusGraphMultiQueryStrategy` (260, 13,779 = `60fa1caf`; `:45`, `:47`, `:58-107`, `:59-61`, `:68-72`, `:74-77`, `:80-94`, `:96-104`, `:101-103`, `:118-128`, `:113-114`, `:130-147`, `:133-142`, `:138`, `:146`, `:149-157`, `:165-210`, `:175-177`, `:179`, `:181-201`, `:199-201`, `:200`, `:203-208`, `:207`, `:212-233`, `:218`, `:219-222`, `:224-227`), `JanusGraphUnusedMultiQueryRemovalStrategy` (94, 4,348 = `225ead59`; `:41`, `:47-71`, `:63-70`, `:73-84`), `JanusGraphNoOpBarrierVertexOnlyStep` (64, 2,746 = `2810c6f3`; `:30`, `:44-57`, `:46-48`, `:49`, `:53-55`), `JanusGraphTraversalUtil` (431, 22,204 = `8290888d`; `:254-312`, `:259-263`, `:319-321`, `:331-345`, `:334-343`, `:350-365`, `:361`, `:373-380`, `:382-388`, `:390-403`, `:414-420`, `:428-430`), `JanusGraphRepeatStep` (148, 7,289 = `3a6f1b34`; `:42`, `:55`), `MultiQueryPositions` (28, 1,003 = `5b47c194`; `:24-26`), `MultiQueriableStepBatchFetcher` (146, 7,250 = `e4a25136`; `:32`, `:36-42`, `:55-69`, `:71-81`, `:83-91`, `:93-97`, `:99-109`, `:104`, `:115-125`, `:117-118`, `:127-129`, `:131-138`, `:140-142`), `VertexStepBatchFetcher` (41, 1,700 = `d226dcef`; `:24-40`), `BatchProcessingQueue` (94, 2,693 = `141dcb7e`; `:29-33`, `:40-42`, `:44-53`, `:48`, `:55-65`, `:67-77`, `:87-89`), `GraphDatabaseConfiguration` (2,191, 137,928 = `4d5a9167`; `:365-366`, `:368-377`, `:379-384`, `:386-391`, `:401`, `:403-415`, `:668-670`, `:1702-1704`, `:1778-1796`, `:2058`, `:2069-2073`). **Read inline, member-name citations only:** `JanusGraphMultiQueryStrategyTest`, `OptimizerStrategyTest`. **Seen only as `search_code` fragments:** `MultiVertexCentricQueryBuilder`, `QueryProfiler`, `JanusGraphAssert`. Three directory listings; three `search_code` queries, all returning results; `list_commits` on `master`. Three PR candidates logged (**65-67**) in the **new** [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file. Daily Diff: the **2026-11-21** edition (47 stories), deep dive on **Arctic**, the OSDI '26 practical lock-free adaptive radix tree.

  **Findings.** **THE finding: the batch size of a `JanusGraphVertexStep` is written by two strategies, and on the shipped default the second write always wins.** `JanusGraphVertexStep.batchSize` defaults to **`Integer.MAX_VALUE`** (`:54`), not to zero or one. `JanusGraphLocalQueryOptimizerStrategy` pass 1 never calls `setBatchSize` (lesson 23) and pass 5 calls `setBatchSize(txVertexCacheSize)` (`:170`, default **20000**). Then `JanusGraphMultiQueryStrategy:179` executes `boolean barrierSizeSet = !limitedBatch;` — and `query.batch.limited` defaults to **`true`** (`GraphDatabaseConfiguration:379-384`), so `barrierSizeSet` starts `false` and the strategy writes a size over whatever pass 5 left: a user `barrier(n)` if there is one (`:219-222`), else the generated barrier's size (`:224-227`) which is `query.batch.limited-size`, default **2500** (`:386-391`), else `limitedBatchSize` directly (`:199-201`). **So on a default graph `g.V().out()` and `g.V().local(out())` both batch at 2500 and `cache.tx-cache-size` has no effect on batching at all.** The asymmetry lesson 23 asked about only becomes observable with `query.batch.limited=false`, where `barrierSizeSet` starts `true`, no `setBatchSize` call happens anywhere in the strategy, and the top-level step keeps `Integer.MAX_VALUE` — one multi-query over every start vertex — while the local one keeps 20000. The two halves agree: at `limited=false` `JanusGraphMultiQueryStep.initialize()` (`:93-103`) also drains its **entire** upstream into an `ArrayList` before passing it on. **Nobody has run the experiment; Hands-On A is it.** Second: **the two steps are not a matched pair.** `JanusGraphStep` implements `HasContainerHolder` and compiles a `GraphCentricQuery` inside itself; `JanusGraphVertexStep` implements `MultiQueriable` and only forwards arguments to a `BaseVertexQuery` builder. Only `JanusGraphStep` supports an offset (`setLimit`'s low bound, consumed by `MultiDistinct*Iterator` at `:110`/`:119`, never passed to a query); `JanusGraphVertexStep.setLimit` asserts `low == 0` (`:174`). Only `JanusGraphVertexStep` batches. Third: **both of `JanusGraphStep`'s execution arms converge on the same merge**, so the de-duplicating offset-applying `MultiDistinctUnorderedIterator`/`MultiDistinctOrderedIterator` is *the* execution path, not a special case for split `or()`s — and the split arm gives each of N branch queries the **full** `highLimit` (`:179`), so a `limit(10)` over a three-branch `or()` can read 30 elements. The `ids != null && ids.length > 0` arm consults **no index at all**: `iteratorList`/`holdsConditions` (`:322-357`) re-check every folded `has()` and every `or()` in Java. Fourth, **debt (b) is fully settled: all four multi-query annotations are set in one class, `MultiVertexCentricQueryBuilder`** — `multi=true` and `vertices=N` in `constructQuery`, `partitioned=true` in the partitioned-vertex branch, `multiPreFetch=true` in `preFetch()`. **`vertices` is the only one carrying a number, and it is the batch size actually used**, which makes it the observable for Hands-On A. Fifth, the **insert-then-remove** design is deliberate and javadoc'd (`JanusGraphMultiQueryStrategy:113-114`): a `JanusGraphMultiQueryStep` is inserted before every `MultiQueriable` *and* every `TraversalParent`, and `JanusGraphUnusedMultiQueryRemovalStrategy` deletes the ones with three empty client lists **together with the barrier they generated**, identified by reference equality on `getGeneratedBarrierStep()` — the same identity test that distinguishes a generated barrier from a user's `barrier(n)` when choosing a batch size (`JanusGraphTraversalUtil:361`). Sixth: `JanusGraphNoOpBarrierVertexOnlyStep` is a four-line subclass whose entire content is `break`ing out of the fill loop on a non-vertex traverser (`:53-55`) plus refusing to top up a non-empty barrier (`:46-48`). Three PR candidates: **65** (`isLegalMultiQueryPosition` walks `currentStep` and tests `step`, so the walk is a no-op and the answer depends only on the position step's own labels — the one reportable defect, and **not yet confirmed by a run**), **66** (`BatchProcessingQueue.setBatchSize` drops the constructor's `Math.max(batchSize,1)`, and neither `query.batch.limited-size` nor `cache.tx-cache-size` has a `positiveInt()` verifier where `MAX_KEYS_PER_READ` at `:401` does), **67** (`JanusGraphVertexStep`'s copy constructor does not copy `batchSize`; latent, no reachable path found).

- **23 — 2026-11-21** · [`2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md`](2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md) · **Part V opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the strategy layer at the pinned commit. **Blob-SHA verified and cited by line number:** `JanusGraphLocalQueryOptimizerStrategy` (whole, 245 lines, 13,515 bytes = `01de0bed`), `HasStepFolder` (whole, 532 lines, 27,466 bytes = `0232fc9b`), `StandardJanusGraph` (whole, 1,489 lines, 80,408 bytes = `f650acb2`) and `JanusGraphStepStrategyTest` (whole, 546 lines, 46,982 bytes = `abd23a81`). **Read inline, cited by member name:** `JanusGraphStepStrategy`, `JanusGraphStep`, `JanusGraphMixedIndexCountStrategy`, `AbstractJanusGraphMixedIndexAggStrategy`, `OptimizerStrategyTest`. **Read by a delegated reader** (text not locally transcribed, so member-name citations only): the four `AdjacentVertex*` strategies and their abstract base, `JanusGraphHasStepStrategy`, `JanusGraphMultiQueryStrategy`, `JanusGraphUnusedMultiQueryRemovalStrategy`, `JanusGraphMixedIndexAggStrategy`, `JanusGraphIoRegistrationStrategy`. Four PR candidates logged (**61-64**). Daily Diff: the 2026-11-19 / 61-story edition; deep dive on PostgreSQL 19's `REPACK CONCURRENTLY` and the `pg_plan_advice` / `pg_stash_advice` extensions.

  **Findings.** **THE finding: the content of the strategy layer is its dependency graph, and two of the twelve strategies declare no ordering at all.** `StandardJanusGraph`'s one `static` block (`:165-185`) registers twelve strategies, **all** of them `ProviderOptimizationStrategy` — including `JanusGraphIoRegistrationStrategy`, whose entire `apply()` configures an `IoStep` — and registers the same object for `StandardJanusGraph.class` and `StandardJanusGraphTx.class`. OLAP gets **one** of the twelve (`FulgoraUtil.FULGORA_STRATEGIES` is `JanusGraphLocalQueryOptimizerStrategy` alone). Reading the full `applyPrior`/`applyPost` graph produces the fact Part V turns on: **`JanusGraphStepStrategy` and `JanusGraphLocalQueryOptimizerStrategy` have no direct ordering edge**, and the only paths between them run through `JanusGraphMixedIndexCountStrategy` and `JanusGraphMixedIndexAggStrategy`, which a user disables with one `withoutStrategies(...)` call. No traversal was found where the order is observable, so it is filed as a missing declaration (candidate 61), not a defect. Second: **`local()` is the only construct that gets an order or a range to the storage layer.** Pass 1 folds a `has()` only for an edge-returning step, folds no order and no range (`:106`: *"We cannot fold in orders or ranges since they are not local"*) and folds only a *high* limit from a following `RangeGlobalStep` **without removing it**; pass 5 calls `foldInHasContainer`, `foldInOrder` **and** `foldInRange`, sets `setBatchSize(txVertexCacheSize)`, and `unfoldLocalTraversal` deletes the `LocalStep` entirely when its child collapses to one step. A vertex-centric query folds **at most one** order key (`:423-425`); a graph-centric one folds any number. **Pass 1 never calls `setBatchSize` at all** — handed to lesson 24, and answered there. Third, lesson 22's profiler debt: **the attachment order on the normal path is safe**, but `buildGlobalGraphCentricQuery()` **memoizes**, and `JanusGraphMixedIndexAggStep`'s constructor calls it during strategy application, before TinkerPop's `setMetrics`, so a traversal ending in `count()`/`min()`/`max()`/`sum()`/`mean()` is **compiled under `QueryProfiler.NO_OP`, has its index selection done at strategy time, and shows no `constructGraphCentricQuery` span in `.profile()`** when the push-down is declined — the common case with no mixed index. The probe also mutates the step it declined to replace (`hasLocalContainers.put(null, {[]})`); `toString()`, `hashCode()` and the or()-fallback switch were traced and none changes behaviour (candidate 62). Fourth, the second debt is answered **no**: the TinkerPop side never produces a multi-sub-query `ElementQuery`; `or()` is folded only for a `JanusGraphStep` (`:185`), each branch flattened into a `List<HasContainer>` — with an `order().by(k)` inside a branch becoming `has(k, neq(null))` — and then executed either as **one** disjunctive `GraphCentricQuery` or as **N** single-sub-query ones merged **in the step**. So candidates 55 and 56 stand and the right PR is a **comment**, not a deletion. Fifth, the third debt: `search_code` found **nothing** in the repository asserting profile-tree shape, so the `AND-query`/`OR-query` flattening is not a tested contract and candidate 60 must be filed before any new-span patch. Also: `validFoldInOrBranch` is **all-or-nothing across a whole `or()`** (candidate 64), and `AdjacentVertexHasUniquePropertyOptimizerStrategy` is the one strategy permitted to touch the backend, gated on `optimizerBackendAccess()`, which it uses to **run a side traversal during strategy application**.

- **22 — 2026-11-20** · [`2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md`](2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md) · **Findings.** Paragraph compressed 2026-11-23 under the roll-off rule, and the `Verified by lesson 22` block removed in the same run; the full versions are in git history at the 2026-11-20 through 2026-11-22 commits, and the facts lessons 26, 27 and 28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the in-memory filter sits between the backend limit and the element count**, so `limit(10)` bounds what the caller receives and not what is read, and the backend limit **doubles** — `L, 2L, 4L …` with a full cursor-less re-read each time — until enough survive; **`QueryProcessor` creates no profiler span at all**, so merge, sort and filter time is invisible in `.profile()`; half the class is dead (`VertexCentricQuery.isSorted()` hard-coded `true`, `GraphCentricQuery.numSubQueries()` hard-coded `1`); and the single-slice fitted sorted vertex-centric case does not use `QueryProcessor` at all. Nine PR candidates, **52-60**.

- **21 — 2026-11-19** · [`2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md`](2026/11/2026-11-19-janusgraph-graph-centric-query-builder/README.md) · **Findings.** Paragraph compressed 2026-11-22 under the roll-off rule, and the `Verified by lesson 21` block removed in the same run; the full versions are in git history at the 2026-11-19 through 2026-11-21 commits, and the facts lessons 25–28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the two planners are the same kind of function and a different shape** — `GraphCentricQueryBuilder` consults no statistics either, but solves a **weighted set cover** with a lexicographic objective (more clauses covered, then *fewer* indexes, then higher total score), may choose several indexes, and may choose none, which is the full scan. Index selection sits behind a swappable SPI (`IndexSelectionStrategy`), the default `threshold-based` strategy admits `2^10 = 1024` candidate groups per compiled query, a composite index is all-or-nothing and excluded outright under a top-level `Or`, and execution is stream-one/intersect-the-rest with the non-first sub-queries materialised **eagerly and unbounded**. Six PR candidates, **46-51**.


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

- [x] 25 — `ManagementSystem` and the index lifecycle: INSTALLED → REGISTERED → ENABLED, and reindex jobs
- [ ] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30. Lesson 25 opened Part VI; three lessons remain.**

**Re-pin: NOT re-raised, correctly.** Lesson 24 performed the Part V/VI review on 2026-11-22 and the pin
stays for the fifth time. Lessons 25, 26 and 27 are mid-Part and must NOT re-raise it. The next review
is the Part VI boundary — after lesson 28, or at the capstone. Lesson 25 did re-verify the two large
files it cites by line number by blob SHA (`ManagementSystem.java` 86,864 bytes = `a722a7d9`,
`ManagementLogger.java` 24,707 bytes = `1980d1fb`), both matching what the API reported, which is
incidental confirmation that the pin has not moved under the track.

**THE fact to carry forward from lesson 25: most indexes never enter the lifecycle at all, and one
expression decides it.** `canIndexBeEnabled` — `oneNewKey || (constraint != null && constraint.isNew())`
at `ManagementSystem:852` for a composite index, `type.isNew() || any sortKey isNew` at `:365` for a
vertex-centric one — makes the index `ENABLED` at birth (`:860` / `:367`) and skips `REGISTER_INDEX`
entirely (`:877` / `:389-391`). It is sound because a composite index is all-or-nothing over its fields,
so a field key created in this management transaction means **no pre-existing element can qualify**.
Lessons 26–28 must not describe the lifecycle as if every index goes through it.

**Second, and the structural fact lesson 26 needs before it opens `ElasticSearchIndex`: a mixed index's
index-level status is a constant and means nothing.** `createMixedIndex:758` sets the index vertex to
`SchemaStatus.ENABLED` **unconditionally**. The real statuses are **per field**, stored as the **last
`Parameter` of each `INDEX_FIELD` schema edge**, written once by `addIndexKey:791`
(`key.isNew() ? ENABLED : INSTALLED`) and changed by `setStatusEdges:1343-1361`, which **removes and
re-adds the edge** rather than updating it, carrying `MAPPED_NAME` — the backend field name — along in
the copy. `JanusGraphIndexWrapper.getIndexStatus(key)` hides the split by **ignoring its key argument on
a composite index**. A mixed index is therefore not one lifecycle but N, one per `addIndexKey` call.

**Third, lesson 23's debt (c) is settled: which status a reindex relies on.** Composite and
relation-type: the **vertex** status, tested against `SchemaAction.REINDEX.getApplicableStatus()` in
`IndexRepairJob.validateIndexStatus`. Mixed: the **field** statuses, in the same method's else-branch,
which additionally **skips `DISABLED` fields explicitly** (`status != SchemaStatus.DISABLED && !acceptable`)
and fails only if *no* field is acceptable. So a partly-disabled mixed index reindexes; a composite index
has no partial mode. Debts (a) and (b) are also settled: **every** vertex-centric index's backing
relation type is invisible and is not a system type (`buildRelationTypeIndex:368` calls
`maker.invisible()`), and the `RELATIONTYPE_INDEX` schema edge is written at `:387`.

**Fourth, debt (d) is NOT settled and is handed to lesson 26.** What
`AdjacentVertexHasUniquePropertyOptimizerStrategy` does with an index whose status is not `ENABLED`:
`IndexSelectionUtil` was **not located** at the pinned commit by the one `search_code` query lesson 25
spent on it, and the lesson declined to guess rather than invent a status filter. Lesson 26 is in
index-selection code anyway and should settle it. Starting point: `IndexSelectionUtil.existsMatchingIndex`
and whatever `JanusGraphIndex`-level predicate it applies.

**Fifth, the operational fact worth more than the rest put together: the INSTALLED → REGISTERED
transition is a callback in the issuing JVM's heap and nothing persists it.** `REGISTER_INDEX` sets
`INSTALLED` and arms an `UpdateStatusTrigger` in `updatedTypeTriggers` (`:1026-1032`); `commit` hands the
list to `ManagementLogger.sendCacheEviction` (`:256-257`, **after** `transaction.commit()` at `:253`);
the `EvictionTrigger` constructor snapshots `getOpenInstancesInternal()` as the acknowledgement roster;
each receiving instance queues a `SendAckOnTxClose` that waits for **every transaction that was open when
the message arrived** to close before acking; and only when the roster empties does `runTriggers()` call
`UpdateStatusTrigger.call()`, which opens a **new** management transaction and writes `REGISTERED` —
but only for elements still `INSTALLED`, so a concurrent `DISABLE_INDEX` supersedes it. Lose the issuing
JVM mid-handshake and the index is stranded at `INSTALLED` with no recovery path in the code; re-issuing
`REGISTER_INDEX` works because `INSTALLED` is in its applicable set, and nothing documents that.

**Sixth, the unfinished business, now two lessons old: none of lesson 24's three Hands-On experiments
has been run, and none of lesson 25's four has either.** Lesson 24's Hands-On A is the only direct
evidence for the batch-size paragraph below; its Hands-On B is what turns candidate 65 from a reading
into a reportable defect, and **candidate 65 must not be filed upstream until B has been run**. Lesson
25's Hands-On A (three console blocks on `inmemory`) settles `canIndexBeEnabled` and is the cheapest
unrun experiment in the track.

**Carried from lesson 24, unchanged: the batch size of a `JanusGraphVertexStep` is written twice and on
the default configuration the second write wins.** Default field value `Integer.MAX_VALUE`; pass 5 of
`JanusGraphLocalQueryOptimizerStrategy` writes `txVertexCacheSize`; `JanusGraphMultiQueryStrategy` then
writes 2500 over it because `boolean barrierSizeSet = !limitedBatch` (`:179`) and `query.batch.limited`
defaults to `true`. **So `cache.tx-cache-size` has no effect on batching on a default graph**, and the
pass-1/pass-5 asymmetry is only observable at `query.batch.limited=false`. Do not re-derive it and do not
state the asymmetry without the `limited` qualifier. Also settled and not to be re-searched: all four of
`QueryProfiler`'s `multi` / `multiPreFetch` / `vertices` / `partitioned` annotations are set in
`MultiVertexCentricQueryBuilder` and nowhere else, `vertices` is the batch size actually used, and
`JanusGraphAssert.queryProfilerAnnotationIsPresent` matches the literal string `"<annotation>=true"` so
it **cannot** assert on a numeric one.

- **Lesson 26 (`ElasticSearchIndex`) inherits five.** (a) the four feature flags —
  `supportNotQueryNormalForm()`, `supportsOrderingListProperty()`, `supportsExistsQuery()` and
  `supportsOrdering(KeyInformation)` (added by `18594751`, fixes #5120) — with lesson 23's caller for the
  second: `HasStepFolder.validJanusGraphOrder` (`:102-143`) consults
  `graph.getIndexSerializer().allMixedIndexBackendSupportsOrderingListProperty()` to decide whether a
  **vertex** order on a non-`SINGLE` key is foldable at all, so that flag changes query *compilation*.
  (b) `b7c14c53` added `index.[X].solr.max-page-size` (default 10,000) and a page-doubling read in
  `SolrIndex`; compare it with `LimitAdjustingIterator`'s doubling and say whether `ElasticSearchIndex`
  has an equivalent. (c) debt (d) above. (d) **new from lesson 25:** `IndexSerializer.register(...)` is
  called synchronously from `addIndexKey:802` and `IndexSerializer.clearStore(...)` synchronously from
  `updateIndex`'s `DISCARD_INDEX` mixed arm (`:1100-1112`) — both inside the management transaction, both
  wrapping `BackendException` into a user-facing message. Say what each does on the ES side and whether
  either can leave the schema and the index backend disagreeing on failure. (e) **also new:** the four
  `mixed-index-*` scan metrics in `IndexRepairJob` (`mixed-index-flushes`, `mixed-index-flushed-docs`,
  `mixed-index-flush-time-ms` rounded not truncated, plus `doc-updates`) and
  `MIXED_INDEX_REINDEX_BATCH_SIZE` / `MIXED_INDEX_REINDEX_BATCH_ENABLED`, which also govern
  `MixedIndexStaleEntryRemover`.
- **Lesson 27 (CQL) inherits four, three unchanged.** The `setDirectColumnByStartOnlyAllowed` flag from
  lesson 20; the feature-builder classification lesson 18 owed (whether `CQLStoreManager` and
  `HBaseStoreManager` set `.locking(...)`, and so whether `ConsistentKeyLocker` runs on them); from
  lesson 16, whether the CQL `BatchStatement` ordering is explicit or implicit in client-side timestamps
  (`graph.assign-timestamp`, LOCAL, default **true**); and from lesson 22, `PreSortingIterator`'s
  `MAX_SORT_ITERATION = 1_000_000` budget — **per sub-query**, reading the whole result before throwing,
  and rejecting a complete 1,000,000-element result (candidate 53) — which combined with
  `LimitAdjustingIterator`'s cursor-less doubling re-read makes an unfitted ordered query on CQL the worst
  case in the tree. Put a number on it against a real backend.
- **Lesson 28 (transaction log, CDC, stale-index repair) inherits five.** (a) **The deletion window
  lesson 25 named, which is lesson 28's subject stated precisely:** `REINDEX` only adds and
  `REMOVE_STALE_ENTRIES` only removes, they scan **different stores** (edgestore vs graphindex), and a
  delete that lands while an index is `DISABLED` is invisible to a reindex forever. Returning a disabled
  index to trustworthy is two complete passes over the data. (b) From lesson 22: `result.isRemoved()`
  elements are dropped **eagerly** inside `PreSortingIterator` while everything else the transaction
  deleted is filtered **lazily as reached** by `getFilterIterator`, deliberately, *"so that a change which
  the caller makes while it reads the results is seen."* (c) lesson 23's `MultiDistinctOrderedIterator`
  cross-query ordering invariant, which is an `assert`. (d) from lesson 24, the same invariant one layer
  up — `JanusGraphStep:118`'s `assert queries.stream().allMatch(q -> q.getOrder().isEmpty() || q.getOrder().equals(order))`
  is the only thing between two differently-ordered branch queries and a silently mis-ordered merged
  result, and assertions are off in production; a stale mixed index is exactly the condition that produces
  it. (e) **new from lesson 25:** `REMOVE_STALE_ENTRIES` is the only action that changes no status, is
  unsupported for vertex-centric indexes, and routes mixed indexes to `MixedIndexStaleEntryRemover.submit`
  rather than a scan job.
- **Open and unresolved, deliberately not filled in.** (a) Lesson 20's
  `BasicVertexCentricQueryBuilder:711` `Preconditions.checkArgument(bestCandidate != null, …)`
  reachability question — **lesson 25 did not reach it and it now belongs to lesson 27 alone.**
  (b) Whether any production path passes a real limit to `QueryUtil.processIntersectingRetrievals`
  (candidate 51 is a reachability claim, not a confirmed defect). (c) Whether `BackendQueryHolder`'s
  duplicated `isFitted` in `hashCode` (candidate 54) can degrade anything. (d)
  `IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel colliding with
  `IndexCandidate`'s default score (inside candidate 50). (e) Lesson 23's §2.2 ordering question — is the
  `JanusGraphStepStrategy` / `JanusGraphLocalQueryOptimizerStrategy` order observable by any traversal?
  (f) From lesson 24: whether `BatchProcessingQueue.removeLast()` has any caller at all — `HasStepBatchFetcher`
  and `DropStepBatchFetcher` were not read, so it must **not** be called dead code.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, 23 ~59, 24 ~59,
  **25 ~51** (digest ~20). A large share of each is permalink URLs, which are not prose.

**Mechanical notes, carried and extended this run. Read this section before planning a run's pushes.**
`get_file_contents` on a file whose doubled output exceeds roughly 40 KB returns an over-limit error that
saves the JSON to a local file — the cheap way to read a large file, costing nothing in context. The
correct extraction is `json.loads(longest_text)["content"][-1]["resource"]["text"]`, **not** the plain
`text` value (which has a `[Resource from github at repo://…]` banner glued onto line 1). A file under
roughly 20 KB comes back inline with no line numbers and must be written to local disk to be numbered.
**The transcription can be delegated to a sub-agent and the blob SHA verified locally afterwards** —
lesson 24 did this for fourteen files, including a 137,928-byte one. **Lesson 25 found it is often
unnecessary:** a directory listing with `fields: ["name","sha","size"]` gives every file's blob SHA in one
cheap call, so a large file read through the over-limit path can be SHA-verified against that listing
without any transcription step at all. That is now the preferred method; delegated transcription is for
files the over-limit path cannot deliver.

**The real constraint on a run is OUTPUT, not input, and it is now the binding one.** Every file pushed
must be emitted in full as a tool argument; there is no server-side copy or patch. Reading a 134 KB file
costs almost nothing; *rewriting* it costs ~37 K output tokens. Lesson 24's two structural changes —
dropping the session-log entries for lessons 01–19 and splitting the register into
`tracks/janusgraph-registers-2.md` — took this file from **126 KB to ~90 KB**. **Watch the trend.** If it
passes ~110 KB again, the next structural move is to split the `Verified by lesson NN` blocks into
`tracks/janusgraph-verified.md` — they are ~45 KB of the total, they are append-and-roll-off material,
and a run would then emit two medium files instead of one large one. **That is a decision for Aryant**,
and it is flagged rather than taken, because it changes where a run looks for the facts it is told to
cite rather than re-derive. The root `README.md` (64 KB, regenerated in full every run) is the second
such file; its index is one row per line, so a run should edit the row in place rather than rebuild the
table by re-reading every entry's `entry-meta`.

**Mermaid.** Render every diagram with `mmdc` before pushing. Lesson 25's three (two in the lesson, one
in the digest) needed one fix: **a semicolon inside a `sequenceDiagram` message is a statement separator**
and produces `Expecting … got 'NEWLINE'`. Use a comma. The lesson-19 rule still holds otherwise: quote
every node label containing parentheses, commas, `->`, `|` or angle brackets, use HTML entities such as
`&lt;` for comparisons, and never escape a quote inside a label — drop the inner quotes instead.

**Daily Diff procedure — REPLACED this run. The site has a markdown endpoint and it is ahead of the HTML.**
`https://tdd.cat/llms.txt` documents endpoints the HTML site does not advertise: `llms-full.txt`, `/md`
and `/json` for the latest edition, **`https://tdd.cat/{YYYY-MM-DD}.md` for any edition by date**,
`/rss.xml` and `/stats/`. On 2026-11-23 the front page served **2026-11-21 / 47 stories**, `/archive/`
reported *146 Editions* with `Latest >` at `/2026-11-21/` and no link for 11-22, and
`https://tdd.cat/2026-11-22/` returned `client_error` — while **`https://tdd.cat/2026-11-22.md` returned
HTTP 200** with a complete, correctly dated 57,157-character edition of 38 items. That is a fifth
disagreement shape and the first where the tiebreaker is a different *endpoint* rather than a different
page. The `.md` form is also strictly better to read: 57 KB against the HTML's 84 KB for the same content,
no chrome, and it fits in two fetches, so lesson 25's digest covered **all 38 items verbatim** where
recent digests covered roughly the first 21 of 47. **New procedure: (a) fetch
`https://tdd.cat/{today}.md`; (b) on a client error walk back one day at a time; (c) check this session
log in both directions as a floor and a ceiling; (d) use the front page and `/archive/` only to
cross-check the story count, which the `.md` file does not state; (e) only then the uncovered-item
fallback.** `https://tdd.cat/2026-11-23.md` did return a client error, so today's edition genuinely had
not published. **2026-11-20 (50 stories) remains uncovered** and stays in the fallback pool alongside the
unspent items from 2026-11-06 (two) and 2026-11-18 (two). Edition sizes remain volatile: 22, 50, 63, 58,
66, 47, 39, 49, 37, 57, 61, 61, 50, 47, 38 across 11-08 to 11-22. **Constraint still in force: there is
no `WebFetch` tool in this environment** — pages are fetched through a sub-agent which flags which parts
are verbatim and which are an extracted summary, and the summary portion is not reliable enough to quote.
A PDF or any page over ~40 K characters splits the same way, so plan a deep dive around the readable half;
the `.md` endpoint is valuable precisely because a second fetch at an offset covers the remainder.

### Verified by lesson 25, for lessons 26, 27 and 28 to cite rather than re-derive

- **`canIndexBeEnabled` is the whole gate on the lifecycle.** Composite: `oneNewKey || (constraint != null
  && constraint.isNew())` (`ManagementSystem:852`), where `oneNewKey` is set in the key loop at `:841-846`
  and is true if **any one** indexed key has `isNew()`. Vertex-centric: `type.isNew() || Arrays.stream(sortKeys)
  .anyMatch(JanusGraphElement::isNew)` (`:365`). Both then write
  `canIndexBeEnabled ? ENABLED : INSTALLED` (`:860` / `:367`) and call `updateIndex(index, REGISTER_INDEX)`
  only in the negative case (`:877` / `:389-391`). **A mixed index has no such expression**: the index
  vertex is `ENABLED` unconditionally (`:758`) and the per-key equivalent is `addIndexKey:791`'s
  `key.isNew() ? ENABLED : INSTALLED` with `:807`'s `if (!key.isNew()) updateIndex(index, REGISTER_INDEX)`.
- **Status storage, the two shapes.** Composite and relation-type: one `SchemaStatus` as a
  `BaseKey.SchemaDefinitionProperty` tagged `TypeDefinitionCategory.STATUS`, written by
  `setStatusVertex:1328-1341` as one property remove plus one property add, with an idempotence
  short-circuit at `:1334`. Mixed: one status **per field**, the **last element** of the `Parameter[]` in
  each `INDEX_FIELD` schema edge's `SchemaDefinitionDesc` modifier, written by `setStatusEdges:1343-1361`
  as `edge.remove()` + `addSchemaEdge(...)` with a copied array. The "STATUS is last" contract is
  established in one place, `addIndexKey:785-791`, and checked only by an `assert` at `:1351` — off in
  production. `setStatus:1321-1326` dispatches on whether the key subset is empty.
- **`isApplicableStatus` throws rather than returning `false`**, so `updateIndex`'s two
  `if (!updateAction.isApplicableStatus(...)) return null;` branches (`:989-990`, `:995-996`) are
  **unreachable** (candidate 68). The mixed-index equivalent is the `keySubset` build plus
  `Preconditions.checkArgument(!keySubset.isEmpty(), …)` at `:1013-1016`, with `orderedStatuses:1310-1312`
  sorting by enum ordinal and `fieldStatuses:1315-1319` printing `key=STATUS` per field. **`keySubset` is
  also the mechanism by which an action applies to only part of a mixed index.**
- **`SchemaStatus` is persisted by ordinal** (stated in its own javadoc), which is why
  `WRITE_ONLY_ENABLED` sits sixth, after `DISCARDED`. `isStable()` is `this != INSTALLED` and is the enum's
  only behaviour. The write gate remains `== DISABLED` at `StandardJanusGraph:1125` (lesson 16) and the
  read gate `== ENABLED` for composite (lesson 21) / per-`ParameterIndexField` for mixed (lesson 23).
- **`SchemaAction`'s applicable sets, exactly.** `REGISTER_INDEX` {INSTALLED, DISABLED}; `REINDEX`,
  `REMOVE_STALE_ENTRIES` {REGISTERED, ENABLED, DISABLED, WRITE_ONLY_ENABLED}; `ENABLE_WRITE_ONLY`
  {REGISTERED, ENABLED, WRITE_ONLY_ENABLED}; `ENABLE_INDEX` {REGISTERED, DISABLED, ENABLED,
  WRITE_ONLY_ENABLED}; `DISABLE_INDEX` all five of {ENABLED, DISABLED, REGISTERED, WRITE_ONLY_ENABLED,
  INSTALLED}; `DISCARD_INDEX`, `MARK_DISCARDED` {DISABLED, REGISTERED, DISCARDED, WRITE_ONLY_ENABLED};
  `DROP_INDEX` **{DISCARDED} alone**. `DISABLE_INDEX` is the only action applicable from `INSTALLED`.
- **The reindex-to-enable step is a best-effort callback, not part of the job.**
  `IndexIdentifier.getIndexJobFinisher(graph, action)` (`:1436-1458`) runs only when
  `metrics.get(ScanMetrics.Metric.FAILURE) == 0`, opens a **new** management transaction, re-resolves the
  index **by name** through `retrieve(management)`, and calls `updateIndex(index, ENABLE_INDEX)` — catching
  every `Throwable` and logging at error. A fully successful scan can therefore leave the index
  `REGISTERED`. `isWriteOnlyEnabled:1154-1166` is what diverts to the no-argument finisher, and for a mixed
  index it is true if **any** field is `WRITE_ONLY_ENABLED`.
- **The handshake, end to end.** `commit:241-268` sends the eviction **after** `transaction.commit()` at
  `:253`. `ManagementLogger.sendCacheEviction:239-260` writes
  `[CACHED_TYPE_EVICTION][evictionId][n][typeId × n][EVICT|DO_NOT_EVICT]` to the system log;
  `nextEvictionId:235-237` runs from 1 and wraps to 1, never to `UNACKNOWLEDGED_EVICTION_ID = 0` (`:71`),
  which is the sentinel used by `sendUnacknowledgedCacheEviction:275-286` for auto-created constraints and
  `addConnection`/`addProperties`. `EvictionTrigger`'s constructor (`:306-315`) opens its own management
  transaction purely to snapshot `getOpenInstancesInternal()` (the namespaces under `REGISTRATION_NS`,
  `ManagementSystem:209-213`) as the roster. Receivers queue `SendAckOnTxClose:382-441`, which acks only
  once **every transaction open when the message arrived** has closed, re-running every
  `SLEEP_INTERVAL = 100 ms` for up to `graph.management-tx-close-wait-time`
  (`GraphDatabaseConfiguration.MANAGEMENT_TX_CLOSE_WAIT_TIME`). `runTriggers:328-339` fires from
  `receivedAcknowledgement:317-326` or from `updateState:288-296`. `removeDroppedInstances:341-357` drops
  departed instances and, under `graph.management-auto-close-stale-instances` past `ackTimeout`,
  force-closes unresponsive ones. **`UpdateStatusTrigger.call():1227-1287` re-checks before writing**: it
  requires `INSTALLED` (`:1237-1242`) or keeps only still-`INSTALLED` fields (`:1244-1254`), so a racing
  `DISABLE_INDEX` wins; its `equals`/`hashCode` are on `schemaVertexId` alone (`:1289-1299`).
- **`REINDEX` is a full edgestore scan; `REMOVE_STALE_ENTRIES` is a full graphindex scan.** The first adds
  and never removes, the second removes and never adds, and the second **changes no status** and is
  `UnsupportedOperationException` for vertex-centric indexes (`:1052-1056`). `DISCARD_INDEX` on a mixed
  index calls `IndexSerializer.clearStore` **synchronously inside the management transaction**
  (`:1100-1112`); on a composite index it is a graphindex scan job chaining `MARK_DISCARDED`.
  `numOfThreads` defaults to `Runtime.getRuntime().availableProcessors()` (`:967`, `:977`).
- **`AbstractIndexStatusWatcher` defaults**: `statuses = [REGISTERED]`, `timeout = 60 s`
  (**negative means wait forever**), `poll = 500 ms`. `GraphIndexStatusWatcher.call()` opens a management
  transaction per poll, rolls it back in a `finally`, and reads `getIndexStatus` **per field key** — which
  on a composite index returns the same vertex status N times, because
  `JanusGraphIndexWrapper.getIndexStatus` ignores its key argument there. The same per-key reading drives
  `ManagementSystem.filter:628-639`, which includes an index if **any** field's status differs from the
  filter, and `printIndexes:655-679`, which prints one row per key.

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
