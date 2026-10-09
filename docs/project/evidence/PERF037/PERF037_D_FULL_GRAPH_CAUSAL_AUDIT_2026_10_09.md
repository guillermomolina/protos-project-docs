# PERF037 — Full 88-node graph causal audit and 36-node GraalJS reference (2026-10-09)

**Evidence role:** read-only investigation; not an approved implementation, source change or claim that all nodes can safely disappear. Investigation is performed by the coordinator rather than delegated to a future implementation agent.

## Exact published sources and comparability

- **Product:** [`guillermomolina/protos@3e94ba7d01cf94852555d3daa18a3208d3908e35`](https://github.com/guillermomolina/protos/commit/3e94ba7d01cf94852555d3daa18a3208d3908e35), clean product with human-reported local tests PASS.
- **D graph publication:** [`guillermomolina/protos-benchmarks@a599949830cc7a240ffd347de615006c112c94d6`](https://github.com/guillermomolina/protos-benchmarks/commit/a599949830cc7a240ffd347de615006c112c94d6), [D1 `unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/a599949830cc7a240ffd347de615006c112c94d6/results/perf037-d-3e94ba7d-graphs/primitive-object-slot-read/protos/unit.json), [D1 budget-64000 `filter.json.gz` and BGV](https://github.com/guillermomolina/protos-benchmarks/tree/a599949830cc7a240ffd347de615006c112c94d6/results/perf037-d-3e94ba7d-graphs/primitive-object-slot-read/protos/budget-64000/bgv).
- **C baseline:** `d88ed6b83e977a1bf02f425825a015e9e577ec3e`, 151 nodes; published [C graph](https://github.com/guillermomolina/protos-benchmarks/tree/a599949830cc7a240ffd347de615006c112c94d6/results/perf037-c-d88ed6b8-graphs/primitive-object-slot-read/protos/budget-64000/bgv).
- **JS reference:** published [`global-20261008-graphs/primitive-object-slot-read/js/unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/a599949830cc7a240ffd347de615006c112c94d6/results/global-20261008-graphs/primitive-object-slot-read/js/unit.json) and its [selected BGV/filter JSON](https://github.com/guillermomolina/protos-benchmarks/tree/a599949830cc7a240ffd347de615006c112c94d6/results/global-20261008-graphs/primitive-object-slot-read/js/budget-64000/bgv): 36 nodes and stable.
- **Harness recorded in D unit:** `6160b0a0808a2d9391748bf65eba4667c958cad9`; publish commit above is evidence-retention commit, not the harness producer. D product `3e94ba7d`, `EVIDENCE_VALID=True`, `PRODUCT_CLEAN=True`, `STABILIZATION=STABLE`, analyzer errors 0, verified producer/worktree.
- **Measurement boundary:** both invoke a prepared `Value.execute()` after setup; Protos uses `canonical`, JS uses `executable-value`. These surfaces have similar timed-call shape but different binding names and setup adapters, so graph structural differences must not be mistaken for a universal language-performance theorem.
- All counts below concern the graph phase **`After TruffleTier`**, not final machine-code node count or runtime latency.

## Full histogram reconciliation: 88 vs 36

Compared class by class, **Protos has 57 positive class-instance differences** and **5 class-instance deficits**; their net sum is **52**. Do not claim that 52 particular Protos node IDs can simply be deleted: graph optimization and cross-language IR are not node-by-node identical. The excess reflects 25 versus 6 constants (+19), 11 versus 1 FrameStates (+10), 7 vs 0 VirtualObjectState (+7), 4 vs 0 VirtualArrayNode (+4), 8 vs 5 FixedGuard (+3), and the remaining classes below.

| Class | Protos | JS | Delta |
| --- | ---: | ---: | ---: |
| `ConstantNode` | 25 | 6 | +19 |
| `FrameState` | 11 | 1 | +10 |
| `VirtualObjectState` | 7 | 0 | +7 |
| `VirtualArrayNode` | 4 | 0 | +4 |
| `FixedGuardNode` | 8 | 5 | +3 |
| `BeginNode` / `EndNode` | 2 / 2 | 0 / 0 | +4 |
| `IfNode` / `MergeNode` / `ValuePhiNode` | 1 / 1 / 1 | 0 / 0 / 0 | +3 |
| `VirtualInstanceNode` / `TrufflePreserveFrameStateNode` | 1 / 1 | 0 / 0 | +2 |
| `RawLoadNode` / `NarrowNode` | 1 / 1 | 0 / 0 | +2 |
| `InstanceOfNode` / `IsNullNode` / `ObjectEqualsNode` | 2 / 2 / 2 | 1 / 1 / 1 | +3 |
| `PiNode` | 4 | 6 | −2 |
| `LoadFieldNode` | 2 | 3 | −1 |
| `LoadIndexedNode` | 3 | 4 | −1 |
| `BoxNode$AllocatingBoxNode` | 0 | 1 | −1 |
| `IntegerEqualsNode`, `GuardedUnsafeLoadNode`, `StartNode`, `ReturnNode`, `ParameterNode`, `PiArrayNode` | 2, 1, 1, 1, 1, 1 | 2, 1, 1, 1, 1, 1 | 0 |

The counts above reconcile to all 88 Protos nodes and all 36 JS nodes, including the six Protos node IDs omitted by the previously shared top-20 histogram (`RawLoad`, `Start`, `Return`, `TrufflePreserveFrameState`, `ValuePhi`, `VirtualInstance`).

## Exhaustive 88-node causal partition

These disjoint sets are complete: 88 unique node IDs, no unassigned IDs. They group the **node's graph role or dependency**, not the number that can be removed independently:

| Graph family / role | Node IDs in D BGV | Count |
| --- | --- | ---: |
| Common Truffle call-target entry and return scaffolding | `0,1,2,3,5,22,28,29,31,40,57,62,1005` | 13 |
| Host return-type profiling | `990,991,995,996,998` | 5 |
| Generated root deoptimization/resumption and virtualized frame state, including snapshot constants | `79,83,85,86,89,90,91,115,121,123,124,128,129,130,134,139,140,141,142,147,158,287,288,289,290,336,1017,1018,1019,1020,1021,1022,1023,1024` | 34 |
| Captured-owner selection, local PRESENCE and fallback branch | `45,96,244,269,270,271,279,319,320,325,328,330,331,332,333,334,335,430,1007` | 19 |
| Bytecode DSL materialized-local typed load | `185,376,573,591,593,596,612,1013,1014,1027` | 10 |
| Monomorphic receiver/member read | `777,778,847,848,850,1008,1009` | 7 |
| **TOTAL** | **all unique IDs** | **88** |

This partition accounts for every node in the selected Protos D graph. A `FrameState` appears in the root-state group even when its bytecode frame is logically an owner-selection call frame; the partition is deliberately source/dependency-oriented.

## Causal findings, confidence and current limits

### 1. Cached owner selected-frame miss branch — HIGH CONFIDENCE: concrete next code change

The only surviving **`IfNode #328`** has exact Java source `ProtosBytecodeRootNode$CapturedOwnerFrameCache.ownerFrameOrNull`, `ProtosBytecodeRootNode.java:1123`. Its code is:

```java
MaterializedFrame ownerFrame = current.ownerFrame();
return accessor.isCleared(bytecodeNode, ownerFrame) ? null : ownerFrame;
```

The BGV directly connects this one conditional to `Begin #330/#331`, `End #332/#334`, `Merge #333`, `ValuePhi #335`, and a `FrameState #336`. `IsNull #430` consumes `Phi #335`; `FixedGuard #1007` is the generated `profileBranch` for `IsCapturedOwnerFrameSelected`. The condition is fed by a real bytecode-local tag read (`LoadIndexed #320`, `IntegerEquals #325`). The preceding owner-identity cache guard is `ObjectEquals #271` → `FixedGuard #269`. These identifications are literal node-source-position and BGV edge facts, not histogram guesses.

**Concrete proposal with high confidence:** reuse the existing **per-site `CapturedNearerScopeAbsence.seenNoSelection` monotonic profile** on the *cached hit with cleared local* path, not only on its slower `selectOwnerFrameOrNull` fallback. On the first previously unseen cleared result, call `transferToInterpreterAndInvalidate()` **before** setting the profile, then return the same `null`; on subsequent misses return `null` without repeated invalidation. The compiled never-miss branch can then collapse into a guard rather than retaining a merged-frame return. Preserve the tag check every call; never cache the PRESENT result. Test deletion, recreation with `PRESENT(null)`, read sites shared between activations, repeated alternation and uncached behavior. Do not add a separate divergent cache flag when the existing no-selection profile suffices.

**This is a structural prediction, not a measured post-change count.** The dependent seven control/phi node IDs are a candidate removable graph island; a guard and its deoptimization state may remain. Do not promise `88 − 7 = 81` or `36` without a new verified capture.

### 2. Bytecode DSL frame-state materialization — HIGH CONFIDENCE for node *origin*, NOT for safe removal

`TrufflePreserveFrameStateNode #139` is emitted by generated `ProtosSemanticBytecodeRootNodeGen$CachedBytecodeNode.continueAt` at generated Java line 5612, calling `CompilerDirectives.preserveFrameStateHere`. It anchors `FrameState #140`, whose deoptimization mapping references `VirtualInstance #85` (`FrameWithoutBoxing`), `VirtualArray #89/#90/#91` (frame locals/tags) and `VirtualArray #1017` (argument array). The corresponding `VirtualObjectState #1018–#1024` records their values in frame snapshots; these are *virtual*, not proof of runtime heap allocations. The compiled owner path additionally retains `FrameState #287–#290/#336` for the inlined selection and cached branch. These are exact source and dependency facts and account for 34 nodes in the root-state bucket.

GraalJS has only framework entry `FrameState #3`, zero `VirtualObjectState`, zero `VirtualArrayNode` and no `TrufflePreserveFrameStateNode`. **But the existing BGV does not prove why the Bytecode DSL generated its preserve-frame call or that it can be disabled without breaking resumability, yielding, tags, debugging or deoptimization.** Do not task an implementation agent to delete `preserveFrameStateHere` without further verified generator/semantic evidence. In particular `@GenerateBytecode` currently enables yielding, tag instrumentation, materialized local access and uncached execution; disabling these to win a benchmark is not accepted.

### 3. Duplicated-looking tag accesses — HIGH CONFIDENCE for origin, removal not established

`MaterializedLocalAccessor.isCleared` in owner selection generates `LoadIndexed #320` of `byte[]` tag storage and `IntegerEquals #325` against the cleared marker 7. Later the generated `handleLoadLocalMat$generic` reaccesses tag bytes with `RawLoad #591`, `Narrow #593`, `IntegerEquals #1014` against object tag 0, guarded by `FixedGuard #596`, followed by actual value `GuardedUnsafeLoad #612`. The load path also has null check `FixedGuard #573`, constants and offsets. It is one **presence test** and one **typed materialized read**; both currently inspect frame tags. BUG018 requires the latter to preserve correct cross-tier local-kind behavior, so removing the second check solely because it resembles a duplicate would be unsound. A fused representation may eventually share a physical tag load, but current graph/source evidence alone does not establish a safe, public Bytecode DSL lowering capable of doing so.

### 4. Property/member and host return — HIGH CONFIDENCE for origin, no safe unnecessary work proved

`ReadMemberAtRoot.guardedExactReceiver` produces the exact receiver-identity guard `ObjectEquals #778`, `FixedGuard #1009`, the actual selected slot cell's `LoadField #848`, closure-value test `InstanceOf #850` and `FixedGuard #1008`. No repeated generic property walk or invoke remains. The member slot is mutable: removing a receiver-shape/identity condition without a replacement stability proof would risk stale reads.

`OptimizedCallTarget.profileReturnValue` contributes `IsNull #991`, `Pi #996`, `InstanceOf #998` and `FixedGuard #990/#995`: these are return-type speculation safeguards in the Truffle host call target, not user-written Protos property-lookup code. The BGV alone does not show a valid source-level elimination for both guards. GraalJS's return instead includes one `BoxNode$AllocatingBoxNode` representing Java `Integer.valueOf`, and lacks these return guards, so strict equality of histograms is not proof of semantic equivalence.

### 5. Attribution / historical A-B caveat

D1 selected graph is 88, C is 151 (−63 across published revisions). Between the two product SHAs the first PERF038-D source changes introduced `CapturedOwnerFrameCache`, and only later PERF037-D introduced branch profiling. Therefore **do not causally attribute all 63 removed nodes to PERF037-D's profile alone**; the two changes are confounded without an intermediary published graph. New BGV does prove exactly which branches and mechanisms survive now.

## Next decision

**A bounded source change is justified now:** make the cached `isCleared` owner-frame miss use the existing per-site `seenNoSelection` deoptimization-before-profile rule, avoiding the observed seven-node If/Begin/End/Merge/Phi island without changing semantics. Other 88-node buckets have complete *graph-origin* attribution but still lack high-confidence safe-elimination mechanisms. The coordinator must finish that semantic/mechanism work before instructing an agent to remove 34 root-state nodes, 10 typed-load nodes or the remaining member/return guards. Do not replace this audit with an open-ended agent research assignment.

**PERF037/#851 remains OPEN.** Current target is the owner's 36-node GraalJS structural reference; current 88-node evidence is not parity, and no time/latency result is claimed.
