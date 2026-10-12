# D198 — Owner approval of Candidate B: non-finite Float components

**Date:** 2026-10-12  
**Formal decision:** [D198/#876](https://github.com/guillermomolina/protos/issues/876)  
**Consuming implementation:** [I090/#868](https://github.com/guillermomolina/protos/issues/868)  
**Approved independently:** [Architecture Candidate C](D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md)  
**Prior analysis:** [Non-finite component comparative study](D198_NONFINITE_COMPLEX_COMPONENT_POLICY_RESEARCH_2026_10_12.md) (documentation snapshot [`ab2a5f0572ac2b944c4052591753941fe6e2a950`](https://github.com/guillermomolina/protos-project-docs/commit/ab2a5f0572ac2b944c4052591753941fe6e2a950))  
**State:** Owner selected **Candidate B only for component admission**. D198 as a whole is **OPEN and not fully ratified**.

## Explicit provenance and selected scope

The project owner explicitly stated: **"Apruebo B"** on 2026-10-12 after a focused comparison among (A) finite-only Complex components, (B) admitting every legitimate Float component including NaN and ±Infinity, and (C) allowing storage but prohibiting arithmetic. The owner also explicitly requested issue coordination and durable evidence, and authorized direct documentation publication for this task.

This approval selects the following **observable numeric value-domain invariant**:

> A canonical Complex may contain as either its real component or its imaginary component **any genuine semantic Float binary64 value**, including canonical Float NaN, positive infinity, negative infinity, positive zero and negative zero, alongside exactly retained semantic Integer/Fraction components. The finiteness of a legitimate Float component is not a construction or membership precondition.

Consequence: the present I090-C1 Java-side `Double.isFinite` rejection in `ProtosComplexNumericSupport.requireComponent` is a **temporary implementation restriction**, not intended stable Protos language policy; removing it belongs to the later, spec-backed implementation, not to this documentation update. This approval does **not** admit arbitrary Java `Double`, host wrappers, unrelated objects, fake delegators, or unauthenticated foreign Prelude values as semantic components.

## Preexisting D197 invariants retained (not reopened)

1. D197's visible `Number → Integer → SmallInteger/BigInteger`, with sibling `Fraction`, `Float`, `Complex`, is unchanged.
2. Components preserve exact Integer/Fraction values and the precise Float binary64 semantic value, without converting exact components to doubles.
3. A Complex whose imaginary component is **exact Integer zero** canonicalizes to its real component, even if that real component is infinite or NaN Float. Float imaginary `+0.0` and `-0.0` do **not** cause collapse and preserve their signed-zero identity.
4. Ordinary `==` numeric equality is componentwise mathematical equality, which remains nonreflexive for NaN; `===` observes component semantic identity, which regards all Float NaN representations as one Float semantic NaN. Distinct signed-zero components are not semantically identical.
5. The normal hash, identity hash, Map/IdentityMap and Actor/P/multi-Context transfer rules must remain coherent with D197: equal numbers share normal hashes, Float NaN has its designated canonical normal-hash class, and an admitted Complex preserves component families and Float special-value distinctions across transfers.
6. Ordinary Protos source dispatch, D013 overrideability, user-side ownership/validation, and pay-as-you-grow primitive Integer/Float paths remain binding under approved architectural Candidate C.
7. Exact rational division by exact zero still signals `Error`; Float binary64 arithmetic still follows its already ratified IEEE 754 rules. No Complex division-by-zero policy is implied.

Illustrative semantic consequences (NOT ratified factory spelling): `Complex(Float.infinity, 0)` collapses to Float infinity by the existing exact imaginary-zero rule; `Complex(Float.infinity, -0.0)` remains Complex; `Complex(3, Float.nan)` remains a canonical Complex.

## GITHUB021 owner-decision invariant / delta reconciliation

- **D197 exact component preservation:** PASS — non-finite Float values are retained without rounding, and exact components stay exact.
- **D197 exact-zero collapse / Float signed-zero noncollapse:** PASS — no general zero collapse introduced.
- **D197 Float NaN identity and unequal numeric comparison:** PASS — same semantic Float special value propagates to component identity and equality; no new global NaN entity.
- **D197 normal numeric hash coherence:** PASS as required invariant; detailed implementation for exceptional complex hashing remains to be validated.
- **D197 authenticated immutable family/Actor transfer:** PASS as requirement; source implementation must verify it.
- **D198 Candidate C Protos-first minimal kernel:** PASS — the admission choice neither changes architecture nor requires a new runtime facility.
- **No hidden reopening:** No earlier explicit owner invariant forbade NaN/Infinity Complex components. The current normative text says **not defined by this revision**, and the Java finite-only check explicitly documents its provisional status.

These PASS labels signify **compatibility of the approved value-domain decision with the existing normative constraints**, **not** green product tests or normative publication.

## Explicit STOP boundary — not selected by this approval

- NaN/Infinity multiplication, division, addition/subtraction and unary negation: exceptional result tables, propagation and signed-zero behavior — **OPEN**.
- Exact-vs-inexact Complex **division by zero** — **OPEN** except D197's existing exact real zero Error and Float basic division guarantees.
- Stable algorithms and IEEE binary64 operation ordering/rounding for finite and non-finite Complex arithmetic, overflow and underflow — **OPEN**.
- The complete public `Complex(...)` callable factory, `real`/`imaginary` selectors, mathematical-function name/location and `sqrt` branch precision — **OPEN**.
- Normative `guillermomolina/protos:spec/` revision, source/tests, product commit and I090 closure — **NOT performed or authorized by this isolated approval**.

There is no mandate to implement an unapproved intermediate public constructor or to make I090 READY. The next proposed D198 decision is the **exceptional arithmetic/division policy**; the user retains approval authority.

## Files and execution boundary

Decision research/approval was reconciled against `guillermomolina/protos/AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `src/main/java/com/guillermomolina/protos/runtime/ProtosComplexNumericSupport.java`, `src/main/java/com/guillermomolina/protos/execution/ProtosCurrentNumericRelations.java`, `guillermomolina/protos-project-docs/AGENTS.md`, prior D198 research and work records, and GitHub issues #876 and #868. Historical baseline I090-C1: [`c75dea75b8db045eb6833aeab1d5b8710eaa0556`](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556); this evidence does not claim to bind a later product HEAD.

**Changes:** only `guillermomolina/protos-project-docs` documentation and coordinated Issue text. **No product source, specification, binaries or tests were changed or executed** by this task. This record is durable owner-approval evidence, not a full D198 language ratification.

```text
D198_ARCHITECTURE=C_OWNER_SELECTED
D198_NONFINITE_FLOAT_COMPONENT_ADMISSION=B_OWNER_APPROVED_2026_10_12
D198_COMPLEX_EXCEPTIONAL_ARITHMETIC=OPEN
D198_COMPLEX_PRECISION_AND_SQRT=OPEN
D198_COMPLETE_PUBLIC_API=UNRATIFIED
D198_DESIGN_ISSUE=OPEN
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED
D197_RATIFIED_INVARIANTS=UNCHANGED
PRODUCT_TESTS_EXECUTED=NO
```
