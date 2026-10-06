# AUD018 — External package/build and test-runner presentation vs logging audit

Status: **COMPLETE**

Issue: `guillermomolina/protos#808`

Related consumer/design work: `LIB015 / guillermomolina/protos#432`

Research date: **2026-10-06**

## Purpose

This record preserves the external source evidence used to decide the consumer boundary between user-facing presentation/reporting and future `std:logging` use in Protos Package Tool and Test Tool.

The audit does **not** treat stderr, warnings, errors, or functions named `Log` as logging by definition. It follows the implementation path far enough to identify the abstraction that owns the information being emitted.

## Validation and publication context

The research was web/upstream-source-only and changed no product source.

The maintainer reported before coordination:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
PROTOS_SOURCE_CHANGED_BY_AUD018=NO
~~~

## Upstream revisions inspected

The following upstream default-branch revisions were captured on 2026-10-06 so the evidence is reproducible rather than depending only on moving branch names:

| Project | Branch | Revision |
| --- | --- | --- |
| rust-lang/cargo | master | `784be26fd122c9d181def7d5241c2f5adcc042f1` |
| npm/cli | latest | `b317f16c80df02ea3628cfa77170d5ae9b59720c` |
| npm/proc-log | main | `c18205b7d292b2492daa05ef886815193aad7cd3` |
| pypa/pip | main | `a7002c9771a6c3f0317a4e6b9fbdcd22e643f7b6` |
| apache/maven | master | `22451c0efd8c42f71859c43fde56f70a1cd12f08` |
| gradle/gradle | master | `81c85f2d52959d1bb34e523e4309de16ede8fc0c` |
| golang/go | master | `77fb1abefe3a6118959bab495627d8e3452ab484` |
| rust-lang/rust | main | `b57eb9a5fd94f30697f51fc88fbe898e67c7aae6` |
| pytest-dev/pytest | main | `2950bc2392f521fc0fd5e889a6a9857bc9d3ffe8` |
| python/cpython | main | `82c62abc4538e371c976658bc4444c8834c01dcc` |
| jestjs/jest | main | `61050e9323e742539dc2360236671110e37329e6` |
| apache/maven-surefire | master | `72a19f3f29ce8263084d3ee5415d3117b5c0ee41` |

## Classification vocabulary

Each surface is classified with the vocabulary required by AUD018:

~~~text
DIRECT_STREAM
TERMINAL_SHELL
REPORTER_FORMATTER
LOGGING
TRACING
STRUCTURED_EVENT
CAPTURED_PROGRAM_IO
MIXED
NOT_APPLICABLE
~~~

# Package/build tools

## Cargo

Primary source:

- `crates/cargo-util-terminal/src/shell.rs` at `rust-lang/cargo@784be26f`
  - https://github.com/rust-lang/cargo/blob/784be26fd122c9d181def7d5241c2f5adcc042f1/crates/cargo-util-terminal/src/shell.rs
- `src/bin/cargo/main.rs` at the same revision
  - https://github.com/rust-lang/cargo/blob/784be26fd122c9d181def7d5241c2f5adcc042f1/src/bin/cargo/main.rs

`Shell` is Cargo's presentation abstraction. Its status, warning, error and terminal/progress functions ultimately write through the shell-owned stdout/stderr writers. This is a presentation path, not a tracing path.

`setup_logger()` in the Cargo executable configures `tracing_subscriber` using `CARGO_LOG`; Cargo can also configure logging/profile behavior separately from the shell. That means Cargo deliberately has two ownership domains:

~~~text
user-facing Cargo status/warning/error/progress
    -> Shell
    -> terminal writer

internal debug/trace
    -> tracing
    -> CARGO_LOG filtering/subscriber
~~~

Important consequence: a user-visible Cargo warning/error does not become logging merely because it is severe or reaches stderr.

Classification:

