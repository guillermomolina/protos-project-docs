# I184 — TCP abort normative and runtime publication

Date: 2026-11-27. Non-normative publication evidence.

- [I184 #1204](https://github.com/guillermomolina/protos/issues/1204)
- [D270 #1201](https://github.com/guillermomolina/protos/issues/1201)
- [D270-D and D270-D1-B selected decision](../../decisions/language/D270_D_EXPLICIT_TCP_TRANSPORT_ABORT_SELECTED_2026_11_26.md)
- [D1-B explicit approval and pinned Truffle precedents](../D270/D270_D1_B_TRUFFLE_PRECEDENTS_AND_OWNER_APPROVAL_2026_11_26.md)

## Product publication

PROTOS_REVISION=52591e7ce5c7e068f98101db94e77e68a64320da, verified main HEAD at inspection. Parent facde992e6f6d9f11ae1a3ed64e9140a70976e50.
[Commit](https://github.com/guillermomolina/protos/commit/52591e7ce5c7e068f98101db94e77e68a64320da): I184: add explicit TCP transport abort with lifecycle conformance and Native Image coverage.
38 changed paths; 1606 insertions, 115 deletions according to GitHub commit metadata. All file patches available; no added trailing whitespace detected in those patches. This is not an independent local git diff --check execution.

- Implementation [pom.xml](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/pom.xml) and [CHANGELOG.md](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/CHANGELOG.md): 0.4.232-SNAPSHOT.
- Normative [PROTOS_SPEC_CHANGELOG.md](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/PROTOS_SPEC_CHANGELOG.md): 0.1.518, 2026-11-27.
- Primary normative owner [NETWORK.md §9.1](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/NETWORK.md); affected [IO_CORE](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/IO_CORE.md), [BYTE_IO](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/BYTE_IO.md), [PROCESS_IO](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/PROCESS_IO.md), [TLS](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/TLS.md), [HTTP](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/spec/io/HTTP.md).
- [Source-backed TcpConnection.abort](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/protos/lib/core/network/TcpConnection.protos); private native bridge, existing IoLifecycle, IoOperation, ByteIoFlow, TcpConnectionFlow/Value, NIO backend and shared poller. No new Java production class in the commit.
- [Networking guide](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/docs/guide/library/networking.md) and [authority guide](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/docs/guide/11-process-io-filesystems-and-authority.md).

## Selected invariant reconciliation at the published normative boundary

Both exact candidates were approved explicitly in the active interaction: D270-D (“ok aprubeo la recomendación”) and D270-D1-B (“apruebo D270-D1-B.”). The published NETWORK §9.1 states the selected operation, not a generic Abortable protocol: authentic owning Actor TCP capability only; irreversible cutover; no new output attempt after cutover, even for committed writes; reconcile actual effects; preserve complete successful final admitted attempt and independently established results/failures. Existing ordinary close is distinct; followers use fresh Futures and recorded Error identity. Required operation and half-close aftermath and safe physical release both precede success. No implicit cancellation/termination abort, automatic deadline, RST promise, transfer authority or hard realtime. Uncommitted read consumes zero logical bytes; incomplete committed write preserves permitted prefix and fails. TLS/HTTP composition retains authenticated closure and existing ownership and cooperative Host semantics. Post-abort write admission precedes snapshot and capacity reservation per implementation changelog. This is the published normative delta, not certification of every possible runtime interleaving or full secondary-owner audit. No further owner selection inferred.

## Validation provenance

Maintainer explicitly reports in the current interaction: “I184: add explicit TCP transport abort with lifecycle conformance and Native Image coverage pushed. el git diff check esta limpio. todos los tests pasan en local.” Final local validation is maintainer-reported, not independently executed; exact full-suite command and final counts not supplied.

Earlier pasted human-run transcript (Texto pegado(20261127-024121).txt), on pre-metadata 0.4.231-SNAPSHOT candidate:
- git diff --check returned without output.
- Maven targeted ProtosTcpConnectionDuplexLifecycleTest 28, ProtosTerminationCleanupCloseTest 9, ProtosIoLifecycleTest 16; total 53, zero failures/errors/skips, BUILD SUCCESS.
- make build-native returned BUILD_NATIVE_STATUS=0.
- Native executable test files client-handshake 24/24, client-lifecycle 23/23, host-connection 12/12; 59 passed, zero failed.
- scripts/native-test.sh reported NATIVE_REGRESSION_SUITE=PASS.
- Standalone real loopback printed I184_TCP_ABORT_LOOPBACK=PASS from target/native/protos, with explicit Network grant.
- This is built executable evidence, not proof of installed distribution invocation. No ARM target.

Exact product source test coverage inspected:
- [Duplex lifecycle](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosTcpConnectionDuplexLifecycleTest.java): fresh followers, close escalation, failure identity, cancellation, read/write/half-close aftermath, capacity and post-abort rejection.
- [NIO write](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosNioTcpConnectionWriteTest.java): finalAdmittedAttemptPreservesCompleteSuccessAfterAbort, zero and positive first effect, incomplete committed prefix, independent failure and no duplicate success.
- [NIO lifecycle](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosNioTcpConnectionLifecycleTest.java): nonreading peer, pending half-close, no early physical success.
- [Poller](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/java/com/guillermomolina/protos/execution/integration/filesystem/ProtosNioHostIoPollerTest.java): termination/fallback, duplicate physical requests, persistent failure, late key, independent selector failure.
- [Termination cleanup](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosTerminationCleanupCloseTest.java): authorized cleanup, rejection outside authorization, committed backend custody.
- [Public fixture](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/protos/tests/library/network/tcp-abort.protos), [real loopback](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/src/test/resources/i184/tcp-abort-loopback.protos), [Native gate](https://github.com/guillermomolina/protos/blob/52591e7ce5c7e068f98101db94e77e68a64320da/scripts/native-test.sh). The loopback exercises distinct retirement followers, pending read failure, post-cutover rejection, unchanged endpoints/slots, independent connection and cleanup. It does not assert EOF versus RST or claim a stalled-write timing bound. The stalled-write test is JVM coverage.
- TLS/HTTP composition fixtures and repository completeness classification included in publication.

## Coordination checkpoint

Product source, normative publication and changelogs verified at exact PROTOS_REVISION. The earlier design/implementation notes describing no product changes are historical. The normative publication is no longer a blocker. Both issues may record implementation publication and local tests green. Do not equate this evidence with full formal closure: the installed-distribution acceptance is not independently established by the available transcript or explicitly identified in the final report; the previously outstanding complete secondary-owner audit and per-cell comparative confidence record are not certified in this checkpoint; live GitHub Project and native dependency verification remain unavailable. Keep open in review rather than claim completion of unverified gates. No new work identifier required. D269 and WEB016 policy remain separate; no automatic HTTP abort or Timing implementation claimed.
