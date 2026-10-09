# PERF038-B — Compact method-call specialization: publication checkpoint

Date: 2026-10-08
Issue: https://github.com/guillermomolina/protos/issues/852
Product commit: https://github.com/guillermomolina/protos/commit/248b097e968219452b1663b9ed0fce597a719a79
Product revision: `248b097e968219452b1663b9ed0fce597a719a79`
Product version: `0.3.313-SNAPSHOT`
Status: product changes pushed; post-change graph A/B **not yet available**.

## Verified implementation

The product commit changes `ProtosFrameArguments.java`, `ProtosSemanticBytecodeRootNode.java`, and adds `ProtosPerf038BCompactMethodCallTest.java`, alongside `pom.xml` and `CHANGELOG.md`.

- The compact/unmaterialized root-argument discriminator uses frame argument zero for previously admitted compact calls instead of repeating full compact-header validation.
- Supplied-argument count/read chooses compact header layout by array length.
- `BindClosureFrameParameter` separates guarded compact frame-slot store from an authoritative respecializing fallback behind a boundary, intended to prevent activation materialization and lexical authority expansion in compact-only compilation sites.
- The changes preserve fallback behavior for observed/materialized activation and error cases as intended by the implementation. Runtime conformance is not independently re-tested at this checkpoint.

## Revision-bound pre-change evidence

Published baseline: https://github.com/guillermomolina/protos-benchmarks/tree/bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff/results/global-20261008-graphs/primitive-method-call

Product: `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`, harness: `98abc9af7a05a45a7d4056b72f36aef5889cb76f`, GraalVM: `25.4.4.1.1`; `After TruffleTier`, tier 2, STABLE, correctness PASS, evidence valid YES.

| Metric | Protos pre-change | GraalJS reference | GraalPy |
|---|---:|---:|---:|
| Nodes | 1789 | 34 | 74 |
| Allocations | 41 | 1 | 1 |
| Control splits | 107 | 0 | 0 |
| Guards/deopts | 87 | 6 | 6 |
| Invokes | 30 | 0 | 0 |
| FrameState | 389 | 1 | 6 |

The historical Protos vs GraalJS node difference is 1755. These figures are graph structure, not proven timing improvement.

## Post-change acceptance pending

At this checkpoint, the most recent visible `guillermomolina/protos-benchmarks` publication is `bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff` (2026-10-08 18:34:35Z), which precedes product commit `248b097e968219452b1663b9ed0fce597a719a79` (2026-10-08 19:20:31Z). No verified post-PERF038-B capture is yet available.

Human executor should capture the **unchanged** `primitive-method-call` graph at product revision `248b097e`, preserving harness, workload, GraalVM, phase/tier, stabilization and correctness policy. Compare node families and residual invoke targets only if admission is valid. Do not infer a percent improvement from source inspection.

PERF038 / #852 remains OPEN pending valid post-change A/B and applicable functional validation evidence. No new design decision or specification change is recorded.

## Post-change graph acceptance — human-executor result

Human executor reported the following on 2026-10-09, after analyzing previously captured BGVs on the host using `python3 truffle/measure_graphs.py analyze --output results/local/perf038-b`:

```text
ANALYZED=primitive-method-call/protos/budget-16000/bgv/TruffleHotSpotCompilation-2467[ProtosSemanticBytecodeRootNodeGen@22787edc].bgv.gz
ANALYZED=primitive-method-call/protos/budget-64000/bgv/TruffleHotSpotCompilation-2434[ProtosSemanticBytecodeRootNodeGen@60ea6b82].bgv.gz
ANALYZE_FAILURES=0
UNIT=primitive-method-call/protos valid=YES total=956 graphs=1
RUNG=primitive-method-call protos=956 js=None python=None peer=UNRESOLVED protos_stabilization=STABLE protos_status=NOT_EVALUATED node_comparison=SKIPPED final_state=- signals=-
INVALID_UNITS=0
```

Prior capture reported product revision `248b097e968219452b1663b9ed0fce597a719a79`, correctness PASS, natural warmup stable pair `[16000, 64000]`, capture valid YES. The resulting node count is **956**, versus **1789** at product revision `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`: reduction **833 nodes (46.6%)**. Compared with the historical GraalJS 34-node reference, the gap is now 922 nodes. The A/B comparison is conditional on verifying that the post-change `unit.json` retains the same workload, harness, GraalVM, tier, phase, and analysis policy as the baseline. The host-local result is human-reported; it has **not yet been verified from a published post-change evidence directory**.

No new timing claim is made. Residual invoke, allocation, guard, control split and FrameState metrics require the post-change `unit.json`. Do not close issue #852 on this graph reduction alone.

## PERF038-B detailed post-change attribution (human-executor report, 2026-10-09)

Post-change product `248b097e968219452b1663b9ed0fce597a719a79`; harness `bf5af4dc1e6a30c3e614379a5b673d2cfb7c57ff`; GraalVM `25.4.4.1.1`, `After TruffleTier`, tier 2. Human executor reports valid YES, STABLE, 956 nodes. Harness SHA differs from historical baseline `98abc9af`: before any strict A/B declaration, compare source/policy hashes, graph methodology and workloads. No timing assertion.

