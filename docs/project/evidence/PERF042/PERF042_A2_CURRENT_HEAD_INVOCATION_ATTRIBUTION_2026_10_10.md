# PERF042-A2 — Current HEAD direct Closure Call1 investigation and bounded final research handoff

Date: 2026-10-10  
Formal owner: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871)  
Type: **INVESTIGATION** (not implementation or architecture approval)  
Product repository: `guillermomolina/protos`  
Benchmark repository: `guillermomolina/protos-benchmarks`  
Evidence repository: `guillermomolina/protos-project-docs`  
Status: **Source-level findings recorded; exact compiler-decision/IR-edge attribution blocked by inaccessible retained compressed payloads; PERF042 remains OPEN.**

## 1. Authority, revisions, and provenance

- Protos `main` source revision reviewed: **`7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`** (I092 compact Boolean work). Relative to A1's `76de43651079172d90be5bc6b1f535336ae47cfb`, exactly one later commit was found.
- Benchmarks `main`: **`527035f01107e6c71f860e95f438b7553a5428a2`**.
- Project-docs `main` prior to this publication: **`07ce1540ba26a09ab8efbaf591d85f632f2e55c2`**.
- Historic compiler comparison uses GraalVM **25.4.4.1.1** and pinned Oracle GraalJS `58f1795f7d12d12260d0c0423811e846c7632649`, GraalPy `89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3`, and Graal `95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`.
- Governing project guidance inspected: product `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/IMPLEMENTATION.md`, docs `AGENTS.md`. Primary normative owners read: `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`. Ratified PLAT040, PLAT042, PLAT044 and PLAT046 were reviewed. PERF038-H is historical companion evidence, not PERF042 ownership.
- Owner reports local `git diff --check` CLEAN and all local tests PASS. Exact test command, runtime/run SHA, and counts have **not** been provided. No tests, builds, programs, benchmark captures, Git commands, or compiled-graph diagnostics were executed by the investigating agent. These human-reported checks do not establish current-HEAD Tier-2 admission.

Earlier pinned checkpoint: [PERF042-A1](PERF042_A1_SOURCE_AND_GRAPH_INVESTIGATION_2026_10_10.md). This A2 report supersedes its assumptions only where current source provides additional evidence; it does not rewrite that historical snapshot.

## 2. Canonical case

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

Expected result: `1`. It is a **nested, fresh, local Closure with one positional argument** (C1). It is neither C0 (zero arguments), an object method send, nor C2 (actual guest argument collection/Array introspection). Keep the benchmark unchanged.

## 3. Exact historical graph eligibility: two distinct captures

| Capture | Product | Harness | Correctness | Graph eligibility | Selected Protos root |
| --- | --- | --- | --- | --- | --- |
| Clean 2026-10-08 | `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` | `98abc9af7a05a45a7d4056b72f36aef5889cb76f` | PASS | STABLE [16000,64000], Tier 2, child inlined, valid | 4,533 nodes |
| Later 2026-10-09 | `0db24f00ff2d92d642351d7f7535517fe01a55ce` | `d9719253b628faad1ca296daef4a7ed566196216` (**dirty**) | PASS | GRAPH_NOT_STABLE, UNIT_NOT_AT_FINAL_TIER, invalid evidence | 2,349 **Tier-1 only** nodes; separate Cutoff child 116 Tier-1 nodes |

The admitted clean reference has **GraalJS 13, GraalPy 50, Protos 4,533** nodes; Protos's nested Closure was inlined. It **predates PERF038-H**. No current-HEAD stable final-tier Protos graph was available to this investigation. Never use 2,349 as a valid post-H Tier-2 baseline, or infer speedup by comparing different revisions.

Read-only retained `capture.json` exposes the exact sequence:

| Call budget | 2026-10-08 compilation count / primary tier | 2026-10-09 compilation count / primary tier |
| ---: | --- | --- |
| 1,000 | 3 / T1, child Cutoff | 4 / T1, child Cutoff |
| 4,000 | 3 / T1, child Cutoff | 4 / T1, child Cutoff |
| 16,000 | 6 / **T2, child inlined** | 5 / T1, child Cutoff |
| 64,000 | 6 / **T2, child inlined** | 5 / T1, child Cutoff |
| 256,000 | not executed after admission | 5 / T1, child Cutoff |

