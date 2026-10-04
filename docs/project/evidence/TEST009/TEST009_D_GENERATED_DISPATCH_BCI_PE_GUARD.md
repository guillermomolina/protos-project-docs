# TEST009-D generated dispatch BCI PE guard closure evidence

Date: 2026-10-04

## Exact publication

```text
TEST009_D=COMPLETE
PROTOS_REVISION=8973817c9308fddb807983d988d6dac5aedbafa1
COMMIT=TEST009-D: guard generated dispatch BCI PE constancy
PARENT_ISSUE=TEST009/#795
TRIGGER_PARENT=PERF030/#784
SEMANTIC_CHANGE=NO
```

The later revision:

```text
9e8e7bf2b89050f1a6749960b606c4bf3be03845
```

only repairs a stale TEST009-C self-test and does not modify TEST009-D.

## Guard surface

TEST009-D added:

```text
tools/java_generated_bytecode_bci_pe_guard.py
tools/test_java_generated_bytecode_bci_pe_guard.py
tools/java_generated_bytecode_bci_pe_baseline.json
tools/java_generated_bytecode_bci_compilation_check.py
tools/test_java_generated_bytecode_bci_compilation_check.py
```

and the Make target:

```text
make check-generated-bytecode-bci-pe
```

The target packages Protos first so annotation-processor generated Java is current, then runs the static guard self-tests, dynamic compilation-check self-tests, and whole generated-source guard.

It is included in aggregate `make check`.

## Static family contract

At the reviewed Graal/Truffle generator revision:

```text
oracle/graal@95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
```

the cached Bytecode DSL interpreter asserts:

```java
CompilerAsserts.partialEvaluationConstant(bci);
```

inside its MERGE_EXPLODE dispatch loop.

The guard therefore inventories every generated cached dispatch and every value that can feed its next `bci`.

The generated roots covered are:

```text
ProtosBytecodeRootNodeGen
ProtosSemanticBytecodeRootNodeGen
```

and both cached variants are represented:

```text
CachedBytecodeNode.continueAt
CachedBytecodeNodeTailCall.continueAt
```

## Reviewed generated topology

The exact reviewed baseline contains:

```text
GENERATED_ROOTS_FOUND=2
CACHED_DISPATCHES_FOUND=4
DISPATCH_BCI_ASSERTIONS_FOUND=4

BCI_ENTRY_SITES=10
BCI_INITIALIZATIONS=4
BCI_TRANSITIONS=940
```

Entry-site classifications:

```text
ENTRY_OSR_TARGET=4
ENTRY_CONTINUATION_LOCATION=2
ENTRY_ROOT_STATE_LOOP=2
ENTRY_LITERAL_ZERO=2
```

Generated transition/initialization classifications:

```text
PROVEN_HANDLER_RETURN=895
BYTECODE_IMMEDIATE_TARGET=2
SEQUENTIAL_CONSTANT_DELTA=15
START_STATE_DECODE=4
DISPATCH_EXIT=22
THROWING_HANDLER=2
EXCEPTION_HANDLER_TABLE=4
```

The four `EXCEPTION_HANDLER_TABLE` transitions are deliberately classified as structurally known but dynamically discharged rather than falsely claimed as a complete static PE proof.

No risk or unknown entry is baselineable.

## Dynamic discharge

The static proof cannot by itself prove that the non-exploded generated exception-handler search always folds its selected BCI to a PE constant.

TEST009-D therefore includes a real synchronous compilation check using:

```text
-Dpolyglot.engine.AllowExperimentalOptions=true
-Dpolyglot.engine.CompileImmediately=true
-Dpolyglot.engine.BackgroundCompilation=false
-Dpolyglot.engine.CompilationFailureAction=Print
-Dpolyglot.engine.TraceCompilation=true
```

The guest program deliberately covers:

```text
backward branch
ordinary branch
Error.handle / TryCatch handler-table re-entry
ensure / TryFinally
non-local return
ordinary closures
runtime and semantic generated roots
```

The check requires the expected guest result, evidence that both generated roots actually compiled, and zero permanent compilation failure attributable to generated dispatch `partialEvaluationConstant(bci)`.

Other permanent compilation failures are reported but intentionally not owned by TEST009-D; they belong to the next general dynamic compilerability family.

## Aggregate validation

The first TEST009-D aggregate run was blocked only by a stale TEST009-C self-test. That independent self-test inconsistency was repaired in:

```text
9e8e7bf2b89050f1a6749960b606c4bf3be03845
TEST009-C: fix stale Bytecode API guard self-test
```

After that correction, the maintainer reports:

```text
MAKE_CHECK=PASS
ALL_LOCAL_TESTS=PASS
GENERATED_BYTECODE_BCI_GUARD=PASS
DYNAMIC_BCI_COMPILATION_CHECK=PASS
```

Because `check-generated-bytecode-bci-pe` is a prerequisite of `make check`, the successful aggregate run validates both the static generated-source proof and the synchronous Truffle compilation discharge.

The Makefile records the complete generated-BCI target at approximately 31 seconds, so it remains acceptable in the current aggregate check budget.

## Resulting state

```text
TEST009_D=COMPLETE
GENERATED_BYTECODE_DISPATCH_BCI_FAMILY=CLEAN
BCI_TRANSITION_RISK=0
BCI_TRANSITION_UNKNOWN=0
OLD_CODE_FAMILY_DEBT_REMAINING=0
SEMANTIC_CHANGE=NO

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO
```

TEST009-D intentionally leaves general permanent compilation failures outside its ownership. That is now the next known TEST009 family.
