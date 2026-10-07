# TOOL009-F-I3 — Error, regression and non-JSON library consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I3`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `44382cba6d691a1eae8bf74be6ed3e15eda6f0cd`
- Commit:
  `TOOL009-F-I3: consolidate error and non-JSON library test sources`

The project owner reported the published F-I3 candidate as uploaded and tested.

## Scope

F-I3 consolidated:

```text
conformance/error
conformance/regression
conformance/maturity (the 21 manifest-owned sources only)
conformance/library/collections
conformance/library/test
conformance/library/text

repository-explicit:
uri
csv
cli
math/integer
crypto/sha256
network/ip-addresses
network/ip-endpoints
```

The accepted slice reconciliation is:

```text
F_I3_SOURCE_FILES_BEFORE=240
F_I3_SOURCE_FILES_AFTER=59

F_I3_LOGICAL_CASES_BEFORE=240
F_I3_LOGICAL_CASES_AFTER=240

CONFORMANCE_SOURCES_BEFORE=162
CONFORMANCE_SOURCES_AFTER=36

REPOSITORY_EXPLICIT_SOURCES_BEFORE=78
REPOSITORY_EXPLICIT_SOURCES_AFTER=23

FILES_REMOVED=234
FILES_CREATED=53
FILES_REUSED=6
```

## Current repository reconciliation

At the published product revision, the conformance manifest has exactly 242
live rows and the F-I3 families are at their planned targets:

```text
error=4
regression=5
maturity=4
library/collections=14
library/test=5
library/text=4
library/json=89

CONFORMANCE_MANIFEST_ROWS=242
```

The seven repository-explicit library corpora have exactly:

```text
uri=3
csv=5
cli=5
math/integer=4
crypto/sha256=2
network/ip-addresses=3
network/ip-endpoints=1

TOTAL=23
```

`RepositoryCorpusPlans.protos` and
`ProtosTestToolRepositoryCorpusPlansTest.java` were part of the F-I3
publication so the exact explicit membership remains bound to the consolidated
physical sources rather than obsolete aliases.

## Validation

The project owner reported the published candidate as tested.

```text
HUMAN_REPORTED_VALIDATION=PASS
```

This durable record does not invent command-level validation details that were
not supplied in the completion handoff.

## Boundaries preserved

```text
LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE
LANGUAGE_SEMANTICS_CHANGE=NONE
STANDARD_LIBRARY_SEMANTICS_CHANGE=NONE
SPEC_CHANGE=NONE
```

The nested `Test(...)` values inside the existing `library/test` behavioral
tests remain runtime data rather than extra Logical Cases.

## Next slice

F-I4 is the isolated JSON consolidation:

```text
TOOL009-F-I4

CURRENT_JSON_SOURCES=89
TARGET_JSON_SOURCES=18
CURRENT_JSON_LOGICAL_CASES=89
TARGET_JSON_LOGICAL_CASES=89

CONFORMANCE_MANIFEST_ROWS_BEFORE=242
CONFORMANCE_MANIFEST_ROWS_AFTER=171
```

F-I4 does not include Process, Actor, Group, or Package Tool corpora.

## Result

```text
TOOL009_F_I3=COMPLETE
PROTOS_REVISION=44382cba6d691a1eae8bf74be6ed3e15eda6f0cd

SOURCE_FILES_BEFORE=240
SOURCE_FILES_AFTER=59
LOGICAL_CASES_BEFORE=240
LOGICAL_CASES_AFTER=240

F_I3_RECONCILIATION=PASS
HUMAN_REPORTED_VALIDATION=PASS

NEXT_SLICE=TOOL009-F-I4
```
