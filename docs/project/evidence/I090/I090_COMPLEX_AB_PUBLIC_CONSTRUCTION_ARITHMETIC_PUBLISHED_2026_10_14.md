# I090 — Published public Complex construction and arithmetic (A+B)

**Record date:** 2026-10-14
**Owner:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)
**Ratified authorities:** [D197](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md) and [D198](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md), spec baseline `0.1.457`
**PROTOS_REVISION:** `978c25968ffe0815c2e64b729cfd83319c807724`
**Product version:** `0.3.351-SNAPSHOT`
**Product commit:** https://github.com/guillermomolina/protos/commit/978c25968ffe0815c2e64b729cfd83319c807724
**Evidence type:** published source and maintainer-reported validation, not an independently executed test/benchmark.

## Published scope

The product commit **"I090: add public Complex construction, components and standard arithmetic (0.3.351)"** implements coherent I090 Complex blocks A+B.

- Public `Complex(real, imaginary)` admits two authentic real-number components including NaN, infinities and signed zeros, no dispatch or coercion of arguments, and collapses only exact Integer-zero imaginary components.
- Source-backed `Number.real()` and `Number.imaginary()` select components; source-backed Complex `negated`, `+`, `-`, `*`, `/` preserve ordinary overridable method lookup and dispatch.
- Integer, Float and Fraction receivers accept Complex arguments in both directions while standard real-only conversions and ordering still reject Complex.
- `ProtosComplexArithmetic` is the shared private numerical owner used by Complex source closures and native Integer/Float receiver bodies (which cannot synchronously call guest code). It implements separable coordinates, exact-rational coupled arithmetic, exact-zero-divisor E2/A, correctly once-rounded finite binary64 coupled coordinates R, signed-zero F1 and exceptional classifications G1.
- `ProtosBinary64Rounding.divideExactBigIntegers` supports exact quotient rounding. The coupled finite Float kernel uses exact dyadic scaling by `2^1074`, so the result does not inherit premature binary64 product/sum rounding.
- The published change adds `ProtosComplexArithmeticTest` (independent reference oracle and randomized finite/special cases) and `protos/tests/conformance/number/complex-arithmetic.protos` (registered in `manifest.tsv`), and updates corresponding Java architecture/semantic and Protos conformance tests.

The paired A+B implementation avoids an intermediate provisional coupled-Float behavior. Its kernel's Java implementation is justified by the native Integer/Float body boundary; it is reached only for authentic Complex operands and does not redefine ordinary small Integer/Float fast paths.

## Validation provenance

On 2026-10-14 the human executor reported:

- Product commit `978c25968ffe0815c2e64b729cfd83319c807724` pushed.
- `git diff --check` clean.
- All local tests PASS.

The recording agent independently confirmed the published commit and inspected current implementation/specification. It did **not** execute `make test`, other tests, builds, Native Image, benchmarks or the human's Git operations. Exact commands/logs, total executed test counts and runtime are not supplied by this attestation. No independently measured Graal graph/timing improvement is claimed.

## Next steps, ownership and limits

**I090 stays OPEN** until `std:math/Math.sqrt` is implemented with D198 F3 exactness, N4 special values/principal branch cuts and G2-B 4-ULP inexact accuracy, then cross-slice conformance of Actor/P/Context, identity/hash, Map/IdentityMap and interop is validated and the final human-run full-suite closure gate is satisfied. No new implementation microissue is necessary.

[PLAT056 / #866](https://github.com/guillermomolina/protos/issues/866) and [I091 / #869](https://github.com/guillermomolina/protos/issues/869) own the already-approved primitive-first numeric architecture; this work does not reopen them. [PERF040 / #862](https://github.com/guillermomolina/protos/issues/862) separately owns measured primitive numeric arithmetic acceptance, while a Complex-specific performance hypothesis requires measured evidence before any new PERF optimization. [PERF046 / #890](https://github.com/guillermomolina/protos/issues/890) remains independent comparison work.

## References

- [Exact product revision](https://github.com/guillermomolina/protos/commit/978c25968ffe0815c2e64b729cfd83319c807724)
- [I090 live issue](https://github.com/guillermomolina/protos/issues/868)
- [D198](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md)
- [Earlier exact Fraction checkpoint](I090_B_PUBLISHED_EXACT_FRACTION_ARITHMETIC_2026_10_11.md)
