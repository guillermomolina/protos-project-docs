# AUD009-D3 — Network, IP endpoint, TCP, and half-close complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#643`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline:
`927f530f925f241bf6b1ec7eef7832bb4b18f674`

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Initial checkpoint proposal: `guillermomolina/protos#643`, issue comment
`5739768350`.

Owner-approved corrected classification: `guillermomolina/protos#643`, issue
comment `5739793039`, 2026-09-19.

Derived decision routes:

- `D172 / guillermomolina/protos#644` — retained numeric IP value placement
  and ownership boundary.
- `D173 / guillermomolina/protos#645` — retained Network/TCP foundation
  exposure and Standard Library integration.

Neither derived decision is selected by this audit record.

## Purpose and correction history

AUD009-D3 reviewed the portable networking layer:

- numeric IPv4/IPv6 and endpoint values;
- Network authority;
- TCP connect/listen acquisition;
- TcpConnection and TcpListener;
- TCP half-close;
- deliberately absent broader networking facilities.

The initial checkpoint proposed retaining numeric IP capability and explicit
authority as principles while removing the current Core placement and live
Network/TCP institution.

The project owner rejected that removal-oriented conclusion because the
already-built networking foundation is deliberate reusable capability for
near-term client/server work. Lack of a current `.protos` consumer was judged
insufficient evidence for throwing away a coherent, implemented and tested
foundation that the project expects to build on.

After re-evaluation, the project owner explicitly approved the corrected
classification recorded below.

AUD009-D3 itself authorizes no normative or implementation change.

## Evidence summary

### Numeric IP data has a current Standard Library consumer

Current production Standard Library code uses the retained numeric value
families through:

```text
std:network/IpAddresses
std:network/IpEndpoints
```

The numeric values are authority-free and useful independently of live network
I/O.

### Live TCP currently has no Protos-source consumer

Repository evidence at the checkpoint found no production `.protos` consumer
of:

```text
Network.connectTcp
Network.listenTcp
TcpListener.accept
TcpConnection.localEndpoint
TcpConnection.remoteEndpoint
shutdownRead
shutdownWrite
```

Ordinary product launch paths also did not provision Network by default.

That evidence remains true as evidence about **current exposure/use**.

It no longer supports a removal classification by itself.

### Explicit authority remains a positive property

The ordinary product paths using `defaultNetwork = null` demonstrate that
network access is not ambient.

The retained model intentionally separates:

```text
having an address
    !=
having authority to communicate
```

This is a security/authority property, not evidence that the Network capability
is unnecessary.

### Deferral cost is concrete

The live Network/TCP implementation already provides a coherent privileged
foundation for:

- explicit network authority;
- connect and listen acquisition;
- listener and connection resource families;
- Future/I/O commitment and cancellation integration;
- byte readable/writable resource exposure;
- endpoint observation;
- close and half-close;
- transfer restrictions and host/runtime integration.

Removing it would require later rebuilding or recovering substantial privileged
machinery before a TCP client, TCP server or later HTTP library could be built.

AUD009's anti-simplification rule therefore applies: current low use does not
justify deleting valuable encapsulated capability when reintroduction cost is
concrete and the project expects near-term use.

## Final classification ledger

```text
numeric IPv4/IPv6 + endpoint capability    KEEP
explicit/non-ambient network authority     KEEP

canonical IpAddress / IpEndpoint families  KEEP (D172: public ownership moves to std:network)
Prelude IpAddress / IpEndpoint bindings     REMOVE (D172)
native recognizes/equality machinery        KEEP

Core Network capability                    KEEP
Network.connectTcp                         KEEP
Network.listenTcp                          KEEP
TcpConnection / TcpListener                KEEP
TCP half-close                             KEEP
```

No D3 mechanism is classified `REMOVE_NOW_RECONSIDER_LATER` or
`REMOVE_PERMANENTLY`.

## Approved authority invariants

```text
having an address != having authority

Process / Actor / P do not grant Network implicitly

live networking requires explicit Network authority

possession of IpAddress / IpEndpoint alone does not authorize network I/O
```

These invariants constrain later D172/D173 work unless explicitly reopened
through the owner gate.

## Numeric IP / endpoint values remain

The retained capability includes:

```text
numeric IPv4/IPv6 distinction
exact integer address bits
numeric endpoint = address + non-zero port
authority-free parsing/formatting
IPv6 scope/zone not part of portable address identity
no implicit IPv4-mapped-IPv6 normalization
no DNS hidden in numeric parsing
```

The recognition/equality/hash/transfer machinery remains retained.

D172 / #644 subsequently selected Candidate C: the canonical frozen
IpAddress/IpEndpoint family objects and required private runtime support remain,
while their unqualified Prelude bindings move to the retained
`std:network/IpAddresses` and `std:network/IpEndpoints` public domain.

This changes placement, not the D3 KEEP classification of the numeric capability.

Classification: **KEEP**.

## D172 ratification reconciliation

D172 / #644 selected **Candidate C**.

```text
numeric IpAddress/IpEndpoint capability       KEEP
canonical frozen family prototypes            KEEP
recognizes/equality/hash/transfer support      KEEP

Prelude.IpAddress                             REMOVE
Prelude.IpEndpoint                            REMOVE

std:network/IpAddresses.IpAddress             ADD / canonical exposure
std:network/IpEndpoints.IpEndpoint            ADD / canonical exposure
```

