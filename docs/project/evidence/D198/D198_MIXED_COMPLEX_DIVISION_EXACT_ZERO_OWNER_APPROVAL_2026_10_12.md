# D198 — Owner approval: Float-bearing Complex divided by exact Integer zero

**Date:** 2026-10-12
**Live decision:** [D198 #876](https://github.com/guillermomolina/protos/issues/876)
**Implementation consumer:** [I090 #868](https://github.com/guillermomolina/protos/issues/868)
**Status:** Explicit, bounded semantic decision **APPROVED**, not a claim of normative-spec publication or runtime implementation.
**Product HEAD at approval coordination:** `18f71b2a664c56ac808e2f508b6369984de02280`. The product diff in progress is local to the human executor and is not claimed to be present on GitHub.

## Approval provenance

During the D198-NORM review, the owner supplied the executor's uncommitted `spec/semantics/VALUES_AND_COLLECTIONS.md` diff. The executor had left `Complex` with any Float component divided by exact Integer `0` as an expressly unresolved D198 owner decision.

The assistant then distinguished an exact numerator from a Float-bearing numerator, proposed a concrete and complete exception to P2's otherwise coordinate-independent conversion policy, and requested explicit owner approval. The owner replied:

> "Apruebo A"

The exact approved proposal follows. Its observable effect is **newly selected now**, not implicitly ratified by an earlier G1/E2 label.

## Approved rule A — Float-bearing Complex ÷ exact Integer zero

For ordinary standard numeric `C(a,b) / r`, where `r` is the authentic exact semantic Integer value zero:

1. If **both** `a` and `b` are exact real components (Integer or Fraction), standard division signals the ordinary Protos `Error`, preserving E2/D197.
2. If **at least one** authentic component is a semantic Float, the **whole exceptional division** enters the Float domain. Convert the exact Integer zero divisor to binary64 **`+0.0`**. Convert any exact numerator components to binary64 using the approved standard `Float(Integer)` / `Float(Fraction)` rounding rules **before** arithmetic. Keep existing Float components, including signed zero, NaN or infinities.
3. Divide **each** converted numerator component by the same binary64 `+0.0`, independently, using the approved IEEE 754-2019 scalar division rules. Apply the existing Complex normalization: an imaginary Float zero remains Complex; any produced NaN is canonical. A NaN in one coordinate does **not** contaminate the other coordinate merely because it is NaN.
4. For a **nonzero real divisor**, the existing P2 coordinatewise/separable rules remain unchanged. A divisor already of Float family `+0.0` or `-0.0` also follows the pre-existing coordinatewise IEEE rule; this decision does not replace its sign with `+0.0`.
5. This operation is **Complex ÷ real**, not the coupled Complex ÷ Complex or real ÷ Complex operation. Do not fabricate a Complex divisor or move the operation to G1's coupled NaN-first classification. Keep ordinary D013 lookup/overridable dispatch, receiver/arity validation, and other independent protocol errors.

**Binding result examples (notation `C(a,b) = Complex(a,b)`):**

```text
C(1,2)    / 0 -> Error
C(1.0,2)  / 0 -> C(+Infinity,+Infinity)
C(-1.0,2) / 0 -> C(-Infinity,+Infinity)
C(0.0,2)  / 0 -> C(NaN,+Infinity)
C(NaN,2)  / 0 -> C(NaN,+Infinity)
```

Note: the last example retains an unaffected imaginary `+Infinity`, **not** `C(NaN,NaN)`. The mixed condition checks semantic Float-family membership, not merely whether either component is mathematically inexact after result computation.

## GITHUB021 invariant/delta review

| Earlier binding invariant | Effect and explicit owner authority |
| --- | --- |
| D197: authentic Integer/Fraction/Float components, family-sensitive identity and canonical imaginary zero | **Preserved:** no implicit Complex conversion, canonical result unchanged |
| D197: all-exact zero division raises Error | **Preserved:** exact components and exact divisor zero remain Error |
| E2: Float-bearing Complex exceptional conditions produce numeric results rather than Error solely due to zero | **Resolved:** the newly approved mixed case produces per-coordinate Float NaN or Infinity |
| P2: Complex ÷ real is coordinatewise; conversion does not contaminate unrelated coordinates in ordinary operations | **Explicit narrow amendment for exact-zero exceptional divisor only:** both coordinates enter Float domain, so `2/0` no longer signals Error if the other coordinate is Float |
| G1: Float-domain division non-signaling, NaN classification and signed-zero handling | **Preserved:** genuine coupled operations follow G1; Complex ÷ real stays separable |
| F1 and IEEE Float signed zeros | **Preserved:** exact `0` becomes `+0.0`; already-Float `-0.0` remains negative |
| D013 dynamic dispatch, numeric ==/===/hash, Actor/P/Context and pay-as-you-grow | **Preserved:** no new syntactic or public value family, and no unrelated runtime work |
| Previously pending mixed Complex÷Integer(0) output | **NEWLY APPROVED:** exact tuples listed above, with explicit owner statement |

**Consistency:** The new observable conversion/exception priority is **not simply inferred from E2 or P2**; it was exposed as a conflict and separately selected by the owner. The implementation/spec author must not re-open this as a missing decision.

## Required normative publication step

The executing agent currently has only `spec/semantics/VALUES_AND_COLLECTIONS.md` modified locally, with the following placeholder in the standard Complex-arithmetic separable division rules:

> When `r` is the exact Integer zero and at least one component is a Float, the result is not specified by this revision (pending D198 owner decision).

Replace the placeholder with the approved rule A above, including the mandatory `+0.0` divisor conversion and operand-first conversion of the other exact component. This evidence file does **not** patch the human's uncommitted product checkout.

After the normative diff is reviewed and the human reports the applicable validations, assign the next specification revision to `spec/PROTOS_SPEC_CHANGELOG.md` **immediately before** the human's normative commit, per `spec/AGENTS.md`. Do not invent a revision number now. Keep D198 open and I090's remaining public Complex implementation blocked until normative publication/ratification is confirmed.

```text
D198_MIXED_COMPLEX_DIV_EXACT_ZERO=A_OWNER_APPROVED
D198_GITHUB021_EXPLICIT_AMENDMENT=PASS
D198_NORMATIVE_SPEC_PUBLICATION=NOT_YET_CONFIRMED
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED_UNTIL_SPEC_PUBLICATION
PRODUCT_EDITS_BY_THIS_RECORD=NO
TESTS_EXECUTED_BY_THIS_RECORD=NO
```
