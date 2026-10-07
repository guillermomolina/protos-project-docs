# I060-A — D167 buffered byte Standard Library migration investigation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I060
RESEARCH_SLICE=I060-A
PROTOS_ISSUE=guillermomolina/protos#663
DECISION_AUTHORITY=D167/guillermomolina/protos#637
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative investigation evidence. It does not replace
the normative Protos specification, D167, or the live GitHub Issue state.

## Exact investigated and reconciled product state

The detailed I060-A inspection was performed against:

~~~text
INVESTIGATED_PROTOS_REVISION=812b3f0b29dcba59562e3d03930c163db8529410
INVESTIGATED_PROTOS_SUBJECT=I064-A: remove public Filesystem.captureTree while preserving runtime custody
INVESTIGATION_EXECUTION_MODE=READ_ONLY
COMMANDS_EXECUTED=NO
BUILDS_EXECUTED=NO
TESTS_EXECUTED=NO
PRODUCT_MUTATION=NO
~~~

Before durable publication, Protos main advanced by exactly one commit:

~~~text
RECONCILED_PROTOS_HEAD=234314b1791e5dc1c05ee2341ae46a56f8670512
RECONCILED_PROTOS_SUBJECT=BUG018-C: tier-transition-safe retained-frame reads
POST_INVESTIGATION_COMMITS=1
POST_INVESTIGATION_RELEVANT_REGRESSION=NO
~~~

The intervening BUG018-C commit changes retained lexical/frame-read machinery,
related tests, PE baselines, version metadata and changelog material. It does not
modify Core buffered factories, ProtosPrelude standard-module members,
module-creation paths, Actor/P/detached transfer classification, std:io module
resolution, ProtosBufferedByteIo, the buffered C-prime execution classes,
ProtosIoOperation or ProtosIoLifecycle. The I060-A verdict therefore remains
valid at the reconciled HEAD.

## Investigation verdict

~~~text
I060_A_INVESTIGATION=COMPLETE
D167_CANDIDATE_D_IMPLEMENTABLE_FROM_CURRENT_HEAD=YES
EXISTING_GENERAL_STDLIB_MEMBER_SEAM_REUSABLE=YES
BUFFERED_READER_FACTORY_SAFE_AS_STANDARD_MODULE_MEMBER=YES
BUFFERED_WRITER_FACTORY_SAFE_AS_STANDARD_MODULE_MEMBER=YES
NEW_BUFFERED_SPECIFIC_MODULE_INFRASTRUCTURE_REQUIRED=NO
CORE_SOURCE_FILES_CAN_BE_REMOVED_OR_RELOCATED_CLEANLY=YES
PRELUDE_BINDINGS_CAN_BE_REMOVED=YES
D117_CONTRACT_CAN_REMAIN_UNCHANGED=YES
PLAT031_CONTRACT_CAN_REMAIN_UNCHANGED=YES
ACTOR_MODULE_ISOLATION_PRESERVED=YES
FRESH_WRAPPER_IDENTITY_PRESERVED=YES
NEW_LANGUAGE_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
IMPLEMENTATION_READY=YES
~~~

## Current factory and runtime architecture

At the investigated revision, Core still creates the public factories from:

~~~text
protos/lib/core/BufferedReader.protos
    BufferedReader: {}

protos/lib/core/BufferedWriter.protos
    BufferedWriter: {}
~~~

`ProtosCoreBootstrap` then obtains those two ordinary open direct children of
`Object` and installs the real native factory surface through
`ProtosStandardBufferedByteIoProtocol`.

Each factory receives exactly:

~~~text
call
owning
~~~

and is frozen by the installer.

The `bootstrap` argument accepted by
`installReaderFactory`/`installWriterFactory` is only null-checked. It is not
captured by either installed native closure and therefore contributes no
Actor-local authority or later execution state.

The factory closures retain the runtime-owned frozen Bytes prototype together
with the reader/writer and borrowing/owning choice. Every factory invocation
uses the caller's actual `ProtosActivation` to establish the wrapper's
`ActorExecutionDomain`, Future/Error ownership and lifecycle authority.

Therefore the factory itself is immutable Process-standard state; the mutable
buffer/lifecycle/order state begins only when a fresh wrapper is constructed.

## Existing general Standard Library member seam

I066-B introduced a general immutable standard-module initial-member seam in
`ProtosPrelude`:

~~~text
ModuleKey -> immutable Map<memberName, frozen runtime-owned ProtosObjectValue>
~~~

The seam is installed into fresh Actor-local module contexts before source
execution by every canonical module-creation path.

It currently publishes the canonical IP families and is deliberately generic:
module creation has no IP-specific knowledge.

I060 can reuse this same seam for:

~~~text
std:io/BufferedReader
    BufferedReader -> frozen runtime-owned reader factory

std:io/BufferedWriter
    BufferedWriter -> frozen runtime-owned writer factory
~~~

No buffered-specific loader, cache, module value family, resolver rule or
canonical-module path is required.

