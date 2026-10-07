# I072 Phase B — compact ordinary-call ABI implementation evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase B — compact ordinary-call ABI
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=db7171058ecf1cd3867c28dc24f954df704004e3
PROTOS_REVISION=795c1ea75236e028a42bbfb91f8c26f8d12c73e4
PROTOS_VERSION=0.3.91-SNAPSHOT
COMMIT_MESSAGE=I072-B: compact ordinary call ABI
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PLAT040_COMPARATIVE_EVIDENCE=docs/project/evidence/PLAT040/PLAT040_CROSS_TRUFFLE_HOT_CALL_DECISION_EVIDENCE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_A_EVIDENCE=docs/project/evidence/I072/I072_PHASE_A_GUARDED_SELECTED_SEND_IMPLEMENTATION.md@4639bab0e9175781ca0c952eef30e909ee55644f
PHASE_B_STATE=COMPLETE
I072_STATE=OPEN

## Result

I072 Phase B publishes the compact ordinary source-call boundary required by
PLAT040 Candidate F-prime while preserving the Phase A guarded selected-send
substrate.

For admitted source-backed ordinary sends, the valid path no longer constructs a
rich callee `ProtosActivation` before entering the Context-owned Truffle call
target. The call crosses that boundary using a fixed frame-argument layout and
materializes the existing rich activation representation only inside the
semantic target adapter where current Phase B semantics still require it.

The published Phase B outcome is:

```text
PHASE_A_GUARDED_SELECTION_PRESERVED=YES

GENERAL_D013_LOOKUP_ON_VALID_HIT=NO
GENERIC_SELECTED_VALUE_CLASSIFICATION_ON_VALID_HIT=NO

ORDINARY_SOURCE_TARGET_ENTRY_REQUIRES_RICH_PROTOS_ACTIVATION=NO
COMPACT_FRAME_ARGUMENT_ABI=YES
PER_CALL_STATE_MINIMIZED=YES
CAPTURED_LEXICAL_STATE_REUSED=YES

RECEIVER_THIS_EXACT=YES
METHOD_HOME_EXACT=YES
ARGUMENT_VECTOR_EXACT=YES

CURRENT_CONTEXT_MATERIALIZATION_POLICY_PRESERVED=YES
CURRENT_GUEST_ARGUMENT_MATERIALIZATION_POLICY_PRESERVED=YES
OPTIONAL_CONTROL_STATE_SEMANTICS_PRESERVED=YES

GENERIC_FALLBACK_EXACT=YES
NATIVE_FALLBACK_EXACT=YES

PLAT036_I068_COMPATIBILITY=PASS
D179_COMPATIBILITY=PASS
PLAT039_COMPATIBILITY=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

FOCAL_VALIDATION=PASS
STRUCTURAL_HOT_PATH_DIAGNOSTIC=PASS
BROADER_VALIDATION=PASS
PUBLICATION=PASS
```

## Published implementation

The Phase B product revision changes the execution boundary in:

```text
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
pom.xml
CHANGELOG.md
```

### Compact frame-argument boundary

`ProtosFrameArguments.compactImmediateMethodCall` establishes a fixed ordinary
source-call header containing:

```text
selected Closure
receiver / this
exact methodHome
caller provenance
return-home token
supplied positional values
```

Captured lexical state is not copied into a new per-call aggregate; it remains
owned by and reachable from the selected Closure.

`PrepareSendArguments.guardedOrdinarySend` retains the exact Phase A selected
Closure, methodHome, target and lookup-stability dependency, but now creates the
compact frame arguments instead of calling
`ProtosActivation.forImmediateMethodInvocation` before target entry.

The older `fastOrdinarySend` source-backed specialization was aligned with the
same compact target boundary. The authoritative generic replacement
specialization remains unchanged for non-admitted cases.

### Target-entry materialization seam

`EnterClosureCall.direct` and its indirect fallback pass the prepared target
argument vector directly to the selected RootCallTarget.

The Context-owned `ProtosSemanticBytecodeRootNode` now binds its own
`VirtualFrame`. Its semantic-helper operation obtains the activation through
`ProtosFrameArguments.activation(frame)`.

For a compact ordinary call this is the first rich callee-activation
materialization point. It creates the same fresh execution context and frozen
guest supplied-argument Array required by the existing semantics, preserves the
exact return-home relationship, receiver, methodHome, captured lexical state,
module/domain provenance, and inherits Task or direct dynamic-control ownership.

The materialized activation is published back into frame argument zero so the
semantic helper and debugger/NodeLibrary projection observe the same fresh guest
execution-context identity.

This is deliberately a Phase B compatibility seam. Conditional execution-context
materialization and removal of eager guest supplied-Array materialization remain
Phase C work.

## Semantic regression evidence

The human executor reported focused PASS coverage for the Phase B-sensitive
ordinary call surface, including:

