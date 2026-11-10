# I153 — Published RealWorld application and acceptance gap evidence (2026-11-10)

## Inspected published sources

- [Application revision `56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c), on `main` at inspection.
- [README](https://github.com/guillermomolina/protos-realworld-example-app/blob/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c/README.md) — architecture, 19 routes, test commands, security controls, reporting caveats, memory-store limitation.
- [TODO](https://github.com/guillermomolina/protos-realworld-example-app/blob/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c/TODO.md) — durable persistence and restart tests outstanding, deploy hardening separately optional.
- [Makefile](https://github.com/guillermomolina/protos-realworld-example-app/blob/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c/Makefile) — explicit `make test`, `make serve`, `make test-http`, `make test-http-lifecycle`, `make test-hurl` recipe; temp copy of upstream Hurl with patch.
- [Application source tree](https://github.com/guillermomolina/protos-realworld-example-app/tree/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c/src/app) — module names for API, HTTP Worker, authentication and MemoryStore.
- [I153 original acceptance](https://github.com/guillermomolina/protos/issues/1068): published code, complete official backend contract, unmodified official green Hurl conformance, meaningful tests, durable restart persistence, and reproducible clean checkout.

## Reported validation; not independently re-executed

From the committed README and TODO, the maintainer reports 2026-11-09 results against Protos `0.3.459-SNAPSHOT`:

- `make test`: **12 groups PASS**, secure auth included.
- `make test-http`: **PASS** authenticated HTTP/TCP.
- `make test-http-lifecycle`: **five checks PASS**.
- Unmodified RealWorld Hurl: **12/13 files**.
- `make test-hurl` with `src/tests/hurl-refresh-token.patch` applied to a disposable copy: **13/13 files, 157 requests PASS**.
- Hurl mismatch: a test continues using the JWT revoked on password change; the adjusted run uses the newly issued token. It does not modify the upstream checkout or justify an unmodified-suite PASS assertion.
- Exact upstream RealWorld spec/Hurl SHA is **not recorded** in the evidence. The application's `0.3.459-SNAPSHOT` Protos baseline identifies a development version, not an independently verified exact Protos commit for all runs.

## Distinct completion boundaries

**Present:** published 19-route application and module architecture; maintainer-reported local/HTTP/lifecycle passes and adapted Hurl PASS.

**Missing:** durable storage; restart/recovery test and demonstration; unmodified official Hurl 13/13 or a documented accepted reconciliation; final reproducible complete application acceptance against pinned revisions.

**Status recommendation:** I153 is now actionable **READY**, not an untouched INBOX issue and not DONE. Priority intentionally unset and assignee unchanged; no new hard dependencies inferred.

## Notes

This is a snapshot of inspected public project artifacts and already-reported evidence, not a new test run, performance claim, production-security certification or normative semantic statement. Do not collapse the 12/13 and 13/13 Hurl results into one claim.
