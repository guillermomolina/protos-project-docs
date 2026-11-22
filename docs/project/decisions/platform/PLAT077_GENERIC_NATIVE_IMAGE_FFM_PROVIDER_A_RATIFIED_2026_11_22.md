# PLAT077 — Generic trusted FFM support in the installed Protos Native Image

**Status:** RATIFIED, Candidate A, 2026-11-22. Architecture decision only; implementation and Native Image verification are pending.
**Formal issue:** [PLAT077 #1146](https://github.com/guillermomolina/protos/issues/1146).
**Implementation consumer:** [I170 #1148](https://github.com/guillermomolina/protos/issues/1148), independent I-family executable work.
**Explicit owner approval:** On 2026-11-22, after the comparative recommendation of Candidate A and its finite ABI-envelope limitation, the owner replied **"ok apruebo la sugerencia"** and authorized durable project-record publication. The owner had previously clarified: the same `protos` executable must work with and without `--allow-library`; include only the **minimum necessary generic FFM infrastructure**, if unavoidable, and **do not compile or bundle any third-party library** for speculative use.
**Source inspection revision:** [`guillermomolina/protos@1fa0c41728792596a1e4862f75298a2532b56bb1`](https://github.com/guillermomolina/protos/commit/1fa0c41728792596a1e4862f75298a2532b56bb1). This is the inspected product baseline, **not** an I170 implementation commit.
**GITHUB010/GITHUB021 comparison and provenance:** [PLAT077 evidence](../../evidence/PLAT077/PLAT077_A_GITHUB010_GITHUB021_APPROVAL_AND_IMPLEMENTATION_GATES_2026_11_22.md).

## Owner-selected architecture

Integrate the **existing generic** Java 25 FFM provider from `providers/ffm` into the **one** installed Protos Native Image at **build time**. Reuse the already-ratified I085 external-provider SPI, `ProtosForeignPluginProvider.trustedDescriptor`, `ProtosForeignProviderRegistry`, Protos-first `std:ffi` and `std:sqlite` modules, `FfmSession` Actor-local lifecycle, `FfmBinding` exact conversions and bindings, and D259 host authorization.

The Native image carries no SQLite, zlib, or other third-party native library binary, no library-specific Java adapter or symbol table, and no newly invented alternative FFI interpreter. Register physical FFM downcall signatures **by carrier shape**, not by function or library name, using the established GraalVM 25 Native Image `foreign.downcalls` reachability metadata path. The initial Native envelope must cover the physical signatures already needed by the Protos SQLite Standard Library and an **independent** authorized C-library/fixture proof, while remaining explicitly finite and incrementally extendable in future builds. A declaration outside that prepared Native envelope fails closed **before native entry** through the existing ordinary admission/error projection; a new guest-visible error family or altered normative admission contract is **not approved** by this decision.

Keep the existing portable JVM provider JAR build and explicit `--foreign-provider-path` discovery contract unchanged. Native registration of the built-in generic provider must be explicit and confined to the Native invocation with host-supplied authorization, not a new JVM classpath search, JAR loading promise, global provider auto-registration, nor guest-selected path. Only the host provides a `library.<alias> = <target>` mapping; neither FFI declarations nor imports elevate it.

Examples of required invocation behavior:

```sh
protos app.protos
protos --allow-library sqlite3=libsqlite3.so.0 app.protos
protos --allow-library zlib=libz.so.1 app.protos
```

The first command is the ordinary Protos execution path; no FFM session, Linker, Arena or C library is acquired merely because code for the provider exists in Native Image. `--allow-library` only declares authorization; a library is first loaded on authorized `import("ffi:<alias>")` acquisition. Importing `std:ffi` or `std:sqlite` alone loads no library and grants no capability. The second and third invocations are enabled only when the application actually declares ABI shapes in the supported Native envelope and the OS loader can find the authorized target.

## Architecture and packaging constraints

- Preserve `FfmForeignProvider` as the **same generic provider** for portable JVM and Native. No `sqlite3` constants, library names, hard-coded C bindings, per-library Java helpers, NFI/libffi second backend, JNI, JDBC, Sulong, worker, or hidden fallback.
- Preserve D259 B2+A `--allow-library alias=target`: case-sensitive host aliases, exact OS load name or absolute path, default deny, no relative paths, no inferred aliases, no extra search or privilege. Presence of the compiled provider is **not** authority; an authorized library runs `TRUSTED_IN_PROCESS` and may use the full OS process authority.
- Retain per-Actor session/cache, per-Process/Context ownership, synchronous native calls, borrowed epoch, bounded copies, owned-resource destructor custody and fail-safe retirement. No cross-Actor transfer, hidden Task, asynchronous interruption, or guaranteed native fault recovery.
- Do not snapshot live `Linker`, `Arena`, `SymbolLookup`, native address, or bound native `MethodHandle` into the Native image heap. Preserve existing Truffle optimizer/runtime compilation path and eliminate no canonical Bytecode DSL mechanism.
- Keep `providers/ffm` compiled as the **external JAR** for portable JVM as before. Only Native Image build staging may additionally compile/include those existing sources and their service descriptor. Do not accidentally include provider classes in the ordinary portable `protos.jar` distribution.
- Keep `dist/install.sh` and verified Native distribution as consumer entry; no consumer JDK, GraalVM, Maven, second executable or external provider JAR. Only the system's **authorized shared native library** and its OS dependencies need be installed by the OS/user. The Native executable must not have a mandatory `DT_NEEDED` dependency on SQLite.
- Scope initially to the existing Linux x86-64 LP64 / Java 25 / selected GraalVM 25.5 toolchain. No unsupported Windows/macOS/BSD/other-ABI, arbitrary C++/struct/variadic/upcall/callback or sandbox promise. C `i32` is an ABI carrier, **not** a Protos `int` primitive; Protos retains its long/double/boolean primitive design.

## Already-approved invariant reconciliation (GITHUB021)

| Owner/authority | Preserved invariant | New decision consequence |
| --- | --- | --- |
| D238 + I144 + LIB028/029/030/031 | `std:ffi` and SQLite are ordinary Protos source modules over one I085 provider; Java provider is generic | Same provider classes compile into Native; no alternate SQLite implementation |
| D259 + I169 | Host-only allowlist, exact target, default-deny, `TRUSTED_IN_PROCESS`; native calls are not sandboxed by guest Filesystem/Network | Built-in Native registration does not authorize or eagerly load anything |
| D256 `spec/semantics/SQLITE.md` | Actor-local Connection/Statement, synchronous operations, lifecycle, WAL and error policy | No change to SQLite source/module semantics |
| I085 + PLAT052/053 | Canonical plugin descriptors, foreign-value adaptation, Actor sessions, transfer/revocation and trusted profile | Built-in provider uses the existing SPI, not an independent registry or guest primitive |
| PLAT038 + PLAT073 | Self-contained installed Native Protos with optimizing Truffle runtime; Java 25 FFM supported for native boundary | Extend Native build without requiring a consumer JDK or weakening existing native release gates |
| GITHUB010 pay-for-what-you-need | Keep unused capability costs and new infrastructure minimal | Include generic provider code and finite ABI stubs only; measure idle Native cost |
| GITHUB021 owner corrections, 2026-11-22 | No third-party library compiled into the executable; minimum generic FFM is acceptable if necessary | Select build-time inclusion, reject separate dynamic native plugin as unnecessary |

**No prior semantic invariant is reopened.** The Native support **coverage envelope** is a newly selected platform availability promise, not a revision of `std:ffi` R1 declarations or of the already-authorized native ABI. Unsupported Native shapes are a scoped availability limitation, not permission to coerce types, bypass admission, reinterpret a function ABI or change guest-facing errors without a separate D-family approval. If the implementation cannot honor that boundary, stop and reopen the exact conflict for owner approval.

## Rejected and deferred alternatives

- **B: exhaustive combinations up to a maximum arity** — large speculative build-time stub set and an arbitrary ceiling that is not part of R1; do not prepay unused signatures.
- **C: NFI/libffi or other execution backend** — useful precedent for fully dynamic signatures, but would replace/rework the existing FFM binding/conversion/lifecycle backend; not required for this Native adoption goal.
- **D: native shared extension/second Native Image** — adds a separate ABI and lifetime boundary for the existing Java provider; not the smallest solution now.
- **E: external portable-JVM provider only** — keeps current behavior but fails the installed Native requirement.
- Future dynamic/unbounded signatures, other ABIs, native callbacks, C++ and additional native library semantics stay outside this ratification. Later evidence may justify a new decision, never an automatic fallback.

## Implementation handoff and hard acceptance gate

[I170 #1148](https://github.com/guillermomolina/protos/issues/1148) owns a **single coherent edit slice** covering Native-only build staging, built-in provider registration, exact downcall metadata, early missing-signature rejection, CLI/debug/embedding behavior, regression/fixture tests, native distribution/installer validation and accurate docs. New production Java, if any, needs method/annotation/behavior equivalence in a **verified Truffle-language source** recorded in handoff evidence, not references to other runtimes in production names or comments.

Prove with **actual human-executed** JVM and Native builds/tests and a real installed Native artifact: unchanged SQLite source operations (memory and durable reopen/WAL/TEXT/BLOB), a different C library ABI, authorization/no-load negative cases, Actor/session resource integrity, no unregistered downcall crash, JVM external-provider parity, and Native guest Tier-2 JIT/release gates. Record exact product/toolchain and artifacts before claiming implemented.

The owner's 2026-11-22 statement that local tests pass and `git diff --check` is clean predates any I170 change and is **owner-attested baseline evidence only**. This decision makes no executable product change and asserts **no** Native FFM PASS, distribution release or I170 completion.

## Source anchors

- Normative: [FFI](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/spec/semantics/FFI.md), [SQLite](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/spec/semantics/SQLITE.md), [Actors](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/spec/concurrency/ACTORS.md).
- Implementation: [FfmSession](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java), [FfmBinding](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java), [SQLite ABI](https://github.com/guillermomolina/protos/blob/1fa0c41728792596a1e4862f75298a2532b56bb1/protos/lib/sqlite/abi.protos).
- Platform specification: [GraalVM 25 FFM in Native Image](https://www.graalvm.org/jdk25/reference-manual/native-image/native-code-interoperability/ffm-api/) and [reachability metadata](https://www.graalvm.org/jdk25/reference-manual/native-image/metadata/).
