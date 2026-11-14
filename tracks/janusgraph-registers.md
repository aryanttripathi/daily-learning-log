<!--
registers
track: JanusGraph
upstream: JanusGraph/janusgraph
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
split-from: TRACK.md on 2026-11-14 (lesson 16)
-->

# JanusGraph track — PR candidates and open questions

Split out of [`TRACK.md`](../TRACK.md) on **2026-11-14**, at lesson 16, because `TRACK.md` is read in
full at the start of every run and had reached 116 KB — most of it these two append-only registers,
which matter only when choosing a contribution or settling a debt, not when deciding what to teach
next. Nothing was dropped in the move; both sections are reproduced verbatim, with candidates 31 and
32 appended and the lesson-16 obligations resolved in place.

**How a run uses this file.** `TRACK.md` is the source of truth for *what comes next*. This file is
the source of truth for *what is still owed and to whom*. A run reads `TRACK.md` always, and reads
this file when (a) its lesson is named as the owner of an open question, or (b) it is about to add a
PR candidate and needs the next free number.

---

## PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 31-32 are new this run** (lesson 16); 29-30 came from lesson 15,
26-28 from lesson 14, 24-25 from lesson 13 and 21-23 from lesson 12, which also **rewrote** candidate 20
rather than dropping it. **18 is still the most substantial the track has found. The best first
contributions are now 29, 31, 28, 21 and 24** — 29 and 31 are one-line comment/message corrections, 28 is
a one-line error message with no behavioural risk, 21 is doc-only and 24 is a two-line guard fix, and all
five force a maintainer to confirm a reading. Candidates 1-30 carry forward; 31-32 were verified at this
pinned commit on 2026-11-14, 29-30 on 2026-11-13, 26-28 on 2026-11-12, 24-25 on 2026-11-11, 21-23 on
2026-11-10, 18-20 on 2026-11-09, 15-17 on 2026-11-08, 11-14 on 2026-11-07, 8-10 on 2026-11-06, 1 and 7 on
2026-11-05, 2-4 and 6 on 2026-11-04, and **5 still needs its experiment run — and is now worth more**,
because lesson 14 found a second, configuration-driven call site for `getBackend()`.

1. **`KCVSProxy.java:67`** passes `txh` where every sibling method passes `unwrapTx(txh)`. One token.
   Fix the same omission in the test double at `KCVSCacheTest.java:242` and add a `KCVSCacheTest` case
   that calls `getKeys(MultiSlicesQuery, …)` through a cache and asserts the store received the
   unwrapped transaction — nothing currently does. Latent today (no caller above the storage layer), a
   `ClassCastException` the first time anything calls it.
2. **`RetryingTestExecutionExtension.java:77`** prints the wrong variable:
   `"Aborting test #" + minSuccess + " of " + maxInvocations` — `minSuccess` is the *required success
   count*, not the invocation index; `invocation` is in scope and used correctly in
   `evaluateExecutionCondition`'s messages. Also missing a space before `- still have retries left`.
3. **Replace the four empty concurrency overrides with `@Disabled("…")`** —
   `BerkeleyFixedLengthKCVSTest.java:32-40` and `BerkeleyVariableLengthKCVSTest.java:45-53`. Turns four
   reported passes into four reported skips with a stated reason. Fold in the third site lesson 08
   found: `InMemoryPartitionGraphTest` overrides `testPartitionSpreadFlushBatch`,
   `testPartitionSpreadFlushNoBatch` and `testKeyBasedGraphPartitioning` with empty bodies and reports
   them **passing** — the three tests that check partition spread are the three that don't run on the
   in-memory backend.
4. **`BerkeleyVariableLengthKCVSTest.retrieveBackendAndGetKeyValue` cannot fail.** Body `L56-120`, outer
   `catch (BackendException e) { //generic failure }` at **`L113`** with no `fail`, no rethrow and no
   assertion; the inner try `L90-102` wraps the call its own comment at `L93` marks *"Errors out here
   without fix"* and catches the same exception with only `txh.rollback()`. Also imports JUnit 4
   `assertEquals` in a Jupiter suite and opens its own transactions instead of using `getTxConfig()`.
5. **Needs one experiment first: `GraphDatabaseConfiguration.getBackend()`.** Run lesson 07's
   Hands-On §3 second half — call it on an already-open BerkeleyJE graph and record whether a second
   `Environment` opens, BDB's lock refuses it, or a silent second `Backend` appears
   (`jcmd <pid> Thread.print | grep ExpirationStoreCache` counts the cleanup threads). If it succeeds,
   that is a resource leak reachable from three call sites and worth an issue plus a fix to
   `JanusGraphEventualGraphTest:67`; if it fails, the fix is a rename.
