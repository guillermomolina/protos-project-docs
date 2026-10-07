# TEST006-A — Native guest Tier-2 release-admission classification

Status: **CLOSED / PASS**

This durable, non-normative record retains the TEST006-A release-admission
classification for `guillermomolina/protos#755`.

It does not change Protos language or Standard Library semantics, does not
change the Native gate, and does not authorize a release.

## Identity

```text
WORK_ITEM=TEST006-A
TYPE=INVESTIGATION
GITHUB_ISSUE=guillermomolina/protos#755

PROTOS_REFERENCE_REVISION=7770aa135bb65c7219de5dbe9e2b3ac22cdaf31a
PROTOS_REFERENCE_COMMIT=BUG013: harden Native guest Tier-2 admission

UPSTREAM_ISSUE=oracle/graal#14579
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
GRAALVM_UPSTREAM_TAG=vm-25.4.4.1.1
GRAALVM_UPSTREAM_TAG_REVISION=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
```

The investigation used current GitHub/repository authority only. No command,
build, test, runtime, product-repository mutation, release mutation or gate
change was performed.

## Governing Native contract

PLAT038 requires the Native artifact to preserve real optimizing Truffle guest
runtime behavior:

```text
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
GUEST_JIT_BEHAVIOR=REQUIRED_AND_VERIFIED
INTERPRETER_ONLY_NATIVE_ACCEPTABLE=NO
```

The selected DIST005-C artifact model is:

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
```

Neither authority defines a stronger invariant that every guest compilation
attempt must succeed after successful Tier-2 capability has already been
demonstrated.

## Historical release evidence

The public prerelease `v0.3.116` was published from exact candidate:

```text
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
NATIVE_COMPLETE_ADMISSION=PASS
```

Its public Native artifact was admitted as the recommended first-run artifact
and the release claims a validated optimizing Truffle guest/runtime-compilation
path.

At that exact candidate, `build/native/test-native.sh` already failed closed
on:

```text
OPT_FAILED!=0
FRAME_WITHOUT_BOXING_FAILURES!=0
COMPILATION_FAILURES!=0
HELPER_BYTECODE_ROOT_TIER2<1
SEMANTIC_BYTECODE_ROOT_TIER2<1
```

but its forced-JIT workload was the weak immediate-compilation probe:

```text
-e '1'
-Dpolyglot.engine.CompileImmediately=true
```

The later BUG013 hardening revision
`7770aa135bb65c7219de5dbe9e2b3ac22cdaf31a` did not invent those checks. It
replaced the weak probe with a hot captured-local closure workload and
deterministic threshold-triggered compilation.

Therefore the hardened gate exposed a condition that the historical admission
workload did not exercise; the hardening itself does not establish that the
condition first appeared after the published Native release.

## Residual upstream failure

BUG013 separated the Protos-owned raw-`VirtualFrame` lifetime defect from the
remaining Native-only failure.

The residual is independently reproduced outside Protos and filed as:

```text
oracle/graal#14579
Bytecode DSL root fails Tier-2 runtime compilation in Native Image with
FrameWithoutBoxing materialization
```

The independent reproducer removes Protos semantics, custom operations, locals,
loops and yield from the causal requirement. The same Bytecode DSL root passes
Tier-2 on the JVM and fails Native runtime compilation.

No observable Protos semantic corruption, Native Image build failure, process
crash, packaging failure or debugger failure has been established from this
residual upstream condition.

## Graal/Truffle failure behavior

In the exact `vm-25.4.4.1.1` upstream source:

- `CompilationFailureAction=Print` prints a failed Truffle compilation but
  does not throw to the guest caller or exit the VM;
- `OptimizedCallTarget.handleCompilationFailure` marks a permanent/non-bailout
  compilation failure as `compilationFailed=true`;
- subsequent hotness checks do not resubmit that failed call target; and
- guest execution can therefore continue without optimized code for that
  target.

The observed `FrameWithoutBoxing should not be materialized` path originates
from Graal's `EnsureVirtualizedNode.ensureVirtualFailure` as a compiler graph
error. It is not evidence that the guest result itself is incorrect.

This fallback behavior is relevant evidence, but fallback alone is not
sufficient release admission because PLAT038 still requires an optimizing
Native artifact.

## Classification

```text
TEST006_A_STATUS=PASS

NATIVE_IMAGE_BUILD_FAILURE_ESTABLISHED=NO
NATIVE_EXECUTION_FAILURE_ESTABLISHED=NO
OBSERVABLE_PRODUCT_FAILURE_ESTABLISHED=NO

HISTORICAL_NATIVE_ARTIFACTS_OPERATIONAL=YES
HARDENED_GATE_DISCOVERED_PREEXISTING_CONDITION=YES

OPT_FAILED_ZERO_ROLE=DIAGNOSTIC_ADMISSION_GATE

SUCCESSFUL_TIER2_REQUIRED=YES

