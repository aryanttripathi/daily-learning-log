<!--
registers
track: JanusGraph
upstream: JanusGraph/janusgraph
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
volume: 2 (candidates 65 on)
volume-1: tracks/janusgraph-registers.md (candidates 1-64, and the "Still open, and who owns it" ledger)
split-from: tracks/janusgraph-registers.md on 2026-11-22 (lesson 24)
last-updated: 2026-11-22 (lesson 24; candidates 65-67 appended, next free number 68)
-->

# JanusGraph track — PR candidates, volume 2

**Why a second volume.** [`tracks/janusgraph-registers.md`](janusgraph-registers.md) reached **134 KB**
at lesson 23. Every file this log writes has to be re-emitted whole to be pushed, so a 134 KB
append-only file costs a full re-emission to add three lines — which is the same problem that split the
registers out of `TRACK.md` at lesson 16, arriving one level down. Volume 1 is **frozen**: candidates
1–64 and the *"Still open, and who owns it"* ledger stay there, unedited, and remain the place to look
for any candidate numbered 64 or below. New candidates are appended here.

**Numbering is continuous across volumes.** Volume 1 ends at 64; this file starts at 65. Next free
number: **68**.

**How a run uses this file.** `TRACK.md` is the source of truth for what comes next. The registers are
the source of truth for what is still owed and to whom. A run reads this file when it is about to add a
candidate (for the next free number) or when its lesson is named as the owner of an open question
numbered 65 or above. A run that logs candidates must write this file in the **same push** as `TRACK.md`,
per the rule the 2026-11-19 desync produced.

---

## PR candidates, all verified at the pinned commit `a59ae8d8`

### 65 — `JanusGraphTraversalUtil.isLegalMultiQueryPosition` walks `currentStep` and tests `step`

**The only reportable defect of the lesson-24 batch.** File it on its own.

