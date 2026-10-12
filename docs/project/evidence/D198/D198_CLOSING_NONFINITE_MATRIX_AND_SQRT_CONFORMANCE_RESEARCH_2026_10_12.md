# D198 — Closing investigation: exhaustive Complex nonfinite rule generator and sqrt conformance

**Date:** 2026-10-12. **Type:** RESEARCH ONLY / UNAPPROVED REFINEMENT. **Owner:** [D198/#876](https://github.com/guillermomolina/protos/issues/876). **Consumer:** [I090/#868](https://github.com/guillermomolina/protos/issues/868). **Product reference:** \`18f71b2a664c56ac808e2f508b6369984de02280\`. **Docs base before publication:** \`4544ac12f968213ae50239583d6025247628954e\`. No Protos code, normative spec, conformance tests or benchmarks executed or modified.

**Approval boundary:** Previously explicitly approved D197, D198-C/B/E2, P2/N2/R/N4/N5 and F1/F2-A/F2-B/F3/F4 remain binding. This document *proposes* only an exhaustive deterministic generator (G1) and inexact sqrt tolerance (G2); do NOT treat the suggestions as approved or close/unblock issues. The decision's role-first work record is at \`docs/project/work/D198/D198_PUBLIC_COMPLEX_API_DESIGN_GATE_AND_I090_RECONCILIATION_2026_10_11.md\`.

## Grounded precedents and divergence

- [WG14 N1399](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1399.htm): 4 kinds per component (zero, finite nonzero, infinity, NaN); 4^4 = 256 classes per coupled operator *before signs*, competing axis/direction assumptions; naive Annex G can produce wrong tuples. Research evidence, not Protos authority.
- [CPython complexobject.c](https://github.com/python/cpython/blob/main/Objects/complexobject.c): C double-carrier, Smith-like scaled division, selective infinity recovery, and Python zero-complex-divisor exception; not compatible as a whole with D197/E2/R.
- [Apache Commons Numbers Complex](https://commons.apache.org/proper/commons-numbers/commons-numbers-docs/apidocs/org/apache/commons/numbers/complex/Complex.html): distinguishes complex-real from complex-complex operations to protect signed zeros; uses scaled sqrt and C99 special cases; its 1-ulp *abs()* statement MUST NOT be misquoted as a sqrt bound.
- [C++ complex sqrt](https://en.cppreference.com/w/cpp/numeric/complex/sqrt): branch cuts, signed zero, NaN/infinite table.
- [musl csqrt source](https://fossies.org/dox/emscripten-6.0.10/csqrt_8c_source.html): scaled numerical sqrt computation and explicit special cases. Does not itself certify a portable universal componentwise ULP bound.
- [Baudin & Smith](https://arxiv.org/abs/1210.4539): Smith division algorithm can fail from overflow/underflow; robust scaling alone is not a substitute for D198-R's correct-rounding reference.

## G1 candidate — total exceptional result generator (NOT APPROVED)

Notation \`C(a,b)\` means authentic Complex source construction; canonicalize every output under D197. \`Z\` = finite signed zero; \`F\` = finite nonzero; \`I\` = infinite; \`N\` = canonical NaN. Original receiver/argument family plus P2 promotion determines the branch before classifying. Real operands use the approved scalar-real paths and must NOT silently be expanded into a genuine C(real,+0.0) and evaluated using coupled rules. The tables below govern ordinary *standard* numeric fallback, not an override of D013 user methods.

**Separable operators**: \`+\`, \`-\`, and unary sign negation use existing componentwise exact or IEEE binary64 rules. NaN is confined to affected coordinates; Float zero signs and exact imaginary-zero normalization are preserved. Independent real-scalar operations touch only semantically participating coordinates.

**Coupled multiplication** after P2-required operand-first conversion (for every genuine Complex × Complex with a Float coordinate):
1. If any actual input component is NaN: return C(NaN,NaN) (generalization of approved F2-B; unapproved for other tuples).
2. If all four are finite Z/F: exact mathematical (ac-bd, ad+bc), each coordinate correctly rounded once to binary64, F1 signs for an exactly-zero coordinate; no spurious intermediate overflow/underflow, including double cancellation.
3. Otherwise there is at least one Infinity and no NaN: determine both outputs by IEC60559 binary64 extended rectangular arithmetic \`a*c-b*d\` and \`a*d+b*c\`, with Infinity*0 = NaN, Inf-Inf = NaN and canonicalized NaNs. This reproduces F2-A \`C(+Inf,+0)*C(+0,1)=C(NaN,+Inf)\`. Do not use finite overflow-recovery routines to suppress genuine infinity invalids.

**Coupled division**, after P2 conversion, in *priority order*:
1. Any actual input NaN => C(NaN,NaN), without obscuring normal wrong-type/arity Error (unapproved generalization of conservative F2).
2. Numerator and denominator finite, denominator not mathematically zero => exact mathematical complex quotient, both coordinates rounded once to binary64 under R/F1; may legitimately overflow/underflow, never spurious NaN from intermediate overflow.
3. Denominator mathematically zero with Float in domain => \`k=copySign(+Inf,c)\`, where \`c\` is divisor's Float-converted signed real zero, and output C(k*a,k*b) with IEEE binary64 multiplications; imaginary zero sign \`d\` does not affect \`k\`. Illustrative already approved N2 tuples: \`C(1,2)/C(+0,+0)=C(+Inf,+Inf)\`, \`C(1,2)/C(-0,+0)=C(-Inf,-Inf)\`, \`C(0,2)/C(+0,+0)=C(NaN,+Inf)\`. All-exact divisor zero remains Error *before* this branch.
4. Infinity in both numerator and denominator => C(NaN,NaN), if no prior NaN/zero-divisor branch; matches approved N2.
5. Infinite numerator, finite nonzero denominator: build directional \`A=sign(a)×1\` if a infinite, else signed Float zero from a; \`B\` analogously. Obtain *exact*, overflow-free signs of \`X=A*c+B*d\`, \`Y=B*c-A*d\` with F1 zero-term conventions. Return C(+Inf * X, +Inf * Y): signs of nonzero X/Y determine signed infinities; exactly zero coefficients yield NaN. The \`* \` notation in the semantic output is an IEEE sign/classification rule, not mandated intermediate lowering.
6. Finite numerator, infinite denominator: build \`C=sign(c)×1\` if c infinite, else its signed Float zero; \`D\` similarly. Determine exact signs of \`X=a*C+b*D\`, \`Y=b*C-a*D\` with F1 zero-term signs. Return C(+0.0*X,+0.0*Y), including signed zeros as specified. E.g. \`C(1,2)/C(+Inf,-Inf)=C(-0.0,+0.0)\`.
7. No cases remain: every four-component Z/F/I/N tuple matched one of the above; the signed-zero bit refines Z but does not add an uncovered class. This is a *generative table*, not an unverifiable claim that 512 hand-written vectors were tested.

For ***real / Complex*** inexact zero division, a distinct scalar/complex branch must spell out its missing imaginary-coordinate convention; one proposed result is \`1.0 / C(+0.0,+0.0) = C(+Inf,NaN)\`, but this tuple is **NOT already owner-approved**. Do not infer it from a manufactured Complex numerator or publish until expressly selected. Same caveat for commutativity and signs at mixed scalar/complex exceptional boundaries.

**Important precision limit:** R's correctly rounded guarantee governs *finite* coupled ×/÷ mathematical results. Extended Infinity/NaN rules above are explicit semantic classification rules rather than ordinary algebraic equality over infinities. Finite converted inputs whose exact output is infinite on rounding produce signed Infinity per binary64. Signed Float -0 and +0 never collapse to an exact Integer-zero imaginary component.

### Generator coverage and counterexamples

| Class/example | Expected status/output under existing approvals + proposed G1 |
| --- | --- |
| C(1,2) / C(0,0) | exact Error [approved E2; constructor C(0,0) normalizes to integer zero] |
| C(1,2) / C(+0.0,+0.0) | C(+Inf,+Inf) [approved N2] |
| C(1,2) / C(-0.0,+0.0) | C(-Inf,-Inf) [approved N2] |
| C(0,2) / C(+0.0,+0.0) | C(NaN,+Inf) [approved N2] |
| C(+Inf,+0.0) * C(+0.0,1.0) | C(NaN,+Inf) [approved F2-A] |
| C(NaN,+Inf) * C(1.0,1.0) | C(NaN,NaN) [approved F2-B] |
| C(1,2) / C(+Inf,-Inf) | C(-0.0,+0.0) [approved N2] |
| C(1,+0.0) / C(1,+0.0) | finite R/F1: C(1.0,+0.0) [proposed zero-sign oracle] |
| C(+Inf,+Inf) / C(+Inf,-Inf) | C(NaN,NaN) [approved N2 category] |
| C(NaN,1) + C(2,3) | component-local C(NaN,4) [P2/N2 separable] |
| C(1,2) * real 0.0 | component-local (0.0,0.0); not coupled complex multiplication [P2 scalar path] |
| real 1.0 / C(+0.0,+0.0) | proposed C(+Inf,NaN), **OPEN tuple** |
| C(NaN,1) / C(+0.0,+0.0) | proposed C(NaN,NaN), **OPEN tuple** |

**GITHUB021 caution**: The previous N2 owner-approved examples remain normative *inputs* to reconciliation; this packet does not retroactively upgrade a newly generalized rule merely because it contains those examples. A code implementation's C/Java/Python recovery behavior is never semantic authority.

## G2 candidate — explicit sqrt accuracy acceptance (NOT APPROVED)

**Existing authority:** D197 requires exact rational-coordinate results whenever representable; approved F3 prevents an exact radicand \`2^1200+1\` becoming Infinity by premature Float conversion; N4 fixes principal branch and signed imaginary zero, including nonfinite input special cases; correct rounding of all *inexact complex* sqrt coordinates is explicitly **NOT** promised.

**Reference value:** For finite Float-bearing Complex \`z\`, interpret the two finite binary64 components as exact dyadic rationals, form the exact mathematical *principal* complex square root, and define the correctly rounded binary64 oracle for each irrational coordinate. A reference test may use arbitrary-precision interval arithmetic with proven enclosures, or another independently certified oracle; \`Math.sqrt\` or one host \`csqrt\` result is NOT a reference oracle. For exact Integer/Fraction components apply F3 exact classification before approximating any irrational coordinate.

**A (strictest)**: correctly rounded both complex-sqrt coordinates. Strongest portability, not selected by N4, potentially unacceptable complexity on hard-to-round inputs.

**B (RECOMMENDED)**: each finite, nonzero inexact output coordinate within **4 ULP** of the corresponding *correctly rounded binary64 oracle*; count representable Float steps in the ordered nonnegative/negative region, with a correctly rounded **oracle zero** requiring the oracle's signed zero even when the underlying mathematical coordinate is tiny but nonzero, and with mathematically exact zero requiring precisely the F1/N4 sign; no spurious NaN/Infinity due intermediates; gradual underflow (subnormal outputs must be supported); exact-rational coordinates remain exact under F3; preserve N4 principal branch and all exceptional tuples. A mathematically nonzero coordinate that truly rounds to zero may return signed zero (ordinary representable-range underflow); do not promise an impossible finite result for mathematically overflowing outputs.

**C (weaker)**: qualitative stable/finite where representable, no formal ULP upper bound. Lightweight but not precisely portable: two conforming runtimes could differ arbitrarily while describing themselves as "stable".

**Recommendation:** B balances deterministic mathematical fidelity, an independently testable ceiling, portability, and absence of universal correct rounding. This is a new observable guarantee needing exact owner approval; no source reviewed establishes that a particular Protos kernel currently meets 4 ULP. The 1-ULP claim in Apache documentation belongs to \`abs()\`, **not** its complex \`sqrt()\`.

**Specific tests after implementation authorization:** positive/negative real Float roots, integer/fraction perfect and irrational roots, \`sqrt(-2)\` exact real Integer zero, \`sqrt(2^1200+1)\` finite approximate result, \`sqrt(C(-4.0,±0.0))\` with opposite branch signs, all \`±Inf/NaN\` combinations, max-normal/subnormal extremes, nonoverflow of intermediate squares, conjugation where NaN-free, exact rational complex roots such as \`sqrt(C(3,4))=C(2,1)\`, and equality/hash/transfer of results.

**Special-value sqrt partition:** Imaginary ±Inf dominates even NaN real (returns real +Inf and imaginary signed Inf); real +Inf with finite imaginary returns (+Inf, signed zero); real -Inf with finite imaginary returns (+0,signed Inf); +Inf with imaginary NaN gives (+Inf,NaN); real finite with imaginary NaN or NaN real with finite imaginary gives (NaN,NaN); real NaN and imaginary NaN gives (NaN,NaN). The prior approved N4 choice for (-Inf,NaN), including chosen Infinity imaginary sign, must be preserved exactly; **do not assert conjugation for NaN-bearing inputs**, since canonical NaN has no observable sign in Protos. These rules exclude the ordinary exact real \`sqrt(-0.0)=-0.0\` case already approved.

## GITHUB021 reconciliation and next approval gate

| Binding owner approval | Effect of proposed G1/G2 |
| --- | --- |
| D197 exact Integer/Fraction, D196 operand-first mixed Float, signed-zero canonicalization | Preserved: exact first; inexact after conversion; no host-number-family leakage |
| D197 ==/===/hash/transfer, Float canonical NaN, Context isolation | Preserved; output semantic tuple determines existing relations, no new sentinels |
| D198 C Protos-first/minimal Java kernel, D013 and pay-as-you-grow | Preserved; mathematical generator says nothing about mandatory representation or slow path on Integer-only/Float-only |
| D198 B accepts NaN/Infinity | Preserved |
| D198 E2 exact-zero Error versus Float numerical outcome | Preserved |
| P2 and R | Preserved for finite coupled operations; scalar-real path must be independently specified when exceptional |
| N2 and F2-A/B | Exact sample outputs preserved; wider NaN propagation is explicitly a **new G1 candidate** |
| N4 and F3/F4 | Principal branch, exact-coordinate preservation, no destructive early conversion and ordinary math export preserved; numeric ULP ceiling is an explicit **new G2 candidate** |
| F1 signed-zero | Preserved for finite cases; signs of Infinity-direction coefficients explicitly defined by reference arithmetic |

**Decision required before final ratification:**
1. Approve/reject **G1** classification scheme (including its newly generalized NaN handling and exceptional real/Complex zero-divisor rule).
2. Approve/reject **G2-B** (4-ULP complex-sqrt bound) versus another explicitly specified numerical tolerance.

**STOP:** Until the owner explicitly approves these additional consequences and checks the GITHUB021 deltas, keep D198 **OPEN**, I090 **BLOCKED**, normative \`spec/\` untouched, and no implementation. On approval, produce the complete ratification record/normative update in a distinct authorized phase; do not confuse this research packet with publication of normative language policy.

**Research evidence:** source/standards reading only; **TESTS_EXECUTED=NO**, **PRODUCT_CHANGED=NO**, **SPEC_CHANGED=NO**, **DECISION_RATIFIED=NO**.
