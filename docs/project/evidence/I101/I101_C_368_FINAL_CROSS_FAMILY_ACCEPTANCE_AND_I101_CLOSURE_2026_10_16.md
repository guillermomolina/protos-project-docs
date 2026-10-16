# I101-C — Published cross-family acceptance and final I101 closure

**Evidence date:** 2026-10-16
**Formal work:** [I101 / #892](https://github.com/guillermomolina/protos/issues/892) — final implementation acceptance of [PLAT058-D / #891](https://github.com/guillermomolina/protos/issues/891).
**PROTOS_REVISION:** `42c602fc629873436bbc52748acc77391bdb2fc3`
**Published product:** [`42c602fc629873436bbc52748acc77391bdb2fc3`](https://github.com/guillermomolina/protos/commit/42c602fc629873436bbc52748acc77391bdb2fc3) on `main`.
**Product version:** `0.3.368-SNAPSHOT` in the published `pom.xml` and `CHANGELOG.md`.
**Evidence provenance:** public product commit, file paths and Maven version verified by GitHub readback. Focal/integrated test results and clean `git diff --check` were **reported by the maintainer**; the coordinating agent did not execute tests, compilation, graph captures or benchmarks.

## Final published I101-C scope

The final product commit `I101: complete profile-specialized lowering acceptance (0.3.368)` publishes exactly four changed files:

- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101ProfileSpecializedLoweringTest.java` (extended).
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101CGuardedBuilderOracleTest.java` (new).
- `pom.xml` and `CHANGELOG.md` (version and publication description).

I101-C adds acceptance tests rather than a new semantic rule or a second lowering architecture. It exercises NORMAL/DEBUG profiles across:

- D203 flexible Closure argument binding, including positional/default/rest/spread negative inline-admission controls and the absence of obsolete generic arity-rejection roots.
- Lexical capture by reference, nested Closure execution, non-local return, Error identity, `ensure`, suspension/resumption and cancellation.
- A cooperative cancellation witness: after `producer.cancel()`, the `dependency.value()` boundary must prevent executing `afterAwait = 1`; cleanup occurs exactly once and the handler checks the **identity** of the superseding `cleanupError`. Both profiles return `142` through an explicit cooperative `run.invoke()` entry, without imposing RootTask on ordinary synchronous calls.
- Source and RootTags, retained group/CallTarget identities, complete signatures across `ensureComplete()`, and a constrained reparse epilogue: empty, terminal `return`, or `load.null` / zero-or-more `clear.local` / `return`.
- Guard and ordinary fallback parity: the DEBUG Boolean `ifTrue` case may retain the existing I092/PERF041 classifier, but does not gain the NORMAL-only `IsCanonicalBooleanFalseClass` selection; DEBUG emits no I096 literal-Closure inline-admission guard.

The new `ProtosI101CGuardedBuilderOracleTest` has **two independent handwritten** `ProtosSemanticBytecodeRootNodeGen.Builder` oracles, neither derived by invoking the production parser/lowerer for expected bytecode:

1. `i101CGuardedNotMatchesIndependentBuilderIncludingMiss`: the dynamic `receiver.not()` guard, canonical true/false HIT paths, complete ordinary `PrepareSendZero` MISS invocation, full typed bytecode signature equality, stable two-root module/Closure group, and guest/custom-method fallback.
2. `i101CGuardedInlineClosureMatchesIndependentBuilder`: the I096 admitted inline Closure call, physically retained inner Closure root, `AdmitsInlineClosureLiteralCall`, full B-prime `LoadInlineLiteralFallbackCall` and structured continuation/completion, exact outer and inner root signature equality, stable three-root group, and DEBUG ordinary-call control. The guest Integer constant in the inline body is intentionally distinct from the optimized `Long(1)` in the separate physical Closure root.

These structural comparisons retain all emitted branches, typed operands, tags, continuation and fallback instructions; they do not filter away physical machinery in order to claim equality.

## Reported validation and corrected diagnostic attempts

The maintainer first provided focused Maven output: **18 tests, 2 failures, 0 errors**, with causes localized to acceptance-oracle expectations (reparse's valid terminal-only `return` and inline `ProtosIntegerValue` versus a host `Long`). Those two test expectations were corrected without changing production code or weakening complete Builder comparisons.

The subsequent **human-executed** validation on the final substantive files was reported as:

| Gate | Maintainer-reported result |
| --- | --- |
| `mvn -Dtest='ProtosI101ProfileSpecializedLoweringTest,ProtosI101CGuardedBuilderOracleTest' test` | **18 run, 0 failures, 0 errors, 0 skipped** |
| `make check` | **PASS** |
| `make test` | **PASS** |
| `git diff --check` | **Clean** |

The final full-suite case count was not supplied and is not inferred. `pom.xml` and `CHANGELOG.md` were updated only after substantive tests passed; tests were **not** rerun after version-file edits per the human-executor publication contract. The maintainer supplied and pushed product HEAD `42c602fc629873436bbc52748acc77391bdb2fc3`, whose four changed paths and `0.3.368-SNAPSHOT` metadata were independently read back.

## I101 complete: published A / B / C1 / C

| Internal slice | Published product revision | Evidence |
| --- | --- | --- |
| A — Context-owned NORMAL/DEBUG, minimal canonical Boolean fold and first handwritten oracle | [`97404f1cc1c1d678c66ef598b47cf62d32014f21`](https://github.com/guillermomolina/protos/commit/97404f1cc1c1d678c66ef598b47cf62d32014f21) (`0.3.353-SNAPSHOT`) | [I101-A](I101_A_353_CONTEXT_PROFILE_MINIMAL_BOOLEAN_FOLD_PUBLISHED_2026_10_14.md) |
| B — proof-driven arithmetic/scalar locals and independent minimal Builder oracles | [`df4142919d0222635ab8f9f8c54e7a1966e6edaa`](https://github.com/guillermomolina/protos/commit/df4142919d0222635ab8f9f8c54e7a1966e6edaa) (`0.3.357-SNAPSHOT`) | [I101-B](I101_B_357_MINIMAL_NORMAL_ARITHMETIC_BYTECODE_PUBLISHED_2026_10_14.md) |
| C1 — canonical comparison folding and negative-control acceptance | [`c7004f71cb05621dd5a7f89e46459542bd7d9c87`](https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87) (`0.3.358-SNAPSHOT`) | [I101-C1](I101_C1_358_CANONICAL_COMPARISON_FOLDING_PUBLISHED_2026_10_15.md) |
| C final — NORMAL/DEBUG cross-family conformance, guard/fallback, Context/Task, reparse/topology, two guarded complete Builder oracles | [`42c602fc629873436bbc52748acc77391bdb2fc3`](https://github.com/guillermomolina/protos/commit/42c602fc629873436bbc52748acc77391bdb2fc3) (`0.3.368-SNAPSHOT`) | This record |

**Closure boundary:** I101's owner-approved PLAT058-D proof-backed physical lowering and cross-family acceptance is complete as a bounded implementation. It is **not** a claim of stable Tier-1/Tier-2 Graal graphs, measured speedup, GraalJS/GraalPy parity, a new canonical optimizer architecture, Native Image proof, or wholesale elimination of dynamic fallback. Those are separate questions/work items.

Historical A/B/C1 evidence documents the open parent *at those earlier publication dates*; those immutable reports are not rewritten retroactively. The owning [I101 issue #892](https://github.com/guillermomolina/protos/issues/892) is the authoritative live closure/status record.

## Revision-pinned references

- [Product `0.3.368-SNAPSHOT` changelog](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/CHANGELOG.md).
- [Product `0.3.368-SNAPSHOT` Maven version](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/pom.xml).
- [Cross-profile acceptance and cancellation tests](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101ProfileSpecializedLoweringTest.java).
- [Independent guarded Builder-oracle tests](https://github.com/guillermomolina/protos/blob/42c602fc629873436bbc52748acc77391bdb2fc3/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI101CGuardedBuilderOracleTest.java).
- [PLAT058-D ratified architecture](../../decisions/platform/PLAT058_PROFILE_SPECIALIZED_MINIMAL_LOWERING_AND_BUILDER_ORACLES.md).
