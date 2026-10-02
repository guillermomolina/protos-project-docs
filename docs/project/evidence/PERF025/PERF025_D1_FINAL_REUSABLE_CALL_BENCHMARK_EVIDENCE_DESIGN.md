# PERF025-D1 — final reusable-call benchmark evidence design

Date: 2026-10-02

## Identity and scope

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-D1
TYPE=INVESTIGATION

PRODUCT_CHANGE=NONE
HARNESS_CHANGE_DURING_D1=NONE
SHELL_EXECUTION=NO
PROJECT_COMMAND_EXECUTION=NO
REPOSITORY_MODIFICATION_DURING_INVESTIGATION=NO

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

D1 closes the bounded investigation requested after PERF025-C2D. It does not
perform timing, profiling, builds, tests, or benchmark execution. Its purpose is
to define the smallest revision-bound harness change needed to satisfy the one
remaining PERF025 acceptance criterion: retained repeated-call before/after
evidence for PERF025-A and PERF025-B using the established PERF024 primitive
measurement surface.

## Current authorities inspected

The current benchmark repository authority observed during D1 is:

~~~text
REPOSITORY=guillermomolina/protos-benchmarks
HARNESS_REVISION=5486c515369c8a958c8baa6949b818dbfaf7d0f8
COMMIT_SUBJECT=PERF024: align JVM diagnostics with matrix runner
~~~

The exact current Protos closure state supplied by PERF025-C2D is:

~~~text
FINAL_PRODUCT_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
FINAL_PRODUCT_VERSION=0.3.143-SNAPSHOT
CURRENT_GUEST_CARRIER_STACK=16_MIB
DEDICATED_GUEST_CARRIER=RETAINED
~~~

D1 also reconciled the live PERF025/#758 acceptance contract and the current
benchmark implementation. The harness HEAD did not advance from the C2D handoff.

## Why a harness change is required

At the exact benchmark revision above, both current Protos repeated-call routes
measure the dynamic API:

~~~text
TruffleJvmRunner:
  session.invokeTopLevel("run")

ProtosJvmVariantRunner:
  session.invokeTopLevel("run")
~~~

PERF025-B introduced the supported prepared route:

~~~text
prepared = session.prepareTopLevel("run")
prepared.invoke()
~~~

but the current harness never measures it. Therefore existing PERF024 primitive
results cannot be claimed as PERF025-B prepared-callable evidence.

The earlier C2D routing note that a harness change was not expected was
preliminary. D1 supersedes that expectation with direct inspection:

~~~text
HARNESS_CHANGE_REQUIRED=YES
PRODUCT_CHANGE_REQUIRED=NO
~~~

## Exact comparison matrix

The smallest attribution-clean comparison points are:

~~~text
A_BASE_REVISION=f3c44554ddfb9004c43dbde5197b9805990f8a4a
A_BASE_MODE=dynamic

A_CANDIDATE_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6
A_CANDIDATE_MODE=dynamic

B_BASE_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6
B_BASE_MODE=dynamic

B_CANDIDATE_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
B_CANDIDATE_MODE=prepared

FINAL_CONTROL_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
FINAL_CONTROL_MODE=prepared
~~~

Interpretation is intentionally separated:

~~~text
A_DELTA=
  pre-A dynamic
  versus
  post-A dynamic

B_DELTA=
  post-A dynamic
  versus
  post-B prepared

COMBINED_PRE_A_TO_POST_B_DELTA=
  pre-A dynamic
  versus
  post-B prepared

FINAL_CURRENT_STATE=
  final C2D revision in prepared mode
  contextual closure control only
~~~

The final control must not be used to attribute A or B because it includes later
PERF025 work after B.

## Minimal workload set

The selected retained workload subset is:

~~~text
primitive-return-literal
primitive-closure-call
primitive-method-call
~~~

These three are sufficient for the remaining PERF025 acceptance question:

- primitive-return-literal maximizes sensitivity to fixed embedding overhead;
- primitive-closure-call retains guest Closure-call work; and
- primitive-method-call adds ordinary guest dispatch.

The remaining primitive local-read/local-write/integer-add cases remain valid
PERF024 workloads but are not required to establish this bounded A/B embedding
effect. Results must remain per-workload rather than being converted into one
universal Protos performance claim.

The existing primitive measurement unit remains:

~~~text
SAMPLE_CALLS=100
~~~

## Measurement and stability contract

D2 must reuse the bounded PERF024 JVM reference policy rather than the older
lightweight A/B timing loop:

