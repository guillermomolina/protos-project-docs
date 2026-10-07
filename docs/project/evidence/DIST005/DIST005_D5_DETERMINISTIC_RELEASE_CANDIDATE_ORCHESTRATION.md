# DIST005-D5 — deterministic release-candidate orchestration

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Slice: `DIST005-D5 — deterministic release-candidate orchestration`

## Published implementation identity

The D5 orchestration was published in two bounded Protos commits:

```text
D5_IMPLEMENTATION_REVISION=d9dc332942a7954b5aa5cdcf39dac2c39b500b65
D5_REPAIR_REVISION=d47b501a8bf99ad4f67eec08e7a1e8791caee3c1
D5_IMPLEMENTATION_VERSION=0.3.114-SNAPSHOT
PRIMARY_COMMIT_SUBJECT=DIST005-D5: orchestrate release candidate preparation
REPAIR_COMMIT_SUBJECT=DIST005-D5: forward local main baseline authority
```

The repair preserves the existing default `origin/main` authority in the
generic candidate materializer while allowing the composed D5 preparation path
to consume an explicitly selected unpublished local `main` baseline during
the human-executed pre-publication proof.

No normative Protos language or Standard Library specification file changed.

## Composed preparation entry point

D5 added:

```text
dist/prepare_release.py
```

The bounded preparation flow composes the already-proven DIST001/DIST005
mechanisms:

```text
explicit selection
    ->
detached release-only candidate materialization/reuse
    ->
Native Image build
    ->
Native public archive build
    ->
complete Native admission
    ->
portable JVM build
    ->
complete portable JVM admission
    ->
multi-asset envelope preparation
    ->
independent multi-asset envelope verification
```

Preparation deliberately remains separate from publication.

## Exact D5 proof candidate

The final D5 proof used:

```text
RELEASE_BASELINE_REVISION=d47b501a8bf99ad4f67eec08e7a1e8791caee3c1
RELEASE_BASELINE_VERSION=0.3.114-SNAPSHOT
RELEASE_CANDIDATE_SOURCE_REVISION=7644b5fad7ac24f66b8475f47599f839114ba33f
RELEASE_VERSION=0.3.114
RELEASE_TAG=v0.3.114
SPECIFICATION_REVISION=0.1.434
```

This was proof evidence only. It did not select or publish a public release.

## Preparation result

The final proof reported:

```text
DIST005_RELEASE_PREPARATION: PASS
COMMON_CANDIDATE_IDENTITY=PASS
CANDIDATE_FINAL_STATUS=CLEAN
RECOMMENDED_FIRST_RUN_ARTIFACT=NATIVE
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
```

Portable admission included:

```text
DIST_B4A_TEST_TOOL_JOBS=8
TEST001_H_PORTABLE_REPOSITORY_SUITE_CHECK=PASS passed=1263 failed=0
DIST001_B4A_SMOKE=PASS
DIST001_B4B_SMOKE=PASS
DIST001_B5_CROSS_SLICE=PASS
```

The observed portable archive identity was:

```text
PORTABLE_JVM_ARCHIVE=protos-0.3.114-posix-jvm.zip
PORTABLE_JVM_SHA256=40820561219cecabe26757393a1bc892d364c559c766fdc94834e98f794afca6
```

## Publication boundary

D5 is preparation machinery and proof, not release publication.

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
PUBLIC_RELEASE_CREATED=NO
PUBLIC_ASSETS_UPLOADED=NO
```

The proof candidate, version and archive digest above are retained historical
evidence and are not hard-coded release selections for later slices.

## Result

```text
DIST005_D5_STATUS=PASS
PREPARATION_ENTRY_POINT=dist/prepare_release.py
PREPARE_AND_PUBLISH_SEPARATE=YES
COMMON_CANDIDATE_IDENTITY=PASS
CANDIDATE_FINAL_STATUS=CLEAN
REAL_RELEASE_PUBLISHED=NO
```

## Handoff

D5 closed the fragmented preparation procedure. The remaining mechanical gap
was an explicitly authorized, fail-closed publication transaction for one
already-prepared and already-verified `JVM_PLUS_NATIVE` candidate. That work
was assigned to DIST005-D6.
