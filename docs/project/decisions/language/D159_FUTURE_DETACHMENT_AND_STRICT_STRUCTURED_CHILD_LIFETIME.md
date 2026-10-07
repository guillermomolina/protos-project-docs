# D159 — Future detachment and strict structured child lifetime

Status: **RATIFIED — Candidate B (remove Core detachment with no immediate replacement)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#623`
Trigger: AUD009-C1 / `guillermomolina/protos#622`
Protos evidence revision: `ecf563ed01275929d5b85330e8e6259cc85d73d8`
Project-record base: `e8780ac4766ad22f273f900e45937bbb641827a2`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D159 selects **Candidate B**.

Core removes the public `Future.detach()` operation. Task-backed child work
remains owned by the creating asynchronous task scope for the child's complete
lifetime.

The selected boundary is:

```text
Future.detach()                              REMOVE_NOW_RECONSIDER_LATER

task-backed child structured ownership      KEEP
ownership for complete child lifetime       STRICT

Future eventual-result family               KEEP
hidden Task execution/lifetime concept      KEEP
Actor lifetime                              KEEP
cooperative cancellation                    KEEP
Future.then downstream ownership            KEEP
no implicit ownership change on return      KEEP
no public Task/scope                        KEEP ABSENT

background / independent Actor-local work   NOT ADDED NOW
future explicit creation-time escape        DEFERRED
```

D159 does not select a replacement background-work API and does not introduce
a public Task, structured scope object, daemon-task family, or new Actor-like
lifetime category.

## Approval provenance

The complete D159 comparative packet was presented to the project owner,
including repository evidence, the prior-art survey, candidates A through E,
the twelve-dimension scoring matrix, failure modes, deferral/reversibility
analysis, future-scenario stress, compatibility consequences, and the strongest
argument against removal.

The recommendation explicitly distinguished D159 from D158: lack of current
production use was not treated as sufficient by itself. The decisive additional
fact was that `Future.detach()` is a public post-creation mutation of structured
lifetime ownership through an eventual-result handle.

The project owner then explicitly approved Candidate B in the active interaction
on 2026-09-19:

```text
ok aprobada
```

```text
SELECTED_CANDIDATE=B
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## GITHUB021 invariant consistency

Candidate B preserves the fixed D159 authority:

```text
D045_TASK_SCOPED_STRUCTURED_OWNERSHIP         PRESERVED
FUTURE_EVENTUAL_RESULT_FAMILY                 PRESERVED
HIDDEN_TASK_EXECUTION_CONCEPT                 PRESERVED
ACTOR_LIFETIME_BOUNDARY                       PRESERVED
NO_PUBLIC_TASK_OR_SCOPE                       PRESERVED
COOPERATIVE_CANCELLATION                      PRESERVED
ORDINARY_FUTURE_OUTCOME_SEMANTICS             PRESERVED
FUTURE_THEN_DOWNSTREAM_OWNERSHIP               PRESERVED
NO_IMPLICIT_OWNERSHIP_CHANGE_ON_VALUE_ESCAPE  PRESERVED
P_PARALLEL_ISOLATION                          UNAFFECTED
IO_PRODUCER_CUSTODY                           UNAFFECTED
CANCELLATION_COMMITMENT                       UNAFFECTED
```

Approved observable delta:

```text
pending task-backed Future may remove its structured parent edge
    YES -> NO

Future.detach()
    PRESENT -> ABSENT

non-task-backed / terminal detach no-op semantics
    PRESENT -> ABSENT
