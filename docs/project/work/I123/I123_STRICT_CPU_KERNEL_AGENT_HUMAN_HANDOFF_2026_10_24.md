# I123 — Strict CPU-kernel eligibility: editing agent and human executor handoff

**Status:** implementation instructions prepared, not implemented, not built, not tested. **Date:** 2026-10-24.

- [Formal I123 #960](https://github.com/guillermomolina/protos/issues/960) is the one grouped implementation work item; product repository `guillermomolina/protos`.
- [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) remains open; no CPU/GPU acceleration architecture selected.
- [PLAT068 research evidence](../../evidence/PLAT068/PLAT068_PREDECISION_CPU_SIMD_GPU_RESEARCH_2026_10_24.md) has the comparative packet, strict numerical witnesses, per-backend eligibility and upstream primary references.
- D214-S1 normative source: `spec/semantics/VALUES_AND_COLLECTIONS.md` at [product `798797a72ff3d292b52870da59f595c8607ca2d7`](https://github.com/guillermomolina/protos/commit/798797a72ff3d292b52870da59f595c8607ca2d7); no semantic edits permitted.
- Observed public main at allocation: `2174e88694d7b1c11b3c7132f827f09dc004c398`. The executor's actual local HEAD is authoritative; do not reset or force-update to this observation. LIB024 was not yet present on the inspected public main, but the owner's in-progress checkout may contain it.

## Editing agent: precisely bounded changes

**Type:** IMPLEMENTATION of tests and standalone diagnostics, not PLAT architecture implementation. **Repository:** `guillermomolina/protos` already checked out at actual local HEAD. Do not browse websites or other repositories, run builds, tests, validators, benchmark commands or state-changing Git operations.

1. Read local `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/PERFORMANCE.md`, `src/AGENTS.md` for tests, governing `spec/semantics/VALUES_AND_COLLECTIONS.md` D214, and the actual current LIB024 test/runtime files. Treat the fixed facts below as external research already completed; code inspection for exact local APIs is permitted and expected.
2. Create `tools/diagnostics/Plat068CpuKernelProbe.java`: standalone Java 25 source, no new build dependency, no runtime imports. Supply two independent-output-cell `double[]` matrix multiplication loops:
   - **scalar cells:** loops row, column, inner index; first inner product seeds result; subsequent terms add in ascending inner-index order;
   - **column lanes:** loops row, inner index, column, so each output cell's contributions still visit the same ascending inner index, enabling possible SIMD across independent columns.
   - Both **must** use separated multiply then add, no `Math.fma`, lane reduction, `--add-modules` or altered FP modes; preserve `-0.0`, subnormals, NaN and infinities. Control shape seed and arithmetic inputs exactly; verify outcomes by `Double.doubleToRawLongBits` for signed-zero witnesses and specify NaN payload limitations explicitly.
   - Include deterministic correct-output preflight before any timer; if numerical checks fail, no timing result. Optional `--check` mode for fast correctness-only execution. Emit raw labelled data for each shape, warmup/repetitions, scalar and column-lane kernel timing, reference-box packing/unpacking proxy timing, and platform identification. Do not pretend Java `Double[]` is Protos Number storage or a Protos end-to-end measurement.
   - Do not claim SIMD from source structure; print `SIMD_NOT_PROVEN` until compiled machine code is separately inspected by human executor.
3. Create `src/test/java/com/guillermomolina/protos/execution/protocols/values/ProtosI123CpuKernelEligibilityTest.java`, reusing the actual local LIB024 test pattern, `ProtosStandardLinearAlgebraProtocolTest` and `ProtosTestExecutionSupport` if available. Include shape, exact Number-family, F1 left-fold, Hermitian vector dot, bilinear matrix multiply, exact Integer zero empty contraction, Complex finite-coordinate, signing, noncoercion, mutability/identity and mixed Number fallback. A non-Float guest result must be compared through canonical semantic identity, never cast to `double`.
4. Bridge representative actual Protos `std:math/LinearAlgebra` results to the corresponding host diagnostic Float-only shapes without changing observable semantics. **No fake end-to-end comparison:** if LIB024 class/module is absent locally, stop with `BASELINE_UNAVAILABLE` before creating any stub or rewriting another work item.
5. Do not edit `src/main/`, `protos/lib/`, `spec/`, `pom.xml`, `CHANGELOG.md`, build configuration, runtime startup flags, P/Actor transfer or GPU/BLAS/Vector API code. Preserve dirty unrelated files. Respect project 30-Java-file directory guard.
6. At handoff list exact files, methods, witnesses, how to compile/check/measure and unexecuted validations. Do not claim PASS or speedup.

### Numerical contract to bake into files

- `[1e16, -1e16, 1]`: left fold 1; regrouping differs.
- `(1+2^-27)*(1-2^-27)-1`: separate rounded operations +0; FMA yields `-2^-54`.
- `-0.0 * +1.0` must seed the first nonempty contraction with negative zero.
- Empty contraction must yield exact Protos Integer 0, **not** Float +0.
- Include `Double.MIN_VALUE`, NaN, Inf, `Long.MAX_VALUE+1` BigInteger, exact Fraction, heterogeneous matrices.
- `dot([i],[i])` is Hermitian 1; matvec/matmul is bilinear and does not conjugate.
- Complex huge-coordinate finite-input products must follow once-rounded canonical coordinate arithmetic.
- Canonical private R1 remains unchanged; output identity, slots, copy, atPut/freeze and Actor/P transfers remain D214/LIB024 authorities.

### Upstream facts that the agent must use **without researching**

- GraalVM 25.3 compiler adds loop auto-vectorization for eligible array arithmetic; hypothesis toggled by `-Djdk.graal.VectorizeLoops=false`. This is an opt-in experimental comparison **of runtime flags**, not a product flag. Ref: https://www.graalvm.org/release-notes/25.3/
- Java 25 Vector API remains incubating; `Vector.reduceLanes(ADD/MUL)` floating reduction order is deliberately undefined. Protos F1 must never use it. Refs: https://openjdk.org/jeps/508 and https://docs.oracle.com/en/java/javase/25/docs/api/jdk.incubator.vector/jdk/incubator/vector/VectorOperators.html
- GraalPy's optional native extensions and NumPy arrays are a model for an explicit native boundary, **not** proof that heterogeneous Protos values are eligible: https://github.com/oracle/graalpython/blob/master/docs/user/Native-Extensions.md
- GraalJS TypedArrays are public typed objects (not allowed as replacement for ordinary D214 objects): https://github.com/oracle/graaljs/blob/master/docs/user/JavaScriptCompatibility.md
- TruffleRuby array storage strategies are an example of private adaptive representation (not permission for exposed Protos dtype): https://github.com/truffleruby/truffleruby/blob/master/CHANGELOG.md
- Espresso guest JVM mechanisms: https://github.com/oracle/graal/blob/master/espresso/docs/how-espresso-works.md
- Sulong executes LLVM IR; LLVM fast-math and vectorizer choices require strict FP audit: https://github.com/oracle/graal/blob/master/sulong/README.md and https://llvm.org/docs/Vectorizers.html
- SimpleLanguage reference anti-complexity baseline: https://github.com/graalvm/simplelanguage
- Pkl is not materially equivalent to D214 dense matrix numeric kernels: https://pkl-lang.org/main/latest/language-reference/index.html
- BLAS/NumPy/Julia, AMX/SME, CUDA/HIP/Vulkan remain research references only, no integration now. Links in PLAT068 evidence.

## Human executor: only after agent editing handoff

**No agent-run validation.** The maintainer chooses, executes and reports results. The first accepted validation block should capture:

```bash
git rev-parse HEAD
git status --short
git diff --check
javac -d target/i123-cpu tools/diagnostics/Plat068CpuKernelProbe.java
java -cp target/i123-cpu Plat068CpuKernelProbe --check
mvn -Dtest=ProtosI123CpuKernelEligibilityTest test
```

Commands are examples that become executable only if the agent implemented the described files and their mode; rerun no tests after any later metadata-only version change. `javac` and Java output should show the actual JDK/GraalVM version; if missing, report it. For timing, use the agent's documented bounded `--measure` interface only after correctness is green; capture raw warmup/repetition data and the effective CPU/JDK/GC/environment state. Contrasting `VectorizeLoops` on/off requires a new independent run and real compiler evidence before claiming SIMD. If no stable steady state, record `NOT_STABLE`.

Successful human checks alone do **not** authorize integration of any acceleration. Human owns product commit/push, while the authorized coordination agent may publish validated durable results in `docs/project/evidence/I123/`, linked back to PLAT068.

## Explicit outstanding gates

```ini
I123_ISSUE=960
TYPE=IMPLEMENTATION_TEST_ONLY
AGENT_FILE_EDITS=NOT_PERFORMED
AGENT_EXECUTED_TESTS=NO
HUMAN_TEST_RESULTS=PENDING
SIMD_MACHINE_CODE_EVIDENCE=PENDING
JVM_NATIVE_PARITY=PENDING
PLAT068_RATIFIED=NO
```