`ProtosStandardLibraryModuleResolver` already maps those canonical names to:

~~~text
protos/lib/io/BufferedReader.protos
protos/lib/io/BufferedWriter.protos
~~~

using the ordinary Standard Library path convention.

## Required source/module shape

The Core seeds can be removed.

`ProtosCoreBootstrap` can create the two initially empty ordinary factory
objects directly as children of `Object`, pass them through the existing
`ProtosStandardBufferedByteIoProtocol` installers, and register the resulting
frozen factories in the general standard-module-member map.

The new Standard Library source files should be minimal module sources and must
not redeclare:

~~~text
BufferedReader: {}
BufferedWriter: {}
~~~

because the canonical factory member already exists in the module context before
the source body executes. Redeclaration would create or attempt to create a
second public identity and is not the selected architecture.

The guest-visible forms become:

~~~text
BufferedReader: import("std:io/BufferedReader").BufferedReader
BufferedWriter: import("std:io/BufferedWriter").BufferedWriter

reader: BufferedReader(source)
ownedReader: BufferedReader.owning(source)

writer: BufferedWriter(target)
ownedWriter: BufferedWriter.owning(target)
~~~

## Identity and Actor ownership model

The resulting identity model is:

~~~text
Actor A std:io/BufferedReader module instance
    !==
Actor B std:io/BufferedReader module instance

Actor A module.BufferedReader
    ===
Actor B module.BufferedReader
    ===
one frozen runtime-owned reader factory
~~~

and analogously for `BufferedWriter`.

Every call still constructs:

~~~text
new ProtosObjectValue(ProtosObjectValue.rootObject())
~~~

for the wrapper, so successful constructions retain fresh wrapper identity.

The newly constructed `ProtosBufferedByteIo` captures the caller's current
Actor execution domain. Sharing the factory therefore shares no queue, buffer,
lifecycle, lower capability, operation authority or Actor-local state.

## Transfer seam refinement required by relocation

There is one bounded implementation consequence that must be included in I060-B.

Today `BufferedReader` and `BufferedWriter` are Prelude bindings. Actor,
isolated-P and detached transfer therefore recognize their exact frozen factory
objects as shared standard state through the existing Prelude-binding identity
classification.

If the bindings were simply removed and only registered as module members, that
classification would disappear. Actor transfer can then encounter the native
factory closures as non-transferable, while P/detached paths can copy/project
instead of preserving the exact standard factory identity.

D167 does not authorize that behavioral regression.

The minimal fix is general, not buffered-specific:

~~~text
ProtosPrelude.isStandardModuleMemberForRuntime(candidate)
~~~

or an equivalent identity predicate over the values registered in
`standardModuleMembers`.

The following existing transfer paths should consume that general predicate:

~~~text
ProtosActorValueTransfer
ProtosParallelRuntime
ProtosDetachedExecutionValue
~~~

A standard-module member remains an exact shared frozen standard anchor and
transfer must not import, initialize or cache a module as a hidden effect.

This general predicate may also subsume the existing IP-family
standard-module-member transfer classification. I060-B should not add
`BufferedReader`/`BufferedWriter` names, keys or dedicated getters to generic
transfer code.

~~~text
TRANSFER_SEAM_ADJUSTMENT_REQUIRED=YES
TRANSFER_SEAM_ADJUSTMENT_CLASS=GENERAL_IMPLEMENTATION_ONLY
BUFFERED_SPECIFIC_TRANSFER_INFRASTRUCTURE=NO
IMPORT_DURING_TRANSFER=NO
NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
~~~

## D117 and PLAT031 preservation

The public placement move does not require semantic changes to the buffered
operation machinery.

Reader remains:

~~~text
factory call
    -> fresh wrapper
    -> ProtosBufferedByteIo.read
    -> ProtosIoOperation / ProtosIoLifecycle
    -> lower ByteReadable Future/effect
    -> ProtosBufferedByteReaderCPrimeExecution where required
~~~

Writer remains:

~~~text
factory call
    -> fresh wrapper
    -> ProtosBufferedByteIo.write / flush
    -> ProtosIoOperation / ProtosIoLifecycle
    -> lower ByteWritable/Flushable Future/effect
    -> ProtosBufferedByteWriterCPrimeExecution where required
~~~

Close remains:

~~~text
wrapper.close
    -> existing lifecycle cutover/finalization
    -> ProtosIoReleaseCPrimeExecution where required
    -> owned lower close only for owning construction
~~~

The move therefore preserves, without reopening:

~~~text
D117 ZERO/KNOWN/UNKNOWN delegated-effect arbitration
commitment distinct from Future terminal state
pre-commit zero-effect cancellation
post-commit non-rollback
no unsafe replay after unknown lower effect
ordered buffered reader behavior and read-ahead
writer retained-output/frontier behavior
close cutover
borrowing versus owning
owned lower close propagation
PLAT031 one-operation/one-lifecycle authority
ActorExecutionDomain and C-prime custody
~~~

