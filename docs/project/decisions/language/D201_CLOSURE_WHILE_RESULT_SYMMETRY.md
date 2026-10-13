# D201 — Closure loop result symmetry: `whileTrue` / `while`

**Status:** RATIFIED — owner explicitly approved Candidate D on 2026-10-13 following a second adversarial falsification.
**Live authority:** [D201/#882](https://github.com/guillermomolina/protos/issues/882).
**Evidence:** [Comparative survey, GITHUB010 matrix, and falsification](../../evidence/D201/D201_LOOP_RESULT_SYMMETRY_EVIDENCE_2026_10_13.md).
**Related independent decision:** [D200/#880](https://github.com/guillermomolina/protos/issues/880), additive Boolean `if`/`else` ratified; its implementation I097 is independent.
**Observed product HEAD at publication research:** [8f7e5ea5b7ad75f8052adf7ce51945539348b4a1](https://github.com/guillermomolina/protos/commit/8f7e5ea5b7ad75f8052adf7ce51945539348b4a1). This is provenance, NOT a required implementation checkout.
**Authority boundary:** this is non-normative ratification history. Normative changes belong exclusively to `guillermomolina/protos/spec/`; product implementation and tests have not occurred under D201.

## Exact approved Candidate D

Add two distinct ordinary standard Closure-family messages, each requiring exactly one semantic Closure body argument, using the same pre-test condition/body activations and other established loop rules:

- `conditionClosure.whileTrue(bodyClosure)` executes condition then body while each exact condition result is canonical `true`. On normal termination, returns the **exact normally returned result of the last completed body activation**, preserving its identity; returns canonical `null` when there were zero body activations.
- `conditionClosure.while(bodyClosure)` performs the **same pre-test loop** with the same callbacks and normal termination condition but returns the **last normally returned condition value**, necessarily canonical `false` on ordinary termination, including zero iterations. Does not return the body result.
- `while` is not an alias of `whileTrue`: their normal result policies differ. It is an ordinary standard selector, not a reserved word, grammar production, undotted keyword, source rewrite, or a special expression category.
- No standard `whileFalse` is approved. No `break`, `continue`, `elseIf`, implicit truthiness, Future await/adoption by loop methods, new Task, scheduler, ownership scope, hidden checkpoint, or loop-generated cleanup behavior.
- Both methods require original receiver to be an actual semantic Closure and `body` to be an actual semantic Closure; validate arguments/receiver before invoking first condition, as in the existing `whileTrue` contract. A skipped body is still domain-validated up front, but its parameter arity/default values are checked/executed only upon an actual activation.
- Condition must return exactly canonical `true` or `false`; other results, including a Future, signal an Error. Evaluate receiver and body argument expressions in ordinary left-to-right order before standard loop invocation.
- Each activation uses the Closure's own existing zero-argument activation, not an extra `call` lookup. Exact callback count, ordering, receiver/methodHome, captures, default binding, Error identity, NLR/InvalidReturn, suspension/resumption without replay, cancellation and ensure/unwind semantics remain unchanged.
- Error, NLR, cancellation, or other non-normal transfer from the body or final condition takes precedence over any saved result. No normal return occurs on those paths. The final predicate executes after the final successful body activation; it may mutate the object to be returned, whose identity remains unchanged.
- A body result that happens to be a Future is returned **as the exact value** from `whileTrue`; the loop itself never waits for, observes, adopts, flattens or cancels it. Existing *outer* Future-resolution adoption may now apply if this returned Future becomes the result of async execution. `while` discards the body value.
- Last-result retention is an observable lifetime obligation where needed; physical retention may be elided only when semantically unobservable. Implementations must preserve identity/results across suspension, optimized compilation, replay, deoptimization, and all admitted/non-admitted dispatch paths. The candidate does NOT promise zero memory cost.

Normal-result table:

| Completed body activations | `whileTrue` | `while` |
| --- | --- | --- |
| 0 | canonical `null` | canonical `false` |
| 1 | exact result of body activation 1 | canonical `false` |
| N > 1 | exact result of final body activation N | canonical `false` |

## Relation to D200: limited, intentional symmetry

D200's **approved** `ifTrue`/`ifFalse` yield selected action result while the added `if`/`else` retain the canonical Boolean receiver after selected action effects. D201 applies a parallel **result-of-work vs result-of-control naming discipline**: `whileTrue` yields last body result, `while` yields final predicate `false`.

This is **not** identity-of-receiver symmetry. Boolean `if` returns the original Boolean; Closure `while` returns its last condition activation value, not the Closure receiver. Neither is it uniform callback validation: Boolean skipped actions are not callability-validated, while both loop methods require a Closure body even for zero iterations. Chaining `condition.while(body).else(action)` executes `action` following **any normal loop termination**, never as a zero-iteration-specific branch or as guaranteed cleanup after abnormal exit.

## GITHUB021 explicit delta / supersession

| Existing binding invariant | D201 Candidate D | Owner authority |
| --- | --- | --- |
| D044: `whileTrue` normal result is always `null`, every body value discarded | **REOPENED and REPLACED** only for the normal result: exact last body value or `null` when zero | Owner explicitly approved Candidate D after explicit memory, Futures and compatibility counterexamples |
| D180: standard selector `whileTrue` only; no `while` | **REOPENED and REPLACED** to add distinct ordinary standard `while`; `whileTrue` retained | Same explicit Candidate D approval |
| D180: no `whileFalse` | PRESERVED | D does not add `whileFalse` |
| D044/D180: strict Closure-domain receivers/bodies, eager domain validation, canonical Boolean condition, callback activation, failures, NLR, suspension and cancellation | PRESERVED | D changes return policy, not invocation/control rules |
| D050/D200: Boolean `ifTrue` and additive `if`/`else` contracts | PRESERVED | D201 does not modify Boolean methods |
| D051/D199: ordinary message syntax, rejected undotted sugar | PRESERVED | no grammar addition |
| Frozen Object standard slots / lookup, reflection, shadowing | **ADDITIVE observable public slot** `while` | Exact D candidate approval, with migration/test obligation |
| No implicit Future adoption in the loop | PRESERVED | Outer asynchronous Future resolution can now adopt a propagated result, by existing rules |
| Existing performance, identity and execution-domain rules | PRESERVED semantically | Implementation must prove equivalence; no benchmark outcome assumed |

The owner was explicitly told that D breaks prior result/selector contracts, changes potential outer Future outcomes, may retain large last-body objects, and requires changes to the direct, structured and literal-inline code paths, **then** wrote “ok apruebo D”. Approval is scoped to this full Candidate D and its explicit consequences.

## Future, memory and compatibility failure examples

- A `whileTrue` body whose last result is a Future used to be discarded; after D it flows out as that Future. When the enclosing asynchronous Closure completes with this Future, existing Future adoption changes its eventual visible outcome. This is an approved compatibility break, **not** implicit awaiting in `whileTrue`.
- The exact last body object must survive the final predicate, including a possibly suspended final predicate. With an unused result a compiler may remove retention; this is an optimization hypothesis and not an unconditional semantic promise.
- Older `whileTrue` result assertions expecting `null` after nonzero iterations must migrate. A zero-iteration loop still yields `null`. Existing `!Object.hasSlot("while")` conformance must migrate to `Object.hasSlot("while")`, while custom own-slot shadowing stays ordinary.
- The last predicate is evaluated once after the last body; an error or NLR here prevents any normal body result from leaking out.

## Ratification-to-implementation boundary

This record does not change current runtime or normative specification. The next **I-family implementation** should implement **both** methods and semantic/spec/conformance reconciliation as one coherent slice in `guillermomolina/protos`, on the agent's actual local HEAD. Inspect and reconcile direct native protocol, structured Bytecode DSL dispatch, and PERF026/PLAT044 local inline literal `whileTrue` admission/fallback. Coverage must include identity, zero/one/N, invalid values, callbacks, reflection, Future provenance/adoption, retained reference across suspended final predicate, error/NLR/cancellation, and path parity. Runtime resource cost requires evidence, not mere assertions.

The agent edits; **the human runs builds, tests, git status/diff checks, version/changelog promotion, commit and push**, observing `AGENTS.md`. Do not mutate `pom.xml`/`CHANGELOG.md` until all product tests are reported green and publication is ready. No tests after those metadata edits. No product tests or builds were executed during this ratification.
