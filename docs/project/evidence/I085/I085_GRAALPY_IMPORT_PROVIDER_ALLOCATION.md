# I085 — GraalPy foreign-import provider allocation and AUD019 second-branch handoff

Date: 2026-10-08

Live issue: https://github.com/guillermomolina/protos/issues/839

Parent audit: https://github.com/guillermomolina/protos/issues/818

## Why this follow-up exists

AUD019 investigated **two** coordinated deliverables:

1. an explicit interoperability Standard Library interface; and
2. ordinary Protos imports of real libraries from non-Java Truffle languages.

The first deliverable is complete under LIB021/#835 (`std:interop`).
I082/#830 already supplied the shared foreign-module/value/runtime substrate
and the first restricted host-Java provider for `java:`. Its exact closure
record expressly deferred further GraalPy/GraalJS/TruffleRuby language
providers as independently closable work.

Consequently, closing I082 and LIB021 does **not** prove that
`import("python:numpy")` is implemented.

## Existing exact product and decision authority

~~~text
AUD019=#818 COMPLETED_RESEARCH
D188=#819 COMPLETED_AND_NORMATIVE
D189=#820 COMPLETED_AND_NORMATIVE
PLAT052=#821 RATIFIED
PLAT053=#822 RATIFIED
I082=#830 COMPLETED
I082_FINAL_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
LIB021=#835 COMPLETED
LIB021_FINAL_REVISION=ac1e660cc37f8629852062fee41dcc33cf0758f8
IMPLEMENTATION_VERSION=0.3.281-SNAPSHOT
SPECIFICATION_REVISION=0.1.448
NEW_WORK=I085/#839
FIRST_SLICE=I085-A
FIRST_SLICE_TYPE=IMPLEMENTATION
TARGET_REPOSITORY=guillermomolina/protos
~~~

## User-facing target and semantics

Conceptual goal: a module written in another supported guest language should
be importable and usable through *ordinary Protos* syntax as far as D188
permits, with `std:interop` as the explicit escape hatch.

~~~protos
np: import("python:numpy")
arr: np.array([1, 2, 3])
~~~

The specifier is a semantic Protos String under
`spec/semantics/MODULES.md`. This allocation does not add syntax such as
`import(numpy)` for an unbound identifier.

One generic Truffle `InteropLibrary`-based foreign-value substrate already
exists; Python acquisition still requires the distinct language/provider
implementation ratified by PLAT053. Keep the Actor-local facade/ModuleKey
identity, session/generation lifetime, callback and Error rules unchanged.

## Planned implementation

I085-A owns a real optional GraalPy `python:` import provider using the
existing `ProtosForeignModuleProvider`, provider factory/descriptor/registry,
`ProtosForeignValueAdapter`, Actor-local import facades, and D188/D189
projection. The first end-to-end case should use a genuine built-in Python
module, establishing module acquisition, ordinary calls, identity,
lifecycle, authority and zero-use behavior.

I085-B owns real NumPy conformance with an explicitly pre-provisioned
GraalPy-compatible Python package environment. Cover representative
`np.array` construction and faithful ordinary Protos projections; where
mapping is not faithful, use existing `std:interop` or deliberate adapter
logic rather than changing Protos semantics.

NumPy is a valuable acceptance workload because its native extension
dependencies test actual distribution, runtime and authority boundaries.
GraalPy cannot consume arbitrary CPython-ABI native wheels unchanged.
Use its own package-provisioning workflow rather than claiming `pip`
compatibility alone suffices. Provisioning/download/build may be a separately
approved host setup action, **not** a side effect of `import`.

PLAT052 requires zero ambient host authority. Neither a Python import nor the
presence of NumPy grants filesystem, network, process, Java, or native execution
rights. An in-process native-capable NumPy execution path may require explicit
host trust or stronger isolation; do not describe it as restricted unless
confinement is technically proven. Preserve PLAT053 lazy compartments so a
Protos program not using Python pays no compulsory Python runtime cost.

