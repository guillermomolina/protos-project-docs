# PERF025 — Bytes reservation coordination pay as you grow

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the Bytes/ByteRegion reservation-representation change, preserved P semantics,
and maintainer-reported validation. It does not claim a measured performance
magnitude.

## Exact product publication

```text
PROTOS_REVISION=305c81ca7994a0fb98589751303cd8ca72b8b2da
PROTOS_PARENT_REVISION=98680d056659d475bcfec85528e914bfec09209e
PROTOS_VERSION=0.3.162-SNAPSHOT
COMMIT_SUBJECT=PERF025: make Bytes reservation coordination pay as you grow
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
COMMITS=1
FILES_CHANGED=6
ADDITIONS=370
DELETIONS=25
```

The product publication is exactly one commit ahead of the preceding
`0.3.161-SNAPSHOT` Map/IdentityMap physical-index publication.

## Bounded objective

Before this slice, every `ProtosBytesValue` and `ProtosByteRegionValue`
eagerly allocated an empty reservation `ArrayList`, and ordinary indexed
operations were Java-`synchronized` even when the program never used
`parallelRange`.

That representation charged ordinary Bytes execution for P coordination that
was semantically relevant only while live non-empty byte-range reservations
exist.

The implementation now makes that coordination pay as it grows:

```text
NO_LIVE_RESERVATION
  -> reservations == null
  -> no reservation collection allocation
  -> ordinary indexed fast path has no synchronized method modifier

FIRST_NON_EMPTY_RESERVATION
  -> synchronized reservation slow path
  -> immutable reservation snapshot published through volatile field

MORE_DISJOINT_RESERVATIONS
  -> copy/update immutable snapshot
  -> overlap semantics unchanged

LAST_RELEASE_OR_SUCCESSFUL_COMMIT
  -> reservations = null
  -> return to ordinary storage-only representation
```

This is internal representation/synchronization work only. No observable
Bytes, ByteRegion, Future, Task, Actor, P, error, or specification semantic was
changed.

## Lazy reservation authority

Both `ProtosBytesValue` and `ProtosByteRegionValue` now use:

```text
private volatile List<R> reservations;
```

The null state is the ordinary representation.

A successful non-empty reservation creates an immutable snapshot with
`List.copyOf(...)`. Reservation mutation remains serialized in the existing
object-local slow path. Lock-free readers load one published snapshot and never
observe an in-place mutation of that list.

Zero-length reservation keeps the state absent:

```text
ZERO_LENGTH_RESERVATION_ALLOCATES_STATE=NO
```

After the last token is released, or after successful publication removes the
last reservation:

```text
RESERVATION_STATE_AFTER_LAST_RELEASE=null
RESERVATION_STATE_AFTER_LAST_COMMIT=null
```

## Ordinary fast path

The following ordinary `ProtosBytesValue` operations no longer carry a
`synchronized` method modifier:

```text
indexedSize
indexedAt
indexedPut
indexedAdd
indexedRemoveAt
indexedSnapshot
rangeSnapshot
hasReservation
isIndexReserved
```

For `ProtosByteRegionValue`, the corresponding fixed-size indexed operations
likewise remain off the universal monitor fast path.

Only reservation-changing operations remain explicitly coordinated:

```text
tryReserve
releaseReservation
commitReserved
```

This means an ordinary Bytes value that never uses P no longer allocates
reservation bookkeeping or enters a monitor merely to perform normal indexed
work.

## Snapshot slow path

`indexedSnapshot()` and `rangeSnapshot(...)` use a split path:

```text
reservations == null
  -> copy storage directly
  -> no monitor

reservations != null
  -> synchronize
  -> copy a coherent storage snapshot relative to reservation commit
```

This preserves the existing snapshot behavior required by surfaces including
`Bytes.each`, P region formation, transfer/copy paths, and recursive
`ByteRegion.parallelRange`, without imposing monitor coordination on the
ordinary no-P case.

## Ownership and publication boundary

The implementation relies on the already-existing Protos ownership model rather
than introducing new concurrency semantics.

Ordinary Actor-local Protos execution is serialized between its explicit
suspension boundaries. First reservation establishment occurs synchronously on
that owning execution path before isolated P child work can run concurrently.
The isolated child mutates its detached `ByteRegion`, not the parent Bytes
storage.

Completion of an already-active P region is the cross-carrier boundary that may
race with later owner activity. Active reservation state is published through
the volatile snapshot, while reservation-changing and commit operations remain
coordinated.

Successful parent mutation remains owned by the existing Future commitment
boundary:

```text
child succeeds
  -> transferred result accepted
  -> parent reserved bytes published
  -> reservation released
  -> Future resolves

cancellation/failure wins first
  -> reservation released
  -> parent bytes not published
```

No new blocking, waiting, suspension, lock-visible behavior, or guest memory
ordering primitive is introduced.

## P semantics preserved

The slice keeps the normative byte-range rules unchanged:

- a successful non-empty range reserves its half-open interval until terminal
  Future outcome;
- zero-length ranges reserve nothing;
- overlapping non-empty ranges still fail with `ParallelRegionOverlap`;
- parent access inside a live reserved interval still fails with
  `ParallelRegionInUse`;
- access outside live intervals remains ordinary;
- `size` remains readable;
- length/position-changing parent operations remain rejected while any
  reservation exists;
- child mutation remains isolated in `ByteRegion`;
- cancellation/failure releases without publishing child mutation;
- successful publication remains indivisible with Future successful
  terminalization.

