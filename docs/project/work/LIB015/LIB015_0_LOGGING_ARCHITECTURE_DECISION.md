# LIB015-0 — Structured logging architecture decision and ratification record

Status: **RATIFIED — immutable event + explicit Logger + explicit Sink**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Nature: comparative Standard Library architecture record; **non-normative**

Explicit project-owner approval: **2026-10-06**

Approval provenance: issue comment `6021173398`

Protos evidence baseline at durable publication preparation: `41a06e07d5d74053911f291ac27f911678c61ff5`

Project-record base before publication: `5cde5abeeed1291c3f7eeb216379ccf8950f46f0`

Validation class: `GOVERNANCE_DOCUMENTATION_ONLY`

## Purpose

LIB015 defines the future Protos Standard Library boundary for structured
operational logging.

The design goal is to make enabled logging structured, composable and useful
while keeping disabled logging cheap, retaining explicit I/O and clock
authority, avoiding mandatory global mutable state, and remaining suitable for
Tasks, Actors, Processes, Native Image and alternate runtimes.

This record preserves the completed LIB015-0 investigation and the project
owner's explicit approval of the recommended architecture. Observable Standard
Library semantics remain authoritative only when published through the normal
Protos specification/library process in `guillermomolina/protos`. This durable
record is decision history and implementation guidance, not a normative
specification.

## Investigation constraints

LIB015-0 was research only.

~~~text
COMMANDS_EXECUTED=NO
TESTS_EXECUTED_BY_AGENT=NO
IMPLEMENTATION_PERFORMED_DURING_RESEARCH=NO
PROTOS_REPOSITORY_MODIFIED_DURING_RESEARCH=NO
~~~

Maintainer-reported repository state before ratification:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

The investigation preserved these architectural constraints:

1. no ambient stdout/stderr/filesystem/network authority in the logging core;
2. no implicit current-time lookup in pure event construction;
3. no mandatory process-global mutable logger registry;
4. no hidden background Task/thread/queue required by basic logging;
5. Error semantics remain unchanged;
6. structured event semantics remain independent of terminal styling;
7. Package Tool presentation and Test Tool reporting remain outside generic
   logging;
8. existing Standard Library capabilities are reused instead of duplicated;
9. the design must remain portable beyond the current JVM implementation; and
10. optional costs must remain pay-as-you-grow.

## Prior decisions reused

### AUD018 / #808

AUD018 established the consumer boundary that LIB015 must preserve.

Package Tool:

~~~text
contractual result/presentation
progress
machine-readable output
user-facing command warnings/errors
    -> presentation / writer layer

operational/debug/verbose/internal diagnostics
    -> std:logging
~~~

Test Tool:

~~~text
CaseResult / SuiteResult
PASS/FAIL/progress/summary
machine-readable reports
    -> result/event model -> reporter

tested-program stdout/stderr
    -> captured program I/O

runner operational/debug diagnostics
    -> std:logging
~~~

Therefore LIB015 does not become a universal CLI renderer, progress protocol,
test-reporting protocol or replacement for captured program I/O.

### LIB013 / #430

LIB013 ratified pure temporal values separated from current-time authority.

For LIB015 this means:

- timestamp values use `std:datetime/Instant`;
- current time remains an explicit capability/authority concern;
- logging does not introduce `timestampMillis`, `epochNanos`,
  `java.time.Instant` or a logging-specific timestamp as public substitutes;
- temporal parsing/formatting remains owned by datetime profiles/adapters.

At the current Protos baseline, LIB013-A and LIB013-B are published:
`Date`, `Time`, `LocalDateTime`, `Duration` and `Period` exist.
`Instant` is still expected from the already-ratified LIB013-C slice.

## External architecture evidence

The investigation compared current public architecture/documentation and
representative upstream source for:

- Python `logging` and `structlog`;
- Java `java.util.logging`, SLF4J and Log4j2;
- Rust `log`, `tracing` and `slog`;
- Go `log` and `log/slog`;
- .NET `Microsoft.Extensions.Logging` / `ILogger`;
- Swift Logging;
- Erlang/Elixir Logger mechanisms;
- Node pino and winston; and
- OpenTelemetry Logs.

The useful recurring separation is:

~~~text
structured event / record
        +
logger/filter/context policy
        +
handler/sink/output authority
        +
formatting/encoding
~~~

