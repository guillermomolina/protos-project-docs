# TOOL009-F-I7 — Package outcome corpus consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I7`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `9ada962e63e7198b8ae7b468b4c8f9d471bb6cff`
- Commit:
  `TOOL009-F-I7: consolidate Package version + lock + resolution-input source consolidation`

The project owner reported the published F-I7 candidate as uploaded with all validation passing.

## Scope

F-I7 consolidated the three Package Tool case-outcome corpora:

```text
version:          74 -> 9 sources / 74 Logical Cases
lock:             71 -> 10 sources / 71 Logical Cases
resolution-input: 12 -> 3 suite-native sources / 12 Logical Cases
```

Overall:

```text
F_I7_SOURCE_FILES_BEFORE=157
F_I7_SOURCE_FILES_AFTER=22
F_I7_LOGICAL_CASES_BEFORE=157
F_I7_LOGICAL_CASES_AFTER=157

CREATED_SOURCES=22
REUSED_SOURCES=0
REMOVED_SOURCES=157
```

The resolution-input directory also contains non-manifest helper fixtures
(`lockfile-fresh.protos` and `lockfile-stale.protos`); they are not part of
the 12 -> 3 suite-native case-outcome count.

## Published manifests

At the product revision:

```text
protos/tests/package-tool/version/manifest.tsv          = 9 rows
protos/tests/package-tool/lock/manifest.tsv             = 10 rows
protos/tests/package-tool/resolution-input/manifest.tsv = 3 rows
```

The published suite-native target source counts match those manifest counts.

## Validation

The project owner reported:

```text
ALL_TESTS=PASS
HUMAN_REPORTED_VALIDATION=PASS
```

No more granular command output is invented by this record.

## Boundaries preserved

```text
LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE
PACKAGE_TOOL_SEMANTICS_CHANGE=NONE
EXECUTION_REQUIREMENT_CHANGE=NONE
SPEC_CHANGE=NONE
```

## Next and final slice

Only F-I8 remains:

```text
TOOL009-F-I8
Package project-tree + final closure

PROJECT_TREE_SOURCES_BEFORE=40
PROJECT_TREE_SOURCES_AFTER=30
PROJECT_TREE_LOGICAL_CASES_BEFORE=54
PROJECT_TREE_LOGICAL_CASES_AFTER=54

REMOVED_SOURCES=19
CREATED_SOURCES=9
REUSED_SOURCES=21

FULL_SUITE_REQUIRED=YES
```

F-I8 must preserve the exact selector x project-authority Case matrix and, after
cheaper/focal validation, run the integrated final `make test` gate before
TOOL009-F can be closed.

## Result

```text
TOOL009_F_I7=COMPLETE
PROTOS_REVISION=9ada962e63e7198b8ae7b468b4c8f9d471bb6cff

SOURCE_FILES_BEFORE=157
SOURCE_FILES_AFTER=22
LOGICAL_CASES_BEFORE=157
LOGICAL_CASES_AFTER=157

VERSION_MANIFEST_ROWS=9
LOCK_MANIFEST_ROWS=10
RESOLUTION_INPUT_MANIFEST_ROWS=3

ALL_TESTS=PASS
NEXT_SLICE=TOOL009-F-I8
```
