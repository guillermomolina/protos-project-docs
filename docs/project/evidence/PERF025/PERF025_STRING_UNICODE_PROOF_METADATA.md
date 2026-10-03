# PERF025 — String Unicode proof metadata preservation

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the proof-preserving Core String representation change, the affected runtime-copy
surfaces, and maintainer-reported validation. It does not claim a measured
performance magnitude.

## Exact product publication

```text
PROTOS_REVISION=e584c2345a2327eb0952c198905436e0e2205cf2
PROTOS_PARENT_REVISION=305c81ca7994a0fb98589751303cd8ca72b8b2da
PROTOS_VERSION=0.3.163-SNAPSHOT
COMMIT_SUBJECT=PERF025: preserve String Unicode proof metadata
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
COMMITS=1
FILES_CHANGED=10
ADDITIONS=275
DELETIONS=40
```

The product publication is exactly one commit ahead of the preceding
`0.3.162-SNAPSHOT` Bytes/ByteRegion reservation-coordination publication.

## Bounded objective

The retained post-F1 PERF025 audit identified a String-specific cost surface:
already-valid semantic Strings repeatedly forgot facts that had already been
proved.

Before this slice:

- arbitrary host `String` ingress validated the complete UTF-16 representation
  for unpaired surrogates;
- `String.size` rescanned the complete value with `codePointCount(...)`;
- `String.at` extracted one already-valid scalar and then reconstructed a new
  `ProtosStringValue`, causing that one-scalar result to be validated again;
- binary `String +` concatenated two already-valid semantic Strings and then
  rescanned the complete derived result through the public validating
  constructor;
- variadic `String.concat` likewise rebuilt and rescanned its complete result;
  and
- Actor transfer, isolated-P transfer, detached execution and Test Tool discovery
  copied already-valid Strings by reconstructing from `.value()`, forgetting
  their established invariant.

The implementation keeps arbitrary host ingress validating while retaining and
composing the proof for derived semantic String values.

## Validating ingress remains authoritative

The public constructor remains the untrusted host-text boundary.

It still:

- rejects an unpaired high surrogate;
- rejects an unpaired low surrogate;
- accepts exact supplementary Unicode scalars represented as surrogate pairs;
- performs no normalization; and
- establishes the immutable semantic String value.

The same validating traversal now also derives the exact Unicode scalar count:

```text
PUBLIC_HOST_STRING_INGRESS_VALIDATES=YES
PUBLIC_HOST_STRING_INGRESS_COUNTS_SCALARS_IN_SAME_PASS=YES
UNPAIRED_SURROGATE_REJECTION=PRESERVED
UNICODE_NORMALIZATION=NONE
```

No general unchecked raw-host-`String` construction API was introduced. The
proof-preserving `(String, scalarCount)` constructor is private to
`ProtosStringValue`.

## Cached scalar cardinality

`ProtosStringValue` now retains:

```text
private final String value;
private final int scalarCount;
```

`scalarCountForRuntime()` exposes the already-proven derived cardinality to
runtime code.

Standard `String.size` now returns that value directly instead of rescanning
the complete UTF-16 representation with `String.codePointCount(...)`.

The cached count follows the semantic scalar model:

- empty String -> 0;
- BMP scalar -> 1;
- supplementary scalar -> 1 despite two UTF-16 code units;
- combining marks remain independent scalars; and
- grapheme-cluster boundaries are not introduced.

## Proof-preserving scalar extraction

`scalarAtForRuntime(index)` indexes only an already-valid
`ProtosStringValue`.

For an in-range scalar it returns a new semantic String using the private
proof-preserving constructor with exact scalar count 1. Standard `String.at`
therefore no longer sends the one-scalar substring back through the arbitrary
host-text validation boundary.

Out-of-range behavior remains owned by the standard String protocol and still
signals the existing ordinary guest Error.

```text
STRING_AT_SCALAR_SEMANTICS=PRESERVED
SUPPLEMENTARY_SCALAR_COUNTS_AS_ONE=PRESERVED
OUT_OF_RANGE_ERROR_SEMANTICS=PRESERVED
DERIVED_SCALAR_REVALIDATION=REMOVED
```

## Proof-preserving concatenation

Two valid Unicode scalar sequences concatenate to another valid Unicode scalar
sequence. The scalar cardinality composes additively.

