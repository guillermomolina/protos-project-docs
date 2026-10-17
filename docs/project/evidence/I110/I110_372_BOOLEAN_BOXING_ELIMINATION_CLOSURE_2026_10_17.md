# I110 — Boolean boxing elimination in both Bytecode DSL roots: publication and closure evidence (2026-10-17)

**Work item:** [I110/#912](https://github.com/guillermomolina/protos/issues/912), independent implementation related to [PLAT064/#911](https://github.com/guillermomolina/protos/issues/911); no new platform decision or measured performance gate.
**Published product revision:** [`30b5c617fd135f7cc0759a34bb726fa30238f274`](https://github.com/guillermomolina/protos/commit/30b5c617fd135f7cc0759a34bb726fa30238f274) on `main`, following `e07340c2ea6ac26278a04495e145a08e3bb2cd4d`.
**Implementation version:** `0.3.372-SNAPSHOT` in the same commit's `pom.xml` and `CHANGELOG.md`.
**Publication provenance:** GitHub commit and version/source files inspected independently. The maintainer executed and reported validation and the final push; the recording agent did not rerun the tests.

## Published implementation and reviewed generated topology

1. Both `@GenerateBytecode` roots add internal `boolean.class` to `boxingEliminationTypes`, preserving `int`, `long`, `double` and all other Bytecode DSL options.
2. Primitive Java Boolean is an internal predicate carrier, not a newly observable guest value; canonical guest Boolean instances remain `ProtosBooleanValue.TRUE/FALSE`.
3. Focused regressions cover both generated root families, Boolean predicate local load/store, true/false branching, canonical guest results, and uncached-to-cached execution. Existing I075/I091, BUG018 and PERF028 affected regression coverage was run.
4. The I091 diagnostic's expected annotation is aligned with both roots. Enabling Boolean specialization changes generated dispatch shapes, so the exact generated-BCI PE topology baseline was reviewed and updated without weakening its classifier or accepting `RISK` or `UNKNOWN` transitions.
5. The review recorded **251 new** and **117 stale** topology entries relative to the previous baseline; **64 Boolean-related topology entries** were counted by the metadata finalizer. These are topology counts, not measured performance or speedup.

### Exact published files

- [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java)
- [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosSemanticBytecodeRootNode.java)
- [`ProtosI091PrimitiveCarrierRootsTest.java`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI091PrimitiveCarrierRootsTest.java)
- [`i091_boundary_review.py`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/tools/diagnostics/i091_boundary_review.py)
- [`java_generated_bytecode_bci_pe_baseline.json`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/tools/java_generated_bytecode_bci_pe_baseline.json)
- [`pom.xml`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/pom.xml)
- [`CHANGELOG.md`](https://github.com/guillermomolina/protos/blob/30b5c617fd135f7cc0759a34bb726fa30238f274/CHANGELOG.md)

## Maintainer-reported validation

- Focal Maven/JUnit regression set: **40 tests, 0 failures, 0 errors, 0 skipped**; `BUILD SUCCESS`.
- `make test`: **PASS**, including Java and Protos suites.
- `make check`: **PASS** after baseline reconciliation, including the static generated BCI PE guard and the check target's remaining gates.
- Generated topology review: **BCI RISK=0**, **UNKNOWN=0**; candidate matched the current generated roots and report, with duplicate/revision checks before baseline replacement.
- `git diff --check`: clean. Final `pom.xml`/`CHANGELOG.md` edits occurred after behavioral validation; no subsequent tests were run, consistent with the implementation finalization policy.
- Product commit/push to `main`: confirmed at the exact SHA above, **seven changed files**; the GitHub commit includes implementation, regression, diagnostics, reviewed baseline, Maven version and changelog together.

## Boundary and disposition

This is adoption of an owner-authorized normal Truffle Bytecode DSL facility, not a new Protos semantic rule, Closure/frame architecture revision, Native Image fix, compiler workaround, or performance campaign. No runtime speedup, Tier-2 benefit, Native Image acceptance or independently executed test run is claimed. [PLAT064/#911](https://github.com/guillermomolina/protos/issues/911) retains its separate investigation scope.

**Disposition:** I110 implementation, maintainer-reported full correctness gates, exact product publication and required version/changelog metadata are complete at the revision above. The [GitHub Issue](https://github.com/guillermomolina/protos/issues/912) is the live closure authority; this document is immutable revision-coupled evidence, not an additional approval or status source.
