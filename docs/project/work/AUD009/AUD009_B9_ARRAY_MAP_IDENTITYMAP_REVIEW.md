# AUD009-B9 — Array, Map, and IdentityMap complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#620`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`

Closure revalidation revision: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`

No Protos product-repository content changed between the B9 evidence baseline and
the closure revalidation revision.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#620`, issue comment
`5739225689`, 2026-09-19.

Checkpoint proposal: `guillermomolina/protos#620`, issue comment
`5739218137`.

Derived decision routes: **NONE**

## Purpose and boundary

AUD009-B9 reviewed the retained Core collection institutions after the earlier
B1/B6/B7/B8 slices:

- Array as the standard dense indexed mutable collection;
- Map as the standard equality/hash keyed collection;
- IdentityMap as the standard semantic-identity keyed collection;
- collection-owned iteration, ordering, mutation and receiver-domain rules.

B9 is an evidence/classification slice. It does not itself change normative
collection semantics, syntax, implementation or Standard Library behavior.

The review explicitly attempted to remove or relocate the most institution-heavy
parts of the current model. None of those removals survived the evidence.

B9 does not reopen:

- AUD009-A1 / D131 / I041 matching semantics;
- B1 structural open/closed/frozen semantics;
- B6 equality / semantic identity / hashing;
- D136 populated Map construction;
- D142 / I049 `Map.atIfAbsent`;
- D156 fixed-width Integer placement.

## Final classification ledger

```text
Array semantic family                              KEEP
fresh Array identity                               KEEP
ordinary polymorphic Array factory                 KEEP
receiver-owned dense indexed state                 KEEP
zero-based exact-Integer indexing                  KEEP
no holes / no negative indexing                    KEEP
Array.size                                         KEEP
Array.at                                           KEEP
Array.atPut                                        KEEP
open/closed/frozen indexed replacement boundary    KEEP
Array.each                                         KEEP
shallow snapshot iteration                         KEEP
ascending-index iteration order                    KEEP
Array.recognizes                                   KEEP
default identity-based Array == / hash              KEEP

Map semantic family                                KEEP
fresh Map identity                                 KEEP
ordinary polymorphic Map factory                   KEEP
receiver-owned keyed state                         KEEP
normal Map key law: hash + directed ==             KEEP
single query hash / recorded insertion hash        KEEP
representative stored-key retention                KEEP
insertion-order association semantics              KEEP
Map.size                                           KEEP
Map.at                                             KEEP
Map.atPut                                          KEEP
Map.containsKey                                    KEEP
Map.atIfAbsent                                     KEEP
Map.remove                                         KEEP
Map.each                                           KEEP
default identity-based Map == / hash               KEEP

IdentityMap semantic family                        KEEP
IdentityMap direct Object topology                 KEEP
identityHashOf + === key law                       KEEP
same keyed surface as Map where applicable         KEEP
IdentityMap insertion order                        KEEP
IdentityMap snapshot iteration                     KEEP
IdentityMap open/closed/frozen integration          KEEP

Map point-of-mutation state revalidation            KEEP
same-Map mutation restriction during key ==         KEEP
comparison restriction across suspension            KEEP
no blocking lock / no transaction                   KEEP

Core Array append/insert/remove/resize selectors    ABSENT / RETAIN ABSENCE
negative indexing                                   ABSENT / RETAIN ABSENCE
Array holes                                         ABSENT / RETAIN ABSENCE
numeric length/fill Array constructor               ABSENT / RETAIN ABSENCE
Map.recognizes                                      ABSENT / RETAIN ABSENCE
IdentityMap.recognizes                              ABSENT / RETAIN ABSENCE
generic Core Collection prototype                   ABSENT / RETAIN ABSENCE
Association Core value family                       ABSENT / RETAIN ABSENCE
```

No B9 mechanism is classified
`REMOVE_NOW_RECONSIDER_LATER` or `REMOVE_PERMANENTLY`.

## Array analysis

Array remains a small, heavily used Core sequence institution.

The normative model is a fresh identity-bearing object with receiver-owned
dense indexed state:

```text
0 .. size - 1
```

The standard factory is ordinary polymorphic invocation:

```protos
Array()
Array(a)
Array(a, b, c)
```

It has no numeric length/fill overload. `Array(3)` is one element containing
the exact Integer value `3`.

Standard indexed access accepts an exact semantic Integer mathematical value,
uses zero-based bounds, defines no holes and defines no negative indexing.

The retained primitive Core sequence surface is deliberately narrow:

```text
size
at
atPut
each
```

Construction plus those operations are sufficient for current Core needs.
Higher-order transformations such as map/filter/reduce/sort already live in
`std:collections/Array` as ordinary Protos code.

