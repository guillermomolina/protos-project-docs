# TEST001 — Official Test Tool execution cutover

Record state: FINAL DURABLE RECONCILIATION

Nature: non-normative repository validation ownership and closure record

Live coordination: GitHub `TEST001 / #449`

This record does not mirror the live Issue lifecycle state. GitHub remains the
coordination authority; this file records the durable repository result needed
to evaluate final closure.

## Final scope

TEST001 owns the repository runner cutover for tests already written in Protos.
It does **not** own a general campaign to rewrite Java/JUnit tests as Protos
tests.

The final ownership model is:

```text
repository validation
  |
  +-- Java / Truffle / runtime / host / bootstrap / integration evidence
  |     -> Java/JUnit
  |
  +-- official repository-owned tests written in Protos
        -> TOOL002 / protos test
```

The durable rule is about execution ownership:

- `protos test` / TOOL002 is the official repository runner for selected Protos
  corpora;
- Java/JUnit remains the appropriate owner for Java, Truffle, compiler/backend,
  runtime, scheduler, native/host, integration, architecture, and independent
  bootstrap evidence;
- using a `.protos` fixture from Java does not by itself make a Java test a
  duplicate corpus owner;
- an obsolete Java/JUnit wrapper whose primary purpose is to rerun an official
  Protos corpus is retired or reduced to an independently justified host or
  bootstrap assertion.

## Final dependency result

The TEST001 path completed through the following durable work:

- `TEST001-A / #459` — existing Protos-corpus runner inventory: CLOSED;
- `TEST001-B / #460` — direct TOOL002 repository/CI execution: CLOSED;
- `TOOL005 / #468` — repository-wide Protos test corpus execution: CLOSED;
- `TEST001-H / #466` — portable-distribution TOOL002 execution: CLOSED;
- `GITHUB017 / #492` — automatic test CI reactivation: CLOSED;
- `TEST001-I / #467` — superseded ownership-infrastructure cleanup and final
  reconciliation: CLOSED as superseded/not-planned after the useful cleanup was
  published;
- `TEST002 / #538` — legacy Java-to-Protos semantic migration: separate work,
  intentionally not a TEST001 closure dependency.

The abandoned TEST001-C/D/E/F/G semantic-migration decomposition is historical
only. It is not current TEST001 scope or dependency structure.

## Repository-wide runner reconciliation

`docs/project/work/TOOL005/TOOL005_B5_CLOSURE.md` is the durable repository-wide
ownership audit. Its resulting boundary is:

```text
mvn test
  -> Java / Truffle / runtime / host / bootstrap / architecture / integration

protos test
  -> repository-owned tests written in Protos and selected for official validation
```

TOOL005-B5 records that obsolete Java/JUnit full-corpus owners were removed,
mixed wrappers were reduced to legitimate Java/host assertions, retained Java
tests were classified as host/runtime/integration/mechanism evidence, and no
remaining Java/JUnit test was found to own an already-selected official Protos
corpus as a full guest-semantic runner.

Its final integrated evidence was:

```text
JAVA_LANE=1876/1876_PASS
PROTOS_LANE=1301/1301_PASS
PACKAGE=PASS
DIFF_CHECK=PASS
JAVA_FULL_CORPUS_DUPLICATION=REMOVED
OFFICIAL_PROTOS_CORPUS_OWNER=protos_test
JAVA_HOST_RUNTIME_EVIDENCE=PRESERVED
```

## Independent TOOL002 bootstrap floor

TEST001-B retained an independent host/bootstrap proof rather than making the
Test Tool its own only bootstrap evidence. Java/JUnit therefore remains present
where it proves CLI-to-tool resolution, entry-module loading, result/exit
propagation, runtime integration, and other host-side mechanics independently of
the complete Protos corpus.

```text
INDEPENDENT_TOOL002_BOOTSTRAP_FLOOR=YES
HOST_RUNTIME_TESTS=JUNIT
```

## Portable-distribution proof

`docs/project/work/TEST001/TEST001_H_PORTABLE_DISTRIBUTION.md` proves that an
extracted portable distribution invokes its bundled Test Tool through the public
`protos test` entry point outside the source checkout.

The published TEST001-H closure signals include:

```text
PORTABLE_ARTIFACT_EXECUTES_BUNDLED_TEST_TOOL=YES
PORTABLE_TEST_INVOCATION_OUTSIDE_CHECKOUT=YES
OFFICIAL_SUITE_SELECTION_REUSED=YES
CHECKOUT_JUNIT_CORPUS_WRAPPER_REQUIRED=NO
TOOL002_SUCCESS_OUTCOME_PROPAGATED=YES
PORTABLE_SELECTED_CASES=NONZERO
PORTABLE_FAILED_CASES=0
DISTRIBUTION_TEST_TOOL_VALIDATED=YES
```

## Superseded ownership infrastructure cleanup

The earlier semantic-migration expansion introduced a global semantic ownership
registry and publication guard. The project owner later corrected TEST001 back
to the narrower runner-cutover scope.

TEST001-I published the cleanup in `guillermomolina/protos` at exact revision:

```text
PROTOS_REVISION=e99d0baba547ac41b3894f32ddca450172ee1f8b
```

That revision removed the superseded machinery, including:

