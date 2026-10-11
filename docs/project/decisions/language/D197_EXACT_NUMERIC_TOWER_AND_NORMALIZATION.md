# D197 — Exact numeric tower, canonical normalization and floating boundaries

**Current decision status: RATIFIED, amended 2026-10-11.** The **2026-10-11 amendment below is controlling** wherever it differs from the historical 2026-10-09 ratification preserved after it. Normative `guillermomolina/protos:spec/`, runtime, standard-library and conformance implementation remain **pending I090**; this project decision document does not itself change executable semantics.

**Approval provenance:** The project owner first raised the Smalltalk/TruffleSqueak-inspired distinction between the mathematical integer category and its small/large canonical implementations, explicitly asked to revise I090 for the new SmallInteger requirement, received the concrete proposed hierarchy and implications for `parent()`/`recognizes`/normalization, and then instructed **“ok enmienda D197”** on 2026-10-11. This approval amends that **specified hierarchy and its stated immediate consequences**, not any unrelated selector, syntax, factory, parser, rational class or host-carrier policy. The prior 2026-10-09 owner approval remains valid except for the precise revised clauses identified here.

## Current controlling amendment — 2026-10-11: Integer as exact domain, SmallInteger and BigInteger as canonical families

### Exact approved public semantic model

1. **Two levels, not two unrelated integers.** `Integer` is the **common semantic mathematical-integer category and protocol owner**, not the concrete signed-64 family. Its **canonical concrete** value families are `SmallInteger` and `BigInteger`, with visible ordinary prototype delegation:

   ```text
   Number
   ├── Integer                     common exact-integral protocol/recognition
   │   ├── SmallInteger            canonical signed-64 integral values
   │   └── BigInteger              canonical integral values outside signed-64
   ├── Fraction                    canonical exact non-integral rationals
   ├── Float                       IEEE 754 binary64
   └── Complex                     exact/inexact numeric components per D197
   ```

   This is a **semantic Protos prototype hierarchy**, not a mandate for a Java class for each prototype or for a Java BigInteger on ordinary numeric paths. A `Rational` public prototype is **not approved** as part of this amendment; mathematical set inclusion alone does not justify adding a permanent Core protocol owner.

2. **Canonical integer range.** Integral values in `[-2^63, 2^63-1]` inclusive have observable concrete family **`SmallInteger`**. Exact integral values outside that range have observable concrete family **`BigInteger`**. The fixed signed-64 boundary is portable and unchanged; there is **no canonical value whose concrete family is abstract `Integer`**. `SmallInteger` is not an overflow-prone fixed-width arithmetic family: mathematical arithmetic promotes to BigInteger on true overflow and demotes back to SmallInteger on exact results returning to range. Neither fraction nor complex is used as the carrier of an integer simply because it could represent one.

3. **Ordinary delegation and observability.** The Core prototype objects satisfy `Integer.parent() === Number`, `SmallInteger.parent() === Integer`, `BigInteger.parent() === Integer`. The observable parent of a concrete signed-64 integer such as `42` is **`SmallInteger`**, and the parent of an exact out-of-range integer such as `2^100` is **`BigInteger`**. This intentionally supersedes the former `42.parent() === Integer` contract and therefore requires a versioned normative/spec/conformance change. The two subfamilies inherit shared Integer operations through ordinary delegation; no duplication of arithmetic merely to represent the hierarchy. An object that merely delegates to `SmallInteger`/`BigInteger`/`Integer` is **not** thereby a semantic numeric value.

4. **Domain recognition without coercion.** `Integer.recognizes(value)` recognizes **both** canonical integral families and therefore remains true for exact small and large mathematical integers. `SmallInteger.recognizes(value)` recognizes exactly canonical signed-64 SmallInteger values; `BigInteger.recognizes(value)` recognizes exactly canonical beyond-signed-64 BigInteger values. These checks inspect intrinsic semantic family, not Java host class, representability in another family, immediate parent imitation, arbitrary delegation, user callbacks or automatic conversion. Only the exact canonical owner may invoke its standard recognizer, consistent with the existing `Integer.recognizes` receiver/arity discipline; a foreign or user object cannot opt into these families by defining a `recognizes` slot. The initial normative reconciliation must retain `Integer.recognizes` acceptance in existing integer-only `gcd`, `factorial`, `pow`, `powMod`, Array/indexing and library boundaries.

   | Value (conceptual) | `Integer.recognizes` | `SmallInteger.recognizes` | `BigInteger.recognizes` |
   | --- | --- | --- | --- |
   | `42` | true | true | false |
   | `2^100` | true | false | true |
   | `2/5` (canonical Fraction) | false | false | false |
   | `Float(42.0)` | false | false | false |
   | `Complex(0,1)` | false | false | false |

   A computed rational quotient that is mathematically integral is **already normalized** to its SmallInteger/BigInteger family, so its recognition is based on the resulting canonical value; it is not a recognizer-side conversion. Other newly introduced families' public `recognizes` methods are **not implicitly selected** by symmetry; their surface can be specified as part of the corresponding I090 normative closure without inventing an unrelated generic `Number.recognizes`.

