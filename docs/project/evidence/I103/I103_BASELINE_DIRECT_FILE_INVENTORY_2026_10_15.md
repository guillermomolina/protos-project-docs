# I103 — Baseline direct-file inventory (2026-10-15)

**Record type:** immutable, revision-pinned investigation evidence (not acceptance of implementation).
**Product repository:** `guillermomolina/protos`.
**Product revision:** `c7004f71cb05621dd5a7f89e46459542bd7d9c87`.
**Related Issue:** [I103 / #897](https://github.com/guillermomolina/protos/issues/897).
**Inventory method:** GitHub Git tree API; recursively enumerate versioned blobs by directory, count only blobs immediately beneath each containing folder, do not accumulate descendants. The counted tree was not truncated. The full layout was audited across `src/`, `tools/`, `dist/`, `protos/` and other repository roots. Actual implementation/strict guard must separately use Git index regular-file modes and current HEAD.

## First-party directories exceeding 25 directly contained files

| Repository path | Direct files | Excess |
| --- | ---: | ---: |
| `src/test/java/com/guillermomolina/protos/execution` | 446 | 421 |
| `src/main/java/com/guillermomolina/protos/execution` | 231 | 206 |
| `src/main/java/com/guillermomolina/protos/runtime` | 104 | 79 |
| `protos/tests/tooling` | 94 | 69 |
| `src/test/java/com/guillermomolina/protos/runtime` | 79 | 54 |
| `dist` | 54 | 29 |
| `tools` | 38 | 13 |
| `protos/lib/core` | 34 | 9 |
| `protos/tests/conformance/library/io` | 28 | 3 |
| `protos/tests/conformance/library/toml/official/suite` | 28 | 3 |
| **Total** | **1,136** | **886** |

Ten first-party folders violate the 25-direct-file threshold. The excess is the theoretical minimum number of files requiring redistribution without deletion/consolidation, not an implementation count or estimate of new files.

## Outside the first-party guard contract

Fifteen additional folders containing over 25 files are under **only** the retained third-party `protos/tests/conformance/library/toml/official/upstream/v2.2.0/` subtree. It contains a pinned, byte-exact official TOML snapshot with approximately 901 tracked files in that subtree. The exact exemption must not cover the first-party generated sibling `official/suite`, its `Harness.protos`, or `tools/toml_test_projection.py`.

From inspection of the projection script: upstream TOML release `v2.2.0`, pinned commit `ce08da1ddb075d1c7596d663c7fcba9a2ae02c5c`, selected corpus `tests/files-toml-1.1.0`. Generator: `tools/toml_test_projection.py`; generated first-party suite includes 28 directly contained files. Do **not** modify pinned fixtures to evade the directory count.

## Inventory constraints and observed dependency hazards

* `tools`: 38 direct files; includes static PE guards, baselines, diagnostic scripts and tests.
* `dist`: 54 direct files, including 26 `test_*.py` scripts. Treat Python import paths, shell consumers and invoked CLI entry points as compatibility boundaries; do not assume moving scripts preserves `sys.path`.
* Core: `ProtosCoreBootstrap` explicitly loads named `.protos` files from the Core directory and determines an order. Moving Core source without updating this resolver changes behavior.
* Java: `CanonicalToBytecodeLowerer` is not public; other internal classes have package-private members and tightly coupled Truffle Bytecode DSL references; moving into child packages changes Java accessibility.
* JUnit: the package-private `ProtosHostedExecutionTestFixture`, `ProtosPerf006BytecodeTestSupport`, `ProtosPackageRunDriverTestSupport`, `ProtosTestPrelude` and debugger helper form test-placement constraints.
* `scripts/validation_impact.py` matches production/test paths directly under `execution`; paths after move must retain intended impact classification.
* `.github/workflows/tests.yml` currently runs `make test` but not `make check`; final guard CI wiring must not stop at Makefile.
* TOML generated shards import sibling `./Harness.protos`; moving a shard to a child folder requires adjusting generator-generated imports.
* Other near-boundary directories already have 22–23 files; future additions must be guarded globally instead of only patching the ten current violators.

## Acceptance provenance

The project owner **accepted the responsibility-first staged approach** on 2026-10-15. This record is investigative evidence of a published source tree, not a product patch. The owner also said local tests passed and `git diff --check` was clean; those statements were not tied to an implemented I103 delta. Neither the documentation agent nor the inventory tool executed tests, builds or code changes. All I103 final checkboxes remain open.

Source: [product at fixed revision](https://github.com/guillermomolina/protos/tree/c7004f71cb05621dd5a7f89e46459542bd7d9c87), [I103 live Issue](https://github.com/guillermomolina/protos/issues/897), [I103 implementation plan](../../work/I103/I103_DIRECTORY_FILE_LIMIT_IMPLEMENTATION_PLAN.md).
