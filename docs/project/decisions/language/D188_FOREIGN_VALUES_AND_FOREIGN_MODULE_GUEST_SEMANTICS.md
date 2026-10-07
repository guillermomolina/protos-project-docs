# D188 — Foreign values and foreign-module guest semantics

Status: **RATIFIED — OWNER APPROVED; NORMATIVE SPECIFICATION RECONCILIATION PENDING**

Owning decision: `guillermomolina/protos#819` — `D188 — Foreign values and foreign-module guest semantics`

Parent audit: `guillermomolina/protos#818` — `AUD019 — Foreign-library interop and polyglot import architecture audit`

Nature: durable implementation-independent language decision record; **non-normative until reconciled into the applicable `spec/` authority**

Approval date: **2026-10-07**

Revalidated Protos revision:

```text
PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
PROTOS_VERSION=0.3.261-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
```

The owner-review packet was produced against
`915fa3739e3a0a6e7f0934e3975e79d3bb24eba8 / 0.3.260-SNAPSHOT`.
The only product commit between that packet baseline and the revision above is
`TEST009-AD`, which moves one existing unsupported-representation failure out of
Truffle partial evaluation and explicitly makes no specification or semantic
change.

```text
AUD019_INVARIANT_DELTA=NONE_RELEVANT
```

## Approval provenance

The completed D188 investigation recommended:

```text
RECOMMENDED_CANDIDATE=C_HYBRID_PROTOS_SEMANTIC_PROJECTION
```

The project owner explicitly approved the recommendation in the active
owner-review interaction on 2026-10-07:

> apruebo recomendación

That approval selects the exact bounded contract recorded below. It does not
authorize skipping the normative specification reconciliation required for an
observable language-semantic decision.

```text
DECISION_APPROVAL_PROVENANCE=PASS
RECOMMENDED_CANDIDATE=C
SELECTED_CANDIDATE=C
OWNER_APPROVAL_REQUIRED=NO
```

## Ratified semantic contract

### 1. Governing principle

Foreign values do not silently import Java, Python, JavaScript, Ruby, another
guest language, or raw Truffle `InteropLibrary` semantics into Protos.

Ordinary Protos protocols remain authoritative. A foreign capability may
participate in ordinary syntax only where Protos defines a faithful projection
onto an already-existing Protos semantic institution. Operations that are
ambiguous, side-effecting in a way that conflicts with ordinary Protos meaning,
provider-specific, metadata-oriented, or otherwise lack a faithful projection
remain explicit through `std:interop` or a provider-owned facade.

```text
ORDINARY_PROTOS_SEMANTICS_FIRST=YES
DIRECT_INTEROPLIBRARY_SEMANTICS_AS_PROTOS=NO
STD_INTEROP_ESCAPE_HATCH=YES
```

### 2. Foreign module projection and identity

A successfully resolved foreign import uses the existing canonical `ModuleKey`
and Actor-local module-cache lifecycle.

The Protos-visible module instance is an Actor-local Protos module facade over
the provider-acquired foreign target. That facade, not the underlying Python
module, Java class, JavaScript module object, Ruby module/class, host wrapper, or
provider cache entry, is the Protos module instance.

```text
FOREIGN_MODULE_PROJECTION=
  ACTOR_LOCAL_PROTOS_MODULE_FACADE_OVER_PROVIDER_ACQUIRED_FOREIGN_TARGET

FOREIGN_MODULE_IDENTITY=
  SAME_ACTOR_PLUS_SAME_CANONICAL_MODULEKEY_YIELDS_SAME_ACTIVE_MODULE_FACADE
  DIFFERENT_ACTORS_YIELD_DISTINCT_MODULE_FACADES
```

The existing module lifecycle remains authoritative:

```text
specifier
  -> canonical ModuleKey
  -> Actor-local module cache
  -> cache facade before provider initialization
  -> INITIALIZING / partial initialization / cycles
  -> READY on success
  -> eviction on failed initialization
  -> retry creates a new active module attempt
```

A provider or foreign-language module cache cannot replace this Protos
module-identity authority.

