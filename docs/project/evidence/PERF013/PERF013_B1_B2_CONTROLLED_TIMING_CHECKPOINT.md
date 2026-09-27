# PERF013 B1 → B2 controlled timing checkpoint

Date: 2026-09-27

## Scope

This record retains the controlled B1→B2 timing checkpoint required for
PERF013 / #724 after the captured-read and captured-write compiler causal gates
had already passed.

```text
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_B1_REVISION=c498f35a383447c21fa0b63c3414857e777e5300
PROTOS_B2_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=2cc6cfe1216b154b629de77c8321753de51b857c
CPU_AFFINITY=15

JDK_VERSION=25.0.4.1
GRAALVM_TRUFFLE_VERSION=25.3.4.1
```

`CPU_AFFINITY=15` is the logical CPU selected by the timing harness; it is not
a CPU-model claim.

## Harness interpretation

The retained comparator defines a positive `paired_control_effect` as B2 being
faster than B1 after discounting movement in the corresponding workload-control
pair.

Maintainer-reported timing configuration:

```text
WARMUP_ITERATIONS=120
STEADY_ITERATIONS=100
OPERATION_COUNT=10000
BLOCK_COUNT=4
BLOCK_ORDER=A,B,A,B
```

The four workloads are deliberately split between primary call/dispatch paths
and negative controls:

```text
NEGATIVE_CONTROLS:
  micro/slot-read
  micro/closure-call

PRIMARY_WORKLOADS:
  micro/method-call
  runtime/monomorphic-dispatch
```

## Result

| Workload | Paired control effect | MAD | Classification |
|---|---:|---:|---|
| `micro/slot-read` | -2.6010% | 1.44% | no improvement |
| `micro/closure-call` | -0.3920% | 2.82% | essentially unchanged |
| `micro/method-call` | +18.8301% | 2.44% | material improvement |
| `runtime/monomorphic-dispatch` | +22.9525% | 1.39% | clear material improvement |

For all four workloads:

```text
ORDER_EFFECT=NOT_DETECTED
```

The observed pattern is selective rather than host-wide:

```text
slot-read             ≈ unchanged
closure-call          ≈ unchanged
method-call           +18.8301%
monomorphic-dispatch  +22.9525%
```

That pattern is materially stronger causal evidence than a uniform shift across
all workloads.

## Dispersion

The `micro/method-call` result is materially positive but less uniform than the
dispatch result:

```text
METHOD_CALL_MIN=-1.26%
METHOD_CALL_MEDIAN=+18.8301%
METHOD_CALL_MAX=+23.08%
METHOD_CALL_MAD=2.44%
```

One of the four method-call blocks was therefore effectively neutral. The
checkpoint retains the material median effect while preserving that dispersion
instead of treating the workload as uniformly improved.

For `runtime/monomorphic-dispatch`, all four reported block effects were
positive and lay between:

```text
MONOMORPHIC_DISPATCH_BLOCK_MIN=+15.79%
MONOMORPHIC_DISPATCH_BLOCK_MAX=+24.46%
MONOMORPHIC_DISPATCH_MEDIAN=+22.9525%
MONOMORPHIC_DISPATCH_MAD=1.39%
```

This makes the dispatch result particularly robust against a single-block
outlier explanation.

## Structural compiler result already established

The timing checkpoint follows the retained B1 and B2 compiler causal gates.

After B1:

```text
CAPTURED_READ_PE_FAILURE=ABSENT
CAPTURED_WRITE_PE_FAILURE=PRESENT
```

After B2:

```text
CAPTURED_READ_PE_FAILURE=ABSENT
CAPTURED_WRITE_PE_FAILURE=ABSENT
OLD_READ_FAILURE_STACK=ABSENT
OLD_WRITE_FAILURE_STACK=ABSENT
PERF013_B2_COMPILER_GATE=PASS
```

The write root that previously failed on the captured-local PE-constant check
was positively exercised after B2 and proceeded substantially farther before
reaching the independently known `Too deep inlining` failure class.

## Causal interpretation

The combined structural and timing evidence supports the bounded PERF013 claim:

```text
B2 removes the captured-write PE-constant bailout
AND
B2 produces a controlled ~19-23% improvement
on the two primary method-call / monomorphic-dispatch workloads.
```

The result does **not** establish that PERF013 explains the historical overall
Protos performance gap, nor that the remaining `Too deep inlining` failures are
the sole residual cause.

The unchanged `micro/closure-call` result is informative rather than a
falsation of B2: the structural bailout used to expose the defect disappears,
but that workload remains dominated by another cost. The already-observed
`Too deep inlining` class remains a concrete residual candidate to investigate
separately.

## Cross-generation boundary

Slice C established that the B1/B2 fast path is valid only when the accessor and
captured frame belong to the same generated `BytecodeRootNodes` generation.
Cross-generation / isolated rematerialization must retain the runtime lexical
authority fallback.

```text
SAME_GENERATION_MATERIALIZED_LOCAL_ACCESSOR=RETAINED
CROSS_GENERATION_RUNTIME_AUTHORITY_FALLBACK=RETAINED
CROSS_GENERATION_ACCESSOR_FRAME_MIXING=FORBIDDEN
PERF013_C_PRODUCT_CHANGE=NOT_APPLICABLE
```

## PERF013 timing checkpoint

```text
PROTOS_B1_REVISION=c498f35a383447c21fa0b63c3414857e777e5300
PROTOS_B2_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a

SLOT_READ_EFFECT=-2.6010%
CLOSURE_CALL_EFFECT=-0.3920%
METHOD_CALL_EFFECT=+18.8301%
MONOMORPHIC_DISPATCH_EFFECT=+22.9525%

ORDER_EFFECT=NOT_DETECTED
B2_TIMING_EFFECT=MATERIAL_IMPROVEMENT
PERF013_B2_CONTROLLED_TIMING_CHECKPOINT=PASS
```

## Closure relevance

Together with the previously retained implementation, compiler-gate,
cross-generation, and integrated-validation records, this timing checkpoint
satisfies the final PERF013-specific timing retention requirement.

The remaining performance investigation belongs to PERF010-B rather than a new
PERF013 product slice.

Relevant durable records:

- `docs/project/evidence/PERF013/PERF013_SLICE_B1_MATERIALIZED_CAPTURED_READ_IMPLEMENTATION.md`
- `docs/project/evidence/PERF013/PERF013_SLICE_B1_POST_PUBLICATION_COMPILER_GATE.md`
- `docs/project/evidence/PERF013/PERF013_SLICE_B2_MATERIALIZED_CAPTURED_WRITE_IMPLEMENTATION.md`
- `docs/project/evidence/PERF013/PERF013_SLICE_B2_POST_PUBLICATION_COMPILER_GATE.md`
- `docs/project/evidence/PERF013/PERF013_SLICE_C_CROSS_GENERATION_PLATFORM_BOUNDARY.md`
