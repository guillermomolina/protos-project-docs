# D162 — Actor fatal-failure policy authority model

Status: **RATIFIED — Candidate A (retain semantic failure authority)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#630`
Trigger: AUD009-C3 / `guillermomolina/protos#627`
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`
Project-record base: `ef02556fc3f1f0b2d8fc10de7312c78b76ab389e`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D162 selects **Candidate A**.

Core retains the semantic statement that every Actor has a failure authority.

The selected boundary is deliberately abstract:

```text
every Actor
    -> has one failure authority responsible for applying fatal-failure policy

failure authority
    -> is a semantic responsibility
    -> need not be an Actor
    -> need not be a language-visible object
    -> need not be a separately allocated runtime object
    -> need not imply a mailbox, callback, policy object, or supervision tree
```

Core v0.1 retains its fixed observable policy:

```text
non-root unhandled fatal Actor failure
    -> that Actor incarnation terminates

RootActor unhandled fatal failure
    -> containing Process terminates
```

No automatic non-root replacement, escalation to the creator, sibling/subtree
restart, or public configurable supervision/failure-policy API is introduced.

Creation genealogy grants no failure authority to ordinary Protos code.
ActorGroup desired-state reconciliation remains semantically distinct from
fatal-failure policy.

## Approval provenance

The complete D162 comparative packet was presented to the project owner,
including repository evidence, prior art, candidates A through D, the
twelve-dimension scoring matrix, falsification results, incremental-design
analysis, removal/reintroduction cost, compatibility consequences, and the
strongest argument against retention.

The review explicitly applied the owner-requested retrospective criterion that
lack of current use or an appearance of overengineering is not by itself enough
to justify removal. Before removing an already implemented or specified
mechanism, the continuing cost of keeping it must be compared with the actual
cost and benefit of removing it and with the cost of adding it back later.

The recommendation presented for approval was:

```text
D162 = Candidate A — KEEP semantic failure authority
```

The project owner then explicitly approved that pending recommendation in the
active interaction on 2026-09-19:

```text
aoribada
```

The spelling is preserved as approval provenance from the interaction; in
context it was the direct response to the exact pending Candidate A / KEEP
approval request.

```text
SELECTED_CANDIDATE=A
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## GITHUB021 invariant consistency

Candidate A preserves every fixed D162 invariant:

```text
ACTOR_INCARNATION_IDENTITY                         PRESERVED
UNHANDLED_ACTOR_TURN_ERROR_IS_FATAL                PRESERVED
NON_ROOT_FATAL_FAILURE_TERMINATES_INCARNATION      PRESERVED
ROOTACTOR_FATAL_FAILURE_TERMINATES_PROCESS         PRESERVED
NO_AUTOMATIC_NON_ROOT_REPLACEMENT                  PRESERVED
NO_AUTOMATIC_ESCALATION_TO_CREATOR                 PRESERVED
NO_SIBLING_OR_SUBTREE_RESTART                      PRESERVED
NO_PUBLIC_CONFIGURABLE_SUPERVISOR_POLICY           PRESERVED
CREATION_GENEALOGY_GRANTS_NO_AMBIENT_AUTHORITY     PRESERVED
ACTORGROUP_RECONCILIATION_IS_DISTINCT              PRESERVED
```

Observable semantic delta:

```text
NONE
```

Candidate A retains the already-normative relationship and therefore does not
reopen or alter the existing Actor failure behavior.

No materially new semantic or architectural consequence was introduced after
the packet presented for approval.

## Repository evidence

At the evidence revision, failure authority is not merely an unused planned API.

It is already integrated into the normative Actor model:

- `spec/concurrency/ACTORS.md` defines supervision/failure authority and the
  fixed Core policy;
- `spec/concurrency/DISTRIBUTED_RUNTIME.md` defines failure authority as a
  pay-as-you-grow semantic role and keeps Group reconciliation distinct;
- `spec/runtime/ABSTRACT_RUNTIME.md` references failure-authority consequences;
- programmer-facing Actor documentation explains the fixed failure-authority
  policy.

The runtime already has a natural centralized failure-routing boundary:

```text
ProtosActor.failForRuntime(failure)
    -> ProtosProcessRuntime.actorFatalFailureForRuntime(actor, failure)
```

For a Process-bound Actor, that runtime entry distinguishes only:

```text
actor != rootActor
    -> terminate Actor incarnation

actor == rootActor
    -> terminate Process
```

The retained semantic relationship does not currently require:

- a distinct supervisor Actor;
- a distinct failure-authority value;
- a selector or public capability;
- a mutable supervision tree;
- a policy callback;
- one allocation per Actor;
- an extra mailbox;
- distributed coordination merely for local failure handling; or
- runtime work when no richer policy exists.

The current implementation can therefore continue to realize Candidate A with
the same direct root/non-root branch.

## Reconsideration of AUD009-C3

AUD009-C3 had classified the semantic failure-authority relationship as:

```text
REMOVE_NOW_RECONSIDER_LATER
```

D162 was opened precisely because that audit classification did not authorize
semantic removal.

