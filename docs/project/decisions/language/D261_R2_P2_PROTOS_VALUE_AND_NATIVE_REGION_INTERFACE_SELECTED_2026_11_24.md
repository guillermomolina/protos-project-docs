# D261-R2-P2 — Ordinary Protos values and live native regions

**Status:** OWNER-SELECTED PUBLIC INTERFACE MODEL, 2026-11-24. Detailed API spellings and additional lifetime/ABI rules remain PENDING.
**Owner:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
**Approval:** [proposed P2 contract](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7125603303) and [owner's explicit “sí lo apruebo” recorded in D261](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7125716070).
**Architecture retained:** [D261-R2-G2](D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md).
**Product source inspected:** [`c98be4618fa5e8dc7a89f5092d3712616803e501`](https://github.com/guillermomolina/protos/commit/c98be4618fa5e8dc7a89f5092d3712616803e501).
**Authority:** Non-normative project decision record; `guillermomolina/protos/spec/` remains the normative owner.

## Selected interface model

The project owner selected **A** (composable C ABI layouts, owned/borrowed native memory regions and ordinary Closure-based downcalls over Java FFM) with **B / Python-ctypes-like ergonomics** through ordinary Protos objects. The subsequent P2 approval settles this distinction:

1. **Ordinary value objects:** `point: Point(1.5, 2.5)` is ordinary prototype-based `call -> construct -> init`; `point.x` is an ordinary slot read and `point.x = 3.0` updates an existing local slot. No native property trap or hidden setter.
2. **Live C memory:** `region["x"]` and `region["x"] = 4.0` use existing `at` and `atPut` messages on a typed native-region object and read/write actual memory subject to lifetime/bounds constraints.
3. **Explicit copies:** transfers between ordinary local slots and C memory are explicit, not automatic aliasing. A C struct returned **by value** initially becomes an independently owned typed native region, not a Protos object constructed through `init`.
4. **No second language model:** do not add classes, native-field `.` dispatch, a new Core value family, Java classes per C struct, a new invocation mechanism, or a dedicated FFM scheduler.

### Illustrative syntax only — names NOT ratified

```protos
FFI: import("std:ffi")
Point: FFI.struct(
    Array(FFI.field("x", FFI.f64), FFI.field("y", FFI.f64)),
    { init: (x, y) => { this.x: x; this.y: y } }
)
point: Point(1.5, 2.5)
point.x = 3.0

region: FFI.allocate(Point)
FFI.writeFrom(region, point)
region["x"] = 4.0
copy: FFI.readAs(Point, region)
FFI.release(region)
```

This is **not** executable/validated source, and it does not approve these exact public method names, signatures, pointer assertions, destructor rules or layout representation.

## Invariant and downstream audit (GITHUB021)

| Existing authority | Consequence |
| --- | --- |
| `OBJECT_MODEL.md`: reads may delegate, writes never delegate; `:` versus `=` | PRESERVED; live native memory is not an ordinary `.` slot |
| `VALUES_AND_COLLECTIONS.md`: `[]` lowers to `at`/`atPut` | PRESERVED; only the native-region receiver deliberately implements field-by-name |
| `CALLABLES.md`: `call`, `construct`, `init` | PRESERVED; ordinary value prototypes only, no hidden native constructor |
| D238 FFI R1 descriptors, Closures, custody, Errors | PRESERVED except explicitly approved R2 amendments |
| D259 host default-deny, trusted-in-process native effects | PRESERVED |
| D261-R2-G2 RootActor-only native FFM entry | PRESERVED; import of `std:ffi` stays inert from all Actors |
| D262-A `Object.createSlot` / `Object.updateSlot` | INDEPENDENT; can simplify snapshot object construction, not a requirement for native memory and downcalls |

After independent [D262-A selection](https://github.com/guillermomolina/protos/issues/1163), the source-level `FFI.readAs(prototype, region)` adapter can use existing `(prototype) {}` object construction followed by D262 `createSlot(name,value)` to fill local snapshot fields **without invoking `init`**. This ergonomic adapter therefore may depend on [I173 #1164](https://github.com/guillermomolina/protos/issues/1164), but generic native layouts, memory regions and downcalls **must not be blocked by I173**. A native field named `call` may naturally shadow an ordinary object's inherited invocation behavior if copied into its local slots; P2 does not redefine ordinary method semantics.

## Pending exact contract

Not selected by P2: `FFI.struct` prototype-versus-descriptor representation, precise public signatures and ABI scalar set; padding and union handling; C struct-by-value linker allocation; region ownership, slices and release; pointer bounds/lifetime assertions for C-returned unknown-size addresses; cross-library pointer custody; scalar narrowing and f32 rounding; partial copy failure rules; synchronous upcall scope; Native Image descriptor registration. No numeric address forgery or foreign-thread-to-Protos callback ingress is implied.

See the [P3 proposed public contract, ABI risks and test matrix](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7126509008) for **unapproved** detail candidates.

## Next gates

Reconcile the approved G2/P2 choices in `spec/semantics/FFI.md`, `MODULES.md`, `ACTORS.md`, `SQLITE.md` after the exact pending contract receives owner approval. Verify independently with small C ABI fixtures, RootActor and second-Actor cases, JVM and Native Image signatures, and R1 regressions. No product source/spec edits or compilation/test success are claimed by publishing this record. **D261 remains OPEN.**
