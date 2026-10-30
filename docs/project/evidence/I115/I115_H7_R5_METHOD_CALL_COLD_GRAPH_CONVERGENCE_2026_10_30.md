# I115-H7-R5 — Method-call cold graph convergence (2026-10-30)

> **Experimental, non-normative evidence.** I115 intentionally allows recorded divergences from Protos semantics in order to reproduce GraalPy's physical implementation. Compiler-IR node counts and machine-code bytes are **not** execution-speed measurements.

## Immutable product and workload identities

- Formal owner: [I115 #931](https://github.com/guillermomolina/protos/issues/931), with method-call follow-up in [PERF038 #852](https://github.com/guillermomolina/protos/issues/852).
- R5 product publication: [`guillermomolina/protos@cb51532254d177dac909021cefb47283f8331a69`](https://github.com/guillermomolina/protos/commit/cb51532254d177dac909021cefb47283f8331a69), branch `I115`; the published commit changes three Java implementation paths and one H7 focal test. The benchmark harness reported `PRODUCT_VERSION=0.3.403-SNAPSHOT` and a clean product working tree at this commit. **This experimental branch's version did not advance from the R4 baseline**; do not infer a product-release version increment.
- R4 comparator: `guillermomolina/protos@bcf01844f495485145b423eaf2a55589fdb580e1` (reported capture provenance).
- Peer: GraalPy **25.4.4.1.1**, same host-side benchmark workflow.
- Workload: `cases/primitive-method-call/primitive-method-call.protos` and the corresponding GraalPy source; expected result **1**, one `receiver.identity(1)` invocation per `run`. This is **not** the 1,000-iteration H7 implementation-test fixture, whose expected result is **499500**.
- Structural measurement: selected compiled **primary guest root**, phase `After TruffleTier`, separate Tier 1 and Tier 2 cold captures; `WARMUP_CALLS=0`, `INVOCATIONS=2000`, `FIRST_THRESHOLD=100`, `LAST_THRESHOLD=1000000` (Tier 1) or `1000` (Tier 2).
- R5 capture directories: `results/local/i115-h7-r5/protos-tier{1,2}/primitive-method-call/protos/` (local, Git-ignored); R4/peer: `results/local/i115-h7-r4/{protos,python}-tier{1,2}/primitive-method-call/{protos,python}/`. The maintainer supplied the cold summaries, histograms and comparisons in the 2026-10-30 investigation. **Raw BGVs, analyzer exports, hashes, full capture manifests and host identity are not published in this record.** Exact benchmark producer revision must be read from the retained capture manifest before a reference campaign is published.

## Focal implementation and validation

R5 uses a compilation-final `experimentalH7RawIntegerReturn` root flag to keep an admitted H7 `Integer` unwrapped through `EpilogForReturn` and through the direct or indirect `ProtosHostExecutableClosure.Execute` interop boundary; the ordinary conversion remains for non-admitted roots and wide numeric results. The targeted tests assert the raw **Java class** returned from both a direct guest `CallTarget` and `InteropLibrary.execute`, not just the numeric `Value.asLong()`, so reintroducing the allocation at either boundary becomes a regression. The maintainer reports the following focal command **PASS**:

```text
mvn -q -Dtest=ProtosI115H7MethodCallConvergenceTest,ProtosPerf033CanonicalCallableInteropTest test
```

The previously observed H7 loop fixture still returns `499500`, with static DSL FULL `module=41`, `identity=5`, `run=27`. The I115 branch deliberately uses focal validation only; no full-suite, integrated release, semantic conformance or performance-speedup claim is made.

## Measured comparison

The maintainer reported valid cold captures and matching phase/histogram totals for R4, R5 and GraalPy. Values below refer to the same selected primary root, not a total over all compiled roots.

| Metric | Protos R4 | Protos R5 | GraalPy |
| --- | ---: | ---: | ---: |
| Tier 1 compilation ID | 2701 | 2669 | 4882 |
| Tier 1 After TruffleTier nodes | 158 | **153** | 181 |
| Tier 1 Final IR nodes | 295 | **257** | 297 |
| Tier 1 machine-code bytes | 1179 | **1030** | 1104 |
| Tier 2 compilation ID | 2854 | 2801 | 5039 |
| Tier 2 After TruffleTier nodes | 69 | **66** | 82 |
| Tier 2 Final IR nodes | 144 | **110** | 135 |
| Tier 2 machine-code bytes | 614 | **470** | 534 |

From R4 to R5, Tier 1 Final IR decreases by 38 nodes and machine code by 149 bytes; Tier 2 Final IR decreases by 34 nodes and machine code by 144 bytes. R5 machine code is also 74 bytes smaller than GraalPy in Tier 1 and 64 bytes smaller in Tier 2 **for this workload and these captures only**.

### Tier 2 physical attribution

| Node class | R4 | R5 | GraalPy |
| --- | ---: | ---: | ---: |
| `AllocatedObjectNode` | 1 | **0** | 0 |
| `CommitAllocationNode` | 1 | **0** | 0 |
| `VirtualInstanceNode` | 2 | **1** | 1 |
| `BoxNode$AllocatingBoxNode` | 0 | **1** | 1 |
| `FrameState` | 6 | 6 | 6 |
| `GuardedUnsafeLoadNode` | 1 | 1 | 1 |
| `IntegerEqualsNode` | 1 | 1 | 0 |
| `FixedGuardNode` | 5 | 5 | 7 |
| `LoadFieldNode` | 1 | 1 | 6 |
| `LoadIndexedNode` | 6 | 6 | 6 |

The former `ProtosIntegerValue` materialization had been traced to `MaterializeReturnCarrier.perform` → `ProtosNumericValueSupport.guestValue`. In R5 the corresponding `AllocatedObjectNode` and `CommitAllocationNode` disappear from the selected Tier 2 graph; the remaining `BoxNode$AllocatingBoxNode` matches GraalPy by node class. This establishes **local compiled-graph convergence**, not the absence of all machine-level allocation, whole-program equivalence, or runtime superiority.

## Next investigation and publication boundary

R5 is a completed experimental implementation slice, **not closure of I115**. Re-run **all 17 catalogued Protos workloads** from the exact clean R5 commit with the benchmark repository's retained, unmodified graph policy; keep Tier 1 and Tier 2 evidence separate and independently admitted, and compare only matching peer phases/root scopes. Rank the next issue by valid Protos-minus-GraalPy excess, with Final IR, machine bytes, invokes/allocations/guards and semantic divergence as separate signals. Failed or unavailable captures remain explicit `UNAVAILABLE`, not zero.

The previous I115-H6 report and recovered observations remain immutable historical benchmarks; do not overwrite them with R5 results. A new versioned benchmark-repository report and derived node table should be published **after** the full remeasurement, with exact product/harness revisions and retained evidence identities.
