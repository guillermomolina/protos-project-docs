# PERF027 — Machine-word representation for small Integers

Status: **PUBLISHED / VALIDATED**  
Date: 2026-10-03  
Formal owner: `PERF027 / guillermomolina/protos#779`

This durable, non-normative record retains the published small-Integer
representation implementation and its maintainer-reported validation evidence.
It does not claim a measured performance magnitude.

## Exact product identity

```text
PROTOS_REPOSITORY=guillermomolina/protos
BASE_REVISION=51bf02af1fe0fd6247ee17c8a1a764ced77d2732
BASE_VERSION=0.3.157-SNAPSHOT

PRODUCT_REVISION=93cc30cf26bdb2ca2cbf13c282e3cfe22919bca9
PRODUCT_VERSION=0.3.158-SNAPSHOT
COMMIT_SUBJECT=PERF027: use machine-word representation for small Integers
```

Published changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardArrayProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardEncodingProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardIntegerProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumberEqualityProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardNumberOrderingProtocol.java
M src/main/java/com/guillermomolina/protos/execution/ProtosStandardStringProtocol.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosArrayValue.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosIdentity.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosIntegerValue.java
M src/test/java/com/guillermomolina/protos/execution/ProtosStandardIntegerArithmeticTest.java
A src/test/java/com/guillermomolina/protos/runtime/ProtosIntegerValueRepresentationTest.java
```

No specification file changed.

## Sequencing reconciliation

The original PERF027 issue deliberately required an exact-revision measurement
of PERF027-A before a machine-width representation experiment. During the later
PERF025 pay-as-you-grow implementation workflow, the project owner explicitly
directed the representation implementation to proceed **without a measurement
slice** and requested no benchmark work.

That explicit owner direction supersedes the earlier sequencing restriction for
this implementation publication only.

It does **not** mean that PERF027-B was executed, satisfied, or replaced:

```text
PERF027_B_EXECUTED=NO
PERF027_B_MEASUREMENT_SATISFIED=NO
REPRESENTATION_IMPLEMENTATION_EXPLICITLY_REQUESTED=YES
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The representation work therefore publishes as structural pay-as-you-grow
implementation evidence, not as causal performance attribution.

## Implemented representation

Protos still has one semantic exact, unbounded `Integer` family.

`ProtosIntegerValue` now uses a hidden canonical dual physical representation:

```text
signed-long value
    -> smallValue: long
    -> bigValue: null

outside signed-long range
    -> bigValue: BigInteger
```

The representation category is not guest-visible.

Construction from `BigInteger` canonicalizes values that fit the complete
signed-`long` range back to the machine-word form.

## Exact arithmetic behavior

For two machine-word Integers:

- `+`, binary `-`, and `*` use exact primitive arithmetic;
- overflow transparently promotes to `BigInteger`;
- `div` and `mod` use machine-word arithmetic when exact;
- `Long.MIN_VALUE / -1` promotes instead of overflowing;
- a later arbitrary-precision result that fits signed `long` canonicalizes back
  to the small representation.

The existing exact Integer-to-binary64 division authority remains unchanged and
may materialize arbitrary-precision operands because that boundary genuinely
requires exact unbounded input.

## Equality, ordering, identity and interop

Integer/Integer equality, ordering, and semantic identity use the canonical
representation directly and do not require `BigInteger` materialization on
small/small comparisons.

Integral interop width tests and projections consume the machine-word
representation directly when possible.

Historical hash semantics remain unchanged. Boundaries that genuinely require
an arbitrary-precision Java value may still call `value()`; for a small
Integer that method materializes a `BigInteger` only at that explicit boundary.

Actor/value-transfer paths retain semantic Integer value and re-canonicalize on
construction. No SmallInteger/BigInteger semantic family is exposed.

## Bounded consumer follow-up

During full Java validation, an allowlisted integration test exceeded its
existing slow-test budget twice:

```text
ProtosExternalPackagePlanningPreflightTest
  first full-Java attempt: 28.02 s, budget 27 s
  second full-Java attempt: 27.30 s, budget 27 s
```

The second Surefire report split the class time as:

```text
sameVerifiedCustodiesPlanAfterOriginalSourcesAreDeletedAndRemainBorrowed = 24.087 s
failedPlanningTerminatesProcessWithoutClosingBorrowedCustody             =  3.217 s
total                                                                    = 27.304 s
```

No functional Java test failure was reported; the gate failure was the
TEST008 slow-test budget check.

Inspection found that several already-bounded consumers still converted a small
`ProtosIntegerValue` through `value()` to `BigInteger` merely to recover an
`int` index or an octet. The same product slice therefore added exact internal
`int` projection and used it only where the host domain is already bounded:

- Array indexed access/update;
- String scalar indexing;
- Encoding octet extraction/construction.

`ProtosArrayValue` gained internal `int` accessors for those paths while
retaining its arbitrary-precision-facing APIs.

This was a structural correction to avoid immediately inflating the new small
representation at bounded consumers. The later PASS does **not** prove that
these materializations caused the earlier wall-clock excess, and this record
does not make that causal claim.

