# AUD009-A1 — Matching surface complexity and necessity review

Status: **COMPLETE — RECONCILED WITH RATIFIED D131 CANDIDATE C**

Nature: non-normative retrospective audit evidence and recommendation packet

Parent: `AUD009` / `guillermomolina/protos#522`

Execution issue: `AUD009-A1` / `guillermomolina/protos#535`

Semantic decision authority: `D131` / `guillermomolina/protos#503`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

Specification changed: **NO**

Implementation changed: **NO**

Audit date: 2026-09-16

## Purpose

Audit the current Core v0.1 matching surface under the AUD008/AUD009 retrospective methodology and produce evidence-backed proposed classifications without selecting replacement semantics inside AUD009.

D131 remains the semantic decision authority. This record supplies inventory, ongoing-cost evidence, comparative evidence, proposed AUD009 classifications, and follow-up routing. No proposal below becomes project policy until explicitly approved by the project owner through D131 or another required authority.

## Executive result

The audit does **not** support the original simplifying assumption that most advanced-looking matching forms should be removed merely because they have visible machinery.

Three findings dominate:

1. A large part of the matching surface is a coherent, familiar algebra whose complexity is either small in isolation or already required by capabilities with clear programmer value: aliases, OR, fixed structural patterns, Array rest, open/exact Map matching, Map rest and conservative coverage all have substantial precedent and bounded incremental cost.
2. Fixed custom-matcher capture naming (`captures(a, b)`) is unusual syntax, but it is the bridge that keeps arbitrary ordinary `pattern.match(subject)` objects capable of acting as **extractors**, not merely predicates. Removing it while retaining only built-in Array/Map binders would make user-defined matchers materially less expressive and would privilege built-in structural forms.
3. Two mechanisms fail the current-necessity test:
   - dynamic opaque capture rest (`captures(first, ...rest)`) and its complete-arm terminality rule create disproportionate cross-feature binding machinery for a capability with no demonstrated current requirement;
   - bare Map remainder discard (`...` with no nested pattern) is semantically redundant because normal Map patterns are already open/subset by default.

The proposed classification therefore retains the matching model substantially intact while removing one speculative capability and one duplicate surface form.

## Current repository evidence

### Surface and grammar

The current grammar exposes:

- postfix expression-valued `subject match { case ... }`;
- contextual `match`, `case`, and `when`;
- ordinary matcher/value patterns;
- `@name` and `_`;
- aliases;
- OR patterns;
- fixed and dynamic `captures(...)` interfaces;
- fixed, terminal-rest and middle-rest Array patterns;
- open, exact and remainder Map patterns;
- guard/body `=>` disambiguation;
- static linearity, irrefutability and dynamic-rest terminality restrictions.

This is genuine programmer-facing surface, not merely hidden runtime machinery.

### Repeated implementation taxonomy

The same conceptual pattern taxonomy is represented in multiple implementation layers:

- `SurfaceMatchPattern` contains Binder, Wildcard, Alias, Group, Or, Value/CaptureInterface, ArrayPattern and MapPattern;
- `CanonicalMatchPattern` mirrors Binder, Wildcard, Alias, Or, Value/CaptureInterface, ArrayPattern and MapPattern;
- the retained AST execution backend (`ProtosMatchNode`) independently implements the same families;
- `CanonicalToBytecodeLowerer` implements the production Bytecode path for the same canonical families;
- parser and conformance suites retain source-validity and interaction cases.

This duplication is an ongoing implementation cost. It is also partly attributable to the independent AUD009-G AST-execution question; AUD009-A1 therefore does not count all AST duplication as an argument against the public matching model itself.

### Array and Map cost is mostly pay-for-use

Array remainder allocation occurs only when a remainder is semantically captured. A bare remainder can avoid the aggregate allocation. Map matching already establishes its stable association snapshot for the standard Map contract; exactness is then a bounded selected-association check, and a fresh frozen remainder Map is materialized only when a remainder is actually captured.

Consequently, exact Map and Map remainder add public/conformance surface but do not create an unrelated always-on runtime subsystem.

### Static coverage is smaller than the surface name suggests

