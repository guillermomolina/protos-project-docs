# DIST005-C — final self-contained artifact-model selection

Status: OWNER APPROVED
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-C — final self-contained artifact-model selection packet`

## Selection identity

The read-only DIST005-C investigation inspected:

```text
PROTOS_REVISION=a6aa7f1177f99c77e5148b8363a77d7250184793
PROTOS_VERSION=0.3.110-SNAPSHOT
PROJECT_RECORD_BASE_REVISION=d7dc450fcefcc3fa874e3b228fe67d03a942ddd9
```

The project owner explicitly approved the recommended model on 2026-09-29:

```text
Aprobado B — JVM_PLUS_NATIVE
```

Selected product model:

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE

RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT

NATIVE_FIRST_RUN_REQUIRES_EXTERNAL_JAVA=NO
PORTABLE_JVM_REQUIRES_SUPPORTED_EXTERNAL_GRAALVM_JDK=YES

DIST001_RELEASE_IDENTITY_MODEL=RETAIN
SECOND_RELEASE_IDENTITY_MODEL=NO
MULTI_ASSET_RELEASE_ENVELOPE_REQUIRED=YES

JVM_DEVELOPMENT_PATH_RETAINED=YES
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
EXTERNAL_PROTOS_RESOURCE_TREE_PRESERVED=YES

DIST007_BLOCKED_BY_DIST005=NO
DIST007_PORTABLE_JVM_ROLE_RETAINED=YES

NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

This approval selects the artifact relationship and recommended first-run role.
It does not authorize a tag, GitHub Release, exact public Native platform matrix,
CPU-ISA compatibility promise, or exact release candidate.

## Evidence basis

DIST005-B proved a complete self-contained Native development distribution on
one canonical target:

```text
target_os=linux
target_arch=x86_64
linkage=dynamic
libc_family=glibc

NATIVE_RELOCATABLE_ARCHIVE_PROOF=PASS
NATIVE_EXTERNAL_JAVA_REQUIRED=NO
NATIVE_EXTERNAL_MAVEN_REQUIRED=NO
NATIVE_EXTERNAL_PROTOS_CHECKOUT_REQUIRED=NO

NATIVE_RUN_SURFACE=PASS
NATIVE_PACKAGE_SURFACE=PASS
NATIVE_TEST_SURFACE=PASS
NATIVE_REPL_SURFACE=PASS
NATIVE_LSP_SURFACE=PASS
NATIVE_DAP_SURFACE=PASS
NATIVE_OPTIMIZING_TRUFFLE_SURFACE=PASS

NATIVE_FULL_TEST_TOOL=1263/1263_PASS
DIST005_NATIVE_COMPLETE_ADMISSION=PASS
```

Exact development-proof artifact:

```text
native_binary_sha256=c46aa09210641e81e0f357f9c25f188acd1dfa669750c185f0b028266dca516e
archive=protos-0.3.110-SNAPSHOT-native-linux-x86_64.zip
archive_sha256=3117fb827149a422b7dd9305f3334193a305d7503408dcef43d06e74d05e03a6
```

That archive is not a clean release candidate because its intentionally
pre-commit metadata records:

```text
source_revision=754de7a2a2d73dd4b39109bb842522ed9cc8153a
source_dirty=true
```

The exact implementation was subsequently published at
`a6aa7f1177f99c77e5148b8363a77d7250184793`.

Therefore:

```text
DOES_CURRENT_NATIVE_PROOF_SATISFY_STRICT_FIRST_RUN_REQUIREMENT=YES
NATIVE_CLOSED_WORLD_BLOCKER_REMAINING=NO
NATIVE_PUBLIC_PLATFORM_MATRIX_READY=NO
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
```

## Candidate resolution

### A — JVM_ONLY

A self-contained JVM-only artifact is technically possible by bundling the full
supported GraalVM/JDK runtime with Protos. A reduced runtime remains conditional
on a Protos-specific proof of the exact Truffle/Graal/JVMCI/service-provider
closure.

The model was not selected because it would add a new self-contained JVM
distribution implementation while discarding the now-proven Native first-run
path and would remove the value of the already-established external-runtime
portable JVM artifact.

### B — JVM_PLUS_NATIVE — SELECTED

The selected role split is:

```text
Native
  = unambiguous recommended first-run path
  = one archive
  = no external Java/GraalVM/Maven
  = ordinary CLI/tooling surface on explicitly supported Native targets

Portable JVM
  = retained compatibility/fallback artifact
  = retained external/development-consumer artifact
  = continues to require the explicitly supported external GraalVM/JDK
  = not the zero-dependency recommended first-run path
