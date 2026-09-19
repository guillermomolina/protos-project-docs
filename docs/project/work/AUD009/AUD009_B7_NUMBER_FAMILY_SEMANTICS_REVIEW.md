# AUD009-B7 — Number-family semantics complexity review

Status: **COMPLETE — OWNER-APPROVED CLASSIFICATION**

Nature: non-normative AUD009 evidence/classification ledger

Tracking issue: `guillermomolina/protos#614`

Parent audit: `guillermomolina/protos#522` — AUD009

Evidence baseline: `8752b03600c11edb893b907f00a0dec65f2611b1`

Closure revalidation revision: `fcd3d8f9d489ae5210ef2076037dfaa074f7654b`

Delta from evidence baseline to closure revision touches only
`CONTRIBUTING.md` and `spec/PROTOS_SPEC_CHANGELOG.md`; no B7 numeric semantic
or runtime owner changed.

Specification changed by this record: **NO**

Implementation changed by this record: **NO**

Owner approval provenance: `guillermomolina/protos#614`, issue comment
`5739027775`, 2026-09-19.

Derived decision route:
`D156 / guillermomolina/protos#616 — Fixed-width integer placement and capability model`.

## Purpose and boundary

AUD009-B7 reviewed the standard Number-family model under the retrospective
complexity/necessity methodology.

The audit separates the numeric mechanisms with demonstrated Core utility
(`Number`, unbounded `Integer`, binary64 `Float`) from the eight current
fixed-width Integer semantic families.

B7 does not itself alter normative specification or executable behavior.

## Final classification ledger

```text
Number prototype                               KEEP
ordinary unbounded Integer                     KEEP
Float binary64                                 KEEP

Integer literal model                          KEEP
Float literal model                            KEEP
explicit Integer conversion                    KEEP
explicit Float conversion                      KEEP
no implicit numeric promotion                  KEEP

Integer arithmetic                             KEEP
Integer div/mod/%                              KEEP
Integer / -> Float                             KEEP
Float IEEE arithmetic                          KEEP

cross-family Integer/Float equality            KEEP
cross-family Integer/Float ordering            KEEP
numeric hash coherence                         KEEP
one semantic Float NaN                         KEEP
NaN == NaN false / NaN === NaN true            KEEP
signed zero equality/identity split            KEEP
IEEE exceptional Float results                 KEEP

UInt8                                          REMOVE_NOW_RECONSIDER_LATER
Int8                                           REMOVE_NOW_RECONSIDER_LATER
UInt16                                         REMOVE_NOW_RECONSIDER_LATER
Int16                                          REMOVE_NOW_RECONSIDER_LATER
UInt32                                         REMOVE_NOW_RECONSIDER_LATER
Int32                                          REMOVE_NOW_RECONSIDER_LATER
UInt64                                         REMOVE_NOW_RECONSIDER_LATER
Int64                                          REMOVE_NOW_RECONSIDER_LATER
fixed-width conversion factories               REMOVE_NOW_RECONSIDER_LATER
fixed-width arithmetic/overflow institution    REMOVE_NOW_RECONSIDER_LATER

implicit numeric coercion                      ABSENT / RETAIN ABSENCE
Int/UInt generic width prototypes              ABSENT / RETAIN ABSENCE
Decimal/Rational/Complex                       ABSENT / RETAIN ABSENCE
```

The fixed-width removal classification is specifically from **Core**.

Fixed-width capability in Protos generally is **not rejected**.

An ordinary Standard Library fixed-width abstraction over unbounded Integer is a
mandatory first-class candidate in D156.

## Retained Core numeric backbone

The retained portable numeric hierarchy is:

```text
Number -> Object
Integer -> Number
Float   -> Number
```

`Number` remains justified because it owns shared numeric equality, ordering
and hashing behavior. Removing it would duplicate or hide that shared numeric
protocol.

Ordinary `Integer` remains exact and unbounded. Production Standard Library and
Tool code relies heavily on exact integer arithmetic, `div`, `mod`, indexes,
sizes and counts.

