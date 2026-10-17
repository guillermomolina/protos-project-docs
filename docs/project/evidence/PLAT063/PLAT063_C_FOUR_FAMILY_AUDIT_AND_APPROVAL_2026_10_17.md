# PLAT063-C — Four-family emission audit and GITHUB010/GITHUB021 record

**Scope:** READ-ONLY architecture investigation over exact [`guillermomolina/protos@c00b36c0edaf6678b7d8101de3a405124f179644`](https://github.com/guillermomolina/protos/commit/c00b36c0edaf6678b7d8101de3a405124f179644); [issue #908](https://github.com/guillermomolina/protos/issues/908). **Owner approval:** 2026-10-17, exact “Apruebo C”, of presented C (one common `While` for the five standard collections). Local tests/diff reported green by maintainer **before any PLAT063-C implementation**, not a C acceptance result. No code, build, benchmark, or executable new oracle produced in this investigation.

## Finite four-family classification

| Family / source / canonical form | Emission trace | Semantic boundary / adversary | Outcome and owner |
|---|---|---|---|
| Ordinary send `obj.m(arg)` / `CanonicalSend` | stage receiver + args; `PrepareSendZero/One/Arguments/Vector` owns PIC/selection; physical invocation follows | argument can mutate selector/receiver, lookup must follow source evaluation | **ALREADY_ENCAPSULATED / KEEP**; Closure execution belongs to PLAT062/#907 |
| Slot/Context `x=f()` / `CanonicalAssign`, `CanonicalLookup` | resolve lexical target; evaluate RHS; write selected exact target; `ReadRootFrameLocal` compact/materialized and scalar-local alternatives | `f()` may shadow/remove/freeze bindings; selection before RHS and exact target identity observable | **JUSTIFIED_DECOMPOSITION / EXISTING_OWNER** PLAT036/D179/PERF034/#845 and PERF036/#850 |
| Construction `%{k:v}` / `CanonicalMapConstruction` | ordinary Map factory then per-entry key/value, `PrepareMapInitialDefinition`, hash callback in comparison `TryFinally`, equality `While` with callback `TryFinally` | hash/equality may suspend, fail or mutate; compare restriction, evaluation order, duplicate-key Error | **JUSTIFIED_DECOMPOSITION / KEEP**, normative collection law |
| Canonical standard `m.each(block)` / selected `CanonicalSend` | five almost identical dispatcher branches, each `IsStructured*EachCall` / `Prepare*` / `TryFinally` / `While(hasNext)` / `PrepareChild` / scoped callback / `Advance` / `Finish` | collection-specific prevalidation/snapshot + callback dynamic, suspend/Error/NLR, copied/overridden home, empty sequence | **CANDIDATE → RATIFIED C**, physical single structural loop only; PERF045/#889 remains performance owner |

Exact pinned first-party anchors:
- [structured dispatcher 5 branches](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java#L868-L1275), [shared prepared interface](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java#L4600-L4660), [existing inline shared loop](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java#L6323-L6403), [shared source-root DSL operations](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java#L6355-L6404).
- [ordinary send emitter](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java#L9164-L9600), [lexical lookup](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java#L7285-L7469), [Map construction](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java#L4383-L4609), [independent handwritten scalar-local Builder oracle](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101BLocalBindingBuilderOracleTest.java#L64-L149).
- Normative: [execution order and lexical destination](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/spec/semantics/EXECUTION_AND_CONTROL.md#L205-L247), [composition](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/spec/semantics/OBJECT_MODEL.md#L340-L420), [ordinary call](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/spec/semantics/CALLABLES.md#L602-L711), [Map construction](https://github.com/guillermomolina/protos/blob/c00b36c0edaf6678b7d8101de3a405124f179644/spec/semantics/VALUES_AND_COLLECTIONS.md#L531-L604). `CanonicalCompilationFacts` and `CanonicalNonSuspensionAnalysis` already own shared proofs; this decision adds none.

## Actual contrast, not an instruction dump

```text
CURRENT: five separate IsStructuredFamilyEachCall / PrepareFamilyEachCall /
  TryFinally { While(FamilyHasNext) { PrepareFamilyChild; Invoke; AdvanceFamily } FinishFamily }
C: retain exact chosen family/preparation; ONE TryFinally {
  While(StructuredLocalEachHasNext) {
    PrepareStructuredLocalEachChildCall; exact scoped prepared invocation;
    AdvanceStructuredLocalEach
  }
  FinishStructuredLocalEach
}
```

The five original branches and shared local literal-callback loop are observable **emitter source**; no dynamic instruction listing, matched compiled graph, timing or CodeSize evidence was captured. Truffle may already erase unused paths. The main falsifier of runtime benefit is polymorphic `PreparedLocalEachCall` dispatch surviving PE; bytecode shortening alone cannot validate speed.

## Source-pinned comparative precedents and differences

- [GraalPy Bytecode DSL `ForIterate`](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java#L1869-L1945) specializes iterator progression, while [its compiler](https://github.com/oracle/graalpython/blob/7e7182b1036d56f8e89ecc9c7a86cee292180aa6/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java#L1392-L1423) retains a structural `While`: C-like operation/control separation, not shared Protos snapshot semantics.
- [GraalJS `IteratorStepNode`](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/IteratorStepNode.java#L50-L71) and [`IteratorNextNode`](https://github.com/oracle/graaljs/blob/4327aa1f2914bac7f810406423d70ae7f8a1888b/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/access/IteratorNextNode.java#L57-L87) encapsulate mechanical next/completion checks; AST nodes are not equivalent to Bytecode DSL operations.
- [Apple Pkl `GeneratorForNode`](https://github.com/apple/pkl/blob/9f79fe47da8580ee91e9aa61302352c5d68bace1/pkl-core/src/main/java/org/pkl/core/ast/expression/generator/GeneratorForNode.java#L96-L208) retains *different* type-specialized loops with shared iteration body; counterargument against compulsory consolidation. Its per-item safepoint is not Protos law.
- [Espresso execution/quickening](https://github.com/oracle/graal/blob/ce5548729f355c81d854f2d52aad26ad515a4dec/espresso/docs/how-espresso-works.md#L81-L99) illustrates specialized semantic mechanics without erasing loop control, but Java invocation is a weaker `each` analogue.
- Outside Truffle, CPython bytecode iteration uses dedicated iterator operations and control flow (general precedent only; no version-specific claim here). TruffleRuby and Sulong AST/LLVM architectures, and SimpleLanguage toy loops, were considered but not treated as equivalent to this already-existing Protos prepared-state interface.

## GITHUB010 candidates and 12-axis scorecard

A = retain five loops (also the deferral path); B = factor emitter Java but still emit five loops; C = one shared **emitted** Bytecode `While` over `PreparedLocalEachCall`. **Scores 1–5 with H/M/L confidence** (subjective design comparisons, not benchmarks):

| Dimension | A | B | C | Justification |
|---|---|---|---|---|
| 1 correctness | 5H | 5M | 3M | C must validate resume/cleanup |
| 2 Protos alignment | 3H | 4H | 5M | C reuses existing state contract |
| 3 present need/pay-as-you-grow | 5H | 4H | 3L | no material cost yet measured |
| 4 incremental growth | 3H | 4H | 5M | compatible families share physical loop |
| 5 future options | 5H | 5H | 4M | incompatible family can still have separate path |
| 6 scalability | 3L | 3L | 4L | less CodeSize possible; megamorphism risk |
| 7 simplicity | 3H | 4H | 4M | less emitted duplication, bounded adapter |
| 8 portability | 5H | 5H | 5M | no public/host coupling |
| 9 runtime/resource | 4L | 4L | 3L | PE, CPU and allocations unmeasured |
| 10 failure/operability | 5H | 4M | 3M | C must retain all TryFinally boundaries |
| 11 deferral/reversibility | 5H | 5H | 4M | later refactor is bounded |
| 12 evidence/risk | 5H | 4M | 3M | internal shared path exists; graph proof absent |

**Anti-overengineering gate:** C would be unjustified if the only objective were prettier printed bytecode; explicit owner approval establishes a durable structural architecture, **not** that runtime improves. The smallest C uses current interface and scoped invocation, introduces no universal iterator API, registry or new core node; test alternatives against actual equal-workload graph/CodeSize only if PERF045 requests measurement. **Cost of deferring:** maintenance and emitted duplication, no known compatibility migration. **Strongest argument against C:** five monomorphic paths could compile faster than one polymorphic `hasNext/prepare/advance` interface. **Escape route:** a specific future incompatible collection retains its own structured control; restoring former emission requires no guest API migration.

Future stress: empty/million-element snapshot, heterogeneous receivers, nested callbacks, multi-Context/Actor/Task/Process, cancellation/error/NLR while suspended, Native Image, DEBUG stepping, reparse and alternate Truffle versions. C must not add polling, allocations, task state or cross-Context cached owners to cold paths.

## GITHUB021 invariant/delta reconciliation

**Approval provenance:** owner explicitly says “Apruebo C”. Check PLAT042 (single structured owner), PLAT044 (physical/semantic callback and tooling), D013 (overridable `call`), D179/PLAT036 (lexical authority), PLAT057–061/D202–D204/I095/I101/I104/I106/I107 (proof/ABI), and normative per-family snapshots and result: **all PRESERVE, OBSERVABLE_DELTA=NONE** for *C as defined*. Retain structurally external Bytecode `While`, `TryFinally`, `Yield` and exact prepared callback fallback. If concrete implementation forces any conflicting delta, STOP for new explicit approval. **DECISION_APPROVAL_PROVENANCE=PASS, DECISION_INVARIANT_CONSISTENCY=PASS (design contract), IMPLEMENTATION_NOT_YET_VALIDATED=YES**.

No overlap: [PLAT062/#907](https://github.com/guillermomolina/protos/issues/907) owns Closure-call machinery; [AUD016/#701](https://github.com/guillermomolina/protos/issues/701) is completed; PERF040/#862 numeric paths and PERF046/#890 equality remain distinct. PERF026-D/#769 literal-callback path is completed and must not be redone. [PERF045/#889](https://github.com/guillermomolina/protos/issues/889) owns measured each-family graph/runtime and transitive library/foreign inlining. New ratified C implementation requires **a separate I-family issue**, not a new PLAT and not an issue per collection.
