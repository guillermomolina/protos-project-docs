# PLAT047 — Portable Java slow-test admission architecture

Status: **RATIFIED**

Selected architecture: **Candidate H — minimal hybrid portable admission**.

Approval: explicit project-owner approval on 2026-10-03:

~~~text
aceptada la propuesta
~~~

Decision Issue: `guillermomolina/protos#786`

Parent validation owner: `TEST008 / guillermomolina/protos#761`

Triggering investigation: `TEST008-A / guillermomolina/protos#785`

Independent performance follow-up: `PERF031 / guillermomolina/protos#787`

Product authority inspected for the decision packet:

~~~text
PROTOS_REVISION=6ca7cee5c3a09112268b7a04ed6922086a994f35
~~~

Nature: durable non-normative validation/platform architecture decision. Observable
Protos language and Standard Library semantics remain unchanged.

## Decision

TEST008 must no longer treat raw Surefire class wall time observed in the
ordinary class-parallel JUnit lane as a portable intrinsic per-class cost.

The selected architecture separates five responsibilities:

~~~text
1. ordinary-test workload appropriateness
2. portable environment normalization
3. per-class regression confirmation
4. broad/global Protos regression detection
5. pathological absolute cost rejection
~~~

Each mechanism is present because removing it loses a fixed PLAT047 invariant.
The architecture does not attempt to turn ordinary correctness CI on
uncontrolled hosted runners into benchmark-grade performance infrastructure.

## Selected Candidate H topology

### A. Ordinary-test workload policy

Ordinary correctness tests must use workloads proportional to the correctness
property being proved.

Benchmark/stress-scale workloads, unnecessarily repeated full bootstrap,
duplicated expensive integration proof, or other disproportionate work must be
reduced or routed to a purpose-specific surface when smaller evidence proves the
same invariant.

Deterministic work-unit limits may be used only where a test naturally exposes
such units. PLAT047 does not create a generic work-unit framework.

PERF031 remains the independent owner for classifying and remediating current
expensive Java integration tests.

### B. Minimal Protos-independent same-run control bundle

Environment normalization uses a small control bundle independent of Protos.

The minimum architecture requires:

~~~text
CPU/JVM-oriented independent control
+
filesystem/process-oriented independent control
~~~

A single CPU control is insufficient for the current suite because ordinary Java
validation includes filesystem-heavy, process-heavy, JIT/GC-sensitive and
class-parallel integration work.

The controls are an environment-coherence and coarse normalization mechanism.
They are not a claim that one synthetic workload exactly models every test.

No per-test weighted control taxonomy is selected.

### C. Bounded normalization domain

A common machine factor may be used only when the independent controls are both:

~~~text
BOUNDED
MUTUALLY_COHERENT
~~~

If the controls disagree materially or fall outside the approved normalization
domain, the run is not silently normalized and does not fall back to the old raw
absolute-budget policy.

The required result is:

~~~text
ENVIRONMENT_NOT_COMPARABLE
=> ERROR
~~~

rather than a false Protos performance FAIL or automatic acceptance.

Exact numerical bounds and coherence tolerances are implementation constants
that require evidence and explicit owner review in TEST008-B. PLAT047 ratifies
their semantics, not invented values.

### D. Per-class normalized canonical cost plus one bounded confirmation

Raw parallel Surefire class wall time is suspicion evidence, not the final
portable per-class performance verdict.

When a class is suspicious, TEST008 may perform exactly one bounded diagnostic
confirmation under reduced/known contention.

The confirmed cost is compared with an explicit version-controlled canonical
expectation after valid environment normalization.

There is no rerun-until-green behavior.

A new class must not acquire an expensive baseline from its first observation.
Intentional baseline creation or update remains an explicit reviewed project
change.

### E. Parallel-interaction preservation

A reduced-contention or serial confirmation that passes must not automatically
erase a pathological signal that exists only in the ordinary parallel lane.

The implementation must preserve both observations and distinguish:

~~~text
isolated confirmed regression
parallel-interaction regression
ordinary scheduling/contention noise
non-comparable environment
~~~

A parallel-only pathological delta with coherent machine controls remains
fail-visible and reviewable.

### F. Independent global regression signal

A broad Protos/runtime regression must not disappear because the suite is used
to normalize itself.

The global regression gate therefore uses a Protos-independent normalization of
the overall Java validation phase wall time or equivalent phase-level makespan.

The architecture expressly rejects as a global gate:

~~~text
test / suite median
test / robust aggregate of the same regressing suite
sum(class wall times)
~~~

Concurrent class wall intervals overlap, so summing Surefire class wall times is
neither phase elapsed time nor attributed CPU consumption.

