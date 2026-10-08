# LM010-A0 — static Hover audit closure and bounded LM010-A implementation scope

**Recorded:** 2026-10-08. **Owner:** [LM010 / protos#493](https://github.com/guillermomolina/protos/issues/493). **Role:** non-normative, revision-bound technical evidence and owner-selected feature acceptance scope. This is neither executable validation nor a new Protos language specification.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_HEAD_INSPECTED=3bb1278d91ee5cea98031462be2a5c4dd3c89019
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
LM010_A0=INVESTIGATION_COMPLETE
NEXT_SLICE=LM010-A
NEXT_TYPE=IMPLEMENTATION
NEXT_EXECUTION_REPOSITORY=guillermomolina/protos
PRODUCT_CHANGES_FROM_A0=NONE
PRODUCT_TESTS_FROM_A0=NOT_RUN
DESIGN_OR_SPEC_CHANGES_FROM_A0=NONE
LM010_PARENT_STATUS=OPEN
```

## Outcome and authority

The read-only LM010-A0 source audit established a **bounded mechanical implementation path** for standard LSP `textDocument/hover`. The user explicitly chose to **skip a redundant LM010-A1 investigation** and proceed to one LM010-A implementation slice, with the following exact presentation boundary:

1. Expose only **proven semantic facts** or **parser-verifiable syntactic facts**, and clearly distinguish the two.
2. Never invent runtime values, runtime-origin identities, nominal types, Method/Function categories, dynamic module-member targets, or heuristic results.
3. Return LSP `null` where no safe, useful result exists.
4. Reuse the existing D110 proof, immutable editor snapshot, canonical source-ownership check and LSP UTF-16 mapping.
5. Introduce no speculative framework, global/persistent index, guest execution, extra runtime cost, or duplicate TypeScript semantic authority.
6. Deliver handler, advertised capability, model/projection and regressions together rather than fragmenting into micro-slices.

This records the owner's LM010 feature acceptance requirements, **not a revision to any normative language semantics**. D110 and D124 keep their original narrow authority; neither is extended to prove arbitrary types or dynamic binding. No new unapproved substantive design choice is delegated to the implementation agent: any materially different public behavior, durable architecture or new language semantics must stop at its real decision gate rather than be guessed.

## Audited implementation entry points

The following paths and surface constraints were verified against the exact Protos HEAD above:

| Path / authority | Relevant fact |
|---|---|
| `AGENTS.md`, `AGENTS.work/{IMPLEMENTATION,COORDINATION,DESIGN,REFERENCE}.md`, `src/AGENTS.md`, `spec/AGENTS.md` | Human executes tests/builds/Git for product; substantive decisions require exact owner approval; slice and source-authority discipline |
| `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/PROTOS_GRAMMAR.md`, `spec/semantics/{OBJECT_MODEL,EXECUTION_AND_CONTROL,CALLABLES,MATCHING,MODULES,VALUES_AND_COLLECTIONS}.md` | Dynamic lookup, Closure invocation and parser forms; no guessed nominal type system |
| `ProtosLanguageServer.java` | Standard LSP4J server currently advertises definition, references, symbols, formatting; Hover not yet advertised |
| `ProtosTextDocumentService.java` | Live open buffer snapshot, `navigationSourceAuthority`, canonical ProjectBinding, `session.isCurrent`, definition/references publication barriers |
| `ProtosStaticAnalysisCore.java` | On-demand parser-authoritative definition/references queries, no guest evaluation; parse failures return empty |
| `ProtosStaticDefinitions.java` / `ProtosStaticDefinitionResult.java` | D110 generation-one exact same-activation Closure-parameter singleton origins only; opacity and side-effect barriers narrow proof |
| `ProtosStaticAnalysisSession.java` / `ProtosDocumentSnapshot.java` | Immutable versioned document snapshots; staleness checking; no existing reusable long-lived semantic proof index |
| `ProtosLspSourcePositions.java` | Existing UTF-16 `Position` ↔ source offset and `SourceSpan` ↔ `Range` |
| `parser/ast/SurfaceLiteral.java` | Parser records `NUMBER`, `STRING`, `TRUE`, `FALSE`, `NULL`; syntax, not a runtime-evaluated type |
| `parser/ast/SurfaceClosure.java` / `SurfaceParameter.java` | Exact Closure parameter declaration syntax, name, rest/default and source spans; not proof of an arbitrary runtime call target |
| `ProtosLanguageServerFoundationTest.java` and existing D110/D124 static/LSP tests | Prior capability assertions and fail-closed regression fixtures must be preserved/updated |

The current model has `SurfaceParameter.span()` possibly covering a default initializer; a Hover `range` must identify the actual hovered token or be omitted, never misrepresent the entire initializer as the identifier.

## Hover capability matrix / counterexamples

| Hover position | Allowed with proof | Disallowed / response when not proved |
|---|---|---|
| D110-proven same-activation parameter read | Explicitly labelled **Proven binding** with exact parameter origin and ordinary parameter role; optional properly mapped range | No lexical-name guessing if D110 returns empty, including after opaque calls |
| Closure parameter *declaration* | **Syntax**: parameter name, optional rest/default presence, declaration role from parser | No claim of runtime value, nominal type, inferred callable identity or evaluation result |
| Syntactically identified Closure expression | **Syntax**: its parameter shape, rest/default presence, expression/block body where parsed | No claim that unrelated symbol/call resolves to this Closure |
| Literal token | **Syntax**: number/string/boolean/null *literal form* supported by `SurfaceLiteral.Kind` | No materialized value, host type, inferred operational value or nominal type |
| Slot creation / structural document symbol | At most syntax explicitly proven by parser, if helpful and genuinely labelled | Structural name alone must not be called resolved lexical binding or runtime receiver member |
| Dynamic receiver/delegation, `this`, `super`, member sends, index | Only a directly visible syntactic form if implementation adds a useful, bounded category | No source origin/type/target guess, no workspace-name fallback |
| Module/import/package specifier | Syntactic literal only if already covered by generic literal logic | No canonical resolved module/package identity without separate authority |
| Unresolved name, opaque-effect barrier, invalid parse or position, closed/stale document, noncanonical URI, no exact ProjectBinding | None | `null`; no stale, non-authoritative or fabricated Hover |
| Legacy `match Binder/Alias` `@name` | **Not applicable**: D131 removed those grammar forms | Do not resurrect historical D110/D124 issue prose as current syntax |

Examples of unsafe claims: a parameter read after a guest call may have been rebound; equal spellings in independent Closures are not an identity proof; receiver delegation and module state are mutable and runtime-selected. Hover should never report such a target merely because an identifier or symbol index looks plausible.

## LSP and implementation boundaries

- Standard `textDocument/hover` handles `HoverParams`, returns `CompletableFuture<Hover>` with **Java `null`** when absent, enables `ServerCapabilities.hoverProvider`; use LSP4J APIs actually present at HEAD (LSP4J `1.0.0`). Contents should be concise human text, preferably `MarkupContent` plain text or Markdown. Keep claimed labels such as `Syntax` and `Proven binding` explicit.
- Compute source ranges using the server's UTF-16 helper; cover astral characters/CRLF and invalid positions. Do not expose misleading declaration-wide ranges where only a name is relevant.
- Apply current ProjectBinding/URI authority and freshness guards both before and after on-demand analysis, exactly as definition/references. Only analyze immutable currently open document bytes; never use disk truth over unsaved overlay.
- Implement editor-neutral extraction/projection in Java static analysis and a thin LSP adapter. Keep `guillermomolina/protos-vscode-extension` unchanged; a client receiving standard Hover already has the LSP request surface.
- Existing D110 and D124 proof semantics, diagnostics, symbols, formatting, source snapshot/domain ownership and no-duplicate semantic authority must remain unchanged. Ordinary guest execution and no-LSP embedding must incur **zero feature-specific work**.
- Tests: capability advertisement; semantic vs syntactic labelling; exact parameters/declarations/Closure/literals; unrelated same names; default-parameter ordering; opaque call invalidation; nested capture; dynamic member no guessing; malformed syntax; unowned/canonical URI; unsaved, didChange, didClose, stale snapshots; UTF-16/non-BMP and CRLF; null on unknown; definition/references no regressions.
- The implementation agent edits source and tests only and requests **human-executed** focal/affected validation and the appropriate integrated suite per changed-delta policy. `pom.xml` and root `CHANGELOG.md` are touched only after the required substantive validation is green, on the then-current HEAD; no tests after purely administrative metadata changes if no executable input changed. Human owns product `git add`, commit and push, with explicit owned paths, no force, no transient branches.

## Deliberately deferred work

LM010-B completion, LM010-C signature help, LM010-D integrated editor evidence/closure remain separate future phases under #493. LM009 ownership, including references and Marketplace packaging, does not move to LM010. This A0 outcome does **not** claim LM010 parent closure, a test result, a production change, a new runtime architecture or Dxxx/PLATxxx ratification. If a counterexample exceeds the exact bounded Hover acceptance above, report it; do not expand silently.

## Source/revision and validation honesty

Original activation evidence: [LM010-A0 reactivation handoff](LM010_A0_REACTIVATION_AND_HOVER_AUDIT_HANDOFF.md). Live tracking: [LM010 #493](https://github.com/guillermomolina/protos/issues/493).

```text
READ_ONLY_A0_AUDIT=COMPLETE
IMPLEMENTATION_OF_HOVER=NOT_STARTED
TESTS_BUILDS_EXECUTED_FOR_A0=NONE
GIT_DIFF_CHECK_FOR_PRODUCT=NOT_RUN
NEW_FORMAL_ISSUE=NOT_CREATED
OWNER_SELECTED_LM010_HOVER_SCOPE=YES
```
