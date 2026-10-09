# PERF037-C — Published captured-lexical optimization, awaiting graph remeasurement (2026-10-09)

**Role:** revision-coupled non-normative implementation-publication checkpoint. This document is not a new design approval or benchmark result.

- **Owning Issue:** [PERF037 / protos#851](https://github.com/guillermomolina/protos/issues/851).
- **Published product revision:** [`d88ed6b83e977a1bf02f425825a015e9e577ec3e`](https://github.com/guillermomolina/protos/commit/d88ed6b83e977a1bf02f425825a015e9e577ec3e), commit message `PERF037-C: specialize captured lexical reads and reduce root graph overhead (#851)`; verified on public `main` on 2026-10-09.
- **Immediately preceding PERF037-B revision:** [`1c02b5d11c6e771910b22e213bc5fa76bc5315f7`](https://github.com/guillermomolina/protos/commit/1c02b5d11c6e771910b22e213bc5fa76bc5315f7). [Bounded B record](PERF037_B_PUBLISHED_IMPLEMENTATION_AND_GRAPH_EVIDENCE_2026_10_09.md).
- **Benchmarks repository HEAD at coordination:** [`175e5b2c4bf740164cdef1f02b40e2165b61afcd`](https://github.com/guillermomolina/protos-benchmarks/commit/175e5b2c4bf740164cdef1f02b40e2165b61afcd), subsequent to the retained original baseline corpus. The next capture must record its *actual* producer revision; the harness revision here is not a claim that a capture has occurred.

## Implementation delta

The public PERF037-C commit changes the following 13 tracked paths:

- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosPerf037CCapturedLexicalReadTest.java`
- `tools/java_generated_bytecode_bci_pe_baseline.json`
- `tools/java_local_accessor_pe_baseline.json`
- `tools/java_local_range_operand_pe_baseline.json`
- `tools/java_local_range_pe_guard_baseline.json`
- `tools/java_local_range_pe_reachability_baseline.json`
- `pom.xml`
- `CHANGELOG.md`

The implementation agent reports one grouped source optimization:

1. **C1 — captured-nearer-scope absence:** a lazy layout-scoped, one-way no-dynamic-binding `Assumption` and an unrolled `CapturedNearerScopeAbsence` proof. The optimized compact read avoids per-scope nominal membership lookups while a compatible frame-backed authority and the relevant absence proofs remain valid. On failed proof, it permanently retires and uses the exact generic walk. The proof does not cache the slot value or owning frame.
2. **C2 — constant bytecode operands:** `name` and `lexicalDepth` become constant operands in three root-level operations. Selection / conditional / `LoadLocalMaterialized` / fallback remains explicitly separate to preserve BUG018 tier-safe retained-frame access.
3. **C3 — compact versus materialized paths:** the Bytecode DSL specializes compact-frame and materialized-activation reads separately; the compact check reuses the PERF038-B argument-0 discriminator. The uncached interpreter falls back to exact generic evaluation.
4. **C4 — generated root cost:** optimization is through C2/C3; `ReadMemberAtRoot`, inline-callback paths and non-root activation paths are intentionally unchanged.

These are implementation-scope descriptions and are **not yet evidence of a reduced compiled graph**.

## Validation and semantic constraints

The maintainer explicitly reports for the **published** PERF037-C slice:

```text
git diff --check = CLEAN
all local tests = PASS
git push = COMPLETED
```

The coordinator has **not** executed local tests, compilation, static guards or benchmarks. No individual test counts or immutable local test transcript have been supplied. The new regression source `ProtosPerf037CCapturedLexicalReadTest` was added in the commit.

No normative semantics changes are requested or claimed: D179 late creation, removal and recreation; scope shadowing; `PRESENT(null)` vs `ABSENT`; deferred/materialized context transitions; exact lexical authority; escaped retained frames and BUG018 tier coherence; and generic fallbacks remain required.

## Previous graph evidence, not a PERF037-C result

Prior **valid, stabilized** `primitive-object-slot-read` graph evidence:

| Runtime | Product revision | After TruffleTier total nodes | Outgoing invokes | Loops | FrameState |
| --- | --- | ---: | ---: | ---: | ---: |
| Protos before B | `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae` | 683 | 14 | 2 | 126 |
| Protos after B | `1c02b5d11c6e771910b22e213bc5fa76bc5315f7` | **253** | **3** | **1** | **31** |
| GraalJS reference | peer source at old baseline | 36 | 0 | 0 | 1 |
| GraalPy reference | peer source at old baseline | 103 | 0 | 0 | 10 |

Every one of the three post-B outgoing invokes was reported as `ProtosLexicalBindingAuthorityCalls.contains`. Presence of these calls in a compiled graph does not establish that each executes on the successful hot path.

## Next human-executor measurement

**Pending:** only `protos`, reference-stage `primitive-object-slot-read` using the unchanged graph harness, with product `/workspaces/protos`, clean HEAD expected to be `d88ed6b83e977a1bf02f425825a015e9e577ec3e`.

Proposed fresh capture path in the benchmarks checkout:
`results/perf037-c-d88ed6b8-graphs/`.

Required post-capture steps: `graphs-capture`, `graphs-analyze`, `graphs-summarize` and `graphs-verify`, with `GRAPH_LANGUAGE=protos`. Peers must not be rerun unnecessarily; compare Protos' new validated unit with the previous immutable JS/Python references after checking harness policy/workload compatibility.

**After-C nodes, invokes, loops, FrameState and latency remain NOT_MEASURED as of this record.** A summary over Protos alone does not contain a newly measured three-language matrix; comparison requires the retained peer units.

**PERF037/#851 stays open**. The 90% structural reduction target (at most 68 from the original 683 nodes) and accepted timing/closure conditions are not yet demonstrated. Raw new BGVs have not been published to `protos-benchmarks` at this checkpoint.
