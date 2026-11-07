# AUD023 — Revision-bound collection protocol evidence (2026-11-07)

**Issue:** [AUD023 #1087](https://github.com/guillermomolina/protos/issues/1087).
**Exact product tree:** [guillermomolina/protos@08c27760eb3f85105277e5b1b1b159abbca67ebe](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe).
**Maintained audit interpretation:** [AUD023 cross-family assessment](../../work/AUD023/AUD023_COLLECTION_PROTOCOL_COHERENCE_AUDIT.md).
**Nature:** source-reading snapshot / documentation-only; **no executable test, build, benchmark, local validation or normative edit** performed for AUD023. The original product commit is a pre-audit immutable baseline, *not* an AUD023 implementation commit.

## Inventory method

1. Queried the full recursively listed tracked repository tree and inspected public Core, `protos/lib/collections`, `protos/lib/math/linear_algebra` and cross-cutting stdlib modules.
2. Inspected the owning normative collection, syntax, iteration, byte-I/O, Process Environment and foreign-projection sections, including explicitly absent capabilities; checked selected Java protocol installers and Protos source-backed implementations at the exact product revision.
3. Inspected representative retained conformance cases for Array, Bytes, Map, IdentityMap, Set, IdentitySet, Range, Vector and Matrix, and inventoried further test paths and consumer/tutorial examples. Where excerpts below name an assertion, this is evidence that the test *exists*, **not** that it passed in this task.
4. Reconciled against existing ratified project records rather than inferring a new proposal from a naming similarity.
5. Used upstream official reference documents for cross-language precedents and recorded disadvantages of copying any one language's hierarchy.

## Verified normative and parser authorities (all at the pinned product revision)

| Owner | Inspected contract and boundary |
|---|---|
| [`spec/PROTOS_GRAMMAR.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/PROTOS_GRAMMAR.md) | `Indexed Access`: one index expression lowers to `at`; indexed assignment to `atPut`, no indexed slot creation. Contextual `for` lowers to ordinary `each`/`eachStep`. Spread uses standard Array rules. |
| [`spec/semantics/VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/semantics/VALUES_AND_COLLECTIONS.md) | `Indexed Access Syntax`; ``Standard Array Indexed Semantics`, `Standard Array iteration`, `Standard Library Array copy/edit`, `bulk transformations`, `Rest argument collections`; `Range/InvertedRange`; `Vector/Matrix`; `String Indexing and Mutability`; `Complete standard Bytes sequence`; `Maps, Hashing, and Key Equality`; `foreign hash containers and iteration`. |
| [`spec/semantics/EXECUTION_AND_CONTROL.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/semantics/EXECUTION_AND_CONTROL.md) | Ordinary message-driven looping and callback-control baseline; no universal collection hierarchy. |
| [`spec/semantics/OBJECT_MODEL.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/semantics/OBJECT_MODEL.md) | Object identity, delegation and `open/closed/frozen`; no hidden collection inheritance merely from common names. |
| [`spec/semantics/MATCHING.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/semantics/MATCHING.md) | Array/Map structural matching does not imply structural `==` or automatic collection-family membership. |
| [`spec/io/BYTE_IO.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/io/BYTE_IO.md) | Bytes element is semantic Integer 0..255, explicit `add/removeAt`; no implicit resize in `atPut`. |
| [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/io/PROCESS_IO.md) | Process `Environment` immutable read-only snapshot, `get`, `contains`, ordered `each`, native-name identity; Process arguments are frozen ordinary Arrays. |
| [`spec/io/FILESYSTEM.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/io/FILESYSTEM.md) | `Path` immutable ordered components with structural value equality; not a general indexed collection. |
| [`spec/runtime/ABSTRACT_RUNTIME.md`](https://github.com/guillermomolina/protos/blob/08c27760eb3f85105277e5b1b1b159abbca67ebe/spec/runtime/ABSTRACT_RUNTIME.md) | Cross-check conceptual implementations only; **informative, not normative** under `AGENTS.md`. |

## Inspected concrete guest and host implementations

| Source path under pinned product | Specific evidence |
|---|---|
| `protos/lib/core/Array.protos` and `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardArrayProtocol.java` | Array's installed `construct`, `at`, `atPut`, `size`, `each`; no installed `add` or in-place resizing. |
| `protos/lib/core/Bytes.protos` and `src/main/java/com/guillermomolina/protos/execution/protocols/io/ProtosStandardBytesProtocol.java` | Bytes installs `construct`, `size`, `at`, `atPut`, `each`, `add`, `removeAt`; `add` validates open state and semantic octet. |
| `protos/lib/core/Map.protos` and `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardMapProtocol.java` | Map installs `at`, `atPut`, `containsKey`, `atIfAbsent`, `remove`, `size`, `each`, `match`. |
| `protos/lib/core/IdentityMap.protos` and `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardIdentityMapProtocol.java` | IdentityMap publishes the matching keyed selectors without Map's ordinary equality law. |
| `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardStringProtocol.java` | Core String defines its indexing/size and concatenation but no installed `each` or indexed mutator. |
| `protos/lib/core/Range.protos` and `protos/lib/collections/range/module.protos` | frozen canonical ranges; `each`/`eachStep`/`invert`/`reverse`; no indexed read or size; compatibility module exports the **same** Core prototypes. |
| `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java` and `protos/lib/math/linear_algebra/module.protos` | Vector publishes `size/at/atPut/add/scale/dot/copy`; Matrix publishes `rows/columns/at/atPut/add/scale/matvec/matmul/transpose/copy`. No `each`; `add` is algebraic. |
| `protos/lib/collections/array/module.protos` | source-first `map/filter/findIndex/reduce/sort/append/concat/slice/insertAt/removeAt/reverse/find/contains/any/all/take/drop` and parallel algorithms; forwards bulk calls to private implementation. |
| `protos/lib/collections/internal/ArrayBulk.protos` | `flatMap`, `zip`, `partition`, `distinct` with source snapshots, new Arrays; `distinct` imports **`std:collections/set`** (not IdentitySet) and uses `Sets.contains/add`. |
| `protos/lib/collections/set/module.protos` and `protos/lib/collections/identity_set/module.protos` | module `construct/contains/size/add/remove/each/union/intersection/difference/sameMembers/isSubset/isSuperset/isDisjoint`; returned values are ordinary Maps/IdentityMaps. |
| `src/main/java/com/guillermomolina/protos/execution/foreign/api/ProtosForeignProjectedOperations.java` | explicit optional projected `at`, `atPut`, `each` and call adapters; foreign iteration is not equivalent to Core Array snapshot. |
| `protos/lib/core/Path.protos`, `protos/lib/text/styled_text/module.protos` | semantically ordered internals but no general collection-facing public API; excluded from the universal interface proposal. |
| `protos/lib/uri/module.protos`, `protos/lib/http/router/module.protos`, `protos/lib/logging/memory_sink/module.protos` | consumers/composition built from standard Arrays/Bytes/Maps; no independent universal collection family inferred. |

## Retained test, tutorial and consumer anchors (not executed)

- Core Arrays: `protos/tests/conformance/collections/array-construction-and-identity.protos`, `array-indexed-read-and-size.protos`, `array-indexed-update.protos`, `array-iteration.protos`. Assertion examples include exact index domain, fixed-size replacement and snapshot behavior.
- Bytes: `protos/tests/conformance/bytes/mutation-and-growth.protos`, `indexed-read-and-size.protos`, `indexed-write.protos`, `iteration.protos`. The growth test rejects Float/negative/out-of-octet-range `add` and verifies removed octet/size.
- Keyed collections: `protos/tests/conformance/collections/map-core.protos`, `map-iteration.protos`, `map-mutability-state.protos`, `identity-map-core.protos`, `identity-map-iteration-and-lookup.protos`, `identity-map-mutability-state.protos`.
- Library Sets: `protos/tests/conformance/library/collections/set-core.protos`, `set-algebra-and-predicates.protos`, `identity-set-core.protos`, `identity-set-iteration.protos`; different equality laws, membership, mutation returns and absent removal.
- Ranges and `for`: `protos/tests/conformance/library/collections/range-core.protos`, `range-callbacks-and-control.protos`, `protos/tests/conformance/control/for-contextual-syntax.protos`; empty-range callback policy, `each` result, custom `each` expression and Map key binder.
- Numeric collection tests: `protos/tests/library/math/linear-algebra/construction-and-indexing.protos`, `identity-and-state.protos` and `src/test/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocolTest.java`; fixed-shape and frozen/closed validation.
- Array library: `protos/tests/conformance/library/collections/array-copy-edits.protos`, `array-bulk-transformations.protos`, `array-common-contracts.protos`, `array-queries-prefix-suffix.protos`. In `array-bulk-transformations`, numeric `1` and `1.0` are considered equivalent by `distinct` as required.
- Guide/tutorials: `docs/guide/04-control-flow-through-protocols.md`, `docs/guide/05-values-identity-equality-and-collections.md`, `docs/guide/library/linear-algebra.md`, `protos/tutorials/05-collections/03-array-copies-and-queries.protos`.
- Narrow consumer sample: `protos/lib/uri/module.protos` accumulates output in growable Bytes; `protos/lib/text/styled_text/module.protos` constructs fresh Arrays rather than pretending to mutate fixed-length Arrays.

## Exact current deltas and intentional omissions

1. **No Core Array resizing**; `Array(...old, item)` and `Arrays.append` build new objects. This is deliberately ratified.
2. **No generic Collection/Iterator/Stream**. Historical AUD009-E1 expressly retained this absence; D213 did not add one.
3. **No general `for` promise for indexed objects.** String/Vector lack `each`. Matrix has binary indexed methods but no multi-index bracket grammar.
4. **No generic `add` law.** Bytes insertion, Set membership addition and Vector/Matrix mathematical addition are separate protocols.
5. **No common `each` result/callback/snapshot law.** Map passes two values, Range returns null, foreign projected each pulls, ordinary custom each may choose its own result.
6. **No hidden Set family.** Any ordinary mutation of a Map-backed Set can change its observable representation; no runtime tag/protection was approved.
7. **No implicit deep equality** for ordinary identity-bearing mutable collections, and no special Array family membership from an arbitrary object implementing `size/at/each`.

## Historical authority checked in the durable project record repository

- [AUD009-B9 Array/Map/IdentityMap review](../../work/AUD009/AUD009_B9_ARRAY_MAP_IDENTITYMAP_REVIEW.md), live [#620](https://github.com/guillermomolina/protos/issues/620).
- [AUD009-E1 stdlib collections review](../../work/AUD009/AUD009_E1_STDLIB_COLLECTIONS_REVIEW.md), live [#646](https://github.com/guillermomolina/protos/issues/646). Historical record predates promotion of Range to Core; do not misread the old module shape as current.
- [LIB001 collections design](../../work/LIB001/LIB001_COLLECTIONS_DESIGN.md), especially Set/IdentitySet underlying Map invariant and deferred hierarchy.
- [D213 ratified library API](../../decisions/language/D213_COLLECTIONS_LIBRARY_API.md), live [#949](https://github.com/guillermomolina/protos/issues/949).
- [D214 ratified Vector/Matrix](../../decisions/language/D214_VECTOR_MATRIX.md), live [#955](https://github.com/guillermomolina/protos/issues/955).
- [D231 contextual for](../../decisions/language/D231_CONTEXTUAL_FOR_F2_PURE_SELECTED.md) and [D235 Range Core](../../decisions/language/D235_RANGE_CORE_PRELUDE_B1F_SELECTED.md).
- [D192 foreign each](../../decisions/language/D192_FOREIGN_EACH_NORMAL_COMPLETION_RESULT.md); Product `spec/semantics/VALUES_AND_COLLECTIONS.md` owns its current normative meaning.

## External primary-source comparisons (not Protos authority)

- [Updated Pharo by Example — Collections](https://files.pharo.org/books-pdfs/updated-pharo-by-example/2016-10-27-UpdatedPharoByExample.pdf) — fixed Array vs growable OrderedCollection.
- [Ruby 3.4 Enumerable](https://ruby-doc.org/3.4.1/Enumerable.html) — compositional algorithms atop `each`.
- [Python 3.15 collections.abc](https://docs.python.org/3/library/collections.abc.html) — separable Iterable/Sized/Sequence/MutableSequence/Mapping capability abstractions.
- [Swift RangeReplaceableCollection](https://developer.apple.com/documentation/Swift/RangeReplaceableCollection) — explicit optional structural replacement/insertion and removal.
- [Rust IntoIterator](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html) — traversal contract for `for` rather than assuming every indexed object iterates.
- [Java SE 25 Collection](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Collection.html) — optional mutators and the cognitive costs of `UnsupportedOperationException`.
- All analogies are informative; copying their models would need a separate owner-approved design.

## Evidence integrity / decision gate

- `PROTOS_REVISION=08c27760eb3f85105277e5b1b1b159abbca67ebe`
- `ISSUE_ID=AUD023/1087`
- `NORMATIVE_PRODUCT_CHANGED=NO`
- `SOURCE_AND_TEST_FILES_READ=YES`
- `TESTS_EXECUTED=NO`
- `BENCHMARKS_EXECUTED=NO`
- `SEMANTIC_CONTRADICTION_CONFIRMED=NO`
- `DESIGN_APPROVAL_CLAIMED=NO`
- `OWNER_REVIEW_OF_PROPOSED_CLASSIFICATIONS=PENDING`

This snapshot records what was inspected and what remains undecided. It does not substitute for the owner's review, formal issue postconditions or any later execution evidence.
