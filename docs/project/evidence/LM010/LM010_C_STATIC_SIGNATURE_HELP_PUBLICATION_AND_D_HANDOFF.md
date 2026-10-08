# LM010-C — published static Signature Help; LM010-D integration/closure handoff

**Date:** 2026-10-08. **Owner:** [LM010 / guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493). **Role:** immutable, revision-specific, non-normative implementation and validation evidence plus the next-slice boundary.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_SHA=129d41785bbcad9d25da03abbcdf0a7f0a5fcf8a
PRODUCT_COMMIT_SUBJECT=LM010-C: static signature help for literal Closure calls (#493)
PRODUCT_VERSION=0.3.304-SNAPSHOT
LM010_A=IMPLEMENTED_PUSHED
LM010_B=IMPLEMENTED_PUSHED
LM010_C=IMPLEMENTED_PUSHED
MAINTAINER_LOCAL_TESTS=ALL_PASS_REPORTED
MAINTAINER_DIFF_CHECK=CLEAN_REPORTED
COORDINATOR_RAN_PRODUCT_TESTS=NO
COORDINATOR_RAN_PRODUCT_BUILDS=NO
SPECIFICATION_CHANGED_BY_LM010_C=NO
LM010_PARENT=OPEN
NEXT_SLICE=LM010-D
NEXT_WORK=INTEGRATED_EDITOR_ACCEPTANCE_AND_CLOSURE
NEW_FORMAL_ISSUE_REQUIRED=NO
```

## Published product revision and scope

The published [exact Protos commit](https://github.com/guillermomolina/protos/commit/129d41785bbcad9d25da03abbcdf0a7f0a5fcf8a) was read back from GitHub `main`. The commit introduces the source-neutral `ProtosStaticSignatureHelpResult`, bounded `ProtosStaticSignatureHelp` analyzer, Core and Session query/freshness wiring, LSP signature capability and `textDocument/signatureHelp` handler, and new static/LSP tests. The shared Completion guards become package-visible for reuse. `pom.xml` and `CHANGELOG.md` were updated in the same product publication. The changelog describes a conservative literal-Closure-only capability rather than inferred types, dynamic callable shapes or workspace indexing.

The current LSP capability advertises `(` and `,` as signature-help triggers. The handler emits at most one source-derived signature with explicit source parameter labels and `activeSignature=0`. It returns no signature when callable identity, active call/argument, source authority, document freshness, or UTF-16 position cannot be proved. Spreads and over-arity without a rest parameter do not invent a highlighted parameter. Empty parameter lists may show an unhighlighted signature.

The analyzer proves only `SurfaceCall` receivers that reduce through `SurfaceGroup` to a literal `SurfaceClosure`. It does not infer a callable from slot declarations, names, members, results of calls, dynamic receiver dispatch, or runtime values. It counts own-level commas from the real lexer and admits narrowly repaired incomplete source only when the tokens and the canonical parser verify the same call/argument identity. Conservative exclusions (nested array/object cursor, comments, or unfinished mid-file lists that cannot be safely closed) remain intentional.

The exact commit's changed paths include:

- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticSignatureHelp.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticSignatureHelpResult.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticCompletion.java`
- `src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java`
- `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticSignatureHelpTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerSignatureHelpTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFoundationTest.java`
- `pom.xml` and `CHANGELOG.md`.

The preceding LM010-A Hover and LM010-B Completion publications retain their own exact historical evidence: [A closure](LM010_A_IMPLEMENTATION_CLOSURE_AND_LM010_B0_HANDOFF.md) and [B closure](LM010_B_IMPLEMENTATION_CLOSURE_AND_C0_SIGNATURE_HANDOFF.md).

## Validation and evidence provenance

The maintainer explicitly reported **all local tests PASS**, a clean **`git diff --check`**, and **LM010-C pushed**. These are human-reported execution claims, not independent execution by the coordinating agent. The published product commit, changed paths, tests and current Maven version were independently observed via read-only GitHub access. No full test transcript, external editor screenshot or real VS Code manual acceptance is claimed. No changes to the normative `spec/` tree are included in the LM010-C commit, and no new Dxxx/PLATxxx decision is represented.

The associated instruction sources were checked in the product repository (`AGENTS.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/IMPLEMENTATION.md`); the durable blocker ledger remains authoritative for genuinely unresolved normative implementation blockers, but no new blocker was identified by this bounded publication reconciliation.

## LM010-D — integrated editor acceptance and honest final closure

LM010/#493 remains **open**. Next, verify that the **published** Hover, Completion and Signature Help actually function through the standard LSP transport and a real editor, not merely in direct Java unit tests. The available `ProtosLanguageServerFoundationTest` already covers real LSP Content-Length framing; feature-specific server tests cover canonical URI/snapshot/UTF-16 and negative responses. LM010-D should first inspect the current product HEAD and existing acceptance machinery, reuse it, and avoid writing another harness if no coverage is missing.

A bounded integrated acceptance matrix must include startup/initialize capabilities; a canonical project open document; positive Hover and Completion; direct literal Closure Signature Help on `(` and `,`; dynamic/non-proven callee `null`; incomplete source; unsaved edits and close; bad/noncanonical URI, cursor and stale snapshot; CRLF/non-BMP where relevant; diagnostics coexistence; and no regression in navigation, formatting and editor responsiveness. Crucially, a successful real-editor acceptance requires **maintainer-observed actual editor behavior**. In-repository protocol tests alone do not prove VS Code integration. Do not conflate LM010 with LM009 extension publication or introduce TypeScript-side semantic authority.

The next slice is a single bounded LM010-D acceptance/reconciliation effort under **#493**, not a new independent formal issue. A product-code change, if and only if the acceptance audit demonstrates a real missing product-owned transport/regression path, is implemented in `guillermomolina/protos` in human-executor mode; the agent never runs builds/tests or product Git publication. No new language semantics or substantive public contract is authorized. If the existing real-editor acceptance already passes and no product edits are required, record the evidence and close #493 under the formal closure gate without manufacturing a commit or changing `pom.xml`/`CHANGELOG.md`. Otherwise keep the issue open with the exact failed gate and perform only the necessary bounded correction and human validation.

```text
LM010_C_CLOSED_AS_PUBLISHED_SLICE=YES
LM010_D_REAL_EDITOR_ACCEPTANCE=NOT_YET_REPORTED
LM010_D_PRODUCT_PATCH=ONLY_IF_REQUIRED_BY_EVIDENCE
LM010_PARENT_CLOSED=NO
```
