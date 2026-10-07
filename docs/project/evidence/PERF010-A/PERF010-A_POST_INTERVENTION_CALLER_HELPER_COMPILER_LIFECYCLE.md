# PERF010-A / PERF011 — post-intervention caller/helper compiler lifecycle

Status: retained post-intervention compiler-lifecycle discriminator; the prepared Context-owned target intervention materially changes the exact hot caller helper lifecycle, allowing successful Tier 1 and Tier 2 compilation before repeated uncommon-trap invalidations eventually return the helper to the generic preparation path and a permanent bailout. No timing attribution or production optimization is selected.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
PROTOS_VERSION=0.3.78-SNAPSHOT
PROTOS_COMMIT=
  PERF010-A: add prepared Context-owned target specialization for PrepareSendArguments

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b
BENCHMARK_COMMIT=
  perf010a: add caller/helper source-identity diagnostics

IMAGE=
  protos-benchmarks-perf010a-product-prepared-target:3a4afc27a96a
IMAGE_ID=
  sha256:3e914f6fb5b9fbe6de68ad7c885b70228874d2beeee2ff325cd8a1b894da1fe4

VARIANT=PRODUCT_PREPARED_TARGET
HARNESS_DOCKER_VARIANT=baseline
HARNESS_ABLATION_SLICE=none
OLD_GUARDED_CALL_PATCH_APPLIED=NO

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
WORKLOAD=micro/method-call
EXPECTED_RESULT=42
WARMUP_ITERATIONS=20
STEADY_ITERATIONS=100
NETWORK=none
CPUSET=NOT_RETAINED_IN_SUPPLIED_ARTIFACT
```

The diagnostic run used heavy compiler tracing and `CompilationFailureAction=Print`. Its timing samples are not interpreted as performance evidence.

The source-identity admission was run immediately before the compiler-lifecycle capture and reported:

```text
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD=micro/method-call
PERF010A_SOURCE_IDENTITY_SMOKE_ITERATIONS=2
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD_RESULT=PASS
PERF010A_SOURCE_IDENTITY_OBSERVED_RECORDS=23

PERF010A_SOURCE_IDENTITY_MATCH_SOURCE=method-call.protos
PERF010A_SOURCE_IDENTITY_MATCH_START_OFFSET=1226
PERF010A_SOURCE_IDENTITY_MATCH_END_OFFSET=1254
PERF010A_SOURCE_IDENTITY_MATCH_LENGTH=28
PERF010A_SOURCE_IDENTITY_MATCH_LINE=29
PERF010A_SOURCE_IDENTITY_MATCH_COLUMN_ONE_BASED=23
PERF010A_SOURCE_IDENTITY_MATCH_TEXT=sink = receiver.identity(42)
PERF010A_SOURCE_IDENTITY_MATCH_ROOT_NODE_CLASS=
  com.guillermomolina.protos.execution.ProtosBytecodeRootNodeGen

METHOD_CALL_SOURCE_IDENTITY_VISIBLE=YES
PRODUCT_PREPARED_TARGET_SOURCE_IDENTITY_GATE=PASS
```

## Supplied raw artifact identity

The human executor supplied the full stdout/stderr capture as a local ZIP. The raw ZIP is not published in this repository; its immutable identities are retained here for correlation.

```text
ZIP=perf010a-product-prepared-target-caller-helper-lifecycle.zip
ZIP_SHA256=b00b40e4858c46266a52e38d164cd2516eb0e0d2bde27dd3d9183d65f0776ead

STDOUT=perf010a-product-prepared-target-caller-helper-lifecycle.stdout
STDOUT_SHA256=4dffe7a514a6001dc6a27250fa0367a6664efc5a5a6881bc7fd48b49fea53963

STDERR=perf010a-product-prepared-target-caller-helper-lifecycle.stderr
STDERR_SHA256=5d7159552f29eaea02c63eef454478d34983906230b207c71f61f6db299a72e4
```

The stdout confirms the expected result, runtime identity, source/process/context reuse, fresh activation per iteration, 20 warmup samples and 100 steady samples. The `steady_ns` values are deliberately not summarized or compared here.

## Exact caller identity

The target source operation remains exactly:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LENGTH=28
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

The compiler trace identifies the exact pair as:

```text
CALLER_SEMANTIC_ROOT=
  id=154
  ProtosSemanticBytecodeRootNodeGen@4eaf3684
  Src method-call.protos:29

CALLER_HELPER_ROOT=
  id=153
  ProtosBytecodeRootNodeGen@3c01cfa1
  Src method-call.protos:29
```

## Previous established lifecycle

Before this product intervention, both the original BASELINE and the prior GUARDED experiment had topology C:

```text
semantic root:
  Tier 1 compilation completed
  Tier 2 compilation completed

