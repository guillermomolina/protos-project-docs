# PLAT046 — intake and trigger evidence

Date: 2026-10-02

## Allocation identity

~~~text
WORK_ITEM=PLAT046
GITHUB_ISSUE=guillermomolina/protos#778
TITLE=Hosted guest carrier boundary and pay-as-you-grow execution cost
STATUS=OPEN
DECISION_SELECTED=NO
IMPLEMENTATION_AUTHORIZED=NO

TRIGGERED_BY=PERF025/#758
HISTORICAL_BUG=BUG008/#681 CLOSED_DO_NOT_REOPEN
~~~

PLAT046 was allocated after PERF025-E2/E3B established that the retained
dedicated guest-carrier transport still imposes a material fixed cost on every
reusable hosted invocation even after the per-call monitor/allocation cleanup.

This record is non-normative evidence. The live Issue is the coordination
authority and no PLAT046 candidate has been approved.

## Current product identity

~~~text
CURRENT_PRODUCT_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
CURRENT_PRODUCT_VERSION=0.3.145-SNAPSHOT
CURRENT_PRODUCT_SUBJECT=PERF025-E2: guest carrier transport cleanup
~~~

The E2 implementation retained:

~~~text
DEDICATED_GUEST_CARRIER=YES
GUEST_STACK_BUDGET=16_MIB
LINKED_BLOCKING_QUEUE=YES
CARRIER_SERIALIZATION=YES
CALLER_THREAD_EXECUTES_GUEST=NO
~~~

while replacing per-call one-element state arrays, the queued completion
wrapper and monitor wait/notify with a typed request plus
`LockSupport.park/unpark`.

## Exact PERF025-E3B trigger evidence

Exact causal comparison:

~~~text
PRE_E2_REVISION=c1eb2c2e1a811d70fcf09f5526217c1aaf14d141
PRE_E2_VERSION=0.3.144-SNAPSHOT

E2_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
E2_VERSION=0.3.145-SNAPSHOT

REFERENCE_PRODUCER_REVISION=3fb036ce86b6c452588230b5322967f55aaf0160
RETAINED_EVIDENCE_REVISION=8441ebfb6fe970cfd9a8994f182cea974f4498a0
MEASUREMENT_DEFINITION=jvm-protos-carrier-e2-ab-v1

REFERENCE_OBSERVATIONS=6
CORRECTNESS_PASS=6/6
ADMISSION_PASS=6/6
REFERENCE_RETRY=NO
~~~

Steady amortized p50:

| workload | PRE_E2 ns/call | E2 ns/call | absolute delta | time change |
| --- | ---: | ---: | ---: | ---: |
| primitive-return-literal | 6692.741 | 6173.223 | -519.519 ns | -7.76% |
| primitive-closure-call | 7522.591 | 7181.618 | -340.973 ns | -4.53% |
| primitive-method-call | 7013.635 | 6589.426 | -424.209 ns | -6.05% |

E2 therefore removed a real few-hundred-nanosecond fixed cost but did not
eliminate the several-microsecond prepared-call floor.

## Benchmark/runtime execution-shape evidence

The current cross-Truffle runner does not create a Protos-only benchmark
thread.

For GraalJS/GraalPy, the runner retains a prepared executable and invokes:

~~~text
benchmark caller thread
  -> Value.execute()
  -> guest
  -> return
~~~

For Protos, the prepared runner invokes
`ProtosStandaloneHostedSession.PreparedTopLevel.invoke()`, whose current
product implementation performs:

~~~text
benchmark caller thread
  -> ProtosGuestCarrier.call()
  -> LinkedBlockingQueue
  -> park caller
  -> dedicated protos-embedded-guest thread
  -> guest execution
  -> unpark caller
  -> return
~~~

`ProtosStandaloneHostedSession` explicitly states that guest operations run
on the dedicated carrier and that the calling thread never executes guest
code.

The additional handoff is therefore Protos runtime architecture, not benchmark
machinery.

## Historical reason for the carrier

