<!--
registers
track: JanusGraph
upstream: JanusGraph/janusgraph
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
volume: 2 (candidates 65 on)
volume-1: tracks/janusgraph-registers.md (candidates 1-64, and the "Still open, and who owns it" ledger)
split-from: tracks/janusgraph-registers.md on 2026-11-22 (lesson 24)
last-updated: 2026-11-23 (lesson 25; candidates 68-70 appended, next free number 71)
-->

# JanusGraph track — PR candidates, volume 2

**Why a second volume.** [`tracks/janusgraph-registers.md`](janusgraph-registers.md) reached **134 KB**
at lesson 23. Every file this log writes has to be re-emitted whole to be pushed, so a 134 KB
append-only file costs a full re-emission to add three lines — which is the same problem that split the
registers out of `TRACK.md` at lesson 16, arriving one level down. Volume 1 is **frozen**: candidates
1–64 and the *"Still open, and who owns it"* ledger stay there, unedited, and remain the place to look
for any candidate numbered 64 or below. New candidates are appended here.

**Numbering is continuous across volumes.** Volume 1 ends at 64; this file starts at 65. Next free
number: **71**.

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

## Filing guidance, lesson 24 batch (65–67)

- **65 goes up alone**, after Hands-On B has been run, with the `explain()` output of both traversals
  attached and the TinkerGraph comparison in the issue body. It is the only one of the three that is a
  behaviour difference rather than a robustness gap.
- **66 and 67 bundle** as one small PR against the batching subsystem, or as two commits in one PR. The
  test is the deliverable in both cases: lesson 24 established that **nothing in the repository asserts a
  numeric batch size**, which is why both of these could exist unnoticed.
- All three are `good first issue` shaped. If the intent is an LFX Mentorship application, 65 is the one
  worth a careful write-up: it is a reading of TinkerPop's `LazyBarrierStrategy` against JanusGraph's
  stated intention to match it, which is exactly the kind of evidence a maintainer can check in a minute.

---

## PR candidates from lesson 25, verified at the pinned commit `a59ae8d8`

### 68 — `updateIndex`'s two `return null` branches are unreachable, because `isApplicableStatus` throws

[`SchemaAction.java`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/core/schema/SchemaAction.java) (read inline, blob `b2bf7951`):

```java
public boolean isApplicableStatus(SchemaStatus status) {
    if (!applicableStatuses.contains(status))
        throw new IllegalArgumentException(String.format("Update action [%s] cannot be invoked for index with status [%s]", this, status));
    return true;
}
```

