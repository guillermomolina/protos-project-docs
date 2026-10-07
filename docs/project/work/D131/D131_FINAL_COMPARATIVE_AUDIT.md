# D131 — Final comparative audit for the exact protocol-first candidate

Status: **READY_FOR_EXPLICIT_OWNER_DECISION**

Nature: non-normative D131 decision evidence

Decision authority: `D131` / `guillermomolina/protos#503`
Related audit: `AUD009-A1` / `guillermomolina/protos#535`
Exact candidate checkpoint: `D131_PROTOCOL_FIRST_BONFIRE_CANDIDATE_CHECKPOINT.md`
Prior broad evidence: `../AUD009/AUD009_A1_MATCHING_REVIEW.md`, `../AUD009/AUD009_A1_MATCHING_SECOND_PASS.md`, `../AUD009/AUD009_A1_MATCHING_PROTOCOL_FIRST_RESULT.md`
Product baseline: `6ccd8b91ca5446958841db125990e4e0756c3dd0`

This packet is the final comparative/invariant review required before asking the project owner to ratify an exact D131 outcome. It does not itself select or implement the candidate.

## 1. Research inheritance and differential scope

D131/AUD009-A1 already performed the broad mature-language comparison requested by the issue, including Rust, Scala 3, Python, C#, Java, Swift, Kotlin, Dart, Ruby, Erlang/Elixir, OCaml/F#, Haskell, Racket/Clojure where useful, and Smalltalk/Self/Io as philosophy/control-flow contrasts.

The protocol-first second pass then re-framed that evidence under the stricter anti-overengineering burden:

```text
fundamental/general-purpose need
OR
ordinary Protos cannot reasonably express it
OR
concrete material cost of adding it later
```

This final pass is therefore differential. It does not repeat the complete language survey; it tests the material deltas introduced after Candidate B:

- direct ordinary `Array.match` / `Map.match` structural behavior;
- `value.caseOf(cases)` as the ordinary selector;
- normal insertion-ordered Map as the exact case carrier;
- `Any` + `Capture` as the only initial standard helper matchers;
- removal of initial `Or`, `Guard` and terminal Array remainder capability;
- explicit receiver eligibility and shallow matcher/subject/case snapshots.

The earlier primary references and comparative evidence remain applicable to the general design-space claims. No new external precedent changes the central conclusion: rich dedicated pattern syntax is common and ergonomic, while message/block-oriented systems demonstrate that ordinary objects/messages/Closures can own control semantics without requiring a second compiler-owned control language.

## 2. Surviving whole-model candidates

### A — Current rich matching grammar

Retain the present postfix `subject match { case ... }` language and most D095/D096 machinery, with only narrow removals.

Main strength: compact, familiar, already implemented.

Main weakness: keeps a second pattern/binding/control sublanguage with large interaction surface.

Qualitative gate: **OVERENGINEERING RED FLAG** under the second-pass burden because ordinary protocol composition now covers the foundational need.

### B — Prior protocol-first Candidate B

Keep `pattern.match(subject)` and ordinary first-success selection, but standardize a broader initial matcher toolkit including Guard, OR and terminal Array remainder capability; keep structural recognition as standard matcher capability without yet committing it directly to ordinary Array/Map values.

Main strength: broad matching power without dedicated grammar.

Main weakness: pre-installs several capabilities that can be added later without changing the fundamental matcher authority.

Qualitative gate: no architectural red flag, but **present-need proportionality concern** after the owner-requested bonfire pass.

### C — Exact protocol-first bonfire candidate

Use:

```text
pattern.match(subject)
Object.match -> this == subject
Array.match  -> fixed structural Array recognition
Map.match    -> open/subset structural Map recognition
Any
Capture
value.caseOf(normal insertion-ordered Map<matcher, callable>)
```

No dedicated match grammar and no initial OR/guard/remainder/exactness institutions.

Main strength: smallest complete ordinary-object model found by the review.

