<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 9
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

- **09 — 2026-11-07** · [`2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md`](2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md) · **stayed on `a59ae8d8`** (mid-Part, no decision available). Read `VertexIDAssigner` (whole, 545 lines), `StandardIDPool` (whole), `IDBlockSizer` (whole), `ConsistentKeyIDAuthority` (whole, 500 lines), `AbstractIDAuthority` (whole), `ConflictAvoidanceMode` (whole), `IDBlock` (whole), `SimpleBulkPlacementStrategy` (whole), `IDManager` L60-130 / L370-430 / L450-460 / L545-565 / L610-650 / L715-725, `GraphDatabaseConfiguration` L1040-1180, `VariableLong` L77-92, `IDAuthorityTest` L75-295, plus the test inventories of `IDPoolTest`, `VertexIDAssignerTest`, `InMemoryIDAuthorityTest`, `CQLIDAuthorityTest`, `HBaseIDAuthorityTest`, `PropertyPlacementStrategy` L95-110, `docs/changelog.md` L2041-2058, `docs/configs/janusgraph-cfg.md` L88-113, `docs/operations/bulk-loading.md` L78-91. **THE finding: `ids.authority.conflict-avoidance-tag-bits` defaults to 4, is `ConfigOption.Type.FIXED`, and its tag defaults to 0 with mode `NONE` — so `ConsistentKeyIDBlock.getId:486-491` returns `count << 4` with a zero tag and every count JanusGraph assigns is a multiple of 16.** The first `NormalVertex` id in a fresh default graph is **4096** at partition 0 (count 16), not 256; the stride between consecutive vertices in one partition is 128, not 8; and `maxAvailableBits = unsignedBitLength(idUpperBound)-1-uniqueIdBitWidth` takes the four bits straight off every namespace's ceiling, so **2^51 of 2^55 vertex counts, 2^54 of 2^58 relation counts and 2^50 of 2^54 schema counts are reachable — 15/16 of each space is not**, on every graph created with the defaults, unchangeably. Second finding: the give-up message at `:420-421` prints `timeout.getNano()`, which is **0** for any whole-second `Duration`, including the 120 s `ids.renew-timeout` default — "Reached timeout 0" on every default deployment (PR candidate 11). Third: the rollback path's anonymous `TransactionalProvider` (`:392-399`) is commented "Use normal consistency level" and builds a transaction config **character-for-character identical** to `openTx()` at `:223-225` (PR candidate 12). Fourth: `IDAuthorityTest`'s two contiguity assertions (`checkBlock:207-209`, `testSimpleIDAcquisition:247-251`) are gated on `hasEmptyUid` and **none of the three `configs()` parameterizations sets tag-bits to 0**, so neither has ever executed (PR candidate 13). Fifth: `accessed()` writes a `volatile long` via `System.currentTimeMillis()` on every element id (`:378`) for a `getLastAccess()` that has **no caller in the tree** — measured **33.0 ns/op** against 0.30 ns for an empty loop under OpenJDK 21 (PR candidate 14, bundled with the `SimpleBulkPlacementStrategy` items). **Closed lesson 08's `count == 0` question properly:** `BASE_ID = 1` (`AbstractIDAuthority:45`), so no count 0 is ever offered — `addPadding`'s refusal was never the thing protecting it. Also: five pools with `getIDNamespace() == ordinal()`, three per partition (`hasOnePerPartition()`); `renewBlockIndex = 0` for `SCHEMA` (block 50) and `PARTITIONED_VERTEX` (block 100) because `RENEW_ID_COUNT = 100` exceeds both, so those two namespaces permanently double-buffer; the negated-counter column order that makes `getCurrentID`'s `setLimit(5)` correct; the 330 ms claim and the ~9,090 vertices/s/pool it implies; and `renewalThreads = 3*partitionIdBound+2` = 98 at defaults against a live-pool ceiling of 32 (`ids.num-partitions = 10`). Daily Diff: the **2026-11-06** edition (52 stories), found via `/archive/` while the front page still served 2026-11-05 — deep dive on Infino's coordinator-free 128-bit ids, whose 40-bit random writer id is JanusGraph's 4-bit conflict-avoidance tag with 36 more bits of budget.
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
2026-10-30.** Lesson 09 is mid-Part, so the re-pin question was not available and was not raised:
**stay on `a59ae8d8`**. The next boundary where re-pinning is allowed is **Part II/III, after lesson
13**; at that point either re-pin and record the new commit in the front-matter and here, or state
explicitly that it is staying, as lesson 08 did.

