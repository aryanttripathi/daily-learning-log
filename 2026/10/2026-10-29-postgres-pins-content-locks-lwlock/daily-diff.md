<!--
entry-meta
date: 2026-10-29
type: daily-diff
category: Daily Diff
title: Daily Diff — 2026-10-29
slug: postgres-pins-content-locks-lwlock
-->

# Daily Diff — 2026-10-29

**2026-10-29 · Daily Diff Digest**

## No edition could be digested this run

This is a short document on purpose. The run could not obtain an edition it had not already covered.
Nothing below is inferred and nothing is invented.

What was actually attempted, at 2026-10-29 03:50–03:55 UTC / 09:20–09:25 IST:

| Fetch | Result |
|---|---|
| [https://tdd.cat/](https://tdd.cat/) (front page) | Served the **2026-10-27** edition, 38 stories — the edition [yesterday's digest](../2026-10-28-postgres-buffer-manager-clock-sweep/daily-diff.md) already covered in full. |
| [https://tdd.cat/archive/](https://tdd.cat/archive/) | Loaded. Lists 122 editions and marks **2026-10-28 (57 stories)** as "Latest". The newest entry overall is 2026-10-28; **there is no 2026-10-29 entry.** The archive's own href for the Latest entry is `https://tdd.cat/2026-10-28/`, with a trailing slash. |
| [https://tdd.cat/2026-10-28/](https://tdd.cat/2026-10-28/) | `client_error` — nothing retrieved. Tried three times, including without the trailing slash. |
| [https://tdd.cat/2026-10-29/](https://tdd.cat/2026-10-29/) | `client_error` — nothing retrieved. |

So:

- **The 2026-10-29 edition has not published.** The archive does not list that date at all.
- **The 2026-10-28 edition exists** — the archive names it as Latest and gives its story count as 57
  — **but its page could not be read this run**, and the front page had not yet rolled forward onto
  it.
- The only edition reachable this run was 2026-10-27, which the previous run digested item by item.
  Re-digesting it would add nothing and would misdate the entry.

No deep dive, and no item list: a deep dive requires following a source link from an edition this run
actually read, and there was none. Per this log's own rule, inventing or reconstructing items from
memory is not an option.

## What to do about it

The failure mode is specific and worth recording, because it has now happened on two consecutive
runs with different shapes. Yesterday the day's own URL 404'd while the archive had already rolled
forward, so the run recovered by fetching the previous day from the archive. Today the archive has
rolled forward to 2026-10-28 but that edition's **page** errors, while the front page still serves
2026-10-27. The site's three surfaces — front page, archive index, and per-date page — are not
updating in lockstep, and the per-date page is the one that fails.

The practical fix for the next run is to try, in order: the day's own date page, the archive index
for the newest date **not already present in this repository's day folders**, and then that date's
page — and to treat a `client_error` on a date the archive lists as Latest as "retry next run",
exactly as recorded above, rather than as "no edition exists". The 2026-10-28 edition (57 stories)
is the one owed; if it becomes readable, it is the obvious candidate for the next run to cover
alongside its own day.

## Sources

- [https://tdd.cat/](https://tdd.cat/) — front page, serving the 2026-10-27 edition (38 stories) at run time.
- [https://tdd.cat/archive/](https://tdd.cat/archive/) — archive index; 122 editions listed, 2026-10-28 (57 stories) marked "Latest", no 2026-10-29 entry.
- [https://tdd.cat/2026-10-28/](https://tdd.cat/2026-10-28/) — the archive's own link for the Latest edition; returned `client_error` on every attempt this run.
- [https://tdd.cat/2026-10-29/](https://tdd.cat/2026-10-29/) — returned `client_error`; the date is absent from the archive.
