# I086 / PLAT054-3D — Embedded Actor carriers and Context close publication

**State:** Slice D PUBLISHED; owning [I086/#840](https://github.com/guillermomolina/protos/issues/840) remains **OPEN**.  
**Date:** 2026-10-08  
**Product revision (exact):** [`e0bb880584908f65cb78e66896eba334f3d5e13b`](https://github.com/guillermomolina/protos/commit/e0bb880584908f65cb78e66896eba334f3d5e13b)  
**Product commit subject:** `I086 PLAT054-3D: Actor carriers and Context close for the standard Polyglot embedding`  
**Product version:** `0.3.289-SNAPSHOT`. **Normative specification:** `0.1.449` unchanged.  
**Parent design:** [PLAT054/#838](https://github.com/guillermomolina/protos/issues/838). **Independent application-module child:** [I087/#841](https://github.com/guillermomolina/protos/issues/841). **Separate performance owner:** [PERF032/#831](https://github.com/guillermomolina/protos/issues/831).

## Published commit and exact ownership

GitHub's current `main` was inspected and its HEAD is `e0bb880584908f65cb78e66896eba334f3d5e13b`, following the unrelated PERF031-B `b7f7175c` and PERF031-C `a463429f` product commits. The I086-D commit changes **exactly eight paths**:

1. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedCarrierPool.java` (new).
2. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`.
3. `src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java`.
4. `src/main/java/com/guillermomolina/protos/execution/ProtosLanguageContext.java`.
5. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedCarrierPoolTest.java` (new).
6. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddingLifecycleTest.java` (new).
7. `pom.xml` (product version `0.3.289-SNAPSHOT`).
8. `CHANGELOG.md` (matching `0.3.289-SNAPSHOT` slice D entry).

The product release metadata is already part of the user's published commit. The project-record publication does not mutate product source, versions, or tests.

## Actual implementation and reviewable conformance surfaces

- The standard Polyglot embedding now uses the **Context-local, lazy** `ProtosEmbeddedCarrierPool` as the `Executor` substrate of the existing `ProtosActorScheduler`. Production host `ProtosPolyglotRuntimeHost` and the generic Actor scheduler are unchanged.
- The pool registers and starts real Truffle-created carrier `Thread` objects under its own start/close synchronization; closure prevents a carrier starting after the closing snapshot and joins real threads without relying on `ThreadPoolExecutor.awaitTermination` bookkeeping. A carrier waiting for a command enters Truffle's blocked-thread interruptible/safepoint machinery so Context cancellation and thread-local actions can reach it. An interrupt alone does not mean the pool is closed.
- Rejected/carrier-start failures use the executor rejection boundary. A fatal `Error` escaping `ProtosActorScheduler.workerLoop` is **not** silently recovered: that worker may leave scheduler `activeWorkers` accounting and an Actor turn incomplete; the pool can still start another carrier for its other accepted work, but the failed scheduler unit is not declared repaired. This is an explicitly documented exceptional runtime limit.
- `ProtosEmbeddedProcess` requests Process termination; ordinary close retains its terminal coordination, while cancelling/exiting Contexts avoid blocking on Process terminalization. `finalizeContext` performs lifecycle closure, and idempotent `disposeContext` handles cancellation paths that skip finalization, without running guest work from disposal.
- The host's thread permission applies to Actor carriers. A refused `Actor.spawn` does not synchronously fail after its creation cutover; the resulting Actor incarnation terminates and a subsequently unaccepted request observes an ordinary failed Future.
- Tests include public `Context.eval` and `getBindings` embedding paths, real Actor thread-permission cases, no Actor substrate for no-Actor programs (PAY AS YOU GROW), stopped real carrier threads after close, cancellable busy host entry, idle safepoint action and continuing Actor work, retained Value denial after close and independent Contexts. The separate `ProtosEmbeddedCarrierPoolTest` covers start-versus-close, refusal, replacement, interruptions and self-close cases.

## Validation and provenance

- **Owner/human executor reported:** all local tests **PASS**, `git diff --check` **CLEAN**, `I086 PLAT054-3D` commit **pushed**.
- The coordinator verified GitHub's exact product commit, changed paths, POM/CHANGELOG version and relevant source/test content.
- The coordinator **did not run the tests**, did not inspect raw full-suite logs, and did not independently certify `dist/build_native.py`, `dist/validate_native.py` or the portable-JAR validation on this revision. **Native Image remains UNVERIFIED** for the packaged Core and latest carrier-pool implementation; no Native PASS is claimed.

## Remaining I086 acceptance and dependencies

1. **Filesystem authority projection is not implemented.** The standard embedding currently supplies no default `filesystem` slot. A substantive host-grant mapping/base choice remains unresolved: Truffle `Env.isFileIOAllowed()` covers broad `IOAccess.ALL`, `allowAllAccess` and host-provided `FileSystem`, while existing direct-NIO backends must not bypass the Context. The future backend must respect host-authorized `TruffleFile` confinement; choose neither implicit unrestricted filesystem nor a public option without approval. See `B011` in the implementation-blocker ledger.
2. **Network authority projection is not implemented.** The standard embedding supplies no default `network` slot. `Env.isSocketIOAllowed()` alone does not settle the Protos capability grant contract (including the meaning of `allowAllAccess`), and the existing NIO poller creates threads outside the embedded Context's thread-creation policy. This is an independent authority/design gate, with backend/worker implementation subsequent to approval. See `B012`.
3. **Non-task Future observation in Java host-invoked Closure is not fully implemented:** the published product changelog explicitly notes that pending `Future.value()` inside an extracted Closure invoked by `Value.execute` produces an internal failure rather than suspending. An ordinary `context.eval` path can observe that Future. `spec/concurrency/FUTURES_AND_TASKS.md` states that otherwise-suspendable non-task contexts may wait without manufacturing a Task, but the host-entry compatibility/lifecycle mechanism still needs a targeted audit; do **not** silently introduce per-call RootTask or a blanket blocking workaround.
4. **I087/#841** independently owns non-`std:` application-module resolution for embedded Actor bootstrap, under native I086 parent hierarchy. The current resolver limitation must not be described as a normative permanent ban.
5. Native Image/portable-distribution validation and any remaining positive/negative host authority tests await the relevant finished implementation and **human-executed** gates. Existing local suite PASS is not evidence of a native build.
6. PERF032/#831 remains a separate, unproven performance/graph-parity question.

## Next-step classification

A bounded **investigation-only** I086/PLAT054-3E0 should attribute the host `Future.value()` suspension failure and prepare explicit, evidence-based owner-decision packets for Filesystem/Network grant mapping. It must distinguish a genuine observable decision from an internal implementation choice and leave publication/test execution to human executor on any later implementation slice. I086 remains OPEN and no unpublished design option is treated as ratified.

```text
ISSUE=I086/#840
SLICE=PLAT054-3D
PRODUCT_REVISION=e0bb880584908f65cb78e66896eba334f3d5e13b
PRODUCT_VERSION=0.3.289-SNAPSHOT
NORMATIVE_SPEC=0.1.449_UNCHANGED
PRODUCT_PUBLICATION=CONFIRMED
GIT_DIFF_CHECK=HUMAN_REPORTED_CLEAN
LOCAL_TESTS=HUMAN_REPORTED_PASS
NATIVE_IMAGE=UNVERIFIED
PORTABLE_DISTRIBUTION=NOT_INDEPENDENTLY_VERIFIED
ACTOR_THREAD_POLICY=IMPLEMENTED
CONTEXT_CLOSE=IMPLEMENTED_WITH_TESTS
HOST_FILESYSTEM=OPEN_B011
HOST_NETWORK=OPEN_B012
HOST_FUTURE_VALUE_SUSPENSION=OPEN_RESIDUAL
APPLICATION_ACTOR_MODULES=I087/#841_OPEN
I086_READY_FOR_CLOSURE=NO
PERF032_PARITY=NOT_CLAIMED
```
