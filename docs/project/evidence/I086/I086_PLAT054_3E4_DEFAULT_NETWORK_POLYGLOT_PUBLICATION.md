# I086 / PLAT054-3E4 — Default Network in standard Polyglot embedding

**Date:** 2026-10-08  
**Issue:** [I086 / #840](https://github.com/guillermomolina/protos/issues/840)  
**Architectural contract:** [PLAT054 / #838](https://github.com/guillermomolina/protos/issues/838)  
**Published product revision:** [`guillermomolina/protos@f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2`](https://github.com/guillermomolina/protos/commit/f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2)  
**Product version:** `0.3.294-SNAPSHOT`  
**Normative spec:** `0.1.450` unchanged  
**Status:** **PUBLISHED / OWNER-REPORTED ALL LOCAL TESTS PASS; B012 implementation complete pending final integration gates**

## Source verification and validation provenance

The live `guillermomolina/protos/main` branch and exact GitHub commit were checked read-only. Commit message: **`I086 PLAT054-3E4: default Network in the standard Polyglot embedding`**. Eight published paths:

1. `CHANGELOG.md`
2. `pom.xml`
3. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkCustody.java` (new Context/Process Network authority custody)
4. `src/main/java/com/guillermomolina/protos/execution/ProtosEmbeddedProcess.java`
5. `src/main/java/com/guillermomolina/protos/execution/ProtosNioHostIoPoller.java`
6. `src/main/java/com/guillermomolina/protos/execution/ProtosNioNetworkHost.java`
7. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkCustodyTest.java` (new)
8. `src/test/java/com/guillermomolina/protos/execution/ProtosEmbeddedNetworkTest.java` (new)

At published HEAD, `pom.xml` reports `0.3.294-SNAPSHOT`. Concurrent unrelated changes between 3E2 (`0.3.291-SNAPSHOT`) and 3E4 include `0.3.292-SNAPSHOT` PERF031-G and `0.3.293-SNAPSHOT` LM010-A; 3E4 owns only its listed paths.

The human maintainer explicitly reported the commit **pushed**, **all local tests passed**, and **`git diff --check` clean**. The coordinator independently verified GitHub publication/code/changelog but did **not** execute builds/tests, review raw test logs, run Native Image, or certify standalone portable acceptance. No observed comparative performance or benchmark parity is asserted.

## Ratified HOST-NET-1 / HOST-NET-2 implementation evidence

**Effective socket authority, not ambient authority.** `ProtosEmbeddedProcess.bootstrap()` now uses the actual `TruffleLanguage.Env.isSocketIOAllowed()` to determine whether initial module bootstrap receives a concrete `ProtosNetworkCapabilityValue`; otherwise its `network` slot is absent. File permission and guest-thread creation permission are deliberately irrelevant to socket entitlement. Additional host evaluations receive no implicit network authority, as required by `PROCESS_IO.md`.

**Lazy host-only TCP backend.** `ProtosEmbeddedNetworkCustody` is the opaque backend/authority target. Its construction provisions Network without opening a selector, poller, socket, or guest worker. The first `connectTcp` / `listenTcp` materializes a single `ProtosNioNetworkHost`, using its existing NIO backend. The NIO poller is a host Java thread and does not execute guest code or enter Truffle Context. A monitor serializes first-use and close admission to prevent starting a poller after close. No per-call Task, poller or scheduler is charged for unused networking.

**Process/Context termination custody.** The new custody closes idempotently, rejecting later acquisitions and retiring its poller and associated connection/listener/in-flight channel resources. The code and release record cover terminalization, Context finalization/disposal, races and cancellation. No guest callback is allowed after the terminal boundary.

**Human-reported test coverage.** The new `ProtosEmbeddedNetworkTest` and `ProtosEmbeddedNetworkCustodyTest` cover socket/file/thread permission combinations; loopback TCP echo/half-close/listen/accept; connection errors and cancellation; concurrent and independent Contexts; close/cancelling close/fatal termination; on-demand activation, failed backend startup and first-use-versus-close races; and ordinary `Future.value()` suspension via PLAT054-3E2. These coverage claims reflect the published release and test code; test PASS comes from the maintainer's report.

## Completion and remaining responsibility

- **HOST-NET-1 / HOST-NET-2:** **IMPLEMENTED/PUBLISHED**, no unratified host permission or thread model added.
- **B012:** Normative authorization and backend implementation are satisfied. Registry may classify **IMPLEMENTED — PRODUCT RELEASE VERIFIED; FINAL NATIVE/PORTABLE ACCEPTANCE UNVERIFIED**, instead of retaining the obsolete READY/not-implemented status. This does not close parent I086 by itself.
- **B011:** Still **BLOCKED — SINGLE UNRATIFIED POLICY CHOICE**. Specification `0.1.450` fixes effective Context file permission, configured provider confinement and base=effective authorized Context working directory, but intentionally leaves the fail-closed consequence for an unrepresentable/unconfined base open: **abort initial bootstrap** or **complete without `filesystem`**. Do not infer that choosing one has already been approved. `ProtosEmbeddedProcess.bootstrap()` still passes `null` for Filesystem, so positive embedded filesystem capability remains unimplemented.
- **Next slice:** **PLAT054-3E3-FS0**, narrow **TYPE=INVESTIGATION/owner decision packet**, not runtime implementation. Resolve only B011's base-failure-versus-absent-slot policy with GITHUB010/owner exact candidate approval. Then publish the corresponding normative change before attempting 3E3 implementation. This is a cost-only substage of PLAT054/I086, not grounds to create a formal issue.
- **I087/#841:** separate host-provided app module resolver remains unaffected.
- **PERF032/#831:** graph and latency parity separate, not demonstrated by 3E4.
- **Native Image and final portable distribution:** unverified; no parent closure.

```text
ISSUE=I086/#840
SLICE=PLAT054-3E4
PRODUCT_REVISION=f8f4ebe2e2d5ad903562518a39c08d3a66c8e0c2
PRODUCT_VERSION=0.3.294-SNAPSHOT
NORMATIVE_SPEC_REVISION=0.1.450
PRODUCT_MAIN_PUBLICATION=VERIFIED
CHANGED_PATHS=8
VALIDATION=OWNER_REPORTED_ALL_LOCAL_TESTS_PASS_AND_DIFF_CHECK_CLEAN
NETWORK_GRANT=IMPLEMENTED
NETWORK_CUSTODY=LAZY_CONTEXT_PROCESS_OWNED
NETWORK_THREAD_POLICY=HOST_ONLY_POLLER_GUEST_THREAD_INDEPENDENT
B012=IMPLEMENTED_PUBLISHED_FINAL_PORTABLE_NATIVE_UNVERIFIED
B011=BLOCKED_UNRATIFIED_FAIL_CLOSED_CONSEQUENCE
NEXT_SLICE=PLAT054-3E3-FS0
NEXT_TYPE=INVESTIGATION
I087=INDEPENDENT
NATIVE_IMAGE=UNVERIFIED
PARENT_STATUS=OPEN
```

**Assistance disclosure:** This record was drafted with ChatGPT from read-only GitHub source/changelog inspection and the maintainer's explicit report. It is durable non-normative evidence; it is not a new owner approval for B011.
