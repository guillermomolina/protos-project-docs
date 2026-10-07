# PLAT053 — owner approval and ratification evidence

Status: **APPROVAL PROVENANCE RECORDED — CANDIDATE B′ SELECTED**

Formal decision: `guillermomolina/protos#822` — PLAT053

Parent audit: `guillermomolina/protos#818` — AUD019

Approval date: **2026-10-07**

## Approval chain

PLAT053 completed its GITHUB010 investigation as research only. The owner-review
packet compared:

~~~text
A  — every foreign language in the existing Protos Process Context
B0 — one lazy foreign Context/runtime per provider and Process
B′ — Process-owned lazy provider compartments with Actor-isolated logical
     sessions and provider-selected physical topology
C  — one foreign Polyglot Context per Actor as a universal rule
D  — every foreign runtime out of process
E  — defer / no foreign execution
~~~

The recommendation was:

~~~text
RECOMMENDED_CANDIDATE=
  B_PRIME_PROCESS_OWNED_LAZY_PROVIDER_COMPARTMENTS_WITH_ACTOR_ISOLATED_SESSIONS_AND_PROVIDER_SELECTED_PHYSICAL_TOPOLOGY
~~~

The project owner then explicitly answered:

> Apruebo B'

This is exact-candidate approval of the Candidate B′ immediately presented for
PLAT053.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=EXPLICIT
SELECTED_CANDIDATE=B_PRIME_PROCESS_OWNED_LAZY_PROVIDER_COMPARTMENTS_WITH_ACTOR_ISOLATED_SESSIONS_AND_PROVIDER_SELECTED_PHYSICAL_TOPOLOGY
~~~

## Selected boundary

The approved architecture establishes:

~~~text
Process owns lazy provider compartments

provider is a logical runtime/acquisition boundary
and is not synonymous with Polyglot Context

mutable foreign application/runtime state is Actor-isolated by default

each Actor/provider pair lazily receives a logical provider session when needed

provider chooses the smallest correct physical topology:
  Context
  realm / inner Context
  interpreter
  class loader / guest VM
  isolate
  external process/service
  or equivalent

immutable implementation artifacts may be shared when semantically invisible

existing Protos Process Context is not the universal foreign Context

foreign runtime caches never replace the Actor-local Protos module cache

foreign values remain bound to the exact provider/session/generation that
created them and never silently rebind after close

provider threading/serialization is local to the provider/session and never
creates a Process-wide Protos GIL

PLAT052 authority profiles apply before foreign initialization

unused providers impose no foreign runtime initialization cost
~~~

The full selected contract is preserved in:

~~~text
docs/project/decisions/platform/PLAT053_FOREIGN_PROVIDER_RUNTIME_LIFECYCLE_ARCHITECTURE.md
~~~

## GITHUB021 invariant/delta consistency

PLAT053 had no earlier owner-approved candidate-specific invariant that is
contradicted or reopened by B′.

The fixed Issue constraints and prerequisite decisions were rechecked.

Candidate B′ preserves:

~~~text
generic foreign-value substrate = YES
generic all-language module importer = NO
per-language/module-system provider = YES
dependency acquisition remains separate
foreign runtime cache != Actor module cache
unused provider/language near-zero runtime cost
host Java != Espresso
PLAT045 Native contract unchanged
~~~

It also preserves PLAT052's owner-approved hard gate:

~~~text
AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT
~~~

The B -> B′ refinement does not add observable language semantics. The added
Actor-isolated logical session rule is required by the already-normative
foreign-mutable-state Actor-isolation contract: distinct Actor-local facades
cannot expose unrestricted shared mutable Java/native/language-global state
merely because that state lives outside the Protos heap.

The refinement therefore changes physical hosting architecture while preserving
existing semantics.

~~~text
PLAT053_FIXED_CONSTRAINT_DELTA=NONE
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
ACTOR_ISOLATION_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Repository and platform evidence

The investigation inspected the current Protos hosting/module boundaries,
including:

~~~text
ProtosPolyglotRuntimeHost
ProtosPolyglotProcessContext
ProtosPolyglotExecutionContext
ProtosLanguageContext
ProtosModuleRuntime
ProtosActorModuleState
ProtosModuleKey
ProtosSourceReadabilityAuthority
ProtosStandaloneHostedSession
module resolvers and Process/Actor lifecycle
~~~

The current implementation already separates:

~~~text
shared Engine
from
one Process-owned Protos Context

Actor-local module cache
from
host resolver/source machinery
~~~

PLAT053 retains those distinctions and adds the provider/session layer instead
of treating existing Context identity as foreign-provider identity.

The comparative investigation covered representative current GraalVM/Truffle
requirements for GraalJS, GraalPy, TruffleRuby, Espresso, LLVM/Sulong and
SimpleLanguage, plus relevant Pkl, Wasmtime and Deno architecture evidence.

The decisive recurring pattern was:

~~~text
share implementation/code infrastructure where safe
but keep mutable execution state, authority and lifecycle in a narrower owner
~~~

The provider-specific findings also showed that one language name can have
multiple materially different authority/runtime profiles, especially managed
versus native execution.

