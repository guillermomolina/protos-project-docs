# I057 — ByteRegion and writable parallel-range removal implementation evidence

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I057
PROTOS_ISSUE=guillermomolina/protos#660
DECISION_AUTHORITY=D161/guillermomolina/protos#626
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
replace the normative Protos specification or the live GitHub Issue state.

## Product publication

The I057 implementation is published on `main` at:

~~~text
PROTOS_REVISION=97260be7f62cbfceb3eb173a9062226d86d00b7a
PROTOS_PARENT=f591376db0b8a7168f5ef43eb60479c550d80874
COMMIT_SUBJECT=I057: remove ByteRegion and writable parallel-range reservations (D161)
IMPLEMENTATION_VERSION=0.3.176-SNAPSHOT
SPECIFICATION_REVISION=0.1.439
~~~

The publication changes 35 files with 166 insertions and 887 deletions.

It deletes:

~~~text
src/main/java/com/guillermomolina/protos/runtime/ProtosByteRegionValue.java
src/test/java/com/guillermomolina/protos/runtime/ProtosBytesReservationPayAsYouGrowTest.java
~~~

and adds the negative Core-surface conformance guard:

~~~text
protos/tests/conformance/core-surface/removed-byte-region-surface.protos
~~~

## D161 Candidate B implementation

Exact-revision inspection confirms the selected D161 removal boundary.

Removed:

~~~text
Bytes.parallelRange
ByteRegion runtime value family
ByteRegion.parallelRange
ordinary Bytes reservation state
tryReserve / releaseReservation / commitReserved
hasReservation / isIndexReserved
parallel-region commit-on-publication machinery
parallel-region release-on-cancel machinery
ParallelRegionOverlap
ParallelRegionInUse
ParallelRegionOutsideP
ByteRegion-specific Actor transfer exclusions
ByteRegion-specific detached-execution exclusions
ByteRegion-specific diagnostic projection
ByteRegion-specific interop expectations
~~~

Retained:

~~~text
ordinary Bytes construction / size / at / atPut / add / removeAt
Closure.parallel / isolated P
Array parallel operations
P snapshot/result transfer isolation
Future cancellation
Actor transfer rules
safe implementation-invisible physical sharing
absence of arbitrary shared mutable Protos memory
~~~

No replacement Buffer/Region API, typed/native buffer model, zero-copy public
semantics, borrow/linear/affine model, generic writable Array/object partition,
compatibility alias, or replacement `parallelRange` spelling was introduced.

## Normative and guide reconciliation

The same product publication advances the live specification to `0.1.439`.

The normative delta removes the writable byte-region institution from the
isolated-P model and removes the three mechanism-specific Core Error prototypes.
The informative abstract runtime and active user guide are reconciled to the
same boundary.

The implementation changelog advances to `0.3.176-SNAPSHOT` and records the
D161/I057 removal.

Historical changelog archives remain historical evidence and are not rewritten.

## Validation provenance

The maintainer published the exact product revision to `main` and subsequently
reported:

~~~text
MAKE_TEST=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

The complete command output and exact local test counts were not supplied and
are therefore not invented here.

The push-triggered GitHub Actions run for the same exact product revision is:

~~~text
CI_RUN_ID=37138406665
CI_RUN_NUMBER=2108
CI_WORKFLOW=CI
CI_EVENT=push
CI_PRODUCT_REVISION=97260be7f62cbfceb3eb173a9062226d86d00b7a
CI_CONCLUSION=FAILURE
~~~

The remote Java test execution itself completed successfully before the policy
guard:

~~~text
PARALLEL_JAVA_TESTS=2403
PARALLEL_JAVA_FAILURES=0
PARALLEL_JAVA_ERRORS=0
PARALLEL_JAVA_SKIPPED=1
PARALLEL_JAVA_BUILD=SUCCESS

SERIAL_JAVA_TESTS=7
SERIAL_JAVA_FAILURES=0
SERIAL_JAVA_ERRORS=0
SERIAL_JAVA_BUILD=SUCCESS
~~~

The CI failure was emitted afterward by `tools/java_slow_test_guard.py`:

~~~text
JAVA_SLOW_TEST_GUARD=FAIL
THRESHOLD_SECONDS=10
UNALLOWLISTED_SLOW_TESTS=2
OVER_BUDGET_SLOW_TESTS=5
~~~

The reported slow-test owners were unrelated test surfaces; the I057 patch does
not modify those test classes. This is sufficient to distinguish the remote
failure from an I057 semantic/test assertion failure, but it does not make the
required CI gate green.

