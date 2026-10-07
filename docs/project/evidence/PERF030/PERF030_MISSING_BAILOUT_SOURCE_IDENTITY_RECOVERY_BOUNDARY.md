# PERF030-R — Missing bailout source-identity recovery boundary

## Scope

PERF030-R is an investigation-only continuation of PERF030 / `#784`.

It determines whether the exact Java source stack / BCI for the permanent
outer-`run` required-constant bailout observed by PERF030-O can already be
recovered from published project artifacts or metadata. If not, it identifies
the smallest mechanically sufficient future capture surface.

This slice executed no build, benchmark, runtime, JFR, IGV, shell, Maven, Java,
Python, Git, or other local command.

## Exact authorities

```text
PRODUCT_FAILURE_AUTHORITY=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
CURRENT_PROTOS_HEAD=a5b00b7c1bb72bacd6ef36bc714a3c8e0972dac4
BENCHMARK_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
PREVIOUS_PROJECT_RECORD_REVISION=efe32375b73526fd0a812ee016fb7eb4fe4b46cc
GRAAL_25_4_SOURCE_REVISION=95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b
FAILING_ROOT_SEMANTIC_ROLE=OUTER_RUN
FAILURE_COMPILER_NODE=36664|Pi
```

Between the PERF030-O/N product failure authority and current Protos `main`,
the changed product paths do not touch the indexed outer-current-creation
corridor. Therefore:

```text
RELEVANT_CORRIDOR_DRIFT=NO
```

## Published-artifact recovery result

The PERF030-O evidence names local diagnostic paths under
`results/local/**`, including `recording.jfr`, `identity.json`, and IGV
artifacts.

At benchmark revision `dfc2a34d...`, however:

- `results/local/` is explicitly ignored by `.gitignore`;
- `results/local/` is absent from the published repository tree;
- GitHub history contains no commits for `results/local` or
  `results/local/truffle-diagnostics`;
- the published project-record repository contains the retained
  `failureReason`/semantic attribution but no Java-source stack or BCI for
  `36664|Pi`; and
- no associated GitHub Actions workflow artifact was found for the exact
  Protos, benchmark, or project-record authority revisions.

Accordingly:

```text
EXISTING_PUBLISHED_SOURCE_IDENTITY=NOT_FOUND
FAILING_ASSERTION=UNRESOLVED
FAILING_VALUE_CATEGORY=UNRESOLVED
FIRST_LOSS_POINT=UNRESOLVED
WHY_36664_PI_MAPS_TO_THIS_VALUE=NOT_PROVEN
```

The numeric Graal node id remains non-durable source identity.

## Where Graal creates the missing identity

At exact Graal revision
`95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`,
`TruffleGraphBuilderPlugins.failPEConstant` calls
`GraphBuilderContext.bailout` at the failing
`CompilerAsserts.partialEvaluationConstant` invocation.

Relevant source:

- `compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/substitutions/TruffleGraphBuilderPlugins.java`
  lines 1596-1609;
- `compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/java/BytecodeParser.java`
  lines 4363-4369;
- `compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/nodes/util/GraphUtil.java`
  lines 677-687 and 723-724; and
- `compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/code/SourceStackTraceBailoutException.java`
  lines 29-53.

The path is:

```text
PEConstantPlugin
 -> failPEConstant
 -> GraphBuilderContext.bailout
 -> BytecodeParser.bailout
 -> create FrameState at current BCI
 -> GraphUtil.approxSourceStackTraceElement
 -> SourceStackTraceBailoutException.stackTrace
```

`SourceStackTraceBailoutException` deliberately replaces its compiler stack
with the Java-source stack of the code being compiled. Therefore the original
bailout already carries the source identity required by PERF030.

## Where the JFR reporting path loses it

The decisive reporting boundary is in
`TruffleCompilerImpl.compileAST`.

At exact Graal revision `95ce1499...`, the compiler listener is called as:

```text
listener.onFailure(
    compilable,
    t.toString(),
    bailout != null,
    permanentBailout,
    task.tier(),
    bailout != null ? null : () -> TruffleCompilable.serializeException(t)
)
```

For a bailout, including the permanent
`SourceStackTraceBailoutException` in PERF030-O, the compiler-listener
`lazyStackTrace` is therefore explicitly `null`.

The JFR path is then:

```text
TruffleCompilerImpl listener.onFailure
  bailout != null -> lazyStackTrace=null
 -> JFRListener.onCompilationFailed
 -> CompilationEventImpl.failed
 -> CompilationFailureEventImpl.setFailureData
 -> CompilationFailure.stackTrace=null
```

The JFR failure event does define a textual `stackTrace` field, but this
specific bailout path supplies no value to it. The event itself is also declared
with `@StackTrace(false)`.

Therefore:

```text
SOURCE_IDENTITY_EXISTS_AT_COMPILER_LAYER=
  BytecodeParser.bailout FrameState/current-BCI
  -> GraphUtil.approxSourceStackTraceElement
  -> SourceStackTraceBailoutException.stackTrace

SOURCE_IDENTITY_LOST_BEFORE_RETAINED_EVIDENCE=
  TruffleCompilerImpl compiler-listener onFailure bailout branch
  sets lazyStackTrace=null before JFRListener/CompilationFailure
```

This identifies the diagnostic reporting loss boundary. It does **not**
identify the first semantic/PE point where the actual value ceased to be a
partial-evaluation constant.

## The exception is preserved on the compilable failure path

The compiler has a second failure-reporting path.

`TruffleCompilerImpl.notifyCompilableOfFailure` passes:

```text
() -> TruffleCompilable.serializeException(finalError)
```

to `TruffleCompilable.onCompilationFailed`.

`TruffleCompilable.serializeException(Throwable)` ultimately calls
`Throwable.printStackTrace`, so the serialized form retains the
`SourceStackTraceBailoutException` source stack.

`OptimizedCallTarget.handleCompilationFailure` emits that serialized
exception when the engine compilation failure action is at least `Print`.

Graal's exact-version Truffle documentation defines:

```text
engine.CompilationFailureAction=Silent|Print|Throw|Diagnose|ExitVM
```

and documents `Print` as printing the exception to the console.

For a Java launcher, upstream examples use:

```text
-Dpolyglot.engine.CompilationFailureAction=Print
```

This is the smallest upstream-supported surface that exposes the already
existing exception source stack.

## Why the recovered stack is mechanically discriminating

At the exact Graal source revision, `LocalRangeAccessor` contains distinct
Java-source assertions for each required-constant value.

`setObject`:

```text
225  setObject(...)
226    partialEvaluationConstant(this)
227    partialEvaluationConstant(bytecodeNode)
228    partialEvaluationConstant(offset)
```

`isCleared`:

```text
340  isCleared(...)
341    partialEvaluationConstant(this)
342    partialEvaluationConstant(bytecodeNode)
343    partialEvaluationConstant(offset)
```

The generated interpreter `partialEvaluationConstant(bci)` is a different
method/source position.

A retained `SourceStackTraceBailoutException` source frame can therefore
distinguish at least:

```text
LocalRangeAccessor.isCleared: this
LocalRangeAccessor.isCleared: bytecodeNode
LocalRangeAccessor.isCleared: offset
LocalRangeAccessor.setObject: this
LocalRangeAccessor.setObject: bytecodeNode
LocalRangeAccessor.setObject: offset
generated interpreter: bci
other exact assertion if present
```

No interpretation of `36664|Pi` is needed.

## Current harness capability

At benchmark revision `dfc2a34d...`,
`truffle/jvm_diagnostic.py`:

- supports `jfr` and `igv` diagnostics;
- inserts its JVM diagnostic options directly into the Java command;
- for JFR uses
  `-XX:StartFlightRecording=...,settings=profile,dumponexit=true`;
- captures combined stdout/stderr with
  `stdout=PIPE, stderr=STDOUT`; and
- already persists that combined output as `run.log`.

The harness does not expose a generic cache-keyed mechanism for injecting an
additional JVM diagnostic option such as
`-Dpolyglot.engine.CompilationFailureAction=Print`.

An untracked ambient JVM option would also be unsuitable because diagnostic
identity/cache reuse must distinguish captures performed with different JVM
diagnostic settings.

## Minimum future capture surface

The smallest mechanically sufficient future change is a bounded generic harness
change in:

```text
guillermomolina/protos-benchmarks
```

It should:

1. provide a generic diagnostic-only JVM-option surface;
2. include the normalized selected options in diagnostic identity/cache-key
   material;
3. inject those options into the existing Java diagnostic command; and
4. use it for the PERF030 capture with:

```text
-Dpolyglot.engine.CompilationFailureAction=Print
```