Main weakness: it makes two intentionally strong public commitments that are less reversible than merely omitting syntax: ordinary Array/Map values gain structural `match` behavior, and `caseOf` initially treats a normal Map's hash/equality/unique-key semantics as part of the case representation.

Qualitative gate: **NO OVERENGINEERING RED FLAG**. A bounded **irreversibility watchpoint** remains on direct Array/Map `match` behavior and the exact case-carrier contract; both are surfaced explicitly below rather than hidden.

### D — Protocol-first semantics plus dedicated sugar immediately

Adopt an ordinary protocol foundation but also retain/redesign compact matching syntax in D131 now.

Main strength: ordinary semantics plus immediate familiar ergonomics.

Main weakness: commits grammar before real ordinary-protocol use demonstrates which sugar is worth paying for.

Qualitative gate: **PRESENT-NEED PROPORTIONALITY RED FLAG**; the cost of deferring sugar is low and bounded.

## 3. GITHUB010 common scoring

Scale: 1 = poor, 5 = strong. Confidence follows each score.

| Dimension | A — rich grammar | B — prior protocol-first | C — exact bonfire | D — protocol + sugar now |
| --- | --- | --- | --- | --- |
| 1. Correctness / invariant preservation | **2 / HIGH** — mature current semantics, but conflicts with the owner-approved protocol-first direction and retains institutions the active bonfire review explicitly challenged. | **4 / HIGH** — preserves protocol-first authority and ordinary callable binding, but retains capabilities the exact checkpoint now intentionally omits. | **5 / HIGH** — preserves every current owner-review invariant recorded in the exact checkpoint. | **4 / HIGH** — protocol foundation is sound, but immediate sugar reopens surface choices intentionally left for evidence-driven later work. |
| 2. Protos alignment | **2 / HIGH** — extensible matcher core is Protos-like, but the compiler-owned pattern language is a large parallel institution. | **4 / HIGH** — mostly ordinary objects/messages/Closures; broader standard matcher toolkit remains. | **5 / HIGH** — ordinary values, messages, Maps, Arrays and Closures own nearly all behavior; minimal privileged surface. | **4 / HIGH** — sound ordinary foundation, but adds a dedicated source institution immediately. |
| 3. Present-need proportionality | **1 / HIGH** — current users pay for aliases, remainders, OR binding rules, static coverage and grammar interactions regardless of use. | **3 / HIGH** — substantially smaller, but still prebuilds Guard, OR and terminal remainder. | **5 / HIGH** — initial standard additions are only `caseOf`, `Any`, `Capture` and direct fixed/open structural matching. | **2 / HIGH** — pays for grammar before usage evidence establishes the need. |
| 4. Incremental growth | **3 / HIGH** — extensible, but additions must coordinate with an already-large pattern algebra and grammar. | **5 / HIGH** — matcher combinators/options can grow without changing the authority. | **5 / HIGH** — omitted capabilities have clear matcher/combinator/sugar growth paths; no speculative machinery required. | **4 / HIGH** — protocol grows well, but early sugar can constrain later ergonomic choices. |
| 5. Future-option resilience | **3 / MEDIUM** — broad existing commitments preserve capability but narrow simplification/syntax options. | **5 / HIGH** — keeps protocol open while avoiding some direct collection/case-carrier commitments. | **4 / MEDIUM** — excellent capability growth path, but direct Array/Map `match` and normal-Map `caseOf` are real public commitments. | **4 / MEDIUM** — ordinary foundation helps, but early syntax reduces option freedom. |
| 6. Scalability | **4 / HIGH** — snapshots and deterministic semantics are robust; complexity is mostly semantic/tooling rather than asymptotic. | **4 / HIGH** — ordinary composition and bounded structural snapshots scale predictably. | **4 / HIGH** — same bounded snapshots/ordered matching; no new closed-world global analysis. | **4 / HIGH** — same runtime foundation as protocol-first candidates. |
| 7. Conceptual simplicity | **1 / HIGH** — large pattern taxonomy and many cross-feature rules. | **3 / HIGH** — one matcher authority but several initial standard combinators/options. | **5 / HIGH** — one matcher authority, two helper matchers, direct structural collections, one ordinary selector. | **2 / HIGH** — protocol plus a second ergonomic syntax surface from day one. |
| 8. Portability / implementation freedom | **4 / HIGH** — specification is abstract, though parser/AST/runtime machinery is broad. | **5 / HIGH** — ordinary object/call semantics leave implementation freedom. | **5 / HIGH** — same; shallow observations specify behavior without requiring a physical copy representation. | **4 / HIGH** — protocol is portable but dedicated lowering/tooling adds implementation surface. |
| 9. Runtime / resource cost | **3 / MEDIUM** — rich pattern machinery and analysis can add work, though many paths are optimized/bounded. | **4 / MEDIUM** — standard combinators/options are pay-for-use but add objects/paths. | **4 / MEDIUM** — minimal machinery; Map case construction still pays normal key hash/equality costs by design. | **4 / MEDIUM** — runtime can lower efficiently, but syntax adds compile/tool cost rather than major execution cost. |
| 10. Failure / operability | **4 / HIGH** — extensively specified failure rules and diagnostics. | **4 / HIGH** — ordinary Error/control behavior dominates; combinators add some failure composition. | **4 / HIGH** — small failure model; invalid receiver, invalid matcher result, selected non-callable and no-match are ordinary failures. Map-key effects during case construction are deliberate and visible ordinary behavior. | **4 / HIGH** — ordinary protocol remains diagnosable; sugar needs extra diagnostics. |
| 11. Cost of deferral / reversibility / migration | **2 / HIGH** — keeping everything now makes later removal a compatibility/migration problem. | **5 / HIGH** — omitted dedicated syntax can be added later; separate structural matcher forms preserve maximum option space. | **4 / HIGH** — deferred OR/guard/rest/sugar are cheap to add; direct Array/Map structural `match` is the main less-reversible commitment. | **2 / HIGH** — deferring sugar is cheap, so committing it now creates avoidable migration risk. |
| 12. Evidence maturity / implementation risk | **5 / HIGH** — already specified and implemented. | **4 / HIGH** — built from existing matcher/call semantics; several exact APIs still needed selection. | **4 / HIGH** — most semantics reuse existing contracts; exact `caseOf`/direct collection ownership is newly selected design territory but mechanically tractable. | **4 / MEDIUM** — foundation is mature; exact redesigned sugar would require another grammar/tooling design cycle. |

