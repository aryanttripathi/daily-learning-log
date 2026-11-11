<!--
track-state
subject: JanusGraph
mode: lesson
mode-history: contributor-reading 2026-10-30 .. 2026-11-02 (sessions 01-04); lesson from 2026-11-03 (lesson 05 on)
started: 2026-10-30
lessons-total: 28
lessons-done: 13
pinned-commit: a59ae8d8024207598eada34998d867617df51adf
pinned-commit-date: 2026-10-28
pinned-commit-reviewed: 2026-11-11 at the Part II/III boundary (lesson 13) — a59ae8d8 is still the tip of
  JanusGraph/janusgraph master, so there is nothing to re-pin to; staying. Re-check at the Part III/IV
  boundary, after lesson 18.
upstream: JanusGraph/janusgraph
reader-fork: aryanttripathi/janusgraph
paused-tracks: PostgreSQL (resume at lesson 11; full state in tracks/postgresql.md)
-->

# Current Track: JanusGraph Codebase

Sequential deep dive through the JanusGraph source, one entry per day, each building on the last.
The destination is unchanged: contributing to JanusGraph before the January 2027 LFX Mentorship
applications open.

**Mode: `lesson`, switched 2026-11-03.** Sessions 01–04 ran as `contributor-reading`: a short
reading guide per day, with four to six questions deliberately left unanswered and the `## My
Notes` half written by Aryant. All four shipped with that half as the untouched template — 24
questions posed, none answered, no evidence a file was opened. Session 04's entry stated the
problem plainly, named the two honest options, and recorded a decision rule here for session 05 to
execute rather than ask a fifth time. **Session 05 executed it.** From lesson 05 on, entries answer
their own questions and carry `Where This Breaks Down`, `Further Study`, `Next Steps` and
`Takeaways` — same syllabus, same pinned commit, same depth bar. The carried questions are tracked
under each entry's `Next Steps` and settled as the syllabus reaches them. Flipping back is one line
of the front-matter above; nothing else in this file depends on the mode.

**Why bottom-up.** JanusGraph is a translation layer: a property graph mapped onto a
BigTable-style key → sorted-columns store, and Gremlin traversals turned into slice queries over
those rows. The syllabus follows that layering upward: storage contract → graph ⇄ bytes → write
path → read path → Gremlin integration → schema, indexes and production backends. Classes that
are reference material rather than reading material (`GraphDatabaseConfiguration`, most of
`StandardJanusGraphTx`) are visited only for the slices a lesson needs.

**Pinned source.** All line numbers and permalinks use commit `a59ae8d8` (2026-10-28) on
`JanusGraph/janusgraph` `master`. Re-pin only at a Part boundary, and record the new commit here.
**Reviewed 2026-11-11 (lesson 13, the last item of Part II, so the Part II/III boundary was available for
the first time since lesson 08): `a59ae8d8` is still the tip of `master` — the three newest commits are
`a59ae8d8` (2026-10-28), `9a30ff14` (2026-10-28) and `5e3e7bc7` *JanusGraph release 1.2.0* (2026-10-24).
There is nothing to re-pin to, so the pin stays and the decision is closed rather than deferred. Next
review: the Part III/IV boundary, after lesson 18.**

**Reader setup (assumed from lesson 02 on).** Fork at `aryanttripathi/janusgraph`, cloned locally
with `upstream` pointing at `JanusGraph/janusgraph`; Java 17 or 21, Maven 3.6.3+;
`mvn clean install -pl janusgraph-inmemory -am -DskipTests` built once; project open in IntelliJ.
From lesson 05, `janusgraph-berkeleyje` is also needed:
`mvn clean install -pl janusgraph-berkeleyje -am -DskipTests`.

## Session Log

Newest first.

- **13 — 2026-11-11** · [`2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md`](2026/11/2026-11-11-janusgraph-system-types-schema-storage/README.md) · **Part II complete; re-pin reviewed and declined — `a59ae8d8` is still the tip of `master`, so there is nothing to re-pin to.** Read `BaseKey` (whole), `BaseLabel` (whole), `BaseRelationType` (whole), `EmptyRelationType` (whole), `SystemTypeManager` (whole), `Token` (whole), `JanusGraphSchemaCategory` (whole), `TypeDefinitionCategory` (whole), `TypeDefinitionMap` (whole), `TypeDefinitionDescription` (whole), `SchemaSource` (whole), `StandardRelationTypeMaker` (whole), `StandardPropertyKeyMaker` (whole), `JanusGraphSchemaVertex` (whole), `PropertyKeyVertex` (whole), `StandardSchemaCache` (whole), `TypeDefinitionDescriptionSerializer` (whole), `EnumSerializer` (whole), `StandardSerializer` again (the registration table, slots 41-57 and 63), `StandardSchemaCacheTest` (whole), plus `StandardJanusGraphTx` (`makeSchemaVertex` 1080-1112, `updateSchemaVertex` 1114-1116, `getSchemaVertex` 1216-1224, `getRelationType` 1231-1239, `getExistingRelationType` 1252-1262, the vertex constructor 641-664, `getOrLoadRelationTypeById` 488-497, `expireSchemaElement` 2253-2275, `newTypeCache` 282/337/341/1109/1217/2209), `StandardJanusGraph` (`typeCacheRetrieval` 611-643, `vertexExistenceQuery` 306, `getSchemaCache` 554-556, `edgeQuery` 682-685) and `IDManager` again (the bit-mask doc 36-62, the `VertexIDType` suffixes, `addPadding` 345-353, `getSchemaId` 691-694, `isSystemRelationTypeId` 717-719).
  **Findings.** The schema is a subgraph in the same `edgestore`: six system property keys and two system edge labels, whose ids are computed arithmetically (`(raw << 6) | 37` for system property keys, `| 53` for system edge labels) so there is no bootstrap read — `VertexExists`=101, `SchemaName`=2085, `SchemaDefinitionProperty`=2149, `SchemaCategory`=2213, `SchemaDefinitionDesc`=2277, `SchemaUpdateTime`=2341, `VertexLabelEdge`=181, `SchemaDefinitionEdge`=2357 (the last of those and `SchemaUpdateTime` share raw id 36 and are separated only by the suffix). A property key is 11 edgestore columns plus 2 graphindex entries; its definition is **exactly 7** `SchemaDefinitionProperty` columns, enforced by set *equality* in `TypeDefinitionMap.isValidDefinition`, and one ordinal byte (5 `DATATYPE` vs 6 `UNIDIRECTIONAL`) is the whole on-disk difference between a property key and an edge label. Property keys and edge labels deliberately share the `rt` namespace, which is why `getRelationType` hard-codes `EDGELABEL.getSchemaName(name)` and why the two kinds can never share a name. **Lesson 11's debt settled**: `getExistingRelationType` bottoms out at `SystemTypeManager`'s static 17-entry map for system ids and otherwise at the vertex constructor, which picks `PropertyKeyVertex`/`EdgeLabelVertex`/`VertexLabelVertex`/`JanusGraphSchemaVertex`/`CacheVertex` **from the id's suffix bits alone**, before any byte of the row is read. **Lesson 12's debt settled**: every `TypeDefinitionCategory` is persisted as its declaration-order ordinal via `EnumSerializer` (`VariableLong.writePositive`, one byte for all 25), with no version marker and only a disabled-by-default `assert` for type safety — the "Don't change the order" comment is the entire mitigation, and it applies to all seventeen registered enums. Also: `StandardSchemaCache`'s `isCacheable` asymmetry (an empty `SchemaDefinitionEdge` is cached, an empty definition is not, because the latter means "not visible here yet"); its two-bit `systemTypeId` field admitting only four cacheable system types; one-way primary-map overflow to the Caffeine backup; every cache miss costing a whole key-consistent transaction with the KCVS cache disabled; and a dead disjunct in `SystemTypeManager.throwIfSystemName` (the lowercased argument can never match the mixed-case system-name map; `Token.isSystemName` is what actually protects the namespace). New PR candidates **24** (fix that dead disjunct) and **25** (pin the ordinals of all seventeen persisted enums in a test). No syllabus revisions.
