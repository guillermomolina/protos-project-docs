# TEST009-AG — post-AF causal owner selection

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AG
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
DURABLE_RECORD_REPOSITORY=guillermomolina/protos-project-docs

EXAMINED_PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
EXAMINED_PROTOS_VERSION=0.3.263-SNAPSHOT
EXAMINED_PROTOS_COMMIT=TEST009-AF: keep context projection failure out of PE
BASE_PROJECT_RECORD_REVISION=28e6b01eb28b00899ed0be33bf850bd53e7f54ee

PRODUCT_CHANGES=NONE
COMMAND_EXECUTION=NONE
ISSUE_MUTATION_DURING_INVESTIGATION=NONE
```

TEST009-AG is the read-only causal-selection slice requested after AF. Its job
was not to implement another reduction or to select the numerically largest
residual. It re-applied the established TEST009 procedure:

```text
real Truffle compilation failure
  -> stable single root
  -> current method/node expansion evidence
  -> exact causal mechanism classification
  -> framework-appropriate bounded repair
  -> same-root causal A/B
```

At publication time both public repositories still had exactly the revisions
examined by AG, so no concurrent product or durable-record advance required
reconciliation before publishing this record.

## Fixed root and post-AF state

AG keeps the selected root fixed:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
ROOT_CHANGE=NO
```

The durable AF result remains:

```text
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE

Throwable.fillInStackTrace
  frames=96
  cumulative=6864

NoSuchElementException.<init>
  frames=66
  cumulative=4884
  DELTA_FROM_POST_AD=0
```

The important distinction is:

```text
NSEE_AGGREGATE_IS_MEASURED_CURRENT=YES
NSEE_EXACT_CURRENT_OWNER_ATTRIBUTION=NO
```

The 66 / 4884 NSEE residual is therefore a current fact, but it is not itself
a causal owner. TEST009's selection rules do not permit promoting an aggregate
constructor family to an implementation repair without decomposing it to one
exact owner/callsite and mechanism.

## Current-source candidate inspection

### `ProtosPrelude.arrayPrototype()`

At the examined product revision the accessor still performs:

```java
Object binding = bindings.readLocalSlot("Array").orElseThrow();
if (!(binding instanceof ProtosObjectValue arrayPrototype)) {
    throw new IllegalStateException(
            "standard Array binding is not an ordinary object");
}
return arrayPrototype;
```

The Prelude constructor requires `bindings` to be frozen, so the binding set is
stable after Prelude construction. However, unlike the canonical `Error`
binding repaired by TEST009-Y, generic Prelude construction does not validate
and retain `Array` eagerly.

Therefore an eager Array cache would move a currently lazy failure from
`arrayPrototype()` observation to Prelude construction:

```text
ARRAY_BINDING_STABLE_AFTER_CONSTRUCTION=YES
ARRAY_ALREADY_CONSTRUCTOR_VALIDATED=NO
EAGER_ARRAY_RETENTION_VALIDATION_TIMING_CHANGE=YES
```

Array remains a viable diagnostic owner, but copying TEST009-Y mechanically is
not authorized. If later current evidence attributes the NSEE residual to this
accessor, the repair must preserve its established validation timing.

### `ProtosPrelude.standardErrorPrototype(String)`

The method still dynamically looks up the requested named binding and walks its
delegation chain until it proves membership in the canonical Error hierarchy.
Both missing bindings and out-of-hierarchy objects are meaningful runtime
validation failures.

```text
DYNAMIC_NAME_LOOKUP=YES
REAL_ERROR_HIERARCHY_VALIDATION=YES
CACHE_ALL_ERROR_SUBTYPES_AUTHORIZED=NO
```

No current post-AF evidence decomposes the NSEE residual to this method or to
one exact branch inside it.

### `ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState(...)`

The current helper remains:

```java
if (caller.task().isPresent()) {
    activation.attachTask(
            caller.task().orElseThrow());
} else {
    activation.inheritDynamicControlState(caller);
}
```

`ProtosActivation.task()` is a direct `Optional.ofNullable(task)` wrapper.
`attachTask()` makes Task identity monotonic once attached: an activation may
keep the same Task but rejects replacement by another Task.

This makes a one-observation snapshot structurally plausible, but the field is
mutable and the two reads are not themselves synchronized. Current source and
published durable evidence do not yet prove the exact publication/ownership
property required to state that replacing two reads by one observation is
semantically indistinguishable in every reachable caller state.

Additionally, the post-AF NSEE aggregate has not been attributed to this helper.

```text
CHECK_THEN_REREAD_OPTIONAL_PATTERN=YES
SINGLE_SNAPSHOT_REPAIR_PLAUSIBLE=YES
CURRENT_NSEE_ATTRIBUTION_TO_THIS_OWNER=UNKNOWN
OWNER_SPECIFIC_OBSERVABILITY_PROOF=INCOMPLETE
IMPLEMENTATION_READY=NO
```