`MatchCoverageAnalyzer` is intentionally conservative and resource-bounded. The current implementation can prove syntactic irrefutability, can identify a limited non-Array/non-Map witness, and otherwise returns `UNKNOWN`; budget exhaustion also returns `UNKNOWN`.

D096 explicitly selected this fail-to-UNKNOWN model so arbitrary matcher objects remain opaque. The audit therefore rejects the characterization of current coverage as a large closed-world exhaustiveness solver.

### Dynamic capture rest is a demonstrated interaction multiplier

D103 exists because `captures(...rest)` composed with later structural bindings cannot map mechanically onto the ordinary Closure positional/rest ABI. The ratified solution requires the dynamic capture segment to be terminal in the **complete final logical arm-binding order**, not merely terminal inside the local `captures(...)` spelling.

That rule reaches parser validation, canonical representation, OR compatibility, structural pattern composition, arm invocation and conformance. This is concrete evidence of continuing complexity caused by a future-oriented capability.

### Repository-use signal

Current repository code-search evidence finds matching most strongly in the specification, guide, conformance corpus and implementation itself. No strong current production-library/tool use of the advanced forms was established by this audit. Because matching is newly implemented, absence of repository production use is treated as weak evidence rather than as proof of low external value.

## Comparative evidence

Primary/reference sources consulted include:

- Python PEP 634 — https://peps.python.org/pep-0634/
- Rust Reference, Patterns — https://doc.rust-lang.org/reference/patterns.html
- Scala 3 extractors — https://docs.scala-lang.org/scala3/reference/changed-features/pattern-matching.html
- C# pattern matching — https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/functional/pattern-matching
- Dart patterns — https://dart.dev/language/patterns and https://dart.dev/language/pattern-types
- Ruby pattern matching — https://ruby-doc.org/3.3.0/syntax/pattern_matching_rdoc.html
- Swift patterns/control flow — https://docs.swift.org/swift-book/ReferenceManual/Patterns.html and the Swift control-flow reference
- Kotlin `when`/guards — https://kotlinlang.org/docs/control-flow.html
- Erlang expressions/patterns — https://www.erlang.org/doc/system/expressions.html
- Elixir patterns and guards — https://hexdocs.pm/elixir/patterns-and-guards.html
- F# pattern matching and Active Patterns — Microsoft Learn
- Haskell 2010 Report, expressions/patterns — https://www.haskell.org/onlinereport/haskell2010/haskellch3.html
- Racket `match` — https://docs.racket-lang.org/reference/match.html
- Clojure `core.match` as a library/macro contrast.

Smalltalk, Self and Io are retained as object/message/control-flow philosophy contrasts rather than treated as direct feature-by-feature matching precedents. Java, C#, Swift and Kotlin provide useful negative/typed-world evidence where their matching models deliberately do not expose a general Protos-style arbitrary matcher-result ABI.

### Cross-system lessons

**Outer match/case, binders, wildcard and guards** are mainstream primitives across otherwise very different pattern systems. Source-order selection and an explicit no-selection outcome are also conventional. This is strong evidence that the basic construct is not a Protos-specific institution.

**Aliases are common and cheap.** Python `as`, Rust `@`, F# `as`, Haskell as-patterns and Erlang's compound matching all support retaining the whole matched value while destructuring it. Protos alias adds one ordinary binding and very little runtime machinery.

**OR patterns are common, but binding compatibility is a real invariant.** Python and Rust require consistent bindings across alternatives; F# exposes OR directly; Dart has logical-or patterns. Ruby is valuable negative evidence: ordinary variable binding is prohibited inside alternatives rather than permitting branch-dependent binding state. Protos D090's stable logical arm interface therefore reflects a recurring problem, not an invented one.

**Middle sequence rest is not exotic.** Python permits its single star pattern at any position; Dart supports list rest; C# list/slice patterns can place a slice among fixed elements. Once Protos already owns prefix + one rest + suffix semantics, terminal-only restriction would save less machinery than originally suspected.

**Open Map + remainder has strong precedent.** Python mapping patterns are subset/open by default and may capture `**rest`. Ruby supports Hash rest capture and `**nil` for exactness. Racket exposes open, closed and residue modes. Exactness and remainder are therefore coherent optional capabilities rather than unusual special cases.

