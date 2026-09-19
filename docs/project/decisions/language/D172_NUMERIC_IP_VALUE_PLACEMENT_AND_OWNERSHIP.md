# D172 — Numeric IP value placement and ownership boundary

Status: **RATIFIED — Candidate C (Standard Library public ownership plus canonical private substrate)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#644`
Trigger: AUD009-D3 / `guillermomolina/protos#643`
Protos evidence revision: `5cf9045b0723c80d8cbbfbf2532e33ecdada16a0`
Project-record base: `295c82532a9c479b796e07b1d6632019cbcff0e0`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D172 selects **Candidate C**.

The numeric IP/endpoint capability remains fully supported, but its long-term
public ownership moves from universal Core Prelude bindings to the Standard
Library networking domain.

Selected boundary:

```text
PUBLIC PRELUDE

IpAddress                         REMOVE
IpEndpoint                        REMOVE

PUBLIC STANDARD LIBRARY

std:network/IpAddresses           KEEP
    canonical IpAddress           EXPOSE
    v4                            KEEP
    v6                            KEEP
    parse                         KEEP
    format                        KEEP

std:network/IpEndpoints           KEEP
    canonical IpEndpoint          EXPOSE
    parse                         KEEP
    format                        KEEP

PRIVATE STANDARD/RUNTIME SUBSTRATE

canonical frozen IpAddress prototype       KEEP
canonical frozen IpEndpoint prototype      KEEP
exact recognition support                  KEEP
structural equality/hash support           KEEP
Actor/P transfer support                   KEEP
runtime Network/TCP consumption            KEEP
```

The public move does **not** convert the values into host-branded Java value
classes, shape-only records, Actor-local module families, or non-transferable
library objects.

## Preserved semantic contract

D172 preserves the numeric data model ratified by D047/D048 and retained by
AUD009-D3:

```text
IpAddress:
    version = Integer 4 or 6
    bits    = exact Integer address bits

IpEndpoint:
    address = recognized IpAddress
    port    = Integer 1..65535
```

The following remain unchanged:

- IPv4 and IPv6 are semantically distinct;
- no implicit IPv4-mapped-IPv6 normalization;
- no hostname or DNS behavior in numeric parsing/construction;
- no interface/scope identity inside portable IpAddress;
- address/endpoint values carry no Network authority;
- construction yields fresh ordinary frozen Protos identities;
- standard family membership requires the canonical immediate parent and exact
  canonical state;
- `recognizes` remains callback-free and host/network-effect-free;
- equality/hash remain structural over canonical numeric state;
- independently constructed equal values need not be identical under `===`;
- Map-key behavior remains consistent with standard equality/hash;
- Actor/P transfer preserves the numeric value contract without transferring
  Network authority; and
- Network/TCP may continue consuming the same canonical numeric values.

## D048 relationship

D048 remains semantic authority for construction, recognition and ordinary-object
representation except for one placement consequence explicitly reopened by
D172.

D048 selected:

```text
canonical frozen IpAddress factory/prototype
canonical frozen IpEndpoint factory/prototype
ordinary frozen instances
exact immediate canonical parent
exact own state
recognizes
structural equality/hash
```

D172 **keeps all of those semantics**.

D172 supersedes only the requirement that the canonical factory/prototypes be
published as unqualified Core Prelude bindings.

Therefore:

```text
D048_CONSTRUCTION_MODEL                     KEEP
D048_RECOGNITION_MODEL                      KEEP
D048_ORDINARY_OBJECT_MODEL                  KEEP
D048_EQUALITY_HASH                          KEEP
D048_PRELUDE_PUBLICATION                    SUPERSEDED_BY_D172
```

## Why a pure source-level module is insufficient

Standard modules are Actor-local module instances.

If a canonical IP family were naively created as ordinary module-local source
state:

```text
Actor A:
    IpAddressPrototype_A
        init closure
        recognizes closure
        == closure
        hash closure
```

an IpAddress instance would delegate to that Actor-local prototype.

Ordinary Actor/P transfer recursively preserves delegation structure. Copying
such a value would therefore have to transfer or reconstruct the module-local
prototype and its behavior closures.

