# DIST017-A — GraalVM/Truffle 25.5 GA source-level compatibility and identity preflight

**Status:** PUBLISHED SOURCE AUDIT; EXECUTABLE MIGRATION NOT STARTED; VALIDATION NOT RUN  
**Work owner:** [DIST017 / guillermomolina/protos#968](https://github.com/guillermomolina/protos/issues/968)  
**Audit date:** 2026-11-04  
**Protos product revision read:** [`250b89bf2faf3e580980a67c2d69aa94d10e4df3`](https://github.com/guillermomolina/protos/commit/250b89bf2faf3e580980a67c2d69aa94d10e4df3) (`main`, `pom.xml` project version `0.3.442-SNAPSHOT`)  
**Project-docs initial reference:** `f1a6eda730ce2703d13ee30c1d765fa61d704bbb`  
**Upstream GA source:** [`oracle/graal@e3cb74fceaf6ee4f50366d5ab2bbf602e464475d`](https://github.com/oracle/graal/tree/graal-25.5.5.0.3) (`graal-25.5.5.0.3`)  
**Provenance:** read-only inspection of repository files, GitHub issue/commit/release objects and official release notes. No Maven, Docker, Java, Protos build, test or runtime command was executed by the investigator.

## 1. Source-confirmed GA identity

- Official [GraalVM 25.5.5.0.3 release notes](https://www.graalvm.org/release-notes/25.5/) date the Community release **2026-11-03**; based on OpenJDK **25.0.5.0.3**. The source tag `graal-25.5.5.0.3` resolves to `e3cb74fceaf6ee4f50366d5ab2bbf602e464475d`, timestamped 2026-10-27. It is a released tag, not an EA build.
- The [Community release](https://github.com/graalvm/graalvm-ce-builds/releases/tag/graal-25.5.5.0.3) was published `2026-11-03T14:43:07Z`; it names the `25i5` channel and `graalvm-community-jdk-25i5-25.0.5.0.3_linux-x64_bin.tar.gz` with GitHub asset SHA-256 `c6438dee6c3ade2220d40b7ccdcad24b512d30e233e61bb4fc70519522de34ef`. That is **a tarball digest**, never a substitute for the digest of a container manifest.
- Official Community [container documentation](https://www.graalvm.org/docs/getting-started/container-images/) defines `25i5` feature channels, `-ol10` variants and distinguishes `jdk-community`, `graalvm-community` and `native-image-community`. No particular 25.5 OL10 manifest digest was authenticated in this audit.
- **Pending, do not invent:** independently confirm exact Maven Central artifacts and transitive closure (`org.graalvm.sdk:graal-sdk`, `org.graalvm.polyglot:polyglot`, `org.graalvm.truffle:truffle-api`, `org.graalvm.truffle:truffle-runtime`, `org.graalvm.polyglot:dap`, `org.graalvm.truffle:truffle-dsl-processor`), exact JDK `java -Xinternalversion`/JVMCI identity, existence and immutable multi-architecture digests of both `graalvm-community:25i5-25.0.5.0.3-ol10` and `native-image-community:25i5-25.0.5.0.3-ol10`, plus the derivative `protos-ci` image digest.

| Gate | Source finding | Execution verdict |
| --- | --- | --- |
| GA and exact upstream source tag | **CONFIRMED** via official release and tag | No local run |
| Community JDK archive SHA-256 | **CONFIRMED** via release asset metadata | Download/hash not repeated |
| Graal/Truffle/Polyglot/DAP/processor Maven closure | **PENDING** independent artifact resolution | NOT RUN |
| JDK/JVMCI exact runtime identity | **PENDING** execution under selected JDK | NOT RUN |
| OL10 development and Native Image manifest digests | **PENDING** registry manifest inspection | NOT RUN |
| Protos 25.5 compatibility and performance | **NOT TESTED** | NOT RUN |

## 2. Current Protos selectors and contracts

At the exact product revision above, `toolchain.json` is authoritative (`protos-toolchain-v2`): Community `25.4.4.1.1`, JDK `25.0.4.1.1`, channel `25i4`, matching Graal components, pinned OL10 image digest, `floating_primary_runtime=false`; Java bytecode target **21**; Maven minimum **3.9.9**. The root `pom.xml` uses `graalvm.version=25.4.4.1.1` for SDK, Polyglot, Truffle API/runtime, DAP and the annotation processor. It excludes `org.graalvm.*` from the Protos shaded JAR and materializes the canonical external runtime closure under `target/runtime`.

Live surfaces materially checked:

| Role | Current source location | Migration implication |
| --- | --- | --- |
| Primary authority | `toolchain.json` | Exact GA release/JDK/channel/immutable OCI digest; retain zero-float policy |
| Maven and generation | `pom.xml` | Matching published runtime and processor; preserve Java target 21 |
| Devcontainer | `.devcontainer/Dockerfile` | New exact Community OL10 image; retain `JAVA_HOME`-first PATH and Maven provisioning |
| Toolchain guards | `tools/verify_toolchain.py`, `tools/tests/test_verify_toolchain.py` | Reconcile projection expectations and stale-value test fixtures, without weakening detection |
| CI | `.github/workflows/tests.yml`, `.github/workflows/ci-image.yml`, `.github/ci-image-lock.json` | Rebuild derivative CI image and update only after obtaining its real digest |
| Native builder | `build/native/Dockerfile`, `dist/build_native.py` | Align with 25.5 OL10; existing native builder reference uses a tag without a digest |
| JVM distribution | `dist/build_portable.py`, `dist/smoke/smoke_optimizing_runtime.sh`, `dist/validate_portable.sh`, `bin/protos` | Exact `RUNTIME.txt`, optimizing runtime, Truffle compiler and DAP closure; extracted ZIP smoke |
| Composite artifacts | `dist/build_artifact_set.py`, `dist/verify_portable.py` | Preserve immutable source/runtime identity and manifest checks |
| Maintained user guide | `docs/guide/tools/cli.md` | Update active JVM requirement after tested adoption |
| Generated backend | `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java` | One production `@GenerateBytecode` interpreter; existing Yield, tag instrumentation, materialized locals, tail-call handlers and boxing-elimination features |
| Real debugger | `ProtosGraalDapReadinessAdapter.java`, `ProtosPolyglotRuntimeHost.java` | 25.4-specific readiness adapter must be tested against actual 25.5 DAP output, not rewritten on version alone |

Scope is compatibility adoption only, preserving ratified [PLAT033](https://github.com/guillermomolina/protos/issues/483): one external intact Graal/Truffle/Polyglot runtime authority, ordinary JVM optimizing runtime, no Graal Shade, no semantic change. Preserve [PLAT045](https://github.com/guillermomolina/protos/issues/772): JVM guest JIT; Native Image uses interpreter-only fallback `-Dtruffle.UseFallbackRuntime=true` until separately approved reassessment. Keep `maven.compiler.release=21` unless new concrete evidence requires a design decision.

## 3. Oracle Bytecode DSL source comparison: UPSTREAM005 is **not in the GA tag**

Related Protos owner: [UPSTREAM005/#893](https://github.com/guillermomolina/protos/issues/893). External report [oracle/graal#14682](https://github.com/oracle/graal/issues/14682), merged upstream [PR #14790](https://github.com/oracle/graal/pull/14790), fix [`095232aebe9d91bde82aa61e52aac32615bce96b`](https://github.com/oracle/graal/commit/095232aebe9d91bde82aa61e52aac32615bce96b) dated 2026-10-28.

Read exact released file:
[`BytecodeRootNodeElement.java` at `graal-25.5.5.0.3`](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/truffle/src/com.oracle.truffle.dsl.processor/src/com/oracle/truffle/dsl/processor/bytecode/generator/BytecodeRootNodeElement.java). Its `createUpdateBytecode()` still generates `oldBytecode.publishOldBytecodes()` **without the required `model.isBytecodeUpdatable()` generation-time branch**. In post-tag `master`, the merged patch surrounds emission with that branch and handles non-updatable nodes separately. The 25.5 GA tag was cut **before** the fix.

**Precise conclusion:** the tracked fix is absent from the released *tagged source*, independently of any runnable reproducer. The exact Maven processor and standalone no-tracing reproducer have **not** been compiled/tested on 25.5.5.0.3; record this as `FIX_IN_GA_TAG_SOURCE=NO`, `GA_MAVEN_REPRO=NOT_RUN`, not as a completed new execution failure. Neither Protos' default generated interpreter nor DIST017 compatibility migration is known to be blocked by the optional `enableInstructionTracing=false` scenario. Keep UPSTREAM005 open and separately classify impact after bounded verification.

## 4. Explicit 25.5 compatibility risks

- Upstream [Truffle 25.5 changelog](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/truffle/CHANGELOG.md) records **mandatory `ClearLocal`**. Search of current indexed Protos source did not identify a custom operation named `ClearLocal`; compilation of the actual generated bytecode interpreter remains the decisive check.
- The new [`GenerateBytecode.unwindExceptions`](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/truffle/src/com.oracle.truffle.api.bytecode/src/com/oracle/truffle/api/bytecode/GenerateBytecode.java) is present with default empty exception list and documented finally/handler behavior. **Do not enable** it during DIST017. Its production architecture is [PLAT070/#1049](https://github.com/guillermomolina/protos/issues/1049). [BUG025/#940](https://github.com/guillermomolina/protos/issues/940) remains an independent real-DAP bug; version change alone is not a fix.
- `TruffleFile.newByteChannel` now returns `FileChannel`. Current Protos read paths include the host filesystem backends, embedded filesystem custody and module resolution. Compilation/foreign-file-provider, permission and confinement tests must determine compatibility; do not presume a source break or API redesign.
- The Graal compiler adds automatically enabled and opt-in loop/guard optimizations. Treat new IR graphs and performance results as a distinct post-adoption baseline; do not overwrite 25.4 records or turn a change in node count into an unsupported regression/benefit claim.
- [UPSTREAM004/#753](https://github.com/guillermomolina/protos/issues/753) / [oracle/graal#14579](https://github.com/oracle/graal/issues/14579) reported a Bytecode DSL Native Tier-2 `FrameWithoutBoxing` failure on a 25.5 development snapshot. The GA must be checked only within existing Native interpreter-only contractual acceptance, not as permission to claim repaired Native guest JIT.
- JDK 25.0.5.0.3 includes certificate trust changes relevant to TLS; retain fail-closed trust behavior and run existing focal TLS acceptance under the new exact JDK.
- No Java target change, semantic or API redesign, JIT tuning flags, private Graal fork, publication/tag, or adopted 25.5 optional feature is authorized by this audit.

## 5. Work graph and bounded next actions

DIST017/#968 remains **OPEN / status:ready**. Independent investigations [PLAT069/#1048](https://github.com/guillermomolina/protos/issues/1048) (new 25.5 capabilities) and [PLAT070/#1049](https://github.com/guillermomolina/protos/issues/1049) (unwindExceptions) are **not** requirements for the as-is compatibility migration. [UPSTREAM001/#480](https://github.com/guillermomolina/protos/issues/480) is a closed historical experiment, not a current passing 25.5 test. [BUG013/#749](https://github.com/guillermomolina/protos/issues/749) and PLAT045 are closed on the supported interpreter-only Native capability; UPSTREAM004 remains open. UPSTREAM005 remains open.

1. **DIST017-A — exact preflight:** Human verifies 25.5 Maven graph/processor, `java -Xinternalversion`, OL10 images + manifest digests, `protos-ci` derivative image handling, and captures raw outputs. No canonical file mutation on unverified values.
2. **DIST017-B — toolchain edit:** After A, update all *live* selectors and tests coherently. Owner's explicit workflow: no `pom.xml`/ `CHANGELOG.md` version edits before the full final tests and immediately-before-publication step. If keeping the root POM unchanged prevents truthful compatible testing, stop and get a bounded validation mechanism rather than claiming the migration passed.
3. **DIST017-C — compatibility:** Human runs *focal* Java/Bytecode DSL, debugger/DAP, non-local control, cleanup, task, file provider, embedded Polyglot and compiler checks on the candidate. Neither positive compilation nor source inspection substitutes for actual DAP or dynamic correctness evidence.
4. **DIST017-D — closure:** Exact portable/extracted/Native smokes and full final `make test` + `make check` only at the final issue closure gate selected by implementation policy; reconcile the exact candidate, then finalize version/changelog once without rerunning tests and let the owner commit/push `guillermomolina/protos`.

**Immediate next handoff:** create a **human-managed Protos work branch** from verified HEAD, finish DIST017-A release/registry preflight and return the exact Maven, JDK/JVMCI, OCI manifest outputs. No branch creation, tests or repository-content publication in `guillermomolina/protos` was performed during this source investigation. For a later code edit, re-check moving `origin/main` and current versions. Any new blocking normative issue must be checked against `docs/project/registries/IMPLEMENTATION_BLOCKERS.md`, whose read did not expose a new DIST017-specific normative blocker.

## 6. Validation and provenance limits

```text
DIST017_A_SOURCE_AUDIT=COMPLETE
PRODUCT_HEAD_READ=250b89bf2faf3e580980a67c2d69aa94d10e4df3
PRODUCT_MODIFICATION=NONE
UPSTREAM_GA_TAG_SOURCE=e3cb74fceaf6ee4f50366d5ab2bbf602e464475d
GRAALVM_COMMUNITY_GA=25.5.5.0.3
JDK_CANDIDATE=25.0.5.0.3
JAVA_BYTECODE_RELEASE=21_UNCHANGED
GA_MAVEN_RESOLUTION=NOT_RUN
GA_OCI_MANIFEST_DIGESTS=NOT_VERIFIED
JVMCI_IDENTITY=NOT_VERIFIED
UPSTREAM005_FIX_IN_GA_TAG_SOURCE=NO
UPSTREAM005_GA_MAVEN_REPRO=NOT_RUN
PROTOS_25_5_BUILD_TEST_DAP_NATIVE=NOT_RUN
FULL_SUITE=NOT_RUN
POM_CHANGELOG_VERSION_CHANGE=NONE
MIGRATION_ACCEPTANCE=NOT_YET
```


## 7. Human-run DIST017-A Maven and OCI checkpoint — 2026-11-04

**Execution provenance:** the maintainer returned terminal output from the local `/workspaces/protos` checkout, inside container `f001205a6c7e`. These are **human-reported outcomes**; the coordinator did not run the commands or inspect the original local files. The branch name actually created was **`graalvm-25-5`**, superseding the previously suggested `dist017/graalvm-25-5`. The maintainer's pre-branch readback showed both HEAD and `origin/main` equal to `250b89bf2faf3e580980a67c2d69aa94d10e4df3`; the Git worktree status output was empty.

### Maven Central direct-artifact preflight — 6/6 exit 0

The maintainer ran `mvn -U -q dependency:get -Dartifact=...` separately for:

```text
org.graalvm.sdk:graal-sdk:25.5.5.0.3                       STATUS=0
org.graalvm.polyglot:polyglot:25.5.5.0.3                   STATUS=0
org.graalvm.truffle:truffle-api:25.5.5.0.3                 STATUS=0
org.graalvm.truffle:truffle-runtime:25.5.5.0.3             STATUS=0
org.graalvm.truffle:truffle-dsl-processor:25.5.5.0.3       STATUS=0
org.graalvm.polyglot:dap:25.5.5.0.3:pom                    STATUS=0
```

This establishes direct artifact resolution at the recorded coordinates; the **full project-effective dependency graph and source compilation remain NOT RUN**.

### OCI manifest-index preflight — two image indices verified

The maintainer used an authenticated anonymous GHCR registry manifest GET and compared the returned `Docker-Content-Digest` header with SHA-256 of the received manifest bytes. Both were reported PASS. Manifest-list/index values are **not** interchangeable with individual platform manifest digests or the JDK tarball digest.

| Repository / 25.5 OL10 tag | Verified index digest | Linux amd64 child | Linux arm64 child |
| --- | --- | --- | --- |
| `graalvm/graalvm-community:25i5-25.0.5.0.3-ol10` | `sha256:7a3e98abbb4af1f9dd085c09e2cb70d45a8abac53fd07277a50d2fbb6a3855f4` | `sha256:4e4995bdab038b0f469f3ff9e4f90566600d9c70acdd8f246373b74d456e6913` | `sha256:236cee15b46ba4fe4a15216073adce89c00156c5669b1eb2d16a9162824b96f8` |
| `graalvm/native-image-community:25i5-25.0.5.0.3-ol10` | `sha256:2800b5bd11cbb9bf11c92591d05ea37d9a735522bdc0e86827302d998843ad79` | `sha256:5413b3f9b57707f8252ec2313004fe4d62d5a37437b17a848ba35070ea8cf483` | `sha256:15ab4562c1a778e9bd49ffcaffcaa5dae214bf21def04b9ab3b204d7cf2120e0` |

**Status after checkpoint:** `DIST017_A_DIRECT_MAVEN=PASS_REPORTED`, `DIST017_A_OCI_INDEX=PASS_REPORTED`, `DIST017_A_FULL_MAVEN_GRAPH=NOT_RUN`, `DIST017_A_JDK_JVMCI=NOT_RUN`, `DIST017_A_PROTOS_COMPILATION=NOT_RUN`, `DIST017_A_DERIVED_CI_IMAGE_DIGEST=PENDING`. No `guillermomolina/protos` implementation file, `pom.xml` project version or `CHANGELOG.md` change was reported. This supersedes the preflight's earlier `GA_MAVEN_RESOLUTION=NOT_RUN` and `GA_OCI_MANIFEST_DIGESTS=NOT_VERIFIED` **for those two bounded checks only**, without rewriting historical observations.


## 8. Human-reported DIST017-B1 intermediate selector audit — 2026-11-04

**Scope:** local uncommitted Protos branch `graalvm-25-5` from `250b89bf2faf3e580980a67c2d69aa94d10e4df3`; report supplied by the maintainer after running the Python editing blocks. **This is intermediate uncommitted workspace evidence, not a versioned product commit.**

Eight edited product paths: `toolchain.json`, `.devcontainer/Dockerfile`, `build/native/Dockerfile`, `dist/build_native.py`, `dist/build_portable.py`, `dist/smoke/smoke_optimizing_runtime.sh`, `tools/verify_toolchain.py`, and `tools/tests/test_verify_toolchain.py`. Runtime selectors set to candidate Graal/Truffle 25.5.5.0.3 / JDK 25.0.5.0.3 with confirmed GHCR index digests (section 7); `toolchain.json` additionally gains a pinned `graalvm.native_container_image`, with fail-closed verification of its digest and distribution-builder binding. No user-reported `pom.xml` or `CHANGELOG.md` modification.

Two focal test failures occurred while preparing the new validator; they were **patch defects**, not GraalVM 25.5 compatibility results: (a) overescaped/native validation regex and ambiguous patch anchor; (b) test fixture `dist/build_native.py` construction generated the wrong content and was rejected by the newly added binder. Subsequent bounded Python fixes were applied and the maintainer supplied the final results:

```text
B1_APPLIED_FILES=8
BRANCH=graalvm-25-5
TOOLCHAIN_VERIFIER_TESTS=PASS
VERIFIER_TEST_STATUS=0
DEVELOPMENT_CHECK_STATUS=0  # reported in prior bounded run
NATIVE_IMAGE_BINDING=PASS
NATIVE_DISTRIBUTION_BUILDER_BINDING=PASS
TOOLCHAIN_DRIFT_COUNT=1
TOOLCHAIN_DRIFT=pom.graal_components
DIFF_CHECK_STATUS=0
POM_PROJECT_VERSION=UNCHANGED
POM_GRAALVM_PROPERTY=25.4.4.1.1  # intentional intermediate incompatibility
CHANGELOG=UNCHANGED
PROTOS_25_5_SOURCE_COMPILE=NOT_RUN
FULL_SUITE=NOT_RUN
```

The remaining `pom.graal_components` drift is **expected but not accepted as final**: direct Maven compilation must temporarily override `-Dgraalvm.version=25.5.5.0.3` and use the actual GraalVM 25.5 JDK; final canonical POM alignment must be separately validated before DIST017 acceptance. The project version/changelog bump is reserved for after the final full-suite validation, per owner instructions.

## 9. Human-reported 25.5 devcontainer/JDK/JVMCI runtime acceptance — 2026-11-04

**Execution provenance:** maintainer's terminal output from rebuilt development container `43c2b83b157c`, checked out on local branch `graalvm-25-5`. Product base revision as previously captured `250b89bf2faf3e580980a67c2d69aa94d10e4df3`; eight local paths modified by DIST017-B1, no `pom.xml` or `CHANGELOG.md` modifications reported.

The maintainer's rebuilt devcontainer reports the following exact identifiers:

```text
java.version=25.0.5.0.3
java.vendor.runtime=GraalVM CE 25.5.5.0.3+1.1
java.vm.build=25.0.5.0.3+1-jvmci-25.5-b26
java.internal.version=25.0.5.0.3+1-jvmci-25.5-b26
java.internal.build_timestamp=2026-10-27T22:27:33Z
java.internal.platform=linux-amd64
maven=Apache Maven 3.9.9 (Red Hat 3.9.9-3)
maven.java.runtime=/opt/graalvm-community-java25i5
runtime_verifier=PASS (RUNTIME_STATUS=0)
runtime_toolchain_drift=0
development_toolchain=PASS (TOOLCHAIN_STATUS=0)
development_toolchain_drift=0
```

The runtime verifier explicitly marked `runtime.maven.version`, `runtime.maven.java_version`, `runtime.maven.java_vendor`, `runtime.maven.java_runtime`, and `runtime.redundant_openjdk_rpm` as `status=PASS`; `make toolchain` checked the selected dev image pinned by index digest, Java bytecode release 21, Maven installation/repository scope, `JAVA_HOME`-first PATH, CI image binding and CI Maven cache, all `status=PASS`. The `protos-ci` image is still the **old digest** `sha256:94c01739a95d6bbcb197180b86ef3aaa8c429483686d29d2b12d50ae35028fc1`; consistency of the development-scope selector does not prove that derivative CI image was rebuilt for 25.5. Also, development-scope verification **does not** check the still intentionally drifting `pom.graal_components` binding. No 25.5 source compile or full-suite outcome was reported yet.

**Distinction:** An earlier attempt to download the official 334 MB x64 JDK tarball inside the old container failed with `curl (23)` during local file write at approximately 76% completion; this is a local I/O/storage failure, not an upstream 25.5 compatibility failure. The maintainer instead rebuilt the actual existing devcontainer using its pinned 25.5 image and obtained the passing results above. No need for a second manually unpacked JDK.

```text
DIST017_B1_FOCAL_TOOLCHAIN_TEST=PASS_REPORTED
DIST017_C1_25_5_RUNTIME_IDENTITY=PASS_REPORTED
DIST017_C1_25_5_JVMCI_IDENTITY=VERIFIED_BY_JAVA_XINTERNALVERSION
DIST017_C1_25_5_SOURCE_GENERATION_AND_COMPILE=NOT_RUN
DIST017_D_NATIVE_AND_PORTABLE_VALIDATION=NOT_RUN
DIST017_D_FULL_SUITE=NOT_RUN
POM_VERSION_AND_CHANGELOG=UNCHANGED
```

## 10. Human-reported 25.5 source compilation and focused C2 JUnit runtime acceptance — 2026-11-04

**Execution provenance:** the maintainer ran the commands in the rebuilt 25.5 Community devcontainer on the local, uncommitted Protos `graalvm-25-5` branch. These are *human-reported logs*, not tests run by the coordinator. The root `pom.xml` still declares `<graalvm.version>25.4.4.1.1</graalvm.version>`, so both commands explicitly supplied `-Dgraalvm.version=25.5.5.0.3` to test the 25.5 Maven Truffle/Polyglot/annotation-processor dependency plane without changing the POM prematurely; this override is not a committed toolchain alignment.

**Clean production compile:** `mvn -U -Dgraalvm.version=25.5.5.0.3 -DskipTests clean compile` reported `BUILD SUCCESS`, compiled 509 Java source files with `javac [debug release 21]`, elapsed 18.769 seconds, and `git diff --check` exit 0. The previously captured B1 checkpoint was preserved before Maven `clean`.

**Focused execution:** `mvn -Dgraalvm.version=25.5.5.0.3 -Dtest=ProtosI109ABytecodeMaterializationTest,ProtosI109ADynamicLocalEachTest,ProtosI115ARuntimeAdmissionTest,ProtosI115BDeferredCallerTest test` reported:

| JUnit class | Tests run | Failures | Errors | Skipped |
| --- | ---: | ---: | ---: | ---: |
| `ProtosI109ABytecodeMaterializationTest` | 4 | 0 | 0 | 0 |
| `ProtosI109ADynamicLocalEachTest` | 6 | 0 | 0 | 0 |
| `ProtosI115ARuntimeAdmissionTest` | 15 | 0 | 0 | 0 |
| `ProtosI115BDeferredCallerTest` | 22 | 0 | 0 | 0 |
| **TOTAL** | **47** | **0** | **0** | **0** |

The maintainer's `DIST017_C2_RUNTIME_TEST_STATUS=0`, `BUILD SUCCESS`, and `DIFF_CHECK_STATUS=0` are reported. These tests cover materialization, dynamic callback suspend/resume and nonlocal control, and I115 closure/caller behavior on the 25.5 candidate. Real debugger/DAP, full dependency packaging, Native Image, portable distribution, new CI image, and required end-of-item full tests remain **NOT RUN**. The intermediate POM drift `pom.graal_components` remains until the owner-authorized final POM update; do not misstate `make toolchain` development-scope PASS as all-surface full alignment.

```text
DIST017_C1_SOURCE_COMPILE=PASS_REPORTED
DIST017_C1_COMPILED_JAVA_SOURCES=509
DIST017_C2_FOCAL_JUNIT=47_PASS_0_FAIL_0_ERROR_0_SKIP
DIST017_C2_FULL_SUITE=NOT_RUN
DIST017_C3_REAL_DAP=NOT_RUN
DIST017_D_NATIVE_PORTABLE_CI=NOT_RUN
POM_AND_CHANGELOG=UNCHANGED
```


## 11. Human-reported DIST017-C3 25.5 DAP lifecycle failure and upstream source attribution — 2026-11-04

**Human-reported focal run (failure):** in the rebuilt JDK 25.0.5.0.3/GraalVM 25.5.5.0.3 container, the maintainer ran `mvn -Dgraalvm.version=25.5.5.0.3 -Djunit.jupiter.execution.parallel.enabled=false -Dtest=ProtosI026FDapTransportTest,ProtosI026FDapBehaviorTest,ProtosI116OptimizedDapAcceptanceTest test`. Result: `DIST017_C3_DAP_STATUS=1`, **5 tests, 5 failures, 0 errors/skips**, `BUILD FAILURE`, `DIFF_CHECK_STATUS=0`. All five assertions failed in test teardown, not at the earlier DAP handshake/breakpoint/value assertions:

- `ProtosI026FDapTransportTest` 1/1 failed in `awaitDapClientConnectionThreadTermination()` (line 179).
- `ProtosI026FDapBehaviorTest` 1/1 failed at the same helper (line 341).
- `ProtosI116OptimizedDapAcceptanceTest` 3/3 failed at the imported helper (test common cleanup line 452).

Exact common error: `GraalVM DAP client connection thread must terminate before Context.close() ==> expected: <false> but was: <true>`. Each test sends and awaits a successful `disconnect` response and closes client socket, then waits up to 8 seconds for DAP's *listener* thread to die **before** closing its Context/Engine. This ordering is the pre-25.5 assumption being tested, not an established 25.5 protocol requirement.

**Independent released upstream source inspection identifies a 25.5 DAP architecture change**; no product patch or test rerun has yet verified the resolution:

- [25.5 `DAPInstrument.java`](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/tools/src/com.oracle.truffle.tools.dap/src/com/oracle/truffle/tools/dap/instrument/DAPInstrument.java): engine instrument now creates a `DebugProtocolServerListener` and calls its `dispose()` on instrument `onFinalize` rather than directly constructing `DebugProtocolServerImpl`.
- [25.5 `DebugProtocolServerListener.java`](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/tools/src/com.oracle.truffle.tools.dap/src/com/oracle/truffle/tools/dap/server/DebugProtocolServerListener.java) explicitly states it owns the Engine's listening socket, serves **one debugger client at a time**, and permits fresh sessions on successive client connections. Its `run()` loops accepting new clients **after** a successful disconnect. `dispose()` closes the listening and client sockets and joins `DAP client connection thread` on Engine teardown.
- [25.5 `DebugProtocolServerImpl.java`](https://github.com/oracle/graal/blob/graal-25.5.5.0.3/tools/src/com.oracle.truffle.tools.dap/src/com/oracle/truffle/tools/dap/server/DebugProtocolServerImpl.java) `disconnect()` now disposes the **session** while keeping the listener alive for reconnect. In [25.4.4.1.1](https://github.com/oracle/graal/blob/graal-25.4.4.1.1/tools/src/com.oracle.truffle.tools.dap/src/com/oracle/truffle/tools/dap/server/DebugProtocolServerImpl.java), `dispose()` called `clientConnectionExecutor.shutDownAndJoin()`. This distinction explains why the 25.4-specific `join`-before-`Context.close()` assertion fails on 25.5.

**Bounded correction hypothesis:** retain all DAP handshake, breakpoint, visibility and nonlocal-control checks; update only test teardown to perform normal Context/Engine close *before* asserting the listener thread terminates. This should preserve leak detection without discarding it or increasing timeouts. The implicit-engine case in the transport test needs an actual focal retest; if it does not stop the thread after `Context.close()`, inspect engine ownership explicitly rather than suppress the failure. No evidence has established BUG025 resolution or Native/portable readiness.

**POM distinction clarified by owner:** the root project `<version>0.3.442-SNAPSHOT</version>` must remain frozen until the final test gate, but the unrelated dependency selector `<graalvm.version>...</graalvm.version>` **should** be updated to 25.5.5.0.3 now so Maven selects the canonical processor and runtime without manual flags. No such POM edit is claimed as completed before the maintainer applies the next Python patch.

```text
DIST017_C3_DAP_FOCAL_INITIAL=FAIL_5_OF_5
COMMON_FAILURE=PRE_25_5_LISTENER_TERMINATION_ORDER
UPSTREAM_25_5_LISTENER_RECONNECT_DESIGN=SOURCE_CONFIRMED
PROTOS_DAP_TEARDOWN_FIX=PROPOSED_NOT_APPLIED
PROTOS_DAP_RETEST=NOT_RUN
POM_DEPENDENCY_SELECTOR_CHANGE=PROPOSED_NOT_APPLIED
PRODUCT_VERSION_CHANGE=NONE
FULL_SUITE=NOT_RUN
```
