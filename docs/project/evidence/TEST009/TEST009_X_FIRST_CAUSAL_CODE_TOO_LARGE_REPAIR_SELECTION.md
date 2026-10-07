# TEST009-X — first causal CodeTooLarge repair selection

## Scope

```text
WORK_ITEM=TEST009/#795
SLICE=TEST009-X
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
EXAMINED_PROTOS_REVISION=7675b726cd95e6491bd83f0f33babb647b2ccc39
EXAMINED_COMMIT_SUBJECT=LIB014-2: add std:regex/Regex matching
```

TEST009-X continued the TEST009-W single-root `CodeTooLarge` diagnosis without
implementing a repair. The purpose was to choose the first structural causal
repair, not to enumerate possible optimizations or resume boundary chasing.

The durable W evidence remained the measurement authority for the selected
root:

```text
SELECTOR=protos-root:088d2ae81075aba8
SOURCE_FAMILY=protos/tools/test/Manifest.protos
TARGET_COMPILATION=FAILED_CODE_TOO_LARGE
AFTER_TRUFFLE_TIER_PRESENT=PASS

Throwable.fillInStackTrace()               self_size=17303
LOCAL_FRAME                                self_size=14230
EXCEPTION_CONTROL                          self_size=4417
```

W had already established:

```text
GUEST_ERROR_STACK_TRACE_CAUSE=FALSIFIED
HOST_DEFENSIVE_EXCEPTION_EXPANSION=CONFIRMED
LOCAL_FRAME_SECOND_MAJOR_AXIS=YES
BOUNDARY_CHASING=FORBIDDEN
CODE_TOO_LARGE_MICRO_REPAIR_BATCH=FORBIDDEN
LOCAL_FRAME_REPAIR_AUTHORIZED=NO
```

## Current-HEAD invariant audit

At the examined revision, `ProtosPrelude` receives a frozen `bindings` object.
Its constructor validates that the `Error` binding exists, is an ordinary
`ProtosObjectValue`, and delegates directly to the root Object. The constructor
then discards that already-proven identity, while `errorPrototype()` rereads the
same frozen slot dynamically through `readLocalSlot("Error").orElseThrow()`.

This differs materially from `Array` and the broader ordinary-binding accessors:
those bindings are present in the real Core bootstrap, but the generic
`ProtosPrelude` constructor does not currently validate all of them. Retaining
or eagerly validating those additional identities would therefore change the
temporal location of existing defensive validation and is not authorized by
this investigation.

`ProtosObjectValue` establishes the stability required for the `Error` result:
once an object is `FROZEN`, local slots cannot be created, reassigned, or
removed. The real Core bootstrap also freezes and validates the shared standard
graph before constructing `ProtosPrelude`.

Therefore the constructor-validated `Error` identity is stable for the entire
lifetime of a valid `ProtosPrelude`.

## Owner findings

### `ProtosPrelude.errorPrototype()`

W attributed `self_size=5863` of the host defensive exception expansion to this
owner, the largest individual Protos owner in the W attribution.

The failure path represented by the repeated slot lookup is not semantically
reachable after successful `ProtosPrelude` construction: the exact slot has
already been validated and the binding container is frozen. Retaining that
validated reference removes repeated dynamic lookup and `Optional` failure
machinery without weakening or relocating constructor validation.

### `ProtosPrelude.arrayPrototype()` and other ordinary bindings

W attributed `self_size=1430` to `arrayPrototype()`, but a general Prelude cache
is not yet justified. The constructor does not validate every ordinary binding,
so eager caching would advance validation that is currently lazy. These
accessors remain unchanged in the selected repair.

### `ProtosPrelude.standardErrorPrototype(String)`

W attributed `self_size=1001`. The method still has meaningful hierarchy
validation. It may benefit indirectly when comparisons to `errorPrototype()`
become a direct identity load, but TEST009-X does not authorize caching every
standard Error subtype.

### `ProtosValueLookup.lookup(...)` / `delegationParent(...)`

W attributed two `self_size=1573` paths around delegation and
`Optional.orElseThrow()`. Current HEAD contains several checked-then-consumed
`Optional` flows for which a nullable/direct private representation may later be
a coherent structural cleanup. This remains a separate candidate rather than
being bundled into the first repair.

The generic `delegationParent(...)` `UnsupportedOperationException` remains a
real host error for unsupported runtime representations and must not be removed
merely because it appears in the expansion tree.

### Task attachment and composed-invocation rejection

`attachTaskOrInheritDynamicControlState(...)` checks `caller.task().isPresent()`
and then consumes a second `caller.task()` with `orElseThrow()`. That repeated
Optional access is avoidable in principle, but changing it together with Prelude
would create the micro-repair batch that W explicitly rejected.

`rejectComposedInvocationProjection(...)` protects a genuinely invalid state:
a source Closure that requires a Context-local execution projection cannot be
composed without an entered Protos Context. Its `UnsupportedOperationException`
is therefore semantically reachable and must remain.

### `PreparedBooleanCall` / generated exhaustive switches

The observed `MatchException` paths are residual exhaustive-switch defensive
machinery with small reach relative to the principal W owners. No evidence in X
promotes them to the primary repair.

### `LOCAL_FRAME`

`LOCAL_FRAME self_size=14230` remains the second major axis. X found no evidence
that overrides W's explicit `LOCAL_FRAME_REPAIR_AUTHORIZED=NO`. Frame work stays
outside the first host-defensive cleanup and must be reconsidered only after the
selected causal repair is measured.