- **12 — 2026-11-10** · [`2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md`](2026/11/2026-11-10-janusgraph-standard-serializer-attribute-encodings/README.md) · **stayed on `a59ae8d8`** (mid-Part; the re-pin decision is due **next run**, after lesson 13 closes Part II). Read `StandardSerializer` (whole), `Serializer` (whole), `OrderPreservingSerializer` (whole), `SupportsNullSerializer` (whole), `InternalAttributeUtil` (whole), `AttributeSerializer` (whole), and every registered attribute serializer that matters: `ByteSerializer`, `ShortSerializer`, `IntegerSerializer`, `LongSerializer`, `CharacterSerializer`, `BooleanSerializer`, `FloatSerializer`, `DoubleSerializer`, `DateSerializer`, `InstantSerializer`, `DurationSerializer`, `UUIDSerializer`, `EnumSerializer`, `StringSerializer`, `ObjectSerializer`, `ArraySerializer`, `SerializableSerializer` (all whole), plus `NumericUtils` (whole), `StringEncoding` (whole), `VariableLong` (whole, now the forward signed path), `VariableString` (whole), `StandardRelationTypeMaker` (whole), `AttributeUtils` in **`janusgraph-driver`** (whole), `RegisteredAttributeClass` (whole), `RegisteredAttributeClassesConverter` (whole), `GraphDatabaseConfiguration` L1363-1375 / L1967-1980, `EdgeSerializer` again (the `InlineType` / `writePropertyValue` / `readPropertyValue` call sites), `SerializerTest` (whole, 840 lines), `UUIDSerializerTest` (whole), `docs/advanced-topics/serializer.md`, and commit `46f9d249` (2026-10-15, Oleksandr Porunov, fixes #5087). **THE finding: the `0x00` flag byte is load-bearing for *order*, not for null.** `ByteSerializer.writeByteOrder(Byte.MAX_VALUE)` is `putByte((byte)(127 - (-128)))` = **the single byte `0xff`** — byte-identical to the missing-value marker `EdgeSerializer:506` writes to end an open sort-key range. The same collision sits at the top of every fixed-width type (`Short.MAX_VALUE` = `ffff`, `Long.MAX_VALUE` = eight `ff`, `Double.NaN` = `fff8…`, `new UUID(-1,-1)` = sixteen `ff`), and the only thing separating "largest value" from "no value" is the flag `writeObjectInternal` prepends for every type that is not a `SupportsNullSerializer`. Verified: re-implementing all eleven `writeByteOrder` bodies line-for-line and sweeping gives **0 violations** of `value < 0xff` across all eleven types. That makes `writesMissingValuesLast`'s odd second clause exact rather than arbitrary — it is the question *"does the first byte come from the two-valued flag protocol?"* — and the clause exists to exclude **exactly one serializer in the tree**, the private inner `ClassSerializer`, which supports null, writes no flag, and encodes null as registration 0 = `80000000`, so a `Class` sort key's missing values sort **first** (`SerializerTest:152-154` pins `Class`, `Instant` and `byte[]`, three types for three different reasons). **Lesson 11's question answered, and it kills a PR as written: no in-tree `readObjectByteOrder` is length-ambiguous.** Ten of the eleven order-preserving datatypes are **fixed width** in byte order (Byte/Boolean 1, Short/Character 2, Integer/Float 4, Long/Double/Date 8, UUID 16, enum 4, Class 4); the eleventh, `String`, is `3 + 2n` and self-delimiting by a `00 00` terminator that `writeByteOrder`'s `Preconditions.checkArgument(c > 0)` refuses to let into the payload — verified read-length == write-length over 20,000 strings, and `3 + 2n` exactly. **PR candidate 20 is therefore downgraded** from "silent corruption" to a one-line hardening whose only trigger is a third-party `OrderPreservingSerializer`, since `checkSortKey`'s sole gate is `instanceof OrderPreservingSerializer` with no width or symmetry check. Second finding: **the same value is encoded three different ways depending on *where in the column* it sits**, selected by `EdgeSerializer`'s private `InlineType` enum (`:383-394`) and its two orthogonal booleans — `KEY` → `writeObjectByteOrder`, `SIGNATURE` → `writeObject`, `NORMAL` → inline relation-type id + `writeObject` — with `writeClassAndObject` as a fourth path reachable only for `dataType == Object.class`, which `checkSignature` bans from both sort keys and signatures, so a sort key is never untyped. The sort-key premium is **zero for every fixed-width type** and lands entirely on the two with variable value encodings: `Integer` is 5 bytes as a sort key against 2 for any `|v| ≤ 63` (its `read`/`write` use `VariableLong`'s **sign-magnitude** zig-zag, `Math.abs(v)<<1 | sign`, while `readByteOrder`/`writeByteOrder` use a fixed biased `putInt`), and `String` is `3 + 2n` against `1 + n` — `'alice'` is **13 bytes** as a sort key against **6** as a value, and that lands in the column a vertex-centric index stores a second full copy of in each of two endpoint rows (20-char sort key, 2 VCIs → **258 bytes of sort key for one edge**). Third: **`Date` is order-preserving and `Instant` is not** — `InstantSerializer` implements plain `AttributeSerializer` only, so `checkSortKey` rejects it, and `StandardRelationTypeMaker.sortKey`'s own javadoc offers *"a property key with a timestamp data type"* as its example. Fixing it is a new 12-byte `[8-byte biased seconds][4-byte biased nanos]` encoding, not a marker, because the nanos field currently rides `VariableLong`'s non-order-preserving zig-zag; `DurationSerializer` is identical. Fourth: **`UUIDSerializer` knowingly violates the `OrderPreservingSerializer` javadoc** — both `write` and `writeByteOrder` are two **unbiased** `putLong`s, so the bytes sort by unsigned bits while `UUID.compareTo` compares each half signed. Measured: **10,070 of 20,000** random pairs disagree with `compareTo`, **0** with string order. Which is why `UUID` is absent from `SerializerTest`'s `TYPES` and therefore from `testSerializedOrder`'s 3,000,000-trial `signum(compareTo) == signum(bytes)` check — it would fail by design — and is covered instead by `UUIDSerializerTest.testByteOrderIsTheOrderOfTheStringForms`. The arc: #948 added the interface in March 2018 reasoning *"supporting native sort order simply means implementing `Comparable`"*; commit `46f9d249` (merged 2026-10-21, fixes #5087) fixed the **comparators** instead, adding `AttributeUtils.compareValues`/`compareUnsigned` and pointing `OrderList`, `RelationComparator`, `RangeInterval` and `Cmp` at them, and says explicitly what it did not fix (TinkerPop's own comparisons, so keyset paging with an unfolded `order()` can still skip UUIDs). Fifth: **enum sort keys are ordered by declaration order and nothing records it** — 17 internal enums are persisted by ordinal, so reordering any of their constants silently reinterprets stored schema; `getValue`'s bounds check catches a removal, never a swap. Sixth: **the registration table has 51 slots used (1, 10-21, 22-30, 41-57, 60-71) and exactly 28 internal slots left (72-99)** before `CLASS_REGISTRATION_OFFSET = 100` — which the code says "may NEVER EVER be changed" — would collide with user attribute 0. Also: `new HashMap<>(60)` for 51 entries **still rehashes** during construction (threshold 48); `normalizeDataType`'s `Instant` branch is **unconditionally dead** (`Instant` is `final`, and the branch returns what the fallthrough returns); `TEXT_COMPRESSION_THRESHOLD = 48` is `public static final` and `StringSerializer` **never reads it** (only `LONG_COMPRESSION_THRESHOLD = 16000` gates the single GZIP compressor, in a 3-bit field with 8 slots and 2 used); the custom-serializer extension point can **add** types but never replace one, with `HashMap` the single hard-coded escape hatch (from #306, a Titan migration collision); both `attributes.custom` options are `GLOBAL_OFFLINE`; `newInstance()` is the deprecated reflective form and its unwrapped constructor exception escapes the `catch`; and the docs' *Custom Object Serialization* section promises a serialize/deserialize round-trip before persisting that **does not exist anywhere in the registration path**. **Lesson 11's second question, answered:** `VariableString.writeBackward` writes characters in reverse then the marker, so `STRING_ID_MARKER` is at the **high** address and the stop bit at the low one — `readBackward`'s `getByte(--position)` peek is correct, and it emits the string in order without reversing. Also found: three near-duplicate in-tree ASCII codecs (`StringSerializer.write`, `StringEncoding.writeAsciiString`, `VariableString.write`) that disagree on the empty string and the marker; the string byte order is **UTF-16 code-unit order, not code-point order** (U+1F600 sorts *before* U+FFFF, verified), i.e. the same class of mismatch as the UUID bug waiting in a different type; a `String` property containing `U+0000` is storable as a value (the UTF path) but throws at serialization time if the key is a sort key, and `verifyAttribute` does not check it; and **three in-tree comparators that disagree** — the bytes, `AttributeUtils.compareValues`, and `AttributeUtils.compare` (which returns 0 for `-0.0` vs `0.0`, `null` for a one-NaN pair, and compares across numeric types where `compareValues` throws). Daily Diff: the **2026-11-09** edition (50 stories) — deep dive on Turso implementing Tantivy's `Directory` trait against its own B-tree, so an immutable-segment FTS index lives in the database file as 512 KB chunks keyed by `(path, chunk_no)` and commits in the SQL transaction, at the price of `NoMergePolicy` — automatic merging disabled, because a background merge would either leave the transaction or hold the single writer open.
- **11 — 2026-11-09** · [`2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md`](2026/11/2026-11-09-janusgraph-parse-relation-relation-cache/README.md) · **stayed on `a59ae8d8`** (mid-Part; the re-pin decision is due after lesson 13, one run away). Read `EdgeSerializer` again (whole, 591 lines, now from the read side), `RelationCache` (whole), `RelationReader` (whole, 10 lines and the point is what it does **not** declare), `VariableLong` (whole, this time the backward section), `IDHandler` (whole), `StaticArrayEntry` (whole), `StaticArrayBuffer` (whole), `ReadArrayBuffer` (whole), `CacheEdge` (whole), `RelationConstructor` (whole), `ModificationDeserializer` (whole), `Multiplicity` (whole), `JanusGraphVertexDeserializer` (whole), `EdgeSerializerTest` (whole, new this autumn), `RelationReparseBenchmark` (whole, new this autumn), plus commit `29c0b18a` (2026-10-02, Oleksandr Porunov, fixes #4995) and fragments of `CacheVertexProperty` and `RelationCacheTest` via code search. **THE finding: the backward encoding is the only genuinely order-preserving varint in the tree, and that — not readability — is why the tail ids are written backward.** `writeUnsignedBackward` puts the length in bits 4-6 of the **first** byte, the mirror image of `writeUnsigned`'s STOP-bit-on-the-**last**-byte, so a longer encoding's first byte is strictly greater: re-implementing `unsignedBackwardLength`/`writeUnsignedBackward`/`readUnsignedBackward` line-for-line and sweeping `0..400000` finds **zero** order inversions, against lesson 10's one at every length boundary in the forward prefix encoding. `262143 → 262144` is `8f 7f 7f → 90 10 00 00`. That is what makes `ImplicitKey.ADJACENT_ID` and `JANUSGRAPHID` legal sort-key positions in `getQuery`. The price is exact: capacity is **`4 + 7·(n-1)`** bits, not `7n`, and there is a **hard 3-byte floor** from `Math.max(3, ...)`, so the first vertex id a default graph assigns (**4096**, lesson 09) costs **3 bytes backward against 2 forward** (`80 20 00` vs `20 80`); beyond the floor the length field costs +1 byte for 3 of every 7 bit-lengths (bitLength ≡ 5, 6, 0 mod 7; boundaries at 19, 26, 33, 40, 47, 54, 61). Over `NormalVertex` ids at counts 16..160,000,000: **38 bytes backward against 34 forward, +12%**. A plain `MULTI` edge — the default multiplicity — is an **8-byte column with an empty value** (2 prefix + 3 `vid←` + 3 `rid←`) against a 6-byte forward ideal, i.e. ~400 MB of length fields on a 100M-edge graph stored twice, on top of lesson 10's ~200 MB of prefix padding from the same four tag bits. Round trip and the capacity law verified over `0..299999` plus `2^k ± {0,1,2}` for k=5..62, 0 failures. **Lesson 10's carried question, answered: the layout does not decide.** `parseRelation` completes the relation for all ten (type × direction) rows; what decides whether a header-only parse is as good as a full one is the four-way test at `:201-202` — `!in.hasRemaining() && keySignature.length == 0 && def.getSignature().length == 0 && !hasPropertyMetaData(data)`. The layout constrains it in one direction only: the two unconstrained rows are the only ones that can carry a sort key, so **every vertex-centric index replica fails clause 2 by construction** and is re-parsed on its first property read, permanently — the #4995 fix helps base columns and cannot help replicas. `EdgeSerializerTest.testRelationIsFullyParsedOnce` pins it at **17 of 50** entries, and the 17 and the 50 both reconstruct exactly. Also: `SIMPLE` **is** constrained (`isConstrained()` is literally `this != MULTI`), so lesson 10's six layouts are ten (type × direction) rows collapsing into **three** parse shapes — forward-only, pivot-at-`valuePosition`, full backward walk — and `SET` is the one constrained case that reads straight *through* `valuePosition` without jumping. **Second finding (PR candidate 18, the most substantial this lesson found): the OLAP path cannot reach the entry cache at all.** `RelationReader` declares only `parseRelation`; `readRelation` — which reads and writes `StaticArrayEntry`'s `transient volatile RelationCache cache` — is a method on `EdgeSerializer` alone. `JanusGraphVertexDeserializer` holds a `RelationReader` from `setup.getRelationReader()` and iterates the same `Iterable<Entry>` **twice**, calling `parseRelation(data, false, typeManager)` both times, so **every edgestore column in a Hadoop/Spark scan is fully parsed twice and memoised never** — and the first loop reads only `relation.typeId` and `getOtherVertexId()`, for which a header-only parse, or `EdgeSerializer.parseTypeId`, would do. Third finding (PR candidate 20): **the ASC sort-key read is unbounded and the DESC read is not.** `:172` sends DESC through `in.subrange(keyLength, true)`, which **allocates** (`new byte[length]` + arraycopy + `(byte)~b` loop + two wrappers) and whose `limit` makes `StaticArrayBuffer.require` throw past the sort-key window; ASC reads in place out of `in` itself, so an over-reading sort-key serializer would walk into the `vid←`/`rid←` tail and `in.movePositionTo(currentPos)` would discard the evidence — silently wrong values on ASC, a loud `ArrayIndexOutOfBoundsException` on DESC. Fourth (PR candidate 19): `RelationCache.get`, `numProperties` and `propertyIterator` all dereference a `null` `properties` on a header-only cache — a bare NPE, documented only in the `isFullyParsed()` javadoc, with exactly **three** hand-written guards behind it (`EdgeSerializer:85`, `CacheEdge.getPropertyMap`, `CacheVertexProperty.getPropertyMap`) and no test. Fifth: the shared static `EMPTY` map is a **mutable** `LongObjectHashMap` and `propertyIterator()` refuses to iterate it because HPPC writes an iteration seed on `iterator()` — a shared immutable sentinel that turns out not to be immutable. Sixth, cosmetic: `unsignedBackwardLength`'s `bitLength <= 4 ? 0 : ...` ternary is **dead** — `Math.max(3, ...)` discards the computed value for every bitLength 1..11 (11 of 64), and the whole function equals `max(3, ceil((bitLength+3)/7))`, verified for all 64 bit lengths. Also closed: `endKeyPos` is **exclusive** and `:169`'s comment ("we are on the last byte of the key") is off by one in wording while the arithmetic is right; `hasPropertyMetaData` is **not** `hasMetaData` (membership in `ImplicitKey.MetaData2ImplicitKey`, so `TIMESTAMP` keeps a parse header-only and a grouped read's `ROW_KEY` does not); `readVertexId(in, false)`'s string/long peek can never misfire on a long, because a backward long's last byte is always `≤ 0x7f` while its stop bit sits at the low address; and `ModificationDeserializer`'s `hasProperties()` is **not** a repeat of the #4995 mistake — it follows an explicit full `readRelation`, so it means "are there any". Daily Diff: the **2026-11-08** edition (22 stories), which lesson 10's run could not see — deep dive on a Postgres replication slot that cannot advance because its captured tables are idle while the server keeps writing WAL (~680 MB held over 25 quiet minutes at defaults), and on why the obvious fix silently loses half the rows on restart.
- **10 — 2026-11-08** · [`2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md`](2026/11/2026-11-08-janusgraph-edge-serializer-column-layout/README.md) · **stayed on `a59ae8d8`** (mid-Part; the re-pin decision is due after lesson 13, not available this run). Read `EdgeSerializer` (whole, 591 lines), `IDHandler` (whole), `VariableLong` L56-235, `EdgeDirection` (whole), `IDManager` L60-340 / L372-420 / L691-723, `Multiplicity` L35-120, `BufferUtil` L178-212, `WriteByteBuffer` L140-154, `StandardJanusGraph` L896-904 / L999-1006 / L1100-1156, `ManagementSystem` L330-400, `RelationTypeVertex` L54-117, `StandardEdgeLabelMaker` L90-110, `StandardRelationTypeMaker` L61-67 / L119-126 / L196-199, `TypeDefinitionCategory` L40-86, `BaseRelationType` L59-68, `ImplicitKey` L151-160, `InternalRelationType` L30-38, `Serializer` L38-44, `StandardSerializer` L262-272, `Interval`/`PointInterval`/`RangeInterval`, `BasicVertexCentricQueryBuilder` L498-723, `IndexRepairJob` L211-219, `TransactionLogHeader` L86-93, `IndexRecordUtil` L454-462, `AbstractTypedRelation` L101-110, `JanusGraphLazyRelation` L157-166, `JanusGraphVertexDeserializer` L124-133, plus `EdgeSerializerTest`, `IDManagementTest` L141-265, `SerializerTest` L139-170, `JanusGraphTest` L973-982 and `docs/advanced-topics/data-model.md`. **THE finding: lesson 09's four conflict-avoidance bits cost one byte in every single cell of every row.** `writePositiveWithPrefix` with `PREFIX_BIT_LEN = 3` gives a **1-byte** relation-type prefix only for `count << 1 | dir ≤ 15`, i.e. **schema count ≤ 7** — and the smallest count a default graph can assign is **16**, so **the cheapest case in the encoding is unreachable on every default deployment**, permanently (`FIXED` option). The 2-byte prefix then holds exactly **63** relation types (counts 16..1008); the 64th (count 1024) spills to 3 bytes. Second finding: **columns are grouped by relation type, not sorted by relation-type id.** Re-implementing `writeUnsigned` / `writePositiveWithPrefix` / `getPrefix` line-for-line and sweeping counts 1-5000 finds exactly **one** inversion — `1023 → 1024` (`7f fe` vs `70 10 80`), and at the default stride of 16 it is **1008 → 1024**, so the 63rd type sorts *after* the 64th. Inversions sit at every byte-length boundary because the STOP bit marks the **last** byte. Nothing in the read path depends on the stronger property — `getBounds` compares only the top 3 bits, per-type slices are exact byte-prefix matches, and the same STOP bit keeps different-length prefixes from interleaving inside a `Direction.BOTH` window — and **no test asserts any of it** (`IDManagementTest.edgeTypeIDTest`'s 1,000,000 trials never compare two type prefixes). PR candidate 15. Third: **six prefix bands exist, and `getBounds` can reach only four of them** — it passes `getPrefix(systemTypes, systemTypes)`, so it emits `(0,1)` or `(2,3)` and never `(4,5)`; bands 4 (`0x80`-`0x9f`) and 5 (`0xa0`-`0xbf`) are the **invisible-non-system** bands, which is exactly where every vertex-centric index replica lives. `ManagementSystem.buildRelationTypeIndex:368`'s `maker.invisible()` is what puts them there, and `StandardEdgeLabelMaker:106` makes invisibility *structurally required* (an IN-only unidirected label is illegal otherwise). An edge with *N* vertex-centric indexes is **2 + N full columns**, each a complete re-serialization. Fourth: **the column *is* the uniqueness constraint** — `valuePosition` is assigned in six places and `MANY2ONE`-at-OUT and `SINGLE` get a bare-prefix column, so last-write-wins needs no read; `SET` puts the value in the column (set semantics for free); `MULTI`/`LIST` put the relation id in the column so nothing is overwritten. Fifth: `EdgeSerializer:481-483`'s `interval.isEmpty()` branch is **dead** — `BasicVertexCentricQueryBuilder:607-610` returns `emptyQuery()` first — and if reached it widens the slice to the whole type band (PR candidate 17). **Closed lesson 08's and 09's loose ends:** `TYPE_LEN_RESERVE = 3` **is** `IDHandler.PREFIX_BIT_LEN`, `MAX_PADDING_BITWIDTH = 6` **is** the one shift in `stripEntireRelationTypePadding`, the "different first-bit convention" needs no reconciling (row keys put the partition at the top, columns put the band at the top — different address spaces), and `BufferUtil.nextBiggerBuffer` **cannot overflow on a type prefix** (band maxes at 5, so byte 0 ≤ `0xbf`) while `padBuffer` turns out never to touch columns at all (its only caller pads **row keys** in the OKVS adapter). Also verified: the missing-value `0xff` marker sorts last in ASC and, flipped to `0x00`, first in DESC, with the exclusive bound applied on the matching side in each case. Daily Diff: **no 2026-11-08 or 2026-11-07 edition exists** — front page and `/archive/` both top out at 2026-11-06, which lesson 09 already digested — so one uncovered item from that edition instead: Postgres `SKIP LOCKED` queues saturating 1 vCPU at 64 connections against a `max_connections` of 100.
- **09 — 2026-11-07** · [`2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md`](2026/11/2026-11-07-janusgraph-id-authority-block-allocation/README.md) · **stayed on `a59ae8d8`** (mid-Part, no decision available). Read `VertexIDAssigner` (whole, 545 lines), `StandardIDPool` (whole), `IDBlockSizer` (whole), `ConsistentKeyIDAuthority` (whole, 500 lines), `AbstractIDAuthority` (whole), `ConflictAvoidanceMode` (whole), `IDBlock` (whole), `SimpleBulkPlacementStrategy` (whole), `IDManager` L60-130 / L370-430 / L450-460 / L545-565 / L610-650 / L715-725, `GraphDatabaseConfiguration` L1040-1180, `VariableLong` L77-92, `IDAuthorityTest` L75-295, plus the test inventories of `IDPoolTest`, `VertexIDAssignerTest`, `InMemoryIDAuthorityTest`, `CQLIDAuthorityTest`, `HBaseIDAuthorityTest`, `PropertyPlacementStrategy` L95-110, `docs/changelog.md` L2041-2058, `docs/configs/janusgraph-cfg.md` L88-113, `docs/operations/bulk-loading.md` L78-91. **THE finding: `ids.authority.conflict-avoidance-tag-bits` defaults to 4, is `ConfigOption.Type.FIXED`, and its tag defaults to 0 with mode `NONE` — so `ConsistentKeyIDBlock.getId:486-491` returns `count << 4` with a zero tag and every count JanusGraph assigns is a multiple of 16.** The first `NormalVertex` id in a fresh default graph is **4096** at partition 0 (count 16), not 256; the stride between consecutive vertices in one partition is 128, not 8; and `maxAvailableBits = unsignedBitLength(idUpperBound)-1-uniqueIdBitWidth` takes the four bits straight off every namespace's ceiling, so **2^51 of 2^55 vertex counts, 2^54 of 2^58 relation counts and 2^50 of 2^54 schema counts are reachable — 15/16 of each space is not**, on every graph created with the defaults, unchangeably. Second finding: the give-up message at `:420-421` prints `timeout.getNano()`, which is **0** for any whole-second `Duration`, including the 120 s `ids.renew-timeout` default — "Reached timeout 0" on every default deployment (PR candidate 11). Third: the rollback path's anonymous `TransactionalProvider` (`:392-399`) is commented "Use normal consistency level" and builds a transaction config **character-for-character identical** to `openTx()` at `:223-225` (PR candidate 12). Fourth: `IDAuthorityTest`'s two contiguity assertions (`checkBlock:207-209`, `testSimpleIDAcquisition:247-251`) are gated on `hasEmptyUid` and **none of the three `configs()` parameterizations sets tag-bits to 0**, so neither has ever executed (PR candidate 13). Fifth: `accessed()` writes a `volatile long` via `System.currentTimeMillis()` on every element id (`:378`) for a `getLastAccess()` that has **no caller in the tree** — measured **33.0 ns/op** against 0.30 ns for an empty loop under OpenJDK 21 (PR candidate 14, bundled with the `SimpleBulkPlacementStrategy` items). **Closed lesson 08's `count == 0` question properly:** `BASE_ID = 1` (`AbstractIDAuthority:45`), so no count 0 is ever offered — `addPadding`'s refusal was never the thing protecting it. Also: five pools with `getIDNamespace() == ordinal()`, three per partition (`hasOnePerPartition()`); `renewBlockIndex = 0` for `SCHEMA` (block 50) and `PARTITIONED_VERTEX` (block 100) because `RENEW_ID_COUNT = 100` exceeds both, so those two namespaces permanently double-buffer; the negated-counter column order that makes `getCurrentID`'s `setLimit(5)` correct; the 330 ms claim and the ~9,090 vertices/s/pool it implies; and `renewalThreads = 3*partitionIdBound+2` = 98 at defaults against a live-pool ceiling of 32 (`ids.num-partitions = 10`). Daily Diff: the **2026-11-06** edition (52 stories), found via `/archive/` while the front page still served 2026-11-05 — deep dive on Infino's coordinator-free 128-bit ids, whose 40-bit random writer id is JanusGraph's 4-bit conflict-avoidance tag with 36 more bits of budget.
- **08 — 2026-11-06** · [`2026/11/2026-11-06-janusgraph-id-bit-layout/README.md`](2026/11/2026-11-06-janusgraph-id-bit-layout/README.md) · **Part II opens; stayed on `a59ae8d8` deliberately** so Part II's byte offsets stay comparable with lessons 02-07. Read `IDManager` (whole, 754 lines), `BufferUtil`, `NumberUtil`, `VertexIDAssigner`, `ManagementSystem`, `IDManagementTest`, `JanusGraphPartitionGraphTest`, `InMemoryPartitionGraphTest`, `docs/advanced-topics/data-model.md`, `docs/advanced-topics/custom-vertex-id.md`. **THE finding: the vertex id and the edgestore row key are two different 64-bit words** — `constructId:450` builds `[0 | count | partition | suffix]`, `getKey:502` rebuilds it as `(partition << partitionOffset) | addPadding(count)`, rotating the partition to the **top**. Second: **every default graph runs at `partitionBits = 5`, not 0.** Third: **`fromVertexId` accepts a temporary (negative) id and returns garbage** (PR candidate 8). Closed lesson 07's `EDGESTORE_MAX_KEY` question (PR candidate 9). Daily Diff: no 2026-11-06 edition at the time; digested **2026-11-05** (62 stories) — GitHub's "Rubicon" Git storage architecture.
- **07 — 2026-11-05** · [`2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md`](2026/11/2026-11-05-janusgraph-backend-store-wiring/README.md) · **Part I complete.** Read `Backend` (775 lines), `BackendTransaction` (568), `BackendOperation`, `CacheTransaction`, `KCVSCache`, `NoKCVSCache`, `KCVSProxy`, `ExpectedValueCheckingStoreManager`, `ExpirationKCVSCache`, `ExpirationCacheTest`, and slices of `GraphDatabaseConfiguration`. **THE finding: the retry backoff is a coin flip, not a ±20% jitter** — `(int)(2.0 / (1 + (rand*2-1)*0.2))` truncates to 1 or 2, measured median **14** attempts (max 38) against the 10 s read budget and **21** (max 48) against the 100 s write budget. Second: **`KCVSProxy.java:67` omits `unwrapTx`** (PR candidate 1). Daily Diff: the **2026-11-04** edition, 56 stories — PostgreSQL 19's `REPACK CONCURRENTLY`.
- **06 — 2026-11-04** · [`2026/11/2026-11-04-janusgraph-test-harness/README.md`](2026/11/2026-11-04-janusgraph-test-harness/README.md) · read the harness: `JanusGraphBaseTest`, `JanusGraphTest` (12,258 lines), `KeyColumnValueStoreTest`, the `@FeatureFlag` machinery, the BerkeleyJE suites, root `pom.xml`, `CONTRIBUTING.md`. Taught the src/main-not-src/test inversion, the one-abstract-hook-per-level contract, the **four** opt-out idioms and their four different reporting semantics, `@FeatureFlag`'s 4-of-24 coverage and its post-`@BeforeEach` ordering. Hands-on: read the Surefire XML rather than the console — 31 bare `<skipped/>` on `InMemoryGraphTest`, `testConcurrentGetSlice` as a 0 ms **pass**. Daily Diff: the **2026-11-03** edition, 52 stories — Spiral's *Statistics are aggregates*.
- **05 — 2026-11-03** · [`2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md`](2026/11/2026-11-03-janusgraph-okvs-adapter-berkeleyje/README.md) · **first entry in `lesson` mode.** Read `OrderedKeyValueStoreManagerAdapter`, `OrderedKeyValueStoreAdapter`, `BerkeleyJEKeyValueStore`, `BerkeleyJEStoreManager`, `KVQuery`, `KeySelector`, `BaseKeyColumnValueAdapter`, both BerkeleyJE KCVS suites. Taught the fixed-vs-variable key-length split, why fixed needs no post-filter and variable does, the three byte layouts, the cursor walk, the batch that isn't, and BerkeleyJE's own two-tier locking. Daily Diff: the 2026-11-02 edition, 49 stories — Neumann's *The Benchmarking Crisis*.
- **04 — 2026-11-02** · [`2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md`](2026/11/2026-11-02-janusgraph-inmemory-slice-path/README.md) · read `InMemoryKeyColumnValueStore` L85–149, `InMemoryColumnValueStore` (whole), `SinglePageEntryBuffer` L36–101, `BufferPage` L39–175, plus `MemoryEntryList` and `BufferPageTest`. Six questions posed, none answered in the entry. Radar: PR #5018, #3864, #1128. **`My Notes` never filled — the fourth consecutive empty session, which triggered the mode switch.**
- **03 — 2026-11-01** · [`2026/11/2026-11-01-janusgraph-store-manager-features/README.md`](2026/11/2026-11-01-janusgraph-store-manager-features/README.md) · read `KeyColumnValueStoreManager`, `StoreManager`, `StoreFeatures` L31–257, `StandardStoreFeatures`, `KeyColumnValueStore` L185–227, `Backend` L249–287 + L339–343, `InMemoryStoreManager`, `InMemoryStoreManagerTest`, `FeatureRequirementCondition`. Six questions posed, none answered. Radar: #2268, #3390, #1128. **`My Notes` never filled.**
- **02 — 2026-10-31** · [`2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md`](2026/10/2026-10-31-janusgraph-kcv-store-contract/README.md) · read `KeyColumnValueStore` (whole), `SliceQuery` L40–153, `KeySliceQuery`, `Entry` L27–39, `StaticBuffer` L28–86, `BufferUtil` L189–217, `KeyColumnValueStoreTest` L862–938, `InMemoryKeyColumnValueStoreTest`. Six questions posed, none answered. Radar: #3864, #3816, PR #4086. **`My Notes` never filled.**
- **01 — 2026-10-30** · [`2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md`](2026/10/2026-10-30-janusgraph-setup-and-data-model/README.md) · read `data-model.md`, `BUILDING.md`, `CONTRIBUTING.md`, `InMemoryStoreManager`, `StandardStoreManager` L28–33, `InMemoryGraphTest`. Six questions posed, none answered. Radar: #3013, #225, #1573. **`My Notes` left empty.**

## Syllabus

### Part I — Orientation and the storage contract (`diskstorage/`, `janusgraph-inmemory`)

- [x] 01 — Setup and the data model: fork, build `janusgraph-inmemory`, module map, and `docs/advanced-topics/data-model.md` (edgestore rows, relation columns)
- [x] 02 — `KeyColumnValueStore`: `getSlice`, `mutate`, `acquireLock`, plus `StaticBuffer`, `Entry`, `SliceQuery`, `KeySliceQuery`
- [x] 03 — `KeyColumnValueStoreManager` and `StoreFeatures`: how a backend advertises ordered scans, locking, batch mutation, and where core branches on them
- [x] 04 — `InMemoryKeyColumnValueStore`: answering a slice query over a sorted in-memory structure
- [x] 05 — `OrderedKeyValueStoreManagerAdapter` and `BerkeleyJEKeyValueStore`: faking key-column-value on an ordered key-value store
- [x] 06 — The test harness: `JanusGraphTest`, `InMemoryGraphTest`, backend test suites, and how a PR's test is expected to look
- [x] 07 — `Backend` and `BackendTransaction`: the stores that get opened (edgestore, graphindex, system_properties, ids, logs) and the cache/locker wrappers

### Part II — Graph ⇄ bytes (`graphdb/idmanagement`, `graphdb/database`)

- [x] 08 — `IDManager`: vertex ID bit layout, partition bits, vertex-type suffixes, and how row keys are derived
- [x] 09 — `VertexIDAssigner` and the ID authority: block allocation from the ids store without per-insert coordination
- [x] 10 — `EdgeSerializer.writeRelation`: the column layout for edges and properties, direction, sort keys
- [x] 11 — `EdgeSerializer.parseRelation` and `RelationCache`: decoding columns back into relations
- [x] 12 — `StandardSerializer`: attribute encodings, order-preserving serializers, and custom types
- [x] 13 — System types and schema storage: schema vertices, `BaseKey`/`BaseLabel`, and how a property-key definition is persisted

### Part III — The write path (`core`, `graphdb/transaction`, `graphdb/database`, `diskstorage/locking`)

- [ ] 14 — `JanusGraphFactory.open` → `GraphDatabaseConfiguration` → `StandardJanusGraph` constructor: local vs global (fixed/masked) config
- [ ] 15 — `StandardJanusGraphTx`: `addVertex`, `addEdge`, property writes, added/deleted relation containers, and the vertex cache
- [ ] 16 — `StandardJanusGraph.commit` and `prepareCommit`: deletes, additions, mutations, and the order they are emitted in
- [ ] 17 — Composite indexes on write: `IndexSerializer` and the graphindex entries
- [ ] 18 — Locking: `ExpectedValueCheckingStore` and `ConsistentKeyLocker`, and how uniqueness survives an eventually consistent store

### Part IV — The read path (`graphdb/vertices`, `graphdb/query`)

- [ ] 19 — `StandardVertex`/`CacheVertex`, lazy adjacency loading, and the transaction-level and database-level caches
- [ ] 20 — `BasicVertexCentricQueryBuilder`: `v.edges(...)` → `SliceQuery`, and vertex-centric indexes
- [ ] 21 — `GraphCentricQueryBuilder`: index selection between composite, mixed, and full scan
- [ ] 22 — `QueryProcessor`, limits, sub-query merging, and query profiling

### Part V — Gremlin integration (`graphdb/tinkerpop`)

- [ ] 23 — The strategy layer: `JanusGraphLocalQueryOptimizerStrategy` and the other optimizer strategies
- [ ] 24 — `JanusGraphStep` and `JanusGraphVertexStep`: folding `has`/`limit`/`order` into storage queries, and multi-query batching

### Part VI — Schema, indexes, and production backends

- [ ] 25 — `ManagementSystem` and the index lifecycle: INSTALLED → REGISTERED → ENABLED, and reindex jobs
- [ ] 26 — Mixed indexes: the `IndexProvider` SPI and `ElasticSearchIndex`
- [ ] 27 — The CQL backend: `CQLKeyColumnValueStore` compared line-for-line with the in-memory store
- [ ] 28 — Transaction log, CDC, and stale-index repair: what happens when the edgestore and a mixed index disagree

## Notes For The Next Run

**Syllabus unchanged this run. No items added, split, merged or reordered; the 28 stand as written on
2026-10-30.** Lesson 13 was handed three specific obligations by lessons 10, 11 and 12 and discharged all
three (bands, the `getExistingRelationType` recursion, the enum ordinals), so nothing needed re-scoping.

**The re-pin decision is CLOSED, and not in the way the last five runs expected.** Lesson 13 closed
Part II, so the Part II/III boundary was available for the first time since lesson 08 declined it. The
check this run: `master`'s three newest commits are `a59ae8d8` *Restore 1.3.0-SNAPSHOT version CTR*
(2026-10-28), `9a30ff14` (2026-10-28) and `5e3e7bc7` *JanusGraph release 1.2.0* (2026-10-24) — i.e.
**`a59ae8d8` is still the tip**, so there is nothing to re-pin to. The pin stays, the front-matter records
the review date, and **the next review is the Part III/IV boundary, after lesson 18**. Do not re-raise it
mid-Part; do re-run `list_commits` on `master` at that boundary rather than assuming the tip has moved or
has not.

**The roll-off rule applied again this run.** The root README states that `TRACK.md` keeps the notes for
the three most recent lessons and older ones roll off into git history. The three most recent are now 11,
12 and 13, so: **lesson 10's compressed block is removed** (it remains verbatim in git history at the
2026-11-08 commit, and in compressed form in the commit immediately preceding this one), **lesson 11's
block is now compressed** to the facts lessons 19, 20 and 27 were promised — the full block is preserved
verbatim in git history at the 2026-11-09 commit — and lessons 12 and 13 keep their blocks in full. Next
run, lesson 12's block becomes the oldest of the three and the same rule applies to it.

