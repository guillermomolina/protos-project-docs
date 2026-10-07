# PLAT053 — Foreign provider, Context, runtime, lifecycle, and distribution architecture

Status: **RATIFIED**

Selected architecture: **Candidate B′ — Process-owned lazy provider compartments with Actor-isolated logical sessions and provider-selected physical topology**.

Approval: explicit project-owner approval on 2026-10-07:

~~~text
Apruebo B'
~~~

Decision Issue: `guillermomolina/protos#822`

Parent audit: `AUD019 / guillermomolina/protos#818`

Direct prerequisites:

~~~text
D188 / #819 = SATISFIED_AND_NORMATIVE
D189 / #820 = SATISFIED_AND_NORMATIVE
PLAT052 / #821 = RATIFIED
SPECIFICATION_REVISION = 0.1.446
~~~

Ratification revalidation:

~~~text
PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
PROTOS_VERSION=0.3.269-SNAPSHOT
SPECIFICATION_REVISION=0.1.446
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative platform/runtime architecture. PLAT053 selects
provider/runtime placement, ownership, lifecycle, caching and distribution
machinery while preserving the already-normative Protos module, Actor, callback,
authority and foreign-value contracts.

## Decision

Foreign interoperability uses **Process-owned lazy provider compartments**.
A provider compartment is a logical runtime/acquisition boundary, not a promise
that every provider is represented by one GraalVM `Context`.

Mutable foreign application/runtime state observable through ordinary Protos
foreign values is **Actor-isolated by default**. Each Actor that first uses a
provider obtains a lazy logical provider session. The provider selects the
smallest physical mechanism that preserves Protos semantics and the PLAT052
authority contract.

A provider session may therefore be realized by, for example:

~~~text
Polyglot Context
inner Context / realm
interpreter
class loader / guest VM
isolate
external process or service session
another equivalent provider-specific mechanism
~~~

The physical choice is implementation-private when it preserves the selected
contract.

## Fixed invariants

Candidate B′ preserves the complete prerequisite chain:

~~~text
GENERIC_FOREIGN_VALUE_SUBSTRATE=YES
GENERIC_ALL_LANGUAGE_MODULE_LOADER=NO
PER_LANGUAGE_OR_MODULE_SYSTEM_PROVIDER=YES
DEPENDENCY_ACQUISITION_REMAINS_SEPARATE=YES

FOREIGN_RUNTIME_CACHE_IS_PROTOS_ACTOR_MODULE_CACHE=NO
UNUSED_PROVIDER_RUNTIME_COST=NEAR_ZERO

HOST_JAVA_IS_ESPRESSO=NO
PLAT045_NATIVE_CONTRACT_PRESERVED=YES

AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT

FOREIGN_MUTABLE_STATE_BYPASSES_ACTOR_ISOLATION=NO

D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

## Provider model and registry

The RuntimeHost owns an immutable host-supplied registry of provider
descriptors/factories.

