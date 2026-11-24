# D261-R2-P3-M3 — Bounded native memory with explicit trusted pointer assertions

**Decision status:** OWNER-SELECTED on 2026-11-24; the exact API and some error/lifetime policy refinements remain pending.
**Project owner issue:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
**Owner selection provenance:** [D261 P3-M3 explicit approval record](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7128530842). After presentation of the exhaustive M0–M4 research and recommendation, the owner replied **“ok apruebo M3”**.
**Research:** [P3-M exhaustive GITHUB010/GITHUB021 evidence](../../evidence/D261/D261_R2_P3_M_GITHUB010_GITHUB021_NATIVE_MEMORY_POINTER_RESEARCH_2026_11_24.md); revision inspected [`8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Prior architecture and public model:** [G2 RootActor-native entry](D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2 native regions versus Protos values](D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1 layouts separate from prototypes](D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md).
**Authority:** Non-normative selected design record; canonical normative publication remains in `guillermomolina/protos/spec/`. This record neither changes `spec/` nor authorizes unreviewed Java infrastructure.

## 1. Exact choice: M3, not M0/M1/M2/M4

The owner selects **M3 — bounded native-region capabilities with explicit provenance, custody and a deliberate, potentially unsafe extension for externally managed C memory**:

- **Pass-only native pointer:** an admitted compatible C pointer without accredited extent may be passed to a separately authorized compatible C ABI signature, but does **not** permit Protos to dereference it. An address is not a region. The pointer is an opaque native capability, never an Integer or a destructor right.
- **Owned native root region:** an explicit allocation has known layout/extent, one custodian, and a revocable lifetime that can end before the library session closes. Typed `at/atPut` and derived native views/slices operate only within known bounds, alignment and layout constraints.
- **Borrowed native view:** a non-owning, bounded region that carries the provenance of an actual logical owner/resource, source session or agreed borrow epoch. The view cannot free memory and becomes unusable when its admitted lifetime ends. Child views/pointers cannot outlive the parent authority.
- **Explicit trusted extent/lifetime assertion:** the Process RootActor can intentionally accredit an already admitted *C-origin unknown-size* pointer by declaring bounds and an admissible native lifetime anchor. This is a **trusted ABI claim, not a proof** of physical liveness. The API must make the unsafe assumption conspicuous; it cannot be used to forge native addresses from arbitrary Protos Integers.

A compatible physical pointer can travel between **independently host-authorized C libraries**, preserving its original resource ownership and destructor. Nominal R1 opaque descriptors retain their preexisting library/session/type checks: M3 does not silently reinterpret them as generic physical pointers.

The native RootActor is still **not** a sandbox for authorized C. Under G2/D259 `TRUSTED_IN_PROCESS`, native code may create threads, access process memory, free earlier than its declaration or corrupt the process. No guest-side temporal check can guarantee safety against a false C lifetime assertion or wild native write.

## 2. Temporal, identity and failure invariants

1. **Root identity:** logical native acquisition identity is not numeric-address identity. Address reuse never resurrects a retired resource. A region/view records private owner, session and relevant generation/borrow epoch and a checked extent.
2. **Release:** a single canonical owner may release; dependent views and pass-derived capabilities are revoked by owner retirement/session close without a mandatory O(N) walk of all descendants. Released or expired memory is rejected **before** dereference. Destructor is attempted at most once after irrevocable logical retirement, even if the native return/status is adverse; uncertain physical outcome is recorded, not automatically retried.
3. **Call admission:** retain R1's pre-entry validation, owned-output adoption before fallible conversion, and active-call shutdown guarantee. Managed memory still being used by an admitted synchronous native call must not be physically reclaimed prematurely. No extra global FFM lock, broker, thread pinning or scheduler change is selected.
4. **Borrow invalidation:** R1 private `borrowed`/`preserving`/epoch/copy laws remain. A non-preserving native call invalidates the *R1 epoch-governed* borrowed view as already specified; **distinct independently owned R2 memory does not expire merely because that call happened**. No rejected pre-entry call advances a borrow epoch. Future public borrowed views must explicitly bind to the appropriate lifetime policy.
5. **Bounded access:** layout-aligned, nonnegative offsets, lengths, arithmetic overflow, liveness and adequate extent are checked before access. Null pointers cannot become readable; a non-null zero-extent pointer may still be passed opaquely. The old **16 MiB** bound applies to R1 private borrowed copies and does **not** automatically become a global native allocation limit.
6. **Isolation:** native capabilities, owners, borrowed regions and views cannot automatically cross Actor or P boundaries. Ordinary detached copies follow the existing complete-value-graph transfer rule. RootActor-only native FFM acquisition/access and host allowlist remain.
7. **Lifetime substrate:** persistent managed native regions must not rely on a call-confined `Arena.ofConfined()`, because RootActor's carrier may change; Java 25 `Arena.ofShared()` or another **existing, independently justified native FFM mechanism** may express their physical lifetime. The existing FfmSession shared Arena controls library lookup and should not become an indivisible lifetime for all independently releasable allocations. No class-per-struct or globally shared pointer registry is authorized.
8. **AOT:** only registered exact downcall signatures/layouts are admissible in Native Image; a pointer-shaped substitute cannot impersonate a group-by-value ABI. ABI/host correctness remains an explicit conformance task.

## 3. Public API illustrations — names NOT selected by M3

```protos
region: FFI.allocate(PointLayout)
region["x"] = 3.0
ptr: FFI.addressOf(region)
view: FFI.view(ptr, PointLayout)                 // checks known extent and lifetime
FFI.release(region)                             // invalidates view
```

For an externally managed C pointer with unknown bounds:

```protos
ptr: cFunctionReturningOpaquePointer()
bounded: FFI.assumeExtent(ptr, declaredLength, lifetimeAnchor)
```

`FFI.assumeExtent` is **illustrative** and not a blanket implementation authorization. It cannot turn expired memory into physically live storage. The owner has approved the **availability of a deliberate trusted assertion**, not its exact name, signature, permitted lifetime anchors or whether the resulting view is read-only/writable.

## 4. Explicitly unresolved P3-M follow-up choices

- Exact public operation names, parameter and return domains and ordinary Errors for an invalid trusted assertion.
- Which owners/anchors are admissible: acquired C-owned opaque resources, managed root regions, session library-static memory, specific borrow epochs and potentially other explicit lifetime policies. A plain String or a live session alone does not prove an arbitrary C heap allocation remains live.
- Read-only versus writable borrowed regions, how a borrowed view is explicitly retired, and which C operations invalidate its borrow epoch or producer lifetime.
- Provenance and bounds when pointer fields in native structs point outside the containing struct; they must **not** inherit the entire containing region's extent by default.
- Release and in-flight call interlock details, physical allocation batching and actual per-root Arena performance, C callback deferral and exact Native Image metadata coverage; each is to be specified and/or validated rather than invented in Java.
- P3-S, covering scalar widths, `f32`, unions, field-name conflicts and native callback policy, remains **unselected**.

## 5. GITHUB021 selection audit and evidence gates

| Applicable approved invariant | M3 result |
| --- | --- |
| G2 RootActor-only general native FFM | PRESERVED |
| D259 host default-deny and `TRUSTED_IN_PROCESS` native effects | PRESERVED; trust not overstated |
| G2 no global lock/broker or invented Java infra | PRESERVED |
| P2 ordinary `.field` vs native-region `["field"]` | PRESERVED |
| P2 independently owned struct-by-value native return | PRESERVED |
| P3-C1 native layout independent of Protos prototype | PRESERVED |
| D238/R1 native owned custody and borrow copy/epoch | PRESERVED, with separately selected new R2 *public* bounded view capability |
| Existing R1 opaque nominal library/session types | PRESERVED |
| R1 public native pointer/view exclusion | **EXPLICIT NEW R2 DELTA approved as M3** |
| R1 Actor-local native FFM capability | **ALREADY deliberately superseded by previously approved G2** |
| Object, Closure, indexed access and Actor/P transfer rules | PRESERVED |
| Explicit trust assertion of unknown-size C pointer | **EXPLICIT NEW R2 AUTHORITY approved as M3**, without Integer forgery |

The exhaustive comparison investigated **ten systems across three design routes**, five meaningful candidates, twelve criteria scored 1–5 with rationales and confidence, non-compensating flags, future scenarios, reversibility, and independent tests; the evidence document remains the source for these details.

**Next owner gates:** resolve specific public semantics and accepted lifetime anchors in a bounded follow-up; close P3-S separately; publish normative `spec/semantics/FFI.md` and owner cross-references when the exact candidate is approved; implement as small independent testable steps with method/annotation/behavior precedents in GraalPy/TruffleRuby and Java FFM. Do not repeat GITHUB010 absent a genuinely new design conflict.

**Publication status:** decision recorded, but D261 stays OPEN. Product/spec edits, compilation, tests, Native Image and performance for D261 are **NOT RUN / NOT PERFORMED** by this record.
