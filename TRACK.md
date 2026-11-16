<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 18
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-16 at the Part III/IV boundary (lesson 18). list_commits on
  JanusGraph/janusgraph master was re-run and returns the same three commits lesson 13 saw —
  a59ae8d8 (2026-10-28) is still the tip, then 9a30ff14 (2026-10-28) and 5e3e7bc7 "JanusGraph release
  1.2.0" (2026-10-24). Nothing to re-pin to; the pin stays and Part IV's byte offsets remain
  comparable with lessons 02-18. Reviewed previously at the Part II/III boundary (lesson 13, same
  answer). NEXT REVIEW: the Part IV/V boundary, after lesson 22. Lessons 19-21 are mid-Part and must
  NOT re-run list_commits.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (PR candidates 1-38, next free 39; "Still open, and who owns it")
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
2026-11-14, and now carrying candidates 1–38 (next free number **39**; 33–38 were added by lesson 18).**
PR candidates and the "Still open, and who owns it" ledger were split out of this
file at lesson 16: this file had reached 116 KB, it is read **in full** at the start of every run, and
those two sections were ~55 KB of append-only material that matters only when choosing a contribution
or settling a debt. Nothing was dropped in the move. A run reads this file always; it reads the
registers file when its lesson is named there as an owner, or when it needs the next free candidate
number. The split is the only structural change of the 2026-11-14 run.

## Session Log

Newest first. **Session-log roll-off:** the full `**Findings.**` paragraph is kept for the three most
recent lessons only — the same window as the `Verified by lesson NN` blocks below. Lessons 09-12 were
compressed on 2026-11-13, lesson 13 on 2026-11-14, lesson 14 on 2026-11-15, and **lesson 15's paragraph
was compressed this run**; the full paragraphs remain verbatim in git history at the commit each line
names, and their substance is carried forward either in the surviving `Verified by` blocks, in the
registers file, or in the root `README.md`'s per-lesson narrative. Next run, lesson 16's paragraph is
the one that goes.

