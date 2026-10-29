# D223 — Explicit Resolver authority and hostname-to-TCP composition (A1)

**State:** RATIFIED after exact owner approval and reapproval of the host-name-service cache qualification, 2026-10-29.

**Live decision:** [D223 #987](https://github.com/guillermomolina/protos/issues/987). **Implementation:** [I132 #990](https://github.com/guillermomolina/protos/issues/990).
**Normative source:** [guillermomolina/protos@aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9](https://github.com/guillermomolina/protos/commit/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9), specification revision `0.1.480`, primarily [`spec/io/NETWORK.md` §14](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/NETWORK.md) and [`PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9/spec/io/PROCESS_IO.md).
**Implementation evidence:** [I132-A / `0.3.421`](../../evidence/I132/I132_A_0_3_421_RESOLVER_COMPLETION_2026_10_29.md), published at project-docs revision [`704a9f5b8c1b21510786948a4e89545170f1be56`](https://github.com/guillermomolina/protos-project-docs/commit/704a9f5b8c1b21510786948a4e89545170f1be56).
**Exact GITHUB021 reconciliation and closure evidence:** [D223 normative ratification](../../evidence/D223/D223_A1_SPEC_0_1_480_RATIFICATION_2026_10_29.md).

## Approval provenance

1. On 2026-10-29 Guillermo Adrián Molina selected the entire D223-A1 design with the statement recorded in [D223 #987](https://github.com/guillermomolina/protos/issues/987): “Apruebo el contrato completo D223-A1 propuesto, incluidos sus límites y aplazamientos.”
2. After implementation, the published `NETWORK.md` §14.9 explicitly qualified A1's original cache-isolation promise: no **Protos-managed** mutable caches or result state cross Resolver authorities/Contexts, while caching internal to an authorized host name service (JVM/OS) is excluded from that promise. This was surfaced as an observable narrowing, not presumed from the original A1 approval.
3. The owner was asked, verbatim, “¿Apruebas la excepción de §14.9 que limita el aislamiento a las cachés gestionadas por Protos y excluye las cachés internas del servicio de nombres del sistema operativo o JVM?” The immediately following owner response on 2026-10-29 was **“Si apruebo”**. This is explicit approval of the exact published qualification, not a generic instruction to continue.
4. The approval does **not** extend any deferred facility or authorize a new implicit Network/Resolver grant. No additional product edit or build is entailed by recording the already-published, now-approved §14.9 text.

## Ratified public contract

- `std:network/Resolvers` exports exactly the authority-free `Resolver` protocol and `ResolutionError`. A concrete live Resolver requires an independent, explicit host grant and optional RootActor bootstrap-local `resolver` slot; neither import nor `Network` nor `Process` authority implies one. The capability is non-transferable to another Actor or P.
- `lookupAddresses(name) -> Future<Array<IpAddress>>` and `close() -> Future<Resolver>` use Protos-first policy, ordinary Future cancellation and I/O lifecycle. Name admission accepts a semantic String of strict ASCII NR-LDH/IDNA2008 A-labels, optional terminal dot, one label allowed, 63/253-octet bounds; no U-label/UTS#46 conversion, search-domain expansion or implicit numeric-literal lookup. TLS's separately approved DNS-reference identity remains unchanged.
- An admitted host name service may return data from A/AAAA, aliases, local or split-horizon sources. A successful lookup yields fresh frozen data with 1–64 distinct canonical frozen `IpAddress` values, without TTL, authority, alias or reachability guarantee; more than 64 distinct addresses fails atomically, and the implementation bounds raw materialization to 256 answers.
- The four `ResolutionError.kind` values are `noAddress`, `unavailable`, `denied` and `limit`, with separate `InvalidIOArgument` and `IOLifecycleError`. The host adapter can report `noAddress` for a `UnknownHostException` it cannot distinguish from transient failure; it does not promise stronger host-diagnostic provenance.
- Host workers, queues and retained states are bounded. Guest Future settlement and result commitment occur on the owning Actor, never on host backend workers. Pre-commit cancellation suppresses late publication but does not promise physical OS query interruption. Each concrete capability owns its Protos backend and closes/retire with its own lifecycle, Process termination or host closure.
- **Approved §14.9 qualifier:** Protos itself does not share mutable cache/result state between Resolver authorities or Polyglot Contexts. The authorized operating system/JVM name service may maintain shared caches; they are outside the Protos-managed-state isolation guarantee, are not exposed as Resolver data/API or authority, and do not bypass host grants. No DNS TTL/cache API is exposed.
- `Network.connectTcp(IpEndpoint)` remains numeric, independently authorized and separate from resolution. Original hostname remains the reference for HTTP Host and TLS SNI/certificate identity; resolution grants no egress permission or SSRF prevention guarantee.

## Explicit deferrals and authority

No `NameConnector` or hostname `connectTcp`, Happy Eyeballs, public DNS records/`std:dns`, DNSSEC/DoH/DoT, public cache/TTL controls, relative search domains, Actor resolver delegation, deadlines/timers, URI/REST/routing/redirect/retry change or TLS policy change is part of A1.

This decision record is **non-normative**. The exact public contract remains the published `spec/` documents at `aa2a8e1c8a51ae8cb5b14bf69ee3a607ce279ce9`. The already-published human-validated I132-A implementation and its evidence are independent of whether later separately tracked work changes the runtime.
