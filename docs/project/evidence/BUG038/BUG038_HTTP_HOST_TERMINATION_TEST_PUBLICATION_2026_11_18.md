# BUG038 — HTTP Host termination/EOF test synchronization, publication and acceptance (2026-11-18)

**Non-normative, revision-bound project evidence.** The change corrects a test's observation order; it does not relax HTTP resource ownership, increase socket timeouts, modify production runtime code or prove every possible concurrency schedule.

## Publication identities

- Owning Issue: [BUG038 / protos#1132](https://github.com/guillermomolina/protos/issues/1132).
- Exact published product commit: [`guillermomolina/protos@91b92332f6870fd9f0903c829c2728a644d0dca4`](https://github.com/guillermomolina/protos/commit/91b92332f6870fd9f0903c829c2728a644d0dca4).
- Commit subject: `BUG038: Synchronize HTTP hosting failure and abort tests with termination before requiring connection EOF`.
- Version: **0.4.158-SNAPSHOT**.
- Exact three changed product paths: `src/test/java/com/guillermomolina/protos/execution/integration/http/ProtosHttpHostFactoryIntegrationTest.java`, `pom.xml` and `CHANGELOG.md`. **No production Java or Protos HTTP implementation change.**
- Prior published Test Tool/callback revision: [BUG037 `afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb`](https://github.com/guillermomolina/protos/commit/afa18073e0fd48804c24b66c2d8d3e26aaa5cbcb), separately tracked.

## Original failure and causal observation

During an integrated `make test` Java-parallel phase configured with **six JUnit class workers**, `ProtosHttpHostFactoryIntegrationTest.rootActorFailureTerminatesTheProcessAndRetiresEveryWorker` failed at `readAll` with `java.net.SocketTimeoutException: Read timed out` (`ProtosHttpHostFactoryIntegrationTest.java:325 -> readAll:369`). The run reported **6004 tests; 0 assertion failures, 1 error, 11 skipped**, so `make test` failed before the serial Java and Protos guest phases. The changed suite was otherwise not shown to be faulty by that run.

Before the fix, the test sent `GET /fail` and immediately demanded connection EOF. The guest handler performs `service.send("fail")` asynchronously; the RootActor's `fail` turn writes `failing` and then signals an unhandled Error, which terminates the Process. The old socket deadline therefore also measured dispatch and termination latency before cleanup was known to have occurred.

The fixed `/fail` test retains the client socket and orders its observations as follows:

1. Send the request while the Host and Process are live.
2. Wait for the RootActor's `failing` output.
3. Await Process termination through the existing bounded `Entry` helper.
4. Require `session.isTerminated()` and `session.failedFromRoot()`.
5. **Only then** require the still-open client connection to reach EOF through the original `readAll(socket)`.

The paired `/abort` test similarly awaits the existing `host abort` completion event (emitted after `host.shutdown("abort").value()`) before requiring EOF, while asserting that the RootActor Process remains alive. The separate `/drain` test is unchanged because drain waits for an admitted session to finish before its completion event.

The five-second socket read timeout, class `@Timeout`, `readAll`, `RootService.connect`, `RootService.close`, `Entry.await` and output-line waiter are unchanged. No time-based retry, added sleep, catch-and-ignore of timeout, test exclusion, lower JUnit parallelism or new host/runtime infrastructure was introduced.

Relevant normative owner: [`spec/io/HTTP.md` §§15.6–15.10](https://github.com/guillermomolina/protos/blob/91b92332f6870fd9f0903c829c2728a644d0dca4/spec/io/HTTP.md); the existing connection custody invariant remains mandatory. Actor fatality and Process termination remain governed by `spec/concurrency/ACTORS.md`.

## Maintainer-provided validation

| Check | Result | Provenance |
| --- | --- | --- |
| Exact failing method before repair, isolated | **1/1 PASS** | Established that a single isolated run did not reproduce the parallel timeout |
| Entire HTTP factory class before repair, isolated | **16/16 PASS** | Did not erase integrated failure |
| Entire HTTP factory class after repair | **16/16 PASS**, exit 0, `BUILD SUCCESS` | `mvn '-Dtest=ProtosHttpHostFactoryIntegrationTest' test`; `/tmp/protos-bug038-focal-Mlwzne9h.log` |
| Selected concurrent JUnit classes after repair, six workers | **73/73 PASS**, exit 0, `BUILD SUCCESS` | 16 HTTP factory + 19 HTTP host + 21 RootActor bootstrap + 17 parallel-execution tests; `/tmp/protos-bug038-parallel-uWWbknxW.log` |
| `git diff --check` | **PASS** | Maintainer terminal output after test-only change |
| Final local tests | **All local tests PASS, reported by maintainer after push** | Owner summary; exact final `make test` and `make check` transcript, counts and post-commit test SHA not separately provided |

This evidence supports the revised test synchronization under representative related concurrency. It does not prove the absence of all potential HTTP Host races or attribute every historical shutdown problem to a test-ordering error. [BUG028 / #979](https://github.com/guillermomolina/protos/issues/979) is an earlier, separately closed HTTP shutdown investigation and is not rewritten by this correction.

## Acceptance and limits

- Published product revision and changelog/version are independently identifiable.
- The target test still fails if Process termination completes but the admitted connection remains open, preserving the resource-ownership assertion.
- No claim is made that an unprovided exact-post-push full test transcript was inspected.
- This record is not a new release or normative policy decision. Live closure belongs to the owning GitHub Issue.

**Evidence source:** maintainer-supplied focal and six-worker JUnit outputs, owner-reported final green local tests and clean diff, and read-only inspection of the exact published Protos commit. No tests were executed while preparing this record.
