# LIB013-D — Explicit temporal text profiles and LIB013 closure evidence

Status: **COMPLETE / PARENT CLOSURE READY**

Owning work item: `guillermomolina/protos#430`

Slice: `LIB013-D — Explicit temporal text profiles`

Nature: implementation/validation/closure evidence; **non-normative**

Product revision: `c851c6570b93503a9269024da34d71fdf4d0eb12`

Immediate product base: `7b9e629bddce9d501a2773a9f3a3973c8de37885`

Product version: `0.3.251-SNAPSHOT`

Commit subject:

~~~text
LIB013-D: add explicit temporal text profiles
~~~

Ratified design owner: LIB013-0 Candidate A — Pure Values First + Explicit Authorities.

## Publication result

LIB013-D is published in `guillermomolina/protos` at the exact product revision
above.

The D commit is exactly one commit ahead of
`7b9e629bddce9d501a2773a9f3a3973c8de37885`.

Other work landed between the historical LIB013-C publication and D, so the
closure record binds each LIB013 slice to its own exact published revision rather
than assuming the A/B/C/D commits are consecutive on main.

## Published ISO8601 profile

Public module:

~~~text
std:datetime/ISO8601
~~~

Published public operations:

~~~text
parseDate(text)
formatDate(date)
parseTime(text)
formatTime(time)
parseLocalDateTime(text)
formatLocalDateTime(dateTime)
parseOffsetDateTime(text)
formatOffsetDateTime(value)
parseInstant(text)
formatInstant(instant)
~~~

The module is explicitly a bounded canonical LIB013 profile inspired by ISO 8601
extended representations; it does not claim complete ISO 8601 acceptance.

### Year grammar

Canonical supported date text is:

- `YYYY` for years `0000..9999`;
- `-YYYY` for years `-0001..-9999`.

Positive expanded/signed years, `-0000`, and years outside Date's supported
range are rejected.

### Date / time grammar

Canonical date:

~~~text
YEAR-MM-DD
~~~

Canonical time:

~~~text
HH:MM:SS[.fraction]
~~~

Fractions accept one through nine decimal digits and are right-padded to
nanoseconds when parsed.

Ten or more digits signal `Error`; precision is never rounded.

Canonical formatting:

- omits the fraction for zero nanoseconds;
- starts from nine fractional digits otherwise;
- removes trailing zeros.

Hour 24 and leap-second label `:60` are rejected.

### LocalDateTime / OffsetDateTime

Local date-time uses exactly uppercase `T`.

Fixed-offset forms accepted by the bounded ISO profile are:

~~~text
Z
+HH:MM
-HH:MM
+HH:MM:SS
-HH:MM:SS
~~~

This preserves LIB013 Offset's one-second resolution.

The Offset value's existing inclusive ±18-hour range remains the semantic
authority.

Negative-zero spellings:

~~~text
-00:00
-00:00:00
~~~

are rejected rather than silently becoming ordinary zero Offset.

Canonical formatting uses:

- `Z` for zero Offset;
- `±HH:MM` when the Offset is minute-aligned;
- `±HH:MM:SS` when second precision is needed.

Stored local fields and offset are preserved; OffsetDateTime formatting does not
normalize structurally distinct values to UTC.

### Instant profile

`parseInstant` interprets any accepted bounded-ISO OffsetDateTime form through
the existing `OffsetDateTime.toInstant` conversion.

`formatInstant` uses the existing fixed-offset conversion at Offset(0) and
emits canonical UTC `Z` text.

Instant itself remains unbounded. Formatting signals `Error` when the resulting
UTC civil Date would lie outside the existing Date range `-9999..9999`; no
alternate epoch-number fallback is introduced.

## Published RFC3339 adapter

Public module:

~~~text
std:datetime/RFC3339
~~~

Published public operations:

~~~text
parseOffsetDateTime(text)
formatOffsetDateTime(value)
~~~

This is a bounded interoperability adapter for representable
`OffsetDateTime` values, not an alias of the ISO8601 module.

Accepted shape is deliberately restricted to:

~~~text
YYYY-MM-DDTHH:MM:SS[.fraction]OFFSET
~~~

with:

- four-digit non-negative years;
- uppercase `T`;
- uppercase `Z`;
- mandatory seconds;
- zero to nine fractional digits;
- numeric offsets at minute resolution: `±HH:MM`.

The adapter deliberately rejects RFC 3339 features that cannot be represented
losslessly by the initial LIB013 model:

- `-00:00`, whose RFC meaning is unknown local offset rather than known zero;
- leap-second `:60`;
- fractions longer than nine digits;
- second-resolution numeric offsets;
- negative/expanded years.

Formatting signals `Error` for:

- negative-year OffsetDateTime values;
- offsets whose exact seconds are not divisible by 60.

No rounding or silent semantic conversion is performed.

## Profile separation

ISO8601 and RFC3339 remain distinct public modules.

Published tests establish, among other cases:

- an ISO second-resolution offset such as `+00:00:01` is accepted by ISO8601
  and rejected by RFC3339;
- a negative-year value is text-representable by the bounded ISO8601 profile
  and rejected by RFC3339;
- `RFC3339 !== ISO8601`.

No universal temporal parser was added.

## Published tests

`iso8601.protos` retains evidence for:

- Date formatting/parsing including year zero, negative years and both year
  boundaries;
