# DIST011-B — exact candidate build and validation checkpoint

Status: **PASS / EXACT CANDIDATE FROZEN / PUBLICATION NOT AUTHORIZED**

This durable, non-normative record retains the completed DIST011-B exact-candidate
build and validation evidence for `guillermomolina/protos#774`.

## Ownership

```text
DATE=2026-10-02
WORK_ITEM=DIST011-B
GITHUB_ISSUE=guillermomolina/protos#774
TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

DIST011-B consumed the exact baseline selected and durably retained by DIST011-A.

## Selected baseline

```text
RELEASE_BASELINE_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
RELEASE_BASELINE_VERSION=0.3.143-SNAPSHOT
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143
SPECIFICATION_REVISION=0.1.436
DIST011_A_PROJECT_RECORD_REVISION=a56d62e670b32f51475d7b294564fdbca7eaa70c
DIST011_A_SELECTION_RECORD=docs/project/evidence/DIST011/DIST011_A_SELECTION.txt
```

## Exact frozen candidate

The maintained release machinery materialized the release-only candidate:

```text
CANDIDATE_SOURCE_REVISION=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
CANDIDATE_PARENT=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
CANDIDATE_DETACHED=YES
CANDIDATE_FINAL_STATUS=CLEAN
RELEASE_ONLY_DIFF_PATHS=pom.xml
VERSION_TRANSITION=0.3.143-SNAPSHOT->0.3.143
```

The development checkout remained unchanged:

```text
MAIN_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
MAIN_VERSION=0.3.143-SNAPSHOT
MAIN_REMAINS_SNAPSHOT=YES
```

## Native artifact and admission

The exact Native public-prerelease artifact is:

```text
NATIVE_ARCHIVE=protos-0.3.143-native-linux-x86_64.zip
NATIVE_SHA256=b8970146ba82d574635d4409b186021f7d2f6934fc8c05ed0ee4ae5f4b5b8534
NATIVE_CHECKSUM_SHA256=1e6005a086685e39e1dc983aee48a124071bfa7176cd7b6b1e22aad5d52e64ee
```

Observed maintained Native admission result:

```text
NATIVE_BUILD=PASS
NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_INTERPRETER_ONLY_ADMISSION=PASS
NATIVE_FULL_TEST_TOOL_ADMISSION=PASS
NATIVE_FULL_TEST_TOOL_PASSED=1284
NATIVE_FULL_TEST_TOOL_FAILED=0

NATIVE_IMAGE=SUPPORTED
NATIVE_EXECUTION=INTERPRETER_ONLY
NATIVE_GUEST_JIT=UNSUPPORTED_UPSTREAM_ORACLE_GRAAL_14579

TARGET_OS=linux
TARGET_ARCH=x86_64
LINKAGE=dynamic
LIBC_FAMILY=glibc
LIBC_ABI_MIN=2.39
GLIBC_OBSERVED_MAX=2.34
CPU_ISA_ASSUMPTION=compatibility
NATIVE_BUILD_MARCH=-march=compatibility
```

The exact candidate therefore satisfies the ratified PLAT045 Native capability
boundary without changing that policy.

## Portable JVM artifact and admission

The exact portable JVM public-prerelease artifact is:

```text
PORTABLE_JVM_ARCHIVE=protos-0.3.143-posix-jvm.zip
PORTABLE_JVM_SHA256=3c2fab126e5f83f8e16b7e7284ccc9a269e5120f109e8f11affec51a3e3baa93
PORTABLE_JVM_CHECKSUM_SHA256=5b18ee683493660a073260b13737cbd00677a27d891063d9ee83886403ca03f4
```

Observed maintained portable-JVM admission result:

```text
PORTABLE_JVM_BUILD=PASS
PORTABLE_JVM_ADMISSION=PASS
PORTABLE_REPOSITORY_SUITE_PASSED=1284
PORTABLE_REPOSITORY_SUITE_FAILED=0
JVM_SELECTED_JDK=25.0.4.1.1
JVM_TRUFFLE_RUNTIME=25.4.4.1.1
JVM_OPTIMIZING_RUNTIME=HotSpotTruffleRuntime
```

## Multi-asset envelope

The maintained JVM_PLUS_NATIVE envelope passed all current checks:

```text
DISTRIBUTION_MODEL=JVM_PLUS_NATIVE
MULTI_ASSET_ENVELOPE_GENERATION=PASS
MULTI_ASSET_ENVELOPE_VERIFICATION=PASS
COMMON_CANDIDATE_IDENTITY=PASS
PER_ARTIFACT_KIND_IDENTITY=PASS
PER_ARTIFACT_ROLE_IDENTITY=PASS
PER_ARTIFACT_PLATFORM_IDENTITY=PASS
PER_ARTIFACT_RUNTIME_IDENTITY=PASS
PER_ARTIFACT_CHECKSUM_IDENTITY=PASS
PER_ARTIFACT_LICENSE_NOTICE_IDENTITY=PASS
UNAMBIGUOUS_RECOMMENDED_FIRST_RUN=PASS
DETERMINISTIC_ASSET_ORDERING=PASS

