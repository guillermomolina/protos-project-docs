# D261-R2-P3-M — Exhaustive native-memory and pointer research

**Date:** 2026-11-24
**Status:** RESEARCH / RECOMMENDATION ONLY — no owner selection of P3-M and no normative publication.
**Work owner:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
**Product revision audited:** [`8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Already approved (not reopened):** [G2](../../decisions/language/D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2](../../decisions/language/D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1](../../decisions/language/D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md).
**Test status:** COMPILATION=NOT_RUN; TESTS=NOT_RUN; NATIVE_IMAGE=NOT_RUN; PERF=NOT_MEASURED. No production or `spec/` change by this research.

## 1. The exact P3-M question

What observable authority does a RootActor receive from a native pointer, how are its **address, bounded extent, lifetime, owner/destructor, provenance, generation and read/write permissions** represented, and how does Protos expose them without inventing a new object model? Specifically, decide whether a RootActor may explicitly assert an *unverifiable* extent/lifetime for a C-managed pointer so it can be dereferenced via already-approved native-region `at/atPut`.

An address is **not** a readable span. A pointer with zero known bytes can still be passed opaquely to another authorized C function, but it cannot be safely read. A *borrowed* view never owns the destructor. An *owned* native region must remain valid independently of a per-call temporary arena until released; borrowed views must expire with their actual declared anchor. A copy into Protos Bytes/slots is independent data, not a view.

**Trust limit:** D259/G2 already chose `TRUSTED_IN_PROCESS`. C may mutate/free/reallocate memory, spawn operating-system threads, or corrupt the process independently of the RootActor. No bounds wrapper can prove the native lifetime. The design must disclose that limitation, not promise sandbox safety.

## 2. Current Protos contracts and exact owners

| Source | Confirmed facts and audit consequence |
| --- | --- |
| [`spec/semantics/FFI.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/FFI.md) | R1 `FFI.bind` is an ordinary Closure. Native owned handles are associated with library/session/generation; all owned results are adopted before fallible return conversion, even after nonzero C status. `release` retires a handle *before* its destructor, never retries, and an old handle does not revive if the address is reused. Private `borrowed`/ `preserving`/ `copy` only, 16MiB per-copy cap, borrowed epoch advanced before any *admitted* non-preserving downcall/destructor, never for pre-entry rejection. R1 excludes public native memory regions and numeric pointers |
| [`MODULES.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/MODULES.md) | `std:ffi` import is inert and creates no native authority. Foreign module facade is Actor-local; G2 is an intentional **native FFM only** RootActor restriction, not a restriction on other foreign providers |
| [`ACTORS.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/concurrency/ACTORS.md) §§16,24J,24K.1,24L | Native references are nontransferable; the complete Actor transfer graph must be valid before message acceptance. Synchronous C calls cannot implicitly interleave other work in the same Actor. Authorized C has host-process effects; no automatic proxy, special scheduler, global FFM lock or foreign-thread-to-Protos ingress |
| [`PARALLEL_EXECUTION.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/concurrency/PARALLEL_EXECUTION.md) | P boundaries preserve resource/owner isolation, no automatic transfer of native FFM resources |
| [`VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/VALUES_AND_COLLECTIONS.md) | `[]` is the normal `at/atPut` message protocol, not dynamic slots. Foreign-reference identity does not come from Java wrapper address or guest Integer |
| [`SQLITE.md`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/SQLITE.md) | Existing copied-row behavior, native-resource cleanup, synchronous execution, and separate SQLiteError policy remain; RootActor-only requires explicit normative compatibility reconciliation |
| [`FfmHandles.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmHandles.java) L27–42,50–144,146–195,252–299 | `Resource` has per-acquisition serial, LIVE/RELEASED/UNCERTAIN; `Borrowed` stores session+borrowEpoch; `Custody` tracks owned resources. The 16MiB private copy uses restricted `MemorySegment.reinterpret(length)` after admission, not physical proof of C memory validity |
| [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java) L31–49,78–117,139–156,195–285 | One **shared** Arena currently controls library lookup/symbol lifetime. Existing per-session active-call admission waits before closing and drains custody. Its library-lifetime Arena must **not** automatically be the only allocator/lifetime for every R2 region: that would prevent per-resource early reclamation |
| [`FfmBinding.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java) L290–400 | Per-call transient `Arena.ofConfined()`; invalidation before native entry; Stage A captures outputs, Stage B adopts every owned result, Stage C performs fallible conversion. A by-value struct return needs a result allocator that outlives this call-local Arena |
| [`FfmValueOperations.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmValueOperations.java) and [`FfmTypes.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmTypes.java) | Private `copy`, `isLive`, `release` exist; no public R2 memory model yet. Existing R1 opaque descriptors intentionally enforce logical library/session provenance; R2 physical pointer ABI crossing must not silently erase those checks |
| [`protos/lib/ffi/module.protos`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/lib/ffi/module.protos) | Source-backed ordinary frozen descriptors/IdentityMap metadata and Closure `bind`; no need for per-C-type Java classes |
| [`ffi_test.c`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/tests/foreign-provider/native/ffi_test.c), [`ffm-abi.protos`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/tests/foreign-provider/ffm-abi.protos), [`ffm-public-api.protos`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/tests/foreign-provider/ffm-public-api.protos), [`ProtosI144FfmFixtureAbiTest.java`](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java) | Existing C-independent fixture verifies nullable/owned results, destructor once, borrowed epochs and copy lifetime, no numeric-address forging and authorization. R2 must extend it, not replace legacy results |

