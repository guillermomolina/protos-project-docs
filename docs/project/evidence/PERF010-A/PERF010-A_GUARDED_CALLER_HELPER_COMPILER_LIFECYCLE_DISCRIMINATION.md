# PERF010-A / PERF011 — guarded caller/helper compiler lifecycle discrimination

Status: retained diagnostic result; existing guarded-call intervention does not remove the established caller-helper compiler stranding; no production optimization selected.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
PROTOS_VERSION=0.3.77-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b
HARNESS_COMMIT=perf010a: add caller/helper source-identity diagnostics

GUARDED_ABLATION=guarded-call
GUARDED_VARIANT=ablation
GUARDED_IMAGE=protos-benchmarks-perf010a-ablationguarded-call-ablation:2b3a88389da7

PRIOR_PROJECT_RECORD_REVISION=d99bb31041316bfbf5632eb095cb51f716909bb9
PRIOR_PROJECT_RECORD=docs/project/evidence/PERF010-A/PERF010-A_CALLER_HELPER_COMPILER_LIFECYCLE_ESTABLISHED.md

RUNTIME=GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Truffle 25.3.4.1
WORKLOAD=micro/method-call
EXPECTED_RESULT=42
OPERATION_COUNT=10000
WARMUP_ITERATIONS=20
STEADY_ITERATIONS=100
CPUSET=0
NETWORK=none
```

The full compiler-lifecycle trace was supplied by the human executor as a local diagnostic ZIP and is not published as benchmark timing evidence by this record.

```text
ZIP=perf010a-guarded-caller-helper-lifecycle.zip
ZIP_SHA256=018471cb0caf9d6449951ed92c9c77ec2c3d525946f966d29cde8082c0105436

STDOUT=perf010a-guarded-caller-helper-lifecycle.stdout
STDOUT_SHA256=d0b65336aff9741a44df2301b8e288a8ccbe16e8a1c55accafa05c94a83c5d9e
STDOUT_BYTES=1379

STDERR=perf010a-guarded-caller-helper-lifecycle.stderr
STDERR_SHA256=0986ec6f8e1fc8a1cda27164a75697ff6f7e4e1900006dc8d0f7c7231c138210
STDERR_BYTES=7308787
```

The diagnostic trace is intentionally separate from retained performance timing. Its timing samples are not used as benchmark evidence.

## Admission gate

Before collecting the full trace, the guarded image passed the exact source-identity admission gate:

```text
GUARDED_VARIANT=YES

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
PERF010A_SOURCE_IDENTITY_MATCH_ROOT_NODE_CLASS=com.guillermomolina.protos.execution.ProtosBytecodeRootNodeGen

METHOD_CALL_SOURCE_IDENTITY_VISIBLE=YES
```

The gate therefore established that the current guarded image contained the source-identity harness and exposed the exact caller source before the expensive compiler trace was collected.

## Runtime correctness

The full diagnostic execution completed successfully with the expected result:

```text
GUARDED_VARIANT=YES
EXPECTED_RESULT=42
RUNTIME=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
SOURCE_REUSED=true
PROCESS_REUSED=true
CONTEXT_REUSED=true
FRESH_ACTIVATION_PER_ITERATION=true
WARMUP_SAMPLE_COUNT=20
STEADY_SAMPLE_COUNT=100
PROCESS_EXIT_CODE=0
WORKLOAD_EXECUTION=PASS
```

A zero process exit code does not mean that every Truffle compilation succeeded. `CompilationFailureAction=Print` allows the workload to continue after the relevant helper compilation permanently bails out.

## Exact guarded caller roots

The exact source-identified caller remains:

```text
SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LENGTH=28
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
```

The guarded lifecycle trace identifies the caller pair as:

```text
GUARDED_CALLER_SEMANTIC_ROOT=
  id=154
  ProtosSemanticBytecodeRootNodeGen@50dfbc58
  Src method-call.protos:29

GUARDED_CALLER_HELPER_ROOT=
  id=153
  ProtosBytecodeRootNodeGen@6bf08014
  Src method-call.protos:29
```

## Guarded helper lifecycle

The exact guarded caller helper reaches the normal first-tier threshold and is submitted:

```text
id=153
ProtosBytecodeRootNodeGen@6bf08014
Tier 1
Count/Thres=400/400
Src method-call.protos:29
opt queued
```

It starts compilation and then fails permanently:

```text
id=153
Tier 1
opt start

id=153
Tier 1
opt failed
Reason: jdk.graal.compiler.core.common.PermanentBailoutException:
        Too deep inlining, probably caused by recursive inlining.
```

The complete stderr contains exactly three lifecycle lines for `id=153`:

```text
queued
start
failed
```

There is no later successful compilation, Tier 2 compilation, retry or invalidation event for that helper in the captured execution.

Therefore:

```text
GUARDED_CALLER_HELPER_COMPILATION_REQUESTED=YES
GUARDED_CALLER_HELPER_COMPILATION_COMPLETED=NO
GUARDED_CALLER_HELPER_COMPILATION_REJECTED=YES
GUARDED_CALLER_HELPER_INVALIDATED=NO
```

## The guarded specialization is on the failed PE path

The permanent bailout is not evidence that the guarded image simply fell through to the baseline generic specialization.

The failed helper's inlined-method trace explicitly contains:

```text
ProtosBytecodeRootNodeGen$CachedBytecodeNode.continueAt
  -> handlePrepareSendArguments_
  -> ProtosBytecodeRootNode$PrepareSendArguments.performGuardedOrdinaryComposedSend
  -> ProtosBytecodeRootNode.taskOwnedBytecodePlan
  -> ProtosLanguageContext.bytecodeExecutionPlanForEnteredClosure
  -> ProtosLanguageContext.bytecodeExecutionPlanForDefinition
  -> deep JDK generic / Locale / Formatter / reflection machinery
  -> PermanentBailoutException: Too deep inlining
