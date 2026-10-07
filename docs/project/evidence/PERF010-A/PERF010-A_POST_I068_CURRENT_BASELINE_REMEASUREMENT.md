# PERF010-A — Post-I068 current baseline remeasurement

Status: **BASELINE ESTABLISHED; HISTORICAL MOVEMENT DESCRIPTIVE ONLY; CAUSAL ATTRIBUTION NOT ESTABLISHED**

This durable, non-normative evidence record retains the cross-language
steady-state baseline measured after completion of I068 / PLAT036 Candidate D.
It does not change Protos semantics, reopen I068, attribute the observed
movement to I068, select a production optimization, or authorize the lazy
execution-context materialization alternative considered by PLAT037.

## Evidence identity

```text
PERF_ITEM=PERF010-A
MEASUREMENT_SLICE=PERF010A_POST_I068_CURRENT_BASELINE_REMEASUREMENT

PROTOS_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT

I068_FINAL_PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
I068_FINAL_PROJECT_RECORD_REVISION=8f462172c4a84cc9099fde1737a2957ccbfdc66e
I068_STATUS=COMPLETE

BENCHMARK_HARNESS_REVISION=d6d96f22775a743f604c1645731fa7ebf0ac7ca4
BENCHMARK_RESULT_REVISION=bdb69e2e5135f95a0b41fd258f8afd8d8ca3d7df
BENCHMARK_RESULTS_PATH=results/perf010a-post-i068-baseline

HISTORICAL_COMPARISON_SOURCE=results/perf004-a
HISTORICAL_HARNESS_REVISION=60dbce7faf5510bd1bd6867a866aa7ca69c48637
HISTORICAL_PROTOS_REVISION=4a03efc15620b37b2e418b3df30b4a26486446ec
HISTORICAL_PROTOS_VERSION=0.2.492-SNAPSHOT
HISTORICAL_COMPARISON_CAUSAL=NO
```

The retained benchmark result is in
`guillermomolina/protos-benchmarks@bdb69e2e5135f95a0b41fd258f8afd8d8ca3d7df`.
Its authoritative raw ordered samples are under the results path above.

## Measurement contract

The reference run used:

- five persistent forks per language/workload;
- 120 warmup iterations per fork;
- 100 steady ordered samples per fork;
- the steady median as the primary statistic;
- one explicit CPU via cpuset;
- the same host for Protos, Python and Node;
- network disabled;
- JFR, compiler tracing, IGV and allocation instrumentation disabled during timing;
- correctness before timing, reported as 15/15 PASS.

Runtime and host identity:

```text
HOST_CPU=AMD Ryzen 7 3700X 8-Core Processor
HOST_ARCH=x86_64
HOST_KERNEL=7.2.6-zen2-1-zen
CPUSET=0
DOCKER_SERVER_VERSION=29.8.1

PROTOS_RUNTIME=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
GRAALVM_RELEASE=25.3.4.1
GRAAL_TRUFFLE_VERSION=25.3.4.1
JDK_VERSION=25.0.4.1
PYTHON_VERSION=3.14.7
NODE_VERSION=24.20.0
```

The five Protos workload blobs are identical to the retained PERF004-A
workloads:

```text
micro/slot-read=c90f22ddc4bce857a62772e88cf44b38d0a5147a
micro/closure-call=5ed62d6e67e2ec02195383987ca3f367c18ac947
micro/method-call=630ed821cc9b62c71f843859fb16b963414a0f52
runtime/monomorphic-dispatch=9ba70de276572f11771de4d98ebbed165e8892b2
algorithms/factorial/recursive=0e6b2ef5f1e3d293e4cb934632b6d15d35ba33b5
```

## Primary common-path result

| workload | historical Protos/Node | current Protos/Node | historical Protos/Python | current Protos/Python |
|---|---:|---:|---:|---:|
| `micro/slot-read` | 424.762469x | 1022.793547x | 31.245163x | 38.628826x |
| `micro/closure-call` | 773.591568x | 1615.329446x | 38.250756x | 50.861539x |
| `micro/method-call` | 438.509381x | 1385.502659x | 31.663089x | 49.722209x |
| `runtime/monomorphic-dispatch` | 444.984786x | 1290.639089x | 31.169229x | 47.481349x |

The current primary common-path ranges are:

```text
POST_I068_COMMON_PROTOS_OVER_NODE_RANGE=1022.793547..1615.329446
POST_I068_COMMON_PROTOS_OVER_PYTHON_RANGE=38.628826..50.861539
```

No one-number aggregate is substituted for these per-workload ranges.

## Absolute-median movement

