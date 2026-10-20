# LM013 / D209 — Local LSP authority investigation and Candidate C ratification evidence

**Recorded:** 2026-10-20.
**Product repository:** `guillermomolina/protos`.
**Current product main independently read:** [0bceacdfc0127453729f346382152bd9908f3de1](https://github.com/guillermomolina/protos/commit/0bceacdfc0127453729f346382152bd9908f3de1) (`BUG022: fix TOOL010 document-boundary comments`), following initial research revision [d32524bd1dec045966dc33f3a3f58abc4dcf0278](https://github.com/guillermomolina/protos/commit/d32524bd1dec045966dc33f3a3f58abc4dcf0278).
**Issue owner:** [LM013/#861](https://github.com/guillermomolina/protos/issues/861); decision [D209/#936](https://github.com/guillermomolina/protos/issues/936).
**Decision:** C, exact project-owner response `Apruebo C`, 2026-10-20, after the presented D209 comparison. [Recorded approval and GITHUB021 check](https://github.com/guillermomolina/protos/issues/936#issuecomment-6289579502).
**Canonical decision:** [D209 record](../../decisions/tooling/D209_UNBOUND_DOCUMENT_LOCAL_LSP_INTELLIGENCE_AUTHORITY.md).
**Evidence class:** source-/Issue-backed design and approval; **NOT** an implementation, build, automated test, Native release or observed VS Code result.

## Verified physical source path and why loose files are empty today

- `src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java`: constructs `ProtosWorkspaceService` and installs `workspaceService::ownsCanonicalSourceUri` into `ProtosTextDocumentService.setNavigationSourceAuthority`.
- `src/main/java/com/guillermomolina/protos/lsp/ProtosWorkspaceService.java`: `ownsCanonicalSourceUri` iterates only current validated ProjectBinding sources, uses literal URI equality and succeeds only on exactly one match. Candidate workspace roots are exact acquisition candidates, never inferred project identity.
- `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java`: definition, references, hover, completion and signature help each reject an unauthorized URI **before** the analysis and recheck `authority.test(uri)` before returning the result. It already stores `didOpen`/`didChange` snapshots in `OPEN_DOCUMENTS_DOMAIN` and removes them on `didClose`. Diagnostics, formatting and document symbols do not use the navigation authority predicate.
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java`: Hover derives parser syntax facts and D110-local proof; Completion validates current source/cursor and parser-accepted reserved expressions plus D110-proven Closure parameter names; Signature Help validates **literal** Closure calls with safe, temporary incomplete-source repair. The core takes a `ProtosDocumentSnapshot`; it does not itself acquire projects or execute Protos.
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java`: snapshot custody is identified by opaque workspace/document IDs; `isCurrent` checks the exact captured immutable snapshot. `putDocument` replacement order, not numeric-version monotonicity, establishes current content.
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticHover.java`, `ProtosStaticCompletion.java` and `ProtosStaticSignatureHelp.java` already distinguish parser-proven syntax, complete local proof and dynamic/unknown failure.

## Baseline tests reviewed

- `ProtosLanguageServerHoverTest.java`: `looseOrNonCanonicalOpenDocumentNeverGainsHoverAuthority`.
- `ProtosLanguageServerCompletionTest.java`: `looseOrNonCanonicalOpenDocumentNeverGainsCompletionAuthority`.
- `ProtosLanguageServerSignatureHelpTest.java`: `looseOrNonCanonicalOpenDocumentNeverGainsSignatureAuthority`.
- `ProtosLanguageServerDefinitionTest.java`: `looseOrNonCanonicalOpenDocumentNeverGainsDefinitionAuthority`.
- `ProtosLanguageServerReferencesTest.java`: `looseOrNonCanonicalOpenDocumentNeverGainsReferenceAuthority`.
- `ProtosLanguageServerIntegratedEditorTest.java`: `unsavedSnapshotsAndDocumentAuthorityGovernEveryLm010Answer` explicitly asserts empty loose/alias Hover/Signature Help/Completion over the standard LSP transport.

The existing unit and integrated assertions are **baseline source facts**, not proof that revised Candidate C is working. Change the first three and integrated LM010 checks to expected local positives plus conservative negatives; retain the definition/references negative checks unchanged. Additional real-editor acceptance remains human executed.

## Historical end-to-end comparator

LM010-D acceptance was reported by the maintainer on 2026-10-09 for **a canonical bound project** carrying exact `protos.toml`, `protos.lock`, `protos.project`, not for bundled loose examples. Native 0.3.312, official VS Code extension 0.2.2 and pinned Devcontainer 5f5993c1b9eecb74a7b9fe5a71d842bc3e4a057f were the actual comparator identities. This explains a genuine UX gap but does **not** establish Podman transport or runtime regression. The independent BUG022 formatter problem is already resolved in a separate product publication and must not be folded into LM013. [Exact LM010-D evidence](../LM010/LM010_D_REAL_VSCODE_ACCEPTANCE_AND_CLOSURE.md).

## Ratified invariants and counterexamples

D085/F′ and D089/A″ retain canonical binding/source membership. D110/B′ and D124/B′ retain definition and references proof boundaries. The explicit owner approval changes only the historical rule that an unbound source can receive **no** LM010 intelligence, allowing `Hover`, `Completion` and `Signature Help` proven from its independently open snapshot; it does not promote the document into the project source inventory. No language semantics, runtime behavior or LSP feature advertisements beyond existing LM010 need change.

Negative matrix for the implementation:

- a call through `f` that stores a Closure: **no signature** (not a direct literal);
- a same-spelling member `receiver.foo`, import, arbitrary `foo` slot or unproved binding: **no inferred binding/completion/definition**;
- unopened/closed document: **no local result**, even when disk contents exist;
- `file:` alias of a canonical file: local analysis of the **exact independently opened URI** may succeed, but definition/references **must remain unauthorized**;
- incorrect/out-of-range UTF-16 cursor, interrupted lexical token or unrelated invalid syntax: existing fail-closed behavior;
- stale results from an earlier `didChange`: **not publishable**.

Positive matrix: syntax-only literal Hover; unique local Closure parameter Hover and Completion; parser-accepted reserved expressions; direct grouped literal-Closure Signature Help and active argument tracking; unsaved `didChange` replacement; independent loose and bound cases.

## Publication/validation boundary

The coordinating agent did not run builds, tests, source programs or a real VS Code UI and did not change `guillermomolina/protos` source. The project owner authorized a bounded direct commit/push of decision/evidence/handoff **only** in `guillermomolina/protos-project-docs`. No product version or release is claimed. The product agent must read actual current local HEAD, check its changed paths, edit code and tests under LM013, and hand all validation, version/CHANGELOG finalization and product Git publication to the human executor.

```text
LM013_RESEARCH=COMPLETE
D209_OWNER_APPROVAL=EXACT_C
D209_INVARIANT_DELTA=PASS
DOCS_DECISION_PUBLICATION=THIS_TASK
PRODUCT_SOURCE_CHANGE=NONE_BY_COORDINATOR
AUTOMATED_TESTS=NOT_RUN
REAL_VSCODE_ACCEPTANCE=NOT_RUN
LM013_CLOSURE=PENDING_IMPLEMENTATION_AND_HUMAN_ACCEPTANCE
```
