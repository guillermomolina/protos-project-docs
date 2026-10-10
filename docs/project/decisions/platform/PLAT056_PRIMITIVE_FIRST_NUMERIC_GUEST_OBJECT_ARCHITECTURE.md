# PLAT056 — Primitive-first numeric values with ordinary Protos guest-object materialization

**Status:** RATIFIED — Candidate C; implementation and feasibility/conformance gates outstanding.
**Owner approval:** 2026-10-10, explicit active owner reply: “ok aprobada la recomendacion”, following the D197-aware re-evaluation of Candidate C and the owner’s confirmation that all local tests passed.
**Live architecture issue:** [PLAT056/#866](https://github.com/guillermomolina/protos/issues/866).
**Ratified semantic inputs:** [D197/#865](https://github.com/guillermomolina/protos/issues/865), [D197 durable decision](../language/D197_EXACT_NUMERIC_TOWER_AND_NORMALIZATION.md); [D196/#864](https://github.com/guillermomolina/protos/issues/864).
**Prior platform constraints:** PLAT035 single Bytecode DSL execution backend, PLAT040 guarded ordinary dispatch/conditional materialization, PLAT036/I075 primitive-carrier precedent.
**Inspected product HEAD at decision:** `guillermomolina/protos@0db24f00ff2d92d642351d7f7535517fe01a55ce`.
**Existing project evidence:** [PLAT056 baseline](../../evidence/PLAT056/PLAT056_NUMERIC_GUEST_VALUE_BASELINE_AND_APPROVAL_GATE.md); [candidate comparison and approval evidence](../../evidence/PLAT056/PLAT056_CANDIDATE_C_RATIFICATION_AND_FALSIFICATION.md).

> This is a non-normative, runtime/platform architecture ratification. D197 and D196 are the separate language-semantic authority; the current Protos specification and runtime do not become conformant by publication of this record. The maintainer's PASS concerns the pre-implementation local suite; no new PLAT056 code, tests, benchmarks, Native Image or product commit are claimed.

## Exact selected architecture: Candidate C

1. **Primitives on ordinary paths.** Carry canonical signed-64 Protos `Integer` values as Java/Truffle primitive `long`, and binary64 `Float` as primitive `double`, through the normal executable path whenever semantics permit. `Integer + Integer` without overflow and `Float + Float` must not unconditionally create `ProtosIntegerValue`, `ProtosFloatValue`, a guest object, a Java `BigInteger`, or Fraction/Complex state. These are structural requirements, not merely optimistic escape-analysis expectations.
2. **Ordinary Protos guest objects for rich values.** The default physical materialization of D197's canonical guest-visible `BigInteger`, `Fraction`, and `Complex` uses the general Protos object/prototype/slot/delegation model with immutable numeric content, rather than inventing a dedicated Java `ProtosBigIntegerValue`, `ProtosFractionValue`, and `ProtosComplexValue` merely to make them values. A narrow encapsulated numeric payload is permitted where exact magnitude/limbs or immutable state cannot sensibly live in ordinary guest slots; it must remain an implementation detail of an ordinary guest-representable value, not a second object universe or a general-purpose VM contract.
3. **Canonical result representation follows D197.** The runtime normalizes exact results to the D197 **semantic families**: signed-64 `Integer` within [-2^63, 2^63-1], `BigInteger` outside; reduced nonintegral exact `Fraction`; and `Complex` with Protos numeric components. A canonical exact-zero imaginary component can collapse to the real family; an inexact IEEE signed-imaginary zero must retain the observable branch-cut information. This platform choice does **not** invent additional semantics beyond D197.
4. **A narrow numeric service boundary.** Expose internal semantic family recognition, exact integer extraction/conversion, arithmetic/normalization, mathematical comparison, semantic numeric identity, normal hash, identity hash, and projection/transfer through cohesive bounded services. Generic VM clients (Array/Map/IdentityMap, indexing, I/O, Actors, Process, parallel isolation, debugger, interop, serializers) consume semantic capabilities or ordinary guest values rather than importing Java `BigInteger` or depending on three new family-specific wrappers. Do **not** merely rename all Java `BigInteger` imports to `ProtosIntegerValue`.
5. **Java numeric tools are encapsulated, not forbidden.** `java.math.BigInteger` remains legitimate for truly large exact arithmetic, parsing, GCD/rational reduction, correct rational-to-binary64 conversion, mathematical helpers, or explicitly documented host Java/interop/transport boundaries. A guest Protos `BigInteger` is a semantic numeric family; Java `BigInteger` is not its semantic definition. The decision targets the spread of representation dependencies across unrelated Java code, not an arbitrary zero-import or fixed-count quota.
6. **Pay as you grow.** Optional Fraction/Complex, large-integer, reflection, interop, Actor/Process, suspension, and richer guest-state machinery must not be carried or allocated by default for trivial arithmetic. Object materialization is conditional on a true semantic need. A stored or escaping value can entail a materialized carrier; the architecture does not claim all Java boxing at all host boundaries is impossible.
7. **Ordinary dispatch remains authoritative.** All numeric operations are ordinary redefinable Protos messages with the approved D013/PLAT040 selection, assumptions, invalidation, deoptimization, and exact fallback. The fast path cannot bypass overridden methods, direct-context ownership, guest-visible identity, reflection, or effect/error semantics. Retain PLAT035's single Bytecode DSL execution backend.
8. **Language-independent semantics.** Semantic family, `==`, non-overridable `===`, hash coherence, Float NaN and signed-zero behavior, guest object recognition, transfer and isolation do not depend on whether the underlying Java carrier is a primitive, host helper, or materialized guest object.

## Explicitly NOT ratified by PLAT056

- No concrete Java numeric-service API, special Truffle node/bytecode opcode, DynamicObject/Shape requirement, storage layout, universal tagged value, or per-family Java subclass has been approved.
- No additional D197 language semantics or new public selectors are selected. In particular **`Integer.recognizes` versus a common exact-integral recognition API remains unresolved**, as explicitly excluded by the D197 approval. A separate semantic owner approval is required before I090 changes that public behavior; this PLAT decision cannot resolve it.
- Candidate B (primitive-first with a dedicated compact large-number runtime carrier) is the credible fallback **if Candidate C's guest-object integration cannot satisfy the mandatory identity, immutability, cross-domain and cost gates**. Failure does not authorize an automatic switch to B; provide counterevidence and return to the project owner for an explicit architecture amendment.
- No requirement to eliminate every textual Java `BigInteger` reference or to hide a previously supported host Java API. Each boundary must be classified as numerical algorithm, physical carrier, intentional Java ABI, or accidental coupling.
- No implementation, benchmark result, Native-image/JIT success, normative change, or removal of I089/PERF040/I090 blocking conditions is implied by this design decision alone.

## Required feasibility and falsification gates before committing to the full migration

**F1 — ordinary immutable rich guest values.** Show with actual Protos source-level and runtime inspection that a `BigInteger`, `Fraction` and `Complex` can have ordinary delegation, immutable numeric content, appropriate identity-by-value and hash coherence, safe reflection, stable family recognition and no gratuitous Java-class proliferation. `ProtosSealedValue` is a useful partial precedent, not the chosen solution: its current identity and isolation-transfer rejection are not acceptable as the numeric contract without further architectural proof.

**F2 — primitive execution end to end.** Trace and exercise literals, both existing Bytecode DSL roots, operands, local slots, argument passing/returns, closures, selected standard sends, invalidation and fallback. Both roots currently use `boxingEliminationTypes={int.class}`; enabling `long` or `double` alone is not proof of a primitive execution chain.

**F3 — result normalization and rounding.** Check ±2^63 boundaries, BigInteger-to-Integer demotion, exact quotient-to-Integer/BigInteger/Fraction, exact rational round-once conversion, D196 operand-first mixed Float arithmetic, Complex(Fraction,BigInteger), exact-zero Complex collapse, and protected IEEE signed-imaginary-zero behavior.

**F4 — semantic observation and isolation.** Verify `==`, `===`, `hash`, `identityHashOf`, Map/IdentityMap, indexing, numeric acceptance, reflection/debugger, foreign host access, multi-Context isolation, Actor/Process/P copy/transfer, serialization and Native-image restrictions without host carrier leakage.

**F5 — dependency containment.** Produce an exhaustive production `src/main` inventory of references to `java.math.BigInteger`, `ProtosIntegerValue` and `ProtosFloatValue`, with classification and before/after counts by module; establish a near-zero *unjustified* dependency footprint outside numerical algorithms and explicit boundaries. Existing “60+ files” search observations are only preliminary, not an audited final census.

**F6 — performance and reversibility.** Demonstrate removal of avoidable ordinary-path materializations structurally (not only allocation speculation), then pinned Graal graph and timing comparisons under PERF040. If the C carrier adds unacceptable object, transfer, lookup or deoptimization taxes, stop and return evidence to PLAT056 for the separately approved fallback gate.

## Work ownership and ordering

- **PLAT056/#866:** completed platform *decision* on publication of this record; design closes independently of implementing it.
- **D197/#865:** previously ratified semantics; **I090/#868** owns their normative/runtime implementation. I090 remains BLOCKED by unresolved public exact-integer-domain/recognition semantics and by the need to safely implement the now-selected C architecture; candidate ratification alone does not assert the implementation feasibility gate is met.
- **D196/#864:** ratified B1; **I089/#867** stays separately READY, implementing only D196 B1 at its actual product HEAD without assuming I090 changes.
- **PERF040/#862:** remains BLOCKED by actual platform implementation/conformance and I089 as applicable; its benchmark evidence consumes the selected architecture and must not itself redefine it.
- A cohesive implementation slice/work item should own the PLAT056 primitive-carrier and numeric-boundary migration, coordinated with I090, without placing Dxxx decisions in an Ixxx issue or creating arbitrary micro-slices.

## Invariant/delta check — GITHUB021

| Existing owner decision | Approved surface retained or prospectively amended |
| --- | --- |
| D006/#164 and D156/#616 | D197 explicitly supersedes the former single unbounded visible Integer and Float-only exact-division assumptions; rejection of eight width-specific modular arithmetic families remains |
| D196/#864 | Operand-first binary64 conversion for mixed Float arithmetic retained; D197 extends the exact real inputs as recorded |
| D197/#865 | Five semantic families and canonical normalization are **inputs**, not PLAT056 inventions; public `recognizes` details remain undecided |
| D013 and PLAT040 | Ordinary dynamic dispatch, guarded fast sends, validity assumptions, fallback and conditional guest-state materialization retained |
| PLAT035, PLAT036/I075 | One Bytecode DSL execution backend and prior primitive-carrier progress preserved |
| PLAT056 older checkpoint | “BigInteger not a guest-visible family” **superseded** by D197; containment goal refers only to **Java** `BigInteger` API spread, not removal of the Protos `BigInteger` family |

**Owner approval scope:** exact D197-aware Candidate C above, including its feasibility gate, restricted Java BigInteger algorithm boundary, no mandatory small-number wrappers, ordinary object preference and alternative B only by future separate approval. No unapproved semantic detail, source patch or deployment has been inferred.

## Provenance, implementation and validation

```text
PLAT056_SELECTED_CANDIDATE=C
PLAT056_OWNER_APPROVAL=EXPLICIT_2026-10-10
PLAT056_DECISION=RATIFIED
D197_NUMERIC_MODEL=RATIFIED_N2
PROTOS_REVISION=0db24f00ff2d92d642351d7f7535517fe01a55ce
PROTOS_RUNTIME_IMPLEMENTATION=NOT_REPORTED
PROTOS_NORMATIVE_UPDATE=NOT_REPORTED
OWNER_REPORTED_LOCAL_TESTS=PASS_PREIMPLEMENTATION_BASELINE
AGENT_EXECUTED_TESTS=NONE
AGENT_EXECUTED_BENCHMARKS=NONE
AGENT_EXECUTED_PRODUCT_GIT=NONE
C_FEASIBILITY_GATES=OPEN
I090_INTEGER_RECOGNITION_SEMANTIC_GATE=OPEN
```

Prepared with AI assistance from the PLAT056 investigation and the owner-reviewed D197-aware recommendation. Project owner expressly selected Candidate C; no independent test execution or performance acceptance is claimed.
