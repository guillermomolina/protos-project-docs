# I082-F — foreign provider authority-profile enforcement closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-F
PROTOS_ISSUE=guillermomolina/protos#830
PLATFORM_AUTHORITY=PLAT052/guillermomolina/protos#821
TOPOLOGY_AUTHORITY=PLAT053/guillermomolina/protos#822
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=ca650e230a72f12685d2dcdb3a927e1b5fba0a33
ENDING_PROTOS_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
ENDING_VERSION=0.3.279-SNAPSHOT
SPECIFICATION_REVISION=0.1.447
COMMIT_SUBJECT=I082-F: enforce foreign provider authority profiles
~~~

The exact published commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosForeignModuleImports.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderAdmission.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderDescriptor.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderEnforcement.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderExecutionProfile.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderProcessLifecycle.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignInertRestriction.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignModuleImportTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignProviderEnforcementTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignProviderLifecycleTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignValueFixture.java
~~~

No normative specification file changed.

## PLAT052 enforcement implementation

I082-F turns the four provider execution profiles from descriptive metadata into
runtime enforcement states.

A single internal `ProtosForeignProviderAdmission` is the provider admission
owner. Its decision uses only immutable host-supplied descriptor metadata and
runs no provider code.

~~~text
DEFAULT_FOREIGN_AUTHORITY=ZERO_AMBIENT

UNAVAILABLE=FAIL_CLOSED

RESTRICTED_IN_PROCESS=
  REQUIRES ProtosForeignProviderEnforcement.InProcessRestriction

STRONGLY_ISOLATED=
  REQUIRES ProtosForeignProviderEnforcement.StrongIsolation

TRUSTED_IN_PROCESS=
  HOST_OR_EMBEDDER_SELECTED
  + NO_ENFORCEMENT_MECHANISM_ATTACHED

AMBIGUOUS_OR_CONTRADICTORY_ENFORCEMENT=FAIL_CLOSED
~~~

The selected PLAT052 invariant remains:

~~~text
AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT
~~~

I082-F provisions no foreign application authority.

## Provider-entry perimeter

Admission now occurs before provider execution at the critical entry points:

~~~text
before import canonicalization
before Actor-local facade publication/initialization
before session acquisition
before compartment opening
~~~

Restricted and strongly isolated compartments are opened only through their
matching enforcement mechanism. Trusted host-configured providers are the only
profile allowed to call the provider factory directly.

D188/D189 value operations are reachable only through a session binding acquired
after this admission chain.

A rejected provider has no provider-side execution effect.

## Immutable host ownership

Provider profile and enforcement are fixed in the immutable RuntimeHost-owned
registry configuration.

~~~text
IMPORT_CAN_SELECT_PROVIDER_ROUTE=YES
IMPORT_CAN_CHANGE_PROVIDER_PROFILE=NO
IMPORT_CAN_UPGRADE_TRUST=NO
DEPENDENCY_CAN_UPGRADE_TRUST=NO
GUEST_CAN_UPGRADE_TRUST=NO
FOREIGN_CODE_CAN_UPGRADE_TRUST=NO
~~~

No guest-visible security API or second Protos capability system is introduced.

## Zero-authority conformance proof

The existing plain-Java conformance providers now use an inert test-only
zero-authority restriction. The published
`ProtosForeignProviderEnforcementTest` proves fail-closed profile behavior and
the provider-entry perimeter.

The published implementation adds no:

~~~text
production provider
provider discovery
ServiceLoader path
general HostAccess
PolyglotAccess
general I/O authority
network authority
subprocess authority
native authority
environment authority
host class lookup service
foreign Context construction
std:interop public API
~~~

The generic runtime does not claim that an arbitrary host library is restricted
merely because its descriptor says `RESTRICTED_IN_PROCESS`.

## Preserved authority

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
D192_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

Existing Process-owned compartment, Actor/provider session, module identity,
D188 value semantics and D189 callback semantics are unchanged.

## Validation provenance

The maintainer reported after publication:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded exactly as:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No exact test count, command list, duration, or CI result is inferred.

The exact product revision was independently re-read from GitHub. It is
`26a844c6f825822e73a419e44635aad4558ebdc4`, version
`0.3.279-SNAPSHOT`, with specification revision `0.1.447`.

## Closure and next routing

I082-F is complete.

The next implementation slice is I082-G: the first real host-Java `java:`
provider and end-to-end proof over the already-closed D188, D189, PLAT052 and
PLAT053 substrate.

I082-H was defined only as an integrated closure slice **if still required**.
I082-G should therefore include the complete end-to-end and zero-use regression
validation needed to close I082. Allocate I082-H only if G reveals a concrete
remaining integration/closure residual.

~~~text
I082_F_STATUS=COMPLETED
I082_STATUS=IN_PROGRESS

NEXT_SLICE=I082-G
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_GOAL=HOST_JAVA_PROVIDER_AND_END_TO_END_I082_BASELINE

I082_H=CONDITIONAL_ONLY_IF_G_LEAVES_REAL_RESIDUAL
~~~

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published product commit, PLAT052/PLAT053 ratified authority,
live GitHub coordination, and maintainer-reported validation. No independent
human review is claimed.
