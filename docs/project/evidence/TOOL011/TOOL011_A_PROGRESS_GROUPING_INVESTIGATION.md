# TOOL011-A — Test Tool progress grouping investigation

Status: **COMPLETED INVESTIGATION — IMPLEMENTATION READY**

Evidence date: **2026-10-06**

Owning work item:

- TOOL011 / `guillermomolina/protos#803`

Audited revisions:

```text
PROTOS_REVISION=b2b132af1338e474857a0e1c9012f2c32f56e869
PROJECT_DOCS_BASE_REVISION=c9d780de0dedf6e520ab586398971130135277f4
```

Classification:

```text
CLASSIFICATION=INVESTIGATION
EXECUTION=NONE
OWNER_DECISION_REQUIRED=NO
```

Nature: non-normative investigation evidence. This record does not change Protos language semantics, Test Tool execution semantics, scheduling, isolation, selector meaning, Case identity, Corpus authority, ExecutionRequirement authority, or final result classification.

## Purpose

TOOL011-A investigated how to replace the opaque historical `[main]` Test Tool progress phase with meaningful, reasonably granular progress groups without changing the executed Logical Case set or any execution authority.

The investigation was required to leave one exact implementation candidate, not another research phase.

## Current model

At the audited Protos revision:

- `RepositorySuite.root` is the authoritative repository suite composition.
- The first leaf is `protos/conformance` / `protos/corpus/conformance` / `protos/test/ordinary`.
- `SuiteGraph` owns SuiteId, CorpusId and ExecutionRequirementId structure and should remain unchanged by TOOL011.
- `Main.protos` contains an independent positional `phaseNames` array whose first value is `main` and indexes it in parallel with `SuiteGraph.flattenLeaves(RepositorySuite.root)`.
- the positional size check detects only differing lengths; it cannot prevent same-size reorder/name drift.
- final source selection and exact `--case` selection occur before progress begins.
- `Manifest.casePath(spec)` remains available for every final retained source/Logical Case.
- `--list-cases` already exits before progress creation and scheduling.

The current conformance manifest has stable path-domain structure including ordinary value, object model, control/concurrency, I/O, Standard Library, language-surface, regression and maturity areas.

## Candidate evaluation

```text
A=REJECTED
```

Using SuiteId alone removes `main` but leaves approximately 1844 Logical Cases under one `protos/conformance` bucket.

```text
B=REJECTED_AS_FINAL_POLICY
```

Automatically exposing every first `Manifest.casePath` component is structurally safe but produces too many progress groups and multiplies bounded milestone output. Coarsening those roots necessarily becomes presentation policy and should therefore be explicit rather than hidden in a heuristic.

```text
C=SELECTED
```

Repository-owned presentation policy: ordinary repository leaves derive their progress name mechanically from SuiteId, while the existing `protos/conformance` leaf owns one explicit ordered fail-closed partition over authoritative `Manifest.casePath` roots.

```text
D=REJECTED
```

Splitting conformance into real leaves/corpora would alter execution authority and risks changing CorpusId/Case identity or introducing invasive plan-partitioning machinery solely for presentation.

## Selected grouping authority

The grouping policy belongs in `protos/tools/test/RepositorySuite.protos`.

It does not belong in:

- `Progress.protos`, which remains a generic progress formatter/counter;
- `Manifest.protos`, which owns source-plan/path representation rather than repository presentation policy; or
- generic `SuiteGraph.protos`, whose established responsibility is suite/corpus/execution-requirement composition.

For every ordinary repository leaf, the progress name is derived mechanically by requiring a `protos/` SuiteId prefix and stripping that prefix.

Examples:

```text
protos/actor                -> actor
protos/library/uri          -> library/uri
protos/package-tool/version -> package-tool/version
```

There is no parallel positional label array.

## Exact conformance progress groups

The `protos/conformance` leaf is partitioned into exactly seven ordered groups.

### `conformance/values`

Accepted first `Manifest.casePath` components:

```text
integer
float
numeric-conversion
boolean
numeric-equality
equality
collections
number
string
bytes
```

### `conformance/object-model`

```text
call
object
reflection
object-structural
matching
execution-context
```

### `conformance/control-concurrency`

```text
error
control
future
```

### `conformance/io`

```text
path
encoding
text-reader
text-writer
network
```

### `conformance/standard-library`

```text
library
```

Every path whose first component is exactly `library` belongs here, including current `library/collections`, `library/test`, `library/json`, `library/toml` and `library/text` sources.

### `conformance/language-surface`

```text
core-surface
surface-sugar
```

### `conformance/regression-maturity`

```text
regression
maturity
```

These rules exhaust the current top-level roots in `protos/tests/conformance/manifest.tsv` at the audited revision.

