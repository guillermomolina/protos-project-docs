# LIB021-0 — Explicit foreign interoperability Standard Library surface research

## Record role

This is durable **research evidence**, not a normative specification and not an owner-ratified design decision.

It records the completed LIB021-0 investigation that routed the substantive public-`std:interop` semantic choice to D193 / `guillermomolina/protos#836`.

```text
FORMAL_OWNER=LIB021/#835
SLICE=LIB021-0
SLICE_TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO

PROTOS_REVISION=c0ac98971df115d64b7bc9f146e8b02e11da30e6
IMPLEMENTATION_VERSION=0.3.280-SNAPSHOT
SPECIFICATION_REVISION=0.1.447
HEAD_SUBJECT=I082-G: add restricted host Java provider baseline

DECISION_GATE=D193/#836
RECOMMENDATION_APPROVAL_STATUS=PENDING_OWNER_APPROVAL
```

## Audited authority

The research inspected the exact product revision above and consumed, without reopening:

- D188 / #819 — foreign-value and foreign-module semantics, normative from specification 0.1.445;
- D189 / #820 — synchronous foreign callback semantics, normative from specification 0.1.446;
- D192 / #833 — exact-receiver projected foreign `each(block)` result, normative from specification 0.1.447;
- PLAT052 / #821 — capability-preserving deny-by-default provider authority gate;
- PLAT053 / #822 — Process-owned lazy provider compartments with Actor-isolated provider sessions;
- I082 / #830 — completed provider-neutral foreign runtime/import foundation plus restricted host-Java provider.

Normative files inspected included:

```text
spec/semantics/VALUES_AND_COLLECTIONS.md
spec/semantics/CALLABLES.md
spec/semantics/MODULES.md
spec/semantics/ERRORS.md
spec/concurrency/FUTURES_AND_TASKS.md
spec/concurrency/ACTORS.md
spec/concurrency/PARALLEL_EXECUTION.md
spec/PROTOS_SPEC_CHANGELOG.md
```

Implementation owners inspected included the shared admission, raw-reference, handle, adapter, projected-operation, operation-boundary, callback, provider-registry/admission/lifecycle, foreign-module and restricted host-Java implementation and their focused tests.

## Current substrate conclusions

The I082 substrate is provider-neutral rather than a Java-specific FFI.

A raw foreign reference is bound to the exact provider session generation that admitted it. The current private admission descriptor can classify capabilities including:

```text
EXECUTABLE
INSTANTIABLE
INDEXED_READ
INDEXED_WRITE
HASH_ENTRIES
ITERABLE
```

Those capability categories are implementation-private and are not a public Protos protocol.

Current ordinary projection deliberately refuses to guess in ambiguous cases:

```text
projected call
    = executable AND NOT instantiable

projected indexed access
    = indexed capability AND NOT hash-entry ambiguity
```

The existing D188/D189 substrate already supplies one authoritative boundary for:

- lossless scalar admission;
- raw foreign identity;
- safe same-provider/session outbound raw references;
- operation-scoped Closure callbacks;
- entered/not-entered failure classification;
- exact synchronous callback Error/control round-trip;
- Actor/P non-transferability;
- exact session-generation lifetime.

A future `std:interop` must reuse those owners rather than directly exposing or mirroring a backend interop API.

## What ordinary Protos already covers

The investigation found that ordinary syntax plus provider imports already covers:

```text
lossless foreign scalar admission
raw foreign === identity and default == / identity hash
faithful foreign-member fallback after ordinary Protos lookup
foreign.member(args...) as read + ordinary invocation
unambiguous executable foreign(...) through projected call
unambiguous indexed at / atPut
declared faithful pull foreign.each(block)
Actor-local foreign module facade identity/cache/lifecycle
provider-specific deliberate facades
restricted java: acquisition through import(...)
```

Therefore the value of an explicit module is not to repeat every backend capability.

## Smallest unresolved public gaps

Four operation classes survived the proportionality test:

1. explicit foreign execution when ordinary callability is intentionally unavailable or ambiguous;
2. explicit foreign instantiation, especially when a target is both executable and instantiable;
3. explicit foreign member read when ordinary Protos projection/slots intentionally take precedence;
4. explicit foreign member write, which D188 deliberately does not map from ordinary assignment.

## Candidate set

### A− — minimal operation-only escape hatch — recommended

```text
std:interop

invoke(target, ...arguments)
instantiate(target, ...arguments)
readMember(target, name)
writeMember(target, name, value)
```

No public capability predicates are required for v1.

### C — small public capability core

Would add public predicates such as `isForeign`, `isExecutable`, `isInstantiable` or `supports(...)` alongside operations.

Survives technically but is not recommended because no current operation requires publishing the private capability vocabulary.

### E — no std:interop yet

Preserves maximum simplicity but leaves the four deliberate D188 escape-hatch gaps inaccessible.

### Rejected families

- B — complete InteropLibrary-style public façade: over-broad and backend-leaking;
- D — provider-directed generic public interop: fragments the shared substrate;
- F — stringly `perform(target, operationName, ...)`: hides rather than removes a second heterogeneous protocol.

## Comparative score

| Criterion | A− operation core | C capability core | E defer |
| --- | ---: | ---: | ---: |
| Correctness / invariant preservation | 5 | 5 | 5 |
| Protos alignment | 5 | 4 | 4 |
| Present-need proportionality | 5 | 3 | 2 |
| Incremental growth | 5 | 5 | 4 |
| Future-option resilience | 5 | 4 | 4 |
| Scalability | 4 | 5 | 3 |
| Conceptual simplicity | 5 | 4 | 5 |
| Portability / implementation freedom | 5 | 4 | 5 |
| Runtime / resource cost | 5 | 4 | 5 |
| Failure / operability | 5 | 4 | 3 |
| Deferral / reversibility / migration | 5 | 3 | 3 |
| Evidence maturity / implementation risk | 4 | 3 | 5 |
| **Total / 60** | **58** | **48** | **48** |

Arithmetic is comparison evidence only; A− is recommended because it closes the exact first-order semantic gaps without publishing a second reflective/capability universe.

## Recommended A− contract for owner review

```text
STD_INTEROP_RECOMMENDED=YES
STD_INTEROP_MODULE=std:interop

PUBLIC_OPERATIONS=invoke, instantiate, readMember, writeMember

ORDINARY_SYNTAX_OVERLAP_POLICY=
  explicit interop only for operations/disambiguation not faithfully expressed
  by ordinary Protos; ordinary D188 projection remains unchanged

EXPLICIT_INVOKE=YES
EXPLICIT_INSTANTIATE=YES
EXPLICIT_MEMBER_READ=YES
EXPLICIT_MEMBER_WRITE=YES
EXPLICIT_MEMBER_INVOKE=NO_COMPOSE_READMEMBER_PLUS_INVOKE

EXPLICIT_IS_FOREIGN=DEFER
EXPLICIT_INDEX_ACCESS=DEFER
EXPLICIT_HASH_ACCESS=DEFER
EXPLICIT_ITERATION=DEFER
EXPLICIT_CONVERSION=NO_V1
METAOBJECT_OR_TYPE_SURFACE=NO_V1
PROVIDER_DISCOVERY_SURFACE=NO
FOREIGN_ACQUISITION_SURFACE=NO

CALLBACK_POLICY=REUSE_D189_EXACTLY
ERROR_POLICY=
  pre-entry invalid/unsupported failures remain ordinary Protos Errors;
  entered foreign failures remain fresh ForeignError;
  exact D189 round-tripped Protos Error/control remains exact

AUTHORITY_POLICY=
  PLAT052 unchanged; possession of an already-held foreign value never grants
  provider discovery, acquisition, registration, trusted mode or additional authority

ACTOR_P_POLICY=NO_AUTOMATIC_TRANSFER_PROXYING_OR_REMATERIALIZATION
SESSION_LIFETIME_POLICY=
  exact existing provider session generation; closed generation never rebinds

DEPENDENCY_ACQUISITION=NO
NEW_PLATxxx_REQUIRED=NO
NEW_Dxxx_REQUIRED=YES
```

### Proposed write result