**One structural revision this run, applied under this file's own roll-off rule** (the root README
states that `TRACK.md` keeps the notes for the three most recent lessons and older ones roll off into
git history). Lesson 07's `Verified by lesson 07` block is now **compressed to the facts later lessons
were promised**; the full block, with every line number and measurement, is preserved verbatim in git
history at commit `513a317b2650b49e6069eb35e3bdfb70441a913a`. Lessons 08 and 09 keep their blocks in
full because Part II lessons 10–13 cite them directly. Next run, lesson 08's block becomes the oldest
of the three and the same rule applies to it.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01–04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Part I is complete; Part II is half done.** Lessons 01–07 cover the storage contract, two store
  implementations, the adapter, the harness, and the wiring; lessons 08 and 09 cover the id layout and
  the id supply. Lessons 10–13 are serialization: `EdgeSerializer` out, `EdgeSerializer` back in,
  `StandardSerializer`, and schema persistence.
- **Entry size.** Lesson 05 ≈35 KB, 06 ≈63 KB, 07 ≈78 KB, 08 ≈67 KB, 09 ≈61 KB (a large share of each
  is permalink URLs, which are not prose). In line with the SQLite and PostgreSQL lessons. The old
  5–9 KB budget was a `contributor-reading` constraint and does not apply.

### Verified by lesson 09, for lessons 10, 16, 18 and 27 to cite rather than re-derive

- **The default conflict-avoidance tag is 4 bits wide, `FIXED`, and always zero.**
  `ids.authority.conflict-avoidance-tag-bits` = **4** (`ConfigOption.Type.FIXED`),
  `conflict-avoidance-mode` = `NONE` (`GLOBAL_OFFLINE`), `conflict-avoidance-tag` = **0** (`LOCAL`),
  `ids.authority.wait-time` = **300 ms** (`GLOBAL_OFFLINE`), `randomized-conflict-avoidance-retries` = 5.
  `ConsistentKeyIDAuthority:191/:200` therefore set `randomizeUniqueId = false`, `uniqueId = 0`, and
  `ConsistentKeyIDBlock.getId:486-491` returns `((start+i) << 4) + 0`. **Every count is a multiple of
  16.** Verified at pb=5 with `NormalVertex` (offset 3, suffix 0): count 16 → id **4096** at partition 0,
  **4232** at partition 17; counts 32/48/64 → 8192/12288/16384. At tag-bits 0 the first id would be 256.
  Consecutive vertices in one partition are **128 apart**, not 8. Lessons 10 and 27 must assume this
  when reasoning about edgestore locality and key distribution.
- **The reachable count space, exact.** `claimIDBlock:287-290`:
  `maxAvailableBits = (VariableLong.unsignedBitLength(idUpperBound) - 1) - uniqueIdBitWidth`,
  `idBlockUpperBound = 1L << maxAvailableBits`; `unsignedBitLength` is
  `64 - numberOfLeadingZeros` (`VariableLong:85-87`). At pb=5, tag-bits 4:
  `NORMAL_VERTEX`/`UNMODIFIABLE_VERTEX`/`PARTITIONED_VERTEX` **2^51** of 2^55;
  `RELATION` **2^54** of 2^58; `SCHEMA` **2^50** of 2^54. **15/16 of every space is unreachable** and
  the option is `FIXED`, so it cannot be reclaimed.