```text
PERF010A_FRESH_ACTIVATION_PER_HIT=PASS
PERF010A_METHOD_HOME_EXACT_PER_HIT=PASS
PERF010A_REMOVED_OVERRIDE_OBSERVES_DELEGATION_PARENT=PASS
PERF010A_NATIVE_SEND_UNAFFECTED=PASS
PERF010A_FRESH_MATERIALIZATION_IDENTITY_CHURN_STABLE=PASS
PERF010A_MONOMORPHIC_SEND_RESULT=PASS
PERF010A_REPLACED_SELECTION_MISSES_STALE_HIT=PASS

PERF006_B2A_RECEIVER_METHOD_HOME_PRESERVED=PASS
PERF006_B2A_CAPTURED_RETURN_HOME_PRESERVED=PASS
PERF006_B2C3A_REST_BINDING=PASS
PERF006_B2C3B2_SUPPLIED_ARGUMENT_SUPPRESSES_DEFAULT=PASS
PERF006_B2D1_SOURCE_SEND_COMPOSITION=PASS
PERF006_B2D1_SEND_SUSPENSION_NO_REPLAY=PASS
PERF006_B2D5C_DEFAULT_SPREAD_SNAPSHOT_AT_ARGUMENT_POSITION=PASS
```

The second focused gate reported PASS across Task/control/suspension and
multi-Context projection, including:

```text
PERF006_B3B_TASK_BACKED_SOURCE_COMPOSITION=PASS
PERF006_B3B_SOURCE_CALL_TASK_IDENTITY=PASS
PERF006_B3C_NESTED_NATIVE_SUSPENSION_CPRIME_CHAIN=PASS
PERF006_B3D_TOP_LEVEL_TASK_PUBLICATION=PASS
PERF006_B4C_NLR_CROSSES_CLEANUP=PASS
PERF006_B4E_CANCELLATION_INJECTED_INTO_CPRIME=PASS
PERF006_B4G_BYTECODE_REPLAY_CURSOR=NO
PERF006_B6A6A1R_FOREIGN_CONTEXT_BYTECODE_PROJECTION=PASS
PERF006_B6A6A1_CONTEXT_LOCAL_BYTECODE_DIRECT=PASS
PERF006_C3D_P_BYTECODE_REMATERIALIZATION=PASS
```

The complete Maven gate was then reported green:

```text
mvn -q verify
RESULT=PASS
```

Representative integrated markers include debugger scope authority, lexical
capture by reference, exact NLR home/payload, Task ownership, suspension without
replay, native fallback, current/foreign Context bytecode projection and the
full PERF010-A guarded-send regression set.

Current D179 execution-context conformance remains in the integrated suite,
including remove/recreate and capture-by-reference execution-context cases; the
complete `verify` gate passed after the Phase B executable changes.

## Structural hot-path discriminator

The human executor inspected the generated annotation-processor output after
compilation.

For `PrepareSendArguments`, the generated DSL still contains the Phase A
`guardedOrdinarySend` specialization and its cached selector-specific Truffle
`Assumption`. On a valid generated hit the path checks the cached assumption,
receiver identity, selector and entered Context, then enters
`guardedOrdinarySend` without executing the general lookup/classification path.

The generated `EnterClosureCall` retains target-cached
`DirectCallNode` specialization plus the indirect fallback. The source
specialization now invokes the target using `PreparedClosureCall.targetArguments`
rather than a preconstructed callee activation.

The generated semantic target adapter binds the target frame and delegates rich
activation acquisition to `ProtosFrameArguments.activation(frame)`, placing the
compatibility materialization after target entry.

Therefore the Phase B structural discriminator is satisfied:

```text
SPECIALIZATION_ESTABLISHMENT=SEPARATE
VALID_GUARDED_HIT_GENERAL_LOOKUP=NO
VALID_GUARDED_HIT_GENERIC_SELECTED_CLASSIFICATION=NO
PRE_TARGET_RICH_CALLEE_ACTIVATION=NO
COMPACT_FRAME_ARGUMENT_TARGET_ENTRY=YES
TARGET_ENTRY_COMPATIBILITY_MATERIALIZATION=YES
GENERIC_FALLBACK=EXACT
```

Timing attribution is intentionally not claimed by this phase; PERF010-A remains
the owner of attributable performance conclusions.

## Publication

Before finalization the human executor re-fetched current `origin/main` and
reported:

```text
HEAD=db7171058ecf1cd3867c28dc24f954df704004e3
origin/main=db7171058ecf1cd3867c28dc24f954df704004e3
git diff --check=PASS
```

Late finalization selected implementation version `0.3.91-SNAPSHOT` and added
the matching root changelog entry.

The resulting product commit is:

```text
795c1ea75236e028a42bbfb91f8c26f8d12c73e4
I072-B: compact ordinary call ABI
```

The same publication commit also carries the independently requested Surefire JVM
option `--sun-misc-unsafe-memory-access=allow` alongside the existing
`--enable-native-access=ALL-UNNAMED`. That Maven warning-suppression setting is
not part of the Phase B call architecture and does not change Protos guest
semantics.

## Phase boundary

Phase B intentionally preserves:

```text
eager fresh guest execution-context creation after target entry
eager frozen guest supplied-argument Array creation after target entry
existing PreparedClosureCall control/suspension carrier machinery
generic/native fallback representation
```

Those preserved costs are not evidence against Phase B; they define the next
bounded I072 work.

## Next slice

```text
NEXT_I072_PHASE=Phase C — execution-context and argument materialization
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

Phase C must start from current `origin/main`, preserve the Phase A/B guarded
selection and compact target-entry ABI, and move ordinary optimized execution to
frame-backed/invocation state first. It must materialize the guest-visible
execution-context representation only when semantically observed or escaped and
must stop creating a guest supplied-argument Array solely for ordinary internal
transport while preserving rest/default/escape/identity, I068 lexical authority,
D179 remove/recreate behavior, reflection/debugger projection, Task/control,
suspension/no-replay and multiple-Context semantics.
