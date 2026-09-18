# D142 — Map expected-absence and default lookup semantics

## Decision state and authority

This is the durable non-normative decision packet for
`guillermomolina/protos#572`.

```text
D142_STATUS=OWNER_APPROVED
SELECTED_CANDIDATE=C
SELECTED_SELECTOR=atIfAbsent
PROTOS_REVISION=675e03fcdf8043b7bba75d402c43e14b49b13ece
PROJECT_DOCS_BASE=a383c8957c6796bdf726f328b0568ddc68c5de54
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

Observable Protos semantics remain authoritative only through the normative
specification under `guillermomolina/protos:spec/**`. This packet records the
research, comparison, exact approved candidate, and intentionally deferred
surface.

## Exact decision problem

Current Core distinguishes required lookup from presence testing:

```text
map.at(key)
    present -> exact stored value
    absent  -> Error

map.containsKey(key)
    present -> true
    absent  -> false
```

Every Protos object, including `null` and `false`, is a valid stored Map value.
Expected absence therefore cannot be represented by returning an ordinary
sentinel without collapsing:

```text
ABSENT != PRESENT_WITH_NULL
```

Repository production code repeatedly composes `containsKey` with indexed lookup
for expected absence. On normal `Map`, that composition can perform two logical
key searches. Because `hash` and directed `==` are ordinary observable behavior,
the duplicate search is not merely an implementation-performance detail.

D142 therefore asks for the smallest standard keyed-lookup protocol, if any,
that handles expected absence while preserving current Map semantics.

## Current Protos constraints

The selected candidate preserves all of the following existing constraints:

- missing `at(key)` continues to signal `Error`;
- `containsKey(key)` remains the non-failing presence query;
- stored `null`, `false`, and every other ordinary value remain present values;
- normal `Map` key search continues to use the existing deterministic query
  `hash` and directed `queryKey == storedKey` semantics;
- `IdentityMap` continues to use semantic identity hashing and `===`;
- search callbacks, their effects, suspension, failure, and same-Map comparison
  restrictions remain governed by the existing Map rules;
- ordinary receiver and argument evaluation remains left-to-right and exactly
  once;
- standard Map/IdentityMap receiver-domain invariants remain unchanged;
- open/closed/frozen rules remain authoritative;
- no hidden absence sentinel, `Optional`/`Maybe`, truthiness, ambient default
  state, syntax, transaction, rollback, or concurrency atomicity is introduced.

## Repository usage classification

The audited repository contains `containsKey` across 41 `.protos` files, with 22
production Standard Library/Tool files in the search result.

The uses separate into materially different classes.

### Presence-only and validation

Set membership, uniqueness validation, duplicate detection, graph/cycle checks,
schema validation, and other pure presence questions are already well served by
`containsKey`. They are not evidence for another lookup operation.

Representative areas include:

- `protos/lib/collections/Set.protos`;
- `protos/lib/collections/IdentitySet.protos`;
- CLI option/name validation;
- JSON/TOML duplicate-name validation;
- package-resolution uniqueness checks;
- Test Tool graph/resource identity checks.

### Required lookup with domain-specific failure

Several helpers perform:

```protos
entries.containsKey(name).ifFalse(() => {
    fail()
})
entries[name]
```

This is real repetition, but it does not independently justify a separate Core
"required lookup with custom error" API. The approved lazy absence operation can
express this case by using a failing fallback.

### Non-mutating read with fallback

CSV, CLI, JSON/TOML, package metadata, and Test Tool code contain forms such as:

```protos
nextResult: result
chunks.containsKey(level).ifTrue(() => {
    nextResult = Array(...result, ...chunks[level])
})
```

and:

```protos
packageLocator: null
entries.containsKey("locator").ifTrue(() => {
    packageLocator = entries["locator"]
})
```

This is the principal demonstrated need for D142.

### Lazy create-on-absence plus insertion

TOML document construction and Test Tool structures also contain patterns of the
form:

```protos
child: null
entries.containsKey(name).ifTrue(() => {
    child = entries[name]
})
entries.containsKey(name).ifFalse(() => {
    child = makeChild()
    entries[name] = child
})
```

This is a real use case, but it has materially heavier semantics than a
non-mutating lookup fallback and is intentionally deferred rather than folded
into the initial Core surface.

## Comparative research

### Smalltalk / Pharo

Pharo `Dictionary>>at:ifAbsent:` treats expected absence as an ordinary
per-call remedial block rather than requiring exception handling.

Primary reference:
https://books.pharo.org/deep-into-pharo/

Contribution to D142: a mature message-oriented precedent for a lazy,
non-mutating absence callback with no sentinel institution.

### Self

Self collection protocols include `at: IfAbsent:` as an ordinary selector and
`includesKey:` as a separate presence query.

Primary reference:
https://handbook.selflanguage.org/2024.1/usefulselectors.html

Contribution to D142: especially relevant prototype/message-oriented evidence
that expected absence can remain an ordinary message/callback protocol.

### Python

Python separates non-mutating `dict.get(key, default)` from mutating
`dict.setdefault(key, default)`.

Primary reference:
https://docs.python.org/3/library/stdtypes.html

Contribution to D142: evidence that read-default and insert-default are
semantically distinct operations rather than one overloaded behavior.

### Ruby

Ruby `Hash#fetch` supports a per-call default value or block while Hash also
supports ambient default/default-proc state.

Primary reference:
https://docs.ruby-lang.org/en/master/Hash.html

Contribution to D142: confirms both eager and lazy per-call models, while the
ambient-default facility illustrates additional persistent policy/state that the
current Protos need does not justify.

### Java

Java distinguishes `Map.getOrDefault` from `computeIfAbsent`.
`computeIfAbsent` treats a mapping to `null` as absent-like for computation and
carries mutation/reentrancy qualifications.

Primary reference:
https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/util/Map.html

Contribution to D142: strong evidence for separating non-mutating fallback from
compute-and-insert semantics, and a warning not to import Java's null
conflation into Protos.

### Kotlin

Kotlin exposes `getOrDefault`, lazy `getOrElse`, and mutable-map `getOrPut`.
Its newer missing/null-sensitive variants explicitly separate missing keys from
mapped null values.

Primary reference:
https://kotlinlang.org/docs/map-operations.html

Contribution to D142: evidence that eager, lazy, and mutating default handling
are distinct useful contracts and that missing-vs-null semantics matter.

### JavaScript

JavaScript `Map.get` returns `undefined` when no key is present and `Map.has`
must be used to distinguish some presence cases.

Primary reference:
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/get

Contribution to D142: a contrasting sentinel-like ordinary return model that
does not fit Protos because Protos deliberately has no `undefined` and every
ordinary value must remain storable without losing presence information.

## Complete candidate set

### Candidate A — status quo

Keep only `containsKey` + `at` and domain helpers.

Advantages: zero new public surface and maximal immediate reversibility.

Cost: demonstrated repetitive source remains; present-path composition can
repeat observable key-search behavior.

### Candidate B — eager non-mutating default lookup

Add a conceptual operation such as:

```text
map.atOr(key, defaultValue)
```

Advantages: simple, no callback invocation on absence, and a single logical key
search.

Cost: it solves only already-computed defaults. A later lazy form would still be
needed for computed/failing fallbacks.

### Candidate C — lazy non-mutating absence fallback

Add:

```text
map.atIfAbsent(key, fallback)
```

with one logical key search and path-sensitive fallback invocation.

This is the selected candidate.

### Candidate D — lookup-or-insert / initialize on absence

Conceptually add a mutating operation such as `atIfAbsentPut`.

This is not part of the selected initial surface. It introduces additional
semantic questions around callback mutation, key mutation, state revalidation,
representative-key preservation, search reuse, insertion order, and failure.

### Candidate E — explicit presence/result carrier

Return an object carrying both presence and value.

Rejected for the initial design: it introduces a new carrier/institution for a
problem already expressible through ordinary callbacks and `containsKey`.

### Candidate F — Standard Library helper only

Keep Core unchanged and implement convenience through `containsKey` + `at`.

Rejected as the standard solution because it cannot in general guarantee the
approved one-search behavior using only the existing public keyed operations.

### Ambient Map default state

Ruby-style receiver-wide default/default-proc state was considered and rejected
for the initial design. It adds persistent policy, state, copy/freeze/lifetime
interactions, and coordination for a need that is demonstrated per call.

## Comparative scorecard

Scores are 1–5. They are comparison aids, not decision authority.

| Criterion | A | B | C |
|---|---:|---:|---:|
| Correctness / invariant preservation | 5/HIGH | 5/HIGH | 5/HIGH |
| Protos alignment | 4/HIGH | 4/HIGH | 5/HIGH |
| Present-need proportionality | 2/HIGH | 4/HIGH | 5/HIGH |
| Incremental growth | 5/HIGH | 4/HIGH | 5/HIGH |
| Future-option resilience | 5/HIGH | 4/MEDIUM | 5/HIGH |
| Scalability | 3/HIGH | 5/HIGH | 5/HIGH |
| Conceptual simplicity | 4/HIGH | 5/HIGH | 5/HIGH |
| Portability / implementation freedom | 5/HIGH | 5/HIGH | 5/HIGH |
| Runtime / resource cost | 2/HIGH | 5/HIGH | 4/MEDIUM |
| Failure / operability | 3/HIGH | 5/HIGH | 5/HIGH |
| Deferral / reversibility / migration | 4/HIGH | 4/HIGH | 5/HIGH |
| Evidence maturity / implementation risk | 5/HIGH | 5/HIGH | 5/HIGH |

Non-authoritative totals are A=47, B=55, C=59.

### Score rationale

**A** preserves the smallest surface and every existing invariant, but current
callers continue paying source complexity and, on present lookup paths, possible
duplicate observable search. Deferral remains possible but the audited pressure
is already current rather than speculative.

**B** is semantically simple and efficient for already-computed defaults. It is
less general than the demonstrated need because lazy computation and
domain-specific failure still require another mechanism. Adding lazy lookup
later would be compatible, so B is coherent but not the smallest mechanism that
covers the current cases.

**C** uses only an ordinary message plus the existing invokable protocol. One
operation covers constant/default capture, computed fallback, and custom failure
without introducing mutation semantics, sentinel values, new carrier objects,
or syntax. Eager and mutating variants can be added later without changing C.

## Selected semantics — Candidate C

The selected standard selectors are:

```text
Map.atIfAbsent(key, fallback)
IdentityMap.atIfAbsent(key, fallback)
```

The exact approved contract is:

1. Receiver, `key`, and the expression producing `fallback` are evaluated under
   ordinary left-to-right, exactly-once invocation rules before the standard
   behavior executes. Creating a Closure argument creates the Closure but does
   not execute its body.
2. The receiver is validated under the existing standard Map/IdentityMap
   receiver-domain rule.
3. Exactly one logical key search is performed using the receiver's existing key
   semantics:
   - normal `Map`: the existing query-hash plus directed `queryKey == storedKey`
     search rules;
   - `IdentityMap`: the existing semantic identity-hash plus `===` search rules.
4. If a matching association exists, return its exact stored value object,
   including canonical `null`, `false`, or any other value.
5. On the present path, `fallback` is neither callability-validated nor invoked.
6. If no association matches, `fallback` is required to be invokable through the
   ordinary polymorphic invocation protocol and is invoked exactly once with
   zero positional arguments.
7. Callability validation is therefore path-sensitive. It does not suppress the
   earlier ordinary evaluation of the argument expression that produced the
   fallback object.
8. The exact normal result of the fallback invocation is returned unchanged.
9. Errors, suspension, non-local control transfer, and other ordinary invocation
   behavior propagate according to their existing semantics. No effects are
   rolled back.
10. `atIfAbsent` itself performs no keyed-entry mutation.
11. A fallback runs only after the missing-key search has completed. It may
    perform ordinary mutations, including on the same Map, when those mutations
    are otherwise permitted by the existing state and reentrancy rules.
12. If the fallback inserts, removes, closes, freezes, or otherwise affects the
    receiver after absence was established, `atIfAbsent` does not perform a
    second search and does not reinterpret its result. It returns the fallback's
    exact result.
13. The fallback receives neither the key nor the Map as implicit positional
    arguments. Code that needs them may capture them through ordinary closure
    semantics.
14. No atomicity, key reservation, transaction, rollback, memoization, or
    compute-once guarantee is implied.
15. Existing `at`, indexed access, `containsKey`, mutation, removal, ordering,
    hashing/equality, IdentityMap identity, and open/closed/frozen semantics are
    unchanged.

## Why one logical search is part of the contract

On normal `Map`, `hash` and directed `==` are observable Protos behavior and may
have effects, signal, or suspend.

Therefore:

```protos
map.containsKey(key).ifTrue(() => {
    map[key]
})
```

can be observably different from a single keyed operation: the first form may
perform two independent searches.

The selected operation closes that semantic gap. "One logical search" does not
mandate a table layout, caching strategy, or physical algorithm; an
implementation remains free to use any representation that produces exactly the
existing Map key-search behavior without beginning a second lookup.

## Map and IdentityMap ownership

The same `atIfAbsent` protocol belongs to both standard `Map` and `IdentityMap`.

The operation itself is identical at the absence/fallback level. The search is
not identical: each receiver retains its already-ratified key law.

This preserves the existing rule that later standard `IdentityMap` operations
defined in terms of finding a key use identity-key search rather than normal
Map hash/equality callbacks.

## Mutating get-or-create adversarial result

A mutating initial operation was deliberately not selected.

Suppose a producer runs after a missing-key search:

```protos
map.atIfAbsentPut(key, () => {
    mutateKeyOrMap()
    value
})
```

The producer can alter the same Map or the key before insertion. Reusing the
pre-producer search/hash may then be invalid. A robust mutating design must make
a further explicit choice, for example:

- search again after the producer, repeating observable search behavior;
- forbid some same-Map/key mutation dynamically during the producer; or
- introduce reservation/transaction machinery.

Those are substantive semantics, not implementation details. No current evidence
justifies selecting one merely to ship get-or-create together with expected
read fallback.

## Incremental-design gates

### Pay for what you need

Candidate C adds one ordinary selector and incurs callback invocation only on an
actual miss. Programs not using the selector pay no new conceptual or runtime
coordination cost.

### Grow as you need

An eager `atOr` can be added later as a convenience without changing
`atIfAbsent`.

A future mutating `atIfAbsentPut`-class operation can also be added later after
its own mutation/reentrancy contract is resolved. Candidate C does not reserve
or predefine that contract.

### Cost of deferral

Deferring eager lookup costs closure ceremony for callers whose fallback is
already computed. The later addition is purely additive.

Deferring lookup-or-insert means current miss paths may perform one search in
`atIfAbsent` and another during subsequent insertion. This is a real performance
and observable-effect cost, but it does not require a future change to Map
identity, persistence, representation, ownership, or the semantics selected
here.

The bounded deferral cost is smaller than prematurely fixing the heavier
producer/mutation contract.

### Smallest sufficient solution

The smallest solution satisfying the demonstrated cases and one-search
requirement is one lazy non-mutating operation.

No current evidence justifies also adding eager, mutating, result-carrier,
ambient-default, Optional/Maybe, or syntax facilities.

### Over/underengineering red flags

Candidate A carries an underengineering warning because the repository already
contains repeated expected-absence code and composition can repeat observable
search.

Candidate C has no non-compensating overengineering warning: it introduces no
new value family, syntax category, persistent Map state, concurrency guarantee,
or mutation institution.

Adding eager plus lazy plus mutating variants immediately would carry an
overengineering warning because their additional contracts are not all required
to solve the present semantic gap.

## Future-scenario stress test

### Larger Maps and workloads

The selected operation avoids a second logical search on the present path and
does not prescribe representation. It remains valid for hash tables, persistent
structures, specialized indexes, or another conforming implementation.

### Suspension and Actors

Normal `Map` search keeps its existing callback/suspension rules. The fallback
begins only after search completion, so D142 introduces no new search-held
critical region.

### Future shared/concurrent Maps

D142 makes no atomic compute/install promise. A future genuinely shared or
atomic map abstraction may define a stronger operation without changing the
meaning of `atIfAbsent`.

### Alternative runtimes

The contract is expressed only in observable lookup and invocation semantics.
It does not depend on Truffle, the JVM, physical hashing, a particular table
representation, locks, or host facilities.

### Future requirement that could make C regrettable

If memoization/get-or-create becomes pervasive, the extra insertion search and
producer/reentrancy constraints may justify a first-class mutating operation.

The escape path remains additive: design that operation from then-current
evidence without weakening or reinterpreting `atIfAbsent`.

## GITHUB021 invariant/delta consistency

The approved candidate was checked against every invariant recorded in D142.

```text
AT_MISSING_REMAINS_ERROR=PASS
CONTAINS_KEY_REMAINS_PRESENCE_QUERY=PASS
ABSENT_DISTINCT_FROM_PRESENT_NULL=PASS
NORMAL_MAP_HASH_AND_DIRECTED_EQUALITY=PASS
IDENTITY_MAP_IDENTITY_KEY_LAW=PASS
SEARCH_CALLBACK_EFFECTS_AND_FAILURES=PASS
LEFT_TO_RIGHT_EXACTLY_ONCE_EVALUATION=PASS
MAP_RECEIVER_DOMAIN=PASS
OPEN_CLOSED_FROZEN_RULES=PASS
NO_HIDDEN_SENTINEL=PASS
NO_OPTIONAL_MAYBE=PASS
NO_TRUTHINESS=PASS
NO_SYNTAX_CHANGE=PASS
NO_TRANSACTION_ROLLBACK=PASS
NO_CONCURRENCY_ATOMICITY_PROMISE=PASS
MATERIALLY_NEW_HIDDEN_CONSEQUENCE=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

No recorded owner-approved invariant is reopened or contradicted.

## Strongest argument against Candidate C

The repository already contains create-on-absence patterns, so a lazy
non-mutating operation does not eliminate every repeated lookup. For a miss that
immediately inserts, a later ordinary `atPut` may search the key again.
Additionally, callers with an already-computed fallback must still wrap it in an
invokable object.

A larger initial surface could remove those costs.

The reason not to do so is that eager lookup can be added later with no semantic
migration, while mutating lookup introduces unresolved producer/reentrancy/key
mutation choices. Paying those permanent semantics now is not justified by the
current evidence.

## Owner approval

The project owner explicitly approved the exact Candidate C presented with the
semantics above in the active D142 interaction:

```text
aprobado
```

The approval followed the exact request to approve:

```text
C with Map/IdentityMap.atIfAbsent(key, fallback), one logical key search,
zero-argument path-sensitive fallback invocation, no initial atOr or
atIfAbsentPut, and all existing Map/IdentityMap invariants preserved.
```

```text
DECISION_APPROVAL_PROVENANCE=PASS
```

## Deferred work

D142 intentionally leaves the following outside the selected surface:

- eager `atOr`-class convenience;
- mutating `atIfAbsentPut` / compute-and-insert semantics;
- callback key/receiver arguments;
- dedicated custom-failure lookup;
- ambient Map defaults;
- presence/result carrier objects;
- Optional/Maybe;
- hidden sentinels;
- new syntax;
- atomic compute/install semantics.

Before implementation, the selected semantics must be published through the
applicable normative specification owner in
`spec/semantics/VALUES_AND_COLLECTIONS.md` and recorded in
`spec/PROTOS_SPEC_CHANGELOG.md`.

D142 itself does not authorize implementation to precede that normative
publication.
