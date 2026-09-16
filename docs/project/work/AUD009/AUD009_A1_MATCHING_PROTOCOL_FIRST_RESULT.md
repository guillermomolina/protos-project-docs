# AUD009-A1 — Matching protocol-first second-pass result

Status: **NEEDS_USER_DECISION**

Nature: non-normative retrospective audit evidence and recommendation packet

Parent: `AUD009` / `guillermomolina/protos#522`
Execution issue: `AUD009-A1` / `guillermomolina/protos#535`
Semantic decision authority: `D131` / `guillermomolina/protos#503`
Methodology: `AUD009_A1_MATCHING_SECOND_PASS.md`
Protocol checkpoint: `AUD009_A1_MATCHING_PROTOCOL_DIRECTION_CHECKPOINT.md`
Supersedes as current recommendation: `AUD009_A1_MATCHING_REVIEW.md`
Protos baseline: `e8a74f29bcbb321c0308c932446ae3950824ed8f`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`
Specification changed: **NO**
Implementation changed: **NO**
Project-owner semantic approval: **PENDING through D131**

## Executive result

The resumed second pass no longer supports treating the existing
`subject match { case ... }` grammar as the fundamental matching mechanism.

The strongest recommendation is **Candidate B — protocol-first matching, without
dedicated matching grammar in Core v0.1 initially**.

The fundamental model remains the already-ratified ordinary matcher authority:

```text
pattern.match(subject)
```

with the current exact normal outcome carrier:

```text
false        -> mismatch
true         -> success with zero captures
[x, ...]     -> success with positional captures
```

Multi-way selection can then be one ordinary ordered-selection operation over
ordinary matcher values and lazy ordinary callable/Closure bodies. On a capture
result, the selected body consumes the capture vector through existing ordinary
call spread and Closure parameter binding.

Conceptually only, without selecting names or a public case representation:

```text
select(subject, orderedCases)

for each reached case in order:
    result = case.matcher.match(subject)

    false        -> continue
    true         -> invoke body()
    [captures]   -> invoke body(...captures)
    other normal -> Error

