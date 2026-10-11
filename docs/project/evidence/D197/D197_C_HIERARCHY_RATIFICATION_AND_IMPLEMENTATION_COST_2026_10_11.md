# D197 — Candidate C: approval evidence, numeric-domain contract and implementation-cost audit

**Date:** 2026-10-11 (Europe/Madrid). **Status:** Candidate C owner-approved in the active conversation, **design only**. **Owner's exact instruction:** “ok apruebo C, Actualiza issues ...”.
**Formal owner:** [D197/#865](https://github.com/guillermomolina/protos/issues/865). **Implementation owners:** [I090/#868](https://github.com/guillermomolina/protos/issues/868) (public language semantics) and [I091/#869](https://github.com/guillermomolina/protos/issues/869) (physical primitive-first carriers and Java numeric dependency containment).
**Inspected product HEAD:** `guillermomolina/protos@1c414e08fefc2378b828bba7be7d263ea32e10ce`; **previous I091 token census:** `ced3746f9664ec5ceb562461c077bdf0627e5663`, with the current HEAD differing by one PERF042-related commit and no changes to the previously audited numeric carrier paths. Recheck the actual local HEAD and concurrent modifications when implementing.
**Selected record:** [D197 exact numeric tower](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md), 2026-10-11 amendment. **Other ratified constraints:** D196, PLAT056 Candidate C. This evidence is non-normative; product `spec/`, Core, runtime, tests, release and benchmark graph are NOT thereby changed or verified.

## 1. Approved candidate, exact GITHUB021 amendment

```text
Number
├── Integer              mathematical exact-integral protocol, not a concrete integer value family
│   ├── SmallInteger     canonical signed-64 [-2^63, 2^63-1]
│   └── BigInteger       canonical exact integer outside signed-64
├── Fraction             normalized, nonintegral exact rational
├── Float                IEEE binary64
└── Complex              D197 exact/inexact components and IEEE-zero exception
```

This **supersedes only** the original 2026-10-09 D197 clauses asserting that signed-64 values have concrete visible family `Integer` and that `BigInteger` is its sibling under `Number`. The rest of D197's signed-64 boundary, exact overflow/demotion, exact `/` normalization, Float contagion, Fraction/Complex, equality/identity/hash, actor isolation and D013 guarded dispatch requirements remain intact.

- `Integer.recognizes(small)` and `Integer.recognizes(large)` are true for genuine Core integer values; `Integer.recognizes(Fraction(2,5))`, `Integer.recognizes(Complex(0,1))`, and `Integer.recognizes(userObjectDelegatingToInteger)` are false. Membership uses trusted numerical family recognition, **not delegation-only ancestry** and not coercion.
- `SmallInteger.recognizes(x)` is true only for canonical signed-64 integer values; `BigInteger.recognizes(x)` only for canonical integer values outside signed-64. Preserve receiver and arity requirements; do not permit guest spoofing via prototype mutation.
- The observable immediate parent is `SmallInteger` for `42` and `BigInteger` for `2^100`; both concrete prototypes delegate to common `Integer`, whose parent is `Number`. This intentionally supersedes conformance such as `42.parent() === Integer` and `large.parent() === Integer`. `Integer` owns the common operations/recognition protocol.
- Preserve ordinary `Integer(value)` as the shared integer-domain conversion/normalization entry point where currently defined; a conversion's canonical result chooses SmallInteger or BigInteger. Detailed new public `SmallInteger(value)` and `BigInteger(value)` constructor admission/error policy, if exposed, was **not** approved independently; I090 must avoid inventing or silently shipping those semantics.
- `==` remains mathematical with compatible cross-family ordinary hash; `===` is non-overridable family-sensitive semantic identity. A conversion between physical Java carriers does not create a semantic type difference within a canonical guest family.
- No separate `Rational` common ancestor has been approved. `Fraction`, `Float` and `Complex` remain `Number` children under this amendment.
- `java.math.BigInteger` and the **guest prototype** `BigInteger` are unrelated categories. `long` primitive paths stay primitive where no observation requires guest materialization. C **does not** require per-value `ProtosSmallIntegerValue` allocation.

## 2. Alternatives and evidence

**A — original ratified D197 sibling model.** `Number → Integer` (signed-64) and `Number → BigInteger` (large), independent concrete siblings. Cheapest immediate small-parent compatibility, but no ordinary integer common prototype; shared domain acceptance is exceptional or independently invented. The original D197 remains historical approval, not current C.

**B — asymmetric inheritance.** `Number → Integer → BigInteger`, with signed-64 values concrete directly under `Integer`. Preserves `42.parent() === Integer` and adds common domain cheaply, but `Integer` mixes concrete signed-64 identity with mathematical common-domain responsibility. Its initial high implementation score overvalued backward compatibility to today's Java/runtime code.

**C — selected explicit concrete split.** `Number → Integer → (SmallInteger, BigInteger)` with common mathematical recognition and distinct canonical families. Requires migration of immediate-parent observability and guarded dispatch; no necessary additional small-number wrapper or Java BigInteger on the hot path. Selection does not prove compiled graph stability.

**Prior art / falsification rather than imitation:**
1. [Pharo Integer](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/Integer.class.st) is abstract with concrete SmallInteger and large integer variants; [Pharo SmallInteger](https://github.com/pharo-project/pharo/blob/Pharo15/src/Kernel/SmallInteger.class.st) uses architecture-dependent tagged width, **not Protos's fixed signed-64 boundary**.
2. [TruffleSqueak class lookup](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/accessing/SqueakObjectClassNode.java) associates Java primitive `long` with guest SmallInteger class. Its AST mechanisms are not transplanted literally into Protos Bytecode DSL.
3. [GraalPython implementation notes](https://github.com/oracle/graalpython/blob/master/docs/contributor/IMPLEMENTATION_DETAILS.md) describe primitive Java integer representations and the `PInt` object boundary while retaining the public Python `int` family; this is a credible one-public-family alternative, but not evidence that host BigInteger can never leak.
4. Ruby/TruffleRuby's ordinary public `Integer` domain with internal large-value representation illustrates a less-visible split, but Protos's previous one-domain implementation suffered cross-module host-number leakage. **That history is a reason to enforce representation boundaries, not a proof every one-family implementation inevitably leaks.**
5. [ECL/Common Lisp fixnum and bignum](https://ecl.common-lisp.dev/static/files/manual/ecl-26.5.5/Manipulating-Lisp-objects.html) distinguish small vs large integral representations beneath the integer domain; machine-dependent fixnum limits and value dispatch are not identical to C.
6. [GNU Emacs Lisp integer model](https://www.gnu.org/software/emacs/manual/html_node/elisp/Integer-Type.html) also separates fixnum/bignum with canonical normalization but machine-dependent small range; a counterexample to assuming all exact integers require big-number storage.
7. Java distinguishes primitive `long` from `java.math.BigInteger` as host types; it has no guest prototype delegation contract. This is a **transport/algorithm implementation comparison**, not an argument to reflect Java types into Protos.

## 3. GITHUB010 twelve-axis comparison (1–5, higher is better)

These are architectural judgments, **not measured benchmarks or implementation hours**. `H` = high confidence, `M` medium, `L` low.

| Criterion | A | B | C (approved) | Rationale for C and confidence |
| --- | ---: | ---: | ---: | --- |
| 1. Correctness/invariants | 3 | 4 | **5** | One exact-integral common protocol and explicit concrete families (**M**; runtime untested) |
| 2. Protos alignment | 3 | 4 | **4** | Prototype inheritance expresses a real domain; costs a new public entity (**M**) |
| 3. Pay for present need | 4 | **5** | 3 | Public SmallInteger is extra semantic surface, though no per-value allocation required (**H**) |
| 4. Incremental growth | 3 | 4 | **5** | Rich exact families extend under Number without changing the integer abstraction (**M**) |
| 5. Future-option resilience | 3 | 4 | **5** | Explicit subtype extension and stable integer-domain acceptance (**M**) |
| 6. Scalability | 4 | 4 | 4 | Large arithmetic and cross-domain copies still require bounded handling (**L**) |
| 7. Conceptual simplicity | 3 | 3 | **4** | Domain-vs-concrete roles symmetric rather than mixed (**M**) |
| 8. Portability | 4 | 4 | 4 | Signed-64 semantic threshold is fixed independent of tagged-object width (**H**) |
| 9. Runtime/resource cost | 4 | 4 | 4 | Primitive-first carriers equally possible in A/B/C; no gain assumed from C alone (**L**) |
| 10. Failure/operability | 3 | 3 | **4** | Distinct families make promotion/recognition bugs more diagnosable (**M**) |
| 11. Migration/reversibility | 4 | **5** | 3 | Parent/reflection tests and D013 guards must change; breaking semantic delta (**H**) |
| 12. Evidence maturity/risk | 4 | 3 | 4 | Pharo/TruffleSqueak precedent plus concrete Protos audit; DSL feasibility outstanding (**M**) |
| **Total, unweighted** | **42** | **47** | **49** | Ranking is narrow; authoritative owner decision is explicit, not arithmetic score |

**Anti-overengineering check:** C pays for one additional visible concrete family now; avoids physical wrapper proliferation; retains native signed-64 path; preserves a shared exact-integral domain. Deferring SmallInteger until after implementing B would force another observable `parent()`/guard/API migration and invalidate conformance twice. The required `Fraction`/`Complex` facilities are already independently approved in D197; adding `Rational` now fails the smallest-sufficient test. No implementation performance advantage is inferred merely from choosing C.

## 4. Actual HEAD implementation-cost and falsification map

| Production owner / directly grounded source | Concrete C delta and failure risk |
| --- | --- |
| `protos/lib/core/Integer.protos`, `prelude.protos`, `ProtosCoreBootstrap.java`, `ProtosPrelude.java` | Publish SmallInteger/BigInteger as children of abstract Integer, retain common methods; update Core binding checks and factory membership. |
| `ProtosIntegerValue.java`, `ProtosLargeIntegerValue.java`, `ProtosNumericValueSupport.java` | Small represented parent becomes SmallInteger; large frozen carrier becomes BigInteger; preserve signed-64 normalization and destination Prelude ownership. |
| `ProtosValueLookup.lookupGuardedInteger`, `ProtosStandardIntegerProtocol`, `ProtosBytecodeRootNode`, `ProtosSemanticBytecodeRootNode` | Guard currently requires parent exactly Integer. Update protected canonical lookup, inherited method home, override invalidation and deoptimization **in the same coherent implementation slice as prototype mutation**. A green semantic suite alone cannot prove no hotpath degradation. |
| `ProtosStandardNumericConversionProtocol`, `ProtosCurrentNumericRelations`, `ProtosIdentity`, `ProtosStandardHashSupport` | Common integer factory normalization, concrete recognizers, numeric family identity/hash, mathematical `==` across Number; no delegation spoofing. |
| `ProtosNioNetworkHost`, `ProtosNioNetworkBackend`, `ProtosNioTcpListenerBackend`, `ProtosEmbeddedNetworkCustody` | Currently propagate the Integer prototype to rematerialize unsigned 128-bit IPv6 bits. C changes the required owner to BigInteger **if this coupling is retained**. Prefer investigate moving guest-object materialization behind a narrow interop/value adapter instead of multiplying prototype-dependent backend interfaces. |
| `ProtosActorValueTransfer`, `ProtosSemanticTransferPayload`, `ProtosForeignPluginValueAdapter`, public SPI | Large-value transfer must preserve Core family/context, while signed-64 values must not be forced through host BigInteger; bounded ABI migration needed. |
| Conformance in `integer/prototype-and-receiver.protos`, `number/prototype-and-hash.protos`, `reflection/parent.protos`, `core-surface/removed-fixed-width-bindings.protos` | Change old parent/binding assertions intentionally. Maintain `Integer.recognizes` across the many libraries, including Math, datetime, SemVer, network, JSON/TOML. |

**Planning estimate, low confidence:** roughly **12–25 production Java files** for the C-specific hierarchy migration, several Core/source files, and corresponding test/conformance edits. This is not a prepared patch nor a proof of exact changed-file count. **Whole I090** additionally includes substantial Fraction/Complex/exact-division work and is not estimated by this range.

**Existing published I091 census at `ced3746f`:** production `BigInteger` tokens 447→73, `ProtosIntegerValue` 187→59, `ProtosFloatValue` 49→39, 683→171 total. These are token occurrences, NOT allocations. [Complete source census](../I091/I091_F5_COMPLETE_NUMERIC_DEPENDENCY_RECONCILIATION_2026_10_11.md); [adversarial per-token counterexamples](../I091/I091_F5_ADVERSARIAL_BIG_INTEGER_CASE_BY_CASE_AUDIT_2026_10_11.md). Remaining **unjustified small-integral Java BigInteger** interfaces include `ProtosSemanticTransferPayload` and the inbound/outbound foreign-provider SPI/Polyglot adapter. Legitimate exceptional uses include truly-large algorithms, typed Truffle `asBigInteger` requests and explicit host Java formal parameters.

**Network coupling observation, not an approved new design:** `std:network/IpAddresses.protos` constructs IPv6 addresses with native Protos arithmetic; nevertheless `ProtosNioNetworkBackend` and `ProtosNioTcpListenerBackend` also construct guest `IpAddress`/`IpEndpoint` objects, fill slots, and carry a numeric prototype for 128-bit IP bits. This duplicates semantic knowledge across Java and Protos. A candidate boundary is (a) pure Protos address/endpoint protocol, (b) single conversion adapter, (c) Java NIO backend with host `InetAddress`, `InetSocketAddress`, byte arrays/channels and no Core numeric-prototype dependency. Preserve actual network authority, IPv6 scope, Actor cancellation, snapshots and isolation before selecting or implementing that separate refactor.

## 5. Execution boundaries and acceptance

**Ordering:** D197 C ratification **now** → coordinate I090 public model with I091 physical carriers **without waiting for I091 closure** → human-executed source/tests/commit → PERF040 pinned structural compiled graph/timing. Independent small-integer host transport cleanup in I091 can progress in parallel. Do not build or publish a temporary contradictory model or remove required exact arithmetic.

**Explicit falsifiers:** `42.parent()===SmallInteger`, `(2^100).parent()===BigInteger`, both subclasses' `parent()===Integer`, common and concrete `recognizes` behavior, rejecting forged ordinary children, signed-64 overflow and demotion, exact hash/identity under family change, overridden numeric send and guard invalidation, small `return 1` and `1+2` no mandatory host BigInteger/wrapper path, IPv6 128-bit round trip across Context and Actor/P, actual signed-64 SPI in/out without BigInteger, and compatibility of existing explicit `asBigInteger` host ABI. No test or graph evidence was produced by this research record.

**Explicit exclusions:** no Protos product source/spec changes, no new `Rational`, no full implementation factory policy beyond the common-domain `Integer(value)` requirement, no mandate to delete legitimate Java `BigInteger`, no network architecture amendment, no claim I091/PERF040 finished, no invented test PASS.

```text
D197_C_HIERARCHY=OWNER_RATIFIED_2026_10_11
D197_PREVIOUS_2026_10_09=HISTORICAL_UNALTERED_EXCEPT_EXPLICIT_C_DELTA
PRODUCT_HEAD_INSPECTED=1c414e08fefc2378b828bba7be7d263ea32e10ce
PROTOS_IMPL_CHANGED_BY_THIS_RECORD=NO
PROTOS_TESTS_EXECUTED=NO
PERF_GRAPH_PROVEN=NO
I090_PUBLIC_HIERARCHY_GATE=RESOLVED_BY_OWNER_APPROVAL
I091_PHYSICAL_AND_SPI_REMAIN_OPEN=YES
NETWORK_INTEROP_REFACTOR=OBSERVATION_ONLY
```
