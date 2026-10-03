# PERF030 — PE-constant frame-local creation repair publication

## Scope

This record preserves the exact published Protos product revision for the
PERF030 implementation slice. It is non-normative evidence for the bounded
compilerability repair that makes the statically known
`CreateCurrentFrameLocal` frame-local ordinal a Bytecode DSL constant operand.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains the owner of the subsequent
cross-runtime Protos/GraalJS/GraalPy physical graph comparison after external
acceptance.

## Published authority

```text
PROTOS_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PROTOS_VERSION=0.3.177-SNAPSHOT
COMMIT_SUBJECT=PERF030: make frame-local creation ordinal PE-constant
BASELINE_TRIGGER_REVISION=63df450263feb5bfbfb3160aecf33128f456e08c
BENCHMARK_HARNESS_TRIGGER_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
```

## Triggered compiler failure

The PERF024 raw physical inventory found a second Protos semantic bytecode root
that permanently failed partial evaluation:

```text
rootFunction=ProtosSemanticBytecodeRootNodeGen@22fad29a
truffleTier=1
permanentFailure=true
failureReason=jdk.graal.compiler.code.SourceStackTraceBailoutException$1:
  Partial evaluation did not reduce value to a constant,
  is a regular compiler node: 8231|Pi
```

This failure is distinct from PERF029's previously removed `7799|Pi` and
`5866|AnyNarrow` bailouts.

Exact source inspection of the trigger revision established:

- `BindClosureFrameParameter.ordinal` was already a Bytecode DSL constant
  operand from PERF029;
- `CreateCurrentFrameLocal.ordinal` remained an ordinary runtime operation
  operand;
- lowering emitted `builder.emitLoadConstant(ordinal)` before
  `endCreateCurrentFrameLocal()`; and
- `CreateCurrentFrameLocal.perform` passed that runtime ordinal to
  `LocalRangeAccessor.isCleared(..., ordinal)` and
  `LocalRangeAccessor.setObject(..., ordinal, ...)`.

For `primitive-closure-call`, the outer `run` closure creates its local
`identity` binding through this path.

## Published implementation

The product commit makes the bounded repair requested by PERF030-A:

1. `ProtosSemanticBytecodeRootNode.CreateCurrentFrameLocal` now declares:
   ```java
   @ConstantOperand(type = int.class, name = "ordinal")
   ```

2. `CanonicalToBytecodeLowerer.emitCreateCurrentBinding` now supplies the
   statically known ordinal directly to
   `beginCreateCurrentFrameLocal(..., ordinal)` instead of emitting it as a
   runtime stack operand.

3. The compact-call direct creation path and activation fallback remain
   unchanged, including duplicate-creation Error behavior, OPEN/frozen/conflict
   semantics and returned-value behavior.

4. The focused regression extends
   `ProtosPerf025CompactCalleeExecutionTest` to prove that
   `CreateCurrentFrameLocal` carries its own instruction constant while the
   direct compact execution behavior still succeeds.

## Product files changed

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
```

## Validation state

The maintainer reported all requested tests PASS before publication.

```text
MAINTAINER_REPORTED_TESTS=PASS
PRODUCT_PUBLICATION=PASS
```

This record does not claim that the external compiler diagnostic has already
removed `8231|Pi`. That assertion belongs to PERF030-B and must be established
against the published product revision using the unchanged generic diagnostic
surface.

Required external acceptance regime:

```text
PROTOS_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PROTOS_VERSION=0.3.177-SNAPSHOT
BENCHMARK_HARNESS_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
REQUIRED_DIAGNOSTICS=igv,jfr
```

Acceptance must prove:

- correctness remains PASS;
- the ordinary IGV path produces BGV output;
- the JFR capture has no recurrence of the permanent `8231|Pi` failure; and
- if a different permanent compiler blocker appears, it is classified as new
  evidence rather than folded into this implementation publication.

## Coordination state

```text
PERF030_A=COMPLETE
PERF030_PRODUCT_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PERF030_PRODUCT_VERSION=0.3.177-SNAPSHOT
SOURCE_REPAIR=PUBLISHED
MAINTAINER_REPORTED_TESTS=PASS
EXTERNAL_IGV_JFR_ACCEPTANCE=PENDING
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEXT_ACTION=PERF030-B unchanged external IGV/JFR acceptance
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
published product commit, the PERF024 diagnostic evidence, and the
maintainer-reported validation result.
