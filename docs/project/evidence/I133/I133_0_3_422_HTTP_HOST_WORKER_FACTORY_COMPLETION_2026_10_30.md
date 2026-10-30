# I133 — Protos-first explicit HTTP Host Worker factory (0.3.422)

**Date:** 2026-10-30. **Role:** revision-bound, non-normative publication and validation evidence for [I133 #995](https://github.com/guillermomolina/protos/issues/995), implementing the owner-approved partial decision [D225-A #992](https://github.com/guillermomolina/protos/issues/992). Normative semantics remain exclusively in `guillermomolina/protos/spec/`; GitHub Issues remain live work authority.

**PROTOS_REVISION:** [`e88f4f0e510320ede0ec61bd67dfd671f30fc2cb`](https://github.com/guillermomolina/protos/commit/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb) — `I133: Implement explicit HTTP Host Worker factory`.
**Implementation version:** `0.3.422-SNAPSHOT`; **global specification revision:** `0.1.481`.
**Normative owner:** [`spec/io/HTTP.md` §15.13 at exact revision](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/io/HTTP.md); [specification revision entry](https://github.com/guillermomolina/protos/blob/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb/spec/PROTOS_SPEC_CHANGELOG.md).
**Decision provenance:** [D225-A selected record](../../decisions/language/D225_EXPLICIT_HTTP_HOST_WORKER_FACTORY.md) and [research-stage evidence](../D225/D225_A_RESEARCH_AND_APPROVAL.md). D225-B service/broker authority remains unselected.

## Exact published implementation

The verified product commit modifies/creates **12 paths**: `protos/lib/http/Host.protos`, `internal/HostLifecycle.protos`, `internal/HostWorker.protos`; two suite-native Host tests (`host-construction.protos`, `host-lifecycle.protos`); two Java **test-only** classes (`ProtosHttpHostFactoryIntegrationTest.java`, `ProtosHttpHostFactoryFixtureSources.java`); `spec/io/HTTP.md`, `spec/PROTOS_SPEC_CHANGELOG.md`, `docs/guide/library/http.md`, `pom.xml` and `CHANGELOG.md`. No Java production/runtime/Truffle primitive changes appear in the published path set.

- The new, opt-in operation is `Host.owningWithFactory(listener, factoryModule, factoryBinding, initialization, options) -> host`. The original `Host.owning` remains present. `std:http/Host` still exposes `Host` and `HostError`.
- The constructing Actor supplies `initialization` to each Worker through its ordinary `worker.request("serve", initialization)` message, **not** through `Actor.spawn`. The ordinary snapshot/copy boundary preserves `ActorRef` destination identity and rejects caller-supplied `TcpAcceptor`, `Network`, Resolver, Future, Closure and other non-transferable values. The special spawn delegation remains reserved for the Host's own derived acceptors.
- Each Worker imports its factory module locally, invokes the factory at most once, validates a callable result, and reuses it for `handler(request)`. Factory results are not automatically awaited/adopted. Repeated `serve` cannot invoke the factory again.
- **Two readiness boundaries are explicitly distinguished:** the Worker may become Actor `READY` before receiving its initialization; `host.ready()` resolves only when all Workers have finished construction **and entered the acceptance service**. The final `HostWorker` includes an `entered` gate avoiding the earlier construction-Future waiter race. Failure/cutover paths also open this gate; abort cooperatively cancels suspended construction. The normative text explicitly places this rule under HTTP hosting readiness (§§15.7, 15.13), without changing Core Actor readiness.
- D216 listener custody, quota distribution, draining, abort, Worker stop, cleanup precedence, and host observation behavior remain the underlying machinery. Referenced application service Actors remain independently owned: Host neither provisions nor terminates them. The guide shows composition with a **pre-existing** ActorRef, not a newly authorized outbound broker.

## Validation provenance

The maintainer reported after pushing the complete I133 commit on 2026-10-30:

```text
WORK_ITEM=I133
PROTOS_REVISION=e88f4f0e510320ede0ec61bd67dfd671f30fc2cb
PRODUCT_VERSION=0.3.422-SNAPSHOT
GLOBAL_SPEC_REVISION=0.1.481
COMMIT_CHANGED_PATHS=12
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
AGENT_RERAN_TESTS=NO
NATIVE_IMAGE_VALIDATION=NOT_CLAIMED
LIVE_PUBLIC_NETWORK_TEST=NOT_CLAIMED
```

The I133 implementation discussion recorded an earlier **pre-repair** seven-error integration-test run; two subsequently focused tests passed, and a real `ready` race was then found and corrected with the `entered` gate. The final integrated PASS reported by the maintainer is **after the final code was committed and pushed**; this evidence does not infer per-test result counts or a separate native-runtime gate from the summary. The two Java test classes are tests/fixtures only and not production authority changes.

## GITHUB021 invariant and residual-design reconciliation

Preserved from approved D225-A: an explicit opt-in public API, one Worker-local factory/handler, ordinary Actor-value copying, retained ActorRef identity and authority, strict non-transferability, only Host-internal `TcpAcceptor` special delegation, unchanged `Host.owning` and D215 `handler(request)`, existing D216 cleanup/quotas, no ambient networking grant. The term “Worker READY” in the original selection is clarified in the published HTTP normative section as **hosting readiness**, not `ACTORS.md` Actor READY; the Worker Actor exists and can receive messages before the Host starts accepting. This clarification was discussed during implementation and is documented explicitly in the published specification, without revising Core Actor lifecycle.

**What is NOT delivered by I133:** automatic provision of an outbound `Network`/`Resolver`-owning broker Actor, a RootActor message-serving interface, attenuated ActorRefs, service registry/DI, automatic retries or new TLS/REST/DNS behavior. The remaining D225 application-service authorization/provisioning question remains **open** and requires separately approved design; I133 closure does not close D225.

## Publication / audit boundary

- `PROTOS_REVISION` is exact, retrieved from the published GitHub commit with its 12 changed paths.
- Local gate results are the maintainer's explicit report, **not** independently executed by the documenting agent.
- This record is a snapshot of I133 product publication; it does not become a second normative source or substitute for live GitHub Issue state.