### New source candidate: `ProtosActivation.forObjectConstruction(...)`

AG also found the same shape outside the historical shortlist:

```java
if (enclosing.task().isPresent()) {
    construction.attachTask(enclosing.task().orElseThrow());
} else {
    construction.inheritDynamicControlState(enclosing);
}
```

It has the same observability question as the bytecode-root helper and no
selected-root attribution in the current durable evidence.

```text
NEW_CANDIDATE=YES
CURRENT_CAUSAL_ATTRIBUTION=NONE
IMPLEMENTATION_READY=NO
```

### `PreparedBooleanCall.hasCallback()` and generated switch paths

`PreparedBooleanCall.kind` is final, constructor-validated non-null state, and
`hasCallback()` exhaustively covers all current `StructuredCallbackKind` enum
values:

```text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
```

This strengthens the source-level case that a generated exhaustive-switch
defensive `MatchException` path can be unreachable for legal instances.
However, the durable W evidence identified only a broader
`PreparedBooleanCall / generated exhaustive-switch defensive paths` family.
The same state machine contains other switches and defensive failures.

```text
CLOSED_ENUM_STATE_SOURCE_INVARIANT=YES
EXACT_GENERATED_EXCEPTION_OWNER_POST_AF=UNKNOWN
HAS_CALLBACK_UNIQUE_ATTRIBUTION=NO
IMPLEMENTATION_READY=NO
```

### `finishPreparingComposedCall(...)`

AG found a particularly strong source-level invariant inside the broad helper:

```java
if (closure.nativeBody().isPresent()) {
    ProtosNativeClosureBody nativeBody =
            closure.nativeBody().orElseThrow();
    ...
}
```

`ProtosClosureValue.nativeBody` is a final field and `nativeBody()` is exactly
`Optional.ofNullable(nativeBody)`. For one Closure instance there is therefore
no state transition by which the first observation can be present and the
second observation empty.

A future bounded repair could observe the final field through one Optional or
nullable snapshot and reuse that exact observation. Unlike the Task candidate,
this source-level rewrite has no publication/concurrency ambiguity.

However, W only attributed an historical aggregate of roughly 858 expansion
units to `finishPreparingComposedCall(...)` as a whole. The current post-AF
record does not show that any of the 66 NSEE frames originate at this exact
`nativeBody().orElseThrow()` callsite. The same helper also contains other
Optional unwraps and real failure branches.

```text
NATIVE_BODY_FIELD_FINAL=YES
PRESENT_THEN_EMPTY_REACHABLE=NO
BOUNDED_SNAPSHOT_REPAIR_SOURCE_SAFE=YES
CURRENT_POST_AF_NSEE_CALLSITE_ATTRIBUTION=NONE
IMPLEMENTATION_READY=NO
```

This is the strongest source-level repair candidate found by AG, but TEST009
does not permit source plausibility to substitute for current causal evidence.

### `LOCAL_FRAME`

The historical W aggregate remains:

```text
LOCAL_FRAME_W=14230
```

and included, among other owners:

```text
ProtosFrameArguments.hasCompactHeader(Object)
CachedBytecodeNode.handleLoadLocal$generic(...)
FrameWithoutBoxing.unsafePutObject(...)
ProtosFrameArguments.activation(Object)
```

That measurement predates the later causal repairs and is a multi-owner family.
There is no directly comparable post-AF decomposition that releases one frame
owner and one mechanism for repair.

```text
LOCAL_FRAME_CURRENT_OWNER=UNKNOWN
BROAD_FRAME_REFACTOR_AUTHORIZED=NO
IMPLEMENTATION_READY=NO
```

## Historical evidence compatibility

The principal candidate method bodies inspected by AG are materially unchanged
between the W checkpoint source revision and AF:

```text
ProtosPrelude.arrayPrototype
ProtosPrelude.standardErrorPrototype
ProtosBytecodeRootNode.attachTaskOrInheritDynamicControlState
PreparedBooleanCall.hasCallback
ProtosBytecodeRootNode.finishPreparingComposedCall
```

This allows the W owner list to remain useful as a candidate map. It does **not**
make the old expansion sizes current. Changes in callers, specialization,
inlining and surrounding graph structure mean historical numeric weights cannot
be promoted to current causal measurements.

## Candidate decision table