**Relevant other norms read:** `OBJECT_MODEL.md`, `CALLABLES.md`, `ERRORS.md`, `PROTOS_GRAMMAR.md`; P3-M should not change member syntax, `call`, Boolean control, lexical bindings, or foreign-Error taxonomy.

## 3. Verified external precedents — 10 systems, 3 design routes

**Route A, scoped/typed memory**:
1. **Java 25 FFM** — [MemorySegment](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/MemorySegment.html), [Arena](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Arena.html), [Linker](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Linker.html). C-returned pointers are normally **zero-length, globally scoped** segments: passable but not dereferenceable. `reinterpret` and address-target layouts are restricted, because Java cannot know C's real size/lifetime. A Shared Arena is closeable from any host thread; a Confined Arena is owner-thread only. Group return introduces `SegmentAllocator`.
2. **Rust** — [slice::from_raw_parts](https://doc.rust-lang.org/core/slice/fn.from_raw_parts.html), [NonNull](https://doc.rust-lang.org/stable/core/ptr/struct.NonNull.html): raw pointer + length is an **unsafe assertion** of alignment, validity and lifetime. Rust's safe references/borrow checker are language features *absent from Protos*, but its explicit trusted bounds are informative.
3. **Swift** — [withUnsafeMutableBytes](https://developer.apple.com/documentation/swift/array/withunsafemutablebytes(_:)), [manual memory management](https://developer.apple.com/documentation/Swift/manual-memory-management): scoped closure borrow makes temporal bounds explicit. An implicit scope on all Protos native operations would complicate suspension and long-lived C resources.
4. **Zig** — [pointer and slice reference](https://ziglang.org/documentation/master/): slices have explicit lengths, C pointer families need trusted access. Shows that address and bounded span are distinct even in a native language.

**Route B, direct unsafe C-data access**:
5. **Python ctypes** — [ctypes](https://docs.python.org/3/library/ctypes.html), [CPython reference source](https://github.com/python/cpython/blob/main/Doc/library/ctypes.rst): `from_address(integer)` permits unbounded-risk raw admission; `from_buffer` shares and `from_buffer_copy` snapshots; `_b_base_` and `_objects` hold some Python owners alive but cannot prove that arbitrary native C hasn't freed the data.
6. **LuaJIT FFI** — [semantics](https://luajit.org/ext_ffi_semantics.html), [API](https://luajit.org/ext_ffi_api.html): `ffi.cast` permits raw pointer reinterpretation; `ffi.gc(ptr, finalizer)` can arrange eventual native cleanup. The documentation explicitly warns about unsafe memory and C-owned lifetime.
7. **Ruby FFI / TruffleRuby** — [MemoryPointer](https://rdoc.info/github/ffi/ffi/FFI/MemoryPointer), [AutoPointer](https://rubydoc.info/gems/ffi/FFI/AutoPointer), [TruffleRuby PointerNodes](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/extra/ffi/PointerNodes.java), [FFMSupport](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/platform/FFMSupport.java): explicit allocate/read/write/free and optional GC destructor; `@Specialization` methods provide real Truffle precedent, but exposed raw pointer addresses do not confer memory safety.
8. **GraalPy** — [NativeMemory](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeMemory.java) and [CStructAccess](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/cext/structs/CStructAccess.java): `malloc/free/readByteArrayElements/writeByteArrayElements`, `allocate/readIntField/writeIntField` specialize native operations. GraalPy has stronger CPython compatibility obligations and different internal ownership; do not copy it blindly.

**Route C, custodied native handles**:
9. **.NET SafeHandle** — [official API](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.interopservices.safehandle?view=net-10.0), [runtime guidance](https://learn.microsoft.com/en-us/dotnet/fundamentals/runtime-libraries/system-runtime-interopservices-safehandle): native handle identity, ownership, managed close and P/Invoke keepalive, plus explicit `DangerousGetHandle` escape. Supports R1 custody, but a SafeHandle alone does not establish an interior pointer's bounds.
10. **Espresso/NFI** — [NativeMemory](https://fossies.org/dox/graal-vm-25.5.5.0.3/interfacecom_1_1oracle_1_1truffle_1_1espresso_1_1ffi_1_1memory_1_1NativeMemory.html) and [NFINativeAccess](https://fossies.org/linux/graal-vm/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/ffi/nfi/NFINativeAccess.java): native memory and raw pointer interop in Truffle's guest-Java context. Its JNI/class/GC conventions are not portable Protos semantics.

**Broader Truffle survey:** Sulong executes C/LLVM pointer semantics at a distinct execution substrate; GraalJS focuses on JS/node interop rather than exposing a matching generic Java 25 C memory API; Apple Pkl is a configuration language with no materially comparable native pointer lifetime API. These cannot replace the ten relevant comparisons. **Platform differences:** Linux LP64 initial target; Windows LLP64 CRT allocator mismatch, Darwin/BSD ABI/allocator differences and 32-bit pointer widths must not be inferred from Java or a Linux fixture. [GraalVM 25 Native Image FFM](https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/) supports FFM including shared arenas and exact runtime downcall-descriptor registration, but not arbitrary unregistered signatures.

## 4. Meaningful alternative designs

| ID | Contract | Disqualifying risk |
| --- | --- | --- |
| **M0** status quo | Existing R1 opaque + 16MiB private copy only | **Underengineering:** fails approved P2 live-region requirement, needs later rewrite/shims |
| **M1** raw ctypes | Guest-visible raw addresses/casts, optional GC free | **Catastrophic:** arbitrary address forging, UAF, double free; not aligned with G2 non-numeric pointers |
| **M2** managed session only | Every allocation/view tied to the existing shared session Arena | No independent early release; C can invalidate external borrowed bytes before session shutdown |
| **M3** bounded provenance + explicit trusted assertion | Distinguish owned bounded region, borrowed anchored view and pass-only pointer; allow RootActor to deliberately assume C-owned extent/lifetime after authorization | **Recommended**, with honest unsafe escape and no false safety guarantees |
| **M4** Core borrow checker | Affine/linear typed owner graph for all native references | **Overengineering:** changes ordinary Protos objects/closures, still cannot constrain arbitrary C |

## 5. GITHUB010 twelve-axis scoring

Each cell is **1–5 + H/M/L confidence + short rationale**; H=high, M=medium, L=low. Scores are comparative aids, not numeric approval.

| Criterion | M0 R1 copy-only | M1 raw ctypes | M2 session-only arena | M3 bounded + trusted assertion | M4 linear/borrow-check |
| --- | --- | --- | --- | --- | --- |
| 1. Correctness | 2H no live region | 1H uncontrolled UAF | 3M misses C-borrow expiry | 5M explicit roots, views and trust | 3L rewrites guest semantics |
| 2. Protos fit | 2H fails P2 | 2H numeric addresses | 4M existing at/atPut | 5M ordinary protocols, no Core type | 1H affine/linear Core |
| 3. Pay for now | 5H R1 already exists | 4M tiny API, external safety costs | 3M easy initial implementation | 4M one bounded-unsafe escape | 1H whole language pays |
| 4. Incremental growth | 1H ABI model must change | 2M unsafe API hard to withdraw | 2M impossible independent frees | 5M add policies without redesign | 3L only after a Core rewrite |
| 5. Future resilience | 1H shims/copies | 3M powerful but unsafe | 2M session-lifetime lock-in | 5M opaque and typed paths coexist | 3L C interoperability uncertain |
| 6. Scalability | 2M copy cost | 2H native races/UAF | 2M arena retention | 4M root-local, no global lock | 2L alias tracking explosion |
| 7. Conceptual simplicity | 5H current R1 only | 2H simple surface, hidden hazards | 3M owner unclear | 4M explicit four roles | 1H novel ownership calculus |
| 8. Portability | 4M copy is portable | 2H address-width exposed | 3M JDK backend | 4M ABI isolated to provider | 2M language/runtime portability risk |
| 9. Runtime cost | 2H large copies | 4M fast but unguarded | 2M excessive retention | 4M checks on native operations | 1L global loan tracking |
| 10. Failures | 4H bounded copy failures | 1H arbitrary crashes | 3M local scope checks only | 4M pre-entry diagnostics for known state | 2L complex dynamic rule |
| 11. Deferral/migration | 1H rewrite for native views | 1H cannot retract unsafe calls | 2M change resource identity later | 5M provenance model stable | 2L new Core migration |
| 12. Evidence maturity | 5H existing tested R1 | 4H LuaJIT/ctypes precedent | 4H FFM Arena precedent | 4M Java FFM + R1 + Rust + .NET | 2L no such Protos prototype |

**Non-compensating gates:** M0 has a real migration cliff from copy-only to live regions; M1's unchecked UAF is not outweighed by speed; M2's lifetime retention is not cured by its small code footprint; M4 imposes a new Core ownership discipline without preventing native C from violating lifetimes. M3 has one serious risk: a falsely asserted C extent/lifetime can still crash the process. That risk must remain prominent in the API and documentation.

## 6. Recommended exact semantic candidate: M3 — NOT YET APPROVED

### 6.1 Four distinct roles

**Opaque/pass-only pointer:** a C pointer, including an ordinary zero-length native return, may be passed to an ABI-compatible function, but cannot be dereferenced without an accredited span. Pointer is an admitted native capability, not an Integer, destructor right or freeable owner.

**R2-owned region:** `allocate(layout)` under RootActor returns typed native storage with an exact extent and an independent lifetime. The session owns its cleanup; logical release invalidates it and every derived view. A per-resource Shared Arena or *equivalent existing FFM primitive* may implement its physical lifetime, while the library's existing long-lived Arena keeps its symbols. A per-call Confined Arena remains useful only for true call-temporary input/output buffers.

**Borrowed region:** a bounded view into native memory **owned elsewhere** carries a lifetime anchor (R2 allocation, an owned C resource, session/library-static state with an explicit trusted promise, or a borrow epoch) and never has its own destructor. Root-owner retirement, invalidation of its epoch or session close revokes it and all slices. Independently allocated R2 memory does not expire on unrelated non-preserving calls merely because R1 private borrowed data do.

**Explicitly trusted extent assertion:** a RootActor may elect to turn an already admitted C pointer with unknown extent into a bounded readable/writable view **only by explicitly asserting size, matching layout and a valid lifetime anchor**. That declaration is an unchecked claim about C, not proof. The caller bears responsibility for calls that invalidate C memory; no arbitrary `Integer -> address`, no global unsafe host escape, and no right for non-RootActor to invoke native FFM. R1 private borrow and ownership semantics remain.

### 6.2 Proposed operations (spelling not ratified)

```protos
FFI: import("std:ffi")
PointLayout: FFI.struct(Array(
    FFI.field("x", FFI.i32),
    FFI.field("y", FFI.i32)
))
region: FFI.allocate(PointLayout)
region["x"] = 12
ptr: FFI.addressOf(region)                   // native pointer, not Integer
view: FFI.view(ptr, PointLayout)             // known root extent: no guess
part: FFI.slice(region, 0, FFI.sizeOf(PointLayout))
FFI.release(region)                          // retires every derived view
// view["x"] now fails before memory access
```

```protos
unknown: ffiCallReturningUnboundedCPointer()
// unknown can be forwarded to compatible authorized native functions, not read
bounded: FFI.assumeExtent(unknown, byteCount, lifetimeAnchor)
// explicit trusted assumption only; NOT a check that C keeps memory alive
```

These are source sketches, **not executable code or public-name ratification**. `lifetimeAnchor` cannot be an arbitrary String purporting to be a memory owner. A library/session anchor establishes at most that the native library remains loaded; it does **not** prove a C allocation has not been freed earlier. An expired stack pointer cannot be made valid by `assumeExtent`. Whether a distinct `assumeExtent`, `borrow` or scoped `withBorrowed` operation is the final spelling is an owner decision.

### 6.3 Non-negotiable memory laws for owner review

1. **Known bounds:** validate offset/count nonnegativity, arithmetic overflow, exact allocation size, ABI alignment, layout compatibility and live owner **before** every read/write. Deriving a slice cannot enlarge original known extent. Null pointers cannot become a nonnull view; zero-length nonnull pointers may be forwarded opaquely.
2. **Provenance:** owner/session/root identity and generation are distinct from native address. Reused address creates a new native acquisition, not a resurrection of a released token. Any pointer through a struct field must preserve or explicitly regain accredited bounds/anchor, never silently inherit the parent root's full allocation if pointee is independent.
3. **Custody:** one releasing owner. Views and derived pointers never free native memory; C-owned resources are destroyed with the declared original producer/destructor, not a later consumer library. Native destructor is attempted at most once after logical retirement, even when it fails. Record uncertain physical outcome without retry.
4. **Lifetime:** root release or session close revokes all descendants without requiring O(number of views) enumeration; use root generation/epoch validation. Known invalid operations fail pre-entry. C's independent free/realloc/foreign-thread operations remain trusted hazards.
5. **Native calls:** preserve R1 `beginCall/endCall` shutdown admission and Stage B pre-publication owned custody. No physical release during a synchronous admitted call that still uses memory; do not add a global lock or pinned thread. The existing per-session library Arena **is not** automatically the lifetime of every native allocation.
6. **Borrow epochs:** existing R1 `preserving` assertion and invalidation ordering retained for R1 borrows. R2 borrowing may additionally anchor to a particular owned resource (retired on destructor). A rejected pre-entry operation must not advance epochs or revoke any view.
7. **Cross-library:** passing a compatible physical typed pointer from one authorized native library to another is allowed, independently authorizing both symbols. **Legacy nominal R1 `opaque` checks remain** for those old handles; passing a pointer never transfers destructor ownership.
8. **Isolation:** native owners/regions/pointers/views, and objects containing them, are not automatically Actor/P transferable. Explicit copies into independent Protos Bytes or ordinary objects follow existing transfer rules.
9. **Guest API safety claim:** native FFM is `TRUSTED_IN_PROCESS`, not a sandbox; even with correct guest bounds and liveness, a false C contract or concurrent trusted native thread can cause corruption/crash.
10. **Call vs resource allocations:** long-lived RootActor regions must not rely on `Arena.ofConfined` because logical RootActor execution may resume on another host carrier; Shared Arena is supported in Java 25 and GraalVM 25 Native Image. `Arena.global()` or `Arena.ofAuto()` cannot promise deterministic early release.
11. **Copy budget:** R1's 16MiB restriction covers **private borrowed copying only**; it is not silently a global R2 allocation/copy limit. Reject size/overflow and allow explicit resource limits as separately specified.
12. **Errors/AOT:** malformed public values, nonexistent owner, expired views, release of borrowed memory, unauthorized RootActor and unsupported exact Native Image descriptors fail before native memory access/entry as ordinary Errors. Entered foreign failures use existing ForeignError and native status is data. Exact aggregate FunctionDescriptor registration required, not pointer substitutions.

## 7. Stress/falsification and independent conformance

| Probe | Acceptance/rejection |
| --- | --- |
| C returns zero-length nonnull pointer | forwardable to compatible C, dereference rejected until accredited |
| C returns NULL | nullable `null` only, never a dereferenceable region |
| C returns a pointer plus known length | explicit bounded/anchored assertion, trusted warning |
| Address points to freed C stack/heap | no false safety guarantee even with assertion |
| Negative/huge length, offset overflow, misalignment | ordinary rejection before any memory access |
| Nested region and slices, root release | all children invalid; release once |
| C reuses freed address | old identity stays revoked |
| Borrowed view passed to `FFI.release` | Error; no native free |
| Destructor throws or returns nonzero | logical retired once, never retried, uncertain physical outcome recorded as needed |
| C-owned out pointer captured, later output conversion fails | prior owned output still in custody |
| Independent root region across unrelated C call | stays live unlike R1 borrow epoch |
| SQLite column borrowed data invalidated by stepping | proper epoch/owner policy, independent snapshot still valid |
| Cross-library pointer, both libraries authorized | physical-pointer case permitted, destructor owner unchanged |
| R1 opaque handle from wrong library | nominal mismatch still rejected |
| RootActor message-service switches host carrier | long-lived region still accessible |
| Non-RootActor imports `std:ffi` | inert import succeeds |
| Non-RootActor native deref/alloc/library bind | denied before FFM native entry |
| Actor/P message contains live region in nested graph | `NonTransferableValue`; no partial delivery |
| Trusted C thread races a read/free | outside guest safety guarantee, no false atomicity |
| Native Image exact group descriptor unregistered | reject before native call |
| Struct by-value return | independent result allocator, not call-temp memory |
| No FFM use in program | no eager library load or shared resource allocation |

**Existing test anchors:** `ProtosI144FfmFixtureAbiTest`, `native/ffi_test.c`, `ffm-abi.protos`, `ffm-public-api.protos`, `sqlite-borrow-model.protos`. Extend with independent C fixture cases for owned regions and cross-library pointers; run JVM and *actual* Native Image, not tests that merely assert expected metadata. Existing R1 multi-Actor native cases need explicit reconciliation against already-approved G2.

## 8. Incremental cost, scalability and escape paths

**Smallest sufficient today:** Root-owned typed region with deterministic release, borrowed typed view with a lifetime anchor, zero-length pass-only C pointer, explicitly trusted extent/lifetime assertion, ordinary `at/atPut`, no numeric address forge and existing R1 custody/epoch/error/lifecycle behavior. General R2 structs/arrays require these basics. A mandatory GC finalizer, global memory registry, process-wide FFM worker, per-field Arena, Rust-style borrow checker, per-library C shim or per-struct Java class has no current justification.

**Concrete cost of deferring required features:** if a raw address API is published first, later provenance/alias invalidation would break its identity and error semantics. If a session Arena owns every allocation, programs that allocate and release thousands of regions retain them until session shutdown and future per-region release would need changing both lifetime and custody. If borrowed C-owned memory cannot be explicitly bounded, actual C APIs require dedicated Java/C shims rather than the approved generic provider.

**Real overengineering danger:** fine-grained static borrowing across closures/Actors, per-field lifetime tokens, implicit Protos callback scopes, all-host ABI tables or a custom native allocator. Such capabilities can be added later where evidence warrants; M3 keeps an extension path without preimplementation.

**Stress:** (i) million tiny allocations → benchmark Arena-per-root cost; batching *physically* same-lifetime allocations may be allowed only while independent logical release and revocation remain; (ii) 10k slices of one owner → generation/epoch invalidation should avoid O(N) traversal; (iii) carrier switch → long-lived shared vs call-confined separation; (iv) two libraries / one owner → authorization separate from ownership; (v) late trusted C free → cannot catch, documented trust; (vi) session close during active C call → existing admission prevents premature close; (vii) persistence/distribution → never serialize native pointer as ordinary data; (viii) later asynchronous FFM/callbacks → independent explicit lifetime decision, no speculative scheduler.

**Strongest objection to M3:** a checked Protos view may look safe while its caller's native-bound assertion remains unverifiable. The API must make the assumption explicit and preserve the existing catastrophic native-failure disclaimer. An unrestricted raw ctypes API would be simpler but would discard real existing R1 protections for no required current capability.

## 9. GITHUB021 invariant/delta audit

| Previously approved rule | Proposed M3 effect | Classification |
| --- | --- | --- |
| G2 RootActor-only FFM | Only RootActor performs native allocation/read/write/assertion | PRESERVE |
| G2/D259 native OS threads/effects trusted | No promise to confine C or prohibit pthreads | PRESERVE |
| G2 no extra global lock/broker/scheduler | Reuse existing session + native Arena, no process-wide extra service | PRESERVE |
| G2 Native Image exact ABI descriptors | Register only actual downcall shapes, no dynamic full parity promise | PRESERVE |
| P2 ordinary dot slots vs live native at/atPut | Native views only implement existing indexing protocol | PRESERVE |
| P2 group-by-value returns independent | Post-call owned storage, not temporary call Arena | PRESERVE |
| P3-C1 layout separate from prototype | Native region records layout; prototype remains ordinary | PRESERVE |
| R1 owned identity/destructor once | Same custody and logical retirement, no double free | PRESERVE |
| R1 private borrowed copy/epoch | Retain existing private semantics, add optional public views | NEW PUBLIC SEMANTIC DELTA; owner gate |
| R1 multi-Actor FFM | Already intentionally changed by G2 | PRIOR APPROVED DELTA |
| R1 no numeric pointer construction | Keep Integer/address conversion absent | PRESERVE |
| M3 public trusted extent assertion | New deliberate capability over C-origin pointer only | NEW PUBLIC AUTHORITY; owner gate |
| Actor/P isolation & foreign-value identity | Live native capabilities nontransferable, no address equality | PRESERVE |

**GITHUB021 result:** recommendation is *consistent with G2/P2/C1*, and its two new public semantic/authority elements are identified, **not already approved**. Exact memory/pointer API names, acceptable lifetime anchors, read-only policy, deliberate borrow invalidation operation, error precedence, and concurrency details require approval before changing production Java or normative `spec/`.

## 10. Next gates and source-change discipline

1. Ask the owner to select **M3** or another candidate and, separately as needed, the explicit unsafe extent/lifetime assertion.
2. Resolve exact trusted-assertion signature and allowable anchors, pointer field provenance and read-only policy; publish an owner-approved durable P3-M decision only after exact selection.
3. Reconcile `FFI.md`, `MODULES.md`, `ACTORS.md`, `SQLITE.md` and relevant foreign-value/error cross-references. D262/I173 remains independent of native memory; its published product commit is not itself D261 implementation evidence.
4. For every later Java production change cite *exact method/annotation/behavior* in GraalPy `NativeMemory.malloc/free/readByteArrayElements/writeByteArrayElements` or `CStructAccess.readIntField/writeIntField`; TruffleRuby `PointerNodes` `@Specialization` and `FFMSupport.functionDescriptor/createDowncallHandle`; and Java FFM primitives `Arena.allocate/close`, `MemorySegment.asSlice/reinterpret`, `Linker.downcallHandle`. No invented Java infrastructure.
5. Perform bounded C fixture/JVM/Native Image checks, correct error and lifetime tests; report actual results rather than anticipated passes.

**D261 remains OPEN / status:ready. P3-M has NOT been selected or ratified.**
