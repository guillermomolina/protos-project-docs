# I065 — standalone application explicit Network grant propagation

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

The standalone-application I065 plumbing slice is published at:

~~~text
PROTOS_REVISION=49dc0a4b4e70a04f7ce9d05a078b31a4bdddaa46
COMMIT_SUBJECT=I065: thread Network grant through standalone application hosting
IMPLEMENTATION_VERSION=0.3.209-SNAPSHOT
~~~

The commit changes exactly:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliPolyglotRoutingArchitectureTest.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
src/test/java/com/guillermomolina/protos/embedding/ProtosStandaloneHostedExecutionEmbeddingTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSessionLifecycleTest.java
~~~

No normative specification file, Standard Library Network module, TCP semantic
implementation, manifest schema or CLI option is changed by this slice.

## Standalone host-aware bootstrap

At the exact product revision above, the existing standalone bootstrap remains
Network-less by default while a host-aware overload carries the already-defined
policy-neutral:

~~~text
NetworkGrant.NONE
NetworkGrant.HOST_NETWORK
~~~

The host-aware path creates the exact application Prelude and then, only for
`HOST_NETWORK`, provisions the capability through:

~~~text
runtimeHost.provisionHostNetwork(prelude)
~~~

before passing that capability into the full
`ProtosStandaloneProcessBootstrap` contract.

~~~text
DEFAULT_STANDALONE_BOOTSTRAP_NETWORK=NONE
EXPLICIT_STANDALONE_NETWORK_SELECTION=YES
EXACT_APPLICATION_PRELUDE_PROVISIONING=YES
~~~

## RuntimeHost ownership inversion

The standalone/session hosting sequence now opens the owning RuntimeHost before
bootstrapping the Process.

This preserves the D173 lifetime invariant:

~~~text
open owning RuntimeHost
create exact application Prelude
provision Network from that Prelude on owning RuntimeHost when selected
bootstrap Process with that Network
host Process on the same RuntimeHost
keep RuntimeHost live through Process lifetime
~~~

The prior networkless behavior did not require this ordering; the new ordering
is what makes an explicit production grant safe without introducing a temporary
or ambient host.

Failure paths close the RuntimeHost when bootstrap does not complete, while the
existing bind/close paths retain their Process/Context cleanup responsibilities.

~~~text
SAME_RUNTIME_HOST_PROVISION_AND_HOST=YES
RUNTIME_HOST_OPEN_BEFORE_GRANTED_BOOTSTRAP=YES
BOOTSTRAP_FAILURE_HOST_CLEANUP=YES
~~~

## Standalone embedding/session APIs

`ProtosStandaloneHostedSession.open(...)` preserves its historical default as
`NetworkGrant.NONE` and gains an explicit-grant overload.

`ProtosStandaloneHostedExecution.executeFile(...)` likewise preserves all
historical overloads as Network-less and gains an explicit-grant overload that
reuses the hosted session path.

The session retains the same Process, Polyglot Process Context, RuntimeHost and
entry-module state until close; Network does not create a second execution or
lifecycle path.

~~~text
STANDALONE_SESSION_DEFAULT_NETWORK=NONE
STANDALONE_SESSION_EXPLICIT_HOST_NETWORK=YES
EXECUTE_FILE_DEFAULT_NETWORK=NONE
EXECUTE_FILE_EXPLICIT_HOST_NETWORK=YES
SESSION_LIFETIME_MODEL_CHANGED=NO
~~~

## CLI/session/debug boundary

The CLI's internal ordinary-session and debug-session construction seams now
accept the same policy-neutral Network selection.

Every current public CLI caller explicitly remains `NONE`:

~~~text
-e=NONE
DIRECT_FILE=NONE
REPL=NONE
DEBUG=NONE
BUNDLED_TOOL_SESSIONS=NONE
~~~

The debug RuntimeHost opens before the Process bootstrap, and readiness is
published only after bootstrap succeeds. No CLI syntax or public selection
policy is introduced.

