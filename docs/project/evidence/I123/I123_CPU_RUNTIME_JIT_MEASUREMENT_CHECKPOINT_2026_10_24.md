# I123 — strict CPU kernel: human measurement and validation checkpoint (2026-10-24)

**Record role:** provisional, revision-qualified diagnostic evidence. **Owning Issue:** [I123 #960](https://github.com/guillermomolina/protos/issues/960). **Related decision:** [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959). This record is **not** a production optimization, ratification, or I123 closure.

## Provenance and authority

- **Normative:** D214-S1, `Standard Library Vector and Matrix in `spec/semantics/VALUES_AND_COLLECTIONS.md`; C1 identity and D214-F1 strict arithmetic remain authoritative. Original ratified source: [798797a72ff3d292b52870da59f595c8607ca2d7](https://github.com/guillermomolina/protos/commit/798797a72ff3d292b52870da59f595c8607ca2d7).
- **Published library baseline:** [LIB024 Protos d86f587beeefa413e323d345f139ccbb164462ca](https://github.com/guillermomolina/protos/commit/d86f587beeefa413e323d345f139ccbb164462ca), released as 0.3.401.
- **Current read-only public source observation:** [Protos 3f37ef3fdd507504720bf30008317b6304ccffa6](https://github.com/guillermomolina/protos/commit/3f37ef3fdd507504720bf30008317b6304ccffa6). This is **not** identified as the source of the local I123 measurements.
- **Unpublished local diagnostic source:** `ProtosI123CpuKernelEligibilityTest.java`, `Plat068CpuKernelProbe.java`, `Plat068NumericPolicyProbe.java` are reported in the human executor's local checkout but absent from the consulted public HEAD; their exact contents, local diff and resulting commit SHA were **not independently audited**.
- **Environment reported in the investigation:** GraalVM 25.4.4.1.1 / JDK 25.0.4.1.1. Exact CPU ISA/affinity, heap, GC settings and full per-run command manifest are not independently verified here.

## Human-executed correctness and repository checks

On 2026-10-24 the maintainer reported `git diff --check` clean and all local tests PASS. This is accepted as a **human-reported** local validation status, without falsely claiming that this coordinating agent ran tests, saw the complete test transcript, or verified a public I123 product commit. A clean diff check does not imply an empty worktree.

The diagnostic report states three `--measure` Java runs completed with return code 0 and exact-result comparisons for the investigated strict kernels. Numeric witnesses include FMA, reassociation, signed zero, NaN/Infinity, subnormals, empty contractions, integer/fraction fallback and Complex cases. The precise per-test transcript is not retained in this record.

## CPU-only Java kernel measurements (provided evidence, not Protos speedups)

`strictColumnLanes` over `strictCells` (same host-kernel workload; approximate ratios from repeated run observations):

| m × k × n | Isolated strict kernel gain | Assessment |
| --- | ---: | --- |
| 32 × 64 × 32 | 3.29–3.61× | Repeatable host-kernel gain |
| 64 × 65 × 64 | 4.08–4.23× | Repeatable host-kernel gain |
| 128 × 256 × 128 | 5.53–6.99× | Repeatable host-kernel gain |

These results do **not** prove SIMD machine instructions. The column-lane loop order can also improve caching, checks and compiler optimization without vectorization. GraalVM introduced eligible loop vectorization in 25.3, default enabled, but emission must be confirmed by IR/LIR and disassembly.

When temporary primitive packing and result materialization are included, the gain contracts:

| m × k × n | Full temporary-buffer proxy gain |
| --- | ---: |
| 32 × 64 × 32 | 1.11–1.12× |
| 64 × 65 × 64 | 1.35–1.38× |
| 128 × 256 × 128 | 1.47–1.68× |

Controls at 1×1 and 4×7×8 may regress; an eight-operation chain of simulated R2 packing did not establish persistent adaptive storage as uniformly superior.

## Protos baseline: NOT_STABLE

An observed Protos 32×64×32 baseline reported about 234309.9 ns with dispersion 0.2302, `NOT_STABLE`. Other compared observations (kernel, packing, materialization) also had large dispersion, roughly 0.2959–0.4583, and were `NOT_STABLE`. The baseline included reported GC batches absent from compared host-kernel samples.

**Forbidden inference:** divide unlike Protos guest and Java host timings to advertise a speedup. No stable end-to-end Protos comparison, guest Tier/IR parity, emitted SIMD instruction, or native-image parity is established.

## Source-level bottleneck that motivates the next diagnostic slice

[`ProtosStandardLinearAlgebraProtocol` at observed Protos revision](https://github.com/guillermomolina/protos/blob/3f37ef3fdd507504720bf30008317b6304ccffa6/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java) snapshots `Object[]`-backed Vector/Matrix values, then performs multiply/add through [`ProtosComplexArithmetic.canonicalNumberBinary`](https://github.com/guillermomolina/protos/blob/3f37ef3fdd507504720bf30008317b6304ccffa6/src/main/java/com/guillermomolina/protos/runtime/ProtosComplexArithmetic.java), annotated `@TruffleBoundary`. This prevents straightforward guest partial-evaluation across each canonical scalar operation, although Java host compilation of boundary methods remains possible.

The next **diagnostic-only** comparison is A1 (R1 Float-homogeneous primitive accumulation) versus T (R1 temporary `double[]` packing). It must distinguish baseline guest-end-to-end execution from standalone Java candidate proxies; neither proxy is a production A1/T Protos route.

## Pending proof and work state

- Inspect and preserve the three actual local I123 files before editing the next slice.
- Establish reproducible human-run full guest baseline, component costs, replicated stability and compilation/tier evidence; record `NOT_STABLE` when appropriate.
- Show emitted machine SIMD instructions **or** `SIMD_NOT_PROVEN`; test vectorization enabled/disabled only as an experimental comparator.
- Preserve D214-F1, full Number fallback, object identity, shape, mutation and Actor/P transfers.
- Do not close I123 before its diagnostic acceptance and revision-bound publication; do not close PLAT068 or select an accelerator architecture without separate exact owner approval.

`I123_LOCAL_TESTS=HUMAN_REPORTED_PASS`; `DIFF_CHECK=HUMAN_REPORTED_CLEAN`; `I123_PRODUCT_COMMIT=NOT_ESTABLISHED`; `PROTOS_E2E_SPEEDUP=NOT_PROVEN`; `SIMD_MACHINE_CODE=NOT_PROVEN`; `PLAT068_RATIFIED=NO`.
