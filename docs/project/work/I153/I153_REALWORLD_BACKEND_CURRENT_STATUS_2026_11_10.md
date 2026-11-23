> **OWNER-DIRECTED IMPLEMENTATION CLOSURE — 2026-11-23:** The owner explicitly requested closure of [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) as implemented while acknowledging the unresolved official RealWorld Hurl gate. The published SQLite application is [`RealWorld@9ee38c0930c912cd6ceef598231dc77e28ae7df0`](https://github.com/guillermomolina/protos-realworld-example-app/commit/9ee38c0930c912cd6ceef598231dc77e28ae7df0) against [`Protos@9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4). Maintainer-reported SQLite/restart/recovery/HTTP tests and the **adjusted** Hurl suite (13/13; 157 requests) pass; **official unmodified Hurl still fails 1/13** on post-password-change revoked JWT behavior (expected 200 vs observed 401). The exact upstream checkout SHA and independent final clean-checkout run remain unreported. The owner accepts **implementation closure with this explicit exception**, not full RealWorld conformance. [Dedicated closure evidence](../../evidence/I153/I153_OWNER_DIRECTED_IMPLEMENTATION_CLOSURE_OFFICIAL_HURL_EXCEPTION_2026_11_23.md). Live state is authoritative in the Issue. Earlier dated status blocks below are retained as history, not current coordination instructions.

> **CURRENT PUBLICATION / ACCEPTANCE — 2026-11-22:** RealWorld's D255-SQL1 SQLite-only application was published and independently re-read at [`guillermomolina/protos-realworld-example-app@9ee38c0930c912cd6ceef598231dc77e28ae7df0`](https://github.com/guillermomolina/protos-realworld-example-app/commit/9ee38c0930c912cd6ceef598231dc77e28ae7df0), using the Native generic FFM capability published under [I170 / Protos `9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4). The maintainer reports clean diff, SQLite persistence/security, independent-process reopen, SIGKILL recovery, HTTP/lifecycle and **adapted Hurl 13/13 / 157 requests** PASS. **Unmodified official Hurl remains 12/13**, because it reuses a credential-generation-revoked JWT after password change. The local upstream test SHA and final clean-checkout acceptance were not supplied; the JVM-specific absent-provider scenario was not run. **I153 remains OPEN / Review** until its explicit official-green and reproducibility acceptance conditions are reconciled. [Revision-bound implementation and validation evidence](../../evidence/I153/I153_SQLITE_NATIVE_FFM_PUBLICATION_AND_ACCEPTANCE_2026_11_22.md); [I153 #1068](https://github.com/guillermomolina/protos/issues/1068). The historical 2026-11-10 snapshot below is retained verbatim.

> **CURRENT STATUS — 2026-11-22, D255-SQL1 OWNER APPROVED:** The owner explicitly approved **SQLite as RealWorld's only authoritative data store**, with one RootActor-owned Connection and private ordinary Protos repositories. This supersedes the previous D255-H1 application journal *for I153* without erasing the historical approval or historical test evidence. The reusable [D256 SQLite specification `0.1.507`](https://github.com/guillermomolina/protos/commit/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5) and LIB031 implementation are already published. [Canonical approved D255-SQL1 design](../../decisions/language/D255_SQL1_REALWORLD_SQLITE_SINGLE_AUTHORITY_RATIFIED_2026_11_22.md) and [comparison/approval/invariant evidence](../../evidence/D255/D255_SQL1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_22.md). **I153 #1068 stays OPEN** until the separate [RealWorld source repository](https://github.com/guillermomolina/protos-realworld-example-app) implements and validates application persistence, restart, JWT revocation and official-conformance requirements. The 2026-11-10 snapshot below is preserved as dated evidence, not current permission to use an application journal. No application code or tests changed by this coordination.

# I153 — RealWorld backend current implementation status (2026-11-10)

**Formal owner:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), in `guillermomolina/protos`. This record is non-normative and does not replace the issue, RealWorld official API authority, or Protos `spec/`.

