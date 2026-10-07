# D166 — Actor placement, capacity-demand, and HA architecture boundary

Status: **RATIFIED — Candidate C (smaller portable placement contract)**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#635`  
Trigger: AUD009-C4 / `guillermomolina/protos#632`  
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`  
Project-record base: `438ab9d5cb31716d77138c74dfc96d52b006b5e3`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D166 selects **Candidate C — retain a smaller portable placement contract**.

The decision deliberately rejects both extremes:

- it does **not** retain every current scheduler/capacity policy as portable Core semantics; and
- it does **not** reduce Core to local admission only.

Instead, D166 keeps the architectural invariants whose later reintroduction would risk foundational redesign, while returning scheduler policy, demand representation and infrastructure-control details to implementation/future-facility freedom.

## Exact selected boundary

### Actor creation and admission

```text
Actor incarnation created before admission             KEEP
INITIALIZING / READY boundary                           KEEP
capacity shortage backpressures initialization         KEEP
capacity shortage != new Actor                         KEEP
capacity shortage != automatic spawn failure           KEEP
ActorRef.stop during INITIALIZING                       KEEP
weak admission fairness                                KEEP
bounded/correct local scheduling                       KEEP
```

### Placement

```text
hard feasibility must be respected                     KEEP
hard constraint != soft placement preference           KEEP

mandatory two-stage filter+score algorithm              REMOVE
mandatory dynamic scoring architecture                  REMOVE
specific scoring-input catalogue                        REMOVE
"normal Actors should not request CPU/memory"           REMOVE_AS_NORMATIVE_RULE
runtime-learning placement policy                       IMPLEMENTATION_FREEDOM
```

Core may describe required feasibility/invariant outcomes without standardizing one scheduler pipeline.

### Adaptive admission

```text
runtime may delay admission under pressure              KEEP
fixed portable CPU/memory thresholds                    REMAIN_ABSENT

mandatory "adaptive multidimensional" architecture      REMOVE
mandatory three-pressure taxonomy                       REMOVE
  placement pressure
  capacity-demand pressure
  admission pressure
```

### Capacity and autoscaling

```text
capacity shortage may be externally observable          KEEP
external infrastructure may add raw capacity            KEEP
new capacity enters through Protos runtime lifecycle    KEEP
Actor.spawn does not provision infrastructure           KEEP

mandatory CapacityDemand semantic institution           REMOVE_NOW_RECONSIDER_LATER
mandatory proactive-demand timing                       REMOVE
public CapacityDemand API                               REMAIN_ABSENT
specific autoscaler/controller API                      REMAIN_ABSENT
```

The retained boundary permits scheduler-to-infrastructure feedback without freezing one portable demand object, signal cadence, aggregation model or autoscaling API.

### Failure domains

```text
Process as failure domain                               KEEP
generic/overlapping failure domains                     KEEP
logical topology != physical failure independence       KEEP
unknown independence != independent                     KEEP
failure-domain discovery mechanism                      REMAIN_OPEN
```

A runtime may know host/rack/power/AZ/site or other shared-fate domains. Unknown topology must not be treated as demonstrated independence.

### High availability

```text
HA claim requires demonstrable evidence                 KEEP
unknown availability must not be claimed satisfied      KEEP

HA placement != persistence                             KEEP
HA placement != replication                             KEEP
HA placement != consensus                               KEEP
HA placement != exactly-once                            KEEP

