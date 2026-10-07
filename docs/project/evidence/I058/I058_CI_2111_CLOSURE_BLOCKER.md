# I058 — CI #2111 closure blocker

Date: 2026-10-03

This snapshot records the exact remote-CI result for the published I058 product revision. It supplements, and does not rewrite, the earlier implementation checkpoint.

## Stable identities

```text
PROTOS_REVISION=1ad6c5b5566a7e2c639a27ae95a8a545b03fd402
GITHUB_ISSUE=guillermomolina/protos#661
CI_RUN=2111
CI_RUN_ID=37140570878
CI_JOB_ID=111253981075
CI_CONCLUSION=FAILURE
RELATED_GUARD_OWNER=TEST008/guillermomolina/protos#761
```

## What passed

The CI `Run repository tests` step completed the Java test executions with no semantic test failures:

```text
PARALLEL_JAVA_TESTS_RUN=2402
PARALLEL_JAVA_FAILURES=0
PARALLEL_JAVA_ERRORS=0
PARALLEL_JAVA_SKIPPED=1
SERIAL_JAVA_TESTS_RUN=7
SERIAL_JAVA_FAILURES=0
SERIAL_JAVA_ERRORS=0
SERIAL_JAVA_SKIPPED=0
```

The maintainer independently reported that the required local tests passed before publication.

## Exact CI failure

The job failed only after successful Java test execution when the TEST008 slow-test post-validation guard ran:

```text
JAVA_SLOW_TEST_GUARD=FAIL
REPORTED_TEST_CLASSES=499
ALLOWLISTED_SLOW_TESTS=7
UNALLOWLISTED_SLOW_TESTS=10
OVER_BUDGET_SLOW_TESTS=6
MAKE_RESULT=test-java failed with exit code 2 from the guard
```

Representative over-budget entries included `ProtosExternalPackagePlanningPreflightTest`, `ProtosPackageExecutionPlanAdapterTest`, `ProtosWorkspacePackageAuthorityIsolationIntegrationTest`, and `ProtosWorkspacePackagePreflightTest`. The guard also reported newly unallowlisted slow classes in package/test-tool/TOML coverage.

This failure is owned by TEST008/#761's cross-repository Java slow-test admission guard; the CI log contains no failing I058 semantic test. Nevertheless, #661 explicitly requires green repository CI before closure, so the failed exact-revision run cannot satisfy that gate.

## Remaining I058 closure evidence

Independent of the CI guard failure, #661 requires execution evidence for the migrated canonical `parallel-array-map` benchmark. The product revision migrates the benchmark source to `Arrays.parallelMap(...)`, but no benchmark execution result or valid before/after comparison has yet been supplied to the coordinating record.

```text
I058_PRODUCT_IMPLEMENTATION=COMPLETE
MAINTAINER_REPORTED_TESTS=PASS
EXACT_REVISION_CI=FAIL_TEST008_SLOW_GUARD
I058_SEMANTIC_TEST_FAILURE_OBSERVED=NO
BENCHMARK_EXECUTION_EVIDENCE=NOT_RECORDED
ISSUE_661_CLOSURE_READY=NO
```
