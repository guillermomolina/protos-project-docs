# PERF030 — indexed persistent-authority current creation publication

## Scope

This record preserves the published PERF030-D product repair that followed the
PERF030-C source re-attribution.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains paused for cross-runtime physical graph
interpretation until unchanged external IGV/JFR acceptance proves that the
selected `primitive-closure-call` path no longer has the permanent compiler
blocker.

This is non-normative implementation/publication evidence. It does not alter
Protos language semantics and does not itself establish an external performance
or compiler-acceptance result.

## Published authority

```text
SLICE=PERF030-D
WORK_TYPE=IMPLEMENTATION

PROTOS_REVISION=12a42ff718144162ee72bb321b3eca7d70ce3cc9
PROTOS_VERSION=0.3.179-SNAPSHOT
COMMIT_SUBJECT=PERF030: index persistent-authority current creation by constant ordinal

PREVIOUS_PERF030_A_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PERF030_C_RECORD_REVISION=e570404f58afda541fa16ad439b57174ffed0cab
BENCHMARK_HARNESS_AUTHORITY=dfc2a34dacc40e57e2e625789c5d3300615bc979

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
BENCHMARK_CHANGE=NO
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
```

## Re-attributed blocker being repaired

PERF030-C established that the surviving `primitive-closure-call` bailout did
not execute the PERF030-A `CreateCurrentFrameLocal` path.

For the outer `run` Closure, the nested `identity` Closure literal forces
persistent frame authority. Its target-less creation therefore followed:

```text
CreateCurrentLocalSlot
 -> ProtosActivation.createCurrentLocalSlotForRuntime
 -> ProtosFrameLexicalBindingAuthority.containsBinding
 -> ProtosFrameLexicalLayout.offsetOf(name)
 -> LocalRangeAccessor.isCleared(..., offset)
```

The runtime name-resolved `offset` was the surviving PE-visible non-constant
local index.

## Published implementation

The product commit introduces a bounded indexed creation path for exactly the
statically proven persistent-authority case.

### Lowering-time identity proof

`CanonicalToBytecodeLowerer.indexedCreateOrdinal(CanonicalCreate)` admits an
indexed creation only when:

- the root has `currentRootIndexedParameterLayout`;
- no alternate current activation local is active;
- root analysis is available;
- the binding is a current root frame local; and
- `currentRootAnalysis.identityOf(create)` proves that the create identity is
  owned by `currentRootTopScope` and has the expected name.

The ordinal is obtained from the root's known
`ProtosFrameLexicalLayout` during lowering.

### New Bytecode DSL operation

The implementation adds:

```text
CreateCurrentIndexedLocalSlot
```

with:

```java
@ConstantOperand(
        type = ProtosFrameLexicalLayout.class,
        name = "frameBackedLayout")
@ConstantOperand(type = int.class, name = "ordinal")
```

The known ordinal is passed as a Bytecode DSL constant operand; it is not loaded
as a runtime stack operand.

### Runtime admission and fallback

`ProtosBytecodeRootNode.createIndexedCurrentLocalSlot(...)` follows the
existing indexed-parameter precedent:

```text
activation.currentAuthorityAdmittingLocalCreationForRuntime()
 -> ProtosFrameLexicalBindingAuthority
 -> authority.storesLayout(frameBackedLayout)
 -> authority.createFrameBackedBindingAt(ordinal, value)
```

The indexed path still translates invalid/duplicate creation to the ordinary
Protos Error behavior.

If the expected authority/layout guard is not admitted, execution delegates to
the unchanged:

```text
CreateCurrentLocalSlot.perform(activation, name, value)
```

No second authority or alternate semantic creation rule is introduced.

## Focused regression

`ProtosPerf025CompactCalleeExecutionTest` now covers the exact relevant shape:

```protos
() => {
    identity: (value) => { value }
    identity(1)
}
```

The published regression verifies:

- `InstallFrameLexicalAuthority` is present;
- `CreateCurrentIndexedLocalSlot` is present;
- ordinary `CreateCurrentLocalSlot` is absent from that proven path;
- `CreateCurrentFrameLocal` is also absent from the outer persistent-authority
  path;
- the indexed creation carries the same persistent layout installed by the
  root;
- its ordinal is the layout ordinal for `identity`;
- execution returns `1`;
- duplicate creation still signals a Protos Error; and
- a foreign-layout case takes the named fallback and retains its duplicate
  creation behavior.

## Product files changed

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
```

## Validation state

The maintainer reported all local tests PASS after publishing the product
commit.

```text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
PRODUCT_PUBLICATION=PASS
```

This record does not claim that the external compiler blocker has disappeared.

## Required external re-acceptance

The next slice must rerun the existing generic diagnostics against this exact
published product revision without changing the workload or diagnostic regime:

```text
PRODUCT_REVISION=12a42ff718144162ee72bb321b3eca7d70ce3cc9
PRODUCT_VERSION=0.3.179-SNAPSHOT
BENCHMARK_HARNESS_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979

WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
REQUIRED_DIAGNOSTICS=igv,jfr
REQUIRED_CORRECTNESS_RESULT=1
```

Acceptance requires:

- correctness PASS for both generic diagnostic paths;
- ordinary IGV output/BGV admission remains successful;
- JFR contains no permanent recurrence attributable to the repaired
  `8231|Pi` blocker;
- PERF029's previously removed `7799|Pi` and `5866|AnyNarrow` blockers remain
  absent; and
- any distinct new permanent bailout is classified independently rather than
  silently folded into PERF030-D.

## Coordination

```text
PERF030_D=COMPLETE
PERF030_D_PRODUCT_REVISION=12a42ff718144162ee72bb321b3eca7d70ce3cc9
PERF030_D_PRODUCT_VERSION=0.3.179-SNAPSHOT
PRODUCT_REPOSITORY_VALIDATION=PASS_REPORTED_BY_MAINTAINER
EXTERNAL_IGV_JFR_ACCEPTANCE=PENDING

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_WORK=PERF030-E unchanged external IGV/JFR acceptance
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
published Protos commit, PERF030-C durable source re-attribution, live GitHub
coordination, and the maintainer-reported local validation result.