6. **The reproduction test for [#4781](https://github.com/JanusGraph/janusgraph/issues/4781)**, specified
   by lesson 06. Second test still needed for the silent lock loss at
   `BerkeleyJEKeyValueStore.java:159-162`, which a fix touching only `beginTransaction` would miss.
7. **Issue, not a PR: the backoff formula.** `PERTURBATION_PERCENTAGE = 0.2` can only mean ±20% and the
   implementation cannot express it, but changing it changes retry timing for every deployment. File an
   issue with lesson 07's jshell reproduction and the measured attempt distribution, asking whether the
   intended behaviour is jitter around an exponential base or the current coin flip. **Check
   `git log -p --follow BackendOperation.java` first** — the `(int)` cast may predate `java.time.Duration`
   and have been harmless in a `long`-milliseconds version.
8. **`IDManager.fromVertexId`, `IDManager.java:632-636`** is missing `toVertexId`'s positivity check, so
   it accepts a temporary (negative) id and returns a negative "user id" instead of throwing.
   Precondition clause 1 uses `>>>`, clause 2 compares the **signed** negative id against a positive
   bound, the body uses `>>`. Verified at pb ∈ {0, 3, 5, 16}. At the production pb=5,
   `fromVertexId(getTemporaryVertexID(NormalVertex, 1))` returns **-36,028,797,018,963,968**; the
   legitimate round trip `fromVertexId(toVertexId(7)) == 7` is unaffected. `fromVertexId` is public,
   user-facing API (`docs/advanced-topics/custom-vertex-id.md`). The fix is one
   `checkArgument(id > 0, ...)` plus a `VertexIDAssignerTest` case — that test currently probes
   out-of-range with exactly `new long[] {0, idManager.getVertexCountBound()}`, never a negative.
   **File as an issue first**, not a PR: tightening a public method's input validation can break a
   caller relying on the current leniency. Attach lesson 08 §8's table.
9. **A comment, not code: `BackendTransaction`'s `EDGESTORE_MIN_KEY`/`EDGESTORE_MAX_KEY`.** Lesson 08 §9
   proves both bounds lie strictly outside the assignable key space, and lesson 09 completes the
   `MIN_KEY` half with `BASE_ID = 1`. That proof is what makes the half-open scan interval safe and it
   exists nowhere in the tree. Smallest possible contribution; pairs naturally with candidate 10.
10. **Two documentation bugs, one trivial PR each.** (a) `docs/advanced-topics/data-model.md` states the
    64-bit vertex id *"is the key which points to the row containing the vertex's adjacency list"* and
    describes no transformation — true only at `partitionBits = 0`, which **no real graph uses**. One
    paragraph, plus a sentence about the partition rotation. (b)
    `docs/advanced-topics/custom-vertex-id.md`'s two groovy samples call
    `graph.getIDManager().toVertexID(123L)` and `graph.getIDManager().fromVertexID(long)` — capital-D
    `ID`, while the actual methods are `toVertexId` and `fromVertexId`. The samples as written do not
    compile.
11. **The smallest the track has found until 31: `ConsistentKeyIDAuthority.java:420-421`** formats the
    timeout with `timeout.getNano()`, the nanosecond-of-second component, not the total. The timeout
    reaching `claimIDBlock` is `ids.renew-timeout`, default `Duration.ofMillis(120000)`, whose
    `getNano()` is **0** — measured under OpenJDK 21, along with `PT2S` → 0, `PT0.3S` → 300000000,
    `PT1.5S` → 500000000. So the one message emitted when ID allocation finally gives up, which per the
    `IDS_RENEW_TIMEOUT` javadoc fails the entire database, reads *"Reached timeout 0"* on every
    whole-second configuration including every default deployment. Fix is `timeout` or
    `timeout.toMillis()` with the format specifier adjusted. Zero behavioural risk. **Send this one
    first.**
12. **`ConsistentKeyIDAuthority.java:392-399`: a comment that describes unimplemented intent.** The
    rollback path builds an anonymous `BackendOperation.TransactionalProvider` commented *"Use normal
    consistency level for these non-critical delete operations"*, whose `openTx()` body is
    `manager.beginTransaction(storeTxConfigBuilder.build())` — character-for-character `openTx()` at
    `:223-225`. `storeTxConfigBuilder` had the key-consistent custom options baked in at construction
    (`:189`, `:196`, `:198`) and is never re-derived, so the "normal" consistency level is the critical
    path's consistency level. The anonymous class differs from `this` only in a no-op `close()` that
    `BackendOperation.execute` never calls. Either implement the intent (derive a non-key-consistent
    config) or delete the comment and the anonymous class and pass `this`. **File as an issue** — which
    of the two is right is a maintainer's call.
13. **Two dead assertions in `IDAuthorityTest`, plus the parameterization that would make them live.**
    `checkBlock:207-209` (a block's ids are evenly spaced) and `testSimpleIDAcquisition:247-251`
    (consecutive blocks abut, with `previous` write-only as a result) are both gated on
    `hasEmptyUid == (uidBitWidth == 0)`, and `configs():84-103` produces exactly three configurations
    with tag-bits **4** (default), **9** and **7**. **Neither assertion has ever executed in any run of
    any backend's `IDAuthorityTest`.** Nothing in the suite checks the two properties `getId`'s shift is
    responsible for. The fix is a fourth `configs()` entry with `IDAUTHORITY_CAV_BITS = 0`; the
    assertions then run as written and the commented-out `IDAUTHORITY_RANDOMIZE_UNIQUEID` lines at `:99`
    and `:135` can go at the same time.
14. **Two small things in the id-assignment hot path, one PR.** (a)
    `VertexIDAssigner.PartitionIDPool.accessed()` (`:378`, `:533-535`) writes
    `lastAccess = System.currentTimeMillis()` on a `volatile long` for **every element id assigned**, and
    its only reader `getLastAccess():537-539` has **no caller in the tree** (code-searched at the pinned
    commit). Measured **33.0 ns/op** against 0.30 ns for an empty loop, 50M iterations after warmup under
    OpenJDK 21 — roughly 1.3 s on a 10M-vertex, 3-relations-each load, for a field nothing reads. Delete
    the field, `accessed()` and `getLastAccess()`. (b) `SimpleBulkPlacementStrategy:58` uses one shared
    `java.util.Random` for every placement decision on every thread (a contended CAS on its seed) while
    `ConsistentKeyIDAuthority:246` uses `ThreadLocalRandom` for the same kind of draw; and the class
    javadoc (`:37-45`) claims the strategy "assigns all vertices created in a transaction to the same
    partition id", which is true of `getPartitions(Map):100-105` and **false** of
    `getPartition(element):95-97`, which re-rolls per call and is the path every relation takes. Fix the
    `Random`, fix the javadoc. `currentPartitions` being a non-volatile `int[]` written from any thread
    is worth a sentence in the PR body but not a change — a stale read is still a valid partition.

15. **A test for the relation-type ordering contract.** `IDManagementTest.edgeTypeIDTest:141-201` runs
    1,000,000 trials and asserts the round trip, the `DirectionID`, full buffer consumption, the inline
    round trip, `DataOutput`-vs-`getRelationType` equality, and both `getBounds` bracketings — and never
    compares two **different** type prefixes to each other. So the actual ordering property is unpinned:
    the encoding is monotone *within* a byte-length class and **inverts across one**. Add a case asserting
    exactly that, with the two concrete pairs lesson 10 verified — `count 1023` (`7f fe`) vs `count 1024`
    (`70 10 80`), and the default-stride pair `1008` vs `1024` — so "columns are grouped by type, not
    sorted by type id" is documented in the suite rather than folklore. Phrase the assertions as the
    contract (`compareTo(...) > 0` with a comment naming the STOP-bit cause), not as a bug. The same patch
    can fix `int trails = 1000000` at `:145` (`trails` → `trials`). **Smallest-risk candidate on the list
    after 11** — pure test addition, no production change.
16. **One paragraph in `docs/advanced-topics/data-model.md`.** The doc's "Individual Edge Layout" section
    (L66-88) describes the relation-type id, the direction bit and the sort key correctly, and says nothing
    about the **six prefix bands** — which are the single most useful thing to know when reading a raw
    edgestore row, and the reason a `v.edges()` slice does not return vertex-centric index replicas. Add
    the band table (0-1 system, 2-3 user, 4-5 invisible = index replicas, 6-7 unreachable), one sentence
    on `getBounds`, and one sentence making explicit that columns are **grouped** by type rather than
    ordered by type id. Docs-only, no code risk.
17. **`EdgeSerializer.java:481-483` — the dead empty-interval branch.** `interval.isEmpty()` means
    *unsatisfiable*, and breaking out of the loop leaves `sliceStart == null`, which falls into `:533` and
    yields a slice over the **whole type band** — the maximum over-read, where the correct answer is the
    empty set. It is unreachable today because `BasicVertexCentricQueryBuilder:607-610` returns
    `BaseVertexCentricQuery.emptyQuery()` for any empty interval before `getQuery` is called, so this is
    not a live bug. Either delete the branch as dead or make the empty case produce an empty slice;
    which one is a maintainer's call, so **open as an issue first**, with the trace from `:481` to `:533`
    to `:548`, rather than as a PR.

18. **Hoist `readRelation` onto `RelationReader`, and make the OLAP label scan header-only.** The
    entry cache (`StaticArrayEntry`'s `transient volatile RelationCache cache`) is applied by
    `EdgeSerializer.readRelation`, which is **not** declared on the `RelationReader` interface — that
    interface has exactly one method, `parseRelation`. `JanusGraphVertexDeserializer.readHadoopVertex`
    holds a `RelationReader` from `setup.getRelationReader()`, so it cannot reach the cache, and it
    iterates the same `Iterable<Entry>` **twice** calling `parseRelation(data, false, typeManager)` both
    times: once in the vertex-label loop and once in the decode loop. So **every edgestore column in a
    Hadoop/Spark scan is fully parsed twice and memoised never**, which is precisely the cost `29c0b18a`
    (#4995) removed from the OLTP path. Two independent wins: add `readRelation` as a `default` method on
    `RelationReader` delegating to `parseRelation` (so implementations without an entry cache are
    unaffected) and switch `janusgraph-hadoop` to it; and change the label-finding loop to
    `parseRelation(data, true, typeManager)`, or to `EdgeSerializer.parseTypeId(data)`, since it reads
    only `relation.typeId` and, for the one matching column, `getOtherVertexId()` — today it fully parses
    the sort key, signature and every inline property of every column to find one vertex-label column.
    **Check before filing:** whether a second `RelationReader` implementation exists that a default
    method would surprise, and whether an open issue already covers OLAP scan cost. Land candidate 19
    first — `decodeProperties` guards on `hasProperties()`, which returns `false` for a header-only cache
    *without* throwing, so a careless reuse of a header-only parse there would silently drop properties.
19. **Make a header-only `RelationCache` throw instead of NPE.** `get(long)` (`:67-69`),
    `numProperties()` (`:85-87`) and `propertyIterator()` (`:97-101`) all dereference `properties`
    unguarded, so on a header-only cache (`properties == null`) each is a bare `NullPointerException`
    with no message. The contract is documented only in the `isFullyParsed()` javadoc (`:71-76`), and the
    entire enforcement is three hand-written conditions — `EdgeSerializer:85`,
    `CacheEdge.getPropertyMap`, `CacheVertexProperty.getPropertyMap`. `RelationCacheTest` asserts
    `isFullyParsed()` and `hasProperties()` on a header-only cache and asserts nothing about the other
    three. Throw `IllegalStateException("relation was parsed header-only")` from each, with a
    `RelationCacheTest` case per method. Smallest of the three new candidates, and it is the guard that
    makes 18 safe to write.
20. **Bound the ASC sort-key read as the DESC read is bounded — REWRITTEN by lesson 12; the trigger is
    a third-party serializer, not an in-tree one.** `EdgeSerializer:169-174` sends DESC through
    `in.subrange(keyLength, true)`, whose `limit` is exactly the sort-key window, so
    `StaticArrayBuffer.require` throws past it; ASC reads **in place** out of `in` itself and nothing
    compares the bytes consumed against `keyLength`. **Lesson 12 settled reachability: no.** Ten of the
    eleven in-tree order-preserving datatypes are fixed-width in byte order, and the eleventh (`String`,
    `3 + 2n`) is self-delimiting by a `00 00` terminator that `writeByteOrder`'s
    `Preconditions.checkArgument(c > 0)` refuses to let into the payload — read length equals write
    length over 20,000 strings. **What survives** is that `StandardRelationTypeMaker.checkSortKey`'s only
    gate is `instanceof OrderPreservingSerializer`, with no width or symmetry check, so a **custom**
    serializer registered through `attributes.custom.attribute[X].serializer-class` can be accepted as a
    sort key and read past its window — silently wrong on ASC, a loud `ArrayIndexOutOfBoundsException` on
    DESC. File it as *"bound the ASC sort-key read so a third-party `OrderPreservingSerializer` cannot
    read past the sort-key window"*: one line,
    `in.subrange(keyLength, def.getSortOrder() == Order.DESC)` (the non-inverting `subrange` returns a
    **view** with no byte copy), plus a test that registers a deliberately over-reading serializer and
    asserts the exception rather than garbage. Low priority, trivially reviewable, no behaviour change for
    any in-tree type. **If reviewers prefer, drop it — but record that decision rather than leaving it
    open a fourth lesson.** Note the inverting `subrange` *does* allocate (`new byte[length]` +
    arraycopy + `(byte)~b` loop + two wrappers), so every DESC index replica pays three allocations and a
    copy on its first property read. **Lesson 16 adds one reason to care:** the delete-side
    `acquireEdgeLock(key, entry)` compares the **full serialized value** byte for byte, so a
    non-deterministic third-party serializer breaks expected-value checking on deletes as well as
    ordering.
21. **Fix `docs/advanced-topics/serializer.md`'s *Custom Object Serialization* section. Doc-only,
    smallest candidate in the list, and the best first contribution this track has found.** The page
    states that *"JanusGraph will test both serialization and deserialization of a custom class before
    persisting data to disk"* and that the attribute class must implement `AttributeSerializer`, have a
    no-argument constructor and implement `equals(Object)`. Reading the whole registration path —
    `RegisteredAttributeClassesConverter.convert` → `RegisteredAttributeClass.registerWith` →
    `StandardSerializer.registerClass` → `registerClassInternal` — **there is no such round trip**: the
    registration validates the number, the non-nullity of the type and serializer and the absence of a
    collision, then puts two map entries. No instance is constructed, nothing is serialized, `equals` is
    never called. The no-arg-constructor requirement is real but applies to the **serializer**
    (`Class.forName(serializerName).newInstance()`), not the attribute class, which the paragraph
    conflates. Four edits: delete the round-trip claim, split the two requirements, drop the stale
    `equals` requirement, and add the two facts the page omits — that a **built-in** type's serializer
    cannot be replaced through configuration (with the hard-coded `HashMap` exception, from #306) and that
    both options are `GLOBAL_OFFLINE`. Zero code risk, and the review forces a maintainer to confirm the
    finding.
22. **Document the sort-key datatype restrictions where they are discoverable, and clear three dead
    spots in the same area.** `StandardRelationTypeMaker.sortKey`'s javadoc gives as its example *"the
    edge label friend has the sort key (since), which is a property key with a timestamp data type"* —
    and `InstantSerializer` is **not** an `OrderPreservingSerializer`, so the canonical example in the API
    documentation is precisely the case `checkSortKey` rejects. Replace it with `Long`/`Date`, list the
    eleven permitted types, and note that a `String` sort key forbids `U+0000` (enforced only at
    serialization time by `StringSerializer.writeByteOrder`; `verifyAttribute` checks only `MAX_LENGTH`).
    Bundle, or split if reviewers prefer one concern per PR: remove `StringSerializer`'s
    `TEXT_COMPRESSION_THRESHOLD`, which is `public static final`, never read by `StringSerializer`, and
    read only by a test to size random strings; remove `normalizeDataType`'s `Instant` branch, which is
    unconditionally dead (`java.time.Instant` is `final`, and the branch returns exactly what the
    fallthrough returns); and collapse `makeDefinition`'s triple `checkSortKey`/`checkSignature` calls to
    one each.
23. **Make `InstantSerializer` and `DurationSerializer` order-preserving.** Largest of the three new
    candidates and the most useful — `Instant` is the type users reach for first for a time-ordered
    vertex-centric index, and today it is rejected outright. The encoding has to be
    `[8-byte biased seconds][4-byte biased nanos]` = 12 bytes in byte order, leaving `read`/`write`
    untouched for on-disk compatibility, because the current nanos field rides `VariableLong`'s
    sign-magnitude zig-zag, which is **not** order-preserving (lesson 10's STOP-bit inversion applies).
    Tests: add both to `SerializerTest`'s `TYPES` (which moves `testSerializedOrder`'s
    `assertEquals(10, sortTypes.size())` to 12 — and that count is itself a fragile assertion worth
    replacing with a set comparison in the same PR) and to
    `testMissingValuesOfTheTypesWhichWriteThemLastComeAfterTheirValues`. **Search for an existing issue
    first**; #948 (UUID) is the precedent for how such a request is framed, and #1248 is the
    user-facing symptom report for the UUID version of the same complaint.

24. **The dead disjunct in `SystemTypeManager.throwIfSystemName`.** The only validation a user-supplied
    type name passes through is
    `if (SystemTypeManager.isSystemType(name.toLowerCase(Locale.ROOT)) || Token.isSystemName(name))`.
    `isSystemType` is `SYSTEM_TYPES_BY_NAME.containsKey(name) || ADDITIONAL_RESERVED_NAMES.contains(name)`,
    and the keys of `SYSTEM_TYPES_BY_NAME` are mixed-case (`~T$VertexExists`, `~T$SchemaName`, …) while
    the argument has been lowercased — so **the first disjunct can never match from this call site**, the
    only one a repository-wide code search surfaces. Nothing is exploitable: `Token.isSystemName`'s `~`
    prefix check still rejects every system name, and `ADDITIONAL_RESERVED_NAMES` (`key`, `vertex`,
    `edge`, `element`, `property`, `label`) is all lowercase and does fire. But the line reads as though
    two independent checks protect the namespace and only one does. Either drop the `isSystemType` call
    or compare case-insensitively and keep it meaningful; add a test asserting that
    `makePropertyKey("~T$SchemaName")` throws **and** naming which check catches it. Zero behavioural
    risk either way, and the review forces a maintainer to confirm the reading.
25. **Pin the ordinals of every persisted enum in a test, because the ordinal IS the on-disk format.**
    `EnumSerializer.write` is `VariableLong.writePositive(object.ordinal())` and `read` is
    `values[ordinal]` with only a **range** check, so all seventeen enums registered at
    `StandardSerializer` slots 41-57 (plus 69) are persisted by declaration order with no version marker.
    `TypeDefinitionCategory`'s first line is the comment *"Don't change the order because it breaks
    backward compatibility"* — and that comment is the entire mitigation, because
    `TypeDefinitionMap.setValue`'s `assert type.verifyAttribute(value)` is an `assert`, disabled in
    production. Inserting a constant mid-list silently reinterprets stored schema (a `SIGNATURE` reads as
    a `SORT_ORDER`, surfacing much later as a `ClassCastException` in `PropertyKeyVertex.dataType()`);
    deleting one shifts everything above it. The cheap, reviewable version is a new
    `PersistedEnumOrdinalTest` (or a block in `SerializerTest`) holding a golden name → ordinal list for
    all seventeen enums, so that a mid-list insertion fails CI instead of corrupting a graph. The
    expensive version — a version field, or name-based encoding — is an on-disk format change and almost
    certainly out of scope; **propose the test first and say so in the PR body.** **Lesson 16 adds one
    more persisted enum to the list:** `LogTxStatus`, written by
    `TransactionLogHeader.serializeHeader`'s `out.writeObjectNotNull(status)`, and `LogTxMeta`, whose
    **ordinal is written as a raw byte** (`out.putByte(VariableLong.unsignedByte(metaEntry.getKey().ordinal()))`)
    and read back as `LogTxMeta.values()[ordinal]` with no range check at all — so reordering `LogTxMeta`
    breaks WAL recovery across a version boundary, and that is a stronger case than the schema one.

26. **A `MASKABLE` option set once, by the first instance, becomes the cluster default forever — silently.**
    `ReadConfigurationBuilder.getGlobalSubset` (`:172-178`) filters on `ConfigOption.isGlobal()`, which
    **includes `MASKABLE`** (`ConfigOption:182`), so first-open copies every MASKABLE option present in
    that instance's properties file into the `system_properties` `"configuration"` row and then freezes
    it. That copy is never refreshed — `getGlobalSubset` runs only in the not-frozen branch — and never
    discrepancy-checked, because `getManagedSubset` (`:180-186`) filters on `isManaged()`, which
    excludes MASKABLE. But `MixedConfiguration.get:49-58` *does* fall through to it when the option is
    absent locally. Net effect: instance A creates the graph with `cache.db-cache-size=0.5`; instance B,
    months later, omits the line entirely and gets **0.5, not the code default**, with no warning in any
    log and nothing in the configuration reference that says so. Start doc-only plus a test: the
    `@ParameterizedTest` pair in `ReadConfigurationBuilderTest` (`:301`/`:328`) already drives the
    managed/not-managed split, and lesson 14's Hands-On §2 step 3 is the integration-level assertion
    (open a temp BerkeleyJE graph with a MASKABLE option set, reopen without it, assert the value). Then
    ask the maintainers which of the three fixes they want: document it, warn when a frozen MASKABLE
    global value is used with no local value present, or stop persisting MASKABLE in `getGlobalSubset`
    (a behaviour change, and the only one that is genuinely a fix). **Lesson 16 sharpens the
    consequence:** `storage.drop-whole-row-on-vertex-removal` is MASKABLE and defaults **true**, and it
    changes what a vertex removal emits to storage *and* what a CDC stream can identify — so it is a
    MASKABLE option whose first-instance value silently decides an index-correctness property for every
    later instance.
27. **`ModifiableConfiguration.set` has an `UPGRADEABLE_FIXED` escape hatch and `remove` does not.**
    `set` (`:37-44`) guards with `hasUpgradeableFixed(option.getName()) || !option.isFixed() ||
    !isFrozen()`; `remove` (`:54-59`) guards with only `!option.isFixed() || !isFrozen()`. So with
    `graph.allow-upgrade=true` you may **set** `graph.storage-version` on a frozen graph but may not
    **remove** it, and the failure is an `IllegalArgumentException` reading "Cannot change configuration
    option" that does not mention the asymmetry. This may well be deliberate — removing a version marker
    is not something an upgrade should do — but nothing says so, and the two guards sit nine lines apart
    looking like a copy-paste divergence. The small, reviewable PR is a one-line comment on `remove`
    stating the intent, plus a test pinning that `remove` of an upgradeable-fixed option throws even
    under `allow-upgrade`. If a maintainer says it is a bug instead, the fix is one token. Either way
    the review settles it.
28. **`KCVSConfiguration.asReadConfiguration()` throws a message-less `IllegalArgumentException` on a
    datatype mismatch.** `:202` is
    `Preconditions.checkArgument(!entries.containsKey(key) || dataType.isAssignableFrom(entries.get(key).getClass()));`
    — no message, no format arguments. This fires at open time, from
    `MixedConfiguration.has`/`get`, whenever a value stored in the `"configuration"` row does not match
    the `ConfigOption`'s current datatype — which is exactly what happens if an option's declared type
    changes across releases, or if a custom backend option collides on a path. The user sees a bare
    `java.lang.IllegalArgumentException` from inside a lambda, with no key, no expected type and no
    actual type. The whole fix is adding a message:
    `"Configuration value for key [%s] is a %s, not the expected %s"`. Zero behavioural risk, improves
    every support thread that ever hits it, and small enough to be a clean first PR. Worth pairing with
    a one-line test that stores a `Long` under a `String`-typed option's path and asserts the message
    names the key.

29. **`MIN_VERTEX_CACHE_SIZE`'s comment cites the wrong issue.** `StandardJanusGraphTx.java:201-205`
    reads *"This is a workaround for #893. Cache sizes small relative to the level of thread parallelism
    can lead to JanusGraph generating multiple copies of a single vertex in a single transaction."*
    Read with `issue_read` at the source:
    [JanusGraph/janusgraph#893](https://github.com/JanusGraph/janusgraph/pull/893) is a pull request
    opened 2018-01-17 by `davidclement90`, titled *"Remove TTL support for Elasticsearch in
    documentation."*, closed 2018-01-18, milestone `Release v0.2.1`, label `cla: yes`. It has nothing
    to do with vertex caches or thread parallelism; the number is almost certainly a carried-over
    **Titan** issue id from before the fork. Anyone trying to understand why a 100-element floor exists
    follows the comment into unrelated 2018 documentation history. One line, no behaviour change.
    **Do the homework before opening it:** search the upstream tracker (and Titan's archived tracker)
    for the actual duplicate-vertex report, and if it cannot be found, say so in the PR and propose
    describing the mechanism in the comment instead of citing a number — substituting one wrong number
    for another is worse than the status quo. The mechanism is independently verifiable and worth
    stating in the PR body: `CaffeineVertexCache.get`'s tier-2 (`volatileVertices`) → tier-3
    (`retriever.get`) sequence is not atomic, so two threads of one multi-threaded transaction can both
    miss and both construct; only one wins `cache.get(id, k -> v)`, and the loser has already been
    handed to its caller.
30. **`AddedPropertiesSetValue.isNull()` and `AddedPropertiesListValue.isNull()` are hardcoded
    `false`, which makes a cleanup branch dead code.** `SimpleAddedRelations.removeProperty` ends with
    ```
    if (propertiesValue.isNull()) { propertiesMap.remove(key); }
    ```
    `AddedPropertiesSingleValue.isNull()` returns `internalRelation == null`, so a `SINGLE` key's entry
    is pruned correctly. `AddedPropertiesSetValue.isNull()` and `AddedPropertiesListValue.isNull()` both
    `return false` unconditionally, so for a `SET` or `LIST` key the branch can **never** fire: remove
    every property of that key from a vertex and you are left with an empty `AddedPropertiesSetValue`
    wrapping an empty `HashMap` (or an empty `LinkedHashSet`), keyed by the key's name, for the rest of
    the transaction. `getAllProperties()` then concatenates an empty iterator per dead key on every full
    view of that vertex's properties. Bounded by the number of distinct key names ever written, so this
    is a wart rather than a leak — but it is an inconsistency between three implementations of one
    two-method interface, and the fix is `return internalRelationMap.isEmpty();` /
    `return internalRelationSet.isEmpty();`. **The test is the larger half of the PR:**
    `SimpleAddedRelations` has **no dedicated unit test** anywhere in the tree at the pinned commit, so
    this needs a new test class that adds and removes `SET`- and `LIST`-cardinality properties and
    asserts both `propertiesSize` and the pruning. Worth checking first whether any call site depends
    on the entry surviving — none was found in `StandardVertex` or `StandardJanusGraphTx`, but
    `getView(Set, Predicate)`'s `allProperties` short-circuit should be re-read before claiming that in
    the PR.

31. **NEW, and tied with 11 for the smallest: `prepareCommit`'s phase-7 log message mislabels its
    count.** `StandardJanusGraph.java:871` is
    `logForPrepareCommit.debug("7. Prepare commit is done with mutated vertex count={} in duration={}", mutations.size(), duration)`.
    `mutations` is a `ListMultimap<Object, InternalRelation>`, and Guava's `Multimap.size()` returns the
    **total number of values**, not the number of distinct keys — so the "mutated vertex count" is
    actually the number of (vertex, relation) pairs, inflated by a factor of the average relations per
    vertex. The number the message means is `mutations.keySet().size()`, which is also exactly the number
    of `mutateEdges` calls phase 5 made (`:1104` iterates `mutations.keySet()`), so the corrected figure
    is strictly more useful: it is the storage-level fan-out of the commit. One method call plus a
    message, zero behavioural risk. **Pair it in the PR body with a second observation in the same
    method, as a question rather than a change:** the gate at `:827` is
    `logForPrepareCommit.isDebugEnabled() && totalMutations >= backend.getBufferSize()`, so the entire
    phase trace is unavailable on any transaction smaller than `storage.buffer-size` (default **1024**) —
    a *logging* decision bound to a *storage batching* option, which looks deliberate (don't log the hot
    path) but means the trace cannot be used to debug an ordinary transaction. Ask whether a separate
    threshold, or a trace-level unconditional variant, would be welcome; do not change it unilaterally.
32. **NEW: `StandardJanusGraph.commit`'s stage numbering restarts mid-method and is genuinely
    misleading.** Comment-only. `:1218-1239` number themselves `//1. Finalize transaction`,
    `//2. Assign JanusGraphVertex IDs`, `//3. Commit`, then `//3.1 Log transaction`, `//3.2 Commit schema
    elements` — and then inside the `if (commitSummary.hasModifications)` branch the comments restart at
    `//1a. Add success message to tx log`, `//1. Commit storage`, `//2. Commit indexes`, `//3. Log
    transaction if configured`, before the `finally` resumes the outer scheme at `//4.` and `//5.`. So
    `//2.` means "assign IDs" in one place and "commit indexes" in another, 96 lines apart, and a reader
    tracing the failure ladder has to work out which scheme each number belongs to. The fix is to renumber
    the inner block as `3.3`, `3.4`, `3.5` (or `4a`, `4b`, `4c`) so one scheme covers the method, touching
    no code. Zero risk, and the review's value is that it forces a maintainer to confirm the intended
    structure — the `[FAILURE]` comments are the best documentation of commit's error semantics that
    exists and they are currently indexed by two overlapping numbering schemes. **File separately from 31
    and from anything behavioural**, and attach lesson 16 §7's linearised table as the proposed reading.

---

## Still open, and who owns it

- **Closed by lesson 14 — all three obligations lesson 13 handed it.** (a) **Construction order:**
  established line by line for `StandardJanusGraph:264-360`; the answer to "what must exist before a
  value can be read out of `system_properties`" is **a `KeyColumnValueStoreManager` and nothing else** —
  the config row is never read through `typeCacheRetrieval`, which is why the open path can create,
  use and close an entire store manager before `getBackend()` runs. `schemaCache` at `:308` is safe
  before `isOpen` at `:320` only because nothing in that window touches it; there is no assertion and
  no test. (b) **FIXED / GLOBAL_OFFLINE / GLOBAL / MASKABLE at open time:** the first three are
  **indistinguishable** — every open-path site uses `isGlobal()`/`isManaged()`/`isLocal()`, and the only
  two sites in the tree that read the constant are `ModifiableConfiguration.set:37-44` and
  `ManagementSystem`'s `configVerifier:190-207`. MASKABLE is the only type whose handling differs at
  open, in `MixedConfiguration.get`'s precedence and in its exclusion from the discrepancy check — and
  that exclusion combined with its inclusion in `getGlobalSubset` is PR candidate 26. (c)
  **`MetricInstrumentedSchemaCache` counts and nothing else**, confirmed by reading all 74 lines: a
  `StandardSchemaCache` with a MISS-incrementing retriever, no HIT counter, no counted eviction, and
  JVM-global counter names. Also corrected lesson 13's note that `attributes.custom.*` is FIXED — it is
  `GLOBAL_OFFLINE`.
- **Closed by lesson 15 — all three obligations lesson 14 handed it.** (a) **The no-backend window is
  harmless by construction:** every value `StandardJanusGraphTx`'s constructor reads off `graph`
  (`:308-317`) is a pre-built object, `txHandle` is the only backend-facing field and is non-final, and
  the constructor does no I/O whatsoever. `StandardTransactionBuilder`'s constructor reads **fifteen
  pre-resolved fields** off the `GraphDatabaseConfiguration` — not off a `Configuration` — so building a
  transaction consults no config row and cannot fail on a missing option. (b) **A user vertex can carry
  a temporary id past creation, but only with `ids.flush=false`, and the default is `true`** — so
  normally the temporary id lives for the three statements between `:686` and `:690`. Lesson 13's
  schema-vertex finding is confirmed: `makeSchemaVertex:1098` assigns unconditionally and asserts
  `longId() > 0`. The containers hold, in the meantime, a `StandardVertex` at lifecycle byte `New` plus
  its `VertexExists` property, which is the relation through which `assignIDs` reaches the vertex at
  commit. (c) **`newTypeCache` resolves name → id and `vertexCache` resolves id → object**, and they meet
  only in `getSchemaVertex:1216-1225`; `newTypeCache` is checked first, written only at
  `makeSchemaVertex:1109`, and emptied at `:2206`.
- **Closed by lesson 16 — all four things lesson 15 owed it, one of them by correcting lesson 15.**
  (a) **`getAllUnsafe()`'s live view reaches `prepareCommit` untouched** and is iterated twice, once by
  `prepareCommitAdditions` and once by `ChangedSchemaVertices.of`, plus a third time by
  `TransactionLogHeader.serializeModifications` when `tx.log-tx` is on; nothing copies it and nothing
  detects concurrent modification (`hppc` iterators are not fail-fast, and `commit()` is `synchronized`
  on the transaction while `addVertex` is not). (b) **`ChangedSchemaVertices` is cache invalidation, not
  persistence** — it filters on reference identity against `BaseLabel.SchemaDefinitionEdge`, keeps two
  sets (`all` for local expiry, `kept` = not-removed for the cross-transaction and cross-instance
  broadcast), is collected at `:1236` **before** any write because it reads live transaction state, and
  is consumed in a `finally` whose failures are logged and never escalated. The broadcast writes the
  management log **synchronously**, so it is on the commit's critical path. (c) **The `upsert` flag is
  not a commit-path mechanism at all** — a repository-wide code search at the pinned commit finds
  `isUpsert` in exactly one file, and its only read is inside `StandardVertexProperty.remove()`, where it
  runs the slice query the blind-overwrite optimisation skipped. Lesson 15's description of it as a
  "promise to the commit path" was wrong about the layer. (d) **The emission order is not established in
  the commit path.** `prepareCommit` collects deletes (phase 1) before additions (phase 2) and takes
  composite-index deletion locks before addition locks (phase 4), but phase 5 hands both lists to one
  `mutateEdges` per key; the resolution is `Mutation.consolidate()` keyed on
  `KCVEntryMutation.ENTRY2COLUMN_FCT` (the **column**, value ignored), under the documented rule that
  "additions apply logically after deletions" — i.e. lesson 02's `KeyColumnValueStore` contract, spent.
- **Owed to lesson 17 by lessons 15 and 16.** From lesson 15: a property write takes one
  `getLock(IndexLockTuple(index, match))` per unique composite index **record**, built from
  `IndexRecordUtil.indexMatches(vertex, index, key, normalizedValue)` (`StandardJanusGraphTx:997-1000`),
  and `TypeUtil.hasAnyIndex(key)` is one of the three conditions gating the blind-overwrite path. Lesson
  17 must say what a *record* is there before it explains the graphindex entries. From lesson 16 it
  inherits the full commit-side frame and only needs `IndexSerializer`: the **three** generation sites
  (`:919` per deleted relation, `:1007` per added relation, `:1022-1024` per mutated vertex via
  `parallelStream`), the `IndexUpdate` type itself, and `itx.needsCompleteDocument` /
  `registerCompleteDocument` (`:1196-1204`) — read while the transaction can still read, because the
  index commit runs after the storage commit. Cost baselines to cite rather than re-derive: lesson 13's
  11 edgestore columns + 2 graphindex entries per property key, and lesson 10's 2+N columns per edge.
- **Owed to lesson 18 by lessons 13, 14, 15 and 16, and it is now the most heavily owed item in the
  track.** From lesson 15: everything in `StandardJanusGraphTx` is transaction-local — `getLock`,
  `LockTuple`, `ReentrantTransactionLock`, `CombinerLock`, `FakeLock`, a hardcoded 5000 ms timeout, and a
  `uniqueLocks` map that never shrinks; the file reads `ConsistencyModifier.LOCK` at exactly one place
  (`:1030`). From lesson 16: **all four storage-lock call sites are in `StandardJanusGraph`** —
  `acquireEdgeLock(key, Entry)` at `:901` (delete: expected value = the full serialized relation),
  `acquireEdgeLock(key, column)` at `:1004` (add: expected value `null`, i.e. "must not exist"),
  `acquireIndexLock(key, Entry)` at `:1087` and `acquireIndexLock(key, column)` at `:1094`, with
  deletions locked before additions — plus the second of exactly two reads of `ConsistencyModifier.LOCK`
  in the tree, the static `acquireLock(relation, pos, acquireLocksConfig)` at `:761-766`, whose third
  conjunct means a `MULTI` edge never locks and a `SIMPLE` edge locks only at `pos == 0`. Lesson 18 owes:
  how `ExpectedValueCheckingTransaction.prepareForMutations()` turns those into `ConsistentKeyLocker`
  traffic; what a `null` expected value means on the wire; why a failed expected-value check at
  `commitStorage()` leaves the schema sub-transaction's locks held until they expire (`:1278-1280` says so
  in a comment); and how a transaction-local lock and a storage lock compose on one property write. Also
  still owed from lesson 14: `KCVSConfiguration.set(..., checkExpectedValue=true)` calls `store.acquireLock`
  at `:153` on the config row — who calls that overload, and does the config row therefore go through
  `ConsistentKeyLocker` exactly as a `SchemaName` graphindex entry does? Related and unanswered:
  `StandardJanusGraph:341`'s `REGISTRATION_TIME` write goes through `ModifiableConfiguration.set`, i.e. the
  **non**-expected-value overload, so provisionally the instance registration is **unlocked** and
  `ManagementSystem`'s `openInstances.size() < 2` guard for GLOBAL_OFFLINE is racing it. And from lesson 13:
  `BaseKey.SchemaName`'s internal composite index declares `ConsistencyModifier.LOCK` and
  `getBackingIndexName() == Token.INTERNAL_INDEX_NAME`, so "one name, one type" rests entirely on
  `ConsistentKeyLocker` — establish whether two instances creating the same property key concurrently fail
  cleanly or leave two ids under one name. Plus from lesson 09: `ConsistentKeyIDAuthority`'s comment at
  `:90-92` says its slice bounds mirror `ConsistentKeyLocker`'s; read the two protocols side by side, with
  `ExpectedValueCheckingStore`, `BerkeleyJETx`, `BerkeleyJELocks` and `LockKeyColumnValueStoreTest`.
  **Lesson 18 is also the Part III/IV boundary, so it owes the re-pin review** — re-run `list_commits` on
  `JanusGraph/janusgraph` `master` rather than assuming either answer.
- **Owed to lesson 19 by lessons 15, 16, 08 and 04.** From lesson 15: why `StandardVertex.hasLoadedRelations`
  is a flat `false` and `loadRelations` returns `EntryList.EMPTY_LIST` for a new vertex while `CacheVertex`
  must answer both truthfully; and what `relTypeCache` (`:239`, a `ConcurrentHashMap<>(30)` allocated only
  when `isLazyLoadRelations()`, populated by `getOrLoadRelationTypeById:486-496`) actually caches, given its
  javadoc says "only in `JanusGraphLazyRelation`". From lesson 16: the read side of cache invalidation —
  `CacheTransaction.flushInternal`'s final pass calls `cache.invalidate(key, entries)`, or
  `invalidate(key, EMPTY_LIST)` for a `hasValidateKeysOnly()` cache, and lesson 07 established that
  invalidation **marks rather than evicts**; say what a transaction that commits and immediately re-reads
  actually sees. From lesson 08: the partitioned-vertex read cost — one logical vertex, up to
  `2^partitionBits` representative rows, one slice query each. Carried from lesson 04 and sharpened by
  lesson 07: the `getNoCopy` aliasing question belongs in `KeyColumnValueStoreTest` and must mutate through
  a `CacheTransaction`; the `MemoryEntryList` sizing question and the `InMemoryColumnValueStore.java:162`
  NPE candidate also land here. `MultiPageEntryBuffer`, `BufferPageUtils` and `BufferPage.merge` remain
  deferred with no owner; propose lesson 19.
- **Owed to lesson 25 by lessons 13, 14, 10 and 16.** From lesson 16, and it is the whole of §10:
  `tellInstancesToExpire(kept)` → `managementLogger.sendUnacknowledgedCacheEviction(schemaIds)` is the
  **unacknowledged** eviction, sent from a `finally` with failures logged at warn; `ManagementSystem.commit`
  sends an acknowledged one. Establish both paths and why a commit that merely writes definition edges gets
  the weaker one, and carry the tail-latency note (the management log writes synchronously, so a struggling
  backend holds the commit for up to its max-write-time). From lesson 13: `JanusGraphSchemaVertex.resetCache()`
  and `StandardJanusGraphTx.expireSchemaElement` are the invalidation half of a schema change, and
  `updateSchemaVertex` (which writes `SchemaUpdateTime`) is the only thing that marks one — is
  `SchemaUpdateTime` read by anything or only written? From lesson 14: `ManagementSystem`'s `modifyConfig`
  is a `ModifiableConfiguration` with `Restriction.GLOBAL` over a `TransactionalConfiguration` (`:175-177`),
  so the whole `JanusGraphManagement` config surface is writes to the one `"configuration"` row, and
  `graphShutdownRequired` (`:204`) is the GLOBAL_OFFLINE consequence — trace where it is honoured. From
  lesson 10, and now confirmed by lesson 16: the `SchemaStatus` gate is at `StandardJanusGraph:1125`, inside
  `prepareCommitAddRelationMutations`, and it checks only `== DISABLED` — `INSTALLED` and `REGISTERED` index
  types **do** receive writes, which is what lets a reindex job finish. Pair with `ManagementSystem:365-390`'s
  `canIndexBeEnabled` / `REGISTER_INDEX` branch. Also: changing a multiplicity rewrites the column layout and
  there is **no migration in `writeRelation`**.
- **Owed to lesson 28 by lesson 16.** The WAL producer side is fully mapped: four record shapes from
  `TransactionLogHeader` (`serializeModifications` for `PRECOMMIT` and `USER_LOG`, `serializePrimary` for
  `PRIMARY_SUCCESS`/`COMPLETE_SUCCESS`, `serializeSecondary` for `SECONDARY_SUCCESS`/`SECONDARY_FAILURE`
  carrying `[boolean userLogSuccess][int n][index name] × n`), the row key
  `HashingUtil.hashPrefixKey(SHORT, getLongBuffer(transactionId))`, the per-JVM `txCounter` starting at 0
  (`:321`) so ids collide across restarts and recovery must order by the header timestamp, and the fact that
  `logTransaction` is force-disabled under batch loading (`:1232`). Lesson 28 owes the consumer:
  `StandardTransactionLogProcessor`, how `getContentAsModifications` reconstructs `Change.ADDED` then
  `Change.REMOVED`, and what `recovery.getStatistics()`'s three counters actually count. Read
  `InMemoryTransactionRecoveryTest` and `InMemoryTransactionRecoveryThreadsTest`, neither of which any lesson
  has opened. Pair with [#833](https://github.com/JanusGraph/janusgraph/issues/833) (open since 2017, "Should
  we throw exceptions when commit indexes?"), which is the policy question stage 7's swallow raises.
- **Closed by lesson 12:** both questions lesson 11 owed it. **No in-tree `readObjectByteOrder` is
  length-ambiguous** — ten of eleven order-preserving datatypes are fixed-width in byte order and
  `String` is self-delimiting by a terminator its own writer refuses to let into the payload — so PR
  candidate 20's silent-corruption framing is unreachable and the candidate was **rewritten**, not
  dropped: the surviving trigger is a custom third-party `OrderPreservingSerializer`, which
  `checkSortKey` accepts on nothing but an `instanceof`. And **`VariableString.writeBackward` does put
  `STRING_ID_MARKER` at the high address**, with the `0x80` stop bit at the low one, so
  `readBackward`'s `getByte(--position)` peek is correct for the string case as it is for the long case.
  Also closed from lesson 10's list: `writeObjectByteOrder` vs `writeObject` vs `writeClassAndObject`
  (three positions selected by `InlineType`, plus a fourth path confined to `Object.class` keys),
  `writesMissingValuesLast` (a question about the flag byte, excluding exactly `ClassSerializer`), and
  **is every `isOrderPreservingDatatype` also fixed-width? — no, `String` is not, but it does not matter,
  because it is self-delimiting**.
- **Closed by lesson 13 — all three obligations lessons 10, 11 and 12 handed it.** (a) **The bands:**
  schema-definition relations do land in bands 0 and 1, because `BaseRelationType.isInvisibleType()` is
  an unconditional `true` and every `BaseKey`/`BaseLabel` id is a system id (suffix 37 or 53); implicit
  keys are user-band as lesson 10 suspected, and `writeRelation:364-365` filters them out of the
  remaining-properties loop regardless. (b) **The recursion:** `getExistingRelationType` bottoms out at
  `SystemTypeManager`'s static 17-entry `HashMap` for system ids, and otherwise at the vertex constructor
  at `StandardJanusGraphTx:641-664`, which picks the Java class **from the id's suffix bits alone**,
  before any byte of the row is read — so the format is not self-describing anywhere; the *ids* are. (c)
  **The ordinals:** every `TypeDefinitionCategory` reaches disk as a one-byte declaration-order ordinal
  through `EnumSerializer`, `getValue` validates only the range, `TypeDefinitionDescriptionSerializer`
  adds **no** versioning of its own, and schema values do use `writeClassAndObject` — `SchemaDefinitionDesc`
  has datatype `Object.class`, so every definition value is self-describing at the *type* level and
  unversioned at the *value* level. New PR candidates 24 and 25.
- **Owed to lesson 17 by lesson 13:** the eight **edge** `TypeDefinitionCategory` values (ordinals 17-24)
  are the whole of what hangs off `SchemaDefinitionEdge`. Establish which carry a real modifier
  (`INDEX_FIELD` with `Parameter[]`, `CONNECTION_EDGE` with a `String` are the two with datatypes) and
  what `CompositeIndexTypeWrapper`/`MixedIndexTypeWrapper` read out of `getRelated`.
- **Owed to lessons 21 and 26 by lesson 12, and it decides whether an inconsistency is a bug:** three
  comparators disagree — the stored bytes (`-0.0 < 0.0` strictly, NaN last), `AttributeUtils.compareValues`
  (same, via `Double.compareTo`), and `AttributeUtils.compare` (returns 0 for `-0.0` vs `0.0`, `null` for
  a one-NaN pair, and compares across numeric types where `compareValues` throws), plus
  `RelationComparator.compareValues` as a third entry point. Establish which one answers a `Cmp` range,
  which one `GraphCentricQueryBuilder` scores an index with, and whether any of the three divergences is
  reachable from a user query. Related: #5120, a mixed index given an order by a TEXT key no backend can
  sort by, is the same shape one layer up. And the string byte order is **UTF-16 code-unit order, not
  code-point order**, which is the UUID bug's twin waiting in `String` — Elasticsearch, OpenSearch and
  Solr sort strings by code point.
- **Owed to lesson 17 by lesson 12:** `IndexRecordUtil` makes the same `hasGenericDataType` →
  `writeClassAndObject` branch that `EdgeSerializer` does, so graphindex field values take the
  **non**-byte-ordered path. Establish whether a composite index key is byte-ordered at all, and if not,
  what gives a composite index its range behaviour. Carry lesson 12's cost table: the sort-key premium is
  zero for fixed-width types and `3 + 2n` vs `1 + n` for strings.
- **Owed to lessons 19-20 by lesson 11:** `RelationCache`'s header-only state is what
  `tx.getConfiguration().isLazyLoadRelations()` and `JanusGraphLazyRelationConstructor` trade on
  (`RelationConstructor`'s entry iterator branches on it). Establish which Gremlin steps settle for a
  header and which force a full parse — **17 of 50 is the static answer; the dynamic one is lesson 20's.**
  And lesson 20's starting point is now sharper: the extended sort key works *because* the backward
  encoding is order-preserving.
- **Closed by lesson 10:** `TYPE_LEN_RESERVE` and `MAX_PADDING_BITWIDTH` (they are `PREFIX_BIT_LEN` and
  the one shift in `stripEntireRelationTypePadding`); the `IDHandler`-vs-`getKeyID` first-bit convention
  (two different address spaces, nothing to reconcile); `BufferUtil.nextBiggerBuffer` (cannot overflow on
  a type prefix) and `padBuffer` (pads row keys, never columns); and where lesson 09's stride of 16 and
  the edge/endpoint co-location actually show up in the column layout — the stride as one byte of prefix
  in every cell, the co-location as the reason a vertex-centric index slice is local.
- **Owed to lesson 17 by lesson 10:** composite indexes go to the **graphindex** store with a different
  key and column scheme. Carry lesson 10's §8 column arithmetic over (2 + N columns per edge for N
  vertex-centric indexes, each a full re-serialization) so the two index kinds can be costed side by side.
- **Owed to lesson 20 by lessons 09 and 10:** `BasicVertexCentricQueryBuilder` properly —
  `constructSliceQueries`' recursion, the candidate scoring at `:683-713`, `getExtendedSortKey`, and the
  `ADJACENT_ID` / `JANUSGRAPHID` position assertions at `EdgeSerializer:466-471` that it has to satisfy.
  Plus, from lesson 09: edges take the partition of an incident vertex (`VertexIDAssigner:206-219`), so
  vertex-centric queries have a locality property that falls out of id assignment, not out of the query
  layer. Worth one paragraph.
- **Closed by lesson 16, from lessons 09 and 10:** `assignIDs(Iterable<InternalRelation>)`
  (`VertexIDAssigner:278-333`) is called from **`StandardJanusGraph.commit` at `:1226`, not from
  `prepareCommit`** (lesson 09's note had the caller one frame off), guarded by
  `!tx.getConfiguration().hasAssignIDsImmediately()`, and it is handed the **relations**, which is why
  `addVertex:692` unconditionally adds a `VertexExists` property — that property is the relation through
  which a never-assigned vertex is reached. Lesson 10's `wholeRowDeletion` short-circuit is at
  `:1136-1138` inside `prepareCommitAddRelationMutations` (the method moved since lesson 10 read it at
  `:1112-1138`), and the comment still says it is a serialization cost, not a correctness one. **Still
  unowned from lesson 09:** the bulk path's two-pass structure and its own 1000-attempt `leftOvers` loop,
  plus `VertexIDAssignerTest#testDisableIdsFlush:123` — propose them to lesson 19, which owns the
  partitioned-vertex read cost and is the next lesson with a reason to care about placement.
- **Owed to lesson 22:** `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299+`, shared mutable `current`, per-key limit at
  `L381`) plus `SimpleScanJob`.
- **Owed to lesson 27 by lessons 09, 10 and 16:** the ids-store column layout —
  `[-(end)][timestamp][uid]` with an empty value, slice bounds `zeroBuffer(1)` to `oneBuffer(17)` — is the
  smallest real column layout in the codebase and the cleanest thing to compare against
  `CQLKeyColumnValueStore`. Hands-On §2's byte dump is specified for it. Also the count stride of 16, which
  affects CQL token distribution. From lesson 10: the edgestore column layout is fully specified, so lesson
  27 can compare CQL's clustering-key encoding against the six-band prefix directly. **From lesson 16, and
  this is the sharpest one:** `CQLKeyColumnValueStore.mutateMany` is where the contract everything above
  depends on is either honoured or not — "deletions apply before additions in one `KCVMutation`", plus
  `KCVMutation.hasWholeRowDeletion()`'s promise that a whole-row delete is applied **before** any additions
  in the same mutation. Lesson 03 established that nothing in the codebase verifies a backend's declared
  features; lesson 16 established that `prepareCommit`'s correctness rests on exactly these two promises.
  Lesson 27 should read the CQL `BatchStatement` construction and say whether the ordering is explicit, or
  implicit in client-side timestamps (`graph.assign-timestamp`, LOCAL, default **true** — "one timestamp to
  all insertions and another slightly earlier timestamp to all deletions in the same batch", which is the
  real mechanism and is stated only in that option's description).
- **Carried from sessions 01-04:** 24 questions posed, none answered by Aryant. The half-open interval now
  has **three** implementations on record (`SinglePageEntryBuffer.getSlice` binary search;
  `BerkeleyJEKeyValueStore.java:258`'s per-record `compareTo`;
  `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange`) and two consumers relying on the upper
  bound being exclusive (`BackendTransaction`'s `EDGESTORE_MAX_KEY`, and `ConsistentKeyIDAuthority`'s
  `UPPER_SLICE = oneBuffer(17)`). **Lesson 10 adds a third consumer:** every `getQuery` slice end is
  `nextBiggerBuffer`-exclusive, and the missing-value marker relies on exclusivity on the ASC upper bound
  and the DESC lower bound. Lesson 27 adds CQL as the fourth implementation. The contract is in
  `KeyColumnValueStore`'s javadoc and **none** of the enforcement is. **Lesson 14 adds a fifth:**
  `KCVSConfiguration.get` builds `KeySliceQuery(rowKey, column, nextBiggerBuffer(column))` for a
  single-column point lookup, which is correct only because the upper bound is exclusive. Still
  outstanding from sessions 01-04 specifically: the `StoreFeatures` question about which core branches read
  `hasLocking()` (lesson 18 will have to answer it) and session 03's question about
  `KeyColumnValueStoreManager.openDatabase` naming. Neither is blocking.
