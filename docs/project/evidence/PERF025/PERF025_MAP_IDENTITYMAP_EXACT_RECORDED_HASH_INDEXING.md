# PERF025 — Map / IdentityMap exact recorded-hash physical indexing

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the physical Map/IdentityMap indexing change, the preserved semantic boundaries,
and maintainer-reported validation. It does not claim a measured performance
magnitude.

## Exact product publication

```text
PROTOS_REVISION=98680d056659d475bcfec85528e914bfec09209e
PROTOS_PARENT_REVISION=16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb
PROTOS_VERSION=0.3.161-SNAPSHOT
COMMIT_SUBJECT=PERF025: index Map and IdentityMap by recorded hash
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
COMMITS=1
FILES_CHANGED=8
ADDITIONS=406
DELETIONS=53
```

The product publication is exactly one commit ahead of the preceding
`0.3.160-SNAPSHOT` object-model pay-as-you-grow publication.

## Bounded objective

Before this slice, keyed lookup retained the correct semantic hash filtering but
paid for physical whole-collection work:

- ordinary `Map` search iterated `keyedSnapshot()`;
- ordinary/structured `IdentityMap` search iterated a copied keyed snapshot;
- structured Map read lookup materialized a stable association snapshot;
- structured Map initial-definition, `atPut`, and `remove` copied
  `map.keyedSnapshot()`;
- each search then discarded entries whose recorded hash did not equal the
  already-computed query hash.

The implementation now preserves insertion order as the logical sequence while
maintaining a second physical authority indexed by the exact recorded
`BigInteger` hash.

This is representation/indexing work only. No observable collection rule was
changed.

## Map physical representation

`ProtosMapValue` now maintains:

```text
entries
  = logical insertion-order sequence

entriesByHash
  = exact recorded BigInteger hash -> ordered bucket of live Entry identities
```

Each bucket owns one cached read-only List view over its internal ordered entry
list. Search therefore does not allocate `List.copyOf(...)` or an unmodifiable
wrapper on every lookup.

Append establishes one `Entry` identity in both authorities:

```text
entries.add(entry)
entriesByHash[entry.recordedHash()].add(entry)
```

Replacement mutates only the value of the existing Entry. It does not replace
or relocate:

- the representative key;
- the recorded hash;
- insertion position; or
- bucket position.

Removal deletes the same Entry identity from both authorities and removes the
bucket when it becomes empty. Reinsertion is a new append and therefore appears
after remaining colliding entries and at the end of logical insertion order.

```text
MAP_ENTRY_IN_INSERTION_SEQUENCE=EXACTLY_ONCE
MAP_ENTRY_IN_HASH_BUCKET=EXACTLY_ONCE
EMPTY_HASH_BUCKET_RETAINED=NO
REPLACEMENT_REINDEXES_ENTRY=NO
REINSERTION_APPENDS=YES
```

## Ordinary Map lookup

The existing semantic query-hash authority remains unchanged.

Ordinary keyed search now executes:

```text
query hash
  -> candidatesForRecordedHash(queryHash)
  -> directed queryKey == storedKey over that exact bucket only
  -> first true candidate wins
```

The existing comparison-scope machinery remains around equality evaluation.
The search still applies the existing exact recorded-hash defense in
`matchesStoredKey`; the physical index merely prevents unrelated hashes from
becoming candidates in the first place.

```text
MAP_QUERY_HASH_COMPUTED_ONCE=PRESERVED
MAP_DIFFERENT_RECORDED_HASH_CANDIDATES=NOT_VISITED
MAP_EQUALITY_DIRECTION=QUERY_TO_STORED
MAP_FIRST_TRUE_CANDIDATE_WINS=PRESERVED
MAP_REPRESENTATIVE_KEY=PRESERVED
MAP_RECORDED_HASH=PRESERVED
MAP_INSERTION_POSITION=PRESERVED
```

## Structured Map lookup

Four resumable structured machines now select the exact bucket only after the
guest hash result has been accepted:

```text
PreparedMapReadLookupCall
PreparedMapInitialDefinition
PreparedMapAtPutCall
PreparedMapRemoveCall
```

Their former whole-Map snapshot state is replaced with:

```text
List<ProtosMapValue.Entry> candidates
candidates = map.candidatesForRecordedHash(queryHash)
```

They preserve the existing state-machine ownership for:

- guest hash suspension/resumption;
- equality suspension/resumption;
- comparison-scope enter/leave;
- cancellation/unwind;
- invalid hash/equality result handling;
- open/closed/frozen rechecks; and
- fallback preparation.

The index does not move any of those responsibilities.

