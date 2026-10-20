# I115 / PLAT066-C — lexical terminal C1 caller emission: matched PERF042 run graph acceptance

**Date:** 2026-10-20. **Classification:** snapshot evidence of an *unpublished working-tree implementation slice* under the existing [I115/#931](https://github.com/guillermomolina/protos/issues/931); not a new I identifier, a product release, a benchmark of elapsed guest execution, or completion of PLAT066-C.
**Research:** [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924), [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).
**Historical matched comparator:** [PERF042 0.3.386 cold run baseline](../PERF042/PERF042_386_COLD_RUN_IDENTITY_FULL_BASELINE_2026_10_20.md), referencing `guillermomolina/protos@607973c493dcb0c5ba3a7eeba9897b045eb049d8` and GraalPy JVM 25.4.4.1.1.
**Current implementation provenance:** the human-provided diff reports modified local `main` files and **no commit/push of this slice**. The latest separately confirmed remote `protos/main` before measurement was `60af4eac59a08d31184d517d064878243cdb0914` (0.3.387 product ancestry); the capture logs do *not* stamp their checkout SHA. Do not upgrade their provenance to a verified product revision.
**Source evidence:** human-uploaded `perf042-plat066c.tar.gz`, SHA-256 `f7cd6721fca9b295e6fc85576bcfb9ede670dc452ec2cd5fa4e112e8a603511c`, retained by the maintainer; binaries are **not checked into this documentation repository**. Individual file hashes below make the report independently checkable against the original archive.

## Implemented mechanism (scope: existing I115)

`CanonicalToBytecodeLowerer.admitsTerminalClosureCall` now admits an eligible terminal one-argument call with a `CanonicalLookup` receiver present in `currentRootBindingCallCandidates` despite the previous I096 inline-literal exclusions. Structural and argument operand gates remain. The existing `emitRootBody → emitDiscardedPrefix → emitTerminalClosureCall` path emits one `c.InvokeClosureOneTail` instead of copying the Closure body plus admission/fallback/continuation into the caller. The callee retains an independent physical root and CallTarget; tiered Graal compilation owns actual inline decisions. Nonterminal calls, immediate literal calls and other arities remain out of scope.

**Changed files in supplied working diff:**
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java` — admission predicate/Javadoc only.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI096InlineClosureLiteralCallTest.java` — structural and semantic regression adaptation.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TerminalCallBudgetTest.java` — three-root physical census.
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosPerf042DefinitionKeyedDirectCallTest.java` — direct binding + alias specialization.
- The separate opt-in diagnostic in `ProtosPerf042IdentityBuilderOracleTest.java` was already locally modified when the slice was edited, and was preserved; it supplies the measured physical `run` root BGV evidence. No product `pom.xml`, `CHANGELOG.md` or normative `spec/` modifications were reported.

## Matched `run → identity(1)` full-bytecode census

```protos
run: () => {
    identity: (value) => { value }
    identity(1)
}
```

The **matched** I115 current-tier diagnostic defines only `run` at module level; the historical cold baseline uses that same three-root source shape. A separate I112 smoke/test appends `run()` as a module expression; its module FULL count differs and must not be included in the matched program total.

| BytecodeConfig.DEFAULT FULL instruction count | Prior Protos 0.3.386 | Measured I115 worktree | GraalPy cold comparator |
| --- | ---: | ---: | ---: |
| Module | 16 | 16 | 12 |
| `run` | **112** | **18** | 12 |
| `identity` | 5 | 5 | 5 |
| Descriptive sum of independent physical roots | 133 | 39 | 29 |

`run`: **94 fewer FULL instructions (−83.93%)**. The 18 emitted physical operations, in order:

```text
00 c.EnterRoot
01 c.CurrentActivation
02 c.InstallFrameLexicalAuthority
03 load.constant
04 load.constant
05 c.MaterializeCurrentClosure
06 store.local
07 c.CurrentActivation
08 load.constant
09 load.local
10 c.CreateCurrentIndexedLocalSlot
11 store.local
12 clear.local
13 c.ReadRootFrameLocal
14 load.constant
15 c.InvokeClosureOneTail
16 c.EpilogForReturn
17 return
```

`identity` remains its own five-operation root: `c.EnterRoot`, `c.BindUnaryFrameParameter`, `c.ReadRootFrameLocal`, `c.EpilogForReturn`, `return`. There is exactly one `InvokeClosureOneTail` in `run`, no copied inline callee body and no extra source roots.

**Independent I112 executed-module fixture:** `I112_CASE=lexical-c1-normal`, root FULL `10+18+5=33`, `I112_EXTRA_GROUP_ROOTS=0`, result 1, with `PERF042_RUN_PHYSICAL_CALL=PASS`, `PERF042_RUN_INLINE_BODY_IN_LOWERER=ABSENT`, `PERF042_RUN_TERMINAL_OPERATION=InvokeClosureOneTail`. The **different** module FULL=10 reflects `run()` in the I112 program; never substitute 33 for matched 39.

## Physical Graal compiler result for `run` (not the module)

Same Java/GraalVM generation as baseline (25.4.4.1.1), requested tiers in independent JVM test runs, 128 `run` invocations, CACHED interpreter, stable physical `run` root; each BGV is tied to exactly one compiled root. Counts represent `After TruffleTier` node census for `run`, not a sum of three root graphs.

| Run-only metric | Prior Protos cold | I115 worktree Tier 1 | I115 worktree Tier 2 |
| --- | ---: | ---: | ---: |
| `After TruffleTier` Tier 1 | 7,137 | **1,119** (−84.32%) | — |
| `After TruffleTier` Tier 2 | 7,078 | — | **922** (−86.97%) |
| Final IR Tier 1 | 19,720 | 3,117 (−84.19%) | — |
| Final IR Tier 2 | 12,086 | — | 2,381 (−80.30%) |
| Native `CodeSize` Tier 1 bytes | 123,547 | 18,787 (−84.79%) | — |
| Native `CodeSize` Tier 2 bytes | 88,854 | — | 17,723 (−80.05%) |
| Inlining | prior run: 0Y/0N | **0Y/4N** | **4Y/0N** |
| Compilation | — | `opt done`, CompId 3571 | `opt done`, CompId 3562 |

**GraalPy remaining gap (cold matching shape):** GraalPy `run` FULL 12, After TruffleTier Tier 1 268 and Tier 2 165. Current Protos is **1.50× FULL**, **4.18× Tier 1** and **5.59× Tier 2**. The substantial lowering and graph improvement is **not** GraalPy parity and is **not** evidence of a throughput speedup.

**The distinct physical `run` target was verified by output:** `protos-root:83c047c2b4bd1384[CLOSURE|truffle:f1d8f055ecf6195535f5f7204349225ee309f0ba97ce59489d60839355faa8b6/perf042-nested-run-identity-program.protos|17+46]`, with successful `PERF042_NESTED_RUN_CAPTURE=READY` in both tiers. `After TruffleTier` stage counts correspond to the selected run compilation, not incidental engine compilations.

## Archival artifact manifest

| Archive member | Size (bytes) | SHA-256 |
| --- | ---: | --- |
| `dsl-full.log` | 5,572 | `b9862494133e992721d166af4f894f7cfff185335e6d75f4d6cd72637b3c85aa` |
| `tier1.log` | 47,092 | `af514b5c0cab38801785b80451d2ad09c0f772350fd8f5a901ddbb9d748a848c` |
| `tier2.log` | 47,092 | `46d9b6a7af9e10fdaedbe0babac0c3b66feffba4b469189e2cadddab06e5b4e7` |
| `tier1/TruffleHotSpotCompilation-3571[...].bgv` | 2,324,795 | `923d1c17799b06ab6cfe7d07681fc19b9df9ba265a37cf20e997102b35066baa` |
| `tier2/TruffleHotSpotCompilation-3562[...].bgv` | 2,393,380 | `5fa188d63415c01b7f7fde853355adbd84c8a04d96d9aee99a9a2eb5a4944880` |

## Acceptance, constraints and next action

- **Owner-reported focused tests:** green before graph capture. The uploaded targeted I112 census and both opt-in Tier captures completed successfully. The agent did **not** execute product tests.
- **Physical-measurement gate:** satisfied for this I115 slice: fewer caller DSL instructions **and** a smaller directly measured Graal graph at both requested tiers. This meets the owner's prerequisite to *allow* integrated `make test`; it does not attest that `make test` has occurred.
- **Still pending:** human-executed integrated `make test`, post-green product version/CHANGELOG update, exact product implementation commit and push, and verification/readback. Do not run more tests after the product version update. No product publication SHA exists for this slice in the supplied evidence.
- **Semantics:** no intentional D013 / D179 / ReturnHome / Context / Task or universal B5+ ABI change. New focused regressions exercise reassignment, instance `call` override/remove, non-local return and suspend/resume. Full integrated regression coverage and runtime benchmark remain pending.
- **Coordination:** existing **I115/#931** remains **OPEN** and owns the slice; **PLAT066-C/#930**, parent **PLAT066/#924**, and **PERF042/#871** remain **OPEN**. Do not create a separate I issue, close the parent design work, or infer that residual graph overhead is entirely due to `InvokeClosureOneTail`.
- **Next PLAT066-C research:** compare surviving physical `run` FrameState/deoptimization metadata, lexical authority materialization, local-slot/activation setup, D013 guarded direct path and `EpilogForReturn` against GraalPy. Require per-node source/graph attribution and safety proof before selecting a further I115 change. The measured gap is substantial despite the improvement; do not optimize from node histograms alone.

**Evidence status:** `WORKTREE_MEASUREMENT_ACCEPTED`, `PRODUCT_INTEGRATED_TEST=PENDING`, `PRODUCT_PUBLICATION=PENDING`, `I115_OPEN=YES`, `PLAT066_C_OPEN=YES`.
