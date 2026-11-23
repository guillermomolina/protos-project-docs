# D261-R2-G2 — GITHUB010 comparative research, owner approval, GITHUB021 invariants

**Date:** 2026-11-23  
**Issue / owner:** [D261 #1158](https://github.com/guillermomolina/protos/issues/1158)  
**Recorded approval:** [D261 issue comment #7105424394](https://github.com/guillermomolina/protos/issues/1158#issuecomment-7105424394)  
**Selected, non-normative decision:** [D261-R2-G2](../../decisions/language/D261_R2_G2_GENERAL_JAVA_FFM_ROOT_ACTOR_ARCHITECTURE_SELECTED_2026_11_23.md)  
**PROTOS_REVISION:** [`89028494587d4d22309a5c048e37da3518170b18`](https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18)  
**PROJECT_RECORD_REVISION:** use the exact publication commit linked in the D261 GitHub issue.  
**Source of owner authorization:** current conversation, owner explicitly approved the G2 recommendation and authorized direct commit/push in `guillermomolina/protos-project-docs` for D261 documentation.

## 1. What was actually approved — chronology and provenance

D261 originally focused on a small incremental R2 driven by `ncursesw`/`termbox2`. An initial `FFI.inBytes`/`FFI.outBytes` recommendation was **not approved**. The owner objected that it confused a particular library's needs with Protos's general ability to import native libraries; that Java FFM already knows structures; and that the RootActor should suffice for native acquisition without prebuilding multi-Actor access.

Material corrections explicitly made by the owner before the final recommendation:
1. FFM must be a **general Java FFM-based mechanism** and not a terminal-specific escape hatch.
2. **Only RootActor opens/uses native system libraries from Protos** for this release; native access from other Actors and multithreaded Protos ingress can grow later, under pay-as-you-grow.
3. **Do not restrict a trusted native library's own OS threads/effects.** Just like guest `--allow-filesystem` cannot confine an allowed native library, native `pthread`/thread creation, C globals, native I/O and code are the library's responsibility under D259.
4. RootActor direct execution normally runs on the host calling thread; other RootActor bootstrap/scheduling modes already exist and do **not** warrant an FFM thread manager or permanent carrier pinning.
5. **No invented Java infrastructure.** Copy specific verified implementation patterns from GraalPy/other Truffle where production Java changes are necessary; keep public ABI description and binding policy in ordinary Protos source; minimize Java classes.
6. Preserve the host's explicit default-deny native authorization, existing foreign-value semantics, Native Image provider, and separated terminal feature responsibilities.

The agent's **revised recommendation** was `D261-R2-G2`: general FFM descriptors/layouts, memory/pointers and downcalls; RootActor-only entry; authorized libraries otherwise free to use host system/thread resources; no guest scheduler/locks/broker/foreign-thread callbacks; support C-compatible ABI exports regardless of native library implementation language; incremental delivery and separate exact public-contract gate.

The owner answered verbatim **“ok apruebo recomendación.”** This is an **explicit architectural approval** of G2 after the previous RootActor-only R1 incompatibility was surfaced, **not** the approval of any previously illustrative code spelling, exact `FFI.*` operations, arbitrary numeric-address policy, complete callbacks, Native Image universal ABI or a change to D260's Screen API.

**Publication permission:** the owner additionally asked to update issues in `guillermomolina/protos`, add evidence in `guillermomolina/protos-project-docs` and explicitly authorized direct commit/push to the latter even if AGENTS would otherwise restrict it. No permission to commit product Java or normative `spec/` was inferred.

## 2. Normative sources and implementation snapshot

| Inspected exact path / work item | Grounded result, not a new selection |
| --- | --- |
| [`AGENTS.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/AGENTS.md), [`AGENTS.work/DESIGN.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/AGENTS.work/DESIGN.md), `REFERENCE.md`, `COORDINATION.md` | GITHUB010 research + 12 criteria, GITHUB021 invariant check, owner selection, no normative/product change by implication, role-first durable records |
| [`spec/semantics/FFI.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/semantics/FFI.md) | R1 `void/i32/i64/f64/sizeT/utf8z/opaque/nullable/owned/out/bind/release`; no public C aggregate or native memory view; foreign resources Actor-owned, currently not root-exclusive |
| [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/semantics/MODULES.md) | Ordinary Actor-local foreign facades; import cache identity; `std:ffi` import inert; acquiring `ffi:<alias>` foreign facade separately enters provider |
| [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/concurrency/ACTORS.md) §§24J, 24K.1, 24L, 32 | Synchronous foreign segment, no automatic suspension; authorized native library globals intentionally not isolated; handles not auto-proxied; RootActor ordinary logical Process Actor |
| [`spec/semantics/VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/semantics/VALUES_AND_COLLECTIONS.md), `FUTURES_AND_TASKS.md` | D188 foreign primitive admission, errors; D189 sync-only callback dynamic extent, no retained/foreign-thread callback ingress |
| `spec/semantics/ERRORS.md`, `CALLABLES.md`, `IO_CORE.md`, `PARALLEL_EXECUTION.md` | `ForeignError` vs preentry Error, ordinary callable, resource lifecycle, P/nontransferability boundaries |
| [`spec/semantics/SQLITE.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/semantics/SQLITE.md), `protos/lib/sqlite/module.protos` | Public SQLite opens `ffi:sqlite3` lazily; RootActor-only directly changes which Actor may open SQLite |
| [`protos/lib/ffi/module.protos`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/protos/lib/ffi/module.protos) | Public frozen ordinary descriptors, Closure `bind`, provider operations hidden by Protos module |
| [`FfmTypes.java`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmTypes.java) | Scalar layouts and `ADDRESS`, exact Linux x86-64 LP64 requirement; no compound layout public representation; same-library opaque types |
| [`FfmBinding.java`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java) | `NATIVE_SHAPES` 27 scalar/carrier strings; `Signature.descriptor()`, `isLinkable()`, `belongsTo()`, `admitsArgument()`, per-call `Arena.ofConfined()`, staged owned-result custody |
| [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java) | Host alias authorization precedes `SymbolLookup.libraryLookup`; `Arena.ofShared()` session, active-call tracking, binding reuse and destructor custody |
| `FfmHandles.java`, `FfmValueOperations.java` | Private borrowed copy bounded to 16MiB, epochs, resource generation; private OCTETS input currently encodes via String, not public Bytes ABI; no public memory region equivalence |
| `ProtosActor.java`, `ProtosActorBootstrap.java`, `ProtosActorScheduler.java`, `ProtosPolyglotProcessContext.java` | `isRootActorForRuntime()` already exists. Direct RootActor entry is caller-hosted; message-service bootstrap uses scheduling carriers. No new Actor class needed |
| `ProtosForeignModuleImports.java` | Generic foreign import route currently acquires a provider target for any Actor; RootActor-only must target native FFM specifically, **not** the generic foreign provider SPI |
| `src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java`, `protos/tests/foreign-provider/ffm-public-api.protos`, `ffm-abi.protos` | Existing SQLite-independent ABI fixtures and Native matching checks; these can be extended without building an entire downstream library |
| D238 #1047; D259 #1139; PLAT077 #1146; I170 #1148 | Existing owner-approved R1, trusted native authorization, built-in Native provider and finite signature envelope; no reopening except explicit RootActor-only delta |
| D260 #1154; PLAT078 #1155; LIB032 #1152 | Screen public semantics, terminal backend architecture and product implementation have **separate** owners and approval gates |

Snapshot is repository-read evidence, not current local running tests. Source revision is pinned to avoid attributing subsequent unseen changes.

## 3. Cross-system GITHUB010 comparative evidence

At least five substantial systems across three different approaches were examined, with real code-level correspondences where production Java would be necessary:

| System / family | Verified source or official documentation | Specific consequence |
| --- | --- | --- |
| **Java 25 FFM** — general layout/function/memory interface | [`Linker`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Linker.html), [`MemoryLayout`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/MemoryLayout.html), [`MemorySegment`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/MemorySegment.html), [`Arena`](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Arena.html) | `structLayout`, `unionLayout`, `sequenceLayout`, `paddingLayout`, allocation, region access and downcalls. Group-layout result may require leading `SegmentAllocator`; address validity cannot be inferred from C |
| **GraalVM 25 Native Image** — AOT | [official FFM documentation](https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/) | Native supports C structs by value and native shared library loading; FFM downcall/upcall descriptors must be available to the image builder. Metadata, tracing agent and hosted registration are options. Cannot claim JVM-like arbitrary unregistered dynamic shapes |
| **GraalPy** — Truffle/native | [`CStructAccess.readIntField/writeIntField/allocate`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/cext/structs/CStructAccess.java), [`NativeMemory.calloc/malloc/memcpy/readByteArrayElements/writeByteArrayElements`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeMemory.java), [`GenerateNativeDowncallsProcessor.collectDowncalls/generateInvoker`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python.processor/src/com/oracle/graal/python/processor/GenerateNativeDowncallsProcessor.java) | Concrete methods for C layout field access, memory and finite generated downcalls; processor uses `@DowncallSignature`, generated `@TruffleBoundary`; do not invent special Java nodes |
| **TruffleRuby** — Truffle/FFM | [`FFMSupport.carrierLayout/functionDescriptor/createDowncallHandle/createUpcallStub`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/platform/FFMSupport.java), [`PointerNodes`](https://github.com/truffleruby/truffleruby/blob/master/src/main/java/org/truffleruby/extra/ffi/PointerNodes.java), [`struct_layout.rb`](https://github.com/truffleruby/truffleruby/blob/master/lib/truffle/truffle/ffi_backend/struct_layout.rb) | FFM calls/upcalls and native memory; `@Primitive`, `@Specialization` on pointer methods; StructLayout field offset/get/put is Ruby-level; no need for per-struct Java class |
| **Espresso** — Truffle/NFI | [`NFINativeAccess.bindSymbol/getCallableSignature/createNativeClosure`](https://github.com/oracle/graal/blob/master/espresso/src/com.oracle.truffle.espresso/src/com/oracle/truffle/espresso/ffi/nfi/NFINativeAccess.java) | Alternative dynamic call/closure pattern; demonstrates decoupling ABI signature and call site, but does not justify replacing existing Java FFM |
| **Sulong/LLVM** — alternative guest-native VM | [Oracle GraalVM LLVM documentation](https://www.graalvm.org/latest/reference-manual/llvm/) | Alternative execution substrate; not a simpler substitute for the existing FFM provider |
| **Python ctypes** — dynamic FFI | [official ctypes](https://docs.python.org/3/library/ctypes.html) | User-level C struct/union/pointer/function/callback shapes; illustrates that native ABI generality is not tied to a specific C library |
| **Python CFFI** — ABI vs compiled API | [official CFFI](https://cffi.readthedocs.io/en/latest/overview.html) | Direct ABI and generated API are distinct options; optional generator does not belong in runtime core |
| **LuaJIT** — declaration driven | [LuaJIT FFI](https://luajit.org/ext_ffi.html) | General C declarations in the guest language, with explicit unsafe native memory responsibilities |
| **Rust / Zig** — compiled `extern` models | [Rust type layout](https://doc.rust-lang.org/reference/type-layout.html), [Zig C interoperability](https://ziglang.org/documentation/master/#C) | Precise ABI type layout and external binding; portability depends on platform/toolchain layout |
| **.NET Native AOT** — compiled interop | [Microsoft LibraryImport](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation) | Compile-time marshalling strategy; optional generation is a separate implementation choice |
| **OpenJDK jextract** — header generation | [OpenJDK jextract](https://github.com/openjdk/jextract) | Useful future Protos declaration generator and layout verification; not mandatory runtime functionality |

**Other surveyed Truffle languages:** GraalJS separates JS interop and native Node/native-addon ecosystems; it does not provide a smaller reusable Java 25 FFM provider for Protos. SimpleLanguage is a Truffle instructional language, not a production generic C ABI. Apple Pkl was considered as required by DESIGN.md; its configuration-language execution model has no materially comparable general C ABI surface. These are explicit relevance exclusions, not missing precedents.

**Portability:** Java ABI layouts differ among Linux/ELF, macOS/Darwin, Windows LLP64, BSD and processor architectures (e.g. size of C `long`, `wchar_t`, alignment, calling conventions). The first tested Protos provider remains **Linux x86-64 LP64**. A general descriptor API does not imply tested Windows/macOS binaries. C++-implemented libraries with C-compatible native symbols can be used; C++ class/exception/virtual ABI mapping is outside the selected architecture.

## 4. Credible alternatives and twelve-dimensional comparison

**A** Keep R1 unchanged; **B** per-library C normalization shim; **C** two-operation binary buffer-only increment; **D** generic Java FFM nucleus, RootActor-only and incremental (selected); **E** implement everything immediately (foreign-thread callbacks, all variants/platforms, complete Java FFM mirror). Optional header code generation as mandatory path was examined but rejected: it would impose a tool/build dependency instead of allowing direct Protos declarations.

Scoring: 1 weak .. 5 strong. `H` = high evidence confidence, `M` = medium, `L` = low. Each score includes its short reason. **Qualitative red flags override arithmetic totals**; these are not measured performance.

| Mandatory DESIGN.md criterion | A current R1 | B C shims | C Bytes-only | **D general G2** | E full now |
| --- | --- | --- | --- | --- | --- |
| 1 Correctness / invariants | 5H unchanged | 3M divergent ABIs | 4M safe copies only | **4M explicit custody** | 2L wide edge cases |
| 2 Protos alignment | 2H blocks types | 2M more C layers | 3M narrow fix | **5M general objects** | 3L parallel universe |
| 3 Present-need proportionality | 5H no cost | 3M per-library cost | 4H two methods | **4M baseline nucleus** | 1H speculative systems |
| 4 Incremental growth | 2H can't express types | 2M shim accumulation | 2M layout rewrite | **5M additive ABI** | 4M all prebuilt |
| 5 Future-option resilience | 4H no commitment | 2M C shim lock-in | 3M partial escape | **5M ABI decoupling** | 4L broad but fixed |
| 6 Scalability | 3M existing limit | 3L varied wrappers | 3M repeated copies | **4M on-demand regions** | 3L unknown global cost |
| 7 Conceptual simplicity | 5H already familiar | 2M diffuse per lib | 4H tiny but incomplete | **4M familiar FFM primitives** | 1H excessive semantics |
| 8 Portability | 2H LP64 only | 2M OS shims | 3M bytes portable | **4M canonical ABI layouts** | 3L big matrix |
| 9 Runtime/resource cost | 5H zero new | 3M distribution costs | 3M copying | **4M pay on use** | 1L eager overhead risk |
| 10 Failures/operability | 4H known R1 | 2M multiplied boundaries | 3M partial guards | **4M explicit lifetime** | 2L complex ingress |
| 11 Deferral/reversibility | 1M compatibility blockage | 2M migration from shims | 2H future overhaul | **5M reusable base** | 3M hard to simplify |
| 12 Evidence/implementation risk | 5H existing | 4H C bridges standard | 4H bounded-copy precedent | **4M Java+Truffle sources** | 3L end-to-end absent |

### Non-compensating flags and recommendation

- **A — UNDERENGINEERING:** cannot satisfy the explicit general-FFM objective; another special case will recur.
- **B — UNDERENGINEERING/MIGRATION:** per-library native shims proliferate even where the ABI has first-class Java FFM layouts, produce version/packaging burden, and need non-portable C code.
- **C — UNDERENGINEERING:** `inBytes/outBytes` omits actual C aggregate and pointer/memory models; replacing it later changes the basis of public FFM usage. Rejected former recommendation.
- **D — SELECTED (bounded implementation):** layout, memory and calls are an irreducible coherent foundation; reuse Java's underlying concepts, but defer optional callback/concurrency/OS features. RootActor-only removes speculative guest concurrency infrastructure.
- **E — OVERENGINEERING:** user pays for retained callbacks/foreign-thread ingress/extra ABI targets and complex lifetime semantics before use cases justify them. No complete Java FFM API mirror is selected.

Strongest objection to **D**: a general region/layout model costs more initially than a byte copy and permits unsafe C mistakes. Counterweight: the ABI is fundamentally structured, and the next struct pointer/by-value result would force a second model if memory were represented only by copied byte buffers. The trusted-in-process D259 decision explicitly accepts incorrect native ABI declarations potentially crashing the process; it does **not** eliminate the need for truthful resource-lifetime contracts.

## 5. Adversarial cases and current vs future cost

| Counterexample / future scenario | G2 answer / required admission boundary |
| --- | --- |
| RootActor in direct CLI entry | Authorized native alias may be acquired by the RootActor; existing entry thread, no new worker |
| RootActor message-service entry on scheduler | Logical RootActor still authorized; use call-local or appropriately scoped region memory; no permanent native carrier pinning |
| Secondary Actor calls `ffi:<alias>` | Deny **before native lookup**; do not restrict unrelated foreign providers. No implicit broker |
| Secondary Actor imports `std:ffi` | Inert source module import remains allowed, no native authority |
| Allowed C library creates OS threads, uses filesystem/network/global state | Unrestricted by guest permissions and RootActor-only; D259 trusted host-process effects remain accepted |
| C library calls Protos from a new native thread | Protos callback ingress not promised; no invented Task/queue/reentry on foreign thread |
| Native library crashes process, wrong pointer layout or double-free | D259 accepts process-level risks; cannot promise recovery or safety impossible for arbitrary trusted C |
| `struct { char; int; }` with alignment, nested union or array | Correct ABI padding/alignment/layout required; test C `sizeof`/`offsetof`; no one-Java-class-per-struct |
| Struct by value return | `Linker` group result requires extra `SegmentAllocator`; scalar-only `NATIVE_SHAPES` must be expanded in actual invocation mechanism |
| One C library produces pointer and another consumes it | Respect pointer's physical representation and lifetime; do not deny solely because library alias differs. Both symbols still require host authorization |
| C returns pointer to unknown-size/borrowed memory | `MemorySegment` cannot derive size/lifetime; explicit trusted assertion/ownership semantics required before dereference |
| Reconstruct native address from guest Integer | Implicit conversion remains forbidden; an **explicit unsafe operation** was discussed but exact selection deferred |
| Native Image has no registered descriptor | Reject before native entry; do not coerce to a different ABI or claim all JVM signatures work |
| Program never calls FFM | No native library lookup or mandatory FFM session; measure any compiled image-size tax separately |
| Many Actors / P workers / Contexts | FFM ingress only RootActor per Protos Process; live resources remain nontransferable; OS process-global C state may still be shared by independent embedded Processes |
| Windows/LLP64, Darwin, different architectures | Reuse ABI-canonical type model; no untested host support claim |
| Long-lived callback retained by GObject/other native library | Deferred callback-lifetime and foreign-thread ingress contract; not assumed by choosing Java FFM |
| Additional C++/Qt object binding | Specialized C++ ABI layer deferred; a library's compatible exported ABI symbols are not banned by its implementation language |

**Pay as you grow:** all native resource acquisition remains explicit; no mandatory guest-native scheduler. **Grow as you need:** additional ABI layouts, Native AOT signatures, variadic calls, header generators, and later callback policies can be separate increments. **Deferral cost:** deferring the base layout/region model now forces rewrite of a byte-only FFI; deferring foreign-thread callback ingress and multi-Actor FFM preserves the model without requiring its scheduling policy now. **Reversibility:** specialized C shims remain possible at library level where absolutely unavoidable, but are not the default.

## 6. GITHUB021 owner invariant and delta reconciliation

| Owner-approved invariant / earlier decision | Assessment of owner-selected G2 |
| --- | --- |
| D238: Protos-first ordinary descriptors and Closures | **PRESERVED**, extended for generic ABI layouts |
| D238: native access from distinct Actors through Actor-local provider sessions | **INTENTIONALLY REOPENED:** RootActor-only FFM was explained as an R1 compatibility break **before** G2 recommendation and expressly approved in the next turn. Must be normatively published; no claim existing R1 behavior is preserved here |
| D259: host-only `--allow-library` default-deny | **PRESERVED**, no native acquisition from ordinary `std:ffi` import |
| D259: allowlisted code is `TRUSTED_IN_PROCESS` and may use OS resources | **PRESERVED**, specifically owner-reaffirmed unrestricted native thread/process effects |
| PLAT077: existing generic provider inside Native Image; finite AOT descriptors | **PRESERVED** as implementation substrate; broader R2 descriptors require actual metadata/invocation changes, not a promise of universal Native |
| Existing Actor/P nontransferability and no automatic proxy | **PRESERVED**, no guest FFM sharing, no broker |
| D188 error boundaries and identity | **PRESERVED** at architectural level; exact new public error cases pending |
| D189 callback dynamic extent / ACTORS §24J | **PRESERVED**, arbitrary native-thread ingress and retained callbacks not selected |
| RootActor is an ordinary Process-scoped Actor, not a constant OS thread | **PRESERVED**, correction accepted that direct CLI entry uses invoking thread while message-service may schedule segments |
| Protos-first minimal Java changes with verifiable Truffle precedents | **PRESERVED**, explicit methods/annotations/source anchors above, no per-library Java classes |
| Pay-as-you-grow | **PRESERVED**, omit speculative ingress/threads/binding generators; general C layout/memory/ABI is smallest coherent basis |
| Qt/C++ specialized interop deferred | **PRESERVED IN SCOPE:** specialized C++ class ABI not selected; compatible C-ABI exported symbols remain usable |
| D260 public Screen independent of backend | **PRESERVED**, not silently narrowed by this FFM record; PLAT078 checks downstream feasibility |
| Earlier `FFI.inBytes/outBytes` R2-C1 proposal | **SUPERSEDED UNAPPROVED PROPOSAL**; no evidence of public approval or implementation |
| Earlier exact names/upcall/numeric-address/Native build policy | **UNRESOLVED**, architectural approval cannot stand in for an exact public contract |

**GITHUB010_COMPARATIVE_RESEARCH=RECORDED.**  
**GITHUB021_OWNER_ARCHITECTURE_APPROVAL=PASS** (user selected G2).  
**GITHUB021_ARCHITECTURE_INVARIANT_CONSISTENCY=PASS for explicitly presented deltas**, notably RootActor-only versus D238.  
**EXACT_R2_PUBLIC_API_SELECTION=PENDING.**  
**SPEC_NORMATIVE_PUBLICATION=PENDING.**  
**PRODUCT_IMPLEMENTATION=PENDING.**  
**JVM_NATIVE_FFM_CONFORMANCE=NOT_RUN.**  
**LIB032_TERMINAL_B1_IMPLEMENTATION=NOT_RUN.**  
**PLAT078_BACKEND_SELECTION=PENDING.**

## 7. Downstream handoff, evidence integrity and validation

**D261 remains OPEN.** The owner chose the general architecture, not the exact R2 API. A further D261 public-contract decision must select final `FFI.*` names, true region/borrow/ownership semantics, ABI canonical type policy, output/aggregate behavior, unsafe native-address admission and initial callback scope. Only after that explicit agreement should an authorized human implementation slice publish normative `spec/semantics/FFI.md` and reconcile `MODULES.md`, `ACTORS.md`, `SQLITE.md`, tests and guide. Do not silently bypass the owner gate.

A generic C fixture (not terminal or SQLite product functionality) can prove ABI padding, pointer memory, by-value results, cross-authorized-library pointer passage, RootActor admit/secondary Actor refusal, resource release and JVM/Native metadata equivalence. **No fixture was compiled/executed as part of this documentation task**, and no JVM/Native Image output is inferred from previously published tests. No new production Java was edited. No tests, local Git commands, or local `git diff --check` were executed by this read/GitHub-publication session; source files and published GitHub revisions should be re-read by the exact returned commit.

The existing docs repository's AGENTS explicitly permits direct publication and the owner additionally authorized commit/push in this request; these permissions do **not** authorize changes to Protos `spec/` or Java production code.

**Records are historical non-normative evidence. GitHub issues remain the live work authority; Protos `spec/` remains the language authority.**
