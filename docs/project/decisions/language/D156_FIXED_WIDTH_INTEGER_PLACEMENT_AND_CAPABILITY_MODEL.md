# D156 — Fixed-width integer placement and capability model

Status: **RATIFIED**

Nature: durable non-normative language decision record

Tracking issue: `guillermomolina/protos#616`

Trigger: `AUD009-B7 / guillermomolina/protos#614`

Implementation route: `I052 / guillermomolina/protos#628`

Protos evidence/closure revision:
`a5f4444f25d1f2722669f4cfdf5d7f62b1af6d88`

Project-owner approval provenance:
`guillermomolina/protos#616`, issue comment `5739410881`, 2026-09-19.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

## Decision

D156 selects **Candidate D′ — remove fixed-width numeric families from Core
while preserving and rehoming reusable fixed-width machinery toward the
width-bearing domain that actually needs it**.

The retained Core numeric model is:

```text
Number -> Object
Integer -> Number
Float   -> Number

Integer = exact unbounded mathematical integer
Float   = IEEE 754-2019 binary64
```

The following cease to be permanent Core semantic numeric families:

```text
UInt8  Int8
UInt16 Int16
UInt32 Int32
UInt64 Int64
```

The dedicated Core conversion/arithmetic institution that exists specifically
for those semantic families is also removed from Core.

This decision does **not** reject fixed-width capability in Protos.

Width and signedness are instead owned by a domain that demonstrates a real
representation-width requirement, especially future FFI/ABI, host interop,
binary-structure/encoding, memory-layout, bit-vector/cryptographic, or another
explicitly width-bearing facility.

No public FFI API is selected by D156.

## Preservation invariant

Removal from Core is not permission to throw away useful implementation work.

Implementation work following D156 must audit the existing fixed-width machinery
and preserve/rehome reusable parts when they remain useful for a future explicit
interop or binary boundary.

Preservation candidates include:

- width/signedness/range metadata;
- exact range validation;
- exact integral host conversion support;
- Truffle `InteropLibrary` fits/conversion behavior;
- boundary-focused tests that prove reusable conversion behavior.

A neutral internal helper/representation may retain such machinery when doing so
does not recreate guest-visible fixed-width numeric semantics.

Historical implementation effort remains sunk cost for deciding semantics, but
working code may still be valuable implementation material in its proper layer.

## Explicit non-inheritance boundary

The following properties of the old Core institution are **not automatically
part of a future FFI/interop design**:

- same-family checked `+`, `-`, `*`, `div`, `mod`, and `%`;
- fixed-width participation in Core `Number` equality, ordering and hashing;
- semantic `===` identity defined by fixed-width family plus mathematical value;
- eight permanent Core prototypes/prelude bindings;
- automatic widening of unrelated Core exact-Integer consumers to fixed-width
  semantic values.

A future FFI or binary facility may use width descriptors only at a boundary:

```text
Protos Integer
    -> explicit width/range conversion
    -> foreign/ABI/binary representation

foreign/ABI/binary representation
    -> explicit conversion
    -> Protos Integer
```

Persistent width-bearing guest values remain possible, but require later
evidence and a separate applicable owner-approved design.

## Owner-approved invariants preserved

AUD009-B7 established:

```text
fixed-width Core semantic families
    REMOVE_NOW_RECONSIDER_LATER

fixed-width capability in Protos generally
    NOT REJECTED

ordinary Standard Library fixed-width abstraction over Integer
    REQUIRED FIRST-CLASS D156 CANDIDATE
```

D156 preserves all three.

The Standard Library wrapper model was evaluated as a first-class candidate and
was rejected for immediate adoption because it would require Protos to select
identity, equality, width catalogue, arithmetic/overflow policy and wrapper
lifetime before a real consumer demonstrates which semantics are needed.

D156 does not reopen the B7 KEEP outcomes for:

- `Number`;
- exact unbounded `Integer`;
- binary64 `Float`;
- numeric literals;
- explicit Integer/Float conversion;
- no implicit numeric promotion;
- retained Integer and Float arithmetic;
- retained Integer/Float equality and ordering;
- numeric hash coherence;
- Float NaN semantics;
- signed-zero equality/identity distinction.

## Repository evidence

AUD009-B7 found no production guest use of `UInt8(...)`, `Int8(...)`,
`UInt16(...)`, `Int16(...)`, `UInt32(...)`, `Int32(...)`,
`UInt64(...)`, or `Int64(...)` outside the Core fixed-width institution.

