# LM012-A0 — Source diagnostics and lint authority audit

Status: **COMPLETED INVESTIGATION — D194 PUBLIC-POLICY CHECKPOINT REQUIRED**
Evidence date: **2026-10-08**
Owner: [LM012/#671](https://github.com/guillermomolina/protos/issues/671)
Public-policy decision (open, not approved): [D194/#842](https://github.com/guillermomolina/protos/issues/842)

```text
PROTOS_REVISION=3bb1278d91ee5cea98031462be2a5c4dd3c89019
VSCODE_EXTENSION_REVISION=c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a
PROJECT_DOCS_BASE_REVISION=60ef85905dd25768cabdcdbff7e892099ca34c78
LM012_A0=COMPLETED_INVESTIGATION
PRODUCT_EDITS=NONE
SPEC_EDITS=NONE
BUILDS_EXECUTED=NO
TESTS_EXECUTED=NO
PUBLIC_LINT_POLICY=NOT_SELECTED
IMPLEMENTATION_AUTHORIZED=NO
```

This is an investigation of published source and project authorities, not a report of executed acceptance tests. Read-only GitHub and standards/documentation access was used. No code was run.

## 1. Existing owners and actual behavior

| Layer | Verified current behavior | Consequence for LM012 |
| --- | --- | --- |
| Grammar and parser | `spec/PROTOS_GRAMMAR.md`, `ProtosParser`, `ParseError` define and detect lexical/syntax invalidity; `SourceSpan` anchors failures | Do not reclassify grammar violations or add parallel regex syntax checks |
| Editor-neutral static core | `ProtosStaticAnalysisCore.parse` calls the real parser and yields `ProtosStaticParseResult.Parsed/Failed` on an immutable `ProtosDocumentSnapshot`; no lint rule engine exists there | Natural reusable core, but parser errors are already claimed by LM009-G |
| Snapshot lifetime | `ProtosStaticAnalysisSession` owns document/workspace snapshot custody, current-version and stale-result guards | Lint would consume exact immutable snapshots and fail closed on supersession |
| LSP diagnostics | `ProtosTextDocumentService.didOpen/didChange` publishes a single parser failure as `DiagnosticSeverity.Error`, `source="protos"`, current document version and UTF-16 `SourceSpan` range; success publishes an empty list, close clears it without version | Existing `publishDiagnostics` pathway can carry approved lint records; aggregation must preserve parser errors, ordering, version and clearing |
| LSP capability | `ProtosLanguageServer` offers full-document sync, symbols, definition, references, formatting. No code-action provider is advertised | Quick fixes require new, separately authorized capabilities; do not claim they already work |
| Layout/trivia authority | `ProtosSourceLayoutView` provides on-demand exact source/tokens/trivia/Surface-tree relationships for the formatter, not a mandatory global CST or lint policy | Candidate source-only checks may reuse this optional evidence; no new runtime or daemon needed for initial rules |
| Project/definition proofs | `ProtosStaticDefinitions` proves only bounded exact closure-parameter and match-binding origins; deliberately refuses guessed workspace/member/receiver/module identities | No broad undefined-name, unused-property or guessed-dead-code lint based on names alone |
| Tool CLI | `ProtosCli` dispatches run/debug/language-server/package/test/format; no public lint/check driver command is listed | CLI spelling/exit and CI contract remain future public policy, not implementation trivia |
| Extension | `guillermomolina/protos-vscode-extension` creates a standard `LanguageClient` using the Protos executable and does not own a TS lint engine | VS Code Problems can consume server diagnostics; no independent TS static analysis |
| Formatter | D183/D184 and LM011 own canonical formatting, preserving source/semantics through the selected formatter authority | Formatter difference does **not** automatically define a lint violation or imply formatting is lint-clean |
| Matching | D096/#390 asks unresolved exhaustiveness/redundancy questions over effectful open matching | Do not smuggle match analysis into LM012 as an already-authorized rule |

The evidence does not establish a separate host/compiler static-semantic-warning pipeline reusable as a lint source. Existing CLI/runtime failures, when they occur, are not proof that a warning is statically sound over unexecuted source.

## 2. Soundness boundary, with counterexamples

| Candidate finding | Provability at this revision | Correct action |
| --- | --- | --- |
| Invalid lexeme, unexpected token, unterminated literal, illegal separator | Parser-owned and already surfaced through LM009-G1 | Keep parser error, do not duplicate as lint |
| Trailing horizontal whitespace outside comments and String tokens | Potentially mechanically recognizable with exact source plus lexer/trivia; classification still needs careful token boundary and multiline literal proof | **Possible opt-in hygiene candidate**, not an approved rule or automatic fix |
| Repeated blank lines, indentation style, redundant parentheses, formatting differences | Syntax-preserving editor style, not a Protos correctness property; lexical/newline and literal preservation are nontrivial | Leave to LM011 formatter unless distinct rule policy is ratified |
| Unused parameter or unused bare name | Dynamic lexical contexts, ordinary lookup, reflection, captures, member/delegation and invocation defeat generic closed-world inference | Do not diagnose without a bounded static proof and approved false-positive threshold |
| Assignment to missing slot | `=` selects existing writable binding according to runtime context and ordinary program effects | Cannot infer absence solely from local text |
| Duplicate source slot spelling | Construction, composition, mutation and ordinary slot semantics are context-dependent | No generic name-count heuristic |
| Unreachable code / non-local return / concurrency misuse | Control effects and runtime dispatch need stronger semantic proof; optimizer analyses are not editor policy | No heuristic lint masquerading as guaranteed correctness |
| Non-exhaustive or redundant match case | Open/effectful matcher semantics; separate D096 decision remains open | Exclude until owning authority is ratified |
| Missing imports/package module | Only canonical exact source inventory and project binding may prove identities | No filesystem/workspace-name guess |
| Source text with an incomplete parse | Parser already owns the initial failure; partial AST proof may be stale or incomplete | Default proposed fail-closed: do not emit unsupported semantic lint |

Concrete adversarial examples: trailing whitespace **inside** a triple-double String can be part of its value; a same-name member may be added dynamically after a call; a closure may mutate a captured context; ordered match guards and custom `pattern.match` may have effects. None can be assumed away for attractive editor squiggles.

## 3. Boundaries mechanically reusable today

```text
canonical source/parse authority
      -> immutable snapshot + SourceSpan
      -> optional on-demand rule evaluation [ONLY APPROVED RULES]
      -> editor-neutral diagnostic records [rule identity and severity: D194]
      -> LSP mapping/publishing via PLAT024 [LM009-G1 path]
      -> VS Code Problems or other standard LSP client
```

No new process, global index, durable cache, long-lived runtime listener, Truffle Context, Actor/Task allocation or TS semantic implementation is necessary merely to evaluate source-local rules from an active LSP request. Optional on-demand source-layout evidence already exists. Cross-file rules would need independently proven project identity and lifetime/cost analysis.

PLAT024 therefore **appears sufficient** for initial source-local linting. A new PLATxxx must be raised only if later implementation identifies a genuinely new durable process/index/lifetime authority—not because LM012 has a new feature name.

The common toolchain architecture in `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md` says officially bundled developer-tool policy should normally use the common tool boundary when expressible there; a future promoted lint tool belongs to the `TOOLxxx` family, not automatically CLI or LM-specific Java policy. This audit does not select a new command, tool package or Java/Protos execution bridge.

## 4. Public choices that are **not** mechanically settled

1. Rule inventory, default versus opt-in, categories, versioning and proof threshold.
2. Stable code/ID/source, severity levels and ordering; whether warnings ever fail a CI gate.
3. Invalid/incomplete-document treatment, disable/suppression mechanisms and configuration scope.
4. LSP warning presentation versus CLI/CI output and capability negotiation.
5. Whether autofix exists, explicit-only versus automatic, proof and conflict checks, and stale-buffer revalidation.
6. CLI command spelling, stdout/stderr, exit codes, file/project scope, and whether a separate bundled `TOOLxxx` is warranted.

These affect public tooling behavior and require the D194 decision checkpoint before LM012-B implementation. D194 must not preempt separately governed CLI/bundled-tool specifics.

## 5. Work decomposition and result

- **LM012-A0:** completed static ownership audit (this evidence).
- **D194:** comparative public lint-policy investigation; separate independently blocked approval checkpoint, not selected/ratified. Its research packet is `docs/project/evidence/D194/D194_LINT_POLICY_COMPARATIVE_RESEARCH.md`.
- **LM012-B:** pending ratification; smallest rule implementation should be in canonical Protos static core, with focused tests and no false semantic inference.
- **LM012-C:** after B, decide toolchain/CI adapter under actual public authority; do not reserve `protos lint` or `protos check` by assumption.
- **LM012-D:** merge approved lint diagnostics with existing push diagnostics, maintaining exact version/cancellation/close behavior and a thin extension.
- **LM012-E:** only demonstrably safe quick fixes, if approved; never automatic source mutation merely because LSP supports actions.
- **LM012-F:** end-to-end CLI/CI/real installed editor evidence and final parent closure, after required maintainer-executed tests/build/publication.

No child LM012-A/B issue is needed for a mere slice. D194 is independent because approval can block implementation. Native GitHub hierarchy for D194 -> LM012 remains pending due to the available connector lacking a sub-issue mutation action.

```text
LM012_A0_AUDIT=COMPLETE
PUBLIC_LINT_POLICY_GATE=D194/#842
RATIFICATION=NOT_APPROVED
PLAT_NEW_GATE=NOT_ESTABLISHED
LM012_IMPLEMENTATION=NOT_AUTHORIZED
LM012_PARENT_CLOSED=NO
NEXT_ACTION=OWNER_DECISION_AFTER_D194_RESEARCH
```

## 6. Material sources inspected

**Product / exact reference HEAD:** `AGENTS.md`; `AGENTS.work/IMPLEMENTATION.md`; `AGENTS.work/COORDINATION.md`; `AGENTS.work/DESIGN.md`; `AGENTS.work/REFERENCE.md`; `src/AGENTS.md`; `spec/AGENTS.md`; `spec/PROTOS_LANGUAGE_SPEC.md`; `spec/PROTOS_GRAMMAR.md`; `spec/semantics/OBJECT_MODEL.md`; `spec/semantics/EXECUTION_AND_CONTROL.md`; `spec/semantics/CALLABLES.md`; `spec/semantics/MATCHING.md`; `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md`; `docs/design/IDEAS.md`; `src/main/java/com/guillermomolina/protos/parser/ParseError.java`; `parser/ProtosParserSourceFacts.java`; `analysis/ProtosStaticAnalysisCore.java`; `analysis/ProtosStaticParseResult.java`; `analysis/ProtosStaticAnalysisSession.java`; `analysis/ProtosStaticDefinitions.java`; `analysis/ProtosSourceLayoutView.java`; `lsp/ProtosLanguageServer.java`; `lsp/ProtosTextDocumentService.java`; `cli/ProtosCli.java`; `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDiagnosticsTest.java`; and relevant issues LM009-G/#360, LM012/#671, PLAT024/#342, D096/#390, LM011/#670, D183/#791.

**Extension:** `guillermomolina/protos-vscode-extension:extension.js`, `package.json` at exact HEAD above.

**Durable records:** `guillermomolina/protos-project-docs:AGENTS.md`, `docs/project/README.md`, `docs/project/evidence/README.md`, `docs/project/registries/IMPLEMENTATION_BLOCKERS.md`, LM011-A prior audit, and LM010-A0 handoff. The blocker ledger text search found no existing LM012-specific entry.

This document does not certify runtime tests or a `git diff --check` run. Documentation-link/format validation remains a distinct publication/maintainer gate.