The closest architectural precedents for Protos are Go `slog`, Rust `slog`
and Swift Logging:

- derived loggers carry explicit contextual metadata;
- disabled levels can be rejected before expensive event construction;
- output behavior is delegated to an explicit handler/drain/backend;
- baseline logging does not require a universal process-global registry.

Python logging, JUL and Log4j2 provide strong evidence for keeping event records,
formatters and handlers distinct, but their conventional global hierarchy and
registry patterns are not selected for Protos.

Rust `tracing` and OpenTelemetry demonstrate a future path toward richer
event/span processing and correlation without requiring baseline LIB015 to
become a tracing system.

Representative source/documentation references used during research include:

- Python logging:
  https://docs.python.org/3/library/logging.html
- structlog:
  https://www.structlog.org/
- Java logging:
  https://docs.oracle.com/en/java/javase/25/docs/api/java.logging/java/util/logging/package-summary.html
- SLF4J:
  https://www.slf4j.org/manual.html
- Log4j2 architecture:
  https://logging.apache.org/log4j/2.x/manual/architecture.html
- Rust log:
  https://docs.rs/log/
- Rust tracing:
  https://docs.rs/tracing/
- Rust slog:
  https://docs.rs/slog/
- Go slog:
  https://pkg.go.dev/log/slog
- .NET ILogger:
  https://learn.microsoft.com/dotnet/core/extensions/logging
- Swift Logging:
  https://github.com/apple/swift-log
- Erlang logger:
  https://www.erlang.org/doc/apps/kernel/logger.html
- Elixir Logger:
  https://hexdocs.pm/logger/Logger.html
- pino:
  https://github.com/pinojs/pino
- winston:
  https://github.com/winstonjs/winston
- OpenTelemetry Logs data model:
  https://opentelemetry.io/docs/specs/otel/logs/data-model/
- NO_COLOR convention:
  https://no-color.org/

## Existing Protos capabilities to reuse

| LIB015 need | Existing/future Protos capability | Decision |
| --- | --- | --- |
| timestamp value | `std:datetime/Instant` | reuse after LIB013-C |
| temporal formatting | datetime profiles/adapters | reuse; logging does not own a second temporal formatter |
| current-time authority | explicit runtime/capability boundary | remain explicit |
| text output | Core `TextWriter` | reuse |
| process stdout/stderr adaptation | `std:io/ProcessStreams` | caller-owned adapter only |
| encoding | Core/text I/O | reuse |
| structured fields | ordinary Protos Map/Array/String/Number/Boolean/null | reuse with bounded snapshot/representability rules |
| JSON | `std:json/JSON` | reuse through explicit event-to-JSON projection |
| diagnostic error | Core `Error` | attach without redefining or mutating |
| terminal style | no current clear owner | separate reusable terminal/styled-text abstraction |

The current `ProcessStreams` API requires an explicit Process and constructs
borrowing `TextWriter` wrappers from that Process's selected streams and
encodings. LIB015 preserves that authority boundary rather than discovering
stdout/stderr internally.

The existing JSON library already owns JSON node construction, compact encoding,
cycle rejection and malformed-tree failure. LIB015 therefore does not introduce a
logging-private JSON serializer.

## Candidate architectures

### A — Immutable Event + explicit Sink

Strong purity, testability and portability. Insufficient by itself for ergonomic
pre-event filtering and derived context.

### B — Explicit Logger owning policy + Sink

Strong ergonomics and early filtering. By itself it risks making the Logger the
semantic owner of too much event state.

### C — Explicit derived/scoped Logger

A base Logger can derive another Logger with additional explicit context without
mutating the original. This composes naturally with Tasks, Actors and Processes.

### D — Subscriber/event pipeline

Strongest tracing-style future flexibility, but introduces processors/subscriber
machinery not justified by baseline operational logging.

### E — Process-global registry/logger

Common external precedent, but rejected for Protos because it weakens explicit
authority, isolation, test determinism and alternate-runtime freedom.

## Candidate scorecard

Scale: 1 = weakest, 5 = strongest.

