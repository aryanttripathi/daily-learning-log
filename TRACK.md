<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 11
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

- **11 — 2026-11-09** · [`2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md`](2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md) · **stayed on `a59ae8d8`** (mid-Part; the re-pin decision is due after lesson 13, one run away). Read `EdgeSerializer` again (whole, 591 lines, now from the read side), `RelationCache` (whole), `RelationReader` (whole, 10 lines and the point is what it does **not** declare), `VariableLong` (whole, this time the backward section), `IDHandler` (whole), `StaticArrayEntry` (whole), `StaticArrayBuffer` (whole), `ReadArrayBuffer` (whole), `CacheEdge` (whole), `RelationConstructor` (whole), `ModificationDeserializer` (whole), `Multiplicity` (whole), `JanusGraphVertexDeserializer` (whole), `EdgeSerializerTest` (whole, new this autumn), `RelationReparseBenchmark` (whole, new this autumn), plus commit `29c0b18a` (2026-10-02, Oleksandr Porunov, fixes #4995) and fragments of `CacheVertexProperty` and `RelationCacheTest` via code search. **THE finding: the backward encoding is the only genuinely order-preserving varint in the tree, and that — not readability — is why the tail ids are written backward.** `writeUnsignedBackward` puts the length in bits 4-6 of the **first** byte, the mirror image of `writeUnsigned`'s STOP-bit-on-the-**last**-byte, so a longer encoding's first byte is strictly greater: re-implementing `unsignedBackwardLength`/`writeUnsignedBackward`/`readUnsignedBackward` line-for-line and sweeping `0..400000` finds **zero** order inversions, against lesson 10's one at every length boundary in the forward prefix encoding. `262143 → 262144` is `8f 7f 7f → 90 10 00 00`. That is what makes `ImplicitKey.ADJACENT_ID` and `JANUSGRAPHID` legal sort-key positions in `getQuery`. The price is exact: capacity is **`4 + 7·(n-1)`** bits, not `7n`, and there is a **hard 3-byte floor** from `Math.max(3, ...)`, so the first vertex id a default graph assigns (**4096**, lesson 09) costs **3 bytes backward against 2 forward** (`80 20 00` vs `20 80`); beyond the floor the length field costs +1 byte for 3 of every 7 bit-lengths (bitLength ≡ 5, 6, 0 mod 7; boundaries at 19, 26, 33, 40, 47, 54, 61). Over `NormalVertex` ids at counts 16..160,000,000: **38 bytes backward against 34 forward, +12%**. A plain `MULTI` edge — the default multiplicity — is an **8-byte column with an empty value** (2 prefix + 3 `vid←` + 3 `rid←`) against a 6-byte forward ideal, i.e. ~400 MB of length fields on a 100M-edge graph stored twice, on top of lesson 10's ~200 MB of prefix padding from the same four tag bits. Round trip and the capacity law verified over `0..299999` plus `2^k ± {0,1,2}` for k=5..62, 0 failures. **Lesson 10's carried question, answered: the layout does not decide.** `parseRelation` completes the relation for all ten (type × direction) rows; what decides whether a header-only parse is as good as a full one is the four-way test at `:201-202` — `!in.hasRemaining() && keySignature.length == 0 && def.getSignature().length == 0 && !hasPropertyMetaData(data)`. The layout constrains it in one direction only: the two unconstrained rows are the only ones that can carry a sort key, so **every vertex-centric index replica fails clause 2 by construction** and is re-parsed on its first property read, permanently — the #4995 fix helps base columns and cannot help replicas. `EdgeSerializerTest.testRelationIsFullyParsedOnce` pins it at **17 of 50** entries, and the 17 and the 50 both reconstruct exactly. Also: `SIMPLE` **is** constrained (`isConstrained()` is literally `this != MULTI`), so lesson 10's six layouts are ten (type × direction) rows collapsing into **three** parse shapes — forward-only, pivot-at-`valuePosition`, full backward walk — and `SET` is the one constrained case that reads straight *through* `valuePosition` without jumping. **Second finding (PR candidate 18, the most substantial this lesson found): the OLAP path cannot reach the entry cache at all.** `RelationReader` declares only `parseRelation`; `readRelation` — which reads and writes `StaticArrayEntry`'s `transient volatile RelationCache cache` — is a method on `EdgeSerializer` alone. `JanusGraphVertexDeserializer` holds a `RelationReader` from `setup.getRelationReader()` and iterates the same `Iterable<Entry>` **twice**, calling `parseRelation(data, false, typeManager)` both times, so **every edgestore column in a Hadoop/Spark scan is fully parsed twice and memoised never** — and the first loop reads only `relation.typeId` and `getOtherVertexId()`, for which a header-only parse, or `EdgeSerializer.parseTypeId`, would do. Third finding (PR candidate 20): **the ASC sort-key read is unbounded and the DESC read is not.** `:172` sends DESC through `in.subrange(keyLength, true)`, which **allocates** (`new byte[length]` + arraycopy + `(byte)~b` loop + two wrappers) and whose `limit` makes `StaticArrayBuffer.require` throw past the sort-key window; ASC reads in place out of `in` itself, so an over-reading sort-key serializer would walk into the `vid←`/`rid←` tail and `in.movePositionTo(currentPos)` would discard the evidence — silently wrong values on ASC, a loud `ArrayIndexOutOfBoundsException` on DESC. Fourth (PR candidate 19): `RelationCache.get`, `numProperties` and `propertyIterator` all dereference a `null` `properties` on a header-only cache — a bare NPE, documented only in the `isFullyParsed()` javadoc, with exactly **three** hand-written guards behind it (`EdgeSerializer:85`, `CacheEdge.getPropertyMap`, `CacheVertexProperty.getPropertyMap`) and no test. Fifth: the shared static `EMPTY` map is a **mutable** `LongObjectHashMap` and `propertyIterator()` refuses to iterate it because HPPC writes an iteration seed on `iterator()` — a shared immutable sentinel that turns out not to be immutable. Sixth, cosmetic: `unsignedBackwardLength`'s `bitLength <= 4 ? 0 : ...` ternary is **dead** — `Math.max(3, ...)` discards the computed value for every bitLength 1..11 (11 of 64), and the whole function equals `max(3, ceil((bitLength+3)/7))`, verified for all 64 bit lengths. Also closed: `endKeyPos` is **exclusive** and `:169`'s comment ("we are on the last byte of the key") is off by one in wording while the arithmetic is right; `hasPropertyMetaData` is **not** `hasMetaData` (membership in `ImplicitKey.MetaData2ImplicitKey`, so `TIMESTAMP` keeps a parse header-only and a grouped read's `ROW_KEY` does not); `readVertexId(in, false)`'s string/long peek can never misfire on a long, because a backward long's last byte is always `≤ 0x7f` while its stop bit sits at the low address; and `ModificationDeserializer`'s `hasProperties()` is **not** a repeat of the #4995 mistake — it follows an explicit full `readRelation`, so it means "are there any". Daily Diff: the **2026-11-08** edition (22 stories), which lesson 10's run could not see — deep dive on a Postgres replication slot that cannot advance because its captured tables are idle while the server keeps writing WAL (~680 MB held over 25 quiet minutes at defaults), and on why the obvious fix silently loses half the rows on restart.
- **10 — 2026-11-08** · [`2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md`](2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md) · **stayed on `a59ae8d8`** (mid-Part; the re-pin decision is due after lesson 13, not available this run). Read `EdgeSerializer` (whole, 591 lines), `IDHandler` (whole), `VariableLong` L56-235, `EdgeDirection` (whole), `IDManager` L60-340 / L372-420 / L691-723, `Multiplicity` L35-120, `BufferUtil` L178-212, `WriteByteBuffer` L140-154, `StandardJanusGraph` L896-904 / L999-1006 / L1100-1156, `ManagementSystem` L330-400, `RelationTypeVertex` L54-117, `StandardEdgeLabelMaker` L90-110, `StandardRelationTypeMaker` L61-67 / L119-126 / L196-199, `TypeDefinitionCategory` L40-86, `BaseRelationType` L59-68, `ImplicitKey` L151-160, `InternalRelationType` L30-38, `Serializer` L38-44, `StandardSerializer` L262-272, `Interval`/`PointInterval`/`RangeInterval`, `BasicVertexCentricQueryBuilder` L498-723, `IndexRepairJob` L211-219, `TransactionLogHeader` L86-93, `IndexRecordUtil` L454-462, `AbstractTypedRelation` L101-110, `JanusGraphLazyRelation` L157-166, `JanusGraphVertexDeserializer` L124-133, plus `EdgeSerializerTest`, `IDManagementTest` L141-265, `SerializerTest` L139-170, `JanusGraphTest` L973-982 and `docs/advanced-topics/data-model.md`. **THE finding: lesson 09's four conflict-avoidance bits cost one byte in every single cell of every row.** `writePositiveWithPrefix` with `PREFIX_BIT_LEN = 3` gives a **1-byte** relation-type prefix only for `count << 1 | dir ≤ 15`, i.e. **schema count ≤ 7** — and the smallest count a default graph can assign is **16**, so **the cheapest case in the encoding is unreachable on every default deployment**, permanently (`FIXED` option). The 2-byte prefix then holds exactly **63** relation types (counts 16..1008); the 64th (count 1024) spills to 3 bytes. Second finding: **columns are grouped by relation type, not sorted by relation-type id.** Re-implementing `writeUnsigned` / `writePositiveWithPrefix` / `getPrefix` line-for-line and sweeping counts 1-5000 finds exactly **one** inversion — `1023 → 1024` (`7f fe` vs `70 10 80`), and at the default stride of 16 it is **1008 → 1024**, so the 63rd type sorts *after* the 64th. Inversions sit at every byte-length boundary because the STOP bit marks the **last** byte. Nothing in the read path depends on the stronger property — `getBounds` compares only the top 3 bits, per-type slices are exact byte-prefix matches, and the same STOP bit keeps different-length prefixes from interleaving inside a `Direction.BOTH` window — and **no test asserts any of it** (`IDManagementTest.edgeTypeIDTest`'s 1,000,000 trials never compare two type prefixes). PR candidate 15. Third: **six prefix bands exist, and `getBounds` can reach only four of them** — it passes `getPrefix(systemTypes, systemTypes)`, so it emits `(0,1)` or `(2,3)` and never `(4,5)`; bands 4 (`0x80`-`0x9f`) and 5 (`0xa0`-`0xbf`) are the **invisible-non-system** bands, which is exactly where every vertex-centric index replica lives. `ManagementSystem.buildRelationTypeIndex:368`'s `maker.invisible()` is what puts them there, and `StandardEdgeLabelMaker:106` makes invisibility *structurally required* (an IN-only unidirected label is illegal otherwise). An edge with *N* vertex-centric indexes is **2 + N full columns**, each a complete re-serialization. Fourth: **the column *is* the uniqueness constraint** — `valuePosition` is assigned in six places and `MANY2ONE`-at-OUT and `SINGLE` get a bare-prefix column, so last-write-wins needs no read; `SET` puts the value in the column (set semantics for free); `MULTI`/`LIST` put the relation id in the column so nothing is overwritten. Fifth: `EdgeSerializer:481-483`'s `interval.isEmpty()` branch is **dead** — `BasicVertexCentricQueryBuilder:607-610` returns `emptyQuery()` first — and if reached it widens the slice to the whole type band (PR candidate 17). **Closed lesson 08's and 09's loose ends:** `TYPE_LEN_RESERVE = 3` **is** `IDHandler.PREFIX_BIT_LEN`, `MAX_PADDING_BITWIDTH = 6` **is** the one shift in `stripEntireRelationTypePadding`, the "different first-bit convention" needs no reconciling (row keys put the partition at the top, columns put the band at the top — different address spaces), and `BufferUtil.nextBiggerBuffer` **cannot overflow on a type prefix** (band maxes at 5, so byte 0 ≤ `0xbf`) while `padBuffer` turns out never to touch columns at all (its only caller pads **row keys** in the OKVS adapter). Also verified: the missing-value `0xff` marker sorts last in ASC and, flipped to `0x00`, first in DESC, with the exclusive bound applied on the matching side in each case. Daily Diff: **no 2026-11-08 or 2026-11-07 edition exists** — front page and `/archive/` both top out at 2026-11-06, which lesson 09 already digested — so one uncovered item from that edition instead: Postgres `SKIP LOCKED` queues saturating 1 vCPU at 64 connections against a `max_connections` of 100.
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
- [x] 10 — `EdgeSerializer.writeRelation`: the column layout for edges and properties, direction, sort keys
- [x] 11 — `EdgeSerializer.parseRelation` and `RelationCache`: decoding columns back into relations
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
2026-10-30.** Lesson 11 is mid-Part, so the re-pin question was not available and was not raised:
**stay on `a59ae8d8`**. The next boundary where re-pinning is allowed is **Part II/III, after lesson
13** — one run away. At that point either re-pin and record the new commit in the front-matter and
here, or state explicitly that it is staying, as lessons 08-11 did.

