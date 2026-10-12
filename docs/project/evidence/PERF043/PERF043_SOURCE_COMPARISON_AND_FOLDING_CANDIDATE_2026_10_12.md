# PERF043 — Source comparison and guarded constant-folding candidate

**Date:** 2026-10-12  
**Owning issue:** [PERF043/#877](https://github.com/guillermomolina/protos/issues/877)  
**Product revision inspected:** [`18f71b2a664c56ac808e2f508b6369984de02280`](https://github.com/guillermomolina/protos/commit/18f71b2a664c56ac808e2f508b6369984de02280), `0.3.342-SNAPSHOT`  
**Benchmark-harness revision cited by issue:** `1725cd030d6a82167c49124d434944b7b364cd9b`  
**Status:** source-level research checkpoint; no PERF043 product implementation published, no PERF043 benchmark gain established.

This record is non-normative. It documents an optimization hypothesis and the boundary of current evidence, not an approved semantic change or proof of a compiler bug.

## Motivating workload and retained observations

```protos
run: () => {
    left: 1
    right: 2
    left + right
}
```

The [owning issue](https://github.com/guillermomolina/protos/issues/877) records a human-supplied `After TruffleTier` Protos graph at product `18f71b2a`: `2,771` total nodes, `IntegerAddExactNode=1`, `IntegerAddExactOverflowNode=1`, `FrameState=582`, `FixedGuardNode=145`, `IfNode=164`. The graph's selected root was `ProtosSemanticBytecodeRootNodeGen` with `src=n/a`. The matching source SHA is `ae8ba162beb092793920a40655bc10d1ff57b4dc62edbae370ad86c0d1e97aff`. Historical October 8 peers have GraalJS `14` and GraalPy `49` nodes without surviving arithmetic nodes. Historical peers and the current product are **not a same-revision controlled A/B comparison**. Node presence does not establish executed hot-path cost.

The issue records an accepted reference timing `p50=463.094 ns/call` (`CORRECTNESS=PASS`, `STEADY_STATE_ADMISSION=PASS`, `MEASUREMENT_VALID=YES`). This is a reference, not evidence of a PERF043 improvement.

Full `unit.json`, `capture.json`, BGV phase sequence and root trace for the latest graph could not be re-read at their referenced repository paths during this source comparison. **Root identity is NOT_ESTABLISHED independently; the first IR point where constant information is lost is NOT_ESTABLISHED.**

## Verified Protos implementation observations

At the inspected product revision:

1. [`CanonicalToBytecodeLowerer.java`, `scalarLocalNamesForRoot`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java#L1163-L1228) accepts limited straight-line create/read/assign/literal forms and returns an empty admitted-local set for other expressions. The workload's final `CanonicalSend` is one such excluded form. This is a **proven scalar-admission limitation**, not yet proven to be the causal reason why `IntegerAddExactNode` survives compilation.
2. [`emitBodyCreate`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java#L3707-L3770), [`emitScalarCreate`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java#L2350-L2402) and [`emitLookup`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java#L6019-L6070) distinguish compact scalar, frame-backed, and activation-aware paths. The specialized [`ReadRootFrameLocal`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java#L1000-L1072) retains correct materialized-context fallback.
3. [`emitComposedSend`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java#L6770-L6954) stages receiver and argument once, offers primitive `Long`/numeric carriers, emits `TryPrimitiveNumericSendOne`, and materializes carriers on the miss path.
4. [`TryPrimitiveNumericSendOne`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java#L6557-L6620) retains selector, entered Context, Prelude, exact canonical-selection capability, and `cachedInteger.stability()` guards. [`ProtosValueLookup.lookupGuardedInteger`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java#L393-L431) proves family-independent lookup and records invalidation. Those checks cannot be removed just because operands have literal origins.
5. [`ProtosPrimitiveNumericSendFastPathsBase.IntegerArithmeticNode`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/main/java/com/guillermomolina/protos/execution/ProtosPrimitiveNumericSendFastPathsBase.java#L52-L116) uses `Math.addExact` and an overflow-generalized exact Integer path. The arithmetic itself closely resembles GraalPy's primitive numeric specializations.
6. [`ProtosPerf025CompactCalleeExecutionTest`](https://github.com/guillermomolina/protos/blob/18f71b2a664c56ac808e2f508b6369984de02280/src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java#L189-L366) already covers scalar lane vs materialized activation. Emitted fallback bytecodes can remain even when the compact lane is fast; an emitted slow operation does not prove its execution in optimized machine code.

## Source-level upstream comparison

- [GraalPy `AstOptimizer.visit(ExprTy.BinOp)`](https://github.com/oracle/graalpython/blob/63022137a797dc10251acac5460bb2e5b0a113cb/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/AstOptimizer.java#L132-L155) folds an AST binary operator if *both children are directly constant AST nodes*, using `foldBinOp` / numeric dispatch to obtain its constant. This is **not evidence** of name-to-constant propagation for locals.
- [GraalPy `IntBuiltins.AddNode`](https://github.com/oracle/graalpython/blob/63022137a797dc10251acac5460bb2e5b0a113cb/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/ints/IntBuiltins.java#L490-L516) provides `doII`, `doLL`, and `doLLOvf` exact integer additions with exceptional large-integer handling.
- [GraalJS `JSAddNode.create`](https://github.com/oracle/graaljs/blob/3c038b6edd15f143f9888e1644ed15e10c57de4c/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddNode.java#L86-L104) computes two directly constant integer child nodes at AST-node construction when `UseSuperOperations` applies. It also creates left/right constant-operand specialized nodes.
- [GraalJS `JSAddConstantRightNumberNode`](https://github.com/oracle/graaljs/blob/3c038b6edd15f143f9888e1644ed15e10c57de4c/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddConstantRightNumberNode.java) is a useful specialization model, **not** a semantic license to replace Protos' D013 sends.

Python and JavaScript's own arithmetic contracts differ from Protos' ordinary overridable `+`. Neither peer's early constant fold may be transplanted as an unconditional source-level rewrite. No exact phase-by-phase peer causal trace was newly captured here.

## Candidate and next gate

**Candidate:** propagate literal-domain constants through proven stable local values and attach a precomputed **candidate result** to a canonical numeric-send lowering, while leaving guest dispatch governed by its existing exact selected-Closure, method-home, Prelude, entered Context and `Assumption` proof. A valid canonical branch may return the precomputed exact result; every admission miss preserves evaluation order and the unchanged D013 selection, Error, context-materialization and interop behavior.

The recognition/constant-evaluation stage and semantic admission are distinct: a source-level `1 + 2` may be *recognized early*; it cannot be replaced with bare `3` before canonical selection is justified. Do not add a duplicate arithmetic policy or retain a new cache across invalidation without proof. Preserve signed-64 normalization and large Integer overflow.

**Preferred bounded next implementation slice (owner-directed, not yet published):**
- First inspect the current HEAD and retained graph/root evidence and re-check the scalar-admission hypothesis. If it is disproven, do not blindly implement it.
- Implement an early, conservative constant-expression recognition/provenance path plus a guarded constant-result path for directly literal `1 + 2` **only if** it can reuse existing D013 admission correctly, and extend eligible straight-line local operand propagation without discarding compact-versus-observed semantics. Group changes when the same few files and regressions cover both; split when doing so would otherwise expand the surface without proof.
- Test unmodified direct literal and local literal operands; overridden Integer `+`, canonical-operation invalidation; entered Context/Prelude; runtime-varying operands; signed-64 overflow; observability and generic fallback.
- Human executes compilation, focused/full tests, graph captures and git publication. Establish pinned graph/IR improvement and separately measured stable timing before claiming a speedup.
- Remain inside PERF043/#877; no new `Ixxx` is mandated by this evidence.

## Validation and publication provenance

The maintainer reported `git diff --check` clean and all local tests PASS on 2026-10-12. No new PERF043 product changes, test executions, graph captures or benchmarks accompanied that statement in this interaction; **the report is not evidence of this proposed optimization**. The checkpoint is explicitly source inspection and handoff preparation. No specifications were edited and no design decision was ratified.

**Open gates:** independently establish exact benchmark root/phase identity, first IR propagation barrier, canonical guard preservation after proposed lowering, and actual compiled-graph/timing result. Keep PERF043 open.
