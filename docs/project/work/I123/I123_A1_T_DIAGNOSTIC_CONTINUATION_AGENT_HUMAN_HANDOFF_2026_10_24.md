# I123 continuation — A1 versus temporary packing: editing agent and human-executor handoff

**Date:** 2026-10-24. **Work type:** IMPLEMENTATION, diagnostic/test-only, **not** PLAT068 ratification. **Repository:** `guillermomolina/protos`, human executor's checkout already at its actual HEAD. **Owning issue:** [I123 #960](https://github.com/guillermomolina/protos/issues/960). No new formal identifier or Issue: this is a bounded follow-on slice of existing I123. [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959) remains open.

**Exact owner approval:** investigate/measure A1 versus temporary T in NORMAL runtime context, retain R1 and D214-F1, defer R2/native/GPU. Approval is of diagnostic investigation **only**, not a production backend implementation.

## Editing agent: operative contract

**You are an editing agent, not the human executor.** Perform only source-file edits in `guillermomolina/protos`. Do not execute commands, builds, tests, validators, diagnostic Java programs, formatting commands, Git mutations, GitHub issue/API writes, network calls, other repositories, or web browsing. Do not write a further research essay: the external design facts required are specified below. If an exact local signature differs, inspect the actual local HEAD and adapt the bounded edit; if an essential fact conflicts with the approved semantics, stop and report the blocker.

Read the local `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/PERFORMANCE.md`, `src/AGENTS.md`, `AGENTS.work/REFERENCE.md` for license notice, the D214-S1 section of `spec/semantics/VALUES_AND_COLLECTIONS.md`, and only the directly affected LIB024/I123 source/tests and relevant local blocker-ledger entry. Do not re-research cross-language design. Inspect the precise current content and status of every owned path before editing: human local I123 sources may be staged/untracked and are **not** safely reconstructed from GitHub public main.

### Batch 1 — extend the existing three local diagnostic files only

**File 1: `tools/diagnostics/Plat068CpuKernelProbe.java`**

- Preserve existing `--check` / `--measure` modes, strict witness inputs, output labels and working kernels; augment without deleting or rewriting the human's diagnostics. If a requested method already exists, use the existing implementation and adjust only its missing measurement/correctness boundary.
- Add one labelled Java-host R1 boxed-Float *proxy*: hold `Object[]` of `ProtosFloatValue`-like boxing **without importing the Protos runtime** (use `Double` as explicitly labelled proxy; never claim `Double[]` has the same costs as guest Protos carrier), preflight every lane, perform strict first-product-seeded `double` arithmetic, and materialize boxed results. Time the eligibility scan, data access and materialization separately if measurement precision allows.
- Add the T proxy on exactly the same deterministically seeded data: eligibility → complete primitive `double[]` packing → strict `row,k,column` column-lane kernel with independent per-column accumulators → materialization. Record `PACK_NS`, `KERNEL_NS`, `MATERIALIZE_NS`, `TOTAL_NS` separately. Total timing must include scan and allocation. Do not exclude one-time setup from a method only when comparing it to the other.
- Keep existing strict scalar-cell, strict column-lane and blocked-column kernels as independent controls, including explicit result validation **before** timing. Preserve `k` ascending per cell, use first product as seed, no FMA, no positive-zero seed, no tree reduction and no reordering of one output cell's terms.
- Include 1×1, 4×7×8, 32×64×32, 64×65×64 and 128×256×128 or retain equivalent existing cases; identical seed/input across candidates. Include all-Float, a heterogeneity/failure-of-guard proxy with its final element ineligible, and operation-chain reuse with honest independent-copy/materialization costs. Never transform heterogeneous data into zeros or silently compare an admitted candidate to an ineligible reference.
- Preserve diagnostic output compatibility; add machine-readable `CASE`, `PATH`, `ELIGIBILITY`, `CORRECTNESS`, `DISPERSION`, `STABILITY`, `TIMING_DOMAIN=JAVA_HOST_PROXY` and `SIMD=NOT_PROVEN`. Explicitly label any host proxy; don't claim Protos A1/T implementation exists.

**File 2: `tools/diagnostics/Plat068NumericPolicyProbe.java`**

