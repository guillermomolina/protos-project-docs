# DIST013-A — Protos 0.3.236 release readiness and exact baseline selection

Status: **PASS / OWNER BASELINE SELECTED / DIST013-B READY**

This durable, non-normative record retains the release preflight and the project
owner's exact baseline selection for DIST013 / `guillermomolina/protos#805`.
DIST013 exists to publish the first Protos release that contains LM011-D1 and
thereby unblock LM011-E / `guillermomolina/protos#670`.

## Ownership

```text
DATE=2026-10-06
WORK_ITEM=DIST013-A
GITHUB_ISSUE=guillermomolina/protos#805
DEPENDENT_WORK_ITEM=LM011/#670
TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

DIST013-A used read-only GitHub/repository evidence. It performed no product
repository mutation, candidate materialization, build, test, tag creation,
GitHub Release creation, or asset publication.

## LM011 prerequisite

LM011-A through LM011-D are complete. The exact server-side formatting revision
that must be present in a released runtime is:

```text
LM011_D1_REVISION=75cfed853c7eef8c9d3dfb0c6f2507fdf8864ecb
LM011_D1_CAPABILITY=textDocument/formatting -> TOOL010
```

The committed VS Code extension runtime lock remains on Protos v0.3.139, which
predates LM011-D1.

## Published-release preflight

The latest Protos GitHub Release observed by the preflight was:

```text
LATEST_PUBLISHED_RELEASE=0.3.143
LATEST_PUBLISHED_TAG=v0.3.143
LATEST_PUBLISHED_CANDIDATE=75418458f6f7d6dae27ad7009fe4bd57e7499c2d
PUBLISHED_RUNTIME_CONTAINING_LM011_D1=NO
```

Git ancestry inspection established that the v0.3.143 candidate does not contain
LM011-D1. Version-number comparison alone was not used as release identity.

## Selected baseline

The preflight identified the following exact development revision as a viable
release baseline:

```text
RELEASE_BASELINE_REVISION=22e3aa9466d172ac816b3a74b4e3ca9db155a92e
RELEASE_BASELINE_VERSION=0.3.236-SNAPSHOT
RELEASE_VERSION=0.3.236
RELEASE_TAG=v0.3.236
SPECIFICATION_REVISION=0.1.444

BASELINE_CONTAINS_LM011_D1=YES
TAG_COLLISION=NO
```

The project owner explicitly approved that exact baseline/version/tag selection
in the active interaction on 2026-10-06.

```text
OWNER_BASELINE_SELECTION=APPROVED
DECISION_APPROVAL_PROVENANCE=ACTIVE_PROJECT_OWNER_INTERACTION
```

This approval freezes the DIST013 release baseline and authorizes candidate
preparation/validation. It does not authorize release publication.

## Maintained release path

DIST013 reuses the existing release machinery in `guillermomolina/protos`,
including the maintained detached candidate, JVM_PLUS_NATIVE build, release
admission, envelope/checksum, and separate publication paths.

The selected flow is:

```text
22e3aa9466d172ac816b3a74b4e3ca9db155a92e / 0.3.236-SNAPSHOT
    -> detached release-only 0.3.236 candidate
    -> Native + portable JVM build and complete admission
    -> multi-asset envelope + identity/checksum verification
    -> STOP
    -> separate exact publication authorization
    -> v0.3.236 + GitHub prerelease + validated assets
```

The maintained publication mechanism requires the later authorization to be
bound to the exact candidate source revision, public version/tag, release
manifest SHA-256, prerelease=true and draft=false.

## Prepublication state

```text
CANDIDATE_SOURCE_REVISION=UNMATERIALIZED
RELEASE_PUBLICATION_AUTHORIZED=NO
GIT_TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
RELEASE_ASSETS_PUBLISHED=NO
MAIN_REMAINS_SNAPSHOT=YES
```

No future movement of `main` changes the owner-selected DIST013 baseline.

## DIST013-B authorization

DIST013-B is released for implementation in `guillermomolina/protos`.

It must be one productive candidate-preparation/validation slice: consume the
machine-readable selection record, materialize the exact detached release-only
candidate, build and validate both public artifacts, validate the common
release envelope and LM011-D1 runtime presence, retain the exact candidate and
digest identities, and stop before any public mutation.

```text
DIST013_A_STATUS=PASS
DIST013_B_RELEASED=YES
NEXT_EXECUTABLE_UNIT=DIST013-B
NEXT_EXECUTABLE_TYPE=IMPLEMENTATION
NEXT_EXECUTABLE_REPOSITORY=guillermomolina/protos
PUBLICATION_AUTHORIZED=NO
```

DIST013-C remains a later publication slice and requires a separate explicit
project-owner authorization after DIST013-B produces the immutable candidate
source revision and release-manifest SHA-256.

## LM011 impact

LM011-E remains blocked while DIST013 is unpublished.

Once DIST013-C has published and independently verified v0.3.236, LM011-E may
consume that real release in the VS Code extension lock and perform its single
final installed-VSIX -> released Protos LSP -> TOOL010 corpus/end-to-end closure
slice.
