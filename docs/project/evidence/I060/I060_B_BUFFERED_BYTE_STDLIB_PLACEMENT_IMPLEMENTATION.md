# I060-B — buffered byte Standard Library placement implementation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I060
IMPLEMENTATION_SLICE=I060-B
PROTOS_ISSUE=guillermomolina/protos#663
DECISION_AUTHORITY=D167/guillermomolina/protos#637
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not replace
the normative Protos specification, D167, or the live GitHub Issue state.

## Exact published product state

~~~text
PROTOS_REVISION=b118fa5540deb1ddebfe1bdec58489b80cbb0622
COMMIT_SUBJECT=I060-B: move buffered byte wrappers to std:io
IMPLEMENTATION_VERSION=0.3.217-SNAPSHOT
SPECIFICATION_REVISION=0.1.444
PRODUCT_PUBLICATION=PUSHED
~~~

The exact revision is the Protos HEAD observed at this checkpoint.

## Implemented outcome

D167 Candidate D is implemented.

~~~text
D167_CORE_BUFFEREDREADER_BINDING_REMOVAL=IMPLEMENTED
D167_CORE_BUFFEREDWRITER_BINDING_REMOVAL=IMPLEMENTED
D167_STDLIB_BUFFEREDREADER=IMPLEMENTED
D167_STDLIB_BUFFEREDWRITER=IMPLEMENTED
GENERAL_STANDARD_MODULE_MEMBER_SEAM_REUSED=YES
GENERAL_TRANSFER_STANDARD_MEMBER_IDENTITY=IMPLEMENTED
D117_CONTRACT_CHANGED=NO
PLAT031_CONTRACT_CHANGED=NO
NORMATIVE_SPEC_RECONCILIATION=IMPLEMENTED
~~~

The former Core sources were moved to the Standard Library locations:

~~~text
protos/lib/core/BufferedReader.protos
    -> protos/lib/io/BufferedReader.protos

protos/lib/core/BufferedWriter.protos
    -> protos/lib/io/BufferedWriter.protos
~~~

GitHub records both moves as renames. Their former source-level factory
declarations were removed: the new `std:io` module files contain only the
placement explanation and do not redeclare the factory identities.

`protos/lib/core/prelude.protos` no longer publishes `BufferedReader` or
`BufferedWriter`. The Core surface conformance test now requires ordinary
`SlotNotFound` for both bare names.

The public Standard Library locations are:

~~~text
std:io/BufferedReader.BufferedReader
std:io/BufferedWriter.BufferedWriter
~~~

## Runtime factory placement

`ProtosCoreBootstrap` no longer loads the two factories as Core source-owned
objects. It asks `ProtosStandardBufferedByteIoProtocol` to create ordinary
direct children of `Object`, installs the existing reader/writer factory
protocol on those objects, freezes them with the standard graph, and registers
them through the already-general standard-module initial-member seam.

The registration is:

~~~text
std:io/BufferedReader
    BufferedReader -> exact frozen runtime-owned reader factory

std:io/BufferedWriter
    BufferedWriter -> exact frozen runtime-owned writer factory
~~~

The generic module lifecycle remains unaware of BufferedReader and
BufferedWriter. No buffered-specific resolver, loader, module cache or module
identity machinery was added.

## Identity model

The implementation retains the selected identity model:

~~~text
module instance
    Actor-local

factory
    exact frozen runtime-owned standard object
    shared across Actor-local module instances

wrapper
    fresh per successful call/owning construction
    bound to the caller's execution domain
~~~

The new `ProtosStandardBufferedByteIoPlacementTest` covers Actor-local module
identity, shared exact factory identity and fresh wrapper construction.

## General transfer refinement

I060-A identified that removal from Prelude would otherwise make the factories
lose the shared-standard classification they previously obtained from being
Prelude bindings.

I060-B closes that gap generically.

`ProtosPrelude` now precomputes an identity set over all registered
`standardModuleMembers` and exposes:

~~~text
isStandardModuleMemberForRuntime(candidate)
~~~

as an exact-identity predicate.

The existing transfer paths now use that general predicate:

~~~text
ProtosActorValueTransfer
ProtosParallelRuntime
ProtosDetachedExecutionValue
~~~

The former IP-family-only transfer predicate is no longer the classification
used by those paths. As a result, every registered frozen standard-module member
can remain an exact shared standard anchor without importing its module.

No BufferedReader/BufferedWriter-specific branch or runtime getter was added to
the transfer machinery.

The placement regression covers:

~~~text
ACTOR_TRANSFER_EXACT_FACTORY_IDENTITY
DETACHED_TRANSFER_EXACT_FACTORY_IDENTITY
P_TRANSFER_EXACT_FACTORY_IDENTITY
TRANSFER_CAUSES_NO_MODULE_IMPORT
TRANSFER_CAUSES_NO_MODULE_SOURCE_EXECUTION
~~~

