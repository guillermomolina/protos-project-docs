# LIB015-B1 — Plain formatter and explicit sinks implementation evidence

Status: **COMPLETE**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-B1 — Plain formatter, TextWriter sink, MemorySink and LogEvent recognition hardening`

Nature: implementation evidence; **non-normative**

Product revision: `6252f3b6ddaffd2246e5f96eb2e746df8f1ff267`

Implementation version: `0.3.253-SNAPSHOT`

LIB015-0 ratification record:
`docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`

LIB015-B0 contract decision:
`docs/project/work/LIB015/LIB015_B0_PLAIN_FORMATTING_TEXTWRITER_CONTRACT_DECISION.md`

## Outcome

LIB015-B1 publishes the owner-ratified plain human logging layer and explicit
text/memory sinks.

Published commit:

~~~text
6252f3b6ddaffd2246e5f96eb2e746df8f1ff267
LIB015-B1: add plain formatter and explicit text sinks
~~~

The published version is:

~~~text
0.3.253-SNAPSHOT
~~~

At evidence preparation time this commit is the current
`guillermomolina/protos` main revision.

## Changed paths

The exact product commit changes:

- `CHANGELOG.md`;
- `pom.xml`;
- `protos/lib/logging/LogEvent.protos`;
- `protos/lib/logging/MemorySink.protos`;
- `protos/lib/logging/TextFormatter.protos`;
- `protos/lib/logging/TextSink.protos`;
- `protos/tests/library/logging/events.protos`;
- `protos/tests/library/logging/levels.protos`;
- `protos/tests/library/logging/logger.protos`;
- `protos/tests/library/logging/recognition.protos`;
- `protos/tests/library/logging/sinks.protos`;
- `protos/tests/library/logging/text-formatter.protos`;
- `protos/tools/test/RepositoryCorpusPlans.protos`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosLoggingFacility.java`;
- `src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java`;
- `src/test/java/com/guillermomolina/protos/execution/ProtosLoggingFacilityTest.java`; and
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java`.

## LogEvent hardening

`LogEvent` now uses `std:logging/LogEvent` as the immediate parent of successful
event values and publishes:

~~~text
LogEvent.recognizes(value)
~~~

Recognition accepts exactly frozen canonical event state and is backed by a
private runtime facility so candidate `parent`, reflection, collection
iteration, equality, hashing, or other ordinary behavior is not executed merely
to classify the candidate.

The same facility recognizes attached Core Error ancestry without sending
messages to the Error candidate.

No hidden public brand or new general type system is introduced.

The slice also fixes the LIB015-A nested-Array snapshot path: it now uses the
existing Array collection facility rather than the nonexistent Array `add`
operation.

## Plain TextFormatter

`std:logging/TextFormatter.format(event)` now renders one deterministic logical
line with no terminator, timestamp, source metadata, color or ANSI control
sequence.

The ratified grammar is implemented:

~~~text
LEVEL "message" {fields} error=true
~~~

with optional fields and Error marker.

Properties retained by the implementation include:

- double-quoted Strings;
- escaping of quote, backslash, LF, CR and TAB;
- deterministic escaping of remaining C0/C1 controls, DEL, U+2028 and U+2029;
- one physical line per formatted event;
- recursive Array/Map rendering;
- Unicode-scalar lexicographic Map-key ordering at every depth;
- canonical Integer rendering;
- deterministic shortest-round-trip Float rendering including `NaN`,
  `Infinity`, `-Infinity`, `0.0` and `-0.0`; and
- attached Error represented only by `error=true`, without inspecting the
  Error.

The new formatter invokes no arbitrary behavior stored in a recognized event.

## TextSink

`std:logging/TextSink(formatter, writer)` borrows the explicitly supplied
TextWriter.

Its emission boundary is exactly:

~~~text
writer.writeLine(formatter.format(event)).value()
~~~

Therefore:

- the sink, not the formatter, owns line framing;
- a pending write may suspend the caller under normal Future semantics;
- a failed write signals from direct `emit`;
- ordinary Logger calls retain LIB015-A's sink-Error containment boundary;
- no background Task, queue or buffer is created;
- no stdout/stderr/file/network destination is discovered;
- no per-event flush is introduced; and
- the sink never closes or takes ownership of the writer.

## MemorySink

`std:logging/MemorySink()` now retains exact recognized LogEvent identities in
emission order.

`events()` returns a fresh frozen Array snapshot. Previously returned snapshots
remain unchanged after later emissions.

The baseline deliberately has no clear operation, capacity, eviction, queue,
flush, close or drop policy.

## Tests and corpus

The logging corpus expands from three to six files and adds focused coverage for:

- safe LogEvent recognition;
- plain TextFormatter byte-for-byte behavior;
- String/control escaping;
- recursive structured values;
- deterministic key ordering;
- numeric edge cases;
- TextSink write/Future/failure/lifecycle behavior; and
- MemorySink ordering, identity and frozen snapshots.

The new suite-native files are:

- `protos/tests/library/logging/recognition.protos`;
- `protos/tests/library/logging/text-formatter.protos`; and
- `protos/tests/library/logging/sinks.protos`.

The runtime facility has independent Java regression coverage in
`ProtosLoggingFacilityTest` and the native-boundary architecture guard is updated.

## Maintainer-reported validation

After publication, the project owner reports:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No additional test run is inferred beyond that report.

## Specification, architecture and license

~~~text
SPECIFICATION_CHANGED=NO
LIB015_0_ARCHITECTURE_REOPENED=NO
LIB015_B0_CONTRACT_REOPENED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
~~~

The implementation changelog explicitly records no specification change.

The new Protos-owned source files and the new Java runtime facility carry the
project Adaptive Public License notice; modified source files retain the
existing notice.

## Deferred surfaces remain deferred

LIB015-B1 does **not** implement:

- JSON logging;
- timestamps/current-time authority;
- color or StyledText;
- FilteringSink;
- FanOutSink;
- buffering/async queues;
- global logger configuration; or
- ambient output discovery.

## Next-slice reconciliation

LIB015-0 selected the next architectural phase as a JSON adapter reusing
`std:json`. Current HEAD provides `std:json/JSON` node constructors and
`JSON.encode(root)`, and the logging event structured-value domain is now stable.

However, the ratified architecture does not yet fix several observable JSON
contracts that an implementation would otherwise have to invent:

1. the exact top-level JSON object/schema and whether structured fields are
   nested or flattened;
2. whether optional components are omitted or represented explicitly;
3. the representation of an attached Core Error without arbitrary
   stringification;
4. the policy for non-finite Protos Floats (`NaN`, `Infinity`, `-Infinity`),
   which JSON has no native numeric representation for;
5. the exact finite-Float projection into `std:json`'s exact decimal number
   model;
6. whether logging JSON promises byte-for-byte deterministic object-member
   ordering even though ordinary JSON object order is not semantic and
   `JSON.encode` currently consumes its node object's Map order; and
7. whether the JSON formatter returns compact unterminated text for reuse by
   the existing TextSink or requires a distinct public sink boundary.

These are public Standard Library behavior and therefore require a focused
contract closure before implementation rather than being guessed inside
LIB015-C.

The next slice is consequently:

~~~text
NEXT_SLICE=LIB015-C0
NEXT_SLICE_NAME=JSON event projection and formatter contract closure
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=
IMPLEMENTATION_EXECUTED=NO
~~~

The C0 investigation should be narrow: reuse LIB015-0/B0 and current
`std:json`; do not repeat the broad logging architecture survey.

## Final machine-readable state

~~~text
LIB015_B1_STATUS=COMPLETE
PRODUCT_REVISION=6252f3b6ddaffd2246e5f96eb2e746df8f1ff267
IMPLEMENTATION_VERSION=0.3.253-SNAPSHOT

LOGEVENT_CANONICAL_PARENT=YES
LOGEVENT_PUBLIC_RECOGNIZES=YES
LOGEVENT_RECOGNITION_NO_ARBITRARY_CANDIDATE_BEHAVIOR=YES

PLAIN_TEXT_FORMATTER_IMPLEMENTED=YES
PLAIN_OUTPUT_SINGLE_LINE=YES
PLAIN_OUTPUT_CONTAINS_ANSI=NO
STRUCTURED_VALUE_RENDERING_DETERMINISTIC=YES
MAP_ORDERING_UNICODE_SCALAR=YES
ERROR_RENDERING_PRESENCE_ONLY=YES

TEXT_SINK_IMPLEMENTED=YES
TEXT_SINK_BORROWS_WRITER=YES
TEXT_SINK_USES_WRITELINE=YES
TEXT_SINK_AWAITS_WRITE_FUTURE=YES
TEXT_SINK_FLUSHES_PER_EVENT=NO
TEXT_SINK_CLOSES_WRITER=NO

MEMORY_SINK_IMPLEMENTED=YES
MEMORY_SINK_PRESERVES_EVENT_IDENTITY=YES
MEMORY_SINK_EVENTS_RETURNS_FROZEN_SNAPSHOT=YES

FILTERING_SINK_IMPLEMENTED=NO
FANOUT_SINK_IMPLEMENTED=NO
JSON_IMPLEMENTED=NO
TIMESTAMP_IMPLEMENTED=NO
COLOR_IMPLEMENTED=NO

PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

NEXT_SLICE=LIB015-C0
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=
~~~
