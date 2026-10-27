# I126 — Protos-first continuous HTTP host: agent and human executor

**Date:** 2026-10-27. **Status:** implementation handoff, **not** implementation evidence.
**Owning implementation:** [I126 #967](https://github.com/guillermomolina/protos/issues/967).
**Exact initial normative basis:** [`protos@636e545de6f8ae193940d6a2138b08889c91472e`](https://github.com/guillermomolina/protos/commit/636e545de6f8ae193940d6a2138b08889c91472e), spec **0.1.474**, `spec/io/HTTP.md` §15.
**Owner-approved D216 record and proof:** [D216 ratification](../../decisions/language/D216_OPTIONAL_CONTINUOUS_HTTP_HOSTING_CONTRACT.md); [D216 publication](../../evidence/D216/D216_SPEC_0_1_474_PUBLICATION_2026_10_27.md) at `protos-project-docs@610e0485e951f8f066374d9db9b8c7db161e4dd7`.

This handoff is subordinate to the live Issue and the ratified specification. The implementation has not been started by publication of this file.

## Editing agent — next work

**Class:** IMPLEMENTATION. **Repository:** `guillermomolina/protos`. Assume the human working checkout is already at true HEAD, but refresh and reconcile current `origin/main` at every publication boundary. Follow root `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/LIBRARY.md`, `AGENTS.work/REFERENCE.md`, `protos/AGENTS.md`, scoped `src/AGENTS.md` and `spec/AGENTS.md` if applicable.

**No design investigation.** The exact semantics D216-1…D216-12 are already owner-approved and normatively ratified. Read `spec/io/HTTP.md` §15, its directly referenced `NETWORK.md`, `IO_CORE.md`, `BYTE_IO.md`, `ACTORS.md`, `FUTURES_AND_TASKS.md`, `MODULES.md`, `ERRORS.md`, the implementation blocker ledger, and only the necessary implementation/test entry points in I126 #967. Do not re-open D216 merely because implementation is difficult.

**Protos-first:** begin with `protos/lib/http/Host.protos` and the smallest necessary `internal/{HostConfig,HostLifecycle,HostWorker,HostConnection}.protos`. Reuse `std:http/Server.serveOne.owning`; do not copy its HTTP parser or framer. Use existing `TcpAcceptor` Actor.spawn delegation, ActorRef lifecycle and ordinary Futures/Tasks; one Worker accepts independently with its own fixed quota and connection Tasks. Do not introduce a Java HTTP engine or an unratified primitive. `ProtosHttpAsyncFacility.java` and `ProtosCoreBootstrap.java` are read-only examples of the existing module-private facility boundary; a genuinely required change to the bootstrap mapping must be minimal, argued and covered by tests without inventing new guest semantics.

**Consolidate work into I126, not one Issue per file.** Slice A covers actual public Host construction, strict options, private Worker bootstrap, quotas and normal lifecycle. Slice B closes all concurrent NIO, failure, cancellation, cutover and custody tests. Combine when feasible without compromising coherent intermediate state; split only on a real independently valid boundary.

**Known normative reference drift:** `spec/io/NETWORK.md` §1's generic “all other HTTP facilities deferred” sentence predates Host, and the `spec/semantics/ERRORS.md` Standard Library prototype table lacks HostError. Reconcile strictly against the ratified `HTTP.md` §15 (no semantic invention). If a true semantic conflict surfaces, stop the affected implementation and request owner guidance; otherwise correct stale references under ordinary specification revision/publication rules.

**Test locations:** existing `protos/tests/library/http/*.protos`, `src/test/java/com/guillermomolina/protos/execution/integration/http/ProtosHttpServerIntegrationTest.java` and `.../network/ProtosTcpAcceptorActorDelegationTest.java`. Add focused guest and Java true-NIO tests for option validation, same-Process worker readiness, two-client/worker concurrency, quotas, drain/abort escalation, observer cancellation, error kinds and close-once races. Demonstrate no cross-Actor resource transfer, Java HTTP engine or unbounded admission. Use the existing test harness; never modify assertions only to force green.

**External Truffle reference requirement:** no outside runtime pattern is needed for the approved design, because admission, HTTP policy and supervision are guest Protos operations. If touching Java module bootstrap, use the checked-in `ProtosHttpAsyncFacility`/Core bootstrap facility pattern; Java/Truffle is integration only. Do not port speculative GraalJS/GraalPy handler/scheduler machinery.

## Human executor — execution and publication

1. The editing agent supplies a bounded source/test edit and explains changed files and expected test cost. The human runs all builds, tests, programs, validation and state-changing Git commands. The agent does **not** run them.
2. Before I126 is complete, run only the focused tests relevant to the slice; do **not** run `make test` on intermediate partial work.
3. When the full I126 closure candidate is ready, inspect changes, run `git diff --check`, applicable targeted checks, **`make test`** and **`make check`**. Report PASS or the bounded failure output. This expensive full gate runs **once** on the complete closure candidate, not after each helper file.
4. **Only after all required tests pass**, and immediately before committing/pushing, update product `pom.xml` and root `CHANGELOG.md` for the next available version. **Never run tests after those metadata changes.** Add only patch-owned paths, verify staged diff, use the real current `origin/main` and one-line English `I126: ...` commit message. Commit/push are human actions.
5. After each human-published slice, report exact product SHA and validation. The coordinating agent updates [I126 #967](https://github.com/guillermomolina/protos/issues/967), publishes any required revision-pinned evidence in `protos-project-docs`, and chooses the next bounded slice. No issue closure until all I126 acceptance criteria and the full gate are satisfied.
6. Normative D216 #964 is already completed. D215 #961, LIB025 #962, D217 #965 and I125 #966 remain untouched and complete.

**Implementation output contract:** precise patch scope, source/tests modified, decisions reused, PASS/FAIL and skipped gates, a compact next human command block with prose before and after, and explicit unresolved blockers only when real normative ambiguity is found. Do not claim unrun validation or unpublished code.