```

No owner-approved invariant is contradicted.

## Repository evidence

At the evidence revision, `Future.detach()` is a visible standard Future
selector implemented by `ProtosStandardFutureProtocol` and
`ProtosFutureValue.detach()`.

`ProtosFutureValue` carries dedicated `detached` state. For a pending
task-backed Future, `detach()` obtains the producer Task and calls
`detachFromParent()`.

`ProtosTask.detachFromParent()` removes the current parent relationship and
causes the former parent to remove that child from its structured child set.

The parent/child machinery itself is not removable: structured ownership,
child-drain behavior, task terminalization, and Actor-local task lifetime still
require it. D159 therefore does not claim a large runtime simplification or a
meaningful speed improvement.

The continuing gain is semantic and architectural: the ownership edge of a
task-backed child is no longer publicly mutable after child creation through the
Future result handle.

Repository search at the C1 baseline and re-review found no production
`.detach()` use under `protos/lib` or `protos/tools`. Current guest occurrences
are conformance/control fixtures, especially cancellation and cleanup tests.

That absence of production use is supporting compatibility evidence only; it is
not the sole reason for the decision.

## Comparative evidence

The investigation compared materially different concurrency/lifetime models:

- Java structured concurrency: subtasks belong to an explicit structured scope
  and cannot outlive that scope merely by mutating a result handle.
- Kotlin coroutines: structured concurrency is the default, while independent
  work is selected through a distinct creation context rather than by detaching
  a child Future afterward.
- Swift concurrency: detached/unstructured work is created explicitly as such,
  distinct from structured child tasks.
- Python asyncio: `TaskGroup` supplies structured lifetime while separately
  created tasks provide unstructured lifetime.
- Tokio/Rust async: spawned tasks are independent from the caller's lexical
  lifetime; dropping a join handle does not retroactively convert a structured
  child because the creation model is already unstructured.
- Go: goroutines are independently created and structured cancellation is added
  explicitly through other mechanisms.
- Erlang/BEAM: process lifetime is a separate concurrent identity/lifetime model
  with explicit linking/supervision relationships.

The relevant cross-system pattern is not that independent work must be absent.
It is that when independent lifetime exists, making that choice at work creation
is generally clearer than converting an already-structured child afterward via
its result handle.

## Candidate set

### Candidate A — retain `Future.detach()`

Retains current semantics and compatibility.

Rejected because it allows an eventual-result handle to mutate producer lifetime
ownership after creation. This weakens locality of the otherwise strict
task-scoped ownership model and keeps state-dependent no-op semantics for
non-task-backed and terminal Futures.

### Candidate B — remove Core detachment with no immediate replacement

Selected.

It gives task-backed structured ownership a simple complete-lifetime invariant
and removes post-creation lifetime mutation from Future.

### Candidate C — move detachment outside Core

Rejected because an ordinary Standard Library operation cannot honestly remove
a hidden Task parent without privileged runtime authority. Merely moving the
selector's spelling would not remove the semantic institution.

### Candidate D — introduce another explicit lifetime abstraction now

Rejected because no current production requirement justifies a new public
background-job, Task, scope, or lifetime institution.

### Candidate E — explicit independent work selected at creation

Identified as a credible future direction, but not selected now.

If real Actor-local background-work requirements appear, a creation-time
independent-work primitive or abstraction is cleaner than post-creation
`Future.detach()`. Current evidence does not justify installing it in advance.

## Comparative scoring

Scores use 1–5. Confidence is shown as H/M.

| Criterion | A | B | C | D | E |
| --- | ---: | ---: | ---: | ---: | ---: |
| Correctness / invariants | 4H | 5H | 2H | 4M | 5H |
| Protos alignment | 3H | 5H | 2H | 2M | 4H |
| Present-need proportionality | 2H | 5H | 2H | 1H | 2H |
| Incremental growth | 4M | 5H | 3M | 3M | 5H |
| Future-option resilience | 4M | 5H | 3M | 4M | 5H |
| Scalability | 3M | 5H | 3M | 4M | 4H |
| Conceptual simplicity | 3H | 5H | 2H | 2H | 4H |
| Portability / implementation freedom | 4H | 5H | 3M | 3M | 5H |
| Runtime / resource cost | 4H | 5H | 3M | 3M | 4H |
| Failure / operability | 3H | 5H | 2M | 3M | 4H |
| Deferral / reversibility / migration | 4M | 4H | 3M | 2M | 4H |
| Evidence maturity / implementation risk | 4H | 5H | 2M | 3M | 5H |
| Total / 60 | 42 | 59 | 30 | 34 | 51 |

The arithmetic total is not the decision authority. Candidate E has strong
architectural scores but carries an overengineering red flag because the
required independent Actor-local background-work use case does not currently
exist.

## Why Candidate B was selected

The strongest reason is not source-count reduction and not the fact that current
production code does not call `detach()`.

The selected invariant is:

```text
task-backed child created in task scope
    -> remains owned by that task scope until terminal
