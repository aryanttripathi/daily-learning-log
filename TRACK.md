<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 28
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-26 at the Part VI boundary (lesson 28) — PERFORMED, and the pin stays
  for the SIXTH review in a row. `list_commits` on `master` returned the same three newest commits as at
  lessons 13, 18, 22 and 24: `a59ae8d8`, `9a30ff14` (both 2026-10-28) and `5e3e7bc7` *JanusGraph release
  1.2.0* (2026-10-24). The repository has not moved in 29 days, so every line number in Parts I-VI is
  mutually comparable and the capstone may cite any of them without re-verifying the pin.
  The method is blob-SHA identity and nothing weaker: every file cited by line number has its git blob SHA
  computed locally and compared against what the API reported, the cheap form being a directory listing
  with fields: ["name","sha","size"] which returns every file's blob SHA in ONE call. Lesson 27 applied it
  to TEN files and all ten matched on the first attempt, which is incidental confirmation that the pin has
  not moved under the track. Lesson 27 also established that the plain `text` value of an over-limit
  get_file_contents result is LOSSY, not merely banner-prefixed, and that the resource path matches
  byte-for-byte: see the mechanical notes.
  NEXT REVIEW: none is due. Part VI is closed and the capstone adds no new reading of its own; if it does
  read new files, it re-pins only on the same rule — run list_commits on master, record the answer here.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
