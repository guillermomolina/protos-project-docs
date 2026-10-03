# PERF025 final benchmark and closure

## Scope

This record closes the final benchmarking/bookkeeping phase of
`PERF025 / guillermomolina/protos#758` after all non-benchmark product work
completed at Protos `0.3.169-SNAPSHOT`.

It preserves three distinct evidence classes without conflating them:

1. the one-shot exact PRE_A -> FINAL reusable-call reference attempt;
2. an endpoint `make test-protos` observation at the same PERF025 start/final
   product authorities; and
3. the admitted final current-state cross-Truffle primitive comparison.

The exact PRE_A -> FINAL reference was **not admitted** and was not retried.
Its observed magnitudes are retained here only as rejected diagnostic output,
not as an admitted reference claim.

## Exact authorities

```text
PERF025_ISSUE=guillermomolina/protos#758

PRE_A_REVISION=f3c44554ddfb9004c43dbde5197b9805990f8a4a
PRE_A_VERSION=0.3.129-SNAPSHOT
PRE_A_RUN_MODE=dynamic

FINAL_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
FINAL_VERSION=0.3.169-SNAPSHOT
FINAL_RUN_MODE=prepared

VERSION_INDEPENDENT_HARNESS_REVISION=aebaaa737d8d162f8e96859dc63e3077177bd821
PER_LANGUAGE_SAMPLE_SIZING_REVISION=dfc2a34dacc40e57e2e625789c5d3300615bc979
```

The version-independent harness makes the selected Protos checkout and
dynamic/prepared execution mode part of the generic JVM measurement authority.
The later `dfc2a34d` harness change adds only an explicit per-language
`sample_calls` override to make tiny cross-language samples long enough for
the existing bounded steady-state gate.

## 1. Exact PRE_A -> FINAL reusable-call reference attempt

Maintainer-executed policy:

```text
CPU=0
WORKLOADS=
  primitive-return-literal
  primitive-closure-call
  primitive-method-call
SAMPLE_CALLS=10000
WARMUP=50
STEADY=10
ADMISSION_SCOPE=steady-only
REFERENCE_RETRY=NO
```

Observed p50 amortized values:

| workload | PRE_A ns/call | FINAL ns/call | observed delta | PRE_A admission | FINAL admission |
| --- | ---: | ---: | ---: | --- | --- |
| `primitive-return-literal` | 6852.4 | 564.3 | -91.77% | PASS | NOT_STABLE |
| `primitive-closure-call` | 8462.2 | 1391.0 | -83.56% | PASS | PASS |
| `primitive-method-call` | 7466.5 | 951.9 | -87.25% | PASS | PASS |

The rejected case was FINAL `primitive-return-literal`. Its steady-state
median drift and MAD checks were within limits, but the internal-shape gate
reported:

```text
steady_largest_internal_gap_pct=21.42
STABILITY_MAX_INTERNAL_GAP_PCT=20.0
reference_admission=NOT_STABLE
```

Therefore:

```text
EXACT_PRE_A_TO_FINAL_REFERENCE=REJECTED_NOT_STABLE
REFERENCE_RETRY=NO
ADMITTED_PERF025_FINAL_DELTA_CLAIM=NO
```

The observed -91.77% / -83.56% / -87.25% magnitudes describe the rejected
one-shot output only. They are not promoted into admitted reference evidence.

The raw files for this attempt remained under the maintainer's disposable
`results/local/` tree and are not represented here as benchmark-repository
retained raw evidence.

## 2. PERF025 start/final Test Tool endpoint observation

The maintainer then executed the ordinary repository target with
`PROTOS_TEST_JOBS=16` at the two endpoint revisions.

```text
PRE_A:
  revision=f3c44554ddfb9004c43dbde5197b9805990f8a4a
  tests=1264 passed, 0 failed
  Protos tests total time=86 s

FINAL:
  revision=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
  tests=1284 passed, 0 failed
  Protos tests total time=50 s
```