BUG008 / #681 established that the then-current Bytecode-backed runtime could
not execute the canonical recursive Closure benchmark at its checked-in depth
of 10,000 on the ambient/default host stack.

The published BUG008 repair classified the root cause as a host-resource-budget
gap and provisioned a fixed 64 MiB guest carrier stack.

Subsequent runtime work removed artificial per-level execution-root multipliers:

~~~text
PLAT042 / PERF025-C:
  ordinary semantic/structured interpreter topology simplified

PLAT043 / PERF025-C2B:
  standard Boolean-control ownership moved into semantic interpreter

PLAT044 / PERF026-B:
  eligible immediate literal callbacks execute inline without callback
  CallTarget

PERF025-C2D:
  fixed carrier stack reduced from 64 MiB to 16 MiB
~~~

The dedicated carrier nevertheless remained mandatory for ordinary hosted
execution.

## Pay-only-for-use / pay-as-you-grow tension

The maintained Protos design philosophy requires that unused capability avoid
unnecessary cost including:

~~~text
threads
synchronization
coordination
runtime machinery
~~~

and defines pay-as-you-grow as stronger capabilities adding only the minimum
new machinery required by the stronger problem while lower levels retain their
cost model.

The current hosted architecture charges an ordinary sequential invocation,
including a top-level Closure whose body only returns a literal, for a
dedicated-thread handoff originally introduced to guarantee deep-recursion host
stack capacity.

That tension is concrete enough to require a durable platform decision rather
than another implementation-local PERF025 optimization.

## Decision boundary

PLAT046 must determine whether to retain the always-on carrier or replace it
with an architecture in which ordinary sequential guest execution avoids that
cost while the retained deep-recursion capability remains available.

The minimum candidate space allocated by the Issue includes:

~~~text
A = retain mandatory dedicated carrier
B = direct ordinary caller-thread execution + explicit deep-stack path
C = eliminate host-stack-proportional guest recursion
D = direct path with bounded safe fallback/transition
E = materially distinct architecture discovered by research
~~~

No candidate is selected by this record.

## Preserved invariants

Unless explicitly reopened through the PLAT046 approval gate:

~~~text
OBSERVABLE_PROTOS_SEMANTICS=UNCHANGED
TASK_ACTOR_PROCESS_FUTURE_SEMANTICS=UNCHANGED
MULTI_CALLER_SAFETY=REQUIRED
CLOSE_TEARDOWN_BEHAVIOR=REQUIRED
DEEP_RECURSION_10000_CAPABILITY=RETAINED_REQUIREMENT
BUG008=HISTORICAL_CLOSED
PLAT045_NATIVE_INTERPRETER_ONLY_POLICY=UNCHANGED
BYTECODE_DSL_ARCHITECTURE=UNCHANGED
~~~

## Coordination consequence

~~~text
PERF025_STATUS=PAUSE_FOR_PLAT046
FURTHER_CARRIER_MICRO_OPTIMIZATION=DEFERRED
EXISTING_PERF025_MEASUREMENT_RADAR=REUSE
NEW_BENCHMARK_METHODOLOGY_REQUIRED=NO

NEXT_SLICE=PLAT046-A
NEXT_SLICE_TYPE=INVESTIGATION
COMMAND_EXECUTION=NONE
~~~

PLAT046-A must produce the full GITHUB010 platform decision packet and stop for
explicit project-owner selection before implementation.

## Cross references

- `guillermomolina/protos#778` — PLAT046 live decision Issue.
- `guillermomolina/protos#758` — PERF025 trigger/consumer.
- `guillermomolina/protos#681` — BUG008 historical stack defect.
- `guillermomolina/protos@ff618f0dd8ef680f7884c9145552f77912f040a2` — current E2 product state.
- `guillermomolina/protos-benchmarks@8441ebfb6fe970cfd9a8994f182cea974f4498a0` — retained E3B A/B evidence.
- `docs/project/evidence/PERF025/PERF025_E3B_CARRIER_TRANSPORT_AB_REFERENCE.md`.
- `docs/project/registries/PLATFORM_ARCHITECTURE_DECISIONS.md`.
