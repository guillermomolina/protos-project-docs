# GITHUB018-E — Cross-repository authority propagation closure

Status: **CLOSED on publication**

Live coordination: GitHub `GITHUB018 / #533`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Purpose

GITHUB018 propagates the repository-authority architecture established by
DOC007 across the maintained Protos companion repositories.

The goal is discoverability and consistency, not governance centralization.
Every companion repository now makes the same four-way distinction:

```text
OPERATIONAL_PROTOS_PROJECT_AUTHORITY = guillermomolina/protos
NORMATIVE_PROTOS_AUTHORITY           = guillermomolina/protos/spec
DURABLE_PROTOS_PROJECT_RECORDS       = guillermomolina/protos-project-docs:docs/project/**
LOCAL_PRODUCT_IMPLEMENTATION         = owning companion repository
```

This record closes the ecosystem-wide propagation after publication of the four
product-side policy changes.

## Revisions audited

The authoritative Protos control-plane snapshot observed during the final audit
was:

```text
PROTOS_REVISION=e4701b936c9fb5a0e3b2254b50f4066547ee099f
```

That revision is recorded as the exact source-policy snapshot observed at
closure time. GITHUB018 itself did not require a product implementation commit
in `guillermomolina/protos`; its live coordination authority is Issue `#533`.

The DOC007 durable-record architecture was already closed at:

```text
DOC007_PROJECT_RECORD_REVISION=efd4c6ca24268329d2641cb5bdeb64057df26819
```

The four companion-repository publications are:

```text
PROTOS_BENCHMARKS_REVISION=c3b981305cfc9189de1aa6c894871c3c23fe68a4
PROTOS_WEBSITE_REVISION=a6e83ab128ab9d8af3c878bce28a06bbb1c25668
PROTOS_VSCODE_EXTENSION_REVISION=82759a654f8d438b6ecdee9f86fbf94d050a70a2
PROTOS_DEVCONTAINER_REVISION=e893284d7170aa8edb080841043392317666aa43
```

The final revision containing this record is captured in live Issue `#533`
after publication because a commit cannot contain its own final SHA.

## GITHUB018-A — Protos Benchmarks

Published revision:

```text
c3b981305cfc9189de1aa6c894871c3c23fe68a4
```

The root `AGENTS.md` now states that:

- formal Protos `PERFxxx` live work remains owned by `guillermomolina/protos`;
- normative Protos authority remains with the Protos specification;
- durable non-normative Protos project records live in
  `guillermomolina/protos-project-docs:docs/project/**`;
- benchmark implementation, raw measurements, generated reports/artifacts and
  harness-local reproducibility evidence remain owned by
  `guillermomolina/protos-benchmarks`;
- durable project evidence references exact benchmark revisions or immutable
  artifact identities instead of copying product-local evidence.

Result:

```text
GITHUB018_A_STATUS=CLOSED
BENCHMARK_LOCAL_AUTHORITY=PRESERVED
FORMAL_PERF_LIFECYCLE_DUPLICATED=NO
RESULT=PASS
```

## GITHUB018-B — Protos Website

Published revision:

```text
a6e83ab128ab9d8af3c878bce28a06bbb1c25668
```

The root `AGENTS.md` now states that:

- formal Protos work remains operationally governed from
  `guillermomolina/protos`;
- normative language and Standard Library authority remains
  `guillermomolina/protos/spec`;
- durable non-normative Protos project records live in
  `guillermomolina/protos-project-docs:docs/project/**`;
- website-local implementation, presentation, deployment behavior and
  website-local Issues remain owned by `guillermomolina/protos-website`;
- canonical Protos and VS Code inputs remain consumed through exact revision
  locks rather than being maintained as forks;
- formal Protos work is not duplicated as an independent website lifecycle.

Result:

```text
GITHUB018_B_STATUS=CLOSED
WEBSITE_LOCAL_AUTHORITY=PRESERVED
CANONICAL_SOURCE_FORK_CREATED=NO
RESULT=PASS
```

## GITHUB018-C — Protos VS Code extension

Published revision:

```text
82759a654f8d438b6ecdee9f86fbf94d050a70a2
```

The root `AGENTS.md` now distinguishes:

- formal Protos project work governed from `guillermomolina/protos`;
- normative language and Standard Library semantics under
  `guillermomolina/protos/spec`;
- durable formal Protos project records under
  `guillermomolina/protos-project-docs:docs/project/**`;
- extension-local source, packaging, VSIX/release artifacts, evidence,
  documentation and `BUG` / `CI` / `DOC` / `REL` Issue families remaining
  owned by `guillermomolina/protos-vscode-extension`.

Result:

```text
GITHUB018_C_STATUS=CLOSED
EXTENSION_LOCAL_AUTHORITY=PRESERVED
LOCAL_ISSUE_FAMILIES_PRESERVED=YES
FORMAL_PROTOS_LIFECYCLE_DUPLICATED=NO
RESULT=PASS
```

