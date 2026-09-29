# DIST005-D4 — clean Native candidate proof

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D4 — clean Native candidate proof`

## Published implementation identity

DIST005-D4 implementation is published on `guillermomolina/protos/main` as:

```text
IMPLEMENTATION_BASE_REVISION=96f066c3689359afc0f03d6760de171bdfe2311f
IMPLEMENTATION_REVISION=1e64469eb781c2ae5a15087d80b0cd4ad0e74214
IMPLEMENTATION_VERSION=0.3.113-SNAPSHOT
COMMIT_SUBJECT=DIST005-D4: add clean Native candidate proof
```

Changed implementation surfaces:

```text
CHANGELOG.md
dist/build_native.py
dist/native_elf.py
dist/release_asset_envelope.py
dist/test_multi_asset_release_metadata.py
dist/test_native_build_environment.py
dist/test_native_elf.py
dist/test_native_public_release_mode.py
dist/validate_native.py
pom.xml
```

No normative Protos language or Standard Library specification file changed.

## Exact detached proof candidate

A release-only detached candidate was materialized from the published D4
implementation baseline:

```text
PROOF_CANDIDATE_BASELINE=1e64469eb781c2ae5a15087d80b0cd4ad0e74214
PROOF_CANDIDATE_SOURCE_REVISION=4812007bcf48533b89b14a5c6c0a7e8b24b1a992
PROOF_CANDIDATE_VERSION=0.3.113
PROOF_CANDIDATE_TAG_IDENTITY=v0.3.113
PROOF_CANDIDATE_CHANGED_PATHS=pom.xml
CANDIDATE_BRANCH=DETACHED
CANDIDATE_CLEAN_CHECK=PASS
CANDIDATE_RELEASE_ONLY_DIFF=PASS
```

The proof candidate deliberately created no public Git tag, GitHub Release, or
published release asset.

## Native build environment and build-policy proof

The exact candidate was built in the selected Oracle Linux 10 environment.

Observed build environment:

```text
native_build_host_os_id=ol
native_build_host_os_version=10.2
native_build_host_glibc=2.39
canonical_native_build_container=ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10
```

Selected and retained public Native policy:

```text
PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
PUBLIC_NATIVE_ABI_POLICY=GLIBC_2.39
PUBLIC_NATIVE_GLIBC_MIN=2.39
```

The Native Image builder itself reported:

```text
Java version=25.0.4.1.1
GraalVM CE=25.4.4.1.1
target machine=compatibility
C compiler=gcc 14.3.1 x86_64
runtime compiled methods=2137
```

## Empirical ELF and ABI evidence

D4 deliberately separates the selected build policy from observable post-link
ELF evidence.

Observed ELF identity:

```text
NATIVE_ELF_OS=linux
NATIVE_ELF_ARCH=x86_64
NATIVE_LINKAGE=dynamic
NATIVE_LIBC_FAMILY=glibc
NATIVE_ELF_INTERPRETER=/lib64/ld-linux-x86-64.so.2
NATIVE_DT_NEEDED=libc.so.6,libm.so.6,libz.so.1
NATIVE_DYNAMIC_LIBRARY_CLOSURE=libc.so.6,libm.so.6,libz.so.1
```

Observed GLIBC symbol versions:

```text
2.2.5
2.3
2.3.2
2.3.4
2.4
2.6
2.7
2.9
2.14
2.15
2.17
2.32
2.34
```

Therefore:

```text
OBSERVED_GLIBC_MAX=2.34
PUBLIC_NATIVE_GLIBC_MIN=2.39
GLIBC_ABI_FLOOR_PROOF=PASS
```

The binary's observable GNU x86 ISA note was:

```text
POST_LINK_CPU_ISA_EVIDENCE=x86-64-baseline,x86-64-v2,x86-64-v3
```

This evidence is not reinterpreted as a baseline-only binary. The selected
build policy remains the independently proven:

```text
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
BUILD_CPU_POLICY_PROVEN=YES
```

## Native artifact identity

The clean candidate produced:

```text
NATIVE_BINARY_SHA256=d6c3aa2ae57f206d55fe910a9d7777d7bb6c373e66e76b807bdac40783a7ee3c
NATIVE_ARCHIVE=protos-0.3.113-native-linux-x86_64.zip
NATIVE_ARCHIVE_SHA256=0f7c85d56e7e5c495db603badef5a5f4e53b5dde84fd44497f7f20b3d3c7c089
NATIVE_DIST_ARCHIVE_CRC_CHECK=PASS
NATIVE_DIST_LAYOUT_CHECK=PASS
NATIVE_DIST_NO_JVM_PLANE_CHECK=PASS
NATIVE_DIST_INTERNAL_CHECKSUM_CHECK=PASS
PUBLIC_NATIVE_SOURCE_IDENTITY=PASS
PUBLIC_NATIVE_EMPIRICAL_METADATA=PASS
```

## Complete Native admission

The exact archived Native candidate passed the complete D4 admission:

```text
NATIVE_DIST_GLIBC_ABI_FLOOR_PROOF=PASS
NATIVE_DIST_DYNAMIC_LIBRARY_CLOSURE_CHECK=PASS
NATIVE_DIST_EMPIRICAL_ELF_ADMISSION=PASS
NATIVE_DIST_RELOCATED_ROOT_CHECK=PASS
NATIVE_DIST_NO_JAVA_PATH_CHECK=PASS
NATIVE_DIST_NO_MAVEN_PATH_CHECK=PASS
NATIVE_DIST_VERSION_CHECK=PASS
NATIVE_DIST_HELP_CHECK=PASS
NATIVE_DIST_EVAL_CHECK=PASS
NATIVE_DIST_EXTERNAL_SOURCE_CHECK=PASS
NATIVE_DIST_UNRELATED_CWD_CHECK=PASS
NATIVE_DIST_PACKAGE_TOOL_ADMISSION_CHECK=PASS
NATIVE_DIST_WORKSPACE_RUN_ADMISSION_CHECK=PASS
NATIVE_DIST_TEST_TOOL_FOCAL_ADMISSION_CHECK=PASS
NATIVE_DIST_REPL_PTY_ADMISSION_CHECK=PASS
NATIVE_DIST_LSP_STDIO_ADMISSION_CHECK=PASS
NATIVE_DIST_DAP_ADMISSION_CHECK=PASS
NATIVE_DIST_FORCED_GUEST_JIT_CHECK=PASS
NATIVE_DIST_HELPER_BYTECODE_ROOT_TIER2_CHECK=PASS
NATIVE_DIST_SEMANTIC_BYTECODE_ROOT_TIER2_CHECK=PASS
```

Full Native Test Tool result:

```text
NATIVE_DIST_FULL_TEST_TOOL_JOBS=16
NATIVE_DIST_FULL_TEST_TOOL_STATUS=0
NATIVE_DIST_FULL_TEST_TOOL_SECONDS=81.23
NATIVE_DIST_FULL_TEST_TOOL_CONTEXT_TEARDOWN_FAILURES=0
NATIVE_DIST_FULL_TEST_TOOL_REFLECTION_FAILURES=0
NATIVE_DIST_FULL_TEST_TOOL_PASSED=1263
NATIVE_DIST_FULL_TEST_TOOL_FAILED=0
DIST005_NATIVE_FULL_TEST_TOOL_ADMISSION=PASS
```

Final Native integrity/admission:

```text
NATIVE_DIST_OUTER_CHECKSUM_CHECK=PASS
NATIVE_DIST_ARCHIVE_UNCHANGED_CHECK=PASS
DIST005_NATIVE_BASIC_ADMISSION=PASS
DIST005_NATIVE_A_G_ADMISSION=PASS
DIST005_NATIVE_EXTRACTED_JIT_ADMISSION=PASS
DIST005_NATIVE_COMPLETE_ADMISSION=PASS
DIST005_D4_NATIVE_ADMISSION_STATUS=0
```

The candidate worktree remained clean after admission.

## Portable JVM artifact from the same candidate

The retained compatibility/fallback artifact was built from exactly the same
candidate revision:

```text
PORTABLE_JVM_ARCHIVE=protos-0.3.113-posix-jvm.zip
PORTABLE_JVM_SHA256=6827dc267bb82f09ba3a0989da719fced2b5bd8dae76e8f7c7acb74ba69f354c
PORTABLE_SOURCE_REVISION=4812007bcf48533b89b14a5c6c0a7e8b24b1a992
PORTABLE_RELEASE_VERSION=0.3.113
PORTABLE_RELEASE_TAG=v0.3.113
PORTABLE_SOURCE_IDENTITY=PASS
```

Portable admission reported:

```text
DIST001_B2_VERIFY=PASS
DIST001_B3_SMOKE=PASS
TEST001_H_PORTABLE_REPOSITORY_SUITE_CHECK=PASS passed=1263 failed=0
DIST001_B4A_SMOKE=PASS
DIST001_B4B_SMOKE=PASS
DIST001_B5_CROSS_SLICE=PASS
PORTABLE_JVM_ADMISSION=PASS
```

The candidate worktree remained clean after the portable build/admission.

## Exact multi-asset release envelope

Both artifacts were composed into one deterministic public-prerelease envelope:

```text
release_envelope_format=protos-public-prerelease-envelope-v2
distribution_model=JVM_PLUS_NATIVE
release_version=0.3.113
release_tag=v0.3.113
source_revision=4812007bcf48533b89b14a5c6c0a7e8b24b1a992
release_baseline_revision=1e64469eb781c2ae5a15087d80b0cd4ad0e74214
release_baseline_version=0.3.113-SNAPSHOT
specification_revision=0.1.434
asset_count=2
```

Selected roles:

```text
native-linux-x86_64-glibc-2.39-dynamic-compatibility=recommended-first-run
portable-jvm=compatibility-fallback
```

Envelope verification reported:

```text
MULTI_ASSET_ENVELOPE_VERIFICATION=PASS
PER_ARTIFACT_KIND_IDENTITY=PASS
PER_ARTIFACT_ROLE_IDENTITY=PASS
PER_ARTIFACT_PLATFORM_IDENTITY=PASS
PER_ARTIFACT_RUNTIME_IDENTITY=PASS
PER_ARTIFACT_CHECKSUM_IDENTITY=PASS
PER_ARTIFACT_LICENSE_NOTICE_IDENTITY=PASS
COMMON_CANDIDATE_IDENTITY=PASS
UNAMBIGUOUS_RECOMMENDED_FIRST_RUN=PASS
DETERMINISTIC_ASSET_ORDERING=PASS
DIST005_D1_MULTI_ASSET_ENVELOPE=PASS
MULTI_ASSET_ENVELOPE_VERIFY_STATUS=0
```

Final candidate identity remained:

```text
CANDIDATE_FINAL_STATUS=CLEAN
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK
```

## Publication boundary

D4 is a candidate/distribution proof, not a public release.

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
PUBLIC_NATIVE_RELEASE_ARTIFACT_CREATED=NO
```

