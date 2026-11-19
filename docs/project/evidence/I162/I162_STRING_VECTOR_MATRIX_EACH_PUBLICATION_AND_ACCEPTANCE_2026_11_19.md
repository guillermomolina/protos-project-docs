# I162 — Standard String, Vector and Matrix `each`: publication and acceptance (2026-11-19)

**Role:** revision-bound implementation and validation evidence, not a new language decision or normative specification.
**Implementation issue:** [I162 / protos#1099](https://github.com/guillermomolina/protos/issues/1099).
**Ratified authority:** [D252-C1 / protos#1098](https://github.com/guillermomolina/protos/issues/1098); [decision record](../../decisions/language/D252_COLLECTION_TRAVERSAL_STRING_VECTOR_MATRIX_C1_RATIFIED.md) and [owner approval/invariant evidence](../D252/D252_C1_OWNER_APPROVAL_AND_INVARIANT_RECONCILIATION_2026_11_09.md). D252 is an already-closed design decision; this record documents its subsequent implementation.
**Exact published product revision:** [`guillermomolina/protos@c30c7dc42424a34a6a6fd8c41987316030d2992c`](https://github.com/guillermomolina/protos/commit/c30c7dc42424a34a6a6fd8c41987316030d2992c).
**Commit subject:** `I162: Implement standard String, Vector and Matrix each traversal`.
**Product version in this commit:** `0.4.178-SNAPSHOT` (`pom.xml`, with matching `CHANGELOG.md` entry).
**Normative specification revision:** `0.1.504` (`spec/PROTOS_SPEC_CHANGELOG.md` and `spec/semantics/VALUES_AND_COLLECTIONS.md`).

## Published scope, independently checked against the exact commit

The published I162 commit modifies **26 paths**. The five modified Java production files are:

- `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardStringProtocol.java` — public standard `String.each`.
- `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java` — public standard `Vector.each` and `Matrix.each` on the module-owned prototypes.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/structured/ProtosStructuredEachCalls.java` — prepared iteration state, Unicode-forward cursor and logical numeric snapshots.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosPreparedCalls.java` — exact standard-body capability recognition and preparation.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java` — extend the existing `IsStructuredLocalEach` and `PrepareStructuredLocalEach` admission/selection.

No new standalone production Java class or independent callback execution engine is introduced by this commit. A nested prepared state `PreparedValueEachCall` reuses the existing structured phases and `ProtosCallRequest` callback dispatch. The existing `ProtosStructuredDispatchLowerer` is not changed. The design preserves ordinary message selection/overrides while standard-body execution uses the Truffle Bytecode DSL path rather than guest element-access sends or a Java callback `for` loop.

Observable C1 contract: `String.each` yields one semantic String per Unicode scalar (including astral symbols, combining marks and the three U+200D scalars in the family emoji); `Vector.each` yields ascending Numbers from one shallow pre-callback snapshot; `Matrix.each` yields row-major Numbers from one logical snapshot independent of R1/R2-Float representation. All three validate receiver, arity and polymorphic callability, including on empty receivers; invoke a callback with one argument per element; ignore ordinary callback results; and return the exact original receiver. Error, non-local return, `ensure`, suspension/reentrancy, cancellation and normally returned Future follow existing invocation/control contracts. Generic `for` remains ordinary `each` lookup and respects overrides. No universal Iterator/Collection API, change to `Vector.add`/`Matrix.add`, grammar change or Matrix bracket indexing is claimed.

## Published tests, guides and integration

- Conformance: new `protos/tests/conformance/collections/string-iteration.protos`, updated `protos/tests/conformance/control/for-contextual-syntax.protos` and `protos/tests/conformance/manifest.tsv`.
- Algebra: new `protos/tests/library/math/linear-algebra/traversal.protos`, registered by `protos/tools/test/RepositoryCorpusPlans.protos` and its Java corpus-plan regression.
- Java callback/architecture tests: new `ProtosStandardValueEachIterationTest.java`, updated `ProtosI109ASharedLocalEachLoweringTest.java`, `ProtosCoreNativeBoundaryArchitectureTest.java`, and String/linear-algebra protocol tests. Coverage includes exact structured CallTarget recognition, published synchronous message entry, polymorphic callability, Unicode scalar visits, R1/R2-Float, snapshot mutation, control/suspension and guest `at`/`size`/`rows`/`columns` overrides that must not be invoked by standard traversal.
- User-facing material: control-flow and collection guides, linear-algebra guide and module documentation, and both existing vector/matrix tutorials. Specification changes are normative only in the product repository; this evidence is descriptive.
- Version and release-notes metadata: `pom.xml` and root `CHANGELOG.md` updated as part of the published commit. A public release or Native image is **not** implied by a SNAPSHOT version bump.

## Local validation provenance and scope

**Maintainer-supplied pre-publication logs:**

- A focused Maven run of `ProtosStandardValueEachIterationTest` reported **12/12 tests PASS** and `BUILD SUCCESS`; a separate single-test Maven run reported **1/1 PASS**.
- The selected Protos corpus output reported **129 passed, 0 failed**. This was a local corpus checkpoint, not a separately verified Native run.
- After the next regression edit, `mvn -Dtest=ProtosStandardValueEachIterationTest,ProtosI109ASharedLocalEachLoweringTest test` reported **20/20 tests PASS** (13 callback + 7 architecture), zero failures/errors/skips and `BUILD SUCCESS`, completed **2026-11-19T13:01:46Z**. The later Unicode and guest-access override tests were added after that focused transcript; do not misattribute the 20/20 execution to the final test source.

**Final maintainer acceptance after the published I162 commit (2026-11-19):** the owner explicitly reported that `I162: Implement standard String, Vector and Matrix each traversal` had been pushed, **`git diff --check` was clean**, and **all local tests passed**. This attestation covers the final local work as reported by the executor; no additional build or run is required merely to repeat the acceptance. The exact published commit, its file list, version and normative revision are independently verified using GitHub read-only access.

**Evidence limit:** no full post-publication command-by-command output, final exact test count, CI run identity, independent Native-image/installed distribution proof, or public release artifact was supplied. Do not invent those details; their absence does not negate the explicit maintainer acceptance of all local tests.

## Coordination and adjacent work

- I162 completes the already-approved D252-C1 implementation. GitHub's native parent of I162 is D252 #1098; no replacement decision or child implementation issue is needed.
- [DOC021 #1089](https://github.com/guillermomolina/protos/issues/1089) remains a **separate open documentation-only** cross-family capability-matrix task. I162 publishes its own affected guides, but does not by itself prove DOC021's wider table/examples complete. DOC021's old baseline statement that these families lacked `each` is historical and must be read against the published `c30c7dc4` state.
- Keep D252's 2026-11-09 original decision and baseline evidence as historical truth: its at-ratification implementation-pending statement was correct at that time.

## Compact evidence markers

```text
WORK_ITEM=I162
OWNER_DECISION=D252_C1_RATIFIED
ISSUE=https://github.com/guillermomolina/protos/issues/1099
NATIVE_PARENT=D252_1098_VERIFIED
PROTOS_REVISION=c30c7dc42424a34a6a6fd8c41987316030d2992c
PROTOS_REVISION_VERIFIED=PASS
PRODUCT_VERSION=0.4.178-SNAPSHOT
SPEC_CHANGELOG_REVISION=0.1.504
CHANGED_PATHS=26
NEW_STANDALONE_PRODUCTION_JAVA_CLASSES=0
SHARED_STRUCTURED_LOWERER_MODIFIED=NO
FOCAL_JAVA_CHECKPOINT=20_20_PASS_PRE_FINAL_EDIT
CONFORMANCE_CHECKPOINT=129_PASS_0_FAIL_PRE_PUBLICATION
FINAL_LOCAL_TESTS=PASS_OWNER_REPORTED
FINAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
EXACT_POST_COMMIT_FULL_SUITE_TRANSCRIPT=NOT_PROVIDED
INDEPENDENT_NATIVE_IMAGE_PROOF=NOT_PROVIDED
ISSUE_DISPOSITION=COMPLETE_ON_OWNER_LOCAL_TEST_ACCEPTANCE
DOC021_DISPOSITION=INDEPENDENT_OPEN_SCOPE
PUBLIC_RELEASE=NOT_CLAIMED
```
