# D198 — Exceptional Float component admission: comparative research (2026-10-12)

**Lifecycle:** INVESTIGATION ONLY — **NO OWNER APPROVAL** of non-finite Complex components, arithmetic exceptions, numerical accuracy, or final public API.  
**Decision:** [D198/#876](https://github.com/guillermomolina/protos/issues/876).  
**Already approved separately:** [Candidate C architecture](D198_C_ARCHITECTURE_OWNER_SELECTION_2026_10_12.md) (Protos-first with minimal host kernel); D197 numeric invariants remain ratified.  
**Scope of this packet:** whether a semantic Complex may retain Float NaN, +Infinity, or -Infinity in a real or imaginary component. **Non-goal:** selecting the complete complex division/sqrt numerical result table.

## Current Protos facts

1. The numeric model in [VALUES_AND_COLLECTIONS.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/VALUES_AND_COLLECTIONS.md) already requires Float IEEE binary64 NaN, signed zeros, infinities, subnormals and componentwise mixed equality/identity/hash; Float NaN is unequal to itself under numeric equality, but all Float NaNs are semantically identical under the nonoverridable identity relation. The ordinary numeric hash of NaN is a single class within an execution.
2. D197 ratified Complex components that preserve their **genuine semantic real family**: exact Integer/Fraction stay exact; Float retains binary64 semantics and signed zero. Exact Integer zero as imaginary component normalizes back to the real component; imaginary Float +0.0/-0.0 does not collapse.
3. The current normative paragraph explicitly says Complex NaN/Infinity Float components are **not defined by this revision**, rather than forbidden as a final design. Their public admission therefore requires owner approval and normative revision.
4. [ProtosComplexNumericSupport.java](https://github.com/guillermomolina/protos/blob/main/src/main/java/com/guillermomolina/protos/runtime/ProtosComplexNumericSupport.java) currently rejects non-finite Float components through Double.isFinite(), with a comment identifying this as an **I090-C1 temporary internal precondition**. This restriction is not a ratified future-language prohibition.
5. [ProtosCurrentNumericRelations.java](https://github.com/guillermomolina/protos/blob/main/src/main/java/com/guillermomolina/protos/execution/ProtosCurrentNumericRelations.java) already computes Complex componentwise equality and hash, using Float NaN / infinity semantics and retaining componentwise identity. The implementation must still be audited and tested on admitted exceptional components before claims of conformance.
6. Exact division by zero signals Error in Protos. Float binary64 division by zero returns IEEE infinity/NaN. Neither fact alone establishes the exceptional **Complex arithmetic** contract.

## Relevant cross-system comparison (five systems / more than three design approaches)

| System | What it demonstrates | What Protos must not copy uncritically |
|---|---|---|
| Python complex/cmath | Complex with NaN/Infinity components is legitimate; cmath exposes isfinite/isinf/isnan and uses signed zeros to choose branch-cut sides ([Python documentation](https://docs.python.org/3/library/cmath.html)). | Python real and complex division by zero raises exceptions; that differs from Protos's ratified Float binary64 division behavior. Python also converts complex components to floating-point rather than keeping exact Fraction/BigInteger. |
| Julia Complex | NaN/Inf are valid components ([manual](https://docs.julialang.org/en/v1/manual/complex-and-rational-numbers/)); its implementation treats IEEE exceptional and overflow cases explicitly, including scaled complex division ([source](https://github.com/JuliaLang/julia/blob/master/base/complex.jl)). | Julia component-promotion and sqrt(-1) real-domain behavior are not D197's approved exact component and sqrt goals. |
| Racket | General Number hierarchy includes exact and inexact complex numbers, NaN, infinities, signed zeros and a special exact-zero division rule; numeric = and eqv? differ for NaN ([Numbers](https://docs.racket-lang.org/reference/numbers.html), [Generic numerics](https://docs.racket-lang.org/reference/generic-numbers.html)). | Racket constrains inexact complex component combinations more than D197 permits. Its exceptions for some zero cases cannot simply replace the Protos Float contract. |
| C99 / IEC 60559 complex | Built-in complex operators treat nonfinite parts as first-class exceptional inputs; complex infinity and finite/infinite division require special rules, and naive intermediate overflow is explicitly problematic ([C arithmetic](https://en.cppreference.com/w/c/language/operator_arithmetic), [projection](https://en.cppreference.com/w/c/numeric/complex/cproj)). | C's representation, status flags, exact-integer division rules and some undefined/error behavior are not portable Protos policy. |
| C++ std::complex | Floating real/imaginary components and IEEE non-finite values are routine; even mixed infinite/NaN complex values have projection rules ([std::complex](https://en.cppreference.com/w/cpp/numeric/complex), [std::proj](https://en.cppreference.com/w/cpp/numeric/complex/proj)). | Its ordinary std::complex floating specializations cannot represent Protos BigInteger/Fraction component values exactly. |
| R complex | Complex values are constructed from numeric real and imaginary vectors and used with usual complex arithmetic ([base documentation](https://search.r-project.org/R/refmans/base/help/complex.html)). | Its vectorization/coercion model and available evidence do not establish the exact exceptional-result table that Protos would need. |

No single precedent simultaneously has D197's exact component families, IEEE Float, identity/hash, Protos's dispatch, and Actor/P transfer. Precedents inform constraints; they do not ratify source-level Protos examples.

## Distinct candidates for the component ADMISSION question

**A — FINITE_ONLY:** permanently reject Complex values that contain non-finite Float components, despite non-finite Float being legitimate elsewhere in Number. Conserves current implementation precondition. Fails closure under otherwise normal Float operations: a Float component that overflows to Infinity cannot remain part of Complex without signaling an unmotivated Error or performing an invented conversion.

**B — ALL_FLOAT_VALUES (RECOMMENDED; UNAPPROVED):** permit every semantic Float binary64 value as either Complex component, including NaN and ±Infinity, while preserving D197 Float identity, signed zero, exact component normalization and no host-payload exposure. Mathematical operations with these values require a separate **explicit** IEEE-compatible result/exception policy; merely admitting components does not approve specific division, sqrt or multiplication tables.

**C — ADMIT_BUT_REJECT_ARITHMETIC:** permit storage and transfer of NaN/Infinity Complex values but signal Error whenever a Complex arithmetic operator encounters them. Preserves an escape hatch but establishes a new discontinuity: ordinary Float can compute NaN/Infinity while Complex containing the same Float cannot. It might be viable as a temporary implementation stage, not as a coherent default long-term language contract.

**Rejected variant D — SYNTHETIC_COMPLEX_INFINITY:** collapse every complex Infinity into one global sentinel/Riemann-sphere projection. Rejected as a default because it destroys exact component and signed-zero semantic identity and adds an unrelated special value family; an ordinary projection function could be a future library facility without changing stored values.

## GITHUB010 twelve-axis comparative scoring

All scores are preliminary 1 (weak) to 5 (strong). H/M/L denotes evidence confidence. The reasons are deliberately focused on **component admission** only, not a premature score for the full arithmetic policy.

| Axis | A finite-only | B admit all | C admit/reject operations |
|---|---|---|---|
| 1 Correctness/invariants | 2 H: rejects legitimate Float outputs | 5 H: retains Float and D197 component semantics | 3 M: legitimate values but operations discontinuous |
| 2 Protos alignment | 2 H: special case in numeric family | 5 H: same real component universe | 2 H: arbitrary arithmetic family carve-out |
| 3 Present proportionality | 4 M: no exceptional-result design needed today | 4 M: removal of admission guard cheap, test burden real | 2 M: new branch and restrictions |
| 4 Incremental growth | 2 H: changing public rejection later | 5 H: later algorithms can refine without changing admission | 3 M: operations can be relaxed later |
| 5 Future-option resilience | 2 M: constrained numeric interoperability | 5 M: admits future IEEE numerical algorithms | 3 M: component representation extensible, operations closed |
| 6 Scalability | 4 M: ordinary numeric workloads unaffected | 5 M: no new guest family or global state | 4 M: failures local but costly handling |
| 7 Conceptual simplicity | 3 H: easy rule but counterintuitive distinction | 5 M: a Float is valid whether finite or not | 2 H: valid to store but invalid to compute |
| 8 Portability | 4 H: easy on IEEE hosts | 5 H: Protos already commits to binary64 special values | 4 M: straightforward but idiosyncratic |
| 9 Runtime/resource cost | 5 M: minimal validation code | 5 M: no cost when Complex unused; guard can be removed | 3 M: special-case checks on operations |
| 10 Failure/operability | 3 M: predictable early Error but surprising | 4 M: classification and IEEE table still required | 2 M: arithmetic Error after value admission |
| 11 Deferral/migration | 2 M: published finite-only API becomes compatibility trap | 5 M: component domain stable now | 3 M: later removal of arithmetic errors observable |
| 12 Evidence/risk | 3 H: implementation already behaves this way | 5 H: precedents support non-finite storage | 2 M: relatively little comparable precedent |

**Qualitative overengineering red flag:** interpreting B as an order to implement every C99 Annex G transcendental function or add floating-point status flags to Protos. Neither follows from component admission.  
**Qualitative underengineering red flag:** removing Double.isFinite and assuming raw textbook complex multiplication/division is IEEE-correct, or using Java double pairs that round exact Fraction/BigInteger components. B admits values; it does not waive the arithmetic correctness gate.

## Adversarial examples to resolve / test

These are **proposed test intentions, not current source acceptance**; symbolic Float.infinity and Float.nan below identify semantic Float values, not ratified public accessor spellings.

1. Complex(3, Float.nan) — should produce an authenticated Complex if B is approved; its numeric == with itself is false, while identity === with itself is true.
2. Complex(Float.infinity, 0) — should normalize to the real Float infinity due to the ratified **exact imaginary Integer zero** rule; it does not create a Complex.
3. Complex(Float.infinity, -0.0) — remains a Complex because a **Float signed-zero imaginary** component cannot collapse.
4. Complex(2, Float.infinity) — retains a Complex whose real component is exact Integer and whose imaginary component is IEEE Float infinity; no eager full-double conversion at construction.
5. Complex(Float.nan, -0.0) — normal numeric equality to itself false, semantic identity to itself true; normal hashing and IdentityMap behavior must follow already approved Float/Complex rules.
6. Complex(3, 4) / Complex(0, 0) — the denominator canonically reduces to exact Integer zero, so the exact zero-division Error policy remains binding.
7. Complex(3, 4) / Complex(0.0, 0.0) — an inexact complex zero divisor is an **unresolved** exceptional-arithmetic case: do not assume a precise NaN/Inf tuple or Error merely from admission B.
8. Complex(Float.infinity, 1.0) * Complex(0.0, 1.0) — NaN/Infinity propagation and potential spurious NaNs must be defined in the separate operation policy.
9. Actor/P transfer of an admitted special-component Complex — preserves component families, Float special semantic identity, signed zeros and canonical hash with no imported Prelude authority.
10. A forged object that delegates to Float/Complex — must remain inadmissible; B broadens semantic Float **value** domain, not public coercion or delegation-based numeric membership.

## Migration and implementation boundary

Under B the existing finite-check precondition in ProtosComplexNumericSupport is narrowed/removed **only after** owner approval and the normative update. Do not change the family token, component record, normalization on exact imaginary zero, public ownership, or signed-zero identity. Audit component-wise NaN identity, deterministic normal hash, actor transfer, interop, and mixed-form arithmetic. No source changes are authorized by this research.

The next independent numerical policy must still compare: (i) exact-domain zero divisor => Error; (ii) approximate Complex operations following a specified IEEE/Annex-G-inspired exceptional-result table; and (iii) always-Error complex-zero policies like Python. The choice must cover value classification, non-finite propagations, zero signs, overflow, underflow, rounding order and sqrt branches, and must not silently change ratified Float semantics.

## Decision checkpoint

**Recommendation:** SELECT **B — admit every semantic Float binary64 value as a Complex real or imaginary component**, including NaN and both infinities. Preserve D197's exact-imaginary-zero canonical reduction and signed Float zero nonreduction. **Do not** yet select the arithmetic exception table, precision policy, or final API names.

**Strongest counterargument:** the admission makes non-finite operands unavoidable in Complex arithmetic and requires a careful numerical specification and regression suite; finite-only is cheaper in the short term. **Answer:** that cost is real, but the current finite-only check would create an observable inconsistency and a later breaking API/domain change. Separating component admission from exceptional operators makes the smallest durable decision now without pretending the algorithms are solved.

```text
D198_ARCHITECTURE=C_ALREADY_SELECTED
D198_NONFINITE_ADMISSION_RECOMMENDATION=B_UNAPPROVED
D198_COMPLEX_EXCEPTIONAL_ARITHMETIC=OPEN
D198_COMPLEX_PRECISION=OPEN
D198_PUBLIC_API=UNRATIFIED
D198_SPEC_OR_PRODUCT_MODIFICATION=NONE
D198_HUMAN_TESTS_REQUESTED=NONE
```
