# PERF009-B TOML encoder reintegration checkpoint

Status: **IN PROGRESS — THIRD QUARANTINED CLASS REINTEGRATED**

This immutable checkpoint records the completed TOML encoder reintegration under
`PERF009-B — Quarantined Java test normalization and reintegration`
(`guillermomolina/protos#546`). It does not close PERF009-B: five Java classes
remain in `JAVA_SLOW_TEST_EXCLUDES`.

## Exact product publication

```text
PROTOS_REVISION=7a22c26dbeff978c38f637f033e8edf1ecc6fdee
IMPLEMENTATION_VERSION=0.3.19-SNAPSHOT
PRODUCT_COMMIT=PERF009-B: reintegrate TOML encoder tests
```

At this checkpoint,
`7a22c26dbeff978c38f637f033e8edf1ecc6fdee` is the published `main` revision.

No Protos-visible language, specification, Standard Library, or production
runtime behavior changes are part of this publication.

## Reintegration shape

The formerly quarantined class `ProtosTomlEncoderModuleTest` now shares one Core
bootstrap across the class and executes guest code through the canonical
`ProtosTestExecutionSupport` boundary instead of rebuilding Core for every
`evaluate()` call.

The previous deep-container tests combined three distinct responsibilities in
one expensive path:

```text
deep semantic-container construction
TOML.encode at stress depth
TOML.parse + semantic walk at the same stress depth
```

PERF009-B separates those responsibilities without deleting semantic coverage.
Ordinary nested encode/parse round-trip coverage remains in
`ProtosTomlEncoderModuleTest` at a bounded depth of 32, while the historical
input-depth recursion regression is owned by the new ordinary class:

```text
ProtosTomlEncoderDeepContainerRegressionTest
```

The deep regression class verifies the encoder itself at evidence-derived depths
that reproduce the prohibited historical host-recursive implementation.

## Historical recursion evidence

The historical pre-LIB010-C2 encoder revision used host-recursive semantic
container descent. Temporary isolated probes against that implementation were
used to locate the actual failure boundary rather than retaining the old
2,048-Array / 1,024-Table stress sizes by convention.

### Arrays

```text
HISTORICAL_LAST_PASS=74
HISTORICAL_FIRST_FAIL=75
SELECTED_GUARD_DEPTH=139
HISTORICAL_FAILURE=StackOverflowError
```

The current iterative encoder successfully encodes the selected depth of 139.
The deep regression verifies deterministic full traversal through the encoded
result rather than requiring the parser to consume the same stress-scale tree.

### Inline tables

```text
HISTORICAL_LAST_PASS=77
HISTORICAL_FIRST_FAIL=78
SELECTED_GUARD_DEPTH=142
HISTORICAL_FAILURE=StackOverflowError
SELECTED_GUARD_REPEATED_HISTORICAL_FAILURE=3/3
```

The selected inline-table guard is therefore well above the first demonstrated
historical failure boundary and is not a knife-edge threshold.

The current iterative encoder successfully encodes the selected depth of 142.

## Preserved semantic coverage

The normalization preserves these distinct properties:

```text
all ten TOML semantic kinds
D109 canonical shortest-round-trip Float spelling
hard binary64 round-trip values
basic-string key/value escaping
invalid-root rejection
unknown-node rejection
cycle rejection
shared acyclic container reuse
D104 second=60 temporal spelling
zero-offset Z spelling
nested Array encode/parse semantic round-trip
nested inline-Table encode/parse semantic round-trip
Array traversal beyond the historical recursive failure boundary
inline-Table traversal beyond the historical recursive failure boundary
```

The original 2,048/1,024 diagnostic depths are therefore no longer treated as
ordinary-test contracts. PERF009-B retains the actual correctness property: a
bounded workload must remain large enough to fail the prohibited historical
implementation shape.

## Repeated timing evidence

The approved PERF009-B limits remain:

