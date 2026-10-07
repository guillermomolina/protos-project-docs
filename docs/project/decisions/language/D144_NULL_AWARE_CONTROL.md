# D144 — Null-aware control, transformation and fallback semantics

## Decision

D144 ratifies **Candidate B′ — ordinary null-aware protocol on `Object`**.

```text
D144_STATUS=RATIFIED
SELECTED_CANDIDATE=B_PRIME
PROTOS_REVISION=d1bbab2c1c1023e980b43ca01e7b2adafcac05f8
PROJECT_RECORD_BASE=327f980138e90bb286a45adf084b29de71fe88b6
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative under `guillermomolina/protos:spec/**`.

The complete research, candidate comparison, scorecard, adversarial analysis,
and deferral rationale are recorded in:

```text
docs/project/work/D144/D144_NULL_AWARE_CONTROL_DECISION_PACKET.md
```

## Owner approval

The project owner explicitly approved the exact Candidate B′ in the active D144
interaction:

```text
apruebo B′
```

## Ratified public protocol

The selected standard owner and selectors are exactly:

```text
STANDARD_OWNER=Object

PUBLIC_OPERATIONS=
    ifNull(block)
    ifNotNull(block)
```

No optional-navigation or null-coalescing syntax is selected.

## Exact null domain

Standard behavior distinguishes only exact canonical `null` identity.

```text
NULL_TEST=EXACT_CANONICAL_NULL_IDENTITY
TRUTHINESS=NO
DELEGATION_TO_NULL_COUNTS_AS_NULL=NO
```

An ordinary object delegating to `null` remains a non-null identity-bearing
object and takes the non-null branch when inherited standard behavior is used.

## `ifNull(block)`

For canonical `null`, invoke `block()` exactly once through the ordinary
polymorphic invocation protocol and return its exact normal result.

For any non-null receiver, do not callability-validate or invoke `block`; return
the exact receiver.

## `ifNotNull(block)`

For canonical `null`, do not callability-validate or invoke `block`; return
canonical `null`.

For any non-null receiver, invoke `block(receiver)` exactly once through the
ordinary polymorphic invocation protocol and return its exact normal result.

The callback receives the exact receiver object as its one supplied argument.

## Evaluation and control

Both operations accept exactly one supplied positional argument.

Receiver and argument expressions retain ordinary left-to-right eager evaluation.
A Closure-producing argument expression therefore creates the Closure before the
message invocation, while the Closure body remains lazy until selected.

Callability validation is path-sensitive and occurs only when the callback is
actually reached.

Errors, non-local returns, suspension/cancellation, and other ordinary control
propagate unchanged. A selected callback's normal result is returned unchanged,
including a Future. There is no implicit await, Future adoption, wrapping,
conversion, or hidden suspension.

## Lookup and Map boundaries

D144 does not change failed lookup:

```text
missing member on non-null receiver -> ordinary Error
```

D142 remains independently authoritative for expected Map-key absence:

```text
ABSENT != PRESENT_WITH_NULL
```

D144 introduces no absence sentinel and no Map semantic change.

## GITHUB021 consistency

```text
CANONICAL_NULL_SINGLETON=PRESERVED
NO_UNDEFINED=PRESERVED
FAILED_LOOKUP_REMAINS_ERROR=PRESERVED
NULL_DELEGATION_TO_OBJECT=PRESERVED
DELEGATION_DOES_NOT_CONFER_NULL_IDENTITY=PRESERVED
NO_TRUTHINESS=PRESERVED
ORDINARY_DISPATCH=PRESERVED
ORDINARY_ARGUMENT_EVALUATION=PRESERVED
D142_ABSENT_VS_PRESENT_NULL=PRESERVED
NEW_SYNTAX=NO
NEW_OPTIONAL_MAYBE_INSTITUTION=NO
HIDDEN_REOPENING=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Intentionally deferred

D144 does not select or reserve:

- optional member/message chaining syntax;
- null-coalescing syntax;
- optional assignment;
- optional indexed mutation;
- optional slot creation;
- combined `ifNullIfNotNull`-class operation;
- eager value fallback;
- Option/Maybe;
- failed-lookup-to-null conversion.

These may be reconsidered only from future evidence.

## Normative and implementation boundary

This durable record selects the design but is not normative language authority.

Before executable implementation proceeds, the approved semantics must be
published in the applicable normative specification owner(s), including:

```text
guillermomolina/protos:spec/semantics/VALUES_AND_COLLECTIONS.md
guillermomolina/protos:spec/PROTOS_SPEC_CHANGELOG.md
```

Implementation, tests, caller migrations, implementation versioning, and any
future syntax are separate work.
