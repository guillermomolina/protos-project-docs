# AUD009-C4 — Distributed runtime, ActorGroup, placement, and authority complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#632`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence and closure revalidation revision:
`05a8754e55c1ce5cb44effc621f1b454f1fa9697`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#632`, issue comment
`5739468643`.

Owner approval provenance: `guillermomolina/protos#632`, issue comment
`5739490503`, 2026-09-19.

Derived decision routes:

- `D164 / guillermomolina/protos#633` — ActorGroup and GroupRef necessity and
  placement.
- `D165 / guillermomolina/protos#634` — Distributed topology, membership, and
  Authority ontology.
- `D166 / guillermomolina/protos#635` — Actor placement, capacity-demand, and
  HA architecture boundary.

None of the derived decisions is selected by this audit record.

## Purpose and boundary

AUD009-C4 reviewed the portable distributed-runtime institutions layered above
local Actor semantics: Process, Node, Cluster, Group/GroupRef, distributed
failure knowledge, membership/partition boundaries, placement/admission,
capacity demand, failure domains, HA policy, Authority/controllers, routing,
discovery and physical-topology boundaries.

C4 deliberately separated currently exercised Process/Actor transport semantics
from distributed institutions that are primarily normative architecture.

C4 is an evidence/classification slice. It does not itself alter normative
semantics or implementation.

## Evidence summary

### Process is current production architecture

Production Protos code uses the Process capability directly.

Examples include:

- Package Tool;
- Test Tool;
- `std:io/ProcessStreams`;
- `process.args()`;
- `process.environment()`;
- Process stdin/stdout/stderr and their selected encodings;
- fresh/exact Process execution used by current tooling architecture.

Process is therefore a current execution, bootstrap, capability and failure
boundary rather than speculative distributed infrastructure.

### ActorGroup is implemented but not production-consumed

Repository-wide Protos-source search at the C4 baseline found no
`protos/lib` or `protos/tools` consumer of:

```text
Actor.group(...)
GroupRef.send(...)
GroupRef.request(...)
```

Current use is examples, tutorials, conformance and runtime tests.

The four main Group runtime classes alone account for approximately:

```text
ProtosActorGroupRuntime.java       409 lines
ProtosGroupRefValue.java           175 lines
ProtosGroupRequest.java            597 lines
ProtosGroupSendOperationValue.java 487 lines
                                   ----------
                                   1668 lines
```

before shared Actor transport/protocol/tests/specification cost.

### Node, Cluster and Authority are primarily normative

C4 found:

- no public Core Node binding;
- no public Core Cluster binding;
- no public Authority capability/API;
- no concrete `ProtosNode` or `ProtosCluster` semantic runtime family
  corresponding to the extensive normative ontology;
- no current program-visible operation that requires an application to present
  `Authority(scope)`.

The existing Actor transport route can preserve ActorRef identity, pass-by-value
message semantics and delivery knowledge without requiring Node or Cluster as
portable semantic entities.

## Final classification ledger

