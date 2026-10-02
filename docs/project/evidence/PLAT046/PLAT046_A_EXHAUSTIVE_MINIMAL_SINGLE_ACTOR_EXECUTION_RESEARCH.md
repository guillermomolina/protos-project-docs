# PLAT046-A — exhaustive minimal single-Actor hosted-execution research

Date: 2026-10-02

Status: **PROPOSED — research complete, exact implementation architecture awaiting project-owner selection**

Formal decision: `PLAT046 / guillermomolina/protos#778`

Consumer: `PERF025 / guillermomolina/protos#758`

Historical defect: `BUG008 / guillermomolina/protos#681` remains closed.

## Research baseline

~~~text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
PRODUCT_VERSION=0.3.145-SNAPSHOT
PRODUCT_SUBJECT=PERF025-E2: guest carrier transport cleanup

PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
RESEARCH_DATE=2026-10-02

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
IMPLEMENTATION_AUTHORIZED=NO
~~~

This record completes the PLAT046-A comparative investigation required by
`AGENTS.work/DESIGN.md`. It is durable non-normative evidence, not a ratified
platform decision.

## Project-owner invariant recorded under GITHUB021

During PLAT046-A the project owner corrected the investigation model in a way
that materially constrains the active platform decision.

For the ordinary hosted case:

~~~text
one host caller
one Protos Process
one RootActor
synchronous execution
no active capability requiring independent concurrency
~~~

a mandatory second execution thread is not a performance hypothesis to be
accepted or rejected by benchmark results. Introducing that second execution
thread merely because a stronger hypothetical case may need it is an
architectural error.

The owner further fixed the research sequence:

1. investigate the best semantics-preserving way to execute the one-Actor case;
2. compare how mature peer runtimes implement the same physical boundary and
   identify any hidden cost;
3. select and implement the architecture only after that investigation;
4. re-run the existing benchmark radar afterward;
5. investigate multi-thread/multi-Actor or exceptional stack requirements when
   those stronger cases are actually being studied.

The current approximately-large primitive cross-language ratios remain valid
measurements of the current public embedding surfaces, but they must not be
interpreted as pure guest-language execution ratios while Protos inserts a
mandatory thread handoff that the compared JS/Python paths do not.

This owner invariant supersedes any earlier PLAT046-A reasoning that treated a
10,000-deep ordinary-caller stack experiment as a prerequisite for selecting the
minimal execution architecture.

## Exact decision after the owner correction

PLAT046 no longer asks:

> Can a benchmark prove that removing the dedicated carrier is fast enough or
> that the caller stack is large enough?

It asks:

> What is the smallest correct physical execution model for ordinary synchronous
> hosted Protos execution with one Process and only its RootActor, and what
> machinery must be added only when a stronger capability actually requires it?

The retained 10,000-deep regression remains a real conformance requirement. It
is orthogonal to whether an otherwise sequential one-Actor invocation should
manufacture a second execution thread.

## Protos philosophy audit

PLAT046 is unusually constrained by explicit existing philosophy. The following
are not new principles invented for this decision.

### Small universe; mechanisms over institutions

`AGENTS.work/REFERENCE.md` requires a small conceptual universe and general
mechanisms rather than permanent special institutions. A physical helper thread
must therefore earn its existence from a concrete physical requirement; it is
not justified merely because the runtime could eventually need concurrency.

### Ordinary things should remain ordinary

The same reference requires ordinary facilities to remain ordinary and warns
against parallel universes introduced for implementation convenience.

For PLAT046 this means an ordinary synchronous call should remain an ordinary
synchronous call unless some real semantic/platform boundary requires more
machinery.

### Pay only for what you use

`docs/design/PROTOS_DESIGN_PHILOSOPHY.md` §14 is explicit that unused
capability should avoid unnecessary:

~~~text
runtime initialization
memory
threads
synchronization
coordination
network infrastructure
metadata
conceptual complexity
failure modes
operational machinery
~~~

The same section gives the intended shape by example:

~~~text
single-threaded program
  -> should not need unused concurrency machinery

local Actor
  -> should not require cluster discovery

standalone Process
  -> should not require membership consensus
~~~

The current reusable hosted path violates the same rule physically: a
single-caller/single-RootActor invocation pays a second platform thread, a
second stack, queueing, park/unpark and cross-thread coordination even when no
guest concurrency is requested.

### Pay as you grow

`docs/design/PROTOS_DESIGN_PHILOSOPHY.md` §15 gives the explicit ladder:

~~~text
ordinary sequential computation
        |
        v
cooperative concurrency
        |
        v
isolated CPU parallelism
        |
        v
persistent isolated Actors
        |
        v
multi-process / distributed Actor systems
        |
        v
cluster coordination where actually required
~~~

Each step is required to add only the minimum machinery required by the stronger
problem.

A second execution thread in the first step because a later step might need one
reverses that architecture.

### Scale by composition without infecting lower layers

§16 says a new scaling layer is justified only by a real lifecycle, identity,
isolation, authority, persistence, failure, distribution or coordination
boundary and should solve that boundary without infecting simpler layers.

§17 is stronger still:

~~~text
Strengthening one layer must preserve the guarantees and cost model
of weaker layers unless a deliberate language-wide change is justified.
~~~

PLAT046 therefore cannot make ordinary sequential execution pay the physical
cost model of a stronger execution case merely to preserve an implementation
shortcut.

### Prefer independence over coordination

§18 explicitly notes that a simple synchronous operation on local state can be
preferable to additional concurrency machinery. The goal is to minimize
required coordination, not merely the count of locks.

This is directly relevant to a direct caller path with a narrow session
serialization gate: one local uncontended gate is qualitatively different from
queueing every invocation to another execution thread.

### Qualitative threshold: zero to one matters

`AGENTS.work/REFERENCE.md` says the distance from zero instances of a
mechanism to one can be architecturally larger than the distance from one to
ten.

