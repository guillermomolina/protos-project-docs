# D261-R2-G2 — Protos-first general native FFM; RootActor-only direct entry

**Status:** OWNER-SELECTED ARCHITECTURE, 2026-11-23; exact R2 public API and normative specification **NOT YET RATIFIED/PUBLISHED**.  
**Decision owner:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).  
**Explicit approval provenance:** [Owner selection comment](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7105424394); owner response to the revised G2 comparative recommendation: **"ok apruebo recomendación."**  
**Inspected Protos revision:** [`89028494587d4d22309a5c048e37da3518170b18`](https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18).  
**Supporting research and invariant audit:** [D261 GITHUB010/GITHUB021 evidence](../../evidence/D261/D261_R2_G2_GITHUB010_GITHUB021_APPROVAL_AND_SOURCE_AUDIT_2026_11_23.md).

## 1. Selected boundary

R2 shall expose **general, reusable ABI/memory/function capabilities of Java 25 Foreign Function & Memory** to Protos-first source declarations. The FFI model must not be invented around SQLite, ncursesw, termbox2, GTK or another individual C library. These are consumers and potential falsification cases, not architectural limits or required proof-of-concept implementations.

Retain the existing source-backed ordinary `std:ffi` MODULE, foreign-value boundary, host-authorized `ffi:<alias>` acquisition, and generic `protos-ffm` Java FFM provider, including its Native Image build-time integration. No per-library Java provider, universal C normalization shim, new Actor framework, second FFM backend, mandatory header parser, mandatory code generator, or alternative invocation semantics.

The **RootActor** of the current Protos Process is the only Actor authorized to **acquire and directly use native FFM** in R2. This is a new restriction on native FFM alone; it must not modify the general foreign-module provider SPI, other foreign languages, or ordinary source MODULE imports. `import("std:ffi")` remains inert and available for ordinary description-building; it does not acquire native authority or load a shared library. Authorization from the host remains separately mandatory.

**Critical distinction:** RootActor-only governs entry **from Protos into FFM**; it does **not** prohibit or attempt to govern actions performed by a host-authorized native library. Under the already-approved D259 `TRUSTED_IN_PROCESS` contract, a library may create operating-system threads, read/write files, access networks, call other native code and maintain process-global memory even where no guest `--allow-filesystem` or other guest permission exists. The host explicitly accepts all process-level native risks. Native threads are not Protos Actors. No attempt to sandbox, serialize, monitor or schedule the library's internal native threads is part of this design.

RootActor identity is logical Process authority, **not a promised thread-affinity guarantee**: direct RootActor entry uses its host calling thread; a RootActor in message-service mode may execute separate segments on scheduler carriers. One synchronous downcall executes in its current segment. Existing `Arena.ofConfined()` for call-temporary memory and `Arena.ofShared()` for session lifetime provide the implementation baseline; no special FFM worker, global lock or permanent RootActor pinning may be introduced to address a speculative concurrency requirement.

## 2. General mechanisms chosen at architectural level

**ABI types/layouts.** Support a composable declaration model for fixed-width and ABI-canonical C scalar types, pointers, structs, unions, sequences/arrays, nested layouts, size, padding and alignment. Java's `MemoryLayout`, `ValueLayout`, `Linker.canonicalLayouts()`, `FunctionDescriptor` and corresponding layouts are the technical substrate. Structures passed by pointer **and by value** are ordinary FFM capabilities where supported by the host ABI, including the `SegmentAllocator` parameter needed by Java FFM for group-layout returns. Do not create a Java class per C struct or library. ABI-specific packing, bitfields, union interpretation and layout provenance must be admitted accurately, not guessed; generic layout support does not automatically parse C headers.

**Native memory and pointers.** Support memory-region allocation, field/element access, slices/views, read/write/copy and release through `MemorySegment` and `Arena` concepts. Distinguish allocated/owned, borrowed and C-owned memory; preserve the actual resource lifetime and error boundaries. Do not force every access through byte-copy snapshots. FFM does not know a foreign pointer's live extent or true allocation size automatically; declaring it incorrectly is a trusted-ABI risk. A future explicit unsafe numeric-address constructor is a design question, **not approved by the G2 architecture selection**; no implicit Integer-to-address conversion is approved.

**Function calls.** Continue to expose ordinary Protos Closures and explicit binding declarations; extend accepted layouts/carriers through the existing generic FFM provider. The physical C pointer/ABI representation must be distinct from (a) the owner/destructor/provenance of a resource, and (b) the library exporting the called symbol. R1's blanket same-library handle/destructor admission must not become an intrinsic C-ABI restriction: valid cross-library pointers need an explicit compatible descriptor and surviving native lifetime. Avoid assuming a loader symbol follows the lexical C-identifier regex. Exact public operation names/parameters remain to be specified, not inferred from illustrative code.

**Upcalls.** Java FFM provides `Linker.upcallStub`. Synchronous, same-dynamic-extent callbacks have an already-published Protos semantic baseline (D189 / `ACTORS.md` §24J). The first implementation's upcall scope and public names are **deferred**. Retained callbacks, callback entry from arbitrary foreign-created threads, asynchronous reentry into the RootActor, new Tasks/queues and implicit message brokers are **not selected**. A native library's creation of its own C threads is still authorized: no Protos callback ingress is implied.