```

That invariant is simpler, more local, and easier to reason about under
cancellation, cleanup, child draining, and Actor termination than an ownership
edge that can later disappear because a holder of the child's Future invokes
`detach()`.

`Future` remains the result/cancellation/observation handle. It no longer doubles
as authority to rewrite producer lifetime ownership.

No meaningful parser or runtime performance improvement is claimed.

## Strongest argument against Candidate B

The real lost capability is Actor-local background work.

Without `detach()`, code running inside one asynchronous task cannot create a
task-backed child, let the parent terminalize, and allow that child to continue
inside the same Actor domain. Creating another Actor is not semantically
equivalent because it changes identity, isolation, communication, and lifetime.

That capability is genuine even though no production consumer currently uses it.

The decision accepts that loss because current evidence does not justify keeping
a permanent post-creation ownership escape hatch, and because the implementation
already supports Actor-domain Tasks without structured parents for other runtime
purposes. A future explicit independent-work facility therefore does not require
a foundational rewrite of Future identity, Actor lifetime, cancellation, or the
scheduler model.

## Incremental-design and deferral result

Smallest sufficient current model:

```text
structured child creation
    -> owned for complete child lifetime

independent Actor lifetime
    -> Actor mechanisms

independent Actor-local background task
    -> absent until demonstrated need
```

Pay for what you need:

- ordinary structured users no longer pay the conceptual cost of a detachable
  ownership exception;
- no replacement background-work abstraction is introduced speculatively;
- runtime parent/child bookkeeping that structured concurrency requires remains.

Grow as you need:

- if real background work appears, add a separately justified explicit
  creation-time lifetime mechanism;
- do not reserve `Future.detach()` or promise its exact return.

Concrete cost of deferral:

- add one explicit work-creation/lifetime operation or abstraction;
- wire it to creation of Actor-local work without a structured parent;
- specify cancellation, Actor-termination, failure, and result-handle behavior;
- add focused conformance and implementation support.

That is bounded new work. It does not require changing String/object identity,
Future terminal states, Actor identity, P isolation, I/O commitment, or the
fundamental scheduler model.

## Future-scenario stress

A future server, event loop, cache refresh, telemetry publisher, periodic
maintenance loop, or other long-lived Actor-local service may need work that
outlives the task that started it.

Under Candidate B, such a use case cannot be represented as an ordinary child
Future whose ownership is later detached.

The escape path is an explicit future decision for independent work at creation
time. That future design can then define exactly who owns the work, how Actor
termination affects it, how it is cancelled, and how failures are observed,
without inheriting the accidental semantics of post-creation `detach()`.

## Compatibility and migration

Current repository evidence shows no production `.detach()` consumers under
`protos/lib` or `protos/tools`.

Conformance/control fixtures that use detached Futures to manufacture test
dependencies must be rewritten to preserve the test's intended cancellation or
cleanup condition without relying on removed public semantics.

The programmer-facing Future guide and normative concurrency/runtime references
must be reconciled consistently.

Because `Future.detach()` is observable Core behavior, D159 ratification alone
does not change the implementation or specification. A separate implementation
owner must perform the normative/runtime/test reconciliation.

## Intentionally deferred questions

D159 does not decide:

- a future background-work selector or syntax;
- whether independent Actor-local work should return Future or another handle;
- whether it should be a Closure operation, Future-producing operation, Actor
  operation, Standard Library abstraction, or another mechanism;
- daemon/service lifetime semantics;
- restart/supervision semantics for background Actor-local work;
- public Task or scope objects;
- Actor redesign;
- P/parallel lifetime changes;
- I/O producer custody changes.

Any such public semantic choice requires its own evidence and approval.

## Normative and implementation routing

Candidate B changes observable Core semantics.

The implementation owner must:

1. remove the public `Future.detach()` selector and normative contract;
2. make task-backed structured child ownership strict for the child's complete
   lifetime;
3. remove `ProtosFutureValue` detached state and the public detachment path;
4. remove `ProtosTask.detachFromParent()` if no remaining runtime-internal
   consumer requires it;
5. preserve normal parent/child removal on true child terminalization;
6. reconcile `Future.then` prose that currently mentions downstream detachment;
7. reconcile Actor-lifetime and abstract-runtime references;
8. update programmer-facing Future documentation;
9. rewrite conformance/control fixtures that currently use `.detach()` only as
   test scaffolding;
10. update `spec/PROTOS_SPEC_CHANGELOG.md`;
11. run focused Future/cancellation/structured-ownership validation and the
    complete repository validation required by current `AGENTS.md`.

```text
D159_STATUS=RATIFIED
SELECTED_CANDIDATE=B
FUTURE_DETACH=REMOVE_NOW_RECONSIDER_LATER
TASK_CHILD_OWNERSHIP=STRICT_COMPLETE_LIFETIME
BACKGROUND_WORK_REPLACEMENT=DEFERRED
NORMATIVE_RECONCILIATION_REQUIRED=YES
IMPLEMENTATION_RECONCILIATION_REQUIRED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