Production use is broad across Standard Library parsers/codecs, Package Tool,
Test Tool, CLI support, benchmarks and ordinary language carriers. Removing
Array from Core would therefore force a replacement primitive or pervasive
library dependency without deleting the underlying dense-sequence requirement.

Classification: **KEEP**.

### Deliberate Array absences

B9 corrected one initial scope assumption during evidence gathering: Core Array
does not currently define `add`, `removeAt`, insertion, arbitrary resize,
holes, negative-from-end indexing or a length/fill constructor.

Those are not mechanisms available for B9 to remove.

The approved classification preserves those absences:

```text
append/insert/remove/resize     ABSENT / RETAIN ABSENCE
negative indexing               ABSENT / RETAIN ABSENCE
holes                           ABSENT / RETAIN ABSENCE
length/fill constructor         ABSENT / RETAIN ABSENCE
```

Any later need belongs to a separate ergonomic/library/design path.

## Array iteration

`Array.each(block)` establishes a shallow logical snapshot at invocation
start and visits that snapshot in ascending index order.

This survives the audit because a live backend iterator would make mutation
during callbacks depend on physical representation or would require a different
restriction such as mutation prohibition or locking.

The current contract is deliberately representation-independent:

- later element replacement does not rewrite an already-established visit;
- suspension does not create an Array-wide lock;
- nested work may continue under ordinary Actor/task rules;
- an implementation need not allocate an eager copied Array.

Arrays that are never iterated do not pay a mandated snapshot-copy cost.

Classification: **KEEP**.

## Map analysis

Map is the standard ordinary equality/hash keyed collection.

Its key law is:

```text
query hash      -> queryKey.hash()
key comparison  -> queryKey == storedKey
```

The comparison direction is deterministic and intentionally does not invent
symmetry for arbitrary user-defined `==`.

This composes directly with the B6-retained equality/hash protocol. Replacing
Map matching with primitive identity would collapse it into IdentityMap.
Introducing a separate keyable/hashable type category would add a new privileged
semantic family and would not simplify the current object model.

Production Map use is foundational across JSON, TOML, CSV, CLI, package
resolution/manifest/document code, Test Tool and Standard Library collections.

Classification: **KEEP**.

## Recorded insertion hash and representative key

A normal Map obtains the query key hash for insertion/search and logically
records the insertion hash with a new association.

This is necessary because user-defined key state may later change. Recomputing
all stored-key hashes during lookup would cause additional observable user-code
execution and substantially different cost/effect semantics.

When an equal query updates an existing association, the original stored key
object remains the representative key and retains insertion position.

That preserves deterministic iteration and avoids silently replacing identity
merely because a later equal object was supplied.

Classification: **KEEP**.

## Insertion order

Map and IdentityMap preserve association insertion order.

B9 explicitly tested dropping this guarantee.

The ordering is not only conformance-test convenience. Current production code
consumes Map iteration order directly. In particular:

- JSON object serialization emits object members in Map `each` order;
- TOML serialization/snapshot code obtains ordered associations through Map
  iteration;
- collection/library/tool code uses deterministic Map iteration for reproducible
  processing.

Making iteration order unspecified would expose backend/hash-table accidents and
could make equivalent guest data serialize differently across implementations or
runtime revisions.

Core still does not prescribe a physical storage representation. Hash tables,
trees, ordered vectors, persistent structures or hybrids remain valid when they
preserve the observable order.

Classification: **KEEP**.

## Missing-key surface

The retained normal keyed surface distinguishes required lookup, presence
testing and expected absence:

```text
at(key)
    present -> exact stored value
    absent  -> Error

containsKey(key)
    present -> true
    absent  -> false

atIfAbsent(key, fallback)
    present -> exact stored value
    absent  -> invoke fallback once and return its exact result

remove(key)
    present -> remove and return previous exact value
    absent  -> Error
```

No ordinary value can be reserved as an absence sentinel because `null`,
`false` and every other ordinary object are legal stored values.

`containsKey` and `remove` have broad production use.

`atIfAbsent` is newer and has limited repository adoption at this baseline,
but D142 was triggered by repeated production expected-absence patterns and
selected a single-search lazy operation. A source helper implemented as
`containsKey + at` can observably invoke user `hash` / `==` twice and is
therefore not semantically equivalent.

AUD013 owns later repository-wide adoption. B9 does not reopen D142.

Classification: **KEEP**.

## Map / IdentityMap iteration snapshots

`Map.each` and `IdentityMap.each` establish a shallow logical association
snapshot in insertion order.

Each snapshot element preserves the representative key object and the exact
mapped value object observed at snapshot time.

Later insertion, removal or value replacement does not rewrite the current
iteration.

The strongest alternative was live iteration. It was rejected because it would
require one of:

- backend-dependent skipped/duplicated/reordered visits;
- hidden mutation prohibition;
- a collection lock around user callbacks;
- or another new iterator consistency institution.