- **Five namespaces, and the namespace byte is an enum ordinal.** `PoolType.getIDNamespace()` is
  `ordinal()` (`VertexIDAssigner:458-460`): 0 `NORMAL_VERTEX`, 1 `UNMODIFIABLE_VERTEX`,
  2 `PARTITIONED_VERTEX`, 3 `RELATION`, 4 `SCHEMA`. Reordering the enum reinterprets every existing
  graph's id rows. `hasOnePerPartition()` is true for 0, 1 and 3 only — hence
  `renewalThreads = 3 * partitionIdBound + 2` (`:123`) = **98** at `cluster.max-partitions = 32`, against
  a live-pool ceiling of **32** because `ids.num-partitions` = 10; at `cluster.max-partitions = 65536`
  the formula asks for **196,610** threads (lazily created, `allowCoreThreadTimeOut(true)`).
- **Block sizes and renewal triggers, exact at `ids.block-size = 10000` and `ids.renew-percentage = 0.3`**
  (`SimpleVertexIDBlockSizer:429-446`, `StandardIDPool:213`,
  `renewBlockIndex = max(0, numIds - max(RENEW_ID_COUNT=100, round(numIds*0.3)))`):
  `NORMAL_VERTEX` 10,000 → **7,000**; `UNMODIFIABLE_VERTEX` 1,000 → **700**;
  `PARTITIONED_VERTEX` 100 → **0**; `RELATION` 80,000 (`base*8`) → **56,000**;
  `SCHEMA` **50** (a literal, unaffected by `ids.block-size`) → **0**, clamped from −50.
  **`SCHEMA` and `PARTITIONED_VERTEX` therefore fire a renewal on the first id of every block and
  permanently hold one spare block in flight** — a side effect of `RENEW_ID_COUNT` exceeding their
  block size, not a decision. Operational number: 3,000 ids of runway against a 330 ms claim is
  **~9,090 vertices/s per pool**, ~90,000/s per instance at `ids.num-partitions = 10`.
- **The claim protocol, and why it is safe over an eventually consistent store.**
  Row key (`getPartitionKey:252-261`) is two big-endian ints:
  `[(partition << (32 - partitionBitWidth)) | tag][idNamespace]` — **lesson 08's row-key rotation one
  word narrower**, non-overlapping because `partitionBitWidth <= 16` (`:172`) and
  `uniqueIdBitWidth <= 16` (`:175`) and `32-16 = 16`, exactly tight. Verified at pb=5: partition 0/ns 0
  → `0x0000000000000000`; partition 17/ns 0 → `0x8800000000000000`; partition 17/ns 3 →
  `0x8800000000000003`; partition 0/ns 4 → `0x0000000000000004`; partition 1/ns 2 →
  `0x0800000000000002`; partition 17/ns 0/tag 9 → `0x8800000900000000`.
  Column is `[8 bytes -(block end)][8 bytes timestamp][var uid]` with an **empty value**
  (`StaticArrayEntry.of(target)`, `:333`). **The counter is negated, so the larger counter sorts
  earlier** — verified: `nextEnd=20001` → `ffffffffffffb1df…` sorts before `nextEnd=10001` →
  `ffffffffffffd8ef…`. That is what makes `getCurrentID`'s `setLimit(5)` (`:229`) correct: it reads the
  five **newest** claims, so the `max` loop is defensive, not load-bearing, and a row may hold thousands
  of historical claims at no read cost. Within one counter value the order is timestamp then uid, both
  ascending, so **the earliest writer is `blocks.get(0)`** and that is the seniority rule.
  Write is unconditional; the decision is a later read; losers delete their own column with
  `ROLLBACK_ATTEMPTS = 5` and `rollbackWaitTime = 200 ms`. **A failed rollback or a crash between write
  and delete burns ids permanently and never duplicates them** — that asymmetry is the whole safety
  argument. Lesson 18 should compare this directly with `ConsistentKeyLocker`; the comment at `:90-92`
  says the slice bounds mirror it.
