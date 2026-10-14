# I090-C — Published Standard Library Math.sqrt

**Record date:** 2026-10-14
**Formal implementation owner:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)
**Governing decisions:** [D197](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md), [D198](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md)
**Normative specification:** `0.1.457`
**PROTOS_REVISION:** `8b452c83b42dad3ab59914e3b17b4fb2de886b3e`
**Version:** `0.3.352-SNAPSHOT`
**Product publication:** https://github.com/guillermomolina/protos/commit/8b452c83b42dad3ab59914e3b17b4fb2de886b3e
**Parent source checkpoint:** [I090 Complex A+B at `978c25968ffe`](I090_COMPLEX_AB_PUBLIC_CONSTRUCTION_ARITHMETIC_PUBLISHED_2026_10_14.md)
**Evidence type:** verified product publication and maintainer-reported local validation, without independent tests or benchmark reruns.

## Published functionality

The published commit is **"I090: add std:math/Math sqrt with exact, special-value and correctly rounded roots (0.3.352)"**.

- `protos/lib/math/Math.protos` defines only the ordinary source-backed `sqrt(value)` module Closure, privately capturing a runtime facility then removing its bootstrap binding. It is imported explicitly as `std:math/Math`, not added to Prelude or Number.
- `ProtosMathFacility` supplies an immutable private native implementation boundary with exact receiver/arity and genuine-Number validation, obtaining the current Prelude for destination-owned result construction and sending no guest message to its argument.
- `ProtosSquareRoot` implements D198 principal square root. Rational exact roots of real and Complex operands remain exact (F3); Float real and Float-bearing Complex NaN, signed zero and infinities follow the specified N4 priority table.
- Inexact coordinates use exact rational representation of source values and exact midpoint comparisons to select binary64 correctly rounded `roundTiesToEven` results, stronger than D198's four-ULP G2-B requirement. The implementation avoids premature exact-component-to-double conversion.
- The changed bootstrap provisions the facility for the standard module only. The published `CHANGELOG.md` identifies version `0.3.352-SNAPSHOT`.
- New Java tests `ProtosMathModuleTest` and `ProtosMathSqrtAccuracyTest` exercise module interface, authentic operands, exactness, N4 cases, signed zeros and an independently formulated accuracy oracle; the native Protos conformance file is `protos/tests/conformance/number/math-sqrt.protos` and is registered in `manifest.tsv`. The Core native-boundary architecture guard is updated for the single private non-Core facility.

This record describes the published candidate and its reported validation, not an independent proof of numerical correctness across all representable inputs. No measured performance speedup or Graal fast-path acceptance is claimed.

## Human-executor validation

On 2026-10-14 the maintainer reported:

- The above exact product commit was pushed.
- `git diff --check` was clean.
- All tests passed locally.

The coordinating agent independently read the published commit, `protos/lib/math/Math.protos` and current `CHANGELOG.md`. It did not execute a build, JUnit, conformance, `make test`, Native Image or benchmarks; no command-by-command logs, test totals or durations were provided. The statement "all tests passed locally" remains maintainer attestation.

## Remaining I090-D closure and distinct performance work

**I090 stays OPEN.** A final consolidated semantic/conformance review must account for the D197/D198 numeric family boundary and interactions: canonicalization, real/Complex dispatch and overrides, equality and hash coherence, IdentityMap versus Map, isolated Actor/P transfer/rematerialization, distinct Prelude/Context authority, interop and error behavior, and G2-B/N4 conformance at the public imported-module surface. Reuse existing tests and add only coverage needed for demonstrated gaps; fix real defects in an approved implementation slice. A human-run final full integrated suite and revision-coupled publication record precede formal closure.

[PLAT056 / #866](https://github.com/guillermomolina/protos/issues/866) and [I091 / #869](https://github.com/guillermomolina/protos/issues/869) are separate closed architecture/representation work. [PERF040 / #862](https://github.com/guillermomolina/protos/issues/862) remains the separate performance/compiled-graph owner; this implementation does not establish improved primitive arithmetic or Complex compiled performance. [PERF046 / #890](https://github.com/guillermomolina/protos/issues/890) separately investigates cross-family numeric comparison optimization.

## Exact publication links

- [Product commit `8b452c83b42d`](https://github.com/guillermomolina/protos/commit/8b452c83b42dad3ab59914e3b17b4fb2de886b3e)
- [Owning I090](https://github.com/guillermomolina/protos/issues/868)
- [D198 ratification](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md)
- [Previous I090 A+B evidence](I090_COMPLEX_AB_PUBLIC_CONSTRUCTION_ARITHMETIC_PUBLISHED_2026_10_14.md)
