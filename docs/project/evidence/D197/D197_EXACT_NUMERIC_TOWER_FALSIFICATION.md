# D197 — Comparative evidence and attempted falsification of the exact numeric tower

**Evidence class:** read-only design/source investigation; no product code change or execution.
**Owner selection:** D197 approved in active conversation on 2026-10-09; authoritative proposed future semantics are recorded in [D197 decision](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md).
**Product baseline reviewed:** `guillermomolina/protos@0db24f00ff2d92d642351d7f7535517fe01a55ce`.
**Issues:** [D197/#865](https://github.com/guillermomolina/protos/issues/865), [D006/#164](https://github.com/guillermomolina/protos/issues/164), [D156/#616](https://github.com/guillermomolina/protos/issues/616), [D196/#864](https://github.com/guillermomolina/protos/issues/864), [PLAT056/#866](https://github.com/guillermomolina/protos/issues/866), [I089/#867](https://github.com/guillermomolina/protos/issues/867).

## Comparison grounded in reference implementations and language definitions

| Reference | Observed semantic design | Relevance and limits for Protos |
| --- | --- | --- |
| [TruffleSqueak `ArithmeticPrimitives.java`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/primitives/impl/ArithmeticPrimitives.java) | `Math.addExact(long,long)` / subtract/multiply specializations; overflow switches to large integer helper; exact integral small `/` returns long, otherwise image-level Fraction | Concrete cross-Truffle precedent for guest-visible numeric families with primitive fast-path; do not copy Squeak VM representation as normative Protos semantics |
| [TruffleSqueak `LargeIntegers.java`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/plugins/LargeIntegers.java) | Java `BigInteger` in slow helpers; `normalize(BigInteger)` returns Java `long` when its bit length fits, otherwise image `NativeObject` | Strong evidence for automatic large→small demotion, no mandatory generic host BigInteger in every small operation |
| [TruffleSqueak `SqueakObjectClassNode.java`](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/accessing/SqueakObjectClassNode.java) | A primitive `long` is reflected as the image SmallInteger class | Numeric family/class observable in guest despite primitive Java carrier |
| [Pharo `SmallInteger.class.st`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/SmallInteger.class.st) | 31- or 63-bit SmallInteger depending on image architecture; fast primitives and fallback when output is not SmallInteger | Shows size bound need not equal Java `long`, and exposes host-dependent class boundary |
| [Pharo `LargePositiveInteger.class.st`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/LargePositiveInteger.class.st) | `normalize` shrinks large integer to SmallInteger if range permits | Direct evidence of canonical demotion |
| [Pharo `Integer.class.st`](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/Integer.class.st) | exact integer operations normalized, exact `/` produces integral result or reduced Fraction | Strong precedent for exact division without compulsory Fraction allocation for integral quotients |
| [Common Lisp numeric tower](https://www.lispworks.com/documentation/HyperSpec/Body/t_integer.htm) | `integer` includes `fixnum` and `bignum` in common integer domain; fixnum range implementation-dependent | Confirms distinction between mathematical integer acceptance and concrete magnitude category; portable fixed 64-bit bound is Protos policy, not copied from CL |
| [Racket number model](https://docs.racket-lang.org/reference/numbers.html) | exact/inexact arithmetic, exact rational and complex, inexact transcendental results | Demonstrates impossibility of promising all math functions exactly representable in finite type tower |
| [Python fractions](https://docs.python.org/3/library/fractions.html) and [cmath](https://docs.python.org/3/library/cmath.html) | explicit Fraction and complex branch cuts with signed zero | Contrasting opt-in arithmetic policy and proof of signed-imaginary-zero branch significance |
| [C++ complex sqrt reference](https://en.cppreference.com/w/cpp/numeric/complex/sqrt.html) | principal complex square root and branch cut sensitive to sign of zero | Strong counterexample to unconditional Complex→real demotion |
| [Julia numbers](https://docs.julialang.org/en/v1/manual/complex-and-rational-numbers/) | rational `//` distinct from `/`; Complex can hold parametric component numeric types | Shows alternate operator policy and support for non-double complex components |
| [Clojure numerical operators](https://clojure.org/reference/data_structures#_numbers) | checked primitive operations and promoting variants; BigInt and Ratio | Other dynamic numeric-tower trade-off; Protos selects universal exact arithmetic and normalization instead of operator-specific promotion |
| [Ruby Rational](https://docs.ruby-lang.org/en/master/Rational.html) | Rational is a numeric family with exact numerator and denominator | Contrasts with canonical demotion of rationals with integer quotient |

## Adversarial cases and expected decision implications

| Case / attempted counterexample | Falsified naïve rule | Selected design constraint / open detail |
| --- | --- | --- |
| `2/1`; `2/4`; `(1/2)+(1/2)` | “every integer division returns Fraction” | exact quotient normalizes to Integer/BigInteger when integral; otherwise reduced Fraction |
| `9223372036854775807+1`; subtract 1 | “large numeric type stays large forever” | first BigInteger; second Integer, fixed portable signed-64 boundary |
| `(-9223372036854775808)/(-1)` | “division of long inputs always yields long” | BigInteger `2^63`, not wrap or Float |
| `sqrt(Complex(-4.0,+0.0))` vs `sqrt(Complex(-4.0,-0.0))` | “Complex imaginary zero always reduces to real” | preserve sign of inexact IEEE zero and branch-side information |
| `sqrt(4)`; `sqrt(1/4)`; `sqrt(-4)` | “sqrt is always approximate” | exact results preserved when expressible |
| `sqrt(2)`; `sqrt(-2)`; `sin(1)` | “every mathematical result stays exact within five families” | permit specified approximation for irrational results |
| `(10^400+1)/(10^400)` then `.asFloat` | “convert numerator and denominator to double then divide” | must round exact rational quotient to binary64 directly |
| exact `1/10` vs Float `0.1`; exact `1/2` vs Float `0.5` | “printed decimal determines equality” | compare mathematically without Float coercion; first unequal, second equal |
| `Map[Fraction(1,2)]` vs `Map[Float(0.5)]` | “hash by family” | equal numeric values must share normal Map hash class; identity map remains family-sensitive |
| `Integer.recognizes(2^100)` under old programs | “changing Integer range only affects math operators” | existing library gcd/factorial, Array indices and other clients require an explicit common-integer-domain contract; exact selector behavior left unratified |
| exact Fraction series with many coprime denominators | “automatic normalization is always cheap” | GCD and large values may consume extensive resources; optimize lazily when possible without changing semantics |
| `BigInteger + Float` where big exact value rounds to `Infinity` | “compute mathematically exact mixed result then round once” | owner-approved D196 operand-first binary64 rule extended; `Infinity*0` can yield NaN |
| `Complex(Fraction, BigInteger)` | “Complex always stores `double,double`” | components are Protos numeric values; no forced lossy conversion |

## Three materially different candidate strategies

- **N1 — unconditional numeric demotion**: direct owner motivational rule without a distinction for inexact complex zero. REJECTED: destroys IEEE branch information.
- **N2 — result-canonical exact tower with domain-sensitive IEEE preservation**: chosen design target, owner-approved subject to precise subordinate protocols. Keeps common math simplification while avoiding lossy complex-zero normalization.
- **N3 — type-preserving wrappers / opt-in Fraction and Complex**: mature alternative in Ruby, Python, Julia but conflicts with the owner's approval of automatic exact division and demotion.

## GITHUB010 twelve-dimension comparison, scores 1–5

Scoring is qualitative research judgment with **MEDIUM** confidence overall (HIGH for mathematical/IEEE falsifiers, LOW for compiled performance in absence of measurements). It is not benchmark output.

| Dimension | N1 | N2 | N3 | Rationale for N2 |
| --- | ---: | ---: | ---: | --- |
| Correctness / invariant preservation | 1 | 4 | 2 | Guards branch cuts and precision while keeping exact canonicality |
| Protos alignment | 3 | 5 | 3 | Expresses cohesive ordinary numeric semantics without opt-in rationals |
| Present-need proportionality / pay only for need | 4 | 4 | 3 | Costly exact values appear on demand rather than on all small ops |
| Incremental growth | 2 | 5 | 4 | Allows later numeric protocols without undoing core canonical model |
| Future-option resilience | 2 | 5 | 4 | No forced `double,double` Complex or host carrier |
| Scalability | 3 | 4 | 3 | Fast primitive paths permitted but large math complexity persists |
| Conceptual simplicity | 4 | 4 | 3 | Exact/inexact domain exception is principled, not ad hoc |
| Portability / implementation freedom | 3 | 4 | 4 | Portable signed-64 semantic boundary independent of carrier |
| Runtime / resource cost | 4 | 4 | 3 | Can avoid unnecessary allocation; still needs empirical confirmation |
| Failure / operability | 2 | 4 | 4 | Domain and Error/IEEE exceptional paths explicit |
| Cost of deferral / reversibility / migration | 2 | 4 | 3 | Pays immediate D006 compatibility migration instead of later rewrite |
| Evidence maturity / implementation risk | 2 | 4 | 5 | Smalltalk, Lisp, Racket prove model components, integration still untested |

**Noncompensating gates:** N1 is disqualified by information loss regardless of total score. N3 is semantically viable but undercuts owner-defined automatic exactness/demotion. N2 carries a real overengineering risk if all five families are implemented as ambient boxed VM-wide objects; PLAT056 must prevent such unconditional tax. N2 also has a real underengineering risk if the first runtime slice neglects the common exact-integer protocol and breaks ordinary library clients.

## GITHUB021 approved invariant versus changed historical invariant

| Prior source decision | Earlier rule | Owner-approved D197 replacement or retained rule |
| --- | --- | --- |
| D006 numeric families | Integer unlimited; Float only | signed-64 Integer; BigInteger; Fraction; Complex alongside Float |
| D006 integer integer division | binary64 Float | exact Integer/BigInteger when integral, reduced Fraction otherwise |
| D156 fixed-width removal | remove eight fixed-width arithmetic families | RETAIN; magnitude split Integer/BigInteger is not reintroducing UInt8…Int64 |
| D196 mixed real arithmetic | Integer→Float before mixed operator | RETAIN B1 order; extend to BigInteger/Fraction inputs as approved |
| PLAT056 old input | BigInteger host helper only, not Protos type | overridden: Protos BigInteger is semantic; Java BigInteger remains optional host helper |
| Current numeric identity/hash | immutable values, family-sensitive `===`, cross-family exact `==` | extend to new semantic families, preserve Float NaN and signed-zero exceptions |
| Truffle fast path | long/double may be used | RETAIN goal; no approved exact carrier design yet |

## Conformance seeds (NOT executed)

1. Exact integer boundary additions, subtraction demotion, multiplication overflow and `Long.MIN_VALUE/-1` division.
2. `Integer/Integer`, `BigInteger/Integer`, `Integer/BigInteger`, `BigInteger/BigInteger` divisions with integral/nonintegral/negative/zero quotients, canonical sign and gcd.
3. Fraction arithmetic closure with integer demotion; numerator and denominator very large, cancellation and rational-to-binary64 one-shot rounding, ties-to-even, subnormal, overflow.
4. `Fraction+Float`, `Float/Fraction`, `BigInteger+Float`, IEEE infinities, NaNs and signed zeros; operand-first B1 arithmetic, but exact numeric equality.
5. Complex exact mixed components, exact-zero reduction, inexact signed-imaginary-zero non-erasure; exact/approximate sqrt including negative real cases and branch sides.
6. Cross-family `==` / `===` / `hash`, Map/IdentityMap, receiver overrides, guarded fallback, `recognizes` after separately settling semantic-domain protocol.
7. Numeric families in Arrays, strings, byte/index bounds, modules, Actor/P transfer, serialization and foreign/Java interop, including failure on unsupported lossy conversions.
8. Primitive optimized hot path checks against materialization at boundaries; compare TruffleSqueak's long strategy while preserving Protos observable behavior.

## Sources inspected and validation boundaries

Protos: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `spec/AGENTS.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, `spec/semantics/OBJECT_MODEL.md`, `spec/semantics/ERRORS.md`, `spec/concurrency/ACTORS.md`, `protos/lib/math/Integer.protos`, `protos/lib/core/Number.protos`, `protos/lib/core/Integer.protos`, plus live D006, D156, D196, D197, PLAT056 and I089 issue bodies. External comparisons use the source links above. Not all `src/main` call sites, every protocol, or every cross-Truffle runtime were fully read during this **D197 language-semantic** checkpoint: the exhaustive implementation/architecture inventory remains PLAT056 work.

**No runtime tests, builds, benchmarks, validation scripts or product Git writes were run as part of this investigation.** The D197 owner approval reflects the decisions above, not an empirical claim about implementation cost.