no selected case -> Error
```

This is a research model, not a ratified API. D131 must still select exact
ownership, case representation, evaluation timing, and public matcher/combinator
spellings.

The decisive simplification is that **capture names can belong to ordinary
Closure parameters rather than to a compiler-owned pattern sublanguage**.
A matcher returning `[left, right]` can be consumed by `(left, right) => ...`;
a dynamic capture tail can be consumed by `(head, ...rest) => ...`.

## Why D130 changes the fair baseline

The protocol checkpoint paused until D130 because ordinary ordered case data
written with `Array(...)` would artificially penalize the protocol approach.
D130/I039 is now published: `[...]` is pure ordinary-call sugar for `Array(...)`,
including spread. Matching can therefore use concise ordinary aggregate data
without inventing a matching-specific collection-construction mechanism.

D130 remains a sequencing dependency only, not a matching semantic dependency.

## Foundation to keep

### Matcher authority and default values

Keep one public matcher authority:

```text
pattern.match(subject)
```

and the standard inherited value behavior:

```text
Object.match(subject) -> this == subject
```

This preserves ordinary lookup, dispatch, effects, Error propagation, non-local
control, cancellation, suspension, and user override/shadowing.

### Matcher result carrier

Keep:

```text
false | true | non-empty standard Array
```

It is small, avoids allocation for zero-capture success, and feeds ordinary
callable invocation directly through call spread.

### Ordered selection capability

Keep the capability, not the current grammar:

- evaluate the subject once;
- consider cases in declared order;
- invoke each reached matcher once;
- `false` advances;
- first success commits;
- selected body result becomes the complete result;
- no selection signals ordinary Error unless an accepting final case exists;
- non-normal control propagates ordinarily;
- completed effects are not rolled back.

The audit recommends one standard ordinary abstraction for this repeated
consumer algorithm. Raw manual decoding at every call site is not sufficient.

### Ordinary body parameters own capture names

This removes the need for a second arm-binding ABI. Fixed and rest parameters,
arity validation, defaults and spread remain owned by normal Closure/callable
semantics.

## Composition instead of special interaction rules

### Guard/refinement

The capability is fundamental; current `when` syntax is not.

An ordinary guard matcher/combinator can invoke a nested matcher and, on success,
invoke a strict-Boolean predicate with the same capture vector. Canonical `true`
returns the original success; canonical `false` becomes mismatch; another normal
predicate result is Error.

Composition makes OR/guard behavior explicit:

```text
Guard(Or(a, b), g)
```

means first successful OR branch, then one guard; rejection does not reopen `b`.

```text
Or(Guard(a, g), Guard(b, g))
```

intentionally permits `b` after guarded failure of `a`.

### Ordered OR

Keep ordered alternative capability as an ordinary matcher combinator that
returns the first successful result unchanged.

Remove the matching-specific binding-name equivalence institution. The consuming
Closure owns parameter names and arity. Branches with incompatible positional
shapes either use separate outer cases/bodies or fail by ordinary invocation when
selected; the compiler need not synthesize one named arm interface.

### Whole-subject access / alias

The capability does not require `@whole: nestedPattern` syntax. At the outer
selection level the subject is already an ordinary lexical value available to a
body Closure. At a nested level an ordinary matcher wrapper can add the current
subsubject to a nested matcher's capture vector when genuinely useful.

## Structural matching under the protocol

Removing dedicated pattern syntax does not remove structural recognition.
A standard structural matcher can remain an ordinary object whose public
matching authority is still only `match(subject)`.

No exact constructor/selector names are selected by this audit.

### Array

Retain as capabilities:

- fixed standard-Array structural matching;
- exact fixed length when no remainder is requested;
- one **terminal** remainder capability for common head/rest decomposition;
- shallow pre-child observation so later mutation of the source Array cannot
  change the references already selected for the attempt;
- fresh shallow standard Array when a captured remainder value is required.

Do not retain initially:

- middle Array remainder;
- dedicated bare-rest syntax;
- mandatory freezing of the fresh remainder Array.

Middle slicing is specialized and can be added later without changing
`pattern.match(subject)`. Freshness already isolates the remainder from later
indexed mutation of the source; mandatory freezing adds policy without a
currently demonstrated fundamental need.

### Map

Retain as capabilities:

- normal standard-Map structural recognition;
- open/subset behavior as the basic mode;
- stable shallow association observation before effectful query/equality/nested
  matching can perturb the current attempt;
- ordinary mismatch for an absent required association.

Do not retain initially:

- exact Map mode;
- Map remainder capture;
- bare Map remainder discard;
- a fresh frozen Map-remainder aggregate institution.

Open/subset matching covers the basic structured-data need. Exactness and
remainder can be added later as ordinary matcher options/combinators without
changing the fundamental matcher authority.

Bare Map remainder discard is a permanent rejection under this model: open Map
matching already ignores unrelated associations, so it adds no capability.

### D136 boundary

D136 has independently ratified expression-position sequential Map construction:

```protos
%{
    key: value
}
```

AUD009-A1 does not reopen it. The protocol-first recommendation removes the
current pattern-position `%{...}` grammar with the rest of the dedicated pattern
surface, leaving D136 construction untouched. Any future Map-pattern sugar must
be re-evaluated against the then-current D136 construction contract rather than
restoring D095 by inertia.

## Evaluation timing remains a D131 design axis

A naïve ordinary data form such as:

```text
select(subject, [
    [matcherExpression1, body1],
    [matcherExpression2, body2]
])
```

would use ordinary eager argument/Array construction semantics. If D131 requires
matcher-producing expressions to remain lazy until their case is reached, case
descriptors can instead carry ordinary zero-argument producer Closures.

The audit does **not** select eager vs lazy matcher production. The important
finding is that both are expressible through existing Closure/call mechanisms;
dedicated pattern VM semantics is not required merely to support laziness.

## Second-pass classification matrix

These are AUD009-A1 proposals only; D131 remains the decision authority.

| Current mechanism | Retained capability / alternative | Proposed outcome |
| --- | --- | --- |
| `pattern.match(subject)` | Ordinary extensible recognition | `KEEP` |
| inherited `Object.match` | Ordinary `==` default | `KEEP` |
| `false / true / non-empty Array` carrier | Directly feeds ordinary calls | `KEEP` |
| source-order first-success selection | Standard ordinary selector | `KEEP` capability |
| postfix `subject match { ... }` | Ordinary selector + Arrays + Closures | `REMOVE_NOW_RECONSIDER_LATER` |
| `case` arm grammar | Ordinary case data/body Closure | `REMOVE_NOW_RECONSIDER_LATER` |
| `@name` binder syntax | Capture-any matcher + body parameter | `REMOVE_NOW_RECONSIDER_LATER` |
| `_` wildcard syntax | Accept-all matcher + body ignoring captures | `REMOVE_NOW_RECONSIDER_LATER` |
| guard/refinement capability | Ordinary guard combinator | `KEEP` capability |
| `when` syntax | Ordinary guard combinator | `REMOVE_NOW_RECONSIDER_LATER` |
| guard/body `=>` delimiter rule | Ordinary Closure/call grammar | `REMOVE_PERMANENTLY` |
| fixed Array structural recognition | Standard ordinary Array matcher | `KEEP` capability |
| bracket Array-pattern grammar | Ordinary matcher construction | `REMOVE_NOW_RECONSIDER_LATER` |
| terminal Array remainder capability | Standard Array matcher option | `KEEP` capability |
| middle Array remainder | Later extension | `REMOVE_NOW_RECONSIDER_LATER` |
| bare Array remainder syntax | Remainder matcher with ignored result | `REMOVE_NOW_RECONSIDER_LATER` |
| shallow Array observation | Encapsulated correctness rule | `KEEP` |
| fresh captured Array remainder | Fresh shallow standard Array | `KEEP` |
| forced frozen Array remainder | Freshness without freeze | `REMOVE_NOW_RECONSIDER_LATER` |
| open/subset Map recognition | Standard ordinary Map matcher | `KEEP` capability |
| `%{...}` Map-pattern grammar | Ordinary matcher construction | `REMOVE_NOW_RECONSIDER_LATER` |
| stable Map association observation | Encapsulated correctness rule | `KEEP` |
| exact Map mode | Later matcher option/combinator | `REMOVE_NOW_RECONSIDER_LATER` |
| Map remainder capture | Later matcher capability | `REMOVE_NOW_RECONSIDER_LATER` |
| bare Map remainder discard | Open/subset already ignores residue | `REMOVE_PERMANENTLY` |
| frozen Map remainder aggregate | No initial remainder need | `REMOVE_NOW_RECONSIDER_LATER` |
| alias syntax | Lexical subject or matcher wrapper | `REMOVE_NOW_RECONSIDER_LATER` |
| ordered OR capability | Ordinary OR matcher | `KEEP` capability |
| `|` pattern syntax | Ordinary OR matcher | `REMOVE_NOW_RECONSIDER_LATER` |
| OR binding-name/interface equivalence | Closure parameters own names/arity | `REMOVE_PERMANENTLY` |
| fixed `captures(a,b)` | `(a,b) => ...` + spread | `REMOVE_PERMANENTLY` |
| dynamic `captures(first,...rest)` | `(first,...rest) => ...` + spread | `REMOVE_PERMANENTLY` |
| D103 complete-arm dynamic-rest terminality | Ordinary Closure rest ABI | `REMOVE_PERMANENTLY` |
| matching-specific arm-binding ABI | Ordinary callable invocation | `REMOVE_PERMANENTLY` |
| tri-state coverage framework | Future syntax-local analysis if needed | `REMOVE_NOW_RECONSIDER_LATER` |
| structural arm unreachability source errors | No equivalent for ordinary dynamic combinators | `REMOVE_NOW_RECONSIDER_LATER` |

## Reconsideration triggers

- **Dedicated match sugar:** repeated real programs show the ordinary protocol is
  materially verbose, obscures intent, or causes recurring mistakes. New sugar
  must start from the then-current protocol and justify its lowering.
- **Binder/wildcard sugar:** ordinary match-any/accept-any matcher values plus
  Closure parameters are demonstrably noisy in common source.
- **Array/Map pattern sugar:** ordinary standard matcher APIs have real use and
  stable semantics. D136 remains a fixed neighboring constraint for Map syntax.
- **Middle Array remainder:** common programs repeatedly need simultaneous
  prefix/middle/suffix decomposition.
- **Exact Map:** boundary/validation code repeatedly needs proof that no
  unselected associations exist.
- **Map remainder:** programs repeatedly need selected fields plus a residual Map
  as one recognition operation.
- **Frozen remainder aggregates:** concrete correctness/security/concurrency
  evidence shows that a fresh residual value must also reject downstream mutation.
- **Static coverage:** future dedicated syntax or another closed static pattern
  representation makes useful sound coverage analysis possible without pretending
  arbitrary matcher objects are closed-world.

## Permanent-rejection rationale

- **Bare Map remainder discard:** duplicate of open/subset behavior.
- **Guard/body `=>` delimiter institution:** exists only because current guard
  syntax collides with existing Closure `=>`; ordinary composition removes it.
- **OR binding-name equivalence:** exists only because pattern syntax owns names;
  ordinary Closure parameters supersede it.
- **`captures(...)`:** exists only to map positional matcher results into
  compiler-owned arm names; body parameters + spread already do that.
- **D103 complete-arm terminality:** repairs dynamic match-capture composition;
  ordinary Closure rest binding already owns the corresponding rule.
- **Matching-specific arm-binding ABI:** duplicates ordinary callable invocation.

## Comparative evidence

The resumed pass compares three materially different design families.

1. **Message/block control — Self, Io.** Self implements control structures using
   blocks and messages; Io expresses `if`/`for` as normal messages with selective
   argument evaluation. These are strong prototype/message-oriented precedents
   for keeping control mechanisms ordinary.
2. **Extensible extractor protocol + dedicated pattern syntax — Scala 3, F#.**
   Scala's `unapply`/`unapplySeq` and F# Active Patterns show the durable value of
   user-extensible recognition/extraction, while also showing that the consuming
   pattern syntax is a separable layer.
3. **Rich compiler-owned pattern algebra — Python, Rust.** PEP 634 and Rust define
   dedicated match/arm/pattern/guard rules, including binding and OR interactions.
   They demonstrate strong ergonomics but also the exact continuing grammar and
   interaction costs that Protos should pay only if ordinary protocol use proves
   them necessary.

Primary references:

- Self blocks/control: `https://handbook.selflanguage.org/2024.1/blocks.html`
- Io Guide: `https://iolanguage.org/docs/Guide/index.html`
- Scala 3 extractors: `https://docs.scala-lang.org/scala3/reference/changed-features/pattern-matching.html`
- F# Active Patterns: `https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/active-patterns`
- Python PEP 634: `https://peps.python.org/pep-0634/`
- Rust match expressions: `https://doc.rust-lang.org/reference/expressions/match-expr.html`
- Rust patterns: `https://doc.rust-lang.org/reference/patterns.html`

