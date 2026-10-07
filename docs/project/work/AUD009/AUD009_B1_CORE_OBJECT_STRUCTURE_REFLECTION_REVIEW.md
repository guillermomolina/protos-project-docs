# AUD009-B1 — Core object structure and reflection complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#601`

Parent audit: `guillermomolina/protos#522` — AUD009

Initial evidence baseline: `e27b0e69808df10fa608595ffffff6fc80221124`

Closure revalidation revision: `98997d173ef482210dd726a900ec75d23f02ce7b`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#601`, issue comment
`5738526686`, 2026-09-19.

## Purpose and boundary

AUD009-B1 reviews the existing Core object-structure, structural-state, and
local-reflection model under the retrospective complexity/necessity methodology
ratified for AUD009.

The question is not whether each mechanism has many present application call
sites. B1 asks whether the continuing public and internal complexity is justified
by a fundamental object-model capability, real current integration, or concrete
reintroduction cost.

B1 does not redesign syntax already classified by AUD009-A2, does not reopen
D140 horizontal composition, does not invent missing ergonomics owned by
AUD011, and does not audit Closure/control, value-family breadth, concurrency,
I/O lifecycle, Standard Library breadth, tooling architecture, or AST/Bytecode
architecture.

Between the initial evidence baseline and closure revalidation revision, the
only product-repository change was to root `AGENTS.md`; none of the B1 semantic
or implementation owners changed.

## Final classification ledger

```text
unique root Object                                  KEEP
exactly one immutable delegation parent otherwise  KEEP
delegated reads                                     KEEP
local-only structural writes                        KEEP
writes never mutate ancestors                       KEEP
root parent absence; no manufactured sentinel       KEEP

hasSlot(name)                                       KEEP
slotValue(name)                                     KEEP
slotNames()                                         KEEP
parent()                                            KEEP
semantic-String reflection names                    KEEP
local-only reflection boundary                      KEEP

removeSlot(name)                                    KEEP

freeze() / FROZEN                                   KEEP
close() / CLOSED structural capability              KEEP
open -> closed -> frozen integrity ladder           KEEP
```

No B1 mechanism is classified for removal. No Dxxx removal route is required.

## Delegation and local structure

Core retains one unique structural root, `Object`, with no parent. Every other
ordinary object has exactly one immutable delegation parent. Reads may delegate;
ordinary structural mutation remains local.

This is not an organizational convenience layered over another object model. It
is the object model. Lookup, `super`, reflection, standard prototype topology,
construction, and receiver behavior all depend on a single-parent chain
terminating at the unique root.

The parent edge is immutable independently of OPEN/CLOSED/FROZEN state. Ordinary
writes do not mutate an ancestor merely because lookup would find a binding
there. This keeps mutation authority local and prevents a child write from
silently changing shared prototype state.

`Object.parent()` signaling at the root remains correct. Returning `null` or
another sentinel would manufacture a language value for an edge that does not
exist and would make reflection describe a different topology.

Classification: **KEEP**.

Confidence: **HIGH**.

## Core local reflection

The retained ordinary-message reflection surface is:

```text
hasSlot(name)
slotValue(name)
slotNames()
parent()
```

Reflection names are semantic Strings. `hasSlot`, `slotValue`, and `slotNames`
observe receiver-local structure rather than delegated lookup. `parent()`
observes the immutable delegation edge.

### `hasSlot(name)` and `slotValue(name)`

These have direct production library/tool consumers and express the query/read
half of local structural reflection. Delegated lookup is deliberately not a
substitute: generic code must be able to distinguish own structure from
inherited behavior.

Classification: **KEEP**.

### `slotNames()`

Current production guest use is smaller, but removing enumeration would leave
reflection able to ask only about names already known by the caller. Generic
local-structure reflection requires enumeration.

D032 remains authoritative for the result contract: each call returns a fresh
independent standard Array snapshot in deterministic Unicode-scalar
lexicographic order. That cost is call-local; it does not impose ordered storage
or ordinary-lookup overhead.

Classification: **KEEP**.

### `parent()`

This is the direct reflective projection of the delegation topology and has real
Tool use. Removing it would make a fundamental object relation observable by
dispatch but unavailable to ordinary reflection.

Classification: **KEEP**.

### Name domain and locality

D033's semantic-String name domain and the local-only boundary remain retained.
They avoid host selector IDs, coercion, and accidental delegated mutation or
observation.

Classification: **KEEP**.

Confidence for the reflection surface: **HIGH**.

## `removeSlot(name)`

The initial B1 pass considered removal because no non-test guest Protos caller was
found. That classification was rejected after reviewing the capability as part
of the object model rather than as a usage-frequency convenience.

For an open prototype object, the structural algebra is:

```text
create local slot
inspect local slot
modify existing local slot
remove local slot
```

Without `removeSlot`, local shape can grow but cannot contract while preserving
the same object identity. A local slot that shadows an inherited binding could
never be removed to reveal the delegated binding again.

For example, conceptually:

```protos
animal: { alive: true }
dog: animal { alive: false }

dog.removeSlot("alive")
dog.alive
```

