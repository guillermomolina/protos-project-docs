# D238 — B1-SQL-R1 research, approval and GITHUB021 audit

**Date:** 2026-11-03.
**Issue:** [D238 #1047](https://github.com/guillermomolina/protos/issues/1047).
**Decision record:** [D238 Protos-first FFI B1-SQL-R1](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md).
**Exact source baseline:** [`guillermomolina/protos@250b89bf2faf3e580980a67c2d69aa94d10e4df3`](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3), version `0.3.442-SNAPSHOT`; Java source target 21 and GraalVM `25.4.4.1.1`.
**Scope:** Non-normative design evidence; not a claim that FFI has been built, tested or published as a Standard Library API.

## 1. Exact approval provenance

1. [First owner checkpoint](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6606659205): B1-SQL direction, Protos-first header modules, SQLite real use case, optional host-authorized Java FFM, existing I085/D188/PLAT052/PLAT053 reuse.
2. [Second owner checkpoint](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6611817885): immutable descriptors, hidden output storage, session-owned opaque pointers, copying borrowed memory before invalidation, explicit cleanup, no speculative pointer/callback universe.
3. [Full B1-SQL-R1 research checkpoint](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6612262252): proposed new exact result Array, `FFI.owned`, `FFI.release`, SQLite child-first close with `sqlite3_close_v2`, host logical aliases and Linux/x86-64/JDK25 initial target. At the time of that issue comment these were expressly unapproved.
4. **Exact candidate approval in this conversation, 2026-11-03:** after the assistant wrote **“Mi recomendación es aprobar B1-SQL-R1 y pasar a la especificación normativa y la descomposición en issues de implementación”**, the project owner answered **“Aprobada, Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs.”** This approves R1, not unspecified future features. Earlier GitHub checkpoint comments were agent-authored records, not owner-authored GitHub replies.

## 2. Current-revision Protos findings

| Inspected path at product baseline | Relevant constraint |
|---|---|
| `AGENTS.md`; `AGENTS.work/DESIGN.md`; `AGENTS.work/COORDINATION.md` | Exact owner approval, GITHUB021 invariant reconciliation, evidence and closure gates |
| [`spec/semantics/MODULES.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/semantics/MODULES.md) | Actor-local ModuleKey identity, cache, initialization, failure retry; no second import system |
| [`spec/semantics/VALUES_AND_COLLECTIONS.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/semantics/VALUES_AND_COLLECTIONS.md) | D188 scalar admission, identity, ordinary `call` and pre-entry versus `ForeignError` distinction |
| [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/concurrency/ACTORS.md) | No native mutable-state shared escape hatch, no implicit suspension or reentry, no resource auto-proxy |
| [`spec/io/IO_CORE.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/io/IO_CORE.md) | `Closable.close()` returns Future, commits irreversible cutover, records exact lifecycle outcome |
| [`spec/io/FILESYSTEM.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/io/FILESYSTEM.md) | Possession of path/String never grants native filesystem authority |
| [`spec/io/TEXT_IO.md`](https://github.com/guillermomolina/protos/blob/250b89bf2faf3e580980a67c2d69aa94d10e4df3/spec/io/TEXT_IO.md) | Strict UTF-8 decode, no silent corrupt SQLite TEXT |
| `ProtosForeignProviderPlugin.java`; `ProtosForeignPluginEnvironment.java`; `ProtosForeignPluginValueAdapter.java`; `ProtosForeignInteropFacility.java` | Existing I085 provider/handle/value operations; SPI callbacks rejected, no arrays/bytes as arbitrary outbound values |
| `protos/tests/foreign-provider/plugins/libc/src/i085b/libc/LibcProvider.java`; `dist/test_external_foreign_providers.py` | Concrete JDK25/Linux x86-64 FFM `strlen` fixture; not generic bindings |
| `bin/protos`; `pom.xml` | JVM launcher native-access flag; explicit provider-path loading; Protos release-21 compilation |
| [PLAT052](../../decisions/platform/PLAT052_FOREIGN_AUTHORITY_SANDBOX_ENFORCEMENT.md), [PLAT053](../../decisions/platform/PLAT053_FOREIGN_PROVIDER_RUNTIME_LIFECYCLE_ARCHITECTURE.md), [D193](../../decisions/language/D193_EXPLICIT_STD_INTEROP_PUBLIC_OPERATION_CONTRACT.md) | Existing privilege boundary, Actor-local sessions, possessed-value interop remain authoritative |

This list is the targeted source audit, not evidence that the new FFI was executed.

## 3. Primary technical references

**SQLite C API:** [open](https://www.sqlite.org/c3ref/open.html), [prepare](https://www.sqlite.org/c3ref/prepare.html), [step](https://www.sqlite.org/c3ref/step.html), [column text/blob](https://www.sqlite.org/c3ref/column_blob.html), [finalize](https://www.sqlite.org/c3ref/finalize.html), [close_v2](https://www.sqlite.org/c3ref/close.html), [bind_text](https://www.sqlite.org/c3ref/bind_blob.html), [threading](https://www.sqlite.org/threadsafe.html), [temporary files](https://www.sqlite.org/tempfiles.html).

**Java and alternative FFI prior art:** [JDK25 Linker](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Linker.html), [Arena](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/Arena.html), [SymbolLookup](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/foreign/SymbolLookup.html), [OpenJDK jextract](https://github.com/openjdk/jextract), [Graal Native Image FFM](https://www.graalvm.org/latest/reference-manual/native-image/metadata/), [Python ctypes](https://docs.python.org/3/library/ctypes.html), [CFFI](https://cffi.readthedocs.io/en/latest/overview.html), [LuaJIT FFI](https://luajit.org/ext_ffi.html), [Ruby FFI](https://github.com/ffi/ffi), [GraalPy native access](https://github.com/oracle/graalpython), [Truffle NFI](https://www.graalvm.org/latest/graalvm-as-a-platform/language-implementation-framework/Native-Function-Interface/), [.NET LibraryImport](https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke-source-generation), [Rust libloading](https://docs.rs/libloading/latest/libloading/), [GObject Introspection](https://gi.readthedocs.io/en/latest/).

## 4. GITHUB010 comparative matrix

Candidates: **A** existing per-library Java I085 provider; **B1** optional Protos-first FFI with generic FFM (selected); **B2** Java-first public FFI; **C** unrestricted ctypes clone; **D** mandatory generated bindings; **E** mandatory dual dynamic/generated solution; **F** defer all new work. Numbers 1–5; suffix H/M/L indicates evidence confidence, not measured latency.

| Dimension | A | B1 | B2 | C | D | E | F |
|---|---:|---:|---:|---:|---:|---:|---:|
| Correctness/invariants | 4H | 4M | 4M | 1H | 4M | 3L | 5H |
| Protos alignment | 3H | 5M | 2M | 1H | 3M | 3L | 3H |
| Pay only for present need | 4H | 4M | 2M | 1H | 2M | 2M | 5H |
| Incremental growth | 3M | 5M | 3M | 3M | 3M | 4L | 2M |
| Future-option resilience | 3M | 5M | 3M | 2M | 4M | 5L | 4M |
| Scalability | 3M | 3L | 3L | 1H | 4L | 3L | 5H |
| Conceptual simplicity | 5H | 4M | 2M | 1H | 3M | 2M | 5H |
| Portability | 3M | 3M | 3M | 2M | 3M | 3L | 5H |
| Runtime/resource cost | 5H | 4L | 3L | 2L | 4L | 2L | 5H |
| Failure/operability | 3M | 3M | 3M | 1H | 4M | 3L | 5H |
| Deferral/reversibility | 3M | 5M | 2M | 2M | 3M | 4M | 2M |
| Evidence maturity | 5H | 4M | 3M | 4H | 4H | 3L | 5H |

**Reasons and strongest counterargument:** A is proven but requires Java per function/library; B1 retains ordinary Protos headers and optional cost but has native lifecycle and Actor risks still needing tests; B2 duplicates a JVM-facing API, C exposes unbounded unsafe memory, D mandates a generator, E doubles backend obligations, F preserves status quo but cannot offer reusable bindings. **B1 still cannot prevent a bad ABI declaration or trusted native code from corrupting or crashing the process.** Immutable signature descriptions are not ABI verification. Scores do not compensate for this trust limit.

## 5. Adversarial cases that shaped R1

- `sqlite3_open_v2` may fail but return non-NULL: create owned out handle under provider custody at native result admission; no adoption gap.
- `sqlite3_prepare_v2` may yield null statement after successful blank SQL; may consume only the first statement in input: never invent successful prepared object or implicit multi-statement behavior.
- `sqlite3_finalize` destroys statement even if it returns prior execution error: invalidate handle on first call, reject later native reuse.
- `sqlite3_close` may report `SQLITE_BUSY`; `close_v2` may leave zombie while children remain: finalize tracked statements then close, maintain Protos `Closable` permanently terminal even when cleanup fails.
- Column pointers are borrowed; embedded NUL and invalid UTF-8 are possible; SQL NULL differs from empty BLOB: copy using precise SQLite byte length before step/reset/finalize and decode text strictly.
- `SQLITE_TRANSIENT` is a special destructor-function pointer sentinel: defer text/blob *input* binding rather than adding unsafe raw pointer construction. Numeric input binds plus copied TEXT/BLOB reads provide the first useful proof.
- Same native pointer address recycled: logical generation identity prevents stale wrapper resurrection.
- Two Actors import same native alias: separate module facades/sessions and owned resources; thread-safe SQLite does not by itself satisfy isolation.
- Unapproved alias or missing plugin: reject before native loading; no guest filename fallback. Native code can still create temporary files and bypass Protos Filesystem: trusted-in-process is **not** a sandbox.
- Java21 runtime with absent JDK25 plugin, macOS/Windows and Native Image: optional/no-use path remains functional; no untested platform claim.

## 6. GITHUB021 delta and preservation audit

| Approved prior invariant | B1-SQL-R1 outcome |
|---|---|
| Protos-first declarations, ordinary modules, SQLite use case | PRESERVED by source headers and SQLite wrappers |
| Reuse I085/D188/D193 and no second call/value/import universe | PRESERVED by scalar/handle projection and ordinary callable |
| Explicit native authority, not sandboxing | PRESERVED by host logical aliases and trusted provider |
| PLAT053 Actor-local sessions, no implicit resource transfer | PRESERVED by actor-owned opaque logical handles |
| Immutable signatures, no arbitrary pointer address | PRESERVED by frozen descriptors and no raw address escape |
| Hidden temporary outs, borrowed copies | PRESERVED; R1 **adds approved** fresh [returnValue, outs...] Array shape |
| Deterministic release and existing `Closable` | PRESERVED; R1 **adds approved** `FFI.owned`/`FFI.release`, finalize children, `close_v2` |
| Pay-as-you-grow; no baseline Java upgrade | PRESERVED; optional JDK25 plugin and deliberately deferred features |

**GITHUB021:** PASS for the selected R1 design. The additional observable choices were in the R1 packet before the owner's final **“Aprobada”**. No previously approved invariant was silently contradicted or reopened.

## 7. Limits and next evidence

No FFI implementation, tests, benchmark, spec version bump or portable Native Image support is asserted. Required implementation tests: provider absent; alias denied; signature pre-entry failures; out allocation/cleanup on failed SQLite open; copied text/invalid UTF-8; SQL null and empty blob; wrong call arity/type/range; statement finalize returning error with no double-free; close-v2 dependency order/first-error precedence; Actor/P transfer rejection and session retirement; physical library/arena lifetime; Linux JDK25 distribution verification; no unused-provider cost.

The durable decision record is non-normative. New observable implementation details must be reviewed under `AGENTS.work/DESIGN.md` and published to the proper `spec/` owner before release. Evidence and product revisions must remain separate.
