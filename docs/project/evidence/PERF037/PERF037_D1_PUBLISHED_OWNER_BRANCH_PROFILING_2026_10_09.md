# PERF037-D1 — Published owner-frame branch profiling, graph capture pending (2026-10-09)

**Purpose:** Revision-linked, non-normative publication checkpoint for the first implementation within PERF037-D. This record distinguishes the already-validated PERF037-C baseline from a PERF037-D graph measurement that **has not yet occurred**.

## Identifiers and exact published source

- **Owning live Issue:** [`guillermomolina/protos#851`](https://github.com/guillermomolina/protos/issues/851), PERF037; Issue remains **OPEN**.
- **Product commit verified on `main`:** [`3e94ba7d01cf94852555d3daa18a3208d3908e35`](https://github.com/guillermomolina/protos/commit/3e94ba7d01cf94852555d3daa18a3208d3908e35), `PERF037-D: profile captured owner-frame selection branches (#851)`.
- **Benchmark harness repository `main` verified during this checkpoint:** [`d61317d5f2990283800d17c7229c70eac6a9c2d0`](https://github.com/guillermomolina/protos-benchmarks/commit/d61317d5f2990283800d17c7229c70eac6a9c2d0), `PERF037-C: retain stable object-slot-read graph evidence (#851)`. The next capture must record its own actual producer identity.
- **Previous implementation baseline:** [PERF037-C stable graph evidence](PERF037_C_STABLE_GRAPH_RESULT_2026_10_09.md) and product `d88ed6b83e977a1bf02f425825a015e9e577ec3e`.

## Product files committed

The published commit touches exactly eight files:

- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalEnvironment.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf037CCapturedLexicalReadTest.java`
- `tools/java_local_accessor_pe_baseline.json`
- `pom.xml`
- `CHANGELOG.md`

The optimization changes `CapturedNearerScopeAbsence` to record **monotonic per-site control-flow profiles** for owner representations and no-selection outcomes. A never-taken branch calls `transferToInterpreterAndInvalidate()` before setting its profile bit, allowing the first optimized compilation to omit the corresponding branch and a later one to include it if actually observed. The proof still selects each invocation's current owner and re-reads authority, retained frame, and `MaterializedLocalAccessor.isCleared` state; it does not cache the owner frame or binding value. Root read specialization wiring is adjusted to use the profile.

Read-only runtime accessors allow probing whether a captured owner's Context exists without causing materialization. The uncached profile starts with all outcomes marked seen and does not write its profile state. The original fallback, D179 capture-by-reference/late changes, shadowing, `PRESENT(null)` versus `ABSENT`, and BUG018-safe materialized local read remain necessary.

Two additional tests in `ProtosPerf037CCapturedLexicalReadTest` are reported: `ownerRepresentationProfilesStayExactAfterWarmUp` and `alternatingOwnerRepresentationsInvalidateOncePerProfile`, covering changes of owner representation and repeated alternation across one read site. Their existence is verified in the published test source; test execution is reported by the human, not independently rerun by the coordinator. The deferred-but-published owner representation was described as not separately covered by the alternating test.

## Human-reported validation

```text
PUBLISHED_PRODUCT=3e94ba7d01cf94852555d3daa18a3208d3908e35
GIT_DIFF_CHECK=CLEAN (maintainer report)
LOCAL_TESTS=PASS (maintainer report)
COORDINATOR_BUILD_OR_TEST_EXECUTION=NONE
```

No local log, separate class PASS counts, or Graal compiled graph was supplied for PERF037-D. Neither a node reduction nor latency gain is claimed before a new capture is independently examined.

## Baseline graph and current target

| Provenance | Nodes | Invokes | Loops | Allocations | FrameState |
| --- | ---: | ---: | ---: | ---: | ---: |
| Protos original, `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` | 683 | 14 | 2 | 17 | 126 |
| Protos PERF037-B, `1c02b5d11c6e771910b22e213bc5fa76bc5315f7` | 253 | 3 | 1 | 0 | 31 |
| Protos PERF037-C, `d88ed6b83e977a1bf02f425825a015e9e577ec3e` | **151** | **0** | **0** | **0** | **12** |
| Protos PERF037-D1, `3e94ba7d01cf94852555d3daa18a3208d3908e35` | **NOT_MEASURED** | **NOT_MEASURED** | **NOT_MEASURED** | **NOT_MEASURED** | **NOT_MEASURED** |
| Unchanged GraalJS reference | **36** | — | — | — | — |
| Unchanged GraalPy reference | 103 | — | — | — | — |

After C, `SelectCapturedMaterializedOwnerFrameAtRoot_Node` had **48 attributed self nodes**, `CachedBytecodeNode` 23, and `ReadMemberAtRoot_Node` 5. The D1 branch-profile optimization was aimed at that 48-node owner-selector region. Its impact is unknown.

**Acceptance target is parity with the GraalJS reference: at most 36 nodes for this exact comparable workload and evidence policy, without weakening Protos semantics, making benchmark-specific shortcuts or hiding cost behind host boundaries.** The earlier 68-node threshold is not the owner's current requested acceptance target.

## Next human measurement and residual work

Run a **fresh clean-product reference capture, Protos only**, from the maintainer's `guillermomolina/protos-benchmarks` checkout with:

```text
PROTOS_CHECKOUT=$HOME/Fuentes/protos
GRAPH_WORKLOAD=primitive-object-slot-read
GRAPH_LANGUAGE=protos
GRAPH_OUTPUT=results/perf037-d-3e94ba7d-graphs
EXPECTED_PRODUCT_REVISION=3e94ba7d01cf94852555d3daa18a3208d3908e35
REFERENCE_BASELINE_PROTOS=151
REFERENCE_GRAALJS=36
```

Use `graphs-capture`, `graphs-analyze`, `graphs-summarize`, `graphs-verify`, verifying `PRODUCT_CLEAN=True`, exact product revision, stable/admissible unit, and source/producer match. No `--allow-dirty-product` should be necessary for the published clean checkout. No remeasurement of unchanged GraalJS or GraalPy references is requested. Inspect remaining node classes, operation attribution, invokes, guards, `FrameState`, loads and loops from the newly retained local `unit.json` to select the **next implementation within the still-open PERF037-D work**, not to declare D finished merely because its first branch-profile change was published.

New raw BGV evidence and latency for D1 have **not** yet been recorded here, and cannot be linked as published artifacts until the maintainer publishes them in the benchmarks repository.
