# I076 — JVM standalone embedding closure evidence

Date: 2026-09-30

## Scope

This record preserves the implementation and closure evidence for I076.

It complements, and does not rewrite, the earlier
`I076_JVM_STANDALONE_EMBEDDING_GAP.md` trigger snapshot.

Operational issue state remains authoritative in `guillermomolina/protos`.

## Published implementation

I076 was published in Protos revision:

```text
dd16d26ab9c2b39cd738d4770c6309201fcaf555
I076: expose standalone hosted execution for JVM embedding
```

The implementation advances the development version to
`0.3.127-SNAPSHOT`.

The public embedding entry is:

```java
ProtosStandaloneHostedExecution.executeFile(...)
```

The convenience form accepts the Core root and source-file path. The fuller
form additionally accepts application arguments and standard streams.

The operation executes the standalone source on the dedicated Protos guest
carrier with the existing BUG008 stack budget and returns the terminal
`ProtosExecutionOutcome`.

This means an external JVM consumer can observe the terminal guest expression
directly. A benchmark consumer does not need to rewrite:

```protos
fibonacci(30)
```

as a benchmark-specific `print(fibonacci(30))` wrapper.

## Shared standalone authority

The implementation extracted/shared the standalone execution machinery rather
than adding a benchmark execution path.

`ProtosCli` now delegates common responsibilities to
`ProtosStandaloneHostedExecution`, including the relevant standalone Core and
Process bootstrap, stream/environment provisioning, Process-to-Polyglot binding,
and canonical direct-file execution.

The embedding caller therefore does not construct:

```text
ProtosActivation
ProtosProcessRuntime
ProtosStandaloneProcessBootstrap
ProtosPolyglotProcessContext
```

The implementation preserves Process-scoped Polyglot Context ownership and
terminates/releases the Process, Context/runtime host, resolver, and associated
resources before the hosted call returns.

The published changelog explicitly records no observable Protos semantic
change, no raw `Context.eval` behavior change, and no CLI behavior change.

## Acceptance coverage

Revision `dd16d26ab9c2b39cd738d4770c6309201fcaf555` adds
`ProtosStandaloneHostedExecutionEmbeddingTest` with external-consumer-style
coverage for:

- recursive Fibonacci, terminal value `832040`;
- recursive factorial, terminal value `2432902008176640000`;
- three repeated independent hosted invocations returning `42`.

The CLI Polyglot routing architecture test was also updated to verify that the
CLI routes direct-file execution through the new shared authority.

After publication, the maintainer reported that the requested I076 tests were
all green.

## Coordination closure

I076 is:

```text
guillermomolina/protos#750
status: completed
state: closed
implementation: dd16d26ab9c2b39cd738d4770c6309201fcaf555
semantic change: no
```

I076 had been the blocker introduced for the Protos JVM common-runner cell of
PERF021. With the hosted embedding surface published and validated, PERF021
(`guillermomolina/protos#748`) was restored from `status:blocked` to
`status:ready`.

PERF021 can now consume the public hosted execution API from its common JVM
benchmark machinery without requiring a Protos release and without reproducing
CLI/runtime bootstrap internals.

## References

- `guillermomolina/protos#750` — I076
- `guillermomolina/protos#748` — PERF021
- Protos revision `dd16d26ab9c2b39cd738d4770c6309201fcaf555`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java`
- `src/test/java/com/guillermomolina/protos/embedding/ProtosStandaloneHostedExecutionEmbeddingTest.java`
- `docs/project/evidence/I076/I076_JVM_STANDALONE_EMBEDDING_GAP.md`