Derived endpoint observation:

```text
WALL_DELTA=-36 s
WALL_DELTA_PCT=-41.86%
PRE_A_TO_FINAL_SPEEDUP=1.72x
```

This is a descriptive endpoint comparison, not a single-change causal A/B:
the final corpus contains 20 additional tests and the product interval includes
the complete accumulated PERF025 line. It is useful evidence that the final
state did not merely improve reusable-call microbenchmarks while regressing the
ordinary Protos Test Tool wall time.

## 3. Final admitted current-state cross-Truffle primitive comparison

After the generic version-independent harness was published, the first
`sample_calls=100` `primitive-return-literal` run was correctly rejected as
undersized: the ~0.44 ms Protos samples were dominated by millisecond-scale
outliers.

The bounded fix in
`guillermomolina/protos-benchmarks@dfc2a34dacc40e57e2e625789c5d3300615bc979`
allows explicit per-language sample sizing without changing runners, product
identity, warmup/steady policy, or stability thresholds.

Final current-state policy:

```text
PROTOS_REVISION=cfc0fb433e82f0478c9fff9cc965c3fc506fabc9
PROTOS_RUN_MODE=prepared
CPU=0
REFERENCE_WARMUP=60
REFERENCE_STEADY=10

JVM_SAMPLE_CALLS_PROTOS=100000
JVM_SAMPLE_CALLS_JS=500000
JVM_SAMPLE_CALLS_PYTHON=500000
```

All nine language/workload observations passed the published benchmark
steady-state admission gate.

| workload | Protos ns/call | GraalJS ns/call | GraalPy ns/call | Protos/JS | Protos/Py |
| --- | ---: | ---: | ---: | ---: | ---: |
| `primitive-return-literal` | 216.56 | 43.90 | 46.92 | 4.93x | 4.62x |
| `primitive-closure-call` | 751.15 | 40.22 | 44.71 | 18.68x | 16.80x |
| `primitive-method-call` | 359.18 | 41.56 | 51.86 | 8.64x | 6.93x |

These are current-state cross-language microbenchmark observations, not
whole-language rankings.

The smallest current basal floor among the three Protos workloads is the
literal-return path. Relative to that floor, the Protos current-state residual
is approximately:

```text
closure-call minus return-literal ~= 534.59 ns/call
method-call  minus return-literal ~= 142.62 ns/call
```

The dominant residual in this three-workload radar is therefore the
`primitive-closure-call` path.

## Closure interpretation

PERF025 achieved the intended product-line transition from the original
dynamic reusable embedding path through prepared execution and the subsequent
pay-as-you-grow/runtime specialization work, while retaining semantics and the
existing historical admitted references for their exact revisions.

The final exact PRE_A -> FINAL reference attempt is deliberately preserved as
rejected and non-retried. Closure does not reinterpret that result as PASS.

Project-owner direction on 2026-10-03 is to close PERF025 after completing the
final benchmarking campaign and preserve the remaining cross-Truffle residual
as separate diagnostic work.

```text
PERF025_NON_BENCHMARK_PRODUCT_WORK=COMPLETE
EXACT_FINAL_REFERENCE=REJECTED_NOT_STABLE
EXACT_FINAL_REFERENCE_RETRY=NO
CURRENT_CROSS_TRUFFLE_PRIMITIVE_MATRIX=9_OF_9_ADMISSION_PASS
TEST_TOOL_ENDPOINT_OBSERVATION=86_S_TO_50_S
PERF025_STATUS=CLOSED_BY_OWNER_DIRECTION

RESIDUAL_FOCUS=primitive-closure-call
RESIDUAL_OWNER=PERF024/#756
NEXT_WORK_TYPE=INVESTIGATION
NEW_PRODUCT_IMPLEMENTATION_AUTHORIZED=NO
```

PERF024 remains the correct owner for the residual investigation because it
already owns cross-Truffle primitive decomposition and focused JFR/IGV
discrimination without authorizing a Protos product optimization.
