# LM010-B0 — static completion authority audit and bounded LM010-B implementation handoff

**Date:** 2026-10-08. **Work owner:** [LM010 / guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493). **Role:** non-normative research evidence and implementation handoff, not executable validation or new language semantics.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_HEAD_FROM_B0=f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2
PRODUCT_HEAD_READ_BACK_ON_GITHUB=f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2
PRODUCT_VERSION_AT_B0=0.3.294-SNAPSHOT
PARENT=LM010/#493
B0_STATUS=INVESTIGATION_COMPLETE
B0_PRODUCT_MODIFICATIONS=NONE
B0_TESTS_BUILDS=NONE
B0_LOCAL_COMMANDS=READ_ONLY_INSPECTION_REPORTED
NEXT_SLICE=LM010-B
NEXT_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
DESIGN_GATE_FOR_D110_ONLY_CONSERVATIVE_SCOPE=NONE_IDENTIFIED
ISSUES_CREATED=NONE
```

## Evidence provenance

The LM010-B0 executor reported inspecting the **local checkout** at `f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2`, including read-only `git log -1`, `cat`, `sed`, `grep` and `ls`, rather than GitHub and without running builds, tests, state-changing Git or source modifications. The B0 report also explicitly acknowledges that it did **not** read `AGENTS.work/COORDINATION.md` or `AGENTS.work/DESIGN.md` and did not examine full D110/D124/PLAT024 decision records. **Do not claim literal zero commands or a complete decision audit.**

The coordinating agent independently checked GitHub main at that exact SHA and read governing `AGENTS.md`, `AGENTS.work/{IMPLEMENTATION,COORDINATION}.md`, current Protos parser/lexer/static-definition code, relevant specification provisions, and standard LSP completion `isIncomplete` semantics. Thus the main factual source and contractual conclusions below are supported; the proposed synthetic-cursor parse is still **an implementation hypothesis, not an executed working prototype**.

## Facts confirmed at current product revision

- `ProtosStaticDefinitions` D110 generation-one `Resolver` tracks only same-activation Closure parameter origins in a mutable `Facts` mapping; current `analyzeName` answers for one exact `SurfaceName` under the source offset. Opaque invocation and effectful forms erase facts; default-argument paths are intersected; nested Closure and object scopes do not inherit proven facts.
- `ProtosStaticAnalysisCore.parse` delegates to `ProtosParser.parseProgram`; a normal parse requires a full syntactically valid document. `ProtosParserSourceFacts` is produced only when parsing succeeds; it is not an existing incomplete-source recovery engine.
- `ProtosLexer` exposes token source spans, `tokenizeOccurrencesWithTrivia` and comments. Prefix-only lexing `source[0:cursor]` may help classify a source prefix and reject unclosed strings/comments/invalid lexemes without changing normative grammar. This needs actual regression tests before being declared robust.
- Open document snapshots and post-analysis `session.isCurrent`, exact canonical ProjectBinding admission, UTF-16 conversion and standard-LSP capability plumbing already exist through LM009 and LM010-A.
- Normative `CALLABLES.md` requires a parameter to be actually established before it becomes a local binding; the current/later parameters cannot be read as those local bindings from an earlier default. Normative `EXECUTION_AND_CONTROL.md` permits `removeSlot` and later re-creation in captured context, making dynamic presence and delegation unsuitable for optimistic completion.
- The current grammar has exactly `this`, `context`, `super`, `true`, `false`, `null` as reserved words, but `super` is **not** a standalone read expression. D131 removed matching-specific Binder/Alias syntax. Do not expand reserved-word suggestions to historical constructs, stdlib names or imaginary keywords.

## Candidate matrix: conservative implementation scope

| Context | Scope for LM010-B | Why |
|---|---|---|
| Same-activation parameter with D110-origin fact at a `SurfaceName` read site | Suggest with proven-binding detail | Same D110 proof and opacity/default barriers, no broadened source-origin model |
| Earlier parameters inside later default | Suggest only genuinely already bound parameter origins | CALLABLES binding order; avoid future parameter claims |
| Nested closure captures, ambiguous scopes | Do not suggest unproven names | D110 generation 1 does not prove captured origins |
| Names introduced by body slot creations | Do not suggest as proven binding | Structural syntax is not guaranteed runtime slot presence |
| Following effectful/opaque operations | No parameter-origin suggestions when `Facts` is cleared | A call may remove an earlier context slot |
| `receiver.`, `super.`, dynamic delegation | No member candidate guesses | No bounded member authority |
| Prelude, imports, packages/modules, workspace spelling matches | No candidates absent a separate proven authority | No inference from name lookup, text or source indexes |
| Expression-read position | Only `this`, `context`, `true`, `false`, `null` syntax suggestions; `super` excluded | Current normative grammar; these are syntax/reserved candidates, **not** a promise of resolved runtime slot identity |
| Malformed, stale, closed or noncanonical editor buffer | Fail closed | No fabricated or stale result |
| Comments, literal interiors, invalid lexical prefix, mid-identifier cursor | No candidates | Cursor classification and edit safety |

**Adversarial evidence:** `(first,k)=>{k(context);first}` can have first removed by k; `(a)=>{()=>a}` uses a capture that D110 cannot presently prove; `(a=b,b=1)` cannot read later parameter b as the local parameter; `{a:1}.` cannot prove member candidates; a prefix in `fi => 1` must not be confused with a read by discarding the real suffix.

A proof-only candidate is a **bounded proposal for implementation of existing D110 grammar authority**. It does not ratify wider syntax-only unproven-name proposals. Widening to captures, body slots or dynamic receiver members would require separately authorized behavior and/or D110-generation expansion.

## Partial-buffer feasibility and safety

Investigate/implement within the **single LM010-B implementation slice**, not via another research round:

1. Capture exact editor snapshot; permit `sourceOffset == source.length()` for completion (unlike Hover/Definition ranges), reject invalid UTF-16 positions.
2. Derive replacement prefix from lexer/token spans, with trivia admission. A cursor inside a comment/string, on a malformed prefix, or in the middle of an identifier must fail closed.
3. Preserve the real **entire** buffer for context classification, including text after cursor; never truncate the suffix to force a `SurfaceName`.
4. If full parse succeeds, use its actual AST/read-role; for an empty-prefix genuinely incomplete expression, optionally try **one** well-bounded parse with a synthetic identifier inserted at the cursor. Accept it only when it yields a real `SurfaceName` *read* at the synthetic token. Avoid identifier collision, lexical hazards and any keyword/statement transformation by explicit checks.
5. The proposed bounded strategy is still unproven across all malformed-buffer cases: when syntax cannot safely be classified, return no items, not a speculative parser recovery or partial-grammar shadow.
6. Reuse/restructure D110 shared traversal carefully to expose the exact `Facts` at a read site for *all* candidate names without widening Definition's origin proof. Do not regress Hover/References.
7. Form UTF-16 `TextEdit` for the exact replaceable prefix (avoid consuming unintended suffix), with rigorous non-BMP/CRLF and current-snapshot checks.
8. No background index, persistent cache, workspace guessing, guest evaluation, VS Code semantic duplication or ordinary-runner overhead.

### LSP nuance requiring a correction from the B0 report

Official LSP completion semantics define `CompletionList.isIncomplete=true` as *this candidate list is incomplete and further typing should recompute it*, **not** a generic flag for parse errors, lexer errors, stale results or unauthorized URIs. The B0 report proposes `true` for any empty list caused by failed analysis. **Do not hardcode that heuristic.** An empty fail-closed answer may be a complete empty list (`isIncomplete=false`) or `null` where permitted. Use `true` only if the server intentionally returns a known partial candidate set and can justify retriggering. See the [official LSP 3.17 specification](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.17/specification.md) and actual LSP4J `1.0.0` API. This is a protocol-correctness correction, **not** a new language design.

## Implementation packaging and authority

**NEXT: `LM010-B` / TYPE=IMPLEMENTATION / repo `guillermomolina/protos`.** Group source-model/extraction, context classification, handler/capability and Java analyzer/LSP regressions into one coherent slice. The **agent edits only**; the **human executes tests/builds/git**. Keep release metadata `pom.xml` and root `CHANGELOG.md` untouched until human reports the substantive tests green and derive the then-current next SNAPSHOT version. Do not rerun unchanged executable tests after metadata-only edits.

Do not independently change `AGENTS.md` in product through this docs action. The implementing agent reads and complies with all current instructions at HEAD; it should report any truly new design issue rather than treat this research document as normative approval.

**No new issue** is needed: LM010-B0 and LM010-B are bounded slices within the open #493. Signature help (LM010-C), integrated editor acceptance/closure (LM010-D) remain open. This record and implementation handoff do not assert builds/tests or completion implementation from B0.

```text
LM010_B0=INVESTIGATION_COMPLETE
NEXT_SLICE=LM010-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPO=guillermomolina/protos
DESIGN_SELECTED_FOR_BROADER_SYNTACTIC_COMPLETION=NO
NEW_D_OR_PLAT_ISSUE=NO
TESTS_EXECUTED_BY_COORDINATING_AGENT=NO
GIT_PRODUCT_WRITES=NO
LM010_PARENT_CLOSED=NO
```