```text
STRUCTURED_MAP_READ_STABLE_SNAPSHOT=REMOVED_FROM_SEARCH
STRUCTURED_MAP_INITIAL_DEFINITION_FULL_COPY=REMOVED
STRUCTURED_MAP_ATPUT_FULL_COPY=REMOVED
STRUCTURED_MAP_REMOVE_FULL_COPY=REMOVED
COMPARISON_SCOPE_MACHINERY=PRESERVED
SUSPENSION_RESUMPTION=PRESERVED
CANCELLATION_UNWIND=PRESERVED
```

## IdentityMap physical representation

`ProtosIdentityMapValue` analogously maintains:

```text
entriesByIdentityHash
  = exact recorded identity BigInteger hash -> ordered bucket
```

Candidate resolution remains exclusively:

```text
ProtosIdentity.identical(queryKey, storedKey)
```

The implementation does not use Java `IdentityHashMap`, Java reference
equality, or ordinary Java `equals` to resolve IdentityMap key collisions.

Absent ordinary `IdentityMap.atPut` now computes the semantic identity hash
once, uses that value for candidate selection, and reuses the same value when
appending the new entry.

```text
IDENTITYMAP_EXACT_RECORDED_IDENTITY_HASH_BUCKET=YES
IDENTITYMAP_COLLISION_TEST=PROTOS_IDENTITY_IDENTICAL
JAVA_IDENTITY_HASH_MAP_USED=NO
JAVA_REFERENCE_IDENTITY_AS_KEY_SEMANTICS=NO
IDENTITYMAP_ATPUT_IDENTITY_HASH_REUSED=YES
```

## Snapshot boundary deliberately unchanged

This slice removes snapshots from productive keyed search only.

Genuine logical/stability snapshots remain unchanged for surfaces that require
them, including:

```text
Map.each
IdentityMap.each
Map.match stableSnapshot
associationSnapshot
transfer/copy/render paths
```

Array and Bytes representation is outside this slice and unchanged.

```text
MAP_EACH_SNAPSHOT_REPRESENTATION=UNCHANGED
IDENTITYMAP_EACH_SNAPSHOT_REPRESENTATION=UNCHANGED
MAP_MATCH_STABLE_SNAPSHOT=UNCHANGED
ARRAY_BYTES=UNCHANGED
```

## Structural regression evidence

New test:

```text
ProtosPerf025MapPhysicalIndexTest
```

It freezes:

- exact hash-bucket candidate selection;
- insertion order among colliding entries;
- replacement without key/hash/order relocation;
- removal from both authorities;
- remove/reinsert append behavior;
- reuse of the bucket view instead of per-lookup copied search snapshots;
- semantic IdentityMap identity across distinct Java objects;
- absence of `keyedSnapshot()` search in ordinary Map/IdentityMap protocols;
- absence of structured `List.copyOf(map.keyedSnapshot())`;
- absence of structured read `stableSnapshot(map)`; and
- presence of exact-bucket selection in all four structured Map machines.

The focal test was run by the maintainer and reported PASS.

## Affected semantic regression

The maintainer then ran the bounded affected set including:

```text
ProtosPerf025MapPhysicalIndexTest
ProtosStandardMapProtocolTest
ProtosIdentityMapConformanceTest
ProtosPerf006Plat028MapReadLookupCallbackTest
ProtosPerf006Plat028MapAtPutCallbackTest
ProtosPerf006Plat028MapRemoveCallbackTest
ProtosMapConstructionFPrimeProtocolTest
ProtosMapConstructionFPrimeBytecodeFoundationTest
ProtosMapMatchProtocolTest
ProtosPerf006Plat028MapEachCallbackTest
ProtosPerf006Plat028IdentityMapEachCallbackTest
ProtosPerf026D2AssociationEachInlineCallbackTest
```

Reported evidence included:

