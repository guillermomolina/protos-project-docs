# I070-A0 handwritten-main warning reconciliation evidence

I070-A0 is the read-only reconciliation checkpoint for I070-A /
`guillermomolina/protos#713`, child of I070 / #712. It determines the first
implementation slice for handwritten `src/main/java/**` Java/Javac and Truffle
DSL warnings without treating the historical 0.3.87-SNAPSHOT warning inventory
as current authority.

## Exact product revision

The source investigation was performed against:

```text
SOURCE_REVISION=9bed87430f99f3df8c2c3112f5a51b429a6e3176
SOURCE_VERSION=0.3.218-SNAPSHOT
SOURCE_SUBJECT=I054: retire generic GraalVM dynamic-LSP test evidence (#654)
```

While the investigation was in progress, `main` advanced to:

```text
RECONCILED_HEAD=00526bdeffaec4360f2ed2deed3d59e16ae3a431
RECONCILED_VERSION=0.3.219-SNAPSHOT
RECONCILED_SUBJECT=I054: reconcile publication metadata
```

The latter commit changes only `pom.xml` and `CHANGELOG.md`. The materially
inspected implementation blobs remain unchanged, so the source analysis applies
to the reconciled HEAD.

Observed build configuration at the reconciled revision:

```text
JAVA_RELEASE=21
GRAALVM_TRUFFLE=25.4.4.1.1
MAVEN_COMPILER_PLUGIN=3.13.0
SHOW_WARNINGS=true
XLINT=all
TRUFFLE_DSL_PROCESSOR=enabled
```

## Investigation boundary

No compilation, build, test, program, validator, mutable Git operation, code
change, Issue mutation, or product publication was performed as part of I070-A0.

Therefore this record distinguishes:

```text
STATIC_HEAD_RECONCILIATION=ESTABLISHED
EXACT_DYNAMIC_WARNING_BASELINE=NOT_ESTABLISHED
```

I070-A / #713 requires that exact dynamic baseline before editing.

## Historical warning-family reconciliation

### Truffle Bytecode DSL binds

The old intake counted four redundant explicit
`@Bind("$bytecodeNode")` / `@Bind("$frame")` expressions.

Current static inventory is much larger:

```text
ProtosSemanticBytecodeRootNode:
  @Bind("$bytecodeNode") = 42
  @Bind("$frame")        = 43

ProtosBytecodeRootNode:
  @Bind("$bytecodeNode") = 10
  @Bind("$frame")        = 7
```

These counts are candidate expressions, not emitted-warning counts.

The current repository also contains PE guards, including
`tools/java_local_accessor_pe_guard.py` and
`tools/java_local_range_operand_pe_guard.py`, that treat an explicit
`@Bind("$bytecodeNode")` parameter as structural provenance evidence.
Accordingly, a Truffle-redundant expression cannot be removed mechanically
without preserving the PE proof recognized by those guards.

### currentEnteredContext

The old intake counted two Truffle classification warnings.

Current source contains eleven `currentEnteredContext(Node)` helper
definitions across the two Bytecode roots. They ultimately call
`ProtosLanguageContext.current(node)`, which delegates to
`TruffleLanguage.ContextReference.get(node)`.

For the pinned Truffle version, `ContextReference.get` is classified by the
DSL processor as non-idempotent. Therefore, if the current dynamic baseline
still reports helper-classification warnings, `@NonIdempotent` is the
defensible classification; `@Idempotent` is not.

### serialVersionUID

Current statically evident serializable exception subclasses without an
explicit UID include:

- `ProtosEncodingValue.ConversionFailure`
- `ProtosTestResourceProviderRegistry.UnknownProviderException`
- `ProtosTestResourceProviderCoordinator.InvalidLeaseCoverageException`
- `ProtosTestResourceProviderCoordinator.DuplicateResourceKeyException`
- `ProtosTaskCancellationException`
- `ProtosParallelRuntime.NonParallel`
- `ParseError`
- `ProtosNonLocalReturnException`
- `ProtosSignalException`

`ProtosLexer.LexicalError` and
`ProtosBytecodeControlTransferException` already declare
`serialVersionUID = 1L`.

### Non-serializable fields

Several serializable-by-inheritance exceptions contain runtime state whose
declared field types are not serializable, including:

- `ParseError.span`
- `ProtosNonLocalReturnException.target`
- `ProtosNonLocalReturnException.value`
- `ProtosSignalException.error`
- `ProtosSignalException.selectedHandlerFrame`
- `ProtosSignalException.originBytecode`
- `ProtosSignalException.terminalDiagnosticTrace`
- `ProtosBytecodeControlTransferException.originBytecode`

There is no static repository evidence of an intended Java serialization
contract for these runtime/control-transfer exceptions. The warning must not be
silenced by mechanically adding `transient`, by making Protos runtime objects
serializable, or by inventing `readObject`/`writeObject` semantics. Any
suppression, if needed, must be narrow and justified as accidental
serializability inherited from the Java/Truffle exception hierarchy.

### Dangling documentation comments

The three historical unattached documentation comments still have a clear
static counterpart in
`src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java`.

Immediately before the real Javadoc for `dynamicControlState()`, the file
contains three stale consecutive Javadocs describing:

1. materialization of legacy evaluator replay state;
2. non-creating B6B inspection of legacy evaluator replay state;
3. host execution of one Task-owned C-prime segment.

They are not attached to declarations and describe retired/distinct surfaces.
The following fourth Javadoc correctly documents `dynamicControlState()`.

This is the lowest-risk independently valid implementation slice.

## Recommended slice order

```text
I070-A1  remove the three stale dangling Javadocs in ProtosTask
I070-A2  UID-only serial warnings confirmed by the dynamic baseline
I070-A3  accidental exception-serialization warnings requiring per-class treatment
I070-A4  currentEnteredContext Truffle classification warnings
I070-A5+ redundant Bytecode DSL binds, coordinated with PE guards
```

The exact A2-A5 partition may be adjusted to the dynamic compiler baseline.
A1 is independently justified by current static source evidence.

## Next implementation checkpoint

```text
NEXT_SLICE=I070-A1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
PRECONDITION=exact current clean Maven warning baseline before editing
PRIMARY_FILE=src/main/java/com/guillermomolina/protos/runtime/ProtosTask.java
PATCH_SCOPE=remove only the three stale dangling Javadocs
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
```

The implementation agent must use the real current repository HEAD, capture the
clean Maven compilation warning baseline before changing files, and must not
assume the historical total of 23 handwritten-main warnings remains current.

I070-A0 verdict:

```text
VERDICT=NEEDS_DYNAMIC_BASELINE_BEFORE_PATCH
DESIGN_BLOCKER=NO
FIRST_IMPLEMENTATION_SLICE=READY_AFTER_BASELINE
```

This is non-normative project evidence. Live work status remains owned by
GitHub Issues #713 and #712.