`Float` remains the exact IEEE 754-2019 binary64 family. Integer division uses
Float as the ordinary fractional result family, and production code uses Float
conversion/parsing.

Classification: **KEEP**.

## Literals and explicit conversion

Radix/decimal integer literals continue to produce ordinary unbounded Integer;
decimal point/exponent forms produce Float.

Literal spelling does not create width-selected families.

Explicit conversion between retained Integer and Float remains visible and
bounded. Core retains no implicit Integer/Float arithmetic promotion.

Classification: **KEEP**.

## Arithmetic

Ordinary Integer arithmetic remains exact.

`div`, `mod` and `%` retain the exact quotient/remainder law.

Integer `/` returns correctly rounded binary64 Float from the exact rational
quotient.

Float arithmetic remains binary64 with roundTiesToEven and gradual underflow.

Classification: **KEEP**.

## Numeric equality, ordering, identity and hashing

Retained Integer/Float equality and ordering continue to compare exact
mathematical values without converting one operand into the other.

Semantic identity remains family-sensitive:

```text
1 == 1.0   -> true
1 === 1.0  -> false
```

Normal numeric hashing remains coherent with numeric equality.

Classification: **KEEP**.

## Float NaN and signed zero

Core retains one semantic Float NaN value:

```text
NaN == NaN   -> false
NaN === NaN  -> true
```

Host NaN payload/sign representation is not portable semantic identity.

Signed zero remains:

```text
0.0 == -0.0   -> true
0.0 === -0.0  -> false
```

because zero sign affects later binary64 behavior.

IEEE exceptional arithmetic results remain Float values rather than automatic
Protos Errors.

Classification: **KEEP**.

## Fixed-width Core families

Current Core standardizes:

```text
UInt8  Int8
UInt16 Int16
UInt32 Int32
UInt64 Int64
```

as eight distinct semantic value families.

That creates permanent Core obligations for:

- eight public prototypes/bindings;
- eight semantic identity families;
- eight conversion factories;
- dedicated represented-value machinery;
- same-family checked arithmetic;
- overflow/range behavior;
- equality/order/hash/identity integration;
- Actor/P transfer;
- interop;
- widened exact-integer domains in other Core APIs;
- substantial conformance and architecture surface.

AUD009-B7 found no production guest use of these constructors outside their own
Core definitions.

Standard Bytes does not require UInt8: byte elements are semantic Integer values
restricted to `0..255`.

Therefore the current eight-family Core institution is classified:

**REMOVE_NOW_RECONSIDER_LATER**.

This classification includes the dedicated fixed-width conversion factories and
fixed-width arithmetic/overflow institution.

## Fixed-width capability is not rejected

The audit finding is specifically about **Core placement**, not whether Protos
may ever provide fixed-width values.

B7 verified that ordinary Protos Standard Library code already has the mechanisms
needed to construct a fixed-width abstraction over Integer:

- `Integer.recognizes`;
- exact arbitrary-precision arithmetic;
- explicit min/max range validation;
- ordinary object construction;
- ordinary frozen value objects;
- ordinary `==` and `hash`;
- ordinary operator/message selectors;
- ordinary Error signaling.

Existing sources such as `std:math/Integer`, `std:collections/Range`, and
`std:test/Test` demonstrate these mechanisms.

Therefore no `ProtosFixedIntegerValue` primitive is presumed necessary merely
to expose a checked fixed-width abstraction.

## D156 mandatory alternatives

The approved B7 classification is routed to D156/#616.

D156 must compare at least:

```text
A. Remove Core fixed-width families with no immediate replacement.

B. Relocate fixed-width values to ordinary Standard Library code over Integer.

C. Standard Library fixed-width API with native backing only if evidence
   proves ordinary source-backed values insufficient.

D. Defer replacement until FFI / ABI / binary-structure needs establish the
   correct width-bearing abstraction.

E. Retain the current eight Core families, as the strongest falsification case.
```

Candidate B is mandatory and first-class.

B7 itself does not decide:

- library namespace;
- generic versus eight named factories;
- wrapper layout;
- whether equal wrappers are `===`;
- checked versus wrapping versus saturating policy;
- exact width catalog;
- FFI/ABI integration;
- native backing.

