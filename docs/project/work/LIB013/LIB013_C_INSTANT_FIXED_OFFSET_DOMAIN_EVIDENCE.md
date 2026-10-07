# LIB013-C — Instant and fixed-offset domain implementation evidence

Status: **COMPLETE**

Owning work item: `guillermomolina/protos#430`

Slice: `LIB013-C — Instant and fixed-offset domain`

Nature: implementation/validation evidence; **non-normative**

Product revision: `d6981daa330a2a3a767865e9cd88a6fb9f626e81`

Immediate product base: `0abd607a3a354ecd2b6551e344661d4b727d7910`

Product version: `0.3.246-SNAPSHOT`

Commit subject:

~~~text
LIB013-C: add instant and fixed-offset domain
~~~

Ratified design owner: LIB013-0 Candidate A — Pure Values First + Explicit Authorities.

## Publication result

LIB013-C is published in `guillermomolina/protos` at the exact product revision above.

The C commit is exactly one commit ahead of
`0abd607a3a354ecd2b6551e344661d4b727d7910`.
Other Protos work landed between the historical LIB013-B publication and C, so
this record uses the C commit's actual immediate base rather than treating B as
its direct parent.

The slice adds:

- `std:datetime/Offset`;
- `std:datetime/Instant`;
- `std:datetime/OffsetDateTime`;
- exact Instant/Duration arithmetic;
- exact fixed-offset civil/timeline conversion;
- explicit same-instant comparison and offset-preserving conversion;
- canonical family/prototype recognition across all datetime families.

No named timezone, tzdb, clock, parsing, formatting, locale, timer, scheduler or
TOML authority is added.

## Published Offset surface

Public module and constructor:

~~~text
std:datetime/Offset
Offset(seconds)
~~~

Public state:

~~~text
seconds
~~~

Contract:

- exact signed Integer whole seconds;
- inclusive range `-64800 .. 64800`;
- one-second resolution;
- fixed displacement only;
- not a timezone identity;
- no zone name/rules/tzdb authority.

Published ordering:

~~~text
Offset.compare(left, right)
~~~

Offset equality is exact fixed-displacement equality with coherent hash and
normal Map-key behavior.

## Published Instant surface

Public module and constructor:

~~~text
std:datetime/Instant
Instant(nanoseconds)
~~~

Public state:

~~~text
nanoseconds
~~~

The Integer is the exact signed count of uniform nanoseconds from the conceptual
reference:

~~~text
1970-01-01T00:00:00Z
~~~

The timeline is Unix-like and uses uniform 86400-second days.

The model does not represent leap seconds, TAI or GPS time and does not claim
physically exact UTC.

There is no artificial host/JVM timestamp range.

Published operations:

~~~text
Instant.compare(left, right)
Instant.addDuration(instant, duration)
Instant.subtractDuration(instant, duration)
Instant.durationBetween(start, end)
~~~

`durationBetween(start, end)` answers the exact
`Duration(end.nanoseconds - start.nanoseconds)`.

## Published OffsetDateTime surface

Public module and constructor:

~~~text
std:datetime/OffsetDateTime
OffsetDateTime(dateTime, offset)
~~~

Public state:

~~~text
dateTime
offset
~~~

The components are a recognized `LocalDateTime` and a recognized `Offset`.

Published conversion/comparison operations:

~~~text
OffsetDateTime.toInstant(value)
OffsetDateTime.fromInstant(instant, offset)
OffsetDateTime.sameInstant(left, right)
OffsetDateTime.withOffsetSameInstant(value, offset)
~~~

Equality is structural:

- same LocalDateTime;
- same Offset.

There is deliberately no `OffsetDateTime.compare`.

Two structurally unequal values may identify the same Instant and
`sameInstant` handles that case explicitly.

## Civil/timeline conversion

`toInstant` computes an exact proleptic-Gregorian day number relative to
1970-01-01, combines it with the LocalDateTime's Time and subtracts the fixed
offset in seconds.

`fromInstant`:

1. adds the requested fixed offset to the Instant;
2. floor-normalizes signed nanoseconds into whole local days plus a non-negative
   nanosecond-of-day remainder;
3. reuses `Period.addToDate(Period(0, 0, days), Date(1970, 1, 1))` for
   day-count -> Date conversion;
4. constructs the exact Time from the normalized remainder;
5. returns `OffsetDateTime(LocalDateTime(date, time), offset)`.

This preserves correct pre-epoch behavior, including:

~~~text
Instant(-1) @ Offset(0)
=> 1969-12-31 23:59:59.999999999 +00:00
~~~

A requested local civil representation outside Date's supported
`-9999..9999` year range signals `Error`.

Instant itself remains unbounded.

## Canonical datetime family recognition

LIB013-C resolves the structural-family collision exposed by Duration and Instant.

All datetime families now follow the canonical factory/prototype discipline:

- Date;
- Time;
- LocalDateTime;
- Duration;
- Period;
- Offset;
- Instant;
- OffsetDateTime.

Each constructed value has the corresponding module as its immediate delegation
parent.

Each module publishes:

~~~text
recognizes(value)
~~~

Recognition checks:

- exact immediate canonical parent;
- exact expected local-slot shape;
- valid family-specific state.

Coincidental matching slots under another parent and transitive-only ancestry do
not establish membership.

This makes:

~~~text
Duration(5) == Instant(5)
~~~

false and causes family-specific operations to reject values from the wrong
family.

The A/B public data contracts remain unchanged.

### Frozenness observation limitation

Constructors freeze all published datetime values.

