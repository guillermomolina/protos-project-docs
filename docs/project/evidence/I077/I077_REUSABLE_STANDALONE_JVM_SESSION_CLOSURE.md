# I077 — reusable standalone JVM embedding session closure evidence

Date: 2026-10-01

## Scope

This record preserves the published implementation evidence for I077, the
reusable standalone JVM embedding session required by PERF021.

Operational Issue state remains authoritative in `guillermomolina/protos`.

## Published implementation

I077 was published in Protos revision:

```text
f0791896c3c022a747b4d47af629227da58acfab
I077: expose reusable standalone JVM embedding session
```

The implementation advances the development version to
`0.3.128-SNAPSHOT`.

The supported reusable embedding surface is:

```java
ProtosStandaloneHostedSession.open(...)
ProtosStandaloneHostedSession.initialOutcome()
ProtosStandaloneHostedSession.invokeTopLevel(String name)
ProtosStandaloneHostedSession.close()
```

Opening a session performs the I076 standalone sequence once: Core bootstrap,
standalone Process bootstrap, Process-scoped Polyglot Context binding and
canonical direct-file execution. Unlike the one-shot I076 entry point, the
session retains the semantic Process, Polyglot Context, Engine/JIT state and
runtime host until close.

## Repeated top-level invocation

`invokeTopLevel(name)` reads the named entry-module slot and requires a
source-backed `ProtosClosureValue`. It then executes that retained Closure via
`ProtosRootTaskExecution.executeClosure(...)` inside the same live Process.

The repeated path therefore does not parse textual `run()` source for each
iteration. The slot is intentionally read again on every call so normal guest
reassignment remains observable while the execution runtime identity remains
stable.

This supplies the JVM warmup/steady-state authority PERF021 was missing after
I076: repeated benchmark invocations can accumulate profiling and JIT state in
one Process-scoped Polyglot runtime.

## Guest carrier and lifecycle

I077 extracts the dedicated guest-thread authority into `ProtosGuestCarrier`.
The carrier is long-lived for the session and uses the existing BUG008 stack
budget:

```text
64 MiB
```

Session open, repeated guest invocation and close are serialized on that
carrier, so guest recursion does not depend on an external caller thread's
ambient stack.

Session close is idempotent and attempts the established lifecycle in order:

```text
request Process termination
await Process termination
await Process-scoped Context terminal disposition
close runtime host
close direct-file module resolver
stop/join guest carrier
```

Cleanup failures are retained rather than silently abandoning later cleanup
steps.

## One-shot compatibility

`ProtosStandaloneHostedExecution.executeFile(...)` remains supported. The
published implementation now delegates its one-shot behavior through the
reusable session authority and returns `initialOutcome()` before closing the
session.

The changelog records no observable Protos semantic change, no CLI behavior
change and no benchmark policy added to Protos.

## Acceptance coverage in the published revision

Revision `f0791896c3c022a747b4d47af629227da58acfab` adds
`ProtosStandaloneHostedSessionEmbeddingTest` covering:

- recursive Fibonacci: initial and repeated `run` results `832040`;
- recursive factorial: initial and repeated `run` results
  `2432902008176640000`;
- mutable entry-module state preserved across repeated calls, proving reuse of
  one live session rather than fresh `executeFile(...)` calls;
- rejection of absent/non-Closure top-level slots; and
- closed-session rejection plus idempotent close.

It also adds `ProtosStandaloneHostedSessionLifecycleTest`, which checks that the
Process and Process-scoped Context remain live during session use and reach
clean terminal state on close.

This durable record describes the published code and test coverage. No tests
were independently rerun as part of this documentation publication step.

## PERF021 routing

I077 is the direct blocker introduced when PERF021 reached the JVM
cold/warmup/steady-state stage.

With the reusable session now published, PERF021 can resume in
`guillermomolina/protos-benchmarks` and replace its current one-shot Protos JVM
execution with a session that prepares the workload once and invokes top-level
`run` repeatedly in the same live runtime.

The existing correctness checkpoint remains:

```text
fibonacci: Protos = GraalJS = GraalPy = 832040
factorial: Protos = GraalJS = GraalPy = 2432902008176640000
```

No Protos release is required for this development workflow; the locally built
and installed `0.3.128-SNAPSHOT` artifact is sufficient.

## Coordination closure

I077 is:

```text
guillermomolina/protos#751
implementation: f0791896c3c022a747b4d47af629227da58acfab
semantic change: no
benchmark policy change: no
```

PERF021 remains:

```text
guillermomolina/protos#748
next repository: guillermomolina/protos-benchmarks
next stage: JVM repeated-invocation timing machinery
```

## References

- `guillermomolina/protos#751` — I077
- `guillermomolina/protos#748` — PERF021
- Protos revision `f0791896c3c022a747b4d47af629227da58acfab`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java`
- `src/test/java/com/guillermomolina/protos/embedding/ProtosStandaloneHostedSessionEmbeddingTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSessionLifecycleTest.java`
