# LIB015-A — Structured logging core implementation evidence

Status: **COMPLETE**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-A — structured event + levels + Logger/filter/context`

Nature: implementation evidence; **non-normative**

Product revision: `c5bce4ef03417cce96b99924a0ae58e7cb95e7bb`

Implementation version: `0.3.247-SNAPSHOT`

LIB015-0 ratification record:
`docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`

Project-record base before publication: `271d9cacbd7c898d401fead52de5639ee7024ca3`

## Outcome

LIB015-A publishes the first executable `std:logging` core.

The slice implements:

- `std:logging/Level`;
- `std:logging/LogEvent`;
- `std:logging/Logger`;
- canonical `TRACE < DEBUG < INFO < WARN < ERROR` severity ordering;
- immutable structured events;
- deep field snapshots over the approved ordinary structured-data domain;
- explicit Logger construction with no global/default registry;
- `isEnabled(level)` early filtering;
- level convenience calls;
- explicit derived context through `logger.with(fields)`;
- base < derived < call-site field precedence;
- one explicit sink boundary, `emit(event)`;
- ordinary sink-Error containment for normal Logger emission.

No formatter, JSON encoding, terminal styling, timestamp lookup, clock authority,
filesystem/network authority, global Logger registry or hidden background Task is
introduced by this slice.

## Published product revision

Commit:

~~~text
c5bce4ef03417cce96b99924a0ae58e7cb95e7bb
LIB015-A: add structured logging levels, events, and Logger core
~~~

Published implementation version:

~~~text
0.3.247-SNAPSHOT
~~~

The commit is present at current `guillermomolina/protos` main at evidence
preparation time.

## Changed paths

The published product commit contains these paths:

- `CHANGELOG.md`
- `pom.xml`
- `protos/lib/logging/Level.protos`
- `protos/lib/logging/LogEvent.protos`
- `protos/lib/logging/Logger.protos`
- `protos/tests/library/logging/events.protos`
- `protos/tests/library/logging/levels.protos`
- `protos/tests/library/logging/logger.protos`
- `protos/tools/test/RepositoryCorpusPlans.protos`
- `protos/tools/test/RepositorySuite.protos`
- `src/main/java/com/guillermomolina/protos/cli/ProtosCli.java`
- `src/main/java/com/guillermomolina/protos/cli/ProtosTestCorpusRegistry.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolCorpusRegistryTest.java`
- `src/test/java/com/guillermomolina/protos/cli/ProtosTestToolFileSelectionWiringTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolSuiteGraphTest.java`
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolTool011ProgressGroupingTest.java`

The executable/library delta is deliberately accompanied by the required current
implementation version/changelog finalization and by Test Tool corpus registration
for the new suite-native logging corpus.

## Level model

`std:logging/Level` publishes exactly these canonical String level values:

~~~text
TRACE
DEBUG
INFO
WARN
ERROR
~~~

Severity order is exposed through `Level.compare(left, right)`.

There is no public numeric severity rank and no `FATAL` level. Invalid level
arguments signal ordinary `Error`.

## Event model

`LogEvent(level, message, fields, error)` creates one frozen event carrying:

~~~text
level
message
fields
error
~~~

There is intentionally no timestamp in LIB015-A.

The structured field domain is ordinary Protos data:

~~~text
null
Boolean
String
Integer
Float
Array
normal Map with String keys
~~~

Arrays and Maps are recursively copied and frozen so later mutation of caller
data cannot change an already-materialized event. Cyclic structured data and
unsupported values signal `Error`; they are not converted to String.

The attached Core `Error` is retained by identity and is neither copied,
mutated, signalled nor stringified.

## Logger model

`Logger(minimumLevel, sink)` creates an explicit frozen Logger.

The public core includes:

~~~text
isEnabled(level)
log(level, message, ...)
trace(...)
debug(...)
info(...)
warn(...)
error(...)
with(fields)
~~~

There is no default Logger, registry or mutable global logging configuration.

The supplied sink is the only output authority used by Logger.

## Filtering and pay-as-you-grow

A disabled Logger call validates its level first and then returns without:

- copying structured fields;
- constructing a LogEvent; or
- invoking the sink.

Callers can use `isEnabled(level)` before constructing expensive diagnostic
data.

