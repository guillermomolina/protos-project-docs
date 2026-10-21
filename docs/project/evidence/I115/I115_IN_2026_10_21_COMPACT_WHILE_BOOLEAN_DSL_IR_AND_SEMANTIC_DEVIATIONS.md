# I115-I through I115-N / PLAT066-C — experimental compact whileTrue and Boolean control: gains, mechanisms, and newly introduced semantic deviations

**Date:** 2026-10-21. **Scope:** the bounded experimental compact-control sequence **I115-I/J/K/L/M/N**, not the accumulated deviations of earlier I115-A through I115-H. **Classification:** technical evidence, NOT normative Protos conformance, NOT a ratified design decision, NOT a measured throughput or latency speedup.
**Repository:** [guillermomolina/protos](https://github.com/guillermomolina/protos), branch `I115`, inspected remote head [`04c04e998d1c`](https://github.com/guillermomolina/protos/commit/04c04e998d1cefd0e7c55ab335ad704607f652b2). Product `main` incorporation, POM/version/changelog update, tag, and release **are not claimed**.
**Live ownership:** [I115 #931](https://github.com/guillermomolina/protos/issues/931) (implementation), [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930) (architectural experiment), [PERF039 #858](https://github.com/guillermomolina/protos/issues/858) (peer workload definitions), [PERF041 #863](https://github.com/guillermomolina/protos/issues/863) (Boolean control measurements). This slice is accepted by the maintainer; parent work items remain open.

## 1. Comparable source programs (same algorithm, not identical language protocols)

**PERF039: one-iteration loop, expected result 1.**

~~~protos
run: () => {
    i: 0
    (() => i < 1).whileTrue() {
        i = i + 1
    }
    i
}
~~~

~~~python
def run():
    i = 0
    while i < 1:
        i = i + 1
    return i
~~~

Both compute the same answer, check the condition before and after the body, and perform one increment. They are **not** protocol-equivalent: Protos specifies ordinary overridable message lookup and actual Closure activation; Python uses a built-in `while` statement.

**PERF041: selected Boolean callback, expected result 1.** The actual current benchmark-shaped Protos source is:

~~~protos
run: () => {
    value: true
    result: null
    value.ifTrue() {
        result = 1
    }
    result
}
~~~

The Python counterpart is ordinary local assignment followed by `if value: result = 1`. Both execute the selected branch and return 1, but Protos `ifTrue` is an ordinary message with a callback Closure; Python `if` is grammar and uses different protocols. Do not infer general semantic interchangeability.

## 2. What was actually gained

| Measurement | Before / reference | Current experimental Protos | Delta and interpretation |
| --- | ---: | ---: | --- |
| PERF039 Protos initial DSL FULL (all roots) | 1,181 = module 16 + run 750 + condition 199 + body 216 | **26 = module 11 + run 15**, 2 roots | **−97.8%** versus initial Protos case |
| PERF039 matched GraalPy 25.4.4.1.1 DSL FULL | **28 = module 12 + run 16** | **26** | **−2 (−7.1%)**; same loop algorithm |
| PERF041 matched GraalPy 25.4.4.1.1 DSL FULL | **25 = module 12 + run 13** | **24 = module 11 + run 13** | **−1 (−4.0%)**; same Boolean-branch algorithm |
| PERF041 historical Protos DSL count reported during investigation | 277 (not independently re-audited in this checkpoint) | **24** | Approximately **−91.3%**, historical reference only |

**PERF039 I115-M structural generalization**, with I115-N still present: base/different-variable 26 FULL; two variables 32; two consecutive loops 35; nested loops 41; integer-only arithmetic 27; consumed `whileTrue` 31; zero iterations 31. All reported correctness PASS; admitted roots have two physical roots. Whole-root negative admission case PASS without mixing compact operations.

**PERF041 I115-N structural generalization:** main case 24 FULL; `true.ifTrue` 24; `false.ifFalse` 25; `ifTrueIfFalse(true)` 26; `ifTrueIfFalse(false)` 34; computed condition 28; two consecutive `ifTrue` 33; `ifTrue` nested inside `whileTrue` 36; consumed selected `ifTrue` 30; discarded `ifTrue` 26. All reported correctness PASS, and the negative-callback case returns through the whole general lowerer.

**Maintainer validation:** the reported combined focal run is **24 tests, 0 failures, 0 errors, 0 skipped, BUILD SUCCESS**; the maintainer additionally reports local tests passing and `git diff --check` clean. These are maintainer-supplied observations, not connector-executed tests or an attestation of a clean, released product SHA.

### PERF039 JVM Tier 1 and Tier 2 (selected real `run` Closure)

The maintainer independently captured `run` in fresh JVMs with explicit `compile(false)` and `compile(true)`, one matched BGV per tier, same stable root selector `protos-root:5d94dacfc4213945`, `GRAPH_CAPTURE=READY`, `CORRECTNESS=PASS`, and `DSL_FULL_TOTAL=26`:

| Compiler trace measurement | Protos Tier 1 | Protos Tier 2 |
| --- | ---: | ---: |
| Graal `opt done` `IR` first / second value | **103 / 107** | **46 / 43** |
| `CodeSize` | **382** | **168** |
| Compilation ID | 3645 | 3648 |
| BGV files attributed to selected root | 1 | 1 |

Earlier conversation reported equivalent GraalPy `opt done` pairs of **181/264** (Tier 1) and **73/62** (Tier 2), corresponding to comparison of **103 vs 181** and **46 vs 73** for the first IR counter. **Provenance limit:** the peer raw logs/BGV and their hashes were not re-read in this evidence transaction; treat peer numbers as previously reported, not newly verified from artifacts. Do not relabel either of Graal's `opt done` counters as an independently counted **`After TruffleTier` StructuredGraph node count**. Compilation timings/CodeSize are not guest throughput.

## 3. How the reductions were achieved — this compact-control experiment only

| Step | Change and physical effect |
| --- | --- |
| **I115-I** | Replace prepared `whileTrue` callback dispatch for admitted literal condition/body with Bytecode DSL `beginWhile/endWhile`; eliminate callback roots from the admitted loop, bringing the workload from four roots to two. |
| **I115-J** | Direct Integer comparison and addition operations with primitive Java `boolean` for branch conditions; no generic numeric sends in the admitted lane. |
| **I115-K** | Use root `BytecodeLocal` to hold the loop-carried variable, dropping materialized context/frame dispatch. The first version was pattern-specific. |
| **I115-L** | Fuse the module's single Closure binding creation and return so the module costs 11 FULL. |
| **I115-M** | Replace benchmark-shape recognition with **whole-root admission**: small Integers, root binding identity, arithmetic `+/-/*`, ordering `</<=/>/>=`, nested/consecutive `whileTrue` and recursive sequences. Admission is all-or-nothing: unadmitted roots keep the general lowerer. Results of consumed loops store the last body result or null for zero iterations; discarded loops do not need a result local. |
| **I115-N** | Extend the same compact root with Boolean/null literal kinds and identity-keyed abstract binding kinds, branch merges and loop-head fixpoint. Lower `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `if`, `else` with DSL `IfThen/IfThenElse`; emit comparisons directly as Java booleans, otherwise use `ExperimentalCanonicalBooleanCondition`. Inline admitted callback bodies; materialize a Boolean-control result local only when consumed. `not/and/or` remain on the general route. |

Representative inspected product surfaces at branch `04c04e998d1c`: `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java`, `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`, `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI115MCompactLoweringTest.java`, `ProtosI115NCompactBooleanTest.java`, `ProtosPerf039WhileOnceCensusTest.java`.

The **physical win** is not merely constant-folding a fixed benchmark: the recorded positive cases vary variable names, literal values, number of bindings, number/depth of loops and Boolean control; negative whole-root tests demonstrate isolation of the experimental path. This does **not** establish semantic equivalence beyond the tested subset.

## 4. Exact semantic guarantees this experiment deliberately bypasses

**Only the new I115-I/J/K/L/M/N shortcuts are listed here.** This is NOT a restatement of prior I115-H static-call/ABI/Closure-creation deviations.

1. **Dynamic numeric sends are replaced by intrinsic operations (I115-J/M).** Standard Protos `+/-/*` and numeric comparison operators retain ordinary message lookup, override/shadowing, receiver validation and guarded-send semantics. The experimental operations dispatch only on the source selector and admitted integer-kind facts: overridden numeric methods, late mutations, lookup consequences and generic numeric-family handling can be skipped. Conformance risk is deliberate, not an inferred safe specialization.

2. **The Integer domain is narrowed to machine `long` (I115-J/M).** `Math.addExact/subtractExact/multiplyExact` and `experimentalInteger(...).longValue()` do not perform the required exact `SmallInteger` to `BigInteger` promotion and arbitrary-precision math. Crossing the signed-64 boundary may throw a host exception rather than produce a valid Protos `BigInteger`; larger integer carriers/families and cross-family standard arithmetic/ordering are not implemented in this direct lane. This is a concrete numeric contract violation, not just a missed optimization.

3. **Guest lexical context/binding authority is bypassed (I115-K/M/N).** For admitted root bindings, values live directly in `BytecodeLocal`; the persistent guest `Context` slot establishment/read/write, reflection, removal/rebinding/topology and alias/captured-context observability are not preserved in general. Static `CanonicalBindingIdentity` plus a pre-admission kind map is not proof of runtime presence or context authority. Captured lookups/assignments in inlined callbacks operate on the physical root's locals, not ordinary lexical context objects.

4. **`whileTrue` ordinary lookup and callback activation disappear (I115-I/M).** `condition.whileTrue(body)` is a normal overridable message with selected Closure receiver/body, evaluated Closure objects, validations and a fresh semantic callback activation per test/body invocation. The compact `While` bypasses dispatch, selection/override, materialized callback identity, activation/capture/return-home and complete failure/control-transfer semantics. Explicit suspension/resumption, nonlocal return, cooperative cancellation and structured effects are not proven equivalent. One-iteration and zero-iteration normal results are tested, but do not discharge these guarantees.

5. **Boolean messages become source-selector control flow (I115-N).** The ordinary overridable `ifTrue/ifFalse/ifTrueIfFalse/if/else` sends are replaced by `IfThen/IfThenElse` and native Boolean conditions. This suppresses ordinary message selection/overrides, evaluation/materialization and observable identity of the literal callback Closure(s), ordinary selected-callback invocation/activation, validation/error timing, return-home and possible suspension/control-transfer behavior. The selected/unselected **normal results** are preserved by tests for the admitted benchmark subset; that is narrower than Protos's complete protocol contract.

6. **Incorrect admission/error boundaries may manifest host internals (I115-N).** `ExperimentalCanonicalBooleanCondition` accepts `ProtosBooleanValue`, but non-Boolean input reaches `CompilerDirectives.shouldNotReachHere` rather than the language's ordinary error/dispatch behavior. Static abstract kinds and merges do not themselves enforce the guest's dynamically observable object/binding protocols.

**What is not a violation by itself:** emitting `beginWhile` or `IfThen`, fewer DSL instructions, primitive carriers, source-section tagging or a whole-root admission pass. A conforming later implementation could retain these representations while restoring message/Context/Closure/numeric semantics. This experiment **has not yet done so**.

**Normative owners:** `spec/semantics/EXECUTION_AND_CONTROL.md` (loop validation, activation, continuation and results), `spec/semantics/CALLABLES.md` (ordinary invocation, capture), `spec/semantics/VALUES_AND_COLLECTIONS.md` (Boolean dispatch and exact unbounded numeric model), and `spec/semantics/OBJECT_MODEL.md` (ordinary lookup/context/slot semantics). No normative spec change is authorized or made.

## 5. Disposition and next constraints

- **Experimental slice I115-I/N: maintainer accepted/closed as a bounded investigation checkpoint.** Its positive/negative tests and confirmed DSL structure are real local observations; release/product integration is **not** implied.
- **I115 #931 remains open**, as do **PLAT066-C #930**, **PERF041 #863** and the separately owned **PERF039 #858** benchmark-workload issue. No broader PLAT066 or D013 ratification.
- A future production path must restore exact protocol selection, guest Context observability, callback activation/ReturnHome/suspension and arbitrary-precision numeric behavior before normative conformance can be claimed.
- Preserve the exact chosen comparators, compiler tier, `run` root identity, workload source and BGV provenance; don't claim compiled-node parity from DSL FULL equality, an `opt done` counter or a passing functional suite.
