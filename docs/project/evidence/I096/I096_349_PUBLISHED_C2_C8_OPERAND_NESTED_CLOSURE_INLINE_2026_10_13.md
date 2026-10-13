# I096 — Published C2–C8, operand-position and nested-Closure ordinary-call inlining (0.3.349)

**Evidence date:** 2026-10-13

**Formal implementation owner:** [I096/#884](https://github.com/guillermomolina/protos/issues/884)

**Architecture authority:** [PLAT057/#881](https://github.com/guillermomolina/protos/issues/881), owner-ratified Candidate B

**PROTOS_REVISION:** `ffd6da2fb265f4d999ee26ddbfc19369f0f0403a` — [published implementation commit](https://github.com/guillermomolina/protos/commit/ffd6da2fb265f4d999ee26ddbfc19369f0f0403a)

**PRODUCT_VERSION:** `0.3.349-SNAPSHOT`, verified independently in `pom.xml` and `CHANGELOG.md` at the exact product revision.

**Immediate product parent:** `ac7c5f65800fc558843e6060cc3bc5e7c356cfbb` (I095-C, 0.3.348).

**Previous I096 publication:** `ff57d22b6fa1abdc252200c3c114d015bb80a315` (I096-A/B1, 0.3.347).

**Validation attribution:** human executor reports **all local tests PASS** and **`git diff --check` clean** after publication. This coordinating agent did not execute tests, native builds, benchmark runs or Protos product writes.

## Revision-pinned published implementation

The exact commit modifies eight files: `CanonicalToBytecodeLowerer.java`, `ProtosSemanticBytecodeRootNode.java`, `ProtosI096InlineClosureLiteralCallTest.java`, `ProtosI095BooleanControlInlineTest.java`, two PERF026 callback regression tests, `pom.xml`, and `CHANGELOG.md`.

- **Exact positional C0–C8:** `MAX_INLINE_CLOSURE_CALL_ARGUMENTS = 8` bounds expansion; the existing C0/C1 direct fast path remains limited to 0/1. The admitted in-root C2–C8 body uses once-staged receiver and positional arguments through the existing variadic carrier. C9+ and unproved defaults/rest/spread keep the ordinary physical call.
- **Binding calls at operand sites:** `collectBindingClosureCallCandidates` traverses supported operand positions evaluated by the same activation: call/send operands, create/assignment operands, indexed assignment, member, identity, inequality, return. It recognizes binding **identity**, not spelling, and does not traverse unrelated nested Closure/Object scopes, sequences or unsupported map/compose shapes. Ordinary `Object.call` selection remains run-time guarded by `AdmitsInlineClosureLiteralCall`; guard misses use the already prepared physical fallback without re-evaluating operands.
- **Bodies that create nested Closures:** these are no longer categorically rejected by `containsClosure`. `requiresPersistentFrameAuthority` selects the existing non-frame-native named-activation path; `LoadInlineCallbackActivation` makes the callee's own semantic activation available, and ordinary nested `MaterializeClosure` captures that activation **by reference**, including for children that escape after the physical caller returns. The nested Closure retains its own CallTarget: it is the admitted **outer** Closure call that loses a physical callee entry.
- **Group Root reuse and reparse:** `currentInlineClosureCallBody` and identity set `closureRootsLoweredThisParse` prevent repeated nested-root emission inside inline copies while resetting membership on each parse/reparse. `capturedOwnerBytecodeLocal` no longer treats the inlined callee's semantic scope as a physical owner frame; the runtime lexical-authority path supplies the owner.
- **Large Integer regression:** inside an admitted callback region, `LargeIntegerLiteral` resolves the Core Prelude using `PrepareSendOne.callerPrelude`, not `TryDirectSendOne.callerPrelude`. This correction and `ProtosI095BooleanControlInlineTest.largeIntegerLiteralsInsideAdmittedBodies` are published **in 0.3.349**, not retroactively attributed to 0.3.348.
- **PERF026 compatibility:** B3/C1 source RootTag expectations account for additional legitimate in-root I096 regions. No guest-language specification or architecture decision changed.

## Published tests and evidence boundaries

The I096 test source at this revision contains structural bytecode assertions on `PrepareInlineClosureLiteralCall`, `AdmitsInlineClosureLiteralCall`, positional `LoadInlineClosureArgument`, fallback and `RootTag`, combined with physical stack inspection using Truffle frames and a `Probe`. The tests distinguish actual execution inside the calling Root from mere emission of an unused inline region.

Tested forms include C2 literal/binding, C3 own shadowing, C8 budget admission versus C9 fallback, operand evaluation exactly once, four independently executed operand inline sites, intervening reassignment, overridden `call` followed by restored canonical selection, and DEBUG's physical route.

New nested cases cover no escape, sibling mutable captures, escaped counters invoked after the physical caller has finished, different invocation identities, Context observation with remove/recreate, reentrant calls, inherited nonlocal ReturnHome, suspension/resumption without replay, bounded recursive expansion and rejected guard cases. Tests also inspect nested-definition root identity/count and stable `BytecodeRootNodes` reparse.

The human previously showed a **39/0 focal pass** before the later nested-Closure and regression-test additions. That historical count is **not** the final focal suite count. For the final published product the owner states all local tests PASS; no exact final count, command transcript, Native gate, `make check`, CI status, timing benchmark, or final-tier graph was independently provided. Do not manufacture those results.

## Semantics, exclusions and follow-up

PLAT057-B remains unchanged: D013 ordinary overridable `Object.call`, D179 PRESENT/ABSENT / remove/recreate and pre-RHS destination selection, one distinct semantic activation and Context per eligible call, shared capture-by-reference authority, receiver/`this`/`methodHome`, ReturnHome, Errors, `ensure`, continuation, Task/Actor boundaries, SourceSection/RootTag, NORMAL/DEBUG and physical fallback. No universal rematerialization engine or new generic synthetic debugger-frame architecture was introduced. The trivial case does not inherit a new mandatory guest Context, RootTask, RootActor, RootProcess or guest argument Array.

The published implementation is **source-level/Bytecode lowering**: an admitted caller physically executes the selected body without entering its callee Root, with ordinary fallback preserved. That is different from Graal compiler inlining of a separate CallTarget. No measured speedup, final-tier node reduction, CodeSize improvement or allocation reduction is claimed.

Remaining fallbacks include C9+, unproved defaults/rest/spread, dynamic method sends/extracted methods, changed `call` selection and unapproved ReturnHome/tooling shapes. They are intentional PLAT057-B limits, not evidence of missing functionality in this published patch.

**I096/#884 remains OPEN / READY** because its Issue also carries an independent compiled-graph/CodeSize/timing acceptance gate. [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) owns the canonical ordinary `primitive-closure-call` compilation/stability and quantitative cost attribution. [I095/#878](https://github.com/guillermomolina/protos/issues/878) and [PERF041/#863](https://github.com/guillermomolina/protos/issues/863) own Boolean-control policy and graphs; PLAT057's ratification is already closed. Earlier [I096 0.3.347 publication evidence](I096_PUBLISHED_GENERIC_CLOSURE_INLINE_VALIDATION_2026_10_13.md) remains historical and unchanged.

```text
FORMAL_IDENTIFIER=I096
PROTOS_REVISION=ffd6da2fb265f4d999ee26ddbfc19369f0f0403a
PRODUCT_VERSION=0.3.349-SNAPSHOT
PRODUCT_COMMIT_PUBLICATION=VERIFIED_GITHUB
PRODUCT_CHANGED_FILES=8
HUMAN_REPORTED_ALL_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
AGENT_EXECUTED_PRODUCT_TESTS=NO
AGENT_EXECUTED_PERFORMANCE_TESTS=NO
I096_C2_C8_OPERANDS=PUBLISHED
I096_NESTED_CAPTURE_ESCAPE=PUBLISHED
I096_NESTED_ROOT_REPARSE=PUBLISHED
LARGE_INTEGER_INLINE_REGRESSION=PUBLISHED
MATCHED_POST_I096_FINAL_TIER_GRAPHS=PENDING
I096_IMPLEMENTATION_PUBLICATION=COMPLETE_CHECKPOINT
I096_ISSUE_STATUS=OPEN_READY
```
