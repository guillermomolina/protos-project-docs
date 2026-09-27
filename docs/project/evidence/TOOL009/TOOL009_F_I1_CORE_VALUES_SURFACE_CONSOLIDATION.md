# TOOL009-F-I1 — Core values and surface consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I1`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `343e74eca5870d6d119612be79d968a9a8cce4c9`
- Commit:
  `TOOL009-F-I1: consolidate F-I1 conformance corpus to 58 suite-native sources`

This record is implementation evidence for the first publication slice of
TOOL009-F. The repository-wide target map and the investigation that authorized
the slice remain in
`TOOL009_F_TEST_SOURCE_CONSOLIDATION_FEASIBILITY.md`.

## Scope implemented

F-I1 consolidated these suite-native conformance families:

```text
boolean
float
integer
number
numeric-conversion
numeric-equality
equality
string
object
object-structural
path
call
core-surface
matching
network
surface-sugar
```

The physical-source reconciliation published by the product commit is:

```text
F_I1_SOURCE_FILES_BEFORE=282
F_I1_SOURCE_FILES_AFTER=58

F_I1_LOGICAL_CASES_BEFORE=282
F_I1_LOGICAL_CASES_AFTER=282

FILES_REMOVED=281
FILES_CREATED=57
FILES_REUSED=1

F_I1_CASE_RECONCILIATION=PASS
UNASSIGNED_TESTS=0
DUPLICATED_TESTS=0
LEFTOVER_OLD_FILES=0
MANIFEST_SOURCE_PATHS=PASS
OBSOLETE_LIVE_PATH_REFERENCES=0
```

The exact Test-name multiset and Test-body inventory were preserved while the
physical source layout changed. The source-granularity rule is now recorded in
`protos/AGENTS.md`.

The only existing single-Case source intentionally reused by this slice is:

```text
protos/tests/conformance/network/network-prototype.protos
```

## Path-coupling reconciliation

F-I1 replaced the old `integer/add-small.protos` physical source with the
consolidated `integer/arithmetic-and-unary.protos` source.

Live path-coupled fixtures/assertions were reconciled in the same product
commit, including:

```text
protos/tests/tooling/tool002-d2-manifest-plan.protos
protos/tests/tooling/tool002-e1b-package-toml-filesystem.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolManifestPlanTest.java
```

No compatibility source or migration alias was retained.

## Out-of-scope fixture exposed by F-I1

The directory `protos/tests/conformance/network/` also contains:

```text
ip-data-parallel-transfer.protos
```

That file is not one of the nine F-I1 suite-native network sources and is not a
Logical Case:

- it declares no `Test(...)`;
- it is not represented as a suite-native manifest Case;
- it is loaded directly by
  `ProtosParallelExecutionTest.ipDataTransferConformanceSourceRoundTripsThroughP()`;
- the Java test owns the hosted execution domain, Future completion, task drain,
  and host/runtime integration assertions.

The fixture therefore remained unchanged in F-I1. Moving it during F-I1 would
have mixed suite-native source-granularity work with a distinct Java/JUnit-owned
fixture-placement cleanup.

Current test-placement policy already provides the destination class for such
evidence: JUnit-owned component fixtures under `protos/tests/tooling/` remain
externally owned and do not become production TestPlan ownership merely because
they contain Protos source.

The cleanup is routed independently as TEST004 rather than changing the
TOOL009-F target count or treating this fixture as a missing Logical Case.

## Documentation drift observed but not folded into F-I1

`docs/guide/tools/test-tool.md` still contains a historical/minimal example
using the spelling `integer/add-small.protos` together with manifest content
that does not describe the current suite-native manifest format. F-I1 did not
rewrite that example because it is documentation-content maintenance rather
than a live path dependency of the consolidation.

This record does not allocate or resolve that documentation follow-up.

## Result

```text
TOOL009_F_I1=COMPLETE
PROTOS_REVISION=343e74eca5870d6d119612be79d968a9a8cce4c9

F_I1_SOURCE_FILES_BEFORE=282
F_I1_SOURCE_FILES_AFTER=58
F_I1_LOGICAL_CASES_BEFORE=282
F_I1_LOGICAL_CASES_AFTER=282

LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE

ORPHAN_SUITE_NATIVE_CASE=NO
JUNIT_OWNED_PROTOS_FIXTURE_IDENTIFIED=YES
FOLLOWUP=TEST004
```

No remote-CI result is claimed by this record.
