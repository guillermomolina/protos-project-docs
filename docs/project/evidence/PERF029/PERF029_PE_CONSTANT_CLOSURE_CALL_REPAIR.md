# PERF029 — PE-constant closure-call frame-local repair publication

## Scope

This record preserves the exact published Protos product revision for the first
PERF029 implementation slice. It is non-normative evidence for the compilerability
repair that removes the two previously observed permanent Truffle partial-evaluation
bailouts from the ordinary source-backed closure-call path.

The owning live work item is
`guillermomolina/protos#783` (**PERF029 — Remove permanent PE bailouts from
closure-call bytecode roots**). PERF024 / #756 remains the owner of the later
cross-runtime Protos/GraalJS/GraalPy physical inventory.

## Published authority

```text
PROTOS_REVISION=63df450263feb5bfbfb3160aecf33128f456e08c
PROTOS_VERSION=0.3.172-SNAPSHOT
COMMIT_SUBJECT=PERF029: make closure-call frame local ordinals PE-constant
BASELINE_TRIGGER_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
BENCHMARK_HARNESS_TRIGGER_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
WORKLOAD=primitive-closure-call
OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
```

## Triggered compiler failures

The baseline diagnostic had two current Protos guest roots terminate partial
evaluation with permanent bailouts before an optimized BGV could be emitted.

### A. Name-keyed persistent frame authority

```text
ProtosFrameLexicalBindingAuthority.putBinding
  -> LocalRangeAccessor.isCleared(bytecodeNode, frame, offset)
  -> CompilerAsserts.partialEvaluationConstant(...)
  -> PermanentBailoutException
  -> 7799|Pi
```

The local ordinal was derived from a runtime binding name through
`ProtosFrameLexicalLayout.offsetOf(name)`. That runtime-name path cannot provide
the PE-time constant index required by `LocalRangeAccessor`.

### B. Compact closure parameter binding

```text
ProtosSemanticBytecodeRootNode.BindClosureFrameParameter.perform
  -> LocalRangeAccessor.isCleared(bytecodeNode, frame, ordinal)
  -> CompilerAsserts.partialEvaluationConstant(...)
  -> PermanentBailoutException
  -> 5866|AnyNarrow
```

The frame-local ordinal was passed as an ordinary stack/runtime operand even
though lowering already knew the exact layout ordinal.

## Published implementation

The product commit makes two bounded changes.

1. `BindClosureFrameParameter` now declares `ordinal` as a Bytecode DSL
   `@ConstantOperand(type = int.class, name = "ordinal")`.
   `CanonicalToBytecodeLowerer` passes the statically known ordinal directly
   as that constant operand instead of emitting it as an ordinary runtime
   value.

2. `ProtosFrameLexicalBindingAuthority.putBinding(String, Object)` is now a
   `@TruffleBoundary`. This path is intentionally name-keyed and resolves its
   frame ordinal dynamically, so it is kept outside partial evaluation rather
   than presenting a non-constant index to `LocalRangeAccessor`.

The product commit also adds/extends focused regression coverage for:

- the constant ordinal encoded on `BindClosureFrameParameter`;
- persistent frame-authority installation over pre-existing bindings;
- migration order and values;
- ABSENT versus PRESENT semantics;
- `PRESENT(null) != ABSENT`;
- duplicate creation rejection; and
- removal followed by recreation ordering.

## Product files changed

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI075DLexicalAuthorityCurrentBytecodeNodeTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java
```

## Validation and acceptance state

This coordination step has stable evidence that the product commit is published
on GitHub. It does not independently reproduce or claim a local build/test run.

The decisive PERF029 acceptance is still external to this product commit:

```text
EXTERNAL_IGV_ACCEPTANCE=PENDING
EXPECTED_COMMAND_SURFACE=truffle/jvm_diagnostic.py igv protos primitive-closure-call
EXPECTED_RESULT=Protos guest compilation succeeds and emits one or more relevant BGV files
```

The external diagnostic must use the existing generic benchmark harness without
workload-specific changes. If a distinct new permanent bailout appears after
these two repairs, it is new evidence and must not be silently folded into this
publication claim.

## Coordination state

```text
PERF029_PRODUCT_SLICE=PUBLISHED
PERF029_PRODUCT_REVISION=63df450263feb5bfbfb3160aecf33128f456e08c
PERF029_PRODUCT_VERSION=0.3.172-SNAPSHOT
KNOWN_BAILOUT_A_SOURCE_REPAIR=PUBLISHED
KNOWN_BAILOUT_B_SOURCE_REPAIR=PUBLISHED
EXTERNAL_IGV_ACCEPTANCE=PENDING
PERF029_CLOSE_READY=NO
PERF024_PHYSICAL_INVENTORY_READY=NO
NEXT_ACTION=RUN_UNCHANGED_PROTOS_IGV_ACCEPTANCE
```

AI assistance: this durable evidence record was drafted with ChatGPT from the
published commit and previously captured diagnostic evidence.