```

Both artifacts belong to one selected release identity and one tag/GitHub
prerelease. The release envelope must become multi-asset rather than creating a
second release model.

### C — NATIVE_ONLY

Native-only is technically possible for the proven target while retaining the
JVM development path required by PLAT038, but it was not selected. Removing the
portable JVM asset would make the complete public compatibility surface depend
immediately on the still-unclosed Native platform matrix.

## Release-envelope consequence

Current release metadata is mechanically JVM/single-archive-centric. The
selected model requires a bounded extension with per-artifact identity at least
for:

```text
artifact kind
platform
runtime identity
source/candidate identity
checksum
license/notice identity
```

The existing DIST001 detached-candidate/release identity remains authoritative:

```text
selected V-SNAPSHOT baseline
  -> detached release-only V candidate
  -> one or more validated artifacts from that exact candidate
  -> one tag vV
  -> one GitHub prerelease
```

No selected implementation may create independent JVM and Native release
identities.

## Native platform and CPU-ISA boundary

DIST005-B reported an `x86-64-v3` Native Image target machine while the
development archive deliberately retained:

```text
cpu_isa_assumption=unresolved
```

Current GraalVM JDK 25 Native Image documentation states that `-march`
defaults to `x86-64-v3` on AMD64 and that `-march=compatibility` is available
for broader CPU compatibility.

Upstream reference:
https://www.graalvm.org/jdk25/reference-manual/native-image/overview/Options/

This investigation does not silently convert that fact into a Protos support
promise.

```text
PUBLIC_NATIVE_CPU_ISA_POLICY=UNSELECTED
PUBLIC_NATIVE_PLATFORM_MATRIX=UNSELECTED
```

The implementation must carry exact target/ISA identity in release metadata and
fail closed when the public claim is not explicitly established.

## Licensing/update boundary

GraalVM Community Edition is distributed under GPLv2 with the Classpath
Exception; individual components may carry additional licensing requirements.

Upstream reference:
https://www.graalvm.org/faq/

The selected multi-asset model therefore requires artifact-specific
license/notice identity and final public-release compliance review. This
selection does not claim that DIST005-B's development notices alone are
sufficient for public release.

## Measurements not established by DIST005-C

No measurement was invented for criteria not retained by current evidence:

```text
NATIVE_ARCHIVE_SIZE=UNKNOWN
SELF_CONTAINED_JVM_ARCHIVE_SIZE=UNKNOWN
NATIVE_VS_JVM_COLD_START=REQUIRES_MEASUREMENT
NATIVE_VS_JVM_WARMED_PERFORMANCE=REQUIRES_MEASUREMENT
CROSS_PLATFORM_NATIVE_BEHAVIOR=UNPROVEN_OUTSIDE_CURRENT_TARGET
```

These unknowns do not prevent selection of the artifact relationship. They
remain release/support evidence obligations where applicable.

## Authority check

PLAT038 already ratifies Native Image as a first-class artifact, retention of
the JVM development path, required Native Truffle runtime compilation, and the
external Protos resource-tree model.

PLAT039 already ratifies the PE-visible guest-kernel / narrow-host-gateway
runtime-compilation boundary.

DIST005 owns the end-user self-contained distribution relationship.

Therefore:

```text
NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
DIST005_OWNER_SELECTION_AUTHORITY=SUFFICIENT
```

## Immediate implementation decomposition

The selected model is implemented in bounded slices rather than one
release-sized patch.

The next slice is:

```text
DIST005-D1 — multi-asset release-envelope foundation
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
```

D1 should extend the existing DIST001 release-envelope machinery so one release
candidate can describe and validate multiple artifacts with explicit
per-artifact identity while retaining compatibility with the existing portable
JVM release path.

D1 must not:

- materialize or publish a release candidate;
- create a tag or GitHub Release;
- select a public Native CPU-ISA promise;
- advertise a multi-platform Native matrix;
- rebuild or re-prove the complete DIST005-B Native artifact merely to test the
  metadata schema;
- remove the portable JVM artifact;
- create a second release identity;
- modify observable Protos semantics; or
- depend on another repository or web research for implementation.

After D1 is published, later bounded work can own clean Native candidate/platform
closure, measurements/public claims, release documentation, and eventual
publication authorization.

## Selection result

```text
DIST005_C_STATUS=OWNER_APPROVED
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
FINAL_DIST005_ARTIFACT_MODEL_SELECTED=YES

RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
MULTI_ASSET_RELEASE_EXTENSION_REQUIRED=YES

NATIVE_PUBLIC_PLATFORM_MATRIX_READY=NO
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
PUBLIC_NATIVE_CPU_ISA_POLICY=UNSELECTED

NEW_DXXX_REQUIRED=NO
NEW_PLATXXX_REQUIRED=NO
DIST007_BLOCKED_BY_DIST005=NO

NEXT_SLICE=DIST005-D1
NEXT_SLICE_WORK_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
