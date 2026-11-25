<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 27
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-22 at the Part V/VI boundary (lesson 24) — PERFORMED, and the pin stays
  for the FIFTH review in a row. Lessons 25, 26 and 27 were all mid-Part and correctly did NOT re-raise it.
  The method is blob-SHA identity and nothing weaker: every file cited by line number has its git blob SHA
  computed locally and compared against what the API reported, the cheap form being a directory listing
  with fields: ["name","sha","size"] which returns every file's blob SHA in ONE call. Lesson 27 applied it
  to TEN files and all ten matched on the first attempt, which is incidental confirmation that the pin has
  not moved under the track. Lesson 27 also established that the plain `text` value of an over-limit
  get_file_contents result is LOSSY, not merely banner-prefixed, and that the resource path matches
  byte-for-byte: see the mechanical notes.
  NEXT REVIEW: the Part VI boundary — lesson 28 MUST perform it, running list_commits on master rather
  than assuming either answer, and recording the result here.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (candidates 1-64, FROZEN, plus the "Still open, and who owns
  it" ledger) and tracks/janusgraph-registers-2.md (candidates 65-78, next free 79)
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open. **Part VI opened on 2026-11-23 with lesson 25. Lessons 26 and 27 shipped 2026-11-24 and 2026-11-25. One lesson remains: 28, and then the capstone.**

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
time. Lessons 25 and 26 were mid-Part and correctly did not re-raise it; lesson 27 must not either.** `list_commits` on `master` returns the same three newest commits as at lessons 13, 18 and 22 —
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
lesson 16 on 2026-11-17, lesson 17 on 2026-11-20, lesson 20 on 2026-11-21, lesson 21 on 2026-11-22,
lesson 22 on 2026-11-23, and **lesson 23's paragraph was compressed this run with its
`Verified by lesson 23` block removed**. The full paragraphs remain verbatim in git history at the commit
each line names. **This run compressed lesson 24's paragraph and removed its `Verified by lesson 24` block.** Next run, lesson 25's block and paragraph are the ones that go.

**On 2026-11-22 the one-line entries for lessons 01–19 were dropped from this log; lesson 20's went on
2026-11-23, lesson 21's on 2026-11-24 and lesson 22's this run.** They carried
nothing the rest of the repository does not carry better — the **Syllabus** below lists all 28 lesson
titles as taught, and the root [`README.md`](README.md) index links every entry and every digest by date
and number — while costing ~23 KB of a file that is read in full at the start of every run and
re-emitted in full on every push. Their full text is in git history at the commit immediately preceding
the 2026-11-22 entry. Entries roll off this log one per run from here; the window is the five most
recent lessons.

- **27 — 2026-11-25** · [`2026/11/2026-11-25-janusgraph-cql-store-vs-inmemory/README.md`](2026/11/2026-11-25-janusgraph-cql-store-vs-inmemory/README.md) · **Mid-Part, so the re-pin question was correctly NOT re-raised — but lesson 28 MUST perform the Part VI review.** Read the CQL storage backend against the in-memory store at the pinned commit. **Blob-SHA verified against the API's own directory listings and cited by line number** (all ten matched on the first attempt): `CQLKeyColumnValueStore` (873 lines, 47,914 bytes = `f11ea637`; cited at `:123-127`, `:141-154`, `:156-170`, `:200-201`, `:211-343`, `:217-222`, `:224-244`, `:234-244`, `:248-263`, `:255-256`, `:265-303`, `:284-285`, `:291-299`, `:305`, `:323-328`, `:332-333`, `:336-338`, `:339-340`, `:355-368`, `:370-379`, `:387-397`, `:399-411`, `:413-424`, `:432-437`, `:450-480`, `:482-491`, `:493-523`, `:499-504`, `:503-523`, `:516-520`, `:525-545`, `:529-531`, `:539-542`, `:547-583`, `:595-625`, `:613-617`, `:641-652`, `:673-684`, `:686-695`, `:718-750`, `:731-737`, `:752-774`, `:776-789`, `:802-813`, `:815-818`, `:820-872`, `:829-872`, `:840-849`, `:851-863`), `CQLStoreManager` (374, 18,740 = `09f7b055`; `:104`, `:157-158`, `:311-358`, `:346-349`, `:352-358`, `:356`), `CQLConfigOptions` (48,346 = `2f1adabe`; `:47-55`, `:64-76`, `:85-89`, `:91-102`, `:105-110`, `:113-118`, `:120-130`, `:237-251`, `:253-265`, `:267-280`, `:844-862`), `CQLStoreFeaturesBuilder` (129, 7,091 = `a13eacac`; `:58-67`, `:62`, `:63`, `:65`, `:68-70`, `:80-87`, `:88-119`, `:94-100`, `:101-104`), `CQLTransaction` (60, 2,609 = `349bf72b`; `:30-47`, `:49-53`), `CQLColValGetter` (65, 2,423 = `66ea2658`; `:26-64`, `:59-60`), `AbstractCQLMutateManyFunction` (100, 5,614 = `ea850c4e`; `:35-41`, `:39-40`, `:59-91`, `:63`, `:68-77`, `:78-90`), `CQLMutateManyUnloggedFunction` (125, 6,189 = `7d3d3ee6`; `:71-124`, `:84`, `:88`, `:96-109`, `:122-123`), `CQLMutateManyLoggedFunction` (106, 4,860 = `8a9a9f7d`; `:65-104`, `:71-85`, `:95-102`), `InMemoryKeyColumnValueStore` (471, 19,258 = `83650bf4`; `:85-158`, `:94`, `:99`, `:109-114`, `:117-124`, `:127-129`, `:131-134`, `:137-144`, `:146-149`). **Read inline, member-name citations only:** `InMemoryColumnValueStore` (`5569f1a0`), `DistributedStoreManager` (`89ed24d5`), `CQLSliceQueryUtil` (`2efd226b`). **Seen only as `search_code` fragments:** `SliceQuery`, `EdgeSerializer`, `CQLSliceQueryUtilTest`. Three directory listings with `fields: ["name","sha","size"]`; one `search_code` query, returning three results; two delegated transcription batches, nine files, all blob-SHA verified. **Two external pages fetched:** the Cassandra Dynamo architecture page and the PlanetScale failover post. Five PR candidates logged (**74-78**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md). Daily Diff: **the 2026-11-25 edition had not published at 03:33 UTC**, and the run discovered that **`https://tdd.cat/{date}/` works where `{date}.md` 404s** — so the digest covered the latest published and previously uncovered **2026-11-24** edition (59 items, read in full across three offset fetches), with a deep dive on **PlanetScale Neki's leaseless unplanned failover for sharded Postgres**.

  **Findings.** **THE finding: the CQL backend's entire write-ordering guarantee is one bit of the commit timestamp, and the in-memory store reaches the same semantics by the exactly opposite route.** Take the commonest interesting mutation — a transaction that deletes column `c` and re-adds `c` with a new value, which is every single-cardinality property update. `InMemoryColumnValueStore.mutate` resolves it **structurally and client-side**: `synchronized`, sort the additions, then drop every deletion found among them by `binarySearch`, so the tombstone never reaches the buffer at all. `CQLKeyColumnValueStore.mutate` (`:673-676`) resolves **nothing** — it wraps its arguments and delegates — and the resolution happens in Cassandra, because `MaskedTimestamp.getDeletionTime` is `times.getTime(t) & 0xFFFFFFFFFFFFFFFEL` and `getAdditionTime` is the same with `| 1L`. **The addition time is the deletion time plus exactly one, by bit arithmetic**: the least significant bit of the provider's resolution is reserved as a delete/add discriminator, and per-column last-write-wins does the rest. That is also the answer to lesson 26's handover question — **a cell write at a fixed `(key, column, timestamp)` is an assignment, so replaying it is free; a JSON array append is an accumulation, so it is not.** The dividing line between the two write paths is not distributed-versus-local, it is assignment-versus-accumulation, and `storage.cql.idempotent-writes` (default `true`) is the one-line consequence. **Why the mechanism has to be temporal rather than structural: a CQL mutation is not one request.** The default unlogged path does `Iterator.concat(deletions, additions).grouped(20)` and fires each group as its own async `UNLOGGED` batch, so a key with more than 20 column operations is sent as several concurrent requests whose arrival order is unspecified — and the result is still deterministic. **Three old debts discharged.** Lesson 18's: `fb.locking(useExternalLocking)` with the option defaulting `false`, so `ConsistentKeyLocker` **does** run on Cassandra — and setting it `true` makes `acquireLock` a method whose body is only its inverted guard, so the option **disables locking rather than delegating it**, which the option's own description admits in the words "read uncommitted" and "phantom reads". Lesson 16's: batch ordering is **implicit in the timestamps, not explicit in the batch**, and the `LOGGED` alternative is one unbounded batch across every table and key. Lesson 20's `setDirectColumnByStartOnlyAllowed`: its only consumer in the codebase is `CQLSliceQueryUtil`, whose grouping rule `groupSize = min(slice-limit, limit)` means **a limit-1 direct-equality query forms a group of one, so `IN`-grouping cannot fire for it** — sound, because CQL's `LIMIT` is per result set rather than per `IN` element. **Fourth, the module's character is graceful degradation**: five places — token-range scans, `PER PARTITION LIMIT`, parallel scan, Cassandra-5 compression option names, and `cdc=true` on a pre-existing table — where an unsupported backend produces one `warn` at store open and a quietly different store for the rest of its life, on the stated principle that a scan-only optimisation must not fail store open and with it every graph open. Reading those five comments is the fastest way to understand what "CQL-compatible" costs. **Fifth, the most promising defect-shaped thing in the module is a reachability question, not a finding:** with `graph.assign-timestamp=false` the statements carry no `USING TIMESTAMP` at all, and the unlogged path's `.grouped(20)` can then put a delete and its matching insert into different concurrent batches — at which point the whole correctness argument above stops holding. That is candidate 74, and it is deliberately **not** stated as a confirmed defect.

- **26 — 2026-11-24** · [`2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md`](2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md) · **Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the mixed-index SPI and its Elasticsearch implementation at the pinned commit. **Blob-SHA verified and cited by line number:** `ElasticSearchIndex` (2,028 lines, 119,295 bytes = `d80949e3`; cited at `:120`, `:185-188`, `:215-222`, `:224-232`, `:234-250`, `:252-266`, `:526-536`, `:543-562`, `:564-566`, `:568-576`, `:770-800`, `:802-813`, `:815-824`, `:826-835`, `:837-844`, `:846-854`, `:855-876`, `:866-868`, `:869-873`, `:874-875`, `:881-888`, `:897-916`, `:932-958`, `:1010-1047`, `:1015-1017`, `:1018-1023`, `:1025-1038`, `:1039-1046`, `:1162-1167`, `:1169-1171`, `:1173-1217`, `:1179-1192`, `:1197-1201`, `:1219-1238`, `:1275-1381`, `:1292-1295`, `:1300-1306`, `:1307-1350`, `:1314-1315`, `:1319`, `:1322`, `:1326-1330`, `:1336`, `:1344`, `:1346`, `:1352-1356`, `:1364-1368`, `:1369-1380`, `:1371-1372`, `:1383-1406`, `:1408-1422`, `:1424-1462`, `:1436-1441`, `:1451-1452`, `:1457-1458`, `:1459-1461`, `:1464-1468`, `:1507-1708`, `:1709-1728`, `:1718`, `:1730-1735`, `:1794-1804`, `:1806-1825`, `:1816`, `:1827-1847`, `:1854-1856`, `:1864-1870`, `:1872-1881`, `:1883-1925`, `:1928-1942`, `:1944-1956`, `:1959-1962`, `:1964-1967`, `:1995-2007`). **Read inline, member-name citations only:** `IndexProvider` (`b7b342bb`), `IndexFeatures` (`6651e246`), `IndexInformation` (`ded18cc9`), `IndexMutation` (`d2c44e46`), `IndexSelectionUtil` (`3ec0ec2a`), `AbstractESCompat` (`89543cd9`), and all four compat subclasses read in full — `ES6Compat` (`d686ce02`), `ES7Compat` (`fb0efe32`), `ES8Compat` (`bbd2b413`), `ES9Compat` (`73af9fc2`). **Seen only as `search_code` fragments:** `AdjacentVertexHasUniquePropertyOptimizerStrategy`, `AbstractIndexSelectionStrategy`, `BruteForceIndexSelectionStrategy`, `ApproximateIndexSelectionStrategy`, `IndexSerializer`, `HasStepFolder`, `IndexSelectionUtilTest`. Three directory listings with `fields: ["name","sha","size"]`; two `search_code` queries, both returning results. Three PR candidates logged (**71-73**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file — **the register was brought to local disk by a delegated transcription and blob-SHA verified (18,388 bytes = `642bd570`) before being appended to**, the first time that method was used on a file being edited rather than read. Daily Diff: **today's edition had not published** (`2026-11-24.md` and `2026-11-23.md` both client errors), so the **fallback pool was drawn on for the first time** — the uncovered **2026-11-20** edition, 50 items, read in full across two fetches; deep dive on **Valkey 9.2's forkless snapshots**.

  **Findings.** **THE finding: the index SPI is the only write path in JanusGraph that is not idempotent by construction, and `LIST` cardinality is the entire reason.** Audit the five mutation primitives against the Painless scripts and exactly two are non-idempotent: `ctx._source[field].add(value)` for a `LIST` field, which appends a duplicate on a second application, and `remove(indexOf(value))`, which removes another occurrence or none. `SET` is saved by one guard — `field.cardinality != 'SET' || indexOf(value) == -1` — and every `SINGLE` operation is naturally idempotent (`ctx._source.remove(name)` on an absent key, and `putAll(params.doc)` rewriting the same value). **The fix reached up into the core rather than down into the backend**: `IndexMutation` gained a `completeDocument` field carrying the element's whole indexed state, *"supplied once, from the element's state at commit"*, with the design note that *"reading the element again would read the same state"* — so it costs no read. `addUpdatesCarryingTheCompleteDocument` (`:855-876`) then collapses every change to one document into **one** update whose upsert is the complete document, exploiting three mechanisms at once: Elasticsearch **writes the upsert and skips the script** when the document is missing, so "recreate whole" and "apply the delta" become the same request; one item per document is the **maximum atomicity** the bulk API offers; and a retried item therefore never replays a *part* that had applied. The in-source comment at `:846-854` is the clearest statement of the problem in the codebase. Second: **`mutate` has three request shapes and the non-idempotent one is still live.** Shape A is a whole-document delete, guarded on `hasDeletions() && isDeleted()`; shape B is the single complete-document update; shape C — reached whenever `completeDocument` is `null` for an existing document — emits up to **three** separate bulk items for one `_id` (field-deletion script `:1314-1315`, addition script `:1336`, `{"doc": …}` merge `:1344`), with `upsert` deliberately `null` when deletions are present (`:1326-1330`). That is exactly the shape shape B's own javadoc argues against (candidate **73**, reachability untraced: `IndexSerializer` and `IndexTransaction` were not read). Third: **the retry machinery and the idempotence machinery are a matched pair.** `retainUnappliedDocuments` (`:770-800`) **mutates the caller's map in place** — a contract declared in `IndexProvider.mutate`'s own javadoc, *"a caller must not rely on the map being unchanged after a failure"* — in four independent steps, each wrapped by `narrowQuietly` (`:805-813`) so that an immutable map simply resends that store whole, and guarded by `namesPendingDocuments` (`:826-835`) so that a response naming documents the mutation no longer holds narrows nothing. Every step fails **toward** resending more, which is only sound because shape B makes the duplicate work correct. The residual hazard the lesson does **not** claim either way: shape B guarantees all-or-nothing within a document, not script idempotence against a document that already holds the outcome, so a bulk item that applied and was then reported failed still double-applies a `LIST` add. Fourth: **`register` is a non-transactional side effect of a transaction.** Called synchronously from `ManagementSystem.addIndexKey:802` (lesson 25), it can `checkForOrCreateIndex` **and** `pushMapping` before the management transaction commits, and nothing compensates in either direction — a failed register can leave a stray ES index, and a successful register followed by a rollback leaves ES holding a mapping the schema has never heard of. ES mappings being additive and practically irremovable, the drift accumulates one way, with no documented repair. Fifth, **debt (d) is settled and the answer is "no filter at all"**: `IndexSelectionUtil` has six public static methods and **not one reference to `SchemaStatus`**; `existsMatchingIndex`/`getMatchingIndexes` apply only the caller's `Predicate<IndexType>`, and `AdjacentVertexHasUniquePropertyOptimizerStrategy` passes `indexType -> indexType.isCompositeIndex() && …`. So a unique composite index at `INSTALLED`, `REGISTERED` or `DISABLED` still drives the rewrite although the composite read gate is `== ENABLED`. **The absence is verified; the consequence is not** (candidate **72**, Hands-On B). Sixth, a **correction to the handover**: there is no `supportsExistsQuery()` in `IndexFeatures` at the pinned commit — the file was read in full and the nearest member is `supportsGeoExists()`. Seventh: **all four `*Compat` classes produce identical `IndexFeatures`**, each being `coreFeatures().setWildcardField(CUSTOM_ALL_FIELD).supportsGeoContains().build()`, so `coreFeatures()`'s `setWildcardField("_all")` is **unreachable** (candidate **71**) and version dependence lives in a boolean field, an `IllegalArgumentException` and an `unmapped_type` comment instead. `supportsDocumentTTL` is `false` for every ES version, leaving `IndexMutation.determineTTL()`'s uniform-TTL `checkArgument` with no ES consumer. Eighth, inherited item (b) settled: **Elasticsearch has no page-doubling equivalent of `SolrIndex`'s** — it sizes pages once, with three options (`limited-result-max-page-size` **250**, `adaptive-point-in-time-max-page-size` **500**, `adaptive-point-in-time-max-shards` **1**), all three carrying `positiveInt()` verifiers where lesson 24's candidate 66 found none on the core batching options. The doubling that does affect an ES-backed query is `LimitAdjustingIterator`'s, one layer up.

- **25 — 2026-11-23** · [`2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md`](2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md) · **Part VI opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the management and index-lifecycle surface at the pinned commit. **Blob-SHA verified against the directory listing and cited by line number:** `ManagementSystem` (1,815 lines, 86,864 bytes = `a722a7d9`; cited at `:190-207`, `:209-213`, `:241-268`, `:253`, `:256-257`, `:340-393`, `:365`, `:367`, `:368`, `:387`, `:389-391`, `:622-639`, `:628-639`, `:655-679`, `:704-706`, `:720-724`, `:746-767`, `:758`, `:770-808`, `:785-791`, `:791`, `:802`, `:807`, `:831-879`, `:841-846`, `:852`, `:860`, `:877`, `:967`, `:977`, `:980-1140`, `:987-996`, `:1000-1019`, `:1013-1016`, `:1026-1032`, `:1033-1051`, `:1052-1056`, `:1100-1112`, `:1130-1135`, `:1142-1147`, `:1154-1166`, `:1207-1301`, `:1227-1287`, `:1237-1242`, `:1244-1254`, `:1289-1299`, `:1303-1305`, `:1310-1319`, `:1321-1326`, `:1328-1341`, `:1334`, `:1343-1361`, `:1351`, `:1392-1459`, `:1436-1458`) and `ManagementLogger` (505 lines, 24,707 bytes = `1980d1fb`; `:71`, `:169-230`, `:176-207`, `:208-224`, `:235-237`, `:239-260`, `:275-286`, `:288-296`, `:306-315`, `:317-326`, `:328-339`, `:341-357`, `:382-441`). **Read inline, member-name citations only:** `SchemaStatus` (`7b40611d`), `SchemaAction` (`b2bf7951`), `JanusGraphIndexWrapper` (`94d0458d`), `GraphIndexStatusWatcher` (`3ebe2f6a`), `AbstractIndexStatusWatcher` (`e0ab9913`), `IndexUpdateJob` (`f1b9b521`), `IndexRepairJob` (`434abee9`). Two directory listings, one with `fields: ["name","sha","size"]` — **the cheap SHA-verification method this run introduced**. One `search_code` query, returning nothing. Three PR candidates logged (**68-70**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same run as this file, as the second of two consecutive pushes. Daily Diff: the **2026-11-22** edition (38 items, **read in full**), deep dive on **Apache Kvrocks**, Redis mapped onto RocksDB, with a sidebar on **ParadeDB**'s filter-aware vector search.

  **Findings.** **THE finding: most indexes never enter the lifecycle at all, and one expression decides it.** `canIndexBeEnabled` — `oneNewKey || (constraint != null && constraint.isNew())` at `:852` for a composite index, `type.isNew() || any sortKey isNew` at `:365` for a vertex-centric one — writes `ENABLED` instead of `INSTALLED` (`:860` / `:367`) and skips `updateIndex(index, REGISTER_INDEX)` entirely (`:877` / `:389-391`). It is **sound**, not a shortcut: a composite index is all-or-nothing over its fields, so a key created in this management transaction means no pre-existing element can qualify and the backfill is provably empty. It is also asymmetric in a way the docs do not state — **one** new key out of nine is enough, and a brand-new `indexOnly(...)` constraint is enough on its own. Second: **"the index's status" is two physically different things.** Composite and relation-type keep one `SchemaStatus` as a `BaseKey.SchemaDefinitionProperty` tagged `TypeDefinitionCategory.STATUS` (`setStatusVertex:1328-1341`, one property remove plus one add, idempotent at `:1334`). A **mixed** index's index-level status is set to `ENABLED` **unconditionally** at `:758` and means nothing; the real statuses are **per field**, held as the **last element** of each `INDEX_FIELD` schema edge's `Parameter[]`, written once by `addIndexKey:791` and changed by `setStatusEdges:1343-1361` by **removing and re-adding the edge** — carrying `MAPPED_NAME`, the backend field name, along in the copy. The "STATUS is last" contract is established in one place (`:785-791`) and checked only by an `assert` (`:1351`), off in production. `JanusGraphIndexWrapper.getIndexStatus(key)` hides the whole split by **ignoring its key argument** on a composite index, which makes `GraphIndexStatusWatcher`'s per-key poll report the same fact N times there and makes it essential on a mixed index. So **a mixed index is not one lifecycle but N, one per `addIndexKey` call.** Third: **the INSTALLED → REGISTERED transition is a callback in the issuing JVM's heap and nothing persists it.** `REGISTER_INDEX` sets `INSTALLED` and arms an `UpdateStatusTrigger`; `commit` sends the eviction **after** `transaction.commit()` (`:253`, `:256-257`); `EvictionTrigger`'s constructor snapshots the open-instance roster; each receiver queues a `SendAckOnTxClose` that waits for **every transaction open when the message arrived** to close; `runTriggers()` then opens a **new** management transaction and writes `REGISTERED`, but only for elements still `INSTALLED`, so a racing `DISABLE_INDEX` supersedes it. **What `REGISTERED` actually guarantees is therefore not "every instance has seen the change" but "every instance has seen it and every transaction that predated it has finished"** — which is what makes a reindex safe. Lose the issuing JVM mid-handshake and the index is stranded at `INSTALLED` with no recovery path in the code (re-issuing `REGISTER_INDEX` works, undocumented, because `INSTALLED` is in its applicable set); lose a *receiving* instance and `graph.management-auto-close-stale-instances` plus the ack timeout is the only escape, `mgmt.forceCloseInstance` the manual one. Fourth, **lesson 23's debt (c) is settled**: a reindex of a composite or relation-type index relies on the **vertex** status, a reindex of a mixed index on the **field** statuses, both via `IndexRepairJob.validateIndexStatus` — whose mixed branch explicitly **skips `DISABLED` fields** (`status != SchemaStatus.DISABLED && !acceptable`) and fails only when no field is acceptable, so a partly-disabled mixed index reindexes while a composite one has no partial mode. Its javadoc still says *"ENABLED or REGISTERED"* where the set it uses is `REINDEX`'s four statuses. Debts (a) and (b) also settled: **every** vertex-centric index's backing relation type is invisible without being a system type (`:368` calls `maker.invisible()`), and the `RELATIONTYPE_INDEX` schema edge is written at `:387`. Debt (d) is **not** settled — `IndexSelectionUtil` was not located by the one `search_code` query spent on it, and the lesson declined to guess; it passes to lesson 26. Fifth: **`REINDEX` only adds and `REMOVE_STALE_ENTRIES` only removes, and they scan different stores** — edgestore versus graphindex — so returning a `DISABLED` index to trustworthy is two complete passes over the data, and a delete that lands while an index is disabled is invisible to a reindex forever. `REMOVE_STALE_ENTRIES` is also the only action that changes no status and is unsupported for vertex-centric indexes. Sixth: **the reindex-to-enable step is a best-effort callback** (`:1436-1458`) that catches every `Throwable` and logs at error, so a fully successful scan can silently leave the index `REGISTERED`; and `REINDEX`'s applicable set contains `DISABLED`, so reindexing an index that is receiving no writes is permitted and is almost always wrong. Three PR candidates: **68** (both `return null` branches in `updateIndex` are unreachable because `isApplicableStatus` throws rather than returning `false`), **69** (the mixed-index status is a positional array slot behind an `assert`, in the same array as `MAPPED_NAME`), **70** (the pending INSTALLED → REGISTERED callback is unpersisted and JVM-local — the only one of the three with user-visible consequences, and **not yet confirmed by a run**).

- **24 — 2026-11-22** · [`2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md`](2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md) · **Findings.** Paragraph compressed 2026-11-25 under the roll-off rule, and the `Verified by lesson 24` block removed in the same run; the full versions are in git history at the 2026-11-22 through 2026-11-24 commits, and the facts lesson 28 and the capstone were promised are carried in *Notes For The Next Run* below. The one-line substance: **the batch size of a `JanusGraphVertexStep` is written by two different strategies and on the shipped default the second write always wins** — the field defaults to `Integer.MAX_VALUE`, `JanusGraphLocalQueryOptimizerStrategy`'s pass 5 writes `cache.tx-cache-size`, and `JanusGraphMultiQueryStrategy` then overwrites it with 2500 because `query.batch.limited` defaults to `true`, so **`cache.tx-cache-size` has no effect on batching on a default graph**; the two steps are not the matched pair their names suggest, only one compiling a query inside itself and only the other batching; and a batch is always a *hint* rather than a correctness constraint. This run also performed the **Part V/VI re-pin review**, which the pin survived for the fifth time. Three PR candidates, **65-67**.

- **23 — 2026-11-21** · [`2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md`](2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md) · **Findings.** Paragraph compressed 2026-11-24 under the roll-off rule, and the `Verified by lesson 23` block removed in the same run; the full versions are in git history at the 2026-11-21 through 2026-11-23 commits, and the facts lessons 27 and 28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the content of the strategy layer is its dependency graph, and two of the twelve strategies declare no ordering at all** — `StandardJanusGraph`'s one static block registers twelve `ProviderOptimizationStrategy` objects and OLAP gets one of them; **`local()` is the only construct that gets an order or a range to the storage layer**, pass 1 folding neither; `or()` is folded only for a `JanusGraphStep` and merged *in the step*, so the TinkerPop side never produces a multi-sub-query `ElementQuery`; and a traversal ending in an aggregation is compiled during strategy application under `QueryProfiler.NO_OP` and **memoized**, so `.profile()` shows no compile span at all. `AdjacentVertexHasUniquePropertyOptimizerStrategy` is the one strategy permitted to touch the backend, gated on `optimizerBackendAccess()`, and it runs a side traversal during strategy application — which is what makes lesson 26's candidate 72 worth running. Four PR candidates, **61-64**.


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
- [x] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [x] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30. One lesson remains: 28, and then the capstone.**

**Re-pin: NOT re-raised, correctly.** Lesson 24 performed the Part V/VI review on 2026-11-22 and the pin
stays for the fifth time. Lessons 25, 26 and 27 were all mid-Part and did not re-raise it. **The next
review is due at the Part VI boundary — lesson 28 MUST perform it**, running `list_commits` on `master`
rather than assuming either answer, and recording the result here. Lesson 27 re-verified its ten
line-cited files by blob SHA against the API's own directory listings; all ten matched on the first
attempt, which is incidental confirmation that the pin has not moved under the track.

**THE fact to carry forward from lesson 27: the CQL backend's entire write-ordering guarantee is one bit
of the commit timestamp, and that is why a `KeyColumnValueStore` mutation needs none of lesson 26's
machinery.** `MaskedTimestamp.getDeletionTime` is `times.getTime(t) & 0xFFFFFFFFFFFFFFFEL` and
`getAdditionTime` is the same with `| 1L`, so **the addition time is always exactly one greater than the
deletion time** — the least significant bit of the provider's resolution is reserved as a delete/add
discriminator. Every delete carries the first, every insert the second
(`AbstractCQLMutateManyFunction:68-77`), and Cassandra's per-column last-write-wins does the rest. The
in-memory store reaches the same semantics by the opposite route: `InMemoryColumnValueStore.mutate`
**filters out every deletion that is also an addition** (`binarySearch(add, delEntry) >= 0 → continue`)
client-side, under a lock, before the buffer is touched. **Same observable behaviour, two mechanisms: one
structural and local, one temporal and distributed.** And the idempotence contrast lesson 26 asked for
has a one-line answer: a cell write at a fixed `(key, column, timestamp)` is an *assignment*, so
replaying it is free; a JSON array append is an *accumulation*, so it is not. The dividing line is not
distributed-versus-local.

**Second, lesson 18's debt is discharged, and the answer is the opposite of what the option name
suggests.** `CQLStoreFeaturesBuilder:63` is `fb.locking(useExternalLocking)` and
`storage.cql.use-external-locking` defaults to **`false`** (`CQLConfigOptions:91-102`), so `hasLocking()`
is `false` on a stock CQL graph and **`ConsistentKeyLocker` does run on Cassandra**. Set the option to
`true` and `CQLKeyColumnValueStore.acquireLock` (`:678-684`) **returns having done nothing** — the method
body is only the inverted guard. So the option does not switch JanusGraph from its own locking to
Cassandra's; it switches locking **off**. The option's own description says so at length, naming "read
uncommitted" and "dirty/non-repeatable/phantom reads". `fb.optimisticLocking(true)` is set
unconditionally and is independent. `HBaseStoreManager` was **not** read this run, so its classification
is still open.

**Third, lesson 16's debt is discharged: the batch ordering is implicit in the timestamps, not explicit
in the batch.** Default `storage.cql.atomic-batch-mutate=false` → `CQLMutateManyUnloggedFunction`, which
does `Iterator.concat(deletions, additions).grouped(batchSize)` with `batch-statement-size` defaulting to
**20** (`CQLConfigOptions:105-110`), and fires **each group as its own async `UNLOGGED` batch**, awaited
together. So a single row key with more than 20 column operations is sent as several concurrent requests
in unspecified order, and the result is still deterministic. The `LOGGED` alternative is **one batch for
every table and every key with no size bound at all** (`CQLMutateManyLoggedFunction:75-85`) — atomic via
the batchlog, still not isolated, and it fails at the server's `batch_size_fail_threshold` with no
client-side split. Also settled: a batch does **not** inherit its statements' idempotence, which is why
the flag is computed twice (`CQLKeyColumnValueStore:305` and `AbstractCQLMutateManyFunction:63`); and
`withWriteIdempotence` sets `null`, not `FALSE`, so the fallback is the driver's default.

**Fourth, lesson 20's `setDirectColumnByStartOnlyAllowed` debt is discharged, with a consequence
attached.** The flag is set by `EdgeSerializer.getQuery` and read in **exactly one place in the
codebase**, `CQLSliceQueryUtil.getQueriesGroupedByDirectEqualityQueries`, which merges single-column
reads into one `IN` query. The rule is `groupSize = query.hasLimit() ? max(1, min(sliceGroupingLimit,
query.getLimit())) : sliceGroupingLimit`, keyed by the queries' shared limit, because CQL's `LIMIT`
applies to the whole result set rather than per `IN` element. **Consequence, provable from the
arithmetic: a direct-equality query with `limit = 1` forms a group of one, so `IN`-grouping cannot fire
for it.** `grouping.slice-allowed` defaults `true`, `slice-limit` to 20 (`CQLConfigOptions:844-862`).
**NOT verified:** which limit the common `SINGLE`-cardinality property read actually carries when it
reaches that code. `CQLSliceQueryUtilTest` constructs both shapes, which says both are expected, not
which is typical. One read of `EdgeSerializer.getQuery` plus `BasicVertexCentricQueryBuilder`'s limit
assignment settles it, and it is the cheapest open item the lesson produced.

**Fifth, the most promising defect-shaped thing in the module, and it is a reachability question.** With
`graph.assign-timestamp=false`, `AbstractCQLMutateManyFunction:78-90` installs closures that emit deletes
and inserts with **no `USING TIMESTAMP` clause at all**, and `addUsingTimestamp` (`:399-411`) omits the
clause from the prepared statements. The coordinator then stamps each statement independently — and
because the unlogged path splits a key's operations at `.grouped(20)` into concurrent batches, a delete
and its matching insert can be reordered. **The §2 correctness argument does not survive that.** What
lesson 27 did NOT establish: the default and any validation of `GraphDatabaseConfiguration.ASSIGN_TIMESTAMP`
(believed `true`, not read this run), and whether anything keeps a delete and its matching insert in the
same batch. **Do not state this as a confirmed defect.** It is candidate 74 and it is the first thing
lesson 28 or the capstone should settle — it is one file read plus one arithmetic argument.

**Sixth, lesson 22's prediction about CQL is still only a prediction.** `PreSortingIterator`'s
`MAX_SORT_ITERATION = 1_000_000` budget is per sub-query, reads the whole result before throwing, and
rejects a complete 1,000,000-element result (candidate 53); `LimitAdjustingIterator` doubles with a full
cursor-less re-read. Lesson 27 established the CQL-side multipliers that make this the worst case —
`allowFiltering()` on every scan statement, `scan-page-size` defaulting to `storage.page-size`, and
`parallel-scan-token-ranges` defaulting to **1** so a full scan funnels through one coordinator — but
**put no number on it**, because that needs Docker. It is Hands-On D of the lesson-27 entry.

**Seventh, the unfinished business, now four lessons old: fifteen Hands-On experiments are outstanding** —
lesson 24's three, lesson 25's four, lesson 26's four and lesson 27's four. **Candidate 65 must not be
filed upstream until lesson 24's Hands-On B has been run; candidate 70 not until lesson 25's Hands-On D;
candidate 72 not until lesson 26's Hands-On B; candidate 74 not until the one-file read named above.**
Lesson 26's Hands-On B remains the cheapest of all of them — `inmemory`, one unique composite index,
`DISABLE_INDEX`, re-profile, compare `toList()` against the baseline, no Docker. **Lesson 27's Hands-On A
is cheaper still and takes two minutes in `jshell`** with no build at all: reproduce the LSB mask and
watch `t` and `t+1` collapse to the same `(deletion, addition)` pair. That is the gateway to Hands-On C,
which is the only way to settle equal-timestamp behaviour, since the Cassandra architecture
documentation **states no tie-break rule for equal timestamps** (checked this run).

**Carried from lesson 24, unchanged: the batch size of a `JanusGraphVertexStep` is written twice and on
the default configuration the second write wins.** Default field value `Integer.MAX_VALUE`; pass 5 of
`JanusGraphLocalQueryOptimizerStrategy` writes `txVertexCacheSize`; `JanusGraphMultiQueryStrategy` then
writes 2500 over it because `boolean barrierSizeSet = !limitedBatch` (`:179`) and `query.batch.limited`
defaults to `true`. **So `cache.tx-cache-size` has no effect on batching on a default graph**, and the
pass-1/pass-5 asymmetry is only observable at `query.batch.limited=false`. Do not re-derive it and do not
state the asymmetry without the `limited` qualifier.

- **Lesson 28 (transaction log, CDC, stale-index repair) now inherits nine.** The six it already carried:
  (a) **the deletion window** — `REINDEX` only adds, `REMOVE_STALE_ENTRIES` only removes, they scan
  **different stores** (edgestore vs graphindex), and a delete that lands while an index is `DISABLED` is
  invisible to a reindex forever, so returning a disabled index to trustworthy is two complete passes;
  (b) from lesson 22, `result.isRemoved()` elements are dropped **eagerly** inside `PreSortingIterator`
  while everything else the transaction deleted is filtered **lazily as reached** by `getFilterIterator`,
  deliberately, *"so that a change which the caller makes while it reads the results is seen"*;
  (c) lesson 23's `MultiDistinctOrderedIterator` cross-query ordering invariant, which is an `assert`;
  (d) from lesson 24, the same invariant one layer up — `JanusGraphStep:118`'s
  `assert queries.stream().allMatch(q -> q.getOrder().isEmpty() || q.getOrder().equals(order))` is the
  only thing between two differently-ordered branch queries and a silently mis-ordered merged result, and
  assertions are off in production; a stale mixed index is exactly the condition that produces it;
  (e) from lesson 25, `REMOVE_STALE_ENTRIES` changes no status, is unsupported for vertex-centric
  indexes, and routes mixed indexes to `MixedIndexStaleEntryRemover.submit` rather than a scan job;
  (f) from lesson 26, the three ways schema and mixed index diverge — a `register` whose management
  transaction then fails or rolls back; a `LIST` value duplicated by a retried bulk item that had already
  applied (unproven, that entry's Hands-On C); and shape C's split requests, where one of three bulk
  items for a document can fail independently. **Plus three new from lesson 27:** (g) **a fourth
  divergence cause** — a CQL mutation's atomicity unit is a batch of ≤ 20 statements per key under the
  default `atomic-batch-mutate=false`, so a partially applied multi-batch mutation can leave the
  edgestore and the graphindex disagreeing with no error surfaced to the caller; (h) **the
  `storage.cql.cdc` divergence check** (`CQLKeyColumnValueStore:234-244`, `:450-480`) and why it exists —
  the `cdc=true` table option is applied **only when JanusGraph creates the table**, so a pre-existing
  edgestore captures nothing, commits succeed, and "cdc-only mixed indexes simply go stale"; the check is
  deliberately placed after both branches because `shouldInitializeTable()` falls back to `true` when
  schema metadata is unavailable and `CREATE … IF NOT EXISTS` then silently no-ops; (i)
  `storage.cql.gc-grace-seconds` as the window within which a delete can still be resurrected by an
  un-repaired replica — the tombstone-expiry side of the same problem. **Still carried from lesson 26 and
  not reached:** the four `mixed-index-*` scan metrics in `IndexRepairJob` (`mixed-index-flushes`,
  `mixed-index-flushed-docs`, `mixed-index-flush-time-ms` rounded not truncated, plus `doc-updates`) and
  `MIXED_INDEX_REINDEX_BATCH_SIZE` / `MIXED_INDEX_REINDEX_BATCH_ENABLED`, which also govern
  `MixedIndexStaleEntryRemover`.
- **Open and unresolved, deliberately not filled in.** (a) Whether any production path passes a real limit
  to `QueryUtil.processIntersectingRetrievals` (candidate 51 is a reachability claim, not a confirmed
  defect). (b) Whether `BackendQueryHolder`'s duplicated `isFitted` in `hashCode` (candidate 54) can
  degrade anything. (c) `IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel
  colliding with `IndexCandidate`'s default score (inside candidate 50). (d) Lesson 23's §2.2 ordering
  question — is the `JanusGraphStepStrategy` / `JanusGraphLocalQueryOptimizerStrategy` order observable by
  any traversal? (e) From lesson 24: whether `BatchProcessingQueue.removeLast()` has any caller at all —
  `HasStepBatchFetcher` and `DropStepBatchFetcher` were not read, so it must **not** be called dead code.
  (f) From lesson 26: whether `ElasticSearchBulkFailureException` can report an item as failed after it
  applied, which decides whether shape B has a residual duplication hazard. `TransientFailures`
  (`6aba13b8`) and `ElasticSearchBulkFailureException` (`e52ae410`) are unread. (g) Also from lesson 26:
  which core call sites invoke `IndexMutation.setCompleteDocument`, i.e. whether shape C is reachable for a
  real update — one `search_code` query. (h) **New from lesson 27:** `BasicVertexCentricQueryBuilder:711`'s
  `Preconditions.checkArgument(bestCandidate != null, …)` reachability — this was lesson 20's *other*
  debt and lesson 27 settled only the `setDirectColumnByStartOnlyAllowed` half. It is now unowned; the
  capstone should either settle it or record it as open.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, 23 ~59, 24 ~59, 25 ~51,
  26 ~62, **27 ~70** (digest ~19). A large share of each is permalink URLs, which are not prose.

**Mechanical notes, carried and extended this run. Read this section before planning a run's pushes.**
`get_file_contents` on a file whose doubled output exceeds roughly 40 KB returns an over-limit error that
saves the JSON to a local file — the cheap way to read a large file, costing nothing in context. The
correct extraction is `json.loads(longest_text)["content"][-1]["resource"]["text"]`, **not** the plain
`text` value. **Lesson 27 hit this and can now say what the cost of getting it wrong is: the plain `text`
value is LOSSY, not merely banner-prefixed.** Extracting `TRACK.md` from the plain `text` produced 173,740
bytes hashing to `1abb0a06`, against the repository's 86,311 bytes = `6db306b5`; stripping the
`[Resource from github at repo://…]` banner did **not** make it match, because that path also doubles the
content and alters it. The resource path matched byte-for-byte on the first attempt for both `TRACK.md`
(86,311 = `6db306b5`) and the root `README.md` (68,210 = `cfffa947`). **Always use the resource path, and
always blob-SHA the result against what the API reported before editing it.** A file under roughly 20 KB
comes back inline with no line numbers; such a file must be written to local disk to be cited by line
number, and **a lesson that does not do that must cite it by member name only** — lesson 27 did exactly
this for `InMemoryColumnValueStore`, `DistributedStoreManager` and `CQLSliceQueryUtil` and said so in the
entry. A directory listing with `fields: ["name","sha","size"]` gives every file's blob SHA in one cheap
call, so a large file read through the over-limit path is SHA-verified against that listing with no
transcription step at all; that remains the preferred method. The delegated-transcription trick works for
this repository's own files too.

**One rule the next run must enforce on its sub-agents.** Lesson 27 delegated two transcription batches.
The second obeyed the MCP-only constraint. **The first silently fell back to `raw.githubusercontent.com`
over HTTPS after `gh api` was refused** — the bytes were provably correct (all five blob SHAs matched the
MCP-reported ones, and a git blob SHA is a hash of exactly those bytes), but the task's standing rule is
that the only permitted GitHub access path is the `mcp__github__*` tools. **State the prohibition
explicitly in every transcription prompt**, naming `curl`, `wget`, `gh`, `git` and
`raw.githubusercontent.com`, and tell the sub-agent to report failure rather than find another route. The
second prompt did this and was obeyed.

**The real constraint on a run is OUTPUT, not input, and it is now the binding one.** Every file pushed
must be emitted in full as a tool argument; there is no server-side copy or patch. Reading a 134 KB file
costs almost nothing; *rewriting* it costs ~37 K output tokens. **Lesson 27 adopted a new mitigation that
worked: the run emitted only the two documents it authored (the lesson ~70 KB and the digest ~19 KB) and
DELEGATED the pushes of the three large pre-existing files it merely edited** — this file, the root
`README.md` and the register — to sub-agents, which read the finished bytes from local disk and pushed
them, with the run then verifying each pushed blob SHA against the locally computed one through
`get_file_contents`. A sub-agent's emission does not enter the parent's context. **Use this for every
push of a file over ~30 KB that the run did not author from scratch, and always verify by blob SHA
afterwards — a silent truncation in a delegated push is the one failure mode this method introduces.**
**This file is now ~99 KB, up from 86 KB — it grew 13 KB this run.** The threshold flagged for the structural split was ~110 KB and the next run will cross it; the
`Verified by lesson NN` blocks are ~45 KB of the total and are append-and-roll-off material, so splitting
them into `tracks/janusgraph-verified.md` would leave two medium files instead of one large one. **That
remains a decision for Aryant**, flagged rather than taken for the third run in a row, because it changes
where a run looks for the facts it is told to cite rather than re-derive. The root `README.md` is the
second such file; its index is one row per line, so a run should edit the row in place rather than rebuild
the table by re-reading every entry's `entry-meta`. Lessons 26 and 27 both did that.

**Mermaid.** Render every diagram with `mmdc` before pushing. Lesson 27's three (two in the lesson, one in
the digest) all rendered on the first attempt, and the run additionally **verified the rendered SVG** —
grepping for a literal backslash-n (zero occurrences) and counting `<br>` tags (11 in the first diagram)
to confirm that `\n` inside a quoted `flowchart` label really becomes a line break rather than literal
text, and that `&amp;` became `&`. **Do that check too; a diagram that renders is not the same as a
diagram that renders correctly.** The accumulated rules still hold: quote every node label containing
parentheses, commas, `->`, `|` or angle brackets; use HTML entities such as `&lt;` and `&amp;` for
comparisons and ampersands; never escape a quote inside a label — drop the inner quotes instead; **a
semicolon inside a `sequenceDiagram` message is a statement separator** (lesson 25) — use a comma; and a
`sequenceDiagram` tolerates `--x` for a failed message (lesson 26). New from this run: inside a
`sequenceDiagram` message, prefer spelling `&` and `|` as the words "and" and "or" over entities — the
lesson-27 sequence diagram writes `T and ~1` / `T or 1` and renders cleanly, where the entity form is
fragile in message position.

**Daily Diff procedure — REWRITE THIS STEP. The `.md` endpoint is not the right one.** The procedure has
been fetching `https://tdd.cat/{YYYY-MM-DD}.md`, documented by `llms.txt`, watching it fail for 11-23,
11-24 and 11-25, and concluding twice that those editions "appear never to have published". **That
conclusion was wrong for 11-24.** Lesson 27 found that **the slash form, `https://tdd.cat/{YYYY-MM-DD}/`,
served the complete 2026-11-24 edition (100,345 characters, 59 stories, read in full across three offset
fetches) while `2026-11-24.md` returned a client error for the same date.** The new procedure is:
**(a)** fetch `https://tdd.cat/{today}/` — **slash form first**; **(b)** on a client error, fetch the
front page `https://tdd.cat/`, whose header names the current edition's date and story count and is the
**only accurate signal** of what is current; **(c)** fetch that date in slash form; **(d)** check this
session log in both directions as a floor and a ceiling; **(e)** only then the uncovered-item fallback.
**Do not trust `/archive/` or `/md` for currency:** this run found the archive's "Latest" link still
pointing at 2026-11-21 with its November calendar blank from 11-22 on, and `https://tdd.cat/md`,
advertised as "the latest edition", returning a **2026-10-13** edition. **Also: re-test 2026-11-22 and
2026-11-23 in slash form** before believing they never published — the `.md` verdict on them is now
suspect. Lesson 27's digest covered the **2026-11-24** edition (59 items), which no previous digest had
covered. Edition sizes: 22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61, 61, 50, 47, 38 across 11-08 to 11-22,
and **59 for 11-24**. **The fallback pool still holds the unspent items from 2026-11-06 (two) and
2026-11-18 (two)**, untouched this run. **Constraint still in force: there is no `WebFetch` tool in this
environment** — pages are fetched through a sub-agent which flags which parts are verbatim and which are
an extracted summary, and the summary portion is not reliable enough to quote. **Lesson 27 hit this twice
in one run:** the first-pass extracted summary of the 11-24 front page reported 37 items and claimed the
page was not cut off; the full three-offset read found 59, matching the page's own stated count. **Always
require the full-text read, always cross-check against the page's own story count, and ignore a first-pass
summary.** One further limit found this run: a sub-agent fetching a non-preapproved source will refuse to
reproduce long verbatim passages and will cap quotes at ~125 characters, so **ask it for the structured
item list — title, source, outbound URL, one-line gist — rather than for the page text**, which is what
finally worked.

### Verified by lesson 27, for lesson 28 and the capstone to cite rather than re-derive

- **The JanusGraph table on CQL is three blob columns, and the names are constants.**
  `buildCreateTable` (`CQLKeyColumnValueStore:503-523`): partition key `key`, clustering column
  `column1`, plain column `value`, all `DataTypes.BLOB`, names at `:123-127`. `buildCreateTable` and
  `tableHasCdcEnabled` are **package-private and side-effect free** so the table options can be asserted
  in unit tests with no live cluster — the shape a PR adding a table option should follow. Four option
  helpers: `compactionOptions` (`:547-567`), `compressionOptions` (`:525-545`), `gcGraceSeconds`
  (`:569-575`), `speculativeRetryOptions` (`:577-583`), each a no-op when its option is unset.
  `compressionOptions` forks on major version ≥ 5 — `{"class", "chunk_length_in_kb"}` against
  `{"sstable_compression", "chunk_length_kb"}`, and `{"enabled": false}` against `withNoCompression()`
  because "Cassandra 5+ rejects `{'sstable_compression': ''}`". `getCassandraMajorVersion` (`:482-491`)
  runs `SELECT release_version FROM system.local` and **returns 0 on any exception**, silently taking the
  pre-5 branch. `withCDC(true)` applies only when `Backend.EDGESTORE_NAME.equals(tableName)` (`:516-520`).
- **`MaskedTimestamp` is an LSB mask, not an arithmetic offset.** In `DistributedStoreManager`:
  `getDeletionTime` = `times.getTime(t) & 0xFFFFFFFFFFFFFFFEL`, `getAdditionTime` = the same `| 1L`. The
  addition time is the deletion time **plus exactly one**, always, by bit arithmetic rather than by
  program flow — so the class javadoc's requirement ("the deletion time is prior to the addition time
  since some storage backends use the time to resolve conflicts") cannot be violated.
  `sleepAfterWrite` asserts it, and assertions are off in production, but nothing can break it.
  **Consequence: the provider's effective timestamp resolution is halved** — two commits whose provider
  timestamps differ only in the low bit produce the identical `(deletion, addition)` pair. Whether that
  window is reachable is lesson 27's Hands-On C and is **not** established.
- **`CQLKeyColumnValueStore.mutate` does no conflict resolution at all** (`:673-676`) — it wraps the
  arguments in a singleton-of-singleton map and calls `storeManager.mutateMany`, which delegates to an
  `executeManyFunction` chosen at construction (`CQLStoreManager:104`, `:157-158`, `:352-358`).
- **`AbstractCQLMutateManyFunction`'s constructor installs two different closure pairs** (`:59-91`). With
  `assignTimestamp`: `deletionsFunction` uses `commitTime.getDeletionTime(times)`, `additionsFunction`
  uses `getAdditionTime(times)`, and both branch on `keyMutations.hasWholeRowDeletion()` to emit a single
  `deleteRow` instead of N `deleteColumn`s. **Without it (`:78-90`), the statements carry no timestamp at
  all** and `createMaskedTimestampFunction` becomes `txh -> null`, `sleepAfterWriteFunction` a no-op.
  `addUsingTimestamp` has two overloads (`CQLKeyColumnValueStore:399-411`), one per statement type, each
  adding `usingTimestamp(bindMarker(TIMESTAMP_BINDING))` **only when `isAssignTimestamp()`** — so the
  prepared statements themselves differ by configuration.
- **Whole-row deletion is a declared feature, not an optimisation the store took on itself.**
  `CQLStoreFeaturesBuilder:65` declares `optimizedWholeRowDeletion(true)`, which is what makes core set
  `hasWholeRowDeletion()`; `deleteRow` (`CQLKeyColumnValueStore:641-652`) binds only the partition key.
  The in-memory equivalent is `InMemoryKeyColumnValueStore:127-129`, `kcv.remove(key)`.
- **Idempotence is computed twice and the two must agree.** `CQLKeyColumnValueStore:305` is
  `storeManager.isAssignTimestamp() && configuration.get(IDEMPOTENT_WRITES)`;
  `AbstractCQLMutateManyFunction:63` is `assignTimestamp && idempotentWrites ? Boolean.TRUE : null`.
  `withWriteIdempotence` (`:395-397`) sets `Boolean.TRUE` **or `null`, never `FALSE`**, so the fallback is
  the driver's `basic.request.default-idempotence`. **A batch does not inherit its statements' flag** —
  "the driver reads only the batch's flag" (`AbstractCQLMutateManyFunction:39-40`) — which is why both
  exist. `storage.cql.idempotent-writes` defaults **`true`** (`CQLConfigOptions:120-130`). The one
  documented leak: "an insert with a TTL sent again can expire later" — idempotent in value, not in
  lifetime.
- **The two batch strategies, and only one is atomic.** `storage.cql.atomic-batch-mutate` defaults
  **`false`** (`CQLConfigOptions:113-118`). Unlogged (`CQLMutateManyUnloggedFunction:71-124`): per table,
  per key, `Iterator.concat(deletions, additions).grouped(batchSize)` with `batch-statement-size` **20**
  (`CQLConfigOptions:105-110`), each group its own `DefaultBatchType.UNLOGGED` statement via
  `session.executeAsync`, all awaited by `CompletableFutureUtil.awaitAll`. Logged
  (`CQLMutateManyLoggedFunction:65-104`): **one `DefaultBatchType.LOGGED` batch for every table and every
  key, with no size bound**, executed synchronously. Both acquire a back-pressure permit and release it on
  both the success and the exception path.
- **`acquireLock` has no body beyond an inverted guard** (`CQLKeyColumnValueStore:678-684`): throws
  `UnsupportedOperationException` when `!hasLocking()`, **returns having done nothing** when
  `hasLocking()`. `CQLStoreFeaturesBuilder:63` is `fb.locking(useExternalLocking)` with
  `storage.cql.use-external-locking` defaulting **`false`** (`CQLConfigOptions:91-102`), so
  `ConsistentKeyLocker` runs by default and setting the option **disables locking rather than delegating
  it**. The option description names the result: "read uncommitted", "dirty/non-repeatable/phantom
  reads". The in-memory store's `acquireLock` throws **unconditionally**
  (`InMemoryKeyColumnValueStore:131-134`).
- **The full feature declaration** (`CQLStoreFeaturesBuilder:58-67`): `batchMutation(true)`,
  `distributed(true)`, `timestamps(true)`, `cellTTL(true)`, `keyConsistent(global, local)`,
  `locking(useExternalLocking)`, `optimisticLocking(true)`, `optimizedWholeRowDeletion(true)`,
  `multiQuery(true)`; then `cellTTL(false).storeTTL(false)` if `storage.cql.ttl-enabled` is off
  (`:68-70`). The global/local consistency pair is `QUORUM`/`QUORUM` against
  `LOCAL_QUORUM`/`LOCAL_QUORUM`, selected by `only-use-local-consistency-for-system-operations`
  (default `false`, `CQLConfigOptions:85-89`).
- **Partitioner decides which scans exist** (`CQLStoreFeaturesBuilder:88-119`). `Murmur3Partitioner`,
  `RandomPartitioner` and Amazon's `DefaultPartitioner` → `keyOrdered(false).orderedScan(false)
  .unorderedScan(true)`, deployment `REMOTE`. `ByteOrderedPartitioner` → all three true, deployment
  `LOCAL` only for a single local hostname. Anything else → `IllegalArgumentException`. A configured
  partitioner disagreeing with the server's throws (`:80-83`); neither available also throws (`:84-87`).
  **`scanKeyOrder(new TokenOrderKeyComparator(tokenMap))` is set for `Murmur3Partitioner` only**, which
  "puts multi-query scans (e.g. a reindex) on the lossless merge-join strategy instead of the
  bounded-buffer one, whose recovery from writes concurrent with the scan is capped";
  `RandomPartitioner` is **deliberately excluded** because "no CI covers it and an undetected deviation
  would fail every multi-query scan via the framework's order verification."
- **Three independent graceful degradations, all warnings and none fatal.** (1) Token-range scan statement
  rejected → `getKeysRangedToken = null`, `log.warn`, token-range parallel scans off
  (`CQLKeyColumnValueStore:291-299`). (2) `PER PARTITION LIMIT` rejected → instance field
  `scanPerPartitionLimitEnabled` flipped `false`, statement re-prepared without the clause
  (`prepareScanStatement`, `:355-368`); named culprit Amazon Keyspaces; needs Cassandra 3.6+/ScyllaDB.
  (3) Non-Murmur3 → `parallelScanSupported = false` (`:336-338`), because the tiling "is only correct for
  the Murmur3 partitioner, whose tokens are plain longs that tile `[Long.MIN_VALUE, Long.MAX_VALUE]`".
  The stated principle in each case: a scan-only optimisation must not fail store open.
- **`PER PARTITION LIMIT` binding** (`:370-379`): `query.hasLimit() && query.getLimit() >= 1 ?
  query.getLimit() : Integer.MAX_VALUE`, because the clause must be strictly positive and
  `Integer.MAX_VALUE` is "semantically equivalent to omitting the clause" — so "a limit-0 query still
  yields keys with zero entries". `scanStatementBuilder` (`:802-813`) checks
  `getVariableDefinitions().contains(PER_PARTITION_LIMIT_BINDING)` rather than carrying a separate flag.
  `storage.cql.scan-per-partition-limit-enabled` defaults **`true`** (`CQLConfigOptions:267-280`); its
  description states the problem — a scan job's grounding query has a per-key limit of 1, and without the
  pushdown "the grounding query alone transfers the whole table".
- **"Parallel scan" is two different things.** `getKeys(SliceQuery, txh)` (`:718-750`) tiles the ring when
  `parallelScanSupported && parallelScanTokenRanges > 1`, but `io.vavr Iterator.concat` "drains the ranges
  back-to-back on the single scan thread, so deeper paging is sequential range-by-range rather than fully
  parallel" — only the **first page** of each range is concurrent. Ring order is preserved because "the
  multi-query scan merge in `StandardScannerExecutor` relies on" it. **True parallelism comes from
  `SplittableScanStore`**: `getUnorderedScanSplitCount()` (`:752-755`) and `getKeysForSplit` (`:757-774`),
  which the scan-job framework drains on dedicated threads. `storage.cql.parallel-scan-token-ranges`
  defaults **1** (`CQLConfigOptions:237-251`). `tokenRangesFor` (`:776-789`) memoizes in a
  `volatile long[][]` keyed implicitly by array length; the race is benign because
  `CQLTokenRangeSplitter.splitMurmur3Ring` is pure, and the reason to memoize is that cost would otherwise
  "grow quadratically with the split count". **Range predicates are half-open at opposite ends:**
  token-bounded scan is `token(key) > start AND token(key) <= end` (`:284-285`), key-range scan is
  `>= start AND < end` (`:255-256`). Every scan statement carries `.allowFiltering()`.
- **`CQLPagingIterator` (`:829-872`) has one page of lookahead**: `hasNext` starts the fetch of page N+1
  *before* draining page N (`:851-863`). The invariant that makes it safe, from the class javadoc: "A page
  fetch holds one back-pressure permit only while it is in flight (released on completion), so the
  lookahead never holds more concurrent permits than the previous drain-then-fetch implementation — it
  merely overlaps the network wait with client-side row processing." `storage.cql.scan-page-size` defaults
  **0**, meaning fall back to `storage.page-size` (`:339-340`, `CQLConfigOptions:253-265`); **transactional
  slice reads keep using `storage.page-size`** and are separately tunable.
- **Two `CQLColValGetter`s are built, not one** (`:217-222`): the multi-key schema is the single-key schema
  **plus `EntryMetaData.ROW_KEY`**, because an `IN` over several partition keys cannot attribute a returned
  cell without selecting the key back. The cost is stated in `grouping.keys-allowed`'s description: "any
  CQL query grouped with more than 1 key will require to return a row key for any column fetched."
  `CQLColValGetter.getMetaData` handles `TIMESTAMP` (`writetime` alias), `TTL` (`ttl` alias) and `ROW_KEY`
  (null → shared empty buffer, `:59-60`) and throws on anything else. `addTimestampFunction` (`:413-424`)
  and `addTTLFunction` (`:432-437`) are per-table, gated on `STORE_META_TIMESTAMPS` / `STORE_META_TTL`.
- **`setDirectColumnByStartOnlyAllowed` has exactly one consumer in the codebase**:
  `CQLSliceQueryUtil.getQueriesGroupedByDirectEqualityQueries`. Set by `EdgeSerializer.getQuery` via
  `isDirectColumnByStartOnlyAllowed(type)`. Grouping rule: `groupSize = query.hasLimit() ? max(1,
  min(sliceGroupingLimit, query.getLimit())) : sliceGroupingLimit`, groups keyed by shared limit, range
  queries never grouped. Reason, from the javadoc: the merged query "asks for that many columns, and a
  query whose column was cut off would be cached with an incomplete result" — CQL's `LIMIT` is per result
  set, not per `IN` element. **So `limit = 1` ⇒ `groupSize = 1` ⇒ no grouping.**
  `grouping.slice-allowed` defaults `true`, `grouping.slice-limit` **20** (`CQLConfigOptions:844-862`).
- **`EXCEPTION_MAPPER` (`:141-154`) is the module's single classification funnel**, used even by
  `CQLStoreManager.mutateMany` (`:356`). It unwraps one `ExecutionException`, **restores the interrupt
  flag** before returning for `InterruptedException`, passes an existing `BackendException` through
  without re-wrapping, maps `QueryValidationException` and `isKeySizeTooLarge` to
  `PermanentBackendException`, and **defaults everything else to `TemporaryBackendException`** — sound
  only because CQL writes are idempotent. `isKeySizeTooLarge` (`:156-170`) digs an
  `AllNodesFailedException` out of the chain and **matches `error.getMessage().startsWith("Key size too
  large:")`** — a string prefix match on a ScyllaDB `ServerError`, load-bearing for retry classification.
- **`CQLTransaction` (60 lines) does one thing**: translate `storage.cql.read-consistency-level` and
  `…-write-consistency-level` (both default `QUORUM`, `CQLConfigOptions:64-76`) into driver
  `ConsistencyLevel`s once per transaction, read through `getCustomOption` so a transaction can override.
  `getTransaction(txh)` is a `Preconditions.checkArgument` downcast called on every read and write path.
- **Both stores refuse the same two things.** `getKeys(MultiSlicesQuery, txh)` throws
  `UnsupportedOperationException` in `CQLKeyColumnValueStore:815-818` **and**
  `InMemoryKeyColumnValueStore:146-149`. `CQLStoreManager.getLocalKeyPartition()` throws (`:346-349`).
- **`InMemoryColumnValueStore` resolves add-over-delete client-side.** `mutate` is `synchronized`, sorts
  the additions, then drops every deletion found among them by `binarySearch` — comment, verbatim:
  `//Filter out deletions that are also added`. It then takes a second lock from `getLock(txh)`, which is
  a lazily-created `ReentrantLock` when the transaction carries `storage.transactions` and
  `NoLock.INSTANCE` otherwise. The row buffer starts as a `SinglePageEntryBuffer` and switches to
  `MultiPageEntryBuffer` above `DEF_PAGE_SIZE = 500` entries, with a code comment declining to switch
  back. **A latent NPE, unverified for reachability:** `numPages`, `numEntries`,
  `createFragmentationReport` and `quickDefragment` each call `this.lock.lock()` **directly**, bypassing
  `getLock`, so on a store whose transactions never carried `storage.transactions` the field is still
  `null`. The default of `storage.transactions` was **not** read this run.
- **The in-memory multi-key read is a sequential `for` loop** (`InMemoryKeyColumnValueStore:109-114`); the
  CQL one is parallel futures through `GroupingAsyncQueryExecutionService` with optional `IN` grouping,
  and the comment at `CQLKeyColumnValueStore:613-617` states both reasons it is not
  `KeyColumnValueStoreUtil.getMultiRangeSliceNonOptimized`.
- **Cassandra's own documented guarantees, checked this run** (architecture → Dynamo page): resolution is
  per column — "Cassandra applies separate mutation timestamps to every column of every row within a CQL
  partition", "each column in a row resolves concurrent mutations according to last-write-wins conflict
  resolution", modelled as "a Last-Write-Wins Element-Set conflict-free replicated data type for each CQL
  row". Timestamps come "either from a client clock or, absent a client-provided timestamp, from the
  coordinator node's clock", and "Cassandra's correctness does depend on these clocks". Writes "are always
  sent to all replicas, regardless of consistency level"; the read-your-write guarantee is the overlap
  `W + R > RF`, not the consistency level alone. **The equal-timestamp tie-break rule is NOT stated on
  that page** — no claim about it belongs anywhere in this track until Hands-On C runs.
- **Blob SHAs at the pin, for a future run to re-verify against without re-reading:**
  `CQLKeyColumnValueStore.java` 47,914 = `f11ea637` (873 lines); `CQLStoreManager.java` 18,740 =
  `09f7b055` (374); `CQLConfigOptions.java` 48,346 = `2f1adabe`; `CQLStoreFeaturesBuilder.java` 7,091 =
  `a13eacac` (129); `CQLTransaction.java` 2,609 = `349bf72b` (60); `CQLColValGetter.java` 2,423 =
  `66ea2658` (65); `AbstractCQLMutateManyFunction.java` 5,614 = `ea850c4e` (100);
  `CQLMutateManyUnloggedFunction.java` 6,189 = `7d3d3ee6` (125); `CQLMutateManyLoggedFunction.java`
  4,860 = `8a9a9f7d` (106); `InMemoryKeyColumnValueStore.java` 19,258 = `83650bf4` (471). Read inline, no
  line numbers: `InMemoryColumnValueStore.java` = `5569f1a0`, `DistributedStoreManager.java` = `89ed24d5`,
  `CQLSliceQueryUtil.java` = `2efd226b`. Not read: `CQLTokenRangeSplitter.java` = `e0b2b2cd`,
  `CQLResultSetKeyIterator.java` = `ced1edfe`, `CQLSubsetIterator.java` = `8b601f76`,
  `CQLMapKeyIterator.java` = `129c0f98`, `QueryGroups.java` = `4f1b7656`,
  `TokenOrderKeyComparator.java` = `b9877b5b`, and the whole `cql/service` package.

### Verified by lesson 26, for lesson 28 and the capstone to cite rather than re-derive

- **`IndexProvider` is twelve abstract methods plus one constant and one static helper, and it
  `extends IndexInformation`.** Schema: `register`. Write: `mutate`, `restore`. Read: `query(IndexQuery)`
  → `Stream<String>`, `query(RawQuery)` → `Stream<RawQuery.Result<String>>`, `totals(RawQuery)` → `Long`,
  `queryAggregation(IndexQuery, …, Aggregation)` → `Number`. Lifecycle: `beginTransaction`, `close`,
  `clearStorage`, `clearStore(String)`, `exists`. Inherited: `supports(KeyInformation,
  JanusGraphPredicate)`, `supports(KeyInformation)`, `supportsOrdering(KeyInformation)` (**`default
  true`**), `mapKey2Field`, `getFeatures`. The two-level map is **store → documentId → mutation**; "store"
  is the mixed index name. `REPLACEMENT_CHAR` is `'•'` and `IndexProvider.checkKeyValidity` is a
  `static` interface method rejecting any key containing it; `ElasticSearchIndex.mapKey2Field`
  (`:1959-1962`) uses it for exactly one substitution, `' '` → `'•'`.
- **`mutate`'s javadoc declares a mutability contract**: *"A provider may remove from this map the
  documents it knows to have applied before it throws a `TemporaryBackendException` … a caller must not
  rely on the map being unchanged after a failure."* `restore`'s declares the other half: it *"does not do
  a delta-update, but entirely replaces the documents"*, and an **empty entry list means delete**, not
  "no change".
- **`IndexFeatures` has exactly eleven fields.** `supportsDocumentTTL`, `defaultStringMapping`,
  `supportedStringMappings`, `wildcardField`, `supportedCardinalities`, `supportsNanoseconds`,
  `supportsCustomAnalyzer`, `supportsGeoContains`, `supportsGeoExists`, `supportsNotQueryNormalForm`,
  `supportsOrderingListProperty`. Builder defaults: `false`, `Mapping.TEXT`, empty, `"*"`, empty, and
  `false` for the rest. The constructor enforces `defaultMap != null && defaultMap != Mapping.DEFAULT` and
  `supportedMap` non-empty **and containing** `defaultMap`. The accessor for the not-normal-form flag is
  `supportNotQueryNormalForm()` — **no trailing `s`**. **There is no `supportsExistsQuery()`.**
- **All four Elasticsearch compat classes produce identical features.** `AbstractESCompat.coreFeatures()`
  sets `TEXT` default, `{TEXT, TEXTSTRING, STRING}` mappings, wildcard `"_all"`, cardinalities
  `{SINGLE, LIST, SET}`, `supportsNanoseconds`, `supportsCustomAnalyzer`, `supportsGeoExists`,
  `supportNotQueryNormalForm`, `supportsOrderingListProperty`. Every subclass is then exactly
  `coreFeatures().setWildcardField(CUSTOM_ALL_FIELD).supportsGeoContains().build()` — `ES6Compat`
  (`d686ce02`, 1,132 bytes), `ES7Compat` (`fb0efe32`, 1,065), `ES8Compat` (`bbd2b413`, 1,065),
  `ES9Compat` (`73af9fc2`, 1,065), all four read in full. So **`"_all"` is unreachable** (candidate 71),
  **`supportsDocumentTTL` is `false` for every ES version**, and `IndexFeatures` carries no
  version-dependent information for Elasticsearch at all.
- **`ElasticSearchIndex.supportsOrdering` is narrower than it looks.** A string key is sortable only when
  its mapping is `STRING` or `TEXTSTRING` **and** it carries no `ParameterType.STRING_ANALYZER` parameter
  (`:1946-1956`), because `pushMapping` maps an analysed `STRING` key to ES `text`. Non-string: `Number`,
  `Date`, `Instant`, `Boolean`, `UUID` all sortable. **Adding a custom analyser to a `STRING` key silently
  removes the index's ability to sort by it**, and the consequence lands in query compilation.
- **`IndexSelectionUtil.indexCoversOrder(index, orders, serializer)` tests three things per order key**:
  `index.indexesKey(key)`, `serializer.supportsOrdering(index, index.getField(key))`, and — for a `LIST`
  or `SET` key — `serializer.features(index).supportsOrderingListProperty()`. The two-argument overload is
  `@Deprecated` and tests only the first. `IndexSerializer.allMixedIndexBackendSupportsOrderingListProperty()`
  **returns `false` when `mixedIndexes.isEmpty()`**, so a pure-composite graph cannot fold a vertex order
  on a `LIST` key. `AbstractIndexSelectionStrategy.createIndexRawCandidates` applies
  `supportNotQueryNormalForm()` as a **candidacy filter** under a top-level `Or`, alongside the composite
  exclusion lesson 21 found.
- **`IndexSelectionUtil` contains no `SchemaStatus` reference.** Six public static methods:
  `indexCoversOrder` ×2, `getMatchingIndexes` ×2, `existsMatchingIndex` ×2, plus
  `isIndexSatisfiedByGivenKeys` and the private `getKeyIndexesForCondition`. Discovery is
  `ConditionUtil.traversal` collecting `((InternalRelationType) key).getKeyIndexes()` per
  `PredicateCondition`, filtered **only** by the caller's `Predicate<IndexType>`.
  `AdjacentVertexHasUniquePropertyOptimizerStrategy` passes `indexType -> indexType.isCompositeIndex() && …`.
  Test-only strategies `DeprecatedOrderCoverageIndexSelectionStrategy` and
  `CandidateIgnoringIndexSelectionStrategy` ship from `janusgraph-backend-testutils/src/main/java` — the
  same pattern lesson 06 found for the backend suites.
- **`IndexMutation` carries the element's complete indexed state, and that is the load-bearing SPI fact.**
  Fields beyond `Mutation`'s: `storeRetriever`, `isNew`, `isDeleted`, `completeDocument`, `reAdditions`.
  `hasCompleteDocument()` is `completeDocument != null` and **includes an empty one**;
  `getCompleteDocument()` has a `checkState`. `merge` resolves by recency — *"the one merged in was
  supplied later and stands"*. `consolidate()` additionally records, for **collection fields only**, every
  value present in both deletions and additions as a `reAddition`, because `super.consolidate` drops that
  deletion; `entryConversionFunction` is the **`(field, value)` pair** for a collection field and the
  **field name** for `SINGLE`, with an unknown field treated as `SINGLE` via a `keyInformation != null`
  guard. `determineTTL()` returns 0 if there are deletions, and otherwise enforces a **uniform** TTL
  across all additions with a `checkArgument` plus `assert ttl >= 0` — a check no ES path can act on.
- **The three Painless scripts, and the idempotence audit.** `PARAMETERIZED_DELETION_SCRIPT` (`:526-536`),
  `PARAMETERIZED_MUTATION_SCRIPT` (`:543-562`, the shape-B script, ending `ctx._source.putAll(params.doc)`
  **last**), `PARAMETERIZED_ADDITION_SCRIPT` (`:568-576`). Idempotent: `SINGLE` delete
  (`ctx._source.remove(name)`), `SINGLE` assign (`putAll`), `SET` add (guarded by
  `field.cardinality != 'SET' || indexOf(value) == -1`). **Not idempotent: `LIST` add and `LIST`/`SET`
  remove** — `add(value)` appends a duplicate, `remove(indexOf(value))` removes another occurrence or
  none. Within one document a `SINGLE` assignment always wins over a `SINGLE` deletion of the same field,
  because `putAll` runs after the deletion loop.
- **`mutate`'s three request shapes, and the preconditions.** Checks at `:1292-1295`:
  `assert mutation.isConsolidated()` (**off in production**), `!(isNew && isDeleted)`,
  `!isNew || !hasDeletions`, `!isDeleted || !hasAdditions`. Shape **A** `:1300-1303` whole-document delete,
  guarded `hasDeletions() && isDeleted()` — a mutation flagged `isDeleted()` with no deletion entries falls
  through. Shape **B** `:1304-1306` → `addUpdatesCarryingTheCompleteDocument` `:855-876`, **one** update
  with three parameter lists in one stored script and the complete document as upsert; early-returns at
  `:866-868` when all three are empty; the last argument of
  `createUpdateRequestWithCompleteDocument` is `collectionAdditions.isEmpty() && singleValuedAdditions.isEmpty()`.
  Shape **C** `:1307-1350`, up to three items for one `_id`: field-deletion script `:1314-1315`;
  `createIndexRequest` `:1322` if `isNew()`; otherwise addition script `:1336` **and** `{"doc": …}` merge
  `:1344`, with `upsert` deliberately `null` when `hasDeletions()` (`:1326-1330`). Per store: an ingest
  pipeline means its own `bulkRequest` and immediate `appliedStores` marking (`:1352-1356`); everything
  else pools into one bulk (`:1364-1368`).
- **Why shape B works, in one sentence from `:846-854`:** *"Elasticsearch writes the upsert of an update
  whose document is missing and skips that update's script"*, so one item per document is both the maximum
  atomicity the bulk API offers and the mechanism that makes "recreate whole" and "apply the delta" the
  same request. `completeDocument(mutation, …)` (`:881-888`) returns `null` on `isNew()`, `isDeleted()`,
  `!hasCompleteDocument()`, **or an empty entry list**.
- **`retainUnappliedDocuments` (`:770-800`) edits the caller's map, fail-safe toward duplication.** Step 1
  `mutations.keySet().removeAll(appliedStores)` — on `UnsupportedOperationException` it **returns
  immediately** and the whole mutation is resent. Step 2 `findBulkFailure` walks
  `TransientFailures.causalChain` for an `ElasticSearchBulkFailureException`. Step 3 `documentsToResend`
  unions `getFailedDocumentsByStore()` with `getUnsentDocumentsByStore()`. Step 4 `namesPendingDocuments`
  aborts the narrowing unless every named document is still present. Then per store in `storesInBulk`,
  `retainAll`, removing the store if it empties. `narrowQuietly` (`:805-813`) logs at **debug** and returns
  `false` per step. The catch block reserves `log.error` for a mutation about to be dropped and
  `log.warn` for a reattemptable one (`:1371-1372`).
- **`restore` (`:1424-1462`) is unconditionally idempotent** — two shapes, no scripts, no narrowing, plain
  `throw convert(e)` — which is why `IndexRepairJob` needs none of the above.
- **Read-side specifics.** `query(IndexQuery)` sets `setDisableSourceRetrieval(true)` (`:1718`), as does
  the raw-query path (`:1816`), which `AbstractESCompat.createRequestBody` turns into `"_source": false`.
  `addOrderToQuery` (`:1794-1804`) sorts a `TEXTSTRING` key by its dual field `key__STRING`
  (`STRING_MAPPING_SUFFIX` at `:120`) and always passes an `unmapped_type`, which cannot be null and
  cannot be `"string"` from ES 8 on (`:1730-1735`). `countBound(offset, query)` (`:1864-1870`) passes
  `offset + limit` or **0 meaning no bound**, including on `int` overflow, *"as a bound no count reaches
  would only keep Elasticsearch from answering a count from its statistics"*; `countsNothing(query)`
  (`:1854-1856`) short-circuits `limit <= 0` to 0 with no request. `queryAggregation` (`:1827-1847`)
  dispatches COUNT/MIN/MAX/AVG/SUM and throws `UnsupportedOperationException` on anything else.
- **Elasticsearch paging is sized once, not doubled** — the answer to lesson 24's inherited item (b).
  `limited-result-max-page-size` **250**, `adaptive-point-in-time-max-page-size` **500**,
  `adaptive-point-in-time-max-shards` **1**, `scroll-keep-alive` **60 s** (`GLOBAL_OFFLINE`),
  `paging-mode` default `adaptive_point_in_time`, `allow-partial-results` **false**. All three page-size
  options carry `ConfigOption.positiveInt()`. The documented measurements: on ES 9.5.4 a PIT read pages of
  an index of 2–3 shards **4% to 37% slower** than a scroll even in index order, and kept up with a scroll
  at pages up to 500 hits but not 1,000. `index.max_result_window` is 10,000 by default and **JanusGraph
  lifts it only on the indexes it creates** (`MAX_RESULT_WINDOW` at `:586`). The doubling that affects an
  ES-backed query is `LimitAdjustingIterator`'s, one layer up in `QueryProcessor`.
- **`register` (`:1010-1047`), and what it can leave behind.** Validates mapping against data type
  (`:1015-1017`); rejects `PREFIX_TREE` on ES 8+ with a logged `IllegalArgumentException` — **not** a
  `BackendException` — (`:1018-1023`); then either checks an external mapping, throwing
  `PermanentBackendException` if it is absent or non-dynamic and lacks the field (`:1025-1038`), or calls
  `checkForOrCreateIndex` **and** `pushMapping` (`:1039-1046`). Called synchronously from inside the
  management transaction, so a stray ES index or an orphaned field mapping are both reachable states with
  no compensating code at either end.
- **`getNewDocument` (`:1173-1217`) de-duplicates in a `LinkedListMultimap` and resolves `SINGLE` with
  `Iterators.getLast`** (`:1190`), so last-wins is decided by additions-list order. `SET`/`LIST` go through
  `.toArray()` and reject `byte[]` with a `checkArgument` **inside a stream `filter`** (`:1197-1201`) — a
  side-effecting predicate, sound only because the stream is sequential and fully consumed. A dual-mapped
  key is written twice, under `key` and `key__STRING` (`:1209-1211`). `convertToEsType` (`:1219-1238`)
  narrows every whole `Number` to `long` and every other to `double`, converts `Instant` → `Date`, `UUID` →
  `String`, and throws `IllegalArgumentException` on anything else.

### Verified by lesson 25, for lesson 28 and the capstone to cite rather than re-derive

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
