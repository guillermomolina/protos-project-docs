# PERF045 — Single-producer construction and internal iteration publication

Date: 2026-11-24. Non-normative, revision-bound implementation and graph evidence.

## Publication identities

- Formal owner: [PERF045 #889](https://github.com/guillermomolina/protos/issues/889).
- Published product: [c29d690a0bfbae3e5b264f8a32ea9079f76537e6](https://github.com/guillermomolina/protos/commit/c29d690a0bfbae3e5b264f8a32ea9079f76537e6), version `0.4.213-SNAPSHOT`.
- Publication range: [I173 baseline 8c3be223 through PERF045 final main](https://github.com/guillermomolina/protos/compare/8c3be22391db29bd4ff1abd102884d07d3b2a50e...c29d690a0bfbae3e5b264f8a32ea9079f76537e6); GitHub compare reports 19 commits ahead, zero behind. Versions 0.4.195 through 0.4.213; no spec/ paths changed in this interval.
- Actual measured product: [17f03e874e8a1dcc5b27b3f6cbb65903ec95d7f0](https://github.com/guillermomolina/protos/commit/17f03e874e8a1dcc5b27b3f6cbb65903ec95d7f0), `0.4.193-SNAPSHOT`. Its history diverges from the published main because the commits were transferred onto I173. Do not relabel the measurement as a fresh capture of the final published SHA.
- Main implementation producer commit: [128ac4c3](https://github.com/guillermomolina/protos/commit/128ac4c3fd1bb48378d2fc19e0a9eae5642aa7de), 0.4.212-SNAPSHOT. Final commit refreshes generated BCI PE baseline and I115 numeric structure tests; development container bash completion also added.
- Verified identical Git blob SHAs at measured and published revisions: `ProtosBytecodeRootNode.java` 042794007b5afa92b09490c149cbf04e84de9b27; `CanonicalToBytecodeLowerer.java` f860c3038a110bab486a1fea4599555bc9feb68b; `ProtosNumericSelectionNode.java` 09bc8505e56fd8aa3696f96813e3dd9d7304f56f. This is targeted source identity, not full-tree equality.

## Implementation

The published series converges numeric/indexed operations and captured-cell writes on producer-owned results; eliminates redundant operand-local traffic; preserves assignment Error provenance; specializes frozen selections at specialization time; builds canonical Range slots through existing cached dynamic-object operations; advances primitive unit-step ranges without unnecessary overflow checks; and completes Array and Range construction in one producer. Standard construction does not execute a separate constructor dispatch. Only genuinely selected custom factories invoke through existing dispatch and suspension transport. The lowerer clears compiler-owned Range receiver/step operands before the common internal loop, retaining them until completion on the selected alternative path. Ordinary shadowing, overrides, exact Integers, callback identity, nonlocal return and suspension semantics are preserved by the implementation and regression tests; no normative change in the PERF045 series.

The implementation follows existing I115 infrastructure and GraalPy Bytecode DSL precedents (reference revision `12a66f9ef7ed7fd26f013d72ab8bec0a1da6ad6b`): MakeConstantLongList/MakeList direct collection construction, ReadGlobalOrBuiltinNode specialization-time cached facts, GetIter/ForIterate and RootNodeCompiler For lowering. No performance claim derives solely from the reference.

## Captured evidence

Maintainer-executed harness: `protos-benchmarks`, case `array-iteration-for`, `cold-capture`, LANGUAGE=protos/python, TIER=1/2, COMPILATION_ROOT=all, INVOCATIONS=2000, fresh JVM with one cold call followed by 2000 calls. Run directory `results/local/perf045-single-producer-20261124-165833`. GraalPy peer cache identity `peer-25.4.4.1.1`. Four valid captures, zero analysis or summary failures. Benchmark repository SHA not supplied. Main workload returns 2080; compare selected physical run roots at dump graph 1, After TruffleTier, not startup builtins.

| Graph metric | Previous Protos T1 | Current Protos T1 | GraalPy T1 | Previous Protos T2 | Current Protos T2 | GraalPy T2 |
|---|---:|---:|---:|---:|---:|---:|
| Nodes | 456 | 371 | 378 | 346 | 261 | 271 |
| FrameState | 41 | 28 | 28 | 21 | 8 | 8 |
| VirtualObjectState | 41 | 25 | 26 | 30 | 15 | 17 |
| IfNode | 10 | 7 | 7 | 4 | 1 | 1 |
| FixedGuardNode | 12 | 14 | 14 | 12 | 14 | 14 |
| LoadFieldNode | 19 | 13 | 15 | 12 | 6 | 8 |

Previous capture revision `8eb40f43e07256633e075b973c3a7e0bf4db4b88`. The current run removes 85 nodes in each tier (18.6% T1, 24.6% T2), meets structural budgets 378/271, and has one physical loop. Four BGV files were supplied for direct graph review: perf045-final-protos-t1/t2.bgv.gz and perf045-final-python-t1/t2.bgv.gz. T1 compilation IDs 3202/4941 and T2 IDs 3150/4937 (Protos/Python). Supplied T1 dump SHA-256: Protos `9e12709cbf8b3ffd768e4561fa821afd606e008fdb4d49e10f590acf82233608`, Python `113d1b30ad09aade019e727fd68cf741b44b916ef7eec5ed243b8c877c8b3879`. Raw artifacts remain local; not represented as uploaded to this documentation repository.

Graph review identified elimination of the three redundant Prelude-resolution bifurcations (MakeConstantLongArray, MakeRange, PrepareRangeIteration) and 13 FrameState nodes in each tier. The original virtual Range ProtosObjectValue and its map-backed lexical authority no longer survive in the final loop FrameStates; the existing RangeIterationState cursor does. T2 retains one loop condition and no invokes. Fixed guards for legitimate lexical/selection/bounds/arithmetic conditions remain.

| Comparable root | Protos T1 | GraalPy T1 | Protos T2 | GraalPy T2 |
|---|---:|---:|---:|---:|
| run | 371 | 378 | 261 | 271 |
| Value.execute | 292 | 370 | 437 | 528 |
| Comparable subtotal | 663 | 748 | 698 | 799 |

Python also captured startup auxiliary roots; one T1 and two T2 roots were unavailable. No exhaustive interpreted-root coverage claimed. Later compiler phases do not uniformly favor Protos; low-tier node counts 681/680 T1 and 616/603 T2 (Protos/Python). Node counts are structural evidence, not latency, throughput, code-size or Native Image acceptance.

## Validation provenance

- Earlier focused compilation and 190/190 Java tests PASS (27 ArrayEachNative, 2 EachSelectionOrdering, 5 InlineCallbackPreparation, 79 Perf045RangeEachInline, 36 I145AConstructProtocol, 29 I115MCompactLowering, 12 StandardArrayFactory). After annotation/test refinements, compilation and 79/79 Perf045 rerun PASS.
- Four PE validators PASS with 111/111 validator unit tests (21 LocalAccessor, 32 Bytecode API, 40 LocalRange index, 18 LocalRange operands). LocalAccessor reconciliation retained 74 unchanged entries, added 16 proven and retired 11 obsolete; 90 sinks, 62 proven, four boundary cuts, 24 NOT_PE_REACHABLE, zero new risk/unknown/stale/drift. Bytecode API 36 sinks, seven boundary cuts and 29 NOT_PE_REACHABLE; no new risks or topology drift. These classifications do not prove absence from actual compiled execution. Existing CopySuppliedArguments.perform loop-index risk remains registered and not proven safe, while its receiver and node provenance are proven.
- Maintainer confirms all local tests including I173, clean git diff --check, clean working tree, preserved APL notices and publication on origin/main. Full final-suite counts and independent CI/Native Image logs not supplied; no extra validation after metadata-only version bump claimed. Earlier graph/PE runs belong to the prepublication implementation, not newly executed at final main.

## Disposition and residual scope

The bounded Array/Range numeric-for structural objective is complete and published. The original PERF045 #889 acceptance additionally covers whole-family composed Set/IdentitySet forwarding, projected foreign iteration classification, other receiver workloads and comparable timings or evidence-backed no-action decisions. Those are not established by this run. Record this as a completed implementation/structural checkpoint, not universal each-family parity or measured runtime speedup. I115 #931 remains previously closed; I173 #1164 is separately implemented and locally validated, with independent reflective-slot performance/audit gates not discharged by array iteration.
