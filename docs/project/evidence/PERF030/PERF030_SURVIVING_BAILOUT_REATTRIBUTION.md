# PERF030 — surviving bailout source re-attribution

## Scope

This record preserves the PERF030-C diagnostic re-attribution that followed the
failed PERF030-B external IGV/JFR acceptance.

The owning live work item is
`guillermomolina/protos#784` (**PERF030 — Make frame-local creation ordinal
PE-constant**). PERF024 / #756 remains paused for cross-runtime physical graph
interpretation until Protos has no permanent compiler blocker on the selected
`primitive-closure-call` diagnostic.

This is non-normative diagnostic evidence. It does not change Protos semantics,
does not itself modify product code, and does not claim a performance
improvement.

## Authorities

```text
SLICE=PERF030-C
WORK_TYPE=INVESTIGATION
IMPLEMENTATION=NO

PRODUCT_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
PRODUCT_VERSION=0.3.177-SNAPSHOT
PERF030_A_RECORD_REVISION=542c7dee07a0a799815abc0179bfe225665e2b49
PERF030_B_RECORD_REVISION=9fdc3e76db3ea061483aa145ffff99f9a70a64d4

BENCHMARK_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000

PROJECT_COMMANDS_EXECUTED_BY_AGENT=NO
BUILDS_EXECUTED_BY_AGENT=NO
TESTS_EXECUTED_BY_AGENT=NO
BENCHMARKS_EXECUTED_BY_AGENT=NO
JFR_EXECUTED_BY_AGENT=NO
IGV_EXECUTED_BY_AGENT=NO
GIT_MUTATION_EXECUTED_BY_AGENT=NO
NOTE=One accidental local no-op command ("true") was executed outside the project repositories; it read or changed no project state.
```

The exact failing JFR evidence from PERF030-B remains:

```text
rootFunction=ProtosSemanticBytecodeRootNodeGen@67601481
truffleTier=1
permanentFailure=true
failureReason=jdk.graal.compiler.code.SourceStackTraceBailoutException$1:
  Partial evaluation did not reduce value to a constant,
  is a regular compiler node: 8231|Pi
```

The object suffix is not used as semantic root identity.

## Superseded source attribution

The PERF030-A publication record correctly preserves the product change it
published, but its benchmark-specific attribution is now superseded.

The earlier record stated that the outer `run` closure in
`primitive-closure-call` created `identity` through
`CreateCurrentFrameLocal`. Exact-revision reachability inspection proves that
this was not the path executed by that workload, including at the original
PERF030 trigger revision.

The PERF030-A product repair remains valid as a bounded product improvement:
`CreateCurrentFrameLocal.ordinal` is a Bytecode DSL `@ConstantOperand` and
lowering passes the ordinal directly to that constant operand. The correction is
only to the attribution of the surviving `primitive-closure-call` bailout.

## Exact workload reachability

The workload is:

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

At both:

```text
PERF030_TRIGGER_REVISION=63df450263feb5bfbfb3160aecf33128f456e08c
PERF030_A_REVISION=383ffc025e82918336e3bc2ab350f1275886a576
```

`CanonicalToBytecodeLowerer.requiresPersistentFrameAuthority(...)` delegates to
`demandsPersistentFrame(...)`, and a `CanonicalClosure` demands persistent
frame authority.

The RHS of the outer `identity` creation is a Closure literal. Therefore the
outer `run` root installs persistent frame authority:

```text
requiresPersistentFrameAuthority(run)=true
currentRootFrameNativeLayout=null
currentRootIndexedParameterLayout=frameLocalLayout
```

`frameNativeOrdinal("identity")` consequently returns `-1`, so
`emitCreateCurrentBinding(...)` cannot emit `CreateCurrentFrameLocal`. It
emits the named persistent-authority operation:

```text
CreateCurrentLocalSlot
```

The nested `identity(value)` root is the frame-native root in this workload. It
binds `value` through the PERF029-repaired `BindClosureFrameParameter` path;
it does not create the outer `identity` local.

The prepared benchmark resolves the top-level `run` Closure once and repeatedly
invokes only that Closure. The workload contributes one additional hot guest
Closure root, `identity(value)`. PERF030-B observed one root compiling
successfully at tiers 1 and 2 and one root permanently failing at tier 1. Combined
with the exact reachable operations, the failing semantic root is the outer
`run` root.

```text
FAILING_ROOT_SEMANTIC_ROLE=OUTER_RUN
CREATE_CURRENT_FRAME_LOCAL_REACHED=NO
INNER_IDENTITY_PATH=BIND_CLOSURE_FRAME_PARAMETER
```

## Re-attributed source path

The outer `run` root creates `identity` through:

```text
ProtosSemanticBytecodeRootNode.CreateCurrentLocalSlot
 -> ProtosBytecodeRootNode.CreateCurrentLocalSlot.perform
 -> ProtosActivation.createCurrentLocalSlotForRuntime
 -> ProtosFrameLexicalBindingAuthority.containsBinding
 -> ProtosFrameLexicalLayout.offsetOf(name)
 -> LocalRangeAccessor.isCleared(bytecodeNode, frame, offset)
```

