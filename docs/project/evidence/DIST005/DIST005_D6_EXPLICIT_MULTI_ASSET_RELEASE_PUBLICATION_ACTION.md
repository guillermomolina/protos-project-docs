# DIST005-D6 — explicit multi-asset release publication action

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D6 — explicit multi-asset release publication action`

## Published implementation identity

DIST005-D6 is published on `guillermomolina/protos/main` as:

```text
IMPLEMENTATION_BASE_REVISION=c7be048bab19409a139b1a1f6ad74d23f17b78d7
IMPLEMENTATION_REVISION=1637fe514ee9327241e49964031d981423b16198
IMPLEMENTATION_VERSION=0.3.116-SNAPSHOT
COMMIT_SUBJECT=DIST005-D6: add explicit release publication action
```

Published changed paths:

```text
CHANGELOG.md
dist/README.md
dist/publish_release.py
dist/test_publish_release.py
pom.xml
```

No normative Protos language or Standard Library specification file changed.

## Publication primitive

D6 adds:

```text
PUBLICATION_ENTRY_POINT=dist/publish_release.py
PUBLICATION_AUTHORIZATION_FORMAT=protos-dist005-publication-authorization-v1
PREPARE_AND_PUBLISH_SEPARATE=YES
```

The authorization is a separate exact record bound to:

```text
release_publication_authorized=true
authorization_basis=explicit-user-decision
candidate_source_revision=<exact-40-hex-candidate-sha>
release_version=<V>
release_tag=<vV>
release_manifest_sha256=<exact-manifest-sha256>
github_release_prerelease=true
github_release_draft=false
```

Malformed, incomplete, duplicate, unknown, mismatched or stale authorization
fails closed.

## Pre-publication guards

Before the first public mutation, the implementation requires exact agreement
among the authorization, detached clean release-only candidate, canonical
`guillermomolina/protos` origin, release manifest, release notes, two prepared
archives, both external checksums and the existing independent multi-asset
metadata verifier.

The publication path does not rebuild artifacts or regenerate metadata.

## Retained release identity

D6 retains the DIST001 release identity model:

```text
LIGHTWEIGHT_TAG_MODEL_RETAINED=YES
EXACT_CANDIDATE_TAG_TARGET_REQUIRED=YES
GITHUB_RELEASE_TITLE=Protos <V>
GITHUB_RELEASE_PRERELEASE=true
GITHUB_RELEASE_DRAFT=false
RELEASE_NOTES_USED_AS_BODY=YES
RELEASE_NOTES_UPLOADED_AS_ASSET=NO
```

For the selected current two-artifact `JVM_PLUS_NATIVE` model, the exact
downloadable asset set is:

```text
EXPECTED_PUBLIC_ASSET_COUNT=5
1=Native ZIP
2=Native ZIP.sha256
3=portable JVM ZIP
4=portable JVM ZIP.sha256
5=RELEASE_MANIFEST.txt
```

## Recovery and conflict policy

The transaction supports exact idempotent/resumable states without destructive
repair:

```text
local exact lightweight tag only=resumable
remote exact tag without release=resumable
exact partial release with exact existing assets=resumable
exact complete release=idempotent success
conflicting tag=fail closed
conflicting release metadata/body=fail closed
mismatched asset bytes=fail closed
unexpected public asset=fail closed

FORCE_TAG_ALLOWED=NO
FORCE_PUSH_ALLOWED=NO
ASSET_CLOBBER_ALLOWED=NO
AUTOMATIC_RELEASE_DELETE_ALLOWED=NO
AUTOMATIC_TAG_DELETE_ALLOWED=NO
```

A failure after public tag creation performs a fresh read-only state inspection
and reports whether the exact transaction can be safely resumed.

## Focused and regression validation

D6 focused tests:

```text
FOCAL_TESTS=32/32 PASS
```

Shared regression evidence reported during implementation:

```text
D5_PREPARATION_TESTS=4/4 PASS
MULTI_ASSET_ENVELOPE_TESTS=18/18 PASS
RELEASE_METADATA_VERIFIER_TESTS=6/6 PASS
RELEASE_CANDIDATE_GATE_TESTS=10/10 PASS
RELEASE_CANDIDATE_COMMIT_TESTS=6/6 PASS
```

An intermediate direct invocation of
`test_materializes_unpublished_local_main_baseline` exposed an existing
fixture naming mismatch: the temporary repository's default branch was
`master` while that test requested `--baseline-ref main`. D6 did not modify
that machinery. The repository's final integrated validation passed.

## Final immutable repository validation

Validation was executed against the immutable range:

```text
BASE=c7be048bab19409a139b1a1f6ad74d23f17b78d7
CANDIDATE=1637fe514ee9327241e49964031d981423b16198
```

The repository selected full validation:

```text
SOURCE_STYLE_GUARD=PASS
SOURCE_STYLE_PREVENTION_GATE=PASS
LEGACY_EXECUTION_GUARD=PASS
LEGACY_EXECUTION_PREVENTION_GATE=PASS
VALIDATION_IMPACT=FULL
AFFECTED_TEST_SET=ALL
PROTOS_TEST_TOOL=1263 passed, 0 failed
FULL_TEST_SUITE=PASS
PUBLICATION_VALIDATION=PASS
PUBLICATION_VALIDATION_STATUS=0
```

Final published identity was:

```text
HEAD=1637fe514ee9327241e49964031d981423b16198
ORIGIN_MAIN=1637fe514ee9327241e49964031d981423b16198
WORKTREE=CLEAN
```

## Real-publication boundary

D6 implemented and validated the future publication path but did not execute it.

```text
REAL_RELEASE_PUBLISHED=NO
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
PUBLIC_ASSETS_UPLOADED=NO
```

## Result

```text
DIST005_D6_STATUS=PASS
IMPLEMENTATION_REVISION=1637fe514ee9327241e49964031d981423b16198
IMPLEMENTATION_VERSION=0.3.116-SNAPSHOT
PREPARE_AND_PUBLISH_SEPARATE=YES
PUBLICATION_RECOVERY_MODEL=EXACT_FAIL_CLOSED_RESUMABLE
EXPECTED_PUBLIC_ASSET_COUNT=5
REAL_RELEASE_PUBLISHED=NO
```

## Handoff

With D5 and D6, DIST005 now has separate deterministic preparation and explicit
publication primitives. A real multi-asset release still requires a separate
release-readiness/baseline-selection decision, preparation of the exact selected
candidate, and explicit publication authorization bound to that candidate and
manifest. No successful implementation/test/main state implicitly authorizes
that release.