Closures are deliberately non-transferable across the relevant isolation
boundaries, and independently reconstructed module prototypes would not preserve
D048's exact canonical-parent recognition identity.

A source-only relocation would therefore either:

- break transfer;
- change canonical family identity;
- weaken recognition to shape-only;
- introduce module-import behavior into transfer; or
- create a second special transfer model.

Candidate C rejects all of those outcomes.

## Canonical private substrate

The canonical IpAddress/IpEndpoint factory/prototypes remain frozen standard
objects available to runtime/standard-library integration even though they are no
longer universal Prelude names.

Their exact implementation representation is private.

A conforming implementation may physically share the canonical frozen standard
objects where D049 permits it, rematerialize equivalent implementation state
where semantics allow, or use another bounded mechanism, provided the observable
D048/D172 contract remains unchanged.

The implementation must **not** create IP-specific module identity, IP-specific
import-during-transfer, a second module cache, or a new public semantic value
category merely to perform the relocation.

If a clean general Standard-Library/runtime support seam is unavailable, the
implementation must stop and route the architectural conflict rather than
inventing a special IP-only subsystem.

D167/I060 creates analogous pressure for Standard Library facilities backed by
private runtime support. D172 permits reuse of a general mechanism established
there or elsewhere, but does not itself ratify one particular loader/runtime
architecture.

## Standard Library exposure

D172 assigns public ownership to the existing retained networking Standard
Library domain.

Conceptually:

```text
IpAddresses: import("std:network/IpAddresses")
IpEndpoints: import("std:network/IpEndpoints")

IpAddresses.IpAddress
IpEndpoints.IpEndpoint
```

Those exposed values must designate the canonical standard factory/prototypes,
not Actor-local replacement families.

The existing convenience surface remains:

```text
IpAddresses.v4(...)
IpAddresses.v6(...)
IpAddresses.parse(...)
IpAddresses.format(...)

IpEndpoints.parse(...)
IpEndpoints.format(...)
```

D172 does not require a specific hidden loader injection spelling. The
implementation may choose the smallest general mechanism consistent with module
semantics and the standard-library/runtime boundary.

## Why Candidate A is not selected

Keeping the current Prelude bindings is semantically sound and operationally
cheap.

It has strong precedent: some mature systems place numeric networking values in
very low-level/core libraries, and the current Protos implementation is already
coherent, tested and pay-for-use.

However, the public Prelude is a universal language surface. Numeric IP data is
a networking-domain institution whose parse/format/convenience growth already
has an approved Standard Library home.

Keeping the first two networking data names global would establish pressure to
promote later domain values merely because Core runtime consumes them.

D172 therefore separates:

```text
runtime must understand the canonical value
        !=
every program must receive an unqualified binding for it
```

## Why Candidate B is not selected

Candidate B moves ownership to ordinary source-only Standard Library prototypes.

It is rejected because current module/Actor semantics make that relocation
incompatible with the retained exact-parent recognition and transfer guarantees
without adding substantially more machinery or weakening semantics.

## Why Candidate D is not selected

No useful alternative hybrid justifies semantic loss.

In particular, hiding the canonical factory/prototypes entirely and retaining
only `v4`/`v6`/parse helpers would remove D048's direct numeric construction,
recognition and transparent ordinary-object family surface without producing a
meaningful implementation simplification.

## Approval provenance

AUD009-D3 explicitly retained:

```text
numeric IPv4/IPv6 + endpoint capability    KEEP
Core IpAddress / IpEndpoint families       KEEP pending D172 placement review
native recognizes/equality machinery       KEEP
```

AUD009-E4 explicitly retained:

```text
std:network/IpAddresses                    KEEP
std:network/IpEndpoints                    KEEP
```

The D172 packet separated capability retention from public placement and
recommended:

```text
D172 = Candidate C
       Standard Library public ownership/exposure
       + smallest canonical private runtime substrate
```

with the explicit constraint:

```text
no IP-specific module universe
no Actor-local canonical prototypes
no import-during-transfer
no hidden second module model
```

The project owner explicitly approved that exact pending candidate on
2026-09-19:

```text
aprobada
```

