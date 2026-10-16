# D204 — Suspension capability and proof boundaries: no new language semantics

**Decision:** D204, implementation-independent language question.
**Outcome:** **Candidate A — NO_NEW_SEMANTIC_DECISION_REQUIRED** (owner approved).
**Exact owner approval, 2026-10-16:** “entonces apruebo A, para este ISSUE, pero quiero implementar esta falencia, y que el lowerer lo utilice, (no solo el PLAT), que sea un caso general.”
**Authoritative issue:** [D204/#905](https://github.com/guillermomolina/protos/issues/905).
**Research:** [D204 exhaustive evidence](../../evidence/D204/D204_EXHAUSTIVE_RESEARCH_2026_10_16.md).
**Research source baseline:** [guillermomolina/protos@d1697153256dc3952008cb40e507f09aea0f7101](https://github.com/guillermomolina/protos/commit/d1697153256dc3952008cb40e507f09aea0f7101), 0.3.369-SNAPSHOT, checked 2026-10-16.
**Implementation consumer:** [I107/#906](https://github.com/guillermomolina/protos/issues/906).
**Ratified architecture:** [PLAT061-B/#903](https://github.com/guillermomolina/protos/issues/903), [I106/#904](https://github.com/guillermomolina/protos/issues/904).
**Scope:** semantic interpretation and proof boundary; this document is non-normative decision/evidence, not a new Protos specification clause.

## Selected semantic answer

No discovered normative gap requires a new language concept of a “suspendible function,” a `suspend` keyword or attribute, an effect-typed Closure, or compulsory Task creation for an ordinary call. The already-normative semantics determine **when actual execution suspends**: observing a **pending** Future using `Future.value()` suspends; observing a **resolved** Future returns immediately; creating or returning a Future alone does not implicitly suspend. An outer host-initiated RootActor entry need not be Task-backed to suspend. Dynamic method/call selection, defaults, callbacks, loops, `ensure`, Error propagation and foreign boundaries retain their existing normative behavior.

The compiler can establish facts about *potential suspension* without changing these guest semantics. In particular:

1. **Canonical Closure body proof.** An exact body is non-suspending for all admissible executions under its exact lexical binding analysis and scope. This does not include arbitrary callers, dynamically overridden invocation, or argument/default evaluation.
2. **Emitted-root proof.** A particular emitted root may be provably non-suspending after the lowerer's authorized transformations. That fact describes the effective emitted root, not retrospectively the original canonical body.
3. **Complete invocation proof.** Only a proof composing receiver and actual ordinary `Object.call` selection, argument and spread evaluation, applicable default binding, body, selected callbacks and cleanup can justify omission of full invocation continuation support.
4. **Guarded site-specific proof.** A call can be non-suspending under proven target/binding/presence/Context/profile assumptions. If a guard may fail, a correct generic fallback remains mandatory for the miss.

**UNKNOWN is not FALSE.** A missing no-suspension proof is neither evidence that suspension will occur nor grounds to alter guest behavior. A runtime observation of immediate completion is not a universal semantic proof. Dynamic override, mutable D179 binding identity/presence, recursion, selected callback, selected default and `ensure` effects need conservative handling.

Error signaling, nonlocal ReturnHome, cooperative cancellation, blocking host worker and physical Truffle yields are distinct from the language event of guest suspension. No return-home, Error or control-flow simplification is authorized by misclassifying those events.

## Approved separate implementation requirement

The owner explicitly requested that this be **general functionality of the composable compilation-fact contract**, not just a platform document or a narrow `identity(1)` special case.

[I107/#906](https://github.com/guillermomolina/protos/issues/906) must implement, within the **already-ratified PLAT061-B architecture**:

- A bounded, reusable **pre-emission producer/consumer proof model** usable by any eligible phase that can establish a sound fact, keyed by canonical definition, exact lexical analysis, scope and other validity assumptions. The facts must be consumed by the actual lowerer, not merely recorded for consumers after `builder.endRoot()`.
- Compositional non-suspension reasoning over real canonical expressions, sequences, nested branching/control and known targets, scaling to many safe conditionals without a hard-coded branch count or mandatory whole-program effect analysis. Unresolved dynamic operations conservatively remain UNKNOWN. Creating/returning a Future must not be confused with observing a pending one.
- The lowerer must **produce/refine new facts** when it learns additional information from transformations or effective emission, and publish those facts at their correct scope for downstream consumers. An emitted-root fact may be stronger than a canonical-body fact; neither can silently overwrite a consumed proof.
- The lowerer must use sufficient complete-path/site evidence **before emission** to actually omit unreachable/unnecessary continuation-composition or generic-fallback Bytecode for provably safe paths. Merely setting a boolean, moving a helper method, or relying on JIT dead-code elimination does not satisfy this outcome.
- Dynamic guard misses, arbitrary overridable `Object.call` selection, chosen argument/default/callback/cleanup operations and possible pending-Future observation must retain all required C-prime continuation behavior. Never remove fallback because an expected body was proven safe but the actual selected invocation was not.
- General positive/negative/UNKNOWN/conditional tests, Bytecode-group and independent handwritten Builder oracles, idempotent reparse and destination-Context/NORMAL/DEBUG checks must establish correctness *and physical emission change*. The physical result must be demonstrated in at least one general, fully sound case; no unmeasured PERF042 speedup is assumed.

**Pay as you grow:** no new guest-level capability, mandatory RootTask/RootActor/RootProcess or waiter allocation on trivial calls, per-call runtime effect probing, global mutable fact registry, eager whole-program analysis, second interpreter, rewritten optimized IR, new public Closure variant or ABI. The approach stays demand-driven, compiler-local and soundly guarded. A genuine newly discovered semantics or durable architecture issue requires a fresh owner decision, not an implicit I107 choice.

## GITHUB021 invariant preservation

- **D013:** actual selected ordinary `Object.call` and its dynamic overrides retain authority; a candidate source Closure is not automatically the selected target.
- **D179:** live lexical capture by reference, exact binding identity, removal/re-creation and Context observability remain sound.
- **D202/D203:** flexible positional/required/default/rest/spread semantics and evaluation order remain unchanged; defaults may suspend through explicit operations.
- **Future/Task/Actor, host, foreign and cross-Context:** preserve pending/resolved distinction, cooperative cancellation, non-Task-backed suspension, foreign-callback rejection, Context-local projection and Actor isolation.
- **Errors, `ensure`, ReturnHome:** nonlocal returns and Error signaling are not suspension; cleanup and handlers may themselves suspend, and already committed effects cannot be replayed on continuation resume.
- **PLAT014:** retain C-prime continuation/yield composition for possible suspension.
- **PLAT058/059/060:** unique semantic Bytecode DSL lowerer, NORMAL/DEBUG/tooling/source-tag fidelity, no general canonical optimizer, universal B5 call ABI.
- **PLAT061-B/I106:** existing multi-producer/multi-consumer contract, bounded pre-Builder planning, source/site/emitted-root proof scope separation, demand-driven fact creation, reparse and destination-Context soundness. I106's first four facts are not an architectural ceiling.
- **PERF042:** compiled-graph and performance acceptance remain separate; static 434-vs-38 lowerer/handwritten Builder count does not establish comparable graphs or speedup.

**GITHUB021 result:** No previously approved invariant is contradicted, narrowed or silently reopened.

## Comparison and alternatives

- **A — APPROVED.** Preserve existing normative guest semantics, improve conservative internal proof availability and safe use under PLAT061-B. Lowest guest burden and no mandatory runtime cost.
- **B — not selected.** Add minimal normative prose only if a concrete semantic contradiction emerges. No current contradiction justifies a new language contract.
- **C — rejected for D204.** Public `suspend` annotations, effect rows, separate async Closure kinds or special call typing would change the language, tooling and compatibility model without necessity.
- **D — not independently selected.** A new public/host-wide guarded proof contract duplicates PLAT061-B if it is purely internal; any actually new public guarantee would require its own design decision.
- **E — rejected as full response.** Deferring all compiler proof integration leaves a concrete producer/consumer/emission gap unaddressed; deferral remains appropriate for unproven optional optimizations.

The full GITHUB010 12-axis 1–5 comparison, credible language/VM approaches, counterargument to A and adversarial matrix are in the linked exhaustive research. The strongest counterargument is that too many dynamic sends may remain UNKNOWN for any useful physical emission reduction. I107 mitigates this by requiring a meaningful sound general-use topology improvement without assuming a global effect analysis or benchmark-specific proof.

## Revision, tests and publication boundary

This decision is about **existing semantics**, not an assertion that I107 has already shipped. The maintainer stated on 2026-10-16 that local tests pass and `git diff --check` is clean; those claims lack an exact run identity and do not prove I107 publication. Research baseline `d1697153256dc3952008cb40e507f09aea0f7101` is a pinned inspected product revision, not a new D204 product commit.

This record is published by the agent to `guillermomolina/protos-project-docs` under explicit task-bounded owner permission in the active conversation. The GitHub publication SHA is the exact `PROJECT_RECORD_REVISION` after both evidence and this decision are committed and read back. Neither a new `spec/` amendment, code change, test execution nor benchmark execution is part of this decision publication.

**D204 closure gate:** confirm exact owner approval, durable record at its immutable publication SHA, cross-links and current issue state; record GITHUB015/GITHUB020 evidence in #905 before closing as completed. **I107:** after D204 has met its ratification gate, release from administrative `status:blocked` to `status:ready` (not `in-progress` absent active assignee), without claiming implementation or validation. Native dependency edge is a separate GitHub coordination concern and must not be fabricated.