**C++ libraries.** A shared library implemented in C++ is not banned merely for its implementation language: FFM may call exported functions with a compatible ABI. Automatic mapping of C++ class/object ABI, virtual dispatch, templates and C++ exceptions is not promised in R2 and may be investigated separately in an R3 only when justified. The distinction is capability of an actual native symbol, not an allow/deny list by library type.

**Native Image.** Retain PLAT077's integrated generic provider and build-time FFM descriptor registration. Java/JVM can link more descriptors dynamically than a given already-built Native Image; the compiler must know the concrete downcall/upcall descriptor shapes it generates. Composite/struct-by-value signatures need correct corresponding Native metadata and carrier invocation support; adding a shape name to the existing scalar-only `NATIVE_SHAPES` set alone is not a complete implementation. Research explicit metadata, the tracing agent and hosted registration on concrete requirements. Do not generate combinatorial unused signatures, invent an alternate FFI engine, or promise arbitrary dynamic signatures on a previously built native executable.

## 3. Compatibility and normative gates

The owner was explicitly shown and approved **one observable break**: although D238/R1 currently allows native FFM acquired by separate Actors, R2 introduces **RootActor-only native FFM acquisition and invocation**. This is a *bounded, owner-approved delta* to R1, not an unchanged-behavior claim. It must be published in normative `spec/semantics/FFI.md`, reconciled with `spec/semantics/MODULES.md`, `spec/concurrency/ACTORS.md`, `spec/semantics/SQLITE.md`, documentation and tests before implementation or normative closure. Other foreign providers must retain their existing rules. No automatic proxy or transferred live native handle.

The following earlier decisions remain in force, except for that explicit RootActor-only FFM delta:
- **D238-B1-SQL-R1:** ordinary Protos source MODULE and Closures, laziness, existing resource custody/error domains, no new Core type family.
- **D259-B2+A:** host-only default-deny `--allow-library alias=target`, trusted native code permitted to use full host process effects; guest I/O permissions independent.
- **PLAT077-A:** one Native Image product with existing generic FFM provider compiled in, no bundled SQLite or other third-party C library; exact AOT signature registration.
- **D188/D189, PLAT053, ACTORS §24J/24K.1/24L, P:** existing foreign-value/error boundaries, same-segment synchronous call semantics, resource nontransferability and no automatic proxy, trusted native-global exception.

**D260-B1** remains the already selected public terminal screen model, not an FFM API or backend decision. **PLAT078** remains an independent, bounded terminal backend comparison; it shall not implement the FFM engine or a terminal library as a D261 feasibility proof. D261 does not approve a terminal backend, Qt binding, a GUI or a library-specific C shim.

## 4. Pay-as-you-grow delivery model

The recommended order is **capability-based** and may be grouped into bounded implementation slices, not a mandate for a separate Issue per line:

1. Publish the **exact** R2 public contract and RootActor-only compatibility rules through an independently approved normative specification change; agree only on capabilities actually needed.
2. Implement reusable ABI descriptor/layout and memory-region mechanisms through existing Protos `std:ffi` and FFM provider. Avoid new production Java classes wherever possible; any new Java production operation requires concrete method, annotation and equivalent behavior from GraalPy/TruffleRuby/another Truffle implementation.
3. Add general structured downcall argument/return support, resource provenance across authorized libraries, and only actual Native Image descriptor registrations.
4. Validate on a **small independent C ABI fixture**, not a full ncursesw/termbox2/GTK application, with RootActor allowed/non-root rejected, layouts, by-value structures, regions/ownership, exact symbols, cross-library pointers, JVM/Native and no-use laziness.
5. Investigate and select later upcall/variadic/other-ABI/header-generation capabilities only when separately justified; no native thread ingress or multi-Actor FFM support in R2.

Do not claim zero cost: native image descriptors and generic FFM implementation may add build-time or binary footprint. Measure actual effects when implemented. Inert `std:ffi` import still must not initialize the native library subsystem.

## 5. Explicitly unresolved — owner gate remains

Selected G2 is an **architecture**, not the finished public `FFI.*` contract. Further exact decisions include:
- public names, descriptor forms, treatment of ABI-canonical scalar names, layout/padding verification, bounds and alignment;
- when and how a caller may assert an arbitrary address/length, ownership/borrow invalidation, destructor from a second authorized library, and signedness/bitfields;
- what (if any) synchronous upcalls belong in the initial implementation, what can safely be deferred;
- Native Image signature registration and distribution policy beyond PLAT077's established finite set;
- platform/OS targets beyond the verified Linux x86-64 LP64 profile;
- precise `ForeignError`/ordinary error cases and `std:sqlite` migration.

These choices cannot be installed as implementation details absent owner approval when substantive. **GITHUB021 architecture selection: PASS within explicitly approved scope; exact R2 public semantics: PENDING.** No `spec/` edit, product Java change, native build, tests or experimental runtime acceptance is claimed by this documentation record.

## 6. Most important counterargument

The general mechanism costs more initially than two byte-copy operations. The reason to pay that cost is structural: byte-copy-only FFI would create an incompatible second model once the next library needs a struct pointer, nested union, by-value return or cross-library memory. But extending the system into arbitrary foreign-thread callbacks, C++ object binding or universal ABI generation now would charge current users for speculative work. G2 retains a small coherent memory/layout/function nucleus while keeping those later capabilities genuinely optional.

**Next decision:** exact public R2 API and normative publication. **Issue D261 stays open; publication of this non-normative project record is not implementation or spec conformance.**
