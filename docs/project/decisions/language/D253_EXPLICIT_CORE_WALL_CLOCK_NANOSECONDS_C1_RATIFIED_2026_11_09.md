# D253-C1 — Explicit Core wall-clock nanoseconds capability

**Status: OWNER-RATIFIED DESIGN (2026-11-09); PRODUCT NORMATIVE SPECIFICATION AND IMPLEMENTATION NOT YET PUBLISHED.** This is a durable **non-normative** design-decision record. Observable Protos authority is established only by the applicable product `spec/` publication, not by this file or a GitHub label.

- Decision: [D253 #1101](https://github.com/guillermomolina/protos/issues/1101).
- Downstream consumer: [I153 RealWorld #1068](https://github.com/guillermomolina/protos/issues/1068).
- Allocated implementation: [I163 #1102](https://github.com/guillermomolina/protos/issues/1102), `status:ready`; native parent relation to D253 #1101 **pending** (read-only GitHub REST returned no parent/sub-issue). Product specification and runtime still unpublished.
- Existing JWT contract: [D246-C / I161](https://github.com/guillermomolina/protos/issues/1094), [`spec/auth/JWT.md`](https://github.com/guillermomolina/protos/blob/9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd/spec/auth/JWT.md).
- Inspected **PROTOS_REVISION=`9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd`** (`0.3.458-SNAPSHOT`), before Clock.
- Inspected RealWorld revision `d242b73e6bd327739147bfc8ce85a789b4384533`; RootActor owns the domain state and authenticated HTTP operations are not yet enabled.
- Comparative research, 12-factor scoring, tests, falsification and GITHUB021 consistency: [D253 evidence](../../evidence/D253/D253_C1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_09.md).
- **Owner approval provenance:** in the active 2026-11-09 D253 conversation, following the revised exhaustive comparison and the exact C1 contract, the project owner said **“apruebo D253-C1”**, and authorized GitHub issue updates and direct commit/push of durable evidence in `guillermomolina/protos-project-docs`. This is explicit approval of **C1 as presented**, not the earlier B1 recommendation or unpresented supplementary clock features.

## 1. Selected model and public boundary

**D253-C1 — Explicit Core Wall-Clock Nanoseconds Capability.** A genuine, independently host-provisioned `Clock` capability exposes exactly the initial operation:

```protos
clock.epochNanoseconds()  // synchronous exact semantic Integer
```

This is an authorized read of **system/wall-clock time**, in mathematical Integer nanoseconds since `1970-01-01T00:00:00Z` on the same uniform Unix-like timeline as the existing `std:datetime/instant` family. Negative instants are valid. A standard Integer result is unbounded by the host's signed-64 nanoseconds range; two-part seconds/nanoseconds conversion must be exact, must never pass through Float, and must not silently overflow, clamp, fabricate sub-resolution precision, or coerce incorrect host values.

The capability's ordinary source-backed `Clock` protocol belongs at the Core semantic boundary; only the narrow authorized host-read primitive belongs in the host runtime. The protocol prototype is authority-free. Obtaining it, importing a module, constructing/delegating an ordinary lookalike, or possessing Process/Filesystem/Network/Resolver/Entropy grants does **not** mint a Clock grant. Authenticity and liveness are checked against the host grant, not by apparent shape or the ordinary delegation chain alone.

There is no Core operation that returns a Standard Library `Instant` directly. An explicitly authorized caller composes the already-existing standard constructor:

```protos
Instant: import("std:datetime/instant")
now: Instant(clock.epochNanoseconds())
```

This avoids making the earlier Core bootstrap depend on a later Actor-local `std:datetime` MODULE and leaves any future convenience facade `std:datetime/clock` unimplemented. It does not change `Instant` construction, identity, equality, hash, serialization, transfer rules, or the `std:datetime` library contract.

## 2. Authority, placement and lifetime

- The host grants Clock **independently and optionally**, installing a local slot named `clock` before the initial RootActor SOURCE or MODULE expression; without authority the slot is **absent, not `null`**. This applies equally to direct-entry and explicit RootActor message-service bootstrap.
- The slot is private bootstrap-local state, not a Prelude name, global, public MODULE export, import side effect, registry, or `Process` accessor. Imported modules, later host entries and newly spawned Actors do not inherit it.
- The standard standalone CLI must support the explicit, independent `--allow-clock` selection; embedding provisions the grant through a deliberate host/Context authorization. Exact embedding plumbing remains an implementation/specification integration detail only insofar as it does not create additional public semantics.
- A concrete Clock grant is initially **neither Actor-transferable/delegable nor P-transferable**. Nested graphs reaching the grant observe ordinary `NonTransferableValue` / `NonParallelValue`. The returned exact Integer is ordinary transferable data.
- One grant is bounded to its provisioning host and at most one Process incarnation; Process/Context termination or host revocation makes that authority unusable. No guest-visible `close()` operation is added in C1, and the absence of `close()` must not prevent host retirement.
- A RootActor may broker sampled Integer values to Workers via normal Actor messaging. This is explicit data provision, not authority delegation or a promise of immediate/fresh delivery.

## 3. Execution, errors and trust

- `epochNanoseconds()` is **synchronous**, has no arguments, and returns an ordinary semantic Integer on success. It creates no Future, Task, scheduler operation, hidden suspension, cancellation protocol, or asynchronous close obligation.
- Ordinary message/arity failures retain ordinary invocation semantics. Invalid or forged receivers cannot trigger the host effect. A host clock failure is an ordinary synchronous failure mapped to the established `IOError` category where applicable; revoked/retired grant use signals the established `IOLifecycleError` category. No fallback to `0`, a fixed timestamp, `null`, Float, or a reused stale success is permitted.
- Wall clock readings are **not monotonic**. Two observations may be equal or decrease, including from NTP adjustment, administrator changes, VM suspension/recovery, virtualization or cross-node skew. The operation provides no anti-rollback, tamper-proof, signed-time, global ordering, UTC accuracy or nanosecond *physical resolution* guarantee beyond its authorized host/provider.
- Deterministic tests may use a trusted host-provisioned clock source; an arbitrary Protos module cannot forge a Clock authority. The decision does not promote a test injection hook to a guest grant factory.
- JVM and rebuilt Native CLI must provide the **same language-visible behavior**, but need not return equal samples or rely on the same syscall. A minimal Java 25 host abstraction (for example, an InstantSource-like provider) is an implementation precedent, not a Protos-facing Java API and not approval of FFM or a Linux-only clock backend.

## 4. JWT, HTTP and logging integration

D246-C JWT remains synchronous, host-authority-free and caller-supplied: `JWT.issue(claims, key, policy, now)` and `JWT.verify(token, key, policy, now)` take a genuine `std:datetime/instant` value with exact Integer nanosecond arithmetic. Clock **does not change either JWT API or acquire any authority on JWT import**. RealWorld's RootActor constructs its explicit `Instant(clock.epochNanoseconds())` and supplies it to JWT. A valid `Instant` proves its *type*, not that the caller obtained it from a secure source; correct policy and trusted host time remain application/deployment obligations.

The existing `std:logging/logger` zero-argument time-source callback may use `() => { Instant(clock.epochNanoseconds()) }` when the authorized caller deliberately supplies one. No logging import gains ambient time authority.

C1 does **not** satisfy `spec/io/HTTP.md`'s separate security requirement for portable deadlines/timeouts/Future-race/slow-client bounds. A civil clock must not be used as a monotonic deadline primitive. This decision makes no claim of production-safe Internet-facing HTTP hosting.

## 5. Exclusions, growth and strongest objection

**Deferred on purpose:** monotonic clocks; timer/sleep/deadline APIs; Future-returning clock reads; remote clocks; public transfer/delegation and attenuation; `close()`; wall-clock trust attestation and rollback-resistance; time-zone/calendrical services; extra clock error taxonomy; `std:datetime/clock` facade. Future independent contracts may add these without changing C1's exact Integer or Core/Standard Library separation.

**Pay for what is used:** the unused capability adds no guest time reading, no workers, no time service, no scheduler dependency or per-Actor grants. **Grow as needed:** an `Instant` convenience facade is purely additive; a monotonic capability needs its own domain; Actor-safe grants need a separately approved delegation/custody/placement contract. **Cost of deferral:** an app currently writes `Instant(clock.epochNanoseconds())`; no normative `Instant` replacement or JWT API migration is required later.

**Strongest objection:** direct `clock.now() -> Instant` is more ergonomic, and returning an untyped numeric unit might invite misuse. C1 accepts the explicit construction to preserve Core's lower dependency layer. The operation's `epochNanoseconds` spelling, exact Integer result and focused examples make the unit visible. The alternative `std:datetime/clock.now(clock)` remains a bounded additive option.

## 6. GITHUB021 owner-invariant consistency and publication boundary

Applicable prior invariants were checked before ratification:

| Previously approved owner/design invariant | C1 result |
| --- | --- |
| D243-C: Entropy is independent, host-granted, bootstrap-local and not obtained by import or Process | **PRESERVED**; Clock is separate and does not grant Entropy |
| D246-C: JWT imports no time/entropy/keys; caller supplies authentic policy/key/`Instant` | **PRESERVED**; JWT signature unchanged |
| D251-A1: Actor initialization suspension follows explicit Future/Task semantics | **PRESERVED**; Clock reads synchronously |
| MODULES/ACTORS: private initial-module slots, independent Actor contexts and explicit authority transfer | **PRESERVED**; Clock is bootstrap-local and initially non-transferable |
| `Instant` remains `std:datetime` value, with no ambient `now()` | **PRESERVED**; conversion is explicit |
| D253 user correction: C1 instead of earlier B1, exact Integer returned | **PRESERVED**; no implicit Core `Instant` dependency |

No pre-existing D253 owner-approved invariant is known to conflict with the selected C1. The packet identifies the exact change from the **unapproved** earlier B1 proposal; no B1 assumption is incorporated into approval. **DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS for the presented C1.** This does not ratify further embedding APIs, transferable Clock, hidden monotonic behavior, or future timer contracts.

**Publication separation:** this file ratifies the *design only*. At `PROTOS_REVISION=9c92dbc0cfdfbfe5cb2751339ea27297c8c22bcd`, no `spec/io/CLOCK.md`, Core Clock source, `--allow-clock` public implementation, JVM/Native Clock tests or RealWorld integration is claimed. The separate **I-family** implementation issue must incorporate the normative product specification, source-backed protocol/host bridge, CLI/embedding grants, conformance and portability tests, docs/examples, version/changelog and human-run validation before any implementation acceptance.

**Tests/validation in this decision-only publication: NOT RUN.** No execution, compiled artifact, runtime parity or installed Native success is inferred from design approval.