~~~text
REFERENCE_WARMUP_ITERATIONS=60
REFERENCE_STEADY_ITERATIONS=10

STABILITY_WINDOW=5
STABILITY_MEDIAN_DRIFT_PCT_MAX=15
STABILITY_MAD_PCT_MAX=20
STABILITY_MAX_INTERNAL_GAP_PCT=20
STABILITY_MIN_GAP_CLUSTER_SIZE=3
~~~

A rejected measurement remains:

~~~text
NOT_STABLE
~~~

It must not trigger an open-ended calibration, JFR, affinity, GC, quiescence,
warmup-escalation, or long reference campaign.

## Prepared-call timing boundary

Preparation belongs to setup, outside the repeated timed region:

~~~text
SETUP:
  session.open(...)
  prepared = session.prepareTopLevel("run")

TIMED_CALL:
  prepared.invoke()
~~~

D2 must use direct compile-time invocation of the product API. It must not put
reflection, Method.invoke, per-call symbol lookup, or another adapter with
material primitive-scale overhead into the prepared timed region.

Historical pre-B revisions must not be forced to resolve symbols introduced by
PERF025-B. The implementation therefore needs a revision-compatible separation
between dynamic measurement and the prepared-only code path.

## Measurement identity and cache contract

Every accepted raw observation must distinguish at least:

~~~text
HARNESS_REVISION
PROTOS_REVISION
PROTOS_VERSION
CORE_SHA256
RUN_MODE=dynamic|prepared
WORKLOAD
SOURCE_SHA256
JAVA_VERSION
GRAALVM_VERSION
CPU
CPU_SIBLINGS
CPU_MODEL
ARCHITECTURE
KERNEL
WARMUP_POLICY
STEADY_POLICY
SAMPLE_CALLS
RESULT
~~~

RUN_MODE must be part of the cache identity/key so dynamic and prepared
observations cannot collide.

Reference evidence must come from a clean published harness revision.

## Retention decision

New PERF025 measurements remain benchmark-harness-owned raw evidence. Historical
PERF024/PERF023 retained evidence must not be rewritten.

The current generic retention helper still contains PERF023-specific expected
class counts, so D2 must make the smallest compatible retention extension needed
for the new exact matrix while preserving existing PERF023 verification.

The final retained PERF025 measurement should have:

~~~text
4 revision/mode points
x 3 workloads
= 12 raw reference observations
~~~

A later PERF025 durable closure record may reference the exact benchmark harness
revision, retained manifest, and raw hashes rather than copying raw timing logs
into this repository.

## D1 decision packet

~~~text
PERF025_D1_STATUS=COMPLETE

HARNESS_CHANGE_REQUIRED=YES
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-benchmarks

A_BASE_REVISION=f3c44554ddfb9004c43dbde5197b9805990f8a4a
A_CANDIDATE_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6

B_BASE_REVISION=d0045353d834257b5fb80581846b32aebd43c7e6
B_CANDIDATE_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66

FINAL_CONTROL_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985

INVOCATION_MODES=dynamic,prepared
SELECTED_WORKLOAD_COUNT=3
EXPECTED_REFERENCE_OBSERVATIONS=12

OPEN_ENDED_PROFILING_REQUIRED=NO
PRODUCT_CHANGE_REQUIRED=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_ISSUE_REQUIRED=NO
~~~

## Next slice

~~~text
NEXT_SLICE=PERF025-D2
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos-benchmarks

GOAL=
  implement the smallest revision-compatible dynamic/prepared exact-revision
  primitive measurement capability and retention contract needed for the final
  PERF025 A/B evidence

REFERENCE_MEASUREMENT_DURING_D2=NO
PRODUCT_CHANGE=NO
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

D2 should implement and validate/smoke the harness capability only. Retained
reference measurement belongs to the next bounded measurement slice from the
published clean D2 harness revision.

## Cross references

- guillermomolina/protos#758 — PERF025.
- guillermomolina/protos-benchmarks@5486c515369c8a958c8baa6949b818dbfaf7d0f8 — harness authority inspected by D1.
- docs/project/evidence/PERF025/PERF025_A_DIRECT_ROOT_TASK_DISPATCH.md.
- docs/project/evidence/PERF025/PERF025_B_PREPARED_TOP_LEVEL_CALLABLE.md.
- docs/project/evidence/PERF025/PERF025_C2D_GUEST_CARRIER_STACK_BUDGET_REDUCTION.md.
