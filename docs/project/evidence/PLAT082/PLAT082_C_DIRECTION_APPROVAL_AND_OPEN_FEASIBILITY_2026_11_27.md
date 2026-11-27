# PLAT082 — Candidate C direction approval and open feasibility

Date: 2026-11-27
Issue: [PLAT082/#1214](https://github.com/guillermomolina/protos/issues/1214)
Parent: [AUD025/#1209](https://github.com/guillermomolina/protos/issues/1209)
Research source checkpoint: [509221a9d28b79e9da9accf721fad2358d81ec49](https://github.com/guillermomolina/protos/tree/509221a9d28b79e9da9accf721fad2358d81ec49); original census baseline 52591e7ce5c7e068f98101db94e77e68a64320da.

## Approval provenance

In the active conversation the project owner explicitly stated: “ok apruebo la recomendacion.” This refers to the presented Candidate C, selective Protos-first migration of portable encoding algorithms and text I/O policies. The preceding recommendation explicitly marked research incomplete and PLAT082 open; approval selects the direction, not a proven concrete migration or completed GITHUB010 packet.

## Selected direction and invariant consistency

Prefer ordinary Protos implementations for portable encoding algorithms and text I/O policies whenever existing language mechanisms can preserve the complete normative contract. Retain necessary runtime representation, host codec, bootstrap, execution, commitment and lifecycle boundaries. No new infrastructure, public protocols or semantic changes solely to enable migration. Retain existing implementations wherever migration is not feasible.

This preserves the AUD025 owner constraints: no deletion quotas, LOC targets, benchmark campaigns or success metrics; ordinary correctness and applicable validation remain required. No invented production Java infrastructure; any necessary Java change requires an exact verified method, annotation and behavior precedent in GraalPy or another Truffle language. Guest primitives remain long/double/boolean; preserve message semantics, specialized hot paths, isolation, mutability, suspension and Native Image compatibility. No new Encoding API, host codec framework or normative amendment. No phase-order hard dependency is inferred.

## Observed source evidence

- `src/main/java/com/guillermomolina/protos/runtime/io/ProtosEncodingValue.java`: PortableEncoder.encode uses CharsetEncoder with REPORT under TruffleBoundary; PortableDecoder.preview uses explicit BOM handling, scalar decoding and replacementExtent. Immutable descriptors transfer by identity; per-flow state is separate. HostCodec remains a distinct provisioning boundary.
- `ProtosTextReader.java` in the same directory: scanText, scanLine, resolvePendingLf and appendLineText are policy candidates; retained bytes, ordered requests, deferred errors, cancellation, lifecycle and CPrime lower calls require careful boundary preservation.
- `ProtosTextWriter.java`: driveWrite, ordered requests, encoder previews, downstream effects, flush, cancellation and release; complete-payload validation precedes irreversible output.
- `execution/protocols/io/ProtosStandardEncodingProtocol.java`: standard family admission and conversion dispatch.
- `spec/io/TEXT_IO.md`: exact semantic argument domains, fresh open Bytes, Unicode 17 maximal-subpart replacement invariant under chunking, matching BOM, ISO-8859-1, line byte budget and error precedence, zero-consumption read cancellation, write atomicity and ordered failure.
- `spec/semantics/VALUES_AND_COLLECTIONS.md`: String indexing/iteration returns single-scalar Strings rather than numeric code points. Existence of an adequate numeric scalar construction/extraction mechanism is not yet established.
- `protos/lib/io/files/module.protos`, `protos/lib/csv/csv/module.protos`, `protos/tools/package/LockSyntax.protos`: existing empty Bytes construction via Encoding.UTF8.encode(""); bootstrap circularity must be checked.

## Alternatives and limits

A: retain existing implementation (legitimate fallback). B: migrate everything (not mandatory; risks new bridges and broken lifecycle). C: selectively migrate portable algorithms/policies using existing facilities (approved direction). D: delegate all conversion to host/TruffleString (insufficient alone to establish exact replacement, BOM and byte accounting). A preliminary twelve-criterion comparison was presented, not a measurement or final scored decision packet. Strongest objection to C: a split can increase total complexity and introduce bridges; retain existing code when that happens.

## Remaining gate

Complete normative IO_CORE/BYTE_IO and relevant value/object/transfer reads, scoped instructions, current tests and prior records; establish actual String/Bytes primitives and bootstrap, CPrime boundaries and source-pinned cross-runtime precedents, especially GraalPy. Confirm a concrete feasible migration without new infrastructure and complete GITHUB010 evidence before ratification/closure or allocation of a separate I issue. PLAT081 is relevant to shared buffering policy but no native hard dependency is asserted. No tests, builds, benchmarks, product edits or runtime gains are claimed. No normative changes approved. AUD025 phase D remains unchecked.
