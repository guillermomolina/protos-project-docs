# I065 — fresh Process explicit Network grant slice

Date: 2026-10-03

## Work identity

~~~text
WORK_ITEM=I065
PROTOS_ISSUE=guillermomolina/protos#668
DECISION_AUTHORITY=D173/guillermomolina/protos#645
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence. It does not
redefine Protos semantics, choose user-facing Network-selection syntax, or
replace the live GitHub Issue state.

## Published product slice

The first I065 plumbing slice is published at:

~~~text
PROTOS_REVISION=c4e108a7e850ff2fc31b52684fb041d21a7c03b6
PROTOS_PARENT=5b2c7d5baa402677f8fe6bf2ca3a19859d4bc475
COMMIT_SUBJECT=I065: thread Network grant through fresh Process execution
IMPLEMENTATION_VERSION=0.3.171-SNAPSHOT
~~~

The commit changes exactly:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosCapturedProcessExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosFreshProcessExecutor.java
src/test/java/com/guillermomolina/protos/execution/ProtosFreshProcessExecutorTest.java
~~~

No normative specification file, CLI implementation, workspace-run driver, or
Standard Library Network module is changed by this slice.

## Established hosting seam

At the exact product revision above,
`ProtosFreshProcessExecutor.Request` carries an optional
`ProtosNetworkCapabilityValue defaultNetwork`.

The caller-hosted execution path forwards that exact value to the existing
`ProtosStandaloneProcessBootstrap.create(..., defaultFilesystem, defaultNetwork)`
contract. The executor does not provision Network itself.

The temporary-host convenience overload rejects any request carrying a non-null
Network grant before guest Process execution. This preserves the D173 lifetime
invariant that a production Network created by
`runtimeHost.provisionHostNetwork(prelude)` is used only while the same owning
`ProtosPolyglotRuntimeHost` hosts the Process.

~~~text
FRESH_PROCESS_REQUEST_NETWORK_GRANT=YES
EXPLICIT_HOST_NETWORK_FORWARDING=YES
TEMP_HOST_NON_NULL_NETWORK_REJECTED=YES
SAME_RUNTIME_HOST_LIFETIME_PRESERVED=YES
~~~

## Authority preservation

The no-grant path remains unchanged: the initial application context has no
bootstrap-local `network` slot merely because the plumbing exists.

`ProtosCapturedProcessExecution` explicitly passes `null` for the new grant
field, so captured Tool/Test Process execution remains Network-less.

This slice introduces none of the D173-forbidden authority shortcuts and no
user-facing selection mechanism:

~~~text
NETWORKLESS_DEFAULT_PRESERVED=YES
CAPTURED_TOOL_PROCESS_NETWORKLESS=YES

PROCESS_NETWORK_ACCESSOR_ADDED=NO
GLOBAL_NETWORK_ACCESSOR_ADDED=NO
NETWORK_REGISTRY_ADDED=NO
STDLIB_TCP_FACADE_ADDED=NO
CLI_NETWORK_SYNTAX_ADDED=NO
SPEC_CHANGED=NO
NEW_DESIGN_DECISION_REQUIRED=NO
~~~

## Focused regression evidence

`ProtosFreshProcessExecutorTest` at the exact product revision retains focused
coverage for the new seam:

- a request without a Network grant fails to resolve the initial `network`
  binding and does not initialize the host Network plane;
- an explicitly provisioned Network is observed by guest execution as the exact
  same capability object when execution uses the provisioning RuntimeHost; and
- the temporary-host overload rejects a pre-existing grant before guest code can
  produce output.

The maintainer reported the slice tests passed before publication and reported
the product commit pushed successfully in the active I065 interaction.

~~~text
MAINTAINER_REPORTED_TESTS=PASS
PRODUCT_PUBLICATION=PUSHED
~~~

This evidence does not invent an exact validation command or CI run identity that
was not supplied by the maintainer. GitHub commit status/workflow lookup did not
provide an additional run identity at the time this record was prepared.

## Remaining I065 work

I065 / #668 is not closed by this publication. The product Issue requires the
explicit grant seam to continue through normal application/hosting entry
machinery while preserving default absence and the same-host lifetime rule.

The next bounded dependency is the workspace application path:

~~~text
ProtosWorkspacePackageApplicationExecution
    -> existing same live ProtosPolyglotRuntimeHost
    -> ProtosStandaloneProcessBootstrap
~~~

Subsequent I065 work must still reconcile the remaining application/session
hosting seams where relevant, without choosing CLI/package spelling inside I065.

~~~text
I065_STATUS=OPEN_IN_PROGRESS
SLICE=FRESH_PROCESS_NETWORK_GRANT
SLICE_PRODUCT_REVISION=c4e108a7e850ff2fc31b52684fb041d21a7c03b6
NEXT_SLICE=WORKSPACE_APPLICATION_NETWORK_GRANT_PROPAGATION
~~~