The existing snapshot rule is semantically local and representation-independent.
Persistent structures, versioned cursors or copy-on-write state are permitted.

Classification: **KEEP**.

## Map state and comparison reentrancy

Normal Map search may invoke ordinary user `hash` and `==` behavior. Those
callbacks can have effects and may reach explicit suspension.

That makes same-Map reentrant mutation a real semantic problem: allowing key
comparison code to alter the candidate Map while the search is traversing it
would make results depend on the implementation's physical iterator/table
behavior.

The retained rule is narrow:

- while a normal Map key comparison is active, mutation of that same Map's keyed
  state fails before mutation;
- read-only use of the same Map remains allowed;
- unrelated Maps remain mutable;
- key/value objects themselves remain ordinary mutable objects;
- the rule follows a suspended comparison until that comparison unwinds;
- the restriction fails rather than blocks;
- no Map-wide transaction or global/Actor-wide lock is introduced.

Alternatives were not simpler:

- unrestricted mutation exposes representation accidents;
- full snapshot/transaction search adds greater machinery and runtime cost;
- banning effects or suspension specifically inside equality creates a new
  special callable regime.

Point-of-mutation state revalidation after user callbacks likewise preserves the
actual open/closed/frozen state rather than reserving stale mutation permission.

Classification: **KEEP**.

## IdentityMap analysis

IdentityMap uses semantic identity rather than ordinary equality:

```text
query identity hash -> identityHashOf(queryKey)
key comparison      -> queryKey === storedKey
```

No user-overridable `hash` or `==` is invoked for key search.

IdentityMap has real current production consumers, including:

- JSON cycle detection;
- TOML cycle and state tracking;
- CLI canonicalization cycle tracking;
- shared TOML document state;
- Test Tool SuiteGraph traversal;
- Test Tool resource reservation;
- `std:collections/IdentitySet`.

B9 therefore tested whether the capability could move out of Core while
remaining available.

### Standard Library emulation over normal Map

A source-backed library could wrap each key in an object whose `==` uses
`===`, but without direct semantic identity hashing it either degrades toward
linear/collision-heavy lookup or recreates a second hashing institution in
wrapper code.

That is not a simplification of the capability.

### Standard Library public factory with native backing

Moving only the public `IdentityMap` binding/module to Standard Library while
retaining specialized backing would preserve nearly all current runtime,
transfer, Bytecode and semantic machinery while adding a library/native bridge.

That removes one Core-visible name but does not materially reduce continuing
complexity.

### Strategy-configured generic Map

Another candidate was one generic Map parameterized by a key strategy.

That would charge ordinary Map construction/search with strategy identity,
dispatch and policy merely to avoid one explicit factory for a genuinely
different key law.

It increases general Core abstraction surface and moves rather than removes the
distinction.

The explicit IdentityMap family therefore remains the smallest current
institution that preserves semantic-identity keys efficiently and predictably.

Classification: **KEEP**.

## Collection-family topology

The standard topology remains:

```text
Array       -> Object
Map         -> Object
IdentityMap -> Object
```

IdentityMap does not delegate to Map.

Sharing selector names does not imply that delegation should model a generic
collection hierarchy. Both Map kinds own represented keyed state with different
fundamental key laws.

B9 found no justification for adding:

- a generic Core `Collection` prototype;
- a public `Association` value family;
- `Map.recognizes`;
- `IdentityMap.recognizes`.

B6 already established that exact semantic-family recognition is not a generic
substitute for protocol-oriented programming.

Classification:

```text
generic Collection hierarchy     ABSENT / RETAIN ABSENCE
Association value family         ABSENT / RETAIN ABSENCE
Map recognizer                    ABSENT / RETAIN ABSENCE
IdentityMap recognizer            ABSENT / RETAIN ABSENCE
```

## Strongest attempted removals

### Remove Array from Core

Rejected. Dense indexed storage is pervasive and foundational; removal would
replace rather than eliminate the requirement.

### Remove Map from Core

Rejected. Equality/hash keyed association is pervasive production
infrastructure and composes directly with retained B6 protocols.

### Relocate IdentityMap to Standard Library

Rejected. Current production use is substantial and ordinary emulation either
loses efficient identity hashing or recreates specialized machinery. Keeping
native backing but moving only the name does not materially simplify the
runtime.

### Drop insertion order

Rejected. Production serializers and tools consume deterministic Map iteration.
Unspecified order would expose backend representation.

### Make `each` live instead of snapshot-based

Rejected. Live iteration would require more restrictive mutation rules, hidden
locking or representation-dependent traversal behavior.

### Remove `containsKey`

Rejected. Presence-only checks are common and cannot be represented safely by
`at` because all ordinary values, including `null`, are legal mappings.