- **The mode question is closed. Do not reopen it unprompted.** Four consecutive empty `My Notes`
  triggered the switch recorded in the front-matter. Entries from lesson 05 on write in `lesson` mode
  without re-litigating the decision, without a notice at the top, and without nagging. If Aryant asks
  for reading sessions back, flip `mode:` and the entries resume the other shape from the next run.
- **Never touch an earlier entry's `## My Notes`.** Sessions 01-04 keep their templates. They are his
  to fill at any time; if a future run finds content in any of them, answer those questions in that
  run's entry regardless of mode.
- **Part III opens next run, and lesson 14 is the one item in it that is mostly reference material.**
  `GraphDatabaseConfiguration` is explicitly listed above as a class to visit only for the slices a
  lesson needs, and lesson 14 is that lesson. Three concrete obligations rather than a blank page:
  (a) establish the **construction order** in `StandardJanusGraph`'s constructor — `Backend`,
  `IDManager`/`VertexIDAssigner`, `EdgeSerializer`, `indexSerializer`, `schemaCache` (`:308`),
  `managementLogger` (`:344`) — and which of them must exist before a single value can be read out of
  `system_properties`, given that lesson 13 found `typeCacheRetrieval` opens its own transactions;
  (b) pin down what **FIXED**, **GLOBAL_OFFLINE**, **GLOBAL** and **MASKABLE** actually do differently at
  open time, since lessons 09 and 12 have both now been bitten by a `FIXED` option
  (`ids.authority.conflict-avoidance-tag-bits`, `attributes.custom.*`) whose value is unchangeable after
  the first open; and (c) say whether `MetricInstrumentedSchemaCache` changes any of lesson 13 §7's
  behaviour or only counts it — it wraps the same interface and lesson 13 did not read it.