specific Group-based HA policy                          HANDOFF_TO_D164
desired-cardinality ownership                           HANDOFF_TO_D164
```

D166 does not decide ActorGroup/GroupRef or desired-cardinality ownership. Those remain owned by D164 / #633. If D164 changes Group semantics, D166's generic failure-domain and demonstrable-availability invariants survive independently.

### Infrastructure

```text
external infrastructure != Protos semantics             KEEP
Infrastructure Controller as mandatory institution      REMOVE_NOW_RECONSIDER_LATER
specific provisioning/draining policy                   REMAIN_OPEN
correctness requires graceful draining                  KEEP_ABSENT
```

Infrastructure may provision raw capacity, but a VM/Pod/container/host appearing does not directly manufacture Protos Process/Node/Cluster membership or Actor identity.

## Relationship to D165

D165 / #634 retained the distributed topology, membership and scoped-Authority ontology.

D166 is consistent with that ratification:

- placement may operate across Process/Node/Cluster scopes where those scopes exist;
- no placement decision may infer physical failure independence from logical topology;
- membership/reachability/Authority remain distinct;
- local execution remains pay-for-use and does not require distributed coordination;
- external infrastructure does not define Protos logical identity or Authority.

D166 narrows policy within that architecture; it does not reopen D165.

```text
D165_INVARIANT_CONSISTENCY=PASS
```

## Why the AUD009-C4 blanket removal is superseded

AUD009-C4 classified the entire distributed placement/admission/capacity-demand/failure-domain/HA block as `REMOVE_NOW_RECONSIDER_LATER`.

D166 finds that classification too coarse.

The following are durable correctness/scalability boundaries rather than premature scheduler policy:

- one Actor incarnation survives admission delay;
- admission shortage backpressures initialization rather than fabricating another identity;
- feasible versus infeasible destinations remain meaningfully distinct;
- Process and overlapping physical shared-fate relationships are valid failure domains;
- unknown failure independence must not be treated as known independence;
- an HA guarantee must not be reported as satisfied without evidence;
- placement redundancy does not imply replicated state, persistence, consensus or exactly-once;
- external infrastructure may add raw capacity but does not directly mutate Protos semantic identity/topology;
- correctness may not rely on graceful draining.

Those rules protect future scale and distributed correctness even before a public placement/resource API exists.

At the same time, D166 agrees with the audit that the current specification overcommits to scheduler/control-plane policy where no concrete portable facility requires it.

## Policy removed from portable Core

D166 removes the requirement that portable Protos architecture standardize:

- a mandatory two-stage hard-filter-then-score placement algorithm;
- one canonical dynamic scoring architecture;
- one canonical catalogue of scoring observations;
- a normative expectation that normal Actors should not declare CPU/memory requirements;
- a mandatory three-way taxonomy of placement, capacity-demand and admission pressure;
- a mandatory semantically distinct CapacityDemand institution;
- a requirement that capacity demand become observable at one specific proactive timing point;
- a mandatory Infrastructure Controller semantic institution.

Implementations remain free to use any of these designs internally.

A future public placement/resource/autoscaling facility may standardize some of them when concrete evidence justifies doing so.

## Comparative evidence

The decision packet compared at least the following materially different approaches:

- Erlang/BEAM — runtime-owned scheduling with substantial implementation freedom and distributed nodes separated from scheduler details;
- Akka — deployment/placement/routing policy layered over Actor semantics, with cluster topology/failure rules distinct from ordinary local execution;
- Orleans — multiple evolving placement strategies and extensible filters, demonstrating that placement policy changes over runtime generations;
- Kubernetes — explicit filter/score scheduling framework, topology spread/failure-domain constraints and a separately evolving autoscaling/control-plane layer;
- Nomad — constraints/affinity/spread plus separate autoscaling mechanisms, with explicit scheduling-cost tradeoffs;
- Ray — resource-aware distributed scheduling, placement groups and pending-resource-demand feedback to autoscaling.

The shared lesson is not one universal scheduler algorithm. It is the value of keeping identity, feasibility, topology/failure evidence, scheduling policy and capacity provisioning as distinct concerns.

## Candidate result

### Candidate A — retain the current full Core architecture

Rejected.

It keeps useful safety boundaries but freezes too much current scheduler/autoscaling policy before a portable public requirement exists.

### Candidate B — retain only local Actor admission invariants

Rejected.

It removes too much. Failure-domain knowledge, demonstrable HA claims and the semantic/infrastructure boundary are architectural correctness constraints with high reintroduction cost.

### Candidate C — smaller portable placement contract

**Selected.**

It preserves identity, admission, feasibility, failure-domain, HA-evidence and infrastructure-separation invariants while returning concrete scheduler/capacity-demand policy to implementations and future facilities.

### Candidate D — define a public placement/resource facility now

Rejected.

No current evidence justifies a new user-visible resource/placement/control-plane API merely to give the current normative architecture a consumer.

## Twelve-dimension result

The decision considered:

1. correctness/invariants;
2. Protos alignment;
3. present-need proportionality;
4. incremental growth;
5. future-option resilience;
6. scalability;
7. conceptual simplicity;
8. portability/implementation freedom;
9. runtime/resource cost;
10. failure/operability;
11. deferral/reversibility/migration;
12. evidence maturity/implementation risk.

The supporting totals in the reviewed packet were:

```text
Candidate A  45 / 60
Candidate B  45 / 60
Candidate C  59 / 60
Candidate D  31 / 60
```

The arithmetic is supporting evidence only, not decision authority.

## Strongest argument against Candidate C

A tightly integrated scheduler/capacity-demand/provisioner feedback loop can materially improve autoscaling quality. Systems such as Ray and Kubernetes demonstrate that pending unschedulable demand and placement/resource constraints can usefully drive capacity provisioning.

Removing a mandatory CapacityDemand institution now may mean revisiting scheduler/control-plane interfaces later.

D166 accepts that risk because Candidate C does not forbid or remove such feedback internally. It only refuses to freeze its representation, timing and public semantic identity before a concrete portable consumer exists.

If future evidence shows that a stable capacity-demand contract must cross runtime/tool/infrastructure boundaries, that contract should be designed as a separate evidence-backed facility.

## Pay-for-what-you-need

```text
standalone/local Actor execution
    -> no distributed scheduler/control plane required

