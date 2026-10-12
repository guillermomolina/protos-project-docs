# D198 — Owner approval of three final normative clarifications

**Date:** 2026-10-12
**Decision:** [D198 #876](https://github.com/guillermomolina/protos/issues/876) — selected design, normative publication and closure still pending.
**Implementation consumer:** [I090 #868](https://github.com/guillermomolina/protos/issues/868).
**Prior selection:** [G1/G2-B owner approval](D198_G1_G2_B_OWNER_APPROVAL_AND_GITHUB021_2026_10_12.md).
**Source/implementation revision:** `guillermomolina/protos` HEAD `18f71b2a664c56ac808e2f508b6369984de02280` when this coordination was prepared; no Protos product or spec changes made here.

## Exact owner approval provenance

During the human-executor **D198-NORM** specification-authoring handoff, the agent surfaced three remaining observable ambiguities: **real ÷ Complex**, **sqrt(Complex(-Infinity,NaN))**, and the reach of the **G2-B 4-ULP sqrt bound for real inputs**. The owner confirmed that the first two questions already matched the assistant's recommendations, showed the third question and the assistant recommended its first option, then explicitly stated:

> "ok pues aprobadas las 3 recomendaciones para ti tambien"

This is explicit approval of precisely the three choices presented in that conversation. It does **not** re-open D197, change the approved G1/G2-B semantics beyond these exact refinements, authorize new Vector/Matrix APIs, or constitute publication of a normative spec.

## Q1 — Real divided by Complex: G1 branches without a synthetic imaginary numerator

**APPROVED:** choose **"G1 branches, no b terms (Recommended)"**, the first option of the agent's real÷Complex decision question. A genuine scalar-real numerator contributes **no imaginary component term**. Do not manufacture `Complex(r,+0.0)` to determine signed zeros or exception results.

- Preserve G1 priority, P2 operand-first domain selection, R finite coupled precision and F1 signed-zero reference rules where applicable.
- For a real scalar `r` and Complex divisor `c+di` with finite nonzero divisor, define the mathematical coordinates directly: `real = (r*c)/(c²+d²)`, `imaginary = -(r*d)/(c²+d²)` and correctly round once per Float coordinate after approved conversions. Formulae describe the result and signs, **not mandatory intermediate binary64 lowering**.
- For finite Float scalar zero, use IEEE product and negation signs in this direct real÷Complex expression. Explicit discriminator:
  `+0.0 / Complex(1.0,1.0) -> imaginary -0.0`
  (not `+0.0` produced by a fabricated imaginary `+0.0` numerator).
- For a Float-domain zero Complex divisor, preserve G1's signed-real-zero `k` rule with `Complex(k*r,NaN)`. Thus `1.0 / Complex(+0.0,+0.0) -> Complex(+Infinity,NaN)` remains binding.
- For infinity/infinity and infinity/finite or finite/infinity, follow G1's coupled numerical classification **adapted to real-only numerator without imaginary terms**.
- All-exact division by a mathematically exact-zero divisor remains an `Error` under E2; wrong-arity/type/receiver errors follow their independent protocol.
- No new operator, special syntax or implicit Complex conversion is approved.

## Q2 — sqrt(-Infinity,NaN): deterministic canonical-NaN convention

**APPROVED:** the recommendation for the previously unspecified nonfinite corner:

```text
sqrt(Complex(Float(-Infinity), Float(NaN)))
    -> Complex(Float(NaN), Float(+Infinity))
```

The semantic Float NaN has no observable sign or payload. Choose the **positive** imaginary infinity deterministically rather than depending on an unobservable NaN sign, a host complex sqrt convention, or a platform implementation detail. This rule refines the N4 special-value partition **only for the exact (-Infinity,NaN) input class**; preserve all other N4 sqrt special cases, signed-zero branch cuts, and Complex canonicalization. It is not a new general NaN-contagion rule.

## Q3 — sqrt finite inexact accuracy: 4 ULP applies also to real inputs

**APPROVED:** select **"4 ULP for all sqrt coords (Recommended)"**, the first option in the real-sqrt decision question.

- Apply the same G2-B bound of **at most 4 representable binary64 ULP steps from an independently correctly rounded exact mathematical principal-square-root oracle** to **every finite inexact Float coordinate of a `sqrt` result**, whether the input is a real Integer, Fraction, Float, or a genuine Complex value.
- Examples under this single envelope: Float result of `sqrt(2)` and `sqrt(2.0)`, imaginary Float result of `sqrt(-2)`, and each inexact Float coordinate of `sqrt(Complex(3.0,5.0))`.
- Correct rounding remains **allowed but not universally required**, including real-input square roots; this is not the alternative stricter contract "correctly rounded for real input".
- **Existing exactness invariants prevail:** rationally representable results from exact inputs stay exact (`sqrt(4)` is exact Integer 2, `sqrt(1/4)` is exact Fraction 1/2, `sqrt(Complex(3,4))` is exact Complex(2,1)); Float-family inputs remain in Float-domain outputs under previously approved N4/P2/F3, and the ordinary sqrt branch cuts/nonfinites are unchanged.
- The 4-ULP allowance does **not** relax exact required signed zeros, nonfinite special-result tuples, canonicalization, avoidance of spurious intermediate NaN/Infinity, or gradual subnormal underflow.
- No implementation-compliance claim follows from approving this quality contract; independent-oracle tests belong to the later I090 implementation.

## GITHUB021 approval-boundary / invariant-delta review

| Prior binding invariant | Proposed delta and approved status |
| --- | --- |
| D197 hierarchy, Complex scalar-not-Vector, authentic components and canonical zero demotion | PRESERVED; real÷Complex does not create a semantic synthetic Complex |
| P2, R and F1 | PRESERVED; Q1 specifies absent-imaginary-numerator semantics and signed zeros without changing Float promotion |
| E2 and G1 | PRESERVED; Q1 selects the formerly undetermined real÷Complex domain and the discriminator `+0/(1+i)` |
| N4 special-value sqrt and Float NaN identity | REFINEMENT EXPLICITLY APPROVED: `sqrt(Complex(-Inf,NaN))=Complex(NaN,+Inf)`, rather than leaving infinity sign to host |
| G2-B finite complex sqrt at most 4 ULP | EXTENDED EXPLICITLY APPROVED: all inexact Float sqrt result coordinates, including real inputs, use the same 4-ULP bound |
| F3 exact roots and rational-coordinate preservation | PRESERVED; no Float tolerance for mathematically exact rational outputs |
| D013 overrideable messages, Actor/P/Context transfer, numeric ==/===/hash, pay-as-you-grow | PRESERVED |
| D198 full normative spec already published? | **NO**. Approval selected, publication pending. |

No earlier approved invariant is silently replaced. The two newly observable signed-result choices and the broadened 4-ULP conformance scope have **specific explicit owner approval provenance**. Avoid using this record to imply all design details were already written normatively.

## Coordination outcome

```text
D198_REAL_DIV_COMPLEX_NO_SYNTHETIC_IMAG=OWNER_APPROVED
D198_SQRT_NEG_INF_NAN=COMPLEX_NAN_POS_INF_OWNER_APPROVED
D198_SQRT_4ULP_SCOPE=ALL_INEXACT_FLOAT_RESULT_COORDINATES_OWNER_APPROVED
D198_APPROVED_DESIGN_CANDIDATE=COMPLETE
D198_NORMATIVE_SPEC_PUBLICATION=PENDING
D198_ISSUE_CLOSURE=PENDING
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED_UNTIL_NORMATIVE_PUBLICATION
SPEC_CHANGED=NO
PRODUCT_CHANGED=NO
TESTS_EXECUTED=NO
```

This is **non-normative evidence** in `protos-project-docs`, not a spec revision. The other agent may now complete D198-NORM in `guillermomolina/protos` without re-questioning these three selected semantics. Earlier issue/work entries describing them as unresolved are superseded **only as to approval state**, not as historical investigative statements.
