# D207-A — Retain D013 ordinary `call`-slot polymorphic invocation (ratified 2026-11-10)

## Authority and outcome

**Decision owner approval:** On 2026-11-10, after receiving the recommendation to choose D207-A and close [D207 #932](https://github.com/guillermomolina/protos/issues/932), the owner replied **“si apruebo”**. This explicitly approves **Candidate A: keep D013 unchanged**; it is not a generic instruction to adopt an unreviewed alternative.

**Disposition:** NO observable semantic change, NO normative `spec/` delta, NO runtime or Standard Library modification, NO implementation issue, and no new benchmark/test. [D013 — Ordinary-slot polymorphic invocation (#171)](https://github.com/guillermomolina/protos/issues/171) remains the existing ratified authority. This record is a non-normative decision history, not a replacement for Protos specifications.

## Retained semantic invariants

1. Parenthesized `receiver(args)` evaluates the target and arguments in the prescribed order and performs ordinary mutable/delegating `receiver.call` lookup **after arguments have been evaluated**.
2. A Closure is not granted an exceptional intrinsic invocation bypass merely because its value has the Closure kind. Its ordinary inherited, shadowed or overridden `call` slot participates in normal selection, errors and effective receiver/method home.
3. Ordinary objects and factory prototypes keep their existing `call`-based construction/invocation semantics; callability inspection and callback invocation do not gain a separate Closure-identity escape hatch.
4. Existing D013/D179 and ReturnHome, Actor/Task/Process, suspension, errors and Context contracts continue unchanged. Optimizations are allowed only when they preserve these observable outcomes and invalidate correctly when the selected `call` changes.

## Focused evidence and falsification

- [D207 experimental branch commit `61006b57c9434e4a4a904ee971ae77c846c68497`](https://github.com/guillermomolina/protos/commit/61006b57c9434e4a4a904ee971ae77c846c68497) temporarily bypassed Closure `call` overrides for parenthesized calls, retaining explicit `f.call(...)` lookup; **never merged**, and not authorized as a new language contract.
- Its owner-reported isolated TOP_LEVEL Tier-1 `After TruffleTier` comparison was **645 nodes with D013 vs 645 nodes with intrinsic direct calls**, with **55 vs 55 inclusive `InvokeClosureOneTail` ancestry nodes**. There was **no demonstrated graph reduction** in that exact workload. These are graph nodes, **not timing or throughput**; one negative test does not prove that D013 has zero cost for every possible workload.
- [I115-C/D `e2200088d42d0d8be90626a79b373ddad64978f5`](https://github.com/guillermomolina/protos/commit/e2200088d42d0d8be90626a79b373ddad64978f5) kept D013 and was validated with maintainer-reported `make test` PASS. [Existing exact-revision evidence](../../evidence/I115/I115_CD_385_FINAL_PUBLICATION_AND_PLAT066_C_RESEARCH_HANDOFF_2026_10_19.md).
- Independent [PERF042 #871](https://github.com/guillermomolina/protos/issues/871) and [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930) identify expensive lowerer/admission/fallback/continuation paths in historical captures that do not establish a need to remove polymorphic `call`. Work in [I115 #931](https://github.com/guillermomolina/protos/issues/931) remains independent, and experimental semantic divergence within its branch is **not** normative D013 replacement.

## GITHUB021 no-change reconciliation

| Area | Existing invariant retained | Decision delta |
| --- | --- | --- |
| `CALLABLES.md`, D013 polymorphic `call` | Late ordinary slot lookup, delegation/overrides, Closure and ordinary object invocation | **None** |
| `OBJECT_MODEL.md`, D179, `EXECUTION_AND_CONTROL.md` | Shape/slot mutation, selected receiver and method home, Context, nonlocal return and errors | **None** |
| `PROTOS_GRAMMAR.md`, Core and callback APIs | Parenthesized call syntax and ordinary callability semantics | **None** |
| `ACTORS.md`, `FUTURES_AND_TASKS.md`, parallel and host boundaries | Authority, ownership, cancellation, suspension and task/actor semantics | **None** |
| Bytecode DSL/lowering/runtime and compiler graphs | Can optimize with guards while respecting the observable contract | **No prescribed edit** |
| Regression suites, published products and version | Existing tests and product publication remain historical authorities | **No new run/change** |

**`DECISION_INVARIANT_CONSISTENCY=PASS` for the no-change choice:** none of the cited existing guarantees must be surrendered. A more ambitious alternate-semantic contract would require a separate explicit future decision, not reinterpretation of this approval.

## Comparative-research limits and residual risk

The original D207 issue specified a broad GITHUB010 research packet (multiple languages, 12 scored dimensions, exhaustive counterexample and perf investigation). **That complete packet was not produced for this focused closure.** This ratification is grounded in the owner's explicit preference for preserving semantics, the negative isolated experiment and existing normative contract, not fabricated exhaustive research.

**Strongest counterargument:** dynamic late `call` lookup and invalidation can impose a real cost on particular hot paths, and this experiment cannot exclude benefit for other workloads. The current evidence does not justify replacing a valuable general rule with an exceptional Closure invocation kind. Continue investigating implementation-level costs under the existing PERF/PLAT/I115 owners where warranted; no blanket performance equivalence or future speedup is asserted.

**Decision closure:** [D207 #932](https://github.com/guillermomolina/protos/issues/932) to be closed as **completed** (Candidate A, no change). [D013 #171](https://github.com/guillermomolina/protos/issues/171) remains closed and ratified.