- `protos/tests/test_ownership.json`;
- `scripts/test_ownership_guard.py`;
- its dedicated guard tests;
- publication-validation coupling to the ownership guard; and
- the obsolete AGENTS semantic-ownership-registry policy.

The current repository test-placement rule expresses the durable policy directly:
prefer ordinary Protos/Test Tool coverage when it faithfully proves observable
behavior, while retaining Java/JUnit for materially different Java/Truffle/
runtime/host/bootstrap evidence. A global semantic-ownership registry is not
required.

```text
SUPERSEDED_C_D_E_F_G_PATH=HISTORICAL
OWNERSHIP_REGISTRY_GUARD=RETIRED
```

## TEST002 handoff

The original desire to audit old Java/JUnit tests and migrate those whose primary
contract is observable Protos behavior is intentionally preserved as separate
work:

```text
FUTURE_JAVA_TO_PROTOS_MIGRATION=TEST002/#538
TEST001_REOPENED_FOR_SEMANTIC_MIGRATION=NO
OWNERSHIP_REGISTRY_RECREATION_REQUIRED=NO
```

TEST002 must perform a fresh repository-wide inventory when resumed. It must not
assume the superseded TEST001-C/D/E/F/G decomposition remains valid.

## Final CI evidence for TEST001-I product revision

GitHub Actions run `#1823` (`35081293631`) validates the exact TEST001-I product
revision `e99d0baba547ac41b3894f32ddca450172ee1f8b`.

Attempt 1 failed only in
`ProtosI026FDapTransportTest.realGraalVmDapInstrumentCompletesTcpHandshake` with
a transient `java.net.SocketException: Broken pipe` while the Java parallel lane
reported 1797 tests, 0 failures, and 1 error.

The same test then passed focally, and the same 1797-test Java parallel lane
passed locally. No product correction was made. GitHub Actions attempt 2 reran
the **same exact SHA** and completed successfully:

```text
CI_RUN=35081293631
CI_RUN_NUMBER=1823
CI_ATTEMPT=2
CI_HEAD_SHA=e99d0baba547ac41b3894f32ddca450172ee1f8b
CI_CONCLUSION=SUCCESS
```

The first failure is therefore retained as transient diagnostic history, not as
evidence of an unresolved TEST001 product defect.

## TEST001 closure criteria

The TEST001/#449 closure contract is satisfied as follows:

1. **Known host wrappers inventoried** — TEST001-A and the later TOOL005-B5
   repository-wide reconciliation classified the relevant corpus runners.
2. **Direct TOOL002 repository/CI execution** — TEST001-B established direct
   `protos test` execution; restored CI consumes the separated repository test
   lanes.
3. **Obsolete full-corpus Java/JUnit wrappers retired or justified** — TOOL005-B5
   removed duplicate full-corpus owners and reduced mixed wrappers while
   retaining genuine host/runtime/bootstrap evidence.
4. **Java/JUnit remains available for host/runtime tests** — preserved by the
   final ownership boundary.
5. **Independent bounded TOOL002 bootstrap smoke remains** — preserved by
   TEST001-B and host-side Test Tool integration coverage.
6. **Portable distribution exercises its bundled Test Tool** — proved by
   TEST001-H.
7. **Validation/documentation names `protos test` as the official Protos runner**
   — established by TEST001-B, TOOL005-B5, TEST001-H, current repository test
   placement policy, and this final reconciliation record.

## Final closure signals

```text
TEST001_RESULT=RUNNER_CUTOVER_COMPLETE
OFFICIAL_PROTOS_TEST_RUNNER=TOOL002
PROTOS_CORPUS_EXECUTION=protos_test
LEGACY_JUNIT_CORPUS_WRAPPERS=RETIRED_OR_JUSTIFIED
HOST_RUNTIME_TESTS=JUNIT
INDEPENDENT_TOOL002_BOOTSTRAP_FLOOR=YES
DISTRIBUTION_TEST_TOOL_VALIDATED=YES
SUPERSEDED_C_D_E_F_G_PATH=HISTORICAL
OWNERSHIP_REGISTRY_GUARD=RETIRED
FUTURE_JAVA_TO_PROTOS_MIGRATION=TEST002/#538
PROTOS_REVISION=e99d0baba547ac41b3894f32ddca450172ee1f8b
CI_RUN=35081293631
CI_ATTEMPT=2
CI_CONCLUSION=SUCCESS
SPECIFICATION_CHANGED=NO
PROTOS_IMPLEMENTATION_CHANGED=NO
IMPLEMENTATION_VERSION_CHANGED=NO
```

## Historical record provenance

The detailed pre-scope-correction TEST001 migration record remains recoverable
exactly from Git history and is deliberately not duplicated here as current
policy.

Immediately before this final reconciliation:

```text
PROJECT_DOCS_BASE=a6e6db8345a2805d7db71175c3d02010373d66bd
PRE_RECONCILIATION_BLOB=1fdd93e25bb118bfb8265741a64b658da18ce0b6
```

That historical revision contains the original C/D/E/F/G decomposition,
ownership-registry design, D1-D5 execution evidence, migration ordering, and the
intermediate status table. Those statements are historical snapshots only and
must not be interpreted as current scheduling, dependency, or repository policy.

This replacement follows the project-documentation policy: maintain the canonical
current record at its existing role-first path, preserve historical evidence by
exact Git identity, and do not create a duplicate Markdown closure record merely
for symmetry.
