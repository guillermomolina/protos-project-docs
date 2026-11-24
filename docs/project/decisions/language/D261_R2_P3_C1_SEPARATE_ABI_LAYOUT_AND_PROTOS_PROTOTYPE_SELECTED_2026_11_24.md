# D261-R2-P3-C1 — Separate native ABI layout and ordinary Protos prototype

**Status:** OWNER-SELECTED DESIGN SUBDECISION (2026-11-24). Remaining P3 memory, pointer, scalar, callback and exact-API decisions: PENDING.
**Authority:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158); normative text remains solely in `guillermomolina/protos/spec/`.
**Proposal:** [D261-R2-P3-C1 exact candidate](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7127496003).
**Approval provenance:** Project owner explicitly replied **“aprobada P3-C1”** to that proposal; [GitHub ratification comment](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7127596393).
**Previously selected:** [G2 native FFM / RootActor-only](D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md) and [P2 ordinary values + native regions](D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md).
**Product source inspected for the P3 review:** [`c98be4618fa5e8dc7a89f5092d3712616803e501`](https://github.com/guillermomolina/protos/commit/c98be4618fa5e8dc7a89f5092d3712616803e501). Publication of this record does not change product source.

## Decision — C1 selected, not all of P3

The native layout and the ordinary Protos prototype are **separate ordinary/descriptor entities**, not one object doing both jobs:

- The **native ABI layout descriptor** records the immutable C-compatible shape, field declarations, sizes and alignments. The illustrative `FFI.struct(fields)` produces a layout, not a callable constructor and not a Java type/class created for each native struct.
- The **ordinary Protos prototype** is a normal Protos object. The selected `FFI.prototype(layout, ordinaryPrototype)` association model privately binds the immutable layout identity to that supplied prototype **by exact object identity** and conceptually returns the **same prototype**, not a new wrapper. A conflicting subsequent association cannot silently redefine the native ABI; detailed conflict/error behavior remains to be specified.
- Ordinary invocation `Point(1.5, 2.5)` remains standard `Object.call -> construct -> init`. Ordinary `point.x` and `point.x =` retain standard lookup and local-slot assignment.
- The live native region is a different surface: `region["x"]`, `region["x"] = value` use normal `at/atPut` messages to access C memory with native bounds and lifetime checks. Explicit transfers create independent value snapshots; C by-value group return does not call `Point.init` implicitly.
- Native layout declarations, bounded regions and downcalls are **not dependent** on the new dynamic slot reflection. The convenience snapshot adapter can use independently selected [D262-A](https://github.com/guillermomolina/protos/issues/1163) / [I173](https://github.com/guillermomolina/protos/issues/1164) when available.
- Protos does not gain classes, a new Core value family, magic property getters/setters, a Java adapter per C struct, or another invocation model.

### Illustrative example, not yet runnable or a complete public API ratification

```protos
FFI: import("std:ffi")

PointLayout: FFI.struct(Array(
    FFI.field("x", FFI.f64),
    FFI.field("y", FFI.f64)
))

Point: FFI.prototype(PointLayout, {
    init: (x, y) => {
        this.x: x
        this.y: y
    }
})

point: Point(1.5, 2.5)
point.x = 3.0

region: FFI.allocate(PointLayout)
FFI.writeFrom(region, point)
region["x"] = 4.0
snapshot: FFI.readAs(Point, region)
```

`FFI.struct`, `FFI.prototype` here illustrate the **selected separation and association contract**. They do not imply that every name, descriptor signature, transfer operation or pointer rule from the full P3 candidate was approved. Final exact API spellings and public error/shape admission are subject to the later detailed normative gate.

## Falsification and invariant preservation (GITHUB021)

| Existing invariant | P3-C1 status |
| --- | --- |
| D261-R2-G2 RootActor-only direct FFM / inert `std:ffi` import | PRESERVED; C1 adds no independent native authority |
| D259 host-only alias allowlist, default deny, trusted process effects | PRESERVED |
| D238 native custody, Errors and ordinary Closure-based `FFI.bind` | PRESERVED except already explicitly approved R2 deltas |
| `OBJECT_MODEL.md` single parent and read-delegate/write-local-only | PRESERVED; ordinary prototype and members remain ordinary |
| `CALLABLES.md` call/construct/init ordinary activation | PRESERVED; no new native constructor semantics |
| `VALUES_AND_COLLECTIONS.md` indexing not equivalent to slots | PRESERVED; native regions deliberately implement their `at`/`atPut` |
| D262-A dynamic create/update of local slots | INDEPENDENT enabling convenience, not a G2/C1 prerequisite |

**Adversarial name collisions:** native fields named `call`, `init`, `slotValue`, `createSlot` or other inherited selector names may shadow methods in an ordinary snapshot's local slots. C1 does not change member lookup. Conversion policies for these field names require separate contract specification; native field-by-index remains available. No silent Java privileged reflection escape hatch may be introduced under this decision.

**Strongest objection to C1:** one extra public relationship/declaration between layout and prototype, with potential mismatches after prototype mutation. This is an intentional conceptual separation: immutable ABI metadata must not be redefined by mutating a Protos prototype, and an ordinary object must not silently become live C memory.

## Remaining P3 gates, explicitly not approved by this decision

1. **P3-M (memory/pointers):** custody of native allocations and C-owned results, scope/revocation, bounded borrow, explicit trusted extent assertion for pointer values with unknown provenance or size, cross-library passing and errors.
2. **P3-S (ABI/details):** complete scalar type set, `f32` conversion policy, unions/active members, field-name conflict handling for value snapshots, C canonical type names, synchronous callback/upcall scope and Native Image exact descriptor admission.
3. Exact layout padding/alignment, aggregate-by-value return allocation and `SegmentAllocator` exposure *through existing Java FFM mechanisms*, without inventing Java production infrastructure. Generic native C fixture and independent JVM/Native Image conformance required.
4. **Normative reconciliation:** `spec/semantics/FFI.md`, `MODULES.md`, `SQLITE.md`, `../concurrency/ACTORS.md`; preserve `OBJECT_MODEL.md`, `CALLABLES.md`, indexing and grammar unless an independently owner-approved delta is required.

**Decision approval scope:** P3-C1 only. No specification or production Java edit, compile, tests, Native Image build or release was authorized or performed by publishing this project record. **D261 remains OPEN**, with `status:ready` for the other exact P3 design gates.