- **18 — 2026-11-16** · [`2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md`](2026/11/2026-11-16-janusgraph-locking-consistent-key-locker/README.md) · **Part III complete. Both debts lesson 17 handed this run are settled — one of them by confirming its conjecture and correcting the backend it named. Re-pin reviewed at the Part III/IV boundary and declined: `a59ae8d8` is still the tip of `master`.** Read the whole locking layer at the pinned commit: `ExpectedValueCheckingStore`, `ExpectedValueCheckingStoreManager`, `ExpectedValueCheckingTransaction`, `ConsistentKeyLocker` (whole, 638 lines, numbered locally), `AbstractLocker` (whole), `LocalLockMediator` (whole), `LockerState` (whole), `ConsistentKeyLockerSerializer` (whole), `ConsistentKeyLockStatus` (whole), `StandardLockCleanerRunnable` (whole), `StandardLockCleanerService` (whole), `PermanentLockingException`, `TemporaryLockingException`, plus `Backend` (whole, 775, numbered; cited at `:157`, `:183-202`, `:271-276`, `:292-299`, `:309-325`, `:345-346`, `:385`, `:537-541`), `BackendTransaction` (whole), `StandardJanusGraph` again (whole, 1489; cited at `:761-771`, `:899-901`, `:1002-1005`, `:1077-1097`, `:1230`), `GraphDatabaseConfiguration` again (whole, 2191; cited at `:148`, `:150-155`, `:796-804`, `:937-996`), `KCVSConfiguration` (whole), `IndexUpdate` (whole), `BufferUtil` (whole), `BaseKeyColumnValueAdapter` (whole), `InMemoryStoreManager` (whole), `BerkeleyJEStoreManager` (whole), `LockKeyColumnValueStoreTest` (whole), `InMemoryLockKeyColumnValueStoreTest` (whole), and both `locking/` directory listings. Five `search_code` queries and one doc page: [Eventual Consistency](https://docs.janusgraph.org/advanced-topics/eventual-consistency/).
  **Findings.** **THE finding: the answer to "how does uniqueness survive an eventually consistent store" is one boolean.** `ExpectedValueCheckingStoreManager`'s constructor rebuilds the backing manager's features as `new StandardStoreFeatures.Builder(storeManager.getFeatures()).locking(true).build()`, so every `hasLocking()` branch lesson 03 mapped sees **`true`** on a store with no native locking — the layer above is never told the store cannot lock, because by the time it asks, it can. `Backend:271-276` is the only decision point, and it also means `edgestore`, `graphindex` **and** `system_properties` are all wrapped on such a backend, each with its own derived `<name>_lock_` store (`LOCK_STORE_SUFFIX` at `:157`); the registry of lock backends is a **two**-entry map, `consistentkey` and `test` (`:198-202`). **Second: `acquireLock` checks nothing.** It writes a claim row and files the expected value in `expectedValuesByStore`; the read-and-compare happens once, on the first `mutate`, via `prepareForMutations()` → `checkAllLocks()` → `checkAllExpectedValues()` → `mutationStarted()`, and `isMutationStarted` makes locking a strict prefix of the transaction (a later lock is a `PermanentLockingException`), which is exactly why lesson 16's phases are ordered as they are. **Third: lesson 17's idempotence claim is confirmed, and implemented twice** — `AbstractLocker.writeLock`'s `lockState.has(tx, lockID)` early return and `storeExpectedValue`'s `containsKey` first-value-wins — and that second mechanism is what makes lesson 16's "deletions first" **load-bearing**: reverse the two passes and every hand-off of a unique value from one element to another fails, because the remembered expected value would be `null` against an existing cell. **Fourth: lesson 17's `UPDATE` conjecture HOLDS, derived from three quoted lines** (`IndexUpdate.isDeletion()` is `== DELETE` so `UPDATE` falls into pass 2 at `StandardJanusGraph:1094`; the two-argument `BackendTransaction.acquireIndexLock` hard-codes `null`; `checkSingleExpectedValueUnsafe` turns a null `ev` into `Collections.emptyList()` and compares with `Iterables.elementsEqual`) — **but lesson 17's proposed reproduction could not have worked**: it named BerkeleyJE, which sets `.locking(true)`, so `Backend` never builds the wrapper and none of this code runs there. **The in-memory backend does**: `InMemoryStoreManager` never calls `.locking(...)` and does call `.keyConsistent(...)`, so **every in-memory JanusGraph exercises the whole `ConsistentKeyLocker` protocol, lock rows and 100 ms sleeps included, in a store that does not persist.** PR candidate **38**. **Byte layouts, exact:** lock key = 4-byte big-endian `key.length()` ++ key ++ column; lock column = 8-byte big-endian timestamp ++ rid (UTF-8 of `graph.unique-instance-id`); lock value = `BufferUtil.getIntBuffer(0)` = **4** zero bytes. Two javadoc defects against the default config: the protocol comment says **nanoseconds** (the provider is `TIMESTAMP_PROVIDER`, FIXED, default **MICRO**, and `fromConfig:240` overwrites `AbstractLocker`'s NANO default) and says the value is **"the single byte 0"** (it is four). `LOCK_COL_START`/`LOCK_COL_END` are 1 and **9** bytes, and the 27-line comment at `:154-180` justifies the 9 by **UTF-8**: the 9th byte is the first rid byte and `0xFF` is an illegal UTF-8 byte. **Costs, counted:** for N locked pairs, **2N + 1 extra backend transactions** (each `tryWriteLockOnce` and each `deleteSingleLock` iteration does `manager.beginTransaction` + mutate + commit), 2N extra round trips, and a **~100 ms floor** — one `sleepPast(writeTs + lockWait)` per transaction, not per lock, since `checkLocks` iterates a `HashMap` and any order ends at the same maximum. One lock promotes **every** write in the commit to `strongConsistentTx`. `KCVSConfiguration.set(key, value, expectedValue)` is the one true compare-and-set in the tree and pays all of it for N=1; `set(key, value)` and `remove(key)` take no lock at all. **Severity is inverted:** in-JVM contention is `PermanentLockingException("Local lock contention")` with no retry, cross-JVM contention is `TemporaryLockingException` from `checkSeniority` and is retried. `LocalLockMediator` has **no expiry sweeper** (entries leave only via `unlock` or by being stolen in the stale branch), and lock cleaning is **reactive and off by default** (`clean-expired=false`, 30 s hard-coded cooldown, one daemon thread, triggered only by a `checkSingleLock` that happens to read an expired claim). New PR candidates **33** (`BackendTransaction.hasAcquiredLock()` is dead — `search_code` returns exactly one hit, its own definition), **34** (`AbstractLocker.checkLocks`/`deleteLocks` increment `M_CALLS` instead of `M_EXCEPTIONS` on failure, so `locks.check.exceptions` and `locks.delete.exceptions` are never emitted and the `calls` meters double-count), **35** (`Backend:537-541` declares `edgestore_lock_` as an 8-byte fixed key for the OKVS adapter while `toLockKey` produces `4 + keyLen + colLen`; dead today because every in-tree OKVS manager sets `locking(true)`), **36** (the two javadoc defects above), **37** (`ExpectedValueCheckingStore.deleteLocks` has no null-locker guard while `checkAllLocks` explicitly skips null lockers — one of the two is dead code), **38** (the `UPDATE` conjecture). No syllabus revisions. Daily Diff: the **2026-11-15** edition (49 stories) — **Alex Miller's *Please Implement A Quorum Database***, after a `client_error` on the dated URL that succeeded on retry 13 minutes later.