PLAT046 is exactly such a threshold:

~~~text
0 extra execution threads
  !=
1 mandatory extra execution thread
~~~

The second state introduces stack reservation, scheduling, handoff,
synchronization, lifecycle and failure machinery that the first state does not
have.

### Concurrency hides machinery, not semantics

§20 says callers should not normally have to reason about OS threads, worker
pools, carrier threads, mutexes, barriers or scheduler queues. The runtime may
use those when needed, but they are machinery, not semantic identity.

A RootActor therefore does not acquire a physical-thread requirement simply by
being an Actor.

### Semantics before implementation

`AGENTS.work/REFERENCE.md` explicitly states that Truffle, GraalVM, JVM
behavior, tests and the current implementation do not define Protos semantics.

BUG008's physical workaround therefore cannot become a semantic premise for
PLAT046 merely because it has existed for several releases.

## Normative Protos audit

### Minimal Process/RootActor is intentionally lightweight

`spec/io/PROCESS_IO.md` states that every Protos execution has one Process and
begins with one RootActor even if the RootActor is the only Actor ever created.
It immediately adds that Process existence does not require heavyweight Actor,
Node, Cluster, routing or distributed-runtime infrastructure and that minimal
standalone execution may remain lightweight.

The mandatory Process and RootActor are therefore semantic structure, not an
instruction to instantiate the full physical Actor scheduler.

### RootActor is an ordinary Actor

`docs/design/CONCURRENCY_DESIGN.md` records the intended semantic topology:
RootActor is an ordinary Protos Actor, while Process/Node/Cluster runtime
entities remain distinct.

No normative rule equates an Actor with a Java thread.

### Actor scheduling semantics do not specify carrier topology

`spec/concurrency/ACTORS.md` requires Actor-local execution serialization,
ordering and weak fairness where applicable. It explicitly does not specify a
particular scheduler queue, carrier mapping, round-robin policy or work-stealing
policy.

The semantic requirement for one Actor not to execute two ordinary segments
concurrently is therefore a serialization requirement, not a dedicated-thread
requirement.

### Distribution already applies the same pay-as-you-grow rule

`spec/concurrency/DISTRIBUTED_RUNTIME.md` states that no distributed
coordination mechanism is required merely because Groups or multiple Actors
exist; coordination is paid only when an active invariant or durability
requirement actually needs it.

PLAT046 applies the same principle one layer lower: the existence of one
RootActor is not sufficient evidence for a second physical execution thread.

## Ratified platform-decision audit

### PLAT001 — Context hosting is placement, not semantic thread identity

PLAT001 ratified one multithread Truffle Context per hosted Protos Process and
explicitly declared false:

~~~text
Truffle Engine  == Protos Node
Truffle Context == Protos Process
Java Thread     == Protos Actor
~~~

Carrier threads enter/leave the Process Context; they do not become semantic
Actor or Process owners.

PLAT046 must preserve this separation.

### PLAT010 — bounded carriers, not one thread per Actor

PLAT010 rejected thread-per-Actor and an additional M:N scheduler layer. The
Actor scheduler owns semantic turns while physical carrier count remains
bounded runtime capacity.

Its core lesson for PLAT046 is stronger than merely "pools are good":

~~~text
semantic Actor cardinality does not determine physical thread cardinality
~~~

A single RootActor therefore does not justify one private physical carrier by
identity.

### PLAT011 — semantic Process creation must not allocate a private pool

PLAT011 rejected a CPU-sized carrier pool per Process and selected a shared
RuntimeHost-owned substrate because Process cardinality must not silently drive
thread cardinality.

That same rule is inconsistent with one permanently owned execution thread per
reusable hosted session when the session has no stronger physical requirement.

### PLAT014 / PLAT016 — optional execution machinery is not universal machinery

PLAT014 keeps continuation state with the Task and rejects a carrier per
suspended task. PLAT016 keeps ordinary Closure execution ordinary and avoids
default machinery for capabilities that are not used.

These are direct precedents for removing a universal physical mechanism from a
weaker path while preserving the stronger capability elsewhere.

### PLAT019 / PLAT021 / PLAT027 / PLAT028 — host stack/thread is not semantic state

The continuation/control decisions repeatedly establish that surviving guest
state must not be a parked Java/native continuation frame or carrier identity.

This is important for PLAT046: changing the physical thread used by an ordinary
synchronous call does not change Actor/Task/continuation identity as long as the
existing explicit semantic state and control rules are preserved.

### PLAT037 — useful contrast: when measurement really is the gate

PLAT037 deferred lazy physical execution-context materialization because the
remaining allocation cost after partial escape analysis had not been
established.

That is the correct use of a measurement gate: deciding whether an otherwise
valid representation optimization has a real surviving cost.

PLAT046 differs. The second execution thread, queue, stack reservation and
handoff are known structural machinery and the owner has already fixed that
unused second-thread machinery is invalid in the minimal architecture.
Measurement is still required afterward to quantify performance effect, not to
decide whether the unused mechanism belongs there.

### PLAT040 — the exact architectural analogy

PLAT040 selected a compact ordinary hot-call path with optional state
materialized only when semantically required. It explicitly says:

~~~text
Optional semantic capability must not force unrelated ordinary calls
to carry its full physical machinery.
~~~

It rejected a universal rich invocation carrier on the ordinary hot path.

This is the exact same architecture principle at a different boundary:
deep-stack, multi-caller or multi-Actor capability must not force every
ordinary one-Actor hosted call through a universal physical thread carrier.

### PLAT043 / PLAT044 — optimize known semantic case after ordinary selection

PLAT043 keeps `ifTrue` and related Boolean operations as ordinary messages but
allows their already-selected standard behavior to use a narrower physical
owner. PLAT044 similarly allows an eligible semantic callback activation to
exist without an additional physical callback Root/CallTarget.

