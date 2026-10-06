# TOOL011-B — Test Tool progress grouping implementation checkpoint

Status: **IMPLEMENTATION PUBLISHED — CLOSURE REVIEW REQUIRED**

Evidence date: **2026-10-06**

Owning work item:

- TOOL011 / `guillermomolina/protos#803`

Exact revisions:

```text
PROTOS_REVISION=61c475650bf5b5d385e9482b639d310c682e2b0d
PROJECT_DOCS_BASE_REVISION=adb2cb4e03351878caa29ade940a790f4940c0f3
```

Published product commit:

```text
TOOL011-B: replace positional progress phases with authoritative repository grouping
```

Implementation version:

```text
0.3.235-SNAPSHOT
```

Nature: non-normative implementation evidence. This record does not define Protos language or Standard Library semantics.

## Maintainer-reported validation

The maintainer reports, for the published TOOL011-B candidate:

```text
git diff --check = PASS
all local tests = PASS
publication/push = PASS
```

The available GitHub commit-status interface exposes no status checks for the exact product revision at the time of this checkpoint. This record therefore preserves the maintainer-reported local validation without manufacturing remote-CI evidence.

## Implemented architecture

At exact Protos revision `61c475650bf5b5d385e9482b639d310c682e2b0d`:

- the independent positional `phaseNames` mapping is removed from `protos/tools/test/Main.protos`;
- ordinary repository progress no longer has the historical `[main]` group;
- repository-specific progress grouping policy is owned by `protos/tools/test/RepositorySuite.protos`;
- ordinary repository leaves derive their displayed group name from SuiteId by stripping the exact `protos/` prefix;
- grouping is projected only over final retained Logical Cases after existing selection/discovery completes;
- `--list-cases` remains on its list-only path before progress creation and scheduling;
- empty progress groups are omitted;
- execution-suite identity remains separate from progress-group identity;
- Logical Case completion is attributed to its progress-group observer without changing the LogicalCaseRunner schedule;
- SuiteId, CorpusId, ExecutionRequirementId, CaseRef, selector meaning, Process isolation, result classification and exit-code authority remain outside the progress-group policy; and
- repository grouping violations fail closed rather than falling into `main`, `other`, `misc` or another catch-all.

## Actual published conformance grouping

The published implementation contains ten ordered `protos/conformance` presentation groups:

```text
conformance/values
conformance/object-model
conformance/control-concurrency
conformance/io
conformance/standard-library/toml-official
conformance/standard-library/toml
conformance/standard-library/collections
conformance/standard-library/data-text-test
conformance/language-surface
conformance/regression-maturity
```

The four Standard Library groups are classified from the directory of authoritative `Manifest.casePath` values:

```text
library/toml/official/... -> conformance/standard-library/toml-official
library/toml/<direct-file> -> conformance/standard-library/toml
library/collections/... -> conformance/standard-library/collections
library/json/... | library/text/... | library/test/... -> conformance/standard-library/data-text-test
```

Unknown or multiply matching conformance directories fail closed.

## Reconciliation with TOOL011-A

The retained TOOL011-A investigation at:

```text
docs/project/evidence/TOOL011/TOOL011_A_PROGRESS_GROUPING_INVESTIGATION.md
```

selected Candidate C with the same architecture and invariants, but specified **seven** conformance groups, including one aggregate:

```text
conformance/standard-library
```

that owned every path whose first `Manifest.casePath` component was `library`.

TOOL011-B therefore matches TOOL011-A on:

```text
GROUPING_AUTHORITY=RepositorySuite
POSITIONAL_PHASE_NAMES=REMOVED
MAIN_GROUP=REMOVED
ORDINARY_NAMES=SUITE_ID_DERIVED
GROUPING_AFTER_FINAL_SELECTION=YES
EMPTY_GROUPS=OMITTED
LIST_CASES_EXECUTION_CHANGE=NO
EXECUTION_SUITE_IDENTITY_SEPARATE=YES
UNKNOWN_GROUPING=FAIL_CLOSED
EXECUTION_SEMANTICS_CHANGE=NO
```

but does not exactly match the retained A presentation partition:

```text
TOOL011_A_CONFORMANCE_GROUPS=7
TOOL011_B_CONFORMANCE_GROUPS=10
PRESENTATION_POLICY_DELTA=STANDARD_LIBRARY_SPLIT_INTO_FOUR_GROUPS
```

Because progress-group names and boundaries are externally visible Test Tool policy, this checkpoint does not silently rewrite TOOL011-A or claim exact-candidate closure.

## Closure assessment

The published implementation satisfies the broad Issue #803 goals of removing `[main]`, eliminating the positional label array, introducing meaningful stable grouping, preserving selection/execution ownership, retaining fail-closed grouping, and adding focused regression coverage.

However, final closure requires one explicit reconciliation of the seven-versus-ten-group presentation policy:

```text
OPTION_A=accept the published ten-group Standard Library refinement as final TOOL011 policy
OPTION_B=reconcile product implementation back to the seven-group TOOL011-A partition
```

If the maintainer selects Option A, no further technical implementation slice is required; only final live-Issue/durable-record reconciliation and closure remain.

If the maintainer selects Option B, a bounded product correction is required before closure.

Until that reconciliation:

```text
TOOL011_B_IMPLEMENTATION=PUBLISHED
LOCAL_VALIDATION=PASS_REPORTED
ISSUE_STATUS=REVIEW
ISSUE_CLOSABLE=NO
NEXT_TECHNICAL_SLICE=NONE_UNLESS_SEVEN_GROUP_POLICY_IS_RESTORED
```
