# I117-B — Contextual while syntax: implementation-editor and human-executor handoff

**Date:** 2026-10-21
**Work:** [I117/#942](https://github.com/guillermomolina/protos/issues/942), phase B; no new formal Issue
**Decision:** [D210](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md), Q2 RATIFIED
**Product base observed:** [`5c76e4a633c9aa44067fe0b4fd68cafea365750d`](https://github.com/guillermomolina/protos/commit/5c76e4a633c9aa44067fe0b4fd68cafea365750d)
**Previous phase:** [I117-A publication evidence](../../evidence/I117/I117_A_D210_Q1_PUBLISHED_SINGLE_LOOP_PROTOCOL_2026_10_21.md) (`0.3.393-SNAPSHOT`, spec `0.1.464`, tests PASS in maintainer's publication record).
**Class:** IMPLEMENTATION, repository `guillermomolina/protos`. No investigation or implementation execution was performed for B by this document.

## Editor batch 1: concrete source edits

1. `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceContextualWhile.java`: new source-faithful Surface record with condition expression, trailing body Closure and complete source span; full project APL header.
2. `src/main/java/com/guillermomolina/protos/parser/ast/SurfaceExpression.java`: extend sealed `permits` list.
3. `src/main/java/com/guillermomolina/protos/parser/ProtosParser.java`: in `parseCallSuffix`, admit contextual form **only** for a bare `SurfaceName("while")`, one non-spread explicit argument, and an immediately attached trailing braced parameterless Closure. Construct the new node *before* registering a `TrailingClosureOrigin` tied to an ordinary `SurfaceCall`; never record unreachable source-fact nodes. All other forms remain ordinary calls.
4. `src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java`: lower contextual node to ordinary `CanonicalSend` `whileTrue` on a zero-parameter lexical `CanonicalClosure` whose body reevaluates the source condition; the trailing Closure is the sole argument. Preserve lexical scope, source spans and the ordinary dispatch/guarded fallback semantics. Do not implement a direct second loop.
5. `src/test/java/com/guillermomolina/protos/parser/ProtosParserTrailingClosureTest.java`, `ProtosParserPostfixFoundationTest.java` and `src/test/java/com/guillermomolina/protos/semantic/CanonicalizerCallTest.java`: focused source-form and canonical dispatch tests.

## Editor batch 2: source tools, language tests and normative prose

Handle the new sealed Surface node in `ProtosStaticHover`, `ProtosStaticLint`, `ProtosStaticReferences`, `ProtosStaticDefinitions`, `ProtosDocumentSymbols`, `ProtosSourceLayoutView` and `ProtosSourceLayoutToolBridge`; traverse the condition and body as actual source constructs, and do **not** count the syntactic `while` name as a lexical lookup/reference. Keep source facts, structural paths, trailing-Closure origins, signature help, formatting, DAP source locations and LSP navigation sound. Update `spec/PROTOS_GRAMMAR.md`, the navigation note of `spec/PROTOS_LANGUAGE_SPEC.md` if needed, `docs/guide/04-control-flow-through-protocols.md`, and focused Protos conformance plus Java/tool tests.

Approved precedence exception is **only** `while(x) {body}`; ordinary `while(x)`, `while(a,b) {body}`, `(while)(x) {body}`, `obj.while(x) {body}`, `while {body}`, `while(x)\n{body}`, and `while(...args){body}` are not contextual forms.

## Prior art supplied to editor (no external browsing needed)

- Oracle Truffle SimpleLanguage `SLBytecodeParser.visitWhile_statement` uses `beginWhile/endWhile`, but D210-Q2 requires reuse of Protos's already-canonical `whileTrue` message; do **not** copy its distinct statement runtime.
- GraalJS `WhileNode` uses `LoopNode`/`RepeatingNode` as an independently compiled JavaScript *statement*; do not copy JS truthiness or statement return values into Protos.
- GraalPy control constructs preserve Python truthiness and statement semantics; they do not authorize new Protos semantics. A parser-only surface sugar and ordinary canonical `whileTrue` lowering is the bounded cross-language conclusion.

## Human-executor boundary

Only the agent edits the source files. The human inspects diff and runs focal Java/parser/analysis, affected Protos conformance, and `make test` according to `AGENTS.work/IMPLEMENTATION.md`. Hold `pom.xml`, `CHANGELOG.md`, `spec/PROTOS_SPEC_CHANGELOG.md` until finalization **after** substantive tests are green. Unlike I117-A's first publication attempts, **all three** metadata files must be present in the final product publication with the implementation and normative revision, based on actual local HEAD.

No code, tests, builds or publication of I117-B are claimed by this handoff. Issue I117 stays open until B is implemented, validated and published.
