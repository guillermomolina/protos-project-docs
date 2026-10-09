# D196 — Integer/Float mixed arithmetic: binary64 operand-first promotion

**Decision:** RATIFIED — B1 (operand-first Float promotion)
**Owner approval:** 2026-10-09, explicit in the active owner conversation: “apruebo D196 ahora” immediately after the exact B1 recommendation and clarification of D197 boundaries.
**Live decision:** [D196 / guillermomolina/protos#864](https://github.com/guillermomolina/protos/issues/864)
**Historical amended decision:** [D006 / #164](https://github.com/guillermomolina/protos/issues/164)
**Normative owner:** `spec/semantics/VALUES_AND_COLLECTIONS.md` in `guillermomolina/protos`
**Product revision inspected at ratification:** `0db24f00ff2d92d642351d7f7535517fe01a55ce`
**Project-record parent revision before publication:** `c30926b5b16adacf6ab1233b8a53007f47599124`

> This is a **non-normative decision and investigation-evidence record**, not an assertion that the current product specification or runtime has already been updated. The approved semantics must be published in the normative specification, reflected in its global changelog, and implemented and tested through the human-executor workflow. The product revision above still contains the prior mixed-arithmetic Error rule.

## Selected contract

```text
SELECTED_CANDIDATE=B1_OPERAND_FIRST_BINARY64_PROMOTION
AFFECTED_OPERATORS=+,-,*,/
AFFECTED_OPERAND_FAMILIES=Integer/Float,Float/Integer
RESULT_FAMILY=Float
INTEGER_CONVERSION=Float(Integer), roundTiesToEven, as already specified
CONVERSION_TIME=BEFORE_ARITHMETIC_OPERATION
OPERATION=STANDARD_IEEE754_BINARY64_OPERATOR_ON_CONVERTED_OPERANDS
OPERATION_ROUNDING=IEEE754_BINARY64_PER_OPERATION
MIXED_ORDER=KEEP_ORIGINAL_LEFT_AND_RIGHT_OPERANDS
MIXED_ARITHMETIC_FAILURE_FOR_FAMILY_MISMATCH=REPLACED_BY_FLOAT_RESULT
```

For a standard arithmetic message whose receiver and argument belong to different families, one Integer and one Float, convert the Integer operand to Float using the existing normative `Float(Integer)` operation, **then** evaluate the requested arithmetic operator with the original left/right ordering under IEEE 754 binary64. Thus, `Integer + Float` and `Float + Integer` both return Float; subtraction and division preserve operand order. The standard method must not first perform exact rational/integer arithmetic and only then round the final result.

The conversion retains the already-approved explicit `Float(Integer)` semantics: arbitrary-precision Integer converts to binary64, roundTiesToEven, with values beyond finite range yielding signed infinity. A zero Integer converts to positive zero. The binary64 operation then follows ordinary IEEE semantics for infinities, NaN, signed zero, underflow, overflow and zero division, with no extra special-case integer/Float behavior. No mandatory Java class, VM encoding or specialization strategy is specified here.

The selected semantics apply to the **standard numeric methods**. Arithmetic remains ordinary overridable Protos message dispatch (D013); no universal syntax bypass or unconditional intrinsic is authorized. Preserve left-to-right evaluation, lookup, method overrides, standard error propagation and guarded-fallback equivalence.

## Explicitly preserved invariants and non-decisions

- `Integer + Integer`, `Integer - Integer`, `Integer * Integer` continue to return mathematically **exact, unbounded Integer** results. Machine `long` overflow never promotes to Float.
- `Float + Float`, `Float - Float`, `Float * Float`, `Float / Float` preserve their existing binary64 contract.
- **`Integer / Integer` remains unchanged by D196**: at the ratification product revision it returns one correctly rounded binary64 result from the *exact rational quotient*, even for an integral quotient. Future exact `Fraction` or integral quotient behavior must be approved and amended through **D197 / #865**, not smuggled into this decision.
- Integer `div`, `mod` and `%` remain Integer-only and keep their current truncating/remainder semantics.
- Explicit `Integer(Float)` and `Float(Integer)` retain their existing contracts.
- Numeric `==` and ordering remain exact mathematical cross-family comparisons; **do not** coerce Integer to Float for comparison. `===`, numeric `hash`, numeric identities, signed-zero distinctions and the semantic NaN model remain as ratified.
- No Float promotion is implied for arbitrary nonnumeric operands or custom user methods.
- No `Fraction` or `Complex` family, exact division, negative-real square-root domain extension, square-root API or Fraction/Float coercion protocol is approved by D196. D197 remains an independent design decision. Owner has expressed intent to approve Fraction and Complex, but their *precise* semantics remain subject to D197.
- No choice of VM representation, `ProtosIntegerValue`/`ProtosFloatValue` boxing, numeric carrier, Java `BigInteger` containment, backend, or performance strategy is approved here. **PLAT056 / #866** owns that design; **PERF040 / #862** consumes the eventual approved semantics and platform.
- D006/#164 remains an intact historical decision; D196 **prospectively amends only the previous standard mixed Integer/Float arithmetic Error rule**.

## Distinguishing numerical evidence and conformance seeds

The candidates are not equivalent. Let `N` represent an exact unbounded Integer and `F` a binary64 Float; these equations are mathematical test descriptions, not necessarily literal Protos source syntax.

| Operation | Selected B1: convert Integer first | Rejected B2: exact result then round |
| --- | --- | --- |
| `(2^53+1) + (-2^53.0)` | `+0.0` | `1.0` |
| `(2^63-1) + (-2^63.0)` | `+0.0` | `-1.0` |
| `2^1024 * 0.0` | `NaN` (infinity times zero) | `+0.0` |
| `0.5 / 2^1024` (Float/Integer) | `+0.0` | `2^-1025` (subnormal) |
| `2^1024 + (-Infinity)` | `NaN` | `-Infinity` |

Test both operand orders for each of `+`, `-`, `*`, `/`; ensure order-sensitive operations are not commuted. Cover small/large Integers, ±2^53 boundaries, ±2^63 boundaries, ±2^1024 magnitudes, halfway roundTiesToEven, signed zero, Float NaN and infinities, mixed zero divisors and underflow. In addition, pin existing Integer/Integer division, integer overflow, exact cross-family equality/ordering and numeric hashing as negative regression controls. Verify ordinary overrides, dynamic lookup and canonical guarded fast-send fallback.

## Comparative research digest

The D196-A comparative investigation reviewed multiple coherent policies rather than inferring the language rule from existing Java wrappers:

- **TruffleSqueak / Squeak**: primitive `long + long` with exact overflow fallback and guarded `long + double` / `double + long` paths; the image-level coercion policy and `numericPrimsMixArithmetic()` guard must not be mistaken for unconditional VM-wide language semantics. [TruffleSqueak implementation](https://github.com/hpi-swa/trufflesqueak).
- **Pharo Smalltalk**: mixed Number protocols implement double-dispatch adaptation, including `adaptToFloat:andSend:` and `adaptToInteger:andSend:`, evidencing mixed Float contagion without defining Protos' dispatch. [Pharo sources](https://github.com/pharo-project/pharo).
- **Common Lisp**: numeric contagion and mixed rational/floating calculations provide an extensible exact/inexact precedent. [Common Lisp HyperSpec](https://www.lispworks.com/documentation/HyperSpec/Front/index.htm).
- **Racket**: exact/inexact numeric distinction with exact rationals and complex values illustrates alternative exactness policies. [Racket numbers](https://docs.racket-lang.org/reference/numbers.html).
- **Python**: `int`/`float` mixed arithmetic typically promotes to floating-point but has implementation-specific large-integer conversion failures; Protos does **not** import that overflow exception. [Python numeric types](https://docs.python.org/3/library/stdtypes.html#numeric-types-int-float-complex).
- **Ruby**: Integer/Float coercion and standard arithmetic provide a further dynamic-language precedent. [Ruby Numeric](https://docs.ruby-lang.org/en/master/Numeric.html).
- **JavaScript**: Number versus BigInt mixed arithmetic rejects mixed numeric operators, supporting the explicit-rejection alternative rather than B1. [ECMAScript specification](https://tc39.es/ecma262/).

Candidates explored: **A** retain mixed-Error; **B1** operand-first Float coercion; **B2** exact mixed arithmetic and one final rounding; **C** require explicit conversion. B1 was preferred because it is a single ordinary mixed-arithmetic rule, reuses existing `Float(Integer)`, follows IEEE binary64 for actual operations, avoids an extra exact mixed-operation subsystem, and remains extendable when D197 separately specifies additional exact families. B2 can preserve information in particular cases but gives `Integer + Float` behavior different from the already-standard Float conversion followed by Float arithmetic; the `2^1024 * 0.0` and `0.5 / 2^1024` cases show that the difference is not merely last-bit rounding.

### GITHUB010 evaluation rationale at ratification

The completed D196-A investigation included a twelve-axis candidate comparison. This archival digest records the qualitative decision grounds, **not a fabricated reproduction of original per-axis numerical scores**:

| Mandatory dimension | Reason B1 was selected over alternatives |
| --- | --- |
| Correctness / invariants | Explicitly amends mixed Error only; keeps exact unbounded Integer and comparison invariants |
| Protos alignment | One ordinary standard numeric rule; no new privileged numeric entity |
| Present-need proportionality | Reuses Float conversion and Float arithmetic instead of implementing full rational mixed operators |
| Incremental growth | Allows Fraction/Complex to be evaluated and added later under D197 |
| Future-option resilience | Does not require a fixed host carrier or full numeric tower now |
| Scalability | Keeps large Integer exact while avoiding exact-rational intermediate machinery for mixed Float use |
| Conceptual simplicity | Operand-first promotion is explainable and compositional |
| Portability / implementation freedom | Defined with IEEE binary64 semantics, not Java class layout |
| Runtime / resource cost | No always-on Fraction/Complex work or arbitrary-precision mixed operator needed |
| Failure / operability | IEEE exceptional behavior is explicit, including infinity/NaN and signed zero |
| Deferral / reversibility / migration | The precise breaking change is mixed Error -> Float; D197 can still amend Integer division independently |
| Evidence maturity / implementation risk | Supported by several mature language precedents and existing Protos explicit conversion behavior |

The strongest rejected-case objection is lost Integer precision when mixed with Float, particularly beyond `2^53`. This cost is **intentional**: inserting a Float selects approximate binary64 arithmetic. The user can retain exactness by using only exact operations, with future rational capability evaluated separately.

## GITHUB021 invariant/delta consistency and scope

```text
OWNER_APPROVAL=D196_B1_EXACT_CANDIDATE_2026-10-09
DECISION_APPROVAL_PROVENANCE=PASS
D006_AMENDED_SURFACE=STANDARD_MIXED_INTEGER_FLOAT_ARITHMETIC_ONLY
D006_INTEGER_EXACTNESS=UNCHANGED
D006_INTEGER_INTEGER_DIVISION=UNCHANGED_PENDING_D197
D006_NUMERIC_EQ_ORDER_HASH_IDENTITY=UNCHANGED
D013_ORDINARY_DISPATCH=UNCHANGED
PLAT056_REPRESENTATION_SELECTION=NONE
D197_FRACTION_COMPLEX_APPROVAL=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
```

This is a **bounded prospective override** of the D006 mixed-arithmetic rejection, expressly authorized by the owner. It is not a wholesale reopening of D006. No unresolved D197 or PLAT056 invariant is silently selected.

## Execution handoff and publication status

**Implementation owner: [I089 / guillermomolina/protos#867](https://github.com/guillermomolina/protos/issues/867).** D196 itself is the ratified **design** and does not own executable patches or test gates. The following implementation work remains under I089 in `guillermomolina/protos`:

1. Update the normative mixed-arithmetic table and exact rounding contract in `spec/semantics/VALUES_AND_COLLECTIONS.md` and the global `spec/PROTOS_SPEC_CHANGELOG.md` at the then-current global revision.
2. Implement the standard mixed operators for both receiver families without bypassing ordinary messages or altering unrelated numeric behavior; add focused regression and guard-path tests.
3. Use human-executor commands for validations and product Git publication, then record actual green results and the resulting exact product commit.
4. Maintain D197, PLAT056 and PERF040 as separate coordination surfaces; do not prematurely release PERF040's PLAT056 architecture blocker.

**At this record's initial publication:** owner selection is approved and durably documented; **normative specification update, runtime implementation, tests and product commit are pending**. The user's statement that existing local tests passed and `git diff --check` was clean precedes this normative/runtime change and is not evidence that B1 has been implemented.

## AI assistance

Prepared with AI assistance from the D196-A research and direct source/issue review; exact candidate approved by the project owner. No independently conducted human audit of this document or new implementation tests is claimed.