- **330 ms, and it only happens on distributed stores.** `waitGracePeriod = idApplicationWaitMS/10`
  (`:165`), the sleep at `:353` is `wait + wait/10` = **330 ms** at the default, and both it and the
  slow-write abort at `:340-341` are gated on `manager.getFeatures().isDistributed()` (`:338`, `:352`).
  **So every in-memory and BerkeleyJE id test passes with the coordination latency removed**, and that
  is a property of `isDistributed()`, not of lesson 06's `IDAUTHORITY_WAIT = Duration.ZERO` override.
  `claimsOneAtATime = hasLocking() && !isDistributed()` (`:169`) collapses `blockClaimLocks` to the
  single key `0L` — a third distinct meaning of `hasLocking()` after the two lesson 07 found.
  `ClaimLock` (`:143-157`) is reference-counted inside `ConcurrentHashMap.compute` (`:273`, `:279`), so
  the map holds exactly as many locks as there are claims in flight. Project-measured payoff
  (`docs/changelog.md:2041-2058`): on Cassandra, 32 claims across 32 partitions **0.35 s instead of
  11.2 s**; ten threads × 100 vertices on a fresh graph **1.4 s instead of 7.0 s**; a single thread's
  first transaction still **6–7 s**, its ~20 claims genuinely sequential. **Any benchmark that opens a
  graph and times the first write is measuring this, not the write path.**
- **`BASE_ID = 1`, and it closes lesson 08's last arithmetic question.**
  `AbstractIDAuthority:40-45`, with the comment that it cannot be changed without corrupting existing
  allocations. `getCurrentID` starts there, the first block is `[1, 1+blockSize)`, `getId(0)` is
  `1 << tagBits`, and `IDAuthorityTest.checkBlock:197-214` asserts `id > 0` on every id of every block.
  **Count 0 is never offered by the authority** — `VertexIDType.addPadding`'s refusal was a second line
  of defence, not the first.
- **`IDBlock` is deliberately not a range.** `IDBlock:25-42` is `numIds()` + `getId(long index)` only,
  documented as requiring complete thread safety. That indirection is what lets the authority return a
  block whose ids are 16 apart. `StandardIDPool` uses two anonymous sentinel `IDBlock`s rather than
  flags: `UNINITIALIZED_BLOCK` (`:65-73`, `numIds()==0`, so the first `nextID` falls into `nextBlock`
  by construction) and `ID_POOL_EXHAUSTION` (`:53-63`, compared by reference at `:195`, used to carry an
  `IDPoolExhaustedException` across the `Future` boundary as a value).
- **Partition selection.** `assignID:193-272` → `SCHEMA_PARTITION = 0` for schema vertices,
  `PARTITIONED_VERTEX_PARTITION = 1` for partitioned ones, `placementStrategy.getPartition` otherwise;
  **a relation takes the partition of incident vertex `attempt` on the first `getLen()` attempts**
  (`:206-219`), so edges are co-located with an endpoint — the fact lesson 20's vertex-centric queries
  depend on. Custom **string** vertex ids route through
  `Math.abs(incident.id().hashCode() % partitionIdBound)` (`:218`). `MAX_PARTITION_RENEW_ATTEMPTS = 1000`
  and `IDPoolExhaustedException` means "try another partition" (`:226-228`). The re-homing block at
  `:231-268` sends properties of partitioned vertices to the canonical representative, edges to the
  non-partitioned endpoint, and both-partitioned edges to `getPartitionHashForId(relation.longId())` —
  which lesson 08 showed is the identity below `2^partitionBits`.
- **`ids.num-partitions` is 10 and `cluster.max-partitions` is 32, so 22 partitions are unused.**
  `SimpleBulkPlacementStrategy.currentPartitions` is an `int[10]` filled once by
  `setLocalPartitionBounds:112-119` and changed only on `exhaustedPartition:125-133`.
- **`PartitionIDPool extends EnumMap<PoolType,IDPool>`** (`:499-541`), so the container inherits public
  `put`/`remove`/`clear` and `close()` is `super.clear()`. `getPool:514-517` guards `!exhausted` with
  `Preconditions.checkArgument`, i.e. `IllegalArgumentException`, which the retry loop at `:226` does
  **not** catch — a real but practically unreachable race (it needs a partition's 2^51 counts actually
  exhausted). Record it as a wrong exception type, not a bug anyone will hit.