A future conformance source with an unknown top-level path component must fail closed before progress/scheduling. There is no `other`, `misc`, `main` or catch-all fallback.

## Other repository progress groups

Every other current leaf contributes one group by stripping `protos/` from its SuiteId:

```text
process-snapshot
actor
group
package-toml
library/uri
library/csv
library/cli
library/math/integer
library/crypto/sha256
library/network/ip-addresses
library/network/ip-endpoints
package-tool/version
package-tool/lock
package-tool/resolution-input
package-tool/content-identity
package-tool/resolution-input-lock
package-tool/resolution-root
package-tool/execution-plan
package-tool/project-projection
```

An unfiltered current repository run therefore has at most 26 progress groups: seven conformance domains plus the nineteen other repository leaves.

## Selection and execution behavior

Grouping is a presentation projection over the final retained Logical Case set.

Required behavior:

- `--file`: preserve current FileSelection semantics; emit only groups containing final selected Logical Cases.
- `--directory`: same rule.
- exact `--case`: preserve current CaseSelection semantics; a one-Case selection creates only that Case's group.
- combined selectors: preserve current intersection/selection semantics.
- unfiltered invocation: classify every retained Logical Case.
- `--list-cases`: preserve the current list-only path; it must not create progress state or schedule execution merely for grouping.

Each group's denominator is the number of final retained Logical Cases assigned to it. The union of all groups must equal the final selected Logical Case set exactly once.

The implementation must keep execution-suite identity separate from progress-group identity. `LogicalCaseRunner` remains one flat execution schedule and all CorpusId/ExecutionRequirement dispatch continues to use the existing authoritative suite leaf.

## Compatibility result

Preserved:

```text
SuiteId
CorpusId
CorpusBinding
ExecutionRequirementId
CaseRef / Logical Case identity
manifest loading
discovery
--file
--directory
--case
--list-cases
LogicalCaseRunner scheduling
jobs/resource handling
Process isolation
pass/fail/tool-error classification
invocation exit-code calculation
```

Deliberately changed presentation:

- `[main]` disappears.
- historical short labels that hide namespace become SuiteId-derived names, for example `uri` -> `library/uri` and `package-tool-version` -> `package-tool/version`.

This is an intentional presentation/output compatibility break authorized by TOOL011, not a Test Tool execution compatibility break.

## Expected implementation delta

One coherent implementation slice should normally touch:

```text
protos/tools/test/RepositorySuite.protos
protos/tools/test/Main.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool004CProgressPresentationTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool011ProgressGroupingTest.java
```

Optionally, if a Protos fixture materially simplifies the focused regression:

```text
protos/tests/tooling/tool011-progress-grouping.protos
```

No TOOL011 mechanism change is expected in `SuiteGraph.protos`, `Manifest.protos`, `Progress.protos`, the conformance manifest, FileSelection, CaseSelection, LogicalCaseRunner, Corpus bindings, ExecutionRequirement bindings, scheduling or Process infrastructure.

## Required regression coverage

The implementation must cover at least:

1. no repository `phaseNames` positional mapping;
2. no ordinary repository `[main]` group;
3. all seven exact conformance names;
4. classification coverage for every accepted current conformance top-level root;
5. fail-closed handling for an unknown conformance root;
6. ordinary leaf naming derived from SuiteId;
7. stability when leaf declaration order changes;
8. file/directory/exact-Case selections creating only relevant groups;
9. exact once-only partition of the final selected Logical Case set;
10. aggregate progress counts preserving the final selected Logical Case count;
11. unchanged invocation aggregate result classification;
12. unchanged `--list-cases` list-only behavior; and
13. unchanged SuiteGraph SuiteId/CorpusId/ExecutionRequirementId identities.

## Implementation routing

```text
NEXT_SLICE=TOOL011-B
NEXT_CLASSIFICATION=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
NEXT_OBJECTIVE=replace positional progress phases with authoritative repository progress grouping
```

TOOL011-B should implement the repository grouping policy, `Main.protos` projection/wiring, and focused regressions together as one productive slice. It should not be subdivided into one-file micro-slices.

If its focused/static validation passes, TOOL011 is a top-level executable Tool item and final closure requires the repository's applicable integrated full validation policy before publication/closure.

## Conclusion

```text
TOOL011_A=PASS
SELECTED_CANDIDATE=C
GROUPING_AUTHORITY=RepositorySuite
CONFORMANCE_GROUPS=7
POSITIONAL_PHASE_NAMES=REMOVE
UNKNOWN_CONFORMANCE_DOMAIN=FAIL_CLOSED
EXECUTION_SEMANTICS_CHANGE=NO
OWNER_DECISION_REQUIRED=NO
NEXT=TOOL011-B_IMPLEMENTATION
```
