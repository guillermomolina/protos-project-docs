# I059-B — D166 normative reconciliation implementation evidence

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I059
SLICE=I059-B
PROTOS_ISSUE=guillermomolina/protos#662
DECISION_AUTHORITY=D166/guillermomolina/protos#635
D165_AUTHORITY=guillermomolina/protos#634
D164_HANDOFF=guillermomolina/protos#633
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
replace the normative Protos specification or the live GitHub Issue state.

## Product publication

I059-B is published on `main` at:

~~~text
PROTOS_REVISION=1e8fbb27ee3966ccc48a57e04308c17a58995bdc
PROTOS_PARENT=1ad6c5b5566a7e2c639a27ae95a8a545b03fd402
COMMIT_SUBJECT=I059: narrow mandatory placement/admission/capacity architecture (D166)
SPECIFICATION_REVISION=0.1.441
IMPACT_CLASS=SPECIFICATION_ONLY
~~~

The exact published delta changes only:

~~~text
spec/concurrency/DISTRIBUTED_RUNTIME.md
spec/PROTOS_SPEC_CHANGELOG.md
~~~

No `src/**`, `src/test/**`, `protos/**`, `pom.xml`, implementation
`CHANGELOG.md`, or archived changelog content is changed by this product
publication.

## D166 Candidate C reconciliation

The published normative revision removes the following as mandatory portable
Core architecture:

~~~text
mandatory two-stage filter+score placement pipeline
mandatory dynamic scoring architecture
canonical scoring-input catalogue
normal-Actor no-CPU/memory-request policy
mandatory adaptive multidimensional admission architecture
mandatory placement/capacity-demand/admission pressure taxonomy
mandatory CapacityDemand semantic institution
mandatory proactive-demand timing
mandatory Infrastructure Controller semantic institution
~~~

The revised specification retains:

~~~text
one Actor incarnation and stable ActorRef before admission
INITIALIZING / READY boundary
capacity/admission delay keeps the same incarnation
capacity/admission delay does not synchronously fail Actor.spawn
ActorRef.stop during INITIALIZING
weak admission fairness
hard feasibility
hard constraint != soft preference
no fixed portable CPU/memory/utilization threshold
external raw capacity may be added independently
Actor.spawn does not provision infrastructure
new raw capacity enters through normal Protos lifecycle
Process failure-domain semantics
generic/overlapping failure domains
logical topology != physical failure independence
unknown independence != independent
HA claims require demonstrable evidence
unknown availability/failure independence is not reported satisfied
HA placement != persistence/replication/consensus/transactional replication/
mutable-state failover/exactly-once
correctness does not require graceful draining
~~~

Section 51 is renamed from `Capacity Demand and Infrastructure Integration` to
`Capacity and Infrastructure Boundary`. It now permits implementation-specific
capacity feedback without requiring a portable demand object, stream, taxonomy,
aggregation model, cadence, proactive timing, or controller.

## D165 preservation

The implementation intentionally leaves D165's Process/Node/Cluster and
Authority boundary intact. In particular, the product publication does not
change Process/Node/Cluster semantic identity, membership/reachability
separation, termination knowledge, scoped Authority, or the logical-versus-
physical topology boundary.

~~~text
D165_INVARIANTS_PRESERVED=YES
~~~

## D164 handoff

The revised generic HA section no longer makes Group/cardinality policy the
generic D166 contract. It explicitly leaves Group availability intent, desired
cardinality, and Group-based availability policy undecided by this section.

I059-B does not select a D164 candidate and does not define a replacement Group
HA/cardinality/controller policy.

~~~text
D164_HANDOFF_PRESERVED=YES
~~~

## Public surface

The normative reconciliation introduces no public:

~~~text
placement API
resource-request API
capacity-demand API
autoscaler API
Infrastructure Controller API
availability-status API
~~~

Open-design references to possible future facilities remain allowed as future
topics and are not current mandatory Core architecture.

## Validation provenance

After publishing the exact product revision, the maintainer reported:

~~~text
LOCAL_TESTS=PASS
LOCAL_VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

The exact command output and per-suite counts were not supplied and are not
invented here.

At the time this durable record was prepared, GitHub reported no workflow run
associated with the exact product SHA:

~~~text
REMOTE_CI_RUN_PRESENT=NO
REMOTE_CI_GREEN=NOT_YET_ESTABLISHED
~~~

This is not a CI failure; it is only the absence of remote-run evidence at that
checkpoint.

## Slice checkpoint

~~~text
I059_B_IMPLEMENTATION=COMPLETE
SPECIFICATION_ONLY=YES
RUNTIME_CHANGED=NO
TESTS_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
SPECIFICATION_REVISION=0.1.441
D165_INVARIANTS_PRESERVED=YES
D164_HANDOFF_PRESERVED=YES
MANDATORY_FILTER_SCORE_REMOVED=YES
MANDATORY_THREE_PRESSURE_MODEL_REMOVED=YES
MANDATORY_CAPACITY_DEMAND_INSTITUTION_REMOVED=YES
MANDATORY_INFRASTRUCTURE_CONTROLLER_REMOVED=YES
PUBLIC_PLACEMENT_RESOURCE_DEMAND_API_ADDED=NO
LOCAL_TESTS=PASS
REMOTE_CI_GREEN=NOT_YET_ESTABLISHED
NEXT_SLICE=I059-C
I059_CLOSURE_AUTHORIZED=NO
~~~

I059 remains open because the current-architecture documentation alignment
required by the owning Issue remains to be completed in I059-C, followed by the
Issue's final validation/CI closure gates.