- Preserve prior policy cases and their existing output contract. If present, add missing exact counterexamples: FMA `(1+2^-27)*(1-2^-27)-1`, reassociation `[1e16,-1e16,1]`, negative-zero seeded contraction, subnormals, NaN/Infinity, distinct empty-contraction exact-integer rule. Use raw bits for observable signed zero and semantic classification for NaN; Java high-precision numerical references are diagnostic witnesses **not** Protos language semantics.
- Ensure `--check` fails decisively on a witnessed mismatch and never prints PASS after a failed check. Do not introduce new Float modes or alter D214-F1. If every case already exists, leave this file unchanged and report why.

**File 3: `src/test/java/com/guillermomolina/protos/execution/protocols/values/ProtosI123CpuKernelEligibilityTest.java`**

- Extend existing I123 bridge tests with an opt-in NORMAL runtime diagnostic fixture for actual Protos `std:math/LinearAlgebra` calls; preserve ordinary JUnit correctness tests and all existing assertions. Adapt to the local `ProtosStandardLinearAlgebraProtocolTest` helpers and `ProtosTestExecutionSupport`; do not invent missing APIs.
- One preflight must compare actual guest method results to strict expected Number/shape/identity semantics, including Float bit-conformance, empty exact Integer 0, BigInteger/Fraction/mixed fallback, Complex left-Hermitian dot vs bilinear matmul, signed zeros, separate result identity and copies. Never use tolerance comparisons as the admission criterion.
- For opt-in timing, gather repeated per-shape *actual protocol* `matmul` calls, including argument/dispatch/result costs, independent from any host proxy. Label `TIMING_DOMAIN=PROTOS_PROTOCOL` or `TIMING_DOMAIN=PROTOS_NORMAL_GUEST` **only when that path was actually exercised**. If the available fixture cannot make a genuine warmed guest call target, do not falsify that label: report `GUEST_TIER_NOT_PROVEN` and the actual protocol execution domain.
- Emit enough raw labelled samples and stable/unstable status, with no JUnit elapsed-time threshold assertion. Instrumentation must be dormant in ordinary NORMAL execution (and in ordinary JUnit conformance by default); activate only by an explicit test/diagnostic property.
- The guest baseline and Java-host A1/T proxies are different timing domains. Report them side by side for cost decomposition but **never** divide one by the other to claim a Protos speedup. No production fast path exists in this slice.

### Deliberately forbidden files and changes

No edits to `src/main/**`, `protos/lib/**`, `spec/**`, `pom.xml`, `CHANGELOG.md`, build scripts, version metadata, startup options, Actor/P transfer code, library API, bytecode root/DSL, native dependencies, Java incubator Vector API or GPU code. Do not rename/move/remove any existing local I123 source or modify unrelated dirty paths. Obey the 30-Java-file per-directory rule and preserve the exact project APL source-file notices.

### Hard semantical constraints

D214 C1 ordinary identity-bearing Vector/Matrix, heterogeneous current Number families, exact shape/zero dimensions, frozen standard prototypes, atPut error order and independent snapshots; dot conjugates its left operand only, matrices are bilinear; Complex finite coordinates use canonical once-rounded arithmetic. D214-F1 inner index ascends, the first product seeds any nonempty contraction, empty contraction returns exact Integer 0. No implicit coercion, FMA, reassociation or lane reduction across the contraction. Actor/P graph-aware copy and Context isolation are unchanged.

### External prior art: already researched; do not browse it

