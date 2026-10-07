# LIB013-0 — Date/time foundations decision and ratification record

Status: **RATIFIED — Candidate A selected**

Owning work item: GitHub Issue `#430` — `LIB013 — Date, time, duration and calendar foundations`

Nature: exhaustive comparative Standard Library design record; **non-normative**

Explicit project-owner approval: **2026-10-06**

Approval provenance: issue comment `6017658955`

Protos evidence baseline at durable publication preparation: `e1904bb01b61c6ee0da5fd279da0f7feb143229e`

Project-record base before publication: `5a6a3ad80830ab216be882d5558d20ff6ea8df94`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Purpose

LIB013 defines the future Protos Standard Library foundation for dates, local
times, local date-times, elapsed durations, calendar-relative amounts, absolute
instants, fixed offsets and related formatting profiles.

The design goal is to provide useful temporal values without conflating them
with ambient host clock authority, host-default timezone state, mutable timezone
rule databases, scheduler authority or JVM-specific semantics.

LIB013 was deliberately separated from LIB010/TOML. TOML temporal records remain
format semantics owned by LIB010 and do not define the general Protos datetime
model.

This record preserves the completed LIB013-0 comparative investigation and the
project owner's explicit selection of Candidate A.

Observable Standard Library semantics remain authoritative only when published
through the normal Protos specification/library process in
`guillermomolina/protos`. This durable record is decision history and
implementation guidance, not a normative specification.

## Investigation constraints

LIB013-0 was research only.

~~~text
COMMANDS_EXECUTED=NO
TESTS_EXECUTED=NO
REPOSITORIES_MODIFIED_DURING_RESEARCH=NO
IMPLEMENTATION_PERFORMED_DURING_RESEARCH=NO
~~~

The investigation was required to preserve these constraints:

1. no ambient host clock in a pure value module;
2. no silent host-default timezone;
3. no hidden mutable tzdb singleton required for ordinary temporal arithmetic;
4. JVM `java.time` behavior must not accidentally become Protos semantics;
5. TOML temporal semantics remain independently owned by LIB010;
6. host clock or timezone-database access must cross an explicit audited
   authority/runtime boundary;
7. pure temporal values should remain transferable and deterministic whenever
   their semantics permit it;
8. scheduler, Task, Actor and Process timing semantics are outside this library
   unless a separately approved boundary is established;
9. host implementation convenience does not define observable Protos behavior.

## External systems compared

The investigation materially compared:

- Java `java.time` / JSR-310;
- Rust `time`;
- Rust `chrono`;
- Python `datetime`;
- Python `zoneinfo`;
- .NET `DateTime`, `DateTimeOffset`, and `TimeZoneInfo`;
- Noda Time;
- JavaScript Temporal;
- Swift Foundation date/time APIs and Swift clock APIs;
- Go `time`;
- C++20 `<chrono>`;
- Smalltalk/Pharo temporal models where useful as contrast;
- ISO 8601;
- RFC 3339; and
- IANA Time Zone Database concepts and update/versioning models.

Primary-source references included:

- Java Period:
  https://docs.oracle.com/en/java/javase/26/docs/api/java.base/java/time/Period.html
- Noda Time core types:
  https://nodatime.org/3.2.x/userguide/core-types
- JavaScript Temporal timezone/disambiguation:
  https://tc39.es/proposal-temporal/docs/timezone.html
- JavaScript Temporal Instant:
  https://tc39.es/proposal-temporal/docs/instant.html
- Python zoneinfo:
  https://docs.python.org/3/library/zoneinfo.html
- .NET date/time overview:
  https://learn.microsoft.com/en-us/dotnet/standard/datetime/
- .NET DateTimeOffset exact equality:
  https://learn.microsoft.com/en-us/dotnet/api/system.datetimeoffset.equalsexact
- Go time:
  https://pkg.go.dev/time
- Swift Foundation Date:
  https://developer.apple.com/documentation/foundation/date
- Swift ContinuousClock:
  https://developer.apple.com/documentation/swift/continuousclock
- C++ chrono calendar types:
  https://en.cppreference.com/w/cpp/chrono/year_month_day
