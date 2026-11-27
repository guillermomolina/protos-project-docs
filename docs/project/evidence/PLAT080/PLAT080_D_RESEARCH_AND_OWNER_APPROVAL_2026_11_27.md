# PLAT080 — Selective Protos-first Vector and Matrix: research and approval evidence

Date: 2026-11-27. Work item: [PLAT080 #1212](https://github.com/guillermomolina/protos/issues/1212); umbrella [AUD025 #1209](https://github.com/guillermomolina/protos/issues/1209).
Inspected product revision: [509221a9d28b79e9da9accf721fad2358d81ec49](https://github.com/guillermomolina/protos/tree/509221a9d28b79e9da9accf721fad2358d81ec49).
Owner approval in the active interaction: “ok apruebo la recomendacion.” This expressly selects alternative D as presented, including its feasibility conditions, not any new runtime mechanism. The owner also explicitly authorized direct documentation publication for this task. No fabricated GitHub owner-authored approval comment.

## Source evidence

Paths below refer to the exact product revision above. The module initializer `protos/lib/math/linear_algebra/module.protos` only publishes the canonical prototypes. `src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardLinearAlgebraProtocol.java` installs native methods and implements the mathematical loops; `installVector`, `installMatrix`, `accumulate`, `reduced`, `floatMatmul` are concrete loci. `runtime/values/ProtosVectorValue.java` and `ProtosMatrixValue.java` (under the same Java package root) own private storage, indexed reads/writes, snapshots and transfer shells. Matrix uses `readonlyPackedFloatsForRuntime` and `ownedPackedFloatsForRuntime`; the former uses atomic publication for frozen sharing. `runtime/ProtosComplexArithmetic.java`: `canonicalNumberBinary` and `conjugateNumber` are currently @TruffleBoundary and do not send guest messages. `execution/host/embedding/ProtosCoreBootstrap.java` installs and freezes canonical prototypes and provides them to the module.

Relevant internal precedents: `protos/lib/core/Complex.protos`, `Number.protos`, `network/IpAddress.protos`, `ProtosStandardComplexProtocol.java` under execution/protocols/numeric; source closures and private native kernels before freezing. `protos/lib/math/integer/module.protos` contains guest mathematical algorithms. `protos/lib/collections/array/module.protos` slice contains a balanced construction of a fixed-size Array using spread; this does not prove a direct canonical payload access mechanism or an efficient packed result builder.

Normative sources inspected: `spec/semantics/VALUES_AND_COLLECTIONS.md` D214-S1/F1 and numeric model, `CALLABLES.md` receiver binding, `OBJECT_MODEL.md` reflection; module and Actor/P authority identified. Root AGENTS, DESIGN, REFERENCE, spec/src/protos scoped instructions consulted. Durable authorities: [D214](https://github.com/guillermomolina/protos/issues/955), [I124](https://github.com/guillermomolina/protos/issues/963), [I172](https://github.com/guillermomolina/protos/issues/1161), [PLAT068](https://github.com/guillermomolina/protos/issues/959), [AUD025 plan](../../work/AUD025/AUD025_STANDARD_LIBRARY_PROTOS_FIRST_AUDIT_AND_PHASE_PLAN.md). Test evidence inspected: `protos/tests/library/math/linear-algebra/algebra-and-reductions.protos`, `identity-and-state.protos`, `ProtosStandardLinearAlgebraProtocolTest.java`. In particular redefining size does not change standard dot's result. These are source/test inspections, not newly executed tests.

## Comparative implementation evidence

- GraalPy [SequenceStorageNodes](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/common/SequenceStorageNodes.java): GetItemScalarNode, SetItemScalarGeneralizingNode (@GenerateInline, @Specialization), CopyNode. [MathModuleBuiltins](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/modules/MathModuleBuiltins.java): SumprodNode, ProdNode.doGeneric; sumprod's extended precision is not D214-F1. [PyNumberAddNode](https://github.com/oracle/graalpython/blob/21b9ad46ab3de0ee63960926e51b054e209fd3e3/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/PyNumberAddNode.java) @OperationProxy.Proxyable.
- GraalJS [TypedArrayPrototypeBuiltins](https://github.com/oracle/graaljs/blob/df699e927a4e11b2feffda4630b3682e42826aa2/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/builtins/TypedArrayPrototypeBuiltins.java): CopyTypedArrayElementsNode.execute; public typed-array representation differs.
- TruffleRuby [ArrayStoreLibrary](https://github.com/truffleruby/truffleruby/blob/5d9b60ffb2229012c8dfb10cc70695e9ab3230d3/src/main/java/org/truffleruby/core/array/library/ArrayStoreLibrary.java), @DefaultExport typed stores; Ruby [Matrix library](https://github.com/ruby/matrix/blob/master/lib/matrix.rb) illustrates guest algorithms, not exact Protos semantics.
- Espresso [Target_java_lang_System](https://github.com/oracle/graal/blob/6539700e7a590d726992b21c3797e7ab9f6f992b/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/substitutions/standard/Target_java_lang_System.java): specialized doLongArray/doDoubleArray; internal copying.
- Sulong [LLVMVectorArithmeticNode.doDouble](https://github.com/oracle/graal/blob/6539700e7a590d726992b21c3797e7ab9f6f992b/sulong/projects/com.oracle.truffle.llvm.runtime/src/com/oracle/truffle/llvm/runtime/nodes/op/LLVMVectorArithmeticNode.java): @Specialization @ExplodeLoop; fixed LLVM vector lengths, not dense arbitrary-size guest containers.
- FastR [MatMult](https://github.com/oracle/fastr/blob/f9d9199e51b371f076b19456933ee395d28360bf/com.oracle.truffle.r.nodes.builtin/src/com/oracle/truffle/r/nodes/builtin/base/MatMult.java): multiplyDouble @Specialization and MatMultAsDouble; different reduction and coercion contracts.
- SimpleLanguage [SLAddNode](https://github.com/graalvm/simplelanguage/blob/5a2b35790539ab1fe5c61f02b71e3485fd0e0fb1/language/src/main/java/com/oracle/truffle/sl/nodes/expression/SLAddNode.java): doLong @Specialization(rewriteOn = ArithmeticException.class), doSLBigInteger @TruffleBoundary.
- Pkl [ListNodes](https://github.com/apple/pkl/blob/c254bfbabeaed4d013009a7a3609a7f09d3eafb8/pkl-core/src/main/java/org/pkl/core/stdlib/base/ListNodes.java): fold.eval @Specialization; callback-based fold, not canonical non-dispatch contraction.
- Julia [LinearAlgebra.jl](https://github.com/JuliaLang/LinearAlgebra.jl) illustrates guest generic algorithms and specialized backends. Non-Truffle references inform architecture but cannot authorize Java changes.

## GITHUB010 comparison

A retain current implementation, B fully guest-backed representation (eliminated: conflicts with approved private payload and storage constraints), C broad hybrid migration, D selective migration subject to established mechanisms. Qualitative scores are not AUD025 acceptance metrics.

| Axis | A | C | D |
|---|---|---|---|
| Correctness | 5 existing | 4 reconciliation | 5 bounded/conditional |
| Protos alignment | 3 host algorithms | 5 guest algorithms | 5 natural split |
| Present proportionality | 5 no new cost | 3 possible machinery | 5 cost conditioned |
| Incremental growth | 3 host evolution | 5 guest extension | 5 incremental |
| Future options | 4 reversible | 5 separation | 5 retained options |
| Scalability | 4 known storage | 4 lowering dependent | 4 retain kernels |
| Simplicity | 4 established | 3 added boundary | 4 reuse |
| Portability | 3 Java coupled | 5 guest algorithms | 4 partial migration |
| Runtime cost | 4 known path | 3 unverified | 4 avoid forced changes |
| Failure/operability | 5 existing | 3 interactions | 4 bounded |
| Reversibility | 5 unchanged | 3 broad migration | 5 local |
| Evidence maturity | 5 implemented | 3 hypotheses | 4 partial precedents |

Confidence HIGH in existing implementation facts and B's incompatibility; MEDIUM for D as a policy; MEDIUM/LOW for C and unproven lowering/cost. Scores are conditional judgments, not proof.

Counterexamples: overridden size/at/rows/columns must not intercept standard algorithms; Number and Vector impostors rejected; Integer overflow must promote exactly; mixed and Complex values retain canonical arithmetic; negative zero and NaN/infinity/subnormal results, first-product seed and ascending k; zero-volume shapes with enormous dimensions must not iterate an empty axis; fresh open results must not alias; suspended each retains snapshot; Actor/P preserves cycles, aliases and isolation; frozen Matrix packing remains safe across readers. No public dtype, shape change, implicit scheduling or hardware backend.

Strongest objection: translating short native loops may require more private Java bridge machinery, boxing and dispatch than it removes. D therefore retains an operation when an established exact non-dispatch path is unavailable. No performance campaign or deletion target. Deferral is reversible without public API or representation change; future generic algorithms can be migrated after existing mechanisms support them. No speculative scheduler, OS-specific or AOT mechanism.

## Limits and gate

The source-first direction is owner-approved. Whole-document normative secondary reads and full feasibility reconciliation were not completed in the preceding investigation; no concrete guest-callable canonical payload access and specialized no-message lowering is yet demonstrated. Existing internal bootstrap and storage examples are precedents, not a verified equivalent for a new Java bridge. GITHUB021 design-level invariants are preserved by the conditional direction; per-operation executable/implementation consistency is pending. No new issue identifier or I allocation until an actual viable bounded candidate is established. No product edits, builds, tests, benchmarks, compiler captures, version or spec changes. Keep PLAT080 open; AUD025 B remains incomplete. No closure or full ratification claimed.