- normal output: `TERMINAL_SHELL`
- final result/status: `TERMINAL_SHELL`
- progress/lifecycle: `TERMINAL_SHELL`
- user warning/error: `TERMINAL_SHELL`
- debug/internal: `TRACING`
- tracing/profiling: `TRACING`
- machine-readable output: shell/writer path with explicit serialization
- child process stdout/stderr: separate process streaming/capture path, not `CARGO_LOG`

## npm CLI

Primary sources:

- `npm/proc-log/lib/index.js` at `npm/proc-log@c18205b7`
  - https://github.com/npm/proc-log/blob/c18205b7d292b2492daa05ef886815193aad7cd3/lib/index.js
- `npm/cli/lib/utils/display.js` at `npm/cli@b317f16c`
  - https://github.com/npm/cli/blob/b317f16c80df02ea3628cfa77170d5ae9b59720c/lib/utils/display.js

`proc-log` exposes conceptually separate event families for `output`, `log`, `input`, and `time`. `display.js` installs separate handlers and state for these channels. This is explicit architectural evidence that npm does not define all visible output as logging.

The important distinction is:

~~~text
output.*
    -> command/result presentation events

log.*
    -> leveled log events (warn/info/verbose/silly/etc.)

input.*
    -> input interaction

time.*
    -> timing lifecycle
~~~

Therefore npm can legitimately have both `output.error` and `log.error`; the severity word alone does not determine the owning layer.

Lifecycle child-script output is another separate concern and can be foregrounded or suppressed independently of npm's own logging.

Classification:

- normal output/result: `STRUCTURED_EVENT` (`output`)
- progress: `MIXED` (display/progress infrastructure)
- warnings/errors: `MIXED` (`output` or `log` according to semantics)
- debug/verbose/internal: `LOGGING`
- timing: `STRUCTURED_EVENT`
- child program stdout/stderr: `CAPTURED_PROGRAM_IO` / `MIXED`

## pip

Primary sources:

- `src/pip/_internal/utils/misc.py` at `pypa/pip@a7002c97`
  - https://github.com/pypa/pip/blob/a7002c9771a6c3f0317a4e6b9fbdcd22e643f7b6/src/pip/_internal/utils/misc.py
- `src/pip/_internal/utils/logging.py`
  - https://github.com/pypa/pip/blob/a7002c9771a6c3f0317a4e6b9fbdcd22e643f7b6/src/pip/_internal/utils/logging.py
- `src/pip/_internal/commands/install.py`
  - https://github.com/pypa/pip/blob/a7002c9771a6c3f0317a4e6b9fbdcd22e643f7b6/src/pip/_internal/commands/install.py

pip is a genuine logging-heavy counterexample. `write_output()` delegates textual command output to `logger.info`, and installation/status/warning/error paths use logging levels extensively.

However pip still does not make logging the owner of every output surface. Rich/spinner progress has distinct machinery, and machine-readable `--report -` output is emitted as JSON rather than being represented as ordinary log records. pip documentation/code even needs users to combine report output with quiet behavior so logging text does not contaminate the JSON stream.

Subprocess output may be read and re-emitted via pip's subprocess logger, so pip also demonstrates a deliberate choice to fold some child-process output into the operational log when that is useful.

Classification:

- normal textual output: `LOGGING`
- final textual result: `LOGGING`
- progress: `MIXED`
- warnings/errors: `LOGGING`
- debug/verbose/internal: `LOGGING`
- machine-readable output: `DIRECT_STREAM` / dedicated JSON presentation
- child process stdout/stderr: `MIXED`, often integrated into logging

## Apache Maven

Primary sources:

- `compat/maven-embedder/src/main/java/org/apache/maven/cli/event/ExecutionEventLogger.java` at `apache/maven@22451c0e`
  - https://github.com/apache/maven/blob/22451c0efd8c42f71859c43fde56f70a1cd12f08/compat/maven-embedder/src/main/java/org/apache/maven/cli/event/ExecutionEventLogger.java
- `compat/maven-embedder/src/main/java/org/apache/maven/cli/transfer/ConsoleMavenTransferListener.java`
  - https://github.com/apache/maven/blob/22451c0efd8c42f71859c43fde56f70a1cd12f08/compat/maven-embedder/src/main/java/org/apache/maven/cli/transfer/ConsoleMavenTransferListener.java