- Rust time Date:
  https://docs.rs/time/latest/time/struct.Date.html
- Rust chrono DateTime / Months:
  https://docs.rs/chrono/latest/chrono/struct.DateTime.html
  https://docs.rs/chrono/latest/chrono/struct.Months.html
- ISO 8601-1:2019 catalogue entry:
  https://committee.iso.org/standard/70907.html
- RFC 3339:
  https://www.rfc-editor.org/info/rfc3339/
- IANA tzdb theory:
  https://data.iana.org/time-zones/tzdb-2025c/theory.html

## Comparative conclusions

### Separate civil values from timeline values

The strongest recurring architecture across mature systems is to distinguish
human/civil representations from positions on an absolute timeline.

The useful Protos distinction is:

~~~text
civil values:
    Date
    Time
    LocalDateTime

absolute timeline:
    Instant

fixed-offset bridge:
    Offset
    OffsetDateTime

external rule knowledge:
    future ZoneId / ZoneRules / ZoneDatabase
~~~

A single universal DateTime object was rejected because it hides materially
different invariants.

### Duration is not Period

The investigation found strong precedent for preserving two different concepts:

~~~text
Duration
    fixed elapsed amount
    independent of calendar
    suitable for Instant arithmetic

Period
    calendar-relative amount
    years / months / days
    applied in a calendar context
~~~

A 24-hour Duration is exactly 86,400 uniform seconds in the selected initial
model. A one-day Period means calendar movement and is not globally equivalent
to 24 elapsed hours once timezone rules are involved.

### Offset is not timezone identity

A fixed offset answers how far a local representation is displaced from the
reference timeline at one moment.

A named timezone such as `Europe/Madrid` identifies a rule domain whose
historical and future offsets depend on externally versioned political/civil
rules.

Therefore:

~~~text
OFFSET_IS_TIMEZONE_IDENTITY=NO
OFFSET_ALONE_DEFINES_FUTURE_ZONE_RULES=NO
~~~

### tzdb is external, versioned authority

IANA timezone data changes over time and future rules can change through
governmental decisions.

Python `zoneinfo` also demonstrates that a runtime may obtain timezone data from
different system/package/configuration sources and that serializing only a zone
identifier can produce different behavior under another tzdb version.

The pure datetime value layer therefore must not require ambient tzdb state.

### Wall clock and monotonic time are different authorities

Obtaining the current wall time is environmental authority.

Monotonic time is generally process/runtime-local and is useful for elapsed
measurement, deadlines and timeouts rather than civil date/time representation.

Go provides an instructive counterexample by allowing monotonic readings to live
inside `Time`, which affects equality/comparison behavior and disappears during
serialization. Protos deliberately avoids this coupling.

### Text profiles do not define temporal semantics

ISO 8601 and RFC 3339 are representation/profile standards, not the semantic
object model.

RFC 3339 is deliberately narrower than full ISO 8601 and has special semantics
such as `-00:00`, which means that the UTC instant is known while the local
offset is unknown.

The value model and parser/formatter profiles therefore remain separate layers.

## Candidate architectures

### Candidate A — Pure Values First + Explicit Authorities

Initial public values:

- `Date`
- `Time`
- `LocalDateTime`
- `Duration`
- `Period`
- `Offset`
- `Instant`
- `OffsetDateTime`

Initial exclusions:

- named timezone values;
- tzdb / zone-rule databases;
- `ZonedDateTime`;
- current-time clocks;
- monotonic clocks;
- scheduler/timer APIs;
- locale formatting.

Named-zone resolution and clocks are added only through later explicit authority
boundaries.

### Candidate B — Zoned First with explicit ZoneDatabase

Adds named zones, explicit zone database authority, disambiguation and zoned
values immediately.

This is semantically defensible but imposes a much larger initial contract:
tzdb distribution/versioning, serialization, aliases, gaps/overlaps, Native
Image resource behavior and update responsibility.

### Candidate C — Timeline-centred minimal core

Begins with `Instant`, `Duration`, `Offset` and `OffsetDateTime`, leaving
civil/calendar values secondary.

