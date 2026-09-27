# TEST004 — Parallel-transfer fixture relocation

## Identity

- Formal work item: `TEST004 / guillermomolina/protos#730`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `1f966bd613442bccc49f61147cbccd54c91e7de5`
- Commit:
  `TEST004: relocate Java-owned parallel-transfer fixture out of conformance corpus`
- Trigger: TOOL009-F-I1 / #694

## Purpose

TEST004 removed a placement ambiguity exposed by TOOL009-F-I1.

The Protos source:

```text
protos/tests/conformance/network/ip-data-parallel-transfer.protos
```

was not a suite-native Logical Case. It was a Java/JUnit-owned integration
fixture loaded directly by
`ProtosParallelExecutionTest.ipDataTransferConformanceSourceRoundTripsThroughP()`.

The fixture proves a distinct host/runtime integration boundary involving
hosted execution, caller execution-domain ownership, asynchronous Future
completion, transfer of IpAddress/IpEndpoint values, and final task drain.

## Published change

The product commit performs exactly this relocation:

```text
R  protos/tests/conformance/network/ip-data-parallel-transfer.protos
   -> protos/tests/tooling/parallel-execution-ip-data-transfer.protos

M  src/test/java/com/guillermomolina/protos/execution/ProtosParallelExecutionTest.java
```

GitHub records the fixture as a rename with zero content additions/deletions.
The Java test changes only the literal path used to load the fixture.

The fixture remains Java/JUnit-owned. It was not converted to
`std:test/Test`, was not added to the conformance manifest, and does not
become a TOOL009/Test Tool Logical Case.

## Validation

Human-executed validation was reported green.

```text
GIT_DIFF_CHECK=PASS
DIFF_SCOPE=PASS
CONFORMANCE_MANIFEST=UNAFFECTED
MAVEN_FOCAL_TEST=PASS
ALL_REQUIRED_TESTS=PASS
```

The focal command reported PASS:

```text
mvn -Dtest=ProtosParallelExecutionTest test
```

The project owner additionally reported that all tests executed for the final
candidate passed.

## Invariants

```text
OLD_FIXTURE_PATH_ABSENT=PASS
NEW_FIXTURE_PATH_PRESENT=PASS
FIXTURE_CONTENT_BEHAVIOR_PRESERVED=PASS
JUNIT_PATH_UPDATED=PASS

CONFORMANCE_MANIFEST_OWNERSHIP=UNCHANGED
SUITE_NATIVE_LOGICAL_CASE_COUNT_CHANGE=0
TEST_TOOL_SEMANTICS_CHANGE=NONE

PARALLEL_SEMANTICS_CHANGE=NONE
IP_ADDRESS_SEMANTICS_CHANGE=NONE
IP_ENDPOINT_SEMANTICS_CHANGE=NONE
TRANSFER_SEMANTICS_CHANGE=NONE

SPEC_CHANGE=NONE
MAVEN_VERSION_BUMP=NONE
CHANGELOG_CHANGE=NONE
```

## License

The existing Protos-owned APL source header was preserved unchanged through
the rename. No new source file content was authored.

## Relationship to TOOL009-F

TEST004 is complete and does not alter TOOL009-F source/case target counts.

TOOL009-F-I1 is fully complete, including the separately routed fixture
placement cleanup discovered during that slice.

The parent TOOL009-F / #694 remains open because its planned implementation
sequence still includes F-I2 through F-I8.

## Result

```text
TEST004=COMPLETE
PROTOS_REVISION=1f966bd613442bccc49f61147cbccd54c91e7de5
FINAL_REQUIRED_VALIDATION=PASS

TOOL009_F_I1=COMPLETE
TOOL009_F_PARENT_CLOSURE_READY=NO
NEXT_PARENT_SLICE=TOOL009-F-I2
```
