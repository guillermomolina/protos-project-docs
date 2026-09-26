# I072 Phase D — optional control/state and prepared-call separation evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase D — optional control/state and prepared-call separation
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=045bfdb8ec3ccaf09be8b35a2053c124a9edebb8
PHASE_D_SLICE_1_REVISION=40b90afa1fc6185bd4484f580ae921184187b652
PROTOS_REVISION=c907978a507a9dde8c7d8dad56bcc54513326316
PROTOS_VERSION=0.3.94-SNAPSHOT
FINAL_COMMIT_MESSAGE=072-D2: complete prepared-call/control-mode separation
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_C_EVIDENCE=docs/project/evidence/I072/I072_PHASE_C_EXECUTION_CONTEXT_ARGUMENT_MATERIALIZATION_IMPLEMENTATION.md@9e9f3c8f076ce6dcded6d60bf69ceb8938f28cf2
PHASE_D_STATE=COMPLETE
I072_STATE=OPEN

## Result

I072 Phase D implements the PLAT040 optional-state requirement:

```text
Optional semantic capability must not force unrelated ordinary calls
to carry its full physical machinery.
```

Phase D was published in two implementation commits.

The first slice:

```text
40b90afa1fc6185bd4484f580ae921184187b652
I072-D1: consolidate structured-call capability vector
0.3.93-SNAPSHOT
```

removed the sixteen-field structured-control capability vector from every
`PreparedClosureCall` instance. Those mutually exclusive capabilities moved
behind one nullable `StructuredCallCapabilities` holder. Ordinary source calls,
plain native calls without a structured capability, and module-initialization
calls do not allocate that holder.

The completing slice:

```text
c907978a507a9dde8c7d8dad56bcc54513326316
072-D2: complete prepared-call/control-mode separation
0.3.94-SNAPSHOT
```

removes the remaining universal physical prepared-call carrier. The shared
`PreparedClosureCall` name is now a small dispatch interface rather than a
class containing state for all invocation modes.

The physical representations are mutually exclusive:

```text
OrdinarySourceCall
NativeCall
ModuleInitializationCall
```

An ordinary source call therefore no longer contains native-body,
structured-control-capability, or module-initialization state.

## D1 — structured capability separation

Before D1, the universal prepared-call representation directly carried
structured-control state for ensure, Error handling, while, Boolean callbacks,
collection callbacks, standard Object.call/import handling and Map operations.

D1 replaced those per-instance fields with:

```text
StructuredCallCapabilities structured
```

where the holder is created only when at least one applicable structured
capability is present.

The ordinary compact and rich source-call constructors set no structured holder.

D1 deliberately did not claim complete Phase D closure: its changelog recorded
that type-level separation of the ordinary path from the shared carrier remained
open.

## D2 — physical prepared-call separation

D2 replaces the universal concrete `PreparedClosureCall` with an interface.

### OrdinarySourceCall

The ordinary source representation carries only:

```text
RootCallTarget bodyTarget
ProtosActivation activation
Object[] targetArguments
return-home / ownership state inherited from ReturnHomeOwningCall
```

For the Phase B compact ABI shape, `activation` is null before target entry and
`targetArguments` is the compact frame-argument vector.

It contains no:

```text
ProtosNativeClosureBody
StructuredCallCapabilities
PreparedModuleInitialization
import/module mode state
```

### NativeCall

Native invocation owns the native-specific representation:

```text
native body
supplied values
activation
optional StructuredCallCapabilities
return-home / ownership state
```

The structured capability predicates and preparation operations live on this
shape rather than on the ordinary physical representation.

### ModuleInitializationCall

Module initialization owns only its module lifecycle state and the source-target
projection derived from that lifecycle handle. It does not acquire ordinary
return-home ownership merely to fit a universal carrier.

### Shared return-home semantics

`ReturnHomeOwningCall` factors the return-home completion and non-local-return
handling shared by the two invocation shapes that actually use those semantics:
ordinary source and native calls.

This is sharing of behavior, not restoration of universal optional state.
`ModuleInitializationCall` does not extend that base.

## Construction-site migration

The Phase D final revision migrates ordinary construction sites to explicit
factories including:

```text
PreparedClosureCall.ordinary(...)
PreparedClosureCall.ordinaryCompact(...)
PreparedClosureCall.nativeCall(...)
PreparedClosureCall.moduleInitialization(...)
```

