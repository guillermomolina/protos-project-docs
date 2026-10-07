# LIB015-D1 — Timestamped LogEvent and explicit Logger time-source evidence

Status: **COMPLETE**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-D1 — Timestamped LogEvent and explicit time-source integration`

Nature: implementation evidence; **non-normative**

Product revision:
`915fa3739e3a0a6e7f0934e3975e79d3bb24eba8`

Published implementation version:
`0.3.260-SNAPSHOT`

Ratified contract:
`docs/project/work/LIB015/LIB015_D0_TIMESTAMP_EXPLICIT_TIME_AUTHORITY_CONTRACT_DECISION.md`

## Outcome

LIB015-D1 publishes the owner-ratified timestamp model from D0.

Published commit:

~~~text
915fa3739e3a0a6e7f0934e3975e79d3bb24eba8
LIB015-D1: add timestamped LogEvent and explicit Logger time source
~~~

At evidence preparation time this commit is the current
`guillermomolina/protos` `main` revision.

The implementation version is:

~~~text
0.3.260-SNAPSHOT
~~~

## Changed paths

The exact product commit modifies:

- `CHANGELOG.md`;
- `pom.xml`;
- `protos/lib/logging/JsonFormatter.protos`;
- `protos/lib/logging/LogEvent.protos`;
- `protos/lib/logging/Logger.protos`;
- `protos/lib/logging/TextFormatter.protos`;
- `protos/tests/library/logging/events.protos`;
- `protos/tests/library/logging/json-formatter.protos`;
- `protos/tests/library/logging/logger.protos`;
- `protos/tests/library/logging/recognition.protos`;
- `protos/tests/library/logging/sinks.protos`;
- `protos/tests/library/logging/text-formatter.protos`; and
- `src/main/java/com/guillermomolina/protos/execution/ProtosLoggingFacility.java`.

No new source file is introduced.

## Canonical LogEvent timestamp state

`LogEvent` now has one canonical family of exactly five local slots:

~~~text
level
message
fields
error
timestamp
~~~

The `timestamp` slot is always present and is either:

~~~text
null
or
std:datetime/Instant
~~~

Four-argument construction remains source-compatible and produces
`timestamp === null`; a fifth argument may supply an explicit Instant.

The pre-D1 four-slot lookalike is no longer recognized as a canonical LogEvent.

The private safe-recognition facility was updated to validate the timestamp
through host representation without invoking candidate behavior.

## Explicit Logger time authority

`Logger` now accepts an optional third `timeSource` argument.

The two-argument form remains valid and produces events with no timestamp.

When a source is supplied:

- it is a zero-argument callable;
- disabled levels never invoke it;
- enabled logging invokes it exactly once after the call-field boundary is
  validated/merged and before LogEvent construction;
- the result must be a recognized `std:datetime/Instant`;
- a signalling source or an invalid result propagates Error to the caller;
- no event reaches the sink after a source failure;
- source failure is not contained as a sink failure; and
- `logger.with(fields)` preserves the same source without reading it merely
  because a derived Logger is created.

No ambient, host, JVM, process-global, Task-local, or implicit clock is added.
No `std:datetime/Clock` abstraction is introduced.

## Plain timestamp formatting

`TextFormatter` preserves prior bytes for events whose timestamp is `null`.

A timestamped event is prefixed with:

~~~text
std:datetime/ISO8601.formatInstant(event.timestamp) + " "
~~~

Examples covered by the published tests include epoch, one-nanosecond precision,
negative/pre-epoch values, and a nanosecond-precision 2026 timestamp.

A valid Instant outside the civil range supported by `ISO8601.formatInstant`
signals Error. There is no numeric fallback in the plain-text grammar.

## JSON timestamp formatting

`JsonFormatter` preserves prior C1 bytes when timestamp is `null`.

When present, timestamp is the first top-level member:

~~~text
"timestamp": <exact signed Instant.nanoseconds Integer>
~~~

It is projected through the existing exact `std:json` number path rather than
through ISO text.

This retains the full unbounded Instant domain and keeps user fields named
`timestamp` non-colliding under the existing nested `fields` object.

All prior C0/C1 policies for field nesting, attached Error presence, exact
Integers, Float handling, deterministic Map ordering, String escaping and
TextSink reuse remain intact.

## Sink boundaries

`MemorySink` retains timestamped LogEvents by identity, including the exact
Instant reference.

No timestamp authority was added to `MemorySink` or `TextSink`.

The existing Logger sink-Error containment remains scoped to `sink.emit(event)`.

## Test coverage added/updated

The published delta contains focused coverage for:

- four-argument LogEvent compatibility;
- explicit fifth-argument Instant;
- exact five-slot canonical shape;
- rejection of the old four-slot shape;
- timestamp recognition and fake/lookalike rejection;
- two-argument Logger compatibility;
- disabled-path zero time-source calls;
- exactly one source call per enabled attempt;
- field-boundary failure before source acquisition;
- derived Logger source preservation;
- source Error, null and non-Instant failure;
- distinction between source failure and sink failure;
- unchanged no-timestamp text output;
- ISO timestamp prefixes and nanosecond precision;
- out-of-range plain timestamp failure;
- unchanged no-timestamp JSON output;
- exact epoch, positive, negative and very large JSON nanoseconds;
- nested user field named `timestamp`; and
- MemorySink identity preservation for timestamped events.

## Maintainer-reported validation

After publication, the project owner reports:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No additional product tests are inferred beyond that report.

## Specification, decision and publication state

The product CHANGELOG records no specification change for D1.

D1 implements the exact owner-ratified D0 contract and does not reopen LIB015-0,
B0, C0, or D0.

~~~text
SPECIFICATION_CHANGED=NO
LIB015_D0_CONTRACT_REOPENED=NO
STD_DATETIME_CLOCK_ADDED=NO
AMBIENT_CLOCK_ADDED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
~~~

## Remaining LIB015 scope

D1 completes the logging core's currently authorized event, filtering, context,
plain formatting, explicit sink, JSON and timestamp sequence.

LIB015-0 also ratified a separate requirement for colored human logging output,
with these boundaries:

- ANSI must not be stored in LogEvent or JSON;
- styling must be owned by a reusable terminal/styled-text capability rather
  than a logging-private ANSI framework;
- policy includes explicit `AUTO`, `ALWAYS`, and `NEVER`;
- environment/TTY conventions are resolved by an authorized terminal/CLI layer,
  not by the pure logging core; and
- the colored logging adapter follows only after that reusable capability exists.

At this reconciliation point:

~~~text
REUSABLE_TERMINAL_STYLING_CAPABILITY_AVAILABLE=NO
REUSABLE_TERMINAL_STYLING_ISSUE_FOUND=NO
~~~

Therefore LIB015 must remain open, but there is no executable next LIB015
implementation slice yet. The next logging adapter is blocked on separately
owned reusable terminal-styling work; that dependency must not be implemented
inside `std:logging` as a private ANSI subsystem.

The already-deferred FilteringSink, FanOutSink, buffering, asynchronous
emission, exporters, OpenTelemetry integration, rotation and global
configuration are not implied as mandatory closure work by D1.

## Machine-readable closure

~~~text
LIB015_D1_STATUS=COMPLETE
PRODUCT_REVISION=915fa3739e3a0a6e7f0934e3975e79d3bb24eba8
IMPLEMENTATION_VERSION=0.3.260-SNAPSHOT

LOGEVENT_CANONICAL_SLOT_COUNT=5
LOGEVENT_TIMESTAMP_ALWAYS_PRESENT=YES
LOGEVENT_TIMESTAMP_DOMAIN=NULL_OR_STD_DATETIME_INSTANT
LOGEVENT_FOUR_ARGUMENT_COMPATIBLE=YES

LOGGER_TWO_ARGUMENT_COMPATIBLE=YES
LOGGER_EXPLICIT_TIME_SOURCE=YES
DISABLED_TIME_SOURCE_CALLS=0
ENABLED_TIME_SOURCE_CALLS=EXACTLY_ONE_WHEN_PRESENT
LOGGER_WITH_PRESERVES_TIME_SOURCE=YES
TIME_SOURCE_FAILURE_PROPAGATES=YES
TIME_SOURCE_FAILURE_IS_SINK_FAILURE=NO

PLAIN_TIMESTAMP_POLICY=STD_DATETIME_ISO8601_PREFIX
PLAIN_NO_TIMESTAMP_BYTES_UNCHANGED=YES
PLAIN_OUT_OF_RANGE_POLICY=ERROR_NO_FALLBACK

JSON_TIMESTAMP_POLICY=EXACT_SIGNED_INSTANT_NANOSECONDS_INTEGER
JSON_TIMESTAMP_FIRST_WHEN_PRESENT=YES
JSON_TIMESTAMP_OMITTED_WHEN_NULL=YES
JSON_NO_TIMESTAMP_BYTES_UNCHANGED=YES
JSON_FULL_UNBOUNDED_INSTANT_DOMAIN=YES

MEMORYSINK_CONTRACT_REOPENED=NO
TEXTSINK_CONTRACT_REOPENED=NO
STD_DATETIME_CLOCK_ADDED=NO
AMBIENT_CLOCK_ADDED=NO

PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

COLOR_REQUIREMENT_REMAINS=YES
REUSABLE_TERMINAL_STYLING_CAPABILITY_AVAILABLE=NO
REUSABLE_TERMINAL_STYLING_ISSUE_FOUND=NO

PARENT_ISSUE_CLOSED=NO
NEXT_LIB015_SLICE=BLOCKED
NEXT_LIB015_SLICE_NAME=Colored human logging adapter over reusable terminal styling
BLOCKER=SEPARATELY_OWNED_REUSABLE_TERMINAL_STYLING_CAPABILITY
NEXT_SLICE_PROMPT_READY=NO
~~~