- **Entry size.** Lesson 05 ≈35 KB, 06 ≈63 KB, 07 ≈78 KB, 08 ≈67 KB, 09 ≈61 KB, 10 ≈70 KB, 11 ≈56 KB,
  12 ≈91 KB, 13 ≈72 KB (a large share of each is permalink URLs, which are not prose). In line with the
  SQLite and PostgreSQL lessons. The old 5-9 KB budget was a `contributor-reading` constraint and does not
  apply.
- **Method note, carried — and lesson 13 is the first entry where it was mostly NOT needed.** Lessons 10,
  11 and 12 each re-implemented the pinned encoding functions line-for-line and ran them, which is what
  produced the inversion counts and the UUID disagreement figure. Lesson 13's arithmetic is small enough
  to do exactly (`(raw << 6) | 37`, the 25 ordinals, the two-byte description), so it was computed from
  the read sources and stated as such in the entry's method note rather than simulated. **Keep the rule
  that produced both:** compute or re-implement rather than assert, say which, and always pair it with an
  exercise that runs the real code — here `StandardSchemaCacheTest`, which already exists and already
  pins the asymmetry. One trap to remember from lesson 12: the first re-implementation pass omitted the
  dispatcher's flag byte and reported 13 false order violations. Model the layer above, not just the unit.

