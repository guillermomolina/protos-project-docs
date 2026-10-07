# DIST009-C — v0.3.139 release publication and closure

Status: **PASS / PUBLICATION VERIFIED / DIST009 CLOSURE READY**

This durable, non-normative record retains the final publication and
post-publication verification evidence for DIST009 / `guillermomolina/protos#743`.

## Ownership

```text
DATE=2026-10-02
WORK_ITEM=DIST009-C
GITHUB_ISSUE=guillermomolina/protos#743
CONSUMER=DIST006-B2/guillermomolina/protos#734
PUBLICATION_REPOSITORY=guillermomolina/protos
RELEASE_URL=https://github.com/guillermomolina/protos/releases/tag/v0.3.139
```

DIST009 owned publication of the frozen JVM_PLUS_NATIVE prerelease candidate
after B1 and B2 exact-candidate validation passed. DIST006-B2 remains the
consumer that updates the public VS Code extension runtime lock and performs
final installed/public Run/Debug acceptance.

## Frozen candidate authority

The published release is the exact candidate already frozen and admitted by
DIST009-B1/B2:

```text
CANDIDATE_SOURCE_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
BASELINE_REVISION=d70d4438170493b61c9da70781130e2a724ce4a1
RELEASE_VERSION=0.3.139
RELEASE_TAG=v0.3.139
SPECIFICATION_REVISION=0.1.436

BUG012_REPAIR_IN_ANCESTRY=YES
DIST009_B1_STATUS=PASS
DIST009_B2_STATUS=PASS
```

The exact B1/B2 checkpoint remains:

```text
B2E_HARNESS_IMPLEMENTATION_REVISION=4b0659b4061a5d0e9a7de46cf30920e7b2004af9
PACKAGED_VSIX_SHA256=d60a43d17d32020c791e6bec4ce4917dadf3efea49b5e4ddfd6063976b152cf7
```

## Explicit publication authorization

The maintained `dist/publish_release.py` path consumed the exact publication
authorization for this candidate and manifest identity:

```text
publication_authorization_format=protos-dist005-publication-authorization-v1
release_publication_authorized=true
authorization_basis=explicit-user-decision
candidate_source_revision=3895206897ddac795dfebd49709ca97f8d0908b1
release_version=0.3.139
release_tag=v0.3.139
release_manifest_sha256=aeee10ec0abb8d9064068d67a1a6c000f03b7bce47ef04bd0d9e6e94c0d5fdc6
github_release_prerelease=true
github_release_draft=false
```

The publication used the retained B1 candidate and release envelope. Nothing
was rebuilt for DIST009-C.

## Publication result

The human executor reported the maintained publication primitive PASS:

```text
DIST009_C_STATUS=PASS

CANDIDATE_SOURCE_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
RELEASE_VERSION=0.3.139
RELEASE_TAG=v0.3.139

RELEASE_MANIFEST_SHA256=aeee10ec0abb8d9064068d67a1a6c000f03b7bce47ef04bd0d9e6e94c0d5fdc6
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS

DIST006_B2_PUBLISHED_NATIVE_PREREQUISITE=READY
```

No partial publication state was observed.

## Independent public verification

GitHub's public tag ref was re-read after publication:

```text
REF=refs/tags/v0.3.139
REF_TYPE=commit
PUBLIC_TAG_TARGET=3895206897ddac795dfebd49709ca97f8d0908b1
PUBLIC_TAG_IDENTITY=PASS
```

The public GitHub Release reports:

```text
RELEASE_NAME=Protos 0.3.139
PRERELEASE=true
DRAFT=false
ASSET_COUNT=5
```

The release notes identify the same candidate revision, baseline, specification
revision and JVM_PLUS_NATIVE distribution model.

## Published asset identities

All five public assets were independently downloaded and re-hashed by the human
executor. The results match the retained release manifest:

```text
8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da  protos-0.3.139-native-linux-x86_64.zip
18d27bfed2343ace104f28ba4b6dcca6c7f5426bf44c44b075fb34260d455b31  protos-0.3.139-native-linux-x86_64.zip.sha256
4aa04e89323d4c7f5f9cdb7ac87d74dfd2351d2b18ae806fb75b6e9c4dcbf302  protos-0.3.139-posix-jvm.zip
7332e6e9e7937d5d97ae9cf8a7f63554dfe3b351043081d500c74c57d4390f72  protos-0.3.139-posix-jvm.zip.sha256
aeee10ec0abb8d9064068d67a1a6c000f03b7bce47ef04bd0d9e6e94c0d5fdc6  RELEASE_MANIFEST.txt
```

GitHub's public release metadata independently exposes matching SHA-256 digests
for these five assets.

The recommended Native artifact is therefore publicly bound to:

```text
NATIVE_ASSET=protos-0.3.139-native-linux-x86_64.zip
NATIVE_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da
```

## Native capability contract

The public release retains the ratified PLAT045 capability boundary:

```text
NATIVE_IMAGE=SUPPORTED
NATIVE_EXECUTION=INTERPRETER_ONLY
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
```

DIST009-C did not weaken or reinterpret this contract.

## GitHub coordination evidence

The publication outcome was recorded on DIST009:

```text
ISSUE=guillermomolina/protos#743
COMMENT_ID=5947553103
```

The cleared prerequisite was recorded on DIST006-B:

```text
ISSUE=guillermomolina/protos#734
COMMENT_ID=5947553359
```

## Scope and mutation accounting

```text
PROTOS_REPOSITORY_FILE_CHANGES=NONE
SPECIFICATION_CHANGED=NO
PROTOS_SOURCE_LOCK_CHANGED=NO
CANDIDATE_REBUILT=NO
PARTIAL_PUBLICATION_STATE=NONE
```

Public mutations owned by DIST009-C were limited to the maintained publication
path:

- lightweight Git tag `v0.3.139`;
- GitHub pre-release `Protos 0.3.139`;
- the five release assets listed above.

## DIST009 final classification

DIST009's required release outcome is complete:

```text
CURRENT_RELEASE_SELECTED=YES
BUG012_REPAIR_IN_ANCESTRY=YES
JVM_PLUS_NATIVE_MODEL_REUSED=YES

NATIVE_COMPLETE_ADMISSION=PASS
NATIVE_DAP_STACKTRACE_REGRESSION=PASS
NATIVE_INTERPRETER_ONLY=PASS
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
PORTABLE_JVM_ADMISSION=PASS
MULTI_ASSET_ENVELOPE=PASS

PREPUBLICATION_VSCODE_RUN=PASS
PREPUBLICATION_VSCODE_DEBUG=PASS

PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS

DIST009_C_STATUS=PASS
DIST009_STATUS=COMPLETE
DIST006_B2_PUBLISHED_NATIVE_PREREQUISITE=READY
```

## Closure and routing

The publication and exact public identities above provide the required durable
closure evidence for DIST009.

```text
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PASS
NEXT_EXECUTABLE_UNIT=DIST006-B2
NEXT_EXECUTABLE_TYPE=IMPLEMENTATION
NEXT_EXECUTABLE_REPOSITORY=guillermomolina/protos-vscode-extension
```

DIST006-B2 may now update `protos-source.lock.json` to the exact published
v0.3.139 runtime identity and perform final installed/public real Run and real
Debug acceptance.