These decisions are direct precedent for the project-owner analogy used during
PLAT046-A:

~~~text
general semantic possibility
  !=
mandatory generic physical machinery in the known ordinary case
~~~

### PLAT045 — Native capability difference stays at the platform boundary

PLAT045 currently makes Native Image guest execution interpreter-only because of
an upstream Bytecode DSL limitation. It does not turn that platform constraint
into a new Protos semantic identity or require a dedicated JVM execution
carrier.

PLAT046 must keep JVM/Native physical consequences separable if they diverge.

## Current product implementation

At the pinned product revision, `ProtosStandaloneHostedSession` constructs a
`ProtosGuestCarrier("protos-embedded-guest")` unconditionally during
`open()`.

Every guest-touching operation then executes through that carrier:

~~~text
open
prepareTopLevel
invokeTopLevel
PreparedTopLevel.invoke
close guest teardown
~~~

`ProtosGuestCarrier` owns:

~~~text
one dedicated platform thread
explicit 16 MiB requested stack
LinkedBlockingQueue
one request at a time
caller LockSupport.park()
carrier execution
LockSupport.unpark(caller)
~~~

The code explicitly states that the calling thread never executes guest code and
that this is done so guest recursion never depends on the ambient caller stack.

This is the exact physical mechanism under review.

## Existing runtime already implements the desired incremental pattern elsewhere

`ProtosPolyglotRuntimeHost.actorSchedulerForRuntime()` is more important than a
foreign precedent because it is current Protos code implementing the project's
own philosophy.

Its scheduler executor is created lazily. The source explicitly states that a
host which never schedules a child Actor pays no Actor-carrier thread cost.

Therefore current Protos already has this shape:

~~~text
RootActor only
  -> no child-Actor carrier pool

child Actor becomes schedulable
  -> lazily create/use shared RuntimeHost Actor carrier substrate
~~~

The mandatory standalone-session guest carrier is an exception to that
incremental rule, not the general Protos Actor architecture.

## RootTask execution already begins directly

`ProtosRootTaskExecution.runRootTask` contains the explicit invariant:

~~~text
The first segment runs directly;
only later runnable re-entries use the Actor queue.
~~~

It calls `domain.runFreshRootTaskDirectly(...)` before
`dispatchUntilTerminal(...)`.

Removing the outer standalone-session thread handoff therefore does not require
bypassing RootTask or Actor semantics. The runtime already distinguishes
"execute the current RootActor segment directly" from "schedule later runnable
work".

## Process Context already supports current-thread entry

`ProtosProcessExecutionHost` is implementation-only placement with no Protos
identity or authority.

`ProtosPolyglotProcessContext.callForRuntime(...)` delegates directly to
`ProtosPolyglotExecutionContext.callEntered(...)`.

`callEntered(...)`:

1. takes the Context lifecycle read lock;
2. enters the Polyglot Context on the current physical thread;
3. executes the action;
4. leaves the Context;
5. releases the lifecycle lock;
6. completes any deferred close.

The class explicitly says that executions enter/leave the Context on the
current carrier and that the lifecycle lock is not a guest-execution GIL.

`ProtosLanguage.isThreadAccessAllowed(...)` returns `true`.

No Protos semantic state is stored by carrier identity; semantic state remains
explicit in `ProtosActivation`.

Consequently there is no existing Truffle/Protos thread-affinity invariant that
requires the standalone session's private carrier.

## Exact benchmark topology

The cross-Truffle harness confirms that the compared repeated-invocation paths
do not have the same physical topology.

For GraalJS/GraalPy, `TruffleJvmRunner` builds one Context, resolves
`truffleRun` once, and the timed invocation is:

~~~text
benchmark runner thread
  -> Value.execute()
  -> guest
  -> return
~~~

For the prepared Protos path, `ProtosPreparedVariantRunner` resolves
`PreparedTopLevel` once and the same timed loop calls `prepared.invoke()`:

~~~text
benchmark runner thread
  -> ProtosGuestCarrier.call()
  -> queue
  -> park caller
  -> protos-embedded-guest
  -> Process Context enter
  -> RootActor task
  -> guest
  -> return
  -> unpark caller
~~~

The extra thread/handoff is therefore product architecture, not benchmark
driver machinery.

The benchmark remains useful as an end-to-end measurement of the current public
embedding surfaces. It is not currently a pure comparison of equivalent
single-thread guest execution shapes.

## Existing PERF025 carrier evidence

PERF025-E2 removed per-call state arrays, the queued completion wrapper and
monitor wait/notify while keeping the same carrier topology.

Retained E3B steady p50:

| workload | PRE-E2 ns/call | E2 ns/call | change |
| --- | ---: | ---: | ---: |
| primitive-return-literal | 6692.741 | 6173.223 | -7.76% |
| primitive-closure-call | 7522.591 | 7181.618 | -4.53% |
| primitive-method-call | 7013.635 | 6589.426 | -6.05% |

Correctness/admission passed 6/6.

This proves the old transport details were real overhead. It does not justify
the transport topology itself.

Further optimization of queue/park/unpark for the minimal path would optimize a
mechanism that the owner invariant says should not exist there.

## External Truffle / runtime survey

The purpose of this survey is not to copy another runtime. It is to identify
whether direct caller execution hides a known mandatory cost that Protos has
missed.

### Truffle Polyglot Context

Current GraalVM Truffle API documentation states:

- `Context.enter()` enters the Context on the **current thread**;
- a Context is safe from one thread, and may be used sequentially by multiple
  threads;
- simultaneous access depends on initialized-language support;
- repeated small operations may pay Context enter/leave overhead, and an
  embedder may explicitly enter once to amortize that overhead;
- `TruffleLanguage.initializeThread` exists to initialize per-thread language
  state when a Context is first accessed from a new thread;
- `initializeMultiThreading` is invoked only when actual simultaneous
  multi-thread access begins.

