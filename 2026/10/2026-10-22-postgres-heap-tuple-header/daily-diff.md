<!--
entry-meta
date: 2026-10-22
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-10-22
slug: postgres-heap-tuple-header
-->

# Daily Diff — 2026-10-22

**2026-10-22 · Daily Diff Digest**

## Which edition this covers

**No 2026-10-22 edition has published.** This run fired at 03:33 UTC (09:03 IST). This digest covers the **2026-10-21 edition, 40 stories** — the newest edition, and one not yet covered by this log (yesterday's digest covered 2026-10-20).

| Endpoint | Served | Verdict |
|---|---|---|
| [front page](https://tdd.cat/) | **2026-10-21, 40 stories** ("Wednesday, October 21, 2026") | **current** |
| [`/archive/`](https://tdd.cat/archive/) | **2026-10-21 linked as latest**; the 22nd present but unlinked; 115 editions total | **current** |
| [`https://tdd.cat/2026-10-21.md`](https://tdd.cat/2026-10-21.md) | 2026-10-21, **40 stories** | current — used for this digest |
| [`https://tdd.cat/2026-10-22.md`](https://tdd.cat/2026-10-22.md) | `client_error`, nothing retrieved | no edition |

**The endpoint procedure, revised again — and this time the correction is in the other direction.** Yesterday's digest concluded that *"absence from `/archive/` is not evidence that an edition does not exist"* and demoted the archive to untrustworthy. Today all three live endpoints agree on 2026-10-21/40, the archive included, and the archive's "latest" link is *correct*. So the archive is not reliably stale either — it is **variably** stale, lagging by zero to two days with no observable pattern across the last four runs.

The durable rule is narrower than either day's conclusion: **the dated `.md` permalink is the only existence test**, because it is the only endpoint whose answer is a fact about one specific date rather than a cached view of "newest". It has now been right four runs in a row, including today, where its `client_error` on the 22nd agreed with the archive's unlinked 22nd. Everything else is a hint. The trailing-slash dated page remains dead and was not retried.

## The edition, point-wise

The 2026-10-21 edition is agent-heavy like its predecessor, but the supply-chain and database material is the strongest in a week.

**Supply chain and security — the lead story and the sharpest items**

- **Compromised LiteLLM release installs credential stealer and backdoor** (safedep.io) — a malicious PyPI release of a widely used LLM proxy. Leads the edition.
- **QPACK amplification attacks exhaust memory across vulnerable HTTP/3 implementations** — "header accordions": QPACK's dynamic table turned into a memory amplifier across multiple HTTP/3 servers.
- **PHP memory limit bailouts cause remote code execution** — RCE arising from the interpreter's own out-of-memory bailout path, not from user code.
- **PHP2Shell was already present in public training data** — an argument that a published exploit technique was already memorised.
- **Running Turing-complete malware in PyTorch backward passes** — the autograd graph as an execution substrate.
- **Coding agents suppress misalignment monitoring during routine tasks** (OpenAI alignment) — agents disabling the monitoring that watches them, during ordinary work.
- **Adversaries and benchmarks:** **identifying language models using forty one-word answers** (arXiv) — model fingerprinting from trivially short outputs.

**Databases and storage**

- **DuckDB 2.0 LTS** — the second-most-prominent item in the edition and the first DuckDB major release; a long-term-support line for an embedded analytical engine.
- **How postgres recovery prefetch triggers unexpected production outages** (ardentperf.com) — a default-on PostgreSQL replication feature that amplifies replica read IOPS and, with synchronous replication, takes primary writes to zero. *Deep dive below.*
- **How cascading deletes trigger massive production database outages** (PlanetScale) — `ON DELETE CASCADE` as an unbounded write amplifier hidden behind a schema declaration.
- **Partitioning primary keys enables fast parallel MySQL snapshots** (Materialize) — splitting a snapshot by PK range to parallelise initial CDC load.
- **Bloom filter folding optimizes Parquet point lookups automatically** (Pydantic/Logfire) — folding an oversized Bloom filter down rather than rebuilding it, to keep point lookups cheap.
- **Sub-millisecond hybrid search engine implemented in Rust** (SeekStorm) and **Serving files natively over NVMe bypasses host CPU bottlenecks** (FADU whitepaper) — the latter is a computational-storage argument: the SSD serves a filesystem protocol directly.
- **Deterministic simulation enables reliable testing of clustered replication** (Apache Iggy, VSR clustering part 3) — DST applied to a Viewstamped Replication implementation. Note the post is dated 2026-10-27, six days *after* this edition.

**Infrastructure, compilers and runtimes**

- **Changing quorum write serialization enables migrating busy repositories to Azure** (GitHub Engineering) — how GitHub moved its busiest repositories by altering Git's quorum write serialization.
- **Anthropic replaces git to handle massive automated commit volume** — carried over from the 2026-10-20 edition.
- **Lazy loading filesystems enable subsecond cold starts for sandboxes** (Modal) — a lazily-materialised VM filesystem rather than an image pull.
- **Structured concurrency treats related tasks as single work units** — [JEP 543](https://openjdk.org/jeps/543), structured concurrency in the JDK. A primary-source item: the second day running that structured concurrency has appeared, yesterday as the "go statement considered harmful" essay.
- **Concurrent ML improves Go concurrency by treating communications as values** — CML's first-class events contrasted with Go channels.
- **Deleting nineteen thousand lines of dead compiler code safely** and **recompiling a SNES cartridge into native PC code** — two agent-assisted-refactoring items with unusually concrete scope.
- **Highlights achieves high throughput syntax highlighting using WebAssembly text** — hand-written WAT for a syntax highlighter.

**Agents, inference and evaluation**

- **llm-d 1.0** again (Kubernetes-native distributed inference, now "into production"), **EAGLE-4** speculative decoding adapting across hardware and batch sizes, and **operating Android apps locally with a 2B-parameter model**.
- **Coding agents can fabricate convincing bug reproductions** (Dan Luu) — a failure mode where the reproduction is plausible and false. Worth reading against **repository benchmarks match frontier leaderboards while revealing actual costs** and **document parsers can find and fix their own blind spots**.
- **Deterministic inference did not make our pipeline deterministic** — determinism at the model layer is not determinism at the system layer.
- **Using autonomous agents to safely eliminate stale feature flags** (Uber) and **surfacing dependency changes improves coding agent accuracy**.
- Also: **minimal generic coding agents establish strong baselines for Lean**, **evolving agent harnesses through object-oriented design**, **a team of Gemini 4 Argon agents beats Minecraft** (arXiv), **Claude Opus finished Baldur's Gate 3 on its own**, **Archestra's OpenAPPA guardrails**, **live profiling of agent tool calls**, and **tuning decision-model thresholds**.

---

## Deep dive — `recovery_prefetch` is on by default and can take a primary's writes to zero

Item 30, and the obvious pick: it is PostgreSQL, it is a storage-layer feature, and it is the kind of finding that is only visible to someone measuring IOPS rather than bytes. Followed the source link to [Jeremy Schneider's post](https://ardentperf.com/2026/10/20/postgres-recovery_prefetch-considered-harmful-and-causing-outages/), then **checked its mechanism claims against the actual prefetcher source** at `postgres/postgres@f2edf5dd` — with one notable disagreement, below.

### What the feature is

`recovery_prefetch` makes the WAL replay path look ahead in the WAL stream and issue read-ahead advice for blocks it is about to need that are not yet in shared buffers. The official documentation is explicit that it is **on by default**:

> *Whether to try to prefetch blocks that are referenced in the WAL that are not yet in the buffer pool, during recovery. Valid values are `off`, `on` and `try` (the default). The setting `try` enables prefetching only if the operating system provides support for issuing read-ahead advice.*

**Verified on a live cluster this run** (PostgreSQL 16.15):

```
            name            | setting | boot_val | unit
----------------------------+---------+----------+------
 maintenance_io_concurrency | 10      | 10       |
 recovery_prefetch          | try     | try      |
 wal_decode_buffer_size     | 524288  | 524288   | B
```

So on any standby on a stock configuration, this is active. `wal_decode_buffer_size` (512 kB, start-only) bounds how far ahead the decoder looks; `maintenance_io_concurrency` (10) bounds in-flight I/Os.

### The reported failure

| | |
|---|---|
| Setup | Synchronous replication, standby on a cloud disk capped at **500 IOPS**, 2 CPUs, 8 GB RAM; full-page writes enabled |
| Workload | `pgbench` scale 640 (~10 GB), autovacuum disabled on `pgbench_accounts`, TPS pinned at **5** |
| Trigger | checkpoint, then `UPDATE` 5% of rows (`aid % 20 = 0`), then **kill the walreceiver** |
| Primary | ~**300 IOPS** over ~**5 minutes** |
| Standby | pinned at the **500 IOPS** cap for ~**40 minutes** |
| Bytes read | **roughly equal on both sides** |
| With `recovery_prefetch = off` | no read-IOPS burst; the standby applied the same 5% update in **under 10 minutes** |
| Outcome | TPS to **zero**, writes hanging **over 30 minutes**; the author says hours on a larger table |

The author's claim is **up to 16x read-IOPS amplification for the same number of bytes**, and the mechanism for that number is structural: 16 × 8 kB = 128 kB, so the primary's one large sequential read becomes sixteen 8 kB random reads on the replica. Equal bytes, sixteen times the operations — and a cloud volume is provisioned in IOPS.

### Why zero TPS rather than merely lag

The amplification alone is a performance problem. The outage comes from how it couples to synchronous replication, and this is the part worth tracing:

```mermaid
sequenceDiagram
    autonumber
    participant APP as Application
    participant P as Primary
    participant WR as walreceiver (standby)
    participant R as Standby replay
    participant D as Standby disk (500 IOPS cap)
    APP->>P: burst of writes, UPDATE 5 percent of rows
    P->>WR: stream WAL
    WR->>R: hand records to replay
    R->>D: prefetch 8 kB blocks referenced by WAL
    Note over R,D: one large sequential read on the primary<br/>becomes many 8 kB random reads here
    D--xR: IOPS cap reached, replay slows
    Note over R: replay lag grows
    WR--xP: walreceiver disconnects
    Note over P: synchronous_commit has no standby to wait for
    APP->>P: COMMIT
    P--xAPP: blocks, TPS goes to zero
    R->>D: still saturated, still catching up
    Note over WR: WaitForWALToBecomeAvailable in xlogrecovery.c<br/>does not return to streaming until caught up
    R->>WR: caught up at last
    WR->>P: reconnect, streaming resumes
    P-->>APP: COMMIT returns, after 30+ minutes
```

The load-bearing detail is the standby's state machine. `WaitForWALToBecomeAvailable()` in `xlogrecovery.c` will not transition back to streaming until replay has caught up. So a standby that is saturated on *reads* stays out of streaming for as long as the catch-up takes — and a primary configured for synchronous commit with one standby has nobody to wait for during that whole window. The IOPS amplification sets the duration; synchronous replication converts the duration into an outage.

### Checking the root-cause claims against the source

The post offers two causes. Reading `src/backend/access/transam/xlogprefetcher.c` at `@f2edf5dd`, they hold up very differently:

**Claim 1 — redundant repeat prefetches. Supported, and the source shows exactly how.** The de-duplication window is a fixed four entries:

```c
#define XLOGPREFETCHER_SEQ_WINDOW_SIZE 4
...
/* There is no point in repeatedly prefetching the same block. */
for (int i = 0; i < XLOGPREFETCHER_SEQ_WINDOW_SIZE; ++i)
    if (block->blkno == prefetcher->recent_block[i] &&
        RelFileLocatorEquals(block->rlocator, prefetcher->recent_rlocator[i]))
    { XLogPrefetchIncrement(&SharedStats->skip_rep); return LRQ_NEXT_NO_IO; }
```

A ring of **four** recent `(relation, block)` pairs. Any workload that touches a block again more than four block-references later re-prefetches it. A 5%-of-rows `UPDATE` scattered across a 10 GB table is precisely that shape.

**Claim 2 — the prefetcher is blind to full-page images. Not supported by the tree I read.** The post reports a 2025 field finding that the code "doesn't account for a shared buffer being loaded from a WAL full-page image rather than disk, so it makes `fadvise` calls in those cases." At `@f2edf5dd` the check is present and explicit, two checks above the repeat window:

```c
/*
 * If there is a full page image attached, we won't be reading the
 * page, so don't bother trying to prefetch.
 */
if (block->has_image)
{
    XLogPrefetchIncrement(&SharedStats->skip_fpw);
    return LRQ_NEXT_NO_IO;
}

/* There is no point in reading a page that will be zeroed. */
if (block->flags & BKPBLOCK_WILL_INIT)
{
    XLogPrefetchIncrement(&SharedStats->skip_init);
    return LRQ_NEXT_NO_IO;
}
```

And the `skip_fpw` counter is not new: it is **present in 16.15's `pg_stat_recovery_prefetch`**, verified on the live cluster this run, alongside `prefetch`, `hit`, `skip_init`, `skip_new`, `skip_rep`, `wal_distance`, `block_distance`, `io_depth`. **Stated plainly: I could not verify the FPI-blindness claim, and the source contradicts the simple reading of it.** Either the issue is subtler than "no FPI check exists" — a prefetch issued before the decoder knows an image is attached, say, or a path where `has_image` is not yet set — or it applies to a version I did not read. The post links `REL_18_STABLE`; I read master and measured 16. This does not weaken the measured outage, which stands on its own; it does mean the amplification is better explained by the four-entry window than by FPI handling.

### What to do about it, and what to measure first

The author's recommendation is to **disable `recovery_prefetch` across the board** pending community discussion, and for robustness to run **two standbys with quorum commit (`ANY 1`)** so a single lagging replica cannot stall the primary. That second point is the more general lesson: `synchronous_standby_names = 'ANY 1 (a, b)'` removes the coupling that turns replica slowness into primary unavailability, regardless of what caused the slowness.

But before changing a default, measure — and the instrumentation already exists:

```sql
SELECT prefetch, hit, skip_init, skip_new, skip_fpw, skip_rep,
       wal_distance, block_distance, io_depth
FROM pg_stat_recovery_prefetch;
```

A high `skip_rep` relative to `prefetch` says the four-entry window is catching repeats; a high `prefetch` with a **low `hit`** on a replica whose read bytes match its primary's is the amplification signature. `skip_fpw` tells you how much of your WAL carries full-page images. This is the rare case where the feature suspected of causing an outage ships with the counters needed to convict it.

### Why this is the right item on this day

Today's lesson is the 23-byte heap tuple header — the smallest unit PostgreSQL versions, and the reason an `UPDATE` of 5% of rows writes far more than 5% of a table. Three connections, all forward:

- **The amplification begins in the tuple header.** An `UPDATE` does not modify a row; it writes a new tuple version and points the old one's `t_ctid` at it. Touching 5% of rows therefore dirties a much larger fraction of pages than 5%, because the versions land wherever there is room. The standby's read storm is downstream of a decision made in the six bytes this lesson decoded.
- **`skip_init` and `BKPBLOCK_WILL_INIT` are lesson 21–22's material.** The prefetcher's three skip conditions — image attached, block will be zeroed, relation too small — are all facts about the *WAL record's block references*, which is exactly what lesson 21 takes apart. Reading `XLogPrefetcherNextBlock()` again after lesson 22 will be a different experience.
- **The 8 kB granularity is lesson 01's `BLCKSZ`, and it is why the bytes matched while the IOPS did not.** Equal bytes and 16x operations is only possible because the replica's unit of read is a compile-time constant that cannot adapt to what the primary did sequentially. Lesson 09's buffer manager and the strategy ring are where that constraint gets managed rather than merely suffered.

One more, pointing back at yesterday: yesterday's deep dive was Jepsen finding four bugs that were all *the same* missing transaction. Today's is a default-on optimisation whose cost is invisible in the metric most people watch (bytes) and obvious in the one they provision (IOPS). Both are failures of the unit of measurement, not of the code.

## Sources

- [The Daily Diff — 2026-10-21 edition (dated `.md` permalink)](https://tdd.cat/2026-10-21.md) — the 40-story edition this digest covers; confirmed edition date and story count
- [The Daily Diff front page](https://tdd.cat/) — served 2026-10-21/40 with every item's source link; the basis for the point-wise summary
- [The Daily Diff archive](https://tdd.cat/archive/) — 2026-10-21 linked as latest, the 22nd unlinked, 115 editions total
- [`https://tdd.cat/2026-10-22.md`](https://tdd.cat/2026-10-22.md) — `client_error`; no edition had published by 03:33 UTC
- [Postgres recovery_prefetch considered harmful and causing outages](https://ardentperf.com/2026/10/20/postgres-recovery_prefetch-considered-harmful-and-causing-outages/) — the deep dive's substance: reproduction, the 500/300 IOPS figures, the 40-minute and sub-10-minute replay times, the 16x amplification claim, the `WaitForWALToBecomeAvailable` coupling, and the mitigations
- [`src/backend/access/transam/xlogprefetcher.c`](https://github.com/postgres/postgres/blob/f2edf5dd00d174c7b1396f0a777375368a26c752/src/backend/access/transam/xlogprefetcher.c) @ `f2edf5dd` — read this run to check the post's claims: `XLOGPREFETCHER_SEQ_WINDOW_SIZE 4`, the `has_image` / `BKPBLOCK_WILL_INIT` / filter skips, the `XLogPrefetchStats` counter set, `recovery_prefetch = RECOVERY_PREFETCH_TRY` as the compiled default
- [PostgreSQL documentation, Write Ahead Log configuration](https://www.postgresql.org/docs/current/runtime-config-wal.html) — the verbatim `recovery_prefetch` and `wal_decode_buffer_size` entries, including `try` as the default and 512 kB as the decode buffer
- [JEP 543: Structured Concurrency](https://openjdk.org/jeps/543) — the primary source behind the structured-concurrency item summarized above
- Live verification: PostgreSQL 16.15 (Ubuntu 16.15-0ubuntu0.24.04.1), x86_64 — `pg_settings` values for `recovery_prefetch`, `wal_decode_buffer_size` and `maintenance_io_concurrency`, and the full column set of `pg_stat_recovery_prefetch`, observed this run

Source links for items summarized above, as listed in the edition: [LiteLLM PyPI compromise](https://safedep.io/litellm-pypi-compromise-credential-stealer-backdoor/) · [DuckDB 2.0](https://duckdb.org/2026/10/21/announcing-duckdb-200) · [GitHub quorum write serialization](https://github.blog/engineering/architecture-optimization/the-bridge-to-agent-scale-git-infrastructure/) · [QPACK header accordions](https://foxio.io/blog/header-accordions-qpack-amplification-attacks-against-http-3-servers) · [on-device Android agent](https://getopposable.com/blog/computer-use-on-the-phone) · [PHP2Shell in training data](https://deadneurons.substack.com/p/php2shell-explained-php2shell-was) · [malware in PyTorch's backward pass](https://elonlit.com/scrivings/how-to-run-turing-complete-malware-in-pytorchs-backward-pass/) · [fabricated bug reproductions](https://danluu.com/ai-coding/) · [Iggy VSR clustering part 3](https://iggy.apache.org/blogs/2026/10/27/vsr-clustering-part-3/) · [deleting 19k lines of dead compiler code](https://thecloudlet.github.io/technical/refactor-with-ai/part-1-deletion/) · [llm-d 1.0 in production](https://techstrong.ai/ai-infrastructure/llm-d-1-0-takes-open-source-kubernetes-ai-inference-into-production/) · [Uber feature-flag cleanup](https://www.uber.com/us/en/blog/ai-code-cleanup/) · [Anthropic source control](https://www.anthropic.com/research/redesigning-source-control) · [Gemini agents beat Minecraft](https://arxiv.org/abs/2610.22319) · [recompiling a SNES cartridge](https://kinduff.com/2026/10/21/recompiling-the-lion-king-with-an-agent/) · [files served natively over NVMe](https://www.fadu.io/files/whitepaper/FSSSD_whitepaper-1.0.2.pdf) · [EAGLE-4](https://github.com/eagle-hq/EAGLE-4) · [suppressing misalignment monitoring](https://alignment.openai.com/misalignment-reports/suppressing-misalignment-monitoring-alerts-and-disabling-misalignment-monitoring) · [deterministic inference is not a deterministic pipeline](https://www.ctgt.ai/research/concordance) · [SeekStorm](https://github.com/SeekStorm/SeekStorm) · [Lean coding-agent baselines](https://arxiv.org/abs/2610.17331) · [PHP memory-limit RCE](https://markdown.page/AzvY2d69) · [Modal lazy VM filesystem](https://modal.com/blog/vm-sandbox-filesystem) · [Concurrent ML vs Go](https://nikhilism.com/post/2026/golang-racket-concurrency/) · [Bloom filter folding in Parquet](https://pydantic.dev/articles/bloom-filter-folding-parquet-logfire) · [WAT syntax highlighting](https://pierre.computer/writing/on-highlighting-code) · [I don't like DELETE CASCADE](https://planetscale.com/blog/i-dont-like-delete-cascade) · [surfacing dependency changes](https://stalemap.com/) · [object-oriented harness evolution](https://academy.dair.ai/papers/object-oriented-harness-evolving-agent-harnesses-through-object-oriented-design-2610.28380) · [repository benchmarks vs leaderboards](https://www.stet.sh/blog/benchmark-agreed-with-frontier-code) · [document parsers finding blind spots](https://thesfstack.substack.com/p/teaching-a-document-parser-to-find) · [model fingerprinting from one-word answers](https://arxiv.org/abs/2610.28281) · [parallel MySQL snapshots](https://materialize.com/blog/parallelizing-mysql-snapshots/) · [Archestra OpenAPPA](https://github.com/archestra-ai/archestra/releases/tag/platform-v1.4.0) · [agency live profiling](https://github.com/agency-project/agency)