The final read observes the inherited `animal.alive` binding. A fresh
`without(...)` composition view is not semantically equivalent because it
changes identity and parent/topology consequences.

The internal runtime also has legitimate unpublished local-slot removal needs,
but those internal uses are not the primary justification for the public
selector. The justification is completeness of the open object's same-identity
structural mutation model.

`removeSlot` also gives CLOSED its precise structural meaning: once closed,
neither creation nor deletion can change shape, while existing state may still
be replaced.

Classification: **KEEP**.

Confidence: **HIGH**.

## OPEN / CLOSED / FROZEN integrity model

B1 performed a second, deeper review after the project owner challenged the
initial proposal to remove CLOSED. That review recovered the distinction between
isolation safety and structural integrity.

The retained monotonic model is:

```text
OPEN
  local structure may grow or shrink
  existing state may change

CLOSED
  local structure is fixed
  existing state may still change

FROZEN
  local structure is fixed
  existing state may not change
```

Allowed transitions are conceptually:

```text
OPEN -> CLOSED -> FROZEN
OPEN -----------> FROZEN
```

There is no reopen or thaw operation.

### `freeze()` / FROZEN

FROZEN has direct production use and a mandatory isolation/publication role.

D049/B010 and the current object/module/concurrency specifications require
physically shared standard objects whose structural state is observable to be
FROZEN before guest observation. The shared root `Object` may not be published
merely OPEN or CLOSED.

This is a shallow integrity state, not deep transitive freezing. It therefore
does not install graph traversal, ownership, or recursive shareability machinery
on ordinary objects.

Classification: **KEEP**.

Confidence: **HIGH**.

### `close()` / CLOSED

CLOSED is not an Actor-transfer admission state and is not sufficient for
physical sharing. Its role is different: it is a same-identity
fixed-shape-but-mutable integrity state.

That distinction applies uniformly beyond ordinary slots.

#### Ordinary objects

A CLOSED object rejects local slot creation and removal while preserving
assignment to already-existing writable slots.

#### Array

CLOSED permits replacement of an existing indexed element but does not grant
new structural Array operations. FROZEN rejects replacement.

#### Bytes

CLOSED permits `atPut` at an existing index while rejecting length-changing
`add` and `removeAt`. FROZEN rejects `atPut` as well.

#### Map / IdentityMap

CLOSED permits replacement of the mapped value for an already-present key while
rejecting insertion and removal. FROZEN rejects replacement as well.

The resulting cross-Core rule is small and regular:

```text
structural/cardinality mutation  -> requires OPEN
replacement of existing state    -> allowed through CLOSED
all mutation                     -> rejected by FROZEN
```

This is stronger evidence than application call counts because it demonstrates
one general mutation-permission concept reused by multiple Core semantic
families.

Classification: **KEEP**.

Confidence: **HIGH**.

## Actor, P, detached-snapshot, and Closure-binding reconciliation

Actor message transfer and isolated parallel execution use logical copy/snapshot
semantics for ordinary mutable values. CLOSED is not the mechanism that makes
those boundaries safe.

The safety distinction is:

```text
FROZEN
  required when a standard Protos object is physically shared across an
  isolation boundary

CLOSED
  observable mutation permission that must be preserved when a logical object
  is copied
```

Current implementation preserves CLOSED/FROZEN state through:

- `ProtosActorValueTransfer`;
- `ProtosParallelRuntime`;
- `ProtosDetachedExecutionValue`; and
- receiver-bound Closure materialization in `ProtosClosureValue`.

Preserving CLOSED on a logical copy is semantically natural: silently copying a
CLOSED source into an OPEN destination would grant structural mutation authority
that the source value did not possess.

This integration is a consequence of retaining CLOSED, not evidence that CLOSED
is itself an Actor security primitive.

## Cost analysis

### Public cognitive surface

The programmer must understand three structural integrity levels instead of two.
That is real cost.

The benefit is correspondingly general: the middle level expresses the common
invariant "shape/cardinality fixed, existing values mutable" on ordinary
objects and mutable Core collections.

The model is monotonic and does not expose reopen/thaw, deep freeze, ownership
transfer, or multiple partially ordered integrity flags.

### Runtime and memory

The runtime already needs per-object mutation state for FROZEN. CLOSED adds a
third enum value and state-sensitive branches; removing it would not eliminate
the state field or mutation checks wholesale.

### Maintenance and interaction cost

CLOSED must be preserved across copy/materialization boundaries and observed by
collection mutation protocols. This is continuing maintenance cost, but the
same small rule explains each integration.

### Reintroduction cost

If removed now and later required, restoring fixed-shape-but-mutable identity
would require more than adding one library helper. It would touch:

- ordinary slot mutation;
- Object protocol publication;
- Array/Bytes/Map/IdentityMap mutation boundaries;
- Actor/P/detached copy fidelity;
- Closure binding/materialization;
- conformance; and
- documentation/specification.

That makes removal/reintroduction materially less attractive than retaining the
small existing state.

## Structural `close` naming debt

