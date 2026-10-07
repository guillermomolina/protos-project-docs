# D189 — Foreign callback execution and lifetime semantics

Status: **RATIFIED — OWNER APPROVED; NORMATIVE SPECIFICATION RECONCILIATION ROUTED TO I081**

Owning decision: `guillermomolina/protos#820` — `D189 — Foreign callback execution and lifetime semantics`

Parent audit: `guillermomolina/protos#818` — `AUD019 — Foreign-library interop and polyglot import architecture audit`

Normative reconciliation owner: `guillermomolina/protos#829` — `I081 — Reconcile D189 foreign callback semantics into normative specification`

Nature: durable implementation-independent language decision record; **non-normative until reconciled into the applicable `spec/` authority**

Approval date: **2026-10-07**

Revalidated Protos state at ratification publication:

```text
PROTOS_REVISION=84ff312ca6c29d2bfa9e6a4d69861c1dfdd07c47
PROTOS_VERSION=0.3.265-SNAPSHOT
SPECIFICATION_REVISION=0.1.445
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

The owner-review packet started from specification revision `0.1.445` and
revalidated live `main` before recommendation. The two product commits after
I080 are PLAT051-B and TEST009-AI; neither changes D189-governing specification
semantics, and TEST009-AI explicitly declares no specification or semantic
change.

```text
D188_AUTHORITY=RATIFIED_AND_NORMATIVE
D188_INVARIANT_DELTA=NONE
D189_DECISION_DELTA=NONE
```

## Approval provenance

The completed D189 GITHUB010 investigation recommended:

```text
RECOMMENDED_CANDIDATE=A_DYNAMIC_SYNCHRONOUS_CALLBACK_BASELINE
```

The project owner explicitly approved that exact recommendation in the active
owner-review interaction on 2026-10-07:

> apruebo A

That approval selects the bounded semantic contract below. It does not authorize
foreign-provider/runtime implementation and does not make this project record
normative specification.

```text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=A_DYNAMIC_SYNCHRONOUS_CALLBACK_BASELINE
OWNER_APPROVAL_REQUIRED=NO
```

## Ratified semantic contract

### 1. Smallest callback institution

The first foreign-to-Protos callback institution is **dynamic and synchronous**.

A callback capability is valid only inside the dynamic extent of the foreign
operation that received it. The callback re-enters the same logical Protos
execution and must complete before that foreign operation continues.

```text
Protos Actor / Task
  -> synchronous foreign operation
       -> synchronous callback
            -> same Actor
            -> same Task
            -> ordinary Protos callback activation
       -> foreign operation continues
  -> Protos
```

This baseline does not define retained/asynchronous callbacks.

### 2. Actor and Task ownership

The callback belongs to the Actor and current Task whose execution entered the
originating foreign operation.

```text
CALLBACK_OWNER_ACTOR=ORIGINATING_FOREIGN_CALL_ACTOR
CALLBACK_OWNER_TASK=SAME_CURRENT_TASK_AND_STRUCTURED_EXECUTION_SCOPE
NEW_TASK_ON_CALLBACK=NO
```

Callback entry creates no hidden Task, Future, Actor turn, structured-concurrency
scope, scheduler boundary, or cancellation checkpoint.

Physical carrier or operating-system thread identity is not semantic Actor/Task
authority.

### 3. Callback semantic value

The foreign-facing capability refers to the **exact Protos semantic callback
value**. A host/runtime adapter may exist as implementation machinery, but its
identity does not replace Closure/object identity.

For a Closure, ordinary Protos activation semantics remain authoritative,
including lexical capture, receiver, `methodHome`, argument binding, return
home, non-local return, Error propagation, and ordinary call/control rules.

A callback export must not reconstruct a merely similar Closure.

### 4. Argument admission

Foreign callback arguments enter Protos through the already-ratified D188
foreign-value admission and conversion substrate.

```text
foreign callback arguments
  -> D188 admission/conversion
  -> ordinary callback arguments
