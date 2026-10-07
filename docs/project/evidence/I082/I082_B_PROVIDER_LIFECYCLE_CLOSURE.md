# I082-B — Process-owned lazy provider compartments and Actor-isolated sessions closure

Date: 2026-10-07

## Work identity

~~~text
WORK_ITEM=I082
SLICE=I082-B
PROTOS_ISSUE=guillermomolina/protos#830
PARENT_WORK=AUD019/guillermomolina/protos#818
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

This record is durable non-normative project evidence.

## Published product revision

~~~text
STARTING_PROTOS_REVISION=65a23bc25bd30dfd65843468dba3812a78121c06
STARTING_VERSION=0.3.272-SNAPSHOT

ENDING_PROTOS_REVISION=50f0107fa9a4e5ed24f74ef6f39a8de28a1045ca
ENDING_VERSION=0.3.273-SNAPSHOT
SPECIFICATION_REVISION=0.1.446

COMMIT_SUBJECT=I082-B: add Process-owned lazy provider compartments and Actor-isolated sessions
~~~

The exact product commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderCompartment.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderFactory.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderProcessLifecycle.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderSession.java
src/main/java/com/guillermomolina/protos/execution/ProtosForeignProviderSessionBinding.java
src/main/java/com/guillermomolina/protos/execution/ProtosPolyglotProcessContext.java
src/main/java/com/guillermomolina/protos/runtime/ProtosProcessExecutionHost.java
src/main/java/com/guillermomolina/protos/runtime/ProtosProcessRuntime.java
src/test/java/com/guillermomolina/protos/execution/ProtosForeignProviderLifecycleTest.java
~~~

No normative specification file changes.

## Implemented lifecycle

The published implementation establishes the PLAT053 lifecycle baseline:

~~~text
PROCESS_OWNS_PROVIDER_COMPARTMENT=YES
COMPARTMENT_LAZY_ON_FIRST_USE=YES
ONE_COMPARTMENT_PER_PROCESS_PROVIDER=YES

ACTOR_PROVIDER_SESSION_OWNER=ACTOR
SESSION_LAZY_ON_FIRST_USE=YES
SAME_ACTOR_PROVIDER_REUSES_SESSION=YES
DIFFERENT_ACTORS_RECEIVE_DISTINCT_SESSIONS=YES
DIFFERENT_PROCESSES_RECEIVE_DISTINCT_COMPARTMENTS=YES

PROVIDER_EQUALS_CONTEXT=NO
SESSION_EQUALS_CONTEXT=NO

FAILED_COMPARTMENT_CONSTRUCTION_CACHED=NO
FAILED_SESSION_CONSTRUCTION_CACHED=NO

ACTOR_TERMINAL_CLOSES_OWN_SESSIONS=YES
PROCESS_TERMINAL_CLOSES_REMAINING_SESSIONS=YES
PROCESS_TERMINAL_CLOSES_COMPARTMENTS=YES
CLEANUP_CONTINUES_AFTER_INDIVIDUAL_CLOSE_FAILURE=YES
TERMINAL_CLEANUP_FAILURE_SURFACED_TO_HOST=YES

SESSION_BINDING_IS_GENERATION_IDENTITY=YES
CLOSED_BINDING_REOPENS=NO
~~~

The provider factory and session factory remain inert until first explicit internal
provider acquisition. No foreign language, foreign Context, HostAccess,
PolyglotAccess, native access, process access, network authority, filesystem
authority or guest-visible interop operation is enabled by I082-B.

## Validation evidence

The exact published test class includes coverage for:

- zero-use laziness;
- one compartment among many Actors when only one Actor uses a provider;
- Actor session isolation;
- Process compartment isolation;
- independent providers;
- unknown and unavailable providers;
- cross-Process Actor rejection;
- retry after failed compartment/session construction;
- Actor-terminal cleanup;
- Process-terminal cleanup and admission rejection;
- cleanup failure continuation and propagation;
- racing first compartment acquisition;
- racing same-Actor first session acquisition.

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

## Reconciliation of prior pending record

The earlier record:

~~~text
docs/project/evidence/I082/I082_B_MAINTAINER_REPORT_PENDING_PUBLICATION_VERIFICATION.md
~~~

correctly captured the temporary state in which the maintainer had reported the
push but GitHub had not yet exposed the product commit.

That publication is now visible and verified. This closure record supersedes only
the pending status conclusion; the earlier record remains valid historical
evidence of the transient discrepancy.

~~~text
I082_B_PRODUCT_PUBLICATION=VERIFIED
I082_B_STATUS=COMPLETED
I082_STATUS=IN_PROGRESS
~~~

## Next routing

I082-C is already published after this commit and owns explicit foreign import
routing, canonical foreign ModuleKeys, Actor-local facades, cache-before-init,
cycles, failure eviction and retry.

## AI-assistance disclosure

This closure evidence was materially prepared with AI assistance from ChatGPT
using the exact published product commit, live GitHub state, the ratified
PLAT053 contract and maintainer-reported local validation.