### Verified by lesson 13, for lessons 14, 17, 18, 20 and 25 to cite rather than re-derive

- **There is no catalog.** The schema is a subgraph in the same `edgestore`, written by the same
  `EdgeSerializer` and read back by the same `parseRelation`. `BaseKey` declares **six** system property
  keys and `BaseLabel` **two** system edge labels, and that is the entire vocabulary.
- **System type ids are computed, never stored**, by `BaseRelationType`'s constructor via
  `IDManager.getSchemaId(type, raw)` = `(raw << offset()) | suffix()`. Suffixes: `UserPropertyKey` 5
  (`000101`), `SystemPropertyKey` **37** (`100101`), `UserEdgeLabel` 21 (`010101`), `SystemEdgeLabel`
  **53** (`110101`), all at offset 6; `GenericSchemaType` 9 (`1001`) at offset 4. The eight actual ids:
  `VertexExists` **101**, `VertexLabelEdge` **181**, `SchemaName` **2085**, `SchemaDefinitionProperty`
  **2149**, `SchemaCategory` **2213**, `SchemaDefinitionDesc` **2277**, `SchemaUpdateTime` **2341**,
  `SchemaDefinitionEdge` **2357**. The last two share raw id 36 and are separated only by the suffix —
  the raw number is unique only *within* a `VertexIDType`. `SystemTypeManager` asserts
  `SYSTEM_TYPES_BY_ID.size()==17` and the same for names.
- **Names: `rt` for both relation kinds, `vl` for vertex labels, `gi` for graph indexes**, joined to the
  user name by `Token.SEPARATOR_CHAR = 0x1e`; `TYPE_MODIFIER.hasName()` is false and stores none. System
  names carry `Token.systemETprefix = Graph.Hidden.hide("T$")`. **Property keys and edge labels share one
  namespace deliberately** — `getRelationTypeName` is hard-coded to `EDGELABEL.getSchemaName(name)`, so
  one lookup serves `getPropertyKey` and `getEdgeLabel` and the two kinds can never share a name.
- **`TypeDefinitionCategory` has 25 constants, ordinals 0-24**, property categories 0-16 and edge
  categories 17-24 (`RELATIONTYPE_INDEX`, `TYPE_MODIFIER`, `INDEX_FIELD`, `INDEX_SCHEMA_CONSTRAINT`,
  `PROPERTY_KEY_EDGE`, `CONNECTION_EDGE`, `UPDATE_CONNECTION_EDGE`, `INDEX_INLINE_KEY`). **Lesson 17
  owns the eight edge categories.**
- **Definition sets are checked by set EQUALITY, not containment** (`TypeDefinitionMap.isValidDefinition`
  asserts `keys.size() == requiredTypes.size()`). `PROPERTYKEY` = 7 (`STATUS INVISIBLE SORT_KEY
  SORT_ORDER SIGNATURE MULTIPLICITY DATATYPE`), `EDGELABEL` = 7 (same but `UNIDIRECTIONAL` for
  `DATATYPE`), `GRAPHINDEX` = 6 (`STATUS ELEMENT_CATEGORY INDEX_CARDINALITY INTERNAL_INDEX BACKING_INDEX
  INDEXSTORE_NAME`), `VERTEXLABEL` = 2 (`PARTITIONED STATIC`, **no `STATUS`**), `TYPE_MODIFIER` = exactly
  1. `CONSISTENCY_LEVEL` and `TTL` are in no list — they live on separate `TYPE_MODIFIER` vertices
  reached by a definition edge, which is **lesson 18's** concern.
- **One ordinal byte is the whole on-disk difference between a property key and an edge label**: 5
  (`DATATYPE`) against 6 (`UNIDIRECTIONAL`).
- **Only two defaults exist** (`TypeDefinitionCategory.defaultValue`): `SORT_ORDER → Order.ASC` and
  `STATUS → SchemaStatus.ENABLED`. The `STATUS` default is live (vertex labels never store one); the
  `SORT_ORDER` default is dead for relation types, because `makeDefinition()` always writes it.
- **Cardinality is a view; multiplicity is the stored concept.** `StandardPropertyKeyMaker.cardinality`
  converts in, `PropertyKeyVertex.cardinality()` converts out, and `cardinalityIsSet` exists only so a
  `DefaultSchemaMaker` can tell an explicit SINGLE from silence. There is no `CARDINALITY` category.
- **A property key's row is 11 edgestore columns** — `VertexExists`, `SchemaName`, `SchemaCategory`,
  `SchemaUpdateTime`, and 7 × `SchemaDefinitionProperty` — **plus 2 graphindex entries** (UNIQUE on
  `SchemaName`, STANDARD/LIST on `SchemaCategory`). **Lesson 17 can cite this as the cost baseline.**
- **The schema vertex's id is assigned EAGERLY**, inside `makeSchemaVertex` (`graph.assignID` +
  `checkArgument(longId() > 0)`), unlike a user vertex, which `addVertex` may leave temporary. The reason
  is `SIGNATURE`/`SORT_KEY`: they store other types' `longId()` values, so a referenced type must already
  be final. **Lesson 15/16 should not expect a schema vertex ever to carry a temporary id past creation.**
- **The same meta-property has two byte layouts because of one override.**
  `EmptyRelationType.getSignature()` returns `long[0]`, so on a `SchemaDefinitionProperty` column the
  `SchemaDefinitionDesc` tag is an ordinary inline property (type id 2277 + value);
  `BaseLabel.SchemaDefinitionEdge.getSignature()` returns `{SchemaDefinitionDesc.id()}`, so on a
  definition **edge** it rides the value's fixed signature slot with no id. Lesson 10's column/value
  split is what makes that distinction possible.
