# BUG028 — HTTP Host shutdown race repair and closure evidence

- **Work item:** [BUG028 #979](https://github.com/guillermomolina/protos/issues/979), independent correctness/regression repair.
- **Product repository:** `guillermomolina/protos`.
- **Published product revision:** [`1c719a50daf66fbd74e7a8b4031cf98636c9e8e9`](https://github.com/guillermomolina/protos/commit/1c719a50daf66fbd74e7a8b4031cf98636c9e8e9).
- **Implementation version:** `0.3.412-SNAPSHOT`.
- **Evidence date:** 2026-10-28.
- **Validation provenance:** maintainer/Human Executor reports; the recording agent did not execute builds or tests.

## Defect and repair

The post-I126 HTTP Host integrated tests exposed two independent shutdown failures:

1. The listener-refusal probe sometimes received a TCP connection reset around the drain cutover and treated it as an unconditional test failure. A reset alone did not establish listener refusal or HTTP admission.
2. When a Worker handler invoked `Actor.current().stop()`, the Worker's own admitted `/stop` connection could remain open after termination even while a separate parked session had already been cleaned up. The owning close needed to execute within the narrowly authorized D121 cancellation-cleanup extent when cancellation was requested but not yet observed.

The published fix retains the first service-entry cancellation observation and the owning/borrowed distinction. In `std:http/internal/Session`, an owning session now invokes `enterService()` inside a nested `ensure` immediately before its mandatory `duplex.close()`; if this observes pending service-Task cancellation, the close executes inside that `ensure`'s cancellation cleanup. The close remains the service Task's one mandatory close, with the existing cleanup/Error precedence. The Java `ProtosHttpAsyncFacility` change documents the two observation boundaries; it does not create new cancellation powers or widen lifecycle-close authority.

The Host integration fixture now establishes drain cutover before probing. A genuine refused connection succeeds; a transient reset or connected-but-unserved probe is retried; an HTTP response after cutover, unexpected I/O error, or connection left open until timeout fails. Additional real-socket regressions cover Worker self-stop from a ready session and cancellation encountered on an attempted body read, alongside the retained already-suspended-I/O case. Guest HTTP custody tests cover mandatory-close failure precedence, deferred-close settlement before session terminality, and release of an unadmitted connection without a write.

D121 tests also cover ordinary pre-cutover close and a close entered from nested termination cleanup. The delegated-Task negative case permits either an explicit `IOLifecycleError` rejection **or cancellation before the attempted close**; it asserts that the unrelated connection was not closed. The cancellation outcome alone is not represented as proof of an explicit lifecycle-error rejection.

No change to the public HTTP, Actor, Future, Task, or I/O specification was made. The existing D121/D216 authorities remain controlling.

## Published product delta

The product revision changes exactly these seven paths:

1. `protos/lib/http/internal/Session.protos`
2. `protos/tests/library/http/host-connection.protos`
3. `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java`
4. `src/test/java/com/guillermomolina/protos/execution/integration/http/ProtosHttpHostIntegrationTest.java`
5. `src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosTerminationCleanupCloseTest.java`
6. `pom.xml`
7. `CHANGELOG.md`

`Makefile` is **not** part of this BUG028 product commit; the diagnostic I130-A serial-lane change was not incorporated into the BUG028 delta. The commit contains no TLS source change.

## Human-reported validation

| Check | Result | Evidence source |
| --- | --- | --- |
| Focal BUG028 regression tests | **PASS** | Maintainer report in the implementation discussion; no run count or raw log attached to this record. |
| `git diff --check` | **PASS** (clean) | Maintainer's 2026-10-28 publication report. |
| `make test` | **PASS** (local) | Maintainer's 2026-10-28 publication report. |
| `make check` | **PASS** (local) | Maintainer's 2026-10-28 publication report. |
| Product commit/push | **PUBLISHED** | Exact GitHub commit linked above; `pom.xml` and `CHANGELOG.md` on the repository default branch show `0.3.412-SNAPSHOT`. |

These are reported local PASS results, not an assertion that the evidence-recording agent ran the commands or independently reproduced the intermittent failures. This record does not claim a separate CI workflow result, stress repetition count, or unreported platform validation.

## Coordination and boundaries

- BUG028 is independent of the completed [I126](https://github.com/guillermomolina/protos/commit/4f45348c39f218fd18503ef52a40bd20b71b8cf7) publication and the previously closed duplicate-tracking [BUG027 #971](https://github.com/guillermomolina/protos/issues/971); those historical closures are not rewritten.
- [I130 #977](https://github.com/guillermomolina/protos/issues/977) previously identified these Host failures as blocking its TLS slice's integrated gate. BUG028's product publication and reported full-suite PASS remove that **independent HTTP defect** as the known obstruction. They do **not** validate or publish I130-A TLS source, settle D220 implementation acceptance, or close I130.
- Issue status, ownership, priority, dependencies, and Project projection remain live GitHub coordination state, not authority carried by this project record.

## Closure identities

```text
WORK_ITEM=BUG028
ISSUE=guillermomolina/protos#979
PROTOS_REVISION=1c719a50daf66fbd74e7a8b4031cf98636c9e8e9
IMPLEMENTATION_VERSION=0.3.412-SNAPSHOT
HUMAN_GIT_DIFF_CHECK=PASS
HUMAN_FOCAL_TESTS=PASS
HUMAN_MAKE_TEST=PASS
HUMAN_MAKE_CHECK=PASS
DURABLE_RECORD_DECISION=REQUIRED_BY_OWNER_REQUEST
```

The exact project-record publication revision is identified by the corresponding final BUG028 Issue closure comment after this record is published. A moving `main` pointer is not a substitute for either revision.
