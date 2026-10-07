# PERF025 — PLAT044 lean inline callback preparation

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the PLAT044 B′ preparation refinement, validation provenance, and the boundary
between this completed slice and later callback lexical/frame-native lowering.

## Exact product publication

```text
PROTOS_REVISION=56f615628efb9c78fee06cbdba6ff21a952539f9
PROTOS_PARENT_REVISION=93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9
PROTOS_VERSION=0.3.159-SNAPSHOT
COMMIT_SUBJECT=PERF025: defer rich PLAT044 callback preparation
OWNING_ISSUE=guillermomolina/protos#758
RELATED_DECISION=PLAT044/guillermomolina/protos#766
BASE_IS_EXACT_PARENT=YES
```

The product publication is exactly one commit ahead of the preceding
`0.3.158-SNAPSHOT` small-Integer representation publication.

## Bounded objective

PLAT044 Candidate B′ already permits an eligible immediate literal callback to
retain a real fresh semantic Closure activation while its body executes as a
parser-inlined resumable region in the caller semantic Bytecode root, without a
distinct callback `RootCallTarget` / `FrameInstance`.

Before this PERF025 slice, the local Boolean, `whileTrue`, and synchronous
`each` paths still prepared a physical rich child call before asking whether the
literal callback was eligible for that inline representation:

```text
ordinary call selection
  -> rich PreparedClosureCall
  -> rich child Activation / guest Context / guest argument Array as required
  -> inspect prepared child for inline admission
  -> inline body when admitted
```

The bounded objective was to reverse that cost order without changing the
ordinary semantic selection authority or PLAT044 semantics.

## Published preparation shape

The runtime now uses a lean internal carrier:

```text
PreparedInlineLiteralCall
```

For an eligible source-backed Closure it retains only the state needed to decide
inline admission and to materialize the exact invocation later:

- the selected source-body `RootCallTarget`;
- compact source-call target arguments;
- return-home ownership information derived from the compact ABI; and
- an already-prepared physical fallback only when ordinary selection did not
  admit the lean source candidate.

The common successful shape is now:

```text
ordinary D013 call selection once
  -> lean PreparedInlineLiteralCall
  -> PLAT044 admission
  -> materialize one fresh semantic Closure Activation
  -> execute parser-inlined callback region
```

The physical fallback shape is:

```text
ordinary D013 call selection once
  -> lean PreparedInlineLiteralCall
  -> PLAT044 admission miss
  -> lazily construct exact PreparedClosureCall from retained selection
  -> unchanged structured/ordinary invocation
```

```text
D013_SELECTION_REPEATED_ON_FALLBACK=NO
RICH_PREPARED_CALL_BEFORE_INLINE_ADMISSION=NO
RICH_ACTIVATION_BEFORE_INLINE_ADMISSION=NO
GUEST_CONTEXT_BEFORE_INLINE_ADMISSION=NO
GUEST_ARGUMENT_ARRAY_BEFORE_INLINE_ADMISSION=NO
FRESH_SEMANTIC_CALLBACK_ACTIVATION=PRESERVED
PHYSICAL_PREPARED_CALL_ON_FALLBACK=PRESERVED
```

## Families covered

The same preparation boundary is consumed by the existing PLAT044 local
callback paths for:

```text
Boolean standard callbacks
Closure.whileTrue condition/body
Array.each
Bytes.each
Environment.each
IdentityMap.each
Map.each
```

The implementation introduces corresponding lean preparation operations in the
semantic Bytecode DSL and a fallback materialization operation used only after
an admission miss.

## Semantic invariants preserved

This slice does not privilege selector spelling or bypass ordinary callable
semantics. Ordinary `call` lookup/selection remains authoritative and existing
invalidation/fallback behavior is retained.

The published implementation preserves:

- canonical `call` overrides and alias-home behavior;
- exact selected Closure / method-home provenance;
- captured lexical state and receiver semantics;
- captured or owned ReturnHome semantics;
- fresh callback activation identity;
- Task and dynamic-control state;
- Error and non-local-return propagation;
- suspension/resumption and cancellation behavior;
- debugger scope projection and RootTag/source behavior;
- collection snapshot/prevalidation/order rules; and
- exact structured or ordinary physical fallback when inline admission fails.

```text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLAT044_DECISION_CHANGED=NO
GENERIC_FALLBACK=PRESERVED
```

## Deliberate boundary retained

This slice does **not** implement callback-specific lexical/frame-native
lowering. In particular, the existing PLAT044 callback lowering boundary that
does not propagate the enclosing root's static frame-local analysis into the
inlined callback remains unchanged.

That later optimization opportunity is independent of this publication.

```text
CALLBACK_SPECIFIC_FRAME_NATIVE_LOWERING_CHANGED=NO
CLOSURE_MATERIALIZATION_ELIMINATED=NO
```

## Structural regression evidence

