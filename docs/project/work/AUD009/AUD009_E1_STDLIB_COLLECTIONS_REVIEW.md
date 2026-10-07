# AUD009-E1 — Standard Library collections complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#646`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`1b4065f0f79a5c2837b4462f00b3b8e481d11b98`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#646`, issue comment
`5739814004`.

Owner approval provenance: `guillermomolina/protos#646`, issue comment
`5739820536`, 2026-09-19.

Derived decision routes: **NONE**

## Purpose

AUD009-E1 reviewed the current importable Standard Library collection modules
layered over already-retained Core Array/Map/IdentityMap semantics:

```text
std:collections/Array
std:collections/Set
std:collections/IdentitySet
std:collections/Range
```

The review asked whether the shipped library surface still earns its maintenance
and semantic cost, whether it duplicates Core unnecessarily, and whether a more
general collection hierarchy would be justified.

AUD009-E1 itself authorizes no normative or implementation change.

## Current implementation shape

The inspected modules are ordinary Protos source only:

```text
protos/lib/collections/Array.protos
protos/lib/collections/Set.protos
protos/lib/collections/IdentitySet.protos
protos/lib/collections/Range.protos
```

Together they are approximately 14 KB of source.

They introduce no:

- native collection value family;
- runtime tag;
- host backend;
- generic iterator;
- lazy pipeline;
- wrapper identity;
- privileged Actor transfer rule;
- shared mutable service.

This low institutional cost is central to the approved KEEP classification.

## Production consumers

### Array

Current bundled-tool consumers found at the evidence baseline include:

```text
Arrays.map
    protos/tools/test/LogicalCasePlan.protos
    protos/tools/test/LogicalCaseRunner.protos
    protos/tools/test/Discovery.protos
    protos/tools/test/LogicalCaseResult.protos

Arrays.findIndex
    protos/tools/test/Discovery.protos

Arrays.sort
    protos/tools/package/LockDocument.protos
    protos/tools/package/ProjectDocument.protos
    protos/tools/package/ResolutionInput.protos
    protos/tools/package/ContentIdentity.protos
```

Package Tool uses sorting for deterministic/canonical ordering; Test Tool uses
map/findIndex in discovery/planning/execution.

No production consumer was found for `filter` or `reduce` at the baseline,
but both remain ordinary local algorithms over retained Core Array and add no
new runtime institution.

### Set

Current Test Tool production use includes:

```text
protos/tools/test/Discovery.protos

Sets()
Sets.contains(...)
Sets.add(...)
```

The module reuses ordinary Map state and behavior.

No production consumer was found for Set algebra/predicate helpers at the
baseline, but those operations are the natural reusable set-specific capability
of the module and do not introduce additional representation or runtime rules.

### IdentitySet

No production consumer was found at the baseline.

IdentitySet remains a small ordinary role over IdentityMap, preserving semantic
identity membership explicitly rather than introducing a configurable equality
strategy.

### Range

No current production source imported Range at the baseline.

However, LIB017 was created from repeated range-shaped loops already found in
production CLI/URI/Test Tool/Package Tool code. Its owner-approved G-prime
implementation deliberately reduced that need to only:

```text
Range.each(start, stop, block)
Range.reverseEach(start, stop, block)
```

with no first-class Range value, iterator hierarchy, arbitrary step, slicing or
syntax.

Zero adoption immediately after that bounded implementation is therefore not
sufficient removal evidence.

## Final classification ledger

```text
STDLIB_ARRAY_MODULE=KEEP
ARRAY_MAP=KEEP
ARRAY_FILTER=KEEP
ARRAY_FIND_INDEX=KEEP
ARRAY_REDUCE=KEEP
ARRAY_SORT=KEEP

STDLIB_SET_MODULE=KEEP
SET_MAP_BACKING=KEEP
SET_BASIC_OPERATIONS=KEEP
SET_ALGEBRA=KEEP
SET_RELATION_PREDICATES=KEEP

STDLIB_IDENTITY_SET_MODULE=KEEP
IDENTITY_SET_IDENTITY_MEMBERSHIP=KEEP

STDLIB_RANGE_MODULE=KEEP
RANGE_EACH=KEEP
RANGE_REVERSE_EACH=KEEP
RANGE_MINIMAL_HALF_OPEN_UNIT_MODEL=KEEP
```