| Owner / exact sub-owner | Current reachability | Evidence status | Exact mechanism understood | Semantic/timing/observability risk understood | Current post-AF attribution sufficient | Bounded repair identifiable | Implementation-ready |
|---|---|---|---|---|---|---|---|
| `ProtosPrelude.arrayPrototype()` | YES | current aggregate + historical owner + source | partial | validation timing blocks eager retention | NO | only conditional cold-failure repair | NO |
| `ProtosPrelude.standardErrorPrototype(String)` | YES | historical + source | partial | real dynamic validation remains | NO | NO | NO |
| `attachTaskOrInheritDynamicControlState(...)` | YES | current aggregate + historical owner + source | YES at source shape | observability proof incomplete | NO | plausible one-snapshot repair | NO |
| `ProtosActivation.forObjectConstruction(...)` Task unwrap | YES | source-inferred new candidate | YES at source shape | observability proof incomplete | NO | plausible one-snapshot repair | NO |
| `PreparedBooleanCall.hasCallback()` | YES | historical family + source | generated defense plausible | closed enum state understood | NO | only after exact switch attribution | NO |
| `finishPreparingComposedCall -> nativeBody().orElseThrow()` | YES | historical method aggregate + strong source invariant | **YES** | **YES; final field, no concurrency ambiguity** | **NO** | **YES; one observation/snapshot** | **NO** |
| `finishPreparingComposedCall(...)` as a whole | YES | historical aggregate | NO | mixed | NO | NO | NO |
| `LOCAL_FRAME` | YES | historical aggregate only | NO | NO | NO | NO | NO |

## Systematic Truffle classification

AG keeps the classification already established by TEST009's upstream
investigation:

```text
cold real failure
  -> transferToInterpreter may be appropriate

state/speculation became invalid
  -> transferToInterpreterAndInvalidate may be appropriate

host helper intentionally outside PE
  -> @TruffleBoundary may be appropriate

stable invariant repeatedly represented through Optional/host machinery
  -> representation/snapshot/direct-invariant repair may be appropriate

oversized valid PE method
  -> inlining/structural-decomposition analysis

frame/local expansion
  -> frame representation / Bytecode DSL / constant-local-access analysis
```

These are classification tools, not recipes. The concrete owner still has to be
identified from current expansion evidence before applying any repair.

The missing diagnostic is exactly what Truffle's method/node expansion tracing
is intended to answer: identify which current caller/callsite owns the residual
constructor subtree rather than inferring it from a historical aggregate.

## Decision

No candidate satisfies all of TEST009's release gates simultaneously.

The strongest source-only candidate is:

```text
finishPreparingComposedCall -> nativeBody().orElseThrow()
```

because the underlying field is final and the checked-then-reread NSEE is
semantically impossible. It is nevertheless not released for implementation:
no durable public post-AF trace attributes any current NSEE frame to that exact
callsite.

The Task helper remains plausible but has two missing proofs rather than one:
current attribution and owner-specific state-publication/observability.

Therefore AG deliberately returns `NONE` instead of selecting by historical
size, patch convenience or structural attractiveness.

```text
TEST009_AG_RESULT=SELECT_NONE_INSUFFICIENT_CURRENT_CAUSAL_ATTRIBUTION

SELECTED_OWNER=NONE
SELECTED_MECHANISM=NONE
SELECTED_REPAIR=NONE

WHY_THIS_OWNER_NOW=No candidate clears the mandatory current-causal-attribution gate; the strongest source-level candidate is finishPreparingComposedCall nativeBody Optional reread, but no public post-AF evidence attributes any of the 66 current NSEE frames to that exact callsite.

EVIDENCE_STATUS=mixed: MEASURED_CURRENT aggregate NSEE 66 frames / 4884 cumulative + HISTORICAL owner attribution from W + SOURCE_INFERRED current-source invariants

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO
CONCURRENCY_OR_OBSERVABILITY_CHANGE=NO

MULTI_OWNER_BATCH=NO
ROOT_CHANGE=NO

EXPECTED_CAUSAL_SIGNATURE=NONE

IMPLEMENTATION_READY=NO
MISSING_EVIDENCE=fresh post-AF same-root decomposition of the 66 NoSuchElementException.<init> frames to exact Protos owner/callsite; if attachTaskOrInheritDynamicControlState wins, additionally prove that one Task observation preserves publication/concurrency semantics

TEST009_COMPLETE=NO

NEXT_SLICE=TEST009-AH
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_REPOSITORY=NONE
NEXT_SCOPE=consume any newly public post-AF same-root method/node-expansion evidence and attribute the residual NoSuchElementException constructor tree to one exact owner/mechanism; if no such public evidence exists, report the exact acquisition evidence still missing without executing diagnostics
```

## AH handoff constraint

TEST009-AH remains an **investigation** slice. Under the project investigation
execution boundary it must not run diagnostic commands itself. It may inspect
current public repository HEADs, live Issue #795, durable records, and public
upstream Truffle/Graal sources.

Therefore AH has two legitimate outcomes:

1. if new public post-AF same-root diagnostic evidence exists by the time AH
   runs, consume it and attempt the exact owner/mechanism selection; or
2. if no such evidence exists, return `SELECTED_OWNER=NONE` and identify the
   exact trace/acquisition evidence that a later human-executed diagnostic slice
   must publish before another implementation can be authorized.

AH must not manufacture current attribution from W's historical sizes.

AI assistance: this durable record was drafted with ChatGPT from the published
AF product revision, the public TEST009/#795 work log, existing TEST009 durable
evidence, current product source at the examined revision, and upstream Truffle
classification already incorporated by the systematic TEST009 procedure.