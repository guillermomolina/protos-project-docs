# PLAT052 — Foreign authority and sandbox enforcement

Status: **RATIFIED**

Selected architecture: **Candidate B′ — capability-preserving deny-by-default with provider enforcement gate**.

Approval: explicit project-owner approval on 2026-10-07:

~~~text
apruebo B'
~~~

Decision Issue: `guillermomolina/protos#821`

Parent audit: `AUD019 / guillermomolina/protos#818`

Direct semantic prerequisites:

~~~text
D188 / #819 = SATISFIED
D189 / #820 = SATISFIED
D189_NORMATIVE_SPECIFICATION_REVISION = 0.1.446
~~~

Ratification revalidation:

~~~text
PROTOS_REVISION=b10569680b1532b277ba3ce49eced2e818833d93
PROTOS_VERSION=0.3.268-SNAPSHOT
SPECIFICATION_REVISION=0.1.446
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative platform/security architecture. PLAT052 does not
create or revise observable Protos semantics. D188 owns foreign-value semantics,
D189 owns the synchronous callback contract, and the existing I/O/concurrency
specification owns Protos capability/isolation semantics.

## Decision

Foreign execution begins with **zero ambient host authority**.

An explicitly granted Protos capability may be mapped to foreign execution only
through an enforcement mechanism that preserves the capability's authority
boundary and does not make that authority ambient to unrelated foreign code.

A provider, foreign runtime, library, module or feature may execute in-process
under restricted mode only when the implementation can show that every relevant
host-effect channel available to that execution cannot amplify the authority
explicitly provisioned to its foreign compartment.

When that proof cannot be established, the conforming choices are:

~~~text
explicit trusted host/embedder policy
OR stronger provider-specific isolation
OR fail closed / unavailable
~~~

Import, dependency presence, language availability and successful module
resolution are never authority grants.

## Fixed invariants

The selected architecture preserves these already-established constraints:

~~~text
IMPORT_IS_AUTHORITY_GRANT=NO
PROCESS_IMPLIES_FILESYSTEM=NO
PROCESS_IMPLIES_NETWORK=NO
PROCESS_IMPLIES_SUBPROCESS=NO
PROCESS_IMPLIES_NATIVE_ACCESS=NO
DEPENDENCY_PRESENCE_IS_EXECUTION_AUTHORITY=NO
FOREIGN_MUTABLE_STATE_BYPASSES_ACTOR_ISOLATION=NO
ALLOW_ALL_ACCESS_DEFAULT=NO
D188_DELTA=NONE
D189_DELTA=NONE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

The provider enforcement gate is therefore an implementation proof obligation,
not a new guest-visible capability or permission system.

## Authority granularity

Protos capabilities are possession-based semantic values. GraalVM Context
permissions are often ambient to the Context/compartment.

PLAT052 forbids silently replacing a narrow possession-based Protos capability
with a wider ambient permission whenever doing so would make the capability
usable by code that did not receive it.

Conceptually:

~~~text
AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT
~~~

If one concrete Protos `Filesystem`, `Network`, standard stream, or future
effect capability cannot be represented faithfully by a Context-wide switch,
the implementation must use a capability-aware adapter/broker, choose an
appropriately scoped compartment under PLAT053, or reject the operation/provider
in restricted mode.

## Provider code-loading authority

A provider may need implementation authority to read already-selected runtime
artifacts, language resources, JARs, modules, scripts, bitcode or package
metadata.

That authority is distinct from application authority:

~~~text
PROVIDER_CODE_LOADING_AUTHORITY=
  IMPLEMENTATION_INTERNAL
  + SCOPED_TO_ALREADY_SELECTED_PROVIDER_ARTIFACTS
  + NON_EXPORTABLE_TO_FOREIGN_CODE
  + NOT_A_PROTOS_FILESYSTEM_OR_NETWORK_CAPABILITY
~~~

Loading a dependency does not grant the loaded code the mechanism used to load
it.

Foreign module initialization itself is inside the enforcement boundary.
Provider/module startup must not run first and receive restrictions only after
initialization effects have already happened.

## Host access

Restricted mode starts with no general host-object/member authority.