**Published application:** [guillermomolina/protos-realworld-example-app](https://github.com/guillermomolina/protos-realworld-example-app), `main` revision [`56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c) (2026-11-09, latest inspected 2026-11-10).

**Current status:** 19/19 required HTTP route handlers implemented, security integration and local test gates reported green; durable persistence and recovery are absent, so I153 **remains OPEN**. See [source and validation evidence](../../evidence/I153/I153_REALWORLD_PUBLISHED_APP_ACCEPTANCE_GAP_2026_11_10.md).

## Implemented architecture

- Two HTTP Worker Actors receive TCP sessions on loopback, snapshot bounded complete HTTP request data and send it to a state-owning RootActor; workers do not receive the mutable domain store, JWT signing key, Clock or Entropy grants.
- 19 endpoint handlers cover registration/login/account, profiles/following, article CRUD/feed/list/detail, comments, favorites and tags.
- Ordinary Protos modules implement application logic; Standard Library Argon2id and HS256 JWT with explicit Network, Entropy and Clock grants provide authentication without a bespoke Java server or application-private crypto module.
- Request validation, status/error paths, authorization/ownership, viewer-aware article/profile projection, pagination and graceful drain/abort are covered by published source and maintainers' reported checks.
- `make serve` runs a loopback development service with ephemeral signing key when not supplied; this is not production deployment.

## Reported validation and exact limitations

The application's README and TODO (at the revision above) attribute to the maintainer, on 2026-11-09:

| Gate | Reported outcome |
| --- | --- |
| `make test` | 12 test groups PASS, including secure auth |
| `make test-http` | Authenticated real TCP `REALWORLD_NATIVE_HTTP=PASS` |
| `make test-http-lifecycle` | Five checks PASS: parallel Workers, request envelope, body limit, drain, abort |
| **Unmodified official RealWorld Hurl** | **12/13 files**, not complete |
| `make test-hurl` (temporary token-refresh adjustment) | **13/13 files, 157 requests** PASS |

The original official `errors_auth.hurl` reused a JWT that the application had revoked after a password change; the tracked `src/tests/hurl-refresh-token.patch` replaces it in a temporary suite copy with the returned token. The external spec checkout remains unmodified. This is **adjusted-suite conformance evidence**, never a claim that the unmodified official suite passed. The exact external RealWorld specification checkout revision was **not recorded**. The agent did not independently execute the tests.

## Outstanding completion gates

1. **Durable persistence:** replace/extend `MemoryStore` with durable state for users, Argon2id password records, subjects and credential generations, articles, comments, follows and favorites; restore correct identity/relationships after a restart.
2. **Restart acceptance:** reproducible durable restart/recovery tests and application-level acceptance. Current restart loses all data; a stable JWT key alone does not fix this.
3. **Official conformance reconciliation:** retain truthful distinction of 12/13 unmodified vs 13/13 adjusted and capture exact official upstream test revision for reproducible conformance reporting. Resolve or explicitly record the remaining mismatch against official authority before asserting full unmodified conformance.
4. **Clean-checkout acceptance:** document and verify the final required run/configuration and associated exact Protos and external-spec revisions. Existing README requires a Protos `0.3.459-SNAPSHOT` development baseline, not a released stable distribution.

SQLite/FFI investigations [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051) and [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052) are **potential approaches**, not newly declared hard blockers. Production HTTPS, hardening and upstream catalog listing are outside the original closure criteria for the local learning application unless separately scheduled.

No source edits, runtime tests, new implementation issues, normative changes or version bump are claimed by this status update.

## D255-H1 design checkpoint ratified (2026-11-10)

The owner explicitly approved **D255-H1 (E+A)** for application-owned, single-writer Protos-first durability over the existing explicitly provisioned confined Filesystem. See [D255 ratification](../../decisions/language/D255_PROTOS_FIRST_DURABLE_PERSISTENCE_H1_RATIFIED_2026_11_10.md), [research and GITHUB021 evidence](../../evidence/D255/D255_H1_GITHUB010_GITHUB021_APPROVAL_AND_COMPARATIVE_EVIDENCE_2026_11_10.md), and [I153 bounded handoff](../D255/D255_H1_IMPLEMENTATION_HANDOFF_2026_11_10.md).

D255's *design* dependency is resolved upon its verified durable publication/closure, but this **does not fulfill I153's remaining acceptance**: no journal, durable recovery, crash tests, or successful restart is claimed at the inspected RealWorld revision. The native `blocked_by` link may remain as historical dependency; I153 stays OPEN / `status:ready` pending implementation and separate official-Hurl reconciliation. Optional SQLite `std:ffi` work (I144/LIB028/LIB029) remains independent and is not an I153 prerequisite. No new I issue is warranted while the work remains application-local slices owned by I153.