- **17 — 2026-11-15** · [`2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md`](2026/11/2026-11-15-janusgraph-index-serializer-graphindex-writes/README.md) · **Lesson 16's "complete frame and one unopened box" held without correction; the box is open.** Read `IndexSerializer` (whole, 614 lines; written up for `:224-408` and `:593-612`), `IndexRecordUtil` (whole), `IndexUpdate`, `IndexUpdateContainer`, `IndexMutationType`, `IndexRecords`, `IndexRecordEntry` (all whole), `StandardJanusGraph` again (whole, 1490; cited at `:267-305`, `:518-533`, `:761-771`, `:1014-1025`, `:1080-1097`, `:1158-1206`, `:1295`), `ManagementSystem` (whole, 1815; cited at `:431` and `:831-879`), `BackendTransaction` (whole), `HashingUtil` (whole), `StaticBuffer` (whole), `StandardJanusGraphTx` again (whole, 2278; cited at `:436` only), `IndexSerializerRemoveElementTest`, `WholeRowDeletionOptimizationTest` and `WholeRowDeletionCapturingStoreManager` (all whole), plus four directory listings. One doc page: [Index Performance](https://docs.janusgraph.org/schema/index-management/index-performance/).
  **Findings.** **THE finding: the composite-index row key is deliberately unorderable and the column width *is* the uniqueness level.** `getIndexKey` is `varint(indexId)` ++ per-field `writeObjectNotNull` — the **non**-order-preserving writer lesson 12 dissected — optionally prefixed with 4 bytes of murmur, and `hashKeys` **is not a config option**: `StandardJanusGraph:304-305` passes `storeFeatures.isDistributed() && storeFeatures.isKeyOrdered()`. So the backends that could scan the graphindex store in order are exactly the ones whose keys are hashed, which is the mechanism behind the documented "composite indexes are equality-only" — a limitation the docs assert without explaining and that no planner change can lift. The column is `0x00` alone for `SINGLE`, `0x00` ++ element id for `SET`, and `0x00` ++ element id ++ one varint per field for `LIST`; uniqueness is enforced by lesson 02's one-value-per-`(key, column)` contract with **no index-side check**, and the cardinality itself is **derived** in `ManagementSystem:848-850` from `unique()` and the keys' own cardinalities, never declared. `acquireLock(CompositeIndexType, …)` excludes exactly `LIST`, the one cardinality whose column cannot collide. **`IndexMutationType.UPDATE` exists only for inline (covering) keys, is reachable from one of the three generation sites, and is written as an addition** — `IndexUpdateContainer` is a two-field state machine that keeps it only while no `ADD`/`DELETE` touches the same index key. **Refinement to lesson 16, not a correction:** "phases 1-4 touch storage only to take locks" is false of phase 3 — `indexMatches` goes through `VertexCentricQueryBuilder` for every removal (`onlyLoaded = rel.isRemoved()`) and every partitioned vertex, from a `parallelStream`, with phase-1/2 locks held; `loadLabelsCheckedByIndexes` batches only the label reads. Also: `mutateIndex` calls the **four**-argument `mutateEntries`, so lesson 16's whole-row-deletion optimisation is edgestore-only **by construction**; a deletion's value bytes exist for the lock, not the mutation; `getCompleteDocument` is a second unbatched read inside phase 6; `indexMatches` emits a full Cartesian product with no guard; and three TTL paths disagree, the composite one taking the minimum over **every field key of the record**. **One conjecture, derived and explicitly not run:** `UPDATE` reaches the addition-lock pass with expected value `null` against a cell that by construction exists — predicted failure for a `LOCK`-consistency, non-`LIST`, inline-key composite index on an inline-only property change. Handed to lesson 18 with a named confirming scenario. No new PR candidates logged (the conjecture becomes one only if lesson 18 confirms it). No syllabus revisions. Daily Diff: the **2026-11-14** edition (39 stories) — **RFC 10008, the HTTP QUERY method**, after XRAID's primaries proved unfetchable.

