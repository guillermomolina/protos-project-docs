# I091-A — Numeric dependency census and Candidate C feasibility checkpoint

**Date:** 2026-10-10.  
**Issue:** [I091/#869](https://github.com/guillermomolina/protos/issues/869).  
**Selected architecture:** [PLAT056/#866](https://github.com/guillermomolina/protos/issues/866), ratified Candidate C.  
**PROTOS_REVISION:** [`76de43651079172d90be5bc6b1f535336ae47cfb`](https://github.com/guillermomolina/protos/commit/76de43651079172d90be5bc6b1f535336ae47cfb).  
**Parent published mixed-arithmetic baseline:** [`eba6250f96aa6873829f7011a3a1aee7b6969887`](https://github.com/guillermomolina/protos/commit/eba6250f96aa6873829f7011a3a1aee7b6969887), I089/#867.  
**No implementation version change:** this checkpoint changes only two audit tools and one JUnit test file, not production code or normative specification.

## Published paths and source-grounded findings

GitHub commit inspection verifies exactly three changed paths:
- `tools/numeric_dependency_census.py` — reproducible full `src/main/java` lexical census, excluding Java comments/string literals; JSON and TSV emitted to `target/`. Fingerprint pins the exact source content.
- `tools/i091_boundary_review.py` — deterministic review queues and static preflight checks. Category names are ownership queues, not claims that dependencies are gratuitous or removable.
- `src/test/java/com/guillermomolina/protos/execution/ProtosSemanticTransferFamilyTest.java` — two test-only Candidate C feasibility fixtures for frozen numeric-rich state/IEEE signed-zero across Actor/P and the presently missing current numeric identity/recognition integration.

**Owner-reported census from the pre-change production tree at `eba6250f96aa6873829f7011a3a1aee7b6969887`:**
- 431 production Java files inspected, 77 with matches, 683 numeric-representation references.
- `BigInteger`: 447 references; `ProtosIntegerValue`: 187; `ProtosFloatValue`: 49.
- Source fingerprint: `605cd68c3e75bbc16cab54bbb89770161f2e5f59b21b0c77875e9c01f68f8034`.
- Review queue counts: IO/network 208, numeric algorithms/carriers 130, foreign/host interop 84, collections/indexing 64, manual ownership review 56, semantic identity/equality/hash 53, tools/library support 44, isolation/transfer 36, Bytecode execution 8.
- 12/12 named static preflight observations reported OBSERVED, not 12/12 feasibility gates accepted. All 683 occurrences remain **UNCLASSIFIED** pending source-level justification.

**F1 partial feasibility:** `ProtosSemanticTransferValue` is a frozen ordinary-object-derived holder for private exact state, with existing PLAT051 authorized `std:`-family rematerialization and Actor/P infrastructure. Test-only rich numeric content used a large exact numerator, denominator, and negative IEEE zero; the owner reports the focused JUnit fixture PASS. This is not evidence that PLAT051 `std:` transfer can automatically represent canonical Core BigInteger/Fraction/Complex; separate bootstrap/family authorization, semantic identity-by-value, cross-context and Process integration still need proof.

**F2 observed obstacle:** both Bytecode DSL roots currently have `boxingEliminationTypes={int.class}`. `ProtosNumberLiteral` creates wrapper instances during source lowering, and successful exact small arithmetic produces `new ProtosIntegerValue` in the current canonical operation path. Changing the root annotation alone will not establish an unboxed long/double end-to-end path.

**F4 observed obstacle:** current `ProtosIdentity`, equality/ordering/hash protocols and receiver recognition are tied to existing Java numeric representations. The test-only new value intentionally exposes this gap without claiming a new guest-visible D197 family.

## Validation provenance

The human executor reported:
- Initial exhaustive census and boundary preflight completed, counts listed above.
- `git diff --check` clean.
- Focused `mvn -q -Dtest=ProtosSemanticTransferFamilyTest test`: PASS.
- Subsequent overall report: all local tests PASS; exact invocation and complete count were not supplied.
- GitHub verifies publication of all three named paths in the exact product commit above.

Tests and runtime performance were **not** executed by this evidence author. No full-suite command, benchmark, numeric acceptance or D197 semantic change is inferred.

## Continuation and stop gates

I091 remains **OPEN**: this checkpoint establishes the baseline and partial data/transfer feasibility only. Next coherent implementation work must establish a bounded current-semantic numeric service and bridge physical carriers to semantic identity, hash and transfer without smuggling new public recognition semantics. Preserve I089 D196 B1 mixed binary64 behavior. Before normative I090 work, the public `Integer.recognizes`/exact-integral domain contract still needs an explicit design decision. Candidate B is not authorized as automatic fallback. Full integrated regression and classified before/after census remain pending.

**PROJECT_RECORD_REVISION:** the commit that creates this immutable evidence document (reported separately by the publication response).
