# I103-A — Published directory guard and tools/dist layout (2026-10-15)

**Record type:** revision-pinned intermediate implementation and maintainer validation evidence, **not** I103 parent closure.
**Live owner:** [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
**Published product revision:** [`cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d`](https://github.com/guillermomolina/protos/commit/cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d).
**Baseline inventory revision:** [`c7004f71cb05621dd5a7f89e46459542bd7d9c87`](https://github.com/guillermomolina/protos/commit/c7004f71cb05621dd5a7f89e46459542bd7d9c87).
**Scope:** I103-A only. Project version and root implementation changelog were not changed.

## Published delta

The product commit contains 56 changed paths, with Git recognizing relocations (686 additions and 67 deletions in the staged, rename-aware summary). It introduces:

- `scripts/directory_file_limit.py`: deterministic index-driven direct-file inventory, with explicit `report` and strict `check` modes. Count only tracked regular modes `100644`/`100755`; symlinks and submodules do not count. Fail closed on malformed/unmerged index entries; list all offenders in deterministic order.
- `scripts/test_directory_file_limit.py`: 15 self-tests, including 24/25/26 boundaries, independent parent/child counts, deterministic multi-offender reporting, symlinks, executables, untracked files, exact exemption and malformed-index errors.
- Exact pinned third-party exemption: `protos/tests/conformance/library/toml/official/upstream/v2.2.0/`. First-party `official/suite/` is **not** exempt.
- `Makefile` manual `directory-file-report` and `directory-file-check` targets. **No failing guard prerequisite in `make check` or CI yet**, because the first-party violations outside this slice remain.
- Responsibility-based moves: 11 `tools/test_*.py` modules to `tools/tests/`; 2 diagnostics to `tools/diagnostics/`; 17 release tests to `dist/tests/release/`, 8 runtime tests to `dist/tests/runtime/`, and 4 shell smoke scripts to `dist/smoke/`. Supporting package initializers, imports, invocation paths, Makefile, distribution/embedding documentation, and two Java test path references were adjusted.
- `dist/test_external_foreign_providers.py` deliberately remains at its original entry-point path.
- No changes under `src/main/` or `protos/lib/`; no Protos language, Standard Library, runtime or ABI semantic revision is claimed.

## Exact index evidence after staging

The maintainer ran `git ls-files --cached --stage -z` against the staged product candidate and counted only direct regular files:

| Directory | Direct files | Limit | Result |
| --- | ---: | ---: | --- |
| `tools` | 25 | 25 | PASS |
| `tools/tests` | 12 | 25 | PASS |
| `tools/diagnostics` | 3 | 25 | PASS |
| `dist` | 25 | 25 | PASS |
| `dist/smoke` | 4 | 25 | PASS |
| `dist/tests` | 1 | 25 | PASS |
| `dist/tests/release` | 18 | 25 | PASS |
| `dist/tests/runtime` | 9 | 25 | PASS |

The maintainer reported `I103A_DIRECTORY_INDEX_CHECK: PASS` and empty unstaged and untracked file lists before committing. Compared with the fixed baseline inventory, the direct counts for `tools` and `dist` decrease from 38 and 54 to 25 each. This table is **not** a current-HEAD census of every other repository directory.

## Human-executed validation

The maintainer reported these results during I103-A implementation:

| Validation | Result |
| --- | --- |
| `python3 -m compileall -q` for new/relocated Python | PASS |
| `python3 scripts/test_directory_file_limit.py` | 15 tests, OK |
| `python3 -m unittest discover -s tools/tests -p 'test_*.py' -v` | 278 tests, OK |
| `python3 -m unittest discover -s dist/tests -t dist -p 'test_*.py' -v` | 234 tests, initially **one ERROR** |
| Focused rerun of `ReleaseCandidateMaterializationTest.test_materializes_unpublished_local_main_baseline` | 1 test, OK after making the fixture's Git initial branch explicitly `main` |
| `python3 tools/tests/test_verify_toolchain.py` | `TOOLCHAIN_VERIFIER_TESTS: PASS` |
| Static shell syntax checks on the relocated smoke scripts and `dist/validate_portable.sh` | Maintainer reported PASS |
| `make test` integrated Java/Protos suite | Maintainer reported PASS |
| Staged `git diff --cached --check` | Clean |
| Staged direct-file index audit | `I103A_DIRECTORY_INDEX_CHECK: PASS` |

The full 234-test distribution discovery was **not** repeated after the one failing fixture was corrected; the failing case was rerun successfully in isolation. The documentation agent did not run any of these tests. GitHub inspection independently confirmed that the product commit was published at the exact SHA above, not its runtime test outcome.

## Outstanding I103 work

At the older baseline, **10** maintained first-party directories exceeded 25 direct files, by **886** aggregate excess files; `tools` and `dist` were two of these. The other eight baseline offenders remain the scope of subsequent slices **subject to re-inventory at each actual HEAD**: Java execution/runtime production and test packages, Protos Test Tool fixtures, `protos/lib/core`, first-party IO tests, and the generated TOML suite.

Next approved planning slice: I103-B, responsibility-based restructuring of the first-party Protos Test Tool/IO/TOML test layouts. Later slices own Core, Java runtime and execution packages. Final I103-H must produce a zero-offender current census and activate the **strict** guard in both `make check` and the real CI route running `make test`.

**Parent issue remains OPEN.** This record establishes published slice A only; it is not a waiver of the final 25-file invariant.

See [accepted implementation plan](../../work/I103/I103_DIRECTORY_FILE_LIMIT_IMPLEMENTATION_PLAN.md) and [immutable baseline inventory](I103_BASELINE_DIRECT_FILE_INVENTORY_2026_10_15.md).
