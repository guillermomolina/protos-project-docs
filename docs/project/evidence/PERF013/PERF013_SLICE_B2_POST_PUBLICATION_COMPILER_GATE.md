# PERF013 Slice B2 — post-publication compiler causal gate

Date: 2026-09-27

## Target

```text
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
PRODUCT_VERSION=0.3.102-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
GATE=PERF013_B2_POST_PUBLICATION_COMPILER_CAUSAL_GATE
```

## Maintainer-reported result

The maintainer reran the bounded `micro/closure-call.protos`
TraceCompilation gate used for PERF013 B1 and reported:

```text
TARGET_PROTOS_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a

CAPTURED_READ_PE_FAILURE=ABSENT
CAPTURED_WRITE_PE_FAILURE=ABSENT

OLD_READ_FAILURE_STACK=ABSENT
OLD_WRITE_FAILURE_STACK=ABSENT

TOO_DEEP_INLINING_FAILURES=PRESENT

PERF013_B2_COMPILER_GATE=PASS
```

The B2 trace contained zero occurrences of the B1/B2 PE-constant failure family:

```text
CompilerAsserts.partialEvaluationConstant
LocalRangeAccessor.isCleared
ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
ResolveCapturedWritableLexicalTarget
Partial evaluation did not reduce value to a constant
```

No source-equivalent captured-read PE-constant bailout appeared either.

## Positive path-exercise evidence

The absence is not classified as a vacuous PASS.

The maintainer reported that the B1 and B2 runs expose the same eight compiler
root ids, 146 through 153, with the same class-shape population.

In the B1 trace, root id 152 (`ProtosBytecodeRootNodeGen`) was the captured-write
PE-constant failure and aborted after about 35 ms.

In the B2 trace, the corresponding id 152 compilation proceeded for about
476 ms before failing for a different reason:

```text
PermanentBailoutException:
Too deep inlining, probably caused by recursive inlining.
```

This establishes that the relevant root was still exercised and progressed
past the old captured-write PE-constant obstacle.

```text
WRITE_PATH_EXERCISED=PASS
OLD_WRITE_PE_CONSTANT_OBSTACLE_CLEARED=PASS
```

## Too-deep-inlining nuance

The B2 id-152 root now reaches a previously known compiler-failure class:
`Too deep inlining`.

The maintainer correlated this root with the Closure body:

```text
() => { sink = identity(42) }
```

and reported that after clearing the captured-write authority check, partial
evaluation proceeds into immediate method-call / Closure-call preparation plus
module-resolver machinery before reaching the same known inlining ceiling.

The total number of too-deep-inlining failures changed:

```text
B1_TOO_DEEP_INLINING_FAILURES=3
B2_TOO_DEEP_INLINING_FAILURES=4
```

The other three failures (ids 146, 148 and 150) were reported unchanged in
content; id 152 is a new occurrence of the already-known failure class exposed
only after the earlier write-specific PE-constant barrier was removed.

This is not reclassified as a PERF013-B2 failure because the declared B2
discriminator is the captured-local PE-constant path, not elimination of all
compiler bailouts.

```text
NEW_B2_FAILURE_CATEGORY=NO
B2_DISCRIMINATOR_SATISFIED=YES
TOO_DEEP_INLINING_OUT_OF_SCOPE=YES
```

## Evidence location

The maintainer reported the raw local files at:

```text
results/perf013-b2-gate/closure-call-trace.stdout.log
results/perf013-b2-gate/closure-call-trace.stderr.log
```

with the retained B1 files left untouched under:

```text
results/perf013-b1-gate/
```

At the time of this durable project-record publication the B2 raw trace files
had not yet been committed to `guillermomolina/protos-benchmarks`. Therefore
this record does not claim an immutable benchmark-repository revision for those
raw files.

## Causal conclusion

The PERF013 sequence now establishes:

```text
BEFORE_B1:
  captured read  -> PE-constant bailout
  captured write -> PE-constant bailout

AFTER_B1:
  captured read  -> old bailout absent
  captured write -> old bailout present

AFTER_B2:
  captured read  -> old bailout absent
  captured write -> old bailout absent
```

The B2 intervention therefore satisfies its predicted structural compiler
effect.

## Routing

```text
PERF013_B1_COMPILER_GATE=PASS
PERF013_B2_COMPILER_GATE=PASS

PERF013_STATUS=IN_PROGRESS
NEXT_SLICE=PERF013-C_CONTEXT_LOCAL_GROUP_REMATERIALIZATION
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

PERF010_B_STEP2=STILL_BLOCKED
```

Slice C owns the remaining isolated/context-local execution-plan rebuild path
that currently lowers one Closure independently and therefore cannot provide
the declaring owner's BytecodeLocal to the B1/B2 MaterializedLocalAccessor path.