This is small and implementation-friendly but a poor fit for ordinary
calendar-domain values such as birthdays, local appointments and calendar
arithmetic.

### Candidate D — JVM/java.time façade

Exposes a model close to Java `java.time`.

This minimizes current JVM implementation work but risks leaking Java year
ranges, parsing behavior, default-zone choices, JDK tzdb behavior and exception
taxonomy into Protos semantics.

Java remains acceptable implementation machinery, not semantic authority.

## Candidate evaluation

The comparative result favored Candidate A.

| Criterion | A | B | C | D |
| --- | ---: | ---: | ---: | ---: |
| correctness | 5 | 5 | 4 | 3 |
| Protos alignment | 5 | 4 | 3 | 1 |
| future-option resilience | 5 | 4 | 3 | 1 |
| scalability | 5 | 5 | 4 | 4 |
| conceptual simplicity | 4 | 2 | 5 | 4 |
| portability / implementation freedom | 5 | 3 | 5 | 1 |
| runtime/resource cost | 5 | 2 | 5 | 5 |
| failure / operability | 5 | 3 | 4 | 2 |
| reversibility | 5 | 3 | 3 | 1 |
| evidence maturity | 5 | 5 | 4 | 5 |

Focused assessment:

| Candidate | Future resilience | Scalability | Protos philosophy |
| --- | ---: | ---: | ---: |
| **A** | **5** | **5** | **5** |
| B | 4 | 5 | 4 |
| C | 3 | 4 | 3 |
| D | 1 | 4 | 1 |

Candidate A was selected because it establishes the semantic distinctions that a
future timezone layer needs while deferring the externally authoritative and
version-sensitive machinery itself.

## Owner approval

The project owner explicitly selected Candidate A on 2026-10-06 in the active
LIB013 interaction with:

> aprobado A

The approval was mirrored to authoritative Issue #430 as comment
`6017658955`.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
LIB013_0_SELECTED_CANDIDATE=A_PURE_VALUES_FIRST_EXPLICIT_AUTHORITIES
~~~

No earlier owner-approved LIB013 invariant was found that conflicts with this
selection.

~~~text
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_PRIOR_INVARIANTS=NONE
~~~

## Ratified initial module architecture

The top-level name remains:

~~~text
std:datetime
~~~

The selected conceptual layering is:

~~~text
std:datetime
    pure values
    validation
    arithmetic
    pure conversions

std:datetime:format
    pure canonical text profiles
    explicit ISO/RFC adapters

future timezone layer
    ZoneId
    immutable/versioned ZoneDatabase
    explicit local-time resolution
    zoned values

runtime/capability domain outside pure std:datetime
    wall clock
    monotonic clock
    deadlines/timers/scheduling
~~~

Exact source-file/module decomposition remains an implementation detail unless it
changes the approved public module surface.

## Ratified initial public value families

Initial public families:

~~~text
Date
Time
LocalDateTime
Duration
Period
Offset
Instant
OffsetDateTime
~~~

Deliberately deferred:

~~~text
ZoneId
ZoneRules
ZoneDatabase
ZonedDateTime
Clock
MonotonicClock
scheduler/timer/sleep APIs
locale formatting/database
business calendars
holiday rules
~~~

## Calendar contract

Initial calendar:

~~~text
CALENDAR=PROLEPTIC_GREGORIAN
ASTRONOMICAL_YEAR_NUMBERING=YES
YEAR_ZERO=SUPPORTED
SUPPORTED_YEAR_MIN=-9999
SUPPORTED_YEAR_MAX=9999
~~~

No generic calendar abstraction is introduced initially.

Month/day combinations must be validated according to the proleptic Gregorian
calendar.

## Precision and representation contract

~~~text
TEMPORAL_PRECISION=NANOSECOND
PUBLIC_FLOATING_POINT_TIME=NO
IMPLEMENTATION_REPRESENTATION=UNSPECIFIED
~~~

An implementation may use field-based storage, epoch-day/nanos-of-day,
seconds+nanos, Java `java.time`, compact custom representations or other
machinery as long as observable Protos semantics remain unchanged.

Java range limits, exception types and parser behavior are not public Protos
contracts.