- **`TypeDefinitionDescription` is registration 63 and its payload is `[category ordinal][modifier]`.**
  For a property category the modifier must be null, so every tag on a property key's definition is
  **exactly two bytes: `[ordinal][0x00]`**. Only `INDEX_FIELD` (`Parameter[]`) and `CONNECTION_EDGE`
  (`String`) carry a real modifier — **lesson 17 again**.
- **Lesson 12's ordinal debt, settled: `EnumSerializer.write` is `VariableLong.writePositive(ordinal)`,
  one byte for every constant below 128, and `getValue` validates only the RANGE.** So every one of the
  seventeen registered enums is persisted by declaration order with no version marker, and
  `TypeDefinitionMap.setValue`'s type check is an `assert` — disabled in production. Inserting a constant
  mid-list silently reinterprets stored schema; deleting one shifts everything above it. The comment at
  `TypeDefinitionCategory:41` is the entire mitigation. **PR candidate 25.**
- **Lesson 11's dispatch debt, settled: the Java class of every vertex comes from the id's suffix bits**,
  in the vertex constructor at `StandardJanusGraphTx:641-664` — `PropertyKeyVertex`, `EdgeLabelVertex`,
  `VertexLabelVertex`, `JanusGraphSchemaVertex`, `CacheVertex`/`PreloadedVertex` — before a byte of the
  row is read. `getExistingRelationType` short-circuits to `SystemTypeManager.getSystemType(id)` for
  system ids. `SchemaCategory` is stored for enumeration, not for dispatch. **Lesson 19 should note that
  `PreloadedVertex` is chosen by the `createStubVertex` flag on the same branch.**
- **`StandardSchemaCache`: two maps, both one-way.** `typeNames` (name → id) and `schemaRelations`
  ((schema id, system type, direction) → raw `EntryList`), each a `volatile` primary plus a bounded
  Caffeine backup; exceeding `MAX_CACHED_TYPES_DEFAULT = 10000` (or `×3` for relations) sets the primary
  to `null` **permanently**. The relation key is `(((schemaId >>> 2) << 2) + systemTypeId) << 1) +
  edgeDir` with a **two-bit** `systemTypeId` — so only four system types are cacheable
  (`SchemaDefinitionEdge` 0, `SchemaName` 1, `SchemaCategory` 2, `SchemaDefinitionProperty` 3) and
  anything else throws `AssertionError`.
- **The `isCacheable` asymmetry is deliberate and worth citing:** an empty `SchemaDefinitionEdge` result
  is cached, an empty name/category/definition result is **not**, because every existing schema vertex
  necessarily has those — so an empty answer there means *"not visible on this instance yet"*, not
  *"none"*. `StandardSchemaCacheTest` pins all four cases plus two eviction races.
- **Both cache layers use the same counter protocol**: increment the counter before clearing
  (`resetCache`, `expireSchemaElement`), and after storing a loaded value re-check the counter and undo
  the store if it moved (`JanusGraphSchemaVertex.keepUnlessReset`, `StandardSchemaCache.publish`).
- **Every schema cache miss is a whole separate transaction** built from
  `backend.getStoreFeatures().getKeyConsistentTxConfig()` with `getTxHandle().disableCache()`, rolled
  back in a `finally` (`StandardJanusGraph:611-643`). A name lookup reads the **graphindex** store
  (`QueryUtil.getVertices(tx, BaseKey.SchemaName, name)`); a definition lookup reads the **edgestore**
  (`edgeQuery` + `queryCache.getQuery(type, dir)`). **Lesson 14 needs this**: `schemaCache` is built at
  `StandardJanusGraph:308` from `configuration.getTypeCache(typeCacheRetrieval)`, and the retriever opens
  transactions, so it must be wired after `Backend` and before anything reads `system_properties`.
- **`SchemaCategory` carries `Index.STANDARD`, i.e. a LIST-cardinality composite index, and that is the
  entire implementation of schema enumeration** — `ManagementSystem.getGraphIndexes` and
  `JanusGraphHadoopSetupImpl.getTypeInspector` both call
  `QueryUtil.getVertices(tx, BaseKey.SchemaCategory, <category>)`. `SchemaName` carries `Index.UNIQUE`
  with `ConsistencyModifier.LOCK` and `getBackingIndexName() == Token.INTERNAL_INDEX_NAME`, so **name
  uniqueness rests on `ConsistentKeyLocker` — lesson 18.**
- **`newTypeCache` is a per-transaction name → id map**, checked *before* the global schema cache in
  `getSchemaVertex`, which is how a type created earlier in the same uncommitted transaction is findable
  by name; the transaction swaps it for `Collections.emptyMap()` on close.
- **A reachability bug in the only name guard:** `throwIfSystemName` lowercases its argument before
  `isSystemType(name)`, whose first disjunct checks a map keyed by mixed-case names (`~T$VertexExists`),
  so that disjunct can never match from this call site. `Token.isSystemName`'s `~` check is what actually
  protects the namespace, and `ADDITIONAL_RESERVED_NAMES` (`key vertex edge element property label`) is
  the only live half of `isSystemType`. **PR candidate 24.**

### Verified by lesson 12, for lessons 17, 20, 21 and 26 to cite rather than re-derive

- **Three positions, three encodings, one enum.** `EdgeSerializer`'s private `InlineType` (`:383-394`)
  has two orthogonal booleans over three cases: `writeInlineKey()` is true only for `NORMAL`,
  `writeByteOrdered()` only for `KEY`. So a sort key is `writeObjectByteOrder`, a signature key is
  `writeObject` with no id (position implies the key), and a remaining property is
  `IDHandler.writeInlineRelationType` + `writeObject`. `writePropertyValue:419-427` and
  `readPropertyValue:250-259` are exact mirrors. A fourth path, `writeClassAndObject`
  (`varint(registrationNo)` + body), is reached only when `InternalAttributeUtil.hasGenericDataType(key)`
  — `dataType == Object.class` — and `StandardRelationTypeMaker.checkSignature` bans `Object.class` from
  **both** sort keys and signatures, so it is structurally confined to a `NORMAL` position and a sort key
  is never untyped. Lesson 17 needs this: `IndexRecordUtil` makes the same `hasGenericDataType` branch
  for graphindex fields.
- **The flag byte, and why it is not about null.** `StandardSerializer.writeObjectInternal` prepends
  `0x00` for a value and writes a bare `0xff` for null, **unless** the serializer is a
  `SupportsNullSerializer`, in which case the serializer handles it. Only two order-preserving serializers
  are: `StringSerializer`, which writes the **same** `0x00`/`0xff` flag itself, and the private inner
  `ClassSerializer`, which writes none and encodes null as registration 0 (`80000000` after the int bias)
  so missing values sort **first**. That is the whole content of
  `writesMissingValuesLast = isOrderPreservingDatatype && (!supportsNullSerialization || instanceof
  StringSerializer)` — read it as *"does the first byte come from the two-valued flag protocol?"*
- **The flag is load-bearing for order at the top of every fixed-width type.** `ByteSerializer`'s body
  for `Byte.MAX_VALUE` is `0xff`, identical to the marker; `Short.MAX_VALUE` is `ffff`, `Long.MAX_VALUE`
  eight `ff`, `Double.NaN` `fff8…`, `new UUID(-1,-1)` sixteen `ff`. Lesson 10's missing-value range trick
  (`EdgeSerializer:504-507`, `colEnd.putByte((byte) -1)` with `endInclusive = false`) is sound **only**
  because of the flag. Sweep: 0 violations of `value < 0xff` across all eleven types.
- **The complete order-preserving set.** Thirteen classes implement `OrderPreservingSerializer`:
  `Byte Short Integer Long Character Boolean Date Float Double String UUID`, `EnumSerializer<E>`, and the
  private `ClassSerializer` — **29 of the 51 registered datatypes** (12 concrete + 17 enums).
  `SerializerTest:595-603` pins the user-facing count at **10** over a `TYPES` map of 21, so `Geoshape`,
  all nine array types and a plain custom serializer are **not** order-preserving; `UUID` is
  order-preserving but **absent from `TYPES`**, so it is outside the 3,000,000-trial check.
- **Byte-order widths, all fixed except one.** Byte 1 · Boolean 1 · Short 2 · Character 2 · Integer 4 ·
  Float 4 · Long 8 · Double 8 · Date 8 · UUID 16 · enum 4 · Class 4 · **String `3 + 2n`**. Every
  whole-number encoding is "add the bias, write big-endian"; `Byte`, `Short`, `Long` and `Date` delegate
  `readByteOrder`/`writeByteOrder` straight to `read`/`write`. **`Integer` and `EnumSerializer` are the
  asymmetric pair**: their `read`/`write` use `VariableLong` (1-5 bytes / 1 byte for a small ordinal)
  while their byte-order form is a fixed biased `putInt`. `CharacterSerializer`'s two biases annihilate,
  so a `Character` in byte order is the **raw UTF-16 code unit, big-endian, unbiased** — which is why
  `'\u0000'` is `00 00`, exactly the string terminator.
- **`VariableLong.write` is sign-magnitude, not standard zig-zag**: `Math.abs(v) << 1 | (v < 0 ? 1 : 0)`.
  Lengths for `Integer`: 1 byte for `|v| ≤ 63`, 2 for `≤ 8191`, 5 at the extremes. Two incidental dead
  spots, neither reachable from the serializers: unsigned `0` and `1` both decode to `0` and only `0` is
  emitted; and `convert2Unsigned` asserts `v > Long.MIN_VALUE` because `Math.abs(Long.MIN_VALUE)` is
  itself — `LongSerializer` never touches `VariableLong`, so nothing can reach it.
- **Float/Double: Lucene's bit flip, and NaN collapses one layer up.** `NumericUtils.sortableDoubleBits`
  is `bits ^ (bits >> 63) & 0x7fff…` — self-inverse, unchanged for non-negatives, magnitude inverted for
  negatives — then `LongSerializer`'s bias on top. The single-NaN behaviour
  (`SerializerTest:158-173`) is **not** in `NumericUtils`: `doubleToSortableLong` calls
  `Double.doubleToLongBits`, which by specification canonicalises every NaN to `0x7ff8000000000000`
  (`doubleToRawLongBits` would not). Stored order: `−∞ < … < −0.0 < 0.0 < … < +∞ < NaN`, which is
  `Double.compare`'s total order. Note `-0.0 < 0.0` **strictly** in the bytes.
- **`Date` is order-preserving, `Instant` and `Duration` are not.** `InstantSerializer` implements plain
  `AttributeSerializer` only, so `checkSortKey` rejects an `Instant` sort key with *"Key must have an
  order-preserving data type"* — while `StandardRelationTypeMaker.sortKey`'s javadoc offers *"a property
  key with a timestamp data type"* as its canonical example. The fix is a new 12-byte
  `[8-byte biased seconds][4-byte biased nanos]`, not a marker interface, because the nanos field rides
  `VariableLong`'s non-order-preserving zig-zag.
