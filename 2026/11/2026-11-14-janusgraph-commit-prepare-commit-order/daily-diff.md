<!--
entry-meta
date: 2026-11-14
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-11-14
slug: janusgraph-commit-prepare-commit-order
-->

# Daily Diff — 2026-11-14

**2026-11-14 · Daily Diff Digest**

**Edition covered: 2026-11-13 ("Friday, November 13, 2026"), 47 stories.**

**How it was found.** At ~03:40 UTC the front page served **2026-11-12 / 66 stories** — the edition lesson 15 digested yesterday — so the ceiling check fired. [`/archive/`](https://tdd.cat/archive/) reported **138 editions** with **2026-11-13 / 47** as its newest, above 11-12/66, 11-11/58, 11-10/63 and 11-09/50. That is the 2026-11-05 one-way cross-check doing exactly its job: the archive's maximum *exceeded* the front page's, so the dated page was probed, and [`https://tdd.cat/2026-11-13/`](https://tdd.cat/2026-11-13/) served the complete 47-story edition with every source URL on the first attempt. Three fetches, no forward probe, no fallback. Rendering note: the filter bar again read *"No stories match the selected filters"* above a full 47-item list — cosmetic, and the fifth appearance in the last ten runs.

One size observation worth carrying: at 47 stories the page fit a single WebFetch, unlike the 109,608-character 11-12 page that needed `offset: 100000`.

---

## The edition, point-wise

**Consensus and distributed systems**

- **Cloudflare chose [QuePaxa](https://blog.cloudflare.com/choosing-quepaxa/) over Raft** for Meerkat, an experimental service backing its internal global control plane. The deep dive below.
- **[Strictly monotonic timestamps and gapless sequences in Rust](https://tsoracle.rs/)** — the single-node version of the same problem QuePaxa solves across continents.
- **[Benchmarking automated root-cause diagnosis across production distributed systems](https://distdebug.com/)**.

**Databases and storage**

- **[Fast search joins eliminate the need to denormalize schemas](https://www.paradedb.com/blog/joins)** — ParadeDB on pushing joins into search-index territory rather than flattening the schema to feed an index. The closest item to this track's subject matter.
- **[Branching and merging PostgreSQL data directly for agents](https://github.com/bitsinfra/gnomon)** — copy-on-write at the data-directory level, which is the Postgres analogue of lesson 15's "a transaction is an in-memory buffer until commit", moved down to the filesystem.
- **[Swapping pgvector for turbopuffer made our database faster and cheaper](https://modem.ai/blog/pgvector-to-turbopuffer)**.
- **[Why financial ledgers must move beyond 64-bit integers](https://tigerbeetle.com/blog/2023-09-19-64-bit-bank-balances-ought-to-be-enough-for-anybody/)** — a 2023 TigerBeetle post resurfacing; relevant to lesson 08's id-width arithmetic for the same reason, which is that a 64-bit budget is spent, not given.
- **[Tparquet halves time-series file sizes using pcodec](https://github.com/aperiodic-io/tparquet/)**.
- **[Trading certainty for space with probabilistic data structures](https://antonz.org/probabilistic-data-structures/)**.
- **[Relate embeds a portable Cypher graph engine anywhere](https://github.com/relatedb/relate)** — an embeddable property-graph engine, i.e. the shape JanusGraph explicitly is not.

**Systems and language runtimes**

- **[Glibc maintainers work to wrap missing Linux system calls](https://lwn.net/Articles/1101647/)**.
- **[Unsafe code in Rust standard library triggers undefined behavior](https://forge-blog.github.io/posts/rust-std-memory-safety/)**.
- **[Publishing breaking Rust changes without requiring coordinated downstream upgrades](https://github.com/dtolnay/semver-trick)**.
- **[Building domain-driven applications with CQRS and event sourcing in Rust](https://github.com/thenativeweb/architecturekit-rust)**.
- **[Compiling pure Python workloads into standalone native executables](https://pyscriptc.dev/)**.
- **[Compiling ordinary loops into heterogeneous multi-core and GPU execution](https://github.com/ThrudTheBarbarian/xc)**.
- **[Classifying shell command effects in microseconds without execution](https://github.com/dirac-run/braket)**.
- **[Migrating an email API to Go reduced latency and memory](https://resend.com/blog/migrating-our-email-api-to-go)**.

**Operations and incident reports**

- **[Lessons from fifty-nine Linux kernel bug reports](https://www.microsoft.com/en-us/security/blog/2026/11/11/lessons-from-59-linux-kernel-bug-reports/)**.
- **[Expanding Azure capacity requires safer operations and failure isolation](https://github.blog/news-insights/company-news/github-availability-report-october-2026/)** — GitHub's October 2026 availability report.
- **[Unread telemetry responses leak file descriptors and crash Chromium](https://blog.dfb.sh/blog/claude-desktop-freeze/)**.
- **[How Aspen scales Git servers for agentic commits](https://reasonmachines.com/blog/aspen)**.

**Browsers and the edge**

- **[Cloudflare open-sources Kitesurf, a stateless browser built for AI agents](https://blog.cloudflare.com/kitesurf-open-source/)**, with the [repository](https://github.com/cloudflare/kitesurf) and an [experimental engine running on serverless workers](https://github.com/cloudflare/kitesurf) listed separately.
- **[Local Chrome extension gives AI agents direct browser automation](https://github.com/214140846/browser-hands)**.

**ML infrastructure and agents** — the largest block of the edition, and the least relevant here:

- **[Managing quantization error in end-to-end four-bit training](https://thinkingmachines.ai/blog/end-to-end-4-bit-training/)**, **[eliminating nondeterminism in LLM inference using integer-only arithmetic](https://nathan.rs/posts/detllm/)**, **[shadow prefilling to cut model-switching latency](https://arxiv.org/abs/2611.13176)**, **[DeepSeek ultrafast with multi-server cache sharing](https://github.com/basetenlabs/deepseek-ultrafast)**, **[llama.cpp v1.0.0 stabilizing core APIs and caching](https://github.com/ggml-org/llama.cpp/releases/tag/v1.0.0)**, **[pooling unified memory across networked Macs](https://modelmesh.app/)**, **[rewriting the hot path doubles token generation in ComfyUI](https://blog.comfy.org/p/doubling-llm-token-rates-in-comfyui)**, **[a Rust gateway relaying native provider APIs](https://github.com/langfuse/langfuse/tree/main/ai-gateway)**.
- Agent harness and evaluation: **[tuning an agent harness](https://github.com/Kiln-AI/harness-tuner)**, **[harness choices affect accuracy and cost](https://joelniklaus-harness-leaderboard.hf.space/)**, **[OWE-Bench for long-horizon engineering projects](https://www.mercor.com/blog/introducing-owe-bench/)**, **[beating the frontier on NetHack](https://antithesis.com/blog/2026/nethack/)**, **[testing whether agents know when to help unprompted](https://deepnote.com/research/benchmark)**, **[designing tools around workflows rather than endpoints](https://p99log.com/designing-tools-for-ai-agents/)**, **[treating tests as data to protect contracts with agents](https://martinfowler.com/articles/tests-as-data.html)**, **[agent fleets beyond attention bandwidth](https://www.stet.sh/blog/running-an-agent-fleet)**, **[always-on agent primitives](https://claude.com/resources/articles/building-always-on-agents)**, **[self-hosting managed agents inside a security perimeter](https://claude.com/resources/articles/self-hosted-claude-managed-agents)**.
- Weight-space work: **[frontier models editing smaller model weights directly](https://www.cmpnd.ai/blog/weight-space-programming-dspy.html)**, **[weight editing curbing fabrication in college advising](https://harlyn1.github.io/weight-space-advising-logs/)**, **[augmenting JEPA world models for long-horizon tasks](https://haddyalchaer.com/jepa)**, **[how decision models work under the hood](https://jbarrow.ai/2026-10-23-deep-dive-into-decision-models/)**.
- Supply chain and reference: **[outdated npm pins revealing AI-generated packages](https://github.com/kobima6813/fossil-versions)**, **[an AI security wiki](https://nachi065.github.io/AI-Security-Wiki/)**.

---

## Deep dive: Cloudflare picked a randomized, leaderless consensus protocol, and the reason is the timeout

Source followed: **[Choosing QuePaxa](https://blog.cloudflare.com/choosing-quepaxa/)**, Cloudflare's engineering blog.

### The problem is fault *detection*, not fault tolerance

Raft, Multi-Paxos and Viewstamped Replication are all leader-based. Safety survives a dead leader; **liveness does not**, until a new one is elected. And electing one requires deciding that the old one is gone — which over a wide-area network is undecidable in the way that matters operationally: a slow network and a crashed node produce the same evidence.

The consequence is a tuning parameter nobody can set correctly. Raft-style election timeouts are conventionally **5–10× the network RTT**, and a round trip across continents is measured in hundreds of milliseconds, so detecting a failed coordinator takes on the order of **a second**. Set it tighter and you get spurious elections; elections interfere with each other and the system can enter a metastable state where it keeps electing and never commits. Cloudflare's motivating incident is concrete: a 2020 partial switch failure made a consensus-backed dashboard **~80× slower for six hours**, with writes unavailable until it was resolved.

That is the "tyranny of timeouts" the [QuePaxa paper](https://bford.info/pub/os/quepaxa/quepaxa.pdf) (Tennage, Băsescu et al., SOSP 2023) is named after.

### What QuePaxa does differently

Three mechanical changes, and they compose:

| Mechanism | Raft / Multi-Paxos | QuePaxa |
|---|---|---|
| Roles | leader + followers; the leader is required | **proposers** + **recorders**; the leader is *optional* |
| Competition between proposals | ballot numbers; a higher ballot invalidates a lower one, so two candidates can lock each other out | **random priorities**; a proposer that sees a higher-priority proposal relayed back by a recorder **adopts** it |
| Progress when the leader is slow | wait out the election timeout, then elect | **hedging**: non-leader proposers wait a small *local* timeout and then propose; no detection required |

The round has four phases, and the shape is what matters:

- **Phase 0** — propose, and adopt any higher-priority proposal a recorder relays. The optional leader always holds the **maximum** priority, so when the leader is healthy its proposal wins immediately: one round trip, decided.
- **Phase 1** — spread the adopted proposal to a majority of recorders.
- **Phase 2** — check whether the highest-priority proposal seen matches yours; if so, decide.
- **Phase 3** — carry the best-seen proposal into the next round.

Safety rests on majority intersection, as in Paxos. Liveness rests on the randomized priorities: because proposers *adopt* rather than *outbid*, the system converges rather than livelocking. The paper's analysis predicts roughly a **50% chance of deciding per round**; Cloudflare reports the observed rate is much higher because of that convergence.

### Why the hedge timeout is not an election timeout in disguise

This is the sharp point of the post, and it is easy to miss. Both designs have a timeout. The difference is what a *badly chosen* value does:

- A Raft election timeout of 5 ms on a 100 ms network **freezes the cluster** — it is permanently electing.
- A QuePaxa hedge timeout of 5 ms on a 100 ms network **adds proposers**. They adopt the leader's higher-priority proposal and the round still decides in phase 0.

So the hedge timeout is not bounded below by network latency. It is a pure cost/latency knob: smaller means more messages and faster recovery from a slow leader, larger means fewer messages. Cloudflare's implementation puts the wait on the **client** rather than on replicas — clients send to all replicas, and non-leader proposers hold back — which reduces message volume relative to the paper's formulation.

### The production numbers

From the Meerkat deployment, which spans servers on nearly all continents and maintains a replicated log of `get`/`put` operations backing a key-value store:

| Measurement | Value |
|---|---|
| Decisions reached in phase 0 (one round trip) | **upwards of 95%** |
| Throughput when decisions fall back to phase 2 | **≤ ~1/3 of normal**, consistent with three round trips |
| Decision rate during a leader failure | fell to **~65% of baseline**, never to zero |
| Time to restore 100% | **~53 s** (15:00:32 → 15:01:25 UTC), i.e. as long as it took to replace the leader |

The 65% figure is the whole argument in one number. A Raft cluster in the same situation is at **0%** until the election completes; QuePaxa degraded to two-thirds throughput and kept committing, because the leader was a latency optimisation rather than a requirement.

### What it changes, and what it does not

- **It changes what you tune.** There is no longer a parameter whose misconfiguration costs availability. That is a different class of operational property from "we tuned it well".
- **It changes the failure mode of a *slow* node**, which is the common case the leader-based protocols handle worst. A leader that is up but at the far end of a congested link no longer gates the cluster.
- **It does not remove the leader**, and Cloudflare keeps one for the fast path. What it removes is the leader's monopoly on progress, plus the ability to swap leaders at runtime without stalling.
- **It is not battle-tested**, and the post says so. QuePaxa's published design is **crash-stop**; Cloudflare is extending it to crash-recovery and formally verifying their Rust implementation alongside simulation and fault injection. Against Raft's decade of production exposure and reference implementations, that is the real cost of the choice.

```mermaid
flowchart TB
    subgraph clients["Clients"]
        C["client sends to ALL replicas<br/>then waits the HEDGE timeout<br/>(local; not bounded by RTT)"]
    end

    subgraph proposers["Proposers — anyone may propose"]
        L["LEADER proposer<br/>priority = MAX (always wins ties)"]
        P2["non-leader proposer<br/>priority = random<br/>proposes only after hedge timeout"]
        P3["non-leader proposer<br/>priority = random"]
    end

    subgraph recorders["Recorders (acceptors)"]
        R1["recorder 1<br/>records every proposal seen<br/>relays the highest-priority one back"]
        R2["recorder 2"]
        R3["recorder 3"]
    end

    subgraph round["One round, four phases"]
        PH0["Phase 0 — propose + ADOPT any higher priority<br/>leader fast path: decided in 1 round trip<br/>&gt;95% of decisions in production"]
        PH1["Phase 1 — spread adopted proposal to a majority"]
        PH2["Phase 2 — highest-priority seen == mine? decide<br/>3 round trips → ~1/3 throughput"]
        PH3["Phase 3 — carry best-seen proposal into next round<br/>paper: ~50% decide per round; observed: higher"]
    end

    DEC["DECIDED — safety from majority intersection,<br/>liveness from priority convergence,<br/>NOT from fault detection"]

    C --> L
    C --> P2
    C --> P3
    L --> R1 & R2 & R3
    P2 --> R1 & R2 & R3
    P3 --> R1 & R2 & R3
    R1 -.->|"relay higher priority"| P2
    R2 -.->|"relay higher priority"| P3
    PH0 --> PH1 --> PH2 --> PH3
    PH3 -->|"new round, best proposal carried"| PH0
    PH0 -->|"leader fast path"| DEC
    PH2 -->|"match"| DEC
    R1 --> PH0
```

### Read against this track

Lesson 16, today's entry, ends with a commit path whose correctness depends on *"deletions are applied before additions"* — a contract the `KeyColumnValueStore` javadoc states and that lesson 03 established **nothing verifies**. QuePaxa is the same move at a different scale: rather than detecting a condition (is the leader alive?), it arranges the protocol so the condition never needs deciding. JanusGraph does the local version of this constantly — the column *is* the uniqueness constraint (lesson 10), the id's suffix bits *are* the type dispatch (lesson 13), `consolidate()` *is* the write ordering (lesson 16). Designing so the hard question disappears beats answering it well, and it is the same instinct both layers down and several thousand kilometres up.

## Sources

- [The Daily Diff, 2026-11-13 edition](https://tdd.cat/2026-11-13/) — 47 stories; fetched this run, the source of every item above
- [The Daily Diff front page](https://tdd.cat/) — fetched this run; served 2026-11-12 / 66 stories, one edition behind
- [The Daily Diff archive](https://tdd.cat/archive/) — fetched this run; 138 editions, newest 2026-11-13 / 47
- [Choosing QuePaxa — Cloudflare blog](https://blog.cloudflare.com/choosing-quepaxa/) — fetched this run; the deep-dive source
- [QuePaxa: Escaping the Tyranny of Timeouts in Consensus (PDF)](https://bford.info/pub/os/quepaxa/quepaxa.pdf) and its [landing page](https://bford.info/pub/os/quepaxa/) — URLs seen in a web search this run; linked for the phase analysis and the per-round decision probability, not fetched
- [QuePaxa — EPFL Infoscience record 308891](https://infoscience.epfl.ch/record/308891) and the [UCL project page](https://sec.cs.ucl.ac.uk/projects/PJ-QUEPAXA) — search results seen this run, for publication metadata
