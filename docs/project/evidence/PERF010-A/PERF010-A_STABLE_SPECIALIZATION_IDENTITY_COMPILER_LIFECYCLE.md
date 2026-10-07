# PERF010-A / PERF011 — stable specialization identity compiler lifecycle

Status: retained compiler-lifecycle evidence establishing that the stable executable-identity cache key removes the previously observed fresh-Closure/methodHome PIC churn for the exact `micro/method-call` caller helper. The source-identified helper reaches Tier 2 and remains compiled for the remainder of the 20-warmup/100-steady capture. Timing samples emitted by the diagnostic run are not interpreted here.

## Evidence identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=3e8e6b565c95eb5098c2168d241536ba13ad19e9
PROTOS_VERSION=0.3.79-SNAPSHOT
PROTOS_COMMIT=
  PERF010-A: key fastOrdinarySend cache on stable Closure definition identity

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
BENCHMARK_REVISION=e0bbf213c8140491f91913712cbf6d9cd0270e0b
BENCHMARK_COMMIT=
  perf010a: add caller/helper source-identity diagnostics

IMAGE=
  protos-benchmarks-perf010a-product-stable-identity:3e8e6b565c95
HARNESS_DOCKER_VARIANT=baseline
HARNESS_ABLATION_PATCH=noop.patch
HARNESS_ABLATION_PATCH_APPLIED=NO

RUNTIME_GRAALVM=25.3.4.1
RUNTIME_JDK=25.0.4.1
RUNTIME_TRUFFLE=25.3.4.1
RUNTIME_MAVEN=3.9.9
CONTAINER_IMAGE=
  ghcr.io/graalvm/graalvm-community:25i3-25.0.4.1-ol10-20260825

WORKLOAD=micro/method-call
EXPECTED_RESULT=42
WARMUP_ITERATIONS=20
STEADY_ITERATIONS=100
CPUSET=0
NETWORK=none
```

The image records `source_dirty=false` and the exact Protos revision above.

## Supplied raw artifact identity

The human executor supplied the complete capture as a local ZIP. As with the previous compiler-lifecycle record, the raw ZIP is not published in this repository; immutable hashes are retained for correlation.

```text
ZIP=perf010a-stable-identity-caller-helper-lifecycle.zip
ZIP_SHA256=8364d0ac62233b8c2d7236c18495394dbd8945a38d3ae4067b94955aefe0803e

LIFECYCLE_STDOUT=lifecycle.stdout
LIFECYCLE_STDOUT_SHA256=807f155a6ef065b677a50ca1395c4ee9558163c98709dcd880015e11ab89f778

LIFECYCLE_STDERR=lifecycle.stderr
LIFECYCLE_STDERR_SHA256=fcdfba73bdbe767516f5a0d35b2ad69feae996b0aa2777159013276ff73de498

LIFECYCLE_INDEX=lifecycle.index.txt
LIFECYCLE_INDEX_SHA256=a753d002cedaa0a553c4ed00f186a40c7f0009db22915775c354d3928d974207

SOURCE_IDENTITY_STDOUT=source-identity.stdout
SOURCE_IDENTITY_STDOUT_SHA256=201d04e377283aa02e7554d72d0d212b7d68a1d4c3b7b3cd9dd398a429658ddf
```

The captured process and driver contract both exited successfully:

```text
LIFECYCLE_EXIT_CODE=0
LIFECYCLE_DRIVER_CONTRACT_EXIT_CODE=0

SOURCE_REUSED=PASS
PROCESS_REUSED=PASS
CONTEXT_REUSED=PASS
FRESH_ACTIVATION_PER_ITERATION=PASS
WARMUP_ITERATIONS=PASS
STEADY_ITERATIONS=PASS

TIMING_INTERPRETATION_FOR_THIS_CAPTURE=FORBIDDEN
```

## Source-identity admission

```text
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD=micro/method-call
PERF010A_SOURCE_IDENTITY_SMOKE_ITERATIONS=2
PERF010A_SOURCE_IDENTITY_SMOKE_WORKLOAD_RESULT=PASS
PERF010A_SOURCE_IDENTITY_OBSERVED_RECORDS=23

SOURCE=method-call.protos
START_OFFSET=1226
END_OFFSET=1254
LENGTH=28
LINE=29
COLUMN_ONE_BASED=23
TEXT=sink = receiver.identity(42)
ROOT_NODE_CLASS=com.guillermomolina.protos.execution.ProtosBytecodeRootNodeGen

METHOD_CALL_SOURCE_IDENTITY_VISIBLE=YES
SOURCE_IDENTITY_GATE=PASS
```

The concrete compiler root IDs were selected from this capture by source section and root class. The previous capture's `id=153` was not assumed.

## Exact caller/helper pair

```text
CALLER_SEMANTIC_ROOT=
  id=154
  ProtosSemanticBytecodeRootNodeGen@5b7a8434
  Src method-call.protos:29

CALLER_HELPER_ROOT=
  id=153
  ProtosBytecodeRootNodeGen@2ce6c6ec
  Src method-call.protos:29
```

The numeric IDs happen to match the previous capture, but that coincidence is not used as identity evidence.

## Helper compiler lifecycle

```text
Tier 1 @ 400/400
  queued
  start
  done
  IR=3588/15634

Tier 2 @ 10000/10000
  queued
  unqueued
  Reason=Stale compilation task