**The roll-off rule applied again this run.** The root README states that `TRACK.md` keeps the notes
for the three most recent lessons and older ones roll off into git history. The three most recent are
now 09, 10 and 11, so: **lesson 08's compressed block is removed** (it remains verbatim in git history
at `8af5383f23dca6b67f6ec56ca71c8164bc38000f`), **lesson 09's block is now compressed** to the facts
lessons 12-13, 16, 18 and 27 were promised — the full block is preserved verbatim in git history at
`1bbfec7cd08bbf33182dcc883135c9b7ba4e84e7` — and lessons 10 and 11 keep their blocks in full. Next
run, lesson 10's block becomes the oldest of the three and the same rule applies to it.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Part II is nearly done; two items left.** Lessons 08-11 cover the id bit layout, the id supply, the
  column layout and the parse. **Lesson 12 is `StandardSerializer`**, which owns the attribute bytes
  lessons 10 and 11 both treated as opaque, and which now has a specific job: lesson 11's PR candidate 20
  is only worth filing if some `readObjectByteOrder` implementation can consume a different number of
  bytes than it wrote, because that is the only way the unbounded ASC sort-key read becomes reachable.
  Lesson 13 is schema persistence, and should verify rather than assume that schema-definition relations
  land in bands 0 and 1 — and additionally that a schema vertex's own row is decodable without
  `getExistingRelationType` recursing forever, since `parseRelation:113` performs that lookup before it
  knows anything else about the cell.