## Candidate falsification

The selected B′ model was checked against at least these cases:

~~~text
two Actors import the same Python module
one Actor uses two providers with different authority
Protos Context holds PLAT022 source-readability authority while JS must not
Python pure versus Python native extension
arbitrary Java library performs Files.write internally
provider Context closes while foreign values remain reachable
retained callback wrapper survives physical provider close
provider initialization fails and a later import retries
10,000 Actors exist while only one uses Python
many independent Processes share one RuntimeHost
provider available on JVM but not Native
provider owns internal threads
isolated provider process crashes
dependency exists but authority profile rejects execution
unused providers must remain uninitialized
foreign loader/runtime has sticky failed-module state
~~~

The sticky-failure case is an explicit implementation proof obligation: the
provider may need a finer private realm/loader/target generation so failed
initialization can retry without destroying unrelated still-live foreign
objects.

## Comparative result

The GITHUB010 scorecard was:

~~~text
A   = 29/60
B0  = 41/60
B′  = 55/60
C   = 35/60
D   = 38/60
E   = 42/60
~~~

Hard semantic/authority/isolation constraints override the arithmetic score.
A and B0 fail required boundaries; D remains a valid stronger-isolation profile
inside B′; C remains a valid physical realization for selected providers rather
than a universal rule.

## Strongest counterargument and regret case

Strongest counterargument: if thousands of Actors use one heavyweight foreign
runtime, Actor-isolated sessions may cost significantly more memory and warmup
than one Process-global foreign runtime.

That cost does not justify silently changing Actor isolation. B′ preserves
optimizations such as shared Engines/code, immutable package metadata,
copy-on-write internals or provider-native isolated realms while retaining
logical ownership.

Regret case:

~~~text
MOST_REAL_FOREIGN_ECOSYSTEMS_NEEDED_BY_PROTOS_REQUIRE_HEAVY_NATIVE_OR_GLOBAL_STATE
AND_ACTOR_LOCAL_IN_PROCESS_SESSIONS_ARE_NOT_PRACTICAL
~~~

Escape path:

~~~text
KEEP_THE_LOGICAL_PROVIDER_SESSION_CONTRACT
MOVE_AFFECTED_PROVIDERS_TO_ISOLATES_PROCESSES_OR_SERVICES
PRESERVE_D188_D189_PLAT052_AND_MODULE_IDENTITY
~~~

No observable Protos semantic redesign is required.

## Revision revalidation

The PLAT053 investigation started from:

~~~text
START_PROTOS_REVISION=b10569680b1532b277ba3ce49eced2e818833d93
START_PROTOS_VERSION=0.3.268-SNAPSHOT
START_SPECIFICATION_REVISION=0.1.446
~~~

Before ratification publication, live Protos was revalidated at:

~~~text
RATIFICATION_PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
RATIFICATION_PROTOS_VERSION=0.3.269-SNAPSHOT
RATIFICATION_SPECIFICATION_REVISION=0.1.446
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

The intervening product commit is:

~~~text
246a24994d74ca084dc43ceddd4c400c2af94b15
  LIB020-C: add pure std:text/ColorMode resolution policy
~~~

Its changed paths are confined to `std:text/ColorMode`, its conformance test,
manifest, version and changelog. It does not alter foreign providers, module
resolution, Actor isolation, Process/Context hosting, D188/D189, PLAT052 or
GraalVM runtime architecture.

~~~text
RATIFICATION_DRIFT=MATERIAL_TO_PLAT053:NO
~~~

## Project-record revision coupling

This publication is prepared from:

~~~text
PROJECT_RECORD_BASE_REVISION=084061dc2e7d11d85f74dc53a5217d2004694ea9
~~~

The exact final project-record revision containing this evidence, the PLAT053
decision record and the platform registry reference is recorded on the
authoritative GitHub Issue after publication.

## Closure and next routing

PLAT053 is the final AUD019 decision packet. Exact owner approval plus durable
ratification publication are sufficient to close PLAT053 as completed.

~~~text
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
NEXT_REQUIRED_PLAT053_SLICE=NONE
~~~

AUD019's established ordering ends with:

~~~text
D188 -> D189 -> PLAT052 -> PLAT053 -> implementation
~~~

No foreign-interop implementation `Ixxx` was found already allocated during
ratification revalidation. PLAT053 therefore does not invent or silently
authorize an implementation Issue or slice.

~~~text
NEXT_REQUIRED_PLAT053_SLICE=NONE
FOLLOW_ON_IMPLEMENTATION=REQUIRES_SEPARATE_COORDINATION_OWNER
IMPLEMENTATION_AUTHORIZED=NO
~~~

## Validation class

This publication changes only durable non-normative project documentation.
No Protos source, normative specification, implementation version or product test
is changed.

The publication is created as one revision over the current
`protos-project-docs` head and is re-read at that exact published revision
before Issue closure.

## AI-assistance disclosure

This evidence record was materially prepared with AI assistance from ChatGPT
from the completed PLAT053 investigation, live repository state and the project
owner's explicit approval interaction. No independent human review is claimed.