No E1 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## Array module remains

The approved Array library surface is:

```text
map
filter
findIndex
reduce
sort
```

The current architecture remains preferable to moving these operations into
Core:

- Core Array stays small;
- applications pay only when importing the module;
- algorithms remain ordinary Protos;
- callback/effect/snapshot rules remain explicit and portable;
- Test Tool and Package Tool already consume the module.

Classification: **KEEP**.

## Set remains an ordinary Map-backed role

The approved representation remains:

```text
Set = standard Map with member -> true associations
```

The module retains construction, membership, mutation, iteration, algebra and
relation predicates.

This design reuses Map equality/hash, insertion order, lifecycle and Actor
transfer rather than inventing a Set runtime family.

Classification: **KEEP**.

## IdentitySet remains distinct

The approved representation remains:

```text
IdentitySet = standard IdentityMap with member -> true associations
```

The semantic distinction from Set is real: membership follows semantic identity
rather than Map hash + equality.

Merging both modules through a configurable equality strategy would introduce a
larger abstraction than the small source duplication it removes.

Classification: **KEEP**.

## Range remains deliberately minimal

The approved Range surface remains:

```text
each(start, stop, block)
reverseEach(start, stop, block)
```

with:

```text
half-open [start, stop)
ordinary unbounded Integer bounds
unit stepping
bounds evaluated once
no proportional materialization
```

The module is the already-selected anti-overengineered response to demonstrated
production iteration pressure.

Classification: **KEEP**.

## Deliberate absences remain absent

```text
generic Collection hierarchy       ABSENT / RETAIN ABSENCE
Iterable / Iterator                ABSENT / RETAIN ABSENCE
Sequence / lazy Stream             ABSENT / RETAIN ABSENCE
backed collection Views            ABSENT / RETAIN ABSENCE
Set runtime family/prototype       ABSENT / RETAIN ABSENCE
equality-strategy Set              ABSENT / RETAIN ABSENCE
growable Core Array                ABSENT / RETAIN ABSENCE

first-class Range value            ABSENT / RETAIN ABSENCE
arbitrary Range step               ABSENT / RETAIN ABSENCE
Range membership/indexing/size     ABSENT / RETAIN ABSENCE
inclusive/generic ranges           ABSENT / RETAIN ABSENCE
Range slicing integration          ABSENT / RETAIN ABSENCE
Range syntax                       ABSENT / RETAIN ABSENCE
```

E1 found no evidence that a universal collection/iteration institution would
reduce total complexity relative to the current module-oriented design.

## Strongest attempted removals

```text
remove std:collections/Array and use explicit loops
    rejected -> current Test Tool and Package Tool production consumers

remove Array.filter/reduce because they currently lack production consumers
    rejected -> distinct low-cost ordinary algorithms; no privileged burden

replace Set with direct Map convention everywhere
    rejected -> repeats the key->true role and loses reusable set algebra

trim Set algebra/predicates because current tools do not use them
    rejected -> tiny ordinary composition; trimming creates arbitrary asymmetry

remove IdentitySet
    rejected -> preserves a real existing IdentityMap membership law

parameterize Set with equality strategy
    rejected -> more general and semantically heavier than separate modules

remove Range because no current production import exists
    rejected -> demonstrated production need predated implementation and G-prime
                is already the minimal bounded form

broaden Range into first-class value/Iterable
    rejected -> larger institution without present requirement
```

## Required AUD009 routing

No removal/redesign route is required.

```text
REMOVAL_ROUTES=NONE
DERIVED_DXXX=NONE
DERIVED_LIBXXX=NONE
```

Future additions remain ordinary separately justified work and are not implied
by this KEEP classification.

## Boundary handoffs

- Core Array/Map/IdentityMap semantics remain owned by their existing Core
  authorities and AUD009-B9.
- Future new collection abstractions require their own demonstrated need.
- Runtime/backend collection representation remains AUD009-G territory.

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=1b4065f0f79a5c2837b4462f00b3b8e481d11b98

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_E1_CLASSIFICATION=COMPLETE
```

AUD009-E1 is complete.
