# I120 — Published single-slice contextual if/else source migration (metadata reconciliation pending)

**Date:** 2026-10-21  
**Formal work:** [I120 / #946](https://github.com/guillermomolina/protos/issues/946), **one substantive implementation slice**  
**Product repository:** `guillermomolina/protos`  
**Exact published source revision:** [`e552feced508db25950747f446c507b8e65e3dff`](https://github.com/guillermomolina/protos/commit/e552feced508db25950747f446c507b8e65e3dff)  
**Published commit message:** `I120: adopt contextual if/else in first-party Protos sources`  
**Predecessor:** [I119 / #945](https://github.com/guillermomolina/protos/issues/945), product [`2a3d373d06792384f0c62cb06845b1f1d62bea6f`](https://github.com/guillermomolina/protos/commit/2a3d373d06792384f0c62cb06845b1f1d62bea6f), implements ratified D211 contextual syntax  
**Implementation handoff:** [single-slice editor instructions](../../work/I120/I120_SINGLE_SLICE_EDITOR_HANDOFF_2026_10_21.md)  
**Normative specification revision:** `0.1.466` **unchanged** — this is an idiomatic source migration, not new language semantics.

## Delivered source migration

Under the already ratified D211 equivalences, I120 migrates ordinary first-party sources from direct `condition.ifTrue(() => { body })` / `condition.ifTrue() { body }` to contextual `if(condition) { body }`, and from `condition.ifTrueIfFalse(() => { yes }, () => { no })` to `if(condition) { yes } else { no }`. The original condition remains eagerly evaluated exactly once and branch arguments remain freshly constructed lexical Closures. Standard `ifFalse`, `ifTrue`, `ifTrueIfFalse`, `and`, `or`, `not` protocol semantics and the language compiler/runtime are unchanged.

**Maintainer/implementation-agent reported counts:** 3,286 one-arm and 51 two-arm transformations, **3,337 total**, across **219** `.protos` source files. Read-only GitHub verification of the exact commit confirmed **225 changed paths**, distributed as follows:

| Tree | Changed .protos files | One-arm changes (agent-reported) | Two-arm changes (agent-reported) |
|---|---:|---:|---:|
| `protos/lib` | 43 | 1,764 | 50 |
| `protos/tools` | 37 | 899 | 0 |
| `protos/tests` | 118 | 599 | 1 |
| `protos/benchmarks` | 17 | 17 | 0 |
| `protos/examples` | 3 | 3 | 0 |
| `protos/tutorials` | 1 | 2 | 0 |
| README and guide 03 snippets | — | 2 | 0 |
| **Total** | **219** | **3,286** | **51** |

Two documentation files also changed: `README.md` and `docs/guide/03-closures-methods-and-receivers.md`. Four Java source-shape assertions were reconciled against changed tool sources:

1. `src/test/java/com/guillermomolina/protos/execution/tools/test/discovery/ProtosTestToolFileSelectionMainAdoptionTest.java`: `selectorsActive.ifTrue` → `if(selectorsActive)`.
2. `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolI8D5CPublicCutoverTest.java`: `(planLoader === "package-toml").ifTrue` → `if(planLoader === "package-toml")`.
3. `src/test/java/com/guillermomolina/protos/execution/tools/test/execution/ProtosTestToolActorGroupOwnershipArchitectureTest.java`: `(kind === "future-integer-one-of").ifTrue` → `if(kind === "future-integer-one-of")`.
4. `src/test/java/com/guillermomolina/protos/execution/tools/test/reporting/ProtosTestToolTool011ProgressGroupingTest.java`: `listCases.ifTrue` source-order anchor → `if(listCases)`.

## Deliberate exclusions and source-preservation boundary

- Direct protocol/conformance cases under `protos/tests/conformance/boolean` retain the explicit `ifTrue` / `ifTrueIfFalse` spelling where it is precisely what is tested.
- `protos/tools/test/Runner.protos` retains 19 `ifTrue` occurrences in generated evaluator-source string literals checked character-for-character by test facilities.
- The canonical desugaring in `docs/guide/04-control-flow-through-protocols.md` is preserved.
- No `ifFalse` conversion to `if(!condition)`, nor fusion of independent `ifTrue` / `ifFalse` sends, was made.
- Existing embedded Java/Python/shell source snippets with explicit protocol/fixture authority were not mechanically migrated.
- The canonical PERF001-F `protos/benchmarks/concurrency` source set is unchanged; 17 other non-frozen benchmarks were migrated.
- Normative `spec/**` was not changed. The agent reported that only `protos/lib/core/Object.protos` (−6 lines), `protos/lib/math/Integer.protos` (−3), and `protos/lib/cli/CommandLine.protos` (−6) had line-count changes from two-arm rewrites and no known fixed-line dependent assertion; this textual source audit is not a runtime diagnostic guarantee.

## Validation and exact published state

The maintainer reported after the product push:

- `git diff --check` **clean**.
- **All local tests PASS**.

These are explicit **maintainer-reported results**; this record's author did not run builds, scripts or tests and does not claim raw CI or test logs independently verified. The preceding agent report mentioned `make check`, `make test-protos`, `make test-java` as requested gates, but no separate individual-command PASS claims are invented here.

**Verified published commit:** `e552feced508db25950747f446c507b8e65e3dff` contains 225 paths including the four Java assertions; it **does not include** `pom.xml` or `CHANGELOG.md`. Both files still identify the prior implementation `0.3.396-SNAPSHOT` / I119. The project implementation rule in `AGENTS.work/IMPLEMENTATION.md` requires a changed distributable `protos/lib/**` source commit to increment `pom.xml` and carry the matching root `CHANGELOG.md` entry **in the same commit**. As a result, the requested one-commit metadata postcondition was not satisfied by the published I120 commit.

This is a **publication metadata nonconformance**, not evidence that the 3,337 source rewrites failed tests, and not permission for the coordinator to rewrite product history or directly publish to `guillermomolina/protos`. The final issue closure should reconcile the omission explicitly via the maintainer's approved product-publication workflow, retaining this exact source-revision provenance. There is no need to change the normative specification revision.

## Status / required reconciliation

**SOURCE_PUBLISHED=PASS**  
**PROTOS_SOURCE_REVISION=`e552feced508db25950747f446c507b8e65e3dff`**  
**CHANGED_PATHS=225 (verified)**  
**REPORTED_TRANSFORMATIONS=3286+51 (maintainer/agent report)**  
**DIFF_CHECK=MAINTAINER_REPORTED_CLEAN**  
**LOCAL_TESTS=MAINTAINER_REPORTED_PASS**  
**SPEC_REVISION=0.1.466 (unchanged)**  
**IMPLEMENTATION_VERSION_AND_CHANGELOG=NOT_RECONCILED**  
**ISSUE_CLOSURE=PENDING_METADATA_RECONCILIATION**

This record is the revision-bound evidence for the actual source publication and the documented metadata gap; it is **not** a claim that every I120 closure condition is complete.