No operation needs to import a module internally.

## Expected implementation scope

Primary product changes:

~~~text
DELETE
protos/lib/core/BufferedReader.protos
protos/lib/core/BufferedWriter.protos

ADD
protos/lib/io/BufferedReader.protos
protos/lib/io/BufferedWriter.protos

MODIFY
protos/lib/core/prelude.protos
src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardBufferedByteIoProtocol.java
src/main/java/com/guillermomolina/protos/runtime/ProtosBufferedByteIo.java
src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java
~~~

The buffered protocol/runtime classes should change only where necessary for
placement/terminology or the general transfer classification. The D117/PLAT031
state machines must not be rewritten merely to make the facility look
source-backed.

No change is expected in the generic module lifecycle classes because I066-B
already applies the standard initial-member seam to the ordinary import and
canonical initial-module paths.

## Test migration and new regression evidence

Existing semantic tests remain valuable and should be migrated rather than
deleted.

At minimum:

~~~text
protos/tests/conformance/core-surface/required-core-bindings.protos
    remove both required Prelude names
    add explicit absence/SlotNotFound assertions

ProtosCoreBootstrapTest
    remove factories from expected Prelude surface
    assert their absence

ProtosCoreSourceNamingArchitectureTest
    remove both Core source filenames

ProtosCoreNativeBoundaryArchitectureTest
    acquire factories through std:io
    retain native call/owning assertions

ProtosStandardBufferedByteIoProtocolTest
    retain semantics
    migrate public factory acquisition

ProtosPerf006Plat031BufferedReaderCPrimeTest
    retain C-prime semantics
    migrate acquisition

ProtosPerf006Plat031BufferedWriterCPrimeTest
    retain C-prime semantics
    migrate acquisition

ProtosBufferedByteIoPlat031FoundationTest
    retain existing foundation assertions
~~~

Add one placement/identity regression analogous to I066's standard-family
placement coverage proving:

~~~text
Prelude BufferedReader/BufferedWriter absent
std:io modules expose the factories
module instances remain Actor-local
factory identity is exact/shared/frozen
each construction returns a fresh wrapper
Actor transfer preserves exact factory identity without imports
P transfer preserves exact factory identity without imports
detached transfer preserves exact factory identity without imports
ordinary module import does not create a duplicate factory identity
~~~

Existing cancellation, unknown-effect, no-replay, flush, close and C-prime tests
must remain effective.

## Normative reconciliation

`spec/io/IO_CORE.md` currently states that the factories are Standard Core
frozen-prelude bindings. I060-B must replace that placement with the ratified
D167 Standard Library placement while preserving the existing byte-buffering
contract.

`spec/io/BYTE_IO.md` owns generic byte I/O rules and does not require a new
placement architecture.

`spec/io/TEXT_IO.md` contains semantic references to BufferedReader when
describing bounded read-ahead/progress; those references remain semantically
valid and do not imply Prelude placement.

Historical specification changelogs must remain historical rather than being
rewritten.

## Consumer audit

The current HEAD search finds no Protos Standard Library/tool/application guest
consumer that invokes the Protos factories by their current global names.

Java occurrences such as `java.io.BufferedReader` in the CLI or Unicode tests
are unrelated and must not be migrated.

The public cutover is therefore bounded primarily to Core publication,
Standard-Library exposure, specification wording and tests/architecture guards.

## Materially inspected product authority

The investigation materially inspected or reconciled:

- `guillermomolina/protos#663` (I060)
- `guillermomolina/protos#637` (D167)
- the D167 durable decision record in `guillermomolina/protos-project-docs`
- I066-B implementation history and its general standard-module-member seam
- `protos/lib/core/BufferedReader.protos`
- `protos/lib/core/BufferedWriter.protos`
- `protos/lib/core/prelude.protos`
- `protos/lib/io/Files.protos`
- `protos/lib/io/ProcessStreams.protos`
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardBufferedByteIoProtocol.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosBufferedByteIo.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosDetachedExecutionValue.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardLibraryModuleResolver.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosCanonicalInitialModuleExecution.java`
- `spec/io/IO_CORE.md`
- `spec/io/BYTE_IO.md`
- `spec/io/TEXT_IO.md`
- current Core surface, buffered-protocol, PLAT031 reader/writer and architecture
  tests.
- post-investigation BUG018-C changed-path reconciliation through current HEAD.

## Research closure

~~~text
I060_A_RESEARCH=COMPLETE
NEXT_SLICE=I060-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
ISSUE_663_MUST_REMAIN_OPEN=YES
ISSUE_663_STATUS=READY
I060_CLOSURE_AUTHORIZED=NO
~~~

I060-B should assume the local `guillermomolina/protos` checkout is already at
the current HEAD selected by the implementer. It needs no information from any
other repository or from the web: D167's implementation constraints and the
complete architecture needed for the slice are restated in the handoff issued
from this investigation.