### Score interpretation

No arithmetic total selects D131. The non-compensating gates matter more:

- A fails the present-need/overengineering gate.
- D fails the present-need gate because sugar can be deferred cheaply.
- B is credible and future-resilient, but retains capability without a current burden-of-proof win.
- C is the smallest sufficient candidate and best matches the active owner-review invariants, with two explicit irreversibility watchpoints rather than hidden costs.

**Recommendation pending owner approval: Candidate C.**

## 4. D131-specific feature matrix for Candidate C

The outcome vocabulary follows D131 exactly. `REMOVE_FROM_CORE_V0_1` means despecify/deimplement now; future reconsideration requires a new explicit decision.

| Current capability / institution | Candidate C outcome | Rationale | Main implementation consequence / recovery path |
| --- | --- | --- | --- |
| `pattern.match(subject)` | `KEEP_IN_CORE_V0_1` | Single open recognition authority. | Preserve ordinary lookup/dispatch/control semantics. |
| inherited `Object.match` -> `this == subject` | `KEEP_IN_CORE_V0_1` | Makes ordinary values zero-capture matchers with no wrapper. | Preserve exactly-once equality send. |
| `false / true / non-empty Array` carrier | `KEEP_IN_CORE_V0_1` | Minimal unambiguous recognition + extraction carrier; feeds ordinary call spread. | Preserve matcher-result validation and capture composition. |
| source-order first-success selection capability | `KEEP_IN_CORE_V0_1` | Fundamental multi-way case behavior. | Rehome in ordinary `value.caseOf(cases)`. |
| postfix `subject match { ... }` grammar | `REMOVE_FROM_CORE_V0_1` | Dedicated grammar not fundamental once ordinary protocol exists. | Remove parser/AST/lowering/runtime/doc acceptance; future sugar may lower to `caseOf`. |
| `case` arm grammar | `REMOVE_FROM_CORE_V0_1` | Replaced by ordinary Map associations `matcher -> callable`. | Remove arm nodes/binding machinery. |
| terminal no-selection `Error` behavior | `KEEP_IN_CORE_V0_1` | Distinguishes no match from successful body result including `null`. | Preserve under `caseOf`. |
| `@name` binder syntax | `REMOVE_FROM_CORE_V0_1` | Capture names belong to Closure parameters. | `Capture` + ordinary callable parameters. Future binder sugar may be reconsidered. |
| `_` wildcard syntax | `REMOVE_FROM_CORE_V0_1` | `Any` is a simple ordinary accept-all matcher. | Provide standard `Any`; future `_` sugar may be reconsidered. |
| duplicate/linear pattern-binding restrictions | `REMOVE_FROM_CORE_V0_1` | No pattern-owned names remain. | Ordinary Closure parameter rules own naming. |
| guard capability | `REMOVE_FROM_CORE_V0_1` initially | Useful but not fundamental to a usable first case model; can be composed later without changing matcher authority. | Future `Guard(...)` ordinary matcher decision if real use justifies it. |
| `when` syntax / guard `=>` delimiter rules | `REMOVE_FROM_CORE_V0_1` | Syntax-specific institution disappears with guard grammar. | Future sugar, if any, starts from ordinary matcher composition. |
| fixed Array structural recognition | `KEEP_IN_CORE_V0_1` | Core practical destructuring need. | Move ownership to standard `Array.match`; exact-length only. |
| Array-pattern grammar | `REMOVE_FROM_CORE_V0_1` | Ordinary Array construction already supplies the matcher value. | `[1, Capture]` is an ordinary Array expression. |
| terminal Array remainder | `REMOVE_FROM_CORE_V0_1` | Useful but addable later without changing `match` foundation. | Future remainder matcher/combinator; optional later sugar. |
| middle Array remainder | `REMOVE_FROM_CORE_V0_1` | More specialized; no demonstrated foundational need. | Future explicit decomposition capability if usage appears. |
| bare Array remainder | `REMOVE_FROM_CORE_V0_1` | Depends on removed remainder institution. | Ordinary code or future remainder matcher. |
| fresh/frozen Array remainder aggregate semantics | `REMOVE_FROM_CORE_V0_1` | No initial remainder means no residual aggregate institution is needed. | Future remainder decision selects freshness/freezing explicitly. |
| standard-Array eligibility distinction | `KEEP_IN_CORE_V0_1` | Delegation must not manufacture built-in Array state. | `Array.match` invalid receiver -> Error; non-Array subject -> false. |
| shallow Array attempt observation | `KEEP_IN_CORE_V0_1` | Prevents child effects from rewriting the current attempt. | Snapshot matcher + subject references before child matching. |
| open/subset Map structural recognition | `KEEP_IN_CORE_V0_1` | Core structured-data destructuring need. | Move ownership to normal standard `Map.match`. |
| Map-pattern `%{...}` grammar | `REMOVE_FROM_CORE_V0_1` | D136 expression Map construction already creates ordinary matcher Maps. | `%{...}` remains expression Map construction only. |
| normal-Map eligibility distinction | `KEEP_IN_CORE_V0_1` | Delegation/Map-like messages do not confer normal Map keyed state. | `Map.match` invalid receiver -> Error; non-Map subject -> false. |
| stable Map association snapshot/query semantics | `KEEP_IN_CORE_V0_1` | Needed for deterministic recognition under effectful hash/equality/children. | Snapshot matcher + subject associations; normal Map key law remains authoritative. |
| exact Map mode | `REMOVE_FROM_CORE_V0_1` | Open/subset is sufficient baseline; exactness can be layered later. | Future `Exact(...)`/equivalent ordinary matcher capability. |
| Map remainder capture | `REMOVE_FROM_CORE_V0_1` | Residual Map capture is specialized and addable later. | Future remainder matcher decision. |
| bare Map remainder discard | `REMOVE_FROM_CORE_V0_1` | Adds no capability over open/subset recognition. | No recovery needed; open matching already ignores residue. |
| fresh/frozen Map remainder semantics | `REMOVE_FROM_CORE_V0_1` | No initial Map remainder institution. | Future remainder decision selects residual semantics explicitly. |
| alias `@whole: pattern` | `REMOVE_FROM_CORE_V0_1` | Outer subject is lexically available; nested whole-subject capture can be a later matcher wrapper. | Ordinary lexical capture now; future wrapper if repeated need emerges. |
| OR pattern capability / `p1 | p2` | `REMOVE_FROM_CORE_V0_1` initially | Common but not necessary for a usable base; duplicate outer cases cover classic case use. | Future `Or(...)`, then possible `|` sugar. |
| OR first-success/binding-interface institution | `REMOVE_FROM_CORE_V0_1` | No initial OR; Closure parameters own names if OR returns later. | Future `Or` can return first successful carrier unchanged. |
| fixed `captures(a,b)` | `REMOVE_FROM_CORE_V0_1` | Ordinary call spread + Closure parameters already bind positional captures. | Delete source/compiler binding bridge. |
| dynamic `captures(first,...rest)` | `REMOVE_FROM_CORE_V0_1` | Ordinary Closure rest parameters own this behavior. | Delete matching-specific dynamic binding machinery. |
| D103 complete-arm dynamic-rest terminality | `REMOVE_FROM_CORE_V0_1` | Exists to repair removed matching-specific binding ABI. | Ordinary callable parameter rules remain. |
| matching-specific selected-arm binding ABI | `REMOVE_FROM_CORE_V0_1` | Duplicates ordinary invocation. | Selected callable invoked normally with spread captures. |
| tri-state static coverage framework | `REMOVE_FROM_CORE_V0_1` | Arbitrary ordinary matcher objects are open-world; no dedicated syntax currently justifies the institution. | Future sufficiently closed sugar may add syntax-local analysis. |
| structural arm unreachability diagnostics | `REMOVE_FROM_CORE_V0_1` | No arm grammar remains; general dynamic matcher redundancy is not statically decidable. | Ordinary runtime behavior; future syntax-local checks if justified. |

