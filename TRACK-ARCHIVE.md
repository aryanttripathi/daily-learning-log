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
| 13 – 24 | SQLite Internals | commit `724c164b9a3edfddd6993c458cbfa87c0c521805` (2026-10-12, the last commit before the first trim) | `e3aa5a6715fee568a4dfdead028d3d7624ca883f` |
| 25 | SQLite Internals | commit `5429a52018a46573b419ac99bf80d8fb3b0777b9` (2026-10-13, the last commit before the second trim) | `29cf5381c8b006917033a062727d9e6ce68b2cfb` |
| 26 | SQLite Internals | commit `ad3839f8be496cef3cf2db47fd4866cc7733b04d` (2026-10-14, the last commit before the third trim) | `68c8bcb936ccc25efac2076b649494b22a755cc2` |
| 27 | SQLite Internals | commit `8d6880a7dace31a374afc865d1556b9e3593caa4` (2026-10-15, the last commit before the fourth trim) | `e5e40ca598fb43fd2f26d090bc2bb10f576f4d5b` |
| 28 | SQLite Internals | commit `815ea964987e5764d01f917e944ca8a7eb5e2851` (2026-10-16, the last commit before the fifth trim) | `b11a76fdd03a5ee9ede69de38ec2585b029d0efd` |
| 29 | SQLite Internals | commit `aea7ae911690ab140bb7036d5588d06369ac10f7` (2026-10-17, the last commit before the sixth trim) | `1cb2703f12208654808b5bebde2299d80fddb95a` |
| 30 | SQLite Internals | commit `99aeb34381108985d51952987e71f9be978f235f` (2026-10-18, the last commit before the seventh trim) | `6c348998957d33c83ab6a3bc80755743eeec223a` |

Notes on the rows:

- The **13 – 24** blob also contains the notes for lessons 25 and 26, which were retained in
  `TRACK.md` at the time of the first trim.
- The **25** blob contains the notes for lessons 25, 26 and 27. It is the more convenient place to
  read lesson 25's note, because the 13–24 blob is 68 KB.
- The **26** blob contains the notes for lessons 26, 27 and 28, and is the most convenient place to
  read lesson 26's note — the one on the `WhereLoop` cost model, the closed-form `SEARCH`/`SCAN`
  crossover, and the tie rule.
- The **27** blob contains the notes for lessons 27, 28 and 29, and is the most convenient place to
  read lesson 27's note — the one on `sqlite_stat1` arithmetic, the `sqlite_stat4` sample
  selection, and the `szTabRow` boundary where the scan-versus-index decision flips because of a
  column the query never mentions. It also carries the standing reminder that `WHERETRACE` needs
  **both** `SQLITE_DEBUG` and `SQLITE_ENABLE_WHERETRACE`, and that `sqlite3WhereTrace` is
  `SQLITE_PRIVATE`, so a harness must `#include "sqlite3.c"` to set it.
- The **28** blob contains the notes for lessons 28, 29 and 30, and is the most convenient place to
  read lesson 28's note — the one on the VDBE sorter: the PMA on-disk format decoded by hexdump, the
  `mxPmaSize` memory threshold and the spill boundary predicted to the single row, the 16-PMA cliff
  reachable by one page of `cache_size`, the `typeMask` fast-path gates, and `LIMIT -1` measuring
  2.74× slower than no `LIMIT` at all. Its unresolved sorter-mmap item was **closed by lesson 31**,
  whose note records the answer.
- The **29** blob contains the notes for lessons 29, 30 and 31, and is the most convenient place to
  read lesson 29's note — the one on savepoints and statement journals: a statement transaction
  *is* a savepoint at `db->nSavepoint + db->nStatement`; `WHERE_ONEPASS` shown to have **no
  effect** on statement journalling because `mayAbort` is never set for a `DELETE`; and the 64 KB
  spill threshold predicted in closed form and then hit to the single input row, 65,600 bytes in
  exactly two `xWrite` calls.
- The **30** blob contains the notes for lessons 30, 31 and 32, and is the most convenient place to
  read lesson 30's note — the one on virtual tables and `xBestIndex`: every output field's entry
  value read from inside the callback (`estimatedCost = 5e+98`, `estimatedRows = 25`, both reset on
  every call), the cost channel's `10·log₂` quantisation and the point above 2×10⁹ where
  `sqlite3LogEstFromDouble` stops computing a logarithm and reads the IEEE-754 exponent instead, the
  K+4 call-schedule bound, the `xDestroy == 0` trap that yields a misleading `no such module`, and
  the `sqlite_dbpage` truncation finding that **contradicts the documented** "any `ROLLBACK TO` will
  cancel the null-INSERT" rule, with the two differing statement sequences measured by file size.
- Lessons 01–12 predate the note block; those lessons have no handover notes, only their documents.

After the 2026-10-18 trim, `TRACK.md` holds the notes for lessons **31, 32 and 33** — the last
three lessons of the SQLite track, which completed on 2026-10-18.

## Procedure for the next trim

When `TRACK.md`'s note block holds more than three lessons' notes:

1. Note the current `TRACK.md` blob SHA and the current commit SHA **before** editing.
2. Remove the oldest notes from `TRACK.md`, keeping the three most recent.
3. Add a row here with the lesson range, that commit, and that blob SHA.

Never delete a note without adding its row here first.

**When a track ends and a new one begins**, the outgoing track's last three notes are the ones most
worth keeping readable: they carry the handoffs that the capstone has to close. Archive them as a
single range row (lessons 31–33 for SQLite) on the run that replaces the syllabus, rather than
trimming them one at a time into a syllabus they no longer describe.

## What this file is not

It is not the record of what was taught. **The lesson documents are**, and they are indexed in the
root [`README.md`](README.md). These notes are working memory between runs: a fresh session reads
them to avoid re-teaching something or contradicting an earlier day. If you want to know what
lesson 18 covered, read lesson 18; read its note only if you need to know what lesson 18 decided
*not* to cover and why.