```

There is no callback-specific second conversion universe.

### 5. Result projection

Automatic callback result projection is deliberately minimal and lossless.

Existing Protos scalars may cross only where the provider can represent them
without changing their value semantics. A raw foreign reference may return
faithfully to its owning provider when the existing D188 identity substrate can
preserve the same foreign target.

An arbitrary identity-bearing Protos object graph does **not** automatically
become a retained foreign handle merely because a callback returns it.

```text
CALLBACK_RESULT_PROJECTION=
  LOSSLESS_MINIMAL_OUTBOUND_PROJECTION
  + FAITHFUL_SAME_PROVIDER_FOREIGN_REFERENCE_ROUNDTRIP
  + EXPLICIT_EXPORT_CONTRACT_REQUIRED_FOR_OTHER_IDENTITY_BEARING_PROTOS_VALUES
```

Exact provider-specific lowering remains implementation freedom.

### 6. Error and non-local control

If a Protos callback signals an already-identified Protos Error/control outcome
and the foreign boundary returns that same runtime-carried outcome recognizably
and unchanged, Protos resumes propagation of the **same semantic Error/control
value**. The boundary does not manufacture a fresh Error merely because the
failure crossed through foreign code.

If foreign code produces, replaces, wraps, transforms, or otherwise turns that
outcome into a foreign failure, D188 remains authoritative: an entered foreign
operation returning that failure to Protos produces a fresh `ForeignError`
occurrence.

Raw Java `Throwable`, `PolyglotException`, foreign exception handles, or mixed
foreign/native stacks never become a second Protos exception hierarchy.

Ordinary non-local-return semantics remain those of the exact Closure and its
real return home; an invalid/dead home remains the ordinary `InvalidReturn`
case.

### 7. Synchronous reentrancy

Nested, sequential, and recursive synchronous callbacks are allowed while the
originating foreign dynamic extents remain live:

```text
Protos
  -> foreign A
       -> callback Protos
            -> foreign B
                 -> callback Protos
```

They remain on the same owning Actor and Task.

Concurrent callback entry against that same logical execution is not allowed.
Foreign runtime concurrency does not create simultaneous Protos execution
against one Actor-local mutable domain.

### 8. Foreign-created threads

A foreign-created thread, worker-pool thread, event-loop thread, or other
unrelated physical thread does not acquire Protos Actor/Task authority merely
because it possesses a callback wrapper.

The baseline rejects such entry **before guest execution**.

It does not silently:

- enqueue into the Actor;
- create a new Task;
- block the foreign thread awaiting an invented Task;
- or fire-and-forget guest work.

Those are semantics of a future explicit retained/asynchronous ingress facility.

### 9. Suspension boundary

A callback may execute ordinary Protos code so long as no **actual suspension**
of the current Task across the foreign dynamic extent is required.

If an existing suspension-capable operation can complete without suspending,
ordinary execution may continue. If it would actually suspend, the baseline
rejects the suspension before it commits.

```text
CALLBACK_SUSPENSION_RULE=
  NO_ACTUAL_TASK_SUSPENSION_ACROSS_ARBITRARY_FOREIGN_DYNAMIC_EXTENT
