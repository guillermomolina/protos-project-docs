# PLAT068 — R2-Float adaptive Matrix storage (owner-ratified partial platform decision)

**Selected component:** PLAT068 R2-Float, 2026-10-25. **Parent platform issue:** [PLAT068/#959](https://github.com/guillermomolina/protos/issues/959), which remains OPEN for SIMD, CPU matrix ISA, native-library and GPU admission. **Implementing issue:** [I124/#963](https://github.com/guillermomolina/protos/issues/963), READY but not yet implemented. **Prior diagnostic:** [I123/#960](https://github.com/guillermomolina/protos/issues/960), separately open.

**Approval provenance:** after the comparison with the current FastR representation and strict canonical Protos Float semantics, the project owner expressly responded **“ok apruebo la decision”** on 2026-10-25 to the R2-Float recommendation. This is an exact, bounded approval of the private Matrix representation and Float-matmul admission below. It does not approve GPU, SIMD, native BLAS, public casts, Vector storage conversion, a changed numerical contract, or measured speedups.

**Reference revisions:**
- [Protos § D214 implementation and spec at `9dc4e50875252508e2afecf91541fb50f756d450`](https://github.com/guillermomolina/protos/commit/9dc4e50875252508e2afecf91541fb50f756d450), inspected 2026-10-25.
- [FastR `f9d9199e51b371f076b19456933ee395d28360bf`](https://github.com/oracle/fastr/commit/f9d9199e51b371f076b19456933ee395d28360bf), repository `master` inspected 2026-10-25.
- [Source-level FastR/Protos evidence](../../evidence/PLAT068/PLAT068_FASTR_R2_FLOAT_SOURCE_COMPARISON_2026_10_25.md); [earlier GITHUB010 CPU/GPU screen](../../evidence/PLAT068/PLAT068_PREDECISION_CPU_SIMD_GPU_RESEARCH_2026_10_24.md); [A1/T checkpoint](../../evidence/PLAT068/PLAT068_CPU_TRUFFLE_JIT_A1_T_APPROVED_EXPERIMENT_2026_10_24.md).

## Ratified, intentionally narrow mechanism

1. **One ordinary Matrix; two invisible storage states.** An ordinary `ProtosMatrixValue` retains its identity, delegation, slots, exact semantic Integer dimensions and row-major coordinate contract. R1 is the initially allocated private `Object[]` of genuine Number values. An exclusively-Float Matrix may transition **lazily**, at first eligible computation, to a private **persistent `double[]`** (IEEE 754 binary64); it is not re-packed for every eligible matrix operation. No eager packing on ordinary construction/import, no global cache, no hardware probes, no public dtype, no mandatory extra mechanism for non-users.
2. **Precise admission.** The eligible domain consists only of elements that are actual Protos Float values (`ProtosFloatValue`, whose private numeric member is Java `double`), with compatible Matrix shapes and nonempty contraction as required. Numeric Integer (including Small/Big), Fraction and Complex values are never silently converted to Float; any heterogeneous Matrix is handled by R1 canonical arithmetic. A rejected R2 attempt may cache a private negative result until a mutation invalidates it.
3. **Persistent backing and mutation.** Repeated eligible calculations reuse R2 without repacking. Valid `atPut` of a Float to R2 updates only its lane. Valid `atPut` of a genuine non-Float Number materializes the old Float lanes into a fresh independent R1 payload, inserts the supplied Number and transitions that same Matrix instance to R1, all without an observable half-written state. Frozen validation, bounds and numeric-admission precedence happen before mutation. Subsequent eligibility is re-evaluated when necessary. No observable dtype promotion/demotion or public forced conversion.
4. **The Float-specific strict matmul path.** Preserve ordinary guest method lookup/dispatch. Within the existing admitted `Matrix.matmul` native operation, after existing argument, dimension, volume and domain gates, route **Float × Float** to primitive row-major arrays and an independent primitive result buffer. For each output cell, multiply at `k=0` to seed the accumulator; for `k=1..inner-1` evaluate a separate product followed by a separate sum in ascending k. Construct genuine result Float Number values only after completing numeric work; the Matrix result is fresh and independent. If the contraction is empty, each result cell is exact Protos Integer `0`, never Float `0.0`. No `Math.fma`, reassociation, tree reduction, blocked partial reduction, BLAS/GPU substitution, or changes to subnormal/NaN/Inf/signed-zero behavior. No promise of automatic CPU vectorization.
5. **Fallback and transfer.** Retain a single canonical heterogenous Number implementation path for all existing Vector/Matrix operations outside the eligible Float-matmul kernel. `indexedAt` and snapshot/transfer materialization must reify genuine Float Number values when backed by R2 without exposing the mutable array. `copy`, `transpose`, mutable/frozen behavior, Actor/P transfer, graph identity and isolation remain equivalent to R1. C1 object slots and delegation remain ordinary.
6. **Scope discipline.** Only Matrix internal representation and admitted `matmul` are covered. Vector R2, Integer `long[]`, SIMD lane scheduling, Native Image compiler graph optimization, GPU/BLAS, floating exception policy amendments and a guest `Matrix.toFloat()` method remain separate, **unapproved** decisions/work. An explicit conversion would create observable library semantics and requires a D-family decision, not this private optimization.

## FastR mechanism and explicit non-transferable details

At pinned FastR `f9d9199e`:
- `RDoubleVector` delegates to data representations (including array, sequence and closure forms); `RDoubleArrayVectorData` owns `double[]` and exports both a direct read-only-access method and a copying method. `VectorDataLibrary` provides representation-polymorphic cached access and a copy fallback.
- `MatMult.MatMultAsDouble` has a typed Truffle `@Specialization` with `@CachedLibrary` data access; `doubleMatrixMultiply` reads `double[]` and writes a `double[]` result. Its block loop explicitly initializes a partial sum with `0.0` and accumulates block contributions. That arithmetic is **not equivalent** to D214-F1's first-product seed and strictly ascending single-cell reduction.
- FastR is a representation/typed-dispatch precedent **only**; it does **not** implement or prove Protos heterogeneous `atPut` generalization or its Actor/P transfer semantics. Those remain Protos-specific obligations.
- A Truffle specialization is not evidence that Protos native Closure calls inline or vectorize. Compiler IR, tier-specific machine instructions and end-to-end guest workload timings must be independently demonstrated later.

## GITHUB021 — exact D214 reconciliation

| Owner-approved invariant | R2-Float effect / required implementation invariant |
| --- | --- |
| C1 ordinary Matrix identity, prototype, slots | Preserved; backing is private, neither a new guest Number family nor a new prototype |
| All genuine Number values, heterogeneous containers | Preserved; only actual Float values qualify; mixed values stay on R1 |
| Exact shape, row-major indexing and mutation/error order | Preserved; validate before storage transition, retain exact semantic dimensions |
| Ordinary library-method message dispatch | Preserved; specialization occurs only after ordinary selected method and existing argument gates |
| Strict D214-F1 reduction | Preserved; first product, ascending k, no FMA/reassociation; empty exact Integer 0 |
| Hermitian dot and bilinear matvec/matmul; Complex rounding | Preserved; optimize only Float matrix matmul, leave other operations and Complex canonical |
| Fresh outputs, freeze, copy, Context/Actor/P identity/isolation | Preserved; snapshots materialize independent values, no shared mutable array |
| Pay-as-you-grow, portable JVM/Native fallback | Preserved by lazy local R2, no new external dependencies, hardware requirements or global initialization |

**Result:** `INVARIANT_CONSISTENCY=PASS` for the bounded selection, conditional on faithful implementation and human validation. This is an architectural permission, not an assertion that source has already changed.

## GITHUB010 decision delta: R1, temporary T and persistent R2

The earlier PLAT068 research compares A/R1, transient T, CPU SIMD, native and GPU across all twelve mandated axes. After source-level FastR comparison and human I123 timing reports, the owner selected only persistent R2-Float for the narrow Float matrix hot path. Scores here are qualitative selection-time estimates, **not measured claims**.

| Dimension (1–5; confidence) | Existing R1 | Transient T | Approved R2-Float |
| --- | --- | --- | --- |
| Correctness/invariants | 5 H | 4 M | 4 M |
| Protos alignment | 5 H | 4 M | 5 M |
| Pay for present need | 5 H | 3 M | 4 M |
| Incremental growth | 3 M | 4 M | 5 M |
| Future-option resilience | 4 H | 4 M | 5 M |
| Scalability | 2 M | 3 M | 4 M |
| Conceptual simplicity | 5 H | 4 M | 3 M |
| Portability | 5 H | 5 H | 5 M |
| Runtime/resource cost | 2 M | 3 L | 4 L |
| Failure/operability | 5 H | 4 M | 3 M |
| Deferral/migration cost | 4 H | 4 M | 4 M |
| Evidence maturity/implementation risk | 5 H | 3 M | 3 M |

`H` = HIGH, `M` = MEDIUM, `L` = LOW confidence. R2's low runtime-cost confidence reflects missing stable guest end-to-end JIT measurement. It is selected for the precise amortizable representation boundary, **not** because it has already proven faster. Strongest counterargument: a persistent dual representation adds mutation, snapshot and transfer complexity and may not amortize for short-lived/small matrices; its lazy admission and generic fallback are essential.

**Growth versus preimplementation:** only one Float Matrix representation and one direct matmul path now; no general multi-type storage framework, SIMD/GPU service, new guest control, native library or forced-cast API. Later specialized backends must establish exact equivalence and separate owner authorization. Deferral of SIMD, GPU and dtype API carries no current semantic migration obligation.

## Delivery and measurable acceptance

- [I124/#963](https://github.com/guillermomolina/protos/issues/963) owns the concrete Java source/test changes, preserving the user-owned dirty I123 diagnostic paths.
- Human executes targeted static checks/tests and the final impacted full-suite gate. `pom.xml`/`CHANGELOG.md` changes, if required, follow only green tests and precede human commit/push; no tests afterward.
- Use byte-exact Float witnesses (including first-product signed zero, FMA difference, non-associativity, NaN/Infinity/subnormal), heterogeneous transitions, error precedence, copying, freeze and Actor/P tests.
- Do **not** claim a guest speedup, SIMD emission or Native Image parity until measured against the exact product commit. I123 retains distinct diagnostic responsibility.
- Exact I124 product SHA and later revision-bound closure evidence are pending. PLAT068 remains open for unselected accelerators.
