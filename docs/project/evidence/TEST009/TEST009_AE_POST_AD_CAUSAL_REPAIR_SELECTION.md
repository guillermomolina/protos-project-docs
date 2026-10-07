# TEST009-AE — post-AD single-owner causal repair selection

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-AE
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
EXAMINED_PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
EXAMINED_PROTOS_VERSION=0.3.261-SNAPSHOT
EXAMINED_COMMIT_SUBJECT=TEST009-AD: keep unsupported lookup failure out of PE
PRODUCT_CHANGES=NONE
COMMAND_EXECUTION=NONE
```

TEST009-AE selects the next single causal repair after TEST009-AD for the
fixed compiler root:

```text
ROOT_SPEC=protos/tools/test/Manifest.protos
ROOT_LABEL=protos-root:088d2ae81075aba8
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
```

This investigation does not rank candidates by patch convenience and does not
batch residual owners. It applies the systematic Truffle compilerability
procedure already established by TEST009:

```text
real Truffle compilation failure
  -> select one stable root
  -> inspect expansion / inlining evidence
  -> classify the causal failure mechanism
  -> choose the framework-appropriate bounded remedy
  -> validate the repair with a same-root causal A/B
```

The earlier TEST009 systematic-procedure record had already rejected automatic
`@TruffleBoundary` search as design authority. AE therefore treats upstream
patterns as evidence for the mechanism classification, not as source-edit
recipes.

## Public HEAD reconciliation

Public `guillermomolina/protos/main` still resolves to the published AD
revision:

```text
92db72eb8b1e5d66f316a7e9f72a8c7023a28278
TEST009-AD: keep unsupported lookup failure out of PE
```

No concurrent product advance needs reconciliation for AE.

## Post-AD causal state

TEST009-AD established on the same logical Case and fixed root:

```text
Throwable.fillInStackTrace
  post-AC = 9867 / 138 frames
  post-AD = 8294 / 116 frames
  delta   = -1573 / -22 frames

UnsupportedOperationException.<init> cumulative
  post-AC = 3108 / 42 frames
  post-AD = 1480 / 20 frames

NoSuchElementException.<init>
  post-AD = 4884 / 66 frames