It is not a predicate: it returns `true` or it throws. Both call sites are written as if it were —
[`ManagementSystem.java:987-996`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/management/ManagementSystem.java#L987-L996)
(blob-SHA verified, 86,864 bytes = `a722a7d9`, 1,815 lines):

```java
if (index instanceof RelationTypeIndex) {
    dependentTypes = Collections.singleton(...);
    if (!updateAction.isApplicableStatus(schemaVertex.getStatus()))
        return null;                                   // unreachable
} else if (index instanceof JanusGraphIndex) {
    ...
    if (indexType.isCompositeIndex()) {
        if (!updateAction.isApplicableStatus(schemaVertex.getStatus()))
            return null;                               // unreachable
```

`!true` is `false`, so neither `return null` can execute. Runtime behaviour is correct — the caller gets
an `IllegalArgumentException` — but the source documents a contract the method does not have, and
`updateIndex` returns a `ScanJobFuture` that callers such as `IndexIdentifier.getIndexJobFinisher` and
every `mgmt.updateIndex(...).get()` in the docs would dereference. The mixed-index branch does **not** have
the dead pair: it uses `Preconditions.checkArgument(!keySubset.isEmpty(), …)` at `:1013-1016` instead,
which is the right shape.

**Not run; the claim is static and complete** — `isApplicableStatus` has no other implementation and no
override. Two possible PRs, and the choice is the interesting part of the write-up: delete the dead
branches and let the throw be the documented contract (small, obviously safe), or make
`isApplicableStatus` a pure predicate and have `updateIndex` throw at one place with a better message
(slightly larger, removes the double negative). **Prefer the first**, because the second changes a public
method's behaviour on a `core.schema` enum and someone outside the repository may be calling it.

Pairs naturally with the `IndexRepairJob.validateIndexStatus` javadoc, which says *"Check that our target
index is in either the ENABLED or REGISTERED state"* while the set it actually tests is
`SchemaAction.REINDEX.getApplicableStatus()` = `{REGISTERED, ENABLED, DISABLED, WRITE_ONLY_ENABLED}`. One
PR, two trivially verifiable corrections, both in the index-lifecycle area.

### 69 — a mixed index's per-field `SchemaStatus` is a positional array slot guarded only by an `assert`

[`ManagementSystem.java:1343-1361`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/management/ManagementSystem.java#L1343-L1361):

```java
Parameter[] parameters = (Parameter[]) desc.getModifier();
assert parameters[parameters.length - 1].key().equals(ParameterType.STATUS.getName());
if (parameters[parameters.length - 1].value().equals(status)) continue;

Parameter[] paraCopy = Arrays.copyOf(parameters, parameters.length);
paraCopy[parameters.length - 1] = ParameterType.STATUS.getParameter(status);
edge.remove();
addSchemaEdge(vertex, edge.vertex(Direction.IN), TypeDefinitionCategory.INDEX_FIELD, paraCopy);
```

The invariant "`STATUS` is the last parameter" is established in exactly one place,
[`addIndexKey:785-791`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/management/ManagementSystem.java#L785-L791),
which builds `[user parameters…][MAPPED_NAME if absent][STATUS]`. Nothing else in main sources appends to
that array, so the invariant holds **today**. The hazard is the failure mode if it ever stops holding:
assertions are disabled in production, so `setStatusEdges` would overwrite whatever parameter happens to
be last — **`MAPPED_NAME`, the backend field name, is the adjacent slot** — and the index would keep its
old status with no error anywhere, while the mixed index backend is addressed under a value that is now a
`SchemaStatus`.

**Not run, and not a live defect.** The PR is a robustness change: find the `STATUS` parameter by key
rather than by position (`ParameterType.STATUS.findParameter(parameters, null)` or an index search),
replace it in place, and keep the `assert` as a comment. Add a unit test that appends a trailing parameter
to an `INDEX_FIELD` edge and asserts that a subsequent status change does not disturb `MAPPED_NAME`.

Small, self-contained, and it is the kind of change a maintainer can evaluate without running anything —
which makes it a good first PR on this subsystem even though nothing is broken.

### 70 — the pending INSTALLED → REGISTERED transition is JVM-local and unpersisted

**The one of this batch with user-visible consequences, and the one that needs a run before it is filed.**

`REGISTER_INDEX` does not set `REGISTERED`. It sets `INSTALLED` and arms a callback
([`ManagementSystem:1026-1032`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/management/ManagementSystem.java#L1026-L1032)):

```java
case REGISTER_INDEX:
    setStatus(schemaVertex, SchemaStatus.INSTALLED, keySubset);
    ...
    setUpdateTrigger(new UpdateStatusTrigger(graph, schemaVertex, SchemaStatus.REGISTERED, keySubset));
```

`setUpdateTrigger` (`:1303-1305`) appends to an `ArrayList` field; `commit` (`:256-257`) hands it to
[`ManagementLogger.sendCacheEviction`](https://github.com/JanusGraph/janusgraph/blob/a59ae8d8024207598eada34998d867617df51adf/janusgraph-core/src/main/java/org/janusgraph/graphdb/database/management/ManagementLogger.java#L239-L260)
(blob-SHA verified, 24,707 bytes = `1980d1fb`), which parks it in an in-heap `ConcurrentHashMap`
(`evictionTriggerMap`) keyed by eviction id. **Nothing about the pending transition is written to the
graph.** If the issuing JVM exits between `commit()` and the final acknowledgement, the index is
`INSTALLED` on disk and no instance in the cluster holds a callback to advance it.

Observable symptom: `ManagementSystem.awaitGraphIndexStatus(g, name)` polls until its **60-second default
timeout** (`AbstractIndexStatusWatcher`, blob `e0ab9913`) and returns a report with `succeeded == false`,
with nothing logged to say why. The index is permanently unusable for queries — it is receiving writes
(the write gate is `== DISABLED`, `StandardJanusGraph:1125`), so it is silently accumulating correct data
that no query will ever consult.

Recovery exists and is undocumented: `REGISTER_INDEX`'s applicable set is `{INSTALLED, DISABLED}`, so
re-issuing it from any live instance re-arms the trigger. That is the cheap half of the fix.

**Three possible PRs, in increasing order of ambition — and the first is the one to send:**

1. **Documentation.** State in the index-lifecycle docs that a `REGISTER_INDEX` whose issuing instance
   dies leaves the index at `INSTALLED`, and that re-issuing `REGISTER_INDEX` is the recovery. Zero risk,
   immediately useful, and it is the half of this that is *certainly* true.
2. **Observability.** Log at `WARN` from `GraphIndexStatusWatcher` when it times out on an index still at
   `INSTALLED`, naming the instances it is still waiting on — `EvictionTrigger` already has that set, but
   only on the issuing instance, so this would read `getOpenInstances()` instead.
3. **Durability.** Persist the pending status change so any instance can complete it. This is a real
   design change and should not be proposed without a maintainer conversation first.

**Status: derived entirely from reading. Nobody has killed a JVM mid-handshake.** Lesson 25's Hands-On D
is that experiment: two instances against a shared backend, a long transaction held open on the second,
then kill it without closing the graph. **Run D before filing even the documentation PR** — the claim
"the index is stranded" needs one observation behind it, and D also settles whether
`graph.management-auto-close-stale-instances` rescues the *receiver* case cleanly, which would narrow the
report to the *issuer* case alone and make it much more credible.

---

## Filing guidance for this batch

- **68 goes up first and alone.** It is static, complete, verifiable in thirty seconds by a maintainer,
  and bundles the `validateIndexStatus` javadoc correction. Nothing needs to be run.
- **69 is the second PR**, independent of 68. Robustness plus a test; no behaviour change today.
- **70 is the valuable one and the one to be careful with.** Run Hands-On D, then send option 1
  (documentation) with the observation attached. Options 2 and 3 are follow-ups, and 3 is an issue for
  discussion rather than a PR.
- Lesson 24's **candidate 65 is still blocked** on its Hands-On B, which has now gone unrun for two
  lessons. If the intent is an LFX Mentorship application in January 2027, 65 and 70 are the two with a
  real finding behind them, and both are waiting on the same thing: somebody running the experiment.
