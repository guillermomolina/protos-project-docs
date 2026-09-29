# DIST005-D1 — multi-asset release-envelope foundation

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Published Protos revision:
`80fb2650399f7edce35599c2399f6e6019660468`

## Scope

DIST005-D1 implements the bounded release-metadata foundation required by the
owner-approved `JVM_PLUS_NATIVE` artifact model.

It extends the existing DIST001 public-prerelease envelope so one exact release
candidate can describe and independently verify multiple release assets with
explicit per-artifact identity.

This slice does not build or publish a public Native artifact, select a public
Native CPU-ISA promise, select a Native platform support matrix, materialize a
release candidate, create a tag/GitHub Release, remove the portable JVM artifact,
or change observable Protos semantics.

## Publication result

The implementation was published to `guillermomolina/protos` as:

```text
80fb2650399f7edce35599c2399f6e6019660468
DIST005-D1: add multi-asset release envelope foundation
```

Published implementation version:

```text
0.3.111-SNAPSHOT
```

Changed product surfaces:

```text
CHANGELOG.md
pom.xml
dist/prepare_release_metadata.py
dist/verify_release_metadata.py
dist/release_asset_envelope.py
dist/test_multi_asset_release_metadata.py
```

No normative specification file was changed.

## Selected artifact relationship retained

DIST005-C remains authoritative:

```text
DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ARTIFACT=RETAIN
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
DIST001_RELEASE_IDENTITY_MODEL=RETAIN
SECOND_RELEASE_IDENTITY_MODEL=NO
```

D1 encodes that relationship without creating a second release identity.

## Multi-asset envelope format

The new multi-asset format is:

```text
release_envelope_format=protos-public-prerelease-envelope-v2
distribution_model=JVM_PLUS_NATIVE
```

The existing single portable-JVM path remains intact. A single `--archive`
with no per-asset role continues through the legacy envelope implementation.
Multiple `--archive` arguments enter the multi-asset path.

The selected model requires:

```text
exactly one portable-jvm artifact
at least one native artifact
exactly one recommended-first-run artifact
recommended-first-run artifact kind=native
portable-jvm role=compatibility-fallback
```

## Per-artifact identity

Each release asset is independently represented with stable identity covering:

```text
artifact key
artifact kind
artifact role
platform identity
distribution format
archive filename
archive SHA-256
adjacent checksum identity
license path and SHA-256
notice path and SHA-256
runtime-specific identity
```

For the portable JVM artifact, runtime identity includes the supported Java /
GraalVM / Truffle runtime contract already owned by the legacy release
machinery.

For Native artifacts, runtime/platform identity includes:

```text
native_runtime_kind
graalvm_release
native_image_version
jdk_version
target_os
target_arch
libc_family
linkage
cpu_isa_assumption
external_java_required=false
```

Native artifact keys include the platform/ISA identity so independently
published Native targets cannot collapse onto one ambiguous artifact identity.

## One-candidate invariant

All assets in one multi-asset envelope must agree on the same exact release
candidate identity:

```text
release_version
release_tag
source_repository
source_revision
release_baseline_revision
release_baseline_version
```

A mismatch in any of those fields fails closed.

The result preserves the selected release lineage:

```text
one selected V-SNAPSHOT baseline
  -> one detached release-only V candidate
  -> one or more assets from that exact candidate
  -> one tag vV
  -> one GitHub prerelease
```

## Determinism and fail-closed behavior

Assets are canonically ordered by stable artifact key.

The implementation rejects at least:

```text
mixed candidate/source identity
duplicate artifact identity
missing Native platform/runtime identity
unresolved public Native CPU ISA
ambiguous recommended-first-run role
unknown public artifact kind
independent checksum tampering
missing license/notice identity
```

Synthetic Native test fixtures use a deliberately synthetic ISA token and do
not constitute a Protos public CPU support promise.

## Compatibility boundary

The legacy portable-JVM preparation and verification paths remain available and
were explicitly regression-tested.

D1 does not require Native assets for the existing single-JVM DIST001/DIST007
workflow. The portable JVM artifact therefore remains usable as the selected
compatibility/fallback and external/development-consumer artifact while the
Native public release path is completed independently.

## Validation evidence

The human-executed validation reported:

```text
git diff --check=PASS
python syntax compilation=PASS

legacy prepare tests=5/5 PASS
legacy verify tests=6/6 PASS
multi-asset envelope tests=8/8 PASS

MULTI_ASSET_ENVELOPE_GENERATION=PASS
MULTI_ASSET_ENVELOPE_VERIFICATION=PASS
COMMON_CANDIDATE_IDENTITY=PASS
DETERMINISTIC_ASSET_ORDERING=PASS
UNAMBIGUOUS_RECOMMENDED_FIRST_RUN=PASS
PER_ARTIFACT_KIND_IDENTITY=PASS
PER_ARTIFACT_ROLE_IDENTITY=PASS
PER_ARTIFACT_PLATFORM_IDENTITY=PASS
PER_ARTIFACT_RUNTIME_IDENTITY=PASS
PER_ARTIFACT_CHECKSUM_IDENTITY=PASS
PER_ARTIFACT_LICENSE_NOTICE_IDENTITY=PASS
DIST005_D1_MULTI_ASSET_ENVELOPE=PASS

make test=PASS
```

The implementation was then committed and pushed as the exact revision recorded
above.

## Remaining DIST005 boundaries

D1 intentionally leaves the following unresolved:

```text
PUBLIC_NATIVE_CPU_ISA_POLICY=UNSELECTED
PUBLIC_NATIVE_PLATFORM_MATRIX=UNSELECTED
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO
PUBLIC_NATIVE_RELEASE_ARTIFACT_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
```

The existing DIST005-B proof remains development-only evidence for
Linux/x86_64/glibc with dynamic linkage. It is not a public support matrix or a
clean release-candidate proof.

## Next bounded work

The next slice is:

```text
DIST005-D2 — Native public platform and CPU-ISA selection packet
WORK_TYPE=INVESTIGATION
COMMAND_EXECUTION=NONE
```

D2 must determine the exact public Native target promise before release
implementation can build a clean public candidate. It should compare at least
the proven Linux/x86_64/glibc target, the CPU compatibility consequences of
Native Image's AMD64 machine-type policy, and whether any additional target can
be advertised from evidence already available or instead requires a later
implementation/proof slice.

D2 must stop with an exact owner-selection packet. It must not silently select a
public CPU-ISA promise, broaden the Native support matrix, modify product
repositories, run project commands, build artifacts, or publish a release.

## Result

```text
DIST005_D1_STATUS=PASS
PROTOS_REVISION=80fb2650399f7edce35599c2399f6e6019660468
PROTOS_VERSION=0.3.111-SNAPSHOT

DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
MULTI_ASSET_RELEASE_ENVELOPE_FOUNDATION=PASS
LEGACY_SINGLE_JVM_ENVELOPE_RETAINED=YES
COMMON_RELEASE_CANDIDATE_IDENTITY_REQUIRED=YES
DETERMINISTIC_ASSET_ORDERING=PASS
PUBLIC_NATIVE_CPU_ISA_FAIL_CLOSED=YES

PUBLIC_NATIVE_CPU_ISA_POLICY=UNSELECTED
PUBLIC_NATIVE_PLATFORM_MATRIX=UNSELECTED
NATIVE_CLEAN_RELEASE_CANDIDATE_PROVEN=NO

NEXT_SLICE=DIST005-D2
NEXT_SLICE_WORK_TYPE=INVESTIGATION
NEXT_SLICE_COMMAND_EXECUTION=NONE
```