```text
Process execution domain                              KEEP
Process isolation/failure domain                     KEEP
one RootActor per Process                            KEEP
Process capability/proxy                             KEEP
Process identity independent of OS process           KEEP
Process ephemeral execution capacity                 KEEP

minimal Actor remote knowledge boundary              KEEP
ActorRef routing representation invisible            KEEP
physical transport not ActorRef identity             KEEP
same-host optimization semantically invisible        KEEP
ActorRef never retargets                             KEEP

Node semantic identity                               REMOVE_NOW_RECONSIDER_LATER
Cluster semantic identity                            REMOVE_NOW_RECONSIDER_LATER
Actor -> Process -> Node -> Cluster hierarchy        REMOVE_NOW_RECONSIDER_LATER
remote Process/Node failure-knowledge framework      REMOVE_NOW_RECONSIDER_LATER
Cluster membership framework                         REMOVE_NOW_RECONSIDER_LATER
split-brain/partition Authority framework            REMOVE_NOW_RECONSIDER_LATER

Actor.group(...)                                     KEEP (D164 supersedes C4)
ActorGroup semantic identity                         KEEP (D164 supersedes C4)
GroupRef semantic capability family                  KEEP (D164 supersedes C4)
GroupRef send/request routing                        KEEP (D164 supersedes C4)
Group membership/routing runtime                     KEEP (D164 supersedes C4)
pre-acceptance Group rerouting                       KEEP (D164 supersedes C4)
stable Group identity across membership replacement KEEP (D164 supersedes C4)

Actor creation identity cutover                      KEEP
INITIALIZING != READY                                KEEP
no application dispatch before READY                KEEP
stop may terminate INITIALIZING Actor                KEEP
public SpawnOperation/Future creation handle         ABSENT / RETAIN ABSENCE

multi-domain placement model                         REMOVE_NOW_RECONSIDER_LATER
two-stage hard-filter/dynamic-score placement        REMOVE_NOW_RECONSIDER_LATER
adaptive admission architecture                     REMOVE_NOW_RECONSIDER_LATER
capacity-demand institution                         REMOVE_NOW_RECONSIDER_LATER
failure-domain topology model                       REMOVE_NOW_RECONSIDER_LATER
HA placement requirement/status model               REMOVE_NOW_RECONSIDER_LATER
desired-cardinality reconciliation model            REMOVE_NOW_RECONSIDER_LATER
Infrastructure Controller integration model         REMOVE_NOW_RECONSIDER_LATER

Authority(scope) semantic institution               REMOVE_NOW_RECONSIDER_LATER
scoped distributed Authority hierarchy              REMOVE_NOW_RECONSIDER_LATER
Group Controller semantic role                      REMOVE_NOW_RECONSIDER_LATER
controller replacement/control-state survivability REMOVE_NOW_RECONSIDER_LATER
mechanism-independent distributed control-state model
                                                     REMOVE_NOW_RECONSIDER_LATER

public Node                                          ABSENT / RETAIN ABSENCE
public Cluster                                       ABSENT / RETAIN ABSENCE
public Authority                                     ABSENT / RETAIN ABSENCE
public Group Controller                              ABSENT / RETAIN ABSENCE
public topology controls                             ABSENT / RETAIN ABSENCE
public transport controls                            ABSENT / RETAIN ABSENCE
Core service discovery                              ABSENT / RETAIN ABSENCE
Core application/service identity                   ABSENT / RETAIN ABSENCE
post-creation Group membership API                  ABSENT / RETAIN ABSENCE
public desired-cardinality API                      ABSENT / RETAIN ABSENCE
```

No C4 mechanism is classified `REMOVE_PERMANENTLY`.

## Process remains Core

Process is directly used and owns real present invariants:

- one RootActor bootstrap/failure domain;
- Process-local args/environment snapshots;
- standard I/O capability selection;
- Process-scoped capability provisioning;
- lifecycle/failure consequences for hosted Actors;
- exact/fresh execution isolation used by tools.

Collapsing it into an operating-system process would improperly expose host
placement as language identity. Collapsing it into Actor would remove the
independent execution/bootstrap/failure scope used by current tooling and I/O.

Classification: **KEEP**.

## Minimal remote Actor knowledge remains

The retained Actor contract needs only a narrow remote-delivery knowledge
boundary.

Routing loss, timeout or transport failure cannot silently retarget an ActorRef,
erase uncertainty, replay accepted work or fabricate known Actor termination.

The current transport SPI can express that boundary without exposing Node,
Cluster, membership or topology.

Classification: **KEEP**.

## Node / Cluster ontology is deferred

Approved classification:

```text
Node semantic identity                        REMOVE_NOW_RECONSIDER_LATER
Cluster semantic identity                     REMOVE_NOW_RECONSIDER_LATER
Actor->Process->Node->Cluster hierarchy       REMOVE_NOW_RECONSIDER_LATER
distributed membership/partition framework    REMOVE_NOW_RECONSIDER_LATER
```

The concepts may be appropriate for a future distributed runtime, but Core does
not currently expose them as application capabilities and does not materially
implement their semantic identity families.

Local Actor, Future, P and Process behavior does not depend on them.

Transport-independent ActorRef semantics also does not depend on them.

The exact future distributed ontology should therefore be chosen from concrete
requirements such as runtime membership, federation, tenancy, control-plane
scope, durable coordination or deployment architecture rather than fixed in
advance.

### Route

D165 / #634 owns the exact decision.

## ActorGroup / GroupRef is deferred

Approved classification:

```text
Actor.group(...)                    REMOVE_NOW_RECONSIDER_LATER
ActorGroup                          REMOVE_NOW_RECONSIDER_LATER
GroupRef                            REMOVE_NOW_RECONSIDER_LATER
Group routing / preaccept reroute   REMOVE_NOW_RECONSIDER_LATER
```

D021 and D039 remain authoritative descriptions of the current Group design
until D164 is ratified. C4 does not retroactively declare those decisions
incorrect.

AUD009 asks a different question: whether Protos should continue paying for the
institution now.

Today the public surface creates a stable routing identity over explicitly held
ActorRefs but exposes none of the larger management facilities that motivate the
full distributed Group architecture: dynamic membership, desired cardinality,
controller policy, discovery, persistence or placement policy.

