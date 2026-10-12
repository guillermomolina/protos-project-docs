# D198 — Published normative specification 0.1.457 and ratification closure evidence

**Date:** 2026-10-12
**Decision:** [D198 #876](https://github.com/guillermomolina/protos/issues/876)
**Implementation consumer:** [I090 #868](https://github.com/guillermomolina/protos/issues/868)
**PROTOS_REVISION:** [`7903d43208a8365c6f75d4c886e5babd815754b2`](https://github.com/guillermomolina/protos/commit/7903d43208a8365c6f75d4c886e5babd815754b2)
**RATIFIED_DECISION_RECORD:** [D198 language ratification at docs commit `4b2cb5950249f3705e2797d911a9158a30ba841e`](https://github.com/guillermomolina/protos-project-docs/blob/4b2cb5950249f3705e2797d911a9158a30ba841e/docs/project/decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md)
**SPEC_REVISION:** `0.1.457`
**DOCUMENTATION_RECORD_OWNER:** `docs/project/decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md`

## Verifiable product publication facts

The published `guillermomolina/protos` default-branch HEAD is the exact `7903d43208a8365c6f75d4c886e5babd815754b2` commit, whose message is:

> D198: specify public Complex construction, arithmetic, and std:math/Math.sqrt

The GitHub commit read-back reports exactly two changed paths:

- `spec/semantics/VALUES_AND_COLLECTIONS.md` — public `Complex` construction/selectors, exact and mixed arithmetic, G1 exceptional classification, signed zeros, E2/A exact-zero behavior, ordinary `std:math/Math.sqrt`, N4/F3 branch/exactness and G2-B bound;
- `spec/PROTOS_SPEC_CHANGELOG.md` — `0.1.457` entry dated 2026-10-12, documenting the above normative delta.

Exact `0.1.457` changelog header and the A rule were read back from remote main, not inferred from the commit title. The uploaded change contains no product `pom.xml`, `CHANGELOG.md`, Java source or runtime test changes.

## Owner approval and GITHUB021

The owner's explicit design selections from D198-C/B/E2, P2/N2/R/N4/N5, F1–F4, G1/G2-B, the final three clarifications and the exact-zero-divisor A confirmation `"Apruebo A"` were recorded **before** the published normative commit. The detailed approval chain and cross-invariant review are preserved in the durable [ratification record](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md) and its linked evidence.

GITHUB021 compares D198 with D197 and D196, Float signed zeros/NaNs, family/canonical equality/hash, D013 overridable dispatch, Actor/P/Context isolation and pay-as-you-grow. **PASS**: no unapproved semantic new family, new Vector/Matrix/Real prototype, runtime carrier constraint, or altered D197 equality/identity/hash contract. The explicit A rule is an approved narrow exception to otherwise coordinate-local P2 promotion.

## Human validation versus agent execution

The owner explicitly reports after push:

- `git diff --check` — CLEAN (human-reported).
- All local tests — PASS (human-reported).
- Git push — PUBLISHED; independently corroborated by the exact GitHub remote commit and changed-path read-back.

The agent did **not** run builds or tests. In particular, previously green tests cannot be claimed as conformance evidence for the still-unimplemented I090 public Complex behaviors.

## Cross-repo publication and handoff

```text
PROTOS_REVISION=7903d43208a8365c6f75d4c886e5babd815754b2
PROTOS_COMMIT_MESSAGE=D198: specify public Complex construction, arithmetic, and std:math/Math.sqrt
SPEC_REVISION=0.1.457
PRODUCT_FILES_CHANGED=spec/PROTOS_SPEC_CHANGELOG.md,spec/semantics/VALUES_AND_COLLECTIONS.md
D198_DESIGN_GATES=OWNER_APPROVED
GITHUB021=PASS
PRODUCT_NORMATIVE_PUBLICATION=PASS
DURABLE_LANGUAGE_DECISION_RECORD=4b2cb5950249f3705e2797d911a9158a30ba841e
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
HUMAN_REPORTED_LOCAL_TESTS=PASS
AGENT_EXECUTED_TESTS=NO
D197_REOPENED=NO
CROSS_REFERENCES=PASS
D198_CLOSURE=READY_AFTER_THIS_EVIDENCE_PUBLISHED_AND_READ_BACK
I090_PUBLIC_COMPLEX_IMPLEMENTATION=READY_AFTER_D198_CLOSURE_IF_UNBLOCKED
```

The **PROJECT_RECORD_REVISION** of this evidence publication is returned by the GitHub publication tool and stored under the authoritative D198 Issue's closure entry; it cannot truthfully be its own embedded pre-publication commit SHA. Read back the exact published record before closing D198.

**Completion scope:** D198 is a language-design/specification work item. The norm was published; actual public Complex construction, arithmetic and Math.sqrt runtime implementation, conformance tests, performance and final acceptance remain the independently tracked work of I090, not D198.