The previous AUD009-A1 packet contains the broader Rust/Scala/Python/C#/Java/
Swift/Kotlin/Dart/Ruby/Erlang/Elixir/OCaml/F#/Haskell/Racket/Clojure evidence.
This resumed pass uses that material but changes the selection question from
"which current features are conventional?" to "which capabilities still need a
special Protos institution after an ordinary protocol exists?"

## Whole-model candidate set

### A — retain the current rich grammar

Keep the postfix envelope and most D095/D096 machinery, with only narrow
removals.

Strength: polished compact syntax already implemented.
Weakness: continues a second source-binding/control sublanguage.
Red flag: **overengineering** after ordinary protocol composition proves most of
its semantic work is not fundamental.

### B — protocol-first, no dedicated matching grammar initially — RECOMMENDED

Keep matcher authority/carrier and provide the smallest ordinary ordered
selection plus standard matcher/combinator capability. Selected bodies are
ordinary callables consuming capture Arrays through ordinary spread.

Strength: smallest complete model; maximum Protos alignment; future sugar remains
cheap to add.
Weakness: exact ordinary case/selector API still requires D131 design and initial
source is less compact than purpose-built syntax.
Red flag: none if D131 supplies a genuinely usable ordinary selector.

### C — protocol-first semantics plus dedicated sugar immediately

