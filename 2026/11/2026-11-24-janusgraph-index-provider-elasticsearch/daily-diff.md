<!--
entry-meta
date: 2026-11-24
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-11-24
slug: janusgraph-index-provider-elasticsearch
-->

# Daily Diff — 2026-11-24

**2026-11-24 · Daily Diff Digest**

> **Today's edition has not published.** `https://tdd.cat/2026-11-24.md` returned a client error, and so did `https://tdd.cat/2026-11-23.md` — the same answer lesson 25 got for 11-23 yesterday, so that edition appears never to have published rather than being merely late. Following the fallback procedure recorded in `TRACK.md`, this digest covers the **2026-11-20 edition, "Friday, November 20, 2026", 50 items**, which was the one uncovered edition in the pool: the 11-21 edition went into lesson 24's digest and 11-22 into lesson 25's. The 50 items were read in full through two fetches of the markdown endpoint (77,940 characters; the second at offset 39,958), so nothing in it is unread.
>
> **Quoting note.** Item titles and source URLs below are verbatim. The one-line descriptions are this digest's own summaries, not the edition's blurb text.

---

## 1. The edition, point by point

**Databases and storage (9 items)** — the densest cluster, and the reason this edition was worth not dropping.

- **PostgreSQL 19 introduces native repack and replica consistency** — [postgresql.org/docs/release/19.0](https://www.postgresql.org/docs/release/19.0/). Native `REPACK`, the plan/stash-advice extensions, and `WAIT` for read-your-writes. Lesson 23's digest already deep-dived `REPACK CONCURRENTLY`.
- **Compacting Postgres tables concurrently without application downtime** — [enterprisedb.com](https://www.enterprisedb.com/blog/squeezing-tables-postgres-19). The same feature from the operator's side: concurrent compaction against `VACUUM FULL`'s `ACCESS EXCLUSIVE` lock.
- **Valkey 9.2 improves efficiency with forkless snapshots and compressed replication** — [valkey.io/blog/valkey-9.2-release](https://valkey.io/blog/valkey-9.2-release/). **Today's deep dive, §2.**
- **Unifying text search and vectors in one Postgres index** — [paradedb.com](https://www.paradedb.com/blog/filter-aware-vector-search). Text, filters and vectors co-located in one index. Lesson 25's digest carried this as a sidebar.
- **Serving pre-aggregated PostgreSQL answers directly from memory** — [github.com/janbjorge/straum](https://github.com/janbjorge/straum). In-memory aggregates kept current by logical replication.
- **Open-source product analytics platform built on ClickHouse and NATS** — [github.com/pug-sh/pug](https://github.com/pug-sh/pug). Go + NATS + PostgreSQL + ClickHouse.
- **Understanding database internals from slotted pages to query plans** — [sql-in-depth.vedgupta.in](https://sql-in-depth.vedgupta.in/). A WebAssembly workbench for SQLite and PostgreSQL internals — directly adjacent to this log's own SQLite track.
- **Strict serializability unifies transaction and operation consistency hierarchies** — [jepsen.io/consistency/models](https://jepsen.io/consistency/models). Jepsen's model hierarchy.
- **How hole punching in sparse files optimizes local S3 caching** — [bzrk.dev](https://www.bzrk.dev/blog/sparse-files/). Caching S3 byte ranges in sparse files via `FALLOC_FL_PUNCH_HOLE`.

**Correctness, verification and formal methods (4 items)** — the second cluster worth flagging.

- **Formal models verify production Raft implementations against protocol guarantees** — [antithesis.com](https://antithesis.com/blog/2026/verifying-formal-models/). TLA+ models plus *trace validation* against etcd-raft and HashiCorp Raft. The runner-up for today's deep dive.
- **TensorLean enables formal verification of production GPU kernels** — [tensorlean.ai](https://tensorlean.ai/). PTX-level verification; 83 of 149 kernels proven, 38 bugs found.
- **Isolating flaky Dynamo export tests by minimizing iteration time** — [dev-discuss.pytorch.org](https://dev-discuss.pytorch.org/t/debugging-story-the-case-of-the-flaky-dynamo-export-tests/1124). pytest-vs-unittest divergence pointed at state leakage.
- **Implementing IMAP reveals deeply circular protocol dependencies** — [raphting.dev](https://raphting.dev/imap-is-indeed-complicated/). `LIST`/`STATUS` couple flags, message state and folder counts.

**Systems, runtimes and performance (8 items)**

- **Zero-allocation Rust architecture unlocks extreme network and storage throughput** — [serviceradar.cloud](https://serviceradar.cloud/blog/ring-per-core-io-in-longshore). Thread-per-core `io_uring`; 36.4 GB/s to S3 from one thread.
- **Scala Native approaches Rust performance with vastly lower memory** — [wvlet.org](https://wvlet.org/uni/blog/native-server-performance). ~80% of Rust/axum throughput, far below JVM memory.
- **Android replaces its vulnerable Binder C driver with Rust** — [lwn.net/Articles/1102982](https://lwn.net/Articles/1102982/).
- **Running a real Linux kernel compiled directly to WebAssembly** — [github.com/pithings/unkernel](https://github.com/pithings/unkernel). Kernel as one Wasm module; user binaries under a RISC-V JIT.
- **Running portable WebAssembly binaries directly on bare-metal microcontrollers** — [github.com/Igwe-Starking/eccles-wasm](https://github.com/Igwe-Starking/eccles-wasm). Heapless C99 interpreter, one fixed arena.
- **Rebuilding the Inferno operating system with Rust and WebAssembly** — [github.com/ericvh/purgatorio](https://github.com/ericvh/purgatorio). Namespaces and 9P2000, rebuilt.
- **Immediate cache hits can starve the event loop** — [octri.dev](https://octri.dev/blog/when-cache-hits-starve-the-event-loop). Awaiting a synchronously-resolved promise never yields.
- **Modern attention variants fundamentally alter transformer inference bottlenecks** — [servingstudio.cs.washington.edu](https://servingstudio.cs.washington.edu/blog/attention-variants/). KV-cache and decode bytes well below GQA at 128K.

**Infrastructure and platform (7 items)**

- **Bedrock deploys fully hardened Kubernetes clusters onto bare machines** — [bedrockctl.com](https://bedrockctl.com/). Includes a TLA+-verified PostgreSQL failover controller.
- **Hosting sleeping Kubernetes control planes in a single binary** — [github.com/JustVugg/nostromo](https://github.com/JustVugg/nostromo). Idle control planes sleep, wake on demand.
- **Filesystem tuning makes microVM environments fast for coding agents** — [kosem.ai](https://kosem.ai/blog/making-microvms-fast-for-coding-agents). virtiofs → virtio-blk is the whole win.
- **Building hosting infrastructure to achieve complete Next.js compatibility** — [easel.sh](https://www.easel.sh/blog/building-the-infrastructure-underneath-nextjs). Rust routing engine validated against Next.js's own test suite.
- **Giving AI coding agents isolated machines instead of credentials** — [github.com/Sanne/incus-spawn](https://github.com/Sanne/incus-spawn).
- **Agent host protocol enables persistent sessions across devices** — [github.blog](https://github.blog/engineering/engineering-principles/building-the-agent-host-protocol-for-sessions-that-continue-across-devices/).
- **Hardening launch infrastructure with an automated chaos loop** — [claude.dev](https://claude.dev/blog/chaos-loop-claude-docs-launch/).

**Security (5 items)**

- **Escaping Docker containers through model runner argument injection** — [thezdi.com](https://www.thezdi.com/blog/2026/11/17/hostile-inference-exploiting-docker-model-runner-for-container-escapes). Argument injection chained to host code execution.
- **Unauthorized replacement of production weights removed safety refusals** — [primeintellect.one](https://primeintellect.one/security/advisories/deepseek-v4-1-flash-model-substitution). A model substituted behind an endpoint; refusals collapsed.
- **Attackers can abuse browser-embedded language models for post-exploitation** — [arxiv.org/abs/2611.25086](https://arxiv.org/abs/2611.25086).
- **Forensic audit exposes severe Google AI Ultra cache overbilling** — [github.com/fatrabbit29/antigravity-ultra-quota-audit](https://github.com/fatrabbit29/antigravity-ultra-quota-audit). Allegation from local log analysis.
- **Forensic tool audits local OpenAI Codex logs for anomalies** — [github.com/NeHumorist/codex-local-audit](https://github.com/NeHumorist/codex-local-audit).

**Engineering practice and tooling (6 items)**

- **Designing software for understandability combats complexity from code generation** — [jschuster.org](https://jschuster.org/blog/2026/11/19/design-for-understandability/). Design by contract and class invariants as the counterweight.
- **Code review bottlenecks open source development despite coding agents** — [cloud.google.com](https://cloud.google.com/blog/products/application-development/lessons-from-building-sashiko-an-agentic-code-review-system). Google's Sashiko, on Linux kernel patches.
- **Handing software delivery to agents eliminates pull request bottlenecks** — [runway.com](https://runway.com/news/engineering/building-a-software-factory). 38% of merged PRs; approval wait down to minutes.
- **Pyrefly achieves full typing specification conformance with faster checks** — [pyrefly.org/blog/v1.5](https://pyrefly.org/blog/v1.5/). Passes the conformance suite, ~40% faster.
- **Tracing monadic Nix builds enables applicative Bazel conversions** — [fzakaria.com](https://fzakaria.com/2026/11/18/build-systems-a-la-trace). Traces turn dynamic Nix builds into static Bazel graphs.
- **Eliminating avoidable agent token costs from broken prompt caches** — [github.com/tejachavali/prefixwise](https://github.com/tejachavali/prefixwise). Finds the byte offset causing a cache miss.

**Data governance, ML research and the rest (11 items)**

- **A boolean PII flag is not enough for governance** — [kore-nordmann.de](https://kore-nordmann.de/blog/a-boolean-pii-flag-is-not-enough.html). Classify identifier *strength* and transformation state separately.
- **Language models can rewrite their own weights** — [arxiv.org/abs/2611.21885](https://arxiv.org/abs/2611.21885). A 27B model editing weights without backprop; +6.6 points over 8 iterations.
- **Reinforcement learning does not expand base model reasoning capacity** — [arxiv.org/abs/2504.13837](https://arxiv.org/abs/2504.13837). RLVR improves k=1, not high-k coverage.
- **Autonomous agents establish collaborative scientific communities and research norms** — [alphaxiv.org](https://www.alphaxiv.org/abs/2611.society-ai-scientists). 936 agents; rated more rigorous, less novel.
- **ReasoningBank distills generalizable strategies from success and failure** — [research.google](https://research.google/blog/reasoningbank-enabling-agents-to-learn-from-experience/).
- **WavePP maximizes pipeline parallel prefill throughput via asynchronous admission** — [arxiv.org/abs/2609.35263](https://arxiv.org/abs/2609.35263). 2–3× prefill in TensorRT-LLM.
- **Haru-4B enables low-latency continuous full-duplex spoken dialogue** — [blog.higurashi.ai](https://blog.higurashi.ai/melte/haru4b/). ~65 ms end-to-end.
- **Benchmarking autonomous coding agents on web performance tuning contests** — [isu-bench.trasta.dev](https://isu-bench.trasta.dev/). ISUCON-style load against Go + PostgreSQL.
- **Stateful Stripe sandbox enables isolated agent evals and tests** — [github.com/Kiln-AI/stripe_world](https://github.com/Kiln-AI/stripe_world).
- **Browser card games can prevent cheating without central servers** — [chiplis.com](https://chiplis.com/blog/serverless-cheaterless-magic). Mental poker plus ZK shuffle proofs.
- **Merkle tree certificates reduce overhead for post-quantum signatures** — [datatracker.ietf.org](https://datatracker.ietf.org/doc/draft-ietf-plants-merkle-tree-certs/). Inclusion proofs replacing per-certificate CA signatures.

---

## 2. Deep dive — Valkey 9.2's forkless snapshots

Picked over the Antithesis Raft-trace-validation piece because it is the item whose mechanism is load-bearing for anyone who has ever had a Redis-family node OOM during a `BGSAVE`, and because the gap between what the release post claims and what the reference page actually documents is itself the finding.

**Sources followed:** the release post [valkey.io/blog/valkey-9.2-release](https://valkey.io/blog/valkey-9.2-release/) (titled *"Valkey 9.2 offers forkless snapshots, compressed replication, ACL roles, and more"*, by Kyle Davis and Dragos Andriciuc, dated 2026-11-19) and the reference page it links, [valkey.io/topics/forkless](https://valkey.io/topics/forkless/).

### 2.1 The problem being solved

A Redis-family snapshot has always been `fork()` plus copy-on-write. The child gets a frozen view of the address space for free; the parent keeps serving writes; every page the parent writes is copied. The cost is therefore proportional to the *write rate during the snapshot*, not to the snapshot's duration alone, and the worst case is pathological: the forkless page states plainly that **memory usage can grow by up to 2× in extreme cases**. That is the number that makes a 60%-memory Redis node a 120%-memory OOM the moment a write burst lands during a save.

### 2.2 What replaces `fork()`

Here the documentation is thinner than the feature deserves, and it is worth being exact about what is and is not stated:

- **Stated:** Valkey 9.2 adds *forkless save* and *forkless full sync*, which run *without calling `fork()`*, taking the snapshot **inside the running process**. The modules section mentions RDB save callbacks running *on a background thread, at the same time as commands run on the main thread*.
- **Stated:** enabling the infrastructure *stores an additional 4 bytes with each key*.
- **Stated about write behaviour:** a write to a key during a forkless operation carries a small risk of **waiting briefly**; larger values raise both the probability and the wait. Reads do not wait, with the exception of reads that internally mutate — `PFCOUNT` is named. *"String values of up to 512 bytes are generally not affected."* Splitting very large collections is the advice.
- **Not stated anywhere on either page:** the thread model, thread names, **what the 4 bytes per key are used for**, and how point-in-time consistency is achieved at all. No CPU or latency numbers; latency is described only as possibly *"slightly higher"* than the fork-based equivalent.

The shape is strongly suggestive — 4 bytes per key plus "a write may wait, proportionally to value size" plus "reads don't wait" reads like a per-key epoch or version stamp with a copy-on-write at *object* granularity rather than page granularity, so the snapshotting thread copies the one object a writer is about to clobber instead of the OS copying a 4 KiB page. **But the documentation does not say that, so this digest does not claim it.** Answering it properly needs the 9.2 source, which was not read.

What *is* unambiguous is the trade being made:

| | fork + COW | forkless |
|---|---|---|
| Granularity of copying | **page** (4 KiB), by the MMU | **object/key**, by Valkey |
| Steady-state cost | none | 4 bytes per key, always |
| Cost during snapshot | up to **2× RSS** worst case | brief waits on writes, proportional to value size |
| Who pays | the whole process, in memory | the individual writer, in latency |
| Reads during snapshot | unaffected | unaffected, except `PFCOUNT` and friends |

That is a textbook memory-for-latency swap, and the reason it is opt-in is that the 4-bytes-per-key tax is unconditional while the 2× memory risk is only realised under write-heavy load.

### 2.3 Configuration, and the fallback surface

Two options, and the second depends on the first:

| Option | Default | Settable | Meaning |
|---|---|---|---|
| `forkless-infrastructure-enabled` | `no` | **startup only** | enables support; **does not by itself change** how saves or syncs run |
| `forkless-policy` | `require-fork` | `CONFIG SET` at runtime | `prefer-forkless` uses forkless where possible and falls back to fork; only settable to `prefer-forkless` when the infrastructure option is `yes` |

So the on-switch is two settings, one of which needs a restart, and the shipped default is fork for both. **`prefer-forkless` is a preference, not a guarantee** — the page is explicit that forkless is *"not yet a full replacement for `fork()`"*, and the fallback list is long enough to matter in production:

- **Always still forks:** AOF rewrite, and atomic slot migration.
- **Falls back for a save:** a loaded module without forkless support; the target node of an in-progress atomic slot migration.
- **Falls back for a full sync:** a replica older than 9.2; a replica using `repl-diskless-load swapdb` when the primary has `repl-diskless-sync yes`; a non-supporting module on the replica when the primary is diskless; **`repl-compression` enabled on both sides**; and `valkey-cli --functions-rdb`.

That fifth one is the sharp edge: **the headline features of this release are mutually exclusive on the full-sync path.** Turning on compressed replication — the feature with the best-measured win in the whole post, 52% less replication bandwidth at under 1% throughput cost on 3 million ~315-byte keys — pushes full syncs back onto `fork()`, and therefore back into the 2× memory risk. Nothing in the release post says so; only the reference page does.

Cancellation is also asymmetric in a way worth knowing: `FLUSHDB` cancels a forkless save outright and *"must be started again"* if it was a `BGSAVE`, while an automatic save-config save is retried; during a diskless full sync, `FLUSHDB` cancels only if the flushed database holds **more than half of all keys**.

### 2.4 Observability, which is unusually good

This is the part to copy if you ever ship a dual-path mechanism. The feature is instrumented so that you can tell which path actually ran, and why the other one did not:

- `INFO persistence`: **`rdb_current_bgsave_type`** (`fork` / `forkless` / `none`), **`rdb_last_bgsave_type`**, and **`forkless_estimated_seconds_remaining`** (`-1` when nothing is running).
- `INFO debug`, five fallback counters: `forkless_fallbacks_module`, `forkless_fallbacks_slot_import`, `forkless_fallbacks_missing_inband_repl`, `forkless_fallbacks_compression`, `forkless_fallbacks_exclude_data`.
- The server log records the method of each full sync **and the reason for each fallback**.

A per-reason fallback counter is the right design for an opt-in path that silently degrades: without it, `prefer-forkless` would be unfalsifiable from the outside. Set it, watch `forkless_fallbacks_compression` climb, and you have discovered §2.3's interaction yourself.

```mermaid
flowchart TD
    REQ["BGSAVE / full sync requested"] --> POL{"forkless-policy"}
    POL -->|"require-fork (default)"| FORK
    POL -->|prefer-forkless| INFRA{"forkless-infrastructure-enabled<br/>= yes?"}
    INFRA -->|no| REJECT["CONFIG SET refused:<br/>policy cannot be prefer-forkless"]
    INFRA -->|yes| ELIG{"eligible?"}

    ELIG -->|"module without support"| FORK
    ELIG -->|"slot migration target"| FORK
    ELIG -->|"replica older than 9.2"| FORK
    ELIG -->|"repl-compression on both sides"| FORK
    ELIG -->|"diskless-load swapdb + diskless-sync"| FORK
    ELIG -->|"--functions-rdb"| FORK
    ELIG -->|otherwise| FL

    FORK["fork + copy-on-write<br/>page-granular copying<br/>up to 2x RSS worst case"] --> OBS
    FL["forkless: snapshot inside the process<br/>background thread<br/>4 bytes per key, always<br/>writers may wait briefly"] --> OBS

    OBS["INFO persistence:<br/>rdb_current_bgsave_type<br/>rdb_last_bgsave_type<br/>forkless_estimated_seconds_remaining"]
    OBS --> CNT["INFO debug fallback counters:<br/>module / slot_import /<br/>missing_inband_repl /<br/>compression / exclude_data"]

    AOF["AOF rewrite"] --> ALWAYS["always forks"]
    SLOT["atomic slot migration"] --> ALWAYS
```

### 2.5 The rest of 9.2, since the same post carries it

- **Compressed replication.** `repl-compression` adds streaming compression to the steady-state replication stream, negotiated **per replica**, so older or opted-out replicas keep receiving plaintext. LZ4 in the benchmark; **52% bandwidth reduction at under 1% throughput cost** on 3M × ~315-byte keys. Separately, `rdbcompression` now accepts `lz4` and `zstd` for whole-stream RDB compression (the legacy per-string LZF behaviour is unchanged), with Zstandard gated behind `BUILD_ZSTD=yes` at build time and pitched at cross-region replication. **Downgrade caveat:** switch `rdbcompression` to a legacy mode and run `BGREWRITEAOF` first — an AOF base written with `lz4`/`zstd` needs a version that can read streaming-compressed RDB.
- **Sorted sets move from skiplist to B+ tree** for large sets: per-member overhead from 50.3 bytes to 28.5–32.0 bytes, 36–43% less, measured on 5M members with 20-byte elements. The B+ tree **compacts sparse leaves on delete**, so shrinking sets and frequent score updates return memory — which a skiplist does not do.
- **Small hashes with field expiration keep the listpack encoding** instead of always converting to a hashtable: the post's example measures `MEMORY USAGE` at **80 bytes versus 240** on 9.1.4.
- **Load handling:** cluster heartbeats, replication and slot migration are given priority over heavy client traffic (`priority-preemptive-poll-interval-us`, `priority-subnets`, `maxclients-reserved`). Separately `repl-throttling-enabled` slows client writes via a token bucket once the largest replica output buffer hits **25% of the configured soft limit** — trading write latency for not losing a replica to a full resync.
- **ACL roles:** named reusable selector sets, `ACL SETROLE` / `role=`, applied immediately to every holder; a user may add permissions on top of a role but **cannot restrict** it.
- **Path Hash:** a new radix-tree-backed type with `PH*` commands for exact lookup, **longest-prefix match** and prefix traversal over binary-safe paths.
- **Conditional commands:** `EXEC` takes `IFEQ` / `IFNE` / `NX` / `XX` for optimistic locking without `WATCH`; new `DELEX`, `INCREX`, `XACKDEL`, `XDELEX`; `SET` gains `IFNE`.
- **Hot key detection:** `HOTKEYS GET` / `HOTKEYS RESET`, enabled by `hotkeys-top-k`, using **Space-Saving** over a fixed-size summary — no rate thresholds, just the top K.
- **Behaviour changes:** `ZRANGEBYSCORE` and `ZCOUNT` now reject a bare `(` or an empty string as a score bound (previously read as `0`); active expiration now counts toward the dirty counter, so configured save points trigger more often.

### 2.6 Why it matters

Two things, and the first is the one with legs.

**Forkless is the Redis family finally paying for its snapshot in the currency it can afford.** `fork()`-plus-COW was free to implement and catastrophic to operate: the cost appears as unbounded memory at the worst possible moment, it cannot be bounded in advance, and the only mitigations are "keep RSS under 50%" and "hope". Moving the copying from page granularity to object granularity turns an unbounded memory spike into a bounded, per-key, measurable latency cost plus a flat 4-bytes-per-key tax. That is a strictly better shape for a capacity planner even where the absolute numbers are worse, and it is the same move JanusGraph's own index layer made this week in a different currency — see this day's lesson, where the fix for a non-idempotent write was to pay a constant cost on every mutation so that the pathological case stops existing.

**And the feature-interaction matrix is the real release note.** `prefer-forkless` plus `repl-compression` silently means forked full syncs. The post advertises both features in its title; only the reference page, under a heading about fallbacks, says they do not compose on that path. If you turn either on, the thing to actually watch is `forkless_fallbacks_compression`.

---

## Sources

All fetched during this run.

- `https://tdd.cat/2026-11-24.md` — **client error, edition not published.**
- `https://tdd.cat/2026-11-23.md` — **client error**, consistent with lesson 25's finding yesterday.
- [`https://tdd.cat/2026-11-20.md`](https://tdd.cat/2026-11-20.md) — HTTP 200, `text/markdown`, 77,940 characters, read in full across two fetches (offsets 0 and 39,958). Front matter: *"Friday, November 20, 2026"*, canonical [`https://tdd.cat/2026-11-20/`](https://tdd.cat/2026-11-20/). 50 items.
- [`https://valkey.io/blog/valkey-9.2-release/`](https://valkey.io/blog/valkey-9.2-release/) — read in full. Deep-dive primary source.
- [`https://valkey.io/topics/forkless/`](https://valkey.io/topics/forkless/) — read in full. The config defaults, the fallback list, the cancellation rules and all of the observability surface come from here, not from the release post.
- Every item URL in §1 is reproduced exactly as the 2026-11-20 edition wrote it. The pages behind them were not fetched, except the two Valkey pages above.
