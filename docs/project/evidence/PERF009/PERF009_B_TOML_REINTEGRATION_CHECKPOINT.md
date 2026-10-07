# PERF009-B TOML parser reintegration checkpoint

Status: **IN PROGRESS — SECOND QUARANTINED CLASS REINTEGRATED**

This immutable checkpoint records the completed TOML parser reintegration under
`PERF009-B — Quarantined Java test normalization and reintegration`
(`guillermomolina/protos#546`). It does not close PERF009-B: six Java classes
remain in `JAVA_SLOW_TEST_EXCLUDES`.

## Exact product publications

The bounded TOML reintegration was published at:

```text
TOML_REINTEGRATION_REVISION=44810767e4c8fdcd5602f05da932107b9431f3c3
IMPLEMENTATION_VERSION=0.3.13-SNAPSHOT
PRODUCT_COMMIT=PERF009-B: reintegrate TOML parser tests
```

The immediately following corrective publication is:

```text
FINAL_CORRECTED_REVISION=079d6056f41c37ed7381b165114bd7bfd77e6b48
IMPLEMENTATION_VERSION=0.3.14-SNAPSHOT
PRODUCT_COMMIT=Fix stale checkout artifact selection
```

At this checkpoint,
`079d6056f41c37ed7381b165114bd7bfd77e6b48` is the published `main` revision
and directly descends from the TOML reintegration revision.

No Protos-visible language, specification, Standard Library, or production
runtime semantics change is part of either publication.

## Reintegration shape

The quarantined monolithic class `ProtosTomlParserStressTest` was removed and
replaced by eight independently schedulable ordinary regression classes:

```text
ProtosTomlParserArrayOfTablesRegressionTest
ProtosTomlParserFlatDocumentRegressionTest
ProtosTomlParserFloatTemporalRegressionTest
ProtosTomlParserIntegerRegressionTest
ProtosTomlParserLexicalKeyRegressionTest
ProtosTomlParserLexicalStringTriviaRegressionTest
ProtosTomlParserNestedValueRegressionTest
ProtosTomlParserPathRegressionTest
```

The split preserves the distinct regression responsibilities previously grouped
inside the stress class while allowing each ordinary responsibility to satisfy
the PERF009-B scheduling budget independently.

The final bounded ordinary workloads cover nested arrays, nested inline tables,
flat assignments, dotted and header paths, lexical keys, string/trivia scanning,
long integers, float and temporal tokens, and repeated arrays of tables.

Historical recursion/scaling guards were retained at bounded scales where they
remain necessary to distinguish the corrected implementation shape from the
prohibited historical behavior. Incidental stress scale was not retained merely
for load generation.

## Repeated timing evidence

The PERF009-B ordinary-test policy remains:

```text
HARD_PER_TEST_WALL_BUDGET=<2.0s
CLASS_QUARANTINE_THRESHOLD=<5s
```

Repeated fresh-JVM measurement of the eight final ordinary classes initially
found one remaining violation in the string/trivia responsibility:

```text
ProtosTomlParserLexicalStringTriviaRegressionTest
max_test=2.109s
result=FAIL
```

That workload was reduced to the bounded semantic/trivia guard using 128 spaces
and rerun repeatedly. The final normalized eight-class set satisfies:

```text
EVERY_ORDINARY_TESTCASE_LT_2S=YES
EVERY_ORDINARY_CLASS_LT_5S=YES
REPEATED_TIMING_POLICY=PASS
```

The normalization therefore did not accept a lucky single run.

## Repository validation

After removing `ProtosTomlParserStressTest` from the quarantine and replacing it
with the eight ordinary regression classes, repository validation completed
successfully:

```text
JAVA_LANE=PASS
PROTOS_TESTS=1309
PROTOS_TEST_FAILURES=0
FULL_TESTS=PASS
TOML_ORDINARY_NORMALIZATION=CLOSED
TOML_SLOW_QUARANTINE=REMOVED
```

## Immediate publication correction

The initial reintegration publication exposed two checkout/developer-path
problems that were corrected immediately in
`079d6056f41c37ed7381b165114bd7bfd77e6b48`.

First, the quarantine edit had accidentally transformed the removed TOML entry
into `**/.java` inside `JAVA_SLOW_TEST_EXCLUDES`. The corrective publication
removes that malformed exclusion.

Second, checkout execution in `bin/protos` selected a built artifact using
lexicographic filename ordering:

```text
find ... -name 'protos-*.jar' | sort | tail -n 1
```

That is not a valid implementation-version selection mechanism. A stale
`protos-0.3.9-SNAPSHOT.jar` could sort after `protos-0.3.13-SNAPSHOT.jar` and
execute older Java implementation code against current Core sources.

The corrected launcher reads Maven build metadata from
`target/maven-archiver/pom.properties` and selects exactly
`target/<artifactId>-<version>.jar`.

Regression validation deliberately placed a stale
`target/protos-0.3.9-SNAPSHOT.jar` beside the current built artifact and reran
the repository tests:

```text
STALE_JAR_SELECTION=PASS
MAKEFILE_QUARANTINE=PASS
IMPLEMENTATION_VERSION=0.3.14-SNAPSHOT
FULL_TESTS=PASS
PROTOS_TESTS=1309
PROTOS_TEST_FAILURES=0
```

## Quarantine state after final correction

The published `JAVA_SLOW_TEST_EXCLUDES` now contains six PERF009-B classes:

```text
ProtosTomlEncoderModuleTest
ProtosPackageToolProtosTest
ProtosExternalPackagePlanningPreflightTest
ProtosWorkspaceRunCliTest
ProtosJsonParserModuleTest
ProtosPackageExecutionPlanAdapterTest
```

`ProtosTomlParserStressTest` is no longer present.

## Scope boundary

No workload from this TOML normalization slice was migrated to
`guillermomolina/protos-benchmarks`. That companion-repository migration was
explicitly kept outside this slice.

## PERF009-B state

```text
QUARANTINED_JAVA_CLASSES_INITIAL=8
QUARANTINED_JAVA_CLASSES_CURRENT=6
REINTEGRATED_CLASSES=2
MANIFEST_REINTEGRATION=PASS
TOML_REINTEGRATION=PASS
TOML_ORDINARY_CLASS_COUNT=8
ORDINARY_TEST_HARD_BUDGET=<2.0s_PER_TEST
CLASS_QUARANTINE_THRESHOLD=<5s
REPEATED_TIMING_EVIDENCE=PASS
FULL_TESTS=GREEN
PERF009_B_CLOSED=NO
```

The remaining PERF009-B work continues class by class under the same timing and
coverage-preservation policy. This checkpoint records the TOML parser slice only
and does not pre-approve remediation choices for the six remaining quarantined
classes.
