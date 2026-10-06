# LIB015-B0 — Plain formatting and TextWriter sink contract decision

Status: **RATIFIED**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-B0 — Plain formatting and TextWriter sink contract closure`

Nature: focused research and owner-approved Standard Library contract decision; **non-normative**

LIB015-0 ratification record:
`docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`

LIB015-A implementation evidence:
`docs/project/work/LIB015/LIB015_A_STRUCTURED_LOGGING_CORE_EVIDENCE.md`

Research product baseline:
`0432e4446cebdc8778be4435d274502c0f482924`

Reconciled Protos HEAD at approval/publication preparation:
`c851c6570b93503a9269024da34d71fdf4d0eb12`

## Purpose

LIB015-0 approved immutable structured `LogEvent`, explicit derived `Logger`,
explicit `Sink`, TextWriter reuse, synchronous baseline logging, plain human
formatting, future styled-terminal formatting and JSON as separate layers.

LIB015-A then published the event/level/Logger core, but left one observable
boundary unresolved before LIB015-B implementation:

- the exact byte-for-byte plain human representation;
- ownership of the line terminator;
- multiline/control-character escaping;
- deterministic structured-field ordering;
- representation of an attached Core Error without arbitrary behavior;
- the composition of synchronous `sink.emit(event)` with TextWriter
  Future-returning writes;
- writer lifecycle ownership;
- the minimum MemorySink API; and
- safe public recognition of genuine logging events.

LIB015-B0 closes only those questions. It does not implement them.

## HEAD reconciliation

The research inspected current public Protos state after LIB015-A, including:

- `protos/lib/logging/Level.protos`;
- `protos/lib/logging/LogEvent.protos`;
- `protos/lib/logging/Logger.protos`;
- `protos/tests/library/logging/events.protos`;
- `protos/tests/library/logging/logger.protos`;
- `spec/io/IO_CORE.md`;
- `spec/io/TEXT_IO.md`;
- `spec/io/PROCESS_IO.md`;
- `protos/lib/io/ProcessStreams.protos`;
- `spec/semantics/OBJECT_MODEL.md`;
- `spec/semantics/VALUES_AND_COLLECTIONS.md`; and
- recent standard-family recognition precedents such as
  `std:datetime/Instant`.

During approval preparation, `guillermomolina/protos` advanced from research
baseline `0432e444...` to `c851c657...`.

The intervening delta is BUG019 plus LIB013-D. It changes distribution/tooling,
datetime text profiles, version/changelog material and related tests. It does not
modify the logging modules, TextWriter/Text I/O contracts, Error domain, object
reflection, or collection/value contracts on which B0 depends.

Therefore:

~~~text
HEAD_RECONCILIATION=PASS
RESEARCH_DECISION_STILL_APPLIES=YES
~~~

## Focused external evidence

The decision packet compared only the formatter/writer/sink boundary rather than
repeating LIB015-0's architecture survey.

Material precedents:

- Go `log/slog` TextHandler: deterministic human text, explicit attribute
  encoding and one handler write boundary.
- Python `logging`: Formatter produces text while StreamHandler owns the
  terminator/output operation.
- Log4j2: Layout is separate from Appender; flush and async behavior belong to
  output policy rather than event semantics.
- Rust `tracing-subscriber fmt`: event formatting and writer are separable,
  with distinct compact/full/pretty/JSON presentation families.
- Swift Logging StreamLogHandler: handler formatting orders metadata
  deterministically before writing to the supplied stream.

External libraries that stringify arbitrary values were not copied into Protos:
LIB015-A deliberately restricts structured field values and forbids arbitrary
object stringification.

## Selected formatter/sink model

The selected baseline is:

~~~text
TextFormatter.format(event)
    -> one deterministic String
    -> no trailing newline

TextSink(formatter, writer).emit(event)
    -> formatter.format(event)
    -> writer.writeLine(text).value()
    -> null
~~~

The formatter remains independent from output authority. The sink owns line
framing because it owns the TextWriter operation.

~~~text
PLAIN_FORMATTER_LINE_ORIENTED=YES
NEWLINE_OWNER=TEXT_SINK
~~~

## Exact plain human grammar

The baseline output grammar is:

~~~text
LINE =
    LEVEL SP MESSAGE
    [SP FIELDS]
    [SP "error=true"]

MESSAGE = STRING
FIELDS = MAP

VALUE =
    null
  | true
  | false
  | INTEGER
  | FLOAT
  | STRING
  | ARRAY
  | MAP

ARRAY =
    "["
    [VALUE (", " VALUE)*]
    "]"

MAP =
    "{"
    [STRING "=" VALUE (", " STRING "=" VALUE)*]
    "}"
~~~

The formatter returns `LINE` with no trailing whitespace and no line
terminator.

Examples:

~~~text
INFO "hello"
INFO "cache refreshed" {"entries"=42}
WARN "a\nb"
ERROR "request failed" {"requestId"="r-7"} error=true
~~~

Empty event fields are omitted rather than rendered as `{}`.

## Structured-value rendering

The already-approved structured-value domain remains exactly:

~~~text
null
Boolean
String
Integer
Float
Array
Map<String, structured-value>
~~~

Rendering is recursive and never invokes arbitrary value behavior.

Selected forms:

~~~text
null            -> null
true / false    -> true / false
Integer         -> canonical base-10 decimal
Float           -> deterministic canonical Float text
String          -> double-quoted escaped text
Array           -> [value, value]
Map             -> {"key"=value, "key"=value}
~~~

Integer text has no leading plus and no unnecessary leading zeroes.

Float rendering must preserve Float identity distinctions that are semantically
observable, including signed zero. The public formatting contract therefore
includes canonical forms for edge cases:

~~~text
NaN
Infinity
-Infinity
0.0
-0.0
~~~

Finite non-zero Float output is deterministic shortest-round-trip binary64 text.
When finite output would otherwise look like an Integer and has no exponent,
`.0` is retained/added so Float and Integer textual forms remain distinguishable.

This is formatter-owned numeric rendering, not guest `toString()` dispatch.

## String escaping and multiline policy

Every String is double quoted.

At minimum the following escapes are exact:

~~~text
"     -> \"
\     -> \\
LF    -> \n
CR    -> \r
TAB   -> \t
~~~

Other terminal/control code points that must not appear literally in the
single-line baseline are emitted with deterministic Unicode escapes.

Ordinary Unicode scalar values are preserved exactly. No Unicode normalization,
locale collation or case folding is introduced.

A message containing a literal newline therefore remains one physical log line:

~~~text
message semantic value: a LF b
plain output:            WARN "a\nb"
~~~

This preserves one-event-per-line behavior, grep/file framing and protection
against log injection.

~~~text
MULTILINE_POLICY=ESCAPE_TO_SINGLE_PHYSICAL_LINE
~~~

## Deterministic Map ordering

`LogEvent.fields` does not acquire semantic ordering.

The formatter imposes deterministic ordering for every rendered Map, including
nested Maps:

~~~text
ASCENDING_LEXICOGRAPHIC_UNICODE_SCALAR_SEQUENCE
~~~

The first differing Unicode scalar determines order; when one key is an exact
prefix of another, the shorter key comes first.

This deliberately matches the canonical ordering rule already used by Core
reflection for `slotNames()`.

Insertion order, Map iteration order, locale and host String comparison are not
part of plain output.

## Attached Error rendering

An attached Core Error is not stringified, inspected for host/JVM class names, or
asked for a message/stack trace.

Baseline human output records presence only:

~~~text
error=true
~~~

The marker is outside the structured fields Map, so a user field named
`"error"` remains unambiguous:

~~~text
ERROR "failed" {"error"="domain-value"} error=true
~~~

Generic Error and Error subtypes use the same baseline presence marker.

~~~text
ERROR_RENDERING_POLICY=PRESENCE_MARKER_ERROR_TRUE_ONLY
~~~

A future richer diagnostic Error renderer may be added explicitly without
changing Core Error semantics.

## LogEvent recognition finding

LIB015-A currently creates a frozen ordinary object with local slots:

~~~text
level
message
fields
error
~~~

but the resulting object uses ordinary `Object` as its immediate parent and
`std:logging/LogEvent` publishes no recognition operation.

That is insufficient for public boundaries such as:

~~~text
TextFormatter.format(candidate)
MemorySink.emit(candidate)
~~~

A four-slot lookalike must not be silently accepted as a validated/snapshotted
logging event.

LIB015-B1 therefore starts with a bounded LIB015-A hardening:

1. successful events use the canonical `std:logging/LogEvent` module as their
   immediate parent;
2. `LogEvent.recognizes(value)` is published;
3. recognition uses safe runtime-backed state observation and invokes no
   candidate callbacks/lookup/equality/hash/conversion;
4. recognition validates the exact canonical local shape and valid event state;
5. no hidden brand or provenance token is introduced; and
6. Error-domain validation is hardened so public recognition/formatting need not
   execute arbitrary behavior on an attached Error candidate.

This follows recent Protos transparent structural-recognition precedent rather
than adding a new type system or hidden runtime identity class.

~~~text
LOGEVENT_RECOGNITION_SUFFICIENT=NO
LIB015_A_FOLLOWUP_REQUIRED=YES
LIB015_B_CAN_IMPLEMENT_WITH_CURRENT_LOGEVENT=NO
HIDDEN_BRAND_REQUIRED=NO
~~~

The follow-up is part of LIB015-B1 rather than a separate micro-slice.

## TextWriter Future boundary

Standard `TextWriter.writeText` / `writeLine` operations return Future.

LIB015-A, however, promises that ordinary `Logger` calls contain an ordinary
Error signalled synchronously by `sink.emit(event)`.

Therefore the TextWriter-backed sink must observe the write Future before
`emit` returns:

~~~text
writer.writeLine(formatted).value()
~~~

If the write Future fails, observation re-signals that Error inside the existing
Logger `Error.handle` boundary.

Fire-and-forget is rejected because a later writer failure would occur after the
Logger containment boundary had already returned.

Changing Logger to return a Future is also rejected because it would reopen the
published LIB015-A public contract without need.

~~~text
TEXTWRITER_SINK_AWAITS_WRITE_FUTURE=YES
~~~

## Suspension and backpressure

Waiting on the write Future may suspend the calling Task when the Future is
pending:

~~~text
CAN_BASIC_TEXT_LOGGING_SUSPEND_CALLING_TASK=YES
~~~

This is compatible with the previously ratified invariant:

~~~text
BASIC_LOGGING_REQUIRES_BACKGROUND_TASK=NO
~~~

No logging Task/thread/queue is created. The caller simply participates in
ordinary Future suspension/backpressure.

The strongest cost is latency from slow sinks. A future explicit `AsyncSink`
or buffered adapter remains the escape path and must define its own queue,
capacity, drop/backpressure, failure and shutdown policy.

## Writer ownership and lifecycle

A TextWriter-backed logging sink borrows its writer.

It does not discover stdout/stderr, does not own a writer supplied by the caller,
does not flush after each event and does not close the writer.

~~~text
TEXTWRITER_SINK_WRITER_OWNERSHIP=BORROWED
TEXTWRITER_SINK_FLUSHES_EACH_EVENT=NO
TEXTWRITER_SINK_CLOSES_WRITER=NO
~~~

This composes directly with `std:io/ProcessStreams.stdoutWriter(process)` and
`stderrWriter(process)`, which already produce borrowing TextWriter wrappers.

The caller that supplied the writer remains responsible for explicit lifecycle
operations when the writer exposes those capabilities.

## MemorySink

The minimum public MemorySink contract is:

~~~text
sink = MemorySink()
sink.emit(event) -> null
sink.events() -> fresh frozen Array snapshot
~~~

Rules:

- `emit` requires a recognized LogEvent;
- the exact event identity is retained, not copied or formatted;
- emission order is preserved;
- `events()` returns a fresh frozen snapshot Array;
- snapshots already returned do not change after later emissions; and
- no `clear`, capacity, ring buffer, queue, metric, flush or close API is added
  in the baseline.

MemorySink is useful for tests but remains an ordinary public sink.

## Optional sink composition

Both additional compositions are deferred:

~~~text
FILTERING_SINK=DEFER
FANOUT_SINK=DEFER
~~~

Logger already performs early severity filtering.

FilteringSink may later support destination-specific routing, but its predicate
contract is unnecessary for B1.

FanOutSink is specifically deferred because partial failure would require an
explicit ordering/failure policy across destinations.

## Styled text, JSON and timestamp compatibility

B0 does not implement terminal color, JSON or timestamps.

The plain formatter is a sibling formatter, not a universal human-rendering
framework. Future StyledText formatting can reuse internal semantic helpers
without making B's plain String representation an intermediate public ABI.

JSON remains an independent projection through `std:json`.

Timestamp integration remains LIB015-D and uses `std:datetime/Instant`; B0
introduces no placeholder timestamp text.

## Candidate scorecard

Scale: 1 = weakest, 5 = strongest.

| Criterion | A: unterminated formatter + writeLine/value | B: terminated formatter + writeText/value |
| --- | ---: | ---: |
| Correctness / invariant preservation | 5 | 5 |
| Protos alignment | 5 | 4 |
| Future-option resilience | 5 | 3 |
| Scalability | 4 | 4 |
| Conceptual simplicity | 5 | 4 |
| Portability / implementation freedom | 5 | 5 |
| Runtime / resource cost | 4 | 4 |
| Failure / operability | 5 | 5 |
| Reversibility / migration cost | 5 | 4 |
| Evidence maturity / implementation risk | 5 | 4 |

The asynchronous writer candidate is hard-disqualified because it either loses
the published Logger failure-containment boundary or requires reopening Logger.

Focused winner assessment:

~~~text
AGUANTE_DE_FUTURO=5
ESCALABILIDAD=4
FILOSOFIA_PROTOS=5
~~~

The winner is selected by responsibility boundaries, not merely by numeric sum.

## Owner approval

The project owner explicitly approved this B0 contract in the active interaction
on 2026-10-06 with:

> aprobado

This approval applies to the complete recommended B0 contract, including the
LogEvent-recognition hardening required at the start of B1.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
OWNER_APPROVAL=YES
~~~

## Maintainer-reported validation context

At approval handoff, the project owner reported:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

B0 itself is research/documentation and performs no product implementation.

## Implementation release

The approved next slice is:

~~~text
LIB015-B1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDS_ON=OWNER_APPROVAL_OF_LIB015_B0
~~~

Goal:

1. harden LogEvent canonical construction and public safe recognition;
2. implement deterministic plain TextFormatter;
3. implement a borrowed TextWriter-backed TextSink using
   `writeLine(...).value()`, with no implicit flush/close;
4. implement the minimum MemorySink; and
5. add byte-for-byte, recognition, ordering, escaping, writer-failure,
   suspension-boundary and lifecycle tests.

FilteringSink and FanOutSink remain deferred.

## Final machine-readable state

~~~text
LIB015_B0_STATUS=RATIFIED

PLAIN_FORMATTER_PUBLIC_CONTRACT_CLOSED=YES
PLAIN_FORMATTER_LINE_ORIENTED=YES
NEWLINE_OWNER=TEXT_SINK
MULTILINE_POLICY=ESCAPE_TO_SINGLE_PHYSICAL_LINE
STRING_ESCAPING=DOUBLE_QUOTED_DETERMINISTIC_CONTROL_ESCAPING
FIELD_ORDERING=ASCENDING_LEXICOGRAPHIC_UNICODE_SCALAR_SEQUENCE

ERROR_RENDERING_POLICY=PRESENCE_MARKER_ERROR_TRUE_ONLY

LOGEVENT_RECOGNITION_SUFFICIENT=NO
LIB015_A_FOLLOWUP_REQUIRED=YES
LIB015_B_CAN_IMPLEMENT_WITH_CURRENT_LOGEVENT=NO

TEXTWRITER_SINK_WRITER_OWNERSHIP=BORROWED
TEXTWRITER_SINK_AWAITS_WRITE_FUTURE=YES
CAN_BASIC_TEXT_LOGGING_SUSPEND_CALLING_TASK=YES

TEXTWRITER_SINK_FLUSHES_EACH_EVENT=NO
TEXTWRITER_SINK_CLOSES_WRITER=NO

MEMORY_SINK_CONTRACT_CLOSED=YES

FILTERING_SINK=DEFER
FANOUT_SINK=DEFER

NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO

READY_FOR_OWNER_APPROVAL=YES
OWNER_APPROVAL=YES
READY_FOR_LIB015_B_IMPLEMENTATION=YES

NEXT_SLICE=LIB015-B1
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~
