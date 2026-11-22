# I153 — SQLite-backed RealWorld Native Image publication and acceptance evidence (2026-11-22)

## Identity and authority

- **Formal work:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068); application implementation, not a Protos language or Standard Library change.
- **Exact published application:** [`guillermomolina/protos-realworld-example-app@9ee38c0930c912cd6ceef598231dc77e28ae7df0`](https://github.com/guillermomolina/protos-realworld-example-app/commit/9ee38c0930c912cd6ceef598231dc77e28ae7df0), published to `main` by the maintainer. GitHub connector independently re-read the exact commit and changed-path list: **21 files**, including SQLite repositories, RootActor integration, authentication updates, new tests and runners, Makefile, README, TODO, and Hurl patch.
- **Native FFM prerequisite:** [I170 / `guillermomolina/protos@9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4), Protos `0.4.192-SNAPSHOT`. The user-tested executable was invoked as `protos`; the terminal transcript did not independently identify its exact binary SHA or prove a fresh installed distribution artifact.
- **Design authority:** [D255-SQL1](../../decisions/language/D255_SQL1_REALWORLD_SQLITE_SINGLE_AUTHORITY_RATIFIED_2026_11_22.md) — SQLite is the sole persistent authority; one Connection in RootActor; older H1 journal is historical only.
- **Evidence source:** maintainer-supplied command output on 2026-11-22, [publication Issue comment](https://github.com/guillermomolina/protos/issues/1068#issuecomment-7079767804), and independently fetched committed application files. No build, test or Hurl run was performed by the coordinating agent.

## Published implementation (revision-bound)

The single persistent SQLite database stores identity, Argon2id password credentials and JWT generation, articles, tags, comments, follows and favorites. `src/app/SqliteDatabase.protos` and the ordinary Protos repository modules retain the database connection in the RootActor, not in HTTP Workers; `MemoryStore` remains solely for test fixtures. The application uses authorized `std:sqlite` over the existing generic FFM provider; no SQLite-specific Java application backend was introduced.

The checked-in `Makefile` uses the installed Protos executable by default, with `SQLITE_ARGS ?= --allow-library sqlite3=libsqlite3.so.0`, no required Native external JAR, and optional `PROTOS_ARGS` for the JVM `--foreign-provider-path` route. The published Hurl-adjusted recipe copies upstream tests to a disposable directory and applies `src/tests/hurl-refresh-token.patch` with **`--fuzz=0`**; the upstream checkout is not edited.

The published README and TODO were drafted while tests were pending. Their historical unchecked validation entries are superseded for the specific commands observed below, but **not** for any command lacking evidence; the publication itself does not independently execute the suite.

## Maintainer-reported checks

| Gate or scenario | Reported result | Provenance and limitation |
| --- | --- | --- |
| `git diff HEAD --check`, `git diff --cached --check` | **PASS** | No whitespace diagnostics before the application commit |
| `make test` | **PASS twice** | Twelve reported in-memory fixture groups, including secure auth |
| `make test-sqlite` | **PASS on final rerun** | Persistence and security checks; first attempt failed at `src/tests/sqlite-persistence.protos:83:25`, then the maintainer had an agent fix the defect *before* the successful rerun; not treated as a flaky test |
| `make test-sqlite-restart` | **PASS** | `SQLITE_PROCESS_RESTART=PASS`: independent-process reopen and verification, credential revocation, deliberate SIGKILL before and after confirmed commit, recovery, HTTP server data retained across restarts |
| `make test-sqlite-failures` | **PASS** | `SQLITE_PROCESS_FAILURES=PASS`: lock contention, unknown version and structure, foreign content preservation, no unauthorized database creation, absent SQLite grant, and corrupt database integrity |
| External provider removal in integrated Native mode | **NOT APPLICABLE** | `SQLITE_EXTERNAL_PROVIDER_NEGATIVE=NOT_APPLICABLE_INTEGRATED` is not a passing JVM external-provider test |
| `make test-http-lifecycle` | **PASS** | Concurrent Workers, HTTP envelope, request limit, drain and abort |
| `make test-http` | **PASS** | Against running SQLite-backed HTTP server, `REALWORLD_NATIVE_HTTP=PASS`; initial no-server connection failure was corrected by starting the server |
| Official unmodified `make test-hurl` | **12/13 files**, **157 requests executed** | `errors_auth.hurl` fails: upstream expects HTTP 200 after password change using a previously issued JWT, while app rejects the revoked JWT with 401. **Not 13/13 official conformance** |
| `make test-hurl-adjusted` | **13/13 files, 157 requests, 0 failed files** | Disposable upstream copy with new-token capture; final rerun 9.170 seconds |
| Patch exactness | **PASS** | `patch --batch --forward --fuzz=0 --dry-run` on local official `errors_auth.hurl` succeeded; final committed recipe includes `--fuzz=0` and Hurl rerun reported no fuzz warning |
| Local development prerequisites | **Observed** | `hurl --version` identified 8.1.0. Upstream suite was cloned into `/tmp/realworld-spec`; its exact local commit SHA was not supplied |

The user further attests `git diff --check` is clean and all applicable local tests pass after publication. That statement does not turn the independently observed **12/13 official Hurl** into green official conformance, nor does it prove an unexecuted JVM-specific scenario.

## Exact conformance distinction and remaining acceptance

The external suite's `errors_auth.hurl` continues to use the earlier JWT after a password update. Protos intentionally revokes credentials by generation; HTTP 401 is the observed outcome. The local compatibility patch captures `$.user.token` from the password update response and reuses the refreshed token, yielding 13/13. This is an **adapted-suite result**, not unmodified official Hurl compliance, and is not permission to weaken revocation silently.

**I153's formal acceptance text still explicitly requires green unmodified official Hurl evidence and a reproducible clean-checkout test pinned to the upstream spec.** Both remain unproven: unmodified Hurl is actually 12/13 and no exact upstream checkout SHA or independently repeated clean-checkout run was provided. JVM external-provider testing was also not reported. Application hardening (HTTPS, production secret storage, backup/restore, deployment supervision) remains outside a production-readiness claim.

Therefore the application source and core Native SQLite tests are **published and locally validated**, but formal I153 closure remains **OPEN / Review**, pending an explicit, evidence-backed reconciliation of the unmodified upstream conformance criterion and outstanding reproducibility evidence. No synthetic 13/13 official pass, unobserved test, release or final closure is claimed.

## Inspected artifacts and preservation

Published source artifacts: [Makefile](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/Makefile), [Hurl patch](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/src/tests/hurl-refresh-token.patch), [SQLite process test runner](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/src/tests/test-sqlite-processes.sh), [server entry](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/src/main.protos), [README](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/README.md), [TODO](https://github.com/guillermomolina/protos-realworld-example-app/blob/9ee38c0930c912cd6ceef598231dc77e28ae7df0/TODO.md).

Preserve the [2026-11-10 application snapshot](I153_REALWORLD_PUBLISHED_APP_ACCEPTANCE_GAP_2026_11_10.md) and [historical I153 work record](../../work/I153/I153_REALWORLD_BACKEND_CURRENT_STATUS_2026_11_10.md) as dated evidence. This new record supersedes their *current-status* claims, not the historical facts at their original revisions.