The clean capture **also had an initial child Cutoff at Tier 1**. Cutoff alone is therefore not an explanation of the later missing Tier 2. Later capture reports zero failures and no selected primary-root post-final invalidation; its one overall invalidation per budget cannot be assigned to the primary root without trace ownership. No repeated deoptimization cycle or bailout was established. Upstream `truffle/docs/DeoptCyclePatterns.md` describes distinct patterns, but none has been proven to occur in this capture.

[Clean unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/protos/unit.json) · [Clean capture](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/protos/capture.json) · [Later unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/unit.json) · [Later capture](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/capture.json).

## 4. New source-level findings at product HEAD

### H1 — Repeated fresh Closure identity prevents the fused direct Call1 identity PIC from converging

`CanonicalToBytecodeLowerer.emitComposedCall` opens `TryDirectCallOne` for ordinary fixed-arity C1 calls; a miss goes to `PrepareClosureCallOne`. The canonical benchmark constructs a fresh `ProtosClosureValue` for `identity` every run.

- `ProtosSemanticBytecodeRootNode.TryDirectCallOne.guardedSource` uses `receiver == cachedReceiver`, entered-context and prelude identity, plus a guarded `Object.call` selection assumption, and `TryDirectCallOne.generic` returns `MISS`.
- `PrepareClosureCallOne.guardedDirect` likewise keys on receiver identity, **but** `PrepareClosureCallOne.fastDirect` keys on a stable `CanonicalClosure` definition and entered context, re-performing ordinary canonical `Object.call` selection for each receiver. Thus Protos already has a compatible per-definition selection strategy but **not at the fused Call1 entry**.
- `ProtosBytecodeRootNode.directClosureCallSelectionForPreludeOrNull` specifically reselects the ordinary `call` slot and rejects noncanonical selection. The candidate must preserve this behavior (a local override or shadowing of `call` is legal); it is not permissible to reuse a stale effective callable solely because a Closure definition matches.
- The fused path uses `compactDirectClosureCallOneNoHome` to avoid `OrdinarySourceCall`; `finishDirectClosureCallOne` and its `PreparedClosureCall` remain useful for generic/alternative calls.
- **Source causal confidence: high; exact Tier-2 surviving-IR impact: not established**. The historical Tier-1 post-H expansion includes `PrepareClosureCallOne_Node` (179 expansion units, 18 conditional expansions in the retained report), but expansion counts are not executable-path, graph-node-removal or allocation counts.

Candidate: a bounded by-definition guarded admission on `TryDirectCallOne` that composes the already correct per-definition `call` re-selection with the direct fused Call1 entry, retaining all other fallbacks, arity, result and error semantics. **No implementation authorization yet.**

### H2 — Persistent lexical authority admitted conservatively for any nested Closure

- In `CanonicalToBytecodeLowerer.requiresPersistentFrameAuthority`, `demandsPersistentFrame(CanonicalClosure)` returns `true` unconditionally. The outer `run` has a locally declared binding `identity` and contains a nested Closure, so its body requests `InstallFrameLexicalAuthority` with `emitCurrentActivation` before the body.
- The authority installs a `MaterializedFrame` via `frame.materialize()`, retaining the same frame-backed lexical binding store for escapes, observation, mutation, and tooling. This is a **real lowering requirement**, not evidence that a guest-visible Context object is eagerly allocated or that every boundary invoke runs on the success path.
- I092 on `7b609c4a` introduced `MaterializeCurrentClosure`, `ProtosFrameArguments.materializeCompactClosure`, and `ProtosLexicalEnvironment.deferredCompactOwner`. However `useCompactClosureCapture()` requires both `currentActivationLocal == null` and `currentRootFrameLocals.isEmpty()`. The outer benchmark root declares `identity`, so that new pathway does **not** automatically remove its persistent authority/activation requirement.
- Semantics require one real current execution context, reference-captured lexical relationships, exact slot `PRESENT/ABSENT` behavior, mutations/escape, and debugger observation. The fact that `identity(value) { value }` does not read a lexical outer variable is **not** proof that the enclosing context may be destroyed or reconstructed under all observers.

Candidate to prove, not presume: lazy retainable frame authority for nested Closures **with frame-owned bindings**, retaining exact shared context identity and reference semantics on late observation. High potential IR impact, **high semantic risk**. No blanket deletion/bypass of authority is warranted.

### H3 — Caller references/materialization differ between sends and direct calls

