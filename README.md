# Daily Learning Log

A daily deep-dive technical learning log. Each entry targets the actual mechanics of a system — storage layouts, wire protocols, scheduler behaviour, failure modes — rather than a feature tour or a summary of marketing pages.

Every entry is written to be *runnable*: each document carries a hands-on exercise you can execute against a real system, at least one structural diagram of the mechanism under discussion, and a source list containing only links that were actually read while writing it.

## Format

Entries live at `YYYY/MM/YYYY-MM-DD-<slug>/README.md`.

When a day's topic has multiple substantial sub-topics, the day's `README.md` becomes a short overview linking to numbered sub-documents (`01-<subtopic>.md`, `02-<subtopic>.md`, …) in the same folder, each a complete document in its own right.

Every document — overview or sub-document — contains, in order:

1. Title and a one-line date/category header
2. A point-wise breakdown of the real mechanics (headed sections with bullets, not prose walls)
3. At least one structural Mermaid diagram — architecture, flow, sequence, or state machine
4. A concrete hands-on exercise: something to actually run and inspect
5. **Further Study** — curated reading beyond the primary sources
6. **Next Steps** — concrete follow-up implementation or learning work
7. **Sources** — only real links verified during that entry's research
8. A short takeaways section

## Rotation

Topics follow a fixed five-slot cycle. The slot for a new entry is `(number of existing day-entries) mod 5`.

| Slot | Category | Scope |
|------|----------|-------|
| 0 | **Database Internals** | Real storage engines, replication, indexing internals — ClickHouse, Redis, Postgres and similar. Mechanics, not feature tours. |
| 1 | **System Design / Paper Analysis** | A system design paper or a real engineering case study, analysed rather than summarised. |
| 2 | **Tech Blog Analysis** | A substantial published engineering post (Netflix, Uber, Cloudflare, Stripe tier) or a significant open-source system's internals, genuinely dissected. |
| 3 | **The Daily Diff Digest** | That day's edition of [The Daily Diff](https://tdd.cat/), summarised point-wise, with a deep dive on the single most significant item. |
| 4 | **Variety** | Occasional slot — an AI/ML research paper analysis, an open-source contribution guide, or a system design / database topic not recently covered. |

## Index

| Date | Title | Category |
|------|-------|----------|
| 2026-08-30 | [ClickHouse MergeTree Internals — Storage, Indexing, and Replication](2026/08/2026-08-30-clickhouse-mergetree-internals/README.md) | Database Internals |
| 2026-09-03 | [Spanner — How TrueTime Turns Clock Uncertainty Into External Consistency](2026/09/2026-09-03-spanner-truetime-external-consistency/README.md) | System Design / Paper Analysis |
| 2026-09-04 | [How Discord Moved Trillions of Messages From Cassandra to ScyllaDB](2026/09/2026-09-04-discord-cassandra-scylladb-migration/README.md) | Tech Blog Analysis |
| 2026-09-05 | [The Daily Diff — Agentic AI, a Kernel io_uring Gotcha, and Meta's ZGateway Proxy for ZippyDB](2026/09/2026-09-05-daily-diff-zgateway-proxy/README.md) | The Daily Diff Digest |
