# PERF042-A1 — Source and retained-graph investigation; bounded A2 handoff

Date: 2026-10-10
Live issue: https://github.com/guillermomolina/protos/issues/871
Owner: PERF042 (independent of PERF038/#852)
Type: **INVESTIGATION**. No implementation, no product change, no benchmark execution, and no design approval.
Status: **Investigation checkpoint, not PERF042 closure**.

## Authority, scope, and validation provenance

- Protos source HEAD inspected: **76de43651079172d90be5bc6b1f535336ae47cfb**.
- Benchmark HEAD inspected: **527035f01107e6c71f860e95f438b7553a5428a2**.
- Project-docs HEAD when this investigation was reconciled: **2ee1b97177f4e146b3b09c0e92f99aa6466cf703**.
- GraalVM **25.4.4.1.1**, Java **25.0.4.1.1**.
- Comparison source: oracle/graaljs tag vm-25.4.4.1.1 / **58f1795f7d12d12260d0c0423811e846c7632649**; oracle/graalpython same tag / **89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3**; oracle/graal same tag / **95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b**.
- The maintainer reported on 2026-10-10 that local git diff --check was clean and all local tests passed. No exact test-run SHA, commands, counts, or CI result were provided for this handoff. These are **human-reported checks**, not tests executed by the agent and not verification of a new Tier-2 benchmark.
- Source and historical graph JSON inspection only. Compressed BGV edge data and compilation trace contents were **not decoded**. Source reachability and node histograms cannot substitute for exact BGV control/data-flow attribution.
- Governing instructions: protos AGENTS.md, AGENTS.work/{PERFORMANCE,COORDINATION,IMPLEMENTATION}.md, protos-project-docs AGENTS.md, DOC002 documentation path contract. Semantic authorities: spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL,OBJECT_MODEL}.md and ratified PLAT040, PLAT042, PLAT044, PLAT046.

## Canonical case, unchanged

~~~protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
~~~

Result 1. The benchmark is a **single-positional-argument direct Closure invocation**, NOT a zero-argument invocation, an instance method send, or argument-array introspection. GraalJS and GraalPy implement equivalent high-level identity calls, but not identical observable language semantics. The canonical workload remains unchanged.

- [Protos workload](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/truffle/workloads/primitive-closure-call/primitive-closure-call.protos)
- [JS workload](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/truffle/workloads/primitive-closure-call/primitive-closure-call.mjs)
- [Python workload](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/truffle/workloads/primitive-closure-call/primitive-closure-call.py)

## Two different historical compiled-graph states: NEVER merge their admissibility

### Clean reference, 2026-10-08, product d59da442

Product: **d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae**.
Harness: **98abc9af7a05a45a7d4056b72f36aef5889cb76f**, clean.
Source-hash and JVM/capture metadata retained in the case JSON.

[Protos admitted unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/protos/unit.json)
[JS admitted unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/js/unit.json)
[Python admitted unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/python/unit.json)

- All three: correctness PASS, stable pair [16000,64000], evidence_valid=true, After TruffleTier, retained selected graph.
- Protos: **Tier 2**, child Closure **inlined**, 4,533 relevant graph nodes; GraalJS **13**, GraalPy **50**.
- Protos counts: 997 FrameState, 285 IfNode, 201 InstanceOfNode, 165 FixedGuardNode, 45 DeoptimizeNode, 44 InvokeWithExceptionNode, 43 AllocatedObjectNode. Selected families include 321 load nodes, 241 guards/deopts, 293 control splits, 86 allocations in the analyzer's family classification; these are different accounting views, not mutually additive buckets.
- The 44 invoke targets: invalidateTrackedLookupDependencies 8; materializeCompactActivation 7; lexical put 6, contains 4, transferAll 4, read 1; selectGuestHandlerOnRootCrossing 5; List.size/get 3; adoptPresentFrameBackedBindingsSlow 1; createFrameBackedBindingAt 1; ReadRootFrameLocal.slowRead 2; BytecodeRootNode.getBytecodeNode 1; ProtosValueLookup.representedDelegationParent 1. These are **graph invoke sites, not runtime execution counts or removable-node quantities**.
- Truffle expansion prominently includes the generated CachedBytecodeNode, PrepareClosureCallArguments, InstallFrameLexicalAuthority and FinishClosureCall producers. Attribution expansion rows are not an exhaustive independent partition of After-TruffleTier nodes.
- **This admitted graph predates PERF038-H. It is NOT an A/B measurement for product current HEAD and cannot be directly compared with post-H product graphs to claim a speedup or regression magnitude.**

### Later historical capture, 2026-10-09, product 0db24f00