Host adapters exposed to foreign code are trusted-computing-base entries and
must be narrow and capability-aware. Merely limiting which host methods are
callable is not proof that those methods preserve Protos authority: an allowed
host method may internally perform filesystem, network, process, reflection,
classloading, native or environment effects.

Therefore:

~~~text
HOST_ACCESS_POLICY=
  NONE_BY_DEFAULT;
  NARROW_EXPLICIT_CAPABILITY_AWARE_ADAPTERS_ONLY

ARBITRARY_HOST_METHOD_EXPOSURE_IS_AUTHORITY_CONFINEMENT=NO
ARBITRARY_HOST_OBJECT_EXPOSURE_IS_AUTHORITY_CONFINEMENT=NO
~~~

## Host class lookup, classloading and reflection

Guest-visible host-class lookup is disabled by default.

Provider-internal resolution of an already-provisioned dependency may use host
implementation mechanisms only when that resolution does not become an ambient
guest service locator or bypass the selected authority profile.

Arbitrary host Java libraries are not considered safely capability-confined
merely because direct guest class lookup is disabled: ordinary JVM code can
perform host effects internally.

~~~text
HOST_CLASS_LOOKUP_POLICY=
  DISABLED_FOR_GUEST_BY_DEFAULT

ARBITRARY_HOST_JAVA_RESTRICTED_IN_PROCESS=
  ONLY_WITH_A_REAL_NON_AMPLIFICATION_PROOF

REFLECTION_OR_CLASSLOADING_RECOVERY_OF_HOST_AUTHORITY=FORBIDDEN
~~~

A Java library/provider that cannot meet the restricted contract requires
trusted mode, stronger isolation, or is unavailable.

## Filesystem, I/O and standard streams

There is no ambient host filesystem authority in restricted mode.

The existing PLAT022 source-readability architecture is retained as a local
precedent: exact implementation-required source paths may be admitted without
granting a general guest filesystem.

A Protos `Filesystem` may be mapped only in a way that preserves its exact
authority/confinement semantics. An unrestricted host filesystem switch is not a
substitute for a restricted `Filesystem`.

Standard input/output/error are likewise explicit hosted resources. Sharing a
foreign Context must not implicitly expose host stdio or a broader Process
environment than the foreign compartment was provisioned.

~~~text
IO_POLICY=
  NO_AMBIENT_HOST_IO;
  EXACT_CAPABILITY_PRESERVING_ADAPTER_OR_COMPARTMENT_REQUIRED
~~~

## Network

No network authority is ambient by implication.

A concrete Protos `Network` may represent a restricted routing/policy domain.
Enabling general host socket access is therefore not a faithful implementation
when it grants destinations or operations outside that capability.

~~~text
GENERAL_HOST_SOCKET_ACCESS_FROM_NETWORK_CAPABILITY=NO
~~~

Use a scope-preserving adapter/broker, a compartment whose complete ambient
network authority is no broader than the granted capability, or fail closed.

## Process/subprocess authority

The standard Protos Process capability does not grant arbitrary child-process
creation.

Restricted foreign code receives no subprocess/process-launch authority by
default.

~~~text
PROCESS_POLICY=
  EXTERNAL_PROCESS_CREATION_DENIED_BY_DEFAULT
~~~

Any future subprocess authority requires its own explicit semantic capability
and an implementation that preserves it; PLAT052 does not create that facility.

## Environment and system properties

Restricted mode does not inherit the host environment or system properties as
ambient authority.

Values explicitly provisioned by existing Protos/host semantics may be supplied
as inert data/snapshots where faithful. That does not grant an environment API
capable of reading unrelated host state.

## Native access and foreign native libraries

Native access is denied by default and is never inferred from import, Process,
dependency presence or provider availability.

A native extension/library that can directly use operating-system or process
authority outside the Protos capability boundary fails the restricted in-process
gate.

~~~text
NATIVE_ACCESS_POLICY=
  DENIED_BY_DEFAULT;
  TRUSTED_MODE_OR_STRONGER_ISOLATION_REQUIRED_WHEN_UNMEDIATED_NATIVE_ACCESS_IS_REQUIRED
~~~

