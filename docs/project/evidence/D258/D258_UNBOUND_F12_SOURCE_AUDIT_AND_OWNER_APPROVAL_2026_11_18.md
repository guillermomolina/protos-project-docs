# D258 — Unbound F12 source audit, observed limitation and owner approval (2026-11-18)

**Type:** revision-bound investigation and ratification evidence, not post-implementation validation.
**Canonical approved decision:** [D258-B durable decision](../../decisions/tooling/D258_UNBOUND_DOCUMENT_LOCAL_GO_TO_DEFINITION_AUTHORITY.md).
**Live:** [D258 #1134](https://github.com/guillermomolina/protos/issues/1134), implementation [I168 #1135](https://github.com/guillermomolina/protos/issues/1135).
**Product research revision:** `guillermomolina/protos@91b92332f6870fd9f0903c829c2728a644d0dca4`.
**Runtime observed:** official `v0.4.158`; VS Code extension source `guillermomolina/protos-vscode-extension@9216e0b06f19497de7c5030f4f36de500a020f64`, extension package `0.2.4`.

## Maintainer observation versus source conclusions

1. In VS Code `/workspaces/protos-devcontainer` the maintainer observed F12 failure/no target. A real terminal listing found `package.toml`, `protos.lock`, `protos.project` **all absent**. No runtime process trace for that individual F12 request was provided.
2. Independently inspected current source at `91b92332...`: `ProtosTextDocumentService.definition()` returns empty **before** consulting `session.definitionCurrent()` whenever `navigationSourceAuthority.test(uri)` fails; this predicate is connected to `ProtosWorkspaceService.ownsCanonicalSourceUri()` and canonical project binding. Thus absent ProjectBinding explains the observation under the current approved contract; it is not evidence of a broken extension keyboard mapping.
3. The source proof engine `ProtosStaticDefinitions.resolve()` already works directly on one parser-authoritative immutable buffer snapshot, independent of project metadata; `ProtosStaticAnalysisSession` owns exact open-buffer snapshot custody and `isCurrent()`. Positive Closure parameter and contextual-`for` proof unit tests already exist. After unknown invocation, facts are discarded. No general dynamic/member/module/slot proof is claimed.
4. `ProtosLanguageServerDefinitionTest.looseOrNonCanonicalOpenDocumentNeverGainsDefinitionAuthority` and integrated JSON-RPC `ProtosLanguageServerIntegratedEditorTest` include deliberate negative F12 assertions for unbound, alias and no-workspace sources. The tests therefore document **old policy** and must be updated as part of I168; their historical pass is not evidence of newly approved behavior.
5. The current maintained LSP guide describes unbound Definition as unavailable and still mentions legacy `match` binders, while normative `spec/PROTOS_GRAMMAR.md` and `spec/semantics/MATCHING.md` confirm D131 removed dedicated match-pattern/binder syntax.

## Materially inspected files and consequences

| Repo and path | Exact insight |
| --- | --- |
| `protos:AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/AGENTS.md`, `src/AGENTS.md` | GITHUB010 12-axis decision, GITHUB021 approved invariant reconciliation, human executes product tests/Git, decision record under tooling role |
| `protos:spec/PROTOS_LANGUAGE_SPEC.md`, `spec/PROTOS_GRAMMAR.md` | Dynamic language; Closure/slot syntax; contextual-`for`; D131 removed special matching grammar |
| `protos:spec/semantics/EXECUTION_AND_CONTROL.md` | Lexical context objects are mutable; reads search lexical parents then receiver; opaque effects invalidate presumptive binding |
| `protos:spec/semantics/CALLABLES.md` | Closures capture by reference; parameter binding and invocation are not JavaScript/Python immutable lexical variables |
| `protos:spec/semantics/OBJECT_MODEL.md`, `MODULES.md`, `MATCHING.md` | Slots can be created/modified/removed, import identities differ, pattern capture names are ordinary parameters not special binders |
| `protos:src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java` | LSP advertises `definitionProvider` and wires the canonical project predicate |
| `protos:src/main/java/com/guillermomolina/protos/lsp/ProtosWorkspaceService.java` | Owns exact ProjectBinding/source URI membership; must remain unchanged |
| `protos:src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java` | Premature project gate on Definition; open/change/close snapshot custody; proof currentness/target equality checks already present |
| `protos:src/main/java/com/guillermomolina/protos/lsp/ProtosLspSourcePositions.java` | Exact positions mapped to UTF-16 source offsets, CRLF/mixed newline handling |
| `protos:src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java`, `ProtosStaticAnalysisCore.java`, `ProtosStaticDefinitionResult.java`, `ProtosStaticDefinitions.java`, `ProtosStaticReferences.java` | Existing independent parser/source proof; immutable snapshots; D110 result can carry complete same-source target set; D124 separate |
| `protos:src/test/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitionsTest.java`, `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDefinitionTest.java`, `ProtosLanguageServerIntegratedEditorTest.java` | Baseline positive proof, bound F12, and explicit negative loose/no-workspace/alias F12 cases |
| `protos:docs/guide/tools/language-server.md` | Maintained feature contract currently requires project for all definition; must describe scoped exception and current proof coverage after I168 |
| `protos-project-docs:docs/project/decisions/tooling/D085_...`, `D089_...`, `D110_...`, `D124_...`, `D209_...` | Ratified predecessor boundaries; D258 explicitly reopens **only** D209 Definition gate and D110 source-identity clauses 11–12 for same-open-snapshot results |

The legacy `D085`/`D089` filenames and their full contracts are preserved. Their exact local candidates, verified project metadata and bounded source inventory are still required where a request claims **canonical project source** authority.

## Relevant external implementations — bounded applicability

- `oracle/graalpython@3289197e6373d8424d8c84d13187cc44ecd3a31d`: `pegparser/sst/SSTNode.java` records a parser-owned `SourceRange`; `ParserCallbacksImpl.java` converts ranges to `SourceSection` only after bounds validation. This is a concrete Truffle-language precedent for **exact source spans** without inventing guest source bindings. No GraalPy JIT/bytecode execution path is needed for this editor rule.
- TruffleRuby / GraalJS preserve source locations through parser/runtime representations; neither justifies a guessed Protos receiver/member lookup.
- GraalVM's generic Truffle LSP supports dynamic runtime-derived tooling; it is explicitly rejected here because D110/D209 prohibit guest execution for static F12.
- TypeScript inferred project, clangd fallback compilation, Pyright/Jedi in-memory documents, rust-analyzer explicit project model, and LuaLS fallback scope were compared in the D258 decision packet. The approved combination is narrow in-memory proof plus canonical project authority for external results, **not** inferred project identity.

## Exact owner approval and evidence boundary

The owner received a full design packet in chat: alternatives A/B/C/D, D258-B target contract, D209-C and D110-B′ express changes, D085/D089/D124 invariants, 12-axis assessment and strongest objection. They responded **“si apruebo todo eso que dices.”** This is the recorded approval evidence. No duplicate issue-level approval comment was fabricated.

The owner also stated that local tests passed and `git diff --check` was clean, **before the I168 implementation was written**. These statements refer to the current Protos baseline. No new tests have been run for the newly ratified document-local F12 behavior; no new product commit, real VS Code F12 acceptance, or new runtime release is claimed.

## Required next evidence

The I168 implementation must yield: exact Protos revision; changed source/tests/docs; human-executed focal and integrated `make test` / applicable `make check`; clean `git diff --check`; real editor F12 result for no-workspace and no-metadata root; continued negative unbound References and Workspace Symbols; unchanged project-bound behavior; exact extension and Protos Native source identities. Until then I168 stays open.

```text
EVIDENCE=D258_2026_11_18_RESEARCH_AND_APPROVAL
PROTOS_RESEARCH_REVISION=91b92332f6870fd9f0903c829c2728a644d0dca4
APPROVAL=D258_B_EXPLICIT_2026_11_18
D209_REOPEN_SCOPE=DEFINITION_ONLY_LOCAL_CURRENT_SNAPSHOT
D110_REOPEN_SCOPE=SOURCE_AUTHORITY_CLAUSES_11_12_ONLY
D085_D089_D124=UNCHANGED
BASELINE_TESTS=HUMAN_PASS_REPORTED
BASELINE_GIT_DIFF_CHECK=HUMAN_CLEAN_REPORTED
I168_IMPLEMENTATION_TESTS=NOT_RUN
I168_PUBLICATION=PENDING
REAL_F12_VSCODE_ACCEPTANCE=PENDING
```