Tier 2 @ 10001/10000
  queued
  start
  done
  IR=3591/15131
```

No later lifecycle event for this helper appears in the complete trace.

After the successful Tier 2 compilation there is no target-helper event matching:

```text
opt deopt
Invalidated=true
uncommon trap
opt failed
PermanentBailoutException
Tier 1 recompilation
Tier 2 recompilation
```

Therefore the exact helper remains compiled for the remainder of the capture.

## Comparison with the previous cache-identity lifecycle

The preceding product revision `3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30` retained ephemeral `ProtosClosureValue` and `methodHome` object identity in the `fastOrdinarySend` PIC key. Its exact source-identified helper showed:

```text
Tier 1 IR=3588
Tier 2 IR=4284
recompiled Tier 1 IR=4971
recompiled Tier 2 IR=4972
then final Tier 1 permanent bailout
```

That lifecycle was causally attributed to fresh source-execution materializations consuming successive `limit=3` fast-specialization entries until `perform(replaces="fastOrdinarySend")` replaced the fast specialization and exposed the old generic preparation path again.

The stable-key revision instead shows:

```text
Tier 1 IR=3588
Tier 2 IR=3591
IR_GROWTH=3
HELPER_INVALIDATIONS_AFTER_TIER2=0
HELPER_RECOMPILATIONS_AFTER_TIER2=0
HELPER_PERMANENT_BAILOUT=0
```

The prior cumulative growth of approximately 700 IR nodes per fresh-cache-state transition is absent.

## Global bailout separation

The complete stderr still contains two permanent bailout failures elsewhere in the same workload:

```text
id=146
ProtosBytecodeRootNodeGen@39d9314d
Src method-call.protos:20
PermanentBailoutException: Too deep inlining

id=148
ProtosBytecodeRootNodeGen@40317ba2
Src method-call.protos:19
PermanentBailoutException: Too deep inlining
```

These failures are not the source-identified line-29 caller helper. Global occurrences of `PrepareSendArguments.perform`, standard-import preparation, or permanent-bailout text that belong to those other roots must not be attributed to `id=153`.

## Discriminator result

```text
PERF010A_STABLE_IDENTITY_LIFECYCLE_DISCRIMINATOR=ESTABLISHED

FAST_SPECIALIZATION_CACHE_CHURN=REMOVED
GENERIC_REPLACING_SPECIALIZATION_ACTIVATED=NO
CALLER_HELPER_PERMANENT_BAILOUT=REMOVED
COMPILER_LIFECYCLE_STABLE=YES

INITIAL_TIER1_COMPILATION=SUCCESS
INITIAL_TIER2_COMPILATION=SUCCESS
CALLER_HELPER_INVALIDATIONS_AFTER_STABLE_KEY=0
CALLER_HELPER_RECOMPILATIONS_AFTER_TIER2=0

TIER1_IR=3588
TIER2_IR=3591
IR_GROWTH=3

TIMING_READY=YES
TIMING_INTERPRETED_IN_THIS_SLICE=NO
```

`GENERIC_REPLACING_SPECIALIZATION_ACTIVATED=NO` is specific to the previously established cache-exhaustion transition for this exact node during this capture. The run executes far beyond the old three-entry boundary while preserving the same Source/process/Context and recreating the outer activation each iteration, yet the target helper records neither the invalidation/rewrite sequence nor the subsequent recompilation that accompanied replacement in the preceding revision.

This result does not claim that generic `PrepareSendArguments.perform` is absent from every root in the process. It establishes that the source-identified line-29 helper no longer transitions into the replacing generic specialization through the previously demonstrated fresh-Closure/home PIC-exhaustion mechanism.

## PERF010-A interpretation

The unique question for this slice was:

> Does `3e8e6b565c95eb5098c2168d241536ba13ad19e9` eliminate fresh Closure/home specialization churn and keep the exact caller helper compiled?

```text
ANSWER=YES

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
```

The lifecycle gate that previously blocked timing attribution has passed. Clean, non-diagnostic paired timing is now admissible.

## PERF011 relationship

This is direct dynamic evidence for the concrete semantic-to-compiler polymorphism-fidelity mismatch already recorded by PERF011:

```text
SEMANTIC_CALL_SITE=MONOMORPHIC
OLD_RUNTIME_SPECIALIZATION_IDENTITY=EPHEMERAL_OBJECT_IDENTITY
OLD_ACCIDENTAL_MEGAMORPHISM=ESTABLISHED
STABLE_EXECUTABLE_IDENTITY_FIX=PUBLISHED
COMPILER_LIFECYCLE_AFTER_FIX=STABLE
```

The result strengthens the call-site specialization finding without authorizing a broad Shape/DynamicObject/Frame migration.

## Next evidence slice

The next PERF010-A slice is clean paired timing, not another implementation change.

Use the retained benchmark methodology without compiler tracing and compare the pre-intervention control with the stable executable-identity product revision on the direct call workloads that exercise this path, at minimum:

```text
micro/method-call
runtime/monomorphic-dispatch
```

Correctness and exact image/revision identity must gate timing. Retain raw paired samples and quantify uncertainty/dispersion. Do not use the diagnostic lifecycle timing arrays from this record.

The output must determine whether the now-stable call-site intervention recovers a material fraction of the common overhead and whether the evidence is sufficient to classify the Pareto-leading cause required by #691.
