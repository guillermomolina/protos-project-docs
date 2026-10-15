# PERF040-D — Owner-authorized reconciliation of numeric DSL fallback

**Date:** 2026-10-15. **Formal owner:** [PERF040/#862](https://github.com/guillermomolina/protos/issues/862), OPEN/READY. **Implementation repository:** `guillermomolina/protos`. **Source baseline:** [Protos `df4142919d0222635ab8f9f8c54e7a1966e6edaa`](https://github.com/guillermomolina/protos/commit/df4142919d0222635ab8f9f8c54e7a1966e6edaa), `0.3.357-SNAPSHOT`. **Benchmark harness published main baseline (not independently proven as the user's local producer):** `guillermomolina/protos-benchmarks@1b28b390b61d5a59e9b1573ea30e26306a6dde86`.

**Disposition:** The owner explicitly requested immediate implementation of the GraalPy/Protos fallback reconciliation and authorized PERF040 coordination and durable evidence publication. **PERF040-D is a bounded implementation slice of PERF040**, not a separately schedulable Issue. The implementation itself has **NOT** been performed or validated by this evidence publication; product `main` is unchanged by this record. No new PLAT decision, new optimizer architecture or language-semantic change is approved.

## Exact source finding

Protos `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`, `TryPrimitiveNumericSendOne` (~L6794–L7066), has numeric fast-path specializations (`canonical`, `canonicalMixedInteger`, `canonicalFloatDD`, `canonicalFloatDL`, `canonicalIntegerOrdering`, `canonicalFloatOrdering`) followed by:

```java
@Specialization(replaces = {
    "canonical", "canonicalMixedInteger", "canonicalFloatDD",
    "canonicalFloatDL", "canonicalIntegerOrdering", "canonicalFloatOrdering"
})
public static Object generic(/* operands */) {
    return MISS;
}
```

By Truffle DSL's documented `@Specialization.replaces` contract, **if** this generic specialization becomes instantiated for an admitted negative input, the replaced specializations can be removed from that node instance and not reinstantiated. The corresponding GraalPy `PyNumberAddNode` instead declares the generic number protocol as `@Fallback` plus `@InliningCutoff`, preserving numeric specializations. This is a **source-level architectural difference with a concrete, falsifiable same-site polymorphism risk**, **not** proof that an existing user benchmark entered the generic specialization or that it caused any measured slowdown.

GraalPy, pinned source `oracle/graalpython@a099d006438942d54c1856c2cce7b6c2839b0c85`:

- [`PyNumberAddNode.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/PyNumberAddNode.java): `@Fallback @InliningCutoff` delegates to numeric slots `NB_ADD`, including Python `__add__`/`__radd__`, without unconditionally replacing `doII`/`doLL`.
- [`PyNumberAddFastPathsBase.java`](https://github.com/oracle/graalpython/blob/a099d006438942d54c1856c2cce7b6c2839b0c85/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/lib/fastpath/PyNumberAddFastPathsBase.java): `doII` and `doLL` use `Math.addExact`; `doLLOvf` promotes exact overflow to `BigInteger`; Float operands are separately specialized.
- Truffle DSL [`Specialization.java`](https://github.com/oracle/graal/blob/f74f0e65eeee38bae7979f961cc6285597fb21a5/truffle/src/com.oracle.truffle.api.dsl/src/com/oracle/truffle/api/dsl/Specialization.java) and [`Fallback.java`](https://github.com/oracle/graal/blob/f74f0e65eeee38bae7979f961cc6285597fb21a5/truffle/src/com.oracle.truffle.api.dsl/src/com/oracle/truffle/api/dsl/Fallback.java): replacement removes earlier specializations for that node; a fallback applies when other specializations do not.

Protos `CanonicalToBytecodeLowerer.java` (~L8870–L8890) emits `TryPrimitiveNumericSendOne` and branches on `IsPrimitiveNumericSendMiss` into the existing dynamic `TryDirectSendOne`/`PrepareSendOne` paths. `ProtosPrimitiveNumericSendFastPathsBase.IntegerArithmeticNode` already performs primitive `Math.addExact` and exact overflow respecialization. **Do not replace D013 fallback with primitive arithmetic**: the selected closure, legitimate dynamic selection and actual operand semantics remain authoritative.

## 2026-10-15 — Truffle DSL negative-cache constraint and exact patch

Further source verification of `oracle/graal@f74f0e65eeee38bae7979f961cc6285597fb21a5` identified a stricter constraint than the initial comparison:

- `NodeParser.java` rejects negating guards bound to `@Cached` values for a `@Fallback` when the specialization may have multiple cached instances (`limit = "3"`). All six existing Protos numeric specializations have that shape. The processor error says, in part, that guards cannot be negated for `@Fallback` if they bind cache parameters and permit multiple instances.
- The same processor emits `truffle-assumption` warnings for assumption-based specializations reaching `@Fallback`, because invalidation removes their instances and can unexpectedly enter fallback; Truffle's changelog GR-43903 confirms that behavior.
- Consequently a literal GraalPy-style `@Fallback` annotation is **not** the correct bounded drop-in edit for this Protos operation.
- **Mandatory patch:** the declaration of `generic` changes from `@Specialization(replaces = {...six names...})` to plain `@Specialization`; its body continues to return `MISS`. No numeric specialization, `@Cached`, limit or `assumptions` is changed. The Truffle DSL `@Specialization.replaces` contract then no longer removes existing fast-path specializations on instantiation of `generic`.
- **Honest scope:** a generic specialization installed before any numeric fast specialization can be a different JIT state; this bounded change only guarantees the source-level no-eviction property for *previously installed* numeric specializations. Do not claim fast-path restoration after a generic-first call without separate structural evidence.
- **Agent is an implementer, not a researcher:** no internet, other repositories, alternative implementation experiments, Truffle design choices, or investigation gate is delegated. Implement exact annotation replacement, add preselected same-site semantics tests (fast → negative → fast for Integer and ordering), and provide a nonexecuting diff for the human to validate. Human runs all validation and product publication.

Oracle references already researched: [NodeParser.java](https://github.com/oracle/graal/blob/f74f0e65eeee38bae7979f961cc6285597fb21a5/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/parser/NodeParser.java) and [DSLWarnings.md](https://github.com/oracle/graal/blob/f74f0e65eeee38bae7979f961cc6285597fb21a5/truffle/docs/DSLWarnings.md).

## Authorized PERF040-D implementation, one bounded slice

1. **Exact implementation decision, no alternatives delegated:** retain `@Specialization` on `TryPrimitiveNumericSendOne.generic`, but **delete its entire `replaces = {...}` attribute** so that the annotation is simply `@Specialization`. Leave the method name, signature, body (`return MISS`), all six cached numeric specializations, their `limit = "3"`, and their `assumptions` unchanged. Do **not** use `@Fallback` here and do not migrate Assumptions into ordinary guards. Preserve the existing `MISS` branch and semantic dynamic-send fallback. The bounded goal is **not evicting previously installed numeric specializations** after a non-admitted invocation, not promising a newly admitted fast path after generic-first execution.
2. Cover the entire operation's existing arithmetic **and ordering** specializations in one patch; do not create a second numeric subsystem, duplicate `+` handling, or split trivial file edits into multiple publication slices. PERF046/#890 remains the separate comparison-family performance owner, not a second owner of this shared fallback fix.
3. Add focused same-call-site transitions: small Integer -> non-admitted value -> small Integer, and alternating Integer/Float/orderings as the language actually admits; include standard canonical selection versus a truly legal dynamic override, different Context/Prelude, and exact overflow. Prove the **static no-eviction contract** by validating that `generic` no longer declares `replaces`; the functional tests demonstrate fast → negative → fast semantic conformance but cannot alone prove physical re-specialization or peak latency. Keep a monomorphic small-Integer control.
4. Ensure whole-program correctness: selected D013 message, evaluation order, error and fallback identity, guest effects, normal/DEBUG, Context isolation, reparse, Actor/Task/ReturnHome semantics where implicated, guest materialization and large-number normalization. The frozen canonical Core invariant from PLAT059 must remain intact: no hypothetical override fallback for proven exact canonical immutable Core expressions.
5. **Before editing**, consume current `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, and the governing `spec/` semantic owners at the then-current local HEAD. Preserve concurrent changes. Agent prepares/edits source and tests; **human-executor alone** runs compilation, tests, git diff checks and product commit/push. Do not change `pom.xml`/`CHANGELOG.md` before tests are green; no tests after version/changelog finalization.
6. After a successful human-run validation and publication, retain exact-revision test/source or graph evidence; do not assert speedup, graph shrinkage or runtime deoptimization cause from the source comparison alone.

## Historical measurements: distinct from acceptance of the new patch

The owner reported a Protos-only, clean-checkout, 17-workload canonical reference round on `df414291`: **17/17 correctness PASS, 10/17 stable/valid, 7/17 NOT_STABLE**. `primitive-integer-add` was valid at **284.611 ns/call** (`left: 1; right: 2; left + right`); `primitive-if-true` and `primitive-if-false` were not stable. These results are user-provided terminal summaries, not connector-retrieved raw `metadata.json` or proof of any fallback transition. The owner separately reported `git diff --check` clean and all tests passing locally for the existing product state; **no PERF040-D code/tests have been executed**.

**Coordination:** [PERF040/#862](https://github.com/guillermomolina/protos/issues/862) remains OPEN/READY until its own independent performance acceptance; [I101/#892](https://github.com/guillermomolina/protos/issues/892) owns minimal proven-normal lowering/constant folding; [PERF046/#890](https://github.com/guillermomolina/protos/issues/890) owns wider numeric comparison performance. PLAT056/#866, I091/#869, I090/#868 and PLAT059/#894 are closed decisions/implementations and are not reopened.
