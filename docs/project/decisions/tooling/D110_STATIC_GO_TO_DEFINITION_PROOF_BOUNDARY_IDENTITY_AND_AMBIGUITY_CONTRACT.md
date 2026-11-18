# D110 — Static go-to-definition proof boundary, identity and ambiguity contract

Status: **RATIFIED — Candidate B′ selected**

**Current effective contract (2026-11-18):** Candidate B′ remains the strict exact-origin tier; the owner also approved an [AST-backed lexical-navigation fallback](#post-ratification-amendment--lexical-declaration-navigation-for-same-document-f12-2026-11-18) for same-document F12. The original ratification text below is preserved as historical approval.

Allocated: **2026-09-12**

Explicit project-owner approval: **2026-09-12**

Decision issue: GitHub #435

Primary consumer: `LM009-G4` / GitHub #360

Predecessors: `D079`, `D082`, `D085`, `D089`, `D102`, `D106` — RATIFIED; `LM009-G3` — CLOSED

Nature: implementation-independent editor/tooling definition-resolution contract

Normative Protos language effect: **none**. D110 constrains only what the static language service may claim as a source definition; Core lookup, delegation, assignment, composition and module semantics remain owned by `spec/`.

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Decision boundary

LM009-G3 publishes exact project/source custody and workspace-symbol search, but a workspace symbol with the same spelling is not proof of a Protos definition.

Core lookup is deliberately dynamic and uniform:

- a bare read searches local slots of the current lexical context and its lexical parents before receiver fallback;
- object bodies are not lexical capture scopes;
- receiver fallback and explicit member reads use ordinary delegation;
- bare assignment selects an existing local destination under rules different from reads;
- composition can contribute local slots at runtime while direct declarations only reserve names against composition;
- `super` depends on dynamic `methodHome`;
- `import(specifier)` returns the real mutable module context;
- cyclic imports may expose partially initialized modules; and
- duplicate same-name source slot creations are valid independent occurrences.

D110 therefore answers one tooling question only:

> When may the static language service claim that a source reference has one or more source definitions?

## Selected contract — Candidate B′

Protos selects **complete finite static proof-set, singleton-first**.

A standard `textDocument/definition` response is successful only when the static analysis can prove a complete finite set of exact source-backed binding origins for the reference under the authoritative current source/project snapshot.

The baseline contract is:

1. Every returned target is an exact source binding origin that can semantically satisfy the reference under current Protos rules and the facts proven by the analysis.
2. The returned set must be **complete** for the analysis claim. A known subset is not a definition result if another runtime-selectable source origin may have been omitted.
3. A proven singleton returns one definition location.
4. A proven complete set with several targets may return all targets through the standard LSP multi-location result.
5. Multiple proven targets have equal semantic status. Ordering is deterministic canonical ordering only; it is not relevance ranking.
6. No definition-result cap may truncate a proven set. If an implementation cannot prove or return the complete set within its supported analysis/resource envelope, it returns no definition rather than a partial semantic claim.
7. Failure to prove a definition is an ordinary **no-result** outcome, not an Error and not permission to guess.
8. Generation 1 is deliberately **singleton-first / mechanically exact**. It may implement only cheap proof forms and decline harder references. Later analysis may monotonically add newly provable singleton or complete finite multi-target cases without changing D110.
9. D106 `workspace/symbol` is an exploratory/name-search facility and is never a fallback definition authority. Fuzzy, same-name, proximity, path, package, popularity, open-document, registration-order or other ranking heuristics do not establish definition identity.
10. Static definition requests execute no guest code, create no live Truffle Context, inspect no Actor/Process runtime state and perform no runtime-assisted dispatch.
11. Project/source identity comes from the already-ratified canonical ProjectBinding/package/source authorities. Filesystem ancestry, recursive scan, basename or editor-workspace guessing is forbidden.
12. The exact current open-document overlay may replace the canonical file content only for the exact canonical URI already owned by the project binding. Stale/current-snapshot mismatch or parse failure produces no definition.
13. The definition universe is **not limited to D079 document/workspace symbols**. Exact lexical binding origins such as Closure parameters and match binders/aliases/capture-interface bindings may be valid definition targets even though D079 intentionally does not publish them as document/workspace symbols.
14. A `:` slot creation is a binding origin. A `=` assignment is not a new definition; when its destination is statically provable, navigation targets the existing selected binding origin under the exact Protos assignment rules.
15. Bare-name references may resolve only when the analysis proves the complete result under the exact lexical-chain-before-`this` lookup order. A potential unresolved receiver fallback makes the reference unproven.
16. Explicit member reads, receiver fallback, delegation and `super` are supported only when the effective receiver/lookup origin and complete selected source-binding set are statically proven. Unknown receiver, dynamic parent/delegation state or unknown `methodHome` yields no definition.
17. Composition provenance may be used only when the complete effective contribution is statically proven and maps to exact explicit source origins. Dynamic/unknown composition does not authorize same-name guessing.
18. Import/module navigation may consume only an existing canonical static package/module resolution authority. Literal spelling by itself is not a filesystem path rule. Module-member proof must respect actual module identity, source ownership, initialization order and cyclic/partially initialized module semantics.
19. Prelude/Core/Standard-Library/dependency/runtime-only targets without an authorized exact source origin yield no source definition in this baseline. Expanding source authority is a separate decision, not an implicit G4 side effect.
20. Independent projects are never joined by same-name search. Multi-project definition identity remains partitioned by exact project/source authority.
21. Proof computation/index/storage is replaceable. Incremental, persistent, sharded or remote machinery may implement D110 as long as successful results preserve completeness and exact identity.
22. A future user-facing “possible implementors”, “candidate definitions” or heuristic-navigation feature may be added separately. It must not silently weaken `textDocument/definition`.

## Why complete proof rather than best effort

The most important separation is:

```text
workspace/symbol
    relaxed exploratory name search
    D106 ranking + global response cap

textDocument/definition
    semantic source-identity proof
    complete set or no result
```

This mirrors the useful distinction visible in dynamic object environments such as Self and Smalltalk: selector/name exploration is valuable, but it is not equivalent to the slot/method selected by actual lookup.

A wrong definition result creates more architectural debt than a missing one. Later references, rename and refactoring facilities need stable identity. D110 makes successful definition results trustworthy enough to become a future input to those tools without making those later features part of this decision.

## Generation-1 implementation envelope

D110 does not require G4 generation 1 to solve every finite static ambiguity.

The first implementation should preferentially support forms whose ownership is mechanical and exact, including where the existing source model can prove them:

- Closure-parameter references;
- match binder/alias/capture references within their exact binding scope;
- lexical/local slot reads whose selected earlier source binding is structurally proven under the real lexical-context rules;
- assignments whose existing destination binding is structurally proven;
- module-top-level local bindings where the same proof is exact; and
- other exact forms discovered during the implementation audit that require no new semantic or durable architectural decision.

Generation 1 should initially decline, unless existing authority proves them completely:

- unresolved receiver fallback through `this`;
- explicit member/inherited lookup with unknown receiver or delegation state;
- `super` with unproven effective lookup origin;
- composition-contributed bindings with dynamic/partial provenance;
- dynamic imports or guessed module paths;
- module-member cases whose result depends on unproven cyclic/partial initialization state;
- prelude/std/dependency source targets without current source authority; and
- any path-dependent/runtime-created binding space for which the static result set is incomplete.

This is an implementation coverage boundary, not a permanent semantic prohibition. Stronger future analysis can expand the successful proof domain without reopening D110.

## Prior-art audit

The D110 audit deliberately covered prototype/message-oriented systems, dynamic languages, gradually/static analyzers and nominal/static counterexamples.

### Self

Self is the closest conceptual precedent. Slot/message lookup is rooted in actual receiver/parent structure and the environment separately offers broad selector/slot exploration. The lesson for Protos is that selector/name search is not definition identity when delegation state matters.

### Pharo / Smalltalk

Smalltalk environments distinguish runtime method lookup from broad “implementors of selector” browsing. This reinforces a separate exploratory candidate-navigation feature instead of weakening exact definition semantics.

### TypeScript / tsserver

TypeScript resolves navigation through compiler symbol/declaration identity and exposes an array-capable definition API. It demonstrates that multiple locations are compatible with strong semantic identity, but its nominal/static assumptions do not transfer wholesale to Protos.

### Python — Pyright and Jedi

Pyright generally anchors navigation in analyzer declarations and does not turn workspace symbol search into a blind fallback when dynamic framework typing is unavailable. Jedi explicitly permits multiple `goto()` results because Python can be dynamic. Together they are strong evidence for honest multi-target results without requiring heuristic incompleteness.

### Ruby — Ruby LSP / RubyIndexer, Solargraph, Steep

Ruby LSP provides the strongest counter-policy: when receiver inference is unavailable it can return a bounded set of same-name method candidates. This is useful UX, but the set is intentionally heuristic/incomplete and therefore unsuitable as Protos baseline definition identity. Solargraph and Steep show how richer maps/types can increase precision, at the cost of additional analysis/type authority.

### LuaLS

LuaLS obtains definition candidates from its VM/static-analysis relation (`getDefs`) and can return several deterministic locations. It demonstrates that a dynamic language can expose a result set from semantic analysis rather than only nominal declarations.

### ElixirSense

ElixirSense uses cursor environment, variable/version metadata and module/function introspection, returning an exact location or no result. Its bounded redirection from `use`-injected behavior back to an explicit `__using__` definition is useful evidence for provenance-aware navigation instead of same-name guessing.

### clojure-lsp

clojure-lsp consumes its analysis database and emits a definition only when the analysis relation identifies one. Unsupported interop/analysis surfaces remain partial rather than being promoted to guessed definition identity.

### Erlang ELP

ELP is a strong scaling precedent: an incremental semantic database supports IDE navigation without making query-time global text search the semantic authority.

### rust-analyzer

rust-analyzer is the strongest static semantic precedent. `goto_definition` classifies identifiers through semantic HIR, descends macro expansions, maps exact definitions back to source and may produce a vector of navigation targets. Multiple targets therefore do not imply heuristic identity.

### clangd

clangd demonstrates excellent project-scale indexing and semantic navigation. Some declaration/best-known-location fallbacks are useful C/C++ product choices, but D110 deliberately does not import a weaker “best known” identity into Protos.

### SourceKit-LSP

SourceKit-LSP combines compiler/sourcekitd semantics with index/build information and demonstrates that project-scale source navigation can remain separate from storage/index topology. “Best known” fallbacks remain weaker than D110's proof contract.

### Eclipse JDT LS

JDT's mature Java model demonstrates semantic/project/classpath authority and large-workspace scalability. Its nominal declaration model is a counterexample to assuming that Protos has comparable statically unique member identity.

### LSP protocol

The protocol permits one location, multiple locations/location links, or no result. It does not require a language server to collapse semantic ambiguity to one guess. D110 uses that flexibility directly and adds no protocol extension.

## Prior-art suitability for D110

Scores measure suitability as a D110 precedent, not overall tool quality.

| System | Future endurance /10 | Scalability /10 | Protos philosophy /10 |
| --- | ---: | ---: | ---: |
| LSP result flexibility | **10** | **10** | **10** |
| rust-analyzer | **10** | **10** | 9.5 |
| Erlang ELP | **10** | **10** | 9 |
| Pyright | 9.5 | 9.5 | 9.5 |
| ElixirSense | 9 | 8.5 | 9.5 |
| clojure-lsp | 9 | 9 | 9 |
| TypeScript / tsserver | 9.5 | 9.5 | 8 |
| Jedi | 8.5 | 7 | 8 |
| LuaLS | 8.5 | 8.5 | 7.5 |
| Ruby LSP | 8.5 | 8.5 | 5.5 |
| clangd | 9.5 | **10** | 8 |
| SourceKit-LSP | 9.5 | 9.5 | 7.5 |
| Self environment | 8 | 6 | **10** |
| Pharo / Smalltalk | 8 | 7 | 8.5 |
| Erlang LS fuzzy-definition precedent | 6.5 | 7.5 | 4.5 |

## Candidate set

### A — unique-proof-only, fail closed

Return only one exact definition; ambiguity always yields no result.

Strongly safe and simple, but unnecessarily discards honest finite ambiguity that LSP can represent and future static analysis may prove.

### B — complete finite proof set

Return every exact possible source definition when the complete finite set is proven.

Semantically strong but underspecified about how a first implementation avoids overcommitting to expensive whole-program proof.

### B′ — complete finite proof set, singleton-first — SELECTED

Use B as the durable meaning and F's narrow exact coverage as an implementation strategy. Successful results remain exact/complete; generation 1 can deliberately decline hard cases and expand monotonically later.

### C — best-effort language-server heuristic

Return plausible same-name/proximity/receiver candidates despite incomplete semantic proof.

Rejected because it turns useful exploratory hints into definition identity.

### D — exact result plus heuristic fallback

Mix exact and heuristic truth levels in the standard definition request.

Rejected because common LSP clients cannot reliably communicate that distinction to users, and the fallback would become compatibility behavior.

### E — runtime-assisted definition

Execute/evaluate enough guest state to discover current runtime lookup.

Rejected for the static baseline: it depends on Actor/process/runtime state, violates no-guest requirements, changes lifecycle/cost and would describe one runtime state rather than source semantics generally.

### F — narrow mechanically exact subset as the permanent contract

Excellent generation-1 strategy and highly Protos-aligned, but too restrictive as a durable contract. It would require a new decision merely to add a later complete multi-target proof.

### G — defer G4

Semantically safe but unnecessary. Exact useful source bindings already exist, and LM009 requires useful S5 navigation rather than universal dispatch resolution.

## Mandatory GITHUB010 scorecard

Scores are 1–5; confidence is HIGH except where a candidate's future inference cost is inherently workload-dependent.

| Criterion | A | B | **B′** | C | D | E | F | G |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5 | 5 | **5** | 2.5 | 3.5 | 4 | 5 | 5 |
| Protos alignment | 5 | 5 | **5** | 2 | 2.5 | 1 | 5 | 5 |
| Future-option resilience | 4 | 5 | **5** | 3.5 | 4 | 2 | 3.5 | 3 |
| Scalability | 5 | 4 | **5** | 4 | 3.5 | 1.5 | 5 | 5 |
| Conceptual simplicity | 5 | 3.5 | **4.5** | 4 | 2.5 | 1.5 | 5 | 5 |
| Portability / implementation freedom | 5 | 5 | **5** | 4 | 3 | 1 | 5 | 5 |
| Runtime / resource cost | 5 | 4 | **5** | 4 | 3 | 1 | 5 | 5 |
| Failure / operability | 5 | 5 | **5** | 2 | 2.5 | 1 | 5 | 5 |
| Reversibility / migration cost | 4 | 5 | **5** | 2 | 3 | 2 | 4 | 5 |
| Evidence maturity / implementation risk | 5 | 4.5 | **5** | 4.5 | 3.5 | 3 | 4.5 | 4 |
| **Total / 50** | **48** | **46** | **49.5** | **32.5** | **30.5** | **18** | **47** | **47** |

Arithmetic is advisory. G scores well because doing nothing is safe, but it delivers no G4/S5 navigation. F scores well because it is an excellent first implementation, but it unnecessarily freezes today's analysis coverage as tomorrow's semantic contract.

### Focused project-owner scores

| Candidate | Aguante de futuro /10 | Escalabilidad /10 | Filosofía Protos /10 |
| --- | ---: | ---: | ---: |
| A | 8.5 | **10** | **10** |
| B | **10** | 8.5 | **10** |
| **B′** | **10** | **10** | **10** |
| C | 6.5 | 8 | 4 |
| D | 7.5 | 7 | 4.5 |
| E | 4 | 3 | 2 |
| F | 7.5 | **10** | **10** |
| G | 5 | **10** | 9 |

## Scalability and future stress

### Large repositories

Definition does not scan all same-name workspace symbols. Proof can stay source/scope/project-local and later consume incremental indexes. Unsupported expensive cases fail closed. No Top-N semantic truncation is allowed.

### Many projects / remote analysis

Canonical ProjectBinding/source identity remains the partition key. A persistent, sharded or remote proof engine can preserve the same response contract.

### Concurrent edits

The exact immutable open-document snapshot discipline remains authoritative. A stale parse/proof is discarded rather than published.

### Dynamic objects, Tasks, Actors and Processes

No live object graph, Actor, Task, Process or Context is consulted. One runtime state cannot silently become editor semantic authority.

### Richer future analysis

Flow analysis, abstract interpretation, provenance tracking or future language-owned type information may prove more references. B′ admits that growth without changing successful-result meaning.

### Generated/virtual/dependency/stdlib sources

D110 does not fabricate source custody. New exact source authorities may be introduced by later approved decisions and then consumed by the same proof contract.

## Strongest argument against B′

B′ can initially feel conservative in a prototype/delegation language. Finding one likely same-name method is much easier than proving that no other receiver/delegation/composition target can apply. Ruby LSP deliberately favors useful fallback candidates in such situations.

That is a real UX trade-off. Protos nevertheless chooses trustworthiness for the standard definition operation: a missing definition is visible and recoverable through workspace symbol search, whereas a wrong definition silently teaches users and later tooling a false identity relation.

## Regret trigger and escape path

**Regret trigger:** real user evidence shows that the exact-only definition surface is too sparse for practical navigation even after reasonable static analysis improvements.

**Escape path:** add a separate candidate/implementor navigation command or explicit future protocol surface that is allowed to be heuristic. Do not reinterpret existing successful `textDocument/definition` results as guesses. D106 workspace-symbol search already supplies one exploratory path.

## Intentionally deferred

D110 does not decide:

- references, rename or find-all-usages;
- completion, hover or signature help;
- a type system or declaration taxonomy;
- a public implementors/candidate-navigation feature;
- dependency/stdlib/generated-source custody;
- persistent/remote index representation;
- live runtime/debugger-assisted navigation;
- exact generation-1 proof coverage beyond the principle that it must be mechanically sound and complete for every result it returns.

## Implementation consequence

After D110 publication, `LM009-G4` is `READY_FOR_IMPLEMENTATION_AUDIT`.

That audit may choose mechanically necessary internal representations and decompose generation-1 exact proof coverage. If it exposes a new substantive semantic or durable architecture choice, G4 must stop at the normal Dxxx/PLATxxx approval gate.

## Post-ratification amendment — lexical declaration navigation for same-document F12 (2026-11-18)

**Effective status:** **RATIFIED — original Candidate B′ exact-origin proof retained as the strict tier, plus the explicitly approved lexical-navigation tier below.**
**Decision issue:** [D110 #435](https://github.com/guillermomolina/protos/issues/435); this is an approved additive amendment to that *existing* decision, not a new D-family identifier and not a retroactive claim about its September approval.
**Owning implementation:** [I168 #1135](https://github.com/guillermomolina/protos/issues/1135), [`guillermomolina/protos@01c8566e6481501b051ee67d475e1d67accadb4a`](https://github.com/guillermomolina/protos/commit/01c8566e6481501b051ee67d475e1d67accadb4a), `0.4.170-SNAPSHOT`.
**Explicit owner approval:** on 2026-11-18, after the coordinating agent explicitly identified the unresolved difference between *AST lexical-declaration navigation* and the ratified *complete exact dynamic binding proof* and said I168 should remain open until the exact D110/D258 policy reconciliation, the owner replied: **“pues apruebo, documenta y cierra”**. This approves the **specific two-tier F12 policy already implemented and tested**, not arbitrary name matching, runtime-assisted guessing, project inference, or any later unrelated extension.

### Approved effective contract (this addendum governs on conflict)

Standard `textDocument/definition` inside a current open Protos source may return:

1. **Exact-binding result (preferred).** The original D110-B′ complete finite proof set of the exact source slot origin(s) dynamically selectable under proven effects, lexical lookup and receiver/member rules. The original proof-set completeness and fail-closed rules still apply **to this tier**. An exact proven target is returned instead of lexical candidates when such a result exists.
2. **AST-backed lexical-declaration navigation (fallback).** If exact binding is *not* proven, the language service **may still navigate a syntactic bare-name read or bare assignment target** to explicit source declarations identified by its *actual AST lexical-context chain*. This is an editor navigation affordance, not evidence that any returned slot still exists, that the runtime will select it, or that the set is the complete universe of dynamically selectable bindings. Any resulting locations must remain within the exact same open URI and immutable current snapshot. A correctly parsed read without identifiable lexical declarations yields the ordinary empty response.

The fallback uses the **canonical Protos syntax tree**, not a search for matching source text, workspace symbols, filesystem names or module paths. The analyzer first builds query-local lexical context identities and declarations, and then navigates the reference's scope chain. A nearer applicable parameter or slot creation shadows a more distant one; `=` is not a declaration. A declaration's source span is the identifier alone. A context with multiple structurally identifiable same-name creations can return those candidates in stable source traversal order, without asserting their runtime selection or ranking by popularity. It must not truncate candidates arbitrarily.

**Temporal scope and closures:** a direct read within its own activation sees declarations completed before that read. At an enclosing context, lookup is anchored to the point the child was created; a deferred Closure can run after subsequent mutations or creations, so structurally applicable later creations may also be shown as *possible navigation destinations* rather than proven bindings. Deferred, stored and nested Closures and callbacks passed to any message use the same rule; no selector-specific rule for `each`, `Range`, a user-defined callback method or its invocation time is permitted.

**Object construction:** while constructing an object, a direct body read can navigate to an earlier declaration of a slot on that object. A later own-slot creation does not satisfy an earlier immediate read. A Closure or nested object created inside the body does **not** capture that object as its lexical parent; it uses the genuine enclosing execution context. A method's unresolved dynamic receiver, delegation or `methodHome` is never synthesized as a lexical declaration.

**Effect boundary:** unknown calls, `removeSlot`, context aliases, new slot creations and deferred invocation can remove an exact proof; they do **not** erase the source AST's lexical relationship. Conversely, a lexical location can be incorrect as an assertion of what binding is dynamically selected. **A Definition result at this tier MUST NOT be reused as proof for References, Rename, Hover, Completion or any refactoring.** The existing strict `ProtosStaticDefinitions.resolve` path remains separate from `resolveDeclaration`; References/Hover consume exact proofs and Completion retains its proven-parameter policy. References and Workspace Symbols remain ProjectBinding-gated under D124/D209.

**Source and protocol authority:** D258-B and the unchanged D085/D089/D209/D124 boundaries continue to govern exact open URI, current snapshot, invalid/stale/closed documents, no cross-document authority, and project gating outside Definition. The standard LSP location list does **not** itself carry a confidence/proof-tier marker; consumers must not infer an exact binding merely from receiving a location. Within Definition the response is allowed to be a lexical navigation location without a complete runtime proof, and the maintained guide must state that limitation clearly. A cursor immediately after the final character of a reference may navigate only when that precise half-open span ends at the requested cursor; no scanning through whitespace or adjacent names.

### Explicit GITHUB021 predecessor invariant/delta reconciliation

| Ratified invariant / prior rule | Later approved delta or preservation |
| --- | --- |
| D110 B′ clauses 1–2, 7–8: every successful Definition is a proven, complete finite exact runtime-origin set; otherwise empty | **Amended for standard Definition only**: the exact tier retains that guarantee, but a separate AST lexical-navigation fallback may answer without a dynamically exact, complete binding set. Empty remains mandatory when neither tier has a source-backed target. The old universal exactness claim no longer covers every successful F12 result. |
| D110 clauses 3–6: singleton, multiple targets, deterministic order, no cap | Retained for proven results; lexical multiple targets are deterministic source-backed navigation candidates, **not** a complete runtime binding set and not ranking. No arbitrary truncation. |
| D110 clause 9: reject workspace/symbol, text, ranking and proximity guesses | **Preserved**; lexical navigation follows AST scope identity and declaration provenance, not name search. |
| D110 clause 10: never execute guest code / Truffle Context | **Preserved**. |
| D110 clauses 11–12, as explicitly amended by D258-B: only canonical project sources or an exact open same-URI/snapshot local Definition | **Preserved**. D258 was a source-authority amendment; this later addendum independently approves the weaker navigation-certainty tier, **not** additional source authority. |
| D110 clause 13: source definitions may include lexical binding origins beyond workspace symbols | Preserved and used; the newer tier still requires a canonical AST source declaration, rather than a workspace-symbol match. |
| D110 clause 14: `:` creates, `=` assigns existing selected binding | **Preserved**. For unproven assignment targets, navigation may show a syntactic lexical declaration without proving the selected destination. |
| D110 clause 15: no answer for an incompletely proven lexical chain/possible receiver fallback | **Amended for bare-name lexical navigation only**: an AST declaration may be returned despite incomplete runtime selection, with no claim about actual receiver fallback. |
| D110 clauses 16–20: member/delegation/`super`, composition/import/module/prelude and multi-project authority | **Preserved for exact proof**. No lexical fallback for unknown member selectors, implicit delegated targets, modules or guessed external sources. |
| D110 clause 21: implementation/index topology replaceable while preserving successful exact identity | **Preserved for strict tier**; lexical-navigation provenance must also remain AST-backed and within the same snapshot. |
| D110 clause 22: a non-exact candidate capability must be separate and not silently weaken standard Definition | **Explicitly superseded only for the described AST lexical-declaration F12 fallback**, now a disclosed standard Definition behavior. Generic possible implementors, arbitrary candidate search and fuzzy ranked navigation remain separate/unapproved. |
| D258-B clauses on exact URI/immutable snapshot, project gating for References and Workspace Symbols, no runtime, no inferred project | **Unchanged**, notwithstanding D258's *historical* statement that every Definition result had to meet the original strict D110 proof. That strict-result wording is superseded only by this later owner-approved D110 amendment. |
| Core Protos execution semantics and runtime/JIT, including lexical contexts as mutable objects | **Unchanged**. This decision governs editor navigation, not guest slot lookup, dispatch or assignment. |

**Owner-approval consistency result:** PASS for this *bounded* delta. The previously owner-ratified unconditional exact-result guarantee and ban on inline non-exact Definition were explicitly surfaced as the remaining conflict *before* the owner's above approval. The approved replacement is limited to AST-backed same-document lexical navigation with an independently unchanged strict proof tier. No approval is inferred for fuzzy/nominal guesses or future extensions.

### Evidence, precedent and regret outcome

The original D110 prior-art research compared Self, Smalltalk/Pharo, TypeScript, Pyright/Jedi, Ruby tooling, LuaLS, ELP, clojure-lsp, rust-analyzer, clangd, SourceKit-LSP and JDT. The later I168 implementation uses the **scope parent/variable-resolution separation** established by GraalPy, GraalJS and TruffleRuby, retaining Protos's mutable context and slot semantics; those implementations are precedents, not semantic authorities.

The original D110 regret trigger is now observed: exact-only Definition was too sparse to navigate a deferred `finalDoors` capture in the real `100-doors.protos` callback. The owner's real VS Code acceptance reports successful F12 after the two-tier implementation. The exact implementation's focused suite reported **207/207 PASS** on substantive pre-finalization source, and the owner reported all local tests PASS, clean `git diff --check`, and installed-editor F12 PASS. No post-finalization full-suite transcript, installed binary digest or public release is claimed.

**Why this narrow alternative instead of generic heuristics:** the original pure-proof-only policy remains sound but misses commonplace callback navigation; broad same-name/fuzzy candidates or runtime inspection break source identity and operational cost; a separate editor command would require a different UX and client contract. This explicitly approved fallback reuses AST scope identity without a second resolver or guest execution. The trade-off is that standard LSP Definition locations no longer all assert exact runtime binding identity. That limitation is an intentional public tooling promise and must stay documented.

```text
DECISION=D110_POST_RATIFICATION_LEXICAL_NAVIGATION_AMENDMENT
APPROVED_BY=PROJECT_OWNER_2026_11_18_ACTIVE_CHAT
APPROVAL_TEXT=pues apruebo, documenta y cierra
GITHUB021_INVARIANT_DELTA=PASS_FOR_EXPLICITLY_REOPENED_DEFINITION_PROOF_GUARANTEE
EFFECTIVE_DEFINITION=EXACT_PROOF_PREFERRED_ELSE_AST_LEXICAL_NAVIGATION
LEXICAL_CANDIDATE_IS_EXACT_RUNTIME_BINDING=NO
PROJECT_AND_SNAPSHOT_AUTHORITY=D258_B_UNCHANGED
REFERENCES_HOVER_COMPLETION=STRICT_PROOFS_UNCHANGED
GUEST_RUNTIME_OR_LANGUAGE_SEMANTICS_CHANGE=NONE
IMPLEMENTATION_REVISION=01c8566e6481501b051ee67d475e1d67accadb4a
IMPLEMENTATION_VERSION=0.4.170-SNAPSHOT
LIVE_IMPLEMENTATION_ISSUE=I168_1135
```

