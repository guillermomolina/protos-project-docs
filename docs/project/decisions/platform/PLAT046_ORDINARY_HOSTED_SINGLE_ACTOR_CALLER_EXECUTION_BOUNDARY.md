# PLAT046 — Ordinary hosted single-Actor caller execution boundary

Status: **RATIFIED**

Selected architecture: **Candidate B — direct caller-thread execution for ordinary synchronous hosted execution, with explicit local session serialization and stronger physical machinery added only when a concrete stronger capability requires it**.

Approval: explicit project-owner approval on 2026-10-02:

~~~text
Ok apruebo candidate B.
~~~

Decision Issue: `guillermomolina/protos#778`

Triggering performance work: `PERF025 / guillermomolina/protos#758`

Historical defect: `BUG008 / guillermomolina/protos#681` remains closed.

Product baseline at ratification:

~~~text
PROTOS_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
PROTOS_VERSION=0.3.145-SNAPSHOT
CURRENT_IMPLEMENTATION=DEDICATED_PROTOS_EMBEDDED_GUEST_CARRIER
CURRENT_GUEST_STACK_BUDGET=16_MIB
~~~

Nature: durable non-normative platform/runtime placement decision. Observable
Protos language and Standard Library semantics remain unchanged.

## Decision

For the ordinary hosted synchronous case:

~~~text
one host caller
one Protos Process
one RootActor
no active capability requiring independent guest concurrency
~~~

the physical execution path is:

~~~text
caller thread
  -> local session execution/serialization gate
  -> Process execution host
  -> Process Polyglot Context enter
  -> RootActor / RootTask guest execution
  -> Context leave
  -> return to caller
~~~

A second guest execution thread is **not** part of this minimum execution model.

The selected architecture is intentionally incremental:

~~~text
ordinary single-Actor synchronous execution
  -> existing caller is the physical carrier

additional Actor work
  -> existing lazy RuntimeHost-owned Actor carrier substrate

other stronger physical requirement
  -> add only the minimum mechanism justified by that concrete requirement
~~~

PLAT046 does not select a speculative strong-stack lane, carrier pool,
remaining-stack heuristic, fallback-on-`StackOverflowError`, or explicit
guest-stack/trampoline representation merely to preserve options for a case that
is not active in the ordinary path.

## Owner invariant

The project owner established during PLAT046-A that a mandatory second guest
execution thread in the minimal case is an architectural error when it exists
only to pre-pay for a stronger hypothetical capability.

~~~text
MINIMAL_ONE_CALLER_ONE_PROCESS_ONE_ROOTACTOR_SYNCHRONOUS_EXECUTION
  MUST_NOT_REQUIRE_A_SECOND_GUEST_EXECUTION_THREAD
  MERELY_FOR_UNUSED_CAPABILITY
~~~

This is an architectural invariant, not a performance hypothesis.

Benchmarking remains required after implementation to measure the performance
consequence. Benchmarking is not required to justify removal of unused
second-thread machinery from the minimum execution architecture.

## Protos philosophy basis

The selected Candidate B follows existing project philosophy rather than adding
a new performance-specific rule.

### Pay only for what you use

The maintained design philosophy explicitly includes unnecessary threads,
memory, synchronization, coordination, runtime initialization, failure modes and
operational machinery among costs that unused capability should avoid.

A one-caller/one-RootActor invocation therefore must not pay a private execution
thread, requested stack, queue, park/unpark and cross-thread publication merely
because some future execution might require stronger physical capacity.

### Pay as you grow

The existing concurrency ladder begins with ordinary sequential computation and
adds cooperative concurrency, isolated parallelism, persistent Actors,
multi-process/distributed Actors and cluster coordination as stronger needs
appear.

Candidate B preserves that direction physically: stronger execution machinery
is introduced when the stronger requirement becomes active.

### Lower layers remain protected

A stronger capability must not silently impose its cost model on a weaker layer.

Deep recursion, multiple external callers, child Actors, host-blocking I/O,
native-extension restrictions or future platform requirements may each justify
additional machinery. None of them makes that machinery part of the ordinary
single-caller execution floor without concrete evidence.

### Prefer independence over coordination

A simple synchronous operation on local state is preferred to additional
concurrency machinery when both preserve the same semantics.

Candidate B therefore replaces cross-thread handoff with the narrow local
serialization required by the reusable-session contract.

## Normative semantic boundary

PLAT046 changes no observable Protos semantics.

