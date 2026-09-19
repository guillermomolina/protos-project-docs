# AUD009-C1 — Future, Task, cancellation, and structured-concurrency complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#622`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `cfc732b154f9e1480d086f97bbc0efd06d62c37b`

Closure revalidation revision: `d5266bc14070fce5a983ac608f1e9d4f613d5535`

The two Protos commits between the C1 evidence baseline and closure revalidation
changed Test Tool implementation/tests plus implementation metadata. They did not
modify the scoped Future/Task normative owners or the core runtime classes
`ProtosFutureValue`, `ProtosTask`, `ProtosStandardFutureProtocol`, or
`ProtosActorExecutionDomain`. The C1 classification therefore remained valid at
closure revalidation.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Checkpoint proposal: `guillermomolina/protos#622`, issue comment
`5739277128`.

Owner approval provenance: `guillermomolina/protos#622`, issue comment
`5739311330`, 2026-09-19.

Derived decision route: `D159 / guillermomolina/protos#623` —
Future detachment and strict structured child lifetime.

## Purpose and boundary

AUD009-C1 reviewed the retained Core Future/task model and ordinary cooperative
asynchronous execution.

The slice covered:

- Future identity and terminal-outcome semantics;
- ordinary Closure `future()` asynchronous execution;
- `Future.value()` suspension and waiter behavior;
- `Future.then(...)` composition;
- `Future.all(...)` deterministic aggregation;
- cooperative cancellation;
- task-scoped structured ownership;
- public detachment through `Future.detach()`;
- the internal/public boundary around Task.

C1 did not audit isolated `parallel` / P semantics or Actor mailbox,
supervision, replacement, distribution, and lifecycle as independent
institutions. Those remain later AUD009-C work.

C1 is an evidence/classification slice. It does not itself alter normative
semantics or implementation.

## Final classification ledger

```text
Future Core family                                  KEEP
fresh Future result identity                        KEEP
same-domain Future ownership                        KEEP
first-terminal-transition stability                 KEEP

pending/resolved/failed/cancelled model             KEEP
cancelled distinct from failed(Error)               KEEP
same-domain failed Error identity                   KEEP
repeated failed observation re-signals same Error   KEEP
cancelled value() fresh Cancelled occurrence        KEEP
producer control state not transported              KEEP

Future result adoption / flattening                 KEEP
outcome-only adoption                               KEEP
no ownership transfer through adoption              KEEP
no upstream cancellation through adoption           KEEP
FutureResolutionCycle                               KEEP

Closure.future()                                    KEEP
asynchrony belongs to execution                     KEEP
ordinary Object.future lookup/shadowing model       KEEP

Future.value()                                      KEEP
pending value() explicit suspension                 KEEP
lost-wakeup exclusion                               KEEP
multiple independent waiters                        KEEP
waiter cancellation isolation                       KEEP
resume-boundary cancellation precedence             KEEP

Future.then                                         KEEP
ordinary-invokable transform                        KEEP
asynchronous/non-inline continuation                KEEP
fresh destination Future                            KEEP
source failure propagation                          KEEP
source cancellation propagation                     KEEP
automatic returned-Future adoption                  KEEP
downstream-only cancellation/ownership               KEEP
D030 eager callability validation                   KEEP

Future.all                                          KEEP
fresh non-task aggregate Future                     KEEP
fresh result Array                                  KEEP
argument-order successful result                    KEEP
ascending-index failure/cancellation frontier       KEEP
aggregate cancellation does not cancel sources      KEEP
zero-argument resolved seed Future                  KEEP

Future.cancel()                                     KEEP
idempotent cancellation request                     KEEP
cooperative cancellation                            KEEP
portable explicit cancellation boundaries          KEEP
cancellation-runnable pre-start/suspended work      KEEP
no arbitrary asynchronous interruption             KEEP
no hidden loop/allocation/JIT cancellation polls   KEEP
no implicit upstream cancellation                   KEEP
committed effects never rolled back                 KEEP

task-scoped structured ownership                    KEEP
child owned by current async task by default        KEEP
normal owner terminalization waits for children     KEEP
owner failure/cancellation cancels/drains children  KEEP
synchronous return does not re-parent child         KEEP
Future escape does not alter ownership              KEEP
child failure does not implicitly fail owner        KEEP
no hidden unobserved-failure/consumed state         KEEP

internal Task execution/lifetime concept            KEEP
public Task value/prototype                         ABSENT / RETAIN ABSENCE
public structured-scope object                      ABSENT / RETAIN ABSENCE

Future.detach()                                     REMOVE_NOW_RECONSIDER_LATER

async Closure/function category                     ABSENT / RETAIN ABSENCE
async keyword                                       ABSENT / RETAIN ABSENCE
await keyword                                       ABSENT / RETAIN ABSENCE
generic Future.race/select                          ABSENT / RETAIN ABSENCE
public Future state/polling protocol                ABSENT / RETAIN ABSENCE
implicit child-failure propagation                  ABSENT / RETAIN ABSENCE
implicit upstream cancellation through dependency  ABSENT / RETAIN ABSENCE
```