`ExecutionEventLogger` converts Maven execution/lifecycle events into logger calls. Project starts, reactor summaries and build-success presentation go through info-level logging; build failures use error-level logging. This is not accidental use of a logger for diagnostics: the Maven build log is a primary user interface.

Interactive transfer progress is a useful limit case: the console transfer listener writes directly to its output stream and flushes/repaints terminal content. Maven therefore remains mixed at surfaces where terminal behavior is not naturally modeled as ordinary log records.

Classification:

- normal lifecycle output: `LOGGING`
- final build result: `LOGGING`
- progress: `MIXED`
- warnings/errors: `LOGGING`
- debug/internal: `LOGGING`
- transfer terminal UI: `DIRECT_STREAM`

## Gradle

Primary source examples at `gradle/gradle@81c85f2d`:

- `platforms/core-runtime/logging/src/main/java/org/gradle/internal/logging/LoggingManagerInternal.java`
  - https://github.com/gradle/gradle/blob/81c85f2d52959d1bb34e523e4309de16ede8fc0c/platforms/core-runtime/logging/src/main/java/org/gradle/internal/logging/LoggingManagerInternal.java
- lifecycle calls such as `logger.lifecycle(...)` are pervasive in Gradle runtime/build actions
  - example: https://github.com/gradle/gradle/blob/81c85f2d52959d1bb34e523e4309de16ede8fc0c/platforms/core-runtime/gradle-cli/src/main/java/org/gradle/launcher/cli/WelcomeMessageAction.java

Gradle is the strongest logging-centric example. Its logging model has a `LIFECYCLE` level specifically suited to ordinary build lifecycle presentation, alongside warning/info/debug/error levels. `LoggingManagerInternal.captureStandardOutput(LogLevel)` and related capture APIs also demonstrate that standard output can deliberately be redirected into Gradle's logging infrastructure.

This model makes sense because Gradle treats the build log itself as a first-class presentation channel with configurable verbosity, rich console rendering and central capture.

Classification:

- normal output: `LOGGING`
- progress/lifecycle: `LOGGING`
- warnings/errors: `LOGGING`
- debug/internal: `LOGGING`
- captured standard output: `LOGGING` / `MIXED`
- tracing as a separate mandatory presentation model: `NOT_APPLICABLE`

## Go command/toolchain

Primary source:

- `src/cmd/go/internal/base/base.go` at `golang/go@77fb1abe`
  - https://github.com/golang/go/blob/77fb1abefe3a6118959bab495627d8e3452ab484/src/cmd/go/internal/base/base.go

The Go command uses direct `fmt.Fprintf`/`fmt.Printf`-style presentation in several CLI paths and explicitly wires child `Stdout`/`Stderr` to the process streams in execution paths. Some errors use `log.Printf`.

This is evidence that a local call to a package named `log` is insufficient to classify the complete architecture as logging-centric. The surrounding CLI remains primarily direct-stream oriented.

Classification:

- normal output: `DIRECT_STREAM`
- progress/command echo: `DIRECT_STREAM`
- warning/error: `MIXED`
- debug/verbose: `DIRECT_STREAM` / `MIXED`
- child process stdout/stderr: `CAPTURED_PROGRAM_IO` / inherited streams

## Package/build matrix

| Tool | Normal output | Progress | Warning/error | Debug/internal | Logging role | Presentation/logging separated? |
| --- | --- | --- | --- | --- | --- | --- |
| Cargo | `TERMINAL_SHELL` | `TERMINAL_SHELL` | `TERMINAL_SHELL` | `TRACING` | internal observability through `CARGO_LOG`/tracing | **Yes, strongly** |
| npm CLI | `STRUCTURED_EVENT` | `MIXED` | `MIXED` | `LOGGING` | leveled logs distinct from `output` events | **Yes, explicitly** |
| pip | `LOGGING` | `MIXED` | `LOGGING` | `LOGGING` | owns much human textual UX and subprocess diagnostics | **Partial** |
| Maven | `LOGGING` | `MIXED` | `LOGGING` | `LOGGING` | build lifecycle log is primary UI | **Mostly no for textual lifecycle; yes for specialized terminal UI** |
| Gradle | `LOGGING` | `LOGGING` | `LOGGING` | `LOGGING` | build log is primary UI; stdout can be captured into it | **No for primary textual build UX** |
| Go command | `DIRECT_STREAM` | `DIRECT_STREAM` | `MIXED` | `MIXED` | limited/convenience role, not the whole UI | **Yes architecturally** |