### G. Secondary absolute pathological ceiling

Absolute time remains only as a post-execution secondary hard limit for a
minutes-scale or otherwise clearly pathological ordinary test.

Its meaning is:

~~~text
this cost is incompatible with ordinary validation regardless of host speed
~~~

not:

~~~text
this test regressed by a small percentage
~~~

The ceiling remains post-execution. PLAT047 does not authorize test killing or
timeouts.

Its exact numeric value is deferred to TEST008-B evidence and owner review.

## Fixed invariants

Candidate H preserves all PLAT047 fixed invariants:

~~~text
CURRENT_RUN_EVIDENCE_ONLY=YES
EXACT_REVIEWABLE_TEST_IDENTITY=YES
AUTOMATIC_ALLOWLIST_GROWTH=NO
AUTOMATIC_BASELINE_GROWTH=NO
NEW_EXPENSIVE_TEST_AUTO_ACCEPTANCE=NO

TIMEOUT_BASED_KILLING=NO
TEST_SKIPPING=NO
ALLOW_FAILURE=NO
ASSERTION_WEAKENING=NO
COVERAGE_REDUCTION=NO
JAVA_TEST_JOBS_RELAXATION_FOR_GREEN_CI=NO

MAVEN_MAKE_FAILURE_PROPAGATION=PRESERVED
CLEAR_MACHINE_READABLE_DIAGNOSTICS=REQUIRED

SLOW_MACHINE_FALSE_PERFORMANCE_FAILURE_PROTECTED=YES
ISOLATED_REGRESSION_DETECTED=YES
GLOBAL_RUNTIME_REGRESSION_DETECTED=YES
PARALLEL_INTERACTION_REGRESSION_DETECTED=YES
PATHOLOGICAL_COST_DETECTED=YES

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

## Adversarial behavior

### Host approximately 2x slower

If both independent controls and ordinary Protos work slow coherently by
approximately the same factor, normalization removes the host-level change.

Result: no false Protos regression.

### Isolated class approximately 4x slower

If controls are stable, other tests remain normal and the bounded confirmation
retains the material slowdown, the class fails the per-class regression gate.

### Broad 40–60% Protos slowdown

If independent controls remain stable while the normalized Java phase makespan
materially rises, the independent global gate fails.

Suite-relative self-normalization is not permitted to hide the regression.

### Filesystem-only host contention

A normal CPU/JVM control and materially slow filesystem/process control make the
environment incoherent for one scalar normalization.

One bounded confirmation may distinguish transient contention. Persistently
incoherent controls produce ERROR rather than product FAIL.

### New expensive test

A new class with no canonical expectation cannot create one automatically from
the current run.

It must be reduced, routed elsewhere, or receive an explicit reviewed baseline
or exception.

### Intentional larger integration test

A legitimate expansion requires an explicit version-controlled baseline update
with rationale and owner authority. A failing run never regenerates its own
baseline.

### Several-minute hang-like test

The secondary unnormalized pathological ceiling fails after execution.

No timeout semantics are introduced.

### Parallel-only regression

A low-contention confirmation is diagnostic but not automatic acquittal.
Pathological ordinary-parallel behavior remains independently fail-visible when
controls are coherent.

## Baseline governance

~~~text
WHAT_IS_BASELINED=
  REVIEWED_NORMALIZED_PER_CLASS_CANONICAL_EXPECTATIONS
  + REVIEWED_NORMALIZED_GLOBAL_JAVA_PHASE_EXPECTATION
  + NATURAL_WORK_UNIT_LIMITS_ONLY_WHERE_JUSTIFIED

BASELINE_UPDATE_AUTHORITY=EXPLICIT_PROJECT_OWNER_APPROVAL

AUTOMATIC_BASELINE_UPDATE=NO
AUTOMATIC_HISTORICAL_LEARNING=NO
FAILED_RUN_AUTO_ACCEPTANCE=NO
~~~

Historical measurements may be retained for diagnosis. They never silently
become policy.

## Rejected standalone candidates

### Candidate A — universal absolute wall time

Rejected as primary architecture. A fixed margin merely moves the
false-positive/false-negative boundary between faster and slower hosts.

### Candidate B — environment-specific absolute budgets

Rejected. It encodes runner/hardware accidents into policy, proliferates
profiles and does not guarantee equivalent performance under the same nominal
runner label.

### Candidate C — same-run machine calibration

Retained only as a component. One control cannot represent all current Java
resource dimensions; a large per-test control taxonomy would be
disproportionate.

### Candidate D — suite-relative normalization

Rejected as primary architecture because broad Protos regressions can inflate
their own denominator and disappear.

