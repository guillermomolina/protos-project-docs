# I166 — External OS execution implementation publication and closure evidence (2026-11-22)

**Record type:** Revision-bound implementation/validation evidence. Non-normative: observable semantics are owned by the published Protos specification, not this record.
**Issue:** [I166 #1111](https://github.com/guillermomolina/protos/issues/1111).
**Design authority:** [D229-B1 ratified decision](../../decisions/language/D229_B1_EXTERNAL_OS_EXECUTION_CAPABILITY_RATIFIED.md); [D229 #1008](https://github.com/guillermomolina/protos/issues/1008).

## Verified product publication

- **Exact implementation commit:** [`guillermomolina/protos@1fa0c41728792596a1e4862f75298a2532b56bb1`](https://github.com/guillermomolina/protos/commit/1fa0c41728792596a1e4862f75298a2532b56bb1). GitHub commit and its changed-file list were read.
- **Subject:** `I166: Implement explicit external OS execution with direct child launch, bounded byte endpoints and POSIX process custody`.
- **Implementation series:** `0.4.191-SNAPSHOT` (published `CHANGELOG.md`). The implementation commit includes the changelog, sources, tests, build/Native Image integration, example and guides.
- **Normative prerequisite:** [`guillermomolina/protos@38789e6f3a9bc43bfcf38f94f3200a2159daa51e`](https://github.com/guillermomolina/protos/commit/38789e6f3a9bc43bfcf38f94f3200a2159daa51e), global specification `0.1.505`, primary owner [`spec/io/EXTERNAL_PROCESS.md`](https://github.com/guillermomolina/protos/blob/38789e6f3a9bc43bfcf38f94f3200a2159daa51e/spec/io/EXTERNAL_PROCESS.md). [Separate normative publication evidence](I166_D229_B1_NORMATIVE_SPECIFICATION_PUBLICATION_2026_11_21.md) retains its distinct revision provenance.
- **Implementation commit scope:** 58 changed paths, including `protos/lib/core/execution/`, a Linux POSIX backend with C library/FFM binding, Core authority and custody integration, bounded NIO pipe transport, CLI/embedding/package/Actor integration, six I166 test classes, `scripts/native-test.sh`, and public documentation. No new public shell or pipeline grammar was selected.

## Implemented contract and architecture

- **Independent default-off authority:** `--allow-execution` and `protos.AllowExecution`; an initial RootActor-scoped `execution` binding, with no implied Filesystem/Network/Resolver/Entropy/Clock permissions, no automatic Actor inheritance and no crossing isolated parallel execution.
- **Protos-first API:** `execution.start(executable, arguments, environment, workingDirectory)` receives a direct executable and exact argv, a complete explicitly supplied environment, and a native working directory. It returns a Future for `ChildProcess`. Public selectors belong to Protos source-backed `Execution`, `ChildProcess`, `ChildStdin`, `ChildOutput` and `ExitStatus` protocols.
- **Host boundary:** Truffle `Env.newProcessBuilder` and a host-installed Linux `ProtosPosixProcessHandler`; no JDK process-provider fallback on provider initialization failure. Native child I/O is passed through existing Protos Byte I/O/Future mechanisms.
- **Streams:** Separate finite stdin/stdout/stderr byte endpoints, NIO pipes and transfer workers with backpressure. Child exit/reap does not imply EOF; inherited writers, including descendants, can keep stdout/stderr open.
- **Ownership:** The Protos Process retains committed children, their native handles, outstanding endpoints, supervision and transport until confirmed release/reap. Actor death does not retroactively cancel committed creation. Termination requests do not count as reap.
- **Exit observation:** The host-selected POSIX provider retains native wait status and distinguishes verified normal exits (including code 143) from real signal termination; the host-owned Context path materializes `ExitStatus` with fresh wait Futures. For externally built embedding Contexts whose provider identity Protos cannot verify, `wait()` conservatively fails with `IOError` rather than inferring a signal from an untrusted numeric code.
- **Host restrictions:** The capability does not sandbox OS child processes by means of the guest Filesystem. This release's native provider is Linux-specific. Other platforms and embedding-host provider selection have separately documented limitations.

## Validation provenance

- **Post-publication maintainer attestation (2026-11-22):** `git diff --check` clean; **all tests pass locally**. This is owner-reported validation of the published I166 change, not an independent GitHub CI run or a copied final full-suite transcript.
- **Earlier focal transcript provided in the implementation conversation, before the final publication commit:** `mvn test -Dtest='ProtosI166*'` reported **153 tests, 0 failures, 0 errors, 0 skipped; BUILD SUCCESS**. Its per-class counts were CLI 17, embedding 38, authority 13, launch 20, streams 24, custody 41. This is useful prepublication evidence but **is not claimed to be an exact-SHA final run**.
- **JVM compilation in that focal transcript:** 550 production Java sources and 785 test sources compiled. Warnings from existing DSL and restricted FFM APIs are not presented as failures.
- **Native Image:** the published `scripts/native-test.sh` contains external-execution cases for grant, streams, descendant EOF, exit categories, empty environment, no Filesystem, stderr, EPIPE/SIGPIPE host survival and native-child reclamation. No separate post-publication Native Image command/output or CI status was supplied here; **do not convert test presence or the general local-PASS attestation into independently observed Native execution evidence**. The product changelog expressly describes native child launch as not yet independently verified.
- **License:** the implementation preserves Protos APL notices and published native-library attribution/licensing and packaging; this record does not alter any license terms.

## Closure interpretation and remaining boundaries

On the maintainer's publication and all-local-tests PASS attestation, I166's accepted Linux JVM/host execution capability is published and the owning implementation Issue may be closed as completed with the validation provenance above. This record does **not** claim full independent Native Image acceptance, Windows/macOS subprocess support, a generic OS-child sandbox, external embedder provider introspection or guaranteed bounded recovery from unrecoverable native supervision failures.

Uncertain launch/reap keeps a custody obligation rather than fabricating a no-child failure; an unrecoverable supervision or stream-release failure is not guaranteed to complete in bounded time. Context-close refusal/reclamation and external embedding limits are documented in the published [authority guide](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/docs/guide/11-process-io-filesystems-and-authority.md) and [embedding guide](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/docs/guide/embedding/java.md). Do not state those mechanisms are infallible.

**Downstream:** [LIB027 #1009](https://github.com/guillermomolina/protos/issues/1009) has its I166 *implementation prerequisite* satisfied by this publication. LIB027's pipeline/composition public contract, failure precedence and related observable API decisions are not approved by D229 and remain its separate design gate; no LIB027 library implementation is included or implicitly authorized.

## Revision-bound compact evidence

```text
WORK_ITEM=I166
DESIGN=D229-B1
PROTOS_REVISION=1fa0c41728792596a1e4862f75298a2532b56bb1
IMPLEMENTATION_VERSION=0.4.191-SNAPSHOT
NORMATIVE_PROTOS_REVISION=38789e6f3a9bc43bfcf38f94f3200a2159daa51e
NORMATIVE_SPEC_REVISION=0.1.505
PRODUCT_COMMIT_VERIFIED=PASS
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
PREPUBLICATION_I166_FOCAL_TESTS=153_PASS_0_FAIL_0_ERROR_0_SKIP
FINAL_EXACT_SHA_TEST_TRANSCRIPT=NOT_PROVIDED
NATIVE_EXECUTION_INDEPENDENT_TRANSCRIPT=NOT_PROVIDED
DURABLE_RECORD_DECISION=REQUIRED
LIB027_IMPLEMENTATION_PREREQUISITE=SATISFIED
LIB027_DESIGN_GATE=UNSELECTED
```