5. **Exact arithmetic, result normalization, and interop.** The 2026-10-09 D197 rules on exact mathematical promotion, demotion, integral/fractional exact quotient, reduced `Fraction`, exact/approximate `Complex`, signed IEEE imaginary-zero preservation, D196 B1 mixed-Float contagion, exact `==`, family-sensitive `===`, coherent normal hash and actor/process/interop isolation **remain approved and unchanged**, except that references to a *canonical concrete signed-64 `Integer` result* now mean **`SmallInteger`**. For example, exact `2/1` yields canonical SmallInteger; exact nonintegral `2/5` yields Fraction; overflow of SmallInteger yields BigInteger and falling back into signed-64 yields SmallInteger. The abstract category `Integer` continues to own the common exact-integer operation domain, including results and parameters that may be either concrete family.

6. **Pay-as-you-grow is binding.** Implement ordinary SmallInteger computation using a primitive `long` wherever permitted by PLAT056 Candidate C. The new semantic `SmallInteger` prototype does **not** require `ProtosSmallIntegerValue`, eager guest boxing, a Java BigInteger conversion, extra numeric wrappers, or new materialization in a trivial `1+2`. Genuine large magnitudes and mathematically exceptional operations may use narrow encapsulated arbitrary-precision arithmetic; explicit typed Truffle/Java host APIs remain independent on-demand exceptions. `Fraction`, `Complex`, Actor/P, interop and generic reflection machinery are **not** mandatory on a trivial small integer path.

### Owner-invariant delta / GITHUB021

| Prior ratified statement | Current amendment | Status |
| --- | --- | --- |
| D197 2026-10-09 §§1–2: concrete signed-64 `Integer`; concrete out-of-range `BigInteger` | **Explicitly replaced:** `Integer` is the exact-integral base; `SmallInteger` is concrete signed-64; `BigInteger` is concrete beyond range | **Reopened by owner and superseded through 2026-10-11 explicit amendment** |
| D197 2026-10-09 §3: promotion/demotion between `Integer` and `BigInteger` | Magnitude threshold unchanged; canonical results now SmallInteger/BigInteger under common Integer | **Semantic family vocabulary amended; mathematical behavior preserved** |
| D197 2026-10-09 §4: exact division normalizes to `Integer` or `BigInteger` | Concrete integral quotient is canonical SmallInteger or BigInteger; Fraction still reduced and only when nonintegral | **Preserved, concrete naming amended** |
| D197 2026-10-09 §10: family-sensitive `===` and coherent hash | Concrete SmallInteger/BigInteger are the two families; abstract Integer is an ancestor, not a third concrete result family | **Preserved with explicit subtype observation** |
| Pre-D197 `Integer.recognizes` accepts all mathematical integers; D197 originally deferred this issue | Integer remains a genuine common recognizer for SmallInteger and BigInteger; concrete subtype recognizers are narrower | **Previously open selector scope explicitly selected** |
| D156 rejected eight fixed-width arithmetic families | SmallInteger is a **bounded canonical magnitude** with exact overflow promotion, not an `Int64` fixed-width modular arithmetic family | **Preserved** |
| D196 Float mixing; D197 Fraction/Complex/IEEE rules; PLAT056 primitive-first physical model | No changes to those substantive rules; new semantic name must not force new Java carrier costs | **Preserved** |
| Pharo/Squeak reference hierarchy | Common Integer plus small/large concrete types; Protos retains *portable 64-bit* threshold and one BigInteger rather than sign-split large subclasses | **Adapted, not blindly copied** |

**Alternative considered and rejected for this amendment:** using only `Number → Integer → BigInteger` (no SmallInteger) would preserve `42.parent() === Integer` and reduce migration cost, but would not express both small and large canonical integer families symmetrically as requested. The owner specifically elected to **amend D197** after the observed costs and the Pharo/TruffleSqueak comparison. No public `Rational` is introduced on mathematical elegance alone.

