# AUD009-B6 — Null, identity, equality, hashing, and recognition complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#612`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `82cc94664be79a3aac121b0babb70cf82b1c4284`

Closure revalidation revision: `8752b03600c11edb893b907f00a0dec65f2611b1`

The only repository delta between the evidence baseline and closure revalidation
revision is root governance work (`AGENTS.md`); no B6 semantic/runtime owner
changed.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#612`, issue comment
`5738900343`, 2026-09-19.

Derived semantic decision route:
`D154 / guillermomolina/protos#613 — Public semantic identity-hash surface`.

## Purpose and boundary

AUD009-B6 audited the shared Core policy for canonical null/absence, semantic
identity, ordinary equality, hashing/identity hashing, and exact standard-family
recognition.

It intentionally precedes later family-specific slices so Number, String/Bytes,
Array and Map do not each re-decide the same cross-cutting identity/equality/hash
institutions.

B6 does not itself alter normative specification or executable behavior.

## Final classification ledger

```text
canonical null singleton                         KEEP
present-null vs structural absence               KEEP
failed lookup remains Error                      KEEP
Object.ifNull                                    KEEP
Object.ifNotNull                                 KEEP

primitive semantic identity ===                  KEEP
closed Core value-identity set                   KEEP
!== complement                                   KEEP

customizable ==                                  KEEP
default Object == -> ===                         KEEP
strict Boolean equality result                   KEEP
derived !=                                       KEEP

customizable hash()                              KEEP
default Object hash -> identityHashOf             KEEP
hash equality-coherence contract                 KEEP
any semantic Integer-family hash result          KEEP

primitive identityHashOf                         KEEP
execution-scoped identity-hash contract          KEEP
public Object.identityHash()                     REMOVE_NOW_RECONSIDER_LATER

String.recognizes                                KEEP
Integer.recognizes                               KEEP
Float.recognizes                                 KEEP
Array.recognizes                                 KEEP
exact-owner/no-candidate-dispatch recognition    KEEP
no generic type/family query                     KEEP current boundary

undefined                                        ABSENT / RETAIN ABSENCE
Optional/Maybe Core family                       ABSENT / RETAIN ABSENCE
```

## Canonical null and absence

Core retains exactly one canonical `null` and no `undefined`.

A slot containing null remains present; a failed lookup remains Error. A Map
entry containing null remains distinct from an absent key. This keeps absence
explicit without adding a second sentinel or converting lookup failure into
value-level absence.

Classification: **KEEP**.

## Null-aware Object control

D144/I048's `Object.ifNull(block)` and `Object.ifNotNull(block)` remain
justified.

Post-I048 production source still contains many explicit `=== null` checks and
currently no direct production call of the two new selectors, but I048
deliberately avoided mechanical migration because many null tests represent EOF,
parser state, validation, bootstrap state or other domain distinctions.

The selected protocol itself remains low-cost ordinary source-backed composition
over `===`, Boolean control and ordinary polymorphic invocation. It adds no
grammar, Optional/Maybe family, truthiness, hidden state or scheduler behavior.

Classification: **KEEP**.

Confidence: **MEDIUM-HIGH**.

## Semantic identity

Primitive `===` remains the non-overridable identity relation. Ordinary
identity-bearing objects use individual semantic object identity; the closed
Core value-identity set consists of Number-family values, String values,
canonical Booleans and canonical null.

Freezing, structural equality, interning, delegation or host representation do
not opt a new category into value identity.

`!==` remains the non-overridable Boolean complement.

Classification: **KEEP**.

## Ordinary semantic equality

`==` remains the single ordinary customizable semantic-equality authority.

Default Object equality is:

```text
Object.==(other) -> this === other
```

Custom equality is demonstrably useful for normal Map logical keys while
IdentityMap simultaneously preserves exact object identity.

Equality results remain canonical Boolean or Error.

D148's derived `!=` remains the complement of exactly one selected/validated
`==` result and does not dispatch an independent `!=` selector.

Classification: **KEEP**.

## Ordinary hashing

Normal Map requires ordinary customizable `hash()` as the coherence companion
to customizable `==`:

```text
a == b  =>  a.hash == b.hash
```

Default Object hash remains primitive semantic identity hashing, which naturally
matches the default identity-based equality relation.

A valid hash result may be any semantic Integer-family value by exact
mathematical Integer value, including fixed-width Integer families. Narrowing
that contract to ordinary unbounded Integer would add an arbitrary conversion
requirement without reducing the underlying numeric model.

Classification: **KEEP**.

## Primitive identity hashing

The non-overridable semantic operation conceptually named
`identityHashOf(value)` remains required.

It is the hash companion to `===` and the authority used by IdentityMap and
other identity-sensitive runtime machinery.

Required coherence is:

```text
a === b  => identityHashOf(a) == identityHashOf(b)
```

Collisions remain permitted.

The observable hash domain is one Protos execution. These values are not
persistent IDs, distributed object identifiers, serialization identities or
cryptographic fingerprints.

Classification: **KEEP**.

## Public ordinary Object.identityHash()

AUD009-B6 found this guest-visible selector to be the only scoped mechanism whose
ongoing public cost is not justified by current capability use.

