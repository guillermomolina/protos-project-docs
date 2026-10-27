# I125-C — real NIO cross-Actor TcpAcceptor conformance (0.3.407)

**Date:** 2026-10-27. **Role:** published implementation evidence, not normative authority.
**Owning issue:** [I125 #966](https://github.com/guillermomolina/protos/issues/966).
**Ratified authority:** [D217 #965](https://github.com/guillermomolina/protos/issues/965), specification `0.1.473` at [product `5f95dc086fc9bc853afb712915db47ab28547417`](https://github.com/guillermomolina/protos/commit/5f95dc086fc9bc853afb712915db47ab28547417).
**Exact product publication:** [`guillermomolina/protos@ecbcd4c19acf9f2561ed03ff7a15351b175f7407`](https://github.com/guillermomolina/protos/commit/ecbcd4c19acf9f2561ed03ff7a15351b175f7407), **`0.3.407-SNAPSHOT`**.
**Preceding implementation:** [I125-A `bc3730539d73c94bb0f95cb9b699e4f2de0dde2d`](https://github.com/guillermomolina/protos/commit/bc3730539d73c94bb0f95cb9b699e4f2de0dde2d), [I125-B `da71217a4d4ecaba4b399e905aee119aef9d14fd`](https://github.com/guillermomolina/protos/commit/da71217a4d4ecaba4b399e905aee119aef9d14fd).

## Exact published delta

GitHub's published commit lists **three changed files only**:

1. `src/test/java/com/guillermomolina/protos/execution/integration/network/ProtosNioNetworkProvisioningTest.java`: a full APL Part 5 notice, supporting test helpers and two new real-NIO integration tests.
2. `pom.xml`: `0.3.406-SNAPSHOT` to `0.3.407-SNAPSHOT`.
3. `CHANGELOG.md`: I125-C's `0.3.407-SNAPSHOT` publication entry.

No production Java, Protos library file, normative specification, or `LICENSE.TXT` changed in I125-C. Its purpose is end-to-end evidence for the already-published runtime.

### `delegatedTcpAcceptorUsesRealNioAndWorkerOwnsAcceptedConnection`

A single provisioned host `Network` opens one real IPv4 loopback NIO listener with an ephemeral port. An accept-only capability is snapshot-transferred and materialized into another Actor hosted in the *same Process*. The Worker invokes `accept` through that capability while the listener-owning Actor invokes `accept` on the original listener.

Two real host sockets send distinct payloads. The test awaits both Actor domains, verifies two independent accepted connections over one listener/port, confirms each Future and connection belongs to its accepting Actor, and rejects cross-Actor reads. It matches peers by observed remote endpoint instead of assuming kernel accept order, reads the Worker's payload, writes a response from that Actor, and checks the matching host socket's bytes. It then closes both connections and the listener and checks `CLOSED_SUCCESS`.

### `delegatedPendingNioAcceptIsCutOverByOwnerListenerClose`

With a pending Worker accept, the owner closes its listener. Both Actor domains are driven until the Worker's accept Future fails with its Actor-local `IOLifecycleError` and the owner's close Future resolves. A second close returns a different already-resolved Future without reopening the lifecycle; a later accept returns a fresh Worker-owned failed Future with a distinct lifecycle Error. The test also checks no connection was delivered and no Worker completion remains queued.

These real-backend checks complement, and do not replace, the existing recording-backend tests that count exact release invocations and exercise close/cancellation races.

## Validation and publication evidence

- **Verified independently in GitHub:** exact `ecbcd4c1` product commit, commit title and three changed paths; the new two test methods and relevant assertions are present.
- **Maintainer report (2026-10-27):** product commit pushed; `git diff --check` clean; **all local tests passed**.
- **Not independently executed by this coordinator:** Maven, `make test`, `make check`, other validators or NIO network programs.
- **Exact integrated-gate caveat:** the maintainer's report did not name the command `make test` or `make check`. Do not label either explicit command `PASS` until confirmed. No test rerun should follow a metadata edit on the already-published candidate.

## Work boundary

I125-A/B/C together supply the D217 acceptance capability, same-Process spawn-only attenuation, shared listener lifecycle, Worker-local connections and real NIO end-to-end regression evidence. The remaining formal I125 closure step is to reconcile the integrated full-suite acceptance gate against the maintainer's already-completed validation (ask whether `make test`, and where applicable `make check`, were included), post the final evidence comment, and close #966 only when that condition is established.

The distinct [D216 #964](https://github.com/guillermomolina/protos/issues/964) remains open for the exact owner-ratified *continuous* HTTP/1.1 hosting API and lifecycle. Neither I125-C nor its evidence approves or implements such an API. A guest-Protos-first HTTP hosting design is to be selected before any implementation; existing `std:http/Server.serveOne.owning` and D217 `TcpAcceptor` should be composed rather than replaced.
