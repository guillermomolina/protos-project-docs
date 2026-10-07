# D177 — Non-local return target reachability across a Task/Future boundary

Status: **RATIFIED — Candidate A**

Approval date: **2026-09-20**
Decision issue: `guillermomolina/protos#683`
Triggered by: BUG009 / `guillermomolina/protos#682`
Normative owner: `spec/semantics/CALLABLES.md` §14
Protos revision at ratification: `c952499db8df8de3da850b266ad291a25eed8a2f`

This is a durable non-normative language-decision record. Observable language
authority remains in the applicable ratified specification under `spec/`.

## Decision

D177 selects **Candidate A — a return home outside the executing Task's own
activation chain is unreachable and signals `InvalidReturn`**.

The selected rule is:

```text
NON_LOCAL_RETURN_OPERATOR=^

RETURN_HOME_REACHABILITY=
    the captured return-home activation must be reachable from the activation
    chain of the Task currently executing ^

HOME_COMPLETED=InvalidReturn
HOME_BELONGS_TO_DIFFERENT_TASK=InvalidReturn

CROSS_TASK_NON_LOCAL_RETURN_DELIVERY=NO
CROSS_TASK_UNWIND_BY_CARET=NO
CROSS_TASK_RESUME_BY_CARET=NO
CROSS_TASK_COMPLETION_BY_CARET=NO

FUTURE_VALUE_COMMUNICATION=UNCHANGED
EXPLICIT_FUTURE_TASK_MECHANISMS=UNCHANGED
```

A captured home is therefore not reachable merely because it is still alive.
A live activation owned by another Task is outside the dynamic control chain
through which the executing `^` may transfer control.

This generalizes the existing `CALLABLES.md` §14 rule from the previously
named completed-home case to the underlying reachability invariant.

## Boundary

D177 does **not** add a mechanism for one Task to force a return into another
Task. If a future requirement needs explicit cross-Task value delivery or
control, that capability requires a separately named and justified mechanism.

In particular D177 does not add:

```text
cross-Task stack unwinding
remote ensure execution
cross-Task continuation delivery
Task resumption by ^
Task completion by ^
new Task/Future ownership semantics
new cancellation semantics
static closure-effect analysis
new Error family
```

The existing `InvalidReturn` classification remains the semantic failure
surface; D177 extends when it applies rather than defining a new failure kind.

## Why

The motivating BUG009 reproduction executes a `^` from cleanup code in a Task
different from the Task that owns the closure's captured return home. The home
is still live, but it is not on the executing Task's activation chain.

Treating that situation as a valid non-local return would require a second,
asynchronous control-transfer mechanism: synchronization with the target Task,
cross-Task unwind, races with target progress, cleanup ordering, and structured
child handling. No present requirement justifies that capability.

Treating the target as unreachable preserves the existing return-home model and
turns the previously unspecified/hanging case into an immediate diagnosable
`InvalidReturn`.

## Prior-art result

The D177 investigation compared Smalltalk-family non-local returns, Ruby
Proc/block return, Common Lisp dynamic-extent exits, Kotlin's compile-time
non-local-return restrictions, concurrent isolation models such as Erlang/Go,
and ordinary function-local return models such as Java/JavaScript.

The converging evidence is that non-local control transfer is scoped to a
reachable current stack/dynamic extent. Systems with independently scheduled
execution units use explicit value/message/join mechanisms rather than allowing
a raw return operator to reach into another unit's suspended stack.

D177 adopts that boundary without importing static-analysis machinery or a new
cross-Task delivery institution.

## Candidate result

### Candidate A — foreign-Task home is unreachable

**Selected.**

It is the smallest extension of the existing §14 reachability rule, keeps Task
boundaries local, imposes no cross-Task coordination, and leaves explicit
future mechanisms possible.

### Candidate B — cross-Task non-local-return delivery

Rejected for D177.

It would create a new asynchronous control-transfer channel and require race,
unwind, cleanup, cancellation and structured-child semantics without a present
use case.

### Candidate C — statically forbid the capture

Rejected for D177.

Protos closures are dynamic first-class values. Soundly rejecting every
possible cross-Task execution ahead of time would require substantially broader
static/effect analysis that Protos does not otherwise require.

### Candidate D — defer / status quo

Rejected.

The gap already has an observable failure mode: BUG009 can hang indefinitely.
Leaving the case unspecified is not a neutral deferral.

## Invariant / delta consistency

The selected candidate preserves the applicable established invariants:

```text
CALLABLES_13_CAPTURED_RETURN_HOME=KEEP
CALLABLES_14_INVALID_RETURN_FOR_UNREACHABLE_HOME=GENERALIZE
CURRENT_TASK_ACTIVATION_CHAIN_IS_CONTROL_BOUNDARY=YES

D045_TASK_EXECUTION_BOUNDARY=KEEP
D045_STRUCTURED_FUTURE_OWNERSHIP=KEEP

D162_FATAL_FAILURE_AUTHORITY=KEEP
D162_NOT_REOPENED=YES

CROSS_TASK_CARET_DELIVERY=NOT_ADDED
STATIC_CLOSURE_EFFECT_SYSTEM=NOT_ADDED
NEW_INVALID_RETURN_SUBTYPE=NOT_ADDED

DECISION_INVARIANT_CONSISTENCY=PASS
```

D177 does not decide D159 and does not depend on changing `Future.detach()`
semantics.

## Implementation consequence

The normative specification must make §14 explicit that reachability is relative
to the Task executing `^`, and that a live captured home belonging to another
Task signals `InvalidReturn`.

BUG009 may then repair the runtime path so this semantic failure is surfaced
rather than escaping through cancellation/continuation machinery and leaving an
awaiting Future blocked forever.

Required regression coverage includes at least:

- same-Task reachable non-local return remains unchanged;
- completed return home still signals `InvalidReturn`;
- live foreign-Task return home signals `InvalidReturn`;
- the BUG009 cancellation/`.ensure()` reproduction terminates diagnostically
  rather than hanging;
- no cross-Task return value is delivered to the foreign home.

## Deliberately deferred

```text
explicit cross-Task value/control-delivery primitive
cross-Actor non-local control transfer
static closure effect analysis
specialized InvalidReturn subtype
new Task/Future communication syntax
```

Any such capability requires independent evidence and a separate design
decision.

## Approval provenance

The exact Candidate A boundary was presented to the project owner in the active
interaction on 2026-09-20. The project owner explicitly approved it:

```text
aprobada
```

Ratification summary:

```text
D177_STATUS=RATIFIED
SELECTED_CANDIDATE=A
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS

FOREIGN_TASK_RETURN_HOME=UNREACHABLE
FOREIGN_TASK_RETURN_RESULT=InvalidReturn
CROSS_TASK_CARET_DELIVERY=NO

BUG009_SEMANTIC_BLOCKER=RESOLVED_BY_D177
```