**Bare Map remainder discard is different.** In Protos the Map is already open/subset when no remainder is written. Bare `...` therefore expresses no additional matching capability, unlike bare Array remainder where it changes fixed-length matching into variable-length matching.

**Custom extractors need a source-to-binding bridge.** Scala `unapply`/`unapplySeq`, F# Active Patterns, Ruby `deconstruct`/`deconstruct_keys`, and Racket app/match-expander mechanisms all provide a way for user extension points to expose decomposed values to pattern bindings. Protos uses the opposite ownership direction (`pattern.match(subject)`), but fixed `captures(a,b)` is currently the source bridge that makes returned positional captures useful.

**Variadic custom extractor capture is less foundational.** Systems supporting variadic extraction usually integrate it into a richer compiler-owned extractor/pattern grammar. Protos's dynamic capture Array plus ordinary Closure ABI creates a special composition problem that D103 had to solve. The external evidence supports preserving the option, not paying this exact complexity before a real use appears.

**Static coverage should be judged by actual implementation.** Statically typed closed-domain languages can require strong exhaustiveness. Python and other dynamic/open systems retain much lighter structural rules. Protos D096 deliberately occupies the latter space: sound proofs where available, otherwise `UNKNOWN`, without matcher reflection.

## Scoring convention

Scores are 1–5.

- `U` utility/current capability value: 5 = high.
- `L` lexer/lexical cost: 5 = high cost.
- `P` parser/grammar cost: 5 = high cost.
- `R` lowering/runtime cost: 5 = high cost.
- `S` static/tooling cost: 5 = high cost.
- `T` retained test/maintenance cost: 5 = high cost.
- `C` programmer cognitive cost: 5 = high cost.
- `E` teaching/remembering cost: 5 = high cost.
- `X` interaction cost with other matching features: 5 = high cost.
- `D` cost of removing now and reintroducing later: 5 = high deferral cost.
- `Sc` scalability/future resilience: 5 = strong.
- `Ph` Protos-philosophy fit: 5 = strong.

Scores are evidence aids, not arithmetic authority.

## Feature scorecard and proposed classification