Define B and also retain/redesign compact match syntax in the same decision.

Strength: clean foundation plus immediate ergonomic syntax.
Weakness: commits grammar before real protocol use identifies exactly what sugar
is worth keeping.
Red flag: **present-need proportionality**; deferring sugar has low foundational
cost.

### D — raw `pattern.match(subject)` only

Remove dedicated grammar but add no standard ordered-selection consumer.

Strength: absolute minimum mechanism.
Weakness: every caller repeats outcome validation, ordering, capture spread and
terminal-no-selection behavior.
Red flag: **underengineering**; common protocol consumption belongs behind one
standard ordinary abstraction.

## GITHUB010 scorecard

Scores are 1–5; totals are advisory. Confidence is HIGH except Candidate B's
exact public API ergonomics, which is MEDIUM until D131 settles and exercises the
ordinary surface.

| Criterion | A | B | C | D |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariants | 5 — already defined | 5 — preserves authority/carrier | 5 — same foundation | 4.5 — sound primitive, duplicated callers |
| Protos alignment | 2.5 — special sublanguage | 5 — ordinary mechanisms | 4.5 — ordinary base + sugar | 5 — entirely ordinary |
| Present-need proportionality | 2 — excess current surface | 5 — fundamental only | 3.5 — premature sugar | 2.5 — user pays boilerplate |
| Incremental growth | 3 — grammar constrains growth | 5 — combinators/sugar add independently | 4.5 — sugar adds constraint | 5 — easy layering |
| Future-option resilience | 3 — syntax/ABI lock-in | 5 — preserves options | 4.5 — some syntax lock-in | 5 — maximal options |
| Scalability | 4.5 — taxonomy grows centrally | 4.5 — object composition | 4.5 — same runtime base | 4.5 — runtime fine, source scales poorly |
| Conceptual simplicity | 2 — special interactions | 5 — one authority + calls | 3.5 — protocol + syntax | 4.5 — primitive simple, repeated consumers |
| Portability / freedom | 5 — backend-neutral | 5 — language-level | 5 — lowering-neutral | 5 — language-level |
| Runtime / resource cost | 4 — special paths optimizable | 4.5 — ordinary paths specialize | 4.5 — same as B | 5 — no extra layer |
| Failure / operability | 4 — exact but many special sites | 4.5 — one standard consumer | 4.5 — same as B | 3 — callers can mishandle outcomes |
| Deferral / reversibility | 2.5 — grammar costly to remove | 5 — sugar cheap later | 3 — premature compatibility | 5 — selector cheap later |
| Evidence / risk | 5 — implemented | 4.5 — strong evidence, API pending | 4.5 — familiar but surface unsettled | 4 — mechanically easy, weak usability |
| **Total / 60** | **42.5** | **58.0** | **51.5** | **53.0** |