```text
SELECTED_CANDIDATE=C
DECISION_APPROVAL_PROVENANCE=PASS
```

## GITHUB021 invariant/delta consistency

Applicable fixed authority:

```text
NUMERIC_IPV4_IPV6_CAPABILITY                 PRESERVED
NUMERIC_ENDPOINT_CAPABILITY                  PRESERVED
ADDRESS_ENDPOINT_AUTHORITY_FREE              PRESERVED
NO_IMPLICIT_DNS                              PRESERVED
NO_MAPPED_V6_NORMALIZATION                   PRESERVED
D048_EXACT_CANONICAL_PARENT                  PRESERVED
D048_EXACT_STATE_RECOGNITION                 PRESERVED
D048_FRESH_FROZEN_IDENTITIES                 PRESERVED
STRUCTURAL_EQUALITY_HASH                     PRESERVED
MAP_KEY_BEHAVIOR                             PRESERVED
ACTOR_P_TRANSFER                             PRESERVED
NETWORK_TCP_CONSUMPTION                      PRESERVED
```

Observable public-placement delta:

```text
Prelude.IpAddress                            REMOVED
Prelude.IpEndpoint                           REMOVED

std:network/IpAddresses.IpAddress            ADDED / canonical exposure
std:network/IpEndpoints.IpEndpoint           ADDED / canonical exposure
```

No approved capability or recognition invariant is weakened.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Comparative evidence

The research compared several mature placement models.

### Rust

Rust exposes numeric IP and socket-address value types in a low-level networking
module with equality/hash and no inherent communication authority.

Contribution: strong evidence that Candidate A is legitimate and that these are
foundational value types rather than mere parsing helpers.

### Go net/netip

Go exposes compact comparable numeric address/endpoint values in an explicit
standard networking package.

Contribution: strong evidence for Candidate C's separation between reusable
numeric values and universal language names.

### Python ipaddress

Python provides authority-free, hashable numeric IP values through an imported
library module rather than builtin/prelude names.

Contribution: supports library-domain ownership.

### Java java.net

Java exposes address/endpoint objects through a networking namespace/library
rather than as universal language names.

Contribution: supports namespaced ownership while showing that runtime network
operations may still consume those values directly.

### .NET System.Net

.NET similarly places IPAddress/IPEndPoint in a networking namespace.

Contribution: supports domain placement and stable structural networking data
without ambient authority.

The survey does not establish one universally correct layer. It shows that the
key design choice is namespace/public ownership, not whether numeric IP data is a
legitimate foundational capability.

## Comparative scoring

Scores use 1-5; confidence H/M/L.

| Criterion | A Core | B stdlib source | C stdlib + substrate | D hybrid |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 5H | 2H | 5H | 4M |
| Protos alignment | 4H | 3M | 5H | 3M |
| Present-need proportionality | 3H | 4M | 5H | 4M |
| Incremental growth | 5H | 2H | 5H | 3M |
| Future-option resilience | 4M | 2M | 5H | 3M |
| Scalability | 5H | 2H | 5H | 4M |
| Conceptual simplicity | 5H | 3M | 4M | 3M |
| Portability / implementation freedom | 5H | 3M | 5H | 4M |
| Runtime / resource cost | 5H | 3M | 5H | 5H |
| Failure / operability | 5H | 2H | 5H | 4M |
| Deferral / reversibility / migration | 4M | 2H | 5M | 3M |
| Evidence maturity / implementation risk | 5H | 3M | 4M | 3M |

Arithmetic is only a comparison aid.

Candidate C carries one important implementation-risk condition: moving two
Prelude names does not justify a large special runtime/module architecture.

If the migration cannot reuse or establish a small general standard
library/runtime support seam, implementation must stop for architectural review.

## Incremental-design analysis

### Smallest sufficient solution

The smallest public networking surface is an explicit Standard Library module
containing numeric construction/parse/format plus the canonical family objects
required by the retained D048 contract.

Core/runtime need retain only the private substrate necessary to make those
standard values canonical, transferable and directly consumable by Network/TCP.

### Pay for what you need

Programs unrelated to networking no longer receive two networking-specific
unqualified names.

The runtime cost of the canonical frozen prototypes remains effectively constant
and small.