The runtime now provides proof-carrying operations that construct the derived
host text and exact count together:

```text
concatenateForRuntime(left, right)
concatenateAllForRuntime(receiver, supplied)
```

The binary result uses:

```text
resultScalarCount = left.scalarCount + right.scalarCount
```

The variadic result sums the already-proven counts of the receiver and all
supplied semantic Strings while building the output text.

Standard `String +` and `String.concat` retain their existing strict String
domain checks in `ProtosStandardStringProtocol`. No implicit conversion or
normalization was introduced.

```text
BINARY_STRING_CONCAT_FULL_RESULT_REVALIDATION=REMOVED
VARIADIC_STRING_CONCAT_FULL_RESULT_REVALIDATION=REMOVED
STRICT_STRING_DOMAIN=PRESERVED
LEFT_TO_RIGHT_ARGUMENT_EVALUATION=PRESERVED
EXACT_SCALAR_SEQUENCE_CONCATENATION=PRESERVED
```

## Runtime-copy proof preservation

`copyForRuntime()` produces a fresh physical `ProtosStringValue` wrapper
while retaining the immutable host text and exact scalar count.

The following already-valid copy/rematerialization paths now use that operation:

```text
ProtosActorValueTransfer
ProtosParallelRuntime
ProtosDetachedExecutionValue
ProtosTestLogicalCaseDiscoveryFacility
```

This preserves the existing fresh-wrapper/isolation behavior of those surfaces
without rescanning the semantic String contents.

```text
ACTOR_TRANSFER_STRING_REVALIDATION=REMOVED
PARALLEL_TRANSFER_STRING_REVALIDATION=REMOVED
DETACHED_EXECUTION_STRING_REVALIDATION=REMOVED
TEST_DISCOVERY_STRING_REVALIDATION=REMOVED
FRESH_WRAPPER_POLICY=PRESERVED
```

## Identity, equality and hashing

This slice does not change String identity/equality/hash authority.

The existing semantic String family continues to use its established exact
String value semantics. The cached scalar count is derived representation
metadata only and is not incorporated as a new guest-visible identity category.

```text
STRING_IDENTITY_SEMANTICS=PRESERVED
STRING_EQUALITY_SEMANTICS=PRESERVED
STRING_HASH_SEMANTICS=PRESERVED
NEW_GUEST_VISIBLE_STRING_CATEGORY=NO
```

## Structural regression evidence

New structural regression:

```text
ProtosPerf025StringProofPreservationTest
```

It freezes:

- absence of `codePointCount(...)` from the standard productive String size
  path;
- proof-preserving scalar extraction in standard `String.at`;
- proof-preserving binary and variadic concatenation;
- removal of validating reconstruction from Actor, P and detached-execution
  copies; and
- proof-preserving Test Tool discovery rematerialization.

Expanded `ProtosStringValueTest` freezes:

- exact cached scalar counts for empty, BMP, combining and supplementary data;
- retained unpaired-surrogate rejection;
- exact concatenated text without normalization;
- additive scalar cardinality for binary and aggregate concatenation;
- one-scalar metadata for BMP combining and supplementary extraction; and
- fresh runtime-copy wrapper with identical value/count.

## Maintainer-reported compile and focused validation

The maintainer ran:

```text
mvn -DskipTests test-compile
```

and reported:

```text
BUILD SUCCESS
```

The focused regression command covered:

```text
ProtosStringValueTest
ProtosPerf025StringProofPreservationTest
ProtosStandardStringProtocolTest
ProtosActorValueTransferTest
ProtosParallelExecutionTest
ProtosDetachedExecutionValueCrossPreludeTest
ProtosTestLogicalCaseDiscoveryFacilityTest
```

The maintainer reported:

```text
Tests run: 35
Failures: 0
Errors: 0
Skipped: 0
BUILD SUCCESS
```

## Concurrent-main reconciliation

The substantive String implementation was prepared while product HEAD was:

```text
98680d056659d475bcfec85528e914bfec09209e
0.3.161-SNAPSHOT
```

During finalization, `main` advanced to:

```text
305c81ca7994a0fb98589751303cd8ca72b8b2da
0.3.162-SNAPSHOT
PERF025: make Bytes reservation coordination pay as you grow
```