Preserved:

~~~text
PROCESS_IDENTITY=UNCHANGED
ROOTACTOR_IDENTITY=UNCHANGED
TASK_IDENTITY_AND_LIFECYCLE=UNCHANGED
ACTOR_LOCAL_SERIALIZATION=UNCHANGED
FUTURE_SEMANTICS=UNCHANGED
CANCELLATION_SEMANTICS=UNCHANGED
ERROR_AND_CONTROL_TRANSFER=UNCHANGED
MODULE_AND_CLOSURE_SEMANTICS=UNCHANGED
PROCESS_FAILURE_AND_TERMINATION=UNCHANGED
~~~

The specification already leaves physical carrier mapping and scheduler
organization as implementation machinery.

The semantic fact that RootActor execution is serialized therefore requires
serialization, not private thread ownership.

## Existing implementation seams supporting Candidate B

### RootTask direct-first execution

`ProtosRootTaskExecution.runRootTask(...)` already executes the fresh root
Task's first segment directly:

~~~text
The first segment runs directly;
only later runnable re-entries use the Actor queue.
~~~

Candidate B does not bypass Actor/Task semantics. It removes an outer host
thread handoff around an execution path that is already direct-first internally.

### Current-thread Context entry

`ProtosPolyglotProcessContext.callForRuntime(...)` delegates to
`ProtosPolyglotExecutionContext.callEntered(...)`.

That path enters/leaves the Polyglot Context on the current physical carrier,
while semantic state remains explicit in `ProtosActivation`.

`ProtosLanguage.isThreadAccessAllowed(...)` permits host-thread access.

No Protos thread-affinity invariant was found that requires the private
standalone-session carrier.

### Lazy child-Actor carrier substrate

`ProtosPolyglotRuntimeHost.actorSchedulerForRuntime()` already creates the
normal Actor executor lazily so a host that never schedules a child Actor pays
no Actor-carrier thread cost.

Candidate B composes with that existing PLAT010/PLAT011 architecture rather than
replacing it.

## Reusable-session serialization

The current private `ProtosGuestCarrier` provides serialization as an
implementation side effect. Candidate B makes that responsibility explicit.

A reusable hosted session must preserve:

~~~text
AT_MOST_ONE_SESSION_GUEST_OPERATION_AT_A_TIME=YES
MULTI_CALLER_SAFETY=YES
CALL_VS_CLOSE_EXCLUSION=YES
NO_NEW_GUEST_OPERATION_AFTER_CLOSE_CUTOVER=YES
SYNCHRONOUS_RESULT_AND_FAILURE_PROPAGATION=YES
~~~

The exact Java synchronization primitive is deliberately not standardized by
PLAT046.

A local lock/monitor/gate is permitted. The implementation must not reintroduce
a hidden private execution thread merely to obtain serialization.

The existing `ProtosPolyglotExecutionContext` lifecycle read/write lock does
not by itself replace this session-level gate: it protects Context lifecycle and
intentionally permits concurrent Process Context entries when semantics allow.

## Close/teardown boundary

Direct caller execution must preserve the established teardown behavior:

~~~text
close cutover is idempotent
new session calls fail after close cutover
already-started operation may finish according to existing contract
Process termination is requested
Process terminal completion is awaited
Process Context terminal disposition is awaited
RuntimeHost is closed after Process Context completion
module resolver is closed
first failure is retained and later cleanup failures are suppressed
~~~

These are ordering/lifecycle requirements, not dedicated-thread requirements.

## Deep-recursion / BUG008 boundary

BUG008 remains historical closed defect evidence and the checked-in 10,000-deep
recursive regression requirement remains retained.

~~~text
DEEP_RECURSION_10000_REQUIREMENT=RETAINED
BUG008_REOPEN_REQUIRED=NO
~~~

PLAT046 changes the architectural role of that requirement:

~~~text
STACK_10000_REQUIREMENT_IS_ORTHOGONAL_TO_BASE_THREAD_TOPOLOGY=YES
STACK_GATE_BLOCKS_BASE_SELECTION=NO
UNIVERSAL_STRONG_STACK_CARRIER_PREAUTHORIZED=NO
~~~

If an approved direct-caller implementation later demonstrates a concrete
supported execution shape that cannot satisfy the retained recursion requirement
with its physical caller, that observation owns a new bounded
capability-specific implementation/platform problem.