D's high arithmetic score does not defeat its underengineering red flag; C's good
score does not defeat its speculative-syntax red flag. The scorecard is evidence,
not authority.

## Incremental-design gate

**Smallest sufficient solution:** open `pattern.match(subject)`, the exact small
result carrier, one standard ordinary ordered-selection consumer, ordinary
callable invocation/spread, and only the standard matcher/combinator capabilities
justified above.

**Pay for what is needed:** users pay for recognition, selection, extraction and
basic structural matching, not a compiler-owned pattern grammar, binding ABI,
coverage solver, exact/remainder modes or dynamic capture-interface rules they do
not currently need.

**Grow as needed:** guards, OR and structural matchers are ordinary composition;
future syntax can lower onto the protocol; exact/remainder features can extend
standard matchers later without changing matcher authority.

**Cost of deferring syntax:** low. No fundamental semantic rewrite is required to
add focused syntax later once the protocol is authoritative.

**Continuing cost avoided now:** Surface/Canonical pattern taxonomies, dedicated
AST/Bytecode matching paths, special arm-binding construction, `captures(...)`,
D103 terminality, OR binding-interface checks, guard delimiter disambiguation,
coverage/unreachability analysis, tooling awareness and the interaction
conformance matrix.

## Strongest argument against B and escape path