Relevant current documentation:

- https://www.graalvm.org/truffle/javadoc/org/graalvm/polyglot/Context
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleContext.html

The Truffle model therefore treats the calling thread as a normal execution
carrier and adds thread-specific/multithread machinery when another thread
actually participates.

### GraalJS

GraalJS documentation says one Context is accessed by one thread at a time.
The same Context may move sequentially between threads with synchronization,
while concurrent access is rejected for JavaScript.

There is no mandatory private execution thread interposed between
`Value.execute()` and ordinary JavaScript execution.

Reference:

- https://www.graalvm.org/javascript/docs/

### GraalPy

GraalPy is useful counter-evidence because it has a real global synchronization
requirement: it always maintains a GIL for compatibility with Python/C-extension
expectations.

Yet its documented `GilNode` implementation profiles the ordinary case so that
when execution is always single-threaded or already owns the GIL, it does not
emit the lock/unlock boundary call. If a second thread later appears, compiled
code may deopt and the stronger path becomes active.

That is a direct pay-as-you-grow precedent: preserve the general requirement,
but do not impose its full physical path on the proven simple case.

References:

- https://github.com/oracle/graalpython/blob/master/docs/contributor/IMPLEMENTATION_DETAILS.md
- https://github.com/oracle/graalpython/blob/master/docs/user/Interoperability.md

GraalPy also documents a genuine stronger physical requirement: Python native
extensions are incompatible with Java virtual threads because native
thread-local state expects platform-thread behavior. Applications that may use
such extensions should dispatch those calls to a platform-thread executor.

That is the correct pattern for PLAT046: a concrete stronger capability may
justify a stronger physical carrier without redefining the minimal path.

Reference:

- https://github.com/oracle/graalpython/blob/master/docs/user/Embedding-Native-Extensions.md

### Apple Pkl

Current `apple/pkl` `EvaluatorImpl` executes normal evaluation by entering
the Polyglot Context on the caller, evaluating, and leaving it.

Pkl does create a separate scheduled executor when an evaluation timeout is
configured. If timeout is absent, that executor is absent.

This is especially relevant prior art:

~~~text
ordinary evaluation
  -> caller thread

optional timeout capability
  -> additional timeout scheduler thread
~~~

Source:

- https://github.com/apple/pkl/blob/main/pkl-core/src/main/java/org/pkl/core/EvaluatorImpl.java

### TruffleRuby

TruffleRuby supports real Ruby threads and runs them in parallel. It also
supports a single-threaded mode; features such as timeout/signal handling are
restricted there because those capabilities genuinely require additional
threads.

This is not evidence that every embedded call needs an invisible private
carrier. It is evidence that thread-requiring features should own the threads
they actually require.

References:

- https://github.com/truffleruby/truffleruby/blob/master/doc/user/polyglot.md
- https://github.com/truffleruby/truffleruby/blob/master/doc/user/compatibility.md

### Espresso

Espresso is deliberately a multithreaded guest Java implementation. Its
Truffle language allows access from any host thread and initializes guest
thread state for the actual host thread that enters execution.

GraalVM documentation also exposes a `java.MultiThreaded=false` mode; features
such as finalizers/reference notification that require extra threads are not
run in that mode.

This is useful contrast: when the guest language actually has thread semantics,
the runtime creates/maintains thread machinery for those semantic threads. It
does not establish a rule that every synchronous host-to-guest call needs an
additional invisible execution thread.

References:

- https://www.graalvm.org/dev/reference-manual/espresso/interoperability/
- `oracle/graal:espresso/.../EspressoLanguage.java`

### Sulong / LLVM

Sulong permits thread access and maintains per-thread LLVM state/resources for
the actual host threads that execute guest code. Thread-local resources are
initialized/disposed for participating threads.

This demonstrates a hidden cost that direct caller execution must account for:
the first use of a new physical caller thread may require thread-local runtime
initialization. It does not imply a mandatory second execution thread for every
call.

Source:

- `oracle/graal:sulong/.../LLVMLanguage.java`

### SimpleLanguage

Current SimpleLanguage uses the Truffle Bytecode DSL by default and relies on
ordinary CallTarget/Context execution. Its implementation invalidates
single-context assumptions only when multiple contexts actually occur.

SimpleLanguage is didactic rather than production authority, but it reinforces
the Truffle pattern: optimize the smaller established state and widen machinery
when the stronger state appears.

Source:

- `oracle/graal:truffle/.../SLLanguage.java`

### Truffle background/compiler threads

GraalVM may run Truffle compilation asynchronously in compiler threads.
Garbage collection and other JVM services may also use host threads.

These threads must be separated conceptually from the PLAT046 question.

PLAT046 does **not** claim:

~~~text
a Protos process has only one operating-system/JVM thread
~~~

It claims only that an ordinary synchronous guest invocation should not require
an additional **guest execution carrier** when no execution capability needs
one.

Current Truffle options explicitly describe
`engine.BackgroundCompilation` and `engine.CompilerThreads` as compiler
infrastructure:

- https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Options/

## Hidden costs of direct caller execution

The external survey does reveal real costs. None justifies the current universal
private carrier.

### 1. Context enter/leave

This cost is real, but it is already paid today inside the dedicated carrier:
`invokeResolved -> ProcessExecutionHost -> callEntered -> Context.enter/leave`.

Moving the guest to the caller does not introduce this requirement.

A later PERF investigation may decide whether repeated prepared calls can
amortize enter/leave, but that is not required for PLAT046.

### 2. Reusable-session serialization

The current single carrier serializes all session guest operations as a side
effect.

Direct execution must preserve the same semantic safety explicitly.

The smallest candidate is a **session execution gate** around guest-touching
operations. It may be an implementation lock/monitor or equivalent mechanism;
PLAT046 need not standardize the Java primitive.

