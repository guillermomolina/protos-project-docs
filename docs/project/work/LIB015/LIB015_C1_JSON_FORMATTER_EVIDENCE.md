# LIB015-C1 — JSON formatter implementation evidence

Status: **COMPLETE**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-C1 — JSON formatter over std:json`

Nature: implementation evidence; **non-normative**

Product revision: `ddf59b1b3f5c97758354688088845d641f7a4e04`

Implementation version: `0.3.257-SNAPSHOT`

LIB015-0 ratification record:
`docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`

LIB015-C0 contract decision:
`docs/project/work/LIB015/LIB015_C0_JSON_EVENT_FORMATTER_CONTRACT_DECISION.md`

## Outcome

LIB015-C1 publishes the owner-ratified JSON logging formatter over `std:json`.

Published commit:

~~~text
ddf59b1b3f5c97758354688088845d641f7a4e04
LIB015-C1: add JSON formatter over std:json
~~~

At evidence preparation time that commit is the current `guillermomolina/protos`
`main` revision.

The published version is:

~~~text
0.3.257-SNAPSHOT
~~~

## Changed paths

The exact product commit changes:

- `CHANGELOG.md`;
- `pom.xml`;
- `protos/lib/logging/JsonFormatter.protos`;
- `protos/tests/library/logging/json-formatter.protos`;
- `protos/tools/test/RepositoryCorpusPlans.protos`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java`;
- `src/main/java/com/guillermomolina/protos/execution/ProtosLoggingFacility.java`;
- `src/test/java/com/guillermomolina/protos/execution/ProtosLoggingFacilityTest.java`; and
- `src/test/java/com/guillermomolina/protos/execution/ProtosTestToolRepositoryCorpusPlansTest.java`.

## JsonFormatter

`std:logging/JsonFormatter.format(event)` now renders one compact JSON object
with no trailing line terminator:

~~~text
{"level":...,"message":...,"fields":{...}[,"error":true]}
~~~

The top-level member order is fixed as `level`, `message`, `fields`, then optional
`error`.

`fields` is always present and remains nested, including when empty. User fields
named `level`, `message`, `fields`, `error`, or `timestamp` therefore do not
collide with logging metadata.

An attached Error is represented only by the top-level presence marker
`"error":true`; the Error is not stringified or otherwise inspected.

## std:json ownership and structured projection

The implementation constructs `std:json` nodes and delegates final serialization
to `JSON.encode`; it does not introduce a logging-private JSON serializer.

The published projection preserves the ratified C0 rules:

- `null`, Boolean, and String map to their corresponding JSON nodes;
- Integer maps exactly through `JSON.number(value, 0)` with no artificial range
  restriction;
- Arrays preserve order;
- every Map is emitted in ascending lexicographic Unicode-scalar key order at
  every depth;
- finite non-zero Float uses the same shortest-round-trip binary64 decimal
  semantics already used by `TextFormatter`;
- Float `+0.0` and `-0.0` both project to JSON numeric zero; and
- `NaN`, `Infinity`, and `-Infinity` signal an Error rather than producing
  non-standard JSON or a surrogate value.

String escaping and final numeric spelling remain owned by `std:json`.

## Reuse of existing logging boundaries

No `JsonSink` is introduced.

`JsonFormatter` composes with the already-published `TextSink`, so operational
line-oriented output remains:

~~~text
TextSink(JsonFormatter, writer)
    -> writer.writeLine(JsonFormatter.format(event)).value()
~~~

The existing writer ownership, Future observation, suspension/backpressure,
flush/close, and Logger sink-Error containment rules remain unchanged.

The private logging numeric facility is extended only so `JsonFormatter` can
reuse the existing shortest-decimal operation. The implementation does not
create a new public numeric/JVM contract, and the published `TextFormatter`
output remains unchanged.

## Tests and corpus

The `library/logging` corpus expands from six to seven files and adds:

`protos/tests/library/logging/json-formatter.protos`

Focused coverage includes:

- basic and empty-field JSON shape;
- optional Error presence marker;
- reserved-looking field names remaining nested;
- recursive Array/Map projection;
- Array order preservation;
- recursive Unicode-scalar Map ordering;
- String escaping through `std:json`;
- exact large Integers;
- finite Float shortest-round-trip cases;
- subnormal Float coverage;
- canonicalization of both signed Float zeros to JSON `0`;
- failure for NaN and both infinities;
- invalid-event rejection;
- formatter no-terminator behavior;
- `TextSink` framing/lifecycle composition; and
- Logger containment of formatter/sink Error.