Historical and current absolute steady medians make the denominator movement
visible:

| workload | historical Protos ns | current Protos ns | Protos movement | historical Python ns | current Python ns | Python movement | historical Node ns | current Node ns | Node movement |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `micro/slot-read` | 49,378,637 | 60,626,087.5 | +22.8% | 1,580,361 | 1,569,452 | -0.7% | 116,250 | 59,275 | -49.0% |
| `micro/closure-call` | 67,611,903 | 88,019,301.5 | +30.2% | 1,767,596.5 | 1,730,567 | -2.1% | 87,400 | 54,490 | -37.7% |
| `micro/method-call` | 55,565,935.5 | 87,286,667.5 | +57.1% | 1,754,912 | 1,755,486.5 | +0.0% | 126,715.5 | 63,000 | -50.3% |
| `runtime/monomorphic-dispatch` | 54,842,150 | 84,427,156 | +53.9% | 1,759,496.5 | 1,778,112 | +1.1% | 123,245 | 65,415 | -46.9% |

For the four primary workloads, the current Protos median is descriptively
22.8%-57.1% higher than PERF004-A. Python remains within approximately -2.1%
to +1.1%, while the Node denominator is approximately 37.7%-50.3% lower.

This explains why the current Protos/Node ratios move more strongly than the
Protos/Python ratios. It does **not** establish why Protos itself moved: the
comparison spans many product revisions and a later retained measurement
harness, so the historical comparison remains explicitly non-causal.

## Recursive sentinel

`algorithms/factorial/recursive` remains a secondary recursive scale sentinel,
not part of the common-overhead attribution.

| metric | historical | current |
|---|---:|---:|
| Protos median | 5,014,013.5 ns | 4,820,484 ns |
| Python median | 1,320 ns | 1,300 ns |
| Node median | 1,040 ns | 395 ns |
| Protos/Python | 3798.495076x | 3708.064615x |
| Protos/Node | 4821.166827x | 12203.756962x |

Here Protos is descriptively about 3.9% lower than the historical median and
Python about 1.5% lower, while Node is about 62.0% lower. The much larger
Protos/Node ratio is therefore strongly affected by denominator movement.

## Interpretation boundary

The post-I068 baseline establishes only the current performance state.

For the four primary common-path workloads:

```text
OBSERVED_CURRENT_PROTOS_ABSOLUTE_IMPROVEMENT_VS_PERF004A=NO
OBSERVED_CURRENT_PROTOS_OVER_PYTHON_RATIO_IMPROVEMENT=NO
OBSERVED_CURRENT_PROTOS_OVER_NODE_RATIO_IMPROVEMENT=NO
```

Those are descriptive observations, not causal findings. In particular:

- `I068_ATTRIBUTABLE_FRACTION` remains `NOT_ESTABLISHED`;
- the evidence does not prove that I068 caused the higher current medians;
- the evidence does not establish that reverting I068 would recover cost;
- the evidence does not establish eager context materialization as the next
  dominant cause;
- PLAT037 Candidate B is not activated by this result;
- no production optimization is selected.

The result therefore closes the missing post-I068 baseline measurement but does
not close PERF010-A's dominant-cost question. Further work should avoid treating
one newly suspected mechanism as established merely because the previous
candidate did not produce an observed cross-version improvement. A causal cost
decomposition remains necessary before selecting another production
optimization.

## Machine-readable conclusion

```text
PERF010A_POST_I068_BASELINE=ESTABLISHED
PRODUCT_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
BENCHMARK_RESULT_REVISION=bdb69e2e5135f95a0b41fd258f8afd8d8ca3d7df

POST_I068_COMMON_GAP_SCALE=PER_WORKLOAD_RANGES_ONLY
POST_I068_COMMON_PROTOS_OVER_NODE_RANGE=1022.793547..1615.329446
POST_I068_COMMON_PROTOS_OVER_PYTHON_RANGE=38.628826..50.861539
POST_I068_FACTORIAL_PROTOS_OVER_NODE=12203.756962
POST_I068_FACTORIAL_PROTOS_OVER_PYTHON=3708.064615

HISTORICAL_COMPARISON_SOURCE=results/perf004-a
HISTORICAL_COMPARISON_CAUSAL=NO
OBSERVED_CURRENT_PRIMARY_PROTOS_MEDIAN_MOVEMENT=+22.8%..+57.1%
OBSERVED_CURRENT_PRIMARY_PYTHON_MEDIAN_MOVEMENT=-2.1%..+1.1%
OBSERVED_CURRENT_PRIMARY_NODE_MEDIAN_MOVEMENT=-50.3%..-37.7%

I068_ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```