~~~text
RuntimeHost
  |
  +-- immutable provider registry
  |
  +-- Protos Process
        |
        +-- existing Protos Process Context
        |
        +-- lazy provider compartment A
        |     +-- Actor-local session A1
        |     +-- Actor-local session A2
        |     `-- ...
        |
        `-- lazy provider compartment B
              `-- ...
~~~

The registry is not a guest-writable namespace and is not a JVM-global
singleton. A public third-party provider-registration API is not selected by
this decision.

Provider registration or discovery must never become an authority-escalation
surface.

## Existing Protos Context is not the universal foreign Context

The existing Process-scoped Protos Context remains the Protos execution domain
selected by PLAT001.

It already contains implementation authority and behavior specific to Protos,
including the PLAT022 source-readability mechanism, Process standard-stream
routing and concurrent Actor/P carrier entry. Foreign languages are therefore
not placed in that Context by default.

~~~text
FOREIGN_LANGUAGE_IN_EXISTING_PROTOS_PROCESS_CONTEXT=NO_BY_DEFAULT
~~~

A later provider-specific exception would require proof that sharing the Context
does not leak Protos implementation authority, collapse Actor isolation, change
Process concurrency or violate PLAT052. No such universal exception is selected.

## Engine and runtime substrate

The existing Protos Engine remains the engine of the Protos hosting substrate.

For foreign providers, a Process-owned provider compartment may lazily own a
provider Engine or equivalent runtime substrate when that ecosystem benefits
from one. Such a substrate may share immutable or semantically invisible
implementation artifacts across that provider's Actor sessions, including
compiled code, parsed code, immutable package metadata and JIT infrastructure.

~~~text
SHARED_IMPLEMENTATION_ARTIFACTS=PERMITTED_WHEN_SEMANTICALLY_INVISIBLE
SHARED_MUTABLE_FOREIGN_APPLICATION_STATE=NOT_PERMITTED_BY_DEFAULT
~~~

Cross-Process foreign Engine/runtime sharing is deliberately deferred. It may be
introduced later as a proof-gated semantically invisible optimization without
changing this decision.

## Actor isolation and foreign sessions

An Actor-local Protos facade is not sufficient isolation if two facades expose
the same mutable foreign global state.

Accordingly, the provider must ensure that ordinary foreign state reachable by
different Actors satisfies the existing Actor isolation rules. Thread safety
alone is not sufficient.

The selected default is:

~~~text
ACTOR A + PROVIDER X -> ACTOR-LOCAL LOGICAL SESSION X/A
ACTOR B + PROVIDER X -> ACTOR-LOCAL LOGICAL SESSION X/B
~~~

The provider may share immutable state below those sessions. It may use another
explicit capability/service boundary where an already-defined Protos contract
permits shared external service state. It may not silently replace Actor-local
state with shared mutable globals plus locks.

## ModuleKey and provider acquisition contract

The ordinary Protos resolution boundary remains authoritative:

~~~text
specifier
  -> provider routing
  -> provider canonical resolution
  -> canonical Protos ModuleKey
  -> Actor-local module-cache lookup
~~~

A foreign ModuleKey contains enough stable provider/acquisition identity to
denote the canonical target, conceptually:

~~~text
provider identity
+
provider-defined canonical target identity
+
exact acquisition provenance when required for stable resolution
~~~

It must not contain or derive semantic identity from a Polyglot Value, Context
pointer, Actor-session identifier, provider cache entry or other transient
physical runtime object.

## Protos module cache versus provider caches

The Actor-local Protos module cache remains the sole owner of observable Protos
module-instance identity.

For a foreign module cache miss:

1. create the Actor-local Protos facade;
2. insert it as `INITIALIZING`;
3. perform provider acquisition/initialization;
4. mark the same facade `READY` on success;
5. remove the cache entry on failure.

Provider-native module caches, class loaders, package caches, compiled-code
caches or runtime registries never replace this lifecycle.

A provider may need a finer private loader/realm/target generation for failed
initialization and later retry. Such machinery must preserve the normative rule
that retry creates a fresh active facade while previously escaped failed
facades/foreign values are not retroactively time-travelled or rebound.

## Provider and foreign-value lifetime

Provider compartments are Process-owned and lazy.

Actor sessions are Actor-owned logical runtime resources and are created only on
the first use of that provider by that Actor.

When an Actor reaches terminal lifecycle, no new calls or callbacks may enter
through that session and the session is released after the existing Task/cleanup
rules permit termination.

Process termination prevents new provider admission, completes the existing
semantic termination protocol, closes Actor sessions and then closes the
Process-owned provider compartments.

A raw foreign reference is tied to the exact provider session and target
generation that created it.

~~~text
CLOSED_SESSION_OR_GENERATION
  -> OLD_FOREIGN_REFERENCE_NEVER_REBINDS_TO_A_NEW_SESSION
~~~

A new provider session/generation may create new physical foreign identities;
D188 remains authoritative for losslessly admitted Protos scalar values.

## Threading profile

No one threading policy is imposed on all providers.

A provider may declare/implement a session as:

~~~text
CONCURRENT
SERIALIZED
DEDICATED_CARRIER_OR_EVENT_LOOP
EXTERNAL_SERVICE
~~~

Provider serialization is confined to the relevant provider/session. It must not
turn the whole Protos Process into a global execution lock or GIL.

Foreign-created worker/event-loop/native threads gain no Protos Actor/Task
authority. D189 remains authoritative: synchronous callbacks run in the
originating Actor and Task, and late or foreign-thread callback ingress is
rejected before Protos entry.

## PLAT052 authority profiles

Provider/library/feature admission is classified by the real execution profile,
not just by language name:

~~~text
RESTRICTED_IN_PROCESS
TRUSTED_IN_PROCESS
STRONGLY_ISOLATED
UNAVAILABLE
~~~

Trusted host authority does not waive Actor isolation.

Strong isolation remains another physical realization of the same logical
provider/session contract. It may use a Polyglot isolate, external process,
service or equivalent mechanism. Out-of-process execution does not itself grant
or prove authority confinement.

A provider crash or isolated-process failure does not transparently replay an
operation whose external effects may already have occurred.

## Representative provider consequences

The selected architecture intentionally permits different provider shapes:

- **Host Java** may use a narrow capability-aware adapter without a guest
  Context. Arbitrary JVM libraries do not become restricted merely through a
  Context or class-loader boundary and require the PLAT052 trusted/isolation
  gate where their effects cannot be confined.
- **GraalJS** naturally fits Actor-isolated lazy Contexts with a provider-owned
  shared Engine/code substrate.
- **GraalPy** managed/pure profiles may fit restricted Actor sessions; native
  extension profiles require independent admission and commonly stronger
  isolation, trust or unavailability.
- **TruffleRuby** may support a restricted native-disabled profile; C/native
  extension profiles require independent admission.
- **Espresso** remains distinct from host Java and owns guest-VM/native-library
  constraints that may limit in-process or Native support.
- **LLVM/Sulong** managed and native execution are separate profiles; native
  execution is not automatically a restricted provider.

These are architecture/admission consequences, not a promise that every
provider is implemented in the first implementation phase.

## JVM, Native and distribution model

Foreign provider availability is artifact-specific.

~~~text
BASE_PROTOS_DISTRIBUTION
  -> DOES_NOT_REQUIRE_ALL_FOREIGN_LANGUAGE_COMPONENTS

JVM_DISTRIBUTION
  -> MAY_INCLUDE_EXACT_OPTIONAL_PROVIDER_COMPONENTS

NATIVE_DISTRIBUTION
  -> PROVIDER_AVAILABILITY_IS_FIXED_BY_THE_EXACT_BUILD/ARTIFACT
  -> PLAT045_PROTOS_INTERPRETER_ONLY_NATIVE_CONTRACT_REMAINS_AUTHORITATIVE
  -> EXTERNAL_STRONGLY_ISOLATED_PROVIDERS_REMAIN_POSSIBLE
~~~

Runtime laziness and distribution size are different costs. A provider that is
never initialized can still increase binary/package/native-image size when its
runtime components are included, so distribution composition must remain exact
rather than using a mandatory all-languages aggregate.

## Pay as you grow

For a Protos program that never uses foreign interoperability:

~~~text
FOREIGN_CONTEXTS=0
FOREIGN_INTERPRETERS=0
FOREIGN_ISOLATES=0
FOREIGN_CHILD_PROCESSES=0
FOREIGN_PROVIDER_RUNTIME_INITIALIZATION=0
~~~

Only bounded registry/routing metadata may exist.

For many Actors where only one Actor uses one foreign provider, only that
provider compartment and that Actor/provider session are required.

## Candidate disposition

### Candidate A — every foreign language in the existing Protos Process Context

Rejected. It combines unrelated authority and lifecycle domains, can leak
PLAT022/Context authority, can couple language thread rules to the whole Process,
and can expose shared foreign global/module state across Actors.

### Candidate B0 — one lazy Context/provider runtime per Process

Retains the useful Process-owned lazy-provider direction but is insufficient
when the provider runtime contains mutable module/global/application state that
would then be exposed across Actors.

### Candidate B′ — Process-owned lazy providers with Actor-isolated sessions

**Selected.**

It is the smallest topology that simultaneously preserves D188 module identity,
D189 callback ownership, PLAT052 authority confinement, PLAT001 Process
concurrency, Actor foreign-mutable-state isolation and pay-as-you-grow behavior
without defining `Context` as a universal provider institution.

### Candidate C — one Polyglot Context per Actor universally

Rejected as the universal rule. It is a valid physical realization for some
Truffle providers but is unnecessarily prescriptive for host Java adapters,
isolates, external services and future runtimes that have a different natural
session representation.

### Candidate D — every provider out of process

Rejected as the universal baseline. Strong isolation is a valid B′ provider
profile, but universal process isolation pre-pays IPC, serialization, callback,
identity, crash/lifecycle and pooling machinery even when a safe in-process
profile exists.

### Candidate E — defer foreign execution

Rejected for the active AUD019 objective. The semantic and authority
prerequisites are already resolved, and deferral would now postpone a concrete
implementation architecture rather than avoid premature design.

## Comparative result

The investigation's GITHUB010 scorecard produced:

~~~text
A   = 29/60
B0  = 41/60
B′  = 55/60
C   = 35/60
D   = 38/60
E   = 42/60
~~~

Hard invariants override arithmetic totals. A and B0 also fail required
authority/isolation conditions, while E does not satisfy the active foreign
interop objective.

## Strongest argument against B′

When very many Actors use one heavyweight foreign ecosystem, Actor-isolated
sessions may consume more memory and warmup than one Process-global runtime.

The project accepts that cost boundary because replacing Actor isolation with
shared mutable foreign globals would change existing Protos semantics. Providers
remain free to share immutable code/artifacts, use copy-on-write/runtime-native
isolation, or introduce a stronger service substrate without changing the
logical session contract.

## Regret scenario and escape path

Regret scenario:

~~~text
MOST_IMPORTANT_FOREIGN_ECOSYSTEMS_REQUIRE_HEAVY_NATIVE_OR_GLOBAL_RUNTIME_STATE
AND_ACTOR_LOCAL_IN_PROCESS_SESSIONS_ARE_TOO_EXPENSIVE_OR_UNAVAILABLE
~~~

Escape path:

~~~text
KEEP_D188_D189_PLAT052_AND_PLAT053_LOGICAL_CONTRACTS
MOVE_AFFECTED_PROVIDER_SESSIONS_TO_ISOLATES_PROCESSES_OR_SERVICES
PRESERVE_MODULEKEY_FACADE_IDENTITY_AUTHORITY_AND_CALLBACK_RULES
~~~

The inverse optimization is also possible when future runtimes gain stronger
safe in-process isolation.

## Intentionally deferred

PLAT053 deliberately does not pre-build or select:

~~~text
third-party provider registration API
provider hot loading/unloading
RuntimeHost-global foreign Engine pools
universal Future/Promise/awaitable bridging
retained/asynchronous callbacks
generic Actor/P foreign-value transfer
shared foreign service semantics between Actors
universal provider worker pools
transparent crash restart/replay
one distribution containing every foreign provider
foreign dependency acquisition policy
~~~

Those capabilities can be added through their proper semantic/platform owners
without changing the selected baseline.

## Ratified result

~~~text
PLAT053_STATUS=RATIFIED

PROVIDER_MODEL=
  PROCESS_OWNED_LAZY_FOREIGN_PROVIDER_COMPARTMENTS;
  PROVIDER_IS_A_LOGICAL_RUNTIME_AND_ACQUISITION_BOUNDARY_NOT_A_POLYGLOT_CONTEXT;
  MUTABLE_FOREIGN_APPLICATION_STATE_IS_ACTOR_ISOLATED_BY_DEFAULT;
  EACH_PROVIDER_MAY_REALIZE_AN_ACTOR_SESSION_WITH_CONTEXT_REALM_INTERPRETER_CLASSLOADER_ISOLATE_OR_EXTERNAL_SERVICE_AS_REQUIRED

PROVIDER_REGISTRY_OWNER=
  RUNTIMEHOST_OWNED_IMMUTABLE_HOST_SUPPLIED_REGISTRY;
  NOT_GUEST_MUTABLE;
  NOT_A_JVM_GLOBAL_SINGLETON;
  THIRD_PARTY_RUNTIME_REGISTRATION_API_DEFERRED

MODULEKEY_PROVIDER_CONTRACT=
  EXPLICIT_PROVIDER_ROUTING;
  PROVIDER_RESOLUTION_PRODUCES_STABLE_CANONICAL_TARGET_IDENTITY_BEFORE_MODULE_INITIALIZATION;
  MODULEKEY_INCLUDES_PROVIDER_IDENTITY_PLUS_PROVIDER_CANONICAL_TARGET_IDENTITY_AND_EXACT_ACQUISITION_PROVENANCE_WHERE_REQUIRED;
  NO_CONTEXT_VALUE_PROVIDER_CACHE_ENTRY_OR_ACTOR_SESSION_ID_IN_SEMANTIC_MODULE_IDENTITY;
  DEPENDENCY_ACQUISITION_REMAINS_SEPARATE

PROTOS_MODULE_CACHE_OWNER=
  ACTOR;
  FOREIGN_MODULE_INSTANCE_IS_THE_ACTOR_LOCAL_PROTOS_FACADE;
  CACHE_BEFORE_PROVIDER_INITIALIZATION;
  FAILURE_EVICTION_AND_RETRY_PRESERVED

FOREIGN_RUNTIME_CACHE_OWNER=
  PROVIDER_COMPARTMENT_FOR_IMMUTABLE_ARTIFACTS_AND_CODE;
  ACTOR_PROVIDER_SESSION_FOR_MUTABLE_LANGUAGE_RUNTIME_AND_MODULE_STATE;
  PROVIDER_CACHE_NEVER_REPLACES_PROTOS_ACTOR_MODULE_CACHE

ENGINE_TOPOLOGY=
  EXISTING_PROTOS_ENGINE_REMAINS_PROTOS_ONLY;
  EACH_USED_PROCESS_PROVIDER_COMPARTMENT_OWNS_ITS_PROVIDER_ENGINE_OR_EQUIVALENT_RUNTIME_SUBSTRATE_WHERE_REQUIRED;
  CROSS_PROCESS_FOREIGN_ENGINE_SHARING_IS_DEFERRED_PROOF_GATED_OPTIMIZATION

CONTEXT_TOPOLOGY=
  NO_UNIVERSAL_FOREIGN_CONTEXT;
  NO_FOREIGN_LANGUAGE_IN_EXISTING_PROTOS_PROCESS_CONTEXT_BY_DEFAULT;
  STATEFUL_TRUFFLE_PROVIDERS_DEFAULT_TO_LAZY_ACTOR_ISOLATED_CONTEXT_OR_EQUIVALENT_SESSION;
  PROVIDER_SELECTED_PHYSICAL_TOPOLOGY_MUST_PROVE_ACTOR_ISOLATION_PLAT052_NON_AMPLIFICATION_AND_D189_CONFORMANCE

PROVIDER_LIFETIME=
  PROCESS_OWNED_PROVIDER_COMPARTMENT;
  LAZY_ON_FIRST_PROCESS_USE;
  ACTOR_SESSION_LAZY_ON_FIRST_ACTOR_USE;
  SESSION_RELEASE_AFTER_ACTOR_TERMINATION_AND_CALLBACK_REVOCATION;
  ALL_PROVIDER_COMPARTMENTS_CLOSE_WITH_PROCESS_TERMINAL_LIFECYCLE

FOREIGN_VALUE_LIFETIME=
  BOUND_TO_EXACT_PROVIDER_ACTOR_SESSION_AND_TARGET_GENERATION;
  CLOSED_SESSION_OR_GENERATION_NEVER_REBINDS_OLD_REFERENCES

THREADING_PROFILE_MODEL=
  PROVIDER_SELECTED_CONCURRENT_SERIALIZED_DEDICATED_CARRIER_OR_EXTERNAL_SERVICE;
  SERIALIZATION_LOCAL_TO_RELEVANT_SESSION;
  NO_PROTOS_PROCESS_GIL;
  FOREIGN_THREADS_GAIN_NO_PROTOS_ENTRY_AUTHORITY

LAZY_INITIALIZATION=
  MANDATORY;
  UNUSED_PROVIDER_CREATES_NO_ENGINE_CONTEXT_INTERPRETER_ISOLATE_OR_CHILD_PROCESS;
  UNUSED_ACTOR_PROVIDER_PAIR_CREATES_NO_SESSION

AUTHORITY_PROFILE_INTEGRATION=
  RESTRICTED_IN_PROCESS_OR_TRUSTED_IN_PROCESS_OR_STRONGLY_ISOLATED_OR_UNAVAILABLE;
  PROFILE_IS_PER_REAL_PROVIDER_LIBRARY_FEATURE_CONFIGURATION;
  TRUSTED_HOST_AUTHORITY_DOES_NOT_WAIVE_ACTOR_ISOLATION;
  PLAT052_GATE_APPLIES_BEFORE_FOREIGN_INITIALIZATION

STRONGER_ISOLATION_TOPOLOGY_BOUNDARY=
  PHYSICAL_REALIZATION_OF_THE_SAME_LOGICAL_ACTOR_SESSION_CONTRACT;
  MAY_USE_ISOLATE_EXTERNAL_PROCESS_OR_SERVICE;
  NO_TRANSPARENT_OPERATION_REPLAY_AFTER_CRASH

JVM_PROVIDER_AVAILABILITY=
  OPTIONAL_EXACT_COMPONENTS;
  HOST_JAVA_DISTINCT_FROM_ESPRESSO;
  PROVIDER_PROFILE_SPECIFIC

NATIVE_PROVIDER_AVAILABILITY=
  ARTIFACT_SPECIFIC_CAPABILITY_MATRIX;
  NO_REQUIREMENT_TO_SUPPORT_EVERY_JVM_PROVIDER_IN_NATIVE;
  PLAT045_PRESERVED;
  EXTERNAL_STRONGLY_ISOLATED_PROVIDERS_ALLOWED

DISTRIBUTION_COST_MODEL=
  EXACT_PROVIDER_COMPONENTS_ONLY;
  RUNTIME_LAZINESS_AND_DISTRIBUTION_SIZE_ARE_SEPARATE_COSTS

PAY_AS_YOU_GROW_GATE=
  ZERO_FOREIGN_USE_IMPLIES_ZERO_FOREIGN_RUNTIME_INITIALIZATION;
  PROVIDER_COST_ONLY_AFTER_FIRST_PROCESS_USE;
  ACTOR_SESSION_COST_ONLY_AFTER_FIRST_ACTOR_USE_OF_THAT_PROVIDER;
  STRONGER_ISOLATION_COST_ONLY_FOR_PROFILES_THAT_REQUIRE_IT

RECOMMENDED_CANDIDATE=
  B_PRIME_PROCESS_OWNED_LAZY_PROVIDER_COMPARTMENTS_WITH_ACTOR_ISOLATED_SESSIONS_AND_PROVIDER_SELECTED_PHYSICAL_TOPOLOGY

D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
NEXT_REQUIRED_PLAT053_SLICE=NONE
IMPLEMENTATION_AUTHORIZED=NO
~~~

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed PLAT053 GITHUB010 investigation, current Protos and
platform evidence, and the project owner's explicit approval. No independent
human review is claimed by this record.