| Family | Baseline 0.3.312 | PERF038-B 0.3.313 | Delta |
|---|---:|---:|---:|
| Nodes | 1789 | 956 | -833 |
| Allocations | 41 | 26 | -15 |
| Control-flow splits | 107 | 46 | -61 |
| Guards/deopts | 87 | 56 | -31 |
| Invokes | 30 | 15 | -15 |
| Loads | 111 | 57 | -54 |

Remaining invokes: `List.get` 1; `List.size` 2; `ProtosBytecodeRootNode.selectGuestHandlerOnRootCrossing` 2; `ProtosFrameArguments.materializeCompactActivation` 4; `ProtosLexicalBindingAuthorityCalls.contains` 5; `ProtosSemanticBytecodeRootNode$ReadRootFrameLocal.slowRead` 1.

Truffle expansion attribution (not final-graph partition): `CachedBytecodeNode` entries 325/17 ifs and 229/21 ifs; `ProtosSemanticBytecodeRootNodeGen` 54/4; `PrepareSendArguments_Node` 32/2; `FinishClosureCall_Node` 15/2; `BindClosureFrameParameter_Node` 3/0 (previously 351/28). This confirms that binding specialization nearly erased that expansion *attribution*, but does not prove how many final nodes each method saved. Residual 922 nodes above historical 34-node JS reference are suspect until justified, without deleting observable semantics.

### PERF038-C next grouped implementation scope

Focus on reducing remaining generic interpreter/call machinery: retained `CachedBytecodeNode` expansions and 15 invokes. Source-ground each elimination; preserve lexical mutation/authority, Context observation and compact-to-materialized transition, dynamic lookup and invalidation, receiver/methodHome/super, arity, error/unwind and nonlocal control. Prefer compact-path guard specialization and partial-evaluation simplification; no benchmark-specific shortcut and no speculative blanket removal of fallbacks. Human executor validates focal/full gates and captures same-workload graph after code changes, with metadata versioning only after green tests and before publish. #852 remains open.

## PERF038-C implementation publication (2026-10-09)

Verified product commit: https://github.com/guillermomolina/protos/commit/0ff9e1fb1633c3bc7e196c43001df0a7ef110058
Product version: `0.3.315-SNAPSHOT`.

Files changed: `ProtosBytecodeRootNode.java`, `ProtosFrameArguments.java`, `ProtosSemanticBytecodeRootNode.java`, new `ProtosPerf038CCompactCallSpecializationTest.java`, plus `pom.xml` and `CHANGELOG.md`.

Implementation as described by the human/agent handoff: complementary guarded specializations for compact versus materialized invocation paths (`CurrentActivation`, argument checks and reads, frame-local and captured-local reads), compact captured fallback behind one Truffle boundary, and lazy compilation of root guest-exception interception using a compilation-final flag. New tests cover compact/published transitions, D179 C0 retargeting, arity errors, guest exception crossing and nonlocal return. This is an implementation description; the commit confirms changed paths but does not itself independently prove test outcomes.

**Acceptance pending**: no post-PERF038-C graph count has yet been supplied. Baseline for this slice is the human-reported valid STABLE 956 nodes from product revision `248b097e`, harness `bf5af4dc`, GraalVM `25.4.4.1.1`, `After TruffleTier`, tier 2. Previous pre-B reference is 1789; GraalJS reference 34. Capture identical `primitive-method-call` without modifying harness. Record graph validity, correct version/revision, compilation tier, final nodes, allocations, splits, guards, invokes, loads, invoke-target changes and Truffle expansion attribution. Do not claim improvement before capture.

The `protos-benchmarks` HEAD inspected when recording this was `e14123532c7fa4fb97cdc247bfed5f1168e1fc73`; no post-C evidence was found in this check. Product issue #852 must remain open.

## Combined PERF038-C + PERF037-C follow-up capture — 2026-10-09

Human executor supplied readout from local `results/local/perf038-c/primitive-method-call/protos/unit.json`:
- Product: `d88ed6b83e977a1bf02f425825a015e9e577ec3e` (commit `PERF037-C`, two commits after PERF038-C `0ff9e1fb`; preceding intervening commit `DOC010-F`). Therefore it is a **combined result, not a PERF038-C-only measurement**.
- Harness: `175e5b2c4bf740164cdef1f02b40e2165b61afcd` (`PERF039-A`, additional control-flow workloads). The harness comparison must be recorded/verified before describing strict parity with previous revisions.
- GraalVM 25.4.4.1.1, phase After TruffleTier, tier 2, evidence valid true, correctness PASS, STABLE natural pair [16000, 64000].
- Total: **261 nodes**, 1 graph, allocations 6, control splits 12, guards/deopts 18, invokes 1, loads 21, loops 0.
- Single remaining invoke: `ProtosFrameArguments.materializeCompactActivation` (1).
- Attribution: `SelectCapturedMaterializedOwnerFrameAtRoot_Node` count 48 / ifs 8; `CachedBytecodeNode` count 40 / ifs 0; `PrepareSendArguments_Node` 32 / ifs 2; `FinishClosureCall_Node` 15 / ifs 2; `CurrentActivation_Node` 7 / ifs 0; `LoadFrameClosureArgument_Node` 5; `UncachedBytecodeNode` 5; `CheckFrameClosureArgumentUpperBound_Node` 2.
- Historical valid totals: 1789 (pre-B product d59da442), 956 (post-B product 248b097e), now 261 (combined PERF038-C/PERF037-C). Combined reduction versus post-B: 695 (72.7%); reduction from 1789: 1528 (85.4%). Historical GraalJS reference: 34, hence 227-node gap.
- No direct graph capture pinned specifically at `0ff9e1fb`, so separate causal contributions of PERF038-C versus PERF037-C **are not established**. No timing claim is made. Evidence is human-reported and local, not yet published as a complete post-change capture.