- `emitInvocationCallerOperand` still emits `CurrentActivation` for ordinary root-level composed calls.
- For sends, `emitSendCallerOperand` can instead emit `CurrentCallerReference`, which returns a compact frame-argument carrier and materializes only in specializations requiring the exact caller.
- This is a genuine physical distinction to investigate together with H1/H2, not a separate one-off micro-slice. It is **not** proven that every composable call admits a compact caller without identity, Task, ReturnHome, error-order or tooling changes.

## 5. The whole invocation package, existing optimization and residual families

Source components inspected in Protos:
- `execution/CanonicalToBytecodeLowerer.java`: nested Closure lowering, capture admission, call staging, Call1 emission, caller seams.
- `execution/ProtosSemanticBytecodeRootNode.java`: guarded direct Call1, CurrentActivation/CurrentCallerReference, frame-native parameter bind/read, captured access, continuation, exception/root-tag behavior.
- `execution/ProtosBytecodeRootNode.java`: ordinary and fused direct entry, prepared carriers, guarded `call` selection, frame-backed lexical authority installation and exception bridging.
- `execution/ProtosFrameArguments.java`: ABI, compact final Call1 argument placement and lazy rich activation.
- `execution/ProtosBytecodeClosureExecutionPlan.java`: true semantic Closure root target, return-home proof and parameter-binding plan.
- `execution/ProtosFrameLexicalBindingAuthority.java`: physical frame authority, presence checks, establishment order, transition/adoption.
- `runtime/ProtosLexicalEnvironment.java`: deferred capture by reference and I092 compact owner.
- `runtime/ProtosActivation.java`: one exact Context identity, captured-environment chain, return-home/Task optional state.
- `runtime/ProtosClosureValue.java`: Closure identity, capture, unobservable ReturnHome marker.
- `runtime/ProtosLexicalBindingAuthorityCalls.java`: `@TruffleBoundary` fallback lexical operations.
- `runtime/ProtosObjectValue.java` and `ProtosValueLookup.java`: semantic slot lookup, dependencies, invalidations and generic delegation.
- `execution/ProtosInvocation.java`, `ProtosClosureInvoker.java`: host/caller invocation boundaries.

| Stage / family | Existing mechanism or confirmed source producer | What remains to verify in IR |
| --- | --- | --- |
| Local Closure creation/capture | Fresh `ProtosClosureValue`, captured `ProtosLexicalEnvironment` | Surviving object/authority representation and escape |
| `identity` binding/lookup | Frame-backed/local binding and semantic fallback | Whether persistent-owner installation dominates ordinary call |
| Canonical `call` lookup/PIC | Guarded identity and definition-level PICs | Miss behavior and after-Tier graph footprint |
| Target/direct entry | `DirectCallNode`, fused `TryDirectCallOne`; prepared generic alternative | Inlining and exact compiler Cutoff decision |
| Positional argument | C1 final frame write, no intermediate supplied array on admitted path | Any residual generic vector/list branches, not duplicate work |
| Frame/native parameter | Fixed ordinal binding and native read, presence-aware fallback | Cold vs hot validation/metadata |
| Activation/authority | Conditional `materializeCompactActivation`; outer eager authority admission | Which surviving invokes are reached on normal success |
| Return-home/control | `ProtosReturnHome.unobservable()`, no-home entry, proven non-suspending body | Remaining error/unwind/continuation alternatives |
| Task/Actor/Process | PLAT046 ordinary caller-thread, no universal RootTask per trivial call | No evidence of task-allocation tax in this case |
| Tooling and PE | RootTag and scope projection; Bytecode DSL generated executor | `FrameState`, guards and deopt edge ownership |

Already implemented: `PrepareClosureCallOne`, guarded PIC, `finishDirectClosureCallOne`, `compactDirectClosureCallOnePrepared`, C1 direct placement, deferred argument Array and activation, `OrdinarySourceCall` separation, no-home and nonsuspending fast paths. Do not repeat PERF038-H. C0, C1 and actual C2 introspection remain distinct physical cases.

### Historic 44 invokes: full target census (sites, not runtime calls)

