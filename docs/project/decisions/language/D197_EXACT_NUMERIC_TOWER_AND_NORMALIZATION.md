# D197 — Exact numeric tower, canonical normalization and floating boundaries

**Decision:** RATIFIED — original 2026-10-09 model amended by explicit owner approval of hierarchy C on 2026-10-11; normative specification and runtime implementation PENDING
**Owner approval:** 2026-10-09, active design conversation. Owner first corrected the proposal to make `BigInteger`, `Fraction` and `Complex` guest-visible Protos numeric types, approved de-promotion including zero-imaginary exact Complex, expressly accepted falsification recommendations 2 (IEEE complex signed-zero preservation), 3 (Float contagion), and 4 (approximation for nonrepresentable mathematical results), then answered **“si aprobado”** to fixing `Integer` at signed 64 bits and using `BigInteger` outside that range.
**Live design issue:** [D197 / #865](https://github.com/guillermomolina/protos/issues/865)
**Historical decisions affected:** [D006 / #164](https://github.com/guillermomolina/protos/issues/164), [D156 / #616](https://github.com/guillermomolina/protos/issues/616)
**Independent ratified input:** [D196 / #864](https://github.com/guillermomolina/protos/issues/864), [decision record](D196_MIXED_INTEGER_FLOAT_ARITHMETIC_PROMOTION.md)
**Platform design (subsequently selected):** [PLAT056 / #866](https://github.com/guillermomolina/protos/issues/866), primitive-first Candidate C, ratified 2026-10-10; distinct from **D197 hierarchy Candidate C**.
**Normative owner:** `guillermomolina/protos:spec/semantics/VALUES_AND_COLLECTIONS.md`, plus any actually affected other normative owners
**Inspected product revision:** `0db24f00ff2d92d642351d7f7535517fe01a55ce` (2026-10-09 baseline). Recheck HEAD before implementation.

> This is a durable **non-normative** owner-decision record, not a claim that `spec/`, runtime code, test suite, or any product commit already conforms. The previously green local suite and clean diff are historical baseline facts, not D197 acceptance.

## 2026-10-11 RATIFIED AMENDMENT — C: abstract Integer, concrete SmallInteger and BigInteger

**Status:** **OWNER APPROVED AND RATIFIED** on 2026-10-11 in the active design conversation: “ok apruebo C”. This is an explicit later amendment following the recorded 2026-10-11 proposal, comparison and implementation-cost investigation; it is **not** retrospective approval of the previously and incorrectly claimed premature ratification. The mistaken revision `866a367fc0f381b647d6e2db9f1f42af60abcc09` remains withdrawn. **Research and falsification:** [D197 C ratification and implementation-cost evidence](../../evidence/D197/D197_C_HIERARCHY_RATIFICATION_AND_IMPLEMENTATION_COST_2026_10_11.md).

```text
Number
├── Integer              abstract exact-integral domain / inherited common protocol
│   ├── SmallInteger     canonical signed-64 concrete family
│   └── BigInteger       canonical concrete family outside signed-64
├── Fraction
├── Float
└── Complex
```

**Exact GITHUB021 delta against 2026-10-09 approval:**

1. **Revised canonical visible family:** signed-64 values remain bounded at `[-2^63, 2^63-1]` but now belong to **guest-visible `SmallInteger`**, not concrete `Integer`. Values outside signed-64 remain guest-visible `BigInteger`. `Integer` is their common mathematical exact-integral prototype, **not a concrete family of numeric values**.
2. **Prototype delegation:** `SmallInteger.parent() === Integer`, `BigInteger.parent() === Integer`, `Integer.parent() === Number`. For numeric values, `42.parent() === SmallInteger` and `100000000000000000000.parent() === BigInteger`. Prior conformance of `42.parent() === Integer` and large integer `.parent() === Integer` is intentionally superseded; this is a public semantic breaking change.
3. **Common recognition:** `Integer.recognizes(x)` accepts *either genuine Core integer family*, including arbitrarily large integer values. `SmallInteger.recognizes(x)` accepts only signed-64 Core integer values; `BigInteger.recognizes(x)` only values outside signed-64. No `Fraction`, `Complex`, `Float`, or ordinary user object becomes an Integer simply through delegation or coercion. Preserve receiver/arity validation and trusted family classification. This explicitly resolves the former open mathematical integer-recognition gate.
4. **Canonical promotion/demotion and identity:** exact overflow promotes SmallInteger to BigInteger; arithmetic reducing magnitude returns SmallInteger. `==` and standard hash preserve mathematical equality/coherence; non-overridable `===` remains semantic-family sensitive; the new concrete public family labels participate in reflection, overrides and identity. No Java host class decides guest membership.
5. **Common protocol and factory:** ordinary standard integer operations and shared mathematical acceptance live under `Integer`, inherited by both concrete families. Existing `Integer(value)` is retained as the shared integer-domain conversion/normalization entry point where applicable, with the canonical concrete family as result. Novel public factory admission/error details for `SmallInteger(value)`/`BigInteger(value)` (if exposed) **remain unselected** and must not be fabricated by I090.
6. **PLAT056 compatibility:** Java `long` and Truffle primitive carriers may represent SmallInteger on ordinary paths, without allocating a dedicated Java per-value SmallInteger wrapper. `java.math.BigInteger` remains only for genuinely exceptional arbitrary-precision algorithms or explicitly requested host/Truffle ABI conversions; it is unrelated to the guest-visible `BigInteger` prototype. A public subtype change alone is not proof of no allocations.
7. **Unchanged D197 contracts:** exact rational division, Fraction normalization, IEEE Float contagion, Complex exact-zero versus signed inexact zero, standard numeric hash/identity, transfer, context isolation, cross-family equality and user-overridable D013 guarded method dispatch all remain as approved. No `Rational` public prototype is introduced.

**Implementation ownership and ordering:** [I090/#868](https://github.com/guillermomolina/protos/issues/868) implements the public prototype hierarchy and D197 normative semantics; [I091/#869](https://github.com/guillermomolina/protos/issues/869) implements PLAT056 primitive-first representation and contains unwanted Java BigInteger dependencies. They **must be coordinated**, but I090 does **not** wait for I091 to close. The visible prototype change must be delivered in the same coherent green gate as protected lookup and D013 override/deoptimization changes, particularly `ProtosValueLookup.lookupGuardedInteger`, `ProtosStandardIntegerProtocol` and the Bytecode DSL routes. [PERF040/#862](https://github.com/guillermomolina/protos/issues/862) remains blocked until executable conformance and pinned graph/timing evidence exist.

**Independent boundary concern, NOT part of C ratification:** the current Java NIO backend materializes Protos IpAddress/IpEndpoint objects and is threaded with an Integer prototype to construct 128-bit IPv6 bits. A possible cleaner interop adapter separating Java `InetAddress` and Protos's dynamic numeric/object model needs its **own source-grounded investigation/approval**, not a new network architecture silently invented by this amendment. See the linked evidence.

```text
D197_HIERARCHY_C=RATIFIED_OWNER_APPROVAL_2026_10_11
D197_ABSTRACT_INTEGER=APPROVED
D197_SMALLINTEGER_BIG_INTEGER_PUBLIC_PROTOTYPES=APPROVED
D197_INTEGER_RECOGNITION_COMMON_DOMAIN=APPROVED
D197_PRIOR_2026_10_09_INVARIANTS=RETAINED_EXCEPT_LISTED_C_DELTA
I090_PUBLIC_HIERARCHY_DECISION_GATE=RESOLVED
I090_I091_RUNTIME_IMPLEMENTATION=PENDING
D197_SPEC_AND_TESTS_UPDATED=NO
PERF040_GRAPH_VERIFIED=NO
```

---

## 2026-10-09 approved invariants (historical baseline; subject to ratified C delta above)


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

---

## Historical investigation proposal — 2026-10-11 (subsequently resolved by ratification of C)

> The statements in this section describe the **previous pending state before owner approval**. They are preserved as historical context, **not** current status. The authoritative later resolution is the ratified C amendment at the top of this record.

**Status: OPEN DESIGN QUESTION / RESEARCH NOT YET COMPLETED. This text does not amend, override or ratify any of the 2026-10-09 D197 clauses above.** The project owner proposed studying Smalltalk's distinction between an abstract mathematical `Integer` domain and `SmallInteger` / large-integer concrete subclasses, and asked for the proposal to be **recorded for investigation and later approval**. An earlier publication on 2026-10-11 incorrectly described the owner's instruction “ok enmienda D197” as approval of the specific semantic changes; the owner explicitly corrected that interpretation: **“te he dicho que lo metieras para ver si lo aprobamos o no, todavia no hemos investigado.”** The alleged approval and purported GITHUB021 resolution in that earlier revision are **withdrawn**. Do not use revision `866a367fc0f381b647d6e2db9f1f42af60abcc09` as evidence of ratification.

**Open proposal, NOT selected:** consider `Number → Integer → SmallInteger / BigInteger` as visible Protos prototypes, with `Integer` a common exact-integral domain, SmallInteger the signed-64 canonical concrete family and BigInteger the outside-signed-64 concrete family. Evaluate whether `Fraction` remains a sibling under Number or whether a rational common ancestor has independently justified public behavior. The actual 2026-10-09 D197 ratified model is **still** `Integer` canonical for signed-64 and `BigInteger` canonical beyond range until a subsequent exact-candidate approval supersedes it. `Rational` and `SmallInteger` are **not currently ratified public types**.

**Candidates that the investigation must compare:**

1. **A — D197 as ratified:** concrete signed-64 `Integer` plus separate concrete out-of-range `BigInteger`; define a consistent shared integer-domain acceptance API, preserving fast paths.
2. **B — minimal inherited mathematical domain:** `Number → Integer → BigInteger`, ordinary signed-64 values still directly under Integer; broad `Integer.recognizes`, separate large recognizer as justified. This preserves `42.parent() === Integer` but changes the role of Integer relative to the original D197.
3. **C — full Smalltalk-like visible split:** `Number → Integer → SmallInteger / BigInteger`. This changes `42.parent()` observably, may introduce more public factories/recognizers and guarded dispatch work, but offers explicit concrete family symmetry. Do not presume C is already preferred or approved.

**Mandatory falsification / specific questions:** (a) show what real Pharo/Squeak/TruffleSqueak expose vs Java primitive representation and where their model diverges from Protos; (b) account for `Integer.recognizes`, subclass-specific `recognizes`, receiver/arity and the fact that simple delegation must not admit user-defined objects as semantic numbers; (c) `parent()`, `===`, equality/hash, serialization, reflection, factory/call grammar, all Core prelude bindings and tests, plus mathematical acceptance by Array indexing, `gcd`, `factorial`, `pow`; (d) `ProtosValueLookup.lookupGuardedInteger`, `ProtosStandardIntegerProtocol`, D013 overrides/deopt, invalidation, transfer/interop and true cost of another prototype step; (e) prove PLAT056 Candidate C still permits primitive `long` on the ordinary path without extra `ProtosSmallIntegerValue` wrappers; (f) prove the new public type earns its long-term semantic/maintenance cost instead of choosing it solely for mathematical neatness; (g) score meaningful candidates under the project's Dxxx comparative and GITHUB021 falsification rules, with exact invariant deltas to the 2026-10-09 approval.

**Primary starting references, not an investigation conclusion:** [Pharo Integer](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/Integer.class.st), [Pharo SmallInteger](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/SmallInteger.class.st), [TruffleSqueak SmallInteger class lookup](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/accessing/SqueakObjectClassNode.java). The inspected Protos `ced3746f9664ec5ceb562461c077bdf0627e5663` still returns `Integer` from `42.parent()`, and has existing integer-prototype guards and conformances. The consumer work item is [I090/#868](https://github.com/guillermomolina/protos/issues/868). I091 continues independently on previously ratified PLAT056 carrier architecture.

**Approval gate:** perform the comparative/implementation-impact investigation, present the complete alternatives and exact proposed public contract to the owner, **ask for explicit approval**, then publish a separate clearly identified ratification delta **only if approved**. Do not update normative `spec/`, product code, conformance or I090 readiness based on this open proposal.

```text
D197_2026_10_09_RATIFICATION=RETAINED
D197_SMALLINTEGER_PROPOSAL=OPEN_NOT_APPROVED
D197_SMALLINTEGER_INVESTIGATION=PENDING
D197_SMALLINTEGER_AMENDMENT=NOT_RATIFIED
INTEGER_RECOGNITION_REDESIGN=OPEN
I090_DEPENDENT_IMPLEMENTATION=BLOCKED
PRODUCT_CODE_OR_TESTS_CHANGED=NO
```


## 2026-10-11 resolution of the historical open proposal

After the comparative and implementation-cost investigation, the owner expressly replied **“ok apruebo C”**. The historical `NOT_RATIFIED` and `INVESTIGATION=PENDING` flags reproduced above are obsolete for the hierarchy and common-recognition question. Candidate C is ratified; remaining new factory/API choices and executable conformance are not claimed approved or completed. Neither `guillermomolina/protos` nor its tests/specification were modified by this documentation-only publication.