registers: tracks/janusgraph-registers.md (candidates 1-64, FROZEN, plus the "Still open, and who owns
  it" ledger) and tracks/janusgraph-registers-2.md (candidates 65-83, next free 84)
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open. **All 28 lessons have shipped. Lesson 28 closed Part VI on 2026-11-26; the syllabus is
complete and the next run writes the CAPSTONE.**

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
lesson 22 on 2026-11-23, lesson 23 on 2026-11-24 and lesson 24 on 2026-11-25. The full paragraphs remain
verbatim in git history at the commit each line names. **This run compressed lesson 25's paragraph and
removed its `Verified by lesson 25` block.** The capstone is the last run; it should roll off nothing, so
that lessons 26, 27 and 28 keep their full paragraphs and `Verified by` blocks as the record of Part VI.

**On 2026-11-22 the one-line entries for lessons 01–19 were dropped from this log; lesson 20's went on
2026-11-23, lesson 21's on 2026-11-24, lesson 22's on 2026-11-25 and lesson 23's this run.** They carried
nothing the rest of the repository does not carry better — the **Syllabus** below lists all 28 lesson
titles as taught, and the root [`README.md`](README.md) index links every entry and every digest by date
and number — while costing ~23 KB of a file that is read in full at the start of every run and
re-emitted in full on every push. Their full text is in git history at the commit immediately preceding
the 2026-11-22 entry. Entries roll off this log one per run from here; the window is the five most
recent lessons.

- **28 — 2026-11-26** · [`2026/11/2026-11-26-janusgraph-tx-log-stale-index-repair/README.md`](2026/11/2026-11-26-janusgraph-tx-log-stale-index-repair/README.md) · **Part VI boundary, so the re-pin review WAS performed: `list_commits` on `master` returned `a59ae8d8` as the newest commit, unchanged since 2026-10-28, and the pin stays for the sixth time.** Read the write-ahead transaction log, the recovery processor and both stale-entry removers at the pinned commit. **Blob-SHA verified against the API's own directory listings and cited by line number** (all nine matched on the first attempt): `StandardTransactionLogProcessor` (585 lines, 27,719 bytes = `300d11dd`; cited at `:92-93`, `:99`, `:120-124`, `:135-136`, `:137-140`, `:146`, `:149-151`, `:155-157`, `:158-162`, `:165`, `:166-168`, `:176-179`, `:187-188`, `:189-203`, `:190-191`, `:200-202`, `:201`, `:230-233`, `:240-247`, `:245`, `:254-256`, `:293-297`, `:299-318`, `:308-309`, `:313-314`, `:328-367`, `:330-333`, `:335-346`, `:340`, `:343`, `:345`, `:353-366`, `:371-442`, `:373`, `:374-405`, `:381-387`, `:390-399`, `:405`, `:409-441`, `:428-429`, `:437`, `:460-471`, `:481-489`, `:489`, `:504`, `:526-529`, `:536-538`), `TransactionLogHeader` (282, 12,329 = `5bc78140`; `:58`, `:60`, `:76-84`, `:80`, `:86-93`, `:101`, `:107-111`, `:113-126`, `:118`, `:124`, `:128-130`, `:132-143`, `:146-161`, `:149`, `:150`, `:151`, `:153`, `:154`, `:157-158`, `:172`, `:227-236`, `:258`), `StandardJanusGraph` (1,490, 80,408 = `f650acb2`; `:1232`, `:1244`, `:1292`, `:1295`, `:1299-1300`, `:1302-1304`, `:1304`, `:1307-1313`, `:1316-1318`, `:1321`, `:1322`, `:1330`, `:1332`, `:1334`, `:1335`, `:1338-1339`, `:1342`, `:1348-1357`, `:1353`, `:1368-1376`), `MixedIndexStaleEntryRemover` (241, 13,523 = `d44cd26b`; `:80-83`, `:90-91`, `:94-108`, `:109`, `:134-137`, `:138`, `:144-146`, `:155`, `:157-159`, `:160-167`, `:172-174`, `:176`, `:177-187`, `:189`, `:205-228`, `:215-220`, `:225-226`, `:230-240`), `StaleIndexEntryRemoveJob` (233, 9,898 = `5e29ea40`; `:53-59`, `:72-78`, `:80-85`, `:136-141`, `:160-162`, `:173-181`, `:183-200`, `:211-214`, `:216-227`), `GraphDatabaseConfiguration` (2,192, 137,928 = `4d5a9167`; `:259-261`, `:263-283`, `:278-280`, `:293-303`, `:300-303`, `:765-767`, `:769-774`, `:796-799`, `:1316`, `:1320`, `:1330-1332`, `:1342-1344`, `:1350-1352`, `:1844`), `LogTxStatus` (32, 1,069 = `3f5ba38c`; `:22`, `:24-30`), `LogTxMeta` (90, 2,584 = `5ba2f284`; `:66-68`, `:70-73`), `StandardLogProcessorFramework` (290, 11,929 = `45d5072f`; read at `:175-250` ONLY — `:178-182`, `:185-196`, `:229-245` cited, the rest unread). Four directory listings with `fields: ["name","sha","size"]`; one `search_code` query (two results); one `search_issues` query returning **six open issues**, all linked in the entry; one delegated transcription batch of six files, all blob-SHA verified. Five PR candidates logged (**79-83**) plus dated updates to **74** and **78** in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md). Daily Diff: **the 2026-11-26 edition had not published at 03:35 UTC**, and `https://tdd.cat/2026-11-25/` **also** errored although the slash form worked for 11-24 last run — so the digest was built from the **front page**, which named 2026-11-25 / 53 stories as current, with 22 of the 53 items recovered; deep dive on **incident.io's Postgres row-comparison rewrite** (33.8% of node CPU to 0.3%), sidebar on **DuckDB property-based testing**.

  **Findings.** **THE finding: the recovery system's whole correctness argument is one borrowed guarantee, and on default CQL it does not hold.** `PRIMARY_SUCCESS` is not written by the log; it is handed to `mutator.getTxLogPersistor()` (`StandardJanusGraph:1302-1304`) so that it commits inside the storage mutation, and the in-source comment states the intent — *"committed atomically with all transactional changes so that we can recover secondary failures"*. But the transaction log is a **different store** (`system_txlog`), its row key is `hashPrefixKey(SHORT, transactionId)` (`TransactionLogHeader:60`) and therefore a **different partition**, and lesson 27 established that a default CQL mutation is `.grouped(20)` fired as several independent async `UNLOGGED` batches. So the entry and the edgestore rows are separate writes that succeed and fail independently. **The two failure directions are not symmetric.** Entry lands, storage does not → recovery runs a repair which re-derives from storage and writes documents matching current truth: **harmless**. Storage lands, entry does not → the last entry is `PRECOMMIT`, the eviction listener counts it in `successTxCounter` (`:200-202`), and a later index failure in that same transaction is **never repaired** — a permanent silent divergence that the write-ahead log reports as a success. **Second, the organising principle of every repair path: the edgestore is the truth and a mixed index is a cache of it, so the log supplies element ids and never document content.** Pass 2 of `restoreExternalIndexes` (`:409-441`) does `retrieve(tx)` and then `reindexElement` if present or `removeElement` if gone — a **re-derivation, not a replay** — which is simultaneously what makes repair idempotent, convergent, concurrency-safe, and correct when the log entry itself is wrong. `MIXED_INDEX_FILTER` (`:489`) is why composite indexes have no write-ahead path at all: graphindex entries ride on the same `commitStorage`. **Third, where re-derivation runs out: the user log.** A `USER_LOG` event is a delivery, not an assignment, so recovery **re-sends** it with a new transaction id and a `SOURCE_TRANSACTION` meta (`:356-357`) — and because `userlogSuccess` is recorded `true` for a message merely *queued* (`StandardJanusGraph:1335`, with `log.send-delay` defaulting to 1000 ms so `isDone()` is normally false), the duplicate is the normal case. **A `ChangeProcessor` must be idempotent and nothing in the API says so.** That is lesson 27's assignment-versus-accumulation line arriving at the application boundary. **Fourth, three defaults that narrow the contract to almost nothing:** `tx.log-tx` defaults to **`false`**, so on a stock graph none of this runs; the gate at `:1232` is `hasLogTransactions() && !hasEnabledBatchLoading()`, so **batch loading silently disables the write-ahead log** — the one mode most likely to leave indexes behind; and `TRANSACTION_LOG_DEFAULT_TTL` is **seven days** (`GraphDatabaseConfiguration:1320`), with already-filed [#5051](https://github.com/JanusGraph/janusgraph/issues/5051) reporting that the status entries bypass the TTL while the content-bearing `PRECOMMIT` entry does not — which makes candidate 82's path reachable by design. **Fifth, the best idea in the file: the expiry clock is the log's READ PROGRESS, not the wall clock** (`:166-168`, a Caffeine ticker over `txLog.getReadProgress()` forced monotonic by `accumulateAndGet(…, Math::max)`), so a slow or lagging reader delays decisions instead of making wrong ones; `nanosUntilExpiry` (`:240-247`) adds exactly one nanosecond because read progress is exclusive and log timestamps are microseconds, and `LONGEST_RECOVERY_WAIT = Duration.ofNanos(Long.MAX_VALUE >>> 1)` (~146 years) is a half-range bound precisely so `ahead + wait + 1` cannot overflow. `expireAfterUpdate` returns `currentDuration`, so **a later entry never extends the deadline**. **Sixth, the arithmetic that makes the shipped defaults unsafe past one index backend:** `warnIfMaxCommitTimeIsTooShort` (`:1853-1879`) requires `max-commit-time` to **exceed** `storage.write-time × (1 + indexBackends)`; at the defaults 300 s and 100 s that is 200 s for one backend (fine) and **300 s for two, which 300 s does not exceed** — so the warning fires at stock settings with two index backends, and the javadoc's own words for the consequence are that recovery can *"restore the index documents of a transaction which is still committing, underneath the index writes that commit has yet to make"*. **Seventh, two stale-entry removers that share three method bodies and no code:** `StaleIndexEntryRemoveJob` is a real `ScanJob` over graphindex with a provable 128-byte slice bound and a hard-coded `EXISTENCE_CHECKS_PER_TX = 10_000`, while `MixedIndexStaleEntryRemover` is a bare daemon `Thread` dressed as a scan job that enumerates by **one exists-query per indexed field** and recycles its check transaction every `batchSize` — so a stale document whose only values sit in non-queryable fields is **permanently invisible**, signalled by one `log.warn` naming the field rather than the gap. **Eighth, candidate 74 is narrowed and 78's gate settled:** `ASSIGN_TIMESTAMP` is default **`true`** with no verifier, so lesson 27's timestamp argument holds out of the box — but it is `ConfigOption.Type.LOCAL`, so two instances writing one Cassandra table can disagree about it through configuration drift, which is a cheaper route to the same hazard than an explicit `false`; and `storage.transactions` defaults `true`, making candidate 78's NPE configuration-specific rather than live. Five new candidates: **79** (a thrown `commitIndexes()` writes `SECONDARY_SUCCESS`, so the transaction is never repaired — reachability turns on one unread file and it is the most consequential of the five), **80** (the `maxTxLength` precondition at `:139-140` cannot fail, because the floor runs first), **81** (`removedIds` is unbounded while the element cache is carefully bounded), **82** (a transaction whose `PRECOMMIT` was never read is counted as repaired, and `getStatistics()` says so), **83** (pass 1 of a repair gets a hard-coded one second, pass 2 gets `storage.write-time`).

- **27 — 2026-11-25** · [`2026/11/2026-11-25-janusgraph-cql-store-vs-inmemory/README.md`](2026/11/2026-11-25-janusgraph-cql-store-vs-inmemory/README.md) · **Mid-Part, so the re-pin question was correctly NOT re-raised — but lesson 28 MUST perform the Part VI review.** Read the CQL storage backend against the in-memory store at the pinned commit. **Blob-SHA verified against the API's own directory listings and cited by line number** (all ten matched on the first attempt): `CQLKeyColumnValueStore` (873 lines, 47,914 bytes = `f11ea637`; cited at `:123-127`, `:141-154`, `:156-170`, `:200-201`, `:211-343`, `:217-222`, `:224-244`, `:234-244`, `:248-263`, `:255-256`, `:265-303`, `:284-285`, `:291-299`, `:305`, `:323-328`, `:332-333`, `:336-338`, `:339-340`, `:355-368`, `:370-379`, `:387-397`, `:399-411`, `:413-424`, `:432-437`, `:450-480`, `:482-491`, `:493-523`, `:499-504`, `:503-523`, `:516-520`, `:525-545`, `:529-531`, `:539-542`, `:547-583`, `:595-625`, `:613-617`, `:641-652`, `:673-684`, `:686-695`, `:718-750`, `:731-737`, `:752-774`, `:776-789`, `:802-813`, `:815-818`, `:820-872`, `:829-872`, `:840-849`, `:851-863`), `CQLStoreManager` (374, 18,740 = `09f7b055`; `:104`, `:157-158`, `:311-358`, `:346-349`, `:352-358`, `:356`), `CQLConfigOptions` (48,346 = `2f1adabe`; `:47-55`, `:64-76`, `:85-89`, `:91-102`, `:105-110`, `:113-118`, `:120-130`, `:237-251`, `:253-265`, `:267-280`, `:844-862`), `CQLStoreFeaturesBuilder` (129, 7,091 = `a13eacac`; `:58-67`, `:62`, `:63`, `:65`, `:68-70`, `:80-87`, `:88-119`, `:94-100`, `:101-104`), `CQLTransaction` (60, 2,609 = `349bf72b`; `:30-47`, `:49-53`), `CQLColValGetter` (65, 2,423 = `66ea2658`; `:26-64`, `:59-60`), `AbstractCQLMutateManyFunction` (100, 5,614 = `ea850c4e`; `:35-41`, `:39-40`, `:59-91`, `:63`, `:68-77`, `:78-90`), `CQLMutateManyUnloggedFunction` (125, 6,189 = `7d3d3ee6`; `:71-124`, `:84`, `:88`, `:96-109`, `:122-123`), `CQLMutateManyLoggedFunction` (106, 4,860 = `8a9a9f7d`; `:65-104`, `:71-85`, `:95-102`), `InMemoryKeyColumnValueStore` (471, 19,258 = `83650bf4`; `:85-158`, `:94`, `:99`, `:109-114`, `:117-124`, `:127-129`, `:131-134`, `:137-144`, `:146-149`). **Read inline, member-name citations only:** `InMemoryColumnValueStore` (`5569f1a0`), `DistributedStoreManager` (`89ed24d5`), `CQLSliceQueryUtil` (`2efd226b`). **Seen only as `search_code` fragments:** `SliceQuery`, `EdgeSerializer`, `CQLSliceQueryUtilTest`. Three directory listings with `fields: ["name","sha","size"]`; one `search_code` query, returning three results; two delegated transcription batches, nine files, all blob-SHA verified. **Two external pages fetched:** the Cassandra Dynamo architecture page and the PlanetScale failover post. Five PR candidates logged (**74-78**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md). Daily Diff: **the 2026-11-25 edition had not published at 03:33 UTC**, and the run discovered that **`https://tdd.cat/{date}/` works where `{date}.md` 404s** — so the digest covered the latest published and previously uncovered **2026-11-24** edition (59 items, read in full across three offset fetches), with a deep dive on **PlanetScale Neki's leaseless unplanned failover for sharded Postgres**.

  **Findings.** **THE finding: the CQL backend's entire write-ordering guarantee is one bit of the commit timestamp, and the in-memory store reaches the same semantics by the exactly opposite route.** Take the commonest interesting mutation — a transaction that deletes column `c` and re-adds `c` with a new value, which is every single-cardinality property update. `InMemoryColumnValueStore.mutate` resolves it **structurally and client-side**: `synchronized`, sort the additions, then drop every deletion found among them by `binarySearch`, so the tombstone never reaches the buffer at all. `CQLKeyColumnValueStore.mutate` (`:673-676`) resolves **nothing** — it wraps its arguments and delegates — and the resolution happens in Cassandra, because `MaskedTimestamp.getDeletionTime` is `times.getTime(t) & 0xFFFFFFFFFFFFFFFEL` and `getAdditionTime` is the same with `| 1L`. **The addition time is the deletion time plus exactly one, by bit arithmetic**: the least significant bit of the provider's resolution is reserved as a delete/add discriminator, and per-column last-write-wins does the rest. That is also the answer to lesson 26's handover question — **a cell write at a fixed `(key, column, timestamp)` is an assignment, so replaying it is free; a JSON array append is an accumulation, so it is not.** The dividing line between the two write paths is not distributed-versus-local, it is assignment-versus-accumulation, and `storage.cql.idempotent-writes` (default `true`) is the one-line consequence. **Why the mechanism has to be temporal rather than structural: a CQL mutation is not one request.** The default unlogged path does `Iterator.concat(deletions, additions).grouped(20)` and fires each group as its own async `UNLOGGED` batch, so a key with more than 20 column operations is sent as several concurrent requests whose arrival order is unspecified — and the result is still deterministic. **Three old debts discharged.** Lesson 18's: `fb.locking(useExternalLocking)` with the option defaulting `false`, so `ConsistentKeyLocker` **does** run on Cassandra — and setting it `true` makes `acquireLock` a method whose body is only its inverted guard, so the option **disables locking rather than delegating it**, which the option's own description admits in the words "read uncommitted" and "phantom reads". Lesson 16's: batch ordering is **implicit in the timestamps, not explicit in the batch**, and the `LOGGED` alternative is one unbounded batch across every table and key. Lesson 20's `setDirectColumnByStartOnlyAllowed`: its only consumer in the codebase is `CQLSliceQueryUtil`, whose grouping rule `groupSize = min(slice-limit, limit)` means **a limit-1 direct-equality query forms a group of one, so `IN`-grouping cannot fire for it** — sound, because CQL's `LIMIT` is per result set rather than per `IN` element. **Fourth, the module's character is graceful degradation**: five places — token-range scans, `PER PARTITION LIMIT`, parallel scan, Cassandra-5 compression option names, and `cdc=true` on a pre-existing table — where an unsupported backend produces one `warn` at store open and a quietly different store for the rest of its life, on the stated principle that a scan-only optimisation must not fail store open and with it every graph open. Reading those five comments is the fastest way to understand what "CQL-compatible" costs. **Fifth, the most promising defect-shaped thing in the module is a reachability question, not a finding:** with `graph.assign-timestamp=false` the statements carry no `USING TIMESTAMP` at all, and the unlogged path's `.grouped(20)` can then put a delete and its matching insert into different concurrent batches — at which point the whole correctness argument above stops holding. That is candidate 74, and it is deliberately **not** stated as a confirmed defect.

- **26 — 2026-11-24** · [`2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md`](2026/11/2026-11-24-janusgraph-index-provider-elasticsearch/README.md) · **Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the mixed-index SPI and its Elasticsearch implementation at the pinned commit. **Blob-SHA verified and cited by line number:** `ElasticSearchIndex` (2,028 lines, 119,295 bytes = `d80949e3`; cited at `:120`, `:185-188`, `:215-222`, `:224-232`, `:234-250`, `:252-266`, `:526-536`, `:543-562`, `:564-566`, `:568-576`, `:770-800`, `:802-813`, `:815-824`, `:826-835`, `:837-844`, `:846-854`, `:855-876`, `:866-868`, `:869-873`, `:874-875`, `:881-888`, `:897-916`, `:932-958`, `:1010-1047`, `:1015-1017`, `:1018-1023`, `:1025-1038`, `:1039-1046`, `:1162-1167`, `:1169-1171`, `:1173-1217`, `:1179-1192`, `:1197-1201`, `:1219-1238`, `:1275-1381`, `:1292-1295`, `:1300-1306`, `:1307-1350`, `:1314-1315`, `:1319`, `:1322`, `:1326-1330`, `:1336`, `:1344`, `:1346`, `:1352-1356`, `:1364-1368`, `:1369-1380`, `:1371-1372`, `:1383-1406`, `:1408-1422`, `:1424-1462`, `:1436-1441`, `:1451-1452`, `:1457-1458`, `:1459-1461`, `:1464-1468`, `:1507-1708`, `:1709-1728`, `:1718`, `:1730-1735`, `:1794-1804`, `:1806-1825`, `:1816`, `:1827-1847`, `:1854-1856`, `:1864-1870`, `:1872-1881`, `:1883-1925`, `:1928-1942`, `:1944-1956`, `:1959-1962`, `:1964-1967`, `:1995-2007`). **Read inline, member-name citations only:** `IndexProvider` (`b7b342bb`), `IndexFeatures` (`6651e246`), `IndexInformation` (`ded18cc9`), `IndexMutation` (`d2c44e46`), `IndexSelectionUtil` (`3ec0ec2a`), `AbstractESCompat` (`89543cd9`), and all four compat subclasses read in full — `ES6Compat` (`d686ce02`), `ES7Compat` (`fb0efe32`), `ES8Compat` (`bbd2b413`), `ES9Compat` (`73af9fc2`). **Seen only as `search_code` fragments:** `AdjacentVertexHasUniquePropertyOptimizerStrategy`, `AbstractIndexSelectionStrategy`, `BruteForceIndexSelectionStrategy`, `ApproximateIndexSelectionStrategy`, `IndexSerializer`, `HasStepFolder`, `IndexSelectionUtilTest`. Three directory listings with `fields: ["name","sha","size"]`; two `search_code` queries, both returning results. Three PR candidates logged (**71-73**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same push as this file — **the register was brought to local disk by a delegated transcription and blob-SHA verified (18,388 bytes = `642bd570`) before being appended to**, the first time that method was used on a file being edited rather than read. Daily Diff: **today's edition had not published** (`2026-11-24.md` and `2026-11-23.md` both client errors), so the **fallback pool was drawn on for the first time** — the uncovered **2026-11-20** edition, 50 items, read in full across two fetches; deep dive on **Valkey 9.2's forkless snapshots**.

  **Findings.** **THE finding: the index SPI is the only write path in JanusGraph that is not idempotent by construction, and `LIST` cardinality is the entire reason.** Audit the five mutation primitives against the Painless scripts and exactly two are non-idempotent: `ctx._source[field].add(value)` for a `LIST` field, which appends a duplicate on a second application, and `remove(indexOf(value))`, which removes another occurrence or none. `SET` is saved by one guard — `field.cardinality != 'SET' || indexOf(value) == -1` — and every `SINGLE` operation is naturally idempotent (`ctx._source.remove(name)` on an absent key, and `putAll(params.doc)` rewriting the same value). **The fix reached up into the core rather than down into the backend**: `IndexMutation` gained a `completeDocument` field carrying the element's whole indexed state, *"supplied once, from the element's state at commit"*, with the design note that *"reading the element again would read the same state"* — so it costs no read. `addUpdatesCarryingTheCompleteDocument` (`:855-876`) then collapses every change to one document into **one** update whose upsert is the complete document, exploiting three mechanisms at once: Elasticsearch **writes the upsert and skips the script** when the document is missing, so "recreate whole" and "apply the delta" become the same request; one item per document is the **maximum atomicity** the bulk API offers; and a retried item therefore never replays a *part* that had applied. The in-source comment at `:846-854` is the clearest statement of the problem in the codebase. Second: **`mutate` has three request shapes and the non-idempotent one is still live.** Shape A is a whole-document delete, guarded on `hasDeletions() && isDeleted()`; shape B is the single complete-document update; shape C — reached whenever `completeDocument` is `null` for an existing document — emits up to **three** separate bulk items for one `_id` (field-deletion script `:1314-1315`, addition script `:1336`, `{"doc": …}` merge `:1344`), with `upsert` deliberately `null` when deletions are present (`:1326-1330`). That is exactly the shape shape B's own javadoc argues against (candidate **73**, reachability untraced: `IndexSerializer` and `IndexTransaction` were not read). Third: **the retry machinery and the idempotence machinery are a matched pair.** `retainUnappliedDocuments` (`:770-800`) **mutates the caller's map in place** — a contract declared in `IndexProvider.mutate`'s own javadoc, *"a caller must not rely on the map being unchanged after a failure"* — in four independent steps, each wrapped by `narrowQuietly` (`:805-813`) so that an immutable map simply resends that store whole, and guarded by `namesPendingDocuments` (`:826-835`) so that a response naming documents the mutation no longer holds narrows nothing. Every step fails **toward** resending more, which is only sound because shape B makes the duplicate work correct. The residual hazard the lesson does **not** claim either way: shape B guarantees all-or-nothing within a document, not script idempotence against a document that already holds the outcome, so a bulk item that applied and was then reported failed still double-applies a `LIST` add. Fourth: **`register` is a non-transactional side effect of a transaction.** Called synchronously from `ManagementSystem.addIndexKey:802` (lesson 25), it can `checkForOrCreateIndex` **and** `pushMapping` before the management transaction commits, and nothing compensates in either direction — a failed register can leave a stray ES index, and a successful register followed by a rollback leaves ES holding a mapping the schema has never heard of. ES mappings being additive and practically irremovable, the drift accumulates one way, with no documented repair. Fifth, **debt (d) is settled and the answer is "no filter at all"**: `IndexSelectionUtil` has six public static methods and **not one reference to `SchemaStatus`**; `existsMatchingIndex`/`getMatchingIndexes` apply only the caller's `Predicate<IndexType>`, and `AdjacentVertexHasUniquePropertyOptimizerStrategy` passes `indexType -> indexType.isCompositeIndex() && …`. So a unique composite index at `INSTALLED`, `REGISTERED` or `DISABLED` still drives the rewrite although the composite read gate is `== ENABLED`. **The absence is verified; the consequence is not** (candidate **72**, Hands-On B). Sixth, a **correction to the handover**: there is no `supportsExistsQuery()` in `IndexFeatures` at the pinned commit — the file was read in full and the nearest member is `supportsGeoExists()`. Seventh: **all four `*Compat` classes produce identical `IndexFeatures`**, each being `coreFeatures().setWildcardField(CUSTOM_ALL_FIELD).supportsGeoContains().build()`, so `coreFeatures()`'s `setWildcardField("_all")` is **unreachable** (candidate **71**) and version dependence lives in a boolean field, an `IllegalArgumentException` and an `unmapped_type` comment instead. `supportsDocumentTTL` is `false` for every ES version, leaving `IndexMutation.determineTTL()`'s uniform-TTL `checkArgument` with no ES consumer. Eighth, inherited item (b) settled: **Elasticsearch has no page-doubling equivalent of `SolrIndex`'s** — it sizes pages once, with three options (`limited-result-max-page-size` **250**, `adaptive-point-in-time-max-page-size` **500**, `adaptive-point-in-time-max-shards` **1**), all three carrying `positiveInt()` verifiers where lesson 24's candidate 66 found none on the core batching options. The doubling that does affect an ES-backed query is `LimitAdjustingIterator`'s, one layer up.

- **25 — 2026-11-23** · [`2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md`](2026/11/2026-11-23-janusgraph-management-index-lifecycle/README.md) · **Part VI opened. Mid-Part, so the re-pin question was correctly NOT re-raised.** Read the management and index-lifecycle surface at the pinned commit. **Blob-SHA verified against the directory listing and cited by line number:** `ManagementSystem` (1,815 lines, 86,864 bytes = `a722a7d9`; cited at `:190-207`, `:209-213`, `:241-268`, `:253`, `:256-257`, `:340-393`, `:365`, `:367`, `:368`, `:387`, `:389-391`, `:622-639`, `:628-639`, `:655-679`, `:704-706`, `:720-724`, `:746-767`, `:758`, `:770-808`, `:785-791`, `:791`, `:802`, `:807`, `:831-879`, `:841-846`, `:852`, `:860`, `:877`, `:967`, `:977`, `:980-1140`, `:987-996`, `:1000-1019`, `:1013-1016`, `:1026-1032`, `:1033-1051`, `:1052-1056`, `:1100-1112`, `:1130-1135`, `:1142-1147`, `:1154-1166`, `:1207-1301`, `:1227-1287`, `:1237-1242`, `:1244-1254`, `:1289-1299`, `:1303-1305`, `:1310-1319`, `:1321-1326`, `:1328-1341`, `:1334`, `:1343-1361`, `:1351`, `:1392-1459`, `:1436-1458`) and `ManagementLogger` (505 lines, 24,707 bytes = `1980d1fb`; `:71`, `:169-230`, `:176-207`, `:208-224`, `:235-237`, `:239-260`, `:275-286`, `:288-296`, `:306-315`, `:317-326`, `:328-339`, `:341-357`, `:382-441`). **Read inline, member-name citations only:** `SchemaStatus` (`7b40611d`), `SchemaAction` (`b2bf7951`), `JanusGraphIndexWrapper` (`94d0458d`), `GraphIndexStatusWatcher` (`3ebe2f6a`), `AbstractIndexStatusWatcher` (`e0ab9913`), `IndexUpdateJob` (`f1b9b521`), `IndexRepairJob` (`434abee9`). Two directory listings, one with `fields: ["name","sha","size"]` — **the cheap SHA-verification method this run introduced**. One `search_code` query, returning nothing. Three PR candidates logged (**68-70**) in [`tracks/janusgraph-registers-2.md`](tracks/janusgraph-registers-2.md), written in the same run as this file, as the second of two consecutive pushes. Daily Diff: the **2026-11-22** edition (38 items, **read in full**), deep dive on **Apache Kvrocks**, Redis mapped onto RocksDB, with a sidebar on **ParadeDB**'s filter-aware vector search.

  **Findings.** Paragraph compressed 2026-11-26 under the roll-off rule, and the `Verified by lesson 25`
  block removed in the same run; the full versions are in git history at the 2026-11-23 through 2026-11-25
  commits, and the facts lesson 28 was promised are carried in *Notes For The Next Run* below and were all
  used. The one-line substance: **most indexes never enter the INSTALLED → REGISTERED → ENABLED lifecycle
  at all, and one expression decides it** — `canIndexBeEnabled`, which writes `ENABLED` directly when any
  one key or the `indexOnly` constraint is new in the same management transaction, because the backfill is
  then provably empty; **"the index's status" is two physically different things**, one `SchemaStatus`
  property on the index vertex for a composite or relation-type index versus **one status per field**, held
  as the last slot of each `INDEX_FIELD` edge's `Parameter[]` behind an `assert`, so a mixed index is N
  lifecycles rather than one; and **the INSTALLED → REGISTERED transition is an unpersisted callback in the
  issuing JVM's heap**, so what `REGISTERED` guarantees is not "every instance has seen the change" but
  "every instance has seen it and every transaction that predated it has finished" — which is what makes a
  reindex safe, and what strands an index at `INSTALLED` with no coded recovery path if that JVM dies
  mid-handshake. Three PR candidates, **68-70**.

- **24 — 2026-11-22** · [`2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md`](2026/11/2026-11-22-janusgraph-step-multiquery-batching/README.md) · **Findings.** Paragraph compressed 2026-11-25 under the roll-off rule, and the `Verified by lesson 24` block removed in the same run; the full versions are in git history at the 2026-11-22 through 2026-11-24 commits, and the facts lesson 28 and the capstone were promised are carried in *Notes For The Next Run* below. The one-line substance: **the batch size of a `JanusGraphVertexStep` is written by two different strategies and on the shipped default the second write always wins** — the field defaults to `Integer.MAX_VALUE`, `JanusGraphLocalQueryOptimizerStrategy`'s pass 5 writes `cache.tx-cache-size`, and `JanusGraphMultiQueryStrategy` then overwrites it with 2500 because `query.batch.limited` defaults to `true`, so **`cache.tx-cache-size` has no effect on batching on a default graph**; the two steps are not the matched pair their names suggest, only one compiling a query inside itself and only the other batching; and a batch is always a *hint* rather than a correctness constraint. This run also performed the **Part V/VI re-pin review**, which the pin survived for the fifth time. Three PR candidates, **65-67**.

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
- [x] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**THE SYLLABUS IS COMPLETE. The next run writes the CAPSTONE, not a lesson** — this is Case C of the task:
a `YYYY-MM-DD-janusgraph-capstone/README.md` assembling Parts I-VI end to end, one diagram covering every
layer, the cross-cutting tradeoffs, and a project-scale exercise. Record JanusGraph under
`## Completed Subjects` with its date range and item count (28 items, 2026-10-30 to 2026-11-26), and then
**resume the paused PostgreSQL track at lesson 11** per `tracks/postgresql.md` — do **not** start a new
subject, and do not pick a fresh database. The paused-track instructions are the authority on how.

**Syllabus unchanged for the whole of Part VI. No item was added, split, merged or reordered at any point;
the 28 stand exactly as written on 2026-10-30.** Lesson 28 was the first unchecked item and is now checked.

**Re-pin: PERFORMED this run and the pin stands for the sixth time.** `list_commits` on `master` returned
`a59ae8d8` (2026-10-28) as the newest commit — the same three newest commits as at lessons 13, 18, 22 and
24, so the repository has not moved in 29 days. **No further review is due**: the capstone cites files the
track already read at this pin. If the capstone reads anything new, verify it by blob SHA against a
directory listing before citing a line number, exactly as every lesson from 22 on has.

**The one sentence the capstone should build its durability section around.** JanusGraph's write-ahead log
does not make a commit atomic; it **borrows** the storage backend's atomicity for one log entry and then
uses that entry as evidence that the storage write landed. Everything else follows: the log carries element
ids rather than document content, so repair is a **re-derivation from storage** and is therefore idempotent,
convergent, order-free and robust to a wrong log entry; composite indexes need no log because they commit
with storage; mixed indexes need one because they commit after it; and the user log is the one consumer for
which re-derivation is impossible, so it gets a **re-send** and an unstated idempotence requirement. On
default CQL the borrowed atomicity is not there at all — different store, different partition, several
concurrent unlogged batches — and the dangerous direction is **losing the log entry while keeping the
storage write**, which `PRECOMMIT`-only handling counts as a success.

**Carried facts the capstone may cite without re-deriving** (full detail in the `Verified by lesson 26/27/28`
blocks below, which this run deliberately did NOT roll off):

- **Defaults that bound everything.** `tx.log-tx` **false** (GLOBAL), `tx.max-commit-time` **300 s**
  (GLOBAL), `storage.write-time` **100 s**, `TRANSACTION_LOG_DEFAULT_TTL` **7 days**, `log.num-buckets`
  **1**, `log.read-threads` **1**, `log.send-delay` **1000 ms**, `graph.assign-timestamp` **true** but
  `LOCAL`, `storage.transactions` **true**, `storage.cql.atomic-batch-mutate` **false**,
  `storage.cql.batch-statement-size` **20**, `storage.cql.use-external-locking` **false**,
  `query.batch.limited` **true** with batch size **2500**.
- **The arithmetic result worth putting in the capstone verbatim:** `max-commit-time` must *exceed*
  `write-time × (1 + indexBackends)`, so the shipped 300 s/100 s pair is safe for **at most one** index
  backend and warns at two.
- **The three-way idempotence taxonomy the track converged on**, which is the best cross-cutting theme it
  produced: a **cell write** at a fixed `(key, column, timestamp)` is an assignment and replays for free
  (lesson 27); a **mixed-index document** is rewritten whole from storage and so is an assignment too, by
  construction (lessons 26 and 28); a **`LIST` field script append** and a **user-log event** are
  accumulations and deliveries, and replay is unsafe for both. The dividing line is never
  distributed-versus-local.

**The capstone's open-items ledger. Nineteen Hands-On experiments are now outstanding** — lesson 24's three,
25's four, 26's four, 27's four and 28's four — and **not one run in this track has ever executed
JanusGraph**, so every finding is read rather than reproduced. State that plainly in the capstone; it is the
single largest qualification on the whole track. Cheapest first:

1. **Lesson 28's Hands-On B** — four lines of `jshell`, no build: proves candidate 80's precondition is dead.
2. **Lesson 27's Hands-On A** — two minutes in `jshell`: reproduces the LSB delete/add mask.
3. **Lesson 26's Hands-On B** — `inmemory`, one unique composite index, `DISABLE_INDEX`, re-profile; settles candidate 72.
4. **Lesson 28's Hands-On C** — one unit test in `janusgraph-core` against the package-private `nanosUntilExpiry`.
5. Everything else needs Docker.

**Filing gates still in force:** candidate 65 not until lesson 24's Hands-On B; 70 not until 25's Hands-On D;
72 not until 26's Hands-On B; 74 is now a *configuration-drift* claim and needs the two-instance case
demonstrated; **79 must not be filed until `BackendTransaction.commitIndexes` has been read**, because its
whole weight rests on whether that method can throw. **80 is the only candidate in the register that is
provable by reading alone and safe to file today**, and it is a two-line diff.

**Three reads the capstone should spend its budget on, in order.** (a)
`BackendTransaction.commitIndexes` — gates candidate 79 and completes the commit path Parts III and VI both
depend on. (b) `IndexTransaction.restore` — the single primitive **both** repair paths funnel into
(`StandardTransactionLogProcessor:428` and `MixedIndexStaleEntryRemover:178`) and never read; it also decides
whether `batchSize` is a memory knob or a request-size knob. (c) one `search_code` for
`MIXED_INDEX_REINDEX_BATCH_SIZE`, which closes lesson 26's metrics debt and names the caller that passes
`batchSize` into `MixedIndexStaleEntryRemover.submit`.

**Open and unresolved across the whole track, deliberately not filled in.** (a) Whether any production path
passes a real limit to `QueryUtil.processIntersectingRetrievals` (candidate 51 is a reachability claim).
(b) `BackendQueryHolder`'s duplicated `isFitted` in `hashCode` (candidate 54). (c)
`IndexCandidateGroup.getTotalScore()`'s `Double.NEGATIVE_INFINITY` sentinel colliding with
`IndexCandidate`'s default score (inside candidate 50). (d) Lesson 23's §2.2 strategy-ordering question.
(e) Whether `BatchProcessingQueue.removeLast()` has any caller — `HasStepBatchFetcher` and
`DropStepBatchFetcher` unread, so it must **not** be called dead code. (f) Whether
`ElasticSearchBulkFailureException` can report an item as failed after it applied, which decides whether
lesson 26's shape B has a residual duplication hazard; `TransientFailures` (`6aba13b8`) and
`ElasticSearchBulkFailureException` (`e52ae410`) unread. (g) Which core call sites invoke
`IndexMutation.setCompleteDocument`, i.e. whether shape C is reachable for a real update. (h)
`BasicVertexCentricQueryBuilder:711`'s `Preconditions.checkArgument(bestCandidate != null, …)` reachability —
lesson 20's other debt, unowned since lesson 27; **the capstone should settle it or record it as open.** (i)
**New from lesson 28:** lesson 24's `JanusGraphStep:118` order assert is still unresolved and a stale mixed
index is precisely its trigger condition, which makes it the one open item with a demonstrated route to
wrong *results* rather than wrong *metrics*.

**Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his to fill
at any time; if a future run finds content in any of them, answer those questions in that run's entry
regardless of mode.

**Entry size.** Lesson 05 ~35 KB, 06 ~63, 07 ~78, 08 ~67, 09 ~61, 10 ~70, 11 ~56, 12 ~91, 13 ~72, 14 ~75,
15 ~86, 16 ~87, 17 ~64, 18 ~78, 19 ~69, 20 ~76, 21 ~103, 22 ~77, 23 ~59, 24 ~59, 25 ~51, 26 ~62, 27 ~70,
**28 ~76** (digest ~17). A large share of each is permalink URLs, which are not prose.

**Mechanical notes, carried and confirmed again this run. Read this section before planning a run's pushes.**
`get_file_contents` on a file whose doubled output exceeds roughly 40 KB returns an over-limit error that
saves the JSON to a local file — the cheap way to read a large file, costing nothing in context. The correct
extraction is `json.loads(longest_text)["content"][-1]["resource"]["text"]`, **not** the plain `text` value,
which lesson 27 proved LOSSY rather than merely banner-prefixed. This run used the resource path for five
files (`StandardTransactionLogProcessor`, `GraphDatabaseConfiguration`, `StandardJanusGraph`, this file and
the root `README.md`) and **every blob SHA matched the API's reported value on the first attempt** — the
method is now five-for-five across two runs. A directory listing with `fields: ["name","sha","size"]` gives
every file's blob SHA in one cheap call and remains the preferred verification source. A file under roughly
20 KB comes back inline with no line numbers; such a file must be written to local disk to be cited by line
number, and **a lesson that does not do that must cite it by member name only**. This run delegated one
transcription batch of six files, all six blob-SHA verified on the first attempt.

**The rule on sub-agents held this run.** Lesson 27's first transcription batch silently fell back to
`raw.githubusercontent.com`. This run stated the prohibition explicitly in the prompt — naming `curl`,
`wget`, `gh`, `git` and `raw.githubusercontent.com`, and instructing the sub-agent to report failure rather
than find another route — and it was obeyed and reported. **Keep doing that verbatim.**

**The real constraint on a run is OUTPUT, not input.** Every file pushed must be emitted in full as a tool
argument; there is no server-side patch. Reading a 134 KB file is nearly free; *rewriting* it costs ~37 K
output tokens. The method that works, used again this run: author only the documents the run writes, edit
the large pre-existing files **in place on local disk with small scripted replacements** rather than
re-emitting them, and **delegate their pushes** to sub-agents that read the finished bytes from disk — then
verify every pushed blob SHA against the locally computed one. A silent truncation in a delegated push is
the one failure mode this method introduces, so the verification is not optional. **This file shrank from
99 KB to ~95 KB this run** by rolling off lesson 25's block and paragraph and lesson 23's one-liner while
adding lesson 28's, so the ~110 KB structural-split threshold was **not** crossed. The capstone is the last
run of this track and should not split anything; the split question dies with the track, and
`tracks/janusgraph-verified.md` was never needed. The root `README.md`'s index is one row per line — **edit
the row in place rather than rebuilding the table**; lessons 26, 27 and 28 all did.

**Mermaid.** Render every diagram with `mmdc` before pushing, and then **verify the rendered SVG** — grep it
for a literal backslash-n (must be zero) and inspect the node labels, because a diagram that renders is not
the same as a diagram that renders correctly. Lesson 28's three (two in the lesson, one in the digest) all
rendered first time, with `\n`-free output, `<br/>` becoming `<br />`, and `&lt;` surviving as an entity so
that `<` displays. Accumulated rules: quote every node label containing parentheses, commas, `->`, `|` or
angle brackets; use HTML entities such as `&lt;` and `&amp;` for comparisons and ampersands; never escape a
quote inside a label — drop the inner quotes instead; **a semicolon inside a `sequenceDiagram` message is a
statement separator** — use a comma; a `sequenceDiagram` tolerates `--x`; and inside a `sequenceDiagram`
message prefer the words "and"/"or" over `&`/`|` entities. New this run: an **edge label** in a
`flowchart` takes a quoted string with `<br/>` in it and renders multi-line correctly, which is the cheapest
way to put a condition on a transition without inventing a node.

**Daily Diff procedure — the slash form is UNRELIABLE, not reliable. Rewrite the expectation, not the steps.**
Lesson 27 found `https://tdd.cat/{YYYY-MM-DD}/` served the complete 11-24 edition where `{date}.md` 404s, and
concluded the slash form was the answer. **This run it failed for both 2026-11-26 and 2026-11-25** in the
same session, while the front page loaded. So: **(a)** fetch `https://tdd.cat/{today}/`; **(b)** on a client
error — now the common case — fetch the front page `https://tdd.cat/`, whose header names the current
edition's date and story count and is the **only endpoint that has loaded every time**; **(c)** try that date
in slash form, and if it also errors, **build the digest from the front page itself**, saying plainly how
many of the stated items were recovered; **(d)** check this session log in both directions as a floor and a
ceiling; **(e)** only then the uncovered-item fallback. **Do not trust `/archive/` or `/md` for currency.**
The front page is truncated by the fetcher at ~40 K characters of ~90 K, so a front-page digest recovers
roughly the first 20-22 items of a 50-60 item edition — **say so rather than implying coverage**. Editions
covered so far, by date and size: 11-08 to 11-22 at 22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61, 61, 50, 47,
38; **11-24 at 59** (lesson 27); **11-25 at 53, 22 items recovered** (this run). **The fallback pool still
holds the unspent items from 2026-11-06 (two) and 2026-11-18 (two)**, untouched again. **Constraint still in
force: there is no `WebFetch` tool in this environment** — pages are fetched through a sub-agent, which flags
which parts are verbatim and which are an extracted summary, and the summary portion is not reliable enough
to quote. **Always ask the sub-agent for the structured item list — title, source, outbound URL, one-line
gist — rather than for the page text**, which a non-preapproved source makes it cap at ~125 characters
anyway, and always cross-check the item count against the page's own stated figure.

### Verified by lesson 27, for the capstone to cite rather than re-derive

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

### Verified by lesson 26, for the capstone to cite rather than re-derive

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
