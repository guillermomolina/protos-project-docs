# AUD009-C3 — Actor identity, messaging, lifecycle, and supervision complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#627`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88`

Closure revalidation revision:
`05a8754e55c1ce5cb44effc621f1b454f1fa9697`

The only Protos commit between the C3 evidence baseline and closure
revalidation changed Test Tool implementation/tests and implementation metadata.
It did not modify `spec/concurrency/ACTORS.md` or the Actor runtime/protocol
owners audited by C3.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#627`, issue comment
`5739421090`.

Owner approval provenance: `guillermomolina/protos#627`, issue comment
`5739431796`, 2026-09-19.

Derived decision routes:

- `D162 / guillermomolina/protos#630` — Actor fatal-failure policy authority
  model.
- `D163 / guillermomolina/protos#631` — Actor runtime-health and watchdog
  contract boundary.

Neither derived decision is selected by this audit record.

## Purpose and boundary

AUD009-C3 reviewed persistent Actor identity, creation, mailbox/turn execution,
communication, lifecycle/termination, supervision/failure policy, monitoring,
authority, transfer, and physical-locality boundaries.

C3 did not reopen the Future/task model already classified by C1, isolated P
already classified by C2, the open D159/D160/D161 decisions, or the distributed
Group/Node/Cluster control-plane as independent institutions.

C3 is an evidence/classification slice. It does not itself alter normative
semantics or implementation.

## Evidence summary

Repository-wide Protos-source search at the evidence baseline found no current
production `protos/lib` / `protos/tools` consumers of:

```text
Actor.spawn
Actor.current
ActorRef.send
ActorRef.request
ActorRef.stop
ActorRef.termination
```

Current language-level use is examples, tutorials, conformance and benchmarks.

That absence did not eliminate the Actor layer. PERF001-F retained
production-hosted Actor scaling evidence. Its `actor-fanout-requests` workload
reported approximate steady-median speedups:

```text
width 1   1.000x
width 2   2.170x
width 4   2.350x
width 8   2.347x
```

Actor therefore remains evidence-backed as a persistent isolated execution
institution even before Standard Library/tooling adoption.

## Final classification ledger

```text
Actor persistent isolated execution domain           KEEP
Actor incarnation identity                            KEEP
ActorRef                                               KEEP
Actor.spawn(moduleSpecifier,bindingName,args...)       KEEP
bootstrap by destination-loadable module identity     KEEP
explicit initialization-value transfer                KEEP
INITIALIZING -> READY cutover                          KEEP
Actor.current()                                        KEEP
RootActor                                              KEEP

implicit runtime event loop                            KEEP
one Actor-local Protos segment at a time               KEEP
reentrancy only at explicit suspension                 KEEP
ordinary Protos dispatch against behavior              KEEP
stable behavior-object reference                       KEEP
same-sender/same-destination FIFO                      KEEP
weak runnable/admission fairness                       KEEP

ActorRef.send                                          KEEP
SendOperation                                           KEEP
SendOperation.cancel                                    KEEP
SendOperation.retry                                     KEEP
ActorRef.request                                        KEEP
request Future                                          KEEP
RequestOutcomeUncertain                                 KEEP
message snapshot before delivery path                  KEEP
pass-by-value Actor transfer                            KEEP
NonTransferableValue                                    KEEP
acceptance boundary                                     KEEP
no transparent replay after acceptance                 KEEP
bounded mailbox / end-to-end backpressure              KEEP

ActorRef.stop()                                         KEEP
irreversible graceful-stop cutover                     KEEP
stop idempotence                                       KEEP
cancellation/drain/cleanup before TERMINATED           KEEP
ActorRef.termination()                                 KEEP
fresh independent observation Future                   KEEP
known termination != unreachable/unknown               KEEP
Actor lifetime independent of ActorRef reachability    KEEP
unhandled Actor-turn Error is fatal                    KEEP
Actor identity ends permanently at termination         KEEP
no ActorRef retargeting to replacement                 KEEP
no transparent message replay after failure            KEEP

fixed non-root fatal-failure policy                    KEEP
RootActor fatal failure -> Process termination         KEEP
public configurable supervisor/policy object           ABSENT / RETAIN ABSENCE
semantic failure-authority relationship                KEEP (D162 supersedes C3 removal classification)

ActorRef explicit communication capability             KEEP
no ambient creator reverse capability                  KEEP
explicit capability provisioning at spawn              KEEP
no shared mutable Protos identity across Actors         KEEP
non-transferable resources fail explicitly             KEEP
no implicit resource auto-proxy                        KEEP
same-host/shared-memory transport invisible            KEEP
transport selection/switching invisible                KEEP
physical-locality discovery non-public                 KEEP
non-local return never crosses Actor boundary          KEEP
dynamic handlers never cross Actor boundary            KEEP
foreign mutable state cannot bypass Actor isolation    KEEP
blocking foreign offload cannot create hidden reentry  KEEP

mandatory Core Actor runtime-health/watchdog fast path REMOVE_NOW_RECONSIDER_LATER
mandatory O(1) non-blocking always-on health contract  REMOVE_NOW_RECONSIDER_LATER

Closure-taking Actor constructor                       ABSENT / RETAIN ABSENCE
public Actor ID                                         ABSENT / RETAIN ABSENCE
public mailbox/receive API                              ABSENT / RETAIN ABSENCE
public scheduler/worker handle                         ABSENT / RETAIN ABSENCE
public parentActor/creator link                        ABSENT / RETAIN ABSENCE
public restart operation                               ABSENT / RETAIN ABSENCE
transparent replay/exactly-once                       ABSENT / RETAIN ABSENCE
reachability-based Actor GC                            ABSENT / RETAIN ABSENCE
public physical locality/transport controls            ABSENT / RETAIN ABSENCE
shared mutable Protos memory between Actors            ABSENT / RETAIN ABSENCE
implicit resource proxying                             ABSENT / RETAIN ABSENCE
hidden Actor suspension/preemption                     ABSENT / RETAIN ABSENCE
```

