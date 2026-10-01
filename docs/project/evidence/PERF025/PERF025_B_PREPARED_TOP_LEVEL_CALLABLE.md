# PERF025-B — prepared reusable top-level callable implementation evidence

Date: 2026-10-01

## Identity

```text
WORK_ITEM=PERF025/#758
SLICE=PERF025-B
TRIGGERED_BY=PERF024/#756
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
PROTOS_VERSION=0.3.131-SNAPSHOT
COMMIT=PERF025-B: add prepared top-level JVM callable
SEMANTIC_CHANGE=NO
```

This record retains the published implementation evidence for the second
product-side optimization promoted from PERF024.

## Trigger

PERF024 showed that the reusable Protos JVM embedding path performed a
top-level slot lookup and source-backed Closure validation on every
`invokeTopLevel(name)` call, while peer Truffle benchmark paths retained an
already-resolved executable value and invoked it repeatedly.

I077/#751 had already allowed one-time preparation of the top-level benchmark
entry, but the original public session API exposed only dynamic lookup.

## Published implementation

Revision
`6811d0cef3735d39ffd3801b3bae6ef48318bb66` adds:

```java
ProtosStandaloneHostedSession.prepareTopLevel(String name)
ProtosStandaloneHostedSession.PreparedTopLevel
PreparedTopLevel.invoke()
```

Preparation runs on the existing guest carrier and:

1. verifies the session is open;
2. resolves the named entry-module slot once;
3. requires a source-backed `ProtosClosureValue`; and
4. returns a handle retaining exactly that selected Closure.

Preparation does not execute the Closure and performs no new source parse,
Process construction or Context construction.

Each `PreparedTopLevel.invoke()` then reuses the owning session's:

```text
guest carrier
live Process
Polyglot Context / Engine / JIT identity
execution-host boundary
ProtosRootTaskExecution.executeClosure(...)
```

The last item means prepared calls automatically use the PERF025-A direct-first
fresh RootTask dispatch published in
`d0045353d834257b5fb80581846b32aebd43c7e6`.

## Dynamic versus prepared semantics

Existing:

```java
session.invokeTopLevel(name)
```

continues to read the module slot on every call and therefore observes guest
reassignment.

The prepared handle intentionally retains the exact Closure selected when
`prepareTopLevel(name)` runs and does not repeat
`entryModule.readLocalSlot(name)` during invocation.

The published tests prove the distinction with externally visible results:

```text
prepare original run
-> prepared invoke returns original result
-> guest replaces run slot
-> prepared invoke still returns original result
-> dynamic invokeTopLevel("run") returns replacement result
-> preparing run again captures replacement
```

No guest language or module-slot semantic change is introduced.

## Structural result

```text
PER_PREPARED_INVOCATION_TOP_LEVEL_SLOT_LOOKUP=NO
PER_PREPARED_INVOCATION_SOURCE_PARSE=NO
NEW_PROCESS_PER_INVOCATION=NO
NEW_CONTEXT_PER_INVOCATION=NO
EXISTING_CARRIER_REUSED=YES
PERF025_A_ROOT_DISPATCH_REUSED=YES
DYNAMIC_INVOKE_TOP_LEVEL_PRESERVED=YES
```

## Regression coverage

`ProtosStandaloneHostedSessionEmbeddingTest` adds focused external-consumer
coverage for:

- repeated prepared invocation;
- shared live module state;
- no execution during preparation;
- prepared-versus-dynamic behavior after top-level reassignment;
- fresh preparation selecting the replacement Closure;
- missing/non-Closure preparation rejection;
- invocation rejection after session close; and
- guest failure reported as a FAILED execution outcome while the prepared
  handle remains usable.

## Changed product surface

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java
src/test/java/com/guillermomolina/protos/embedding/ProtosStandaloneHostedSessionEmbeddingTest.java
```

The product version advances from `0.3.130-SNAPSHOT` to
`0.3.131-SNAPSHOT`.

Existing APL-1.0 notices are retained on modified Protos-owned source files.

## Validation evidence

The maintainer reported the requested tests passing before publication.

This documentation publication does not independently rerun the product test
suite.

```text
MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
PROTOS_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
```

## Result and routing

PERF025-A and PERF025-B have now removed two concrete repeated-invocation costs
identified by PERF024:

```text
A: fresh RootTask initial enqueue/dequeue
B: repeated top-level slot lookup for explicitly prepared embedding calls
```

PERF025 remains open for:

```text
PERF025-C — reduce/retire the BUG008 carrier tax caused by nested Bytecode
CallTarget host-stack amplification
```

Per owner direction, PERF025-C proceeds directly as implementation work before
another benchmark/JFR campaign.

## References

- `guillermomolina/protos#758` — PERF025
- `guillermomolina/protos#756` — PERF024
- Protos revision `6811d0cef3735d39ffd3801b3bae6ef48318bb66`
- PERF025-A revision `d0045353d834257b5fb80581846b32aebd43c7e6`
- historical BUG008 / `guillermomolina/protos#681` remains closed