For the common single-caller case it is uncontended local synchronization, not
a cross-thread handoff.

Important: `ProtosPolyglotExecutionContext`'s lifecycle read lock is not this
gate. That lock intentionally permits concurrent executions in the same Process
Context. RootActor/session serialization is a distinct responsibility.

### 3. call-vs-close lifecycle

The current carrier orders close behind prior calls.

A direct implementation must preserve:

~~~text
no guest invocation after close cutover
all established Process termination steps attempted
Context terminal disposition observed
RuntimeHost closed after Process Context termination
resolver cleanup retained
idempotent close
suppressed-failure ordering retained
~~~

This can compose with the same session execution gate plus the existing
`closeStarted` and Context lifecycle machinery. It does not require a thread.

### 4. Multiple external callers

If different external threads invoke one reusable session sequentially, Truffle
may perform first-use per-thread initialization.

That is a legitimate cost of actually using multiple caller threads.

It is exactly pay-as-you-grow and should not be shifted onto the single-caller
case in advance.

The current public multi-caller guarantee is serialization, not concurrent
RootActor execution. The direct architecture must preserve that contract.

### 5. Child Actors

When guest code actually creates/schedules additional Actors, current Protos
already lazily creates the shared RuntimeHost Actor carrier substrate.

No PLAT046-specific preallocation is needed for the one-Actor case.

### 6. Host-blocking or capability-specific work

Some operations may legitimately need another thread or execution lane.

Current Protos already has this pattern: standalone stdin uses a virtual worker
only when a host read operation is actually issued.

Pkl timeout scheduling and GraalPy native-extension platform-thread constraints
provide external examples of the same principle.

Such capability-specific lanes are compatible with direct ordinary execution.

### 7. JIT, GC and runtime service threads

They remain possible and are not guest execution carriers. Their existence does
not justify a per-session `protos-embedded-guest` thread.

### 8. Stack capacity

Caller stack capacity becomes relevant only when a concrete execution shape
needs more stack than the selected physical caller provides.

BUG008 proved that an earlier runtime topology plus the retained deep-recursion
driver required more host stack than the then-ambient path supplied. Later
PLAT042/043/044/PERF026 removed several artificial per-level roots, and
PERF025-C2D reduced the dedicated requested stack from 64 MiB to 16 MiB.

None of that establishes that an ordinary `return 1` invocation needs another
thread.

The retained depth-10,000 requirement remains. If a future/direct execution
slice exposes a real stack-capacity failure on a supported caller surface, that
becomes a concrete stronger-capability problem and must be solved without
silently reinstalling universal second-thread machinery on the minimal path.

~~~text
DEEP_RECURSION_10000_REQUIREMENT=RETAINED
STACK_GATE_BLOCKS_MINIMAL_ARCHITECTURE_SELECTION=NO
UNIVERSAL_STRONG_STACK_CARRIER_PREAUTHORIZED=NO
~~~

## Candidate set after exhaustive research

### Candidate A — status quo: mandatory dedicated carrier per hosted session

~~~text
all guest-touching session operations
  -> dedicated 16 MiB platform thread
~~~

Advantages:

- known correctness;
- existing deep-stack guarantee;
- multi-caller serialization and close ordering fall out of one queue;
- mature evidence.

Disqualifying problem:

- contradicts the recorded owner invariant;
- contradicts pay-only-for-use and lower-layer protection;
- one reusable one-Actor session allocates a physical execution thread and stack
  even when no concurrency or exceptional stack capability is used;
- every invocation pays cross-thread handoff.

Candidate A remains historical/current-state evidence, not a selectable minimal
architecture under the recorded invariant.

### Candidate B — direct caller is the ordinary physical carrier; explicit local session gate

~~~text
single ordinary call
  caller
    -> session execution gate
    -> Process execution host
    -> Context.enter
    -> RootActor RootTask
    -> guest
    -> Context.leave
    -> return

additional Actor work
  -> existing lazy RuntimeHost Actor scheduler

other stronger physical capability
  -> add only when concrete requirement proves it necessary
~~~

The gate preserves the current reusable-session serialization contract without
creating a second thread.

This candidate does not select a deep-stack lane, pool or heuristic in advance.
It preserves the architectural seam for a future stronger mechanism if a
concrete requirement appears.

This is the **recommended proposal**, pending exact owner selection.

### Candidate C — eliminate host-stack-proportional guest recursion first

Use explicit guest frames, iterative/trampolined dispatch or another execution
representation so deep guest recursion no longer relies proportionally on host
stack, then remove the carrier.

This may be a valuable future architecture, but it is not the smallest solution
to the present one-Actor problem.

It would reopen major PE/JIT, debugger, stack-trace, control-transfer,
continuation, Native interpreter and allocation surfaces to solve a physical
problem that is not established for the ordinary path.

Candidate C is not recommended for PLAT046's immediate objective.

### Candidate D — begin direct, heuristically migrate/fallback before stack exhaustion

This candidate attempts to keep the normal path direct and transfer execution
to a stronger stack only when remaining host stack becomes dangerous.

It is rejected:

- Java exposes no portable precise remaining-stack budget suitable for this;
- safe transfer of a live Java/Truffle call stack is not an ordinary operation;
- catching `StackOverflowError` and replaying is not a semantics-preserving
  general fallback;
- migration would create additional continuation/debugger/control complexity.

This is speculative machinery with poor evidence.

### Candidate E — shared RuntimeHost-owned strong carrier substrate for all hosted RootActor calls

Replace one private carrier per session with a bounded RuntimeHost shared pool.

This improves resource scaling relative to A:

~~~text
session count != thread count
~~~

but still forces the minimal synchronous invocation to queue/handoff to another
execution thread.

It therefore fails the recorded owner invariant as a universal base path.

A shared strong-stack lane may remain a future capability-specific mechanism if
a concrete stronger case later needs one, but it is not the minimal execution
architecture.

