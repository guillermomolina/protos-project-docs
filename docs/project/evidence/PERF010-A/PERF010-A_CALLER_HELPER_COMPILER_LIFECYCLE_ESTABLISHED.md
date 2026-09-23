# PERF010-A — caller/helper compiler lifecycle established

Status: retained compiler-lifecycle evidence; caller/helper compilation boundary established; causal attributable fraction not yet established.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
PROTOS_VERSION=0.3.77-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
SOURCE_IDENTITY_HARNESS_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b
SOURCE_IDENTITY_HARNESS_COMMIT=
  perf010a: add caller/helper source-identity diagnostics

PRIOR_BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
PRIOR_HARNESS_REVISION=3275c108aa9b04d35a67fbe9be1c13e6fe94c58a

PRIOR_PROJECT_RECORD=
  docs/project/evidence/PERF010-A/
  PERF010-A_CALLER_HELPER_TRACE_SOURCE_IDENTITY_BLOCKER.md

PRIOR_PROJECT_RECORD_REVISION=29a38fd3c0fb2f10a1fb5dc4a6a4616e05abe98e

PROJECT_RECORD_BASE_REVISION=12bba303edff2680d41d2d8c42792ccd6f8c0981

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
WORKLOAD=micro/method-call
EXPECTED_RESULT=42
OPERATION_COUNT=10000
WARMUP_ITERATIONS=20
STEADY_ITERATIONS=100
```

The raw trace was supplied by the human executor as a local diagnostic ZIP and is not published as benchmark evidence by this record.

Local artifact identity retained for reproducibility/correlation:

```text
ZIP=perf010a-caller-helper-lifecycle.zip
ZIP_SHA256=8d560085ba1ff8ae82d593932cc0c478fb6607a6d7ff41ead5effafbeca5c41d

STDOUT=perf010a-caller-helper-lifecycle.stdout
STDOUT_SHA256=4fed32c3639093ce8b70bf9d626e5d56fdbedfe25eb61d14c56f3fe1096f1126

STDERR=perf010a-caller-helper-lifecycle.stderr
STDERR_SHA256=4a228185803ff092a3c5378f6f6b0197a7cad1ba5ca28c1c7fc0254e2de7c357
```

## Source-identity admission

Before the full trace, the new harness-local Truffle instrument passed its exact source-identity smoke gate:

```text
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD=micro/method-call
PERF010A_SOURCE_IDENTITY_SMOKE_ITERATIONS=2
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD_RESULT=PASS

PERF010A_SOURCE_IDENTITY_MATCH_SOURCE=method-call.protos
PERF010A_SOURCE_IDENTITY_MATCH_START_OFFSET=1226
PERF010A_SOURCE_IDENTITY_MATCH_END_OFFSET=1254
PERF010A_SOURCE_IDENTITY_MATCH_LENGTH=28
PERF010A_SOURCE_IDENTITY_MATCH_LINE=29
PERF010A_SOURCE_IDENTITY_MATCH_COLUMN_ONE_BASED=23
PERF010A_SOURCE_IDENTITY_MATCH_TEXT=sink = receiver.identity(42)

METHOD_CALL_SOURCE_IDENTITY_VISIBLE=YES
PERF010A_SOURCE_IDENTITY_SMOKE=PASS
```

No Protos product patch was used to obtain source identity.

## Runtime correctness

The full compiler-lifecycle execution completed the expected workload:

```text
EXPECTED_RESULT=42
RUNTIME=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
SOURCE_REUSED=true
PROCESS_REUSED=true
CONTEXT_REUSED=true
FRESH_ACTIVATION_PER_ITERATION=true
WARMUP_SAMPLE_COUNT=20
STEADY_SAMPLE_COUNT=100

WORKLOAD_EXECUTION=PASS
```

The run uses heavy compiler diagnostics and is not retained as timing evidence.

## Exact caller root pair

The target caller source remains:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LENGTH=28
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

The full trace identifies exactly one semantic/helper pair at that source line:

```text
CALLER_SEMANTIC_ROOT=
  id=154
  ProtosSemanticBytecodeRootNodeGen@147a5d08
  Src method-call.protos:29

CALLER_HELPER_ROOT=
  id=153
  ProtosBytecodeRootNodeGen@7cbd9d24
  Src method-call.protos:29
```

This runtime pairing agrees with the previously retained construction-time relation:

```text
ProtosBytecodeClosureExecutionPlan
  -> activationRoot.getCallTarget()
  -> helperTarget passed to ProtosSemanticBytecodeRootNode.wrap(...)
  -> InvokeSemanticHelper
  -> helperTarget.call(activation)
```

## Caller helper lifecycle

The exact caller helper reaches the normal first-tier threshold and is submitted:

```text
id=153 ProtosBytecodeRootNodeGen@7cbd9d24
Tier 1
Count/Thres=400/400
Src method-call.protos:29
opt queued
```

It then starts compilation:

```text
id=153
Tier 1
opt start
Src method-call.protos:29
```

and fails permanently:

```text
id=153
Tier 1
opt failed

