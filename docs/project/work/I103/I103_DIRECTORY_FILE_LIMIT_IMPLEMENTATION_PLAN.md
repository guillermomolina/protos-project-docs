# I103 — Directory direct-file limit implementation plan

Status: **IMPLEMENTATION STRATEGY ACCEPTED; I103-A/B/C/D PUBLISHED; I103 OPEN** (2026-10-15).
Live owner: [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
Baseline product revision: [`c7004f71cb05621dd5a7f89e46459542bd7d9c87`](https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87).
Scope: non-semantic physical organization of tracked first-party files; this is not a new Dxxx/PLATxxx architecture selection.

## Accepted contract

* At final publication, every maintained first-party repository directory has at most **30 directly contained tracked regular files** (30 passes, 31 fails). Discover directories recursively; do not accumulate descendant counts into parent directories.
* Discover tracked files using Git's index / exact candidate inventory. Include regular `100644` and `100755` files; exclude links/submodules, `.git`, ignored/untracked build output, and the byte-exact third-party `toml-test` snapshot alone. Generated *first-party* sources remain in scope: do not exclude `protos/tests/conformance/library/toml/official/suite`.
* No artificial numeric/sharding folders, no gratuitous rewrites and no mass visibility widening. Group files by actual owner and responsibility; preserve Protos language semantics, test identity/discovery, package-private encapsulation, Truffle Bytecode DSL generated classes, ServiceLoader/Truffle registrations and paths used by tooling.
* Keep the existing formal Issue I103 open over the bounded work slices. Do not create micro-issues to represent execution steps. Keep each published intermediate revision coherent; do not insert a relaxed, grandfathered or silently bypassed CI gate.
* Validation and commit/push of `guillermomolina/protos` are **human-executed**, with the agent owning edits and analysis. Documentation-only publication in `guillermomolina/protos-project-docs` may be agent-direct where explicitly authorized. Do not touch `pom.xml` / `CHANGELOG.md` for an implementation slice until human-reported tests are green; once changed, no tests after that metadata step.

## Audited baseline

See [the revision-pinned 2026-10-15 inventory](../../evidence/I103/I103_BASELINE_DIRECT_FILE_INVENTORY_2026_10_15.md): **10** first-party directories above 25; **886** counted files beyond the 25-per-directory allowance. **15** separate offending directories belong to the exact, pinned third-party TOML fixture snapshot and should be deliberately excluded.

Original reported Java counts of 446/231/104/79 were reproduced from the published Git trees. Additional first-party offenders are `protos/tests/tooling` (94), `dist` (54), `tools` (38), `protos/lib/core` (34), `protos/tests/conformance/library/io` (28), and `protos/tests/conformance/library/toml/official/suite` (28).

## Responsibility-first target, subject to dependency inspection

* `execution`: Bytecode/lowering/invocation; bootstrapping and embedding; modules/packages/workspace; foreign providers/value projection; Core protocols; IO/network/filesystem; Test Tool and debugger. Group together classes sharing package-private interfaces and Truffle generation invariants rather than force a filename-only taxonomy.
* `runtime`: actors/groups, tasks/futures, values/numerics, lexical/context/control, IO/filesystem/network, Process and interop. Preserve ordinary shared/runtime owner interfaces and stable Core value identities.
* `src/test`: co-locate private/package-level test support with the appropriate implementation owner when possible; organize by **behavior** rather than historical Issue/PERF identifiers. Preserve JUnit/Surefire discovery and exact impact-router selection.
* `protos/lib/core`: remain **Core**, not `std:` modules. Any physical move must preserve bootstrap ordering, module names, canonical identities, distribution/internal resources and source path authority; inspect `ProtosCoreBootstrap`, `ProtosCoreResource` and the Core architecture record.
* `protos/tests/tooling`: suite-independent, often JUnit-owned fixtures, not automatically `suite-native`. Preserve external consumers and file-specific selectors.
* `protos/tests/conformance/library/io`: 28 direct files and therefore **already compliant** at the owner-revised limit of 30; preserve the cohesive read-all, write-all and Process-stream layout rather than split merely for the quota.
* TOML `official/suite`: 28 first-party generated shards are **already compliant** at 30; do not reorganize the generator, `Harness.protos` imports or manifest solely to remove three files. Preserve the exact upstream pinned `v2.2.0` corpus and its precise exemption.
* `dist` and `tools`: organize executable entry points, supporting libraries, self-tests and baselines by actual purpose; update Make, CI, scripts and import/path references. Avoid extracting scripts that external callers expect at stable paths without an evidenced compatibility plan.

## Execution plan (informal slices inside I103)

| Slice | Responsibility | Publication boundary |
| --- | --- | --- |
| A | Deterministic inventory and guard with self-tests; bounded low-coupling reorganization of `tools/` and `dist/` if supported by actual dependency closure | Guard may exist as an explicit manual check/report but **must not be made a failing prerequisite** while first-party violations still exist. No suppressions or relaxed enforcement. Split A internally if script movements prove too broad. |
| B | Responsibility-based layout of the 94 direct, predominantly JUnit-owned fixture files in `protos/tests/tooling`; update guard threshold from 25 to 30 | Preserve fixture discovery and exact path consumers; do not revisit already-compliant 28-file IO or TOML directories |
| C | Core physical layout and bootstrap/source references | Same identities, order, resource contents and embedding semantics |
| D | `runtime` Java packages and related JUnit | Package visibility, actor/process lifecycles and interop stable |
| E | `execution` modules/packages/workspace/foreign and matching JUnit | Module identity/interop boundary stable |
| F | `execution` protocols/IO/debugger/Test Tool and matching JUnit | Standard mechanisms and tooling contract stable |
| G | Remaining `execution` Bytecode/lowering/invocation + JUnit | No change to Bytecode DSL, lowering, compiler optimization or ABI behavior |
| H | Complete residual census and **strict guard activation in `make check` and actual CI route**; final integrated acceptance | Zero first-party offender directories; reproducible guard evidence and human-reported `make test` PASS |

These are **planning slices, not formal child Issues**. Reevaluate the names/order on actual HEAD before editing. Prefer grouping small safe changes and splitting unexpectedly large patches. In particular, `dist/` has 54 files (26 `test_*.py`) and `tools/` has 38; moving all scripts blindly is not an approved implementation method.

## Guard requirements and acceptance gates

Preferred implementation: a small Python standard-library script under `scripts/` and its self-test. Count **direct** tracked regular files; stable lexicographic diagnostics for **all** offenders; fail closed on Git/index errors or unmerged entries; explicit exact upstream exemption. Test 29/30/31, parent with nested child, tracked/untracked, symlink, exec bit, determinism, exemption exactness, and failure modes. Preserve a reporting mode useful for migration audits but do not interpret report-only success as the strict invariant.

The existing CI workflow is `.github/workflows/tests.yml`: it executes `make test`, not `make check`. Final activation must therefore call the strict guard from actual CI **and** expose a `make check-directory-file-limit` target through `make check`, without accidentally imposing a full test run on routine static checks. Do not integrate a final strict failing gate until all first-party sources comply.

For every published slice, obtain actual current HEAD, changed-path set, `git diff --check`, targeted static checks, the complete affected regression set and the prescribed top-level validation profile. A final whole-item closure candidate requires integrated `make test` under `AGENTS.work/IMPLEMENTATION.md`. No PASS inferred from inspection. Publish product first; publish exact-revision evidence after.

## Verified source/dependency touchpoints (baseline)

* `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `src/AGENTS.md`, `protos/AGENTS.md`, `AGENTS.work/TEST.md`.
* `Makefile` and `.github/workflows/tests.yml`.
* `scripts/validation_impact.py`: path-specific detection of Tool/Package code and tests currently under `src/**/execution/`.
* `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`: package-private and referenced by many other source/tests; Bytecode DSL-root code cannot be relocated by string replacement.
* `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`: Truffle registration and internal Core resource link.
* `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`: explicit `coreDirectory.resolve("<Name>.protos")` sequence.
* `src/main/java/com/guillermomolina/protos/execution/ProtosStandardLibraryModuleResolver.java`: exact, confined `std:` paths.
* `src/test/java/com/guillermomolina/protos/execution/ProtosHostedExecutionTestFixture.java`: package-private support referenced by other tests.
* `tools/toml_test_projection.py`, `tools/test_toml_test_projection.py`, `protos/tests/conformance/manifest.tsv`, TOML `suite/Harness.protos`: generator, manifest and per-shard relative import.
* `docs/project/architecture/CORE_BOOTSTRAP_ARCHITECTURE.md` in the durable repository: Core semantic ownership and source naming invariants.
* `spec/semantics/MODULES.md` and `spec/io/PROCESS_IO.md` remain the normative backstops where a physical move would affect module/import or Process behavior.

## Evidence / limits

The maintainer accepted this recommendation on 2026-10-15 and separately reported `git diff --check` clean and all local tests passing. **I103 code was not implemented in this coordination transaction**, so these general test reports are **not** I103 post-change test evidence. No build, tests, refactor, source publication or strict guard acceptance is claimed here. This paragraph documents the pre-implementation evidence state. Slice A has since been published as recorded below; the parent Issue must stay OPEN.

## I103-A publication checkpoint (2026-10-15)

The maintainer published [product commit `cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d`](https://github.com/guillermomolina/protos/commit/cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d): deterministic 25-direct-file index guard and 15 self-tests, staged-but-not-enforced manual Makefile targets, and responsibility-based `tools/`/`dist/` restructuring. The post-staging inventory reports 25 direct files in each root and at most 18 in each affected subdirectory. Human-reported integrated `make test` and focused validation are documented with the isolated distribution-test correction and its rerun.

[Exact-revision I103-A publication and test evidence](../../evidence/I103/I103_A_PUBLISHED_DIRECTORY_GUARD_AND_TOOLS_DIST_LAYOUT_2026_10_15.md). The 25-file counts above describe immutable I103-A history; the current owner-selected threshold is 30 (revision below). Continue with I103-B against actual current HEAD; I103 remains **OPEN**, and global guard enforcement remains deferred until zero first-party violations.

## Revised directory ceiling: 30 direct files (2026-10-15)

**Owner direction:** do not create artificial subdirectories to split two otherwise coherent 28-file test folders. The active acceptance ceiling is now **30 directly tracked regular files per maintained directory**, with a guard boundary of **29/30/31** and no accumulation of child counts. This supersedes the original 25-file requirement for future I103 slices without rewriting the baseline inventory or the I103-A publication record, whose counts and test results remain immutable history.

- The 28-file first-party `protos/tests/conformance/library/io` and `protos/tests/conformance/library/toml/official/suite` directories are compliant and stay in place. This is *not* a new upstream exemption: the generated suite is still counted; the exact pinned third-party TOML snapshot alone is excluded.
- Reuse the closed [TOOL009-F/#694](https://github.com/guillermomolina/protos/issues/694) work, which consolidated 1,234 suite-native test sources to 270 while preserving all 1,263 logical Cases. No renewed source-consolidation campaign for those Cases is justified by I103.
- The current priority is the 94 direct files in `protos/tests/tooling`, a distinct, largely JUnit-consumed fixture corpus. Relocate by actual fixture consumer/contract family, not by arbitrary size or formal-issue number alone, and reconcile every exact path dependency.
- Next source publication must change `scripts/directory_file_limit.py` and its selftests to 30 as part of a human-executed product commit. Until then, the published executable still enforces the historical 25 limit when manually invoked. Final `make check` and real CI activation remains I103-H work only after zero 30-file offenders remain.

## I103-B publication checkpoint (2026-10-15)

**Exact product publication:** [`f004063207f827d70734c92ef93a0f0dc633234e`](https://github.com/guillermomolina/protos/commit/f004063207f827d70734c92ef93a0f0dc633234e), version `0.3.362-SNAPSHOT`. The guard now has **`LIMIT = 30`** and synthetic 29/30/31 tests. Ninety-four content-identical JUnit-owned Test Tool fixtures moved from the flat `protos/tests/tooling` root into 13 semantic directories (largest: 22 direct files), with Java/TSV consumers reconciled and F1/F2 filesystem roots corrected. First-party 28-file IO and TOML suite directories remain untouched. There is still **no mandatory guard** in `make check` or CI.

**Human validation:** Python unit tests 50/50 PASS; affected JUnit focal run initially had two F1/F2 failures out of 121, both repaired and successfully rerun in isolation; subsequent integrated `make test` PASS; byte-for-byte original-vs-new fixture audit `I103_B_FIXTURES=PASS`; both tracked and staged diff-checks clean. See [exact-revision I103-B publication record](../../evidence/I103/I103_B_PUBLISHED_TOOLING_FIXTURES_AND_30_FILE_LIMIT_2026_10_15.md).

**Next approved slice:** I103-C, physical Core organization, subject to auditing its exact current bootstrap/resource dependencies. At product `f0040632`, `protos/lib/core` contains 34 direct tracked files (33 `.protos` and `README.md`); five cohesive network and TCP source members (`IpAddress.protos`, `IpEndpoint.protos`, `Network.protos`, `TcpConnection.protos`, `TcpListener.protos`) are a candidate `core/network/` family. `ProtosCoreBootstrap` explicitly loads their root-relative paths and order, so preserve canonical object identity, bootstrap order, resource packaging and any additional path consumers. This is a **candidate pending current-HEAD audit**, not prior approval to alter language semantics or module identity. The layout should leave 29 files directly in `core/` and five in `core/network/` if the dependency proof succeeds; do not force the move if invariants disagree. I103-D through H remain outstanding; the parent Issue stays **OPEN**.

## I103-C publication checkpoint (2026-10-15)

**Exact Protos publication:** [`5948ee7ce70d5a70636c921e31566483c4809323`](https://github.com/guillermomolina/protos/commit/5948ee7ce70d5a70636c921e31566483c4809323), version `0.3.363-SNAPSHOT`. All five network/TCP Core sources moved unchanged into `protos/lib/core/network/` (29 files remain in Core root, five in network); corresponding `ProtosCoreBootstrap` path loads, Core naming/license architecture test and maintained IP protocol comments updated. No Truffle package format change.

**Human-reported validation:** focal 49 JUnit tests PASS (0 failures/errors/skips); integrated `make test` PASS; both diff checks clean; source/commit verification recorded in [I103-C durable evidence](../../evidence/I103/I103_C_PUBLISHED_CORE_NETWORK_LAYOUT_2026_10_15.md). No agent-executed tests or product Git actions claimed.

**Next:** I103-D runtime Java package and JUnit ownership reconciliation, with 104 direct production files and 79 direct test files in the existing runtime package at product `5948ee7c`. Java package-private dependencies and cross-package references make path-only moves insufficient; prove the real dependency closure first, group by cohesive owner, and retain compilation/testability in intermediate publications. I103-E/F/G execution Java and I103-H final zero-violation strict `make check`/CI enforcement remain outstanding. Parent I103 stays **OPEN**.

## I103-D publication checkpoint (2026-10-15)

**Exact human-published product commit:** [`d12d890db64a0a4f7b9c512521774c2c4ecaa283`](https://github.com/guillermomolina/protos/commit/d12d890db64a0a4f7b9c512521774c2c4ecaa283), version `0.3.364-SNAPSHOT`. GitHub confirms `main` at that SHA and the published Maven version.

**Runtime root direct-file counts:** production `104 -> 30`, JUnit/test `79 -> 30`; source identities remain present in responsibility-oriented subdirectories. Java `runtime` encapsulation required a mixed approach: genuine subpackages where safe, physical-only groupings that retain `package com.guillermomolina.protos.runtime` where package-private contracts and Truffle exports need it. Relocated-source path consumers and structural source-reading assertions were reconciled without weakening test coverage.

**Maintainer validation:** focused JUnit runs of 20 and 24 tests PASS; `test-compile` PASS; first full Java run identified three stale physical paths in structural tests (0 assertions failed), repaired before the maintainer-reported integrated `make test` PASS. Human executed Git publication; documentation agent neither ran the tests nor published product source.

**Exact evidence:** [I103-D runtime layout publication record](../../evidence/I103/I103_D_PUBLISHED_RUNTIME_JAVA_LAYOUT_2026_10_15.md), anchored to the above Protos commit. No global strict-guard acceptance yet.

**Next planned slice:** **I103-E**, Java `execution` module/package/workspace and foreign-provider/interop physical organization plus focused tests, starting from the live product HEAD. At this checkpoint, `src/main/java/com/guillermomolina/protos/execution` has **240** direct Java files and `src/test/java/com/guillermomolina/protos/execution` has **447**. I103-F/G must finish remaining `execution` owners and I103-H must pass a policy-aware global Git-index census before enabling strict `make check` and actual CI enforcement. Parent I103 stays **OPEN**.
