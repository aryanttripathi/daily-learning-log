<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 20
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-16 at the Part III/IV boundary (lesson 18). list_commits on
  JanusGraph/janusgraph master was re-run and returned the same three commits lesson 13 saw —
  a59ae8d8 (2026-10-28) is still the tip, then 9a30ff14 (2026-10-28) and 5e3e7bc7 "JanusGraph release
  1.2.0" (2026-10-24). Nothing to re-pin to; the pin stays. Lessons 19 (2026-11-17) and 20 (2026-11-18)
  did NOT re-run list_commits — both are mid-Part, as instructed — but each got a free cross-check:
  lesson 19 measured Backend at 775 lines and GraphDatabaseConfiguration at 2191, identical to lesson
  18's independent counts, and lesson 20 measured EdgeSerializer at 591 lines, identical to lesson 10's
  and lesson 11's count. The pin has demonstrably not moved. NEXT REVIEW: the Part IV/V boundary, after
  lesson 22. Lesson 21 is mid-Part and must NOT re-run list_commits; lesson 22 owes the review.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (PR candidates 1-45, next free 46; "Still open, and who owns it")
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
Part IV's byte offsets stay comparable with lessons 02-18. Next review: the Part IV/V boundary, after
lesson 22; lessons 19-21 are mid-Part and should not re-run `list_commits`.**

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

