# I063 — standard File append removal implementation evidence

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I063
PROTOS_ISSUE=guillermomolina/protos#666
DECISION_AUTHORITY=D170/guillermomolina/protos#641
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
replace the normative Protos specification or the live GitHub Issue state.

## Product publication

The bounded I063 implementation is published at:

~~~text
PROTOS_REVISION=15a0440d578a672759f6d982e73dabecfe61e1ab
PROTOS_PARENT=63df450263feb5bfbfb3160aecf33128f456e08c
COMMIT_SUBJECT=I063: remove standard File append semantics
IMPLEMENTATION_VERSION=0.3.173-SNAPSHOT
SPECIFICATION_REVISION=0.1.438
~~~

The publication changes 25 files, with 185 insertions and 769 deletions. It
removes the append-specific conformance test
`ProtosFileAppendConformanceTest.java`.

## D170 Candidate C implementation

Exact-revision inspection confirms the selected D170 boundary.

Removed from the standard File institution:

~~~text
append open option
append-specific File mode / write-placement dimension
ProtosFileFlow.AppendWritableResource
ProtosFileFlow.AppendCompletion
append-only write path
cross-File / cross-alias append-placement contract
Standard Library Files helper append:false forwarding
append-specific conformance machinery
~~~

Retained:

~~~text
ordinary File read/write/close
internal logical File position
ByteSeekable: position / seek / seekBy / seekToEnd
ByteSized: size
Truncatable: truncate
Syncable: sync
truncate-on-open
positioned writes
generic I/O commitment/cancellation/lifecycle
backend-honest optional capability exposure
~~~

At this revision `ProtosFileFlow` has one positioned `WritableResource.writeAt`
path and no append-specific resource/completion interface. Its capability shape
continues to include readable, writable, seekable, sized, truncatable and
syncable state only.

The retained File protocol regression coverage includes ordinary positioned
read/write semantics and an explicit
`seekToEndThenWriteIsAnOrdinaryPositionedWriteAtTheObservedEnd` case, which
does not redefine that composition as atomic append.

## Normative reconciliation

The same product publication advances the normative specification to
`0.1.438`.

The specification now records that:

- standard `filesystem.open` has exactly the ordinary read/write/create/
  createNew/truncate option surface and no append option;
- every writable standard File uses positioned writes;
- `seekToEnd()` followed by `write(bytes)` is ordinary two-operation
  composition on one receiver, not atomic append;
- no cross-File/cross-alias placement or non-interleaving guarantee is attached
  to that composition;
- ByteSeekable, ByteSized, Truncatable and Syncable remain unchanged.

The implementation changelog advances to `0.3.173-SNAPSHOT` and records the
same D170 boundary.

## Publication completeness

Exact revision inspection establishes:

~~~text
APPEND_OPEN_MODE_REMOVED=YES
APPEND_FILE_MODE_REMOVED=YES
APPEND_WRITABLE_RESOURCE_REMOVED=YES
APPEND_COMPLETION_REMOVED=YES
APPEND_WRITE_PATH_REMOVED=YES
APPEND_CROSS_ALIAS_CONTRACT_REMOVED=YES

BYTE_SEEKABLE_PRESERVED=YES
BYTE_SIZED_PRESERVED=YES
TRUNCATABLE_PRESERVED=YES
SYNCABLE_PRESERVED=YES
TRUNCATE_ON_OPEN_PRESERVED=YES
POSITIONED_WRITE_SEMANTICS_PRESERVED=YES

REPLACEMENT_APPEND_FACADE_ADDED=NO
SECOND_IMPLEMENTATION_SLICE_IDENTIFIED=NO
~~~

GitHub code-search results observed while preparing this record still contained
stale pre-publication append fragments. Exact file reads at the immutable
`PROTOS_REVISION` above were used instead as publication authority.

## Validation and closure status

The maintainer supplied successful commit and push evidence for the exact
product revision and subsequently reported that **all tests pass** for the
published candidate.

Repository policy requires one FULL validation for closure of a top-level
executable item. The maintainer's summarized PASS is accepted by the
human-executor contract; the exact command output and test count were not
supplied and are therefore not invented here.

This was a maintainer-direct publication to `main`, not a Pull Request.
Repository policy assigns remote PR CI to the PR contribution path; GitHub
reported no commit-status/workflow identity for this direct-main SHA.
Accordingly there is no separate PR-CI gate to wait for on this publication.

Read-only inspection of every modified Protos-owned production source file at
the exact revision confirms that the existing APL-1.0 notice remains present.
No new source file was introduced.

~~~text
PRODUCT_COMMIT=PASS
PRODUCT_PUSH=PASS
FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED_ALL_TESTS_PASS
EXACT_TEST_COUNT=NOT_REPORTED
PR_CI_GATE=NOT_APPLICABLE_DIRECT_MAIN
LICENSE_COMPLIANCE=PASS
I063_CLOSURE_AUTHORIZED=YES
~~~

No further product implementation slice is identified by the published D170
delta.

~~~text
I063_PRODUCT_IMPLEMENTATION=COMPLETE
I063_TECHNICAL_SLICE_PENDING=NO
I063_STATUS=CLOSED_COMPLETED
FINAL_PRODUCT_REVISION=15a0440d578a672759f6d982e73dabecfe61e1ab
~~~
