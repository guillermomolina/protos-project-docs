# I103-E/F/G/H — Final directory-limit acceptance (2026-10-15)

**Formal work:** [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
**Record type:** exact-product-revision publication evidence and owner-reported local acceptance; not a new normative design decision.
**Published Protos commit:** [`0c617058487be96ea05b26c04df417eda6985cbe`](https://github.com/guillermomolina/protos/commit/0c617058487be96ea05b26c04df417eda6985cbe).
**Published implementation version:** `0.3.366-SNAPSHOT` (verified in published `pom.xml` and `CHANGELOG.md`).
**Immediate product parent:** [`2263d4f66294817524052b9782ffee1562848a50`](https://github.com/guillermomolina/protos/commit/2263d4f66294817524052b9782ffee1562848a50) (I104 Closure binding, `0.3.365-SNAPSHOT`).
**Starting I103 execution-layout parent:** [`d12d890db64a0a4f7b9c512521774c2c4ecaa283`](https://github.com/guillermomolina/protos/commit/d12d890db64a0a4f7b9c512521774c2c4ecaa283) (I103-D, `0.3.364-SNAPSHOT`).
**Prior published evidence:** [I103-A](I103_A_PUBLISHED_DIRECTORY_GUARD_AND_TOOLS_DIST_LAYOUT_2026_10_15.md), [I103-B](I103_B_PUBLISHED_TOOLING_FIXTURES_AND_30_FILE_LIMIT_2026_10_15.md), [I103-C](I103_C_PUBLISHED_CORE_NETWORK_LAYOUT_2026_10_15.md), [I103-D](I103_D_PUBLISHED_RUNTIME_JAVA_LAYOUT_2026_10_15.md).

## Delivered acceptance

The owner-approved ceiling is **30 directly tracked regular files per maintained first-party directory**, counting a file only in its immediate directory. `scripts/directory_file_limit.py` inventories stage-0 Git index entries, counts regular modes `100644`/`100755`, excludes symlinks/submodules/untracked output, rejects malformed/unmerged entries and exempts only the pinned upstream TOML snapshot under `protos/tests/conformance/library/toml/official/upstream/v2.2.0/`. The first-party generated TOML suite remains counted. The 29/30/31 boundary and nested parent/child case are covered by synthetic guard tests introduced and extended during I103-A/B.

GitHub inspection of the exact published product confirms:

- `Makefile` has `check-directory-file-limit`, backed by `scripts/directory_file_limit.py check`, in its standard `make check` prerequisite list.
- `.github/workflows/tests.yml` runs `make check-directory-file-limit` as a distinct required workflow step before `make test`; actual CI invocation is not merely a dormant Makefile target.
- The root `CHANGELOG.md` records I103-E/F/G/H under `0.3.366-SNAPSHOT`, following the published `0.3.365-SNAPSHOT` I104 checkpoint.
- The maintainer reported `make check-directory-file-limit → PASS`, including **zero offending applicable directories**, after final staging. The complete combined candidate subsequently passed `make check → PASS` and `make test → PASS` locally. These executions were **human-performed**; they are not presented as GitHub CI conclusions or independently agent-run tests.

## Execution Java organization and source-identity census

All I103-E/F/G moves concern *physical file organization*, not a blanket Java-package rewrite. The production and test classes whose package-private contracts require `package com.guillermomolina.protos.execution;` retain that declaration and original class identities. Responsibility-grouped physical paths do not necessarily imply new declared Java packages.

| Java execution scope | I103-D baseline, root direct Java | Final root direct Java | Final recursive Java identities | Provenance |
| --- | ---: | ---: | ---: | --- |
| `src/main/java/com/guillermomolina/protos/execution` | 240 | **0** | **240** | Maintainer's validated relocation/inventory and final index staging |
| `src/test/java/com/guillermomolina/protos/execution` | 447 | **0** | **448** | The original 447 test Java identities plus the newly published I104 `ProtosPerf042IdentityBuilderOracleTest.java` |

Thus I103 relocated **687 existing execution Java files** (240 production and 447 tests); the increase to 448 tests is an **I104 addition**, not a duplicated I103 source. The maintainer's source-identity checks and staged-index audit reported PASS after merging I104. The complete exact per-file relocation set is encoded by the published product tree and its diff; this table is an identity/census summary, not a claim that every file is byte-identical to pre-I104 code.

The responsibility-oriented map includes the following **published physical location families** (examples; paths are relative to each `execution/` root):

| Owner / responsibility | Production examples | Test examples |
| --- | --- | --- |
| Bytecode activation | `bytecode/activation/ProtosSemanticBytecodeRootNode.java`, `ProtosBytecodeRootNode.java` | Relevant compiler/activation and lowering test families |
| Canonical lowering | `bytecode/lowering/CanonicalToBytecodeLowerer.java`, `ProtosFrameLexicalBindingAuthority.java` | `compiler/optimizations/ProtosI101ProfileSpecializedLoweringTest.java` |
| Numeric specializations | `bytecode/numeric/ProtosNumericAddNode.java` | Compiler and numeric conformance test families |
| Foreign APIs/providers | `foreign/api/ProtosForeignModuleImports.java`, `foreign/host/ProtosHostJavaProvider.java`, `foreign/providers/ProtosExternalProviderPluginLoader.java` | Foreign provider/import and host Java tests |
| Host/embedding | `host/embedding/ProtosHostExecutableClosure.java` | Host/embedding tests |
| Module/package/workspace | `modules/ProtosModuleRuntime.java`, `packages/ProtosPackageRunDriver.java`, `workspace/ProtosWorkspaceRunDriver.java` | Corresponding module/package/workspace tests |
| Protocols, IO, tools | `protocols/io/ProtosStandardFileProtocol.java`, `host/io/ProtosNioNetworkBackend.java`, `tools/test/resources/ProtosTestResourceProviderRegistry.java` | Corresponding protocol/IO/Test Tool tests |
| I104 oracle integrated into I103 layout | No extra production class | `compiler/optimizations/ProtosPerf042IdentityBuilderOracleTest.java` |

The filesystem-backed tests and guides reading old hard-coded `execution/<Class>.java` paths were reconciled against the new source layout. Class package names and Truffle bindings remained subject to the real compiler/JUnit/PE guards; the final published tree includes the reconciled path inventories in PE JSON baselines and validation-impact routing.

## Concurrent I104 reconciliation (published, not silently overwritten)

Between the I103-D base and final I103 publish, I104-A/B/C was independently published to `main` at `2263d4f6`. The maintainer fast-forwarded onto that commit and recovered the already validated I103 migration from a local stash. Direct `stash apply --index` failed where I104 modified files that I103 had moved; subsequent safe recovery used content-level three-way merges, preserving I104 code.

- **Six Java overlaps** were merged against their actual common base: `CanonicalToBytecodeLowerer`, `ProtosSemanticBytecodeRootNode`, `ProtosI101ProfileSpecializedLoweringTest`, `ProtosPerf025CompactCalleeExecutionTest`, `ProtosPerf025FrameMaterializationSliceTest` and `ProtosPerf025H1IndexedParameterEstablishmentTest`.
- **Two PE baseline overlaps** (`java_local_range_pe_guard_baseline.json`, `java_local_range_pe_reachability_baseline.json`) were reconciled preserving I104 semantics while updating physical paths.
- **One newly added test**, `ProtosPerf042IdentityBuilderOracleTest.java`, was placed in `execution/compiler/optimizations/`; the final index census became **240 production + 448 test** Java files.
- The `java_local_range_operand_pe_baseline.json` candidate added **exactly four** safe/topologically accounted sinks from I104: two `PE_REACHABLE` sites with receiver/BytecodeNode proven and two `NOT_PE_REACHABLE` sites, with zero new PE risks/unknowns and all previous entries preserved.
- The generated Bytecode BCI topology baseline was reconciled against the combined generated code. The maintainer's guard output reported **1112** transitions (**1108** proven, **4** dynamic), **0 risk**, **0 unknown**. The changed topology represented new binding instructions and deterministic generated-switch case layout, not permission to baseline unknown or risky transitions.

None of these steps changed the approved I103 directory-limit policy or retroactively rewrote the already published I104 commit.

## Final human validation and publication

| Gate / checkpoint | Result and provenance |
| --- | --- |
| I103 pre-I104 structural correction | First full `make test` hit 2 assertion failures and 4 missing-file errors in foreign/interop source-inspecting tests; corrected source paths and recursive source discovery without weakening forbidden-access assertions |
| I103 focused regressions | Maintainer reported focused four-class interop/foreign JUnit rerun **PASS** |
| I103 pre-I104 integration | `make test → PASS` on the I103 candidate before the I104 HEAD merge |
| I103/I104 source-identity merge | `SIX_JAVA_OVERLAPS_RECONCILED=PASS`, `TWO_PE_BASELINES_RECONCILED=PASS`, `I103_I104_INTEGRATED_STAGING=PASS`, no unstaged changes; owner-reported |
| Operand PE baseline / generated BCI topology | Exact candidate reconciliation reviewed, no PE-reachable risks or unknowns reported |
| **Final combined static checks** | **`make check → PASS`**, maintainer-reported after combined PE baseline fixes |
| **Final combined integrated suite** | **`make test → PASS`**, maintainer-reported on I103+I104 before late Maven/changelog finalization |
| Diff cleanliness | `git diff --check` clean, maintainer-confirmed; prior `git diff --cached --check` clean |
| Final product publication | User executed `git push origin main`: **`2263d4f6..0c617058`**; GitHub independently confirms `main` at full exact SHA and version `0.3.366-SNAPSHOT` |

The first full `make test` failure and guard topology-baseline mismatches were **intermediate diagnostics**, not final accepted outcomes. No tests were requested after the late `pom.xml`/`CHANGELOG.md` update. The two local stashes reported by the maintainer remain independent local recovery artifacts and are **not** part of this product publication.

## Scope and closure boundaries

- I103 acceptance is the **maintained first-party repository layout and strict 30-file regression gate**, not a new guest-language architecture or benchmark performance claim.
- The initially published I103-A 25-file numbers and the original 446/231 user inventory are preserved as historical records; the owner ratified **30** as the ultimate limit. The 28-file IO and TOML first-party suites were intentionally not split.
- This document is durable **non-normative evidence**. Live closure, labels, scheduling and priority belong to [GitHub Issue #897](https://github.com/guillermomolina/protos/issues/897).
- The published GitHub commit and retained artifact history form the revision-pinned source of truth. Local test outcomes are explicitly maintainer-reported, not independently reproducible from this documentation transaction.

**Acceptance disposition:** all I103-A through I103-H implementation slices and the final strict-gate acceptance were published; Issue #897 is eligible for completed closure after this record's publication and live-Issue reconciliation.
