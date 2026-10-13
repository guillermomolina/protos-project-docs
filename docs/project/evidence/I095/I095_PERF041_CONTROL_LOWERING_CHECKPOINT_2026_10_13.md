# I095 — PERF041 explicit-control lowering handoff and A–E experimental checkpoint

**Recorded:** 2026-10-13 (Europe/Madrid)  
**Canonical coordination:** [I095/#878](https://github.com/guillermomolina/protos/issues/878)  
**Causal investigation:** [PERF041/#863](https://github.com/guillermomolina/protos/issues/863)  
**Prior implemented foundation:** [I092/#870](https://github.com/guillermomolina/protos/issues/870), closed by owner, not reopened  
**Product checkpoint:** [`guillermomolina/protos@1416302695bdcbf00145778ef1eae036f5ab8a18`](https://github.com/guillermomolina/protos/commit/1416302695bdcbf00145778ef1eae036f5ab8a18)  
**Checkpoint parent:** `18f71b2a664c56ac808e2f508b6369984de02280`  
**Branch:** `experiment/perf041-iftrue-abcd-e1-20261012` (non-main; exact revision above is authoritative)  
**Normative changes:** NONE.

## Distinct evidence levels

1. **Published GitHub fact:** the checkpoint commit exists and contains exactly the following six paths:
   - `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
   - `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
   - `src/main/java/com/guillermomolina/protos/execution/ProtosCanonicalBooleanOneNode.java`
   - `src/main/java/com/guillermomolina/protos/execution/ProtosNativeIfTrueCallbackNode.java`
   - `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
   - `src/test/java/com/guillermomolina/protos/execution/ProtosI092CompactBooleanTest.java`
2. **Maintainer-reported local validation before the commit:** `git diff --check` clean; `mvn -q -DskipTests compile` PASS after A+B+C+D1+D2+E1. These are not full-suite results.
3. **Not yet established:** full `make test` after E1, final `After TruffleTier` graph measurements, machine-code-size comparison, no regression, and performance benefit. Historical graph counts are not attributable to the checkpoint and must never be presented as measured outcomes of it.

## What A–E preserves and what it changes

- A: Bytecode DSL operation proxy for canonical Boolean `One`, introducing `ProtosCanonicalBooleanOneNode`.
- B: specialized cached Boolean and callback operations; new `ProtosNativeIfTrueCallbackNode`.
- C: lowers the B′ generic-miss condition once; only one inline callback region remains at the literal site. Redundant local storage might itself increase the graph and requires A/B measurement.
- D1/D2: primitive `int` classification (`MISS=0, FALSE=1, TRUE=2`); callback preparation only after TRUE, including spread and frame-native cases; the ordinary/default send path is reconciled.
- E1: use the literal's frozen plan when it is provably the effective plan, with the authoritative context-/task-owned resolution retained for other cases; eliminate a subsequent admission check already implied by guards.
- Crucially, the checkpoint still stages a `ProtosClosureValue` and, on TRUE, can create a `PreparedInlineLiteralCall` and compact argument array. These source facts are **not proof** that an allocation survives partial evaluation or executes on the hot path.

## Corrected conceptual conclusion

Both `1 + 1` and `true.ifTrue() { 1 }` are ordinary Protos message sends. The compiler may recognize canonical behavior and lower them physically to primitive operations or control, without changing their guest-visible semantics. The supplied `{ 1 }` is semantically a Closure value, but for a nonescaping, observationally equivalent admitted inline case it does **not** imply a mandatory physical Closure allocation or Closure invocation.

A capable Smalltalk compiler can reduce a provably canonical `true.ifTrue() { 1 }` to the result `1`. That is the *target shape*, not a claim that the current Protos checkpoint already performs the fold or that identity/instrumentation obligations can always be discarded.

**Do not ask Graal to rediscover language-level control semantics from generic call machinery.** Explicitly lower admitted control constructs; then let Graal optimize primitive branches/results. However, retain the A–E specialization family for nontrivial canonical/dynamic cases. Constant folding and dynamic specialization are complementary layers, not alternatives.

## Concrete peer Java source evidence

- [GraalPy `RootNodeCompiler.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java): `visit(StmtTy.If)` emits Bytecode DSL `beginIfThen` or `beginIfThenElse` directly; `visitCondition` avoids generic `Yes` conversion when condition is statically known Boolean. This is explicit frontend control lowering.
- [GraalPy `PyObjectIsTrueNode.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/PyObjectIsTrueNode.java): primitive boolean specialization, `@OperationProxy.Proxyable`, `@GenerateInline(false)`, `@InliningCutoff` on complex generic branch. Protos has strict canonical Boolean, not Python truthiness; copy architectural patterns, **not** truthiness semantics.
- [GraalPy `CallUnaryMethodNode.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java): guarded cached function/root/target and optional `@ForceQuickening`, separately cut-off megamorphic fallback. Borrow only where compatible with Protos' call selection and context/Actor ownership; never cache fresh Closure identity incorrectly.
- [GraalJS `GraalJSTranslator.java`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js.parser/src/com/oracle/truffle/js/parser/GraalJSTranslator.java): source `if` becomes `factory.createIf(test,pass,fail)`.
- [GraalJS `IfNode.java`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/IfNode.java): explicit `JSNotNode` inversion into swapped branches, boolean-specialized node, generic truth conversion and resumable generator form. No proof from this source that `IfNode.create` itself folds all `if(true)`; do not claim GraalJS has a proven frontend literal-fold at that exact location.

## Normative preservation and proof obligations

- [`spec/semantics/VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/main/spec/semantics/VALUES_AND_COLLECTIONS.md), §16: `ifTrue` is **ordinary overridable message lookup**, not a parser-defined unconditional intrinsic; the original receiver must be exactly canonical Boolean when standard behavior is selected; fixed arity; receiver and argument expression evaluation before call in correct order; selected callback invoked once with zero args; unselected callback not called or validated; false returns canonical null.
- `spec/semantics/CALLABLES.md`: immediate trailing Closure is semantically an ordinary Closure with lexical capture, preserved receiver/methodHome/return home and nonlocal-return behavior. A physical elimination needs a proof that no identity, authority, debug/tooling or observer can distinguish it; do not just bypass `MaterializeCurrentClosure` globally.
- `spec/semantics/EXECUTION_AND_CONTROL.md`: Sequence results and home/control scope remain exact across inlining, exceptions, continuation and cancellation.
- PLAT040 F′ / PLAT043 / PLAT044 B′ / root Object frozen publication (D049) remain governing approved implementation boundaries.
- No spec edits, no `ifTrue` selector-only semantic exception, no second model of messages, no extension to `whileTrue` or numeric work.

## Recommended next implementation and evaluation

1. **I095-A**: inspect only the current checked-out experimental branch and existing structural tests; add a guarded compiler-side *constant canonical receiver* route for general semantically safe zero-argument literal bodies (not a special case for integer `1`). Reuse the existing B′ emission framework and debugging boundaries; retain all original fallback and side-effect behavior when proof is not complete.
2. **I095-B**: refine dynamic canonical Boolean lowering where the current Bytecode DSL graph still includes unneeded carrier/control structures, using GraalPy/GraalJS idioms (specialization and cold fallback) without broad Java redesign.
3. **I095-C**: owner-executed focal and full-suite validation, followed by causally matched compiler graph and code-size comparison: baseline `18f71b2`, A–E checkpoint `1416302`, and new candidate; workloads `primitive-if-true`, `primitive-if-false`, `primitive-return-literal`, plus nontrivial dynamic conditions. Keep harness policy fixed; cached GraalPy/GraalJS graph results may be reused if comparable. Explicitly separate BGV node totals, `TraceCompilation` IR, cold paths and machine-code size.

**Publication rule:** no product commit/push by the agent. The experimental branch is a transportable checkpoint, not accepted final product. `pom.xml` and `CHANGELOG.md` change only after the human reports relevant tests green, immediately before human commit/push; no tests after those two files change.

```text
DOCUMENT_KIND=HISTORICAL_EVIDENCE_AND_IMPLEMENTATION_HANDOFF
IMPLEMENTATION_OWNER=I095
EXPERIMENT_PRODUCT_REVISION=1416302695bdcbf00145778ef1eae036f5ab8a18
EXPERIMENT_BRANCH=experiment/perf041-iftrue-abcd-e1-20261012
MAINTAINER_COMPILE=PASS_REPORTED
MAINTAINER_DIFF_CHECK=PASS_REPORTED
FULL_TEST_SUITE=NOT_REPORTED
NEW_FINAL_BGV=NOT_MEASURED
NEW_MACHINE_CODE_COMPARISON=NOT_MEASURED
GRAALPY_GRAALJS_APPROACH=REUSE_EXPLICIT_COMPILER_PATTERNS
A_TO_E_DYNAMIC_INFRASTRUCTURE=RETAIN_AS_EXPERIMENTAL_CANDIDATE
SEMANTIC_CHANGE=NONE
PRODUCT_MAIN_PUBLICATION=NONE
```
