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
