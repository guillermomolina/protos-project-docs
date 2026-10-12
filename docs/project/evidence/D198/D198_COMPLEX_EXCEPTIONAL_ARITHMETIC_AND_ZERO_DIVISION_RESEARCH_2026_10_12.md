# D198 — Complex exceptional arithmetic and division by zero: decision research

**Recorded:** 2026-10-12  
**Decision issue:** [D198/#876](https://github.com/guillermomolina/protos/issues/876)  
**Consumer:** [I090/#868](https://github.com/guillermomolina/protos/issues/868) (remaining public Complex only)  
**Status:** RESEARCH / PROPOSAL; **no owner approval of this arithmetic policy**.  
**Previously owner-approved within D198:** [architecture C](D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md), [non-finite Float component admission B](D198_B_NONFINITE_FLOAT_COMPONENT_ADMISSION_OWNER_APPROVAL_2026_10_12.md).  
**Normative authority:** [VALUES_AND_COLLECTIONS.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/VALUES_AND_COLLECTIONS.md); [ERRORS.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/ERRORS.md); ordinary Protos call/message semantics and D013 standard operator overrideability.  
**Non-goal:** this research does not ratify floating-point complex precision, every exact NaN/sign output in all mixed exceptional expressions, sqrt branch cuts, or the final public API.

## Exact problem and separation of approval boundaries

The owner selected that a Complex component may be *any* authenticated Float, including NaN and infinities. That is a **value-domain choice**, not a decision about the exceptional *results* of Complex `+`, `-`, `*`, `/`, unary minus, or `sqrt`.

The next narrow semantic question is how standard Complex arithmetic reports **division by zero and IEEE exceptional conditions**: Error versus numerical Float special values. This choice has a clear future compatibility cost and can be selected *without* prematurely ratifying every non-finite-result table or stable rounding algorithm, provided the open numerical contracts remain explicitly blocked.

### Existing Protos invariants that any candidate must respect

1. For exact real `1/0`, `Error` is normative. Exact Integer/Fraction division and reduction are unbounded/exact; no semantic overflow, wrap or unrequested Float.
2. For `1.0/0.0`, `-1.0/0.0`, `0.0/0.0`, IEEE binary64 Float returns the corresponding infinities/NaN **without Error**. Mixed exact real/Float operands are rounded *operand-first*, then use binary64.
3. Float supports two signed zeros, NaN, infinities and subnormals. Floating arithmetic uses roundTiesToEven for each normative basic binary64 operation and must not silently use wider contracted arithmetic.
4. D197 Complex holds authentic exact Integer/Fraction and Float components without collapsing them to two doubles. An exact Integer zero imaginary part causes canonical demotion to the real value; a Float +0.0/-0.0 imaginary part does not.
5. D198 B admitted non-finite Float components. `==` remains NaN-nonreflexive, `===` Float NaN-identity canonical and Float signed-zero-sensitive; normal hash coherence, family identity and Actor/P/Context transfer remain binding.
6. Protos public operations use ordinary overridable message dispatch (D013); source-expressible complex algorithms belong in Protos, irreducible kernels in a minimal Java boundary (approved D198 C). Unused Complex costs must not spill into primitive Integer/Float hot paths.
7. Current `ProtosComplexNumericSupport` is an internal authenticated shell, not a source-executable Complex API. A public factory, operators, or exceptional outputs cannot be inferred from Java's provisional implementation or this research.

## Prior art: six credible systems, three distinct semantic approaches

| System | Verified behavioral lesson | Material difference from Protos |
|---|---|---|
| **Python / CPython** | CPython's `Objects/complexobject.c` maps complex quotient domain failures to `ZeroDivisionError`, while Python `cmath` recognizes NaN/infinite components and signed-zero branch cuts. ([source](https://github.com/python/cpython/blob/main/Objects/complexobject.c), [cmath](https://docs.python.org/3/library/cmath.html)) | Python's built-in Float division by zero also raises, unlike Protos; its complex components are binary64, not exact Fraction/BigInteger. |
| **Racket** | Explicitly distinguishes **exact-zero** divisor (exception) from **inexact-zero** arithmetic (IEEE infinity/NaN for defined scalar cases), and has precise NaN/signed-zero equality vs identity rules. ([Number reference](https://docs.racket-lang.org/reference/numbers.html)) | Racket constrains its complex component exactness combinations differently. Scalar examples alone do not prove a complete complex-zero quotient table. |
| **Julia** | `ComplexF64` division has explicit Infinity handling, a robust quotient method and scale-up/scale-down paths to avoid intermediate overflow/underflow. ([Julia complex.jl](https://github.com/JuliaLang/julia/blob/master/base/complex.jl)) | Julia's type promotion and exact complex representation do not map onto D197; particular zero-divisor signs/values must not be assumed without test/source proof. |
| **ISO C / IEC 60559 complex** | Annex-G-style complex arithmetic models nonfinite components and constrains intermediate overflow; `CX_LIMITED_RANGE` relaxes that correctness protection. ([C complex arithmetic](https://en.cppreference.com/c/language/operator_arithmetic), [complex types](https://en.cppreference.com/c/language/arithmetic_types)) | The C rule that *any* complex with an infinite part may be classified as a complex infinity is not a license to collapse distinct Protos component identities or invent a projective sentinel; IEEE scalar rules do not determine every complex zero-divisor result automatically. |
| **C++ std::complex** | Arithmetic and scalar/complex overloads exist, and implementations pay extra attention to exceptional cases; `-fcx-limited-range` can disable those protections ([operator docs](https://en.cppreference.com/cpp/numeric/complex/operator_arith3)). | C++'s floating template specializations and matching-type scalar overloads omit Protos's exact rational components and receiver-first dynamism. |
| **Apache Commons Numbers Complex** | Open source Java library explicitly models C99 Annex G special cases without throwing exceptions; its multiply/divide routines attempt recovery from `NaN + iNaN`, including **nonzero/zero**, **infinite/finite** and **finite/infinite**. ([API](https://commons.apache.org/proper/commons-numbers/commons-numbers-docs/apidocs/org/apache/commons/numbers/complex/Complex.html), [source](https://github.com/apache/commons-numbers/blob/master/commons-numbers-complex/src/main/java/org/apache/commons/numbers/complex/Complex.java)) | Its binary64-only representation and internal special-case machinery are implementation references, not a substitute for D197 component exactness or a preapproved Protos Java-side numeric API. |

**Underlying approaches, not cosmetic syntax:** (1) exceptions for zero complex quotient (Python), (2) mathematically exact versus approximate/IEEE-domain failure split (Racket), (3) dedicated complex IEEE/C99 special-value algorithms (Julia, C/C++, Apache). Their disagreements demonstrate why "NaN is allowed as a component" alone does not determine division.

## Candidate comparison for the NEXT owner choice

### E1 — Always Error for a zero Complex divisor (strict complex exceptional boundary)

Any complex-valued quotient with a mathematically zero divisor signals an Error, whether the zero was made of exact Integer/Fraction or signed Float zeros. Float special values remain valid components for storage and nondivision arithmetic. This resembles Python's complex quotient policy but creates a new Protos discontinuity: `1/0.0` produces Float infinity while an equivalent Complex denominator with signed Float zero would signal Error. The choice is simple and predictable but not a natural extension of D197 Float arithmetic.

### E2 — Preserve exact Error; use an IEEE-style *non-signaling* exceptional domain for approximate Complex operations (**RECOMMENDED, UNAPPROVED**)

1. **Exact division:** when every input component involved in the quotient is exact Integer/Fraction, perform exact rational complex division and normalize the result; a mathematically zero denominator signals a fresh Protos Error, as D197's exact-domain division rule requires. Never fabricate a numeric infinity from exact arithmetic.
2. **Approximate division:** when the standard quotient involves a genuine Float component (in either operand), IEEE exceptional arithmetic **does not signal a Protos Error solely because a zero/NaN/Infinity/overflow/underflow occurs**. It yields legitimate semantic Float components, including signed zeros/NaN/infinities as specified by a later **deterministic complex exceptional-result table**. Do not claim an exact result table from the scalar IEEE rule alone.
3. **Other arithmetic:** ordinary standard real/Float binary64 suboperations keep their existing D197 IEEE rules; non-finite Complex inputs are not rejected merely because a component is exceptional. The exact behavior of complex multiplication's `0×Infinity` recovery, infinity algebra, NaN propagation and signed-zero outcomes still requires the follow-on detailed contract.
4. **No projective collapse:** preserve stored component pairs, Float identity and D197 canonicalization; do not introduce a complex-infinity singleton or merge `NaN+iInfinity` with unrelated infinite pairs.
5. **Separation from precision:** inexact operation admission and failure **policy** are decided here; exact placement of binary64 conversions, scaling, operation-level rounding and the total exceptional-result table are an explicit further design gate before executable I090 implementation. No host FP flags or runtime-configurable rounding mode are introduced.

This is an approval target **only for the Error-vs-special-Number policy**. It does not endorse any unstated mathematical extension to NaN or Infinity; `Complex(1,2)/Complex(0.0,0.0)` is deliberately classified as **non-Error exceptional numeric computation**, but its exact result component values/signs are **not ratified by E2 alone**.

### E3 — Always signal Error for exceptional Complex arithmetic

Allow non-finite components under already-approved B, but signal Error whenever an operator encounters NaN/Infinity or an inexact zero divisor. Easier to implement initially, but undermines B's usefulness and departs from Float semantics. Changing it later alters public observable Error-versus-value behavior.

### E4 — Naive componentwise double formula without exceptional special handling (REJECTED)

Use `(ac+bd)/(c²+d²)` directly for all operands, convert exact components to doubles and return whatever results appear. **Not a surviving candidate.** Violates exact Fraction/BigInteger retention and normalization, can overflow the intermediate denominator even for a representable quotient, spuriously NaN on infinity/zero terms, and changes portable results with host evaluation strategy.

## GITHUB010 evaluation: twelve mandatory criteria (1–5 and evidence confidence)

The table scores *the policy choice*, not a finished total floating-point complex specification. H = high confidence, M = medium, L = low.

| Criterion | E1 strict zero Error | E2 exact/approx split (recommended) | E3 reject exceptional arithmetic |
|---|---|---|---|
| 1 Invariant correctness | **3 H**: retains exact Error, conflicts with Float-domain continuity | **5 M**: preserves both D197 exact and Float distinctions | **2 H**: Float value domain fails as soon as composed |
| 2 Protos alignment | **3 M**: predictable Error but complex-only exception | **5 M**: one numeric domain distinction, ordinary messages | **1 H**: unnecessary new exceptional blacklist |
| 3 Present proportionality | **4 H**: simple zero check | **4 M**: cheap failure choice; later result-table work real | **4 M**: easy to reject inputs |
| 4 Incremental growth | **2 H**: later removal changes visible Error semantics | **5 M**: stable Error-versus-value envelope before detailed algorithm | **2 H**: cannot use approved B for numeric work |
| 5 Future options | **2 M**: locks out IEEE-style complex zero handling | **5 M**: admits robust algorithm upgrades and new math functions | **2 M**: eventual redesign needed for real IEEE use |
| 6 Scalability | **4 M**: local, stateless and easy dispatch | **5 M**: local stateless calculations, kernels only if used | **4 M**: local but propagating Errors requires handling |
| 7 Conceptual simplicity | **4 M**: uniform Complex zero Error, not uniform Number | **4 M**: exact vs inexact follows existing real split | **2 H**: component permitted, operations forbidden |
| 8 Portability | **5 H**: easy Error policy on all hosts | **4 M**: portable contract needs explicit binary64 exceptional table | **5 H**: easy Error signaling |
| 9 Runtime cost | **5 H**: finite zero check, no extra integer overhead | **4 M**: robust Float path costs when Complex is used, not otherwise | **4 M**: checks exceptional components on every Complex op |
| 10 Failure / operability | **4 H**: clear Error | **4 M**: predictable non-signaling envelope; NaN needs diagnostics/tests | **3 M**: lots of surprising Error sites |
| 11 Deferral / reversibility | **2 H**: changing Error into value is semantic break | **5 M**: semantics compatible with later result tables if no output promises invented | **2 H**: semantic migration breaks handlers |
| 12 Evidence maturity | **5 H**: Python precedent | **4 M**: Racket, Julia, C99, Apache precedent; adaptation still untested | **2 L**: special-value composability not well supported |

### Anti-overengineering and anti-underengineering, candidate by candidate

- **E1:** pay-now is minimal; growth requires replacing its observable Error contract. Its simplicity is real but its divergence from Protos Float is permanent unless changed.
- **E2:** pay-now for a *policy decision* is low; future precise algorithms/test tables are unavoidable and charged only where Complex is used. Underengineering would be claiming that a literal `double` formula implements this policy. Overengineering would be constructing an entire numerical backend, projective complex infinity type or mutable IEEE FP status to support it.
- **E3:** pay-now implementation is cheap, but accepted B values mostly turn into failures; future expansion requires revising exposed behavior. Its exception checking may actually cost more than an ordinary finite fast path with exceptional fallback.

## Adversarial falsification and exact examples

**Notation:** `Complex(a,b)` and Float named constants below are illustrative *candidate* source spellings, not yet ratified factory or special-value accessors. The semantic exactness of each component, not just whether the original expression text mentions a dot, controls the mathematical domain. Results marked OPEN are genuinely unspecified until the later numerical contract.

| Case | Existing binding / proposed E2 consequence |
|---|---|
| `1 / 0` | D197 exact real quotient => **Error** (already normative) |
| `1 / 0.0` | D197 mixed real Float => **+Infinity** (already normative) |
| `0.0 / 0.0` | D197 Float => **NaN** (already normative) |
| `Complex(1,2) / Complex(0,0)` | The denominator normalizes to exact Integer zero; **Error** in exact quotient under E2 |
| `Complex(1,2) / Complex(0.0,+0.0)` | Denominator is a Complex with Float components; E2 => **no Error merely for floating zero**, exact exceptional Float result component values/signs **OPEN** |
| `Complex(1,2) / 0.0` | Float zero operand => **non-signaling approximate domain**, detailed result table **OPEN** |
| `Complex(0,+0.0) / Complex(0,-0.0)` | Approximate zero over approximate zero => non-Error numerical special values; exact output **OPEN** |
| `Complex(1,2) / Complex(+Infinity,1.0)` | Approximate finite/infinite case; no Error solely due to infinity; exact signed-zero result **OPEN** |
| `Complex(+Infinity,+0.0) * Complex(0.0,1.0)` | Naive algebra can create spurious NaNs; E2 does NOT approve the naive result; recovery table/algorithm **OPEN** |
| `Complex(NaN,-0.0) + Complex(2,3)` | NaN component allowed by B; E2 does not turn NaN presence into Error; exact NaN propagation/signed-zero **OPEN** |
| `Complex(1,2) / Complex(1,-2)` | Exact nonzero quotient gives exact reduced real `-3/5`, imaginary `4/5` under preexisting exact rational model; no double conversion |
| `Complex(2^1024,1) / Complex(+Infinity,1.0)` | Mixed exact/Float boundary must address operand-first rounding and Infinity; cannot shortcut to exact zero or do `BigInteger` in Java everywhere |

**Strongest counterargument to E2:** a quotient by a complex inexact zero lacks a unique finite mathematical value or approach direction; it is unsafe to promise a particular Infinity or NaN pair simply because scalar IEEE 754 has a rule for real division by zero. **Resolution:** E2 approves only the non-signaling **result-domain and Error boundary**, not a guessed tuple. The next detailed exceptional-result table must explicitly choose a deterministic rule, justify it against C99 Annex G/Apache and test signed-zero approaches. If that cannot be made portable without unacceptable special cases, the owner can still revisit E2 **before** normative/public release.

## GITHUB021 approved-invariant consistency audit

- **Approved D197 exact division Error:** preserved by E2, even when an exact Complex divisor canonicalizes to Integer zero.
- **Approved D197 Float basic arithmetic non-Error:** preserved; does not silently turn Float zero into an Error merely because Complex is involved.
- **Approved D197 operand-first real mixing:** preserved for already-defined real operations; *new* mixed Complex component-conversion sequencing remains an explicit open precision gate.
- **Approved D197 exact component/value preservation:** E2 keeps exact/exact complex division exact. No automatic two-double representation mandated.
- **Approved D197 signed imaginary zero, hash, identity, transfer:** preserved in principle; the forthcoming exact component-output table must be checked against these.
- **Approved D198 architecture C:** E2 permits narrow native stability kernels only where mathematically justified, but no unconditional cost to simpler programs.
- **Approved D198 admission B:** E2 treats legitimate NaN/Infinity-bearing Complex as values, without an automatic Error at arithmetic admission.
- **New unresolved choices:** no hidden `sqrt` policy, result table, numeric tolerance, NaN propagation convention, or special sentinel is selected.

## Precise next gate after E2, if approved

Research and request owner ratification of a **finite/infinite result table** for `+`, `-`, `*`, `/` and unary negation, including (a) IEEE component roundings vs complex-level corrected results, (b) sign of zeros and infinities, (c) true NaN propagation vs spurious NaNs, (d) exact-component and Float component promotion, (e) scaled/stable division, (f) exceptional transfer and printing/interoperability. Then resolve `sqrt` precision/branch cuts and final public selectors as needed. Keep all within D198 and existing I090; no micro-Issue by default.

### Investigation and publication boundary

Current Protos source/spec/AGENTS materially examined: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/ERRORS.md`, `spec/semantics/CALLABLES.md`, `src/main/java/com/guillermomolina/protos/runtime/ProtosComplexNumericSupport.java`, `src/main/java/com/guillermomolina/protos/execution/ProtosStandardComplexProtocol.java`, `guillermomolina/protos-project-docs/AGENTS.md`, D197 ratification and D198 prior architecture/B records, GitHub #876 and #868. External comparative primary docs/source cited in the table above. Project's previously published I090-C1 baseline remains [`c75dea75b8db045eb6833aeab1d5b8710eaa0556`](https://github.com/guillermomolina/protos/commit/c75dea75b8db045eb6833aeab1d5b8710eaa0556); current branch may have advanced.

**No code changes, normative spec changes, tests, builds, execution or product publication.** This document is **research only**, not the owner's decision.

```text
D198_ARCHITECTURE=C_APPROVED
D198_FLOAT_COMPONENT_ADMISSION=B_APPROVED
D198_EXCEPTION_POLICY_PROPOSAL=E2_UNAPPROVED
D198_EXACT_NONZERO_ARITHMETIC=PRESERVE_D197
D198_COMPLEX_RESULT_TABLE=OPEN
D198_PRECISION_AND_SQRT=OPEN
D198_FINAL_API=OPEN
I090_PUBLIC_COMPLEX=BLOCKED
```
