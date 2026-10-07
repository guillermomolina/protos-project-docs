# PERF010-B — Step-3 final routing and campaign closure

Date: 2026-09-30

## Scope

This record closes PERF010-B/#722 after the planned post-I072 fast-path
convergence sequence reached its declared Step-3 stop gate.

The campaign retained all structurally correct product improvements, measured
their timing effects, and stopped when the final represented-selection work did
not produce the predicted material timing-class change.

## Final product sequence

```text
PERF012/#723  frame lexical-layout PE remediation                 CLOSED
PERF013/#724  captured read/write MaterializedLocalAccessor       CLOSED
PERF014/#725  stable direct Closure-call selection                CLOSED
PERF015/#726  canonical Boolean represented selection             CLOSED
PERF016/#727  Integer represented selection + Step-3 timing       CLOSED
```

Key final product identities:

```text
PRE_STEP3_CONTROL=2e3f56fae3a500d3e4193e3345d8a82c35e4590e
POST_STEP3_PRODUCT=696b0f9797ebc8ced80009fb583027513852f55c
POST_STEP3_VERSION=0.3.119-SNAPSHOT
```

## Final Step-3 timing authority

Harness:

```text
BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=ac59110d23cb4724e4aa438a2a5781aaf1b31a77
TOOLCHAIN=25.4.4.1.1
```

Durable PERF016 timing record:

```text
PROJECT_RECORD_REVISION=69499b35c5522f767cbd30eede3d0967216c6657
PROJECT_RECORD_PATH=docs/project/evidence/PERF016/PERF016_POST_STEP3_CONTROLLED_TIMING_RESULT.md
```

The maintainer-executed reference reported correctness and workload-source
identity PASS.

Retained median effects:

| workload | shared-driver/control | canonical | paired-control |
|---|---:|---:|---:|
| micro/slot-read | +5.3537% | -6.8125% | -10.1862% |
| micro/closure-call | +1.2583% | -6.0053% | -7.2338% |
| micro/method-call | +3.3384% | +3.7213% | -0.6383% |
| runtime/monomorphic-dispatch | +0.1797% | +0.4835% | -3.3498% |

Additional run diagnostics:

```text
ORDER_EFFECT=PRESENT
STATIONARITY_MAX_ABS_LAST_VS_FIRST_QUARTER_PERCENT=22.7265
```

## Campaign conclusion

The Step-3 structural target is complete:

```text
BOOLEAN_REPRESENTED_SELECTION=COMPLETE
INTEGER_REPRESENTED_SELECTION=COMPLETE
```

The final timing classification is:

```text
SHARED_DRIVER_TIMING_EFFECT=SMALL_MIXED_POSITIVE
COMMON_WORKLOAD_TIMING_EFFECTS=MIXED_NO_CONSISTENT_IMPROVEMENT
ORDER_EFFECT=PRESENT
STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED
```

The direct shared-driver measurements contain small positive movement, but the
canonical results are mixed, all four paired-control residuals are negative, and
the run contains meaningful order/stationarity variability. The intervention
therefore does not establish a material consistent timing-class improvement.

This is a falsified performance prediction, not a structural correctness
failure. PERF015 and PERF016 remain accepted product improvements.

## Stop-gate routing

PERF010-B explicitly stated that if stable Boolean + Integer selection became
structurally complete while the shared driver remained in the same cost class,
the campaign must stop and re-evaluate rather than automatically enter Step 4.

That gate is now resolved:

```text
POST_STEP3_CONTROLLED_TIMING_CHECKPOINT=RETAINED
STEP3_NEXT_ROUTING=STOP_AND_REEVALUATE

STEP4_AUTHORIZED=NO
STEP4_ALLOCATED=NO

PERF010_B_STATUS=CLOSED_COMPLETE
```

No residual architecture implementation is authorized by this closure.

Any future architecture experiment must be justified by new bounded causal
evidence and routed through the applicable design/platform gate rather than
treated as an automatic continuation of PERF010-B.

## Separate benchmark-workflow follow-up

The PERF016 reference exposed a cross-cutting benchmark-workflow problem:
per-intervention comparator implementation is too expensive, and the current
reference matrix serializes 64 timed units on one logical CPU.

That concern is independent of the runtime architecture result above. It should
be tracked as separate performance-infrastructure work so future
implement -> measure -> decide cycles can reuse one parameterized comparator and
obtain fast non-retained feedback before paying for retained reference evidence.
