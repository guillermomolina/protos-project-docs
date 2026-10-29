# I115-H6 — experimental host-entry convergence, verified factorial cold graphs

**Date:** 2026-10-29. **Classification:** revision-bound experimental implementation/graph evidence, not normative Protos semantics or measured execution speed.

**Product:** [`guillermomolina/protos@493cf1394fd88efa07c5740541a5873d1ba45e44`](https://github.com/guillermomolina/protos/commit/493cf1394fd88efa07c5740541a5873d1ba45e44), subject `I115-H6: collapse polyglot host entry to a single compact guest call`, on the experimental **I115** branch; no merge to `main` is asserted. The version remains `0.3.403-SNAPSHOT`; H6 does not change `pom.xml` or `CHANGELOG.md`.

**Formal owners:** [I115 #931](https://github.com/guillermomolina/protos/issues/931) and [PLAT066-C #930](https://github.com/guillermomolina/protos/issues/930). **Historical comparison:** [I115-H5 graph evidence](I115_H5_FACTORIAL_PHYSICAL_CONVERGENCE_AND_HOST_ENTRY_RESIDUAL_2026_10_29.md).

## Publication and correctness

- Maintainer reports `git diff --check` clean, local `make test` PASS, and `make check` PASS after publishing H6. These commands were not independently executed by the evidence reviewer.
- The production change to `ProtosHostExecutableClosure.Execute` specializes source-backed zero- or one-argument host calls as one compact no-home B5 guest call (direct when the target is cached, indirect otherwise). A null target or calls outside its admission guards retain the generic path.
- The former root non-suspension eligibility proof can still return false; it no longer controls this experimental host path. The two affected tests in `ProtosHostEntryFutureSuspensionTest` now assert observable host invocation rather than those proof flags. This record does not attribute the proof behavior to H5 or H6.
- Both Protos and GraalPy reference adapters report `CORRECTNESS=PASS`, actual factorial(20) = `2432902008176640000`. Four graph-capture manifests independently repeat correctness PASS and `capture_valid=true`.

## Provenance and verification

- **Producer:** [`guillermomolina/protos-benchmarks@b41c60cd368a1fac7509305ea255150d8560a03d`](https://github.com/guillermomolina/protos-benchmarks/commit/b41c60cd368a1fac7509305ea255150d8560a03d), clean at capture. The product was clean before and after all four captures, with the same exact `493cf1394fd88efa07c5740541a5873d1ba45e44` revision.
- **Runtime:** GraalVM/GraalPy `25.4.4.1.1`. Protos surface `canonical`; GraalPy surface `executable-value`; case `factorial`, no workload edits.
- **Method:** `cold-capture`, fresh JVM, requested Tier 1 and Tier 2 separately, `COMPILATION_ROOT=all`, `WARMUP_CALLS=0`, `INVOCATIONS=2000`, `FIRST_THRESHOLD=100`; compilation-graph phase `After TruffleTier`. The run manifest records one initial cold call plus 2,000 post-warmup invocations. This 0-warmup capture protocol is distinct from older 128-call I115 measurements.
- **Result:** T1 Protos 3 compiled roots/0 unavailable, GraalPy 5/1; T2 Protos 3/0, GraalPy 4/2. Graph analysis and summarization each reported zero failures. GraalPy unavailable roots are bootstrap `ModuleSpec.parent` (ambiguous compilation, both tiers) and `ObjectBuiltins.__setattr__` (Tier 2 not observed); neither is the factorial/run/host comparison root.
- **Source package:** maintainer-supplied `i115-h6-cold-graphs-evidence.tar.gz`, SHA-256 `f8f197764431c30c6897df67afa154c7cf0a38152095a46be164b07e49dd6456`. Binary package is retained in the evidence conversation, **not committed** to this documentation repository. Reviewer read the complete archive, checked all 16 retained BGVs against each capture's uncompressed size/SHA-256 inventory, and checked all four raw compilation traces against their SHA-256 manifest values.

| Requested tier | Language | Uncompressed compilation trace SHA-256 |
| --- | --- | --- |
| 1 | Protos | `3e61b073af1dbfc89105dc03716240f418c4c2bba5922da149b3ced7e83bdc2e` |
| 1 | GraalPy | `ff2c8ce51d3b94345b65ad5c6719d9c4b62eb8bf33bb3565a06154f4e086934f` |
| 2 | Protos | `e93ace28c4c65adb422b3e866008204b271b1eb65e7b0aad7328994fea8889c7` |
| 2 | GraalPy | `2c7921b7c35b53558133ee829d0746161bd0e6b5764f85b8d9b28a138b66bf08` |

## Physical host-entry result

All primary values are **selected BGV `After TruffleTier` node counts**. H5 values are separately captured **historical references**, not a same-run A/B; compare this as evidence of a large directionally consistent reduction, not a statistical estimate.

| Host root (`Value.execute`) | H5 Protos | H6 Protos | GraalPy 25.4.4.1.1 |
| --- | ---: | ---: | ---: |
| Tier 1 nodes | 2,319 | **226** | 370 |
| Tier 2 nodes | 2,678 | **380** | 675 |

Relative to historical H5, H6 reduces Protos host graph nodes by **90.3% (T1)** and **85.8% (T2)**. Its host graph is also **38.9% (T1)** and **43.7% (T2)** smaller than this matched GraalPy capture.

| Host metric | H6 Protos T1 | GraalPy T1 | H6 Protos T2 | GraalPy T2 |
| --- | ---: | ---: | ---: | ---: |
| Final IR nodes | 351 | 585 | 655 | 1,086 |
| Machine code bytes | 1,623 | 2,906 | 2,055 | 3,772 |
| `IfNode` | 7 | 9 | 3 | 6 |
| `InstanceOfNode` | 0 | 2 | 0 | 11 |
| `FixedGuardNode` | 7 | 16 | 5 | 50 |
| `LoadFieldNode` | 11 | 15 | 6 | 25 |
| `FrameState` | 44 | 85 | 98 | 177 |

**Direct guest-call topology, verified from original `trace.log.gz` files:**

- **Tier 1:** Protos host compilation 2586 has exactly one depth-1 `Cutoff` to `ProtosBytecodeRootNodeGen@6a7eba6a` (`run`); GraalPy host compilation 4988 has exactly one depth-1 `Cutoff` to `<bytecode run at 4c9212cb>`.
- **Tier 2:** Protos host compilation 2813 has exactly one depth-1 `Inlined` to `ProtosBytecodeRootNodeGen@216a5dc` (`run`); GraalPy host compilation 5095 has exactly one depth-1 `Inlined` to `<bytecode run at 49c59f10>`.
- The Tier 2 trace archives additionally contain earlier Tier 1 cutoff compilations (Protos 2688, GraalPy 4998). These are **not** the selected Tier 2 host compilations and do not contradict the Tier 2 inlining result.

**Guest roots remain compact:** H6 Protos `factorial()` is 161 nodes (T1) / 298 (T2), and `run()` is 129 / 250; GraalPy guest `factorial()` is 216 / 568 and `run()` is 161 / 407. These H6 guest counts match the recorded H5 checkpoint. These are independent compiled graphs: do not sum their node counts as runtime cost or count inlined code twice.

## Semantic debt and interpretation limits

- The H6 admitted entry deliberately skips normal argument admission/conversion, Context/Process/RootActor checks, host-entry suspension extent, ReturnHome completion, failure mapping, and guest-result projection; its result is the raw guest call result. These are **experimental divergences**, not specification changes or approval to ship the shortcut as normal Protos behavior.
- Existing `provablyNonSuspendingBody()` / `fastNonSuspendingHostEntry` proof logic remains untouched. Replacing its tests with invocation assertions makes the host behavior testable but does not repair or explain a false proof result; track proof correctness separately if it matters outside the experiment.
- The matched GraalPy comparison establishes **graph structure and inlining shape for factorial**, not cross-language semantic equivalence, whole-program superiority, startup cost, or throughput/latency gains. Graph capture instrumentation is not a timing measurement.
- A reference, uninstrumented, same-host factorial timing comparison using the existing `protos-benchmarks` `make measure` workflow remains **pending**. Publish numeric speed claims only after valid samples and source/host identity checks.

**Lifecycle:** I115 #931 and PLAT066-C #930 remain **OPEN**. I115-H6's host-entry physical convergence is structurally accepted; subsequent work may measure latency and investigate guest-root residuals or experimental semantic restoration without claiming the broader workstream finished.
