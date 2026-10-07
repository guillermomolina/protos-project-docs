# PERF025-E3B — retained carrier-transport A/B reference

Date: 2026-10-02

## Evidence identity

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-E3B
TYPE=MEASUREMENT_AND_RETENTION

PRODUCT_REPOSITORY_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

PRE_E2_REVISION=c1eb2c2e1a811d70fcf09f5526217c1aaf14d141
PRE_E2_VERSION=0.3.144-SNAPSHOT

E2_REVISION=ff618f0dd8ef680f7884c9145552f77912f040a2
E2_VERSION=0.3.145-SNAPSHOT

HARNESS_REPOSITORY=guillermomolina/protos-benchmarks
REFERENCE_PRODUCER_REVISION=3fb036ce86b6c452588230b5322967f55aaf0160
RETAINED_EVIDENCE_REVISION=8441ebfb6fe970cfd9a8994f182cea974f4498a0
RETAINED_DESTINATION=results/perf025-e3/

MEASUREMENT_DEFINITION=jvm-protos-carrier-e2-ab-v1
REFERENCE_SAMPLE_CALLS=10000
REFERENCE_WARMUP=50
REFERENCE_STEADY=10
REFERENCE_ADMISSION_SCOPE=steady-only

REFERENCE_OBSERVATIONS=6
CORRECTNESS_PASS=6/6
ADMISSION_PASS=6/6
HARNESSES_DIRTY=NO
REFERENCE_RETRY=NO
~~~

## Exact retained result

All observations use the same prepared reusable top-level call surface and compare
the exact E2 commit with its exact immediate parent.

| workload | PRE_E2 ns/call | E2 ns/call | absolute delta ns/call | E2 time change |
| --- | ---: | ---: | ---: | ---: |
| primitive-return-literal | 6692.741 | 6173.223 | -519.519 | -7.76% |
| primitive-closure-call | 7522.591 | 7181.618 | -340.973 | -4.53% |
| primitive-method-call | 7013.635 | 6589.426 | -424.209 | -6.05% |

Negative time change means E2 took less time.

These are steady amortized per-call p50 values: median of ten steady samples,
each containing 10,000 `PreparedTopLevel.invoke()` calls.

## Admission quality

Every retained observation passed the existing steady-only PERF025 admission
policy.

Observed steady-window diagnostics were comfortably below the retained
thresholds:

~~~text
MAX_MEDIAN_DRIFT_PCT_OBSERVED=2.1163
THRESHOLD=15

MAX_PREVIOUS_MAD_PCT_OBSERVED=2.4760
THRESHOLD=20

MAX_LATEST_MAD_PCT_OBSERVED=1.5251
THRESHOLD=20

MAX_INTERNAL_GAP_PCT_OBSERVED=5.0472
THRESHOLD=20
~~~

The largest individual steady outliers therefore did not invalidate any
observation. No reference observation was rejected or automatically retried.

## Interpretation

E2 produces a consistent lower-time result on all three measured prepared-call
primitives.

The direct same-surface causal evidence supports:

~~~text
E2_CARRIER_TRANSPORT_CLEANUP_EFFECT=POSITIVE
RETURN_LITERAL_DELTA=-7.76%
CLOSURE_CALL_DELTA=-4.53%
METHOD_CALL_DELTA=-6.05%
~~~

The absolute reduction is of order a few hundred nanoseconds per invocation:

~~~text
RETURN_LITERAL_SAVING≈520 ns/call
CLOSURE_CALL_SAVING≈341 ns/call
METHOD_CALL_SAVING≈424 ns/call
~~~

The fact that all three workloads move in the same direction by a similar
order of magnitude is consistent with E2 reducing a fixed per-invocation
transport cost. One retained reference is not sufficient to claim that the
exact nanosecond difference is a universal fixed constant.

Most importantly, E2 does not remove the reusable-call floor. In this exact E3
campaign, the simplest E2 point remains:

~~~text
primitive-return-literal E2≈6173 ns/call
~~~

Therefore removing per-call one-element arrays, the wrapping completion lambda,
and monitor wait/notify removes only a minority of total prepared invocation
latency. The retained dedicated cross-thread handoff, queueing, wakeup/scheduling,
Context/Process entry and other fixed invocation machinery remain candidates for
the residual.

Do not numerically subtract this E3 absolute floor from historical PERF024/D3
absolute values produced by a different campaign. The causal conclusion is the
within-E3 PRE_E2/E2 delta.

## Consequence for PERF025

~~~text
PERF025_E2_IMPLEMENTATION=VALIDATED_POSITIVE
PERF025_E3B_REFERENCE=PASS
PERF025_E3B_RETENTION=PASS

CARRIER_TRANSPORT_MONITOR_CLEANUP=HELPFUL_BUT_INSUFFICIENT
DEDICATED_CARRIER_HANDOFF_RESIDUAL=STILL_MATERIAL

OPEN_ENDED_PROFILING_REQUIRED=NO
HISTORICAL_D3_EVIDENCE_CHANGED=NO
~~~

The next PERF025 work should not rewrite benchmark methodology. The existing
prepared primitive measurement is sufficient as the performance radar.

Further optimization should focus on the remaining per-invocation dedicated
carrier handoff while preserving the currently ratified C2C constraints unless
a separate platform/design decision explicitly reopens them.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos@c1eb2c2e1a811d70fcf09f5526217c1aaf14d141` — exact PRE_E2 baseline.
- `guillermomolina/protos@ff618f0dd8ef680f7884c9145552f77912f040a2` — E2 implementation.
- `guillermomolina/protos-benchmarks@3fb036ce86b6c452588230b5322967f55aaf0160` — exact clean E3 reference producer.
- `guillermomolina/protos-benchmarks@8441ebfb6fe970cfd9a8994f182cea974f4498a0` — retained raw E3 evidence.
- `docs/project/evidence/PERF025/PERF025_E2_GUEST_CARRIER_TRANSPORT_CLEANUP.md`.
- `docs/project/evidence/PERF025/PERF025_D3_FINAL_REUSABLE_CALL_REFERENCE_AND_CLOSURE.md`.