This applies equally to JNI/JNA-like host Java paths, Python/Ruby native
extensions, native LLVM execution and comparable mechanisms.

Provider modes whose native implementation surface is demonstrably mediated by
the runtime may qualify independently; the provider must prove the relevant
authority profile rather than inherit trust from its language name.

## Cross-language / Polyglot access

Cross-language execution and polyglot bindings are not ambient.

~~~text
POLYGLOT_ACCESS_POLICY=
  NONE_BY_DEFAULT
~~~

A future provider relationship may authorize exact directed cross-language
access when required, but it must remain inside the same non-amplification
contract.

Restricted compartments default to no implicit cross-Context value sharing.
Any bridge that does share values must preserve D188 identity/admission semantics
and PLAT052 authority constraints explicitly.

## Threads and callbacks

PLAT052 does not alter D189.

A thread created by foreign code, a foreign worker pool, event loop or native
thread gains no Protos Actor/Task authority by possessing a synchronous callback
wrapper.

D189's same-Actor/same-Task dynamic synchronous callback contract, rejection of
foreign-thread ingress, no concurrent Protos entry, no late callback and no
resurrection remain mandatory.

Host/runtime callback-scoping features may assist implementation but are not
semantic authority and do not replace the D189 lifetime checks.

## Trusted mode

Trusted mode is allowed only as an explicit host/embedder policy.

~~~text
TRUSTED_MODE_POLICY=
  EXPLICIT_HOST_OR_EMBEDDER_OPT_IN_ONLY

IMPORT_CAN_ENABLE_TRUSTED_MODE=NO
DEPENDENCY_CAN_ENABLE_TRUSTED_MODE=NO
FOREIGN_CODE_CAN_UPGRADE_ITSELF=NO
ALLOW_ALL_ACCESS_IS_BASELINE=NO
~~~

Trusted mode may intentionally expose broader host authority, but the choice is
outside ordinary foreign imports and must not be confused with the restricted
Protos capability-preserving contract.

## Stronger isolation

PLAT052 does not require every provider to execute out of process.

It does require stronger isolation or fail-closed behavior whenever the selected
provider/library/feature cannot preserve the authority contract in-process.

Possible mechanisms include a more strongly sandboxed runtime mode, isolated
VM/isolate, operating-system process/service with brokered capabilities, or
another mechanism that provides an equivalent non-amplification proof.

The exact Context/Engine/process/service topology, pooling and lifecycle are
deliberately deferred to PLAT053.

~~~text
STRONGER_ISOLATION_POLICY=
  MANDATORY_OR_FAIL_CLOSED_WHEN_RESTRICTED_IN_PROCESS_NON_AMPLIFICATION_CANNOT_BE_PROVED
~~~

Out-of-process execution is not itself assumed to be a sandbox: inherited host
filesystem/network/environment/process authority must still be confined.

## Provider implications

The decision deliberately permits different physical profiles behind one
Protos-side contract.

Representative consequences from the investigation:

- GraalJS-style guest execution can be a strong in-process restricted candidate
  when host lookup/access, I/O, process, native and cross-language escape paths
  remain disabled or capability-mediated.
- GraalPy without unmediated native extensions may qualify subject to the exact
  runtime profile; native C-extension use does not automatically qualify.
- TruffleRuby may qualify only under a provider profile that disables or
  mediates its native/system escape paths.
- arbitrary host-Java libraries do not qualify merely by configuring
  `HostAccess`; their internal JVM effects must be trusted, mediated or
  isolated.
- Espresso is a distinct provider model from host Java and must prove its own
  native/library authority requirements.
- LLVM/Sulong managed execution and native execution are distinct authority
  profiles; native execution is not treated as restricted merely because the
  call originated from a Truffle language.

These are architectural admission rules, not promises that every named provider
or ecosystem is supported by the first implementation.

## Candidate disposition

### Candidate A — trusted/all-access default

Rejected.

It conflicts directly with the existing Protos capability model and the fixed
PLAT052 constraints. It also turns import into an ambient authority escalation.

### Candidate B — deny-by-default in-process mapping

The deny-by-default direction is retained, but the simple form is insufficient.

