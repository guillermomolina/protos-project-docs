# PERF013 Slice B1 — post-publication compiler causal gate

Date: 2026-09-27

## Target

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=c498f35a383447c21fa0b63c3414857e777e5300
PRODUCT_VERSION=0.3.101-SNAPSHOT

WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722
GATE=PERF013_B1_POST_PUBLICATION_COMPILER_CAUSAL_GATE
```

The gate used the existing `guillermomolina/protos-benchmarks` PERF010-A/PERF006-D3
container and `Perf006dPersistentDriver` TraceCompilation pattern against
`micro/closure-call.protos`.

The maintainer reported a successful bounded execution:

```text
DOCKER_BUILD=PASS
TRACE_EXECUTION_EXIT=0
CPU=0
```

The raw trace files were produced locally under:

```text
results/perf013-b1-gate/closure-call-trace.stdout.log
results/perf013-b1-gate/closure-call-trace.stderr.log
```

Those raw files were not published to the benchmark repository at this checkpoint, so this project
record does not claim an immutable benchmark-artifact identity.

## Captured-read result

The previous source-equivalent captured-read failure was:

```text
CompilerAsserts.partialEvaluationConstant
 -> LocalRangeAccessor.isCleared
 -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
 -> ReadCapturedFrameLocal.perform
```

In the post-B1 trace, no captured-read partial-evaluation-constant bailout is present.

The trace does contain the expected captured-write failure, proving that the grep/capture is seeing
the relevant failure class rather than simply suppressing compiler errors.

Therefore:

```text
CAPTURED_READ_PE_FAILURE=ABSENT
PERF013_B1_COMPILER_GATE=PASS
```

This is the causal prediction B1 was required to satisfy.

## Captured-write result

The trace contains one explicit partial-evaluation-constant captured-write failure:

```text
opt failed ... ProtosBytecodeRootNodeGen
Reason: Partial evaluation did not reduce value to a constant

CompilerAsserts.partialEvaluationConstant
 -> LocalRangeAccessor.isCleared
 -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
 -> ProtosBytecodeRootNode$ResolveCapturedWritableLexicalTarget.perform
```

Therefore:

```text
CAPTURED_WRITE_PE_FAILURE=PRESENT
```

This is expected before PERF013-B2 and does not invalidate B1.

## Other compiler failures

The same bounded trace still contains independent failures classified as:

```text
PermanentBailoutException: Too deep inlining, probably caused by recursive inlining.
```

They occur on other bytecode roots and are not reclassified as captured-read failures.

Their presence means this gate establishes only the specific PERF013-B1 read result. It does not
claim that the complete closure-call workload is now bailout-free or fully optimized.

```text
ALL_OPT_FAILED_ABSENT=NO
B1_SPECIFIC_CAUSAL_GATE=PASS
```

## Causal conclusion

The experiment distinguishes the B1 intervention exactly as intended:

```text
BEFORE_B1:
  captured read  -> runtime LocalRangeAccessor metadata -> PE constant bailout
  captured write -> runtime LocalRangeAccessor metadata -> PE constant bailout

AFTER_B1:
  captured read  -> MaterializedLocalAccessor          -> old PE constant bailout absent
  captured write -> runtime LocalRangeAccessor metadata -> PE constant bailout still present
```

This strongly supports the PERF013 mechanism diagnosis for captured reads and activates the
symmetric write migration.

## Routing

```text
PERF013_SLICE_B1=CAUSALLY_ACCEPTED
NEXT_SLICE=PERF013-B2_MATERIALIZED_CAPTURED_WRITE
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

PERF013_STATUS=IN_PROGRESS
PERF010_B_STEP2=STILL_BLOCKED
```

B2 must preserve destination-before-RHS semantics while moving same-group captured-write physical
local identity to a Bytecode DSL-generated `MaterializedLocalAccessor`.

The isolated/context-local rebuild fallback remains later PERF013 Slice C if still required after
B2.