The strongest argument for KEEP is real: GroupRef can represent stable service
continuity across member replacement and can reroute before concrete acceptance
without retargeting an ActorRef.

The strongest argument for removal is also real: no current production consumer
requires those guarantees, while the mechanism introduces a separate identity
family, transfer rules, routing operations, lifetime rules and substantial
runtime/specification maintenance.

D164 must therefore compare the current privileged Group design against ordinary
Actor/Standard Library router composition and any genuinely smaller privileged
kernel.

### Route

D164 / #633 owns the exact decision.

### D164 ratification reconciliation

D164 / #633 subsequently completed the required semantic decision and selected
**Candidate A — KEEP current Core ActorGroup/GroupRef**.

The C4 `REMOVE_NOW_RECONSIDER_LATER` classification above remains preserved as
the owner-approved historical audit finding that triggered D164. It is
**superseded as current authority** by the later D164 decision and must not be
used by AUD009-H or implementation work as authorization to remove Group.

D164 found that the decisive additional evidence is the removal/reintroduction
balance:

- the current Group machinery has real continuing maintenance cost;
- however, Protos deliberately keeps ActorRef bound to one exact incarnation;
- scalable/distributed Protos systems are therefore likely to need a distinct
  stable logical routing/service identity when concrete Actor incarnations are
  replaceable;
- an ordinary router Actor does not eliminate that need because the router's own
  ActorRef is still incarnation-specific;
- the current Group implementation already solves identity, capability transfer,
  routing, snapshot, acceptance, uncertainty and pre-acceptance rerouting
  boundaries that would likely recur if the facility were removed and later
  reconstructed; and
- the missing membership/discovery/cardinality/controller/placement facilities
  can remain absent until concrete requirements justify them.

The owner also explicitly accepted the reversibility boundary: retaining the
current abstraction does not make it immutable. If real future scalable use shows
that the model is wrong, a later evidence-backed decision may redesign or rewrite
it rather than preserving it merely because it already exists.

Current classification after D164:

```text
Actor.group(...)                                     KEEP
ActorGroup semantic identity                         KEEP
GroupRef semantic capability family                  KEEP
GroupRef send/request routing                        KEEP
Group membership/routing runtime                     KEEP
pre-acceptance Group rerouting                       KEEP
stable Group identity across membership replacement KEEP

public post-creation membership API                  ABSENT / RETAIN ABSENCE
public desired-cardinality API                       ABSENT / RETAIN ABSENCE
public Group Controller API                          ABSENT / RETAIN ABSENCE
Core service discovery                               ABSENT / RETAIN ABSENCE
```

Durable decision record:

`docs/project/decisions/language/D164_ACTORGROUP_AND_GROUPREF_NECESSITY_AND_PLACEMENT.md`

No normative, runtime, public-API or test change is required by D164.

## Placement, capacity and HA architecture is deferred

C4 retains only the Actor-creation invariants that are already observable:

```text
successful creation establishes one Actor identity
returned ActorRef denotes that exact incarnation
INITIALIZING is not READY
external application delivery waits for READY
ActorRef.stop may terminate INITIALIZING
```

The larger architecture is classified:

```text
distributed placement model          REMOVE_NOW_RECONSIDER_LATER
adaptive admission architecture      REMOVE_NOW_RECONSIDER_LATER
capacity-demand institution          REMOVE_NOW_RECONSIDER_LATER
failure-domain topology              REMOVE_NOW_RECONSIDER_LATER
HA placement/status                  REMOVE_NOW_RECONSIDER_LATER
desired-cardinality control          REMOVE_NOW_RECONSIDER_LATER
Infrastructure Controller model      REMOVE_NOW_RECONSIDER_LATER
```

There is no public placement/resource/failure-domain/HA/capacity-demand API and
no complete runtime implementing the current normative architecture.

A local runtime remains free to perform bounded scheduling/admission internally.
Portable placement policy should be standardized when a real cross-runtime
requirement or API exists.

### Route

D166 / #635 owns the exact decision.

## Distributed Authority/controllers are deferred

Approved classification:

```text
Authority(scope)                     REMOVE_NOW_RECONSIDER_LATER
scoped Authority hierarchy           REMOVE_NOW_RECONSIDER_LATER
Group Controller role                REMOVE_NOW_RECONSIDER_LATER
survivable control-state model       REMOVE_NOW_RECONSIDER_LATER
```

Core currently exposes no Authority value, selector, policy callback, fencing
token, controller or lease capability.

The principles behind the model are sound distributed-systems principles, but
they are not yet a current Protos capability.