Context flags alone do not prove capability preservation for host methods,
arbitrary host Java, native extensions, provider-global state, or the mismatch
between object-scoped Protos authority and Context-scoped ambient permissions.

### Candidate B′ — capability-preserving deny-by-default with provider enforcement gate

**Selected.**

B′ retains the minimal in-process path where it can be proved correct, adds no
guest-visible security institution, and fails closed or escalates physical
isolation only for providers/features that need it.

### Candidate C — out-of-process isolation for every foreign runtime

Rejected as the universal baseline.

It can be an appropriate B′ enforcement mechanism for selected providers, but
making it mandatory for all providers imposes IPC, identity/proxy, lifecycle,
callback, serialization, pooling and startup costs even when a faithful
in-process profile exists. It also does not guarantee confinement unless the
child process itself is sandboxed/brokered correctly.

## Strongest argument against B′

B′ permits provider-specific enforcement profiles rather than one universal
physical sandbox. That can make support less uniform: one language/library may
work in restricted in-process mode while another requires isolation or trusted
mode.

The alternative universal out-of-process architecture is conceptually more
uniform.

The project accepts B′ because universal isolation would preselect substantial
PLAT053 machinery and fixed cost before evidence requires it. The observable
authority contract remains uniform even when the physical enforcement differs.

## Regret scenario and escape path

Regret scenario:

~~~text
EARLY_CRITICAL_FOREIGN_ECOSYSTEMS_REQUIRE_UNMEDIATED_NATIVE_OR_HOST_AUTHORITY
AND_MOST_REAL_PROVIDERS_END_UP_REQUIRING_STRONGER_ISOLATION
~~~

Escape path:

~~~text
KEEP_PLAT052_AUTHORITY_CONTRACT
MOVE_THOSE_PROVIDERS_BEHIND_PLAT053_ISOLATED_PROVIDER_SERVICES_OR_POOLS
PRESERVE_D188_AND_D189
~~~

The reverse migration is also valid: if future runtimes provide stronger
in-process confinement, a provider may move from isolation back to in-process
without changing PLAT052's guest-visible contract.

## Pay-as-you-grow and portability

Programs that never use foreign interoperability must not initialize foreign
providers or pay material foreign-sandbox runtime cost merely because Protos
supports the feature.

A provider that is never used should impose near-zero runtime cost aside from
bounded metadata needed for provider discovery/routing.

The decision is backend-neutral at its architectural boundary. GraalVM
`Context.Builder`, `HostAccess`, `IOAccess`, `PolyglotAccess` and
`SandboxPolicy` are implementation tools, not Protos institutions. An
alternate runtime must provide an equivalent non-amplification guarantee with
its own mechanisms.

## Implementation and proof obligations

PLAT052 is a decision owner, not an implementation authorization.

Later provider/runtime implementation must include negative proof for the
effect channels relevant to each supported provider, including as applicable:

~~~text
filesystem
network
subprocess/process creation
environment/system properties
reflection
host class lookup/classloading
native/FFI loading
cross-language access
host object/method escape paths
provider initialization side effects
inner Context privilege amplification
foreign-thread callback ingress
post-close/late authority use
Actor/P mutable-state leakage
~~~

Tests must prove both denial and explicitly authorized positive paths without
turning unrelated providers/modules into holders of the same authority.

## PLAT053 handoff

PLAT053/#822 is the next AUD019 decision.

It must choose provider/Engine/Context/session/process topology subject to this
hard gate:

~~~text
AUTHORITY_OF_FOREIGN_COMPARTMENT
    <=
AUTHORITY_EXPLICITLY_PROVISIONED_TO_THAT_COMPARTMENT
~~~

In particular, PLAT053 may choose a shared Context only when it proves that the
sharing does not make implementation/source-readability or another compartment's
capabilities available to unrelated foreign code.

PLAT053 also owns exact stronger-isolation topology, provider registry/lifetime,
threading profiles, lazy initialization, Native/provider availability and
distribution cost.

## Ratified result

~~~text
PLAT052_STATUS=RATIFIED

DEFAULT_AUTHORITY_MODEL=
  DENY_BY_DEFAULT_EXPLICIT_CAPABILITY_MAPPING_WITH_PROVIDER_ENFORCEMENT_GATE