- **`UUID` is order-preserving against a different comparator, on purpose.** Both `write` and
  `writeByteOrder` are two **unbiased** `putLong`s, so bytes sort by unsigned bits while `UUID.compareTo`
  compares each half signed: **10,070 of 20,000** random pairs disagree, **0** disagree with string order.
  `AttributeUtils` (in **`janusgraph-driver`**, not core) holds `compareValues` — natural order except
  UUIDs, which go to `compareUnsigned` — and that is the comparator the bytes implement. Lessons 21 and 26
  need this: commit `46f9d249` (fixes #5087) pointed `OrderList`, `RelationComparator`, `RangeInterval`
  and `Cmp` at it, and says explicitly that TinkerPop's own comparisons still use `UUID.compareTo`.
- **Three comparators, three disagreements.** The bytes implement `compareValues`; `AttributeUtils
  .compare` is lenient and diverges on exactly three things — `-0.0` vs `0.0` (returns 0; bytes are
  strictly ordered), a one-NaN pair (returns `null`; bytes put NaN last), and cross-numeric-type
  comparison (works via `doubleValue()`; `compareValues` throws `ClassCastException`).
  `RelationComparator.compareValues` is a third entry point. **Which one answers a given predicate is
  lesson 21's job** — that is where this becomes a bug or stops being one.
- **`String` has two encodings that share no code.** Byte order: `[0x00 flag][2 bytes per UTF-16 code
  unit][00 00]` = `3 + 2n`, no compression, no length header; it is exactly `String.compareTo` order
  (0 inversions over 73,606 random strings) and therefore **UTF-16 code-unit order, not code-point
  order** — U+1F600 sorts before U+FFFF, U+10000 before U+E000. Value order: one
  `VariableLong.writePositive` header with the compressor id in the low 3 bits, the full-UTF flag in bit
  3, and a length-or-marker above — ASCII writes marker `2 << 4` (or `1 << 4` for empty) and **stores no
  length at all**, recovering it from the `0x80` stop bit; full UTF stores the **char** count; GZIP stores
  the **byte** count and triggers only above `LONG_COMPRESSION_THRESHOLD = 16000`. `'alice'` is 13 bytes
  as a sort key and 6 as a value.
- **The sort-key premium.** Zero for every fixed-width type; `Integer` 5 vs 2 for small values; `String`
  `3 + 2n` vs `1 + n`. Carry this into lesson 17's graphindex cost comparison and lesson 20's VCI
  arithmetic: a 20-character string sort key on an edge label with two vertex-centric indexes is
  **258 bytes of sort key for one edge** once lesson 10's `2 + N` columns and both endpoint rows are
  counted.
- **Enums are persisted by ordinal and the format does not record declaration order.** 17 internal enums
  (`TypeDefinitionCategory`, `JanusGraphSchemaCategory`, `Multiplicity`, `Cardinality`, `SchemaStatus`,
  `ConsistencyModifier`, `Mapping`, `Order`, `Direction`, …) ride `EnumSerializer`. `getValue`'s
  `ordinal < values.length` check catches a **removal**, loudly; it cannot catch a **swap**. Lesson 13
  owns the consequences.
- **The registration table is an on-disk format with 28 slots left.** Ids in use: `1`, `10`-`21`,
  `22`-`30`, `41`-`57`, `60`-`71` = **51**, gaps at `2`-`9`, `31`-`40`, `58`-`59`.
  `CLASS_REGISTRATION_OFFSET = 100` ("may NEVER EVER be changed"), `MAX_REGISTRATION_NO = 100000`, and
  `registerClass(n)` stores at `100 + n` — so **72 through 99** are the only internal slots left.
- **The custom-serializer extension point is add-only.** `registerClassInternal`'s
  `!handlers.containsKey(datatype)` precondition makes replacing a built-in serializer impossible through
  configuration, with **one** hard-coded exception at the top of `registerClass` for `HashMap` (from #306,
  a Titan migration collision; registration 68 is `SerializableSerializer`, i.e. Java native
  serialization through `commons-lang3`, so a `HashMap` property value is an `ObjectInputStream` payload
  in the edgestore). Both `attributes.custom.attribute[X]` options are **`GLOBAL_OFFLINE`**
  (`GraphDatabaseConfiguration:1365-1375`), because the registration number is persisted.
  `GraphDatabaseConfiguration.getSerializer:1971` builds a **fresh** `StandardSerializer` per call, so the
  serializer is not a singleton and a stateful custom serializer is a hazard (cf. #2429).
- **`ObjectSerializer` (registration 1) is a placeholder, not a serializer**: `write` emits one `0x01`
  byte and `read` returns `new Object()`. It is never reached for a property value, because
  `hasGenericDataType` diverts those to `writeClassAndObject`.
- **`VariableString.writeBackward` puts `STRING_ID_MARKER` at the HIGH address** and the `0x80` stop bit
  at the low one, so `readBackward`'s `getByte(--position)` peek is correct and the string comes out in
  order without reversing. Closes lesson 11's assumption. There are **three** near-duplicate in-tree ASCII
  codecs — `StringSerializer.write`'s ASCII path, `StringEncoding.writeAsciiString`, and
  `VariableString.write` — differing on the marker and on the empty string (header-only / a lone `0x80` /
  rejected outright).

### Verified by lesson 11 — compressed; full block in git history at the 2026-11-09 commit

- **The bytes are not self-describing; the schema is part of the format.** `parseRelation:113-116` looks
  up `tx.getExistingRelationType(typeId)` and reads `multiplicity()`, `getSortKey()`, `getSignature()`
  and `getSortOrder()` off it **before** it can classify any byte after the prefix, and nothing in the
  cell records which schema version wrote it. So a sort-key or signature change is a storage-format
  change, and there is one type lookup per cell on every read. **Lesson 13 closed the recursion
  question**: the lookup bottoms out at `SystemTypeManager`'s static 17-entry map for system ids, and
  otherwise at a vertex constructor that dispatches on the id's suffix bits alone.
- **Ten (type × direction) rows, three parse shapes** (`→` forward, `←` backward):
  `MANY2ONE`@OUT / `ONE2MANY`@IN / `ONE2ONE`@either → col `P`, val `vid→ rid→` (**forward-only**);
  `MANY2ONE`@IN / `ONE2MANY`@OUT / `SIMPLE`@either → col `P vid←`, val `rid→` (**pivot at
  `valuePosition`**); `MULTI`@either → col `P S vid← rid←`, val signature+props (**full backward walk**);
  `SINGLE` property → col `P`, val `value rid→`; `SET` property → col `P value`, val `rid→`;
  `LIST` property → col `P S rid←`, val `value`+props. `Multiplicity.isConstrained()` is `this != MULTI`,
  so `SIMPLE` **is** constrained.
- **THE finding: the backward encoding is order-preserving and the forward one is not.**
  `writeUnsignedBackward` puts the length in bits 4-6 of the **first** byte, so a longer encoding sorts
  after a shorter one — **zero inversions over `0..400000`**, against one per length boundary in
  `writePositiveWithPrefix`. That, not readability, is why the tail ids are written backward: it is what
  makes `ImplicitKey.ADJACENT_ID` and `JANUSGRAPHID` usable as extended sort-key positions in
  `getQuery:466-496`, **lesson 20's starting point**. Capacity `4 + 7·(numBytes-1)` bits with a hard
  3-byte floor; **+12%** over `NormalVertex` ids at counts 16..160,000,000; a default `MULTI` edge is an
  **8-byte column with an empty value**. Lesson 27 should compare against CQL's clustering-key encoding.
- **`valuePosition` is the only framing in the cell, and it is an origin as well as a boundary.** On the
  pivot shape the parser jumps to it, reads the vertex id backward out of the column, jumps back, and
  reads the relation id forward out of the value. `readPositiveBackward` leaves the cursor on the stop
  byte, so `endKeyPos` is **exclusive** and `:169`'s comment is off by one in wording only.
- **`RelationCache` is a three-state memo and the middle state is a `null`.** `properties == null` →
  header-only; shared static `EMPTY` → fully parsed, found none; non-empty map → fully parsed with
  properties. #4995 was the collision between the second and first states through `hasProperties()`, and
  `29c0b18a` fixed it with a third state. **Lesson 19 needs this**: the header-only state is what
  `isLazyLoadRelations()` and `JanusGraphLazyRelationConstructor` trade on. `EMPTY` is a shared
  **mutable** `LongObjectHashMap`, which is why `propertyIterator()` refuses to iterate it.
- **The completeness test, and why index replicas are permanently excluded.** `:201-202` returns a fully
  parsed cache from a header-only parse iff `!in.hasRemaining() && keySignature.length == 0 &&
  def.getSignature().length == 0 && !hasPropertyMetaData(data)`. Clause 2 is a real test for the
  unconstrained shapes, so **every vertex-centric index replica fails it by construction**.
  `EdgeSerializerTest` pins **17 of 50**. `hasPropertyMetaData` tests membership in
  `ImplicitKey.MetaData2ImplicitKey`, not `hasMetaData`, so `TIMESTAMP`/`TTL` keep a parse header-only
  while a grouped CQL read's `ROW_KEY` does not — **lesson 27 should check that against the CQL reader**.
- **The memo is per-`Entry`-object and dies with it** (`transient volatile`, no CAS), and
  **`RelationReader` declares only `parseRelation`** — the entry cache lives on
  `EdgeSerializer.readRelation`, which is not on the interface. That is the structural cause of PR
  candidate 18 and the reason the #4995 fix cannot reach the OLAP path.

### PR candidates, all verified at the pinned commit

Numbering is **stable** — earlier entries reference these numbers, so new candidates are appended rather
than inserted in size order. **Candidates 24-25 are new this run** (lesson 13), and candidates 21-23 came
from lesson 12, which also **rewrote** candidate 20 rather than dropping it. **18 is still the most
substantial the track has found; 21 and 24 are now the smallest and the best first contributions** — 21 is
doc-only and 24 is a two-line guard fix, and both force a maintainer to confirm a reading. Candidates 1-23
carry forward; 24-25 were verified at this pinned commit on 2026-11-11, 21-23 on 2026-11-10, 18-20 on
2026-11-09, 15-17 on 2026-11-08, 11-14 on 2026-11-07, 8-10 on 2026-11-06, 1 and 7 on 2026-11-05, 2-4 and 6
on 2026-11-04, and 5 still needs its experiment run.

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
11. **NEW, and the smallest the track has found: `ConsistentKeyIDAuthority.java:420-421`** formats the
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
    copy on its first property read.
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

24. **NEW, and tied with 21 for the smallest: the dead disjunct in
    `SystemTypeManager.throwIfSystemName`.** The only validation a user-supplied type name passes
    through is
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
    certainly out of scope; **propose the test first and say so in the PR body.**

### Still open, and who owns it

- **Closed by lesson 11:** lesson 10's carried question (the layout does not decide completeness; the
  four-way test at `:201-202` does, and index replicas are excluded by clause 2 forever); why the tail
  ids are written backward (order preservation, not readability — zero inversions over 400,000 values);
  `endKeyPos`'s exclusivity and the off-by-one in `:169`'s comment; the `TIMESTAMP`-vs-`ROW_KEY`
  asymmetry (`hasPropertyMetaData` tests `ImplicitKey.MetaData2ImplicitKey` membership, not
  `hasMetaData`); the `RelationCache` identity assertions (`assertSame` throughout
  `EdgeSerializerTest.checkParsedOnce`, which pins that a full parse is the one instance every later read
  returns); `readInlineTypes`/`hasPropertyMetaData`; and that `SIMPLE` is constrained, correcting lesson
  10's arrow on that row.
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
- **Owed to lesson 14 by lesson 13:** `schemaCache` is assigned at `StandardJanusGraph:308` from
  `configuration.getTypeCache(typeCacheRetrieval)`, and `typeCacheRetrieval` opens its own transactions
  with `getKeyConsistentTxConfig()` — so establish the constructor's exact ordering and which components
  must exist before `system_properties` can be read. Also whether `MetricInstrumentedSchemaCache` changes
  any of lesson 13's cache behaviour or only counts it, and what `FIXED` / `GLOBAL_OFFLINE` / `GLOBAL` /
  `MASKABLE` do differently at open time (lessons 09 and 12 were each bitten by a `FIXED` option).
- **Owed to lesson 17 by lesson 13:** the eight **edge** `TypeDefinitionCategory` values (ordinals 17-24)
  are the whole of what hangs off `SchemaDefinitionEdge`. Establish which carry a real modifier
  (`INDEX_FIELD` with `Parameter[]`, `CONNECTION_EDGE` with a `String` are the two with datatypes) and
  what `CompositeIndexTypeWrapper`/`MixedIndexTypeWrapper` read out of `getRelated`. Cost baseline to
  cite: a property key is 11 edgestore columns plus 2 graphindex entries, and every definition tag is two
  bytes.
- **Owed to lesson 18 by lesson 13:** `BaseKey.SchemaName`'s internal composite index declares
  `ConsistencyModifier.LOCK` and `getBackingIndexName() == Token.INTERNAL_INDEX_NAME`, so "one name, one
  type" rests entirely on `ConsistentKeyLocker`. Establish what `ExpectedValueCheckingStore` does with an
  internal index key, and whether two instances creating the same property key concurrently fail cleanly
  or leave two ids under one name. Also: `CONSISTENCY_LEVEL` and `TTL` live on separate `TYPE_MODIFIER`
  schema vertices reached by a definition edge, which is where the modifier machinery belongs.
- **Owed to lesson 25 by lesson 13:** `JanusGraphSchemaVertex.resetCache()` and
  `StandardJanusGraphTx.expireSchemaElement` are the invalidation half of a schema change, and
  `updateSchemaVertex` (which writes `SchemaUpdateTime`) is the only thing that marks one. Establish how
  `ManagementSystem` and `ManagementLogger` get from a committed schema change to an eviction on every
  other open instance, and whether `SchemaUpdateTime` is read by anything or only written.
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
- **Owed to lesson 16 by lesson 09:** `assignIDs(Iterable<InternalRelation>)`
  (`VertexIDAssigner:278-333`) is called from `StandardJanusGraph.prepareCommit`, and `ids.flush`
  (default `true`) decides whether ids are assigned at element creation or at commit.
  `VertexIDAssignerTest#testDisableIdsFlush:123` is the entry point. The bulk path's two-pass structure
  and its own 1000-attempt `leftOvers` loop belong there. **Add from lesson 10:** `prepareCommit`'s
  `wholeRowDeletion` short-circuit (`StandardJanusGraph:1112-1138`) skips `writeRelation` entirely for a
  fully-removed vertex when the backend advertises the capability — a serialization cost, not a
  correctness one, and the comment at `:1131-1135` says so.
- **Owed to lesson 18 by lesson 09:** `ConsistentKeyIDAuthority`'s comment at `:90-92` says its slice
  bounds mirror `ConsistentKeyLocker`'s; read the two protocols side by side. Also still owed:
  `ExpectedValueCheckingTransaction.prepareForMutations()`, `ExpectedValueCheckingStore`, `BerkeleyJETx`,
  `BerkeleyJELocks`, `LockKeyColumnValueStoreTest`. **Add from lesson 10:** `acquireEdgeLock` locks on
  the **serialized entry** (`StandardJanusGraph:900-901` passes the whole `Entry`; `:1004` passes only
  `entry.getColumn()`) — which is why the remaining-properties sort at `EdgeSerializer:369-371` is a
  correctness requirement, not tidiness. The asymmetry between those two call sites is worth a look.
- **Owed to lesson 19 by lesson 08:** the partitioned-vertex read cost — one logical vertex, up to
  `2^partitionBits` representative rows, one slice query each. Plus, carried from lesson 04 and sharpened
  by lesson 07: the `getNoCopy` aliasing question belongs in `KeyColumnValueStoreTest` and must mutate
  through a `CacheTransaction`; the `MemoryEntryList` sizing question and the
  `InMemoryColumnValueStore.java:162` NPE candidate also land here. `MultiPageEntryBuffer`,
  `BufferPageUtils` and `BufferPage.merge` remain deferred with no owner; propose lesson 19.
- **Owed to lesson 22:** `InMemoryKeyColumnValueStore.RowIterator` and
  `OrderedKeyValueStoreAdapter.KeyIteratorImpl` (`L299+`, shared mutable `current`, per-key limit at
  `L381`) plus `SimpleScanJob`.
- **Owed to lesson 25 by lesson 10:** the `SchemaStatus` gate at `StandardJanusGraph:1125`
  (`DISABLED` index types are skipped at write time) and `ManagementSystem:365-390`'s
  `canIndexBeEnabled` / `REGISTER_INDEX` branch are the write-side half of the index lifecycle. Also:
  changing a multiplicity rewrites the column layout and there is **no migration in `writeRelation`** —
  whatever protects a user from that lives in lesson 25's machinery.
- **Owed to lesson 27 by lesson 09:** the ids-store column layout — `[-(end)][timestamp][uid]` with an
  empty value, slice bounds `zeroBuffer(1)` to `oneBuffer(17)` — is the smallest real column layout in
  the codebase and the cleanest thing to compare against `CQLKeyColumnValueStore`. Hands-On §2's byte
  dump is specified for it. Also the count stride of 16, which affects CQL token distribution. **Add from
  lesson 10:** the edgestore column layout is now fully specified, so lesson 27 can compare CQL's
  clustering-key encoding against the six-band prefix directly.
- **Carried from sessions 01-04:** 24 questions posed, none answered by Aryant. The half-open interval now
  has **three** implementations on record (`SinglePageEntryBuffer.getSlice` binary search;
  `BerkeleyJEKeyValueStore.java:258`'s per-record `compareTo`;
  `OrderedKeyValueStoreAdapter.java:294-297`'s `columnInRange`) and two consumers relying on the upper
  bound being exclusive (`BackendTransaction`'s `EDGESTORE_MAX_KEY`, and `ConsistentKeyIDAuthority`'s
  `UPPER_SLICE = oneBuffer(17)`). **Lesson 10 adds a third consumer:** every `getQuery` slice end is
  `nextBiggerBuffer`-exclusive, and the missing-value marker relies on exclusivity on the ASC upper bound
  and the DESC lower bound. Lesson 27 adds CQL as the fourth implementation. The contract is in
  `KeyColumnValueStore`'s javadoc and **none** of the enforcement is.
- **tdd.cat, 2026-11-11: the front page ran AHEAD of the archive, which is new.** At ~03:40 UTC the
  front page served **2026-11-10** (63 stories, heading *"Tuesday, November 10, 2026"*) while
  `/archive/` still reported **134 editions** with **2026-11-09** as its newest *link* and the November
  10 cell unlinked. So for the first time in the five runs that have tracked this, the maximum came from
  the **front page**, not the archive — the previous four runs all had the archive equal to or ahead of
  the front page (lesson 09 found 2026-11-06 in the archive while the front page served 2026-11-05).
  **The procedure still works and is still the right one, but state it correctly now:** (a) fetch both
  the front page and `/archive/` and take the **maximum of the two**, in either direction; (b) before
  digesting, check this session log for the newest edition already covered — lesson 12 digested
  2026-11-09, so 2026-11-10 was the uncovered one; (c) the uncovered-item fallback is the last resort,
  and the 2026-11-06 edition has two items spent (Infino ids, Postgres `SKIP LOCKED` queues). Two
  practical notes: the front page is now large enough (~107 KB on 2026-11-10) that **one WebFetch covers
  only about 100 KB** and the tail of the item list needs a second call with an offset; and edition sizes
  remain volatile — 22 stories on 2026-11-08, 50 on 2026-11-09, 63 on 2026-11-10 — so a small edition is
  not evidence of a partial publish. The filter-bar rendering bug was not observed this run.

## Paused Subjects

### PostgreSQL Internals — paused 2026-10-30 after lesson 10 of 35

Paused at Aryant's request to make JanusGraph the active track. **Nothing was dropped.** The full
PostgreSQL track state — syllabus with lessons 01–10 checked, the handover note block, environment
recipes, and archive pointers — is preserved verbatim in
[`tracks/postgresql.md`](tracks/postgresql.md). When the JanusGraph track completes, that file
becomes `TRACK.md` again and the next run resumes at **lesson 11 — the heavyweight lock manager**.

## Completed Subjects

### SQLite Internals — 33 lessons, 2026-09-16 to 2026-10-18 (capstone 2026-10-19)

Sequential deep dive through `btree.c`, `pager.c`, `wal.c`, the compiler and the VDBE, in five
parts: the on-disk format and the b-tree layer (01–13), the pager, journals and VFS (14–18),
write-ahead logging (19–20), the SQL compiler and bytecode engine (21–28), and operational and
advanced topics (29–33). Capstone:
[`2026/10/2026-10-19-sqlite-capstone/README.md`](2026/10/2026-10-19-sqlite-capstone/README.md).

The full 33-item syllabus, with every lesson title as taught, is preserved in this file's git
history — the last commit carrying it is the one recorded in
[`TRACK-ARCHIVE.md`](TRACK-ARCHIVE.md) under row "31 – 33". The lesson documents themselves are
indexed in the root [`README.md`](README.md), which is the real record of what was taught.

**Closing summary.** One file, one writer, a bytecode interpreter in the middle, tests instead of
types. The commit point is a filesystem operation (`unlink("-journal")` in rollback mode, a frame
header with a nonzero commit counter in WAL mode). The planner's cost channel is 10 bits wide.
And `quick_check` is a *weaker* check at nearly the same I/O as `integrity_check`, blind to index
key order — the single most useful operational fact the track produced.