| Target | Sites | Source/family; hotness cannot be inferred |
| --- | ---: | --- |
| `ProtosObjectValue.invalidateTrackedLookupDependencies` | 8 | Mutation/lookup invalidation |
| `ProtosFrameArguments.materializeCompactActivation` | 7 | Rich activation transition |
| `ProtosLexicalBindingAuthorityCalls.put` | 6 | Generic lexical mutation |
| `ProtosBytecodeRootNode.selectGuestHandlerOnRootCrossing` | 5 | Error/handler crossing |
| `ProtosLexicalBindingAuthorityCalls.contains` | 4 | Generic lexical presence |
| `ProtosLexicalBindingAuthorityCalls.transferAll` | 4 | Authority transition |
| `List.size` | 2 | Generic list operation; exact producer/edge unresolved |
| `ProtosSemanticBytecodeRootNode.ReadRootFrameLocal.slowRead` | 2 | Noncompact/read fallback |
| `List.get` | 1 | Generic list operation; exact producer/edge unresolved |
| `ProtosFrameLexicalBindingAuthority.adoptPresentFrameBackedBindingsSlow` | 1 | Late authority adoption |
| `ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt` | 1 | Presence-preserving binding creation |
| `ProtosLexicalBindingAuthorityCalls.read` | 1 | Generic lexical read |
| `BytecodeRootNode.getBytecodeNode` | 1 | Bytecode metadata |
| `ProtosValueLookup.representedDelegationParent` | 1 | Generic delegation lookup |
| **Total** | **44** | **No live-edge or removable-count assertion** |

Other admitted historical IR: **997 `FrameState`**, **285 `IfNode`**, **201 `InstanceOfNode`**, **165 `FixedGuardNode`**, **45 `DeoptimizeNode`**, **43 `AllocatedObjectNode`**. Graph nodes in these groups overlap in cause; counts are not necessarily independent, hot, physical allocations or removable.

The historical source expansion also includes the generated `CachedBytecodeNode`, `PrepareClosureCallArguments`, `InstallFrameLexicalAuthority` and `FinishClosureCall`. Their reported expansion attribution is non-exhaustive and is not a partition of After-TruffleTier nodes.

## 6. Exact upstream comparison

- GraalJS `JSFunctionCallNode.java`: distinct `Call0Node`, `Call1Node`, `CallNNode`; `JSFunctionData` caching can supersede instance caching for fresh functions; cached `DirectCallNode`. `JSArguments.java` allocates the final one-argument frame array with **two** runtime slots; it does **not** eliminate every frame array or function semantic state.
- GraalPy `CallNode.java` and `CallDispatchers.java`: cached function identity with a secondary **code/root**-based specialization, protected by relevant code stability assumptions and direct-call targets; `CreateArgumentsNode.java` prepares a signature-correct frame array. `PArguments.java` has **six** header slots before positional parameters. A larger header does not prove a larger compiled hot graph.
- The equivalent reusable idea for Protos is *not* to copy JS semantics: ordinary `Object.call` is an overridable slot, and re-selection/invalidations remain mandatory. The by-definition PIC already in `PrepareClosureCallOne` supplies the specific precedent to combine with the fused Call1 shape.
- Neither upstream implementation establishes an excuse to suppress Closure identity, Context objects, errors, returns, escape, observer state or tooling.

