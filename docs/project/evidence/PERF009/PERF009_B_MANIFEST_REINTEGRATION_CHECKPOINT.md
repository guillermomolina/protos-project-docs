# PERF009-B Manifest planning reintegration checkpoint

Status: **IN PROGRESS — FIRST QUARANTINED CLASS REINTEGRATED**

This immutable checkpoint records the first completed class reintegration under
`PERF009-B — Quarantined Java test normalization and reintegration`
(`guillermomolina/protos#546`). It does not close PERF009-B: seven Java classes
remain in `JAVA_SLOW_TEST_EXCLUDES`.

## Exact product publication

```text
PROTOS_REVISION=9be78cc5d29b9815f33e3b37f799622eeabb2960
IMPLEMENTATION_VERSION=0.3.7-SNAPSHOT
PRODUCT_COMMIT=PERF009-B: reintegrate Manifest planning tests
```

The published product change removes
`ProtosTestToolManifestPlanTest` from the class-granular PERF009 quarantine and
introduces `ProtosTestToolProjectTreePlanTest` as a separate JUnit scheduling
boundary for the D133 project-tree responsibility.

No Protos-visible semantics, specification, Standard Library behavior, or
production runtime behavior changes are part of this slice.

## Preserved regression and coverage properties

The retained Manifest traversal regression guard was reduced from 2048 rows to
160 rows only after confirming that the historical prohibited window-size-1
implementation still fails at 160 rows with `StackOverflowError`. The bounded
ordinary test therefore continues to detect the prohibited recursive shape
without retaining arbitrary stress scale in the ordinary lane.

The Manifest policy tests retain real repository corpus data while bounding the
fixture materialization needed for their exact assertions. Guest-visible
Manifest/TextReader behavior is exercised through a deterministic read-only test
backend implementing the same portable filesystem protocol; NIO backend behavior
remains independently covered by its dedicated filesystem suites.

The D133 project-tree tests were not deleted or weakened. They were moved behind
the distinct `ProtosTestToolProjectTreePlanTest` JUnit class boundary while their
existing test bodies and assertions remain owned by the Manifest planning test
implementation.

## Repeated timing evidence

Three fresh Surefire/JVM executions of both resulting classes produced:

```text
RUN 1
ProtosTestToolManifestPlanTest:
  suite=3.344s
  max_test=0.682s
  tests=18
ProtosTestToolProjectTreePlanTest:
  suite=1.878s
  max_test=0.857s
  tests=4

RUN 2
ProtosTestToolManifestPlanTest:
  suite=3.316s
  max_test=0.670s
  tests=18
ProtosTestToolProjectTreePlanTest:
  suite=1.868s
  max_test=0.853s
  tests=4

RUN 3
ProtosTestToolManifestPlanTest:
  suite=3.158s
  max_test=0.686s
  tests=18
ProtosTestToolProjectTreePlanTest:
  suite=1.825s
  max_test=0.841s
  tests=4
```

Therefore:

```text
MANIFEST_CLASS_MAX_REPEATED_WALL=3.344s
PROJECT_TREE_CLASS_MAX_REPEATED_WALL=1.878s
MAX_ORDINARY_TESTCASE_WALL=0.857s
CLASS_QUARANTINE_THRESHOLD=<5s
HARD_PER_TEST_WALL_BUDGET=<2.0s
REPEATED_TIMING_EVIDENCE=PASS
```

## Ordinary Java lane validation

After removing `ProtosTestToolManifestPlanTest` from
`JAVA_SLOW_TEST_EXCLUDES`, the repository class-parallel Java lane completed:

```text
TESTS_RUN=1829
FAILURES=0
ERRORS=0
SKIPPED=0
BUILD=SUCCESS
TOTAL_TIME=39.875s
```

This validates the reintegrated class in the ordinary scheduling topology rather
than only as an isolated focal run.

## Quarantine state after publication

The published `Makefile` quarantine contains seven classes:

```text
ProtosTomlParserStressTest
ProtosTomlEncoderModuleTest
ProtosPackageToolProtosTest
ProtosExternalPackagePlanningPreflightTest
ProtosWorkspaceRunCliTest
ProtosJsonParserModuleTest
ProtosPackageExecutionPlanAdapterTest
```

`ProtosTestToolManifestPlanTest` is no longer quarantined.
`ProtosTestToolProjectTreePlanTest` is an ordinary class and was never added to
the quarantine.

## PERF009-B state

```text
QUARANTINED_JAVA_CLASSES_INITIAL=8
QUARANTINED_JAVA_CLASSES_CURRENT=7
REINTEGRATED_CLASSES=1
ORDINARY_TEST_HARD_BUDGET=<2.0s_PER_TEST
MANIFEST_REINTEGRATION=PASS
FULL_JAVA_LANE=GREEN
PERF009_B_CLOSED=NO
```

The remaining PERF009-B work continues class by class under the same approved
budget and coverage-preservation rules. This checkpoint records only the
Manifest/D133 reintegration slice and does not pre-approve remediation choices
for the seven remaining quarantined classes.
