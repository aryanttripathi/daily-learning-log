# Daily Learning Log

A daily technical learning log that goes deep. Each entry targets the actual mechanics of a system (storage layouts, wire protocols, scheduler behaviour, failure modes) rather than a feature tour or a summary of marketing pages.

Every lesson is written to be *runnable*: it carries a hands-on exercise you can execute against a real system, at least one structural diagram of the mechanism under discussion, and a source list containing only links that were actually read while writing it.

## How This Works

- **One subject at a time.** The log follows a multi-week *track*: a sequential deep dive into a single database or systems subject. The track's syllabus and progress live in [`TRACK.md`](TRACK.md), which is the single source of truth for what has been taught and what comes next.
- **One lesson per day, each building on the last.** Every day takes the first unchecked lesson from the syllabus. Lessons assume everything earlier in the track is known and open with a *Where This Fits* section linking the previous lesson. Missed days are never backfilled; the track simply continues.
- **A separate Daily Diff digest every day.** Alongside the lesson, each day folder has a `daily-diff.md` covering an edition of [The Daily Diff](https://tdd.cat/): a point-wise summary plus a deep dive on the single most significant item. tdd.cat publishes editions about two days late, so a digest covers the newest edition not yet covered and says which one it is. When no new edition has published, the digest says so explicitly rather than inventing items.
- **Tracks end with a capstone.** When every lesson is checked, that day's document is a capstone assembling the whole system end to end. The subject moves to *Completed Subjects* in `TRACK.md`, and a new track begins.

## Current Track

**SQLite Internals**: Lesson 3 of 32 · started 2026-09-16 · [syllabus and progress →](TRACK.md)

## Format

Day folders live at `YYYY/MM/YYYY-MM-DD-<slug>/`:

- `README.md`: the track lesson. If a lesson is genuinely large, `README.md` becomes a short overview linking numbered sub-documents (`01-<subtopic>.md`, `02-<subtopic>.md`, …) in the same folder.
- `daily-diff.md`: that day's Daily Diff digest.

Every document starts with an `entry-meta` HTML comment (date, type, track, lesson, title, slug). The index below is generated from those blocks.

A lesson contains, in order:

1. Title and a one-line `date · track · lesson N of T` header
2. **Where This Fits**: previous lesson, what this adds, what it sets up
3. A point-wise breakdown of the real mechanics (headed sections, tables, code; no prose walls)
4. At least one structural Mermaid diagram
5. **Hands-On**: something to run that makes the internals visible, with what to look for
6. **Where This Breaks Down**: limits, costs, and tradeoffs
7. **Further Study**, 8. **Next Steps**, 9. **Sources** (only links verified that day), 10. **Takeaways**

Entries before 2026-09-16 predate the track model. They came from a five-slot topic rotation (Database Internals, System Design / Paper Analysis, Tech Blog Analysis, The Daily Diff Digest, Variety) and are kept in the index as history.

## Index

| Date | # | Lesson | Track | Daily Diff |
|------|---|--------|-------|------------|
| 2026-08-30 | — | [ClickHouse MergeTree Internals — Storage, Indexing, and Replication](2026/08/2026-08-30-clickhouse-mergetree-internals/README.md) | pre-track · Database Internals | — |
| 2026-09-03 | — | [Spanner — How TrueTime Turns Clock Uncertainty Into External Consistency](2026/09/2026-09-03-spanner-truetime-external-consistency/README.md) | pre-track · System Design / Paper Analysis | — |
| 2026-09-04 | — | [How Discord Moved Trillions of Messages From Cassandra to ScyllaDB](2026/09/2026-09-04-discord-cassandra-scylladb-migration/README.md) | pre-track · Tech Blog Analysis | — |
| 2026-09-05 | — | — | pre-track · The Daily Diff Digest | [The Daily Diff — Agentic AI, a Kernel io_uring Gotcha, and Meta's ZGateway Proxy for ZippyDB](2026/09/2026-09-05-daily-diff-zgateway-proxy/README.md) |
| 2026-09-06 | — | [C²KV — Making KV-Cache Reuse Compression-Aware and Position-Free](2026/09/2026-09-06-c2kv-composable-kv-cache-reuse/README.md) | pre-track · Variety | — |
| 2026-09-07 | — | [PostgreSQL MVCC Internals — Tuple Versions, Hint Bits, HOT Chains, and VACUUM](2026/09/2026-09-07-postgres-mvcc-vacuum-internals/README.md) | pre-track · Database Internals | — |
| 2026-09-08 | — | [Amazon Aurora — The Log Is the Database, and How It Avoids Consensus](2026/09/2026-09-08-amazon-aurora-log-is-the-database/README.md) | pre-track · System Design / Paper Analysis | — |
| 2026-09-10 | — | [Notion's Postgres Sharding — From Monolith to 480 Shards, Then a Zero-Downtime 3x Re-shard](2026/09/2026-09-10-notion-postgres-sharding/README.md) | pre-track · Tech Blog Analysis | — |
| 2026-09-12 | — | — | pre-track · The Daily Diff Digest | [The Daily Diff — AI Agents Accused of Hacking, a Navier-Stokes Proof, and ClickHouse's WalShadow Physical-WAL Replication](2026/09/2026-09-12-daily-diff-walshadow-physical-wal-replication/README.md) |
| 2026-09-14 | — | [Kubernetes Prow & Tide — How OWNERS Trees and Batch-Tested Merge Queues Actually Work](2026/09/2026-09-14-kubernetes-prow-tide-merge-automation/README.md) | pre-track · Variety | — |
| 2026-09-15 | — | [SQLite WAL Internals — Frame Checksums, Checkpoints, and the 16-Year Reset Race That Broke Tailscale](2026/09/2026-09-15-sqlite-wal-internals-reset-bug/README.md) | pre-track · Database Internals | — |
| 2026-09-16 | 01 | [The Database File Header and How lockBtree() Bootstraps Page 1](2026/09/2026-09-16-sqlite-file-header-page1-bootstrap/README.md) | SQLite | [digest (edition 2026-09-13)](2026/09/2026-09-16-sqlite-file-header-page1-bootstrap/daily-diff.md) |
| 2026-09-17 | 02 | [Varints, Serial Types, and the Record Format](2026/09/2026-09-17-sqlite-varints-serial-types-record-format/README.md) | SQLite | [digest (edition 2026-09-15)](2026/09/2026-09-17-sqlite-varints-serial-types-record-format/daily-diff.md) |
| 2026-09-18 | 03 | [The B-tree Page Header, Cell Pointer Array, and the Four Cell Layouts](2026/09/2026-09-18-sqlite-btree-page-header-cell-layouts/README.md) | SQLite | [digest (no new edition; 2026-09-15 revisited)](2026/09/2026-09-18-sqlite-btree-page-header-cell-layouts/daily-diff.md) |