## GITHUB018-D — Protos Dev Container

Published revision:

```text
e893284d7170aa8edb080841043392317666aa43
```

The repository now has a root `AGENTS.md` that explicitly declares:

```text
OPERATIONAL_PROTOS_PROJECT_AUTHORITY = guillermomolina/protos
NORMATIVE_PROTOS_AUTHORITY           = guillermomolina/protos/spec
DURABLE_PROTOS_PROJECT_RECORDS       = guillermomolina/protos-project-docs:docs/project/**
DEVCONTAINER_PRODUCT_AUTHORITY       = guillermomolina/protos-devcontainer
```

The devcontainer remains authoritative for its configuration, version pins,
bootstrap behavior, curated environment snapshots, documentation and local
build/release evidence.

Result:

```text
GITHUB018_D_STATUS=CLOSED
DEVCONTAINER_LOCAL_AUTHORITY=PRESERVED
INDEPENDENT_PROTOS_SEMANTIC_AUTHORITY=NO
RESULT=PASS
```

## Cross-repository authority audit

The published policies were re-read at their exact revisions rather than relying
only on moving `main` branches.

Across all four repositories:

```text
FORMAL_PROTOS_OPERATIONAL_AUTHORITY=guillermomolina/protos
NORMATIVE_PROTOS_AUTHORITY=guillermomolina/protos/spec
DURABLE_PROTOS_PROJECT_RECORD_AUTHORITY=guillermomolina/protos-project-docs
PRODUCT_IMPLEMENTATION_AUTHORITY=OWNING_REPOSITORY
```

No companion repository claims that `protos-project-docs` owns:

- live formal Protos Issue state;
- formal identifier allocation;
- scheduling or project-owner approval;
- normative Protos semantics;
- companion-product implementation;
- product-local Issues;
- product-local release artifacts;
- raw benchmark measurements;
- website deployment state; or
- devcontainer build state.

Result:

```text
CONTROL_PLANE_CENTRALIZATION_EXPANDED=NO
NORMATIVE_AUTHORITY_MOVED=NO
LOCAL_PRODUCT_AUTHORITY_CENTRALIZED=NO
FORMAL_ID_FEDERATION_CREATED=NO
ISSUE_MIGRATION_REQUIRED=NO
RESULT=PASS
```

## Fresh-agent discoverability

A fresh agent starting in each companion repository can now mechanically
discover both sides of the project topology:

1. where formal Protos operational authority lives; and
2. where durable non-normative Protos project records live.

The same policy also identifies the repository-local authority that must not be
silently moved into either central repository.

Result:

```text
PROTOS_BENCHMARKS_DISCOVERABILITY=PASS
PROTOS_WEBSITE_DISCOVERABILITY=PASS
PROTOS_VSCODE_EXTENSION_DISCOVERABILITY=PASS
PROTOS_DEVCONTAINER_DISCOVERABILITY=PASS
RESULT=PASS
```

## Publication contract

GITHUB018 is an ecosystem-wide consumer of the DOC007 cross-repository
publication model.

The product-side policy commits published first. This durable closure record
then captures their exact revisions.

The closure tuple is:

```text
PROTOS_REVISION=e4701b936c9fb5a0e3b2254b50f4066547ee099f
PROTOS_BENCHMARKS_REVISION=c3b981305cfc9189de1aa6c894871c3c23fe68a4
PROTOS_WEBSITE_REVISION=a6e83ab128ab9d8af3c878bce28a06bbb1c25668
PROTOS_VSCODE_EXTENSION_REVISION=82759a654f8d438b6ecdee9f86fbf94d050a70a2
PROTOS_DEVCONTAINER_REVISION=e893284d7170aa8edb080841043392317666aa43
PROJECT_RECORD_REVISION=CAPTURE_IN_LIVE_ISSUE_AFTER_PUBLICATION
CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS
```

A moving `main` branch is not used as historical closure evidence.

## Final result

```text
GITHUB018_A_STATUS=CLOSED
GITHUB018_B_STATUS=CLOSED
GITHUB018_C_STATUS=CLOSED
GITHUB018_D_STATUS=CLOSED
GITHUB018_E_STATUS=CLOSED_ON_PUBLICATION

COMPANION_AUTHORITY_PROPAGATION=PASS
PRODUCT_LOCAL_AUTHORITY_PRESERVED=PASS
FORMAL_PROTOS_OPERATIONAL_AUTHORITY=PASS
NORMATIVE_PROTOS_AUTHORITY=PASS
DURABLE_PROJECT_RECORD_DISCOVERABILITY=PASS
CROSS_REFERENCES=PASS
REQUIRED_DURABLE_PUBLICATION=PASS

GITHUB018_STATUS=CLOSED_ON_E_PUBLICATION_AND_LIVE_ISSUE_CLOSURE
```

After this record is published and its exact `PROJECT_RECORD_REVISION` is
captured in live Issue `#533`, GITHUB018 satisfies its closure criteria and the
Issue may be closed as completed.
