# PERF010-A — prepared Context-owned target implementation published

Status: bounded causal implementation experiment published in Protos; semantic focal verification reported green; compiler-lifecycle effect not yet measured.

## Publication identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
PROTOS_VERSION=0.3.78-SNAPSHOT
COMMIT_MESSAGE=PERF010-A: add prepared Context-owned target specialization for PrepareSendArguments

PRIOR_PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_MAIN_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b

PRIOR_CAUSAL_RECORD_REVISION=d9f33bb3db5ebf20761d3d53b6cfe47556a36799
PRIOR_CAUSAL_RECORD=docs/project/evidence/PERF010-A/PERF010-A_CALLER_HELPER_BAILOUT_ROOT_CAUSE.md
```

This record captures the publication state only. It does not claim that the exact hot helper now compiles successfully and does not interpret timing.

## Implemented intervention

The published change adds a new `PrepareSendArguments.fastOrdinarySend` specialization.

The specialization:

- re-runs authoritative ordinary D013 lookup on every invocation;
- derives the currently selected ordinary non-native source-backed Closure and `methodHome`;
- admits a fast hit only when selector, selected Closure identity, `methodHome` identity and entered `ProtosLanguageContext` identity match the cached specialization state;
- materializes the effective Context-owned Bytecode activation `RootCallTarget` once when specialization cache state is populated;
- constructs a fresh invocation activation for every call;
- preserves Task/dynamic-control propagation;
- returns `PreparedClosureCall(cachedTarget, activation)` on the guarded hit;
- retains the exact existing generic `perform` path as the `@Specialization(replaces = "fastOrdinarySend")` fallback.

The generic lookup behavior is shared through the extracted `performOrdinarySendLookup` helper rather than reimplemented independently.

## Structural causal contract

The published fast hit is intentionally shaped so that, after successful cache population and guards, it does not re-enter either source root previously established for the permanent bailout:

```text
BASELINE pathological prefix:
  selectedRuntimeForBytecodeIntrinsic
  -> prepareBytecodeImport
  -> standard-library module resolution
  -> Locale-bearing host expansion

PREVIOUS GUARDED pathological prefix:
  taskOwnedBytecodePlan
  -> bytecodeExecutionPlanForEnteredClosure
  -> bytecodeExecutionPlanForDefinition
  -> sharedBytecodeExecutionPlans.computeIfAbsent(...)
```

Cache population may use `taskOwnedBytecodePlan` once in `fastOrdinarySendTarget` to obtain the correct Context-owned target. The causal claim is that this host machinery is no longer executed from the normal repeated fast-hit body.

Therefore the published source satisfies the intended structural intervention:

```text
AUTHORITATIVE_D013_LOOKUP_PRESERVED=YES
FRESH_ACTIVATION_PRESERVED=YES
METHOD_HOME_PRESERVED=YES
CONTEXT_IDENTITY_GUARDED=YES
CACHED_CONTEXT_OWNED_BYTECODE_TARGET=YES

FAST_HIT_STANDARD_IMPORT_RESOLUTION=ABSENT
FAST_HIT_CONTEXT_PLAN_COMPUTE_IF_ABSENT=ABSENT
FAST_HIT_TASK_OWNED_BYTECODE_PLAN=ABSENT

GENERIC_FALLBACK_PRESERVED=YES
ROOT_TOPOLOGY_CHANGED=NO
IMPORT_SEMANTICS_CHANGED=NO
```

These are structural/source properties of the published revision. The compiler-lifecycle consequence remains to be measured.

## Regression coverage added

The publication adds:

```text
src/test/java/com/guillermomolina/protos/execution/
ProtosPerf010APreparedTargetSpecializationTest.java
```

The focused verification reported by the human executor completed:

```text
Tests run: 18
Failures: 0
Errors: 0
Skipped: 0
FOCAL_VERIFICATION=PASS
```

Relevant explicit markers included:

```text
PERF010A_FRESH_ACTIVATION_PER_HIT=PASS
PERF010A_METHOD_HOME_EXACT_PER_HIT=PASS
PERF010A_REMOVED_OVERRIDE_OBSERVES_DELEGATION_PARENT=PASS
PERF010A_NATIVE_SEND_UNAFFECTED=PASS
PERF010A_MONOMORPHIC_SEND_RESULT=PASS
PERF010A_REPLACED_SELECTION_MISSES_STALE_HIT=PASS

