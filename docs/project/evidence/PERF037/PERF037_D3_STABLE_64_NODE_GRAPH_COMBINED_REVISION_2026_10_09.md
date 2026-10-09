# PERF037-D3 — Stable 64-node graph on combined PERF037-D + PERF038-F product revision (2026-10-09)

**Work item:** [`guillermomolina/protos#851`](https://github.com/guillermomolina/protos/issues/851). **Status:** OPEN; structural parity to GraalJS is not achieved.

## Exact measurement identity

The maintainer supplied the completed `measure_graphs.py analyze`, `summarize`, `verify` and `unit.json` selected-class histogram output. This report records **human-supplied console evidence**. **Raw BGV, filtered JSON, capture/unit metadata and trace logs are now published and verified** in [`guillermomolina/protos-benchmarks@dd8b6518057de62d940a10e5b3db1de7ea97929b`](https://github.com/guillermomolina/protos-benchmarks/commit/dd8b6518057de62d940a10e5b3db1de7ea97929b).

- **Measured Protos revision:** [`89e1b038c2fd510f47ddb991496882548d1bbe44`](https://github.com/guillermomolina/protos/commit/89e1b038c2fd510f47ddb991496882548d1bbe44), product main HEAD when reviewed; commit subject `PERF038-F: prelude-split inherited send, length-only argument guards and class-profiled continuation check`.
- **Parent:** [`46e3fca41868983b69971978baf04154ddb05e1f`](https://github.com/guillermomolina/protos/commit/46e3fca41868983b69971978baf04154ddb05e1f), `PERF037-D: owner absence specialization and presence continuity in owner-frame cache`. GitHub confirms the measured product is exactly one commit after the isolated PERF037-D publication.
- **Published capture directory:** [`results/perf037-d-89e1b038-graphs/primitive-object-slot-read/protos/`](https://github.com/guillermomolina/protos-benchmarks/tree/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs/primitive-object-slot-read/protos/). The maintainer corrected the previous misleading local directory name prior to publication. **The actual `unit.json.protos_revision` is `89e1b038...`**, not the intended standalone `46e3fca4`.
- **Workload:** `primitive-object-slot-read`, `language=protos`, `stage=reference`, same configured `canonical` surface and selected `After TruffleTier` compiler-IR phase as previous PERF037-D1 graph. This is a structural graph measurement, **not latency timing**.
- **IGV analysis:** 2 BGVs analyzed at selected budgets `16000` and `64000`, `ANALYZE_FAILURES=0`.
- **Summarization:** `UNIT=primitive-object-slot-read/protos valid=YES total=64 graphs=1`, `protos_stabilization=STABLE`, `INVALID_UNITS=0`. Solo-Protos unit reported `peer=UNRESOLVED` and `node_comparison=SKIPPED` because unchanged peers were not remeasured; **this is not an admission failure**.
- **Producer verification:** `CASES=1`, `WORKING_TREE_MATCHES_PRODUCER=YES`, `HEAD_MATCHES_PRODUCER=YES`.
- **Exact `unit.json` output:** `PRODUCT_REVISION=89e1b038c2fd510f47ddb991496882548d1bbe44`, `EVIDENCE_VALID=True`, `STABILIZATION=STABLE`, `NODES=64`.
- **Actual benchmark producer revision:** `a599949830cc7a240ffd347de615006c112c94d6` (now verified from the published `unit.json` and `capture.json`); the evidence-retention commit is `dd8b6518057de62d940a10e5b3db1de7ea97929b`.

## Structural comparison: published D1 vs new D3

| Graph | Revision | Nodes | Evidence |
| --- | --- | ---: | --- |
| PERF037-D1 | `3e94ba7d01cf94852555d3daa18a3208d3908e35` | 88 | Published [D1 raw evidence](https://github.com/guillermomolina/protos-benchmarks/tree/a599949830cc7a240ffd347de615006c112c94d6/results/perf037-d-3e94ba7d-graphs) |
| PERF037-D3 + PERF038-F | `89e1b038c2fd510f47ddb991496882548d1bbe44` | 64 | [Published full BGV + filtered IR at `dd8b6518`](https://github.com/guillermomolina/protos-benchmarks/tree/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs) |
| GraalJS reference | Published `global-20261008-graphs` reference | 36 | Existing stable 36-node baseline; no peer recapture |

**Observed total:** 88 → 64, **−24 nodes (−27.3%)**. Remaining net gap to JS=36 is **28 nodes**. This is **the combined revision delta**, not a causally isolated effect of PERF037-D.

### Complete selected-phase class histogram, supplied with the new valid unit

| Compiler IR class | Before `3e94ba7d` | New `89e1b038` | Delta |
| --- | ---: | ---: | ---: |
| `BeginNode` | 2 | 0 | −2 |
| `ConstantNode` | 25 | 20 | −5 |
| `EndNode` | 2 | 0 | −2 |
| `FixedGuardNode` | 8 | 6 | −2 |
| `FrameState` | 11 | 6 | −5 |
| `GuardedUnsafeLoadNode` | 1 | 1 | 0 |
| `IfNode` | 1 | 0 | −1 |
| `InstanceOfNode` | 2 | 2 | 0 |
| `IntegerEqualsNode` | 2 | 1 | −1 |
| `IsNullNode` | 2 | 1 | −1 |
| `LoadFieldNode` | 2 | 2 | 0 |
| `LoadIndexedNode` | 3 | 2 | −1 |
| `MergeNode` | 1 | 0 | −1 |
| `NarrowNode` | 1 | 1 | 0 |
| `ObjectEqualsNode` | 2 | 2 | 0 |
| `ParameterNode` | 1 | 1 | 0 |
| `PiArrayNode` | 1 | 1 | 0 |
| `PiNode` | 4 | 4 | 0 |
| `RawLoadNode` | 1 | 1 | 0 |
| `ReturnNode` | 1 | 1 | 0 |
| `StartNode` | 1 | 1 | 0 |
| `TrufflePreserveFrameStateNode` | 1 | 1 | 0 |
| `ValuePhiNode` | 1 | 0 | −1 |
| `VirtualArrayNode` | 4 | 4 | 0 |
| `VirtualInstanceNode` | 1 | 1 | 0 |
| `VirtualObjectState` | 7 | 5 | −2 |
| **TOTAL** | **88** | **64** | **−24** |

The `If/Begin/End/Merge/ValuePhi` island targeted by PERF037-D is no longer present as those compiler-IR node classes, and one `LoadIndexedNode` and one `IntegerEqualsNode` also disappeared. **This is strong structural consistency**, but a graph without these class instances does not by itself isolate causation across concurrent PERF038-F changes. Individual previous BGV IDs (`If #328`, `LoadIndexed #320`, etc.) are **not** reusable as identities for this new BGV.

New D3 still has `FrameState=6`, `VirtualObjectState=5`, `VirtualArrayNode=4`, `TrufflePreserveFrameStateNode=1`. It therefore retains nontrivial Bytecode DSL frame/deopt machinery not present in the stable 36-node JS baseline.

## Cross-commit attribution caveat and next work

The measured product SHA `89e1b038` is **one commit after** the PERF037-D commit `46e3fca4`. PERF038-F changes `ProtosFrameArguments.java` and `ProtosSemanticBytecodeRootNode.java` in addition to its own method-call work. These files are also involved in generic argument/continuation handling and can affect the measured workload. It is **not scientifically justified** to attribute all −24 nodes to PERF037-D on current evidence.

To determine the isolated PERF037-D delta without changing main HEAD or overwriting published evidence, capture `primitive-object-slot-read` in a separate clean, detached worktree at product SHA `46e3fca4`, with a **different output directory**; analyze/summarize/verify using the same pinned harness. No JS/Python recapture. When both valid graphs are published, attribute root-state, owner-check, typed-load, and return node-class changes by exact graph source/edges; only then give a numeric PERF037-D-specific reduction.

## Coordination status

- Product `46e3fca4` was pushed and the human executor reported `git diff --check` clean and all local tests PASS.
- Product `89e1b038` was confirmed to be the child of `46e3fca4` with PERF038-F source changes.
- New 64-node raw graph **has been published** in `guillermomolina/protos-benchmarks@dd8b6518057de62d940a10e5b3db1de7ea97929b`. The coordinator independently verified the remote file listing and the source-identity/validity fields in remote `unit.json` and `capture.json`; a full node-edge audit of the published filtered IR has not yet been performed.
- PERF037/#851 **remains OPEN** pending further causal isolation, structural parity or justified acceptance criteria, and a separate valid runtime measurement.