HOST_ACCESS_POLICY=
  NONE_BY_DEFAULT;
  ONLY_NARROW_EXPLICIT_CAPABILITY_AWARE_HOST_ADAPTERS;
  ARBITRARY_HOST_METHOD_OR_OBJECT_EXPOSURE_IS_NOT_AUTHORITY_CONFINEMENT

HOST_CLASS_LOOKUP_POLICY=
  DISABLED_FOR_GUEST_BY_DEFAULT;
  PROVIDER_INTERNAL_DEPENDENCY_RESOLUTION_MUST_NOT_BECOME_GUEST_CLASS_LOOKUP_OR_AMBIENT_AUTHORITY;
  ARBITRARY_HOST_JAVA_REQUIRES_TRUSTED_MODE_OR_STRONGER_ISOLATION_WITHOUT_A_REAL_NON_AMPLIFICATION_PROOF

IO_POLICY=
  NO_AMBIENT_HOST_IO;
  PROVIDER_CODE_LOADING_AUTHORITY_IS_SEPARATE_NON_EXPORTABLE_IMPLEMENTATION_AUTHORITY;
  FILESYSTEM_NETWORK_AND_STANDARD_STREAM_ACCESS_REQUIRE_EXACT_CAPABILITY_PRESERVING_ADAPTER_OR_COMPARTMENT;
  CONTEXT_WIDE_PERMISSION_MUST_NOT_BROADEN_POSSESSION_BASED_AUTHORITY

PROCESS_POLICY=
  NO_SUBPROCESS_AUTHORITY_FROM_PROCESS;
  EXTERNAL_PROCESS_CREATION_DENIED_BY_DEFAULT;
  FUTURE_SUBPROCESS_AUTHORITY_REQUIRES_AN_EXPLICIT_SEPARATE_CAPABILITY_AND_FAITHFUL_ENFORCEMENT

NATIVE_ACCESS_POLICY=
  DENIED_BY_DEFAULT;
  NEVER_DERIVED_FROM_IMPORT_PROCESS_OR_DEPENDENCY_PRESENCE;
  PROVIDERS_OR_LIBRARIES_REQUIRING_UNMEDIATED_NATIVE_ACCESS_REQUIRE_TRUSTED_MODE_STRONGER_ISOLATION_OR_FAIL_CLOSED

POLYGLOT_ACCESS_POLICY=
  NONE_BY_DEFAULT;
  NO_IMPLICIT_CROSS_LANGUAGE_EVAL_OR_BINDINGS;
  ANY_FUTURE_ACCESS_MUST_BE_EXACT_DIRECTED_AND_EXPLICIT;
  RESTRICTED_COMPARTMENTS_DEFAULT_TO_NO_IMPLICIT_CROSS_CONTEXT_VALUE_SHARING

TRUSTED_MODE_POLICY=
  EXPLICIT_HOST_OR_EMBEDDER_OPT_IN_ONLY;
  NEVER_SELECTABLE_OR_UPGRADABLE_BY_IMPORT_FOREIGN_CODE_OR_DEPENDENCY;
  ALLOW_ALL_ACCESS_IS_NEVER_THE_BASELINE

STRONGER_ISOLATION_POLICY=
  MANDATORY_OR_FAIL_CLOSED_WHEN_A_PROVIDER_LIBRARY_OR_FEATURE_CANNOT_PROVE_NON_AMPLIFICATION_IN_PROCESS;
  EXACT_CONTEXT_PROCESS_ISOLATE_OR_SERVICE_TOPOLOGY_IS_DEFERRED_TO_PLAT053

RECOMMENDED_CANDIDATE=
  B_PRIME_CAPABILITY_PRESERVING_DENY_BY_DEFAULT_WITH_PROVIDER_ENFORCEMENT_GATE

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE_REQUIRED=NO
IMPLEMENTATION_AUTHORIZED=NO
NEXT_REQUIRED_PLAT052_SLICE=NONE
NEXT_WORK_ITEM=PLAT053/#822
~~~

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed PLAT052 GITHUB010 investigation, current repository
and platform evidence, and the project owner's explicit approval. No independent
human review is claimed by this record.
