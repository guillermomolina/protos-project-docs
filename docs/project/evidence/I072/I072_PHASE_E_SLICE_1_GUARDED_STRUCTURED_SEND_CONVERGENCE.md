# I072 Phase E slice 1 — guarded structured-send convergence evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase E — structured-control convergence and closure
SLICE=1
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=c907978a507a9dde8c7d8dad56bcc54513326316
PROTOS_REVISION=e11ca30fd3345da8813114bcffbb76ba0c0f234d
PROTOS_VERSION=0.3.95-SNAPSHOT
COMMIT_MESSAGE=I072 Phase E (slice 1): converge canonical ensure/handle/while/Boolean sends
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_D_EVIDENCE=docs/project/evidence/I072/I072_PHASE_D_OPTIONAL_CONTROL_PREPARED_CALL_SEPARATION_IMPLEMENTATION.md@518c516cbb9d05c410e294d9710f33b88fc4401c
PHASE_E_SLICE_1_STATE=COMPLETE
PHASE_E_STATE=IN_PROGRESS
I072_STATE=OPEN

## Result

This publication begins the final I072 structured-control convergence phase.

The product revision:

```text
e11ca30fd3345da8813114bcffbb76ba0c0f234d
```

adds a new guarded structured-send specialization to `PrepareSendArguments` for the first canonical control families:

```text
Object.ensure
Error.handle
Object.while
Boolean structured callbacks:
  ifTrue
  ifFalse
  ifTrue:ifFalse:
  and
  or
```

The specialization reuses the exact Phase A guarded lookup substrate rather than adding selector privilege or a parallel invalidation mechanism.

## Guarded establishment

The new cached establishment path uses:

```text
ProtosValueLookup.lookupGuarded(...)
```

and retains:

```text
selected ProtosClosureValue
selected methodHome
GuardedStructuredKind
Boolean StructuredCallbackKind where applicable
selector-specific Assumption
```

The selected Closure must be native, and the selected behavior/home must satisfy an existing canonical-selection helper:

```text
ProtosStandardObjectProtocol.isCanonicalStandardEnsureSelection
ProtosStandardErrorProtocol.isCanonicalStandardHandleSelection
ProtosStandardObjectProtocol.isCanonicalStandardWhileSelection
ProtosStandardBooleanProtocol.structuredCallbackKindForCanonicalSelection
```

A selected native Closure that does not satisfy one of these canonical provenance contracts is not admitted to the specialization.

## Valid-hit structure

For an admitted stable canonical selection, `guardedStructuredSend`:

1. reuses the cached selected Closure/home/kind under the cached lookup-stability Assumption;
2. creates the same fresh immediate-method activation required by the existing structured semantics;
3. preserves Task/dynamic-control inheritance;
4. feeds the exact cached structured kind directly into the existing `finishPreparingComposedCall` / `PreparedClosureCall.nativeCall` path.

Therefore the valid hit does not rerun:

```text
general D013 traversal
generic selected-value classification
finishPreparingComposedCallByImplementation classifier scan
```

for the migrated canonical families.

## Semantic fallback and invalidation

The generic `perform` specialization remains the fallback authority.

The guarded path is constrained by receiver, selector, entered Context and the Phase A selector-specific Assumption.

Noncanonical or invalid selections fall through to the existing generic path, including:

```text
override
alias/copy at a noncanonical home
different selected behavior
invalidated selected slot
unsupported lookup representation
```

The implementation does not authorize a structured operation from selector spelling alone.

```text
NO_STANDARD_PROTOCOL_PRIVILEGING=YES
```

## Focused regression evidence in the product

The slice adds:

```text
src/test/java/com/guillermomolina/protos/execution/
ProtosI072PhaseEStructuredSendConvergenceTest.java
```

The retained test source directly exercises two important guarded-path properties:

### Warmed ensure invalidation

The same lowered call site executes canonical `ensure` repeatedly, then a nearer `ensure` slot is added to the receiver.

The test requires the subsequent execution to observe the override and requires the old structured cleanup path not to run again.

The test emits:

```text
I072_PHASE_E_WARMED_ENSURE_HIT_INVALIDATES_ON_OVERRIDE=PASS
```

when successful.

### Warmed Error.handle identity correctness

A warmed `Error.handle` call site is executed repeatedly while the body observes distinct Error objects across iterations.

The test requires each invocation to return the current Error identity rather than stale state retained by the guarded specialization.

The test emits:

```text
I072_PHASE_E_WARMED_ERROR_HANDLE_HIT_REMAINS_CORRECT=PASS
```

when successful.

This durable record describes the published source/test contract; it does not manufacture execution results that were not supplied independently.

## Preserved prior architecture

The slice does not modify the Phase B/C/D representation boundaries.

The product delta relative to Phase D is limited to:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseEStructuredSendConvergenceTest.java
```

No compact call-ABI, conditional Context/Array materialization, or prepared-call physical separation change is introduced by this slice.

## Explicitly remaining Phase E scope

The root changelog at `0.3.95-SNAPSHOT` explicitly records that this is **Phase E slice 1**, not Phase E closure.

Remaining structured families include:

```text
Array callbacks
Bytes callbacks
ProcessArguments callbacks
Environment callbacks
IdentityMap callbacks
Map callbacks

Map read lookup
Map atPut
Map remove

Object.caseOf / match-related structured paths
```

The product also explicitly keeps the current generic compatibility machinery where it remains required by those paths, including the current `StructuredCallCapabilities` / classifier-routing layer.

Dead compatibility machinery review therefore remains deferred until all dependent Phase E paths have migrated.

## Publication and remote CI state

The product revision is published on `guillermomolina/protos`.

At the time this durable record was prepared, the GitHub Actions `test` check for:

```text
e11ca30fd3345da8813114bcffbb76ba0c0f234d
```

was still in progress.

No remote-CI PASS is inferred while that check remains incomplete.

## Current state

```text
I072_PHASE_E_SLICE_1=COMPLETE

GUARDED_STRUCTURED_ENSURE=IMPLEMENTED
GUARDED_STRUCTURED_ERROR_HANDLE=IMPLEMENTED
GUARDED_STRUCTURED_WHILE=IMPLEMENTED
GUARDED_STRUCTURED_BOOLEAN=IMPLEMENTED

PHASE_A_LOOKUP_STABILITY_MODEL_REUSED=YES
SELECTOR_NAME_ALONE_AUTHORIZES_SPECIALIZATION=NO
GENERIC_FALLBACK_PRESERVED=YES

REMAINING_COLLECTION_STRUCTURED_FAMILIES=MIGRATION_REQUIRED
DEAD_COMPATIBILITY_REVIEW=DEFERRED_UNTIL_DEPENDENTS_MIGRATE

PHASE_E=IN_PROGRESS
I072_STATE=OPEN
```

## Next implementation slice

The next I072 slice remains Phase E implementation in `guillermomolina/protos`.

It must extend the already-established `guardedStructuredSend` convergence to the remaining canonical collection/map structured families, preserve exact provenance/invalidation and generic/direct-Closure fallback, and only then re-evaluate dead compatibility machinery and final I072 closure.