helper root:
  Tier 1 threshold reached
  queued
  start
  permanent failure

helper inlined into semantic root:
  NO

BAILOUT=
  jdk.graal.compiler.core.common.PermanentBailoutException:
  Too deep inlining, probably caused by recursive inlining.

REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=YES
```

The helper had no successful compiled version in either retained pre-intervention capture.

## Post-intervention helper lifecycle

The new product revision changes that state immediately.

### First Tier 1

At the normal first-tier threshold:

```text
id=153
Tier 1
Count/Thres=400/400
queued
start
done
AST=7
Inlined=0Y 1N
```

Therefore:

```text
INITIAL_TIER1_COMPILATION=SUCCESS
```

This alone is a material change from the previous topology C, where the first Tier 1 compilation permanently failed.

### First Tier 2

At 10000/10000 the initial Tier 2 task becomes stale. At 10001/10000 the helper queues again. The trace then reports an uncommon-trap invalidation immediately before compilation starts, followed by successful Tier 2 compilation:

```text
id=153
Tier 2
Count/Thres=10001/10000
queued
deopt: Invalidated=true Reason uncommon trap
start
done
AST=7
Inlined=1Y 1N
```

Therefore:

```text
INITIAL_TIER2_COMPILATION=SUCCESS
HELPER_INVALIDATIONS=YES
INVALIDATION_REASON=uncommon trap
```

### Successful recompilation after invalidation

After another uncommon-trap invalidation, the helper recompiles successfully:

```text
id=153
Tier 1
Count/Thres=15926/400
queued
start
done
AST=7
Inlined=0Y 1N
```

It then reaches another successful Tier 2 compilation:

```text
id=153
Tier 2
Count/Thres=20423/10000
queued
start
deopt: Invalidated=true Reason uncommon trap
done
AST=7
Inlined=1Y 1N
```

A further uncommon-trap invalidation follows immediately.

### Final observed recompilation attempt

The final observed lifecycle for the exact helper is:

```text
id=153
Tier 1
Count/Thres=26130/400
queued
start
failed

Reason:
  jdk.graal.compiler.core.common.PermanentBailoutException:
  Too deep inlining, probably caused by recursive inlining.
```

No later successful compilation for `id=153` appears in the supplied trace.

Therefore:

```text
FINAL_HELPER_RECOMPILATION=PERMANENT_BAILOUT
FINAL_HELPER_STATE=UNCOMPILED_AFTER_PERMANENT_BAILOUT
```

## Final failed PE path

The final failed `id=153` partial-evaluation trace contains:

```text
ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt
  -> handlePrepareSendArguments_
  -> ProtosBytecodeRootNode$PrepareSendArguments.perform
  -> ProtosBytecodeRootNode.prepareSend
  -> ProtosBytecodeRootNode.prepareImmediateMethodCall
  -> ProtosModuleRuntime.prepareBytecodeImport
  -> ProtosModuleRuntime.prepareBytecodeImportWithTask
  -> ProtosStandardLibraryModuleResolver.resolve
  -> ProtosStandardLibraryModuleResolver.requireStandardLogicalName
  -> JDK Locale / Formatter / generic-reflection machinery
  -> PermanentBailoutException: Too deep inlining
```

The final failed trace does not contain `fastOrdinarySend`.

Accordingly:

```text
FAST_ORDINARY_SEND_EVER_ON_PE_PATH=INCONCLUSIVE
FAST_ORDINARY_SEND_ON_FINAL_FAILED_PE_PATH=NO

GENERIC_PREPARE_SEND_ON_FINAL_FAILED_PE_PATH=YES
OLD_STANDARD_IMPORT_PATH_ON_FINAL_FAILED_PE_PATH=YES
OLD_CONTEXT_PLAN_COMPUTE_IF_ABSENT_ON_FINAL_FAILED_PE_PATH=NO
```

The absence of `fastOrdinarySend` from the failed stack does not prove that it was absent from the earlier successful compilations: the successful compiler events do not carry the same complete failed-PE method dump. That stronger conclusion is intentionally not inferred.

## `computeIfAbsent` correlation

The complete stderr contains `computeIfAbsent` activity elsewhere, but the relevant occurrences correlate to another helper/root, including `method-call.protos:20`, not the exact source-identified caller helper at line 29.

Therefore the previous GUARDED context-plan-cache pathology must not be attributed to the final line-29 bailout merely because the token appears globally in the trace.

For the exact final `id=153` failure, the re-exposed pathological prefix is the original generic standard-import preparation path.

## Lifecycle discriminator

The required post-intervention classification is:

```text
PERF010A_POST_INTERVENTION_LIFECYCLE_DISCRIMINATOR=ESTABLISHED