Product: **0db24f00ff2d92d642351d7f7535517fe01a55ce**, 0.3.324-SNAPSHOT.
Harness: **d9719253b628faad1ca296daef4a7ed566196216**, **dirty**.

[Protos inadmissible unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/unit.json)
[Retained capture](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/capture.json)

- Correctness PASS, **GRAPH_NOT_STABLE**, evidence_valid=false, budgets [1000,4000,16000,64000,256000].
- Failure gate is **UNIT_NOT_AT_FINAL_TIER**, not a known permanent PE bailout. Primary remains compiled Tier 1; additional child has a **Cutoff** edge and is also Tier 1 in the selected unit. Primary Tier-1 graph 2,349 nodes and child Tier-1 graph 116 are **not admissible final-tier node totals**.
- Zero reported primary post-final invalidations or failures. One overall invalidation appears per budget; **do not attribute that to a repeating primary-root deoptimization cycle without trace evidence**.
- Primary Tier-2 inlining/cutoff decision and compilation selection are not yet causally explained. Raising budgets, guessing a deoptimization cycle, or promoting Tier-1 counts is forbidden.

The earlier clean Tier-2 graph demonstrates feasibility; it does **not** establish that post-PERF038-H or present product HEAD already has a stable Tier-2 shape.

## Whole-path source attribution — confirmed versus unresolved

1. **Nested Closure creation / authority admission.** CanonicalToBytecodeLowerer.emitExpression(CanonicalClosure) evaluates CurrentActivation and MaterializeClosure. MaterializeClosure constructs ProtosClosureValue with captured lexical environment, receiver/home/prelude/return home and plan. CanonicalToBytecodeLowerer.demandsPersistentFrame returns true for any nested CanonicalClosure, so the enclosing root conservatively installs persistent lexical authority. The benchmark's inner identity reads its own parameter, but the existence of nested Closure and capture/observation contracts makes authority deferral a **hypothesis**, not an approved removal.
2. **Already optimized direct Call1.** ProtosSemanticBytecodeRootNode.PrepareClosureCallOne and ProtosBytecodeRootNode.finishDirectClosureCallOne use guarded/cache-admitted target selection. ProtosFrameArguments.compactDirectClosureCallOnePrepared writes supplied0 directly into the final Truffle frame array; no separate intermediate supplied array is made on that admitted path. A guest-visible arguments Array is already deferred in the activation model. Do not reopen PERF038-H's work as though Call1 were absent.
3. **Activation and lexical materialization.** The semantic root has no universal rich-activation prologue; CurrentActivation materializes on demand. The clean graph nevertheless retains seven materializeCompactActivation invokes and lexical authority operations. Without the BGV edges, these cannot be claimed hot or necessarily executed.
4. **Return/control/instrumentation.** OrdinarySourceCall is a lean leaf of PreparedClosureCall; direct/no-home entry, provablyNonSuspendingBody, continuation-specific specialization, and generic transfer/error fallback already exist. A source-reachable fallback alone does not prove it survives the optimized ordinary path. The large FrameState/If/guard footprint needs exact control/data edge inspection.
5. **Task/Actor/Process.** PLAT046 ordinary caller-thread and compact invocation architecture is already ratified; this evidence does not prove any new RootTask/Actor/Process allocation per canonical call. Do not invent that tax.
6. **Upstream comparison.** GraalJS 25.4.4.1.1 JSFunctionCallNode selects Call0/Call1/CallN and JSArguments creates a final frame array with 2 runtime slots. GraalPy CallDispatchers.FunctionCachedCallNode caches PFunction/calltarget under guards and PArguments has 6 runtime slots plus user values. Neither peer universally avoids arrays or all frame state. Their compact admitted trivial graphs argue for capability-aware specialization, not blind ABI imitation.

Exact current Protos paths:
- src/main/java/com/guillermomolina/protos/execution/{CanonicalToBytecodeLowerer,ProtosSemanticBytecodeRootNode,ProtosBytecodeRootNode,ProtosFrameArguments,ProtosBytecodeClosureExecutionPlan,ProtosInvocation,ProtosClosureInvoker}.java
- src/main/java/com/guillermomolina/protos/runtime/{ProtosActivation,ProtosClosureValue}.java
- Oracle JavaScript tagged JSFunctionCallNode.java and JSArguments.java.
- Oracle Python tagged CallNode.java, CallDispatchers.java and PArguments.java.
- Oracle Graal tagged truffle/docs/DeoptCyclePatterns.md: no deoptimization cycle demonstrated by this specific Tier-1-only capture.

## Ranked hypotheses (not approved implementation)

Source/graph confidence and risk follow the A1 audit:

1. Eager persistent lexical authority for nested Closure plus activation/closure materialization: **highest actionable source-and-graph lead**, but high capture/identity/tooling semantic risk.
2. Generic generated-bytecode exception, completion and frame-state alternatives: broad observed footprint but exact live edges unknown.
3. Remaining on-demand rich-activation transitions (seven historical invokes): exact graph sites known; which are hot/cold unknown.
4. Generic return/continuation and dispatch alternatives: already partially specialized by PERF038-H; avoid duplicate work.
5. Frame header residual representation: source-confirmed, compiled impact unproven.
6. Eager *guest argument-array* construction on admitted Call1: **not supported** as a current primary residual cause.

## Bounded next slice: PERF042-A2 (INVESTIGATION, no commands)

**Goal:** complete one coherent source-to-IR causal investigation, not several microscopic implementation slices.

- Read current product/harness HEAD and governing AGENTS/spec/ratifications; this document is a pinned historical starting point, not a substitute for current authority.
- Trace exact Tier-2 compiler admission and child Closure Cutoff for 0db24f00 versus d59da442. Read retained compilation traces and BGV/IGV edges if a read-only mechanism exposes them. Record actual compilation IDs, root identities, inlining reason and deopt/invalidation ownership. Do not infer a cycle without evidence.
- For the admitted d59da442 graph, map **all 44 surviving invoke targets**, dominant If/guard/FrameState groups, lexical-authority and activation paths to executable/cold code; distinguish success-path work from deopt/exception/tooling fallback and graph-only virtual allocations. Follow missing sources with bounded targeted reads.
- Prove whether nested identity Closure actually needs a persistent parent lexical authority or rich activation *at creation* under capture-by-reference, escape, semantic context/argument observation, return-home, debugger, reentrancy, and instrumentation semantics. No semantics change.
- Reconcile what PERF038-H already did in source HEAD, especially Call1 final-frame construction, guarded PIC, provably non-suspending body, return-home marker and direct/no-home entry. Separate C0 (zero args), C1 (positional), and C2 (actual argument-vector observation). Do not change the benchmark.
- Consult exact-tag GraalJS and GraalPy code for every hypothesized cost family; present a source/edge/equivalence matrix with confidence and risks. Recommend **one grouped** ordinary-call implementation only if causal evidence plus normative proof support it; otherwise STOP with precisely identified missing BGV/trace evidence, without guessing.

**Hard gate:** If compressed BGV/trace edges cannot be inspected read-only, report this as a blocker and formulate the smallest human-executed evidence request for a separate approved diagnostic step. Investigation agents execute **no shell commands, builds, tests, benchmarks, validation scripts, Git mutations, source edits, or publication**.

**Human executor role for A2:** No commands are needed for the investigation itself; supply evidence only if an explicit subsequent diagnostic handoff is justified and authorized. The existing maintainer-reported green tests are acknowledged but are not a Tier-2/latency acceptance.

**After A2 only:** If and only if cause and semantic admissibility are proven, prepare a single bounded PERF042 implementation slice in guillermomolina/protos (no automatic new Ixxx merely for an internal slice); if a substantive independent D/PLAT decision or separately tracked implementation ownership becomes necessary, follow AGENTS.md/GITHUB006/GITHUB015 formal issue governance and seek exact owner approval.

## STOP and publication boundaries

- Do not treat Tier-1 graphs as peer parity; do not use historical clean 4,533-node graph as post-H current-HEAD baseline.
- Do not claim runtime cost from BGV site count, nor an A/B speedup from cross-revision node totals.
- Do not weaken Closure capture-by-reference, identity, callable selection, arity/default/rest/spread, argument introspection, return/error precedence, Task/suspension, or instrumentation semantics.
- No modification to Protos source/spec, no new D/PLAT decision, no authorization to implement. PERF042 remains open and independent of PERF038.
- The next agent prepares evidence and conclusions; the human alone performs execution/validation/publication in the product repo if later authorized. The project-docs direct publication for this A1 snapshot is separately owner-authorized.

## Source references

- https://github.com/guillermomolina/protos/issues/871
- https://github.com/guillermomolina/protos/issues/852
- https://github.com/guillermomolina/protos-project-docs/blob/9548c2f94e0e7e0308522c1b747960441c7d0051/docs/project/evidence/PERF038/PERF038_H_PUBLISHED_IMPLEMENTATION_AND_RESIDUAL_GRAPH_CHECKPOINT_2026_10_09.md
- https://github.com/oracle/graaljs/blob/58f1795f7d12d12260d0c0423811e846c7632649/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java
- https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java
- https://github.com/oracle/graal/blob/95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b/truffle/docs/DeoptCyclePatterns.md
