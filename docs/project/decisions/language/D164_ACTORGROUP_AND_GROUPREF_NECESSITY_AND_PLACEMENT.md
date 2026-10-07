# D164 — ActorGroup and GroupRef necessity and placement

Status: **RATIFIED — Candidate A (retain current Core ActorGroup/GroupRef)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#633`
Trigger: AUD009-C4 / `guillermomolina/protos#632`
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`
Project-record base: `ea3b94526bc9488b07a4873d0e30b493d4cb3363`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D164 selects **Candidate A**.

Core retains the current ActorGroup / GroupRef institution:

```text
Actor.group(firstMember, additionalMembers...) -> GroupRef

ActorRef
    -> denotes exactly one Actor incarnation
    -> never retargets

GroupRef
    -> denotes one stable Group identity
    -> routes send/request to one eligible concrete Actor member
    -> preserves one logical message snapshot
    -> may reroute only while concrete acceptance is known not to have occurred
    -> never turns post-acceptance failure or uncertainty into transparent replay
```

The current Group identity, GroupRef semantic identity/transfer rules,
Process-owned Group lifetime, routing behavior, empty-live-Group semantics, and
pre-acceptance rerouting remain in Core.

No new public Group-control facility is introduced by D164.

In particular, D164 does **not** add:

- public post-creation membership mutation;
- service discovery;
- desired-cardinality configuration;
- a public Group Controller;
- persistence of Group identity;
- placement policy;
- Cluster coupling;
- distributed Authority/fencing machinery; or
- a requirement that future distributed control use today's internal mechanism.

Those capabilities remain separately decidable when concrete requirements exist.

## Approval provenance

The complete D164 comparison was presented to the project owner after repository
inspection and comparative prior-art analysis.

The first recommendation was Candidate B / REMOVE_NOW_RECONSIDER_LATER. The
project owner challenged that recommendation on the decisive retrospective
question: if Protos later grows into scalable/distributed tools and the same
stable routing abstraction is needed again, removal now may only spend
implementation and reasoning effort to rebuild substantially the same mechanism.

The decision was then re-evaluated specifically against:

- continuing maintenance cost of KEEP;
- concrete removal cost;
- reintroduction/redesign cost;
- probability that future scalable Protos systems need a stable identity above
  replaceable Actor incarnations;
- whether the current Group model appears structurally wrong or merely incomplete
  above its routing/data-plane layer; and
- whether current Group machinery is causing active runtime or project debt.

The revised recommendation presented for approval was:

```text
D164 = Candidate A — KEEP current Core ActorGroup/GroupRef

KEEP:
    Actor.group(...)
    ActorGroup identity
    GroupRef identity/capability
    transfer semantics
    send/request
    routing to one eligible member
    pre-acceptance rerouting
    empty live Group
    current Process-owned lifetime

DO NOT EXPAND YET:
    public membership API
    discovery API
    desired cardinality API
    Group Controller API
    persistence
    placement policy
    Cluster coupling
    Authority machinery
```

The project owner then explicitly accepted that revised recommendation in the
active interaction on 2026-09-19:

```text
si y si el dia de mañana me pongo con esto y digo oye que me he equivocado
(como me pasó con la tool de test), lo reescribimos y listo
```

This approval also establishes the intended reversibility boundary: retaining
ActorGroup/GroupRef now does not make its current implementation or every detail
of the abstraction immutable. A future evidence-backed decision may redesign or
rewrite it when real requirements demonstrate that the current model is wrong.

```text
SELECTED_CANDIDATE=A
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## GITHUB021 invariant consistency

Candidate A preserves every fixed D164 invariant:

```text
ACTORREF_DENOTES_ONE_CONCRETE_INCARNATION          PRESERVED
ACTORREF_NEVER_RETARGETS                           PRESERVED
DELIVERY_UNCERTAINTY_REMAINS                       PRESERVED
NO_TRANSPARENT_POST_ACCEPTANCE_REPLAY              PRESERVED
PROCESS_REMAINS_EXECUTION_FAILURE_DOMAIN           PRESERVED
ACTOR_ISOLATION_REMAINS                            PRESERVED
MESSAGE_SNAPSHOT_SEMANTICS_REMAIN                  PRESERVED
CORE_SERVICE_DISCOVERY_REMAINS_ABSENT              PRESERVED
PUBLIC_GROUP_CONTROLLER_REMAINS_ABSENT             PRESERVED
PUBLIC_DESIRED_CARDINALITY_REMAINS_ABSENT          PRESERVED
PUBLIC_MEMBERSHIP_MANAGEMENT_REMAINS_ABSENT        PRESERVED
```

Observable semantic delta:

```text
NONE
```

