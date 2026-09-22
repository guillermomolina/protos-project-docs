# PERF010-A — Next common-path causal candidate investigation

Status: INVESTIGATION COMPLETE — NEXT CAUSAL CANDIDATE ESTABLISHED

This durable, non-normative record captures the investigation outcome for
`PERF010-A / #691` after paired-control reconciliation of the first three
causal ablation attempts. It does not authorize a production optimization.

## Revision identity

```text
PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
ABLATION_1_EVIDENCE_REVISION=08d39b1d07b6766c5533486bf56b12d0b881fec2
ABLATION_3_EVIDENCE_REVISION=8a82117c50dbb2906ef80fac69e7421f95beb10a
PAIRED_CONTROL_RECONCILIATION_PROJECT_REVISION=a31a8f3780a4810b89bc4179defe8fec18322514
```

The investigated `ProtosBytecodeRootNode.java`, `ProtosClosureValue.java`,
and four PERF010-A workloads have the same relevant content at the evidence pin
and the current Protos revision above.

## Result

```text
PERF010A_NEXT_CAUSAL_CANDIDATE=ESTABLISHED

CANDIDATE_COMPONENT=duplicate ProtosClosureValue.nativeBody() Optional projection in ProtosBytecodeRootNode.finishPreparingComposedCall
COMMON_PATH=YES
EXACT_OPERATION=YES
SEMANTIC_EQUIVALENCE_BY_CONSTRUCTION=YES
FOUR_WORKLOAD_OBSERVABLE_RESULT_PRESERVED=YES
SINGLE_CAUSAL_COMPONENT=YES
STATIC_STRUCTURAL_CONTRACT_POSSIBLE=YES

RELATION_TO_ABLATION_1=OUTSIDE
RELATION_TO_ABLATION_3=OUTSIDE

CAUSAL_COST=NOT_YET_MEASURED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## Common-path evidence

The retained full-stack evidence places the following path in all four workloads:

```text
Bytecode execution
  -> PrepareSendArguments / call preparation
  -> prepareImmediateMethodCall
  -> finishPreparingComposedCallByImplementation
  -> finishPreparingComposedCall
  -> ProtosClosureValue.nativeBody
  -> java.util.Optional.ofNullable
```

Observed sample shares for that exact path were approximately 4.42–5.52% across
the four workloads. This establishes repeated steady-state presence only; it is
not interpreted as attributable cost.

The profiles also contain effective native-branch execution through
`PreparedClosureCall.enterNative -> EnterClosureCall.nativeCall`, so the branch
containing the duplicate projection is actually reached in every workload.

## Selected operation

Current code performs two projections of the same final field on the native
branch:

```java
if (closure.nativeBody().isPresent()) {
    ProtosNativeClosureBody nativeBody =
            closure.nativeBody().orElseThrow();
    ...
}
```

The future diagnostic intervention is exactly:

```java
java.util.Optional<ProtosNativeClosureBody> nativeBodyProjection =
        closure.nativeBody();

if (nativeBodyProjection.isPresent()) {
    ProtosNativeClosureBody nativeBody =
            nativeBodyProjection.orElseThrow();
    ...
}
```

`ProtosClosureValue.nativeBody()` is `Optional.ofNullable(nativeBody)`, and
`nativeBody` is final. Reusing the first projection therefore preserves the
same native/source classification and the same `ProtosNativeClosureBody`
reference while eliminating only the second projection.

The intervention does not change lookup, receiver/delegation, method binding,
closure capture, arguments, activation identity, return home, errors, nonlocal
return, continuations, RootTag/source/debugger identity, interop, or native
execution.

## Exact scope

Included call sites:

```text
ProtosBytecodeRootNode.finishPreparingComposedCall:
  closure.nativeBody().isPresent()
  closure.nativeBody().orElseThrow()
```

Excluded and unchanged:

- all other `nativeBody()` call sites in `ProtosBytecodeRootNode`;
- all `nativeBody()` uses in standard protocols, `ProtosClosureInvoker`,
  `ProtosCoreBootstrap`, CLI, tests, and other runtime code;
- `ProtosClosureValue.nativeBody()` itself;
- `finishPreparingComposedCallByImplementation`;
- send/closure preparation outside the bounded rewrite;
- `ProtosValueLookup`, `ProtosActivation`, native-call execution,
  semantic/helper Bytecode, continuation/control transfer, RootTag/source/
  debugger machinery, interop, and all four workload sources.

Optimizing any other `nativeBody()` call in the same diagnostic would destroy
the intended causal isolation.

## Static structural contract for the future ablation

A future `perf010a-validate` must fail closed unless it proves:

```text
PATCH_TARGET_COUNT=1
PATCH_TARGET=src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java

finishPreparingComposedCall:
  baseline closure.nativeBody() calls = 2
  ablation closure.nativeBody() calls = 1

ablation:
  exactly one local Optional projection from closure.nativeBody()
  isPresent() uses that same local
  orElseThrow() uses that same local
  no second closure.nativeBody() remains in finishPreparingComposedCall

all other nativeBody() call sites:
  byte-for-byte unchanged

ProtosClosureValue.java:
  byte-for-byte unchanged
  nativeBody field remains final
  nativeBody() remains Optional.ofNullable(nativeBody)

rest of finishPreparingComposedCall:
  unchanged apart from the bounded projection rewrite

all other production files:
  unchanged by the diagnostic patch
```

This gives both semantic equivalence and exact causal isolation before smoke or
reference execution.

## Other profile-backed candidates considered

`LoadLocal/ClearLocal` has stronger profile presence, but no single
source-level transformation has yet been established that avoids simultaneously
changing operand transport, temporary lifetime, or resume/continuation paths.

`PrepareSendArguments/List.of(supplied)` is also prominent, but the current
evidence does not establish the lifetime and non-aliasing needed to replace the
copy with a view while preserving semantics by construction.

Activation construction exposes copy/allocation activity, but no single common
copy has yet been isolated without touching fresh activation context, argument
snapshot, return-home, or activation identity.

Those candidates therefore do not currently pass the six-condition admission
gate.

## Next bounded step

The next slice may implement one diagnostic causal ablation for the established
duplicate `nativeBody()` projection. It must use the exact static scope contract
above before smoke/reference and must not modify production
`guillermomolina/protos`.

The future measurement must retain canonical/control comparability. Frequency
or stack share alone must not be used as a performance-magnitude claim.
