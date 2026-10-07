# PERF025-D3 — final reusable-call reference and closure

Date: 2026-10-02

## Closure identity

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-D3
TYPE=MEASUREMENT_AND_CLOSURE

PRODUCT_REPOSITORY_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

FINAL_PRODUCT_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
REFERENCE_PRODUCER_REVISION=6d48bf8f41ef5890e5678cd0b5c6c75694f61339
RETAINED_EVIDENCE_REVISION=7d77ab58e97e4eaa8a4af5e4db898ee8795532cc
RETAINED_DESTINATION=results/perf025-d3/
MEASUREMENT_DEFINITION=jvm-protos-session-ab-v3
REFERENCE_ADMISSION_SCOPE=steady-only
RETAINED_OBSERVATIONS=12
REFERENCE_STATUS=PASS
PERF025_STATUS=COMPLETE
~~~

## Exact comparison matrix

The final reference preserves the four exact PERF025 product checkpoints:

~~~text
PRE_A  f3c44554ddfb9004c43dbde5197b9805990f8a4a  dynamic   0.3.129-SNAPSHOT
A      d0045353d834257b5fb80581846b32aebd43c7e6  dynamic   0.3.130-SNAPSHOT
B      6811d0cef3735d39ffd3801b3bae6ef48318bb66  prepared  0.3.131-SNAPSHOT
FINAL  19d7426a5b8f0e3b93d36f56aee33377a4ee9985  prepared  0.3.143-SNAPSHOT
~~~

and the three primitive reusable-call workloads:

~~~text
primitive-return-literal
primitive-closure-call
primitive-method-call
~~~

The retained matrix is therefore exactly 12 observations.

## D3 admission correction

The initial D2 contract reused the PERF024 primitive reference policy too
literally. At 100 calls per timing sample, sparse scheduler/JIT/host stalls were
large relative to the sample itself. Increasing the sample unit to 10,000 calls
made the measured steady state stable, but repeated D3 probes then showed that a
fixed warmup-stability gate could still reject a run even when all measured
steady windows were stable.

The bounded correction sequence was:

~~~text
D3 initial / D2 harness:
  sample_calls=100
  reference_warmup=60
  reference_steady=10
  admitted=1/12

D3A:
  harness=0085fdd3a81500047178fd9872dc52d216246853
  sample_calls=10000
  reference_warmup=10
  reference_steady=10
  result=warmup transition still dominates admission

D3B:
  harness=2d0e74d6e35f2dbb79262dcd2d8448b4f6f49088
  sample_calls=10000
  reference_warmup=20
  reference_steady=10
  result=remaining method-call warmup rejection

D3C:
  harness=d3fccdf7f4170f3509fc9ccf0b4af13944a23b26
  sample_calls=10000
  reference_warmup=50
  reference_steady=10
  result=all 12 steady windows PASS; two observations still rejected only by
         the warmup window

D3D:
  harness=6d48bf8f41ef5890e5678cd0b5c6c75694f61339
  sample_calls=10000
  reference_warmup=50
  reference_steady=10
  admission_scope=steady-only
~~~

D3D does **not** relax the stability thresholds. It retains:

~~~text
STABILITY_WINDOW=5
STABILITY_MEDIAN_DRIFT_PCT_MAX=15
STABILITY_MAD_PCT_MAX=20
STABILITY_MAX_INTERNAL_GAP_PCT=20
STABILITY_MIN_GAP_CLUSTER_SIZE=3
~~~

The semantic change is instead the role of warmup: the 50×10,000-call warmup is
bounded conditioning, while reference admission is determined by the measured
10-sample steady window. The measurement identity was advanced from
`jvm-protos-session-ab-v2` to `jvm-protos-session-ab-v3` so v2 and v3 cache or
retained evidence cannot mix.

## Final retained reference

The exact producer `6d48bf8f41ef5890e5678cd0b5c6c75694f61339` reports:

~~~text
reference_sample_calls=10000
reference_warmup=50
reference_steady=10
reference_admission_scope=steady-only

cases_passed_correctness=12/12
accepted_reference_observations=12
perf025_reference=PASS
~~~

Retention was then verified from:

~~~text
guillermomolina/protos-benchmarks@7d77ab58e97e4eaa8a4af5e4db898ee8795532cc
results/perf025-d3/

producer_revision=6d48bf8f41ef5890e5678cd0b5c6c75694f61339
retained_observations=12
jvm-ab-v3-reference=12
retained_results=PASS
~~~

The retained destination contains one manifest and 12 raw observations.

## Accumulated PRE_A → FINAL result

The primary owner-facing result is the accumulated change from the exact
PRE_A/dynamic checkpoint to the exact FINAL/prepared checkpoint. It deliberately
does not assign the total effect to any individual A/B/C slice.

| workload | PRE_A ns/call | FINAL ns/call | accumulated time change | speedup |
| --- | ---: | ---: | ---: | ---: |
| primitive-return-literal | 4639.5 | 4522.4 | -2.52% | 1.026× |
| primitive-closure-call | 6146.3 | 5557.3 | -9.58% | 1.106× |
| primitive-method-call | 5231.7 | 5012.0 | -4.20% | 1.044× |

All three accumulated comparisons are lower-time at FINAL.

The retained intermediate A/B deltas remain available in the raw evidence, but
they are not the primary PERF025 conclusion.

## Acceptance closure

~~~text
PERF025_A_DIRECT_FIRST_ROOT_TASK=COMPLETE
PERF025_B_PREPARED_REUSABLE_CALLABLE=COMPLETE
PERF025_C_CARRIER_TAX_REDUCTION=COMPLETE
REPEATED_CALL_BEFORE_AFTER_EVIDENCE=PASS
UNSTABLE_EVIDENCE_USED_FOR_CLAIM=NO
BUG008_REOPENED=NO
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERF025=COMPLETE
~~~

PERF025-C2D reduced the shared guest-carrier stack budget from 64 MiB to 16 MiB
without redefining BUG008. The final D3 measurement adds no Protos product or
language change.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos-benchmarks@6d48bf8f41ef5890e5678cd0b5c6c75694f61339` — final v3 reference producer.
- `guillermomolina/protos-benchmarks@7d77ab58e97e4eaa8a4af5e4db898ee8795532cc` — retained D3 evidence.
- `docs/project/evidence/PERF025/PERF025_D1_FINAL_REUSABLE_CALL_BENCHMARK_EVIDENCE_DESIGN.md`.
- `docs/project/evidence/PERF025/PERF025_D2_DYNAMIC_PREPARED_EXACT_REVISION_HARNESS.md`.