- **16 — 2026-11-14** · [`2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md`](2026/11/2026-11-14-janusgraph-commit-prepare-commit-order/README.md) · **All four things lesson 15 owed this run are settled, and one of them by correcting lesson 15.** Read `StandardJanusGraph` (whole, 1490 lines; written up for `:761-1470` plus the constructor fields it depends on), `StandardJanusGraphTx` again (whole, 2278; cited at `:1020-1048` and `:2115-2230`), `BackendTransaction` (whole), `CacheTransaction` (whole), `Mutation` (whole), `KCVEntryMutation` (whole), `TransactionLogHeader` (whole), `StandardVertexProperty` (whole), `GraphDatabaseConfiguration` again (whole, 2192; cited at `:255-270`, `:388-406`, `:748-800`, `:1912-1925`, `:2088-2110`), `WholeRowDeletionOptimizationTest` (whole), `WholeRowDeletionCapturingStoreManager` (whole), plus the `janusgraph-inmemory` test-package listing. Three `search_code` queries at the pinned ref (`isUpsert`, `getBufferSize`, `wholeRowDeletion` under `janusgraph-inmemory`), one `search_issues` query on this repo, and the [Failure & Recovery](https://docs.janusgraph.org/operations/recovery/) doc page.
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
2026-10-30.** Lesson 18 closed Part III exactly where lesson 17 pointed it, and the two inherited debts
are both discharged — the idempotence claim **confirmed** (two independent mechanisms, see the `Verified
by lesson 18` block) and the `UPDATE` expected-value conjecture **confirmed by reading**, with one
correction to lesson 17's method: the reproduction it proposed, on BerkeleyJE, could not have worked,
because BerkeleyJE advertises `locking(true)` and therefore never constructs
`ExpectedValueCheckingStoreManager` at all.

**THE fact to carry forward, because it changes what every later Hands-On can do cheaply: the
in-memory backend goes through the full `ConsistentKeyLocker` protocol.** `InMemoryStoreManager`'s
feature builder never calls `.locking(...)` and does call `.keyConsistent(...)`, so `Backend:271-276`
wraps it. Lock rows, the 100 ms `sleepPast`, expected-value re-reads and the 2N + 1 extra backend
transactions all happen in `janusgraph-inmemory`. **No Cassandra or BerkeleyJE is needed to observe the
locking layer, and `InMemoryGraphTest` has been exercising it since lesson 01 without anyone noticing.**
Anything in Parts IV-VI that wants to watch consistency machinery should reach for
`janusgraph-inmemory` first.

**The re-pin decision is CLOSED for now and NOT due again until lesson 22.** Lesson 18 re-ran
`list_commits` on `JanusGraph/janusgraph` `master` at the Part III/IV boundary, as lesson 17 instructed,
and got the same three commits lesson 13 saw: `a59ae8d8` (tip), `9a30ff14`, `5e3e7bc7`. Nothing to
re-pin to. **Lessons 19, 20 and 21 are mid-Part and must not re-run `list_commits`** — the next review
is the Part IV/V boundary, after lesson 22.

**The roll-off rule applied again this run.** `TRACK.md` keeps the `Verified by` block and the full
`**Findings.**` paragraph for the three most recent lessons only. The three most recent are now 16, 17
and 18, so **lesson 15's `Verified by` block is removed** (it survives verbatim in git history at the
2026-11-13 through 2026-11-15 commits) and lesson 15's session-log paragraph was compressed. Next run,
lesson 16's block and paragraph are the ones that go — but **do not drop these two facts lesson 19 and
lesson 25 were promised**, both restated in the `Verified by lesson 18` block below: `mutateIndex` goes
through `indexStore.mutateEntries`, i.e. the `KCVSCache` wrapper, while the expected-value re-read goes
through `getBackingStore()` and bypasses the cache entirely; and the lock stores are opened by
`Backend`'s locker creator directly off `storeManager`, never `storeManagerLocking` and never through a
`KCVSCache`. The **PR-candidate and open-question registers** remain in
[`tracks/janusgraph-registers.md`](tracks/janusgraph-registers.md); the roll-off rule does **not** apply
there. **Six new PR candidates were logged this run, 33-38, so the next free candidate number is 39.**

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Part IV opens with lesson 19 (`StandardVertex`/`CacheVertex`, lazy adjacency loading, the
  transaction-level and database-level caches).** Its inherited question is unchanged and now sharper.
  Lesson 17 asked whether a composite-index row is evicted or written through; lesson 18 adds that
  **`KCVSCacheInvalidationService` is built with both `getEdgeStoreCache()` and `getIndexStoreCache()`
  (`StandardJanusGraph:298-299`)**, that `BackendTransaction.mutateIndex` goes through
  `indexStore.mutateEntries` (the `KCVSCache` wrapper lesson 07 mapped), and that **two reads in the
  commit path deliberately bypass the cache**: `checkSingleExpectedValueUnsafe`'s one-column slice
  (`store.getBackingStore().getSlice(ksq, strongConsistentTx)`) and every lock-store read (the lock
  store is opened by `CONSISTENT_KEY_LOCKER_CREATOR` off `storeManager`, so it has no cache at all).
  Lesson 19 owns the edgestore/graphindex caches; it does **not** need to re-derive the locking layer.