The D162 investigation found that the C3 removal case established only that:

- no public configurable supervision API exists;
- no independently represented authority object is required by the current
  runtime; and
- the fixed observable policy can be stated directly.

Those facts prove removability, but they do not establish that removal is the
lower-cost or more Protos-aligned choice.

The decisive additional evidence is that retaining the relationship imposes
essentially no runtime or public-API tax, whereas removing it requires normative
and documentation reconciliation while producing no observable simplification
and no meaningful runtime simplification.

The C3 classification is therefore superseded for this mechanism:

```text
AUD009-C3 preliminary classification
    semantic failure-authority relationship
    -> REMOVE_NOW_RECONSIDER_LATER

D162 ratified classification
    semantic failure-authority relationship
    -> KEEP
```

The rest of the approved C3 packet is unaffected.

## Comparative evidence

The investigation compared materially different failure-ownership models.

### Erlang / OTP

OTP makes supervision explicit when supervision carries real policy and
lifecycle consequences: supervisors own child specifications, restart strategy,
restart intensity, and child lifecycle coordination.

Contribution to D162: explicit supervision is justified when the institution
actually supplies configurable recovery behavior. Protos Candidate A does not
preinstall that public institution.

### Akka

Akka supervision gives parent/supervisor relationships concrete recovery
semantics such as stop or restart, while typed actor behavior keeps explicit
failure handling tied to declared supervision behavior.

Contribution to D162: a failure-policy owner can be semantically meaningful
without requiring Protos to adopt Akka's parent/child supervision tree or
restart model.

### Orleans

Orleans separates logical grain identity from transient activation failure and
lets runtime/application mechanisms recreate activations when needed rather than
making every logical actor expose a supervisor object.

Contribution to D162: deterministic failure behavior does not require a public
supervisor entity.

### Swift structured concurrency

Swift task structure makes ownership and failure/cancellation relationships
explicit where lexical task ownership actually carries lifecycle consequences.

Contribution to D162: semantic ownership relationships are useful when they
state responsibility, but they need not imply a separately visible runtime
object.

### Go

Go can define fixed panic/failure consequences without a generic public
supervision relation.

Contribution to D162: Candidate B is technically coherent; failure authority is
not required merely to implement deterministic fatal-failure behavior.

The prior art therefore does not force one universal model. The deciding Protos
question is whether the existing abstract responsibility costs enough to justify
removal.

## Candidate set

### Candidate A — retain semantic failure authority

Selected.

Keep one abstract semantic responsibility for applying Actor fatal-failure
policy. Do not require a dedicated supervisor object or configurable policy
surface.

### Candidate B — remove failure authority from Core semantics

Rejected.

The fixed root/non-root consequences can be specified directly, so this
candidate is correct and implementable. However, removal buys almost entirely a
vocabulary reduction while requiring normative/documentation reconciliation and
discarding an already useful distinction between creation genealogy and
failure-policy responsibility.

### Candidate C — retain only an internal/runtime failure-routing concept

Rejected.

This preserves the implementation seam while deleting the corresponding
portable semantic responsibility. It is a plausible simplification, but the
project would pay a normative migration cost while the runtime would retain
almost the same conceptual boundary. If richer supervision later needs an
explicit policy owner, substantially the same semantic relation may then need
to be reintroduced.

### Candidate D — introduce a public supervision/failure-policy capability now

Rejected.

No current requirement justifies a public supervisor Actor, policy object,
callback, restart strategy, or supervision tree. Such a facility would impose
new authority, lifetime, failure-reporting, replacement, ordering, and
coordination semantics before a real use case requires them.

## Comparative scoring

Scores use 1–5. Confidence is shown as H/M.

| Criterion | A | B | C | D |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 5H | 5H | 4M |
| Protos alignment | 5H | 5H | 5H | 2H |
| Present-need proportionality | 5H | 5H | 5H | 1H |
| Incremental growth | 5H | 4M | 5H | 5H |
| Future-option resilience | 5M | 4M | 5H | 3M |
| Scalability | 5M | 5H | 5H | 4M |
| Conceptual simplicity | 4M | 5H | 5H | 2H |
| Portability / implementation freedom | 5H | 5H | 5H | 3M |
| Runtime / resource cost | 5H | 5H | 5H | 2H |
| Failure / operability | 5H | 4M | 5H | 5M |
| Deferral / reversibility / migration | 5H | 4M | 3H | 3M |
| Evidence maturity / implementation risk | 5H | 5H | 5H | 4M |

The arithmetic scores are comparison aids, not decision authority.

Candidate D carries an overengineering red flag because it adds present public
machinery for absent requirements.

Candidate B/C carry a retrospective-removal concern: the simplification benefit
is mostly terminological while the removal itself has real reconciliation cost.

## Incremental-design analysis

### Smallest sufficient solution

The smallest solution that satisfies current requirements is:

```text
abstract failure-policy responsibility
    +
fixed Core consequences
    +
no public configurable supervision facility
```

Candidate A does not require the implementation to materialize anything beyond
the routing already needed to apply the fixed consequences.

