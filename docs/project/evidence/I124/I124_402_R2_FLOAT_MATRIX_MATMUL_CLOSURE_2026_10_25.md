# I124 — packed R2-Float Matrix and strict binary64 matmul (0.3.402)

**Record role:** revision-coupled implementation publication and closure evidence (non-normative). **Owning Issue:** [I124 #963](https://github.com/guillermomolina/protos/issues/963). **Approved architecture:** [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) and its [R2-Float decision](../../decisions/platform/PLAT068_R2_FLOAT_ADAPTIVE_MATRIX.md). **Related diagnostic:** [I123 #960](https://github.com/guillermomolina/protos/issues/960), separately tracked.

## Verified product publication

- **Repository:** `guillermomolina/protos`.
- **Exact product commit:** [`c9843f0f82dbec6d0c626f35ec22f8f496ffe110`](https://github.com/guillermomolina/protos/commit/c9843f0f82dbec6d0c626f35ec22f8f496ffe110), independently retrieved from GitHub on 2026-10-25.
- **Commit subject:** `I124: pack all-Float Matrix storage for strict binary64 matmul (0.3.402)`.
- **Maven version:** `0.3.402-SNAPSHOT`, verified in the published `pom.xml` diff; the published `CHANGELOG.md` includes the I124 entry.
- **Changed files (nine):**
  - `CHANGELOG.md`
  - `pom.xml`
  - `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java`
  - `src/main/java/com/guillermomolina/protos/runtime/values/ProtosMatrixValue.java`
  - `src/test/java/com/guillermomolina/protos/execution/ProtosI123CpuKernelCandidateSupport.java`
  - `src/test/java/com/guillermomolina/protos/execution/protocols/values/ProtosI123CpuKernelEligibilityTest.java`
  - `src/test/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocolTest.java`
  - `tools/diagnostics/Plat068CpuKernelProbe.java`
  - `tools/diagnostics/Plat068NumericPolicyProbe.java`

The human's publication included the previously local **I123 diagnostic files alongside I124**. Their publication SHA is therefore the same commit. This does not complete I123's independent performance/diagnostic acceptance or change PLAT068's remaining decision state.

## Implemented architecture and D214 reconciliation

1. **Private, lazy R1/R2:** `ProtosMatrixValue` uses a single volatile storage reference to either R1 `Object[]` or packed R2 `double[]`. Ordinary construction and transfer shells start in R1; all-genuine-Float inputs may pack once on admitted `matmul`. Packed results are born directly in R2 and can be reused in chained products without per-cell output boxing and repacking.
2. **Frozen first-packing:** a completely filled R2 array is published using a `VarHandle` compare-and-exchange. Concurrent first readers of a frozen all-Float Matrix converge on one persistent array. Packing does not change semantic elements, frozen status, slots, delegation, Matrix identity or shape.
3. **Mutation and fallback:** eligible Float `atPut` modifies a packed lane; a non-Float genuine Number reifies references and switches the same Matrix back to R1. Bounds, frozen and numeric-admission failures remain atomic. Indexed reads and snapshots materialize genuine independent Float objects; Actor/P transfer continues through independent snapshots and R1 shells.
4. **Strict Float kernel:** Float × Float `Matrix.matmul` uses host IEEE binary64 with first-product initialization and ascending contraction index, separately multiplying and adding; no `Math.fma`, reduction reassociation or partial/block sum. Empty contraction and other Number mixtures retain the D214-S1 generic behavior (including exact Integer zero).
5. **Guest API unchanged:** no exposed dtype, `Matrix.toFloat()`, global eager packing, mandatory external library, new GPU/SIMD backend, or revised normative semantics. The Java packed array accessor is internal trusted-kernel access; its backing must never be mutated or escaped by a consumer.

**Normative owner:** [`spec/semantics/VALUES_AND_COLLECTIONS.md` at the exact product revision](https://github.com/guillermomolina/protos/blob/c9843f0f82dbec6d0c626f35ec22f8f496ffe110/spec/semantics/VALUES_AND_COLLECTIONS.md). No normative specification file changed in this commit. FastR is a precedent for primitive backing and typed access only; its blocked, `0.0`-seeded accumulation was intentionally not copied.

## Human-executed validation and limits

- The maintainer explicitly reports **clean `git diff --check` and all local tests PASS** for the published I124 changes. This is a human-reported result; no local build, test command, or benchmark was run by the coordinating agent.
- A supplied earlier focal Maven transcript at version `0.3.401-SNAPSHOT`, before the metadata-only release bump, showed `BUILD SUCCESS`: **35 tests, 0 failures, 0 errors** — `ProtosI123CpuKernelEligibilityTest` 10/10 and `ProtosStandardLinearAlgebraProtocolTest` 25/25. It also showed successful compilation of production and test sources.
- The test changes exercise persistent packing and chaining, heterogeneous fallback and mutation, frozen and concurrent first-packing, snapshots/copy/other algebra, exceptional Float bit patterns and D214-F1 ordering. The I123 diagnostic bridge was updated to use Protos semantic identity rather than requiring Java reference identity for reified Float objects.
- The precise command/output of the final full-suite run, additional Actor/P regression totals, independent Native Image execution and machine-code disassembly were **not supplied as archived evidence**. The maintainer's broad PASS report is not represented here as an independently witnessed `make test` transcript.
- **No Protos end-to-end speedup, emitted SIMD instruction, Native Image parity or GPU benefit is established by this publication.** Performance follow-up remains in I123/PLAT068 rather than delaying this already bounded I124 implementation closure.

## Coordination outcome

**I124 implementation is published and eligible for formal closure**, with source revision, tests and evidence linked above. Keep [I123 #960](https://github.com/guillermomolina/protos/issues/960) **OPEN** for remaining measurement diagnostics and [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) **OPEN** for undecided SIMD/CPU matrix ISA/native/GPU paths. A public forced conversion, Integer/Vector packed representation, and hardware backends require separate authority.

```ini
IDENTIFIER=I124
FAMILY=I
PRODUCT_COMMIT=c9843f0f82dbec6d0c626f35ec22f8f496ffe110
PRODUCT_VERSION=0.3.402-SNAPSHOT
PRODUCT_CHANGED_FILES=9
FOCAL_TESTS=35_PASS_0_FAILURE_0_ERROR
FINAL_LOCAL_TESTS=MAINTAINER_REPORTED_PASS
DIFF_CHECK=MAINTAINER_REPORTED_CLEAN
NORMATIVE_SPEC_CHANGED=NO
FAST_R_COMPARE=REPRESENTATION_PRECEDENT_ONLY
PROTOS_E2E_SPEEDUP=NOT_PROVEN
SIMD_GPU_BACKENDS=NOT_APPROVED
I123_DIAGNOSTICS=PUBLISHED_IN_SAME_PRODUCT_COMMIT_STILL_OPEN
```