jdk.graal.compiler.core.common.PermanentBailoutException:
Too deep inlining, probably caused by recursive inlining.
```

The complete trace contains exactly three lifecycle lines for id=153:

```text
queued
start
failed
```

There is no later queue, successful compilation, Tier 2 compilation, or retry for that target in the captured execution.

Therefore:

```text
CALLER_HELPER_COMPILATION_REQUESTED=YES
CALLER_HELPER_COMPILATION_COMPLETED=NO
CALLER_HELPER_COMPILATION_REJECTED=YES
CALLER_HELPER_INVALIDATED=NO
```

## Exact failed compilation path

The permanent bailout's inlined-method stack reaches the caller's repeated send path:

```text
ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt
  -> handlePrepareSendArguments_
  -> ProtosBytecodeRootNode$PrepareSendArguments.perform
  -> ProtosBytecodeRootNode.prepareSend
  -> ProtosBytecodeRootNode.prepareImmediateMethodCall
  -> ProtosModuleRuntime.prepareBytecodeImport
  -> ProtosModuleRuntime.prepareBytecodeImportWithTask
  -> ProtosModuleRuntime.resolveModuleKey
  -> ProtosStandardLibraryModuleResolver.resolve
  -> ProtosStandardLibraryModuleResolver.requireStandardLogicalName
  -> deep JDK generic/Locale/Formatter/string-bound machinery
  -> PermanentBailoutException: Too deep inlining
```

The trace therefore directly connects the failed target to the helper containing the repeated ordinary composed-send preparation.

The bailout is not merely a module-bootstrap target that happens to share the same trace: the failed target itself is the source-identified `method-call.protos:29` helper, and its PE stack includes `PrepareSendArguments`.

## Caller semantic lifecycle

The semantic wrapper for the same source compiles successfully.

Tier 1:

```text
id=154
Count/Thres=400/400
opt queued
opt start
opt done
AST=5
Inlined=0Y 0N
Src method-call.protos:29
```

Tier 2 first queues at 10000/10000, that task becomes stale, then it queues again at 10001/10000 and completes:

```text
id=154
Tier 2
opt queued
opt unqueued: Reason Stale compilation task

id=154
Tier 2
opt queued
opt start
opt done
AST=5
Inlined=0Y 0N
Src method-call.protos:29
```

The inlining trace for the semantic wrapper reports:

```text
Truffle Callees=0
Inlined=0Y 0N
```

at both Tier 1 and Tier 2.

Therefore:

```text
CALLER_SEMANTIC_COMPILATION_REQUESTED=YES
CALLER_SEMANTIC_COMPILATION_COMPLETED=YES
CALLER_SEMANTIC_COMPILATION_REJECTED=NO
CALLER_SEMANTIC_INVALIDATED=NO

CALLER_HELPER_INLINED=NO
```

The source implementation uses `helperTarget.call(activation)` rather than an explicit `DirectCallNode`, and the compiled semantic root exposes zero Truffle callees. For this retained lifecycle classification:

```text
CALLER_HELPER_DIRECT_TRUFFLE_CALL=NO
CALLER_HELPER_TARGET_PE_CONSTANT=INCONCLUSIVE
```

No stronger PE-constancy conclusion is inferred from source finality alone.

## Compiler topology

The observed topology is:

```text
caller semantic root
  -> successfully compiled Tier 1 / Tier 2
  -> helper not inlined
  -> helper independently reaches Tier 1 threshold
  -> helper compilation permanently bails out
  -> helper remains without successful compiled code
```

Therefore:

```text
CALLER_COMPILER_TOPOLOGY=C
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=YES
```

This resolves the previous BGV ambiguity. The earlier graph capture showed the small `identity` callee helper but not the caller helper containing the repeated sends. The lifecycle trace now establishes why the caller helper was absent from successful optimized graphs: its compilation is attempted and permanently rejected.

## PERF010-A scale interpretation

The failed helper contains repeated caller-side work including `PrepareSendArguments`, send preparation and immediate method-call preparation. That work executes on the hot caller path while the helper has no successful compiled version in this run.

This has qualitatively different scale from the already-tested few-percent local mechanisms and is consistent with the large cross-runtime performance question.

However, this is still a compiler-lifecycle observation, not an attributable timing intervention.

Therefore:

```text
MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES
ORDER_OF_MAGNITUDE_RELEVANCE=HIGHLY_PLAUSIBLE

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

No claim is made that this boundary explains the full cross-runtime gap.

## PERF011 relationship

The previously retained reactivation condition is now satisfied: the compiler-visibility boundary is concrete, hot-path relevant, and large enough to merit the shared PERF010-A/PERF011 discriminator.

```text
PERF011_REACTIVATION_TRIGGER=SATISFIED
PERF011_STATUS=REACTIVATED_FOR_SHARED_DISCRIMINATOR
```

This does not authorize a broad runtime-representation migration. PERF011 should consume the next bounded causal/diagnostic result rather than launch an independent rewrite.

## Exactly one next discriminator

Do not implement a new ablation yet.

Reuse the already-retained guarded-call intervention and run the **same source-identified compiler lifecycle diagnostic** on the GUARDED image for `micro/method-call`.

The exact discriminator is whether the existing guarded intervention changes the caller helper lifecycle:

```text
BASELINE:
  method-call.protos:29 helper id=153
  -> Tier 1 requested
  -> PermanentBailoutException
  -> no successful compiled helper

GUARDED:
  does the exact source-identified caller helper
  -> still permanently bail out?
  -> or successfully compile?
```

Interpretation:

- If GUARDED still fails with the same effective bailout path, the previously measured negative guarded timing did not causally test removal of the newly established compiler-stranding mechanism.
- If GUARDED causes the helper to compile successfully while retained timing remains negative, the helper-compilation hypothesis is materially weakened as an explanation of recoverable runtime cost.
- If the lifecycle changes in another concrete way, classify that exact difference before designing any new intervention.

No new Protos source change, new fast path, warmup tuning, `continueAt` work, threshold modification, or compiler-policy modification belongs in that slice.