D172 does not reinterpret the original D3 owner-approved KEEP of the numeric
capability as removal. It narrows only public placement. The retained runtime
substrate must not become an IP-specific second module/transfer system.

Implementation migration is owned by I066 / `guillermomolina/protos#669`.

Durable decision record:

`docs/project/decisions/language/D172_NUMERIC_IP_VALUE_PLACEMENT_AND_OWNERSHIP.md`

## Network authority remains

The current Network capability is retained as the explicit authority required
for live network operations.

The absence of ambient provisioning is part of the design's value rather than a
reason to remove it.

Classification: **KEEP**.

## TCP connect/listen and live resources remain

The current TCP foundation remains available:

```text
Network.connectTcp
Network.listenTcp
TcpConnection
TcpListener
TcpListener.accept
connection endpoint observation
resource custody/lifecycle integration
```

The project expects this foundation to support future `std:network` client and
server facilities and later higher-level protocol work.

Classification: **KEEP**.

## TCP half-close remains

Once TcpConnection is retained, removing its directional shutdown semantics
would leave an unnecessarily incomplete TCP resource.

```text
TcpConnection.shutdownRead()   KEEP
TcpConnection.shutdownWrite()  KEEP
```

Classification: **KEEP**.

## Deliberate absences remain absent

D3 does not use retention of TCP as justification to prebuild a wider networking
stack.

```text
DNS / Resolver                         ABSENT / RETAIN ABSENCE
Happy Eyeballs                         ABSENT / RETAIN ABSENCE
UDP/datagrams                          ABSENT / RETAIN ABSENCE
NetworkInterface enumeration           ABSENT / RETAIN ABSENCE
network policy/introspection algebra   ABSENT / RETAIN ABSENCE
TLS                                    ABSENT / RETAIN ABSENCE
QUIC                                   ABSENT / RETAIN ABSENCE
HTTP / WebSocket                       ABSENT / RETAIN ABSENCE
Unix-domain sockets                    ABSENT / RETAIN ABSENCE
raw sockets                            ABSENT / RETAIN ABSENCE
service discovery                      ABSENT / RETAIN ABSENCE
generic socket options                 ABSENT / RETAIN ABSENCE
socket-local deadlines/timeouts        ABSENT / RETAIN ABSENCE
active-socket Actor mailbox delivery   ABSENT / RETAIN ABSENCE
Network/TCP Actor proxy transfer       ABSENT / RETAIN ABSENCE
```

These are future questions only when concrete requirements justify them.

## Required AUD009 routing

```text
FEATURE=retained numeric IP value placement/layering
CURRENT_OUTCOME=STDLIB_PUBLIC_OWNERSHIP_WITH_PRIVATE_CANONICAL_SUBSTRATE
SEMANTIC_DESIGN_OWNER=D172 / guillermomolina/protos#644 RATIFIED
IMPLEMENTATION_OWNER=I066 / guillermomolina/protos#669

FEATURE=retained Network/TCP exposure and stdlib integration
PROPOSED_OUTCOME=KEEP
SEMANTIC_DESIGN_OWNER=D173 / guillermomolina/protos#645
IMPLEMENTATION_OWNER=TBD_AFTER_D173_IF_ANY_CHANGE_IS_APPROVED
```

These routes are evolution questions, not removal routes.

## Boundary handoffs

- **D172 / #644** owns numeric IP value placement/layering while preserving
  functionality.
- **D173 / #645** owns how Standard Library/application code exposes and
  provisions the retained Network/TCP foundation.
- **AUD009-E** owns Standard Library breadth and API necessity.
- **AUD009-G** owns runtime/backend architecture not itself part of portable
  networking semantics.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#643
INITIAL_CHECKPOINT_COMMENT=5739768350
CORRECTED_APPROVAL_COMMENT=5739793039
DATE=2026-09-19

NUMERIC_IP_ENDPOINT_CAPABILITY=KEEP
NUMERIC_IP_FAMILIES=KEEP
PRELUDE_IP_FAMILY_BINDINGS=REMOVE_BY_D172
PRIVATE_CANONICAL_IP_SUBSTRATE=KEEP
IP_NATIVE_RECOGNITION_EQUALITY=KEEP

EXPLICIT_NETWORK_AUTHORITY=KEEP
CORE_NETWORK_CAPABILITY=KEEP
NETWORK_CONNECT_TCP=KEEP
NETWORK_LISTEN_TCP=KEEP
TCP_CONNECTION_LISTENER=KEEP
TCP_HALF_CLOSE=KEEP

D172=guillermomolina/protos#644
D173=guillermomolina/protos#645

NORMATIVE_CHANGE_AUTHORIZED_BY_D3=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_D3=NO
```

## Closure checklist

```text
OWNER_APPROVAL_PROVENANCE=PASS
EVIDENCE_BASELINE=927f530f925f241bf6b1ec7eef7832bb4b18f674

D172_NATIVE_PARENT=#643 PASS
D173_NATIVE_PARENT=#643 PASS

REMOVAL_ROUTES=NONE
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_D3_CLASSIFICATION=COMPLETE
AUD009_D=COMPLETE
```

AUD009-D3 and the top-level AUD009-D partition are complete.