| Architecture | Correctness | Protos alignment | Future resilience | Scalability | Simplicity | Portability | Runtime cost | Failure/operability | Reversibility | Evidence |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| A Event + Sink | 5 | 5 | 4 | 4 | 5 | 5 | 4 | 4 | 5 | 5 |
| B Logger-centric explicit | 4 | 4 | 4 | 4 | 5 | 5 | 5 | 4 | 4 | 5 |
| **A+B+C hybrid** | **5** | **5** | **5** | **5** | **4** | **5** | **5** | **5** | **5** | **5** |
| D Subscriber pipeline | 5 | 4 | 5 | 5 | 2 | 4 | 3 | 4 | 4 | 5 |
| E Global registry | 3 | 1 | 2 | 2 | 3 | 2 | 3 | 3 | 2 | 5 |

Focused winner assessment:

~~~text
AGUANTE_DE_FUTURO=5
ESCALABILIDAD=5
FILOSOFIA_PROTOS=5
~~~

The selected hybrid wins because it combines an independent immutable event
value with an ergonomic explicit Logger and an independently replaceable Sink.
A subscriber/pipeline can later be implemented behind a Sink without changing
ordinary call sites or the LogEvent data model.

## Ratified core model

The owner approved:

~~~text
immutable LogEvent
        +
explicit immutable/derived Logger
        +
explicit Sink
~~~

Conceptually:

~~~text
explicit context
      |
      v
immutable/derived Logger
      |
      +-- level filter before event construction
      |
      v
immutable LogEvent
      |
      +--> human formatter --> styled/plain renderer --> TextWriter
      +--> JSON formatter --> std:json --> TextWriter
      +--> MemorySink
      +--> future adapter/pipeline/exporter
~~~

No mandatory process-global registry is part of the model.

## Levels

The initial canonical level set is:

~~~text
TRACE
DEBUG
INFO
WARN
ERROR
~~~

`FATAL` is deliberately omitted from the baseline because it tends to mix
severity with lifecycle/control-flow policy. Logging a value must not itself mean
that a Process is required to terminate.

Internal ordinal representations remain implementation details; public code
should use canonical level values rather than magic integers.

## Event model

The initial semantic shape is conceptually:

~~~text
LogEvent {
    level
    message
    fields
    timestamp
    error
}
~~~

Rules:

- events are immutable/frozen once constructed;
- `message` is human-readable String data;
- structured fields use ordinary Protos data rather than `LogField` /
  `LogValue` wrappers;
- enabled-event construction snapshots caller-supplied mutable field containers
  so later mutation cannot retroactively change an event;
- field ordering is not semantic;
- deterministic formatters impose deterministic output ordering;
- call-site fields override derived Logger context when the same key is present;
- `timestamp` is optional `std:datetime/Instant`;
- `error` is optional Core Error diagnostic attachment;
- the event contains no sink, formatter, writer, terminal capability or ANSI
  escape sequence.

Initially, source/module/task/actor/process IDs and future
trace/span/request/correlation IDs remain ordinary structured fields rather than
special compulsory LogEvent slots.

Lazy field closures are not part of v1. They complicate side effects,
transferability, lifetime and Error behavior. Early filtering already removes the
important disabled-event cost.

## Logger, filtering and context

Conceptually the Logger owns:

~~~text
minimum level / filter policy
derived context snapshot
sink
~~~

Important behavior:

- `logger.with(fields)` derives a new Logger rather than mutating the original;
- no dynamic/thread-local context is required;
- level filtering occurs before LogEvent creation;
- callers may query an `isEnabled(level)`-style predicate before building
  expensive diagnostic data;
- sink-side filters may exist for routing/fan-out after event construction, but
  they do not replace Logger-side early filtering.

This is the main pay-as-you-grow requirement for large volumes of disabled
DEBUG/TRACE events.

## Timestamp and clock authority

The public timestamp value is:

~~~text
std:datetime/Instant | null
~~~

Current-time authority is separate.

The Logger does not silently call a global `now()`. A caller may provide an
Instant explicitly, and a future explicit clock/enrichment adapter may attach one
when the program has deliberately supplied that authority.

LIB015 must not publish a temporary millisecond/nanosecond integer timestamp API
that would later require migration to `Instant`.

Therefore timestamp integration is staged after LIB013-C without blocking the
event/logger core.

## Formatter model

Formatting remains separate from LogEvent semantics.

Baseline families:

~~~text
plain human text
human styled text
JSON
~~~

A formatter can be a pure transformation whenever practical.

JSON formatting explicitly projects supported event fields to the existing
`std:json` node model and delegates final encoding to `std:json`.
Unsupported/cyclic values fail according to explicit formatter policy rather
than being silently stringified.

