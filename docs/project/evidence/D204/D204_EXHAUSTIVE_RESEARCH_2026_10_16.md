# D204 — Exhaustive suspension capability investigation, proof boundaries and compiler consequences

**Status:** Evidence of owner-approved D204 Candidate A, not a product implementation or normative spec amendment.
**Date:** 2026-10-16
**Source baseline:** [guillermomolina/protos@d1697153256dc3952008cb40e507f09aea0f7101](https://github.com/guillermomolina/protos/commit/d1697153256dc3952008cb40e507f09aea0f7101), version 0.3.369-SNAPSHOT.
**Authority:** [D204/#905](https://github.com/guillermomolina/protos/issues/905); [PLAT061-B/#903](https://github.com/guillermomolina/protos/issues/903); [I106/#904](https://github.com/guillermomolina/protos/issues/904).
**Selection:** Owner explicitly approved A on 2026-10-16, with a separate demand for general fact-contract implementation by [I107/#906](https://github.com/guillermomolina/protos/issues/906).
**Decision:** [D204 decision record](../../decisions/language/D204_SUSPENSION_CAPABILITY_NO_NEW_SEMANTICS.md).

## 1. Normative suspension taxonomy

| Operation/event | Guest suspension? | Sound-proof consequence |
|---|---|---|
| Create or return a Future | Not by itself | No implicit awaiting or suspension. |
| Observe a resolved Future via `value()` | No, on that execution | A completed past run does not give a universal static proof. |
| Observe a pending Future via `value()` | Yes, explicitly | Continuation path needed without a stronger readiness/unreachability guarantee. |
| Read an exact local parameter or evaluate a literal | No, by itself | Eligible primitive constituents of a proof. |
| Invoke dynamically selected `Object.call` or overridable send | Target-dependent | Candidate body and source spelling are not proof of selected callee. |
| Evaluate argument/spread or selected default | Depends on constituent operations | Full call proof must include these phases. |
| Enter a loop or Boolean branch | No implicit suspension | Evaluation of selected bodies/callbacks may suspend. |
| Enter a selected callback | Target-dependent | Dynamic rebinding/override prevents assuming a literal candidate. |
| Enter Error handling or `ensure` cleanup | Cleanup/handler may suspend | Preserve exactly-once effects and continuation across cleanup. |
| Error, ReturnHome or cooperative cancellation | Not inherently suspension | Distinct control-transfer predicates; do not conflate. |
| Block worker in synchronous foreign call | Not guest suspension | Blocking does not create an Actor-local interleaving or C-prime yield. |
| Try suspension through synchronous foreign callback | Rejected by foreign boundary | Rejection path is not proof of callback purity. |
| Host-initiated non-Task-backed RootActor entry | Suspension is permitted | No eager Task requirement follows. |
| Truffle continuation-result/yield machinery | Physical mechanism | Does not redefine when guest execution semantically suspends. |

Normative authority inspected includes `spec/concurrency/FUTURES_AND_TASKS.md` (§23 and §29), `ACTORS.md`, `PARALLEL_EXECUTION.md`, `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `ERRORS.md`, `OBJECT_MODEL.md`, `VALUES_AND_COLLECTIONS.md`, `MODULES.md`; I/O owners `spec/io/IO_CORE.md`, `BYTE_IO.md`, `TEXT_IO.md`, `PROCESS_IO.md`, `NETWORK.md`; `spec/PROTOS_LANGUAGE_SPEC.md` and `spec/PROTOS_GRAMMAR.md`. A semantically mandatory function-level suspend annotation or public effect type was not found.

## 2. Proof domains — four distinct subjects

1. **Canonical body**: exact Closure definition, lexical binding analysis and scope. A proof describes all admissible execution paths of that body, *not* argument binding and dynamically selected call dispatch.
2. **Effective emitted root**: the actual root following authorized constant folds/emission decisions, profile and root group. The lowerer can prove a stronger no-suspension result here, without rewriting the original source-definition fact.
3. **Complete invocation**: receiver evaluation, ordinary `Object.call` method selection, argument/spread evaluation, selected default binding, target body, callbacks, handlers and `ensure` cleanup. A body-only fact never automatically proves this whole package.
4. **Conditional callsite**: exact selected target plus guard/identity/presence, Context/profile and other assumptions. A possible guard miss requires a semantically correct general fallback on that miss.

Keep **observed no suspension**, **proven no suspension**, **known possible suspension**, and **UNKNOWN** distinct. UNKNOWN is neither an error nor a proof that suspension will happen. A later producer may refine an unknown fact at a narrower/new scope; it cannot retroactively invalidate a fact already used by an earlier emitter. Proof identity and provenance must survive the relevant reparse and destination-Context requirements without leaking Builder-local handles.

## 3. Adversarial call-and-control matrix

| Case | Conservative classification | Specific obligation |
|---|---|---|
| `identity: (x) => { x }` | Body safe | Prove callee selection and argument/default path separately. |
| Thousands of nested safe conditionals | Safe if all reachable conditions/arms safe | Structural composition, bounded walk, memoized if shared; no branch-count cut-off. |
| Dynamic `x + y` / sends | UNKNOWN without target proof | User override/dynamic dispatch is semantically significant. |
| Override Closure instance `call` | UNKNOWN without full selection proof | Ordinary `Object.call` is selected dynamically. |
| Rebound/remove/recreate named binding (D179) | UNKNOWN without exact identity/presence proof | Same textual name is not the same proven binding. |
| Unselected default that could suspend | Depends on actual selected entry | D203 evaluates only the applicable default; selected default may suspend. |
| Return an unresolved Future | Safe as to that return alone | Creation/return is not `Future.value()` observation. |
| `Future.value()` on pending Future | Suspending path | Prior runtime resolved instances cannot prove universal safety. |
| Recursive/mutually recursive call graph | UNKNOWN without sound bounded recursion proof | No optimistic recursion or mandatory whole-program fixed point. |
| Boolean if/while with Closure callback | Selection-dependent | Preserve selected-only callback execution, receiver, ReturnHome. |
| Non-suspending try with suspending `ensure` | Potential suspension | Cleanup participates in full path and must not replay effects. |
| Error handler invoking arbitrary Closure | Potential suspension | Errors/unwind are separate, handler body is executable. |
| Synchronous foreign callback suspension attempt | Rejected on boundary | Preserve rejection/Error behavior, not a general purity claim. |
| Host RootActor without Task | Suspension still possible | No eager RootTask or scheduler allocation. |
| DEBUG or destination Context reparse | Valid only under matching facts | Root group, tags, lexical identities and target-profile emission remain correct. |

The number of branches is *not* the hard part: conjunction of independently proven safe constituents can scale with body size. Dynamic call targets are hard because a single reachable unresolved call invalidates an unconditional no-suspension result. Two trivial functions with the same source shape may have different admissible effects when their call selections or contexts differ.

## 4. Current implementation: precise producer/consumer deficiency

At the pinned source baseline:

- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalCompilationFacts.java` (I106) stores exactly four families of canonical proofs: isolated Closure binding analysis, own persistent-frame authority, ReturnHome observability, and binding-associated Closure-call candidates. Identity, analysis and scope lifetimes are explicit. Its current Javadoc correctly says *emitted-root* facts are not canonical source facts. These first four families are **not** the maximum scope of PLAT061-B.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java` retains a very conservative `provesNonSuspendingBody`: no defaults/rest, resolved parameter reads and literals. Calls, dynamic/captured lookups, writes, returns, nested closures, and unknown shapes currently fail closed. `effectiveBody` can replace a legally folded terminal with a literal.
- The lowerer records `recordProvablyNonSuspendingBody` **after** `builder.endRoot()`. This is useful emitted-root metadata, but comes too late to make earlier Builder-emission decisions at sites that need continuation handling.
- `emitContinuationComposition` emits a while-loop around `IsContinuationForCall`, `Yield`, and `ResumeContinuation`. `emitOrdinaryPreparedInvocation` can select current straight-line entry or the generic invocation/composition. Proof of body alone does not establish semantic reachability of the full invocation's generic branch.
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java` retains `provablyNonSuspendingBody`, direct-call admission and `IsContinuationForCall` specializations. Those downstream consumers remain legitimate; the gap is shared, early, composable fact ownership affecting emission.
- Review evidence in `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/`, especially I106 and PERF042 source/Builder-oracle tests. No tests were executed for this D204 research.

The historical 434-versus-38 emitted-Bytecode comparison uses a full semantic lowerer and a deliberately limited handwritten Builder oracle. It is *not* evidence of equivalent guest semantics or admissible compiled Tier2/BGV nodes, and not an established performance win. PERF042/#871 remains the independent final-tier benchmark acceptance authority.

## 5. Language/VM precedent: semantic contract != physical backend

1. **Kotlin:** source `suspend` function capability participates in type/call restrictions and continuation lowering; Protos has not selected that public contract.
2. **C#:** source `async`/`await`, state-machine production and completed-awaiter fast paths; ready on one run is not universal non-suspend proof.
3. **JavaScript:** `async` functions and Promises; returning a Promise does not equivalently suspend the synchronous Protos calling activation.
4. **Python:** `async def` and coroutine objects; invocation/await scheduling differ from Protos ordinary Closure and Future observation semantics.
5. **Rust:** `Future::poll` and `Ready`/`Pending`; runtime readiness differs from proof that a particular operation cannot suspend.
6. **Lua:** coroutine `yield`/`resume` with call-boundary restrictions; dynamic suspension is not necessarily statically annotated in a function type.
7. **Koka / effect inference literature:** recursive/transitive effect proofs are possible in sufficiently constrained static settings; a global guest effect type or whole-program analyzer is not therefore justified for dynamic Protos.
8. **Truffle Bytecode DSL:** physical yield/continuation operations express a backend strategy, not a new source-level language rule. Existing Protos C-prime nesting and continuation composition remain valid.

Relevant implementation evidence was compared with approved PLAT014, PLAT058/059/060 and PLAT061-B, including Truffle continuation paths. Peer techniques do not override Protos's spec.

## 6. GITHUB010 candidate comparison (12 axes, 1–5)

**5 = strongest fit to this demonstrated Protos problem; scores are bounded comparative judgments, not runtime measurements.** A = existing semantics + internal bounded fact proof (approved); B = add minor normative clarification; C = guest `suspend`/effects; D = new general/public guarded proof contract; E = defer/no implementation.

| Axis | A | B | C | D | E | Reason and confidence |
|---|---:|---:|---:|---:|---:|---|
| Semantic coherence | 5 | 4 | 2 | 3 | 4 | Current pending-Future/call semantics suffice (H). |
| Orthogonality/composability | 5 | 4 | 2 | 3 | 2 | Compiler proof subjects can compose without new syntax (H). |
| Completeness/edge cases | 4 | 4 | 3 | 3 | 1 | Conservative UNKNOWN and fallback remain mandatory (H). |
| Existing model consistency | 5 | 4 | 1 | 2 | 5 | PLAT061-B already owns facts contract (H). |
| Expressiveness/evolution | 5 | 4 | 3 | 4 | 2 | Future explicit effects remain optional (M). |
| Conceptual/user burden | 5 | 4 | 1 | 2 | 5 | No new language API (H). |
| Complexity/pay-as-you-grow | 5 | 4 | 1 | 2 | 4 | Bounded, demand-driven reuse (M). |
| Portability/implementation freedom | 5 | 5 | 2 | 2 | 5 | VM machinery stays independent from guest behavior (H). |
| Runtime/resource cost | 5 | 4 | 2 | 3 | 2 | Potentially omit unused physical machinery; unmeasured (M). |
| Tooling/diagnosability | 4 | 4 | 4 | 4 | 1 | Scoped proof provenance can help; implementation pending (M). |
| Migration/compatibility | 5 | 4 | 1 | 2 | 5 | No source/ABI migration (H). |
| Evidence maturity/risk | 4 | 3 | 2 | 2 | 3 | Existing I106 seam; still needs exhaustive emitted-code tests (M). |

**Strongest argument against A:** a conservative static inference may classify many dynamic calls as UNKNOWN. Broadening the analyzer could duplicate general effect-system complexity without delivering safely omitted fallback. This is mitigated by requiring concrete *general* pre-emission producer/consumer paths and independent physical oracles under I107, while retaining generic emission when a complete proof cannot be established. A benchmark-specific identity shortcut or mere root boolean is insufficient.

**Why not B/C/D/E:** B is needless normative churn absent a concrete ambiguity; C changes guest/API semantics, multiplies annotation and tool costs; D would duplicate PLAT061-B unless it changes public language guarantees; E leaves an approved contract underused for real compiler decisions. No speculative future-runtime costs are prepaid.

## 7. GITHUB021 invariants and boundary cost

Preserve D013 selected `Object.call` semantics, D179 binding identity/presence and by-reference lexical capture, D202/D203 positional/default/rest/spread, ReturnHome and genuine local/nonlocal control, Error and suspending `ensure`, cooperative cancellation, Future/Task/Actor and host/foreign boundaries. PLAT014 continuation/no-replay, PLAT058 one lowerer and NORMAL/DEBUG, PLAT059 no general canonical value/effect optimizer, PLAT060 B5 ordinary ABI and PLAT061-B scope-specific multi-producer/multi-consumer proofs remain untouched.

**Pay as you grow:** no mandatory Task/RootTask, scheduler/waiter, extra runtime capability table, per-call effect scan, second generated interpreter or whole-program analysis. Safe calls may omit unneeded physical continuation handling when full-path proof supports it; genuine potentially suspending paths must retain exact C-prime behavior. Context projection, DEBUG Source/RootTag and reparse remain first-class constraints.

**Unresolved implementation questions:** how much proof can be admitted without assuming immutable dynamic dispatch; how to publish an effective-root improvement before affected emission sites; what complete-invocation proof and guard ownership suffice to omit actual Bytecode; which no-suspension facts have two truly independent consumers. These are bounded I107 compiler obligations, not evidence of a missing guest-language semantic decision. If I107 uncovers a genuinely new semantics/architecture choice, STOP for fresh explicit owner approval.

## 8. Disposition and tests/evidence boundary

**D204:** Owner-approved Candidate A — `NO_NEW_SEMANTIC_DECISION_REQUIRED`. The original normative specifications stand; no new suspend keyword, function-kind split or mandatory effects contract.

**I107/#906:** Separate, general implementation of pre-emission composable proofs and *actual* safe omission of fallback Bytecode; no source code, test, performance, or version claim is made by this document.

**PERF042/#871:** Separate stable final-tier BGV/code-size/timing acceptance; no inference of speedup from static instruction counts.

**Human statement:** Maintainer reports local tests all PASS and `git diff --check` clean in the present interaction, but provided no run ID or checked-out product SHA; this is not an independent evidence that I107 has been implemented or published, and no tests were run by the agent in preparing D204.

**Inspected sources:** `AGENTS.md`, `AGENTS.work/{DESIGN,REFERENCE,COORDINATION}.md`, `spec/AGENTS.md`; normative domains listed above; implementation files listed in §4, plus `ProtosBytecodeRootNode.java`, `ProtosBytecodeClosureExecutionPlan.java`, I106 and PERF042 source-oracle tests, and D204/PLAT061/I106/PERF042 GitHub work records. Repository reads were read-only; the only publication in this step is the D204 documentation in project-docs.
