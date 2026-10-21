# D210 — Single standard Closure loop selector and contextual while syntax

**Status:** RATIFIED, owner-approved 2026-10-21.
**Issue:** https://github.com/guillermomolina/protos/issues/941
**Evidence:** [D210 research and invariant audit](../../evidence/D210/D210_WHILE_SYNTAX_FALSIFICATION_AND_RATIFICATION_2026_10_21.md).
**Observed product source:** `a48ef2e4ca1619e13e1156405f6cdf77ea15604f`. This is a pre-implementation baseline, not an implementation commit.
**Approval:** Owner explicitly accepted Q1 candidate C and Q2 real contextual sugar despite incompatible older lexical `while(...){...}` calls. See D210 comments 6311255405, 6311488302 and 6311582929. Implementation remains separate.

## Q1: standard method surface

Keep exactly one standard pre-test Closure selector: `Object.whileTrue`. Remove standard `Object.while`, including its terminal-`false` normal-result behavior and standard slot reflection. No alias. User-defined ordinary `while` identifiers and member slots remain legal outside the precise Q2 grammar.

Standard `conditionClosure.whileTrue(bodyClosure)` still performs strict pre-test looping using semantic Closures and exact canonical Boolean results. Zero body activations returns canonical `null`; otherwise the exact last normally completed body result is returned, preserving identity, even after the terminal predicate. Conditions are re-evaluated before each body activation; domain validation happens before the first condition. Errors, nonlocal returns, suspension, cancellation, cleanup and Future ownership behave exactly as before; no hidden Task, scheduler, break, continue, truthiness, implicit await or new loop runtime. Preserve ordinary selected `whileTrue` lookup, shadowing, extraction and method-home behavior.

## Q2: contextual syntactic sugar

```protos
while(n > 1) { n = n - 1 }
```

has the required observable semantics of:

```protos
(() => n > 1).whileTrue() { n = n - 1 }
```

This is semantic equivalence, not a mandatory string rewrite. Fresh zero-argument lexical condition Closure: evaluate its inner condition expression again for each test. Fresh parameterless lexical body Closure. This contextual form is a result-valued Protos expression returning the exact `whileTrue` result. Preserve lexical `this`, `context`, `super`, nonlocal return home, default binding, validation, Error, Future, suspension/replay and cancellation behavior.

**Recognition:** only an ungrouped bare identifier with exact spelling `while`, followed by exactly one normal non-spread parenthesized condition expression, then an immediately attached parameterless braced trailing Closure under existing logical-NEWLINE rules. Do not reserve `while` globally. This exact source form **takes precedence** over ordinary invocation even if a user defines lexical `while`; this source incompatibility was expressly approved. `(while)(x) { body }` and `object.while(x) { body }` remain ordinary calls; `while(x)` without trailing body remains ordinary; `while { body }` retains object-construction meaning. Newline-separated braces do not attach. Do not extend the exception to `if`, `else`, generic calls, member sends or D199 undotted Boolean syntax.

## GITHUB021 invariant audit

| Previously selected invariant | D210 disposition |
|---|---|
| D044 pre-test Closure behavior, strict Boolean, zero-argument activations and failure propagation | PRESERVED |
| D180 single standard `whileTrue`, no `whileFalse` | RESTORED |
| D180 no dedicated loop syntax | EXPLICITLY REOPENED by Q2 owner approval |
| D201 extra `Object.while` with terminal-false result | EXPLICITLY REOPENED/REMOVED by Q1 owner approval |
| D201 result-producing `whileTrue` | PRESERVED |
| Ordinary bare `while(x){body}` call | EXPLICITLY REINTERPRETED by narrow contextual grammar, accepted owner break |
| D051 ordinary `if`, D199 rejection of undotted Boolean syntax | PRESERVED |
| Existing `whileTrue` ordinary method lookup/override, `this`/`super`/`context`, Future/Error/NLR/suspension/cancel | PRESERVED |

Older D044/D180/D199/D201 records remain intact as historical decisions. The GITHUB010 prior-art survey, 12-dimension scored comparison, strongest adverse counterexamples, pay-as-you-grow analysis and provenance are in the linked evidence.

## Implementation sequence

Create one separate `Ixxx` implementation issue in `guillermomolina/protos`, with two publishable slices **in this order**:

1. **A, first:** remove standard `Object.while` from Object installation/native behavior, prepared/structured/inline paths and slot lists; update execution/callable specs, conformance, source docs and tests. Keep `whileTrue` and user-defined ordinary member `while`.
2. **B, second:** implement the targeted contextual `while(condition) { body }` grammar, parser/source facts, canonical Closure/ordinary-`whileTrue` lowering, normative grammar, conformance and tool tests.

Each slice must be validated separately and published by the human executor. D210 research and ratification execute no product compilation/tests, and the owner's pre-existing local PASS report must not be attributed to a new implementation. The observable compatibility breaks include `Object.hasSlot("while")`, extraction, prior result value and possible outer-Future adoption or last-body retention after migration.