### New standard surface selected by Candidate C if ratified

These are replacements/additions, not legacy features being silently retained:

```text
Object.caseOf(cases) / value.caseOf(cases)
Any
Capture
standard Array.match structural override
standard Map.match structural override
```

Their exact semantics are fixed by the candidate checkpoint and must become normative only after explicit owner ratification.

## 5. D131 required evaluation dimensions — delta analysis

The broad AUD009 packets already scored/assessed the existing mechanisms. The final candidate changes the following dimensions materially:

### Real-world utility / frequency

- equality/value cases, default, capture and fixed Array/Map destructuring cover the recurring baseline;
- OR/guard/rest/exact/remainder remain valuable but are not required to make the baseline useful;
- dedicated static coverage is less useful when matcher objects are deliberately open-world.

### Lexer/parser/grammar complexity

Candidate C removes all dedicated matching grammar initially. `caseOf`, `Any`, `Capture`, Arrays, Maps and Closures use existing ordinary syntax. This is the strongest simplification dimension.

### AST/lowering/compiler/runtime complexity

Pattern AST/lowering and arm-binding institutions can be removed. Runtime complexity concentrates in ordinary `match`, direct Array/Map structural attempts and one `caseOf` traversal.

### Static validation/tooling complexity

Matching-specific static coverage/binding-interface rules disappear. Tools continue to understand ordinary calls, Maps, Arrays and Closures. Future sugar may reintroduce local tooling when justified.

