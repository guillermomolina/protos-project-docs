# D198 — Owner approval of F1–F4: final Complex contract reconciliation

**Date:** 2026-10-12
**Decision authority:** [D198/#876](https://github.com/guillermomolina/protos/issues/876), remains **OPEN** pending exhaustive result matrix and final GITHUB021 check.
**Implementation consumer:** [I090/#868](https://github.com/guillermomolina/protos/issues/868), remaining public Complex work **BLOCKED** until publication/ratification of the complete contract.
**Product reference:** `guillermomolina/protos` HEAD `18f71b2a664c56ac808e2f508b6369984de02280` was inspected during the immediately preceding research; not an implementation or test provenance.

## Exact owner approval and boundary

The owner explicitly selected **P2, N2, R, N4 and N5** in the preceding D198 design discussion: "Promoción P2: APROBADO", "Tabla excepcional N2: APROBADO", "Precisión R: APROBADO", "Raíz N4: APROBADO", "API N5: APROBADO"; expressly **"no implementes aún"**. After a GITHUB021 consistency investigation presented a *single consolidated refinement* **F1 + conservative F2-A + conservative F2-B + F3 + F4**, the owner responded **"Apruebo las sugerencias"**. This ratifies precisely the exposed refinements, **not** all possible unenumerated exceptional tuples, unproposed API effects, a universal sqrt-ULP target, product/spec edits, or closing D198.

Previously selected and unchanged: D197 numeric hierarchy, true Integer/Fraction/Float components, exact-zero imaginary demotion versus signed Float-zero retention, exact equality and hash/identity, actor/P/Context transfer; D198-C Protos-first/minimal justified kernel, D198-B nonfinite Float admission and D198-E2 all-exact zero-divisor Error versus Float-domain numerical exceptional outcome; D013 overrideable dispatch and pay-as-you-grow.

## Approved F1 — signed zero in finite coupled Complex arithmetic

Finite Complex multiplication/division requiring Float uses R's once-rounded *mathematical result per coordinate*, after P2-required operand-first conversion. If a coordinate's exact mathematical result is nonzero but rounds to zero, retain its sign. If it is exactly zero, use the signed-zero outcome of the rectangular terms, with the IEEE binary64 signs of zero products and ties-to-even add/sub cancellation: exact cancellation of nonzero opposing terms gives +0.0; signs of zero terms matter.

```text
real of multiplication = a*c - b*d
imag of multiplication = a*d + b*c
real of division       ∝ a*c + b*d
imag of division       ∝ b*c - a*d

C(1.0,-0.0) * C(1.0,+0.0) -> imaginary +0.0
C(1.0,-0.0) * 1.0         -> imaginary -0.0
C(1.0,-0.0) / C(2.0,+0.0) -> imaginary -0.0
C(1.0,-0.0) + 3           -> imaginary -0.0
```

Here `C(a,b)` abbreviates the approved `Complex(a,b)` ordinary call; the written formulas specify mathematical reference signs, **not** mandatory lowering or intermediate primitive operations. Exact imaginary Integer zero still canonically collapses under D197.

## Approved F2 — conservative nonfinite multiplication/NaN policy

**F2-A:** `C(+Infinity,+0.0) * C(+0.0,1.0) -> C(NaN,+Infinity)`, rather than coordinate `+0.0` recovered by a special geometrical axis rule. Native invalid infinity-times-zero is not silently suppressed by axis rotation.

**F2-B:** `C(NaN,+Infinity) * C(1.0,1.0) -> C(NaN,NaN)` conservatively. For coupled Complex multiplication/division, a genuine incoming NaN must not be erased solely by overflow/infinity recovery. This does **not** impose global NaN contagion on separable componentwise `+` and `-`, nor supersede the independently selected N4 `sqrt` special-value table.

**Scope:** F2 chooses these conservative consequences and the restriction against silently erasing actual incoming NaN. It is **not** a completed 256-case or exhaustive arithmetic exceptional-result matrix. N2's explicitly approved examples, including `C(1,2)/C(+0.0,+0.0) -> C(+Inf,+Inf)`, remain binding. Other combinations with signed zeros, NaNs, infinities, exact/Float mixing and zero divisors remain to be generated, checked and explicitly reconciled; do not infer outcomes from host C99/Java/Python routines without proof.

## Approved F3 — sqrt exactness and approximation boundary

- Exact Number/Complex inputs yielding rational square roots, including rational-coordinate Complex roots, remain exact and normalize under D197.
- For exact inputs with an irrational coordinate, retain any rationally expressible coordinate exactly while approximating only the irrational coordinate in Float; e.g. `sqrt(-2) -> Complex(Integer(0), Float(1.4142135623730951))`.
- Float-bearing inputs stay in the Float/Complex inexact domain, preserving N4 principal branch, signed imaginary-zero branch cuts and nonfinite table; e.g. `sqrt(-4.0) -> Complex(Float(+0.0), Float(2.0))` and `sqrt(-0.0) -> Float(-0.0)`.
- The approximation of an exact input MUST NOT require prematurely rounding the full exact radicand to Float: `sqrt(2^1200 + 1)` must have a finite Float approximation, not spurious +Infinity from intermediate overflow.
- Correct rounding for every inexact *complex* sqrt is **not** promised. Portable acceptable approximation error bounds and exhaustive special-value conformity still need formal treatment; do not invent a universal ULP guarantee.

## Approved F4 — ordinary sqrt export in the math library

`sqrt(value)` is an ordinary exported function of `std:math/Math`; it is **not** a reserved word, primitive grammar form or required `Number.sqrt()` prototype selector. Core Number remains owner of fundamental arithmetic protocols; math functions live in the ordinary library. The eventual private numerics kernel, if justified, must preserve Protos-first semantics and D013 dispatch. The current repo has `protos/lib/math/Integer.protos`, but not an implemented `Math.protos`; this approval is prospective, not an assertion of shipping public API. Import/prelude exposure beyond the proposed module is not selected.

## GITHUB021 audit and STOP

| Existing selected invariant | F1–F4 consequence |
| --- | --- |
| D197 exact components and imaginary exact-zero canonical form | Preserved |
| D197 Float signed zero, binary64 and semantic identity/hash | Preserved and made more observable by F1 |
| D196/D197 operand-first Float promotion | Preserved under P2/R; no mandatory Float conversion before exact sqrt |
| D197 equal/hash, Actor/P/Context | Preserved, no change |
| D198 C/B/E2 | Preserved, including no Error merely for Float exceptional arithmetic |
| P2/N2/R/N4/N5 | Preserved; F1–F4 explicitly owner-approved refinements |
| Extra nonfinite Complex output tuples not exposed in approval | **OPEN**, not implicitly ratified |
| Quantitative approximation conformance for complex sqrt | **OPEN**, no invented bound |
| Exhaustive spec and final decision ratification | **OPEN**; follow GITHUB021 again before publication |

**Required next phase:** one coherent **D198 research-only** pass to enumerate remaining exceptional-result combinations, specify signed-zero classification, define the finite/inexact complex-sqrt approximation acceptance policy and identify any further semantic choices. Reconcile against D197/C/B/E2/P2/N2/R/N4/N5/F1–F4 before final approval. No new Ixxx is required. Product, normative `spec/`, tests, pom/changelog and implementation are untouched by this documentation-only owner-approval record.

`P2=N2=R=N4=N5=OWNER_APPROVED` · `F1=F2_A=F2_B=F3=F4=OWNER_APPROVED` · `D198_FULL_RATIFICATION=NO` · `I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED` · `TESTS_EXECUTED=NO`.
