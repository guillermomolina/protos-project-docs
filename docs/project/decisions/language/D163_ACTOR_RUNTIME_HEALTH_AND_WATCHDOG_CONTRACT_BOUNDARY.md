# D163 — Actor runtime-health and watchdog contract boundary

Status: **RATIFIED — Candidate A (KEEP)**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#631`  
Trigger: AUD009-C3 / `guillermomolina/protos#627`  
Protos evidence revision: `5ce8e039a69489a49fe446d58de7fb39bcbb278f`  
Project-record base: `2f9a8c1dbfafcbf2eb1047e41ffa9010064864bf`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D163 selects **Candidate A — KEEP**.

The existing Actor runtime-health/watchdog guardrail remains part of portable Core architecture:

```text
mandatory inexpensive runtime-health path       KEEP
health fast path O(1)                           KEEP
health fast path non-blocking                   KEEP
health fast path free from global coordination  KEEP
expensive analysis via sampling/instrumentation KEEP

specific public health API                      NOT REQUIRED
specific Health object                          NOT REQUIRED
fixed mandatory metric set                      NOT REQUIRED
progress epoch field                            OPTIONAL
mailbox depth exposure                          OPTIONAL
failure counters                                OPTIONAL
runtime-private counters/probes                 ALLOWED
```

The retained requirement is an **architectural scalability invariant**, not a commitment to a particular public metrics API, health snapshot object, fixed counter schema, watchdog policy, cluster control plane, or instrumentation backend.

## Why the AUD009-C3 provisional removal is superseded

AUD009-C3 proposed `REMOVE_NOW_RECONSIDER_LATER` because there was no public Core consumer/API and most of the illustrative metric set was not materialized in the runtime.

D163 rejects that inference.

The absence of a current public consumer does not remove the architectural value of constraining the future health fast path. The purpose of the O(1), non-blocking, no-global-coordination rule is to prevent a future observability/watchdog implementation from making Actor health depend on:

- traversal of global Actor registries;
- global scheduler or Process locks;
- cluster-wide coordination;
- synchronous remote round trips;
- reconstruction of health by scanning tasks/mailboxes/ownership graphs;
- consistent global snapshots on ordinary health checks.

Those designs may appear acceptable at small Actor counts and become pathological at large scale. The retained rule prevents that class of scaling mistake before implementation architecture calcifies around it.

The C3 classification remains historical audit evidence only. D163 is the authoritative owner-approved decision for this boundary.

## Existing runtime fit

The current runtime already maintains local state required independently by retained Actor semantics, including lifecycle state, bounded mailbox state, runnable scheduling state, live tasks, termination observers and related lifecycle bookkeeping.

Examples at the evidence revision include:

- `ProtosActor.lifecycleState()`;
- local `ProtosActorMailbox.size()` and capacity;
- `ProtosActorExecutionDomain.runnableCount()`;
- `ProtosActorExecutionDomain.liveTaskCount()`.

D163 does **not** require all of these to become public health fields, nor does it require the runtime to add every illustrative counter in ACTORS §30.

The obligation is that the inexpensive mandatory health path remain locally derivable with bounded constant-time work and without blocking/global coordination.

## Fixed authority preserved

Candidate A preserves:

```text
Actor identity/lifecycle                     PRESERVED
weak fairness                                PRESERVED
bounded mailbox/backpressure                 PRESERVED
ActorRef.termination()                       PRESERVED
fatal Actor-incarnation failure semantics    PRESERVED
fresh-incarnation replacement                PRESERVED
no silent repair of failed mutable heap      PRESERVED
distributed reachability/failure authority   PRESERVED
implementation-private diagnostics freedom   PRESERVED
```

The §30 rules that the runtime must not silently repair arbitrary mutable Actor state and that replacement creates a fresh Actor remain retained. They are also supported elsewhere by the Actor/Future lifecycle model and are not treated as dispensable observability policy.

## Comparative evidence

D163 considered multiple approaches across established runtimes:

