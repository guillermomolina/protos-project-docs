# PLAT068 — CPU SIMD, matrix ISA and GPU eligibility for D214 (pre-decision evidence)

- **Record type:** investigation evidence / OPEN. **Not** an approved architecture, an implementation completion, a benchmark result, or a production performance claim.
- **Date:** 2026-10-24.
- **Live owner:** [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959), `family:PLAT`, status ready; no candidate ratified.
- **Normative owner:** [D214 #955](https://github.com/guillermomolina/protos/issues/955); specification at [Protos `798797a72ff3d292b52870da59f595c8607ca2d7`](https://github.com/guillermomolina/protos/commit/798797a72ff3d292b52870da59f595c8607ca2d7) / spec `0.1.470`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, D214-S1. [Durable D214 decision](../../decisions/language/D214_VECTOR_MATRIX.md).
- **Inspected moving-main observation (not a reproducible runtime):** [Protos `2174e88694d7b1c11b3c7132f827f09dc004c398`](https://github.com/guillermomolina/protos/commit/2174e88694d7b1c11b3c7132f827f09dc004c398); `ProtosVectorValue.java` not yet available at that GitHub revision; [LIB024 #957](https://github.com/guillermomolina/protos/issues/957) remains a separate implementation owner. Do not infer the maintainer's newer local worktree state.
- **Evidence method:** source/spec/docs examination and external primary-source review only. **No** compilation, Protos run, measurement, CPU disassembly, JVM IR capture, GPU execution, numerical conformance run, or environment capability probing was performed. The assessment is falsifiable and deliberately provisional.

## Immutable D214-S1 limits (all remain in force)

1. Values are ordinary identity-bearing mutable/frozen Vector/Matrix objects with frozen standard prototypes and private fixed-dense indexed payload, not new Core types.
2. Admit genuine `Number`: SmallInteger, BigInteger, Fraction, Float and Complex, heterogeneous in the same container. No implicit homogeneous primitive coercion.
3. Exact fixed shape, integer index/bounds/freeze/atPut contract; no growth and no publicly observable `dtype`.
4. The only approved public operations are Vector `add, scale, dot, copy` and Matrix `add, scale, matvec, matmul, transpose, copy`, with canonical numeric arithmetic and no callbacks introduced.
5. **F1:** for each contraction, visit increasing inner index, compute each product and then add left-to-right; the first product seeds a nonempty contraction; an empty contraction is exact Integer zero. Never reassociate, fuse, reorder, use lane-reduction trees, or change rounding. Independence across separate result cells is not permission to reorder within a cell.
6. `Vector.dot` is conjugate-linear in its left operand; Matrix multiplication and matvec are bilinear. Complex finite-coordinate results respect D214 once-rounded arithmetic.
7. R1 reference storage is initial; R2 adaptive packing is merely a future **invisible** implementation option, gated by proof. Ordinary identity/slots/overrides/mutation remain observable as approved.
8. Actor/Process/P boundaries require graph-aware copy/isolation and correct cycle/alias publication, independent of physical storage.

## Hardware and numerical eligibility findings

| Operation / backend pattern | Status before proof | Reason / potential eligible shape |
| --- | --- | --- |
| Scalar R1 protocol using canonical arithmetic | PROVABLY_EQUIVALENT by construction **if** its implementation follows D214 | Semantic baseline; numerical performance not yet measured. |
| Independent Float output cells using primitive CPU lanes | CONDITIONALLY_EQUIVALENT | Can process *different cells* simultaneously while preserving each cell's sequential operation order; admission, materialization, exact numerical behavior and publication need proof. |
| Homogeneous Float `add`, `scale` lane-wise CPU SIMD | CONDITIONALLY_EQUIVALENT | Per-element arithmetic is independent, but canonical float/Complex paths, signed zero, NaN, subnormals, exceptional values and no coercion need differential proof. |
| `dot`, `matvec`, `matmul` SIMD across **inner reduction index** / Java Vector `reduceLanes(ADD)` | INELIGIBLE under F1 | Changes addition association/order (the Java 25 API explicitly leaves FP lane reduction order unspecified). |
| CPU BLAS GEMM, implicit fused multiply-add, strict-unsafe AMX/SME tensor tiling | INELIGIBLE as a generic replacement | Contracted sums routinely reassociate/fuse and many tile instructions use narrower operand formats. A strict cell kernel is a distinct unproven candidate. |
| x86 SSE/AVX / Arm NEON/SVE auto-vectorized independent lanes | CONDITIONALLY_EQUIVALENT | Available ISA alone proves nothing; inspect actual emitted instructions, IEEE modes, actual supported hardware, guards and fallback. |
| GPU kernel assigning one independent output cell per lane and sequential F1 inside each cell | NOT_YET_PROVEN | Plausible strictly ordered conceptually; device FP64, FMA disabling, denormals/FTZ, Complex arithmetic, data transfer cost, driver lifecycle, cancellation/isolation/AOT all unknown. |
| CUDA/ROCm BLAS, matrix/tensor-core libraries, relaxed GPU shader floating point | INELIGIBLE as a generic substitution | Default numerical contract is not D214's strict left-fold, exact integer/Fraction and Complex mixed family semantics. |
| Integer, BigInteger, Fraction, mixed Number generic hardware vector path | NOT_YET_PROVEN / default INELIGIBLE | No lawful blanket Double conversion; arithmetic overflow promotion/exact Fraction and genuine receiver/selector behavior require canonical dispatch. |

**Important distinctions:** `PROVABLY_EQUIVALENT` above is a semantic observation about an unchanged reference baseline, **not** a certification of a particular current implementation. No proposed accelerated route is currently approved or verified. All unproven paths must fall back to canonical scalar R1 behavior.

### Adversarial witnesses — bitwise / identity-based, not approximate tolerance

1. `[1e16, -1e16, 1]` left fold yields `1`; regrouping may yield `0`.
2. `(1 + 2^-27) * (1 - 2^-27) - 1`: separate rounded multiply then add yields +0; a fused operation yields `-2^-54`. Reject silent FMA.
3. First product `(-0.0) * 1.0` must seed the fold at `-0.0`; initializing with `+0.0` may change the sign.
4. Empty contraction yields exact `Integer 0`, not `Float 0.0`.
5. `Double.MIN_VALUE * 1.0` must not flush to zero. Include signed zeros, all NaN paths, infinities, overflow/underflow, and finite cancellation.
6. `Long.MAX_VALUE + 1` promotes to exact BigInteger, and Fraction operations remain exact; casting to Double breaks the Number contract.
7. `dot([i],[i]) == 1` because its left operand is conjugated; matrix products **do not** conjugate.
8. Large-coordinate Complex example: `(1e308+i*1e308)*(1e308+i*1e308)` demands the canonical once-rounded finite-coordinate result, not naive `Infinity-Infinity => NaN`.
9. Preserve arbitrary container aliasing and subsequent `atPut`, freeze, slots and Actor/P transfer semantics for every admitted optimized input.

## Primary-source comparison (GITHUB010 mechanism-level survey)

| Implementation / platform | Material precedent | Limitation for D214 |
| --- | --- | --- |
| GraalPy / NumPy | Polyglot Python + native extension and array-computation boundary; optional homogeneous native arrays | NumPy native accumulation rules are not D214's sequential heterogeneous Number contract; interop boundary is not automatically transferable. |
| GraalJS | Typed Array / ArrayBuffer as a **separate public typed representation** | Protos D214 explicitly refuses exposed dtype/typed value family; mechanism is comparative only. |
| TruffleRuby | Array representation strategies, specializations and general-value fallback | Supports considering invisible storage changes, but Ruby float order and container semantics cannot be imported as Protos rules. |
| Espresso | Guest JVM implementation and Java execution infrastructure | Demonstrates Truffle guest/JVM boundary; not evidence of a D214-compliant accelerator. |
| Sulong/LLVM | Native LLVM execution and vector instruction lowering | LLVM permits FP reassociation under relevant fast-math modes; native interop introduces AOT/ownership/portability costs. |
| SimpleLanguage | Minimal Truffle language/compiler specialization baseline | No equivalent public vector/matrix accelerator; useful only as anti-complexity baseline. |
| Pkl | Configuration language and JVM-oriented typed evaluation | No materially comparable guest-language dense mixed-Number strict-matrix backend; survey recorded, excluded from scoring. |
| OpenJDK/GraalVM compiler | GraalVM 25.3 added default auto-vectorization for eligible array arithmetic loops; Java Vector API permits explicit lane-wise SIMD | Actual D214 loops, current GraalVM version, strict Java FP and Truffle PE must be inspected, not presumed vectorized. |
| NumPy, Julia, BLAS, OpenBLAS | High-throughput densely packed numerical kernels, caller-visible numerical/element representations | A speed/performance precedent, **not** semantic proof for Protos mixed Number or F1. |
| CUDA, cuBLAS, Vulkan, CPU AMX/Arm SME | Device/tile/lane compute for large homogeneous numerical workloads | Strict per-cell accumulation, FP64, driver availability, transfer break-even, Native Image/OS portability and context ownership remain open. |

Truffle comparators are mechanism analogies, not claims that each language has a corresponding D214 operation.

## GITHUB010 12-dimensional provisional screening

Scale 1 (poor) to 5 (strong). Each entry is `score / confidence — reason`. **Architecture is not selected;** these are pre-measurement estimates, so future benchmark/proof evidence can change them.

| Required dimension | A: preserve R1 scalar | B: invisible packed CPU lanes | C: native CPU BLAS | D: strict GPU cell kernel | E: eager hybrid CPU/GPU |
| --- | --- | --- | --- | --- | --- |
| 1. Correctness / invariants | 5/H — canonical path | 3/L — guard + bit proof | 1/H — generic F1 mismatch | 2/L — strict GPU unproven | 2/L — multiple unproven paths |
| 2. Protos alignment | 5/H — ordinary objects | 4/M — private optional path | 2/H — host library machinery | 2/M — device machinery | 1/M — broad backend manager |
| 3. Pay as you need | 5/H — zero additions | 4/M — optional packed path | 2/H — FFI/dependency | 1/H — drivers and copies | 1/H — all backend costs |
| 4. Incremental growth | 4/H — private R2 possible | 5/M — staged invisible paths | 3/M — library specialization | 3/M — future device seam | 3/L — complex from outset |
| 5. Future-option resilience | 4/H — no public ABI | 5/M — guard/fallback | 3/M — ABI footprint | 3/L — hardware footprint | 3/L — policy locks choices |
| 6. Scalability | 2/M — scalar compute | 4/L — cell lanes | 5/M — dense compute | 5/L — big-device compute | 5/L — candidate only |
| 7. Conceptual simplicity | 5/H — single rule | 4/M — one optional route | 2/H — host ABI | 1/M — device lifecycle | 1/H — policy matrix |
| 8. Portability / freedom | 5/H — Java / guest | 4/M — ISA fallback | 2/H — library/OS binding | 1/H — driver ecosystem | 1/H — combined constraints |
| 9. Runtime / resources | 2/M — potential CPU cost | 4/L — pack/threshold unknown | 3/L — fast large, FFI | 2/L — transfer break-even unknown | 2/L — dispatch plus staging |
| 10. Failure / operability | 5/H — no new failure | 4/M — local invalidation | 2/M — native loading/failure | 1/M — devices/cancel | 1/M — combined errors |
| 11. Deferral / migration | 5/H — private future change | 5/M — reversible internal route | 2/M — difficult ABI exit | 2/L — difficult portability | 2/L — multiple migrations |
| 12. Evidence maturity / risk | 5/H — semantic baseline | 3/L — auto-vectorization exists, product proof absent | 3/M — mature tools, wrong contract | 1/H — no product proof | 1/H — no product proof |

**Hard veto precedes sums:** candidates cannot compensate semantic ineligibility with throughput. Hence the table must **not** be interpreted as a weighted-winner decision.

## Incremental decision and falsification boundary

The smallest immediate act is **no production acceleration**. Measure whether an invisible CPU-only fast path actually helps enough to offset conversion and dispatch. A positive result would justify returning to the project owner for explicit **PLAT068** selection, not automatically enabling it.

**Next bounded measurement/edit-only experiment proposal, not approved accelerator deployment:**
- Work in `guillermomolina/protos` on the executor's actual current HEAD; do not assume unpublished LIB024 content exists on GitHub. Retain R1 unchanged.
- Add isolated diagnostics under `tools/diagnostics/` or test-only machinery. Compare A: semantic Protos Vector/Matrix R1 (once published), B0: temporary strict `double[]` scalar loops, B1: strictly lane-independent output-cell CPU parallelism/auto-vectorization. Do **not** call `Vector.reduceLanes` for F1. Do not add GraalVM Vector incubator module to production or change public API.
- Compare real end-to-end Protos call cost and separate pure-loop compute cost; record construction, packing, number materialization, dispatch and output copying **separately**. Include tiny, medium and large rectangular matrices, varying contraction length; mixed-Number must take fallback.
- Repeat JVM and Native only if both are available and correctly identified, fixed toolchain/CPU features/GC/heap/warmup; capture actual compiler evidence. Contrast `-Djdk.graal.VectorizeLoops=false` and enabled as a controlled hypothesis, never assume generated SIMD.
- Correctness gate **before timing**: full D214 index/identity/isolation and raw-bit Float/F1 witnesses listed above; exact BigInteger/Fraction and Complex fallback. No timed result is admissible without proven outcome identity.
- Retain reproducible harness commands, source SHA, outputs, warmup/steady-state protocol and explicit confidence. The human executor (not editing agent) runs compilation/tests/benchmarks and product Git actions. No production change, version bump, or CHANGELOG edit is implied by this investigation.

**Stop conditions:** actual current HEAD lacks usable LIB024 baseline, any semantic discrepancy, unsupported hardware/JDK, failing validation, inconclusive compilation shape, unstable steady state, or matrix kernels accidentally using reassociating/fused reductions. Report these; do not silently weaken F1 or broaden to GPU.

## Prior-art and API links (primary)

- [GraalVM 25.3 compiler loop vectorization](https://www.graalvm.org/release-notes/25.3/), [GraalVM 25.4 notes](https://www.graalvm.org/release-notes/25.4/), [GraalVM on JDK 25](https://www.graalvm.org/release-notes/JDK_25/).
- [JEP 508 — Vector API](https://openjdk.org/jeps/508), [Java 25 Vector reductions](https://docs.oracle.com/en/java/javase/25/docs/api/jdk.incubator.vector/jdk/incubator/vector/Vector.html), [Java 25 floating-point VectorOperators](https://docs.oracle.com/en/java/javase/25/docs/api/jdk.incubator.vector/jdk/incubator/vector/VectorOperators.html).
- [LLVM loop vectorizer](https://llvm.org/docs/Vectorizers.html), [LLVM LangRef fast-math](https://llvm.org/docs/LangRef.html).
- [GraalPy native extensions](https://github.com/oracle/graalpython/blob/master/docs/user/Native-Extensions.md), [GraalJS JavaScript compatibility](https://github.com/oracle/graaljs/blob/master/docs/user/JavaScriptCompatibility.md), [TruffleRuby implementation history](https://github.com/truffleruby/truffleruby/blob/master/CHANGELOG.md), [Espresso internals](https://github.com/oracle/graal/blob/master/espresso/docs/how-espresso-works.md), [Sulong](https://github.com/oracle/graal/blob/master/sulong/README.md), [SimpleLanguage](https://github.com/graalvm/simplelanguage), [Pkl language](https://pkl-lang.org/main/latest/language-reference/index.html).
- [NumPy matmul](https://numpy.org/doc/stable/reference/generated/numpy.matmul.html), [Julia LinearAlgebra](https://docs.julialang.org/en/v1/stdlib/LinearAlgebra/), [OpenBLAS](https://github.com/OpenMathLib/OpenBLAS), [grCUDA](https://github.com/NVIDIA/grcuda), [FastR](https://github.com/oracle/fastr).
- [NVIDIA NVCC `--fmad`](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/), [cuBLAS reproducibility](https://docs.nvidia.com/cuda/cublas/index.html), [NVIDIA PTX ISA](https://docs.nvidia.com/cuda/parallel-thread-execution/contents.html), [Intel AMX](https://www.intel.com/content/www/us/en/developer/articles/technical/tuning-guide-for-ai-on-the-4th-generation.html), [Arm SME](https://developer.arm.com/community/arm-community-blogs/b/architectures-and-processors-blog/posts/arm-scalable-matrix-extension-introduction-p2), [Vulkan float controls](https://docs.vulkan.org/refpages/latest/refpages/source/VK_KHR_shader_float_controls.html).

## Pending PLAT068 approval packet

Before ratification: empirical cost/IR evidence, exact eligibility proof per operation and type family, guarded deoptimization/fallback semantics, cross-context/Actor/P copying, F1 bit-level suite, JVM/Native Image proof, OS/device packaging and failure policy, AOT initialization and security/resource lifetime, and explicit owner approval of *one* bounded platform candidate. **No PLAT068 decision record or platform registry entry is justified yet.**
