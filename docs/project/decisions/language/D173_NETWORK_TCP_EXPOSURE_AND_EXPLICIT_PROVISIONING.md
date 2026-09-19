# D173 — Network/TCP foundation exposure and Standard Library integration

Status: **RATIFIED — Candidate A**

Approval date: **2026-09-19**  
Decision issue: `guillermomolina/protos#645`  
Trigger: AUD009-D3 / `guillermomolina/protos#643`  
Protos evidence revision: `5cf9045b0723c80d8cbbfbf2532e33ecdada16a0`  
Project-record base: `fb5082edec99c4e7cff16161ebae9e49c4b07959`

This is a durable non-normative decision record. Observable Protos semantics remain authoritative under `guillermomolina/protos:spec/**`.

## Decision

D173 selects **Candidate A — direct explicit Network-capability composition**.

```text
CORE FOUNDATION

Network capability                               KEEP
Network.connectTcp                               KEEP
Network.listenTcp                                KEEP
TcpConnection                                    KEEP
TcpListener                                      KEEP
TCP half-close                                   KEEP
Future / byte-I/O / lifecycle semantics          KEEP


AUTHORITY

Network authority explicit                       KEEP
ambient Network authority                        KEEP ABSENT

IpAddress / IpEndpoint grants authority           NO
Process grants Network implicitly                 NO
Actor grants Network implicitly                   NO
P grants Network implicitly                       NO

Process.network()                                NOT_ADDED
global/default Network accessor                  NOT_ADDED
import side-effect Network acquisition            NOT_ADDED
service-locator Network                          NOT_ADDED


BOOTSTRAP / PROVISIONING

host may explicitly provision Network             KEEP
initial moduleContext local "network"
when host grants it                               KEEP
"network" slot absent when not granted            KEEP

ordinary application grant by default             NO
explicit launcher/hosting opt-in grant            ADD / ENABLE
exact CLI/package-manifest spelling               DEFERRED_TO_OWNING_CLI_OR_TOOL_WORK


STANDARD LIBRARY

std:network/IpAddresses                           KEEP
std:network/IpEndpoints                           KEEP

std:network/Tcp thin forwarding facade            NOT_ADDED
std:network/Socket universe                       NOT_ADDED
generic Client / Server wrappers                  NOT_ADDED_NOW

future library with real additional semantics
may accept Network explicitly                     ALLOWED
library may retain explicitly supplied Network
inside ordinary encapsulation                     ALLOWED
library may discover Network implicitly            NO


FIRST USEFUL TCP LAYER

application + std:network numeric helpers
    + explicit Network
    + Core TcpConnection/TcpListener              SUFFICIENT
```

All broader networking facilities remain absent unless independently justified.

## Why no Standard Library TCP facade

LIB005 deliberately selected `std:network/IpAddresses` and
`std:network/IpEndpoints` as a data/representation layer and explicitly kept
TCP acquisition/resource semantics in Core.

D173 preserves that architecture.

A module whose complete behavior is:

```text
Tcp.connect(network, endpoint)
    -> network.connectTcp(endpoint)

Tcp.listen(network, request)
    -> network.listenTcp(request)
```

would add a second public spelling and module identity without adding authority,
lifecycle, cancellation, recovery, connection policy, framing or resource
custody semantics.

D173 therefore does not create a Standard Library TCP facade merely to mirror
Core.

This is consistent with the E4 correction that thin modules may be worth keeping
when they are deliberately selected growth seams. In this case the prior
LIB005 decision selected the opposite boundary: TCP remains Core and the initial
library intentionally does not create a second socket/TCP universe.

A future library module becomes justified when it owns additional semantics, not
merely forwarding. Resolver, TLS and HTTP are examples of domains that may later
qualify, each under its own authority and lifecycle design.

## Explicit authority remains the central invariant

Possession of numeric address data remains distinct from authority to perform
network effects.

```text
IpAddress / IpEndpoint
    -> authority-free data

Network
    -> live host-provisioned authority capability
```

Importing a module, possessing Process, creating an Actor or creating P work
does not manufacture Network.

A future library that needs live networking must receive a Network capability
explicitly, or another separately standardized narrower capability. It may retain
that supplied capability privately inside ordinary objects, but it may not
discover ambient authority through Process, globals, import side effects or a
service locator.