[`JanusGraphTraversalUtil.java:390-403`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/tinkerpop/optimize/JanusGraphTraversalUtil.java#L390-L403)
(blob-SHA verified, 22,204 bytes = `8290888d`, 431 lines):

```java
public static boolean isLegalMultiQueryPosition(Step<?, ?> step) {
    if (step.getTraversal().getTraverserRequirements().contains(TraverserRequirement.PATH)) return false;
    boolean labeledPath = false;
    Step currentStep = step.getTraversal().getStartStep();
    while (!currentStep.equals(step, true)) {
        if (step instanceof PathProcessor) {
            final Set<String> keepLabels = ((PathProcessor) step).getKeepLabels();
            labeledPath &= keepLabels == null || !keepLabels.isEmpty();
        }
        labeledPath |= !step.getLabels().isEmpty();
        currentStep = currentStep.getNextStep();
    }
    return !labeledPath;
}
```

The loop advances `currentStep` and the body reads the **method parameter** `step` three times, so the
body is loop-invariant and the walk computes nothing. Evaluating it: if `step` is the traversal's start
step the body never runs and the method returns `true` even when `step` carries a label; otherwise the
result is exactly `step.getLabels().isEmpty()`, independent of every other step.

The javadoc at `:382-388` says the method *"closely matches the behavior implemented in
`LazyBarrierStrategy`"*, which accumulates `labeledPath` over **every** step up to the position. The
`&=` keep-labels line is a faithful translation; only the variable is wrong. Effect: a
`JanusGraphMultiQueryStep` and a generated `JanusGraphNoOpBarrierVertexOnlyStep` are inserted at
positions `LazyBarrierStrategy` would decline, whenever a label appears earlier in the traversal and the
traversal's requirements include `LABELED_PATH` rather than `PATH` — the guard at `:391` only catches
full `PATH`, which is what `testNoMultiQueryStepsInsertedIfPathQuery` covers.

**Status: derived from reading, not from a run.** Lesson 24's Hands-On B is the experiment that would
confirm it (`g.V().hasLabel('leaf').as('a').out().select('a')` versus the same traversal on a
TinkerGraph). **Run it before filing.** Fix is one identifier plus a test; the test is the valuable half.

### 66 — `BatchProcessingQueue.setBatchSize` drops the clamp its constructor has

[`BatchProcessingQueue.java:40-42` and `:87-89`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/tinkerpop/optimize/step/fetcher/BatchProcessingQueue.java#L40-L89)
(blob-SHA verified, 2,693 bytes = `141dcb7e`):

```java
public BatchProcessingQueue(int batchSize) { this.batchSize = Math.max(batchSize, 1); }
...
public void setBatchSize(int batchSize) { this.batchSize = batchSize; }
```

With a non-positive size, `addToBatchToEnd`'s `lastBatch.size() >= batchSize` (`:48`) is true on every
call, so the first add pushes an **empty** list into `batches` and `removeFirst` returns an empty batch —
a multi-query carrying only `requiredFetchVertex`, i.e. batching silently off while still paying the
per-batch allocation.

Reachability: **`query.batch.limited-size` has no verifier.**
[`GraphDatabaseConfiguration.java:386-391`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/configuration/GraphDatabaseConfiguration.java#L386-L391)
ends `ConfigOption.Type.MASKABLE, 2500)` where `MAX_KEYS_PER_READ` two declarations below it (`:401`)
ends `ConfigOption.Type.MASKABLE, 1000, ConfigOption.positiveInt())`. `cache.tx-cache-size` (`:668-670`)
has none either. The value reaches `setBatchSize` through `JanusGraphMultiQueryStrategy:200`.

**Not run.** The claim is "the clamp is absent and the path is open", not "it misbehaves in production".
The better PR is `ConfigOption.positiveInt()` on both options — reject the nonsense at load time — with
the setter clamp as a belt-and-braces second commit.

Bundles cleanly with 67: same subsystem, both small, both want a test.

### 67 — `JanusGraphVertexStep`'s copy constructor does not copy `batchSize`

[`JanusGraphVertexStep.java:56-77`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/tinkerpop/optimize/step/JanusGraphVertexStep.java#L56-L77)
(blob-SHA verified, 8,152 bytes = `d328dff0`). The `originalStep instanceof JanusGraphVertexStep` arm
copies `useMultiQuery`, `hasContainers` and `limit` and **not** `batchSize`, so the copy starts at the
field default `Integer.MAX_VALUE` (`:54`); and because `setUseMultiQuery` is called first (`:62`), the
new `VertexStepBatchFetcher` is constructed with `Integer.MAX_VALUE` (`:74-76`).

**Latent: no path found where it changes behaviour.** The one reachable caller traced is
`JanusGraphLocalQueryOptimizerStrategy.inspectLocalTraversals`, which constructs the copy and then calls
`setBatchSize(txVertexCacheSize)` at `:170`, repairing it immediately. File as code health: copy the
field, add a test that a re-wrapped step keeps its batch size.

**Related, noted and deliberately not filed:** `MultiQueriableStepBatchFetcher.setBatchSize` (`:93-97`)
updates the current- and next-loop queues but not `firstLoopBatchProcessingQueue`. That queue is created
lazily at `:77-79` from the field and strategy application finishes before any traverser runs, so it is
always null at that point. Harmless as written, fragile if anything ever resizes mid-run. Also:
`BatchProcessingQueue.removeLast()` (`:67-77`) has no caller among the files read at lesson 24 — but
`HasStepBatchFetcher` and `DropStepBatchFetcher` were **not** read, so that is not a dead-code claim and
must not be filed as one.

---

## Filing guidance for this batch

- **65 goes up alone**, after Hands-On B has been run, with the `explain()` output of both traversals
  attached and the TinkerGraph comparison in the issue body. It is the only one of the three that is a
  behaviour difference rather than a robustness gap.
- **66 and 67 bundle** as one small PR against the batching subsystem, or as two commits in one PR. The
  test is the deliverable in both cases: lesson 24 established that **nothing in the repository asserts a
  numeric batch size**, which is why both of these could exist unnoticed.
- All three are `good first issue` shaped. If the intent is an LFX Mentorship application, 65 is the one
  worth a careful write-up: it is a reading of TinkerPop's `LazyBarrierStrategy` against JanusGraph's
  stated intention to match it, which is exactly the kind of evidence a maintainer can check in a minute.