Issue #852 remains open; any follow-up should target the residual 261-node graph and avoid attributing the entire delta to PERF038-C alone.

## PERF038-D intermediate implementation published — 2026-10-09

Verified Protos main commit: https://github.com/guillermomolina/protos/commit/0c3ad1350d18008d3c30c36c30dfb7dd7df56440 (push reported `41883727..0c3ad135`). Changes include `CanonicalToBytecodeLowerer`, `ProtosBytecodeRootNode`, `ProtosFrameArguments`, `ProtosFrameLexicalBindingAuthority`, `ProtosSemanticBytecodeRootNode`, runtime `ProtosActivation`, `ProtosLexicalBindingAuthority`, `ProtosObjectValue`, `ProtosPerf038DPrimitiveMethodCallTest`, and two PE baselines. No `pom.xml` or `CHANGELOG.md` change in this commit.

Human executor reports `git diff --check` clean and all local tests PASS (test names/output not independently inspected).

This is the PERF038-D intermediate A/C/D/E implementation checkpoint: captured owner frame cache with authority-installation retirement; compact caller reference/provenance path; guarded send home selection and reduced compact-carrier validation; regression tests for error handlers, Context, lexical owner/binding changes and durable handoff. B (generic bytecode dispatch) and F (residual allocations/splits/guards/loads) are not yet claimed complete.

**Post-D structural graph not measured yet.** Accepted preceding combined PERF038-C/PERF037-C graph baseline is `261` at product `d88ed6b83e977a1bf02f425825a015e9e577ec3e`, harness `175e5b2c4bf740164cdef1f02b40e2165b61afcd`, GraalVM `25.4.4.1.1`, After TruffleTier tier 2, valid/STABLE/PASS. Graph peer reference GraalJS `34`. Human executor should capture `primitive-method-call` with product commit `0c3ad135` and unchanged policy; analyze BGV on host and publish evidence if valid. Do not infer graph improvement or timing improvement from commit alone. Keep #852 open.

## PERF038-D structural graph publication — 2026-10-09

Harness evidence committed to [protos-benchmarks @ 6160b0a](https://github.com/guillermomolina/protos-benchmarks/commit/6160b0a0808a2d9391748bf65eba4667c958cad9): `results/perf038-d-8d863cbb-graphs/primitive-method-call/protos/`, including `unit.json`, `capture.json`, stable budget-16000 and budget-64000 `.bgv.gz` and `.filter.json.gz`. Product `8d863cbb05d539c451d99add54c002bd3451a0ef`; harness `e4e6225dae5a51792c79c9267cf46c9dc404e71a`, GraalVM `25.4.4.1.1`, After TruffleTier tier 2, valid YES, STABLE [16000,64000], correctness PASS. **196 nodes**; allocations 6, splits 7, guards/deopts 21, invokes 1 (ProtosFrameArguments.materializeCompactActivation), loads 11, loops 0. Versus prior 261 nodes: -65 (-24.9%). GraalJS historical comparison 34 nodes; excess 162.

Verified full graph node-class histogram counts: Protos FrameState 25 vs JS 1, ConstantNode 33 vs JS 6, BeginNode+EndNode 25 vs JS 0, FixedGuardNode 18 vs JS 5, InstanceOfNode 10 vs JS 1, IsNullNode 8 vs JS 1, IfNode 7 vs JS 0, VirtualArrayNode+VirtualInstanceNode 8 vs JS 0, LoadIndexedNode 7 vs JS 3. Counts are exact per reported unit.json but class-by-class deltas are not necessarily independently removable.

Truffle attribution residual: PrepareSendArguments_Node 71, CachedBytecodeNode 23, SelectCapturedMaterializedOwnerFrameAtRoot_Node 10, LoadFrameClosureArgument_Node 5; other attributed groups smaller. Priorities: simplify/virtualize PreparedClosureCall and compact arg carrier, shrink guards/exception/frame states, remove residual materializeCompactActivation invoke, eliminate surviving cached-bytecode scaffolding. No causal edge-by-edge attribution has been verified from compressed graph because current GitHub text connector cannot read binary gzip assets. Do not claim complete node-by-node correspondence until these are decoded.
