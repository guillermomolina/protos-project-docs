# PERF033-C — retained canonical callable reference evidence

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=PERF033
ISSUE=guillermomolina/protos#832

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=ddb2a30626f4ec23159f4a79f6d097061b09dcb0
PROTOS_VERSION=0.3.277-SNAPSHOT

BENCHMARK_REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_PRODUCER_REVISION=e3609eca8e678764a2b9453f248849ef8ff6e673
BENCHMARK_EVIDENCE_REVISION=6c27328eda9ad65565eee0ce45de28e2fc624c7b
~~~

The benchmark evidence commit is:

~~~text
PERF033: retain canonical callable reference evidence
~~~

The retained result is:

~~~text
results/perf033-current/
  protos-canonical--primitive-return-literal--reference-jfr/
    20261007T133955608878Z/
~~~

## Product and producer identity

The retained metadata records:

~~~text
PRODUCT_DIRECTORY=/workspaces/protos
PROTOS_REVISION=ddb2a30626f4ec23159f4a79f6d097061b09dcb0
PROTOS_VERSION=0.3.277-SNAPSHOT
PROTOS_CLEAN=YES

HARNESS_GIT_HEAD=e3609eca8e678764a2b9453f248849ef8ff6e673
HARNESS_DIRTY=NO

SURFACE=canonical
SURFACE_TIMED_CALL=prepared.executable().execute()
WORKLOAD=primitive-return-literal
STAGE=reference
PROFILE=jfr
REFERENCE_ELIGIBLE=YES
~~~

The metadata contains exact producer-source hashes and matching before/after
product identities. The selected Protos source-state SHA-256 is:

~~~text
b335476fa11b5b460e8646536af3bc44c5a51ffcd47930928054fc47ed7da29b
~~~

The canonical adapter source SHA-256 is:

~~~text
56c598ef559eb45ffbe2fdb5747559be39d1620490cb45998b52efa800949b82
~~~

## Correctness and measurement validity

~~~text
EXPECTED_RESULT=1
ACTUAL_RESULT=1
CORRECTNESS=PASS

MEASUREMENT_VALID=YES
STEADY_STATE_ADMISSION=PASS
JFR=RECORDED
JFR_SCOPE=steady
JFR_TIMING_IS_SEPARATE_RUN=YES
~~~

Timing policy:

~~~text
WARMUP_ITERATIONS=50
STEADY_ITERATIONS=10
SAMPLE_CALLS=10000
ADMISSION_SCOPE=steady-only
~~~

JFR policy:

~~~text
WARMUP_ITERATIONS=20
STEADY_ITERATIONS=10
SAMPLE_CALLS=1000000
STACK_DEPTH=256
~~~

The retained JFR SHA-256 is:

~~~text
e02bd688e5254586a6a147244492ec45465e73134199a36d2f9fb9bae623a167
~~~

## Retained timing

~~~text
STEADY_AMORTIZED_P50_NS_PER_CALL=390.97025
STEADY_MIN_SAMPLE_NS=3436507
STEADY_MAX_SAMPLE_NS=6478232
STEADY_SAMPLE_CALLS=10000
~~~

The ten retained steady sample durations are:

~~~text
5066179
4023858
6478232
3795547
3436507
4569198
4963249
3625887
3483816
3551826
~~~

The steady admission check passed. Its retained split-window diagnostics include:

~~~text
PREVIOUS_P50_NS=4023858
LATEST_P50_NS=3625887
MEDIAN_DRIFT_PERCENT=9.89028439870393
LATEST_MAD_PERCENT=3.9182412469004135
STEADY_SHAPE_STATUS=PASS
~~~

No whole-language performance claim is made from this microbenchmark.

## Relationship to the earlier PERF033 structural result

PERF033-A established the canonical executable Protos callable boundary and its
pay-as-you-grow structure. The retained reference in this record measures that
canonical public surface on later current Protos revision
`ddb2a30626f4ec23159f4a79f6d097061b09dcb0`.

This record does not reinterpret the earlier non-retained ~115 ns diagnostic as
retained evidence and does not claim that the difference between that diagnostic
and this retained 390.97025 ns/call result is a product regression. They are not
the same retained measurement unit.

## Remaining acceptance work

The PERF033 issue requires compatible same-workload cross-Truffle timing after
the structural implementation.

The published `results/perf033-current/` evidence tree at benchmark revision
`6c27328eda9ad65565eee0ce45de28e2fc624c7b` contains only the Protos canonical
reference. It does not contain retained GraalJS or GraalPy reference results.

Therefore:

~~~text
PROTOS_CANONICAL_REFERENCE=RETAINED
CROSS_TRUFFLE_PEER_REFERENCE=NOT_YET_RETAINED
NEXT_PRODUCT_CHANGE_REQUIRED=NO
PERF033_CLOSEABLE=NO
REMAINING_GATE=retain compatible JS and Python primitive-return-literal references
~~~

The next bounded work is measurement/publication only. It must use the stable
revision-independent harness without changing Protos or harness source.

## AI-assistance disclosure

This durable record was materially prepared with AI assistance from ChatGPT
from the exact published benchmark metadata, summary, raw correctness/timing
logs, and live PERF033 acceptance criteria.
