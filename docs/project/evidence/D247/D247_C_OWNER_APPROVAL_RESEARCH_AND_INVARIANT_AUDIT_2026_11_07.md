# D247 — Candidate C research, source evidence, owner approval and GITHUB021 invariant audit

**Date:** 2026-11-07. **Status:** RATIFICATION EVIDENCE ONLY; no I154 source, build, Native TCP test or release was produced by this research/coordination work.

- Decision: [D247/#1073](https://github.com/guillermomolina/protos/issues/1073).
- Implementation: [I154/#1076](https://github.com/guillermomolina/protos/issues/1076).
- [Durable ratified contract](../../decisions/tooling/D247_EXPLICIT_NETWORK_ROOTACTOR_SERVICE_CLI_RATIFIED.md).
- **Exact inspected Protos source:** [`8a6366e486d0b58b58cce17ce627734d9cadcc2d`](https://github.com/guillermomolina/protos/commit/8a6366e486d0b58b58cce17ce627734d9cadcc2d) (0.3.450-SNAPSHOT). HEAD advanced one I152 package-manifest migration from prior D247 baseline `9088e87d339558af168676a4dd3f31774e016b65`; that movement changed package files/docs, not the Network authority decision.

## 1. Exact approval provenance and bounded scope

After a detailed candidate comparison in this conversation, the assistant recommended **D247 Candidate C** and asked: **«¿Apruebas el candidato C con `--allow-network` y `--service=<binding>` como opciones independientes?»** The project owner answered **«si, aprobado»** on 2026-11-07. The follow-up **«pero hace falta otros allow? de otras capacidades?»** was answered by comparing existing grants and recommending separate decisions; the owner then requested publication and new issues. Therefore the approved D247 scope is **only** the two independent switches. D248/#1074 and D249/#1075 were allocated **as investigations**, not as ratified additional permissions.

GitHub coordination is an agent-produced record of the owner's actual approval **in this conversation**, not a purported owner-authored GitHub comment. Approval does not prove implementation, Native parity or validation.

## 2. Inspected source/contract observations

| Source at exact product revision | Concrete observed fact |
| --- | --- |
| `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java` | CLI supports explicit `--inspect` and `--foreign-provider-path`, but app entry paths select `NetworkGrant.NONE`; session seams already accept Network grant |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosStandaloneHostedExecution.java` | Network `NONE` vs `HOST_NETWORK` and **independent** Resolver `NONE` vs `HOST_RESOLVER` bootstrap choices |
| `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosRootActorServiceSession.java` | Owning RootActor service open/await/close host session exists; service binding selection is distinct from ordinary file one-shot |
| `src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageApplicationExecution.java` and package-run driver | Standard one-shot finally requests termination; existing package materialization scope must not be retired while service still runs |
| `spec/io/PROCESS_IO.md` §§22, 28 | Process-local slots; no implicit host authority, Process termination is host-custody transfer without an automatic guest HTTP drain |
| `spec/io/NETWORK.md` and D223-A1 | TCP/Network and hostname Resolver are orthogonal; Network does not imply DNS |
| `spec/io/HTTP.md`, `std:http/host`, D225-B1 | HTTP Host owns listener; D225 message-service root is a different readiness/lifecycle boundary; workers receive only delegated acceptor |
| `spec/semantics/MODULES.md`, D242, current I151/I152 | SOURCE local bindings differ from MODULE public exports (`this.service`) and package manifests use `package.toml` |
| `bin/protos`, `scripts/native-test.sh` | Native binary is preferred when installed; existing Native smokes do **not** certify real Native TCP/HTTP end-to-end |
| `src/main/java/com/guillermomolina/protos/execution/host/io/ProtosNioConfinedFilesystemBackend.java` | Existing confined direct-child file backend is not proof of a general host-directory Filesystem grant; D248 remains separate |

Sources above are paths in the linked product revision. D173/#645 + I065/#668 have already established explicit Network grant plumbing; D225/#992 and D223/#987 constrain service and name resolution. AUD020/#872 and I093/#873 explicitly did **not** prove installed Native TCP viability.

## 3. Prior-art mechanism comparison (not normative authority)

| System | Relevant mechanism | Protos consequence |
| --- | --- | --- |
| [Deno permissions](https://docs.deno.com/runtime/fundamentals/security/) | Explicit network opt-in, optionally scoped endpoints | Supports permission-first user experience; endpoint ACLs need additional implementation |
| [Node.js permissions](https://nodejs.org/api/permissions.html) | Host permission model distinct from application service architecture | Do not infer long-lived server lifecycle from a security flag |
| [WASI capability model](https://github.com/WebAssembly/WASI) | Guest access depends on handles/capabilities provided by host | Reuse Protos bootstrap-local objects instead of ambient global network |
| [GraalVM Polyglot embedding](https://www.graalvm.org/latest/reference-manual/embed-languages/) | Host Context policy determines available services | Keep CLI grant a host decision feeding the existing Protos capability |
| [Erlang/OTP supervisors](https://www.erlang.org/doc/system/sup_princ.html) | Explicit application/service lifecycle instead of accidental process retention | Service lifetime needs an owning entry contract; do not invent another OTP runtime |
| [Python asyncio server](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.create_server) | Explicit listener and long-lived event-loop supervision | TCP listen and Process continuation must both work |
| [Go HTTP Server](https://pkg.go.dev/net/http#Server.Shutdown) | Serve lifecycle and deliberate shutdown are distinct | No fake automatic graceful drain on Ctrl-C |
| [Tokio shutdown](https://tokio.rs/tokio/topics/shutdown) | Cancellation/lifetime coordinated by owner | Scope captured resource cleanup to real service termination |

These are mechanism parallels only. Do not port their threading, context or shutdown semantics into Protos without separate authority.

## 4. GITHUB010 candidate scoring

Scale: **1 poor → 5 strong**; suffix **H/M/L** = high/medium/low confidence in qualitative reasoning, **not** empirical performance. A status quo Java embedding; B Network flag only; C independent Network + service flags; D dedicated `service` command; E granular ACLs first; F manifest permission; G detect service implicitly.

| Dimension | A | B | C (approved) | D | E | F | G |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Correctness and invariant preservation | 5H | 3M | 5M | 4M | 3L | 1H | 1H |
| Protos-first alignment | 2H | 3M | 5M | 4M | 3L | 1H | 1H |
| Current-need sufficiency | 1H | 2H | 5M | 5M | 4M | 4M | 3L |
| Incremental addition of later capabilities | 2M | 4M | 5M | 4M | 3L | 2H | 2M |
| Future-option resilience | 3M | 4M | 5M | 4M | 4L | 2H | 1H |
| Scalability / serving lifecycle | 2H | 2H | 5M | 5M | 4L | 3L | 2M |
| Conceptual simplicity | 4H | 5M | 4M | 3M | 1M | 3M | 2M |
| JVM / Native parity feasibility | 4M | 4L | 4L | 4L | 2L | 2L | 2L |
| Runtime/resource cost when unused | 5H | 5M | 5M | 4M | 2L | 3M | 2M |
| Failure diagnosis / operability | 2H | 3M | 5M | 5M | 3L | 2M | 1H |
| Deferral / reversibility | 3M | 5M | 5M | 4M | 2L | 1H | 1H |
| Evidence and implementation maturity | 5H | 4M | 4M | 4M | 2L | 2H | 2M |

Why C wins: it completes the present use case with two independently testable opt-ins and existing grants/RootActor service architecture. A forces Java for users; B still lacks long-lived service semantics; D is viable but adds a new command institution; E implements speculative network policy without backend admission proof; F violates explicit host-grant sovereignty; G contradicts existing D225 mode selection. Low confidence on Native viability is a **verification gate**, not a reason to claim it already works.

## 5. GITHUB021 invariant / delta consistency audit

| Prior owner-approved or normative invariant | D247-C preservation and delta | Result |
| --- | --- | --- |
| D173 / `PROCESS_IO.md`: host-granted Network explicitly; absence otherwise | New CLI switch selects existing `HOST_NETWORK`; no permission by default | PASS |
| D223-A1: Resolver independent from Network | `--allow-network` never supplies `resolver`, even for a server; D249 is separate | PASS |
| D225-B1: RootActor direct entry vs explicit message-service entry | `--service=<binding>` selects existing service entry; no heuristic binding detection | PASS |
| D242 SOURCE/MODULE separation | SOURCE local binding, MODULE exported own binding, no private-export bypass | PASS |
| Process/Actor/module isolation | Root bootstrap-local authority; no Actor/import/Tool ambient propagation | PASS |
| `PROCESS_IO.md` §28 host cleanup | No implicit HTTP drain, callbacks after termination or invented signal guarantee | PASS |
| I065 existing grant seam | Reuses established `NetworkGrant` rather than manufacturing a new permission-bearing object | PASS |
| CLI010 `--inspect`, I085 provider opt-in | Existing independent options preserved; no `--allow-all` authority bundle | PASS |
| Owner-approved scope of D247-C | Exactly two orthogonal flags; later discussion was **not** approval for Filesystem/Resolver | PASS |

**No known conflicting owner-approved D247 invariant was discovered.** One material consequence is the new public CLI entry-mode and service lifetime (intended purpose of Candidate C), explicitly contained in the approval packet. Future real Native failures or contradictory evidence must be reported, not silently reinterpreted as approval to change the semantics.

## 6. Falsification and I154 test gates — NOT YET RUN

1. No-flag direct file or `protos run` cannot see `network`; a Network-only application has no `resolver` or `filesystem` binding. Bundled Tools cannot request a grant.
2. Source `service:...` and Module `this.service:...` work only in their respective entry kinds. A private Module binding cannot be exported by the launcher. `--service` with REPL, `-e`, `debug` or Tool fails usage.
3. All permutations: flag-before-entry placement, duplicates, missing binding, trailing application args, `--inspect` and provider option coexistence, invalid command combinations, exit codes 2/1/70.
4. Direct SOURCE and package MODULE service survive past initial evaluation; multiple sequential actual HTTP requests succeed; package/mount/captured scopes remain live; failure/exit cleanup retires owned resources.
5. Real installed **Native binary**, not JVM wrapper or Java embedding: loopback TCP listen/accept/close and HTTP request/response; compare with JVM; verify absence of Java compiler/JDK bootstrap on user machine. Record environment and exact binaries.
6. Actor/network isolation and explicit `TcpAcceptor` transfer, separate RootActor READY and HTTP `Host.ready`, cancellation, pending sessions and process termination. Do not infer graceful HTTP drain from Process shutdown.

All tests are **planned**, not executed. Native/JVM support and signal behavior remain **unverified**; I154 owns future implementation, evidence and any necessity for a new design decision.

## 7. Strongest objection, regret and future extension

**Objection:** a `protos service` subcommand may read more clearly and facilitate future operations, while unrestricted `HOST_NETWORK` can listen on public interfaces. The counterargument is that Protos already has an explicit RootActor service entry and exact host grant: independent options preserve orthogonality, avoid new CLI-only service ontology and keep the unused path cheap. They do **not** provide an OS sandbox or outbound host/port restriction. Operators must choose their listener address and use host/container firewall controls as appropriate.

**Regret scenario:** future deployment needs whitelist-only network access or per-service permissions. Safe extension is a separately approved narrowing/policy syntax, not retroactive meaning change of `--allow-network`; host provisioning remains the enforcement boundary. D248/D249 may independently add confined file or DNS authority without changing service semantics. If viable Native support requires a different host backend, retain the guest-visible authorization and service contracts, changing only the host substrate with bounded evidence.

**GITHUB010 incremental floor:** first make one real network server usable from the installed CLI on Native and JVM. Do not simultaneously create a permission registry, broad Filesystem mount, Resolver coupling, generic daemon manager, automatic package authorization, arbitrary native execution, or cluster service supervision.

## 8. Publication/provenance limits

All three Issues D248/#1074, D249/#1075, I154/#1076 are separate formal work items. The `I154` textual `Parent: #1073` is not itself a native GitHub sub-issue link: native parentage must be verified and repaired by the authorized hierarchy workflow if not already established. The owner approved D247-C and authorized publishing this evidence; no new permissions for other capabilities were owner-approved, and no product implementation, tests, build or release are asserted.