Package/build conclusion:

~~~text
PACKAGE_BUILD_DOMINANT_PATTERN=MIXED_NO_DOMINANT_PATTERN
~~~

There are two mature families:

1. presentation/output separated from operational logging/tracing — Cargo, npm, largely Go;
2. build-log-as-interface — Maven, Gradle, and partially pip.

Maven/Gradle prove that routing lifecycle/warning/error presentation through a logging framework is valid **when the build log itself is intentionally the product UI**. They do not prove that every package tool should make logging own presentation.

# Test runners

## Rust libtest / cargo test reporting path

Primary source:

- `library/test/src/console.rs` at `rust-lang/rust@b57eb9a5`
  - https://github.com/rust-lang/rust/blob/b57eb9a5fd94f30697f51fc88fbe898e67c7aae6/library/test/src/console.rs

The core path is an event/result model feeding a formatter:

~~~text
TestEvent
    -> console event handling
    -> OutputFormatter
       -> write_test_start
       -> write_timeout
       -> write_result
       -> write_run_finish
    -> writer
~~~

Completed-test output is carried with the test result and passed to formatting. It is not redefined as generic runner logging.

Classification:

- test start/progress: `REPORTER_FORMATTER`
- PASS/FAIL/result: `REPORTER_FORMATTER`
- timeout: `REPORTER_FORMATTER`
- summary: `REPORTER_FORMATTER`
- machine-readable formats: `REPORTER_FORMATTER`
- tested stdout: `CAPTURED_PROGRAM_IO`
- generic operational logging as result owner: `NOT_APPLICABLE`

## pytest

Primary sources:

- `src/_pytest/terminal.py` at `pytest-dev/pytest@2950bc23`
  - https://github.com/pytest-dev/pytest/blob/2950bc2392f521fc0fd5e889a6a9857bc9d3ffe8/src/_pytest/terminal.py
- `src/_pytest/logging.py`
  - https://github.com/pytest-dev/pytest/blob/2950bc2392f521fc0fd5e889a6a9857bc9d3ffe8/src/_pytest/logging.py

`TerminalReporter` owns test progress/result presentation through pytest hooks such as runtest log/report callbacks and a terminal writer. Separately, `LoggingPlugin`/`LogCaptureHandler` capture Python `logging.LogRecord` data.

pytest's own capture options distinguish `stdout`, `stderr`, and `log`, which is strong semantic evidence that captured program streams and logging are independent categories.

Classification:

- test result/progress: `REPORTER_FORMATTER`
- runner warning/error: `REPORTER_FORMATTER` / `MIXED`
- runner tracing/debug: separate hook/trace facilities
- tested stdout/stderr: `CAPTURED_PROGRAM_IO`
- tested Python logs: `LOGGING` captured as a separate channel

## Python unittest

Primary sources:

- `Lib/unittest/runner.py` at `python/cpython@82c62abc`
  - https://github.com/python/cpython/blob/82c62abc4538e371c976658bc4444c8834c01dcc/Lib/unittest/runner.py
- `Lib/unittest/result.py`
  - https://github.com/python/cpython/blob/82c62abc4538e371c976658bc4444c8834c01dcc/Lib/unittest/result.py

`TestResult` owns structured execution state such as failures, errors and skipped tests. `TextTestResult` / `TextTestRunner` render that result to a stream. Buffering of test stdout/stderr is test-program I/O handling, not generic operational logging.

Classification:

