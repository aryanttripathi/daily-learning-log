# TRACK-ARCHIVE.md

Where to find the per-lesson scope and handover notes that have rolled off the end of
[`TRACK.md`](TRACK.md).

## Why this file exists

`TRACK.md` carries a comment block of per-lesson notes: what each lesson took, what it
deliberately left for a later lesson, which source it read, which measurements reproduced and
which did not. On 2026-10-12 that block had reached **68 KB**, and because the GitHub API can
only replace a file whole, every daily run had to rewrite all 68 KB in order to tick one
checkbox and append one note.

So `TRACK.md` now keeps the notes for the **three most recent lessons** only. Older notes are
**not deleted and not duplicated** — they are already stored permanently and immutably in this
repository's git history, which is what git is for. This file is the index into that history.

## How to read an archived note

Fetch `TRACK.md` at the commit or blob listed below and read its comment block. With the GitHub
MCP tools:

```
get_file_contents(owner="aryanttripathi", repo="daily-learning-log",
                  path="TRACK.md", sha="<commit sha from the table>")
```

The `sha` argument accepts a commit SHA and returns the file as it stood at that commit. The blob
SHA in the last column is the stronger pointer: it identifies the exact bytes regardless of which
commit you reach them through.

## Archived ranges

| Lessons | Subject | Where the notes live | `TRACK.md` blob SHA at that commit |
|---|---|---|---|
| 13 – 24 | SQLite Internals | commit `724c164b9a3edfddd6993c458cbfa87c0c521805` (2026-10-12, the last commit before the trim) | `e3aa5a6715fee568a4dfdead028d3d7624ca883f` |

That blob also contains the notes for lessons 25 and 26, which were retained in `TRACK.md` at the
time of the trim. Lessons 01–12 predate the note block; those lessons have no handover notes, only
their documents.

## Procedure for the next trim

When `TRACK.md`'s note block holds more than three lessons' notes:

1. Note the current `TRACK.md` blob SHA and the current commit SHA **before** editing.
2. Remove the oldest notes from `TRACK.md`, keeping the three most recent.
3. Add a row here with the lesson range, that commit, and that blob SHA.

Never delete a note without adding its row here first.

## What this file is not

It is not the record of what was taught. **The lesson documents are**, and they are indexed in the
root [`README.md`](README.md). These notes are working memory between runs: a fresh session reads
them to avoid re-teaching something or contradicting an earlier day. If you want to know what
lesson 18 covered, read lesson 18; read its note only if you need to know what lesson 18 decided
*not* to cover and why.
