# I123 — Post-I124 SIMD assessment and owner-approved deferral C (2026-10-25)

**Role:** revision-bound, non-normative investigation and decision evidence. **Owner:** [I123 #960](https://github.com/guillermomolina/protos/issues/960). **Related platform decision:** [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959). **Implemented baseline:** [I124 #963](https://github.com/guillermomolina/protos/issues/963).

**Owner decision:** on 2026-10-25, after receiving the bounded I123 investigation recommending **C — retain I124 and defer SIMD**, the maintainer explicitly answered **“apruebo C”**. This accepts a no-further-implementation disposition for the present evidence, **not** evidence that SIMD is absent, that it is emitted, or that speedup has been demonstrated. It does not authorize a new CPU/GPU/backend implementation.

## Exact product and repository provenance

- **Published product baseline:** [`guillermomolina/protos@c9843f0f82dbec6d0c626f35ec22f8f496ffe110`](https://github.com/guillermomolina/protos/commit/c9843f0f82dbec6d0c626f35ec22f8f496ffe110), Maven `0.3.402-SNAPSHOT`. I124's production R2-Float kernel and I123's diagnostic source were published together. [I124 revision-bound closure evidence](../I124/I124_402_R2_FLOAT_MATRIX_MATMUL_CLOSURE_2026_10_25.md).
- **Later observed remote main:** [`a6b67abdfcc031dad17acadc83e065c3a54d0bcd`](https://github.com/guillermomolina/protos/commit/a6b67abdfcc031dad17acadc83e065c3a54d0bcd), after the separate LIB025-A publication. This is a source observation, not a new SIMD run or a replacement for the pinned I124 baseline.
- **Normative contract:** `spec/semantics/VALUES_AND_COLLECTIONS.md`, Standard Library Vector and Matrix (D214-S1), strict Float reductions (D214-F1), at the [I124 product revision](https://github.com/guillermomolina/protos/blob/c9843f0f82dbec6d0c626f35ec22f8f496ffe110/spec/semantics/VALUES_AND_COLLECTIONS.md). No normative amendment is proposed or authorized here.
- **Audited implementation:** `src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java` and `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java`. The former is physically under `runtime/values/` but declares the `com.guillermomolina.protos.runtime` package.
- **Audited diagnostics:** `ProtosI123CpuKernelEligibilityTest.java`, `ProtosI123CpuKernelCandidateSupport.java`, `tools/diagnostics/Plat068CpuKernelProbe.java` and `tools/diagnostics/Plat068NumericPolicyProbe.java`. Repository root/scoped agent instructions and `pom.xml` were also consulted.

## Kernel facts and SIMD eligibility

The published private `floatMatmul(double[], double[], int, int, int)` uses row–column–ascending-`k` order. Every cell starts with its first binary64 product and performs subsequent multiply and add operations separately in ascending `k`; eligible all-Float matrices reuse persistent R2 `double[]` storage, and the result starts directly in R2. Empty contractions and non-Float Number combinations retain canonical generic behavior. The source contains **no explicit SIMD implementation**.

- **A — vectorizing within one cell's inner reduction:** a parallel partial-sum/horizontal reduction would violate D214-F1's association and strict FP rounding; it is not an eligible substitute.
- **B — vectorizing across independent output columns:** the test-only `strictColumnLanes` / `columnLaneMatmul` arrangement processes a row as ascending `k` followed by contiguous independent columns, preserving each cell's operation order. This is a concrete, *conditional* compiler opportunity, not production proof.
- **C — vectorized pack, copy or materialization:** possible independently of matmul but would not prove packed arithmetic in the production kernel.

The first-product seed, independent multiply/add rounding, no FMA, no reassociation, signed zero, NaN/Infinity and subnormal handling, exact empty Integer zero and generic Number/Complex fallbacks remain mandatory.

## Existing measurement evidence — explicitly separated

| Source | Timing/evidence domain | Proven limit |
| --- | --- | --- |
| `optionalEndToEndMeasurement` | `HOST_TO_PROTOS_PROTOCOL` plus `HOST_KERNEL` subseries | Calls `ProtosInvocation.invokeMessage` from Java; not a complete Truffle guest program |
| `optionalCandidateMeasurement` / `A0_PUBLIC_MATMUL` | `HOST_TO_PROTOS_PROTOCOL` | Public protocol from Java; no guest tier proven |
| Same method / `A1_R1_FLOAT`, `T_PACKED`, control | Test-local Protos carrier kernel and `HOST_KERNEL` | Candidate path and packing costs are not the current R2 production path |
| `Plat068CpuKernelProbe` and `Plat068NumericPolicyProbe` | `HOST_KERNEL` and host storage model | No guest speedup and no machine-code SIMD attribution |
| `COMPILER_EVIDENCE` | None established | No verified packed instructions tied to `floatMatmul` |
| `GUEST_END_TO_END` | None established | No stable full guest workload result |

The [2026-10-24 I123 measurement checkpoint](I123_CPU_RUNTIME_JIT_MEASUREMENT_CHECKPOINT_2026_10_24.md) records repeatable **isolated Java** `strictColumnLanes` gains over `strictCells`: approximately **3.29–3.61×** for 32×64×32 and **4.08–4.23×** for 64×65×64. Earlier cross-domain Protos comparisons were `NOT_STABLE`. These observations predate published persistent R2 and **are not** guest end-to-end speedups or proof of emitted SIMD.

## Compiler-access assessment and deliberately unexecuted measurement plan

[GraalVM 25.3 release notes](https://www.graalvm.org/release-notes/25.3/) describe enabled-by-default automatic vectorization of eligible array-arithmetic loops and the switch `-Djdk.graal.VectorizeLoops=false`. That ability does not prove admission of Protos's current loop. The [JDK 25 `-XX:CompileCommand=print` facility](https://docs.oracle.com/en/java/javase/25/docs/specs/man/java.html) could request targeted generated assembly for production `floatMatmul` and diagnostic `columnLaneMatmul`; readable disassembly may require `hsdis`, and method inlining can defeat a separately named compilation. No `hsdis` installation or Graal IR/IGV infrastructure was performed.

Two **proposed, not executed** human Surefire runs were bounded to approximately 15 minutes total: one normal GraalVM invocation, one with `VectorizeLoops=false`, using 32×64×32 for the existing protocol measurement and 64×65×64 for candidate paths, focused only on `optionalEndToEndMeasurement` and `optionalCandidateMeasurement`. `JDK_JAVA_OPTIONS` was selected to reach the Surefire fork despite the literal `pom.xml` `argLine`; Surefire 3.5.6 promotes Maven user properties by default. An on/off speed difference alone would still not establish SIMD instruction emission. Unstable results must stop as `NOT_STABLE`, without extra runs.

**No such runs were reported or performed for this decision.** No tests, compilation, benchmark, assembly capture, validation scripts, source edit or product Git publication occurred as part of this investigation.

## Approved decision and lifecycle boundary

**C — Keep I124 and defer SIMD implementation.** The present production kernel remains authoritative. The evidence fails to establish (A) that the kernel already emits packed SIMD arithmetic or (B) sufficient production-representative benefit to justify immediate column-lane implementation. This is a decision under uncertainty, **not a negative hardware capability finding**.

- Do **not** create a new `Ixxx`, replace `floatMatmul`, add Java Vector API, FMA, GPU/BLAS/native machinery, public dtype, or new runtime flags.
- Keep [I123 #960](https://github.com/guillermomolina/protos/issues/960) **open but paused**: the source diagnostics are published, yet its original stable guest baseline/attribution and compiler-evidence acceptance remain unsatisfied. Pausing does not falsify completion or delete those acceptance criteria.
- Keep [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) **open** with prior R2-Float ratification intact. Its SIMD branch is deferred; separate CPU matrix ISA/native/GPU questions are not decided by C.
- Reconsider a narrowly scoped R2 `floatMatmul` column-lane slice only upon new direct method-linked compiler evidence and/or stable protocol/guest cost attribution demonstrating non-marginal benefit, while retaining every D214-F1 invariant and obtaining a separate owner implementation approval.
- GITHUB021 consistency: **PASS for non-change**. The selection preserves all eight earlier C1/S1 constraints; no approved R2 behavior is superseded.

```ini
IDENTIFIER=I123
OWNER_DECISION=C
OWNER_APPROVAL_DATE=2026-10-25
PRODUCT_BASELINE=c9843f0f82dbec6d0c626f35ec22f8f496ffe110
SOURCE_FILES_CHANGED=NO
COMPILATION_TEST_BENCHMARK_RUNS=0
MEASUREMENT_PROPOSAL_MAX_RUNS=2
MEASUREMENT_PROPOSAL_BUDGET=15_MINUTES
SIMD_EVIDENCE_STATUS=NOT_PROVEN
PROTOS_GUEST_SPEEDUP=NOT_PROVEN
RECOMMENDATION=KEEP_I124_DEFER_SIMD
I123_LIFECYCLE=OPEN_PAUSED_ACCEPTANCE_INCOMPLETE
PLAT068_LIFECYCLE=OPEN
NORMATIVE_CHANGE=NONE
```
