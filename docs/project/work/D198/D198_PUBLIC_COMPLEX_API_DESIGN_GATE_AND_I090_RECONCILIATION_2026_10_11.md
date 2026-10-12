# D198 — Public Complex semantics design gate and I090 reconciliation

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