CALLER_HELPER_PERMANENT_BAILOUT=MATERIALLY_CHANGED
CALLER_COMPILER_TOPOLOGY=CHANGED_FROM_C

INITIAL_TIER1_COMPILATION=SUCCESS
INITIAL_TIER2_COMPILATION=SUCCESS
RECOMPILATION_TIER1=SUCCESS
RECOMPILATION_TIER2=SUCCESS

HELPER_INVALIDATIONS=YES
INVALIDATION_REASON=uncommon trap

FINAL_HELPER_RECOMPILATION=PERMANENT_BAILOUT
FINAL_HELPER_STATE=UNCOMPILED_AFTER_PERMANENT_BAILOUT
```

The intervention therefore passes the previously defined lifecycle admission criterion `REMOVED | MATERIALLY_CHANGED` through the `MATERIALLY_CHANGED` outcome.

It does **not** establish stable removal of compiler stranding.

## Optimized-region classification

The previous single classification `REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION=YES` is no longer sufficient.

The retained lifecycle supports:

```text
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_INITIAL=NO
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_DURING_SUCCESSFUL_TIER2=NO
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_FINAL=YES
```

That is qualitatively different from the pre-intervention state: the exact caller helper now spends real execution phases with successful compiled Tier 1/Tier 2 versions, but those versions are repeatedly invalidated and the final observed recompilation permanently fails.

## Causal interpretation

The prepared-target intervention has changed the exact large compiler boundary previously under investigation. This is not another nearby few-percent source-only mechanism: the caller helper that previously could not compile at all now successfully compiles through Tier 2, recompiles again after invalidation, and only later returns to permanent bailout.

At the same time, the experiment exposes a narrower unresolved mechanism:

```text
successful helper compilation
  -> uncommon-trap invalidation(s)
  -> recompilation
  -> generic PrepareSendArguments.perform becomes PE-visible
  -> original standard-import / Locale host path returns
  -> permanent bailout
```

The current evidence does not yet establish why the fast specialization ceases to protect the final compilation. In particular it does not establish whether:

- `fastOrdinarySend` is invalidated/replaced in the generated DSL state;
- one of its selector/Closure/`methodHome`/Context/target guards stops matching;
- a specialization rewrite caused by the uncommon trap makes the generic `perform` path compiler-visible; or
- another generated-node lifecycle effect explains the transition.

Those alternatives require a bounded investigation before interpreting timing as the attributable cost of stable bailout removal.

## Timing status

Although the lifecycle driver emitted timing arrays, this diagnostic was run under heavy compiler tracing. No timing conclusion is drawn.

```text
TIMING_INTERPRETATION=DEFERRED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

The prior lifecycle admission gate has passed as `MATERIALLY_CHANGED`, but timing should remain deferred until the newly exposed uncommon-trap/fallback transition is understood well enough to know what state a paired timing run would actually be measuring.

## PERF011 relationship

This result strengthens the earlier PERF011 call-site compiler-visibility finding without authorizing a broad runtime-representation migration:

```text
CALL_SITE_COMPILER_VISIBILITY_MISMATCH=ESTABLISHED
PREPARED_TARGET_BOUNDARY_COMPILER_MATERIAL=YES
STABLE_PREPARED_TARGET_COMPILATION=NOT_ESTABLISHED
BROAD_REPRESENTATION_MIGRATION_AUTHORIZED=NO
```

The bounded prepared-target representation is compiler-material because it changes the exact caller from immediate Tier 1 permanent rejection to successful Tier 1/Tier 2 compilation.

PERF011 should remain aligned with PERF010-A rather than launching a separate Shape/DynamicObject/Frame migration.

## Next discriminator

The next slice is investigation only.

It should explain the exact transition from successful compiled `id=153` versions to the final generic-path permanent bailout, focusing on the recorded uncommon-trap invalidations and the generated `PrepareSendArguments` specialization lifecycle.

The investigation should determine, without modifying Protos or the benchmark harness:

```text
UNCOMMON_TRAP_CAUSE=...
FAST_SPECIALIZATION_STATE_BEFORE_TRAP=...
FAST_SPECIALIZATION_STATE_AFTER_TRAP=...
GENERIC_PERFORM_BECOMES_PE_VISIBLE_BECAUSE=...
FINAL_BAILOUT_REENTRY_CAUSE=...

STABLE_PREPARED_TARGET_COMPILATION_BLOCKER=
  ESTABLISHED | NOT_ESTABLISHED
```

Only after that transition is explained should a new implementation experiment or timing attribution be selected.
