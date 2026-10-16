# D204 — Suspension capability and proof boundaries of Closure invocation

**Role:** durable, non-normative, owner-ratified language-semantics *no-change* decision.  
**Status:** RATIFIED — Candidate A, 2026-10-16.  
**Authority:** [D204/#905](https://github.com/guillermomolina/protos/issues/905).  
**Exact owner approval:** “entonces apruebo A, para este ISSUE, pero quiero implementar esta falencia, y que el lowerer lo utilice, (no solo el PLAT), que sea un caso general.”  
**Decision token:** `NO_NEW_SEMANTIC_DECISION_REQUIRED`.  
**Product revision inspected:** [`d1697153256dc3952008cb40e507f09aea0f7101`](https://github.com/guillermomolina/protos/commit/d1697153256dc3952008cb40e507f09aea0f7101).  
**Investigation and falsification evidence:** [D204 research](../../evidence/D204/D204_A_SUSPENSION_PROOF_FALSIFICATION_2026_10_16.md).  
**Separate implementation:** [I107/#906](https://github.com/guillermomolina/protos/issues/906), consuming [PLAT061-B/#903](https://github.com/guillermomolina/protos/issues/903).

## Exact ratified selection

**Candidate A — existing normative Protos semantics are sufficient to distinguish potential suspension from a compile-time proof of no suspension.** No user-visible semantic addition is needed to implement a conservative, scoped proof mechanism. The missing capability is *compiler analysis, sharing, refinement, and physical emission planning*, not a new effect discipline for Protos.

This decision does **not** assert that every Closure is statically classifiable or that any syntactically simple call can never suspend. A conservative `UNKNOWN` answer is correct when dynamic behavior, mutable bindings, callback selection, defaults, host entries or a transitive call are not demonstrably bounded. The lowerer must preserve full continuation/dispatch machinery on those paths.

The owner additionally requested a **general** implementation whose lowerer both consumes and contributes proven facts, not a specialized PLAT-only flag, post-emission annotation, or optimization solely for `identity(1)`. This request is tracked under I107 and uses the already-ratified PLAT061-B lifecycle.

No new `suspend` modifier, async/sync callable kind, effect type, annotation, public reflection property, language syntax, exception category, public ABI, implicit task, or assumption of immutable `Object.call` is approved.

## Distinct subjects and validity

1. **An observed execution** that happened not to suspend establishes no universal static guarantee.
2. **Canonical body:** proof for the exact canonical definition, binding analysis and lexical scope. It concerns that body's admissible evaluations, not automatically its invocation prologue or caller expressions.
3. **Effective emitted root:** proof for the actual operations emitted after authorized compile-time transformations, including entry/default behavior. It may be stronger than a proof of the source body, but must have its *own* authority and profile validity.
4. **Complete invocation:** additionally covers receiver, selected ordinary `Object.call`, argument/spread evaluation, sequential parameter/default binding, selected callback execution, cleanup and any transitive operations. A safe body does not prove this larger subject.
5. **Conditionally admitted site:** a claim under concrete target/binding/Context/profile/selection assumptions is valid only while those assumptions remain guaranteed or while a correct runtime guard routes every miss into the full fallback.
6. **UNKNOWN, invalidated and contradictory:** lack of proof is not proof of suspension. Incompatible facts must fail closed; an already-consumed proof must not be silently overwritten. The compilation pipeline must not conflate semantic impossibility, conservative static uncertainty, and an invalid analysis scope.

Compiler stages may **produce, consume and refine** facts in their own validity scopes. This is expressly authorized by PLAT061-B and does not require a separate language-level decision.

## Normative anchors (unchanged)

- [`spec/concurrency/FUTURES_AND_TASKS.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/concurrency/FUTURES_AND_TASKS.md), especially pending `Future.value()`, resumed observation, cancellation and §29 host-initiated suspendible RootActor entries: pending observation may suspend; resolved observation returns immediately; no OS-thread block is implied by suspension.
- [`spec/semantics/CALLABLES.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/CALLABLES.md), ordinary dynamic `Object.call`, evaluation order, D203 positional/default/rest/spread, return homes and dynamic selection. Parameter binding introduces no hidden suspension point; evaluating a default may encounter an existing explicit suspension point.
- [`spec/semantics/EXECUTION_AND_CONTROL.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/EXECUTION_AND_CONTROL.md), explicit suspension of structured loops and `ensure`, no replay of committed work or premature cleanup.
- [`spec/semantics/ERRORS.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/ERRORS.md): Error signaling, unwinding and `ensure` are distinct from suspension and do not create a general resumable Error.
- [`spec/semantics/OBJECT_MODEL.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/semantics/OBJECT_MODEL.md), [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/d1697153256dc3952008cb40e507f09aea0f7101/spec/concurrency/ACTORS.md), and applicable foreign/host/I/O authority: dynamic lookup and execution/ownership boundaries remain authoritative.

A Future-producing operation is not automatically a suspension. Error, `ReturnHome`, cancellation, scheduler yield, blocking a host thread and continuation-machine instructions are not synonymous. An actual guest suspension remains governed by its existing explicit semantic boundaries.

## GITHUB021 prior-invariant reconciliation

| Existing authority | D204-A effect |
| --- | --- |
| D013 / `Object.call` | Unchanged dynamic selection; overrides and indirect sends cannot be assumed safe. |
| D179 / lexical Context | Captures by reference; `Resolved` binding identity is not a permanent proof of PRESENT state or selected value after remove/recreate. |
| D202/D203 / I104/I105 | Exact argument/default/rest binding and evaluation order remain mandatory; missing positions can select suspendible defaults. |
| `ReturnHome`, Error/ensure and cancellation | Independent control/unwind/lifetime contracts, never inferred from the no-suspension fact. |
| PLAT014 | Existing C-prime continuation and resume/no-replay semantics remain the fallback authority. |
| PLAT057 / I096 | Guarded source/literal candidate inlining retains dynamic selection and physical fallback. |
| PLAT058 | Single canonical-to-Bytecode lowerer and NORMAL/DEBUG profile differentiation remain. |
| PLAT059 | No general optimizer or transformed IR is authorized. |
| PLAT060-B5 | Five-header-slot universal ordinary-call ABI unchanged. |
| PLAT061-B / I106 | Existing bounded, composable pre-Builder facts are extended in I107; lowerer is both producer and consumer. |
| PERF042 | Static Builder instruction counts are not runtime performance or semantic-removability evidence. Acceptance remains independent. |

**Invariant-consistency conclusion:** `PASS` — Candidate A changes no previously ratified observable behavior and adds no architecture outside the separately approved PLAT061-B boundary.

## Implementation routing, non-goals, and proof obligations

I107 must implement a demand-driven, composable no-suspension proof, with exact identity/scopes, an explicit UNKNOWN result, pre-emission lowerer consumption, a legitimate emitted-root refinement, and physical suppression of continuation/fallback instructions *only on paths proved completely safe*. It must verify actual emitted Bytecode and retain complete fallbacks on guards' misses. Long sequences and nested safe conditionals must not require unbounded special-case checking or impose a fixed runtime tax.

Proving only `CanonicalToBytecodeLowerer.provesNonSuspendingBody` and publishing `ProtosSemanticBytecodeRootNode.provablyNonSuspendingBody` after `builder.endRoot()` is insufficient for the pre-emission objective. A body proof cannot by itself remove an ordinary dynamic `call` fallback or a selected callback/cleanup continuation.

No normative `spec/` update is required for Candidate A, because it selects **no semantic change**. No product source, build, test, benchmark, ABI, version, or release result is claimed by this record. The implementation and acceptance of I107 remain separately human-validated; full PERF042 acceptance remains separately owned.

## Closure provenance

- `DECISION=D204-A`
- `OWNER_APPROVAL=2026-10-16`
- `DECISION_STATUS=RATIFIED_BY_DURABLE_PUBLICATION`
- `NORMATIVE_SEMANTIC_CHANGE=NO`
- `PLAT061_B_REOPENED=NO`
- `GENERAL_PROOF_IMPLEMENTATION=I107/#906`
- `PRODUCT_SOURCE_REVISION=d1697153256dc3952008cb40e507f09aea0f7101`
- `PRODUCT_TESTS_EXECUTED_FOR_DECISION=NO`
- `PERF042_PERFORMANCE_ACCEPTANCE=INDEPENDENT`

The exact `PROJECT_RECORD_REVISION` is assigned by the documentation publication commit and must be read back before D204's GitHub Issue is closed.