The current implementation nevertheless contains dedicated fixed-width
machinery across:

- Core prototype/prelude publication;
- `ProtosFixedIntegerValue`;
- numeric conversion;
- same-family arithmetic;
- numeric equality/order/hash/identity;
- transfer/rematerialization;
- diagnostics/rendering;
- host interop;
- widened exact-Integer consumers;
- conformance and architecture tests.

The strongest preservation evidence is that `ProtosFixedIntegerValue` already
exports integral Truffle interop such as `fitsInByte`, `fitsInShort`,
`fitsInInt`, `fitsInLong`, `asByte`, `asShort`, `asInt`, and
`asLong`.

That is useful evidence for reusing width/range conversion machinery at a future
interop boundary. It is not evidence that the values must remain Core numeric
families.

## Comparative research

D156 compared materially different approaches.

### C and C++

Fixed-width integer types are strongly motivated by machine representation,
memory layout, ABI and systems programming. Width is part of the type contract.

Lesson: fixed-width semantics are justified when representation width is itself
a first-class program requirement.

### Rust

Rust has primitive signed/unsigned widths, but the same width exposes distinct
checked, wrapping, saturating and overflowing arithmetic operations.

Lesson: an N-bit width does not uniquely determine arithmetic policy.

### Java

Java has fixed primitive widths and defined narrowing/overflow behavior.

Lesson: fixed widths can be coherent as a permanent VM/language numeric model,
but doing so commits the language to machine-width semantics globally.

### C# / .NET

C# has fixed primitive widths and can vary overflow behavior through checked and
unchecked contexts.

Lesson: width and overflow policy are separable design dimensions.

### Python

Python `int` is arbitrary precision while `struct`, byte conversion and
other binary facilities carry explicit width/signedness/encoding requirements at
the boundary.

Lesson: a language can keep ordinary integer semantics unbounded while placing
width in binary/ABI representation facilities.

### Smalltalk / Pharo

Integer representation may vary internally without making machine width the
semantic identity of ordinary integers. FFI facilities separately describe
foreign representation and conversion.

Lesson: dynamic object languages can keep the mathematical integer model
separate from foreign-width requirements.

### JavaScript

General numeric values are separate from width-bearing typed arrays and
`DataView`; BigInt also provides explicit width-restriction operations.

Lesson: width may be attached to storage/conversion operations rather than to
every integer value.

### Io

Io exposes typed sequence element representations such as fixed-width integer
item types, associating width with representation-oriented sequence storage.

Lesson: a prototype-oriented language need not make every fixed-width
representation a universal semantic numeric family.

## Candidate result

### Candidate A — Core removal, no immediate replacement

Semantically small and viable, but it gives no positive preservation/ownership
rule for already useful width/interop machinery.

### Candidate B — ordinary Standard Library wrappers over Integer

Technically feasible without a native primitive.

It was not selected because immediate adoption would preselect unresolved
questions such as:

- generic versus named widths;
- ordinary object identity versus semantic value identity;
- cross-width/cross-Integer equality;
- hash behavior;
- checked versus wrapping versus saturating versus bit-vector arithmetic;
- width catalogue and lifetime.

### Candidate C — Standard Library API with native backing

Not selected. Current evidence does not prove a need for a new privileged
guest-visible representation merely to provide width/range checking.

### Candidate D — defer to FFI/ABI/binary ownership

Provides the strongest separation between unbounded guest arithmetic and
external representation requirements, but in its original form did not make
preservation of useful existing work explicit.

### Candidate D′ — selected

D′ refines D by adding an explicit preservation/rehome invariant:

```text
remove fixed-width numeric semantics from Core
preserve fixed-width capability
reuse existing width/range/interop machinery where applicable
assign eventual public semantics to the domain that demonstrates the need
do not preselect FFI API, arithmetic or identity policy
```

### Candidate E — retain the current Core model

Not selected. It would reopen the owner-approved B7 removal invariant and retain
substantial permanent Core surface without demonstrated production demand.

## Mandatory arithmetic-policy conclusion

D156 explicitly rejects the assumption:

```text
N-bit integer => one obvious arithmetic semantics
```

Checked, wrapping/modular, saturating and bit-vector behavior serve different
domains.

