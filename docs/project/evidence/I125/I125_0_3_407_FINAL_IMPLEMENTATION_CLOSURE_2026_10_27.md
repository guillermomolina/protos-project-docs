# I125 — final D217 TcpAcceptor implementation closure (0.3.407)

**Date:** 2026-10-27. **Role:** exact-revision implementation closure evidence, not normative authority.
**Owning work:** [I125 #966](https://github.com/guillermomolina/protos/issues/966).
**Ratified design:** [D217 #965](https://github.com/guillermomolina/protos/issues/965), spec `0.1.473` at [`5f95dc086fc9bc853afb712915db47ab28547417`](https://github.com/guillermomolina/protos/commit/5f95dc086fc9bc853afb712915db47ab28547417).
**Final product revision:** [`guillermomolina/protos@ecbcd4c19acf9f2561ed03ff7a15351b175f7407`](https://github.com/guillermomolina/protos/commit/ecbcd4c19acf9f2561ed03ff7a15351b175f7407), version **`0.3.407-SNAPSHOT`**.
**Source verification:** The GitHub `main` ref resolves to the above exact product SHA at this closure checkpoint. This coordinating agent did not execute local tests.

## Implemented and published work

- **I125-A / `0.3.405` / [`bc373053`](https://github.com/guillermomolina/protos/commit/bc3730539d73c94bb0f95cb9b699e4f2de0dde2d):** canonical frozen `TcpAcceptor` prototype, `listener.acceptCapability()`, Actor-local accept calls sharing the existing logical TCP listener/flow, lifecycle and negative-transfer coverage.
- **I125-B / `0.3.406` / [`da71217a`](https://github.com/guillermomolina/protos/commit/da71217a4d4ecaba4b399e905aee119aef9d14fd):** `Actor.spawn` initialization-only delegation between Actors of the same Process; fresh destination-local capability without copying source slots/state; graph atomicity, aliases, Actor-local accepted connections, owner-published close outcome, cancellation/termination and forbidden transfers.
- **I125-C / `0.3.407` / [`ecbcd4c1`](https://github.com/guillermomolina/protos/commit/ecbcd4c19acf9f2561ed03ff7a15351b175f7407):** two real NIO/IPv4 loopback integration tests for simultaneous original-listener and delegated-Worker acceptance on one port, Worker-specific connection ownership/read/write, and owner-close cutover of pending Worker accepts. Exactly three changed paths in the final slice (test, `pom.xml`, `CHANGELOG.md`); no additional production Java.

Published evidence checkpoints: [A](I125_A_0_3_405_ACTOR_LOCAL_ACCEPTOR_PUBLICATION_2026_10_26.md), [B](I125_B_0_3_406_SAME_PROCESS_ACTOR_DELEGATION_2026_10_26.md) and [C](I125_C_0_3_407_REAL_NIO_CONFORMANCE_2026_10_27.md). The C checkpoint correctly documents what was known before explicit full-gate confirmation; this final record adds the subsequent statement.

## Final human validation confirmation

The maintainer first reported on 2026-10-27 that I125-C was pushed, `git diff --check` was clean and all local tests passed.

When explicitly asked whether **both `make test` and `make check`** were included, the project owner answered **“sí”** on 2026-10-27. This is the human executor's explicit PASS confirmation for both canonical gates on the already-published final candidate. The coordinator neither reran nor independently observed the commands, and no raw transcripts were provided. No tests were requested or run after the release metadata publication.

## Closure boundary

I125 satisfies its ratified D217 acceptance and formal human integrated-validation gate, with final product SHA and durable cross-repository evidence. Close I125 as **completed**, not deferred.

Continuous HTTP hosting is **not** part of I125. [D216 #964](https://github.com/guillermomolina/protos/issues/964) remains open pending the owner's exact continuous-hosting API/lifecycle approval; closed D215/LIB025/D217 semantics and I125 TCP capabilities are preserved.