```text
I049_IDENTITY_MAP_AT_IF_ABSENT_FALLBACK_SUSPEND_RESUME=PASS
I049_IDENTITY_MAP_AT_IF_ABSENT_FALLBACK_REPLAY=NO
PERF006_PLAT028_IDENTITY_MAP_EACH_SNAPSHOT=PASS
PERF006_PLAT028_MAP_ATPUT_EQUALITY_SUSPENSION=PASS
PERF006_PLAT028_MAP_ATPUT_EQUALITY_DIRECTION=QUERY_TO_STORED
PERF006_PLAT028_MAP_ATPUT_REPRESENTATIVE_KEY_RETAINED=YES
PERF006_PLAT028_MAP_ATPUT_HASH_SINGLE_INVOCATION=PASS
PERF006_PLAT028_MAP_ATPUT_RECORDED_HASH_REUSE=PASS
PERF006_PLAT028_MAP_EQUALITY_DIRECTION=QUERY_TO_STORED
PERF006_PLAT028_MAP_RECORDED_HASH_FILTER=PASS
PERF006_PLAT028_MAP_HASH_SINGLE_INVOCATION=PASS
PERF006_PLAT028_MAP_REMOVE_EQUALITY_DIRECTION=QUERY_TO_STORED
PERF006_PLAT028_MAP_REMOVE_RECORDED_HASH_FILTER=PASS
PERF006_PLAT028_MAP_REMOVE_HASH_SINGLE_INVOCATION=PASS
PERF006_PLAT028_MAP_REMOVE_COMPARISON_SCOPE_LEAK=NO
PERF026_D2_MAP_ASSOCIATION_SNAPSHOT_PRESERVED=YES
PERF026_D2_MAP_INSERTION_ORDER_PRESERVED=YES
PERF026_D2_IDENTITY_MAP_ASSOCIATION_SNAPSHOT_PRESERVED=YES
PERF026_D2_IDENTITY_MAP_INSERTION_ORDER_PRESERVED=YES
PERF026_D2_SUSPENSION_RESUMPTION_PRESERVED=YES
PERF026_D2_COMPLETED_PREFIX_REPLAY=NO
```

No failing affected regression was reported.

## Concurrent-main revalidation

During finalization, `main` advanced from the original implementation base to:

```text
16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb
PERF025: make object model generality pay as used
```

That intervening publication touched `ProtosBytecodeRootNode.java`, but only in
the ordinary `ReadMember` region. The Map state-machine edits remained
non-overlapping.

The maintainer revalidated on the new exact base before version/changelog
materialization:

```text
STRUCTURED_EXACT_BUCKET_SELECTION_SITES=4
STRUCTURED_FULL_SCAN_COPY_OCCURRENCES=0
STRUCTURED_READ_STABLE_SNAPSHOT_OCCURRENCES=0
GIT_DIFF_CHECK=PASS
AFFECTED_REGRESSION=PASS
```

The publication version was consequently advanced from the already-consumed
`0.3.160-SNAPSHOT` to `0.3.161-SNAPSHOT`.

## Integrated validation and publication

After required publication metadata was materialized, the maintainer ran:

```text
make test
```

and reported:

```text
make test=PASS
FULL_VALIDATION=PASS
```

The maintainer then committed and pushed:

```text
COMMIT=98680d056659d475bcfec85528e914bfec09209e
COMMIT_SUBJECT=PERF025: index Map and IdentityMap by recorded hash
PUSH=16432b0b..98680d05 HEAD -> main
PUBLICATION_PUSH=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_FOCAL_VALIDATION=PASS
MAINTAINER_REPORTED_AFFECTED_VALIDATION=PASS
MAINTAINER_REPORTED_FULL_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

## Exact product delta

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardIdentityMapProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardMapProtocol.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosIdentityMapValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosMapValue.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025MapPhysicalIndexTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.161-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Performance-claim boundary

No benchmark, timing comparison, JFR campaign, allocation profile, or
exact-revision performance measurement was performed for this slice.

The durable claim is structural:

```text
MAP_LINEAR_FULL_SCAN_FOR_DIFFERENT_HASHES=REMOVED
MAP_SEARCH_KEYEDSNAPSHOT_COPY=REMOVED
IDENTITYMAP_LINEAR_FULL_SCAN_FOR_DIFFERENT_HASHES=REMOVED
IDENTITYMAP_SEARCH_KEYEDSNAPSHOT_COPY=REMOVED
INSERTION_ORDER=PRESERVED
RECORDED_HASH_SEMANTICS=PRESERVED
DIRECTED_EQUALITY=PRESERVED
IDENTITY_SEMANTICS=PRESERVED
REPRESENTATIVE_KEY=PRESERVED
SINGLE_QUERY_HASH=PRESERVED
OPEN_CLOSED_FROZEN=PRESERVED
COMPARISON_SCOPE_MACHINERY=PRESERVED
EACH_SNAPSHOT_REPRESENTATION=UNCHANGED
ARRAY_BYTES=UNCHANGED
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## PERF025 status

This bounded physical-indexing slice is complete. PERF025 itself remains open
for the continuing post-F1 pay-as-you-grow line.

```text
PERF025_MAP_IDENTITYMAP_EXACT_RECORDED_HASH_INDEXING=COMPLETE
PERF025_STATUS=OPEN
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@98680d056659d475bcfec85528e914bfec09209e`;
- the exact one-commit comparison against
  `16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb`;
- the published `0.3.161-SNAPSHOT` changelog/version delta;
- `ProtosPerf025MapPhysicalIndexTest`;
- PERF025 / `guillermomolina/protos#758`;
- the existing PLAT028/I049/F-prime/Map.match/PERF026-D2 regression authority;
  and
- maintainer-reported focal, affected, revalidation, and integrated validation
  outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
