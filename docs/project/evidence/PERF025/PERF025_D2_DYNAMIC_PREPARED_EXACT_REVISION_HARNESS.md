# PERF025-D2 — dynamic/prepared exact-revision benchmark harness

Date: 2026-10-02

## Publication identity

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-D2
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks

BASE_HARNESS_REVISION=5486c515369c8a958c8baa6949b818dbfaf7d0f8
HARNESS_REVISION=e0bc5a5c2f8c0b373405c75bf9df9e5a54e94b2b
COMMIT_SUBJECT=PERF025-D2: add dynamic/prepared exact-revision A/B harness

PRODUCT_REPOSITORY_CHANGE=NO
PROTOS_CHANGE=NONE
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The publication is a single commit whose parent is the exact harness revision
inspected by PERF025-D1.

## Published paths

~~~text
BENCHMARKING.md
Makefile
truffle/Makefile
truffle/jvm_protos_ab.py
truffle/perf025_ab.py
truffle/retain_results.py
truffle/src/main/java/com/guillermomolina/protos/benchmarks/truffle/ProtosJvmVariantRunner.java
truffle/src/prepared/java/com/guillermomolina/protos/benchmarks/truffle/ProtosPreparedVariantRunner.java
~~~

No `guillermomolina/protos` product file is changed by D2.

## Measurement matrix implemented

The harness pins the four exact PERF025 product points selected by D1:

~~~text
PRE_A  f3c44554ddfb9004c43dbde5197b9805990f8a4a  dynamic   0.3.129-SNAPSHOT
A      d0045353d834257b5fb80581846b32aebd43c7e6  dynamic   0.3.130-SNAPSHOT
B      6811d0cef3735d39ffd3801b3bae6ef48318bb66  prepared  0.3.131-SNAPSHOT
FINAL  19d7426a5b8f0e3b93d36f56aee33377a4ee9985  prepared  0.3.143-SNAPSHOT
~~~

The selected primitive workloads are:

~~~text
primitive-return-literal
primitive-closure-call
primitive-method-call
~~~

with:

~~~text
SAMPLE_CALLS=100
EXPECTED_OBSERVATIONS=12
~~~

## Dynamic/prepared compilation boundary

The common Maven runner remains compatible with the pre-B Protos API and uses
the dynamic call path. D2 adds a separate prepared-only Java source root that is
compiled directly against the B-and-later exact variant classpaths.

The prepared path performs:

~~~text
SETUP:
  session.prepareTopLevel("run")

TIMED:
  prepared.invoke()
~~~

The prepared runner is not part of the common Maven source root. Structural
`javap` checks reject prepared API references in the dynamic runner and reject
reflection-based compatibility adaptation.

## Reference policy

D2 reuses the existing PERF024 JVM admission policy:

~~~text
REFERENCE_WARMUP=60
REFERENCE_STEADY=10
STABILITY_WINDOW=5
STABILITY_MEDIAN_DRIFT_PCT_MAX=15
STABILITY_MAD_PCT_MAX=20
STABILITY_MAX_INTERNAL_GAP_PCT=20
STABILITY_MIN_GAP_CLUSTER_SIZE=3
~~~

A non-admitted reference observation is written as rejected local evidence and
is not cached or automatically retried.

The D2 smoke path is intentionally non-reference:

~~~text
SMOKE_WARMUP=2
SMOKE_STEADY=3
SMOKE_CASES=12
SMOKE_TIMING_INTERPRETATION=NONE
~~~

## Identity and retention

The new measurement definition is:

~~~text
jvm-protos-session-ab-v2
~~~

Its identity includes the clean exact harness revision, run mode, exact Protos
revision/version/Core hash, source hash, GraalVM/Java/host identity, warmup and
steady policies, sample calls, and admission policy. Dynamic and prepared runs
therefore cannot share one cache identity.

D2 adds an independent `perf025` retention profile while preserving the
historical PERF023 profile. The future retained destination is:

~~~text
results/perf025-d3/
~~~

and requires exactly:

~~~text
jvm-ab-v2-reference=12
~~~

accepted observations from the exact clean producer revision.

## Publication verification boundary

The coordinating agent verified the exact remote commit identity, parent, and
changed-path set after the maintainer reported the D2 push.

No benchmark, test, or validation command was re-executed by the coordinating
agent. This record therefore does not fabricate an independent PASS claim for
commands whose output was not included in the publication handoff.

## Next slice

~~~text
NEXT_SLICE=PERF025-D3
TYPE=IMPLEMENTATION_MEASUREMENT
REPOSITORY=guillermomolina/protos-benchmarks

REFERENCE_PRODUCER_REVISION=e0bc5a5c2f8c0b373405c75bf9df9e5a54e94b2b
REFERENCE_RUN_COUNT=ONE_BOUNDED_CAMPAIGN
RERUN_D2_SMOKE_BY_DEFAULT=NO
FULL_TEST_SUITE=NO

GOAL=
  execute the retained 12-observation reference matrix once, retain and verify
  the raw evidence, and report the accumulated PRE_A/dynamic -> FINAL/prepared
  effect per workload as the primary PERF025 result
~~~

D3 must stop rather than enter a retry campaign if reference admission reports
`NOT_STABLE`. It must not add JFR, IGV, GC tuning, affinity experiments, longer
warmup, or repeated smoke/reference runs merely to obtain a preferred result.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos-benchmarks@e0bc5a5c2f8c0b373405c75bf9df9e5a54e94b2b` — D2 harness publication.
- `docs/project/evidence/PERF025/PERF025_D1_FINAL_REUSABLE_CALL_BENCHMARK_EVIDENCE_DESIGN.md`.
