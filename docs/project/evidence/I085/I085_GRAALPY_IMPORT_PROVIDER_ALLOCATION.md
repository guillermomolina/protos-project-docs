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