LOCAL_CHECKED_OPTIONAL_NSEE_SUBTREE=0
```

The 22 frames / 1628 cumulative subtree attributed to
`ProtosValueLookup.delegationParent` disappeared completely.

The remaining `UnsupportedOperationException.<init>` cumulative expansion is
attributed to one already-known owner:

```text
ProtosBytecodeRootNode.rejectComposedInvocationProjection
UOE_REMAINING_CUMULATIVE=1480
UOE_REMAINING_FRAMES=20
```

Its previously recorded expansion profile remained unchanged across AD:

```text
403 / 1693 / 1592
```

AE does not reinterpret those three values without the original tree-field
context. Their relevant property here is stability across the prior isolated
repair.

The aggregate NSEE residual is numerically larger than the remaining UOE, but
post-AD durable evidence does not attribute its 4884 / 66 frames to one exact
remaining owner. That distinction matters: AE selects on causal attribution plus
repairability, not raw aggregate size.

## Systematic Truffle classification

The selected branch is:

```java
private static void rejectComposedInvocationProjection(
        ProtosClosureValue closure) {
    if (closure.requiresContextLocalExecutionProjectionForRuntime()
            && ProtosLanguageContext.currentIfEnteredForRuntime() == null) {
        throw new UnsupportedOperationException(
                "Context-local Closure projection requires an entered Protos Context");
    }
}
```

The failure is semantically real. A Closure whose execution plan requires a
Context-local projection cannot be composed when no Protos Context is entered.
The failure must therefore remain a failure; it must not be converted to a miss,
sentinel, guest Error, or stackless control carrier.

The two conditions do not update specialization state or invalidate a compiled
assumption:

- `requiresContextLocalExecutionProjectionForRuntime()` reads the Closure's
  implementation marker;
- `currentIfEnteredForRuntime()` observes whether a compatible Context is
  currently entered.

The Context lookup helper is already a `@TruffleBoundary`, so its host lookup
work is already cut out of partial evaluation. The residual expansion comes from
the Java UOE construction after that helper returns `null`.

This classifies the residual as:

```text
FAILURE_CLASS=HOST_EXCEPTION_CONSTRUCTION_IN_PE
FAILURE_SEMANTICS=REAL_REACHABLE_FAILURE
SPECIALIZATION_STATE_CHANGE=NO
ASSUMPTION_INVALIDATION=NO
```

The systematic remedy is therefore:

```text
CompilerDirectives.transferToInterpreter()
```

immediately before the existing UOE construction/throw.

This is intentionally not:

```text
CompilerDirectives.transferToInterpreterAndInvalidate()
@TruffleBoundary
stackless/control-flow exception
semantic miss/fallback
```

## Upstream mechanism check

The Graal/Truffle API at the Graal revision aligned with the TEST009 V3 tooling
(`95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`) defines:

- `transferToInterpreter()`: discontinue compilation at this position and
  transfer execution to the interpreter;
- `transferToInterpreterAndInvalidate()`: do the same and additionally
  invalidate currently executing machine code.

That distinction matches the selected Protos branch: execution must leave the
compiled path for a cold real failure, but no observed fact invalidates the
compiled specialization or assumptions.

The same Graal revision contains direct framework/language precedents for the
non-invalidating shape:

```java
CompilerDirectives.transferToInterpreter();
throw new UnsupportedOperationException(...);
```

including Truffle NFI internal-language access and Wasm instrumentation paths.
GraalPython similarly uses non-invalidating interpreter transfers before
constructing/raising cold failures.

These examples are not copied mechanically. Their shared architectural property
is the relevant evidence: a real exceptional outcome must be preserved while
its construction does not belong in the compiled fast path, and observing the
failure does not itself invalidate compiled state.

## Candidate comparison

| Owner | Current reachability | Post-AD causal evidence | Exact mechanism | Semantic/timing risk | Implementation-ready |
|---|---|---|---|---|---|
| `rejectComposedInvocationProjection` | YES | MEASURED_CURRENT: UOE 1480 / 20, uniquely attributed | real UOE construction remains PE-reachable | low; preserve exact failure | **YES** |
| `ProtosPrelude.arrayPrototype()` | YES | aggregate NSEE current; owner size only historical | lazy slot lookup uses `orElseThrow()` | eager retention would move validation | NO |
| `ProtosPrelude.standardErrorPrototype(String)` | YES | owner size historical | dynamic named lookup plus real Error-hierarchy validation | validation remains meaningful | NO |
| `attachTaskOrInheritDynamicControlState(...)` | YES | aggregate NSEE current; owner size historical | `caller.task().isPresent()` followed by a second `caller.task().orElseThrow()` | single-snapshot repair plausible, but exact post-AD attribution and concurrency/observability proof are not yet complete | NO |
| `PreparedBooleanCall.hasCallback(...)` | YES | current exact owner attribution UNKNOWN | exhaustive-switch family may introduce generated defensive `MatchException` paths | constructor/helper attribution not proven | NO |
| `finishPreparingComposedCall(...)` | YES | current exact owner attribution UNKNOWN | broad structured-call classification; no one material expansion isolated | no single repair demonstrated | NO |
| `LOCAL_FRAME` | YES | HISTORICAL aggregate 14230; no directly comparable current classifier | spans compact frame headers, local load/store, generated bytecode, materialization and lexical/frame boundaries | broad multi-owner refactor would violate the bounded-slice rule | NO |

### Array

`arrayPrototype()` still performs:

```java
bindings.readLocalSlot("Array").orElseThrow()
```

followed by ordinary-object validation.

Unlike `Error` in TEST009-Y, `Array` is not already constructor-validated and
retained under the generic `ProtosPrelude` construction contract. Retaining it
eagerly would move failure timing from accessor use to Prelude construction.

```text
VALIDATION_TIMING_CHANGE=YES
```

Therefore Array is not a valid mechanical repetition of Y.

### Standard Error prototypes

`standardErrorPrototype(String)` dynamically selects a binding by name and then
walks its delegation parents to prove membership in the canonical Error
hierarchy. The validation can still fail legitimately. The dynamic name and
hierarchy check mean that retaining/caching these values is not equivalent to
the already-proven stable `errorPrototype()` repair.

### Task attachment

`attachTaskOrInheritDynamicControlState(...)` remains a strong later candidate:

```java
if (caller.task().isPresent()) {
    activation.attachTask(caller.task().orElseThrow());
} else {
    activation.inheritDynamicControlState(caller);
}
```

`ProtosActivation.task()` wraps the current `task` field in an Optional, and
`attachTask` makes that field monotonic with respect to task identity in the
inspected implementation. A future bounded repair may therefore capture one
`Optional<ProtosTask>` snapshot and reuse it.

AE nevertheless does not select it now because the post-AD 4884 / 66 aggregate
NSEE is not durably decomposed to that one owner, and AE does not treat the W
historical 1430 attribution as a current measurement. The concurrency and
observability argument for replacing two reads by one snapshot also deserves an
owner-specific proof before implementation.

### PreparedBooleanCall

`PreparedBooleanCall.hasCallback()` is an exhaustive switch, but the same state
machine contains other exhaustive switches and explicit defensive failures.
The durable evidence identifies a PreparedBooleanCall/generated-switch family,
not one exact constructor uniquely owned by `hasCallback()`.

```text
IMPLEMENTATION_READY=NO
```

until the exact generated/host exception mechanism is attributed.

### finishPreparingComposedCall

The helper aggregates multiple structured-protocol classifications. Its
historical appearance in the expansion tree does not identify one exact
exception construction or one bounded structural repair.

```text
IMPLEMENTATION_READY=NO
```

### LOCAL_FRAME

The W value:

```text
LOCAL_FRAME_W=14230
```

was a large family aggregate that included, among other costs,
`ProtosFrameArguments.hasCompactHeader`, generated
`CachedBytecodeNode.handleLoadLocal$generic`, `FrameWithoutBoxing` access and
activation/materialization work.

The exact W classifier is no longer available as a directly comparable
post-AD measurement. More importantly, the evidence still does not collapse
LOCAL_FRAME to one owner/mechanism. Selecting it now would authorize the broad
frame refactor AE explicitly forbids.

## Expected AF causal signature

TEST009-AF must edit only the selected failure branch.

The primary causal acceptance condition is:

```text
rejectComposedInvocationProjection UOE subtree in PE -> removed
```

while preserving:

```text
exception class
exception message
failure position/contract
NoSuchElementException aggregate as independent control
other measured residual owners
selected root identity
```

AF must not predeclare an exact `Throwable.fillInStackTrace` delta from the
1480 UOE cumulative number. The correct proof is a fresh same-root A/B showing
that the selected UOE construction no longer expands in PE and that equivalent
cost has not merely moved elsewhere.

The selected root may still fail `CodeTooLarge` after AF; AF succeeds causally
if its one selected subtree disappears cleanly.

## Decision

```text
TEST009_AE_RESULT=SELECT_REJECT_COMPOSED_INVOCATION_PROJECTION_PE_CUT