The research recommends:

```text
writeMember(target, name, value) normal success
    -> exact original Protos value argument
```

This is proposed observable public behavior and is **not** existing D188 authority.

## Error and callback boundary

No new public Error family is recommended.

Before provider entry, invalid/non-foreign targets, unsupported operation shapes, outbound-admission failures and closed-generation failures remain ordinary Protos failures.

After provider entry, foreign-produced failures remain fresh sanitized `ForeignError` values under D188.

Closure arguments reuse D189 exactly:

```text
same Actor
same current Task / structured scope
no new Actor turn or Future
operation dynamic-extent lifetime
no foreign-thread ingress
no actual Task suspension across the live foreign extent
no resurrection
```

No retained/asynchronous callback surface is proposed.

## Authority and lifetime boundary

The proposed module is possession-based only.

It grants none of:

```text
provider discovery or registration
trusted mode
host-class lookup/loading
filesystem/network/process/native authority
environment/reflection access
cross-language eval
dependency acquisition
```

Importing `std:interop` itself should create zero provider compartments, zero provider sessions, zero Polyglot Contexts, zero discovery and zero authority.

Explicit operations continue through the exact provider/session/generation of the already-held foreign value.

Actor and P transfer rules remain unchanged.

## Deliberately deferred surface

```text
isForeign
public capability predicates
invokeMember
explicit indexed operations
explicit hash operations
explicit iterator/cursor surface
explicit scalar conversion
metaobject/type/language/display metadata
provider discovery
foreign acquisition
session/generation/lifetime inspection
retained/asynchronous callbacks
```

All are intended to remain additive future work if concrete workloads justify them.

## Prior art used

The investigation compared materially different approaches:

- GraalVM Truffle `InteropLibrary`: separate execute/instantiate and member operations demonstrate real ambiguity, but its broad message universe is too large to copy as Protos public semantics.
- `TruffleLanguage.Env`: discovery/evaluation is authority-sensitive even within the backend; no reason to expose the RuntimeHost provider registry.
- Clojure Java interop: explicit construction/member mutation can coexist with ordinary language semantics, but it is Java-specific rather than provider-neutral.
- Racket FFI: explicit low-level FFI and acquisition show the authority cost of combining operations with loading.
- Python `ctypes`: shows the safety and foreign-thread callback hazards Protos deliberately avoids.
- LuaJIT FFI: demonstrates the power and ambient-authority risk of global native symbol access.
- WebAssembly Component Model: narrow explicit contracts and capability-like imports support keeping authority/acquisition separate from value operations.

Prior art is evidence only and does not override Protos authority.

## Strongest argument against A−

The only production provider at the audited revision is restricted host Java, and its admitted constructor/static/instance use cases already work through `import()` plus ordinary Protos syntax.

Therefore publishing `std:interop` before a second provider exists risks solving architecturally known but not yet workload-driven gaps.

A− is still recommended because D188 deliberately leaves these exact operation classes outside ordinary projection, and the proposed four verbs do not commit Protos to Truffle's wider reflection/container/metaobject universe.

## Regret scenario and escape path

A future Python/JS/Wasm-like provider may need indexed/hash/interface operations more than member operations.

That does not require rewriting A−. The escape path is additive:

```text
add only the proven operation family
reuse the same D188/D189/PLAT052/PLAT053 substrate
do not reinterpret the original four
```

The larger lock-in risk would be publishing a comprehensive backend-shaped capability/metaobject protocol now.

## Routing

LIB021-0 found that this surface changes/newly defines observable public Standard Library behavior.

Therefore implementation remains blocked on D193.

```text
D193_ISSUE=https://github.com/guillermomolina/protos/issues/836
D193_STATUS=PROPOSED
RECOMMENDED_CANDIDATE=A_MINUS_MINIMAL_OPERATION_ONLY_ESCAPE_HATCH
OWNER_APPROVAL_REQUIRED=YES
IMPLEMENTATION_AUTHORIZED=NO
NEW_PLATxxx_REQUIRED=NO
```

At publication time there is no owner approval of A−. Permission to update Issues or publish this research evidence is not design approval.
