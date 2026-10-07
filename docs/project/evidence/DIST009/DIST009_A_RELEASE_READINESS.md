# DIST009-A — current release readiness

Status: **READY FOR IMPLEMENTATION**

This durable, non-normative record captures the live GitHub/repository evidence
used to close the DIST009-A investigation on 2026-09-30.

## Scope

```text
WORK_ITEM=DIST009-A
GITHUB_ISSUE=guillermomolina/protos#743
TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
BUG012=guillermomolina/protos#742
BUG012_REPAIR_REVISION=417e44a8c60eaba1db7859d78bbb1e61c5dace64
```

DIST009 publishes the next current Protos `JVM_PLUS_NATIVE` prerelease that
contains BUG012 in its ancestry. The investigation does not freeze or publish a
candidate.

## Live baseline

At investigation time the authoritative `main` state was:

```text
CURRENT_MAIN=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
CURRENT_MAIN_COMMIT=I075-D: enable int boxing elimination with current-node lexical authority
CURRENT_SNAPSHOT_VERSION=0.3.125-SNAPSHOT
```

GitHub compare evidence from BUG012 repair
`417e44a8c60eaba1db7859d78bbb1e61c5dace64` to current `main` reported:

```text
STATUS=ahead
AHEAD_BY=5
BEHIND_BY=0
MERGE_BASE=417e44a8c60eaba1db7859d78bbb1e61c5dace64
BUG012_IN_ANCESTRY=YES
```

Therefore the current baseline contains the BUG012 repair and is not pinned to
the older `0.3.121-SNAPSHOT` state where BUG012 first landed.

## Release identity implied by current policy

The retained DIST001 policy maps an explicitly selected development baseline
`V-SNAPSHOT` to public version `V` and tag `vV`.

For the observed current baseline:

```text
PROPOSED_RELEASE_VERSION=0.3.125
PROPOSED_RELEASE_TAG=v0.3.125
```

Live GitHub checks found no matching tag ref and no GitHub Release for
`v0.3.125`:

```text
TAG_COLLISION=NO
RELEASE_COLLISION=NO
```

The existing published Native release remains `v0.3.116`, which predates
BUG012.

This readiness result is not an advance freeze of `0.3.125`. If `main`
advances before candidate materialization, DIST009-B1 selects the then-current
intended `V-SNAPSHOT` baseline, provided BUG012 remains in its ancestry. Once a
concrete candidate enters validation, that exact candidate is frozen for that
release attempt.

## Existing release machinery

The current repository still carries the established DIST001/DIST005 release
path:

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
dist/prepare_release.py
dist/publish_release.py
```

`dist/prepare_release.py` composes candidate materialization, Native archive
build/admission, portable JVM archive build/admission, and common multi-asset
release-envelope preparation/verification. `dist/publish_release.py` remains a
separate explicitly authorized publication step.

The release model remains:

```text
MODEL=JVM_PLUS_NATIVE
NATIVE_ROLE=recommended-first-run
PORTABLE_JVM_ROLE=compatibility-fallback
JVM_PLUS_NATIVE_MACHINERY_REUSABLE=YES
ARCHITECTURAL_CHANGE_REQUIRED=NO
```

No second release path is required.

## Candidate-specific validation

Product bytes change for the new release candidate, so the following evidence
must be regenerated for the exact candidate:

```text
NATIVE_COMPLETE_ADMISSION=REQUIRED
NATIVE_DAP_BREAKPOINT_STACKTRACE_REGRESSION=REQUIRED
NATIVE_FORCED_GUEST_TIER2_JIT=REQUIRED
PORTABLE_JVM_ADMISSION=REQUIRED
MULTI_ASSET_RELEASE_ENVELOPE=REQUIRED
EXACT_CANDIDATE_VSCODE_RUN=REQUIRED
EXACT_CANDIDATE_VSCODE_DEBUG=REQUIRED
```

The maintained BUG012 regression is
`build/native/test-dap-stacktrace.py`, driven by
`build/native/test-native.sh`. That gate proves a real suspended breakpoint,
successful `stackTrace`, continuation, clean termination, and the existing
forced guest Tier-2 compilation checks.

A relevant orchestration detail is that `dist/prepare_release.py` invokes the
packaged Native admission in `dist/validate_native.py`, but it does not itself
invoke `build/native/test-native.sh`. DIST009 candidate validation must
therefore retain the maintained Native regression as an explicit gate rather
than assuming the release-preparation composition subsumes it.

The already-retained prepublication VS Code result proves the BUG012-fixed editor
path is viable, but it used different unreleased bytes and cannot substitute for
Run/Debug acceptance against the exact DIST009 candidate.

## Reusable evidence versus regenerated evidence

Reusable without rerunning as a design decision:

- DIST001 release identity and release-only-lineage policy;
- DIST005 `JVM_PLUS_NATIVE` asset architecture;
- Native-first / portable-JVM-fallback roles;
- existing candidate materialization and publication machinery;
- current release platform/runtime policy encoded by the repository;
- BUG012 root-cause/repair ownership and the maintained regression mechanism.

Must be regenerated because candidate bytes changed:

- Native complete admission;
- maintained Native DAP breakpoint/stackTrace regression;
- forced guest Tier-2/JIT admission;
- portable JVM complete admission;
- multi-asset envelope verification;
- real VS Code Run acceptance against the exact candidate;
- real VS Code Debug acceptance against the exact candidate.

No incidental Maven/plugin patch version is promoted to a new DIST009 invariant.
Tool versions remain requirements only where the current repository machinery
itself declares or enforces them.

## Readiness result

```text
DIST009_A_STATUS=READY_FOR_IMPLEMENTATION
CURRENT_MAIN=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
CURRENT_SNAPSHOT_VERSION=0.3.125-SNAPSHOT
BUG012_IN_ANCESTRY=YES
PROPOSED_RELEASE_VERSION=0.3.125
PROPOSED_RELEASE_TAG=v0.3.125
TAG_COLLISION=NO
JVM_PLUS_NATIVE_MACHINERY_REUSABLE=YES
PREPUBLICATION_VSCODE_GATE_REQUIRED=YES
BLOCKERS=NONE
```

## Next execution

To preserve repository ownership boundaries, DIST009-B is executed as two
bounded slices inside the same Issue:

```text
DIST009-B1=materialize/build/validate exact JVM_PLUS_NATIVE candidate
DIST009-B1_REPOSITORY=guillermomolina/protos

DIST009-B2=real VS Code Run/Debug acceptance against exact B1 candidate runtime
DIST009-B2_REPOSITORY=guillermomolina/protos-vscode-extension
```

B1 must stop after producing a frozen candidate identity and all Protos-owned
candidate validation evidence. B2 then consumes that exact candidate runtime;
neither slice publishes the release. Publication remains DIST009-C and requires
the exact candidate to have passed both B1 and B2 plus explicit publication
authorization.
