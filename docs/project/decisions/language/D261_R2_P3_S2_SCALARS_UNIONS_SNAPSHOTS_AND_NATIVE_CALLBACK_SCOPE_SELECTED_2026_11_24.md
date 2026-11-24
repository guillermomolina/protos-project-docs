# D261-R2-P3-S2 — C scalar ABI, unions, ordinary snapshots and callback scope

**Decision:** OWNER-SELECTED S2, 2026-11-24. Exact public signatures and normative source publication remain pending.
**Owner work item:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
**Approval provenance:** [Recorded explicit owner acceptance](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7128811603). The owner answered **“ok lo apruebo”** directly to a concise recommendation to approve S2 as a whole, explicitly covering f32 narrowing, union selection, ordinary slot semantics and callback deferral.
**Research provenance:** [P3-S full GITHUB010/GITHUB021 evidence](../../evidence/D261/D261_R2_P3_S_GITHUB010_GITHUB021_SCALARS_UNIONS_CALLBACKS_RESEARCH_2026_11_24.md), published at [revision `e859c083ef7bcc8b46330aad4c311a5b7067a87a`](https://github.com/guillermomolina/protos-project-docs/commit/e859c083ef7bcc8b46330aad4c311a5b7067a87a), comparing six candidates across twelve criteria, ten ecosystems and three design routes.
**Product revision audited for the underlying recommendation:** [`8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Preserved earlier decisions:** [G2](D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2](D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1](D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md), [P3-M3](D261_R2_P3_M3_BOUNDED_NATIVE_MEMORY_AND_TRUSTED_POINTER_ASSERTION_SELECTED_2026_11_24.md).
**Authority:** non-normative durable project decision. The normative language owner remains `guillermomolina/protos/spec/`.

## 1. Owner-selected S2: the explicit F/U/H/C contract

### S2-F — Fixed-width signed and unsigned native scalars

The public R2 ABI includes native signed and unsigned integer widths **8, 16, 32 and 64**, the existing `sizeT` domain, and `f32/f64`. Existing R1 `void/i32/i64/f64/sizeT/utf8z` rules are retained. Exact Protos Integer values alone are accepted for integer input, in the mathematical interval specified by signed or unsigned width. Wrong numeric family and overflow are errors before native entry, never wrap, saturation, text parsing or silent coercion. Native integer results normalize to semantic Protos Integer, so `u64` values beyond signed-64 are BigInteger, not negative host longs.

**Explicit, new, owner-approved R1 exception:** the newly selected native `f32` descriptor admits a semantic Protos Float (binary64) and deliberately narrows it to IEEE 754 binary32 using nearest/ties-to-even rounding as an ABI conversion. Native `float` output widens to semantic binary64 Protos Float, preserving signed zero, infinity and NaN semantic identity (not distinct NaN payloads). `f32` does not perform Integer-to-Float coercion. This is a deliberate *typed*, *opt-in* exception to R1's universal “inputs are never rounded” rule, **not** a replacement of that rule for `f64`, `iN`, `sizeT` or any previous descriptor.

Only C-native type spellings with **verified** supported `Linker.canonicalLayouts()` and correct semantic conversion may later be exported. No blind assumptions about signed `char`, `_Bool`, native enum widths, `wchar_t`, `long double`, unknown bit-widths, Windows LLP64 or non-x86-64 ABI profiles. The exact public `cType` naming/table and validated platform-specific cases remain a later API/spec gate. No Core `Int`, `UInt` or Float32 family is added.

### S2-U — Structs/unions/arrays and explicit union member selection

Composed ABI layouts must match the target C calling and memory convention: exact size, `_Alignof`, `offsetof`, interior and tail padding, array stride, overlap in unions and group-by-value function descriptors. Java 25 `MemoryLayout.structLayout` does not insert C padding automatically. A typed live union region may explicitly access a named interpretation through the already-approved `at/atPut` native region protocol. **There is no inferred active union member:** a semantic copy from/to an ordinary Protos object must receive an **explicit caller-selected active member for each union path**. Copies never fabricate a tag and do not materialize all conflicting overlapping fields as independently meaningful slots.

Group-by-value argument/return ABI must be verified through native C fixtures, with an independent owned post-call return region and Java FFM `SegmentAllocator` where required. Unproven by-value union combinations must be rejected before native entry rather than passed as pointers to pretend ABI support.

Packed structs, bitfields, flexible array members, varargs and nonstandard scalar/vector/long-double ABI are **deferred** pending independent platform-specific evidence/decision. An R2 descriptor must reject unsupported layout constructions, not silently approximate C memory.

### S2-H — Arbitrary native member names and ordinary Protos slots

Native field access stays distinct from ordinary Protos member access: `region["field"]` invokes `at/atPut`, while `point.field` follows ordinary slots/delegation. Ordinary snapshot slots named `call`, `init`, `createSlot`, `slotValue` or any other selector keep their full normal shadowing behavior. No native getter/setter magic, privileged Java reflection escape or new Core object family is introduced.

D262-A public `Object.createSlot` can support ordinary source-level snapshot construction when the canonical method remains accessible; where safe, the adapter may capture an ordinary receiver-bound `createSlot` **before** inserting fields which shadow its name. It cannot treat an arbitrary prototype that already overrides `createSlot` or a value overriding `slotValue` as if it exposes unforgeable reflection. The *exact* admissibility, preflight/failure order, field-association Map/Array alternative and public conversion signature are not selected by S2; present separately in the final API decision. Native region access remains usable independently of snapshot convenience.

### S2-C — Defer native C upcalls/callbacks from first R2 delivery

First bounded R2 does **not** export a C-native callback/upcall capability. This is a deliberate sequencing decision, **not** deletion of Core's existing synchronous foreign callback semantics. `VALUES_AND_COLLECTIONS.md`, `ACTORS.md` §24J and `CALLABLES.md` already require same Task/Actor/execution segment, actual Closure identity, correct `this/methodHome`, Error identity/nonlocal return behavior, dynamic extent, and exclusion of foreign-created-thread ingress. A future C upcall adapter must meet those rules, including how the native ABI receives a safe result without any Java exception escaping `Linker.upcallStub` (which may abruptly terminate the JVM), and must use actual JVM/Native Image tests. Retained callbacks, native-thread actor attachment, event-loop ingress and hidden scheduler/broker are not approved.

Callback-dependent C functions are therefore deliberately outside the initial R2 public binding surface, rather than partially supported through unsafe fake function-pointer Integers.

## 2. Explicit GITHUB021 selection/invariant map

| Previously selected owner/norm | S2 effect |
| --- | --- |
| G2 RootActor-only native FFM and host allowlist | PRESERVED |
| D259 `TRUSTED_IN_PROCESS` C process effects | PRESERVED, no false sandbox |
| P2 ordinary dot slots versus native bracket/indexed region | PRESERVED |
| P2 struct-by-value native region results, explicit conversion | PRESERVED |
| C1 native layout descriptor separate from prototype identity | PRESERVED |
| M3 managed/borrowed/pointer provenance, no Integer-to-address | PRESERVED |
| R1 exact native Integer admission and `f64` | PRESERVED |
| R1 blanket “no input rounding” | **EXPLICIT OWNER-APPROVED f32-ONLY EXCEPTION** |
| R1 no public typed native struct/union/f32 descriptors | **EXPLICIT OWNER-APPROVED R2 EXTENSION** |
| R1 no native FFM C callback | PRESERVED IN FIRST R2 / DEFERRED |
| Core synchronous foreign callback behavior | PRESERVED, remains authoritative |
| D262 ordinary reflective create/update slots | PRESERVED, no privileged bypass |
| Ordinary `FFI.bind` Closure, `out` Array sequence and existing error domains | PRESERVED |
| Native Image exact registered signatures | PRESERVED; new shapes must be registered/verified |

## 3. Accepted test obligations, not test results

- Exact `i8/u8/i16/u16/i32/u32/i64/u64` boundary and signed/unsigned carrier round trips, including `u64=2^64-1` into Protos BigInteger; out-of-range and wrong numeric-family errors before C touch counter changes.
- `f32` nearest/ties-to-even, subnormal, underflow, overflow/infinities, NaN semantic equivalence and signed zero; `f64` unchanged and no Integer coercion.
- `sizeof/_Alignof/offsetof` against a C fixture for mixed-field structs, nested structs, array stride, unions and exact overlap.
- Explicit union active-member path for native-to-object and object-to-native snapshots; absent/ambiguous selection errors before changes.
- Live fields with reserved/ordinary-method selector names; D262 canonical `createSlot` snapshot setup and deliberate failure/fallback with an existing overridden `slotValue` or `createSlot`.
- Struct group-by-value arguments/returns across exact x86-64 SysV ABI; result outlives transient call arena; no fake pointer signatures.
- Callback-dependent function declaration rejected in initial R2; no Java exception crossing native-upcall boundary; future native callback acceptance requires separate decision and fixtures.
- RootActor versus non-root native authority, owned/borrowed region lifetimes, Native Image exact descriptor registration, and R1 regression suites; no claimed tests are run by publishing this record.

## 4. Deliberately open exact API and production gates

S2 chooses **semantic scope**. It does **not** approve every illustrative `FFI.*` spelling, every C-native scalar name, exact `cType` key table, union selection-path representation, snapshot collision fallback, M3 `assumeExtent` lifetime-anchor types/read-write policy, ABI failure precedence, Native Image registration code, nor any Java per-struct helper/extra generic dispatcher.

Next: finish the **single exact API contract** for P3, including the M3 pointer trust/anchor interface and the S2 union/snapshot cases; reconcile the matching normative owners `spec/semantics/FFI.md`, `MODULES.md`, `ACTORS.md`, `SQLITE.md` and only necessary cross-references in numeric/Object/Callable rules. Then prepare bounded independent implementation work citing **actual** GraalPy or other Truffle production method/annotation/behavior precedents; run concrete JVM and Native Image ABI conformance tests. No special native C++ object ABI is authorized.

**State after this decision:** `D261=OPEN/status:ready`; `G2=P2=P3-C1=P3-M3=P3-S2=APPROVED`; `EXACT_PUBLIC_API=PENDING`; `NORMATIVE_SPEC=PENDING`; `PRODUCTION_CODE=UNCHANGED`; `COMPILATION/TESTS/NATIVE_IMAGE/PERFORMANCE=NOT_RUN`. Publication of this decision does not itself authorize final implementation or assert test success.
