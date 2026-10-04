# TOOL001-F2E5 — Provider-backed mixed package run composition

Status: **PUBLISHED CHECKPOINT — F2E5 remains IN_PROGRESS**

Product revision:

~~~text
REPOSITORY=guillermomolina/protos
REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
SUBJECT=TOOL001-F2E5: add provider-backed mixed package run composition
VERSION=0.3.185-SNAPSHOT
ISSUE=guillermomolina/protos#93
DECISION=PLAT048 Candidate B′
LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

## Published surface

The revision adds:

- `ProtosExactPackageMaterializationProvider`, a run-scoped host seam from one
  complete exact external identity to one already-present selected local root;
- `ProtosPackageRunDriver`, the CLI-neutral F2E5 composition driver;
- `VerifiedExternalPackage.fromIdentity`, preserving exact Registry/Git
  identity fields when constructing F2E3 verified planning inputs;
- a shared `executeEntry` application bootstrap used by generation-1 and mixed
  package-backed execution without changing workspace semantics;
- focused integration and lifecycle tests for the new composition.

The product revision changes exactly these paths:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosExactPackageMaterializationProvider.java
src/main/java/com/guillermomolina/protos/execution/ProtosExternalPackagePlanningPreflight.java
src/main/java/com/guillermomolina/protos/execution/ProtosPackageRunDriver.java
src/main/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageApplicationExecution.java
src/test/java/com/guillermomolina/protos/execution/ProtosPackageRunDriverTest.java
~~~

## PLAT048 B′ composition

The published host path is:

~~~text
Package Tool exact requirements
    -> List<ProtosExactExternalPackageIdentity>
    -> one exact provider lookup per requirement, in lock order
    -> one F2E2 capture + ContentIdentity verification per selected root
    -> F2E3 mixed V2 raw planning
    -> F2E4 defensive V2 detach
    -> F2E4 exact custody/resource-scope reconciliation
    -> mixed V2 application execution
    -> application Process TERMINATED
    -> resource scope close
~~~

The lookup key remains the complete exact identity:

~~~text
kind
PackageId
exact ReleaseVersion OR exact Git revision
ContentIdentity
~~~

The provider does not solve, fetch, verify, write, scan alternatives or fall
back. Content verification remains exclusively F2E2.

## Workspace fast path

When the exact-requirements preflight returns no external requirements,
`ProtosPackageRunDriver` delegates to the existing generation-1
`ProtosWorkspaceRunDriver`. No materialization-provider call, capture, V2 plan,
external resource scope or mixed resolver is created.

## Ownership and failure evidence

Before successful F2E4 reconciliation, the driver owns every successful verified
custody.

The published tests exercise:

- provider miss after an earlier successful verification;
- wrong materialization causing Nth F2E2 verification failure;
- planning failure after all required verification;
- reconciliation failure with no ownership transfer;
- successful mixed execution proving the application Process is TERMINATED
  before the resource scope closes.

Every pre-reconciliation failure closes all previously successful custodies and
does not start an application Process. Successful reconciliation transfers
ownership to `ProtosExternalPackageResourceScope`.

The successful integration test also proves exact-identity separation for:

- the same PackageId at distinct exact versions; and
- distinct Registry/Git identities sharing one ContentIdentity.

## Validation provenance

The maintainer reported:

~~~text
Todos los tests han pasado en local
~~~

This record preserves that report as maintainer validation. It does not claim
independent CI execution by the documentation publisher.

## Remaining F2E5 boundary

This revision deliberately leaves the following for the final bounded F2E5
slice:

~~~text
public protos run bootstrap
    -> implementation-private read-only local materialization backend
    -> ProtosExactPackageMaterializationProvider
    -> ProtosPackageRunDriver
    -> transparent public CLI mixed execution
    -> final F2E5 / F2 external-execution closure reconciliation
~~~

The final slice must preserve PLAT048 B′:

- no new CLI flag;
- no new Protos-specific environment variable;
- no public configuration surface;
- no canonical public store layout;
- no ambient scanning or partial-identity lookup;
- no solve, fetch, network, credentials, store writes, GC or lock mutation;
- generation-1 workspace-only behavior remains unchanged.

Therefore `TOOL001-F2E5 / #93` remains **IN_PROGRESS** at this checkpoint.
