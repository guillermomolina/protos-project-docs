# PERF042-A3 — Complete direct Closure Call1 causal investigation and retained-evidence gate

Date: 2026-10-10
Formal owner: [PERF042 / #871](https://github.com/guillermomolina/protos/issues/871)
Type: **INVESTIGATION**; no product implementation, no new semantic/architecture approval.
Status: **A3 source/comparison investigation concluded; exact Tier-2 compiler cause and incoming IR control/data edges BLOCKED on compressed retained artifacts; PERF042 OPEN.**

## 1. Scope and authority

This evidence reports the consolidated final research handoff requested by [A2](PERF042_A2_CURRENT_HEAD_INVOCATION_ATTRIBUTION_2026_10_10.md), retaining [A1](PERF042_A1_SOURCE_AND_GRAPH_INVESTIGATION_2026_10_10.md) as a versioned historical checkpoint.

Pinned Protos source reviewed: `7b609c4ad0e146d6f02a7a3d036c6ff938ccad08` (I092). Benchmark/evidence tree reviewed: `527035f01107e6c71f860e95f438b7553a5428a2`. At this documentation-publication reconciliation the benchmark `main` advanced to `8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`, adding I092 graphs for `primitive-if-false`, `primitive-if-true`, and `primitive-return-literal` only. It did **not** alter the historical `primitive-closure-call` artifacts examined here. Project-docs publication parent at preparation: `5e6ef3a0c816efc306cb2efb92a4deada979fc6a`.

Relevant policy: Protos `AGENTS.md`; `AGENTS.work/PERFORMANCE.md`, `COORDINATION.md` and `IMPLEMENTATION.md`; docs `AGENTS.md` and ratified DOC002 path contract. Normative semantic owners: `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`. Ratified architecture reviewed: PLAT040 guarded compact invocation; PLAT042 structured-call root ownership; PLAT044 narrowly admitted literal callbacks, **not** general Closure call inlining; PLAT046 ordinary caller-thread/pay-as-you-grow invocation.

Pinned comparison sources: GraalVM CE `25.4.4.1.1` / Java `25.0.4.1.1`; GraalJS `58f1795f7d12d12260d0c0423811e846c7632649`, GraalPy `89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3`, Graal `95ce1499c8c96ab7d5a6697c5b4bf42160f3b68b`.

Owner-reported local `git diff --check` CLEAN and tests PASS belong to the earlier A1/A2 handoff, without test-run SHA/count. These are not test commands run by this investigator and are **not** current-HEAD Tier-2 evidence. No product tests, builds, benchmarks, scripts, benchmark-capture changes, or repository mutation occurred during the A3 research itself. The later bounded documentation publication and Issue reconciliation are separate coordination actions.

## 2. Canonical unmodified workload

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

Expected result: `1`. This is a fresh, nested, local one-positional-argument Closure **C1**, not C0, not a method send, and not C2 actual argument Array introspection. Workload source and same-policy JS/Py equivalents are at `protos-benchmarks/truffle/workloads/primitive-closure-call/`, retained at exact benchmark revision above. The canonical workload must remain unchanged.

## 3. Historical graph admissibility — never compare dissimilar tiers

| Capture | Product | Harness | Correctness | Eligibility | Protos graph |
| --- | --- | --- | --- | --- | --- |
| Clean, 2026-10-08 | `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` | `98abc9af7a05a45a7d4056b72f36aef5889cb76f`, clean | PASS | Stable [16000,64000], final **Tier 2**, nested Closure inlined | **4,533 valid historical nodes** |
| Later, 2026-10-09 | `0db24f00ff2d92d642351d7f7535517fe01a55ce` | `d9719253b628faad1ca296daef4a7ed566196216`, dirty | PASS | **GRAPH_NOT_STABLE / UNIT_NOT_AT_FINAL_TIER** even at 256000; selected primary and Cutoff child both Tier 1 | **2,349 primary + 116 child, invalid as final-tier totals** |

The clean historical reference has GraalJS **13** and GraalPy **50** selected nodes on their equivalent same-policy cases. It **predates PERF038-H**, so it is neither a present-product baseline nor a controlled A/B for `7b609c4a`. The later dirty capture is not eligible for final-tier comparison.

Exact retained JSON:
- [Clean unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/protos/unit.json) and [capture](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/global-20261008-graphs/primitive-closure-call/protos/capture.json).
- [Later unit](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/unit.json) and [capture](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/capture.json).

Clean compilation counts: budgets 1000/4000: 3 and Tier 1; 16000/64000: 6 and Tier 2 with child inlined. Later: 1000/4000: 4 and Tier 1; 16000/64000/256000: 5 and Tier 1. **Both captures initially have a Tier-1 child Cutoff**. Cutoff alone cannot explain the later nonadmission. No compiler failure is reported; an overall later invalidation is not attributable to a primary-root recurring deopt cycle without exact trace ownership. Graal's `DeoptCyclePatterns.md` documents possibilities, not proof that one occurred.

## 4. Whole-path source attribution

The minimum path must be evaluated as **Closure construction + capture + local binding/lookup + ordinary `call` selection + cached/fused dispatch + target/inlining + C1 ABI + parameter bind/read + activation/authority + return/exception/continuation + tooling/deopt recovery**.

### H1 — source-proven cache mismatch and bounded candidate

In `ProtosSemanticBytecodeRootNode.TryDirectCallOne.guardedSource`, the PIC guards `receiver == cachedReceiver`. The canonical `identity` Closure is freshly constructed on each `run`; its Java receiver identity changes, while its `CanonicalClosure` definition is stable. A receiver-identity cache cannot converge across these new instances; `TryDirectCallOne.generic` returns MISS into the prepared path.

Protos already has `PrepareClosureCallOne.fastDirect` keyed on `closureDefinition == cachedClosureDefinition`, with current-instance `directClosureCallSelectionOrNull(receiver, caller)`. This reselects ordinary `Object.call` via `ProtosValueLookup.lookup` and rejects noncanonical behavior. Thus the proposal is **not** to invent a new function cache but to combine the existing definition-keyed selection with the already-existing fused `TryDirectCallOne` and `DirectCallNode`. Preserve the **current** Closure, capture, receiver, home and call selection; never substitute a cached Closure instance. Source-level cause: high confidence. Quantified Tier-2 IR effect: unknown.

### H2 — persistent lexical authority, I092 boundary

`CanonicalToBytecodeLowerer.demandsPersistentFrame(CanonicalClosure)` returns true; the enclosing `run` declares the frame-owned `identity` binding, so lowering installs `InstallFrameLexicalAuthority`, backed by `frame.materialize()`, on root entry. `beginClosureCapture` uses `MaterializeClosure` + `CurrentActivation` when a root owns locals. I092's `MaterializeCurrentClosure` / `ProtosLexicalEnvironment.deferredCompactOwner` fast path applies only where `currentActivationLocal == null && currentRootFrameLocals.isEmpty()`; this canonical root does **not** qualify. Eager authority is source-proven; permission to remove it is **not** established. Correct late observation of a single Context identity, frame-backed local mutation, escape, `PRESENT/ABSENT`, `null`, open/closed/frozen and deoptimization must be maintained without a second binding store.

### H3 — caller asymmetry, only conditionally actionable

`emitInvocationCallerOperand` uses `CurrentActivation` for ordinary composed calls; eligible root-level sends use `emitSendCallerOperand` / `CurrentCallerReference` to defer activation and sometimes inherit provenance. However fused C1's `TryDirectCallOne` currently calls `PrepareSendArguments.exactCaller(caller)` before constructing the final frame. Changing the caller operand alone is therefore not a proved physical saving; a compatible caller ABI and source/callee observer proof would also be needed. Keep this inside a single grouped change only if demonstrably safe.

### Other stages, existing optimizations

Existing and **not** to repeat: `PrepareClosureCallOne`, `finishDirectClosureCallOne`, `compactDirectClosureCallOnePrepared`, `compactDirectClosureCallOneNoHome`, direct placement of `supplied0` in the **final frame array**, deferred rich activation and guest supplied Array, no-home and nonsuspending paths, frame-native parameter bind/read, existing error/exception fallback and tooling projection.

No eager *guest* argument Array is demonstrated on the admitted C1 path; final JVM frame `Object[]` is still real. C0 / C1 / C2 have different legitimate physical costs. No universal RootTask/Actor/Process execution tax is demonstrated for this trivial call.

The source producers of broad residual families include `CachedBytecodeNode`, `PrepareClosureCallOne`, `InstallFrameLexicalAuthority`, `CurrentActivation`, lexical authority transitions, error-handler crossing and `FinishClosureCall`. Which branches survive as executed success-path edges is **unresolved**.

### Retained clean Tier-2 IR census

`FrameState` **997**; `IfNode` **285**; `InstanceOfNode` **201**; `FixedGuardNode` **165**; `DeoptimizeNode` **45**; `InvokeWithExceptionNode` **44**; `AllocatedObjectNode` **43**. These are graph-node counts, not operation execution, physical heap allocation rates, or independently removable quantities.

Full **44 invoke-site** target census: `ProtosObjectValue.invalidateTrackedLookupDependencies` 8; `ProtosFrameArguments.materializeCompactActivation` 7; `ProtosLexicalBindingAuthorityCalls.put` 6; `selectGuestHandlerOnRootCrossing` 5; `ProtosLexicalBindingAuthorityCalls.contains` 4; `ProtosLexicalBindingAuthorityCalls.transferAll` 4; `List.size` 2; `ReadRootFrameLocal.slowRead` 2; `List.get` 1; `adoptPresentFrameBackedBindingsSlow` 1; `createFrameBackedBindingAt` 1; `ProtosLexicalBindingAuthorityCalls.read` 1; `BytecodeRootNode.getBytecodeNode` 1; `ProtosValueLookup.representedDelegationParent` 1. None is proven executed on the ordinary success path merely from its presence.

## 5. Exact pinned peer pattern

- [GraalJS `JSFunctionCallNode`](https://github.com/oracle/graaljs/blob/58f1795f7d12d12260d0c0423811e846c7632649/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/JSFunctionCallNode.java): distinct `Call0Node`, `Call1Node`, `CallNNode`; caching on reusable `JSFunctionData` when appropriate despite fresh function instances; `DirectCallNode`.
- [GraalJS `JSArguments`](https://github.com/oracle/graaljs/blob/58f1795f7d12d12260d0c0423811e846c7632649/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSArguments.java): two runtime header slots and a final single-argument array.
- [GraalPy `CallDispatchers`](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java): identity-keyed and code/root-keyed cached direct dispatch, guarded by code stability and call-target equality.
- [GraalPy `PArguments`](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/function/PArguments.java) and [`CreateArgumentsNode`](https://github.com/oracle/graalpython/blob/89a3b1f7fc0c0fa87eeeb26be22f68ad7c3d39c3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/argument/CreateArgumentsNode.java): six header slots and signature-correct arguments; neither larger headers nor smaller graph totals prove a Protos hot-path cause.

Reusable physical principle: cache stable definition/code and target, retain the **current** callable instance and all language-specific observable selection. JS/Py do **not** authorize bypassing Protos `Object.call` overrides, lexical capture or return semantics.

## 6. Ranked hypotheses (0–5, higher semantic-risk score means riskier)

| Hypothesis | Source | Graph | Causal confidence | Potential structural impact | Semantic risk |
| --- | ---: | ---: | ---: | ---: | ---: |
| H1 identity-keyed fused C1 misses fresh instances | 5 | 2 | 4 | 4 | 3 |
| H2 eager persistent lexical authority for nested Closure | 5 | 3 | 4 | 5 | 5 |
| H3 exact caller activation versus caller reference | 5 | 2 | 3 | 3 | 4 |
| H4 lexical generic fallbacks/transitions | 5 | 5 | 2 | 4 | 4 |
| H5 errors, guards, deopt/frame-state recovery | 5 | 5 | 2 | 4 | 4 |
| H6 compact-header width | 5 | 1 | 1 | 2 | 3 |
| Eager guest argument Array on admitted C1 | 0 | 0 | 0 | 0 | 2 |
| Universal RootTask in trivial call | 0 | 0 | 0 | 0 | 4 |

No predicted percentage, removable-node total, speedup, current-HEAD graph size or full Tier-2 causal diagnosis is admissible.

## 7. Semantic/non-regression and authorization boundary

Preserve: fresh Closure identity; ordinary overridable `call` selection and dynamic shadowing with invalidation; genuine reference-captured lexical Context identity and binding store; structural OPEN/CLOSED/FROZEN and `PRESENT/ABSENT` rules; receiver and positional/default/rest/spread evaluation order and error precedence; fresh activation, nonlocal returns and ReturnHome; recursion/reentrancy; Task/Actor/Process/continuation and errors when actually needed; debugger/instrumentation and deoptimization recovery. PLAT044's narrow eligible immediate callback optimization is **not** a general Closure/root-elision license.

**Conditional coherent implementation candidate** (not approved now): combine the definition-keyed revalidated ordinary `Object.call` PIC with the existing fused `TryDirectCallOne`, preserving its `DirectCallNode`, final C1 frame and generic fallbacks. Primary files: `ProtosSemanticBytecodeRootNode.java` and possibly `ProtosBytecodeRootNode.java`. Touch `CanonicalToBytecodeLowerer.java` or caller/frame collaborators only if independently proved H3 improvements fit the **same** coherent physical-call change. **Do not** bundle speculative H2 deferred persistent-authority rewrite or introduce a new physical root architecture. Implementation inside PERF042 does not automatically justify a new Ixxx; create an independent formal Issue only if coordination's independently closable granularity genuinely applies.

## 8. Precise evidence blocker and smallest separately authorized human diagnostic

Available GitHub text reads expose historical `capture.json` / `unit.json` tiers, compile IDs, counts, selected roots and source-expansion metadata. They **do not** decode retained compressed binary `trace.log.gz`, `*.filter.json.gz` or BGV edge payloads through the read-only research connection. Consequently **Tier-2 nonadmission reason and success-versus-cold incoming control/data edges are unknown**.

Needed existing artifacts (no new capture):
1. `results/global-20261008-graphs/primitive-closure-call/protos/budget-16000/trace.log.gz` for successful Tier-2 and earlier Cutoff chronology;
2. `results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/budget-256000/trace.log.gz` for missing Tier-2 decision/invalidation ownership;
3. `results/global-20261008-graphs/primitive-closure-call/protos/budget-64000/bgv/TruffleHotSpotCompilation-2495[ProtosSemanticBytecodeRootNodeGen@5897a84c].filter.json.gz` and corresponding retained `*.bgv.gz` for clean Tier-2 source-to-IR edges;
4. existing selected primary+child later filtered graphs/BGV at `results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/budget-256000/bgv/`.

Smallest **human-executed and separately authorized** diagnostic: decode existing compressed artifacts into narrow textual tables of exact compilation ID, root, tier, inline/Cutoff reason, invalidation/deopt owner, and the incoming control/data graph edges for all 44 invoke sites plus dominant guards, `FrameState`, checks and allocations. Record `UNAVAILABLE` for a field absent from the retained data; do not manufacture a compiler cause. Do not rerun benchmarks, increase budgets, change workloads, run suites or invent a new harness. The research agent may only inspect/read the resulting text; it must not execute commands.

If compiler decisions/edges remain inaccessible, stop with that factual blocker. No repeated automatic A4/A5, no spurious `GRAPH_STABLE`, no implementation approval inferred from this report.

## 9. Disposition and handoff

```text
IDENTIFIER=PERF042
SLICE=PERF042-A3
TYPE=INVESTIGATION
SOURCE_INVESTIGATION=COMPLETE_TO_AVAILABLE_EVIDENCE
CURRENT_PRODUCT_HEAD=7b609c4ad0e146d6f02a7a3d036c6ff938ccad08
BENCHMARK_EVIDENCE_REVISION=527035f01107e6c71f860e95f438b7553a5428a2
BENCHMARK_MAIN_AT_PUBLICATION=8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1
TIER2_NONADMISSION_CAUSE=BLOCKED_ON_RETAINED_COMPRESSED_TRACES
SOURCE_TO_LIVE_IR_EDGES=PARTIAL_BLOCKED
IMPLEMENTATION_AUTHORIZED=NO
NEW_FORMAL_ISSUE=NO
NEXT_ACTION=SEPARATELY_AUTHORIZED_HUMAN_RETAINED_EVIDENCE_EXTRACTION
PRODUCT_TESTS_OR_BENCHMARKS_EXECUTED=NO
ISSUE_STATE=OPEN_STATUS_READY_PRIORITY_UNSET
```

This is an immutable investigation snapshot, not a new normative decision, release, production change, or closure of PERF042.
