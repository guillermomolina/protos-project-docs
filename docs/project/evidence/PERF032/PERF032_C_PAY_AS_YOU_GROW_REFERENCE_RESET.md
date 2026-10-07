# PERF032-C — pay-as-you-grow callable-entry reference reset

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF032
ISSUE=guillermomolina/protos#831
PRODUCT_REVISION=c21ff278e6dca1219429cc7f08568c54af2bd951
PRODUCT_VERSION=0.3.274-SNAPSHOT
WORKLOAD=primitive-return-literal
OBSERVABLE_RESULT=1
~~~

This record is durable non-normative performance/architecture evidence. No
product, benchmark or normative specification file is changed by this record.

## Why the prior PERF032-C recommendation was reset

The first PERF032-C analysis correctly identified that the current prepared
Protos path spends the dominant part of its time before the first compiled guest
CallTarget, but it proposed preserving eager RootTask/Actor machinery while
moving the surrounding host boundary under standard Truffle host-to-guest
execution.

That preservation premise is rejected.

The project owner reiterated the cross-cutting Protos invariant that stronger
capabilities are physically absent from the weaker case until the program
actually needs them: RootActor is physically "no Actor" in the single-Actor
minimum case, root coroutine is physically "no coroutine" in the single-
coroutine case, Process/Node/Cluster machinery is not prepaid when unused, and
local/captured state is materialized only when its observability requires it.

The revised research therefore treats GraalJS/GraalPy canonical callable
execution as the implementation reference and requires current Protos machinery
to justify every additional minimum-path layer.

## Accepted PERF032 measurements

PERF032-A retained current profiling evidence:

~~~text
PreparedTopLevel.invoke                         97.9% inclusive
ProtosPolyglotExecutionContext.callEntered      89.2%
ProtosRootTaskExecution.runRootTask             53.9%
OptimizedCallTarget.call (guest compiled unit)   7.5%
~~~

Its residual attribution included:

~~~text
fresh RootTask lifecycle ~=44%
Polyglot Context enter/leave ~=18%
~~~

The later accepted PERF032-B evidence placed approximately 187-190 ns/call
before the first Truffle CallTarget while the compiled guest call itself was
approximately 44-47 ns/call.

The retained PERF025 admitted cross-Truffle reference was:

~~~text
primitive-return-literal:
  Protos  216.56 ns/call
  GraalJS  43.90 ns/call
  GraalPy  46.92 ns/call
~~~

The guest compiled unit is already in the same order of magnitude as the peer
total call cost. The unresolved question is therefore the Protos-owned envelope,
not a requirement to optimize the literal body.

## Surface mismatch in the retained benchmark

The retained benchmark harness executes the peers through the canonical
Polyglot callable surface:

~~~text
GraalJS/GraalPy
  Value run = ...
  run.execute()
~~~

while Protos executes:

~~~text
ProtosStandaloneHostedSession.PreparedTopLevel prepared = ...
prepared.invoke()
~~~

The latter routes through Protos-owned session/process/context/RootTask
machinery before reaching the guest target.

The historical measurements are valid measurements of those public surfaces,
but the ratio is not a pure same-topology callable comparison.

## Current Protos evidence against universal Task ownership

At the exact product revision above:

- `ProtosClosureInvoker.invoke(...)` is explicitly synchronous non-Task
  execution;
- `ProtosActivation.task()` is optional;
- PERF025 H1 proves `() => { 1 }` can execute through the compact source-call
  ABI without materializing a rich callee `ProtosActivation`;
- the same H1 coverage separately proves exact Task provenance when a Task is
  actually present;
- PLAT040 rejects a universal rich-activation/prepared-call carrier hot path and
  requires conditional guest-state materialization;
- D189 defines a synchronous foreign callback as ordinary activation of the exact
  Protos Closure and explicitly says that callback invocation creates no Task,
  Future, structured-concurrency scope or cancellation checkpoint.

Therefore a fresh RootTask is not an intrinsic property of executing a Protos
Closure.

## Reference architecture

The accepted implementation reference is:

~~~text
GraalJS/GraalPy:
Value.execute()
  -> framework HostToGuestRootNode
  -> InteropLibrary.execute
  -> language callable
  -> guest CallTarget

Protos target:
Value.execute()
  -> framework HostToGuestRootNode
  -> InteropLibrary.execute(Protos callable)
  -> Protos callable execute node
  -> compact ordinary Closure invocation
  -> cached/direct guest call
  -> result
~~~

The target is structural convergence, not a promised numerical equality before
measurement.

## Rejected universal minimum-path machinery

For an ordinary callable that does not use stronger capabilities, no current
implementation layer is protected merely because Protos supports that stronger
capability.

The following are therefore rejected as universal minimum-path requirements:

~~~text
fresh RootTask
ProtosTask allocation
Actor live-task registration
pre-start cancellation bookkeeping
Actor task dispatch
terminal Task polling/publication
rich callee activation
per-call Process execution-host routing
Protos explicit Context.enter/leave envelope
Protos host ThreadLocal bookkeeping when unobserved
ProtosExecutionOutcome wrapping for canonical Value.execute
reusable-session multi-caller gate on the weaker canonical surface
generic indirect call when the established stable direct ABI applies
~~~

Each stronger mechanism remains available when its capability is actually
active.

## PLAT046 reconciliation

PLAT046 is not discarded. Its central pay-as-you-grow invariant is extended
inward.

The 2026-10-02 implementation constraint `ROOT_TASK_EXECUTION=UNCHANGED`
bounded the F1 carrier-topology slice. It is not permanent authority requiring
eager RootTask materialization for every future callable surface.

The 2026-10-07 project-owner approval authorizes the durable PLAT046 amendment
that makes this distinction explicit.

## Owner approval and routing

The project owner approved the revised direction in the active interaction:

~~~text
ok aprobado
~~~

The complete implementation/convergence work has independent closure and is
allocated as:

~~~text
NEW_OWNER=PERF033/guillermomolina/protos#832
TITLE=Make ordinary Protos callable interop fully pay-as-you-grow

OLD_PERF032_C_CANDIDATE_A_AS_STATED=REJECTED
UNIVERSAL_ROOT_TASK_FOR_MINIMUM_CALL=REJECTED
GRAALJS_GRAALPY_REFERENCE=ACCEPTED
STRUCTURAL_CONVERGENCE_SCOPE=COMPLETE_MINIMUM_PATH

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PRODUCT_CHANGE=NO
BENCHMARK_CHANGE=NO
COMMAND_EXECUTION=NONE
~~~

PERF032 remains the causal single-workload evidence owner; PERF033 owns the
cross-cutting implementation required to remove or justify the complete fixed
minimum-call envelope.

## AI-assistance disclosure

This evidence was materially prepared with AI assistance from ChatGPT using
current repository state, retained PERF025/PERF032 evidence, current Graal
implementation evidence inspected during PERF032-C, and explicit project-owner
approval.