## Closure checkpoint

~~~text
PRODUCT_COMMIT=PASS
PRODUCT_PUSH=PASS
I057_PRODUCT_IMPLEMENTATION=COMPLETE
LOCAL_FULL_VALIDATION=PASS
REMOTE_JAVA_ASSERTIONS=PASS
REMOTE_CI_GREEN=NO
REMOTE_CI_FAILURE_CLASS=JAVA_SLOW_TEST_GUARD
DURABLE_EVIDENCE_PUBLISHED=YES
NATIVE_PARENT_DECLARED=guillermomolina/protos#626
NATIVE_PARENT_RELATION=ABSENT
SECOND_I057_PRODUCT_SLICE_IDENTIFIED=NO
I057_CLOSURE_AUTHORIZED=NO
I057_STATUS=OPEN_PENDING_CI_AND_NATIVE_PARENT_RECONCILIATION
~~~

Issue #660 explicitly requires required CI green before closure. Live GitHub
inspection also shows that its textual parent declaration points to #626 while
the native parent/sub-issue relation is absent. Current coordination policy
requires that native relation for formal closures created after the hierarchy
enforcement instant.

Therefore this record intentionally does not claim final closure. Once the exact
published candidate has a qualifying green CI result and the native parent
relation to #626 is reconciled, the live Issue can receive its final closure
comment and close completed without a new I057 product implementation slice
unless new evidence exposes an actual product defect.

## Final closure review — 2026-10-05

A final falsifying closure review was performed against the then-current product
`main` after the two gates recorded above had changed state.

The reviewed product state was:

~~~text
I057_IMPLEMENTATION_REVISION=97260be7f62cbfceb3eb173a9062226d86d00b7a
CURRENT_MAIN_HEAD=a153b4198da1c5d74be857cb7e24a11b1f7c53c6
IMPLEMENTATION_ANCESTRY=PASS
AHEAD_BY=41
BEHIND_BY=0
MERGE_BASE=97260be7f62cbfceb3eb173a9062226d86d00b7a
~~~

GitHub's compare relation therefore confirms that the published I057
implementation remains in the ancestry of current `main`.

The required product CI gate is now green for that exact current HEAD:

~~~text
CI_WORKFLOW=CI
CI_RUN_ID=37285625766
CI_RUN_NUMBER=2151
CI_EVENT=push
CI_HEAD=a153b4198da1c5d74be857cb7e24a11b1f7c53c6
CI_STATUS=completed
CI_CONCLUSION=success
CI_TEST_JOB=success
~~~

The native GitHub Issue hierarchy is also reconciled. Issue
`guillermomolina/protos#660` has native parent
`guillermomolina/protos#626`, and #626 reports #660 as a native sub-issue.

The final repository review found no reintroduction of the removed I057
institution. In current product/runtime code:

- `ProtosByteRegionValue` remains absent;
- Standard `Bytes` does not install `parallelRange`;
- `ProtosBytesValue` carries no writable-range reservation state;
- the removed reservation/publication helpers and
  `ProtosFutureValue.resolveWithCommit` remain absent;
- `ParallelRegionOverlap`, `ParallelRegionInUse`, and
  `ParallelRegionOutsideP` remain absent from the active Core taxonomy; and
- the retained negative conformance test continues to require
  `Bytes.parallelRange`, `ByteRegion`, and the three retired error bindings
  to be ordinary missing slots.

References that remain in historical changelogs, negative tests, descriptive
documentation, or unrelated resource/lexical reservation mechanisms are not
product reintroductions.

The retained guarantees remain represented in current code, tests, and
specification: ordinary `Bytes`, isolated `Closure.parallel`, P input
snapshot/result transfer isolation, Future cancellation, and Actor transfer.

After that review the maintainer additionally reported the current local test
validation green:

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
MAINTAINER_REPORT="Todos los tests han pasado en local"
~~~

Final closure state:

~~~text
I057_PRODUCT_COMPLETE=YES
I057_NATIVE_PARENT=PASS
I057_REQUIRED_CI=PASS
I057_REGRESSION_CHECK=PASS
I057_RETAINED_GUARANTEES=PASS
I057_CLOSURE_AUTHORIZED=YES
SECOND_I057_PRODUCT_SLICE_IDENTIFIED=NO
NEXT_IMPLEMENTATION_SLICE=NONE
I057_STATUS=READY_TO_CLOSE
~~~

No new Buffer/Region/ownership design, optional cleanup, or future writable
partitioning facility is made part of I057 by this closure review.