### Pay for what you need

Current programs do not pay for a supervisor Actor, policy object, mailbox,
callback, restart tree, or extra distributed coordination.

The continuing cost of Candidate A is primarily one semantic concept and its
documentation.

### Grow as you need

A future supervision facility may make failure authority observable or
configurable and define richer policy. It can do so without changing the
fundamental statement that one entity is responsible for applying the applicable
failure policy.

Candidate A does not reserve a specific future Supervisor Actor, parent tree,
policy enumeration, restart strategy, failure-report representation, or API.

### Cost of removal now

Removing the relationship would require reconciling at least the Actor normative
specification, distributed-runtime wording, abstract-runtime references,
programmer documentation, and runtime terminology.

That work would not remove the actual fatal-failure routing required by the
retained root/non-root policy.

No parser cost, object allocation, synchronization path, mailbox, public selector,
or per-Actor data structure is removed by Candidate B.

### Cost of adding the concept later

If Candidate B/C were selected and future supervision later required explicit
ownership of recovery policy, Protos would need to define that responsibility
again and reconcile it with creation genealogy, ActorGroup reconciliation,
failure reporting, replacement identity, and Process/RootActor failure.

That is not necessarily a foundational compatibility break, but it is
non-zero redesign/reintroduction cost for a concept whose present carrying cost
is already low.

## Required falsification results

1. Removing failure authority does not make the current root/non-root observable
   behavior underspecified if those consequences are specified directly.

2. ActorGroup desired-state reconciliation does not depend on failure authority;
   the two responsibilities remain independent.

3. Runtime diagnostics and Process termination do not require a portable,
   separately represented authority object.

4. A future public supervision facility could still be added after removal
   without necessarily breaking Actor identity or the current fixed failure
   consequences.

5. Retaining the invisible relationship has real value only if it remains an
   abstract responsibility and not speculative machinery. Candidate A satisfies
   that condition.

6. Adding public supervision now fails pay-for-what-you-need because no current
   requirement justifies its additional authority/lifecycle surface.

These results make B/C viable, but do not establish a benefit large enough to
justify removing the already integrated abstraction.

## Future-scenario stress

Candidate A remains compatible with:

- large numbers of Actors;
- multicore execution;
- distributed Actor placement;
- richer Group control/reconciliation;
- future failure-reporting facilities;
- future configurable supervision;
- alternative schedulers and runtimes; and
- implementations that compile the semantic responsibility down to a direct
  local branch.

The plausible future requirement most likely to stress Candidate A is a richer
policy model where more than one component participates in recovery or policy
composition.

Escape path: a future Dxxx may refine how policy authority is represented,
delegated, or composed. D162 does not freeze the authority as one particular
object, Actor, tree, or protocol.

## Compatibility and implementation consequences

Candidate A preserves current observable Core semantics.

Therefore:

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=NO
RUNTIME_CHANGE_REQUIRED=NO
PUBLIC_API_CHANGE_REQUIRED=NO
TEST_MIGRATION_REQUIRED=NO
IMPLEMENTATION_OWNER_REQUIRED=NO
```

No `Ixxx` is created merely to reimplement the status quo.

The only required reconciliation is non-normative project history: AUD009-C3's
preliminary removal classification must be marked as superseded by this ratified
decision so final AUD009 reconciliation cannot treat it as current authority.

## Strongest argument against Candidate A

Core can express every current observable failure rule without naming a
failure-authority relationship.

Because no program can obtain or configure that authority today, keeping the
term adds conceptual surface that an independent implementer must understand
even though the implementation may reduce it to one root/non-root branch.

If that semantic vocabulary begins to constrain implementations, invite
speculative supervisor architecture, or require maintenance disproportionate to
its role, Candidate B/C should be reconsidered.

D162 accepts that conceptual cost because it is presently small, already
integrated, separates failure-policy responsibility cleanly from creation
genealogy and Group reconciliation, and costs less to retain than to remove
without producing speculative runtime machinery.

## Intentionally deferred questions

D162 does not decide:

- a public supervision API;
- Supervisor Actor identity or hierarchy;
- restart/replacement policy configuration;
- restart-intensity limits;
- one-for-one versus one-for-all strategy;
- failure-report or Error-snapshot representation;
- remote failure-report transport;
- policy delegation or composition;
- whether a future failure authority is represented as an ordinary Protos
  capability; or
- how future supervision interacts with distributed placement beyond the
  retained current invariants.

Any such observable choice requires separate evidence and approval.

## Ratified result

```text
D162_STATUS=RATIFIED
SELECTED_CANDIDATE=A
SEMANTIC_FAILURE_AUTHORITY=KEEP
PUBLIC_SUPERVISION_API=REMAINS_ABSENT
CORE_FATAL_FAILURE_POLICY=UNCHANGED
CREATION_GENEALOGY_AUTHORITY=UNCHANGED
ACTORGROUP_RECONCILIATION_BOUNDARY=UNCHANGED
NORMATIVE_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_RECONCILIATION_REQUIRED=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