No one of them is selected as the universal fixed-width arithmetic policy by
D156.

The existing checked same-family Core arithmetic may remain useful evidence or
implementation material for a later checked-width library, but it does not become
FFI semantics automatically.

## Identity/equality conclusion

An ordinary Standard Library wrapper could coherently have:

```text
a: UInt8(7)
b: UInt8(7)

a == b   -> true
a === b  -> false
```

while the old Core represented-value model made family+value semantic identity
possible.

D156 deliberately does not select either identity model for future fixed-width
capability.

A boundary-only FFI descriptor may require no persistent width-bearing guest
value at all.

## Mandatory-question conclusions

```text
CURRENT_NEED_FOR_WIDTH_AS_NUMERIC_IDENTITY=NO_EVIDENCE
FIXED_WIDTH_ARITHMETIC_CORE_NEED=NO
ORDINARY_LIBRARY_WRAPPERS_FEASIBLE=YES
ORDINARY_EQUALITY_HASH_CAN_SUPPORT_WRAPPERS=YES
ORDINARY_EQUALITY_HASH_REPRODUCES_CORE_SEMANTIC_IDENTITY=NO
ONE_WIDTH_IMPLIES_ONE_ARITHMETIC_POLICY=NO
GENERIC_VS_NAMED_WIDTH_API=DEFERRED
FFI_REQUIRES_PERSISTENT_WIDTH_BEARING_GUEST_VALUE=NOT_ESTABLISHED
BOUNDARY_ONLY_WIDTH_CONVERSION=SUPPORTED_DESIGN_PATH
BINARY_ENCODING_CAN_OWN_WIDTH_EXTERNALLY=YES
BYTES_REQUIRES_UINT8_CORE_FAMILY=NO
COMPATIBILITY_IMPACT_CURRENT_PRODUCTION_GUEST_USE=LOW_BASED_ON_REPOSITORY_EVIDENCE
REUSABLE_IMPLEMENTATION_WORK_SHOULD_BE_PRESERVED=YES
```

## Comparative score summary

Scores are 1–5 comparison aids, not decision authority.

| Criterion | A | B | C | D | D′ | E |
|---|---:|---:|---:|---:|---:|---:|
| Correctness / invariant preservation | 5 | 4 | 4 | 5 | 5 | 3 |
| Protos alignment | 5 | 4 | 2 | 5 | 5 | 2 |
| Present-need proportionality | 5 | 2 | 1 | 5 | 5 | 1 |
| Incremental growth | 4 | 4 | 3 | 5 | 5 | 2 |
| Future-option resilience | 5 | 3 | 2 | 5 | 5 | 2 |
| Scalability | 5 | 4 | 5 | 5 | 5 | 5 |
| Conceptual simplicity | 5 | 3 | 2 | 5 | 5 | 1 |
| Portability / implementation freedom | 5 | 5 | 3 | 5 | 5 | 3 |
| Runtime / resource cost | 5 | 3 | 4 | 5 | 5 | 2 |
| Failure / operability | 4 | 3 | 3 | 5 | 5 | 3 |
| Deferral / reversibility / migration | 4 | 4 | 2 | 5 | 5 | 1 |
| Evidence maturity / implementation risk | 4 | 3 | 2 | 5 | 5 | 2 |

Confidence was predominantly HIGH for repository evidence and Core-removal
consequences, and MEDIUM where future FFI/library usage remains hypothetical.

Non-compensating red flags:

- B: overengineering risk from selecting wrapper semantics before need;
- C: stronger overengineering risk from adding privileged native backing;
- E: permanent Core complexity with no demonstrated production consumer;
- D: possible loss-of-work concern if interpreted as deletion rather than
  semantic deferral;
- D′: addresses that D weakness by preserving reusable implementation capability
  without turning it back into Core semantics.

## Incremental-design gate

### Pay for what you need

Current ordinary Protos programs pay only for `Integer` and `Float` semantics.
Width-specific machinery is paid where a future domain explicitly requests it.

### Grow as you need

FFI, binary encoding, bit vectors, modular arithmetic or a checked-width library
can each add their own width-bearing contract later without changing the
fundamental unbounded `Integer` model.

### Cost of deferral / reversibility

Adding boundary descriptors later is bounded because exact Integer values and
range checking already exist.

The expensive future case would be discovering that width must become permanent
Core semantic identity. Current repository evidence does not justify paying that
cost today.

