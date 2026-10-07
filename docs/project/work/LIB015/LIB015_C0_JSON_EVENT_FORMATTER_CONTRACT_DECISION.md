# LIB015-C0 — JSON event projection and formatter contract decision

Status: **RATIFIED**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-C0 — JSON event projection and formatter contract closure`

Nature: focused research and owner-approved Standard Library contract decision; **non-normative**

LIB015-0 ratification record:
`docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`

LIB015-B0 ratification record:
`docs/project/work/LIB015/LIB015_B0_PLAIN_FORMATTING_TEXTWRITER_CONTRACT_DECISION.md`

LIB015-B1 implementation evidence:
`docs/project/work/LIB015/LIB015_B1_PLAIN_FORMATTER_EXPLICIT_SINKS_EVIDENCE.md`

Research and approval Protos baseline:
`5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407`

## Approval provenance

The project owner explicitly approved the completed LIB015-C0 recommendation on
2026-10-07 in the active LIB015 interaction with:

> ok apruebo la recomendación para LIB015-C.

This approval selects the exact C0 recommendation summarized below. It closes
the JSON formatter contract gate and releases LIB015-C1 implementation. It does
not close the parent LIB015 Issue.

## Purpose

LIB015-0 already selected `std:json` as the serialization owner for structured
logging JSON, and LIB015-B1 published the validated `LogEvent`, deterministic
plain formatter, explicit `TextSink`, and `MemorySink` baseline.

C0 resolves the remaining observable JSON questions before implementation:

- the top-level JSON event schema;
- nesting versus flattening of structured fields;
- optional-member policy;
- attached Error projection without arbitrary Error behavior;
- exact Integer and Float projection, including non-finite values and signed
  zero;
- deterministic recursive member ordering;
- String escaping ownership;
- formatter versus sink/framing ownership;
- failure behavior; and
- compatibility with later `std:datetime/Instant` timestamp integration.

No implementation is contained in this record.

## HEAD reconciliation

The focused research inspected current Protos state at
`5c1fd0dc1b0aa0bb3a7719ba6c0e3f5d0b759407`, including the published LIB015-B1
logging implementation and the current `std:json` node/encoder behavior.

At owner approval and publication preparation, `protos/main` remains at the same
revision. No reconciliation delta exists.

~~~text
HEAD_RECONCILIATION=PASS
RESEARCH_DECISION_STILL_APPLIES=YES
~~~

## Current-library findings

The current `std:json` API already supplies the required ownership boundary:

- Strings are passed as semantic Strings to `JSON.string`; `std:json` owns JSON
  escaping.
- Exact decimal numbers are represented through `JSON.number(coefficient,
  exponent)`.
- Arrays preserve element order.
- Objects retain caller construction order through compact encoding, so the
  logging formatter can impose deterministic ordering without changing
  `std:json`.
- The JSON number model has no representation for negative zero distinct from
  zero.

The current logging implementation already owns a deterministic shortest
round-trip binary64 decomposition for finite non-zero Float rendering in the
plain formatter. C1 may reuse/refactor that internal numeric mechanism, but it
must not expose JVM-specific representation as public logging semantics.

No `std:json` change is required by the ratified C0 contract.

## Focused external evidence

The C0 audit compared the current Protos boundary with materially relevant
precedents and standards rather than repeating LIB015-0's broad logging survey.

Material findings:

- RFC 8259 requires interoperable JSON syntax and does not admit NaN or Infinity
  as JSON numbers.
- RFC 8785 / JCS demonstrates deterministic JSON serialization, but its complete
  contract is not adopted because its binary64-only number model, UTF-16
  property ordering, and negative-zero behavior do not match Protos requirements.
- Go JSON and structured-logging precedents reinforce failing rather than
  emitting invalid bare non-finite numeric tokens.
- Rust `tracing` JSON output provides precedent for keeping structured event
  fields under a dedicated namespace instead of forcing them into reserved
  top-level names.
- Pino/Bunyan-style flattened event objects demonstrate the convenience of flat
  fields but also the reserved-name/collision problem that Protos should avoid.
- structlog demonstrates that deterministic key ordering can be formatter policy
  independent of event storage semantics.
- OpenTelemetry's distinction between event/log metadata and attributes supports
  keeping user structured fields in their own namespace.

## Ratified top-level schema

The baseline JSON representation is one compact JSON object:

~~~text
{
  "level": <String>,
  "message": <String>,
  "fields": <Object>,
  ["error": true]
}
~~~

Exact policy:

- `level` is always present.
- `message` is always present.
- `fields` is always present; an event with no fields uses `{}`.
- `error` is omitted when no Error is attached.
- `error` is exactly JSON Boolean `true` when an Error is attached.
- no timestamp member is emitted by C1.

Top-level member construction order is exactly:

~~~text
level
message
fields
error   # only when present
~~~

## Structured fields remain nested

Event fields are placed under the top-level `fields` object. They are not
flattened into the event object.

This preserves the complete already-approved field-key domain: a user field may
legitimately be named `level`, `message`, `fields`, `error`, or a future
`timestamp` without colliding with logging metadata.

Flattening is therefore rejected for the baseline contract.

~~~text
JSON_FIELDS_POLICY=NESTED
~~~

## Attached Error projection

C0 preserves the LIB015-B0/B1 rule that formatting must not execute arbitrary
behavior on an attached Core Error.

JSON records Error presence only:

~~~text
"error": true
~~~

The formatter must not stringify the Error, inspect JVM/host class names, obtain
an arbitrary stack trace, or invoke guest callbacks merely to enrich JSON.

~~~text
JSON_ERROR_POLICY=PRESENCE_BOOLEAN_TRUE_OMIT_WHEN_ABSENT
~~~

## Recursive structured-value projection

The already-approved structured LogEvent value domain maps recursively to
`std:json` nodes as follows:

~~~text
null       -> JSON.nullValue
Boolean    -> JSON.boolean(value)
String     -> JSON.string(value)
Integer    -> JSON.number(value, 0)
Array      -> JSON.array(project each element in order)
Map        -> JSON.object(project members in deterministic sorted-key order)
Float      -> policy below
~~~

No arbitrary object conversion or universal serializer hierarchy is introduced.

## Integer policy

Every Protos Integer is projected exactly as a JSON number with coefficient equal
to the Integer and exponent zero.

~~~text
Integer n -> JSON.number(n, 0)
~~~

C0 deliberately imposes no artificial binary64, signed-64-bit, or JavaScript
safe-integer restriction. Downstream consumers remain responsible for selecting
a JSON implementation capable of preserving numbers their application requires.

~~~text
JSON_INTEGER_PROJECTION=EXACT_INTEGER_AS_JSON_NUMBER_COEFFICIENT_VALUE_EXPONENT_0
~~~

## Finite Float policy

For a finite non-zero Float, the formatter uses the same semantic numeric policy
already ratified and implemented for the plain formatter: deterministic shortest
round-trip binary64 decimal decomposition.

The result is supplied to `JSON.number(signedCoefficient, exponent)` and encoded
by `std:json`.

Examples of semantic projection include:

~~~text
1.0     -> coefficient 1, exponent 0
1.23    -> coefficient 123, exponent -2
5e-324  -> coefficient 5, exponent -324
~~~

The exact textual orthography remains owned by `std:json`, not by a private
logging serializer.

## Signed zero policy

Both Float `+0.0` and Float `-0.0` project to JSON numeric zero:

~~~text
+0.0 -> JSON.number(0, 0)
-0.0 -> JSON.number(0, 0)
~~~

Negative-zero identity is therefore intentionally not preserved in baseline JSON
logging.

This is an explicit, owner-approved loss relative to plain text. The current
`std:json` exact-decimal number node has no signed-zero representation, and C0
does not expand that public library solely for logging.

If a future independently justified `std:json` evolution gains an explicit
signed-zero representation, LIB015 may reconsider this point through the normal
contract process.

~~~text
JSON_NEGATIVE_ZERO_PRESERVED=NO
~~~

## Non-finite Float policy

NaN, positive Infinity, and negative Infinity cannot be represented as baseline
JSON numbers.

C0 selects strict formatter failure:

~~~text
NaN       -> Error
Infinity  -> Error
-Infinity -> Error
~~~

Rejected alternatives include quoted strings, `null`, tagged objects, and
invalid bare non-finite tokens.

The formatter must construct/project the event before encoding/writing so a
non-finite value cannot produce partially written JSON output.

A future explicitly named alternate formatter/policy may choose a tagged
representation if a real telemetry use case requires it; baseline `JsonFormatter`
does not.

~~~text
JSON_NONFINITE_FLOAT_POLICY=FORMATTER_ERROR
~~~

## Deterministic recursive ordering

`LogEvent.fields` and nested Maps do not acquire semantic ordering.

`JsonFormatter` imposes deterministic output order for every Map recursively:

~~~text
ASCENDING_LEXICOGRAPHIC_UNICODE_SCALAR_SEQUENCE
~~~

This is the same ordering rule selected by the plain formatter.

Arrays retain their original order. Object construction supplies the desired
member sequence to current `std:json`, which then owns encoding.

C0 does not adopt JCS UTF-16 property ordering.

~~~text
JSON_BYTE_OUTPUT_DETERMINISTIC=YES
~~~

## String escaping ownership

The logging formatter passes raw semantic Strings to `JSON.string`.

`std:json` exclusively owns quote escaping, backslash escaping, control-character
escaping and final JSON String syntax.

Logging must not pre-escape Strings, normalize Unicode, inject ANSI sequences, or
maintain a second JSON String encoder.

~~~text
JSON_STRING_ESCAPING_OWNER=STD_JSON
ANSI_PRESENT_IN_JSON=NO
~~~

## Formatter and sink/framing boundary

`JsonFormatter.format(event)` returns one compact JSON String with no trailing
line terminator.

C1 reuses the existing explicit `TextSink(formatter, writer)`:

~~~text
JsonFormatter.format(event)
    -> compact unterminated JSON text

TextSink(JsonFormatter, writer).emit(event)
    -> writer.writeLine(jsonText).value()
~~~

This yields one JSON value per physical line when used with `TextSink`, suitable
for line-oriented/NDJSON-style operational output, without introducing a
logging-private `JsonSink`.

The writer remains borrowed and existing TextSink lifecycle/backpressure/failure
semantics remain unchanged.

~~~text
JSON_FORMATTER_RETURNS_UNTERMINATED_TEXT=YES
JSON_REUSES_TEXT_SINK=YES
JSON_OUTPUT_LINE_ORIENTED=YES
~~~

## Failure semantics

Direct formatting fails with Error when:

- the supplied value is not a recognized valid LogEvent;
- any recursively projected Float is non-finite; or
- `std:json` itself rejects the projected node/encoding operation.

There is no fallback to `null`, `{}`, a replacement String, or
`"<unserializable>"`.

Direct `TextSink.emit` exposes formatter/write Error to its caller, as already
ratified for B1. Normal Logger calls retain the already-published LIB015-A/B1
containment behavior for ordinary sink Error.

## Future timestamp compatibility

LIB015-D may later add an explicit optional `std:datetime/Instant` timestamp.
C0 deliberately does not select its exact JSON representation.

The nested-fields schema leaves a clean future top-level metadata slot, while a
user field named `timestamp` remains unambiguous inside `fields`.

C1 must not invent a placeholder timestamp or read current time implicitly.

## Rejected baseline alternatives

The focused scorecard favored nested strict JSON projection over the principal
alternatives:

- flattened fields: rejected because reserved metadata names collide with valid
  user fields;
- private logging JSON serializer: rejected because LIB015-0 explicitly selects
  `std:json` ownership;
- full JCS contract: rejected because its number and ordering constraints do not
  match Protos;
- non-finite Strings/null/tagged values: rejected for baseline because they
  silently change numeric meaning or add an unneeded parallel data schema;
- arbitrary Error serialization: rejected because it would reopen Error
  semantics and portability constraints; and
- dedicated JsonSink: rejected because the existing formatter/TextSink boundary
  already supplies the required framing and authority model.

The strongest cost of the selected contract is that an otherwise valid LogEvent
containing a non-finite Float cannot be emitted by baseline JSON formatting.
The explicit escape path is a future alternate formatter/policy rather than
weakening strict baseline JSON.

A second known cost is canonicalization of Float `-0.0` to JSON `0`; this is
accepted rather than expanding `std:json` solely for logging.

## LIB015-C1 implementation release

C1 is authorized to implement exactly this ratified contract in
`guillermomolina/protos`.

Expected bounded implementation decomposition:

1. add public `std:logging/JsonFormatter`;
2. validate/recognize the supplied LogEvent using the published B1 boundary;
3. recursively project the approved structured-value domain into `std:json`
   nodes;
4. project Integer exactly through `JSON.number(value, 0)`;
5. reuse/refactor the existing shortest-round-trip finite non-zero Float numeric
   mechanism without creating public JVM-dependent semantics;
6. canonicalize both Float zeros to JSON zero;
7. reject all non-finite Floats before encoding/writing;
8. sort every Map recursively by ascending Unicode scalar sequence;
9. construct top-level JSON members in `level`, `message`, `fields`, optional
   `error` order;
10. call `JSON.encode` once for final serialization;
11. reuse existing `TextSink`; do not add JsonSink merely for framing;
12. add focused positive, ordering, escaping, nesting, exact-Integer, Float edge,
   invalid-event, direct-failure and Logger-containment tests; and
13. preserve timestamp/color work for their separately owned slices.

A small internal numeric-facility refactor is allowed when required to share the
already-published shortest-round-trip behavior. It must remain implementation
machinery and must not create a new public logging/JVM number contract.

No new Dxxx or PLATxxx is required by this ratified contract.

## Machine-readable closure

~~~text
LIB015_C0_STATUS=RATIFIED
LIB015_C0_RESEARCH_COMPLETE=YES
DECISION_APPROVAL_PROVENANCE=PASS
HEAD_RECONCILIATION=PASS

JSON_FORMATTER_SCHEMA_CLOSED=YES
JSON_TOP_LEVEL_SCHEMA=OBJECT(level:String,message:String,fields:Object[,error:true])
JSON_FIELDS_POLICY=NESTED
JSON_EMPTY_FIELDS_POLICY=ALWAYS_PRESENT_EMPTY_OBJECT
JSON_ERROR_POLICY=PRESENCE_BOOLEAN_TRUE_OMIT_WHEN_ABSENT
JSON_INTEGER_PROJECTION=EXACT_INTEGER_AS_JSON_NUMBER_COEFFICIENT_VALUE_EXPONENT_0
JSON_FINITE_FLOAT_PROJECTION=SHORTEST_ROUNDTRIP_DECIMAL_FOR_FINITE_NONZERO;ZERO_CANONICALIZED_TO_JSON_ZERO
JSON_NEGATIVE_ZERO_PRESERVED=NO
JSON_NONFINITE_FLOAT_POLICY=FORMATTER_ERROR
JSON_BYTE_OUTPUT_DETERMINISTIC=YES
JSON_MEMBER_ORDERING=TOP_LEVEL_LEVEL_MESSAGE_FIELDS_ERROR_IF_PRESENT;MAP_KEYS_ASCENDING_LEXICOGRAPHIC_UNICODE_SCALAR_SEQUENCE_RECURSIVE
STD_JSON_CHANGE_REQUIRED=NO
JSON_STRING_ESCAPING_OWNER=STD_JSON
JSON_USES_STD_JSON=YES
JSON_PRIVATE_SERIALIZER=NO
JSON_FORMATTER_RETURNS_UNTERMINATED_TEXT=YES
JSON_REUSES_TEXT_SINK=YES
JSON_OUTPUT_LINE_ORIENTED=YES
NO_ARBITRARY_ERROR_BEHAVIOR=YES
NO_JVM_ERROR_SCHEMA=YES
ANSI_PRESENT_IN_JSON=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO

REQUIRED_DURABLE_PUBLICATION=PASS
PARENT_ISSUE_CLOSED=NO
NEXT_SLICE=LIB015-C1
NEXT_SLICE_NAME=JSON formatter over std:json
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~
