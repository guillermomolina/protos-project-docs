# PERF042-A3 — Retained compilation-trace readback and implementation-ready bounded handoff

Date: 2026-10-10  
Formal owner: [PERF042/#871](https://github.com/guillermomolina/protos/issues/871)  
Nature: **read-only compiler-evidence reconciliation and source-backed PERF042-B implementation recommendation**. This is an **A3 supplement**, not another A4/A5 research slice, an implementation publication, or a design ratification.  
Prior evidence: [A3 consolidated report](PERF042_A3_COMPLETE_CAUSAL_INVESTIGATION_AND_RETAINED_EVIDENCE_GATE_2026_10_10.md), [A2](PERF042_A2_CURRENT_HEAD_INVOCATION_ATTRIBUTION_2026_10_10.md), [A1](PERF042_A1_SOURCE_AND_GRAPH_INVESTIGATION_2026_10_10.md).

## 1. Provenance and material read

Product source HEAD verified: **`7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`**.  
Benchmark HEAD now: **`8c91b495ac82ebe4a345ef02aa0db43fa5afe8c1`**, exactly one later commit beyond `527035f01107e6c71f860e95f438b7553a5428a2`; that commit adds unrelated I092 `primitive-if-true`, `primitive-if-false` and `primitive-return-literal` evidence and does not change the retained `primitive-closure-call` files.  
GraalVM CE **25.4.4.1.1** with Java 25.0.4.1.1, exact historical source/harness revisions as in A3.

Unlike A3's GitHub connector UTF-8 limitation, the connector's **explicit `fetch_file(encoding=base64)`** made the existing gzip trace contents readable in memory, and the analysis independently decoded their gzip DEFLATE payloads without running the product, harness, Java, Maven, benchmarks, validation scripts or any project command. Two retained traces were actually read:

- `results/global-20261008-graphs/primitive-closure-call/protos/budget-16000/trace.log.gz` (clean product `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`, harness `98abc9af7a05a45a7d4056b72f36aef5889cb76f`);
- `results/graphs-full-0db24f00ff2d/primitive-closure-call/protos/budget-256000/trace.log.gz` (later product `0db24f00ff2d92d642351d7f7535517fe01a55ce`, dirty harness `d9719253b628faad1ca296daef4a7ed566196216`).

The cleaned 64000 graph `*.filter.json.gz` (2,703,988-byte Git blob) and later primary graph `*.filter.json.gz` (1,395,134-byte Git blob) cannot be transferred through GitHub Contents API's >1 MiB inline-content response limit: `fetch_file(encoding=base64)` returns empty `content` for both. The later child graph (61,953-byte Git blob) is obtainable. **The large graph IR incoming control/data edges are not claimed inspected**. This limitation does **not** invalidate the separately source-proven bounded optimization below and does **not** authorize speculative removal of node families.

## 2. Critical correction: later entry DID reach Tier 2

The A3 wording `GRAPH_NOT_STABLE` applies to the **specific selected standalone Protos guest root Tier-2 gate** in `unit.json`; it does **not** imply no Tier-2 compilation of the measured host call or no successful guest inlining.

| Event | Clean trace, budget 16000 | Later trace, budget 256000 |
| --- | --- | --- |
| Initial host `Value<ProtosHostExecutableClosure>.execute` | T1 CompId **2324** | T1 CompId **2295** |
| Standalone Protos primary root T1 | CompId **2377**, child **Cutoff** | CompId **2427**, child **Cutoff** |
| Standalone Protos child T1 | CompId **2392** | CompId **2389** |
| Later host T1 invalidation | None in selected clean trace | Host id 275 deopt `Invalidated true`, `Reason marked for deoptimization`, then T1 host CompId **2405** |
| **Host Tier 2** | CompId **2483**; Protos primary **Inlined**, Protos child **Inlined** | CompId **2521**; Protos primary **Inlined**, Protos child **Inlined** |
| Protos standalone primary Tier 2 | CompId **2516**, child **Inlined**, eligible **4,533 nodes** | **Not observed** in this trace |
| Protos standalone child Tier 2 | CompId **2522** | **Not observed** in this trace |
| Framework host Tier-2 `IR after TruffleTier / final` from trace | 5,766 / 10,498 | 3,630 / 5,402 |

**Therefore:** the later capture **does compile and inline the complete reachable guest call hierarchy in the Tier-2 host graph**. The selected stand-alone guest-root rule in `unit.json` still says `GRAPH_NOT_STABLE`: this is an evidence-admission outcome, not proof of runtime inability to reach Tier 2. The later observed explicit deoptimization belongs to the host `Value.execute` target, not to the selected Protos primary root. There is no evidence here of a repeating Protos guest-root deoptimization cycle. The exact policy/threshold cause of the missing separate guest Tier-2 compilation is **not established**; it need not be guessed or diagnosed through new measurements before the source-proven implementation.

**Do not compare** clean host 5766 against later host 3630 as an admitted performance A/B: product and harness revisions differ, and later harness is dirty. Neither count predicts new HEAD results. **Do not change graph policy**, artificially extend warmup or label the later standalone Tier-1 graph as valid.

The traced clean standalone primary Tier 2 and later host Tier 2 also demonstrate that an initial child Cutoff is not itself a permanent Truffle-inlining prohibition.

## 3. Source-level mechanism: one real grouped C0/C1 correction

At exact current HEAD:

- `ProtosSemanticBytecodeRootNode.TryDirectCallOne.guardedSource` guards `receiver == cachedReceiver`, with `@Specialization(replaces="guardedSource")` generic MISS path. `TryDirectCallZero` has the **same receiver-identity-only** limitation. Fresh literal Closures naturally exceed instance PIC limits.
- Existing `PrepareClosureCallOne.fastDirect` and zero-arg `PrepareClosureCall.fastDirect` already cache **`CanonicalClosure` definition** plus entered-context target, while using **`directClosureCallSelectionOrNull`** on the *current* receiver to re-run ordinary `Object.call` selection.
- `ProtosBytecodeRootNode.directClosureCallSelectionForPreludeOrNull` invokes `ProtosValueLookup.lookup(receiver, "call", prelude)`, rejects noncanonical selection/native receiver, and returns the **current** `ProtosClosureValue`. `fastOrdinarySendTarget` derives a reusable Context-owned target.
- `TryDirectCallZero.admitted(guarded, arity)` requires the bytecode source root, `provablyNonSuspendingBody`, exact arity target and current `invocationReturnHomeForRuntime()==unobservable()`. The fused C1 and C0 direct entry already use `DirectCallNode`, compact no-home argument headers and transfer bridging.
- The prepared C1 fallback constructs `OrdinarySourceCall` and finishes separately. A same-definition fused direct hit should skip **that** materialized prepared carrier, not bypass mandatory D013 lookup, Closure identity or the current caller/Context.

**Implementation-ready PERF042-B proposal: one grouped change in the same surface for both ordinary Call0 and Call1**. Add definition-keyed, current-receiver-revalidated fused fast paths to `TryDirectCallZero` and `TryDirectCallOne`, reusing the exact already-established selection/target logic and exact current-instance return-home/arity/nonsuspending admission. Retain all generic fallbacks, poly/mega behavior, shadowing and caller provenance. Manage Truffle DSL `@Specialization` / `replaces` order carefully: the first fresh Closure identity miss must **not permanently disable** the definition-keyed specialization.

Why include C0? Both fused classes have the **identical proven defect**, same source file, same D013 selection law, and same one-time validation surface. A separate C0 micro-slice would duplicate implementation and tests. Leave methods/sends outside scope.

Primary intended code files: `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`; `ProtosBytecodeRootNode.java` only if a narrow helper is necessary. Prefer **no changes** to `CanonicalToBytecodeLowerer`, `ProtosFrameArguments`, persistent lexical authority, `ProtosActivation`, `ProtosLexicalEnvironment`, `spec/`, or benchmark policy for this slice. Include the exact canonical `run` C1 and a fresh C0 analog as focal tests.

## 4. Exact semantic constraints and test floor

Governing specs at current HEAD: `spec/semantics/CALLABLES.md`, `EXECUTION_AND_CONTROL.md`, `OBJECT_MODEL.md`. Ratified platform: PLAT040 F′, PLAT042 B′, PLAT044 B′ (only narrow literal callback, **not** general root-elision), PLAT046 caller-thread/pay-as-you-grow. No unresolved blocker for this source-only PIC change was found in `docs/project/registries/IMPLEMENTATION_BLOCKERS.md`.

The emitted frame must contain the *current* Closure instance and captures. Correctness must cover distinct fresh Closure identities across >3 iterations, different captured environments of one definition, fresh Context per activation where observed, lexical escape and mutation/removal, custom per-instance `call` override before and after warming, remove/re-create/cycle, prelude/Context differences, native Closure, mismatched supplied arity/default and spread, fresh ReturnHome/nonlocal return, Error, suspension/continuation and instrumentation. A rejection may take the generic path; never bypass semantics by caching the first Closure value.

Existing regression owners include:
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf014DirectClosureCallSpecializationTest.java` (fresh materializations/definition tier, `call` shadow/override and invalidation).
- `ProtosPerf038EResidualCallGraphTest.java` (caller/materialization/error and polymorphic fallback).
- `ProtosPerf038BCompactMethodCallTest.java` (fresh Context, extraction identity, nonlocal returns).
- `ProtosPerf006B2D3BSelectedStandardObjectCallIntrinsicTest.java` (ordinary canonical selection/instrumentation).
- `ProtosPerf038FResidualPrimitiveCallTest.java` (Call0/Call1 ABI and fallback).

Add focused tests in existing directly relevant classes or one bounded PERF042 test class; do not assume a measured speedup merely because functional tests pass. Validation must follow `AGENTS.work/IMPLEMENTATION.md` impact-aware rules: `git diff --check`, affected focal Java tests, and **one** final integrated `make test` for shared compiler/runtime source. Human executor runs validation, Git, versioning/publication; agent edits only. Change `pom.xml` and `CHANGELOG.md` **only after human reports tests green, immediately before commit/push; do not run tests after those files are changed**. Use Python patches, not diffs, `grep` not `rg`, no `set`, `pipefail`, `exit`; standalone code blocks with explanatory text before/after.

## 5. H2/H3 and residual IR — explicitly not part of this patch

Eager persistent outer frame lexical authority (H2) and caller `CurrentActivation` (H3) are real physical costs, but neither source evidence nor old retained IR permits removing them without a complete preservation proof. Rewriting them as part of the PIC change would conflate representation architecture with a bounded cache correction and risk Context identity, escape, frame ownership, return-home, debugger and deopt semantics. Historical 44 invoke sites, 997 `FrameState` and 285 `IfNode` are **not** established ordinary success-path execution costs or removable node counts. Leave them unchanged.

A fresh graph/timing assessment may be made **once after real product changes and human green tests**, with the repository's published policy, to evaluate the implementation. Respect an admissibility STOP rather than loop warmup. If the guest-root final-tier gate rejects a run whose host is T2, disclose **both** facts; do not change measurement policy without separate authority.

## 6. Final gate and next action

```text
OWNER=PERF042/#871
RESEARCH_SLICE=A3_COMPLETED_WITH_RETAINED_TRACE_SUPPLEMENT
CURRENT_PRODUCT_HEAD=7b609c4ad0e146d6f02a7a3d036c6ff938ccad08
LATER_HOST_TIER2_AND_BOTH_GUEST_ROOTS_INLINED=PROVEN
LATER_STANDALONE_GUEST_TIER2=NOT_OBSERVED
LATER_REPEATED_GUEST_DEOPT_CYCLE=NOT_PROVEN
MAIN_GRAPH_INCOMING_EDGES=NOT_AVAILABLE_GITHUB_API_GT_1_MIB
SOURCE_DEFECT_FRESH_CLOSURE_IDENTITY_PIC_C0_C1=PROVEN
IMPLEMENTATION_READY=YES_FOR_BOUNDED_C0_C1_PIC_CONVERGENCE
RECOMMENDED_NEXT_SLICE=PERF042-B
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEW_FORMAL_ISSUE=NO
PRODUCT_EDITED_OR_TESTED=NO
```

Source investigation is sufficient to issue the **bounded implementation prompt** without new pre-implementation benchmarks or further research slices. This does **not** establish total PERF042 performance closure, graph parity, or permission to eliminate H2/H3. Keep Issue OPEN.
