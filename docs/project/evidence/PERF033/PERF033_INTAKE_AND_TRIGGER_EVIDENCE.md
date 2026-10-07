# PERF033 — callable interop pay-as-you-grow intake and trigger evidence

Date: 2026-10-07

## Work identity

~~~text
IDENTIFIER=PERF033
ISSUE=guillermomolina/protos#832
TITLE=Make ordinary Protos callable interop fully pay-as-you-grow

TRIGGERED_BY=PERF032/guillermomolina/protos#831
RELATED_DECISION=PLAT046/guillermomolina/protos#778
RELATED_ARCHITECTURE=PLAT040
RELATED_INTEROP_SEMANTICS=D189

IMPLEMENTATION_REPOSITORY=guillermomolina/protos
INITIAL_STATUS=READY
PRIORITY=INTENTIONALLY_UNSET
~~~

This record is durable non-normative project evidence. Live lifecycle,
scheduling and priority remain owned by GitHub.

## Trigger

PERF032 established that the trivial prepared `primitive-return-literal`
workload spends the dominant part of its measured cost before the first
compiled guest CallTarget.

Reconciliation with Protos pay-as-you-grow architecture then established that
the work cannot stop at optimizing or deleting RootTask alone. The complete
minimum host/foreign callable path must remove or justify every Protos-owned
fixed layer not required by the active program or standard Truffle
host-to-guest execution.

## Governing reference

The minimum callable reference is the canonical GraalJS/GraalPy Polyglot shape:

~~~text
Value.execute()
  -> framework HostToGuestRootNode
  -> InteropLibrary.execute
  -> language callable
  -> guest CallTarget
  -> result
~~~

Protos target:

~~~text
Value.execute()
  -> framework HostToGuestRootNode
  -> InteropLibrary.execute(Protos callable)
  -> compact ordinary Closure invocation
  -> cached/direct guest call
  -> result
~~~

This does not privilege peer-language semantics over Protos. It treats their
mature Truffle boundary as the implementation reference while Protos normative
semantics remain authoritative.

## Scope guard

PERF033 owns complete structural convergence of the minimum ordinary callable
path. It is not complete merely because one large current owner disappears.

For the weaker ordinary callable case, stronger machinery is paid only when its
capability becomes active. Candidate universal costs include RootTask/Task,
Actor task bookkeeping, cancellation start bookkeeping, scheduler/terminal
publication, rich activation, Protos-owned explicit Context entry envelope,
unobserved host ThreadLocal state, Process execution-host routing, outcome
wrapping, stronger reusable-session serialization, and generic indirect dispatch
where a stable direct call is established.

The Issue must preserve exact stronger semantics when those capabilities are
actually used.

## Existing implementation assets

PERF033 is expected to reuse rather than duplicate:

- PLAT040 compact invocation and conditional materialization;
- PERF025 H1 compact source Closure entry;
- existing explicit non-Task synchronous Closure execution;
- current Task-owned compact execution for the actual Task case;
- public Truffle `InteropLibrary` executable-value mechanisms;
- D189 ordinary synchronous callback semantics.

No private/internal Graal API dependency is authorized.

## Completion rule

~~~text
ROOT_TASK_REMOVED_ALONE=INSUFFICIENT

CLOSE_WHEN=
  CANONICAL_VALUE_EXECUTE_PATH_EXISTS
  AND MINIMUM_CALL_HAS_NO_UNUSED_STRONGER_CAPABILITY_MACHINERY
  AND EVERY_REMAINING_PROTOS_SPECIFIC_FIXED_LAYER_IS_JUSTIFIED
  AND STRONGER_SEMANTICS_REMAIN_EXACT
  AND SAME_WORKLOAD_CORRECTNESS_AND_COMPATIBLE_TIMING_ARE_RETAINED
~~~

No predetermined nanosecond target is a closure condition.

## Initial next work

~~~text
NEXT_SLICE=PERF033-A
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos

GOAL=
  establish the canonical executable Protos callable boundary
  and route ordinary minimal host callable execution through
  the existing compact non-Task Closure machinery
~~~

Adjacent implementation steps should be grouped when they touch only a small
cohesive set of files and preserve one valid intermediate candidate.

## AI-assistance disclosure

This intake evidence was materially prepared with AI assistance from ChatGPT
using the newly allocated live Issue, current product state, the approved
PLAT046 amendment, PERF032 findings, PLAT040, PERF025 H1 and D189.
