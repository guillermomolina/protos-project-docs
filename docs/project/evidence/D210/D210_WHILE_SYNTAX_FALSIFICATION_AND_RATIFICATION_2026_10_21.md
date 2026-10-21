# D210 — Closure loop protocol and contextual while: research, falsification, owner-approved invariants

**Evidence role:** durable, non-normative GITHUB010/GITHUB021 research and ratification support, 2026-10-21.
**Decision:** [D210 / #941](https://github.com/guillermomolina/protos/issues/941).
**Observed Protos main:** [a48ef2e4ca1619e13e1156405f6cdf77ea15604f](https://github.com/guillermomolina/protos/commit/a48ef2e4ca1619e13e1156405f6cdf77ea15604f), read-only. The eventual implementation checkout/commit is separate.
**Companion non-normative decision:** [D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md).

## Approval provenance, chronology, limits

Q1 — The owner explicitly approved Candidate C: remove standard `Object.while` and preserve standard `whileTrue` with last-body-value / zero-`null` policy ([owner checkpoint](https://github.com/guillermomolina/protos/issues/941#issuecomment-6311255405)). Q2 — The owner expressly refused editor snippets in place of real syntax and explicitly accepted the collision: "asume que while deja de ser un metodo ordinario (aunque no prohibido), si alguien pone un while y lo rompe todo es su problema" ([approved invariant](https://github.com/guillermomolina/protos/issues/941#issuecomment-6311488302), [detailed source-boundary contract](https://github.com/guillermomolina/protos/issues/941#issuecomment-6311582929)). A following "ok vamos allá" authorizes progression of the already selected design toward ratification, not unrelated features. Previous ratified D201 is retained as historical truth, not silently edited.

This evidence does not claim new D210 product tests executed or any tested implementation. The owner separately reports local tests passing and a clean `git diff --check`; without a published D210 implementation revision those reports are **not** evidence that the prospective new syntax already works.

## Current authority and inspected physical implementation

- [Core grammar](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/spec/PROTOS_GRAMMAR.md) §§2,4–5,14–15,18–21,35–36: `while` is an ordinary identifier; `name(args) { body }` is an ordinary call with final trailing parameterless Closure; `name { body }` constructs a parented object; newlines separate completed expressions and only leading `.` has a postfix newline exception. [Language spec](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/spec/PROTOS_LANGUAGE_SPEC.md) §36 explicitly leaves `while` as possible future syntax.
- [Execution/control](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/spec/semantics/EXECUTION_AND_CONTROL.md) §17: `whileTrue` exact last body result or `null`, strict Boolean repeatedly evaluated; distinct `while` terminal `false` result; standard domain validation, zero-arg Closure callback, no truthiness, no loop-specific Task/suspension/cancel/await. [CALLABLES](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/spec/semantics/CALLABLES.md) §11: standard `Object.while` and `Object.whileTrue` currently observable as ordinary slots with extraction, lookup and overrides.
- [Parser](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java): `parsePostfixFoundation`, `parseCallSuffix`, `parseClosureBody`, `isParentExpression`, `recordTrailingClosureOrigin`. [SourceFacts](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/parser/ProtosParserSourceFacts.java) records source-origin facts used by tooling.
- [Native protocol](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardObjectProtocol.java): both native methods currently share `preTestLoop`, with a return-policy flag and two standard Object registrations. [PreparedCalls](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosPreparedCalls.java): ordinary selected-call guarded structured-while admission. [StructuredControlCalls](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/structured/ProtosStructuredControlCalls.java) stores boolean result policy. [StructuredDispatchLowerer](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java) and [CanonicalToBytecodeLowerer](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java): structured and literal-inline loop execution; [CoreBootstrap](https://github.com/guillermomolina/protos/blob/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java) lists both standard selector names.
- [Control conformance](https://github.com/guillermomolina/protos/tree/a48ef2e4ca1619e13e1156405f6cdf77ea15604f/protos/tests/conformance/control), especially `while-loop-results.protos`, `while-basic-and-validation.protos`, `while-binding-and-transfer.protos`, `while-future-ownership.protos`, `while-suspension-and-cancellation.protos`, `control-spelling-is-ordinary.protos`; lexical surface tests explicitly permit `while: 17`.

## Strongest falsifications — actually incompatible, not implementation bugs

**F1, decisive and explicitly waived by the owner:** existing legal program

```protos
while: (value, block) => { block(); value }
result: while(123) { 42 }  // old result 123
```

is an ordinary eager `while` call today. With new contextual sugar it is a loop, the condition `123` fails strict-Boolean validation, and the old callable is not invoked. Renaming a binding to `while` can therefore change a call's syntactic category. Q1's deletion of **standard** `Object.while` cannot eliminate this collision with **user-defined** lexical `while`; it was an explicit Q2 owner acceptance, not a refutation of F1.

**F2, expression/source form:** `while(n > 0) { ... }` has fully delimited condition, unlike rejected D199 `x > 0 ifTrue { ... }`. It does **not** collide with parented-object `while { ... }`, because the former has a completed parenthesized argument list plus trailing Closure. Keep the existing exact newline contract: `while(n > 0)\n{...}` is two expressions, not a loop. Grouped `(while)(n) { ... }` retains ordinary call meaning. Member send `holder.while(n) { ... }` remains an ordinary call.

**F3, rejected semantic fallbacks:** 'only when no lexical `while` binding' depends on dynamic lookup and changes parse/capture; 'only for a Boolean argument' eagerly evaluates once rather than deferring/repeating; implementing `while` as an ordinary prelude function cannot recover laziness under Protos' eager argument evaluation; a global reserved token needlessly breaks `while: value` and `obj.while`; a new built-in loop runtime would bypass ordinary selected `whileTrue` dispatch and shadowing.

**F4, zero/one/N result identity:** `whileTrue` must preserve exact final body result identity or `null`, even if the terminal condition mutates that object. Removing terminal-`false` `while` changes reflection, extraction, callers observing `false`, and lifetime/outer-Future adoption. Current `while` avoids retaining a last large body object; `whileTrue` may retain it through the terminal condition. No empirical memory/JIT claim is made.

**F5, control/scheduler:** condition evaluated after creation of a fresh zero-arg lexical Closure and on every activation (never eagerly before selecting `whileTrue`); preserve `this`, `super`, `context`, nonlocal `^` home, Error identity, no body invocation on `false`, no repetition after suspension, cancellation and `ensure` unwind, Future-result identity and outer-asynchronous adoption. No hidden checkpoint/new control institution. AST/source spans, parser origin facts, formatter, DAP and LSP must represent new syntax truthfully.

## Comparative precedent (primary sources; relevant differences)

| System | Real approach and lesson for Protos |
|---|---|
| [Self](https://handbook.selflanguage.org/2024.1/blocks.html) | Looping through ordinary block messages; aligns with retained `whileTrue`, not with overriding existing call syntax |
| [GNU Smalltalk](https://www.gnu.org/software/smalltalk/manual-base/gst-base.html) | `BlockClosure` condition/body loop messages; explicit block evaluation, selector surface vs results matter |
| [Io](https://iolanguage.com/guide/guide.html) | `while` message with unevaluated message arguments; unlike Protos eagerly evaluated call arguments, not directly transferable |
| [Ruby](https://docs.ruby-lang.org/en/master/syntax/control_expressions_rdoc.html) | Privileged while syntax, nil normal result; grammar-first loop, not an after-the-fact spelling collision |
| [Rust](https://doc.rust-lang.org/reference/expressions/loop-expr.html) | `while` expression returns unit; `loop` + break can carry value; distinct result policy from D201 `whileTrue` |
| [Kotlin](https://kotlinlang.org/docs/control-flow.html) | Dedicated while syntax but loop not a result-valued expression in the Protos sense |
| [Python](https://docs.python.org/3/reference/compound_stmts.html#the-while-statement) | Dedicated while statement + else clause, inappropriate to assume loop else or break for Protos |
| [Groovy](https://groovy-lang.org/dsls.html#_command_chains) | Command chains demonstrate surface brevity vs ambiguous precedence; D199 rejected similar unparenthesized sends |

Language comparison gives pressure cases, not a requirement to mimic another runtime. For this narrowly syntactic issue Truffle language implementations do **not** decide Protos semantics; Bytecode DSL compatibility must be tested locally and not inferred from GraalPy or GraalJS source shapes.

## GITHUB010 scorecard — viable complete designs

Candidates: **N** Q1 C + exact contextual `while(expr) { body }`, with intentional collision (selected); **G** Q1 C + distinct source form e.g. `while { cond } { body }` avoiding exact whole-expression call collision; **F** Q1 C and no sugar/tool snippets; **A** original D201 pair, no syntax. Scoring 1–5; H/M/L confidence. The table's qualitative hard gates outweigh numeric totals.

| Dimension | N (selected) | G (different shape) | F (defer syntax) | A (status quo) |
|---|---|---|---|---|
| 1 Correctness/invariants | 3 H: explicit ordinary-call exception | 4 M: avoids same full expression; prefix reinterpretation | 5 H: no grammar delta | 5 H: old semantics |
| 2 Protos alignment | 3 H: one contextual privileged spelling | 3 M: odd syntax, still privileged | 4 H: messages remain ordinary | 3 H: redundant standard result policies |
| 3 Pay-as-needed | 4 M: targeted surface, no new runtime | 3 M: extra parsing/form rules | 3 H: user pays source verbosity | 3 H: redundant selector |
| 4 Incremental growth | 4 M: existing protocol reused | 3 M: future conventional spelling difficult | 3 H: later compatibility cost | 3 H: still two selectors |
| 5 Future optionality | 3 M: one syntactic spelling captured | 4 M: fewer call clashes | 4 H: no new syntax promises | 3 M: extra public slot stays |
| 6 Scalability | 4 M: ordinary loop runtime | 4 M: same runtime | 4 H: existing runtime | 4 H: existing runtime |
| 7 Conceptual simplicity | 4 M: one syntactic rule/selector | 2 M: surprising twin-block form | 3 H: verbose condition Closure | 2 H: two selectors, differing return policy |
| 8 Portability | 4 H: semantic lowering platform-neutral | 4 H: possible platform-neutral lowering | 5 H: no new parser rule | 5 H: existing grammar |
| 9 Runtime/resources | 4 M: same loop, may retain last body | 4 M: same result retention | 4 M: same last-value retention | 4 M: distinct discard-only loop available |
| 10 Failure/operability | 3 M: rename/source-refactor hazard | 3 M: parse category of prefix can change | 4 H: current diagnostics | 4 H: stable current diagnostics |
| 11 Migration/deferral | 3 H: breaking `while(...){...}` accepted | 4 M: no full-form collision | 2 H: later break remains | 2 H: Q1 simplification deferred |
| 12 Evidence/risk | 3 M: grammar analyzed, untested implementation | 2 L: no prototype/test | 5 H: no implementation | 5 H: already implemented |

**Hard gates:** F and A fail the owner's explicit real-sugar and one-selector requirements; G fails the required concise conventional `while(expr)` surface and risks D199-style contextual prefix interpretation. N's hard source-compatibility red flag was explicitly presented and waived by the owner, **only for this exact grammar pattern**. The scorecard does not override that scope.

**Smallest sufficient implementation:** reuse the already admitted standard `whileTrue` native/prepared/structured/literal-inline mechanisms, remove only `Object.while` standard surface and dead return-policy branching, and add one narrowly matched source construct. No global keyword/macro engine/new executor, speculative generic syntax framework or extra scheduling/ownership subsystem.

**Cost of delay:** currently many standard-library/tool/example loops carry explicit `(() => cond).whileTrue` boilerplate. Delaying contextual syntax keeps that current cost and may enlarge eventual migration. Removing `Object.while` ahead of contextual sugar is an independently conformant slice, reducing the standard surface; implementing sugar then leverages the single selector.

**Growth/stress:** many actors/tasks/contexts or native-image runtimes use identical ordinary Closure-loop semantics; source parsing stays context-free with no global parser caches, runtime keyword map or distributed coordination. Largest risk is downstream source using `while(expr){...}` as ordinary call and tools (LSP/DAP/formatter) accidentally misclassifying it. Escape: grouping the callable or naming a custom member; authoritative docs must explicitly note incompatibility. No future extra Boolean/while else/break/continue/whileFalse approved.

## GITHUB021 invariant/delta check

| Authority | Selected D210 delta |
|---|---|
| D044 receiver/body semantic Closure, strict Boolean, effects/error/NLR/suspension | PRESERVED |
| D180 sole standard `whileTrue`, no `whileFalse` | RESTORED single selector; preserve no inverse |
| D180 no dedicated `while` syntax | EXPLICITLY REOPENED by owner Q2 |
| D201 added distinct `Object.while` returning terminal false | EXPLICITLY REOPENED and REMOVED by owner Q1 C |
| D201 `whileTrue` exact last result or zero-null | PRESERVED |
| Generic ordinary bare `while(condition){closure}` meaning | EXPLICITLY NARROWED under owner accepted special syntax; legacy form is a break |
| D051 `if(condition){body}` remains ordinary and D199 rejected undotted Boolean syntax | PRESERVED; no Boolean parser rewrite |
| Ordinary `.whileTrue` selection, `Object` delegation, shadowing/extraction | PRESERVED |
| Standard Error/Future/NLR/cancel/return-home and no truthiness | PRESERVED |

**Ratification gate:** owner-approved exact direction with bounded, already studied source collision; record may be published. **Implementation gate:** independent I-family, human-run tests/build/commit. User's reported clean diff and green local tests are not represented as D210 implementation verification. Other future semantic refinements need fresh approval.
