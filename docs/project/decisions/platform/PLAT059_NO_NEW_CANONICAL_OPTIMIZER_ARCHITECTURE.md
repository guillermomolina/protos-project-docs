# PLAT059 — No additional canonical-optimizer architecture required

**Outcome:** `NO_NEW_ARCHITECTURE_REQUIRED` — owner-approved resolution of an architectural investigation, **not** selection of a new optimizer, compiler phase, language semantic or runtime contract.
**Owner approval (2026-10-14):** The project owner replied **"ok apruebo"** to the PLAT059 decision packet recommending continuation of already-ratified PLAT058-D/I101 without a new optimizer architecture. That approval applies to this exact *no-new-architecture* disposition, not to a hypothetical future fact-analysis layer.
**Formal Issue:** [PLAT059/#894](https://github.com/guillermomolina/protos/issues/894).
**Existing architectural authority:** [PLAT058-D](PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md), ratified under [PLAT058/#891](https://github.com/guillermomolina/protos/issues/891).
**Implementation owner:** [I101/#892](https://github.com/guillermomolina/protos/issues/892), with A already published and B/C still pending; PERF043/#877 remains closed and PERF040/#862 and PERF046/#890 independently own numeric performance and comparison questions.
**Exact inspected Protos baseline:** [`8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff`](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff).
**Comparative upstream baselines:** GraalJS `3df14182071768bd4162734ca69e8b2a81eed5d4`, GraalPy `a099d006438942d54c1856c2cce7b6c2839b0c85`; further survey references in the [decision packet](../../evidence/PLAT059/PLAT059_FINAL_RESEARCH_AND_OWNER_APPROVAL_2026_10_14.md).
**Evidence:** [PLAT059 final 12-axis comparison, source audit, falsification, acceptance and approval](../../evidence/PLAT059/PLAT059_FINAL_RESEARCH_AND_OWNER_APPROVAL_2026_10_14.md). Previous [preliminary comparison](../../evidence/PLAT059/PLAT059_PRELIMINARY_CANONICAL_OPTIMIZER_COMPARISON_2026_10_14.md) and [GraalJS](../../evidence/PLAT059/PLAT059_GRAALJS_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md) / [GraalPy](../../evidence/PLAT059/PLAT059_GRAALPY_DETAILED_OPTIMIZATION_INVENTORY_2026_10_14.md) inventories remain unchanged historical evidence.

## Approved disposition and scope

1. **Keep PLAT058-D authoritative.** One shared proof-driven Canonical → Truffle Bytecode DSL lowerer; immutable Context-owned NORMAL/DEBUG physical-compilation profiles; deterministic/idempotent reparse; no runtime mode branch in admitted profile-fixed paths; minimum justified physical emission **before** Graal; every claimed elimination needs an independent handwritten Bytecode DSL Builder oracle including whole executable signature and root-group topology.
2. **Implement only justified current needs under I101-B/C.** Extend existing local immutable proof/emission decisions, binding/escape/suspension analyses, Boolean and Closure optimizations and admitted small pure arithmetic. Do not duplicate I101's existing implementation owner, create micro-slices or reimplement PERF043's scoped Integer optimizations.
3. **Defer a dedicated global/pre-bytecode canonical value/effect optimizer.** A bounded immutable per-site fact analysis (without rewriting the original Canonical AST) is an optional future candidate *only if* repeated cross-statement fact needs or duplicated proof work cannot be handled economically by the approved design. Such a materially new architecture would require a separate explicit owner decision, GITHUB010/GITHUB021 assessment and durable ratification.
4. **Reject a wholesale rewritten optimized Canonical AST as current scope.** Its additional node identity, source, Closure-root, binding-analysis, DEBUG and reparse integration risks are disproportionate to current proved needs.
5. **No semantic change.** Literal values, lexical binding identity and a known receiver are not authority to elide an ordinary overridable D013 message. D179 `PRESENT`/`ABSENT`, shadowing and by-reference Closure capture, evaluation effects, ReturnHome, suspension/cancellation, Context isolation and DEBUG observability remain binding. Failure of proof retains exactly the generic or guarded fallback; fully proven elimination must not pay an unnecessary runtime guard.

## Approved follow-on and unresolved static finding

I101-B should deliver the minimum coherent set of proof-driven reductions with independently authored Builder oracles, including `() => { true.ifTrue() { 1 } }` and applicable local-binding/Closure cases. I101-C owns full NORMAL/DEBUG, source/tag, Context, Task, reparse and negative semantic acceptance. Graph size and timings are supplementary, not substitutes for physical bytecode or behavioral tests.

**Potential arithmetic correctness issue — NOT VERIFIED:** At the inspected HEAD, `CanonicalToBytecodeLowerer.emitComposedSendUnfolded` has a literal `foldedIntegerAddition(send, Map.of())` branch that emits a constant without an obvious per-site canonical-send selection guard (around lines 8418–8429). By contrast, `ProtosSemanticBytecodeRootNode.TryPrimitiveNumericSendOne.canonical` checks selected canonical behavior, Context/Prelude, stability and actual operands (around lines 6775–6840). Static inspection alone does not establish a guest-observable defect or rule out another guaranteeing invariant. Audit the path and obtain an actual adversarial override/Context regression before assigning BUG authority or reopening closed PERF043. Do not silently treat the suspected defect as a verified failure or as a reason to introduce a new optimizer.

## GITHUB021 approval-invariant consistency

| Previously owner-ratified invariant | PLAT059 result |
| --- | --- |
| PLAT058 one proof-driven lowerer, pre-Graal minimal physical instructions | **PRESERVED** — no competing optimizer/emitter |
| Immutable Context-owned NORMAL/DEBUG physical profiles | **PRESERVED** — no ambient runtime-switch hack or obligatory late NORMAL→DEBUG conversion |
| Exact Builder oracle, complete typed constant operands and root groups | **PRESERVED** — each new optimization retains this acceptance |
| D013 ordinary overridable messages, D179 live lexical Context slots | **PRESERVED** — no constant-result bypass without selection/presence/effect proof |
| Genuine Closure capture, ReturnHome, Task and observable effects | **PRESERVED** — fallback upon failed admission |
| Pay as you grow; no universal Task/Actor/Process/activation machinery | **PRESERVED** — no new per-program compulsory optimizer/runtime objects |
| Stable deterministic Bytecode DSL reparse, Context isolation and debug provenance | **PRESERVED** — original Canonical AST and analysis authority |

**Approval delta:** no existing architectural or semantic invariant is reopened. The owner selected continuation within PLAT058, not a new independent optimizer architecture. No new platform decision is required for routine internal implementation consistent with the above rules.

## Publication / validation boundary

This is documentation and issue coordination only. **No `guillermomolina/protos` source, tests, `pom.xml`, `CHANGELOG.md`, runtime or specification were changed** by PLAT059 resolution. The owner reports `git diff --check` clean and all local tests passing; no test scope/count/commit for those statements was supplied here, and this research did not run tests/builds/benchmarks or establish an implementation gain. This closure does not close I101, prove the suspect arithmetic defect or assert benchmark parity.