- **`AbstractIDAuthority.getPartitionKey(int)` (`:99-101`) has no caller.**
  `ConsistentKeyIDAuthority` declares a three-argument private method of the same name and never
  overrides or calls the inherited one. Also `getBlockSize:108-114` evaluates `getIdUpperBound` **twice**
  per call because Guava's `checkArgument` message arguments are eager.
- **`nextID`'s upper-bound check runs after the increment** (`StandardIDPool:230-233`), so the id that
  trips `idUpperBound` is consumed and lost and the pool is never left repeatable. Harmless because the
  caller treats the exception as "partition finished".
- **The runaway-renewal path.** `waitForIDBlockGetter:150-183` cannot interrupt a renewal on a backend
  without `supportsInterruption()` — an interrupt can break BerkeleyJE's whole `Environment`
  (`VertexIDAssigner:174-178`) — so it parks the `Future` in a 4-element `closeBlockers` deque, swept one
  element per append (`:165-171`), and `close()` waits on each. Thread names to grep in a stack dump:
  `JanusGraphID-renewal-%d` for the shared executor (`:126`), `JanusGraphID(partition)(namespace)[%d]`
  for a pool that fell back to its own thread (`StandardIDPool:131-137`). On a graph opened through
  `JanusGraphFactory` the second count should be **0**.

### Verified by lesson 07 — compressed; full block in git history at `513a317b`

- **`Backend` is two-phase.** The constructor turns `StoreFeatures` into configuration; `initialize`
  opens **six** named stores (`janusgraph_ids`, `edgestore`, `graphindex`, `system_properties`,
  `systemlog`/`txlog`), and **three of the six** go through the locking manager, so a locking-emulated
  backend really runs **nine** physical stores. **The ids store is opened through the *unwrapped*
  manager with no cache and no lock wrapper** — which is why lesson 09 found no cache in that path.
  The fixed-key-length map (`Backend.java:537-542`) gives `8` to `edgestore`, `edgestore_lock_` and the
  ids store **by name**; everything else pays the variable-length cost.
- **The transaction stack is four levels deep** (`BackendTransaction` → `CacheTransaction` →
  `ExpectedValueCheckingTransaction` → two real backend transactions, opened unconditionally) and
  `unwrapTx` strips exactly one level per wrapper. `KCVSProxy.java:67` omits it — PR candidate 1.
- **THE finding: the retry backoff is a coin flip, not a ±20% jitter.** `BackendOperation`'s
  `(int)(2.0 / (1 + (rand*2-1)*0.2))` truncates to 1 or 2, so `pertubTime(d)` ∈ {d, d/2}; reproduced
  verbatim under OpenJDK 21 (`{1=500803, 2=499197}` over 1,000,000 draws) and measured at a median of
  **14** attempts (max 38) against the 10 s read budget and **21** (max 48) against the 100 s write
  budget, against documentation that says "backoff exponentially". PR candidate 7 (file as an issue).
- **Cache invalidation does not evict** (`ExpirationKCVSCache.invalidate:266-270`): it marks the row so
  reads bypass for at least `db-cache-clean-wait`, and the sweep is triggered by a `CountDownLatch(5)`
  counted down by wasted reads. `cache.db-cache-size` default **0.3**. `NoKCVSCache` is 44 lines of
  empty overrides passing `metricsName = null`.
- **`CacheTransaction` is the write buffer**: `Map<KCVSCache, Map<StaticBuffer, KCVEntryMutation>>`
  with a monotonic `numMutations`; `storage.buffer-size` default **1024**, a durability knob that some
  backends ignore. `flushInternal` holds the caller's `Entry` objects across the whole flush and
  invalidates only afterwards — which is why lesson 04's `getNoCopy` aliasing test must mutate through
  a `CacheTransaction`.
