# LIB024 — D214 C1/S1 Vector and Matrix Standard Library: 0.3.401 publication

Date: 2026-10-24. This is revision-bound, non-normative implementation closure evidence for [LIB024 #957](https://github.com/guillermomolina/protos/issues/957). The ratified semantic authority remains [D214 #955](https://github.com/guillermomolina/protos/issues/955), its [eight-point owner approval](https://github.com/guillermomolina/protos/issues/955#issuecomment-6380265698), and the [published D214-S1 values specification](https://github.com/guillermomolina/protos/blob/798797a72ff3d292b52870da59f595c8607ca2d7/spec/semantics/VALUES_AND_COLLECTIONS.md).

## Exact publication identity

```text
WORK_ITEM=LIB024
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=d86f587beeefa413e323d345f139ccbb164462ca
PRODUCT_COMMIT=LIB024: add std:math/LinearAlgebra Vector and Matrix (0.3.401)
IMPLEMENTATION_VERSION=0.3.401-SNAPSHOT
PRODUCT_CHANGED_PATH_COUNT=26
NORMATIVE_SPEC_REVISION=0.1.470
NORMATIVE_SPEC_CHANGE_IN_THIS_COMMIT=NO
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The exact [published product commit](https://github.com/guillermomolina/protos/commit/d86f587beeefa413e323d345f139ccbb164462ca) was retrieved and inspected on GitHub. Its 26 changed paths include the two release-metadata files, the new module and three guest suites, new Java values/protocol, Actor/P transport, core numeric and bootstrap integration, corpus registration, and associated Java tests. `pom.xml` and root `CHANGELOG.md` both identify `0.3.401-SNAPSHOT`. No `spec/**` file changed in LIB024.

The normative [D214 specification publication](https://github.com/guillermomolina/protos/commit/798797a72ff3d292b52870da59f595c8607ca2d7) and its [decision record](../../decisions/language/D214_VECTOR_MATRIX.md) are separate, preceding authorities. This record establishes implementation publication only; it does not reopen or silently amend D214-F1.

## Shipped public module and value model

`protos/lib/math/LinearAlgebra.protos` documents the explicitly imported `std:math/LinearAlgebra` module. `ProtosCoreBootstrap.standardModuleMembers` publishes exactly the frozen standard prototypes `Vector` and `Matrix`, together, through the existing module-member mechanism. Neither is placed in the Prelude or added to the core `Number` family.

The correct guest call shape binds prototype members before invoking their constructor behavior:

```protos
LA: import("std:math/LinearAlgebra")
Vector: LA.Vector
Matrix: LA.Matrix
v: Vector(1, 2, 3)
m: Matrix(2, 2, 1, 2, 3, 4)
```

`LA.Vector(1)` is not an equivalent constructor form: ordinary message dispatch selects a prototype slot, not a callable Closure, and signals an Error.

- `ProtosVectorValue` and `ProtosMatrixValue` are identity-bearing ordinary objects. Private `Object[]` numeric payloads are not ordinary slots; user-visible slots and prototype delegation remain ordinary object behavior.
- Constructors admit genuine semantic `Number` values only, not host values or impostors. Matrix dimensions remain exact semantic Integers, including zero-volume shapes with BigInteger extents; bounds are verified before physically representable indexing/size conversion.
- Vector selectors: `size`, `at`, `atPut`, `add`, `scale`, `dot`, `copy`.
- Matrix selectors: `rows`, `columns`, `at`, `atPut`, `add`, `scale`, `matvec`, `matmul`, `transpose`, `copy`.
- Indexed mutation honors the normative `frozen → bounds → Number` rejection order and fails atomically. Index errors become guest Errors. The guest grammar is unchanged; two-dimensional matrix reads use the public selector `at(row, column)`, not an invented comma-separated bracket suffix.

## D214-F1 arithmetic and transfer integrity

`ProtosComplexArithmetic` adds the internal `canonicalNumberBinary` ADD/MULTIPLY and `conjugateNumber` helpers without introducing a new guest numeric family or invoking guest callbacks. Vector and matrix contractions use the prescribed ascending index order, the first product as initial accumulator and Integer zero for empty contractions. `dot` conjugates only its left input; `matvec` and `matmul` remain bilinear. The committed Java tests exercise mixed Integer/Float/Fraction/Complex, exact arithmetic, signed zero, subnormal values, NaN, infinities and order-sensitive rounding.

`ProtosActorValueTransfer` and `ProtosParallelRuntime` allocate and memoize definitive Vector/Matrix shells before populating their elements, slots and state. Their semantic Fraction/Complex representations rematerialize to genuine Numbers before guest observation. The tests exercise aliasing, self-/mutual cycles, prototype sharing, state (`closed`/`frozen`), transfer failures, and lack of unintended mutable sharing.

Huge *nonzero* materializations that are physically unrepresentable still signal an Error rather than truncate or invent storage; zero-volume shapes retain exact dimensions. SIMD/CPU-specialized kernels and GPU offload are **not** part of LIB024; that work belongs to the separate [PLAT068 #959](https://github.com/guillermomolina/protos/issues/959).

## Published acceptance coverage

The product commit registers one explicit filesystem corpus `protos/corpus/library/math/linear-algebra` with three suite-native guest cases:

1. `protos/tests/library/math/linear-algebra/construction-and-indexing.protos`
2. `protos/tests/library/math/linear-algebra/algebra-and-reductions.protos`
3. `protos/tests/library/math/linear-algebra/identity-and-state.protos`

The registered suite leaf is `protos/library/math/linear-algebra`, with requirement `protos/test/ordinary`; the repository-suite expectation increases from 24 to 25 leaves. The publication contains new `ProtosStandardLinearAlgebraProtocolTest`, extended Actor and P integration tests, and updated Test Tool corpus/file-selection/suite-graph tests. The P end-to-end differential coverage uses 19 scalar-valued probes and `ProtosIdentity.identical` against ordinary execution; each worker returns a Number. Full-value Vector/Matrix transfer integrity is also checked through the dedicated transfer helpers. This avoids confusing the independent [BUG026 #958](https://github.com/guillermomolina/protos/issues/958) worker result regression with LIB024 arithmetic.

## Validation provenance and handoff

**Maintainer report, 2026-10-24:** `git diff --check` clean, **all local tests passed**, and the 0.3.401 product implementation was committed and pushed. These statements are maintainer-provided final acceptance, not tests run by the documentation agent.

**Documentation-agent verification:** fetched the exact GitHub product commit, inspected its 26-path changed-file list, published module documentation, version/changelog delta, and formal implementation issue. No Maven/test command, integrated `make test` transcript, total suite count, independent Native Image run, GPU execution, or benchmark measurement was supplied here. Accordingly none is asserted as independently verified. A separate command-specific `make test` acceptance trace remains **unrecorded**, even though the maintainer reports all local tests green.

LIB024's grouped A/B/C scope ships atomically in the referenced product revision; no further public Vector/Matrix selector is deferred to a LIB024 subissue. Subsequent acceleration decisions remain with PLAT068 and do not retroactively change D214-F1 semantics.

## References

- [Formal LIB024 issue #957](https://github.com/guillermomolina/protos/issues/957)
- [Exact 0.3.401 implementation commit](https://github.com/guillermomolina/protos/commit/d86f587beeefa413e323d345f139ccbb164462ca)
- [Ratified D214 specification commit](https://github.com/guillermomolina/protos/commit/798797a72ff3d292b52870da59f595c8607ca2d7)
- [D214 normative closure evidence](../D214/D214_S1_NORMATIVE_PUBLICATION_CLOSURE_2026_10_24.md)
- [PLAT068 acceleration work #959](https://github.com/guillermomolina/protos/issues/959)