No C3 mechanism is classified `REMOVE_PERMANENTLY`.

## Actor identity and creation remain justified

Actor owns a combination that neither C/Future nor P provides:

```text
persistent isolated mutable state
stable incarnation identity
mailbox communication
independent lifecycle
parallel execution relative to other Actors
```

Replacing Actor with Future/task would lose isolated persistent lifetime.
Replacing Actor with P would lose persistent identity, mailbox and state
continuity.

Module/binding bootstrap also survives. A Closure-taking spawn would require
transferring caller lexical context or inventing a special transferable Closure
kind. Code identity plus explicit values preserves the Actor isolation model and
supports local or remote placement without transporting execution state.

`Actor.current()` remains the minimal explicit way to obtain self capability,
including when a creator deliberately provisions its own ActorRef to another
Actor. Removing it would pressure Core toward ambient `selfActor` or implicit
creator/parent authority.

RootActor remains the smallest uniform initial execution/failure domain and may
be optimized away physically when no additional Actor machinery is needed.

Classification: **KEEP / RETAIN ABSENCE**.

## Actor turns and communication remain justified

One Actor-local Protos segment executes at a time, with reentrancy only at
explicit suspension boundaries. This preserves race-free local mutable state
without exposing locks, atomics, memory barriers or implicit hidden suspension.

`send()` and `request()` remain distinct. Request creates reply authority and
a Future whose normal result is the transferable handler result. Send is
one-way and intentionally ignores that result.

`SendOperation` remains distinct from Future because delivery can be uncertain
without Protos being able to assert either accepted or definitely-not-delivered.
Its public Core surface stays deliberately small:

```text
cancel()
retry()
```

Explicit retry reuses the original logical message snapshot and cannot be
silently replaced by automatic retry because an uncertain attempt may already
have effects.

Bounded mailbox/end-to-end backpressure remains necessary for scalability.
Unbounded intermediate queues would only move rather than remove the resource
problem.

Classification: **KEEP**.

## Lifecycle and monitoring remain justified

`stop()` is lifecycle authority/action. `termination()` is lifecycle
observation. They remain orthogonal and should not be conflated.

`termination()` reuses ordinary Future semantics rather than creating a
monitor-handle/event-stream family. Independent observers can monitor a concrete
incarnation without acquiring stop or failure-policy authority.

Actor lifetime remains explicit rather than reachability-based. Concrete
ActorRefs may be remote or in flight; proving last-reference death would require
global coordination or implementation-dependent reference tracking.

Unhandled Error escape remains fatal to the affected Actor incarnation because
continuing after an unhandled turn failure could preserve partially mutated
Actor-local state without a defined recovery path.

