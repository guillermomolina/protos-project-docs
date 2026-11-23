# PLAT068-R3 — Approved private numeric storage alignment for Vector and Matrix

**Decision owner:** [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959). **Date:** 2026-11-23. **Source baseline:** [Protos `89028494587d4d22309a5c048e37da3518170b18`](https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18). **Audited evidence:** [AUD024](../../work/AUD024/AUD024_RUNTIME_VALUE_REPRESENTATION_CORRECTNESS_AUDIT_2026_11_23.md), [exact source trace](../AUD024/AUD024_RUNTIME_REPRESENTATION_SOURCE_TRACE_2026_11_23.md). **Implementation owner:** [I172 #1161](https://github.com/guillermomolina/protos/issues/1161).

## Explicit owner selection and limits

The project owner first expressly directed an architectural-correctness review, not performance work: `Bytes` should hold actual `byte[]` and `Array`, `Vector`, `Matrix` should have all **semantically applicable** primitive storage specializations. The audit was presented and explicitly accepted by the owner («sí, las doy por buenas»). Subsequently the agent stated the **exact intended change to PLAT068**, listing the target storage per family and explaining that this supersedes the *previous exclusion* of Vector / Integer packing:

- Array: retain `long[]`, `double[]`, `boolean[]`, `Object[]` already implemented.
- Vector: add `long[]`, `double[]` and retain `Object[]` fallback.
- Matrix: add `long[]`, preserve existing lazy persistent Float `double[]` and generic `Object[]`.
- `Bytes` `byte[]` is **not** PLAT068 scope: [I171 #1160](https://github.com/guillermomolina/protos/issues/1160).
- No numeric family coercion, no public dtype or new selectors, no change of message semantics, no performance benchmark as acceptance gate.

**Owner's exact response:** «ok actualiza PLAT068». This is an approval to amend the existing decision with the above **bounded physical storage model**, not approval of another solution, public API, numerical kernel, SIMD/GPU, or a claim that implementation passed tests. Historical PLAT068-R2 Float approval (2026-10-25) and post-I124 SIMD deferral C remain in force. No additional Ixxx/Dxxx/PLATxxx is necessary.

## Sources inspected and actual change boundary

- `ProtosArrayValue.IndexedGeneration`: existing `long[]`/`double[]`/`boolean[]`/`Object[]` and copy-on-write snapshot (KEEP).
- `ProtosVectorValue`: only `Object[] elements`; constructor, `indexedAt`, `indexedPut`, `indexedSnapshot`, `transferShellForRuntime`, `transferPutForRuntime` require typed storage consistency.
- `ProtosMatrixValue`: volatile `Object storage` with `Object[]`/persistent `double[]`; `indexedAt`, `indexedPut`, `indexedSnapshot`, `physicalLength`, `readonlyPackedFloatsForRuntime`, and transfer population must handle `long[]`. **Exact hazard:** the Float reader assumes any non-double storage is `Object[]`; after this amendment it must reject `long[]` safely. CAS return may not unconditionally be cast to `double[]`.
- `ProtosNumericValueSupport`: authentic semantic SmallInteger `long` carrier and Float binary64 `double` or existing wrapper; BigInteger/Fraction/Complex remain genuine Numbers stored in `Object[]`, with no implicit narrowing/conversion.
- Normative `spec/semantics/VALUES_AND_COLLECTIONS.md` D214 C1/S1: Number only (not Boolean) for Vector/Matrix, strict freeze/bounds/admission order, exact assigned object answer, shape, snapshots, graph transfer, and D214-F1 arithmetic. R2 permitted only when invisible.

## Concrete Truffle provenance

These mechanisms, rather than invented Java framework machinery, govern the implementation:

- [GraalPy `LongSequenceStorage`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/sequence/storage/LongSequenceStorage.java): `long[] values`, `getLongItemNormalized`, `setLongItemNormalized`, `getCopyOfInternalArrayObject`.
- [GraalPy `DoubleSequenceStorage`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/sequence/storage/DoubleSequenceStorage.java): `double[] values`, `getDoubleItemNormalized`, `setDoubleItemNormalized`, `getCopyOfInternalArrayObject`.
- [GraalPy `ObjectSequenceStorage`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/sequence/storage/ObjectSequenceStorage.java): `Object[]` fallback, `insertItem`, `getCopyOfInternalArrayObject`.
- [TruffleRuby `LongArrayStore`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/core/array/library/LongArrayStore.java): `@ExportLibrary(ArrayStoreLibrary.class, receiverType=long[].class)`, `@ExportMessage` `acceptsValue`, `CopyContents.copyContents`, `GeneralizeForValue.generalize` with `@Specialization`.
- [TruffleRuby `DoubleArrayStore`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/core/array/library/DoubleArrayStore.java): same typed backing, `@ExportMessage`/`@Specialization` for generalization; `ObjectArrayStore` generic counterpart.
- FastR `RDoubleArrayVectorData` / `RIntArrayVectorData` with `@ExportLibrary(VectorDataLibrary.class)`: typed primitive vector data; its `int[]` cannot be copied as Protos long. GraalJS `AbstractDoubleArray.convertToObject` adds corroborating generic fallback. Espresso Java arrays are statically typed rather than guest-number-generalizing; Sulong models native layouts, SimpleLanguage doesn't define these specialized numeric collection states, and Apple Pkl does not provide the same mutable numeric Vector/Matrix storage precedent. None is normative authority over D214.

## GITHUB021 — original ratified invariants versus amendment

| Prior ratified invariant | R3 impact / proof obligation |
| --- | --- |
| One ordinary C1 Matrix/Vector, fixed shape, prototypes and slots | **KEEP**; typed payload is private, no public type or new object |
| Genuine Number recognition incl. rich/mixed values | **KEEP**; only all-SmallInteger or all-Float receive typed lanes; `Object[]` for everything else; Boolean excluded |
| Frozen → bounds → admission check and unchanged state on failure | **KEEP**; validate first, then select/write/copy storage; preserve returned argument object |
| Ordinary guest message lookup for algebraic methods | **KEEP**; no bypass via direct host arithmetic for unselected numeric cases |
| D214-F1 exact first product, ascending contraction, no FMA | **KEEP**; retain current Float `matmul`; do not invent primitive Integer/vector reductions |
| Hermitian dot, bilinear matvec/matmul, coupled Complex products | **KEEP**; no change in scalar algorithms or numeric conversion |
| Copy/snapshot/Actor/P alias and immutable isolation | **KEEP**; snapshot of primitive backing must be independent of subsequent writes |
| Pay-as-you-grow, portable JVM/Native Image, independent per-value backing | **KEEP**; no global cache, hardware requirement, package or Java class hierarchy |
| Previous PLAT068 scope excluded Integer/Vector primitive packing | **EXPLICITLY AMENDED** by owner's later direction and exact «ok actualiza PLAT068»; this changes *private scope*, not the eight D214 invariants |
| Previously approved persistent `double[]` Matrix Float and SIMD deferral | **KEEP** unchanged; R3 adds no SIMD/GPU/BLAS and requires no benchmark evidence |

`GITHUB021_INVARIANT_DELTA=PASS_FOR_PRIVATE_STORAGE_ONLY`; `PUBLIC_SEMANTIC_DELTA=NONE`; `R2_FLOAT_PERSISTENCE=UNCHANGED`. All implementation behavior remains to be verified by human-run correctness tests; this is not source execution.

## GITHUB010 — viable alternatives, comparative judgment

| Candidate | Technical shape | Disposition |
| --- | --- | --- |
| A — keep old R1/R2-Float only | Vector `Object[]`, Matrix `Object[]/double[]` | Reject: leaves owner-identified representation incoherence |
| **B — extend existing value classes** | `long[]/double[]/Object[]` in Vector/Matrix, preserve existing Array and Float Matrix model | **Selected**: smallest correct private architecture |
| C — common storage library with typed adapters | Cross-family `@GenerateLibrary` + several new Java classes | Defer: unnecessary coupling and machinery for current invariants |
| D — forcibly convert all Numbers to Float | One physical `double[]` | Disqualified: violates D214 admission, exactness, mixed family and signed64/rich Number |

**Scores: 1–5 (H/M/L = confidence), architectural judgment not benchmarking:**

| Axis | A keep | B selected | C framework |
| --- | --- | --- | --- |
| Correctness/invariant preservation | 4H: old semantics but audit goal unmet | 5M: typed + correct fallback | 4M: more states to prove |
| Protos-first alignment | 3H: present inconsistency | 5M: no new public category | 3M: extra Java abstraction |
| Pay for present need | 2H: missing requested storage | 5M: per-value admissible backing | 2M: speculative framework |
| Grow as needed | 2H: defers known correction | 5M: simple fallback and extension | 4M: shared extension potential |
| Future-option resilience | 3M: later rework | 5M: primitive states remain private | 4M: extensible but coupled |
| Scalability | 3M: generic refs in all vectors | 4M: per-value storage, no globals | 4M: reuse but costs |
| Conceptual simplicity | 5H: existing state | 4M: local types only | 2M: hierarchy burden |
| Portability | 5H: plain Java arrays | 5H: plain Java arrays | 4M: more generated nodes |
| Runtime/resource costs | 3M: reference array | 4M: typed when applicable | 3L: more framework pressure |
| Failure/operability | 5H: established R1 | 4M: strict switch/CAS required | 3M: extra indirection |
| Deferral/migration/reversibility | 2H: keeps current mismatch | 5M: transparent storage change | 3M: rollback across classes |
| Evidence maturity/risk | 5H: existing implementation | 4M: GraalPy/TruffleRuby typed precedents | 3M: template exists, Protos need unproven |

Counterexamples/stress cases: inhomogeneous Number mixture, transition after mutation, `long` limits and BigInteger, -0.0/NaN/Inf Float, zero-volume Matrix with arbitrarily large semantic dimensions, frozen Matrix R2 CAS read, snapshot during reentrant `each`, Actor/P graph cycles, native-image builds, many independent Contexts, later SIMD/GPU. B remains applicable **only when every observable Number/value/transfer invariant survives**.

**Against B:** more branching for every indexed read/write, representation state transitions and snapshot/transfer proof. Those costs are paid by implementations only, within existing value classes; a new generalized framework is not warranted. No old infrastructure, indirect guest messages or new numeric kernels are justified by this amendment. No benchmark/speedup gate.

## Exact outcome, responsibilities and deferrals

- `PLAT068_R3_PRIVATE_NUMERIC_STORAGE=OWNER_SELECTED_AND_DURABLY_RATIFIED`.
- `I172=READY_FOR_CORRECTNESS_IMPLEMENTATION`; independent from [I171 #1160](https://github.com/guillermomolina/protos/issues/1160), which may be implemented in the same editing slice.
- `ARRAY_STORAGE=KEEP`; `VECTOR_STORAGE=LONG_DOUBLE_OBJECT`; `MATRIX_STORAGE=LONG_DOUBLE_OBJECT`; `BYTES=SEPARATE_I171`.
- `SIMD_GPU_BLAS_CPU_MATRIX_ISA=UNSELECTED`; `NEW_PUBLIC_API=NONE`; `PRODUCT_EDITS=NONE`; `TESTS=NOT_RUN`; `BENCHMARKS=NOT_REQUIRED`.
- PLAT068 Issue stays **OPEN/READY** for its independent *hardware/backend* questions, not for any unresolved I172 storage choice. Any new observable semantics require separately approved Dxxx; this record does not amend the normative specification.