- **Entry size.** Lesson 05 ≈35 KB, 06 ≈63 KB, 07 ≈78 KB, 08 ≈67 KB, 09 ≈61 KB, 10 ≈70 KB, 11 ≈56 KB
  (a large share of each is permalink URLs, which are not prose). In line with the SQLite and PostgreSQL
  lessons. The old 5-9 KB budget was a `contributor-reading` constraint and does not apply.
- **Method note, carried and used again.** Lesson 10's byte sequences came from re-implementing the pure
  encoding functions line-for-line from the pinned sources and running them, paired with an in-tree
  exercise that runs the real code. Lesson 11 did the same for the **backward** encoding
  (`unsignedBackwardLength`, `writeUnsignedBackward`, `readUnsignedBackward`) and specified Hands-On §8.3
  as the in-tree check. Keep both halves for lesson 12's order-preserving serializers: state the method,
  and always pair it with an exercise that runs the real code.

### Verified by lesson 11, for lessons 12-13, 19, 20 and 27 to cite rather than re-derive

- **The bytes are not self-describing; the schema is part of the format.** `parseRelation:113-116` looks
  up `tx.getExistingRelationType(typeId)` and reads `multiplicity()`, `getSortKey()`, `getSignature()`
  and `getSortOrder()` off it **before** it can classify any byte after the prefix. Nothing in the cell
  records which schema version wrote it. Three consequences later lessons need: a sort-key or signature
  change is a storage-format change; there is one type lookup per cell on every read; and lesson 13 must
  find where the lookup stops recursing for schema vertices, which is the one place the format *is*
  self-describing.
- **Ten (type × direction) rows, three parse shapes.** `Multiplicity.isConstrained()` is `this != MULTI`,
  so **`SIMPLE` is constrained** and `isUnique(dir)` is false for it both ways. Correcting and expanding
  lesson 10's table (`→` read forward, `←` read backward):
  `MANY2ONE`@OUT / `ONE2MANY`@IN / `ONE2ONE`@either → col `P`, val `vid→ rid→` (**forward-only**);
  `MANY2ONE`@IN / `ONE2MANY`@OUT / `SIMPLE`@either → col `P vid←`, val `rid→` (**pivot at
  `valuePosition`**); `MULTI`@either → col `P S vid← rid←`, val signature+props (**full backward walk**);
  `SINGLE` property → col `P`, val `value rid→`; `SET` property → col `P value`, val `rid→`;
  `LIST` property → col `P S rid←`, val `value`+props. Lesson 10's arrow on the `SIMPLE` row was `→` and
  should have been `←`; the placement was right.
- **`valuePosition` is the only framing in the cell, and it is an origin as well as a boundary.** On the
  pivot shape (`:128-132`) the parser jumps to `valuePosition`, reads the vertex id **backward** out of
  the column, jumps back to the same offset, and reads the relation id **forward** out of the value — two
  encodings meeting at one offset, read in opposite directions. `SET` (`:147-148`) is the one constrained
  case that reads straight through it without jumping, because `StaticArrayEntry.of` arraycopies column
  and value into a single `byte[]`. The column/value split is a storage-engine concept the parser
  otherwise ignores.
