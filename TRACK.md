<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 26
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-22 at the Part V/VI boundary (lesson 24) — PERFORMED, and the pin stays
  for the FIFTH review in a row. Lesson 26 was mid-Part and correctly did NOT re-raise it, as lesson 25
  also did not. The method is blob-SHA identity and nothing weaker: every file cited by line number has
  its git blob SHA computed locally and compared against what the API reported. Lesson 25 established the
  cheap form — a directory listing with fields: ["name","sha","size"] returns every file's blob SHA in ONE
  call, so a large file read through the over-limit path is verified with no transcription step at all.
  Lesson 26 used it on ElasticSearchIndex.java (119,295 bytes = d80949e3), matching on the first attempt,
  which is incidental confirmation that the pin has not moved under the track. Lesson 26 also used
  DELEGATED transcription for one of this repository's own files (tracks/janusgraph-registers-2.md,
  18,388 bytes = 642bd570), verified the same way — the first time the method was applied to a file being
  edited rather than read.
  NEXT REVIEW: the Part VI boundary — after lesson 28, or at the capstone. Lesson 27 is mid-Part and must
  NOT re-raise it.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (candidates 1-64, FROZEN, plus the "Still open, and who owns
  it" ledger) and tracks/janusgraph-registers-2.md (candidates 65-73, next free 74)
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open. **Part VI opened on 2026-11-23 with lesson 25. Lesson 26 shipped 2026-11-24. Two lessons remain: 27 and 28.**

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
each line names. Next run, lesson 24's block and paragraph are the ones that go.

**On 2026-11-22 the one-line entries for lessons 01–19 were dropped from this log; lesson 20's went on
2026-11-23 and lesson 21's this run.** They carried
nothing the rest of the repository does not carry better — the **Syllabus** below lists all 28 lesson
titles as taught, and the root [`README.md`](README.md) index links every entry and every digest by date
and number — while costing ~23 KB of a file that is read in full at the start of every run and
re-emitted in full on every push. Their full text is in git history at the commit immediately preceding
the 2026-11-22 entry. Entries roll off this log one per run from here; the window is the five most
recent lessons.

- **26 — 2026-11-24** · [`2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md`](2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md) · **Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the mixed-index SPI and its Elasticsearch implementation at the pinned commit. **Blob-SHA verified and cited by line number:** `ElasticSearchIndex` (2,028 lines, 119,295 bytes = `d80949e3`; cited at `:120`, `:185-188`, `:215-222`, `:224-232`, `:234-250`, `:252-266`, `:526-536`, `:543-562`, `:564-566`, `:568-576`, `:770-800`, `:802-813`, `:815-824`, `:826-835`, `:837-844`, `:846-854`, `:855-876`, `:866-868`, `:869-873`, `:874-875`, `:881-888`, `:897-916`, `:932-958`, `:1010-1047`, `:1015-1017`, `:1018-1023`, `:1025-1038`, `:1039-1046`, `:1162-1167`, `:1169-1171`, `:1173-1217`, `:1179-1192`, `:1197-1201`, `:1219-1238`, `:1275-1381`, `:1292-1295`, `:1300-1306`, `:1307-1350`, `:1314-1315`, `:1319`, `:1322`, `:1326-1330`, `:1336`, `:1344`, `:1346`, `:1352-1356`, `:1364-1368`, `:1369-1380`, `:1371-1372`, `:1383-1406`, `:1408-1422`, `:1424-1462`, `:1436-1441`, `:1451-1452`, `:1457-1458`, `:1459-1461`, `:1464-1468`, `:1507-1708`, `:1709-1728`, `:1718`, `:1730-1735`, `:1794-1804`, `:1806-1825`, `:1816`, `:1827-1847`, `:1854-1856`, `:1864-1870`, `:1872-1881`, `:1883-1925`, `:1928-1942`, `:1944-1956`, `:1959-1962`, `:1964-1967`, `:1995-2007`). **Read inline, member-name citations only:** `IndexProvider` (`b7b342bb`), `IndexFeatures` (`6651e246`), `IndexInformation` (`ded18cc9`), `IndexMutation` (`d2c44e46`), `IndexSelectionUtil` (`3ec0ec2a`), `AbstractESCompat` (`89543cd9`), and all four compat subclasses read in full — `ES6Compat` (`d686ce02`), `ES7Compat` (`fb0efe32`), `ES8Compat` (`bbd2b413`), `ES9Compat` (`73af9fc2`). **Seen only as `search_code` fragments:** `AdjacentVertexHasUniquePropertyOptimizerStrategy`, `AbstractIndexSelectionStrategy`, `BruteForceIndexSelectionStrategy`, `ApproximateIndexSelectionStrategy`, `IndexSerializer`, `HasStepFolder`, `IndexSelectionUtilTest`. Three directory listings with `fields: ["name","sha","size"]`; two `search_code` queries, both returning results. Three PR candidates logged (**71-73**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file — **the register was brought to local disk by a delegated transcription and blob-SHA verified (18,388 bytes = `642bd570`) before being appended to**, the first time that method was used on a file being edited rather than read. Daily Diff: **today's edition had not published** (`2026-11-24.md` and `2026-11-23.md` both client errors), so the **fallback pool was drawn on for the first time** — the uncovered **2026-11-20** edition, 50 items, read in full across two fetches; deep dive on **Valkey 9.2's forkless snapshots**.

  **Findings.** **THE finding: the index SPI is the only write path in JanusGraph that is not idempotent by construction, and `LIST` cardinality is the entire reason.** Audit the five mutation primitives against the Painless scripts and exactly two are non-idempotent: `ctx._source[field].add(value)` for a `LIST` field, which appends a duplicate on a second application, and `remove(indexOf(value))`, which removes another occurrence or none. `SET` is saved by one guard — `field.cardinality != 'SET' || indexOf(value) == -1` — and every `SINGLE` operation is naturally idempotent (`ctx._source.remove(name)` on an absent key, and `putAll(params.doc)` rewriting the same value). **The fix reached up into the core rather than down into the backend**: `IndexMutation` gained a `completeDocument` field carrying the element's whole indexed state, *"supplied once, from the element's state at commit"*, with the design note that *"reading the element again would read the same state"* — so it costs no read. `addUpdatesCarryingTheCompleteDocument` (`:855-876`) then collapses every change to one document into **one** update whose upsert is the complete document, exploiting three mechanisms at once: Elasticsearch **writes the upsert and skips the script** when the document is missing, so "recreate whole" and "apply the delta" become the same request; one item per document is the **maximum atomicity** the bulk API offers; and a retried item therefore never replays a *part* that had applied. The in-source comment at `:846-854` is the clearest statement of the problem in the codebase. Second: **`mutate` has three request shapes and the non-idempotent one is still live.** Shape A is a whole-document delete, guarded on `hasDeletions() && isDeleted()`; shape B is the single complete-document update; shape C — reached whenever `completeDocument` is `null` for an existing document — emits up to **three** separate bulk items for one `_id` (field-deletion script `:1314-1315`, addition script `:1336`, `{"doc": …}` merge `:1344`), with `upsert` deliberately `null` when deletions are present (`:1326-1330`). That is exactly the shape shape B's own javadoc argues against (candidate **73**, reachability untraced: `IndexSerializer` and `IndexTransaction` were not read). Third: **the retry machinery and the idempotence machinery are a matched pair.** `retainUnappliedDocuments` (`:770-800`) **mutates the caller's map in place** — a contract declared in `IndexProvider.mutate`'s own javadoc, *"a caller must not rely on the map being unchanged after a failure"* — in four independent steps, each wrapped by `narrowQuietly` (`:805-813`) so that an immutable map simply resends that store whole, and guarded by `namesPendingDocuments` (`:826-835`) so that a response naming documents the mutation no longer holds narrows nothing. Every step fails **toward** resending more, which is only sound because shape B makes the duplicate work correct. The residual hazard the lesson does **not** claim either way: shape B guarantees all-or-nothing within a document, not script idempotence against a document that already holds the outcome, so a bulk item that applied and was then reported failed still double-applies a `LIST` add. Fourth: **`register` is a non-transactional side effect of a transaction.** Called synchronously from `ManagementSystem.addIndexKey:802` (lesson 25), it can `checkForOrCreateIndex` **and** `pushMapping` before the management transaction commits, and nothing compensates in either direction — a failed register can leave a stray ES index, and a successful register followed by a rollback leaves ES holding a mapping the schema has never heard of. ES mappings being additive and practically irremovable, the drift accumulates one way, with no documented repair. Fifth, **debt (d) is settled and the answer is "no filter at all"**: `IndexSelectionUtil` has six public static methods and **not one reference to `SchemaStatus`**; `existsMatchingIndex`/`getMatchingIndexes` apply only the caller's `Predicate<IndexType>`, and `AdjacentVertexHasUniquePropertyOptimizerStrategy` passes `indexType -> indexType.isCompositeIndex() && …`. So a unique composite index at `INSTALLED`, `REGISTERED` or `DISABLED` still drives the rewrite although the composite read gate is `== ENABLED`. **The absence is verified; the consequence is not** (candidate **72**, Hands-On B). Sixth, a **correction to the handover**: there is no `supportsExistsQuery()` in `IndexFeatures` at the pinned commit — the file was read in full and the nearest member is `supportsGeoExists()`. Seventh: **all four `*Compat` classes produce identical `IndexFeatures`**, each being `coreFeatures().setWildcardField(CUSTOM_ALL_FIELD).supportsGeoContains().build()`, so `coreFeatures()`'s `setWildcardField("_all")` is **unreachable** (candidate **71**) and version dependence lives in a boolean field, an `IllegalArgumentException` and an `unmapped_type` comment instead. `supportsDocumentTTL` is `false` for every ES version, leaving `IndexMutation.determineTTL()`'s uniform-TTL `checkArgument` with no ES consumer. Eighth, inherited item (b) settled: **Elasticsearch has no page-doubling equivalent of `SolrIndex`'s** — it sizes pages once, with three options (`limited-result-max-page-size` **250**, `adaptive-point-in-time-max-page-size` **500**, `adaptive-point-in-time-max-shards` **1**), all three carrying `positiveInt()` verifiers where lesson 24's candidate 66 found none on the core batching options. The doubling that does affect an ES-backed query is `LimitAdjustingIterator`'s, one layer up.

- **25 — 2026-11-23** · [`2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md`](2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md) · **Part VI opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the management and index-lifecycle surface at the pinned commit. **Blob-SHA verified against the directory listing and cited by line number:** `ManagementSystem` (1,815 lines, 86,864 bytes = `a722a7d9`; cited at `:190-207`, `:209-213`, `:241-268`, `:253`, `:256-257`, `:340-393`, `:365`, `:367`, `:368`, `:387`, `:389-391`, `:622-639`, `:628-639`, `:655-679`, `:704-706`, `:720-724`, `:746-767`, `:758`, `:770-808`, `:785-791`, `:791`, `:802`, `:807`, `:831-879`, `:841-846`, `:852`, `:860`, `:877`, `:967`, `:977`, `:980-1140`, `:987-996`, `:1000-1019`, `:1013-1016`, `:1026-1032`, `:1033-1051`, `:1052-1056`, `:1100-1112`, `:1130-1135`, `:1142-1147`, `:1154-1166`, `:1207-1301`, `:1227-1287`, `:1237-1242`, `:1244-1254`, `:1289-1299`, `:1303-1305`, `:1310-1319`, `:1321-1326`, `:1328-1341`, `:1334`, `:1343-1361`, `:1351`, `:1392-1459`, `:1436-1458`) and `ManagementLogger` (505 lines, 24,707 bytes = `1980d1fb`; `:71`, `:169-230`, `:176-207`, `:208-224`, `:235-237`, `:239-260`, `:275-286`, `:288-296`, `:306-315`, `:317-326`, `:328-339`, `:341-357`, `:382-441`). **Read inline, member-name citations only:** `SchemaStatus` (`7b40611d`), `SchemaAction` (`b2bf7951`), `JanusGraphIndexWrapper` (`94d0458d`), `GraphIndexStatusWatcher` (`3ebe2f6a`), `AbstractIndexStatusWatcher` (`e0ab9913`), `IndexUpdateJob` (`f1b9b521`), `IndexRepairJob` (`434abee9`). Two directory listings, one with `fields: ["name","sha","size"]` — **the cheap SHA-verification method this run introduced**. One `search_code` query, returning nothing. Three PR candidates logged (**68-70**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same run as this file, as the second of two consecutive pushes. Daily Diff: the **2026-11-22** edition (38 items, **read in full**), deep dive on **Apache Kvrocks**, Redis mapped onto RocksDB, with a sidebar on **ParadeDB**'s filter-aware vector search.

  **Findings.** **THE finding: most indexes never enter the lifecycle at all, and one expression decides it.** `canIndexBeEnabled` — `oneNewKey || (constraint != null && constraint.isNew())` at `:852` for a composite index, `type.isNew() || any sortKey isNew` at `:365` for a vertex-centric one — writes `ENABLED` instead of `INSTALLED` (`:860` / `:367`) and skips `updateIndex(index, REGISTER_INDEX)` entirely (`:877` / `:389-391`). It is **sound**, not a shortcut: a composite index is all-or-nothing over its fields, so a key created in this management transaction means no pre-existing element can qualify and the backfill is provably empty. It is also asymmetric in a way the docs do not state — **one** new key out of nine is enough, and a brand-new `indexOnly(...)` constraint is enough on its own. Second: **"the index's status" is two physically different things.** Composite and relation-type keep one `SchemaStatus` as a `BaseKey.SchemaDefinitionProperty` tagged `TypeDefinitionCategory.STATUS` (`setStatusVertex:1328-1341`, one property remove plus one add, idempotent at `:1334`). A **mixed** index's index-level status is set to `ENABLED` **unconditionally** at `:758` and means nothing; the real statuses are **per field**, held as the **last element** of each `INDEX_FIELD` schema edge's `Parameter[]`, written once by `addIndexKey:791` and changed by `setStatusEdges:1343-1361` by **removing and re-adding the edge** — carrying `MAPPED_NAME`, the backend field name, along in the copy. The "STATUS is last" contract is established in one place (`:785-791`) and checked only by an `assert` (`:1351`), off in production. `JanusGraphIndexWrapper.getIndexStatus(key)` hides the whole split by **ignoring its key argument** on a composite index, which makes `GraphIndexStatusWatcher`'s per-key poll report the same fact N times there and makes it essential on a mixed index. So **a mixed index is not one lifecycle but N, one per `addIndexKey` call.** Third: **the INSTALLED → REGISTERED transition is a callback in the issuing JVM's heap and nothing persists it.** `REGISTER_INDEX` sets `INSTALLED` and arms an `UpdateStatusTrigger`; `commit` sends the eviction **after** `transaction.commit()` (`:253`, `:256-257`); `EvictionTrigger`'s constructor snapshots the open-instance roster; each receiver queues a `SendAckOnTxClose` that waits for **every transaction open when the message arrived** to close; `runTriggers()` then opens a **new** management transaction and writes `REGISTERED`, but only for elements still `INSTALLED`, so a racing `DISABLE_INDEX` supersedes it. **What `REGISTERED` actually guarantees is therefore not "every instance has seen the change" but "every instance has seen it and every transaction that predated it has finished"** — which is what makes a reindex safe. Lose the issuing JVM mid-handshake and the index is stranded at `INSTALLED` with no recovery path in the code (re-issuing `REGISTER_INDEX` works, undocumented, because `INSTALLED` is in its applicable set); lose a *receiving* instance and `graph.management-auto-close-stale-instances` plus the ack timeout is the only escape, `mgmt.forceCloseInstance` the manual one. Fourth, **lesson 23's debt (c) is settled**: a reindex of a composite or relation-type index relies on the **vertex** status, a reindex of a mixed index on the **field** statuses, both via `IndexRepairJob.validateIndexStatus` — whose mixed branch explicitly **skips `DISABLED` fields** (`status != SchemaStatus.DISABLED && !acceptable`) and fails only when no field is acceptable, so a partly-disabled mixed index reindexes while a composite one has no partial mode. Its javadoc still says *"ENABLED or REGISTERED"* where the set it uses is `REINDEX`'s four statuses. Debts (a) and (b) also settled: **every** vertex-centric index's backing relation type is invisible without being a system type (`:368` calls `maker.invisible()`), and the `RELATIONTYPE_INDEX` schema edge is written at `:387`. Debt (d) is **not** settled — `IndexSelectionUtil` was not located by the one `search_code` query spent on it, and the lesson declined to guess; it passes to lesson 26. Fifth: **`REINDEX` only adds and `REMOVE_STALE_ENTRIES` only removes, and they scan different stores** — edgestore versus graphindex — so returning a `DISABLED` index to trustworthy is two complete passes over the data, and a delete that lands while an index is disabled is invisible to a reindex forever. `REMOVE_STALE_ENTRIES` is also the only action that changes no status and is unsupported for vertex-centric indexes. Sixth: **the reindex-to-enable step is a best-effort callback** (`:1436-1458`) that catches every `Throwable` and logs at error, so a fully successful scan can silently leave the index `REGISTERED`; and `REINDEX`'s applicable set contains `DISABLED`, so reindexing an index that is receiving no writes is permitted and is almost always wrong. Three PR candidates: **68** (both `return null` branches in `updateIndex` are unreachable because `isApplicableStatus` throws rather than returning `false`), **69** (the mixed-index status is a positional array slot behind an `assert`, in the same array as `MAPPED_NAME`), **70** (the pending INSTALLED → REGISTERED callback is unpersisted and JVM-local — the only one of the three with user-visible consequences, and **not yet confirmed by a run**).

- **24 — 2026-11-22** · [`2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md`](2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md) · **Part V complete. The Part V/VI re-pin review was PERFORMED and the pin stays for the fifth time. All four debts lesson 23 handed over are discharged.** Read the whole TinkerPop step and multi-query surface at the pinned commit, **fourteen files transcribed to disk and blob-SHA verified locally**, all matching on the first attempt: `JanusGraphStep` (374 lines, 17,030 bytes = `315640d2`; cited at `:61`, `:63-70`, `:73-123`, `:77-122`, `:78-83`, `:86`, `:88`, `:89-90`, `:94-99`, `:101-104`, `:105`, `:107`, `:109-121`, `:110`, `:116-118`, `:119-120`, `:125-142`, `:130-132`, `:134-138`, `:144-163`, `:153-157`, `:158`, `:161`, `:165-172`, `:174-181`, `:179`, `:183-196`, `:185-186`, `:188-190`, `:192`, `:193`, `:198-203`, `:205-241`, `:214-218`, `:254`, `:265`, `:270-278`, `:285-289`, `:322-334`, `:336-357`, `:344-355`, `:359-370`), `JanusGraphVertexStep` (197, 8,152 = `d328dff0`; `:49`, `:51-54`, `:56-69`, `:60-68`, `:71-77`, `:74-76`, `:79-98`, `:100-115`, `:109`, `:112`, `:113`, `:117-129`, `:131-137`, `:174`, `:193-196`), `MultiQueriable` (65, 3,310 = `8496dfe1`), `JanusGraphMultiQueryStep` (182, 7,262 = `1f5e9a63`; `:40`, `:48-50`, `:89-104`, `:93-103`, `:106-114`, `:116-124`, `:117`), `JanusGraphMultiQueryStrategy` (260, 13,779 = `60fa1caf`; `:45`, `:47`, `:58-107`, `:59-61`, `:68-72`, `:74-77`, `:80-94`, `:96-104`, `:101-103`, `:118-128`, `:113-114`, `:130-147`, `:133-142`, `:138`, `:146`, `:149-157`, `:165-210`, `:175-177`, `:179`, `:181-201`, `:199-201`, `:200`, `:203-208`, `:207`, `:212-233`, `:218`, `:219-222`, `:224-227`), `JanusGraphUnusedMultiQueryRemovalStrategy` (94, 4,348 = `225ead59`; `:41`, `:47-71`, `:63-70`, `:73-84`), `JanusGraphNoOpBarrierVertexOnlyStep` (64, 2,746 = `2810c6f3`; `:30`, `:44-57`, `:46-48`, `:49`, `:53-55`), `JanusGraphTraversalUtil` (431, 22,204 = `8290888d`; `:254-312`, `:259-263`, `:319-321`, `:331-345`, `:334-343`, `:350-365`, `:361`, `:373-380`, `:382-388`, `:390-403`, `:414-420`, `:428-430`), `JanusGraphRepeatStep` (148, 7,289 = `3a6f1b34`; `:42`, `:55`), `MultiQueryPositions` (28, 1,003 = `5b47c194`; `:24-26`), `MultiQueriableStepBatchFetcher` (146, 7,250 = `e4a25136`; `:32`, `:36-42`, `:55-69`, `:71-81`, `:83-91`, `:93-97`, `:99-109`, `:104`, `:115-125`, `:117-118`, `:127-129`, `:131-138`, `:140-142`), `VertexStepBatchFetcher` (41, 1,700 = `d226dcef`; `:24-40`), `BatchProcessingQueue` (94, 2,693 = `141dcb7e`; `:29-33`, `:40-42`, `:44-53`, `:48`, `:55-65`, `:67-77`, `:87-89`), `GraphDatabaseConfiguration` (2,191, 137,928 = `4d5a9167`; `:365-366`, `:368-377`, `:379-384`, `:386-391`, `:401`, `:403-415`, `:668-670`, `:1702-1704`, `:1778-1796`, `:2058`, `:2069-2073`). **Read inline, member-name citations only:** `JanusGraphMultiQueryStrategyTest`, `OptimizerStrategyTest`. **Seen only as `search_code` fragments:** `MultiVertexCentricQueryBuilder`, `QueryProfiler`, `JanusGraphAssert`. Three directory listings; three `search_code` queries, all returning results; `list_commits` on `master`. Three PR candidates logged (**65-67**) in the **new** [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file. Daily Diff: the **2026-11-21** edition (47 stories), deep dive on **Arctic**, the OSDI '26 practical lock-free adaptive radix tree.

  **Findings.** **THE finding: the batch size of a `JanusGraphVertexStep` is written by two strategies, and on the shipped default the second write always wins.** `JanusGraphVertexStep.batchSize` defaults to **`Integer.MAX_VALUE`** (`:54`), not to zero or one. `JanusGraphLocalQueryOptimizerStrategy` pass 1 never calls `setBatchSize` (lesson 23) and pass 5 calls `setBatchSize(txVertexCacheSize)` (`:170`, default **20000**). Then `JanusGraphMultiQueryStrategy:179` executes `boolean barrierSizeSet = !limitedBatch;` — and `query.batch.limited` defaults to **`true`** (`GraphDatabaseConfiguration:379-384`), so `barrierSizeSet` starts `false` and the strategy writes a size over whatever pass 5 left: a user `barrier(n)` if there is one (`:219-222`), else the generated barrier's size (`:224-227`) which is `query.batch.limited-size`, default **2500** (`:386-391`), else `limitedBatchSize` directly (`:199-201`). **So on a default graph `g.V().out()` and `g.V().local(out())` both batch at 2500 and `cache.tx-cache-size` has no effect on batching at all.** The asymmetry lesson 23 asked about only becomes observable with `query.batch.limited=false`, where `barrierSizeSet` starts `true`, no `setBatchSize` call happens anywhere in the strategy, and the top-level step keeps `Integer.MAX_VALUE` — one multi-query over every start vertex — while the local one keeps 20000. The two halves agree: at `limited=false` `JanusGraphMultiQueryStep.initialize()` (`:93-103`) also drains its **entire** upstream into an `ArrayList` before passing it on. **Nobody has run the experiment; Hands-On A is it.** Second: **the two steps are not a matched pair.** `JanusGraphStep` implements `HasContainerHolder` and compiles a `GraphCentricQuery` inside itself; `JanusGraphVertexStep` implements `MultiQueriable` and only forwards arguments to a `BaseVertexQuery` builder. Only `JanusGraphStep` supports an offset (`setLimit`'s low bound, consumed by `MultiDistinct*Iterator` at `:110`/`:119`, never passed to a query); `JanusGraphVertexStep.setLimit` asserts `low == 0` (`:174`). Only `JanusGraphVertexStep` batches. Third: **both of `JanusGraphStep`'s execution arms converge on the same merge**, so the de-duplicating offset-applying `MultiDistinctUnorderedIterator`/`MultiDistinctOrderedIterator` is *the* execution path, not a special case for split `or()`s — and the split arm gives each of N branch queries the **full** `highLimit` (`:179`), so a `limit(10)` over a three-branch `or()` can read 30 elements. The `ids != null && ids.length > 0` arm consults **no index at all**: `iteratorList`/`holdsConditions` (`:322-357`) re-check every folded `has()` and every `or()` in Java. Fourth, **debt (b) is fully settled: all four multi-query annotations are set in one class, `MultiVertexCentricQueryBuilder`** — `multi=true` and `vertices=N` in `constructQuery`, `partitioned=true` in the partitioned-vertex branch, `multiPreFetch=true` in `preFetch()`. **`vertices` is the only one carrying a number, and it is the batch size actually used**, which makes it the observable for Hands-On A. Fifth, the **insert-then-remove** design is deliberate and javadoc'd (`JanusGraphMultiQueryStrategy:113-114`): a `JanusGraphMultiQueryStep` is inserted before every `MultiQueriable` *and* every `TraversalParent`, and `JanusGraphUnusedMultiQueryRemovalStrategy` deletes the ones with three empty client lists **together with the barrier they generated**, identified by reference equality on `getGeneratedBarrierStep()` — the same identity test that distinguishes a generated barrier from a user's `barrier(n)` when choosing a batch size (`JanusGraphTraversalUtil:361`). Sixth: `JanusGraphNoOpBarrierVertexOnlyStep` is a four-line subclass whose entire content is `break`ing out of the fill loop on a non-vertex traverser (`:53-55`) plus refusing to top up a non-empty barrier (`:46-48`). Three PR candidates: **65** (`isLegalMultiQueryPosition` walks `currentStep` and tests `step`, so the walk is a no-op and the answer depends only on the position step's own labels — the one reportable defect, and **not yet confirmed by a run**), **66** (`BatchProcessingQueue.setBatchSize` drops the constructor's `Math.max(batchSize,1)`, and neither `query.batch.limited-size` nor `cache.tx-cache-size` has a `positiveInt()` verifier where `MAX_KEYS_PER_READ` at `:401` does), **67** (`JanusGraphVertexStep`'s copy constructor does not copy `batchSize`; latent, no reachable path found).

- **23 — 2026-11-21** · [`2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md`](2026/11/2026-11-21-janusgraph-optimizer-strategy-layer/README.md) · **Findings.** Paragraph compressed 2026-11-24 under the roll-off rule, and the `Verified by lesson 23` block removed in the same run; the full versions are in git history at the 2026-11-21 through 2026-11-23 commits, and the facts lessons 27 and 28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the content of the strategy layer is its dependency graph, and two of the twelve strategies declare no ordering at all** — `StandardJanusGraph`'s one static block registers twelve `ProviderOptimizationStrategy` objects and OLAP gets one of them; **`local()` is the only construct that gets an order or a range to the storage layer**, pass 1 folding neither; `or()` is folded only for a `JanusGraphStep` and merged *in the step*, so the TinkerPop side never produces a multi-sub-query `ElementQuery`; and a traversal ending in an aggregation is compiled during strategy application under `QueryProfiler.NO_OP` and **memoized**, so `.profile()` shows no compile span at all. `AdjacentVertexHasUniquePropertyOptimizerStrategy` is the one strategy permitted to touch the backend, gated on `optimizerBackendAccess()`, and it runs a side traversal during strategy application — which is what makes lesson 26's candidate 72 worth running. Four PR candidates, **61-64**.

- **22 — 2026-11-20** · [`2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md`](2026/11/2026-11-20-janusgraph-query-processor-iterator-stack/README.md) · **Findings.** Paragraph compressed 2026-11-23 under the roll-off rule, and the `Verified by lesson 22` block removed in the same run; the full versions are in git history at the 2026-11-20 through 2026-11-22 commits, and the facts lessons 26, 27 and 28 were promised are carried in *Notes For The Next Run* below. The one-line substance: **the in-memory filter sits between the backend limit and the element count**, so `limit(10)` bounds what the caller receives and not what is read, and the backend limit **doubles** — `L, 2L, 4L …` with a full cursor-less re-read each time — until enough survive; **`QueryProcessor` creates no profiler span at all**, so merge, sort and filter time is invisible in `.profile()`; half the class is dead (`VertexCentricQuery.isSorted()` hard-coded `true`, `GraphCentricQuery.numSubQueries()` hard-coded `1`); and the single-slice fitted sorted vertex-centric case does not use `QueryProcessor` at all. Nine PR candidates, **52-60**.


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
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30. Two lessons remain: 27 and 28.**

**Re-pin: NOT re-raised, correctly.** Lesson 24 performed the Part V/VI review on 2026-11-22 and the pin
stays for the fifth time. Lessons 25 and 26 were mid-Part and did not re-raise it; **lesson 27 must not
either.** The next review is the Part VI boundary — after lesson 28, or at the capstone. Lesson 26 did
re-verify the one large file it cites by line number by blob SHA (`ElasticSearchIndex.java`, 119,295
bytes = `d80949e3`), matching what the directory listing reported.

**THE fact to carry forward from lesson 26: the index SPI is the only write path in JanusGraph that is
not idempotent by construction, and `LIST` cardinality is the whole reason.** Of the five mutation
primitives, `SINGLE` delete, `SINGLE` assign and `SET` add are idempotent (the last by the explicit
`field.cardinality != 'SET' || indexOf(value) == -1` guard); `LIST` add appends a duplicate on a second
application and `LIST`/`SET` remove removes another occurrence or none. **The fix was a core SPI change,
not a backend change**: `IndexMutation.completeDocument` carries the element's whole indexed state,
supplied free at commit, and `addUpdatesCarryingTheCompleteDocument` (`ElasticSearchIndex:855-876`)
collapses a document's changes into one update whose upsert is that whole document — Elasticsearch writes
the upsert and **skips the script** when the document is missing, and one item per document is the maximum
atomicity the bulk API offers. Lesson 27 should hold this as the contrast case: a `KeyColumnValueStore`
mutation is idempotent because a cell write is last-writer-wins on a fixed column, which is exactly what
a JSON array field is not.

**Second, the structural fact to keep straight: `mutate` has three request shapes and the
non-idempotent one is still live.** Shape A whole-document delete (guarded `hasDeletions() &&
isDeleted()`); shape B the single complete-document update; shape C — whenever `completeDocument` is
`null` for an existing document — up to **three** bulk items for one `_id`, which is the shape shape B's
own javadoc argues against. **Candidate 73's reachability is untraced**: `IndexSerializer` and
`IndexTransaction` were not read, so which core call sites invoke `setCompleteDocument` is unknown. One
`search_code` query settles it and it is the cheapest open item in the register.

**Third, debt (d) is settled: `IndexSelectionUtil` applies no `SchemaStatus` filter anywhere.** Six public
static methods, not one reference to `SchemaStatus`. `existsMatchingIndex`/`getMatchingIndexes` collect
`((InternalRelationType) key).getKeyIndexes()` per `PredicateCondition` and apply only the caller's
`Predicate<IndexType>`; `AdjacentVertexHasUniquePropertyOptimizerStrategy` passes
`indexType -> indexType.isCompositeIndex() && …` with no status test. So a unique composite index at
`INSTALLED`, `REGISTERED` or `DISABLED` still drives the rewrite although the composite read gate is
`== ENABLED`. **The absence is verified; the consequence is not** — candidate 72, Hands-On B. Do not
restate this as a confirmed defect without the run.

**Fourth, a correction worth not losing: there is no `supportsExistsQuery()` in `IndexFeatures` at the
pinned commit.** The file was read in full; it has eleven flags and the nearest member is
`supportsGeoExists()`. The lesson-24 handover named a method that does not exist. The three real flags
are `supportNotQueryNormalForm()` (no trailing `s` on the accessor), `supportsOrderingListProperty()` and
`IndexInformation.supportsOrdering(KeyInformation)`, the last being a `default true` on the interface.
Also settled: `IndexSerializer.allMixedIndexBackendSupportsOrderingListProperty()` **returns `false` when
there are no mixed indexes at all**, so a pure-composite graph cannot fold a vertex order on a `LIST`
key — the empty-set quantifier is implemented as `false` rather than as vacuous truth.

**Fifth, the operational fact: `register` is a side effect that no rollback undoes.** Called synchronously
from `ManagementSystem.addIndexKey:802`, `ElasticSearchIndex.register` can create the ES index
(`checkForOrCreateIndex`) and push a field mapping before the management transaction commits. A failed
register can leave a stray ES index; a successful register followed by a rollback leaves ES holding a
mapping the schema has never heard of. ES mappings are additive and practically irremovable, so the drift
is one-directional with no documented repair. Lesson 28 should state this as one of the three concrete
ways the edgestore and a mixed index can disagree.

**Sixth, the unfinished business, now three lessons old: eleven Hands-On experiments are outstanding** —
lesson 24's three, lesson 25's four and lesson 26's four. **Candidate 65 must not be filed upstream until
lesson 24's Hands-On B has been run; candidate 70 not until lesson 25's Hands-On D; candidate 72 not until
lesson 26's Hands-On B.** Lesson 26's Hands-On B is the cheapest of the three — `inmemory`, one unique
composite index, `DISABLE_INDEX`, re-profile, compare `toList()` against the baseline, no Docker — and it
settles a question the track has carried since lesson 23.

**Carried from lesson 24, unchanged: the batch size of a `JanusGraphVertexStep` is written twice and on
the default configuration the second write wins.** Default field value `Integer.MAX_VALUE`; pass 5 of
`JanusGraphLocalQueryOptimizerStrategy` writes `txVertexCacheSize`; `JanusGraphMultiQueryStrategy` then
writes 2500 over it because `boolean barrierSizeSet = !limitedBatch` (`:179`) and `query.batch.limited`
defaults to `true`. **So `cache.tx-cache-size` has no effect on batching on a default graph**, and the
pass-1/pass-5 asymmetry is only observable at `query.batch.limited=false`. Do not re-derive it and do not
state the asymmetry without the `limited` qualifier.

- **Lesson 27 (CQL) inherits four, three unchanged.** The `setDirectColumnByStartOnlyAllowed` flag from
  lesson 20; the feature-builder classification lesson 18 owed (whether `CQLStoreManager` and
  `HBaseStoreManager` set `.locking(...)`, and so whether `ConsistentKeyLocker` runs on them); from
  lesson 16, whether the CQL `BatchStatement` ordering is explicit or implicit in client-side timestamps
  (`graph.assign-timestamp`, LOCAL, default **true**); and from lesson 22, `PreSortingIterator`'s
  `MAX_SORT_ITERATION = 1_000_000` budget — **per sub-query**, reading the whole result before throwing,
  and rejecting a complete 1,000,000-element result (candidate 53) — which combined with
  `LimitAdjustingIterator`'s cursor-less doubling re-read makes an unfitted ordered query on CQL the worst
  case in the tree. Put a number on it against a real backend. **New from lesson 26:** the idempotence
  contrast above is the frame for the whole lesson — say explicitly why a `mutate` on a
  `KeyColumnValueStore` needs none of §4's machinery, and whether CQL's client-side timestamps change that
  answer. Also still owned by 27 alone: lesson 20's `BasicVertexCentricQueryBuilder:711`
  `Preconditions.checkArgument(bestCandidate != null, …)` reachability question.
- **Lesson 28 (transaction log, CDC, stale-index repair) inherits six.** (a) **The deletion window:**
  `REINDEX` only adds and `REMOVE_STALE_ENTRIES` only removes, they scan **different stores** (edgestore
  vs graphindex), and a delete that lands while an index is `DISABLED` is invisible to a reindex forever;
  returning a disabled index to trustworthy is two complete passes. (b) From lesson 22:
  `result.isRemoved()` elements are dropped **eagerly** inside `PreSortingIterator` while everything else
  the transaction deleted is filtered **lazily as reached** by `getFilterIterator`, deliberately, *"so
  that a change which the caller makes while it reads the results is seen."* (c) lesson 23's
  `MultiDistinctOrderedIterator` cross-query ordering invariant, which is an `assert`. (d) from lesson 24,
  the same invariant one layer up — `JanusGraphStep:118`'s
  `assert queries.stream().allMatch(q -> q.getOrder().isEmpty() || q.getOrder().equals(order))` is the
  only thing between two differently-ordered branch queries and a silently mis-ordered merged result, and
  assertions are off in production; a stale mixed index is exactly the condition that produces it. (e)
  from lesson 25: `REMOVE_STALE_ENTRIES` changes no status, is unsupported for vertex-centric indexes, and
  routes mixed indexes to `MixedIndexStaleEntryRemover.submit` rather than a scan job. (f) **new from
  lesson 26, and this is lesson 28's subject stated concretely — the three ways schema and mixed index
  diverge:** a `register` whose management transaction then fails or rolls back (§3.1 of the lesson 26
  entry); a `LIST` value duplicated by a retried bulk item that had already applied (§5, unproven,
  Hands-On C); and shape C's split requests, where one of three bulk items for a document can fail
  independently of the others. **Reassigned to lesson 28 from lesson 26, which did not reach them:** the
  four `mixed-index-*` scan metrics in `IndexRepairJob` (`mixed-index-flushes`, `mixed-index-flushed-docs`,
  `mixed-index-flush-time-ms` rounded not truncated, plus `doc-updates`) and
  `MIXED_INDEX_REINDEX_BATCH_SIZE` / `MIXED_INDEX_REINDEX_BATCH_ENABLED`, which also govern
  `MixedIndexStaleEntryRemover`. They sit more naturally in the stale-entry lesson than in the SPI lesson.
- **Open and unresolved, deliberately not filled in.** (a) Whether any production path passes a real limit
  to `QueryUtil.processIntersectingRetrievals` (candidate 51 is a reachability claim, not a confirmed
  defect). (b) Whether `BackendQueryHolder`'s duplicated `isFitted` in `hashCode` (candidate 54) can
  degrade anything. (c) `IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel
  colliding with `IndexCandidate`'s default score (inside candidate 50). (d) Lesson 23's §2.2 ordering
  question — is the `JanusGraphStepStrategy` / `JanusGraphLocalQueryOptimizerStrategy` order observable by
  any traversal? (e) From lesson 24: whether `BatchProcessingQueue.removeLast()` has any caller at all —
  `HasStepBatchFetcher` and `DropStepBatchFetcher` were not read, so it must **not** be called dead code.
  (f) **New from lesson 26:** whether `ElasticSearchBulkFailureException` can report an item as failed
  after it applied, which decides whether shape B has a residual duplication hazard. `TransientFailures`
  (`6aba13b8`) and `ElasticSearchBulkFailureException` (`e52ae410`) are unread. (g) **Also new:** which
  core call sites invoke `IndexMutation.setCompleteDocument`, i.e. whether shape C is reachable for a real
  update — one `search_code` query.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72,
  14 ~75, 15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, 23 ~59, 24 ~59, 25 ~51,
  **26 ~62** (digest ~25). A large share of each is permalink URLs, which are not prose.

**Mechanical notes, carried and extended this run. Read this section before planning a run's pushes.**
`get_file_contents` on a file whose doubled output exceeds roughly 40 KB returns an over-limit error that
saves the JSON to a local file — the cheap way to read a large file, costing nothing in context. The
correct extraction is `json.loads(longest_text)["content"][-1]["resource"]["text"]`, **not** the plain
`text` value (which has a `[Resource from github at repo://…]` banner glued onto line 1). A file under
roughly 20 KB comes back inline with no line numbers; such a file must be written to local disk to be
cited by line number, and **a lesson that does not do that must cite it by member name only** — lesson 26
did exactly this for the six SPI and compat files and said so in the entry. A directory listing with
`fields: ["name","sha","size"]` gives every file's blob SHA in one cheap call, so a large file read
through the over-limit path is SHA-verified against that listing with no transcription step at all; that
remains the preferred method. **Lesson 26 added one thing: the same delegated-transcription trick works
for this repository's own files.** `tracks/janusgraph-registers-2.md` came back inline (18 KB), which
would have meant retyping it to append to it; instead a sub-agent was asked to fetch it and write it to
disk verbatim with a short python script, and the result was blob-SHA verified locally (18,388 bytes =
`642bd570`) before anything was appended. **Use this for every inline-returned file that has to be
edited rather than merely read.** It turns a retyping risk into a verified copy for the cost of one
sub-agent.

**The real constraint on a run is OUTPUT, not input, and it is now the binding one.** Every file pushed
must be emitted in full as a tool argument; there is no server-side copy or patch. Reading a 134 KB file
costs almost nothing; *rewriting* it costs ~37 K output tokens. This run emitted four files —
the lesson (~62 KB), the digest (~25 KB), this file (~82 KB) and the root `README.md` (~66 KB) — plus the
register (~29 KB), which is about 265 KB of output for one day. **Watch the trend.** If this file passes
~110 KB, the next structural move is to split the `Verified by lesson NN` blocks into
`tracks/janusgraph-verified.md` — they are ~45 KB of the total, they are append-and-roll-off material,
and a run would then emit two medium files instead of one large one. **That is a decision for Aryant**,
and it is flagged rather than taken, because it changes where a run looks for the facts it is told to
cite rather than re-derive. The root `README.md` is the second such file; its index is one row per line,
so a run should edit the row in place rather than rebuild the table by re-reading every entry's
`entry-meta`. Lesson 26 did that: it fetched the README through the over-limit path, edited the Current
Track section and appended one index row with a python script, and emitted the result — no row was retyped.

**Mermaid.** Render every diagram with `mmdc` before pushing. Lesson 26's three (two in the lesson, one
in the digest) all rendered on the first attempt. The accumulated rules still hold: quote every node
label containing parentheses, commas, `->`, `|` or angle brackets; use HTML entities such as `&lt;` for
comparisons; never escape a quote inside a label — drop the inner quotes instead; and **a semicolon
inside a `sequenceDiagram` message is a statement separator** (lesson 25) — use a comma. One new one from
this run: a `sequenceDiagram` tolerates `--x` for a failed/aborted message and renders it as a crossed
arrow, which is the right mark for an error return.

**Daily Diff procedure — the fallback pool was drawn on for the first time this run, and it worked.**
`https://tdd.cat/llms.txt` documents endpoints the HTML site does not advertise: `llms-full.txt`, `/md`
and `/json` for the latest edition, **`https://tdd.cat/{YYYY-MM-DD}.md` for any edition by date**,
`/rss.xml` and `/stats/`. The procedure is: **(a)** fetch `https://tdd.cat/{today}.md`; **(b)** on a
client error walk back one day at a time; **(c)** check this session log in both directions as a floor and
a ceiling; **(d)** use the front page and `/archive/` only to cross-check the story count, which the `.md`
file does not state; **(e)** only then the uncovered-item fallback. On 2026-11-24 both `2026-11-24.md` and
`2026-11-23.md` returned client errors — **the second time 11-23 has failed, so that edition appears never
to have published rather than being late** — and 11-22 and 11-21 were already covered by lessons 25 and
24, so step (e) fired and the digest covered the **2026-11-20** edition (50 items, 77,940 characters, read
in full across two fetches at offsets 0 and 39,958). **The fallback pool now holds only the unspent items
from 2026-11-06 (two) and 2026-11-18 (two).** If 11-25 also fails to publish, the pool is nearly empty and
the next run should say so plainly rather than stretching two leftover items into a digest. Edition sizes
remain volatile: 22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61, 61, 50, 47, 38 across 11-08 to 11-22, with
11-23 and 11-24 absent. **Constraint still in force: there is no `WebFetch` tool in this environment** —
pages are fetched through a sub-agent which flags which parts are verbatim and which are an extracted
summary, and the summary portion is not reliable enough to quote. Lesson 26 hit this squarely: the
sub-agent's first extracted summary of the 11-20 edition reported 25 items and was **wrong**, and the
second, full-text read found 50. **Always require the full-text read and ignore a first-pass summary.** A
PDF or any page over ~40 K characters splits the same way, so plan a deep dive around the readable half;
the `.md` endpoint is valuable precisely because a second fetch at an offset covers the remainder.

### Verified by lesson 26, for lessons 27 and 28 to cite rather than re-derive

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

### Verified by lesson 25, for lessons 27 and 28 to cite rather than re-derive

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

### Verified by lesson 24, for lessons 27 and 28 to cite rather than re-derive

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