This preserves the LIB015-0 pay-as-you-grow requirement for disabled DEBUG/TRACE
paths.

## Context model

`logger.with(fields)` creates a fresh derived frozen Logger.

Context precedence is:

~~~text
base context
    < derived context
    < call-site fields
~~~

The original Logger is not mutated.

Derived context uses the same structured-data snapshot rules as LogEvent.

No thread-local, dynamic, Task-local or Process-global Logger context is
introduced.

## Sink and failure boundary

The core sink contract is intentionally minimal:

~~~text
sink.emit(event)
~~~

Enabled events are sent synchronously.

An ordinary Core `Error` signalled by `emit(event)` is contained by the
normal Logger call: it is not propagated to the caller, retried or redirected
through an ambient fallback.

The slice does not add:

- stdout/stderr discovery;
- filesystem/network authority;
- fallback logging;
- buffering;
- async Tasks/threads;
- queues;
- flush/close lifecycle;
- strict sink mode.

Those concerns remain later explicit layers.

## Tests

The commit adds the suite-native logging corpus:

- `protos/tests/library/logging/levels.protos`;
- `protos/tests/library/logging/events.protos`;
- `protos/tests/library/logging/logger.protos`.

It also registers `protos/library/logging` through the current Test Tool corpus
and repository-suite infrastructure and updates the associated Java guards.

The project owner reports after publication:

~~~text
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No separate `git diff --check` result was reported in this LIB015-A completion
handoff, so this evidence does not fabricate one.

## Specification and architecture impact

~~~text
SPECIFICATION_CHANGED=NO
LIB015_0_ARCHITECTURE_REOPENED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
~~~

The implementation remains within the owner-ratified LIB015-0 architecture.

## Dependency update

Between LIB015-0 ratification and LIB015-A completion, LIB013-C was published.

Therefore `std:datetime/Instant` is now available in Protos.

This removes the implementation dependency previously expected for LIB015-D.

During post-publication reconciliation of LIB015-A, the next plain-format/sink
slice exposed one still-unratified observable contract: standard TextWriter
writes return Futures while LIB015-A's sink boundary is synchronous, and the
exact deterministic human-text representation was not fixed by LIB015-0.
Therefore a focused LIB015-B0 research gate now precedes public LIB015-B
implementation. This does not reopen the LIB015-0 core architecture.

## Deferred surfaces

LIB015-A intentionally leaves these outside the slice:

~~~text
plain human formatter         -> LIB015-B after LIB015-B0 contract closure
TextWriter-backed sink        -> LIB015-B after LIB015-B0 contract closure
MemorySink                    -> LIB015-B
additional sink composition   -> LIB015-B where justified by the ratified scope
JSON formatter/projection     -> LIB015-C
Instant timestamp integration -> LIB015-D
terminal StyledText/color     -> separately owned reusable capability + adapter
async/buffering/exporters     -> optional future extensions
~~~

## Closure state

~~~text
LIB015_A_STATUS=COMPLETE
PRODUCT_REVISION=c5bce4ef03417cce96b99924a0ae58e7cb95e7bb
IMPLEMENTATION_VERSION=0.3.247-SNAPSHOT

LEVEL_MODEL=TRACE_DEBUG_INFO_WARN_ERROR
EVENT_MODEL=FROZEN_STRUCTURED_DEEP_SNAPSHOT
LOGGER_MODEL=EXPLICIT_FROZEN_DERIVED_LOGGER
CONTEXT_MODEL=BASE_DERIVED_CALLSITE_PRECEDENCE
FILTERING_MODEL=PRE_EVENT_MINIMUM_LEVEL
SINK_BOUNDARY=EXPLICIT_EMIT_EVENT
SINK_FAILURE_POLICY=ORDINARY_ERROR_CONTAINED

TIMESTAMP_IMPLEMENTED=NO
JSON_IMPLEMENTED=NO
COLOR_IMPLEMENTED=NO

SPECIFICATION_CHANGED=NO

GIT_DIFF_CHECK=NOT_REPORTED_IN_COMPLETION_HANDOFF
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB015-B0
NEXT_SLICE_NAME=Plain formatting and TextWriter sink contract closure
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=
~~~