The implementation commit is present on `guillermomolina/protos/main`.
A final `scripts/publication_validation.py` result for the D4 implementation
range was not supplied in the active session, so this record deliberately does
not claim `PUBLICATION_VALIDATION=PASS`.

That omission does not alter the empirical candidate/distribution results above;
it remains explicit validation evidence not claimed by this record.

## Result

```text
DIST005_D4_STATUS=PASS

IMPLEMENTATION_BASE_REVISION=96f066c3689359afc0f03d6760de171bdfe2311f
IMPLEMENTATION_REVISION=1e64469eb781c2ae5a15087d80b0cd4ad0e74214
IMPLEMENTATION_VERSION=0.3.113-SNAPSHOT

PROOF_CANDIDATE_SOURCE_REVISION=4812007bcf48533b89b14a5c6c0a7e8b24b1a992
PROOF_CANDIDATE_VERSION=0.3.113
PROOF_CANDIDATE_TAG_IDENTITY=v0.3.113

PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
POST_LINK_CPU_ISA_EVIDENCE=x86-64-baseline,x86-64-v2,x86-64-v3
PUBLIC_NATIVE_GLIBC_MIN=2.39
OBSERVED_GLIBC_MAX=2.34
GLIBC_ABI_FLOOR_PROOF=PASS

NATIVE_CLEAN_ARCHIVE_BUILD=PASS
NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_FULL_TEST_TOOL=1263/0
NATIVE_TRUFFLE_RUNTIME_COMPILATION=PASS
NATIVE_ARCHIVE_INTEGRITY=PASS

PORTABLE_JVM_CANDIDATE_BUILD=PASS
PORTABLE_JVM_ADMISSION=PASS

MULTI_ASSET_ENVELOPE_PREPARE=PASS
MULTI_ASSET_ENVELOPE_VERIFY=PASS
COMMON_CANDIDATE_IDENTITY=PASS

RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK

PUBLICATION_VALIDATION=NOT_REPORTED_IN_ACTIVE_SESSION
RELEASE_PUBLICATION_AUTHORIZED=NO
PUBLIC_RELEASE_CREATED=NO
TAG_CREATED=NO
```

## Next bounded slice

The D4 proof demonstrates that the required machinery works, but the human
procedure is still unnecessarily fragmented. The next bounded implementation
slice should compose the already-proven mechanisms into one deterministic,
fail-closed release-candidate preparation workflow while preserving a separate
explicit publication authorization boundary.

```text
NEXT_SLICE=DIST005-D5
NEXT_SLICE_TITLE=deterministic release-candidate orchestration
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

D5 must reuse the existing DIST001/DIST005 identity and validation mechanisms;
it must not invent a second release identity, weaken the exact-candidate
invariant, automatically publish a Git tag/GitHub Release, or broaden the
selected Native support matrix.