### Remove `atIfAbsent`

Rejected. D142 already established real production friction and an observable
single-search semantic reason why ordinary `containsKey + at` composition is
not equivalent.

### Relax same-Map mutation restriction during equality

Rejected. It would make an in-progress search depend on implementation-specific
candidate traversal. Stronger alternatives cost more.

## Boundary handoffs

- **AUD009-C** owns Actor/task/concurrency model independently of the narrow
  collection-owned suspension consequences already specified here.
- **AUD009-E** owns Standard Library collection breadth including
  `std:collections/Array`, Set/IdentitySet, Range and later optional collection
  families.
- **AUD009-G** owns physical collection representations and backend/runtime
  implementation duplication.
- **AUD009-A1 / D131 / I041** remain authoritative for `Array.match` and
  `Map.match`.
- **D136/I040** remain authoritative for populated Map source construction.
- **D142/I049** remain authoritative for `atIfAbsent`.
- **D156** remains the independent fixed-width Integer placement decision; B9's
  exact-Integer indexing/hash contracts follow the normative numeric model that
  exists at each revision.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#620
CHECKPOINT_COMMENT=5739218137
APPROVAL_COMMENT=5739225689
DATE=2026-09-19

ARRAY_CORE_MODEL=KEEP
MAP_CORE_MODEL=KEEP
IDENTITYMAP_CORE_MODEL=KEEP
MAP_INSERTION_ORDER=KEEP
COLLECTION_SNAPSHOT_ITERATION=KEEP
MAP_REENTRANCY_STATE_RULES=KEEP
MAP_LOOKUP_ABSENCE_SURFACE=KEEP

ARRAY_RESIZE_REMOVE_CORE_SURFACE=ABSENT_RETAIN_ABSENCE
NEGATIVE_INDEXING_HOLES=ABSENT_RETAIN_ABSENCE
GENERIC_COLLECTION_HIERARCHY=ABSENT_RETAIN_ABSENCE
MAP_IDENTITYMAP_RECOGNIZERS=ABSENT_RETAIN_ABSENCE
ASSOCIATION_VALUE_FAMILY=ABSENT_RETAIN_ABSENCE

DERIVED_DECISION=NONE
NORMATIVE_CHANGE_AUTHORIZED_BY_B9=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_B9=NO
```

## Closure checklist

```text
ARRAY_FAMILY=KEEP
ARRAY_FRESH_IDENTITY=KEEP
ARRAY_FACTORY=KEEP
ARRAY_DENSE_INDEXED_STATE=KEEP
ARRAY_ZERO_BASED_EXACT_INTEGER_INDEXING=KEEP
ARRAY_SIZE_AT_ATPUT=KEEP
ARRAY_EACH_SNAPSHOT=KEEP
ARRAY_ASCENDING_ORDER=KEEP
ARRAY_RECOGNIZES=KEEP

MAP_FAMILY=KEEP
MAP_FACTORY=KEEP
MAP_HASH_DIRECTED_EQUALITY_KEY_LAW=KEEP
MAP_RECORDED_INSERTION_HASH=KEEP
MAP_REPRESENTATIVE_KEY=KEEP
MAP_INSERTION_ORDER=KEEP
MAP_SIZE_AT_ATPUT_CONTAINSKEY_ATIFABSENT_REMOVE=KEEP
MAP_EACH_SNAPSHOT=KEEP

IDENTITYMAP_FAMILY=KEEP
IDENTITYMAP_IDENTITYHASH_IDENTITY_KEY_LAW=KEEP
IDENTITYMAP_INSERTION_ORDER=KEEP
IDENTITYMAP_EACH_SNAPSHOT=KEEP

MAP_STATE_REVALIDATION=KEEP
MAP_COMPARISON_MUTATION_RESTRICTION=KEEP
MAP_COMPARISON_SUSPENSION_LIFETIME=KEEP
NO_COLLECTION_LOCK_OR_TRANSACTION=KEEP

ARRAY_APPEND_INSERT_REMOVE_RESIZE=ABSENT_RETAIN_ABSENCE
NEGATIVE_INDEXING=ABSENT_RETAIN_ABSENCE
ARRAY_HOLES=ABSENT_RETAIN_ABSENCE
GENERIC_COLLECTION_HIERARCHY=ABSENT_RETAIN_ABSENCE
MAP_RECOGNIZER=ABSENT_RETAIN_ABSENCE
IDENTITYMAP_RECOGNIZER=ABSENT_RETAIN_ABSENCE
ASSOCIATION_VALUE_FAMILY=ABSENT_RETAIN_ABSENCE

OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTE=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B9_CLASSIFICATION=COMPLETE
```

AUD009-B9 is complete once this durable record and the required live GitHub
closure postconditions are verified.
