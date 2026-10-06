# LIB013-B — Temporal amounts and calendar arithmetic implementation evidence

Status: **COMPLETE**

Owning work item: `guillermomolina/protos#430`

Slice: `LIB013-B — Temporal amounts and calendar arithmetic`

Nature: implementation/validation evidence; **non-normative**

Product revision: `41a06e07d5d74053911f291ac27f911678c61ff5`

Product version: `0.3.243-SNAPSHOT`

Commit subject:

~~~text
LIB013-B: add temporal amounts and calendar arithmetic
~~~

Ratified design owner: LIB013-0 Candidate A — Pure Values First + Explicit Authorities.

## Publication result

LIB013-B is published in `guillermomolina/protos` at the exact product revision
above.

The published delta is exactly one commit ahead of LIB013-A revision
`b0bf563c30a612210cd67c6a0bbea01f44617c3c`.

The implementation adds pure temporal amounts and Gregorian calendar-relative
arithmetic without adding instant/offset, timezone/tzdb, clock, parsing,
formatting, locale, timer, scheduler, or TOML authority.

## Published Duration surface

Public module:

~~~text
std:datetime/Duration
~~~

Published constructor:

~~~text
Duration(nanoseconds)
~~~

The value has one public signed Integer slot:

~~~text
nanoseconds
~~~

It represents the exact total elapsed amount at nanosecond precision. It uses no
Float and inherits Protos Integer's exact unbounded arithmetic semantics.

Published module operations:

~~~text
Duration.compare(left, right)
Duration.add(left, right)
Duration.subtract(left, right)
Duration.negate(duration)
~~~

Published semantics include:

- fixed elapsed-time meaning only;
- positive, zero and negative durations;
- exact semantic equality;
- coherent hash / normal Map-key behavior;
- ordinary identity under `===`;
- natural total ordering;
- exact addition/subtraction/negation;
- no calendar, offset, zone or clock meaning;
- no application to Date, Time or LocalDateTime.

## Published Period surface

Public module:

~~~text
std:datetime/Period
~~~

Published constructor:

~~~text
Period(years, months, days)
~~~

Public signed Integer slots:

~~~text
years
months
days
~~~

The components are structural and are never normalized across units.

Examples of deliberately distinct values include:

~~~text
Period(1, 0, 0)
Period(0, 12, 0)
~~~

Published semantics include:

- structural semantic equality;
- coherent hash / normal Map-key behavior;
- ordinary identity under `===`;
- frozen values;
- no natural total order;
- deliberately no `Period.compare`.

Published negation:

~~~text
Period.negate(period)
~~~

Negation is component-wise and does not otherwise normalize the Period.

## Published calendar-application surface

`Period` owns calendar application.

Published module operations:

~~~text
Period.addToDate(period, date)
Period.subtractFromDate(period, date)
Period.addToLocalDateTime(period, dateTime)
Period.subtractFromLocalDateTime(period, dateTime)
~~~

Subtraction delegates to application of the component-wise negated Period.

No symbolic datetime `+` / `-` protocol was added.

## Calendar arithmetic contract retained

Period application implements the ratified order:

1. combine years and months into one month displacement;
2. apply that displacement;
3. clamp the original day once to the last valid day in the resulting month;
4. apply the Period's day displacement afterward.

The implementation therefore preserves cases such as:

~~~text
2025-01-31 + (0y, 1m, 0d) -> 2025-02-28
2024-01-31 + (0y, 1m, 0d) -> 2024-02-29
2024-02-29 + (1y, 0m, 0d) -> 2025-02-28
2024-03-31 + (0y,-1m, 0d) -> 2024-02-29
~~~

Years and months are not clamped sequentially, and repeated one-month
application may differ from one two-month application.

Day displacement may pass through intermediate years outside Date's public range
so long as the final constructed Date lies inside `-9999..9999`. Final
out-of-range results signal `Error`; they do not wrap or saturate.

LocalDateTime application changes only the Date component and retains the exact
existing Time object.

## Exact product changed-path set

