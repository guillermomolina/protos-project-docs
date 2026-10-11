# I093-A — Published source-backed canonical IP protocols and private octet codec

- **Date:** 2026-10-11.
- **Work item:** [I093 / guillermomolina/protos#873](https://github.com/guillermomolina/protos/issues/873).
- **Architectural impetus:** [AUD020 / #872](https://github.com/guillermomolina/protos/issues/872) and [owner-approved direction](../AUD020/AUD020_PROTOS_FIRST_HOST_NETWORK_BOUNDARY_OWNER_DIRECTION_2026_10_11.md).
- **PROTOS_REVISION:** `b47be4b698ca27b537898abee89dd6f4c8c49fc3` ([published product commit](https://github.com/guillermomolina/protos/commit/b47be4b698ca27b537898abee89dd6f4c8c49fc3)).
- **Implementation version:** `0.3.334-SNAPSHOT`.
- **Publication state:** I093-A PUBLISHED, substantive local tests reported PASS by the human executor. **I093 remains OPEN** for B (host descriptors/NIO boundary) and C (architecture/conformance closure).

## Published implementation and retained semantics

The product commit moves canonical `IpAddress` and `IpEndpoint` construction, structural equality and exact hashing from native Java implementations to source closures in:

- `protos/lib/core/IpAddress.protos`
- `protos/lib/core/IpEndpoint.protos`

The canonical factories retain strict receiver validation; `IpAddress(version, bits)` requires ordinary exact Integer version 4/6 and unsigned 32/128-bit bounds; `IpEndpoint(address, port)` requires a recognized canonical address and Integer port 1–65535. Both construct ordinary frozen two-slot instances. Equality requires a recognized receiver, returns false for a nonrecognized comparison argument, and compares only the specified structural state. Exact source hashes remain `bits * 31 + version` and `address.hash() * 31 + port`, including beyond signed 64 bits.

The source parser does not permit `==:` as a literal slot declaration. Each Core source instead defines `_coreIpAddressEquals` or `_coreIpEndpointEquals`; the corresponding Java installer verifies source provenance and moves **the same Closure** to the canonical `==` slot, following the existing `Integer.%` installation precedent. No native equality/hash wrapper survives. Each installed prototype has exactly `call`, `==`, `hash` and `recognizes` as local slots.

Core bootstrap cannot invoke an immediately invoked source factory before its Prelude exists. Source closures therefore refer lexically to `_coreIpAddressCanonical` and `_coreIpEndpointCanonical` anchors in the existing private bootstrap context, instead of invoking factories during source load. The public Core Prelude still excludes `IpAddress` and `IpEndpoint`; the Process-shared canonical frozen families continue to be published through the two `std:network` modules. No bootstrap order or `ProtosPrelude` API change was required.

The remaining native Java bridges, `ProtosStandardIpAddressProtocol.recognizesValue` and `ProtosStandardIpEndpointProtocol.recognizesValue`, inspect frozen state, exact immediate parent, exactly two own slots, and exact numeric/address invariants directly, without dispatching on untrusted candidate methods. The Java `init`, equality, hash and their now-unused helper machinery were removed. `ProtosCoreNativeBoundaryArchitectureTest` expects one native closure per IP protocol rather than four (total Core native closures **138 → 132**, provider count unchanged).

`protos/lib/network/IpAddresses.protos` adds a private, frozen `_ipOctets` codec with `from`/`to` operations over exactly 4/16 network-order Integer octets. It composes/splits numeric bits by exact Protos Integer arithmetic, preserves leading zero octets, and handles all 128 IPv6 bits including values above signed 64-bit range. Existing `v4`, `v6` and `format` use the codec; parsing logic and the exported module selector set remain unchanged. The codec binding is removed from module surface after source closures capture it.

## Exact published product change surface

The published product commit modifies the following 15 paths:

- **Core source:** `protos/lib/core/{IpAddress,IpEndpoint}.protos`.
- **Standard Library source:** `protos/lib/network/IpAddresses.protos`.
- **Native recognition/install bridges:** `src/main/java/com/guillermomolina/protos/execution/{ProtosStandardIpAddressProtocol,ProtosStandardIpEndpointProtocol}.java`.
- **Conformance:** `protos/tests/conformance/network/{ip-address,ip-endpoint}.protos`.
- **Library tests:** `protos/tests/library/network/ip-addresses/{parse-format,surface,validation}.protos`.
- **JUnit:** `src/test/java/com/guillermomolina/protos/execution/{ProtosCoreNativeBoundaryArchitectureTest,ProtosStandardIpFamilyPlacementTest}.java`.
- **Build and release metadata:** `pom.xml`, `CHANGELOG.md`.
- **Generated-bytecode baseline:** `tools/java_generated_bytecode_bci_pe_baseline.json` (the published changelog records 157 new proven sites, 92 removed stale entries, no risky/unknown transitions).

No normative `spec/` file, NIO backend, network Flow contract, `ProtosCoreBootstrap`, `ProtosPrelude`, or numeric representation was changed by this slice. The existing source-file APL headers were preserved.

## Validation provenance and limits

**Human-executor report:** the maintainer confirmed all local tests green and `git diff --check` clean before publishing the product commit. The preceding focal-validation handoff covered IP placement/architecture/networking JUnit tests, IP conformance/library selections and the integrated `make test` lane; the final confirmation reports all tests PASS without retaining individual raw logs here. The maintainer applied `pom.xml` and `CHANGELOG.md` only after successful substantive validation; no additional post-version-change test execution is asserted.

**Source review:** the changed-source diff was reviewed for source-backed method provenance, canonical lookup/capture, callback-free recognition, exact arithmetic, module-surface confinement and equality/hash/map-key tests. This review and the human-reported green suite are different forms of evidence; this record does not claim an independently reproduced build, CI result, Native Image build, benchmark/speedup, or separately executed bytecode baseline gate.

## Remaining I093 work

**I093-B NOT STARTED:** move the guest/host network boundary to immutable defensive-copied 4/16-octet/version/port descriptors and eliminate guest objects, integer prototypes and numeric construction from NIO backend signatures and fields. Guest encoding/decoding and late completion materialization must occur on the owning Actor/Context domain; preserve D173 authority, host IPv6 scope, cancellation, poller, custody and Future lifecycle. The current private codec is a source-level foundation, not an exposed host descriptor API.

**I093-C NOT STARTED:** complete the architectural/conformance closure including all relevant flows and external checks, the Native Image-relevant gate where applicable, and evidence-backed coupling/performance comparisons. Do not infer the entire I093 acceptance matrix from I093-A tests. AUD020's broader audit also remains independently open.

```text
I093_A_PUBLICATION=PUBLISHED
PROTOS_REVISION=b47be4b698ca27b537898abee89dd6f4c8c49fc3
IMPLEMENTATION_VERSION=0.3.334-SNAPSHOT
PRODUCT_TESTS=PASS_HUMAN_REPORTED
PRODUCT_DIFF_CHECK=CLEAN_HUMAN_REPORTED
I093_B_STATUS=NOT_STARTED
I093_C_STATUS=NOT_STARTED
I093_PARENT_ISSUE=CANNOT_CLOSE_YET
D172_D173_NORMATIVE_AMENDMENT=NONE
```