Classification: **KEEP**.

## Fixed failure policy remains; semantic failure authority does not

The following observable Core policy remains:

```text
non-root unhandled fatal failure
    -> that Actor incarnation terminates

RootActor unhandled fatal failure
    -> containing Process terminates
```

Core additionally defines no automatic non-root replacement, no escalation to
creator, no sibling/subtree restart, and no public configurable supervisor or
policy object.

The separate statement that **every Actor has a failure authority** is classified:

```text
REMOVE_NOW_RECONSIDER_LATER
```

Core exposes no failure-authority value, selector, capability, supervisor Actor,
policy callback or mutable supervision tree.

The implementation currently realizes the observable fixed policy through a
direct root/non-root branch in the Process runtime rather than an independently
represented policy-bearing authority object.

The semantic relationship therefore pre-installs terminology and architecture
for a richer supervision model that Core deliberately does not expose.

Removing the institution does not remove deterministic failure handling. The
fixed consequences remain directly normative.

### Required AUD009 routing fields

```text
FEATURE=semantic Actor failure-authority relationship
CURRENT_AUTHORITY=spec/concurrency/ACTORS.md
CURRENT_IMPLEMENTATION=direct Actor/Process fatal-failure routing
PUBLIC_CONSUMERS=NONE

PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER

SEMANTIC_DECISION_OWNER=D162 / guillermomolina/protos#630
IMPLEMENTATION_REMOVAL_OWNER=TBD_AFTER_D162_RATIFICATION

RECONSIDERATION_TRIGGER=
    concrete supervision/failure-reporting capability requiring explicit
    ownership of recovery policy, restart/replacement authority, transferable
    failure reports, or policy composition

RECONSIDERATION_SCOPE=
    design failure-policy authority from the then-current Actor/Group/Process
    model; do not preselect Supervisor Actor, supervision tree, creator authority,
    or the current internal failure-authority vocabulary

CONFIDENCE=HIGH
```

### D162 ratification reconciliation

D162 / #630 subsequently tested the C3 removal classification against the
current `AGENTS.md` retrospective-removal rule, including the concrete cost of
removing an already integrated mechanism versus the continuing cost of leaving
it in place.

The project owner ratified D162 Candidate A on 2026-09-19:

```text
semantic Actor failure-authority relationship    KEEP
public configurable supervision API              REMAINS ABSENT
fixed Core fatal-failure policy                  UNCHANGED
```

The decisive additional evidence was that the retained semantic relationship
does not require a distinct Actor, value, capability, mailbox, policy object,
allocation, supervision tree, or extra runtime work, while removing it would
require normative/documentation reconciliation without removing the fatal-failure
routing already required by the fixed root/non-root policy.

Therefore the C3 `REMOVE_NOW_RECONSIDER_LATER` classification above remains
historical audit evidence but is **superseded by D162** and must not be treated as
current removal authority.

Durable decision record:
`docs/project/decisions/language/D162_ACTOR_FATAL_FAILURE_POLICY_AUTHORITY_MODEL.md`.

## Runtime Health/Watchdog mandatory Core contract is a removal candidate

ACTORS §30 currently requires inexpensive always-on health information and a
mandatory O(1), non-blocking, no-global-coordination fast path, with possible
information including lifecycle state, progress epoch, mailbox depth, failure
count and root/failure-authority ownership.

C3 found no public portable Core API, no Protos-source consumer, no current
observable Actor result that depends on this mandated health path, and no direct
implementation surface corresponding to most of the proposed metric set.

Approved classification:

```text
mandatory Core Actor runtime-health/watchdog fast path
    REMOVE_NOW_RECONSIDER_LATER

mandatory O(1), non-blocking always-on health-information contract
    REMOVE_NOW_RECONSIDER_LATER
```

This does not remove internal lifecycle, scheduler, counter, epoch, probe,
tombstone or diagnostic state needed by retained semantics or implementation.
It questions only the anticipatory portable architecture obligation.

### Required AUD009 routing fields

