<!--
entry-meta
date: 2026-11-13
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-11-13
slug: janusgraph-tx-write-path-containers
-->

# Daily Diff digest

**2026-11-13 · Daily Diff Digest**

**Edition covered: [2026-11-12](https://tdd.cat/2026-11-12/), 66 stories.** A procedural note, since it changes the standing instruction: the front page of [tdd.cat](https://tdd.cat/) and [the archive](https://tdd.cat/archive/) *both* reported 2026-11-11 as newest (58 stories, 136 editions) when fetched at ~03:40 UTC today — but `https://tdd.cat/2026-11-12/` **exists and is complete**, with 66 stories. So the front page and the archive can agree with each other and still both be a day behind the published edition. The max-of-two rule from the previous three runs is no longer sufficient; the direct dated URL has to be probed as well. The 2026-11-11 edition was already covered by lesson 14, so 2026-11-12 is today's.

---

## The edition, point-wise

**Database internals and storage** — the strongest cluster today, and unusually so:

- **PostgreSQL security releases 18.7 / 17.12 / 16.16 / 15.20 / 14.25, plus 19 RC 2** — [announcement](https://www.postgresql.org/about/news/postgresql-187-1712-1616-1520-1425-and-19-rc-2-released-3442/). **52 CVEs** and over 140 bugs. 14.25 is the final 14 release; PostgreSQL 14 is now end-of-life. The deep dive below.
- **[Turning a 22-year-old PostgreSQL type-confusion primitive into RCE (CVE-2026-6460)](https://mehmetince.net/part-3-6-how-i-ended-up-turning-a-22-year-old-postgresql-type-confusion-primitive-into-rce-cve-2026-6460/)** — the research writeup behind the headline CVE of that release. Also the deep dive.
- **[Sharding Postgres scales durable workflows to one million per second](https://www.dbos.dev/blog/one-million-workflows)** (DBOS) — durable-execution state in Postgres rather than a bespoke log.
- **[Fast search joins eliminate the need to denormalize schemas](https://www.paradedb.com/blog/joins)** (ParadeDB) — pushing joins into the search path; this one also appeared in the 2026-11-11 edition.
- **[Lakebase uses the lake as primary storage without replication](https://www.databricks.com/blog/dispelling-ltap-myths)** — the LTAP ("lakehouse + transactional") argument against maintaining a separate OLTP copy.
- **[Explicit shard placement solves load skew in distributed clusters](https://www.uber.com/us/en/blog/introducing-shard-manager/)** — Uber's shard manager; relevant to anything with a partition-bits scheme.
- **[Durable execution reaches the speed of raw disk access](https://www.bitsinfra.com/fusee)**; **[Zpq improves analytical query throughput via cache-aware aggregation](https://github.com/chicagobuss/zpq/releases/tag/v0.5.2)**; **[Tparquet halves time series file sizes using pcodec](https://github.com/aperiodic-io/tparquet/)**; **[Relate embeds a portable Cypher graph engine anywhere](https://github.com/relatedb/relate)** — the last one is directly adjacent to this track's subject.
- **[Replacing cloud vendor SDKs with Apache OpenDAL in databases](https://opendal.apache.org/blog/2026/11/11/scopedb-on-opendal/)** — storage-abstraction-layer work; the same shape of problem as JanusGraph's `KeyColumnValueStoreManager` SPI.
- **[KiviDB achieves high throughput as a multithreaded Redis alternative](https://github.com/kividbio/kividb/blob/master/README.md)**.

**Security, beyond Postgres:**

- **[JSON schema references introduce external entity expansion vulnerabilities](https://www.svix.com/blog/json-schema-external-entities/)** — XXE's structural analogue in `$ref`.
- **[Differing URL parsers universally fail security and interoperability](https://mblunt.dev/papers/parsers-of-babel.pdf)** — a paper on parser differentials.
- **[Mismatched permission fields in Nvidia drivers grant host root access](https://stealthium.io/blog/nvroot)**; **[Public email inboxes allow hijacking dormant GitHub accounts](https://shmulc.substack.com/p/click-fraud-bot-github-public-inbox-accounts)**; **[GitHub Actions attack leverages shared infrastructure and stolen tokens](https://research.veryserious.systems/w-hat-the-fx-infrastructure-re-use-on-the-biggest-github-action-attack-ever/)**; **[Attackers exploit Qubole vendor access to compromise customer environments](https://www.wiz.io/blog/qubole-cross-account-access-abused-in-the-wild)**.

**Verification and correctness** — a notably heavy day:

- **[Extending concurrent separation logic to verify the xv6 kernel](https://arxiv.org/abs/2609.04043)**; **[Cedar symbolic compiler verifies policy properties beyond standard tests](https://dvlog.dev/posts/cedar-symbolic-compiler-proofs/)**; **[Formal verification catches edge-case bugs that tests miss](https://read.thecoder.cafe/p/barista-2)**; **[Verifying correctness in AI-generated Lean proofs requires systematic testing](https://xenaproject.wordpress.com/2026/11/12/and-the-proof-is-checked-in-lean/)**; **[Treating tests as data protects contracts with AI agents](https://martinfowler.com/articles/tests-as-data.html)**; **[Quality ratchets miss critical silent failures in agentic software delivery](https://zenodo.org/records/24169186)**.

**Runtime, compilers, systems:**

- **[Retrofitting CPython with yk delivers meaningful performance gains](https://tratt.net/laurie/blog/2026/adapting_cpython_for_yk.html)** — meta-tracing JIT work on CPython.
- **[GCC optimizations hide hardcoded path strings from binary audits](https://stoppels.ch/2026/11/12/paths-in-pieces.html)** — string splitting defeating `strings`-based auditing.
- **[Using exponential decay snapshots to enable time travel debugging](http://chunkofcoal.com/posts/exponential-decay/)**; **[Dynamic debugging enables inspecting optimized builds without performance loss](https://devblogs.microsoft.com/cppblog/c-dynamic-debugging-is-now-generally-available/)**; **[Compiling Erlang into standalone native executables with Rust](https://www.process-one.net/blog/nera-erlang-that-ships-like-go/)**; **[Checkpointing Kubernetes controllers eliminates costly cold start bottlenecks](https://developer.confluent.io/blog/controller-checkpointing/)**; **[Fitting four 32 GB resizable BARs behind a PCIe switch](https://doug.sh/posts/gpu-bars-behind-a-pcie-switch/)**.

**Local inference and agents** — the usual large block, lower signal for this log: offloading routed experts to run 800B models ([InferSSD](https://wici.ai/article/running-800b-models-with-inferssd)), streaming weights on tiny devices ([smolmaxxing](https://github.com/hyperionaidev/smolmaxxing/)), [isolated interpreters vs. VMs for agent code](https://pydantic.dev/logfire/agent-sandboxes), [MCP servers vs. CLIs on context tokens](https://openrouter.ai/blog/insights/mcp-vs-cli-vs-skills/), [worse group outcomes when users delegate to individual agents](https://alignment.anthropic.com/2026/worse-together/).

---

## Deep dive: 52 CVEs, one umbrella, and a type system that never versioned its values

Followed the source links from both the announcement and the research writeup, then the [CVE-2026-6460 advisory page](https://www.postgresql.org/support/security/CVE-2026-6460/) itself.

### What shipped

On 2026-11-12 PostgreSQL released **18.7, 17.12, 16.16, 15.20, 14.25 and 19 RC 2**, fixing **52 CVEs** and 140-plus bugs. 14.25 is PostgreSQL 14's last release. The announcement's post-update section requires reindexing GIN, `pg_trgm` GiST and `bpchar` B-tree indexes, and a fresh full backup before resuming incremental backups — so this is not a drop-in binary swap.

The headline, CVE-2026-6460, carries CVSS 3.0 **8.8 (High)**, vector `AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H`. Component: core server. **PR:L** means "an object creator" — anyone who can run DDL. Affected: everything before the five fixed versions.

### The actual mechanism

The advisory names two root causes, and they are the whole story:

> *The system inadequately tracked typed values.* / *Values were not self-identifying.*

In other words: a datum in PostgreSQL is bytes plus a type *looked up elsewhere*. The bytes do not say what they are. Everything downstream trusts the catalog's current answer.

The research writeup traces one concrete instance. A composite type's attribute is declared `uuid`; a prepared statement builds a value of that composite; `ALTER TYPE ... ALTER ATTRIBUTE ... TYPE ... CASCADE` then changes the attribute to `bytea` **between `PREPARE` and `EXECUTE`**. Per the writeup, `ALTER TYPE` "changed the current definition... but it did not convert or rewrite the binary value." So 16 bytes written as a UUID are now read out as a `bytea`. The named call chain:

1. `record_out()` formats the composite and chooses the output function from the attribute's **current** type.
2. `byteaout()` fetches the datum and detoasts it.
3. `detoast_external_attr()` reads the leading external-marker byte pattern and interprets it as an expanded-object reference.
4. `DatumGetEOHP()` casts that reference to an `ExpandedObjectHeader *`.
5. `EOH_get_flat_size()` calls through `eohptr->eoh_methods` — a function-pointer dereference on a pointer derived from attacker-chosen UUID bytes.

That last step is the primitive: **16 bytes the attacker fully controls become a pointer that is dereferenced for a method table.** The writeup dates the two halves separately — the expanded-object mechanism to commit `1dc5ebc9077a` (2015), the mutable-attribute behaviour to `077db40fa1f3` (2004) — which is where "22-year-old" comes from. Neither commit is wrong on its own; the bug is their composition.

The writeup is explicit that the crash reproduces "as a lowest-privileged login role", and equally explicit that it does not publish the full chain from NULL-pointer crash to command execution (it needs pointer control plus an address leak to defeat ASLR). The advisory publishes no proof-of-concept either. Both are the right call; the mechanism is the interesting part.

### Why it is 52 CVEs and not one

CVE-2026-6460 is described in the writeup as the **umbrella fix**, introducing *composite attribute versioning* — making values self-identifying enough that a stale one is detected rather than reinterpreted. The writeup says 21 variants were reported. The release's CVE list bears that out: filter it for the same failure mode and the pattern is unmistakable.

**Type confusion, each its own CVE at 8.8:**

| CVE | Where the confusion happens |
|---|---|
| CVE-2026-6460 | after `ALTER ... SET DATA TYPE` (the umbrella) |
| CVE-2026-18447 | output-parameter types |
| CVE-2026-116306 | composite-type change during field assignment |
| CVE-2026-116309 | `plperl` blessed references |
| CVE-2026-116329 | statistics import |
| CVE-2026-116331 | min/max aggregates |
| CVE-2026-116335 | commutator / negator result types |
| CVE-2026-120141 | materialized set-returning functions |
| CVE-2026-120142 | dropped columns |
| CVE-2026-120144 | type-cache invalidation gap |
| CVE-2026-120148 | extended-statistics pseudo-types (8.1) |
| CVE-2026-120151 | `plpgsql` record fields |

**Use-after-free, same severity band:** CVE-2026-116304 (parallel `RECORD` typmod), -116307 / -116319 / -116320 / -116326 (four separate `plperl` paths), -116310 (`COPY` error context), -116323 (`pltcl` reentrant result conversion), -116332 (portal setup), -120143 (transition-table cursor), -120147 (`dblink` reentrant disconnect), -120150 (expanded-record domain check), -120152 (`pg_show_all_settings()`, disclosure only at 4.3).

**Integer wraparound:** CVE-2026-116328 (generation memory context), CVE-2026-120139 (tuple size). **Heap overflow:** CVE-2026-116315 (`tsvector_delete()`), CVE-2026-116317 (`ts_rank_cd()`). **Injection in tooling:** CVE-2026-116318 (`pg_dump` not-null constraint names), -116330 (`pg_dump` tar `restore.sql`), -116325 (`reindexdb` index-name quoting), -120153 (extension-script substitutions), CVE-2026-6467 (extension-script schema handling).

Count the type-confusion and procedural-language rows and you have most of the 8.8s. **The procedural languages are where the blast radius concentrates** — `plperl` alone accounts for five distinct CVEs. Every one of them is an interface between PostgreSQL's non-self-identifying datums and another runtime's object model, which is precisely where "the bytes do not say what they are" stops being survivable.

```mermaid
flowchart LR
    subgraph root["Two root causes (per the advisory)"]
        R1["Values are not self-identifying<br/>bytes carry no type tag"]
        R2["Typed values inadequately tracked<br/>stale values persist in catalog entries,<br/>PL/pgSQL variables, caches"]
    end

    subgraph trigger["DDL that changes a type under a live value"]
        T1["ALTER ... SET DATA TYPE"]
        T2["ALTER TYPE ... ALTER ATTRIBUTE ... CASCADE<br/>between PREPARE and EXECUTE"]
        T3["ADD COLUMN; ROLLBACK; ADD COLUMN"]
    end

    subgraph chain["The primitive, as traced in the writeup"]
        C1["record_out()<br/>picks output fn from the CURRENT type"]
        C2["byteaout()<br/>fetch + detoast"]
        C3["detoast_external_attr()<br/>leading byte read as external marker"]
        C4["DatumGetEOHP()<br/>→ ExpandedObjectHeader *"]
        C5["EOH_get_flat_size()<br/>calls eohptr->eoh_methods<br/>— attacker-controlled fn ptr"]
    end

    subgraph classes["52 CVEs, grouped by failure mode"]
        K1["type confusion ×12<br/>6460, 18447, 116306, 116309,<br/>116329, 116331, 116335,<br/>120141, 120142, 120144,<br/>120148, 120151"]
        K2["use-after-free ×12<br/>incl. 5× plperl, 1× pltcl"]
        K3["integer wraparound ×2<br/>116328, 120139"]
        K4["heap overflow ×2<br/>116315, 116317"]
        K5["tooling injection ×5<br/>116318, 116330, 116325,<br/>120153, 6467"]
        K6["privilege / RLS / disclosure<br/>the remaining lower-CVSS rows"]
    end

    FIX["Fix: composite attribute versioning<br/>18.7 / 17.12 / 16.16 / 15.20 / 14.25<br/>CVE-2026-6460 is the umbrella"]

    R1 --> C3
    R2 --> C1
    T1 --> C1
    T2 --> C1
    T3 --> C1
    C1 --> C2 --> C3 --> C4 --> C5
    C5 --> K1
    R1 --> K1
    R1 --> K3
    R1 --> K4
    R2 --> K2
    R2 --> K6
    T1 --> K5
    K1 --> FIX
    K2 --> FIX
    K3 --> FIX
    K4 --> FIX
```

### Why it matters, and what it changes

**Operationally**, this is a patch-now release with a non-trivial update procedure (the GIN / `pg_trgm` GiST / `bpchar` B-tree reindexes, and a new base backup). Until patched, the only real mitigation the advisory offers is narrowing who can run DDL — which is unsatisfying because in most shops application roles own their schemas, so PR:L is satisfied by the application itself.

**Architecturally**, it is a cleaner statement of a problem that shows up in every database that separates values from their types, and it lands pointedly in the middle of this track. JanusGraph's answer to the same question is different in a way worth stating precisely:

- **JanusGraph tags the value with a registration number.** Lesson 12 established that `StandardSerializer.writeClassAndObject` writes a `VariableLong` registration prefix ahead of the payload, and lesson 14 established that all 13 accepted config datatypes are below 128 so the prefix is one byte. A JanusGraph value *is* self-identifying.
- **But its schema metadata is not versioned**, and lesson 13 found the consequence: `EnumSerializer.write` is `VariableLong.writePositive(ordinal)` with no version marker, and `TypeDefinitionMap.setValue`'s type check is an `assert` — disabled in production. Insert a constant mid-list in `TypeDefinitionCategory` and stored schema is silently reinterpreted. That is PostgreSQL's "values were not self-identifying" failure mode, relocated from data to schema.
- **And lesson 15 (today's entry) adds the transaction-level version**: `verifyAttribute` (`StandardJanusGraphTx:735-759`) normalises and validates a property value against `key.dataType()` **once**, at write time, and everything downstream trusts that. A JanusGraph property key's data type is `GLOBAL_OFFLINE` and changing it requires every other instance shut down (lesson 14's `configVerifier` finding), which is a heavier-handed answer than versioning — but it is an answer, and it is the reason the `ALTER`-under-a-live-value shape is not reachable the same way.

The transferable lesson is the one the advisory states in six words: **values that are not self-identifying are a type-confusion surface, and tracking them "adequately" out-of-band does not scale to 52 call sites.** PostgreSQL's fix is to make composite attributes versioned. The cheaper design is to tag the bytes — which JanusGraph does for data and does not do for schema ordinals. PR candidate 25 in `TRACK.md` is exactly that gap.

## Sources

- [The Daily Diff — 2026-11-12 edition](https://tdd.cat/2026-11-12/) (66 stories)
- [tdd.cat front page](https://tdd.cat/) and [archive](https://tdd.cat/archive/) — both reporting 2026-11-11 as newest at fetch time; recorded for the procedural note above
- [PostgreSQL 18.7, 17.12, 16.16, 15.20, 14.25 and 19 RC 2 released](https://www.postgresql.org/about/news/postgresql-187-1712-1616-1520-1425-and-19-rc-2-released-3442/)
- [CVE-2026-6460 advisory](https://www.postgresql.org/support/security/CVE-2026-6460/)
- [How I ended up turning a 22-year-old PostgreSQL type-confusion primitive into RCE (CVE-2026-6460)](https://mehmetince.net/part-3-6-how-i-ended-up-turning-a-22-year-old-postgresql-type-confusion-primitive-into-rce-cve-2026-6460/)
