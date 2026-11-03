# BUG033 — TOOL010 D237 named-argument preservation and closure evidence

- **Work item:** [BUG033 #1041](https://github.com/guillermomolina/protos/issues/1041).
- **Date:** 2026-11-03.
- **Product repository:** `guillermomolina/protos`.
- **Published product revision:** [`489db6e1dc18554d7b2e5c9a86eca5f8d157f660`](https://github.com/guillermomolina/protos/commit/489db6e1dc18554d7b2e5c9a86eca5f8d157f660), on `main` (confirmed by the maintainer's successful push and independent GitHub readback).
- **Baseline revision:** [`2c89adf6062f212c4d9e20f68b919e843a05b523`](https://github.com/guillermomolina/protos/commit/2c89adf6062f212c4d9e20f68b919e843a05b523).
- **Implementation version:** `0.3.441-SNAPSHOT`, verified in the published product commit and changelog.
- **Evidence provenance:** maintainer-executed local validation, followed by GitHub commit verification; no independent build, test, native-image, or VSIX run by the evidence recorder.

## Defect and published correction

After D237 contextual named arguments were implemented in I143, the TOOL010 source-layout bridge projected each `SurfaceArgument` with its spread flag and expression, but omitted its optional name. The Protos-first `Structural.formatArgument` consequently reconstructed labeled arguments as positional arguments. This was a source-inspection-confirmed semantic data-loss path; a separate pre-fix runtime reproduction of the exact formatting output was **not** executed and is not claimed.

The published repair preserves each argument's canonical optional `name` and existing structural `path` through `ProtosSourceLayoutToolBridge.arguments`. The bundled Protos formatter reconstructs `label: expression` directly from that projection, without parsing the source text again. Positional arguments, spreads, grouped positional slot creation, source order, ordinary member and `super` sends, nested calls, Array construction with ordinary shadowing, Unicode NFC names, and trailing Closures remain distinct.

For comments around the named-argument prefix, `ProtosSourceLayoutView` introduces the minimal `NAMED_ARGUMENT_BEFORE_COLON` and `NAMED_ARGUMENT_AFTER_COLON` attachment kinds. These carry the precise argument/value structural paths into the existing trivia projection; `Structural.protos` reuses the existing boundary-comment formatter across embedded, end-of-line, and whole-line placements where syntactically valid. An invalid `//` placement before the colon is not admitted by changing the parser.

The unchanged normative authority remains D237 in `spec/PROTOS_GRAMMAR.md` §17 and §12.3 and `spec/semantics/CALLABLES.md`, alongside the D183/PLAT050 preservation contract. This is an implementation/tooling repair, not a new syntax or runtime semantic decision and not a separate VS Code formatter.

## Exact published product delta

The [published product commit](https://github.com/guillermomolina/protos/commit/489db6e1dc18554d7b2e5c9a86eca5f8d157f660) contains **9 changed paths, 554 insertions and 8 deletions**:

1. `protos/tools/formatter/Structural.protos` — render named labels; generalize existing token-boundary comment handling.
2. `src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java` — attach comments on both sides of named-argument colons to their structural owner/value.
3. `src/main/java/com/guillermomolina/protos/execution/host/tooling/ProtosSourceLayoutToolBridge.java` — export argument names and paths.
4. `src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java` — nested argument/comment attachment tests.
5. `src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java` — stdin/file formatting, duplicate-name fail-closed handling, and unchanged actual execution result.
6. `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosFormatterToolBootstrapTest.java` — exact bridge projection; D237 form matrix; canonical semantic equivalence, comments and idempotence.
7. `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java` — actual LSP formatting, edit/reformat and invalid-source handling.
8. `pom.xml` — `0.3.440-SNAPSHOT` to `0.3.441-SNAPSHOT`.
9. `CHANGELOG.md` — corresponding BUG033 entry.

No normative specification, parser, canonicalizer, runtime invocation path, VS Code extension repository, or release artifact changed.

## Maintainer-reported validation

| Gate | Result | Evidence |
| --- | --- | --- |
| Initial formatter focal class | **PASS: 34/34** | Maintainer's Maven output after the first named-label/bridge patch |
| Full four-class regression set after comment repair | **PASS: 106/106** | `ProtosSourceLayoutViewTest` 35, `ProtosCliTest` 22, `ProtosFormatterToolBootstrapTest` 35, `ProtosLanguageServerFormattingTest` 14 |
| Batch 3 affected regression classes | **PASS: 59/59** | `ProtosCliTest` 23 and `ProtosFormatterToolBootstrapTest` 36 |
| Final `make test` | **PASS** | Maintainer report on the final executable-impact candidate, before the metadata bump |
| Final `make check` | **PASS** | Same provenance |
| Staged `git diff --cached --check` | **PASS** | Maintainer terminal output; exact staged diff reviewed before finalization |
| Version/changelog metadata finalization | **PASS** | Preflight at baseline HEAD, `0.3.440-SNAPSHOT` → `0.3.441-SNAPSHOT`; tests intentionally not rerun after metadata-only change |
| Product commit, push, clean working tree | **PUBLISHED** | Maintainer shell output; exact product SHA independently confirmed by GitHub |

During targeted regression development, an incorrectly escaped Java test fixture and then a syntactically invalid comment-before-colon fixture caused intermediate test failures. The fixtures were corrected without weakening the parser or altering D237. Subsequent focal suites and the final integrated gates passed.

The runtime regression specifically verified that calling a three-parameter Closure with `third: 3, first: 1, second: 2` still prints `123` after formatting. Canonical equivalence tests additionally distinguish `f(a: 1)` from grouped positional `f((a: 1))`. These are verified as maintainer-reported tests, not independently re-executed in this evidence workflow.

**Evidence limits:** no independent pre-fix CLI execution, Native Image, published VSIX acceptance, CI run, performance measurement, or external audit is claimed. The published product tree, diff and version identity are independently GitHub-verifiable.

## Closure identity and authority

```text
WORK_ITEM=BUG033
ISSUE=guillermomolina/protos#1041
PROTOS_REVISION=489db6e1dc18554d7b2e5c9a86eca5f8d157f660
BASELINE_PROTOS_REVISION=2c89adf6062f212c4d9e20f68b919e843a05b523
IMPLEMENTATION_VERSION=0.3.441-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=9
HUMAN_FORMATTER_FOCAL=34/34_PASS
HUMAN_FOUR_CLASS_FOCAL=106/106_PASS
HUMAN_BATCH_3_FOCAL=59/59_PASS
HUMAN_MAKE_TEST=PASS
HUMAN_MAKE_CHECK=PASS
HUMAN_DIFF_CHECK=PASS
SPEC_CHANGE=NONE
DURABLE_RECORD_DECISION=REQUIRED_BY_BUG033_CONTRACT_AND_OWNER
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

This file is an immutable, non-normative publication evidence snapshot. [BUG033 #1041](https://github.com/guillermomolina/protos/issues/1041) remains the live coordination and closure authority. Its final closure comment must link this record at the **exact project-docs publication revision**, alongside the exact product revision; a moving `main` is not sufficient for cross-repository closure evidence.
