# DIST011-A — release readiness and exact baseline selection

Status: **PASS / OWNER BASELINE SELECTED / DIST011-B READY**

This durable, non-normative record retains the DIST011-A readiness investigation
and the project owner's exact release-baseline selection for DIST011 /
`guillermomolina/protos#774`.

## Ownership

```text
DATE=2026-10-02
WORK_ITEM=DIST011-A
GITHUB_ISSUE=guillermomolina/protos#774
TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

DIST011 succeeds the retired DIST007 portable-JVM-only plan and owns the next
selected current `JVM_PLUS_NATIVE` Protos prerelease.

DIST011-A performed no build, test, candidate materialization, tag creation,
GitHub Release creation, asset publication, product-repository mutation, or
Issue mutation before the owner selection.

## Current authoritative product state

The exact current `main` state inspected by DIST011-A was:

```text
CURRENT_MAIN=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
CURRENT_SNAPSHOT_VERSION=0.3.143-SNAPSHOT
CURRENT_SPECIFICATION_REVISION=0.1.436
CANONICAL_GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
```

The current root Maven version is a canonical development snapshot suitable for
the maintained release machinery.

## Latest public release

The current verified public Protos release remains:

```text
LATEST_PUBLIC_RELEASE=0.3.139
LATEST_PUBLIC_RELEASE_TAG=v0.3.139
LATEST_PUBLIC_RELEASE_MODEL=JVM_PLUS_NATIVE
DIST009_STATUS=COMPLETE
DIST009_CANDIDATE_SOURCE_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
DIST009_PROJECT_RECORD_REVISION=6302bb3663c0eedefbddd5840764b2c42201c6f8
```

No newer Protos GitHub Release was found during DIST011-A.

## Proposed and selected release identity

The maintained DIST001 release contract maps an explicitly selected
`V-SNAPSHOT` baseline mechanically to release version `V` and tag `vV`.

For current `main`, DIST011-A derived:

```text
PROPOSED_RELEASE_BASELINE_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
PROPOSED_RELEASE_BASELINE_VERSION=0.3.143-SNAPSHOT
PROPOSED_RELEASE_VERSION=0.3.143
PROPOSED_RELEASE_TAG=v0.3.143

TAG_COLLISION=NO
GITHUB_RELEASE_COLLISION=NO
```

The project owner explicitly approved that exact baseline/version/tag selection
in the active interaction on 2026-10-02.

The selected DIST011 release identity is therefore:

```text
OWNER_BASELINE_SELECTION=APPROVED
DECISION_APPROVAL_PROVENANCE=ACTIVE_PROJECT_OWNER_INTERACTION

RELEASE_BASELINE_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
RELEASE_BASELINE_VERSION=0.3.143-SNAPSHOT
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143
```

This selection freezes the baseline identity for DIST011-B. It does not itself
materialize a release-only candidate or authorize publication.

## Release architecture and Native capability

Current release architecture remains:

```text
MODEL=JVM_PLUS_NATIVE
NATIVE_ROLE=RECOMMENDED_FIRST_RUN_ARTIFACT
PORTABLE_JVM_ROLE=COMPATIBILITY_FALLBACK_AND_EXTERNAL_CONSUMER_ARTIFACT
JVM_PLUS_NATIVE_MODEL=CONFIRMED
```

PLAT045 remains applicable:

```text
NATIVE_IMAGE_SUPPORT=SUPPORTED
NATIVE_RUNTIME_MODE=INTERPRETER_ONLY_FALLBACK
NATIVE_GUEST_JIT_SUPPORT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
UPSTREAM_ORACLE_GRAAL_14579_STATUS=OPEN
PLAT045_CHANGE_REQUIRED_BEFORE_RELEASE=NO
```

At the investigation checkpoint, `oracle/graal#14579` remained open and had
not satisfied PLAT045's objective re-enable gate. DIST011 therefore keeps the
ratified interpreter-only Native guest-runtime contract without reopening
platform policy.

The public Native distribution policy remains:

```text
PUBLIC_NATIVE_TARGET_OS=linux
PUBLIC_NATIVE_TARGET_ARCH=x86_64
PUBLIC_NATIVE_LINKAGE=dynamic
PUBLIC_NATIVE_LIBC_FAMILY=glibc
PUBLIC_NATIVE_LIBC_ABI_MIN=2.39
PUBLIC_NATIVE_CPU_ISA_ASSUMPTION=compatibility
PUBLIC_NATIVE_BUILD_MARCH=-march=compatibility
```

## Maintained release machinery

Current HEAD retains the reusable exact-candidate and multi-asset release path:

```text
dist/prepare_release_candidate_worktree.py
dist/transition_release_candidate_version.py
dist/commit_release_candidate.py
dist/materialize_release_candidate.py
dist/build_native.py
dist/validate_native.py
dist/build_portable.py
dist/validate_portable.sh
dist/prepare_release_metadata.py
dist/release_asset_envelope.py
dist/verify_release_metadata.py
dist/validate_release_candidate.py
dist/prepare_release.py
dist/publish_release.py
```

The investigation established:

```text
JVM_PLUS_NATIVE_MACHINERY_REUSABLE=YES
DETACHED_OR_EXACT_CANDIDATE_MODEL_REUSABLE=YES
MULTI_ASSET_ENVELOPE_REUSABLE=YES
EXPLICIT_PUBLICATION_AUTHORIZATION_SUPPORTED=YES
POST_PUBLICATION_VERIFICATION_SUPPORTED=YES
MAIN_REMAINS_SNAPSHOT=YES
```

The release-only candidate model preserves development `main` on its
`0.3.143-SNAPSHOT` line. The candidate is a detached, single-parent commit
whose release-only source delta is the exact root `pom.xml`
`0.3.143-SNAPSHOT -> 0.3.143` transition.

DIST010's exact-revision artifact-set and D064 publication work does not replace
this public release path. Public release publication remains a separate explicit
operation.

## Required DIST011-B candidate gates

The selected exact candidate must regenerate and pass the current maintained
candidate evidence:

```text
NATIVE_BUILD=REQUIRED
NATIVE_COMPLETE_ADMISSION=REQUIRED
NATIVE_INTERPRETER_ONLY_ADMISSION=REQUIRED
PORTABLE_JVM_BUILD=REQUIRED
PORTABLE_JVM_ADMISSION=REQUIRED
MULTI_ASSET_ENVELOPE=REQUIRED
LICENSE_NOTICES=REQUIRED
CHECKSUMS=REQUIRED
CANDIDATE_IDENTITY=REQUIRED
```

Current gate ownership includes:

- Native build through `dist/prepare_release.py`, which composes the maintained
  Native Image builder and `dist/build_native.py --public-prerelease`.
- Complete extracted Native admission through `dist/validate_native.py`.
- PLAT045 interpreter-only admission in `dist/validate_native.py`, with focused
  policy tests in `dist/test_validate_native_plat045.py`.
- Portable public-prerelease build through `dist/build_portable.py`.
- Portable complete release admission through `dist/validate_portable.sh`.
- Multi-asset preparation and verification through
  `dist/prepare_release_metadata.py` and `dist/verify_release_metadata.py`.
- Candidate checkout, release-only lineage, selected-specification identity,
  claims/blockers audit, tag availability and portable release gate through
  `dist/validate_release_candidate.py`.
- Per-artifact `LICENSE.TXT`, `DEPENDENCIES.txt`, archive checksums and
  common-candidate identity bound by the JVM_PLUS_NATIVE release envelope.

## Consumer-specific gate

DIST009's prepublication VS Code Run/Debug gate was release-specific: v0.3.139
existed to deliver the BUG012-fixed runtime needed by DIST006-B2.

Current authoritative state does not establish an equivalent DIST011 consumer
dependency. DIST006-B2 already consumes the published v0.3.139 release.

```text
CONSUMER_SPECIFIC_PREPUBLICATION_GATE_REQUIRED=NO
CONSUMER_SPECIFIC_GATE_OWNER=NOT_APPLICABLE
CONSUMER_SPECIFIC_GATE_REASON=CANONICAL_PROTOS_RELEASE_ADMISSION_IS_SUFFICIENT
```

## Blocker review

The durable implementation blocker ledger currently has no open blocker that
prevents materializing, validating, or publishing the selected candidate.
BUG013 and TEST006 are closed, and PLAT045 explicitly accommodates the still-open
upstream Native guest-JIT limitation.

```text
BLOCKERS=NONE
```

Unrelated open implementation/performance/documentation work is not promoted to
a release blocker merely because it remains open.

## DIST011-A final classification

```text
DIST011_A_STATUS=PASS
OWNER_BASELINE_SELECTION_REQUIRED=NO
OWNER_BASELINE_SELECTION=APPROVED

RELEASE_BASELINE_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
RELEASE_BASELINE_VERSION=0.3.143-SNAPSHOT
RELEASE_VERSION=0.3.143
RELEASE_TAG=v0.3.143

TAG_COLLISION=NO
GITHUB_RELEASE_COLLISION=NO
JVM_PLUS_NATIVE_MODEL=CONFIRMED
PLAT045_CHANGE_REQUIRED_BEFORE_RELEASE=NO
CONSUMER_SPECIFIC_PREPUBLICATION_GATE_REQUIRED=NO
BLOCKERS=NONE
MAIN_REMAINS_SNAPSHOT=YES

CANDIDATE_CREATED=NO
TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
ASSETS_PUBLISHED=NO
```

## Next execution

The selected next slice is:

```text
NEXT_EXECUTABLE_UNIT=DIST011-B
NEXT_EXECUTABLE_TITLE=exact candidate build and validation
NEXT_EXECUTABLE_TYPE=IMPLEMENTATION
NEXT_EXECUTABLE_REPOSITORY=guillermomolina/protos
```

DIST011-B must assume the product repository is already at the selected exact
baseline, materialize/freeze the detached release-only candidate, build and
validate the Native and portable JVM assets from that candidate, validate the
common release envelope and exact identities, retain the resulting candidate
and manifest identities, and stop before publication.

Publication remains DIST011-C and still requires separate explicit project-owner
authorization bound to the exact frozen candidate and release manifest.