## Regression coverage

New `ProtosIntegerValueRepresentationTest` covers:

- signed-`long` canonicalization;
- values immediately outside the signed-`long` range;
- exact small arithmetic;
- promotion on add/subtract/multiply overflow;
- arbitrary-precision result normalization back to the small form;
- signed quotient/remainder behavior;
- exact `Long.MIN_VALUE / -1` promotion;
- representation-independent semantic identity/comparison; and
- exact internal `int` projection boundaries.

Existing arithmetic, interop, numeric equality/ordering, guarded Integer,
Actor-transfer, Array, String, Encoding and package-planning tests cover the
affected integration surfaces.

## Validation provenance

The maintainer executed the initial focused regression set after the core
representation patch and reported PASS:

```text
ProtosIntegerValueRepresentationTest
ProtosStandardIntegerArithmeticTest
ProtosIntegralInteropTest
ProtosStandardNumberEqualityTest
ProtosStandardNumberOrderingProtocolTest
ProtosPerf027AGuardedIntegerDeferredActivationTest
ProtosGuardedLookupTest
ProtosActorValueTransferTest
```

After the bounded-consumer follow-up, the maintainer executed and reported PASS
for:

```text
ProtosIntegerValueRepresentationTest
ProtosStandardIntegerArithmeticTest
ProtosStandardArrayFactoryTest
ProtosStandardStringProtocolTest
ProtosStandardEncodingProtocolTest
ProtosExternalPackagePlanningPreflightTest
```

Final Java lane:

```text
make test-java
JAVA_SLOW_TEST_GUARD=PASS
THRESHOLD_SECONDS=10
REPORTED_TEST_CLASSES=490
ALLOWLISTED_SLOW_TESTS=1
UNALLOWLISTED_SLOW_TESTS=0
OVER_BUDGET_SLOW_TESTS=0
```

Final Protos lane:

```text
make test-protos
RESULT=PASS
```

Publication-sensitive checks reported before commit:

```text
git diff --check = PASS
LOCAL_HEAD_BEFORE_PUBLICATION=51bf02af1fe0fd6247ee17c8a1a764ced77d2732
ORIGIN_MAIN_BEFORE_PUBLICATION=51bf02af1fe0fd6247ee17c8a1a764ced77d2732
PUBLICATION=PASS
```

No benchmark, profiler, allocation measurement, or retained performance
comparison was executed for this representation publication.

## Acceptance result

```text
PERF027_MACHINE_WORD_INTEGER_REPRESENTATION=PASS

ONE_SEMANTIC_UNBOUNDED_INTEGER_FAMILY=YES
SMALL_INTEGER_PHYSICAL_CARRIER=SIGNED_LONG
ARBITRARY_PRECISION_FALLBACK=BIGINTEGER
TRANSPARENT_OVERFLOW_PROMOTION=YES
BIG_TO_SMALL_RECANONICALIZATION=YES
LONG_MIN_DIV_NEGATIVE_ONE_EXACT=YES

INTEGER_EQUALITY_REPRESENTATION_INDEPENDENT=YES
INTEGER_ORDERING_REPRESENTATION_INDEPENDENT=YES
INTEGER_IDENTITY_REPRESENTATION_INDEPENDENT=YES
INTEGRAL_INTEROP_PRESERVED=YES
ACTOR_TRANSFER_PRESERVED=YES

BOUNDED_ARRAY_INDEX_BIGINTEGER_INFLATION=REMOVED
BOUNDED_STRING_INDEX_BIGINTEGER_INFLATION=REMOVED
BOUNDED_ENCODING_OCTET_BIGINTEGER_INFLATION=REMOVED

BYTECODE_LONG_CARRIER_INTRODUCED=NO
BOXING_ELIMINATION_LONG_ADDED=NO
GUEST_VISIBLE_SMALLINTEGER_CATEGORY=NO

FOCUSED_VALIDATION=PASS
FINAL_JAVA_LANE=PASS
FINAL_PROTOS_LANE=PASS
FULL_INTEGRATED_VALIDATION=PASS

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PERFORMANCE_EFFECT_MEASURED=NO
PERFORMANCE_MAGNITUDE_CLAIMED=NO
PUBLICATION=PASS
```

## Relationship to PERF025

PERF025's post-F1 pay-as-you-grow audit identified mandatory
`ProtosIntegerValue(BigInteger)` storage as a residual cost surface. PERF027
owns the Integer-specific invocation/representation follow-up.

The immediately preceding PERF025 publication
`51bf02af1fe0fd6247ee17c8a1a764ced77d2732` removed rich invocation machinery
from successful exact canonical local Integer arithmetic while deliberately
leaving representation unchanged.

This PERF027 publication consumes that now-direct arithmetic boundary and removes
the mandatory `BigInteger` representation for values that fit signed `long`.

PERF025 remains a separate open workstream. This record closes only the
published representation implementation checkpoint; it does not close PERF025
or assert completion of every remaining PERF027 measurement/coordination item.
