# I072 Phase A — guarded selected-send implementation evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase A — guarded selected-send substrate
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=2d8f04a8a01ff8639e98e03fba9a176170d54936
PROTOS_REVISION=db7171058ecf1cd3867c28dc24f954df704004e3
PROTOS_VERSION=0.3.90-SNAPSHOT
COMMIT_MESSAGE=I072-A: guard stable selected sends
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PLAT040_COMPARATIVE_EVIDENCE=docs/project/evidence/PLAT040/PLAT040_CROSS_TRUFFLE_HOT_CALL_DECISION_EVIDENCE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_A_STATE=COMPLETE
I072_STATE=OPEN

## Result

I072 Phase A publishes the first production layer of the ratified PLAT040
Candidate F-prime architecture: exact D013-equivalent selected-send stability
is represented by a selector-specific Truffle `Assumption`, and the stable
monomorphic hit reaches the already-selected Closure, exact `methodHome`, and
Context-owned `RootCallTarget` without repeating general lookup or selected
value classification.

The phase deliberately retains the pre-existing invocation representation.
`ProtosActivation`, the current execution-context materialization behavior,
the supplied-argument representation, and `PreparedClosureCall` remain in the
hit. Their removal or compaction belongs to later I072 phases.

The published Phase A outcome is:

```text
D013_GUARDED_HIT_EQUIVALENCE=PASS
LOOKUP_STABILITY_INVALIDATION=PASS
GENERAL_LOOKUP_ON_VALID_HIT=NO
DELEGATION_TRAVERSAL_ON_VALID_HIT=NO
GENERIC_CLASSIFICATION_ON_VALID_HIT=NO
SELECTED_CLOSURE_STABLE=YES
METHOD_HOME_STABLE=YES
TARGET_STABLE=YES

CURRENT_ACTIVATION_CONSTRUCTION_PRESERVED=YES
CURRENT_CONTEXT_MATERIALIZATION_PRESERVED=YES
CURRENT_ARGUMENT_REPRESENTATION_PRESERVED=YES
GENERIC_FALLBACK=PASS

PLAT036_I068_COMPATIBILITY=PASS
PLAT039_COMPATIBILITY=PASS
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO

FOCAL_VALIDATION=PASS
STRUCTURAL_HOT_PATH_DIAGNOSTIC=PASS
BROADER_VALIDATION=PASS
PUBLICATION=PASS
```

## Published implementation

The product revision changes the following Phase A surfaces:

```text
src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/runtime/ProtosGuardedLookupTest.java
pom.xml
CHANGELOG.md
```

### Selector-specific lookup stability

`ProtosValueLookup.lookupGuarded` establishes a fresh Truffle `Assumption`
while executing the same authoritative lookup implementation used by generic
D013 lookup.

Every admitted ordinary object visited by the lookup registers that same
assumption for the exact selector, including nearer objects where the selector
is currently absent. This makes later addition of the same selector to a nearer
object invalidate the earlier selection just as replacement or removal of the
selected slot does.

Unsupported representations do not receive a weaker guard. Subclass-backed,
execution-context-backed, represented-value, and other unsupported chains
invalidate or reject guarded establishment and continue through authoritative
generic lookup.

### Exact mutation invalidation

`ProtosObjectValue` maintains selector-keyed weak dependency sets only for
ordinary mutable objects participating in guarded lookup.

Successful mutation of an exact selector invalidates its dependent assumptions
on:

```text
createLocalSlot
assignLocalSlot
removeLocalSlot
composeLocalSlotsFrom
```

Unrelated selector mutation and mutation of unrelated objects do not invalidate
the selected-send assumption merely for convenience. Frozen ordinary objects
need no mutable dependency registry.

The dependency bookkeeping is outside the hot compiled lookup guard; the valid
hit checks the Truffle assumption directly.

### Guarded selected-send specialization

`ProtosBytecodeRootNode.PrepareSendArguments.guardedOrdinarySend` caches:

```text
receiver identity
selector
entered ProtosLanguageContext identity
selected ProtosClosureValue
exact ProtosObjectValue methodHome
Context-owned RootCallTarget
lookup-stability Assumption
```

Cache establishment may perform D013 lookup and selected-value classification.
A valid cached hit constructs the same fresh immediate-method
`ProtosActivation`, preserves Task/dynamic-control propagation, and returns
the same `PreparedClosureCall` representation used before Phase A.

The existing generic `perform` specialization remains the exact replacement
fallback for invalidated, unsupported, absent, native, or otherwise non-admitted
cases.

## Invalidation regression evidence

The new `ProtosGuardedLookupTest` covers the Phase A invalidation contract,
including:

