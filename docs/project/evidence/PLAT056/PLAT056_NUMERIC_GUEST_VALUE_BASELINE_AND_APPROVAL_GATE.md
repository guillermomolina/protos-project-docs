# PLAT056 — Numeric guest-value baseline and architecture approval gate

**Evidence status:** INVESTIGATION CHECKPOINT — not a platform ratification, completed comparison, or implementation acceptance.  
**Issue:** [PLAT056 / guillermomolina/protos#866](https://github.com/guillermomolina/protos/issues/866)  
**Product revision inspected:** `0db24f00ff2d92d642351d7f7535517fe01a55ce` (`guillermomolina/protos`, `main` at inspection, 2026-10-09)  
**Normative owner:** `spec/semantics/VALUES_AND_COLLECTIONS.md`  
**Decision gate:** `PLAT056_ARCHITECTURE_APPROVAL=PENDING`; `IMPLEMENTATION_AUTHORIZED=NO`

## Provenance and validation boundary

The project owner reported on 2026-10-09 that the local `git diff --check` was clean and all local tests had passed. The report did not identify a command transcript, an exact test suite, or a product commit corresponding to a **PLAT056 numeric-representation implementation**. These are owner-reported baseline-validation facts, not proof of a newly selected or installed architecture. No numeric platform implementation is attributed to this checkpoint. The repository publication of **this evidence record** must not be confused with a product change.

No exact PLAT056 architectural candidate was explicitly selected in the active approval record. The live issue remains open. Owner constraints in the issue are research invariants, not authorization to choose a physical design.

## Current source-backed baseline

At the inspected product revision:

1. [`ProtosIntegerValue.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosIntegerValue.java) uses a `long smallValue`, a nullable `BigInteger bigValue`, exact small-`long` arithmetic with `Math.addExact`/`subtractExact`/`multiplyExact`, and `@TruffleBoundary` large-arithmetic fallbacks. The small result still constructs `ProtosIntegerValue`. `value()` converts a small `long` to host `BigInteger` on demand. This is concrete wrapper/API coupling, not evidence that `BigInteger` arithmetic runs on every successful small addition.
2. [`ProtosFloatValue.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosFloatValue.java) contains the semantic `double` in a Java object with Truffle interop exports; whether this box is eliminated in a specific compiled path is a separate empirical question.
3. [`ProtosNumberLiteral.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/runtime/ProtosNumberLiteral.java) first parses an Integer literal through `new BigInteger(..., radix)` and constructs `ProtosIntegerValue`; Float literals construct `ProtosFloatValue`. The code alone does **not** show per-call `BigInteger` parsing in a compiled steady-state workload; distinguish lowering from execution.
4. [`ProtosStandardNumericConversionProtocol.java`](https://github.com/guillermomolina/protos/blob/0db24f00ff2d92d642351d7f7535517fe01a55ce/src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumericConversionProtocol.java) routes Integer-to-Float conversion via `integer.value()` and `ProtosBinary64Rounding.divideExactIntegers(..., BigInteger.ONE)`. Existing IEEE correctness must be maintained; removing this generic route from small-value hot paths is an architectural question, not permission to change rounding.
5. The live [PLAT056 issue](https://github.com/guillermomolina/protos/issues/866) lists additional cross-layer consumers in arrays, maps, identity, transfer, I/O, SPI, and tooling. **This is a cited investigative inventory, not a verified exhaustive import/call-site census**; quantify and classify it on current HEAD before final candidate selection.

## Semantic and decision boundaries

- The semantic **Integer** is one exact, unbounded family, regardless of whether a computation uses a primitive machine `long`, a host `BigInteger` algorithm, or a future guest materialization. A large result must normalize back to an ordinary semantic Integer; there is no guest-visible SmallInteger/BigInteger split.
- The semantic **Float** remains binary64. Signed zero, NaN, infinity, identity, numeric hashing, exact cross-family comparison, guarded D013 dispatch, object reflection, and interop must retain their existing contracts.
- [D196/#864](https://github.com/guillermomolina/protos/issues/864) **has** approved B1 for standard mixed Integer/Float `+ - * /`: convert the Integer to binary64 *before* the binary64 operator and preserve operand order. [Durable D196 decision](../../decisions/language/D196_MIXED_INTEGER_FLOAT_ARITHMETIC_PROMOTION.md). The distinct [I089/#867](https://github.com/guillermomolina/protos/issues/867) must still publish and test the normative/runtime implementation. D196 does **not** select PLAT056's carrier design.
- [D197/#865](https://github.com/guillermomolina/protos/issues/865) still owns optional Fraction/Complex and exact-division questions. No mandatory `Fraction`, `Complex`, or `ProtosBigIntegerValue` Java representation is approved. Java `BigInteger` remains permitted as a private algorithmic tool or an explicit host boundary type.
- [PLAT035](../../decisions/platform/PLAT035_SINGLE_EXECUTION_BACKEND_LEGACY_AST_RETIREMENT.md), [PLAT040](../../decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md), [PLAT036](../../decisions/platform/PLAT036_BYTECODE_DSL_LEXICAL_STATE_REPRESENTATION_BOUNDARY.md) and the I075 precedent constrain architecture; do not create a second backend or bypass ordinary guarded sends.

## Research questions still requiring evidence

The next investigation must establish an exhaustive classified `src/main` dependency inventory for `BigInteger`, `ProtosIntegerValue`, and `ProtosFloatValue`; distinguish arithmetic algorithms, legitimate interop, runtime carrier mechanics, unrelated leaky imports, and tooling. It must trace full lifecycle of small `long` and `double` carriers across **both** Bytecode DSL roots, local slots, calls/returns, closure capture, dispatch, deoptimization, object materialization, arrays/maps, identity/hash, semantic transfer, I/O and host interop.

Falsify the attractive primitive-carrier/lazy-materialization recommendation against: observable numeric identity and delegation, direct storage in collections, `Value` embedding, exact long-overflow fallback, large-result demotion back to small values, actor/process transfer and copy boundaries, IEEE 754 negative zero and huge-integer conversion, compiled fallback, debugger inspection, multi-Context isolation, native-image interpreter-only behavior and genuine guest-visible materialization. Explicitly explain why a Java wrapper is needed or absent **at each boundary**, not merely count Graal nodes.

Compare source-grounded alternative designs in TruffleSqueak, GraalJS, GraalPy, TruffleRuby, Espresso, SimpleLanguage, Enso and Pkl, plus relevant non-Truffle precedents. Include credible status quo, localized-specialization, primitive-carrier/lazy-materialization, guest-object-backed large Integer and tagged/NaN-boxing falsification alternatives. Score each surviving candidate on **all twelve** `AGENTS.work/DESIGN.md` axes, with evidence confidence, counterexamples, predicted compiled costs, migration order, and precise acceptance/compatibility criteria. Separate observed facts, hypotheses, and recommendations.

## Coordination and execution gates

`PLAT056=OPEN`; `EXACT_OWNER_APPROVAL=NOT_RECORDED`; `INVARIANT_DELTA_CHECK=PENDING`; `REQUIRED_RATIFICATION_PUBLICATION=PENDING`; `RUNTIME_IMPLEMENTATION=NOT_AUTHORIZED_BY_PLAT056`.

The investigation may produce a **proposed** exact platform candidate for owner review, but cannot treat it as selected. An approved candidate requires a separate durable record under `docs/project/decisions/platform/`, revision-bound review and ratification **before** closure or enabling dependent platform code changes. Do not disguise research as an implementation slice.

I089 is independently actionable under ratified D196; its correctness work does **not** require PLAT056 first. PERF040 remains blocked until both the approved/published platform decision and I089 normative/runtime conformance are available. No new formal issue is necessary merely to carry on PLAT056's own design investigation.
