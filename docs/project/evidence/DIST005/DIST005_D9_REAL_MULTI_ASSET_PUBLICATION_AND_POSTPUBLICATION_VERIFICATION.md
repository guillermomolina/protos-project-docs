# DIST005-D9 — real multi-asset publication and post-publication verification

Status: PUBLISHED AND VERIFIED  
Date: 2026-09-29  
Owning Issue: `guillermomolina/protos#549`  
Slice: `DIST005-D9 — explicit publication authorization and real multi-asset publication`  
Work type: IMPLEMENTATION / RELEASE PUBLICATION  
Implementation repository: `guillermomolina/protos`

## Boundary

DIST005-D9 published the exact owner-authorized `JVM_PLUS_NATIVE` Protos
prerelease selected and prepared by DIST005-D7/D8.

D9 did not rebuild either artifact, select another baseline, change the release
notes, alter checksums, broaden platform support, update user-facing
documentation, or close the parent DIST005 work item.

The public release is:

```text
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
RELEASE_TITLE=Protos 0.3.116
GITHUB_RELEASE_PRERELEASE=true
GITHUB_RELEASE_DRAFT=false
PUBLIC_RELEASE_URL=https://github.com/guillermomolina/protos/releases/tag/v0.3.116
```

## Exact owner authorization

The project owner explicitly authorized publication in the active D9
interaction for exactly:

```text
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
RELEASE_MANIFEST_SHA256=7704d1a10072c1d906ae529b1824b40938dbf0a827f0645904ed9a1a149c52f1
GITHUB_RELEASE_PRERELEASE=true
GITHUB_RELEASE_DRAFT=false
```

The ephemeral authorization record used the existing exact format:

```text
publication_authorization_format=protos-dist005-publication-authorization-v1
release_publication_authorized=true
authorization_basis=explicit-user-decision
candidate_source_revision=0336ae20216bf2eec17854bea0f6435e4e1e9b19
release_version=0.3.116
release_tag=v0.3.116
release_manifest_sha256=7704d1a10072c1d906ae529b1824b40938dbf0a827f0645904ed9a1a149c52f1
github_release_prerelease=true
github_release_draft=false
```

The authorization record remained local/ephemeral and was not committed to
`main`.

## Exact selected release identity

```text
RELEASE_BASELINE_REVISION=1637fe514ee9327241e49964031d981423b16198
RELEASE_BASELINE_VERSION=0.3.116-SNAPSHOT
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
SPECIFICATION_REVISION=0.1.434

DIST005_SELECTED_MODEL=JVM_PLUS_NATIVE
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
```

The public tag resolves exactly to the candidate revision above. The later
publication-tooling repair on `main` is not part of the released source
identity.

## Publication-tooling repair during D9

The first publication attempt failed before public mutation because
`dist/publish_release.py` used:

```text
git show-ref --verify --hash refs/tags/v0.3.116
```

as the absence probe and assumed a missing ref returns status 1. In the active
Git environment, the missing exact ref returned status 128 with:

```text
fatal: 'refs/tags/v0.3.116' - not a valid ref
```

The bounded repair changed local-tag absence detection to probe with
`git show-ref --verify --quiet` before requesting the hash, and added a real
missing-local-tag regression test.

Published repair:

```text
PUBLICATION_TOOLING_REVISION=7e73b334b6f2111c2aafc561cce531b3b806ced1
PUBLICATION_TOOLING_COMMIT=DIST005: handle absent local release tag
FOCAL_REGRESSION_TESTS=4/4_PASS
PUBLISH_RELEASE_TEST_SUITE=PASS
```

This repair did not change the authorized candidate/version/tag/manifest tuple
or release semantics.

A later retry also stopped before public mutation because GitHub CLI was not
authenticated in the publication container. After authenticating the existing
`gh` client, the same authorized publication was rerun. No candidate or
release metadata was regenerated.

## Canonical publication result

D9 used only the existing authoritative publication entry point:

```text
dist/publish_release.py
```

Successful canonical output:

```text
DIST005_RELEASE_PUBLICATION=PASS
RELEASE_CANDIDATE_SOURCE_REVISION=0336ae20216bf2eec17854bea0f6435e4e1e9b19
RELEASE_VERSION=0.3.116
RELEASE_TAG=v0.3.116
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
RELEASE_PUBLICATION_AUTHORIZED=YES
TAG_CREATED_OR_REUSED=YES
PUBLIC_RELEASE_CREATED_OR_REUSED=YES
```

No partial public-state recovery path was exercised:

```text
PUBLIC_PARTIAL_RECOVERY_PATH=NO
```

Both pre-success failures occurred before any public mutation.

## Public asset set and identities

The GitHub prerelease exposes exactly five downloadable assets:

```text
1. protos-0.3.116-native-linux-x86_64.zip
2. protos-0.3.116-native-linux-x86_64.zip.sha256
3. protos-0.3.116-posix-jvm.zip
4. protos-0.3.116-posix-jvm.zip.sha256
5. RELEASE_MANIFEST.txt
```