The guarded Phase A ordinary-send specialization and the retained fast ordinary
send create the lean compact ordinary shape.

Task-owned source calls and composed source calls create the ordinary source
shape while retaining their existing Task/control provenance in the activation
where semantically required.

Native and structured paths create the native shape.

Module initialization creates the module-initialization shape.

## Structural discriminator

D2 adds:

```text
src/test/java/com/guillermomolina/protos/execution/
ProtosI072PhaseDPreparedCallSeparationTest.java
```

The test directly checks that:

```text
PreparedClosureCall is an interface
OrdinarySourceCall and special leaves are distinct representations
OrdinarySourceCall declares no native-named state
OrdinarySourceCall declares no structured-named state
OrdinarySourceCall declares no module-initialization-named state
ordinary prepared source calls instantiate OrdinarySourceCall
ordinary calls report no native/immediate/structured capability
ordinary calls reject structured/native-only entry points
ordinary no-control preparation does not eagerly allocate ProtosDynamicControlState
```

This is a representation-level structural discriminator. Phase D does not rely
on a hypothesis that Graal partial escape analysis will erase a universal
carrier after the fact; the ordinary Java representation itself no longer owns
the unrelated special-mode fields.

## Preserved earlier I072 architecture

The final Phase D changelog explicitly preserves the previous phases:

```text
Phase A
  guarded selector-specific stable selection

Phase B
  compact frame-argument target entry

Phase C
  conditional guest Context/Array materialization
```

D2 changes the prepared-call representation after selection/preparation; it does
not reintroduce general lookup, a pre-target rich activation for compact ordinary
calls, eager guest Context creation, eager frame materialization, or eager guest
supplied-Array transport.

## Phase boundary

Phase D does not perform Phase E structured-control convergence.

The current native/structured capability machinery remains semantically intact
behind `NativeCall` and `StructuredCallCapabilities`.

Phase E remains responsible for converging stable canonical Boolean/control and
collection callback paths onto the guarded/direct structure without turning
standard selectors into privileged language semantics, followed by final I072
cross-slice validation and closure.

## Publication

The complete Phase D product history relative to Phase C is exactly two commits:

```text
BASE=045bfdb8ec3ccaf09be8b35a2053c124a9edebb8
D1=40b90afa1fc6185bd4484f580ae921184187b652
D2=c907978a507a9dde8c7d8dad56bcc54513326316
COMMITS_AHEAD=2
```

The aggregate product delta is limited to:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseDPreparedCallSeparationTest.java
```

At durable-evidence capture, GitHub Actions had accepted the D2 push and its
single `test` check was still running. This record does not reinterpret that
in-progress remote check as a completed validation result.

## Phase D outcome

```text
I072_PHASE_D=COMPLETE

ORDINARY_SOURCE_CALL_USES_LEAN_CALL_REPRESENTATION=YES
ORDINARY_SOURCE_CALL_REQUIRES_UNIVERSAL_PREPARED_CLOSURE_CALL=NO

ORDINARY_SOURCE_CALL_PHYSICALLY_CARRIES_STRUCTURED_CAPABILITIES=NO
ORDINARY_SOURCE_CALL_PHYSICALLY_CARRIES_NATIVE_MODE_STATE=NO
ORDINARY_SOURCE_CALL_PHYSICALLY_CARRIES_MODULE_INITIALIZATION_STATE=NO
ORDINARY_SOURCE_CALL_PHYSICALLY_CARRIES_IMPORT_MODE_STATE=NO

ORDINARY_SOURCE_CALL_CARRIES_ONLY_REQUIRED_RETURN_CONTROL_STATE=YES
ORDINARY_NO_CONTROL_CALL_EAGER_DYNAMIC_CONTROL_STATE=NO

PHASE_E_STRUCTURED_CONTROL_CONVERGENCE_STARTED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

PUBLICATION=PASS
I072_STATE=OPEN
```

## Next slice

```text
NEXT_I072_PHASE=Phase E — structured-control convergence and closure
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

Phase E is the final implementation phase selected by PLAT040. It must implement
the already-ratified structured-control convergence in one coherent pass, retain
ordinary semantic selection and exact fallback, run the final cross-slice
validation required by #719, and close I072 only when the complete closure
contract is satisfied. It is not a new PLAT/PERF investigation.
