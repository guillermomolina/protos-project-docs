# I091 — PLAT056 Candidate C implementation, validation and closure evidence

**Recorded:** 2026-10-12. **Owner:** [I091/#869](https://github.com/guillermomolina/protos/issues/869). **Governing decision:** [PLAT056 Candidate C](../../decisions/platform/PLAT056_PRIMITIVE_FIRST_NUMERIC_GUEST_OBJECT_ARCHITECTURE.md). **Product publication:** [`1fc6899bbc6e18359bd84d2357bd63e78523b53c`](https://github.com/guillermomolina/protos/commit/1fc6899bbc6e18359bd84d2357bd63e78523b53c) (`0.3.341-SNAPSHOT`). **Normative specification:** `0.1.456` in the same product commit.

## Outcome and provenance

The product revision above publishes the final I091 reconciliation: primitive numeric carrier acceptance, concrete Integer/Float guest-value materialization, semantic numeric admission and equality/identity/hash, bounded Java `BigInteger` use, rich-number and isolated Actor/P transfer behavior, source-level feasibility gates, and targeted Java/Protos/Native Image regressions. The final commit changes exactly twelve paths: `pom.xml`, `CHANGELOG.md`, `tools/i091_boundary_review.py`, `build/native/test-native.sh`, `protos/tests/conformance/manifest.tsv`, `protos/tests/conformance/integer/primitive-call-boundaries.protos`, `spec/semantics/OBJECT_MODEL.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, `src/main/java/com/guillermomolina/protos/execution/ProtosStandardObjectProtocol.java`, `src/test/java/com/guillermomolina/protos/execution/ProtosComplexSemanticsTest.java`, `src/test/java/com/guillermomolina/protos/execution/ProtosI091PrimitiveCarrierPropagationTest.java`, and `src/test/java/com/guillermomolina/protos/execution/ProtosI091ParallelTransferAcceptanceTest.java`. Earlier I091 implementation commits are preserved in product history; this is the final publication, not a claim that all implementation originated in this one commit.

**Human executor reported** `make test -> PASS`, `make -C build/native test -> PASS`, and clean `git diff --check` over the substantive final delta before the version/changelog-only finalization. Earlier affected JUnit acceptance: 18 tests, 0 failures, 0 errors, 0 skipped. `pom.xml` and root `CHANGELOG.md` were changed **after** tests passed and immediately before the final commit; tests were **not repeated** after metadata-only edits, per the repository publication contract. The coordinating assistant did not run Java, Native Image, Protos tests, or compiler benchmarks. These outcomes are maintainer-reported, not reproduced CI certificates.

**Read-only independently checked in published GitHub source:** the final twelve-path publication and version `0.3.341-SNAPSHOT`; the changed Object-number guard and specification `0.1.456`; long-only `ProtosIntegerValue` storage; a small `ProtosSemanticTransferPayload.integer(long)` leaf that directly returns the `long`; `F2_DIRECT_CALL1_RESULT` and `F2_DIRECT_SEND0_RESULT` static observations; the fourth numeric conformance fixture wired into Native Image. The current production inventory numbers below are **human-run** canonical script outputs, not an independently executed crawler.

## F1–F5 final reconciliation

| Gate | Revision-grounded evidence | Bounded conclusion |
| --- | --- | --- |
| **F1 — ordinary immutable rich numbers** | Frozen rich guest object backed by the Protos object hierarchy, semantic `isCurrentNumber` recognition and canonical representation; numeric reflection/construction guards; JUnit regression families and I090's already-published Fraction/Complex support | No second generic numeric-object model is required; internal carriers and numeric object semantics remain distinct |
| **F2 — primitive paths** | Both DSL roots admit `long`/`double` boxing-elimination types, primitive send/return paths, source lowering for Call1 and Send0, local/call/result propagation, and explicit materialization on observation/fallback. Published checks `F2_DIRECT_CALL1_RESULT` and `F2_DIRECT_SEND0_RESULT` | Source/bytecode architecture supports conditional primitive propagation. **Not** a claim that every compiled site allocates zero bytes or achieves steady-state Tier-2 success |
| **F3 — normalization** | `ProtosNumericValueSupport`, long-first exact arithmetic and `ProtosBinary64Rounding`; tests for signed-64 overflow and demotion, Float NaN/signed zero, exact Fraction operations and I090 family conformance | Existing D196/D197 numerical contracts retained; I090 remains the owner of public Complex capabilities not yet selected |
| **F4 — identity, reflection and isolation** | Semantic equality/identity/hash and key coherence, explicit standard Object rejection of genuine Numbers, conditional Actor/P transfer, deferred P destination shell population with parent/child cycles, sharing, Array/Bytes/Map/IdentityMap, hashes and mutation-state publication; fresh Context and Native Image acceptance | Cross-domain and observation boundaries have focused regression coverage; no claim of formal proof over all possible graphs |
| **F5 — Java representation containment** | Exhaustive canonical production census and source-owner/per-site inventory, with ordinary long leaf transfer, long-first SPI admission and legacy interop compatibility. Every currently counted occurrence is assigned to one of 22 source owners and 204 site records | The earlier *confirmed* compulsory small-`BigInteger` SPI and transfer defects have been repaired. Remaining genuine large arithmetic, explicit host/SPI contract, exact hash and requested `asBigInteger` boundaries are not banned by policy |

The maintainer executed `python3 tools/numeric_dependency_census.py` on original review HEAD `3ff2835525cad30787a1a2b45f81936ddd18160f` (same production Java sources as final `1fc6899bbc6e18359bd84d2357bd63e78523b53c`) and reported:

```text
PRODUCTION_JAVA_FILES=441
MATCHING_FILES=22
TOTAL_REFERENCES=204
BigInteger=93
ProtosIntegerValue=68
ProtosFloatValue=43
MODULE_EXECUTION=51
MODULE_RUNTIME=148
MODULE_SPI=5
FINGERPRINT=06415edc151fdacc59bcaafd4f94487ae8ffaca09cd59df5f255da9214abe8a0
F1_F5_SOURCE_OBSERVATIONS=22/22
F5_SOURCE_OWNERS=22/22
F5_INDIVIDUAL_SITES=204/204
```

Since the final commit changes one existing production Java file (`ProtosStandardObjectProtocol.java`) and only removes the special-number `close`/`freeze` logic, it does **not** change any numeric-symbol census occurrence or require a new production census. The cited fingerprint belongs to the pre-finalization production tree; after any production-source byte change the canonical SHA-256 fingerprint **must not** be described as identical. The **204 numeric-symbol occurrences and 22 owner assignments** remain applicable because the only production change adds/removes no counted numeric identifier. The final issue evidence is therefore a combined pre-finalization census and exact final published source review, **not** a claim of a second final-SHA census execution.

Historical baseline [I091-A](I091_A_NUMERIC_CENSUS_AND_CANDIDATE_C_FEASIBILITY_CHECKPOINT_2026_10_10.md) and [full F5 reconciliation](I091_F5_COMPLETE_NUMERIC_DEPENDENCY_RECONCILIATION_2026_10_11.md): 431 production Java files, 77 matching files, 683 occurrences (`BigInteger` 447, `ProtosIntegerValue` 187, `ProtosFloatValue` 49), by module: `cli` 8, `execution` 366, `runtime` 303, `spi` 6. Final count **204**, a net reduction of **479 lexical references** (~70.1%), and `cli` 0 / `execution` 51 / `runtime` 148 / `spi` 5. This is a *lexical dependency* reduction, not a measured allocation, compilation or time improvement. The earlier [adversarial token review](I091_F5_ADVERSARIAL_BIG_INTEGER_CASE_BY_CASE_AUDIT_2026_10_11.md) flagged unconditional small-integer representation defects; its source-specific failure conclusions are historical and superseded by the product source and tests now reported.

## Final semantic clarification — Numbers are not frozen ordinary objects

The project owner explicitly selected **Reject** for inherited standard structural `Object.close()` and `Object.freeze()` on Numbers. Semantic Number immutability does not imply `FROZEN` ordinary object state. For `SmallInteger`, `BigInteger`, `Float`, `Fraction`, and `Complex`, both methods signal `Error`, including repeated calls. Structural `removeSlot`, `without` and `alias` continue to reject Number receivers. **Ordinary objects delegating through a Number** retain ordinary structural `close`/`freeze` state transitions. This owner choice is normative in `spec/semantics/OBJECT_MODEL.md` at spec `0.1.456`; the matching Java and Protos tests are in the final product commit. No unapproved change to PLAT056 is inferred.

## Closure boundary and handoffs

**I091 scope is implementation/structural architecture and conformance, not the open benchmark measurements.** PERF040/#862 owns pinned Graal IR/assembly/allocations/steady-state timing and any measured optimization or Tier-2 claim. Existing PERF042/PERF032 follow their own graph acceptance. **I090/#868** retains its independently blocked public Complex interface/mathematics pending **D198/#876** ratification; I091 does not close, unblock or change that gate. I093/#873 owns Protos-first network↔NIO encapsulation; this closure does not claim to refactor that independent host layer.

**Acceptance judgement:** implementation gates F1–F5 are supported at the structural/source level and by human-reported affected and integrated/native tests; no remaining **confirmed** I091-owned production defect is identified in this closure review. The retained exceptions and comparative performance matters are assigned to their existing owners, not hidden as `PASS`. The formal GitHub [I091 issue](https://github.com/guillermomolina/protos/issues/869) remains the live closure authority; this immutable report is revision-coupled evidence, not independent authority to declare it closed.

```text
FORMAL_IDENTIFIER=I091
PLATFORM_DECISION=PLAT056_C_RATIFIED
PROTOS_REVISION=1fc6899bbc6e18359bd84d2357bd63e78523b53c
IMPLEMENTATION_VERSION=0.3.341-SNAPSHOT
SPEC_REVISION=0.1.456
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_NATIVE_TEST=PASS_REPORTED
HUMAN_DIFF_CHECK=CLEAN_REPORTED
F1_F5_STATIC_SOURCE_OBSERVATIONS=22/22
F5_SOURCE_OWNERS=22/22
F5_CLASSIFIED_OCCURRENCES=204/204
GRAAL_PERF_MEASUREMENT=NOT_CLAIMED_PERF040
PROJECT_RECORD_REVISION=SEE_FINAL_ISSUE_PUBLICATION_RECORD
```