### Candidate F — defer carrier removal until the depth-10,000 caller-stack test passes

This was an earlier investigation direction and is now rejected.

It makes a speculative stronger resource requirement a prerequisite for fixing
known unused machinery in the weak path. That contradicts the owner correction,
pay-as-you-grow and GITHUB010's speculation burden of proof.

The depth regression remains required; it simply no longer defines the base
thread topology.

## GITHUB010 comparative scoring

Scores are 1–5 comparison aids, not selection arithmetic. Confidence is shown
as H/M/L. Hard invariant violations cannot be compensated by higher totals.

| Dimension | A mandatory private carrier | B direct caller + gate | C explicit guest-stack/trampoline | D heuristic fallback | E shared mandatory carrier |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness / invariants | 5 H — proven current behavior | 4 M — straightforward but serialization/close must be implemented exactly | 3 M — broad control/debugger/JIT surface | 1 L — safe migration/replay not established | 4 M — serialization feasible |
| 2 Protos alignment | 1 H — unused machinery infects minimal path | 5 H — directly matches pay-only/pay-grow | 3 M — principled but overbuilt now | 2 H — heuristic special machinery | 1 H — still mandatory second execution thread |
| 3 Present need | 1 H — no present one-Actor need for second thread | 5 H — solves exact active mismatch | 1 H — host-stack elimination not presently required for trivial path | 1 H — no present need | 2 M — solves per-session thread count, not invocation topology |
| 4 Incremental growth | 1 H — strongest machinery paid immediately | 5 H — stronger machinery added when used | 2 H — large up-front machinery | 3 M — intended incremental but complex | 2 H — pool still paid before capability needs it |
| 5 Future options | 3 M — known but constraining | 5 H — keeps actor/strong-lane/trampoline options open | 4 M — powerful but commits representation | 2 L — difficult control model | 4 M — can evolve, but establishes handoff as base |
| 6 Scalability | 1 H — thread/stack per session | 5 H — no per-session execution thread; existing shared actor pool composes | 5 M — could scale well after major work | 3 M | 4 M — bounded shared pool |
| 7 Simplicity | 3 H — simple because already built, but extra topology | 4 H — one explicit serialization responsibility | 1 H — major execution-engine complexity | 1 H | 3 M |
| 8 Portability | 3 M — requested stack size/platform details matter | 5 M — standard Truffle current-thread model; caller constraints remain explicit | 4 M — could reduce host dependence but backend work large | 1 H — remaining-stack heuristic nonportable | 3 M |
| 9 Runtime/resource cost | 1 H — queue/park/thread/stack every session | 5 H — no extra carrier in minimal case | 4 M — potential runtime overhead depends representation | 4 M on fast path, but fallback state exists | 2 H — queue/handoff remains |
| 10 Operability | 4 H — mature and observable | 4 M — simpler topology; multi-caller/close gate must be diagnosed clearly | 2 M — harder stack/debugger model | 1 H — difficult failure diagnosis | 3 M |
| 11 Reversibility / deferral | 3 H — removable but entangled with stack guarantee | 5 H — future stronger lane can be added behind same non-semantic boundary | 2 M — large rewrite to undo | 2 M | 4 M |
| 12 Evidence maturity / risk | 5 H — current product evidence | 4 H — Truffle API + peers + current Protos direct-first/lazy-Actor evidence; product cutover not yet implemented | 2 M — plausible VM technique, no Protos proof | 1 H — weak/unsafe precedent | 3 M — ordinary pool design is known, but wrong base invariant |

Hard-gate result:

~~~text
A=DISQUALIFIED_BY_OWNER_INVARIANT_FOR_MINIMAL_PATH
B=SURVIVES
C=SURVIVES_AS_FUTURE_ARCHITECTURE_BUT_FAILS_PRESENT_NECESSITY_GATE
D=REJECTED
E=DISQUALIFIED_AS_UNIVERSAL_BASE_PATH
F_DEFER=REJECTED
~~~

## Mandatory adversarial incremental-design questions

### What is the smallest solution satisfying requirements known today?

For the reusable hosted one-RootActor path:

~~~text
existing caller thread
+ existing Process/Context/RootTask semantics
+ narrow session serialization/lifecycle gate
~~~

No new scheduler, pool, dedicated stack thread, continuation representation or
guest-stack VM is required to express the known ordinary case.

### What concrete evidence justifies every capability beyond it?

- child Actor carriers: justified only when child Actor work becomes schedulable;
  current RuntimeHost already provisions them lazily;
- blocking host-I/O helper: justified when that I/O operation is invoked;
- timeout/cancellation helper: only if a future API requires an independent
  timing/cancellation agent;
- special platform-thread/native lane: only for a concrete native/platform
  restriction;
- deep-stack mechanism: only if a supported direct-caller execution shape
  demonstrates that requirement.

No evidence justifies any of those as mandatory machinery for `return 1`.

### If the stronger capability is omitted now, can it be added later without breaking the model?

Yes.

PLAT001/010/011 already separate semantic Actor/Process identity from physical
carrier placement. `ProtosProcessExecutionHost` is an implementation placement
boundary. The Actor scheduler is already lazy and shared.

A future capability-specific lane can therefore be added behind existing
non-semantic runtime boundaries without changing Protos object/Actor/Task
semantics.

### What would need rewriting later?

If a concrete strong-stack requirement appears, the runtime may need:

- an explicit embedding/runtime capability or execution surface selecting that
  lane;
- a RuntimeHost-owned strong-stack worker/lane; or
- a deeper guest-recursion representation change.

Those are bounded implementation/platform changes. They do not require changing
Actor identity, Process identity, Task semantics, message semantics or public
language syntax merely because the base path executes on the caller today.

### What current complexity would make us regret pre-building the future case?

The current product demonstrates it directly:

