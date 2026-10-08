# D194 — Comparative investigation of canonical Protos lint policy

**Status: COMPLETED INVESTIGATION / OWNER DECISION REQUIRED. No option is ratified.**

Date: 2026-10-08. Formal authority: [D194/#842](https://github.com/guillermomolina/protos/issues/842), triggered by [LM012/#671](https://github.com/guillermomolina/protos/issues/671).

```text
PROTOS_REFERENCE_REVISION=3bb1278d91ee5cea98031462be2a5c4dd3c89019
VSCODE_EXTENSION_REFERENCE_REVISION=c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a
LM012_A0_PUBLICATION=739ece903a64bcd01d32229f61153d449510c207
TYPE=INVESTIGATION
RECOMMENDATION=OPTION_B_PENDING_OWNER_APPROVAL
PUBLIC_CONTRACT_SELECTED=NO
DESIGN_RATIFIED=NO
IMPLEMENTATION_AUTHORIZED=NO
NEW_PLAT_ARCHITECTURE_SELECTED=NO
TESTS_EXECUTED=NO
BUILDS_EXECUTED=NO
PRODUCT_CODE_MODIFIED=NO
```

The [LM012-A0 source authority audit](../LM012/LM012_A0_DIAGNOSTICS_LINT_AUTHORITY_AUDIT.md) establishes the exact current technical boundary. This packet researches the *public tooling policy* that must not be inferred from that technical feasibility. GitHub issue and source content was inspected read-only, and comparative external primary documentation was consulted. No runnable experiment or validation is claimed.

## 1. Decision and motivation

**Question:** Should Protos initially expose any new lint diagnostics beyond already-owned parser/compiler diagnostics; if so, which rules are justified by *proof* from source, at what severity and default enablement, with which stable identities and user-visible correction semantics? How should policy remain portable between the Protos toolchain, VS Code/LSP and a later explicitly chosen CLI/CI contract?

The public behavior matters: adding a rule to the default suite, assigning Error rather than Hint/Warning, declaring source text "unused", specifying suppression/exit semantics, or labeling a source rewrite safe can change users' diagnostics, automated gates, editor behavior and workflow. None was selected merely by opening LM012.

The first implementation must **not** reimplement already canonical parser errors, D096 matching coverage design, LM011 formatter style, language/runtime semantics, or an editor-local semantic model. A lint decision is not an excuse to design a new compiler, always-on index, global registry or runtime supervisor. The owner must explicitly ratify a particular policy before LM012-B.

## 2. Verified Protos constraints and counterexamples

- Real parser + `ParseError` + `SourceSpan` and `ProtosStaticAnalysisCore` already produce immutable `Parsed/Failed` results. LM009-G1/#360 publishes parser errors with versioned `textDocument/publishDiagnostics`; on success and close it clears them. They are **not** a new lint rule.
- PLAT024/#342 fixes an editor-neutral static core, client-session-owned dedicated LSP stdio process, thin TS `LanguageClient`, exact current source buffers and zero LSP static-analysis cost for ordinary Protos runtime executions. `ProtosSourceLayoutView` can be acquired on demand for source/token/trivia facts and is not a source-wide mandatory CST.
- D110/D124 exact source definition/referrer proofs deliberately refuse guessing for arbitrary names, receivers, members, cross-module or workspace names.
- The language is dynamically typed and object/prototype/slot based; ordinary lookup, delegation, mutable captured lexical contexts, dynamic calls and reflective operations invalidate naïve "unused variable", "undefined member", "unreachable expression" and "duplicate binding" heuristics. `=` may select an existing binding at runtime, not just a lexically adjacent `:` occurrence.
- `spec/PROTOS_GRAMMAR.md` assigns newline/semicolon, comment consumption, Unicode, multiple literal spellings and triple-double-quoted String whitespace semantics. A quick-fix that deletes apparent whitespace *inside* a String, changes semicolon/newline structure, or rewrites a comment-boundary newline can change meaning.
- `spec/semantics/MATCHING.md` and D096/#390 leave open how/if effectful custom matchers and guards permit static coverage or redundancy diagnostics. Do **not** classify an arm as unreachable because it follows an apparent textual default unless the exact semantics are already approved.
- D183/D184 and LM011 own canonical formatting/CLI format behavior. A source differing from formatter output is not automatically a lint error and `format` must not quietly become `lint --fix`.
- `ProtosCli` has no ratified lint/check command, exit contract or package-wide lint policy. The selected `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md` favors a common driver and bounded bundled-tool policy when naturally expressible as Protos code, rather than inventing a permanent Java `LintPolicy` facility. Concrete promoted official tooling would be `TOOLxxx`, not implicitly owned by LSP.
- The standalone VS Code extension already launches the external standard LSP server; a diagnostic or future code-action feature must be server-owned and should not replicate parsing/analysis in TypeScript.

**Falsification examples** (tests to implement only *after* approval): (a) spaces that look trailing but are interior to a triple-double-quoted String; (b) a bare read after a closure invocation that creates/mutates a captured slot; (c) a member found through a dynamic delegation parent; (d) a reflected or dynamically acquired name absent from local text; (e) an effectful custom matcher/guard; (f) a document changed between computation of a fix and application; (g) CRLF plus supplementary-plane Unicode before the range; (h) invalid/incomplete open buffer being edited; (i) content from an untrusted/out-of-domain URI and (j) different package root bindings that must never be merged based on a coincidentally equal filename.

## 3. Comparative study: materially different systems

| System / primary authority | Approach | Learning for Protos | Inadmissible transfer |
| --- | --- | --- | --- |
| [ESLint: rule configuration](https://eslint.org/docs/latest/use/configure/rules), [core concepts](https://eslint.org/docs/latest/use/core-concepts) | Rule-oriented JavaScript lint with explicit `off/warn/error`, fixable rules and suggestions | Rules and diagnostic severity must be separately identified; user choice about enabling/configuration is public behavior. Explicit suggestions need not silently rewrite source | JavaScript semantic assumptions and a large plugin/config institution are not justified for Protos today |
| [Rust Clippy](https://doc.rust-lang.org/clippy/index.html) / [configuration](https://doc.rust-lang.org/clippy/configuration.html) | Compiler-informed, lint-group levels such as correctness/style/pedantic, opt-in stronger checks | Distinguish highly sound checks from subjective style/pedantic diagnostics; avoid making every optional rule a default error | Rust's typed compiler IR/closed binding proofs cannot be projected onto open dynamic Protos slots |
| [Ruff linter](https://docs.astral.sh/ruff/linter/) / [Ruff formatter](https://docs.astral.sh/ruff/formatter/) | Rule selection and explicit safe/unsafe fix boundary; separate formatter | Lint versus formatter is a real authority distinction; fix safety requires particular rule- and source-context proof; unsafe fixes must not become automatic | A huge built-in catalogue and wholesale Python rule semantics would overfit the wrong language |
| [Go vet](https://pkg.go.dev/cmd/vet) | Curated suspicious-construct static analyzer rather than every conceivable style warning | Small, targeted checks with clear evidence can be valuable independently from formatting | Go's static binding/type guarantees and analyzer reach are not evidence of Protos semantic reach |
| [gopls analyzers](https://go.dev/gopls/analyzers) / [diagnostics](https://go.dev/gopls/features/diagnostics) | Language-server diagnostics with source analyzers and editor actions | One canonical analysis core can project into editor reports; diagnostics/actions can have explicit analyzer provenance | Editor visibility is not sufficient proof of safety, and LSP semantics do not authorize new Protos rules |
| [LSP 3.18 specification](https://github.com/microsoft/language-server-protocol/blob/gh-pages/_specifications/lsp/3.18/specification.md) | Protocol interoperability boundary: diagnostic ranges/codes/severities and code-action edits/commands | Standard transport can project approved rule identity, source and severity; fixes need exact snapshot/version consistency and capability handling | LSP is *not* a language semantic authority; a `CodeAction` field does not prove a rewrite valid |

This spans compiler-adjacent lint (Clippy/Go vet), configurable rule engines (ESLint/Ruff), and editor-protocol analyzers (gopls/LSP). Other plausible sources (Pyright, Pylint, TypeScript/TS Server, Sonar) largely add breadth in typed inference, project-centric heuristics or configurable warning taxonomies, without overturning the key Protos-specific open-world proof boundary. They are not used as substitutes for the actual Protos spec.

## 4. Candidate set and elimination

- **A — Status quo / defer lint:** keep parser errors only, no new rule IDs, CLI, code actions or service work. This is the smallest cost and zero false-positive risk today, but leaves LM012's useful, distinct objective unfulfilled. A remains viable if no first useful rule survives source-context proof.
- **B — Conservative opt-in source-local lint (recommended, not selected):** introduce *only owner-approved and statically proven* rules, opt-in initially; stable diagnostic code and source provenance; no automatic source edits in the baseline. Reuse immutable snapshots and static core, defer CLI/check exit policy and a full configuration/plugin framework. One first rule must be specified exactly, including exclusions (e.g. a proposed whitespace hygiene rule excluding literal/comment contents), before code begins. Advantage: exercises the correct extension seam without infecting the runtime or surprising all users. Cost: an opt-in feature may initially have very little visibility.
- **C — Small default conservative core + opt-in extras:** default only proven high-confidence, low-noise rules; explicit stable identities; defer broad customization and make quick fixes explicit-only where proven. Benefit: visible IDE value without configuration. Risk: "obvious" default warnings can be incorrect in a dynamic/interactive language and changing default severity is a compatibility event. No concrete default rule has been justified yet.
- **D — Extensive configurable plugin-style lint engine now:** include rule-group hierarchy, severity overrides, suppressions, per-project configuration, CLI/CI modes and code actions before first rule. Benefits users of large mature static-analysis ecosystems, but imposes durable policy surface, parser/source integration, maintenance/security/versioning complexity with no demonstrated Protos workload today. **Overengineering red flag**; keep this as a considered but not recommended alternative.
- **E — Broad heuristic semantic lint:** infer dead/unused names, undefined properties, likely invalid calls, match exhaustiveness or concurrency model problems from local syntax and guessed workspace identities. **Eliminated as a correctness candidate:** current exact proof authorities do not justify those claims; the errors can be semantically false due to runtime mutation, open dispatch/reflectivity, and effectful matching. Reconsider only after independent normative/analysis evidence changes the proof boundary, not on the strength of naming heuristics.

No hybrid requiring a mandatory hosted analyzer/always-on index is justified; the real hybrid possibility is **B → C** after a proven rule set and explicit owner authorization. This transition is incremental, not a preinstalled D engine.

## 5. GITHUB010 scoring — surviving candidates A–D

Scores are 1 (poor) to 5 (strong). H/M/L is confidence in each judgement. These are *relative estimates*, not runtime measurements or design ratification.

| Dimension | A: defer | B: opt-in/proven | C: default+opt-in | D: full framework |
| --- | --- | --- | --- | --- |
| 1. Correctness / invariant preservation | **5/H** — no new false claims | **5/M** — proof-first and explicit enrollment, rule-specific validation still needed | **4/M** — soundness possible but default expectations raise error risk | **3/M** — broad integration and optional policy interactions enlarge risk |
| 2. Protos philosophical alignment | **4/H** — no new institutions, but no useful growth | **5/H** — mechanisms before lint institution, keep ordinary runtime untouched | **4/M** — small core plausible but premature normative-seeming defaults | **2/H** — many special policy layers without actual demand |
| 3. Pay only for present needs | **5/H** — zero cost | **5/M** — on-demand for opted-in users, few rules | **4/M** — all LSP users pay analysis cost | **1/H** — large upfront user/developer/config burden |
| 4. Incremental growth | **3/M** — no initial extension seam to exercise | **5/M** — grow one vetted rule at a time | **4/M** — add rules but default migration is visible | **3/M** — extensible, yet premature framework constrains evolution |
| 5. Future option resilience | **4/M** — postpones decisions but also defers evidence | **5/M** — stable codes + neutral evidence allow CLI/editor evolution | **4/M** — similar seam, early default obligations | **3/M** — locks in version/config/plugin contract early |
| 6. Scalability | **5/H** — no analysis | **5/M** — bounded per-snapshot checks, no new global state | **4/M** — bounded if default checks stay local | **3/L** — larger rules and configuration can spur global analysis/caching |
| 7. Conceptual simplicity | **5/H** — parser only | **5/M** — diagnostics, no new language category | **4/M** — small default rule profile adds policy | **1/H** — configuration/suppression/plugin semantics explode |
| 8. Portability / implementation freedom | **5/H** — no new coupling | **5/M** — editor-neutral rules and ranges, optional protocol mapping | **5/M** — same if careful | **3/M** — host/plugin/config and editor assumptions probable |
| 9. Runtime and resource cost | **5/H** — unchanged | **5/M** — only active opted-in tooling pays local costs | **4/M** — default LSP lint work, ordinary runtime unchanged | **2/M** — possible index, startup and memory overhead |
| 10. Failure and operability | **5/H** — preexisting parser path | **4/M** — new rules isolated/fail closed, limited diagnostics | **4/M** — default noise/ordering needs maintenance | **2/M** — rule interactions, partial fixes and configuration conflicts |
| 11. Deferral/reversibility/migration | **3/M** — feature eventually must add IDs/provider | **5/M** — B→C bounded enablement change, code identity preserved | **3/M** — reverting defaults creates user surprise | **2/M** — public framework ABI/config harder to undo |
| 12. Evidence maturity/implementation risk | **5/H** — observed published baseline | **4/M** — analogous precedent and existing source layout, first rule untested | **3/M** — no proven Protos default rule today | **2/M** — no evidence Protos needs full framework |

**Qualitative gate:** A is safest but risks *underengineering LM012's concrete editor/tooling objective* by delivering nothing. D fails the present-need/complexity test despite apparent extensibility. C might be a better user experience once at least one concrete default rule is proven. B presently best balances bounded utility, reversibility and proof obligations. E violates the hard soundness boundary and is not scored as a surviving, approvable design.

## 6. Adversarial and future stress analysis

**False confidence in source-level proof.** A candidate whitespace rule should distinguish trailing horizontal whitespace after complete code tokens from whitespace inside lexical strings, comment forms and continuation constructs. Even a harmless-looking automatic removal must demonstrate unchanged parse and semantic values. An effect-free lexical change is *not* assumed simply because it matches a regex. On invalid source, token/trivia boundaries may be incomplete; fail closed by default instead of inventing a partial parse policy.

**Dynamic soundness.** A closure can mutate captured lexical context; a delegated member may be located dynamically; method invocation can execute effects; `pattern.match` and guards can be effectful. Source-only heuristics cannot prove negative name resolution or match exhaustiveness. If a future optimizer can prove a concrete fact, proof artifact and exact validity horizon must be exposed to an editor-neutral static layer; an optimizer's partial-evaluation assumption is not automatically a user-visible warning.

**Snapshot/offset races.** Each diagnostic must be bound to immutable source text and version. Reparse/close must clear invalidated outcomes; out-of-date asynchronous results must not overwrite a newer buffer. An explicit fix must be tied to the exact source and validate applicability, especially on rapid edits. UTF-16 offsets, supplementary characters, CRLF and non-file URIs must follow canonical source-position utilities.

**Scale and architecture.** 100k-line documents, multiple roots, simultaneous editors and project churn stress recomputation and isolation. B can remain local and request-scoped within PLAT024; a project-wide unused-name or dependency check would not. No new mandatory Truffle context, Actor, Task, Process, thread, scheduler, daemon or global registry for a `hello world`. Only the active LSP/explicit lint tool should pay analyzer work. Any actual future shared cache/index introduces an independently evaluated lifetime/ownership issue under PLATxxx; do not precreate it.

**Backend and toolchain evolution.** Pure source/snapshot/AST diagnostics survive Truffle Bytecode DSL changes, Native Image packaging and possible non-Truffle backends more readily than optimizer/compiler-IR-dependent guesses. On-demand layout introspection has a different cost profile than default runtime loading. Cross-platform source encodings/line endings and package/source identities must remain exact; host filesystem enumeration is not semantic project authority.

**Future closed-world demand.** If Protos later acquires a proven typed/closed analysis island or dedicated matching-coverage semantics, its canonical proof can supply a rule without rewriting B's basic diagnostic projection. Rules that require context-sensitive dependency graph evidence would need an actual new backend and approved per-project indexing discipline; "extensible" should mean a migration path, not an already running index.

**Fix conflicts and safety.** Explicit diagnostics and source transformations are separate products. Two proposed edits can overlap, and an edit valid for the original snapshot can become invalid after another action. The server must not claim safe `fixAll`, automatic save rewriting, or `CodeActionKind.QuickFix` correctness until a per-rule proof, conflict ordering and staleness contract exist. An unsafe suggestion may be presented as explanatory text only if separately authorized.

## 7. Pay-for-what-you-need and regret gates

**Smallest sufficient present solution:** once the owner approves one useful strictly source-proven rule, implement an editor-neutral, snapshot-in/diagnostics-out evaluator for that rule alone; stable ID and exact range; optional activation; no CLI exit behavior or automatic edits. Existing LM009-G diagnostic transport remains the only LSP publisher.

**Who pays today?** In A nobody pays, but nobody gets lint. In B only a user who explicitly opts into a rule and the active analysis request pays. In C every participating IDE source buffer pays default check processing. In D even users with no rule benefit pay configuration, policy, maintenance and possibly parse/index costs. Neither B nor C requires ordinary Protos program execution to pay any tooling cost.

**What is omitted and exact future rewrite cost?** Under B, adding more independently verified rules is primarily incremental additions to rule collection/evaluation and documentation. Enabling an approved default changes policy and UX but not the data model. Adding a public CLI/CI adapter requires proper argument, result, exit and toolchain admission policy, not a forced redesign of parser-based rules if diagnostics stay editor-neutral. Suppression/configuration would add persistent profile parsing/identity policy; safe fixes require a distinct edit/proof structure. More sophisticated cross-file analysis needs new project-scoped identity and possibly index lifetime; no existing evidence establishes that cost as high enough to justify implementing it now. A would need to introduce the same neutral rule-result seam later; modest and reversible, but postpones validation of the seam.

**Regret if B proves inadequate:** future users may expect out-of-box warnings (C) or sophisticated project-specific checks (D). The escape is to adopt proven high-signal rules by default with explicit release notes, or to add a separately approved configuration/plugin/tool layer when actual demand establishes costs. Retain stable rule codes and avoid misleading "absence means proven safe" claims.

**Regret if C/D now:** default noise and auto-corrections may become a long-lived compatibility expectation even after discovered false positives. A large rule/config API is costly to withdraw. The escape is a breaking change, suppression migrations, or permanent compatibility shims; that is a real rather than speculative present burden.

**Strongest argument against recommendation B:** opt-in-only with no preapproved high-value rule may result in effectively no accessible lint at all; the first user cannot discover value without configuration. If evidence demonstrates one genuinely safe universally valuable warning, C's minimal default is simpler for users. The owner should consider that possibility when selecting the first rule and default status, not preselect an entire warning taxonomy.

## 8. Recommended exact decision packet — **proposal, not ratification**

Recommend **B: source-local conservative opt-in lint**, contingent on the owner explicitly accepting the following public contract or replacing it with an exact alternative:

1. Existing grammar/parser diagnostics retain their authority and existing LM009-G published behavior. A lint rule must contribute genuinely distinct value.
2. An initial lint rule is eligible only with a documented proof of the reported source fact, explicit positive/negative examples, stable source span and semantic counterexamples. A *potential* trailing-whitespace hygiene check is a **research example**, not an approved initial rule.
3. Each approved rule has stable identifier, description, provenance and single selected level; default opt-in unless explicitly ratified otherwise. No global lint disable/allow pragma or plugin API is introduced implicitly.
4. Editor-neutral analysis over exact immutable snapshots produces no speculative runtime-based claims; a superseded, unparseable or out-of-domain source does not create guessed semantic lint.
5. LSP projects only the approved diagnostic record; parser and lint outcomes are not duplicated, must clear on change/close and must be revision-bound. Editors remain thin.
6. Baseline lint emits **no source edits**. A future rule-specific quick fix requires an independently proven semantics-preserving transformation, explicit opt-in/action, exact text/version checks and non-overlap policy before code action exposure.
7. Do **not** select the spelling `protos lint` or `protos check`, CI fail level, CLI stdout/stderr, status codes, project discovery, new TOOL identifier, bundled tool host/guest bridge, or D096 matching coverage here. Those surfaces need their actual owning contract when implementation reaches them.
8. Reuse PLAT024 and existing on-demand source layout. New PLATxxx only for demonstrated new durable lifecycle/index/hosting requirements.

**Owner decision still required:** approve/modify/reject B vs A/C/D, select at least one exactly specified initial useful rule (or explicitly defer it), and settle that rule's enabled-by-default status, diagnostic level/ID and safe-fix policy. Until then the correct state is `status:needs-decision` for D194 and LM012 implementation blocked. A design packet being published is *not* approval. If the selected public contract expands into CLI/CI or changes a durable platform boundary, follow independent governance; no ratification by proximity.

## 9. Implementation path after an explicit approval (not authorized now)

- **D194** owner approval, bounded ratification publication, re-read exact record and live issue state; only then close D194 if postconditions pass.
- **LM012-B** single coherent baseline rule/evaluator and focused proof/negative tests, avoiding many micro-slices; owner decides if a separately promoted `TOOLxxx` is actually necessary.
- **LM012-C** only after public CLI/CI behavior and toolchain policy are selected; no ad hoc host `LintPolicy`.
- **LM012-D** aggregate already-owned parser errors and new approved results once, project to LSP with exact versions and clear-on-close; no TypeScript semantic implementation.
- **LM012-E** explicit safe code actions only after independent proof and selected public edit contract.
- **LM012-F** real toolchain/editor/CLI release integration and final required regression/validation evidence; human executes builds/tests/product Git publication.

One D194 Issue is justified as an independently blocking substantive decision. No LM012-A/B/C subissue is justified merely for patch slicing. Native D194 -> LM012 parent relation and exact blocked-by edge should be established through native GitHub relations when a capable tool is available; textual mention does not substitute for either.

## 10. Audit coverage and limitations

Materially inspected authoritative files/records are enumerated in the companion [LM012-A0 audit](../LM012/LM012_A0_DIAGNOSTICS_LINT_AUTHORITY_AUDIT.md), including root and scoped AGENTS, DESIGN/REFERENCE/COORDINATION/IMPLEMENTATION policies, grammar/language/object/execution/callable/matching specs, parser/static analysis/LSP/CLI/test source, PLAT024, D096, LM009-G, LM011 and D183, plus the exact standalone extension files. The decision-comparison sources are linked in §3. This research was performed from published HEAD and readable source/docs; no local instrumented parser behavior, editor-session performance, build, tests or `git diff --check` was executed. Candidate scores are reasoned estimates at the stated references, not benchmark measurements or a conclusion about unpublished concurrent changes.

```text
D194_RESEARCH=COMPLETE
COMPARATIVE_SCOPE=ESLINT_CLIPPY_RUFF_GO_VET_GOPLS_LSP
GITHUB010_DIMENSIONS=12
OWNER_APPROVAL=REQUIRED
RATIFICATION=NOT_STARTED
LM012_B_IMPLEMENTATION_AUTHORIZED=NO
NEXT=OWNER_EXACT_POLICY_DECISION
```
