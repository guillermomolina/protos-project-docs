# DOC021 — Receiver-specific collection capabilities: publication and acceptance (2026-11-19)

**Role:** Immutable, revision-bound documentation publication and owner-validation evidence, not normative language or implementation authority.  
**Owning issue:** [DOC021 / guillermomolina/protos#1089](https://github.com/guillermomolina/protos/issues/1089).  
**Maintained work record:** [DOC021 collection-protocol documentation scope](../../work/DOC021/DOC021_COLLECTION_PROTOCOL_DOCUMENTATION_SCOPE_2026_11_07.md).  
**Published product revision:** [`guillermomolina/protos@2f03ce820b85f0fff9ec12c67cc58b4023e106eb`](https://github.com/guillermomolina/protos/commit/2f03ce820b85f0fff9ec12c67cc58b4023e106eb).  
**Commit subject:** `DOC021: Document receiver-specific collection capabilities`.  
**Product version at this revision:** `0.4.178-SNAPSHOT` (unchanged; documentation-only publication).  
**Prior published traversal implementation:** [I162 / `c30c7dc42424a34a6a6fd8c41987316030d2992c`](https://github.com/guillermomolina/protos/commit/c30c7dc42424a34a6a6fd8c41987316030d2992c), implementing owner-ratified D252-C1.

## Product commit inspection

Read-only GitHub commit inspection confirms exactly **five changed paths**:

1. `docs/guide/05-values-identity-equality-and-collections.md` — reader-facing receiver-specific capability matrix with the indexed/shape/traversal/growth/fresh-result/state/equality axes for Array, Bytes, Map, IdentityMap, Map-backed Set and IdentitySet roles, Range/InvertedRange, Vector, Matrix, String, Process Environment, and explicitly projected foreign containers. Adds the distinct-result and non-generic-container explanations and the runnable tutorial link.
2. `docs/guide/04-control-flow-through-protocols.md` — concise ordinary `for` / receiver-selected `each` clarification: no shared arity, result, collection class or universal snapshot promise.
3. `docs/guide/README.md` — guide-to-tutorial navigation.
4. `protos/tutorials/05-collections/05-collection-capabilities.protos` — new licensed standalone Protos-first tutorial exercising fixed Array replacement versus fresh append, keyed Map insertion, resizable Bytes, Set/IdentitySet roles, mathematical Vector/Matrix addition, Unicode-scalar String traversal and computed Range traversal; fails by signalling `Error` if its combined checks are false.
5. `protos/tutorials/README.md` — collection tutorial progression/navigation.

The diff shows **no change to `src/`, `spec/`, `protos/lib/`, Java classes, parser, runtime, public APIs, `pom.xml`, or `CHANGELOG.md`**. It introduces no independent `Collection` / `Iterable` / `Iterator` / `Stream` family, Array resizing operation, Matrix bracket indexing, foreign iteration snapshot rule, new Standard Library API, or alternative dispatch path.

The pre-existing [linear-algebra guide](https://github.com/guillermomolina/protos/blob/2f03ce820b85f0fff9ec12c67cc58b4023e106eb/docs/guide/library/linear-algebra.md) needed no DOC021 edit: the published I162 work had already established algebraic `add`, fixed shapes, explicit `Matrix.at(row,column)`, standard `each` and snapshot semantics. DOC021 does not duplicate or reverse that implementation.

## Semantic and historical reconciliation

The allocation-time owner approval of AUD023 Alternative B remains historical provenance. The owner subsequently selected C and ratified D252-C1, and I162 published standard `String.each`, `Vector.each` and `Matrix.each` before this documentation slice. DOC021 correctly documents those **already-published** capabilities rather than claiming their absence or making new language decisions.

The matrix distinguishes:

- Mutable indexed replacement, in-place structural growth/removal, and fresh-copy results rather than treating similarly named `add` operations as one law.
- `closed` versus `frozen` object state and String intrinsic immutability, including frozen argument/rest Arrays.
- Equality-keyed Map/Set membership (`hash` and `==`) versus identity-keyed IdentityMap/IdentitySet membership (`identityHashOf` and `===`); Sets remain ordinary backing Maps rather than a separate runtime family.
- Snapshot traversal in standard Array/Bytes/Map/Vector/Matrix, generated Range traversal returning `null`, lexically ordered read-only Environment iteration, and provider-specific pull-based projected foreign iteration.
- The absence of generic indexing/shape/iteration laws and the deliberate exclusion of Path and StyledText from general-purpose collections.

Normative owners remain [Values and Collections](https://github.com/guillermomolina/protos/blob/2f03ce820b85f0fff9ec12c67cc58b4023e106eb/spec/semantics/VALUES_AND_COLLECTIONS.md), [Execution and Control](https://github.com/guillermomolina/protos/blob/2f03ce820b85f0fff9ec12c67cc58b4023e106eb/spec/semantics/EXECUTION_AND_CONTROL.md), [Process Environment](https://github.com/guillermomolina/protos/blob/2f03ce820b85f0fff9ec12c67cc58b4023e106eb/spec/io/PROCESS_IO.md), and the product grammar. These documentation changes do not supersede them.

## Acceptance provenance and limitations

On 2026-11-19, after publishing the exact DOC021 commit, the maintainer explicitly reported:

- `git diff --check`: **clean**.
- **All local tests: PASS**.
- The named DOC021 commit was **pushed**.

The published product commit and its five-path scope were independently verified by read-only GitHub inspection. Local execution success and diff validation are the **maintainer's attestations**; a command-by-command test transcript, precise suite count, separate CI run identity, and independent Native-image execution were not supplied. Do not fabricate them or make a redundant testing requirement of this already-accepted documentation-only slice.

DOC021's standalone deliverables are satisfied. [AUD023](https://github.com/guillermomolina/protos/issues/1087) remains historical motivation, and [I162](https://github.com/guillermomolina/protos/issues/1099) remains its separate completed implementation. This record is evidence for closing [DOC021 #1089](https://github.com/guillermomolina/protos/issues/1089) as **completed**, after publication of this project-record change.

## Evidence markers

```text
WORK_ITEM=DOC021
ISSUE=https://github.com/guillermomolina/protos/issues/1089
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_COMMIT=2f03ce820b85f0fff9ec12c67cc58b4023e106eb
PRODUCT_COMMIT_INSPECTED=YES
PRODUCT_VERSION=0.4.178-SNAPSHOT
CHANGED_PRODUCT_PATHS=5
NEW_PROTOS_TUTORIAL=protos/tutorials/05-collections/05-collection-capabilities.protos
JAVA_PRODUCTION_CHANGE=NO
SPEC_CHANGE=NO
STANDARD_LIBRARY_IMPLEMENTATION_CHANGE=NO
VERSION_BUMP=NO
FOR_EACH_LAW=RECEIVER_SPECIFIC
I162_CURRENT_TRAVERSAL=PUBLISHED
LOCAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
EXACT_TEST_TRANSCRIPT=NOT_PROVIDED
PROJECT_EVIDENCE_PUBLICATION=THIS_RECORD
ISSUE_DISPOSITION=COMPLETED_AFTER_PROJECT_RECORD_PUBLICATION
```