## Identity consequence to evaluate

Moving fixed-width values to ordinary library objects may naturally yield:

```text
a: UInt8(7)
b: UInt8(7)

a == b    -> true
a === b   -> false
```

B7 does not pre-approve that model; D156 must evaluate it explicitly.

## Reconsideration trigger

Reconsider fixed-width semantic values when real Protos code needs width and/or
signedness as semantic information rather than merely as external encoding
metadata.

Relevant triggers include:

- FFI/ABI;
- binary structures;
- memory-mapped data;
- cryptographic/bit-vector APIs;
- repeated width-preserving arithmetic;
- schemas whose guest values materially benefit from width identity.

## Reconciliation boundaries

- B6 identity/equality/hash framework remains unchanged.
- Array/String/Bytes exact-Integer consumers are downstream consequences only if
  D156 later removes Core fixed-width families.
- Standard Bytes remains Integer-octet based.
- detailed String/Bytes/Array/Map audits remain later AUD009-B slices.
- FFI/binary-structure design remains outside B7.
- runtime representation cleanup remains implementation work after D156.
- I032 historical implementation cost is sunk cost and not evidence of
  continuing necessity.

## Owner approval and routing

```text
ISSUE=guillermomolina/protos#614
CHECKPOINT_COMMENT=5739000683
REFINEMENT_COMMENT=5739020840
APPROVAL_COMMENT=5739027775
DATE=2026-09-19

FIXED_WIDTH_CORE_CLASSIFICATION=REMOVE_NOW_RECONSIDER_LATER
FIXED_WIDTH_CAPABILITY_REJECTED=NO
STANDARD_LIBRARY_RELOCATION_CANDIDATE=MANDATORY
DERIVED_DECISION=D156/#616
DERIVED_DECISION_STATE=NEEDS_USER_DECISION
NORMATIVE_CHANGE_AUTHORIZED_BY_B7=NO
IMPLEMENTATION_CHANGE_AUTHORIZED_BY_B7=NO
```

## Closure checklist

```text
NUMBER=KEEP
INTEGER=KEEP
FLOAT=KEEP
NUMERIC_LITERALS=KEEP
INTEGER_FLOAT_CONVERSION=KEEP
NO_IMPLICIT_PROMOTION=KEEP
INTEGER_ARITHMETIC=KEEP
FLOAT_ARITHMETIC=KEEP
NUMERIC_EQUALITY_ORDER_HASH=KEEP
FLOAT_NAN_MODEL=KEEP
FLOAT_SIGNED_ZERO=KEEP

CORE_UINT8=REMOVE_NOW_RECONSIDER_LATER
CORE_INT8=REMOVE_NOW_RECONSIDER_LATER
CORE_UINT16=REMOVE_NOW_RECONSIDER_LATER
CORE_INT16=REMOVE_NOW_RECONSIDER_LATER
CORE_UINT32=REMOVE_NOW_RECONSIDER_LATER
CORE_INT32=REMOVE_NOW_RECONSIDER_LATER
CORE_UINT64=REMOVE_NOW_RECONSIDER_LATER
CORE_INT64=REMOVE_NOW_RECONSIDER_LATER
FIXED_WIDTH_FACTORIES=REMOVE_NOW_RECONSIDER_LATER
FIXED_WIDTH_ARITHMETIC=REMOVE_NOW_RECONSIDER_LATER

FIXED_WIDTH_CAPABILITY_RETAINABLE_OUTSIDE_CORE=YES
STANDARD_LIBRARY_RELOCATION_MANDATORY_D156_CANDIDATE=PASS
OWNER_APPROVAL_PROVENANCE=PASS
REMOVAL_ROUTE=D156/#616
REMOVAL_IMPLEMENTED_BY_AUDIT=NO
SPECIFICATION_CHANGED_BY_AUDIT=NO
IMPLEMENTATION_CHANGED_BY_AUDIT=NO
AUD009_B7_CLASSIFICATION=COMPLETE
```

AUD009-B7 is complete once this durable record and the required live GitHub
closure postconditions are verified.
