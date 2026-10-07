# I082-D2 — foreign pull iteration publication pending D192 semantic closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-D2
PROTOS_ISSUE=guillermomolina/protos#830
DECISION_GATE=D192/guillermomolina/protos#833
PARENT_WORK=AUD019/guillermomolina/protos#818
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

This is durable non-normative project evidence. It records a published
implementation and its validation provenance, but it is deliberately **not**
I082-D2 semantic closure evidence because a newly discovered observable result
contract remains unresolved under D192.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=32f61e4781270e0e5aca29aa1809a53a17f5bb02
STARTING_VERSION=0.3.275-SNAPSHOT

ENDING_PROTOS_REVISION=271a27662ee600212b7163d73253c2060875b0e8
ENDING_VERSION=0.3.276-SNAPSHOT
SPECIFICATION_REVISION=0.1.446

COMMIT_SUBJECT=I082-D2: add D188 foreign pull iteration through ordinary each
~~~

The exact product commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignAdmissionDescriptor.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignEachCall.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignHandle.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProjectedOperations.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignValueAdapter.java
src/main/java/com/guillermomolina/protos/execution/ProtosStructuredDispatchLowerer.java
src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignPullEachTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignValueFixture.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignValueProjectionTest.java
~~~

No normative specification file changed in this product commit.

## Implemented pull-iteration mechanics

The publication completes the mechanical D188 foreign pull-iteration model:

~~~text
PROVIDER_DECLARED_ITERABLE_CAPABILITY=YES
ITERABILITY_INFERRED_FROM_SHAPE=NO
FOREIGN_MEMBER_NAMED_EACH_HIJACKS_PROTOS_EACH=NO

BLOCK_PREVALIDATED_BEFORE_FOREIGN_ENTRY=YES
BLOCK_REQUIRES_ORDINARY_CALLABILITY_NOT_CLOSURE_IDENTITY=YES
BLOCK_EXPORTED_TO_FOREIGN=NO

ITERATION_MODEL=PULL
EAGER_FOREIGN_SNAPSHOT=NO
ELEMENT_ADMISSION_USES_D188_SUBSTRATE=YES

ITERATOR_ACQUISITION_IS_ENTERED_FOREIGN_OPERATION=YES
HAS_NEXT_IS_ENTERED_FOREIGN_OPERATION=YES
NEXT_IS_ENTERED_FOREIGN_OPERATION=YES

BLOCK_INVOCATION_MODEL=ORDINARY_PROTOS_INVOCATION
NEW_TASK_PER_ELEMENT=NO
NEW_FUTURE_PER_ELEMENT=NO
NEW_ACTOR_TURN_PER_ELEMENT=NO

BLOCK_ERROR_OR_CONTROL_STOPS_FURTHER_PULL=YES
BLOCK_ERROR_BECOMES_FOREIGNERROR=NO

SESSION_GENERATION_ENFORCED=YES
CLOSED_SESSION_REBIND=NO

RAW_REFERENCE_EACH=YES
FOREIGN_MODULE_FACADE_EACH=YES

ARRAY_OR_MAP_FAMILY_CONFERRED=NO
ARRAY_OR_MAP_SNAPSHOT_SEMANTICS_CONFERRED=NO

D189_CALLBACK_BRIDGE=NO
CONCRETE_PROVIDER=NO
HOST_AUTHORITY_EXPANSION=NO
STD_INTEROP_PUBLIC_API=NO
~~~

The implementation uses a dedicated `ProtosForeignEachCall` cursor and integrates
the Task-owned path with the existing structured C-prime dispatcher so a Protos
block remains under ordinary Protos execution ownership.

## Discovered semantic gap

During implementation, the current normative D188 text was found to specify the
pull loop but not the value returned after normal exhaustion.

The product commit currently implements:

~~~text
CURRENT_IMPLEMENTATION_FOREIGN_EACH_NORMAL_RESULT=EXACT_ORIGINAL_RECEIVER
~~~

and its product changelog explicitly states:

~~~text
On normal completion, each returns the receiver, matching every standard each
(owner decision; the D188 text does not yet state it).
~~~

However, under the repository design-authority rules, implementation, tests,
changelog prose, merge/push, and implementation instructions are not sufficient
semantic authority for a newly defined observable result.

No exact owner approval of the Candidate Receiver result is recorded in the
active D192 decision interaction at this point.

Therefore:

~~~text
CURRENT_IMPLEMENTATION_CHOICE=EXACT_RECEIVER
CURRENT_IMPLEMENTATION_IS_NORMATIVE_AUTHORITY=NO

D188_TEXT_FIXES_NORMAL_RESULT=NO
OBSERVABLE_SEMANTIC_GAP=YES
SPECIFICATION_CHANGE_REQUIRED=YES
~~~

## Formal decision routing

The gap has been promoted under the ISSUE-SLICE decision-checkpoint rule to:

~~~text
DECISION=D192
GITHUB_ISSUE=guillermomolina/protos#833
TITLE=Normal-completion result of projected foreign each(block)
STATE=OPEN
STATUS=status:needs-decision
ASSIGNEE=guillermomolina
NATIVE_PARENT=I082/#830
EFFECTIVE_PRIORITY=INHERITED_FROM_I082_PRIORITY_P1
~~~

D192 is intentionally unresolved and owns only the normal-completion result.

The candidate floor is:

~~~text
A = exact original receiver
B = canonical null
C = another explicitly justified fixed result
~~~

Current implementation is evidence for Candidate A, not its ratification.

## Coordination effect

Until D192 is investigated, exactly approved, durably ratified, and the required
normative specification reconciliation is published:

~~~text
I082_D2_IMPLEMENTATION_PUBLICATION=VERIFIED
I082_D2_LOCAL_VALIDATION=PASS_MAINTAINER_REPORTED
I082_D2_SEMANTIC_CLOSURE=BLOCKED
I082_D_STATUS=BLOCKED_ON_D192
I082_STATUS=BLOCKED
I082_E_RELEASE=NO
~~~

The published I082-D2 code may remain in current main while the decision is
resolved, but dependent work must not treat the receiver result as normative
authority.

## Validation provenance

The maintainer reported after the product commit was pushed:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded exactly as:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No unreported test command, count, duration or suite composition is inferred.

## Next routing

The next work item is not I082-E yet.

~~~text
NEXT_WORK_ITEM=D192/#833
NEXT_WORK_TYPE=INVESTIGATION
COMMAND_EXECUTION=NO
IMPLEMENTATION_AUTHORIZED=NO
OWNER_APPROVAL_REQUIRED=YES
~~~

After D192 ratification and specification reconciliation, I082-D2 can be closed
and I082-E can be released without repeating the already-published pull-loop
mechanics.

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
using the exact published I082-D2 commit, live GitHub coordination, current
normative specification state, repository governance, and the maintainer-reported
validation. No independent human review is claimed.
