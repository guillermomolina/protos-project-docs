# I153 — Owner-directed implementation closure with official Hurl exception (2026-11-23)

## Identity and closure scope

- **Owning issue:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), `guillermomolina/protos`.
- **Published application:** [`guillermomolina/protos-realworld-example-app@9ee38c0930c912cd6ceef598231dc77e28ae7df0`](https://github.com/guillermomolina/protos-realworld-example-app/commit/9ee38c0930c912cd6ceef598231dc77e28ae7df0); 21 changed paths in the SQLite implementation publication.
- **Published Protos dependency:** [`guillermomolina/protos@9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4); I170 generic FFM Native integration, `0.4.192-SNAPSHOT`.
- **Earlier exact acceptance record:** [I153 SQLite Native publication and validation (2026-11-22)](I153_SQLITE_NATIVE_FFM_PUBLICATION_AND_ACCEPTANCE_2026_11_22.md), originally published at [project-docs `9db1ee058094c85aa1231d4857ea07c3aba594ca`](https://github.com/guillermomolina/protos-project-docs/commit/9db1ee058094c85aa1231d4857ea07c3aba594ca).

On 2026-11-23 the owner explicitly directed **closure of I153 as implemented**, while expressly identifying that **official, unmodified RealWorld Hurl conformance is still missing**. This is an owner-directed acceptance of the *published SQLite-backed application implementation* with a disclosed exception to the issue's original all-official-Hurl-green and pinned-clean-checkout completion criteria. It is **not** a claim of full RealWorld API conformance, a change to the external RealWorld specification, or evidence that the failed official test passed.

This newer owner direction supersedes the earlier *open-until-official-green* coordination disposition for I153 only. The original criterion and historical 2026-11-22 evaluation remain auditable in the issue and earlier evidence record. Live closed/open state remains authoritative in GitHub.

## Published implementation and maintainer-reported validation

The application uses SQLite as the sole authoritative persistent store, through ordinary Protos repository modules and one RootActor-owned connection, with Native generic FFM authorized by `--allow-library sqlite3=libsqlite3.so.0`. The published application includes 19 RealWorld HTTP handlers, Argon2id credentials, generation-revoked HS256 JWTs, article/comment/tag/follow/favorite persistence and worker-isolated HTTP handling.

| Check | Evidence status |
| --- | --- |
| `git diff --check` | Owner reports clean; all applicable local tests also reported passing on 2026-11-23, subject to the explicitly known official-Hurl exception |
| `make test`; SQLite persistence/security; independent-process reopen | PASS, maintainer-reported in the 2026-11-22 acceptance record |
| SQLite SIGKILL recovery, lock/schema/corruption/authorization failure paths | PASS, maintainer-reported |
| `make test-http`; `make test-http-lifecycle` | PASS, maintainer-reported |
| `make test-hurl-adjusted` | **13/13 files; 157 requests; zero failures**, using a disposable suite copy with exact `--fuzz=0` refreshed-token patch |
| **Official, unmodified `make test-hurl`** | **12/13 files, NOT fully conformant**; `errors_auth.hurl` expects 200 using an older JWT after password change, whereas the application rejects the revoked JWT with 401 |
| Exact upstream Hurl checkout SHA and final clean-checkout rerun | **NOT SUPPLIED** |
| Separate JVM external-provider `no-ffm-provider` negative | **NOT RUN in supplied evidence**; integrated Native variant was not an applicable substitute |

The owner attestation of local checks does not override the explicitly observed 12/13 unmodified official suite outcome. No coordinating agent reran the application, official Hurl, JVM variant or full suite for this closure. Neither production readiness nor official catalog eligibility is established.

## Remaining external-conformance work

Full, unchanged official Hurl compliance is **unresolved and excluded from the owner's narrowed I153 implementation-closure disposition**. If pursued, it requires an independently scheduled decision/work item: pin the upstream test revision, reproduce `errors_auth.hurl` without patching, and reconcile the post-password-change JWT expectation **without silently weakening Protos credential revocation**. Do not describe the adapted Hurl suite as an official green run. No new issue, formal decision, code, test, spec, release, or upstream modification is authorized or claimed by this record.

## Closure evidence contract

```text
FORMAL_IDENTIFIER=I153
FAMILY=I
APPLICATION_REVISION=9ee38c0930c912cd6ceef598231dc77e28ae7df0
PROTOS_REVISION=9c56aba2c8b84a9c44440cb361db85b53fcf30b4
DURABLE_RECORD_DECISION=REQUIRED
OWNER_DIRECTED_IMPLEMENTATION_CLOSURE=YES_2026_11_23
LOCAL_APPLICABLE_VALIDATION=MAINTAINER_REPORTED_PASS
UNMODIFIED_OFFICIAL_HURL=FAIL_12_OF_13
ADAPTED_HURL=PASS_13_OF_13_157_REQUESTS
OFFICIAL_CONFORMANCE_CLAIM=NO
UPSTREAM_HURL_REVISION=NOT_REPORTED
CLEAN_CHECKOUT_FINAL_ACCEPTANCE=NOT_REPORTED
JVM_EXTERNAL_PROVIDER_NEGATIVE=NOT_RUN
CLOSURE_SCOPE=PUBLISHED_SQLITE_APPLICATION_WITH_DISCLOSED_OFFICIAL_HURL_EXCEPTION
```

The final issue comment must reference this published document at an exact project-record revision and the application/Protos revisions. The owning GitHub issue is the authority for whether the final state transition actually succeeded.
