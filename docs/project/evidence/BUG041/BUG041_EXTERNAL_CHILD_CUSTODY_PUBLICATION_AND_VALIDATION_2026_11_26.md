# BUG041 — External child custody completion ordering: publication and local validation

Date: 2026-11-26

- Owning issue: [BUG041 #1183](https://github.com/guillermomolina/protos/issues/1183).
- Related: [I166 #1111](https://github.com/guillermomolina/protos/issues/1111), [D229 #1008](https://github.com/guillermomolina/protos/issues/1008).
- Blocks acceptance of [GITHUB026 #1176](https://github.com/guillermomolina/protos/issues/1176) and [PR #1182](https://github.com/guillermomolina/protos/pull/1182).
- PROTOS_REVISION: [28d3c1df6531e31fd795ed9673b31e18bb13fc82](https://github.com/guillermomolina/protos/commit/28d3c1df6531e31fd795ed9673b31e18bb13fc82), verified on main; 0.4.224-SNAPSHOT.
- Commit: BUG041: Correct external child custody completion ordering in host close reclamation.

## Historical failure

Manual CI [attempt 1, Java shard 1](https://github.com/guillermomolina/protos/actions/runs/45760355890/job/139709118271) failed on b1c9413f62453be059c5f21f35cc5d38259db691; [attempt 2](https://github.com/guillermomolina/protos/actions/runs/45760355890/job/139712560170) passed on the same candidate. [PR CI shard 1](https://github.com/guillermomolina/protos/actions/runs/45762242458/job/139715079236) failed on cd0352e59254319a7d186ad700bcab7e3783c8de. Both failing lanes reported 1,596 tests, zero assertion failures, one error and six skipped. The failing method was ProtosI166ExternalExecutionLaunchTest.terminateIsARequestAndRepeatedWaitsObserveTheSameStatus, during session close: Polyglot Process Context failed to reach clean terminal disposition, caused by external children of the Process are still in host custody.

The issue records a concrete publication race, not proof that this exact interleaving occurred in those historical CI runs.

## Published implementation

The verified commit changes exactly four paths:

- `src/main/java/com/guillermomolina/protos/runtime/io/execution/ProtosExternalChildValue.java`
- `src/test/java/com/guillermomolina/protos/runtime/ProtosI166ExternalProcessCustodyTest.java`
- `pom.xml`
- `CHANGELOG.md`

The host close path retains the dependent stages already returned by the existing termination and transport retirement callbacks and awaits them instead of raw native completion. Removes duplicate callback invocations. Callback completion includes any custody discharge, registry removal and admission return performed by that callback. A completed native termination with pending recording is not grounds for forced termination. Existing endpoint release recording, lock ordering, bounded waits, TruffleBoundary, guest wait observation, honest refusal/retry and original launch integration test are retained. No additional production classes, synchronization primitives, threads, public protocol changes, specification changes, CI changes or production timeout increases.

The source review compared completion ordering with GraalPy [PosixResources.waitpid](https://github.com/oracle/graalpython/blob/4414eeab7e306b04ee3dbfb52650e957d27a9e98/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/PosixResources.java#L498), EmulatedPosixSupport.waitpid (host boundary), and Truffle ProcessHandlers.ProcessDecorator.waitFor / waitForCopiers / removeFromActiveSubProcesses (cleanup and registry removal before returning success). These are behavioral precedents, not a claim that GraalPy implements identical asynchronous stages. The existing Protos releaseHandle and abandonNativeChildForRuntime completion-recording patterns supply the local mechanism.

## Maintainer-executed validation

- Initial focal custody and external launch tests reported green by the maintainer.
- After the two Process-level regressions were added, `mvn -q -DskipTests test-compile` completed; `mvn -Dtest=ProtosI166ExternalProcessCustodyTest test` transcript at 2026-11-26T07:10:19Z: 49 tests, 0 failures, 0 errors, 0 skipped; test time 11.41 s, total 42.953 s, BUILD SUCCESS.
- Eight BUG041 regressions cover blocked registry publication (transport and supervisor), terminal observation independent of transport, completed native exit with pending recording, pending endpoint release, exceptional transport retirement, and single-invocation Process reclamation waiting for each recording callback.
- Negative control: only the production source was temporarily replaced with its HEAD version while retaining the new tests; five selected regressions compiled and ran. At 2026-11-26T07:12:49Z, all five failed by the intended assertions, 0 errors, 0 skipped, test time 1.344 s, total 32.203 s. The transcript records BASELINE_MAVEN_RETURN_CODE=1 and CORRECTED_SOURCE_RESTORED=YES. This is a successful negative-control experiment, not a failed candidate validation.

| Regression | Original implementation | Corrected implementation |
| --- | --- | --- |
| bug041TransportReclamationWaitsForCustodyPublication | FAIL: success while registry removal blocked | PASS |
| bug041SupervisorReclamationWaitsForCustodyPublication | FAIL: success while reap recording blocked | PASS |
| bug041RecordedReapPendingAfterNativeExitDoesNotForceTermination | FAIL: expected false, observed true | PASS |
| bug041ProcessReclamationWaitsForRecordedTermination | FAIL: result published before waiting in awaitReap | PASS |
| bug041ProcessReclamationWaitsForRecordedTransportRetirement | FAIL: result published before waiting in awaitTransport | PASS |

- Maintainer subsequently confirms publication, clean git diff --check, and all local tests passing after the integrated validation handoff. No separate full-suite count, invocation transcript, publication-validator result or Native Image run identity was provided; do not invent them.
- The local logs and exact published source establish a reproducible synchronization defect and the correction against deterministic tests. They do not establish that the historical CI failures used that exact interleaving.

## Remaining remote acceptance

At recording time, no successful representative remote concurrent shard 1 or complete mandatory CI run containing the correction has been verified. The commit workflow lookup returned no PR-triggered runs; combined commit status returned no statuses. These observations do not establish absence of push-triggered runs or check runs.

BUG041 remains open pending its explicit remote acceptance criterion. GITHUB026 remains open and blocked for acceptance until the correction is included in the distributed CI candidate and applicable automatic PR/push validation succeeds. Do not rerun an old uncorrected candidate as evidence for this fix, weaken the gate, increase production timeouts, or merge the PR solely on local success.

DURABLE_RECORD_DECISION=REQUIRED (maintainer-requested, includes local negative-control evidence not recoverable from product history alone).
