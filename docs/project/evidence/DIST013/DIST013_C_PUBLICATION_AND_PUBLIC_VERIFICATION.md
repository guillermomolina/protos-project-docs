# DIST013-C — Protos 0.3.237 publication and independent verification

Status: **COMPLETE**

DIST013-C published the exact owner-authorized Protos 0.3.237 `JVM_PLUS_NATIVE` prerelease and independently re-read the public GitHub state after the maintained publisher reported success.

## Authorization identity

The explicit publication authorization was bound to:

```text
candidate_source_revision=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
release_version=0.3.237
release_tag=v0.3.237
release_manifest_sha256=cf997603d218a5fbf181287a5d52d7b35ff82eaa86e5676e39e37c6540ab0458
github_release_prerelease=true
github_release_draft=false
```

No broader or moving authorization was used.

## Maintained publication path

The publication was executed through `dist/publish_release.py`, which reported:

```text
DIST005_RELEASE_PUBLICATION: PASS
RELEASE_CANDIDATE_SOURCE_REVISION=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
RELEASE_VERSION=0.3.237
RELEASE_TAG=v0.3.237
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
RELEASE_PUBLICATION_AUTHORIZED=YES
TAG_CREATED_OR_REUSED=YES
PUBLIC_RELEASE_CREATED_OR_REUSED=YES
```

## Independent public verification

A fresh GitHub API read after publication established:

```text
PUBLIC_TAG=v0.3.237
PUBLIC_TAG_OBJECT_TYPE=commit
PUBLIC_TAG_TARGET=f627734c8feaf3c26c4a32903b6ea0e25f4a9956

GITHUB_RELEASE_ID=404612232
GITHUB_RELEASE_NAME=Protos 0.3.237
GITHUB_RELEASE_PRERELEASE=true
GITHUB_RELEASE_DRAFT=false
GITHUB_RELEASE_PUBLISHED_AT=2026-10-06T11:07:11Z
PUBLISHED_ASSET_COUNT=5
```

The public Release body also independently contains the exact candidate SHA, baseline SHA, specification revision `0.1.444`, distribution model `JVM_PLUS_NATIVE`, Native role `recommended-first-run`, and portable role `compatibility-fallback`.

## Public asset set and GitHub digests

GitHub exposes the following exact uploaded asset set and SHA-256 digests:

```text
protos-0.3.237-native-linux-x86_64.zip
sha256:510537c7c47a323ed05a5ea8e4e46bc3cd63c761a58e68cb0639548feb8cc46c

protos-0.3.237-native-linux-x86_64.zip.sha256
sha256:2321a11dae0ca9feca2d891ef0a76cc2cc6df92a2e46fc78e1d96b5fdfe75e75

protos-0.3.237-posix-jvm.zip
sha256:28f200f3c1d9b218ddbfdb5c906204418c543260371a4ca58119a228d5e79565

protos-0.3.237-posix-jvm.zip.sha256
sha256:5a8587ac512abd1b7d652b74876932533822c1865e347bf00db24aa2bb7d5700

RELEASE_MANIFEST.txt
sha256:cf997603d218a5fbf181287a5d52d7b35ff82eaa86e5676e39e37c6540ab0458
```

All five public digests match the immutable DIST013-B identities exactly. No unexpected public assets were present.

## Closure

```text
PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
RELEASE_MANIFEST_IDENTITY=PASS
POST_PUBLICATION_VERIFICATION=PASS
DURABLE_RELEASE_EVIDENCE=PASS
DIST013=COMPLETE
```

The published Protos runtime now contains LM011-D1, so LM011-E no longer has a release prerequisite blocker. Its extension runtime lock may now be updated to this real release and the final installed-VSIX -> released LSP -> TOOL010 end-to-end closure can proceed.
