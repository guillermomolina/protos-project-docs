# I115-E2-A/B — experimental source publication, focal acceptance and paired Tier-1 BGV evidence

**Date:** 2026-10-20. **Classification:** historical experimental implementation and measured IR evidence; not a product release or a decision ratification.  
**Owners:** historical [I115/#931](https://github.com/guillermomolina/protos/issues/931) (closed for C/D), physical follow-up [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), parent [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924) and performance tracking [PERF042/#871](https://github.com/guillermomolina/protos/issues/871).  
**Product commit:** [`guillermomolina/protos@2d98a4156e2636884783d042634697faf26487ea`](https://github.com/guillermomolina/protos/commit/2d98a4156e2636884783d042634697faf26487ea), **`experimental/i115`**, message `I115-E2: specialize published Closure creation and materialized lexical capture`.  
**Parent commit:** [`61b94c874c80ed88259051146d00afc0ca14be54`](https://github.com/guillermomolina/protos/commit/61b94c874c80ed88259051146d00afc0ca14be54), I115-E1; this publication builds on E1.  
**Product `main` at verification:** [`e2200088d42d0d8be90626a79b373ddad64978f5`](https://github.com/guillermomolina/protos/commit/e2200088d42d0d8be90626a79b373ddad64978f5). E2 is **not merged into `main`**.

## 1. GitHub source verification

GitHub commit inspection confirms exactly **three modified paths** (not three empty/deleted files):

| Path | Commit diff |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java` | +40 / −5 |
| `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java` | +45 / −0 |
| `src/test/java/com/guillermomolina/protos/execution/compiler/captures/ProtosPerf025LazyLexicalCaptureTest.java` | +129 / −0 |

- **E2-A:** `MaterializeCurrentClosure.published` directly constructs a fresh `ProtosClosureValue` using nullable `methodHome`, `returnHome` and `prelude` runtime getters; public `Optional` APIs remain available.
- **E2-B:** published creation splits into a guarded materialized-context specialization and a general specialization. `ProtosActivation.canCaptureMaterializedContextForRuntime()` requires `!construction && context != null`; the new materialized helper uses the **same `currentEnvironment` cache** as the general `lexicalEnvironmentForClosureCapture()`. The general fallback and deferred capture remain valid for other activations.
- The previous generic `MaterializeClosure.perform` and compact/rich creation paths are retained. No new `@Operation`, Bytecode argument, B5 layout, eager environment creation or new lexical store is introduced. This source property alone is not a runtime/throughput claim.
- The previously published [E2-A investigation and file-edit handoff](I115_E2A_SOURCE_COMPARISON_AND_EDIT_HANDOFF_2026_10_20.md) is historical: its then-pending implementation state is superseded by this **later** commit and validation checkpoint, not retroactively rewritten.

## 2. Human-executed focal acceptance

The maintainer reported, in the experiment's working tree **before publication**:

| Gate | Reported result |
| --- | --- |
| `git diff --check` | `I115_E2B_DIFF_RC=0` |
| `mvn -q test-compile` | `I115_E2B_COMPILE_RC=0` |
| Focal Maven test invocation | `I115_E2B_FOCAL_RC=0` |
| Tier-1 capture script (module + isolated identity) | `I115_E2B_CAPTURE_RC=0`; two BGVs |

Focal log markers included `TRIVIAL_CLOSURE_DOES_NOT_FORCE_OUTER_CONTEXT=PASS`, `MULTIPLE_CLOSURES_SHARE_ENVIRONMENT_IDENTITY=PASS`, `CAPTURE_BY_REFERENCE=PASS`, `CONTEXT_INTRINSIC=PASS`, `D179_C0_REMOVAL_FALLBACK=PASS`, `REMOVE_RECREATE=PASS`, `PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS`, `DEBUGGER_SCOPE_PROJECTION=PASS`, `OBJECT_BODY_LEXICAL_BOUNDARY=PASS`, `BIND_METHOD_CAPTURE_PRESERVED=PASS` and `PARALLEL_PROJECTION_BOUNDARY_PRESERVED=PASS`. Existing and new E2 tests were compiled and the focal invocation returned zero.

**Provenance:** human-reported local Maven/diagnostic execution, *not* independently executed by the documentation publisher. No fresh `make test`, `make check`, Native Image, DAP session, complete-suite test or execution-speed benchmark is claimed for this E2 commit. The 11+5 FULL instruction budget is preserved by the intended unchanged lowering/operation surface but **was not independently re-counted in the displayed E2 capture logs**: `COMPLETE_INSTRUCTIONS=48/28` from the capture script is not a FULL count.

## 3. Exact paired E2-A → E2-B Tier-1 measurements

Both captures use GraalVM 25.4 Tier 1, the same TOP_LEVEL program `perf042-bare-identity-program.protos` (the bare `identity(1)` two-root case) and the independently compiled `perf042-identity-tier1-source.protos` Closure root. Compare the logged `[engine] opt done` `IR <Tier1>/<final>` fields **within this pair**, not with different historical E1 expansion-tree conventions.

| Measurement | E2-A baseline | E2-B after edit | Delta |
| --- | ---: | ---: | ---: |
| Module `opt done` Tier-1 IR | 651 | **640** | **−11 (−1.69%)** |
| Isolated identity Tier-1 IR | 263 | 263 | 0 |
| Independent two-root sum Tier-1 IR | 914 | **903** | **−11 (−1.20%)** |
| Module final IR | 2513 | **2499** | −14 |
| Identity final IR | 1699 | 1699 | 0 |
| Module `CodeSize` (bytes) | 15287 | **15265** | −22 |
| Identity `CodeSize` (bytes) | 8880 | 8880 | 0 |

Matched module compilation IDs: E2-A **3492** and E2-B **3464**. Both BGVs were supplied by the maintainer; their binary SHA-256 fingerprints (verified against attached bytes) are:

- **E2-A** `TruffleHotSpotCompilation-3492[TOP_LEVEL...].bgv`: `337ac3bb2be00eaf8d3901771c602d96223f067ad2243b5eeac2e2b69341049e`.
- **E2-B** `TruffleHotSpotCompilation-3464[TOP_LEVEL...].bgv`: `ebfb811496655594195e28623b8bb3516d7b63631f0e5d7bdd16384cb9032db3`.

The BGVs are **conversation attachments**, not files committed to the documentation repository. The human's capture artifact directory for E2-B was `target/i115-e2a/tier1/20261020T030333912605Z`; the filename's E2-A prefix is the reused capture script, **not** evidence that the second run compiled old source.

The script reports `I115_E2A_REVISION=27cb31d20e517d8377e58963ea595e5ec16bfd15` in both runs; this is an older **Git HEAD identifier printed by the script while compiling the local uncommitted experiment**, *not* the Git revision of the E2-A or E2-B working-tree content. The final experimental source was subsequently published at `2d98a415`; do not falsely relabel a local BGV as a recapture built from that clean commit.

## 4. Decoded BGV graph evidence

Both binaries contain nine graph stages. At **graph 3, `After TruffleTier`**, the observed module totals are 651→640. Relative node-class counts:

| IR type | E2-A | E2-B | Difference |
| --- | ---: | ---: | ---: |
| `BeginNode` | 46 | 42 | −4 |
| `EndNode` | 27 | 24 | −3 |
| `IfNode` | 18 | 16 | −2 |
| `ValuePhiNode` | 12 | 10 | −2 |
| `MergeNode` | 9 | 8 | −1 |
| `FrameState` | 183 | 182 | −1 |
| `LoadFieldNode` | 42 | 41 | −1 |
| `VirtualObjectState` | 14 | 13 | −1 |
| `FixedGuardNode` | 17 | 20 | +3 |
| `PiNode` | 29 | 30 | +1 |
| `AllocatedObjectNode` | 18 | 18 | 0 |
| `VirtualInstanceNode` | 8 | 8 | 0 |
| `RawStoreNode` | 36 | 36 | 0 |
| `MethodCallTargetNode` | 10 | 10 | 0 |

Selected **inclusive Java source-position ancestry** (not disjoint ownership or removable-node budgets):

| Method / operation | E2-A | E2-B |
| --- | ---: | ---: |
| `MaterializeCurrentClosure` | 76 | 70 |
| `ProtosActivation.lexicalEnvironmentForClosureCapture` | 28 | 0 |
| `ProtosActivation.lexicalEnvironmentForMaterializedContextCaptureForRuntime` | 0 | 17 |
| `ProtosLexicalEnvironment.deferred` | 2 | 0 |
| `ProtosLexicalEnvironment.materialized` | 1 | 4 |
| `EnterRoot` | 98 | 98 |
| `CreateCurrentFrameLocal` | 27 | 27 |
| `ReadRootFrameLocal` | 22 | 22 |
| `InvokeClosureOneTail` | 125 | 125 |
| `EpilogForReturn` | 16 | 16 |

**Interpretation:** for the observed materialized module path, Java source positions identify `publishedMaterialized` but not `publishedGeneral`. The deferred-environment constructor has no surviving attributed node on this compiled path; the fallback still exists in source. The net 11-node reduction comes predominantly with simpler control-flow graph nodes, while the number of global virtual/allocated object representations and raw stores is unchanged. The three additional `FixedGuardNode` entries show why a per-method attribution decrease is **not** itself the physical saving.

`AllocatedObjectNode`, `VirtualInstanceNode` and `RawStoreNode` totals are not runtime allocation counts. Source-position ancestry is inclusive. Graph presence/absence does not by itself establish reachable-path frequency or execution cost.

## 5. Interpretation, limits and follow-up

- **E2-B's narrow physical criterion passes:** 651→640 module, identity 263→263, 2513→2499 final module IR. No measured throughput or latency improvement is claimed from −22 bytes of machine code.
- **E2-A's direct-creation mechanism is source-verified**, and its BGV does not retain `Optional` source ancestry in the published materialization path; the isolated **marginal IR benefit of E2-A over a matched post-E1 pre-E2-A graph was not established**. Do not claim the prior 31 Optional-attributed nodes were saved.
- **E1 comparison caution:** prior source/docs report 448+106 = 554 as *expansion-tree* counts and 446 module `opt done`; the 651+263 E2-A figures are a separate capture setup / graph checkpoint. Do not compute E1→E2 gains or regressions by treating these figures as one controlled A/B.
- **Status:** retain E2-A/B on `experimental/i115`; no evidence that they were integrated into `main`. Keep I115/#931's previously completed C/D status and PLAT066-C/#930, PLAT066/#924, PERF042/#871 open. No D013/D207 semantic change or new approved implementation slice is inferred.
- Future work may evaluate remaining module PE costs using exact matched graphs and paths. This record does **not** request repeated tests, another build, an immediate new slice or formal closure of PLAT066-C.

**Source and evidence chain:** [E1 experimental publication](I115_E1_EXPERIMENTAL_AB_ACCEPTANCE_AND_NEXT_PROLOG_BCI_CANDIDATE_2026_10_20.md) · [E2-A research handoff](I115_E2A_SOURCE_COMPARISON_AND_EDIT_HANDOFF_2026_10_20.md) · [I115-C/D final `main` publication](I115_CD_385_FINAL_PUBLICATION_AND_PLAT066_C_RESEARCH_HANDOFF_2026_10_19.md).
