# PERF041-A1 — primitive-if-true: source-path and retained-graph causal checkpoint

**Status:** PUBLISHED INVESTIGATION EVIDENCE; **not** an implementation authorization, design approval, exhaustive BGV topology audit, or issue closure.  
**Formal owner:** [PERF041 / guillermomolina/protos#863](https://github.com/guillermomolina/protos/issues/863)  
**Investigation scope:** `true.ifTrue() { 1 }`; `primitive-if-false` diagnostic control, `primitive-return-literal` root reference; `whileTrue` explicitly outside implementation scope.  
**Product revision examined:** [`0db24f00ff2d92d642351d7f7535517fe01a55ce`](https://github.com/guillermomolina/protos/commit/0db24f00ff2d92d642351d7f7535517fe01a55ce) (same HEAD as the retained capture when checked).  
**Benchmarks repository revision examined:** [`527035f01107e6c71f860e95f438b7553a5428a2`](https://github.com/guillermomolina/protos-benchmarks/commit/527035f01107e6c71f860e95f438b7553a5428a2).  
**Capture directory:** `guillermomolina/protos-benchmarks/results/graphs-full-0db24f00ff2d/`.  
**Selected evidence:** budget 64000; tier 2; `After TruffleTier`; `STABLE`; each focal Protos unit has one selected guest compilation graph. The harness metadata reports `harness_dirty=true` with tracked `harness_source_sha256` values; do **not** erase that provenance by claiming a clean harness tree.  
**Execution:** no commands, tests, builds, benchmark reruns, or product edits; retained public source and JSON/BGV-derived metadata inspected through GitHub.  
**Evidence caveat:** source file reads and retained `unit.json` analysis were available; decompressed `.bgv.gz` node/edge topology could **not** be retrieved through the accessible file-content interface. Accordingly, final-node producer-by-producer attribution remains OPEN.

## 1. Normative and ratified boundaries

- [Boolean semantics](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/spec/semantics/VALUES_AND_COLLECTIONS.md): ordinary overridable `ifTrue` message; exact canonical Boolean receiver for the standard selected implementation, fixed arity, eager evaluation of callback-producing expressions, only selected callback must be invokable and executes once with zero arguments; exact result or canonical null; no truthiness; propagate nonlocal transfer, Error, suspension and cancellation.
- [Trailing Closure grammar](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/spec/PROTOS_GRAMMAR.md): `true.ifTrue() {1}` appends an ordinary zero-parameter Closure value, not a privileged conditional AST statement.
- `spec/semantics/CALLABLES.md`, `OBJECT_MODEL.md`, `EXECUTION_AND_CONTROL.md`: ordinary lookup/selection, receiver and methodHome, callback callability, capture, context, semantic activation/ReturnHome, dynamic control and continuation behavior remain authoritative.
- [PLAT040 Candidate F-prime](../../decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md): stable selected-send guard/invalidation, compact invocation state, conditional rich materialization, exact fallback, allowed standard-protocol specialization **after** ordinary selection.
- [PLAT043/#763](https://github.com/guillermomolina/protos/issues/763): locally orchestrated standard Boolean control in the semantic Bytecode root, no structured helper root for admitted Boolean dispatch.
- [PLAT044 Candidate B-prime](../../decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md), [PERF026-B/#767](https://github.com/guillermomolina/protos/issues/767): admitted immediate literal callback executes in the source root under preserved semantic activation and custom RootTag, **without** a distinct physical callback RootCallTarget/FrameInstance; approved debugger/stack delta and exact generic fallback stand.
- No new Dxxx/PLATxxx decision selected in this investigation. The approved architectural bounds do **not** imply a particular new guard/deopt technique is already proven correct.

## 2. Exact source route and discriminators

`CanonicalToBytecodeLowerer.emitComposedSend` (approximately lines 6120–6380) stages receiver and candidate literal argument; `inlineLiteralCallbackCandidates` sees a zero-parameter Closure literal with no nested literal Closures. The send is **not** admitted by `TryDirectSendOne` because its condition requires `inlineCallbackPositions.isEmpty()`; deleting that check is **not** a justified fix, since `TryDirectSendOne` handles a source-Closure direct-send shape, whereas canonical Boolean `ifTrue` is native structured control.

`emitExpression(CanonicalClosure)` emits `MaterializeClosure` with creator activation, definition and `ProtosClosureExecutionPlanCell`. The operand is semantically an ordinary evaluated Closure; a compiler can avoid its physical object only when identity and all other observations remain equivalent.

`PrepareSendOne` has guarded standard-structured selection. `ProtosBytecodeRootNode.PrepareSendArguments.createGuardedStructuredSend` establishes canonical Boolean behavior via selection identity/home and a stability assumption, rather than selector spelling alone. **The selected guarded hit still calls** `guardedStructuredSendPrepared` (approximately lines 9277–9300), which invokes `ProtosActivation.forImmediateMethodInvocation`, dynamic-control inheritance, and `finishPreparingComposedCall` to form a generic `NativeCall`. This is a source-path fact; physical allocations per invocation are not yet established.

`emitPreparedInvocation` (approximately lines 4716–4820) branches on `IsStructuredBooleanCall` and retains generic dispatched preparation as the alternative. `emitLocalBooleanInvocation` (approximately lines 5263–5345) emits `CompleteClosureCall` in a finally region, `PrepareStructuredBooleanCall`, `StructuredBooleanHasCallback`, `PrepareInlineStructuredBooleanCallbackCall`, `AdmitsInlineLiteralCallback`, inline region/ordinary child fallback, and final result selection. `PreparedBooleanCall` (approximately lines 4075–4210) wraps kind, receiver, supplied list and activation. `prepareInlineLiteralCall` (approximately lines 8411–8440) performs ordinary selected `call` lookup and, for a matching canonical source Closure, produces `PreparedInlineLiteralCall` with compact frame arguments; otherwise creates the ordinary selected-call fallback.

**Already solved, not a new issue:** `ProtosPerf025InlineCallbackPreparationTest.booleanLiteralAdmissionPrecedesActivationAndPhysicalFallback` checks light preparation/admission occurs before rich callback activation and physical fallback. `emitInlineLiteralCallback` (approximately lines 5383–5530) contains the existing inline semantic region and state/tooling projection. It does not prove the *outer Boolean send* avoids generic prepared state.

## 3. Retained numeric evidence

All counts below are `After TruffleTier` **IR node counts**, not machine instructions, time/call, or allocations/call.

| Workload | Protos | GraalJS | GraalPy | Protos IfNode | Protos invokes |
| --- | ---: | ---: | ---: | ---: | ---: |
| primitive-return-literal | 39 | 13 | 49 | 0 | 0 |
| primitive-if-true | **5,193** | 13 | 49 | 298 | 47 |
| primitive-if-false | **892** | 13 | 49 | 38 | 9 |

`primitive-if-true`: 906 `FrameState`, 645 `BeginNode`, 473 `EndNode`, 298 `IfNode`, 292 `LoadFieldNode`, 265 `FixedGuardNode`, 236 allocation-related nodes (retained metric), and 47 invokes. Retained invoke targets include `Throwable.fillInStackTrace` (7), `ProtosLanguageContext.currentIfEnteredForRuntime` (6), `ProtosLexicalBindingAuthorityCalls.read` (6), `ProtosFrameArguments.materializeCompactActivation` (3), `ProtosActivation.inheritDynamicControlState` (3), and `ProtosInlineCallbackFrameBindings.transferFrameBindingsToDurableActivation` (1).

`primitive-if-false`: 162 `FrameState`, 38 `IfNode`, 44 `FixedGuardNode`, 52 allocation-related nodes, 9 invokes. The selected callback body is not reached, but the argument expression still evaluates under the semantics. `primitive-if-true` exceeds `primitive-if-false` by 4,301 graph nodes: **do not** attribute that difference entirely to executed callback code.

`primitive-return-literal`: 39 nodes, 6 `FrameState`, zero retained invokes and zero `IfNode`; this is a baseline control, not an exact subtractable cost center.

Published trace expansion `self.count` entries for `primitive-if-true` include `CachedBytecodeNode` (3,518), `PrepareStructuredBooleanCall_Node` (154), `CheckInlineClosureArgumentUpperBound_Node` (106), `PrepareSendOne_Node` (30), and `CompleteClosureCall_Node` (25), with separately reported additional entries. **These do not form an additive final-IR node-producer accounting.** `primitive-if-false` reports `PrepareSendOne_Node` 304 and `PrepareStructuredBooleanCall_Node` 183 in the expansion ledger. No value here proves per-call physical allocations or hot-path execution of retained invokes.

Canonical evidence: [if-true `unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-if-true/protos/unit.json), [if-false `unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-if-false/protos/unit.json), [return `unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/527035f01107e6c71f860e95f438b7553a5428a2/results/graphs-full-0db24f00ff2d/primitive-return-literal/protos/unit.json).

## 4. Peer implementations and semantic differences

- [GraalJS `IfNode.java`](https://github.com/oracle/graaljs/blob/4c9cd0a6d1d5270b6cd039817851756b0b01b254/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/IfNode.java): explicit local condition profile with `thenPart.execute(frame)`/`elsePart.execute(frame)`; no dynamic `ifTrue` method selection. Its compiled `if(true)` has no `IfNode`.
- [GraalPy `RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java): `visit(StmtTy.If)` emits `beginIfThen`/`beginIfThenElse` directly in Bytecode DSL. Python has no Protos Boolean callback/message semantics.
- [TruffleSqueak `InterpreterSistaV1Node.java`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/interpreter/InterpreterSistaV1Node.java): `EXT_JUMP_IF_TRUE` / `EXT_JUMP_IF_FALSE` are physical conditional jump handlers. This does **not**, by itself, prove every Smalltalk `ifTrue:` send is always open-coded by its image compiler.
- The key transferable principle is **semantic message lookup plus guarded physical control open-coding**, not syntax equivalence or a mandatory 13-node target.

## 5. Falsification status

| Hypothesis | Assessment | What remains open |
| --- | --- | --- |
| H1: generic outer prepared state precedes admitted B-prime region | PARTIALLY CONFIRMED by source | precise physical allocation, final-graph node producers |
| H2: generic admission/dispatch/fallback IR survives | PARTIALLY CONFIRMED by source and retained targets | BGV hot/cold edges, escaping/deopt states and final-node counts |
| H3: literal materialization/provenance is an additional tax | PARTIALLY CONFIRMED by representation, not per-call allocation | exact Closure object virtuality and plan-cell producer attribution |
| H4: unrelated root/frame baseline | PARTIALLY CONFIRMED; 39-node reference observed | irreducible common nodes within Boolean graph |
| H5: selected/unselected graph asymmetry | CONFIRMED (5,193 vs 892) | quantitative assignment of 4,301-node difference |

The strongest supported source-level diagnosis is **late physical specialization**: canonical standard Boolean selection exists, but its valid hit still assembles the generic outer `NativeCall` / `ProtosActivation` and creates a `PreparedBooleanCall` before using the admitted inline callback. Eliminating this physical route under valid canonical guards, without changing semantic lookup or B-prime tooling, is a plausible follow-up, **not a proven graph-size win**.

## 6. Candidate assessment and pending completion gate

A. Keep generic preparation and trust Graal: semantically conservative, not effective in the published compiled graph.  
B. Locally slim `PreparedBooleanCall`: bounded but likely only partial, because it retains the exterior prepared-send pipeline.  
C. Add a guarded Boolean fast path with a full ordinary fallback in the same emitted Bytecode: may still retain the cold branch's IR.  
D. Move canonical guarded selection ahead of generic exterior preparation; use compact semantic state and a sufficiently cold exact fallback/deopt boundary, reusing B-prime. Best **architectural hypothesis** for the observed problem, not ratified code or a proven quantitative attribution.

**Unresolved required BGV gate:** analyze retained BGV topology, source/expansion edges and subsequent phase transitions; separate warm hot-hit IR from deopt/fallback/exception/tooling; attribute per-producer final nodes without adding expansion figures or assuming every invoke executes; state where BGV source mapping is missing. Avoid new measurements when already published data suffice. If binary graph tools are unavailable under the strict research-only/no-command workflow, report the exact access limitation instead of claiming completion.

**Recommended next slice within the existing INVESTIGATION ONLY Issue:** `PERF041-A2` — complete published BGV causal attribution and reconcile the guarded early-selection plan against all normative and PLAT040/043/044 invariants. STOP for owner review with an implementation-ready bounded candidate **only after** the required attribution; if a new architecture choice appears, route it through a proper PLATxxx approval checkpoint. No new Issue for a merely internal investigation slice. An implementation request would require its own authorization and formal scope; this evidence does not confer it.

## 7. Audit trail and non-claims

Materially inspected: product `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`; the normative Boolean, callable, object, execution and grammar owners; `CanonicalToBytecodeLowerer`, `ProtosSemanticBytecodeRootNode`, `ProtosBytecodeRootNode`, `ProtosStandardBooleanProtocol`, `ProtosPerf025InlineCallbackPreparationTest`; PLAT040/PLAT044 ratification records and PERF026 records; benchmark workload source and `unit.json` for return/if-true/if-false; peer source files linked above. This investigation did not execute or independently validate the tests. A metadata source path or graph expansion entry is **not** direct proof of its final IR node producer or live call frequency.

```text
INVESTIGATION_ONLY=YES
EXECUTION=NONE
PRODUCT_REPOSITORY_CHANGES=NONE
DESIGN_RATIFICATION=NONE
BENCHMARK_RECAPTURE=NONE
BGV_EDGE_ATTRIBUTION_COMPLETE=NO
IMPLEMENTATION_AUTHORIZED=NO
```
