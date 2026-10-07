# TOOL012 — Final closure

Status: **COMPLETED**

Closure date: **2026-10-07**

Owning work item:

- TOOL012 / `guillermomolina/protos#814`

Related completed work:

- TOOL011 / `guillermomolina/protos#803`

Exact product revision:

```text
PROTOS_REVISION=8bd5c37c7cca13599df39d18de2eadf01e4a86eb
```

Product commit:

```text
TOOL012: bound toml-official progress groups to at most 100 Cases
```

Implementation version:

```text
0.3.256-SNAPSHOT
```

Project-record base revision before this closure record:

```text
PROJECT_DOCS_BASE_REVISION=20a09b8a1b3c5a23db6931203b0c588a57526c59
```

Nature: non-normative implementation/closure evidence. This record does not define Protos language or Standard Library semantics.

## Published implementation

At exact Protos revision `8bd5c37c7cca13599df39d18de2eadf01e4a86eb`, TOOL012 replaces the former single `conformance/standard-library/toml-official` progress bucket with deterministic bounded subgroups while retaining `RepositorySuite` as the single repository progress-grouping authority.

The published changelog records the concrete projection:

- each retained TOML-official Case presents under `conformance/standard-library/toml-official/<directory>`;
- `<directory>` is derived from the Case's D153 selector without its final segment, preserving upstream toml-test direction/directory structure;
- root-level Cases use their stable root category such as `valid`;
- the current corpus yields **39** TOML-official progress subgroups;
- the largest current subgroup contains **76** retained Logical Cases, below the required maximum of 100;
- subgroup order follows the first retained Case in authoritative plan order;
- focused runs therefore materialize only groups that actually own retained selected Cases;
- malformed selectors and presented-group mismatches fail closed.

```text
MAX_RETAINED_CASES_PER_TOML_OFFICIAL_PROGRESS_GROUP=100
CURRENT_TOML_OFFICIAL_PROGRESS_GROUPS=39
CURRENT_MAX_TOML_OFFICIAL_GROUP_CASES=76
BOUND=PASS
```

## Implementation architecture

The product commit changes Test Tool presentation machinery rather than Case or scheduler semantics:

- `protos/tools/test/RepositorySuite.protos` owns subgroup construction and classification;
- `protos/tools/test/Main.protos` retains, after selection, the presentation-only tuple needed for grouping: execution-suite index, authoritative `Manifest.casePath`, and retained Case selector;
- `Main` asks `RepositorySuite` for group names from each suite's final retained Case set and asks it for each Case's group index;
- grouping therefore remains downstream of selection and upstream only of progress presentation/attribution;
- no second independent grouping authority is introduced;
- TOOL011's fail-closed repository grouping model is preserved.

The implementation deliberately derives subgroup identity from authoritative retained Case/corpus information instead of maintaining a second per-Case enumeration.

## Preserved invariants

TOOL012 is presentation-only. The published change does not redefine:

```text
Manifest.casePath
Logical Case identity
CaseRef identity
discovery
rematerialization
--file semantics
--directory semantics
exact --case semantics
--list-cases semantics
--jobs
scheduler behavior
ExecutionRequirement/resource authority
fresh-Process isolation
pass/fail/tool-error classification
final aggregate counts
TOML parsing/encoding semantics
Protos language semantics
Standard Library semantics
```

Failure attribution uses the bounded subgroup that owns the failing Case; it does not change the failure itself or execution ownership.

## Regression evidence in the product commit

The exact product revision adds focused TOOL012 regression coverage in:

```text
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool012TomlOfficialProgressGroupingTest.java
```

and adapts the existing TOOL011/TOOL004-C guards to the retained-Case-aware RepositorySuite interface.

The commit/changelog records focused coverage for:

- the 100-Case upper bound over the real TOML-official corpus;
- complete and unique TOML-official Case coverage;
- deterministic subgroup names/order;
- preservation of the `--directory` selected Case set;
- focal `--file` presentation;
- exact `--case` presentation;
- fail-closed malformed selectors/group names;
- preservation of TOOL011 progress-group authority.

## Validation and publication evidence

The maintainer reported for the published exact candidate:

```text
git diff --check = PASS
all local tests = PASS
publication/push = PASS
```

These are maintainer-reported local validation results. No additional remote-CI result is invented by this record.

The exact published product commit includes version finalization to `0.3.256-SNAPSHOT` and the matching `CHANGELOG.md` entry.

## Acceptance reconciliation

Issue #814 acceptance is satisfied:

```text
ZERO_TOML_OFFICIAL_GROUPS_OVER_100=PASS
EXACT_CASE_UNION_PRESERVED=PASS
EACH_CASE_EXACTLY_ONE_SUBGROUP=PASS
DETERMINISTIC_NAMES_AND_ORDER=PASS
NATURAL_CORPUS_STRUCTURE_USED=PASS
FILE_SELECTION_PRESERVED=PASS
DIRECTORY_SELECTION_PRESERVED=PASS
EXACT_CASE_SELECTION_PRESERVED=PASS
LIST_CASES_SEMANTICS_PRESERVED=PASS
SCHEDULING_AND_JOBS_PRESERVED=PASS
RESOURCE_AUTHORITY_PRESERVED=PASS
PROCESS_ISOLATION_PRESERVED=PASS
RESULT_CLASSIFICATION_PRESERVED=PASS
AGGREGATE_COUNTS_PRESERVED=PASS
FAILURE_ATTRIBUTION_TO_BOUNDED_SUBGROUP=PASS
FOCUSED_REGRESSION_COVERAGE=PASS
LOCAL_FULL_VALIDATION=PASS_REPORTED
SPECIFICATION_CHANGE=NO
```

## Closure result

```text
TOOL012=COMPLETED
PROTOS_REVISION=8bd5c37c7cca13599df39d18de2eadf01e4a86eb
IMPLEMENTATION_VERSION=0.3.256-SNAPSHOT
SPECIFICATION_CHANGE=NO
OWNER_DECISION_REQUIRED=NO
NEXT_TECHNICAL_SLICE=NONE
ISSUE_814_CLOSABLE=YES
```

No further TOOL012 slice is required by the accepted issue contract.
