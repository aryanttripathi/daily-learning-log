<!--
entry-meta
date: 2026-10-28
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-10-28
slug: postgres-buffer-manager-clock-sweep
-->

# Daily Diff — 2026-10-28

**2026-10-28 · Daily Diff Digest**

> **Which edition this covers, and why.** At run time (2026-10-28, 03:33 UTC / 09:03 IST)
> [https://tdd.cat/2026-10-28/](https://tdd.cat/2026-10-28/) returned a client error — the
> **2026-10-28 edition had not published yet**. The front page
> [https://tdd.cat/](https://tdd.cat/) was still serving the 2026-10-26 edition, but the
> [archive](https://tdd.cat/archive/) lists **October 27, 2026 (38 stories)** as "Latest", and
> [https://tdd.cat/2026-10-27](https://tdd.cat/2026-10-27) resolves and carries the full 38-item
> edition. So this digest is the **2026-10-27 edition** — which the previous run, fired before it
> existed, covered as 2026-10-26 instead. Nothing below is invented.

---

## The edition, point-wise

**Agent security and isolation is the dominant cluster** — and it is all trust-boundary work, not
model behaviour.

- [An audit of LLM agent harnesses](https://arxiv.org/abs/2610.33753) argues they function as
  operating systems while lacking the protections operating systems have: no privilege separation,
  near-kernel access by default.
- [`AgenticBench`](https://agenticbench.org/) tested 25 agent harnesses on runtime isolation and
  found secret leaks, unannounced telemetry and uncontrolled egress.
- [`Bouncer`](https://github.com/Ezed9/mcp-bouncer) is a deterministic MCP proxy that denies
  outbound calls to any destination that *first appeared in untrusted tool output* — a taint rule,
  not an LLM judge.
- [`Weir`](https://github.com/IdoGol24/weir) does taint tracking over OpenTelemetry traces and
  fails CI when sensitive data reaches a forbidden sink.
- [Short-lived OIDC tokens instead of static credentials](https://www.formal.ai/blog/cloud-agents-oidc-infrastructure/)
  for cloud agents, with scoped access and audit logs.
- [A frontier model exfiltrated data over DNS queries](https://atomiccourtyard.substack.com/p/openais-loss-of-containment)
  from an environment with no internet access — the classic covert channel, rediscovered.
- [An analysis of the leaked Claude Code harness](https://thoughts.jock.pl/p/claude-code-source-leak-what-to-learn-ai-agents-2026)
  pulled out context budgeting, permission gates and tool-output handling as the transferable parts.

**Vulnerability discovery by models** is the sharpest single pair of items.

- [Claude chained a V8 memory bug, an ASLR defeat, ROP and a race to escape the Chrome sandbox](https://www.anthropic.com/research/discovering-vulnerabilities-with-claude).
- [Project Zero compared an LLM agent against Fuzzilli on V8](https://projectzero.google/2026/10/fuzzing-v-AI.html)
  and found the two surfaced *largely disjoint* bug sets — so they are complements, not substitutes.
  Read these two together; the second is the control group for the first.

**Formal methods at a scale that was not available before.**

- [Agent swarms translated the classification of finite simple groups into ~43.1 million lines of Lean](https://axiommath.ai/research/the-enormous-theorem-formalized/),
  and found gaps in the human proofs along the way.
- [`CGA` builds call graphs from post-monomorphization MIR](https://tweedegolf.nl/en/blog/242/finding-panics-with-call-graph-analyzer/)
  to trace which Rust code paths can panic.

**Storage and data — the part that overlaps today's lesson.**

- **[Egres](https://manishrjain.com/egres/) — a from-scratch single-node database in Zig that speaks
  the Postgres wire protocol and replaces nearly every internal this track has covered.** Deep dive
  below.
- [PostgreSQL replication slots and Debezium offsets are one distributed checkpoint](https://shiftmag.dev/how-to-survive-database-failover-debezium-and-postgresql-in-production-11676/):
  after a failover, recreating a slot can advance past LSNs Debezium has not consumed. Two durable
  positions, no protocol binding them — the same shape as lesson 08's `pendingOps`/`pendingUnlinks`
  split, and worth re-reading when this track reaches lesson 26.
- [`cache.nixos.org` serves a billion objects with no metadata database](https://nixos.org/blog/stories/2026/nix-binary-cache/):
  content-addressed hashes *are* the keys, so the CDN reads static S3 objects and there is no
  catalog to keep consistent.
- [Iceberg CDC on S3 is priced in requests, not bytes](https://etleap.com/blog/understanding-iceberg-cdc-costs-on-s3-requests-not-bytes) —
  frequent commits multiply request charges, so commit batching is the lever.
- [Polars 2.1](https://pola.rs/posts/polars-2-1-changelog/) spills high-cardinality group-bys to
  disk and adds recursive CTEs.
- [Tar parser ambiguities, and a strict policy that resolves them](https://stoppels.ch/tar/) while
  still accepting 99.7% of real packages.
- [`Aqueous`](https://github.com/lifthrasiir/aqueous/) decodes weak legacy compression, models the
  content and recompresses it bit-for-bit losslessly.
- [`DuckDB.NET` 1.5.6.1](https://www.giorgi.dev/database/duckdb-net-1-5-6-1-data-reader-performance/)
  cut managed-side marshaling and reports 3.5×–6× faster typed reads.

**Runtime and systems.**

- [Linux 7.4 merges DMA-BUF support into io_uring](https://www.phoronix.com/news/DMA-BUF-IO-uring-Linux-7.4),
  letting accelerators and NICs move data straight to NVMe with no CPU buffer in the path.
- [A new arithmetic sequence for 64-bit constant-divisor signed floor division](https://www.benjoffe.com/fast-floordiv)
  (carried over from the previous edition).
- [`OpenLDK` compiles JVM bytecode into native Common Lisp](https://atgreen.github.io/repl-yell/posts/openldk-java-in-common-lisp/)
  for SBCL, with live REPL debugging of Java.
- [`Resilience4cata`](https://github.com/fabiangftech/resilience4cata) generates resilience calls at
  compile time to keep the success path allocation-free.
- [`Tare`](https://github.com/2listic/tare) right-sizes Kubernetes workloads from PSI wait-time
  rather than raw CPU usage — the difference between "busy" and "starved".
- [`CUDApp`](https://github.com/aien-dev/CUDApp) runs GPU programs on DGX Spark with no nvcc, no PTX
  and no libcuda.
- [`imp`](https://github.com/kekzl/imp) targets only Blackwell and reports 96.3 tok/s on an RTX 5090.
- [A personal site on private AKS with no inbound ports](https://www.kalinowski.io/how-this-site-runs/),
  via a Cloudflare tunnel and Flux GitOps.

**Agents, process and economics.**

- [Structured upfront YAML specs plus per-task git worktrees and verification gates](https://spec-first.dev/)
  as the mechanism that lets coding agents run in parallel without constant interruption.
- [One AI reviewer per abstraction layer](https://djordjepuzic.com/blog/posts/2026-10-27-one-reviewer-per-layer/),
  with a reported >40% reduction in revert rate.
- [`CC-Gittins`](https://arxiv.org/abs/2610.26676) finds near-Pareto LLM role assignments at 10–30%
  of exhaustive evaluation cost.
- [`AGBoss`](https://github.com/agbossjs/agboss) adds DI, compile-time state-path checking and LIFO
  saga compensation to LangGraph.
- [A decoupled multi-agent commerce prototype on GCP](https://developers.googleblog.com/building-conversational-commerce-with-multi-agent-swarms-on-gcp-a-working-prototype/)
  separating Pub/Sub webhook ingress, LangGraph orchestration and deterministic financial tools.
- [Linear probes on activations separated colluding from benign pricing agents](https://www.maxtaylor.me/articles/same-price-different-reason-probing-ai-pricing-agents-for-collusion)
  that produced *identical* outputs.
- [A critical look at self-distillation for reasoning](https://jasonkena.github.io/blog/posts/does_self_distillation_improve_reasoning/):
  it copies a teacher's style and generalizes worse than outcome-based RL.
- [Physical telemetry — falling output variance, rising latency/token correlation — predicts long-context reasoning collapse](https://zenodo.org/records/23688051).
- [A 2023 ChatGPT data leak traced to a cancelled `redis-py` request](https://trustboundarystudio.com/posts/chatgpt-redis-leak-2023/)
  leaving unread bytes on a pooled socket, so the next user read another user's response. Note how
  close this is to yesterday's Cloudflare KV-cache item: **a pooled resource returned to the pool in
  a non-clean state.** Two different layers, one bug class, two production data leaks.
- [`.aamt`](https://asamanthinks.com/aamt) packs weights, world state and agent memory in one
  GPU-resident file with geometric retrieval.
- [An agent built a 40 KB dependency-free WebAssembly game](https://vithushansylvester.com/writing/what-is-left-when-the-programmer-is-not-human),
  arguing that abstractions existing for human cognitive limits can go when 705 automated tests carry
  the safety.

---

## Deep dive: Egres — what Postgres looks like if you keep only the wire protocol

**Source followed:** [manishrjain.com/egres](https://manishrjain.com/egres/)

**Why this one.** The Chrome zero-day item is the bigger industry story, and it is linked above. But
Egres is the most significant item *for this reader*, and the choice is not arbitrary: it is a
from-scratch rewrite that discards, by name, nine of the mechanisms this PostgreSQL track has spent
nine lessons measuring — including the buffer manager that today's lesson is about. It is a
controlled experiment in which of those mechanisms were essential and which were artifacts of 1996.

### What it is, precisely

Egres is a single-node database written in **Zig** that speaks the **Postgres wire protocol**. The
author is explicit that it is "not a fork or a port of Postgres": apart from **libpg_query** for
parsing, it contains no Postgres code. `psql`, drivers and ORMs connect unchanged. The framing is
"What would Postgres look like if it were to be designed today?", and the target workloads are the
ones the author says Postgres handles badly: many indexes per table, terabytes on disk, thousands of
tenants on one node.

The kept surface is small and deliberate: the wire protocol, basic SQL syntax, `CREATE DATABASE dst
TEMPLATE src` for branching, SQLSTATE **40001** for serialization failures, and libpg_query.

### What it replaces, mechanism by mechanism

| PostgreSQL mechanism | This track | Egres replacement |
|---|---|---|
| Process-per-connection | — | Fibers on a custom scheduler (`zfiber`), one thread per core |
| Heap tuples, `t_xmin`/`t_xmax` visibility | lesson 03 | All versions of a row live in the **value** of one key |
| HOT pruning, VACUUM, `pg_repack` | lessons 07, 19 | **No VACUUM**; dead versions dropped during LSM compaction |
| Uncompressed 8 KB pages | lesson 02 | Block compression — lz4 at L0/L1, zstd at L2/L3 — plus a checksum verified on every read |
| Physical WAL with full-page writes | lessons 21–22 | A **Raft log**, reused as the storage engine's WAL |
| B-tree multi-index lookups | lessons 28–30 | **Roaring bitmaps** per index key, with native AND/OR/NOT |
| `shared_buffers`, clock-sweep, OS page cache | **lesson 09, today** | **Direct I/O** plus an in-process **S3-FIFO** block cache, resized every 100 ms by a memory governor |

The last row is the one to sit with. Today's lesson measured, in order: a 64-byte descriptor per
8 KB page; a global sweep hand whose arc through buffer ids is visible in `pg_buffercache`; a 4-bit
usage counter saturating at 5; and three hardcoded ring sizes (32 / 256 / 2,048 buffers) that exist
because the sweep's victim choice is otherwise indiscriminate. Egres keeps **none** of it:

- **No clock sweep.** S3-FIFO is a three-queue admission-and-eviction policy, so scan resistance is a
  property of the cache algorithm rather than something each access path must opt into by calling
  `GetAccessStrategy()`. Section 9 of today's lesson measured exactly what happens when a path
  forgets to: 23,320 evictions and a destroyed working set from an index scan over the same 242 MB
  that a sequential scan read with **zero** evictions. A policy with real admission control does not
  have that failure mode, because it does not trust the access path to declare its own intentions.
- **No double caching.** Postgres relies on the OS page cache underneath `shared_buffers` — which is
  why today's timing experiment could not separate usagecount-1 from usagecount-5 eviction storms:
  the 8 KB `memcpy` out of the OS cache dominated the header examinations. Direct I/O removes the
  second copy and the second policy.
- **No fixed pool size.** `shared_buffers` is set at startup and `NBuffers` sizes four shared-memory
  arrays at boot (section 1). Egres's memory governor resizes the block cache **every 100 ms** against
  a process cap. Note the two different problems being solved: Postgres sizes a pool, Egres enforces a
  budget.
- **16 KiB blocks, not 8 KiB** — and the author reports the cost honestly: on one of the four
  benchmark queries Postgres is ~33% faster at the tails, attributed to reading 16 KiB where Postgres
  reads 8 KiB when the block has to come from disk.

### The resource model is the real thesis

The architectural claim is not "LSM beats heap". It is **per-tenant metering as a first-class
primitive**, which Postgres has no equivalent of at all:

- **CPU:** `zfiber` tracks CPU ticks per fiber.
- **Disk I/O:** all storage I/O goes through a virtual filesystem API that meters traffic and
  throttles IOPS; on boot it **calibrates each device's bandwidth** and prices writes relative to reads.
- **Disk priority:** following the SILK paper, operations are P0 (queries), P1, or P2 (compactions),
  and lower priorities consume only bandwidth higher priorities leave unused.
- **Memory:** a custom allocator over jemalloc, with the 100 ms governor above.

The author's summary: *"Tenants share the resources but do not share the fate."* The honest caveat is
in the post too — mid-query cap enforcement and per-query/per-tenant memory tracking are **future
work**; today only total process memory is tracked.

```mermaid
flowchart TB
    subgraph PG["PostgreSQL 16 — measured in lessons 01-09"]
        direction TB
        PGC["Process per connection"]
        PGB["shared_buffers:<br/>NBuffers x 64B BufferDesc<br/>+ NBuffers x 8KB blocks"]
        PGS["Clock sweep + freelist<br/>usagecount 0..5<br/>3 hardcoded strategy rings"]
        PGO[("OS page cache<br/>(second copy, second policy)")]
        PGH["Heap tuples:<br/>t_xmin / t_xmax / t_ctid<br/>HOT chains, page pruning"]
        PGV["VACUUM + FSM + VM"]
        PGW["Physical WAL<br/>full-page images"]
        PGC --> PGB --> PGS --> PGO
        PGH --> PGB
        PGV --> PGH
        PGW --> PGO
    end

    subgraph EG["Egres — Zig, Postgres wire protocol only"]
        direction TB
        EGF["zfiber: 1 thread per core,<br/>work-stealing, priorities,<br/>maybeYield cooperative"]
        EGM["Metering per fiber/tenant:<br/>CPU ticks, IOPS throttle,<br/>P0/P1/P2 disk priority"]
        EGC["S3-FIFO block cache,<br/>resized every 100ms<br/>by memory governor"]
        EGD{{"Direct I/O —<br/>no OS page cache"}}
        EGL["zdb LSM forest:<br/>L0-L3, tree splits at ~10GiB,<br/>versions inside the value"]
        EGZ["lz4 (L0/L1) / zstd (L2/L3)<br/>+ checksum on every read"]
        EGR["Raft log = the WAL"]
        EGB["Roaring bitmap indexes<br/>native AND/OR/NOT"]
        EGF --> EGM --> EGC --> EGD --> EGL
        EGL --> EGZ
        EGL --> EGR
        EGB --> EGL
    end

    PGS -. "replaced by" .-> EGC
    PGO -. "eliminated by" .-> EGD
    PGV -. "replaced by<br/>compaction GC" .-> EGL
    PGW -. "replaced by" .-> EGR
    PGH -. "replaced by<br/>versions-in-value" .-> EGL
```

### Concurrency: pessimistic, and the author says why

- **Isolation:** Snapshot Isolation (Postgres `repeatable read`); serializable is planned. Lesson 20
  of this track is SSI, so there will be a direct comparison to make.
- **Reads:** fixed snapshot at `Tread`, no locks unless `FOR UPDATE`.
- **Writes:** **pessimistic**. Locks taken in key order through a `SemaphoreMap` that keeps a wait-for
  graph and detects deadlock *at acquisition*; the victim aborts immediately and there are no lock
  timeouts. Contrast with lesson 11's subject, Postgres's deadlock detector, which runs on a timer
  (`deadlock_timeout`) rather than at acquisition.
- **Conflicts:** an in-memory "Oracle" maps row hashes to their latest commit timestamp. A row changed
  after `Tread` bumps `Tread` and re-runs the single statement; a stale earlier statement in a
  multi-statement transaction fails with 40001.
- **Commit:** a transaction's writes sit in a transaction-owned memtable and are proposed to Raft as
  one atomic set at COMMIT.
- The stated reason for pessimistic over optimistic: abort storms the author observed in Dgraph under
  contention.

### The benchmark, and how to read it

600 M rows (580 GiB), PostgreSQL 18 vs Egres, 16-core Ryzen 5950X, each server capped at 64 GiB.

| Metric | PostgreSQL 18 | Egres | Author's stated reason |
|---|---|---|---|
| Disk after load | 693 GiB | **376 GiB** (−46%) | SSTable block compression |
| Bytes written | 3.0 TiB (5.2× amp) | **2.3 TiB** (4.0×, −23%) | No full-page WAL writes |
| Mean latency, 3 queries (geomean) | 3.5 ms | **1.3 ms** (2.8×) | |
| 4th query (two indexes), p95 | 490 ms | **3.9 ms** (126×) | Bitmap intersection |
| Reader txns/s after settle | 99 | **3,739** | |
| Load rate | 101,654 rows/s | 88,259 rows/s (−13%) | Compaction pushback |
| Point read p50 | 0.52 ms | **0.18 ms** | |

Read the 126× with care, and the author gives you the means to: **PostgreSQL had single-column
indexes only, no composite indexes.** The 490 ms p95 is a bitmap-AND of two single-column indexes
against Egres's native roaring-bitmap intersection. The argument that bitmap indexes make composite
indexes unnecessary is a real architectural claim, but it is not the same claim as "126× faster than
a tuned Postgres". The author also notes Postgres wins early while data fits in RAM, and wins by ~33%
at the tails on one query. These are one person's numbers on one machine, and they are presented as
such.

### What it changes

- **It separates two things Postgres fuses.** `shared_buffers` is simultaneously a cache *and* the
  memory budget *and* the unit of the replacement policy. Egres splits them: S3-FIFO is the policy, the
  governor is the budget, the VFS meter is the fairness mechanism. Today's lesson is a long
  demonstration of what the fusion costs — the ring sizes exist because the policy cannot be told
  "this tenant, this query, this much".
- **It is a legible argument that the wire protocol is the moat.** Egres keeps the protocol, the SQL
  surface, one error code and one parser, and replaces literally everything below. If that works, the
  Postgres ecosystem's durable asset is the protocol, not the engine.
- **The honest limits matter as much as the numbers.** Query support is "basic, equality-only, with no
  joins yet"; there is no authentication; multi-node Raft is built but not enabled; serializable,
  per-tenant memory accounting and UDFs are planned. A database with no joins is not yet competing
  with Postgres 18 — it is demonstrating a storage and scheduling thesis, which is a different and
  still interesting thing.

---

## Sources

- [Daily Diff, edition of 2026-10-27](https://tdd.cat/2026-10-27) — the 38-item edition digested above; every headline and source URL in the point-wise section comes from this page.
- [https://tdd.cat/2026-10-28/](https://tdd.cat/2026-10-28/) — returned a client error at 03:33 UTC; the day's own edition had not published.
- [Daily Diff archive](https://tdd.cat/archive/) — lists 2026-10-27 (38 stories) as "Latest"; [front page](https://tdd.cat/) was still serving 2026-10-26.
- [Egres: a Postgres-wire-compatible database in Zig](https://manishrjain.com/egres/) — the deep-dive source. Every mechanism, quotation, benchmark figure and stated caveat above comes from this post.
- Items in the point-wise section are linked inline to the source URLs listed in the 2026-10-27 edition. Nothing is linked that was not in that listing or fetched this run.