- **GraalPy `DoubleSequenceStorage`**: private primitive `double[]` and copying/boxing to object arrays; API example `getInternalDoubleArray()` and `getCopyOfInternalArray(...)`. Source: https://github.com/oracle/graalpython/blob/5bdcb9241c6e39a74b7a761969a7a0f953b2600f/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/sequence/storage/DoubleSequenceStorage.java. It motivates separate primitive tests, not R2 adoption.
- **GraalJS `ZeroBasedDoubleArray`**: primitive array strategy, `ArrayCopy.doubleToObject` on generalization; https://github.com/oracle/graaljs/blob/dd265d542fad72d0e7e1ae4032ef7c040911e700/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/array/dyn/ZeroBasedDoubleArray.java.
- **TruffleRuby `DoubleArrayStore`**: export library for primitive `double[]`, typed read and generalization; https://github.com/truffleruby/truffleruby/blob/5b228e4625a27e49dd461ea5e9bba54d27dcbca4/src/main/java/org/truffleruby/core/array/library/DoubleArrayStore.java. Do not copy any semantics that discard a signed zero.
- **FastR `MatMult.java`**: `@Specialization`/`@CachedLibrary`, readonly `double[]`, 64-wide blocked matrix kernel; its zero-initialized/blocked reductions are not D214-F1-compatible. https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.nodes.builtin/src/com/oracle/truffle/r/nodes/builtin/base/MatMult.java.
- **Espresso**: specialized double-array bytecode loads/stores, not ordinary Protos objects; https://github.com/oracle/graal/tree/master/espresso.
- **Sulong**: `LLVMDoubleVector` with primitive-vector semantics, not Protos Vector; https://github.com/oracle/graal/tree/master/sulong.
- **Pkl**: immutable persistent `VmList`, not mutable dense numeric tensor storage; https://github.com/apple/pkl/blob/939e844c032ab9b2acb972ff770978cc611674f2/pkl-core/src/main/java/org/pkl/core/runtime/VmList.java.
- **Graal compiler 25.4.4.1.1**: `LoopVectorizationPhase` option `VectorizeLoops=true`, eligible array arithmetic; compiler can reject floating-fold transformations. https://github.com/oracle/graal/blob/graal-25.4.4.1.1/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/vector/phases/LoopVectorizationPhase.java. Vectorization flag changes are human measurements only.
- **Truffle `@TruffleBoundary`**: host compilation remains possible, guest partial evaluation cuts off at annotated boundary; source https://github.com/oracle/graal/blob/092eca6772a74d4ce501d5656f61cfb26745ce73/truffle/docs/HostCompilation.md.
- **JDK25 Vector API**: `DoubleVector` supports independent double lanes, but `reduceLanes(ADD)` is not an F1 left-fold guarantee; https://docs.oracle.com/en/java/javase/25/docs/api/jdk.incubator.vector/jdk/incubator/vector/DoubleVector.html. Do not add module dependency.
- **NumPy/BLAS, Julia, oneMKL CNR, ReproBLAS**: may accelerate or reproduce reductions, but none proves identical output to D214-F1. No BLAS, JNI, FFM, C, CUDA, HIP, GPU or Native Image integration in this slice.
- **Relevant Protos physical source observed:** `ProtosStandardLinearAlgebraProtocol.java` does snapshots and one `ProtosComplexArithmetic.canonicalNumberBinary` per product/add, and that helper carries `@TruffleBoundary`. That is a hypothesis of overhead, not a measured guest JIT bottleneck.

### Editing-agent delivery

Report (1) exact owned files and method names changed; (2) guards and strict numerical witnesses added; (3) exact timing domains and emitted measurement fields; (4) compatibility with existing local I123 changes; (5) blocked/missing elements; (6) exact human-executor commands required. Declare explicitly `BUILDS=NOT_RUN`, `TESTS=NOT_RUN`, `BENCHMARKS=NOT_RUN`, `GIT_MUTATIONS=NONE` and `PRODUCTION_ACCELERATION=NO`.

## Human executor: commands only after edit handoff

Run commands in `guillermomolina/protos` actual local HEAD. **You**, not the editor, execute programs, validation, compilations, benchmarks, Git publication, and capture results. Start with the cheap stage, then focused checks; never run product tests after a later product metadata/version change.

**First command batch — inspect scope and whitespace (cheap):**

```bash
git rev-parse HEAD
git status --short
git diff --check
git diff --cached --check
```

If paths are unexpectedly modified, stop. The working tree need not be clean: unrelated user edits are preserved.

**Second command batch — after source edits, isolated correctness:**

```bash
javac -d target/i123-cpu tools/diagnostics/Plat068CpuKernelProbe.java tools/diagnostics/Plat068NumericPolicyProbe.java
java -cp target/i123-cpu Plat068CpuKernelProbe --check
java -cp target/i123-cpu Plat068NumericPolicyProbe --check
mvn -Dtest=ProtosI123CpuKernelEligibilityTest test
```

If the existing local numeric probe does not expose `--check`, run its documented human command instead; the editor must state the exact interface. No benchmarks if correctness is not green.

**Later separate measured batches:** only when correctness succeeds and an identical benchmark configuration is recorded. Perform independent cold/warm and ≥3 repeated sessions for guest baseline and Java-host candidate proxies; distinguish stability, CPU/GC, JVM/Native, compilation tier and code machine. The source alone is never proof of SIMD. For GraalVM comparison only, repeat an otherwise identical Java-host run with and without `-Djdk.graal.VectorizeLoops=false`. Do not run an expensive full suite until the change-impact rules require it.

After stable human evidence, decide whether I123 can publish/close or whether additional diagnosis is needed. A production A1/T candidate must go through the separate PLAT068 exact approval gate. No agent writes product commits or modifies Issue state from the editor phase.
