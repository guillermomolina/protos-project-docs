# PERF009-A closure evidence

Status: **CLOSED — FINAL ATTRIBUTION AND BUDGET APPROVED**

This record closes `PERF009-A — Current full-suite baseline and critical-path attribution` (`guillermomolina/protos#531`) and supersedes only the open/in-progress closure state recorded in the earlier PERF009 diagnostic checkpoint. The retained diagnostic evidence in `docs/project/evidence/PERF009/README.md` remains historical evidence and is not rewritten by this closure record.

## Exact published Protos revision

```text
PROTOS_REVISION=42503330b671b6b13e442a4524cd63d26fa1051b
IMPLEMENTATION_VERSION=0.3.4-SNAPSHOT
```

The published Protos commit is:

```text
PERF009-A: correct TOML stress execution path
```

It changes exactly:

- `src/test/java/com/guillermomolina/protos/execution/ProtosTomlParserStressTest.java`;
- `pom.xml`;
- `CHANGELOG.md`.

The stress harness now enters guest execution through `ProtosTestExecutionSupport.evaluate(...)` instead of direct `ProtosSourceCompiler().compile(...).call(...)`. This removes the legacy direct-harness execution distortion from the retained TOML stress measurements without changing Protos-visible semantics, the specification, Standard Library behavior, or a production runtime path.

## Canonical publication validation

The published change was validated through the repository-canonical Java and Protos test lanes.

```text
JAVA_TESTS_RUN=1807
JAVA_FAILURES=0
JAVA_ERRORS=0
JAVA_SKIPPED=0
JAVA_TOTAL_TIME_SECONDS=39.622

PROTOS_TESTS_PASSED=1308
PROTOS_TESTS_FAILED=0
PROTOS_TEST_TOOL_TOTAL_TIME_SECONDS=75
```

The Java lane was executed through `make test-java`; the Protos lane was executed through the repository Test Tool path used by `make test` / `make test-protos`.

## Reproducible current baseline

Warm PERF009-A measurements on the frozen measurement revision/toolchain established:

```text
JAVA_LANE_MEDIAN_SECONDS=55.262
PROTOS_LANE_MEDIAN_SECONDS=100.690
SEQUENTIAL_SUM_SECONDS=155.952
```

The Protos lane is therefore the larger current lane. The historical Package/TOML observation is not an adequate description of the current complete repository-test critical path.

## Package/TOML causal result

The original `ProtosTomlParserStressTest` path used a direct compiler/call harness that did not enter the ordinary test execution context. Correcting that path removed the legacy-fallback-heavy distortion while retaining all nine semantic cases.

Subsequent measurements established that the retained large flat-document and array-of-tables workloads scale approximately linearly over the measured reductions rather than showing the previously suspected catastrophic quadratic accumulation/rebuilding behavior. Local TOML source/byte transport experiments did not reveal a Package/TOML-specific dominant defect; broader general Protos/Truffle execution cost remained material.

Final answer:

```text
IS_PACKAGE_TOML_STILL_A_DOMINANT_CURRENT_BOTTLENECK=NO
```

TOML stress remains an expensive workload, but its historical apparent dominance was substantially harness-induced. General direct Java guest-entry classification is owned by `AUD012/#541`; broad Java-to-Protos semantic migration remains separate under `TEST002`; general Truffle/JIT optimization findings are not silently folded into PERF009-A.

## Approved continuation budgets

The project owner explicitly approved the following PERF009 continuation budgets:

```text
FULL_VALIDATION_WALL_BUDGET<125s
HARD_PER_TEST_WALL_BUDGET<2.0s
```

The full-validation target requires a material reduction from the `155.952 s` measured sequential baseline while leaving enough headroom for correct reintegration of quarantined coverage rather than optimizing against an incomplete topology.

The per-test budget applies to ordinary correctness tests reintegrated from the temporary PERF009 quarantine. Stress-scale evidence that cannot satisfy that ordinary-test budget must be retained on an explicit stress/benchmark surface rather than forcing every ordinary validation run to pay for it. Coverage and semantic evidence must not be weakened.

## Continuation decomposition

`PERF009-B/#546` owns quarantined Java test normalization and reintegration under the approved `<2.0 s` ordinary-test budget and the existing class-level quarantine policy.

The next work must proceed from measured current bottlenecks. No general runtime/JIT redesign, general Java-to-Protos migration, or unrelated direct-entry cleanup is approved by this closure.

## Closure transaction

```text
PERF009_A_CLOSED=YES
IS_PACKAGE_TOML_STILL_A_DOMINANT_CURRENT_BOTTLENECK=NO
FULL_VALIDATION_WALL_BUDGET=<125s
HARD_PER_TEST_WALL_BUDGET=<2.0s
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS
PROTOS_REVISION=42503330b671b6b13e442a4524cd63d26fa1051b
```

`PROJECT_RECORD_REVISION` is the exact `protos-project-docs` commit that publishes this file and is recorded in the final GitHub Issue closure comment after publication and re-read.