It must not silently restore a mandatory second execution thread to every
ordinary invocation.

Candidate B therefore supersedes the narrower PERF025-C2C/C2D implementation
assumption that the dedicated carrier must remain universal. It does **not**
invalidate their historical measurements or the retained recursion requirement.

## Benchmark consequence

The existing cross-Truffle benchmark remains the measurement authority.

Before Candidate B implementation, the repeated prepared paths are physically:

~~~text
GraalJS / GraalPy
  benchmark caller
    -> Value.execute()
    -> guest
    -> return

Protos
  benchmark caller
    -> ProtosGuestCarrier.call()
    -> queue
    -> park
    -> protos-embedded-guest
    -> guest
    -> unpark
    -> return
~~~

The current measurements are valid end-to-end measurements of the public
surfaces, but their absolute ratios must not be interpreted as pure
same-topology guest execution ratios.

After implementation:

~~~text
NEW_BENCHMARK_METHODOLOGY_REQUIRED=NO
EXISTING_PERF025_PREPARED_PRIMITIVE_RADAR=REUSE
REMEASURE_AFTER_IMPLEMENTATION=YES
~~~

Performance improvement is a falsifiable consequence, not an approval
precondition.

## External implementation precedent

The PLAT046-A research found a common compatible pattern:

- Truffle Polyglot Context execution normally occurs on the thread that enters
  the Context.
- GraalJS executes ordinary `Value.execute()` synchronously on the caller and
  adds multi-thread constraints only when more threads actually participate.
- GraalPy retains a real GIL requirement while optimizing the established
  single-thread/already-owned case and activates stronger machinery when another
  thread appears.
- Apple Pkl evaluates on the caller and creates a scheduler thread only when the
  optional timeout capability is configured.
- Espresso and Sulong associate thread-specific guest/runtime state with threads
  that actually participate instead of requiring an invisible private carrier
  for every synchronous host invocation.

These implementations are evidence, not semantic authority over Protos. They
show no hidden general Truffle requirement that invalidates Candidate B.

## Rejected candidates

### Candidate A — mandatory private 16 MiB carrier

Rejected for the minimum path.

It is mature and preserves a predictable private stack, but it imposes thread,
stack, queue, handoff and wakeup machinery on every ordinary hosted call.

That contradicts the owner invariant and the established pay-only/pay-as-you-grow
architecture.

### Candidate C — eliminate host-stack-proportional recursion first

Architecturally possible but not required to fix the minimum execution model.

It would reopen major interpreter/JIT, debugger, control, stack-trace,
continuation and Native surfaces before a concrete need establishes that cost.

### Candidate D — direct caller with heuristic stack transition/fallback

Rejected.

No portable precise remaining-stack budget or safe general migration of a live
Java/Truffle call stack was established. Catch-and-replay after
`StackOverflowError` is not an acceptable general semantics-preserving
fallback.

### Candidate E — universal shared RuntimeHost strong carrier pool

Rejected as the minimum path.

It improves thread-per-session scaling but still forces every ordinary
invocation through queue/handoff to another execution thread.

A shared special lane remains a possible future mechanism for a concrete
stronger requirement, but PLAT046 does not pre-authorize it.

### Defer base selection until depth-10,000 caller-stack proof

Rejected as an architectural gate.

It allows a speculative stronger resource requirement to determine the weaker
execution topology. The retained regression must still pass under the supported
final product, but it does not define whether `return 1` requires another
thread.

## GITHUB021 invariant/delta consistency

The exact approved Candidate B is consistent with the owner invariant recorded
during PLAT046-A.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS

OWNER_INVARIANT=
  MINIMAL_ONE_CALLER_ONE_PROCESS_ONE_ROOTACTOR_SYNCHRONOUS_EXECUTION
  MUST_NOT_REQUIRE_A_SECOND_GUEST_EXECUTION_THREAD
  MERELY_FOR_UNUSED_CAPABILITY

CANDIDATE_B_MATCHES_OWNER_INVARIANT=YES
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

Previously ratified platform decisions remain consistent:

