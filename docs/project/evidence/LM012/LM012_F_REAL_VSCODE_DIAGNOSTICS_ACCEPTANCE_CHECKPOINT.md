# LM012-F — real VS Code diagnostics acceptance checkpoint

**Date:** 2026-10-09. **Owner:** [LM012 / guillermomolina/protos#671](https://github.com/guillermomolina/protos/issues/671). **Role:** historical, non-normative human-observed editor evidence and acceptance gate ledger. **Final LM012 acceptance assessment (2026-10-09):** REAL EDITOR ACCEPTANCE PASS and core CLI contract covered by implemented tests with maintainer-reported local suite PASS. The earlier additional requirement for direct `protos lint` smoke tests on both extracted release archives is recognized as **separate distribution validation, not an LM012 closure prerequisite**. The release-archive smoke tests were not performed here and no PASS is claimed for them. LM012 is eligible for closure after publication and GitHub reconciliation.

## Authority and immutable revisions

- The two default-on D194 Warning rules and LSP publication were implemented in [LM012-B1, Protos `a8027f6a78fac057f409103d8a79500faead72e2`](https://github.com/guillermomolina/protos/commit/a8027f6a78fac057f409103d8a79500faead72e2).
- The ratified D195 `protos lint` CLI/CI entry point was published in [LM012-C1, Protos `a326eb537ce2dc4dc20287300f2b6fd677e53e75`](https://github.com/guillermomolina/protos/commit/a326eb537ce2dc4dc20287300f2b6fd677e53e75).
- The [published v0.3.312 prerelease](https://github.com/guillermomolina/protos/releases/tag/v0.3.312) reports product source revision `f8ff34498f3a193c9bfe15f215a181c23504a6ad`. The current published devcontainer manifest selects Protos `0.3.312` and VS Code extension `guillermomolina.protos@0.2.2`; the maintainer independently observed `Protos 0.3.312` from the **running** container.
- The extension acceptance suite's `protos-source.lock.json` still points to the older `v0.3.237` runtime, so existing VSIX Run/Debug/Format acceptance is not proof of the new LM012-B1/C1 lint diagnostics.
- D194/#842 and D195/#843 are **already closed**, with native parent relations established; their approved contracts are unchanged. [LM012-E0 owner-selected Candidate A](LM012_E0_QUICK_FIX_SAFETY_INVESTIGATION_AND_OWNER_SELECTION.md) prohibits CodeActions / source edits for these warnings.

The source-release revision is **release metadata**, not an independently read checksum of the maintainer's running container, nor a claim that the latest Protos `main` HEAD was the revision used for the reported local tests. The owner has not supplied an exact test-run SHA.

## Human-observed real VS Code acceptance

The maintainer used the existing VS Code Remote Dev Container, opening `/workspaces/protos-devcontainer/lm012-test.protos` with:

```protos
f: () => {
    ^1
    a
    b
}
c === { x: 1 }
```

The editor's **Problems** panel showed **2** findings in the actual screenshot:

| Observed diagnostic | Visible editor position | Result |
| --- | --- | --- |
| `protos/unreachable-after-nonlocal-return`, warning for code following `^` | Ln 3, Col 5 | PASS — displayed with rule code and expected message |
| `protos/always-different-fresh-object`, warning for fresh-object `===` | Ln 6, Col 1 | PASS — displayed with rule code and expected message |

The maintainer then edited the buffer to remove both warning causes (`^1` to `1`, `c === { x: 1 }` to `c === c`). They explicitly confirmed that **both warnings disappeared**, before saving: live unsaved-buffer diagnostic replacement **PASS reported**.

Next, after appending a standalone `)` as line 7, the maintainer supplied a second actual Problems-panel screenshot: **exactly 1 parser Error** with the message `Expected a primary expression but found RPAREN` at **Ln 7, Col 1**, with no stale lint warnings. This proves the invalid-buffer parser diagnostic path and its displayed location in the tested case.

**Chronological clarification (2026-10-09):** at the time of the initial checkpoint, removal of the parser error had not yet been independently recorded. In the subsequent conversation the maintainer explicitly clarified that the full parser-error recovery check **had already been performed and passed**, including deletion of the invalid `)` and disappearance of the diagnostic. Accept this specific gate as **PASS, maintainer-reported**; do not require a repeat. No separate screenshot of the cleared parser Error was supplied, and VS Code close/reopen or a new VSIX acceptance run has not been reported. The coordinator read the prior screenshots and owner reports; it did not operate VS Code.

Original chronological live coordination and screenshots-as-user-observations are described in these GitHub updates:

- [LM012-F0 acceptance gap inventory](https://github.com/guillermomolina/protos/issues/671#issuecomment-6073457303)
- [Real VS Code two-rule Problems-panel observation](https://github.com/guillermomolina/protos/issues/671#issuecomment-6073547020)
- [Maintainer-confirmed unsaved-buffer warning clearing](https://github.com/guillermomolina/protos/issues/671#issuecomment-6073566953)
- [Observed parser error and location](https://github.com/guillermomolina/protos/issues/671#issuecomment-6073583971)

## Maintainer validation report

In the current handoff, the maintainer states:

> el git diff check esta limpio.
> Todos los tests han pasado en local

Treat this exactly as **maintainer-reported `git diff --check` CLEAN and all local tests PASS**, with no submitted test log or immutable revision binding. This statement is **not** the coordinator running tests/builds/commands; it does **not** mean that `git diff --check` was executed on the new documentation commits, and does **not** establish remote CI or packaged CLI end-to-end results.

## LM012-F acceptance and release-artifact boundary

| Gate | Status | Required completion evidence |
| --- | --- | --- |
| Real VS Code two D194 warnings | PASS observed | Two exact rule codes, severity and editor locations in Problems panel |
| Live unsaved-buffer warning clearing | PASS maintainer reported | Both warnings disappear after edits without save |
| Real parser Error on invalid buffer | PASS screenshot observed | `RPAREN` parser diagnostic, Ln 7 Col 1, no stale warnings |
| Remove parser error and observe clean editor | PASS maintainer-confirmed | Maintainer expressly clarified that deleting `)` made the parser Error disappear |
| Direct `protos lint` smoke from extracted Native `v0.3.312` release | NOT PERFORMED / NOT LM012 GATE | Additional distribution-artifact acceptance; do not infer it from editor acceptance or local tests |
| Direct `protos lint` smoke from extracted portable JVM `v0.3.312` release | NOT PERFORMED / NOT LM012 GATE | Separate distribution-artifact acceptance; do not fabricate PASS |
| LM012 formal closure | ELIGIBLE; pending GitHub closure transaction | Already published B1/C1 functional implementation, local test report and real-editor acceptance; verify exact docs publication and issue state |

Existing [CLI contract and output/exit-code documentation](https://github.com/guillermomolina/protos/blob/f8ff34498f3a193c9bfe15f215a181c23504a6ad/docs/guide/tools/cli.md) and [LSP diagnostics documentation](https://github.com/guillermomolina/protos/blob/f8ff34498f3a193c9bfe15f215a181c23504a6ad/docs/guide/tools/language-server.md) define supported behavior. Product test sources `ProtosCliLintTest` and `ProtosLanguageServerDiagnosticsTest` cover the CLI contract and protocol projection respectively; the maintainer reports the integrated local suite PASS. A separate direct smoke of each packaged archive would still add release-specific evidence, but is not necessary to claim that the LM012 implementation and editor behavior are accepted. Tests committed and local suite reports must not be misrepresented as actual checks of either extracted archive.

**Disposition:** LM012 functional/editor scope is completed and qualifies for formal GitHub closure. No defect is currently evidenced and no new implementation, semantic decision, Quick Fix, CLI change or Protos source edit is authorized. Distribution-level direct archive smoke remains **not performed** (not a blocking LM012 gate). The canonical live state of LM012/#671 is set by the subsequent GitHub closure transaction, not by this evidence snapshot; D194/#842 and D195/#843 remain closed. No project normative specification was changed by this documentation checkpoint.

```text
LM012_F_EDITOR_TWO_WARNINGS=PASS_HUMAN_SCREENSHOT
LM012_F_EDITOR_UNSAVED_LINT_CLEAR=PASS_HUMAN_REPORTED
LM012_F_EDITOR_PARSER_ERROR=PASS_HUMAN_SCREENSHOT
LM012_F_EDITOR_PARSER_RECOVERY=PASS_HUMAN_CONFIRMED
LM012_F_NATIVE_RELEASE_CLI=NOT_PERFORMED_NOT_LM012_GATE
LM012_F_PORTABLE_JVM_RELEASE_CLI=NOT_PERFORMED_NOT_LM012_GATE
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
MAINTAINER_REPORTED_GIT_DIFF_CHECK=CLEAN
TEST_RUN_COMMIT_SHA=NOT_PROVIDED
COORDINATOR_RAN_TESTS=NO
COORDINATOR_RAN_BUILDS=NO
COORDINATOR_RAN_PRODUCT_GIT_WRITES=NO
NORMATIVE_SPEC_CHANGED=NO
LM012_671=ELIGIBLE_FOR_CLOSURE
```