- **`GraphDatabaseConfiguration.getBackend()` is a factory, not an accessor** (`:1951-1956`): every
  call builds a new `Backend`. Reachable from three call sites — PR candidate 5, which still needs its
  experiment run. `close()`/`clearStorage()` share one `hasAttemptedClose` flag.
- **`BackendTransaction` has five feature branches** against the raw features from `Backend.java:249`,
  including the `hasScan`/`hasOrderedScan` pair that answers lesson 06's scan-test questions, and
  `EDGESTORE_MIN_KEY`/`EDGESTORE_MAX_KEY`, whose safety lesson 08 proved and lesson 09's `BASE_ID = 1`
  completes. `commit()` is storage-then-indexes; `StandardScanner` is built from the unwrapped manager
  (`Backend.java:305`). **No non-key-consistent fallback exists**: `initialize` L339-343 throws
  `IllegalStateException`.
- **`ExpirationCacheTest` is the single concrete subclass of `KCVSCacheTest`**, and
  `#testGracePeriod` asserts the cache-bypass semantics directly (`getSliceCalls` 0 → 2 → 4 on two
  identical reads), needing exactly `PENALTY_THRESHOLD = 5` reads to wake the cleanup thread.

### Verified by lesson 08, for lessons 10-13 and the read path to cite rather than re-derive

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
  **Lesson 09 narrows all of these by four bits in practice** — see its block above.
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
  `zeroBuffer(8)` needs `count == 0`, forbidden by `addPadding:347` and asserted at `getKey:501` — and
  **lesson 09 showed the authority never offers count 0 at all** (`BASE_ID = 1`). The half-open interval
  is therefore safe here — **by accident of the suffix encoding, documented nowhere**. See PR candidate 9.
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

### PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 11–14 are new this run** (lesson 09), and **11 is now the
smallest the track has found**. Candidates 1–10 carry forward and were **not** re-verified this run:
8–10 were verified at this pinned commit on 2026-11-06, 1 and 7 on 2026-11-05, 2–4 and 6 on 2026-11-04,
and 5 still needs its experiment run.

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

### Still open, and who owns it

- **Closed by lesson 09:** where `count` and `partition` come from; the full claim protocol and its
  330 ms; the five pools and their block sizes; lesson 08's `count == 0` question (`BASE_ID = 1`); and
  lesson 06's note about `IDAUTHORITY_WAIT = Duration.ZERO`, which turns out to be subsumed by the
  `isDistributed()` gate.
- **Owed to lesson 10 by lessons 08 and 09:** `TYPE_LEN_RESERVE = 3` and `MAX_PADDING_BITWIDTH = 6` are
  accounted for but not spent — `EdgeSerializer.writeRelation` spends them, with
  `SCHEMA_COUNT_BOUND = 1 << 54` as the budget. `IDHandler`'s **different** first-bit convention (per
  `getKeyID`'s javadoc at `:523-529`) must be reconciled there, and
  `IDManagementTest.edgeTypeIDTest`'s 1,000,000-trial `IDHandler` loop is the starting point.
  `BufferUtil.nextBiggerBuffer` / `padBuffer` also go to lesson 10. **From lesson 09:** relation counts
  arrive from a pool whose block is 80,000 wide in steps of **16**, and an edge is co-located with one
  endpoint's partition — both relevant to the column layout's locality claims.
- **Owed to lesson 16 by lesson 09:** `assignIDs(Iterable<InternalRelation>)`
  (`VertexIDAssigner:278-333`) is called from `StandardJanusGraph.prepareCommit`, and `ids.flush`
  (default `true`) decides whether ids are assigned at element creation or at commit.
  `VertexIDAssignerTest#testDisableIdsFlush:123` is the entry point. The bulk path's two-pass structure
  (assign vertices, then relations) and its own 1000-attempt `leftOvers` loop belong there, not in 09.