placement/admission under ordinary local capacity
    -> implementation may use minimal local machinery

failure-domain/HA knowledge absent
    -> runtime makes no unsupported independence/availability claim

autoscaling integration absent
    -> no Infrastructure Controller or CapacityDemand object required
```

## Grow-as-you-need

A future implementation/facility may add:

- resource requests/limits;
- placement hints/constraints;
- alternative feasibility/optimization solvers;
- resource-aware or topology-aware placement;
- GPU/special-resource scheduling;
- explicit capacity-demand streams;
- autoscaler/control-plane APIs;
- draining and scale-down policy;
- richer failure-domain providers;
- federation/region placement;
- concrete availability objectives.

Those additions must preserve D166's retained identity/failure/HA/infrastructure boundaries and D165's topology/Authority invariants.

## Approval provenance

The exact Candidate C packet above was presented to the project owner in the active interaction on 2026-09-19.

The project owner explicitly approved it:

```text
aprobado
```

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Normative and implementation consequence

Candidate C requires specification reconciliation.

At minimum the implementation owner must:

1. preserve the Actor creation/admission identity and weak-fairness rules;
2. preserve hard-feasibility semantics without standardizing one filter/score pipeline;
3. remove mandatory dynamic-scoring and three-pressure architecture;
4. remove the mandatory CapacityDemand institution/proactive timing while retaining the semantic-to-infrastructure separation;
5. preserve failure-domain and demonstrable-HA rules;
6. hand Group-specific HA/desired-cardinality claims back to D164 rather than deciding them inside D166;
7. remove the mandatory Infrastructure Controller institution while retaining implementation/external provisioning freedom;
8. reconcile guide/design material that presents removed policy as portable Core;
9. preserve D165 topology/membership/Authority boundaries.

No public replacement API is authorized by D166.

```text
D166_STATUS=RATIFIED
SELECTED_CANDIDATE=C

ACTOR_ADMISSION_IDENTITY_BOUNDARY=KEEP
WEAK_ADMISSION_FAIRNESS=KEEP
HARD_FEASIBILITY=KEEP
HARD_SOFT_PLACEMENT_DISTINCTION=KEEP

MANDATORY_FILTER_SCORE_PIPELINE=REMOVE
MANDATORY_DYNAMIC_SCORING_ARCHITECTURE=REMOVE
MANDATORY_SCORING_INPUT_CATALOGUE=REMOVE
NORMAL_ACTOR_NO_RESOURCE_REQUESTS_RULE=REMOVE
MANDATORY_THREE_PRESSURE_TAXONOMY=REMOVE

CAPACITY_SHORTAGE_EXTERNALLY_OBSERVABLE=KEEP
EXTERNAL_RAW_CAPACITY_PROVISIONING=KEEP
ACTOR_SPAWN_DOES_NOT_PROVISION_INFRASTRUCTURE=KEEP
MANDATORY_CAPACITY_DEMAND_INSTITUTION=REMOVE_NOW_RECONSIDER_LATER
MANDATORY_PROACTIVE_DEMAND_TIMING=REMOVE

PROCESS_FAILURE_DOMAIN=KEEP
GENERIC_OVERLAPPING_FAILURE_DOMAINS=KEEP
UNKNOWN_INDEPENDENCE_NOT_INDEPENDENT=KEEP
HA_CLAIM_REQUIRES_EVIDENCE=KEEP
HA_NOT_PERSISTENCE_REPLICATION_CONSENSUS_EXACTLY_ONCE=KEEP

GROUP_SPECIFIC_HA=OWNED_BY_D164
DESIRED_CARDINALITY=OWNED_BY_D164

EXTERNAL_INFRASTRUCTURE_NOT_PROTOS_SEMANTICS=KEEP
MANDATORY_INFRASTRUCTURE_CONTROLLER=REMOVE_NOW_RECONSIDER_LATER
GRACEFUL_DRAINING_REQUIRED=ABSENT_KEEP

PUBLIC_PLACEMENT_API=ABSENT_KEEP
PUBLIC_CAPACITY_DEMAND_API=ABSENT_KEEP
PUBLIC_INFRASTRUCTURE_CONTROLLER_API=ABSENT_KEEP

NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
