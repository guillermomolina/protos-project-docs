# TOOL009-F-I6 — Package TOML consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I6`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `f6daf53bfd803021954681c9ed8307d523711421`
- Commit:
  `TOOL009-F-I6: consolidate Package TOML test sources`

The project owner reported the published F-I6 candidate as uploaded and fully green.

## Scope

F-I6 consolidated the Package TOML suite-native corpus:

```text
protos/tests/package-tool/toml-syntax/
```

Final reconciliation:

```text
F_I6_SOURCE_FILES_BEFORE=102
F_I6_SOURCE_FILES_AFTER=14

F_I6_LOGICAL_CASES_BEFORE=102
F_I6_LOGICAL_CASES_AFTER=102
```

Two target paths already existed in the baseline and were correctly reused
edit-in-place:

```text
string-multiline-basic.protos
string-multiline-literal.protos
```

Therefore the corrected physical bookkeeping is:

```text
CREATED_SOURCES=12
REUSED_SOURCES=2
REMOVED_SOURCES=100
PREEXISTING_TARGET_PATHS=2
```

This corrects the earlier planning-only assumption of 14 created / 0 reused /
102 removed without changing the 102 -> 14 physical-source result or the
102 -> 102 Logical Case result.

## Published manifest

At the product revision:

```text
protos/tests/package-tool/toml-syntax/manifest.tsv = 14 rows
protos/tests/conformance/manifest.tsv = 171 rows
```

The ordinary conformance manifest therefore remains unchanged by F-I6.

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

## Next slice

The next planned publication is F-I7:

```text
TOOL009-F-I7

version:          74 -> 9 sources
lock:             71 -> 10 sources
resolution-input: 12 -> 3 sources

SOURCE_FILES_BEFORE=157
SOURCE_FILES_AFTER=22
LOGICAL_CASES_BEFORE=157
LOGICAL_CASES_AFTER=157
```

F-I8 remains the final project-tree consolidation and integrated closure slice.

## Result

```text
TOOL009_F_I6=COMPLETE
PROTOS_REVISION=f6daf53bfd803021954681c9ed8307d523711421

SOURCE_FILES_BEFORE=102
SOURCE_FILES_AFTER=14
LOGICAL_CASES_BEFORE=102
LOGICAL_CASES_AFTER=102

CREATED_SOURCES=12
REUSED_SOURCES=2
REMOVED_SOURCES=100

PACKAGE_TOML_MANIFEST_ROWS=14
MAIN_CONFORMANCE_MANIFEST_ROWS=171

ALL_TESTS=PASS
NEXT_SLICE=TOOL009-F-I7
```
