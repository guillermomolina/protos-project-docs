# D165 — Distributed topology, membership, and Authority ontology

Status: **RATIFIED — Candidate A (KEEP)**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#634`  
Trigger: AUD009-C4 / `guillermomolina/protos#632`  
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`  
Project-record base: `9f7c118a17d02cbc4822e6fc575b660f15d80466`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D165 selects **Candidate A — retain the current distributed ontology**.

```text
Process semantic identity                         KEEP

Node semantic identity                            KEEP
Cluster semantic identity                         KEEP
Actor -> Process -> Node -> Cluster hierarchy     KEEP

membership != reachability                        KEEP
reachability != physical existence                KEEP
UNREACHABLE != TERMINATED                         KEEP
UNKNOWN != TERMINATED                             KEEP

no automatic death-from-silence                   KEEP
no implicit split-brain winner                    KEEP

Authority semantic role                           KEEP
Authority scoped                                  KEEP
Authority pay-as-you-grow                         KEEP
no Authority from mere reachability               KEEP
no Authority from majority/age/liveness alone     KEEP
local work without higher-scope Authority          KEEP

public Node API                                    REMAINS ABSENT
public Cluster API                                 REMAINS ABSENT
public Authority API                               REMAINS ABSENT

membership protocol                               REMAINS UNSPECIFIED
consensus algorithm                               REMAINS UNSPECIFIED
failure detector                                  REMAINS UNSPECIFIED
fencing mechanism                                 REMAINS UNSPECIFIED
physical topology                                 REMAINS IMPLEMENTATION DETAIL
```

The retained model is architectural authority, not a requirement to implement distributed Cluster machinery for standalone programs.

## Approval provenance

AUD009-C4 had provisionally classified Node, Cluster, the fixed runtime hierarchy, distributed membership/partition ontology, and scoped Authority as `REMOVE_NOW_RECONSIDER_LATER`.

D165 re-examined that classification under the project rule that current non-use is not sufficient removal evidence when an institution is a low-cost architectural guardrail whose later reintroduction could require foundational redesign.

The exact Candidate A packet was presented to the project owner, including repository evidence, prior-art comparison, Candidate A-D tradeoffs, twelve-dimension scoring, the strongest argument against KEEP, and the distinction between retaining architecture versus pre-implementing a public distributed-control API.

The project owner explicitly approved the exact candidate in the active interaction on 2026-09-19:

```text
ok aprobada
```

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Why AUD009-C4's provisional removal is superseded

C4 correctly observed that current Core exposes no public Node, Cluster, or Authority capability and that no concrete `ProtosNode` or `ProtosCluster` semantic runtime family exists.

D165 finds that those facts do not establish that the architecture is unnecessary.

The retained rules prevent a future distributed implementation from collapsing distinct concerns merely because an early implementation is simpler:

- logical execution identity versus physical deployment identity;
- reachability versus lifecycle termination;
- membership versus physical existence;
- communication capability versus exclusive decision authority;
- local coordination versus distributed coordination;
- local progress versus higher-scope authoritative operations.

Those distinctions become more important, not less, as a runtime scales.

D165 therefore supersedes the C4 removal classification for this block. The C4 record remains historical audit evidence but is not the current owner-approved authority for D165.

## Current repository evidence

At the evidence revision:

- Process is already a concrete Core execution, bootstrap, isolation and failure domain;
- non-local Actor communication already has a transport abstraction through `ProtosActorTransportRoute`;
- ActorRef transport/routing remains semantically invisible and does not retarget Actor identity;
- no public Node, Cluster or Authority object/API is exposed;
- no concrete `ProtosNode` or `ProtosCluster` semantic runtime family exists;
- local Actor/Future/P/Process execution does not require Cluster machinery;
- distributed topology semantics are therefore largely a constraint on future implementation architecture rather than current per-program runtime machinery.

That combination makes Candidate A strongly pay-for-use: ordinary local programs do not pay for a distributed control plane merely because Node/Cluster/Authority remain normative concepts.

## Safety boundary retained

The most important retained distributed rules are negative/safety constraints:

```text
remote silence != proof of death
timeout != proof of termination
transport failure != proof of termination
partition suspicion != authoritative partition proof
reachability != membership
membership != Authority
liveness != Authority
majority/age/topology alone != Authority
```

An implementation may use failure detectors, gossip, consensus, leases, witnesses, fencing, external coordination services, or other mechanisms, but those mechanisms must justify any stronger authoritative conclusion they expose.

If an operation requires exclusive Authority and currently valid Authority cannot be demonstrated, the operation must not proceed merely because one side is still alive or reachable.

Unrelated local work may continue when higher-scope Authority is unavailable.

## Why retain Node and Cluster specifically

Candidate C would retain only abstract safety principles while removing concrete Node/Cluster semantic identities.

D165 considered that option seriously.

The retained definitions are already intentionally abstract:

- a **Node** is a logical ephemeral runtime member capable of coordinating one or more Processes; it is not a VM, host, container, Kubernetes Node, or other physical infrastructure identity;
- a **Cluster** is a logical coordination domain whose identity is independent of any particular current Node and may outlive Node churn only when its defining control state is explicitly preserved.

This gives a useful pay-as-you-grow coordination ladder:

```text
Actor
  -> Process      local execution/isolation/failure scope
  -> Node         coordination over multiple Processes
  -> Cluster      coordination over multiple Nodes
