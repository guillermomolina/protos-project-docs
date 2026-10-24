# PLAT068 — CPU Truffle/JIT follow-up and A1/T experiment approval (2026-10-24)

**Status:** pre-decision research checkpoint; **PLAT068 remains OPEN**. It approves a comparison agenda, **not** the CPU/GPU production backend, an R2 representation, a changed float policy, or a ratified platform decision.

- [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959); [I123 #960](https://github.com/guillermomolina/protos/issues/960); [D214 #955](https://github.com/guillermomolina/protos/issues/955).
- [Earlier twelve-dimension and CPU/GPU eligibility screen](PLAT068_PREDECISION_CPU_SIMD_GPU_RESEARCH_2026_10_24.md).
- [Human-run I123 measurement and validation checkpoint](../I123/I123_CPU_RUNTIME_JIT_MEASUREMENT_CHECKPOINT_2026_10_24.md).
- **Product source actually inspected:** [`3f37ef3fdd507504720bf30008317b6304ccffa6`](https://github.com/guillermomolina/protos/commit/3f37ef3fdd507504720bf30008317b6304ccffa6); human checkout may have different local/unpublished I123 files.
- **Approval provenance:** the project owner said “ok lo apruebo” on 2026-10-24 in response to four precise proposals: investigate A1 vs temporary T with Protos NORMAL/runtime/JIT evidence; leave R2/native/GPU open but unimplemented; retain D214-F1; finish the local I123 audit when its files are available. **No exact production architecture or semantic revision was selected.**

## Source architecture finding

In `ProtosStandardLinearAlgebraProtocol.java`, actual Vector/Matrix standard methods are native Closure slots, validated via ordinary dispatch. Operations clone object-element snapshots, call `ProtosComplexArithmetic.canonicalNumberBinary` for each multiply/add, and materialize fresh Vector/Matrix results. `canonicalNumberBinary`, `binary` and `conjugateNumber` carry `@TruffleBoundary`. The host compiler may compile methods separately; the Truffle guest compiler cannot freely partial-evaluate across those boundaries.

For Float×Float the canonical helper returns a new `ProtosFloatValue` after arithmetic on two `double` values. Hence per-scalar boundary calls, intermediate boxes, representation scans/copies and result materialization must be profiled **before** selecting a hardware acceleration path.

## Candidates for one controlled experiment

| Candidate | Design | Advantage to test | Semantic / cost gate |
| --- | --- | --- | --- |
| A0 | Existing R1 canonical path | Real Protos guest reference | Preserve unchanged |
| A1 | R1 object storage; homogeneous-Float guard; unboxed strict primitive accumulator; box output once | Eliminate scalar canonical boundaries and intermediate boxes | Same D214 Number, shape, snapshots, errors, first-product seed and signed zeros |
| T | R1, explicit temporary packed `double[]` and strict column-lane loops | Favor memory/locality and compiler vectorization | Guard + packing + computation + materialization must amortize |
| R2 | Persistent private adaptive primitive storage | May reuse packed state across operation chains | Not selected; mutation, generalized Number fallback and Actor/P graph copies |
| N/G | CPU native library or GPU compute | May help very large shapes | Not selected; strict numeric equivalence, packaging, failure and cost not demonstrated |

A1/T are **diagnostic candidates**, not approved product implementations. The immediate edit may touch only I123 test/diagnostic files; `src/main/**`, `protos/lib/**`, normative `spec/**` and build/version metadata remain unchanged.

## Semantic invariant reconciliation (GITHUB021)

1. C1 identity/ordinary slots/prototypes unchanged; no public dtype or Vector family.
2. Homogeneity is a private eligibility condition; all Small/Big Integer, Fraction, Complex and heterogeneous paths remain canonical.
3. Matrix shape/zero dimensions, physical volume checks and mutation/error ordering preserved.
4. Standard operation dispatch remains ordinary; any candidate applies only inside eligible operation.
5. D214-F1 remains exact: ascending inner `k`, first product is seed, zero-length contraction returns exact Integer 0; no FMA/reassociation/tree reduction.
6. `Vector.dot` conjugates its left operand; matrix operations are bilinear; generic Complex finite coordinates retain exact once-rounded arithmetic.
7. Fresh result identity, copy independence, frozen prototype, Context/Actor/P graph isolation remain fully observable.
8. JIT/JVM and Native Image must provide deterministic and semantically identical fallback, without upfront backend costs for non-users.

## Peer and compiler evidence already examined

- [GraalPy `DoubleSequenceStorage`](https://github.com/oracle/graalpython/blob/5bdcb9241c6e39a74b7a761969a7a0f953b2600f/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/sequence/storage/DoubleSequenceStorage.java): private `double[]`, boxing on demand; R2 precedent but no F1 proof.
- [GraalJS `ZeroBasedDoubleArray`](https://github.com/oracle/graaljs/blob/dd265d542fad72d0e7e1ae4032ef7c040911e700/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/array/dyn/ZeroBasedDoubleArray.java): primitive strategy and conversion to Object.
- [TruffleRuby `DoubleArrayStore`](https://github.com/truffleruby/truffleruby/blob/5b228e4625a27e49dd461ea5e9bba54d27dcbca4/src/main/java/org/truffleruby/core/array/library/DoubleArrayStore.java): primitive store specialization and generalization.
- [FastR `MatMult`](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.nodes.builtin/src/com/oracle/truffle/r/nodes/builtin/base/MatMult.java): `@Specialization` + `@CachedLibrary` + packed double data; its blocked accumulation and positive-zero seeding are **not** F1 compatible.
- [Graal compiler `LoopVectorizationPhase` version 25.4.4.1.1](https://github.com/oracle/graal/blob/graal-25.4.4.1.1/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/vector/phases/LoopVectorizationPhase.java): eligible array vectorization, guarded floating fold transformations.
- [Truffle host versus guest compilation](https://github.com/oracle/graal/blob/092eca6772a74d4ce501d5656f61cfb26745ce73/truffle/docs/HostCompilation.md): `@TruffleBoundary` is a guest partial-evaluation cutoff.
- [JDK 25 DoubleVector API](https://docs.oracle.com/en/java/javase/25/docs/api/jdk.incubator.vector/jdk/incubator/vector/DoubleVector.html): lane operations exist; reduction trees do not inherit D214-F1 permission.

## Required next evidence

The next I123 continuation should compare A0/A1/T at matched shapes, inputs and exact outputs and separate: guest dispatch, validation, homogeneous scan, snapshot, pack/unpack, kernel, materialization, object creation, allocation/GC and compiler tier. A1/T test-side proxies are **not** guest-integrated implementations; distinguish them from guest end-to-end baseline explicitly. Include small shapes, rectangular cases, chained reuse and mixed-number negative controls. Record stability, multiple runs, compiler IR, generated assembly when available, and on/off `-Djdk.graal.VectorizeLoops=false` comparisons. If machine instructions are absent from evidence, report `SIMD_NOT_PROVEN`.

Owner must make a further **exact candidate selection** after the evidence; that approval, GITHUB010/GITHUB021 reconciliation and durable platform decision would precede production optimization work. D214-F1 must not be silently reopened.
