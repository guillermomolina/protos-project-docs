# LM013 and BUG023 — final real VS Code Native LSP acceptance

**Evidence date:** 2026-10-21.
**Owner:** [LM013 #861](https://github.com/guillermomolina/protos/issues/861) (CLOSED, Completed).
**Related independently closed defect:** [BUG023 #937](https://github.com/guillermomolina/protos/issues/937) (CLOSED, Completed).
**Evidence class:** maintainer-reported real VS Code observations, maintainer-reported local test and Git diff checks, and independently fetched GitHub product commits/Issue outcomes. No raw suite output or fresh binary/PID attestation accompanied this final confirmation.

## Exact published identities

- **LM013 feature implementation:** [`guillermomolina/protos@cb027faa4205ada04ffe9f875e5242f19067e5e8`](https://github.com/guillermomolina/protos/commit/cb027faa4205ada04ffe9f875e5242f19067e5e8), `0.3.389-SNAPSHOT`; D209-C document-local Hover, Completion and direct-literal Closure Signature Help in open unbound buffers, while Definition/References/Workspace Symbols stay constrained by exact canonical ProjectBinding.
- **BUG023 Native fix:** [`guillermomolina/protos@d7e48d86698c08b0f74e176080fd2b3b4526c0cd`](https://github.com/guillermomolina/protos/commit/d7e48d86698c08b0f74e176080fd2b3b4526c0cd), `0.3.390-SNAPSHOT`; restores ICU4J normalization data `icudata/nfc.nrm` to Native reachability metadata, retaining previous ICU `.icu` resources and LSP4J reflection settings. The fix commit also added `build/native/test-lsp-workspace-symbols.py` and integrated it into `build/native/test-native.sh`.
- **Prior standalone editor checkpoint:** [`LM013_389_REAL_VSCODE_LOOSE_DOCUMENT_ACCEPTANCE_2026_10_20.md`](LM013_389_REAL_VSCODE_LOOSE_DOCUMENT_ACCEPTANCE_2026_10_20.md), published at [project-docs `41967cada42569824ed9f235fba39d38df2107dd`](https://github.com/guillermomolina/protos-project-docs/commit/41967cada42569824ed9f235fba39d38df2107dd). It preserves the exact previously observed Native 0.3.389 PID/command line and partial editor results as an **historical checkpoint**, not the final project state.
- **Release:** no public release was required or performed to carry out this local Native Image acceptance.

## Reproduction history and final human-observed matrix

The first loose-file acceptance attempt started the older published 0.3.312 Native LSP by mistake. After correcting the VS Code Remote Machine `protos.runtime.executable` setting and reloading, the maintainer verified the separate unpublished 0.3.389 Native LSP process through `/proc/<pid>/cmdline`; document-local Hover, Completion and literal Closure Signature Help passed. A later nonempty `workspace/symbol` query exposed a distinct BUG023 missing ICU4J NFC resource exception, causing LSP JSON-RPC `-32603` and Native process exit 70.

BUG023 was independently repaired, tested and closed at `0.3.390-SNAPSHOT`. The maintainer then followed the prescribed retest with the corrected Native distribution and a real VS Code workspace containing the canonical fixture root `/home/vscode/lm013-bound`, and replied **"si todo funciona"** to the full stated acceptance matrix.

| Editor operation | Expected result | Evidence |
|---|---|---|
| Loose/unbound Hover over Closure-body `value` | Proven binding | **PASS**, maintainer reported on Native 0.3.389 |
| Loose/unbound `val` + `Ctrl+Space` | Proposed Closure parameter `value` with proof detail | **PASS**, maintainer reported on Native 0.3.389 |
| Loose/unbound direct literal Closure Signature Help | Direct literal parameter signature | **PASS**, maintainer reported on Native 0.3.389 |
| Loose/unbound Go to Definition | No unauthorized resolution | **PASS**, maintainer reported on Native 0.3.389 |
| Bound `lm013-bound/Main.protos`, `Ctrl+T` search `f` | Workspace slot symbol returned; no ICU exception | **PASS**, maintainer reported on corrected Native 0.3.390 |
| Bound `Main.protos`, `F12` on Closure-body `value` | Navigate to parameter declaration | **PASS**, maintainer reported on corrected Native 0.3.390 |
| Bound `Main.protos`, `Shift+F12` on `value` | Two references in body | **PASS**, maintainer reported on corrected Native 0.3.390 |
| Bound Hover on `value` | Proven binding | **PASS**, maintainer reported on corrected Native 0.3.390 |
| Bound `val` + `Ctrl+Space` | Proven `value` completion | **PASS**, maintainer reported on corrected Native 0.3.390 |

The original negative F12/Shift+F12 attempts cannot be treated as an implementation regression: the fixture appeared beneath the `protos-devcontainer` workspace instead of being admitted as the exact root. The final reported retest used the corrected project-root procedure.

This final conversation did not separately provide a fresh `/proc` command line, runtime `SOURCE.txt` metadata, exact installed extension build, screenshots of individual final editor actions, or raw output for an explicit no-workspace VS Code window. The LSP JSON-RPC no-workspace and Unicode behavior is covered by the published automated tests and BUG023 closure evidence. **Do not extrapolate the report beyond the named checks.**

## Maintainer local validation and publication status

On 2026-10-21 the maintainer explicitly reported:

> el git diff check esta limpio. Todos los tests han pasado en local.

Record this as **HUMAN_REPORTED_PASS** for local `git diff --check` and all tests in the maintainer's local workflow. The user did not supply the command transcript, suite count, fresh local HEAD printout, or separate result for the documentation repository. This evidence therefore does **not** claim the coordinating assistant executed builds/tests or independently established exact test counts. BUG023's closure already records its own separately stated targeted JVM/Native acceptance, Native `std:regex` regression, and integrated Native validation.

GitHub confirms both Issues have their completed closure states and the product `main` at `d7e48d86698c08b0f74e176080fd2b3b4526c0cd` when this record was prepared. No new product code change, specification change, release, branch, or tag was necessary for this evidence publication.

## Outcome and boundaries

**LM013 accepted and closed. BUG023 fixed and closed.** The decisive resolution is the sequence of separate product commits and the maintained LSP behavior: document-local intelligence works on open loose Protos sources without widening project authority; proven Definition and References remain available for canonical bound sources; and Native workspace symbol lookup no longer crashes on ICU4J NFC data.

This is an evidence record, **not** a new ratification or a source of Protos semantics. The historical 0.3.389 loose-file and publication checkpoints remain immutable and accurate as of their dates; they are superseded as *current-state* summaries only by this final record and the live closed GitHub Issues.

```text
LM013_PRODUCT_SHA=cb027faa4205ada04ffe9f875e5242f19067e5e8
BUG023_PRODUCT_SHA=d7e48d86698c08b0f74e176080fd2b3b4526c0cd
BUG023_NATIVE_VERSION=0.3.390-SNAPSHOT
LM013_ISSUE=861_CLOSED_COMPLETED
BUG023_ISSUE=937_CLOSED_COMPLETED
LOOSE_DOCUMENT_EDITOR_ACCEPTANCE=HUMAN_PASS
BOUND_PROJECT_EDITOR_ACCEPTANCE=HUMAN_PASS
LOCAL_ALL_TESTS=HUMAN_REPORTED_PASS
PRODUCT_GIT_DIFF_CHECK=HUMAN_REPORTED_PASS
RAW_LOCAL_TEST_LOGS=NOT_PROVIDED
RELEASE_PUBLISHED=NO
```