Preserving reusable range/interop machinery further reduces implementation
reintroduction cost.

### Smallest sufficient solution

The smallest sufficient current semantic design is:

```text
Core Number/Integer/Float
+
no persistent fixed-width numeric family
+
explicit preservation of reusable width/range/interop implementation machinery
for future domain ownership
```

## GITHUB021 invariant/delta check

Applicable owner-approved invariants:

```text
B7_FIXED_WIDTH_CORE_CLASSIFICATION=REMOVE_NOW_RECONSIDER_LATER
B7_FIXED_WIDTH_CAPABILITY_REJECTED=NO
B7_STANDARD_LIBRARY_RELOCATION_CANDIDATE=MANDATORY_FIRST_CLASS
B7_RETAINED_NUMBER_INTEGER_FLOAT_MODEL=KEEP
```

Selected D′:

```text
PRESERVES_FIXED_WIDTH_CORE_REMOVAL=YES
PRESERVES_FIXED_WIDTH_CAPABILITY=YES
STANDARD_LIBRARY_CANDIDATE_EVALUATED_FIRST_CLASS=YES
REOPENS_NUMBER_INTEGER_FLOAT=NO
REOPENS_NUMERIC_LITERAL_OR_FLOAT_MODEL=NO
NEW_PUBLIC_FFI_API=NO
NEW_FIXED_WIDTH_ARITHMETIC_POLICY=NO
NEW_FIXED_WIDTH_IDENTITY_POLICY=NO
PRESERVES_REUSABLE_IMPLEMENTATION_WORK=YES
HIDDEN_REOPENING=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Intentionally deferred

D156 intentionally does not decide:

- public FFI/interop namespace or spelling;
- whether a future API uses named `UInt8`/`Int32` descriptors or a generic
  width/signedness descriptor;
- persistent width-bearing guest values;
- wrapper identity/equality semantics;
- checked/wrapping/saturating/bit-vector arithmetic library APIs;
- exact width catalogue beyond demonstrated domain needs;
- ABI calling conventions;
- binary layout/endianness APIs;
- native backing for future guest-visible values;
- FFI itself.

Those decisions require evidence from the future owning domain.

## Reconsideration triggers

Reconsider persistent fixed-width guest values when real Protos code requires
width and/or signedness to remain semantic information after crossing a boundary.

Relevant triggers include:

- FFI/ABI APIs whose values must retain width after calls;
- binary structures or memory-mapped data whose guest model materially benefits
  from width identity;
- cryptographic/bit-vector workloads;
- repeated width-preserving arithmetic;
- schemas requiring width-bearing guest values rather than encoding metadata.

## Implementation routing

Implementation is separately owned by:

```text
I052 / guillermomolina/protos#628
Remove Core fixed-width integer families and preserve interop capability
```

I052 must remove the obsolete Core semantic institution while preserving/rehome
useful width/range/interop machinery where it can remain non-semantic or belongs
to a future explicit boundary.

A public FFI library is **not** authorized by D156 or I052 merely because the
implementation material is preserved.

## Ratification

```text
ISSUE=guillermomolina/protos#616
APPROVAL_COMMENT=5739410881
DATE=2026-09-19
SELECTED_CANDIDATE=D_PRIME

CORE_FIXED_WIDTH_FAMILIES=REMOVE
CORE_FIXED_WIDTH_CONVERSION_INSTITUTION=REMOVE
CORE_FIXED_WIDTH_ARITHMETIC_INSTITUTION=REMOVE
FIXED_WIDTH_CAPABILITY_REJECTED=NO
REUSABLE_WIDTH_RANGE_INTEROP_WORK=PRESERVE_AND_REHOME_WHERE_APPLICABLE

IMMEDIATE_STANDARD_LIBRARY_FIXED_WIDTH_VALUES=NO
PUBLIC_FFI_API=DEFERRED
PERSISTENT_WIDTH_BEARING_GUEST_VALUES=DEFERRED
FIXED_WIDTH_ARITHMETIC_POLICY=DEFERRED
FIXED_WIDTH_IDENTITY_POLICY=DEFERRED
NATIVE_BACKING=NOT_PREAPPROVED

IMPLEMENTATION_ROUTE=I052/#628
SPECIFICATION_CHANGED_BY_DECISION_RECORD=NO
IMPLEMENTATION_CHANGED_BY_DECISION_RECORD=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
