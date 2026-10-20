# LM013 / 0.3.389 — D209-C document-local LSP product publication

**Recorded:** 2026-10-20.
**Product repository:** `guillermomolina/protos`.
**Published product commit:** [`cb027faa4205ada04ffe9f875e5242f19067e5e8`](https://github.com/guillermomolina/protos/commit/cb027faa4205ada04ffe9f875e5242f19067e5e8).
**Parent:** [`0bceacdfc0127453729f346382152bd9908f3de1`](https://github.com/guillermomolina/protos/commit/0bceacdfc0127453729f346382152bd9908f3de1) (independent BUG022 publication).
**Verified implementation version:** `0.3.389-SNAPSHOT` in the product `pom.xml`, with matching `CHANGELOG.md` entry.
**GitHub work:** [LM013 / #861](https://github.com/guillermomolina/protos/issues/861), still OPEN pending real-editor acceptance.
**Owner-ratified decision:** [D209-C / #936](https://github.com/guillermomolina/protos/issues/936) and [durable decision](../../decisions/tooling/D209_UNBOUND_DOCUMENT_LOCAL_LSP_INTELLIGENCE_AUTHORITY.md).
**Earlier investigation:** [LM013 / D209 source-boundary and approval evidence](LM013_D209_BOUNDARY_AND_RATIFICATION_EVIDENCE.md).
**Evidence status:** product **source publication independently verified**; local test results and diff-check **reported by the human maintainer**; real VS Code/Native editor acceptance **not yet evidenced**. This record is not an independently observed VS Code run, and does not close LM013.

## Exact publication audit

The published GitHub commit `cb027faa4205ada04ffe9f875e5242f19067e5e8` was fetched and enumerated. It changes **exactly eight files**:

| Path | Role |
|---|---|
| `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` | LSP adapter contract: remove canonical ProjectBinding requirement **only** from Hover, Completion, Signature Help |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerHoverTest.java` | Loose/alias/current buffer, proven syntax/binding, update/close and UTF-16/CRLF acceptance |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerCompletionTest.java` | Proven local parameter and reserved-expression candidates, source replacement, negative/no-guessing, UTF-16/CRLF |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerSignatureHelpTest.java` | Direct literal Closure signatures in loose/alias buffers; named/dynamic negative, changed/closed document and UTF-16/CRLF |
| `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerIntegratedEditorTest.java` | Real LSP4J/JSON-RPC transport harness, current-buffer, canonical/loose/alias, negative Definition/References, and **open document without a workspace folder** |
| `docs/guide/tools/language-server.md` | Maintained public distinction between document-local intelligence and project-authorized navigation |
| `pom.xml` | Implementation version `0.3.388-SNAPSHOT` → `0.3.389-SNAPSHOT` |
| `CHANGELOG.md` | New `0.3.389-SNAPSHOT` entry for LM013/D209-C |

The tracked commit is directly atop the previously verified BUG022 main commit; no other product files are changed in this publication.

## Implemented scope and observed source invariants

At the published revision, `textDocument/hover`, `textDocument/completion` and `textDocument/signatureHelp` consult the current exact open-document snapshot, verify the cursor position, and reject stale results or results whose snapshot URI differs from the requested URI. They no longer require the project's `navigationSourceAuthority` predicate.

`textDocument/definition` and `textDocument/references` still each have the original **initial and final** `navigationSourceAuthority` tests. The total number of `authority.test(uri)` occurrences in the published service is four, retained on these two navigation handlers. `ProtosLanguageServer` / `ProtosWorkspaceService` and the existing static analysis core are not among the changed paths. Exact ProjectBinding identity, project indexing, the Workspace Symbol facility, D110/D124 navigation proof and all language semantics are unchanged.

The static generation-one scope is unchanged: `Syntax` and `Proven binding` Hover facts; parser-accepted reserved-expression and D110-local Closure-parameter Completion; only grouped direct literal Closure Signature Help. No member, slot, module, import or named/dynamic-call inference, guest execution or implicit ProjectBinding promotion is added.

The new integrated test `standaloneDocumentWithoutWorkspaceHasLocalIntelligenceButNoNavigation` is present in published source, uses `connectWithoutWorkspace` and initializes without `rootUri` or `workspaceFolders`. It opens a `Standalone.protos` URI from the in-memory client without creating the file on disk, checks the three document-local methods, confirms empty Definition/References, and checks empty results after `didClose`. This is verified test-source coverage, **not** a claim of an independent real VS Code observation.

## Validation evidence and provenance

The human maintainer explicitly reported:

- `git diff --check`: **clean** on the local publication candidate.
- **All local tests: PASS**, following the agreed focal Maven suite and integrated `make test` validation sequence.
- `LM013: enable proven document-local LSP intelligence (0.3.389)` **pushed** to `guillermomolina/protos`.

The remote GitHub commit and its exact eight-file change set were independently confirmed by the coordinating agent. The chat report contains no preserved raw Maven/JUnit output, native-run artifact hash or exact test counts, so none are invented here. The tests were run by the maintainer, not by the coordinating agent; `make test`'s intended Java/Native coverage does not substitute for a human-observed editor session.

### Remaining acceptance before LM013 closure

A maintainer-observed **real VS Code session** using the actual newly built/published Protos LSP (record runtime and extension identities) must compare:

1. A canonical bound project: Hover, Completion and Signature Help remain correct; Definition/References continue working when exact proof exists.
2. A genuinely loose `.protos` file, including a workspace-free session: local proven Hover/Completion/direct literal Signature Help work with the current unsaved buffer.
3. Conservative negatives: calls through a name or receiver do not get speculative signature help; loose/alias Definition and References remain empty, and closing a document invalidates local results.

Record the actual observed identities, results and any exceptions before marking LM013 complete. No further product slice or new Issue is implied by this acceptance checkpoint alone.

```text
LM013_IMPLEMENTATION_COMMIT=cb027faa4205ada04ffe9f875e5242f19067e5e8
LM013_VERSION=0.3.389-SNAPSHOT
PRODUCT_PUBLICATION=VERIFIED_GITHUB
CHANGED_PRODUCT_PATHS=8
HUMAN_REPORTED_DIFF_CHECK=PASS
HUMAN_REPORTED_LOCAL_TESTS=PASS
AUTOMATED_TESTS_RUN_BY_COORDINATOR=NO
REAL_VSCODE_ACCEPTANCE=PENDING
LM013_ISSUE=CANNOT_CLOSE_YET
```
