# PERF043 — Published canonical Integer folding, graph evidence and closure

Date: 2026-10-13

Formal authority: [PERF043/#877](https://github.com/guillermomolina/protos/issues/877).

**Disposition:** completed bounded implementation and retained graph-evidence scope. No proven steady-state timing speedup is claimed.

## Exact publication identity

| Item | Pinned revision |
| --- | --- |
| Protos product | [`8f7e5ea5b7ad75f8052adf7ce51945539348b4a1`](https://github.com/guillermomolina/protos/commit/8f7e5ea5b7ad75f8052adf7ce51945539348b4a1) |
| Product implementation version | `0.3.344-SNAPSHOT` |
| Protos-benchmarks evidence publication | [`1b28b390b61d5a59e9b1573ea30e26306a6dde86`](https://github.com/guillermomolina/protos-benchmarks/commit/1b28b390b61d5a59e9b1573ea30e26306a6dde86) |
| Graph producer harness HEAD (clean) | `99ba341d88091b5bf0c76f18039e18912e1f611c` |
| Historical peer graph harness producer | `98abc9af7a05a45a7d4056b72f36aef5889cb76f` |
| Earlier I095 integrated into product | `57a43b6d94697f9a6256e36c7a775dfa4a1218d7` |

The older [2026-10-12 source investigation](PERF043_SOURCE_COMPARISON_AND_FOLDING_CANDIDATE_2026_10_12.md), pinned to Protos `18f71b2a`, remains an immutable *pre-implementation* checkpoint; its original pending-status text is historical, not current.

## Implementation outcome and constraints

The published product changes `CanonicalToBytecodeLowerer.java` and `ProtosSemanticBytecodeRootNode.java` to recognize and propagate provable canonical Integer-add constants in Bytecode lowering. It handles literal and nested literal additions and admitted straight-line scalar-local values; source `CallTag` observability is retained. `ProtosPerf025CompactCalleeExecutionTest.java` and `protos/tests/conformance/integer/arithmetic-and-unary.protos` cover the admitted and fallback behavior. `pom.xml` and `CHANGELOG.md` carry the matching version.

This is **not an unrestricted source-level `1 + 2 -> 3` substitution**. The `+` operation is an ordinary, overridable D013 message. The existing canonical-selection/context/Prelude/stability-assumption boundary remains authoritative; dynamic, invalidated or otherwise unproven operations retain ordinary message dispatch. Compact scalar results must fall back correctly if a debugger materializes and mutates the activation. The maintainer-supplied regression example observed `3` normally and `42` after a breakpoint changed `left` from `1` to `40`. Exact Integer semantics, overflow promotion and the noncanonical path are retained as validation boundaries.

The product publication combined the earlier WIP PERF043 implementation with I095 without rewriting I095. Temporary investigation-only `target/perf043_*.py` scripts were excluded from the squashed product `main` commit. No semantic change is ratified by this project record.

## Final retained compiled graph

[Evidence directory](https://github.com/guillermomolina/protos-benchmarks/tree/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs) contains [README](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs/README.md), [capture.json](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs/primitive-integer-add/protos/capture.json), [unit.json](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs/primitive-integer-add/protos/unit.json), [matrix.json](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs/matrix.json), plus the retained raw gzipped trace logs, BGV captures, analyzed BGV output, and capture marker.

The `primitive-integer-add` unit is the canonical source `run: () => { left: 1; right: 2; left + right }`:

- Product revision: `8f7e5ea5b7ad75f8052adf7ce51945539348b4a1`; clean product before and after capture.
- Producer revision: `99ba341d88091b5bf0c76f18039e18912e1f611c`; harness `dirty=false`.
- Workload SHA-256: `ae8ba162beb092793920a40655bc10d1ff57b4dc62edbae370ad86c0d1e97aff`.
- Product source-state SHA-256: `8c84caf84fa88fe87e466e92eecc64fed942d7351ecb4260d7f9f14b35fb4fce`.
- Correctness: **PASS**, actual `3`. Capture valid, `evidence_valid=true`.
- Stable Tier-2 compilation in `After TruffleTier`, natural-warmup stable pair **[16000, 64000]**, one language-owned graph with **42 relevant nodes**.
- Graph families: control-flow splits `0`, guards/deopts `0`, invokes `0`, loops `0`, loads `2`, allocation-related nodes `2` (a structural graph-family count, **not** evidence of two physical allocations).
- Dominant exact classes include `ConstantNode=13`, `FrameState=6`, `VirtualObjectState=5`, `VirtualArrayNode=4`, `PiNode=3`, `LoadIndexedNode=2`, `VirtualInstanceNode=2` and `ReturnNode=1`.
- Environment: x86_64 Linux `7.2.9-arch1-1`, Intel Xeon Platinum 8362 CPU, GraalVM CE `25.4.4.1.1+1.1`, runtime JDK `25.0.4.1.1+1-jvmci-25.4-b23`. Exact JVM flags, host CPU/affinity, graph policy and toolchain are recorded in capture metadata.

The maintainer reported `ANALYZE_FAILURES=0`, `INVALID_UNITS=0`, `WORKING_TREE_MATCHES_PRODUCER=YES` and `HEAD_MATCHES_PRODUCER=YES`. Integrated product validation was **`make test=PASS` and `make check=PASS`**, with `git diff --check` clean. These are maintainer-run observations, not tests executed by the coordinating agent.

### Historical context, not matched peer remeasurement

| Reference unit | After TruffleTier nodes | Evidence |
| --- | ---: | --- |
| Published Protos 0.3.344 | **42** | [Pinned current capture](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf043-344-graphs/primitive-integer-add/protos/unit.json) |
| GraalJS, 2026-10-08 | **14** | [Historical JS unit](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/global-20261008-graphs/primitive-integer-add/js/unit.json) |
| GraalPy, 2026-10-08 | **49** | [Historical Python unit](https://github.com/guillermomolina/protos-benchmarks/blob/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/global-20261008-graphs/primitive-integer-add/python/unit.json) |
| Earlier Protos 0.3.342 | **2,771** | [PERF040 separate reference](https://github.com/guillermomolina/protos-benchmarks/tree/1b28b390b61d5a59e9b1573ea30e26306a6dde86/results/perf040-342-graphs/primitive-integer-add/protos) |

Historical JS and Python units were independently `evidence_valid=true`, `STABLE`, phase `After TruffleTier`, tier 2, at harness `98abc9af7a05a45a7d4056b72f36aef5889cb76f`, originally recording product `d59da442fd9bc6cf590fb9b2c82d3ff55c0391ae`. The peers were not recaptured on the 0.3.344 product. Consequently the new single-language matrix correctly reports `peer_reference=UNRESOLVED`, `protos_status=NOT_EVALUATED`, and `peer_node_comparison=SKIPPED`. `FIRST_DIVERGENT_RUNG=NONE` is not cross-language convergence when peer evidence is absent from that matrix.

The historical Protos difference (2,771 -> 42 nodes) is a real structural observation across **different revisions**, not an isolated proof of PERF043-only causation because I095 and source state changed. Fewer graph nodes are **not** a measured reduction in latency. No timing speedup or universal Closure-inlining result is asserted.

## Closure boundary

PERF043/#877's bounded source transformation, integrated correctness checks, exact-version publication and stable, raw-retained graph proof are completed. The independently scoped PERF040 numeric arithmetic and I095/PLAT057 general Closure inlining are not closed or reassigned by this record. Live formal closure and status belong exclusively to [PERF043/#877](https://github.com/guillermomolina/protos/issues/877).
