# LIB015-D0 — Timestamp and explicit time-authority contract decision

Status: **RATIFIED**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-D0 — Timestamp and explicit clock-authority contract closure`

Nature: focused research and owner-approved Standard Library contract decision; **non-normative**

Previous durable records:

- `docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`
- `docs/project/work/LIB015/LIB015_C0_JSON_EVENT_FORMATTER_CONTRACT_DECISION.md`
- `docs/project/work/LIB015/LIB015_C1_JSON_FORMATTER_EVIDENCE.md`

Published C1 product revision:
`ddf59b1b3f5c97758354688088845d641f7a4e04`

Research reconciliation Protos revision:
`4da84e57c8a6716982412dd955583fcc34d4a0b7`

## Approval provenance

The D0 investigation reduced the final `LogEvent` representation choice to:

- **E1** — one canonical five-slot event family; `timestamp` always exists and
  is `null` or `std:datetime/Instant`;
- **E2** — separate four-slot and five-slot canonical shapes depending on whether
  a timestamp exists.

The project owner explicitly selected E1 in the active LIB015 interaction on
2026-10-07:

> si claramente acepto E1

E1 was the remaining structural trade-off presented at the approval handoff.
The rest of the focused D0 recommendation remains unchanged: explicit
caller-supplied time authority, no ambient clock, filtering before time
acquisition, canonical datetime reuse for human text, and exact Instant
nanoseconds for JSON.

This closes D0 and releases LIB015-D1 implementation. It does not close LIB015.

## HEAD reconciliation

D0 initially inspected the C1 product state at
`ddf59b1b3f5c97758354688088845d641f7a4e04`.

During the investigation `protos/main` advanced by one direct child commit:

~~~text
4da84e57c8a6716982412dd955583fcc34d4a0b7
PLAT051-A: add generic Standard Library semantic-value transfer mechanism
~~~

That commit does not change the logging or datetime surfaces governing D0.
`LogEvent`, `Logger`, `Instant`, and temporal text behavior were re-read at
the new HEAD.

~~~text
HEAD_RECONCILIATION=PASS
RESEARCH_DECISION_STILL_APPLIES=YES
~~~

## Ratified LogEvent contract

The canonical event family becomes exactly:

~~~text
level
message
fields
error
timestamp
~~~

Timestamp domain:

~~~text
timestamp === null
OR
std:datetime/Instant.recognizes(timestamp)
~~~

Absence is represented by the always-present `timestamp = null` slot. A
missing slot is not a second canonical event shape.

The existing call remains valid:

~~~text
LogEvent(level, message, fields, error)
    -> timestamp === null
~~~

D1 should preserve that API with the ordinary default-parameter mechanism,
conceptually:

~~~text
LogEvent(level, message, fields, error, timestamp = null)
~~~

Explicit historical/event time is accepted as the fifth argument when it is a
recognized `std:datetime/Instant`.

`LogEvent` never obtains current time itself.

`LogEvent.recognizes` migrates cleanly from the exact four-slot family to the
exact five-slot family. A previously materialized four-slot object is not kept
indefinitely as a second legacy canonical shape.

~~~text
LOGEVENT_CANONICAL_SHAPES=ONE
LOGEVENT_CANONICAL_SLOT_COUNT=5
LOGEVENT_TIMESTAMP_SLOT_ALWAYS_PRESENT=YES
LOGEVENT_TIMESTAMP_ABSENCE=NULL
LOGEVENT_FOUR_ARGUMENT_CALL_COMPATIBLE=YES
LOGEVENT_OBTAINS_CURRENT_TIME=NO
~~~

## Ratified Logger time authority

The existing Logger remains valid without temporal authority.

Conceptual construction becomes:

~~~text
Logger(minimumLevel, sink, timeSource = null)
~~~

When `timeSource === null`, enabled events use `timestamp = null` and no
time acquisition occurs.

When provided, `timeSource` is an explicit zero-argument callable capability.
For each enabled logging attempt it is invoked exactly once and must answer a
recognized `std:datetime/Instant`.

Conceptually:

~~~text
timestamp = timeSource()
require Instant.recognizes(timestamp)
~~~

No public `std:datetime/Clock` is required or introduced by D1. A future Clock
can be adapted to this capability without changing Logger.

`logger.with(fields)` preserves the same time source by identity and does not
invoke it during derivation.

~~~text
LOGGER_TIME_SOURCE_POLICY=OPTIONAL_EXPLICIT_ZERO_ARGUMENT_CALLABLE_RETURNING_INSTANT
LOGGER_WITH_PRESERVES_TIME_SOURCE=YES
STD_DATETIME_CLOCK_REQUIRED=NO
AMBIENT_CLOCK_ALLOWED=NO
~~~

## Ratified evaluation order

The pay-as-you-grow disabled path is preserved:

~~~text
1. validate level
2. apply minimum-level filtering

3. if disabled:
       return null
       do not inspect message/fields/error data
       do not merge/copy event data
       do not call timeSource
       do not construct LogEvent
       do not call sink

4. if enabled:
       extract call fields / attached Error
       validate call-fields Map boundary and top-level String keys
       merge base + call-site field references

5. acquire timestamp:
       null when no source
       otherwise call timeSource exactly once
       require std:datetime/Instant

6. construct LogEvent
       LogEvent remains owner of message/Error/deep-field validation
       and structured-data snapshotting

7. call sink.emit(event) under the existing Logger sink-Error containment

8. return null
~~~

D1 must not duplicate all LogEvent validation inside Logger merely to move
time acquisition after deep event validation.

~~~text
CLOCK_CALLED_FOR_DISABLED_LEVEL=NO
CLOCK_CALLS_PER_ENABLED_ATTEMPT=ZERO_OR_ONE
~~~

## Ratified time-source failure semantics

If an enabled Logger's time source:

- signals an Error;
- returns `null`; or
- returns a non-`Instant` value,

the logging call signals Error. No event is emitted.

This failure is not converted to `timestamp = null`, not retried, and not
classified as a sink failure. The existing containment boundary remains scoped
to `sink.emit(event)`.

~~~text
TIME_SOURCE_FAILURE_POLICY=PROPAGATE_ERROR
TIME_SOURCE_INVALID_RESULT_POLICY=ERROR
TIME_SOURCE_FAILURE_IS_SINK_FAILURE=NO
~~~

## Ratified plain-text timestamp contract

When `event.timestamp === null`, `TextFormatter` output remains byte-for-byte
identical to B1.

When timestamp is present, plain output is:

~~~text
std:datetime/ISO8601.formatInstant(event.timestamp)
+ " "
+ EXISTING_B1_LINE
~~~

Examples:

~~~text
1970-01-01T00:00:00Z INFO "started"
1970-01-01T00:00:00.000000001Z DEBUG "tick"
2026-10-07T03:15:22.123456789Z ERROR "request failed" {"requestId"="r-7"} error=true
~~~

Timestamp is unquoted and unbracketed. No logging-private temporal grammar or
local timezone lookup is introduced.

If a valid Instant lies outside the civil range supported by
`ISO8601.formatInstant`, plain formatting signals Error. There is no alternate
integer fallback in plain text.

~~~text
PLAIN_TIMESTAMP_POSITION=PREFIX
PLAIN_TIMESTAMP_FORMAT=STD_DATETIME_ISO8601_FORMATINSTANT
PLAIN_NO_TIMESTAMP_BYTES_UNCHANGED=YES
PLAIN_OUT_OF_RANGE_INSTANT_POLICY=FORMATTER_ERROR_NO_FALLBACK
~~~

## Ratified JSON timestamp contract

JSON preserves the complete unbounded Instant domain.

When timestamp is present, the first top-level member is:

~~~text
"timestamp": event.timestamp.nanoseconds
~~~

The value is the exact signed Protos Integer projected through existing
`std:json` exact-number semantics.

Top-level order becomes:

~~~text
timestamp   # only when present
level
message
fields
error       # only when present
~~~

When `event.timestamp === null`, the member is omitted entirely, preserving C1
JSON bytes.

Examples:

~~~json
{"level":"INFO","message":"started","fields":{}}
~~~

~~~json
{"timestamp":0,"level":"INFO","message":"started","fields":{}}
~~~

~~~json
{"timestamp":1,"level":"DEBUG","message":"tick","fields":{}}
~~~

A user field named `timestamp` remains non-colliding because user data stays
nested inside `fields`.

JSON does not use `ISO8601.formatInstant`, so a valid unbounded Instant never
fails merely because it lies outside the civil-date text range.

The deliberate representation split is:

~~~text
plain = canonical human UTC civil text
JSON  = exact timeline nanoseconds Integer
~~~

~~~text
JSON_TIMESTAMP_MEMBER=timestamp
JSON_TIMESTAMP_REPRESENTATION=EXACT_SIGNED_INTEGER_INSTANT_NANOSECONDS
JSON_TIMESTAMP_POSITION=FIRST_WHEN_PRESENT
JSON_TIMESTAMP_ABSENCE=OMIT_MEMBER
JSON_NO_TIMESTAMP_BYTES_UNCHANGED=YES
JSON_TIMESTAMP_DOMAIN=FULL_UNBOUNDED_INSTANT_DOMAIN
~~~

## Existing sink and Error boundaries remain unchanged

D1 grants no time authority to formatters or sinks.

- `MemorySink` retains the exact event including timestamp.
- `TextSink` remains a borrowed-writer sink with no clock.
- direct formatter/TextSink failures remain observable to direct callers.
- ordinary Logger sink failures retain the existing containment behavior.
- attached Core Error identity and presence-only formatting remain unchanged.

D0 does not add observed/emission time, monotonic elapsed-time measurement,
ambient/system clock access, async/buffering/exporters, or color/styling.

## Focused external evidence

Material precedents used only to test the unresolved timestamp boundary:

- Go `log/slog`: enablement is checked before current time is acquired and
  before the Record is constructed.
  <https://go.dev/src/log/slog/logger.go>
- Java `LogRecord`: event time belongs to the record; Java's ambient system
  clock choice itself is not adopted.
  <https://docs.oracle.com/en/java/javase/26/docs/api/java.logging/java/util/logging/LogRecord.html>
- Java `Clock`: explicit/fixed clocks demonstrate testable authority
  separation without requiring logging to own a Clock abstraction.
  <https://docs.oracle.com/en/java/javase/24/docs/api/java.base/java/time/Clock.html>
- Python `LogRecord`: record creation time and presentation are distinct.
  <https://docs.python.org/3/library/logging.html>
- Rust `tracing-subscriber`: injectable timers demonstrate testability, while
  formatter-owned time acquisition is deliberately not selected.
  <https://docs.rs/tracing-subscriber/latest/tracing_subscriber/fmt/time/trait.FormatTime.html>
- .NET `TimeProvider`: wall time and high-frequency timestamp facilities are
  distinct concepts.
  <https://learn.microsoft.com/dotnet/api/system.timeprovider>
- OpenTelemetry Logs distinguishes event `Timestamp` from
  `ObservedTimestamp` and uses epoch nanoseconds in structured telemetry.
  <https://opentelemetry.io/docs/specs/otel/logs/data-model/>
- Pino demonstrates separate epoch-number and ISO timestamp presentation
  policies.
  <https://github.com/pinojs/pino/blob/main/docs/api.md>

No external precedent overrides Protos' explicit-authority architecture.

## Candidate closure

### E1 — one five-slot event family — selected

Benefits:

- one canonical public event shape;
- `event.timestamp` always exists;
- recognition and consumers stay simple;
- no reflective presence test.

Cost:

- events without timestamps pay for one additional `null` slot.

The owner explicitly accepted E1.

### E2 — separate four-slot/five-slot shapes — rejected

Although E2 maximizes fixed-cost pay-as-you-grow behavior, it permanently creates
two public shapes and forces presence checks and more complex recognition.

### Formatter/sink time acquisition — rejected

It would turn event time into emission/formatting time.

### Ambient/global clock — rejected

It violates explicit authority and deterministic testability.

### Mandatory new `std:datetime/Clock` before D1 — rejected as unnecessary

The explicit zero-argument capability is sufficient.

## Compatibility matrix

| Surface | D1 contract |
|---|---|
| `LogEvent(level,message,fields,error)` | preserved |
| timestamp from four-argument call | `null` |
| explicit timestamp | fifth `Instant` argument |
| canonical LogEvent slot count | changes 4 -> 5 |
| legacy materialized four-slot object | not a second canonical family |
| `Logger(minimumLevel,sink)` | preserved |
| timestamp from two-argument Logger | `null` |
| time-aware Logger | optional third callable |
| disabled logging | no time-source call |
| `logger.with(fields)` | preserves source |
| `MemorySink()` | API unchanged |
| `TextSink(formatter,writer)` | API unchanged |
| plain without timestamp | byte-for-byte unchanged |
| JSON without timestamp | byte-for-byte unchanged |
| user `fields["timestamp"]` | valid and non-colliding |

## LIB015-D1 implementation release

D1 is authorized to implement exactly this ratified contract in
`guillermomolina/protos`.

Expected bounded implementation work:

1. extend canonical `LogEvent` to five slots with `timestamp = null | Instant`;
2. preserve four-argument construction via default fifth `null`;
3. migrate recognition and tests to the exact five-slot family;
4. extend Logger with optional zero-argument `timeSource`;
5. preserve two-argument Logger construction;
6. call the source only after filtering and exactly once per enabled attempt;
7. validate the result as `std:datetime/Instant`;
8. propagate source failures outside sink containment;
9. preserve the same source through `logger.with(fields)`;
10. prefix plain text with `ISO8601.formatInstant(timestamp)` when present;
11. keep no-timestamp plain bytes unchanged;
12. prepend optional JSON `timestamp` as exact signed nanoseconds Integer;
13. keep no-timestamp JSON bytes unchanged;
14. retain full Instant range in JSON and plain ISO range failure;
15. prove zero source calls for disabled levels;
16. cover source Error/null/non-Instant results;
17. cover explicit historical Instants, epoch, nanosecond precision, large and
    negative Instants, user `fields["timestamp"]`, and existing sink/Error
    semantics; and
18. do not add `std:datetime/Clock`, ambient time, observed time, monotonic
    time, async logging, color, or unrelated refactors.

No new Dxxx or PLATxxx is required.

## Maintainer-reported validation at D0 handoff

The project owner reported:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

D0 itself contains no product implementation; this validation is retained as
handoff evidence and is not treated as validation of the future D1 patch.

## Machine-readable closure

~~~text
LIB015_D0_STATUS=RATIFIED
LIB015_D0_RESEARCH_COMPLETE=YES
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_SELECTED_EVENT_SHAPE=E1
HEAD_RECONCILIATION=PASS

PROTOS_REVISION=4da84e57c8a6716982412dd955583fcc34d4a0b7
C1_PRODUCT_REVISION=ddf59b1b3f5c97758354688088845d641f7a4e04

TIMESTAMP_EVENT_OWNERSHIP=LOGEVENT
LOGEVENT_CANONICAL_SHAPES=ONE
LOGEVENT_CANONICAL_SLOT_COUNT=5
LOGEVENT_TIMESTAMP_SLOT_ALWAYS_PRESENT=YES
LOGEVENT_TIMESTAMP_ABSENCE=NULL
LOGEVENT_FOUR_ARGUMENT_CALL_COMPATIBLE=YES
LOGEVENT_OBTAINS_CURRENT_TIME=NO

LOGGER_TIME_SOURCE_POLICY=OPTIONAL_EXPLICIT_ZERO_ARGUMENT_CALLABLE_RETURNING_INSTANT
LOGGER_TWO_ARGUMENT_CALL_COMPATIBLE=YES
LOGGER_WITH_PRESERVES_TIME_SOURCE=YES
CLOCK_CALLED_FOR_DISABLED_LEVEL=NO
CLOCK_CALLS_PER_ENABLED_ATTEMPT=ZERO_OR_ONE
TIME_SOURCE_FAILURE_POLICY=PROPAGATE_ERROR
TIME_SOURCE_FAILURE_IS_SINK_FAILURE=NO
STD_DATETIME_CLOCK_REQUIRED=NO
AMBIENT_CLOCK_ALLOWED=NO

PLAIN_TIMESTAMP_FORMAT=STD_DATETIME_ISO8601_FORMATINSTANT_PREFIX
PLAIN_NO_TIMESTAMP_BYTES_UNCHANGED=YES
PLAIN_OUT_OF_RANGE_INSTANT_POLICY=FORMATTER_ERROR_NO_FALLBACK

JSON_TIMESTAMP_MEMBER=timestamp
JSON_TIMESTAMP_REPRESENTATION=EXACT_SIGNED_INTEGER_INSTANT_NANOSECONDS
JSON_TIMESTAMP_POSITION=FIRST_WHEN_PRESENT
JSON_TIMESTAMP_ABSENCE=OMIT_MEMBER
JSON_NO_TIMESTAMP_BYTES_UNCHANGED=YES
JSON_TIMESTAMP_DOMAIN=FULL_UNBOUNDED_INSTANT_DOMAIN

MEMORYSINK_PUBLIC_CONTRACT_CHANGE=NO
TEXTSINK_PUBLIC_CONTRACT_CHANGE=NO
OBSERVED_TIMESTAMP_ADDED=NO
MONOTONIC_CLOCK_SEPARATE_FUTURE_CONCEPT=YES

NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO

PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB015-D1
NEXT_SLICE_NAME=Timestamped LogEvent and explicit time-source integration
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~
