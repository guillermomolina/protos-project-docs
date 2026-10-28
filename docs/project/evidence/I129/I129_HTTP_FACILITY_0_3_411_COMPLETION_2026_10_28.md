# I129-A — Private HTTP facility `invokable` deduplication (0.3.411)

**Recorded:** 2026-10-28. **Role:** exact-revision implementation and human-validation evidence; not normative authority.
**Implementation issue:** [I129 #974](https://github.com/guillermomolina/protos/issues/974).
**Ratified decision:** [D219-B #973](https://github.com/guillermomolina/protos/issues/973), with [owner-approved decision record](../../work/D219/D219_PRIVATE_RUNTIME_CAPABILITIES_RATIFICATION_2026_10_28.md).
**Exact published Protos revision:** [`guillermomolina/protos@7f3113ad5c29a15584053c6440640bff6911e60a`](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a) on `main`.
**Implementation version:** `0.3.411-SNAPSHOT`. **Specification:** unchanged by I129.
**Product commit title:** `I129-A: remove redundant invokable from ProtosHttpAsyncFacility in favor of provides(value, \"call\") (0.3.411)`.

## Published scope

The product commit changes six paths: `CHANGELOG.md`, `pom.xml`, `protos/lib/http/Client.protos`, `protos/lib/http/Server.protos`, `protos/lib/http/internal/HostWorker.protos` and `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java`.

- HTTP Server admission now checks `facility.provides(handler, "call")`.
- Worker bootstrap continues to import and resolve its handler in its own Actor, demands the own module binding, then checks `facility.provides(handler, "call")` before starting acceptance.
- HTTP Client admission now checks `facility.provides(onResponse, "call")`. This is the third consumer at the published HEAD: I127 introduced `std:http/Client` after the D219 baseline audit, so I129 included its newly present `invokable` call site rather than leaving an orphaned use.
- The facility no longer registers or implements `invokable(value)`. Its JavaDoc descriptions were updated. The package declaration, `hasCallable` delegated non-invoking lookup and its `ProtosClosureValue` filter and exception handling remain unchanged.
- Exactly eight private operations remain, with unchanged arities: `failed`/1, `resolved`/1, `provides`/2, `closable`/1, `ordinaryObject`/1, `isBytes`/1, `startService`/1, `enterService`/0. The facility remains frozen, deferred and module-private for each Actor-local module instance.

No new generic runtime service, registry, public Core API, global allocation, Task, Actor or normative specification change was introduced. The D215/D216/D217, I126/I127 and D121 behavior boundaries remain governed by their existing sources and specification. The broader runtime-capability architecture remains deferred under D219-B.

## Validation and publication evidence

Guillermo Adrián Molina, acting as Human Executor, confirmed on 2026-10-28 that the published I129-A candidate passed the following local gates:

| Gate | Result and evidence provenance |
| --- | --- |
| `git diff --check` | PASS — reported clean |
| `make test` | PASS — local integrated Java/JUnit and Protos tests, owner-reported |
| `make check` | PASS — local routine static/compilation checks, owner-reported |
| Product commit and push | PASS — GitHub confirms exact `main` commit [`7f3113ad5c29a15584053c6440640bff6911e60a`](https://github.com/guillermomolina/protos/commit/7f3113ad5c29a15584053c6440640bff6911e60a) |

The separate preliminary three-file diff review was clean before the current-HEAD I127 Client call site was also included in the final commit. The product commit itself was inspected through GitHub for the final six-path scope. No raw test logs, exact test counts, independent test execution by this coordinating agent, or post-version-bump retest are claimed.

## Closure and remaining boundaries

I129 #974 implements the exact D219-B private-operation deduplication, including all actual HTTP consumers at the release revision. The exact product revision above and this durable project record support completed Issue closure after the project-record publication is verified. I128 #972 remains closed as not planned. D219's ratified choice and trigger for any future generic extraction remain unchanged: require an independent second consumer with genuinely equivalent authority/lifetime semantics or concrete high deferral cost.

This record does not modify Protos language semantics, HTTP wire behavior, Future/Task cancellation, Actor ownership or any normative `spec/` contract.