~~~text
PLAT001_THREAD_IDENTITY_BOUNDARY=PRESERVED
PLAT010_ACTOR_NOT_THREAD=PRESERVED
PLAT011_SHARED_LAZY_RUNTIMEHOST_CARRIERS=PRESERVED
PLAT014_016_OPTIONAL_MACHINERY_BOUNDARY=PRESERVED
PLAT019_021_027_028_HOST_STACK_NOT_SEMANTIC_CONTINUATION=PRESERVED
PLAT040_OPTIONAL_PHYSICAL_STATE_NOT_UNIVERSAL=PRESERVED
PLAT043_044_SEMANTIC_VS_PHYSICAL_SPECIALIZATION_BOUNDARY=PRESERVED
PLAT045_NATIVE_GUEST_JIT_POLICY=PRESERVED

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The only direct delta is to the current implementation placement selected as a
historical BUG008/PERF025 workaround:

~~~text
CURRENT_UNIVERSAL_DEDICATED_SESSION_CARRIER
  -> NOT PART OF ORDINARY_MINIMUM_EXECUTION_MODEL

RETAINED_DEEP_RECURSION_REQUIREMENT
  -> UNCHANGED
~~~

## Implementation authority

PLAT046 ratification releases PERF025 / #758 to implement Candidate B.

The first implementation slice should be bounded to the reusable standalone
hosted-session path that owns the benchmark mismatch.

Required first-slice outcome:

~~~text
ORDINARY_PROTOS_STANDALONE_HOSTED_SESSION_GUEST_EXECUTION=CALLER_THREAD
SESSION_MULTI_CALLER_SERIALIZATION=EXPLICIT_LOCAL_GATE
PROCESS_CONTEXT_ENTRY=UNCHANGED
ROOT_TASK_EXECUTION=UNCHANGED
CHILD_ACTOR_SCHEDULER=UNCHANGED_AND_LAZY
CLOSE_ORDERING=PRESERVED
PUBLIC_EMBEDDING_RESULT_ERROR_CONTRACT=PRESERVED
~~~

The implementation should not simultaneously redesign:

- Actor scheduling;
- Test Tool parallel execution;
- CLI command-carrier policy outside the exact hosted-session path unless the
  slice proves it is the same bounded ownership surface;
- guest recursion representation;
- stack fallback/strong-lane policy;
- benchmark methodology.

After the product slice is validated and published, reuse the existing PERF025
prepared primitive benchmark radar unchanged.

## Ratified result

~~~text
PLAT046_STATUS=RATIFIED
SELECTED_CANDIDATE=B

ORDINARY_PHYSICAL_CARRIER=CALLER_THREAD
DEDICATED_SECOND_GUEST_EXECUTION_THREAD_FOR_MINIMAL_PATH=NO

SESSION_SERIALIZATION=EXPLICIT_LOCAL_GATE
MULTI_CALLER_SAFETY=REQUIRED
CALL_VS_CLOSE_EXCLUSION=REQUIRED
CONTEXT_ENTER_LEAVE=REQUIRED
ROOTACTOR_TASK_SEMANTICS=UNCHANGED

CHILD_ACTOR_CARRIERS=
  EXISTING_LAZY_RUNTIMEHOST_OWNED_SUBSTRATE

PREBUILT_STRONG_STACK_MECHANISM=NO
DEEP_RECURSION_10000_REQUIREMENT=RETAINED
STACK_GATE_BLOCKS_BASE_SELECTION=NO

EXISTING_PERF025_BENCHMARK_RADAR=REUSE_UNCHANGED
REMEASURE_AFTER_IMPLEMENTATION=YES

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008_REOPEN_REQUIRED=NO

IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_OWNER=PERF025/#758
~~~

## Evidence and references

- `guillermomolina/protos#778` — PLAT046 decision Issue.
- `guillermomolina/protos#758` — PERF025 implementation/performance owner.
- `guillermomolina/protos#681` — BUG008 historical closed defect.
- `docs/project/evidence/PLAT046/PLAT046_INTAKE_AND_TRIGGER_EVIDENCE.md`.
- `docs/project/evidence/PLAT046/PLAT046_A_EXHAUSTIVE_MINIMAL_SINGLE_ACTOR_EXECUTION_RESEARCH.md`.
- `docs/project/evidence/PLAT046/PLAT046_CANDIDATE_B_RATIFICATION_EVIDENCE.md`.
- `docs/project/evidence/PERF025/PERF025_E3B_CARRIER_TRANSPORT_AB_REFERENCE.md`.
- `docs/project/decisions/platform/PLAT010_ACTOR_PLATFORM_CARRIERS.md`.
- `docs/project/decisions/platform/PLAT011_RUNTIMEHOST_CARRIER_SUBSTRATE.md`.
- `docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md`.
- `docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md`.