The facade itself does not authorize shared mutable foreign state across Actors.
Authority, provider/Context topology, and isolation enforcement remain later
PLAT052/PLAT053 work.

### 3. Primitive admission

Automatic conversion is restricted to a source semantic category that is known
and a conversion that is lossless.

```text
FOREIGN_BOOLEAN
  -> Protos Boolean

TRUE_FOREIGN_ABSENCE_NULL
  -> Protos null

VALID_UNICODE_SCALAR_SEQUENCE
  -> Protos String

SOURCE_CLASSIFIED_INTEGRAL_VALUE
  -> unbounded Protos Integer

SOURCE_CLASSIFIED_BINARY_FLOATING_VALUE_EXACTLY_REPRESENTABLE_AS_BINARY64
  -> Protos Float
```

The following are not automatically collapsed into Protos value families:

```text
ambiguous null-like values
JavaScript undefined
decimal/rational/custom numeric families
lossy numeric conversions
text that is not a valid Unicode scalar sequence
foreign arrays/lists/maps/dicts/hashes merely because they have collection shape
```

`InteropLibrary.fitsIn*`-style representability checks may help prove a lossless
representation but do not by themselves prove the source semantic numeric
family.

### 4. Raw foreign-reference identity

Values not admitted into an existing Protos scalar value family are
identity-bearing foreign references.

```text
PROTOS_===_REMAINS_AUTHORITATIVE=YES
JAVA_REFERENCE_IDENTITY_IS_AUTHORITY=NO
PHYSICAL_TRUFFLE_WRAPPER_IDENTITY_IS_AUTHORITY=NO
```

Where the provider/runtime exposes a stable foreign-object identity contract,
distinct physical wrappers denoting that same foreign object must preserve one
Protos foreign-reference identity. `InteropLibrary` identity may be used by the
current implementation to realize that rule, but it does not redefine `===`.

Where no stable foreign identity exists, separate independent admissions are
distinct Protos foreign-reference identities unless the program is merely
aliasing an already-admitted reference.

If an original Protos value crosses into foreign code and returns recognizably
as that same Protos value without semantic replacement, it recovers the original
Protos identity rather than becoming a new foreign-reference identity.

No global wrapper cache is required for correctness; canonicalization is an
implementation freedom only when all observable identity results remain the
same.

### 5. Facade identity

A foreign module facade has the Actor-local module identity defined above.

Other provider-created Protos facades are ordinary Protos identity-bearing
objects unless another ratified semantic family says otherwise.

A facade and its underlying foreign target are not automatically identical:

```text
moduleFacade === rawForeignModule
    -> false
```

There is no universal rule that facade identity equals foreign target identity.

### 6. Equality and hashing

For raw foreign references, ordinary default equality is Protos default Object
equality:

```text
a == b
    -> a === b
```

The following are not automatically imported as Protos `==`:

```text
Java equals()
Python __eq__
JavaScript == or ===
Ruby ==
provider-specific equality
foreign hash-table equality
InteropLibrary identity
```

A deliberate Protos-facing facade may define ordinary Protos `==` and `hash`
behavior like any other Protos object. Explicit source-language/provider
equality, if useful, belongs in provider-specific behavior or `std:interop`.

Raw foreign-reference `hash` follows Protos semantic identity hashing and must
remain coherent with the selected Protos `==`.

```text
HASH_RULE=
  RAW_FOREIGN_REFERENCE_HASH_USES_PROTOS_SEMANTIC_IDENTITY_HASH

MAP_RULE=
  EXISTING_==_PLUS_HASH
  RAW_FOREIGN_REFERENCES_ARE_IDENTITY_EQUALITY_KEYS_BY_DEFAULT

IDENTITYMAP_RULE=
  EXISTING_===_PLUS_IDENTITYHASHOF
```

Foreign `hashCode`, `__hash__`, hash-entry identity/equality, or equivalent
foreign rules do not become Protos normal-`Map` rules.

A foreign hash container does not become standard Protos `Map`.

### 7. Member read

