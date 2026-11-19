<!--
entry-meta
date: 2026-11-19
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-11-19
slug: janusgraph-graph-centric-query-builder
-->

# Daily Diff — 2026-11-19

**2026-11-19 · Daily Diff Digest**

**Edition covered: 2026-11-18, 61 stories.** At ~03:40 UTC the front page served **2026-11-18 / 61 stories**, `/archive/` named **2026-11-18** as the newest and listed every day from 2026-11-01 to 2026-11-18, and `https://tdd.cat/2026-11-19/` returned a `client_error` — today's edition has not published yet. The two indexes **agreed** this run, ending the three-run streak of the front page running ahead; the 2026-11-18 edition was the one uncovered edition (lesson 20 covered 2026-11-17). Edition sizes across 11-08 to 11-18: **22, 50, 63, 58, 66, 47, 39, 49, 37, 57, 61**.

## The notable items

**Databases and storage**

- **`CREATE TEMP TABLE` in a hot path costs ~23× throughput, and the bill is catalog churn plus a shared invalidation ring** — [postgres.ai](https://postgres.ai/blog/20261118-hidden-costs-of-create-temp-table). Measured on Postgres 18.6. Today's deep dive.
- **Native vector search over open lakehouse tables** — [Onehouse "Lakegres"](https://www.onehouse.ai/blog/lakegres-vector-search-on-open-table-formats) indexes Iceberg/Hudi tables directly rather than copying vectors into a separate store. The interesting claim is index *placement*, not recall.
- **Higher traffic *reduces* Postgres read-replica replication lag** — [ClickHouse](https://clickhouse.com/blog/tuning-postgres-read-replicas). Counter-intuitive and worth the read: the startup process replays faster when its buffers are already warm from concurrent read traffic, so an idle replica is a slow replica.
- **Re-evaluating custom Galera node weights** — [Percona](https://www.percona.com/blog/just-because-you-can-doesnt-mean-you-should-re-evaluating-galera-node-weights/), on quorum weights that look like tuning and behave like a split-brain generator.
- **Routing analytical queries away from the OLTP path** — [ProxySQL on MariaDB HTAP](https://proxysql.com/blog/mariadb-htap-measured-again/).
- **Indexed codebases for code retrieval at lower cost** — [Voyage "Wayfinder"](https://blog.voyageai.com/2026/11/18/wayfinder-code-search/), and [embedded-DuckDB single-binary observability](https://github.com/labstack/pitot). Both are "put the index next to the data" arguments.

**Security, and two supply-chain compromises in one edition**

- **NGINX HTTP/2 remote code execution** — [Wiz](https://www.wiz.io/blog/nginx-http2-rce-vulnerabilities) and [xint.io](https://xint.io/blog/nginx2root-remote-code-execution). Unauthenticated RCE in the HTTP/2 handling; if you run nginx anywhere, this is the item of the day for you rather than the Postgres one.
- **Microsoft's `agent-framework` PyPI releases compromised to steal credentials** — [SafeDep](https://safedep.io/blog/microsoft-agent-framework-pypi-compromise) and [opensourcemalware.com](https://opensourcemalware.com/blog/agent-framework-pypi-compromised). A legitimate, widely-installed package, not a typosquat.
- **One in four public MCP servers can perform irreversible actions** — [Rinwall's scan](https://github.com/RinwallDev/rinwall/blob/master/research/2026-10-public-mcp-scan.md). Relevant to anyone wiring agents to tools.
- **GPU load-store unit contention as a side channel** — [arXiv 2611.23022](https://arxiv.org/abs/2611.23022).
- **Passing upstream tokens through to MCP servers breaks trust boundaries** — [Permit.io](https://www.permit.io/blog/oauth-on-mcp).

**Systems and performance**

- **Content-addressable storage for very large Git repositories** — [Google's ABFS](https://developers.googleblog.com/en/accelerating-development-for-massive-git-repositories/). Worth pairing with [live-syncing Jujutsu repos across machines](https://www.serpentine.com/posts/2026/announcing-jj-musubi/).
- **36M lines of code counted in 67 ms** — [Mezura on the Linux kernel](https://petros-papatheodorou.dev/writing/mezura/counting-the-linux-kernel/); the write-up is mostly about I/O strategy and SIMD line counting.
- **100 Gbit/s networking, in practice** — [Pixeldrain](https://docs.pixeldrain.com/posts/2024-03-07_network_optimizations/), and [the Firezone macOS client past 5 Gbit/s](https://www.firezone.dev/blog/apple-client-performance).
- **Rate-distortion optimisation making ASTC textures LZ-compressible** — [Jon Olick](https://www.jonolick.com/home/lz-rdo-lands-in-arms-astc-encoder). A clean example of designing an encoder for the *next* stage of the pipeline.
- **Object storage presented as a distributed POSIX filesystem** — [FiberFS](https://fiberfs.io/).
- **Orchestrated pytest workers for CI** — [Close](https://making.close.com/posts/faster-ci-with-orchestrated-pytest-workers/).

**Agents and models** — a third of the edition, and the honest items are the negative results: [coding agents fail to deliver complete organizational tasks](https://arxiv.org/abs/2611.20218), [poor coordination undermines the benefits of scaling agent teams](https://benchmarks.bespokelabs.ai/multi-agent-coordination-bench/), [passing an eval does not make an agent production-ready](https://deepsense.ai/blog/your-agent-passed-the-eval-should-you-ship-it/), and [in-page agent tools accidentally bypassing UI safeguards](https://giovannifilardo.com/posts/webmcp-agent-shortcuts-and-unintentional-dark-patterns/). Also [Gemini 4 Neon's million-token context](https://aistudio.google.com/docs/models/gemini-4-neon), [Apple FastFrames](https://huggingface.co/apple/FastFrames-L), and [data contracts as datasets-as-services](https://martinfowler.com/articles/data-contracts.html).

---

## Deep dive — `CREATE TEMP TABLE` is three costs, and the third one is global

Source followed: [postgres.ai, *How `CREATE TEMP TABLE` costs Postgres: catalog churn and the invalidation queue*](https://postgres.ai/blog/20261118-hidden-costs-of-create-temp-table), measured on **Postgres 18.6**.

### 1. The throughput numbers, 32 clients, no idle sessions

| Pattern | TPS |
|---|---|
| CTE, no temp table at all | **86,997** |
| Temp table kept per session, `ON COMMIT DELETE ROWS` | 17,866 |
| Created once, `TRUNCATE` per transaction | 4,405 |
| `CREATE … ON COMMIT DROP` per transaction | 3,783 |
| Created once, `DELETE` per transaction | 1,514 |

Two separable steps: **86,997 → 17,866** is the cost of using a temp table at all (three `ACCESS EXCLUSIVE` locks at each commit, WAL-logged under `wal_level = replica`, forcing an XID, a commit record and a flush), and **17,866 → 3,783** (~4.7×) is the catalog and file work on top.

### 2. Where the catalog work lands

One create-and-drop of `(id int, v text)` **inserts and deletes 31 catalog rows** and creates and removes **3 files** — the table, its TOAST table, and the TOAST index, because a `text` column can exceed a page. A bare `CREATE TEMP TABLE t (id int)` inserts **13 rows**, across `pg_class`, `pg_attribute`, `pg_type`, `pg_index` and `pg_depend`.

The file work is the part with no wait event: **~64% of the temp-table backends' time was in the kernel**, in path lookup and `unlink()`, which shows up only as `CPU*` in active session history. Moving `temp_tablespaces` to tmpfs bought **+27% TPS**.

On bloat, the article **corrects the common premise**: autovacuum *does* process system catalogs, so dead catalog rows are removed. What it cannot do is shrink the file — `VACUUM` truncates only empty pages at the *end*, so `pg_attribute` grew from **737 MiB to 839 MiB in ~70 seconds at 128 clients** with autovacuum on, and getting that back needs `VACUUM FULL` on a catalog, with the lock that implies. Separately, autovacuum **cannot touch temporary tables at all** — only the owning session can — so a long-lived temp table emptied with `DELETE` accumulates dead tuples *and* holds back freezing until the session vacuums, drops it, or disconnects.

### 3. The third cost, and the one that reaches unrelated backends

Every backend caches catalog rows (catcache) and relation descriptors (relcache). Changing a catalog row means telling every other backend to discard its copy, through **shared invalidation (sinval)**:

- **One ring, 4096 slots, 16 bytes per message, instance-wide** — all databases.
- **Each temp-table transaction sends 132 messages**: 71 for the `CREATE`, 61 for the `DROP`. Postgres sends them even for temp tables, because it cannot cheaply distinguish them from permanent objects.
- Writers append in batches of up to 64 under `SInvalWriteLock`; contention shows as `LWLock:SInvalWrite`.
- **Every backend must read every message, idle ones included.** A slot is reusable only once all backends have read it, so a sleeping backend pins the tail of the ring.
- At **half full (2048 behind)** the writer signals the furthest-behind backend — the catch-up signal.
- A backend more than a full ring behind is **lapped**: its messages are gone, so it discards *all* catcache entries, relation descriptors and cached plans and reloads them. A **full cache reset**, and the next queries re-read the catalogs and replan.

Measured at a 500 TPS target with 16 DDL writers:

| Idle sessions | Temp-table TPS | `LWLock:SInvalWrite` share of writers' active time | Full cache resets/s |
|---|---|---|---|
| 0 | 502 | 1.3% | 0.1 |
| 1000 | 502 | 54% | 49 |
| 3000 | 274 | 86% | 194 |
| 5000 | 127 (249 in a second run) | 91% | 457 |

At 5000 idle sessions, **84.7% of all backends' wait samples** were `LWLock:SInvalWrite` in the second run — most of it from woken idle backends that `pg_stat_activity` still reports as `idle`. **Lookup clients that issued no DDL at all lost 27–29% of their TPS**, because the queue is instance-wide. The article's leading (explicitly unconfirmed) hypothesis for those waits is `FastPathTransferRelationLocks()`, which scans about `max_connections` backend slots before granting a strong lock.

Standbys pay too: at 2000 TPS for 60 s a replica fell **731 MiB** behind, **1065 MiB** with 3000 idle sessions, with the startup process spending **60–64%** of its CPU samples in `StandbyAcquireAccessExclusiveLock` replaying temp-table locks. Single run, one server.

```mermaid
flowchart TD
    subgraph writers["16 DDL writers"]
        W1["CREATE TEMP TABLE<br/>71 sinval messages"]
        W2["DROP / ON COMMIT DROP<br/>61 sinval messages"]
    end
    W1 --> CAT["catalog rows: pg_class, pg_attribute,<br/>pg_type, pg_index, pg_depend<br/>31 rows inserted + deleted per cycle"]
    W2 --> CAT
    CAT --> FILES["3 files created and unlinked<br/>table + TOAST table + TOAST index<br/>~64% of backend time in kernel path lookup and unlink"]
    W1 --> LOCK["SInvalWriteLock<br/>append in batches of up to 64"]
    W2 --> LOCK
    LOCK --> RING["shared invalidation ring<br/>4096 slots x 16 bytes, instance-wide<br/>132 messages per temp-table transaction"]
    RING --> R1["reader: DDL writer backend"]
    RING --> R2["reader: lookup client<br/>no DDL of its own, still -27 to -29% TPS"]
    RING --> R3["reader: 5000 idle backends<br/>reported as 'idle' in pg_stat_activity"]
    R3 --> TAIL["a sleeping backend pins the ring tail<br/>slot reusable only when ALL backends have read it"]
    TAIL --> HALF["half full = 2048 behind<br/>writer signals the furthest-behind backend"]
    HALF --> LAP["more than one ring behind = LAPPED<br/>messages lost"]
    LAP --> RESET["full cache reset: drop all catcache,<br/>relcache and cached plans, reload<br/>457 resets/s at 5000 idle sessions"]
    CAT --> WAL["ACCESS EXCLUSIVE locks WAL-logged<br/>at wal_level = replica"]
    WAL --> STBY["standby startup process<br/>60-64% CPU in StandbyAcquireAccessExclusiveLock<br/>731 MiB behind at 2000 TPS"]
```

### Why it matters, and what changes

The mechanism to take away is that **temp-table DDL is not local**. It looks like the most private thing a session can do — a table only it can see, in its own namespace — and it is in fact a write into a fixed-size, instance-wide broadcast ring that every other backend, in every other database, must drain. That inverts the usual mitigation instinct: the lever with the largest measured effect here is not "make the temp table cheaper", it is **"have fewer idle connections"**, which has nothing to do with temp tables at all. A connection pool sized for peak and left idle turns a moderate DDL rate into a cache-reset storm.

The article's own ordering of fixes is the right one:

1. **Measure before changing anything.** `LWLock:SInvalWrite` in ASH; idle backend count from `pg_stat_activity`; and the delta of `n_tup_del` for `pg_attribute` in `pg_stat_sys_tables` divided by **19** as an estimate of temp tables dropped per second (for the article's two-column table with TOAST).
2. **Get temp tables out of hot paths**: a CTE where the logic allows, PL/pgSQL variables or arrays plus `unnest()` where it does not, `insert … select * from unnest(…)` for bulk upserts.
3. **If one must stay**, `CREATE TEMP TABLE IF NOT EXISTS … ON COMMIT DELETE ROWS`, once per session — still 3 locks and 5 invalidation messages per commit, still requires a session-mode pooler, and in PgBouncer transaction mode it works only if one transaction creates, uses and drops it.
4. **Reduce idle backends**: pooler, smaller application pools, `idle_session_timeout` (14+).
5. **`VACUUM FULL` on the catalog** is the only way to reclaim the file space, with the lock.
6. **Upstream ideas listed**: size the ring from `max_connections`, decouple cleanup from idle backends, add wait events for file create/unlink, and global temporary tables — which would remove the per-session catalog rows entirely and so remove costs 2 and 3 at the source.

**Caveats the article states itself**: the standby result is a single run on one server, the `FastPathTransferRelationLocks()` attribution is a hypothesis, the 5000-idle row varies between runs (127 and 249 TPS), and catalog *index* bloat was not measured.

### The connection to today's lesson

Both halves of today's work are about **a planner or a cache that cannot see a cost that is real**. JanusGraph's index selector scores a thousand-value `IN` exactly like a two-value one because `k` is not in its scoring function; Postgres' temp-table path sends 132 invalidation messages per transaction because it cannot cheaply tell a private object from a shared one. In both cases the mechanism is correct and the accounting is missing, and in both cases the symptom lands on something that did not issue the query — a second index read in JanusGraph's case, an unrelated backend's cache reset in Postgres'.

---

## Sources

- [tdd.cat front page](https://tdd.cat/) — fetched this run; served the 2026-11-18 edition, 61 stories.
- [tdd.cat archive](https://tdd.cat/archive/) — fetched this run; newest edition 2026-11-18, November 2026 complete through the 18th.
- `https://tdd.cat/2026-11-19/` — probed this run, returned `client_error`; today's edition had not published at ~03:40 UTC.
- [postgres.ai — *How `CREATE TEMP TABLE` costs Postgres: catalog churn and the invalidation queue*](https://postgres.ai/blog/20261118-hidden-costs-of-create-temp-table) — fetched this run; the deep-dive source, all numbers above are from it.
- All other links in *The notable items* are the edition's own source URLs as listed on the fetched front page this run.