**Cross-runtime primary evidence:** [Pharo `Integer`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/Integer.class.st) is an abstract common integer class; [`SmallInteger`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/SmallInteger.class.st) inherits from it; [`Fraction`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/Fraction.class.st) inherits from Number; [TruffleSqueak `SqueakObjectClassNode`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/accessing/SqueakObjectClassNode.java) maps primitive Java `long` to guest SmallInteger class without mandatory boxed value; [TruffleSqueak `ArithmeticPrimitives`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/primitives/impl/ArithmeticPrimitives.java) uses primitive arithmetic with overflow fallback.

### Deliberately unselected contracts (not gates to re-decide the approved hierarchy)

- Specific construction/normalizing **factory and call** behavior of `SmallInteger(...)`, `BigInteger(...)`, and abstract `Integer(...)`, including whether explicit out-of-range conversion fails or normalizes, is **not inferred** from recognition or hierarchy alone. Preserve existing `Integer(value)` contract until normative reconciliation specifies the approved behavior; any genuine new public choice needs separate owner confirmation.
- Fraction/Complex constructor spelling, new public recognizer methods beyond the explicitly chosen integer-family trio, generic `Number.recognizes`, `Rational`, new mandatory general-purpose `Integral`/Number tag types, and host-Java carriers remain **unselected** unless separately ratified.
- Continue to respect the already recorded IEEE signed-zero Complex exception, equality/identity/hash/Map, Actor/P, interop and D013 overrides. The concrete implementation of guarded lookup across `SmallInteger → Integer` and `BigInteger → Integer`, with exact selected-home and invalidation semantics, belongs to I090/I091 and must pass conformance without semantic shortcuts.

### Implementation and coordination after amendment

- **D197 remains an approved language design.** The new exact SmallInteger / Integer / BigInteger semantic hierarchy and the three recognition domains are **ratified by the owner's 2026-10-11 instruction**. No additional design issue is needed for this specifically approved change. The historical 2026-10-09 body is preserved below so a reader can distinguish originally approved invariants from their explicit amendment.
- **I090/#868** owns normative `spec/`, visible Core bindings, `Integer`/SmallInteger/BigInteger delegation and constructors, source-level conformance, exact Fraction and Complex operations and integration. Revise I090-A plan to implement the approved new hierarchy, not the obsolete concrete signed-64 `Integer` model.
- **I091/#869** owns primitive-first physical carriers, guarded bytecode operations, `ProtosValueLookup`/numeric services, BigInteger containment, and proof of no mandatory small-path boxing, on the actual current product HEAD. D197 does **not** retroactively declare any untested C feasibility proof satisfied.
- **Current product** `guillermomolina/protos@ced3746f9664ec5ceb562461c077bdf0627e5663` still publishes `Integer` as the immediate parent of a small integer and `Integer.recognizes` for both existing host representations. `protos/tests/conformance/reflection/parent.protos` asserts the old small parent; `ProtosStandardIntegerProtocol` and `ProtosValueLookup.lookupGuardedInteger` anchor direct-send selection to the old path. These are **future I090/I091 changes**, not tests already passing for the amendment.
- Normative `spec/semantics/VALUES_AND_COLLECTIONS.md`, relevant `spec/` owners, `protos/lib/core/prelude.protos`, bootstrap, recognizers, integer protocols and conformance must be updated atomically with the approved implementation by the human-executor workflow. **No normative/prod source edit, new tests, benchmark or release is claimed by this decision publication.**

```text
D197_OWNER_AMENDMENT=EXPLICITLY_APPROVED_2026-10-11
D197_AMENDED_INTEGER_HIERARCHY=RATIFIED
INTEGER_RECOGNITION_DOMAIN=SMALL_AND_BIG
CANONICAL_SIGNED64_FAMILY=SMALLINTEGER
CANONICAL_OUT_OF_RANGE_FAMILY=BIGINTEGER
SMALLINTEGER_PARENT=INTEGER
BIGINTEGER_PARENT=INTEGER
RATIONAL_PUBLIC_PROTOTYPE=NOT_SELECTED
HOST_JAVA_CARRIER_CHANGE=NOT_SELECTED
PLAT056_C=UNCHANGED
NORMATIVE_PRODUCT_SPEC=AWAITING_I090
RUNTIME_CONFORMANCE=AWAITING_I090_I091
PRODUCT_TESTS=NOT_RUN_FOR_THIS_AMENDMENT
```

---

## Historical ratification — 2026-10-09 (preserved verbatim; superseded where identified)

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

The independently tracked implementation owner is [I090 / guillermomolina/protos#868](https://github.com/guillermomolina/protos/issues/868) (family:I), currently **BLOCKED** by the unselected PLAT056 numeric-carrier architecture and by the narrow open integer-domain recognition contract. I089/#867 remains independently READY for the already-approved D196 B1 rule. Do not conflate this closed design with an implemented language release; I090 requires its own conformance, green human tests and product publication.

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