```text
HARD_PER_TEST_WALL_BUDGET=<2.0s
CLASS_QUARANTINE_THRESHOLD=<5s
```

Five fresh-JVM runs of the final two-class shape produced:

```text
RUN 1
ProtosTomlEncoderModuleTest:
  suite=3.498s
  max_test=0.656s
ProtosTomlEncoderDeepContainerRegressionTest:
  suite=1.758s
  max_test=1.019s

RUN 2
ProtosTomlEncoderModuleTest:
  suite=3.417s
  max_test=0.622s
ProtosTomlEncoderDeepContainerRegressionTest:
  suite=1.748s
  max_test=0.999s

RUN 3
ProtosTomlEncoderModuleTest:
  suite=3.425s
  max_test=0.622s
ProtosTomlEncoderDeepContainerRegressionTest:
  suite=1.768s
  max_test=1.013s

RUN 4
ProtosTomlEncoderModuleTest:
  suite=3.424s
  max_test=0.619s
ProtosTomlEncoderDeepContainerRegressionTest:
  suite=1.766s
  max_test=1.006s

RUN 5
ProtosTomlEncoderModuleTest:
  suite=3.425s
  max_test=0.623s
ProtosTomlEncoderDeepContainerRegressionTest:
  suite=1.792s
  max_test=1.036s
```

Therefore:

```text
ENCODER_MODULE_CLASS_MAX=3.498s
ENCODER_MODULE_TEST_MAX=0.656s
DEEP_REGRESSION_CLASS_MAX=1.792s
DEEP_REGRESSION_TEST_MAX=1.036s
EVERY_ORDINARY_TESTCASE_LT_2S=YES
EVERY_ORDINARY_CLASS_LT_5S=YES
REPEATED_TIMING_EVIDENCE=PASS
```

The final timing result was:

```text
TIMING_GATE=PASS
ENCODER_REINTEGRATION_ELIGIBLE=YES
```

## Repository validation

The final publication removed `ProtosTomlEncoderModuleTest` from
`JAVA_SLOW_TEST_EXCLUDES` and completed the repository validation before commit:

```text
ENCODER_REINTEGRATION=PASS
REPEATED_TIMING_EVIDENCE=PASS
FULL_TESTS=PASS
```

## Quarantine state after publication

The published `JAVA_SLOW_TEST_EXCLUDES` now contains five PERF009-B classes:

```text
ProtosPackageToolProtosTest
ProtosExternalPackagePlanningPreflightTest
ProtosWorkspaceRunCliTest
ProtosJsonParserModuleTest
ProtosPackageExecutionPlanAdapterTest
```

`ProtosTomlEncoderModuleTest` is no longer quarantined.
`ProtosTomlEncoderDeepContainerRegressionTest` is an ordinary class and was
never added to the quarantine.

## Scope boundary

No workload from this encoder normalization slice was migrated to
`guillermomolina/protos-benchmarks`.

The historical stress sizes were replaced by evidence-derived ordinary
regression guards in `guillermomolina/protos`; this is a test-structure and
harness normalization only.

## PERF009-B state

```text
QUARANTINED_JAVA_CLASSES_INITIAL=8
QUARANTINED_JAVA_CLASSES_CURRENT=5
REINTEGRATED_CLASSES=3
MANIFEST_REINTEGRATION=PASS
TOML_PARSER_REINTEGRATION=PASS
TOML_ENCODER_REINTEGRATION=PASS
ORDINARY_TEST_HARD_BUDGET=<2.0s_PER_TEST
CLASS_QUARANTINE_THRESHOLD=<5s
REPEATED_TIMING_EVIDENCE=PASS
SEMANTIC_COVERAGE=EQUIVALENT_OR_STRONGER
FULL_TESTS=GREEN
PERF009_B_CLOSED=NO
```

The remaining PERF009-B work continues class by class under the same timing and
coverage-preservation policy. This checkpoint records the TOML encoder slice only
and does not pre-approve remediation choices for the five remaining quarantined
classes.