## Overflow and validation

~~~text
ARITHMETIC_OVERFLOW=EXPLICIT_FAILURE
OUT_OF_SUPPORTED_RANGE=EXPLICIT_FAILURE
SILENT_WRAP=NO
SILENT_SATURATION=NO
INVALID_CALENDAR_DATE=EXPLICIT_FAILURE
~~~

The exact ordinary Protos Error taxonomy should reuse existing Standard Library
validation/error conventions where those conventions already determine the
answer.

If implementation discovers a genuinely new public error-taxonomy choice, it
must stop at the normal substantive-decision gate rather than inventing one.

## Duration and Period contract

~~~text
DURATION=FIXED_ELAPSED_AMOUNT
PERIOD=CALENDAR_RELATIVE_AMOUNT
DURATION_AND_PERIOD_DISTINCT=YES
PERIOD_NATURAL_TOTAL_ORDER=NO
PERIOD_EQUALITY=STRUCTURAL
~~~

`Date + Duration` is not part of the selected model.

A future zoned layer must preserve the distinction between timeline Duration
arithmetic and local-calendar Period arithmetic.

## Month/year arithmetic

The selected policy is clamp-to-valid-date.

Examples:

~~~text
2025-01-31 + 1 month = 2025-02-28
2024-01-31 + 1 month = 2024-02-29
2024-02-29 + 1 year  = 2025-02-28
2024-03-31 - 1 month = 2024-02-29
~~~

For a composite Period containing years/months/days:

1. apply the combined year/month displacement;
2. clamp the day once to the last valid day of the resulting month;
3. then apply the day displacement.

Repeated addition is not required to equal multiplication of a Period.

For example:

~~~text
(date + 1 month) + 1 month
~~~

need not equal:

~~~text
date + 2 months
~~~

because the intermediate clamp is observable calendar arithmetic.

## Equality, ordering and hashing

Selected conceptual rules:

| Value | Equality | Natural total order |
| --- | --- | --- |
| `Date` | same calendar date | yes |
| `Time` | same local time including nanoseconds | yes |
| `LocalDateTime` | same local fields | yes |
| `Duration` | same elapsed amount | yes |
| `Period` | same structural components | no |
| `Offset` | same fixed offset | yes |
| `Instant` | same timeline position | yes |
| `OffsetDateTime` | same local fields and same offset | no default total order |

For `OffsetDateTime`, structural equality is distinct from same-instant
comparison.

Two differently offset values may identify the same `Instant` without being the
same `OffsetDateTime`.

Implementations must keep equality/hash laws coherent with these distinctions.

## Offset contract

~~~text
OFFSET_IS_ZONE_ID=NO
OFFSET_RESOLUTION=SECOND
OFFSET_MIN=-18:00:00
OFFSET_MAX=+18:00:00
~~~

A fixed offset contains no future or historical timezone rules.

## Instant and leap-second policy

The initial `Instant` model is a Unix-like uniform-second timeline.

~~~text
INSTANT_EPOCH=1970-01-01T00:00:00Z_CONCEPTUAL_REFERENCE
INSTANT_SUBSECOND_PRECISION=NANOSECOND
LEAP_SECONDS_MODELLED=NO
TAI_MODELLED=NO
GPS_TIME_MODELLED=NO
PHYSICALLY_EXACT_UTC_CLAIM=NO
~~~

A parser must not silently normalize a leap-second-labelled timestamp into a
different supported time.

## Timezone/tzdb boundary

Named timezone support is not part of the initial implementation.

A future timezone design must preserve this boundary:

~~~text
PURE_VALUES_REQUIRE_TZDB=NO
AMBIENT_HOST_TZDB=NO
HIDDEN_MUTABLE_TZDB_SINGLETON=NO
ZONE_DATABASE_EXPLICIT=YES
ZONE_DATABASE_IMMUTABLE=YES
ZONE_DATABASE_VERSIONED=YES
~~~

A future zone database may be bundled, runtime-provided, application-provided,
OS-derived or otherwise sourced only after an explicit design chooses that
distribution/provenance model.

Updating timezone rules must not mutate the semantics of already-held pure
values through hidden global state.