## Future remains the Core eventual-result institution

Future has broad real use across ordinary asynchronous Closure execution,
filesystem/network/byte/text I/O, Actor request and lifecycle observations,
isolated-parallel result delivery, asynchronous exact execution used by tooling,
and Standard Library streaming/adapters.

Removing Future would therefore require another common eventual-result
institution rather than eliminating the underlying requirement.

Fresh result identity remains necessary because independent Future-producing
invocations must not accidentally share cancellation, waiters, observation state,
ownership effects, or identity merely because they have equal producer inputs or
eventual outcomes.

Classification: **KEEP**.

## Four terminal-outcome states remain justified

C1 explicitly tested collapsing:

```text
cancelled
```

into an ordinary:

```text
failed(CancelledError)
```

That simplification was rejected.

Cancellation is a lifecycle/control outcome and can exist without manufacturing
one producer Error object. Retaining it separately permits adoption, `then`, and
`Future.all` to propagate cancellation as cancellation while allowing each
consumer-side `value()` observation to create a fresh ordinary `Cancelled`
Error occurrence in that consumer's dynamic handler context.

A failed Future, by contrast, stores the exact same-domain Error outcome and
re-signals that object on repeated observation.

The distinction therefore removes ambiguity between producer outcome and
consumer observation rather than introducing gratuitous state.

Classification: **KEEP**.

## Adoption and flattening remain necessary

Future resolution adopts a Future result instead of resolving to a nested Future.

That behavior is especially important because Protos deliberately has no
separate async-function/await callable category. Without adoption, ordinary
asynchronous composition naturally accumulates nested Future values and every
consumer must rebuild flattening behavior.

Adoption mirrors only eventual outcome. It does not merge Future identity,
structured ownership, detachment, or upstream cancellation authority.

Cycle failure remains required because a direct or transitive adoption cycle
otherwise has no semantic event guaranteed to make the destination terminal.

Classification: **KEEP**.

## Closure `future()` remains Protos-aligned

The same ordinary Closure can execute synchronously through `closure()` or
asynchronously through `closure.future()`.

Production Standard Library code, including `std:io/Files`, uses this mechanism
directly.

The model avoids a second async callable kind and avoids dedicated `async` /
`await` syntax. D025's ordinary `Object.future` placement remains coherent
with the callable/object model retained by AUD009-B2.

Classification:

```text
Closure.future()                 KEEP
async callable category          ABSENT / RETAIN ABSENCE
async keyword                    ABSENT / RETAIN ABSENCE
await keyword                    ABSENT / RETAIN ABSENCE
```

## `Future.value()` suspension remains foundational

`Future.value()` is heavily used in production Standard Library and Tool source.

For a pending Future, explicit suspension requires atomic lost-wakeup exclusion
between terminal observation and waiter registration. Cancelling one waiter must
remove only that wait relationship and must not cancel the producer or other
waiters.

These are correctness rules, not prescriptions for monitors, callbacks, CAS,
continuations, carrier parking, or another physical scheduler strategy.

Classification: **KEEP**.

## `Future.then(...)` remains Core

There is production use in JSON and CSV streaming adapters and Test Tool
composition.

C1 tested whether `then` could become an ordinary source helper equivalent to:

```protos
(() => transform(source.value())).future()
```

That is not semantically equivalent while cancellation remains a distinct Future
outcome.

A cancelled source observed through `value()` produces a fresh consumer-side
`Cancelled` Error. The wrapper task would therefore fail rather than propagate
the source's semantic `cancelled` outcome.

The ordinary source replacement also changes ownership and cancellation timing.
The retained non-inline continuation Task is therefore a real composition
primitive, not merely syntactic convenience.

C1 also challenged D030's eager callability validation. It remains justified
because it reuses existing Core callability inspection, rejects an invalid
transform before manufacturing a continuation Task/destination Future, and does
not pin the later actual invocation lookup.