No new output transport is required: the existing `run.log` already captures
the needed console exception.

The future capture should then reproduce the same bounded workload authority and
retain the exact source frame from the permanent outer-root bailout as durable
PERF030 evidence.

A product change is not required to obtain this evidence.

```text
MINIMUM_FUTURE_CAPTURE_SURFACE=
  generic cache-keyed JVM diagnostic option
  + CompilationFailureAction=Print
  + existing run.log retention

CAPTURE_REPOSITORY=guillermomolina/protos-benchmarks
PRODUCT_CHANGE_REQUIRED=NO
BENCHMARK_HARNESS_CHANGE_REQUIRED=YES
```

## Alternative larger surfaces

A higher-verbosity IGV capture could also be made mechanically useful.

`failPEConstant` requests its graph dump at
`DebugContext.VERBOSE_LEVEL`, which is level 3 at the exact Graal revision.
The current benchmark IGV diagnostic uses:

```text
-Djdk.graal.Dump=Truffle:1
```

so the specific pre-bailout dump is not guaranteed by the current capture.
A future `Truffle:3` dump, optionally combined with
`compiler.NodeSourcePositions=true`, is therefore a larger alternative, not
the minimum capture.

`engine.TraceCompilation` / `TraceCompilationDetails` is not sufficient:
the listener prints the failure `reason` and does not consume the supplied
lazy exception stack.

`CompilationFailureAction=Diagnose` is also broader than needed because
`Print` already exposes the exception representation carrying the desired
stack.

## Guard status

PERF030-R does not prove which value failed the PE-constant contract.

The previous guard classification therefore remains unchanged:

```text
STATIC_GUARD_INDEX_MODEL_SOUND=YES
STATIC_GUARD_COMPLETE_FOR_LOCAL_RANGE_ACCESS=NO
GUARD_REPAIR_REQUIRED=YES

OFFSET_PE_PROVENANCE=PE_CONSTANT_FROM_OPERATION
LOCAL_RANGE_ACCESSOR_RECEIVER_PE_CONSTANT=NOT_PROVEN
CURRENT_BYTECODE_NODE_PE_CONSTANT=NOT_PROVEN
OTHER_REQUIRED_CONSTANT=CANNOT_BE_EXCLUDED
```

No product repair is authorized from this investigation.

## Issue boundary

The future capture is one bounded diagnostic continuation of PERF030. No
independent closure, blockage, scheduling, dependency, decision checkpoint, or
multi-publication trigger was found.

```text
NEW_FORMAL_ISSUE_REQUIRED=NO
PROMOTION_TRIGGER=NONE
```

## Result

```text
PERF030_R=INCOMPLETE

PRODUCT_FAILURE_AUTHORITY=cf6eb4c9aeaef4049fa5b56089d2dd1d2654d44b
CURRENT_PROTOS_HEAD=a5b00b7c1bb72bacd6ef36bc714a3c8e0972dac4
RELEVANT_CORRIDOR_DRIFT=NO

EXISTING_PUBLISHED_SOURCE_IDENTITY=NOT_FOUND

FAILING_ASSERTION=UNRESOLVED
FAILING_VALUE_CATEGORY=UNRESOLVED
FIRST_LOSS_POINT=UNRESOLVED
WHY_36664_PI_MAPS_TO_THIS_VALUE=NOT_PROVEN

PRODUCT_REPAIR_BOUNDED=NO
DESIGN_DECISION_REQUIRED=NO
GUARD_REPAIR_REQUIRED=YES

PERF030_CLOSE_READY=NO
PERF024_GRAPH_INTERPRETATION_READY=NO

NEW_FORMAL_ISSUE_REQUIRED=NO
PROMOTION_TRIGGER=NONE

NEXT_SLICE=PERF030-S
NEXT_WORK_TYPE=IMPLEMENTATION
NEXT_IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks
NEXT_PURPOSE=Add a generic cache-keyed JVM diagnostic option surface and use CompilationFailureAction=Print to preserve the bailout source stack
```

## Validation and provenance

PERF030-R itself is investigation-only and executed no local validation command.
The maintainer subsequently reported that all local tests pass.

AI assistance: this durable evidence record was drafted with ChatGPT from the
current published GitHub state of `guillermomolina/protos`,
`guillermomolina/protos-benchmarks`, and
`guillermomolina/protos-project-docs`, plus exact GraalVM 25.4 source at
`95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`.