NATIVE_ROLE=recommended-first-run
PORTABLE_JVM_ROLE=compatibility-fallback
```

Frozen envelope identities:

```text
RELEASE_NOTES_SHA256=2336538d61f72da67c804109e95fb4e5b0a06827aa37ac6cbb26fd03b4ef5146
RELEASE_MANIFEST_SHA256=c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591
```

The release manifest binds:

```text
release_envelope_format=protos-public-prerelease-envelope-v2
distribution_model=JVM_PLUS_NATIVE
release_version=0.3.143
release_tag=v0.3.143
source_revision=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
release_baseline_revision=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
release_baseline_version=0.3.143-SNAPSHOT
specification_revision=0.1.436
asset_count=2
```

## Collision/publication state

The human executor verified the selected public identity remains unused:

```text
LOCAL_TAG_v0.3.143=ABSENT
REMOTE_TAG_v0.3.143=ABSENT
GITHUB_RELEASE_v0.3.143=ABSENT
TAG_COLLISION=NO
GITHUB_RELEASE_COLLISION=NO
```

GitHub's release-by-tag lookup returned HTTP 404 for `v0.3.143`.

No publication occurred during DIST011-B:

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
ASSETS_PUBLISHED=NO
```

## Consumer-specific gate

DIST011-A established that no current consumer-specific prepublication gate is
required. DIST011-B therefore used the canonical Protos-owned release gates only.

```text
CONSUMER_SPECIFIC_PREPUBLICATION_GATE_REQUIRED=NO
CONSUMER_SPECIFIC_GATE=NOT_REQUIRED
```

## DIST011-B final classification

```text
DIST011_B_STATUS=PASS

EXACT_BASELINE_SELECTED=YES
EXACT_CANDIDATE_FROZEN=YES

CANDIDATE_SOURCE_REVISION=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143
SPECIFICATION_REVISION=0.1.436

NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_INTERPRETER_ONLY=PASS
PORTABLE_JVM_ADMISSION=PASS
MULTI_ASSET_ENVELOPE=PASS
LICENSE_NOTICES=PASS
CHECKSUMS=PASS
CANDIDATE_IDENTITY=PASS
CONSUMER_SPECIFIC_GATE=NOT_REQUIRED

RELEASE_MANIFEST_SHA256=c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591

MAIN_REMAINS_SNAPSHOT=YES
PUBLICATION_AUTHORIZATION=NOT_YET_GRANTED

TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
ASSETS_PUBLISHED=NO
```

## Next gate

DIST011-C may proceed only after explicit project-owner authorization of this
exact immutable publication identity:

```text
candidate_source_revision=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
release_version=0.3.143
release_tag=v0.3.143
release_manifest_sha256=c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591
github_release_prerelease=true
github_release_draft=false
```

Until that exact approval exists, DIST011-C MUST NOT create or push the tag,
create the GitHub Release, or publish assets.
