# D238 — Protos-first native FFI, approved B1-SQL-R1

**Decision status:** RATIFIED DESIGN SELECTION (2026-11-03); **not** a claim of implemented or published Protos Standard Library semantics.
**Authoritative issue:** [D238 #1047](https://github.com/guillermomolina/protos/issues/1047).
**Owner selection:** In the active design conversation, after the full B1-SQL-R1 exact-contract investigation and the assistant's explicit recommendation to approve that bundle, the project owner replied **“Aprobada”** on 2026-11-03. Prior owner checkpoints separately approved the B1-SQL direction and bounded native-memory/resource principles. [Direction checkpoint](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6606659205); [resource checkpoint](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6611817885); [R1 research proposal](https://github.com/guillermomolina/protos/issues/1047#issuecomment-6612262252).
**Source baseline:** [Protos `250b89bf2faf3e580980a67c2d69aa94d10e4df3`](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3), `0.3.442-SNAPSHOT`, `maven.compiler.release=21`, GraalVM `25.4.4.1.1`. Baseline is not an FFI implementation commit.
**Comparative and invariant evidence:** [D238 B1-SQL-R1 research and approval audit](../../evidence/D238/D238_B1_SQL_R1_RESEARCH_APPROVAL_AND_INVARIANT_AUDIT_2026_11_03.md).
**Authority boundary:** This project record is durable **non-normative rationale**. Observable Protos semantics belong to `guillermomolina/protos/spec/`; implementation, conformance and product publication are independent subsequent work.

## 1. Selection

Select **B1-SQL-R1**: an optional **Protos-first `std:ffi`** public facility for typed C-ABI downcalls through reusable ordinary Protos-source declaration modules. A generic, **explicitly host-authorized trusted-in-process Java FFM** provider supplies the privileged native substrate. Reuse existing import/module caches, D188/D189 foreign values and errors, D193 `std:interop`, I085 external-provider SPI, PLAT052 admission and PLAT053 Actor/Process provider lifecycle. No new parser syntax, foreign-value universe, callable category, module loader or native-address number type.

The initial practical proof is SQLite, split into a reusable `sqlite/abi.protos` declaration module and an idiomatic `sqlite.protos` consumer-facing module. The application imports the latter and should not need to know C signatures or Java FFM. Bindings are authored in Protos, not by adding one Java method per C symbol. A future optional generator can produce source declarations, but no generator is mandatory in R1.

The following public names are the **R1 design contract**, subject to exact publication in the normative Standard Library specification before they can be treated as available runtime API: `FFI.bind`, `FFI.opaque`, `FFI.owned`, `FFI.out`, `FFI.release` and supported scalar/nullable/string descriptors. Module spelling is `std:ffi`; logical host-authorized provider import is illustrated by `import("ffi:sqlite3")`. These names do not exist in the baseline Protos runtime.

## 2. Signatures, arguments and results

- Types/signatures are **immutable ordinary Protos descriptors**. Use fixed-width integer and binary floating descriptors, explicit ABI-dependent widths where needed, nullable opaque native pointers, NUL-terminated UTF-8 input, and `void`. The initial implementation need only support the scalar subset proven by SQLite; do not infer portable widths for C `long` or `size_t` from Java types.
- `FFI.bind(library, symbol, returnDescriptor, argumentDescriptors)` produces an **ordinary Protos callable/Closure** linked to a native entry point in the **exact host-authorized logical library**. A bound callable never outlives its originating provider session/library generation. Equivalent bind expressions are not promised canonical Closure identity.
- Native arguments are projected via D188/I085. Protos arrays describing a signature are processed in Protos and projected as supported scalar/handle provider operations; they do not silently become public SPI Array arguments. Invalid argument kind, range, arity, nullability or embedded NUL in `utf8z` fail **before** the native invocation.
- `FFI.out(T)` in a C signature denotes **hidden, initialized temporary output storage** allocated by the provider for that invocation; callers supply no argument in that position. No public mutable `T**` out-cell protocol is introduced.
- A downcall with **no outputs** returns its admitted scalar or `null` for `void`. A downcall with one or more `out` descriptors returns a **fresh ordinary Protos Array** `[nativeReturn, out0, out1, ...]` in C declaration order. The low-level layer does **not** interpret a nonzero SQLite status as automatic failure; the wrapper receives status and any non-null output for correct cleanup.
- Native C `NULL` becomes Protos `null` only in pointer/nullable value domains. Arbitrary native addresses are **never** exported as Integers. Incorrect C signatures, wrong annotations and unsafe native code remain faults of trusted ABI declarations: FFM cannot verify the binary signature.

## 3. Opaque memory and ownership

- `FFI.opaque(library, nativeTypeName)` declares a **type-tagged native reference** with logical identity within its provider/session/generation. Different resource generations remain distinct even if native address bits are later reused. Such references are nontransferable between Actors/P absent a separately authorized transfer contract; no auto-proxy, pointer arithmetic, arbitrary memory inspection, cast-from-integer or user-visible raw address.
- `FFI.owned(opaqueType, releaseSymbol, releaseReturnDescriptor)` declares an **owning resource-return shape**, with a native unary release symbol from the **same authorized library**. A resource produced through `FFI.out(ownedType)` acquires provider-side release custody **as part of result admission**, including non-null outputs of C calls that return an error code. No fallible “adopt later” gap. The release symbol is a native ABI function, **not** a Protos Closure callback transported through I085.
- `FFI.release(ownedHandle)` attempts that declared native release **synchronously** and retires the public resource token irreversibly. Native status is returned/admitted; failure of a C release is not permission to resurrect the token or retry a potentially recycled address. Provider/host retains whatever safe backend cleanup custody is required where the native release state is uncertain. No finalizer-based deterministic release promise.
- Borrowed memory is valid only within its declared native lifetime. The initial public/high-level access path must **copy exact bounded bytes before invalidation** and then use existing Protos `Bytes` and strict `Encoding.UTF8` semantics. The first SQLite proof **does not expose a generic persistent borrowed pointer**; wrong bounds/lifetime annotations remain an unsafe trusted-ABI error. SQL NULL is distinct from zero-length text/Blob.
- R1 does not add general C struct layout, pointer arithmetic, arbitrary raw buffers, variadic functions, callback export, retained callbacks, `SQLITE_TRANSIENT` sentinel construction or a universal native allocator. Those require separately specified extensions if needed.

## 4. SQLite lifecycle and errors

- The initial binding demonstrates in-memory connection open, prepare, numeric input binds, step, row materialization, finalization and connection close. A SQLite high-level module may construct useful ordinary query/each wrappers. Its `each` callback is invoked by Protos, not exported as C callback.
- `sqlite3_open_v2` may return a non-null connection even when its integer status reports failure; the **owned out-result** must be released rather than discarded. `sqlite3_prepare_v2` may succeed with a null statement (empty/comment SQL). The first implementation must not quietly ignore SQL after the first prepared statement.
- `sqlite3_column_text/blob` returns **borrowed data**: obtain type/length, copy before next step/reset/finalize, distinguish NULL and empty, and validate UTF-8 strictly for public Protos String. Reading a BLOB produces own `Bytes`, not implicit text.
- A Statement that adopts Protos `Closable` has a synchronous native `sqlite3_finalize` underneath a **fresh standard Future-returning `close()`**. It becomes permanently unusable at close cutover; `finalize` is invoked **at most once** even if it returns a prior execution error. Later `close()` calls observe the same recorded outcome with fresh Future identity, as required by `spec/io/IO_CORE.md`.
- A Connection `close()` rejects further use, finalizes its tracked dependent Statements, then invokes `sqlite3_close_v2`. Mandatory cleanup is still attempted after a prior child cleanup failure. The **first established cleanup failure** is the public primary failure; later cleanup failures do not overwrite it. Successful close/failed close outcomes obey the existing permanent, idempotent `Closable` Future contract. Native release bookkeeping is not automatically the same thing as the public close lifecycle.
- SQLite `SQLITE_STATIC` / `SQLITE_TRANSIENT` policy for **input TEXT/BLOB binding** is explicitly **deferred**, not inferred from `utf8z`. The R1 SQLite proof uses numeric input binds and safe copied text/bytes *reads*. Adding input text/blob binding later requires an approved lifetime or ABI-sentinel extension.
- D188 error boundary is preserved: Protos-side validation failure before foreign entry is an ordinary Error; actual provider-operation failures after entry use `ForeignError`. A SQLite C status is a native integer result unless and until the Protos SQLite wrapper explicitly interprets it as a library error. Native undefined behavior and process crashes are not catchable `ForeignError` promises.

## 5. Authority, execution and portability

- `std:ffi` import alone grants **no native authority**, opens no arbitrary library and does not discover ambient host libraries. The host registers/trusts the optional provider and maps exact **logical aliases** such as `ffi:sqlite3` to approved native libraries. The initial provider may support only that alias. Guest-controlled filenames, arbitrary library paths, fallback to unrestricted default symbols and a supposed sandbox-by-symbol-allowlist are **not** selected.
- The initial execution profile is **TRUSTED_IN_PROCESS**; code with this capability is not constrained by Protos `Filesystem` authority and can crash or access host state. It is inappropriate for untrusted code without separately specified strong isolation. Approval of one provider is not universal permission to run arbitrary C binaries.
- PLAT053 Actor-local provider sessions, per-Process lazy provider compartments, session-generation pinning and D188 identity persist. SQLite `serialized` mode is not by itself proof of Actor isolation. Owned connection/statement resources belong to their Actor; cross-Actor sharing is only through separately explicit service/capability boundaries. Native library global mutable state may not leak as an ordinary shared Protos object.
- Calls remain **synchronous** within the Actor's current execution segment; no hidden suspension, reentry or cancellation. FFM arenas and native library lifetimes must be safe for Protos scheduling and provider shutdown; a confined arena is not automatically equivalent to an Actor.
- **First validated target:** Linux x86-64 portable JVM with an optional **JDK 25** external FFM provider and explicitly provisioned SQLite library. Retain the Protos **Java 21 compilation target**. No compatibility promise for Windows/macOS, Native Image runtime-loaded plugin JARs, or a global native CLI as part of R1. No Java/FFM initialization cost is imposed when the optional provider is absent.

## 6. GITHUB021 preservation and newly selected R1 deltas

| Earlier owner-approved invariant | Exact R1 consequence | Check |
|---|---|---|
| Protos-first source modules/headers, SQLite as real target | Ordinary Protos declarations and SQLite wrapper | PRESERVED |
| One D188/I085/`std:interop` universe | Scalar/opaque handle projection only; no new call or import semantics | PRESERVED |
| Native authority explicit and optional; no sandbox claim | Host-authorized `ffi:` alias, trusted in-process provider | PRESERVED |
| Actors and session-generation isolation | Actor-owned resources, no transparent transfers or proxying | PRESERVED |
| Descriptors immutable; opaque pointers not numbers | Frozen descriptors and type-tagged logical handles | PRESERVED |
| Out parameters hidden and returned safely | Newly **selected** fresh result-Array shape | NO CONFLICT |
| Ownership at native acquisition; explicit cleanup | Newly **selected** `FFI.owned` and `FFI.release`, no adoption gap | NO CONFLICT |
| Borrowed native data copied before invalidation | Byte-length copy and strict decoding | PRESERVED |
| Standard `Closable` on public high-level wrapper | Fresh Future, irreversible cutover and no destructor retry | PRESERVED |
| Incremental pay-as-you-grow | No callbacks, structs, generators or Native Image first | PRESERVED |

**Delta finding:** The out-result Array shape, owned-descriptor/release operations, `close_v2` choice, primary cleanup error precedence, first supported platform and host alias authorization were introduced explicitly in the B1-SQL-R1 packet **after** the earlier direction/principles checkpoints and were presented before the final exact owner answer **“Aprobada.”** No prior owner invariant was silently contradicted or reopened. GITHUB021 consistency: **PASS for the selected design scope**; engineering validation and normative implementation are **not** implied.

## 7. Non-selected work and handoff

Meaningful alternatives were I085 Java-per-library status quo; Java-first public object model; full Python-ctypes-style unsafe FFI; generator-only Java bindings; mandatory dual dynamic/generated backend; or deferral. B1-SQL is chosen because it meets a concrete SQLite requirement while keeping the C-ABI exposure optional and preserving Protos-first binding authorship. Its strongest objection is that a wrong ABI signature or in-process native code can corrupt or terminate Protos: no amount of descriptor immutability turns C into a sandbox.

**Not approved:** native C++/Qt ABI, arbitrary pointer arithmetic, Native Image external-JAR hot loading, callback export, privilege confinement of trusted FFM, automatic host filesystem access from possession of an alias, and a general text-binding `SQLITE_TRANSIENT` escape hatch.

Subsequent normative Standard Library text in `guillermomolina/protos/spec/` must be published before implementation can be considered conformant. Implementations and tests must verify actual I085 routing, pre-entry errors, resource state transitions, SQLite open/failure/copy/close behavior, Actor isolation, compiler/OS/JDK gating and no unused-provider cost. A new materially observable choice discovered in implementation must return to explicit owner approval rather than being invented by an I/LIB slice.

This record is the approved **design**, not evidence of native FFI code/tests, specification updates, or a product release.