Pinned sources: [JSFunctionCallNode](https://github.com/oracle/graaljs/blob/58f1795f7d12d12260d0c0423811e846c7632649/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java), [JSArguments](https://github.com/oracle/graaljs/blob/58f1795f7d12d12260d0c0423811e846c7632649/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSArguments.java), [Py CallDispatchers](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java), [PArguments](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/function/PArguments.java), [CreateArgumentsNode](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/argument/CreateArgumentsNode.java), [DeoptCyclePatterns](https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/docs/DeoptCyclePatterns.md).

## 7. Ranked hypotheses, 0–5

Score columns: source backing, graph backing, causal confidence, expected structural impact (not measured speedup), **semantic risk** (5 = high).

| Hypothesis | Source | Graph | Cause | Impact | Risk |
| --- | ---: | ---: | ---: | ---: | ---: |
| H1 fused C1 receiver-identity-only PIC misses fresh Closures | 5 | 2 | 4 | 4 | 3 |
| H2 eager persistent outer lexical authority for nested Closure | 5 | 3 | 4 | 5 | 5 |
| H3 unnecessary exact caller activation relative to lazy caller reference | 5 | 2 | 3 | 3 | 4 |
| H4 residual lexical fallback/materialization expands IR | 5 | 5 | 2 | 4 | 4 |
| H5 error/guard/frame-state recovery footprint | 5 | 5 | 2 | 4 | 4 |
| H6 residual compact header size | 5 | 1 | 1 | 2 | 3 |
| Eager guest argument Array on admitted C1 | 0 | 0 | 0 | 0 | 2 |
| Universal RootTask per trivial call | 0 | 0 | 0 | 0 | 4 |

## 8. Mandatory semantic invariants and permission boundary

- One semantically fresh Closure identity for every evaluation, even when the compiled target/metadata are shared.
- `call` is ordinary D013/OBJECT_MODEL lookup: dynamic shadowing, mutation, selected Closure, correct receiver/methodHome, stability invalidation and error order.
- Capture genuine lexical Context **by reference**, not value copy; identity, escaped Closure, reflective access, structural presence/removal, OPEN/CLOSED/FROZEN state, mutation, delegation and debugger behavior must remain identical.
- All receiver/arguments are evaluated before dispatch/activation; left-to-right positional/default/rest/spread and parameter error precedence; no eager guest Argument Array for unobserved C0/C1.
- Fresh semantic activation; correct return-home creation/capture and nonlocal return; suspension/continuation, Task/Actor/Process and instrumentation behavior when actually used.
- No bypassing existing PLAT040/042/044/046, no new language architecture implied by a performance investigation.
- No predicted speedup, allocation reduction or exact removable-node total without final-tier graph edges and A/B validation.

## 9. Precise evidence gap and STOP

The GitHub connector can read the retained `capture.json` and `unit.json`, including compilation IDs/tiers, counts, selected graph names and invoke-target histograms, **but could not decode/read the compressed binary** `trace.log.gz` or `.bgv.gz`/`.filter.json.gz` at source. No complete BGV incoming control/data edges were available. Consequently:

1. Why the late root did not reach Tier 2 is **unresolved**, beyond its exact observed Tier-1-only admission.
2. Which of the 44 invokes and major `FrameState`/`IfNode`/guard groups survive on the successful ordinary path versus only exception, deopt, instrumentation or eliminated paths is **unresolved**.
3. No semantic proof currently permits removing the eager nested-Closure lexical authority for a root with its own bindings.

**Minimal separately authorized human diagnostic**, if the research agent cannot consume retained compressed artifacts read-only: decode existing `trace.log.gz` and relevant existing `*.filter.json.gz`/BGV into narrowly scoped textual tables of compilation IDs/root/tier/inlining/Cutoff reasons/invalidation ownership and control/data edges of the 44 invoke sites and dominant guards/frame states. Do not capture new benchmarks, change workloads, increase warmup or run suites merely because decoding is inconvenient. This is a request for retained evidence, not permission to execute commands in the investigation.

## 10. One consolidated next investigation, not a micro-slice chain

**PERF042-A3 — COMPLETE CAUSAL CLOSURE INVESTIGATION**, `TYPE=INVESTIGATION`. This is intended to be the **final investigation handoff** under PERF042, covering compiler tier/admission, all residual direct-invocation/lexical/control costs, cross-Truffle comparisons, semantics, source-to-IR causal attribution and recommendation **as one package**. Do not automatically subdivide A3 into A4/A5/etc.

The research agent reads the full existing retained evidence; if compressed edge/trace artifacts cannot be accessed under investigation's no-execution rule, it **must finish its report with the exact access blocker and smallest human diagnostic**, not invent causes, open repeated investigation slices, or request redundant tests. The human alone can authorize/run a later diagnostic. Once sufficient evidence is provided, the same unified investigation can be completed against it without manufacturing another formal issue.

**Conditional grouped implementation candidate**, only after evidence and normative proof: converge a per-definition, properly revalidated `Object.call` lookup with the fused `TryDirectCallOne` entry, potentially fixing avoidable caller materialization *within the same bounded change* if proven. Primary potential files: `ProtosSemanticBytecodeRootNode.java`, `ProtosBytecodeRootNode.java`; `CanonicalToBytecodeLowerer.java` and lexical/caller collaborators only if an independently proved physical need arises. Eager persistent capture deferral is **not** approved by current evidence and requires its own complete semantic proof before inclusion. A PERF-internal implementation slice does not automatically require an Ixxx; create a formal new Issue only if genuinely independently closable per `AGENTS.work/COORDINATION.md`.

**HUMAN EXECUTOR A3:** no commands required now, reported clean `git diff --check` and all local tests PASS noted; any diagnostic later must be separately approved and strictly derived from retained evidence.

No product/spec/benchmark source mutation, no design decision or implementation authorization, no Issue closure.