| # | Mechanism | U | L | P | R | S | T | C | E | X | D | Sc | Ph | Proposed outcome |
|---:|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| 1 | postfix expression-valued `match` / `case` envelope | 5 | 1 | 2 | 2 | 1 | 2 | 2 | 2 | 2 | 5 | 5 | 4 | `KEEP` |
| 2 | source-order arms + terminal no-selection Error | 5 | 1 | 1 | 2 | 1 | 2 | 1 | 1 | 2 | 5 | 5 | 5 | `KEEP` |
| 3 | ordinary value pattern / inherited `Object.match` | 5 | 1 | 1 | 2 | 1 | 2 | 1 | 1 | 2 | 5 | 5 | 5 | `KEEP` |
| 4 | arbitrary user `pattern.match(subject)` matcher | 5 | 1 | 1 | 3 | 2 | 3 | 2 | 2 | 3 | 5 | 5 | 5 | `KEEP` |
| 5 | matcher result carrier `false | true | non-empty Array` | 4 | 1 | 1 | 3 | 2 | 3 | 3 | 3 | 3 | 4 | 5 | 4 | `KEEP` |
| 6 | `@name` binder | 5 | 1 | 1 | 1 | 1 | 2 | 1 | 1 | 2 | 5 | 5 | 5 | `KEEP` |
| 7 | `_` wildcard/discard | 5 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 5 | 5 | 5 | `KEEP` |
| 8 | guards + strict-Boolean continuation semantics | 5 | 1 | 2 | 3 | 2 | 3 | 2 | 2 | 3 | 5 | 5 | 5 | `KEEP` |
| 9 | top-level guard/body `=>` delimiter rule | 3 | 1 | 3 | 1 | 1 | 2 | 2 | 3 | 2 | 3 | 4 | 4 | `KEEP` |
| 10 | fixed Array patterns | 5 | 1 | 2 | 3 | 2 | 3 | 2 | 2 | 3 | 5 | 5 | 4 | `KEEP` |
| 11 | terminal Array remainder capture | 5 | 1 | 2 | 3 | 2 | 3 | 2 | 2 | 3 | 4 | 5 | 4 | `KEEP` |
| 12 | middle Array remainder | 4 | 1 | 2 | 3 | 2 | 3 | 2 | 2 | 3 | 2 | 5 | 4 | `KEEP` |
| 13 | bare Array remainder discard | 4 | 1 | 1 | 2 | 1 | 2 | 1 | 1 | 2 | 3 | 5 | 5 | `KEEP` |
| 14 | Array shallow snapshot + fresh frozen captured remainder | 4 | 1 | 1 | 3 | 1 | 3 | 1 | 1 | 2 | 4 | 5 | 5 | `KEEP` |
| 15 | open/subset Map pattern | 5 | 1 | 2 | 4 | 2 | 4 | 2 | 2 | 3 | 5 | 5 | 4 | `KEEP` |
| 16 | exact Map mode | 3 | 1 | 1 | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 4 | 4 | `KEEP` |
| 17 | Map remainder capture | 4 | 1 | 2 | 3 | 2 | 3 | 2 | 2 | 3 | 3 | 5 | 4 | `KEEP` |
| 18 | bare Map remainder discard | 1 | 1 | 1 | 1 | 1 | 2 | 2 | 2 | 1 | 1 | 5 | 2 | `REMOVE_PERMANENTLY` |
| 19 | Map stable snapshot + fresh frozen captured remainder | 4 | 1 | 1 | 4 | 1 | 4 | 1 | 1 | 3 | 4 | 5 | 5 | `KEEP` |
| 20 | whole-subject alias | 4 | 1 | 2 | 1 | 1 | 2 | 2 | 1 | 2 | 2 | 5 | 5 | `KEEP` |
| 21 | OR first-success ordered choice | 4 | 1 | 2 | 2 | 2 | 3 | 2 | 2 | 3 | 3 | 5 | 4 | `KEEP` |
| 22 | OR fixed binding-interface equivalence | 4 | 1 | 3 | 1 | 3 | 3 | 3 | 3 | 3 | 3 | 5 | 4 | `KEEP` |
| 23 | fixed opaque matcher `captures(a,b)` | 4 | 1 | 2 | 3 | 3 | 3 | 3 | 3 | 3 | 4 | 5 | 5 | `KEEP` |
| 24 | dynamic opaque matcher `captures(first,...rest)` | 2 | 1 | 3 | 4 | 4 | 4 | 4 | 4 | 5 | 2 | 4 | 2 | `REMOVE_NOW_RECONSIDER_LATER` |
| 25 | complete-arm dynamic-rest terminality rule | 1 | 1 | 3 | 2 | 5 | 4 | 4 | 4 | 5 | 2 | 3 | 2 | `REMOVE_NOW_RECONSIDER_LATER` |
| 26 | resource-bounded tri-state coverage (`EXHAUSTIVE/NON_EXHAUSTIVE/UNKNOWN`) | 3 | 1 | 1 | 1 | 3 | 3 | 1 | 2 | 2 | 3 | 5 | 5 | `KEEP` |
| 27 | syntax-stable structural unreachability + advisory warning/lint scope | 4 | 1 | 2 | 1 | 3 | 3 | 2 | 2 | 2 | 4 | 5 | 5 | `KEEP` |

## Rationale by disputed group

### KEEP — aliases

Alias is an ergonomic binding composition primitive rather than a parallel matching institution. Its runtime implementation is essentially “bind the whole subject in addition to nested captures”. It is common across mature pattern systems and does not force matcher reflection, new ownership or a second result carrier.

Strongest argument against KEEP: the body can sometimes reconstruct or separately retain the subject. That is not generally equivalent once the matched substructure is nested or once duplication harms readability. The saved machinery is too small to justify deleting the capability.

### KEEP — OR + fixed binding-interface rule

OR's runtime rule is simple ordered choice. The nontrivial part is ensuring that one arm body sees one stable binding interface. This is not accidental Protos complexity; Python, Rust, Dart and ML-family systems face the same invariant, while Ruby demonstrates the alternative of restricting bindings.