### Test/conformance maintenance

Large current syntax/interaction suites can be removed/replaced by focused protocol tests: root match, carrier, Array/Map structural behavior, Any/Capture and caseOf ordering/invocation/failure.

### Programmer cognitive cost / teaching

The base model becomes:

```text
matcher.match(value)
value.caseOf(%{ matcher: action })
Any
Capture
Array/Map structural match
```

This is materially smaller than the current pattern taxonomy.

### Interaction complexity

Most current cross-feature rules disappear because names/arity return to callable semantics and OR/guard/remainder are absent initially.

### Future-proofing / add-later cost

Most removed features can be added by ordinary matcher objects/combinators and optional sugar without changing `pattern.match(subject)`. The two less-reversible decisions are direct standard Array/Map `match` ownership and the normal-Map case carrier; those are therefore explicit ratification watchpoints.

### Scalability

No global matcher registry, closed-world analysis or generic structural-projection framework is added. Structural attempts are bounded by observed Array length/Map requirements and ordinary child calls.

### Protos philosophy

Candidate C maximizes mechanisms-over-institutions: ordinary values and messages remain the primary abstraction. It also accepts the message-oriented `caseOf` spelling for coherence with `ifTrue`, even though future surface sugar may later pursue the project's "feels like Smalltalk, writes like JavaScript" ergonomic goal.