I031-C exposed BUG004 when inherited structural `Object.close()` collided with
the I/O-domain `Closable.close()` selector. Generic I/O capability probes that
treated any callable `close` as lifecycle authority began finding root
`Object.close()` on ordinary objects.

Production code still contains the resulting distinction in TextReader,
TextWriter, and BufferedByteIo capability recognition: root structural `close`
must not masquerade as I/O lifecycle `close`.

This is genuine ongoing complexity.

B1 nevertheless separates the semantic capability from the selector spelling:

```text
fixed-shape-mutable CLOSED capability     KEEP
current structural close() capability    KEEP for B1
whether "close" is the best spelling     NOT DECIDED HERE
```

A `seal`-like spelling would describe the structural operation more directly and
has strong prior art, but renaming the selector would be a new observable
language decision. AUD009-B1 does not manufacture that decision from a naming
observation.

The naming collision is therefore retained as explicit non-blocking debt, not
as a removal route and not as an AUD011 missing-capability proposal.

## Comparative evidence

B1 used materially different object models as supporting evidence, not as
authority.

JavaScript provides the closest direct analogue: `Object.seal()` fixes the
property set while still permitting writes to writable existing properties;
`Object.freeze()` adds the stronger restriction on existing data-property
writes.

Python's `__slots__` demonstrates a different route to fixed instance shape:
shape is constrained by class/type construction rather than by a per-object
runtime sealing transition, while existing slot values remain mutable.

Ruby's ordinary object model provides the stronger freeze operation without an
equivalent general middle integrity transition.

Prototype-oriented Self and Io make object slot structure a first-class concern,
supporting the broader conclusion that structural mutation is not incidental in
a prototype model.

Lua tables remain dynamically extensible and use metatable mechanisms rather
than an equivalent integrity-state transition; this demonstrates that the
capability is not universal, but does not provide an equivalent same-identity
seal.

The comparison supports treating CLOSED as a coherent design point rather than
an implementation accident.

## D140 reconciliation

D140's retained horizontal composition model is unchanged.

`without(name)` and `alias(sourceName, aliasName)` are non-mutating constructors
of fresh ordinary structural views. They do not replace `removeSlot` because
they do not mutate the same object identity.

Composition into a receiver remains subject to ordinary structural state:
CLOSED/FROZEN restrictions continue to follow the object model.

No D140 invariant is reopened by B1.

## AUD009-A2 and AUD011 boundaries

B1 does not reopen syntax/operator outcomes from A2.

The possible future question of a better spelling for structural sealing is not
silently converted into a new language feature by this audit. If the project
later chooses to investigate `close` versus `seal` or another spelling, that
must enter the normal design route with its own approval boundary.

Missing reflection facilities or richer metaobject protocols remain outside B1
and belong to AUD011 or another explicitly routed owner if real need appears.

## Strongest attempted removal cases and why they failed

### `removeSlot`

Removal looked attractive from repository call counts, but failed the
fundamental-capability test: without it, an open object's local shape can expand
but not contract under the same identity, and delegated bindings hidden by a
local override cannot be revealed again without replacing the object.

### CLOSED

Removal looked attractive because structural `close()` has little direct
production guest use and its selector collides with I/O lifecycle `close`.

The deeper review rejected that argument because:

- the state expresses a coherent integrity level distinct from FROZEN;
- the same rule already spans objects and several Core collection families;
- alternative constructions change identity or assignment semantics;
- the runtime cost is incremental over already-required FROZEN state;
- reintroduction would cross several semantic boundaries; and
- BUG004 primarily demonstrates selector-name debt, not absence of value in the
  fixed-shape-mutable capability.

## Owner approval

After the deep CLOSED review, the project owner explicitly approved the proposal
that every B1-scoped semantic mechanism remain KEEP.

Approval provenance is the active AUD009-B1 coordination thread,
`guillermomolina/protos#601`, issue comment `5738526686`, dated 2026-09-19.

This approval is classification authority for AUD009-B1. It does not itself
change normative language semantics, because every B1 outcome retains the
already-existing behavior.

## Closure checklist

```text
SCOPED_SEMANTIC_INVENTORY=COMPLETE
DELEGATION_AND_LOCAL_STRUCTURE=KEEP
LOCAL_REFLECTION_SURFACE=KEEP
REMOVESLOT=KEEP
FREEZE_FROZEN=KEEP
CLOSE_CLOSED=KEEP
EVERY_SCOPED_MECHANISM_HAS_ONE_AUD009_CATEGORY=PASS
ACTOR_P_PROCESS_RECONCILIATION=PASS
ARRAY_BYTES_MAP_IDENTITYMAP_RECONCILIATION=PASS
D140_BOUNDARY=PASS
AUD009_A2_BOUNDARY=PASS
AUD011_BOUNDARY=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTES=NOT_REQUIRED
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B1_CLASSIFICATION=COMPLETE
```

AUD009-B1 is therefore ready for closure once its required durable publication
and live GitHub coordination postconditions are satisfied. The parent AUD009
audit should then advance to the next bounded Core-semantics slice rather than
continue exploring B1.
