# D225-B1 — RootActor service bootstrap: owner selection and consistency evidence

**Evidence date:** 2026-10-30.
**Type:** immutable/snapshot-like design investigation and approval evidence, **not** a normative specification or implementation proof.
**Formal owner:** [D225 #992](https://github.com/guillermomolina/protos/issues/992).
**Selected decision record:** [D225-B1](../../decisions/language/D225_B1_OPT_IN_ROOT_ACTOR_MESSAGE_SERVICE_BOOTSTRAP.md).
**Prior selected/implemented decision:** [D225-A](../../decisions/language/D225_EXPLICIT_HTTP_HOST_WORKER_FACTORY.md), [I133 implementation evidence](../I133/I133_0_3_422_HTTP_HOST_WORKER_FACTORY_COMPLETION_2026_10_30.md).
**Inspected baseline:** [Protos `e88f4f0e510320ede0ec61bd67dfd671f30fc2cb`](https://github.com/guillermomolina/protos/commit/e88f4f0e510320ede0ec61bd67dfd671f30fc2cb), spec revision `0.1.481`; **not** a published B1 revision.

## Exact approval and non-inference

The project owner, after the comparative D225-B1 investigation in the active interaction, stated:

> Apruebo D225-B1: bootstrap explícito opt-in del RootActor como servicio de mensajes y GroupRef de comunicación, con las condiciones y límites del informe. Procede con la ratificación según AGENTS.md.

This is explicit approval of **B1**. It is **not** approval of alternative B2 independent Actor network/Resolver grant, C per-Worker grants, arbitrary `Actor.serve()`, a new ActorRef attenuation semantic, automatic service discovery, a new CLI flag spelling, or an unsupported promise that the standard embedding grants Resolver.

The prior, separately approved D225-A stays complete and published in I133. B1's contract is an additional **optional RootActor initial-entry bootstrap mode** plus application-level single-member communication-only GroupRef; the GroupRef operation already exists. The application retains responsibility for service lifecycle, exposed selectors, credentials and egress validation.

## Comparative alternatives, non-compensating review

**B0 — defer/status quo.** No additional public/specified behavior. Zero runtime cost, highest compatibility, but an ordinary Host Worker still cannot generally obtain a newly created root-authorized message service without an explicitly service-ready RootActor.

**B1 — selected RootActor bootstrap + one-member GroupRef.** Uses the current RootActor, existing module/binding bootstrap and existing GroupRef identity/transfer, with no `Network`/`Resolver` transfer or new Actor public method. Costs one opt-in entry mode plus application-owned request/response routing; RootActor has Process-wide fatal-failure consequences and single-Actor execution.

**B2 — independent service Actor with explicit capability provisioning.** Better isolation and parallelism. Requires new host-authorized, scope-limited provision/delegation contracts for two non-transferable resources with creation, shutdown, revocation, placement, identity, error and Process termination rules. No demonstrated present need justifies those costs now; preserves future escape path.

**C — worker-local outbound grants.** Avoids hop and central broker, but mixes inbound/egress concerns and multiplies live outbound authority across Workers, conflicting with the minimal D215/D216 permission discipline absent explicit new security design.

**Precedent breadth from the research packet:** Erlang/OTP process initialization/message server; Akka Typed initial behavior and explicit spawning; Rust/Tokio exclusively owned resources via tasks/channels; Pony authority capabilities; WASI capability/resource provisioning; Go net/http explicit handler dependency composition; Orleans hosted service/activation patterns. Four materially different design families: message-bearing Actors, single-owner async task/channel, explicit capability systems, and handler/service lifecycle composition. They are architectural precedents, **not** copied APIs, a normative source or independent product validation.

### GITHUB010 matrix (score/5; confidence H/M/L)

Rationale attached to every score. Larger is better. Scores compare *decision fitness*, not implementation quality or measured benchmark output.

| Required dimension | B0 status quo | B1 Root/group | B2 independent grant | C per-Worker grant |
|---|---|---|---|---|
| 1 Correctness/invariants | 3H: safe but no broker | 4M: same isolation, new opt-in | 4M: explicit but new authority model | 2H: broadens Worker authority |
| 2 Protos alignment | 4H: no new abstraction | 5M: ordinary Actor+Group | 4M: explicit capabilities | 2H: couples inbound and outbound |
| 3 Pay for present need | 2H: need not solved | 5M: minimal selected entry | 2M: premature grant framework | 2H: multiplied grants |
| 4 Incremental growth | 3M: current gap remains | 5M: can later add B2 | 4M: scales with more grant forms | 3M: service code shifts into Workers |
| 5 Future options | 4M: no new constraints | 5M: B2 still possible | 4M: fixes delegation design | 2M: commits to per-Worker model |
| 6 Scalability | 2H: no authorized service | 3M: Root serial execution | 5M: separate service domains | 4M: distribute work among Workers |
| 7 Conceptual simplicity | 5H: no change | 4M: one new entry mode | 2M: grant policy/lifetime matrix | 2H: mixed responsibility |
| 8 Portability | 5H: unchanged | 4M: conceptual Actor bootstrap | 3M: host-provisioning backend | 3M: host-dependent per Worker |
| 9 Runtime cost | 5H: zero | 4M: opt-in only | 3M: provisioning and Actors | 3M: resources/Worker |
| 10 Failure/operability | 2H: absent service | 3M: Root fatal impacts Process | 5M: fault isolation | 3M: dispersed egress failures |
| 11 Deferral/reversibility | 2M: blocks immediate use | 5M: Host protocol stable for B2 | 3M: commits prematurely | 2M: later policy migration |
| 12 Evidence/implementation risk | 5H: proven status quo | 4M: existing primitives, entry gap | 3M: precedents, new contract | 2M: new exception matrix |

**Qualitative gate:** B1 carries an acknowledged **underengineering flag** if independent service failure domains, production throughput or runtime-provisioned broker replication become immediate requirements. B2 carries the **overengineering flag** now: a new general grant/custody model without evidence that present work requires it. B0 fails the motivating need. C has a capability-expansion risk. No arithmetic total overrides these constraints.

### Adversarial, future and incremental checks

- **Smallest sufficient today:** one opt-in RootActor entry mode; the GroupRef is already available and does not authorize creation of any privileged service. No `Network`/`Resolver` transfer.
- **Pay-as-needed:** default direct program/CLI/embedding remains unchanged; no universal service registry, routing group, lifecycle manager or policy evaluation allocated for trivial programs.
- **Cost of deferring B2:** when scaling/HA becomes necessary, authorize and instantiate independently provisioned service Actors, replace root-specific application setup and adjust frontend references. D225-A `handler(request)` remains stable; neither ActorRef nor GroupRef retargeting is silently introduced. This is **bounded application migration**, not a promise of zero-cost upgrade.
- **Risk of B1:** RootActor fatal Error kills the Process and all hosted HTTP workers; one Actor-local execution domain can be a bottleneck; a hostile application handler may issue any selector advertised by the root service; workers must receive only a GroupRef, not the RootActor's full ActorRef.
- **Future scenarios:** many Workers and Hosts still share root service only by ordinary messages; no global FIFO/transparent retries, group membership mutation, remote-discovery, cross-Process resource transfer, automatic failover or persistent state. Future independent service Actors require additional authorization design rather than expanding this decision.
- **Bootstrap falsifier:** if bootstrap itself suspends waiting for Host READY while Workers are blocked waiting for the RootActor, the model deadlocks. Do not implicitly dispatch pre-READY messages to fix it. The bootstrap must return its behavior independently; application can await hosting readiness later through ordinary explicit coordination.
- **Cancellation falsifier:** cancelling the HTTP session does not automatically revoke an already accepted RootActor request or committed outbound effect. Late replies are retired under the existing request/session rules.
- **Authority falsifier:** a GroupRef limits authority surface to `send`/`request`, not to chosen selectors. Publishing a generic service `fetch(arbitraryUrl)` is not made safe by group transfer. Root service has to validate input, endpoint and credential use.
- **Embedding falsifier:** the standard Polyglot embedding has no automatic Resolver grant. B1 cannot be treated as proof that public GitHub HTTPS works without separately granted Resolver and D222-C UrlClient implementation.
- **Portability:** host/launcher-specific selection surface needs exact specification; runtime mechanics may differ across JVM/Native/other backends without changing RootActor behavior identity or Process lifetime.

## GITHUB021 approved-invariant reconciliation

| Previously approved rule | Result | Evidence |
|---|---|---|
| D225-A opt-in factory, unchanged D215 handler and D216 Host lifecycle | PASS | B1 uses GroupRef as ordinary factory initialization; changes no Host selector |
| One Worker-local factory invocation, Actor-copy transfer, no resource transfer | PASS | GroupRef transfers as communication capability; original I133 spec §15.13 unchanged |
| D047 `Network`, D223-A1 `Resolver` separately granted, Root-only | PASS | Neither capability crosses Actors; host grants remain explicit |
| Actor bootstrap returns ordinary behavior before READY | PASS | Reuses existing bootstrap representation; does not allow post-READY replacement |
| Core ActorRef retains full `stop` authority | PASS | Application uses GroupRef public communication surface, not ActorRef attenuation |
| No automatic RootActor service, registry, parent lookup or new identity | PASS | Selected mode is **explicit opt-in**; no ambient discovery |
| RootActor fatal failure terminates Process, Actor transfer preserves isolation | PASS | Risk is accepted and testable, not altered |
| No generic Actor Group discovery/membership mutation/rebinding | PASS | Existing `Actor.group(Actor.current())` single-member GroupRef only |
| D222-C route/URL client and TLS/JSON contracts independent | PASS | B1 provides messaging/authority location only |

**DECISION_APPROVAL_PROVENANCE=PASS** based on the exact 2026-10-30 user statement.

**DECISION_INVARIANT_CONSISTENCY=PASS** for the bounded B1 candidate: no approved invariant changed. The initial module/host opt-in entry API requires its own accurate normative formulation, and any materially new public behavior not encompassed by the selected architecture must go back to the owner.

## Ratification and closure gates

```text
WORK_ITEM=D225-B1
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
DURABLE_DECISION_PUBLICATION=PASS_WHEN_PUBLISHED_AND_REREAD
B1_NORMATIVE_SPEC_PRODUCT_COMMIT=NOT_YET_PUBLISHED
B1_IMPLEMENTATION_COMMIT=NOT_YET_PUBLISHED
D225_ISSUE_CLOSURE=NOT_AUTHORIZED_YET
```

The project-documentation publication is durable selected-design/approval evidence only. It must not assign a new normative spec revision, claim executed tests, independently approve CLI flag/API choices, or close D225 before that item's normative publication postconditions are met.