- result/progress: `REPORTER_FORMATTER`
- runner warning/error: `REPORTER_FORMATTER`
- verbose behavior: reporter configuration
- tested stdout/stderr: `CAPTURED_PROGRAM_IO`
- logging as result owner: `NOT_APPLICABLE`

## Jest

Primary sources:

- `packages/jest-reporters/src/types.ts` at `jestjs/jest@61050e93`
  - https://github.com/jestjs/jest/blob/61050e9323e742539dc2360236671110e37329e6/packages/jest-reporters/src/types.ts
- `packages/jest-reporters/src/BaseReporter.ts`
  - https://github.com/jestjs/jest/blob/61050e9323e742539dc2360236671110e37329e6/packages/jest-reporters/src/BaseReporter.ts
- `packages/jest-reporters/src/DefaultReporter.ts`
  - https://github.com/jestjs/jest/blob/61050e9323e742539dc2360236671110e37329e6/packages/jest-reporters/src/DefaultReporter.ts

The Reporter API receives structured callbacks such as run start, test start, test-case result, test result and run complete. `BaseReporter.log()` eventually writes to a stream; its method name does not turn it into a generic logging framework.

Captured console data belongs to the test result and is rendered by reporter code.

Classification:

- result/progress: `STRUCTURED_EVENT` -> `REPORTER_FORMATTER`
- runner warning/error: `REPORTER_FORMATTER`
- tested console output: `CAPTURED_PROGRAM_IO`
- generic logger as primary result model: `NOT_APPLICABLE`

## Go testing and test2json

Primary sources:

- `src/testing/testing.go` at `golang/go@77fb1abe`
  - https://github.com/golang/go/blob/77fb1abefe3a6118959bab495627d8e3452ab484/src/testing/testing.go
- `src/cmd/internal/test2json/test2json.go`
  - https://github.com/golang/go/blob/77fb1abefe3a6118959bab495627d8e3452ab484/src/cmd/internal/test2json/test2json.go
- `src/cmd/test2json/main.go`
  - https://github.com/golang/go/blob/77fb1abefe3a6118959bab495627d8e3452ab484/src/cmd/test2json/main.go

`T.Log` / `T.Logf` are semantically test-attached output. They feed the test result/output model and are commonly revealed according to test success/verbosity. They are not equivalent to a generic operational logger with independent sinks and filtering.

`test2json` makes the structured-event boundary explicit: actions, package/test identity, elapsed time and output are serialized as test events.

Classification:

- test result/progress: `DIRECT_STREAM` / `STRUCTURED_EVENT`
- `T.Log` / `Logf`: test-attached output, not generic operational `LOGGING`
- machine-readable output: `STRUCTURED_EVENT`
- tested/process stdout/stderr: `CAPTURED_PROGRAM_IO` feeding test output/event conversion

## Maven Surefire

Primary sources:

- `maven-surefire-common/src/main/java/org/apache/maven/plugin/surefire/AbstractSurefireMojo.java` at `apache/maven-surefire@72a19f3f`
  - https://github.com/apache/maven-surefire/blob/72a19f3f29ce8263084d3ee5415d3117b5c0ee41/maven-surefire-common/src/main/java/org/apache/maven/plugin/surefire/AbstractSurefireMojo.java
- `maven-surefire-common/src/main/java/org/apache/maven/plugin/surefire/log/PluginConsoleLogger.java`
  - https://github.com/apache/maven-surefire/blob/72a19f3f29ce8263084d3ee5415d3117b5c0ee41/maven-surefire-common/src/main/java/org/apache/maven/plugin/surefire/log/PluginConsoleLogger.java

`AbstractSurefireMojo` constructs distinct reporter responsibilities, including `SurefireStatelessReporter`, `SurefireConsoleOutputReporter`, and `SurefireStatelessTestsetInfoReporter`. Separately, `PluginConsoleLogger` adapts Surefire diagnostics/console logging to Maven/SLF4J levels.

This is the key exception test for AUD018: even inside a logging-centric host, Surefire preserves a reporter/result model for test semantics and captured output. Integration with Maven logging affects the sink/console path; it does not collapse test results into generic log records.

