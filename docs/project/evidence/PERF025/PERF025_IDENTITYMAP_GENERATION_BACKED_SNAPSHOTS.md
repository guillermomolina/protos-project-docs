# PERF025 — IdentityMap generation-backed snapshots

## Status

```text
STATUS=COMPLETE
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=85bd050e8706c02205db4ede8824681a1180d075
PRODUCT_PARENT_REVISION=4fa64c3821e9e6566fd9a1f0b38fdfcba875fd78
PRODUCT_VERSION=0.3.167-SNAPSHOT
PRODUCT_COMMIT=PERF025: make IdentityMap snapshots generation-backed

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
PERFORMANCE_MEASUREMENT_PERFORMED=NO
```

## Purpose

Remove unconditional O(n) physical copying from standard `IdentityMap`
snapshot publication while preserving the existing shallow logical snapshot
semantics exactly.

This is the family-specific completion of the PERF025 collection-snapshot
representation line for `IdentityMap`. It follows the earlier Array
generation-backed publication at
`f48f67a94553b8929bb3830005850d55ccc53cf3`.

## Published representation

`ProtosIdentityMapValue` now owns a generation containing:

- insertion-order entries;
- exact recorded-identity-hash buckets;
- lazily published keyed and association snapshot views; and
- publication state controlling copy-on-write detachment.

The physical behavior is:

```text
BEFORE_FIRST_SNAPSHOT_MUTATION=IN_PLACE
REPEATED_UNCHANGED_SNAPSHOT=REUSE_VIEW
FIRST_MUTATION_AFTER_PUBLICATION=DETACH_GENERATION
OLD_SNAPSHOT_MEMBERSHIP=STABLE
OLD_SNAPSHOT_ORDER=STABLE
OLD_SNAPSHOT_KEY_REFERENCES=STABLE
OLD_SNAPSHOT_VALUE_REFERENCES=STABLE
RECORDED_IDENTITY_HASH_BUCKETS=REBUILT_ON_DETACH
PER_ASSOCIATION_PAIR_ALLOCATION_ON_ASSOCIATION_SNAPSHOT=REMOVED
```

A published generation is never mutated again. Replacement, removal or append
after publication moves the live map to a fresh generation, cloning the live
entry state and rebuilding the exact-hash bucket index as required.

## Structured IdentityMap.each

The structured `IdentityMap.each` preparation path now retains
`value.associationSnapshot()` directly instead of immediately wrapping that
already-stable snapshot in another `List.copyOf(...)`.

This removes the redundant second O(n) reference copy while keeping the
callback snapshot boundary unchanged.

## Preserved semantics

The publication preserves:

- exact semantic identity matching through `ProtosIdentity.identical`;
- exact recorded identity hashes;
- collision-bucket search;
- representative key references;
- insertion order;
- replacement without reordering;
- remove/reinsert tail ordering;
- shallow snapshot membership/value capture;
- OPEN/CLOSED/FROZEN mutation rules;
- callback suspension/resumption;
- nested snapshot independence;
- Actor transfer behavior; and
- isolated-P transfer behavior.

The independently known P-transfer recorded-identity-hash debt was deliberately
not changed by this slice.

## Validation

Maintainer-executed validation reported:

```text
NEW_GENERATION_FOCAL=4_PASSED_0_FAILED
AFFECTED_EACH_AND_INDEX_REGRESSIONS=20_PASSED_0_FAILED
ACTOR_AND_COLLECTION_STATE_REGRESSIONS=16_PASSED_0_FAILED
P_TRANSFER_FOCAL=1_PASSED_0_FAILED
POST_REBASE_AFFECTED_REGRESSIONS=17_PASSED_0_FAILED
FULL_MAKE_TEST=PASS
GIT_DIFF_CHECK=PASS
PUBLISHED_WORKTREE=CLEAN
```

The first execution of the new focal found one test-only textual-source guard
that matched a different legitimate `Map` path. Productive code was already
correct; the guard was scoped to `PreparedIdentityMapEachCall`, after which the
focal passed 4/4. No productive change was made in response to that test-only
failure.

## Publication

```text
PRODUCT_REVISION=85bd050e8706c02205db4ede8824681a1180d075
PRODUCT_BRANCH=main
PRODUCT_PUSH=PASS
PRODUCT_WORKTREE=CLEAN
```

## PERF025 residual consequence

This closes the `IdentityMap` sub-slice of collection snapshot physical
representation.

The current reconciled PERF025 coordination state is:

```text
ARRAY_SNAPSHOT_REPRESENTATION=COMPLETE
MAP_ADDITIONAL_SNAPSHOT_REPRESENTATION=NOT_JUSTIFIED
IDENTITYMAP_SNAPSHOT_REPRESENTATION=COMPLETE
BYTES_ADDITIONAL_SNAPSHOT_REPRESENTATION=NOT_JUSTIFIED
D179_CAPTURED_MEMBERSHIP_SPECIALIZATION=NOT_JUSTIFIED
SHARED_SHAPE_MEMBER_LOOKUP=BOUNDED_H3A_COMPLETE

CALLBACK_LAZY_SEMANTIC_ACTIVATION=NEXT
OPTIONAL_RESIDUAL_ACTIVATION_CONSUMER_SPECIALIZATION=AFTER_LAZY_ACTIVATION_IF_JUSTIFIED
FINAL_BENCHMARK_PENDING=YES
```

Therefore no further Map, Bytes, D179, Shape, Actor/Task or collection-snapshot
product slice remains in PERF025 at this checkpoint.
