# I115-F / 0.3.387 — verified product publication and four-graph reconciliation

**Date:** 2026-10-20. **Product:** [`guillermomolina/protos@65541f46cb6dc9488c2f904e39131f5668fbb999`](https://github.com/guillermomolina/protos/commit/65541f46cb6dc9488c2f904e39131f5668fbb999), `0.3.387-SNAPSHOT`. **Work owner:** [I115/#931](https://github.com/guillermomolina/protos/issues/931). **Record purpose:** revision-bound follow-up to the earlier [I115-F local BGV acceptance](I115_F_387_ROOT_LOCAL_FOUR_GRAPH_ACCEPTANCE_2026_10_20.md), whose product-release identity was then unavailable.

## Exact published revision

GitHub `guillermomolina/protos/main` was independently read after the maintainer reported publication; HEAD `60af4eac59a08d31184d517d064878243cdb0914` (`Fix tmp permissions`, devcontainer configuration only) has the **direct parent** `65541f46cb6dc9488c2f904e39131f5668fbb999`, the exact I115-F implementation commit. I115-F has parent `607973c493dcb0c5ba3a7eeba9897b045eb049d8`, the accepted I115-E / 0.3.386 reference.

- **Commit message:** `I115-F: read present published root frame locals directly (0.3.387)`.
- **Published product paths (5):**
  1. `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`
  2. `src/test/java/com/guillermomolina/protos/execution/compiler/lexical/ProtosPerf025D179LexicalMembershipStabilityReadTest.java`
  3. `tools/java_local_accessor_pe_baseline.json`
  4. `pom.xml`
  5. `CHANGELOG.md`
- **Published POM:** `0.3.387-SNAPSHOT`; **published changelog:** I115-F present published root frame locals read directly; maintainer `make check` PASS and integrated `make test` PASS. This agent read these published statements but did not execute any validation.
- **Published mechanism:** `ReadRootFrameLocal.publishedPresent` directly reads the `LocalAccessor` for eligible published genuine-Context activations where the binding is PRESENT. The `canReadPublishedFrameLocal` guard retains one-way D179 membership assumption and dynamic `isCleared` check after invalidation. The unchanged compact path remains; non-genuine or ABSENT cases retain the materialized/slow lexical fallback. The tested observable D179 contract and D013 invocation semantics are not changed. No added Bytecode DSL operation.
- **Published regression:** root-local read from suspended real roots across PRESENT, structural remove, recreate, null-present and independent activations. The standard `SlotNotFound` prototype is used for exception identity on each activation.

## Physical measurements: exact comparison scope

The maintainer-supplied local four-capture archive was previously indexed with BGV SHA-256, CompId, `[engine] opt done` stage counts, code sizes, and JVM settings in the [I115-F local-capture record](I115_F_387_ROOT_LOCAL_FOUR_GRAPH_ACCEPTANCE_2026_10_20.md). Its capture logs were not stamped with the full local checkout SHA at execution time, so those BGVs are a **human-provided measurement attributed to I115-F**, not proof of a repeated compilation of the newly verified GitHub commit. This addendum verifies the matching source change and publication identity, not re-execution.

| Metric | Published prior 0.3.386 reference | I115-F captured working tree | Difference |
| --- | ---: | ---: | ---: |
| FULL Bytecode DSL module | 11 | 11 | 0 |
| FULL Bytecode DSL identity | 5 | 5 | 0 |
| **FULL combined descriptive sum** | **16** | **16** | **0** |
| Tier 1 module `After TruffleTier` IR | 379 | 375 | -4 |
| Tier 1 separate identity root | 106 | 106 | 0 |
| **Tier 1 combined descriptive sum** | **485** | **481** | **-4 (-0.82%)** |
| Tier 2 module IR | 264 | 260 | -4 |
| Tier 2 separate identity root | 49 | 49 | 0 |
| **Tier 2 combined descriptive sum** | **313** | **309** | **-4 (-1.28%)** |

All counts represent **separate** real `TOP_LEVEL` and `identity` compilations for `identity: (value) => { value }\nidentity(1)`. Their summed nodes do not constitute one compiler graph or a performance/speed claim. GraalPy Tier-2 JVM 25.4.4.1.1 descriptive reference 286+51=337 and historical GraalPy Tier-1 Native Image 25.3.4.1 469+110=579 are further detailed in the [PERF042 0.3.386 measurement record](../PERF042/PERF042_386_BARE_TOP_LEVEL_IDENTITY_THREE_METRIC_ACCEPTANCE_AND_RUN_IDENTITY_HANDOFF_2026_10_20.md). The latter is **not** a JVM-matched Tier-1 comparator.

This verified source release does not upgrade the BGV source-provenance limitation or independently prove `readRootFrameLocalSlow` is absent from every execution/fallback; the physical evidence is restricted to the selected captured hot graph and compiler IR counts.

## Live coordination and follow-up

[I115/#931](https://github.com/guillermomolina/protos/issues/931) is **OPEN / `status:ready`**, by express maintainer instruction, for a further optimization slice not yet selected. The prior C/D, E and F publication checkpoints remain historically accepted, but none closes the continuing I115 workstream. No new `I` issue is needed solely for this bounded slice. [PLAT066-C/#930](https://github.com/guillermomolina/protos/issues/930) and [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) retain their separate, still-open physical research/benchmark objectives; PERF042's next accepted scenario is `run → identity`.

**PROTOS_REVISION(I115-F)=65541f46cb6dc9488c2f904e39131f5668fbb999**

**Historical capture evidence:** [I115_F_387_ROOT_LOCAL_FOUR_GRAPH_ACCEPTANCE_2026_10_20.md](I115_F_387_ROOT_LOCAL_FOUR_GRAPH_ACCEPTANCE_2026_10_20.md). No changes to normative specs, product source, tags, or releases were performed by the documentation work.
