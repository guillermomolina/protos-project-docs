# I076 — JVM standalone embedding gap evidence

Date: 2026-09-30

## Scope

This record preserves the evidence that caused PERF021 to stop its Protos JVM
common-runner integration and allocate I076.

It is non-normative project evidence. It does not define Protos language
semantics or benchmark policy.

## Trigger

PERF021 requires Protos, GraalJS, and GraalPy to participate in the same
controlled JVM/Truffle comparison plane.

The PERF021 Java runner successfully discovered the registered `protos`
language through the normal Polyglot class path. A direct
`Context.eval(Source)` of the Protos recursive Fibonacci workload then failed
inside Protos with:

```text
java.lang.IllegalStateException:
Execution node requires a Protos activation or compact source-call ABI
```

The failure reached `ProtosFrameArguments.activation()` from
`ProtosSemanticBytecodeRootNode$InvokeSemanticHelper.perform()`.

This distinguishes the failure from language discovery or class-path setup:
Polyglot entered Protos guest execution, but the entry did not carry the
standalone semantic activation required by the current execution architecture.

## Repository evidence

At the Protos HEAD inspected when I076 was allocated, `ProtosLanguage.parse()`
compiles a top-level source directly through the bytecode source compiler.

The successful CLI standalone path establishes additional runtime state before
guest execution. Its path includes:

```text
ProtosCoreBootstrap
-> ProtosStandaloneProcessBootstrap
-> ProtosPolyglotRuntimeHost
-> ProtosPolyglotProcessContext
-> ProtosActivation
-> guest execution
```

`ProtosPolyglotRuntimeHost` and `ProtosPolyglotProcessContext` are public
implementation pieces, and the latter requires a `ProtosActivation` belonging
to the hosted Process. The CLI assembly that provides a normal standalone
invocation remains driver-owned/private machinery.

Duplicating that assembly in `guillermomolina/protos-benchmarks` would make
PERF021 depend on Protos runtime internals and create a second standalone
execution adapter solely for benchmarking.

## Coordination outcome

I076 was allocated as:

```text
I076 — Expose reusable standalone hosted execution for JVM embedding
GitHub issue: guillermomolina/protos#750
Status at allocation: ready
Semantic change: no
```

PERF021 is `guillermomolina/protos#748`. Its live status was changed from
`ready` to `blocked` because its required Protos JVM common-runner cell
cannot proceed cleanly until I076 provides the supported embedding surface.

The intended unblock condition is a narrow external JVM API that executes a
standalone Protos source through the existing semantic bootstrap/hosting
architecture without requiring the caller to construct Process, Activation,
standalone bootstrap, or Process-scoped Polyglot Context objects.

## Preserved PERF021 work

The blocker does not invalidate the already demonstrated GraalJS/GraalPy
Polyglot runner path, the canonical Fibonacci/Factorial workload equivalence,
or the local Protos JVM/Native artifacts.

PERF021 should resume its Protos JVM common-runner integration after I076 is
implemented and published. It should not replace the common JVM plane with a
benchmark-specific Protos CLI adapter.

## References

- `guillermomolina/protos#748` — PERF021
- `guillermomolina/protos#750` — I076
- `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotRuntimeHost.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java`
- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`