However, current Core reflection exposes no direct frozenness predicate usable
by guest Protos code. Therefore the published `recognizes(value)` functions
cannot independently require or observe frozenness.

The C CHANGELOG explicitly records this limitation.

This is a known implementation-model boundary, not a datetime-specific runtime
exception.

## Actor transfer boundary

Datetime values remain ordinary Protos object graphs.

The family/prototype migration does not add:

- a native datetime runtime kind;
- a datetime-specific Actor transfer exception;
- mutable shared temporal authority.

The current datetime values remain non-transferable across Actors under the
existing general transfer limitation.

## License repair

The C commit restores the missing final line of the Adaptive Public License
notice in:

- `protos/lib/datetime/Date.protos`;
- `protos/lib/datetime/Time.protos`;
- `protos/lib/datetime/LocalDateTime.protos`.

## Exact product changed-path set

The exact `0abd607a..d6981daa` one-commit delta contains 15 paths:

~~~text
CHANGELOG.md
pom.xml
protos/lib/datetime/Date.protos
protos/lib/datetime/Duration.protos
protos/lib/datetime/Instant.protos
protos/lib/datetime/LocalDateTime.protos
protos/lib/datetime/Offset.protos
protos/lib/datetime/OffsetDateTime.protos
protos/lib/datetime/Period.protos
protos/lib/datetime/Time.protos
protos/tests/library/datetime/conversions.protos
protos/tests/library/datetime/families.protos
protos/tests/library/datetime/instants.protos
protos/tools/test/RepositoryCorpusPlans.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java
~~~

The datetime Test Tool corpus grows from five to eight suite-native files.

## Published test evidence

`instants.protos` retains evidence for:

- Offset range boundaries and second-resolution offsets;
- Offset equality/hash/Map behavior/order;
- Instant zero/positive/negative values;
- exact values beyond 64-bit carrier range;
- Instant equality/hash/Map behavior/order;
- exact Instant + Duration / - Duration;
- exact `durationBetween`;
- frozen Offset/Instant construction;
- explicit absence of zone identity and ambient `now`.

`conversions.protos` retains evidence for:

- epoch anchors;
- one-nanosecond pre/post epoch behavior;
- positive and negative fixed offsets;
- negative Instant normalization;
- consistency with Period day arithmetic;
- LocalDateTime/Instant fixed-offset round trips;
- year zero and negative-year cases;
- structural equality versus same-instant equality;
- `withOffsetSameInstant`;
- second-resolution non-minute offsets;
- nanosecond precision;
- supported civil-year boundary errors;
- unbounded Instant behavior independent of civil representation.

`families.protos` retains evidence for:

- exact own-family recognition across all eight datetime families;
- canonical immediate parents;
- Duration/Instant family separation;
- distinct Map keys for Duration and Instant with the same numeric payload;
- wrong-family operation rejection;
- rejection of matching shape under the wrong parent;
- rejection of transitive-only ancestry;
- Date and compound-family shape rejection;
- exact parent/shape/state structural recognition.

## Explicitly not implemented

~~~text
ZoneId=NO
ZoneRules=NO
ZoneDatabase=NO
ZonedDateTime=NO
TZDB=NO
WallClock=NO
MonotonicClock=NO
InstantNow=NO
Parsing=NO
Formatting=NO
LocaleFormatting=NO
TimersOrScheduling=NO
TOMLIntegration=NO
SymbolicDatetimeArithmetic=NO
~~~

## Maintainer-reported validation

After publication, the maintainer reported:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No additional test result is inferred from source inspection.

## Specification / publication result

The product CHANGELOG records:

~~~text
SPECIFICATION_CHANGE=NO
~~~

The product version is `0.3.246-SNAPSHOT`.

## Next slice

The ratified sequence has one remaining authorized slice:

~~~text
NEXT_SLICE=LIB013-D
NEXT_SLICE_NAME=Explicit temporal text profiles
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~

LIB013-D owns deterministic explicit temporal parsing/formatting profiles and
bounded ISO/RFC 3339 interoperability.

Named timezone/tzdb support and clock/current-time capability remain separate
future decision gates and are not implicit follow-ons.

## Closure state

~~~text
LIB013_C_STATUS=COMPLETE
PRODUCT_REVISION=d6981daa330a2a3a767865e9cd88a6fb9f626e81
PRODUCT_VERSION=0.3.246-SNAPSHOT
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
SPECIFICATION_CHANGED=NO

OFFSET_IMPLEMENTED=YES
INSTANT_IMPLEMENTED=YES
OFFSETDATETIME_IMPLEMENTED=YES
FIXED_OFFSET_CONVERSION_IMPLEMENTED=YES
SAME_INSTANT_IMPLEMENTED=YES
INSTANT_DURATION_ARITHMETIC_IMPLEMENTED=YES
CANONICAL_DATETIME_FAMILY_RECOGNITION=YES
RECOGNITION_CAN_OBSERVE_FROZENNESS=NO

OFFSET_RANGE_SECONDS=-64800..64800
INSTANT_EPOCH=1970-01-01T00:00:00Z
INSTANT_PRECISION=NANOSECOND
OFFSETDATETIME_STRUCTURAL_EQUALITY=YES
OFFSETDATETIME_NATURAL_ORDER=NO

TZDB_ADDED=NO
CLOCK_AUTHORITY_ADDED=NO
PARSING_FORMATTING_ADDED=NO
NEW_UNAPPROVED_SEMANTICS=NO

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB013-D
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from
ChatGPT using the exact published C commit, the ratified LIB013-0 record,
published A/B evidence, current repository policy, and maintainer-reported
validation. No independent human review is claimed by this record.
