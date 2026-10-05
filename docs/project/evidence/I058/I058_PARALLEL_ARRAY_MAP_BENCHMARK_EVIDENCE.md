# I058 — canonical parallel-array-map benchmark evidence

Date: 2026-10-05

This snapshot records the performance evidence required by I058 /
`guillermomolina/protos#661` after D160 Candidate B moved the five parallel
Array algorithms from Core into `std:collections/Array`.

It supplements the earlier implementation and CI-blocker snapshots. It does not
rewrite those historical observations.

## Stable identities

```text
FORMAL_WORK_ITEM=I058
GITHUB_ISSUE=guillermomolina/protos#661
BENCHMARK_REVISION=564dc97aacb593826011a8876554d69dd6529faa
BENCHMARK_VERSION=0.3.206-SNAPSHOT
BENCHMARK=protos/benchmarks/concurrency/parallel-array-map.protos
BENCHMARK_SOURCE_SHA256=ec0b37b3d2f6c183a11c124181bce2dab9fa74185c2033102f514309e1e2c0b2
```

## Environment

```text
HOST=VMware guest
OS=Linux 7.2.8-arch1-2
ARCH=x86_64
CPU=Xeon Platinum 8362
AVAILABLE_CPUS=16
VCPUS_PER_CORE_REPORTED=1
NUMA_NODES=1
JAVA=OpenJDK 25.0.4.1.1
GRAALVM=GraalVM CE 25.4.4.1.1+1.1
AFFINITY_TOOL=taskset
```

The canonical source uses `Arrays.parallelMap` from
`std:collections/Array`, so the benchmark exercises the D160/I058 placement.

## Measurement method

The current run used fresh JVM processes through `bin/protos` under explicit
CPU affinity:

```text
MEASURED_WIDTHS=1,2,4,8
CPUSETS=1:0,2:0-1,4:0-3,8:0-7
WARMUP_ITERATIONS=20
STEADY_SAMPLES=20
```

Each current sample is process wall time, so it includes JVM startup, Truffle
warmup, and JIT activity.

PERF001-F used repeated `run()` measurements in one long-lived process.
Absolute timing comparison is therefore not methodologically valid. Different
host/JVM/GraalVM identities further prevent an absolute before/after timing
claim.

Scaling shape is still useful with this qualification because every current
width was measured with the same current method and fixed logical workload.

## Current results

| Width | CPU set | Median (s) | MAD (s) | p95 (s) | Min (s) | Max (s) | Speedup | Efficiency |
| ---: | :--- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 1 | 0 | 7.533 | 0.634 | 8.934 | 4.303 | 8.966 | 1.000 | 1.000 |
| 2 | 0-1 | 4.392 | 0.790 | 6.133 | 2.359 | 6.239 | 1.715 | 0.857 |
| 4 | 0-3 | 2.961 | 0.495 | 4.066 | 2.059 | 4.395 | 2.544 | 0.636 |
| 8 | 0-7 | 1.684 | 0.170 | 2.841 | 1.468 | 2.992 | 4.473 | 0.559 |

Noise is material: MAD is roughly 8–18% of the median and every width has a
long tail. Width-1 nevertheless reproduced closely across the two attempts
(7.556 s and 7.533 s median).

## PERF001-F reference comparison

The retained PERF001-F scaling reference was:

```text
REFERENCE_PROTOS_REVISION=0372a58addc63f305c911811659edd9b2b508420
REFERENCE_SPEEDUP=1:1.000,2:1.397,4:2.582,8:4.168
REFERENCE_EFFICIENCY=1:1.000,2:0.699,4:0.645,8:0.521
```

| Width | Current speedup | PERF001-F speedup | Speedup delta | Current efficiency | PERF001-F efficiency | Efficiency delta |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| 2 | 1.715 | 1.397 | +22.8% | 0.857 | 0.699 | +22.6% |
| 4 | 2.544 | 2.582 | -1.5% | 0.636 | 0.645 | -1.4% |
| 8 | 4.473 | 4.168 | +7.3% | 0.559 | 0.521 | +7.3% |

Width 4 is effectively unchanged relative to the reference within the observed
noise, while widths 2 and 8 are higher. The measurements therefore provide no
evidence that D160's move to source-backed Standard Library composition caused a
material scaling regression.

```text
ABSOLUTE_TIMING_COMPARABLE=NO
SCALING_SHAPE_COMPARABLE=YES_WITH_QUALIFICATION
MATERIAL_PERFORMANCE_REGRESSION=NO
I058_PERFORMANCE_CLOSURE_GATE=PASS
```

## Liveness anomaly discovered during measurement

One width-2 warmup execution in the first attempt remained live beyond 300
seconds. It was not reproduced in roughly 200 immediately subsequent executions,
including a complete 160-run rerun with automatic hang capture enabled.

BUG017/#800 later reproduced and explained the hang class: an unexpected host
exception on a P carrier could escape before settling its Completion, leaving the
producer Task and Future pending indefinitely.

BUG017's liveness containment is published independently at exact Protos revision
`a85d9ca846d7408f869916a59b72f02ee12b9222`.

The host exception that triggered the reproduced BUG017 failure was a
`FrameSlotTypeException` in the captured-materialized lexical read path. That
guest-execution correctness defect remains independently tracked as BUG018/#801.

The anomaly therefore does not invalidate the performance result, but I058 must
not claim its retained P/Future correctness/liveness path fully green while
BUG018 remains capable of failing the canonical workload.

## Current I058 closure state

```text
I058_PRODUCT_COMPLETE=YES
I058_PERFORMANCE_EVIDENCE=PASS
I058_PERFORMANCE_CLOSURE_GATE=PASS
BUG017_LIVENESS_CONTAINMENT=PUBLISHED
BUG018_EXECUTION_DEFECT=OPEN
I058_CORRECTNESS_LIVENESS_GATE=BLOCKED
I058_READY_TO_CLOSE=NO
```

AI assistance: this evidence record was drafted with ChatGPT from the
maintainer-provided benchmark measurements and BUG017 diagnostic result, plus
exact published Protos revisions and live Issue state.
