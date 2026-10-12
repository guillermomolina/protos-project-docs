# D198 — Public Complex semantics design gate and I090 reconciliation

## 2026-10-12 — D198 normative 0.1.457 published and ratified

**Public product read-back:** [`guillermomolina/protos` commit `7903d43208a8365c6f75d4c886e5babd815754b2`](https://github.com/guillermomolina/protos/commit/7903d43208a8365c6f75d4c886e5babd815754b2) is published on `main` with the exact title **"D198: specify public Complex construction, arithmetic, and std:math/Math.sqrt"**. The commit changes only `spec/semantics/VALUES_AND_COLLECTIONS.md` and `spec/PROTOS_SPEC_CHANGELOG.md`, and the latter assigns global normative spec revision **`0.1.457` (2026-10-12)**.

**Durable decision authority:** [D198 published language ratification](../../decisions/language/D198_PUBLIC_COMPLEX_CONSTRUCTION_ARITHMETIC_AND_SQRT.md) with bounded owner approvals C/B/E2, P2/N2/R/N4/N5, F1/F2-A/F2-B/F3/F4, G1/G2-B, three subsequent clarifications and approved A exact-zero-divisor policy. [Publication and human-validation closure evidence](../../evidence/D198/D198_SPEC_0_1_457_PUBLICATION_AND_CLOSURE_2026_10_12.md). This maintained work record is not a second source of language norm.

**GITHUB021:** PASS; D197 remains closed and binding, D013 overridable dispatch/pay-as-you-grow retained, Complex stays an independent scalar numeric family, and the spec carries E2/A, G1, N4/G2-B and all owner-approved signed-zero/NaN exceptions. Prior records declaring D198 decisions unselected or normative publication pending are **historical entries now superseded** by the normative `0.1.457` publication.

**Human-executor evidence:** Owner reports `git diff --check` clean, local tests PASS and product commit pushed. Agent did not execute tests or builds. Normative publication does not mean that I090 has implemented or tested the newly public Complex API.

**Lifecycle:** D198's decision ratification and spec publication conditions are satisfied once the bounded durable records above are published/read back; then close D198 and move I090 from `status:blocked` to `status:ready` provided no other live blockers. The separate public Complex implementation and acceptance remain owned by I090.

```text
D198_DESIGN_APPROVAL=COMPLETE
D198_SPEC_REVISION=0.1.457
PROTOS_REVISION=7903d43208a8365c6f75d4c886e5babd815754b2
D198_GITHUB021=PASS
D198_NORMATIVE_PUBLICATION=PUBLISHED
D198_DURABLE_DECISION=PUBLISHED
D197_REOPENED=NO
I090_COMPLEX_RUNTIME_AND_CONFORMANCE=PENDING
HUMAN_REPORTED_GIT_DIFF_CHECK=PASS
HUMAN_REPORTED_LOCAL_TESTS=PASS
AGENT_EXECUTED_TESTS=NO
```

---

## 2026-10-12 — A approved: Float-bearing Complex divided by exact Integer zero

The owner explicitly answered **"Apruebo A"** after reviewing the unresolved D198-NORM case `Complex(1.0,2) / 0` and the proposed **narrow E2/P2 reconciliation**. [Bounded owner-approval evidence and GITHUB021 audit](../../evidence/D198/D198_MIXED_COMPLEX_DIVISION_EXACT_ZERO_OWNER_APPROVAL_2026_10_12.md).

**Binding A:** If `C(a,b)` is divided by exact Integer `0`, then **both exact components** still signal `Error`; if **any component is genuine Float**, convert that zero divisor to Float `+0.0` and convert the remaining exact component(s) to Float *before* coordinatewise binary64 division. The result remains componentwise (one NaN does not contaminate the other). `C(1.0,2)/0=C(+Inf,+Inf)`, `C(-1.0,2)/0=C(-Inf,+Inf)`, `C(0.0,2)/0=C(NaN,+Inf)`, `C(NaN,2)/0=C(NaN,+Inf)`. This narrow zero-divisor rule does not change `Complex ÷` nonzero real or existing `Complex ÷` Float-zero behaviors, nor G1's coupled division rules.

**GITHUB021 delta:** owner explicitly chose the observable exceptional-domain priority rather than deriving it silently from E2 or P2; D197 all-exact Error and ordinary per-coordinate operations outside exact-zero exception are preserved. G1/G2-B, N4/F3, D013, and Complex-scalar-not-Vector/Matrix remain approved/unchanged.

**State:** No owner decision blocker remains for the mixed division-zero case. D198-NORM may replace its placeholder in the human's currently uncommitted `spec/semantics/VALUES_AND_COLLECTIONS.md` and proceed to normative review. Product `spec/` publication and D198 final closure remain unconfirmed; I090 public Complex remainder stays blocked until the norm is published. This GitHub coordination did not modify product files or execute tests.

```text
D198_MIXED_COMPLEX_DIV_EXACT_ZERO=A_OWNER_APPROVED
D198_GITHUB021_AMENDMENT=PASS
D198_NORMATIVE_SPEC_PUBLICATION=PENDING
D198_FINAL_CLOSURE=PENDING
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED_UNTIL_SPEC_PUBLICATION
```

---

## 2026-10-12 — Three final Complex/sqrt clarifications explicitly owner approved

During the D198-NORM specification-authoring handoff the human executor surfaced three previously unbound observable edges. The owner confirmed the assistant's recommendations in all three cases, then expressly replied **"ok pues aprobadas las 3 recomendaciones para ti tambien"**. The exact three selections are recorded in [D198 final clarifications owner approval and GITHUB021 review](../../evidence/D198/D198_THREE_FINAL_SEMANTIC_CLARIFICATIONS_OWNER_APPROVAL_2026_10_12.md).

**Approved Q1:** `real ÷ Complex` uses **G1 branches with no synthetic numerator imaginary `b` terms**. Float finite result signs derive from direct `r*c` and `-(r*d)` math, not `C(r,+0)` conversion; `+0.0/C(1.0,1.0)` has imaginary `-0.0`. Float complex-zero divisor retains `C(k*r,NaN)` and the already approved `1.0/C(+0,+0)=C(+Inf,NaN)`. G1 infinity direction and E2 exact-zero Error remain binding.

**Approved Q2:** `sqrt(C(-Infinity,NaN))=C(NaN,+Infinity)` deterministically; Protos Float NaN exposes no sign and other N4 special cases remain unchanged.

**Approved Q3:** **G2-B 4 ULP per finite inexact Float output coordinate applies to `sqrt` of real inputs as well as Complex inputs.** E.g. `sqrt(2)`, `sqrt(2.0)`, imaginary coordinate of `sqrt(-2)`; independently correctly rounded oracle, exact required zeros, F3 exact results and N4 exceptional cases are preserved. Correct rounding of all real-input roots is **not** newly mandated.

**GITHUB021:** The exact observable refinements were exposed to and approved by the owner. They preserve D197, D198 C/B/E2/P2/N2/R/N4/N5/F1–F4/G1/G2-B and Complex-scalar-not-Vector. **No design selection gate remains for these three points.** Product `spec/` authoring/validation/publication must still complete; do not claim it completed from this docs coordination. D198 remains OPEN until normative publication and closure; I090's remaining public Complex work stays BLOCKED until that publication.

```text
D198_REAL_DIV_COMPLEX_NO_SYNTHETIC_B=OWNER_APPROVED
D198_SQRT_NEG_INF_NAN=COMPLEX_NAN_POS_INF_OWNER_APPROVED
D198_SQRT_4ULP_ALL_INEXACT_COORDINATES=OWNER_APPROVED
D198_NORMATIVE_PUBLICATION=PENDING
D198_ISSUE_CLOSURE=PENDING
I090_REMAINING_PUBLIC_COMPLEX=BLOCKED_UNTIL_NORMATIVE_PUBLICATION
TESTS_EXECUTED=NO
```

---

## 2026-10-12 — G1 and G2-B explicitly owner approved; publication gate remains

The owner clarified his intent explicitly after the assistant had left the closing candidates marked pending: **"te dije que aprobaba todas, pense que ya lo habias registrado"**. This confirms the exact **G1** exceptional-case generator and **G2-B** maximum-4-ULP inexact Complex `sqrt` contract in the [closing research](../../evidence/D198/D198_CLOSING_NONFINITE_MATRIX_AND_SQRT_CONFORMANCE_RESEARCH_2026_10_12.md). The assistant's previous narrower approval interpretation was incorrect; do not continue to request another G1/G2-B selection.

**Owner approval and GITHUB021 evidence:** [D198 G1/G2-B approval and invariant review](../../evidence/D198/D198_G1_G2_B_OWNER_APPROVAL_AND_GITHUB021_2026_10_12.md). All earlier D197/D198 approvals remain binding, including the independent Complex scalar-not-Vector/Matrix distinction. This does **not** authorize a new Matrix/Vector API, language syntax or host arithmetic policy.

**State:** D198's semantic candidate selection is complete. The separate authority/publication gate remains: reconcile the exhaustive result generator and the `sqrt` 4-ULP requirement into authoritative normative `spec/`, perform the final publication-time GITHUB021 check, then close D198 and unblock I090's remaining public Complex implementation work. No tests, program, product code or `spec/` edits were performed in this approval-recording step.

```text
D198_G1=OWNER_APPROVED
D198_G2_B=OWNER_APPROVED
D198_SEMANTIC_SELECTION=COMPLETE
D198_NORMATIVE_PUBLICATION=PENDING
D198_FINAL_CLOSURE=PENDING
I090_REMAINING_PUBLIC_COMPLEX=BLOCKED_UNTIL_SPEC_PUBLICATION
D197_REOPENED=NO
TESTS_EXECUTED=NO
```

**All lower entries identifying G1 or G2-B as unapproved are historical statuses now superseded.**

---

## 2026-10-12 — Approved Complex scalar separation from future Vector/Matrix

The owner explicitly approved retaining **Complex as a distinct semantic scalar Number family**, **not** a disguised `Vector(2)` or a `Matrix(2,2)`, after an architectural comparison. D197 is **not reopened**. Future Vector/Matrix containers may carry Complex elements, but no Vector/Matrix family hierarchy, factory, storage/mutability semantics, algebraic interface, module, or generic protocol is selected. Potential shared ordinary algebraic abstractions require a separate future research and explicit owner decision, rather than deriving Complex multiplication/division from vector multiplication.

**Owner approval and GITHUB021 evidence:** [Complex scalar vs Vector/Matrix approval](../../evidence/D198/D198_COMPLEX_SCALAR_VS_VECTOR_MATRIX_OWNER_APPROVAL_2026_10_12.md). Preserves D197 hierarchy, exact/Float components, signed zero, numeric equality/identity/hash, Actor/P/Context, D198-C/B/E2, P2/N2/R/N4/N5/F1–F4, D013 and pay-as-you-grow; makes **no** new numeric arithmetic, sqrt or API choice. The closing **G1** exceptional generator and **G2-B** sqrt 4-ULP proposal stay **UNAPPROVED**.

**State:** D197 CLOSED/RATIFIED; D198 OPEN; I090 public Complex remainder BLOCKED pending final G1/G2 selection and normative publication. No `spec/` or product changes/tests, and no new formal Vector/Matrix Issue allocated for unspecified future work.

```text
D198_COMPLEX_SCALAR_SEPARATE_FROM_VECTOR_MATRIX=OWNER_APPROVED
D197_REOPENED=NO
VECTOR_MATRIX_PUBLIC_CONTRACT=NOT_SELECTED
GENERIC_ALGEBRA_API=NOT_SELECTED
D198_G1=PROPOSED_NOT_APPROVED
D198_G2_B=PROPOSED_NOT_APPROVED
D198_FULL_RATIFICATION=NO
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED
TESTS_EXECUTED=NO
```

---

## 2026-10-12 — Closing research G1/G2 candidate pending owner selection

After express owner approvals of P2/N2/R/N4/N5 and F1/F2-A/F2-B/F3/F4, a single [closing investigation](../../evidence/D198/D198_CLOSING_NONFINITE_MATRIX_AND_SQRT_CONFORMANCE_RESEARCH_2026_10_12.md) compared C/CPython, Apache Commons Numbers, C++/musl and robust division research. It proposes **G1**, a generative and exhaustive exceptional Complex arithmetic classification (256 Z/F/I/N kinds per coupled binary operation, plus signs), and **G2-B**, at most four binary64 ULP steps from an independent correctly rounded complex-sqrt oracle on finite inexact components. This records **research proposals only**. Provisional scalar-real / complex signed-zero exceptional outputs and generalized NaN propagation are new observable consequences and MUST NOT be inferred from previously approved N2/F2.

**Remaining owner checkpoint:** select G1 and G2-B explicitly (or alternatives), then perform final GITHUB021 and normative publication. D198 OPEN, I090 BLOCKED for public Complex remainder; D197 and C/B/E2/P2/N2/R/N4/N5/F1–F4 remain approved. No product/spec editing, implementations or tests were done.

\`\`\`text
D198_G1_EXCEPTIONAL_GENERATOR=PROPOSED_NOT_APPROVED
D198_G2_B_SQRT_4ULP=PROPOSED_NOT_APPROVED
D198_P2_N2_R_N4_N5_F1_F2_F3_F4=OWNER_APPROVED
D198_FULL_RATIFICATION=NO
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED
TESTS_EXECUTED=NO
\`\`\`

---

## 2026-10-12 — P2/N2/R/N4/N5 and F1–F4 explicitly owner-approved; exhaustive contract pending

**Owner-approved design gates:** P2 (operation-local Float promotion), N2 (specified exceptional numeric tuples and non-signaling envelope), R (finite Complex multiply/divide precision), N4 (exact/principal-branch sqrt), N5 (ordinary public Complex factory/selectors/operations), **plus** the subsequent exact refinements **F1** (finite arithmetic signed-zero), **F2-A/B conservative** (axis Infinity-times-zero remains NaN; genuine incoming NaN not silently erased in coupled operations), **F3** (exact-coordinate-preserving sqrt and no destructive premature exact-radicand-to-Float conversion), **F4** (`std:math/Math` ordinary exported `sqrt`). The owner explicitly approved the five initial named candidates and subsequently said **"Apruebo las sugerencias"** after the F1–F4 consolidated decision packet.

**Authority/provenance:** [F1–F4 approval and GITHUB021 invariant audit](../../evidence/D198/D198_F1_F4_RECONCILIATION_OWNER_APPROVAL_2026_10_12.md); preceding [E2 approval](../../evidence/D198/D198_E2_EXCEPTIONAL_ARITHMETIC_OWNER_APPROVAL_2026_10_12.md). Preserve D197 closed ratification, D198-C/B/E2 and D013; this evidence is non-normative.

**Still OPEN:** a deterministic and exhaustive signed-zero/NaN/infinity/overflow/underflow result specification beyond the expressly approved N2/F2 examples; formal portable approximation acceptance for inexact complex `sqrt`; final GITHUB021 approval after these remaining contract details are surfaced. No new architectural candidate or Ixxx is required. D198 remains **OPEN**, and I090's remaining public Complex work remains **BLOCKED** until the complete design/normative contract is ratified. The current `spec/`/product has not been updated by this record; **no tests run**.

```text
D198_P2_N2_R_N4_N5=OWNER_APPROVED
D198_F1_F2_A_F2_B_F3_F4=OWNER_APPROVED
D198_EXCEPTIONAL_EXHAUSTIVE_TABLE=OPEN
D198_INEXACT_COMPLEX_SQRT_CONFORMANCE=OPEN
D198_FULL_RATIFICATION=NO
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED
TESTS_EXECUTED=NO
```

---

## 2026-10-12 — Owner APPROVES E2 exceptional arithmetic signaling policy; detailed results still OPEN

**Explicit owner decision:** "apruebo E2" after the E1/E2/E3 comparison, with an express separation between Error-vs-numeric-outcome policy and the **unselected** exact numerical output tuple. **E2 APPROVED**: entirely exact Complex division by exact complex zero signals fresh `Error`; if an operation genuinely enters the inexact Float domain, IEEE numeric exceptions (zero, NaN, Infinity, over/underflow) are **not in themselves reasons to signal Error**. Correct normal protocol/argument failures may still signal Error; E2 does not bypass them.

**Approval and GITHUB021 invariants:** [E2 owner approval evidence](../../evidence/D198/D198_E2_EXCEPTIONAL_ARITHMETIC_OWNER_APPROVAL_2026_10_12.md), first published in documentation commit [`279c80b77001cbaa6a717a6c6d15373c699ca2ae`](https://github.com/guillermomolina/protos-project-docs/commit/279c80b77001cbaa6a717a6c6d15373c699ca2ae). Previously approved C Protos-first architecture and B non-finite Float-component admission remain binding, as do D197 exact preservation, normalized exact imaginary zero, Float signed imaginary zeros, numeric hash/identity and Actor/P transfer.

**STOP (not approved):** detailed finite/infinite Complex component-result/NaN recovery table for `+`, `-`, `*`, `/`, unary minus; Float conversion order, precision, rounded/stable algorithms, `sqrt` exactness/branches and final public API construction/access/math selectors. In particular a not-yet-ratified `Complex(1,2) / Complex(0.0,0.0)` must yield a numerical result under E2, but its specific components/signs are *not* approved yet. Normative `spec/`, product, and tests are unchanged by the documentation work.

**Next slice:** single consolidated **D198 INVESTIGATION**, not new Ixxx or implementation; the agent researches the remaining result-table/precision/sqrt/API interaction and returns an exact decision packet, with **no source edits or tests**. I090 remains `status:blocked` for its public Complex remainder. The human reported local `git diff --check` clean and all tests PASS without a linked revision/run; this is not claimed as E2 implementation validation.

```text
D198_C_ARCHITECTURE=APPROVED
D198_B_COMPONENT_ADMISSION=APPROVED
D198_E2_ERROR_VS_NUMERIC_RESULT=APPROVED_2026_10_12
D198_EXCEPTION_OUTPUT_TABLE=OPEN
D198_PRECISION_SQRT_PUBLIC_API=OPEN
D198_FULL_DECISION=OPEN
I090_PUBLIC_COMPLEX_REMAINDER=BLOCKED
PRODUCT_OR_NORMATIVE_CHANGES_IN_THIS_COORDINATION=NONE
```

---

## 2026-10-12 — Next decision packet: Complex exceptional arithmetic (RESEARCH ONLY)

The owner-approved **C architecture** and **B Float-component admission** are preserved without any implied arithmetic approval. [The comparative investigation of complex division by zero and exceptional arithmetic](../../evidence/D198/D198_COMPLEX_EXCEPTIONAL_ARITHMETIC_AND_ZERO_DIVISION_RESEARCH_2026_10_12.md) was published at [documentation commit `e36a1f1f3aff4a043d86a8a78f06f6eb8555ca11`](https://github.com/guillermomolina/protos-project-docs/commit/e36a1f1f3aff4a043d86a8a78f06f6eb8555ca11).

**Proposed E2, NOT APPROVED:** exact/exact complex division by zero signals an Error, while operations entering the approximate Float domain do not signal an Error merely for IEEE numerical exceptional conditions (zero, Infinity, NaN, overflow, underflow). A later **detailed portable Complex exceptional-result table**, signed-zero behavior, component rounding/precision and stable algorithms remain unselected. Alternative E1 signals Error for any zero Complex divisor; alternative E3 rejects non-finite Complex arithmetic. The packet compares Python, Racket, Julia, C/C++, Apache Commons Numbers with the twelve GITHUB010 axes and falsification cases.

**Status:** D198 OPEN; I090 blocked only for remaining public Complex semantics. No new `Ixxx`, no product code or normative `spec/` modifications, no tests.

```text
D198_COMPLEX_EXCEPTION_POLICY=E2_PROPOSED_NOT_APPROVED
D198_EXCEPTIONAL_RESULT_TABLE=OPEN
D198_NUMERICAL_PRECISION=OPEN
```

---

## 2026-10-12 — Candidate B approved: NaN and Infinity Float components

**Explicit owner approval:** "Apruebo B" after reviewing the three alternatives for admission of exceptional Float components in Complex. **B is selected strictly for the Complex component VALUE DOMAIN:** either component may be any genuine semantic Float binary64 value, including canonical NaN and positive/negative infinity. Integer/Fraction components remain exact and Float signed zeros remain distinct under semantic identity.

**Evidence:** [D198 Candidate B owner approval and D197 invariant audit](../../evidence/D198/D198_B_NONFINITE_FLOAT_COMPONENT_ADMISSION_OWNER_APPROVAL_2026_10_12.md) (documentation commit [`705a2420908e8b8d7446e14f390b10c7ea2dee56`](https://github.com/guillermomolina/protos-project-docs/commit/705a2420908e8b8d7446e14f390b10c7ea2dee56)), following the [comparative investigation](../../evidence/D198/D198_NONFINITE_COMPLEX_COMPONENT_POLICY_RESEARCH_2026_10_12.md).

**Invariant preservation:** Exact Integer imaginary zero collapses to the real component even when the real component is Float infinity/NaN; imaginary Float +0.0 or -0.0 never collapses. Float NaN preserves its already-approved `==` nonreflexivity and `===` semantic identity; Complex normal/identity hash, authenticated family, Actor/P/Context transfer and exact component retention are unchanged. The current Java finite-only check is provisional and must be reconciled under later specification-backed implementation.

**Unapproved and STILL BLOCKING:** Complex arithmetic NaN/infinity propagation; exact/inexact complex division by zero; mixed numeric Float conversion and complex division/sqrt stability; final public constructor/component/math API and normative publication. This approval **does not** ratify the whole D198 or authorize an unfinished public API; **I090 remains blocked only for the public Complex remainder**. Prior D197 and I090-A/B/C1 are unchanged; no `protos` source/spec/tests modified here.

```text
D198_ARCHITECTURE=C_SELECTED
D198_NONFINITE_FLOAT_COMPONENT_ADMISSION=B_SELECTED_BY_OWNER_2026_10_12
D198_EXCEPTIONAL_ARITHMETIC=UNSELECTED
D198_PRECISION=UNSELECTED
D198_FINAL_PUBLIC_API=UNRATIFIED
D198_ISSUE=OPEN
I090_REMAINDER=BLOCKED
```

---

## 2026-10-12 — Owner-selected Candidate C architecture; D198 remains OPEN

**Explicit owner approval:** "ok apruebo C" (2026-10-12), after the architectural selection had been explicitly separated from NaN/Infinity and precision decisions. **Scope: Protos-first Complex architecture with the smallest justified host kernel**, ordinary Protos protocol/dispatch, no host-dominated double-pair representation, D197 invariants preserved and no mandatory cost on programs that do not use Complex.

**Durable evidence:** [D198 Candidate C owner selection and falsification](../../evidence/D198/D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md), published in [documentation commit `d4a2589f021ec49d07ed14d0f2b188d9dcedb848`](https://github.com/guillermomolina/protos-project-docs/commit/d4a2589f021ec49d07ed14d0f2b188d9dcedb848).

**Still unresolved:** NaN/Infinity admission and exceptional arithmetic; Float conversion/rounding, stable complex division and square-root precision; exact public constructors/component selectors/math module/error contracts. Exploratory API spellings are **not** independently ratified simply because architecture C was selected. The normative `spec/` and implementation are untouched. D198 stays **OPEN**, and I090 remains **BLOCKED only on its still-unratified public Complex contract**; A/B/C1 stay published.

```text
D198_ARCHITECTURE=C_SELECTED_BY_OWNER_2026_10_12
D198_EXCEPTIONAL_IEEE_POLICY=UNSELECTED
D198_PRECISION_POLICY=UNSELECTED
D198_COMPLETE_PUBLIC_API=UNRATIFIED
D198_ISSUE_STATE=OPEN
D197=UNCHANGED_RATIFIED
I090_REMAINING_PUBLIC_COMPLEX=BLOCKED
PRODUCT_OR_NORMATIVE_CHANGES=NONE
```

---

## Original 2026-10-11 intake (historical; superseded where selection state differs)

**Recorded:** 2026-10-11  
**Formal design issue:** [D198 / #876](https://github.com/guillermomolina/protos/issues/876) — **OPEN, not selected or ratified**  
**Preceding ratified model:** [D197 / #865](https://github.com/guillermomolina/protos/issues/865) — **CLOSED / RATIFIED**  
**Consuming implementation:** [I090 / #868](https://github.com/guillermomolina/protos/issues/868) — **OPEN**, remaining public Complex design/implementation gate  
**Verified product baseline:** [`guillermomolina/protos@c75dea75b8db045eb6833aeab1d5b8710eaa0556`](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556)  
**Implementation version / global specification revision:** `0.3.338-SNAPSHOT` / `0.1.455`  
**Previous human validation:** `git diff --check` clean; local focused 20 tests PASS; integrated `make test` PASS; pushed. The coordinator did not execute tests. [Pinned C1 publication evidence](../../evidence/I090/I090_C1_PUBLISHED_CANONICAL_COMPLEX_2026_10_11.md).

## Finding: not a contradiction in D197's approved mathematical invariants; incomplete executable contract for I090

D197 ratified a mathematical numeric **model**, not every public selector and exceptional operation needed to expose that model. In particular it approves the family hierarchy, real Integer/Fraction/Float components of Complex, exact-imaginary-zero reduction and inexact signed-zero preservation, Float contagion and exact-normalized arithmetic, mathematical value/identity/hash rules, and the policy that mathematically representable exact function results remain exact where specified. The ratified record explicitly reserves public `Complex(...)` API and selector conventions, detailed math-function/sqrt branch selectors, and IEEE exceptional arithmetic for subsequent decisions.

I090 was originally scoped to implement both the mathematical model **and** visible construction/functions/conformance that D197 had not specified fully. Publishing only the approved portion in independent slices is correct; claiming that the whole parent was done after its internal Complex foundation is not. The original I090-C / I090-D target should remain visible rather than silently deleting acceptance criteria or backfilling design approval.

### Product implementation boundary

- **I090-A**: [`888c2b4f2390b96de5990533fdb1b306587e0002`](https://github.com/guillermomolina/protos/commit/888c2b4f2390b96de5990533fdb1b306587e0002) — source-visible abstract Integer with SmallInteger/BigInteger canonical concrete families.
- **I090-B**: [`e769869d155a999501e96873eb7d26d633b7c2e7`](https://github.com/guillermomolina/protos/commit/e769869d155a999501e96873eb7d26d633b7c2e7) — exact `Integer/Integer` division and canonical Fraction.
- **I090-C1**: [`c75dea75b8db045eb6833aeab1d5b8710eaa0556`](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556) — authenticated, immutable Complex with real numeric components, normalization, equality/hash/identity, and isolated Actor/P transfer; Core exposes `Complex.recognizes` but **no** public constructor/operators/component selectors or `sqrt`.
- **Pending:** publicly usable Complex construction and arithmetic; public mathematical functions including `sqrt` and exact negative-square cases, branch-cut/error/NaN/Infinity policies, and the relevant end-to-end conformance/interop closure. A public program cannot currently construct an arbitrary Complex value even though internal runtime/JUnit tests can.

### Formal allocation and roles

D198 is a new **language Dxxx** because choosing observable Complex operations requires independent approval. It is not an `Ixxx`, not an ordinary implementation slice, and not a reopening/erasure of D197. The issue was created with exact label `family:D` and `status:ready`, unassigned, without an invented priority. Allocation uniqueness was verified against GitHub Issues and indexed durable records; D198 is the sole retrieved allocation. No owner approval was inferred from this allocation.

D198 can be explored next, using a single coherent research packet rather than several artificial micro-decisions. `AGENTS.work/DESIGN.md`'s GITHUB010 comparison/falsification/12-axis scoring applies. Compare prior art including Common Lisp, Smalltalk/Pharo, Python/cmath, C++/std::complex, and Julia, and reconcile with the current Protos source/spec before proposing any public API. The investigation must **STOP without code/spec publication or approval claim**; the owner chooses the exact semantics subsequently.

After that explicit approval and durable publication, **the existing I090** implements the remainder and its conformance, with human-executor validation and late version/changelog metadata. Never add a new Ixxx just to consume D198 absent an independently meaningful implementation scope. D197 remains historically closed.

### Gate / acceptance checklist

1. D198 research: source-grounded alternatives, counterexamples, exact example programs and failure behavior, 12-dimension justified scores and smallest-sufficient Protos recommendation.
2. Owner checkpoint: explicit approval of **precisely specified** public construction/operations/function semantics; any deferred claims and migration consequences recorded honestly.
3. Durable D198 decision ratification record under `docs/project/decisions/language/`, then normative specification under `guillermomolina/protos:spec/`, with separate governance and revision publication.
4. I090-C remainder: source-first Complex functionality with bounded Java numeric kernels only where unavoidable; integration and representative end-to-end conformance; correct Actor/P/Map and ordinary dispatch.
5. I090-D and closure: final affected/integrated human tests, clean final diff, same-commit late version/changelog, push, SHA-pinned publication and final issue closure evidence, or explicit owner-approved narrowing/reassignment.

**Native link limitation:** the available GitHub connector exposes Issue create/update/comments, but not native `blocked by` link mutation. The intended direct edge `I090/#868 blocked by D198/#876` is **PENDING NATIVE MUTATION**; textual cross-references are for readability and do not claim the native dependency exists. D198 is treated as a standalone top-level decision (no invented parent). Project/priority reconciliation is owned by the repository automation; no Project status was fabricated.

## State assertions

```text
D197_DECISION=RATIFIED_AND_CLOSED
D198_DECISION=OPEN_UNSELECTED
D198_GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/876
I090_A=PRODUCT_PUBLISHED
I090_B=PRODUCT_PUBLISHED
I090_C1=PRODUCT_PUBLISHED_HUMAN_SUITE_REPORTED_PASS
I090_REMAINDER=AWAIT_D198_PUBLIC_SEMANTICS
I090_PARENT_CLOSURE=NOT_SATISFIED
NATIVE_ISSUE_DEPENDENCY=UNVERIFIED_PENDING_MUTATION
NEW_TESTS_OR_PRODUCT_CHANGES_IN_THIS_COORDINATION=NONE
```