Removing OR would force repeated arms/bodies for a common “same consequence, multiple shapes” problem and would not remove the general need to reason about source-order effects among arms.

Strongest argument against KEEP: effectful arbitrary Protos matchers make OR ordering more observable than in many typed algebraic systems. D090 already chooses the conservative first-success/source-order rule, which bounds that risk without rollback or branch state.

### KEEP — middle Array remainder

Once one Array remainder and prefix/suffix representation exist, middle placement is incremental rather than foundational complexity. Python explicitly permits the star subpattern in any position; C# slice patterns and Dart list-rest provide comparable use cases.

Removing only middle placement would retain most of the same parser/runtime concepts while reducing compositional expressiveness. Reintroduction would be easy, but the current ongoing cost is also bounded.

### KEEP — exact Map mode

Exact mode is not the default and does not add a second Map matching engine. The stable association snapshot needed by current Map semantics already exists; exactness adds a bounded key-set condition. Ruby and Racket provide explicit exact/closed modes, showing a real strict-shape use case alongside open matching.

Strongest argument against KEEP: open/subset matching covers many configuration/message/JSON cases and exact matching may be uncommon. The implementation/public cost is sufficiently small that the audit does not find a present-complexity red flag.

### KEEP — Map remainder capture

Python, Ruby and Racket all expose keyed remainder/residue capture. In Protos it is pay-for-use: the fresh remainder Map is materialized only when a nested remainder pattern actually needs the value. It composes with the existing stable snapshot rather than creating a second keyed observation protocol.

Strongest argument against KEEP: there is no demonstrated current repository use and copying a large residual Map has cost. That cost is paid only when the feature is used; it does not justify removing the capability from Core on current evidence.

### REMOVE_PERMANENTLY — bare Map remainder discard

Normal `%{ ... }` Map patterns are already open/subset. A bare final `...` neither captures nor narrows the subject and therefore duplicates the default semantics.

This is not a future-capability question. If users want to ignore unspecified keys, omitting the remainder already expresses exactly that behavior. A future need for explicit exactness is served by `exact %{...}`; a future need for the residue is served by `...@rest` or another nested remainder pattern.

**Rejection rationale:** duplicate spelling for behavior already provided by the simpler default form.

**Superseding direction:** open/subset Map pattern with no remainder.

### KEEP — fixed `captures(a,b)`

The syntax exposes positionality, but removing it would also remove the current source-level consumption mechanism for decomposed values returned by arbitrary user matcher objects. That would turn `pattern.match(subject)` custom extensions into predicate-only extensions while built-in Array/Map forms retained privileged extraction/binding powers.

Scala extractors, F# Active Patterns, Ruby deconstruction and Racket match expanders/app patterns all demonstrate that user-extensible recognition is substantially more useful when it can also expose decomposed values.

Fixed arity is bounded: the parser/compiler knows the exact logical binding positions and ordinary arm Closure invocation consumes them directly. The major complexity spike comes from **dynamic** capture arity, not fixed capture naming.

Strongest argument against KEEP: `captures(...)` visibly exposes Protos's positional matcher-result ABI. A future syntax could hide that better. Replacing a working general extraction bridge solely for aesthetic abstraction is not justified by AUD009 evidence.

### REMOVE_NOW_RECONSIDER_LATER — dynamic `captures(...rest)`

There is no demonstrated current requirement for an opaque matcher to return an arbitrary number of arm bindings while also composing with other nested bindings. Its presence forced D103's complete-final-interface terminality rule because ordinary Closure rest parameters are terminal whereas nested matching can place later fixed captures after a dynamic segment.

That complexity is paid in source validity rules, parser validation, static interface reasoning, OR composition, tests and documentation. It is a direct example of future-preimplemented capability.

`RECONSIDERATION_TRIGGER`: a concrete Standard Library/tool/user use case needs a user-defined matcher/extractor whose successful result has genuinely variable capture cardinality and cannot be modeled cleanly as one aggregate Array capture or a fixed capture interface.

