# TEST008-A — CI/local timing-admission investigation

## Status

```text
WORK_ITEM=TEST008-A
ISSUE=guillermomolina/protos#785
RESEARCH_STATUS=COMPLETE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
PRODUCT_CODE_CHANGE=NO
```

TEST008-A investigated why the Java slow-test guard is locally green but repeatedly fails on GitHub-hosted CI after the Java assertions themselves pass.

Current Protos remote HEAD at durable-record publication preparation:

```text
PROTOS_HEAD_AT_RECORDING=1e8fbb27ee3966ccc48a57e04308c17a58995bdc
```

The maintainer additionally reported that the local test suite passes. TEST008-A itself is an investigation and introduced no product change; the maintainer report is therefore retained as contextual validation rather than as implementation acceptance evidence for this research slice.

```text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

## Trigger evidence

Two push-CI checkpoints established the portability failure.

### CI #2110

```text
PROTOS_REVISION=07bf82194556fe0a318b6b751e1100458f768a25
REMOTE_CI_JAVA_ASSERTIONS=PASS
JAVA_SLOW_TEST_GUARD=FAIL
UNALLOWLISTED_SLOW_TESTS=2
OVER_BUDGET_SLOW_TESTS=5
```

### CI #2111

```text
PROTOS_REVISION=1ad6c5b5566a7e2c639a27ae95a8a545b03fd402
PARALLEL_JAVA_TESTS=2402
PARALLEL_FAILURES=0
PARALLEL_ERRORS=0
SERIAL_JAVA_TESTS=7
SERIAL_FAILURES=0
SERIAL_ERRORS=0
JAVA_SLOW_TEST_GUARD=FAIL
REPORTED_TEST_CLASSES=499
ALLOWLISTED_SLOW_TESTS=7
UNALLOWLISTED_SLOW_TESTS=10
OVER_BUDGET_SLOW_TESTS=6
```

The changing offender population across the two runs is evidence against treating each Surefire class wall-clock duration as a machine-independent intrinsic property.

## Current guard authority

At the investigated repository state:

- `tools/java_slow_test_guard.py` uses `THRESHOLD_SECONDS = 10.0`.
- `tools/java_slow_tests_allowlist.txt` stores exact fully-qualified class identities plus absolute per-class budgets.
- the retained TEST008-A baseline policy documented in that allowlist derives budgets from the highest of two local observations multiplied by 1.25 and rounded upward;
- `Makefile` defaults `JAVA_TEST_JOBS ?= 6` and runs test classes concurrently with fixed JUnit class parallelism;
- Surefire `testsuite time` therefore observes a class while it may be contending with other classes, JVM/JIT/GC work, filesystem work, and Protos Processes launched by integration tests.

## Finding

The investigation classifies the local-vs-CI discrepancy as:

```text
ROOT_CAUSE_CLASSIFICATION=MIXED_CAUSES
PRIMARY_COMPONENTS=MACHINE_FACTOR + CONTENTION_FACTOR + CLASS_PARALLEL_INTERFERENCE
AMPLIFIERS=BOOTSTRAP_COST + INTEGRATION_TEST_COST
PER_TEST_REGRESSION_EVIDENCE=NOT_ESTABLISHED
```

The central architectural finding is that an absolute wall-clock number calibrated on one host is not a portable regression signal for a class timed under contention on another host.

```text
CURRENT_ABSOLUTE_10S_POLICY_PORTABLE=NO
CURRENT_PER_CLASS_ABSOLUTE_BUDGETS_PORTABLE=NO
```

This does not mean that absolute time has no useful role. The research retains it as a possible secondary pathological ceiling rather than as the primary cross-machine admission metric.

## Candidate analysis summary

The investigation compared:

1. one universal absolute threshold;
2. absolute per-class budgets;
3. same-run machine calibration;
4. same-run suite-relative ratios;
5. per-test relative baselines normalized by a control;
6. test-cost classification and workload relocation;
7. robust statistical outlier detection;
8. separate warning / regression / hard-limit signals;
9. repeat/confirmation strategies.

The strongest research proposal is a small hybrid:

```text
PROPOSAL_STATUS=RECOMMENDED_BY_TEST008_A_NOT_RATIFIED