```text
PARALLEL_REGION_OVERLAP_SEMANTICS=PRESERVED
PARALLEL_REGION_IN_USE_SEMANTICS=PRESERVED
ZERO_LENGTH_SEMANTICS=PRESERVED
DISJOINT_RESERVATION_SEMANTICS=PRESERVED
SUCCESS_COMMIT_SEMANTICS=PRESERVED
CANCELLATION_RELEASE_SEMANTICS=PRESERVED
FAILURE_RELEASE_SEMANTICS=PRESERVED
SIZE_READABILITY=PRESERVED
```

## Structural regression evidence

New test:

```text
ProtosBytesReservationPayAsYouGrowTest
```

It freezes:

- absent reservation state for fresh ordinary Bytes;
- zero-length reservation without state materialization;
- first non-empty reservation materialization;
- disjoint reservation coexistence;
- overlapping reservation rejection;
- removal of one token while another remains;
- return to null state after last release;
- successful committed-slice publication followed by null state;
- equivalent recursive ByteRegion reservation lifetime;
- absence of `synchronized` modifiers from ordinary indexed methods; and
- retention of synchronization on the three reservation-changing methods.

The pre-existing indexed-interop test was renamed to remove the obsolete claim
that the projected indexed state itself is synchronized; its read-only interop
semantics are unchanged.

## Maintainer-reported focused validation

The maintainer ran:

```text
mvn -Dtest=ProtosBytesReservationPayAsYouGrowTest,ProtosAdditionalIndexedInteropTest test
```

and reported:

```text
Tests run: 9
Failures: 0
Errors: 0
Skipped: 0
BUILD SUCCESS
```

## Maintainer-reported affected regression

The bounded affected set included:

```text
ProtosParallelExecutionTest
ProtosParallelPolyglotContextRoutingTest
ProtosActorValueTransferTest
ProtosPerf006Plat028BytesEachCallbackTest
ProtosPerf026D1IndexedEachInlineCallbackTest
ProtosPerf006C3DParallelBytecodeRematerializationTest
```

The maintainer reported:

```text
Tests run: 37
Failures: 0
Errors: 0
Skipped: 0
BUILD SUCCESS
GIT_DIFF_CHECK=PASS
```

That set covers general P execution, nested/context-routed P work, Bytes transfer
snapshot behavior, the structured Bytes.each callback path, the PERF026 inline
Bytes.each callback path, and parallel bytecode rematerialization.

## Concurrent-main reconciliation

The substantive implementation began on:

```text
16432b0baf2fe9bd8e56acbc07bbef7ca0acf2cb
```

During finalization, `main` advanced by the preceding PERF025 Map/IdentityMap
slice to:

```text
98680d056659d475bcfec85528e914bfec09209e
```

That intervening commit did not touch either Bytes runtime class or the two test
paths owned by this slice. It did consume `0.3.161-SNAPSHOT`, so the final
publication was reconciled onto the new exact parent and advanced to:

```text
0.3.162-SNAPSHOT
```

Because the fast-forward contained executable changes, the final integrated
validation was rerun on the definitive reconciled bytes after version and
CHANGELOG materialization.

## Final integrated validation and publication

On the definitive final candidate, the maintainer ran:

```text
make test
```

and reported:

```text
make test=PASS
Protos tests: 1284 passed, 0 failed
Protos tests total time: 43 s
GIT_DIFF_CHECK=PASS
```

The elapsed test-tool time is retained only as validation provenance. It is not
used as a product performance measurement.

The maintainer then committed and pushed:

```text
COMMIT=305c81ca7994a0fb98589751303cd8ca72b8b2da
COMMIT_SUBJECT=PERF025: make Bytes reservation coordination pay as you grow
PUSH=98680d05..305c81ca HEAD -> main
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
M src/main/java/com/guillermomolina/protos/runtime/ProtosByteRegionValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosBytesValue.java
M src/test/java/com/guillermomolina/protos/runtime/ProtosAdditionalIndexedInteropTest.java
A src/test/java/com/guillermomolina/protos/runtime/ProtosBytesReservationPayAsYouGrowTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.162-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Performance-claim boundary

No benchmark, timing comparison, JFR campaign, allocation profile, or
exact-revision performance measurement was performed for this slice.

The durable claim is structural:

```text
EAGER_EMPTY_RESERVATION_LIST=REMOVED
ORDINARY_BYTES_METHOD_MONITOR=REMOVED
ORDINARY_BYTEREGION_METHOD_MONITOR=REMOVED
ZERO_LENGTH_RESERVATION_BOOKKEEPING=ABSENT
ACTIVE_RESERVATION_STATE=LAZY_IMMUTABLE_SNAPSHOT
SNAPSHOT_MONITOR_SLOW_PATH=ACTIVE_RESERVATIONS_ONLY
LAST_RELEASE_RETURNS_TO_NULL_STATE=YES
LAST_COMMIT_RETURNS_TO_NULL_STATE=YES
P_SEMANTICS=PRESERVED
BYTES_EACH_SEMANTICS=PRESERVED
ACTOR_TRANSFER_SEMANTICS=PRESERVED
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## PERF025 status

This bounded Bytes/ByteRegion reservation-coordination slice is complete.
PERF025 itself remains open for the continuing post-F1 pay-as-you-grow line.

```text
PERF025_BYTES_RESERVATION_PAY_AS_YOU_GROW=COMPLETE
PERF025_STATUS=OPEN
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@305c81ca7994a0fb98589751303cd8ca72b8b2da`;
- the exact one-commit comparison against
  `98680d056659d475bcfec85528e914bfec09209e`;
- the published `0.3.162-SNAPSHOT` version/changelog identity;
- `ProtosBytesValue`;
- `ProtosByteRegionValue`;
- `ProtosBytesReservationPayAsYouGrowTest`;
- PERF025 / `guillermomolina/protos#758`;
- the existing P reservation/commit semantic authority; and
- maintainer-reported focal, affected, and final integrated validation outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