PERF006_B2D1_SOURCE_SEND_COMPOSITION=PASS
PERF006_B2D1_SEND_SUSPENSION_NO_REPLAY=PASS
PERF006_B2D1_IMMEDIATE_METHOD_RECEIVER_HOME=PASS
PERF006_B2D1_DEFAULT_SEND_SPREAD_LOWERING=PASS
PERF006_B2D1_NATIVE_SEND=PASS
PERF006_B2D1_NATIVE_SEND_ARGUMENT_IDENTITY=PASS

PERF006_B2D4A_CALL_TARGET_FROM_SEND=PASS
PERF006_B2D4A_TARGET_BEFORE_ARGUMENTS=PASS
PERF006_B2D4A_TARGET_ARGUMENTS_EXACTLY_ONCE=PASS
PERF006_B2D4A_CALL_TARGET_FROM_CALL=PASS
PERF006_B2D4A_DEFAULT_COMPOSED_CALL_TARGET=PASS
PERF006_B2D4A_DEFAULT_SEND_SPREAD_LOWERING=PASS
PERF006_B2D4A_TARGET_NO_REPLAY_ACROSS_ARGUMENT_SUSPENSION=PASS
PERF006_B2D4A_TARGET_SUSPENSION_COMPOSED=PASS
PERF006_B2D4A_TARGET_ACTIVATION_IDENTITY_NO_REPLAY=PASS
```

This record does not invent an integrated/full-suite result beyond the verification output explicitly supplied.

## Version and changelog

The published revision advances:

```text
0.3.77-SNAPSHOT
->
0.3.78-SNAPSHOT
```

The new CHANGELOG entry correctly describes the change as a bounded implementation experiment. It explicitly does not claim dominant-cause attribution, attributable fraction, or production optimization selection.

## Remaining causal gate

The publication itself is not the result PERF010-A is trying to establish.

The exact hot caller remains:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

The next evidence step must run the already-existing source-identity/compiler-lifecycle diagnostic against exactly:

```text
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
```

and classify:

```text
CALLER_HELPER_PERMANENT_BAILOUT=
  REMOVED |
  MATERIALLY_CHANGED |
  UNCHANGED
```

The existing baseline result to compare against is:

```text
BASELINE_CALLER_COMPILER_TOPOLOGY=C
BASELINE_HELPER_RESULT=PERMANENT_BAILOUT
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_BASELINE=YES
```

Timing must not be interpreted as causal evidence until the lifecycle gate demonstrates that this intervention actually changes the compiler-stranding state.

## Current work state

```text
PERF010A_PREPARED_TARGET_INTERVENTION=PUBLISHED
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
PROTOS_VERSION=0.3.78-SNAPSHOT

FOCAL_VERIFICATION=PASS
STRUCTURAL_CAUSAL_CONTRACT=SATISFIED

CALLER_HELPER_LIFECYCLE_AFTER_INTERVENTION=NOT_MEASURED
CALLER_HELPER_PERMANENT_BAILOUT=NOT_MEASURED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Next slice

The next slice is investigation/evidence only.

Use the existing `guillermomolina/protos-benchmarks` compiler-lifecycle/source-identity harness at revision `e0bbf213c8140491f91913712cbf6d9cd0270e0b` to build/run the newly published Protos revision and determine whether the exact caller helper's permanent bailout is removed, materially changed, or unchanged.

Do not modify Protos.

Do not time the implementation before this lifecycle discriminator is known.
