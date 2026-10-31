# INC003 — Range iteration syntactic sugar: DEFER evidence (2026-10-31)

> **Status: DEFER accepted by project owner, 2026-10-31.** This is a bounded, non-normative Incubator investigation snapshot, not a selected language design, a specification amendment, an implementation authorization, or a ratification record. Live state belongs to [INC003 #1007](https://github.com/guillermomolina/protos/issues/1007).

## Source identity and provenance

- **Product inspection revision:** [`guillermomolina/protos@73bd9fe913421ff493146b546869f9a1f0d2b34e`](https://github.com/guillermomolina/protos/commit/73bd9fe913421ff493146b546869f9a1f0d2b34e) (`main` as read for the investigation, 2026-10-31).
- **Related approved library decision:** [LIB017 #570](https://github.com/guillermomolina/protos/issues/570), [LIB017_RANGE_API_DECISION.md](../../work/LIB017/LIB017_RANGE_API_DECISION.md), Candidate G-prime: direct, bounded, half-open unit Integer iteration; no first-class Range value or syntax.
- **Related, independent optimization:** [PERF045 #889](https://github.com/guillermomolina/protos/issues/889) evaluates compiled `each` behavior. INC003 neither duplicates that work nor asserts performance results.
- **Owner instruction:** "Acepto defer" in the active 2026-10-31 INC003 interaction. Approval is limited to **deferring** a new syntax; no candidate syntax, semantics, code, schedule, or priority is approved.
- **Evidence method:** targeted, read-only inspection of repository specs, implementation, examples, conformance Cases, and relevant issues; comparative reasoning. **No** full source census, grammar prototype, runtime execution, benchmarks, parser tests, or independent reviewer validation were conducted for INC003.

## Findings grounded in the Protos revision

### Existing capability, not a missing feature

The [`std:collections/Range` source](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/lib/collections/Range.protos) exports **`each(start, stop, block)`** and **`reverseEach(start, stop, block)`**, not `forEach`. They visit ordinary unbounded Integer values from the half-open interval `[start, stop)` in forward or reverse unit order. Normal result is `null`; a reached callback is an ordinary polymorphic call and its result is ignored. Neither operation materializes a Range object.

```protos
Range: import("std:collections/Range")
Range.each(0, count, i => {
    print(i)
})
```

The selected LIB017 contract also establishes: each endpoint expression is evaluated exactly once; equal or reversed bounds visit zero values; a non-invokable callback is **not** inspected for an empty interval; Error and non-local return propagate; there is no implicit Future await/adoption, generic iterator protocol, step parameter, or syntax.

The actual [`range-core.protos`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/tests/conformance/library/collections/range-core.protos) and [`range-callbacks-and-control.protos`](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/tests/conformance/library/collections/range-callbacks-and-control.protos) cover endpoint evaluation, ordering, unbounded Integer bounds, polymorphic callbacks, callback argument contract, Error identity/propagation, empty-interval non-invokable callbacks, and `^` non-local return. These are **existing test sources**, not tests re-executed for this investigation.

### Demonstrated usage and limits

Existing `Range.each` uses include [FizzBuzz](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/examples/algorithms/fizzbuzz.protos), [100 doors](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/examples/algorithms/100-doors.protos), [value capture](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/examples/closures/value-capture.protos), and [first-class functions](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/examples/closures/first-class-functions.protos). This API is already reasonably concise for those use cases.

Some manually counted `while` loops are plausible candidates for replacing with the **existing** helper before proposing additional syntax. Examples: the fixed `count` copy in [Array.slice](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/lib/collections/Array.protos) and the explicit `start`/`end` copy in [HTTP MessageSyntax](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/protos/lib/http/internal/MessageSyntax.protos). Actual eligibility still depends on stable endpoints, callback/context consequences and layering, and is **not established merely by similar spelling**.

Counterexamples appear in the same HTTP source: `while(equal && index < expected.size())` stops on dynamically detected mismatch and cannot be rewritten as unconditional `Range.each` without preserving that early-exit behavior. The same caution applies to parser/scanner, mutable-size, early-failure, variable-stride and mutable-loop-counter cases. LIB017 explicitly forbids mechanical replacement where a `while` re-evaluates a semantically changing bound.

### Grammar, callable and tooling implications

- The [normative grammar](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/PROTOS_GRAMMAR.md) already has exact contextual `if`, `while`, and `until` surface expressions lowering to ordinary message sends. `for` is an ordinary, unreserved identifier; a new context-sensitive shape must resolve collision with ordinary calls. The existing trailing-Closure form is **parameterless**; parameterized trailing closures were expressly removed to avoid parse ambiguity.
- The same grammar defines `..` as having no current Core meaning; `..<` is not an approved range operator. Introducing `0..<n` is a separate lexical/grammar change, not punctuation obtainable free of a design decision.
- [CALLABLES](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/semantics/CALLABLES.md) requires lexical closures capturing by reference, ordinary invocation/parameter binding, receiver/context and ReturnHome semantics; `^value` is a non-local return, **not** a `break`. [EXECUTION_AND_CONTROL](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/semantics/EXECUTION_AND_CONTROL.md) owns loop control and distinguishes `whileTrue`'s most-recent-body-result contract from the library Range helper's `null` result.
- [MODULES](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/spec/semantics/MODULES.md) makes imports and bindings ordinary. A contextual form must either preserve explicit `Range`/aliased module lookup and ordinary overrides, or openly justify a new privileged library identity; the latter is a significant design cost.
- Existing lowering is visible in [ProtosParser.java](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/src/main/java/com/guillermomolina/protos/parser/ProtosParser.java) and [Canonicalizer.java](https://github.com/guillermomolina/protos/blob/73bd9fe913421ff493146b546869f9a1f0d2b34e/src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java). A new syntax would also require source spans/comments, static definitions/references, formatter, LSP and associated tests to remain coherent. No product patch was made.

## Options assessed — all unapproved

| Option | Illustrative spelling | Assessment |
| --- | --- | --- |
| A: Keep current ordinary API | `Range.each(0, n, i => { ... })` | Minimal cost; good existing expressivity; **default while deferred**. |
| B: Contextual numeric loop | `for(i, 0, n) { ... }` | Possible canonical lowering, but clashes with an otherwise ordinary `for(...)` call shape and must specify the loop binding/import lookup and callback activation. |
| C: Parameterized trailing closure | `Range.each(0, n) (i) { ... }` | Potentially general benefit but reopens an explicit prior grammar/ambiguity decision beyond INC003. |
| D: Generic `for-in` plus range operator | `for i in 0..<n { ... }` | Visually compact but implies range-expression grammar and potentially range values/iteration protocols, contrary to LIB017's minimum without new evidence. |
| E: Integer-oriented range API | `0.until(n).each(i => { ... })` | Uses ordinary-looking messages but is a new public API and may imply a deferred Range value and distinct `until` semantics. |

These spellings **except A are illustrative only** and are not asserted to parse today.

Comparative precedents reinforce the layering distinction: Pharo/Smalltalk's message/block bounded iteration motivated LIB017; [Kotlin ranges and progressions](https://kotlinlang.org/docs/ranges.html) combine a more compact surface with distinct progression abstractions; [Rust `for` and `IntoIterator`](https://doc.rust-lang.org/stable/reference/expressions/loop-expr.html) depend on an iterator contract; [Go 1.22](https://go.dev/doc/go1.22) shows integer-range iteration and the observable importance of per-iteration binding/capture semantics. Popular syntax alone does not justify importing these additional semantics into Protos.

## Owner-accepted incubation outcome

```text
INC003_RECOMMENDATION=DEFER
OWNER_ACCEPTED=YES
INCUBATION_STATE=PAUSED_OPEN
SYNTAX_SELECTED=NO
FORMAL_DXXX_PROMOTION=NO
IMPLEMENTATION_AUTHORIZED=NO
PUBLIC_API_CHANGE=NO
NORMATIVE_SPEC_CHANGE=NO
PERFORMANCE_CLAIM=NONE
PRIORITY=INTENTIONALLY_UNSET
ASSIGNEE=INTENTIONALLY_UNSET
```

**Rationale:** some manually maintained counter loops are verbose, but the current Range helper already eliminates that boilerplate with no new syntax. Targeted repository examples have not demonstrated *residual* ergonomic friction after writing equivalent code with `Range.each`. A privileged `for` rule, new range operator, range object, or generic iterator model would increase language complexity without that evidence.

**Revisit only if:** representative real programs demonstrate recurring, important friction *after* idiomatic `Range.each` use; the candidate defines exact structural parsing and interaction with unreserved names, ordinary import/lookup, per-invocation binding and closure capture, non-local returns, Error and Future behavior, normal result, evaluation order and stable bounds; and its value outweighs parser/tooling/compatibility cost. If a substantive candidate is then chosen for decision, allocate a separate `Dxxx` under `AGENTS.work/DESIGN.md` and request **exact owner approval** before normative or implementation changes.

INC003 remains an independent top-level idea, not a sub-issue of LIB017 or PERF045. `status:paused` records **defer without rejection or closure**. This evidence is published because the owner explicitly requested durable evidence for the completed exploration; it does not create a standing requirement for a document per Incubator idea.
