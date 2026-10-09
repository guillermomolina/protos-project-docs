# PERF037-D2 — Published absence specialization and owner presence-continuity cache (2026-10-09)

**Issue:** [`guillermomolina/protos#851`](https://github.com/guillermomolina/protos/issues/851) — OPEN.

**Production commit (published):** [`guillermomolina/protos@46e3fca41868983b69971978baf04154ddb05e1f`](https://github.com/guillermomolina/protos/commit/46e3fca41868983b69971978baf04154ddb05e1f), commit subject `PERF037-D: owner absence specialization and presence continuity in owner-frame cache`, version **`0.3.321-SNAPSHOT`**. The implementation was committed 2026-10-09; evidence was recorded afterward.

## Scope and proof-bearing source paths

The actual published changeset (GitHub commit diff, not a proposed patch) modifies:

1. `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
2. `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
3. `src/test/java/com/guillermomolina/protos/execution/ProtosPerf037CCapturedLexicalReadTest.java`
4. `tools/java_local_accessor_pe_baseline.json`
5. `pom.xml`
6. `CHANGELOG.md`

### Change A — Monotonic absence specialization

`CapturedNearerScopeAbsence.profileNoSelection()` calls `transferToInterpreterAndInvalidate()` before first setting the `@CompilationFinal seenNoSelection` flag. The helper is used by both the ordinary selection fallback and the cached-hit cleared-binding path. It does not change any owner, lexical authority, frame or binding. Subsequent absence observations do not re-invalidate for this particular profile.

### Change B — Presence continuity admitted once, rechecked by assumption

`CapturedOwnerFrameCache.Entry` gains an optional `presentContinuity` `Assumption`. The existing `@TruffleBoundary record(...)` obtains a token by exact `storedLayout().offsetOf(name)`, validates it before and after checking `!accessor.isCleared(bytecodeNode, ownerFrame)`, and admits none if absent or invalid. A cached hit still verifies `owner == current.owner()` and `current.installed().isValid()`; if the admitted token remains valid, it returns the selected materialized frame without another physical presence check. Otherwise it checks `accessor.isCleared` and profiles any absence. The actual **value** is still read using Bytecode DSL builtin `LoadLocalMaterialized` (preserving BUG018's physical local-kind behavior).

The semantic continuity token predates this change: `ProtosFrameLexicalLayout.presentContinuityAt(ordinal)`. `ProtosFrameLexicalBindingAuthority.removeBinding` invalidates it before physical clearing, and recreation does not renew it. `ProtosSemanticBytecodeRootNode.SelectCapturedMaterializedOwnerFrameAtRoot.compact` now forwards the constant name needed for the exact layout ordinal. No global token registry or benchmark-specific special case is introduced.

### Regression coverage and guarded access

The updated test class covers shared owner read sites, repeated removal and `SlotNotFound`, recreation with `PRESENT(null)`, a monotonic no-selection flag, shared layout-continuity invalidation by a *different* activation, physically present cached frame after token invalidation, absent-at-recording behavior, and uncached/retired cache behavior. The Java accessor partial-evaluation baseline adds `admittedPresentContinuityOrNull` with `BOUNDARY_CUT`, updates the cached-hit method signature, and retains the relevant access guards.

## Reported validation and limitations

The human executor explicitly reported on 2026-10-09:
- `git diff --check` **clean**.
- **All local tests PASS**.
- Commit `46e3fca4` **pushed**.

The coordinator independently verified the public commit SHA, file list, version/changelog and source delta through GitHub. The coordinator **did not execute local builds or tests**, and no per-test log or numeric runtime score was supplied. The status above is attributed to the human report; it is not a claim of independently reproduced execution.

## Graph comparison: planned, not yet measured

| Evidence | Protos revision | Selected phase | Graph nodes | Status |
| --- | --- | --- | ---: | --- |
| Published pre-change D1 | `3e94ba7d01cf94852555d3daa18a3208d3908e35` | `After TruffleTier` | **88** | `STABLE`, `EVIDENCE_VALID=True` |
| Published JS peer | JS producer from `global-20261008-graphs` | `After TruffleTier` | **36** | Stable reference; do not remeasure unchanged JS |
| New D2 product | `46e3fca41868983b69971978baf04154ddb05e1f` | `After TruffleTier` | **NOT MEASURED** | Awaiting new Protos-only reference capture |

D1 raw benchmark graph publication: [`protos-benchmarks@a599949830cc7a240ffd347de615006c112c94d6/results/perf037-d-3e94ba7d-graphs/`](https://github.com/guillermomolina/protos-benchmarks/tree/a599949830cc7a240ffd347de615006c112c94d6/results/perf037-d-3e94ba7d-graphs). Full causal D1 audit: [PERF037_D_FULL_GRAPH_CAUSAL_AUDIT_2026_10_09.md](PERF037_D_FULL_GRAPH_CAUSAL_AUDIT_2026_10_09.md).

The expected targets are the former D1 `IfNode` / `BeginNode` / `EndNode` / `MergeNode` / `ValuePhiNode` owner-absence island, plus presence-related `LoadIndexed #320` and `IntegerEquals #325` (IDs refer **only** to D1; node IDs must be remapped before making claims about the new graph). Whether these nodes actually disappear, their replacement guards/deopt states, total net node reduction and performance are all **unknown until new valid steady graph evidence is analyzed**.

## Human-executor measurement handoff

Do not mix environments: within the benchmark **devcontainer**, Protos checkout is `/workspaces/protos`; on the **host**, the benchmark checkout is `~/Fuentes/protos-benchmarks` and Docker is available.

**Capture, inside benchmark devcontainer**, after checking product `HEAD == 46e3fca41868983b69971978baf04154ddb05e1f` and clean:
```sh
cd /workspaces/protos-benchmarks
python3 truffle/measure_graphs.py capture --dir /workspaces/protos --stage reference --workload primitive-object-slot-read --language protos --output /workspaces/protos-benchmarks/results/perf037-d-46e3fca4-graphs
```

**Analyze, then summarize and verify, on Docker-capable host**:
```sh
cd ~/Fuentes/protos-benchmarks
python3 truffle/measure_graphs.py analyze --output results/perf037-d-46e3fca4-graphs
python3 truffle/measure_graphs.py summarize --output results/perf037-d-46e3fca4-graphs
python3 truffle/measure_graphs.py verify --output results/perf037-d-46e3fca4-graphs
```

This is **structural graph capture, not latency timing**. Retain and publish new raw result evidence only after verifying valid capture, source identities, stability and analyzer results. Keep PERF037/#851 **OPEN** pending node/graph and latency acceptance; do not claim 36-node convergence or a reduction before analysis.
