# I090-B — Published exact Fraction arithmetic and canonical rational normalization

**Record date:** 2026-10-11  
**Formal work:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868)  
**Decision:** [D197-C](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md), with D196-B1 Float promotion and PLAT056-C primitive-first execution  
**PROTOS_REVISION:** `e769869d155a999501e96873eb7d26d633b7c2e7`  
**Published implementation:** https://github.com/guillermomolina/protos/commit/e769869d155a999501e96873eb7d26d633b7c2e7  
**Version at this exact revision:** `0.3.337-SNAPSHOT`  
**Validation provenance:** human-executor attestation of local suite PASS and clean `git diff --check`; not independently executed here.

## Published implementation

The source commit **"I090: implement exact Fraction arithmetic and canonical rational normalization"** adds `protos/lib/core/Fraction.protos`, `src/main/java/com/guillermomolina/protos/execution/ProtosStandardFractionProtocol.java`, Java conformance `ProtosFractionSemanticsTest` and native conformance `fraction-arithmetic.protos`/`fraction-relations.protos`. The standard `Fraction` prototype delegates directly to `Number`, is published in Core Prelude, and has a trusted recognizer but no public constructor. Protos source closures implement Fraction-receiver arithmetic and negation. A private runtime kernel reads canonical components, normalizes exact rational values and rounds a complete exact quotient to binary64 when explicitly required.

`ProtosNumericValueSupport` now recognizes canonical Fraction values, normalizes positive-denominator reduced components, demotes integer-valued results, provides exact comparison/identity/hash integration and rematerialization capabilities. Fraction values use an authenticated frozen `ProtosSealedValue` with private family identity rather than a dedicated Java Fraction value class. Ordinary Fraction-delegating objects do not become Core numbers.

`Integer / Integer` is exact: integral quotients remain normalized Integer, nonintegral quotients become Fraction. `ProtosSemanticBytecodeRootNode` preserves primitive `long` results for integral division and returns to ordinary canonical execution for nonintegral results, zero, and signed overflow; Float mixed arithmetic stays operand-first binary64 per D196-B1.

Integer-receiver operators with a Fraction argument use a **bounded direct exact operation** in `ProtosStandardIntegerProtocol` (add/subtract/multiply/divide, reusing the existing canonical numeric normalization service). The originally proposed use of `ProtosInvocation.invokeMessage` from a native method was rejected after a maintainer/agent static investigation identified that synchronous native-to-guest invocation inside a Task throws rather than entering the Task-owned C-prime path and can stall termination. The published approach does **not** create new Bytecode dispatch infrastructure. Fraction-receiver arithmetic remains in Protos. This is a narrow implementation exception, not a general decision to move numeric semantics into Java.

Cross-family exact equality and order, normal numeric hashing/Map coherence, family-sensitive `===` and IdentityMap identity, `Float(Fraction)` exact quotient rounding, and Actor/P destination-owned transfer using inert descriptors are integrated. Other sealed value families remain nontransferable. TOML and detached Float-bit tooling now explicitly use `Float(a / b)` where a Float result is intended; logging remains deliberately restricted to its own approved scalar contract.

The commit includes normative `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, informative `spec/runtime/ABSTRACT_RUNTIME.md`, `pom.xml`, and `CHANGELOG.md`. The corresponding published contract is **not** evidence that D197's Complex family is implemented.

## Human executor validation and caveats

The maintainer reported on 2026-10-11:

- `git diff --check` clean.
- All tests passed locally.
- The I090-B commit was pushed.

This documentation action independently retrieved the published commit and changed-path inventory. It did **not** rerun Java tests, Protos tests, the integrated suite, compiler checks, CI or benchmarks. Test logs, exact run/test counts, machine time and performance measurements were not provided. Do not describe the reported green tests as independently reproduced.

The implemented numeric service has grown beyond the recommended 800-line guideline. Any future extraction should be separately justified by responsibility boundaries and verified preserved semantics, not inserted into Complex implementation as unrelated cleanup.

## Remaining I090 work

**I090 stays OPEN.** Its approved remaining target is Core `Complex` with exact Integer/Fraction components, canonical reduction for exact imaginary zero, preservation of inexact signed imaginary zero for IEEE branch cuts, numeric equality/identity/hash, Actor/P/Context transfer and regression coverage. D197 explicitly leaves **construction/public selectors, nontrivial IEEE exceptional operations, detailed principal-branch math functions and some conversion/error rules unselected**. Do not imply that implementing I090-B approved them. I091/PLAT056 containment and PERF040 graph/performance remain independent.

## Exact-revision provenance

- [Published I090-B product commit](https://github.com/guillermomolina/protos/commit/e769869d155a999501e96873eb7d26d633b7c2e7)
- [Published I090-A predecessor](I090_A_PUBLISHED_SMALL_AND_BIG_INTEGER_FAMILIES_2026_10_11.md)
- [Ratified D197-C](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md)
- [Live implementation issue](https://github.com/guillermomolina/protos/issues/868)