- strict rejection of unsupported basic/week/ordinal/reduced date forms;
- canonical nanosecond fractional formatting;
- rejection of hour 24, leap-second label `:60`, comma fractions and excess
  precision;
- uppercase-T LocalDateTime grammar;
- fixed offsets at minute and second resolution;
- zero-offset canonicalization to `Z`;
- negative-zero offset rejection;
- Offset ±18h boundaries and malformed offset rejection;
- exact OffsetDateTime structural round trips;
- Instant epoch, ±1ns, leap-day and Date-range boundary formatting;
- Instant formatting failure outside the supported civil Date range.

`rfc3339.protos` retains evidence for representative timestamps:

~~~text
1985-04-12T23:20:50.52Z
1996-12-19T16:39:57-08:00
1937-01-01T12:00:27.87+00:20
~~~

and for:

- RFC3339 parse/format round trips;
- canonical shortening of accepted trailing-zero fractions;
- `-00:00` rejection;
- leap-second rejection;
- excess-precision rejection;
- negative-year formatting rejection;
- non-minute Offset formatting rejection;
- explicit ISO/RFC profile distinction.

## Exact product changed-path set

The exact `7b9e629b..c851c657` one-commit delta contains eight paths:

~~~text
CHANGELOG.md
pom.xml
protos/lib/datetime/ISO8601.protos
protos/lib/datetime/RFC3339.protos
protos/tests/library/datetime/iso8601.protos
protos/tests/library/datetime/rfc3339.protos
protos/tools/test/RepositoryCorpusPlans.protos
src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java
~~~

The datetime suite-native corpus gains the two profile files.

## Explicitly excluded authority

LIB013-D adds no:

~~~text
ZoneId
ZoneRules
ZoneDatabase
ZonedDateTime
tzdb
wall clock
monotonic clock
current-time API
timer
scheduler
locale formatting/database
TOML semantic ownership
host-default timezone
symbolic datetime arithmetic
~~~

Those were deliberately separated by the ratified LIB013-0 Candidate A decision
and are not incomplete A-through-D work.

## Maintainer-reported validation

After publication, the maintainer reported:

~~~text
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No additional test result is inferred from source inspection.

## Specification / publication result

The product CHANGELOG records no specification change.

Published product version:

~~~text
0.3.251-SNAPSHOT
~~~

## LIB013 A-through-D closure

The ratified Candidate A implementation decomposition is fully published:

### LIB013-A — Pure civil temporal kernel

~~~text
PRODUCT_REVISION=b0bf563c30a612210cd67c6a0bbea01f44617c3c
~~~

Published:

- Date;
- Time;
- LocalDateTime.

### LIB013-B — Temporal amounts and calendar arithmetic

~~~text
PRODUCT_REVISION=41a06e07d5d74053911f291ac27f911678c61ff5
~~~

Published:

- Duration;
- Period;
- ratified calendar arithmetic.

### LIB013-C — Instant and fixed-offset domain

~~~text
PRODUCT_REVISION=d6981daa330a2a3a767865e9cd88a6fb9f626e81
~~~

Published:

- Offset;
- Instant;
- OffsetDateTime;
- fixed-offset conversion;
- same-instant operations;
- canonical datetime family recognition.

### LIB013-D — Explicit temporal text profiles

~~~text
PRODUCT_REVISION=c851c6570b93503a9269024da34d71fdf4d0eb12
~~~

Published:

- bounded canonical ISO8601 profile;
- bounded RFC3339 adapter.

All four authorized implementation slices are complete.

There are no LIB013 entries in the current implementation-blocker registry.

The parent Issue therefore has no remaining work inside the approved
LIB013-0 Candidate A A-through-D boundary.

Timezone/tzdb, clock/current-time, monotonic clock, scheduler/timer and locale
work remain deliberately outside this parent closure and require separate future
decision/work gates if pursued.

## Closure state

~~~text
LIB013_D_STATUS=COMPLETE
LIB013_PARENT_STATUS=CLOSURE_READY

PRODUCT_REVISION=c851c6570b93503a9269024da34d71fdf4d0eb12
PRODUCT_VERSION=0.3.251-SNAPSHOT
GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
SPECIFICATION_CHANGED=NO

ISO8601_PROFILE_IMPLEMENTED=YES
RFC3339_PROFILE_IMPLEMENTED=YES
ISO8601_FULL_STANDARD_CLAIMED=NO
RFC3339_MINUS_00_00_REJECTED=YES
LEAP_SECOND_TEXT_REJECTED=YES
EXCESS_NANOSECOND_PRECISION_REJECTED=YES
ISO_SECOND_RESOLUTION_OFFSET_SUPPORTED=YES
RFC3339_OFFSET_RESOLUTION=MINUTE

LOCALE_FORMATTING_ADDED=NO
TZDB_ADDED=NO
CLOCK_AUTHORITY_ADDED=NO
TOML_OWNERSHIP_CHANGED=NO
NEW_UNAPPROVED_SEMANTICS=NO

LIB013_A_THROUGH_D_COMPLETE=YES
REMAINING_AUTHORIZED_LIB013_SLICE=NO
PARENT_ISSUE_CLOSURE_CANDIDATE=YES
~~~

## AI-assistance disclosure

This durable evidence record was materially prepared with AI assistance from
ChatGPT using the exact published product commits, the ratified LIB013-0 record,
published A/B/C evidence, current repository policy, current blocker registry,
and maintainer-reported validation. No independent human review is claimed by
this record.