## 6. Strongest counterarguments

### Counterargument 1 — The current rich syntax is already implemented and more immediately readable

True. Removing it discards sunk work and temporarily gives up familiar `match/case` ergonomics.

Response: D131 exists specifically before v0.1 compatibility hardens. Sunk implementation cost is not a reason to keep a large public language institution. Future sugar remains available after the ordinary protocol earns usage evidence.

### Counterargument 2 — Guard and OR are common pattern-matching capabilities

True across many mature pattern systems.

Response: common does not imply foundational. Classic case selection works without them; both can be added later as ordinary matchers without changing the public matcher authority. Deferral cost is bounded.

### Counterargument 3 — Head/tail remainder is common enough to keep now

Credible. Candidate B retained terminal remainder for this reason.

Response: fixed Array structural matching already provides the common finite-shape baseline. A future remainder matcher can be added without replacing `Array.match` or `caseOf`; therefore the bonfire burden favors deferral until real Protos code demonstrates recurring use.

### Counterargument 4 — Direct `Array.match` / `Map.match` is a stronger commitment than a separate structural matcher object

This is the strongest architectural counterargument to Candidate C.

Today ordinary Arrays/Maps inherit root `Object.match` equality behavior; Candidate C changes their standard recognition meaning to structural matching while ordinary `==` remains unchanged. Reverting that later would be a public semantic break.

Response: the separation is intentional and already supported by the matcher protocol: objects may recognize differently from equality. Direct structural collections produce exceptionally simple ordinary source (`[1, Capture]`, `%{"name": Capture}`), remove a separate pattern-constructor institution and compose recursively with the same `match` authority. The cost is acknowledged as a real ratification commitment, not hidden as implementation detail.

### Counterargument 5 — A normal Map is not a neutral case sequence

True. Case construction invokes ordinary key `hash`/`==`, equal keys cannot coexist and custom matcher key behavior is observable.

Response: the owner review explicitly treats those properties as useful/acceptable case-table semantics rather than defects. They yield a very readable `matcher: action` surface and reuse D136 Map construction/order. Exotic duplicate/equal matcher cases remain expressible through distinct wrapper objects if a future real need appears. This is also a real public commitment and is surfaced explicitly.

### Counterargument 6 — Removing static coverage loses useful diagnostics

True.

Response: the new ordinary matcher universe is intentionally open. A general compile-time coverage claim would either be weak/UNKNOWN-heavy or require another closed static institution. Future dedicated sugar can carry local static analysis if evidence justifies it.

## 7. Plausible regret scenarios and recovery paths

### Regret: `caseOf` is too verbose / insufficiently JavaScript-like

