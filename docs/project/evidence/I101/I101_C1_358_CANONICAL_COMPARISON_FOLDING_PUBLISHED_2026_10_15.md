# I101-C1 — Published canonical comparison folding with minimal NORMAL Bytecode

**Evidence date:** 2026-10-15  
**Owning work:** [I101 / #892](https://github.com/guillermomolina/protos/issues/892), slice **C1 PUBLISHED**; parent **OPEN/READY** for remaining cross-family C acceptance.  
**Governing decisions:** [PLAT058-D / #891](https://github.com/guillermomolina/protos/issues/891) (immutable NORMAL/DEBUG compilation profiles) and [PLAT059 / #894](https://github.com/guillermomolina/protos/issues/894) (no new optimizer architecture; proven canonical Core dispatch is not guarded for hypothetical override).  
**PROTOS_REVISION:** `c7004f71cb05621dd5a7f89e46459542bd7d9c87`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87  
**Product version:** `0.3.358-SNAPSHOT`  
**Publication provenance:** GitHub commit/version/source independently read back. The maintainer reports local `make test` PASS, `make check` PASS, and `git diff --check` clean; this documenting agent did **not** execute tests, check, compile, or benchmarks.

## Published scope

Product `main` contains the commit **`I101: fold canonical comparisons into minimal NORMAL bytecode`**, changing exactly these five files:

- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosI101CComparisonBuilderOracleTest.java` (new)
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java`
- `pom.xml`
- `CHANGELOG.md`

The lowerer statically folds admitted, proven canonical Boolean/Number comparison results in NORMAL: `<`, `<=`, `>`, `>=`, `==`, `!=`, `===`, `!==`. The published tests exercise signed-64 and large Integer/Float boundary examples, equality versus identity, nested arithmetic/comparison expressions, Boolean and null identity, and propagation through proven scalar-local bindings. Exact canonical Core cases need neither runtime send nor a guard against unrelated user-defined same-name selectors; genuinely dynamic/custom receivers retain their ordinary message dispatch. This is a bounded proof-based lowering change, not a new optimizer/IR, not a universal assertion that comparison messages cannot be overridden.

## Physical and semantic acceptance

The published `ProtosI101CComparisonBuilderOracleTest` independently builds a reference Boolean-literal root with `ProtosSemanticBytecodeRootNodeGen.Builder` and asserts complete physical instruction/constant signature equality with the product-generated NORMAL roots for `() => { 1 < 2 }` and `() => { true }`, **before** Graal optimization. It asserts removal of runtime numeric send, invocation preparation, local/frame authority and return-carrier materialization operations from the admitted minimal literal case. The comparison-family table checks both true and false results against the corresponding physical Boolean-literal signatures. Scalar-local cases test guest truth values and absence of surviving numeric dispatch. The test also checks reparse/root/CallTarget stability for the direct oracle case.

Negative controls in the test preserve behavior when operands are dynamic or an ordinary object supplies its own comparison operators: those cases must not be replaced with an unconditional constant. The PERF025 regression suite continues to assert exact four-/six-instruction NORMAL scalar-local bodies, rather than accepting an added `PublishObservedDirectScalarLocals` operation. That speculative publication operation was **removed** before publication and is absent from the five-file product commit.

The owner clarified the mode boundary during validation: **NORMAL pays for guest-language observations explicitly requested**, whereas **DEBUG supports full debugger observability**, including source tags and lexical bindings. The failing legacy PERF043 test had pre-materialized a Java activation and demanded full implicit scope visibility while executing NORMAL. The final regression runs this explicit pre-entry external-observation case in DEBUG, preserving assertions for `left`, `right`, activation identity, source `StatementTag` and arithmetic `CallTag`. A separate pre-existing test `i101ExplicitContextObservationRetainsRealBindingsInNormal` covers true guest `context` observation in NORMAL. No universal debug instrumentation was added to NORMAL.

## Validation trail (human executor)

The maintainer reported these successful gates, in order:

1. Independent I101-B/C1 Builder oracles plus PERF034 minimum-body regressions: **14 tests, 0 failures, 0 errors, 0 skipped**.
2. The complete affected `ProtosPerf025CompactCalleeExecutionTest` after correcting the DEBUG test source literal: **32 tests, 0 failures, 0 errors, 0 skipped**.
3. Integrated `make test` **PASS** and `make check` **PASS**; exact integrated suite count was **not** supplied in the final passing report.
4. Final `git diff --check` clean and maintainer-confirmed push. GitHub product commit `c7004f71cb05621dd5a7f89e46459542bd7d9c87` and matching `0.3.358-SNAPSHOT` in `pom.xml` and `CHANGELOG.md` independently verified.

Earlier failing attempts revealed outdated test expectations and an unacceptable extra runtime publication operation; neither is counted as passing acceptance. The published result is the corrected, independently verifiable snapshot.

## Boundaries and remaining work

- **I101-C1: PUBLISHED.** **I101 parent: OPEN/READY** for remaining C cross-family acceptance of NORMAL/DEBUG, other canonical families, dynamic fallback, capture, Task, ReturnHome, source instrumentation, reparse and topology. C1 alone is not authorization to close #892.
- Independent Graal Tier-1/Tier-2 convergence, benchmark speedup, GraalJS/GraalPy parity, Native Image and all-family semantic completeness are **not claimed**.
- PLAT060's subsequently ratified universal Closure-call ABI is a separate future implementation concern; this evidence records the specific published `0.3.358-SNAPSHOT` product state, not migration or validation of that ABI.
- No additional product specification, `AGENTS.md` or architectural decision change is claimed in this slice.

## Revision-pinned references

- [Product publication `c7004f71cb05621dd5a7f89e46459542bd7d9c87`](https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87)
- [Changelog 0.3.358](https://github.com/guillermomolina/protos/blob/c7004f71cb05621dd5a7f89e46459542bd7d9c87/CHANGELOG.md)
- [I101-C1 independent Builder oracle and negative controls](https://github.com/guillermomolina/protos/blob/c7004f71cb05621dd5a7f89e46459542bd7d9c87/src/test/java/com/guillermomolina/protos/execution/ProtosI101CComparisonBuilderOracleTest.java)
- [PERF025 NORMAL/DEBUG regression tests](https://github.com/guillermomolina/protos/blob/c7004f71cb05621dd5a7f89e46459542bd7d9c87/src/test/java/com/guillermomolina/protos/execution/ProtosPerf025CompactCalleeExecutionTest.java)
- [I101-B historical published evidence](I101_B_357_MINIMAL_NORMAL_ARITHMETIC_BYTECODE_PUBLISHED_2026_10_14.md)
- [Live I101 work item #892](https://github.com/guillermomolina/protos/issues/892)