Classification: **KEEP**.

## `Future.all(...)` remains Core

Production consumers include `std:io/Files`, Test Tool Runner, and Test Tool
Manifest loading.

A source helper that waits on source Futures through `value()` cannot preserve
the current cancellation outcome for the same reason described for `then`.
A cancelled source becomes a consumer-side `Cancelled` Error rather than a
cancelled aggregate Future.

The current ascending-index frontier also makes failure/cancellation selection
deterministic without exposing host callback order or scheduler timing.

The zero-argument case is used in production as an already-resolved composition
seed.

Classification: **KEEP**.

## Cooperative cancellation remains the smallest safe cancellation model

Production `std:io/Files` already depends on explicit Future cancellation to
preserve acquisition/resource custody across cancellation races.

The retained model observes cancellation at portable boundaries rather than
injecting arbitrary asynchronous interruption into ordinary code.

Programs that never suspend do not acquire a semantic cancellation poll at every
call, allocation, or loop edge. Suspension and explicitly cancellation-aware
operations pay the coordination cost.

Committed effects are never retroactively rolled back by cancellation.

Classification: **KEEP**.

## Task-scoped structured ownership remains justified

D045's task-scoped ownership model survives the retrospective audit.

Task-backed asynchronous child work is owned by the current asynchronous task
scope. Ordinary synchronous calls do not create nested structured scopes, and
returning/storing/wrapping a Future does not change ownership.

The owner waits for non-detached child terminality on otherwise-normal
completion and cancels/drains owned children during failure/cancellation unwind.

Ownership controls lifetime, not result relevance. A failed child does not
implicitly fail its owner; code observes child outcome explicitly through
ordinary Future operations.

This avoids hidden "unobserved failure" state while still preventing accidental
child-lifetime escape.

Classification: **KEEP**.

## Internal Task remains necessary but should remain non-public

Future and Task represent different concerns.

A Future is an eventual-result object. Not every Future is task-backed: I/O and
other asynchronous producers can own non-task-backed Futures.

Conversely, task-like cooperative execution and structured lifetime belong to
running computation and need not always correspond to a public Future object.

PLAT029 supplied concrete evidence for this separation: manufacturing hidden
`Future.then` Tasks merely to retain non-task I/O continuation state would
change ownership, detachment, and Actor-lifecycle semantics.

The internal Task concept therefore remains necessary, but exposing a public Task
or scope value would add identity/state/lifetime API surface without current
need.

Classification:

```text
internal Task concept             KEEP
public Task                       ABSENT / RETAIN ABSENCE
public structured scope          ABSENT / RETAIN ABSENCE
```

## `Future.detach()` is the surviving removal candidate

Approved classification:

```text
Future.detach()                   REMOVE_NOW_RECONSIDER_LATER
```

At the evidence baseline, repository search found no production `.detach()`
use under `protos/lib` or `protos/tools`.

Current uses are conformance/control fixtures, primarily creating dependencies
for cancellation and cleanup tests. Those tests prove current semantics; they do
not establish production need for public detachment.

Continuing complexity includes:

- one public Future selector;
- Future detached-state bookkeeping;
- mutation of structured parent topology;
- no-op rules for non-task-backed and terminal Futures;
- detachment-specific clauses for adoption and `then`;
- Actor-lifecycle clauses clarifying that detached work is still Actor-local;
- implementation and conformance branches for those distinctions.

Without public detachment, the structured lifetime rule becomes:

```text
task-backed child created in a task scope
    -> remains owned by that scope until terminal
```

The underlying Task parent/child mechanism remains required, so later
reintroduction of a justified lifetime-escape capability does not require
rebuilding structured ownership from zero.

### Required AUD009 routing fields

```text
FEATURE=Future.detach()
CURRENT_AUTHORITY=spec/concurrency/FUTURES_AND_TASKS.md
CURRENT_IMPLEMENTATION=Future protocol + ProtosFutureValue/ProtosTask ownership edge
CURRENT_CONSUMERS=conformance/control fixtures; no production protos/lib or protos/tools use found

PROPOSED_OUTCOME=REMOVE_NOW_RECONSIDER_LATER

SEMANTIC_DECISION_OWNER=D159 / guillermomolina/protos#623
IMPLEMENTATION_REMOVAL_OWNER=TBD_AFTER_D159_RATIFICATION

RECONSIDERATION_TRIGGER=
    concrete production/library/tooling evidence for Actor-local asynchronous
    work that intentionally must outlive its creating structured task scope,
    cannot safely remain structured, and does not fit an existing Actor,
    Process, I/O producer, or other already-owned lifetime boundary

RECONSIDERATION_SCOPE=
    design an explicit lifetime-escape/background-work capability from the
    then-current Protos model and evidence; do not preselect Future.detach(),
    its current selector spelling, its current semantics, or dormant runtime hooks

STATUS=CLASSIFICATION_APPROVED_REMOVAL_BLOCKED_ON_D159
CONFIDENCE=HIGH
```