GraalJS and TruffleRuby providers are additional independent candidates:
I085's GraalPy focus does not imply that those languages are implemented.

## Coordination status at publication

~~~text
FORMAL_IDENTIFIER=I085
FAMILY=family:I
ISSUE_STATUS=status:ready
ISSUE_ASSIGNEE=UNASSIGNED_READY
EFFECTIVE_PRIORITY=INTENTIONALLY_UNSET
TEXTUAL_PARENT=#818
NATIVE_PARENT=PENDING_RECONCILIATION
NATIVE_PARENT_VERIFIED=NO
FORMAL_IDENTIFIER_SEARCH_UNIQUE=YES
PROJECT_ROUTING=NOT_INDEPENDENTLY_VERIFIED
~~~

The available GitHub connector can publish Issues and comments but does not
expose native sub-issue mutation. The initial GET of `/issues/839/parent`
returned `404 No parent issue found` and #839 was not yet present in
the native `#818/sub_issues` list. The `Parent: #818` prose is bootstrap
input for the repository's `scripts/issue_intake.py` reconciliation, not
proof that the hierarchy is established.

I085 is therefore *allocated but not fully structurally reconciled* under
GITHUB015 until the native parent is verified.

## No execution or publication claim

No product code, tests, builds or runtime NumPy workload were executed for
this allocation. This record is a task handoff and does not assert that
GraalPy has already been integrated or that NumPy import succeeds.

## AI assistance disclosure

Prepared with AI assistance using live AUD019, I082, LIB021 and I085 Issue
evidence, current Protos sources/specification, and current GraalPy
embedding/native-extension documentation. No independent human review or
runtime execution is claimed.

## Subsequent live hierarchy reconciliation

After the publication of the allocation snapshot, a fresh read of
`GET /repos/guillermomolina/protos/issues/839/parent` returned native parent
`#818`. The repository intake automation therefore converged the native
Parent/Sub-issue relationship without treating textual prose as sufficient.

~~~text
NATIVE_PARENT_AFTER_INTAKE=#818
NATIVE_PARENT_VERIFICATION=PASS
ORIGINAL_ALLOCATION_SNAPSHOT=PRESERVED_ABOVE
~~~

The earlier pending state is retained above as a timestamped coordination
observation, not represented as the current hierarchy status.

## Scope correction after genericity review

The initial Python-only I085 scope was corrected on the live Issue while
preserving the historically accurate intake above. AUD019's actual architectural
promise is **not** one independent interop framework per language:

~~~text
COMMON_FOREIGN_VALUE_AND_MODULE_SUBSTRATE=I082_ALREADY_PUBLISHED
COMMON_GUEST_TRUFFLE_PROVIDER_MECHANICS=I085-A_SHARED_WORK
FIRST_TWO_GUEST_LANGUAGE_PROOFS=GRAALPY_AND_GRAALJS
LANGUAGE_SPECIFIC_MODULE_ACQUISITION=THIN_ADAPTER_PER_MODULE_SYSTEM
UNIVERSAL_TRUFFLE_PACKAGE_IMPORT_API=NO
NUMPY_REAL_CONFORMANCE=I085-B_REQUIRED
~~~

The shared mechanics are guest-language Context/session/resource wiring under
PLAT053 and D188's existing generic interop operations, lifecycle, identity,
projections, errors, and authority checks. Do not duplicate them per language.
Python's importlib and GraalJS ESM resolution are different, so only module
acquisition/environment-specific details belong in language adapters.

I085-A must prove **both** a real GraalPy module and a real GraalJS ES module
over the same generic shared guest-language mechanics. NumPy remains required
for final I085 acceptance. Unsupported GraalJS Node builtins/CommonJS and
unsafe Python native-extension privileges are not silently enabled. Additional
guest ecosystems remain future adapter integrations rather than replicas of
the runtime substrate.