`RECONSIDERATION_SCOPE`: reconsider the **capability to consume variable-arity custom matcher extraction**, not the existing `captures(...rest)` syntax, positional partitioning model or D103 terminality rule. Redesign from the then-current matcher/binding model.

The underlying D072 ability for an arbitrary matcher result to contain a non-empty Array need not be narrowed merely to delete this source consumption form unless D131 separately determines that doing so simplifies the protocol without losing useful fixed extraction.

### REMOVE_NOW_RECONSIDER_LATER — complete-arm dynamic-rest terminality

This rule has no independent user value; it exists solely to reconcile dynamic capture-rest with the ordinary arm Closure ABI. If dynamic `captures(...rest)` is removed, the rule should disappear with it rather than remain as dormant static machinery.

`RECONSIDERATION_TRIGGER`: same trigger as dynamic capture-rest.

`RECONSIDERATION_SCOPE`: any future variable-arity extractor design must re-evaluate binding composition from first principles; D103's current terminal-final-interface rule is not reserved.

### KEEP — tri-state coverage and structural diagnostics

The actual analyzer is intentionally small and conservative. `UNKNOWN` is a normal answer and budget exhaustion fails to `UNKNOWN`; arbitrary matchers are not introspected. Stable syntactic unreachability checks prevent obviously dead source without establishing a closed pattern universe.

Removing this mechanism would save limited code while discarding useful diagnostics and a future-compatible lattice that can accept stronger facts later without changing matcher semantics.

## Resulting smallest coherent candidate

The recommended Core v0.1 candidate is therefore:

```text
KEEP
  postfix expression-valued match/case
  source-order arms + terminal no-selection Error
  pattern.match(subject) as the open matcher authority
  Object.match(subject) ordinary value behavior
  false | true | non-empty Array matcher outcome carrier
  @binder and _ wildcard
  guards and current => disambiguation
  fixed Array patterns
  one Array remainder at terminal or middle position
  bare Array remainder discard
  shallow Array snapshot + fresh frozen captured remainder
  open/subset Map patterns
  exact Map mode
  Map remainder capture
  stable Map snapshot + fresh frozen captured remainder
  aliases
  OR ordered choice + stable fixed binding interface
  fixed captures(a,b)
  bounded tri-state coverage + structural diagnostics

REMOVE_NOW_RECONSIDER_LATER
  captures(first,...rest)
  D103 complete-final-binding-interface terminality machinery

REMOVE_PERMANENTLY
  bare Map remainder discard `...`
```

## Whole-model stress check

Removing dynamic capture rest simplifies the most interaction-heavy corner without weakening ordinary user-defined fixed extractors, built-in structural matching, OR, guards or collection rest.

Removing bare Map discard removes a duplicate spelling without reducing keyed matching capability.

The retained model remains future-compatible:

- user matcher objects remain ordinary and open;
- fixed custom extraction stays available;
- future variable extraction can be redesigned when a real use appears;
- Array/Map features remain pay-for-use at runtime;
- coverage remains open-world and fail-to-UNKNOWN;
- no matcher metadata registry, closed Pattern class hierarchy, rollback system or capture-frame institution is introduced.

The principal remaining public complexity is that users must learn the distinction between ordinary matcher result captures, explicit fixed `captures(...)`, and structural `@` bindings. The audit considers that cost justified by keeping custom matcher extraction on equal footing with built-in structural forms.

## Required follow-up if approved

D131 should ratify the feature-by-feature selection. AUD009-A1 itself must not modify semantics.

For the two selected removal directions, D131's execution plan should route:

1. normative grammar/semantics removal through D131 publication;
2. executable deimplementation through a new bounded `Ixxx` matching-deimplementation work item (or a correctly owned existing implementation continuation if repository governance explicitly permits it);
3. removal/reconciliation across parser, Surface/Canonical IR, Bytecode lowering, retained AST execution path while it exists, static validation, tests, guide and status records;
4. rejection tests where useful so removed syntax does not remain accidentally accepted.

The implementation work should remove dynamic capture-rest and D103-only terminality machinery completely; no dormant parser/runtime compatibility hook should remain.

The bare Map discard removal should leave ordinary open Map patterns and captured/nested Map remainder patterns intact.

