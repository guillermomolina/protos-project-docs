# I158 — Taskless Actor bootstrap suspension (2026-11-08)

Issue: [I158 #1091](https://github.com/guillermomolina/protos/issues/1091). Owner-approved design: [D251-A1 #1088](https://github.com/guillermomolina/protos/issues/1088).

The implementation permits explicit `Future.value()` suspension in an INITIALIZING Actor's initial SOURCE/MODULE, nested imports and bootstrap binding without allocating an implicit Task or dispatching unrelated same-Actor guest work. Actor-local ownership persists across suspension; safe internal Future completions resume at the suspended point. It preserves single READY cutover, lifecycle/stop custody, SOURCE/MODULE import identity and Process/Context cleanup. Principal additions: `ProtosActorBootstrapSuspension.java`, `ProtosActorBootstrapCPrimeExecution.java`; integrations in `ProtosActorBootstrap.java`, Actor scheduler, execution domain, module runtime, RootActor service hosting, and normative Actor/Future/MODULE specs.

## Revision and maintainer-reported validation

- Product revision: [`b27a925367047be04c7a272501624ff4ded462c0`](https://github.com/guillermomolina/protos/commit/b27a925367047be04c7a272501624ff4ded462c0), published to `main` on 2026-11-08.
- Version: `0.3.455-SNAPSHOT` (version bump after green tests; no tests after bump).
- Product commit: 85 files, +14,027 / -640 lines; final working tree clean.
- Final `make test`: PASS (`FINAL_MAKE_TEST_EXIT=0`); final `make check`: PASS (`FINAL_CHECK_EXIT=0`); `git diff HEAD --check`: PASS.
- `make test` runs Java and Protos tests, **not** `make test-native`. No rebuilt Native Image or installed Native executable test result was provided for this product revision. The implementation contains Native smoke harnesses, but their execution is **not claimed**.

These are exact-product-revision implementation evidence and owner-reported validation, not a new design approval or a public release certification.

- I158 dedicated class: 23 tests passed in the final combined regression.
- Two rounds of Process Context/I158 concurrent tests: 252 executions, zero failures/errors/skips.
- Combined FFM/Filesystem/host/I158 suite: 134 tests, zero failures/errors, two environmental skips.
- Exact Execution/Captured Process/Fresh Process classes: 22 tests passed after awaiting Process termination and Context disposition before Polyglot host close.
- No implicit suspend on `import`, I/O calls or a returned Future; only explicit pending `Future.value()` is newly eligible in initialization.

`PROTOS_REVISION=b27a925367047be04c7a272501624ff4ded462c0`
