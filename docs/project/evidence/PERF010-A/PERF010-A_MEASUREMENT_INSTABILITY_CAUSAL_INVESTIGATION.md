# PERF010-A — Measurement-instability causal investigation

Status: PARTIALLY ESTABLISHED — NEXT METHODOLOGICAL INTERVENTION ESTABLISHED

This durable, non-normative record reconciles the retained PERF010-A no-op
measurement-discrimination reference with the actual harness lifecycle and the
retained raw timing series.

It does not change the historical measurements retained by the earlier
measurement-discrimination record. It revises only the methodological
interpretation of what experiment should come next.

## Evidence identity

```text
PROTOS_REVISION=4c4aa95a5852119bd280ceb40483871d5d2cbb82
BENCHMARK_HARNESS_REVISION=57cb307baad3346758d34aa3da1eda01997a4208
BENCHMARK_REFERENCE_RETENTION_REVISION=25f5b52ce0feb074db1529c939b7d5aec1b5aa30
PREVIOUS_PROJECT_RECORD_REVISION=cdfd6dfba8c8efdd72defc89f9727ff7e9235061

PERF010A_MEASUREMENT_DISCRIMINATION=ESTABLISHED
NO_OP_EXPERIMENT=VALID
NO_OP_RUNTIME_EQUIVALENCE=PASS
ABLATION_5_MEASUREMENT_GATE=CLOSED
```

Retained benchmark evidence:

```text
guillermomolina/protos-benchmarks@25f5b52ce0feb074db1529c939b7d5aec1b5aa30

results/perf010a-discrimination/README.md
results/perf010a-discrimination/raw.json
results/perf010a-discrimination/discrimination-blocks.tsv
results/perf010a-discrimination/discrimination-summary.tsv
results/perf010a-discrimination/SHA256SUMS
```

Harness source materially inspected at
`57cb307baad3346758d34aa3da1eda01997a4208`:

```text
runner/perf010a.py
docker/protos-perf010a/Dockerfile
docker/protos-perf010a/Perf010aTimingDriver.java
config/perf010a-0.json
docker/protos-perf010a/noop.patch
```

The no-op patch is literally empty and baseline/no-op relevant runtime source
identity was confirmed before timing. Any baseline/no-op movement discussed
below is therefore measurement-system movement, not a Protos runtime effect.

## Result

```text
PERF010A_MEASUREMENT_INSTABILITY_CAUSE=PARTIALLY_ESTABLISHED

REFERENCE_EVIDENCE_VALID=YES
NO_OP_RUNTIME_EQUIVALENCE=PASS

INSTABILITY_LOCATION=BOTH
CROSS_WORKLOAD_BLOCK_SIGNAL=ABSENT
PAIRED_CONTROL_SUBTRACTION=MIXED

BLOCK_LIFECYCLE_ISOLATION_JUSTIFIED=NO
MORE_COUNTERBALANCED_BLOCKS_JUSTIFIED=YES

NEXT_METHODOLOGICAL_INTERVENTION=ESTABLISHED
SELECTED_INTERVENTION=EXTEND_PER_TIMED_UNIT_WARMUP_ONLY

ABLATION_5_MEASUREMENT_GATE=CLOSED
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

The evidence narrows the instability to fresh-JVM timing-unit nonstationarity /
settling combined with independent canonical/control measurement and fixed
temporal sequencing. It does not identify the physical cause uniquely; JIT,
Truffle compilation dynamics, GC, scheduler behavior, CPU-frequency behavior,
host cache state, and other runtime/host mechanisms remain unresolved.

## Actual execution lifecycle

The retained reference executes:

```text
reference
  -> build baseline image once
  -> build no-op image once
  -> verify source equivalence
  -> block A/B/A/B
     -> workload
        -> counterbalanced variant
           -> canonical
              -> fresh docker run --rm
              -> fresh Java process / JVM
              -> fresh Protos bootstrap/process context
              -> warmup x20
              -> steady x100
           -> control
              -> fresh docker run --rm
              -> fresh Java process / JVM
              -> fresh Protos bootstrap/process context
              -> warmup x20
              -> steady x100