This correction supersedes the original Python-only initial slice and the
original deferral of GraalJS testing, but retains its NumPy acceptance and
PLAT052/PLAT053 safety/lifecycle invariants.

Live updated issue:
https://github.com/guillermomolina/protos/issues/839

## Owner clarification — NumPy is optional, library choice is open

The owner clarified on 2026-10-08 that NumPy is an illustrative example,
**not a mandatory closure requirement**. The earlier NumPy-specific acceptance
claims in this retained allocation history are superseded.

The I085 objective is now the generic guest-language Truffle integration,
demonstrated with real modules in **both GraalPy and GraalJS**. A representative
real third-party library, if selected for extended proof, may be any suitable
pre-provisioned Python or JavaScript library and need not be NumPy. This
clarification does not weaken D188/D189/PLAT052/PLAT053 invariants, per-language
import adapters, security, lifecycle or zero-use requirements.

~~~text
CURRENT_SCOPE_AUTHORITY=I085/#839_OWNER_CLARIFICATION
FIRST_SLICE=I085-A
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
REQUIRED_REAL_LANGUAGES=GRAALPY_AND_GRAALJS
NUMPY_REQUIRED_FOR_I085_CLOSURE=NO
THIRD_PARTY_LIBRARY_NAME=UNCONSTRAINED
IMPORT_INSTALLS_DEPENDENCIES=NO
~~~

## Owner correction — external plugin SPI, no in-product guest languages

The owner explicitly rejected the earlier proposals to introduce direct
GraalPy/GraalJS integrations, language-specific Maven dependencies, or special
cases inside Protos. A hypothetical `graalmeinventoellenguaje` must be
integrable through an external provider artifact without changing or rebuilding
Protos.

This latest owner direction **supersedes all previous I085-A/B
Python/JavaScript implementation and NumPy closure requirements recorded
above**. The historical drafts remain visible for provenance only.

The newly prescribed I085-A is **IMPLEMENTATION**, not investigation.
The work is the minimal external provider SPI and controlled opt-in loading
from a host-supplied classloader/JAR path via Java `ServiceLoader`, bridging
to the I082 provider-neutral substrate. The default RuntimeHost retains an
empty provider registry and no plugin discovery; it has no guest-language
dependency, no foreign runtime initialization, and no need to edit `pom.xml`
for additional language ecosystems. Active registries remain immutable under
PLAT053; authority remains host-selected and fail-closed under PLAT052.

The acceptance proof is an out-of-core fixture JAR with a novel invented
import scheme created under the repository's test scope, loaded using only
the public SPI and no guest-language Maven dependencies. Language-specific
providers are built/installed externally as separate artifacts later;
Protos does not special-case their language IDs, modules or packages.

The source PLAT053 decision explicitly deferred a public third-party provider
registration API; the owner is now *explicitly* requesting that additional
external-extension capability. Do not misrepresent PLAT053's historical
decision as already containing a ratified public SPI design. Preserve its
existing immutable registry, lazy compartments, Actor isolation, authority
and lifetime invariants while following the latest owner-mandated extension.

~~~text
CURRENT_OWNER_SCOPE=EXTERNAL_PROVIDER_SPI
NEXT_SLICE=I085-A
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AGENT_RESEARCH=FORBIDDEN
LOAD_MECHANISM=EXPLICIT_PROVIDER_PATH_PLUS_JAVA_SERVICELOADER
DEFAULT_RUNTIMEHOST_PROVIDERS=NONE
FOREIGN_LANGUAGE_DEPS_IN_PROTOS_POM=NONE
FUTURE_LANGUAGE_ADDITION_REQUIRES_PROTOS_EDIT=NO
GRAALPY_GRAALJS_IMPLEMENTATION_IN_PRODUCT=NO
NUMPY_REQUIRED=NO
EXTERNAL_FIXTURE_PROOF=REQUIRED
~~~

Canonical current coordination:
https://github.com/guillermomolina/protos/issues/839
