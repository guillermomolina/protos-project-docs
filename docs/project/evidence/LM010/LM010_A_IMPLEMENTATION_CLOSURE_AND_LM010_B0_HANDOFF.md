# LM010-A — published static Hover implementation and LM010-B0 completion handoff

**Date:** 2026-10-08. **Live owner:** [LM010 / guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493). **Role:** revision-bound, non-normative implementation/evidence checkpoint and next-slice technical handoff. This record is not a normative Protos language contract or an independently observed test log.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_COMMIT=15ac29dbc3bcc1779fb0b15235cfda64f16ecee5
PRODUCT_COMMIT_SUBJECT=LM010-A: static hover in the Protos language server
PRODUCT_VERSION=0.3.293-SNAPSHOT
IMPLEMENTATION_SLICE=LM010-A
IMPLEMENTATION_STATUS=PUSHED
MAINTAINER_LOCAL_TEST_REPORT=ALL_PASS
MAINTAINER_GIT_DIFF_CHECK_REPORT=CLEAN
AGENT_RAN_PRODUCT_TESTS=NO
AGENT_RAN_PRODUCT_GIT=NO
PARENT_ISSUE=493
PARENT_STATUS=OPEN
NEXT_SLICE=LM010-B0
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_IMPLEMENTATION_REPOSITORY=NOT_APPLICABLE
```

## Publication evidence

The exact [product commit](https://github.com/guillermomolina/protos/commit/15ac29dbc3bcc1779fb0b15235cfda64f16ecee5) is the observed `main` HEAD at this checkpoint. It adds or changes:

- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java`
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticHover.java` (new)
- `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticHoverResult.java` (new)
- `src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java`
- `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java`
- `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticHoverTest.java` (new)
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFoundationTest.java`
- `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerHoverTest.java` (new)
- `pom.xml`, `CHANGELOG.md`.

The published changelog records `0.3.293-SNAPSHOT`. The Java language server now advertises `hoverProvider`. The editor-neutral on-demand projection first consumes **D110 generation-one exact same-activation Closure parameter** proofs and labels the resulting information as `Proven binding`. Otherwise it may provide explicitly labelled `Syntax` facts for Closure parameter declarations/heads and parser literal kinds. Unknown or inadmissible positions yield null; the LSP service validates exact canonical ProjectBinding source ownership, immutable current open-buffer snapshots and post-query freshness, with source ranges mapped to UTF-16.

The source test additions include isolated analyzer and LSP-provider coverage for labels, proof boundaries, parameters/defaults/rest, literals, canonical URI, stale and unsaved documents, invalid requests, CRLF and non-BMP source positions. The changelog records no specification change. This checkpoint **does not assert external VS Code end-to-end manual validation** and does not reopen D110 or D124.

## Validation provenance and limitations

**Maintainer's explicit report** in the handoff message:

- "LM010-A: static hover in the Protos language server, pushed."
- "`git diff --check` está limpio."
- "Todos los tests han pasado en local."

These are user-reported outcomes; the coordinating agent did not independently execute a test, build, `git diff --check` or a product Git publication command. Commit contents and revision have been independently inspected through read-only GitHub access. Test-case presence is independently visible in the repository, but individual test execution logs were not retrieved in this task. This distinction must not be flattened into false CI/log claims.

Prior [LM010-A0 technical scope](LM010_A0_HOVER_AUDIT_AND_LM010_A_IMPLEMENTATION_SCOPE.md) remains historically accurate. This record supersedes only its *pending implementation* state, not its observations or owner-scoped feature boundary.

## Next work: LM010-B0 — completion authority and incomplete-source feasibility

The **next unresolved capability is completion** (not Hover rework). An additional bounded, read-only investigation is justified by two concrete, new implementation risks, not by ceremony:

1. The current `ProtosStaticAnalysisCore.parse` requires `ProtosParser.parseProgram()` success. Interactive completion routinely occurs with partial identifier prefixes and grammatically incomplete expressions. Establish whether existing `ProtosLexer`, `TokenOccurrence`, `TokenCursor`, `ProtosParserSourceFacts` or equivalent opt-in tooling primitives can supply **exact cursor-aware context without altering runtime grammar or adding speculative recovery machinery**.
2. Protos local execution-context slots are mutable and removable; lookup may continue through lexical parents and receiver/delegation; opaque calls invalidate D110's proven-origin facts. Therefore a name appearing in a visible source region is not automatically an available runtime binding. Distinguish parser-proven declaration *suggestions* from guaranteed semantic availability. The existing `ProtosStaticHover` syntax/proof distinction may provide the presentation precedent, but it does **not** itself authorize completion behavior.

The read-only investigation must produce: exact source/decision authority; a small real cursor-context/candidate matrix (at least Closure parameters, nested/defaulted parameters, ordinary slot creation, member suffix, intrinsics/keywords and unknown source); incomplete-prefix behavior; source-range/UTF-16 edit rules; malformed/closed/unowned/stale snapshot behavior; negative examples for opaque calls, mutable contexts and dynamic members; cost/architecture path; and a recommended bounded next implementation **or an explicit substantive Dxxx gate** if its public contract is not already determined. Do not use pre-D131 match binders/aliases, infer source symbols as runtime members, or create an index/guest runtime.

The product's LSP4J `1.0.0` server presently exposes Hover, Definition, References, Symbols and Formatting, but does not advertise completion. Verify the **then-current** repository HEAD; do not assume `15ac29d` remains latest.

LM010-B0 is an **investigation, not a code implementation**: no local commands, tests, builds, environment/Git operations, source edits, new Dxxx/PLATxxx ratification, Issue allocation, or speculative completion capability. It should be deliberately narrow and finish with a yes/no conclusion on the real blocker. If the risk is merely mechanical, the next implementation can combine the bounded Java analysis model, standard LSP handler, capability and tests in one slice. No recurring research loops.

Later phases remain LM010-C signature help and LM010-D integrated editor acceptance/closure, subject to their independently verified preconditions. LM010 does not block LM009.

## Authority and revision record

- Product initial audit HEAD: `3bb1278d91ee5cea98031462be2a5c4dd3c89019`.
- Product implementation commit: `15ac29dbc3bcc1779fb0b15235cfda64f16ecee5`.
- Issues and design gates: [LM010 #493](https://github.com/guillermomolina/protos/issues/493); D110, D124, D131, PLAT024 remain governed by their existing records.
- Working discipline: `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/DESIGN.md`, scoped source/spec instructions.
- Product source inspected for this checkpoint: published commit file list; `ProtosStaticAnalysisCore.java`, `ProtosStaticHover.java`, `ProtosStaticHoverResult.java`, `ProtosStaticAnalysisSession.java`, `ProtosLanguageServer.java`, `ProtosTextDocumentService.java`, `ProtosParser.java`, `ProtosLexer.java`, `ProtosLspSourcePositions.java`, Hover tests; targeted normative `CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `PROTOS_GRAMMAR.md`.

```text
LM010_A_STATUS=IMPLEMENTED_AND_PUSHED
LM010_A_EVIDENCE=HUMAN_REPORTED_GREEN
LM010_B0_IMPLEMENTATION=NOT_AUTHORIZED
LM010_PARENT_CLOSED=NO
NEW_FORMAL_ISSUE_ALLOCATED=NO
```
