# I082-A — provider-neutral foreign provider registry foundation closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-A
PROTOS_ISSUE=guillermomolina/protos#830
PARENT_WORK=AUD019/guillermomolina/protos#818
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_SCOPE=PROVIDER_NEUTRAL_FOREIGN_PROVIDER_REGISTRY_FOUNDATION
~~~

This record is durable non-normative project evidence. It does not replace the
normative Protos specification, the ratified D188/D189/PLAT052/PLAT053
authorities, or live GitHub coordination.

## Published product revision

I082-A was published as:

~~~text
STARTING_PROTOS_REVISION=d80536af4db3632d7268d370d216418c1c4fa533
STARTING_IMPLEMENTATION_VERSION=0.3.271-SNAPSHOT

ENDING_PROTOS_REVISION=aa7ac80a44b81c2cd4bb6420a475c0457aad45ab
ENDING_IMPLEMENTATION_VERSION=0.3.272-SNAPSHOT
SPECIFICATION_REVISION=0.1.446

COMMIT_SUBJECT=I082-A: add provider-neutral foreign provider registry foundation
~~~

The exact I082-A commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderCompartment.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderDescriptor.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderExecutionProfile.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderFactory.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderId.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderRegistry.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotRuntimeHost.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignProviderRegistryTest.java
~~~

No normative specification file changes.

## Implemented foundation

I082-A establishes the provider-neutral model selected by PLAT053 without
creating foreign execution behavior.

The exact published implementation provides:

- `ProtosForeignProviderId` as stable internal provider identity;
- `ProtosForeignProviderExecutionProfile` with the four ratified profiles:
  `RESTRICTED_IN_PROCESS`, `TRUSTED_IN_PROCESS`,
  `STRONGLY_ISOLATED`, and `UNAVAILABLE`;
- `ProtosForeignProviderDescriptor` as immutable provider metadata + factory;
- `ProtosForeignProviderFactory` as a host-supplied lazy compartment factory;
- `ProtosForeignProviderCompartment` as a logical Process-owned provider
  boundary explicitly documented as not synonymous with a Polyglot Context;
- `ProtosForeignProviderRegistry` as immutable exact-id registry with
  deterministic duplicate rejection and no ambient discovery;
- ownership of one immutable registry by `ProtosPolyglotRuntimeHost`;
- an empty registry for normal and debug RuntimeHost construction;
- test-only construction sufficient to prove non-empty registry behavior without
  exposing guest-visible registration.

The published `CHANGELOG.md` states that registration itself invokes no provider
factory and creates no compartment, session, foreign Context, or foreign runtime.

## Ratified authority preserved

The slice does not reopen or alter the prerequisite decision chain:

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
~~~

In particular:

~~~text
PROVIDER_REGISTRY_OWNER=RUNTIMEHOST
REGISTRY_MUTABLE_AFTER_CONSTRUCTION=NO
REGISTRY_GUEST_VISIBLE=NO
AMBIENT_PROVIDER_DISCOVERY=NO
PROVIDER_EQUALS_CONTEXT=NO

FOREIGN_IMPORT_ROUTING_IMPLEMENTED=NO
FOREIGN_MODULEKEY_IMPLEMENTED=NO
FOREIGN_MODULE_FACADE_IMPLEMENTED=NO
FOREIGN_VALUE_DISPATCH_IMPLEMENTED=NO
FOREIGN_CALLBACK_IMPLEMENTED=NO
HOSTACCESS_BROADENED=NO
POLYGLOTACCESS_BROADENED=NO
FOREIGN_CONTEXT_CREATED=NO
FOREIGN_RUNTIME_INITIALIZED=NO
STD_INTEROP_IMPLEMENTED=NO
~~~

The existing Protos Engine and Process Context placement remain unchanged.

## Pay-as-you-grow result

The foundation keeps the zero-use baseline inert:

~~~text
NORMAL_RUNTIMEHOST_PROVIDER_REGISTRY=EMPTY
DEBUG_RUNTIMEHOST_PROVIDER_REGISTRY=EMPTY
PROVIDER_FACTORY_INVOCATION_ON_HOST_OPEN=NO
PROVIDER_COMPARTMENT_CREATION_ON_HOST_OPEN=NO
PROVIDER_SESSION_CREATION_ON_HOST_OPEN=NO
FOREIGN_CONTEXT_CREATION_ON_HOST_OPEN=NO
FOREIGN_RUNTIME_INITIALIZATION_ON_HOST_OPEN=NO
~~~

Only bounded immutable provider-registry metadata is introduced.

## Validation provenance

The maintainer reported after the product commit was pushed:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

This record preserves exactly that provenance:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No command name, test count, duration, suite composition, or additional test
detail is inferred beyond the maintainer report.

The published commit and branch state were independently re-read from GitHub.
The exact I082-A parent is
`d80536af4db3632d7268d370d216418c1c4fa533`, and the ending product revision is
`aa7ac80a44b81c2cd4bb6420a475c0457aad45ab`.

## Slice closure

I082-A satisfies its bounded foundation objective and closes as completed.

The parent I082 remains open. The next dependency-ordered implementation slice is
I082-B, which owns Process-scoped lazy compartment materialization and
Actor-scoped logical provider-session lifetime.

I082-B is deliberately not merged with I082-C for planning purposes because C
crosses into a different implementation surface: foreign import routing,
canonical foreign ModuleKey production, Actor-local foreign-module facade/cache
lifecycle, cycle handling and failure retry. B can establish and validate
lifecycle ownership independently while C remains untouched.

~~~text
I082_A_STATUS=COMPLETED
I082_STATUS=IN_PROGRESS
NEXT_SLICE=I082-B
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_GOAL=PROCESS_OWNED_LAZY_PROVIDER_COMPARTMENT_AND_ACTOR_SESSION_LIFECYCLE
~~~

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published I082-A commit, current live GitHub coordination, the
ratified AUD019 decision chain, and the maintainer-reported local validation.
No independent human review is claimed by this record.