```

Therefore:

```text
JVM_LIFETIME=ONE_TIMED_UNIT
PROCESS_LIFETIME=ONE_TIMED_UNIT
CONTAINER_LIFETIME=ONE_TIMED_UNIT
IMAGE_LIFETIME=WHOLE_REFERENCE
HOST_LIFETIME=WHOLE_REFERENCE
```

No JVM, process, Protos process context, or container survives a
canonical/control boundary, workload boundary, or block boundary.

The earlier recommendation to add process/container lifecycle isolation between
blocks is therefore not supported by the actual harness architecture: that
lifecycle is already fresh at a finer boundary than the block.

## Reconstructed paired-control observations

Sign convention:

```text
canonical_difference = baseline_canonical - noop_canonical
control_difference = baseline_control - noop_control
paired = canonical_difference - control_difference
```

Times below are retained medians in milliseconds.

| block | workload | baseline canonical | baseline control | noop canonical | noop control | paired % |
|---:|---|---:|---:|---:|---:|---:|
| 0 A | micro/slot-read | 49.261 | 50.711 | 50.481 | 45.957 | -12.1293 |
| 0 A | micro/closure-call | 65.977 | 47.488 | 64.606 | 47.689 | +2.3830 |
| 0 A | micro/method-call | 59.712 | 47.447 | 62.038 | 45.404 | -7.3154 |
| 0 A | runtime/monomorphic-dispatch | 60.353 | 48.877 | 56.165 | 45.284 | +0.9862 |
| 1 B | micro/slot-read | 49.139 | 67.820 | 47.823 | 47.387 | -38.9049 |
| 1 B | micro/closure-call | 56.191 | 45.453 | 78.791 | 48.480 | -34.8335 |
| 1 B | micro/method-call | 65.052 | 44.957 | 69.217 | 47.034 | -3.2091 |
| 1 B | runtime/monomorphic-dispatch | 58.790 | 45.553 | 59.138 | 46.535 | +1.0794 |
| 2 A | micro/slot-read | 46.259 | 45.647 | 46.615 | 45.626 | -0.8163 |
| 2 A | micro/closure-call | 64.753 | 46.740 | 59.875 | 47.171 | +8.2006 |
| 2 A | micro/method-call | 59.965 | 49.521 | 65.714 | 48.777 | -10.8286 |
| 2 A | runtime/monomorphic-dispatch | 56.400 | 46.357 | 54.703 | 45.838 | +2.0901 |
| 3 B | micro/slot-read | 53.196 | 47.266 | 46.827 | 47.171 | +11.7933 |
| 3 B | micro/closure-call | 58.379 | 45.745 | 68.771 | 47.340 | -15.0683 |
| 3 B | micro/method-call | 55.831 | 46.248 | 58.996 | 47.605 | -3.2389 |
| 3 B | runtime/monomorphic-dispatch | 59.611 | 49.893 | 55.199 | 46.198 | +1.2019 |

The paired-control magnitude is not generated by one common component:

- slot-read block 1 is dominated by a baseline-control excursion;
- closure-call block 1 is dominated by a no-op canonical excursion;
- method-call movement is more distributed;
- monomorphic-dispatch stays comparatively tight.

## ABS_MAX leverage

The current discrimination floor is the maximum absolute paired observation.
Removing the single floor-determining block only as a diagnostic calculation,
without discarding it from authoritative evidence, gives:

| workload | floor-determining block | retained floor % | floor without max block % |
|---|---:|---:|---:|
| micro/slot-read | 1 | 38.9049 | 12.1293 |
| micro/closure-call | 1 | 34.8335 | 15.0683 |
| micro/method-call | 2 | 10.8286 | 7.3154 |
| runtime/monomorphic-dispatch | 2 | 2.0901 | 1.2019 |

The very large slot-read and closure-call floors therefore have substantial
single-observation leverage, while method-call remains materially unstable after
its maximum block is excluded descriptively.

No observation is discarded or reclassified.

## Raw within-unit temporal structure

The strongest causal clue is inside individual fresh-JVM timing units.

### slot-read, block 1, baseline control

```text
median = 67.820 ms
first-quarter median = 47.511 ms
last-quarter median = 74.524 ms
last-quarter vs first-quarter = +56.9%
```

The last five steady samples include approximately:

```text
133.962 ms
76.819 ms
151.919 ms
58.426 ms
134.200 ms
```

That fresh timed unit alone contributes a +20.434 ms control difference and
drives the -38.9049% paired observation.

### closure-call, block 1, no-op canonical

```text
median = 78.791 ms
first-quarter median = 81.672 ms
last-quarter median = 57.242 ms
last-quarter vs first-quarter = -29.9%
```

That fresh timed unit drives the -34.8335% workload floor primarily through
canonical movement.

### slot-read, block 3, baseline canonical

```text
median = 53.196 ms
first-quarter median = 46.612 ms
last-quarter median = 77.342 ms
last-quarter vs first-quarter = +65.9%
```

Its neighboring control remains comparatively stable, producing the
+11.7933% paired observation.

These observations establish that some fresh JVM timing units continue changing
substantially during the 100 samples currently classified as steady, even after
their own 20 warmup iterations.

They do not establish whether JIT, GC, scheduler, CPU-frequency behavior, or
another physical mechanism causes that movement.

## Canonical/control stability

Across blocks, relative spread `(max-min)/median` is strongly
workload/component dependent.

| workload | baseline canonical | baseline control | noop canonical | noop control |
|---|---:|---:|---:|---:|
| slot-read | 14.1% | 45.3% | 8.2% | 3.8% |
| closure-call | 15.9% | 4.4% | 28.4% | 2.8% |
| method-call | 15.4% | 9.7% | 16.0% | 7.1% |
| monomorphic-dispatch | 6.7% | 9.1% | 8.0% | 2.7% |

There is therefore no single "control instability" explanation. Instability can
enter through canonical or control independently.

## A/B order reinterpretation

The retained paired observations by order are:

```text
slot-read:
A = [-12.1293, -0.8163]
B = [-38.9049, +11.7933]

