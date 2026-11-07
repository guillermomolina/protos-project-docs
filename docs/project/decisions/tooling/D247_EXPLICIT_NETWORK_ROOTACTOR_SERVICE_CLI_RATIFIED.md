# D247 — Explicit Network grant and RootActor service mode in the Protos CLI

**Status: RATIFIED — owner-approved Candidate C, 2026-11-07.** This is an implementation-independent CLI/hosting policy, not a claim of implementation, Native Image TCP success or new language syntax.

- Decision owner: [D247/#1073](https://github.com/guillermomolina/protos/issues/1073).
- Implementation: [I154/#1076](https://github.com/guillermomolina/protos/issues/1076), published in [`guillermomolina/protos@398e6cb7478b0d7735ffcb6787d43c723cc32937`](https://github.com/guillermomolina/protos/commit/398e6cb7478b0d7735ffcb6787d43c723cc32937) (`0.3.451-SNAPSHOT`). [Revision-coupled JVM/Native evidence](../../evidence/I154/I154_D247_C_NETWORK_ROOTACTOR_CLI_JVM_NATIVE_IMPLEMENTATION_EVIDENCE_2026_11_07.md).
- Related adoption: [I153/#1068](https://github.com/guillermomolina/protos/issues/1068).
- Independent, **unratified** CLI decisions: [D248/#1074](https://github.com/guillermomolina/protos/issues/1074) (confined Filesystem) and [D249/#1075](https://github.com/guillermomolina/protos/issues/1075) (Resolver). Neither is selected or implemented here.
- Owner approval provenance: in the active 2026-11-07 conversation, after the D247 investigation explicitly recommended **Candidate C** with independent `--allow-network` and `--service=<binding>`, the owner replied **«si, aprobado»**. A follow-up asked about other capabilities and then authorized issue/evidence publication; it did **not** approve the other grants.
- Research, alternatives, source and approval/invariant audit: [D247 evidence](../../evidence/D247/D247_C_OWNER_APPROVAL_RESEARCH_AND_INVARIANT_AUDIT_2026_11_07.md).
- Inspected product baseline: [`guillermomolina/protos@8a6366e486d0b58b58cce17ce627734d9cadcc2d`](https://github.com/guillermomolina/protos/commit/8a6366e486d0b58b58cce17ce627734d9cadcc2d), 0.3.450-SNAPSHOT. This is a **pre-I154** source snapshot, not an implementation revision.

## Selected public CLI interface

Two independent, explicitly selected options with no implicit authority inference:

```text
protos --allow-network application.protos arg1 arg2
protos --allow-network --service=service server.protos arg1 arg2
protos run --allow-network app
protos run --allow-network --service=service app
protos --allow-network -e '...'
protos --allow-network                         # REPL
protos debug --allow-network server.protos
protos --service=service local-service.protos  # service mode alone grants no Network
```

**`--allow-network`** provisions existing `NetworkGrant.HOST_NETWORK` into the initial application's bootstrap-local `network` slot. The default is `NONE`/absent. This initial grant is deliberately broad host TCP authority, not a host/port/CIDR-scoped policy. It does not grant a Resolver, Filesystem, subprocess, foreign-native, inspection or general host capability.

**`--service=<binding>`** selects the existing D225-B1 **RootActor message-service entry** and its long-lived owning session. It is independent of Network: a service may be local-only and a short-lived program may use the Network. The CLI never detects service mode from an entry-binding name, an imported `Host`, presence of `network`, an HTTP listener, or package contents.

### Entry-mode eligibility and argument placement

| Execution entry | `--allow-network` | `--service=<binding>` |
| --- | --- | --- |
| Direct SOURCE file | Yes | Yes: exact SOURCE-local binding |
| `protos run <app>` | Yes | Yes: exact public MODULE export |
| `-e` | Yes | No |
| REPL | Yes | No |
| `protos debug` | Yes | No (first release) |
| `protos test`, `package`, `format`, `lint`, language-server or another Tool | No | No |

Flags must appear in the CLI option region before the entry/target, or following the `run`/`debug` command before its target. Arguments after the chosen source or package entry are application arguments, preserving their order and exact `process.args()` snapshot. No network flags are injected into Tool execution or child Processes. Duplicate flags, invalid/missing binding names, illegal command combinations and unknown option placements fail as usage errors (exit 2); guest failures use the existing guest-failure exit (1) and internal failures retain 70. Preserve current `--inspect` and `--foreign-provider-path` independent opt-in behavior.

### SOURCE/MODULE authority and service behavior

- A direct SOURCE can select its exact local `service: ...` RootActor bootstrap binding. A package `protos run` entry is a MODULE under D242; its service must be an exported own binding such as `this.service: ...`. A private MODULE binding cannot be treated as a public service export.
- Service bootstrap invokes the selected binding once using the D225-B1 message-service contract, installs the resulting behavior and exposes RootActor READY according to that contract. RootActor readiness is **not** proof that `std:http/host` has completed listener startup; `Host.ready()` remains a separate readiness boundary.
- A service stays alive until its real Process/service termination. The CLI may not reuse one-shot `finally` cleanup that destroys the Process, Context, RuntimeHost or package/materialization scope immediately after evaluating the entry.
- For HTTP, use the existing `std:http/host` / `Host.owning(...)` behavior and explicit transfer of a `TcpAcceptor` to workers. The Network grant is not silently copied into imported modules, Actor children or worker threads.
- `Process` termination retains the normative §28 custody/cleanup semantics. CLI Ctrl-C/signal mapping and graceful HTTP drain must be tested/documented; this decision does **not** claim an implicit drain, callbacks after termination, automatic restart or enhanced signal protocol.

## Preserved authority boundaries

1. Explicit grant at the **initial RootActor** only; no `network` with default invocation.
2. `resolver` is independent under D223-A1, and remains **absent** for Network-only CLI use. Hostname resolution from this option is not promised.
3. `filesystem` remains absent on standalone CLI application entry; reading the program's source/package does not confer guest file I/O.
4. Import, standard-library module access, package dependency presence and Actor spawning never grant host authority. Bundled Tools retain their narrower separate host grants.
5. Native and portable JVM must present the same public contract where supported; native TCP ability remains to be verified by running the actual installed Native binary.
6. Service is a Process/RootActor execution-mode selection, not a new general daemon manager, a permission bundle or a package manifest declaration.

## Alternatives and incremental-design judgment

- A — retain Java host/bootstrap only: rejects the requirement for an ordinary installed Protos executable.
- B — `--allow-network` only: useful for client/one-shot TCP, but does not define a long-lived D225 service for HTTP hosting.
- **C — independent `--allow-network` and `--service=<binding>`: selected**; reuses existing grants, runtime and service semantics with opt-in costs.
- D — separate `protos service` subcommand: viable alternative but a new top-level CLI institution without additional semantic value at present.
- E — granular network ACL/host/port policy now: more narrow in principle, but not backed by the current Host Network grant and imposes speculative policy and platform divergence.
- F — automatic permission from `package.toml`: rejected because dependency/manifest content must not create host authority.
- G — infer service mode by binding, import or HTTP Host presence: rejected because it contradicts D225's explicit mode and makes execution effects depend on incidental code.

Keep additional capability grants as separately approved small extensions, not a mandatory universal permission manager. D248 (Filesystem) and D249 (Resolver) are open research decisions. The initial host Network grant does **not** imply either.

## Implementation and acceptance

`I154` owns the code and verification. Expected touchpoints include `ProtosCli`, package-run application execution/driver, session ownership and RootActor service bootstrap; CLI documentation, relevant normative text only when a public observable contract requires it, JVM/Native test matrices and distribution checks. Verify a genuine native executable listening on loopback TCP and serving a real HTTP response as well as JVM parity, multi-request persistence and cleanup. Reject missing grants, invalid modes, implicit Tool/Actor inheritance and false RootActor/HTTP readiness assumptions.

**Ratification/publication boundary:** D247 was ratified before the I154 implementation and did not itself publish CLI functionality. Subsequent I154 product publication is the distinct revision `398e6cb7478b0d7735ffcb6787d43c723cc32937`, with maintainer-reported JVM and Native SOURCE HTTP evidence in the linked I154 record. This update does not retroactively turn the original D247 research into implementation evidence. I154 acceptance status remains governed by its GitHub Issue, including the not-separately-reported Native MODULE `run` HTTP smoke. No additional Filesystem or Resolver permission is ratified here.
