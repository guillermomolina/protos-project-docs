# D146 — Fresh generic Error signaling ergonomics

## Decision

D146 ratifies **Candidate B-prime — one ordinary zero-argument standard prelude `fail` callable**.

```text
D146_STATUS=RATIFIED
SELECTED_CANDIDATE=B_PRIME
PROTOS_REVISION=f61bd24cf1e935591c00a07832e006fb7a256828
PROJECT_RECORD_BASE=eaaf5ae30edb8d40dad9724ca01ac6f7fad3923c
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative only through the applicable ratified material
under `guillermomolina/protos:spec/**`.

## Owner approval

The project owner explicitly approved the exact candidate in the active D146 interaction:

```text
apruebo B'
```

The approval refers to:

```text
docs/project/work/D146/D146_FRESH_GENERIC_ERROR_SIGNALING_DECISION_PACKET.md
PACKET_REVISION=eaaf5ae30edb8d40dad9724ca01ac6f7fad3923c
SELECTED_CANDIDATE=B_PRIME
```

## Ratified public surface

The standard frozen prelude gains exactly one binding:

```text
fail
```

Its value is an ordinary standard callable.

```protos
fail()
```

is the complete initial public operation.

The binding is not:

- syntax;
- a keyword;
- an intrinsic;
- a reserved identifier.

It participates in ordinary lexical lookup and may be shadowed by an explicit
local binding created with `:`.

## Exact semantic contract

A valid reached call:

```protos
fail()
```

does exactly this semantically:

1. create one fresh ordinary generic standard Error occurrence;
2. give that occurrence immediate parent equal to the canonical standard
   `Error` prototype for the current Core environment;
3. enter the existing D005 non-resumable Error signaling operation with that
   exact fresh object.

The operation never returns normally.

## Freshness and identity

Every reached valid invocation is one distinct standard failure occurrence.

Therefore:

```text
FRESH_IDENTITY_PER_OCCURRENCE=YES
ERROR_POOLING_OR_SINGLETON=NO
```

The existing D005 rules remain authoritative:

- identity is observable where ordinary Error identity is observable;
- separate occurrences must remain distinct when identity can be observed;
- a recorded Future failure preserves the exact recorded Error on same-domain
  re-observation;
- crossing boundaries follows the already-ratified Error transfer/reconstruction
  semantics.

## Canonical Error provenance

The standard `fail` facility uses the canonical standard `Error` prototype.

It does not resolve a caller-local lexical binding named `Error`.

Therefore:

```protos
Error: someOtherObject
fail()
```

still creates and signals a fresh standard generic Error.

Likewise, extracting the callable:

```protos
f: fail
f()
```

does not change its standard meaning according to the caller's local bindings.

This consequence is part of the exact approved decision.

## Arity and arguments

The standard `fail` callable accepts exactly zero supplied arguments.

D146 does not add:

- message arguments;
- cause arguments;
- payload/data arguments;
- existing-Error arguments;
- Error-prototype arguments;
- rethrow/current-error mode.

A call with supplied arguments follows the ordinary standard callable arity
failure contract. It does not first execute the generic `fail()` occurrence.

## Existing Error semantics preserved

D146 does not reinterpret:

```protos
Error.signal()
error.signal()
```

The former still signals the standard `Error` prototype itself.

The latter still signals exactly `error`.

D146 also does not alter:

- handler matching;
- exact re-signaling identity;
- unwinding;
- `ensure` behavior;
- Future failure identity;
- Actor/P boundary semantics;
- the shallow standard Error taxonomy;
- the non-resumable Error model.

## Prelude-name compatibility

After D146, a previously absent bare lookup of:

```text
fail
```

resolves to the standard prelude binding.

That is an intentional compatibility change.

Existing explicit local bindings such as:

```protos
fail: () => {
    ...
}
```

remain valid and shadow the standard prelude binding under ordinary lexical lookup.

The name is not reserved.

## Error payload, diagnostics and taxonomy

D146 adds no new standard Error prototype and no mandatory guest-visible Error
payload.

It does not standardize:

- message;
- cause;
- code;
- source;
- stack;
- location;
- arbitrary data;
- checked/unchecked classification.

Implementation-private diagnostic metadata remains governed by D005.

## Assertion interaction

Bare standard:

```protos
fail()
```

always means fresh generic standard Error.

It never means `AssertionFailure`.

The test library remains free to define qualified assertion-specific operations
under its own module/API if separately designed later.

## Safe migration boundary

A source occurrence may migrate:

```protos
Error().signal()
```

to:

```protos
fail()
```

only when the original expression intentionally creates a fresh generic Error
using the canonical standard `Error` binding.

Do not mechanically migrate:

- `error.signal()`;
- `Error.signal()`;
- custom Error-subtype construction/signaling;
- stateful local `fail` helpers;
- code whose local `Error` binding intentionally changes construction.

## Deferred surface

D146 does not decide:

- custom Error-prototype factory shorthand;
- message-bearing constructors;
- cause chains;
- payloads;
- guest-visible stack/source reflection;
- dynamic rethrow convenience;
- resumable conditions/restarts;
- assertion-specific `fail`;
- throw/raise syntax;
- checked/unchecked exceptions;
- Result/Option redesign.

These remain independent future decisions.

## Programming-model invariant

D146 is an ergonomic convenience over already-ratified generic Error signaling.

It does not imply that generic Error is the preferred representation for every
domain failure.

Expected domain failures may continue to use ordinary values, and domain-specific
exceptional contracts may continue to use specific Error prototypes where
appropriate.

## GITHUB021 consistency

Applicable D005 invariants:

```text
FRESH_STANDARD_FAILURE_OCCURRENCE=PRESERVED
STANDARD_FAILURE_IDENTITY_IS_OBSERVABLE=PRESERVED
EXISTING_ERROR_RESIGNAL_USES_EXACT_OBJECT=PRESERVED
ERROR_PROTOTYPES_ARE_NOT_FAILURE_SINGLETONS=PRESERVED
Error.signal()_SIGNALS_Error_ITSELF=PRESERVED
SIGNALING_IS_NON_RESUMABLE=PRESERVED
PORTABLE_ERROR_TAXONOMY_REMAINS_SHALLOW=PRESERVED
NO_IMPLICIT_ERROR_PAYLOAD_OR_CONVERSION=PRESERVED
NO_GUEST_VISIBLE_STACK_STRUCTURE_FROM_D146=PRESERVED
```

Material approved deltas:

```text
NEW_STANDARD_PRELUDE_BINDING=fail
FAIL_BINDING_IS_ORDINARY_AND_SHADOWABLE=YES
FAIL_IS_EXTRACTABLE_CALLABLE=YES
FAIL_ARITY=ZERO

FAIL_USES_CANONICAL_STANDARD_ERROR=YES
FAIL_USES_CALLER_LEXICAL_Error_BINDING=NO

PREVIOUSLY_ABSENT_BARE_fail_LOOKUP_BECOMES_BOUND=YES
LOCAL_fail_CREATION_STILL_SHADOWS_STANDARD=YES
```

No existing owner-approved invariant is reopened or contradicted.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Normative-publication ordering

D146's closure contract requires normative publication before implementation.

Therefore the implementation owner must publish the specification reconciliation
for the ratified semantics under `guillermomolina/protos:spec/**` before
executable implementation begins.

This project record is non-normative and does not itself satisfy that condition.

## Implementation routing

Normative publication and executable implementation belong to a separate work
item.

That work must preserve the exact B-prime scope and must not broaden `fail` into
payload, subtype-factory, rethrow, assertion, or syntax behavior without a new
decision.
