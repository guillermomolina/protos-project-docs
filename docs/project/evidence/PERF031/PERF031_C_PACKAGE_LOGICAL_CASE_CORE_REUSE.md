# PERF031-C — Package logical Case test-only Core reuse checkpoint

Date: 2026-10-08

## Exact publication

```text
ISSUE=guillermomolina/protos#787
SLICE=PERF031-C
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
PROTOS_REVISION=a463429f34d677ad896279861fa09d9dea251c1a
VERSION=0.3.288-SNAPSHOT
STATUS=PUBLISHED
SCOPE=ONE_TEST_CLASS_PLUS_POM_AND_CHANGELOG
OBSERVABLE_SEMANTICS_CHANGED=NO
TEST_INFRASTRUCTURE_CHANGED=NO
SPECIFICATION_CHANGED=NO
```

GitHub source: [exact PERF031-C commit](https://github.com/guillermomolina/protos/commit/a463429f34d677ad896279861fa09d9dea251c1a). The commit was independently read from remote `main`, after a report originally described its three paths as uncommitted. Do not treat the earlier uncommitted report as the current publication state.

Published paths:

- `src/test/java/com/guillermomolina/protos/execution/ProtosPackageTestLogicalCaseExecutionFacilityTest.java`
- `pom.xml`
- `CHANGELOG.md`

## Result and safety

The exact source diff replaces eleven caller-side `ProtosCoreBootstrap().bootstrap(CORE, resolver)` calls and per-method `packageResolver()` construction with a private class-local lazy `SharedCore` holder containing the Package-flavored resolver and caller Prelude. The requested structural result is `CALLER_CORE_BOOTSTRAPS=11->1`.

All eleven `@Test` methods and their assertions are retained. Each test still creates an independent module activation, runtime host, submission queue and logical Case facility. The attempt bridge's own lazy Prelude remains per bridge; physical project-tree Case authority and per-Case Process creation remain separate. The diff does not touch production source, the specification, shared fixture infrastructure or public Tool execution.

The user-provided execution report says the focal class **passed**. It did not provide a full Surefire summary with independently attributable test counts or timings. No full `make test` run is claimed; the report judged it unnecessary for this one-class change while PERF031 stays open.

```text
FOCAL_VALIDATION=PASS_REPORTED
VALIDATION_PROVENANCE=USER_SUPPLIED_AGENT_REPORT
TEST_METHODS_RETAINED=11
FULL_SUITE=NOT_REPORTED
PER_CLASS_TIMING_BEFORE_AFTER=UNAVAILABLE
SPEEDUP_MEASURED=NO
STRUCTURAL_CORE_REDUCTION=CONFIRMED_BY_PUBLISHED_DIFF
```

## Remaining PERF031 scope: stop one-file micro-slices

The completed PERF031-A/B/C work establishes a reduction of repeatedly bootstrapped Core, but it does **not** establish the intrinsic cost or proportionality of all materially slow Java integration classes.

The next effort must measure the **whole remaining cohort in one coherent diagnostic**, not publish another tiny optimization based only on class-parallel CI wall times. Use reduced-contention, same-host, per-class Surefire report evidence for the complete current set of relevant seven approved slow identities plus the current still-existing historical CI #2111 offenders; distinguish class runtime from Maven build time. Group only truly related, safe remediation opportunities into one implementation batch afterwards. Retain real filesystem, Package, Process, Actor, concurrency and authority coverage.

The current `Makefile` provides `test-java-confirm` with `JAVA_CONFIRM_TESTS` and `JAVA_CONFIRM_REPORTS`. The human executes commands; the agent does not run tests or state-changing Git operations in `guillermomolina/protos`.

No source-identity guess is valid for removed historical TOML classes. No duration or speedup claim is justified absent comparable measurements.

```text
PERF031_STATUS=OPEN_READY
CURRENT_SLICE_COMPLETE=PERF031-C
NEXT_WORK=COHORT_MEASUREMENT_AND_PROPORTIONALITY_CLASSIFICATION
MEASUREMENT_EXECUTION=HUMAN
NEXT_IMPLEMENTATION_SCOPE=NOT_YET_JUSTIFIED
NEW_ISSUE_REQUIRED=NO
PERF032_DEPENDENCY=NONE
```
