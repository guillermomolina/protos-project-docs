# D259 — B2+A approval provenance, revision-bound source audit and implementation handoff

**Evidence date:** 2026-11-21. **Live design:** [D259 #1139](https://github.com/guillermomolina/protos/issues/1139). **New independent implementation:** [I169 #1140](https://github.com/guillermomolina/protos/issues/1140).
**Published design record:** [D259 B2+A at `649c0dfbc596fb43875ef3d102a20ddf6df08549`](https://github.com/guillermomolina/protos-project-docs/blob/649c0dfbc596fb43875ef3d102a20ddf6df08549/docs/project/decisions/language/D259_TRUSTED_NATIVE_FFM_ALLOWLIST_B2_A_RATIFIED_2026_11_21.md).
**Inspected Protos product revision:** [`2df251847ead94012f8d5a87c2b81d2d981882c7`](https://github.com/guillermomolina/protos/commit/2df251847ead94012f8d5a87c2b81d2d981882c7).
**Classification:** non-normative snapshot and reproducible investigation record. This is **not** evidence of a D259 code patch, a new product specification revision, or a successful D259 test run.

## 1. Explicit owner authority and exact selection

Chronological owner approvals recorded in the live D259 issue:

1. The project owner selected candidate **B**: implement a reusable generic `TRUSTED_IN_PROCESS` FFM provider with a host-supplied library allowlist, rather than keeping a SQLite-specific provider indefinitely.
2. The owner selected the **B2** configuration scope: authorization available both from CLI and Java embedding, reusing the I085/I144 infrastructure rather than introducing a second provider universe.
3. The owner corrected the trust boundary: **“no quiero asumir nada, quieero que al decirle a protos allow-library sqlit estas asumiendo que aceptas todos los riesgos de su uso”**. This is an instruction to accept *all* risks of the explicitly authorized native code, not an invitation to inspect or restrict libraries by their identity or behavior.
4. Following a presentation comparing **A explicit alias→target map**, **B inferred native name**, and **C general provider options CLI**, the owner answered **“si apruebo A”**. This approves option A in the approved B2 architecture, with empty-by-default library authority and no automatic SQLite exception.

The owner did **not** claim that the product implementation, normative specification, or D259 tests had already been published. Generic permission to update issues or documentation is not substituted for the explicit design selections above.

## 2. Revision-bound Protos inspection

At product SHA `2df251847ead94012f8d5a87c2b81d2d981882c7`, the existing implementation and normative owners are:

| Inspected source | Relevant established behavior / D259 delta |
| --- | --- |
| [`providers/ffm/.../FfmLibraries.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmLibraries.java) | `Policy` and `Authorization` exist but the distributable policy admits `sqlite3` with an implicit `libsqlite3.so.0` fallback; must become a host-defined empty-by-default alias map |
| [`FfmForeignProvider.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmForeignProvider.java) | `canonicalTarget` checks the current hardcoded policy; `openSession(environment)` receives host options; move authorization to acquisition without an SPI change |
| [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java) | Native Linker/Arena and `SymbolLookup.libraryLookup`; ensure unlisted alias fails **before** any native initialization/load |
| [`ProtosForeignProviderConfiguration.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/src/main/java/com/guillermomolina/protos/execution/foreign/providers/ProtosForeignProviderConfiguration.java) | Already offers immutable `trustedInProcess(List<Path>)` and `withProviderOptions(id, Map<String,String>)`; reuse for embedding |
| [`ProtosExternalProviderPluginLoader.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/src/main/java/com/guillermomolina/protos/execution/foreign/providers/ProtosExternalProviderPluginLoader.java) | Validates option provider IDs and supplies host options, not a general permission framework for FFM native libraries |
| [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java) | Existing `--foreign-provider-path` and separate guest permissions; `openApplicationRuntimeHost()` does not populate FFM options; `run`/`debug` have distinct parsing |
| [`ProtosForeignModuleImports.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/src/main/java/com/guillermomolina/protos/execution/foreign/api/ProtosForeignModuleImports.java) | Canonical module resolution precedes Actor-local acquisition; target acquisition occurs inside `ProtosForeignOperation.enter`, preserving D188 error projection |
| [`spec/semantics/FFI.md`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/spec/semantics/FFI.md) | `std:ffi` imports grant no authority; C-ABI R1, Actor-local managed resource custody, synchronous calls and native crash risks |
| [`spec/concurrency/ACTORS.md`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/spec/concurrency/ACTORS.md) | §24J synchronous foreign call semantics and §24K strict foreign-global Actor isolation; requires **explicitly approved trusted-native boundary delta**, not an implicit bypass |
| [`spec/io/PROCESS_IO.md`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/spec/io/PROCESS_IO.md) | Protos Process is distinct from OS process; independent guest bootstrap grants; no automatic capability expansion |

Test reference: [`ProtosI144FfmFixtureAbiTest.java`](https://github.com/guillermomolina/protos/blob/2df251847ead94012f8d5a87c2b81d2d981882c7/src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java) demonstrates embedding `withProviderOptions` for a host-selected native test fixture and is a direct extension seam, not evidence of D259 tests passing.

Verified GraalPy Java analogues for loader mechanics:
- [`NativeContext.loadLibrary(String name, int flags)`](https://github.com/oracle/graalpython/blob/0aada1a6bbbd266bca3584c307348ec8608bb4c1/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeContext.java) — uses `CompilerAsserts.neverPartOfCompilation()`, delegates to the native platform loader and tracks native-library lifetime.
- [`NativePosixSupport.loadNativeLibrary(PythonContext)`](https://github.com/oracle/graalpython/blob/0aada1a6bbbd266bca3584c307348ec8608bb4c1/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/NativePosixSupport.java) — `@TruffleBoundary`, runtime-selected path and explicit load failure handling.
- [`NativeAccessSupport.libraryLookup`](https://github.com/oracle/graalpython/blob/0aada1a6bbbd266bca3584c307348ec8608bb4c1/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeAccessSupport.java) — existing native symbol lookup helper.

These are equivalent **mechanism** precedents; the D259 trust decision and Protos-specific Actor semantics remain governed by the explicit owner approval and Protos specification, not copied from GraalPy.

## 3. GITHUB021 contradiction audit

- **Retained:** Protos-first FFI declarations and D188/I085 projection; explicit host authority; no new Protos native privilege from `std:ffi`; all ordinary guest I/O grants independent; bounded C ABI R1; lazy execution; managed Protos object/handle non-transferability.
- **Newly approved and explicit:** no native global-state isolation guarantee for an authorized trusted C library, including across Actors sharing one OS process; all native execution risks accepted by the host. This is the owner-directed scope correction of the stricter §24K default and MUST be published normatively before implementation.
- **Not selected:** infer names from `sqlite3`, a generic provider-option CLI as the main interface, auto-authorize SQLite, workers/IPC/Landlock/locking, symbol-specific sandboxing, C++/Qt ABI support, arbitrary raw pointers, default Filesystem or Network grants.
- **No unrelated gate clearance:** D256/PLAT074/LIB031 SQLite durable authorization and cancellation decisions are independent, not solved by the D259 design.
- **GITHUB021 result:** PASS **for approved design selection**, with explicitly documented normative adjustment outstanding. No contradictory unapproved semantics introduced.

## 4. Tests, local checkout and publication claims

The user reported **“el git diff check esta limpio. todos los tests pasan en local.”** on 2026-11-21. This is accepted as a user report of existing local validation, **not** a test log, an implementation publication SHA, or proof that unimplemented D259 functionality passes. At the inspected product revision, D259-specific behavior and tests have **not** been published. The agent did not run any builds or tests, and no D259 test result is asserted.

The design record is published at `649c0dfbc596fb43875ef3d102a20ddf6df08549` in the project docs repository. This evidence record has its own publication commit and must be referenced by **its exact SHA** in the live D259/I169 coordination update after publication and verification.

The next **independently tracked implementation** is [I169 #1140](https://github.com/guillermomolina/protos/issues/1140) (family:I, status:blocked pending normative publication). Mandatory first gate: publish `FFI.md`, `ACTORS.md` §24K and any truly affected `PROCESS_IO.md` cross-reference, with a new `spec/PROTOS_SPEC_CHANGELOG.md` entry; confirm new Protos commit. Then execute the scoped Java/CLI/embedding and tests under the human-executor and precise GraalPy-equivalence requirements. D259 design closure **does not** imply I169 completion.

```text
EVIDENCE_OWNER=D259
EVIDENCE_KIND=OWNER_APPROVAL_AND_REVISION_BOUND_HANDOFF
OWNER_APPROVAL=B2_PLUS_CONFIGURATION_A_AND_ALL_NATIVE_RISKS
GITHUB021_INVARIANT_CHECK=PASS_FOR_APPROVED_DESIGN
PROTOS_REVISION=2df251847ead94012f8d5a87c2b81d2d981882c7
DESIGN_RECORD_REVISION=649c0dfbc596fb43875ef3d102a20ddf6df08549
IMPLEMENTATION_ISSUE=I169/#1140
NORMATIVE_SPEC_PUBLISHED=NO
D259_IMPLEMENTATION_PUBLISHED=NO
D259_TEST_RESULTS=NOT_AVAILABLE
USER_LOCAL_VALIDATION_REPORT=CLEAN_DIFF_AND_ALL_LOCAL_TESTS_PASS_UNATTRIBUTED_TO_D259
EVIDENCE_RECORD_REVISION=THIS_RECORD_PUBLICATION_COMMIT
```