## Color / terminal styling decision

Color is required for human terminal logging, but ANSI is not part of LogEvent.

The approved ownership direction is a **separate reusable styled-text / terminal
rendering abstraction**, conceptually:

~~~text
LogEvent
    -> HumanFormatter
    -> StyledText
        -> Plain renderer
        -> Terminal renderer(capabilities, theme, color mode)
            -> TextWriter
~~~

This abstraction is reusable by CLI help, Package Tool, Test Tool, diagnostics
and logging. A logging-private terminal framework would put the capability in the
wrong architectural owner.

Color modes:

~~~text
AUTO
ALWAYS
NEVER
~~~

`AUTO` consumes explicitly supplied terminal capabilities. The pure logging
core does not query a TTY or read `NO_COLOR`, `FORCE_COLOR`, `CLICOLOR` or
other environment variables itself. A CLI/terminal authority layer may resolve
those conventions and construct the appropriate renderer/capability value.

The first implementation should stay deliberately portable: basic ANSI
16-color-class styling plus small theme/style support such as bold/dim where the
separate terminal abstraction approves it. 256-color and truecolor support are
future extensions, not baseline requirements.

A reasonable default human theme may visually distinguish TRACE/DEBUG/WARN/ERROR,
while INFO may remain unstyled or subtle. Exact palette constants belong to the
terminal/theme layer, not LogEvent.

Formatting tests use deterministic supplied capabilities and themes; no real
terminal is required.

## Sink model

The minimum conceptual sink operation is:

~~~text
sink.emit(event)
~~~

Useful compositions include:

~~~text
TextWriter-backed formatted sink
MemorySink
FilteringSink
FanOutSink
~~~

Filesystem/network authority is supplied outside LIB015. A file sink receives an
already-authorized writer or other explicit authority rather than discovering a
path by itself.

Basic logging is synchronous. It requires no hidden Task, background thread or
global queue.

Future optional layers may include:

~~~text
BufferedSink
AsyncSink
BatchingSink
network exporter
OpenTelemetry adapter
~~~

Those layers must explicitly define queue bounds, backpressure/drop behavior,
flush, shutdown, ownership and ordering.

No global total order is promised across concurrent producers. A sink may define
its own local ordering contract.

## Sink failure

Ordinary diagnostic logging must not unexpectedly become application
control-flow merely because a downstream logging destination fails.

The baseline architecture therefore separates ordinary best-effort log calls
from explicit sink lifecycle/strictness policy.

The implementation slice must make failure visibility deterministic and testable,
but it must not silently invent filesystem/network recovery authority. Explicit
flush/close or a deliberately strict sink policy may surface failures; ordinary
Logger calls should use containment policy rather than surprising guest control
flow.

The exact public names are implementation/API-sketch territory, not additional
architecture authority.

## Security / redaction

LIB015 does not automatically stringify arbitrary objects.

Reasons:

- arbitrary conversion can expose secrets or PII;
- it can execute unexpected user code;
- it weakens deterministic JSON behavior;
- it makes transferability and cost unpredictable.

The baseline uses an explicit supported structured-value domain and allows
redaction/filtering policy in formatter/sink composition. Secret discovery is
not magic core behavior.

## Concurrency and future tracing

Derived immutable Logger configuration and immutable events avoid a mandatory
global lock.

Tasks, Actors and Processes may each receive independent Loggers or share a sink
whose own concurrency contract permits it.

Correlation fields such as:

~~~text
traceId
spanId
requestId
correlationId
~~~

fit naturally in structured context. This preserves an escape path to an
OpenTelemetry or tracing adapter without making baseline std:logging an
OpenTelemetry implementation.

## Strongest argument against the selected architecture

The winner exposes more concepts than a classic single Logger API:

~~~text
Logger
LogEvent
Formatter
Sink
~~~

For tiny programs that can appear heavier than direct printing.

The mitigation is deliberate minimality: each contract remains small and Protos
does not add a manager, registry, configuration tree, backend SPI or global
bootstrap layer merely for symmetry with mature enterprise frameworks.

## Regret scenario and escape path

A future observability architecture may need processors, subscribers, spans,
sampling and remote routing.

The escape path is to implement a pipeline/subscriber adapter behind `Sink`:

~~~text
Logger -> LogEvent -> PipelineSink / SubscriberSink -> processors/exporters
~~~

The ordinary LogEvent and Logger APIs do not need to change.

