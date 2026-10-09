# D197 — Exact numeric tower, canonical normalization and floating boundaries

**Decision:** RATIFIED — owner-approved core semantic model; normative specification and runtime implementation PENDING
**Owner approval:** 2026-10-09, active design conversation. Owner first corrected the proposal to make `BigInteger`, `Fraction` and `Complex` guest-visible Protos numeric types, approved de-promotion including zero-imaginary exact Complex, expressly accepted falsification recommendations 2 (IEEE complex signed-zero preservation), 3 (Float contagion), and 4 (approximation for nonrepresentable mathematical results), then answered **“si aprobado”** to fixing `Integer` at signed 64 bits and using `BigInteger` outside that range.
**Live design issue:** [D197 / #865](https://github.com/guillermomolina/protos/issues/865)
**Historical decisions affected:** [D006 / #164](https://github.com/guillermomolina/protos/issues/164), [D156 / #616](https://github.com/guillermomolina/protos/issues/616)
**Independent ratified input:** [D196 / #864](https://github.com/guillermomolina/protos/issues/864), [decision record](D196_MIXED_INTEGER_FLOAT_ARITHMETIC_PROMOTION.md)
**Platform design, NOT selected:** [PLAT056 / #866](https://github.com/guillermomolina/protos/issues/866)
**Normative owner:** `guillermomolina/protos:spec/semantics/VALUES_AND_COLLECTIONS.md`, plus any actually affected other normative owners
**Inspected product revision:** `0db24f00ff2d92d642351d7f7535517fe01a55ce` (2026-10-09 baseline). Recheck HEAD before implementation.

> This is a durable **non-normative** owner-decision record, not a claim that `spec/`, runtime code, test suite, or any product commit already conforms. The previously green local suite and clean diff are historical baseline facts, not D197 acceptance.

## Exact approved invariants (GITHUB021)

1. **Guest-visible semantic families.** `Integer`, `BigInteger`, `Fraction`, `Float` and `Complex` are Protos numeric types; no Java host class and no physical carrier defines their guest semantics. They belong in the integrated Number model, not an opt-in `std:math/Fraction` wrapper required to perform ordinary exact integer division.
2. **Signed 64-bit `Integer`.** Mathematical integral values from `-2^63` through `2^63-1`, inclusive, have canonical visible type `Integer`; integral values outside that interval have canonical visible type `BigInteger`. The boundary is portable, not a host/VM word-size option.
3. **Exact promotion and demotion.** Exact integral arithmetic preserves mathematical value without wrap/saturate/unrequested Float. Overflow returns BigInteger and a later operation returning to the signed-64 range returns Integer. `BigInteger` is a guest type; Java `java.math.BigInteger` may still be an encapsulated helper, never the semantic definition.
4. **Exact rational division and canonical fractions.** For a nonzero exact integral divisor, a mathematically integral quotient returns the canonical integer type; a nonintegral quotient returns exact reduced `Fraction`, numerator/denominator mathematically integral, positive denominator. `2/1` returns Integer; `1/2` returns Fraction; exact rational arithmetic can normalize back to Integer or BigInteger. Dividing by exact zero remains an Error. No silent Float division for exact integer operands.
5. **Exact Complex component generality.** Complex can contain exact Protos numeric components, including Fraction and BigInteger, without pre-converting to binary64. When the imaginary component is **exact zero**, canonicalization reduces the result to its canonical real component family. This is not a two-`double`-only complex model.
6. **IEEE exception to unconditional Complex demotion.** When a Complex contains an inexact IEEE zero imaginary component, canonicalization MUST NOT destroy the sign-of-zero / branch-cut distinction. The owner approved preserving IEEE information (recommendation 2), thus the simple rule `Complex(x,0) always becomes x` has an express domain-sensitive limit. Define all detailed Float-component normalization in conformance specifications; do not invent implementation policy that loses observable information.
7. **Float contagion.** When a standard arithmetic operation mixes an exact real numeric operand (Integer, BigInteger, Fraction) with Float, use operand-first IEEE binary64 conversion and arithmetic; approximate Float is a deliberate numeric domain boundary. This extends the policy previously selected for Integer/Float by D196, without rewriting D196 history. `Fraction.asFloat` must round the *exact rational quotient* once: separately converting huge numerator and denominator then doing `double/double` is observably wrong. Exact equality and ordering are **not** implicitly converted to Float.
8. **Mathematical functions.** Preserve exactness where the exact result is expressible (for example exact square roots of perfect rational squares, or negative perfect squares yielding exact Complex). Other mathematical functions may return an **explicitly specified approximate numeric result** instead of inventing irrational symbolic values or falsely promising mathematical exactness for `sqrt(2)`. The owner accepted this limited approximation policy (recommendation 4). Details of selectors, tolerances, principal branches and exceptionally large inputs remain the responsibility of precise normative/test contracts, not guesses about host `Math`.
9. **Pay-as-you-grow and protocol integrity.** Small Integer and Float calculations should permit primitive `long`/`double` paths; large arithmetic, rational representation, complex components, Actor/Process transfer, interop, reflection and generic boxed guest machinery must not be imposed unconditionally on trivial calculations. Protos standard arithmetic retains ordinary overridable message dispatch and guarded fallback (D013). This invariant constrains PLAT056 but selects no specific carrier or `Protos*Value` class.
10. **Conversions, value semantics and integration.** Numeric `==` remains mathematical and symmetric across supported numeric families; `===` distinguishes canonical semantic numeric families and preserves existing Float signed zero/NaN treatment. Standard numeric hash must be coherent with cross-family `==`; normal Map and IdentityMap remain distinct. Actor/Process isolation, equality, interop and numeric acceptance protocols must handle newly introduced families without host Java class leakage or quietly accepting lossy conversion.

## Adversarial evidence / falsification

- **Unconditional Complex normalization is false.** `sqrt(Complex(-4.0,+0.0))` and `sqrt(Complex(-4.0,-0.0))` can lie on opposite sides of the IEEE branch cut. Converting both to real `Float(-4.0)` destroys the information necessary to choose a signed complex result. Preserve inexact signed zero; compare Common Lisp rational-only normalization and the C++/Python complex branch-cut rules.
- **All functions cannot stay exact in this tower.** `sqrt(4)`, `sqrt(1/4)` and `sqrt(-4)` have exactly representable results; `sqrt(2)`, `sqrt(-2)`, `sin(1)` do not in the selected families. Approximation needs an explicit observable policy; exact rational operators do not lose precision.
- **Float conversion can fail catastrophically if naive.** `(10^400+1)/(10^400)` rounds to Float `1.0`; converting numerator and denominator separately to `double` yields `Infinity/Infinity=NaN`. A rational-to-binary64 algorithm must round the quotient directly, respecting ties-to-even, subnormals and overflow.
- **Cross-type numeric equality is not printed-format equality.** Exact `1/10` is not mathematically equal to binary64 `0.1`, whereas exact `1/2` equals binary64 `0.5`. `===` remains family-sensitive, and hashing must follow `==`, not `===`.
- **Domain recognition is affected.** The pre-D197 `Integer.recognizes(x)` contract matches ALL unbounded integers. `protos/lib/math/Integer.protos` uses it for gcd/lcm/factorial/pow; Array indexing and other integer-only boundaries rely on that domain. Introducing visible BigInteger without specifying what “accept an exact integer” means could break valid programs. The investigation proposed a common mathematical integer domain (possibly `Integer.recognizes` accepting both), but this exact recognition API delta was **not the explicit question** answered by the owner's final “si aprobado”; do not silently ratify it. Preserve the common-domain requirement and seek a separately explicit protocol choice if normative implementation exposes a choice.
- **Arbitrary-precision allocation/CPU is not constant.** Rationals can grow huge; exactness and canonical values are language guarantees, while cross-cancellation and GCD optimization are implementation objectives, not a promised O(1) cost.

See supporting primary-code/source references and comparison in the sibling [D197 falsification evidence](../../evidence/D197/D197_EXACT_NUMERIC_TOWER_FALSIFICATION.md).

## Exact D006 / D156 / D196 / PLAT056 consistency and precedence

- **D006/#164 (historically RATIFIED):** prospectively **amended** by D197 for the *visible small/large integer family split*, signed-64 boundary, exact `/` returning Integer/BigInteger/Fraction instead of Float, new Fraction/Complex families, domain-sensitive canonicalization and resulting numeric identity/hash behavior. Do not reopen, erase, or silently rewrite the historical decision.
- **D156/#616 (historically RATIFIED):** the former unbounded single Integer retained by D156 is expressly superseded on this numeric-family surface, but D156's rejection of eight *fixed-width arithmetic families* `UInt8/Int8/.../Int64` and its FFI/binary-boundary policies are not automatically reversed. `Integer` 64-bit versus `BigInteger` canonical magnitude is NOT reintroduction of eight fixed-width modular families.
- **D196/#864 (RATIFIED):** B1 operand-first Float promotion remains valid and governs mixing Float with signed-64 Integer. D197 additionally extends approximate mixed arithmetic to BigInteger and Fraction; preserve the B1 conversion-before-operation order. I089/#867 is a separate still-pending implementation of D196 and must reconcile the product HEAD and not assume D197 runtime is installed.
- **PLAT056/#866 (UNSELECTED):** its earlier research premise “BigInteger is not a guest-visible type” conflicts with and is overridden by this explicit D197 owner decision. PLAT056 remains responsible for carrier and representation architecture; its requirements about avoiding Java numeric wrapper proliferation and VM-wide host-type leakage remain. It must now compare candidates that support `BigInteger`, `Fraction`, `Complex` as guest numeric families while preserving primitive fast paths; no specific runtime representation is approved.
- **Interoperability compatibility:** existing expectations of unbounded `Integer` or Float-valued `Integer/Integer` division are deliberate prospective changes. Numeric family recognition, `Integer(...)`/`Float(...)` conversions, printing, serialization, interop/FFI, Actor and P-transfer, Map/IdentityMap, reflection, docs, library return domains and tests require direct impact audit.
- **Normative precedence:** product `spec/` remains authoritative for executable conformance until human-executed changes are actually published. This decision record establishes owner-approved *future contract*, not a pretense that the old specification is already changed.

## Owner approval scope and unselected detail

**Approved:** the ten core invariants above insofar as they restate the four expressly accepted choices and prior owner model corrections.

**Not separately selected:** detailed `Integer.recognizes` protocol behavior or a general `Integral.recognizes` owner, exact construction API and grammar spelling, complex IEEE exceptional arithmetic beyond preserving signed-zero branch semantics, complex ordering/error selectors, every interoperability conversion and error outcome, and particular Java/runtime carrier, Truffle specialization or performance target. Keep these subordinate decisions visible; raise a design checkpoint rather than silently choosing a new public contract.

The intended implementation sequence is to settle PLAT056 representation compatibility and the domain-recognition acceptance contract, then implement via a dedicated Ixxx work item. Do not conflate a closed design with an implemented language release.

## Release/validation status

```text
D197_OWNER_SELECTION=APPROVED_2026-10-09
D197_CORE_INVARIANT_DELTA=DOCUMENTED
D006_D156_PRECEDENCE=PROSPECTIVE_AMENDMENT
D196_B1=RETAINED
PLAT056_ARCHITECTURE=UNSELECTED
NORMATIVE_SPEC=NOT_UPDATED_BY_THIS_PUBLICATION
RUNTIME=NOT_IMPLEMENTED_BY_THIS_PUBLICATION
TESTS=NOT_EXECUTED_BY_THIS_PUBLICATION
PRODUCT_PUBLISH=NOT_PERFORMED_BY_THIS_PUBLICATION
```

## AI assistance

Comparative evidence synthesized with AI assistance; project owner selected the semantics directly. No independent execution or green validation of new code is claimed.