Candidate A retains current behavior and therefore requires no normative or
runtime change.

No materially new observable consequence was introduced after the revised
packet was presented for approval.

## Repository evidence

At the evidence revision, ActorGroup/GroupRef is not merely speculative
documentation.

The public Core and runtime already implement:

- `Actor.group(...)` acquisition from explicit ActorRefs;
- independent Group identity;
- independent semantic GroupRef capability identity;
- GroupRef transfer/rematerialization while preserving semantic identity;
- GroupRef identity hashing;
- Process ownership and termination integration;
- send/request routing to one eligible member;
- message snapshot preservation across routing attempts;
- local and routed-member handling;
- pre-acceptance rerouting;
- acceptance/uncertainty boundaries;
- empty-live-Group pending behavior; and
- integration with Actor lifecycle, diagnostics, bootstrap and transfer rules.

The four primary Group-specific runtime classes contain approximately 1,668
lines at the audited baseline:

```text
ProtosActorGroupRuntime.java        409
ProtosGroupRefValue.java            175
ProtosGroupRequest.java             597
ProtosGroupSendOperationValue.java  487
                                  -----
                                   1668
```

This is a real continuing maintenance cost. D164 does not treat historical
implementation effort as a reason to KEEP.

However, the runtime also already contains internal membership seams:

```text
addMemberForRuntime(...)
removeMemberForRuntime(...)
addRemoteReadyMemberForRuntime(...)
removeRemoteMemberForRuntime(...)
```

These are not public Protos membership APIs. Their existence shows that the
current implementation is naturally decomposed into a stable routing/data-plane
identity plus future control-plane policy, rather than proving that the Group
identity itself is unnecessary.

## The key reconsideration of AUD009-C4

AUD009-C4 classified ActorGroup/GroupRef as:

```text
REMOVE_NOW_RECONSIDER_LATER
```

That classification correctly established several removal arguments:

- no current `protos/lib` or `protos/tools` production consumer uses
  `Actor.group`, GroupRef.send or GroupRef.request;
- no public membership, desired-cardinality, controller, discovery, persistence
  or placement API exists;
- Group adds a second routing path and a distinct identity/capability family; and
- maintaining Group-aware request/send behavior has real cost.

D164 found that these facts establish **removability**, but not that removal is
the lower-cost long-term choice.

The missing control-plane facilities do not imply that the current data-plane
identity is wrong. They can instead mean that Group is an intentionally partial
lower layer:

```text
today:
    stable Group identity
    + routing
    + acceptance boundary

later, only if required:
    membership reconciliation
    discovery
    desired cardinality
    placement
    persistence
    authority/fencing
```

Most importantly, Protos has already selected the invariant that ActorRef denotes
one exact Actor incarnation and never retargets. If scalable applications later
need a logical service/routing identity that remains meaningful while concrete
Actor incarnations change, some distinct stable identity is required.

An ordinary router Actor does not fully eliminate this need: its ActorRef still
denotes one concrete router incarnation. Preserving service identity across
router replacement requires another stable layer, potentially recreating the
same essential role as GroupRef.

The C4 classification is therefore superseded for this mechanism:

```text
AUD009-C4 preliminary classification
    Actor.group / ActorGroup / GroupRef / Group routing
    -> REMOVE_NOW_RECONSIDER_LATER

D164 ratified classification
    Actor.group / ActorGroup / GroupRef / Group routing
    -> KEEP
```

The rest of AUD009-C4 remains unaffected.

## Comparative evidence

The investigation compared different approaches to stable routing and actor
identity.

### Erlang / Elixir

BEAM systems can build routing and membership using ordinary processes,
registries and process-group facilities. Concrete process identity remains
distinct from higher-level names/groups.

Contribution to D164: ordinary composition is a credible option for routing, but
stable naming/group membership is still a separate concern when continuity
across process replacement matters.

### Akka

Akka supports ordinary router Actors for relatively direct routing composition,
while Cluster Sharding introduces a stronger logical identity/routing layer for
distributed entities.

Contribution to D164: Candidate C is viable for simple routing, but a router
Actor and a stable distributed identity solve different problems.

### Orleans

Orleans deliberately separates logical grain identity from transient activation
placement. Activations may change while the logical target remains meaningful.

Contribution to D164: scalable actor systems commonly need a layer whose
identity is not the identity of one concrete execution incarnation.

### Pony

Pony demonstrates that useful Actor systems do not intrinsically require a
GroupRef institution.

Contribution to D164: Candidate B remains technically coherent for a smaller
Core. GroupRef is justified only if Protos expects the stronger continuity
boundary to matter.

### Swift distributed actors

