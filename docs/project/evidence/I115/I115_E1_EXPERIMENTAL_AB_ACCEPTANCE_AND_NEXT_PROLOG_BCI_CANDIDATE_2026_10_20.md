# I115-E1 — experimental branch A/B acceptance and next non-E2 candidate

Date: 2026-10-20
Formal references: [I115/#931](https://github.com/guillermomolina/protos/issues/931), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924), [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).
**State at publication:** implementation committed to the **experimental** branch; maintainer reports targeted acceptance and A/B evidence; not integrated into product `main`, no integrated suite or post-merge acceptance has been reported. The issue I115/#931 remains closed for the earlier C/D publication; recording E1 must not silently reopen it.

## 1. Verified GitHub artifacts and provenance

- **Public product base:** [`guillermomolina/protos@e2200088d42d0d8be90626a79b373ddad64978f5`](https://github.com/guillermomolina/protos/commit/e2200088d42d0d8be90626a79b373ddad64978f5) (`main` at read time; I115-C/D / `0.3.385-SNAPSHOT`).
- **Experimental branch:** `experimental/i115`, head [`61b94c874c80ed88259051146d00afc0ca14be54`](https://github.com/guillermomolina/protos/commit/61b94c874c80ed88259051146d00afc0ca14be54) (committed 2026-10-20T02:29:37Z), message `I115-E1: defer lexical authority and eliminate eager root installation`.
- GitHub compare shows the experimental branch is **exactly one commit ahead, zero behind** the baseline, with six changed files. This verifies that E1 code has been published on the *experimental* branch; it is **not** proof of an E1 merge into `main`.
- The maintainer supplied the A/B counts, focal return codes, debugger/DAP, future suspension and graph breakdown described below. These measurements are **human-reported**; the full original graph binaries, command logs and exact post-test local worktree diff were not independently re-executed here. GitHub source commit and paths were independently inspected.

## 2. Source-verified implementation in experimental/i115

Six paths changed relative to `e2200088`:

1. `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`: removes the persistent authority fields/registration from `ProtosBytecodeRootNode`; `EnterRoot` remains an `@Prolog` with a no-argument empty `perform()` specialization. Truffle root physical instruction shape stays present.
2. `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosFrameLocalBindings.java`: `promoteLiveFrameLexicalAuthority` invokes the existing `installFrameLexicalAuthorityOnTransition` mechanism with `liveFrame.materialize()` only when required. Existing `createCurrentFrameBinding` retains its transition path; a `VirtualFrame` is not stored into a durable escaping object.
3. `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java`: stops recording the eager prologue frame authority; retains the frame-native layout/range and current-binding operations.
4. `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalCompilationFacts.java`: updates implementation-level documentation to distinguish needs for durable lexical state from eager installation.
5. `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025FrameMaterializationSliceTest.java`: adds capture/persistence/root handoff/exception regressions.
6. `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java`: adds genuine `Future.value()` suspension, resumption and non-repeat effects regression.

The experimental branch contains **no new `@Operation`**, retains a single optimized Bytecode DSL interpreter and preserves D013/D179 contracts per the intended patch. The source diff supports the change mechanism; semantic acceptance is grounded in maintainer-reported focused test results, not independent execution.

## 3. Maintainer-reported matched A/B: GraalVM 25.4 Tier 1

Program (same in both Protos variants):

```protos
identity: (value) => { value }
identity(1)
```

The two columns below are the **same `after truffleTier` expansion-tree counting method**, as reported by the maintainer for the original `e2200088d` source versus I115-E1:

| Independently compiled root | A: e2200088d | B: experimental I115-E1 | Change |
| --- | ---: | ---: | ---: |
| TOP_LEVEL | 648 | 448 | −200 (−30.86%) |
| identity Closure | 263 | 106 | −157 (−59.70%) |
| **Sum of independent roots** | **911** | **554** | **−357 (−39.19%)** |

TOP_LEVEL separately reported `opt done` metrics:

| Metric | Before | After |
| --- | ---: | ---: |
| after truffleTier expansion-tree nodes | 648 | 448 |
| `opt done` IR | 645 | 446 |
| `opt done` machine code | 15,584 bytes | 6,081 bytes |

The module machine-code change is about **−61.0%**. This is neither a throughput/latency speedup measurement nor the same metric as 448 expansion-tree nodes.

**Important baseline clarification:** the prior PERF042 report contains 645 + 263 = 908 *selected JSON graph* nodes. The exact matched E1 A/B experiment contains 648 + 263 = 911 *expansion-tree* nodes. Do not silently overwrite the historic 645/908 with 648/911 as if one previous measurement were false; they belong to different counting conventions. For subsequent *same-convention expansion-tree* work use 448/106/554. Use 446 if reproducing the post-E1 `opt done` IR, never interchange these metrics.

## 4. Attribution reported in I115-E1 acceptance

Before E1:
- TOP_LEVEL `EnterRoot`: 96 exclusive attributed nodes (84 + 12).
- identity `EnterRoot`: 102 exclusive attributed nodes.
- Total exclusive `EnterRoot` attribution: 198.

After E1:
- `EnterRoot`: **0 individually attributed** nodes for both roots.
- TOP_LEVEL `CreateCurrentFrameLocal`: 6 exclusive nodes (unchanged).
- TOP_LEVEL `ReadRootFrameLocal`: 3 exclusive nodes.
- TOP_LEVEL `InvokeClosureOneTail`: 19 exclusive nodes.
- TOP_LEVEL `MaterializeCurrentClosure`: 91 exclusive nodes.
- TOP_LEVEL Root non-child attribution: 180 nodes.
- TOP_LEVEL generated `CachedBytecodeNode`: 132 nodes.

**Interpretation:** The matched whole-tree reduction of 357 nodes is real *in the reported matched graph metric* and exceeds the 198 exclusive nodes once attributed directly to `EnterRoot`. It is not evidence that 357 nodes are directly owned by that handler. Multiple downstream inline/guard/allocation/state effects can disappear when the prologue cost is removed. A/B reduction is not merely a 198-node ownership transfer. Per-component exclusive counts are **not independently removable instruction budgets**.

## 5. Maintainer-reported acceptance and remaining gate

- `I115_E1_CORRECTION_RC=0`
- `I115_E1_SUSPENSION_RC=0`
- `I115_E1_FINAL_FOCAL_RC=0`
- `git diff --check` clean
- Reference binding survival after a module return; capture by reference; cross-root handoff; exceptional exit.
- Real `Future.value()` Task suspension and resume without repeated effects.
- Debugger attached after initial execution and real DAP local observation.
- Exact **11 module + 5 identity = 16 FULL** Bytecode DSL instructions, with identical census when debugger is attached; no extra guest source root.
- No integrated `make test` or full-suite result for E1 is available yet: **integration gate pending**.
- Commit is in experimental branch, not main.

## 6. Physical-convergence implications

- E1 modifies root lexical authority, **not** `InvokeClosureOneTail`. Its broad node reduction invalidates attempts to attribute the former root-entry cost to the call specialization.
- The new **post-E1 experimental expansion-tree checkpoint** is 448 TOP_LEVEL / 106 identity / total 554 (all Tier 1).
- The older GraalPy Native reference 469 module / 110 identity was captured with different runtime/version and BGV methods. The apparent lower post-E1 Protos totals do **not** establish equivalent speed or execution quality across JVM vs Native Image.
- Continue separate measurement of both independent roots, preserving no-inlining conditions and full 16 DSL instructions.
- E2-A is a **separate** published *edit-only plan* targeting Closure metadata `Optional` wrappers; do not mark E2-A implemented as a consequence of this E1 result.

## 7. Next non-E2 candidate: empty Prolog bytecode-index maintenance

A small, source-verified **investigation/experimental implementation candidate**, **not yet performed/approved as a formal new issue**, is to change only the empty E1 prolog annotation:

```java
// experimental/i115 today
@Prolog
public static final class EnterRoot {
    @Specialization
    public static void perform() {}
}

// proposed minimal A/B
@Prolog(storeBytecodeIndex = false)
public static final class EnterRoot {
    @Specialization
    public static void perform() {}
}
```

- Source reference, [GraalPy 25.4.4 `PBytecodeDSLRootNode.EnterCalleeContext`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java): uses `@Prolog(storeBytecodeIndex = false)` despite doing more work than Protos's new empty prologue.
- [Truffle `@Prolog` API](https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/Prolog.html): `storeBytecodeIndex` defaults to `true` and the effect is limited to generated configurations that store BCI in frame or enable uncached interpreter. Protos currently enables `enableUncachedInterpreter = true`. The actual impact on Tier-1 compiled graph may be zero; the API does **not** guarantee savings.
- This is a **generated Bytecode DSL bookkeeping hypothesis** motivated by the newly dominant, non-operation-exclusive Root 180 and `CachedBytecodeNode` 132 nodes. Neither category has been traced to BCI maintenance specifically: **do not present the 312 combined nodes as reclaimable**.
- Keep physical `EnterRoot` instruction, 11+5 FULL, one interpreter and on-demand debugger/DAP. Do not remove the `@Prolog`, change RootTag instrumentation, change `EpilogForReturn`, or modify generated source by hand.
- Human-only A/B tests: same Tier-1 expansion-tree and `opt done` counts (module and identity independently), same capture options, focal DAP/exception/suspension and I112 debugger census. If the compiled graph count is unchanged, record no physical win and move on; avoid speculative cascading changes.

For larger gains, map the **Root / `CachedBytecodeNode` IR node ownership** using the full new BGV after E1, including metadata/frame-state and branch edges, before choosing any interpreter-generator intervention. Do not claim root 180 or `CachedBytecodeNode` 132 are directly removable.

**Publication state:** this record publishes evidence, not Protos source, a completed E1 merge, a validated E2 or an approved new I-family issue. No GitHub issue was closed, reopened, or changed in status by this evidence record.