Ordinary member read preserves Protos semantics first.

Conceptually:

```text
foreign.name

1. resolve Protos-facing semantic protocol/slot projection
2. only after a miss, permit a faithful foreign-member fallback
3. otherwise signal the ordinary Protos missing-member failure
```

Names that constitute Protos-facing behavior such as `call`, `at`, `atPut`,
`each`, `==`, and `hash` cannot be silently captured by an underlying foreign
member of the same spelling. The foreign member remains reachable explicitly
through `std:interop` where the public library later exposes such an operation.

A generic foreign-member read is eligible for ordinary fallback only when it is
faithful to an ordinary member read. A source/runtime member read known to carry
material read side effects, or otherwise not equivalent to a Protos read, must
use an explicit provider facade/protocol or `std:interop`.

A foreign method-like value exposed through ordinary read must preserve the
required foreign receiver binding. If that cannot be represented faithfully,
generic ordinary projection is not available and the provider or explicit
interop path owns the operation.

### 8. Member write

Ordinary Protos `=` is not redefined as foreign `writeMember`.

```text
MEMBER_WRITE_RULE=
  ORDINARY_=_REMAINS_LOCAL_PROTOS_SLOT_MODIFICATION
  RAW_FOREIGN_MEMBER_MUTATION_IS_NOT_IMPLICIT
```

A raw foreign reference therefore does not gain writable Protos local slots
merely because `InteropLibrary.writeMember` exists.

A true Protos facade may have ordinary local Protos slots and those slots follow
the normal write rules.

Foreign-member mutation requires an explicit `std:interop` operation or a
deliberately defined provider-facing protocol/facade.

### 9. Member invocation

Ordinary:

```protos
foreign.member(args...)
```

is defined in terms of the projected/read member followed by ordinary Protos
invocation of that resulting value.

A hidden `InteropLibrary.invokeMember` shortcut is not semantic authority.
An implementation may use it only when it is proven observationally equivalent
to the selected read/bind plus ordinary Protos invocation semantics, including
evaluation order, receiver binding, result admission, failure translation, and
side effects.

Otherwise member invocation remains provider-specific or explicit
`std:interop`.

### 10. Foreign callability

`InteropLibrary.isExecutable` does not create a second Protos callable
predicate.

Ordinary foreign callability is represented through the existing `call`
protocol:

```text
foreign(args...)
  -> ordinary lookup of foreign.call
  -> selected call value must be a Closure
  -> ordinary Closure activation
  -> the adapter Closure performs the foreign execute operation
```

A foreign executable may therefore receive a Protos-facing `call` Closure, but
the ordinary callable model remains unchanged.

A foreign non-executable does not acquire hidden callability.

### 11. Construction

`InteropLibrary.isInstantiable` does not create a new Protos construction syntax
and does not by itself make a raw foreign value callable.

Baseline explicit construction is through `std:interop.instantiate` or
equivalent later-approved explicit API.

A provider facade for a type/class/module may deliberately expose an ordinary
Protos `call` Closure whose documented Protos meaning is construction. That is a
provider projection through the existing callable model, not hidden
`instantiate` semantics.

A value that is both executable and instantiable must not be resolved by a
universal hidden heuristic; the provider facade or explicit interop surface
must disambiguate.

### 12. Indexed/array values

Bracket syntax remains the existing indexing protocol:

```text
x[i]
  -> x.at(i)

x[i] = value
  -> x.atPut(i, value)
```

A foreign value with array elements may expose faithful Protos-facing `at` and,
where semantically justified, `atPut` behavior.

That does not make the foreign value a standard Protos `Array` and does not
grant standard Array family membership, identity, copying, iteration-snapshot,
Actor/P-transfer, or other Array-family semantics.

When one foreign value exposes both array-element and hash-entry capabilities,
the generic layer must not guess which capability `at` denotes. A provider
facade or explicit `std:interop` operation must resolve that ambiguity.

### 13. Foreign hashes/maps

A provider may map foreign hash-entry access to a custom Protos-facing `at` /
`atPut` protocol when that is the selected unambiguous provider semantics.