D172 is therefore primarily a public-conceptual placement improvement, not a
runtime-performance optimization.

### Grow as you need

Future networking-domain values such as DNS names, CIDR/network prefixes or
higher protocol descriptors can grow under `std:network` without establishing a
precedent that every runtime-understood networking value belongs in the Prelude.

This decision does not prebuild those values.

### Cost of deferral

Retaining Candidate A would require no immediate work, but moving later after
more public networking consumers exist would increase compatibility/migration
surface.

Performing the move now is bounded while the project is still 0.x.

### Reversibility

The canonical value semantics remain unchanged, so a future placement adjustment
would not require changing serialized numeric state, equality/hash laws or
Network authority semantics.

## Future-scenario stress test

A future networking stack may add DNS, UDP, TLS, HTTP, QUIC, service discovery
or CIDR/network values.

Candidate C keeps that growth namespaced under Standard Library while allowing
runtime/native code to consume canonical standard values where privileged I/O
requires it.

The main regret scenario is discovering that a general stdlib/runtime seam is
substantially more complex than the two removed Prelude names justify.

The escape path is explicit: stop the implementation and reopen the architecture;
D172 does not authorize an IP-specific second module/transfer system merely to
force the selected surface through.

## Strongest argument against Candidate C

Candidate A is already excellent: only two Prelude bindings, mature tests,
constant-size frozen prototypes, straightforward bootstrap, and direct runtime
consumption.

Candidate C creates migration work principally for conceptual namespace
cleanliness rather than CPU/memory reduction.

D172 nevertheless selects C because the existing `std:network` domain is the
approved growth boundary and the canonical runtime substrate can remain private.
The selection is conditional on keeping that substrate general and bounded.

## Intentionally deferred

D172 does not decide:

- DNS/Resolver;
- scoped IPv6 address values;
- CIDR/network-prefix values;
- UDP;
- TLS/QUIC/HTTP;
- Network provisioning/exposure, owned by D173;
- exact hidden stdlib/runtime injection architecture;
- a generic public native-module mechanism;
- serialization/wire formats for IP values; or
- replacement of ordinary-object IP values with branded host objects.

## Implementation consequences

D172 requires bounded implementation work.

```text
NORMATIVE_SPEC_CHANGE_REQUIRED=YES
PRELUDE_BINDING_CHANGE_REQUIRED=YES
STDLIB_NETWORK_SURFACE_CHANGE_REQUIRED=YES
RUNTIME_CANONICAL_PROTOTYPE_RETENTION_REQUIRED=YES
ACTOR_P_TRANSFER_REGRESSION_REQUIRED=YES
NETWORK_TCP_REGRESSION_REQUIRED=YES
GENERAL_STDLIB_RUNTIME_SEAM_REUSE_OR_REVIEW_REQUIRED=YES
IMPLEMENTATION_OWNER_REQUIRED=YES
```

A dedicated Ixxx owns the migration.

## Ratified result

```text
D172_STATUS=RATIFIED
SELECTED_CANDIDATE=C

NUMERIC_IP_CAPABILITY=KEEP
NUMERIC_ENDPOINT_CAPABILITY=KEEP

PRELUDE_IPADDRESS=REMOVE
PRELUDE_IPENDPOINT=REMOVE

STDLIB_IPADDRESSES=KEEP
STDLIB_IPENDPOINTS=KEEP
STDLIB_CANONICAL_IPADDRESS_EXPOSURE=ADD
STDLIB_CANONICAL_IPENDPOINT_EXPOSURE=ADD

D048_CONSTRUCTION=KEEP
D048_RECOGNITION=KEEP
D048_CANONICAL_PARENT=KEEP
D048_EQUALITY_HASH=KEEP
ACTOR_P_TRANSFER=KEEP
NETWORK_TCP_CONSUMPTION=KEEP

PRIVATE_CANONICAL_RUNTIME_SUBSTRATE=KEEP
IP_SPECIFIC_MODULE_SYSTEM=NOT_AUTHORIZED
IMPORT_DURING_TRANSFER=NOT_AUTHORIZED
SHAPE_ONLY_RECOGNITION=NOT_AUTHORIZED

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