- dedicated platform thread per session;
- requested 16 MiB stack per session;
- queue lifecycle;
- park/unpark;
- cross-thread publication;
- carrier shutdown protocol;
- every tiny prepared call crosses that boundary;
- benchmark topology diverges from peer languages before guest work begins.

Building more shared-carrier/stack-selection machinery now without a concrete
need would repeat the same mistake at a different abstraction level.

## Future-scenario stress test

### Many reusable sessions

Candidate B scales without one execution thread/stack per session. Candidate A
does not. Candidate E bounds thread count but retains invocation handoff.

### Several external callers to one session

Candidate B retains the existing serialized session contract through the local
execution gate. Actual new caller threads may pay Truffle per-thread
initialization only when they participate.

### Multiple Actors

Candidate B composes with the already-ratified PLAT010/011 RuntimeHost-owned
lazy Actor carrier substrate. RootActor physical placement remains non-semantic.

### Blocking I/O

Capability-specific offload remains permitted. It is not a reason to move all
ordinary guest computation to another thread.

### Very deep recursion

The requirement remains. If direct execution reveals a real supported-surface
failure, solve that exact stronger physical requirement. Do not retroactively
make every ordinary call pay the solution.

### Native Image

PLAT045 remains authoritative. Native/JVM may require different physical
implementation details, but the platform difference remains below Protos
semantics. PLAT046 does not pre-authorize a JVM-only assumption as a language
rule.

### Future Truffle/Bytecode DSL changes

Candidate B depends only on the ordinary Truffle property that guest execution
may occur on an allowed current thread and semantic state is explicit. It does
not depend on a private undocumented scheduler trick.

If Truffle later provides better guest-stack/continuation machinery, Candidate C
can be revisited without changing the base semantic model.

### Alternative backend away from Truffle

Direct synchronous execution of the current call on the current execution
thread is a weak architectural assumption compared with a mandatory dedicated
Truffle-specific carrier. Candidate B therefore reduces rather than increases
backend coupling.

## Failure modes and implementation acceptance risks for Candidate B

An implementation must be rejected if it:

1. permits two segments of the same RootActor/session to execute concurrently;
2. weakens existing multi-caller serialization;
3. races invocation with session close;
4. changes Process/RootActor/Task identity or terminal mapping;
5. bypasses `ProcessExecutionHost` / Process Context entry;
6. makes physical caller identity observable to Protos;
7. stores semantic activation/Actor/Task state in host ThreadLocal state;
8. eagerly initializes the child-Actor carrier pool in the one-Actor case;
9. silently weakens the retained depth-10,000 conformance requirement;
10. introduces stack-size heuristics or `StackOverflowError` replay as an
    unapproved fallback;
11. changes benchmark workloads or methodology to manufacture an improvement.

## Strongest argument against Candidate B

The current dedicated carrier does two useful things at once:

1. serializes reusable-session operations; and
2. supplies a runtime-owned predictable requested stack budget independent of
   whichever host thread calls the API.

Moving to the caller means the implementation must make serialization explicit,
and supported caller threads may have different physical stack capacities.

That is a real engineering cost.

It does **not** overturn the recommendation because neither responsibility
requires that every minimal invocation execute on another thread:

- serialization can be local synchronization;
- a concrete stack requirement can own a stronger mechanism if and when it is
  demonstrated for a supported path.

The stronger case remains an escape path, not a base-path tax.

## Recommendation

~~~text
PLAT046_RECOMMENDED_PROPOSAL=B

B_NAME=
  DIRECT_CALLER_ORDINARY_EXECUTION
  + EXPLICIT_SESSION_SERIALIZATION_GATE
  + LAZY_EXISTING_ACTOR_CARRIER_SUBSTRATE_ONLY_WHEN_ACTOR_WORK_REQUIRES_IT
  + NO_PREBUILT_STRONG_STACK_MECHANISM_WITHOUT_CONCRETE_REQUIREMENT

RECOMMENDATION_CONFIDENCE=HIGH_ON_ARCHITECTURAL_DIRECTION
EXACT_IMPLEMENTATION_PRIMITIVE=DEFERRED_TO_IMPLEMENTATION_IF_OWNER_SELECTS_B

OWNER_INVARIANT=
  MINIMAL_ONE_CALLER_ONE_PROCESS_ONE_ROOTACTOR_SYNCHRONOUS_EXECUTION
  MUST_NOT_REQUIRE_A_SECOND_GUEST_EXECUTION_THREAD_MERELY_FOR_UNUSED_CAPABILITY

THREAD_CONFINEMENT_REQUIRED_BY=NONE_FOUND
ROOTACTOR_SERIALIZATION_REQUIRED=YES
SESSION_MULTI_CALLER_SERIALIZATION_REQUIRED=YES
CONTEXT_ENTER_LEAVE_REQUIRED=YES
DEDICATED_EXECUTION_THREAD_REQUIRED_BY=NONE_FOR_MINIMAL_PATH
STACK_10000_REQUIREMENT=RETAINED_ORTHOGONAL_REQUIREMENT
STACK_GATE_BLOCKS_BASE_SELECTION=NO

CURRENT_BENCHMARK_SURFACES_PHYSICALLY_EQUIVALENT=NO
REMEASURE_AFTER_IMPLEMENTATION=YES
NEW_BENCHMARK_METHODOLOGY_REQUIRED=NO

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
BUG008_REOPEN_REQUIRED=NO
PLAT001_DELTA=NONE
PLAT010_DELTA=NONE
PLAT011_DELTA=NONE
PLAT014_016_DELTA=NONE
PLAT019_021_027_028_DELTA=NONE
PLAT040_DELTA=NONE
PLAT043_044_DELTA=NONE
PLAT045_DELTA=NONE

DECISION_STATUS=PROPOSED_AWAITING_EXACT_OWNER_SELECTION
IMPLEMENTATION_AUTHORIZED=NO
~~~

