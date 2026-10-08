# I086 / PLAT054-3E2 — Suspendible host-entry Future.value() publication

**Date:** 2026-10-08  
**Issue:** [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
**Design:** [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
**Exact product commit:** [`guillermomolina/protos@55f06a29bd42f470a3e5c7ce07afa38737f5e76d`](https://github.com/guillermomolina/protos/commit/55f06a29bd42f470a3e5c7ce07afa38737f5e76d)  
**Preceding normative release:** `3bb1278d91ee5cea98031462be2a5c4dd3c89019` (spec 0.1.450)  
**Product version at published HEAD:** `0.3.291-SNAPSHOT` (independent intervening `0.3.290-SNAPSHOT` PERF031-F already present)  
**Status:** **PUBLISHED / OWNER-REPORTED FULL LOCAL TEST PASS**. Runtime slice complete, I086 parent **OPEN**.

## Exact publication proof and validation provenance

Read-only inspection of GitHub `guillermomolina/protos/main` verified the HEAD and the published commit message:

> I086 PLAT054-3E2: suspendible host-entry Future.value() in the Polyglot embedding

The product commit includes these **12 paths**:

- `CHANGELOG.md`
- `pom.xml`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosClosureInvoker.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosHostEntrySuspension.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosHostExecutableClosure.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosInvocation.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandardFutureProtocol.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosActorExecutionDomain.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosFutureValue.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosHostEntryFutureSuspensionTest.java`

`pom.xml` at the verified HEAD contains `0.3.291-SNAPSHOT`; `CHANGELOG.md` has the corresponding 3E2 release entry. Specification remains `0.1.450`; this product commit changes no `spec/` documents.

**Maintainer validation:** “el git diff check esta limpio. Todos los tests han pasado en local” and “pushed.” The exact commit and changed paths were independently verified via read-only GitHub. Test PASS and clean diff are **owner-reported**: this coordinator did not rerun Maven, `make test`, a static validation command, Native Image or portable-distribution gate, and did not inspect raw test logs. No benchmark or performance parity is implied.

## Implementation conformance evidence

The published `CHANGELOG.md` documents:

1. **HOST-FUT-1 fully exposed:** standard Polyglot `Value.execute()` on an extracted Protos Closure may observe pending `Future.value()` and suspend, while the Java call stays synchronous until the terminal guest result/failure.
2. **C-prime exact continuation:** a pending observation installs one ordinary Future observer atomically with pending-state inspection and yields the existing native suspension leaf; the captured continuation resumes without replay, duplicate effects or a second evaluator.
3. **Generic/native Closure coverage:** an outermost host entry with no compact target uses the Context-cached C-prime selected-call preparation path. This addresses the legitimate native `Error.handle`/generic host entry left open during initial review. Nested foreign callbacks retain their *no suspension through synchronous foreign extent* rule and signal an ordinary fresh Error on illegal suspension.
4. **RootActor and lifecycle:** the suspended Java entry retains exclusive RootActor ownership, permits Actor-local runnable producer progress, otherwise parks in a Truffle-interruptible wait, and never resumes guest code after Process termination/Context close even if termination itself terminalizes the Future. Observer cleanup and fatal unhandled Error propagation remain required.
5. **PAY AS YOU GROW:** non-suspending calls allocate no mandatory Task, RootTask, waiter, continuation or scheduler. The release entry describes one domain extent swap per host entry plus one result type check. Rich generic/native execution is on demand, not imposed on compact source calls.
6. **Tests:** `ProtosHostEntryFutureSuspensionTest` covers terminal/pending/failed/cancelled Future, exact no-replay continuation, concurrency races, Error handling/ensure/ReturnHome, source and generic/native calls, foreign callbacks, RootActor exclusivity, Context close/termination and Context independence. Existing embedding/future/Actor tests are retained.

This is a publication description and owner-reported validation record, not independent execution proof of the implementation.

## Pending work and sequencing after 3E2

**I086/#840 remains OPEN**, notwithstanding completed slices 3A/3B/3C/3D/3E1/3E2:

- **B011 Filesystem remains BLOCKED:** spec `0.1.450` establishes effective Polyglot file permission and Context working-directory confinement but expressly does **not** decide whether an unrepresentable/unconfined authorized base aborts initial bootstrap or omits the bootstrap-local Filesystem slot. This is one substantive guest-visible owner choice; do not silently decide or implement positive host authority on an unapproved default. Future 3E3 must respect the decision gate.
- **B012 Network remains READY FOR IMPLEMENTATION:** spec `0.1.450` already ratifies socket-grant mapping, host-bounded TCP capability and independent guest thread policy. Current `ProtosEmbeddedProcess.bootstrap()` still supplies `null` for default Network. `ProtosNioNetworkHost` creates a Java `ProtosNioHostIoPoller` thread; integration needs Context-local ownership, proper close/revoke, effective socket-permission checks, and no guest code from host-only polling threads. No current positive embedded Network capability or Native validation was claimed.
- **Recommended next slice: `PLAT054-3E4` / `TYPE=IMPLEMENTATION` / `REPO=guillermomolina/protos`**. Execute B012 Network grant+backend as one coherent bundle, independently of the blocked FS work. The numerical 3E3 Filesystem slice is deferred, not skipped or closed.
- **I087/#841:** independent host-supplied application module resolution for Actor.spawn; no change here.
- **PERF032/#831:** comparative graph/latency parity separately unproven. Final Native Image and portable acceptance are unverified.

No new cost-only Issue is needed.

```text
ISSUE=I086/#840
SLICE=PLAT054-3E2
TYPE=IMPLEMENTATION
PRODUCT_REVISION=55f06a29bd42f470a3e5c7ce07afa38737f5e76d
PRODUCT_VERSION=0.3.291-SNAPSHOT
SPEC_REVISION=0.1.450
PRODUCT_MAIN_PUBLICATION=VERIFIED
VALIDATION=OWNER_REPORTED_ALL_LOCAL_TESTS_PASS
DIFF_CHECK=OWNER_REPORTED_CLEAN
HOST_FUTURE_VALUE=IMPLEMENTED_AND_PUBLISHED
SOURCE_AND_NATIVE_GENERIC=IMPLEMENTED_IN_PRODUCT
PAY_AS_YOU_GROW=GUARDED_BY_PRODUCT_TESTS
B011=BLOCKED_SINGLE_UNSELECTED_BOOTSTRAP_FAILURE_POLICY
B012=READY_NETWORK_BACKEND_NOT_IMPLEMENTED
NEXT_SLICE=PLAT054-3E4
NATIVE_IMAGE=UNVERIFIED
BENCHMARK_PARITY=UNVERIFIED
PARENT_STATUS=OPEN
```

**AI assistance:** this coordination/evidence document was drafted with ChatGPT, using the live GitHub commit and the project owner's explicit publication/testing report. It does not independently attest to executable test results or new design approval.
