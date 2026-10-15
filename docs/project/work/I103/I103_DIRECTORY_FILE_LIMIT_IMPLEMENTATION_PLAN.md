# I103 — Directory direct-file limit implementation plan

Status: **IMPLEMENTATION STRATEGY ACCEPTED; I103-A PUBLISHED; I103 OPEN** (2026-10-15).
Live owner: [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
Baseline product revision: [`c7004f71cb05621dd5a7f89e46459542bd7d9c87`](https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87).
Scope: non-semantic physical organization of tracked first-party files; this is not a new Dxxx/PLATxxx architecture selection.

## Accepted contract

* At final publication, every maintained first-party repository directory has at most **25 directly contained tracked regular files** (25 passes, 26 fails). Discover directories recursively; do not accumulate descendant counts into parent directories.
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
* `protos/tests/conformance/library/io`: cohesive read-all, write-all and Process-stream domains; update the manifest/discovery paths deliberately.
* TOML `official/suite`: first-party generated shards; update the deterministic generator `tools/toml_test_projection.py`, derived paths, `Harness.protos` imports and manifests together. Leave the upstream pinned `v2.2.0` corpus byte-for-byte unchanged.
* `dist` and `tools`: organize executable entry points, supporting libraries, self-tests and baselines by actual purpose; update Make, CI, scripts and import/path references. Avoid extracting scripts that external callers expect at stable paths without an evidenced compatibility plan.

## Execution plan (informal slices inside I103)

| Slice | Responsibility | Publication boundary |
| --- | --- | --- |
| A | Deterministic inventory and guard with self-tests; bounded low-coupling reorganization of `tools/` and `dist/` if supported by actual dependency closure | Guard may exist as an explicit manual check/report but **must not be made a failing prerequisite** while first-party violations still exist. No suppressions or relaxed enforcement. Split A internally if script movements prove too broad. |
| B | First-party Protos test suite layouts, including Test Tool fixtures, IO and generated TOML projection | Stable selectors, manifest and generator output |
| C | Core physical layout and bootstrap/source references | Same identities, order, resource contents and embedding semantics |
| D | `runtime` Java packages and related JUnit | Package visibility, actor/process lifecycles and interop stable |
| E | `execution` modules/packages/workspace/foreign and matching JUnit | Module identity/interop boundary stable |
| F | `execution` protocols/IO/debugger/Test Tool and matching JUnit | Standard mechanisms and tooling contract stable |
| G | Remaining `execution` Bytecode/lowering/invocation + JUnit | No change to Bytecode DSL, lowering, compiler optimization or ABI behavior |
| H | Complete residual census and **strict guard activation in `make check` and actual CI route**; final integrated acceptance | Zero first-party offender directories; reproducible guard evidence and human-reported `make test` PASS |

These are **planning slices, not formal child Issues**. Reevaluate the names/order on actual HEAD before editing. Prefer grouping small safe changes and splitting unexpectedly large patches. In particular, `dist/` has 54 files (26 `test_*.py`) and `tools/` has 38; moving all scripts blindly is not an approved implementation method.

## Guard requirements and acceptance gates

Preferred implementation: a small Python standard-library script under `scripts/` and its self-test. Count **direct** tracked regular files; stable lexicographic diagnostics for **all** offenders; fail closed on Git/index errors or unmerged entries; explicit exact upstream exemption. Test 24/25/26, parent with nested child, tracked/untracked, symlink, exec bit, determinism, exemption exactness, and failure modes. Preserve a reporting mode useful for migration audits but do not interpret report-only success as the strict invariant.

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

[Exact-revision I103-A publication and test evidence](../../evidence/I103/I103_A_PUBLISHED_DIRECTORY_GUARD_AND_TOOLS_DIST_LAYOUT_2026_10_15.md). Continue with approved I103-B work against actual current HEAD; I103 remains **OPEN**, and global guard enforcement remains deferred until zero first-party violations.