**The registers live in [`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md), new on
2026-11-14, and now carrying candidates 1–45 (next free number **46**; 43–45 were added by lesson 20).**
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

- **20 — 2026-11-18** · [`2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md`](2026/11/2026-11-18-janusgraph-vertex-centric-query-builder/README.md) · **stayed on `a59ae8d8`. The count question lesson 19 handed over is answered: the number of `SliceQuery` shapes a traversal makes per vertex is a product, not a constant. Lesson 08's partitioned-read debt is fully discharged; lesson 19's `MultiSliceQueriesGroupingUtil` hand-off is traced to its single entry condition.** Read the whole vertex-query package at the pinned commit: `BasicVertexCentricQueryBuilder` (whole, 973 lines, numbered locally; cited at `:205-212`, `:325-334`, `:336-342`, `:344-367`, `:376-409`, `:498-517`, `:537-544`, `:546-731`, `:733-774`, `:787-801`, `:816-847`, `:849-929`, `:960-969`), `BaseVertexCentricQueryBuilder`, `VertexCentricQueryBuilder`, `MultiVertexCentricQueryBuilder`, `SimpleVertexQueryProcessor`, `BaseVertexCentricQuery`, `VertexCentricQuery`, `BasicVertexCentricQueryUtil`, `BackendQueryHolder` (all whole), plus `EdgeSerializer` again (whole, 591, numbered; cited at `:430-434`, `:436-552`, `:554-577`, `:579-588` — the line count matches lessons 10 and 11, a free confirmation the pin has not moved), `QueryUtil` (whole, 521, numbered; cited at `:57-71`, `:89-95`), `IDHandler` (whole), `InternalRelationType` (whole), `RelationTypeVertex` (whole). Tests read: `JanusGraphBaseTest` (whole, 764, numbered; `evaluateQuery` at `:589-659`), `JanusGraphTest` (whole, 12,258, numbered; cited at `:2120-2262`, `:4722-4761`, `:4818-4834`, `:5086-5106`, `:5265-5293`), `InMemoryGraphTest` (whole, for its `@Disabled` list). Three directory listings. Four `search_code` queries, of which `VertexCentricQueryTest` returned **0 hits** — there is no dedicated unit test for this compiler at all; its entire coverage lives inside `JanusGraphTest`. One doc page: [Index Performance](https://docs.janusgraph.org/schema/index-management/index-performance/).
  **Findings.** **THE finding: the vertex-centric planner is one `double` with no statistics, and its arithmetic has a crossover nobody states.** A 1-point equality on a leading sort-key position scores +5.0, a *k*-point `Contain.IN` scores +5.0/k, any range (including the `RangeInterval` that `has(key)` compiles to) scores a flat +1 and then **breaks the loop**, and +3 is added when the candidate's declared sort order and key prefix satisfy the `orderBy` — which, with no `orderBy`, **every** candidate gets, making the ordering bonus a constant offset rather than a tiebreaker. So `5.0/k` crosses the flat `+1` at **k = 5**: a six-value `IN` list is scored *below* an open range on the same key, although it compiles to six tight fitted slices rather than one wide one (**PR candidate 44**, derived, paired with Hands-On B which makes a real planner choose). Second: **`score > bestScore` is strict and `RelationTypeVertex.getRelationIndexes()` yields `Iterables.concat(singletonList(this), …)` — the base relation type is always candidate #0 — so every tie resolves to the plain label slice**, which is the mechanism behind the docs' prose warning that a constraint must land on an index's *first* sort key. Third, and this is the answer lesson 19 asked for: **sub-queries per vertex = Σ over named types of ( dirs × ∏ |points| over the leading run of point-interval sort-key positions )**, with dirs ∈ {1,2} and an `isUnbounded`-BOTH collapse forcing 1; `JanusGraphTest` pins 1, 2 and 3 at `:4829-4834` and 2 at `:5267-5268`. Crossing one is the switch that changes machinery — `bq.getQueries().size() == 1` picks `executeMultiQuery`, anything more picks `executeMultiSliceMultiQuery`, i.e. the grouping tree the db-cache rewrites in place. Fourth: **direction is the low bit of the varint *inside* the relation type**, not a level above it, which is why an untyped `direction(OUT)` query is unfitted (`:5290-5293` asserts the pair three lines apart), why `sliceLimit *= 2` is arithmetic rather than a heuristic, and why a typed BOTH query is two slices. Fifth: **`has(key)` is not "no constraint"** — `intersectConstraints` returns `new RangeInterval<>()` for it, so it occupies a slot in `intervalConstraints` and makes the collapsed BOTH form **unfitted** (`:2247-2248`). Two more candidates: **43** (`getAllRelationsQuery()` allocates a `SliceQuery` per call where the sibling `ALL_PROPERTIES_SLICE` in the same package is a `static final`, with an in-tree comment saying why) and **45** (`BackendQueryHolder.hashCode()` is `Objects.hash(backendQuery, isFitted, isFitted)` — `isFitted` hashed twice, `isSorted` never, while `equals` compares both). Candidates 43-45 logged; next free number **46**.

- **19 — 2026-11-17** · [`2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md`](2026/11/2026-11-17-janusgraph-vertex-caches-lazy-adjacency/README.md) · **Part IV opens. Lesson 18's inherited question is answered in full — invalidation *marks* a row key, it does not evict and it does not write through — and the release turns out to be a `CountDownLatch(5)`, not a clock. Three debts from lessons 15 and 16 discharged; lesson 08's partitioned-read debt half-discharged and explicitly handed to lesson 20.** Read the whole vertex and cache layer at the pinned commit: `AbstractVertex`, `StandardVertex`, `CacheVertex` (all whole), `VertexCache`, `CaffeineVertexCache`, `EmptyVertexCache` (all whole), `KCVSCache`, `ExpirationKCVSCache`, `NoKCVSCache`, `CacheTransaction` (all whole), `StandardTransactionBuilder` (whole), plus `StandardJanusGraphTx` again (whole, 2277 lines, numbered locally; cited at `:206`, `:230-268`, `:345-365`, `:486-496`, `:526-607`, `:609-665`, `:1428-1467`, `:1562-1573`, `:2091-2103`, `:2200-2210`, `:2252-2276`), `Backend` again (whole, 775, numbered; cited at `:162-165`, `:257`, `:344-381`, `:768-774`) and `GraphDatabaseConfiguration` again (whole, 2191, numbered; cited at `:610-698`, `:2055-2066`) — both line counts identical to lesson 18's independent counts, a free confirmation that the pin has not moved. Tests read whole: `CacheVertexRefreshTest`, `ExpirationCacheTest`. Four directory listings. Five `search_code` queries, of which `hasValidateKeysOnly` (**3 hits, repo-wide**) is the basis of the dead-branch finding. One doc page: [`docs/operations/cache.md`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/docs/operations/cache.md). **Findings.** (1) **Invalidation granularity is the row key and that is the only implemented path:** `hasValidateKeysOnly()` returns `true` and is overridden nowhere, so the `else` branch of `CacheTransaction.flushInternal` — which builds a `List<CachableStaticBuffer>` from the mutation's entries — is unreachable, and `ExpirationKCVSCache.invalidate`'s precondition is a tautology (**PR candidate 39**). (2) **An invalidated row is uncacheable, not merely uncached:** while the key sits in `expiredKeys`, every `getSlice` takes the `isExpired` branch, reads the store and does **not** repopulate, on all three read paths, enforced by the `key -> null` sentinel from `fillResultAndReturnMisses`; release needs `PENALTY_THRESHOLD = 5` latch countdowns plus `getAge >= db-cache-clean-wait`, and `ExpirationCacheTest.testGracePeriod`'s `minCleanupTriggerCalls = 5` is that constant verbatim. (3) **`db-cache-time = 0`, the setting the docs recommend for a single instance, makes the invalidation *mark* eternal too** — `getExpirationTime()` uses the same `cacheTimeMS`, so `isExpired`'s self-healing branch can never fire and release falls entirely to a 1-in-1000 probabilistic cleanup. (4) **Cleanup is an `O(cache size)` key-set scan** under `synchronized`, with no key → `KeySliceQuery` index (**PR candidate 40**). (5) **Invalidation runs after `persist`**, so a `persist` that throws mid-chunk skips the whole invalidation loop and `clear()`, leaving pre-mutation rows cached for up to `db-cache-time` (**PR candidate 41**, derived from reading, marked *do not file until observed*). (6) **"Pinned in the cache" is implemented on the way out:** `CaffeineVertexCache` lets Caffeine evict by size and rescues modified/removed vertices into an unbounded `volatileVertices` from the removal listener, so `cache.tx-cache-size` bounds clean vertices only. (7) The public two-argument `CacheVertex.addToQueryCache` **skips the refresh-counter guard** and its one caller is the graph-index `vertexIDConversionFct` path (**PR candidate 42**). (8) `refresh()` has exactly one production caller chain — `ManagementSystem`/`ManagementLogger` → `tx.expireSchemaElement` → `sv.refresh()` — so a user-data `CacheVertex` is never refreshed by JanusGraph itself. Candidates 39-42 logged; next free number **43**.

- **18 — 2026-11-16** · [`2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md`](2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md) · **Part III complete. Both debts lesson 17 handed this run are settled — one of them by confirming its conjecture and correcting the backend it named. Re-pin reviewed at the Part III/IV boundary and declined: `a59ae8d8` is still the tip of `master`.** Read the whole locking layer at the pinned commit: `ExpectedValueCheckingStore`, `ExpectedValueCheckingStoreManager`, `ExpectedValueCheckingTransaction`, `ConsistentKeyLocker` (whole, 638 lines, numbered locally), `AbstractLocker` (whole), `LocalLockMediator` (whole), `LockerState` (whole), `ConsistentKeyLockerSerializer` (whole), `ConsistentKeyLockStatus` (whole), `StandardLockCleanerRunnable` (whole), `StandardLockCleanerService` (whole), `PermanentLockingException`, `TemporaryLockingException`, plus `Backend` (whole, 775, numbered; cited at `:157`, `:183-202`, `:271-276`, `:292-299`, `:309-325`, `:345-346`, `:385`, `:537-541`), `BackendTransaction` (whole), `StandardJanusGraph` again (whole, 1489; cited at `:761-771`, `:899-901`, `:1002-1005`, `:1077-1097`, `:1230`), `GraphDatabaseConfiguration` again (whole, 2191; cited at `:148`, `:150-155`, `:796-804`, `:937-996`), `KCVSConfiguration` (whole), `IndexUpdate` (whole), `BufferUtil` (whole), `BaseKeyColumnValueAdapter` (whole), `InMemoryStoreManager` (whole), `BerkeleyJEStoreManager` (whole), `LockKeyColumnValueStoreTest` (whole), `InMemoryLockKeyColumnValueStoreTest` (whole), and both `locking/` directory listings. Five `search_code` queries and one doc page: [Eventual Consistency](https://docs.janusgraph.org/advanced-topics/eventual-consistency/).
  **Findings.** **THE finding: the answer to "how does uniqueness survive an eventually consistent store" is one boolean.** `ExpectedValueCheckingStoreManager`'s constructor rebuilds the backing manager's features as `new StandardStoreFeatures.Builder(storeManager.getFeatures()).locking(true).build()`, so every `hasLocking()` branch lesson 03 mapped sees **`true`** on a store with no native locking — the layer above is never told the store cannot lock, because by the time it asks, it can. `Backend:271-276` is the only decision point, and it also means `edgestore`, `graphindex` **and** `system_properties` are all wrapped on such a backend, each with its own derived `<name>_lock_` store (`LOCK_STORE_SUFFIX` at `:157`); the registry of lock backends is a **two**-entry map, `consistentkey` and `test` (`:198-202`). **Second: `acquireLock` checks nothing.** It writes a claim row and files the expected value in `expectedValuesByStore`; the read-and-compare happens once, on the first `mutate`, via `prepareForMutations()` → `checkAllLocks()` → `checkAllExpectedValues()` → `mutationStarted()`, and `isMutationStarted` makes locking a strict prefix of the transaction (a later lock is a `PermanentLockingException`), which is exactly why lesson 16's phases are ordered as they are. **Third: lesson 17's idempotence claim is confirmed, and implemented twice** — `AbstractLocker.writeLock`'s `lockState.has(tx, lockID)` early return and `storeExpectedValue`'s `containsKey` first-value-wins — and that second mechanism is what makes lesson 16's "deletions first" **load-bearing**: reverse the two passes and every hand-off of a unique value from one element to another fails, because the remembered expected value would be `null` against an existing cell. **Fourth: lesson 17's `UPDATE` conjecture HOLDS, derived from three quoted lines** (`IndexUpdate.isDeletion()` is `== DELETE` so `UPDATE` falls into pass 2 at `StandardJanusGraph:1094`; the two-argument `BackendTransaction.acquireIndexLock` hard-codes `null`; `checkSingleExpectedValueUnsafe` turns a null `ev` into `Collections.emptyList()` and compares with `Iterables.elementsEqual`) — **but lesson 17's proposed reproduction could not have worked**: it named BerkeleyJE, which sets `.locking(true)`, so `Backend` never builds the wrapper and none of this code runs there. **The in-memory backend does**: `InMemoryStoreManager` never calls `.locking(...)` and does call `.keyConsistent(...)`, so **every in-memory JanusGraph exercises the whole `ConsistentKeyLocker` protocol, lock rows and 100 ms sleeps included, in a store that does not persist.** PR candidate **38**. **Byte layouts, exact:** lock key = 4-byte big-endian `key.length()` ++ key ++ column; lock column = 8-byte big-endian timestamp ++ rid (UTF-8 of `graph.unique-instance-id`); lock value = `BufferUtil.getIntBuffer(0)` = **4** zero bytes. Two javadoc defects against the default config: the protocol comment says **nanoseconds** (the provider is `TIMESTAMP_PROVIDER`, FIXED, default **MICRO**, and `fromConfig:240` overwrites `AbstractLocker`'s NANO default) and says the value is **"the single byte 0"** (it is four). `LOCK_COL_START`/`LOCK_COL_END` are 1 and **9** bytes, and the 27-line comment at `:154-180` justifies the 9 by **UTF-8**: the 9th byte is the first rid byte and `0xFF` is an illegal UTF-8 byte. **Costs, counted:** for N locked pairs, **2N + 1 extra backend transactions** (each `tryWriteLockOnce` and each `deleteSingleLock` iteration does `manager.beginTransaction` + mutate + commit), 2N extra round trips, and a **~100 ms floor** — one `sleepPast(writeTs + lockWait)` per transaction, not per lock, since `checkLocks` iterates a `HashMap` and any order ends at the same maximum. One lock promotes **every** write in the commit to `strongConsistentTx`. `KCVSConfiguration.set(key, value, expectedValue)` is the one true compare-and-set in the tree and pays all of it for N=1; `set(key, value)` and `remove(key)` take no lock at all. **Severity is inverted:** in-JVM contention is `PermanentLockingException("Local lock contention")` with no retry, cross-JVM contention is `TemporaryLockingException` from `checkSeniority` and is retried. `LocalLockMediator` has **no expiry sweeper** (entries leave only via `unlock` or by being stolen in the stale branch), and lock cleaning is **reactive and off by default** (`clean-expired=false`, 30 s hard-coded cooldown, one daemon thread, triggered only by a `checkSingleLock` that happens to read an expired claim). New PR candidates **33** (`BackendTransaction.hasAcquiredLock()` is dead — `search_code` returns exactly one hit, its own definition), **34** (`AbstractLocker.checkLocks`/`deleteLocks` increment `M_CALLS` instead of `M_EXCEPTIONS` on failure, so `locks.check.exceptions` and `locks.delete.exceptions` are never emitted and the `calls` meters double-count), **35** (`Backend:537-541` declares `edgestore_lock_` as an 8-byte fixed key for the OKVS adapter while `toLockKey` produces `4 + keyLen + colLen`; dead today because every in-tree OKVS manager sets `locking(true)`), **36** (the two javadoc defects above), **37** (`ExpectedValueCheckingStore.deleteLocks` has no null-locker guard while `checkAllLocks` explicitly skips null lockers — one of the two is dead code), **38** (the `UPDATE` conjecture). No syllabus revisions. Daily Diff: the **2026-11-15** edition (49 stories) — **Alex Miller's *Please Implement A Quorum Database***, after a `client_error` on the dated URL that succeeded on retry 13 minutes later.

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
2026-10-30.** Lesson 20 opened the query-compilation half of Part IV and answered, in full, the counting
question lesson 19 deliberately left: **the number of `SliceQuery` shapes a vertex-centric traversal
makes per vertex is a product** — `Σ over named types ( dirs × ∏ |points| over the leading run of
point-interval sort-key positions )`, with `dirs ∈ {1,2}` for an edge label and the `isUnbounded`-BOTH
collapse forcing 1.

**THE fact to carry forward, because it decides which machinery runs: crossing one sub-query is a
switch, not a gradient.** `bq.getQueries().size() == 1` sends both `VertexCentricQueryBuilder.execute`
and `MultiVertexCentricQueryBuilder.execute` to `tx.executeMultiQuery`; anything greater sends them to
`tx.executeMultiSliceMultiQuery`, which is the **only** door into the grouping tree that lesson 19 found
`ExpirationKCVSCache.getMultiSlices` rewrites in place. A traversal with a single equality constraint
never touches it; a three-value `Contain.IN` always does. Lesson 22 should not describe the multi-slice
path as a general optimisation — it is reached by exactly this condition.

**Second fact, and it is the one to be careful about when writing lesson 21: the vertex-centric planner
has no statistics at all.** The score is a function of schema and query text only: +5.0 per 1-point
equality on a leading sort-key position, +5.0/k for a k-point `IN`, a flat +1 for any range and then
`break`, +3 when the candidate's declared sort order and key prefix satisfy the `orderBy`. With no
`orderBy` **every** candidate collects the +3, so it is a constant offset, not a tiebreaker. `score >
bestScore` is strict and `RelationTypeVertex.getRelationIndexes()` yields the base relation type first,
so **every tie resolves to the plain label slice**. Derived from the same two lines: `5.0/k` crosses the
flat `+1` at **k = 5**, so a six-value `IN` is ranked *below* an open range on the same key (**PR
candidate 44**). Lesson 21 should state plainly whether `GraphCentricQueryBuilder` is the same kind of
function or a different one; do not assume symmetry.

**Third, and it is a layout fact that keeps paying: direction lives inside the relation type.**
`IDHandler.writeRelationType` packs `strippedId = (strippedCount << 1) + dirID.getDirectionInt()`, so
the direction bit is the **low bit of the varint payload**, below the 3-bit visibility/category prefix.
A prefix slice therefore cannot select a direction — which is the whole content of the untyped `fitted`
expression, the entire justification for `sliceLimit *= 2`, and the reason a typed BOTH query splits in
two. The prefix table derived this run, for lessons 25-28 to reuse: **0** system property, **1** system
edge, **2** normal visible property, **3** normal visible edge, **4** invisible non-system property,
**5** invisible non-system edge.

**The re-pin decision stays CLOSED and is next due after lesson 22.** Lesson 20 did **not** re-run
`list_commits` — it is mid-Part, as lesson 18 and 19 instructed. It did get a free cross-check:
`EdgeSerializer` measured 591 lines, identical to lessons 10 and 11's independent counts, so the pin
demonstrably has not moved. **Lesson 21 must not re-run `list_commits` either; lesson 22 is the Part
IV/V boundary and owes the review.**

**The roll-off rule applied again.** The three most recent are now 18, 19 and 20, so **lesson 17's
`Verified by` block is removed** (it survives verbatim in git history at the 2026-11-15 through
2026-11-17 commits) and lesson 17's session-log paragraph was compressed this run. Next run, lesson 18's
block and paragraph are the ones that go — but **do not drop the two facts lesson 25 and lesson 27 were
promised** by lesson 18's block, and carry forward lesson 17's `ManagementSystem.createCompositeIndex`
pointer, which lesson 25 still starts from. **Three new PR candidates were logged this run, 43-45, so
the next free candidate number is 46.**

- **The mode question is closed. Do not reopen it unprompted.** Entries from lesson 05 on write in
  `lesson` mode without re-litigating the decision, without a notice at the top, and without nagging.
  If Aryant asks for reading sessions back, flip `mode:` and the entries resume the other shape from
  the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Lesson 21 (`GraphCentricQueryBuilder`) inherits three things lesson 20 deliberately did not do.**
  (a) **The second scoring function**, to be compared against today's explicitly — in particular whether
  the graph-centric side consults any statistics where the vertex-centric side consults none.
  (b) **`QueryUtil.processIntersectingRetrievals(List<IndexCall<R>>, int limit)`** (`QueryUtil:463`),
  read this run by name only; it is the graph-centric analogue of today's sub-query merge. (c) Whether
  the strategy layer can fold a `has()` into an already-compiled vertex-centric query or must re-enter
  `constructQuery` — decided by what `GraphCentricQueryBuilder` exposes, and consumed by lessons 23-24.
- **Lesson 20 discharged, in full, the debts it was handed. Do not re-pose them.** From lesson 19:
  the sub-query count (above); the partitioned-vertex read fan-out — `allRequiredRepresentatives`
  returns `ImmutableList.of(tx.getCanonicalVertex(v))` when `hasAllCanonicalTypes()` (every named type
  is a property key or unique in `dir`, and `types.length == 0` returns `false` immediately), otherwise
  `tx.getAllRepresentatives(v, restrict2Partitions)`, so the fan-out is `|representatives| ×
  sub-queries`; and the `MultiSliceQueriesGroupingUtil` entry condition (above). **Lesson 08's
  partitioned-read debt, half-discharged by lesson 19, is now fully closed.**
- **Lesson 22 (`QueryProcessor`) inherits two.** (a) What the generic processor actually does with an
  unfitted or unsorted `BackendQueryHolder`: how `ResultMergeSortIterator` and `ResultSetIterator`
  compose the per-sub-query iterators, and whether the in-memory filter runs before or after the limit.
  (b) `LimitAdjustingIterator` proper — lesson 20 read only `SimpleVertexQueryProcessor`'s subclass and
  its `sliceQuery.getLimit() != query.getLimit()` trigger.
- **Lesson 25 (`ManagementSystem`) inherits two new ones.** (a) **Whether any shipped relation type has
  `isInvisibleType() == true` without being a system type.** `getAllRelationsQuery()` spans prefixes
  `[0, 4)` — system plus normal visible — so prefixes 4-5 are outside it, and `.all()`'s only caller is
  `MultiVertexCentricQueryBuilder.drop()`. If such types exist, `drop()` leaves their cells behind.
  (b) `mgmt.buildEdgeIndex` / `buildPropertyIndex` and the `TypeDefinitionCategory.RELATIONTYPE_INDEX`
  schema edge that `getRelationIndexes()` walks `Direction.OUT` across — lesson 20 read the candidate
  list as a schema traversal; lesson 25 owns the write side. Lesson 25 still starts at
  `ManagementSystem.createCompositeIndex:848-850`, per lesson 17.
- **Open and unresolved, deliberately not filled in.** `BasicVertexCentricQueryBuilder:711`'s
  `Preconditions.checkArgument(bestCandidate != null, …)` could not be reached by any schema lesson 20
  could construct, because the unidirected-mismatch case is already `continue`d at `:640` and the base
  relation type is always a candidate. It is **not** claimed to be dead code; it needs a reachability
  argument and would make a good small upstream question.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76 (a large share of each is permalink URLs,
  which are not prose). The old 5-9 KB budget was a `contributor-reading` constraint and does not apply.
  **Mechanical notes, carried and confirmed this run.** A single `create_or_update_file` call carrying
  ~87 KB succeeds. `get_file_contents` on a file whose *doubled* output exceeds roughly 40 KB returns
  an over-limit error that saves the JSON to a local file — the cheap way to read a large source file
  **and the only way to get trustworthy line numbers**, since a file returned inline has none. It also
  costs nothing in context, which makes it the right way to read a 12,000-line test file: lesson 20 read
  the whole of `JanusGraphTest` (649 KB) this way for the price of a preview. **The defensive parser
  lesson 19 described was needed again and worked unchanged**: take the longest `text` anywhere in the
  saved JSON, strip a leading `[Resource from github …] ` prefix, and if the result starts with
  `{"content"` parse it again. A `wc -l` of `0` is the symptom of getting it wrong. Lesson 20 numbered
  five files locally (`BasicVertexCentricQueryBuilder` 973, `EdgeSerializer` 591, `QueryUtil` 521,
  `JanusGraphBaseTest` 764, `JanusGraphTest` 12,258) and reserved `#Lstart-Lend` fragments for exactly
  those, citing everything else by member name with whole-file permalinks, and said so in its Sources
  preamble. Repeat that split. `README.md` and `TRACK.md` are both far over the limit, so both are read
  to a local file and edited with a script rather than re-emitted from context.
- **Method note; lesson 20 is the eighth entry where simulation was not needed.** Lessons 10-12
  re-implemented the pinned encoding functions and ran them; lessons 13-20 computed from the read
  sources instead and said so. Lesson 20's two derived claims — the `5.0/k` crossover and the `.all()`
  prefix range — are each labelled where they appear and each is paired with an exercise: Hands-On A
  makes the reader break a `numSubQueries` assertion on purpose so the product is *measured* by
  `JanusGraphBaseTest.evaluateQuery`'s own counter; Hands-On B adds a throwaway test and a breakpoint on
  the `if (score > bestScore)` line so the score is read off a live planner; Hands-On C breakpoints the
  `return new SliceQuery(...)` and inspects the `ASC`/`DESC` buffer swap and the `-1` sentinel in bytes.
  **Keep the rule that produced them:** compute or re-implement rather than assert, say which, and always
  pair it with an exercise that runs the real code. **Render every Mermaid diagram with `mmdc` before
  pushing** — both of this run's diagrams and the digest's were extracted from the finished Markdown and
  rendered clean on the first attempt, with the lesson-19 rule observed throughout (quote every node
  label containing parentheses, commas, `->`, `|` or angle brackets, use HTML entities such as `&lt;`
  for generics, and never escape a quote inside a label — drop the inner quotes instead).
- **Daily Diff procedure — the 2026-11-16 amendment held for a third consecutive run.** At ~03:35 UTC
  the front page served **2026-11-17 / 57 stories**, and `https://tdd.cat/2026-11-18/` returned a
  `client_error`, so today's edition had not published. The front page has now been ahead of `/archive/`
  on three consecutive runs. The procedure stands: (a) read the front page and the archive and take the
  maximum; (b) probe the dated URL for today and yesterday; (c) check this session log for the newest
  edition already covered — 13 covered 11-10, 14 covered 11-11, 15 covered 11-12, 16 covered 11-13,
  17 covered 11-14, 18 covered 11-15, 19 covered 11-16, 20 covered 11-17, so every run so far has found
  exactly one uncovered edition; (d) the uncovered-item fallback is still the last resort, and the
  2026-11-06 edition still has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). WebFetch
  caches per URL for 15 minutes, so a retry must be genuinely later — do other work in between. Edition
  sizes remain volatile: 22, 50, 63, 58, 66, 47, 39, 49, 37, 57 across 11-08 to 11-17.

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

### Verified by lesson 18, for lessons 19, 22, 25, 26, 27 and 28 to cite rather than re-derive

- **One boolean is the whole design.** `ExpectedValueCheckingStoreManager`'s constructor does
  `new StandardStoreFeatures.Builder(storeManager.getFeatures()).locking(true).build()`, so a store with
  no native locking reports `hasLocking() == true` to everything above `Backend`. `Backend:271-276` is
  the only decision point: `if (!storeFeatures.hasLocking())` → wrap, else `storeManagerLocking =
  storeManager`. Every graph store is opened through `storeManagerLocking` (`:345-346` edgestore and
  graphindex, `:385` system_properties), so on a non-locking backend all three are
  `ExpectedValueCheckingStore`s with their own `<name>_lock_` store (`LOCK_STORE_SUFFIX = "_lock_"`,
  `:157`). Exactly **two** lock backends exist: `consistentkey` and `test` (`:198-202`); anything else
  throws at `:292-299`.
- **Backend classification, read off the feature builders.** `InMemoryStoreManager` never calls
  `.locking(...)` and does call `.keyConsistent(...)` → **wrapped; the in-memory backend runs the full
  `ConsistentKeyLocker` protocol.** `BerkeleyJEStoreManager` sets `.locking(true)` → **native; the
  wrapper is never constructed and none of the `consistentkey` package runs.** CQL/HBase were not read
  this run (lesson 27 owes that).
- **One `BackendTransaction` is two store transactions.** `beginTransaction` calls the underlying
  manager twice: `inconsistentTx` (client data, no consistency guarantee) and `strongConsistentTx`
  (`getKeyConsistentTxConfig()` merged **under** the caller's own custom options, so the caller wins).
  `maxReadTime` = `storage.read-time`, default **10 s**. **The switch is per mutation and
  all-or-nothing per transaction:** `prepareForMutations()` returns `!expectedValuesByStore.isEmpty()`
  and both `ExpectedValueCheckingStore.mutate` and `...StoreManager.mutateMany` route **every** write
  to `strongConsistentTx` once any lock is held. A *failed* lock never promotes, because the throw
  precedes `storeExpectedValue`.
- **`acquireLock` checks nothing.** It rejects a lock after `isMutationStarted` with
  `PermanentLockingException("Attempted to obtain a lock after mutations had been persisted")`, calls
  `locker.writeLock(new KeyColumn(key, column), tx.getConsistentTx())`, and files the expected value.
  The check happens once, on the first `mutate`: `prepareForMutations()` → `checkAllLocks()` →
  `checkAllExpectedValues()` → `mutationStarted()`, in that order — **no expected value is read until
  every lock is confirmed held.** The `locker == null` branch is live and degrades to a pass-through.
- **Idempotence is implemented twice, and the second mechanism is what makes "deletions first"
  correct.** `AbstractLocker.writeLock` opens with `if (lockState.has(tx, lockID)) return;`
  (`LockerState` is a `MapMaker().concurrencyLevel(8).weakKeys()` map keyed on the `StoreTransaction`
  **by identity**), and `storeExpectedValue` keeps the **first** expected value and logs-and-discards
  later ones. So in a transaction that moves a unique value from A to B, pass 1's deletion lock
  (expected value = the old entry's value bytes) is remembered and pass 2's addition lock (expected
  value `null`) is dropped. **Reverse the passes and every such hand-off fails.** Covered by
  `transactionMayReenterLock`.
- **The expected-value check is a one-column slice plus an exact-match filter, and it bypasses the
  cache.** `checkSingleExpectedValueUnsafe` builds `new KeySliceQuery(kc.getKey(), kc.getColumn(),
  BufferUtil.nextBiggerBuffer(kc.getColumn()))`, reads it from **`store.getBackingStore()`** with
  `strongConsistentTx`, filters for columns that `equals` the target exactly (the slice is half-open on
  bytes, so a prefix can come back), then `Iterables.elementsEqual(expectedValues, actualValues)` where
  a `null` expected value becomes `Collections.emptyList()` — **that empty list IS the "must not exist"
  semantics, and the comparison is symmetric.** Wrapped in
  `BackendOperation.executeDirect(..., maxReadTime)` with `toString()` = `"ExpectedValueChecking"`.
- **Lock row layout, exact.** `toLockKey` = 4-byte big-endian `key.length()` ++ key ++ column
  (`WriteByteBuffer(key.length() + column.length() + 4)`). `toLockCol` = 8-byte big-endian
  `provider.getTime(ts)` ++ rid, rid being the UTF-8 bytes of `graph.unique-instance-id`. Value =
  `ConsistentKeyLocker.zeroBuf` = `BufferUtil.getIntBuffer(0)` = **4 bytes of 0x00** (verified against
  `BufferUtil.getIntBuffer`, which allocates `intSize` and asserts the length). `LOCK_COL_START =
  zeroBuffer(1)`, `LOCK_COL_END = oneBuffer(9)` = **9 bytes of 0xFF**, and the justification at
  `ConsistentKeyLocker:154-180` is a **UTF-8** argument: the 9th byte is the first byte of the rid and
  `0xFF` is not a legal UTF-8 byte, so it sorts after every real lock column. Two javadoc defects:
  the protocol comment says the timestamp is in **nanoseconds** (the provider is `TIMESTAMP_PROVIDER`,
  **FIXED, default MICRO**, and `fromConfig:240` overwrites `AbstractLocker.Builder`'s NANO default),
  and says the value is **"the single byte 0"** (four bytes).
- **The protocol, per lock.** `writeSingleLock` loops `lockRetryCount` times; a write counts as
  successful only if `wr.getDuration() <= lockWait`, so **a slow success is a failure** and is retried
  with a new timestamp while the previous column is queued for deletion **in the same `mutate` call**
  (lesson 02's additions-after-deletions rule). Each attempt runs in its **own** backend transaction
  via `overrideTimestamp` → `manager.beginTransaction(...commitTime(start)...)` → mutate → commit.
  Exhausting the retries throws `TemporaryBackendException("Lock write retry count exceeded")` —
  deliberately temporary. `checkSingleLock` returns early if `ls.isChecked()`, then
  `times.sleepPast(writeTimestamp + lockWait)`, slices `LOCK_COL_START..LOCK_COL_END`, drops claims
  older than `lockExpire` (throwing `ExpiredLockException` if the expired claim is **ours**), and runs
  `checkSeniority`: a foreign rid in front of us is `TemporaryLockingException`, our own timestamp
  found means we hold it, our own older timestamps are skipped with a warning, `trCount == 0` is
  temporary, and our-rid-but-no-matching-timestamp is `PermanentBackendException`. `deleteSingleLock`
  deletes exactly one column, again in a fresh transaction per attempt, and
  `AbstractLocker.deleteLocks` removes the local mediator entry **regardless** of whether the store
  delete succeeded.
- **Cost, counted: for N locked `(key, column)` pairs — 2N + 1 extra backend transactions, 2N extra
  round trips, and a ~100 ms floor.** The sleep is **per transaction, not per lock**: `checkLocks`
  iterates a `HashMap`'s `entrySet()` in arbitrary order and every order ends at
  `max(writeTimestamp) + lockWait`, so the `// We could be smarter about sleeping by iterating oldest
  -> latest...` TODO at `:446-447` would not change the total.
- **Config, with types.** `storage.lock.retries` MASKABLE **3**; `storage.lock.wait-time` MASKABLE
  **100 ms** (doubles as the max acceptable write latency **and** the pre-read quiet period — there is
  no third knob); `storage.lock.expiry-time` **GLOBAL_OFFLINE** **300 000 ms**;
  `storage.lock.clean-expired` MASKABLE **false**; `storage.lock.backend` **GLOBAL_OFFLINE**
  `"consistentkey"`; `storage.lock.local-mediator-group` LOCAL, no default, generated at startup;
  `graph.unique-instance-id` LOCAL, no default (hostname + pid + process-wide counter);
  `graph.timestamps` **FIXED**, **MICRO**; `storage.read-time` MASKABLE **10 s**.
  (`GraphDatabaseConfiguration:148`, `:150-155`, `:796-804`, `:937-996`.)
- **`LocalLockMediator` is tier one, and it has no sweeper.** `lock()` is a four-way branch on
  `putIfAbsent`: absent → acquired; same holder → `replace` to extend expiry; recorded expiry in the
  past → `replace` to steal; otherwise **`false`**. `AuditRecord.equals`/`hashCode` depend **only** on
  `holder`, which is what makes both the extend branch and `unlock`'s
  `locks.remove(kc, new AuditRecord<>(requester, null, null))` work. Re-entrance is not counted — one
  `unlock` suffices. Entries leave the map only via `unlock` or by being stolen, so a transaction that
  dies without rollback leaks its `KeyColumn` until someone contends for it. Mediators are
  process-global by name through `LocalLockMediators.INSTANCE`.
- **Severity is inverted.** In-JVM contention → `PermanentLockingException("Local lock contention")`,
  **no retries**, and `PermanentLockingException extends PermanentBackendException` so lesson 07's
  `BackendOperation` backoff will not retry it. Cross-JVM contention → `TemporaryLockingException
  extends TemporaryBackendException` from `checkSeniority`, and **is** retried. The suite declines to
  pin it: `testLocalLockContention` asserts only `Permanent || Temporary`.
- **Lock cleaning is reactive and off by default.** No periodic sweep exists; the only trigger is a
  `checkSingleLock` that reads an expired claim, so an expired row on a `(key, column)` nobody locks
  again is never deleted. `StandardLockCleanerService` dedups per `KeyColumn` through a Caffeine
  `expireAfterWrite` map with a **hard-coded 30 s** cooldown, on a `ThreadPoolExecutor(0, 1, 5 s,
  LinkedBlockingQueue)` of daemon `LockCleaner-%d` threads. `StandardLockCleanerRunnable` is handed the
  **caller's `StoreTransaction`** and runs on another thread, so it can touch a transaction its owner
  has already committed — the class javadoc owns this as "best-effort". Its log line says "before or at
  cutoff" while the predicate is `isBefore(cutoff)`.
- **`KCVSConfiguration.set` is the one true compare-and-set in the tree.** `set(key, value,
  expectedValue)` → `checkExpectedValue = true` → `store.acquireLock(rowKey, column,
  expectedValueBuffer, txh)` then `store.mutate(...)` **inside one `BackendOperation.execute` callable**
  (`toString()` = `"setConfiguration"`, budget `maxOperationWaitTime`, a mutable field defaulting to
  **10 000 ms**, not a config option). `expectedValueBuffer` is `null` when `expectedValue` is null even
  with the check on, so "create only if absent" is expressible. The `WriteConfiguration` override
  `set(key, value)` passes `false` and takes **no lock**, and `remove(key)` → `set(key, null)` is
  therefore **never** a compare-and-set.
- **Lesson 17's `UPDATE` conjecture HOLDS (derived from reading, not run).** `IndexUpdate.isDeletion()`
  is `mutationType == IndexMutationType.DELETE` and `isUpdate()` is separate, so an `UPDATE` is skipped
  by pass 1 and taken by pass 2 at `StandardJanusGraph:1094`, which calls the **two**-argument
  `BackendTransaction.acquireIndexLock(key, column)` — hard-coded `indexStore.acquireLock(key, column,
  null, storeTx)`. Null means "must not exist", against a cell that by construction exists (an `UPDATE`
  is generated only when an inline key changed and the field-key values did not). Firing conditions:
  composite index, `ConsistencyModifier.LOCK`, cardinality != `LIST`, ≥1 inline key, a transaction
  changing only an inline property of an existing element, **and a backend without native locking**.
  The suite tests the **mirror** case only (`expectedValueMismatchCausesMutateFailure`:
  expected-present, actually-absent); there is no test for expected-absent, actually-present.
  **PR candidate 38.**
- **Two reads in the commit path deliberately bypass `KCVSCache`** (for lesson 19): the expected-value
  slice goes through `getBackingStore()`, and the lock stores are opened by
  `CONSISTENT_KEY_LOCKER_CREATOR` off **`storeManager`**, not `storeManagerLocking`, so lock rows are
  never cached and never invalidated. `BackendTransaction.mutateIndex` still goes through
  `indexStore.mutateEntries`, i.e. the `KCVSCache` wrapper.
- **`BackendTransaction.hasAcquiredLock()` is dead** — `search_code` across the repo returns exactly
  one hit, its own definition, so the `acquiredLock` field is write-only (and is set **before** the
  `acquireLock` call that may throw). **PR candidate 33.**
- **The test suite to reach for.** `LockKeyColumnValueStoreTest` (in `janusgraph-backend-testutils`,
  `src/main`, not `src/test` — lesson 06's inversion) has **13** tests; the runnable concrete subclass
  for this track is `InMemoryLockKeyColumnValueStoreTest` in `janusgraph-inmemory`
  (`mvn -pl janusgraph-inmemory -Dtest=InMemoryLockKeyColumnValueStoreTest test`). It fakes **eight
  hosts in one JVM** by setting `LOCK_LOCAL_MEDIATOR_GROUP` to `concreteClassName + i` and
  `UNIQUE_INSTANCE_ID` to `"inst" + i`, raises `LOCK_RETRY` to **10** and drops `LOCK_EXPIRE` to
  **5 000 ms**. `testRemoteLockContention` is overridden with an **empty body** in the in-memory
  subclass and still discovered through the superclass's `@Test` — **another silent ~0 ms pass**, lesson
  06's fourth opt-out idiom. `expiredLocalLockIsIgnored`, `expiredRemoteLockIsIgnored` and
  `repeatLockingDoesNotExtendExpiration` each burn >5 s on `Thread.sleep`.
  `parallelNoncontendedLockStressTest` runs 8 x 100 lock-and-commit cycles and asserts only
  `succeeded + temporaryFailures == 100` per thread, so a run that mostly fails to lock still passes.
  `testLocksOnMultipleStores` pins the per-store locker count with EasyMock: 6 stores, locks on 4,
  exactly 4 `writeLock` and 4 `checkLocks`.

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