### Candidate E — bounded two-stage confirmation

Retained as a required component. It removes major contention false positives
but cannot solve cross-machine comparison alone.

### Candidate F — timing is not the primary ordinary-test gate

Retained as a conceptual component. Workload appropriateness and performance
regression are separate questions, but timing evidence is still required by the
TEST008 regression invariant.

### Candidate G — resource/work units

Retained only where natural. Work units can detect benchmark-scale test design
but cannot detect a runtime regression at unchanged workload size.

### Candidate I — same-host reference-revision A/B

Deferred as the preferred future escape path when Protos needs benchmark-grade
performance CI. It offers stronger measurement but requires substantially more
build/reference/orchestration cost than the current admission problem justifies.

## Strongest argument against Candidate H

Candidate H adds several policy surfaces while hosted correctness CI still
cannot become precision benchmark infrastructure.

The control bundle cannot guarantee that every arbitrary future test experiences
host conditions proportionally to the controls, and the host may change between
measurements.

The scope limit is therefore architectural:

> TEST008 detects large abnormal/regressive costs suitable for admission
> control. It does not provide precision benchmarking.

If later requirements demand reliable small-regression measurement, the project
must move toward same-host A/B or dedicated performance CI rather than
accumulating correction factors in TEST008.

## 2026-10-04 owner amendment — local authoritative gate, CI advisory

After TEST008-B was implemented and published at
`guillermomolina/protos@e7b2ae2cc6688d4ec647306ab3fec270a5687976`,
the project owner explicitly approved the exact deployed enforcement policy and
the seven concrete calibration constants:

~~~text
acepto exactamente eso
~~~

The approval referred to the explicitly presented policy:

~~~text
LOCAL_TEST008_GUARD=AUTHORITATIVE_FAIL_CLOSED
CI_TEST008_GUARD=ADVISORY

CONTROL_FACTOR_MIN=0.5
CONTROL_FACTOR_MAX=4
CONTROL_COHERENCE_LIMIT=1.5
CLASS_REGRESSION_FACTOR=2.5
GLOBAL_REGRESSION_FACTOR=1.35
PARALLEL_INTERACTION_LIMIT=25
PATHOLOGICAL_CEILING_SECONDS=180
~~~

This is a deliberate amendment to the original PLAT047 enforcement invariant,
not evidence that the original 2026-10-03 approval already contained this
exception.

Candidate H's measurement and classification architecture remains unchanged:

- Protos-independent CPU/JVM and filesystem/process controls;
- bounded/coherent machine normalization;
- one reduced-contention confirmation for per-class suspects;
- preserved parallel-interaction classification;
- independently normalized Java-phase makespan;
- secondary post-execution pathological ceiling;
- exact version-controlled canonical expectations;
- no automatic allowlist/baseline growth.

The amendment changes only where the resulting verdict is authoritative:

~~~text
LOCAL_DEVELOPER_GATE=
  FAIL_CLOSED_AND_AUTHORITATIVE

CI_GATE=
  ADVISORY_DIAGNOSTIC_ONLY
  SAME_CLASSIFICATION_LOGIC
  NONZERO_POLICY_VERDICT_NOT_ENFORCED_AS_JOB_FAILURE
~~~

Rationale accepted by the owner: the local pre-push validation is the
authoritative performance-admission gate, while hosted CI remains useful as a
different-machine observation surface without allowing runner variance to block
otherwise-correct publication.

Therefore the fixed invariant is amended from:

~~~text
TEST008_REMAINS_FAIL_CLOSED_REGRESSION_GUARD=YES_EVERYWHERE
~~~

to:

~~~text
TEST008_LOCAL_ADMISSION_REMAINS_FAIL_CLOSED=YES
TEST008_CI_ADMISSION_IS_ADVISORY=YES
LOCAL_PRE_PUSH_VALIDATION_IS_AUTHORITATIVE=YES
~~~

This amendment does not authorize:

- automatic baseline growth;
- automatic acceptance of new expensive local tests;
- timeouts, skips, allow-failure of Java assertions, or coverage reduction;
- CI-specific absolute budget tables;
- environment-specific replacement semantics;
- weakening Maven/Make failure propagation for the underlying test execution.

The `--advisory` behavior applies only to the slow-test admission verdict after
the ordinary CI test phases themselves have completed. Ordinary build/test
failures remain failures.

Exact implementation evidence:

~~~text
PROTOS_REVISION=e7b2ae2cc6688d4ec647306ab3fec270a5687976
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
CI_RUN_NUMBER=2121
CI_RUN_ID=37176553510
CI_JOB_ID=111360243076
CI_CONCLUSION=SUCCESS
CI_RUN_REPOSITORY_TESTS_STEP=SUCCESS
~~~

