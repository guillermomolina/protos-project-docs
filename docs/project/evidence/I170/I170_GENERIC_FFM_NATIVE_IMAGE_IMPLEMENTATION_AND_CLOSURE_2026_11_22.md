# I170 — Generic FFM provider integrated into standalone Native Image: implementation and closure evidence (2026-11-22)

**Kind:** Durable, revision-bound implementation and validation evidence; non-normative.
**Implementation issue:** [I170 #1148](https://github.com/guillermomolina/protos/issues/1148).
**Ratified platform authority:** [PLAT077 Candidate A, issue #1146](https://github.com/guillermomolina/protos/issues/1146), [canonical design](../../decisions/platform/PLAT077_GENERIC_NATIVE_IMAGE_FFM_PROVIDER_A_RATIFIED_2026_11_22.md) and [prior approval/GITHUB021 evidence](../PLAT077/PLAT077_A_GITHUB010_GITHUB021_APPROVAL_AND_IMPLEMENTATION_GATES_2026_11_22.md).
**Exact executable publication:** [`guillermomolina/protos@9c56aba2c8b84a9c44440cb361db85b53fcf30b4`](https://github.com/guillermomolina/protos/commit/9c56aba2c8b84a9c44440cb361db85b53fcf30b4), published to `main` on 2026-11-22.
**Implementation version:** `0.4.192-SNAPSHOT`, with matching `pom.xml` and root `CHANGELOG.md`.

## 1. Verified product revision and scope

The GitHub commit and changed-file inventory were read directly. Its parent is `1fa0c41728792596a1e4862f75298a2532b56bb1`, the pre-implementation PLAT077 inspection baseline. Subject: **`I170: Integrate the generic FFM provider into the standalone Native Image with lazy authorized library loading`**. The commit changes **26 paths (+1,354 / −45 lines)**, including Native-only build staging, the existing generic FFM implementation, provider loading, CLI/embedding, Native reachability metadata, smoke/distribution validators, JUnit and Python tests, public documentation, implementation version and changelog. No `spec/` file or third-party shared library is included in this change.

The product commit is verified read-only; the maintainer performed its local build/test validation. This record does not assert an independently executed post-commit CI job or a public release/tag.

## 2. Published implementation and authority boundaries

| Boundary | Revision-pinned product evidence | Verified implementation intent |
| --- | --- | --- |
| Native-only staging | [`pom.xml`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/pom.xml), [`scripts/native-generate-init-args.sh`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/scripts/native-generate-init-args.sh) | Copy application resources into `target/native-classes`, compile the **existing** `providers/ffm` sources plus ServiceLoader descriptor only for the Native Image, defer provider package initialization to runtime; preserve the ordinary portable shaded JAR and JVM provider path |
| Native/JVM host selection | [`ProtosExternalProviderPluginLoader.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/java/com/guillermomolina/protos/execution/foreign/providers/ProtosExternalProviderPluginLoader.java), [`ProtosPolyglotRuntimeHost.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosPolyglotRuntimeHost.java), [`ProtosCli.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/java/com/guillermomolina/protos/cli/ProtosCli.java) | Reuse the existing SPI and trusted descriptor; register the integrated provider in Native only when explicit `--allow-library alias=target` host options exist; no implicit JVM classpath discovery; Native rejects dynamic `--foreign-provider-path` |
| Provider lifetime and exact ABI | [`FfmBinding.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java), [`FfmSession.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmSession.java), [`FfmValueOperations.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmValueOperations.java) | Preserve host authorization before native-library acquisition, existing Actor-local sessions, foreign-value admission and owned/borrowed resource management; `ImageInfo.inImageRuntimeCode()` distinguishes Native-only finite-linkability admission from portable JVM support |
| Reachability registration | [Native `reachability-metadata.json`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json) | Finite **27-shape** physical downcall set using `jint` (C i32 carrier), `jlong`, `jdouble`, `void*` and `void`; synchronized with provider shape admission; no per-symbol/per-library table; unregistered shapes fail with an ordinary Error before C entry |
| Runtime compiler containment | [`ProtosRawForeignValue.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/java/com/guillermomolina/protos/runtime/ProtosRawForeignValue.java), [`ProtosForeignHandle.java`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/src/main/java/com/guillermomolina/protos/execution/foreign/api/ProtosForeignHandle.java) | `@TruffleBoundary` on provider-defined stable identity equality and foreign `each` capability probing, avoiding runtime compiler reachability explosion while retaining the existing optimizer |
| Distribution/provider regression | [`dist/build_ffm_provider.py`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/dist/build_ffm_provider.py), [`dist/validate_native.py`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/dist/validate_native.py), [`scripts/native-test.sh`](https://github.com/guillermomolina/protos/blob/9c56aba2c8b84a9c44440cb361db85b53fcf30b4/scripts/native-test.sh) | External JVM provider builder uses `lib/protos.jar` plus the exact matching distribution Native Image SDK API JAR **at compile time only**, packages no SDK classes; Native validator checks extracted launcher, absence of bundled SQLite and absence of SQLite `DT_NEEDED`, default-off, lazy authorization, real SQLite and unsupported shape; native smoke exercises independent C libraries |

The same installed Native executable is intended to support both `protos app.protos` and `protos --allow-library sqlite3=libsqlite3.so.0 app.protos`. The OS must supply the authorized shared object; the Native binary needs no consumer JDK, GraalVM, Maven or provider JAR. Authorization itself does not load any C library: acquisition occurs on `import("ffi:<alias>")`. `std:ffi` and `std:sqlite` remain ordinary Protos source libraries. C `i32` is an ABI carrier, never a guest Protos primitive `int`.

## 3. Source precedents for production Java changes

Exact analogous Truffle-language methods were inspected in the implementation handoff. These references document provenance here; they are not to be embedded in production Java identifiers or comments.

- GraalPy `NativeAccessSupport.createImpl` chooses Native/JVM host behavior with `ImageInfo.inImageCode()`; GraalPy `EmulatedPosixSupport.determinePid` and `NativePosixSupport.signalSelf` use `ImageInfo.inImageRuntimeCode()` for run-time-native behavior. See [GraalPy `NativeAccessSupport.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/nativeaccess/NativeAccessSupport.java), [`EmulatedPosixSupport.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/EmulatedPosixSupport.java), and [`NativePosixSupport.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/NativePosixSupport.java).
- GraalPy `PythonUtils.equals(Object,Object)` places `@TruffleBoundary` on a delegated `Object.equals` call; `InteropMap.hasKey(String)` places it on `Map.containsKey`, and `ConcurrentWeakSet.contains(E)` on collection membership. See [`PythonUtils.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/util/PythonUtils.java), [`InteropMap.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/interop/InteropMap.java), [`ConcurrentWeakSet.java`](https://github.com/oracle/graalpython/blob/master/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/util/ConcurrentWeakSet.java).
- The original ratified comparison documents GraalPy `GenerateNativeDowncallsProcessor` / `NativeDowncallMethodHandleGenerator.emitMethodHandleField` and TruffleRuby `FFMSupport.functionDescriptor` / `createDowncallHandle` / `NativeLibrary.open` as signature-registration and FFM acquisition precedents. The implementation reuses Protos's existing provider rather than adding a new backend.

## 4. Validation provenance: transcript versus owner attestation

**Directly read earlier maintainer-supplied local transcript (before the final publication commit):**

- `mvn test -Dtest='ProtosCliTest,ProtosWorkspaceRunCliTest,ProtosLm009DPublicDebugCliTest'`: **63 tests, 0 failures, 0 errors, 0 skipped; BUILD SUCCESS**.
- `mvn -Pnative package -DskipTests`: **BUILD SUCCESS** at `0.4.191-SNAPSHOT`, producing `target/native/protos` (reported executable file size **96.07 MiB**). This is a pre-version-finalization executable, not proof of the final versioned distribution archive.
- `scripts/native-test.sh`: **`NATIVE_I170_FFM=PASS cases=20`**, including authorization/default-off behavior, SQLite in memory and persistent reopen/TEXT/BLOB, independent C libraries, unknown symbol and unsupported signature; **`NATIVE_GUEST_JIT=PASS`**, **`BYTECODE_ROOT_TIER2_COMPILATIONS=2`**, **`BUILTIN_CALL_ROOT_TIER2_COMPILATIONS=2`**, **`OPT_FAILED=0`**, and **`NATIVE_REGRESSION_SUITE=PASS`**.
- The original Native compilation blocklist reachability failure (198 violations before the final two boundaries) was subsequently resolved: the successful Native build and guest Tier-2 results above are affirmative execution evidence after those repairs.

**Post-publication owner handoff (2026-11-22):** the owner explicitly reports that I170 was **pushed**, `git diff --check` is clean, and **all tests pass locally**. This is an owner-attested result on the published candidate, not an independently reproduced suite or automatically sourced CI status.

**Evidence boundary:** The transcript provided in this conversation does **not** separately contain final-`0.4.192-SNAPSHOT` `make test`, `dist/build_native.py --allow-dirty`, extracted Native ZIP `dist/validate_native.py --archive ...`, `dist/install.sh --local-native`, portable distribution provider-builder end-to-end output, or baseline-relative image-size/startup measurements. Their exact per-command results must not be invented. The owner’s broader statement that all local tests passed is recorded without promoting absent individual logs to directly observed evidence. The published scripts and validator checks are implementation coverage, not proof those specific distribution checks were executed.

## 5. Acceptance scope and limitations

- **Published:** one generic Native-integrated FFM provider and the existing JVM external-provider path, host authorization default-off, lazy OS library loading, finite physical ABI envelope, ordinary Protos source contracts, runtime compiler retained, Native FFM smoke/Tier-2 validation.
- **Not claimed:** unbounded dynamic Native signatures; other platform/ABI support; native callbacks, variadics, structs-by-value or C++ ABI; sandboxing arbitrary trusted native code; all consumer OS installations; a tagged public release or independently measured startup/size delta.
- **Separate downstream ownership:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) remains responsible for its own RealWorld application and its acceptance; completing I170 does not automatically complete I153 or change the already-ratified PLAT077 decision.

## 6. Revision-bound closure transaction

```text
FORMAL_IDENTIFIER=I170
ISSUE=guillermomolina/protos#1148
DESIGN=PLAT077-A
DESIGN_ISSUE=guillermomolina/protos#1146
PROTOS_REVISION=9c56aba2c8b84a9c44440cb361db85b53fcf30b4
PARENT_REVISION=1fa0c41728792596a1e4862f75298a2532b56bb1
IMPLEMENTATION_VERSION=0.4.192-SNAPSHOT
CHANGED_PRODUCT_PATHS=26
PRODUCT_COMMIT_VERIFIED=PASS
PRODUCT_VERSION_CHANGELOG=PASS
MAINTAINER_DIFF_CHECK=PASS_REPORTED
MAINTAINER_ALL_LOCAL_TESTS=PASS_REPORTED
PREPUBLICATION_CLI_FOCAL_TESTS=63_PASS_0_FAIL_0_ERROR_0_SKIP
PREPUBLICATION_NATIVE_BUILD=PASS
PREPUBLICATION_NATIVE_FFM=PASS_20_CASES
PREPUBLICATION_NATIVE_GUEST_TIER2=PASS
PREPUBLICATION_NATIVE_REGRESSION=PASS
FINAL_EXACT_SHA_FULL_SUITE_LOG=NOT_PROVIDED
FINAL_INSTALLED_NATIVE_ARCHIVE_LOG=NOT_PROVIDED
PUBLIC_RELEASE_TAG=NOT_CLAIMED
DOCS_PROJECT_RECORD=THIS_REVISION
```

The formal Issue may be closed as implemented/completed on the verified product publication and the owner's explicit local-PASS attestation, **without** claiming that the missing individual release/install transcripts were independently reviewed. PLAT077 remains ratified/closed and its original approval evidence remains historical.