```

and permits Authority to be scoped to the smallest domain that actually requires exclusivity.

Retaining those identities does not expose them as application APIs and does not require standalone programs to initialize distributed infrastructure.

## Comparative evidence

The D165 analysis compared multiple established distributed approaches.

Relevant recurring patterns include:

- Erlang/OTP treats Node as a runtime-distribution identity distinct from the processes it hosts;
- Akka Cluster separates logical cluster membership from physical hosting and distinguishes unreachable from removed/downed membership;
- Orleans uses silo incarnations plus explicit cluster membership state/versioning;
- Kubernetes separates membership/capacity from lease-based exclusive coordination;
- Consul/Nomad-style systems separate member/service observations from explicit sessions/locks/coordination authority;
- systems with external topology still require some distinction between failure suspicion, membership and exclusive authority when correctness depends on fencing or a single writer.

The exact mechanisms differ, but the separation between identity, membership/reachability and exclusive authority is mature distributed-systems prior art.

## Candidate result

### Candidate A — retain current distributed ontology

**Selected.**

Keeps Node, Cluster, the intrinsic Actor -> Process -> Node -> Cluster layering, distributed knowledge/split-brain safety boundaries and scoped Authority while retaining all current public absences and implementation freedom.

### Candidate B — remove/defer all positive distributed ontology

Rejected.

It maximizes immediate specification simplicity but permits the next distributed implementation to grow without an explicit distinction between runtime membership, reachability and Authority. Correcting such a topology after production distributed mechanisms exist may require foundational redesign.

### Candidate C — keep abstract safety invariants, remove concrete Node/Cluster entities

Rejected, although it is the strongest alternative.

It preserves most correctness guardrails and maximizes future ontology freedom, but discards an already abstract, pay-as-you-grow coordination hierarchy before any concrete evidence demonstrates that the hierarchy is wrong. The retained Node/Cluster concepts impose no current public API or standalone runtime tax.

### Candidate D — define a smaller generic membership abstraction

Rejected.

No current requirement demonstrates a smaller replacement abstraction that is both sufficient and better than the existing logical Node/Cluster separation. Introducing another abstraction now would replace one future-facing ontology with another rather than reduce uncertainty.

## Twelve-dimension result

The comparison considered:

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

Candidate A and Candidate C were the strongest options. Candidate A was selected because the retained hierarchy is already abstract and pay-for-use, while removing it now creates a larger risk of foundational redesign once real distributed runtime machinery begins depending on a flatter or conflated model.

The numeric comparison was supporting evidence only, not decision authority.

## Strongest argument against Candidate A

The fixed `Actor -> Process -> Node -> Cluster` hierarchy may prove too rigid for future designs involving regions, cells, federation, tenants, overlapping coordination domains, edge/cloud topology, or other non-hierarchical scopes.

That objection is accepted.

D165 nevertheless keeps Candidate A because:

- the current Node/Cluster identities are logical rather than physical;
- no public Node/Cluster API freezes application syntax or compatibility today;
- additional orthogonal scopes can be introduced later if real requirements justify them;
- removing the hierarchy now provides little current runtime benefit;
- rebuilding identity/membership/Authority boundaries after a distributed implementation grows around a flatter model has materially higher migration risk.

## Pay-for-what-you-need

Candidate A does not require every Protos execution to instantiate Node/Cluster machinery.

```text
standalone Process
    -> may remain a lightweight local runtime

local Actors/Futures/P
    -> require no Cluster membership

distributed membership
    -> activated only by a runtime/facility that actually provides it

Authority
    -> paid only at the smallest scope where an exclusive invariant needs it
```

The absence of public Node/Cluster/Authority controls remains intentional.

## Grow-as-you-need

Future work may define:

- concrete Node/Cluster runtime representations;
- membership/discovery protocols;
- failure detectors;
- partition/downing policies;
- fencing/lease/consensus mechanisms;
- administrative observability;
- federation or additional coordination scopes;
- explicit distributed-control APIs.

None is selected by D165.

Such facilities must compose with the retained identity, failure-knowledge and Authority invariants rather than silently redefine them.

## Compatibility and implementation consequence

Candidate A is status quo.

```text
normative semantic delta      NONE
public API delta              NONE
runtime implementation delta  NONE
test migration                NONE
follow-up Ixxx                NOT REQUIRED
```

No Protos repository content change is authorized or required by D165.

## Fixed authority consistency

D165 preserves the issue's fixed authority:

```text
Process remains Core                                      PASS
ActorRef identity independent of transport/location       PASS
transport failure does not retarget ActorRef              PASS
unreachability alone does not prove termination           PASS
local execution does not require Cluster machinery        PASS
public Node/Cluster/Authority/topology controls absent     PASS
```

No previously approved invariant is reopened.

## Ratification summary

```text
D165_STATUS=RATIFIED
SELECTED_CANDIDATE=A

PROCESS=KEEP
NODE_SEMANTIC_IDENTITY=KEEP
CLUSTER_SEMANTIC_IDENTITY=KEEP
ACTOR_PROCESS_NODE_CLUSTER_HIERARCHY=KEEP

MEMBERSHIP_REACHABILITY_DISTINCTION=KEEP
UNREACHABLE_NOT_TERMINATED=KEEP
UNKNOWN_NOT_TERMINATED=KEEP
NO_AUTOMATIC_DEATH_FROM_SILENCE=KEEP
NO_IMPLICIT_SPLIT_BRAIN_WINNER=KEEP

AUTHORITY_SEMANTIC_ROLE=KEEP
AUTHORITY_SCOPED=KEEP
AUTHORITY_PAY_AS_YOU_GROW=KEEP
LOCAL_EXECUTION_WITHOUT_HIGHER_AUTHORITY=KEEP

PUBLIC_NODE=ABSENT_KEEP
PUBLIC_CLUSTER=ABSENT_KEEP
PUBLIC_AUTHORITY=ABSENT_KEEP

NORMATIVE_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_OWNER=NOT_REQUIRED

AUD009_C4_D165_CLASSIFICATION=SUPERSEDED_BY_D165

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