The exact published baseline already contains the approved values, so this owner
amendment requires no further product-code change.

## GITHUB021 invariant/delta consistency

The owner approved the exact Candidate H recommendation after the PLAT047-A
packet had surfaced the fixed invariants, adversarial cases, candidate
falsification, scoring and strongest argument against the recommendation.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL_TEXT="aceptada la propuesta"
OWNER_APPROVAL_DATE=2026-10-03

FIXED_PLAT047_INVARIANTS_PRESERVED=YES
NEW_HIDDEN_ARCHITECTURAL_CONSEQUENCE=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
NEW_LANGUAGE_DECISION_REQUIRED=NO

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Implementation authority

Ratification releases a bounded TEST008-B implementation in
`guillermomolina/protos`.

TEST008-B is authorized to implement Candidate H but not to invent policy
silently.

The concrete values for the following implementation constants were subsequently evidence-backed by TEST008-B and explicitly approved by the project owner on 2026-10-04:

~~~text
CONTROL_FACTOR_MIN
CONTROL_FACTOR_MAX
CONTROL_COHERENCE_LIMIT
CLASS_REGRESSION_FACTOR
GLOBAL_REGRESSION_FACTOR
PARALLEL_INTERACTION_LIMIT
PATHOLOGICAL_CEILING_SECONDS
~~~

TEST008-B must preserve the already-correct TEST008 machinery for current-run
report isolation, retained live output, exact test identity and Maven/Make failure
propagation unless a local implementation change is strictly necessary for
Candidate H.

PERF031 is non-blocking and remains independently responsible for whether
specific expensive Java integration tests are proportional to their correctness
evidence.

## Ratified result

~~~text
PLAT047_STATUS=RATIFIED
SELECTED_CANDIDATE=H_MINIMAL_HYBRID

CURRENT_ABSOLUTE_CLASS_WALL_TIME_PRIMARY_GATE=REJECTED
RAW_PARALLEL_SUREFIRE_CLASS_TIME_ROLE=SUSPICION_EVIDENCE
ABSOLUTE_TIME_ROLE=SECONDARY_HARD_LIMIT

TEST_WORKLOAD_POLICY_ROLE=ORDINARY_SUITE_APPROPRIATENESS
MACHINE_CONTROL_ROLE=BOUNDED_INDEPENDENT_NORMALIZATION_AND_ENVIRONMENT_VALIDITY
PER_CLASS_BASELINE_ROLE=EXPLICIT_VERSION_CONTROLLED_NORMALIZED_CANONICAL_COST
TWO_STAGE_CONFIRMATION_ROLE=EXACTLY_ONE_BOUNDED_CONFIRMATION
GLOBAL_REGRESSION_ROLE=INDEPENDENT_CONTROL_NORMALIZED_JAVA_PHASE_MAKESPAN
HISTORICAL_DATA_ROLE=DIAGNOSTIC_ONLY
REPEAT_MEASUREMENT_ROLE=ONE_BOUNDED_CONFIRMATION_NOT_BENCHMARK_STATISTICS

AUTOMATIC_ALLOWLIST_GROWTH=NO
AUTOMATIC_BASELINE_GROWTH=NO
CURRENT_CI_OFFENDERS_AUTOMATICALLY_ACCEPTED=NO

PERF031_DEPENDENCY=NON_BLOCKING

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
NEW_LANGUAGE_DECISION_REQUIRED=NO

IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_SLICE=TEST008-B
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

LOCAL_TEST008_GUARD=AUTHORITATIVE_FAIL_CLOSED
CI_TEST008_GUARD=ADVISORY

CONTROL_FACTOR_MIN=0.5
CONTROL_FACTOR_MAX=4
CONTROL_COHERENCE_LIMIT=1.5
CLASS_REGRESSION_FACTOR=2.5
GLOBAL_REGRESSION_FACTOR=1.35
PARALLEL_INTERACTION_LIMIT=25
PATHOLOGICAL_CEILING_SECONDS=180
~~~

## Evidence and references

- `guillermomolina/protos#786` — PLAT047 decision Issue.
- `guillermomolina/protos#785` — TEST008-A trigger investigation.
- `guillermomolina/protos#761` — TEST008 implementation owner.
- `guillermomolina/protos#787` — PERF031 independent expensive-test follow-up.
- `docs/project/evidence/PLAT047/PLAT047_A_PORTABLE_JAVA_SLOW_TEST_ADMISSION_RATIFICATION.md`.
- `docs/project/evidence/TEST008/TEST008_A_CI_LOCAL_TIMING_ADMISSION_INVESTIGATION.md`.
