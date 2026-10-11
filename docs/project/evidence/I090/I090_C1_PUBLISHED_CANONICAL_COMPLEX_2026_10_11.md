# I090-C1 — Published canonical Complex numeric family, exact components and isolated transfer

**Record date:** 2026-10-11  
**Formal implementation issue:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)  
**Ratified language decision:** [D197-C](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)  
**Architecture:** PLAT056-C primitive-first numeric execution  
**PROTOS_REVISION:** `c75dea75b8db045eb6833aeab1d5b8710eaa0556`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556  
**Implementation version:** `0.3.338-SNAPSHOT`  
**Global specification revision:** `0.1.455`  
**Validation provenance:** maintainer/human executor reports local focal tests PASS, integrated `make test` PASS, `git diff --check` clean, and commit pushed. Tests were not independently run by the documenting agent.

## Published implementation verified against the exact product revision

The independently retrieved product commit **"I090: introduce canonical Complex values with exact components and isolated transfer"** publishes all 23 expected changed paths, including implementation, tests, conformance, normative/informative specification changes, Maven version and implementation/specification changelogs.

- **Core surface:** `protos/lib/core/Complex.protos` defines `Complex: Number {}`; the Prelude binds `Complex`, and bootstrap validates its exact parent, loads its source and installs only the trusted `Complex.recognizes` provider. The recognizer requires the canonical owner/arity and private family authentication; inheritance or frozen imitation does not confer membership. No public construction, component selectors, operators, literals or complex ordering are introduced.
- **Representation and canonicalization:** `ProtosComplexNumericSupport` uses a private family token with a frozen, sealed `ComplexState(real, imaginary)`. Genuine Integer, Fraction and Float components retain exact semantic families; exact Integer-zero imaginary values collapse to the real component, while imaginary Float `+0.0` and `-0.0` retain separate Complex representations and signed-zero identity.
- **Numeric relations:** `ProtosNumericValueSupport` distinguishes real-only values from the full Number domain. `ProtosCurrentNumericRelations` implements symmetric mathematical equality across Complex/real components, coherent normal hashing (zero imaginary takes the real hash), and retains real-only ordering with the standard Protos Error for Complex. Non-overridable `===` and identity hash are component- and family-sensitive. The native relation does not dispatch guest component callbacks.
- **Actor / P isolation:** Source-stage inert transfer descriptors contain only host integer scalars, Fraction descriptors or Float raw-bit descriptors; destination-stage rematerialization reconstructs BigInteger and Fraction with the destination Prelude's canonical prototypes and mints destination-owned Complex. Identity memos preserve aliases; Float signed zeros retain raw bits. Non-Complex primitive long/double numeric execution paths are not redesigned.
- **Validation coverage:** `ProtosComplexSemanticsTest` exercises bootstrap/authentication, canonicalization, exact mixed-family relations, normal/identity hashes, ordering rejection, conversions, Actor/P transfers and standalone Map/IdentityMap operations with Java-created internal Complex values. Two additional tests cover key lookup and Actor transfer for both map families. Architecture tests audit the updated 36 Core native providers / 137 Core native closures. Native conformance verifies the exposed Core surface, inheritance, forged-value rejection and receiver/arity errors.

## Maintainer validation and provenance boundary

The maintainer reported during the interactive handoff:

1. `mvn test-compile` completed; the later selected focal test run exposed a provider-census expectation and two test-harness attempts to transfer Prelude-owned collection prototypes into a different Prelude.
2. Only the tests were corrected: architecture expected provider census was updated and collection-containing Actor tests were restricted to their existing same-Prelude transfer contract. An independent Complex-only cross-Prelude test verifies destination rematerialization. There was no redesign of the product Complex transfer mechanism in this correction.
3. Corrected focused run: **20 tests, 0 failures, 0 errors, 0 skipped** (`ProtosComplexSemanticsTest` and `ProtosCoreNativeBoundaryArchitectureTest`).
4. Maintainer reported **integrated `make test` PASS** (no complete run transcript or exact total count provided) and `git diff --check` clean; commit was subsequently **pushed**.

The coordinator independently fetched the exact product commit and the published version/specification changelog, but did **not** execute a build, tests, performance benchmarks or Actor processes independently. The green tests are **human-executor attestation**, not independently reproduced evidence.

## Issue boundary: I090-C1 complete; I090 parent still open

This commit completes the bounded **I090-C1** foundation as approved. Together with [I090-A](I090_A_PUBLISHED_SMALL_AND_BIG_INTEGER_FAMILIES_2026_10_11.md) and [I090-B](I090_B_PUBLISHED_EXACT_FRACTION_ARITHMETIC_2026_10_11.md), the existing public integer hierarchy, exact fractions and finite-component Complex value semantics have been published.

**Do not treat I090-C1 as total closure of I090/#868.** The original parent scope also mentions mathematical functions (e.g. exact/approximate `sqrt`, including negative exact real inputs) and a final integration/conformance phase. D197-C's detailed public `Complex` construction/API, `sqrt` selectors/principal branches, IEEE exceptional arithmetic, and associated error contracts remain **unselected**: current normative spec intentionally defers them. No public constructor or math function is ratified or implemented by C1. Final parent closure needs either (a) a separately owner-ratified contract followed by implemented/validated remaining behaviors, or (b) explicit owner approval to narrow/reassign the outstanding work. Do not imply that C1's PASS silently selects or discharges that design gate.

I091/PLAT056 physical containment, independent networking work I093, and PERF040 graph/performance measurement retain their separate issue ownership.

## Exact-revision links

- [I090-C1 product commit](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556)
- [I090 implementation issue](https://github.com/guillermomolina/protos/issues/868)
- [D197 approved numeric-tower decision](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)
- [I090-B published evidence](I090_B_PUBLISHED_EXACT_FRACTION_ARITHMETIC_2026_10_11.md)
- [I090-A published evidence](I090_A_PUBLISHED_SMALL_AND_BIG_INTEGER_FAMILIES_2026_10_11.md)
