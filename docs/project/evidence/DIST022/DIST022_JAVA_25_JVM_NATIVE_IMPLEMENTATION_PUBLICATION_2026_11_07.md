# DIST022 — Java 25 bytecode and JVM/Native distribution implementation publication

**Date:** 2026-11-07. **Owning issue:** [DIST022 #1085](https://github.com/guillermomolina/protos/issues/1085); **native parent and consumer:** [I156 #1083](https://github.com/guillermomolina/protos/issues/1083). **Approved platform contract:** [PLAT073-C](../../decisions/platform/PLAT073_JAVA_25_FFM_AMENDMENT_TO_PLAT072_E.md), preserving PLAT072/D248-B confinement requirements.

## Exact source publication

- **PROTOS_REVISION:** [`08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe) — verified GitHub commit on `main`, exactly one commit after [research/product pre-change revision `bfb276f9b76c663d3ef1fb68823e13a22ede136c`](https://github.com/guillermomolina/protos/commit/bfb276f9b76c663d3ef1fb68823e13a22ede136c); 19 changed paths.
- **IMPLEMENTATION_VERSION:** `0.3.453-SNAPSHOT` in the published `pom.xml` and first current `CHANGELOG.md` entry. The version/changelog bump was performed after the green substantive validation; the owner confirms no tests after the bump.
- **APPROVED FLOOR:** `toolchain.json: java.bytecode_release=25`, `pom.xml: maven.compiler.release=25`, Java classfile major **69**. Selected GraalVM Community **25.5.5.0.3**, JDK **25.0.5.0.3**, Native Image OL10 digest, Maven policy, and Truffle dependency versions remain pinned and unchanged.
- **IMPLEMENTED SCOPE:** update toolchain verifier and negative/drift fixtures; assert classfile major in shaded JAR and portable distribution; record and verify Java bytecode release in portable and Native runtime metadata; retain Java 21-targeted external-provider JAR compilation/linkage on the selected Java 25 host while also exercising a Java 25 FFM provider; update JVM embedding, CLI, provider and introduction documentation.
- **D242/I152 smoke repairs:** the extracted-toolchain Package Tool smoke now writes and validates `package.toml`; the embedding smoke imports the lowercase `std:test/assertions` MODULE and materializes the home-only marker as a MODULE directory with `module.protos` and an explicit `this.origin` export. The runtime implementation and language semantics were not changed by these smoke repairs.

## Human-executed validation before implementation-version bump

- **Diff/static admission:** the maintainer reports `git diff --check` clean; `python3 tools/tests/test_verify_toolchain.py` PASS and `python3 tools/verify_toolchain.py --mode check` PASS, including `TOOLCHAIN_DRIFT_COUNT: 0` and exact release 25 binding.
- **Focal JVM provider test:** `mvn -q -Dtest=ProtosExternalForeignProviderTest test` PASS. Portable distribution external-provider harness: **13/13 PASS**, including `PROTOS_CLASSFILE_BASELINE=PASS` (CLI major 69), `EXTERNAL_JAR_COMPILE=PASS`, `PROTOS_NATIVE_FFM_PROVIDER=PASS`, Java Math/Date providers, opaque/multiple providers, negative discovery and `DISTRIBUTION_UNMODIFIED=PASS`. Java 21 provider bytecode (major 65) is a *producer* compatibility test; running Protos on a Java 21 JRE is **not supported**.
- **Portable JVM ZIP:** `dist/build_portable.py --allow-dirty` → `DIST_BUILD: PASS`, classfile major **69**. `dist/validate_portable.sh --allow-dirty-source --archive ...` → `DIST001_B5_CROSS_SLICE: PASS` and all B2/B3/B4A/B4B gates PASS; bundled Test Tool suite **3,684 passed, 0 failed**; packaged Polyglot embedding, std/home/override/application-module cases all PASS; optimizing runtime is `com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime`, Java 25.0.5.0.3, Truffle/compiler 25.5.5.0.3.
- **Native Image:** `make build-native` → Maven `BUILD SUCCESS`, fresh executable `target/native/protos` (~55.32 MiB); `python3 dist/build_native.py --allow-dirty` → `NATIVE_DIST_BUILD: PASS` with CRC, layout, checksum and no-JVM-plane checks. Canonical build host OL **10.2**, target Linux **x86_64**, dynamically linked, declared minimum glibc **2.39**, observed max required glibc symbol **2.34**, DT_NEEDED `libc.so.6`, `libm.so.6`, `libz.so.1`; compatibility-march evidence recorded.
- **Native regressions:** `bash scripts/native-test.sh` → `NATIVE_REGRESSION_SUITE=PASS`. Version/help/guest execution, bundled Test Tool, four numeric conformance files, three TLS conformance files, DAP, LSP, regex and inspector authorization/denial PASS. PLAT045 interpreter-only fallback confirmed (`NATIVE_FALLBACK_RUNTIME_MARKERS=1`, `OPT_DONE=0`, `OPT_FAILED=0`, `COMPILATION_FAILURES=0`); no Native guest JIT support is claimed.
- **Native installed ZIP:** offline `dist/install.sh --inspect --local-native ...` and installation to an isolated temporary prefix → `LOCAL_NATIVE_VERIFIED=PASS`, `DIST022_NATIVE_INSTALL=PASS`, installed `--version` and guest `-e` execution PASS, plus `DIST022_NATIVE_INVALID_PREFIX_REJECTED=PASS` for a relative prefix; aggregate `DIST022_NATIVE_INSTALLATION_ACCEPTANCE=PASS`. No system-wide installation or network access was performed.
- **Final integrated gates:** maintainer's submitted `make test` log shows JVM/JUnit lanes **4,890 tests (0 failures, 0 errors, 4 skipped)**, **17 tests (0/0)**, **317 tests (0/0)**, and Protos **3,684 passed, 0 failed**. `make check` PASS, including generated Bytecode DSL guard, real Truffle compilation **15 completed, zero permanent / BCI failures**, and directory-limit **0 violations**. The owner independently reiterates `make test` and `make check` PASS locally, followed by a clean diff and product push.

## Tested artifact identities and provenance boundary

- **Portable JVM ZIP (pre-bump version):** `protos-0.3.452-SNAPSHOT-posix-jvm.zip`; reported SHA-256 `3e39cfdbe7f7510b8e41fb7330a5dbcfc50f250cb994c564f6b54a075bb73fc4`; `DIST_B5_SINGLE_ARCHIVE_CHECK=PASS`.
- **Native ZIP (pre-bump version):** `protos-0.3.452-SNAPSHOT-native-linux-x86_64.zip`; reported SHA-256 `952f939dfbcb9528bae7d4316ed318be41f3029b6a15e978411105cbeacdfda1`; `NATIVE_DIST_BUILD=PASS`.
- **Artifact source channel:** both development archives were built while the tracked checkout was dirty and `HEAD=bfb276f9b76c663d3ef1fb68823e13a22ede136c` (explicit `--allow-dirty`); they contain the validated substantive implementation **before** the post-test version/changelog edit. The later published product revision is `08c27760eb3f85105277e5b1b1b159abbca67ebe` with version `0.3.453-SNAPSHOT`. The ZIP checksums therefore **do not** certify a rebuilt `0.3.453-SNAPSHOT` release artifact or published distribution assets; no post-bump tests or rebuilds are asserted.
- **Host/negative evidence limit:** the installer rejected a relative prefix and portable B4B validated the exact selected JDK/runtime. An actual Java 21 JRE failure test, a real unsupported Linux host execution, and an official public-release installation are **not** asserted by this evidence.

## Scope and downstream coordination

- DIST022 completes the independent **Java 25 toolchain, SPI compatibility and distribution floor** selected by PLAT073-C. It does **not** implement the Linux confined Filesystem FFM-only bridge, libc `syscall` ABI, `openat2`/Landlock grants, worker isolation, or PLAT072 T01–T16 security proof. Those remain within **I156 #1083**; especially T04/T05 may not be inferred from green DIST022 packaging/tests. I155 #1079 remains independent.
- **Native hierarchy:** GitHub `issues/1085/parent` confirms I156 #1083; `issues/1083/sub_issues` includes DIST022 #1085.
- **Native dependency at evidence inspection:** GitHub `issues/1083/dependencies/blocked_by` listed the already closed PLAT072 #1082 and PLAT073 #1084, **but not DIST022 #1085**. The missing explicit `I156 blocked_by DIST022` edge still needs native GitHub reconciliation; text alone is not the relationship. Closing the now-green DIST022 does not complete I156, revoke other dependencies, or prove its security gates.
- **Publication authority:** product code and tested results were performed/reported by the human maintainer. This project-record publisher verified the exact product GitHub revision and its authoritative files, and records the supplied execution evidence; it did not independently run Maven, Native Image or install tests.

```text
FORMAL_IDENTIFIER=DIST022
ISSUE=guillermomolina/protos#1085
PROTOS_REVISION=08c27760eb3f85105277e5b1b1b159abbca67ebe
PROTOS_PREBUMP_BASELINE=bfb276f9b76c663d3ef1fb68823e13a22ede136c
IMPLEMENTATION_VERSION=0.3.453-SNAPSHOT
JAVA_BYTECODE_RELEASE=25
JAVA_CLASSFILE_MAJOR=69
GRAALVM_RELEASE=25.5.5.0.3
JDK_VERSION=25.0.5.0.3
HUMAN_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
PORTABLE_DIST_B5=PASS_REPORTED
NATIVE_DIST_BUILD=PASS_REPORTED
NATIVE_REGRESSION_SUITE=PASS_REPORTED
NATIVE_INSTALLATION_ACCEPTANCE=PASS_REPORTED
PRODUCT_PUBLICATION=GITHUB_VERIFIED
NATIVE_PARENT=I156_VERIFIED
I156_BLOCKED_BY_DIST022=MISSING_NATIVE_EDGE_AT_INSPECTION
I156_FFM_BRIDGE_SECURITY_PROOF=NOT_PART_OF_DIST022
NO_TESTS_AFTER_VERSION_BUMP=OWNER_WORKFLOW_REPORTED
```