```

The guarded intervention therefore changes the PE path and is active at the target operation, but it does not remove the effective compiler-stranding state.

The baseline and guarded failure paths are not byte-for-byte identical: baseline reached the bailout through the generic ordinary-send/import-resolution path, while guarded reaches it through the guarded specialization and its task-owned bytecode-plan resolution. What is common and discriminating here is the final lifecycle state: the exact hot caller helper is submitted and permanently rejected with the same failure class/reason, leaving no successful compiled helper.

## Guarded semantic-root lifecycle

The semantic wrapper for the same source compiles successfully.

Tier 1:

```text
id=154
Count/Thres=400/400
queued -> start -> done
AST=5
Inlined=0Y 0N
Truffle Callees=0
```

Tier 2:

```text
Count/Thres=10000/10000
queued -> stale
Count/Thres=10001/10000
queued -> start -> done
AST=5
Inlined=0Y 0N
Truffle Callees=0
```

Therefore:

```text
GUARDED_CALLER_SEMANTIC_COMPILATION_REQUESTED=YES
GUARDED_CALLER_SEMANTIC_COMPILATION_COMPLETED=YES
GUARDED_CALLER_HELPER_INLINED=NO
```

The semantic wrapper is optimized while the separate helper containing the repeated send has no successful compiled version.

## Baseline versus guarded discriminator

The previously retained baseline result is:

```text
BASELINE_CALLER_COMPILER_TOPOLOGY=C
BASELINE_HELPER_RESULT=PERMANENT_BAILOUT
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_BASELINE=YES
```

The guarded result is:

```text
GUARDED_CALLER_COMPILER_TOPOLOGY=C
GUARDED_HELPER_RESULT=PERMANENT_BAILOUT
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_GUARDED=YES
```

Therefore:

```text
GUARDED_REMOVES_BASELINE_COMPILER_STRANDING=NO
```

This is the requested discriminator.

## Interpretation of the previously retained negative guarded timing

The existing guarded-call timing remains valid for the intervention it measured, but it does not causally test removal of the now-established compiler-stranding mechanism.

Both BASELINE and GUARDED leave the exact repeated-send caller helper without successful compiled code. The previously measured guarded regression therefore cannot be used to conclude that removing this compiler boundary would fail to recover material runtime cost.

```text
NEGATIVE_GUARDED_TIMING_RELEVANCE_TO_STRANDING=NOT_DISCRIMINATING
```

This record does not infer the converse. A permanent bailout is a large architectural candidate, not proof that it is the dominant runtime cost. No attributable percentage is established.

## Reconciled result

```text
PERF010A_GUARDED_LIFECYCLE_DISCRIMINATOR=ESTABLISHED

BASELINE_CALLER_COMPILER_TOPOLOGY=C
GUARDED_CALLER_COMPILER_TOPOLOGY=C

BASELINE_HELPER_RESULT=PERMANENT_BAILOUT
GUARDED_HELPER_RESULT=PERMANENT_BAILOUT

GUARDED_REMOVES_BASELINE_COMPILER_STRANDING=NO
NEGATIVE_GUARDED_TIMING_RELEVANCE_TO_STRANDING=NOT_DISCRIMINATING

REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_BASELINE=YES
REPEATED_SEND_WORK_OUTSIDE_OPTIMIZED_REGION_GUARDED=YES

MATERIAL_COMMON_PATH_COST_PLAUSIBLE=YES
ORDER_OF_MAGNITUDE_RELEVANCE=HIGHLY_PLAUSIBLE

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED

PERF011_REACTIVATION_TRIGGER=SATISFIED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

## PERF011 relationship

The retained PERF011 reactivation trigger remains satisfied because the material compiler-visibility boundary is concrete and survives the existing guarded intervention.

This result still does not authorize a broad Shape/DynamicObject/Frame migration or any other representation rewrite. PERF011 should remain aligned with the same bounded compiler-stranding investigation rather than fork a separate experiment.

## Next-work constraint

This slice does not select or design a new production optimization.

The next causal experiment must target the established failure state itself rather than another nearby few-percent mechanism. Its admission criterion must demonstrate, for the exact source-identified `method-call.protos:29` caller helper, that the permanent bailout has actually been removed or materially changed before any resulting timing is interpreted as evidence about compiler stranding.

In particular, do not return merely to warmup tuning, `continueAt`, generic hotspot ranking, RootTag folding, another local representation change, or another guarded classification bypass unless that work is first shown to be necessary to remove the established caller-helper bailout.

Only after a bounded intervention actually changes the helper lifecycle can timing discriminate the recoverable contribution of this compiler-stranding mechanism and allow PERF010-A to establish or reject it as the dominant cause.