## Future DST gap/overlap boundary

When named-zone support is eventually designed, nonexistent and ambiguous local
times must be resolved deliberately.

The approved future boundary reserves at least:

~~~text
reject
earlier
later
~~~

with rejection as the safe default unless a later explicit decision changes
that contract.

No initial LIB013 implementation may silently install host/JVM default
disambiguation behavior.

## Clock/current-time authority

Current wall time is environmental authority and is not a pure
`std:datetime` operation.

~~~text
DATE_NOW_STATIC_AMBIENT_API=NO
LOCALDATETIME_NOW_STATIC_AMBIENT_API=NO
INSTANT_NOW_STATIC_AMBIENT_API=NO
WALL_CLOCK_AUTHORITY=FUTURE_EXPLICIT_RUNTIME_CAPABILITY
~~~

The future clock API requires a separate authority/runtime decision.

## Monotonic time boundary

Monotonic time does not belong to the civil value model.

~~~text
MONOTONIC_TIME_IN_std:datetime=NO
MONOTONIC_INSTANT_SERIALIZABLE_AS_CIVIL_TIME=NO
MONOTONIC_TIME_DISTRIBUTED_TIMESTAMP=NO
MONOTONIC_CLOCK_OWNER=FUTURE_RUNTIME_CAPABILITY
~~~

Timeout, deadline, timer, sleep and scheduler semantics remain separate work.

## Parsing and formatting boundary

Parsing/formatting is separate from the value model.

The initial formatting layer may provide explicit canonical profiles for:

- Date;
- Time;
- LocalDateTime;
- OffsetDateTime;
- Instant; and
- bounded RFC 3339 interoperability.

A generic host parser must not silently define the accepted Protos grammar.

~~~text
VALUE_MODEL_OWNS_TEXT_GRAMMAR=NO
FORMAT_PROFILE_EXPLICIT=YES
LOCALE_FORMATTING_INITIAL=NO
~~~

## ISO 8601 / RFC 3339 boundary

ISO 8601 and RFC 3339 are not treated as equivalent names for one parser.

RFC 3339 `-00:00` represents unknown local offset rather than ordinary UTC zero
offset.

Because initial `OffsetDateTime` cannot represent that distinction without
information loss:

~~~text
RFC3339_MINUS_00_00_TO_OFFSETDATETIME=SILENT_CONVERSION_FORBIDDEN
~~~

The initial adapter should reject that direct conversion rather than change its
meaning.

Leap-second syntax is likewise not silently normalized.

## TOML interoperability

LIB010 remains the owner of TOML temporal format semantics.

Selected architecture:

~~~text
LIB010_TOML_DEPENDS_ON_LIB013_SEMANTICS=NO_REQUIRED_OWNERSHIP
LIB013_KNOWS_ABOUT_TOML=NO
TOML_DATETIME_INTEROP=ADAPTER_BOUNDARY
~~~

Future adapters may convert TOML local-date, local-time, local-date-time and
offset-date-time records into LIB013 values when representable.

Conversions must fail rather than silently change:

- precision;
- leap-second behavior;
- offset semantics;
- range; or
- invalid calendar fields.

TOML must remain independently implementable.

## Transferability and concurrency

The initial temporal values are intended to be immutable pure values suitable
for ordinary transfer/copy semantics across Tasks, Actors, Processes, isolated
execution and future distributed boundaries wherever the existing Protos value
model permits.

No shared mutable temporal global state is required for ordinary arithmetic.

Environmental authorities such as future clocks and zone databases are not
silently treated as ordinary pure transferable values.

## JVM and Native Image portability

Java `java.time` may be used internally where useful.

It must not expose as Protos semantics:

- Java's supported year range;
- Java exception classes;
- Java system-default timezone;
- JDK `ZoneRulesProvider` choice;
- JDK tzdb version;
- `Clock.systemDefaultZone()`;
- Java parser permissiveness; or
- Java-specific time-scale implementation details.

The initial pure-value design intentionally avoids requiring tzdb resources or
host timezone discovery, which preserves a small Native Image footprint and
future alternate-runtime freedom.

