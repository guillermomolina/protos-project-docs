# PLAT071 — Dynamic Error.handle catch classification in Truffle DAP: source and observation dossier

## Authority and provenance

- **Investigation date:** 2026-11-05
- **Live decision owner:** [PLAT071 / #1054](https://github.com/guillermomolina/protos/issues/1054), exact-candidate approval **not given**; `status:needs-decision`.
- **Triggered by:** [BUG025 / #940](https://github.com/guillermomolina/protos/issues/940), the earlier incorrect DAP stop on a valid `^`.
- **Exact product source:** [`6787089691803824d7596e02c98ab69c4d3c92e3`](https://github.com/guillermomolina/protos/commit/6787089691803824d7596e02c98ab69c4d3c92e3), `0.3.444-SNAPSHOT`, with GraalVM Community / Truffle `25.5.5.0.3`.
- **Pinned upstream source inspection:** Oracle Graal `e3cb74fceaf6ee4f50366d5ab2bbf602e464475d` (read-only source inspection). Source paths and conclusions are specifically bound to that revision; no claim of a future upstream contract.
- **Observation source:** Maintainer-provided terminal output from a *locally modified, uncommitted* `ProtosI026FDapBehaviorTest` matrix (6 tests, 0 failures/errors; `BUILD SUCCESS`). **The matrix is not in product commit `6787089691803824d7596e02c98ab69c4d3c92e3`**. Retain this distinction in any final closure claim.
- **Nature:** non-normative platform research; not an approved design record, release, merged patch, CI result or a benchmark.

## Reproduction and quantitative matrix

Stop counts from real GraalVM DAP; `none` is no filter, `all` both caught and uncaught, `uncaught` only unhandled.

| Case | none | all | uncaught | Guest outcome |
| --- | ---: | ---: | ---: | --- |
| A: valid `Array.findIndex` `^` | 0 | 0 | 0 | completed with Integer 1 |
| B: `Error().signal()` without handler | 0 | 1 | 1 | failed with ordinary Error |
| C: `Error.handle(() => { Error().signal() }, handler)` in one Task | 0 | 1 | **1** | completed with Integer 42 |
| D: child Task `InvalidReturn`, consumer handles `child.value()` | 0 | 2 | **2** | completed with InvalidReturn Error value |
| E: child Task ordinary Error, consumer handles `child.value()` | 0 | 2 | **2** | completed with Error value |

Case C alone proves the mismatch: the program's handler catches its Error, yet DAP's `uncaught` filter stops. A/B are valid negative and positive controls for valid control transfers and truly unhandled Error. D/E contain separate producer and consumer exception *occurrences*, even though the failed Future preserves the same guest Error *object*. The first producer failure is legitimately unhandled **inside the producer Task**; the consumer-side re-signal may be caught. Do not suppress both events simply because the eventual consumer returns normally.

Every observed exception stop reported `reason=exception`, description `Paused on uncaught exception`, `exceptionId=Error`, `breakMode=unhandled` and Java class `com.guillermomolina.protos.runtime.ProtosSignalException` as the description. All listed stops used DAP `threadId=2`; that identifier is **not** a Protos Task identity. Locations distinguished the separate producer/consumer occurrences. The effective `none` filter produced no stops.

These findings extend the earlier [published PLAT070 source / validation record](../PLAT070/PLAT070_TRUFFLE_25_5_DIRECT_UNWIND_0_3_444_PUBLICATION_2026_11_05.md): valid `^` no longer throws the old `ProtosBytecodeControlTransferException` wrapper and produces no DAP stop even with `all`.

## Normative constraints already settled

[`ERRORS.md`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/spec/semantics/ERRORS.md), [`CALLABLES.md`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/spec/semantics/CALLABLES.md) and [`FUTURES_AND_TASKS.md`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/spec/concurrency/FUTURES_AND_TASKS.md) own the observable semantics. No new semantic contract is selected here.

- `Error.handle` is an ordinary callable/protocol. Argument/receiver evaluation and validation precede handler installation. Matching is by the Error object's delegation ancestry to the handler's match prototype, choosing the dynamically innermost *active* matching frame.
- The selected handler frame is consumed/inactivated **before** the unwind crosses `ensure`. Its handler Closure must not recatch its own new Error. A later Error in cleanup may supersede the selected transfer and must search only still-active handlers. Nonmatching Errors continue outward.
- Task-local dynamic handler frames do not move to child Tasks, Actors, Processes, or consumer Future observations. The Future stores Error object identity, not handler authority; each `Future.value()` failure observation re-signals the same Error as a fresh exception occurrence in the current consumer context.
- Valid `^` and internal Task cancellation are direct `ControlFlowException`-based transfers after PLAT070, not guest `Error` exception breakpoints. `Cancelled` signalled by Future observation is different from internal cancellation unwind.
- Ratified platform constraints: [PLAT005 / #228](https://github.com/guillermomolina/protos/issues/228) semantic tool tags; [PLAT026 / #375](https://github.com/guillermomolina/protos/issues/375) semantic-versus-helper roots; PLAT034 explicitly defers `TryBlockTag`; PLAT049 failure-only immutable stack diagnostics; PLAT067 unified proof-optimized Bytecode stream/lazy tooling. No debug-only AST runtime, all-call stack walk or global handler registry.

## Product execution and error authority

Relevant source inspected at pinned product commit:

- [`ProtosDynamicControlState.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/runtime/ProtosDynamicControlState.java) — handler frames, newest-first match by delegation and inactivation when selected.
- [`ProtosSignalException.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/runtime/ProtosSignalException.java) — one exceptional signaling occurrence with a selected handler frame; separate from the Error guest object.
- [`ProtosCoreErrors.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/runtime/ProtosCoreErrors.java) — selects matching handler at ordinary signal / root-crossing.
- [`ProtosStandardErrorProtocol.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/protocols/actors/ProtosStandardErrorProtocol.java) — ordinary native `Error.handle` implementation.
- [`ProtosStructuredDispatchLowerer.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java) and [`ProtosStructuredControlCalls.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/structured/ProtosStructuredControlCalls.java) — Bytecode DSL `TryCatch` for standard structured `Error.handle`, with exact prepared handler token and cleanup/leave. The Bytecode catch is **not** automatically a `TryBlockTag` debugger catch.
- [`ProtosBytecodeTagTreeNodeExports.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeTagTreeNodeExports.java) — `NodeLibrary` scope projection, not `InstrumentableNode.getNodeObject` catch metadata.
- [`ProtosLanguage.java`](https://github.com/guillermomolina/protos/blob/6787089691803824d7596e02c98ab69c4d3c92e3/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosLanguage.java) — current provided tags: Statement, Expression, Call, Root, AlwaysHalt; **no TryBlockTag**.

## Truffle 25.5 API and DAP mechanism — verified negative findings

1. [`BreakpointExceptionFilter.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/truffle/src/com.oracle.truffle.api.debug/src/com/oracle/truffle/api/debug/BreakpointExceptionFilter.java) checks whether a thrown object is interop-exception; then iterates physical `FrameInstance`s using the throwing node/current frame's call node, walking parent nodes until an instrumentable `TryBlockTag` candidate is found. If `getNodeObject()` has invocable interop member `catches(exception)`, its boolean decides whether that candidate catches. **Absent metadata, the tag is treated as catch-all.** The API passes the exception to `catches`, not the physical frame or its Task identity. It cannot directly discover Protos's separate task-local dynamic handler stack.
2. [`StandardTags.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/truffle/src/com.oracle.truffle.api.instrumentation/src/com/oracle/truffle/api/instrumentation/StandardTags.java) explicitly defines optional `catches` and the catch-all convention for absent node-object predicate. A static `TryBlockTag` is not automatically correct for prototype-selective or inactive/consumed handler frames.
3. [`TagNodeElement.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/TagNodeElement.java) generates an `InstrumentableNode` tag tree for Bytecode DSL with `hasTag`, but **no custom `getNodeObject`**. [`InstrumentableNode.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/truffle/src/com.oracle.truffle.api.instrumentation/src/com/oracle/truffle/api/instrumentation/InstrumentableNode.java) returns null by default. Protos's custom `tagTreeNodeLibrary` only changes `NodeLibrary`, which does **not** override that missing metadata. Therefore just emitting `beginTag(TryBlockTag)` would create false catch-all predictions where visible.
4. [`DebugProtocolServerImpl.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/tools/src/com.oracle.truffle.tools.dap/src/com/oracle/truffle/tools/dap/server/DebugProtocolServerImpl.java) maps filters `all` => caught+uncaught, `uncaught` => uncaught, `none` => disabled. Its `exceptionInfo` response currently **hardcodes** `breakMode="unhandled"` even if debugger catch metadata were available. The Java class description and generic `Error` ID are a *separate* interoperability/presentation improvement.
5. [`TagTreeNode.java`](https://github.com/oracle/graal/blob/e3cb74fceaf6ee4f50366d5ab2bbf602e464475d/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/TagTreeNode.java) is generated-only; the public extensibility point shown for `tagTreeNodeLibrary` customizes `NodeLibrary`, not the generated instrumentable node. No public 25.5 frame/occurrence-aware catch hook matching the full requirements was verified. This is an **evidence limit**, not proof such a mechanism could never be added upstream.

## Comparative runtimes (GITHUB010 research coverage)

| Implementation | Relevant observed approach | Fit / falsification for Protos |
| --- | --- | --- |
| GraalJS | Explicit [`TryCatchNode`](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/control/TryCatchNode.java) has `TryBlockTag` and owns the actual AST catch extent | Protos has runtime dynamic handler state and optimized/native `Error.handle`; a source-text-only try tag cannot represent all invocations |
| GraalPy | `PythonLanguage` declares `TryBlockTag` | Declaration alone is not proof of proper dynamic `except` debugger classification; no exact 25.5 counterpart confirmed |
| Enso | `EnsoLanguage` declares `TryBlockTag` | Likewise no verified full frame-aware `catches` contract |
| TruffleRuby | Public code survey found no `TryBlockTag` use | Negative finding, not proof of missing all debugger exception semantics |
| Oracle SimpleLanguage | Bytecode DSL / standard tags, no demonstrated equivalent dynamic handler classifier | A native `TryCatch` table is not automatically generic-DAP catch metadata |
| Espresso | Own Java/JDWP debugger integration; language advertises narrower standard tag list | Does not supply a minimal generic DAP integration model |
| Apple Pkl | Narrow expression-only tag scheme | Good evidence for truthful tag membership, not a dynamic Error handler precedent |
| Truffle instrumentation test language | Implements actual interop `catches(exception)` on an owned instrumentable Try node | Proves the predicate protocol is real; not generated Bytecode tag tree with recursive task-local contexts |
| HotSpot/JDI, V8, CLR | Exception classification is debugger-specific and can depend on catch-site metadata | Analogous debugger concerns, not reusable Truffle APIs or a reason to fork DAP |

## Candidate analysis; **no ratification**

- **A, document limitation / defer:** no implementation cost or risk; fails the demonstrated same-Task user-facing correctness requirement.
- **B, simple static `TryBlockTag`:** disqualified. Because the generated tag lacks `catches`, it claims a catch-all and can hide nonmatching, consumed or escaping Errors. It may also miss the runtime-native `Error.handle` path entirely.
- **C, Protos-specific DAP fork:** can inspect semantics directly, but duplicates wire protocol, listener, thread/event, Native lifecycle and debugger compatibility. High coupling and cost for the current single need.
- **D, research recommendation:** ask the standard Truffle exception debugger about current dynamic Protos handler selection through a read-only, failure/debugger-path-only, **frame-/occurrence-aware** callback, potentially an upstream extension. There is no verified public 25.5 hook of exactly this shape. Upstream design feasibility and review are blocking **implementation knowledge**, not proof of owner approval.
- **E, AST/tag bridge inside Protos:** can expose a custom predicate in principle, but the current physical `FrameInstance.getCallNode` search, continuation boundaries, inline callbacks, and generic/optimized handler paths make correctness unproven. Adds maintenance and potential no-debug cost.

**Strongest argument against D:** the time and dependency of upstream integration may be disproportionate. **Escape path:** only after falsifying an efficient standard integration, seek a separate exact owner decision on a bounded adapter or deliberately documented limitation. Do not silently convert D into authorization for C or E.

### GITHUB010 comparison, 1–5 (confidence H/M/L)

| Dimension | A | C | D |
| --- | ---: | ---: | ---: |
| Correctness | 2/H | 4/M | 5/M |
| Protos alignment | 4/H | 2/H | 5/M |
| Need proportionality | 5/H | 1/H | 3/M |
| Incremental growth | 4/H | 3/M | 5/M |
| Future-option resilience | 4/H | 2/H | 5/M |
| Scaling | 5/H | 3/M | 5/M |
| Simplicity | 5/H | 1/H | 4/M |
| Portability | 5/H | 1/H | 4/M |
| Runtime/resource cost | 5/H | 2/M | 5/L |
| Failure/operability | 1/H | 4/M | 5/M |
| Reversibility/deferral | 3/H | 2/M | 4/M |
| Evidence maturity | 5/H | 3/M | 2/L |
| **Total (advisory only)** | **48** | **28** | **52** |

B is excluded for semantic incorrectness; E is excluded for unproven physical-frame/tag correctness and disproportionate maintenance. No total overrides those hard gates. D's resource score is conditional on truly optional debugger-only machinery; measure before claiming zero overhead.

## Adversarial acceptance and growth constraints

The smallest acceptable solution must represent **one real Error occurrence and its actually selected active handler**, not retroactively judge the outcome of an entire Task or shared Error object. The mandatory cases are: A valid internal non-local return; B truly unhandled Error; C same-Task handled Error; D/E separate producer and consumer occurrences; prototype mismatch; nested handlers; handler's own new Error; an Error superseding selected handler during `ensure` cleanup; suspended/resumed Task, generic and extracted standard methods, inline Closure callback, concurrent Tasks on same Actor, multiple Contexts, JVM/Native Image; no debugger-specific guest logic or fake source/frames.

Expected stop classification under `none/all/uncaught`: A `0/0/0`, B `0/1/1`, C `0/1/0`, D and E `0/2/1` (the unhandled **producer** only under `uncaught`). The `all` event's catch classification must be confirmed by the underlying debugger event/catch metadata; the current DAP `breakMode` field is not authoritative.

**Pay only for use:** no new global mutable registry, saved live frame graph, Task-wide synchronization, extra frame per ordinary call, normal-path exception probes, compatibility fallback, AST mirror interpreter or instrumentation induced by an unattached debugger. Any upstream hook may perform bounded handler inspection only when the debugger asks about an existing guest Error. Successful `^` remains free of stack walks.

**Grow as needed:** avoid preimplementing a second exception hierarchy or new guest-visible error metadata. Distinct `exceptionId` and guest-friendly `description` are a separate Protos/Truffle interop presentation topic and must not block correct catch prediction. Future upstream API changes may replace a chosen adapter without changing guest semantics or retaining new state. Deferring this integration leaves a visible wrong DAP classification but does not require changing already-normalized language semantics; re-evaluate if upstream proves impossible.

## Formal decision boundary

```text
RESEARCH_RECORD=NON_NORMATIVE
PLAT071_STATE=OPEN_NEEDS_DECISION
OWNER_EXACT_CANDIDATE_APPROVAL=NOT_GIVEN
SELECTED_ARCHITECTURE=NONE
UPSTREAM_25_5_FRAME_AWARE_CATCH_API=NOT_VERIFIED
PRODUCT_CODE_CHANGED_FOR_PLAT071=NO
SPEC_SEMANTICS_CHANGED=NO
LOCAL_DAP_MATRIX=PASS_WITH_REPORTED_DISCREPANCY
LOCAL_MATRIX_SOURCE_PUBLISHED=NO
IMPLEMENTATION_AND_RATIFICATION=NOT_AUTHORIZED
```

If the owner later approves an exact candidate, perform GITHUB021 invariant/delta consistency and publish a **separate ratified** decision record under `docs/project/decisions/platform/`. Do not turn this evidence snapshot into that decision by changing its status or interpreting a request to create the Issue as approval.

**BUG025 boundary:** the original false `uncaught` on a valid `^index` is technically corrected by PLAT070; close BUG025 *only after* its original real-DAP, full-example, and publication/closure-evidence gate. This PLAT071 owns the independently exposed handled-guest-Error classification question. Its unresolved implementation is not a prerequisite for completing the original valid-`^` fix.