- **Lesson 19 must NOT re-derive** lesson 18's wrapper stack, the two-transaction split, the lock row
  layout, the write/sleep/read/seniority protocol, the lock config defaults, or the `2N + 1`
  transaction count. Cite the `Verified by lesson 18` block.
- **Lesson 27 (the CQL backend) owes two answers lesson 18 could not get.** Whether `CQLStoreManager`
  omits `locking` from its `StandardStoreFeatures` (expected, but **not verified this run**), and what
  its `getKeyConsistentTxConfig()` actually sets — the consistency level that the lock-held promotion
  in `ExpectedValueCheckingStore.mutate` switches to is the entire safety argument, and lesson 18 saw
  only the mechanism, never the value. Lesson 28 (stale-index repair) should also note that
  `ExpectedValueCheckingTransaction.commit()` releases locks **between** the data commit and the lock
  transaction's commit, and `rollback()` releases them first.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78 (a large share of each is permalink URLs, which are not
  prose). The old 5-9 KB budget was a `contributor-reading` constraint and does not apply.
  **Mechanical notes, carried and extended this run:** a single `create_or_update_file` call carrying
  ~87 KB succeeds; `get_file_contents` on a file whose *doubled* output exceeds roughly 40 KB returns
  an over-limit error that *saves the JSON to a local file* — note that the tool returns the file
  content **twice** (once inline, once as a resource), so the trigger threshold is about half the
  apparent file size, and a ~28 KB source file is enough. That saved-payload path is the cheap way to
  read a large source file (`python3`/`nl`/`sed` on the saved JSON) **and the only way to get
  trustworthy line numbers** — a file returned inline has no numbering. Lesson 18 numbered four files
  locally (`ConsistentKeyLocker` 638, `Backend` 775, `StandardJanusGraph` 1489,
  `GraphDatabaseConfiguration` 2191) and reserved `#Lstart-Lend` fragments for exactly those,
  citing everything else by member name with whole-file permalinks, and said so in its Sources
  preamble. Repeat that split rather than guessing around it. `README.md` and `TRACK.md` are both far
  over the limit, so both are read to a local file and edited with a script rather than re-emitted
  from context.
