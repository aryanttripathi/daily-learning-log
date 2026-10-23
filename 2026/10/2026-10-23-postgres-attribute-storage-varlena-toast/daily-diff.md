<!--
entry-meta
date: 2026-10-23
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-10-23
slug: postgres-attribute-storage-varlena-toast
-->

# Daily Diff — 2026-10-23

**2026-10-23 · Daily Diff Digest**

## Which edition this covers

**No 2026-10-23 edition has published.** This run fired at 03:33 UTC (09:03 IST). This digest covers the **2026-10-22 edition, 42 stories** — the newest edition, and one not yet covered by this log (yesterday's digest covered 2026-10-21).

| Endpoint | Served | Verdict |
|---|---|---|
| [front page](https://tdd.cat/) | **2026-10-22, 42 stories** ("Thursday, October 22, 2026") | **current** — used for this digest |
| [`/archive/`](https://tdd.cat/archive/) | **2026-10-22 linked as "Latest"**; 116 editions total; the 23rd present as plain text, not a link | **current** |
| [`https://tdd.cat/2026-10-22.md`](https://tdd.cat/2026-10-22.md) | `client_error`, nothing retrieved | **false negative** |
| [`https://tdd.cat/2026-10-23.md`](https://tdd.cat/2026-10-23.md) | `client_error`, nothing retrieved | no edition (agrees with the archive) |

**The endpoint procedure, corrected again — and this time the rule from the last four runs is the one that broke.** Yesterday's digest concluded that *"the dated `.md` permalink is the only existence test, because it is the only endpoint whose answer is a fact about one specific date rather than a cached view of 'newest'"*, and credited it with four correct runs. Today it returned `client_error` for **2026-10-22**, an edition that demonstrably exists: the front page serves its 42 stories and the archive links it as "Latest". The permalink produced a false negative on an edition it had itself been expected to confirm.

So the honest state of the procedure is that **no single endpoint is a reliable existence test.** The archive and the front page agreed today, and agreed yesterday; the permalink agreed yesterday and disagreed today. The narrower durable rule, which survives all five runs: **an edition exists if the front page serves it or the archive links it**, and the dated permalink is corroboration when it answers and worth nothing when it does not. One run is not enough to tell whether the `.md` route has been retired or is merely flaky — but the asymmetry matters. A `client_error` there can no longer be read as "no edition"; only the archive's unlinked day can, and that is what is being relied on for the 23rd.

## The edition, point-wise

42 stories. Agents still dominate by count, but the strongest material is in storage and in the two items about replacing a database engine with a different one underneath the same interface.

**Databases, storage and compression**

- **Plugging ClickHouse MergeTree into PostgreSQL enables fast vectorized analytics** — MergeTree embedded in PostgreSQL as a *table access method*, plus a vectorized executor. *Deep dive below.*
- **Replacing large Redis clusters with storage-dense Lavik nodes** — NVMe-backed storage claimed to match Redis throughput while going from **300 nodes to 3**.
- **How cascading deletes trigger massive production database outages** (PlanetScale) — `ON DELETE CASCADE` holds row locks that block schema migrations and stall queries. Carried over from the 2026-10-21 edition.
- **Storing only changed columns makes triggers slower** (depesz) — a column-delta audit scheme that saves storage and pays for it in per-`UPDATE` trigger CPU. A direct measurement rebuttal of a popular pattern.
- **How Postgres `TABLESAMPLE` avoids full table scans for exploratory queries** (boringsql) — `TABLESAMPLE SYSTEM` reads a random subset of *pages*, which is why it is cheap and why it is biased.
- **ZXC achieves ultra-fast decode speeds for read-heavy workloads** — a compressor tuned for decode rather than ratio, claimed ahead of LZ4 and zstd on read-heavy data.
- **Replacing custom filesystems with simple block devices improves reliability** (Fly.io) — Sprites swapped a bespoke filesystem for a block device backed by chunked object storage.
- **Why freed application memory is not returned to the kernel** — glibc arenas keep freed pages mapped, so RSS stays flat after frees.
- **Qdrant internal gRPC API exposed distributed clusters without authentication** — pre-1.20.0, unauthenticated peers could join a cluster and read data. [GHSA-frc7-44gm-99qq](https://github.com/qdrant/qdrant/security/advisories/GHSA-frc7-44gm-99qq).

**Runtimes, memory layout and compilers**

- **Hash tables make dynamic language object attribute access slow** (bernsteinbear) — object shapes and inline caches instead of per-access hashing. Worth reading *directly against today's lesson*: it is the same problem — "where in this record is field X" — solved by a hidden class rather than by a `TupleDesc` and an `attcacheoff`.
- **Building a garbage collector in and for Rust** — a cycle-collecting GC built from safe Rust abstractions.
- **Triangulating millions of map polygons faster than earcut** — ear clipping replacing constrained Delaunay triangulation.
- **Offloading Brotli compression to thread pools preserves NGINX latency** — compression off the event loop; flat tail latency, higher memory.
- **Tracing Linux command wall-clock latency with eBPF** — `whytime` attributes wall-clock time to I/O, locks, network waits and scheduling.
- **Queue placement in shared memory resolves gVisor sandbox hangs** — a missed queue relocation left work unclaimed and the watchdog never fired.
- **Display-p3 canvas rendering leaks the underlying CPU architecture** — Skia's colour math differs between ARM and x86, which is a fingerprint.
- **The rule of two mitigates severe memory corruption vulnerabilities** — [Chromium's rule-of-2](https://chromium.googlesource.com/chromium/src/+/master/docs/security/rule-of-2.md): untrusted input, unsafe language, high privilege — pick two.

**Security and supply chain**

- **Turning PHP out-of-memory errors into code execution** — an OOM `longjmp` corrupts a `HashTable` in PHP 8.5, yielding a use-after-free. The third consecutive edition carrying a PHP-bailout RCE item.
- **Running Turing-complete malware in PyTorch backward passes** — a loss term that evaluates to zero and still steers weights to execute code. Third consecutive edition.
- **Guarding AI agents and blocking unauthorized actions** — taint tracking to stop untrusted data reaching privileged outputs.
- **Safe and private runtime for autonomous AI agents** — [NVIDIA/OpenShell](https://github.com/NVIDIA/OpenShell), a Rust runtime isolating agent shells behind policy. Leads the edition.

**Infrastructure and industry**

- **Anthropic builds its own source control system** — Git struggled at roughly **30,000 pull requests per day**; the internal replacement is called **Scion**. Third consecutive edition, now with a name and a number.
- **Splitting trillion-parameter models across wafer-scale chips for fast inference** (Cerebras) — quantize weights and KV caches, partition layers to stay in on-chip SRAM.
- **Serving existing vLLM models on Google Cloud TPUs** — an out-of-tree plugin running unmodified vLLM model code via `torch.compile` and XLA.
- **Off-policy correction enables reinforcement learning across mixed accelerators** (Modal) — cheaper mixed-precision samplers feeding a single BF16 trainer.
- **A lightweight Go proxy replaces memory-heavy coding agent gateways** — a single-binary LLM gateway idling under 5 MB.
- **Transport and coordination layer for the disconnected edge** — BLE / Wi-Fi Direct / Reticulum / Nostr with MLS encryption.
- **Splitting model weights enables private inference across multiple computers** — layer-sharding to pool memory across machines.

**Agents, evaluation and generation**

- **A team of Gemini 4 Argon agents beats Minecraft** ([arXiv 2610.22319](https://arxiv.org/abs/2610.22319)) — four agents, shared memory and chat, Ender Dragon in 27 hours. Carried over.
- **Reinforcement-trained small model flies Tesla Roadster better than frontier models** — a 4B model beating frontier models on a low-latency control task.
- **Hierarchical next-concept modeling enables fast parallel text generation** ([HazyResearch/ncm](https://github.com/HazyResearch/ncm)) — coarse-to-fine hierarchical codes generated in parallel.
- **Operating Android apps locally using a two-billion parameter model** — a quantized on-device GUI model with ~40 common actions handled by deterministic code. Carried over.
- **Mercury 3 offers competitive speed but specialized hardware decodes faster**, **branching and rewinding agent state with effectively-once side effects**, **how browser agents systematically handle automated bot challenges**, **comparing generated thinking with natural language activation explanations**, **humans approve intent while machines prove conformance**, **wake-sleep architectures for long-horizon agents**, **autonomous build workflows require measured promotion gates**, **how recommendation loops learn user preferences within seconds**, **optimized linear-attention and state-space kernels for JAX on Hopper**, and **large language models make formal verification scalable at assembly level** (Anthropic — LLM-generated proofs over RISC-V assembly, finding bugs testing missed).

---

## Deep dive — ClickHouse's MergeTree as a PostgreSQL table access method

Item 38, and the only possible pick on this day. Today's lesson spent its length on what a *heap* tuple's bytes look like — `MAXALIGN` padding between attributes, three varlena header formats, a 18-byte TOAST pointer — and syllabus lesson 35 is *"The table AM interface: what `heap` assumes and what a different table AM must supply."* This item is somebody answering lesson 35 the hard way. Followed the source link to [Igor Suhorukov's post](https://igor-suhorukov.github.io/blog/2026/10/21/my-postgresql-is-faster-than-yours/) and read it against the mechanism.

### What was actually built

Three PostgreSQL 19 extensions, **no fork and no core patches**:

| Extension | Repo | What it is |
|---|---|---|
| `ch_table_engines` | `igor-suhorukov/pg_ch_table_engines` | ClickHouse `MergeTree` exposed as a **table access method** |
| `pg_vexec` | `igor-suhorukov/pg_vexec` | a **vectorized executor** built on the ported ORCA optimizer |
| `iheap` | `igor-suhorukov/pg_inplace_update_am` | a table AM that **updates rows in place**, with an undo log |

The DDL is the part that should make an internals reader sit up:

```sql
-- postgresql.conf: shared_preload_libraries = 'ch_table_engines'
CREATE EXTENSION ch_table_engines;
CREATE TABLE hits (...) USING mergetree
    WITH (order_by = 'CounterID, EventDate, UserID', storage = 'pages');
```

`USING mergetree`. Not a foreign data wrapper, not an extension-managed shadow table — the table access method slot, the same slot `heap` occupies. Everything today's lesson decoded (`t_hoff`, the null bitmap, `varatt_external`) is `heap`'s private business, and `USING mergetree` replaces all of it.

**How ClickHouse got in-process.** Roughly **28 of ClickHouse's ~150 submodules statically linked** into `libchte_engine.so`, from the `dbms` library directly — explicitly **not** chdb and **not** clickhouse-local. The author reports that stock `StorageMergeTree` could not be used as-is: a second process opening the same directory produced `"Another server instance in same directory is already running"`, which is ClickHouse assuming one server per data directory and PostgreSQL assuming one *process per connection*. He wrapped it to survive that model. The ClickHouse version is not stated.

### The numbers

Headline, **ClickBench on a laptop, 10M rows**, geometric mean:

| Configuration | Geomean | vs. ClickHouse |
|---|---|---|
| ClickHouse | 16.8 ms | 1× |
| PostgreSQL 19 + `mergetree` + `pg_vexec` | **49.9 ms** | ~3× slower |
| Vanilla PostgreSQL | — | ~**39×** slower |

Per-query, the ones with a visible mechanism:

| Query | Before | After |
|---|---|---|
| Q28 | 1,090 ms | 213 ms |
| Q23, single-threaded | 471 ms | 247 ms |
| Q10, single-threaded | 148.2 ms | 85.6 ms |
| Q20, page-skipping simulation only | 510 ms | **5.3 ms** |
| `ORDER BY CounterID LIMIT 10` | 2.1 s | 67–76 ms |
| TPC-DS Q2 | 3,447 ms | 1,246 ms |

And `pg_vexec` alone on **heap**, no MergeTree: 625 → 506 ms with the standard planner (−19%), 1,647 → 1,349 ms under ORCA (−18%). Vectorizing the executor over row-store heap pages buys about a fifth. Vectorizing it over a column store buys the other order of magnitude.

### The finding that matters

Buried in the post, and it is the whole lesson:

> **without `pg_vexec`, MergeTree lost to heap, because the stock executor receives rows one at a time through a slot.**

The TPC-H and TPC-DS tables under ORCA say it in numbers:

| Benchmark | MergeTree + stock executor | heap + stock executor | MergeTree + `pg_vexec` |
|---|---|---|---|
| TPC-H SF1 | **61.8 s** | 17.2 s | **6.9 s** |
| TPC-DS | **380.5 s** | 58.8 s | **48.6 s** |

MergeTree with PostgreSQL's ordinary executor is **3.6× slower than heap** on TPC-H and **6.5× slower** on TPC-DS. Swap in the vectorized executor and the same storage becomes 2.5× *faster* than heap on TPC-H and 1.2× faster on TPC-DS — a swing of roughly **9×** on TPC-H from changing nothing but how rows leave the scan.

```mermaid
flowchart LR
    subgraph ROW["heap + stock executor: the row-at-a-time contract"]
        H1["heap page, 8 kB<br/>24-byte header, MAXALIGN padding,<br/>varlena headers, TOAST pointers"]
        H2["heap_deform_tuple<br/>walk attributes, peek pad bytes"]
        H3["TupleTableSlot<br/>ONE tuple"]
        H4["ExecProcNode<br/>per-tuple expression eval"]
        H1 --> H2 --> H3 --> H4
    end

    subgraph BAD["MergeTree + stock executor: 61.8 s, LOSES to heap"]
        B1["MergeTree part<br/>columnar, compressed, LowCardinality dictionaries"]
        B2["materialize whole rows<br/>to satisfy the slot contract"]
        B3["TupleTableSlot<br/>ONE tuple"]
        B4["ExecProcNode<br/>per-tuple expression eval"]
        B1 --> B2 --> B3 --> B4
    end

    subgraph GOOD["MergeTree + pg_vexec: 6.9 s"]
        G1["MergeTree part<br/>granule skipping on the order_by key"]
        G2["read the FIRST predicate's columns only"]
        G3["batch of 1024 rows, columnar<br/>LowCardinality arrives as dictionary indexes"]
        G4["VecScan / VecHashJoin / VecAgg / VecSort<br/>kernels bound to function OIDs"]
        G5["read remaining columns ONLY for<br/>batches with surviving rows"]
        G6{"kernel exists for this expression?"}
        G7["stock Postgres expression interpreter<br/>fallback, row-at-a-time"]
        G1 --> G2 --> G3 --> G4 --> G6
        G6 -- yes --> G5
        G6 -- no --> G7
    end

    ROW -. "17.2 s TPC-H SF1" .-> VERDICT
    BAD -. "61.8 s: columnar storage, row-at-a-time pipe" .-> VERDICT
    GOOD -. "6.9 s" .-> VERDICT
    VERDICT["the storage format and the executor<br/>contract have to change together"]
```

### Why that is the interesting claim

The table AM interface is a *storage* abstraction. It lets you replace how bytes sit on disk. It does not let you replace **how rows leave the scan**, and the interface's unit is one tuple in one `TupleTableSlot`. So a columnar engine plugged in through the AM slot pays for its format twice: once to read columns it will throw away, and again to reassemble rows it did not store as rows. That is a 3.6× regression against the row store it replaced — *using faster storage*.

`pg_vexec` is the other half, and it works through ordinary extension hooks rather than core changes: `VecScan`, `VecHashJoin`, `VecAgg` and `VecSort` are added as plan nodes that **compete on cost** with the normal ones, priced by ORCA's model when ORCA is driving. Batches are **1,024 rows in columnar form**, kernels are **bound to function OIDs**, and an expression with no kernel **falls back to the stock expression interpreter**. Two details are the real engineering: the scan reads the first predicate's columns first and only fetches remaining columns for batches that still have surviving rows; and low-cardinality columns arrive as **dictionary indexes**, so an expression is evaluated once per distinct value rather than once per row. That second one is the mechanism behind Q20's 510 ms → 5.3 ms.

The author is also candid about the scars: a missing `T_TargetEntry` branch was producing **single-row batches** (a vectorized executor accidentally running row-at-a-time), and `VecSort`'s cost went **negative, −46403**, in forced mode. And one honest anti-result: a test table with `LowCardinality`, no compression and small granules came out **11.8 GB versus 4.8 GB** — attributed to a benchmark restart that interrupted merges and left **35 parts instead of 3**. Columnar storage being 2.5× *larger* than the row store is the sort of number most benchmark posts omit.

### What I could not verify

The closing claim is that **replicas, `pg_dump` and MVCC "work as with heap" for both `mergetree` and `iheap`, "because PostgreSQL still takes care of them."** That is the author's assertion and the post offers no measurement behind it, which is the one place scepticism is warranted — and today's lesson is exactly why.

MVCC in PostgreSQL is **not** a service the core provides to a storage layer. It is encoded in bytes that `heap` owns: `t_xmin`, `t_xmax`, `t_infomask`'s twelve transaction bits, the hint bits, `t_ctid`'s forward pointer. Lesson 03 catalogued all sixteen `t_infomask` bits and found the word **full**. A table AM that does not store a `HeapTupleHeaderData` has to supply its own answer to "is this version visible to this snapshot", and `iheap`'s own design — old versions in an undo log in the main fork, index entries tombstoned through a `dmbtree` btree fork — is an admission of exactly that: it had to build a visibility mechanism, not inherit one. The post gives no equivalent account for `mergetree`. **Stated plainly: I did not verify the MVCC, replication or `pg_dump` claims, and the architecture of the thing makes them the claims most in need of a test.** The performance numbers stand on their own and do not depend on this.

The post also states no limits on writes, transactions or indexes for `mergetree`. For a `MergeTree` whose merges are asynchronous background work and whose parts are immutable, "no stated limits" is an absence of information rather than an absence of limits.

### Why this is the right item on this day

- **Today's lesson is the inventory of what `heap` assumes.** `t_hoff` is `MAXALIGN`ed so pointer alignment and offset alignment coincide; pad bytes are *required* to be zero so an unaligned 1-byte varlena header is decodable; the 18-byte TOAST pointer carries two OIDs and no type, so decoding needs `pg_class` and `pg_attribute`. None of that is PostgreSQL's contract with a table AM — all of it is `heap`'s private arrangement. Reading `USING mergetree` immediately after measuring those bytes is the clearest possible statement of where the seam actually is.
- **It makes lesson 35 concrete 31 lessons early.** The syllabus asks "what must a different table AM supply?" The answer, from someone who shipped one: its own visibility story, its own process model (the `"Another server instance"` error is PostgreSQL's connection model colliding with ClickHouse's), and — the part the interface does not even ask for — a matching executor, or it loses to `heap` by 3.6×.
- **It is the inverse of today's TOAST material.** TOAST exists because a row-store page cannot hold a wide value, so PostgreSQL chops one *column* out of a row and files it in 1996-byte chunks elsewhere. MergeTree's answer is to never assemble the row at all. The 11.8 GB-versus-4.8 GB anti-result is a reminder that the columnar answer is not free either — it just moves the cost from indirection to part management.

One more, pointing back: yesterday's deep dive was a default-on optimisation (`recovery_prefetch`) whose cost was invisible in bytes and obvious in IOPS. Today's is a storage format whose benefit is invisible through a row-at-a-time slot and obvious through a 1,024-row batch. Both are the same shape of error — the mechanism was fine, the *interface between it and everything else* was measured in the wrong unit.

## Sources

- [The Daily Diff front page](https://tdd.cat/) — served the **2026-10-22 edition, 42 stories** ("Thursday, October 22, 2026") with every item's title, summary and source link; the basis for the point-wise summary and the only endpoint that returned content this run
- [The Daily Diff archive](https://tdd.cat/archive/) — **2026-10-22 linked as "Latest"**, 116 editions total, the 23rd present as plain text and not linked; the basis for concluding no 2026-10-23 edition exists
- [`https://tdd.cat/2026-10-22.md`](https://tdd.cat/2026-10-22.md) — `client_error`, nothing retrieved; the **false negative** that falsified yesterday's permalink rule
- [`https://tdd.cat/2026-10-23.md`](https://tdd.cat/2026-10-23.md) — `client_error`, nothing retrieved
- [My PostgreSQL is faster than yours](https://igor-suhorukov.github.io/blog/2026/10/21/my-postgresql-is-faster-than-yours/) — the deep dive's substance: the three extensions and their repos, `USING mergetree` DDL with `order_by` and `storage = 'pages'`, the ~28-of-150 submodule static link, the `"Another server instance in same directory is already running"` collision, the 49.9 / 16.8 ms and ~39× ClickBench figures, every per-query number, the TPC-H SF1 (61.8 / 17.2 / 6.9 s) and TPC-DS (380.5 / 58.8 / 48.6 s) tables, the 1,024-row batches and function-OID kernel binding, the dictionary-index path for `LowCardinality`, the `T_TargetEntry` and −46403 `VecSort` bugs, the 11.8-vs-4.8 GB size result and its 35-parts explanation, and the unverified MVCC / replica / `pg_dump` claim
- Source links for items summarized above, as listed in the 2026-10-22 edition: [NVIDIA OpenShell](https://github.com/NVIDIA/OpenShell) · [faster map polygon triangulation](https://globus.software/blog/2026/10/16/cutting-map-polygons/) · [LLM formal verification at assembly level](https://www.anthropic.com/research/formal-verification) · [Fly.io Sprites block device](https://fly.io/blog/sprites-block-device/) · [Brotli in NGINX thread pools](https://m-compressors.github.io/2026/ngx-mbrotli-thread-pool-benchmarks/) · [object shapes vs hash tables](https://bernsteinbear.com/blog/object-shapes/) · [RL-trained 4B model flying a Roadster](https://skypilot.ai/blog/flying-roadster-rl) · [300 Redis nodes to 3](https://lavik.dev/en/blog/300-redis-nodes-to-3-lavik-nodes/) · [Lumabri split-weight inference](https://github.com/JustVugg/lumabri/blob/main/README.md) · [Turing-complete malware in PyTorch's backward pass](https://elonlit.com/scrivings/how-to-run-turing-complete-malware-in-pytorchs-backward-pass/) · [offline-protocol SDK](https://github.com/Offline-Protocol/offline-protocol-sdk) · [PHP OOM to code execution](https://github.com/cookiengineer/actual-php2shell) · [branching agent state with an effect log](https://github.com/clibequilibrium/forked) · [Gemini 4 Argon agents beat Minecraft](https://arxiv.org/abs/2610.22319) · [I don't like DELETE CASCADE](https://planetscale.com/blog/i-dont-like-delete-cascade) · [Cerbère agent taint tracking](https://github.com/chrismsmr-celcom/cerbere-AG) · [session-latency recommendation loops](https://heyyviv.github.io/blog/what-reel-you-will-get-next/) · [where did my freed memory go](https://yapfast.com/blog/where-did-my-freed-memory-go/) · [Anthropic builds its own source control](https://newsletter.pragmaticengineer.com/p/the-pulse-anthropic-builds-its-own) · [display-p3 canvas CPU fingerprinting](https://scrapfly.dev/posts/display-p3-canvas-cpu-architecture/) · [vLLM on Cloud TPUs](https://vllm.ai/blog/2026-10-20-vllm-torchtpu) · [ZXC decode-optimized compressor](https://github.com/hellobertrand/zxc/blob/main/README.md) · [Muse bot-challenge sub-agent](https://github.com/sneakers-the-rat/muse-skills/tree/main/topics/bot_challenge) · [next-concept modeling](https://github.com/HazyResearch/ncm) · [on-device Android GUI model](https://getopposable.com/blog/computer-use-on-the-phone) · [off-policy correction across mixed accelerators](https://modal.com/blog/toward-rl-on-mixed-precision-mixed-accelerator-samplers) · [GoRoute LLM gateway](https://github.com/ziuus/goroute) · [changed-columns audit triggers are slower](https://www.depesz.com/2026/10/16/old-tech-is-sometimes-better-a-case-study-based-on-blogpost-from-now-next/) · [Qdrant GHSA-frc7-44gm-99qq](https://github.com/qdrant/qdrant/security/advisories/GHSA-frc7-44gm-99qq) · [trillion-parameter models on Cerebras](https://www.cerebras.ai/blog/how-we-deploy-trillion-parameter-frontier-models-on-cerebras) · [dumpster, a GC in safe Rust](https://claytonwramsey.com/blog/dumpster/) · [Mercury 3 vs fast autoregressive models](https://zbruceli.org/blog/mercury-3-vs-the-fast-autoregressive-models/) · [whytime eBPF latency attribution](https://github.com/Arinerron/whytime) · [measured promotion gates](https://contextfirst.blog/superlite-philosophy-gates-decisions/) · [Dolittle activation explanations](https://github.com/plainsight-systems/dolittle) · [humans approve intent, machines prove conformance](https://jovidecroock.com/blog/gittifacts/) · [wake-sleep agent architectures](https://twitter.com/nikogrupen/status/2108226990792900876) · [JAX linear-attention kernels on Hopper](https://github.com/jla-org/jax-linear-attention) · [gVisor spinning queue gap](https://www.catchkill9.dev/posts/gvisor-spinning-queue-gap.html) · [Chromium rule of 2](https://chromium.googlesource.com/chromium/src/+/master/docs/security/rule-of-2.md) · [Postgres TABLESAMPLE](https://boringsql.com/posts/tablesample/)