with resolver-counting/state checks rather than timing races.

## D117 and PLAT031 preservation

The product diff does not rewrite the buffered byte state machines.

`ProtosBufferedByteIo` changes only its placement description from Core to
`std:io`.

The existing D117/PLAT031 machinery remains the implementation path for:

~~~text
reader ordering and read-ahead
writer buffering and flush frontier
pre-commit cancellation
post-commit aftermath
ZERO / KNOWN / UNKNOWN delegated effects
no replay after unknown lower effect
close cutover
borrowing versus owning
owned lower close
reader C-prime
writer C-prime
release C-prime
~~~

Existing protocol and PERF006 PLAT031 tests were retained and migrated to obtain
the factories through the standard-module-member seam.

## Normative reconciliation

Specification revision `0.1.444` updates `spec/io/IO_CORE.md` so the buffered
byte wrappers are no longer described as Core/Prelude bindings.

The normative public placement is now:

~~~text
std:io/BufferedReader.BufferedReader
std:io/BufferedWriter.BufferedWriter
~~~

reachable through explicit import.

The same revision explicitly preserves the borrowing/owning factory forms,
fresh wrapper construction, ownership rules and the complete buffering, D117
and PLAT031 contracts.

Historical specification changelogs are not rewritten.

## Product files materially changed

The exact implementation commit changes:

- `CHANGELOG.md`;
- `pom.xml`;
- `protos/lib/core/prelude.protos`;
- renamed `protos/lib/core/BufferedReader.protos` to
  `protos/lib/io/BufferedReader.protos`;
- renamed `protos/lib/core/BufferedWriter.protos` to
  `protos/lib/io/BufferedWriter.protos`;
- `protos/tests/conformance/core-surface/required-core-bindings.protos`;
- `spec/PROTOS_SPEC_CHANGELOG.md`;
- `spec/io/IO_CORE.md`;
- `ProtosCoreBootstrap`;
- `ProtosDetachedExecutionValue`;
- `ProtosParallelRuntime`;
- `ProtosStandardBufferedByteIoProtocol`;
- `ProtosActorValueTransfer`;
- `ProtosBufferedByteIo`;
- `ProtosPrelude`;
- Core/bootstrap/native-boundary/source-naming tests;
- PERF006 reader/writer C-prime tests;
- `ProtosStandardBufferedByteIoPlacementTest`;
- `ProtosStandardBufferedByteIoProtocolTest`;
- `ProtosStandardModuleMemberTestSupport`.

## Validation and publication evidence

The maintainer reported after the implementation was pushed:

~~~text
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PRODUCT_PUBLICATION=PUSHED
~~~

The GitHub commit status / workflow-run interfaces available at this checkpoint
do not expose a push-triggered CI result for the exact SHA:

~~~text
CI_HEAD=b118fa5540deb1ddebfe1bdec58489b80cbb0622
COMBINED_STATUS_ENTRIES=0
COMMIT_WORKFLOW_RUNS_EXPOSED=0
PUBLICATION_VALIDATION=NOT_YET_CLAIMED
~~~

This absence is not treated as CI failure. It means the Issue's explicit
`PUBLICATION_VALIDATION=PASS` closure gate has not yet been independently
established in this checkpoint.

## Closure-gate state

Based on the published diff plus maintainer-reported local validation:

~~~text
D167_CORE_BINDING_REMOVAL=PASS
D167_STDLIB_SURFACE=PASS
D117_CONTRACT_PRESERVED=PASS
PLAT031_CONTRACT_PRESERVED=PASS
NORMATIVE_SPEC_RECONCILIATION=PASS
FOCUSED_VALIDATION=PASS
REQUIRED_FULL_VALIDATION=PASS
PUBLICATION_VALIDATION=PENDING_EVIDENCE
~~~

Therefore I060-B is technically implemented and published, but I060/#663 should
remain open until a final closure review proves the publication gate and
reconciles any post-I060-B HEAD movement.

## Coordination state

~~~text
I060_B_IMPLEMENTATION=COMPLETE
NEXT_IMPLEMENTATION_SLICE=NONE
I060_CLOSURE_CANDIDATE=YES
I060_CLOSURE_AUTHORIZED=NO
ISSUE_663_STATE=REVIEW
NEXT_SLICE=I060-C
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_ACTIVITY=FINAL_CLOSURE_REVIEW
~~~

I060-C must be read-only: inspect current GitHub repository/Issue/CI state,
confirm that `b118fa5540deb1ddebfe1bdec58489b80cbb0622` remains an ancestor of
current HEAD, falsify any relevant post-I060 regression, verify the exact-sha
publication-validation result when observable, and close #663 only if every
closure gate is PASS.

AI assistance: this durable checkpoint was drafted with ChatGPT from the exact
published Protos commit, the live I060/D167 GitHub authority, and
maintainer-reported local validation.