## Approval gate

This packet stops here.

The project owner must explicitly approve or alter the proposed classifications before D131 is changed, before the specification is edited, or before implementation removal is allocated/executed.


## AUD009-H final reconciliation

The earlier A1 packets are historical evidence and recommendation stages. They
are **not** the final authority for the matching surface.

D131 / `guillermomolina/protos#503` subsequently completed the full
implementation-independent decision process and the project owner explicitly
ratified **Candidate C — protocol-first bonfire**.

Durable authority:

`docs/project/decisions/language/D131_PROTOCOL_FIRST_MATCHING_MODEL.md`

D131 implementation/reconciliation is complete under I041 / #550.

For AUD009 closure, A1 is therefore reconciled to the ratified/implemented D131
outcome:

```text
PATTERN_MATCH_PROTOCOL=KEEP
OBJECT_MATCH_DEFAULT=KEEP
MATCH_RESULT_FALSE_TRUE_NONEMPTY_ARRAY=KEEP

FIXED_ARRAY_STRUCTURAL_RECOGNITION=KEEP
OPEN_SUBSET_MAP_STRUCTURAL_RECOGNITION=KEEP
STANDARD_ARRAY_MAP_ELIGIBILITY_AND_SNAPSHOT_RULES=KEEP
FIRST_SUCCESS_SELECTION_SEMANTICS=KEEP_REHOMED_IN_CASEOF
NO_SELECTION_FRESH_ERROR=KEEP_REHOMED_IN_CASEOF
ORDINARY_CALLABLE_CAPTURE_CONSUMPTION=KEEP

DEDICATED_MATCH_CASE_GRAMMAR=REMOVE_NOW_RECONSIDER_LATER
DEDICATED_WHEN_GUARD_SURFACE=REMOVE_NOW_RECONSIDER_LATER
BINDER_WILDCARD_SYNTAX=REMOVE_NOW_RECONSIDER_LATER
ARRAY_REMAINDER_INSTITUTION=REMOVE_NOW_RECONSIDER_LATER
EXACT_MAP_MATCH_MODE=REMOVE_NOW_RECONSIDER_LATER
MAP_REMAINDER_CAPTURE=REMOVE_NOW_RECONSIDER_LATER
ALIAS_PATTERN_SYNTAX=REMOVE_NOW_RECONSIDER_LATER
OR_PATTERN_CAPABILITY_AND_SYNTAX=REMOVE_NOW_RECONSIDER_LATER
FIXED_DYNAMIC_CAPTURES_SOURCE_FORMS=REMOVE_NOW_RECONSIDER_LATER
DEDICATED_STATIC_COVERAGE_FRAMEWORK=REMOVE_NOW_RECONSIDER_LATER

BARE_MAP_REMAINDER_DISCARD=REMOVE_PERMANENTLY
MATCHING_SPECIFIC_SELECTED_ARM_BINDING_ABI=REMOVE_PERMANENTLY
OR_BINDING_NAME_INTERFACE_EQUIVALENCE=REMOVE_PERMANENTLY
D103_DYNAMIC_REST_TERMINALITY=REMOVE_PERMANENTLY
```

The permanent classifications above apply to the **matching-specific duplicate
institutions**, not to the underlying capabilities. Ordinary callable binding,
Closure rest parameters, open Map matching and future ordinary matcher
composition remain available. A future ergonomic syntax decision may add new
sugar over the protocol without resurrecting the retired ABI machinery.

D131 also selected the ordinary replacement surface:

```text
Any
Capture
value.caseOf(cases)
```

These are ratified D131 additions/replacements, not retroactive AUD009
classifications of previously existing mechanisms.

Reconsideration of removed matching capabilities remains evidence-driven and must
start from the then-current protocol-first model rather than restoring the old
pattern-language implementation by inertia.

```text
D131_DECISION_APPROVAL_PROVENANCE=PASS
D131_IMPLEMENTATION=I041/#550 COMPLETE
AUD009_A1_SUPERSEDED_INTERMEDIATE_RECOMMENDATIONS=RECONCILED
AUD009_A1_CLASSIFICATION=COMPLETE
```
