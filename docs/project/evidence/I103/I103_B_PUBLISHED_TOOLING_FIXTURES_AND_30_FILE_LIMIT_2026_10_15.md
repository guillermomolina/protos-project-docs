# I103-B — Published Test Tool fixture layout and 30-file directory guard (2026-10-15)

**Record type:** exact-revision intermediate implementation and maintainer-reported validation evidence; **not** closure of parent I103.
**Live owner:** [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
**Published product revision:** [`f004063207f827d70734c92ef93a0f0dc633234e`](https://github.com/guillermomolina/protos/commit/f004063207f827d70734c92ef93a0f0dc633234e).
**Implementation version:** `0.3.362-SNAPSHOT` (updated `pom.xml` and the root `CHANGELOG.md` after green tests).
**Preceding I103-A publication:** [`cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d`](https://github.com/guillermomolina/protos/commit/cfe41f56eec4ae9cb2c1b582d8ad05ee6ad0f36d).

## Product delta (verified against published GitHub revision)

- `scripts/directory_file_limit.py`: `LIMIT = 30`. Git-index direct regular-file policy remains unchanged: only tracked regular modes, no descendant accumulation, narrow pinned `toml-test v2.2.0` upstream exemption, deterministic reporting, fail-closed malformed/unmerged index behavior.
- `scripts/test_directory_file_limit.py`: new 29 PASS, 30 PASS and 31 FAIL boundaries, parent/child independence and exact TOML exemption; `scripts/test_validation_impact.py` covers relocated Test Tool paths. Do **not** make the directory guard mandatory in `make check` or CI until I103-H.
- 94 original directly contained `protos/tests/tooling` files relocated **without byte-content changes** into 13 responsibility-based directories, keeping JUnit rather than suite-native ownership.
- Java/JUnit consumers, TSV manifest roots and retained Future shape source paths reconciled. `ProtosTestToolFutureResolvedMechanismTest` and `ProtosTestToolFutureTerminalMechanismTest` now root their filesystem capabilities at `protos/tests/tooling/future/lifecycle`. No production Java/runtime semantics modified.
- The 28-file first-party conformance `library/io` and TOML `official/suite` directories and the exact upstream TOML exemption were left alone.

## Exact tooling layout after relocation

All counts below are direct files in `protos/tests/tooling/`, not recursive roll-ups:

| Relative directory | Direct files |
| --- | ---: |
| `discovery` | 4 |
| `execution` | 6 |
| `expectations` | 6 |
| `numeric` | 18 |
| `packages` | 5 |
| `future/lifecycle` | 7 |
| `future/stored-inspection` | 22 |
| `future/fresh-inspection` | 3 |
| `future/observation` | 7 |
| `future/common` | 2 |
| `scheduling` | 5 |
| `results-reporting` | 6 |
| `logical-cases` | 3 |
| **Total** | **94** |

Original `protos/tests/tooling` root: **0** direct fixture files. Largest new directory: **22** direct files, within the 30-file ceiling.

## Human-executed validation evidence

| Gate | Maintainer report |
| --- | --- |
| Python guard and validation-impact unit tests | `Ran 50 tests ... OK` |
| Affected JUnit suite | First focal run: 121 tests, two failures in Future F1/F2 due to old filesystem roots; after a targeted root correction, both failed tests were rerun and passed; no need to re-run the other 119 |
| Integrated `make test` | `PASS`, as reported by the maintainer |
| Fixture integrity | `I103_B_FIXTURES=PASS`: 94 original files, 94 relocated files, identical contents |
| Formatting/delta | Maintainer reports `git diff --check` and `git diff --cached --check` clean |
| Changelog finalization | `0.3.362-SNAPSHOT` entry was added and historical path rewrites were reverted before publication |
| Product publication | GitHub confirms the exact `f004063207f827d70734c92ef93a0f0dc633234e` commit |

The documentation agent independently inspected the committed source paths, guard threshold, boundary tests, F1/F2 roots, project version and changelog; the agent **did not execute** any build, tests, or local Git validation. `make test` PASS and the fixture hash census are maintainer-provided evidence.

## Remaining I103 scope

Parent [I103/#897](https://github.com/guillermomolina/protos/issues/897) stays **OPEN**: zero violations across the *entire* first-party repository has not been established, and the strict guard remains manual-only. Proceed with the approved Core physical organization (I103-C), then runtime Java and test packages, execution Java and associated JUnit (D/E/F/G), and final enforcement in `make check` plus the actual CI route (H). Re-inventory the current HEAD before each slice.

See [accepted implementation plan](../../work/I103/I103_DIRECTORY_FILE_LIMIT_IMPLEMENTATION_PLAN.md) and [I103-A published evidence](I103_A_PUBLISHED_DIRECTORY_GUARD_AND_TOOLS_DIST_LAYOUT_2026_10_15.md).
