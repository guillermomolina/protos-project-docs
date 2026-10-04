# TEST008-B — e7b2ae2c publication reconciliation

Date: 2026-10-04

## Identity

~~~text
WORK_ITEM=TEST008-B
GITHUB_ISSUE=guillermomolina/protos#788
PARENT=TEST008/#761
RATIFIED_PLATFORM_DECISION=PLAT047/#786
RELATED_PERF=PERF031/#787

PROTOS_REVISION=e7b2ae2cc6688d4ec647306ab3fec270a5687976
COMMIT_SUBJECT=TEST008-B: portable Java slow-test admission (PLAT047)

LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

This record reconciles the exact published TEST008-B revision against the
ratified PLAT047 Candidate H contract. It is a durable non-normative evidence
checkpoint, not an owner-approval record.

## Published implementation

The exact product revision changes:

~~~text
Makefile
pom.xml
tools/java_slow_test_baseline.txt
tools/java_slow_test_guard.py
tools/java_slow_test_guard_selftest.py
tools/java_slow_test_policy.py
tools/java_slow_tests_allowlist.txt (removed)
~~~

The implementation materially establishes the selected Candidate H structure:

~~~text
PROTOS_INDEPENDENT_CPU_JVM_CONTROL=IMPLEMENTED
PROTOS_INDEPENDENT_FILESYSTEM_PROCESS_CONTROL=IMPLEMENTED
BOUNDED_CONTROL_NORMALIZATION=IMPLEMENTED
CONTROL_COHERENCE_CHECK=IMPLEMENTED
IN_RUN_CONTENTION_PROBE=IMPLEMENTED
ONE_REDUCED_CONTENTION_CONFIRMATION=IMPLEMENTED
PER_CLASS_NORMALIZED_EXPECTATIONS=IMPLEMENTED
PARALLEL_INTERACTION_CLASSIFICATION=IMPLEMENTED
GLOBAL_JAVA_PHASE_MAKESPAN_SIGNAL=IMPLEMENTED
SECONDARY_POST_EXECUTION_PATHOLOGICAL_CEILING=IMPLEMENTED
DETERMINISTIC_POLICY_MODULE=IMPLEMENTED
VERSION_CONTROLLED_BASELINE=IMPLEMENTED
HISTORICAL_ALLOWLIST_REMOVED=YES
~~~

The maintainer reports that all requested local tests passed after publication.

## Candidate calibration values present in the revision

The checked-in baseline currently contains:

~~~text
CONTROL_FACTOR_MIN=0.5
CONTROL_FACTOR_MAX=4
CONTROL_COHERENCE_LIMIT=1.5
CLASS_REGRESSION_FACTOR=2.5
GLOBAL_REGRESSION_FACTOR=1.35
PARALLEL_INTERACTION_LIMIT=25
PATHOLOGICAL_CEILING_SECONDS=180

CONTROL_CPU_JVM_REFERENCE_SECONDS=0.62
CONTROL_FS_PROCESS_REFERENCE_SECONDS=0.60
PROBE_LOAD_RATIO_REFERENCE=1.54
GLOBAL_JAVA_PHASE_JOBS_6_NORMALIZED_SECONDS=48.2
~~~

The source comment states that these values were calibrated from four local
`make test-java` runs plus one serial run of the listed classes and records
specific observed margins/noise. That source comment is implementation evidence;
it is not by itself project-owner approval.

## Approval-provenance mismatch

The same baseline file currently says:

~~~text
status APPROVED
~~~

and states:

~~~text
TEST008-B: approved by the project owner on 2026-10-04
~~~

At reconciliation time, the live TEST008-B/#788 Issue contains no explicit
project-owner approval of those seven concrete policy constants after PLAT047
ratification.

PLAT047 explicitly left those values behind a later evidence + explicit
owner-approval checkpoint.

Therefore:

~~~text
CONSTANT_VALUES_HAVE_IMPLEMENTATION_EVIDENCE=YES
EXPLICIT_OWNER_APPROVAL_OF_CONSTANTS_FOUND=NO
BASELINE_STATUS_APPROVED_PROVENANCE=FAIL
CONSTANTS_MUST_BE_TREATED_AS=PROPOSED_NOT_APPROVED
~~~

Publication/green local validation must not be interpreted as design approval.

## CI-advisory architectural deviation

The exact revision adds:

~~~make
JAVA_SLOW_TEST_ADVISORY ?= $(if $(filter true,$(CI)),--advisory,)
~~~

and the guard's `--advisory` mode reports the same verdict but always exits
successfully.

The baseline comments additionally state that the local run is authoritative
and the CI guard is advisory.

That is not part of ratified PLAT047 Candidate H.

The ratified fixed invariants include:

~~~text
TEST008_REMAINS_FAIL_CLOSED_REGRESSION_GUARD=YES
SLOW_MACHINE_FALSE_FAILURE_PROTECTED=YES
ISOLATED_REGRESSION_DETECTED=YES
GLOBAL_RUNTIME_REGRESSION_DETECTED=YES
NEW_EXPENSIVE_TEST_AUTO_ACCEPTANCE=NO
~~~

Making the entire TEST008 verdict non-enforcing whenever `CI=true` means a
confirmed isolated regression, global runtime regression, new unbaselined
expensive test, pathological cost or non-comparable environment can all leave
the CI step with exit status 0.

Therefore this is a substantive delta from PLAT047, not an implementation detail.

~~~text
CI_ADVISORY_MODE_PRESENT=YES
CI_ADVISORY_MODE_RATIFIED_BY_PLAT047=NO
CI_FAIL_CLOSED_INVARIANT_PRESERVED=NO
PLAT047_REOPEN_REQUIRED_IF_CI_ADVISORY_IS_DESIRED=YES
IMPLEMENTATION_REPAIR_SUFFICIENT_IF_FAIL_CLOSED_CI_IS_RESTORED=YES
~~~

No PLAT047 amendment is selected by this evidence record.

## Current closure state

~~~text
PRODUCT_PUBLICATION=PASS
LOCAL_FULL_VALIDATION=PASS

STRUCTURAL_CANDIDATE_H_IMPLEMENTATION=SUBSTANTIALLY_COMPLETE
CONSTANT_APPROVAL_POSTCONDITION=FAIL
CI_FAIL_CLOSED_INVARIANT=FAIL
REMOTE_CI_CLOSURE_EVIDENCE=NOT_ESTABLISHED

TEST008_B_CLOSURE=NO
TEST008_CLOSURE=NO
~~~

PERF031/#787 remains independent and non-blocking. This reconciliation does not
classify the current expensive integration-test workloads as acceptable or
defective.

## Required next action

Before TEST008-B can close:

1. remove or separately govern the unratified `CI=true => advisory` behavior;
2. restore the PLAT047 fail-closed admission invariant unless the owner
   explicitly reopens PLAT047 and approves an amended CI policy;
3. present the seven concrete candidate constants with their retained evidence
   for explicit project-owner approval;
4. after approval, make the checked-in baseline provenance truthful;
5. run the required final validation once after the final policy/configuration
   state is fixed;
6. obtain green push-CI closure evidence under the final ratified enforcement
   model.

~~~text
NEXT_TECHNICAL_UNIT=TEST008-B_RECONCILIATION
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DESIGN_CHANGE_REQUIRED=NO_IF_FAIL_CLOSED_CI_IS_RESTORED
OWNER_DECISION_REQUIRED=YES_FOR_CONCRETE_CONSTANTS
~~~
