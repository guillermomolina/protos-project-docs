# I114 — Tier-1 BGV physical-path attribution for 0.3.380

Date: 2026-10-18. Owner: [I114/#928](https://github.com/guillermomolina/protos/issues/928). Related: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924).

**Product source revision:** [fcc5c405413d70df546c5a9e5fa2c57b78c8ec91](https://github.com/guillermomolina/protos/commit/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91), `0.3.380-SNAPSHOT`.

**Artifact:** User-supplied local `TruffleHotSpotCompilation-3501[protos-root:886eb56b583af3af[TOP_LEVEL|...perf042-bare-identity-program.protos|0+42]].bgv`, SHA-256 `53979e1181d69adce89c28526c252153ae5a948f78ed8cadc203f03e1d768187`, 2,128,328 bytes. This original binary is available in the conversation, **not published in this repository**. It was captured in the local I114 checkout before the final product commit and version-metadata publication.

## Decode and phase identity

The complete BGV was decoded as `BIGV 7.0`, using the Graal BGV BinaryReader-compatible pool, node, source-position, input-port and successor-edge format. The parser consumed the full 2,128,328-byte payload with no read errors. The graph metadata identifies `TruffleHotSpotCompilation-3501`, `protos-root:886eb56b583af3af` and the **TOP_LEVEL** root of the bare two-statement program:

```protos
identity: (value) => { value }
identity(1)
```

| BGV graph | Phase | Nodes |
| ---: | --- | ---: |
| 0 | After PE Tier | 1,163 |
| 1 | Call Tree / Before Inline | 2 |
| 2 | Call Tree / After Inline | 2 |
| **3** | **After TruffleTier** | **1,080** |
| 4 | AST / After TruffleTier | 2 |
| 5 | Before HighTierLoweringPhase | 1,080 |
| 6 | After high tier | 1,314 |
| 7 | After mid tier | 1,872 |
| 8 | After low tier | 3,505 |

Graph 3 contains `FrameState=247`, `IfNode=53`, `InvokeWithExceptionNode=13`, `MethodCallTargetNode=13`, `DeoptimizeNode=13`, `AllocatedObjectNode=25`, `CommitAllocationNode=17`, `ReturnNode=1`. IR and virtual/allocation nodes do not by themselves imply an equal number of runtime heap allocations.

## All remaining non-inlined Java invoke targets (graph 3)

| Invoke ID | Target method | Classification |
| ---: | --- | --- |
| 69 | `OptimizedCallTarget.lastTierCompile` | Truffle compilation infrastructure |
| 324 | `EnterRoot.firstBodyBytecodeIndex` | Root entry metadata |
| 421, 469 | `ProtosLexicalBindingAuthorityCalls.transferAll` | Root lexical authority branches |
| 439, 493 | `ProtosLexicalBindingAuthority.retireInstallation` | Root lexical authority branches |
| 1494 | `CreateCurrentFrameLocal.slowCreate` | Declaration slow path |
| 1658 | `ReadRootFrameLocal.slowRead` | Local-read slow path |
| **2200** | **`ProtosLexicalBindingAuthorityCalls.contains`** | Direct-admission guard for nonempty local authority |
| **2254** | **`ProtosStandardObjectProtocol.rebuildCanonicalRootCallCertificate`** | Direct-admission guard when root certificate invalid/absent |
| **2733** | **`ProtosFrameArguments.materializeCompactActivation`** | Conditional `PrepareSendArguments.exactCaller` branch within direct `definitionSource` |
| **2837** | **`ProtosTailCallContinuation.deferred`** | Genuinely suspended terminal result path after direct invocation |
| **3528** | **`OptimizedCallTarget.callBoundary`** | Expected direct guest Closure call, from `OptimizedDirectCallNode.call` |

Each listed `InvokeWithExceptionNode` has a paired `MethodCallTargetNode`, totaling **13 call sites**, not 26.

## Positive evidence for I114 generic-path isolation

In all 1,080 graph-3 node source-position chains, `InvokeClosureOneTail.prepareAndEnter`, `InvokeClosureOneTail.rejectedSource`, `ProtosValueLookup`, and `ProtosTailCallContinuation.completeOrDefer` have **zero matching source-ancestry nodes**. The complete list of thirteen opaque call targets above also contains **no** `prepareClosureCallOneFallback`, `executeTerminalPreparedCall`, ordinary D013 lookup, indirect call or generic `prepareAndEnter` invocation. The only `generic` substring source ancestors were generated `handlePop$generic` instructions associated with the discarded declaration result, **not the invocation operation's generic specialization**.

The direct specialization remains instantiated in this graph: source ancestry for `InvokeClosureOneTail.definitionSource` covers 96 nodes; `enterDirect` 63; admission `admittedInstanceOrNull` 230; and the broader `InvokeClosureOneTail`/generated dispatch hierarchy 429. **These counts overlap** and cannot be added or construed as disjoint cost attribution.

For ReturnHome, the graph contains constant references to the unobservable marker and primitive-result variant at node IDs **2298** and **2774**, but **no** `ProtosReturnHome` virtual instance, allocated object or opaque constructor call. Per-invocation ReturnHome allocation is **not observed** in this particular graph.

## Residual control/data path observations (not execution-frequency claims)

- `IfNode 2196` selects the empty-local-authority shortcut or the nonempty-authority branch calling `contains` at **2200**. A graph route exists from **2200** to direct `callBoundary` at **3528**. The gate still occupies IR and can run for eligible Closures with local slots other than `call`; the ordinary empty-local Closure can bypass it.
- The invalid-certification branch calls `rebuildCanonicalRootCallCertificate` at **2254**. It has a graph path to **3528**. This is certificate restoration, not re-expanded receiver `lookup("call")`.
- `IfNode 2706` tests the caller's representation via an `InstanceOfNode`. Its non-activation branch can reach `materializeCompactActivation` at **2733** and then **3528**. This **is a materialization call reachable in the direct specialization's compiled graph**, although conditional; a claim of absolute removal of all caller-activation materialization would be false. It is not the same as Context target re-projection.
- Direct target `callBoundary` at **3528** has a control path to suspension handling. `ProtosTailCallContinuation.deferred` at **2837** is controlled by outcome checks under `ProtosTailCallContinuation.isSuspended` (starting `IfNode 2804`); the synchronous result path does not require entering `deferred`. Cold continuation is still represented as a call in the same compiled graph for an actual suspension.

The reachability claims use recorded successor edges plus `MergeNode` end-to-merge links. **Structural reachability is a conservative overapproximation:** it does not establish branch execution probabilities or feasibility for one concrete guest input. Node-source ancestry counts include transitive callers and must not be summed.

## Acceptance and outstanding work

**SUPPORTED for this selected Tier-1 graph:** Source-backed direct invocation excludes the old expanded generic `call` selection/prepared-call path, the indirect-invoke fallback and the prior `completeOrDefer` owner-aware tree. ReturnHome allocation is not observed, while true suspension is preserved behind a conditional cold call.

**REMAINING physical residuals:** Admission still has a substantial node footprint (230 ancestry matches, overlapping), including a conditional `contains` boundary; the direct body retains a conditional `materializeCompactActivation` call; genuine suspension has a conditional `deferred` invoke. These are different mechanisms from the removed generic fallback. No exact per-call cost or removable-node percentage follows without further validation.

**NOT PROVEN by this artifact:** The whole-I114 Section F conditions for warmed alternating override/direct deoptimization behavior, multiple contexts/root shapes and matched version/root/tier GraalPy comparisons; complete broader source-to-graph equivalence; causal before/after runtime improvement. The user-reported local tests PASS and clean diff-check belong to the earlier I114 publication evidence. No tests, compilation or benchmarks were run during this BGV inspection.

**Issue state after analysis:** retain **I114 OPEN / status:review** pending remaining acceptance; PERF042 and PLAT066 remain open independently.

```text
PROTOS_REVISION=fcc5c405413d70df546c5a9e5fa2c57b78c8ec91
BGV_SHA256=53979e1181d69adce89c28526c252153ae5a948f78ed8cadc203f03e1d768187
COMPILATION_ID=3501
GRAPH_INDEX_AFTER_TRUFFLE_TIER=3
AFTER_TRUFFLE_TIER_IR=1080
FINAL_IR=3505
GENERIC_PREPARED_FALLBACK=NOT_PRESENT_IN_IR
GUARD_LOCAL_CONTAINS_INVOKE=2200
CERTIFICATE_REBUILD_INVOKE=2254
CONDITIONAL_CALLER_MATERIALIZATION_INVOKE=2733
SUSPENSION_DEFER_INVOKE=2837
DIRECT_CALL_INVOKE=3528
RETURNHOME_ALLOCATION=NOT_OBSERVED
GRAPH_DIRECT_GENERIC_ISOLATION=SUPPORTED
I114_FULL_ACCEPTANCE=PENDING
```
