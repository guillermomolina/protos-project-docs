# DIST007 retirement and DIST011 successor allocation

Status: **PASS / DIST007 CLOSURE READY / DIST011 ALLOCATED**

This durable, non-normative record retains the coordination evidence for retiring
DIST007 and replacing its historical portable-JVM-only publication plan with the
current JVM_PLUS_NATIVE publication work owned by DIST011.

## Ownership

```text
DATE=2026-10-02
RETIRED_WORK_ITEM=DIST007
RETIRED_GITHUB_ISSUE=guillermomolina/protos#737
SUCCESSOR_WORK_ITEM=DIST011
SUCCESSOR_GITHUB_ISSUE=guillermomolina/protos#774
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
```

## Why DIST007 is retired instead of rewritten

DIST007 was opened with an intentionally narrow release contract:

```text
RELEASE_MODEL=PORTABLE_JVM_ONLY
NATIVE_IMAGE_REQUIRED=NO
SELF_CONTAINED_CLAIM=NO
```

Its purpose was to publish a portable JVM prerelease for external consumers,
including the then-pending DIST006-B2 VS Code acceptance path. Later work
changed the release-engineering reality without changing DIST007's historical
identity:

- DIST005 selected and implemented the `JVM_PLUS_NATIVE` artifact model;
- DIST005 published and verified `v0.3.116` with Native plus portable JVM assets;
- DIST009 reused that machinery, added the exact-candidate consumer gate needed
  for BUG012/DIST006-B2, and published and verified `v0.3.139`;
- DIST007 never selected an exact baseline/version, never materialized a release
  candidate, and never published a tag or GitHub Release.

Rewriting DIST007 to require Native plus JVM would therefore change the meaning
of an established formal work item and contradict its retained coordination
history. The historical work item is retired instead.

## Current release authority

The current verified public release at this transition is DIST009's
JVM_PLUS_NATIVE prerelease:

```text
LATEST_VERIFIED_PUBLIC_RELEASE=v0.3.139
DIST009_STATUS=COMPLETE
DIST009_CANDIDATE_SOURCE_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
DIST009_PROJECT_RECORD_REVISION=6302bb3663c0eedefbddd5840764b2c42201c6f8
POST_PUBLICATION_VERIFICATION=PASS
```

The ratified Native capability boundary remains:

```text
NATIVE_IMAGE=SUPPORTED
NATIVE_EXECUTION=INTERPRETER_ONLY
NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579
```

No language or Standard Library semantics are changed by this coordination
transition.

## Observed current product state

At successor allocation time, current `guillermomolina/protos` `main` was
observed as:

```text
PROTOS_MAIN_OBSERVED_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
PROTOS_MAIN_OBSERVED_VERSION=0.3.143-SNAPSHOT
```

These values are observation evidence only. They do not select the DIST011
release baseline. DIST011-A must re-read authoritative current state and prepare
the exact baseline-selection packet at execution time.

## DIST011 allocation

The successor was allocated as:

```text
WORK_ITEM=DIST011
GITHUB_ISSUE=guillermomolina/protos#774
TITLE=DIST011 — Publish the next current JVM_PLUS_NATIVE Protos prerelease
FAMILY=family:DIST
STATUS=status:ready
FORMAL_IDENTIFIER_UNIQUE=PASS
NATIVE_PARENT=NOT_APPLICABLE
EFFECTIVE_PRIORITY=INTENTIONALLY_UNSET
```

DIST011 deliberately starts without an exact release baseline, version or tag.
Its first slice is investigation only:

```text
NEXT_SLICE=DIST011-A
NEXT_SLICE_TITLE=release readiness and exact baseline-selection packet
WORK_TYPE=INVESTIGATION
COMMAND_EXECUTION=NONE
```

DIST011-A must establish the then-current `V-SNAPSHOT` baseline, derive `V` and
`vV`, verify collision/blocker state, identify the exact candidate gates, and
stop for explicit project-owner selection before any candidate is materialized.

## DIST007 closure classification

DIST007 is ready to close as superseded/not planned rather than completed:

```text
DIST007_EXACT_BASELINE_SELECTED=NO
DIST007_CANDIDATE_MATERIALIZED=NO
DIST007_RELEASE_PUBLISHED=NO
DIST007_ORIGINAL_CONSUMER_NEED=SUPERSEDED_BY_DIST009
DIST007_RELEASE_MODEL=SUPERSEDED_BY_JVM_PLUS_NATIVE
DIST007_FINAL_CLASSIFICATION=SUPERSEDED_NOT_PLANNED
```

The successor does not inherit a stale DIST007 candidate because none existed.
Historical DIST007 comments and scope remain unchanged as evidence.

## Scope and mutation accounting

```text
PROTOS_REPOSITORY_FILE_CHANGES=NONE
SPECIFICATION_CHANGED=NO
RELEASE_PUBLISHED_BY_THIS_TRANSITION=NO
DIST007_HISTORY_REWRITTEN=NO
DIST011_BASELINE_SELECTED=NO
DIST011_CANDIDATE_CREATED=NO
```

Live GitHub coordination mutations owned by this transition are limited to:

- allocation of DIST011 / `guillermomolina/protos#774`;
- closure of DIST007 / `guillermomolina/protos#737` after this durable record is
  published;
- compact cross-reference comments on the affected Issues.

## Closure/publication gate

```text
DIST007_ISSUE_CLOSURE_COMMENT=REQUIRED_AFTER_THIS_PUBLICATION
CLOSURE_EVIDENCE_IDENTIFIED=PASS
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=PENDING_UNTIL_THIS_RECORD_IS_PUBLISHED

DIST011_FORMAL_IDENTIFIER_UNIQUE=PASS
DIST011_CANONICAL_STATUS=PASS
DIST011_ASSIGNEE_INVARIANT=PASS
DIST011_NATIVE_PARENT=NOT_APPLICABLE
DIST011_EFFECTIVE_PRIORITY=INTENTIONALLY_UNSET
DIST011_DECISION_APPROVAL_PROVENANCE=NOT_APPLICABLE
DIST011_DECISION_INVARIANT_CONSISTENCY=NOT_APPLICABLE
```

After this record is published and re-read at its exact project-record revision,
DIST007 may be closed with `state_reason=not_planned`, and both Issues may record
the exact durable revision as the transition authority.
