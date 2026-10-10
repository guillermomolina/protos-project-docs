# I091 — Post-publication numeric containment and Candidate C closure audit

**Date:** 2026-10-10  
**Owning implementation issue:** [I091/#869](https://github.com/guillermomolina/protos/issues/869) — OPEN; this is **not** closure evidence.  
**Governing ratified architecture:** [PLAT056/#866](https://github.com/guillermomolina/protos/issues/866), Candidate C.  
**Published numeric migration:** `PROTOS_REVISION=0fec04835eef0ad625303dfb758918205eb3fd13` ([commit](https://github.com/guillermomolina/protos/commit/0fec04835eef0ad625303dfb758918205eb3fd13)).  
**Previous published numeric implementation:** `7dfceb7ee96964eb5cb5ed6577b8be2fbd91b68f` ([commit](https://github.com/guillermomolina/protos/commit/7dfceb7ee96964eb5cb5ed6577b8be2fbd91b68f)).  
**Later observed main HEAD, separate concurrent work:** `70ee9504224c59c033c647f9185bf8fb9b266b93` (I092); the implementation agent must inspect real local HEAD and preserve concurrent changes.  
**Validation provenance:** owner reports `git diff --check` clean and all local tests PASS for the published work; command output, test inventory, exact execution time and benchmarking are not supplied. The evidence author did **not** compile, run tests or benchmark.  
**Record type:** source-grounded audit / incomplete closure acceptance; not a normative amendment or an owner-approved new architecture.

## What the published work demonstrably changed

- `ProtosIntegerValue` has **only** `private final long value` as numeric instance state and one numeric constructor taking `long`. No nullable large-number state or Java `BigInteger` constructor survives.
- Large exact mathematical Integers are normalized into `ProtosLargeIntegerValue`, a frozen subclass of ordinary `ProtosObjectValue` delegating to the minting Prelude's Integer prototype, with a narrow immutable `java.math.BigInteger` magnitude. Demotion to signed-64 materializes `ProtosIntegerValue`.
- `ProtosNumericValueSupport` centralizes recognition, primitive and large exact arithmetic, comparison, hashing, unsigned encodings and destination-owned numeric copies. The ordinary path does not automatically execute large-number algorithms.
- Both Bytecode DSL roots specify `boxingEliminationTypes={int.class,long.class,double.class}`; protected arithmetic, comparison, frame local, call and return carrier machinery is present in source.
- `ProtosFixedIntegerInteropValue` now stores width/signedness and long bits, while retaining `BigInteger` on the expressly supported arbitrary-width host contract where appropriate; `ProtosIntegralInteropSupport` has long and BigInteger projection helpers.
- Tests exist for numeric boundary/overflow/rounding, identity, cross-domain rematerialization, primitive carriers and rich numeric feasibility. Existence of tests and owner-reported suite PASS does not prove every architectural falsification gate has been satisfied.

## Why I091 cannot yet close

### 1. Full arbitrary-precision algorithm review is still owed — not just a lexical census

The current `ProtosBinary64Rounding` contains:

```java
private static int floorBinaryExponent(
        BigInteger numerator, BigInteger denominator, int roughExponent) {
    if (roughExponent >= 0) {
        return numerator.compareTo(denominator.shiftLeft(roughExponent)) < 0
                ? roughExponent - 1 : roughExponent;
    }
    return numerator.shiftLeft(-roughExponent).compareTo(denominator) < 0
            ? roughExponent - 1 : roughExponent;
}
```

This calculates an exact comparison of an arbitrary-precision rational. Using `BigInteger` *at a genuinely unbounded arithmetic boundary* is permitted by PLAT056; however, the exact comparison currently constructs scaled big operands. It is **not sufficient** to classify the function `NECESSARY` and stop: investigate allocation-avoiding compare techniques, safe primitive cases and fully proved equivalence.

The same class has `divideExactIntegers(Object,Object)` with an admitted primitive quotient only if the operands are exact binary64 integers (or zero numerator). When signed-64 inputs fail that narrow guard, it unconditionally converts them via `ProtosNumericValueSupport.exactBigInteger` and goes through `BigInteger` arithmetic. This is a **specific, source-verified candidate** for an improved exact signed-64 round-once algorithm, or a narrower safe primitive domain, before accepting unavoidable large-number work. Investigate the full chain: `roundedMagnitudeBits`, `floorBinaryExponent`, `roundedScaledQuotient`, `divideAndRemainder`, scaled operand construction, ties-to-even, subnormal/normal transition, overflow/underflow and positive signed zero. Do not trade correctness for fewer imports; compare to the existing exact oracle at random and adversarial boundaries. Do not implement an entire arbitrary-precision library to remove legitimate `BigInteger` uses.

`ProtosIntegerValue.asBigInteger()` is an explicit Truffle interop projection, so its temporary `BigInteger.valueOf(long)` is legitimate *at that call*, not general permission to construct that projection inside every VM operation. `ProtosLargeIntegerValue` may retain a narrow exact host payload under Candidate C if its object-model proof holds.

### 2. Representation observability / F1-F4 unresolved

`ProtosStandardObjectProtocol.call` checks `receiver instanceof ProtosObjectValue` and constructs a new object delegating to the receiver. A small `ProtosIntegerValue` fails the test; a large `ProtosLargeIntegerValue` passes because it subclasses `ProtosObjectValue`. The other reflective object natives use `ordinaryReceiver()` to exclude large Integers. This is **a concrete candidate for D013-dependent observable divergence**, especially when the Object method is selected through override, alias or forwarding; not yet a dynamically reproduced defect. Prove it unreachable or fix with a narrowly scoped regression, while keeping ordinary dispatch/prototype semantics.

Also audit every other generic `instanceof ProtosObjectValue` and interop/tooling/reflection entry point that could treat large Integers as mutable/prototypable objects rather than canonical numeric values. Review Actor/P/Process/detached/multi-Context transfers, aliasing, maps, identity, ordinary `==`, `===`, hash and immutability. Do not make a Java-specific subclass observable. Establish whether the frozen narrow-payload subclass is an acceptable Candidate C *implementation detail* with ordinary delegation, or a substantive counterexample to Candidate C; Candidate B is not automatically authorized.

### 3. Required exhaustive containment closure

The lexical census measures references in all `src/main/java` and excludes strings/comments, but its `classification` field remains `UNCLASSIFIED_NEEDS_SOURCE_AUDIT`. The issue requires an individual source/line inventory of `java.math.BigInteger`, `ProtosIntegerValue` and `ProtosFloatValue`, tagged by real responsibility and measured before/after on pinned source bytes.

Work across ALL remaining families, not only `ProtosBinary64Rounding`:
- Numeric payload and services: `ProtosIntegerValue`, `ProtosLargeIntegerValue`, `ProtosNumericValueSupport`, `ProtosFloatValue`, `ProtosCurrentNumericRelations`, `ProtosStandardNumericConversionProtocol`, `ProtosNumberLiteral`.
- Truffle fixed-width and host interop: `ProtosFixedIntegerInteropValue`, `ProtosIntegralInteropSupport`, `ProtosForeignAdmissionDescriptor`, `ProtosHostJavaProvider`, `ProtosHostJavaCatalogue`, `ProtosForeignPluginValueAdapter`, `ProtosPolyglotValueOperations`, and public SPI classes.
- Explicit contracts: `ProtosSemanticTransferPayload` and `ProtosRegexSemanticTransferFamily` (PLAT051 specifies BigInteger payload leaves); `ProtosPackageExecutionPlanV2` and its adapter (unbounded SemVer fields).
- Lowering/runtime, collection/identity, indexing, byte/File/network, Actor/Process/P, debugger, native-image and test utilities — examine hidden numeric costs even where there is **no textual BigInteger import**.

No blind quota is approved, but neither is blanket `all remaining = necessary`. For each occurrence: preserve when demonstrably required for unbounded precision/host ABI; otherwise eliminate *the unnecessary conversion/allocation/representation dependency*. Remove avoidable work without merely moving imports into utility classes. Check the eager `ProtosNumericValueSupport.OCTETS` static 256-wrapper cache against PLAT056 pay-as-you-grow; this is a review candidate, not a measured regression.

### 4. Work-owner and semantic gates

The current normative `spec/semantics/VALUES_AND_COLLECTIONS.md` still describes an unbounded visible `Integer` with unobservable large representation, and explicitly does not expose a Core `BigInteger` family. `D197/#865` ratifies a distinct future family but the public `Integer.recognizes` contract was explicitly left unresolved. `I090/#868` owns the eventual public numeric-tower/spec implementation; I091 must not install unapproved visible semantics. The current large Integer being *semantically* Integer is consistent with the current spec.

`PERF040/#862` owns pinned compiled-graph/steady-state measurement; no performance PASS is claimed by a reduction in Java lexical references.

## Status and next acceptance

**I091 remains OPEN / implementation continuation warranted**. Do not create a separate implementation issue to relocate unfinished I091 acceptance work merely to close #869. Next handoff must be a comprehensive implementation/acceptance pass spanning the entire source-grounded audit, numeric exact algorithms, representation observability and final evidence. Group related source changes; don't stop after one helper function or an arbitrary lexical count.

Require one integrated human-run `make test` after final implementation changes, additional focused deterministic and differential tests as needed under the impact policy, and an explicit source/policy audit. After green tests, version `pom.xml`/`CHANGELOG.md` immediately before human-executed commit/push; **no tests after editing those files**. Final product publication must be pinned to a real SHA, followed by a separate durable final acceptance record and GITHUB020 closure comment; keep I091 open until those closure gates really pass.

**Audit evidence is not a claim of tests/benchmarks run by the author, a bug reproduced, or a new PLAT056/D197 approval.**