SELECTED_OWNER=ProtosBytecodeRootNode.rejectComposedInvocationProjection
SELECTED_MECHANISM=real reachable context-local projection failure constructs UnsupportedOperationException in PE after no entered Protos Context is observed
SELECTED_REPAIR=insert CompilerDirectives.transferToInterpreter() immediately before the existing UnsupportedOperationException construction/throw, with no invalidation and no new boundary

WHY_THIS_OWNER_NOW=it is the only post-AD UOE owner with exact current attribution (1480 cumulative / 20 frames), its failure contract is real and source-localized, and Truffle supports a non-invalidating interpreter transfer for this cold failure without changing semantics

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PUBLIC_API_CHANGE=NO
VALIDATION_TIMING_CHANGE=NO

MULTI_OWNER_BATCH=NO
ROOT_CHANGE=NO

IMPLEMENTATION_READY=YES
MISSING_EVIDENCE=NONE

NEXT_SLICE=TEST009-AF
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=add only CompilerDirectives.transferToInterpreter() immediately before the existing rejectComposedInvocationProjection UnsupportedOperationException and causally remeasure protos-root:088d2ae81075aba8
```

AI assistance: this record was drafted with ChatGPT from the public
`guillermomolina/protos` HEAD, public TEST009/#795 history, durable TEST009
records, and the pinned public Graal/Truffle sources identified above. The
project owner explicitly approved the selected AE recommendation before this
record was published.