- **BEAM/Erlang** exposes process introspection such as status, queue length and reductions while keeping the process model and scheduler internals separate from application semantics.
- **Akka** treats telemetry as operational instrumentation and commonly uses sampling to avoid turning observability into continuous heavy coordination.
- **JVM/JFR/JMX** separate language semantics from runtime observability, with event/counter infrastructure designed to keep disabled or unconsumed instrumentation cheap.
- **.NET diagnostics/EventCounters/Metrics** similarly separate runtime health/metrics plumbing from ordinary language semantics.
- **Go runtime metrics/pprof** expose implementation-oriented operational information without making one global diagnostic representation part of ordinary program semantics.

The lesson is not that Protos should remove its scalability invariant. It is that Protos should retain the invariant while leaving the concrete metric schema, exposure mechanism, sampling strategy and tooling boundary open.

## Candidate result

### Candidate A — retain mandatory Core health fast path

**Selected.**

It preserves an explicit scaling guardrail while leaving representation and public API choices open.

### Candidate B — remove health/watchdog from portable Core

Rejected. It would allow a conforming implementation to defer health architecture until a future consumer appears, potentially baking in globally coordinated or blocking mechanisms that are expensive to reverse at scale.

### Candidate C — retain only semantic hooks already needed by Actor behavior

Rejected as insufficient. Lifecycle/mailbox/scheduler state alone does not constrain a future health implementation to remain O(1), non-blocking and free from global coordination.

### Candidate D — define a concrete administrative observability facility now

Rejected as premature. No evidence justifies selecting a public health object, metrics API, watchdog protocol, authority model, local/remote exposure, sampling contract or cluster-control facility now.

## Strongest argument against Candidate A

The retained rule constrains implementation architecture before a concrete public consumer exists. Some runtimes may prefer event streams, sampling, tracing or backend-specific diagnostic structures rather than continuously maintained health state.

That objection is accepted but bounded.

The current contract does not require a fixed field set or large always-on object. It only requires the mandatory fast path to remain inexpensive, O(1), non-blocking and free from global coordination, while explicitly permitting more expensive analysis through sampling or optional instrumentation.

That is a deliberately small portability constraint relative to the scaling failure modes it prevents.

## Pay-for-what-you-need and grow-as-you-need

Pay for what you need:

- no public health API is added now;
- no fixed metrics set is added now;
- expensive analysis remains optional/sampled;
- no distributed health/control-plane service is required by ordinary programs.

Grow as you need:

- implementation-private counters, epochs and probes may evolve;
- a future admin/observability facility can select its own authority and API from real consumer evidence;
- future metrics can be local, sampled, streamed or externally exported, provided the retained mandatory fast path does not regress into blocking/global coordination.

## Compatibility and implementation consequence

Candidate A is status quo.

```text
normative semantic delta      NONE
public API delta              NONE
runtime implementation delta  NONE
test migration                NONE
follow-up Ixxx                NOT REQUIRED
```

No Core source/spec change is authorized or required by D163.

## Approval provenance

The first recommendation during D163 analysis was Candidate B. The project owner rejected it because the scaling restrictions exist deliberately "para que nadie se columpie al escalar".

The decision analysis was then revised to recognize §30 as an architectural scalability guardrail rather than a speculative metrics API. Candidate A — KEEP was presented explicitly, including the retained boundaries above.

The project owner then explicitly approved that revised candidate in the active interaction on 2026-09-19:

```text
ok ahora si aprobada
```

```text
D163_STATUS=RATIFIED
SELECTED_CANDIDATE=A
RUNTIME_HEALTH_GUARDRAIL=KEEP
HEALTH_FAST_PATH_O1=KEEP
HEALTH_FAST_PATH_NON_BLOCKING=KEEP
HEALTH_FAST_PATH_NO_GLOBAL_COORDINATION=KEEP
FIXED_PUBLIC_HEALTH_API=NOT_REQUIRED
NORMATIVE_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_RECONCILIATION_REQUIRED=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
