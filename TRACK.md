<!--
track-state
subject: JanusGraph
mode: contributor-reading
started: 2026-10-30
lessons-total: 28
lessons-done: 3
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase (Contributor Reading)

Sequential reading of the JanusGraph source, one session per day, each building on the last.
**This track is different from SQLite and PostgreSQL.** Those were lessons *about* a system. This
one is a guided **reading session** in a codebase Aryant intends to contribute to before the
January 2027 LFX Mentorship applications open. The generated part of each entry is a short guide;
the other half — `## My Notes` — is written by Aryant after reading. A session whose `My Notes`
stays empty did not count.

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

## Session Log

Newest first. The next run reads the most recent entry's `## My Notes` before writing.

- **03 — 2026-11-01** · [`2026/11/2026-11-01-janusgraph-store-manager-features/README.md`](2026/11/2026-11-01-janusgraph-store-manager-features/README.md) · read `KeyColumnValueStoreManager` (whole), `StoreManager` (whole), `StoreFeatures` L31–257, `StandardStoreFeatures` (`hasScan` L54, `Builder` from L180, the private ctor), `KeyColumnValueStore` L185–227 (the four `getKeys` overloads session 02 deferred), `Backend` L249–287 + L339–343, `InMemoryStoreManager` L64–72 + L134–135, plus `InMemoryStoreManagerTest` and `FeatureRequirementCondition`. Six questions posed, none answered in the entry. Hands-on: `InMemoryStoreManagerTest#testStoreCycle`, a throwaway `dumpFeatures()` with every value predicted first, and breakpoints on `Backend.java:261`/`:271` under `InMemoryGraphTest`. Radar: #2268, #3390, #1128. **Sessions 01 and 02 both left `My Notes` empty — streak 2; today's entry carries the one-line notice and has no `From Your Last Notes`.** **`My Notes` awaiting Aryant.**
- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery` (whole), `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, plus `KeyColumnValueStoreTest` L862–938 and `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered in the entry. Hands-on: `InMemoryKeyColumnValueStoreTest#getSliceRespectsAllBoundsInclusionArguments` and `#getSliceRespectsColumnLimit`. Radar: #3864, #3816, PR #4086. **Session 01's `My Notes` were empty — one consecutive empty session; today's entry carries the one-line notice.** **`My Notes` never filled.**
- **01 — 2026-10-30** · [`2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md`](2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md) · read `data-model.md`, `BUILDING.md`, `CONTRIBUTING.md`, `InMemoryStoreManager`, `StandardStoreManager` L28–33, `InMemoryGraphTest`. Six questions posed, none answered in the entry. Hands-on: `WholeRowDeletionOptimizationTest`. Radar: #3013, #225, #1573. **`My Notes` left empty.**

## Syllabus

### Part I — Orientation and the storage contract (`diskstorage/`, `janusgraph-inmemory`)

- [x] 01 — Setup and the data model: fork, build `janusgraph-inmemory`, module map, and `docs/advanced-topics/data-model.md` (edgestore rows, relation columns)
- [x] 02 — `KeyColumnValueStore`: `getSlice`, `mutate`, `acquireLock`, plus `StaticBuffer`, `Entry`, `SliceQuery`, `KeySliceQuery`
- [x] 03 — `KeyColumnValueStoreManager` and `StoreFeatures`: how a backend advertises ordered scans, locking, batch mutation, and where core branches on them
- [ ] 04 — `InMemoryKeyColumnValueStore`: answering a slice query over a sorted in-memory structure
- [ ] 05 — `OrderedKeyValueStoreManagerAdapter` and `BerkeleyJEKeyValueStore`: faking key-column-value on an ordered key-value store
- [ ] 06 — The test harness: `JanusGraphTest`, `InMemoryGraphTest`, backend test suites, and how a PR's test is expected to look
- [ ] 07 — `Backend` and `BackendTransaction`: the stores that get opened (edgestore, graphindex, system_properties, ids, logs) and the cache/locker wrappers

### Part II — Graph ⇄ bytes (`graphdb/idmanagement`, `graphdb/database`)

- [ ] 08 — `IDManager`: vertex ID bit layout, partition bits, vertex-type suffixes, and how row keys are derived
- [ ] 09 — `VertexIDAssigner` and the ID authority: block allocation from the ids store without per-insert coordination
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

**Syllabus unchanged this run.** No reordering, no splits. The 28 items stand as written on 2026-10-30.

- **Empty-notes streak: 2.** Sessions 01 *and* 02 left `My Notes` as the untouched template. Session 03's entry
  carries the one-line notice and has no `From Your Last Notes` section, because there was nothing to answer.
  **Session 04 must re-check session 03's notes and increment to 3 or reset to 0.** Three consecutive empty sessions
  is the point at which the track's premise — that the reader's half is where the learning happens — stops being
  true, and session 04 should say so plainly rather than repeating the same sentence a third time.
- **Session 03 confirmed, at the pinned commit:** `KeyColumnValueStore.java` is **245 lines, not 221** — session 02's
  entry said 221. Its *line anchors* were all correct (`acquireLock` really is `L183`); only the total was wrong. The
  four `getKeys` overloads are `L185-L187` (a `default` throwing `NotImplementedException`), `L200` (`KeyRangeQuery`,
  "only supported by stores which keep keys in byte-order"), `L213` (`SliceQuery`, "stores which do **not**"), and
  `L227` (`MultiSlicesQuery`, "mandatory for stores which do not guaranty key orders while running parallel scans").