```

D189 does not retain, capture, reify, or later resume arbitrary Java/native/
foreign stacks.

This preserves PLAT014 C′, PLAT019 B′, and PLAT028 Candidate C: suspendible
guest/post-callback control belongs to the Protos/C′ continuation domain, not to
arbitrary host/native frames. No hidden child Task is created merely to preserve
a foreign stack.

D189 does not require a new public Error prototype solely for this rejection.
The normative reconciliation should use the smallest existing Error institution
unless specification consistency proves that a new category is necessary.

### 10. Cancellation

Callback entry adds no new cancellation-observation boundary.

The callback executes as the same Task, so the existing cooperative cancellation
rules apply only at already-defined boundaries reached by that Task.

D189 does not standardize universal cancellation of the foreign operation.

### 11. Dynamic lifetime

The semantic callback capability becomes live for the originating foreign
operation and expires when that operation leaves its dynamic extent by normal
return, Error/failure, non-local control, cancellation unwind, or termination.

Physical retention of a host/runtime wrapper does not extend semantic lifetime.

After expiry the wrapper must not semantically root or revive:

```text
Process
Actor
Task
provider / Context
Closure execution authority
return home
```

Exact adapter/token/generation representation and reference-release mechanics
remain implementation freedom.

### 12. Late callbacks and termination

A callback attempt after the dynamic lifetime ends is rejected before guest
entry.

The same fail-closed rule applies once the owning Task is terminal, the Actor can
no longer admit ordinary work, the Process has terminated/closed, or the
applicable provider/Context has closed.

```text
NO_NEW_TASK=YES
NO_ACTOR_RESURRECTION=YES
NO_PROCESS_RESURRECTION=YES
NO_CONTEXT_PROVIDER_RESURRECTION=YES
```

If such a failure occurs inside a separately entered foreign operation that
later returns to Protos, that foreign operation follows the existing D188
`ForeignError` contract. Without a waiting Protos execution there is no
invented Protos Error delivery channel.

## Rejected/deferred candidates

### Candidate B — new Actor-local Task per callback

Rejected.

A new Task is not required to explain a synchronous nested callback and would
introduce new structured ownership, scheduling, task-start cancellation,
result-routing, and possible same-Actor deadlock/reentrancy questions. Running a
new Task inline would create a semantically privileged Task with no justified
asynchronous institution.

### Candidate C — retained/general asynchronous callback baseline

Not rejected as a future capability; **deferred as a separate additive
institution**.

Supporting listeners, timers, event loops, callbacks after return, callbacks
from foreign threads, and async completions requires explicit decisions for
Actor affinity, Task creation/ownership, ingress scheduling, result routing,
rooting, revocation, cancellation, provider/Process lifetime, and concurrent
entry. D189 intentionally does not make every synchronous callback pay that
complexity.

## Pay-as-you-grow and escape path

Programs that never export callbacks pay no callback-specific semantic/runtime
cost. Foreign calls without callbacks pay no callback-specific cost. A used
synchronous callback requires only bounded dynamic capability/ownership state.

A later retained/asynchronous callback facility can be added without changing
this contract by making retention explicit and assigning its own ingress,
Task/result, rooting, revocation, cancellation, and lifecycle semantics.

A future suspendible foreign boundary is also possible where all control that
must survive the suspension is represented in C′/Protos runtime state rather
than an arbitrary host/native stack.

```text
STRONGEST_ARGUMENT_AGAINST_RECOMMENDATION=
  MANY_REAL_FOREIGN_APIS_RETAIN_CALLBACKS_OR_INVOKE_THEM_FROM_WORKER_EVENT_THREADS,
  AND_A_CALLBACK_MAY_NEED_TO_SUSPEND

REGRET_SCENARIO=
  AN_EARLY_CRITICAL_FOREIGN_LIBRARY_REQUIRES_RETAINED_THREAD_SHIFTED_OR
  SUSPENDIBLE_CALLBACKS_AS_ITS_PRIMARY_API

ESCAPE_PATH=
  ADD_AN_EXPLICIT_RETAINED_ASYNC_CALLBACK_INGRESS_FACILITY;
  ADD_SUSPENDIBLE_FOREIGN_BRIDGES_ONLY_WHERE_SURVIVING_CONTROL_IS_OWNED_BY_CPRIME
```

## Intentionally deferred

```text
retained callback handles
foreign-thread ingress
event listeners
timers
foreign event loops
async completion callbacks
callback Task creation/ownership
async result channels
persistent rooting and revocation
retained-callback cancellation API
provider/Context/Engine topology
universal Promise/CompletionStage/awaitable bridging
```

## Normative reconciliation

D189 is an observable Protos-semantic decision, so the ratification record alone
does not make the selected contract normative.

Specification reconciliation is routed to:

```text
I081 / guillermomolina/protos#829
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
```

Expected owners include `CALLABLES.md`, `VALUES_AND_COLLECTIONS.md`,
`ERRORS.md`, `EXECUTION_AND_CONTROL.md`, `ACTORS.md`, and
`FUTURES_AND_TASKS.md`, but I081 must modify only the actual normative owners
found at execution-time HEAD.

```text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=YES
SPECIFICATION_CHANGE_REQUIRED=YES
FOREIGN_RUNTIME_IMPLEMENTATION_AUTHORIZED=NO
D189_STATUS=COMPLETED_AFTER_DURABLE_PUBLICATION
NEXT_IMPLEMENTATION_OWNER=I081/#829
```

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed D189 GITHUB010 investigation, current repository and
upstream evidence, and the project owner's explicit approval. No independent
human review is claimed by this record.