- **THE finding: the backward encoding is order-preserving and the forward one is not.**
  `writeUnsignedBackward` lays out `byte 0 = [stop=1 | numBytes-3 (3 bits) | value bits (4)]` then
  `[0 | 7 value bits]` per byte, so the length is in the **first** byte and a longer encoding sorts after
  a shorter one. **Zero inversions over `0..400000`** (against lesson 10's one per length boundary in
  `writePositiveWithPrefix`): `262143 → 262144` is `8f 7f 7f → 90 10 00 00`. **This, not readability, is
  why the tail ids are written backward** — it is what makes `ImplicitKey.ADJACENT_ID` and
  `JANUSGRAPHID` usable as extended sort-key positions in `getQuery:466-496`, which is **lesson 20's
  starting point**. Capacity is `4 + 7·(numBytes-1)` bits (byte 0 donates 3 of its 7 payload bits to the
  length; `readUnsignedBackward` masks the first byte `& 0x0F` to match), with a **hard 3-byte floor**.
  Costs, exact: 4096 (the first default vertex id) is `80 20 00` backward vs `20 80` forward, 3 bytes vs
  2; +1 byte for bitLength ≡ 5, 6, 0 mod 7; boundaries at bitLength 19, 26, 33, 40, 47, 54, 61; **+12%**
  over `NormalVertex` ids at counts 16..160,000,000; a default `MULTI` edge is an **8-byte column with an
  empty value**. Lesson 27 should compare this against CQL's clustering-key encoding, which gets its
  ordering from fixed-width components instead.
- **`readPositiveBackward` leaves the cursor on the stop byte**, i.e. one *past* the last byte it
  consumed in reading order, which is what lets a `MULTI` column chain `rid←` then `vid←` with no length
  header between them. Hence `endKeyPos` (`:138`, `:152`) is **exclusive** and `keyLength = endKeyPos -
  startKeyPos` is exact; `:169`'s comment ("we are on the last byte of the key") is off by one in wording
  only.
- **`readVertexId(in, false)`'s string/long disambiguation cannot misfire on a long.** It peeks
  `getByte(position - 1)` for `STRING_ID_MARKER` (`0x80`), and a backward-written long's byte at the high
  address is `0 | (0x7F & value) ≤ 0x7f` — the stop bit sits at the **low** address, the opposite end.
  **Lesson 12 must verify** that `VariableString.writeBackward` therefore places a high-bit byte at the
  **high** address; custom string vertex ids ride this path.
- **`RelationCache` is a three-state memo and the middle state is a `null`.** `properties == null` →
  header-only (`isFullyParsed()` false); shared static `EMPTY` → fully parsed, found none
  (`isFullyParsed()` true, `hasProperties()` false); non-empty map → fully parsed with properties. #4995
  was the collision between the second and first states through `hasProperties()`, and the fix
  (`29c0b18a`) is a third state, not a better predicate. `EdgeSerializer:85`'s condition
  `map == null || !(parseHeaderOnly || map.isFullyParsed())` is sound in all five reachable quadrants.
  **Lesson 19 needs this**: the header-only state is exactly what `isLazyLoadRelations()` and
  `JanusGraphLazyRelationConstructor` trade on.