`ProtosFrameLexicalLayout.offsetOf(String name)` is a runtime map lookup:

```java
Integer offsetOf(String name) {
    return offsets.get(name);
}
```

`ProtosFrameLexicalBindingAuthority.containsBinding(String name)` then passes
that `Integer offset` to `LocalRangeAccessor.isCleared(..., offset)`.

That is the surviving PE-visible non-constant local index. The Truffle Bytecode
DSL contract for `LocalRangeAccessor` requires local offsets used by these
accessors to be partial-evaluation constants. The GraalVM 25.4
`@ConstantOperand` contract separately confirms that constant operands have
compilation-final semantics and exist specifically to avoid relying on ordinary
runtime stack operands becoming constants during PE.

The subsequent `ProtosFrameLexicalBindingAuthority.putBinding(...)` is already
behind `@TruffleBoundary`; therefore the first relevant PE-visible
`LocalRangeAccessor` access on this creation path is the
`containsBinding(name)` check.

```text
PERF030_C_ATTRIBUTION=DIFFERENT_PATH
SURVIVING_NON_CONSTANT_VALUE=runtime name-resolved frame ordinal
SURVIVING_LOCAL_ACCESS=ProtosFrameLexicalBindingAuthority.containsBinding
PERF030_A_CONSTANT_OPERAND_REPAIR_IS_SURVIVING_CAUSE=NO
```

## Existing repair seam

The product already contains the semantics-preserving primitive needed for a
bounded repair:

```java
ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt(
        int ordinal,
        Object value)
```

It performs the exact PRESENT/duplicate check and write at a known ordinal.

The existing `bindIndexedClosureParameter(...)` path is the precedent: it asks
`activation.currentAuthorityAdmittingLocalCreationForRuntime()`, verifies that
the authority is the expected `ProtosFrameLexicalBindingAuthority` storing the
expected layout, uses the known ordinal through
`createFrameBackedBindingAt(...)`, and otherwise falls back to the named
semantic path.

The next bounded product slice should reuse that pattern for a statically proven
target-less current creation in a root that already has persistent frame
authority:

```text
persistent-authority root
+ statically proven current binding identity
+ known ProtosFrameLexicalLayout ordinal
    -> indexed current-creation operation
       carrying layout and ordinal as Bytecode DSL constants
    -> currentAuthorityAdmittingLocalCreationForRuntime()
    -> createFrameBackedBindingAt(ordinal, value)

otherwise
    -> existing CreateCurrentLocalSlot named fallback
```

No new language semantics, specification decision, or new runtime authority is
required.

## Expected implementation boundary

The bounded implementation is expected to be confined primarily to:

```text
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
```

Normal product publication metadata may additionally require `pom.xml` and
`CHANGELOG.md`.

`ProtosFrameLexicalBindingAuthority` already exposes the required indexed
creation seam, so no change there is presently justified by PERF030-C.

## Focused regression requirement

The focused regression should exercise the actual failing semantic shape:

```protos
() => {
    identity: (value) => { value }
    identity(1)
}
```

It should prove:

- the outer root still installs persistent frame authority;
- the statically known `identity` creation does not use the runtime-name
  `CreateCurrentLocalSlot` ordinary path;
- the new indexed creation carries the expected constant ordinal/layout;
- normal result remains unchanged;
- duplicate creation / PRESENT-vs-ABSENT semantics remain unchanged; and
- the named fallback remains authoritative when the indexed guard is not
  admitted.

## External re-acceptance

After the next product repair is published, the unchanged external PERF030
acceptance remains:

```text
WORKLOAD=primitive-closure-call
DIAGNOSTIC_WARMUP=60
DIAGNOSTIC_STEADY=10
DIAGNOSTIC_SAMPLE_CALLS=100000
REQUIRED_DIAGNOSTICS=igv,jfr
REQUIRED_CORRECTNESS_RESULT=1
REQUIRED_8231_RESULT=ABSENT
```

A different permanent compiler blocker must be classified independently rather
than silently folded into this attribution.

## Coordination

This bounded repair does not cross a project Issue-promotion trigger. It is a
continuation of PERF030/#784 and cannot close independently of PERF030's
unchanged external acceptance.

```text
NEW_FORMAL_ISSUE_REQUIRED=NO
OWNING_ISSUE=guillermomolina/protos#784
PERF030_C=COMPLETE
SOURCE_REATTRIBUTION=COMPLETE
ORIGINAL_BENCHMARK_ATTRIBUTION=SUPERSEDED
PRODUCT_PATCH_AUTHORIZED_BY_EVIDENCE=YES
IMPLEMENTED=NO
PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
NEXT_WORK=PERF030 implementation slice for indexed persistent-authority current creation
```

AI assistance: this durable evidence record was drafted with ChatGPT from exact
historical Protos and benchmark revisions, PERF030-A/PERF030-B durable records,
live GitHub coordination, and GraalVM 25.4 Bytecode DSL source/API contracts.
