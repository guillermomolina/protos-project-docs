# PERF037-E — Published guarded member reads and terminal returns; graph-measurement handoff (2026-10-09)

**Live work item:** [guillermomolina/protos#851](https://github.com/guillermomolina/protos/issues/851) — **OPEN**, no graph-parity claim.

## Published product and validation provenance

- Exact product commit: [`guillermomolina/protos@70c40d269294937856c898a753af2f830e9fba25`](https://github.com/guillermomolina/protos/commit/70c40d269294937856c898a753af2f830e9fba25), subject `PERF037-E: optimize terminal returns and guarded member reads (#851)`. GitHub publication independently confirmed.
- Version: `0.3.323-SNAPSHOT` in `pom.xml`; `CHANGELOG.md` entry published. Exactly ten changed files: five production Java, three regression Java, and those two metadata files.
- Maintainer reports successful local complete `make test` and clean `git diff --check` before product publication; no logs or independent build execution retained here. Do not conflate a reported local PASS with a coordinator-executed test.
- The maintainer reports the commit pushed with clean diff. No separate clean-checkout benchmark evidence for `70c40d26` exists at the moment this record was authored.

## Changes covered by PERF037-E

- `CanonicalToBytecodeLowerer`: direct terminal return for admitted literal/lookup/member/identity value expressions; Closure terminal roots prepared before opening return; nonterminal simple values discarded rather than stored/reloaded; composed-expression scratch shared where admitted. The scalar-local read and exception/control-transfer fallback paths are retained.
- `ProtosSemanticBytecodeRootNode`: `ReadMemberAtRoot`, `ReadInlineMember` and ordinary `ReadMember` accept constant member names; `DiscardValue` consumes evaluated unused expressions. Stable ordinary-data member reads have their own Truffle DSL specialization alongside the preexisting generic fresh Closure-extraction path.
- `ProtosBytecodeRootNode` and runtime `ProtosValueLookup`/`ProtosMapBackedLexicalBindingAuthority`: guarded exact-receiver and shared-inherited reads can load the current physical `SlotCell` value under both selection stability and a lazily allocated, one-way non-Closure continuity assumption; a data-to-Closure write invalidates before replacement. The parent, shadowing, owner and fresh method-binding rules remain in the fallback.
- Regressions cover direct terminal return, sequence ordering, error/control transfer, instrumentation tags, constant member names, non-Closure assumptions and transitions, inherited shadowing, preserved fresh Closure extraction, and pay-as-you-grow object storage.

## Reference graph and strict next measurement (current-HEAD policy)

- Last measured **pre-E** product: `89e1b038c2fd510f47ddb991496882548d1bbe44`, retained valid/stable `After TruffleTier` graph **64 nodes** (mixed PERF037-D/PERF038-F revision) in [benchmarks published capture](https://github.com/guillermomolina/protos-benchmarks/tree/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs).
- Historical stable JavaScript comparison for `primitive-object-slot-read`: **36 nodes**, `executable-value` surface; Protos uses `canonical`, so a numerical node gap alone does not prove identical compiled work or a guaranteed net deletion count.
- **E measurement pending**. Do not claim a node count, stabilization, latency or attainment of the `<=36` target before the new evidence is captured and verified.
- Required new evidence unit: the **real current clean Protos HEAD at capture time**, recorded as an exact full SHA by the harness; workload `primitive-object-slot-read`, language `protos`, stage `reference`, currently committed `truffle/measure_graphs.py` and `truffle/measure/graphs.json` policies. The PERF037-E publication SHA `70c40d26` is historical implementation provenance, **not a checkout constraint**. Concurrent legitimate follow-up commits must not be rolled back. Name the output `results/perf037-current-<PRODUCT_HEAD_SHORT>-graphs` to avoid overwrites.
- Capture only Protos. Correctness must PASS. Do not suppress `NOT_STABLE`, expand budgets silently, overwrite old captures, or recapture unchanged GraalJS/GraalPy.

Human-executor commands (benchmark devcontainer, using the current clean `/workspaces/protos` HEAD; do not check out the earlier E revision):

```bash
cd /workspaces/protos-benchmarks
PRODUCT_SHA=$(git -C /workspaces/protos rev-parse --short=8 HEAD)
python3 truffle/measure_graphs.py capture --dir /workspaces/protos --stage reference --workload primitive-object-slot-read --language protos --output "results/perf037-current-${PRODUCT_SHA}-graphs"
```

On a host with the pinned Docker IgvUtility analyzer and the same `results/` volume:

```bash
cd ~/Fuentes/protos-benchmarks
# Set RESULTS_DIR to the exact "results/perf037-current-<SHA>-graphs" path from capture.
RESULTS_DIR="results/perf037-current-REPLACE_WITH_CAPTURED_PRODUCT_SHA-graphs"
python3 truffle/measure_graphs.py analyze --output "$RESULTS_DIR"
python3 truffle/measure_graphs.py summarize --output "$RESULTS_DIR"
python3 truffle/measure_graphs.py verify --output "$RESULTS_DIR"
```

Acceptance gate: read exact `capture.json.product_before` identity/clean flag, `unit.json.protos_revision`, `capture_valid`, `evidence_valid`, stabilization status, selected-phase graph and full histogram/attribution. If valid and stable, compare new Protos-only `total_nodes` with 64 and the archived 36-node JS baseline, **but attribute any change to the complete intervening commit range rather than uniquely to PERF037-E**; retain raw BGVs and selected filtered graph before declaring graph progress. Any failed correctness/stabilization invalidates the relevant claim. Issue remains OPEN pending graph evidence and independently evaluated latency/cost.


## Diagnostic capture received: dirty product state

The maintainer executed `truffle/measure_graphs.py capture` for `primitive-object-slot-read/protos` under the current working tree. The first reference capture was rejected because the product checkout was dirty. The maintainer then deliberately used `--allow-dirty-product`, preserving concurrent agents' work instead of resetting any files. Harness output:

```text
RESULT_DIR=results/perf037-current-70c40d26-graphs
SURFACE_SUPPORTED=YES
ADAPTER_CACHE=compiled
CORRECTNESS=primitive-object-slot-read/protos PASS
BUDGET=primitive-object-slot-read/protos 1000 NOT_STABLE units=1 problems=UNIT_NOT_AT_FINAL_TIER
BUDGET=primitive-object-slot-read/protos 4000 NOT_STABLE units=1 problems=UNIT_NOT_AT_FINAL_TIER
BUDGET=primitive-object-slot-read/protos 16000 STABLE_CANDIDATE units=1 problems=-
BUDGET=primitive-object-slot-read/protos 64000 STABLE_CANDIDATE units=1 problems=-
CAPTURE=primitive-object-slot-read/protos valid=YES stabilization=STABLE pair=[16000, 64000]
CAPTURE_INVALID_CASES=0
```

**Scope:** this is a **diagnostic capture of a dirty product working tree**. The `70c40d26` short revision embedded in the output directory labels only the repository commit; it does **not** mean the compiled sources match that clean commit. The capture metadata includes the product source-state SHA256 to distinguish the working tree. The graph driver accepts explicit dirty-product captures for diagnosis; its `summarize_case` structural validity check does not independently reject a dirty product, so `evidence_valid=YES` by itself would not make this a clean reference. Do not attribute the graph exclusively to PERF037-E and do not discard concurrent uncommitted work.

**Next:** run analyzer, summarizer and producer-hash verification against the existing output directory, inspect the measured total and node histogram; compare diagnostically with historical Protos 64 and JS 36. Retain this capture as dirty diagnostic evidence, and perform a fresh clean-HEAD reference capture later only if a durable, reproducible comparison is required.

## Analyzed 62-node dirty-checkout graph (maintainer-supplied, 2026-10-09)

Maintainer supplied results of analyzer + summarizer for `results/perf037-current-70c40d26-graphs/primitive-object-slot-read/protos/unit.json`:

- `PRODUCT_HEAD=70c40d269294937856c898a753af2f830e9fba25`
- `PRODUCT_CLEAN=False`
- `SOURCE_STATE=f86e9092be45e83191e85a992ad4af05241f1ff1e402fc6ed8d2844f887dc106`
- `EVIDENCE_VALID=True`, `STABILIZATION=STABLE`, `NODES=62`.
- Stable historical pre-E Protos baseline is `64`, giving a **descriptive difference −2 nodes**, and archived GraalJS is `36`, leaving **26 nodes gap**; because the new product source state was dirty and may include concurrent unpublished changes, the two-node delta **cannot be attributed solely to PERF037-E**. Do not label it reproducible clean-HEAD reference or infer that a `70c40d26` checkout by itself produces 62 nodes.
- Exactly two graph classes changed from the historical 64-node histogram: `FixedGuardNode: 6→5` and `InstanceOfNode: 2→1`. All other class counts are unchanged, notably `ConstantNode=20`, `FrameState=6`, `VirtualArrayNode=4`, `VirtualObjectState=5`, `VirtualInstanceNode=1`, `TrufflePreserveFrameStateNode=1`, and `NarrowNode=1`. `families` now: `allocations=0, control_flow_splits=0, guards_deopts=6, invokes=0, loads=5, loops=0`. These are Graal IR virtual structures, not asserted runtime allocations.
- The unchanged workbook source is `holder: { value: 1 }\nrun: () => { holder.value }`; `holder` is lexically captured by `run` and therefore requires the captured-binding owner selection/read mechanism. The source path in `CanonicalToBytecodeLowerer.emitCapturedMaterializedRead` selects the owner frame then uses the Bytecode DSL `LoadLocalMaterialized(ownerLocal)`, with D179 fallback on absence; the member read follows. Attribution of individual surviving frame/guard nodes requires *the actual filtered graph's edges/properties*, not class frequencies alone.
- **Next investigation:** obtain the existing `unit.json.units[0].analysis` compressed filtered graph and map all surviving nodes to captured-owner frame selection, loaded scalar/tag, member-read, and Bytecode root/frame-state machinery. Preserve D179/BUG018 correctness; no speculative closure/captured-path removal, no arbitrary `@TruffleBoundary` concealment. Do not re-run compiler capture merely to get the already-generated filtered graph.

Note: the owner supplied only the unit summary, not the actual filtered graph or `verify` command output; do **not** claim that producer-hash verification passed. 

## Four-way peer graph comparison: Protos pre/post, GraalJS and GraalPy

**Workload:** `primitive-object-slot-read` in the existing harness. All four `unit.json` records report `evidence_valid=true`, `STABLE`, the `After TruffleTier` graph, final tier 2, and the stable budget pair `[16000, 64000]`. JS/Python records are unchanged, already published in `guillermomolina/protos-benchmarks@623e494493f0d3f91136d10048f365c24407c78c/results/global-20261008-graphs/`; no peer reruns were performed. Captured Protos current source remains **dirty**, so the source-state SHA256, not the short commit label, distinguishes the actual code.

| Relevant graph | Nodes | Delta vs GraalJS 36 | Evidence and provenance |
|---|---:|---:|---|
| GraalJS | **36** | baseline | [archived peer unit](https://github.com/guillermomolina/protos-benchmarks/blob/623e494493f0d3f91136d10048f365c24407c78c/results/global-20261008-graphs/primitive-object-slot-read/js/unit.json), `executable-value`, stable |
| Protos pre-E | **64** | +28 | [old product `89e1b038`](https://github.com/guillermomolina/protos-benchmarks/blob/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs/primitive-object-slot-read/protos/unit.json), `canonical`, stable |
| Protos current diagnostic | **62** | +26 | owner-provided current `unit.json`, product dirty, source-state `f86e9092be45e83191e85a992ad4af05241f1ff1e402fc6ed8d2844f887dc106`, `canonical`, stable |
| GraalPy | **103** | +67 | [archived peer unit](https://github.com/guillermomolina/protos-benchmarks/blob/623e494493f0d3f91136d10048f365c24407c78c/results/global-20261008-graphs/primitive-object-slot-read/python/unit.json), `executable-value`, stable |

Selected node-class counts (missing classes are zero):

| Node class | Protos pre | Protos current | JS | Python |
|---|---:|---:|---:|---:|
| ConstantNode | 20 | 20 | 6 | 30 |
| FrameState | 6 | 6 | 1 | 10 |
| FixedGuardNode | 6 | 5 | 5 | 7 |
| InstanceOfNode | 2 | 1 | 1 | 2 |
| VirtualArrayNode | 4 | 4 | 0 | 4 |
| VirtualObjectState | 5 | 5 | 0 | 7 |
| VirtualInstanceNode | 1 | 1 | 0 | 2 |
| TrufflePreserveFrameStateNode | 1 | 1 | 0 | 1 |
| NarrowNode | 1 | 1 | 0 | 0 |
| GuardedUnsafeLoadNode | 1 | 1 | 1 | 1 |
| LoadFieldNode | 2 | 2 | 3 | 6 |
| LoadIndexedNode | 2 | 2 | 4 | 6 |
| PiNode | 4 | 4 | 6 | 10 |
| **All node types (total)** | **64** | **62** | **36** | **103** |

This is **structural comparison, not directly equivalent semantic overhead**: peer program shape maps the same read-and-return idea, but Protos is `canonical` and JS/Python are `executable-value`; the JavaScript/Python captures use an earlier benchmark producer and product reference. Do not treat a numerical 26-node gap as proof that every extra node can or should be removed. The full four filtered graphs (their own `unit.json.units[0].analysis` paths) are required to attribute surviving frame and guard graphs causally.


## Four-filtered-graph causal attribution (owner-uploaded JSON)

The owner uploaded the **complete filtered `After TruffleTier` graphs** (not just histograms), four independently parseable `StructuredGraph` JSON objects containing node IDs, Java `nodeSourcePosition`, typed edges and virtual-object mappings. Local graph inspection only; this note does not invent a clean source revision or independent benchmark execution.

| Graph | Nodes | Edges | Uploaded filtered JSON SHA256 |
|---|---:|---:|---|
| Protos pre-E (`89e1b038`) | 64 | 94 | `65a2b76ced9958c1cf14f25422fd4da0e593316ce5d880f838074d7e91148678` |
| Protos current, dirty (`source_state=f86e9092...`) | 62 | 90 | `6bcd472dc0456e1eab541d5e6f2c1116db11a03cfc39f22b54d4bb7fa423d715` |
| GraalJS published peer | 36 | 57 | `ecc9572f1045437f2b214ee7d107d3d520a92ca9d8e4961f3e6e58324fc0d413` |
| GraalPy published peer | 103 | 177 | `711b9f0c752ff46113a8efbd4fdddb6cacf3f9f577842c163514bb5828ea0253` |

### Exact explanation of observed Protos 64 → 62

- Previous member read: `SlotCell.value` load **#797** called via `ProtosValueLookup.materializeGuardedMemberRead` / `ReadMemberAtRoot.guardedExactReceiver`; `InstanceOfNode #799` checks `ProtosClosureValue`, `FixedGuardNode #957` consumes that condition.
- Current member read: `SlotCell.value` load **#722** called via `ProtosValueLookup.guardedPlainSlotValue` / `ReadMemberAtRoot.guardedExactReceiverPlain`. The Closure check and its guard are absent; the current receiver/slot guard survives as `FixedGuardNode #802`. This precisely matches histogram changes `InstanceOfNode 2→1`, `FixedGuardNode 6→5`, all other classes unchanged.
- The current remaining `InstanceOfNode #792` checks **`ProtosIntegerValue`** at `OptimizedCallTarget.profileReturnValue` (Truffle runtime); it is **not** the original Closure/member guard. Related runtime return-profile structures are `IsNullNode #785`, `FixedGuardNode #784` and `FixedGuardNode #789`. Do not optimize them as member lookup operations.
- The virtual frame's Object, long and byte slot arrays shrink **6→5** entries (`VirtualArrayNode #89/#90/#91`), consistent with fewer Bytecode locals from direct terminal return. They remain virtual arrays of the same count; this change has no histogram-node benefit. Since product source is dirty, this is causal code-path consistency **not** strict single-commit isolation.

### Runtime value path in the current 62-node Protos graph

Typed `Successor`/`Value`/`Condition` edges prove this particular compiled chain:

1. Java `ProtosClosureValue.capturedEnvironment` load **#244**, then owner identity compare **#271** → `FixedGuardNode #269` from `CapturedOwnerFrameCache.ownerFrameOrNull`. This is the captured binding `holder` and its recorded owner.
2. `handleLoadLocalMat$generic` (generated semantic Bytecode DSL) loads a frame slot tag from an embedded byte array: `RawLoadNode #539` → `NarrowNode #541` → `IntegerEqualsNode #804` → `FixedGuardNode #544`; then `GuardedUnsafeLoadNode #560` reads the actual owner-local object from embedded `Object[]`. It is a **captured-owner frame local** read, not receiver slot extraction.
3. `GuardedUnsafeLoad #560` → `ObjectEquals #703` → `FixedGuard #802` checks stable receiver identity before `LoadFieldNode #722` accesses `ProtosMapBackedLexicalBindingAuthority$SlotCell.value`, then `ReturnNode #799` receives the value through runtime return profiling.
4. No `InvokeNode`, back-edge/loop or explicit `NewInstanceNode` exists in the `After TruffleTier` graph. Runtime memory traffic is not equal to histogram size; especially virtual nodes must not be advertised as physical allocations.

### State/deoptimization bundle, and differences from the peers

- `TrufflePreserveFrameStateNode #139` holds `stateAfter` → **`FrameState #140`**; `FrameState #140` owns exactly five `virtualObjectMappings` edges to **`VirtualObjectState #809–#813`**. State **#809** describes virtual `FrameWithoutBoxing #85`, referencing four virtual arrays (`#808 Object[2]` arguments, `#89 Object[5]`, `#90 long[5]`, `#91 byte[5]`); state **#810** describes the `Object[2]` argument array, containing the Closure **#45** and `ProtosActivation #62`. `#62` is only used in that deoptimization-state mapping; it is not a live read in the optimized data path. The other three mappings describe indexed frame arrays. The generated `CachedBytecodeNode.continueAt` frames `#140–#142` plus runtime `#3/#123/#124` account for all six FrameState nodes.
- **GraalJS (36 nodes):** one common runtime `FrameState #3`; no Bytecode-DSL `continueAt` preservation, no `VirtualObjectState`, no `VirtualArrayNode`. However, JavaScript **does not avoid captured-frame semantics**: `JSFunctionObject.enclosingFrame #241` → null/class guards `#246/#248` → `FrameWithoutBoxing.indexedTags #274` and tag-indexed loads `#278/#298` → `indexedLocals #307` → `GuardedUnsafeLoad #319`; its property path includes object/profile guard and an `Integer` boxing node `#484`. JS's `FrameWithoutBoxing` is an already existing captured frame **loaded through the Closure**, whereas Protos also models a new virtual **current execution frame** for Bytecode-DSL deoptimization.
- **GraalPy (103 nodes):** like Protos it uses generated `PBytecodeDSLRootNodeGen$CachedBytecodeNode.continueAt` and a `TrufflePreserveFrameStateNode #197`; it retains 10 `FrameState`, 4 `VirtualArrayNode`, 7 `VirtualObjectState`, 2 `VirtualInstanceNode`. Its additional global-dictionary/shape/attribute paths (`PHashingCollection.storage #335`, `DynamicObjectStorage.store #347`, `DynamicObject.shape #426/#606`, `DynamicObject.extVal #831`/unsafe load `#842`) and explicit branch explain why Python is structurally larger, not why Protos must retain that overhead.
- Exact **current-Protos minus JS** histogram delta is **+26**, consisting of positive classes: `ConstantNode +14`, `FrameState +5`, `VirtualObjectState +5`, `VirtualArrayNode +4`, `VirtualInstanceNode +1`, `TrufflePreserveFrameStateNode +1`, `RawLoadNode +1`, `NarrowNode +1`, `ObjectEqualsNode +1`; negative classes relative to JS: `BoxNode$AllocatingBoxNode −1`, `IntegerEqualsNode −1`, `LoadFieldNode −1`, `LoadIndexedNode −2`, `PiNode −2`. Thus **the raw +30-looking frame/constant classes do not imply 30 separately removable runtime operations**; the net gap also reflects different legitimate approaches to the captured read. The raw-delta categories overlap conceptually and are not an allocation count.

### Bounded next investigation; no implementation approved here

Investigate pay-as-you-grow **as a package**, not one micro-guard at a time:

1. **Primary:** whether this trivial captured-read root needs the Bytecode-DSL `TrufflePreserveFrameState` + complete virtual current-frame reconstruction in **all** compiled steady executions. Compare with GraalJS's existing captured-frame read and with GraalPy's Bytecode-DSL example. Any work on alternate execution root must be coordinated with **PERF038** (existing root-alternative ownership), avoiding duplicate root machinery and preserving deoptimization, debugging, instrumentation, bytecode state and D179/BUG018.
2. **In parallel:** exact owner-frame cache/tag constraints and whether the simple immutable stable captured-owner scenario can specialize to a direct typed/stable read without general receiver/captured fallback being compiled; ensure partial evaluation invalidation, dynamic nearer binding/shadowing, different activations, owner authority retirement, and PRESENT(null) versus ABSENT are preserved. Do not remove tags or assumptions just to reduce node count.
3. **Return/member specialization:** the surviving current member path is already `guardedExactReceiverPlain`; investigate return-type profiling independently from member lookup, distinguish Truffle optimizer return-profile guards from Protos-owned checks, and compare JS integer boxing/type propagation. No blanket root-return-profile bypass or semantic relaxation.
4. Separate **compiler-state metadata** from **runtime loads/guards/allocations**. Use raw filtered graphs and compiled executable measurements before predicting wall-time impact. Do not silently change Truffle optimization policy, warmup, capture surface, or workload.

All four full filtered JSON graphs are user-provided artifacts whose SHA256 hashes are recorded above; this repository stores the analytical summary, not a speculative graph re-generation. The issue remains open.


## GraalJS implementation precedent: use AST direct root, not Bytecode DSL interpreter

The maintainer clarified the governing implementation strategy: treat the existing GraalJS compiled trivial method as the **desired pay-as-you-grow execution topology** and copy the relevant runtime behavior in Protos, rather than repeatedly asking whether unused features can be removed from the generated Protos Bytecode DSL root.

**Direct upstream source findings (inspected in `oracle/graaljs` and `oracle/graal`):**

1. [GraalJS `FunctionRootNode.java`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/FunctionRootNode.java) has `executeInRealm(VirtualFrame frame) { return body.execute(frame); }`, with a direct `@Child JavaScriptNode body`. [`ReturnNode.TerminalPositionReturnNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/ReturnNode.java) returns `expression.execute(frame)` directly. [`FunctionBodyNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/FunctionBodyNode.java) has a direct `body.execute(frame)` and materializes `DeclareTag` instrumentation only on request.
2. [GraalVM `BytecodeNodeElement.java`](https://github.com/oracle/graal/blob/master/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeNodeElement.java), method `emitContinueAt`, *unconditionally generates for the cached Bytecode DSL interpreter*, in compiled compilation roots, `CompilerDirectives.preserveFrameStateHere()`. The code specifically emits `if (wasCompiled && inCompilationRoot())` without yield support; with yield support it emits `if (wasCompiled && (inCompilationRoot() || continuationRootNode != null))`. **Thus setting `enableYield=false` on the current generated interpreter alone does not eliminate that marker.** The condition is a generated-interpreter architecture choice rather than a dynamic test of whether the executed Closure uses yield.
3. [`TruffleGraphBuilderPlugins.java`](https://github.com/oracle/graal/blob/master/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/substitutions/TruffleGraphBuilderPlugins.java) registers an intrinsic that adds `new TrufflePreserveFrameStateNode()` for `preserveFrameStateHere`; [`TrufflePreserveFrameStateNode.java`](https://github.com/oracle/graal/blob/master/compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/nodes/TrufflePreserveFrameStateNode.java) documents its precise frame-state split/deopt-barrier purpose.
4. Current Protos `ProtosSemanticBytecodeRootNode` is a generated `@GenerateBytecode` interpreter. [`ProtosBytecodeClosureExecutionPlan`](https://github.com/guillermomolina/protos/blob/70c40d269294937856c898a753af2f830e9fba25/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java) fixes `activationTarget = activationRoot.getCallTarget()`. [`ProtosHostExecutableClosure`](https://github.com/guillermomolina/protos/blob/70c40d269294937856c898a753af2f830e9fba25/src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java) dispatches via `RootCallTarget` and `DirectCallNode`. [`ProtosHostEvalRootNode`](https://github.com/guillermomolina/protos/blob/70c40d269294937856c898a753af2f830e9fba25/src/main/java/com/guillermomolina/protos/execution/ProtosHostEvalRootNode.java) is **not** the solution: it is only an embedding wrapper and calls the Bytecode target across a boundary, so the guest closure would still execute through `continueAt`.

**Concrete source design for next implementation (not yet authored or validated):** Dispatch statically eligible, ordinary source-backed Closures to an *AST-based Truffle `RootNode` with direct `@Child` value operations and terminal return*, preserving the same call-target ABI, lexical closure and mutation/absence behavior, source sections, tags, arity rejection, and deoptimization fallback as the Bytecode path. Do **not** implement an AST root wrapper that delegates to the Bytecode root on every successful hot call; that would retain its `preserveFrameStateHere` node. Keep the current Bytecode interpreter for source forms that need its specialized facilities. Selection must be driven by supported source expression/semantics, **never by the named benchmark or `holder.value` literal**. Coordinate with PERF038, which already owns alternate-root/call architecture. PERF037's captured-read workload is a mandatory acceptance workload for that architecture, not an excuse to duplicate root alternatives.

**Acceptance criterion:** for the ordinary value-only closure, compiled `After TruffleTier` root must no longer include the generated `CachedBytecodeNode.continueAt` preserve-frame-state node and its associated reconstructed *current* bytecode-frame bundle; report remaining live captured-frame guards/loads independently. Preserve correctness through existing human-executor tests, instrumentation/visible semantics and generic fallback. The current 62-node graph is dirty-source diagnostic evidence; no claimed target node count or runtime speedup yet.
