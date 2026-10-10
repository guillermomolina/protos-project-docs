# PERF041-A2 — Causal topology of `primitive-if-true` After TruffleTier graph

**Status:** Investigation evidence published; **no implementation or new architectural decision approved by this record**.
**Owner:** [PERF041 / guillermomolina/protos#863](https://github.com/guillermomolina/protos/issues/863).
**Product revision:** [`0db24f00ff2d92d642351d7f7535517fe01a55ce`](https://github.com/guillermomolina/protos/commit/0db24f00ff2d92d642351d7f7535517fe01a55ce).
**Published benchmarks repository revision read during investigation:** [`527035f01107e6c71f860e95f438b7553a5428a2`](https://github.com/guillermomolina/protos-benchmarks/commit/527035f01107e6c71f860e95f438b7553a5428a2).
**Capture metadata's recorded `harness_git_head`:** `d9719253b628faad1ca296daef4a7ed566196216`; **`harness_dirty=true`**. The specific source hashes retained in the capture carry reproducibility identity; do not describe this as a clean harness checkout.
**Retained data:** `results/graphs-full-0db24f00ff2d/{primitive-if-true,primitive-if-false,primitive-return-literal}/protos/`, stable at budget 64,000 vs 16,000, compilation tier 2, one selected `StructuredGraph / After TruffleTier` per case; `correctness.result=PASS`, `evidence_valid=true` in the capture metadata.
**Additional data inspected:** independently supplied `PERF041_A2_topologia.tar.gz`, containing the three original `unit.json`, `capture.json`, `trace_log` and matching `*.filter.json.gz` export files. The compressed analysis-file bytes match the respective `analysis_sha256` in their `unit.json`. BGV originals are identified by published hashes but were not independently rehashed from the supplied archive.
**Execution:** read-only export/source analysis. No builds, benchmarks, Protos runtime runs, test execution or source modifications.

## 1. Graph identities and quantitative contrast

| `After TruffleTier` metric | `primitive-if-true` | `primitive-if-false` | `primitive-return-literal` | True minus false |
| --- | ---: | ---: | ---: | ---: |
| IR nodes | **5,193** | **892** | **39** | +4,301 |
| All IGV edge kinds | 14,125 | 2,373 | 56 | +11,752 |
| IGV blocks | 892 | 128 | 1 | +764 |
| `IfNode` | 298 | 38 | 0 | +260 |
| `FrameState` | 906 | 162 | 6 | +744 |
| `BeginNode` | 645 | 83 | 1 | +562 |
| `EndNode` | 473 | 61 | 0 | +412 |
| `FixedGuardNode` | 265 | 44 | 0 | +221 |
| `DeoptimizeNode` | 58 | 13 | 0 | +45 |
| `InvokeWithExceptionNode` | 47 | 9 | 0 | +38 |
| `AllocatedObjectNode` | 116 | 25 | 0 | +91 |
| `CommitAllocationNode` | 116 | 25 | 0 | +91 |
| `LoopBeginNode` | 6 | 3 | 0 | +3 |

The block-entry connectivity traversal reaches 891/892 blocks for true and 127/128 for false; one recorded block in each graph is not reachable from B0 under that traversal. Counts of IGV edges include value and state edges, **not just CFG branches**. Frame states and allocation-related IR nodes cannot be equated to per-iteration runtime or heap cost.

Selected compressed export SHA-256:

| Workload | `analysis_sha256` | `bgv_sha256` retained in metadata |
| --- | --- | --- |
| `primitive-if-true` | `abd7fd54f14a2f9ac37e5e5fd1af78bab94d65b54e5a9a0c5545dee3c9fcddb3` | `d4a68a1adf8f97d9095773935d9286816a40ae6c8a56305653b08daa358652c9` |
| `primitive-if-false` | `61b44cd47ff602c6226fc32f5109c4deb28200b0201f18f1a99ad327f2bf71be` | `276e0ffff411bae601ed58e9a47a888626e434cdb185e7da4b5d065123490613` |
| `primitive-return-literal` | `ec73935ed757df85bddab7811a88e684d8c1277b12a416d3eaf14ab6c2602c7d` | `b37dbf9771deb5c453c31441e232a8454e63f474a4fd37951c8ffcaaa0c782ce` |

The JS/GraalPy 13/49-node reference figures come from the earlier A1 benchmark evidence, **not** a symmetrical re-export of peer BGVs in this A2 archive; syntactic `if` is not semantically equivalent to an overrideable Protos ordinary message.

## 2. Mandatory entry-path activation tax (both Boolean cases)

The **entry block B0** of both Boolean graphs contains an `InvokeWithExceptionNode` targeting `ProtosFrameArguments.materializeCompactActivation`, with estimated `relativeFrequency=1.0` and normal `next` and `exceptionEdge` successors: node **630** in true; node **426** in false. This call is absent from the 39-node literal-root graph.

Source-position chains identify `ProtosSemanticBytecodeRootNode.CurrentActivation.materialize` and `ProtosFrameArguments.activation`. The product lowerer `CanonicalToBytecodeLowerer.emitExpression(CanonicalClosure)` emits `MaterializeClosure` with `emitCurrentActivation` **while evaluating the Closure literal as an argument**, before choosing whether the Boolean receiver will execute its callback.

Consequently, `false.ifTrue() { 99 }` still requests materialization of the **outer/root activation** to create the Closure literal, even though the callback is not called. The ordinary rule that the callback-producing expression is eagerly evaluated remains obligatory; eager **rich context materialization** does not follow as a general requirement if all identity, capture, escape, observer and tooling invariants are preserved.

Other `materializeCompactActivation` calls appear in a conditional `PrepareSendOne.exactCaller` path (true node 2335, false node 1445, relative-frequency estimate ~0.375); true also contains a callback-argument-authority path (node 16503, ~0.0294). These are **not** all mandatory B0 calls. The method's `@TruffleBoundary` does not mean its result cannot be requested on the steady source path.

## 3. Exclusive source-position cohort accounting

Each graph node is assigned **once**, by the first matching family in the priority order below, using the entire `nodeSourcePosition.Java` inline/source-position chain. This is a **disjoint provenance classification**, not a cost, dominance, liveness, heap-allocation or eliminability accounting.

| First-matching origin family | True | False | Difference |
| --- | ---: | ---: | ---: |
| `PrepareInlineStructuredBooleanCallbackCall` | **2,676** | 0 | **+2,676** |
| `CheckInlineClosureArgumentUpperBound` | 109 | 0 | +109 |
| `AdmitsInlineLiteralCallback` | 51 | 0 | +51 |
| `FinishStructuredBooleanCallback` | 66 | 0 | +66 |
| `guardedStructuredSend*` (outer send) | 304 | 304 | 0 |
| `PrepareStructuredBooleanCall` | 184 | 184 | 0 |
| `MaterializeClosure` (excluding earlier activation origin) | 97 | 95 | +2 |
| `CompleteClosureCall` | 35 | 20 | +15 |
| Other mapped origins | 769 | 91 | +678 |
| No `nodeSourcePosition` | **902** | **198** | **+704** |
| **Total** | **5,193** | **892** | **+4,301** |

The 2,676-node callback-preparation cohort spans **672** distinct blocks and includes 496 `BeginNode`, 273 `EndNode`, 231 `IfNode`, 215 `FrameState`, 190 `LoadFieldNode`, 153 `FixedGuardNode` and 85 `AllocatedObjectNode`. This cohort has **zero** matching nodes in false. Of 902 true nodes lacking Java source position, at least 504 are `FrameState`, 103 `EndNode`, 86 `VirtualObjectState`. These **cannot be uniquely attributed to a Protos producer**. The true−false difference of 4,301 is not entirely equivalent to actual callback execution costs.

## 4. Source-backed causal sequence

```text
CanonicalToBytecodeLowerer.emitComposedSend
  evaluate receiver and supplied Closure-literal expression once, in order
    emitExpression(CanonicalClosure)
      MaterializeClosure(emitCurrentActivation(), definition, plan)
  PrepareSendOne
    canonical guarded structured selection by receiver, selector, home/stability
    guardedStructuredSendPrepared
      ProtosActivation.forImmediateMethodInvocation
      attachTaskOrInheritDynamicControlState
      finishPreparingComposedCall -> generic prepared NativeCall
  IsStructuredBooleanCall
  PrepareStructuredBooleanCall -> PreparedBooleanCall
  StructuredBooleanHasCallback
  [only when selected, e.g. true.ifTrue]
    PrepareInlineStructuredBooleanCallbackCall
      PreparedBooleanCall.prepareInlineCallback
        prepareInlineLiteralCall
          selectClosureCall("call", caller)  // ordinary selection
          if canonical and eligible bytecode plan:
             compactDirectClosureCall / PreparedInlineLiteralCall.direct
          else:
             ordinary selected-call preparation/fallback
    AdmitsInlineLiteralCallback
    PLAT044 B-prime inline callback region OR exact ordinary fallback
    FinishStructuredBooleanCallback
  [otherwise] StructuredBooleanImmediateResult
  CompleteClosureCall / exact cleanup and propagation
```

At the source level, the guarded outer send is **already present**, but its stable hit still creates rich generic prepared-state before the B-prime region. Inside the callback preparation cohort, examples of deeply expanded leaf-method source positions include `Optional.ofNullable` (543), `StructuredCallCapabilities.of` (138), `ProtosClosureValue.executionPlanForRuntimeInvocation` (108), `finishPreparingComposedCall` (108), `Objects.requireNonNull` (82), `ProtosClosureValue.invocationReturnHomeForRuntime` (77) and `ProtosObjectValue.authorityRead` (70). These are *origin-chain observations*, not individually proven hot-path executions.

The exterior `guardedStructuredSend*` and `PrepareStructuredBooleanCall` cohorts are equal in true and false (304/184 nodes). The very large differential emerges downstream of callback selection; **fixing only the exterior dispatcher or only the inner callback does not meet the full goal**.

Source anchors at the pinned product revision:
- [`CanonicalToBytecodeLowerer.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java): `emitComposedSend`, `emitExpression`, `emitLocalBooleanInvocation`, `emitInlineLiteralCallback`.
- [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java): `guardedStructuredSendPrepared`, `PreparedBooleanCall`, `prepareInlineLiteralCall`, `PreparedInlineLiteralCall`.
- [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java): `CurrentActivation` and specialized send operations.
- [`ProtosFrameArguments.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java): compact activation materialization boundary.

## 5. Existing authority and evaluated approaches

The normative owners are `spec/semantics/VALUES_AND_COLLECTIONS.md` (standard Boolean behavior), `CALLABLES.md`, `OBJECT_MODEL.md`, `EXECUTION_AND_CONTROL.md`, and `spec/PROTOS_GRAMMAR.md` (trailing literal Closure is an ordinary value). The approved architecture is [PLAT040 F-prime](../../decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md), [PLAT044 B-prime](../../decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md), plus the PLAT043 semantic-root orchestration boundary.

| Candidate | Addresses | Remaining problem | Assessment |
| --- | --- | --- | --- |
| A. Keep generic pipeline | Conserves existing semantics | All 892/5,193-node structures | Fails structural objective |
| B. Slim `PreparedBooleanCall` | Part of control carrier | B0 activation, exterior prepared send, inner ordinary callback selection | Incomplete |
| C. Early guarded Boolean selection only | Generic exterior preparation | B0 literal capture and callback preparation can remain | Incomplete |
| **D\*. Joint guarded compact Boolean path** | Earlier exact selected-method guard, compact state, lazy literal/context, direct admitted B-prime callback and genuinely cold exact generic fallback | Must validate every semantic, instrumentation and deopt invariant | **Recommended conditionally**, as implementation technique under existing approved PLAT040/043/044 boundaries, not newly ratified architecture |

Observable invariants are non-negotiable: ordinary D013 lookup and exact invalidation, overridability/aliasing, eager once-only argument **expression evaluation**, exact receiver/arity/callback eligibility, canonical Boolean result/null, Closure value identity/capture, fresh semantic activation when observable, `this`/`context`/`methodHome`, `ReturnHome`, Task/Actor/domain, dynamic control, Error/exception cleanup, cancellation, suspension/continuations, multi-Context isolation, debugger/RootTag/scope/stepping and Native Image. Noneligible or invalidated sites take the original generic path; selector spelling and test-workload constants never grant special semantics. No broad `whileTrue`, numeric, or general Closure semantics refactor.

The selected follow-up should optimize all **three** evidenced taxes in a coherent integrated implementation **only if** a faithful and bounded patch is feasible. If an unratified externally observable change or genuinely new architecture is necessary, STOP and route it through an explicit owner decision. A graph reduction to GraalJS's 13 nodes is not a ratified acceptance threshold.

## 6. Falsification, acceptance and evidence limits

- **H1 outer rich preparation:** confirmed source path, 304-node outer provenance cohort, present in both true/false; per-call heap allocation is not established.
- **H2 generic inner admission/fallback expansion:** confirmed 2,676-node callback-preparation provenance cohort in true; graph presence is not proof of repeated hot-path execution.
- **H3 closure/activation tax:** confirmed `materializeCompactActivation` in B0 for both Boolean cases, including unselected callback. Literal identity/escape correctness governs any deferral.
- **H4 baseline/root:** 39-node literal-root graph, but this number is **not directly subtractable** from other cases as a fixed-cost bucket.
- **H5 selected/unselected asymmetry:** measured +4,301 IR nodes, exhaustively classified into **disjoint origin buckets**, with 902/198 nodes unmapped to Java source positions. Do not invent an individual producer for them.
- Exports establish After TruffleTier graph topology and source origins; they do **not** establish per-call timing, hotness, actual heap allocations or fully precise Java-method ownership of unmapped nodes. No claim of exhaustive all-stage peer-graph attribution is made.

Implementation validation, if separately authorized and undertaken, must first prove correctness (existing/focal conformance, override, alias, replaced slot, custom callable, expression evaluation, activation identity, debugger, error/return/suspend behavior), then recapture the **same three cases** on the same graph policy and compare node totals, cohorts, B0 invoke, If/FrameState/allocations, graph stability and true/false contrast. Stop on `NOT_STABLE` or correctness failure. Avoid claiming absolute JS parity or benchmark speedup without fresh stable results. The human executor—not the implementation agent—runs validation and benchmark capture.

## 7. Investigation STOP

```text
SLICE=PERF041-A2
INVESTIGATION_ONLY=YES
BGV_FILTER_EXPORTS=VERIFIED
SELECTED_GRAPH_PHASE=After TruffleTier
SELECTED_PROTOS_GRAPHS=3
TRUE_NODES=5193
FALSE_NODES=892
RETURN_LITERAL_NODES=39
TRUE_FALSE_NODE_DELTA=4301
SOURCE_POSITION_PARTITION=EXHAUSTIVE_NONOVERLAPPING
JAVA_PRODUCER_ATTRIBUTION=PARTIAL (902 true, 198 false unmapped)
ENTRY_ACTIVATION_MATERIALIZATION_IN_BOTH_BOOLEAN_CASES=YES
CALLBACK_PREPARATION_ORIGIN_BUCKET=2676
EARLY_BOOLEAN_DISPATCH_ALONE=INSUFFICIENT
NEXT_IMPLEMENTATION_CANDIDATE=D_STAR_CONDITIONAL
TESTS_RUN=NONE
BUILDS_RUN=NONE
BENCHMARKS_RERUN=NONE
PROTOS_REPOSITORY_CHANGES=NONE
DESIGN_RATIFICATION=NONE
```

Previous independently retained source-only checkpoint: [PERF041-A1](PERF041_A1_PRIMITIVE_IF_TRUE_CAUSAL_CHECKPOINT.md).
