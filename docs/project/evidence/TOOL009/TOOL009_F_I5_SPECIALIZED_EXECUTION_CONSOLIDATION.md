# TOOL009-F-I5 — Specialized execution corpus consolidation

## Identity

- Parent work item: `TOOL009-F / guillermomolina/protos#694`
- Slice: `TOOL009-F-I5`
- Product repository: `guillermomolina/protos`
- Published product revision:
  `f586752d8009a7a27db59a41e5d868d045c29fb3`
- Commit:
  `TOOL009-F-I5: consolidate specialized execution corpus sources`

The project owner reported the published F-I5 candidate as uploaded with all tests passing.

## Scope

F-I5 consolidated the three specialized execution corpora:

```text
Process Snapshot: 15 -> 3 sources / 15 Logical Cases
Actor:            11 -> 4 sources / 11 Logical Cases
Group:            10 -> 3 sources / 10 Logical Cases
```

Overall:

```text
F_I5_SOURCE_FILES_BEFORE=36
F_I5_SOURCE_FILES_AFTER=10
F_I5_LOGICAL_CASES_BEFORE=36
F_I5_LOGICAL_CASES_AFTER=36
```

## Published manifests

At the product revision:

```text
protos/tests/conformance/process/manifest.tsv = 3 rows
protos/tests/conformance/actor/manifest.tsv   = 4 rows
protos/tests/conformance/group/manifest.tsv   = 3 rows

SPECIALIZED_MANIFEST_ROWS=10
```

The published target sources are:

```text
process/args.protos
process/environment.protos
process/snapshot-and-process-surface.protos

actor/current-and-identity.protos
actor/spawn.protos
actor/request-order-and-state.protos
actor/lifecycle-and-transfer.protos

group/acquisition-identity-and-transfer.protos
group/request-routing.protos
group/stopped-and-surface.protos
```

## Bootstrap overlays

The Actor and Group workers overlays remain physically separate and retained:

```text
protos/tests/conformance/actor/modules/workers.protos
protos/tests/conformance/group/modules/workers.protos
```

They are bootstrap/module-overlay fixtures, not Logical Case sources, and are not
part of the 36 -> 10 source count.

## Main conformance manifest

The ordinary conformance manifest remains unchanged by F-I5:

```text
protos/tests/conformance/manifest.tsv = 171 rows
```

This preserves the isolation between the ordinary conformance corpus and the
three specialized execution corpora.

## Validation

The project owner reported:

```text
ALL_TESTS=PASS
HUMAN_REPORTED_VALIDATION=PASS
```

This durable record does not invent more granular command output beyond that
reported completion state.

## Boundaries preserved

```text
LOGICAL_CASE_COVERAGE_CHANGE=NONE
TEST_TOOL_SEMANTICS_CHANGE=NONE
EXECUTION_REQUIREMENT_CHANGE=NONE
BOOTSTRAP_CHANGE=NONE
ACTOR_WORKERS_OVERLAY=PRESERVED
GROUP_WORKERS_OVERLAY=PRESERVED
SPEC_CHANGE=NONE
```

## Next slice

The next planned publication is F-I6:

```text
TOOL009-F-I6
Package TOML

SOURCE_FILES_BEFORE=102
SOURCE_FILES_AFTER=14
LOGICAL_CASES_BEFORE=102
LOGICAL_CASES_AFTER=102
```

F-I6 is isolated from Package version/lock/resolution-input (F-I7) and Package
project-tree/final closure (F-I8).

## Result

```text
TOOL009_F_I5=COMPLETE
PROTOS_REVISION=f586752d8009a7a27db59a41e5d868d045c29fb3

SOURCE_FILES_BEFORE=36
SOURCE_FILES_AFTER=10
LOGICAL_CASES_BEFORE=36
LOGICAL_CASES_AFTER=36

PROCESS_SOURCES_AFTER=3
ACTOR_SOURCES_AFTER=4
GROUP_SOURCES_AFTER=3

MAIN_CONFORMANCE_MANIFEST_ROWS=171
ALL_TESTS=PASS

NEXT_SLICE=TOOL009-F-I6
```