Current standard behavior merely exposes:

```text
value.identityHash() -> identityHashOf(value)
```

but ordinary lookup means the selector itself is overrideable.

The actual identity authority is not this message:

- IdentityMap uses primitive semantic identity hashing directly;
- default Object `hash()` uses primitive semantic identity hashing directly;
- ActorRef/GroupRef and transfer/rematerialization machinery use primitive/runtime
  identity authority;
- no production guest Protos caller of `.identityHash()` was found;
- direct guest use found by B6 is conformance-only.

This creates two similarly named concepts with different authority:

```text
identityHashOf(value)    non-overridable semantic authority
value.identityHash()     ordinary overrideable message
```

The latter is therefore classified:

**REMOVE_NOW_RECONSIDER_LATER**.

Removal does not include `===`, `!==`, IdentityMap, primitive
`identityHashOf`, default Object `hash()`, or any runtime/capability identity
machinery.

### Reconsideration trigger

Reconsider guest identity-hash access if real Standard Library, Tool or
application code repeatedly needs a stable numeric hash companion to semantic
identity outside IdentityMap itself and the need cannot be expressed adequately
with IdentityMap, `===`, or ordinary identity-bearing keys.

### Reconsideration scope

Re-evaluate the minimal guest API from the then-current identity model. Compare
a non-overridable operation/helper, ordinary message, library facility, or other
minimal mechanism. Do not assume the removed overrideable
`Object.identityHash()` spelling/authority should return unchanged.

### Required route

The owner-approved audit classification is routed to:

```text
D154 / #613 — Public semantic identity-hash surface
STATUS=NEEDS_USER_DECISION
```

D154 must independently complete the current Dxxx comparative research,
scoring, falsification and exact-candidate approval process before any normative
or implementation removal occurs.

## Exact standard-family recognition

D147/I045's evidence-scoped owner-side recognizers remain justified:

```text
String.recognizes(value)
Integer.recognizes(value)
Float.recognizes(value)
Array.recognizes(value)
```

B6 found production uses in 18 Protos files across Standard Library and bundled
tools.

The predicates replaced indirect family-validation probes and, for Array,
incorrect immediate-parent approximations.

Recognition remains exact-owner, Boolean-valued and non-dispatching on the
candidate. Delegation, equality/hash behavior and user overrides do not confer
Core semantic-family membership.

No generic `typeOf`, `is`, family descriptor, universal recognizer or symmetric
recognizer catalog is added.

Classification: **KEEP**.

## Strongest attempted removals

- Removing null-aware control would restore the repeated explicit-null-control
  friction D144 already established while saving only two source-backed Object
  methods.
- Removing value identity would expose allocation/boxing or require another
  equality-like identity mechanism.
- Removing customizable `==`/`hash` would destroy user-defined normal Map key
  semantics or require a separate comparator/hasher institution.
- Restricting hash results to ordinary unbounded Integer saves no meaningful
  institution and forces needless conversion.
- Removing `recognizes` would restore discarded-operation validation probes and
  incorrect Array parent tests.
- Removing public `Object.identityHash()` loses no currently demonstrated Core
  capability and has low reintroduction cost.

Only the last removal survives the retrospective standard.

## Reconciliation boundaries

- A2 equality/identity operator surface remains unchanged.
- B1 ordinary object identity/delegation remains unchanged.
- B5 strict Boolean/no-truthiness outcomes remain unchanged.
- D144/I048 null-aware semantics remain unchanged.
- D147/I045 exact-family recognition remains unchanged.
- D148/I043 derived inequality remains unchanged.
- detailed Number identity/equality/hash behavior remains for B7.
- String/Bytes family behavior remains a later B slice.
- Array/Map/IdentityMap detailed collection behavior remains later B slices.
- Actor/P/Process identity rematerialization remains AUD009-C.
- runtime identity-hash representation/caching remains AUD009-G.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#612
APPROVAL_COMMENT=5738900343
DATE=2026-09-19

IDENTITY_HASH_PUBLIC_SELECTOR_CLASSIFICATION=REMOVE_NOW_RECONSIDER_LATER
DERIVED_DECISION=D154/#613
DERIVED_DECISION_STATE=NEEDS_USER_DECISION
NORMATIVE_CHANGE_AUTHORIZED_BY_B6=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_B6=NO
```

## Closure checklist

```text
NULL_MODEL=KEEP
NULL_AWARE_CONTROL=KEEP
SEMANTIC_IDENTITY=KEEP
VALUE_IDENTITY_SET=KEEP
ORDINARY_EQUALITY=KEEP
DERIVED_INEQUALITY=KEEP
ORDINARY_HASH=KEEP
IDENTITY_HASH_PRIMITIVE=KEEP
PUBLIC_OBJECT_IDENTITY_HASH=REMOVE_NOW_RECONSIDER_LATER
EXACT_FAMILY_RECOGNITION=KEEP

EVERY_SCOPED_EXISTING_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTE=D154/#613
REMOVAL_IMPLEMENTED_BY_AUDIT=NO
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B6_CLASSIFICATION=COMPLETE
```

AUD009-B6 is complete once this durable record and the required live GitHub
closure postconditions are verified.