Classification:

- test result/progress: `REPORTER_FORMATTER`, with a logging-integrated console sink in parts
- runner warning/error/debug: `LOGGING` via Maven integration
- XML/report artifacts: `REPORTER_FORMATTER`
- tested stdout/stderr: `CAPTURED_PROGRAM_IO` -> dedicated output reporter

## Test-runner matrix

| Tool | Test result/progress | Runner warning/error | Runner debug/internal | Tested-program output | Logging role | Reporter/logging separated? |
| --- | --- | --- | --- | --- | --- | --- |
| Rust libtest | `STRUCTURED_EVENT` -> `REPORTER_FORMATTER` | `REPORTER_FORMATTER` | no central generic logger owns results | `CAPTURED_PROGRAM_IO` | not result owner | **Yes, strongly** |
| pytest | `REPORTER_FORMATTER` | `REPORTER_FORMATTER` / `MIXED` | separate trace/logging facilities | `CAPTURED_PROGRAM_IO`; logs separately captured | capture/diagnostics, not result ownership | **Yes, explicitly** |
| unittest | `REPORTER_FORMATTER` | `REPORTER_FORMATTER` | no central operational logger | `CAPTURED_PROGRAM_IO` | no central role | **Yes** |
| Jest | `STRUCTURED_EVENT` -> `REPORTER_FORMATTER` | `REPORTER_FORMATTER` | reporter/internal facilities | `CAPTURED_PROGRAM_IO` | reporter `log()` is not a generic logger | **Yes** |
| Go testing | `DIRECT_STREAM` / `STRUCTURED_EVENT` | result machinery | no central operational logger | `CAPTURED_PROGRAM_IO` | `T.Log` is test-attached output | **Yes conceptually** |
| Maven Surefire | `REPORTER_FORMATTER` + Maven-integrated console sink | `LOGGING` | `LOGGING` | `CAPTURED_PROGRAM_IO` -> reporter | host integration/diagnostics | **Yes in model; partially joined at sink** |

Test-runner conclusion:

~~~text
TEST_RUNNER_DOMINANT_PATTERN=REPORTER_PLUS_OPTIONAL_LOGGING
~~~

The repeated model is:

~~~text
execution/result/event state
    -> reporter / formatter
       -> human terminal
       -> JSON/XML/machine report
       -> summary

operational diagnostics
    -> logging/tracing when present

tested program stdout/stderr
    -> captured program I/O
    -> associated with test result/event
~~~

# Cross-tool conclusions

## 1. Package/build tools

There is no single dominant architecture across the mandatory sample. Cargo/npm/Go separate presentation from logging/tracing; Maven/Gradle and much of pip deliberately make the logging system part of normal command/build presentation.

The decisive architectural question is therefore not whether an output is a warning/error, but whether the tool has intentionally defined **the build log as its user interface**.

## 2. Test runners

Test runners show a much stronger common pattern: test execution state is modeled as test-domain results/events and only then rendered through reporters/formatters. Generic logging can coexist with this model, but does not normally replace it.

## 3. Package Tool and Test Tool are only partially the same

They share one boundary:

~~~text
operational/internal diagnostics
    -> logging
~~~

But Test Tool needs an additional semantic layer:

~~~text
CaseResult / SuiteResult / execution events
    -> reporter/event model
    -> human or machine rendering
~~~

Package Tool does not inherently need that extra test-result domain and can use ordinary command presentation writers for contractual command output.

# Protos application

## Package Tool recommended boundary

Unless Protos explicitly decides to adopt a Maven/Gradle-style **build-log-is-the-UI** architecture, the external evidence supports keeping the current conceptual ownership split:

~~~text
command/result presentation
user-facing warning/error
terminal progress
machine-readable command output
    -> presentation / TextWriter / dedicated serializer

operational diagnostics
debug/verbose internals
resolution internals
    -> std:logging
~~~

Maven/Gradle are valid precedents for another design, but adopting that precedent should be an explicit tool-architecture decision rather than an automatic consequence of adding `std:logging`.

