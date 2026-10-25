# PLAT068 — FastR versus Protos R2-Float source comparison and approval checkpoint (2026-10-25)

**Evidence class:** immutable source and explicit owner decision checkpoint, not a benchmark or proof of compiled machine-code acceleration. **Outcome:** R2-Float Matrix storage and strict admitted `matmul` selected; SIMD/native/GPU still unselected. [Canonical non-normative platform decision](../../decisions/platform/PLAT068_R2_FLOAT_ADAPTIVE_MATRIX.md). [PLAT068/#959](https://github.com/guillermomolina/protos/issues/959); [implementation I124/#963](https://github.com/guillermomolina/protos/issues/963); [diagnostics I123/#960](https://github.com/guillermomolina/protos/issues/960).

## Revisions and approval

- Protos current remote `main` observed at [`9dc4e50875252508e2afecf91541fb50f756d450`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450) (2026-10-25), after [LIB024 0.3.401](https://github.com/guillermomolina/protos/commit/d86f587beeefa413e323d345f139ccbb164462ca); this pin does **not** identify or include unpublished local I123 work.
- FastR remote `master` verified at [`f9d9199e51b371f076b19456933ee395d28360bf`](https://github.com/oracle/fastr/commit/f9d9199e51b371f076b19456933ee395d28360bf), dated 2026-07-30; sources below fetched at that exact SHA, not assumed from stale blog material.
- Project owner on 2026-10-25 approved the previously described R2-Float candidate with **“ok apruebo la decision”**. Scope is the private Float representation and strict automatic Float matmul admission; `Matrix.toFloat()` remains a possible future public API requiring separate normative approval.

## Mechanism comparison from actual Java source

| Concern | FastR source at pinned SHA | Protos source at pinned SHA | Ratified transfer |
| --- | --- | --- | --- |
| Float width | `RDoubleArrayVectorData.java` L48–54: `private final double[] data` | `ProtosFloatValue.java` L29–40: `private final double value`; Matrix presently stores `Object[]` | Retain Protos Float binary64 as private `double[]` lanes, not Java `float[]` |
| Read / copy boundary | `RDoubleArrayVectorData.java` L95–103: direct `getReadonlyDoubleData`; `getDoubleDataCopy` uses `Arrays.copyOf` | `ProtosMatrixValue.java` L140–160: `indexedAt` returns Number object; `indexedSnapshot` clones reference array | Private read-only kernel access; public semantic snapshots independently materialize Number values |
| Data strategy | `RDoubleVector.java` L58–94: array / sequence / data closure backing via `setData` | `ProtosMatrixValue.java` L35–69: single R1 `Object[]` | Small Matrix-specific R1/R2 state, pack lazily; no generic library of layouts |
| Typed call | `MatMult.java` L653–674: `@Specialization` and `@CachedLibrary`, dispatch to double-specific matrix routine | `ProtosStandardLinearAlgebraProtocol.java` L263–303: ordinary `matmul` Closure slot executes generic Number helper | Admit only homogeneous genuine Float after existing method/shape checks |
| Numeric kernel | `MatMult.java` L759–820 obtains backing `double[]` and writes `double[]` output; `multiplyBlock` L155–170 seeds `double x = 0.0` and blocked partial accumulations | D214-F1 spec requires first product seed, ascending k, nonfused rounds, empty Integer 0 | **Do not copy FastR reduction**. Retain canonical D214-F1 and merely reuse primitive backing |
| Interop of types | `VectorDataLibrary.java` L406 supports cast-to-R-type data closure; L600–605 default read-only double accessor may copy | D214 admits Integer/Fraction/Float/Complex mixed elements, no implicit numeric promotion | No cast, no public dtype; heterogeneous fallback mandatory |
| Truffle compilation | FastR typed library/admission nodes belong to its Truffle compilation surface | `ProtosComplexArithmetic.canonicalNumberBinary` scalar host helper has `@TruffleBoundary`, repeatedly boxes Float values | Avoid per-product boundary for admitted Float without claiming compile graph/vectorization proved |

**Pinned upstream source URLs:**
- [FastR `MatMult.java`](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.nodes.builtin/src/com/oracle/truffle/r/nodes/builtin/base/MatMult.java)
- [FastR `RDoubleArrayVectorData.java`](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.runtime/src/com/oracle/truffle/r/runtime/data/RDoubleArrayVectorData.java)
- [FastR `RDoubleVector.java`](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.runtime/src/com/oracle/truffle/r/runtime/data/RDoubleVector.java)
- [FastR `VectorDataLibrary.java`](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.runtime/src/com/oracle/truffle/r/runtime/data/VectorDataLibrary.java)
- [Protos `ProtosFloatValue.java`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/src/main/java/com/guillermomolina/protos/runtime/ProtosFloatValue.java)
- [Protos `ProtosMatrixValue.java`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java)
- [Protos `ProtosStandardLinearAlgebraProtocol.java`](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java)
- [Protos D214 normative owner](https://github.com/guillermomolina/protos/blob/9dc4e50875252508e2afecf91541fb50f756d450/spec/semantics/VALUES_AND_COLLECTIONS.md)

## Semantic counterexamples and implementation requirements

- `(-0.0) * (+0.0)` is negative zero; initializing the accumulator to +0.0 before adding the product can change sign.
- For a nonempty product sequence, the first product **is** the accumulator, not +0.0. Each subsequent product is multiplied separately, then added in original order, with two floating rounds.
- `Math.fma(a,b,c)` computes a single fused round and differs from `(a*b)+c`. Witness `(1+2^-27) * (1-2^-27) - 1`.
- Reassociation on e.g. `[1e16, -1e16, 1]` changes the result. Blocked partial sums, order-changing vector reduction or BLAS must not be substituted.
- Zero inner dimension returns genuine Integer 0 even when both containers are vacuously homogeneous; no special Float zero result.
- Heterogeneous elements (including Integer, Fraction, Complex) retain exact R1 semantics; assigning such a Number to a packed Matrix must reify R1 without changing Matrix identity or exposing an intermediate state.
- Snapshot/copy/transpose/Actor/P transfers preserve value semantics and independent ownership; the packed primitive array must not leak to guest or other runtime contexts.

## Limits of existing performance evidence

[I123 human-reported timing checkpoint (2026-10-24)](../I123/I123_CPU_RUNTIME_JIT_MEASUREMENT_CHECKPOINT_2026_10_24.md) reports repeatable **isolated Java** column-lane kernels and smaller gains when temporary packing is included. The tested eight-operation simulated R2 chain did **not** prove a uniform R2 advantage. End-to-end Protos baseline was `NOT_STABLE` (e.g. 32×64×32 ≈234310 ns, dispersion 0.2302); neither actual emitted SIMD instructions nor native-image parity were proven. Local diagnostic sources may remain staged/uncommitted. Therefore:

- `R2_FLOAT_ARCHITECTURE=OWNER_APPROVED`
- `PRODUCTION_R2_IMPLEMENTATION=NOT_YET_PUBLISHED`
- `PROTOS_END_TO_END_SPEEDUP=NOT_PROVEN`
- `SIMD_MACHINE_CODE=NOT_PROVEN`
- `GPU_NATIVE_MATRIX_BACKEND=NOT_APPROVED`
- `PUBLIC_TOFLOAT_API=DEFERRED`

No build, test, benchmark, or repository-content mutation in `guillermomolina/protos` was performed to create this source-comparison record.
