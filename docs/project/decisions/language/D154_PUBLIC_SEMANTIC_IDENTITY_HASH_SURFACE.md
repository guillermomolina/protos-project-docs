# D154 — Public semantic identity-hash surface

Status: **RATIFIED — Candidate A (remove ordinary `Object.identityHash()`)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#613`
Trigger: AUD009-B6 / `guillermomolina/protos#612`
Protos evidence revision: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`
Project-record base: `cd69f89c7b93d3da98599190461f38ca5ffeb596`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D154 selects **Candidate A**.

Core v0.1 removes the ordinary guest-visible convenience selector:

```protos
object.identityHash()
```

The removal is deliberately limited to that ordinary overridable message.
Semantic identity hashing itself remains part of Core/runtime semantics.

The retained model is:

```text
Object.identityHash()                     REMOVE
identityHashOf(value)                     KEEP
===                                       KEEP
!==                                       KEEP
IdentityMap identity semantics            KEEP
Object.hash() default                     KEEP
runtime identity-hash machinery           KEEP
ActorRef / GroupRef semantic identity     KEEP
```

No replacement guest-visible numeric identity-hash API is selected by D154.

## Approval provenance

After reviewing the complete D154 comparative research, candidate set, scoring,
falsification, future-scenario analysis, incremental-design analysis, and exact
invariant/delta check, the project owner explicitly approved:

```text
aprobar D154 Candidate A
```

```text
SELECTED_CANDIDATE=A
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Preserved semantic authority

D154 does not remove or weaken the primitive semantic operation conceptually
written as:

```text
identityHashOf(value)
```

That operation remains non-overridable and remains the identity-hash companion
to primitive semantic identity `===`.

Identity-sensitive machinery continues to use primitive authority rather than
ordinary message dispatch. In particular, `IdentityMap` continues to use
`identityHashOf(key)` together with `===` and must not dispatch a user-defined
`identityHash` message.

The standard default `Object.hash()` also continues to use primitive semantic
identity hashing where that default applies. User-defined ordinary `hash`
behavior remains independently customizable under the existing Map contract.

## GITHUB021 invariant consistency

The exact selected candidate preserves every applicable owner-approved
AUD009-B6 invariant:

```text
SEMANTIC_IDENTITY_===                         PRESERVED
SEMANTIC_NONIDENTITY_!==                     PRESERVED
IDENTITY_MAP_IDENTITY_RELATION               PRESERVED
IDENTITY_MAP_USES_NONOVERRIDABLE_AUTHORITY   PRESERVED
PRIMITIVE_IDENTITY_HASH_AUTHORITY            PRESERVED
IDENTITY_HASH_COLLISIONS_ALLOWED             PRESERVED
EXECUTION_SCOPED_IDENTITY_HASH_DOMAIN        PRESERVED
DEFAULT_OBJECT_HASH                          PRESERVED
CUSTOMIZABLE_ORDINARY_HASH                   PRESERVED
ACTORREF_GROUPREF_IDENTITY                   PRESERVED
```

The only approved observable delta is:

```text
STANDARD_GUEST_OBJECT_IDENTITYHASH_SELECTOR=REMOVE
```

Removing the selector does not alter primitive identity-hash numeric values,
stability rules, collision rules, execution scope, rematerialization semantics,
or identity categories.

## Rationale

Repository evidence at the decision revision showed no production guest Protos
caller of `.identityHash()`. The only direct guest call was conformance coverage
for the existing selector.

The facilities that actually require semantic identity hashing do not use the
ordinary selector:

- `IdentityMap` calls the primitive runtime identity-hash authority directly;
- default `Object.hash()` calls primitive identity hashing directly;
- ActorRef and GroupRef identity/rematerialization machinery uses primitive or
  runtime identity authority;
- distributed identity semantics require the primitive `identityHashOf`
  contract, not message dispatch.

Therefore the ordinary selector adds public surface without supplying authority
needed by any current production consumer.

The current selector is also a poor semantic authority boundary: because it is
an ordinary Protos message, user code may shadow or override it, while
`IdentityMap` and `===` deliberately ignore that override. A spelling that looks
like the semantic identity-hash authority can therefore return an unrelated
number during explicit message dispatch.

## Comparative evidence

The research considered materially different approaches across Smalltalk,
Java, JavaScript, Python, Ruby, .NET, and Self-style prototype-oriented design.

The relevant distinction was authority rather than spelling:

- systems that expose a true identity-hash operation commonly distinguish it
  from ordinary overridable hashing;
- systems can provide identity comparison and identity-keyed collections
  without exposing a numeric identity hash to application code;
- object-ID facilities such as Python `id()` or Ruby `object_id` are a stronger
  institution than the collision-permitting execution-scoped hash considered
  by D154 and are not selected here.

The evidence therefore did not justify retaining the current overridable
selector merely because numeric identity-hash APIs exist elsewhere.

## Rejected candidates

### Candidate B — keep the ordinary overridable selector

Rejected for the current language because no production need justifies the
surface and because ordinary overriding makes it an ambiguous representation of
the non-overridable semantic identity-hash authority.

### Candidate C — expose a non-overridable guest authority

Rejected for now because it would correctly align authority with `===` and
`IdentityMap` but would still expose a numeric capability for which current
Standard Library, Tool, and application code provides no demonstrated need.

### Public identity token / object ID

Not selected. A public identity token or object ID would be a materially stronger
contract than a collision-permitting identity hash and would reopen persistent,
distributed, debugging, serialization, and interoperability questions outside
D154.

## Incremental-design result

Candidate A is the smallest solution satisfying current requirements.

Removing the guest selector does not discard difficult implementation machinery:
primitive identity hashing must remain implemented for `IdentityMap`, default
hash behavior, semantic identity, and distributed/rematerialized identity.

If later code genuinely requires the numeric companion itself, public access can
be added without redesigning the underlying identity model. In contrast,
retaining an overridable public selector now would create compatibility cost if
it later needed to become non-overridable or be replaced.

## Reconsideration trigger

D154 preserves the AUD009-B6 reconsideration trigger:

```text
Real Standard Library, Tool, or application code repeatedly needs a stable
numeric hash companion to semantic identity outside IdentityMap itself, and the
requirement cannot be expressed adequately with IdentityMap / === / ordinary
identity-bearing keys.
```

A future reconsideration must compare the then-current minimal public API. It
must not assume that the removed overridable `Object.identityHash()` message
should be restored unchanged.

## Explicit non-goals and deferred questions

D154 does not redesign:

- `===` or `!==`;
- ordinary `==` or `!=`;
- normal `hash()`;
- Map or IdentityMap lookup semantics;
- semantic identity categories;
- ActorRef or GroupRef identity;
- persistent or distributed object IDs;
- cryptographic hashing;
- serialization;
- host/runtime identity-hash storage strategy.

A future public `identityHashOf(value)` operation, identity namespace, object-ID
facility, identity-set abstraction, or debugging identity token remains deferred
until separately justified.

## Normative and implementation routing

Candidate A changes observable Core surface, so ratification alone does not
modify the language implementation or normative specification.

Follow-up reconciliation must remove the ordinary public selector consistently
from the applicable normative specification, runtime specification, bootstrap
surface, implementation, documentation, and conformance coverage while
preserving primitive identity hashing and every retained invariant above.

That reconciliation must not opportunistically redesign identity hashing or add
a replacement guest API.

```text
D154_STATUS=RATIFIED
SELECTED_CANDIDATE=A
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
