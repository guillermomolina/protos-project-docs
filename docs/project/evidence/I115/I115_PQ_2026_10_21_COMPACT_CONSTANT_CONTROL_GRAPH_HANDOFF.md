# I115-P/Q — compact constant control and literal-return physical graph checkpoint

**Date:** 2026-10-21. **Role:** revision-pinned experimental evidence, not normative semantics or a compiler-performance benchmark.
**Product:** [`guillermomolina/protos@44204ec158aeedcb8262ea50b66f7f27b7583456`](https://github.com/guillermomolina/protos/commit/44204ec158aeedcb8262ea50b66f7f27b7583456) on the **I115 experimental branch**, directly after [I115-O `ab632560f19e1b876b9b6c8f2018f5cb86e9eb13`](https://github.com/guillermomolina/protos/commit/ab632560f19e1b876b9b6c8f2018f5cb86e9eb13). Product commit subject: `I115-Q: compact literal returns and constant control-flow lowering`.
**Issue owners:** [I115/#931](https://github.com/guillermomolina/protos/issues/931) — experimental implementation; [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930) — residual compiler graph and semantics; [PERF039/#858](https://github.com/guillermomolina/protos/issues/858) — cross-Truffle workload definitions. **None are closed by this record.**

**Provenance:** User-supplied technical report on 2026-10-21 says focused tests are green and all **8/8** independent Tier 1 / Tier 2 captures completed. The published source commit and exact five changed paths were independently verified through GitHub. Raw BGV files, compiler logs and test execution were **not** independently replayed for this record; all numeric measurement results below are maintainer-reported. No product main/release integration is claimed.

## 1. What the published patch does

The product comparison `ab632560...I115` is one commit ahead with five paths:

- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java` — only production file; **I115-P** broadens the compact whileTrue condition to already-proven Boolean-valued expressions; **I115-Q** prepares guest Integer literal returns and emits true/false literal conditions directly as Java booleans.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI115PCompactBooleanWhileTest.java` — new general Boolean-condition regression tests.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI115QCompactLiteralReturnTest.java` — guest-Integer literal returns, controls, fallbacks, and representation tests.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI115QCompilerGraphTest.java` — opt-in real selected-`run` compilations.
- `tools/i115_q_graph_capture.py` — eight-case/tier capture driver, DSL and compilation-log summary, BGV retention.

I115-M/N's whole-root compact admission remains the enabling architecture: admitted controls lower to structured bytecode without separate callback roots or ordinary prepared sends; inadmissible roots stay in general lowering. I115-O already specialized certified bindings storing guest Integer constants. I115-Q adds a **terminal guest Integer literal** certification, so a literal `run: () => { 1 }` loads the prepared guest value and uses `epilogProjectsReturnCarrier=false`. For literal `true` or `false` conditions, the existing Boolean lowering now loads a primitive Java Boolean constant without the `ExperimentalCanonicalBooleanCondition` conversion. I115-P similarly permits a Boolean-valued condition in `whileTrue`, not only an integer comparison.

No new Bytecode DSL operations are introduced in this commit. Importantly, the **absence of operation names containing `Experimental` is not sufficient to identify the compact lowering**; tests check the absence of generic send/Closure dispatch.

## 2. Four measured `run` sources and Bytecode DSL FULL

The opt-in capture class declares four separate `run` sources, all returning 1:

| Exact graph-capture case | Module FULL | `run` FULL | Total FULL | Roots |
| --- | ---: | ---: | ---: | ---: |
| `primitive-return-literal`: `run: () => { 1 }` | 11 | **4** | 15 | 2 |
| `primitive-if-true`: `result: null; true.ifTrue() { result = 1 }; result` | 11 | **10** | 21 | 2 |
| `primitive-if-false`: `false.ifTrue() { 99 }; 1` | 11 | **8** | 19 | 2 |
| `primitive-while-zero`: `(() => false).whileTrue() { 99 }; 1` | 11 | **9** | 20 | 2 |

**Source-comparability caveat:** the opt-in **`primitive-if-true` graph-capture program is not the exact original cross-Truffle harness `primitive-if-true` source**, which returns the literal callback value directly. It uses a `result` binding. Do not assign its 10-FULL census or its graphs to the official benchmark without an exact source match. The `primitive-while-zero` graph-capture source does match the minimal benchmark expression shape.

**Bounded incremental Q win:** the maintainer reports **`primitive-while-zero` `run` FULL 10 in I115-P → 9 in I115-Q**, one instruction saved. The other numbers describe the accumulated I115-M/N/O/P/Q experimental state; they are **not all reductions caused by Q**. A post-O matched per-case baseline for all four sources is not supplied.

## 3. Real Tier 1 and Tier 2 compiler counters

All four captured `run` programs have the same **maintainer-reported compiler log IR and CodeSize counters** at the specified tier, despite different Bytecode FULL counts:

| `[engine] opt done` counter | Tier 1 | Tier 2 |
| --- | ---: | ---: |
| IR first counter | **104** | **47** |
| IR second counter | **123** | **59** |
| CodeSize | **476 bytes** | **256 bytes** |

The values are equal **in count** across the four cases, not proof of identical compiled graphs, instruction sequences, semantics, execution paths, throughput or latency.

**Critical phase-provenance limitation:** the published `tools/i115_q_graph_capture.py` parses `[engine] opt done ... IR X / Y ... CodeSize Z`, assigning the first counter to `after_truffle_tier_ir` and the second to `final_ir`. It **does not decode or count the exact `After TruffleTier` and `After low tier` `StructuredGraph` snapshots in the BGV binaries**. The script explicitly prints: `I115_Q_BGV_NOTE=Raw graphs retained; StructuredGraph phase selection not yet validated`. Therefore `104/47` and `123/59` must be described as **compiler-log counters**, *not independently verified BGV phase-node census*. Raw BGVs are retained by the human's capture, but this publication does not include their binaries, phase hash identities or exact compiler IDs.

Tier 1 versus Tier 2 is a compiler-tier comparison, **not** an improvement between Protos revisions. CodeSize is generated code size, not a timing result. **No new matched GraalPy/GraalJS comparison was performed** in the supplied evidence. Prior PERF041V numbers describe a different source program and must not be blindly carried into this four-case matrix.

## 4. Human validation and acceptance limits

Human report: focal tests green; **8/8** Tier 1/Tier 2 captures for the four cases correct, each with a BGV associated with the selected `run` root; two physical roots for each program; observable result 1. The new tests cover signed-64 literal returns, Boolean/null alternatives and negative constant-return admission, host/Closure consumers, condition true/false/dynamic cases, structural dispatcher absence, cached/uncached and reparse boundaries. The exact aggregate JUnit test count and integrated `make test` outcome were not supplied here; mark those **not independently verified**.

**Not established:** complete semantic conformance, compiler-graph isomorphism, phase-level BGV node equality, steady-state benchmark stability, speedup, throughput, latency, or broad production readiness.

## 5. Semantic shortcuts: newly expanded admission vs inherited deviations

**I115-P/Q additions to the physical admission surface:**

1. Statically admitted Boolean-valued conditions, including literal `false`, can now use structured `While` without ordinary callback call targets (P).
2. Literal Boolean conditions use a direct primitive Java `boolean` load rather than passing through `ExperimentalCanonicalBooleanCondition` (Q).
3. Signed-64 terminal Integer literals can use guest constants and bypass numeric return-carrier projection, extending I115-O's early materialization of certified binding stores (Q).

**These representation choices alone are not automatically language violations.** The observable semantics require separate proof. Any broader use of compact control exposes the following **existing I115-M/N/P experimental deviations**, not all newly created in Q:

- Ordinary method lookup and overridable `ifTrue` / `whileTrue` dispatch can be bypassed.
- Condition/callback Closure object identity and materialized activations, lexical capture, `this`, `methodHome`, and return/control transport need not match general Protos execution.
- Guest Context/slot authority, reflection, mutation and lexical observation may be bypassed by root Bytecode locals.
- `ReturnHome`, `InvalidReturn`, Task, cancellation, Future, continuation and error/fallback paths are not universally preserved.
- Earlier I115-M signed-64 native arithmetic does not implement general arbitrary-precision promotion.

**Additional verification needed for the Q return optimization:** value identity of guest Integer, `IdentityMap`/hash/parent recognition, interop, multiple Contexts, and callers that accept primitive carriers. The report identifies **no independently confirmed new language-semantic violation caused specifically by the Q materialization change**, but this is not a conformance proof.

Authoritative normative owners remain `spec/semantics/VALUES_AND_COLLECTIONS.md`, `CALLABLES.md`, `OBJECT_MODEL.md`, and `EXECUTION_AND_CONTROL.md`. No specification amendment or PLAT design approval is implied.

## 6. PLAT066-C follow-up and state

I115-Q is a useful **compact-IR experimental reference**, not a production architecture mandate: it demonstrates 4–10 FULL instructions in each `run` and eight successful compilation captures, at the cost of bypassing existing dynamic guarantees.

Next checks before drawing stronger conclusions:

1. Independently decode and classify the correct BGV compiler-IR phases (`After TruffleTier`, `After low tier`), report distinct phase counts and graph fingerprints, and compare exact source/root/tier selections.
2. Obtain truly matched **I115-P → I115-Q** physical A/B captures, rather than treating across-tier differences or accumulated I115-M/Q counts as an incremental gain.
3. Measure benchmark steady-state admission/performance and compare with GraalPy only for matched program shapes and JVM/compiler configurations.
4. Design guarded/invalidation-backed ways to restore method lookup, Closure/Context observability, activation and control transfer without losing all the physical gains. Such semantics-restoration proposals belong to the open PLAT work and require approval of any substantive design change.

**Disposition:** `PRODUCT_COMMIT=44204ec158aeedcb8262ea50b66f7f27b7583456`; `SOURCE_PUBLICATION=VERIFIED`; `FOCAL_TESTS=HUMAN_REPORTED_GREEN`; `GRAPH_CAPTURE=8_OF_8_HUMAN_REPORTED`; `DSL_FULL=4/10/8/9_RUN`; `GRAAL_IR_OPT_DONE=104/47_FIRST_AND_123/59_SECOND`; `CODESIZE=476/256_BYTES`; `BGV_PHASE_CENSUS=NOT_INDEPENDENTLY_VALIDATED`; `NEW_COMPARATIVE_SPEEDUP=NOT_MEASURED`; `NORMATIVE_CONFORMANCE=NOT_ESTABLISHED`; `I115_PLAT066C_PERF039=OPEN`.
