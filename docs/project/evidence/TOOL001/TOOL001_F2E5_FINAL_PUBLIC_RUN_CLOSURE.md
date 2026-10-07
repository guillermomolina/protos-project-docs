# TOOL001-F2E5 — Final public-run external execution closure

Status: CLOSED

Nature: retained non-normative implementation/validation evidence

Owning work:
- TOOL001-F2E5 / `guillermomolina/protos#93`
- TOOL001-F2E / `guillermomolina/protos#412`
- TOOL001-F / `guillermomolina/protos#411`
- TOOL001 / `guillermomolina/protos#47`

Ratified architecture:
- PLAT048 / `guillermomolina/protos#789`
- Candidate B′ remains unchanged.

## Published product revision

```text
REPOSITORY=guillermomolina/protos
PROTOS_REVISION=cd710a0cff2768691c6b654b1ab85fbb96d9d6ec
SUBJECT=TOOL001-F2E5: complete public external package run closure
VERSION=0.3.187-SNAPSHOT
PARENT_REVISION=873c9a6ac466ddcae7eedfd7d23ed38ab17c569f
LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

The final product publication adds the implementation-private read-only local
exact materialization backend and routes public
`protos run <entry> [args...]` through the already-published
`ProtosPackageRunDriver`.

The backend lookup key is the complete exact external package identity:

```text
(kind, PackageId, exact version/revision, ContentIdentity)
```

It converts that tuple into one deterministic opaque implementation-private
location. Package-controlled identity text is not used as an unconstrained
filesystem path component. Selection performs no directory enumeration,
partial-identity matching, alternative lookup or ambient cache scan.

If the exact already-present root is absent, selection fails closed. The backend
does not create, fetch, repair or mutate materializations.

## Final public execution chain

```text
project/lock
    -> bundled Package Tool exact external requirements
    -> complete exact identity
    -> private host exact local materialization selection
    -> F2E2 capture + ContentIdentity verification
    -> F2E3 mixed V2 planning
    -> F2E4 defensive detach + exact custody reconciliation
    -> mixed external/workspace module resolution
    -> application Process
    -> application Process TERMINATED
    -> verified-custody resource scope close
```

The selected physical root remains untrusted until unchanged F2E2 capture and
ContentIdentity verification. Physical store/root authority does not enter the
PackageExecutionPlan or application Process.

Workspace-only public runs retain the generation-1
`ProtosWorkspaceRunDriver` route and do not consult the external materialization
provider.

## Validation evidence

The maintainer reported the following local validation while implementing and
publishing the final slice:

- `make compile`: BUILD SUCCESS;
- focused
  `mvn -Dtest=ProtosWorkspaceRunCliTest,ProtosPackageRunDriverTest test`:
  13 tests, 0 failures, 0 errors, 0 skipped after rebasing onto product parent
  `873c9a6ac466ddcae7eedfd7d23ed38ab17c569f`;
- repeated `git diff --check` checks were clean during the handoff;
- final integrated `make test`: PASS;
- the product commit was pushed to `main` as
  `cd710a0cff2768691c6b654b1ab85fbb96d9d6ec`.

This evidence records maintainer-reported local results. The project-docs
publication did not independently execute Maven, Protos tests or CI.

## Focused closure properties

Published focused coverage verifies:

- full exact-identity materialization-key separation;
- no fallback from one exact version/revision/kind to another;
- missing exact materialization fails closed without creating the store;
- the lockfile remains unchanged on a missing materialization;
- workspace-only public execution performs zero external-provider lookups; and
- a public mixed run imports and executes source from an exact verified external
  package while preserving application arguments and standard-stream behavior.

The pre-existing `ProtosPackageRunDriverTest` coverage remains green and retains
provider lookup multiplicity, verification/planning/reconciliation failure
cleanup, exact registry/Git identity handling and Process-termination-before-
scope-close evidence.

## Scope intentionally not claimed

This closure does not add or define:

- dependency solving or lock update;
- registry discovery or remote fetch;
- network transport or credentials;
- package-store writes, installation, repair or garbage collection;
- a public/canonical cache or store layout;
- a new CLI option, Protos environment variable or public configuration input;
- vendor/system stores or multi-store composition; or
- package publication.

Those are future separately scoped capabilities if allocated. They are not
residual work required to close F2E5 or the current bounded TOOL001 lifecycle.

## Closure reconciliation

With this publication:

```text
TOOL001-F2E5 = CLOSED
TOOL001-F2E  = CLOSED
TOOL001-F2   = CLOSED
TOOL001-F    = CLOSED
TOOL001      = CLOSED
PLAT048      = RATIFIED / CLOSED (unchanged)
```

The corresponding GitHub live-coordination issues may therefore be closed in
dependency order after this retained evidence is published.