## Approved implementation decomposition

The owner-approved post-investigation implementation sequence is deliberately a
small number of substantial slices.

### LIB013-A — Pure civil temporal kernel

Objective:

- implement `Date`, `Time`, `LocalDateTime`;
- implement validation;
- establish approved equality/order/hash behavior;
- preserve pure immutable/transfer-safe value semantics.

Principal dependencies:

- current Protos Standard Library value conventions;
- current object/value equality/hash/order conventions;
- current error/validation conventions;
- current Standard Library module/documentation conventions.

Required evidence includes:

- minimum/maximum supported years;
- year zero and negative years;
- Gregorian leap-year rules;
- invalid month/day combinations;
- nanosecond precision boundaries;
- equality/hash/order laws;
- transfer/copy behavior appropriate to current Protos value conventions.

Out of scope:

- Duration;
- Period;
- Instant;
- Offset;
- OffsetDateTime;
- text parsing/formatting;
- named zones/tzdb;
- current-time clocks;
- monotonic clocks;
- timers/scheduling.

### LIB013-B — Temporal amounts and calendar arithmetic

Objective:

- implement `Duration` and `Period`;
- implement signed arithmetic, overflow and clamp semantics.

Required evidence includes January 31, February 29, subtraction, negative
periods, repeated-addition-versus-multiplied-period and range-overflow cases.

Named timezone/DST behavior remains out of scope.

### LIB013-C — Instant and fixed-offset domain

Objective:

- implement `Offset`, `Instant`, `OffsetDateTime`;
- implement fixed-offset conversion and same-instant operations;
- preserve structural `OffsetDateTime` equality.

Out of scope:

- named zones;
- tzdb;
- current-time access.

### LIB013-D — Explicit temporal text profiles

Objective:

- provide deterministic canonical parsing/formatting profiles;
- provide bounded ISO/RFC 3339 adapters without making the parser the temporal
  semantic model.

Required negative evidence includes leap-second labels, RFC 3339 `-00:00`,
unsupported forms, excess precision, malformed offsets and out-of-range years.

Locale formatting remains out of scope.

## Explicitly separate future gates

The following are not implicitly authorized as later implementation slices merely
because Candidate A was approved:

- named timezone values;
- tzdb distribution/provider architecture;
- timezone database update policy;
- timezone serialization/version pinning;
- DST gap/overlap default policy beyond the reserved explicit boundary;
- host wall-clock capability;
- monotonic clock capability;
- timer/deadline/sleep/scheduler APIs;
- locale formatting/database;
- business/holiday calendars.

These require separately surfaced decisions when actual work reaches them.

## Implementation gate

LIB013-0 is complete and ratified.

Implementation may proceed with LIB013-A against the current repository HEAD.

Implementation must stop rather than invent semantics if current repository
evidence exposes a public choice not fixed by this record or already-fixed Protos
semantics.

~~~text
LIB013_0_STATUS=RATIFIED
RECOMMENDED_CANDIDATE=A_PURE_VALUES_FIRST_EXPLICIT_AUTHORITIES
OWNER_DECISION_REQUIRED=NO_FOR_CANDIDATE_A
IMPLEMENTATION_AUTHORIZED=YES_FOR_LIB013_A_THROUGH_D_WITHIN_RATIFIED_BOUNDARY

NEXT_SLICE=LIB013-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos

TIMEZONE_TZDB_IMPLEMENTATION_AUTHORIZED=NO
CLOCK_IMPLEMENTATION_AUTHORIZED=NO
MONOTONIC_CLOCK_IMPLEMENTATION_AUTHORIZED=NO

SPECIFICATION_CHANGED_BY_RATIFICATION=NO
STANDARD_LIBRARY_IMPLEMENTATION_CHANGED_BY_RATIFICATION=NO
TESTS_EXECUTED_FOR_RATIFICATION=NO
~~~

## AI-assistance disclosure

This durable research/ratification record was materially prepared with AI
assistance from ChatGPT from the approved LIB013-0 research packet, the cited
public upstream evidence, current project policy and the project owner's explicit
Candidate A approval. No independent human review is claimed by this record.