## Provisioning boundary

The existing normative model already defines the semantic grant:

```text
host grants Network
    -> initial moduleContext has local slot "network"

host does not grant Network
    -> local slot "network" is absent
```

The current runtime already supports creation of a concrete host Network through
`ProtosPolyglotRuntimeHost.provisionHostNetwork(prelude)`, and
`ProtosStandaloneProcessBootstrap` already supports an optional default Network.

The remaining product gap is entry-path plumbing: ordinary application execution
paths currently construct Processes without requesting/providing that optional
Network.

D173 authorizes enabling an **explicit opt-in** grant through host/application
entry machinery.

It does not authorize always granting unrestricted Network to every application.

The exact user-facing selection syntax — CLI switch, package/application
declaration, embedding API configuration, or another owning surface — is not
selected by D173 and must be owned by the appropriate CLI/Tool/application
configuration work.

## No ambient-by-default fallback

The following is rejected:

```text
every application automatically receives host Network
```

Even though the value would still be an explicit object, always injecting it
would make network authority practically ambient at application bootstrap.

The selected model remains:

```text
no grant
    -> no network slot

explicit grant
    -> explicit network capability in bootstrap-local context
```

## First useful real-network composition

No additional Core or Standard Library network institution is required to build
the first numeric TCP clients/servers.

Conceptually:

```text
IpEndpoints.parse(...)
    -> IpEndpoint

explicit Network
    -> connectTcp(endpoint)
    -> Future<TcpConnection>
```

and:

```text
explicit Network
    -> listenTcp(localRequest)
    -> Future<TcpListener>

TcpListener.accept()
    -> Future<TcpConnection>
```

Existing Future commitment/cancellation, byte-flow, close, endpoint observation
and half-close semantics remain authoritative.

## Deliberate absences retained

```text
DNS / Resolver                       ABSENT KEEP
Happy Eyeballs                       ABSENT KEEP
UDP                                  ABSENT KEEP
TLS / QUIC                           ABSENT KEEP
HTTP / WebSocket                     ABSENT KEEP
NetworkInterface                     ABSENT KEEP
generic socket options               ABSENT KEEP
timeouts/deadlines                   ABSENT KEEP
service discovery                    ABSENT KEEP
Unix/raw sockets                     ABSENT KEEP
active-socket Actor delivery         ABSENT KEEP
resource proxying across Actors      ABSENT KEEP
```

D173 finds none of these is a prerequisite for the first useful numeric TCP
client/server layer.

In particular, DNS must not be smuggled into endpoint parsing or
`connectTcp`. A future Resolver may independently map names to one or more
endpoint values and then compose with explicit Network acquisition.

## Comparative evidence

The research compared several approaches.

- Rust, Go and Java demonstrate that low-level TCP connect/listen resources can
  be useful directly without requiring a higher Client/Server framework.
- Pony demonstrates explicit networking authority/capability objects at
  connection/listener acquisition boundaries.
- WASI demonstrates host-controlled capability provisioning at component
  instantiation rather than implicit global authority.
- mature networking libraries also demonstrate the coupling cost of combining
  hostname resolution, connection policy and socket acquisition under one
  convenience API.

The selected Protos model retains host provision plus explicit capability
possession while keeping name resolution and higher protocols separate.

## Candidate result

### Candidate A — direct explicit Network capability composition

**Selected.**

Keeps the already-retained Core TCP foundation directly usable, preserves
explicit authority, retains the existing Standard Library numeric modules, and
enables opt-in application provisioning without a second socket API.

### Candidate B — thin `std:network/Tcp` wrapper

Rejected for now.

It adds no semantics beyond forwarding to Core and conflicts with the deliberate
LIB005 boundary that TCP remains Core.

### Candidate C — higher-level Standard Library Client/Server layer now

Rejected.

There is no concrete present contract for connection policy, server-loop
ownership, retry policy, pooling, framing or other behavior that would justify
such a layer.

### Candidate D — Process/global/default Network discovery

Rejected.

It weakens the explicit authority model and creates practical ambient authority
or a service-locator institution.

## Twelve-dimension result

The decision considered:

1. correctness/invariants;
2. Protos alignment;
3. present-need proportionality;
4. incremental growth;
5. future-option resilience;
6. scalability;
7. conceptual simplicity;
8. portability/implementation freedom;
9. runtime/resource cost;
10. failure/operability;
11. deferral/reversibility/migration;
12. evidence maturity/implementation risk.

Candidate A was strongest because nearly all required semantics and runtime
machinery already exist. The unresolved gap is provisioning integration, not TCP
API design.

## Strongest argument against Candidate A

A `std:network/Tcp` module created now could become a stable future home for
connection policy, retries, resolver/Happy-Eyeballs integration and server
helpers, reducing later call-site migration.

D173 rejects pre-creating that seam because those concerns are not necessarily
one semantic domain. Resolver, connection policy, TLS and HTTP carry distinct
authority/lifecycle questions, while a future library can wrap existing Core
calls without changing their meaning.

## D172 boundary

D172 / #644 independently owns the long-term Core-vs-Standard-Library placement
of numeric `IpAddress` / `IpEndpoint` values.

D173 does not decide D172.

Whatever placement D172 ratifies must preserve the numeric endpoint capability
that D173 composes with. D173 requires only that live TCP acquisition consume
the retained endpoint semantics without granting authority through the data
value itself.

```text
D172_AUTHORITY_PRESERVED=PASS
D173_DOES_NOT_SELECT_IP_VALUE_PLACEMENT=PASS
```

## Implementation consequence

D173 requires bounded application/hosting plumbing, not a new network model.

The implementation owner must:

1. preserve `Network.connectTcp`, `listenTcp`, `TcpConnection`,
   `TcpListener`, half-close and all existing I/O semantics;
2. preserve the existing bootstrap rule that `network` exists only when the
   host explicitly grants it;
3. preserve default absence of Network authority;
4. allow application/hosting entry machinery to carry an explicitly selected
   optional Network grant to the existing Process/bootstrap mechanism;
5. reuse `ProtosPolyglotRuntimeHost.provisionHostNetwork` and existing
   `ProtosStandaloneProcessBootstrap` semantics rather than inventing a second
   authority type;
6. keep tools/preflight/test Processes networkless unless their owning entry
   path explicitly grants Network;
7. introduce no `Process.network()`, global accessor, service locator, import
   side effect or implicit capability propagation;
8. introduce no `std:network/Tcp`, Socket, Client or Server facade;
9. introduce no DNS/TLS/HTTP/UDP/deadline/socket-option feature;
10. route exact user-facing CLI/package/configuration spelling through the owning
    CLI/Tool work rather than deciding it inside D173.

```text
D173_STATUS=RATIFIED
SELECTED_CANDIDATE=A

CORE_NETWORK=KEEP
NETWORK_CONNECT_TCP=KEEP
NETWORK_LISTEN_TCP=KEEP
TCP_CONNECTION=KEEP
TCP_LISTENER=KEEP
TCP_HALF_CLOSE=KEEP

EXPLICIT_NETWORK_AUTHORITY=KEEP
AMBIENT_NETWORK_AUTHORITY=ABSENT_KEEP
PROCESS_NETWORK_ACCESSOR=NOT_ADDED
GLOBAL_NETWORK_ACCESSOR=NOT_ADDED

HOST_NETWORK_PROVISIONING=KEEP
BOOTSTRAP_LOCAL_NETWORK_SLOT=KEEP
NETWORK_SLOT_ABSENT_WITHOUT_GRANT=KEEP
DEFAULT_APPLICATION_NETWORK_GRANT=NO
EXPLICIT_APPLICATION_NETWORK_GRANT=ENABLE

STDLIB_IP_ADDRESSES=KEEP
STDLIB_IP_ENDPOINTS=KEEP
STDLIB_TCP_FORWARDING_FACADE=NOT_ADDED
HIGH_LEVEL_CLIENT_SERVER=NOT_ADDED_NOW

D172_INDEPENDENT=PASS
NORMATIVE_NETWORK_MODEL_CHANGE=NO
IMPLEMENTATION_PLUMBING_REQUIRED=YES

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Approval provenance

The exact Candidate A boundary was presented to the project owner in the active
interaction on 2026-09-19.

The project owner explicitly approved it:

```text
aprobada
```