closure-call:
A = [+2.3830, +8.2006]
B = [-34.8335, -15.0683]

method-call:
A = [-7.3154, -10.8286]
B = [-3.2091, -3.2389]

monomorphic-dispatch:
A = [+0.9862, +2.0901]
B = [+1.0794, +1.2019]
```

For closure-call, the first variant's canonical run is slower than the second
variant's canonical run in all four blocks. For method-call, a similar
first/second position pattern is visible in the controls.

With only two A and two B blocks, variant order and temporal position cannot be
cleanly separated. The evidence is consistent with temporal-position effects,
not with a Protos variant effect.

Slot-read does not show a coherent A/B direction: the two B blocks contain both
the most negative and a large positive paired result.

## Cross-workload structure

No coherent host/block-wide movement is established.

In block 1, for example:

- slot-read's large excursion is in baseline control;
- closure-call's large excursion is in no-op canonical;
- method-call moves much less;
- monomorphic-dispatch remains near 1%.

That is not the signature of one block-wide disturbance moving every
workload/mode together.

Host effects remain possible, but the retained evidence does not identify a
single block-wide host mechanism.

## Paired-control subtraction

Canonical and control are separate fresh containers/processes/JVMs. The control
matches the canonical run in revision, image variant, toolchain, CPU affinity,
operation count, warmup/steady policy, bootstrap structure, and surrounding
benchmark source, but does not share JVM/process/runtime state or temporal
sample identity.

The subtraction can therefore remove common movement or amplify independent
movement.

Observed comparison of absolute canonical difference vs absolute paired
difference:

```text
slot-read:                 amplify 3/4, reduce 1/4
closure-call:              amplify 2/4, reduce 2/4
method-call:               amplify 2/4, reduce 2/4
monomorphic-dispatch:      amplify 1/4, reduce 3/4
```

Hence:

```text
PAIRED_CONTROL_SUBTRACTION=MIXED
```

This is diagnostic evidence only; the authoritative paired-control metric and
floor definition remain unchanged.

## Methodological reconciliation

The previous durable record reported the harness-generated recommendation:

```text
process/container lifecycle isolation between blocks
plus more counterbalanced blocks
```

After source inspection:

```text
BLOCK_LIFECYCLE_ISOLATION_JUSTIFIED=NO
```

because every timed unit already receives a fresh container/process/JVM.

More counterbalanced blocks remain justified as later replication to determine
whether excursions and first/second-position patterns recur:

```text
MORE_COUNTERBALANCED_BLOCKS_JUSTIFIED=YES
```

but increasing block count first would characterize frequency without directly
testing the strongest observed mechanism.

## Selected next methodological intervention

The narrowest one-variable intervention is:

```text
NEXT_METHODOLOGICAL_INTERVENTION=ESTABLISHED
SELECTED_INTERVENTION=EXTEND_PER_TIMED_UNIT_WARMUP_ONLY

warmup_iterations: 20 -> 120
steady_iterations: 100 unchanged
block count: 4 unchanged
block order: A/B/A/B unchanged
variant order: unchanged
canonical/control order: unchanged
fresh container/process/JVM policy: unchanged
CPU affinity: unchanged
workload order: unchanged
paired-control calculation: unchanged
floor definition: unchanged
```

Rationale: the decisive excursions occur inside the current post-warmup
100-sample window. Adding exactly 100 more warmup executions moves the retained
window beyond the entire lifecycle interval currently observed without changing
another methodological variable.

Interpretation of the next no-op reference:

- if within-unit drift and no-op floors contract substantially, the evidence
  supports that the current retained window begins too early in the fresh-JVM
  lifecycle;
- if they remain, that hypothesis is weakened and the next investigation should
  move to independent-run pairing, build/image identity, or additional
  host/runtime instrumentation.

Do not simultaneously add blocks, change canonical/control ordering, add
lifecycle changes, or alter the floor. One methodological variable should
change at a time.

## Reconciled state

```text
PERF010A_MEASUREMENT_INSTABILITY_CAUSE=PARTIALLY_ESTABLISHED

BLOCK_LIFECYCLE_ISOLATION_JUSTIFIED=NO
MORE_COUNTERBALANCED_BLOCKS_JUSTIFIED=YES

NEXT_METHODOLOGICAL_INTERVENTION=ESTABLISHED
SELECTED_INTERVENTION=EXTEND_PER_TIMED_UNIT_WARMUP_ONLY

ABLATION_5_MEASUREMENT_GATE=CLOSED

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCTION_OPTIMIZATION_SELECTED=NO
PERF010_READY=NO
```

PERF010-A remains open. The next bounded action is a no-op measurement
reference changing only per-timed-unit warmup from 20 to 120. Ablation 5 and
production optimization remain unauthorized.
