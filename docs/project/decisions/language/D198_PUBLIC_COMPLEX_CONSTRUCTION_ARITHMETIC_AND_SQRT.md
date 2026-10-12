# D198 — Public Complex construction, arithmetic and mathematical sqrt

**Decision:** RATIFIED — all owner-approved bounded proposals and refinements published normatively on 2026-10-12.
**Live design owner:** [D198 / guillermomolina/protos#876](https://github.com/guillermomolina/protos/issues/876)
**Implementation consumer:** [I090 / guillermomolina/protos#868](https://github.com/guillermomolina/protos/issues/868)
**Independent governing hierarchy:** [D197 — Exact numeric tower and canonical normalization](D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md), closed.
**Normative authority:** `guillermomolina/protos/spec/semantics/VALUES_AND_COLLECTIONS.md`
**Normative specification revision:** `0.1.457`, `spec/PROTOS_SPEC_CHANGELOG.md`
**Exact product publication:** [`7903d43208a8365c6f75d4c886e5babd815754b2`](https://github.com/guillermomolina/protos/commit/7903d43208a8365c6f75d4c886e5babd815754b2)
**Project-record publication:** This file is the durable *non-normative* ratification record. Its exact commit SHA is reported in live D198 Issue closure after publication.

> Only the published `spec/` owns Protos-observable semantics. This record makes owner decisions, rationale and handoff auditable; it is neither a second specification nor proof of runtime conformance.

## Approval chain and final selection

Owner-selected decisions are recorded in the D198 evidence corpus. The selection is not an agent extrapolation or a generic approval of unspecified behavior.

1. **C** — Protos-first public behavior and minimal justified Java numeric kernel; ordinary overridable D013 dispatch, Actor isolation and pay-as-you-grow preserved.
2. **B** — authenticate Float NaN, ±Infinity and signed-zero as eligible genuine Complex components.
3. **E2** — an all-exact Complex division by exact zero signals Protos `Error`; Float-domain exceptional arithmetic produces numeric results without Error solely from IEEE exceptions.
4. **P2, N2, R, N4 and N5** — coordinate-local vs coupled Float promotion, nonfinite result examples, once-rounded finite coupled multiplication/division, principal sqrt and exact roots, ordinary public factory, selectors and arithmetic.
5. **F1, F2-A/B, F3 and F4** — coordinate zero signs, conservative infinity/NaN product classification, exactness of rational sqrt components and no premature Float conversion, ordinary `std:math/Math.sqrt` export.
6. **G1 and G2-B** — complete deterministic exceptional-result classification for coupled Complex operations and up-to-4-ULP accuracy for inexact sqrt coordinates, explicitly confirmed by the owner after the earlier incomplete recording.
7. **Three final authoring clarifications**, explicitly owner-approved: real ÷ Complex has no fabricated numerator imaginary component; `sqrt(C(-Infinity,NaN))=C(NaN,+Infinity)`; 4-ULP bound includes sqrt of real inputs.
8. **A**, explicitly selected with **"Apruebo A"**: Float-bearing Complex ÷ exact Integer zero converts the zero divisor to Float `+0.0`, converts any remaining exact numerator components to Float before independent per-coordinate IEEE divisions. Both exact components ÷ exact zero continue to signal `Error`.

Exact provenance and GITHUB021 invariant/delta records:

- [C architecture](../../evidence/D198/D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md), [B nonfinite components](../../evidence/D198/D198_B_NONFINITE_FLOAT_COMPONENT_ADMISSION_OWNER_APPROVAL_2026_10_12.md), [E2 failure envelope](../../evidence/D198/D198_E2_EXCEPTIONAL_ARITHMETIC_OWNER_APPROVAL_2026_10_12.md).
- [F1–F4 refinement and P2/N2/R/N4/N5 approvals](../../evidence/D198/D198_F1_F4_RECONCILIATION_OWNER_APPROVAL_2026_10_12.md).
- [G1 and G2-B explicit owner approval](../../evidence/D198/D198_G1_G2_B_OWNER_APPROVAL_AND_GITHUB021_2026_10_12.md), [final three clarifications](../../evidence/D198/D198_THREE_FINAL_SEMANTIC_CLARIFICATIONS_OWNER_APPROVAL_2026_10_12.md), [A mixed division exact-zero approval](../../evidence/D198/D198_MIXED_COMPLEX_DIVISION_EXACT_ZERO_OWNER_APPROVAL_2026_10_12.md).
- [Complex scalar is not Vector/Matrix](../../evidence/D198/D198_COMPLEX_SCALAR_VS_VECTOR_MATRIX_OWNER_APPROVAL_2026_10_12.md).
- [D198 maintained work/decision timeline](../../work/D198/D198_PUBLIC_COMPLEX_API_DESIGN_GATE_AND_I090_RECONCILIATION_2026_10_11.md).

The source investigations and earlier pending markers are historic stage evidence; later explicit approvals supersede those pending statuses, not the previous investigation's facts.

## Published semantic boundary (digest, not substitute for spec)

- `Complex` is an authentic immutable **scalar Number family**, distinct from Vector/Matrix. Genuine Integer/Fraction/Float components are preserved; exact Integer-zero imaginary collapses to real, but retained Float ±0.0 preserves Complex.
- Public `Complex(real,imaginary)` is an ordinary canonical `call` factory. `Number.real()` and `Number.imaginary()` are ordinary selectors. Real receivers return themselves and exact Integer zero for imaginary. `Integer(...)` and `Float(...)` still reject Complex arguments. No new literal, `Real`, Matrix, Vector or algebra superclass.
- Addition/subtraction, negation and Complex ×/÷ real use separable real operations, with the specifically approved A exact-zero-divisor exception. Coupled Complex × Complex, Complex ÷ Complex and real ÷ Complex have exact-rational results for exact inputs, operand-first Float promotion where specified, R once-rounded binary64 finite output coordinates, F1 signed zeros, and G1 ordered NaN/infinity/zero direction classification. Real ÷ Complex never materializes a synthetic imaginary numerator.
- `std:math/Math` exports ordinary `sqrt(value)`, with principal complex branch and N4 exceptional table, rational roots and rational coordinates exact under F3, no premature conversion of large exact radicands, and G2-B maximum **4 ULP per inexact Float output coordinate** against an independent correctly rounded mathematical oracle, including real inputs. Exactly specified signed zeros and special values remain exact.
- D197 equality/identity/hash, no Complex ordering, Actor/P/Context transfer, ordinary overridden D013 methods and no hidden callback/suspension are unchanged.

## GITHUB021 invariant/delta closure check

| Pre-existing binding decision | Ratified D198 interaction |
| --- | --- |
| D197 authentic families, exact normalization, Complex identity/hash and numeric equality | Preserved; D197 remains closed |
| D196/ D197 operand-first mixed Float arithmetic and P2 separability | Preserved except explicitly approved A narrow exact-zero divisor exception |
| E2 exact Error / Float exception numerical result | Preserved; A resolves mixed exact-zero ambiguity |
| N2/F2-G1 signed exceptional tuples | Preserved; generalized G1 and three signed-edge clarifications explicitly approved |
| N4/F3/G2-B sqrt principal branch, exactness and portable error ceiling | Preserved; real-input G2-B scope expressly approved |
| D013 overridable dispatch, Actor/P/Context, pay-as-you-grow | Preserved; no implementation representation or fixed runtime cost mandated |
| Complex independent of Vector/Matrix and no new Real prototype | Preserved |

**GITHUB021=PASS** for the published normative selection. The normative product commit is an exact read-back, not a test-implementation claim.

## Completion boundary and downstream work

```text
D198_DESIGN_OWNER_APPROVAL=EXPLICIT_ALL_GATES
D198_SPEC_REVISION=0.1.457
PROTOS_REVISION=7903d43208a8365c6f75d4c886e5babd815754b2
D198_NORMATIVE_PUBLICATION=VERIFIED
D198_DECISION_RATIFICATION=COMPLETE_ON_DURABLE_RECORD_PUBLICATION
D197_REOPENED=NO
D198_GITHUB021=PASS
I090_PUBLIC_COMPLEX_IMPLEMENTATION=REMAINS_TO_DO
HUMAN_REPORTED_GIT_DIFF_CHECK=PASS
HUMAN_REPORTED_LOCAL_TESTS=PASS
AGENT_EXECUTED_TESTS=NO
```

The human reports `git diff --check` clean and all local tests PASS after the normative-only commit. Those checks are human-reported, not freshly executed or independently proven to cover not-yet-implemented Complex runtime functionality.

Following this record's publication and read-back, D198 may be closed `status:completed`, and I090 may be moved out of `status:blocked` to `status:ready` **if no other live blocker remains**. Publication of a language specification alone does not satisfy I090's runtime/test acceptance criteria or close I090.