- **Method note, carried; lesson 18 is the sixth entry where simulation was not needed.** Lessons 10-12
  re-implemented the pinned encoding functions and ran them; lessons 13-18 computed from the read
  sources instead and said so. Lesson 18's two derived claims are each labelled where they appear — the
  confirmed `UPDATE` conjecture ("derived from reading, not from a run") and the `2N + 1` transaction
  count — and each is paired with an exercise that runs real code (Hands-On A reads the Surefire XML of
  `InMemoryLockKeyColumnValueStoreTest`; B times a locked vs unlocked commit to expose the 100 ms
  `sleepPast`; C is written to **fail** if the conjecture holds). **Keep the rule that produced them:**
  compute or re-implement rather than assert, say which, and always pair it with an exercise that runs
  the real code. A Mermaid diagram should be rendered with `mmdc` before it is pushed — lesson 18's
  sequence diagram failed twice (an HTML entity in a `participant` alias, then a `;` inside a message,
  which Mermaid reads as a statement separator) and both would have shipped broken.
- **Daily Diff procedure — amended again, and this amendment is the useful one.** At ~03:35 UTC on
  2026-11-16 the front page served **2026-11-14 / 39 stories** (the edition lesson 17 digested) while
  `/archive/` marked **2026-11-15 "Latest" at 49 stories** (archive total **140**, up from 138) — the
  first time the two indexes have disagreed. `https://tdd.cat/2026-11-15/` returned a **`client_error`**
  on the first probe and then served the **complete 49-story edition on a retry at ~03:48**. **So: a
  dated-URL `client_error` is NOT evidence that an edition is unpublished when `/archive/` lists it
  with a story count — retry it later in the run before falling back to an uncovered item.** WebFetch
  caches per URL for 15 minutes, so the retry must be genuinely later, not immediate; do other work in
  between. The rest of the procedure stands: (a) fetch the front page and `/archive/` and take the
  maximum; (b) also probe the dated URL for today and yesterday directly; (c) before digesting, check
  this session log for the newest edition already covered — 13 covered 11-10, 14 covered 11-11, 15
  covered 11-12, 16 covered 11-13, 17 covered 11-14, 18 covered 11-15, so every run so far has found
  exactly one uncovered edition; (d) the uncovered-item fallback is still the last resort, and the
  2026-11-06 edition still has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). Edition
  sizes remain volatile — 22, 50, 63, 58, 66, 47, 39, 49 across 11-08 to 11-15 — so a small edition is
  not evidence of a partial publish. At 49 stories the page fit one WebFetch. Two further notes from
  this edition: it contained several **near-duplicate pairs** (two entries on the same dormant-neuron
  backdoor result) and one item repeated from the previous edition (Jane Street's OxCaml DSL post), so
  a story count is not a count of distinct items.

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
