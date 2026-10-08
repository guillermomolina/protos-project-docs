# LM010-B — published static completion; LM010-C0 exact-call signature-help handoff

**Date:** 2026-10-08. **Issue authority:** [LM010 / guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493). **Document role:** revision-specific, non-normative publication evidence and bounded research handoff. Historical investigation [LM010-B0](LM010_B0_COMPLETION_AUTHORITY_AUDIT_AND_B_HANDOFF.md) remains retained, not retroactively altered.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_SHA=5461a54b0204d266218ffb5e357c103794e340c1
PRODUCT_COMMIT_SUBJECT=LM010-B: static completion in the Protos language server
PRODUCT_VERSION=0.3.296-SNAPSHOT
LM010_B=IMPLEMENTED_PUSHED
MAINTAINER_TESTS=ALL_PASS_REPORTED
MAINTAINER_DIFF_CHECK=CLEAN_REPORTED
COORDINATING_AGENT_RAN_PRODUCT_TESTS=NO
LM010_PARENT=OPEN
NEXT_SLICE=LM010-C0
NEXT_TYPE=INVESTIGATION
NEXT_IMPLEMENTATION_REPOSITORY=NOT_APPLICABLE
```

## Product evidence read back from published commit

The exact [product commit](https://github.com/guillermomolina/protos/commit/5461a54b0204d266218ffb5e357c103794e340c1) was independently read and observed as `main` at this checkpoint. The commit modifies/adds:

- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticCompletion.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticCompletionResult.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitions.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticHover.java`
- `src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java`
- `src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java`
- `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticCompletionTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerCompletionTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFoundationTest.java`
- `pom.xml`, root `CHANGELOG.md`.

The published changelog identifies `0.3.296-SNAPSHOT` and describes:
- Conservative completion for exact D110 generation-one same-activation Closure parameter origins at parser-proven read sites, plus parser-accepted expression-position `this`, `context`, `true`, `false`, `null`. No `super` bare expression, dynamic members, body slots, captured/unproven names, prelude or workspace guessing.
- Shared D110 proof walk reused by Definition, References, Hover and Completion without intended change to earlier semantics.
- Cursor context using lexer/trivia, full-buffer parse for typed prefixes and one bounded synthetic-name insertion for empty valid read sites. Unclassifiable source yields empty candidates.
- Standard `completionProvider` with no resolve or triggers. Completion lists are complete (`isIncomplete=false`), under canonical source URI, exact current open snapshot and UTF-16 checks.
- The additional lexical-error hardening requested during implementation: `ProtosLexer.LexicalError` exposes a structured offset and `ProtosStaticAnalysisCore.parse` projects the error into an exact empty-span failure so that `didOpen`/`didChange` can report a diagnostic rather than throwing for an ordinary lexical mistake.

**Validation provenance:** the project owner explicitly reported that the commit was pushed, `git diff --check` was clean, and **all local tests passed**. The coordinator did not execute the tests, read a full local build transcript, or run product Git operations. The existence and claimed coverage of tests can be verified in the product commit; only the maintainer attests the run result. The product specification was not changed by LM010-B.

## Next: LM010-C0 — small *read-only* signature-help proof and cursor audit

The remaining user-visible feature is `textDocument/signatureHelp`. The uncertainty is materially new, not a repetition of Completion:

1. **Callee identity is not proven by a name.** D110 generation one establishes same-activation Closure *parameter origins* for reads, not what runtime value a variable/parameter/member currently contains. A source declaration with a literal Closure assigned to a slot does not establish that the slot still contains that Closure at a later call. A dynamically bound call could receive a different callable value or fail.
2. **Some callee expressions can be exact from syntax alone.** Grammar `PROTOS_GRAMMAR.md` explicitly permits direct invocation of grouped Closure literals, e.g. `(x => x * 2)(10)`; AST `SurfaceCall.receiver` can be a `SurfaceGroup` of `SurfaceClosure` and `SurfaceClosure.parameters` stores the exact declaration (default/rest).
3. **Signature help is requested before calls close.** The parser consumes complete `argument-list` including `)`, with comma-separated expressions, nested calls, layouts, `...spread` items and optional trailing Closure. A correct argument index must not be estimated by raw comma counts, assumed one-spread-item-per-argument, or obtained by truncating a nested expression; investigate how existing parser/lexer can prove the active invocation and argument before offering an LSP signature.

The **only** next task is a bounded read-only feasibility matrix, **not** a fresh broad design investigation: inspect the current product HEAD, AST `SurfaceCall`, `SurfaceClosure`, `SurfaceGroup`, `SurfaceParameter`, `SurfaceArgument`, `SurfaceMember`, `SurfaceSuperSend`; parser call/argument/Closure rules; lexer/trivia and LM010-B cursor classification; current snapshot/URI/freshness/UTF-16 LSP pattern; normative `spec/PROTOS_GRAMMAR.md` and `spec/semantics/CALLABLES.md`.

Specifically answer:
- Which **syntactic direct-Closure call** forms can have a signature proven with no runtime evaluation, and when parentheses or other expressions break exactness?
- Can any indirect name/member call be soundly identified by an *existing* source-value authority, not by guessed textual source ancestry? Negative examples should include mutation/removal, default/rest and receiver delegation.
- How to determine the innermost active call, argument number and partial-source positioning in `f(|`, `f(a, |)`, nested calls, comma-containing nested data, multline/CRLF/non-BMP source, malformed/closed/unowned/stale buffers, argument spread and trailing closure?
- What can the LSP4J version in current `pom.xml` support for `SignatureHelpOptions`, `SignatureHelpParams`, `SignatureInformation`, `ParameterInformation`, and `activeSignature`/`activeParameter`? In particular do not mistake a declared parameter index for a known runtime vector offset after unknown spread.
- Whether a tiny direct-literal-first implementation is sufficiently correct and useful to advertise; recommend one coherent `LM010-C` implementation (Java projection, handler, capability, tests) or precisely identify an unapproved public-contract Dxxx checkpoint. No silent generalization to named callables.

The research should be based on **read-only remote HEAD and spec**, produce concrete counterexamples and a small exact support matrix, and be completed in one response. Do not issue shell commands, tests, builds or Git changes; do not create new work IDs, ratify choices, or modify either repository as part of C0. After C0, **avoid a further research micro-slice** unless the current implementation authority exposes an actual substantive decision.

## Governance and future work

- No independent Issue is justified by a one-step bounded feasibility audit: track under [#493](https://github.com/guillermomolina/protos/issues/493).
- Next *implementation* would run in `guillermomolina/protos`, assume current local HEAD and be autonomous from other repos/web. Agent edits; human performs builds/tests and state-changing Git/publication. Version and changelog updated only after tests green and immediately before human commit.
- LM010-D editor smoke/acceptance and parent closure follow **actual** Hover/Completion/Signature Help functionality. LM010 is not a dependency of LM009.
- This evidence record is not ratification of a new public signature contract or execution proof.

```text
LM010_B_CLOSED_AS_SLICE=YES
LM010_B_TEST_RESULT=HUMAN_REPORTED_PASS
LM010_C0=RESEARCH_ONLY
LM010_C_IMPLEMENTATION_STARTED=NO
LM010_PARENT_CLOSED=NO
NEW_ISSUE=NO
```