- **The completeness test, and why index replicas are permanently excluded.** `:201-202` returns a
  *fully parsed* cache from a header-only parse iff `!in.hasRemaining()` (no inline properties — correct
  in all ten rows, since every branch leaves the cursor at the start of the signature region) **and**
  `keySignature.length == 0` **and** `def.getSignature().length == 0` (a signature field is written even
  when the relation has no value for it) **and** `!hasPropertyMetaData(data)`. Clause 2 is free for the
  constrained shapes (`:302`'s write-side assert) and is a real test for the unconstrained ones — so
  **every vertex-centric index replica fails it by construction**, a sort key being the point of an index
  type. `EdgeSerializerTest` pins **17 of 50**: 7 of 8 edge labels (`signed` is out by clause 3) × 2
  positions × the no-property variant = 14, plus 3 of 4 property keys = 3; the `indexedasc`/`indexeddesc`
  **base** entries are among the 17 because only the index type carries the sort key.
- **`hasPropertyMetaData` is not `hasMetaData`.** `:212-224` tests membership in
  `ImplicitKey.MetaData2ImplicitKey`, so `EntryMetaData.TIMESTAMP` and `TTL` keep a parse header-only
  while a grouped CQL multi-key read's `ROW_KEY` — which stands for no property — does not, and a
  `ROW_KEY` set beside a `TIMESTAMP` does not hide it. `EdgeSerializerTest`'s metadata block is the only
  place in the tree that pins this; **lesson 27 should check it against the CQL reader that sets
  `ROW_KEY`.**
- **The memo is per-`Entry`-object and dies with it.** `cache` is `transient volatile` on
  `StaticArrayEntry`, written by `setCache` with no CAS, so two threads racing the same entry both parse
  and one write wins (harmless — equal immutable values). "Parsed once" is really "parsed once per thread
  that races", and the hit rate across transactions is a property of lesson 07's caching layers, not of
  the serializer.
- **`EMPTY` is a shared *mutable* `LongObjectHashMap`**, and `propertyIterator()` returns
  `Collections.emptyIterator()` rather than iterating it because HPPC writes an iteration seed on
  `iterator()`. Also, the constructor's `(properties == null || properties.size() > 0) ? properties :
  EMPTY` normalisation has an **unreachable arm from inside `parseRelation`**: the map is lazily
  allocated by the first `put`, so a non-null map always has size ≥ 1.
- **`RelationReader` declares only `parseRelation`.** The entry cache lives on
  `EdgeSerializer.readRelation`, which is not on the interface — the structural cause of PR candidate 18
  and the reason the #4995 fix cannot reach the OLAP path. Any future consumer that is handed a
  `RelationReader` instead of an `EdgeSerializer` is excluded from the cache by construction.

### Verified by lesson 10, for lessons 12-13, 17 and 20 to cite rather than re-derive

- **`writeRelation` writes one buffer and returns one offset.** `EdgeSerializer:288-381` produces a
  `StaticArrayEntry(buffer, valuePosition)`; there is no second buffer. `valuePosition` is assigned in
  **six** places and each assignment is a different storage contract. The full table (P = relation-type
  prefix, S = byte-ordered sort-key values, `→` forward-readable, `←` backward-readable):
  edge `MULTI` → col `P S vid← rid←`, val signature+props (`:325-327`); edge `SIMPLE` → col `P vid→`,
  val `rid`+… (`:320-322`); edge unique-in-dir (`MANY2ONE` at OUT, `ONE2MANY` at IN, `ONE2ONE` either)
  → col `P`, val `vid→ rid`+… (`:317-318`); property `SINGLE` → col `P`, val `value rid`+… (`:338-339`);
  property `SET` → col `P value`, val `rid`+… (`:341-342`); property `LIST` → col `P S rid←`, val
  `value`+… (`:347-349`). `Multiplicity.convert`: `LIST→MULTI`, `SET→SIMPLE`, `SINGLE→MANY2ONE`.
- **The column *is* the uniqueness constraint.** A bare-prefix column means one physical cell per
  `(vertex, type)`, so a second `MANY2ONE` out-edge **overwrites** the first with no read, no lock and
  no check; `SET`'s `P value` column is set semantics for free; `MULTI`/`LIST` put the relation id in
  the column so nothing is ever overwritten. This is why `isDirectColumnByStartOnlyAllowed:566-577`
  returns true **only** for `Cardinality.SINGLE` property keys (and only for a `PropertyKeyVertex`,
  reading `TypeDefinitionCategory.MULTIPLICITY` out of the definition map rather than calling
  `multiplicity()`), and why `ManagementSystem:348-349` refuses to index a constrained direction.
  **"Sort key ⟺ unconstrained multiplicity" is enforced three times in three files** —
  `ManagementSystem:348`, `StandardEdgeLabelMaker:103-105`, and `EdgeSerializer:302`, the only one in
  the write path and a **bare `assert`**.
- **THE finding: lesson 09's four tag bits cost one byte in every cell.** `writePositiveWithPrefix`
  (`VariableLong:160-186`) with `PREFIX_BIT_LEN = 3` lays the first byte out as
  `[band(3) | continue(1) | top 0-4 bits of (count<<1 | dirBit)]`, so a **1-byte** prefix needs
  `count << 1 | dir ≤ 15`, i.e. **count ≤ 7**. The smallest count a default graph can assign is **16**
  (lesson 09: tag-bits `FIXED` at 4, tag always 0), so **the 1-byte prefix is unreachable on every
  default graph** and is not reachable later either. Exact prefixes at count 16: `50 a0` (property),
  `70 a0` (edge OUT), `70 a1` (edge IN). The **2-byte prefix holds counts ≤ 1023 → exactly 63
  multiples of 16** (16..1008); count 1024 is 3 bytes. ~200 MB of pure padding on a 100M-edge graph.
- **Columns are grouped by relation type, NOT sorted by relation-type id.** Sweeping counts 1-5000:
  exactly **one** inversion, `1023 → 1024` (`7f fe` vs `70 10 80`); at the default stride of 16 it is
  **1008 → 1024**. Cause is structural — `writeUnsigned:60-70` sets `STOP_MASK` on the **last** byte,
  so a longer encoding's second byte is always `< 0x80` and a 2-byte encoding's is always `≥ 0x80`.
  Inversions sit at every length boundary (`131071 → 131072` next). **Nothing in the read path depends
  on the stronger property**, and the same STOP bit is why: `getBounds` compares only the first byte's
  top 3 bits; `getQuery(type, dir, sortKey)` writes the same prefix bytes with the same function for
  both ends, so a per-type slice is an exact byte-prefix match; and a 3-byte prefix sharing a first byte
  can never land inside a 2-byte `Direction.BOTH` window (`70 a0`..`70 a2`) because its second byte is
  `< 0x80`. **Untested and undocumented** — `IDManagementTest.edgeTypeIDTest:141-201` runs 1,000,000
  trials and asserts round trip, `DirectionID`, full consumption, inline round trip, `DataOutput`
  equality and both `getBounds` bracketings, and never compares two different counts. PR candidate 15.
- **Six prefix bands; `getBounds` reaches four.** `DirectionID.getPrefix:93-96` is
  `((systemType?0:invisible?2:1)<<1) + (id>>>1)`, and the direction does **not** affect the band:
  0 = system property (`0x00`-`0x1f`), 1 = system edge (`0x20`-`0x3f`), 2 = user property
  (`0x40`-`0x5f`), 3 = user edge, **OUT and IN interleaved per type** (`0x60`-`0x7f`), 4 =
  invisible-non-system property (`0x80`-`0x9f`), 5 = invisible-non-system edge (`0xa0`-`0xbf`), 6-7
  unreachable. `getBounds:185-206` only ever passes `getPrefix(systemTypes, systemTypes)`, so it emits
  `[0x00,0x40)` or `[0x40,0x80)` and **never touches bands 4-5**. `IDManagementTest.testDirectionPrefix`
  asserts both bounds are exactly 1 byte and `< 0xff`.
- **Bands 4-5 are the vertex-centric index bands, and that is deliberate.**
  `ManagementSystem.buildRelationTypeIndex:340-393` creates a **whole new relation type** named
  `<baseTypeId>:<indexName>`, with `maker.invisible()` (`:368`), `Multiplicity.MULTI` (`:369`), the sort
  keys, the sort order, and a signature pruned of the sort keys (`:374-385`); it is linked to the base
  type by a `RELATIONTYPE_INDEX` schema edge (`:387`). `RelationTypeVertex.getRelationIndexes:109-117`
  returns `[this] ++ indexTypes`, and `StandardJanusGraph:1124-1149` calls `writeRelation(edge, type,
  pos, tx)` once per (index type × covered position). **Invisibility is structurally required**, not
  just convenient: `ManagementSystem:355` can pass `Direction.IN` to `lm.unidirected`, and
  `StandardEdgeLabelMaker:106` forbids an IN-only unidirected label unless `INVISIBLE` is set. A
  unidirected index type fails the `isUnidirected` check at `StandardJanusGraph:1127-1128` for the other
  position, so **an edge with *N* vertex-centric indexes is 2 + N full columns**, each with its own
  prefix, sort-key window, tail ids, signature and inline properties — and each index type burns a
  schema count, i.e. 16 at default stride, out of the same 63-type 2-byte window.
  `IndexRepairJob:211-219` re-runs the identical loop, which is why reindexing needs no new code path;
  `JanusGraphVertexDeserializer:124-128` filters bands 4-5 in application code because it scans whole
  rows instead of issuing band-bounded slices.
- **`TYPE_LEN_RESERVE = 3` IS `IDHandler.PREFIX_BIT_LEN`, and `MAX_PADDING_BITWIDTH = 6` IS one shift.**
  `stripEntireRelationTypePadding:701-704` is `UserEdgeLabel.removePadding` = `id >>> 6`; all four proper
  relation-type suffixes are 6 bits (`000101`, `100101`, `010101`, `110101`) so one shift strips any of
  them, and `readRelationType:143-155` **reconstructs** those 6 bits from the 3-bit band plus the 1
  direction bit. `SCHEMA_COUNT_BOUND = 1L << (63 - 6 - 3) = 1 << 54` (`IDManager:410`) is sized so
  `count << 1` plus a 3-bit prefix still fits a positive long. **Lesson 08's and 09's loose ends,
  closed.** The inline path differs by two bits on purpose: `writeInlineRelationType:168-171` uses
  `stripRelationTypePadding` = `>>> 4`, keeping property-vs-edge and user-vs-system, because an inline
  key id has no band to rebuild itself from.
- **The "different first-bit convention" needs no reconciling.** In a **row key** the high bits are the
  partition and the low bits are the type suffix (`getKeyID`'s javadoc, `IDManager:523-529`); in a
  **column** the high bits are the band and the **low bit of the encoded value** is the direction. The
  `writeRelationType` javadoc (`IDHandler:120-128`) lists the *logical* field order, not the byte order.
  Two different address spaces, same 64-bit vocabulary. Lesson 08's question, answered.
- **`BufferUtil.nextBiggerBuffer:193-212` cannot overflow on a type prefix**, and `padBuffer` never
  touches columns. `nextBiggerBuffer` increments **in place, no length change**, carrying right-to-left,
  and throws on carry-out unless `allowOverflow`; byte 0's top 3 bits are the band, which maxes at 5, so
  byte 0 ≤ `0xbf` and the carry always stops. A prefix *can* end in `0xff` (count 63, `EDGE_IN_DIR` →
  `70 ff`) and then yields `71 00`, still a correct exclusive bound. `padBuffer:180-187`'s only caller is
  `OrderedKeyValueStoreAdapter:143`, padding **row keys** in the lesson-05 OKVS adapter. Both items
  owed to lesson 10, closed.
- **Everything the reader needs after the sort keys is written backward.** `writePositiveBackward:217`
  and `writeVertexId(out, id, false)` lay bytes down so a reader walking from the **tail** of the column
  finds the relation id, then the adjacent vertex id, leaving the middle as the sort-key window with no
  length header. The `forward` flag tracks exactly whether the id is in the value (`true`, parsed front
  to back) or in the column (`false`); `readVertexId:218-234` peeks `getByte(position-1)` for
  `STRING_ID_MARKER` when reading backward. This is what makes `ImplicitKey.ADJACENT_ID` and
  `ImplicitKey.JANUSGRAPHID` addressable as pseudo-sort-keys in `getQuery:484-496`: they are the
  backward-written tail fields, and `:466-471` pins their only legal positions — `ADJACENT_ID` exactly
  at `i == sortKeyIDs.length`, `JANUSGRAPHID` at `sortKeyIDs.length` or `+1`, both only for non-unique
  directions. That is the contract `BasicVertexCentricQueryBuilder:683-690`'s extended sort key must
  satisfy — **lesson 20's starting point.**
- **`InlineType` is two independent booleans, not three cases.** `KEY` = no inline key id + byte-ordered
  (sort keys, in the column); `SIGNATURE` = no inline key id, not byte-ordered (in the value); `NORMAL`
  = inline key id, not byte-ordered (everything else). `:406`'s assert means a generically typed key
  (`dataType() == Object.class`, written with `writeClassAndObject`) can **only** be a `NORMAL` inline
  property — never a sort key, never a signature field. Remaining properties are `Arrays.sort`ed by id
  (`:369-371`) so that two serializations of the same relation are byte-identical, which
  `acquireEdgeLock` (`StandardJanusGraph:900-901`) depends on.
- **`DESC` is a one's complement over exactly `[keyStartPos, keyEndPos)`.**
  `WriteByteBuffer.getStaticBufferFlipBytes:146-153` is `b.put(i,(byte)~b.get(i))` over that half-open
  window only — not the prefix (so a DESC type stays in its band and its slice stays an exact prefix
  match), not the tail (so tail-reading is order-independent). For a constrained type
  `keyStartPos == keyEndPos` and the flip is a no-op; `getStaticBuffer()` **is**
  `getStaticBufferFlipBytes(0,0)` (`:142`), so `EdgeSerializer:378-380`'s ternary is cosmetic.
- **The missing-value marker is correct in both sort orders.** `StandardSerializer.writesMissingValuesLast:264-267`
  plus `SerializerTest:139-155`: a missing value of any order-preserving type is the single byte `0xff`,
  above every real value (and `Class`, `Instant`, `byte[]` are excluded). ASC (`:511-516`):
  `sliceEnd = [prefix][0xff]` with `endInclusive = false`, so missing values are excluded at the top.
  DESC (`:518-523`): the `0xff` is **inside the flipped window** and becomes `0x00`, so missing values
  sort **first**, and `if (!endInclusive) sliceStart = nextBiggerBuffer(...)` bumps the lower bound to
  `[prefix][0x01]` — excluded at the bottom. The javadoc at `:554-561` is order-neutral and right; only
  the inline comment at `:504-505` ("written as -1 after the values") is ASC-phrased.
- **`EdgeSerializer:481-483` is dead code with a bad failure mode.** `RangeInterval.isEmpty():110-115`
  means *unsatisfiable*; breaking out leaves `sliceStart == null`, which falls into `:533` and produces a
  slice over the **whole type band** — maximum over-read instead of nothing. It never fires:
  `BasicVertexCentricQueryBuilder:607-610` returns `BaseVertexCentricQuery.emptyQuery()` for any empty
  interval before `getQuery` is ever called. PR candidate 17.
- **Two columns per edge, two different shapes.** `EdgeDirection.fromPosition`: **0 → OUT, 1 → IN**.
  `StandardJanusGraph:1126-1149` loops `pos` over `edge.getArity()` and writes only the position whose
  endpoint is the row's vertex — which is where the data-model doc's "each edge has to be stored twice"
  physically happens. For a `MANY2ONE` edge the OUT column is bare-prefix and the IN column carries the
  adjacent vertex id: **same edge, two layouts.** Three other callers write a column outside the commit
  path: `TransactionLogHeader:86-93` (always `position = 0`), `IndexRecordUtil:454-462` (inline index
  fields, always `0`), and `IndexRepairJob:211-219`.

### Verified by lesson 09 — compressed; full block in git history at `1bbfec7c`

- **The default conflict-avoidance tag is 4 bits wide, `FIXED`, and always zero.**
  `ids.authority.conflict-avoidance-tag-bits` = **4** (`ConfigOption.Type.FIXED`),
  `conflict-avoidance-mode` = `NONE`, `conflict-avoidance-tag` = **0**, so
  `ConsistentKeyIDAuthority:191/:200` set `randomizeUniqueId = false`, `uniqueId = 0` and
  `ConsistentKeyIDBlock.getId:486-491` returns `((start+i) << 4) + 0`. **Every count JanusGraph assigns
  is a multiple of 16.** At pb=5 with `NormalVertex`: count 16 → id **4096** at partition 0, **4232** at
  partition 17; counts 32/48/64 → 8192/12288/16384. Consecutive vertices in one partition are **128**
  apart, not 8. At tag-bits 0 the first id would be 256. Lessons 10 and 11 both chain off this (one byte
  of prefix per cell; the 3-byte backward floor paid on the very first id), and **lesson 27 must assume
  it** when reasoning about CQL token distribution.
- **The reachable count space, exact.** `claimIDBlock:287-290`:
  `maxAvailableBits = (unsignedBitLength(idUpperBound) - 1) - uniqueIdBitWidth`. At pb=5, tag-bits 4:
  `NORMAL_VERTEX`/`UNMODIFIABLE_VERTEX`/`PARTITIONED_VERTEX` **2^51** of 2^55; `RELATION` **2^54** of
  2^58; `SCHEMA` **2^50** of 2^54. **15/16 of every space is unreachable**, and the option is `FIXED`, so
  it cannot be reclaimed.
- **Five namespaces, and the namespace byte is an enum ordinal.** `PoolType.getIDNamespace()` is
  `ordinal()` (`VertexIDAssigner:458-460`): 0 `NORMAL_VERTEX`, 1 `UNMODIFIABLE_VERTEX`,
  2 `PARTITIONED_VERTEX`, 3 `RELATION`, 4 `SCHEMA`. **Reordering the enum reinterprets every existing
  graph's id rows.** `hasOnePerPartition()` is true for 0, 1 and 3 only, hence
  `renewalThreads = 3 * partitionIdBound + 2` (`:123`) = **98** at `cluster.max-partitions = 32` against
  a live-pool ceiling of **32**, and **196,610** at 65536 (lazily created, `allowCoreThreadTimeOut(true)`).
- **`BASE_ID = 1`** (`AbstractIDAuthority:45`), so **no count 0 is ever offered** — which is what
  actually protects `EDGESTORE_MIN_KEY`'s unreachability (lesson 08's question), not `addPadding`.
- **`SCHEMA` (block 50) and `PARTITIONED_VERTEX` (block 100) permanently double-buffer**, because
  `renewBlockIndex = max(0, numIds - max(RENEW_ID_COUNT=100, round(numIds*0.3)))` is 0 when
  `RENEW_ID_COUNT = 100` exceeds the block size. Defaults: `ids.block-size = 10000`,
  `ids.renew-percentage = 0.3`, `ids.renew-timeout = 120 s`, `ids.authority.wait-time = 300 ms`,
  `ids.num-partitions = 10`.
- **Owed to lesson 27, still owed:** the ids-store column layout — `[-(end)][timestamp][uid]` with an
  empty value, slice bounds `zeroBuffer(1)` to `oneBuffer(17)` — is the smallest real column layout in
  the codebase and the cleanest thing to compare against `CQLKeyColumnValueStore`. The negated counter is
  what makes `getCurrentID`'s `setLimit(5)` correct.
- **PR candidates 11-14 all come from this lesson** and carry forward unchanged: the
  `timeout.getNano()` give-up message that prints 0 for any whole-second `Duration` (11), the rollback
  path's character-for-character duplicate `TransactionalProvider` (12), `IDAuthorityTest`'s two
  contiguity assertions that have never executed because no `configs()` parameterization sets tag-bits to
  0 (13), and `accessed()`'s 33.0 ns/op `volatile` write for a `getLastAccess()` with no caller (14).

### PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 18-20 are new this run** (lesson 11); **18 is the most
substantial the track has found** and 20 should wait on lesson 12. **11 is still the smallest.**
Candidates 1-17 carry forward and were **not** re-verified this run: 15-17 were verified at this pinned
commit on 2026-11-08, 11-14 on 2026-11-07, 8-10 on 2026-11-06, 1 and 7 on 2026-11-05, 2-4 and 6 on
2026-11-04, and 5 still needs its experiment run.

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
20. **Bound the ASC sort-key read as the DESC read is bounded — but wait for lesson 12.**
    `EdgeSerializer:165-175` sends DESC through `in.subrange(keyLength, true)`, whose `limit` is exactly
    the sort-key window, so `StaticArrayBuffer.require` throws past it; ASC reads **in place** out of
    `in` itself and nothing compares the bytes consumed against `keyLength`, so an over-reading sort-key
    serializer would read into the `vid←`/`rid←` tail and return silently wrong values, with
    `in.movePositionTo(currentPos)` discarding the evidence. The fix is one line —
    `in.subrange(keyLength, def.getSortOrder() == Order.DESC)`, since the non-inverting `subrange`
    returns a **view** with no byte copy — at the cost of two short-lived objects where ASC currently
    allocates none. **Only worth filing if the failure mode is reachable**, which is lesson 12's job to
    establish: if every `isOrderPreservingDatatype` serializer is length-unambiguous, this is a style
    change. Note the inverting `subrange` *does* allocate (`new byte[length]` + arraycopy + `(byte)~b`
    loop + `StaticArrayBuffer` + `ReadArrayBuffer`), so every DESC index replica — which can never be
    complete at the header — pays three allocations and a copy on its first property read.

### Still open, and who owns it

- **Closed by lesson 11:** lesson 10's carried question (the layout does not decide completeness; the
  four-way test at `:201-202` does, and index replicas are excluded by clause 2 forever); why the tail
  ids are written backward (order preservation, not readability — zero inversions over 400,000 values);
  `endKeyPos`'s exclusivity and the off-by-one in `:169`'s comment; the `TIMESTAMP`-vs-`ROW_KEY`
  asymmetry (`hasPropertyMetaData` tests `ImplicitKey.MetaData2ImplicitKey` membership, not
  `hasMetaData`); the `RelationCache` identity assertions (`assertSame` throughout
  `EdgeSerializerTest.checkParsedOnce`, which pins that a full parse is the one instance every later read
  returns); `readInlineTypes`/`hasPropertyMetaData`; and that `SIMPLE` is constrained, correcting lesson
  10's arrow on that row.
- **Owed to lesson 12 by lesson 11, and it decides a PR:** is any `readObjectByteOrder` implementation
  length-ambiguous, i.e. can it consume a different number of bytes than `writeObjectByteOrder` wrote? If
  not, PR candidate 20 is a style change and should be dropped. Second question, independent: does
  `VariableString.writeBackward` place its high-bit marker at the **high** address? `readVertexId(in,
  false)` peeks `getByte(position-1)` for `STRING_ID_MARKER`, and the long side is safe because a
  backward long's last byte is always `≤ 0x7f` — the string side is assumed, not verified.
- **Owed to lesson 13 by lesson 11:** `parseRelation:113` calls `tx.getExistingRelationType(typeId)`
  before it knows anything else about the cell, so a schema vertex's own row must be decodable without
  that lookup recursing forever. Find where the recursion bottoms out; it is the one place the format
  *is* self-describing.
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
- **Owed to lesson 12 by lesson 10:** `writeObjectByteOrder` vs `writeObject` vs `writeClassAndObject`,
  the order-preserving serializers, and `writesMissingValuesLast`. The question that matters for the
  column layout: **is every `isOrderPreservingDatatype` also fixed-width?** A variable-width
  order-preserving sort key would make the sort-key window's end ambiguous without the backward-tail
  trick, so the two mechanisms are coupled. `SerializerTest:139-170` (the `0xff` missing-value marker and
  the single-value NaN encoding after `+∞`) is the entry point.
- **Owed to lesson 13 by lesson 10:** verify rather than assume that a schema vertex's own row uses the
  same six bands. `BaseRelationType.isInvisibleType():62-64` returns an unconditional `true` and
  `BaseKey`/`BaseLabel` ids are system ids, so schema-definition relations should land in bands **0 and
  1** — but `ImplicitKey.isInvisibleType():154-156` returns `false`, so implicit keys are user-band, and
  `writeRelation:364-365` filters `ImplicitKey` out of the remaining-properties loop entirely.
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
  `KeyColumnValueStore`'s javadoc and **none** of the enforcement is.
- **tdd.cat, 2026-11-09: the two skipped days published, and the fallback precedent paid off.** At
  ~03:40 UTC the front page served **2026-11-08** (22 stories), labelled *Latest*, and `/archive/`
  reported **133 editions** with 2026-11-08 as the newest and the 9th shown unlinked. So **no 2026-11-09
  edition exists**, and the two days lesson 10 found missing (2026-11-07, 2026-11-08) have both since
  been published — the archive count went 131 → 133. Lesson 11 therefore digested the **2026-11-08**
  edition, which no run had covered, rather than falling back a second time. **Practical consequence for
  the next run:** the site is running roughly a day behind and backfills, so (a) always check
  `/archive/`'s maximum, not just the front page; (b) before digesting, check the session log for the
  newest edition already covered — a date the front page serves may be one a previous run digested, and a
  date a previous run reported as non-existent may now exist; (c) the uncovered-item fallback remains the
  last resort, and the 2026-11-06 edition now has two items spent (Infino ids, Postgres `SKIP LOCKED`
  queues). The front-page-vs-archive lag has now shown three different patterns in four days; neither
  source is dependable alone. The filter-bar rendering bug ("No stories match the selected filters")
  was **not** observed this run.

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