Swift keeps distributed actor identity and resolution behind the selected actor
system rather than standardizing a Group abstraction identical to Protos.

Contribution to D164: implementation freedom matters; D164 therefore does not
standardize Group control-plane topology, discovery or membership protocol.

### Service-oriented systems

Service abstractions such as Kubernetes Service demonstrate the recurring
separation between a stable logical destination and a changing backend set.

Contribution to D164: when Protos grows from local programs to scalable tools and
services, the stable-target/changing-members distinction is likely to recur even
if the eventual control plane differs substantially from today's incomplete
design.

## Candidate set

### Candidate A — retain current Core Group/GroupRef

**Selected.**

Keep the already implemented stable Group identity and communication capability
without adding the currently missing control plane.

This preserves a likely useful scaling layer while keeping its unused higher
capabilities absent.

### Candidate B — remove with no immediate replacement

Rejected after reconsideration.

This candidate produces the smallest present Core and removes real maintenance
surface. It remains a coherent future choice if Group becomes active drag or its
semantic model proves wrong.

The decisive weakness is reintroduction risk: a future scalable Protos system is
likely to require a logical communication/routing identity distinct from exact
Actor incarnation identity. Reconstructing that facility would need to revisit
many of the same hard boundaries already implemented: identity, capability
transfer, routing, acceptance, failure uncertainty and no transparent replay.

Removing the current institution therefore has a substantial risk of being churn
rather than durable simplification.

### Candidate C — ordinary router/service Actor in Standard Library

Rejected as the Core replacement.

A router Actor is useful prior art and can implement simple source-backed routing,
but its `ActorRef` denotes one concrete router incarnation. It also introduces a
mailbox hop and router lifecycle/failure domain.

It does not automatically provide the stable non-Actor routing identity,
transfer semantics and pre-acceptance routing boundary of GroupRef.

No present requirement justifies adding such a library abstraction merely to
replace the retained Group institution.

### Candidate D — smaller privileged routing capability

Rejected for now.

A smaller kernel may eventually be correct, but there is no current requirement
that identifies which Group guarantees are unnecessary. Redesigning an already
working boundary without such evidence risks replacing known machinery with a
new speculative abstraction.

## Comparative scoring after reconsideration

Scores use 1–5. Confidence is H/M/L.

| Criterion | A | B | C | D |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 5H | 5H | 5M |
| Protos alignment | 5H | 5H | 5H | 4M |
| Present-need proportionality | 4H | 5H | 4H | 2M |
| Incremental growth | 5H | 3M | 4H | 4M |
| Future-option resilience | 5M | 3M | 4H | 4M |
| Scalability | 5M | 4M | 3H | 5M |
| Conceptual simplicity | 3H | 5H | 4H | 4M |
| Portability / implementation freedom | 4H | 5H | 5H | 4M |
| Runtime / resource cost | 5H | 5H | 4H | 5M |
| Failure / operability | 5H | 4M | 3H | 5M |
| Deferral / reversibility / migration | 5M | 2M | 4H | 3L |
| Evidence maturity / implementation risk | 5H | 4H | 4H | 2L |

The arithmetic totals are not decision authority.

Candidate A carries a conceptual/maintenance-cost warning because a distinct
identity/routing institution must continue to be understood and maintained.

Candidate B carries the stronger retrospective underengineering warning because
future stable routing/service identity is plausible, ActorRef cannot satisfy it
without violating its exact-incarnation invariant, and reintroduction would
likely revisit substantial already-solved semantics.

## Incremental-design analysis

### Smallest sufficient solution

For current small/local programs, Actor/ActorRef alone is sufficient.

For the already retained Group capability, the smallest justified state is the
one currently exposed:

```text
explicit ActorRefs
    -> Actor.group(...)
    -> GroupRef
    -> send/request routing
```

D164 does not justify adding a control plane merely because the current runtime
contains internal membership machinery.

### Pay for what you need

Programs that never create a Group do not acquire Group identities, Group
routing queues, Group membership sets or Group request/send operations.

There remains project-wide conceptual and maintenance cost, especially when
Actor messaging/transport changes, but no evidence showed a material runtime tax
on ordinary non-Group programs.

### Grow as you need

A future system can add control-plane capability around the retained identity
without changing ActorRef semantics:

```text
GroupRef stable destination
        +
future membership/reconciliation
        +
future discovery/placement/authority only where required
```

The current design therefore supports incremental growth rather than requiring
the complete distributed system to exist today.

### Cost of removal now

Removal would touch at least:

- four Group-specific runtime classes;
- Actor and Process lifecycle integration;
- standard Actor protocol/bootstrap;
- represented-value identity/hash behavior;
- Actor value transfer;
- diagnostic representation;
- conformance tests;
- Java runtime tests;
- examples/tutorials;
- Actor/distributed/abstract-runtime specifications; and
- programmer documentation.

