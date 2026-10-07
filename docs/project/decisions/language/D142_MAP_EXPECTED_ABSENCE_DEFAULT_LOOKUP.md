# D142 — Map expected-absence and default lookup semantics

## Decision

D142 ratifies **Candidate C — lazy non-mutating expected-absence lookup**.

```text
D142_STATUS=RATIFIED
SELECTED_CANDIDATE=C
SELECTED_SELECTOR=atIfAbsent
PROTOS_REVISION=675e03fcdf8043b7bba75d402c43e14b49b13ece
PROJECT_RECORD_BASE=a383c8957c6796bdf726f328b0568ddc68c5de54
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is durable non-normative decision/rationale evidence. Observable Protos
semantics remain authoritative under `guillermomolina/protos:spec/**`.

The complete comparative investigation, scorecard, adversarial analysis, and
deferral rationale are recorded in:

```text
docs/project/work/D142/D142_MAP_EXPECTED_ABSENCE_DEFAULT_LOOKUP_DECISION_PACKET.md
```

## Owner approval

The project owner explicitly approved the exact Candidate C in the active D142
interaction:

```text
aprobado
```

That approval immediately followed the exact candidate request for:

```text
Map/IdentityMap.atIfAbsent(key, fallback)
one logical key search
zero-argument lazy fallback invoked only on absence
no initial atOr
no initial atIfAbsentPut
all existing Map/IdentityMap invariants preserved
```

## Ratified semantics

The selected standard protocol is:

```text
Map.atIfAbsent(key, fallback)
IdentityMap.atIfAbsent(key, fallback)
```

The ratified semantic contract is:

- receiver, `key`, and the fallback-producing argument expression are evaluated
  under the ordinary left-to-right, exactly-once call rules;
- one logical key search is performed using the receiver's existing key law;
- normal `Map` therefore uses its existing query hash and directed
  `queryKey == storedKey` semantics;
- `IdentityMap` uses its existing identity-hash and `===` semantics;
- if a matching association exists, its exact stored value is returned,
  including `null` or `false`;
- on the present path, the fallback is neither callability-validated nor
  invoked;
- if the key is absent, the fallback is required to be invokable through the
  ordinary polymorphic invocation protocol and is invoked exactly once with zero
  positional arguments;
- the fallback's exact normal result is returned unchanged;
- invocation/search failure, suspension, effects, and non-local control behavior
  follow their existing semantics with no rollback;
- `atIfAbsent` itself performs no keyed-entry mutation;
- fallback execution begins after the missing-key search completes and may
  perform ordinary mutations when existing rules permit them;
- there is no second lookup after fallback execution;
- the fallback receives no implicit key or receiver arguments;
- no atomicity, key reservation, transaction, rollback, memoization, or
  compute-once guarantee is introduced.

## Preserved invariants

D142 does not change:

```text
at(key) missing -> Error
containsKey(key) -> explicit non-failing presence query
ABSENT != PRESENT_WITH_NULL
Map hash/equality key law
IdentityMap identity key law
Map/IdentityMap receiver-domain rules
open/closed/frozen rules
same-Map comparison reentrancy rules
insertion order
remove semantics
ordinary invocation/evaluation order
```

It introduces no hidden sentinel, `Optional`/`Maybe`, truthiness, ambient Map
default state, syntax, rollback, or concurrency atomicity.

## Why the operation is Core rather than a library-only wrapper

A normal Protos `Map` search can execute observable user-defined `hash` and
directed `==` behavior.

A library wrapper composed from:

```text
containsKey(key)
at(key)
```

can therefore perform two independent searches on the present path. D142
selects one logical search as observable standard semantics.

The contract still leaves physical representation and search implementation
free: "one logical search" does not prescribe a hash-table layout or internal
algorithm.

## Why only the lazy non-mutating operation is selected

The repository demonstrates current need for expected-absence read fallback.
The lazy form also expresses:

- an already-computed fallback by capturing it;
- computed fallback;
- domain-specific required lookup by using a fallback that fails.

An eager `atOr` convenience can be added later without changing this model, so
there is no present need to add both.

A mutating get-or-create operation is materially heavier. A producer can mutate
the same Map or key after a miss, forcing a future design to choose explicitly
between re-search, mutation restrictions, reservation/transaction machinery, or
another contract. D142 does not pre-decide that separate semantic boundary.

## GITHUB021 consistency

The selected candidate preserves every D142 owner-approved/current normative
invariant checked before approval.

```text
PRESERVES_RECORDED_INVARIANTS=YES
HIDDEN_REOPENING=NO
NEW_SENTINEL_INSTITUTION=NO
NEW_OPTIONAL_INSTITUTION=NO
NEW_MUTATION_CONTRACT=NO
NEW_CONCURRENCY_CONTRACT=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Deferred questions

The following remain intentionally unselected:

- eager `atOr`-class lookup;
- `atIfAbsentPut` / compute-and-insert;
- producer key/receiver parameters;
- ambient receiver-wide default behavior;
- explicit presence-result carrier;
- Optional/Maybe;
- syntax;
- atomic compute/install behavior.

These may be reconsidered only from future evidence and must not be inferred from
D142.

## Normative and implementation boundary

This durable record selects the design but is not normative language authority.

Before implementation proceeds, the ratified semantics must be published in the
applicable normative owner:

```text
guillermomolina/protos:spec/semantics/VALUES_AND_COLLECTIONS.md
```

with the corresponding global entry in:

```text
guillermomolina/protos:spec/PROTOS_SPEC_CHANGELOG.md
```

Implementation, tests, examples, migration of existing callers, implementation
versioning, and any later eager or mutating surface are separate work.
