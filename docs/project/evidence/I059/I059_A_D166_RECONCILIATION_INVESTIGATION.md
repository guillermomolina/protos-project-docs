# I059-A — D166 placement/capacity/HA reconciliation investigation

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I059
RESEARCH_SLICE=I059-A
PROTOS_ISSUE=guillermomolina/protos#662
DECISION_AUTHORITY=D166/guillermomolina/protos#635
D165_AUTHORITY=guillermomolina/protos#634
D164_HANDOFF=guillermomolina/protos#633
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative investigation evidence. It does not replace
the normative Protos specification or the live GitHub Issue state.

## Exact investigated product state

~~~text
PROTOS_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PROTOS_COMMIT_SUBJECT=PERF030: make frame-local creation ordinal PE-constant
D166_SELECTED_CANDIDATE=C
INVESTIGATION_EXECUTION_MODE=READ_ONLY
COMMANDS_EXECUTED=NO
BUILDS_EXECUTED=NO
TESTS_EXECUTED=NO
PRODUCT_MUTATION=NO
~~~

The later PERF030 product commit was inspected and changes bytecode frame-local
lowering/tests/versioning only; it does not alter Actor/distributed-runtime
semantics. Therefore the code-search/runtime conclusions below apply to the exact
product revision named above.

## Investigation verdict

~~~text
I059_IMPLEMENTATION_READY=YES
NEW_LANGUAGE_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
D164_BLOCKS_GENERIC_D166_RECONCILIATION=NO
REMOVED_D166_ARCHITECTURE_ACTUALLY_IMPLEMENTED=NO
RUNTIME_PRODUCT_CHANGE_REQUIRED=NO
~~~

D166 Candidate C already supplies the required semantic authority. The remaining
I059 work is primarily normative/document reconciliation rather than a scheduler
or autoscaling redesign.

## Normative mismatch map

The current `spec/concurrency/DISTRIBUTED_RUNTIME.md` still contains policy
that D166 explicitly removes from portable mandatory Core architecture.

### Reconcile/narrow

- §43 retains the correct Actor creation/admission identity contract but still
  names semantic capacity-demand signals and an Infrastructure Controller.
- §45 still requires:
  - the normal-Actor no-CPU/memory-request expectation;
  - a conceptual two-stage hard-filter then score pipeline;
  - a concrete dynamic scoring-input catalogue;
  - runtime-learning placement policy as normative architecture.
- §46 still requires:
  - adaptive multidimensional admission;
  - the placement/capacity-demand/admission three-pressure taxonomy;
  - proactive capacity-demand timing.
- §49 correctly retains generic failure/HA evidence rules but ties them to Group
  HA/cardinality and the capacity-demand model.
- §50 contains D164-owned Group Controller/desired-cardinality semantics and also
  has residual D166 coupling to capacity-demand/controller terminology.
- §51 still standardizes semantic Capacity Demand and Infrastructure Controller
  institutions rather than only the retained semantic/infrastructure boundary.
- §61 still names capacity-demand signals and Infrastructure Controllers as
  higher-level reactions to Process loss.

### Already conformant / retain

- `spec/concurrency/ACTORS.md` creation and INITIALIZING/READY semantics;
- `DISTRIBUTED_RUNTIME.md` §44 Process/Node/Cluster hierarchy;
- §48 generic/overlapping failure domains, logical-topology independence rule,
  unknown-independence rule, and demonstrable availability evidence;
- `spec/runtime/ABSTRACT_RUNTIME.md` delegation to the concurrency owners and
  implementation-private machinery freedom;
- current open-design entries that keep future placement/resource/controller
  APIs open rather than making them current Core.

## Runtime result

Inspection of the current runtime found the retained D166 machinery and did not
find a concrete implementation of the architecture D166 removes.

Retained runtime authority includes:

~~~text
ProtosStandardActorProtocol
    -> creation cutover before bootstrap/scheduler work
    -> ActorRef returned for the already-created incarnation

ProtosActor
    -> fixed incarnation identity
    -> INITIALIZING / READY / termination lifecycle
    -> bounded mailbox/delivery ownership

ProtosActorBootstrap
    -> INITIALIZING -> READY cutover

ProtosActorScheduler
    -> bounded local scheduling machinery

ProtosActorDeliveryAdmission
    -> bounded delivery admission / weak-fairness machinery

ProtosProcessRuntime
    -> Actor ownership and Process lifecycle/failure-domain semantics
~~~

No current runtime/type/API was found for:

