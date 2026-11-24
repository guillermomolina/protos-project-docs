# D261-R2-N1 — Demand-driven generic Native Image FFM ABI envelope

**Status:** OWNER-SELECTED DESIGN REFINEMENT, 2026-11-24. Exact R2 public API, detailed registration implementation, normative specification and runtime acceptance remain pending.
**Live owner issue:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158).
**Approval provenance:** [N0–N3 proposal and recommendation of N1](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7128982225); the owner replied **“ok aprobado”** to the recommendation. [Approval recorded in the coordinating issue](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7129018375).
**Prior platform authority:** [PLAT077 Candidate A, selected](PLAT077_GENERIC_NATIVE_IMAGE_FFM_PROVIDER_A_RATIFIED_2026_11_22.md); [I170 implemented generic Native Image provider](https://github.com/guillermomolina/protos/issues/1148).
**Earlier D261 decisions:** [G2 architecture](../language/D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md), [P2 two-surface model](../language/D261_R2_P2_PROTOS_VALUE_AND_NATIVE_REGION_INTERFACE_SELECTED_2026_11_24.md), [P3-C1 layout/prototype](../language/D261_R2_P3_C1_SEPARATE_ABI_LAYOUT_AND_PROTOS_PROTOTYPE_SELECTED_2026_11_24.md), [P3-M3 native memory](../language/D261_R2_P3_M3_BOUNDED_NATIVE_MEMORY_AND_TRUSTED_POINTER_ASSERTION_SELECTED_2026_11_24.md), [P3-S2 scalars/unions/callback scope](../language/D261_R2_P3_S2_SCALARS_UNIONS_SNAPSHOTS_AND_NATIVE_CALLBACK_SCOPE_SELECTED_2026_11_24.md).
**Product baseline inspected:** [`guillermomolina/protos@8c3be22391db29bd4ff1abd102884d07d3b2a50e`](https://github.com/guillermomolina/protos/commit/8c3be22391db29bd4ff1abd102884d07d3b2a50e).
**Authority boundary:** This record is non-normative. `guillermomolina/protos/spec/` is the normative owner of public language semantics, and PLAT077 remains the governing implemented generic platform architecture.

## 1. The exact selected distinction

The owner originally proposed a list of authorized libraries already integrated into Protos, such as SQLite, so that unsupported libraries would not impose Native Image implementation costs. After distinguishing authorization from compiled ABI capability, the owner **approved N1**, the evidence-driven **physical ABI registration** approach, rather than the stronger **N2** per-library runtime restriction.

**N1:** Extend the set of *compiled* Native Image FFM downcall shapes only when an actual, validated Protos C ABI integration requires them. SQLite is an existing concrete source of approved signatures; later Protos Standard Library implementations and independent fixtures can justify further, exact native signatures. Existing finite generic signature coverage remains available; N1 governs how to grow it, not a rollback of existing registrations.

This is a **physical signature catalogue**, not a `sqlite3`-only executable, a symbol-based allowlist or a new Java backend per library. An ABI signature includes its true return/parameter layouts and linker options; by-value aggregates use their exact group layout and are never disguised as pointer-shaped functions. A catalog entry expresses **ability to link a signature**, not host permission to load a library.

## 2. Public behavior and intentional JVM/Native divergence

| Concern | Portable JVM | Installed Native Image |
| --- | --- | --- |
| Guest `std:ffi` surface | One generic Protos-first source API | The same source/API and generic provider |
| ABI registration | Valid backend-supported signatures can be linked dynamically | Only exact shape/options pre-registered at Native build time |
| Growth policy | Existing JDK 25 FFM profile; no per-library source restriction | Add signatures when justified by independently verified Protos-integrated use cases |
| Library authorization | Explicit host `--allow-library alias=target`, default deny | Same host-only default-deny policy |
| Unlisted yet host-authorized C library | May link a supported physical signature | May link **if its signature is already in the compiled envelope** |
| Missing physical signature | Backend may link according to JVM capability | Reject **before entering C**; requires catalog extension and rebuilt/released image |
| Third-party `.so` embedded | None implied | **None embedded**; host OS supplies explicitly authorized shared object |
| Absent FFM use | No native acquisition | Provider inclusion still inert: no native library loaded or FFM session merely on ordinary launch |

**N2 NOT SELECTED:** Native must **not** reject a library solely because it is absent from an official Protos integration list if the host explicitly authorized it and its physical ABI is already compiled. N1 therefore **preserves** PLAT077's generic runtime interoperability rather than introducing a new per-library policy.

**N3 NOT SELECTED:** No SQLite- or terminal-specific provider, Java adapter, native-link stub, JNI/NFI alternate backend, second executable, bundled C library, or new Protos value family.

## 3. GITHUB021 invariant reconciliation

| Earlier contract | N1 effect |
| --- | --- |
| PLAT077 Candidate A generic Native-integrated FFM, finite exact downcall envelope | PRESERVED: additive evidence-driven growth |
| I170 existing native provider and 27-shape metadata baseline | PRESERVED: no rollback or claims of extended conformance |
| D259 host-only exact library alias/target allowlist and lazy loading | PRESERVED; build signatures never grant OS authority |
| G2 RootActor-only native entry and trusted C OS/process effects | PRESERVED; no non-RootActor native admission |
| P2 ordinary dot access vs typed native `at/atPut` | PRESERVED |
| C1 separately immutable native layout vs ordinary prototype | PRESERVED |
| M3 bounded provenance, logical owners, trusted extent assertion, no Integer-to-address | PRESERVED |
| S2 exact unsigned/signed scalars, opt-in `f32` rounding, explicit union member selection, upcalls deferred | PRESERVED: Native ABI entries must match these actual layouts when supported |
| R1 Closure `FFI.bind` / owned custody, `out` result conventions and errors | PRESERVED |
| Java FFM Native Image exact build-time registration and unsupported-shape rejection | PRESERVED; unsupported R2 shapes do not silently become pointer downcalls |
| JVM generic external provider JAR and installed Native consumer without JDK/GraalVM | PRESERVED |
| No eager library load for `std:ffi` import or ordinary Protos execution | PRESERVED |

**GITHUB021 outcome:** N1 is consistent with PLAT077 and all separately approved D261 semantic invariants; it refines the **selection criterion for new build-time FFM registrations**, without introducing a newly approved per-library runtime admission restriction. If an actual future implementation must narrow an already supported Native ABI shape or alter host authorization, it is **outside this approval** and requires its own explicit decision.

## 4. Exact implementation boundaries and test gates

1. Source of ABI entries: a **reviewable, deterministic, reproducible** catalog of actually needed and verified native layouts/options. The specific source-of-truth (checked manifest, build-generated proof based on native fixtures, or verified extraction of Protos-source declarations) remains a bounded *future* choice; N1 does not approve speculative Java build infrastructure or a source parser.
2. Native Image registration must agree with `FfmBinding.Signature.descriptor/physicalShape`, `NATIVE_SHAPES`, and `src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json`; preserve unrelated FFM registrations such as confined Filesystem. R2 group layouts require exact ABI registration; a letter `P` replacement is not equivalent.
3. Audit `protos/lib/sqlite/abi.protos` (including private R1 provider usages), `protos/tests/foreign-provider/ffm-abi.protos`, the separate C fixture, and any new verified Protos C integration. Physical ABI equivalence may permit other host-authorized C libraries to reuse the same compiled registration.
4. Required independent tests: unchanged installed Native SQLite, a different host-authorized C library reusing a registered signature, an authorized but unregistered signature rejected before C entry without raw `MissingForeignRegistrationError`, default-deny import and inert `std:ffi`, JVM dynamic supported signature, genuine R2 f32 and struct-by-value cases when their catalog entries are selected, and Native Image exact descriptor/options parity.
5. Account for Native build duration, executable size, idle startup/memory and no-FFM execution cost. Do not pre-register speculative combinations of all scalar widths/arities/group arrangements.
6. Any future **production Java** edit must cite a specific method/annotation/behavior precedent from GraalPy or other Truffle source, including GraalPy `GenerateNativeDowncallsProcessor.collectDowncalls/generateInvoker` and TruffleRuby `FFMSupport.functionDescriptor/createDowncallHandle` as candidates for exact audit; do not copy an unverified framework or invent an alternate FFM dispatcher.
7. Normative `spec/semantics/FFI.md`, `MODULES.md`, `ACTORS.md`, `SQLITE.md` and any Native distribution documentation can be reconciled after completing the remaining **exact R2 public contract** and source owner approval. This record does not grant an implementation or normative publication shortcut.

## 5. What remains to decide in D261

N1 does **not** select exact public `FFI.*` names/signatures, permitted M3 lifetime anchors and mutation rights, union active-member path representation, snapshot field collisions/fallback, pre-entry and entered-error precedence, or a specific generator/manifest implementation. D261 is still OPEN with `status:ready` for these gates, then bounded implementation and real JVM/Native Image validation.

**Publication/validation status:** Owner approval recorded and this non-normative decision may be published. Product code and spec are unchanged; compilation, tests, Native Image, benchmarking and local `git diff --check` are NOT RUN by this recording. A future implementation must report actual results, not infer them from prior I170 evidence.
