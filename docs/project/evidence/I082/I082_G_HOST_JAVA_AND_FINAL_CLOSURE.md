# I082-G — restricted host-Java provider and final I082 closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-G
PROTOS_ISSUE=guillermomolina/protos#830
PARENT_WORK=AUD019/guillermomolina/protos#818
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative implementation and closure evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=26a844c6f825822e73a419e44635aad4558ebdc4
ENDING_PROTOS_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
ENDING_VERSION=0.3.280-SNAPSHOT
SPECIFICATION_REVISION=0.1.447
COMMIT_SUBJECT=I082-G: add restricted host Java provider baseline
~~~

No normative specification file changed.

## Published host-Java baseline

I082-G adds the first production foreign provider for the `java:` scheme.

~~~text
JAVA_BACKEND=HOST_JAVA
ESPRESSO_USED=NO
EXTRA_POLYGLOT_CONTEXT=NO
DEPENDENCY_ACQUISITION=NO
ARBITRARY_CLASSPATH_LOOKUP=NO
~~~

`ProtosHostJavaCatalogue` is the immutable embedder-supplied catalogue of
already-provisioned exact Java classes and exact public constructors/methods
that may cross the provider boundary. Classpath presence alone never authorizes
an import.

The baseline permits at most one exposed constructor per class and one exposed
static/instance method per selector and kind, so I082-G introduces no
guest-visible Java overload-selection rule. Only public members declared by the
admitted class are eligible; inherited members such as `getClass` are not
implicitly exposed.

`ProtosPolyglotRuntimeHost.openWithHostJava(catalogue)` installs the provider.
Ordinary `open()` still configures no foreign provider and no Java authority.

## PLAT052 / PLAT053 preservation

~~~text
JAVA_PROVIDER_PROFILE=RESTRICTED_IN_PROCESS
PROVISIONED_FOREIGN_APPLICATION_AUTHORITY=NONE
IMPORT_CAN_UPGRADE_AUTHORITY=NO
~~~

A non-admitted Java class is resolved as a catalogue miss; guest input does not
trigger class loading, initialization, classpath scanning, or dependency
acquisition.

The production provider does not expose unrestricted reflection, HostAccess,
PolyglotAccess, filesystem, network, subprocess, native, environment, or
classloader authority to Protos code. Implementation-private reflection is used
only to invoke exact members preselected by the embedder.

Provider compartments and Actor/provider sessions remain lazy and generation
bound. The ordinary RuntimeHost retains the empty provider registry.

## D188 host-Java use

Imported Java classes remain Actor-local Protos foreign-module facades.

The provider-specific default-false
`ProtosForeignModuleProvider.publishesFacadeCall()` hook lets a provider
deliberately publish the facade's local `call` for an unambiguously executable
target. Host Java uses that D188-approved construction path for an exposed
constructor; other providers remain unchanged.

The end-to-end baseline proves:

~~~text
STATIC_METHOD_CALL=PASS
CONSTRUCTION=PASS
INSTANCE_METHOD_CALL=PASS
~~~

Java scalar/result admission follows D188 lossless categories. Other admitted
Java objects remain RAW and use host reference identity; Java `.equals()` and
`hashCode()` are not imported as Protos equality/hash.

Unsupported or lossy outbound conversions fail before the Java member executes.
Entered Java failures become fresh `ForeignError` values without exposing the
host Throwable as a guest value.

## Lifetime, isolation and zero use

Published tests prove:

~~~text
ACTOR_LOCAL_MODULE_IDENTITY=PASS
SESSION_CLOSE_INVALIDATES_OLD_REFERENCES=PASS
CLOSED_REFERENCE_REBIND=NO
ACTOR_TRANSFER_OF_JAVA_RAW=NO
P_TRANSFER_OF_JAVA_RAW=NO
ZERO_USE_CONFIGURED_JAVA_PROVIDER=PASS
ZERO_USE_DEFAULT_RUNTIMEHOST=PASS
ORDINARY_IMPORT_REGRESSION=PASS
~~~

No Java application object becomes Process-global provider state.

## I082 acceptance closure

The completed A-G publication sequence satisfies I082/#830 acceptance:

~~~text
1  PLAT053 provider registry/compartment/session architecture = PASS
2  zero foreign use initializes no foreign runtime = PASS
3  canonical foreign ModuleKey + Actor-local module identity = PASS
4  cycle/failure eviction/retry semantics = PASS
5  D188 value/identity/member/call/index/iteration/error baseline = PASS
6  D189 same-Actor/current-Task synchronous callback/lifetime = PASS
7  PLAT052 zero-ambient authority/fail-closed profiles = PASS
8  host-Java static/constructor/instance baseline without Espresso = PASS
9  mutable Java/foreign state does not bypass Actor isolation = PASS
10 ordinary non-foreign imports/runtime unchanged = PASS
11 shutdown leaves no resurrected foreign references/callbacks = PASS
12 integrated tests and git diff --check = PASS_MAINTAINER_REPORTED
13 exact publication and closure revisions identified = PASS
~~~

No technical residual requires the conditional I082-H slice.

~~~text
I082_H_REQUIRED=NO
I082_STATUS=COMPLETED
I082_READY_FOR_CLOSURE=YES
~~~

## Validation provenance

After publication the maintainer reported:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded exactly as:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No exact test count, command list, duration, or CI result is inferred.

## Final scope boundary

I082 intentionally leaves separately closable future work for public
`std:interop`, additional guest-language providers, retained/asynchronous
callbacks, generic foreign cancellation, Actor/P foreign-value rematerialization,
third-party provider registration, dependency acquisition, and provider
hot-loading.

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published I082-G revision, I082 acceptance contract, ratified
D188/D189/PLAT052/PLAT053 authority, and maintainer-reported validation.
