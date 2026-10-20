# D209 — Unbound document-local LSP intelligence authority

**Status:** RATIFIED — Candidate C, explicitly approved by the project owner on 2026-10-20.
**Live decision:** [D209 / #936](https://github.com/guillermomolina/protos/issues/936); [exact approval record](https://github.com/guillermomolina/protos/issues/936#issuecomment-6289579502).
**Implementation consumer:** [LM013 / #861](https://github.com/guillermomolina/protos/issues/861), remains open for implementation and real-editor acceptance.
**Decision type:** implementation-independent, observable editor/tooling authority. No Protos language semantics, canonical source identity, ProjectBinding ABI, or Standard Library API changes.
**Research source revision:** [guillermomolina/protos@d32524bd1dec045966dc33f3a3f58abc4dcf0278](https://github.com/guillermomolina/protos/commit/d32524bd1dec045966dc33f3a3f58abc4dcf0278). The subsequently observed product `main` revision is [0bceacdfc0127453729f346382152bd9908f3de1](https://github.com/guillermomolina/protos/commit/0bceacdfc0127453729f346382152bd9908f3de1); it contains the completed independent BUG022 formatter publication, **not** LM013 implementation.
**Historical accepted comparator:** [LM010-D bound-project VS Code acceptance](../../evidence/LM010/LM010_D_REAL_VSCODE_ACCEPTANCE_AND_CLOSURE.md) (Native Protos 0.3.312, extension 0.2.2).
**Supporting investigation:** [D209 decision alternatives and scorecard](https://github.com/guillermomolina/protos/issues/936); [LM013 source-based authority evidence](../../evidence/LM013/LM013_D209_BOUNDARY_AND_RATIFICATION_EVIDENCE.md).

## Selected contract — Candidate C

A request for **Hover, Completion or Signature Help** on an open Protos document may obtain facts proved solely from the *current exact in-memory text snapshot* held by the server, even if that document is not owned by a canonical Protos project.

The local-only facts are **exactly the already-implemented LM010 generation-one set**:

- **Hover**: parser-proven literal/Closure declaration/Closure head syntax, labeled `Syntax`; and uniquely D110-proven *same-document* Closure parameter origins labeled `Proven binding`. Do not invent local slot, member, import, nominal type, runtime value, or dynamic receiver facts.
- **Completion**: parser-accepted reserved expressions (`this`, `context`, `true`, `false`, `null`) and Closure parameter names proven in scope by existing D110 generation-one local reasoning. Do not suggest project/module/Std Library or same-spelling member/slot names without separate exact proof.
- **Signature Help**: parameter shape of a **literal Closure callee**, after grouping parentheses only, when the existing parser/lexer and bounded incomplete-buffer proof establish that exact call. No Signature Help for a Closure value obtained through a name, dynamic member, import or computed call target.

This is **not** generalized Protos project membership. Opening or changing a document establishes only **buffer custody**, not membership in any canonical package/project/source inventory. The same in-memory analysis is available to a loose file, unsaved buffer, or file outside any workspace; it is on-demand and requires no disk/project contents. The local-only result must derive from its exact requested URI and snapshot, with no URI normalization used to infer project authority.

Keep `textDocument/definition`, `textDocument/references` and `workspace/symbol` gated by the existing exact canonical ProjectBinding/source authority. They may never gain a best-effort local-only fallback through D209. Existing bound-project Hover, Completion, Signature Help, navigation and workspace results must remain unchanged.

## Freshness, failure and resource boundaries

- Preserve existing full-document `didOpen`/`didChange` snapshot custody and `didClose` eviction. Publish no result from a superseded/closed snapshot.
- Preserve request-position validation, source offsets, UTF-16 and non-BMP handling, CRLF/mixed-line-ending behavior, and fail-closed malformed/ambiguous syntax.
- Source snapshot *replacement order* establishes currentness in `ProtosStaticAnalysisSession`; its numeric version is not currently a monotonicity-enforced authority. This decision does **not** introduce a new version-counter semantic.
- An alias such as `file:///..././Main.protos` is **not** thereby admitted to a bound project. If independently opened, it can receive only truthful local facts from the exact alias-keyed buffer; no path-identity or cross-document claims follow.
- Do not run guest Protos, create a Truffle Context, import or resolve packages, scan folders, index dependencies, start background daemons, invoke Package Tool, or duplicate analysis in the VS Code extension.
- Cost is paid only when editor-local intelligence is requested. Runtime/ordinary non-editor execution must not acquire new startup/indexing cost.

## GITHUB021 predecessor invariant/delta reconciliation

| Existing authority | Retained | D209-C exact prospective change |
|---|---|---|
| D085/F′ and D089/A″ | Package Tool owns canonical ProjectBinding and exact source/project membership; no workspace/path/URI inference | Historical `G2/document-only` feature coverage for loose files is extended **only for the three current-document LM010 queries**; no project identity is created |
| D110/B′ | Definition requires a complete finite static proof set; no name/fuzzy/path guessing | Definition remains project-gated; Hover/Completion reuse already-existing *same-snapshot* local proof only |
| D124/B′ | References use exact singleton seed and sound proven occurrences; no rename guarantee | References stay project-gated and unchanged |
| LM009/LM010 | Static analysis, canonical parser, labeled facts, exact overlay, conservative generation-one capabilities | Only removal of unnecessary **project authority gate** for three local feature requests; their semantic result sets do not expand |
| Normative grammar/Closure semantics | `spec/PROTOS_GRAMMAR.md`, `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md` stay authoritative | No normative Protos language revision or guest behavior change |

**Consistency finding:** PASS, because the exact owner-approved scope explicitly supersedes the older **feature-coverage restriction**, not the previously selected ProjectBinding/definition/reference authority. The strongest risk is mistaking local proof for project proof; reject any implementation that fuses these authorities.

## Research and selection rationale

[D209 / #936](https://github.com/guillermomolina/protos/issues/936) records the investigated alternatives, twelve-dimension GITHUB010 comparison, confidence, counterexamples and prior-art URLs. The owner was presented with and selected **C** rather than:

- **A**: unchanged bound-project-only UI (truthful but leaves basic loose-file assistance absent);
- **B**: parser syntax only, artificially excluding already available same-document Closure parameter proof;
- **D**: inferred projects, workspace/path scans, broad name/module fallback (disqualified by D085/D089 and proof-first constraints).

Relevant precedents: TypeScript inferred loose-file projects (not copied because inference is too broad), Jedi in-memory analysis, clangd fallback compilation, rust-analyzer externally sourced project descriptions, Pyright open-buffer snapshots, and standard LSP document-position requests. Unlike their more permissive discovery models, Protos's project/source identity remains explicit and canonical.

**Present sufficiency:** Candidate C reuses the existing parser and proof engine, adds no new project provider or indexing architecture, and costs only requested local analyses. **Incremental growth:** module/cross-file intelligence can be considered later behind separate canonical authority. **Regret/escape:** editors might overinterpret a local binding as globally authoritative; keep labeling and project-only navigation strict, and independently add stronger proven project facilities later if required.

## Handoff and acceptance ownership

Implementation belongs to **LM013 / #861**, in `guillermomolina/protos`, with agent edits and human-executed validation, product version/changelog finalization, commit and push. Work from **actual local product HEAD**, not the research commit above.

A coherent implementation should adjust the three LSP feature handlers, **not** the definition/references handlers or `ProtosWorkspaceService::ownsCanonicalSourceUri`; replace old loose-file negative tests with proof-positive/negative/URI/snapshot cases, and extend the integrated JSON-RPC test. Update maintained `docs/guide/tools/language-server.md` to describe the new boundary after code validation. The independent BUG022 formatter and DAP/VS Code extension must not be altered. Exact human-executor commands/validation are maintained in [LM013 implementation handoff](../../work/LM013/LM013_AGENT_HUMAN_EXECUTOR_HANDOFF.md).

**Owner approval:** exact `Apruebo C` after the presented D209 candidate set and authority constraints, 2026-10-20. Project docs publication itself is not evidence of product implementation or successful tests.

```text
D209_DECISION=C
OWNER_APPROVAL=PASS_2026_10_20
DECISION_INVARIANT_CONSISTENCY=PASS
CANONICAL_PROJECT_AUTHORITY=UNCHANGED
LOOSE_LOCAL_INTELLIGENCE=AUTHORIZED_ONLY
LM013_IMPLEMENTATION=NOT_YET_PUBLISHED
LM013_REAL_EDITOR_ACCEPTANCE=PENDING
LANGUAGE_SPEC_CHANGE=NONE
```
