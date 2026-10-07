# TOOL001-F2E5 exact external requirements Package Tool preflight

Date: 2026-10-04

Nature: immutable implementation/publication checkpoint; non-normative

Formal owner: `TOOL001-F2E5 / guillermomolina/protos#93`

Ratified platform authority: `PLAT048 / guillermomolina/protos#789`,
Candidate B′.

## Exact product publication

~~~text
PROTOS_REVISION=eff751f7cde2009feb9b91bc5d2cd31509f3ef9e
COMMIT_SUBJECT=TOOL001-F2E5: add exact external requirements Package Tool preflight (PLAT048 B')
PRODUCT_VERSION=0.3.183-SNAPSHOT
PUBLICATION_STATUS=PUSHED
LOCAL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

The maintainer reports that all local tests passed after publication. This record
does not convert that report into an independent CI claim.

An unrelated PERF030-G commit
`ee066755219dfbe6ac7dd69a68b3ac76492bf7ee` was already present immediately
before this TOOL001 publication. The exact TOOL001 commit itself changes only
the six paths listed below.

## Published changed paths

~~~text
CHANGELOG.md
pom.xml
protos/tools/package/ExecutionPlan.protos
src/main/java/com/guillermomolina/protos/execution/ProtosExactExternalRequirementsPreflight.java
src/main/java/com/guillermomolina/protos/execution/ProtosPackageExecutionPlanV2Adapter.java
src/test/java/com/guillermomolina/protos/execution/ProtosExactExternalRequirementsPreflightTest.java
~~~

No specification file, CLI implementation, public-run driver, materialization
provider, store backend or application-execution path changed in this slice.

## Implemented boundary

The published Package Tool operation is:

~~~text
ExecutionPlan.exactExternalRequirements(projectTreeFilesystem)
~~~

It derives the external requirements from the current project and canonical
lock before any external package materialization is selected or captured.

The operation validates the current lock against the current ResolutionRoot
through the same shared generation-2 authority used by later V2 planning:

- resolution-input/header agreement;
- exact root agreement;
- workspace membership;
- workspace-declared dependency agreement;
- exact registry/Git locked-node indexing.

It returns one inert value per locked external node:

~~~text
{
    ref: exact registry/Git NodeRef
    content: exact ContentIdentity
}
~~~

It intentionally does not open external manifests at this stage. External
exports and dependencies declared by immutable external manifests remain owned
by the later F2E3 planning pass over the same F2E2-verified captures.

## Shared Package Tool refactor

The publication factors the current-lock/V2 validation into shared helpers:

~~~text
loadCurrentV2Lock(...)
indexLockedExternalNodes(...)
collectV2WorkspaceDependencies(...)
~~~

`buildV2FromVerifiedCaptures(...)` now reuses those helpers. Its external
planning policy remains unchanged.

This avoids creating a parallel Java interpretation of `protos.lock` and keeps
Package Tool policy in bundled Protos code as required by PLAT048 B′.

## Host preflight boundary

The new host-side
`ProtosExactExternalRequirementsPreflight`:

1. opens a fresh RuntimeHost;
2. creates a read-only confined project Filesystem;
3. creates a fresh Package Tool Process;
4. invokes `ExecutionPlan.exactExternalRequirements`;
5. defensively detaches every requirement;
6. terminates the Package Tool Process before returning.

The detached result is:

~~~text
List<ProtosExactExternalPackageIdentity>
~~~

and contains only host-inert exact identity data.

The detach path reuses the exact generation-2 ref and ContentIdentity shape
parsers from `ProtosPackageExecutionPlanV2Adapter`.

## Exact identity preservation

The result preserves the complete PLAT048 lookup key:

### Registry

~~~text
PackageId
canonical exact ReleaseVersion text
ContentIdentity
~~~

### Git

~~~text
PackageId
exact revision
ContentIdentity
~~~

The published tests prove that:

- two versions of one PackageId remain distinct;
- registry and Git identities remain distinct;
- distinct exact identities sharing one ContentIdentity remain distinct;
- deduplication never occurs by PackageId alone or ContentIdentity alone.

Duplicate rejection is by the complete exact external identity/NodeRef shape
required by the lock model.

## Authority isolation

The detached requirements contain no:

~~~text
Path
locator
registry authority endpoint
fetch URL
mirror
store/cache location
Filesystem
custody
resolver
host handle
guest object
~~~

The Package Tool Process is terminated before the preflight call returns, and
its root Filesystem authority is not retained.

Therefore this slice establishes only:

~~~text
project/lock authority
    ->
complete exact external identities
~~~

It does not yet establish:

~~~text
exact external identity
    ->
physical materialized root
~~~

That remains the next PLAT048 B′ implementation boundary.

## Validation coverage published

`ProtosExactExternalRequirementsPreflightTest` covers at least:

- workspace-only project -> empty requirement list;
- mixed registry/Git locked graph;
- multiple versions of one PackageId;
- same ContentIdentity under distinct logical identities;
- Package Tool Process termination before return;
- stale project lock failure;
- workspace membership mismatch failure;
- current-root/lock mismatch failure;
- exact defensive detached shape;
- invalid/foreign fields and unknown kinds fail closed;
- duplicate exact requirement rejection;
- host-inert detached result.

The maintainer reports the complete local test set passed after the commit was
pushed.

## PLAT048 consistency

~~~text
PACKAGE_TOOL_DERIVES_EXACT_REQUIREMENTS=YES
JAVA_REINTERPRETS_LOCK_POLICY=NO
COMPLETE_EXACT_EXTERNAL_IDENTITY_PRESERVED=YES
PHYSICAL_MATERIALIZATION_AUTHORITY_ADDED=NO
MATERIALIZATION_PROVIDER_ADDED=NO
STORE_LAYOUT_SELECTED=NO
AMBIENT_STORE_LOOKUP_ADDED=NO
FETCH_OR_NETWORK_ADDED=NO
STORE_WRITE_OR_GC_ADDED=NO
PUBLIC_CLI_CHANGED=NO
PUBLIC_RUN_BEHAVIOR_CHANGED=NO
V1_WORKSPACE_ONLY_ROUTE_CHANGED=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

D053, D056, D057 and PLAT012 remain unchanged.

## Remaining F2E5 work

F2E5 remains open.

The next bounded implementation must consume these exact requirements through
the ratified PLAT048 B′ host materialization authority and compose successful
verified material with the already-published F2E2/F2E3/F2E4 pipeline.

At minimum the remaining lifecycle is:

~~~text
exact external requirements
    ->
run-scoped exact materialization provider
    ->
already-present selected roots
    ->
F2E2 capture + ContentIdentity verification
    ->
F2E3 V2 planning
    ->
F2E4 detach + exact scope reconciliation
    ->
mixed application execution
    ->
application Process termination
    ->
scope close
~~~

Workspace-only public runs must continue using the existing generation-1 route.

The remaining work must not introduce solving, fetch, network, credentials,
store writes, GC, ambient scanning, PackageId/version-only lookup, lock mutation
or application-visible store authority.

## Coordination result

~~~text
TOOL001_F2E5_STATUS=IN_PROGRESS
EXACT_REQUIREMENTS_SLICE=CLOSED
EXACT_REQUIREMENTS_REVISION=eff751f7cde2009feb9b91bc5d2cd31509f3ef9e
LOCAL_TESTS=PASS_MAINTAINER_REPORTED

NEXT_BOUNDARY=EXACT_MATERIALIZATION_PROVIDER_AND_PUBLIC_RUN_COMPOSITION
PLAT048=RATIFIED
IMPLEMENTATION_AUTHORIZED=YES
~~~

## References

- `guillermomolina/protos#93` — TOOL001-F2E5 live implementation owner.
- `guillermomolina/protos#47` — TOOL001 root coordination.
- `guillermomolina/protos#412` — TOOL001-F2E phase coordination.
- `guillermomolina/protos#789` — ratified PLAT048 Candidate B′.
- `docs/project/decisions/platform/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_AUTHORITY_BOUNDARY.md`.
- `docs/project/evidence/PLAT048/PLAT048_PUBLIC_RUN_EXTERNAL_MATERIALIZATION_DECISION_EVIDENCE.md`.
- `docs/project/evidence/TOOL001/TOOL001_F2E5_PUBLIC_RUN_MATERIALIZATION_LIFECYCLE_AUDIT.md`.
