# I065 — workspace application explicit Network grant propagation

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

The workspace-application I065 plumbing slice is published at:

~~~text
PROTOS_REVISION=564dc97aacb593826011a8876554d69dd6529faa
COMMIT_SUBJECT=I065: thread Network grant through workspace application execution
IMPLEMENTATION_VERSION=0.3.206-SNAPSHOT
~~~

The commit changes exactly:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosPackageRunDriver.java
src/main/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageApplicationExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosWorkspaceRunDriver.java
src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageApplicationExecutionTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosWorkspacePackageAuthorityIsolationIntegrationTest.java
src/test/resources/workspace-package-application-process/NetworkGrant.protos
src/test/resources/workspace-package-authority-isolation/NetworkGrant.protos
~~~

No normative specification file or Standard Library Network module is changed by
this slice.

## Established workspace hosting seam

At the exact product revision above,
`ProtosWorkspacePackageApplicationExecution` defines the policy-neutral hosting
selection:

~~~text
NetworkGrant.NONE
NetworkGrant.HOST_NETWORK
~~~

The default public application route selects `NONE`.

When an owning host explicitly selects `HOST_NETWORK`,
`ProtosWorkspacePackageApplicationExecution.executeEntry` first creates the
application Prelude and then calls:

~~~text
runtimeHost.provisionHostNetwork(prelude)
~~~

using that exact application Prelude. The resulting
`ProtosNetworkCapabilityValue` is supplied to the existing
`ProtosStandaloneProcessBootstrap` Network parameter.

The same live `ProtosPolyglotRuntimeHost` then hosts the application Process.
The capability therefore delegates to the exact application Network prototype
and its concrete host authority remains owned by the RuntimeHost whose lifetime
covers the Process.

~~~text
WORKSPACE_NETWORK_SELECTION_SEAM=YES
POLICY_NEUTRAL_SELECTION=YES
EXACT_APPLICATION_PRELUDE_PROVISIONING=YES
SAME_RUNTIME_HOST_PROVISION_AND_HOST=YES
BOOTSTRAP_NETWORK_FORWARDING=YES
~~~

## Workspace-run ownership and default absence

`ProtosWorkspaceRunDriver.execute(Request)` preserves the existing default and
delegates with `NetworkGrant.NONE`.

The explicit overload carries the already-selected hosting choice while one
RuntimeHost owns the complete run. Package Tool preflight executes before the
application on that same host but is not granted Network authority by this
selection.

The no-grant path does not initialize the RuntimeHost Network plane merely
because the explicit seam exists.

~~~text
DEFAULT_WORKSPACE_NETWORK_GRANT=NONE
PREFLIGHT_NETWORK_GRANT=NONE
NO_GRANT_NETWORK_HOST_INITIALIZATION=NO
AMBIENT_NETWORK_AUTHORITY_ADDED=NO
~~~

## External-package and CLI boundary

The external-package application path in `ProtosPackageRunDriver` is explicitly
adapted to call the shared application bootstrap with `NetworkGrant.NONE`.
This slice therefore does not silently grant Network to mixed/external-package
applications.

The published changelog also records that the CLI remains Network-less. No
command-line switch, manifest policy, environment convention or other
user-facing selection spelling is introduced.

~~~text
EXTERNAL_PACKAGE_DEFAULT_NETWORK_GRANT=NONE
CLI_NETWORK_GRANT=NONE
CLI_NETWORK_SYNTAX_ADDED=NO
MANIFEST_NETWORK_POLICY_ADDED=NO
~~~

## Authority preservation

This slice introduces none of the D173-forbidden authority shortcuts:

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

## Regression evidence

The published product commit adds focused workspace-application tests that
cover both default absence and explicit grant behavior.

The direct workspace application test verifies that:

- the default route cannot resolve the bootstrap-local `network` binding;
- `NetworkGrant.NONE` does not initialize the RuntimeHost Network plane;
- `NetworkGrant.HOST_NETWORK` produces a real
  `ProtosNetworkCapabilityValue`;
- that capability delegates to the exact application `Network` prototype; and
- the hosting RuntimeHost remains live through Process execution and teardown.

The workspace authority-isolation integration coverage verifies both the
networkless default route and explicit-grant route through
`ProtosWorkspaceRunDriver`.

The maintainer reported all local tests passing after publication:

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
MAINTAINER_REPORT="Todos los tests han pasado en local"
PRODUCT_PUBLICATION=PUSHED
~~~

No remote CI success is asserted by this record.

## Relationship to the previous I065 slice

The earlier evidence remains at:

~~~text
docs/project/evidence/I065/I065_FRESH_PROCESS_NETWORK_GRANT_SLICE.md
PROTOS_REVISION=c4e108a7e850ff2fc31b52684fb041d21a7c03b6
~~~

That slice established the lower-level fresh-Process Network capability carrier.
This publication extends the same D173 authority model through the workspace
application hosting path without changing the prior contract.

## Remaining I065 work

I065 / #668 remains open after this slice.

The current product state deliberately leaves the external-package route on
`NetworkGrant.NONE`, and D173/I065 still requires the remaining normal
application/hosting entry seams to be reconciled where relevant without choosing
user-facing CLI/package syntax inside I065.

The next bounded implementation should start from the current product HEAD and
determine the smallest policy-neutral propagation seam for the package-backed
public application route, then reconcile any remaining session/debug hosting
paths required by #668.

~~~text
I065_STATUS=OPEN_IN_PROGRESS
SLICE=WORKSPACE_APPLICATION_NETWORK_GRANT_PROPAGATION
SLICE_PRODUCT_REVISION=564dc97aacb593826011a8876554d69dd6529faa
NEXT_BOUNDARY=PACKAGE_BACKED_APPLICATION_NETWORK_GRANT_PROPAGATION
LATER_RECONCILIATION=SESSION_DEBUG_HOSTING_AS_REQUIRED
~~~
