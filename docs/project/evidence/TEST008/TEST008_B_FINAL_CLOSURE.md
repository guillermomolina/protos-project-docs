# TEST008-B — final closure evidence

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
~~~

## Validation

The maintainer explicitly reported the full local validation green for the
published revision.

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

Exact push-CI evidence was then verified from GitHub Actions:

~~~text
CI_WORKFLOW=CI
CI_RUN_NUMBER=2121
CI_RUN_ID=37176553510
CI_EVENT=push
CI_HEAD_SHA=e7b2ae2cc6688d4ec647306ab3fec270a5687976
CI_STATUS=completed
CI_CONCLUSION=success

CI_JOB_ID=111360243076
CI_JOB_NAME=test
CI_JOB_CONCLUSION=success
CI_RUN_REPOSITORY_TESTS_STEP=success
~~~

## Implemented Candidate H architecture

The published revision implements:

~~~text
PROTOS_INDEPENDENT_CPU_JVM_CONTROL=YES
PROTOS_INDEPENDENT_FILESYSTEM_PROCESS_CONTROL=YES
BOUNDED_CONTROL_NORMALIZATION=YES
CONTROL_COHERENCE_CHECK=YES
IN_RUN_CONTENTION_PROBE=YES

RAW_PARALLEL_CLASS_TIME_IS_SUSPICION_EVIDENCE=YES
ONE_REDUCED_CONTENTION_CONFIRMATION=YES
EXACT_PER_CLASS_CANONICAL_EXPECTATIONS=YES
PARALLEL_INTERACTION_CLASSIFICATION=YES

GLOBAL_JAVA_PHASE_MAKESPAN_SIGNAL=YES
SUITE_SELF_NORMALIZATION_GLOBAL_GATE=NO
SUM_CLASS_WALL_TIMES_GLOBAL_GATE=NO

SECONDARY_POST_EXECUTION_PATHOLOGICAL_CEILING=YES
AUTOMATIC_BASELINE_GROWTH=NO
HISTORICAL_AUTO_LEARNING=NO
~~~

The historical absolute allowlist file is removed and replaced by the reviewed
baseline representation.

## Owner-approved enforcement amendment

The published implementation uses:

~~~text
LOCAL_TEST008_GUARD=AUTHORITATIVE_FAIL_CLOSED
CI_TEST008_GUARD=ADVISORY
~~~

The original PLAT047 ratification did not contain that CI exception. During
post-publication reconciliation the project owner was shown the exact difference
and explicitly approved it on 2026-10-04:

~~~text
acepto exactamente eso
~~~

The same approval explicitly selected the exact deployed constants:

~~~text
CONTROL_FACTOR_MIN=0.5
CONTROL_FACTOR_MAX=4
CONTROL_COHERENCE_LIMIT=1.5
CLASS_REGRESSION_FACTOR=2.5
GLOBAL_REGRESSION_FACTOR=1.35
PARALLEL_INTERACTION_LIMIT=25
PATHOLOGICAL_CEILING_SECONDS=180
~~~

The calibration references already published in the baseline are:

~~~text
CONTROL_CPU_JVM_REFERENCE_SECONDS=0.62
CONTROL_FS_PROCESS_REFERENCE_SECONDS=0.60
PROBE_LOAD_RATIO_REFERENCE=1.54
GLOBAL_JAVA_PHASE_JOBS_6_NORMALIZED_SECONDS=48.2
~~~

Exact reviewed per-class normalized expectations:

~~~text
com.guillermomolina.protos.cli.ProtosTestToolFileSelectionPublicIntegrationTest 3.8
com.guillermomolina.protos.cli.ProtosWorkspaceRunCliTest 1.6
com.guillermomolina.protos.conformance.ProtosFilesystemLibraryConformanceTest 1.0
com.guillermomolina.protos.execution.ProtosExternalPackagePlanningPreflightTest 5.7
com.guillermomolina.protos.execution.ProtosPackageExecutionPlanAdapterTest 2.0
com.guillermomolina.protos.execution.ProtosWorkspacePackageAuthorityIsolationIntegrationTest 1.3
com.guillermomolina.protos.execution.ProtosWorkspacePackagePreflightTest 1.0
~~~

The owner amendment is retained in the maintained platform decision record:

`docs/project/decisions/platform/PLAT047_PORTABLE_JAVA_SLOW_TEST_ADMISSION_ARCHITECTURE.md`.

No product-code change is required after this approval because
`e7b2ae2c` already contains the exact approved policy and values.

## CI advisory boundary

CI advisory mode applies to the TEST008 slow-admission verdict only.

It does not convert underlying Java/Maven/Make assertion/build failures to
success. Push CI #2121 confirms that the ordinary repository test step itself
completed successfully at the exact product revision.

The owner-approved operational contract is:

~~~text
LOCAL_PRE_PUSH_TEST008_ADMISSION=AUTHORITATIVE
LOCAL_SLOW_TEST_FAIL=NONZERO
LOCAL_SLOW_TEST_ERROR=NONZERO

CI_TEST_EXECUTION_FAILURE=NONZERO
CI_TEST008_SLOW_ADMISSION_VERDICT=ADVISORY
CI_TEST008_DIAGNOSTICS=RETAINED
~~~

## Closure

~~~text
PLAT047_OWNER_AMENDMENT=APPROVED
CONCRETE_CONSTANT_OWNER_APPROVAL=PASS
BASELINE_APPROVAL_PROVENANCE=PASS

PRODUCT_PUBLICATION=PASS
LOCAL_FULL_VALIDATION=PASS
REMOTE_PUSH_CI=PASS

TEST008_B_CLOSURE=PASS
TEST008_PARENT_CLOSURE=PASS

PERF031_DEPENDENCY=NON_BLOCKING
PERF031_STATUS=INDEPENDENT_READY

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

The earlier
`TEST008_B_E7B2AE2C_PUBLICATION_RECONCILIATION.md` remains historical evidence
of the pre-approval checkpoint and is not rewritten to imply that owner approval
already existed at publication time.
