# D258 — Document-local Go to Definition authority for unbound Protos files

**Status:** RATIFIED — **Candidate B**, explicitly approved by the project owner in the originating ChatGPT conversation on **2026-11-18**: “si apruebo todo eso que dices”, immediately after receiving the complete D258-B proposal and its express D209-C/D110-B′ amendments.
**Live design:** [D258 / #1134](https://github.com/guillermomolina/protos/issues/1134).
**Implementation consumer:** [I168 / #1135](https://github.com/guillermomolina/protos/issues/1135), independently tracked. Implementation, product tests, commit, push and real-editor acceptance remain **pending**.
**Decision domain:** implementation-independent editor/tooling source authority. No new guest Protos semantics, grammar, runtime/JIT behavior, standard-library API, ProjectBinding format or executable protocol.
**Research product source:** [`guillermomolina/protos@91b92332f6870fd9f0903c829c2728a644d0dca4`](https://github.com/guillermomolina/protos/commit/91b92332f6870fd9f0903c829c2728a644d0dca4) (2026-11-18 research baseline). User's observed runtime was official [`v0.4.158`](https://github.com/guillermomolina/protos/releases/tag/v0.4.158). Do not confuse research baseline with implementation or release revision.
**User observation:** `/workspaces/protos-devcontainer` is the VS Code workspace root, with no `package.toml`, `protos.lock` or `protos.project`; F12/Go to Definition is empty. The extension `0.2.4` CI002 full acceptance with Protos `v0.4.158` passed the existing scenarios but did **not** test F12. This issue is an intentional-contract review, not an established VS Code/Native regression.

## 1. Exact approved behavior — Candidate B

For standard `textDocument/definition`, the current open document's exact server-held immutable source snapshot may serve as **document-local source authority**, independently of canonical ProjectBinding, **only if** all of the following hold:

1. The exact requested URI is presently open in the LSP document custody, and the source offset is valid for its current full-document snapshot. URI text is an opaque exact identity; no normalization or filesystem ancestry grants extra authority.
2. The canonical Protos parser and existing D110 static proof engine derive a **complete finite set** of source-backed binding origins for the requested read. The empty/unknown/incomplete case returns the ordinary empty LSP result; same-name, approximate, ranked and partially proven locations are forbidden.
3. The reference, every returned target and the *current* server-held snapshot have **the same exact URI and immutable snapshot value**. Any target outside this document is not admitted merely because the source is open or a filename matches. Where project-wide or cross-file source identity is required, the independently approved D085/D089 canonical ProjectBinding remains mandatory.
4. At publication, retain all existing D110 checks for currentness, source spans, UTF-16 LSP positions, malformed syntax, closed or replaced buffers, and fail-closed responses. No location from a superseded/closed snapshot may be presented.
5. Successful local results use the already-supported LSP standard multi-location shape and D110 deterministic complete-target policy. A finite complete multi-origin set entirely inside the same snapshot is allowed if future proof analysis supports it; there is no arbitrary result cap or synthetic single origin.
6. This **does not promote** a loose document to a project, package, module, workspace source, import target, canonical filesystem location or cross-document authority. Open-buffer custody and project/source ownership are distinct.

An unbound, open source such as:

```protos
f: (value) => {
    value
}
```

can report the proven parameter declaration when F12 is requested on the body `value`. It cannot infer the definition of `this.slot`, `object.method`, an import or an arbitrary same-name source elsewhere.

Generation 1 is intentionally constrained by **existing `ProtosStaticDefinitions` proof coverage**: mainly same-activation Closure parameters and exact parameter origins lowered from contextual `for`, before invalidating effects. D258 does not require any new generic local-slot, method, member, imported module, nested captured-outer-context, or cross-document origin algorithm.

## 2. Explicit predecessor reopening and GITHUB021 reconciliation

This ratification approves **exactly two amendments** to existing owner-approved tooling contracts; all other invariants survive:

| Authority | Before D258 | D258-B delta / preserved invariant |
| --- | --- | --- |
| [D209-C / #936](https://github.com/guillermomolina/protos/issues/936) | Hover, Completion and Signature Help use local snapshots, but **all** Definition, References and Workspace Symbols require ProjectBinding | **Explicitly reopen the Definition-only prohibition** for complete proven same-URI/snapshot origins. Hover/Completion/Signature Help unchanged. **References and Workspace Symbols still require ProjectBinding**. D209-C's exact URI, lifecycle, snapshot, no-runtime and no-project-inference safeguards remain. |
| [D110-B′ / #435](https://github.com/guillermomolina/protos/issues/435) | Definition must provide a complete finite proof set; clauses 11–12 require exact ProjectBinding source identity and allow open overlay only for an already-authorized canonical source | **Explicitly amend only source authority clauses 11–12**: the exact presently open single-document snapshot is independently admissible **for same-snapshot/URI targets only**, not as canonical project authority. All D110 semantics, proof completeness, soundness, multiple origins, no heuristics, barriers, ordering, source spans, no guest execution and other clauses remain. |
| [D085-F′ / #370](https://github.com/guillermomolina/protos/issues/370) | The canonical Package Tool owns project identity; candidate folders cannot infer or discover projects | **Unchanged.** A buffer is not a `ProjectBinding`; sources outside local-only navigation continue to require full validated authority. |
| [D089-A″ / #376](https://github.com/guillermomolina/protos/issues/376) | Valid generated `protos.project`, compatible lock and exact metadata freshness are required for project acquisition | **Unchanged.** Missing/stale metadata still fails the **project binding** closed, without generation, repair or search. |
| [D124-B′ / #491](https://github.com/guillermomolina/protos/issues/491) | `textDocument/references` requires exact singleton seed and sound may-reference subset in canonical source authority | **Unchanged.** Loose references remain empty; F12 doesn't authorize rename or references. |
| Core `spec/` | Protos lexical context objects, mutable slots, closures by reference, dynamic receiver fallback, module isolation, contextual `for` | **Unchanged.** This is a public editor/tooling policy change, not a language semantic amendment. |

**GITHUB021 invariant/delta consistency:** PASS **under the exact approval above**. D209-C and D110-B′ must retain their historical original records; this D258 record is the explicit later scoped amendment, not a silent retroactive rewrite or a claimed authorization for other features.

## 3. Why the precise document-local boundary is sound

The existing stack already separates editor custody from canonical project membership:

- `ProtosTextDocumentService.definition()` currently checks `navigationSourceAuthority.test(uri)` before asking the session for a proof, then repeats that check before publishing.
- `ProtosStaticAnalysisSession.definitionCurrent()` works on the exact open immutable snapshot and `isCurrent()` ensures the computed proof matches the current document. No package binding is needed by the parser/proof engine.
- `ProtosStaticDefinitions` walks the parsed surface syntax with query-local parameter-origin facts, left-to-right default-parameter installation, separate closure activations and opaque-invocation invalidation. It returns no inferred member/receiver/module bindings.
- `ProtosWorkspaceService.ownsCanonicalSourceUri()` validates exact project-source ownership independently and **must not** be modified or bypassed for References or Workspace Symbols.

Example adversarial case:

```protos
f: (value) => {
    sink(value)
    value
}
```

The first `value` can still be proven before the opaque `sink` invocation; the later read is unproven because the invocation might modify or remove a captured context slot. D258 must **not** erase that barrier. An assignment target `value = 2` is also not currently claimed as a D110 reference origin in generation 1, and a `value:` named argument label is not a read.

A logical alias `file:///workspace/./Main.protos` does not acquire canonical project membership. If the **exact alias URI** is independently open, it can receive only local answers anchored to that alias snapshot; it cannot borrow source/project identity from `file:///workspace/Main.protos`. A closed/stale URI cannot yield a result.

Current Protos grammar D131 removed special `match` binders/aliases. Some historical D110 and LSP guide wording predates that removal. They are **not** an accepted generation-one navigation capability; I168 must correct the maintained guide, not resurrect matching grammar.

## 4. Prior art, including the Truffle implementation space

**Relevant comparators / evidence (2026-11-18 survey):**

- **GraalPy**, reference `oracle/graalpython@3289197e6373d8424d8c84d13187cc44ecd3a31d`: [parser `SSTNode`](https://github.com/oracle/graalpython/blob/3289197e6373d8424d8c84d13187cc44ecd3a31d/graalpython/com.oracle.graal.python.pegparser/src/com/oracle/graal/python/pegparser/sst/SSTNode.java) preserves the parser-owned [`SourceRange`](https://github.com/oracle/graalpython/blob/3289197e6373d8424d8c84d13187cc44ecd3a31d/graalpython/com.oracle.graal.python.pegparser/src/com/oracle/graal/python/pegparser/tokenizer/SourceRange.java); [`ParserCallbacksImpl`](https://github.com/oracle/graalpython/blob/3289197e6373d8424d8c84d13187cc44ecd3a31d/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/ParserCallbacksImpl.java) maps them to precise Truffle `SourceSection` with boundary checks. The transferable precedent is **source-owned precise spans**, not arbitrary runtime-assisted navigation; Protos already has this mechanism in its parser and `ProtosLspSourcePositions`.
- **TruffleRuby** tracks parser-AST source ranges and translates Ruby AST into Truffle AST; **GraalJS** carries `SourceSection` through instrumentable language nodes. Both are instructive for source location/provenance, but neither authorizes a guessed Protos lexical origin or requires changes to the Protos interpreter. Relevant public code: [GraalJS `JavaScriptNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/JavaScriptNode.java); [TruffleRuby source](https://github.com/oracle/truffleruby).
- **GraalVM Truffle generic LSP** can use runtime/instrumentation data and variable read/write tags for navigation ([GraalVM language server documentation](https://www.graalvm.org/jdk23/tools/lsp/), [`ReadVariableTag`](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/instrumentation/StandardTags.ReadVariableTag.html)). The dynamic evaluation path is **not adopted**: D110/D209 forbid guest execution for static editor requests.
- **TypeScript/tsserver** inferred projects work for files lacking `tsconfig.json`, but inferred package/project identity contradicts D085/D089.
- **Jedi/Pyright** offer parser/static analysis of unsaved open source without necessarily requiring a canonical build project; their precise source-buffer custody is the relevant analogue, not Python name-resolution semantics.
- **clangd** offers fallback compile commands without a compile database; Protos rejects such inferred semantic environments when claiming a source definition.
- **rust-analyzer** supports explicit `rust-project.json` project descriptions; project identity remains separate from loose source-text custody.
- **Lua Language Server** uses fallback scopes for otherwise-unowned files; Protos adopts **only** a much narrower single-document exact-proof domain.
- Standard **LSP** imposes no ProjectBinding metadata prerequisite on `textDocument/definition`, and accepts an exact location list or empty/no-result answer.

The survey covers dynamic/prototype-oriented, static/inferred-project and metadata-backed/static-source approaches. It does **not** justify copying runtime instrumentation/JIT machinery for a purely static editor request.

## 5. Candidates assessed

- **A — retain D209 project gate, improve Package Tool provisioning.** Preserves original gate but forces toolchain/project setup to navigate two positions already proved inside one buffer. Still desirable independently for genuine project navigation.
- **B — allow complete current-snapshot D110 proof without ProjectBinding, strictly same URI/snapshot. SELECTED.** Smallest sufficient change, no new semantic engine, no new project/ownership mechanism, no unrelated runtime cost.
- **C — inferred workspace/folder, filesystem/URI search or fuzzy best-effort definitions. REJECTED.** Violates D085/D089 and D110; LSP locations carry no uncertainty/confidence marker.
- **D — defer.** Safe but leaves ordinary loose-file F12 unusable and no public metadata generator available.
- **E — extension-only alternate F12 command. REJECTED.** Duplicates semantic authority in a thin VS Code client and evades standard `textDocument/definition`.

## 6. Mandatory twelve-axis scoring

Scores 1–5; confidence H=high, M=medium, L=low. B is scored under the explicitly approved source-authority amendment, not under unrevised D209. Comparisons are advisory; correctness objections cannot be compensated by points.

| Criterion | A | **B** | C | D |
| --- | --- | --- | --- | --- |
| 1. Correctness / invariants | 5/H unchanged | **4/M** exact proof, scoped amendment | 1/H heuristic locations | 5/H unchanged |
| 2. Protos philosophy | 3/H project-heavy | **5/H** existing mechanism | 1/H invented inference | 3/H little UX value |
| 3. Present-need proportionality | 2/H metadata overhead | **5/H** requested local proof only | 1/M discovery/index overhead | 1/H unmet need |
| 4. Incremental growth | 3/M provisioning prerequisite | **5/H** local now, exact projects later | 2/M identity entanglement | 3/H later possible |
| 5. Future-option resilience | 4/M canonical intact | **5/M** boundaries remain separate | 2/M sticky inferred identity | 4/M unchanged |
| 6. Scalability | 5/H bounded canonical | **5/M** one snapshot on demand | 2/M wider scan cost | 5/H no work |
| 7. Conceptual simplicity | 4/H single authority | **4/H** two explicit scopes | 1/H inferred scopes | 5/H no change |
| 8. Portability | 5/H editor neutral | **5/H** opaque URI+LSP | 3/M host discovery | 5/H unchanged |
| 9. Runtime/resource cost | 5/H no added runtime | **5/H** no guest cost | 2/L scan/caches | 5/H none |
| 10. Failure / operability | 5/H fail closed | **5/M** complete-or-empty | 1/H silent wrong locations | 5/H predictable empty |
| 11. Deferral / reversibility | 2/H public metadata absent | **5/M** local reversible | 2/M compatibility debt | 3/M postponed |
| 12. Evidence maturity | 5/H known contract | **4/H** D110 tested; unbound E2E pending | 3/L unlike Protos | 5/H known empty |

**Non-compensating red flag:** C's wrong-definition risk disqualifies it regardless of resource scores. B's only meaningful conceptual cost is a second, deliberately smaller **document-local source-authority domain** distinct from ProjectBinding; preventing its accidental expansion is a required acceptance gate.

## 7. Adversarial/future review

- **Source mutation and escaped contexts:** never return a binding origin after an opaque invocation if D110 facts no longer prove it. Existing D110 proof logic remains unchanged.
- **Snapshots, close and re-open:** `didOpen`/`didChange` replace full documents; `didClose` evicts; `isCurrent` must be rechecked when publishing. This decision preserves the established snapshot-value currentness rule and **does not** introduce a stronger monotonic version-counter or session-epoch model.
- **Remote/virtual URIs, case, `./` aliases:** operate only on the exact opened URI, never normalize or merge identities; cross-file authorization remains canonical.
- **Large files / workspaces / many Actors:** local proof is on demand and single-buffer; no project indexing, guest Truffle contexts, Actor inspection, parallel worker service or global background state.
- **Different OS, Native image, JVM, alternative VM/backend:** no platform-specific authority, no dependence on hot-path shape or Truffle execution nodes. Source span and protocol remain portable.
- **Future import/member/rename/References:** new complete source-origin proof and proper canonical authority must be decided/implemented separately; no implicit generalization from local F12. Standard rename cannot treat D124's incomplete may-reference set as complete.
- **Regret scenario:** future navigation across multiple virtual documents might require separate authoritative source identity. Escape path: extend a recognized canonical provider through an independently approved decision; do not turn URI similarity into source identity.
- **Cost of not building now:** existing D110 and exact snapshots could support useful F12 today, but users without a public metadata generator cannot access it; no proven architecture rewrite is required if deferred.
- **Cost of building extra capability now:** guessed method/member targets, inferred projects or a new Java resolver would require unverified semantic rules and ongoing synchronization. None is justified by this work.

**Strongest objection to B:** an editor may mistake a same-buffer result for canonical project identity and expand it to ambiguous source targets. Defense: same exact URI+snapshot for all locations, independent ProjectBinding predicate unchanged for other LSP handlers, explicit negative regression tests and no source scanning.

## 8. Consequences, boundaries and handoff

**Implementation owner:** [I168 #1135](https://github.com/guillermomolina/protos/issues/1135), one cohesive slice. Minimal production modification belongs in existing `ProtosTextDocumentService.definition()`; preserve `ProtosStaticAnalysisSession.definitionCurrent()` and `ProtosStaticDefinitions` proof behavior. No new Java class, Truffle node, guest call, ProjectBinding source or VS Code JavaScript semantic resolver. Edit existing LSP tests for bound/unbound/no-workspace/alias/unsaved changes/closed documents and real JSON-RPC; maintain `docs/guide/tools/language-server.md` with accurate D131 grammar. The human executes focal/integrated tests, `git diff --check`, `make test`/applicable `make check`, commits/pushes and reports exact final revision. Real VS Code F12 acceptance with pinned Native/extension identities is required before closure.

**No product implementation or I168 tests are claimed by this ratification.** The owner's statement that all tests passed and `git diff --check` was clean refers to the product baseline preceding I168, not to a feature that was not yet edited.

## 9. Approval record

The decision packet was presented in the originating conversation and explicitly identified:
- B's single-document current-snapshot authority;
- the **express reopening of D209-C's Definition project gate**;
- the **narrow change to D110-B′ clauses 11–12 source authority**;
- unchanged D085/D089/D124 and no general slot/import/member navigation;
- full twelve-axis comparison, alternatives, counterexamples, future-regret/escape and the strongest argument against B.

The project owner answered on 2026-11-18: **“si apruebo todo eso que dices.”** This constitutes approval of that exact scoped D258-B package, **not** of unrelated semantic or tooling changes. This durable ratification documents the approved delta without altering predecessor history.

```text
WORK_ITEM=D258
DECISION=B
OWNER_APPROVAL=PASS_2026_11_18_ORIGINATING_CHAT
APPROVAL_SCOPE=D209_C_DEFINITION_ONLY_PLUS_D110_11_12_LOCAL_SOURCE_AUTHORITY
PREDECESSOR_INVARIANT_CONSISTENCY=PASS
PROJECT_BINDING_AUTHORITY=UNCHANGED
REFERENCES_AND_WORKSPACE_SYMBOLS_AUTHORITY=UNCHANGED
LOCAL_RESULT=COMPLETE_D110_SAME_EXACT_URI_AND_CURRENT_SNAPSHOT_ONLY
GUEST_SEMANTICS=UNCHANGED
GUEST_EXECUTION_FOR_LSP=PROHIBITED
NEW_JAVA_INFRASTRUCTURE=NOT_REQUIRED
IMPLEMENTATION_OWNER=I168_ISSUE_1135
PROTOS_RESEARCH_REVISION=91b92332f6870fd9f0903c829c2728a644d0dca4
I168_PRODUCT_REVISION=NOT_YET_PUBLISHED
I168_TESTS=NOT_YET_RUN
REAL_VSCODE_F12_ACCEPTANCE=PENDING
```

---

## Subsequent tooling-contract amendment — 2026-11-18

The preceding D258-B ratification remains a correct **historical record of what was approved at that time**: exact open-buffer custody as same-document source authority, without project inference and with References/Workspace Symbols still ProjectBinding-gated. Its original complete-exact-D110-proof wording was not a grant for candidate navigation.

**Later, independently approved policy:** following the implementation and editor acceptance of I168, the owner explicitly approved an additional, narrowly scoped change to D110 through the [D110 post-ratification lexical navigation amendment](./D110_STATIC_GO_TO_DEFINITION_PROOF_BOUNDARY_IDENTITY_AND_AMBIGUITY_CONTRACT.md#post-ratification-amendment--lexical-declaration-navigation-for-same-document-f12-2026-11-18). Standard F12 may now prefer an exact binding proof and otherwise return an **AST-backed lexical declaration navigation target**, which is not guaranteed to be an exact dynamically selected runtime slot. This later decision supersedes the older *unconditional proof requirement for Definition results only*. It does **not** supersede any of D258's URI, open-snapshot, source ownership, ProjectBinding or no-runtime constraints.

**Owner approval provenance:** the project owner said **“pues apruebo, documenta y cierra”** on 2026-11-18 in direct response to the explicitly identified pending D110/D258 exact-proof versus lexical-navigation discrepancy. Source implementation: [`guillermomolina/protos@01c8566e6481501b051ee67d475e1d67accadb4a`](https://github.com/guillermomolina/protos/commit/01c8566e6481501b051ee67d475e1d67accadb4a). Formal implementation owner: [I168 #1135](https://github.com/guillermomolina/protos/issues/1135). No change to Protos guest semantics, runtime/JIT or the earlier project-source authority ratification.

