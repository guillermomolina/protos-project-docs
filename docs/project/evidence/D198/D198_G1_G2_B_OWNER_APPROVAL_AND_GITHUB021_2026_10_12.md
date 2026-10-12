# D198 — Owner approval of G1 and G2-B (GITHUB021)

**Date:** 2026-10-12
**Live decision:** [D198 #876](https://github.com/guillermomolina/protos/issues/876)
**Implementation owner:** [I090 #868](https://github.com/guillermomolina/protos/issues/868)
**Decision:** G1 and G2-B **OWNER APPROVED**, with normative-spec publication and final D198 closure still pending.

## Approval provenance

The project owner explicitly clarified, after being informed that G1 and G2-B remained pending: **"te dije que aprobaba todas, pense que ya lo habias registrado"**. This confirms approval of both recommendations, rather than only the earlier P2/N2/R/N4/N5 and F1–F4 groups. The earlier agent left this approval unrecorded; this document corrects that coordination omission. It does not fabricate an earlier verbatim approval identifying G1/G2-B by name.

**Exact candidate source:** [D198 closing investigation](D198_CLOSING_NONFINITE_MATRIX_AND_SQRT_CONFORMANCE_RESEARCH_2026_10_12.md). Its G1 and G2-B technical contents are selected in full, including explicitly identified new observable consequences; no alternative algorithms or unstated choices are implied.

## G1 approved — exhaustive exceptional-result classifier

G1 uses the four categories Z (signed zero), F (finite nonzero), I (signed Infinity), N (canonical NaN), with P2's operation-local Float promotion and ordinary D013 dispatch preserved.

- Addition, subtraction and unary negation are separable by coordinate: an incoming NaN does not contaminate an independent coordinate automatically.
- Genuine **Complex × Complex**: any actual incoming NaN gives `Complex(NaN,NaN)`; four finite components use R's correctly rounded coupled mathematical product and F1's signed zeros; infinities with no input NaN use the specified IEEE extended rectangular rules, including the approved F2-A case `C(+Inf,+0)*C(+0,1)=C(NaN,+Inf)`.
- Genuine **Complex ÷ Complex**: the ordered G1 branches are incoming NaN, finite/finite nonzero quotient under R/F1, Float-domain complex-zero divisor under N2, infinity/infinity, infinity/finite, and finite/infinity with exact directional signs. All-exact mathematical zero divisors still produce `Error` under E2.
- G1's conservative NaN generalization applies to genuinely coupled multiplication/division, not to independent addition/subtraction nor the separately approved N4 square-root table.
- G1 selects the exceptional **real ÷ Complex** result convention explicitly proposed in the closing packet, including `1.0 / C(+0.0,+0.0) -> C(+Inf,NaN)`. Do not model the real-scalar path by silently creating a synthetic Complex.
- Signed zeros, canonical NaN, exact-zero imaginary demotion and other D197 invariants remain intact. All sign refinements and the complete rule-generator text are owned by the linked exact candidate; conformance enumeration/tests remain future work.

## G2-B approved — inexact complex sqrt conformance

For finite inexact complex square roots, each inexact binary64 result coordinate must be **within 4 ULP** of an *independently computed correctly rounded reference* for the exact mathematical principal root. The approved contract additionally requires:

- rationally expressible coordinates of exact inputs stay exact under F3;
- oracle-zero results have precisely the required signed zero, including cases rounded to zero from nonzero mathematical values;
- no spurious NaN/Infinity or flush-to-zero caused by intermediate numerical instability;
- preservation of the N4 principal branch, signed-zero cuts and all approved NaN/Infinity cases;
- no universal correctly-rounded complex-sqrt guarantee beyond this 4-ULP bound;
- ordinary exported `std:math/Math.sqrt(value)` under F4, without new intrinsic syntax.

Implementation compliance with the 4-ULP bound is **not** established by this owner approval; independently certified reference testing is required later.

## GITHUB021 invariant/delta audit

| Previously approved invariant | G1/G2-B |
| --- | --- |
| D197 Number numeric tower and Complex authentic exact/Float components | Preserved |
| D197 exact-zero imaginary demotion versus inexact signed zero; equality, identity and hash | Preserved |
| D196/D197 binary64 scalar, operand-first Float promotion | Preserved; R governs genuinely coupled complex arithmetic |
| D198 C/B/E2; D013; pay-as-you-grow; Actor/P/Context | Preserved |
| P2/N2/R/N4/N5 and F1/F2-A/F2-B/F3/F4 | Preserved and completed by the explicitly selected generalizations |
| Complex is a scalar Number, not a Vector/Matrix | Preserved; no generic algebra API is selected |
| G1 new NaN/zero/infinite tuples | Explicitly owner approved |
| G2-B quantitative 4-ULP bound | Explicitly owner approved |

No existing approved invariant needs to be reopened for G1 or G2-B. This GITHUB021 check is a decision-level consistency check, **not** a claim of normative-spec or implementation conformance.

## Boundary and next step

```text
D198_G1=OWNER_APPROVED
D198_G2_B=OWNER_APPROVED
D198_SEMANTIC_SELECTION=COMPLETE
D198_NORMATIVE_SPEC_PUBLICATION=PENDING
D198_FINAL_CLOSURE=PENDING
I090_COMPLEX_REMAINDER=BLOCKED_UNTIL_NORMATIVE_PUBLICATION
D197_REOPENED=NO
PRODUCT_CHANGED=NO
SPEC_CHANGED=NO
TESTS_EXECUTED=NO
```

No product files, `spec/`, `pom.xml` or `CHANGELOG.md` are modified by this coordination action. D198 remains open only for publication/closure gates. Earlier proposal documents saying G1/G2-B were unapproved are historical states superseded by this explicit approval.
