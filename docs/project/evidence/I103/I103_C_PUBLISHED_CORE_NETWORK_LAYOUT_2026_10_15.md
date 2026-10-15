# I103-C — Published Core network source layout (2026-10-15)

**Record type:** exact-revision intermediate implementation and maintainer-reported validation; **not** I103 parent closure.
**Live owner:** [I103 / guillermomolina/protos#897](https://github.com/guillermomolina/protos/issues/897).
**Published product revision:** [`5948ee7ce70d5a70636c921e31566483c4809323`](https://github.com/guillermomolina/protos/commit/5948ee7ce70d5a70636c921e31566483c4809323).
**Project version:** `0.3.363-SNAPSHOT`, finalized in `pom.xml` and root `CHANGELOG.md` after green human-executed tests.
**Prior slice:** [I103-B evidence](I103_B_PUBLISHED_TOOLING_FIXTURES_AND_30_FILE_LIMIT_2026_10_15.md).

## Published changes

The human-published product revision moves the following five Core sources, preserving their content and case-sensitive canonical filenames:

| Former relative path under `protos/lib/core/` | Published location |
| --- | --- |
| `IpAddress.protos` | `network/IpAddress.protos` |
| `IpEndpoint.protos` | `network/IpEndpoint.protos` |
| `Network.protos` | `network/Network.protos` |
| `TcpConnection.protos` | `network/TcpConnection.protos` |
| `TcpListener.protos` | `network/TcpListener.protos` |

After the move: **29** direct regular files in `protos/lib/core/` and **5** in `protos/lib/core/network/`, both at or below the owner-approved 30-file limit.

- `ProtosCoreBootstrap` resolves five nested paths without changing source evaluation order, activation or Core identities.
- `ProtosCoreSourceNamingArchitectureTest` recognizes nested Core sources while retaining the exact canonical name inventory and Part 5 license checks.
- `ProtosStandardIpAddressProtocol` and `ProtosStandardIpEndpointProtocol` have maintained Javadoc physical-source paths reconciled.
- No change to the Truffle resource packager; its `lib/**` fileset includes nested paths. No new `std:` module, bootstrap semantics or Core concepts.
- Only the live implementation `CHANGELOG.md` received a new I103-C entry; historical entries were left untouched.

## Validation and provenance

| Gate | Evidence |
| --- | --- |
| Focused Java/JUnit checks | Maintainer reported **49 tests, 0 failures, 0 errors, 0 skipped** |
| Integrated validation | Maintainer reported **`make test` PASS** |
| Candidate metadata | Maintainer showed `git diff --check` and `git diff --cached --check` clean, and only one new `0.3.363-SNAPSHOT` changelog section |
| Prepublication state | `git rev-parse HEAD` and `git ls-remote origin refs/heads/main` both `f004063207f827d70734c92ef93a0f0dc633234e` |
| Publication | Human pushed `f0040632..5948ee7c` to `main`; linked product revision confirmed through GitHub |

The five source files were moved with a byte-preserving Python rename and source-content identity checks, as directed; the maintainer reported the relevant validations green. The documentation agent did **not** execute builds, tests, Git staging, or product publication. This record does not claim a new independent cryptographic hash census after the final commit.

## Remaining I103 work

The approved `protos/` tree reorganization is complete under the 30-file policy, subject to a final repository-wide census at I103-H. At the published revision, the remaining known oversized maintained roots are in Java:

- `src/main/java/com/guillermomolina/protos/runtime`: 104 direct Java sources.
- `src/test/java/com/guillermomolina/protos/runtime`: 79 direct Java tests/support sources.
- `src/main/java/com/guillermomolina/protos/execution`: 240 direct Java sources.
- `src/test/java/com/guillermomolina/protos/execution`: 447 direct Java tests.

These counts come from the corresponding GitHub directory listings at the publication HEAD and do **not** replace the final policy-aware Git-index inventory. Next: I103-D runtime Java ownership/package layout plus affected tests; I103-E/F/G execution Java; I103-H strict guard activation only after zero first-party offenders. The parent issue stays **OPEN**, with no final acceptance box checked.

See [accepted and updated I103 plan](../../work/I103/I103_DIRECTORY_FILE_LIMIT_IMPLEMENTATION_PLAN.md).