This work is substantial but mechanically feasible.

### Cost of reintroduction

If future requirements again need stable routing across replaceable Actor
incarnations, reintroduction would require deciding and implementing at least:

- a stable target identity distinct from ActorRef;
- capability acquisition and transfer;
- identity/equality/hash semantics;
- lifetime ownership;
- routing selection;
- snapshot preservation across routing;
- local/remote routing behavior;
- the acceptance cutover;
- known pre-acceptance failure versus uncertainty;
- cancellation;
- termination behavior; and
- interaction with future membership/reconciliation.

The concrete implementation may differ, but a significant fraction of the
semantic problem would recur.

This makes removal materially different from deferring an unimplemented future
feature.

### Reversibility if KEEP is wrong

Candidate A does not claim the present abstraction is permanently correct.

If real distributed/scalable use later shows that Group identity, GroupRef
identity, routing boundaries or implementation architecture are wrong, Protos
may explicitly reopen the design and rewrite it.

The project prefers paying for such a rewrite when backed by real requirements
over paying removal plus likely reintroduction merely to minimize current line
count.

## Required adversarial answers

**What is the smallest solution for today's requirements?**

Actor/ActorRef is enough for programs that do not need Group. For programs that
choose Group, the current explicit acquisition plus send/request routing is the
smallest already exposed useful capability. No additional control plane is
required now.

**If Group is omitted today, can it be added later without breaking the model?**

Yes, but because ActorRef intentionally cannot retarget, adding stable service
identity later requires a new identity/capability layer. It is feasible but not
cheap and would revisit existing semantic work.

**What exactly would be rewritten later?**

At minimum identity/transfer/lifetime plus send/request routing, acceptance,
uncertainty and cancellation semantics, with additional integration for whatever
membership/control plane motivates the future feature.

**What current complexity could make us regret KEEP?**

Group request/send machinery must evolve with Actor messaging and transport;
Group adds conceptual identity surface; and an incorrect Group abstraction could
later constrain a better distributed model.

No current evidence shows those costs are severe enough to outweigh removal and
reintroduction risk.

## Compatibility and implementation consequences

Candidate A preserves current observable Core behavior.

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=NO
RUNTIME_CHANGE_REQUIRED=NO
PUBLIC_API_CHANGE_REQUIRED=NO
TEST_MIGRATION_REQUIRED=NO
IMPLEMENTATION_OWNER_REQUIRED=NO
```

No implementation Issue is created merely to preserve the status quo.

The required change is project-record reconciliation so the earlier AUD009-C4
removal classification cannot later be treated as current authority.

## Strongest argument against Candidate A

No production Protos library or tool currently needs GroupRef, and Group-aware
send/request duplicates substantial messaging machinery. The strongest advertised
future benefit — stable identity across membership replacement — is not yet
controllable through a public membership/reconciliation API.

If Group continues for a long period without real consumers while repeatedly
forcing expensive messaging/runtime work, Candidate B may become preferable.

D164 accepts that risk because the likely future need is structurally close to
what GroupRef already models, because ActorRef deliberately cannot absorb that
role, and because KEEP does not require completing the speculative distributed
control plane now.

## Intentionally deferred questions

D164 does not decide:

- public membership-management operations;
- desired cardinality;
- Group Controller representation;
- service-discovery API or naming;
- persistent/durable Group identity;
- Group policy representation;
- Cluster topology;
- distributed membership protocol;
- placement;
- autoscaling/capacity-demand signaling;
- Authority/fencing;
- Group persistence/recovery;
- broadcast/multicast API; or
- whether a future requirement justifies redesigning or replacing the current
  Group implementation.

## Ratified result

```text
D164_STATUS=RATIFIED
SELECTED_CANDIDATE=A
ACTOR_GROUP=KEEP
GROUP_REF=KEEP
ACTOR_GROUP_ACQUISITION=KEEP
GROUP_SEND_REQUEST_ROUTING=KEEP
PRE_ACCEPTANCE_REROUTING=KEEP
PUBLIC_MEMBERSHIP_API=REMAINS_ABSENT
PUBLIC_DISCOVERY_API=REMAINS_ABSENT
PUBLIC_DESIRED_CARDINALITY_API=REMAINS_ABSENT
PUBLIC_GROUP_CONTROLLER_API=REMAINS_ABSENT
NORMATIVE_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_RECONCILIATION_REQUIRED=NO
FUTURE_REDESIGN_ALLOWED=YES_BY_SEPARATE_APPROVED_DECISION
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