- **`StoreFeatures` declaration lines**, for sessions 04, 05, 07, 18 and 27 to cite rather than re-derive: `hasScan`
  37, `hasUnorderedScan` 43, `hasConsistentScan` 51, `hasOrderedScan` 57, `hasMultiQuery` 64, `hasLocking` 71,
  `hasBatchMutation` 78, `isKeyOrdered` 85, `getScanKeyOrder` 103-105 (`default`, returns null),
  `hasLocalKeyPartition` 127, `isKeyConsistent` 138, `hasCellTTL` 171, `supportsPersistence` 195,
  `hasOptimisticLocking` 247, `hasOptimizedWholeRowDeletion` 257.
- **Every feature consumer in `Backend.java` lives in two windows** — the fact session 07 needs: `L249`
  (`storeFeatures = storeManager.getFeatures()`), `L261-263` (`!hasBatchMutation()` → `bufferSize =
  Integer.MAX_VALUE`, consumed by the `CacheTransaction` built at `L619`), `L271-276` (`!hasLocking()` → wrap in
  `ExpectedValueCheckingStoreManager`, with `L272` asserting `isKeyConsistent()`), `L278-287` (`hasMultiQuery()` →
  skip the `PARALLEL_BACKEND_OPS` thread pool), and `L339-343` (`isKeyConsistent()` → `ConsistentKeyIDAuthority`,
  else `IllegalStateException`). Nothing else in those 775 lines reads a flag. Session 18 owns
  `ExpectedValueCheckingStoreManager`; session 09 owns `ConsistentKeyIDAuthority`.
- **`StandardStoreFeatures.Builder` defaults are not all false**, contrary to its own javadoc at `L208`
  ("everything disabled/unsupported/false/null"): `consistentScan = true` (`L182`), `supportsPersist = true`
  (`L198`), `supportsInterruption = true` (`L203`). The private constructor takes **24 positional parameters**, 18 of
  them `boolean`. If a lightweight PR candidate is ever wanted, correcting that javadoc sentence is a one-line
  documentation fix — but do **not** open it for him.
- **Session 03's six questions, left unanswered on purpose**, so a later session knows what is still open: (1)
  in-memory declares both `orderedScan` and `unorderedScan` though the two `getKeys` javadocs are mutually exclusive;
  (2) `hasBatchMutation()` is false while `mutateMany` is fully implemented (`InMemoryStoreManager.java:117-131`); (3)
  `hasLocking()` vs `hasOptimisticLocking()`, and which branch the in-memory backend takes at `Backend.java:271`; (4)
  `InMemoryStoreManager.java:71` uses the **single-arg** `keyConsistent(Configuration)`, so
  `localKeyConsistentTxConfig` stays null; (5) `getLocalKeyPartition()` throws unconditionally at `L134-135` while
  `hasLocalKeyPartition()` is false — is the javadoc promise at `StoreFeatures.java:118-127` enforced or only
  documented; (6) what `getScanKeyOrder()` returning null costs a backend that *is* ordered. **Sessions 04 and 05
  answer 1 and 4 almost incidentally** — do not pre-empt them in the entry text; answer them in
  `From Your Last Notes` only if Aryant raises them.
- **`FeatureRequirementCondition.java:37-59`** is the test-skip mechanism: `@FeatureFlag` → `TestAbortedException`.
  Only four flags are wired (`CellTtl`, `Scan`, `OrderedScan`, `UnorderedScan`); the `default` arm throws
  `UnsupportedOperationException`. Session 06 (the test harness) owns this properly.
- **Issue search this run:** nothing open is *about* `StoreFeatures` — both the issue and the PR searches returned
  zero. Adjacent and used: **#2268** (LockEntry TTL, `area/core`, unassigned), **#3390** (CQLPagingIterator for
  Amazon Keyspace, `kind/bug/confirmed`, unassigned), **#1128** (berkeleyje in-memory, assigned to `amcp`, untouched
  since 2018). Repo-wide `good first issue` is **still exactly four**: #3013, #3126, #3127, #1573 — unchanged for
  three runs; #3126/#3127 stay reserved for session 25. Re-run the search each session, but stop re-listing the four
  in full.
- **Still deferred:** `janusgraph-inmemory`'s paging machinery (`MultiPageEntryBuffer`, `BufferPage`,
  `BufferPageUtils`, `SinglePageEntryBuffer`, `InMemoryColumnValueStore`) → session 04. The
  `KeyColumnValueStoreTest.java:887-893` dead-loop PR candidate from session 02 is **still unverified by Aryant** and
  still must not be opened for him.
- **Entry size.** Session 03's generated part came in at ~15 KB against the mode's 5–9 KB budget, after two
  deliberate trimming passes that cut explanation and kept specifics. The overage is structural: six reading rows, a
  nine-flag table, six questions and eleven pinned permalinks. If session 04 wants to come in under budget, the lever
  is **fewer files** (three, not six), not shorter sentences.

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
