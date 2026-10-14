# I093-C1 — Published post-commit Actor termination custody fix (2026-10-14)

**Kind:** non-normative, immutable implementation/publication evidence. No public semantic decision, independent test execution, performance benchmark, or full I093-C acceptance is asserted.

- **Implementation issue:** [I093/#873](https://github.com/guillermomolina/protos/issues/873).
- **Audit owner:** [AUD020/#872](https://github.com/guillermomolina/protos/issues/872) (independently OPEN).
- **PROTOS_REVISION:** `8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff` ([published product commit](https://github.com/guillermomolina/protos/commit/8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff)).
- **Parent checkpoint:** I093-B2 at `f8e2d4e3dcf9b5e13d4850e14605248cae326125` ([publication evidence](./I093_B2_TCP_LISTEN_ACCEPT_HOST_DESCRIPTOR_ACTOR_COMPLETION_PUBLICATION_2026_10_14.md)).
- **Implementation version:** `0.3.356-SNAPSHOT` (`pom.xml` and `CHANGELOG.md` in the same product commit).
- **Current publication classification:** C1 **PUBLISHED**; overall I093-C acceptance **INCOMPLETE**; I093 and AUD020 **OPEN**.

## Defect: committed host resource stranded in a discarded Actor completion

At B2, `connectTcp`, `listenTcp` and `TcpListener.accept` correctly post host callbacks to the owning Actor, where guest materialization is allowed. But the following interleaving could strand already-acquired host custody:

1. The backend succeeds and crosses the I/O operation's commitment boundary, enqueuing a targeted Future completion with a live, not-yet-transferred host resource.
2. A `Future.cancelRequest()` after commitment records a request without converting the Future into `CANCELLED` or releasing host custody.
3. Actor termination clears `targetedRuntimeCompletions` without running the queued completion.
4. The subsequent termination cancellation request does not invoke the operation's registered handler again because cancellation was already recorded.
5. The completion is neither dispatched nor asked to release its untransferred resource.

The problem is lost *host resource custody*, not a license to execute guest materialization after Actor termination or to undo already-committed effects. C1 adds cleanup at the exact queue-discard boundary rather than changing the public Future/Network semantics.

## Exact published remediation

- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java`: `TargetedRuntimeCompletion` now optionally carries `onDiscard`. The original two-argument `enqueueTargetedFutureCompletionForRuntime(future, completion)` remains available and supplies a no-op discard action. Termination snapshots and clears the pending queue while synchronized, then runs the discarded entries' host-only cleanup outside the domain lock, before requesting cancellation of remaining Actor-local work. A cleanup exception must not interrupt the termination cutover.
- `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkConnectFlow.java`: a queued connect-completion passes a host-only `onDiscard` invoking its existing claim-once `AtomicReference<Runnable>` custody gate.
- `src/main/java/com/guillermomolina/protos/runtime/ProtosNetworkAcquisitionInvocation.java`: the same claim-once dropped-delivery reconciliation is shared by listen and accept.

The claim-once gate means only one of Actor dispatch, termination discard, producer cancellation handling or enqueue rejection can release the same queued host resource. No source closure, `ProtosObjectValue` construction, callback message or new Task is required for the discard path. A completion dequeued for dispatch is not in the dropped queue; ownership/dispatch remain governed by the existing Actor executor and operation commitment rules.

**Scope:** the published product commit contains exactly **8 paths**: these three runtime production files, three focal JUnit files (`ProtosNetworkConnectAcquisitionTest.java`, `ProtosNetworkListenAcquisitionTest.java`, `ProtosTcpListenerAcceptTest.java`) under `src/test/java/com/guillermomolina/protos/execution/`, and `pom.xml` plus `CHANGELOG.md`. The user-reported commit summary is **65 insertions / 7 deletions**. No `spec/` or guest Protos source is modified.

## Regression and validation provenance

All execution below was performed or reported by the **human maintainer**, not by this documentation agent.

- **C1 focal validation:** `mvn -Dtest=ProtosNetworkConnectAcquisitionTest,ProtosNetworkListenAcquisitionTest,ProtosTcpListenerAcceptTest test` was reported **29 tests, 0 failures, 0 errors, 0 skipped**.
- **Integrated suite:** maintainer reported `make test -> PASS` and `make check -> PASS` on the validated substantive C1 candidate.
- **Native gate:** maintainer reported **native PASS**; output includes Truffle's intentionally selected interpreter-only fallback warning. The existing PLAT045 policy intentionally does not promise guest runtime native-code compilation.
- **Native IP data checks:** on the Native executable, the maintainer supplied results for standard IP address conformance (**10 passed**), standard IP endpoint conformance (**8 passed**) and standard endpoint parsing/formatting (**2 passed**): **20 passed, 0 failed**. These are IP data tests, not a host-authorized end-to-end Native TCP connection/listener gate.
- **Diff check:** maintainer reported `git diff --check` clean and all locally performed tests passed.
- **Environment qualification:** the later publication shell on `~/Fuentes/protos-1` printed `bash: mvn: orden no encontrada`. That invocation **did not run Maven tests** and must not be counted as green. Prior PASS reports above are retained as human-reported validation evidence from an environment that ran them; no assertion is made that Maven was subsequently executable or rerun in the publishing shell.
- `pom.xml` and `CHANGELOG.md` were finalized after the validated substantive code candidate. The published product commit verifies both files in the same 8-path commit. **No post-metadata build or tests are claimed.**

## Architectural implications and remaining checks

C1 fixes a concrete host-custody edge in the existing B1/B2 architecture while preserving the direct NIO/guest separation. No new public authority, guest protocol, default poller or new mandatory RootTask is introduced.

**Still not established:** (1) a Native Image TCP `connect/listen/accept` operation with explicit `Network` authority; (2) an exhaustive runtime Actor/P/Context matrix for *all* host completion races; (3) actual before/after memory allocations, CPU costs or latency; (4) a completed independent AUD020 audit matrix or D172/D173 ratification change. These are **NOT_MEASURED / NOT_VERIFIED**, rather than treated as PASS by association.

I093-C must retain these limits when publishing a final architecture/conformance record. Do not close the I093 parent solely on C1.

```text
I093_A=PUBLISHED
I093_B1=PUBLISHED
I093_B2=PUBLISHED
I093_C1=PUBLISHED
I093_C_FINAL_ACCEPTANCE=PENDING
PROTOS_REVISION=8c1a54c9bfff2b09deb693eab8ad8fa11b18f0ff
IMPLEMENTATION_VERSION=0.3.356-SNAPSHOT
C1_FOCAL_TESTS=29_PASS_HUMAN_REPORTED
PRODUCT_MAKE_TEST=PASS_HUMAN_REPORTED
PRODUCT_MAKE_CHECK=PASS_HUMAN_REPORTED
NATIVE_GATE=PASS_HUMAN_REPORTED
NATIVE_IP_DATA_CONFORMANCE=20_PASS_HUMAN_REPORTED
PUBLICATION_SHELL_MAVEN=NOT_AVAILABLE
NATIVE_AUTHORIZED_TCP=NOT_VERIFIED
PERFORMANCE_COSTS=NOT_MEASURED
I093_ISSUE=OPEN
AUD020_ISSUE=OPEN
```
