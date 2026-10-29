# D221 — Brace/Closure shorthand: source audit, alternatives and falsification

**Research:** 2026-10-29. **Owner selection:** D221-A, existing grammar unchanged.
**Owning Issue:** [D221/#984](https://github.com/guillermomolina/protos/issues/984).
**Decision record:** [D221 — preserve explicit zero-argument Closures](../../decisions/language/D221_PRESERVE_EXPLICIT_ZERO_ARGUMENT_CLOSURES.md).
**Exact Protos source baseline:** [ce6cdec09a615a31edf668e22bd022be6cac2626](https://github.com/guillermomolina/protos/commit/ce6cdec09a615a31edf668e22bd022be6cac2626) (2026-10-28).
**Evidence class:** static source/specification comparison and cross-language references, **not** executable or test evidence.

## Source and authority inventory

- Normative: [`spec/PROTOS_GRAMMAR.md`](https://github.com/guillermomolina/protos/blob/ce6cdec09a615a31edf668e22bd022be6cac2626/spec/PROTOS_GRAMMAR.md), [`spec/PROTOS_LANGUAGE_SPEC.md`](https://github.com/guillermomolina/protos/blob/ce6cdec09a615a31edf668e22bd022be6cac2626/spec/PROTOS_LANGUAGE_SPEC.md), `spec/semantics/OBJECT_MODEL.md`, `spec/semantics/CALLABLES.md` and `spec/semantics/EXECUTION_AND_CONTROL.md`.
- Parser: [`ProtosParser.java`](https://github.com/guillermomolina/protos/blob/ce6cdec09a615a31edf668e22bd022be6cac2626/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java), [`TokenCursor.java`](https://github.com/guillermomolina/protos/blob/ce6cdec09a615a31edf668e22bd022be6cac2626/src/main/java/com/guillermomolina/protos/parser/TokenCursor.java).
- Canonicalizer: [`Canonicalizer.java`](https://github.com/guillermomolina/protos/blob/ce6cdec09a615a31edf668e22bd022be6cac2626/src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java).
- Tool/source-fact boundaries: `ProtosParserSourceFacts.java`, `ProtosSourceLayoutView.java`, `ProtosWholeDocumentFormatter.java`, `protos/tools/formatter/Structural.protos`.
- Targeted existing tests: `ProtosParserTrailingClosureTest.java`, `ProtosParserSourceFactsTest.java`, `CanonicalizerCallTest.java`, `ProtosI121ContextualUntilCanonicalizationTest.java`, `ProtosCallableSurfaceNegativeFixtureTest.java`, `ProtosFormatterToolBootstrapTest.java`; tutorial `protos/tutorials/04-control-flow/03-post-test-loops.protos`.
- Earlier design decisions: D013 ordinary invocation; [D199](../../decisions/language/D199_FLUENT_BOOLEAN_SYNTAX_NO_CHANGE.md), [D210](../../decisions/language/D210_CLOSURE_WHILE_SELECTOR_AND_CONTEXTUAL_SYNTAX.md), [D211](../../decisions/language/D211_CONTEXTUAL_IF_BUNDLE.md), [D212](../../decisions/language/D212_POST_TEST_UNTIL_AND_WHILE_FALSE.md), [D203](../../decisions/language/D203_FLEXIBLE_CLOSURE_ARGUMENT_BINDING.md).

The grammar distinguishes Object construction `{ object-body-items }` from a lexical Closure `(parameters) => { expression-sequence }`. Because an Object body accepts ordinary expressions, valid braced-Closure expression content can also be valid Object-body content: contents alone cannot disambiguate the two kinds. `ProtosParser.parsePrimaryFoundation()` branches on `LBRACE` to `parseContextualUntil()` **only when** `TokenCursor.bracedBodyFollowedByContextualUntil()` matches; otherwise it selects `parseObjectBody(null)`.

The trailing Closure parser recognizes the existing braced final argument after a completed parenthesized call, while contextual `if`, `while` and suffix `until` have separate narrow recognition rules. Logical NEWLINE separates expressions and changes attachment; line comments can introduce that boundary, while an inline block comment without a logical newline need not. These exceptions are syntactic, never inferred from the receiver's runtime type or selector meaning.

## Current-grammar admissibility matrix

| Source position / example | Current parse/semantics | Brace as Closure? | D221 outcome |
| --- | --- | --- | --- |
| `x: { 42 }` / `x = { 42 }` | Object expression | No | Preserve |
| `f({ 42 })` / `f(a, { 42 })` | Object argument | No | Preserve |
| Standalone `{ 42 }` | Object expression | No | Preserve |
| `{ 42 }.someSelector()` | Send to Object | No | Preserve |
| `{ 42 }()` | Ordinary polymorphic Object invocation | No | Preserve |
| `({ 42 })` | Grouped Object | No | Preserve |
| `parent { x: 1 }` | Object with parent | No | Preserve |
| `f() { 42 }` | Trailing zero-arg Closure final argument | Yes | Existing |
| `f()` NEWLINE `{ 42 }` | Detached ordinary Object expression | No | Existing |
| `if(c) { a } else { b }` | Exact contextual if/else | Branch bodies as Closures | Existing |
| `while(c) { a }` | Exact contextual pre-test loop | Condition and body as Closures | Existing |
| `{ a } until(c)` without separating logical NEWLINE | Exact contextual post-test loop | Body and predicate as Closures | Existing |
| `{ a }` NEWLINE `until(c)` | Separate ordinary expressions | No | Existing |
| `(() => { a }).untilFalse() { c }` | Explicit body Closure + trailing predicate Closure | Yes, only in those roles | Existing |
| `{ a }.untilFalse({ c })` | Object receiver plus Object argument | No | Do not change |
| `^result` inside a Closure | Lexical nonlocal return | Explicit Closure | Existing |
| Nested braced forms | Determined by enclosing productions | No global inference | Preserve |

Exactly one trailing Closure may attach under the present call syntax; newline and comments must preserve formatter round-trip boundaries. A braced item with Object-only composition syntax is not automatically a legal Closure body, even in the special contextual-until case.

## Minimal falsifications and required invariants

1. `x: { 42 }` and `f({ 42 })` are ordinary Object values today. An inferred or global rule silently alters valid programs.
2. `holder.untilTrue({ x: 1 })` may dispatch to a user-defined selector receiving an Object. Selector-based reinterpretation breaks normal dynamic sends and overrides.
3. `{ body }.untilTrue({ cond })` and `{ body }.untilFalse({ cond })` do not provide semantic Closures to the standard loop protocols. Treating them as correct loops would be a new syntactic/semantic rule.
4. `{ body }()` follows the ordinary Object `call` contract; `({ body })` is still an Object. Invocation syntax and grouping cannot convert a value kind.
5. The approved `{ body } until(cond)` is precisely D212's narrow syntactic exception, not evidence of a generic anonymous Closure literal.
6. New source sugar would have to lower to the exact existing zero-parameter canonical Closure, retaining creation versus execution, fresh identity, capture by reference, lexical `this`/`methodHome`, flexible D203 activation/defaults, D013 `call` selection, nonlocal return home, evaluation order, Error, Future, suspension and cancellation semantics.
7. Any new grammar would affect parser/source spans, formatter comment ownership, LSP, debugging and error diagnostics; no changes are authorized by this selection.

## Cross-language comparative evidence

| Language and primary reference | Relevant design | Applicability |
| --- | --- | --- |
| [Pharo / Smalltalk](https://books.pharo.org/updated-pharo-by-example/) | `[ ... ]` block versus separately delimited collection syntax; lexical and return-home model | Separate delimiters avoid Protos's bare-brace overlap |
| [Ruby](https://docs.ruby-lang.org/en/master/syntax/methods_rdoc.html) | Attached method blocks `f { ... }`, explicit `proc { ... }`, separate hash syntax | Block attachment and proc construction carry context; cannot assume identical Protos call semantics |
| [Swift](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures/) | `{ ... }` closure expressions and trailing closures | Ordinary Object construction does not compete for the same primary-expression braces |
| [Kotlin](https://kotlinlang.org/docs/lambdas.html) | `{ ... }` lambdas and trailing lambdas; distinct `object` construction | Different initial grammar; omitted parameter list is not generally proof of zero arity (implicit `it`) |
| [JavaScript](https://tc39.es/ecma262/#sec-arrow-function-definitions) | `() => { ... }` body; parenthesized object expression `() => ({ ... })` | Distinct expression versus body positions resolve apparent brace overlap |
| [Lua](https://www.lua.org/manual/5.4/manual.html#3.4.9) | `{ ... }` table constructor; `function() ... end` closure; `f{ ... }` table argument | Close cautionary precedent that bare braces can remain data |

These six systems span message/block-oriented, dynamic, statically typed and table/data-construction families. Syntax alone is insufficient: identity, lexical capture, nonlocal returns, invocation and runtime dispatch differ.

## Candidate comparison and GITHUB010 scores

- **A — Current grammar:** explicit `() => { ... }` plus existing contextual/trailing forms. **Owner-selected.**
- **B — New explicit marker:** a future `=> { ... }` zero-parameter syntax. Unambiguous but trivial punctuation savings and an additional frontend rule; not selected.
- **C — Selector-sensitive shorthand:** treat braces as Closure when beside a specific selector. **Disqualified** by existing dynamic sends.
- **D — Type/content inference:** infer Closure from expected use or body. **Disqualified** by grammar overlap and dynamic dispatch.
- **E — Global bare-brace-as-Closure:** replace Object interpretation of `{ ... }`. **Disqualified** as a source-wide compatibility break.

Scores 1–5 (5 favorable), followed by evidence confidence H/M/L and a short rationale. They are qualitative assessments, not performance measurements. Non-compensating compatibility failures override any numerical total.

| Dimension | A status quo | B explicit marker | C selector-sensitive | D inferred | E global |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness/invariants | 5H existing | 5M explicit | 1H breaks sends | 1H ambiguous | 1H breaks objects |
| 2 Protos alignment | 5H ordinary values | 3M extra rule | 1H privileged names | 1H hidden inference | 1H loses object base |
| 3 Present-need cost | 5H zero new cost | 2H tiny saving | 2H complexity | 1H semantic burden | 1H migration |
| 4 Incremental growth | 5H additive option | 4M extensible | 2M special cases | 1H brittle model | 1H redesign |
| 5 Future-option resilience | 5H unrestricted | 4M explicit future | 2M selector traps | 1H inference trap | 1H brace conflict |
| 6 Scalability | 5H existing | 5M local frontend | 3M name growth | 2M inference | 4M simple parse, broken apps |
| 7 Conceptual simplicity | 5H one boundary | 3H new surface | 1H selector magic | 1H implicit semantics | 2H superficially small |
| 8 Portability/freedom | 5H portable | 5M frontend only | 3M dispatch coupling | 1M contextual engine | 4H portable but incompatible |
| 9 Runtime/resource cost | 5H unchanged | 5M likely neutral | 3L guard cost | 2L unknown inference | 4L unspecified migrations |
| 10 Failure/operability | 5H known | 4M clear marker | 2M surprising dispatch | 1H hard errors | 1H widespread break |
| 11 Deferral/reversibility | 5H additive later | 4M reversible | 1H existing breaks | 1H existing breaks | 1H source migration |
| 12 Evidence/implementation risk | 5H present code | 4M viable grammar | 1H counterexamples | 1H body overlap | 1H direct conflicts |

**Hard gates:** C, D and E are incompatible with required ordinary Object and dispatch semantics. B remains technically credible, but no current workload demonstrates a benefit proportional to parsing, canonicalizer, formatter, diagnostics and language-surface maintenance. A is the smallest sufficient design.

**Pay for what you need:** A charges no new syntax/tooling/compatibility costs today. **Grow as you need:** a separately approved, non-conflicting explicit marker remains available. **Deferral cost:** additive syntax could be introduced later without changing existing Object or Closure syntax. **Strongest argument against A:** verbose standalone zero-parameter Closure literals persist, particularly for explicit post-test body receivers. This ergonomics cost is visible but does not justify breaking Object expressions or adding a nearly equivalent marker now.

## Future stress, GITHUB021 and validation boundary

More Closure-heavy libraries, Tasks, Actors, Process isolation, alternate schedulers, Truffle backends and Native execution do not remove this lexical overlap. Any future syntactic candidate must preserve the existing canonical value semantics across those contexts; runtime specialization does not establish grammar authority.

**GITHUB021 check:** D199 no undotted generalization, D210 bounded contextual `while`, D211 bounded `if/else`, D212 bounded suffix `until` and ordinary bare-brace fallback, D013/D203 invocation and Closure parameters, capture/NLR/error/cancellation all remain unchanged. No explicit prior owner-approved invariant was reopened, and no new consequence was added after the D221-A recommendation.

**Publication scope:** documentation-only ratification/evidence; no new `Ixxx`, normative specification edit, Protos commit, compilation, test execution or performance claim.
