# I090-D — Final numeric integration and human-validated closure

**Evidence date:** 2026-10-14
**Owning formal implementation:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)
**Governing semantic authority:** D197 exact numeric tower and D198 public Complex/sqrt; specification baseline `0.1.457`
**PROTOS_REVISION:** `5a57c2edb5d62fc89d96386da69eb88370955632`
**Product commit:** https://github.com/guillermomolina/protos/commit/5a57c2edb5d62fc89d96386da69eb88370955632
**Published implementation version:** `0.3.352-SNAPSHOT` (unchanged by the test-only I090-D commit)
**Evidence class:** verified product commit and scope; maintainer-reported local validation; not independently executed test evidence.

## Final I090-D publication

The product commit **"I090: validate derived numeric results across Actor and P"** is a one-commit descendant of I090-C `8b452c83b42dad3ab59914e3b17b4fb2de886b3e` and modifies **only** `src/test/java/com/guillermomolina/protos/execution/ProtosMathModuleTest.java` (**98 additions**, no deletions or runtime changes).

The added `sqrtResultsRetainFamiliesAndSignedZerosThroughActorAndPTransfer` JUnit test exercises published derived numeric results—not just directly constructed values—through both staged Actor transfer and parallel P capture/rematerialization into another Prelude:

- `sqrt(Complex(3,4))` remains canonical `Complex(2,1)`, with the destination's Complex prototype; a repeated root preserves aliasing.
- `sqrt(1/4)` stays an exact Fraction with the destination's Fraction prototype.
- `sqrt(Complex(-4.0,-0.0))` retains positive-real zero and negative-imaginary `-2.0`, compared by binary64 bits.
- `sqrt(Complex(2^1200,1.0))` retains the properly represented finite Float coordinates (`2^600` and `2^-601`) under destination transfer.
- `sqrt(2^360)` remains an exact BigInteger result with the destination BigInteger prototype.
- A derived Complex product `Complex(1,2) * Complex(3,4)` remains exact `Complex(-5,10)` after transfer.
- The receiver's Prelude owns the new semantic guest values; semantic numeric identity is preserved when transferring a value, without requiring physical object identity with the source.

Source inspection of the preexisting tests supports that `ProtosComplexSemanticsTest` already verifies authentic numeric values, canonicalization, mathematical equality and identity, hash coherence, Map versus IdentityMap, host-state hiding/interop, explicit Actor/P transfer, cross-Prelude rematerialization and aliasing. `ProtosComplexArithmeticTest` covers D198 arithmetic, finite accuracy and exceptional cases; `ProtosMathModuleTest` and `ProtosMathSqrtAccuracyTest` cover the public `std:math/Math.sqrt` API and numeric accuracy. The final I090-D test fills the identified derived-result transfer gap without duplicating those suites.

## Human-executor validation provenance

For the candidate later committed as `5a57c2edb5d62fc89d96386da69eb88370955632`, the human reported:

1. Focal Maven JUnit subset: **57 tests run; 0 failures, 0 errors, 0 skipped**.
2. Canonical integrated **`make test` FULL: PASS** (human response "pasa").
3. Git commit/push published; local working tree **clean**, with no additional staged changes per human report.

The coordinating agent did not execute tests or verify machine-local build logs; no independently reproduced JUnit, conformance, Native Image or benchmark results are claimed. The exact FULL test count and time were not provided.

The human reported that no further modifications remained to commit once `5a57c2ed` was pushed. The observed GitHub compare from `8b452c83` to `5a57c2ed` includes exactly this one test-only change. Under the published implementation-versioning policy, an exclusively test-file change does not require an implementation version increment; `pom.xml` and `CHANGELOG.md` remain untouched in I090-D. No tests were run after a version/changelog edit in this slice.

## Earlier I090 publication chain

The closure rests on published, exact-revision implementation checkpoints rather than an inferred single rewrite:

- [I090-A SmallInteger / BigInteger families](I090_A_PUBLISHED_SMALL_AND_BIG_INTEGER_FAMILIES_2026_10_11.md): Protos `888c2b4f2390b96de5990533fdb1b306587e0002`.
- [I090-B exact Fraction arithmetic](I090_B_PUBLISHED_EXACT_FRACTION_ARITHMETIC_2026_10_11.md): Protos `e769869d155a999501e96873eb7d26d633b7c2e7`.
- [I090 Complex construction/components and D198 arithmetic](I090_COMPLEX_AB_PUBLIC_CONSTRUCTION_ARITHMETIC_PUBLISHED_2026_10_14.md): Protos `978c25968ffe0815c2e64b729cfd83319c807724` / `0.3.351-SNAPSHOT`.
- [I090-C public Math.sqrt](I090_C_352_PUBLISHED_STANDARD_MATH_SQRT_2026_10_14.md): Protos `8b452c83b42dad3ab59914e3b17b4fb2de886b3e` / `0.3.352-SNAPSHOT`.
- [I090-D final Actor/P integration](https://github.com/guillermomolina/protos/commit/5a57c2edb5d62fc89d96386da69eb88370955632): same implementation version, test-only publication and maintainer-validated FULL closure candidate.

## Closure disposition and scope boundary

**I090-D acceptance: PASS on maintainer-reported local focal and integrated validation, with product publication verified.** The remaining formal closure action is the live GitHub issue transition after this exact durable record is published and read back.

I090 implements D197/D198 specified numeric semantics and API. Closing I090 **does not** establish a performance claim for Complex arithmetic, compiled Graal graph acceptance, Native Image validation beyond previously published evidence, or a cross-language benchmark result.

[PERF040 / #862](https://github.com/guillermomolina/protos/issues/862) retains separate compiled-graph/timing acceptance for primitive numeric arithmetic. [PERF046 / #890](https://github.com/guillermomolina/protos/issues/890) retains distinct numeric comparison performance work. Completed [I091 / #869](https://github.com/guillermomolina/protos/issues/869) and [PLAT056 / #866](https://github.com/guillermomolina/protos/issues/866) remain separate architecture/representation history.

## Links

- [Exact final Protos commit](https://github.com/guillermomolina/protos/commit/5a57c2edb5d62fc89d96386da69eb88370955632)
- [I090 live issue and closure](https://github.com/guillermomolina/protos/issues/868)
- [D197 decision](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)
- [D198 decision](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md)