The runtime provisioning and repository corpus plan have independent Java
regression coverage in the modified execution tests.

## Maintainer-reported validation

After publication, the project owner reports:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No additional test execution is inferred beyond that report.

## Specification, architecture and license

The product changelog records that this slice changes no specification.

~~~text
SPECIFICATION_CHANGED=NO
LIB015_0_ARCHITECTURE_REOPENED=NO
LIB015_C0_CONTRACT_REOPENED=NO
STD_JSON_PRIVATE_SERIALIZER_ADDED=NO
JSON_SINK_ADDED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
~~~

The new Protos-owned source and test files carry the project Adaptive Public
License notice; modified source files retain the existing notice.

## Deferred surfaces remain deferred

LIB015-C1 does **not** implement:

- timestamp storage or current-time acquisition;
- the textual or JSON representation of an optional timestamp;
- a logging Clock abstraction;
- ANSI/color or styled terminal output;
- FilteringSink;
- FanOutSink;
- buffering/async queues;
- global logger configuration; or
- ambient output discovery.

## Next-slice reconciliation

LIB015-0 already selected optional `std:datetime/Instant` timestamp integration as
the next logging phase once Instant existed. `std:datetime/Instant` and explicit
temporal text profiles are now published, but the exact observable logging
contract is still not closed.

Current `LogEvent` has exactly the local slots `level`, `message`, `fields`, and
`error`; current Logger explicitly reads no clock; current plain and JSON
formatters emit no timestamp. Implementing a timestamp directly would therefore
have to invent observable API/formatting behavior.

A focused contract closure is required before implementation. It must settle at
least:

1. exact `LogEvent` timestamp state and constructor compatibility;
2. whether absence is represented by `null` and whether recognition requires
   `std:datetime/Instant` when present;
3. how explicit current-time authority is supplied to Logger without ambient
   clock access and without paying timestamp cost for disabled levels;
4. whether callers may also construct events with explicit historical Instants;
5. plain-text timestamp placement and exact temporal text profile;
6. JSON timestamp member name, position, omission policy, and representation;
7. formatter failure/range behavior when an Instant cannot be rendered by the
   chosen profile;
8. preservation of `logger.with(fields)` and existing sink/error semantics; and
9. compatibility/migration of existing four-argument `LogEvent` and two-argument
   `Logger` construction.

The next slice is consequently:

~~~text
NEXT_SLICE=LIB015-D0
NEXT_SLICE_NAME=Timestamp and explicit clock-authority contract closure
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=
IMPLEMENTATION_EXECUTED=NO
~~~

D0 should reuse the ratified LIB015 architecture and the published LIB013
`Instant`/text profiles rather than repeating either broad design investigation.

## Final machine-readable state

~~~text
LIB015_C1_STATUS=COMPLETE
PRODUCT_REVISION=ddf59b1b3f5c97758354688088845d641f7a4e04
IMPLEMENTATION_VERSION=0.3.257-SNAPSHOT

JSON_FORMATTER_IMPLEMENTED=YES
JSON_USES_STD_JSON=YES
JSON_PRIVATE_SERIALIZER=NO
JSON_FIELDS_NESTED=YES
JSON_EMPTY_FIELDS_ALWAYS_PRESENT=YES
JSON_ERROR_PRESENCE_ONLY=YES
JSON_INTEGER_EXACT=YES
JSON_FINITE_FLOAT_SHORTEST_ROUNDTRIP=YES
JSON_NEGATIVE_ZERO_PRESERVED=NO
JSON_NONFINITE_FLOATS_FAIL=YES
JSON_MAP_ORDERING_UNICODE_SCALAR=YES
JSON_STRING_ESCAPING_OWNER=STD_JSON
JSON_REUSES_TEXT_SINK=YES
JSON_SINK_IMPLEMENTED=NO
TIMESTAMP_IMPLEMENTED=NO
COLOR_IMPLEMENTED=NO

PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB015-D0
NEXT_SLICE_TYPE=INVESTIGATION
NEXT_SLICE_REPOSITORY=
~~~
