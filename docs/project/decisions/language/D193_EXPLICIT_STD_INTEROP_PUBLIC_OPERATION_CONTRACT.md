# D193 — explicit std:interop public operation contract

Status: **RATIFIED — Candidate A− / minimal operation-only escape hatch**

Formal decision: `guillermomolina/protos#836`

Parent library work: `LIB021 / guillermomolina/protos#835`

Normative reconciliation owner: `I084 / guillermomolina/protos#837`

Decision date: **2026-10-07**

## Decision

D193 defines the first public explicit foreign-interoperability Standard Library
surface over the D188/D189/PLAT052/PLAT053 substrate.

The selected public module and v1 operations are:

~~~text
SELECTED_CANDIDATE=A_MINUS_MINIMAL_OPERATION_ONLY_ESCAPE_HATCH

STD_INTEROP_MODULE=std:interop
PUBLIC_OPERATIONS=invoke, instantiate, readMember, writeMember
~~~

The module is an explicit operation layer over already-held foreign values and
facades. It is not a provider registry, discovery service, dependency loader,
authority broker, reflection universe, or one-to-one mirror of a host runtime
API.

## Public operation contracts

### invoke

~~~text
invoke(target, ...arguments)
~~~

means explicit foreign execution of the underlying already-held foreign target.

It does not perform ordinary Protos `call` lookup, does not redefine ordinary
callability, and does not make a raw foreign value ordinarily callable.

Arguments cross the existing D188 outbound boundary, with Closure arguments
using D189 exactly where the provider accepts them. Normal results use D188
admission.

### instantiate

~~~text
instantiate(target, ...arguments)
~~~

means explicit foreign instantiation/construction of the underlying already-held
foreign target.

It is semantically distinct from `invoke` even when one target supports both
execution and instantiation. Ordinary Protos `call` remains unchanged and the
runtime does not guess which foreign operation the program intended.

### readMember

~~~text
readMember(target, name)
~~~

requests the exact named member of the underlying foreign target through the
foreign operation boundary.

This explicit operation bypasses ordinary Protos slot/projection precedence only
for the requested foreign member operation. It does not change ordinary
`target.name`.

In particular, the D188-protected Protos institution names remain protected
under ordinary lookup:

~~~text
call
at
atPut
each
==
hash
~~~

A same-spelling foreign member is reachable through explicit `readMember`
without changing the meaning of ordinary Protos lookup.

The member name must be a Protos String accepted before provider entry. A normal
foreign member result uses D188 admission.

### writeMember

~~~text
writeMember(target, name, value)
~~~

is the explicit generic foreign-member mutation operation.

It never reinterprets ordinary Protos:

~~~text
target.name = value
~~~

as foreign mutation.

The outbound value uses the existing D188/D189 boundary. When the foreign write
completes normally, the exact original Protos `value` argument is the normal
result:

~~~text
WRITE_MEMBER_NORMAL_RESULT=EXACT_ORIGINAL_PROTOS_VALUE_ARGUMENT
NO_READBACK=YES
NO_RESULT_READMISSION=YES
~~~

This mirrors the established Protos write-result convention without importing a
provider-specific completion token.

## Error boundary

D193 keeps the existing D188 entered/not-entered boundary:

~~~text
non-foreign or invalid target before provider entry
    -> ordinary Protos Error

invalid arity or invalid member-name type before provider entry
    -> ordinary Protos Error

unsupported explicit operation known before provider entry
    -> ordinary Protos Error

outbound value not exportable before provider entry
    -> ordinary Protos Error

closed session/generation before provider entry
    -> ordinary Protos Error

provider capability unavailable before provider entry
    -> ordinary Protos Error

foreign operation actually entered and then fails
    -> fresh ForeignError

exact D189 Error/control outcome traverses the same foreign operation unchanged
    -> propagate the exact original Protos outcome
~~~

No Java/Python/JavaScript/provider-specific public Error hierarchy is added.

## Callback contract

~~~text
CALLBACK_POLICY=REUSE_D189_EXACTLY
~~~

The four operations do not create a second callback model.

Where a Closure argument is accepted it remains the exact operation-scoped D189
callback capability:

~~~text
same originating Actor
same current Task / structured scope
no new Task
no new Future
no new Actor turn
nested/sequential/recursive synchronous re-entry allowed
concurrent re-entry rejected
foreign-thread ingress rejected before guest entry
actual suspension across the live foreign extent rejected before commit
callback lifetime = originating foreign operation dynamic extent
late/post-close callbacks rejected
no Process/Actor/Task/provider/Context resurrection
retained/asynchronous callbacks deferred
~~~

## Authority contract

`std:interop` is possession-based only.

Importing the module creates:

~~~text
zero provider compartments
zero provider sessions
zero Polyglot Contexts
zero provider/language discovery
zero dependency acquisition
zero authority
~~~

The public API does not select or expose:

~~~text
provider
execution profile
trusted mode
host access
filesystem/network/process/native authority
classloading
reflection
cross-language evaluation
provider registration
provider hot-loading
dependency acquisition
~~~

Every operation uses only the provider/session/generation and authority already
attached to the foreign value or facade the program possesses.

## Actor, P, and lifetime

The selected surface changes none of the existing isolation/lifetime rules:

~~~text
automatic Actor transfer = NO
automatic P transfer = NO
automatic proxy = NO
rematerialization = NO
provider reopening = NO
old-reference generation rebinding = NO
~~~

The exact existing provider/session/generation handle remains the lifetime
authority. A closed generation never rebinds to a later generation.

## Deliberately deferred public surface

D193 does not publish in v1:

~~~text
isForeign
isExecutable
isInstantiable
supports(...)
hasMembers
hasArrayElements
hasHashEntries
hasIterator

invokeMember

readElement
writeElement

hash-entry operations
iterator/cursor objects
explicit iterator protocol

explicit scalar/raw conversion

metaobject
type
language
source/display metadata

provider discovery
language discovery
foreign class/type/module acquisition
dependency acquisition
provider registration
provider hot-loading
session/generation/lifetime inspection
retained/asynchronous callbacks
~~~

These families remain additive future work only when a concrete Protos
provider/workload establishes need.

## invokeMember delta

The D193 owner-review refinement discovered one material correction to the
earlier LIB021-0 rationale:

~~~text
INVOKE_MEMBER_COMPOSITION_IS_NOT_UNIVERSAL=YES
~~~

A backend may support a member that is invocable but not readable. Therefore
`invokeMember(target, name, args...)` is not universally equivalent to:

~~~text
invoke(readMember(target, name), args...)
~~~

D193 nevertheless defers `invokeMember` because no current Protos provider or
workload requires the invocable-but-unreadable case.

The deferral rationale is therefore:

~~~text
INVOKE_MEMBER=DEFER
REASON=NO_CURRENT_REQUIRED_WORKLOAD
FUTURE_ADDITION=ADDITIVE_DISTINCT_OPERATION
~~~

It is not permissible to later describe this deferral as proof that
`invokeMember` is universally redundant.

## Indexed/hash ambiguity

D188 continues to project ordinary `at` / `atPut` only when that mapping is
faithful and unambiguous.

D193 adds no explicit indexed or hash operations in v1:

~~~text
INDEXED_ACCESS=DEFER
HASH_ACCESS=DEFER
~~~

A future provider/workload that genuinely needs to distinguish indexed from hash
semantics may justify an additive explicit family without reinterpreting the
four selected operations.

## Conversion, introspection, and discovery

~~~text
EXPLICIT_CONVERSION=NO_V1
PUBLIC_CAPABILITY_PREDICATES=NONE_V1
METAOBJECT_OR_TYPE_SURFACE=NONE_V1
PROVIDER_DISCOVERY_SURFACE=NONE
FOREIGN_ACQUISITION_SURFACE=NONE
DEPENDENCY_ACQUISITION=NO
~~~

D188's source-classified lossless scalar admission remains the automatic
conversion contract. D193 does not create a parallel conversion or capability
ontology.

## Portability

The public semantics are provider-neutral and do not require Truffle.

A restricted host-Java provider, a Graal guest-language provider, a
Wasm/component-like provider, an isolated external service/runtime, or a custom
host API provider may implement only the selected operations meaningful to its
contract. Unsupported operations fail through the pre-entry ordinary Protos
boundary.

Backend message names and capability vocabularies remain implementation-private.

## GITHUB021 invariant / delta result

The selected candidate preserves the existing approved foundations:

~~~text
D188_DELTA=NONE
D189_DELTA=NONE
PLAT052_DELTA=NONE
PLAT053_DELTA=NONE
D192_DELTA=NONE

NEW_DELTA=
  invokeMember composition is not universal;
  this changes the deferral rationale only;
  the v1 operation set remains invoke, instantiate, readMember, writeMember

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

No selected D193 rule transfers foreign values, changes ordinary lookup/call/
assignment/indexing/iteration, widens callbacks, enlarges authority, alters
provider topology, or changes projected `each`.

## Strongest counterargument and regret path

The strongest argument against A− is that the only production provider at
selection time is restricted host Java, whose admitted constructor and method
workloads already work through `import()` plus ordinary Protos projection.
Several selected escape-hatch operations therefore precede a production
workload that cannot already be expressed.

The regret scenario is a future provider dominated by invocable-only members or
ambiguous indexed/hash operations, making those later families more important
than the original four.

The escape path is additive:

~~~text
add only the newly proven explicit operation family
reuse D188/D189/PLAT052/PLAT053
do not reinterpret the original four
do not reinterpret ordinary Protos semantics
~~~

## Normative reconciliation

This durable decision record is non-normative.

Because D193 defines observable public Standard Library behavior, its selected
contract is routed to:

~~~text
I084 / guillermomolina/protos#837
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_SCOPE=NORMATIVE_SPECIFICATION_RECONCILIATION_ONLY
~~~

Expected primary owner at allocation time:

~~~text
spec/semantics/VALUES_AND_COLLECTIONS.md
  Foreign Values
    Relation to explicit interoperability
~~~

The current normative text intentionally describes `std:interop` as a future
facility. I084 must replace that placeholder with the exact selected D193
contract and advance the global specification revision before LIB021 runtime/
library implementation proceeds.

## AI-assistance disclosure

This durable decision record was materially prepared with AI assistance from
ChatGPT from the completed LIB021-0/D193 investigation, current repository
evidence, and the project owner's explicit approval. No independent human review
is claimed.
