# D261-R2-P4 — Exact public API and cross-invariant contract packet

**Scope:** Owner requested **“ok cierra el contrato”** after selecting G2/P2/C1/M3/S2/N1. This packet closes the *technical candidate* for the remaining R2 public FFM surface; it is **NOT a claim that the owner separately ratified the newly introduced method names, exact anchor classes, union selector representation or snapshot restrictions**. Formal final approval is still required under `AGENTS.work/DESIGN.md` §§24–110 and GITHUB021. No normative `spec/` or product code is changed here.
**Date:** 2026-11-24.
**Issue:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158) — OPEN, `family:D`, `status:ready`.
**Protos revision inspected:** [`8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Previously owner-selected:** [G2 RootActor architecture](../../decisions/language/D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2 ordinary values versus native regions](../../decisions/language/D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1 separate layout/prototype](../../decisions/language/D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md), [P3-M3 native custody/borrowed/pointers](../../decisions/language/D261_R2_P3_M3_BOUNDED_NATIVE_MEMORY_AND_TRUSTED_POINTER_ASSERTION_SELECTED_2026_11_24.md), [P3-S2 numeric ABI, union and callback deferral](../../decisions/language/D261_R2_P3_S2_SCALARS_UNIONS_SNAPSHOTS_AND_NATIVE_CALLBACK_SCOPE_SELECTED_2026_11_24.md), and [N1 demand-driven Native ABI catalogue](../../decisions/platform/D261_R2_N1_NATIVE_IMAGE_DEMAND_DRIVEN_ABI_ENVELOPE_SELECTED_2026_11_24.md).
**Normative owner:** `guillermomolina/protos/spec/`, especially [FFI.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/FFI.md), [OBJECT_MODEL.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/OBJECT_MODEL.md), [CALLABLES.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/CALLABLES.md), [VALUES_AND_COLLECTIONS.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/VALUES_AND_COLLECTIONS.md), [ACTORS.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/concurrency/ACTORS.md), [MODULES.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/MODULES.md), [SQLITE.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/SQLITE.md), [ERRORS.md](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/spec/semantics/ERRORS.md). [Existing FFI module](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/lib/ffi/module.protos), [C ABI SQLite declarations](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/protos/lib/sqlite/abi.protos), [FFM provider](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java), and [Native registration](https://github.com/guillermomolina/protos/blob/8c3be22391db29bd4ff1abd102884d07d3b2a50e/src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json) were inspected.

## 1. Selectable exact surface — P4-C recommended

`std:ffi` remains an inert import and a Protos-first ordinary MODULE. Existing R1 public operations and descriptors retain their identity and semantics. The following are *new exact candidate names and arities*, subject to explicit owner ratification.

### 1.1 Public immutable descriptors

| Source/API | Parameters | Result | Admission |
| --- | --- | --- | --- |
| `FFI.void/i32/i64/f64/sizeT/utf8z` | none | existing identical singleton descriptors | R1 unchanged |
| `FFI.i8/u8/i16/u16/u32/u64/f32` | none | new singleton frozen scalar descriptors | exact Integer range; explicit f32 narrowing as already approved S2 |
| `FFI.cType(name)` | semantic String | frozen ABI scalar descriptor | initial Linux x86-64 LP64 whitelist below; rejects other names |
| `FFI.field(name, layout)` | semantic String, layout descriptor | frozen field declaration | arbitrary String, including punctuation/reserved words; rejects duplicate names in a group |
| `FFI.struct(fields)` | ordinary Array of field declarations | immutable native struct layout | computes C padding/alignment; nonempty; no overlapping members |
| `FFI.union(fields)` | ordinary Array of field declarations | immutable native union layout | nonempty, overlapping members; no implicit active tag |
| `FFI.array(element, count)` | native layout, positive exact Integer | immutable fixed C array layout | count >=1; total size/stride signed-addressable without overflow |
| `FFI.pointer(pointee)` | any ABI layout or `FFI.void` | nonnullable typed native pointer descriptor | pointer to void has no inferred pointee size |
| `FFI.nullable(pointer)` | nonnullable pointer descriptor | nullable native pointer descriptor | additive R2 case; R1 nullable opaque behavior unchanged |
| `FFI.borrowedPointer(pointer)` | nonnullable pointer descriptor | return-only, epoch-anchored borrowed pointer descriptor | R1 private borrowed mechanism stays intact |
| `FFI.out(descriptor)` | suitable native scalar/pointer/struct/union or R1 opaque/owned | output descriptor | extends R1; no nested out, void, utf8z, or unsupported array-by-value |
| `FFI.opaque/owned/nullable` | existing R1 signatures | existing nominal resource declarations | R1 nominal custody and destructor rules unchanged |

**Field name domain:** semantic Strings exactly as D262-A reflective slots; duplicate field names within one group rejected, array order of declarations retained. Layout objects are identity-associated immutable ordinary frozen Protos objects, not new Core family or Java class per declaration. Structs insert actual C ABI internal/tail padding (JDK `MemoryLayout.structLayout` does not auto-add it); unions overlap storage and are rounded up to required alignment. `array` stride is native element size/alignment, and overflow is checked before representation. Unsupported packed/bitfield/flexible-array/varargs/function-pointer-upcall forms rejected rather than approximated. Pointer to recursive C struct may initially use `pointer(FFI.void)`, followed by explicit bounded `view`; no forward-declaration registry is created.

**Initial cType whitelist, Linux x86-64 LP64 only:** `"int"→i32`, `"long"→i64`, `"long long"→i64`, `"size_t"→sizeT`, `"double"→f64`; only if `Linker.canonicalLayouts()` verifies those keys and widths on the active profile. `cType` does not grant signedness to ambiguous `char`, `_Bool` or enum. All other keys, including `"float"` until independent canonical-key support is verified, are rejected; use fixed `FFI.f32` directly. This deliberately avoids promising arbitrary platform C spellings. The whitelist may later expand through separately verified ABI profiles without modifying existing descriptors.

### 1.2 Layout queries and ordinary prototype registration

| Operation | Result/observable rule |
| --- | --- |
| `FFI.sizeOf(layout)` | exact native layout byte size as Protos Integer |
| `FFI.alignOf(layout)` | exact native required alignment as Protos Integer |
| `FFI.offsetOf(layout, path)` | exact byte offset; path is ordinary Array of String field selectors and Integer array indices (0-based), with all intermediate layout checks |
| `FFI.prototype(layout, ordinaryPrototype)` | returns **same** supplied ordinary object; privately records immutable layout identity keyed by exact prototype identity; same association is idempotent, conflicting association Error |
| `FFI.layoutOf(prototype)` | returns descriptor only for exactly registered prototype identity; no guessing via delegation |

No custom native `call`, `construct`, `init` or native `.member` dispatch is created. Ordinary `Point(...)` continues its existing `call → construct → init` path. `offsetOf` is a query, not pointer arithmetic or native access.

### 1.3 Memory and pointer operations (G2/M3)

| Operation and arity | Semantic result | Preconditions |
| --- | --- | --- |
| `FFI.allocate(layout)` | owned, bounded, initialized-to-zero typed live native region | only RootActor, valid nonvoid layout and size; deterministic independent lifetime; allocation does not implicitly load a C library or grant one |
| `FFI.release(owned)` | R1 exact destructor C result for R1 owned handle; `null` for own R2 region | only canonical owner once; revoke before physical release; view/borrowed pointer cannot be freed |
| `FFI.addressOf(regionOrOwnedHandle)` | non-Integer native pointer capability with original provenance | R2 region has known remaining extent and lifetime; R1 owned handle remains opaque until this **explicit** passage to physical pointer; no implicit release transfer |
| `FFI.slice(region, byteOffset, byteLength)` | non-owning byte-region view with `u8` element indexing | exact nonnegative Integers; overflow, range, access and lifetime checks; never extends original extent |
| `FFI.view(pointer, layout)` | borrowed typed native view over already accredited pointer extent | known sufficient bounds, original provenance/lifetime and alignment; matching typed pointee or explicit void-pointer interpretation |
| `FFI.assumeExtent(pointer, byteLength, anchor, access)` | bounded pointer capability, not an owned region | **deliberately unsafe trusted C ABI assertion**; RootActor only; nonnull C-origin native pointer, positive exact byte length, approved anchor, access exactly `"read"` or `"readWrite"`; does not prove C's actual storage |
| `FFI.bind(library, symbol, returns, parameters)` | fresh ordinary Closure, existing R1 4 arguments unchanged | finite exact descriptor/capability admission and authoritative host library authorization |
| `FFI.bindPreserving(library, symbol, returns, parameters)` | fresh ordinary Closure with trusted **no borrowed-epoch invalidation** claim for that C function | same R1 binding domains, explicit ABI assertion that earlier session-epoch-borrowed data remain valid for this call |
| `FFI.out(descriptor)` | retained R1 result-Array contract `[return,out0,...]` | owned outputs fully adopted before fallible conversion; by-value aggregate outputs are independent owned regions |

**Admissible `anchor`:** (a) a live owned R2 region, with a pointer physically inside its known extent (checkable); (b) a live acquired owned C opaque handle, **trusted** to govern the pointed storage's native lifetime; (c) the same C pointer if it already carries an admitted provider-issued borrowed-epoch provenance from `borrowedPointer` (not a naked self-asserted owner); or (d) an authorized live library facade for *specifically asserted library-static storage*, with the explicit false-assertion risk documented. Plain String, Integer, arbitrary Protos object and fabricated pointer objects are rejected. The effective life is the **intersection** of every existing pointer condition with the new anchor; `assumeExtent` cannot lengthen an expired pointer or undo an older epoch revocation. For C-managed memory, physical validity remains a trusted claim; a library-session anchor does not prove C cannot `free` it.

**Read/write policy:** `"read"` grants no `atPut` or mutable group-byte copy; `"readWrite"` is a deliberate trust assertion, constrained by any pre-existing read-only provenance. Slices/views inherit the stricter permission. No numeric pointer address export or `fromAddress(Integer)`. Null pointer is `null` only through `FFI.nullable(pointer)`; a nonnullable C null is ForeignError upon native result conversion. `FFI.assumeExtent` never accepts `null`.

**Borrowed return and invalidation:** `FFI.borrowedPointer(FFI.pointer(T))` may occur only in return/out descriptors. It supplies an epoch-bearing pass-only pointer, not an invented readable length. Standard R1 private borrowed epochs continue to advance **before admitted non-preserving downcalls**, not before rejected calls; `bindPreserving` declares a trusted exception. New R2 owned regions are not invalidated by unrelated downcalls, and C-owned anchor lifetime and epoch invalidation intersect rather than override one another. At most one owner may run a destructor. Resource-close/session-close revokes descendants even after cleanup failure or C address reuse. No guest-protocol lifetime guarantee against trusted native threads.

**Typed `at`/`atPut`:** struct or union member uses exact String key; fixed array or byte slice uses zero-based Integer index. Primitive read produces canonical Protos Integer/Float; pointer read produces pointer capability retaining source provenance rules, but **not** an assumed extent for the pointee; nested group read produces a non-owning typed native view. Primitive write applies exact scalar admission; group write requires compatible live region and copies that group's bytes (no arbitrary Protos object coercion, no implicit init). A successful native `atPut` returns the exact value supplied, matching ordinary indexed behavior. All bounds/alignment/access checks occur before bytes are touched. Union member key selects a bit interpretation explicitly, not an active tag; independent native C writes may race under `TRUSTED_IN_PROCESS`.

### 1.4 Explicit field/value projection (S2-H/U + D262)

| API | Inputs and result | Admission/behavior |
| --- | --- | --- |
| `FFI.unionArm(path, memberName)` | `path` = Array of exact String and 0-based Integer indexes identifying one union location; `memberName` exact String declared in that union; returns frozen selection declaration | no concatenated dot path (fields may themselves contain dots), no inferred tag |
| `FFI.readFields(region, arms)` | reads one compatible bounded native group; `arms` ordinary Array of selection declarations; returns fresh ordinary Map tree (nested structs→Map, fixed arrays→Array) | exactly one selection per reached union path, no duplicate/unused selections; only selected union member appears |
| `FFI.writeFields(region, fields, arms)` | writes from ordinary Map/Array tree, returns exact `region` | exact required member/index shape, extras/missing elements Error; **stage/validate all** data and arm choices before any native write, then copy; not atomic against trusted C threads |
| `FFI.readAs(prototype, region, arms)` | copy into fresh ordinary child whose parent is the exact registered prototype; no `call/construct/init` activation | uses ordinary dynamic slot creation (D262); nested structs ordinary children of `Object`, arrays normal Arrays; no implicit view or native getter; canonical reflective method must be reachable before insertion |
| `FFI.writeFrom(region, value, arms)` | copies exact ordinary local slot values into matching region, returns `region` | only a value with exact registered prototype as **immediate delegation parent** and usable canonical ordinary local-slot reflection; otherwise Error before bytes change; use `writeFields` for arbitrary objects/colliding methods |

**Public snapshots are copies.** `FFI.readFields`/`writeFields` are the **universal** representation for any C field names, including `call`, `init`, `createSlot`, `slotValue` and non-lexable strings, because Map keys are indexed associations, not slots. `readAs` is a convenience for admitted ordinary prototypes, implemented through existing D262 ordinary `createSlot` (bind it before inserting any field that shadows its name). If a prototype already overrides that method, reject *before constructing/publishing a result*. For `writeFrom`, if canonical local-slot reading cannot be shown through ordinary messages due to a shadowed method, fail before any native write rather than inventing privileged Java reflection. The alternative `writeFields` remains available, and **no automatic conversion of a C pointer or union to Protos object occurs**. A `readAs` snapshot with a local slot named `call` may itself be uncallable thereafter, exactly under `CALLABLES.md`.

**Snapshot guarantee limit:** no FFI convenience may suppress errors/side effects from ordinary user methods that it is actually permitted to invoke. The universal Map path avoids the reflective prototype problem entirely. Native writes are validated and staged before beginning to mutate C memory; user-defined Map keys/equality may execute ordinary guest code during staging, but that happens before native writes, not under a hidden kernel bypass.

### 1.5 Binding admission and result semantics

- Scalar R1 descriptors remain exact. New `iN/uN` inputs are exact Protos Integers in signed/unsigned width, `u64` returns may be BigInteger, and `f32` alone intentionally rounds Float binary64 to IEEE binary32 nearest/ties-even before passing C, re-admitting a semantic Float binary64 on output while preserving signed zero/NaN semantics. No implicit Integer-to-Float.
- Return by value: `struct` or verified `union` returns independent R2-owned typed region, *not* a Protos object and not memory from a call-confined Arena. Use native ABI group return `SegmentAllocator` behavior. By-value input `struct/union` requires compatible live region and actual C ABI support; C array direct by-value and unsupported union classification are rejected. `out(struct)` captures an independent owned region while preserving the R1 `[return,out0,...]` ordering.
- FFI pointer parameters accept matching physical pointer capabilities/owned region addresses only through explicit `addressOf`; must preserve original owner and revocation. R1 `opaque` nominal library/session checks remain intact for callers that explicitly declare an R1 opaque parameter. A new pointer argument does not grant library load permission, create a new destructor or transfer memory ownership.
- C status results are ordinary data; native output adoption always precedes fallible conversion, and session close drains unclaimed adopted resources. `FFI.bind` always returns a **fresh ordinary Closure**, even when a native call handle is cached. No callback/upcall, implicit await, C-thread entry or native trampoline is installed by R2.

### 1.6 Errors and temporal ordering

1. Normal Protos receiver/argument evaluation and Closure named/positional association take place first, including all existing side effects/Errors. Invalid arity/extras/names/types and lookalike descriptors fail as ordinary `Error`.
2. A library load requires exact host `--allow-library` authorization **before** Linker/Arena/library initialization, as R1/D259 prescribe. `import("std:ffi")` remains inert; non-RootActor native acquire/allocate/read/write/assert/release is rejected before entering foreign/native operations. No general restrictions on unrelated foreign providers.
3. Declarative layout/signature validation, Native Image *exact descriptor/options admission*, ownership/epoch/access/liveness, bounds, alignment and input values are checked **before native entry** and signal ordinary Error on failure, without advancing a borrow epoch or changing native bytes. A missing Native registration is not delegated to `MissingForeignRegistrationError` nor faked as a pointer signature.
4. Full `writeFields/writeFrom` candidate and union-path preflight occurs before **any** native write. No rollback promise for independently running native C threads or user effects performed earlier while evaluating arguments.
5. After complete call admission, advance relevant epoch before a non-preserving entered downcall or destructor. Then enter trusted C; after return, capture **all** native outputs, adopt every owned native resource before any fallible conversion and apply R1 Array result ordering.
6. Failure of an *entered foreign operation* such as native symbol resolution, native conversion or cleanup is fresh `ForeignError` under existing D188; public invalid admission is ordinary Error. A nonzero C status remains data. Native physical faults/memory corruption may terminate the process, as G2 says. Duplicate release rejects before any additional native destructor; no retry after uncertain cleanup. When a later conversion fails, previously adopted owners remain in custody.
7. RootActor temporal execution may move between host threads, so persistent R2 regions cannot be stored in call-confined Arenas; preserve existing per-session active-call closure and independent owned region lifetimes, with no new global native broker/lock.

## 2. GITHUB010: exact public API alternatives, proportionality and 12 axes

| Candidate | Model | Decisive issue |
| --- | --- | --- |
| P4-A | `readAs/writeFrom` and Protos objects only | Cannot faithfully support all native field names/reflection overrides |
| P4-B | raw `ctypes`-style addresses and implicit native properties | Contradicts P2/M3, allows forged address and undetectable UAF |
| **P4-C recommended** | bounded native region `at/atPut`, independent ordinary prototype, universal Map projection and admitted object convenience | Pays for explicit projection and a clearly unsafe C extent assertion, but preserves invariants |
| P4-D | per-C-library Java adapters/class generators | Duplicates infrastructure, violates generic G2 and N1 pay-as-you-grow |

Score: 1 poor to 5 strong; H high/M medium/L low confidence, reasoning in every cell.

| Criterion | A objects only | B raw ctypes | C explicit region+Map+object | D library adapters |
| --- | --- | --- | --- | --- |
| 1. Correctness | 2M: name collisions | 1H: pointer and union guesses | 5M: explicit bounds/arms | 3M: library-specific truth |
| 2. Protos semantics | 3M: native conversion only | 1H: magic foreign members | 5M: dot/at separation | 2M: provider-specific APIs |
| 3. Cost now | 4M: small facade | 3M: small initial API, large risks | 4M: new methods only on FFI | 2H: adapter proliferation |
| 4. Growth | 2M: cannot represent all C names | 3M: unbounded unsafe syntax | 5M: add ABI shapes as needed | 2M: one adapter per library |
| 5. Future options | 2M: reflectability lock-in | 3M: maximal but hard to retract | 5M: stable capability model | 2M: duplicated semantics |
| 6. Scale | 2M: per-field reflective calls | 4M: fast, unmanaged failures | 4M: typed native views | 2M: shims and copies |
| 7. Simplicity | 3M: apparent, exceptions | 2M: implicit semantics | 4M: explicit surfaces | 2M: many institutions |
| 8. Portability | 3M: guest-only | 2H: numeric pointers OS-dependent | 4M: exact C profiles | 3M: packages platform-specific |
| 9. Resource cost | 3M: copied snapshots | 4M: minimal checks, high hazard | 4M: owner-root lifetimes | 2M: generated glue |
| 10. Errors | 2M: overridden slots can lie | 1H: UAF and leak | 4M: pre-entry checks | 3M: errors inconsistent |
| 11. Reversibility | 2M: retrofit Map later | 1H: unsafe API permanent | 5M: additive exact API | 2M: retire adapters later |
| 12. Evidence | 3M: ordinary Protos | 4H: ctypes/LuaJIT precedent | 5M: R1+Java25+TruffleRuby+GraalPy | 3M: jextract precedent |

**Non-compensating red flags:** P4-A lies about shadowed names/overridden reflection; P4-B conflicts with an owner-approved no-Integer-pointer invariant (cannot be bought by simplicity or performance); P4-D reintroduces per-library Java infrastructure in place of a generic source-first FFM, contradicting G2/N1. P4-C is not a full native memory safety system and must never claim one. **Strongest objection to P4-C:** it expands the public MODULE beyond R1 and introduces Map projection even for simple ordinary structs; the exact mitigation is that `readAs/writeFrom` remain ergonomic, while Map is optional and only paid by callers that need arbitrary field names/collision immunity.

**Survey:** Java 25 `MemoryLayout.structLayout/unionLayout`, `Linker.downcallHandle` group return and `SegmentAllocator`, `MemorySegment.reinterpret/asSlice`, GraalVM 25 Native `foreign.downcalls` exact registration; pinned TruffleRuby `FFMSupport.functionDescriptor/createDowncallHandle`, `PointerNodes @Specialization` and struct layout source; GraalPy `CStructAccess.readIntField/writeIntField`, `NativeMemory.malloc/free`, `GenerateNativeDowncallsProcessor.collectDowncalls/generateInvoker`; CPython ctypes `Structure/Union/from_buffer/from_buffer_copy`, LuaJIT FFI, Rust `repr(C)` unions/`from_raw_parts`, Zig extern structs/unions and .NET P/Invoke explicit field offsets/SafeHandle. Compared earlier D261 P3-M and P3-S 10-system research records rather than repeating settled architecture selections. Java 25 [MemoryLayout](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/MemoryLayout.html), [Linker](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Linker.html) and [GraalVM 25 Native FFM](https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/) are normative external technical constraints, not Protos semantic owners.

**Pay only for current use:** no eager FFM on import, no new guest scalar class, no Java C-class generator, no per-view Arena, no callback stub registration, no universal C header parser or function-pointer trampoline, no Native ABI Cartesian-product registration. **Grow as needed:** add ABI signatures under approved N1 based on actual integrated fixtures and exact registered descriptors; an authorized unrelated C library may reuse a compiled signature, without an additional library allowlist. Dynamic JVM valid descriptors remain possible when the backend supports them. **Deferral costs:** delaying an explicit lifetime anchor, typed region and union path after publishing unsafe casts would force backward-incompatible memory rules; deferring callbacks/bitfields/packed/variadics is reversible because R2 explicitly declares them unsupported. **Future scenarios:** large native arrays, many slices with O(1) generation invalidation, original-owner destructor after cross-library use, secondary Actor rejection, RootActor migrating host carrier, native C racing Guest reads, mismatched LLP64/LP64, Native Image missing group metadata and independent maps/snapshots.

## 3. GITHUB021 approved-invariant delta table

| Owner-approved/normative invariant | P4-C effect | Finding |
| --- | --- | --- |
| G2 RootActor-only native FFM, trusted in-process C effects, no extra scheduler | Whole native public surface restricted to RootActor, no new scheduler | PRESERVED |
| D259 alias allowlist host-only, default deny and lazy load | Build ABI registration never grants runtime library authorization | PRESERVED |
| P2 ordinary `.field` versus live native `["field"]` | Distinct, explicit projection | PRESERVED |
| P3-C1 separate layout descriptor and original ordinary prototype identity | `FFI.prototype` retains same object and immutable associated layout | PRESERVED |
| P3-M3 owned roots, borrowed views, opaque pass-only pointers, RootActor trusted extent | `allocate/addressOf/view/slice/assumeExtent` with real anchor and explicit access | PRESERVED; NEW EXACT NAMES/ANCHORS REQUIRE APPROVAL |
| P3-S2 exact Integer widths, f32-only ABI rounding, union active member, callback deferral | Exactly retains all choices | PRESERVED |
| N1 existing generic Native provider + demand-driven registered ABI | Non-integrated host-authorized library reuses registered shapes; no default new shapes | PRESERVED |
| R1 ABI/Closure/out/owned/borrow/identity/error lifetime | Additive descriptors and public borrowed view; preserve legacy rules | PRESERVED / DISCLOSED R2 EXTENSION |
| D262 normal reflectivity and method shadowing | Universal Map route, ordinary object conversion only when admitted | PRESERVED; NEW CONVENIENCE LIMITATION REQUIRES APPROVAL |
| ACTORS/VALUES callback and nonlocal return contracts | No C upcalls exposed in initial R2 | PRESERVED |
| Native/portable differences | Native exact registered set, JVM runtime-composable subset | PRESERVED |

**New P4-only substantive choices — approval gate:** 1) exact public operation names and four-argument `assumeExtent`; 2) accepted anchors and explicit mode values; 3) `bindPreserving/borrowedPointer` session-epoch public declarations; 4) union `unionArm(path,member)` Array-of-path and Map projection; 5) canonical reflection admission and `writeFrom` restrictions; 6) initial `cType` whitelist and allocation/out parameter extensions; 7) exact error priority. These **must not be treated as owner-approved merely because G2/P2/C1/M3/S2/N1 were approved, or because the owner asked to “close the contract.”** A decision packet is complete; exact-candidate ratification awaits an explicit owner statement or clear bounded delegation under AGENTS.

## 4. Independent falsification/test plan (actual execution NOT RUN)

| Area | Required probes |
| --- | --- |
| Descriptors | R1 six singleton identities + six Closures unchanged; new i8/u8/.../u64/f32 range and NaN/signed-zero cases; fake descriptor identities rejected; `cType` whitelist |
| Layout | independent C `sizeof`, `_Alignof`, `offsetof`, nested array stride/tail padding, union field overlap, unsupported bitfield/packed/varargs rejection |
| Typed memory | zero initialized root, region/slice/view and u8 byte slice bounds/alignment, access-mode enforcement, view invalidated after root release, reused C address never revives stale view |
| Pointers | pointer return zero extent passable not dereferenceable, null refusal, full trust-anchor class matrix, lifetime intersection/epoch cannot be widened, cross-library physical pointer accepted with both libraries separately host-authorized |
| Binding | `bind` fresh Closure, return by-value group with post-call allocator, `out` Array result order and owned adoption before fallible conversion, no native status magic |
| Borrow epochs | invalidating versus `bindPreserving` call, pre-entry rejection no epoch advance, R1 16MiB *private copy only*, independently owned region survives unrelated C calls |
| Projections | nested unions require exactly one `unionArm` path, arbitrary field String keys, Map roundtrip, field named `call/init/createSlot/slotValue`, already overridden reflection rejected or Map path, no partial native write after type failure |
| Errors | ordinary Error for pre-entry invalidity, ForeignError for entered resolution/result/destructor failure, duplicate release no retry, wrong RootActor denied |
| Native | finite `foreign.downcalls` coverage, non-registered group fails before C, registered group/float ABI fixture succeeds, reused registered shape works for another allowlisted library, idle import inert, no bundled .so |
| Lifecycle | active downcall prevents premature region/session teardown, Actor/P transfer rejection, no native upcalls or callback retention, Native Image distribution and JVM parity within registered shape |

**Source-level implementation precedents required:** GraalPy `NativeMemory.malloc/free/readByteArrayElements/writeByteArrayElements`, `CStructAccess.readIntField/writeIntField`, `GenerateNativeDowncallsProcessor.collectDowncalls/generateInvoker`; TruffleRuby `FFMSupport.functionDescriptor/createDowncallHandle` and `PointerNodes @Specialization`. Every new production Java method needs its own method/annotation/behavior citation before entering a human-executor prompt; no ad-hoc global state, Java C-class factory, alternative guest invocation or foreign-thread dispatcher.

## 5. Next workflow

1. Ask for exact owner ratification of **P4-C** together with the disclosed *new* API/anchor/Map/epoch/Native details, or request bounded edits to it; neither prior G2/P2/C1/M3/S2/N1 approval nor publication of this report alone selects P4-C.
2. After ratification, publish selected decision at `docs/project/decisions/language/` and reconcile normative `spec/semantics/FFI.md`, `MODULES.md`, `SQLITE.md`, `ACTORS.md`, `ERRORS.md`, and only necessary Object/Collections/Callable cross-references. Product spec source editing/publication remains human-executed under the product `AGENTS.md`.
3. Split implementation into independent executable issues: descriptors/layout and C ABI; owned regions/pointer/epoch; group return and Native Image exact registration; guest Map/object snapshots and mismatch cases; final real JVM/Native conformance. Release/version/CHANGELOG only after actual tests.
4. Keep D261 **OPEN, status:ready** until normative publication and issue-closure evidence. Current document publication is decision research, not normative language authority.

**Provenance status:** `EXACT_API_CANDIDATE=P4_C_READY_FOR_OWNER_RATIFICATION`, `PRODUCT_SPEC_CHANGED=NO`, `PRODUCT_JAVA_CHANGED=NO`, `GIT_DIFF_CHECK=NOT_RUN`, `COMPILATION=NOT_RUN`, `TESTS=NOT_RUN`, `NATIVE_IMAGE=NOT_RUN`, `PERFORMANCE=NOT_MEASURED`. This avoids inventing acceptance evidence.
