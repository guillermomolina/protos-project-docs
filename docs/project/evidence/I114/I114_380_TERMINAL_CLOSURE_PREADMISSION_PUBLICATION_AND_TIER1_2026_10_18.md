# I114 — Terminal Closure pre-admission: 0.3.380 publication and first Tier-1 graph checkpoint

**Date:** 2026-10-18  
**Implementation owner:** [I114/#928](https://github.com/guillermomolina/protos/issues/928)  
**Related focused research:** [PLAT066-A/#926](https://github.com/guillermomolina/protos/issues/926); broader [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924)  
**Performance owner:** [PERF042/#871](https://github.com/guillermomolina/protos/issues/871)  
**Exact product revision:** [`fcc5c405413d70df546c5a9e5fa2c57b78c8ec91`](https://github.com/guillermomolina/protos/commit/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91)  
**Product commit subject:** `I114: admit terminal Closure direct calls before the specialization body (0.3.380)`  
**Source version:** `0.3.380-SNAPSHOT`; no release/tag or distributed-binary claim.

## Published product scope and independent readback

GitHub `guillermomolina/protos/main` resolved to the exact product commit above at the publication readback. The commit modifies **11 tracked paths**: 7 runtime/Bytecode helper files and 2 regression-test files, plus `pom.xml` and `CHANGELOG.md`. No normative `spec/**` path or new Bytecode DSL `@Operation` is changed in this publication.

- `ProtosBytecodeRootNode.InvokeClosureOneTail.definitionSource` moves current-instance admission to a DSL `@Bind` / guard ahead of cache initialization, with an admitted current Closure, a definition-keyed cached target and `DirectCallNode`. The formerly in-body admission/fallback branch and direct specialization's cached `IndirectCallNode` are removed. Existing `rejectedSource` and `generic` remain separate.
- `ProtosClosureCallPreparation.directClosureCallSelectionForPreludeOrNull` no longer performs per-instance full D013 `lookup(receiver,"call",prelude)` for the admitted definition tier. It checks the Closure's own local `call` presence and a lazy invalidatable certificate for canonical root `Object.call`.
- `ProtosStandardObjectProtocol` and `ProtosValueLookup` establish that root certificate using existing guarded selection/assumption dependencies. `ProtosObjectValue` exposes a nonallocating local-`call` presence query.
- `ProtosClosureValue` introduces a nonallocating predicate for an unobservable invocation ReturnHome, leaving semantic home materialization to the appropriate execution paths.
- `ProtosTailCallContinuation.deferDirect` receives actual suspended outcomes on a cold branch; the ordinary direct-call return path checks for suspension and projects the result without entering `completeOrDefer`'s owner-aware tree.
- New regressions in `ProtosPerf042DefinitionKeyedDirectCallTest` and `ProtosI112TailContinuationTest` exercise fresh same-definition Closures, local `call` override/recovery, definition cache continuity, lexical capture/receiver, ReturnHome and exception/fallback paths.

This is an **implementation-specific physical dispatch change**, not a modification to dynamic D013 `Object.call`, B5 frame ABI, or guest language semantics. Code-level separation is independently visible in the published revision; compiled reachability is a separate gate.

## Human-executed validation provenance

The maintainer reported after publication:

| Gate | Result / authority |
| --- | --- |
| Product commit pushed | VERIFIED by exact `main` readback at `fcc5c405413d70df546c5a9e5fa2c57b78c8ec91` |
| `git diff --check` | CLEAN, maintainer-reported |
| All locally executed tests | PASS, maintainer-reported; no independently archived test-run manifest |
| Revision/version metadata | `pom.xml` and `CHANGELOG.md` included in verified product commit |
| Additional test reruns by coordinating agent | NONE |

A static diff review was performed before the human compilation/test run; it was not a substitute for executable validation. The user's green-suite report is accepted as a human-executor result, not represented as independently re-run CI.

## First local Tier-1 graph observation (separate, nonfinal evidence)

The maintainer supplied the **console log**, captured in the local I114 working checkout **before the final publication metadata/commit**, from `ProtosPerf042IdentityBuilderOracleTest#captureCompleteRunRootAtRequestedTier`. This test compiles the real **TOP_LEVEL** root for the bare two-statement program:

```protos
identity: (value) => { value }
identity(1)
```

Reported capture parameters and result:

| Field | Observed |
| --- | --- |
| Compiler tier | `Tier 1`; `PERF042_RUN_COMPILED_TIER=1` |
| Compilation identifier | `CompId 3501` |
| Root selector | `protos-root:886eb56b583af3af`, `TOP_LEVEL` |
| Source | `perf042-bare-identity-program.protos`, `0+42` |
| Warmup until cached Bytecode interpreter | 17 calls; `BytecodeTier=CACHED` |
| Truffle AST nodes (`AST` trace field) | 2 |
| Truffle inlining summary | `0Y 1N`, **not attributed to the particular Closure call** |
| Graal IR nodes after TruffleTier | **1,080** |
| Graal IR nodes final | **3,505** |
| Machine code size | 19,899 bytes |
| Compiler elapsed time | 307 ms (209 + 98) |
| BGV files attributed to selected root | 1; capture `READY` |
| DEFAULT source Bytecode instructions | **11**; includes `c.InvokeClosureOneTail`; no `trace.` or `tag.` instructions |

The BGV was emitted in the maintainer's local checkout under `target/truffle-compilation/i114-identity-tier1/graal_dumps/`. **The BGV itself was not supplied or published in the evidence repository**; this record retains the human-supplied textual compiler result, not a separately inspectable graph archive.

**Critical provenance restrictions:**

1. **1,080 is the complete top-level root's Tier-1 post-TruffleTier IR count**, not the node contribution of `InvokeClosureOneTail` and not the compiled `identity` callee alone.
2. Historical experimental bare-root `4,804` Tier-1 and `4,970` Tier-2 numbers belong to an **earlier locally modified worktree**, documented under [PERF042/0.3.376](../PERF042/PERF042_376_REAL_IDENTITY_GRAPH_DIAGNOSTICS_PUBLISHED_2026_10_18.md). The present 1,080 count is an encouraging directional observation, **not a controlled before/after causal proof** or permission to assert an exact percentage removed.
3. The historically matched `primitive-closure-call` comparator selected a **wrapped `run` Closure**, not this bare top-level root, and included a GraalPy reference at another tier/product provenance. No Protos-versus-GraalPy graph parity or speedup follows from this capture.
4. `Inlined 0Y 1N` does **not** establish which target was not inlined.
5. Without inspection of the selected BGV's control/data reachability, this result does **not** certify the full I114 condition that no generic D013 lookup, `prepareAndEnter`, indirect call, ReturnHome allocation or terminal continuation machinery survives on the admitted compiled NORMAL path.

## Remaining gates and issue boundaries

- **I114/#928:** Published implementation + maintainer-reported green tests are verified/reported as above. The issue's explicit **After-TruffleTier cold-path isolation / graph reachability acceptance** is still **unverified**. Preserve the issue open for bounded physical graph review; do not claim numerical 100% equivalence to GraalPy or close solely from the 1,080-node trace.
- **PLAT066-A/#926:** Implementation checkpoint answers the bounded pre-admission source-routing question, but does not ratify a new platform design or supersede the parent research.
- **PLAT066/#924:** Full twelve-area source-to-source investigation remains independently open.
- **PERF042/#871:** Benchmark equivalence, matched pinned same-root/tier/provenance comparison, final-tier stability, performance timing, and causal PE graph attribution remain separately open.

## Revision-bound reconciliation

```text
WORK_ITEM=I114/#928
PROTOS_REVISION=fcc5c405413d70df546c5a9e5fa2c57b78c8ec91
SOURCE_VERSION=0.3.380-SNAPSHOT
PRODUCT_MAIN_READBACK=PASS
PRODUCT_CHANGED_PATHS=11
HUMAN_GIT_DIFF_CHECK=CLEAN (reported)
HUMAN_LOCAL_TESTS=PASS (reported)
TIER1_LOCAL_CAPTURE=READY (reported)
TIER1_SELECTED_ROOT=TOP_LEVEL bare identity(1)
TIER1_COMPILATION_ID=3501
TIER1_AFTER_TRUFFLE_TIER_IR_NODES=1080
TIER1_FINAL_IR_NODES=3505
TIER1_BGV_ARTIFACT=LOCAL_NOT_PUBLISHED
DIRECT_FAST_PATH_GRAPH_REACHABILITY=NOT_VERIFIED
MATCHED_GRAALPY_COMPARISON=NOT_PERFORMED
I114_FULL_GRAPH_GATE=PENDING
```

**Evidence and scope:** Published product source/commit; human-reported clean diff-check and local tests; human-provided Tier-1 compiler output; existing source/graph-provenance limitations. No additional tests, benchmark workload, graph decoder or runtime mutation were performed by the coordinating agent.