`RELEASE_NOTES.md` is the GitHub Release body and is not a sixth asset.

Verified public identities:

```text
NATIVE_ARCHIVE_SHA256=61fd90b39a43c574900b3c61d4fe2e65e100166ced66e495281b336f934883e8
PORTABLE_JVM_ARCHIVE_SHA256=ac5848ce3e4f371e7424ca5628899cc53a4d0be23d09381ca48db68218cd2add
RELEASE_MANIFEST_SHA256=7704d1a10072c1d906ae529b1824b40938dbf0a827f0645904ed9a1a149c52f1
RELEASE_NOTES_SHA256=a888884fedc373547bd15cee29a4625cd792327d7b5113fd9e03242b153f8b12
```

The adjacent public checksum files were downloaded and successfully verified
with `sha256sum -c`.

## Independent post-publication verification

Independent public-state verification established:

```text
PUBLIC_TAG_SHA=0336ae20216bf2eec17854bea0f6435e4e1e9b19
PUBLIC_TAG_IDENTITY=PASS

RELEASE_TITLE=Protos 0.3.116
RELEASE_TAG=v0.3.116
PUBLIC_RELEASE_PRERELEASE=true
PUBLIC_RELEASE_DRAFT=false
PUBLIC_ASSET_COUNT=5
GITHUB_PRERELEASE_METADATA=PASS

RELEASE_BODY_IDENTITY=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
CANDIDATE_FINAL_STATUS=CLEAN
POST_PUBLICATION_VERIFICATION=PASS
```

An intermediate verification command reported a false negative for release-body
identity because `gh api --jq '.body'` serialized the extracted string with an
additional trailing newline. A subsequent exact JSON-string comparison using
Python compared 4,430 characters on each side and established:

```text
PUBLIC_BODY_CHARS=4430
EXPECTED_BODY_CHARS=4430
RELEASE_BODY_IDENTITY=PASS
```

No public state was changed by either verification.

## Retained Native/runtime policy

```text
PUBLIC_NATIVE_PLATFORM_MATRIX=LINUX_X86_64_GLIBC_DYNAMIC
PUBLIC_NATIVE_PLATFORM_BUILD_BASE=ORACLE_LINUX_10
PUBLIC_NATIVE_GLIBC_MIN=2.39
PUBLIC_NATIVE_CPU_ISA_POLICY=COMPATIBILITY
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility

GRAALVM_DISTRIBUTION=graalvm-community
GRAALVM_GRAAL_TRUFFLE_VERSION=25.4.4.1.1
JDK_FEATURE=25
JDK_VERSION=25.0.4.1.1
JAVA_BYTECODE_RELEASE=21
```

The release remains an experimental prerelease. Native support is exactly the
declared Linux x86_64 / dynamic glibc / glibc 2.39 minimum / compatibility-ISA
target. No Native support is claimed for macOS, Windows, AArch64, musl, or
earlier glibc.

The portable JVM artifact remains the compatibility fallback/external-consumer
artifact and requires the declared supported external GraalVM/JDK runtime.
Java bytecode release 21 is not a generic Java 21+ compatibility promise.

## Final D9 result

```text
DIST005_D9_STATUS=PUBLISHED_AND_VERIFIED

RELEASE_PUBLICATION_AUTHORIZED=YES

PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
RELEASE_BODY_IDENTITY=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS

PUBLIC_ASSET_COUNT=5
PUBLIC_RELEASE_PRERELEASE=true
PUBLIC_RELEASE_DRAFT=false

CANDIDATE_FINAL_STATUS=CLEAN
REAL_RELEASE_PUBLISHED=YES

DIST007_V0_3_116_AVAILABLE=NO

BLOCKERS=NONE
```

## Materially governing and inspected repository surfaces

- `AGENTS.md` — Human-executor publication boundary, exact project
  coordinates, and repository-publication safety.
- `AGENTS.work/IMPLEMENTATION.md` — shared implementation/validation
  discipline for the DIST slice.
- `AGENTS.work/RELEASE.md` — exact release authorization, prerelease policy,
  and release/snapshot separation.
- `dist/publish_release.py` — canonical fail-closed, idempotent publication
  primitive and the bounded missing-local-tag repair.
- `dist/test_publish_release.py` — publication transaction tests and the new
  missing-local-tag regression.
- DIST005-D7 durable evidence — exact owner-selected release baseline.
- DIST005-D8 durable evidence — exact prepared candidate, measurements,
  admission proof, archives, manifest, and release notes.

No normative Protos language or Standard Library semantics were changed by D9.

## Remaining DIST005 closure work

D9 deliberately stops before user-facing documentation reconciliation and parent
closure.

The remaining bounded work is to make the now-proven release the canonical
first-run path in maintained user-facing documentation, publish an appropriate
news item, reconcile the remaining DIST005 acceptance criteria, and only then
close `guillermomolina/protos#549`.

```text
NEXT_SLICE=DIST005-D10
NEXT_SLICE_NAME=user-facing first-run documentation and release announcement
NEXT_SLICE_WORK_TYPE=DOCUMENTATION / IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