- **Owed to lesson 18 by lesson 09:** `ConsistentKeyIDAuthority`'s comment at `:90-92` says its slice
  bounds mirror `ConsistentKeyLocker`'s. Lesson 18 should read the two protocols side by side — the
  authority writes-waits-reads and the earliest column wins; the locker is claimed to do the same thing
  for a different purpose. Also still owed: `ExpectedValueCheckingTransaction.prepareForMutations()`,
  `ExpectedValueCheckingStore`, `BerkeleyJETx` (6,524 B), `BerkeleyJELocks`,
  `LockKeyColumnValueStoreTest` (23,021 B). Lesson 18 starts already knowing that two backend
  transactions are open per graph transaction and that `*ConcurrentTest.java` is excluded from
  `mvn test` at root `pom.xml:416`.
- **Owed to lesson 19 by lesson 08:** the partitioned-vertex read cost — one logical vertex, up to
  `2^partitionBits` representative rows, one slice query each. Plus, carried from lesson 04 and sharpened
  by lesson 07: the `getNoCopy` aliasing question belongs in `KeyColumnValueStoreTest` (gated by nothing,
  applying to every backend) and must mutate through a `CacheTransaction`; the `MemoryEntryList` sizing
  question and the `InMemoryColumnValueStore.java:162` NPE candidate also land here. `MultiPageEntryBuffer`,
  `BufferPageUtils` and `BufferPage.merge` remain deferred with no owner; propose lesson 19.
- **Owed to lesson 20 by lesson 09:** edges take the partition of an incident vertex
  (`VertexIDAssigner:206-219`), so vertex-centric queries have a locality property that falls out of id
  assignment rather than out of the query layer. Worth one paragraph there.
- **Owed to lesson 22:** `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299+`, shared mutable `current`, per-key limit at
  `L381`) plus `SimpleScanJob` (14,040 B).
- **Owed to lesson 27 by lesson 09:** the ids-store column layout —
  `[-(end)][timestamp][uid]` with an empty value, slice bounds `zeroBuffer(1)` to `oneBuffer(17)` — is
  the smallest real column layout in the codebase and the cleanest thing to compare against
  `CQLKeyColumnValueStore` directly. Hands-On §2's byte dump is specified for it. Also: the count stride
  of 16 at default tag-bits, which affects CQL token distribution.
- **Carried from sessions 01–04:** 24 questions posed, none answered by Aryant. The half-open interval now
  has **three** implementations on record (`SinglePageEntryBuffer.getSlice` binary search;
  `BerkeleyJEKeyValueStore.java:258`'s per-record `compareTo`;
  `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange`) and two consumers relying on the upper
  bound being exclusive (`BackendTransaction`'s `EDGESTORE_MAX_KEY`, and `ConsistentKeyIDAuthority`'s
  `UPPER_SLICE = oneBuffer(17)` against columns of 16+uid bytes). Lesson 27 adds CQL as the fourth
  implementation. The contract is in `KeyColumnValueStore`'s javadoc and **none** of the enforcement is.
- **tdd.cat, 2026-11-07: the archive led the front page for the second consecutive run, and the
  one-way cross-check fired again.** At ~03:40 UTC the front page served **2026-11-05** (62 stories) —
  the edition lesson 08 digested — while `/archive/` reported **131 editions** with **2026-11-06 / 52**
  as *Latest* and 2026-11-07 unlinked. `https://tdd.cat/2026-11-06/` served the complete 52-story
  edition with every source URL on the first attempt, trailing-slash form. Three fetches, no forward
  probe, no fallback. **The front page is now two days behind**, the largest gap recorded since the
  2-day settling window was found, and the rule stands exactly as written on 2026-11-05: front page
  authoritative for "newest available", `/archive/` consulted only to notice a *larger* maximum. Two
  consecutive successful uses is the first time any rule in that table has been *used* twice rather
  than merely not needed. Rendering note, the fourth in five runs: the filter bar read "No stories
  match the selected filters in today's edition" while the header counted 52, so Must-Read /
  Recommended tags were unreadable again — one line, nothing more. **Practical consequence for the
  next run: the front page may well still serve 2026-11-05 or 2026-11-06, so check `/archive/` for a
  2026-11-07 or 2026-11-08 maximum and do not re-digest an edition the session log already names.**

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