Dedicated pattern syntax can represent nested recognition trees more readably
than nested matcher construction. Python, Rust, Scala and F# all demonstrate
that real ergonomic value.

If ordinary Protos usage demonstrates the same pressure, add focused syntax
later. The escape path is deliberately cheap because Candidate B is already the
lowering target. User-defined matchers and the runtime authority need not migrate.

## Recommendation pending explicit D131 approval

Recommend **Candidate B — protocol-first matching with no dedicated matching
grammar initially**.

```text
KEEP AS FUNDAMENTAL
    pattern.match(subject)
    Object.match(subject)
    exact matcher result carrier
    ordered first-success selection capability
    ordinary selected-body invocation with capture spread
    guard/refinement capability as composition
    ordered OR capability as composition
    fixed Array structural matching
    terminal Array remainder capability
    Array shallow observation
    open/subset Map structural matching
    Map stable association observation

REMOVE NOW / RECONSIDER LATER
    postfix match/case syntax
    case grammar
    @ binder syntax
    _ pattern syntax
    when syntax
    Array-pattern syntax
    middle/bare Array remainder syntax
    forced frozen Array remainder
    Map-pattern syntax
    exact Map
    Map remainder capture/remainder aggregate
    alias syntax
    | pattern syntax
    tri-state coverage / structural arm diagnostics

REMOVE PERMANENTLY UNDER THIS MODEL
    bare Map remainder discard
    guard/body => delimiter institution
    OR binding-name/interface equivalence
    captures(a,b)
    captures(first,...rest)
    D103 complete-arm dynamic-rest terminality
    matching-specific selected-arm binding ABI
```

No part of this recommendation is ratified by this document.

## D131 questions remaining if Candidate B is selected

D131 must still settle before implementation:

1. ownership and exact semantics of the ordinary ordered-selection operation;
2. exact case descriptor / matcher+body representation;
3. eager matcher values vs lazy matcher-producer Closures and exact evaluation
   timing;
4. exact standard matcher/combinator public spellings and ownership;
5. exact terminal Array remainder result policy, including proposed removal of
   mandatory freezing;
6. exact retained Map matcher construction and stable-observation contract;
7. deimplementation/migration order for D093/D095/D096/D103 and I038;
8. required spec, guide, conformance, parser, canonical AST, AST/Bytecode and
   tooling reconciliation.

AUD009-A1 deliberately leaves those as D131 semantic/API decisions rather than
smuggling them into the audit.

## Relationship to AUD004

AUD004 should audit the **post-D131 reconciled model**. It should not spend
closure-audit effort proving complete implementation of mechanisms D131 may
remove. After D131's selected model is implemented, AUD004 remains the independent
end-to-end traceability audit.

## Validation

This record is governance/documentation evidence only. It changes no Protos
specification, parser, runtime, library behavior, implementation version, or
public matching contract.
