# TOOL009-F-I4 — JSON conformance consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I4`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `8431be72e1fe5f0c6677eb768e07716bd85d894b`
- Commit:
  `TOOL009-F-I4: consolidate JSON conformance sources`

The project owner reported the published F-I4 candidate as uploaded and all
tests passing.

## Scope

F-I4 consolidated only:

```text
protos/tests/conformance/library/json/
```

The approved reconciliation is:

```text
F_I4_SOURCE_FILES_BEFORE=89
F_I4_SOURCE_FILES_AFTER=18

F_I4_LOGICAL_CASES_BEFORE=89
F_I4_LOGICAL_CASES_AFTER=89

FILES_REMOVED=89
FILES_CREATED=18
FILES_REUSED=0
```

The 18 published target sources are:

```text
constructor-errors.protos
constructors.protos
encoder-errors.protos
encoder-positive.protos
event-parser-errors.protos
event-parser-positive.protos
event-writer-errors.protos
event-writer-positive.protos
final-deep-stress.protos
final-large-materialization.protos
final-streaming-roundtrip.protos
import-surface.protos
parser-number-errors.protos
parser-positive.protos
parser-structural-errors.protos
parser-unicode-errors.protos
text-adapter-reader.protos
text-adapter-writer.protos
```

## Manifest reconciliation

The previous F-I3 checkpoint left 242 conformance manifest rows, including 89
JSON sources.

At the F-I4 product revision:

```text
JSON_MANIFEST_ROWS=18
CONFORMANCE_MANIFEST_ROWS=171
```

This matches the planned physical consolidation exactly.

## Validation

The project owner reported:

```text
TESTS=PASS
HUMAN_REPORTED_VALIDATION=PASS
```

No command-level validation details beyond that handoff are invented by this
record.

## Boundaries preserved

```text
LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE
STANDARD_LIBRARY_SEMANTICS_CHANGE=NONE
LANGUAGE_SEMANTICS_CHANGE=NONE
SPEC_CHANGE=NONE
```

F-I4 did not consume Process, Actor, Group, or Package Tool consolidation work.

## Next slice

The next planned publication is F-I5:

```text
TOOL009-F-I5

PROCESS_SNAPSHOT=15 -> 3 sources / 15 Cases
ACTOR=11 -> 4 sources / 11 Cases
GROUP=10 -> 3 sources / 10 Cases

F_I5_SOURCE_FILES_BEFORE=36
F_I5_SOURCE_FILES_AFTER=10
F_I5_LOGICAL_CASES_BEFORE=36
F_I5_LOGICAL_CASES_AFTER=36
```

Actor and Group retain their existing `modules/workers.protos` bootstrap
overlays. The overlays are not Logical Case sources and are not part of the
36 -> 10 count.

## Result

```text
TOOL009_F_I4=COMPLETE
PROTOS_REVISION=8431be72e1fe5f0c6677eb768e07716bd85d894b

SOURCE_FILES_BEFORE=89
SOURCE_FILES_AFTER=18
LOGICAL_CASES_BEFORE=89
LOGICAL_CASES_AFTER=89

CONFORMANCE_MANIFEST_ROWS=171
HUMAN_REPORTED_VALIDATION=PASS

NEXT_SLICE=TOOL009-F-I5
```
