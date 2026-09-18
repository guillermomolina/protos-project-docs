# D147 — Explicit Core semantic-family and receiver-domain recognition

## Decision

D147 ratifies **Candidate B-prime — evidence-scoped standard-owner `recognizes(value)` operations**.

```text
D147_STATUS=RATIFIED
SELECTED_CANDIDATE=B_PRIME
PROTOS_REVISION=b1b5c91b365a57ed65797b78ab9a6466e7df16f5
PROJECT_RECORD_BASE=ee6ec279d160246499460450fca16bb54ab275e2
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative only through the applicable ratified material
under `guillermomolina/protos:spec/**`.

## Owner approval

The project owner explicitly approved the exact candidate in the active D147
interaction:

```text
aprobado
```

The approval refers to:

```text
docs/project/work/D147/D147_EXPLICIT_CORE_SEMANTIC_FAMILY_VALIDATION_DECISION_PACKET.md
PACKET_REVISION=ee6ec279d160246499460450fca16bb54ab275e2
SELECTED_CANDIDATE=B_PRIME
```

## Ratified public model

Core adds the standard selector:

```text
recognizes(value)
```

to exactly these canonical standard owners:

```text
String
Integer
Float
Array
```

The exact recognized domains are:

```text
String.recognizes(value)
    true iff value is a semantic String value

Integer.recognizes(value)
    true iff value is an ordinary unbounded Integer value

Float.recognizes(value)
    true iff value is a semantic Float value

Array.recognizes(value)
    true iff value owns standard Array indexed state
```

## Common recognition contract

Each selected standard recognizer:

- requires the exact canonical standard owner as receiver;
- accepts exactly one argument;
- accepts any candidate value as that argument;
- returns canonical `true` exactly on recognition;
- returns canonical `false` on candidate mismatch;
- performs no candidate message lookup or dispatch;
- performs no candidate `parent()` lookup;
- performs no candidate equality or hash dispatch;
- invokes no callback;
- performs no conversion or coercion;
- does not allocate a converted replacement value;
- does not grant recognition through delegation;
- does not permit user behavior to claim Core semantic membership.

Invalid recognizer receiver or arity follows the ordinary standard
receiver/arity Error rule.

## Exact family/state boundaries

### String

`String.recognizes(value)` observes exact semantic String-family membership.

An ordinary object merely delegating to `String` or to a String value is not
recognized.

### Integer

`Integer.recognizes(value)` recognizes only the ordinary unbounded Integer
semantic family.

Therefore fixed-width values such as `UInt8(1)`, `Int32(1)`, and the other
fixed-width integer families are not recognized by `Integer.recognizes`.

Internal implementation categories such as SmallInteger/BigInteger remain
unobservable and do not alter recognition.

### Float

`Float.recognizes(value)` observes exact semantic Float-family membership.

No implicit numeric conversion or promotion participates in recognition.

### Array

`Array.recognizes(value)` observes ownership of standard Array indexed state.

Immediate delegation parent identity is not the criterion.

Consequently:

- an ordinary object that merely delegates to `Array` is not recognized;
- a genuine standard Array created by a descendant Array factory is recognized
  even when its immediate parent is that descendant factory;
- open, closed, and frozen standard Arrays are all recognized.

## Standard owner objects

The standard owner/factory objects are not automatically members of the
domains they recognize.

Under the current model:

```text
String.recognizes(String)   -> false
Integer.recognizes(Integer) -> false
Float.recognizes(Float)     -> false
Array.recognizes(Array)     -> false
```

No owner object becomes a semantic value or state-bearing instance merely
because it publishes the recognizer.

## Scope delta explicitly approved

The original D147 trigger focused on semantic value-family validation.

The ratified candidate explicitly widens the decision to include one
evidence-backed state-bearing receiver domain:

```text
D147_SCOPE_DELTA=
    INCLUDE_EVIDENCE_BACKED_STANDARD_ARRAY_RECEIVER_STATE_RECOGNITION
```

This was surfaced before approval and is part of the exact selected candidate.

The decision does not generalize recognition to every standard receiver domain.

## Deferred surface

D147 does not add:

```text
Number.recognizes
UInt8.recognizes
Int8.recognizes
UInt16.recognizes
Int16.recognizes
UInt32.recognizes
Int32.recognizes
UInt64.recognizes
Int64.recognizes
Map.recognizes
IdentityMap.recognizes
Bytes.recognizes
```

and does not add:

- a Boolean descriptor or `Boolean` prelude object;
- a null descriptor;
- a generic value-side type/family query;
- a first-class Core family descriptor;
- user-defined family registration;
- a universal type registry;
- matching semantic changes;
- `typeof` / `is` syntax;
- static typing;
- flow narrowing;
- casts or annotations.

These questions remain deferred until independently justified by real use.

## Programming-model invariant

The ratified recognition surface does not change Protos's normal programming
model:

> Ask whether an object responds correctly to the protocol you need unless an
> API genuinely requires one of Core's exact semantic-family or standard
> receiver-state domains.

The recognizers exist for boundaries where the exact Core domain is already part
of the API contract. They are not a preferred replacement for protocol-oriented
polymorphism.

## Existing precedent

The selected shape deliberately reuses the existing standard-owner recognition
model already established by:

```text
IpAddress.recognizes(value)
IpEndpoint.recognizes(value)
```

D147 therefore does not create a universal type relation or a new syntax
institution.

## Rationale

Current production source repeatedly performs exact recognition indirectly:

```protos
"" + value
value.div(1)
0.0 + value
```

and attempts Array recognition through:

```protos
value.parent() === Array
```

The latter is not semantically equivalent to standard Array-state ownership.

Candidate B-prime exposes the already-existing semantic/state truth directly,
using ordinary owner-local messages and only for the four domains demonstrated
by current production code.

Broader uniform catalogs, generic family relations and first-class descriptors
are intentionally deferred because they solve requirements that current code has
not demonstrated.

## GITHUB021 consistency

The ratified candidate preserves:

```text
DELEGATION_DOES_NOT_CONFER_SEMANTIC_FAMILY_MEMBERSHIP
DELEGATION_DOES_NOT_CONFER_RECEIVER_OWNED_STANDARD_STATE
STANDARD_FAMILY_BEHAVIOR_VALIDATES_ORIGINAL_RECEIVER
ORDINARY_PROTOCOL_ORIENTED_PROGRAMMING_REMAINS_DEFAULT
NO_STATIC_TYPE_SYSTEM
NO_USER_DEFINED_NOMINAL_CLASS_HIERARCHY
NO_MATCHING_SEMANTIC_CHANGE_WITHOUT_EXPLICIT_DECISION
INTERNAL_REPRESENTATION_CATEGORIES_ARE_NOT_PORTABLE_SURFACE
```

Material approved delta:

```text
NEW_PUBLIC_STANDARD_RECOGNIZERS=
    String.recognizes(value)
    Integer.recognizes(value)
    Float.recognizes(value)
    Array.recognizes(value)

ARRAY_STANDARD_STATE_RECOGNITION=PUBLIC
```

No recorded invariant is contradicted.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Normative-publication ordering

D147's closure contract requires normative publication before implementation.

Therefore the implementation owner must publish the specification reconciliation
for the ratified semantics under `guillermomolina/protos:spec/**` before
executable implementation work begins.

This ratification record does not itself satisfy that normative-publication
postcondition because project decision records are non-normative.

## Implementation routing

Normative specification and executable implementation belong to a separate
implementation work item.

That work must preserve the exact ratified scope and must not broaden
recognition to additional owners or introduce generic typing/reflection machinery
without a new decision.
