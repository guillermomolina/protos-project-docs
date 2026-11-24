# D261-R2-P4-C — Exact public generic FFM contract selected

**Status:** OWNER-SELECTED EXACT PUBLIC CONTRACT (2026-11-24); formal D261 issue remains OPEN for normative publication and implementation conformance.
**Owner:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158) in `guillermomolina/protos`.
**Explicit owner approval:** After the exact P4-C packet was presented, the owner responded verbatim **“apruebo P4-C”** and directed issue/evidence publication. [Owner-selection coordination record](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7129273923).
**Revision-pinned exact candidate and GITHUB010/GITHUB021 evidence:** [D261-R2-P4-C — exact public contract, alternatives, full API signatures and invariant audit](https://github.com/guillermomolina/protos-project-docs/blob/e1b668d37f8e3973dab5081adec52f290fd28880/docs/project/evidence/D261/D261_R2_P4_EXACT_PUBLIC_CONTRACT_GITHUB010_GITHUB021_REVIEW_2026_11_24.md). Approval covers the **complete** exact public candidate (P4-C), **not** just the operation names enumerated below.
**Product revision inspected by the approved packet:** [`guillermomolina/protos@8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Authority:** This decision record is durable and **non-normative**. Public normative semantics belong in `guillermomolina/protos/spec/` after separately authorized specification edits. Repository publication here does not constitute implementation or testing.

## 1. Ratification scope

The owner explicitly approved **P4-C**, following the earlier owner-selected six design components: [G2 native RootActor authority](D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2 ordinary values/native-region split](D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1 layout/prototype separation](D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md), [P3-M3 native memory/provenance](D261_R2_P3_M3_BOUNDED_NATIVE_MEMORY_AND_TRUSTED_POINTER_ASSERTION_SELECTED_2026_11_24.md), [P3-S2 scalars/unions/callback deferral](D261_R2_P3_S2_SCALARS_UNIONS_SNAPSHOTS_AND_NATIVE_CALLBACK_SCOPE_SELECTED_2026_11_24.md), and [N1 Native Image demand-driven ABI envelope](../platform/D261_R2_N1_NATIVE_IMAGE_DEMAND_DRIVEN_ABI_ENVELOPE_SELECTED_2026_11_24.md).

The complete P4-C signature tables and their preconditions in the revision-pinned evidence are **ratified as a single exact candidate**, including these material newly selected choices:

- New public operation **names and arities**, immutable descriptor construction and verified canonical C scalar type keys;
- Explicit four-argument trusted `assumeExtent`, the exact anchor categories and access-mode Strings `"read"` and `"readWrite"`, with intersection of existing and asserted lifetime/permission;
- Public `borrowedPointer` / `bindPreserving` declarations and R1-compatible borrowed-epoch rules;
- Union selection by `unionArm(path,memberName)` with an unambiguous Array of field names/indexes;
- Universal nested Map/Array field conversion and conditional ordinary Object snapshot conversion;
- Exact admission/error ordering, preserved existing R1 `out`/owned-resource contracts, RootActor gate and finite Native Image signature coverage.

**No** general numeric address forging, silent native dot-property dispatch, privileged Java slot reflection, new numeric/Core value family, C-to-Protos upcall/retained callback, hidden per-library Java backend or new global FFM scheduler is authorized.

## 2. Public API inventory selected

**All names in these tables are the selected P4-C source-level public contract; implementation is pending.**

### 2.1 Immutable C ABI declarations and lookup

| Selected API | Selected result and admission |
| --- | --- |
| `FFI.void/i32/i64/f64/sizeT/utf8z` | Exact existing R1 immutable scalar descriptors; retain identities and rules |
| `FFI.i8/u8/i16/u16/u32/u64/f32` | New immutable ABI scalar descriptors; integers exact within descriptor bounds, `f32` binary64→binary32 nearest/ties-to-even narrowing deliberately opted into only here |
| `FFI.cType(name)` | Supported, verified Linux x86-64 LP64 canonical C scalar keys only: `"int"`, `"long"`, `"long long"`, `"size_t"`, `"double"`; other names fail until separately verified |
| `FFI.field(name, layout)` | Immutable named field declaration; exact semantic String |
| `FFI.struct(fields)` | Immutable C ABI struct descriptor from ordinary Array of distinct named fields; actual internal and tail padding, no automatic packed/bitfield behavior |
| `FFI.union(fields)` | Immutable C ABI union layout with overlapping storage, no inferred active member |
| `FFI.array(element, count)` | Fixed positive-length native array, checked size/stride and addressable range |
| `FFI.pointer(pointee)` | Nonnullable typed native pointer descriptor, including pointer-to-void |
| `FFI.nullable(pointer)` | Nullable pointer descriptor; additive to legacy nominal R1 `nullable(opaque)` |
| `FFI.borrowedPointer(pointer)` | Return/out-only borrowed epoch-provenanced pointer descriptor |
| `FFI.out(descriptor)` | Retains R1 output position semantics with approved R2 pointer/group extensions |
| `FFI.opaque/owned/nullable` | Original R1 nominal library-specific custodial descriptors unchanged |

No arbitrary C `char` signedness, `_Bool`/enum, `long double`, packed struct, flexible-array, variadic C ABI or callback support is inferred from declaring an ordinary layout. All unverified forms are rejected before C entry. Correct target `sizeof`, `_Alignof`, `offsetof` and group-by-value classification require independent native C fixture conformance.

### 2.2 Queries and ordinary Protos prototypes

| Selected API | Selected result and admission |
| --- | --- |
| `FFI.sizeOf(layout)` / `FFI.alignOf(layout)` | Protos exact Integer layout byte size/alignment |
| `FFI.offsetOf(layout, path)` | Checked byte offset; `path` is an Array of exact semantic Strings for fields and zero-based Integers for array positions |
| `FFI.prototype(layout, ordinaryPrototype)` | Associates a fixed ABI descriptor by exact prototype identity; returns **same ordinary prototype**; identical reassociation is idempotent, conflicting layout association fails |
| `FFI.layoutOf(prototype)` | Only the directly registered prototype's layout, not a guessed ancestor layout |

Normal Protos `call → construct → init`, delegation and local slots remain untouched. No Java class/native constructor per C descriptor is created.

### 2.3 Native memory, pointer and lifetime capability

| Selected API | Selected result and admission |
| --- | --- |
| `FFI.allocate(layout)` | RootActor-owned zero-initialized typed native region with an independently releasable lifecycle; does not implicitly acquire a C library |
| `FFI.release(owned)` | For R1 owned handles, retain exact native destructor result; for an R2 allocated root return `null`. Retires once, invalidates descendants before physical release |
| `FFI.addressOf(regionOrOwnedHandle)` | Non-Integer native pointer capability, retaining original custodian/provenance; address passage does not grant destructor rights |
| `FFI.slice(region, byteOffset, byteLength)` | Non-owning u8 byte-view constrained by source extent and permissions |
| `FFI.view(pointer, layout)` | Typed non-owning view requiring accredited size, layout compatibility, alignment, valid generation and access rights |
| `FFI.assumeExtent(pointer, byteLength, anchor, access)` | RootActor-only **explicit trusted** bounds/lifetime declaration on an admitted nonnull C-origin pointer, with positive exact byte length and `access` exactly `"read"` or `"readWrite"`; never a physical proof |

**Admissible anchors only:** (1) a live R2 owned native region with a demonstrably in-bounds physical pointer; (2) an admitted live C-owned opaque native resource, with a deliberately trusted assumption about its pointee; (3) provider-issued borrowed-epoch provenance carried by the C pointer itself; (4) a live authorized library facade for explicitly asserted library-static storage. Arbitrary guest String, Integer and ordinary object are not anchors. Never extend an expired pointer's life or original permissions. Views/borrowed pointers cannot independently free the original storage; source generation, borrow epoch, owner retirement and session close revoke them. Reuse of a machine address never reactivates a stale token.

**Typed `at/atPut`:** String keys select a struct/union field; Integers index a fixed array/byte slice. Nested aggregate reads yield live *borrowed* native views; pointer fields yield a pass-only capability unless their pointee extent is independently known or explicitly asserted. Valid writes to individual fields follow the selected exact scalar admission and return the exact supplied value. `read`-only views reject mutation. There is no hidden `.field` native routing. C is still **TRUSTED_IN_PROCESS**, so an inaccurate lifetime assertion or native thread can cause actual memory corruption or process termination.

### 2.4 Explicit transfers, Map/Array projection and union selection

| Selected API | Selected behavior |
| --- | --- |
| `FFI.unionArm(path, memberName)` | Frozen selection: `path` = Array of exact String/Integer keys locating the union; `memberName` is one declared arm |
| `FFI.readFields(region, arms)` | Detached ordinary Map (nested struct→Map, fixed arrays→Array); exact one selected member per reached union |
| `FFI.writeFields(region, fields, arms)` | Validates full ordinary Map/Array data, union selections, ranges and permissions **before** native writes; returns exact region |
| `FFI.readAs(prototype, region, arms)` | Detached ordinary object child of exact registered prototype; no `call/construct/init`. Uses reachable canonical D262 `createSlot` |
| `FFI.writeFrom(region, value, arms)` | Copies *local* fields from an exact admitted ordinary child of the registered prototype; rejects preflight failure when canonical ordinary reflection is inaccessible |

**Map is universal**, including fields named `call`, `init`, `slotValue`, `createSlot` or non-source-identifier strings, and cannot be replaced by a privileged Java slot reader. Ordinary object convenience has an explicit narrower admission: methods like `createSlot/slotValue` may be overridden by the prototype or shadowed by local data. When the canonical ordinary reflective route cannot be trusted, reject before native mutation and use the Map/Array path instead. Even a successful `readAs` snapshot may become normally uncallable if it gets a non-Closure `call` slot; that is standard Protos behavior.

No union member is guessed or auto-tagged. For every reached nested union `arms` supplies exactly one valid active path; missing, duplicate or extraneous paths fail before copying or writing. The conversion is a snapshot, not a live native object alias.

### 2.5 Downcalls, R1 compatibility and Native Image

| Selected API | Contract |
| --- | --- |
| `FFI.bind(library,symbol,returnDescriptor,argumentDescriptors)` | **Existing exact four-argument R1 API**; answers fresh ordinary Closure, retains nominal native handles and ownership; now accepts verified R2 scalar, group and pointer descriptors |
| `FFI.bindPreserving(library,symbol,returns,parameters)` | Fresh ordinary Closure plus explicitly trusted declaration that this call preserves session epoch-governed borrowed buffers |
| `FFI.out(descriptor)` | R1 return convention retained: multiple outputs form fresh `[return,out0,...]` Array; all acquired native resources adopted *before* any fallible value conversion |

C aggregate-by-value results are independent, owned typed native regions with a result allocator surviving the native call. Signatures for these aggregates are real C group layouts, not pointer-shaped substitutes. Native Image uses the selected N1 **demand-driven, exact build-time ABI signature envelope** while the JVM may construct other backend-valid signatures. Another host-authorized library can reuse a registered physical shape even if it lacks an official Protos integration. `--allow-library alias=target` remains **separate host-only authorization**, default-deny and lazy; no C library binary is embedded. Native C callback/upcall facilities remain explicitly deferred.

### 2.6 Ordered error/lifecycle rules

1. Ordinary source receiver/argument evaluation and Closure binding remain first; evaluation side effects retain normal semantics.
2. Structural/ABI validation, descriptor identity, RootActor authority, Native Image exact registration, borrowed epoch/liveness, layout/alignment/bounds/access, and input scalar ranges reject invalid states **before native entry** with ordinary `Error`, without calling C or advancing borrow epochs.
3. A native library load/lookup obeys host `--allow-library` before Linker/Arena/symbol acquisition. `import("std:ffi")` stays inert and is not authority for a C library.
4. `writeFields/writeFrom` stage and validate **all** values/arms before any native write; this is no rollback of already-completed guest argument evaluation and no promise of atomicity against trusted concurrent native code.
5. After complete call admission, non-preserving downcalls advance the appropriate epoch before entered C; preserving calls require an explicit `bindPreserving` claim. R1 private borrowed-copy maximum of 16 MiB is **not** a global R2 allocation limit.
6. After entered C returns, adopt all owned results before fallible conversion. Entered foreign failures use the existing `ForeignError` category; nonzero C statuses remain ordinary data. Logically retired owners are never destructed twice, including after uncertain cleanup. Native physical faults are outside guest guarantees.
7. RootActor native FFM capabilities, regions, views and pointers are nontransferable across Actor and P boundaries. No general C-thread guest ingress, scheduler, additional Java interpreter or global FFM dispatcher is introduced.

## 3. GITHUB021 invariant and approval audit

| Source of authority | Result |
| --- | --- |
| G2 RootActor-only native entry, host-native `TRUSTED_IN_PROCESS`, no special scheduler | PRESERVED |
| P2 ordinary `.field` versus native-region `at/atPut` | PRESERVED |
| P3-C1 ABI layout/prototype distinct identity and ordinary invocation | PRESERVED |
| P3-M3 owner+borrowed provenance, explicit trusted unknown-extent C pointer | PRESERVED; **exact public names/anchor/access domains now owner-approved P4-C** |
| P3-S2 Integer exactness, opt-in f32 narrowing, union selection, callback deferment | PRESERVED |
| N1 generic Native provider and evidence-led ABI additions, independently host-authorized libraries | PRESERVED |
| R1 `FFI.bind` Closure, `out` Array, nominal opaque, owned custody, error categories | PRESERVED/ADDITIVE |
| D262 ordinary local reflection, slot naming and shadowing, `CALLABLES.md` extracted closures | PRESERVED; no privileged bypass |
| General numeric values, Actor/P separation and no attached C-created threads | PRESERVED |
| P4-C newly selected exact names, `bindPreserving`, unionArm path, Map projection, accepted anchors and modes | **EXPLICITLY OWNER-APPROVED in this turn** |
| Normative `spec/` language contract and implementation source publication | **NOT YET PERFORMED**; this record cannot masquerade as the specification |

**Approval provenance check:** the user selected **P4-C** by exact candidate identifier following the complete surfaced proposal and verified project evidence. The approval covers the candidate as it was presented; it does **not** authorize changing other closed decisions, claiming test passes or silently adding unsupported C ABI features.

## 4. Remaining work and verification status

- Normative reconciliation remains to be published by the authorized product workflow: `spec/semantics/FFI.md` (primary owner), `MODULES.md`, `SQLITE.md`, `ERRORS.md`, `concurrency/ACTORS.md` and only necessary `OBJECT_MODEL.md`, `CALLABLES.md`, `VALUES_AND_COLLECTIONS.md` cross-references.
- Production implementation must be sliced by a human-executor work plan into descriptor/layout ABI, bounded owned/borrowed native memory, call group ABI/Native Image exact registration, Map/Object snapshot adapter and independent C fixture conformance. Cite **exact production methods/annotations** in GraalPy/TruffleRuby before any new Java production code; no invented Java infrastructure, global lock, Java class per C struct or alternative foreign callback scheduler.
- Run **actual** compilation, native fixtures, R1 regressions, JVM and installed Native Image distribution tests with explicit evidence and negative cases (unsupported signature, owner reuse, reflected names, epoch invalidation, union nested fields, lack of native callbacks). Do not infer those passes from the current record.
- Issue [D261](https://github.com/guillermomolina/protos/issues/1158) remains **OPEN / `status:ready`** for those concrete remaining gates. Neither [PLAT077](https://github.com/guillermomolina/protos/issues/1146) nor D262/I173 is reopened by this approval.

```text
D261_R2_P4_C=OWNER_APPROVED
EXACT_PUBLIC_API=OWNER_SELECTED
D261_NORMATIVE_SPEC=NOT_PUBLISHED
D261_PRODUCTION_IMPLEMENTATION=NOT_PERFORMED
COMPILATION=NOT_RUN
TESTS=NOT_RUN
NATIVE_IMAGE=NOT_RUN
PERFORMANCE_MEASUREMENTS=NOT_RUN
GIT_DIFF_CHECK=NOT_RUN
GIT_PUBLICATION_PROJECT_DOCS=AUTHORIZED_AND_PERFORMED
```

The last publication line describes only this durable documentation publication, and does **not** assert a product checkout state, local clean diff or normative publication.
