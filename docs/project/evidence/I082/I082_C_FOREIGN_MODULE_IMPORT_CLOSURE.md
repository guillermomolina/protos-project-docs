# I082-C — explicit foreign import routing and Actor-local module facade closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-C
PROTOS_ISSUE=guillermomolina/protos#830
PARENT_WORK=AUD019/guillermomolina/protos#818
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

This record is durable non-normative project evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=50f0107fa9a4e5ed24f74ef6f39a8de28a1045ca
STARTING_VERSION=0.3.273-SNAPSHOT

ENDING_PROTOS_REVISION=c21ff278e6dca1219429cc7f08568c54af2bd951
ENDING_VERSION=0.3.274-SNAPSHOT
SPECIFICATION_REVISION=0.1.446

COMMIT_SUBJECT=I082-C: route explicit foreign imports to Actor-local module facades
~~~

The exact product commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosForeignImportRoute.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignModuleFacadeValue.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignModuleImports.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignModuleKey.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignModuleProvider.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderDescriptor.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderRegistry.java
src/main/java/com/guillermomolina/protos/execution/ProtosModuleRuntime.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignModuleImportTest.java
~~~

No normative specification file changes.

## Implemented module/import boundary

I082-C implements the D188/PLAT053 foreign-module lifecycle without introducing a
concrete production provider.

~~~text
EXPLICIT_FOREIGN_ROUTE=YES
UNKNOWN_SCHEME_FALLS_BACK_TO_SOURCE_RESOLVER=YES
SOURCE_SCHEME_CAPTURE_PROHIBITED=YES
DUPLICATE_ROUTE_REJECTED=YES

FOREIGN_MODULEKEY_MODEL=STRUCTURED_COLLISION_SAFE
FOREIGN_KEY_ENCODING=foreign:v1:<base64url-scheme>:<base64url-target>
PROVIDER_CANONICAL_TARGET_PART_OF_KEY=YES
SESSION_OR_ACTOR_ID_PART_OF_KEY=NO

ACTOR_LOCAL_FOREIGN_FACADE=YES
RAW_PROVIDER_TARGET_IS_MODULE_INSTANCE=NO
PROVIDER_CACHE_REPLACES_ACTOR_MODULE_CACHE=NO

CACHE_BEFORE_FOREIGN_INITIALIZATION=YES
CYCLIC_IMPORT_RETURNS_SAME_PARTIAL_FACADE=YES
SUCCESS_MARKS_SAME_RECORD_READY=YES
FAILED_INIT_EVICTS_EXACT_RECORD=YES
RETRY_CREATES_FRESH_FACADE=YES
ESCAPED_FAILED_FACADE_REBOUND=NO

B_SESSION_REUSED=YES
SECOND_SESSION_CACHE_CREATED=NO
~~~

The foreign facade stores its provider/session/canonical-target/acquired-target
attachment privately. Those implementation handles are not guest-visible slots.

I082-C deliberately does not implement:

~~~text
D188_MEMBER_PROJECTION=NO
D188_RAW_FOREIGN_VALUE_SUBSTRATE=NO
D188_FOREIGN_ERROR_RUNTIME=NO
D189_CALLBACK_BRIDGE=NO
PLAT052_AUTHORITY_ENFORCEMENT=NO
CONCRETE_PRODUCTION_PROVIDER=NO
STD_INTEROP_PUBLIC_API=NO
~~~

## Source-module compatibility

The source-backed resolver and ordinary source module lifecycle remain
authoritative for all non-routed imports. Unknown schemes do not implicitly
become foreign imports and do not invoke provider factories.

The foreign ModuleKey namespace is structurally isolated from source keys, so a
provider-controlled canonical target cannot collide with a source-backed
canonical identity.

## Validation evidence

The exact published `ProtosForeignModuleImportTest` includes coverage for:

- explicit route selection and alias canonicalization;
- source resolver authority and unknown schemes;
- route validation and duplicate/source-scheme rejection;
- source/foreign ModuleKey collision safety;
- Actor-local facade identity and Actor-session isolation;
- cache-before-initialization and A -> B -> A cycles;
- failure eviction and fresh retry facade;
- no guest members added by this slice;
- empty-registry zero-use behavior;
- absence of HostAccess/foreign authority enablement.

The maintainer reported after publication:

> el git diff check esta limpio.
>
> Todos los tests han pasado en local

Recorded provenance:

~~~text
GIT_DIFF_CHECK=PASS_MAINTAINER_REPORTED
LOCAL_TESTS=PASS_MAINTAINER_REPORTED
~~~

No unreported command, count, duration or suite composition is inferred.

## Slice closure and next routing

~~~text
I082_B_STATUS=COMPLETED
I082_C_STATUS=COMPLETED
I082_STATUS=IN_PROGRESS

NEXT_SLICE=I082-D
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
NEXT_SLICE_GOAL=D188_GENERIC_FOREIGN_VALUE_SUBSTRATE
~~~

I082-D remains separate from I082-E. D is itself a substantial closure boundary:
foreign scalar admission, raw reference identity, Protos-facing member/call/index/
iteration projection and ForeignError projection. E then adds D189 callback
re-entry, same Actor/Task dynamic ownership, expiry and foreign-thread rejection.
Combining them would enlarge the semantic and lifecycle failure domain rather
than merely reduce publication overhead.

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published I082-B/I082-C commits, current normative specification,
live GitHub state and maintainer-reported local validation.
