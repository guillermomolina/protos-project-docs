# PERF025-E2 — guest-carrier transport cleanup

Date: 2026-10-02

## Publication identity

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-E2
TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos

SLICE_BASE_REVISION=c1eb2c2e1a811d70fcf09f5526217c1aaf14d141
SLICE_BASE_VERSION=0.3.144-SNAPSHOT
PROTOS_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
PROTOS_VERSION=0.3.145-SNAPSHOT
COMMIT_SUBJECT=PERF025-E2: guest carrier transport cleanup

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

The immediate parent is the unrelated BUG014 packaging fix. Using that exact
parent as the E2 baseline isolates the carrier-transport change from the
intervening version bump and packaging work.

## Trigger evidence

PERF025-E1 selected the retained-carrier transport as the next bounded target
after current reusable-call evidence showed a roughly 4.5 microsecond prepared
return-literal floor and historical PERF024 evidence had measured the carrier
handoff itself at approximately 4.8 microseconds per call.

Static comparison also established that
`ProtosGuestCarrier.java` was byte-for-byte unchanged between the historical
carrier-control revision and the pre-E2 product revision, so another open-ended
profiling campaign was not required before attempting the minimal transport
cleanup.

The architectural constraints inherited from PERF025-C2C/C2D remained:

~~~text
DEDICATED_GUEST_CARRIER=RETAIN
GUEST_CARRIER_STACK=16_MIB
CARRIER_SERIALIZATION=RETAIN
AMBIENT_CALLER_STACK_DEPENDENCE=NOT_INTRODUCED
CARRIER_RETIREMENT_AUTHORIZED=NO
~~~

## Published delta

The exact E2 commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java
src/test/java/com/guillermomolina/protos/execution/ProtosGuestCarrierTest.java
~~~

The product version advances:

~~~text
0.3.144-SNAPSHOT -> 0.3.145-SNAPSHOT
~~~

The production transport changes only inside
`ProtosGuestCarrier.call()`.

Before E2, each call used:

~~~text
Object[1]
Throwable[1]
boolean[1]
queued completion lambda
per-call synchronized(done)
wait()/notifyAll()
LinkedBlockingQueue node
supplied Callable
~~~

After E2, the carrier queues one private typed request object directly:

~~~text
CarrierCall<T> implements Runnable
  operation
  waiter Thread
  result
  failure
  volatile done
~~~

Completion uses:

~~~text
LockSupport.park(this)
LockSupport.unpark(waiter)
~~~

The volatile completion flag publishes result/failure before the waiting caller
observes completion.

## Preserved execution contract

The implementation retains:

~~~text
DEDICATED_GUEST_CARRIER=YES
GUEST_STACK_BUDGET=16_MIB
LINKED_BLOCKING_QUEUE=RETAINED
CARRIER_SERIALIZATION=RETAINED
MULTI_CALLER_SAFETY=RETAINED
FIFO_SUBMISSION=RETAINED
STOP_ORDERING=RETAINED
CLOSE_REJECTION=RETAINED

SYNCHRONOUS_RESULT_PROPAGATION=RETAINED
IOEXCEPTION_PROPAGATION=RETAINED
RUNTIME_EXCEPTION_PROPAGATION=RETAINED
ERROR_PROPAGATION=RETAINED
OTHER_CHECKED_THROWABLE_WRAPPING=RETAINED

CALLER_INTERRUPT_DOES_NOT_CANCEL_OPERATION=YES
CALLER_INTERRUPT_STATUS_RESTORED=YES
CARRIER_THREAD_IDENTITY=UNCHANGED
AMBIENT_CALLER_STACK_DEPENDENCE=NO
~~~

No direct caller-thread guest path, queue architecture replacement, batching,
public embedding API change, Process/Actor semantic change, Context-entry
semantic change, or stack-budget change is introduced.

## Interruption handling

`LockSupport.park()` returns immediately when the calling thread is already
interrupted. E2 therefore records interruption while waiting, clears it via
`Thread.interrupted()` so later parks may actually block, continues waiting for
the already-submitted operation, and restores the interrupt status before
returning or rethrowing the operation outcome.

This preserves the previous `wait()`-based contract: interruption does not
abandon or cancel the guest operation.

## Focused regression coverage

E2 adds
`src/test/java/com/guillermomolina/protos/execution/ProtosGuestCarrierTest.java`.

The focused transport coverage includes:

- operation executes on the dedicated carrier and returns its result;
- `IOException` propagation;
- `RuntimeException` propagation;
- `Error` propagation;
- wrapping of another checked exception in `IllegalStateException`;
- interrupt already set before a call;
- interruption while waiting;
- concurrent callers serialized on one carrier;
- close/submitted-work ordering and rejection after close.

The tests use deterministic synchronization primitives rather than timing
assertions for the concurrency/ordering contracts.

## Validation evidence

The maintainer reports that every requested E2 validation and the integrated
test suite passed on the published candidate before push:

~~~text
FOCUSED_VALIDATION=PASS
FULL_VALIDATION=PASS
ALL_REQUESTED_TESTS=PASS
~~~

This durable record preserves that human-reported validation result. It does not
claim that a new performance reference has already been run.

## Performance claim boundary

E2 is an implementation publication, not a measurement result.

Therefore:

~~~text
PERFORMANCE_IMPROVEMENT_CLAIMED_BY_E2=NO
REFERENCE_MEASUREMENT_REQUIRED=YES
OPEN_ENDED_PROFILING_REQUIRED=NO
~~~

The next PERF025 slice should measure the exact parent/candidate pair on the
existing prepared reusable-call primitive surface:

~~~text
PRE_E2_REVISION=c1eb2c2e1a811d70fcf09f5526217c1aaf14d141
PRE_E2_VERSION=0.3.144-SNAPSHOT
E2_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
E2_VERSION=0.3.145-SNAPSHOT
RUN_MODE=prepared

WORKLOADS=
  primitive-return-literal
  primitive-closure-call
  primitive-method-call
~~~

The exact parent/candidate pair is required because the parent already contains
BUG014; comparing E2 directly with the older PERF025-D3 FINAL revision would
mix an unrelated intervening product commit into the causal delta.

## PERF025 consequence

~~~text
PERF025_E2=COMPLETE
PERF025_STATUS=OPEN
NEXT_SLICE=PERF025-E3
NEXT_SLICE_TYPE=BENCHMARK_HARNESS_AND_MEASUREMENT
PRODUCT_CHANGE=NONE
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

PERF025-C2C remains authoritative for the retained dedicated-carrier
architecture. E2 optimizes the handoff transport inside that architecture; it
does not reopen carrier retirement.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#681` — historical BUG008, closed.
- `guillermomolina/protos@c1eb2c2e1a811d70fcf09f5526217c1aaf14d141` — exact E2 parent / PRE_E2 baseline.
- `guillermomolina/protos@ff618f0dd8ef680f7884c9145552f77912f040a2` — published E2 candidate.
- `docs/project/evidence/PERF025/PERF025_C2C_POST_B_PRIME_STACK_GATE_CLOSURE.md`.
- `docs/project/evidence/PERF025/PERF025_C2D_GUEST_CARRIER_STACK_BUDGET_REDUCTION.md`.
- `docs/project/evidence/PERF025/PERF025_D3_FINAL_REUSABLE_CALL_REFERENCE_AND_CLOSURE.md`.