~~~text
CapacityDemand
InfrastructureController
PlacementConstraint
ResourceRequest
mandatory filter+score scheduler
canonical scoring-input model
three separate placement/demand/admission pressure channels
HA-satisfaction inference machinery
~~~

Accordingly I059 should not delete or restructure the retained Actor lifecycle,
scheduler, delivery-admission, or Process state merely because similarly named
specification policy is removed.

## Existing test evidence

Existing tests already cover the executable retained boundary:

- `ProtosActorPublicApiTest`
  - `spawn` returns an ActorRef while the exact created Actor is INITIALIZING;
  - initialization failure terminates the same already-returned incarnation and
    preserves the same identity/reference;
  - public Actor/ActorRef surface remains bounded.
- `ProtosActorMailboxSchedulerTest.initializingActorRetainsAcceptedMessagesUntilReady`
  proves accepted application work is not dispatched before READY.
- `ProtosActorTerminationTest.stopWhileInitializingSuppressesQueuedBootstrapControl`
  proves stop during INITIALIZING.
- `ProtosActorDeliveryAdmissionTest.fifoPendingDisciplinePreservesSameSenderOrderAndWeakAdmissionFairness`
  covers the concrete delivery-admission weak-fairness machinery.
- `ProtosProcessRuntimeTest` covers Process ownership/termination and Actor
  cancellation/unwind integration.
- `ProtosCoreNativeBoundaryArchitectureTest` and Actor public-API tests protect
  the absence of a replacement placement/resource/capacity-demand public API.

The following retained D166 rules are currently specification-only because the
runtime has no corresponding distributed placement/failure-domain/HA facility:

~~~text
capacity-shortage actor-creation admission
unknown physical topology != proven failure independence
HA SATISFIED requires demonstrable failure-domain evidence
~~~

I059 must not invent product machinery merely to create executable tests for
currently unimplemented distributed policy.

## D165 preservation

The required I059 reconciliation can leave D165 intact:

~~~text
Process semantic identity                         KEEP
Node semantic identity                            KEEP
Cluster semantic identity                         KEEP
Actor -> Process -> Node -> Cluster hierarchy     KEEP
membership != reachability                        KEEP
reachability != physical existence                KEEP
UNREACHABLE/UNKNOWN != TERMINATED                 KEEP
Authority semantic role/scoping                   KEEP
no Authority from mere reachability/liveness      KEEP
logical topology != physical deployment identity  KEEP
~~~

No D165 reopening or reinterpretation is required.

## D164 handoff

D164 is ratified KEEP for the current ActorGroup/GroupRef routing/data-plane
institution, while desired-cardinality configuration, public Group Controller,
and placement policy remain outside that ratified expansion boundary.

Therefore I059 must:

- factor generic HA evidence semantics out of §49;
- remove D166-obsolete capacity-demand/controller coupling from Group text where
  necessary;
- leave Group-specific HA policy, desired-cardinality ownership, reconciliation,
  and controller-policy questions to D164;
- not select a new D164 candidate or silently redesign ActorGroup.

The current §50 text is broader than D164's retained public-policy boundary.
That pre-existing D164 reconciliation question does not block the generic D166
reconciliation and must not be solved opportunistically inside I059.

## Implementation slicing

### I059-B — normative D166 reconciliation

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDENCY=NONE
RUNTIME_PRODUCT_CHANGE_EXPECTED=NO
~~~

Atomic scope:

- reconcile `spec/concurrency/DISTRIBUTED_RUNTIME.md` §§43, 45, 46, 49, 50,
  51 and later residual references such as §61;
- preserve §44, §48 and all D165 authority;
- leave D164-owned Group/cardinality policy unresolved;
- add the required new `spec/PROTOS_SPEC_CHANGELOG.md` revision;
- preserve historical changelog entries unchanged;
- use existing tests plus repository-search/public-surface checks to prove the
  retained/removed boundary;
- introduce no public placement/resource/demand/autoscaler/HA API.

### I059-C — non-normative concurrency-design alignment

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDENCY=I059-B
~~~

Reconcile `docs/design/CONCURRENCY_DESIGN.md` so semantic capacity-demand,
Infrastructure Controller, and scheduler scoring are not presented as mandatory
portable Core architecture. Guides require changes only if a post-I059-B search
finds contradictory current wording.

## Research closure

~~~text
I059_A_RESEARCH=COMPLETE
NEXT_SLICE=I059-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
ISSUE_662_MUST_REMAIN_OPEN=YES
I059_CLOSURE_AUTHORIZED=NO
~~~
