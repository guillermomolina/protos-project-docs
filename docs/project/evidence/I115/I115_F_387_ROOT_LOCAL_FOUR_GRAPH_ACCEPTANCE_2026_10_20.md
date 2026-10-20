# I115-F / 0.3.387 — published root-local read: four-graph measurement and verification boundary

**Date:** 2026-10-20. **Owner:** [I115/#931](https://github.com/guillermomolina/protos/issues/931).
**Kind:** maintainer-supplied implementation, local validation and four uploaded BGV/log snapshots; this is **not an independently verified product release record** until the new product commit is readable by SHA.

## Product identity and scope

- **Owner report:** `I115-F: read present published root frame locals directly (0.3.387)` pushed to `guillermomolina/protos`. No exact commit SHA was supplied.
- **Last product revision independently verified via GitHub main/ref and commit:** [`607973c493dcb0c5ba3a7eeba9897b045eb049d8`](https://github.com/guillermomolina/protos/commit/607973c493dcb0c5ba3a7eeba9897b045eb049d8), I115-E / `0.3.386-SNAPSHOT`. The remote `main` ref still returned this revision when this record was prepared. Consequently, the owner-reported `0.3.387` publication, source diff, and its `PROTOS_REVISION` remain **unverified in the live remote**. Do **not** use the older SHA as the revision implementing I115-F.
- **Reported implementation:** `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java` adds `ReadRootFrameLocal.publishedPresent` and `canReadPublishedFrameLocal`, with a direct `LocalAccessor.getObject` read when a published activation has a genuine execution context and the D179 slot is present; unchanged compact path and retained materialized/slow fallback for ineligible reads, including removed/absent slots. Only this operation is specialized; no new DSL instruction.
- **Reported regression edit:** `src/test/java/com/guillermomolina/protos/execution/compiler/lexical/ProtosPerf025D179LexicalMembershipStabilityReadTest.java`; exercises the root-local path and D179 remove/recreate, null-present and distinct-activation cases. Fixture registers `SlotNotFound` and uses `ProtosCoreErrors.prototype(activation, StandardError.SLOT_NOT_FOUND)` for same-Prelude error identity.
- **Semantics:** preserve D179 PRESENT versus ABSENT, lexical fallback on removal, existing D013 dispatch and single Bytecode DSL lowering. No normative specification change is claimed.

## Human validation and local graph capture

The maintainer reported `git diff --check` clean and successful `mvn -q -Dtest=ProtosPerf025D179LexicalMembershipStabilityReadTest test`. The maintainer additionally supplied a successful targeted Maven run with `ProtosI112TailContinuationTest`, `ProtosPerf025CompactCalleeExecutionTest`, `ProtosPerf025FrameMaterializationSliceTest`, `ProtosPerf042IdentityBuilderOracleTest`, and `ProtosI112TerminalCallBudgetTest`. The true FULL census in that output is 11 instructions for TOP_LEVEL plus 5 for `identity`, total 16; 2 expected source roots, 0 extra. **No I115-F integrated `make test` or `make check` result is asserted**, and none was executed by this documentation agent.

The maintainer supplied `perf042-comparison.tar.gz`, containing four distinct GraalVM JVM compilation captures for the two-statement program:

```protos
identity: (value) => { value }
identity(1)
```

`captureCompleteRunRootAtRequestedTier` compiled the actual TOP_LEVEL; `captureIdentityRootAtRequestedTier` compiled the isolated single-parameter Closure as a separate call target. Both were warmed to CACHED bytecode and each requested Tier was confirmed. The separate root counts below are added **only descriptively**; there is no single graph of their sum.

| Metric | Protos 0.3.386 reference | I115-F local capture | Delta | GraalPy reference |
| --- | ---: | ---: | ---: | ---: |
| FULL module | 11 | 11 | 0 | 12 |
| FULL identity | 5 | 5 | 0 | 5 |
| **FULL sum** | **16** | **16** | **0** | **17** |
| Tier 1 module `After TruffleTier` IR | 379 | 375 | **-4** | 469 |
| Tier 1 identity IR | 106 | 106 | 0 | 110 |
| **Tier 1 descriptive sum** | **485** | **481** | **-4 (-0.82%)** | **579** |
| Tier 2 module IR | 264 | 260 | **-4** | 286 |
| Tier 2 identity IR | 49 | 49 | 0 | 51 |
| **Tier 2 descriptive sum** | **313** | **309** | **-4 (-1.28%)** | **337** |

**Reference provenance:** the Protos `0.3.386` IR counts and GraalPy Tier-2 JVM `286+51` are recorded in [PERF042 0.3.386 accepted three-metric measurement](../PERF042/PERF042_386_BARE_TOP_LEVEL_IDENTITY_THREE_METRIC_ACCEPTANCE_AND_RUN_IDENTITY_HANDOFF_2026_10_20.md) pinned to product `607973c493dcb0c5ba3a7eeba9897b045eb049d8`. Its GraalPy Tier-1 numbers `469+110` are **historical Native Image 25.3.4.1**, not a matched JVM 25.4.4.1.1 run. GraalPy Tier 2 was reached via natural warmup, Protos via explicit `compile(true)`; the two-root sums are **not runtime throughput** or platform-equivalence measurements.

### Four new compilation logs

`IR A/B` means the Graal compiler's Truffle-tier-stage IR count **A** and later/final IR count **B** in `[engine] opt done`. Do not rename A as final IR or conflate it with the expansion-tree `Count`.

| Capture | CompId | Confirmed tier | IR Truffle/final | CodeSize | SHA-256 of BGV |
| --- | ---: | ---: | ---: | ---: | --- |
| TOP_LEVEL Tier 1 | 3470 | 1 | **375/938** | 5228 B | `75ad32a7c4f7b0dca9662230f296aa1bf896d9b051a38003891afeb50cb2d8f2` |
| identity Tier 1 | 3456 | 1 | **106/125** | 487 B | `77de46f523f4fa1696d13c9b08505a4a7f860e0f3c03b902d6c147c6b2846608` |
| TOP_LEVEL Tier 2 | 3479 | 2 | **260/644** | 4187 B | `30fcfd700bf0683896bcb1cf15ddb24eba3554589a7839918473128d194d4ad4` |
| identity Tier 2 | 3460 | 2 | **49/61** | 278 B | `bf5864848e2451ecd500a80f75e5aea317569603802fc734bc61907dc89ee707` |

- Archive SHA-256: `e144acbf1a8714a7e0673cc178e13368ff0d10c31011a6260304e78e17b89927`.
- Per-root marker status: `COMPILED_TIER=1/2`, `CAPTURE=READY`, one root-attributed BGV each. Captured on 2026-10-20 from separate JVM invocations.
- Captures use GraalVM/Truffle 25.4.4.1.1 JVM, `NodeSourcePositions=true`, `TraceNodeExpansion=truffleTier`, and `jdk.graal.Dump=Truffle:1`. The actual product SHA used by the local test runner was **not recorded**, so pin the uploaded archive rather than claiming equivalence to an as-yet-unseen published commit.
- The presence of a new fast specialization and the absence of slow-fallback textual ancestry in the captured hot graph were noted during exploratory BGV inspection; the defensible **whole-root IR change is exactly -4 Tier 1 and -4 Tier 2**, not an asserted 22- or 71-node reduction from unrelated older captures. The `identity` root is unchanged.

## Disposition and next step

The owner explicitly directs: **keep I115 OPEN** for a new, not-yet-selected optimization slice. This supersedes historical I115-C/D closeout text *for live coordination only*, without rewriting those completed slices. Current I115 state is Open / `status:ready`, and no next implementation candidate, native hierarchy relationship, D013 change, or further saving is automatically approved. PERF042's **separate `run → identity` scenario** remains an independent next physical-comparison objective.

**Publication-reconciliation task:** once a reachable product commit of I115-F is independently available, fetch and verify its exact SHA, `0.3.387-SNAPSHOT` POM and source/test diff, then append or publish a revision-bound confirmation referencing this immutable capture record. Until that check, **`PROTOS_REVISION(I115-F)=UNVERIFIED`** and no durable record should claim this local trace is tied to the remote 0.3.387 revision.