This is explicitly **not** permanent rejection. If real need appears later,
Protos may design substantially the same capability again. No current syntax,
selector, semantics, implementation branch, or compatibility scaffolding is
reserved by that possibility.

## Generic race/select remains absent

Core defines no portable total order for independent Future terminalizations.

A generic "first completion wins" operation would either expose scheduler/backend
timing as language semantics or impose an unrelated fixed priority and call it a
race.

Domain-specific selection remains possible when a domain supplies an independent
semantic ordering rule.

Classification:

```text
Future.race/select                 ABSENT / RETAIN ABSENCE
generic physical-completion wait-any
                                   ABSENT / RETAIN ABSENCE
```

## Strongest attempted removals

```text
remove Future family                      rejected -> KEEP
collapse cancelled into failed(Error)     rejected -> KEEP
remove fresh Future identity              rejected -> KEEP
remove adoption/flattening                rejected -> KEEP
move then entirely to source library      rejected -> KEEP
remove D030 eager then validation         rejected -> KEEP
move Future.all entirely to source lib    rejected -> KEEP
remove cooperative cancellation           rejected -> KEEP
remove structured child ownership         rejected -> KEEP
collapse hidden Task into Future          rejected -> KEEP
expose public Task/scope                   rejected -> RETAIN ABSENCE
add generic race/select                    rejected -> RETAIN ABSENCE

remove Future.detach                      survives -> REMOVE_NOW_RECONSIDER_LATER
```

## Boundary handoffs

- **D159 / #623** owns the exact semantic decision for public Future detachment
  versus strict task-backed child lifetime.
- **Later AUD009-C slices** own isolated parallel execution/P and Actor-specific
  concurrency institutions.
- **AUD009-D** owns detailed I/O operation/commitment complexity.
- **AUD009-G** owns concrete runtime/continuation/backend complexity.
- **AUD009-B2** remains authoritative for callable/object placement and ordinary
  invocation.
- **AUD009-B3** remains authoritative for Error/handler/`ensure` semantics
  except where C1 necessarily composes cancellation with cleanup.
- **D025**, **D030**, and **D045** remain authoritative retained decisions.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#622
CHECKPOINT_COMMENT=5739277128
APPROVAL_COMMENT=5739311330
DATE=2026-09-19

FUTURE_CORE_MODEL=KEEP
FOUR_STATE_OUTCOME_MODEL=KEEP
FUTURE_ADOPTION=KEEP
CLOSURE_FUTURE=KEEP
FUTURE_VALUE=KEEP
FUTURE_THEN=KEEP
FUTURE_ALL=KEEP
COOPERATIVE_CANCELLATION=KEEP
TASK_SCOPED_STRUCTURED_OWNERSHIP=KEEP
INTERNAL_TASK=KEEP

PUBLIC_TASK=ABSENT_RETAIN_ABSENCE
PUBLIC_SCOPE_OBJECT=ABSENT_RETAIN_ABSENCE
ASYNC_AWAIT=ABSENT_RETAIN_ABSENCE
GENERIC_RACE_SELECT=ABSENT_RETAIN_ABSENCE
PUBLIC_FUTURE_STATE_POLLING=ABSENT_RETAIN_ABSENCE
IMPLICIT_CHILD_FAILURE_PROPAGATION=ABSENT_RETAIN_ABSENCE

FUTURE_DETACH=REMOVE_NOW_RECONSIDER_LATER
DERIVED_DECISION=D159 / guillermomolina/protos#623

NORMATIVE_CHANGE_AUTHORIZED_BY_C1=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_C1=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=cfc732b154f9e1480d086f97bbc0efd06d62c37b
CLOSURE_REVALIDATION=d5266bc14070fce5a983ac608f1e9d4f613d5535
REVALIDATION_OVERLAP_WITH_C1_OWNERS=NONE
REMOVAL_ROUTE=D159
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_C1_CLASSIFICATION=COMPLETE
```

AUD009-C1 is complete once this durable record and the required live GitHub
closure postconditions are verified.
