# I115-E / 0.3.386 — integrated main publication and post-merge three-metric capture handoff

**Date:** 2026-10-20. **Classification:** verified product source integration, human-reported release checks and a **future measurement protocol**, not newly measured Graal IR. Product revision: [`guillermomolina/protos@607973c493dcb0c5ba3a7eeba9897b045eb049d8`](https://github.com/guillermomolina/protos/commit/607973c493dcb0c5ba3a7eeba9897b045eb049d8), message `I115-E: defer lexical authority and specialize published Closure creation (0.3.386)`. `main` was read back at this exact SHA. Previous product main: `e2200088d42d0d8be90626a79b373ddad64978f5` (`0.3.385-SNAPSHOT`).

Formal coordination: [I115/#931](https://github.com/guillermomolina/protos/issues/931), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924), [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).

## 1. Verified main publication

GitHub `main` points to `607973c493dcb0c5ba3a7eeba9897b045eb049d8`, which has **one parent**, `e2200088d42d0d8be90626a79b373ddad64978f5`, and is a single squash-style release commit titled `I115-E: defer lexical authority and specialize published Closure creation (0.3.386)`. Diff to the parent comprises **ten paths**:

- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosFrameLocalBindings.java`
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalCompilationFacts.java`
- `src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`
- `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025FrameMaterializationSliceTest.java`
- `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025LazyLexicalCaptureTest.java`
- `src/test/java/com/guillermomolina/protos/execution/compiler/optimizations/ProtosI112TailContinuationTest.java`
- `pom.xml` and `CHANGELOG.md` (**ten paths**, counting both release files; the above first eight plus two release files).

The release POM is **`0.3.386-SNAPSHOT`** and the published CHANGELOG explicitly reports the maintainer's **`make check` PASS and integrated `make test` PASS**. This publication agent did not run or independently replay tests. The source merge preserves E1 deferred root lexical authority, E2-A direct nullable metadata and E2-B materialized lexical-environment capture with fallback. No new language meaning or D013 change is claimed.

The experimental `experimental/i115` branch at `2d98a4156e2636884783d042634697faf26487ea` is historic predecessor evidence; `main@607973c4` is the new authoritative publication. Previous [E1 exploratory A/B](I115_E1_EXPERIMENTAL_AB_ACCEPTANCE_AND_NEXT_PROLOG_BCI_CANDIDATE_2026_10_20.md) and [E2 A/B BGV evidence](I115_E2_AB_EXPERIMENTAL_PUBLISHED_TIER1_GRAAL_EVIDENCE_2026_10_20.md) remain unchanged historical captures and **are not** post-merge main-branch results.

## 2. Pending three-metric physical measurement

No fresh `0.3.386` main-branch per-root Tier-1 or Tier-2 `After TruffleTier` graph counts, nor an independently repeated FULL census, were supplied as part of this release notification. **Do not invent them.** They will be acquired using existing tests already present in the published main:

1. `ProtosI112TerminalCallBudgetTest#normalIdentityMeetsTheTerminalFullInstructionTarget` and `#attachedDebuggerIdentityHasTheSameCompleteCensus`: count all true Bytecode DSL **FULL** instructions for module and real identity Closure, with and without attached debugger; capture exact `I112_ROOT_FULL_COUNT`, `I112_TOTAL_FULL`, `I112_ROOT_COUNT`, `I112_EXTRA_GROUP_ROOTS`. Historical expectation **11+5=16** is not a new post-merge observation.
2. `ProtosPerf042IdentityBuilderOracleTest#captureCompleteRunRootAtRequestedTier`: independently compile the actual bare two-statement TOP_LEVEL `identity: (value) => { value }\nidentity(1)` with `protos.perf042.graphCapture=true`, stable root identity, Tier 1 and Tier 2 separately.
3. `ProtosPerf042IdentityBuilderOracleTest#captureIdentityRootAtRequestedTier`: compile the real lowered one-argument identity Closure root independently at Tier 1 and Tier 2, using the exact B5 invocation.
4. Preserve four separate artifact directories/runs and confirm `COMPILED_TIER=1|2`, CACHED bytecode, `CAPTURE=READY`, unique target selector, nonempty root-matched BGV and `After TruffleTier` graph node counts. The JVM must see `-Djdk.graal.Dump=Truffle:2`, `-Djdk.graal.DumpPath=<unique path>`, and `-Djdk.graal.PrintGraph=File` at startup (passing only a Maven property is insufficient for the dump).
5. Capture separately: true FULL DSL instruction count; `[engine] opt done ... |Tier 1/2| IR <truffleTier>/<final>`; raw BGV `After TruffleTier` node count; compiled code bytes; target identity and warmup calls. **Never** treat `COMPLETE` instrumented instruction counts (`48/28` in an older capture) as 11+5 FULL. Never sum distinct root graph node counts and call them one Graal compilation.
6. Record precise full product SHA, compiler/runtime version `25.4.4.1.1`, JVM vs Native Image, one selected guest source/root per run, experiment setup, BGV file hashes, root-specific `After TruffleTier` graph counts and `opt done` numbers.

The existing capture methods compile `OptimizedCallTarget.compile(false)` for Tier 1 and `compile(true)` for Tier 2, assert `highestCompiledTier`, and dump root-specific graphs. They explicitly isolate `BytecodeConfig.COMPLETE` in another Context to avoid changing the compiled DEFAULT graph. Root labels differ across module and Closure capture methods, so compare program/root semantics and document name/source differences.

## 3. Interpretive constraints

- Historical experimental **E1 matched expansion-tree** totals 648→448 (module), 263→106 (identity), 911→554 summed are one comparison method; do not confuse with post-merge BGV/opt-done graph measures.
- Historical experimental **E2 matched BGV/opt done** pair is 651→640 (module), 263→263 (identity), 914→903 sum. This was captured from local working-tree states prior to the E2 commit and from a separate capture setup; do not infer E1→E2 regression or savings by subtracting those unrelated measurements.
- Previous **GraalPy Native Image** Tier 1 `After TruffleTier` reference (module **469**, identity **110**) was obtained in a distinct Truffle/Graal version and execution plane. It provides descriptive reference only, **not** a controlled JVM-to-JVM benchmark, and no compatible GraalPy Tier 2 result has been confirmed here.
- A fresh Tier-2 Protos trace is required; the old prepublication `4804/4970` Tier-1/Tier-2 results come from an earlier experimental worktree and cannot stand in for `0.3.386`.
- This checkpoint records integration and a measurement plan only. No performance-speedup, post-merge Tier-1/Tier-2 improvement, GraalPy parity, or new implementation acceptance is asserted without fresh artifacts.

## 4. No new implementation slice

The goal is **MEASUREMENT ONLY** under PERF042 / PLAT066-C, on current main HEAD. Do not edit product source, tests, `pom.xml` or `CHANGELOG.md` merely to collect measurements. If a target cannot be captured with the existing harness, report the exact blocker and request the minimal distinct test-only change, not a speculative production change. Product runtime execution remains human-executed under `AGENTS.md`; this record grants no autonomous test execution or product commit.
