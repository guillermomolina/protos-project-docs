# I065 — package application explicit Network grant propagation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I065
PROTOS_ISSUE=guillermomolina/protos#668
DECISION_AUTHORITY=D173/guillermomolina/protos#645
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
redefine Protos semantics, select user-facing Network opt-in syntax, or replace
the live GitHub Issue state.

## Published product slice

The package-application I065 plumbing slice is published at:

~~~text
PROTOS_REVISION=c1b8a3f87d8c90a654e19069139191dba4d922b6
COMMIT_SUBJECT=I065: thread Network grant through package application execution
IMPLEMENTATION_VERSION=0.3.208-SNAPSHOT
~~~

The commit changes exactly:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosPackageRunDriver.java
src/test/java/com/guillermomolina/protos/execution/ProtosPackageRunDriverTest.java
~~~

No normative specification file, CLI implementation, Standard Library Network
module, or Network/TCP semantic implementation is changed by this slice.

## Package-run grant carrier

At the exact product revision above,
`ProtosPackageRunDriver` carries the already-established policy-neutral:

~~~text
ProtosWorkspacePackageApplicationExecution.NetworkGrant.NONE
ProtosWorkspacePackageApplicationExecution.NetworkGrant.HOST_NETWORK
~~~

The existing public two-argument route remains Network-less:

~~~text
execute(request, provider)
    -> NetworkGrant.NONE
~~~

An explicit hosting route now accepts the already-selected grant without
inventing CLI, manifest, environment or package syntax.

~~~text
PACKAGE_RUN_EXPLICIT_NETWORK_SELECTION=YES
PACKAGE_RUN_DEFAULT_NETWORK_GRANT=NONE
NEW_NETWORK_GRANT_REPRESENTATION=NO
~~~

## Workspace-only fast path

When exact external requirements are empty, the existing generation-1 path is
preserved. The selected grant is forwarded to
`ProtosWorkspaceRunDriver.execute(request, networkGrant)`.

Selecting `HOST_NETWORK` does not cause provider lookup, package capture, V2
planning or resource-scope construction merely to carry Network authority.

~~~text
WORKSPACE_ONLY_PROVIDER_CONSULTED=NO
WORKSPACE_ONLY_CAPTURE_CREATED=NO
WORKSPACE_ONLY_V2_PLAN_CREATED=NO
WORKSPACE_ONLY_RESOURCE_SCOPE_CREATED=NO
WORKSPACE_ONLY_EXPLICIT_GRANT_FORWARDED=YES
~~~

## Mixed/external-package path

When exact external requirements exist, the existing PLAT048 B-prime package
pipeline remains intact:

~~~text
derive exact requirements
select exact materializations
capture and verify custodies
plan and detach the mixed V2 graph
reconcile the run-owned resource scope
construct the mixed application resolver
execute one application Process
await Process termination
close the resource scope
~~~

Requirement derivation, materialization verification, planning and resource
scope reconciliation receive no Network authority from the application grant.

Only the final application Process receives the selected grant. The existing
`ProtosWorkspacePackageApplicationExecution.executeEntry` boundary provisions
`HOST_NETWORK` from the exact mixed-application Prelude on the same live
`ProtosPolyglotRuntimeHost` that hosts the Process.

~~~text
PACKAGE_TOOL_NETWORK_GRANT=NONE
VERIFICATION_NETWORK_GRANT=NONE
PLANNING_NETWORK_GRANT=NONE
RESOURCE_SCOPE_NETWORK_AUTHORITY=NONE
MIXED_APPLICATION_EXPLICIT_GRANT=YES
EXACT_APPLICATION_PRELUDE_PROVISIONING=YES
SAME_RUNTIME_HOST_PROVISION_AND_HOST=YES
~~~

## Default absence and CLI boundary

The default package route remains `NetworkGrant.NONE`, including mixed
applications.

The CLI remains unchanged and continues to call the default package-run route;
there is no user-facing Network opt-in syntax in this slice.

~~~text
DEFAULT_MIXED_APPLICATION_NETWORK_GRANT=NONE
CLI_NETWORK_GRANT=NONE
CLI_NETWORK_SYNTAX_ADDED=NO
MANIFEST_NETWORK_POLICY_ADDED=NO
ENVIRONMENT_NETWORK_POLICY_ADDED=NO
~~~

## Focused regression evidence

The product commit extends `ProtosPackageRunDriverTest` with focused coverage
for the new seam.

The tests establish that:

- the existing default package route remains Network-less;
- an explicit workspace-only grant survives delegation to the generation-1
  workspace driver without consulting the materialization provider or entering
  F2E2-F2E4 work;
- an explicit mixed-application grant yields a real
  `ProtosNetworkCapabilityValue` delegating to that exact application's
  `Network` prototype;
- explicit `NONE` on the mixed route leaves the bootstrap-local `network`
  slot absent; and
- exact materialization selection, verification count, single application
  Process hosting, Process termination, resource-scope closure and custody
  closure remain intact.

The maintainer reported all local tests passing after publication:

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
MAINTAINER_REPORT="Todos los tests han pasado en local"
PRODUCT_PUBLICATION=PUSHED
~~~

No remote CI success is asserted by this record.

## Authority preservation

This slice introduces none of the D173-forbidden shortcuts:

~~~text
PROCESS_NETWORK_ACCESSOR_ADDED=NO
GLOBAL_NETWORK_ACCESSOR_ADDED=NO
NETWORK_REGISTRY_ADDED=NO
IMPORT_SIDE_EFFECT_NETWORK_ACQUISITION=NO
ACTOR_OR_P_IMPLICIT_NETWORK_PROPAGATION=NO
STDLIB_TCP_FACADE_ADDED=NO
SPEC_CHANGED=NO
NEW_DESIGN_DECISION_REQUIRED=NO
~~~

## Relationship to previous I065 slices

Earlier retained evidence:

~~~text
FRESH_PROCESS_REVISION=c4e108a7e850ff2fc31b52684fb041d21a7c03b6
WORKSPACE_APPLICATION_REVISION=564dc97aacb593826011a8876554d69dd6529faa
PACKAGE_APPLICATION_REVISION=c1b8a3f87d8c90a654e19069139191dba4d922b6
~~~

The sequence now carries explicit Network authority from the low-level fresh
Process seam through workspace application execution and package-backed
application execution while preserving default absence.

## Remaining I065 work

I065 / #668 remains open after this publication.

The issue contract still requires the remaining normal application/session
construction seams to be reconciled where relevant, including standalone
session/direct-file/eval/REPL/debug hosting, without selecting user-facing
Network syntax inside I065.

~~~text
I065_STATUS=OPEN_IN_PROGRESS
SLICE=PACKAGE_APPLICATION_NETWORK_GRANT_PROPAGATION
SLICE_PRODUCT_REVISION=c1b8a3f87d8c90a654e19069139191dba4d922b6
NEXT_BOUNDARY=STANDALONE_SESSION_APPLICATION_NETWORK_GRANT_PROPAGATION
~~~