NATIVE_HOT_TIER2_BAILOUT_RELEASE_BLOCKING=NO
CORRECT_GUEST_FALLBACK_SUFFICIENT=NO

CURRENT_NATIVE_GATE_POLICY=NARROW_WITH_EXPLICIT_INVARIANTS
```

The two apparently conflicting conclusions are intentional:

1. the exact known upstream hot recompilation failure is not, by itself,
   release-blocking when all explicit exception invariants are satisfied; and
2. merely obtaining a correct interpreted result is not enough to admit Native,
   because successful guest Tier-2 capability remains mandatory.

## Required fail-closed invariants

Any TEST006 implementation that narrows the gate must retain all of the
following:

```text
NATIVE_IMAGE_BUILD_MUST_PASS=YES
NATIVE_REQUIRED_EXECUTION_AND_CONFORMANCE_MUST_PASS=YES
GUEST_SEMANTIC_RESULT_MUST_REMAIN_CORRECT=YES

HELPER_BYTECODE_ROOT_TIER2>=1
SEMANTIC_BYTECODE_ROOT_TIER2>=1

UNEXPECTED_OPT_FAILED=0
UNEXPECTED_COMPILER_OR_INTERNAL_ERRORS=0
UNKNOWN_FRAME_MATERIALIZATION_FAILURES=0

KNOWN_EXCEPTION_SCOPE=
  oracle/graal#14579;
  GraalVM/Graal/Truffle 25.4.4.1.1;
  Native runtime compilation of generated Bytecode DSL roots;
  exact FrameWithoutBoxing "should not be materialized" failure class;
  correct guest execution;
  independent successful Tier-2 evidence for both required generated root families

GRAAL_TRUFFLE_PLANE_CHANGE_INVALIDATES_EXCEPTION_UNTIL_RECLASSIFIED=YES
GENERIC_OPT_FAILED_SUPPRESSION_ALLOWED=NO
GENERIC_FRAMEWITHOUTBOXING_SUPPRESSION_ALLOWED=NO
```

The implementation must account for every allowed failure event. A substring
ignore that can hide unrelated compiler failures is not acceptable.

## BUG013 and DIST009 consequence

BUG013 remains the technical defect owner and `oracle/graal#14579` remains an
open upstream defect.

Its release-blocking role changes only after the TEST006 gate policy is
implemented and validated:

```text
BUG013_RELEASE_BLOCKING_ROLE=
  BLOCKING_UNTIL_TEST006_POLICY_IS_ENCODED;
  AFTER_ENCODING_THE_KNOWN_UPSTREAM_DEFECT_REMAINS_OPEN_BUT_IS_NOT_BY_ITSELF_RELEASE_BLOCKING

DIST009_CAN_RESUME=NO
IMPLEMENTATION_REQUIRED=YES
ARCHITECTURE_DECISION_REQUIRED=NO
UPSTREAM_COORDINATION_REQUIRED=NO
```

DIST009 must remain blocked under the currently published gate. It can resume
only after TEST006-B publishes and validates the narrowed fail-closed admission
policy.

## Next slice

```text
NEXT_SLICE=TEST006-B
NEXT_SLICE_NAME=Encode known-upstream Native Tier-2 admission exception
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

TEST006-B remains a bounded slice of TEST006/#755. It does not require a new
formal Issue because it has no independent scheduling, blockage, decision or
publication identity outside the existing TEST006 work item.

## Material evidence inspected

- `guillermomolina/protos:AGENTS.md`
- `guillermomolina/protos:AGENTS.work/TEST.md`
- `guillermomolina/protos:AGENTS.work/COORDINATION.md`
- `guillermomolina/protos:AGENTS.work/RELEASE.md`
- `guillermomolina/protos:build/native/test-native.sh@7770aa135bb65c7219de5dbe9e2b3ac22cdaf31a`
- `guillermomolina/protos:build/native/test-native.sh@0336ae20216bf2eec17854bea0f6435e4e1e9b19`
- `guillermomolina/protos#755` — TEST006
- `guillermomolina/protos#749` — BUG013
- `guillermomolina/protos#743` — DIST009
- `guillermomolina/protos#549` — DIST005
- `docs/project/evidence/TEST006/TEST006_NATIVE_TIER2_RELEASE_ADMISSION_ROUTING.md@97d67c9dc3dc20e52663d9b7e59fcad91d64c933`
- DIST005-C, DIST005-D8 and DIST005-D9 durable evidence
- I069 Native guest runtime-compilation implementation evidence
- BUG013-A/B/C durable evidence
- UPSTREAM004 preparation/publication evidence
- DIST009 prepublication VS Code acceptance evidence
- PLAT038 Native Image runtime/release boundary
- PLAT039 Truffle runtime-compilation boundary
- public Protos prerelease `v0.3.116`
- `oracle/graal#14579`
- `oracle/graal@vm-25.4.4.1.1`:
  `OptimizedCallTarget.java`, `TruffleCompilerImpl.java`,
  `EnsureVirtualizedNode.java`, and `truffle/docs/Options.md`
