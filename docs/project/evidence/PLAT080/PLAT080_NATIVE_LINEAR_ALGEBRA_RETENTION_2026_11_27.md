# PLAT080 — Native linear algebra retention and owner approval

**Date:** 2026-11-27.
**Issue:** [PLAT080 #1212](https://github.com/guillermomolina/protos/issues/1212), native child of [AUD025 #1209](https://github.com/guillermomolina/protos/issues/1209).
**Product inspection revision:** [5f2b40910f09f163e2c2987a4731c0d0d499b373](https://github.com/guillermomolina/protos/tree/5f2b40910f09f163e2c2987a4731c0d0d499b373).
**Disposition:** OWNER APPROVED — retain current native fundamental Vector/Matrix operations; prefer Protos for naturally composable higher-level algorithms. No implementation I warranted.
**Approval provenance:** In the active interaction the owner clarified "no quiero representar con Arrays las matrices y los vectores" and "no es remplaazr por remplazr", then expressly approved the final retention recommendation: "si apruebo." This is coordinator-reported owner approval, not an owner-authored GitHub comment.
**Earlier checkpoint:** [conditional selective migration and comparative research](PLAT080_D_RESEARCH_AND_OWNER_APPROVAL_2026_11_27.md), recorded at [99aa62811b487efec37033f1cbc6be54ee42bbbc](https://github.com/guillermomolina/protos-project-docs/tree/99aa62811b487efec37033f1cbc6be54ee42bbbc). Its open feasibility question is resolved by justified retention, not by a claim that migration is impossible.

## Exact disposition and invariant consistency (GITHUB021)

- Retain private Vector/Matrix representation, native constructors, admission, indexed read/write, physical copy, structured each, Actor/P transfer, scalar kernel and packed Float matmul. Retain the current native add, scale, dot, transpose, matvec and generic matmul implementations.
- Do not use Array as representation or temporary algorithm carrier. Do not introduce wrappers, private kernels, compiler specializations, duplicated algorithms or new allocation mechanisms solely to move existing code into Protos. No public API, normative change, hardware work, mandatory deletion, benchmark campaign or audit metric.
- Prefer source ownership for future higher-level mathematical algorithms when composition is natural and preserves the contract. Reconsider fundamental operations only if independently justified existing canonical mechanisms make the migration useful. No speculative follow-up issue.
- D214-C1/S1/F1, D252-C1, I124, I172, PLAT068-R3, D242 and I115 remain intact: genuine Number admission, exact zero-volume shapes, frozen standard prototypes, ordinary overridable public dispatch, private canonical algorithms, first-product seed, exact Integer zero for empty reductions, ascending order, Hermitian dot, bilinear matrix contractions, coupled Complex rounding, fresh open independent results, state, graph-aware transfer, suspension, Native Image compatibility. Guest primitives remain long/double/boolean. No approved invariant is reopened.

## Source-backed feasibility findings

| Surface | Evidence and consequence |
|---|---|
| Algebra | [ProtosStandardLinearAlgebraProtocol.java](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java#L103-L547) owns the current loops, exact admission, dimensions and construction. scale already shares scaled(); contractions share accumulate()/reduced(). There is no substantial duplicated policy that a wrapper alone would remove. |
| Physical storage | [Vector](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/runtime/values/ProtosVectorValue.java) and [Matrix](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java) own private primitive/reference payloads; Matrix preserves lazy persistent Float packing and exact shapes. No storage change proposed. |
| Indexed execution | [StandardIndexedSelectionNode and BinarySubscript](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java#L1720-L1909) admit Array/Bytes only and guard actual selected behavior. Public indexing extensions would not themselves provide canonical access bypassing user overrides. |
| Reflection and super | CALLABLES.md §11, OBJECT_MODEL.md reflection and EXECUTION_AND_CONTROL.md §8: slotValue returns the stored closure, not arbitrary receiver binding; super starts after methodHome and preserves the original receiver. No public mechanism to invoke canonical Vector/Matrix access on arbitrary operands. Spread admits standard Array only (VALUES_AND_COLLECTIONS.md). |
| Existing source boundary | [Path.protos](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/protos/lib/core/Path.protos) and [ProtosStandardPathProtocol](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/execution/protocols/io/ProtosStandardPathProtocol.java) use mint/componentCount/componentAt with native immutable carrier. Fraction/Complex source operations similarly use private kernels. This is a legitimate pattern, not sufficient proof of optimized per-element access in a contraction. |
| Native call execution | [ProtosBuiltinCallRootNode.executeNative](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBuiltinCallRootNode.java#L152-L185) uses @TruffleBoundary for ordinary native bodies. Replacing a whole kernel with repeated native primitive calls introduces per-element boundaries unless an independently established direct mechanism exists. |
| Arithmetic | [ProtosNumericSelectionNode.standard](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/execution/bytecode/numeric/ProtosNumericSelectionNode.java#L625-L653) recognizes canonical Integer/Float arithmetic; Fraction/Complex source closures do not obtain the same direct path. [canonicalNumberBinary/conjugateNumber](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/src/main/java/com/guillermomolina/protos/runtime/ProtosComplexArithmetic.java#L125-L180) serve all genuine Number combinations without user dispatch. Ordinary operators alone do not prove equivalent execution. |
| Result construction | Matrix(rows, columns) is not a capacity/zero-fill constructor; Vector(3) contains one value. A guest transpose or elementwise transform needs canonical access and direct result construction. A new private builder solely for migration is not warranted. Dot avoids output-container construction and is the best future candidate, but still needs canonical reads and conjugation/arithmetic. |
| Tests | [identity-and-state.protos](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/protos/tests/library/math/linear-algebra/identity-and-state.protos) explicitly redefines v.size to 42 while v.dot(v) remains 5. [algebra-and-reductions.protos](https://github.com/guillermomolina/protos/blob/5f2b40910f09f163e2c2987a4731c0d0d499b373/protos/tests/library/math/linear-algebra/algebra-and-reductions.protos) covers exact, mixed, signed-zero, empty and Complex contractions. ProtosStandardLinearAlgebraProtocolTest and ProtosI115IndexedOperationTest cover native storage and indexed-selection invariants. |

## Comparative evidence (GITHUB010)

GraalPy at 21b9ad46ab3de0ee63960926e51b054e209fd3e3:
- [PyObjectGetItem.doList](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/PyObjectGetItem.java#L102-L126): @Specialization, @Cached ListBuiltins.GetItemNode, guarded exact builtin access.
- [SequenceStorageNodes](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/common/SequenceStorageNodes.java): GetItemScalarNode @GenerateInline/@GenerateCached, SetItemScalarGeneralizingNode.set @Specialization/@Cached, CopyNode @GenerateInline/@GenerateCached(false). Native storage operations.
- [ListBuiltins.ListCopyNode.copySequence](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/list/ListBuiltins.java#L476-L493): @Builtin/@GenerateNodeFactory/@Specialization; direct storage copy and construction.
- [ArrayBuiltins](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/array/ArrayBuiltins.java): SetItemNode.setitem @Slot/@Specialization; ReverseNode.reverse @Builtin/@GenerateNodeFactory/@Specialization, physical permutation. This is a typed Python container precedent, not a proposal to represent Protos values as Array.
- [ComplexBuiltins](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/complex/ComplexBuiltins.java): MulNode.doComplex @Slot/@Specialization; ConjugateNode.hash (actual method name) @Builtin/@Specialization; Python numeric contract differs.
- [MathModuleBuiltins.SumprodNode](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/MathModuleBuiltins.java#L2022-L2190): @Builtin/@GenerateNodeFactory, native exact Integer paths and extended-precision Float reduction, not D214-F1-compatible. ProdNode.doGeneric uses cached iterator/multiply nodes. Python statistics.py retains higher-level algorithms in source.

Earlier survey retained as historical evidence: GraalJS typed-array copy, TruffleRuby ArrayStoreLibrary, FastR MatMult, Espresso arraycopy, Sulong LLVM vector arithmetic, Pkl collection operations, SimpleLanguage arithmetic, Ruby matrix and Julia LinearAlgebra. These illustrate physical primitive versus higher-level algorithm ownership; they do not authorize a new Java bridge. No production Java change is proposed.

## Final alternatives (local identifiers, superseding the earlier conditional feasibility checkpoint)

A: retain existing fundamental native operations; future source composition.
B: source loops using private Path-style native calls.
C: source loops with newly established canonical specialized access.
D: source validation wrappers around complete native algorithms.
Array-backed or Array-intermediate candidates are excluded by owner direction.

| Criterion | A | B | C | D |
|---|---|---|---|---|
| Correctness | 5 existing | 4 caution | 4 unverified | 4 split |
| Protos alignment | 4 justified boundary | 4 source | 5 natural | 2 cosmetic |
| Present proportionality | 5 no machinery | 2 boundaries | 2 extra work | 2 wrappers |
| Incremental growth | 5 independent | 3 helpers | 4 reusable | 2 little value |
| Future resilience | 4 ABI preserved | 3 boundary | 5 reusable | 3 neutral |
| Scalability | 4 existing kernels | 2 per-element calls | 5 potential | 4 kernels |
| Simplicity | 5 one implementation | 3 layers | 3 compiler work | 2 split |
| Portability | 4 adaptable runtime | 4 source | 4 source | 3 native dependency |
| Runtime/resource cost | 4 known structure | 2 repeated boundaries | 5 potential only | 3 extra calls |
| Failure/operability | 5 integrated | 3 new boundaries | 4 reconciliation | 3 split |
| Reversibility | 5 no migration | 3 changes | 3 execution changes | 3 unnecessary edits |
| Evidence maturity | 5 existing | 3 Path analogy | 2 unproven | 4 straightforward |

Confidence high for static findings, moderate for B/D consequences, limited for C potential. Scores are design comparison aids, not AUD025 acceptance metrics. B/D have unjustified layer/cost red flags; C requires an independently justified execution mechanism. A has no demonstrated foundational deferral cost. Strongest argument against A: source loops might benefit from partial evaluation and improve ownership; present native boundaries are not proof of optimality. No experimental cost claims. Larger workloads, Actors/Tasks, cancellation, huge zero shapes, alternative runtimes, sparse matrices and hardware acceleration do not justify speculative machinery now. Escape path: revisit individual algorithms if independently useful canonical specialized access and arithmetic become available; PLAT068 hardware remains separate.

An independent static observation: matmul allocates Object[] before checking the packed Float path, which allocates double[]. Possible redundant allocation, not measured, not in this disposition and no follow-up allocated.

## Investigation and validation limits

Material inspection: root AGENTS.md; AGENTS.work/DESIGN.md, REFERENCE.md, IMPLEMENTATION.md; spec/AGENTS.md, src/AGENTS.md, protos/AGENTS.md; normative VALUES_AND_COLLECTIONS.md, CALLABLES.md, OBJECT_MODEL.md, EXECUTION_AND_CONTROL.md, MODULES.md; representation/protocol/bytecode files above; ProtosFunctionCallDispatchers, ProtosExperimentalGetMethodNode, ProtosClosureInvoker, ProtosCoreBootstrap, structured each, Actor/P paths and tests from the prior inspection; D214, I172, PLAT068-R3 and AUD025 records; IMPLEMENTATION_BLOCKERS.md (no specific PLAT080 entry found). Some earlier secondary normative and external repository inspections were targeted rather than whole-document reads. No product edits, implementation, build, tests, benchmark or spec change. This is a static architectural disposition, not a measured performance result.