## Future-stress result

The approved model was checked against:

1. a small single-process CLI;
2. concurrent Package Tool operations;
3. many Test Tool workers;
4. a long-lived server;
5. thousands of filtered DEBUG events;
6. JSON file logging;
7. colored human terminal logging;
8. redirected/piped output without ANSI;
9. simultaneous colored-terminal and plain/JSON sinks;
10. an Actor with an independent Logger;
11. multiple Processes;
12. a slow sink;
13. a failing sink;
14. a future async queue;
15. a future remote collector;
16. correlation IDs;
17. future OpenTelemetry integration;
18. Native Image;
19. a non-JVM runtime; and
20. deterministic unit tests without a real clock or terminal.

No hard architectural blocker was found.

One implementation caveat remains important: ordinary Protos values can carry
transferability constraints. LIB015 must obey the general Actor/Process transfer
rules and must not invent a logging-specific runtime exception that magically
makes arbitrary field objects transferable.

## Approved implementation decomposition

### LIB015-A — structured event + levels + Logger/filter/context

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDS_ON=LIB015-0_RATIFIED
GOAL=Implement immutable LogEvent, TRACE/DEBUG/INFO/WARN/ERROR, explicit derived
     Logger context and pre-event level filtering without timestamps or ambient
     output authority.
~~~

### LIB015-B — plain formatting and explicit sinks

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDS_ON=LIB015-A
GOAL=Implement deterministic plain human formatting and explicit sink
     composition over existing writer authority, including MemorySink and
     bounded failure semantics.
~~~

### LIB015-C — JSON adapter

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDS_ON=LIB015-B
GOAL=Implement LogEvent-to-std:json projection and JSON formatting without a
     logging-private JSON serializer.
~~~

### LIB015-D — Instant timestamp integration

~~~text
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
DEPENDS_ON=LIB015-A,LIB013-C
GOAL=Add optional std:datetime/Instant event timestamps without ambient
     current-time authority.
~~~

### Separate reusable terminal/styled-text work

The reusable terminal/styling abstraction satisfies an independent ownership
boundary and may receive its own formal coordination item before the colored
adapter is implemented. It does not block LIB015-A through the non-colored
logging core.

### Colored logging adapter

After the reusable terminal/styled-text capability exists, LIB015 may add the
human colored logging adapter without changing LogEvent.

## Owner approval

The project owner explicitly approved the recommended architecture on 2026-10-06:

> apruebo la arquitectura recomendada

The approval was mirrored to authoritative Issue #432 as comment
`6021173398`.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_PRIOR_INVARIANTS=NONE
~~~

## Final machine-readable state

~~~text
LIB015_0_RESEARCH_COMPLETE=YES
LIB015_0_STATUS=RATIFIED

RECOMMENDED_CORE_MODEL=IMMUTABLE_EVENT_PLUS_EXPLICIT_LOGGER_AND_SINK

PROCESS_GLOBAL_LOGGER_REQUIRED=NO

STRUCTURED_FIELDS_USE_ORDINARY_PROTOS_DATA=YES

TIMESTAMP_VALUE=STD_DATETIME_INSTANT
CURRENT_TIME_AUTHORITY=EXPLICIT

STD_DATETIME_REUSED=YES
STD_JSON_REUSED=YES
TEXTWRITER_REUSED=YES

COLOR_REQUIRED=YES
COLOR_OWNER=SEPARATE_STYLED_TEXT_LAYER
ANSI_PRESENT_IN_LOG_EVENT=NO
JSON_CAN_CONTAIN_TERMINAL_ANSI=NO
COLOR_MODES=AUTO_ALWAYS_NEVER

BASIC_LOGGING_REQUIRES_BACKGROUND_TASK=NO
BASIC_LOGGING_REQUIRES_GLOBAL_MUTABLE_REGISTRY=NO

PACKAGE_TOOL_PRESENTATION_MOVES_TO_LOGGING=NO
TEST_TOOL_REPORTING_MOVES_TO_LOGGING=NO

NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
SEPARATE_TERMINAL_STYLE_LIBRARY_RECOMMENDED=YES

READY_FOR_OWNER_ARCHITECTURE_APPROVAL=YES
OWNER_ARCHITECTURE_APPROVAL=YES
READY_FOR_IMPLEMENTATION=YES

NEXT_SLICE=LIB015-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~