~~~text
CLI_NETWORK_SYNTAX_ADDED=NO
CLI_HELP_NETWORK_POLICY_ADDED=NO
DEBUG_PUBLIC_DEFAULT_NETWORK=NONE
DEBUG_SAME_HOST_GRANT_SEAM=YES
~~~

## Pay-as-you-grow / authority preservation

`NetworkGrant.NONE` performs no host Network provisioning and does not
initialize the RuntimeHost Network plane merely because the carrier exists.

The slice adds none of the D173-forbidden authority shortcuts:

~~~text
NONE_INITIALIZES_NETWORK_HOST=NO
PROCESS_NETWORK_ACCESSOR_ADDED=NO
GLOBAL_NETWORK_ACCESSOR_ADDED=NO
NETWORK_REGISTRY_ADDED=NO
IMPORT_SIDE_EFFECT_NETWORK_ACQUISITION=NO
ACTOR_OR_P_IMPLICIT_NETWORK_PROPAGATION=NO
STDLIB_TCP_FACADE_ADDED=NO
SPEC_CHANGED=NO
NEW_DESIGN_DECISION_REQUIRED=NO
~~~

## Focused regression evidence

The published tests establish that:

- default standalone hosted sessions leave the `network` slot absent and do
  not initialize the host Network plane;
- an explicit standalone session grant yields a real
  `ProtosNetworkCapabilityValue` delegating to that exact application's
  `Network` prototype;
- the same granted capability remains visible through later operations of the
  same live session;
- session Process/Context/RuntimeHost lifecycle and close behavior remain
  intact;
- default one-shot embedding remains Network-less;
- explicit one-shot embedding receives the exact application's Network;
- public CLI `-e` and direct-file routes remain Network-less;
- the internal CLI session seam can carry an explicit grant to the exact
  application Prelude/host;
- architecture coverage requires current CLI session/debug seam callers to
  select `NONE` and forbids a public `HOST_NETWORK` selection in
  `ProtosCli`.

The maintainer reported all local tests passing after publication:

~~~text
LOCAL_FULL_VALIDATION=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
MAINTAINER_REPORT="Todos los tests han pasado en local"
PRODUCT_PUBLICATION=PUSHED
~~~

No remote CI success is asserted by this record. The available commit-status
and commit-workflow queries returned no qualifying result at evidence-capture
time.

## Full I065 implementation sequence

Published implementation slices now form:

~~~text
FRESH_PROCESS_REVISION=c4e108a7e850ff2fc31b52684fb041d21a7c03b6
WORKSPACE_APPLICATION_REVISION=564dc97aacb593826011a8876554d69dd6529faa
PACKAGE_APPLICATION_REVISION=c1b8a3f87d8c90a654e19069139191dba4d922b6
STANDALONE_APPLICATION_REVISION=49dc0a4b4e70a04f7ce9d05a078b31a4bdddaa46
~~~

A current production-source call-site scan after the fourth slice finds the
normal application-hosting carriers reconciled:

- fresh Process execution carries an explicit optional Network grant;
- workspace application execution carries `NONE/HOST_NETWORK`;
- package-backed application execution carries `NONE/HOST_NETWORK`;
- standalone hosted session and one-shot embedding carry
  `NONE/HOST_NETWORK`;
- CLI ordinary/debug session machinery can carry the selection while every
  current public CLI route remains `NONE`.

Other direct standalone-bootstrap production call sites are Tool/preflight,
verification, formatter or captured-Process boundaries whose Network-less
default is intentional and required by I065.

## Resulting work state

No additional product implementation slice is identified by this publication
review.

I065 should proceed to a falsifying final closure review against the then-current
HEAD. That review must verify the complete D173 carrier/lifetime/default-absence
contract, required validation/CI closure conditions, and absence of forbidden
ambient authority before #668 is closed.

~~~text
I065_IMPLEMENTATION_COVERAGE=COMPLETE_CANDIDATE
I065_STATUS=REVIEW
NEXT_ACTIVITY=FINAL_CLOSURE_REVIEW
NEXT_ACTIVITY_TYPE=INVESTIGATION
NEXT_IMPLEMENTATION_SLICE=NONE
~~~