```text
selected slot replacement invalidates all dependent receivers
selected slot removal rejects stale methodHome and exposes inherited selection
nearer same-selector addition invalidates even when Closure identity is unchanged
unrelated names and objects do not invalidate
successful composition invalidates contributed selectors only
failed/frozen mutation leaves selection valid
execution-context storage remains on generic lookup
represented delegation-parent changes remain visible through generic fallback
```

The focused test was reported green by the human executor.

## Existing send-path regression evidence

The Phase A candidate was validated together with the retained PERF010-A prepared
target regressions. Reported PASS markers include:

```text
PERF010A_FRESH_ACTIVATION_PER_HIT=PASS
PERF010A_METHOD_HOME_EXACT_PER_HIT=PASS
PERF010A_REMOVED_OVERRIDE_OBSERVES_DELEGATION_PARENT=PASS
PERF010A_NATIVE_SEND_UNAFFECTED=PASS
PERF010A_FRESH_MATERIALIZATION_IDENTITY_CHURN_STABLE=PASS
PERF010A_MONOMORPHIC_SEND_RESULT=PASS
PERF010A_REPLACED_SELECTION_MISSES_STALE_HIT=PASS
```

The affected semantic regression set was also reported green for:

```text
ProtosPerf006B2D1OrdinarySendCompositionTest
ProtosPerf006B4BNonLocalReturnTest
ProtosPerf006B4DErrorHandlersTest
ProtosAPlusExecutionProjectionTest
```

Observed explicit markers included send suspension/no-replay, immediate receiver
and `methodHome`, exact non-local-return target/payload, dynamic Error-handler
lifetime, handler suspension/resume, and no protected-prefix replay.

## Structural hot-path discriminator

After annotation processing, the human executor inspected:

```text
target/generated-sources/annotations/
com/guillermomolina/protos/execution/ProtosBytecodeRootNodeGen.java
```

The generated DSL node showed two materially different paths.

Specialization establishment calls
`PrepareSendArguments.createGuardedSend(...)`, captures
`cachedSend.stability()`, and installs the guarded cache entry.

The normal generated valid-hit path instead performs:

```text
Assumption.isValidAssumption(cached assumption)
receiver == cachedReceiver
selector.equals(cachedSelector)
currentEnteredContext()
enteredContext == cachedContext
-> PrepareSendArguments.guardedOrdinarySend(...)
```

The general `performOrdinarySendLookup` and
`ordinarySendClosureOrNull` sequence remains in the older fallback
specialization after the guarded path; it is not executed by the valid guarded
hit.

Therefore the Phase A structural discriminator is satisfied:

```text
GENERAL_D013_LOOKUP_ON_VALID_HIT=NO
DELEGATION_TRAVERSAL_ON_VALID_HIT=NO
GENERIC_SELECTED_VALUE_CLASSIFICATION_ON_VALID_HIT=NO
SELECTED_CLOSURE_STABLE=YES
METHOD_HOME_STABLE=YES
TARGET_STABLE=YES
CURRENT_ACTIVATION_CONSTRUCTION_PRESERVED=YES
CURRENT_CONTEXT_MATERIALIZATION_PRESERVED=YES
CURRENT_ARGUMENT_REPRESENTATION_PRESERVED=YES
GENERIC_FALLBACK_EXACT=YES
```

This is structural/compiler evidence only. Phase A does not claim attributable
timing improvement; PERF010-A remains the owner of timing interpretation.

## Broad validation and publication

The human executor reported the canonical integrated gate:

```text
make test
RESULT=PASS
```

This full-suite run covered the substantive executable candidate. Finalization
then changed only the Maven implementation version and root changelog metadata;
no executable/dependency bytes changed after the reported full-suite PASS.

Before publication:

```text
HEAD=2d8f04a8a01ff8639e98e03fba9a176170d54936
origin/main=2d8f04a8a01ff8639e98e03fba9a176170d54936
git diff --check=PASS
FINAL_VERSION=0.3.90-SNAPSHOT
```

The resulting product commit is:

```text
db7171058ecf1cd3867c28dc24f954df704004e3
I072-A: guard stable selected sends
Refs #719. AI assistance: substantial.
```

GitHub confirms that commit directly follows the audited baseline
`2d8f04a8a01ff8639e98e03fba9a176170d54936`.

## Phase boundary

Phase A intentionally does not implement:

```text
compact ordinary-call frame-argument ABI
conditional guest execution-context materialization
removal of eager guest supplied-argument transport
optional control/state compaction
structured-control convergence
```

Those remain owned by later I072 phases.

## Next slice

```text
NEXT_I072_PHASE=Phase B — compact ordinary-call ABI
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

Phase B must start from current `origin/main`, re-read the live I072/#719
contract and PLAT040 durable authority, and derive the compact frame-argument ABI
from current source evidence and comparable mature Truffle runtime patterns. It
must not opportunistically begin Phase C execution-context or argument
materialization work.