New test:

```text
ProtosPerf025InlineCallbackPreparationTest
```

It freezes the preparation ordering for three representative lowering families:

```text
Boolean:
PrepareInlineStructuredBooleanCallbackCall
  -> AdmitsInlineLiteralCallback
  -> LoadInlineCallbackActivation
  -> LoadInlineLiteralFallbackCall

whileTrue:
PrepareInlineStructuredWhileConditionCall
  -> AdmitsInlineLiteralWhileCondition
  -> LoadInlineCallbackActivation
  -> LoadInlineLiteralFallbackCall

Array.each:
PrepareInlineStructuredLocalEachChildCall
  -> AdmitsInlineLiteralLocalEachChild
  -> LoadInlineCallbackActivation
  -> LoadInlineLiteralFallbackCall
```

The test also rejects the old rich preparation operations from the admitted
inline source-root shape.

Existing PERF026 B1/B2/B3, C1 and D1/D2/D3 tests remain the observable semantic,
tooling, control-transfer and collection-regression authority.

## Exact product delta

Exact comparison:

```text
BASE=93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9
HEAD=56f615628efb9c78fee06cbdba6ff21a952539f9
COMMITS=1
FILES_CHANGED=6
ADDITIONS=687
DELETIONS=101
```

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025InlineCallbackPreparationTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.159-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Validation provenance

During implementation, the first focused run exposed a Bytecode DSL builder
shape error in the newly factored physical fallback:

```text
Operation TryFinally expected exactly 1 child, but 3 provided
```

The repair wrapped the ordinary fallback invocation in the required Bytecode
`Block`; no semantic behavior was changed by that repair.

The existing affected set then passed:

```text
TESTS_RUN=90
FAILURES=0
ERRORS=0
SKIPPED=0
```

The new structural regression initially exposed a test-helper ordering bug
because it selected the first fallback operation in the whole generated root
rather than the fallback following the family-specific admission. The helper
was corrected to search each operation after the preceding operation. A
subsequent Java compilation error from capturing the mutable loop index in an
assertion lambda was corrected within the test only.

The new test then passed:

```text
TESTS_RUN=3
FAILURES=0
ERRORS=0
SKIPPED=0
```

Before final publication, concurrent product work advanced `main` through
`51bf02af1fe0fd6247ee17c8a1a764ced77d2732` and then
`93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9`. The PERF025 work was reapplied
without conflict. Because those revisions included executable changes,
including changes to `ProtosBytecodeRootNode` and the Integer/Array runtime
surface, validation was repeated on the exact final substantive base.

Final focal validation on `93cc30cf26bd`:

```text
TESTS_RUN=93
FAILURES=0
ERRORS=0
SKIPPED=0
FOCAL_VALIDATION=PASS
```

The maintainer then reported the repository integrated gate:

```text
make test=PASS
FULL_VALIDATION=PASS
```

After that green integrated gate, only required publication metadata
(`pom.xml` and `CHANGELOG.md`) was materialized. Repository policy does not
require repeating behavioral tests for metadata-only finalization when no
executable/dependency state changes.

Final publication preflight reported:

```text
LOCAL_BASE=93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9
REMOTE_BASE=93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9
GIT_DIFF_CHECK=PASS
VERSION=0.3.159-SNAPSHOT
FINAL_CHANGED_PATHS=6
```

Therefore:

```text
MAINTAINER_REPORTED_FOCAL_VALIDATION=PASS
MAINTAINER_REPORTED_FULL_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command output beyond the maintainer-reported interaction is invented.

## Performance-claim boundary

The maintainer explicitly requested this implementation slice without
measurement. No benchmark, timing comparison, JFR campaign, allocation profile,
or exact-revision performance attribution was performed.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
PERFORMANCE_EFFECT_MEASURED=NO
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural only: PLAT044 inline callback admission no
longer requires first constructing the rich physical Closure invocation that the
successful inline path does not use.

## PERF025 / PLAT044 status

This bounded PERF025 implementation slice is complete. PERF025 itself remains
open for the continuing post-F1 pay-as-you-grow implementation line.

PLAT044 remains ratified/completed; this slice consumes Candidate B′ and does
not reopen or reinterpret the decision.

```text
PERF025_PLAT044_LEAN_INLINE_CALLBACK_PREPARATION=COMPLETE
PERF025_STATUS=OPEN
PLAT044_STATUS=RATIFIED_COMPLETED
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@56f615628efb9c78fee06cbdba6ff21a952539f9`;
- the exact comparison against
  `93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9`;
- the published `0.3.159-SNAPSHOT` changelog/version delta;
- PERF025 / `guillermomolina/protos#758`;
- PLAT044 / `guillermomolina/protos#766`;
- the existing PERF026 B1/B2/B3, C1 and D1/D2/D3 regression ownership; and
- the maintainer-reported focal and integrated validation outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
