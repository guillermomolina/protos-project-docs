# D198 — Owner approval: Complex is a scalar Number, not a Vector or Matrix

**Date:** 2026-10-12
**Status:** Explicit bounded owner approval; non-normative architectural clarification, NOT complete D198 ratification.
**Live decisions:** [D197 / #865](https://github.com/guillermomolina/protos/issues/865) (already RATIFIED/CLOSED), [D198 / #876](https://github.com/guillermomolina/protos/issues/876) (OPEN), [I090 / #868](https://github.com/guillermomolina/protos/issues/868) (remaining public Complex work BLOCKED on D198).
**Product reference:** `guillermomolina/protos` main HEAD `18f71b2a664c56ac808e2f508b6369984de02280` examined for coordination; **no source edits or product tests performed**.

## Explicit approval provenance and exact scope

The owner asked whether languages distinguish Complex from Vector/Matrix or model Complex as a special case of a two-element vector. After a concrete comparative and mathematical explanation, the recommendation was:

> Do not reopen D197 on this basis. Keep Complex as an independent numeric scalar in the D197 Number tower, separate from Vector/Matrix, and reserve possible shared ordinary algebraic protocols for a later independent design investigation.

The owner responded on 2026-10-12:

> "ok pues apruebo lo que me has recomendado. Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs."

This is **approval of the recommendation above, not approval of a new Vector or Matrix public family, a new Real prototype, a general algebra API, or specific representation/lowering**.

## Approved architectural boundary

1. **Complex remains a mathematical scalar and a distinct semantic Number family**, with real and imaginary authentic Number components under D197. Its semantic identity, arithmetic, canonicalization, equality and hash are determined by D197/D198, **not by Array or a future Vector/Matrix implementation**.
2. **A Complex is NOT defined as a special `Vector(2)` or `Matrix(2,2)`** in the Protos object model. Ordinary Vector/Matrix operations cannot implicitly determine `Complex.*`, `Complex./`, `Complex.sqrt`, or standard numeric `==` and `===`.
3. **Future Vector and Matrix are composition-oriented numeric containers**, whose elements may include authentic Complex values. This describes an intended separation of responsibilities, **not an approved type hierarchy, mutability/storage policy, construction API, element coercion rule, or release commitment for such containers**.
4. **Possible sharing is at the level of ordinary algebraic algorithms/protocols when independently justified**, with semantic requirements and dispatch decided in a separate future research/approval checkpoint. Do not automatically create a common superclass, protocol registry, runtime special case or generic algebra operation today.
5. **Physical representation is not semantic authority.** Implementations MAY use compact pairs, packed storage, SIMD, or internal techniques if independently valid and observably equivalent. The decision imposes no per-value Vector allocation, no Matrix wrapper, no mandatory two-element Array, and no extra cost on trivial Integer/Float programs. Preserve D198-C Protos-first and pay-as-you-grow.

## Mathematical falsification / why a vector analogy is insufficient

For `(1,2)` and `(3,4)`:

- Vector dot product: `1*3 + 2*4 = 11`.
- Componentwise product: `(3,8)`.
- Complex product: `(1+2i)(3+4i) = -5+10i`.

The fact that an additive complex value may be represented geometrically by `(a,b)` does not make its multiplicative structure that of a generic two-component Vector. Likewise the algebra isomorphism `a+bi <-> [[a,-b],[b,a]]` is a mathematical model, **not** a selection of Matrix as the public Complex representation.

## GITHUB021 invariant/delta consistency

| Previously owner-approved invariant | Effect of current clarification |
| --- | --- |
| D197 Number -> Integer/SmallInteger/BigInteger, Fraction, Float, Complex | PRESERVED. Complex remains a sibling semantic numeric family; **no new Real prototype or changes to the hierarchy**. |
| D197 genuine exact/Float components; canonical exact imaginary zero and Float signed zeros | PRESERVED. Vector container behavior cannot replace Complex canonicalization. |
| D197 numeric `==`, `===`, hash, Actor/P/Context transfer | PRESERVED; no array/vector identity or alias semantics transferred to Complex. |
| D198-C ordinary Protos-first and minimal justified kernel / D013 / pay-as-you-grow | PRESERVED; any internal packed pair stays private and optional. |
| D198-B/E2, P2/N2/R/N4/N5, F1/F2-A/F2-B/F3/F4 | PRESERVED; no change to exceptional arithmetic, precision, sqrt or API. |
| Closing research G1 exceptional generator, G2-B 4-ULP sqrt | **STILL PROPOSED, NOT APPROVED**; this approval does not ratify them. |
| Future Vector/Matrix/algebra public contracts | **OPEN / NOT SELECTED**; independent future design approval necessary before altering public semantics. |

**Conclusion:** No conflict requiring D197 to reopen. The new statement is an explicit owner-approved **boundary/reaffirmation** for D198 and its I090 implementation consumer. D198 remains **OPEN**, I090 public Complex remainder remains **BLOCKED** pending G1/G2 and final normative publication. No new Issue required solely for a non-scheduled possible future generic algebra investigation. No existing formal Vector/Matrix-specific Issue was identified as materially owned by this limited decision.

## Publication and validation boundary

This is a documentation/Issue coordination only. No changes to normative `spec/`, product `src/`, `protos/`, `pom.xml` or `CHANGELOG.md`. No test or build executed or claimed. No shell validation or `git diff --check` is claimed by this record; direct repository publication was explicitly authorized for this task. Documentation evidence is subordinate to normative `spec/` and live GitHub Issues.