1. Protos-independent same-run machine-control factor
2. per-class canonical-cost relative regression gate
3. broad normalized aggregate regression gate
4. unnormalized pathological hard ceiling
5. explicit owner-reviewed exceptions only; no automatic baseline growth
```

Conceptually:

```text
machine_factor = control_current / control_reference

class failure when:
  current_class_time > canonical_class_cost * bounded_machine_factor * class_regression_factor

plus:
  a broad/global regression signal
  an absolute pathological hard ceiling
```

The proposal deliberately requires the machine control to be independent of Protos so that a broad Protos/runtime regression cannot automatically inflate its own normalization factor.

## Required design gate

Changing TEST008 from absolute-seconds admission to a normalized relative-cost architecture is a durable platform/validation architecture choice. Under `AGENTS.work/DESIGN.md`, TEST008-A cannot ratify that choice itself.

The unresolved decision is now allocated as:

```text
PLATFORM_DECISION=PLAT047
PLATFORM_ISSUE=guillermomolina/protos#786
TITLE=Machine-independent Java slow-test admission architecture
IMPLEMENTATION_AUTHORIZED=NO
TEST008_B_BLOCKED_BY=PLAT047
```

PLAT047 must perform the exhaustive GITHUB010 comparison and stop for explicit project-owner selection before TEST008-B implementation.

## Separate performance routing

TEST008-A also found that some expensive integration tests may deserve independent performance review instead of being hidden by a more tolerant guard.

That work is allocated separately as:

```text
PERFORMANCE_FOLLOWUP=PERF031
PERFORMANCE_ISSUE=guillermomolina/protos#787
TITLE=Isolate expensive Java integration-test cost from runner variance
```

Initial suspects include filesystem conformance, package planning/execution, workspace isolation/preflight, and the Package Tool/Test Tool/TOML families observed in CI. Their appearance is not itself proof of a product performance regression.

## Methodology caveat

The supplied research report explicitly records one procedural deviation: it initially used a read-only shell `grep` despite TEST008-A's no-command rule, then continued using read-only repository/web retrieval. The command did not mutate repository or environment state. This deviation is retained here rather than hidden.

The research packet also did not itself obtain authenticated raw per-class logs for CI #2110/#2111. Therefore the per-class causal classification is an architectural hypothesis based on the current guard topology, the Issue-retained CI evidence, and the changing offender populations; it is not a newly measured per-class benchmark result.

## Closure and routing

```text
TEST008_A_INVESTIGATION=COMPLETE

CURRENT_ABSOLUTE_10S_POLICY_PORTABLE=NO
CURRENT_PER_CLASS_ABSOLUTE_BUDGETS_PORTABLE=NO

ROOT_CAUSE_CLASSIFICATION=MIXED_CAUSES

RECOMMENDED_ADMISSION_MODEL=Protos-independent same-run machine-control factor + per-class normalized relative gate + broad normalized aggregate gate + unnormalized pathological hard ceiling
RECOMMENDATION_RATIFIED=NO

ABSOLUTE_TIME_ROLE=SECONDARY_HARD_LIMIT
MACHINE_NORMALIZATION_RECOMMENDED=YES
PER_TEST_RELATIVE_BASELINE_RECOMMENDED=YES
REFERENCE_CONTROL_RECOMMENDED=YES
HISTORICAL_DATA_REQUIRED_BY_PROPOSAL=NO
REPEAT_MEASUREMENT_REQUIRED_BY_PROPOSAL=NO

AUTOMATIC_ALLOWLIST_GROWTH=NO
CURRENT_CI_OFFENDERS_AUTOMATICALLY_ACCEPTED=NO

NEW_LANGUAGE_DECISION_REQUIRED=NO
NEW_PLATFORM_DECISION_REQUIRED=YES
PLATFORM_DECISION=PLAT047/#786

GENUINE_TEST_PERF_REMEDIATION_INVESTIGATION_REQUIRED=YES
PERFORMANCE_FOLLOWUP=PERF031/#787

IMPLEMENTATION_SLICE_REQUIRED_AFTER_RATIFICATION=YES
PLANNED_IMPLEMENTATION_SLICE=TEST008-B
PLANNED_IMPLEMENTATION_REPOSITORY=guillermomolina/protos

NEXT_EXECUTABLE_SLICE=PLAT047-A
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_COMMAND_EXECUTION=NONE
```

TEST008/#761 remains open and blocked on PLAT047. TEST008-B is not authorized until the platform decision is explicitly approved and ratified.