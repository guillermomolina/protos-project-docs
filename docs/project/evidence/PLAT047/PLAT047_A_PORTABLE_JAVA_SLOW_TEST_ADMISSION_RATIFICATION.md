# PLAT047-A — portable Java slow-test admission ratification evidence

Date: 2026-10-03

## Identity

~~~text
WORK_ITEM=PLAT047
SLICE=PLAT047-A
DECISION_ISSUE=guillermomolina/protos#786
PARENT_TEST_OWNER=TEST008/#761
TRIGGER=TEST008-A/#785
RELATED_PERF=PERF031/#787

PROTOS_REVISION=6ca7cee5c3a09112268b7a04ed6922086a994f35
~~~

This record retains the owner-approval and decision-closure evidence for the
exhaustive PLAT047-A investigation.

## Established trigger

TEST008-A established that the current portable-admission assumption is false:

~~~text
CURRENT_ABSOLUTE_10S_POLICY_PORTABLE=NO
CURRENT_PER_CLASS_ABSOLUTE_BUDGETS_PORTABLE=NO
ROOT_CAUSE_CLASSIFICATION=MIXED_CAUSES
~~~

Repeated CI showed Java assertions passing while the raw wall-clock guard failed
across a broad set of classes. The investigation treated that as evidence of a
model defect, not as automatic evidence that every current offender is
acceptable.

## PLAT047-A recommendation presented for approval

The exhaustive decision packet recommended:

~~~text
RECOMMENDED_CANDIDATE=H_MINIMAL_HYBRID
RECOMMENDATION_STATUS=PROPOSED_NOT_RATIFIED
~~~

The recommendation separated:

~~~text
ordinary-test workload appropriateness
independent environment normalization
per-class normalized canonical expectations
exactly one bounded confirmation
independent global Java-phase regression signal
secondary unnormalized pathological hard ceiling
~~~

It explicitly rejected automatic allowlist/baseline growth, suite-relative
self-normalization as the global gate, environment-specific budget tables,
rerun-until-green, timeout semantics, assertion weakening, coverage reduction
and changing JAVA_TEST_JOBS merely to get CI green.

## Owner approval

The project owner explicitly approved the presented proposal on 2026-10-03:

~~~text
aceptada la propuesta
~~~

The approval occurred after the packet had surfaced:

- the exact current TEST008/JUnit/Surefire topology;
- portable hosted-runner limitations;
- mature performance-system prior art;
- Candidates A through I;
- adversarial Cases 1 through 8;
- full GITHUB010 scoring for the surviving complete candidates;
- the strongest argument against Candidate H;
- anti-overengineering and future escape paths; and
- the exact implementation authority requested if approved.

Therefore:

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=H_MINIMAL_HYBRID
OWNER_APPROVAL_REQUIRED=YES
OWNER_APPROVAL_RECEIVED=YES
~~~

## Invariant/delta consistency

No invariant from PLAT047/#786 is reopened or weakened by Candidate H.

~~~text
CURRENT_RUN_EVIDENCE_ONLY=PRESERVED
EXACT_REVIEWABLE_TEST_IDENTITY=PRESERVED
NO_AUTOMATIC_ALLOWLIST_GROWTH=PRESERVED
NO_AUTOMATIC_BASELINE_GROWTH=PRESERVED
NO_SILENT_NEW_EXPENSIVE_TEST_ACCEPTANCE=PRESERVED
NO_TIMEOUT_SKIP_ALLOW_FAILURE=PRESERVED
NO_ASSERTION_OR_COVERAGE_WEAKENING=PRESERVED
MAVEN_MAKE_FAILURE_PROPAGATION=PRESERVED

SLOW_MACHINE_FALSE_FAILURE_PROTECTED=YES
ISOLATED_TEST_REGRESSION_DETECTED=YES
GLOBAL_RUNTIME_REGRESSION_DETECTED=YES
PARALLEL_INTERACTION_REGRESSION_DETECTED=YES
PATHOLOGICAL_COST_DETECTED=YES

JAVA_TEST_JOBS_POLICY_REOPENED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
NEW_LANGUAGE_DECISION_REQUIRED=NO

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Ratified architecture

~~~text
RAW_PARALLEL_CLASS_WALL_TIME=INITIAL_SUSPICION_ONLY
MACHINE_CONTROL=MINIMAL_PROTOS_INDEPENDENT_CPU_JVM_PLUS_FILESYSTEM_PROCESS_BUNDLE
NORMALIZATION=BOUNDED_AND_ONLY_WHEN_CONTROLS_ARE_COHERENT
INCOHERENT_ENVIRONMENT=ERROR_NOT_PRODUCT_FAIL_AND_NO_ABSOLUTE_FALLBACK

PER_CLASS_DECISION=
  EXPLICIT_NORMALIZED_CANONICAL_EXPECTATION
  + EXACTLY_ONE_BOUNDED_REDUCED_CONTENTION_CONFIRMATION

PARALLEL_ONLY_PATHOLOGY=REMAINS_FAIL_VISIBLE

GLOBAL_REGRESSION=
  INDEPENDENT_CONTROL_NORMALIZED_JAVA_PHASE_MAKESPAN

SUM_CLASS_WALL_TIMES_AS_GLOBAL_METRIC=REJECTED
SUITE_RELATIVE_SELF_NORMALIZATION_AS_GLOBAL_GATE=REJECTED

ABSOLUTE_TIME_ROLE=SECONDARY_POST_EXECUTION_PATHOLOGICAL_HARD_LIMIT
~~~

## Baseline authority

~~~text
BASELINE_UPDATE_AUTHORITY=EXPLICIT_PROJECT_OWNER_APPROVAL
NEW_TEST_AUTO_BASELINE=NO
FAILED_RUN_AUTO_BASELINE=NO
HISTORICAL_AUTO_LEARNING=NO
CURRENT_CI_OFFENDERS_AUTOMATICALLY_ACCEPTED=NO
~~~

Concrete normalization and regression constants were intentionally not invented
by the architecture investigation.

They remain an implementation evidence gate:

~~~text
CONTROL_FACTOR_MIN=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
CONTROL_FACTOR_MAX=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
CONTROL_COHERENCE_LIMIT=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
CLASS_REGRESSION_FACTOR=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
GLOBAL_REGRESSION_FACTOR=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
PARALLEL_INTERACTION_LIMIT=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
PATHOLOGICAL_CEILING_SECONDS=PENDING_TEST008_B_EVIDENCE_AND_OWNER_APPROVAL
~~~

## Routing after approval

~~~text
PLAT047_STATUS=RATIFIED
TEST008_B_IMPLEMENTATION_AUTHORIZED=YES
TEST008_B_IMPLEMENTATION_REPOSITORY=guillermomolina/protos
PERF031_DEPENDENCY=NON_BLOCKING
PERF031_STATUS=INDEPENDENT
~~~

TEST008-B must assume the product repository is current HEAD at execution time
and must derive all implementation details from that local HEAD plus this
ratified architecture. It must not depend on another local/private repository or
web lookup.

## Closure evidence

~~~text
OWNER_APPROVAL=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REQUIRED_DURABLE_PUBLICATION=PENDING_THIS_RECORD_PUBLICATION
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
IMPLEMENTATION_AUTHORIZED=YES
NEXT_SLICE=TEST008-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~