A future facility that requires exclusive writer ownership, authoritative
membership, fencing, durable Group control or singleton roles must define the
actual authority capability, stale-holder behavior and scope at that time.

D165 owns this together with the distributed topology/membership ontology.

## Negative routing/discovery/topology boundaries remain useful

C4 retains the following constraints because they protect already-retained Actor
semantics without requiring a positive distributed ontology:

```text
ActorRef routing is runtime machinery
physical transport is not identity
transport optimization cannot change message semantics
ActorRef never retargets
no public transport-selection API
no public physical-locality API
no Core service discovery
no Core application/service identity
no implicit authority amplification
```

Classification: **KEEP / RETAIN ABSENCE**.

## Strongest attempted removals

```text
remove Process
    rejected -> KEEP

collapse Process into OS-process identity
    rejected -> KEEP semantic Process

remove minimal Actor remote knowledge
    rejected -> KEEP

retain Node merely because future distribution needs topology
    rejected -> REMOVE_NOW_RECONSIDER_LATER

retain Cluster merely because future coordination needs scope
    rejected -> REMOVE_NOW_RECONSIDER_LATER

retain fixed Actor->Process->Node->Cluster hierarchy
    rejected -> REMOVE_NOW_RECONSIDER_LATER

retain ActorGroup because D021/D039 already ratified it
    rejected -> sunk decision/implementation cost is not necessity evidence

claim router Actor is exactly equivalent to GroupRef
    rejected -> not equivalent; remains a mandatory lower-cost D164 candidate

retain current placement/HA architecture as future-proofing
    rejected -> REMOVE_NOW_RECONSIDER_LATER

retain Authority(scope) without a public authority consumer
    rejected -> REMOVE_NOW_RECONSIDER_LATER

expose topology/transport/discovery controls now
    rejected -> RETAIN ABSENCE
```

## Required AUD009 routing

```text
FEATURE=ActorGroup / GroupRef institution
CURRENT_OUTCOME=KEEP
SEMANTIC_DECISION_OWNER=D164 / guillermomolina/protos#633 RATIFIED
IMPLEMENTATION_OWNER=NOT_REQUIRED

FEATURE=Node / Cluster / distributed membership / Authority ontology
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D165 / guillermomolina/protos#634
IMPLEMENTATION_OWNER=TBD_AFTER_D165_RATIFICATION

FEATURE=placement / capacity-demand / failure-domain / HA architecture
PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER
SEMANTIC_DECISION_OWNER=D166 / guillermomolina/protos#635
IMPLEMENTATION_OWNER=TBD_AFTER_D166_RATIFICATION
```

## Boundary handoffs

- **D164 / #633** owns Group/GroupRef necessity and placement.
- **D165 / #634** owns distributed topology, membership and Authority.
- **D166 / #635** owns placement/capacity/failure-domain/HA architecture.
- **AUD009-D** owns detailed I/O public concepts and lifecycle/cancellation.
- **AUD009-G** owns physical routing, scheduling, transport and backend
  implementation complexity not retained as portable semantics.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#632
CHECKPOINT_COMMENT=5739468643
APPROVAL_COMMENT=5739490503
DATE=2026-09-19

PROCESS_MODEL=KEEP
MINIMAL_ACTOR_REMOTE_BOUNDARY=KEEP

ACTOR_GROUP=KEEP
ACTOR_GROUP_DECISION=D164 / guillermomolina/protos#633 RATIFIED

DISTRIBUTED_TOPOLOGY_AUTHORITY=REMOVE_NOW_RECONSIDER_LATER
DISTRIBUTED_TOPOLOGY_DECISION=D165 / guillermomolina/protos#634

PLACEMENT_CAPACITY_HA=REMOVE_NOW_RECONSIDER_LATER
PLACEMENT_DECISION=D166 / guillermomolina/protos#635

NORMATIVE_CHANGE_AUTHORIZED_BY_C4=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_C4=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_REVISION=05a8754e55c1ce5cb44effc621f1b454f1fa9697
CLOSURE_REVALIDATION=05a8754e55c1ce5cb44effc621f1b454f1fa9697

D164_NATIVE_PARENT=#632 PASS
D165_NATIVE_PARENT=#632 PASS
D166_NATIVE_PARENT=#632 PASS

D164_PROJECT_ROUTING=PASS
D165_PROJECT_ROUTING=PASS
D166_PROJECT_ROUTING=PASS

REMOVAL_ROUTES=D164,D165,D166
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_C4_CLASSIFICATION=COMPLETE
AUD009_C_PARTITION=C1-C4 COMPLETE
```

AUD009-C4 is complete once this durable record and required live GitHub closure
postconditions are verified.