That intervening commit touched Bytes/ByteRegion implementation, their tests and
product metadata, but did not touch this slice's String implementation or
focused-test surfaces.

The local candidate was reconciled onto that exact new parent. The String slice
therefore published as:

```text
0.3.163-SNAPSHOT
```

The earlier prospective `0.3.162-SNAPSHOT` metadata was discarded and never
published for this slice.

## Final candidate checks

Before the final integrated validation, the maintainer confirmed:

```text
LOCAL_HEAD=305c81ca7994a0fb98589751303cd8ca72b8b2da
ORIGIN_MAIN=305c81ca7994a0fb98589751303cd8ca72b8b2da
GIT_DIFF_CHECK=PASS
IMPLEMENTATION_VERSION=0.3.163-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
LICENSE_TXT_PRESENT=YES
MODIFIED_SOURCE_APL_NOTICE_CHECK=PASS
```

The productive standard String protocol inspection retained no
`codePointCount(...)` or validating `new ProtosStringValue(...)` construction
for size/at/concatenation.

## Final integrated validation and publication

On the definitive final candidate, after reconciliation and metadata
materialization, the maintainer ran:

```text
make test
```

and reported:

```text
make test=PASS
```

The detailed final-suite cardinality was not retained in this interaction and is
therefore intentionally not reconstructed here.

The maintainer then committed and pushed:

```text
COMMIT=e584c2345a2327eb0952c198905436e0e2205cf2
COMMIT_SUBJECT=PERF025: preserve String Unicode proof metadata
PUSH=305c81ca..e584c234 HEAD -> main
LOCAL_AFTER_PUSH=e584c2345a2327eb0952c198905436e0e2205cf2
ORIGIN_MAIN_AFTER_PUSH=e584c2345a2327eb0952c198905436e0e2205cf2
WORKTREE_AFTER_PUSH=CLEAN
PUBLICATION_PUSH=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_TEST_COMPILE=PASS
MAINTAINER_REPORTED_FOCAL_VALIDATION=PASS
MAINTAINER_REPORTED_FULL_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

## Exact product delta

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java
M src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardStringProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosTestLogicalCaseDiscoveryFacility.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosStringValue.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025StringProofPreservationTest.java
M src/test/java/com/guillermomolina/protos/runtime/ProtosStringValueTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.163-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
```

## Performance-claim boundary

No benchmark, timing comparison, JFR campaign, allocation profile or
exact-revision performance measurement was performed for this slice.

The durable claim is structural:

```text
HOST_STRING_VALIDATION=PRESERVED
SCALAR_COUNT_CACHED=YES
STRING_SIZE_FULL_RECOUNT=REMOVED
STRING_AT_DERIVED_REVALIDATION=REMOVED
STRING_PLUS_DERIVED_REVALIDATION=REMOVED
STRING_CONCAT_DERIVED_REVALIDATION=REMOVED
ACTOR_TRANSFER_STRING_REVALIDATION=REMOVED
PARALLEL_TRANSFER_STRING_REVALIDATION=REMOVED
DETACHED_EXECUTION_STRING_REVALIDATION=REMOVED
TEST_DISCOVERY_STRING_REVALIDATION=REMOVED
STRING_IDENTITY_EQUALITY_HASH=PRESERVED
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PLATFORM_DECISION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## PERF025 status

This bounded String proof-metadata slice is complete. PERF025 itself remains
open for the continuing post-F1 pay-as-you-grow line.

```text
PERF025_STRING_UNICODE_PROOF_METADATA=COMPLETE
PERF025_STATUS=OPEN
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@e584c2345a2327eb0952c198905436e0e2205cf2`;
- its exact parent
  `305c81ca7994a0fb98589751303cd8ca72b8b2da`;
- the published `0.3.163-SNAPSHOT` version/changelog identity;
- `ProtosStringValue`;
- `ProtosStandardStringProtocol`;
- the Actor/P/detached/discovery copy paths;
- `ProtosStringValueTest`;
- `ProtosPerf025StringProofPreservationTest`;
- PERF025 / `guillermomolina/protos#758`; and
- maintainer-reported compile, focused and final integrated validation outcomes.

This record is evidence only and does not replace live GitHub Issue
coordination.