The exact `b0bf563c..41a06e07` delta contains eight paths:

~~~text
CHANGELOG.md
pom.xml
protos/lib/datetime/Duration.protos
protos/lib/datetime/Period.protos
protos/tests/library/datetime/amounts.protos
protos/tests/library/datetime/arithmetic.protos
protos/tools/test/RepositoryCorpusPlans.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java
~~~

The datetime Test Tool corpus grows from 3 to 5 suite-native files.

## Published test evidence

`amounts.protos` retains evidence for:

- Duration positive/zero/negative construction;
- large exact Integer-backed amounts beyond a 64-bit carrier;
- exact 24-hour = 86,400-second Duration equivalence;
- Duration equality/hash/Map behavior;
- Duration total ordering;
- exact Duration add/subtract/negate;
- Period signed construction;
- Period structural equality/non-normalization;
- Period hash/Map behavior;
- Period component-wise negation;
- absence of Period natural ordering;
- frozen Duration/Period behavior.

`arithmetic.protos` retains evidence for:

- month/year end-of-month clamp;
- positive and negative calendar movement;
- crossing year boundaries and year zero;
- combined years+months before a single clamp;
- days applied after the clamp;
- repeated application differing from combined application;
- subtraction as negated application;
- supported Date range boundaries;
- very large exact Integer displacements;
- final-range failure;
- type/domain rejection;
- LocalDateTime Date-only adjustment with identical Time retention.

## Current transfer/runtime boundary

Duration and Period follow the same ordinary frozen-value implementation style as
LIB013-A.

No datetime-specific runtime value, transfer exception, global cache, shared
mutable authority or host datetime object was added.

The current general Actor transfer limitation for ordinary value graphs carrying
local Closures remains unchanged.

~~~text
SPECIAL_DATETIME_RUNTIME_VALUE=NO
DATETIME_TRANSFER_EXCEPTION=NO
~~~

## Explicitly not implemented

~~~text
DatePlusDuration=NO
SymbolicDatetimeArithmetic=NO
Offset=NO
Instant=NO
OffsetDateTime=NO
ZoneId=NO
ZoneDatabase=NO
ZonedDateTime=NO
WallClock=NO
MonotonicClock=NO
Parsing=NO
Formatting=NO
LocaleFormatting=NO
TimersOrScheduling=NO
TOMLIntegration=NO
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

The product version is `0.3.243-SNAPSHOT`.

New Protos-owned source/test files in this slice carry the project's current
Adaptive Public License notice.

## Next slice

The ratified implementation sequence continues with:

~~~text
NEXT_SLICE=LIB013-C
NEXT_SLICE_NAME=Instant and fixed-offset domain
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~

LIB013-C owns:

- `Offset`;
- `Instant`;
- `OffsetDateTime`;
- fixed-offset conversion;
- same-instant operations;
- structural OffsetDateTime equality.

Named zones/tzdb and current-time authority remain explicitly excluded.

LIB013-D remains the later text-profile slice.

## Closure state

~~~text
LIB013_B_STATUS=COMPLETE
PRODUCT_REVISION=41a06e07d5d74053911f291ac27f911678c61ff5
PRODUCT_VERSION=0.3.243-SNAPSHOT
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
SPECIFICATION_CHANGED=NO
NEW_UNAPPROVED_SEMANTICS=NO

DURATION_IMPLEMENTED=YES
PERIOD_IMPLEMENTED=YES
CALENDAR_ARITHMETIC_IMPLEMENTED=YES
PERIOD_STRUCTURAL_EQUALITY=YES
PERIOD_NATURAL_ORDER=NO
DATE_PLUS_DURATION=NO

TZDB_ADDED=NO
CLOCK_AUTHORITY_ADDED=NO
INSTANT_OFFSET_ADDED=NO
PARSING_FORMATTING_ADDED=NO

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB013-C
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from
ChatGPT using the exact published product commit, the ratified LIB013-0 record,
the published LIB013-A evidence, current repository policy, and
maintainer-reported validation evidence. No independent human review is claimed
by this record.