The underlying foreign container retains its own key semantics for that
operation. Those semantics do not become the equality/hash rules of standard
Protos `Map`.

```text
FOREIGN_HAS_HASH_ENTRIES_IMPLIES_STANDARD_MAP=NO
```

### 14. Iteration

Ordinary foreign iteration, where projected, uses a Protos-facing `each`
protocol and ordinary Protos invocation of the supplied callback.

The baseline implementation must pull foreign iterator elements into Protos and
invoke the callback from Protos. It must not obtain general foreign-to-Protos
callback authority by handing the Protos callback into foreign code; that
semantic institution belongs to D189.

A foreign iterable does not automatically acquire standard Array or Map
snapshot-iteration semantics. It has the explicitly defined custom `each`
semantics of its projection.

### 15. Foreign failures

D188 selects one initial Protos-facing standard error category:

```text
ForeignError -> Error
```

Each actual foreign failure occurrence that has crossed into a foreign
operation produces a fresh Protos `ForeignError` occurrence.

The minimum safe public payload may expose ordinary Protos data such as:

```text
language
operation
category
foreignCategory
message
cause
```

where `cause`, if publicly projected, is another safe Protos `Error` or `null`.
Cause projection must be cycle-safe.

The baseline does not expose as public Error data:

```text
PolyglotException
Java Throwable
foreign exception handles
foreign runtime objects
authority-bearing host handles
foreign/native stack objects
mixed-language stack objects
suppressed exception objects
runtime implementation metadata
```

Such data may be retained as implementation-private diagnostic metadata only
where the applicable tooling/diagnostic authority permits it.

Failures that occur before a foreign operation is actually entered remain the
existing ordinary Protos failures. For example, a non-callable raw value failing
ordinary `call` lookup is not converted merely because the receiver originated
from a foreign runtime.

### 16. `std:interop`

`std:interop` and foreign import share the same foreign-value admission,
identity, conversion, and failure-projection substrate.

The explicit layer remains the escape hatch for operations that lack a natural
ordinary Protos mapping, including likely classes of operation such as:

```text
explicit member read/write/invoke
execute
instantiate
array/hash low-level access
iterator access
foreign identity query
language/metaobject/source/display metadata
explicit scalar conversion
```

The public Standard Library surface is not required to mirror
`InteropLibrary` one-for-one, and authority-bearing acquisition/runtime
primitives remain subject to later PLAT052/PLAT053 decisions.

No universal source-language semantic equality operation is invented where the
foreign substrate itself has no such common operation.

The same underlying foreign identity observed through ordinary syntax and
through `std:interop` must enter Protos through the same admission/identity
rules.

## Candidate resolution

### Candidate A — direct Interop mapping

**Rejected.**

Directly mapping ordinary syntax to whichever `InteropLibrary` message exists
would risk or violate Protos module identity, `===`, `==`/`hash`, `call`,
`at`/`atPut`, slot writes, collection families, and Error semantics.

### Candidate B — explicit `std:interop` only

**Not selected as the target model.**

It is semantically clean and remains an important fallback, but it fails the
AUD019 requirement that foreign libraries should be consumable through ordinary
Protos syntax where an exact semantic projection exists.

### Candidate C — hybrid Protos semantic projection

**Selected.**

Preserve raw foreign values when a faithful projection exists, introduce
Protos-facing facades only where required by module identity or deliberate
semantic adaptation, and retain `std:interop` as the explicit lower-level escape
hatch.

### Additional candidate — universal facade

**Rejected.**

Wrapping every foreign value in a canonical Protos facade would add universal
wrapper allocation, canonicalization/cache lifetime, identity forwarding,
round-trip unwrap/rewrap, and Context-lifetime complexity without demonstrated
need. It conflicts with the pay-for-what-you-need rule and the PLAT013 precedent
that real semantic values should remain the receivers when a wrapper is not
semantically necessary.

## Incremental-design boundary

The smallest sufficient selected institution is:

```text
Actor-local foreign module facade
raw foreign-reference admission/identity
lossless scalar admission
faithful existing-Protos-protocol projection
ForeignError projection
explicit std:interop escape hatch
```

The decision deliberately does not pre-build:

```text
general foreign-to-Protos callback semantics
retained/asynchronous callbacks
Promise/CompletionStage/awaitable -> Future
universal foreign cancellation
general Actor/P/Process foreign transfer
general proxy/copy/rematerialization
third-party provider API
foreign dependency acquisition
provider/Context/Engine topology
HostAccess/sandbox authority
full multi-language Native packaging
```

A program that never acquires a foreign value should not pay semantic or runtime
cost for universal wrapper graphs, provider canonicalization, or foreign-error
state.

## Deferred owners

```text
D189=
  foreign -> Protos callback Actor/Task/lifetime semantics

PLAT052=
  HostAccess / sandbox / foreign authority enforcement

PLAT053=
  ForeignImportProvider / Context / Engine / lifecycle / distribution topology

DEFERRED=
  Promise/CompletionStage/awaitable -> Future
  universal foreign cancellation
  general foreign Actor/P/Process foreign transfer
  third-party provider API
  Maven/PyPI/npm/RubyGems dependency acquisition
  full multi-language Native packaging
```

## Proof obligations for later implementation

A later implementation must prove at minimum:

```text
same canonical ModuleKey imported twice
two spellings canonicalizing to one ModuleKey
same ModuleKey in two Actors
foreign module cycle / partial initialization
foreign initialization failure / eviction / retry

same foreign object acquired twice
same object through two physical wrappers/paths
Protos -> foreign -> Protos round-trip
distinct equal-content foreign objects

=== / == / hash / Map / IdentityMap
member read / collision / extraction / write boundary / invocation
foreign executable / non-executable
foreign instantiable / non-instantiable
array / hash / dual array+hash / iteration
large exact Integer / binary64 / Unicode / true null / JS undefined
lossy or ambiguous conversion rejection

foreign exception / missing member / arity / type failure
no raw host exception leakage

ordinary syntax vs std:interop identity/conversion/error parity
Actor and P non-transferability
```

D189 owns the callback-specific proof matrix.

## Strongest argument against the selected candidate

Candidate C permanently introduces a foreign-receiver branch into some ordinary
Protos-facing protocol projection and therefore requires explicit rules for
collisions, side-effecting member access, dynamically available capabilities,
and failure translation.

Candidate B would keep the ordinary object model mechanically cleaner.

The selected answer accepts that bounded complexity because Candidate B would
discard the ergonomic foreign-library objective of AUD019, while Candidate C
contains the complexity behind the rule:

```text
Protos semantics first
faithful foreign projection second
explicit interop when not faithful
```

## Regret scenario and escape path

A future provider may have highly dynamic or context-dependent identity, member,
call, indexing, or side-effect behavior that cannot be projected faithfully.

The escape path is additive and already preserved:

```text
retain the raw foreign reference
disable/narrow automatic projection for that provider
use an explicit provider facade
use std:interop for lower-level operations
```

No change to `ModuleKey`, `===`, `call`, `at`/`atPut`, or the Protos Error model
is required to take that path.

## Ratification / normative publication boundary

D188 changes observable Protos semantics and therefore requires normative
reconciliation under `guillermomolina/protos:spec/` before the decision can be
reported fully closed and before dependent implementation treats the new
foreign-value behavior as normative language authority.

This project-record publication itself does not modify specification authority.

```text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
AUD019_INVARIANT_DELTA=NONE_RELEVANT

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=YES
SPECIFICATION_CHANGE_REQUIRED=YES
SPECIFICATION_RECONCILIATION=PENDING

IMPLEMENTATION_AUTHORIZED=NO
D188_CLOSURE=PENDING_SPECIFICATION_RECONCILIATION
D189_RELEASE=PENDING_D188_NORMATIVE_RECONCILIATION
```

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed D188 GITHUB010 investigation, current repository and
upstream evidence, and the project owner's explicit approval. No independent
human review is claimed by this record.