Recovery: add compact source sugar in a later Dxxx that lowers explicitly to the ordinary protocol. No change to matcher authority is required.

### Regret: guards are needed constantly

Recovery: standardize `Guard(matcher, predicate)` (or the later chosen ordinary API). Optional guard syntax can follow only if repeated use justifies it.

### Regret: OR is needed constantly

Recovery: standardize ordered `Or(...)` returning the first successful matcher carrier unchanged; later consider `|` sugar. No binding-name equivalence institution is required because callable parameters own names.

### Regret: head/rest decomposition is common

Recovery: add an Array remainder matcher/combinator with explicit residual semantics; later add sugar if useful. Fixed `Array.match` remains the foundation.

### Regret: exact Map / Map remainder are needed

Recovery: add ordinary matcher wrappers/options with explicit semantics. Open/subset direct Map matching remains compatible.

### Regret: identity matching of Array/Map values is common

Recovery: add ordinary `Identity(value)` matcher behavior. This is a local extension and does not require reverting structural `Array.match`/`Map.match`.

### Regret: direct structural `Array.match` / `Map.match` itself was the wrong ownership choice

Recovery cost: **HIGHER THAN THE OTHER DEFERRED FEATURES.** Reverting standard collection `match` semantics after compatibility commitment would be breaking. A future alternative could add explicit identity/structural wrappers while retaining the established standard behavior, but cannot silently restore the old root equality behavior.

This is the primary irreversibility watchpoint the owner must consciously accept when ratifying Candidate C.

### Regret: normal Map is the wrong universal case carrier

Recovery cost: **MODERATE.** Existing `caseOf(Map)` behavior could remain while a later ordinary selector/descriptor form is added for duplicate/equality-independent cases. The original Map contract need not break, but the language would gain another selection representation. This is less clean than choosing perfectly neutral case data now, but bounded.

## 8. Final owner-invariant / delta consistency check

Against `D131_PROTOCOL_FIRST_BONFIRE_CANDIDATE_CHECKPOINT.md`:

1. protocol-first foundation — **PRESERVED**;
2. single `pattern.match(subject)` recognition authority — **PRESERVED**;
3. Closure parameters own capture names — **PRESERVED**;
4. `value.caseOf(...)` selection spelling — **PRESERVED**;
5. normal insertion-ordered Map case carrier and its normal key semantics — **PRESERVED**;
6. no selection -> fresh ordinary Error; selected `null` remains successful — **PRESERVED**;
7. initial helper set exactly `Any` + `Capture` — **PRESERVED**;
8. invalid inherited standard Array/Map receiver -> invalid-receiver Error, no silent root fallback — **PRESERVED**;
9. shallow matcher/subject/case observations protect the current attempt from structural mutation — **PRESERVED**;
10. removed v0.1 features are actually removed, while future reconsideration remains possible — **PRESERVED**.

Materially new consequences surfaced since the checkpoint:

- **none** that change the candidate semantics;
- the comparative audit elevates direct standard Array/Map `match` ownership as the primary irreversibility watchpoint;
- the normal-Map `caseOf` representation is confirmed as a moderate rather than zero-cost future commitment.

No recorded invariant is silently contradicted.

## 9. Recommendation pending explicit owner approval

Ratify **Candidate C — exact protocol-first bonfire** as the D131 Core v0.1 matching model, with the exact semantics and removal direction recorded in the candidate checkpoint and this audit.

The approval question must explicitly include the two watchpoints rather than hiding them inside the bundle:

1. standard Array/Map values gain structural `match` semantics, distinct from their ordinary equality behavior; this is a real public semantic commitment;
2. `caseOf` initially uses a normal insertion-ordered Map, deliberately inheriting normal Map hash/equality/unique-key semantics for case entries.

If approved, the next work is mechanical planning/execution: publish the D131 ratification record, produce the ordered reconciliation/deimplementation slices, then update specification, grammar/parser/AST/lowering/runtime/tests/docs without letting any removed syntax survive as accidental compatibility behavior.
