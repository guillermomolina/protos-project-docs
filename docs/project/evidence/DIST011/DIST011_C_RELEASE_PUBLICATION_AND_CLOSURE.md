# DIST011-C — v0.3.143 release publication and closure

Status: **PASS / PUBLICATION INDEPENDENTLY VERIFIED / DIST011 COMPLETE**

This durable, non-normative record retains the final publication and
post-publication verification evidence for DIST011 / `guillermomolina/protos#774`.

## Ownership

```text
DATE=2026-10-02
WORK_ITEM=DIST011-C
GITHUB_ISSUE=guillermomolina/protos#774
PUBLICATION_REPOSITORY=guillermomolina/protos
RELEASE_URL=https://github.com/guillermomolina/protos/releases/tag/v0.3.143
```

## Frozen publication identity

DIST011-C published the exact candidate frozen and admitted by DIST011-B:

```text
PROTOS_REVISION=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
CANDIDATE_SOURCE_REVISION=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
RELEASE_BASELINE_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
RELEASE_BASELINE_VERSION=0.3.143-SNAPSHOT
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143
SPECIFICATION_REVISION=0.1.436
RELEASE_MANIFEST_SHA256=c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591
```

The exact DIST011-B checkpoint is retained at:

```text
DIST011_B_PROJECT_RECORD_REVISION=dedc574a02bd13383e719c701c4ca52400127eb2
DIST011_B_PROJECT_RECORD_PATH=docs/project/evidence/DIST011/DIST011_B_EXACT_CANDIDATE_VALIDATION_CHECKPOINT.md
```

## Explicit publication authorization

The project owner authorized the exact frozen candidate and manifest identity
before publication.

```text
publication_authorization_format=protos-dist005-publication-authorization-v1
release_publication_authorized=true
authorization_basis=explicit-user-decision
candidate_source_revision=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
release_version=0.3.143
release_tag=v0.3.143
release_manifest_sha256=c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591
github_release_prerelease=true
github_release_draft=false
```

Durable authorization authority:

```text
DIST011_C_AUTHORIZATION_REVISION=f41011385c5131f4c26a059f2b5a2ddd601f4958
DIST011_C_AUTHORIZATION_PATH=docs/project/evidence/DIST011/DIST011_C_PUBLICATION_AUTHORIZATION.txt
```

## Maintained publication result

The human executor invoked the maintained `dist/publish_release.py` path
against the already-prepared candidate and envelope. The successful invocation
reported:

```text
DIST005_RELEASE_PUBLICATION=PASS
RELEASE_CANDIDATE_SOURCE_REVISION=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
RELEASE_PUBLICATION_AUTHORIZED=YES
TAG_CREATED_OR_REUSED=YES
PUBLIC_RELEASE_CREATED_OR_REUSED=YES
DIST011_C_PUBLISH_RC=0
```

An earlier malformed shell invocation failed locally before a valid publication
authorization file was supplied. The maintained publisher rejected that
invocation at authorization parsing; it did not constitute a successful or
partially authorized publication. The subsequent exact invocation above is the
publication authority.

## Independent public tag verification

GitHub's public tag ref was independently re-read after publication:

```text
PUBLIC_TAG_REF=refs/tags/v0.3.143
PUBLIC_TAG_OBJECT_TYPE=commit
PUBLIC_TAG_TARGET=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
PUBLIC_TAG_IDENTITY=PASS
```

## Independent GitHub Release verification

GitHub's public Release was independently re-read:

```text
RELEASE_NAME=Protos 0.3.143
RELEASE_TAG=v0.3.143
PRERELEASE=true
DRAFT=false
PUBLISHED_AT=2026-10-02T11:14:56Z
ASSET_COUNT=5
GITHUB_PRERELEASE_METADATA=PASS
```

The public release notes identify the exact candidate revision, baseline
revision, release version, `JVM_PLUS_NATIVE` model and the ratified Native
interpreter-only limitation.

## Independently reported public asset identities

GitHub's public asset metadata exposes SHA-256 digests matching the frozen
DIST011-B envelope:

```text
b8970146ba82d574635d4409b186021f7d2f6934fc8c05ed0ee4ae5f4b5b8534  protos-0.3.143-native-linux-x86_64.zip
1e6005a086685e39e1dc983aee48a124071bfa7176cd7b6b1e22aad5d52e64ee  protos-0.3.143-native-linux-x86_64.zip.sha256
3c2fab126e5f83f8e16b7e7284ccc9a269e5120f109e8f11affec51a3e3baa93  protos-0.3.143-posix-jvm.zip
5b18ee683493660a073260b13737cbd00677a27d891063d9ee83886403ca03f4  protos-0.3.143-posix-jvm.zip.sha256
c1e1579feb8d42bc90bf8f3886e3b5dbcf948b0fef6c0501d1f42d81b4233591  RELEASE_MANIFEST.txt
```

Therefore:

```text
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS
```

## Release model and capability contract

The publication retains the selected release architecture:

```text
MODEL=JVM_PLUS_NATIVE
NATIVE_ROLE=recommended-first-run
PORTABLE_JVM_ROLE=compatibility-fallback
CANONICAL_GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1

NATIVE_IMAGE=SUPPORTED
NATIVE_EXECUTION=INTERPRETER_ONLY
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
```

DIST011-C did not rebuild the candidate or change PLAT045 policy.

## Mutation accounting

```text
PROTOS_REPOSITORY_FILE_CHANGES=NONE
CANDIDATE_REBUILT=NO
MAIN_REMAINS_SNAPSHOT=YES
SPECIFICATION_CHANGED=NO
```

Public mutations were limited to the maintained release path:

- lightweight Git tag `v0.3.143`;
- GitHub prerelease `Protos 0.3.143`;
- the five exact public assets listed above.

## DIST011 final classification

```text
CURRENT_RELEASE_SELECTED=YES
EXACT_BASELINE_SELECTED=YES
EXACT_CANDIDATE_FROZEN=YES
JVM_PLUS_NATIVE_MODEL_REUSED=YES

NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_INTERPRETER_ONLY=PASS
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
PORTABLE_JVM_ADMISSION=PASS
MULTI_ASSET_ENVELOPE=PASS
LICENSE_NOTICES=PASS
CHECKSUMS=PASS
CANDIDATE_IDENTITY=PASS
CONSUMER_SPECIFIC_GATE=NOT_REQUIRED

MAIN_REMAINS_SNAPSHOT=YES
PUBLICATION_AUTHORIZATION=PASS

PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS

DURABLE_RELEASE_EVIDENCE=PASS
DIST011_C_STATUS=PASS
DIST011_STATUS=COMPLETE
```

## Closure

DIST011's acceptance criteria are satisfied. No further slice remains inside
DIST011.

```text
REQUIRED_DURABLE_PUBLICATION=PASS
CROSS_REFERENCES=PASS
NEXT_EXECUTABLE_UNIT=NONE_WITHIN_DIST011
```
