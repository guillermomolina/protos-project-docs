# LIB017 — Integer range iteration API decision

## Decision

LIB017 selects **Candidate G-prime — bounded unit iteration helpers only**.

```text
LIB017_STATUS=SELECTED
SELECTED_CANDIDATE=G_PRIME
PACKET_REVISION=66f63181b37b795f65d7972aa8c11753ac0e2de0
OWNER_APPROVAL_PROVENANCE=PASS
IMPLEMENTATION_AUTHORIZED=YES
SYNTAX_AUTHORIZED=NO
```

The owner explicitly approved the exact candidate in the active LIB017
interaction:

```text
aprobar G′
```

## Approved public Standard Library surface

The selected module identity is:

```text
std:collections/Range
```

The initial public operations are exactly:

```text
each(start, stop, block)
reverseEach(start, stop, block)
```

No first-class Range value or constructor is part of this decision.

## Bound model

Both operations use one half-open mathematical interval:

```text
start <= x < stop
```

Reversed bounds do not imply descending iteration:

```text
start >= stop
    -> empty interval
```

The two operations differ only in traversal order:

```text
each
    -> start, start + 1, ..., stop - 1

reverseEach
    -> stop - 1, stop - 2, ..., start
```

## Input domains

The initial contract accepts:

```text
start: ordinary unbounded Integer
stop:  ordinary unbounded Integer
```

The initial contract rejects:

```text
fixed-width Integer-family bounds
Float bounds
```

No host-width integer limit is observable.

Bounds are evaluated once under ordinary call evaluation before iteration begins.

## Callback contract

For every visited value:

```text
block(current)
```

is one ordinary polymorphic invocation with exactly one supplied argument.

The supplied callback argument is an ordinary unbounded Integer.

The selected contract is:

```text
EAGER_CALLBACK_VALIDATION=NO
EMPTY_RANGE_INSPECTS_CALLBACK=NO
CALLBACK_RESULT=IGNORED
NORMAL_OPERATION_RESULT=NULL
ERROR_AND_NONLOCAL_CONTROL=PROPAGATE
IMPLICIT_FUTURE_AWAIT_OR_ADOPTION=NO
```

A non-invokable callback is therefore observed only if an iteration actually
attempts a callback invocation.

## Allocation / representation

The selected API does not materialize the represented sequence.

It requires only bounded local iteration state and must not allocate an Array or
another collection proportional to interval size merely to perform iteration.

Mathematically huge ordinary Integer bounds remain valid. Running a huge number
of callbacks may consume correspondingly huge execution time/resources, but the
library must not truncate, wrap, or reject the interval merely because its
cardinality exceeds host-sized limits.

## Migration rule

Existing manual loops may be migrated only when their current upper/lower bound
is semantically stable for the whole traversal.

For example, a loop that re-evaluates a mutable `source.size()` on every
iteration is not automatically equivalent to:

```protos
Ranges.each(start, source.size(), block)
```

because the helper evaluates the bound once.

State-driven scans, parser loops, variable-width stepping and mutable
termination-condition loops remain ordinary `while` use cases.

## Explicitly deferred

LIB017 does not decide or add:

- first-class Range values;
- a Range constructor;
- Range equality/hash/identity;
- arbitrary positive or negative step;
- fixed-width Integer bounds;
- Float ranges;
- membership;
- cardinality/size;
- indexing;
- materialization to Array;
- inclusive Range values;
- generic Comparable intervals;
- slicing/subranges;
- beginless/endless ranges;
- generic Iterable/Iterator;
- lazy streams;
- comprehensions;
- range syntax/operators.

A future syntax decision requires a separate Dxxx if real usage demonstrates
that the ordinary library API is materially insufficient.

## Compatibility / growth model

The decision is intentionally future-compatible rather than
future-preimplemented.

Later work may add:

- another owner-local operation;
- arbitrary-step iteration;
- a first-class compact Range descriptor;
- membership/cardinality;
- collection integration;
- or, through a separate Dxxx, language syntax.

None of those additions requires changing the selected semantics of
`each(start, stop, block)` or `reverseEach(start, stop, block)`.

## Consistency check

The selected candidate preserves:

```text
ORDINARY_INTEGER_UNBOUNDED
HOST_WIDTH_UNOBSERVABLE
CLOSURE_WHILE_REMAINS_GENERAL_LOOP
NO_GENERIC_ITERABLE
NO_FLOAT_COERCION
NO_SLICING_DECISION
NO_RANGE_SYNTAX_DECISION
RANGE_SYNTAX_REVISIT_AFTER_REAL_USAGE
```

Material new public Standard Library surface:

```text
std:collections/Range.each
std:collections/Range.reverseEach
```

No hidden syntax, Core semantic, iterator hierarchy, Range-value identity, or
step model is selected.

## Implementation state

LIB017 is closed.

The owner-approved G-prime surface was published to `guillermomolina/protos` as
commit `d1bbab2c1c1023e980b43ca01e7b2adafcac05f8` with implementation version
`0.3.44-SNAPSHOT`.

Closure evidence for that exact candidate:

```text
PUBLICATION_BASE=578f693b19c7daf739bef69bfbc3d3e1f9abb1b5
PUBLISHED_SHA=d1bbab2c1c1023e980b43ca01e7b2adafcac05f8
PROTOS_TEST_TOOL=1313 passed, 0 failed
FULL_UNRESTRICTED_MAVEN_TEST=PASS
PUBLICATION_VALIDATION=PASS
CI_RUN=35354672105
CI_RUN_NUMBER=1894
CI_HEAD_SHA=d1bbab2c1c1023e980b43ca01e7b2adafcac05f8
CI_CONCLUSION=success
LIB017_STATUS=CLOSED
```

The published implementation adds only
`std:collections/Range.each(start, stop, block)` and
`std:collections/Range.reverseEach(start, stop, block)` under the selected
G-prime semantics. The explicitly deferred Range-value, arbitrary-step,
membership, indexing, slicing, generic Iterable/Iterator, inclusive-range, and
range-syntax questions remain deferred and are not implicitly reopened by this
closure.