## Proposed next sequence

If the project owner explicitly selects Candidate B after reviewing this packet:

1. ratify PLAT046 with a GITHUB021 invariant/delta check;
2. allocate/activate a bounded PERF025 implementation slice in
   `guillermomolina/protos`;
3. implement direct caller execution while preserving session serialization,
   Context entry, RootTask/Actor semantics and close ordering;
4. run focused and full correctness;
5. re-run the existing PERF025 primitive benchmark radar unchanged;
6. only then interpret the new cross-language ratios;
7. if a concrete deep-stack or multi-thread case fails, open the smallest
   capability-specific follow-up required by that evidence.

No command or experiment is part of this PLAT046-A investigation.

## Primary Protos sources reviewed

Philosophy/governance:

- `AGENTS.md`
- `AGENTS.work/DESIGN.md`
- `AGENTS.work/REFERENCE.md`
- `spec/AGENTS.md`
- `docs/design/PROTOS_DESIGN_PHILOSOPHY.md`

Normative/design model:

- `spec/io/PROCESS_IO.md`
- `spec/concurrency/ACTORS.md`
- `spec/concurrency/FUTURES_AND_TASKS.md`
- `spec/concurrency/PARALLEL_EXECUTION.md`
- `spec/concurrency/DISTRIBUTED_RUNTIME.md`
- `spec/runtime/ABSTRACT_RUNTIME.md`
- `docs/design/CONCURRENCY_DESIGN.md`

Ratified platform decisions:

- `PLAT001_TRUFFLE_RUNTIME_HOSTING.md`
- `PLAT010_ACTOR_PLATFORM_CARRIERS.md`
- `PLAT011_RUNTIMEHOST_CARRIER_SUBSTRATE.md`
- `PLAT014_TRUFFLE_COOPERATIVE_CONTINUATIONS.md`
- `PLAT016_BYTECODE_CLOSURE_DEFAULT_EXECUTION_TOPOLOGY.md`
- `PLAT019_NATIVE_SEMANTIC_SUSPENSION_BRIDGE.md`
- `PLAT021_BYTECODE_DYNAMIC_CONTROL_UNWIND.md`
- `PLAT027_TASK_TERMINAL_LIFECYCLE_BOUNDARY.md`
- `PLAT028_C_PRIME_CALLBACK_ORCHESTRATION_NATIVE_LEAF_BOUNDARY.md`
- `PLAT035_SINGLE_EXECUTION_BACKEND_LEGACY_AST_RETIREMENT.md`
- `PLAT037_LAZY_EXECUTION_CONTEXT_PHYSICAL_MATERIALIZATION.md`
- `PLAT038_NATIVE_IMAGE_BOOTSTRAP_RUNTIME_RELEASE_BOUNDARY.md`
- `PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY.md`
- `PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md`
- `PLAT041_BYTECODE_ROOT_TAG_MATERIALIZED_LOCAL_GROUPING_BOUNDARY.md`
- `PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md`
- `PLAT043_STANDARD_BOOLEAN_CONTROL_INTERPRETER_OWNERSHIP_BOUNDARY.md`
- `PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md`
- `PLAT045_NATIVE_IMAGE_GUEST_JIT_CAPABILITY_BOUNDARY.md`

Current product implementation:

- `src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotExecutionContext.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotRuntimeHost.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosRootTaskExecution.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosProcessExecutionHost.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorScheduler.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`

Benchmark authority:

- `guillermomolina/protos-benchmarks:BENCHMARKING.md`
- `truffle/src/main/java/com/guillermomolina/protos/benchmarks/truffle/TruffleJvmRunner.java`
- `truffle/src/main/java/com/guillermomolina/protos/benchmarks/truffle/ProtosJvmVariantRunner.java`
- `truffle/src/prepared/java/com/guillermomolina/protos/benchmarks/truffle/ProtosPreparedVariantRunner.java`
- retained PERF025-E3B evidence.

## External sources reviewed

Current/recent public documentation/source was checked on 2026-10-02:

- GraalVM Polyglot `Context` API:
  https://www.graalvm.org/truffle/javadoc/org/graalvm/polyglot/Context
- Truffle `TruffleLanguage` thread lifecycle API:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleLanguage.html
- Truffle `TruffleContext` API:
  https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/TruffleContext.html
- GraalJS threading:
  https://www.graalvm.org/javascript/docs/
- GraalPy implementation details / GIL:
  https://github.com/oracle/graalpython/blob/master/docs/contributor/IMPLEMENTATION_DETAILS.md
- GraalPy interoperability:
  https://github.com/oracle/graalpython/blob/master/docs/user/Interoperability.md
- GraalPy native-extension embedding constraints:
  https://github.com/oracle/graalpython/blob/master/docs/user/Embedding-Native-Extensions.md
- Apple Pkl `EvaluatorImpl`:
  https://github.com/apple/pkl/blob/main/pkl-core/src/main/java/org/pkl/core/EvaluatorImpl.java
- TruffleRuby polyglot/threading:
  https://github.com/truffleruby/truffleruby/blob/master/doc/user/polyglot.md
- TruffleRuby compatibility/thread behavior:
  https://github.com/truffleruby/truffleruby/blob/master/doc/user/compatibility.md
- Espresso interoperability/multithreading:
  https://www.graalvm.org/dev/reference-manual/espresso/interoperability/
- GraalVM Truffle options/background compilation:
  https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Options/
- Oracle Graal sources for SimpleLanguage, Espresso and Sulong/LLVM.

## Cross references

- `guillermomolina/protos#778` — PLAT046.
- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#681` — BUG008, historical closed.
- `docs/project/evidence/PLAT046/PLAT046_INTAKE_AND_TRIGGER_EVIDENCE.md`.
- `docs/project/evidence/PERF025/PERF025_E3B_CARRIER_TRANSPORT_AB_REFERENCE.md`.