```text
FEATURE=mandatory Core Runtime Health/Watchdog architecture
CURRENT_AUTHORITY=spec/concurrency/ACTORS.md section 30
CURRENT_PUBLIC_CONSUMERS=NONE
CURRENT_PROTOS_SOURCE_CONSUMERS=NONE

PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER

SEMANTIC_DECISION_OWNER=D163 / guillermomolina/protos#631
IMPLEMENTATION_REMOVAL_OWNER=TBD_AFTER_D163_RATIFICATION

RECONSIDERATION_TRIGGER=
    concrete runtime-health/observability/control-plane consumer requiring
    cross-implementation guarantees for probe cost, availability, progress,
    mailbox or failure metrics

RECONSIDERATION_SCOPE=
    define health/observability guarantees from the real consumer and operating
    model; do not preselect the current O(1) fast-path vocabulary or metric set

CONFIDENCE=HIGH
```

## Authority and locality protections remain

ActorRef stays an explicit communication capability. Creation genealogy grants
no reverse capability automatically. Spawn transfers only explicitly
provisioned values/capabilities.

Mutable Protos identity never crosses Actor isolation. Non-transferable live
resources fail rather than being silently replaced with proxies, reopened
resources or duplicated host handles.

Same-host/shared-memory transport, transport switching and physical-locality
discovery remain implementation machinery when they preserve the same logical
Actor transfer, ordering, acceptance, failure and identity contract.

Non-local return homes and dynamic error-handler stacks remain execution-local
control state and never cross Actor boundaries.

Classification: **KEEP**.

## Strongest attempted removals

```text
remove Actor layer entirely
    rejected -> KEEP

replace Actors with Future/C
    rejected -> KEEP

replace Actors with P
    rejected -> KEEP

Closure-taking Actor spawn
    rejected -> KEEP module/binding bootstrap

remove Actor.current
    rejected -> KEEP

remove send and use request-only
    rejected -> KEEP both

replace SendOperation with ordinary Future
    rejected -> KEEP SendOperation

remove bounded mailbox/backpressure
    rejected -> KEEP

merge stop and termination observation
    rejected -> KEEP separate action/observation

use ActorRef reachability as Actor lifetime
    rejected -> RETAIN ABSENCE

add public configurable supervision now
    rejected -> RETAIN ABSENCE

semantic failure-authority relationship
    survives removal -> REMOVE_NOW_RECONSIDER_LATER

mandatory Core Runtime Health/Watchdog institution
    survives removal -> REMOVE_NOW_RECONSIDER_LATER
```

## Boundary handoffs

- **D162 / #630** owns the semantic failure-authority decision.
- **D163 / #631** owns the Core Runtime Health/Watchdog contract decision.
- **D159 / #623** independently owns Future detachment.
- **D160 / #625** independently owns parallel collection algorithm placement.
- **D161 / #626** independently owns ByteRegion/writable partitioning.
- Distributed ActorGroup/Node/Cluster institutions remain with their distributed
  runtime owner and any later AUD009 partition that audits them directly.
- Detailed I/O authority/lifecycle remains AUD009-D.
- Concrete Actor scheduler/backend/runtime complexity remains AUD009-G.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#627
CHECKPOINT_COMMENT=5739421090
APPROVAL_COMMENT=5739431796
DATE=2026-09-19

ACTOR_CORE_MODEL=KEEP
ACTOR_MESSAGING=KEEP
ACTOR_LIFECYCLE=KEEP
ACTOR_MONITORING=KEEP
ACTOR_ISOLATION_AUTHORITY_BOUNDARY=KEEP

SEMANTIC_FAILURE_AUTHORITY=REMOVE_NOW_RECONSIDER_LATER
FAILURE_AUTHORITY_DECISION=D162 / guillermomolina/protos#630

CORE_RUNTIME_HEALTH_WATCHDOG=REMOVE_NOW_RECONSIDER_LATER
HEALTH_WATCHDOG_DECISION=D163 / guillermomolina/protos#631

NORMATIVE_CHANGE_AUTHORIZED_BY_C3=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_C3=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_REVISION=a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88
CLOSURE_REVALIDATION=05a8754e55c1ce5cb44effc621f1b454f1fa9697
REVALIDATION_OVERLAP_WITH_C3_OWNERS=NONE
D162_NATIVE_PARENT=#627 PASS
D163_NATIVE_PARENT=#627 PASS
D162_PROJECT_ROUTING=PASS
D163_PROJECT_ROUTING=PASS
REMOVAL_ROUTES=D162,D163
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_C3_CLASSIFICATION=COMPLETE
```

AUD009-C3 is complete once this durable record and required live GitHub closure
postconditions are verified.
