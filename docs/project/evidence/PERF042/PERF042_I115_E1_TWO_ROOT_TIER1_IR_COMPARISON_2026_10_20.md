# PERF042 / PLAT066-C / I115-E1 — two-root Tier-1 IR comparison and next implementation candidate

Date: 2026-10-20
Status: **diagnostic evidence and implementation direction only**; I115-E1 has **not** been reported implemented, tested, or benchmarked.
Formal references: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930), parent [PLAT066/#924](https://github.com/guillermomolina/protos/issues/924), historical completed [I115/#931](https://github.com/guillermomolina/protos/issues/931).

## 1. Provenance and limitations

The maintainer supplied targeted Maven compilation logs, `After TruffleTier` class histograms, residual-invocation targets and four BGV-exported JSON graphs for the *same minimal guest programs*. The reported capture uses the real `ProtosPerf042IdentityBuilderOracleTest` test methods, not a handwritten call oracle. The reported Tier-1 opt-in tests completed with `RESULTADO_module=0`, `RESULTADO_identity=0`, `PERF042_RUN_CAPTURE=READY` and `PERF042_TIER1_CAPTURE=READY`.

Protos program:
```protos
identity: (value) => { value }
identity(1)
```
GraalPy program:
```python
def identity(value):
    return value

identity(1)
```

- **Protos:** local JVM/HotSpot capture, approximately GraalVM/Truffle 25.4.4.1.1; compiler IDs `3484` TOP_LEVEL and `3450` CLOSURE; capture log timestamps 2026-10-19T14:39:45Z and 2026-10-19T14:40:06Z. The maintainer did not provide a Git SHA for these exact local captures; do not silently label them as measurements of a particular published commit. Most recently recorded product publication before these captures was `e2200088d42d0d8be90626a79b373ddad64978f5` (I115-C/D, 0.3.385-SNAPSHOT), but that is **not** a verified capture SHA.
- **GraalPy:** Native Image `ghcr.io/graalvm/graalpy-community:25i3-25.3.4.1-ol8`, Tier 1, compiled module `469` and `identity` `110`, both `Inlined 0Y`.
- **Matching axes:** program intent, separate module/closure roots, Tier 1, `After TruffleTier`, no guest `identity` inlining. **Unmatched axes:** JVM vs Native Image, GraalVM version, intrinsic lowering/backend representation, and exact product commit. The sum is **descriptive**, not a single compilation nor a benchmark/speedup result.
- The earlier 2026-10-16 Protos isolated Closure result `IR 106/125` (Tier 1, `CodeSize 481`) was from an older product state; the current isolated Closure is `IR 263/1699`. This temporal difference is not proof that I116-A NORMAL/DEBUG unification caused all 157 nodes. Earlier I112 and other revisions are also intervening factors.

## 2. Measured graph totals

| Independent compiled root | Protos Tier-1 nodes | GraalPy Tier-1 nodes | Protos excess |
| --- | ---: | ---: | ---: |
| Module / TOP_LEVEL | 645 | 469 | +176 |
| Closure / identity | 263 | 110 | +153 |
| Sum of independent roots | **908** | **579** | **+329** |

For module: Protos `IR 645/2626`, `CodeSize 15584`, `Inlined 0Y 1N`.
For isolated identity: Protos `IR 263/1699`, `CodeSize 8880`, `Inlined 0Y 0N`.

Selected *direct node-class* comparisons:

| Class / feature | Module Protos | Module GraalPy | Identity Protos | Identity GraalPy |
| --- | ---: | ---: | ---: | ---: |
| `FrameState` | 185 | 81 | 44 | 15 |
| `RawStoreNode` | 36 | 0 | 0 | 0 |
| `BeginNode` | 45 | 25 | 25 | 9 |
| `AllocatedObjectNode` | 21 | 2 | 10 | 0 |
| `CommitAllocationNode` | 13 | 2 | 6 | 0 |
| `IfNode` | 18 | 11 | 9 | 4 |
| `InvokeWithExceptionNode` | 9 | 3 | 7 | 1 |
| `DeoptimizeNode` | 9 | 3 | 8 | 1 |
| `ReturnNode` | 1 | 1 | **0** | **1** |

The isolated Protos identity graph contains an `IntrinsifyFrameAccessor`-origin `RuntimeConstraint` deoptimization and **no `ReturnNode` in that graph phase**. This is a concrete IR warning, not a directly observed count of runtime deoptimization events. Investigate separately from root-entry cost.

## 3. Actual residual Java calls

Protos module (nine `InvokeWithExceptionNode`):

```text
1 OptimizedCallTarget.callBoundary
1 OptimizedCallTarget.lastTierCompile
1 ProtosBytecodeRootNode$EnterRoot.firstBodyBytecodeIndex
1 ProtosFrameLocalBindings.createCurrentFrameLocalSlow
1 ProtosFrameLocalBindings.readRootFrameLocalSlow
2 ProtosLexicalBindingAuthority.retireInstallation
2 ProtosLexicalBindingAuthorityCalls.transferAll
```

GraalPy module (three): `OptimizedCallTarget.lastTierCompile`, `PFunction.kwDefaultsFromStorage`, `SubstrateOptimizedCallTargetInstalledCode.doInvoke` (each one).

Protos identity (seven):

```text
1 OptimizedCallTarget.lastTierCompile
1 ProtosBytecodeRootNode$EnterRoot.firstBodyBytecodeIndex
1 ProtosFrameArguments.materializeCompactActivation
2 ProtosLexicalBindingAuthority.retireInstallation
2 ProtosLexicalBindingAuthorityCalls.transferAll
```

GraalPy identity: `OptimizedCallTarget.lastTierCompile` (one).

These are graph-resident calls; their presence **does not** prove that they execute on the common run-time path.

## 4. Per-operation origin attribution (not marginal cost)

Prior four-graph Java-source-stack classification attributed **358 of the module's 645 nodes** to individual operations:

| Module origin | Inclusive origin-associated nodes |
| --- | ---: |
| `EnterRoot` | 98 |
| `MaterializeCurrentClosure` | **122** |
| `CreateCurrentFrameLocal` | 27 |
| `ReadRootFrameLocal` | 22 |
| `InvokeClosureOneTail` | 55 |
| `EpilogForReturn` | 34 |

Additional module classifications: interpreter dispatch/slot handling 53, root runtime/entry 99, generated root infrastructure 4, and no retained Java source position 131 (sum 645). Counts depend on the classification algorithm and retained source positions. They are not independently removable, and node provenance can be shared.

Identity provenance classified `EnterRoot` 103 and `BindUnaryFrameParameter` 4; 4 interpreter dispatch, 95 runtime/entry, 3 generated root infrastructure and 54 without retained Java source position (sum 263). **Do not read the 103 attributed `EnterRoot` nodes as 103 authority-installation nodes.**

The stable 11 FULL module + 5 FULL Closure = **16 Bytecode DSL instruction budget** is independent of Graal compiler IR node count; the 908-node sum is not 908 Bytecode DSL operations.

## 5. Source-verified root-entry difference

GraalPy source: `PBytecodeDSLRootNode.EnterCalleeContext` (`@Prolog(storeBytecodeIndex=false)`) calls `root.calleeContext.enter(frame,root)`. In `ExecutionContext.CalleeContext.enter`, this initializes `PFrame.Reference` from the caller frame information and stores it. Ordinary entry does **not** call `frame.materialize()`; `exitEscaped` handles materialization when a frame is escaped.

- [GraalPy PBytecodeDSLRootNode](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java)
- [GraalPy ExecutionContext](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/ExecutionContext.java)

Protos source: `ProtosBytecodeRootNode.EnterRoot` invokes `InstallFrameLexicalAuthority.perform` when `prologAuthorityLocals()!=null`. This can call `ProtosFrameArguments.activation(frame)`, discover `firstBodyBytecodeIndex` across a `@TruffleBoundary`, `frame.materialize()`, allocate a `ProtosFrameLexicalBindingAuthority` and install it, potentially reaching binding-transfer / authority-retirement paths.

Crucial nuance verified in `CanonicalToBytecodeLowerer`: `prologFrameAuthority` requires `requiresPersistentFrameAuthority && !frameLocals.isEmpty() && admitsPrologFrameAuthority(...)`; `admitsPrologFrameAuthority` returns false for `activationDefinition != null`. The simple identity **Closure** does not take that eager persistent-authority module path. The first proposed intervention therefore targets the **module**, not the Closure's `IntrinsifyFrameAccessor` issue.

The module's `MaterializeCurrentClosure` source also matters: `useCompactClosureCapture()` currently requires `currentRootFrameLocals.isEmpty()`; the module declares `identity` and therefore does not qualify for that compact-capture route.

## 6. Owner-requested incremental implementation — I115-E1

The maintainer explicitly selected **IMPLEMENTATION**, repository `guillermomolina/protos`, incremental label **I115-E1** (within existing historical identifier), **no issue closures**, **no `make test` or full suite**, only focused tests and Tier-1 node captures. This entry records the approved *direction*, **not** successful implementation or test acceptance. I115/#931 was already CLOSED at last live read due to its prior C/D work; the owner did not ask to change that state.

Physical objective: make the module root prologue GraalPy-shaped: initialize minimal lightweight lexical/frame metadata without eager frame materialization, durable authority construction, binding migration, or `firstBodyBytecodeIndex` slow call on ordinary entry. Keep exact reference capture, original Context identity, D179 PRESENT/ABSENT semantics, binding creation/order, invalidation, ReturnHome/Task, debugger/DAP, exceptions, suspension/restart and source-root metadata. Promote any deferred state **before** frame lifetime ends or when observation/capture semantics require it; never store an escaped `VirtualFrame`. Avoid adding DSL operations/instructions or shifting full original work to a different handler. Keep the single-interpreter architecture. Minimal focused tests and exact two-root BGV recapture after human execution; *no* full suite, *no* `pom.xml`/`CHANGELOG.md` edits in this investigative slice.

**No acceptance result exists yet.** After I115-E1 execution request exact HEAD SHA, test reports, 11+5 FULL check, both graph counts, per-operation origin attribution and whole-root delta. Do not claim a reduction without these artifacts.

## 7. Next difference to implement **after E1 measurement**

**Recommended next candidate: `MaterializeCurrentClosure` in TOP_LEVEL (provisionally "I115-E2", not a separately allocated GitHub issue).** It accounts for **122 source-attributed nodes**—more than any other individually attributed module DSL handler. Inspect why `currentRootFrameLocals.isEmpty()` blocks compact capture of the `identity` literal, and whether its Closure can retain a **deferred reference** to the creator's lexical environment/Context without forcing the creator's rich activation eagerly, following GraalPy's general lazy-frame principle. Preserve any observable capture (including later writes/removes/Context identity, D179, D013 and errors). Do not assume zero free-variable references mean no lexical-context capture in Protos. Do not amend `MaterializeCurrentClosure` until the E1 result is known: E1 can change its admission and node count.

The *independent* Closure `BindUnaryFrameParameter` / `IntrinsifyFrameAccessor` / zero-`ReturnNode` question remains an **urgent separate diagnostic**, not part of E1 or evidence that E2 will solve it. The old isolated 106-node graph's `ReturnNode` status has not been verified.

No issue state, normalized language semantics, product files, benchmark cases, or closure approvals were changed by this evidence publication.
