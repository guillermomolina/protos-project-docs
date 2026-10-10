# PLAT056 — Candidate C comparison, D197 reconciliation and ratification evidence

**Evidence class:** design-source comparison, attempted falsification and owner approval; not an implementation/benchmark acceptance.
**Approval event:** 2026-10-10 — owner said “ok aprobada la recomendacion” after review of the D197-aware Candidate C and its intended containment of Java `BigInteger`.
**Live issue:** [PLAT056/#866](https://github.com/guillermomolina/protos/issues/866).
**Canonical platform decision:** [PLAT056 Candidate C](../../decisions/platform/PLAT056_PRIMITIVE_FIRST_NUMERIC_GUEST_OBJECT_ARCHITECTURE.md).
**Ratified semantic prerequisite:** [D197 N2](../../decisions/language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md) and [D197 falsification](../D197/D197_EXACT_NUMERIC_TOWER_FALSIFICATION.md).
**Source baseline:** `guillermomolina/protos@0db24f00ff2d92d642351d7f7535517fe01a55ce` inspected; `guillermomolina/protos-project-docs@5e0655d696c5323096f3faef4311a6a0b4640404` was the durable D197 baseline before this decision publication.
**Earlier PLAT056 checkpoint:** [numeric guest-value baseline](PLAT056_NUMERIC_GUEST_VALUE_BASELINE_AND_APPROVAL_GATE.md).

## Selection and actual limits of the approval

The active proposal was: retain small `Integer` in primitive `long` and `Float` in primitive `double`; represent canonical Protos `BigInteger`, `Fraction`, `Complex` using ordinary Protos guest values and immutable numeric content; use Java `BigInteger` only inside contained algorithms and explicit Java boundaries; avoid VM-wide host-class leakage; normalize exact results to the D197 family; retain D013 guarded ordinary messages, semantic equality, identity, hash, reflection and transfer; impose no always-on big/rational/complex tax on `1+2`.

The owner approved that recommendation. Approval **does not mean** Candidate C has already passed feasibility, every source dependency has been counted, or runtime conformance has been demonstrated. Candidate B remains a credible contingency, **not an automatic authorized replacement**.

D197 approval is not the same decision: D197 defines visible `Integer` (portable signed-64), visible `BigInteger` (outside the range), exact reduced `Fraction`, binary64 `Float`, and compositional `Complex`. PLAT056 chooses a runtime architecture that implements them, and is explicitly forbidden from reinstating the historical single unbounded *visible* Integer merely because that would simplify host Java storage.

## Current Protos coupling: source-backed illustration, not complete census

The product source at the pinned HEAD has cross-layer dependencies:

| Source / boundary | Observed design debt / retained justification |
| --- | --- |
| [`ProtosIntegerValue.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosIntegerValue.java) | Stores `long smallValue` plus nullable Java `BigInteger bigValue`; no-overflow operations can still return new wrapper objects; large-exact helper is legitimate |
| [`ProtosFloatValue.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosFloatValue.java) | Always presents the binary64 semantic value via wrapper; ordinary arithmetic should not force it |
| [`ProtosNumberLiteral.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosNumberLiteral.java) | Initial numeric lowering parses through Java BigInteger, but this is NOT proven to be repeated per invocation |
| [`ProtosStandardNumericConversionProtocol.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumericConversionProtocol.java) | Integer→Float passes through an exact generic host BigInteger API; preserve rounding but permit primitive fast path |
| [`ProtosValueLookup.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java) | Integer representation tested via Java `instanceof ProtosIntegerValue`; future recognition must refer to ratified semantic type/domain |
| [`ProtosIdentity.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosIdentity.java) | Numeric `===`/identity-hash implementations explicitly recognize wrappers and use BigInteger hash results; must preserve value semantics through C |
| [`ProtosSealedValue.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosSealedValue.java) | Already supports frozen ordinary Protos objects with family-private state, but retained ordinary object identity and rejected isolated transfers are **not** sufficient to fulfill numeric value semantics |
| [`ProtosSemanticBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) and [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java) | `boxingEliminationTypes={int.class}` in both roots; alone insufficient to provide long/double at operations, calls or locals |
| Collections, Map/IdentityMap, I/O, Actor/P transfer, interop, debugger | Current host `BigInteger` and numeric-wrapper use must be classified as required algorithm, explicit Java boundary, or accidental guest representation coupling; exact inventory is outstanding |

Previous search reported **at least 60 production files with `BigInteger` matches** and many wrapper matches, but the search was indexed/result-limited and is **not** an exhaustive audited file total. Some numeric hash/index values are intentionally unbounded exact results; replacing their public contracts with `long` would be an incorrect simplification. The acceptance goal is **removal of unjustified generic VM dependence** on Java `BigInteger`, not an unsupported numerical target.

## External implementation evidence

| Primary reference | Verified approach | What transfers / does not transfer |
| --- | --- | --- |
| [TruffleSqueak arithmetic](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/primitives/impl/ArithmeticPrimitives.java) | `Math.addExact(long,long)`, exact overflow fallback; exact integral `long/long` returns long, otherwise image-level Fraction | Direct evidence for primitive hot path and ordinary-language Fraction; do not equate image `NativeObject` with an unspecialized arbitrary Protos object |
| [TruffleSqueak LargeIntegers](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/nodes/plugins/LargeIntegers.java), [NativeObject](https://github.com/hpi-swa/trufflesqueak/blob/77bbbec529e9869c3b94173235294032576547f8/src/de.hpi.swa.trufflesqueak/src/de/hpi/swa/trufflesqueak/model/NativeObject.java) | Java BigInteger remains in local algorithms, large integers become language/image objects, results demote to long when they fit | Strong B/C hybrid precedent; large storage is still a specialized Java-backed guest native object |
| [TruffleRuby InlinedAddNode](https://github.com/truffleruby/truffleruby/blob/c734f26543003fefd4519a29adbca62d0c711a2d/src/main/java/org/truffleruby/core/inlined/InlinedAddNode.java) | `int`/`long`/`double` specializations, exact overflow to Bignum, standard method assumptions with `rewriteAndCall` fallback | Strong D013/PLAT040 dispatch analogue; does not establish generic Protos rich-number representation |
| [GraalPy PInt](https://github.com/oracle/graalpython/blob/63022137a797dc10251acac5460bb2e5b0a113cb/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/ints/PInt.java) | `PInt` Python guest object owns private Java BigInteger payload and interoperates with primitive small integers | Strong technical support for Candidate B; Python semantics cannot define D197 canonical families |
| [Enso AddNode](https://github.com/enso-org/enso/blob/4135f929f1046ee4ab62d9d09c155377e8542a1f/engine/runtime/src/main/java/org/enso/interpreter/node/expression/builtin/number/integer/AddNode.java), [EnsoBigInteger](https://github.com/enso-org/enso/blob/4135f929f1046ee4ab62d9d09c155377e8542a1f/engine/runtime/src/main/java/org/enso/interpreter/runtime/number/EnsoBigInteger.java) | Primitive exact add plus compact large fallback | Candidate B counterweight: explicit runtime carrier may be simpler/faster than C |
| [SimpleLanguage Bytecode root](https://github.com/oracle/graal/blob/87dc52ba6fd900099533eb37814e77df0c53e89a/truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java) | Boxing elimination explicitly includes `long`, `boolean` | Demonstrates required DSL machinery can be configured; does not prove Protos end-to-end call-path compliance |
| [GraalJS JSAddNode](https://github.com/oracle/graaljs/blob/4c9cd0a6d1d5270b6cd039817851756b0b01b254/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/binary/JSAddNode.java) | Fast primitive arithmetic/specialization, different Number/BigInt semantics | Reuse structural techniques, **not** guest type or mixed numeric policy |
| [Pkl VmTypes](https://github.com/apple/pkl/blob/88a50d167e61b37ebff909c41dd04ef082a9e5fa/pkl-core/src/main/java/org/pkl/core/runtime/VmTypes.java) | Common direct `long`/`double` representation | Proves primitive-first option but bounded integers are not a model for D197 exact overflow |
| [Espresso StaticObject](https://github.com/oracle/graal/blob/87dc52ba6fd900099533eb37814e77df0c53e89a/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/runtime/staticobject/StaticObject.java) | Guest object identity distinct from host Java object abstraction | Guest/host boundary precedent, not a ready-made Protos number implementation |
| [V8](https://github.com/v8/v8/blob/748788e7d260b6611d90a56703304cbe113d4b27/src/objects/tagged.h) and [CPython](https://github.com/python/cpython/blob/5a22a62b96a68c2dd31784c5e3427ad7b365388a/Objects/longobject.c) | Tagged/boxed alternative techniques for numbers | Falsification baselines; neither dictates Truffle JVM or D197 type semantics |

## Five candidate models and GITHUB010 twelve-axis scoring

**A:** keep broad existing Java wrapper API and optimize only local hot arithmetic.
**B:** primitive small/Float plus dedicated compact Java runtime large-number carrier; rich Fraction/Complex guest objects.
**C (selected):** primitive small/Float plus generic ordinary immutable Protos guest-object rich-number model and encapsulated exact algorithms.
**D:** universal number object/record carrier required even on small hot paths.
**E:** globally tagged/NaN-boxed universal VM values.

All scores 1–5 (5 best). `H/M/L` denotes **confidence in this qualitative rating**, not implementation evidence. Every cell states the relevant reason; no sum overrides a serious correctness or pay-as-you-grow failure.

| GITHUB010 axis | A | B | C | D | E |
| --- | --- | --- | --- | --- | --- |
| 1. Correctness/invariants | 3/H: wrappers work with rewrites | 4/M: explicit guards feasible | 4/L: guest semantics need proof | 4/M: single object protocol | 3/L: tags need semantic escape |
| 2. Protos alignment | 2/H: host shape leaks | 4/M: pragmatic compact bridge | **5/M: ordinary values** | 2/H: imposed number institution | 2/L: representation-led model |
| 3. Present need/pay-for-use | 2/H: eager wrappers | 4/M: primitive ordinary | **5/M: primitive ordinary** | 1/H: universal materialization | 3/L: pervasive tag machinery |
| 4. Incremental growth | 2/M: API spread persists | 4/M: add family adapters | **5/M: generic rich extension** | 3/M: add carrier variants | 2/L: global encoding changes |
| 5. Future-option resilience | 2/M: host-bound | 4/M: localized large class | **5/M: rich numeric objects compose** | 3/M: shared type universe | 2/L: backend lock-in |
| 6. Scalability | 3/M: widespread coupling | 4/M: compact rare values | 4/L: object footprint unknown | 3/M: persistent overhead | 4/L: potentially compact |
| 7. Conceptual simplicity | 2/H: many wrappers/adapters | 4/M: explicit small/large split | **4/L: one object model, difficult invariants** | 2/M: duplicated value universe | 2/L: global representation machinery |
| 8. Portability/freedom | 2/H: Java carrier contracts | 3/M: Java large adapter | **5/M: semantic Protos model** | 3/M: platform-dependent record | 2/L: tagged word assumption |
| 9. Runtime/resource cost | 2/M: boxing/deopt tax | **4/M: proven compact precedent** | 4/L: no small tax; rich object unknown | 2/M: eager object tax | 4/L: cost unknown on Truffle |
| 10. Failure/operability | 3/M: familiar, scattered | 4/M: bounded large fallback | 3/L: identity/transfer difficult | 3/M: uniform but heavy | 2/L: debugging and deopt complexity |
| 11. Migration/reversibility | 4/H: minimal patch | 4/M: gradual migration | 3/M: broad generic adapters | 2/M: global rewrite | 1/M: VM-wide rewrite |
| 12. Evidence maturity/risk | 4/H: known baseline | **5/H: GraalPy/Enso/Squeak** | 2/L: core generic proof absent | 3/M: ordinary object precedent | 2/L: V8 not Truffle Protos |

**Decision rationale:** C uniquely targets both the user-visible compositional types required by D197 and the original VM-wide Java dependency inversion while preserving a primitive, pay-as-you-grow execution path. B has the stronger mature implementation precedent and lower feasibility risk, so the C selection includes a noncompensating proof gate and an explicit reapproval path if falsified. A cannot satisfy the API containment requirement; D imposes unused optional numeric machinery on every operation; E has disproportionate whole-VM migration and portability cost.

## Explicit falsifiers, obligations and owners

1. **Identity and hash.** A rich numeric value cloned, transferred or materialized twice must retain approved mathematical equality, family-sensitive semantic identity and cross-family hash coherence. Standard object reference identity does not suffice. [`ProtosSealedValue`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosSealedValue.java) currently lacks the needed transfer policy; using it unmodified fails.
2. **Exact rich arithmetic.** `Long.MAX_VALUE+1` must yield Protos BigInteger, and subtracting one must yield Protos Integer. `Long.MIN_VALUE/-1` becomes BigInteger. For exact integer `2/1` returns Integer, `1/2` reduced Fraction, `(1/2)+(1/2)` Integer. Rich number objects must avoid host-value alias/mutation across contexts.
3. **Complex and IEEE.** A Complex with a Fraction real and BigInteger imaginary component must not first become two doubles. Exact imaginary zero may demote, while `Complex(-4.0,+0.0)` and `Complex(-4.0,-0.0)` must preserve different branch information. Host double handling must not silently collapse `-0.0` or NaN representation.
4. **Rounding and contagion.** `Fraction(10^400+1,10^400).asFloat` rounds the exact rational once to `1.0`, not NaN from `Infinity/Infinity`. Standard mixed arithmetic with Float converts the exact operand to Float *before* IEEE operation, including BigInteger/Fraction; exact equality/order must not be based on that lossy conversion.
5. **Overridable sends.** A selected optimized standard numeric method must deopt/fallback when a nearer override changes the result. Fast operations are not syntax-only bypasses.
6. **Bytecode and invocation.** Both current DSL root families, inline guarded numeric operations, local slots, closure capture, argument/result forwarding, generic sends, debugger/materialized frames must be traced with actual carriers. Merely adding `long.class` and `double.class` to an annotation does not discharge this gate.
7. **Actor/Process/parallel and Java interop.** Prove correct family identity and immutable-value transport across domains, no family-private state leak, proper guest/host admission, and no extra actor bookkeeping on unused paths.
8. **Dependency census and performance.** Audit all `src/main` instances of Java `BigInteger`, `ProtosIntegerValue`, `ProtosFloatValue` by module/boundary owner and compare counts after implementation. Benchmark pinned Graal graph nodes, compilation stability, memory allocation and timing for `1`, `1+2`, overflow/demotion, Float/mixed operations, exact division, and optional rich values. Stop when measurements are NOT_STABLE rather than claiming speedups.
9. **Unknown future feature.** Additional exact algebraic/symbolic families could strain the generic value normalization boundary. The escape path is a new approved language design and, if needed, a narrow specialized physical rich-carrier amendment—not pre-installing a universal numeric tower engine now.

**Important separate semantic STOP:** D197 did not ratify the exact future public meaning of `Integer.recognizes` or a newly introduced common integer-domain protocol. Both the old unbounded `Integer.recognizes` contract and the new visible signed-64 `Integer` must be reconciled by a separately approved semantic amendment, especially for math library gcd/factorial and Array indices. PLAT056 cannot choose this for I090.

## Migration proposal after design ratification (not implementation authorization by itself)

1. Reinspect actual product HEAD, produce full numeric import/call-site ownership census and prototype/recognition/value-boundary inventory. Prove F1 rich guest-value feasibility with identity/transfer semantics; if falsified, stop before distributing carrier-specific changes.
2. Introduce a bounded, internal numeric-value capability/normalization service that recognizes D197 semantic families without spreading `java.math.BigInteger`. Preserve host APIs already promised; no universal new wrapper.
3. Carry long/double through ordinary bytecode, canonical standard dispatch, locals, arguments/returns, direct primitive arithmetic and exact overflow fallback. Preserve deopt and D013 overrides. Integrate I089 B1 at its published product revision and coordinate I090 D197 exactness.
4. Adopt guest rich-number creation/normalization, cross-family equality/hash/identity, collection acceptance, transfer, interop and tools incrementally with conformance checks, then audit dependency containment.
5. Human executor runs focal/integrated suites at coherent green boundaries and performs product Git publication; PERF040 collects pinned BGV and timing/compiled proof only after correct platform and D196/D197 target behavior is available.

The smallest immediate solution is a primitive-first guest value boundary and **only** those rich values that D197 already requires. Fraction and Complex are no longer speculative future facilities, but their entire heavy machinery need not be constructed by unused arithmetic. Omitting the generic boundary today would force a later second VM-wide pass over indices, collections, transfer and interop. Conversely, implementing all rich numeric operations before verifying object identity/transfer would create costly rework.

## Provenance and validation facts

```text
PROTOS_REVISION_INSPECTED=0db24f00ff2d92d642351d7f7535517fe01a55ce
D197_DECISION_RECORD_BASELINE=5e0655d696c5323096f3faef4311a6a0b4640404
PLAT056_OWNER_APPROVAL=EXPLICIT_2026-10-10_C
GITHUB010_12_AXES=COMPARED_QUALITATIVE
GITHUB021_SELECTED_INVARIANT_DELTA=DOCUMENTED
COMPLETE_SRC_MAIN_CENSUS=NOT_DONE
F1_GUEST_OBJECT_IDENTITY_TRANSFER_PROOF=PENDING
CODE_EDITS=NONE
PRODUCT_COMMIT=NONE
OWNER_REPORTED_LOCAL_TESTS=PASS_EXISTING_BASELINE
AGENT_EXECUTED_TESTS=NONE
BENCHMARKS_EXECUTED=NONE
RUNTIME_CONFORMANCE=NOT_CLAIMED
I090_RECOGNITION_APPROVAL=OPEN
```

Prepared with AI assistance and checked against source-linked prior research. The owner, not the agent, approved Candidate C. This evidence does not constitute an independently measured performance improvement or a tested product implementation.