## Candidate ranking

### A. Retain/prevalidate canonical Prelude bindings

```text
CAUSALITY=HIGH_FOR_ERROR_ONLY
ESTIMATED_REACH_IN_W_EVIDENCE=5863_SELF_SIZE_OWNER_PLUS_DOWNSTREAM_ERROR_COMPARISONS
SEMANTIC_SAFETY=HIGH_FOR_ERROR_ONLY
ARCHITECTURAL_COHERENCE=HIGH
RISK_OF_MICRO_PATCHING=LOW_IF_LIMITED_TO_ALREADY_CONSTRUCTOR_VALIDATED_ERROR
VALIDATION_NEEDED_AFTER_IMPLEMENTATION=FULL_FUNCTIONAL_TESTS_PLUS_SAME_ROOT_COMPILER_DIAGNOSIS
```

A broad eager Prelude cache is rejected. The coherent first unit is the exact
`Error` identity already certified during construction.

### B. Remove Optional/exception machinery after proven invariants

```text
CAUSALITY=HIGH
ESTIMATED_REACH_IN_W_EVIDENCE=AT_LEAST_3146_IN_VALUE_LOOKUP_PLUS_1430_TASK_PATH
SEMANTIC_SAFETY=CASE_SPECIFIC
ARCHITECTURAL_COHERENCE=MEDIUM_TO_HIGH
RISK_OF_MICRO_PATCHING=HIGH_IF_MULTIPLE_UNRELATED_OPTIONALS_ARE_BATCHED
VALIDATION_NEEDED_AFTER_IMPLEMENTATION=OWNER_SPECIFIC_REGRESSIONS_PLUS_COMPILER_DIAGNOSIS
```

This is the strongest follow-up family, but it is not mixed into the first
repair.

### C. Split proven receiver paths from generic invalid-family fallbacks

```text
CAUSALITY=MEDIUM
ESTIMATED_REACH_IN_W_EVIDENCE=LOWER_THAN_A_OR_B
SEMANTIC_SAFETY=REQUIRES_PATH_SPECIFIC_PROOF
ARCHITECTURAL_COHERENCE=MEDIUM
RISK_OF_MICRO_PATCHING=MEDIUM
VALIDATION_NEEDED_AFTER_IMPLEMENTATION=INVALID_FAMILY_AND_HOT_PATH_REGRESSIONS
```

No current evidence authorizes removal of real invalid-family errors.

### D. Attack LOCAL_FRAME first

```text
CAUSALITY=POTENTIALLY_HIGH
ESTIMATED_REACH_IN_W_EVIDENCE=14230
SEMANTIC_SAFETY=UNPROVEN
ARCHITECTURAL_COHERENCE=UNRESOLVED
RISK_OF_MICRO_PATCHING=HIGH
VALIDATION_NEEDED_AFTER_IMPLEMENTATION=NOT_APPLICABLE_YET
```

Rejected for the first repair because W explicitly withheld authorization and X
did not establish a bounded structural frame change.

### E. No implementation yet

Rejected. The constructor-validated `Error` identity meets the implementation
selection criteria without semantic change or boundary chasing.

## Decision

```text
TEST009_X_RESULT=FIRST_CAUSAL_STRUCTURAL_REPAIR_IDENTIFIED
IMPLEMENTATION_READY=YES

SELECTED_REPAIR=Retain the constructor-validated canonical Error prototype in ProtosPrelude and make errorPrototype() return that stable identity instead of dynamically rereading frozen bindings through Optional.orElseThrow()
SELECTED_OWNER=ProtosPrelude
SELECTED_SCOPE=Constructor-validated Error canonical identity only; no eager validation or caching of other Prelude bindings

EXPECTED_CAUSAL_EFFECT=Remove the repeated dynamic Error-slot lookup and its Optional/defensive host-exception expansion from every compiled errorPrototype() use; W attributed self_size=5863 to this owner
SEMANTIC_RISK=NONE_IF_CONSTRUCTOR_VALIDATION_AND_EXCEPTION_TIMING_ARE_PRESERVED

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BOUNDARY_CHASING=NO
LOCAL_FRAME_INCLUDED=NO

NEXT_SLICE=TEST009-Y
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

TEST009-Y is bounded to `ProtosPrelude`: retain the exact `Error` object already
validated by the constructor and return it directly from `errorPrototype()`.
It must preserve all existing constructor failures and must not eager-cache or
eager-validate `Array`, standard Error subtypes, `InvalidReturn`, or the other
ordinary Prelude bindings.

The causal acceptance condition is stronger than functional tests: after the
implementation is published, rerun the same compiler diagnosis and show that
`ProtosPrelude.errorPrototype()` no longer owns the W `Throwable.fillInStackTrace()`
expansion (or has fallen to non-equivalent residual cost). If the selected root
stops failing with `CodeTooLarge`, that is direct causal confirmation.

## Post-investigation maintainer report

After this investigation, the maintainer reported:

```text
TEST009_Y_LOCAL_GIT_DIFF_CHECK=PASS
TEST009_Y_LOCAL_TESTS=PASS
```

At publication time for this record, public `guillermomolina/protos/main` still
resolved to the examined TEST009-X revision
`7675b726cd95e6491bd83f0f33babb647b2ccc39` (`LIB014-2: add std:regex/Regex matching`).
No public TEST009-Y product revision was therefore available to bind into this
record, and this document does not claim TEST009-Y product publication or causal
compiler-diagnostic closure.