## Test Tool recommended boundary

The external evidence is much stronger and should be treated as an implementation constraint unless explicitly reopened:

~~~text
CaseResult / SuiteResult
case start/completion
PASS / FAIL / SKIP / timeout
progress
summary
machine-readable test reports
    -> result/event model -> reporter

tested stdout/stderr
    -> captured program I/O associated with case/result

runner operational/debug diagnostics
    -> std:logging
~~~

Generic `std:logging` should not become the canonical representation of test outcomes.

# LIB015 consumer requirements established by AUD018

Evidence supports `std:logging` being suitable for:

- operational/debug messages;
- verbose diagnostics;
- severities/levels;
- structured context;
- filtering;
- explicit sinks;
- log-record formatting;
- internal resolution/runner diagnostics.

Evidence does **not** support requiring `std:logging` to replace:

- command result presentation;
- terminal progress UI;
- machine-readable command output;
- test result/reporting events;
- test reporters/formatters;
- captured child/test stdout;
- captured child/test stderr.

Tracing/profiling should not automatically be collapsed into logging. Cargo has a tracing-oriented observability route, while other ecosystems keep tracing distinct or do not model it as another ordinary severity. LIB015 may leave future interoperability room without needing to own all tracing in its baseline API.

An important semantic rule for future implementation is:

~~~text
severity does not determine ownership
~~~

For example:

- a package compatibility error intended for the user can remain presentation;
- a resolver cache inconsistency can be operational logging;
- `test X failed` is a test result/reporting event;
- text written by the tested program is captured program I/O.

# Implementation handoff

When Package Tool or Test Tool logging integration is requested later, this audit is sufficient external precedent for the consumer boundary. Implementation research should inspect the then-current Protos HEAD to map concrete call sites, but it does **not** need to repeat this external ecosystem survey unless a materially new architecture is proposed.

Expected future implementation rule:

~~~text
PACKAGE_TOOL:
  contractual presentation/results -> existing presentation writer layer
  user warnings/errors             -> presentation unless build-log-as-UI is explicitly selected
  operational/debug                -> std:logging

TEST_TOOL:
  CaseResult/events                -> reporter
  PASS/FAIL/progress/summary       -> reporter
  tested stdout/stderr             -> captured program I/O
  runner operational/debug         -> std:logging
~~~

# Final AUD018 result

~~~text
AUD018_STATUS=COMPLETE

PACKAGE_BUILD_DOMINANT_PATTERN=
    MIXED_NO_DOMINANT_PATTERN

TEST_RUNNER_DOMINANT_PATTERN=
    REPORTER_PLUS_OPTIONAL_LOGGING

PACKAGE_TOOL_RECOMMENDED_BOUNDARY=contractual result/presentation, user warnings/errors, progress and machine output remain presentation/writer concerns; std:logging owns operational/debug/verbose/internal resolution diagnostics unless a separate explicit build-log-as-UI decision is made

TEST_TOOL_RECOMMENDED_BOUNDARY=CaseResult/SuiteResult and execution events feed reporters for PASS/FAIL/progress/summary and machine formats; tested stdout/stderr remain captured program I/O; std:logging owns runner operational/debug diagnostics

PACKAGE_AND_TEST_BOUNDARY_EFFECTIVELY_THE_SAME=PARTIAL

STD_LOGGING_SHOULD_OWN_USER_PRESENTATION=NO

STD_LOGGING_SHOULD_OWN_OPERATIONAL_DEBUG_LOGS=YES

STD_LOGGING_SHOULD_REPLACE_TEST_REPORTING=NO

STD_LOGGING_SHOULD_REPLACE_CAPTURED_PROGRAM_IO=NO

EXTERNAL_EVIDENCE_SUFFICIENT_FOR_LIB015_CONSUMER_DECISION=YES
~~~

No additional AUD018 research slice is required.

## AI-assistance disclosure

This durable research record was materially prepared with AI assistance from ChatGPT using the cited public upstream source evidence. No human authorship or independent human review is claimed by this record.