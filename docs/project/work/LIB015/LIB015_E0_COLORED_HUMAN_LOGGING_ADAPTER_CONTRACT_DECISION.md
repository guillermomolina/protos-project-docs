# LIB015-E0 — Colored human logging adapter contract decision

Status: **RATIFIED**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-E0 — Colored human logging adapter contract closure`

Nature: focused research and owner-approved Standard Library contract decision; **non-normative**

Previous durable records:

- `docs/project/work/LIB015/LIB015_0_LOGGING_ARCHITECTURE_DECISION.md`
- `docs/project/work/LIB015/LIB015_B0_PLAIN_FORMATTING_TEXTWRITER_CONTRACT_DECISION.md`
- `docs/project/work/LIB015/LIB015_B1_PLAIN_FORMATTER_EXPLICIT_SINKS_EVIDENCE.md`
- `docs/project/work/LIB015/LIB015_C0_JSON_EVENT_FORMATTER_CONTRACT_DECISION.md`
- `docs/project/work/LIB015/LIB015_C1_JSON_FORMATTER_EVIDENCE.md`
- `docs/project/work/LIB015/LIB015_D0_TIMESTAMP_EXPLICIT_TIME_AUTHORITY_CONTRACT_DECISION.md`
- `docs/project/work/LIB015/LIB015_D1_TIMESTAMPED_LOGEVENT_TIME_SOURCE_EVIDENCE.md`
- `docs/project/work/LIB020/LIB020_C_COLORMODE_AND_CLOSURE_EVIDENCE.md`

Research and approval Protos baseline:
`246a24994d74ca084dc43ceddd4c400c2af94b15`

Approval provenance: GitHub Issue `#432`, comment `6034169830`.

## Approval provenance

The project owner explicitly approved the completed LIB015-E0 recommendation on
2026-10-07 in the active LIB015 interaction with:

> apruebo recomendación

That approval selects the exact E0 recommendation summarized below. It closes
the remaining colored-human-output public-contract gate and releases LIB015-E1
implementation. It does not close the parent LIB015 Issue.

## Purpose

LIB015 already publishes its structured event, Logger, plain formatter,
TextWriter sink, MemorySink, JSON formatter, and explicit timestamp/time-source
layers. LIB020 separately publishes the reusable text-styling foundation:

~~~text
std:text/Style
std:text/StyledText
std:text/ANSI
std:text/ColorMode
~~~

E0 closes the last observable contract needed to compose those existing
facilities into colored human logging without adding a logging-private ANSI
framework, a second human grammar, a new sink hierarchy, ambient terminal
authority, or a theme institution.

No implementation is contained in this record.

## HEAD reconciliation

The focused E0 research and owner approval were reconciled against:

~~~text
PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
PROTOS_HEAD_MESSAGE=LIB020-C: add pure std:text/ColorMode resolution policy
~~~

At that revision the complete LIB020 reusable foundation is present and Issue
`#432` is open and ready for its final implementation slice.

The research inspected the current logging and text modules and the complete
suite-native logging/text conformance surfaces relevant to the contract. No
current repository evidence requires reopening the recommendation.

~~~text
HEAD_RECONCILIATION=PASS
RESEARCH_DECISION_STILL_APPLIES=YES
~~~

## Ratified public API

The public module is:

~~~text
std:logging/ColoredFormatter
~~~

It is a factory of ordinary frozen formatter values:

~~~protos
formatter: ColoredFormatter(mode, autoEnabled)
text: formatter.format(event)
~~~

The public result contract is:

~~~text
formatter.format(event) -> String
~~~

The formatter does not expose `StyledText`, terminal capabilities, a writer, a
sink, a theme registry, environment access, or TTY detection.

Canonical composition remains:

~~~protos
ColorMode: import("std:text/ColorMode")
ColoredFormatter: import("std:logging/ColoredFormatter")
TextSink: import("std:logging/TextSink")
Logger: import("std:logging/Logger")
Level: import("std:logging/Level")

formatter: ColoredFormatter(ColorMode.AUTO, autoEnabled)
sink: TextSink(formatter, writer)
logger: Logger(Level.INFO, sink)
~~~

## Color policy ownership and lifetime

`mode` must be a canonical `std:text/ColorMode`, and `autoEnabled` must be
exactly Boolean.

Construction resolves the policy once:

~~~protos
stylingEnabled: ColorMode.resolve(mode, autoEnabled)
~~~

The resulting formatter captures that resolved Boolean for subsequent
`format(event)` calls.

This preserves the reusable ownership boundary:

- `std:text/ColorMode` owns AUTO/ALWAYS/NEVER semantics;
- the explicit caller/terminal boundary owns computation of `autoEnabled`;
- `std:logging/ColoredFormatter` only consumes those values;
- no formatter event path reads environment or terminal state.

A caller whose writer/capability situation changes may construct a new formatter.
No mutable global or formatter-wide dynamic policy state is introduced.

## Canonical human grammar ownership

`std:logging/TextFormatter.format(event)` remains the sole owner of canonical
human-record grammar and serialization.

`ColoredFormatter` must first obtain the canonical plain String from
`TextFormatter.format(event)`. It must not independently serialize timestamps,
messages, fields, arrays, maps, floats, nested values, or attached Error state.

The colored representation is then built only by splitting that already
canonical String around the exact level token, decorating that token, and
recombining the original textual fragments through `std:text/StyledText`.

Conceptually:

~~~text
plain = TextFormatter.format(event)

plain prefix
+ StyledText(level token, styleFor(event.level))
+ plain suffix
    -> StyledText.concat(...)
    -> ANSI.render(..., stylingEnabled)
    -> String
~~~

The existing text grammar makes the level span deterministic:

- without a timestamp, the level starts at offset zero;
- with a timestamp, the ISO8601 prefix contains no spaces and the level begins
  immediately after the first ASCII space;
- the level token length is the canonical level String length.

No message, field, map, float, or nested-value parsing is introduced.

## Exact parity requirements

Disabled styling is exactly identical to the existing formatter:

~~~text
ColoredFormatter(mode resolving false).format(event)
==
TextFormatter.format(event)
~~~

for every valid `LogEvent`, including all timestamp, escaping, nested data,
deterministic Map ordering, Float, and `error=true` cases.

Enabled rendering is decoration of that same text, not a second grammar:

~~~text
textual content after removing only ANSI generated by std:text/ANSI
==
TextFormatter.format(event)
~~~

Both parity properties are public requirements.

~~~text
PLAIN_PARITY_REQUIRED=YES
ENABLED_TEXT_PARITY_REQUIRED=YES
SECOND_HUMAN_SERIALIZER_ALLOWED=NO
~~~

## Styled scope

Only the exact canonical `LEVEL` token receives non-default style.

No adjacent spaces, timestamp, message, fields, or `error=true` marker are
included in the styled run.

This minimizes generated ANSI, preserves the plain grammar structurally, and
avoids inventing additional visual semantics for error attachment or payload
structure.

~~~text
STYLED_SCOPE=LEVEL_TOKEN_ONLY
~~~

## Initial fixed palette

The exact v1 palette is:

~~~text
TRACE -> blue
DEBUG -> cyan
INFO  -> default
WARN  -> yellow
ERROR -> red
~~~

The mapping uses only the foreground styles already published by LIB020.

INFO deliberately remains `default`, so ordinary informational lines produce no
generated ANSI even when styling is enabled.

TRACE and DEBUG remain visually distinguishable for callers that deliberately
enable those diagnostic levels.

~~~text
TRACE_STYLE=blue
DEBUG_STYLE=cyan
INFO_STYLE=default
WARN_STYLE=yellow
ERROR_STYLE=red
~~~

## Theme policy

There is no configurable theme in v1.

No theme registry, caller-supplied level-style Map, global palette, dynamic
theme, background color, bold/dim style, 256-color, or truecolor machinery is
introduced.

A future theme capability can be added additively because the current model
already separates:

~~~text
level -> Style
StyledText -> renderer
~~~

Such an extension need not change `LogEvent`, `Logger`, `TextFormatter`,
`TextSink`, `ColorMode`, `StyledText`, or `ANSI`.

~~~text
THEME_CONFIGURABLE_INITIAL=NO
THEME_EXTENSION_DEFERRED=YES
~~~

## TextSink and Logger reuse

The existing `TextSink(formatter, writer)` is reused unchanged.

Because the selected formatter still satisfies:

~~~text
formatter.format(event) -> String
~~~

the current sink continues to own:

- one-line framing through `writer.writeLine(...)`;
- waiting for the write Future with `.value()`;
- borrowed writer lifecycle;
- direct formatting/write Error propagation from `emit`; and
- normal Logger containment of sink Error.

No `StyledTextSink`, `ColoredSink`, writer wrapper hierarchy, or new writer
authority is required.

`Logger` remains unchanged.

## Error behavior

Construction may signal the ordinary Error already produced by
`ColorMode.resolve(mode, autoEnabled)` for an invalid mode or non-Boolean
`autoEnabled`.

`format(event)` may signal the ordinary Errors already produced by
`TextFormatter.format(event)`, including an unrecognized event or a timestamp
outside the civil range accepted by canonical ISO8601 human formatting.

Rendering uses the already-published `StyledText`, `Style`, and `ANSI`
contracts. No logging-specific color Error hierarchy or fallback representation
is introduced.

When used through `TextSink` and `Logger`, the already-published sink failure
semantics remain unchanged.

## JSON and structured event boundaries

The color adapter never participates in JSON formatting.

~~~text
LOGEVENT_CHANGED=NO
JSONFORMATTER_CHANGED=NO
ANSI_IN_LOGEVENT=NO
ANSI_IN_JSON=NO
~~~

`LogEvent` stays structured and ANSI-free. `JsonFormatter` stays outside the
colored path.

## No new runtime or platform work

The selected contract is implementable in ordinary Protos using the already
published Standard Library modules.

No Java/runtime support is required. No new Dxxx or PLATxxx is required.

~~~text
NEW_JAVA_RUNTIME_REQUIRED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
~~~

## Rejected alternatives

E0 compared four coherent architecture families:

1. a public `StyledFormatter` returning `StyledText`;
2. `StyledFormatter` plus a new `StyledTextSink`;
3. extending `TextFormatter` directly with color behavior; and
4. the selected `ColoredFormatter` returning String.

The first alternative is the strongest architectural counterargument because it
maximizes separation between semantic styling and rendering. It is not selected
for v1 because the existing public sink protocol consumes a String formatter;
selecting it now would require public glue or another sink abstraction solely to
recover behavior `TextSink` already provides.

The second adds both a formatter and sink institution without a current
requirement. The third overloads the module whose clear existing contract is the
canonical plain formatter.

The selected form reuses every existing boundary, adds one public logging module,
keeps color optional/pay-as-you-grow, and has the smallest public/runtime surface
that satisfies the current requirement.

## Strongest argument and regret analysis

The concrete regret scenario is a near-term requirement to emit the same
semantic styled log record through several non-ANSI renderers such as HTML, GUI,
or a richer terminal renderer.

The escape path is bounded and additive:

~~~text
future std:logging/StyledFormatter
    -> StyledText

ColoredFormatter
    -> delegates to StyledFormatter
    -> ANSI.render(...)
~~~

The existing `ColoredFormatter(...).format(event) -> String` contract need not
break.

Adding themes later is similarly bounded: introduce a new explicit palette/theme
input or formatter factory while retaining the fixed v1 mapping as the compatible
default.

Changing the default styled scope later is more compatibility-sensitive because
the exact generated ANSI is observable. Such a change should therefore use an
explicit option/new formatter behavior rather than silently changing the v1
default.

The primary divergence risk is maintaining two human serializers. The selected
contract avoids that risk by making `TextFormatter.format(event)` the only
canonical serializer.

## LIB015-E1 implementation release

LIB015-E1 is released to implement exactly this ratified contract in
`guillermomolina/protos`.

Expected bounded implementation:

1. add `protos/lib/logging/ColoredFormatter.protos`;
2. implement `ColoredFormatter(mode, autoEnabled)` as a factory of frozen
   formatter values;
3. resolve `ColorMode` once at construction;
4. call `TextFormatter.format(event)` exactly once per format operation;
5. identify only the canonical level span in that returned String;
6. build corresponding `StyledText` with only that token styled;
7. apply the fixed five-level palette;
8. call `ANSI.render(styledText, stylingEnabled)` and return its String;
9. leave `TextFormatter`, `TextSink`, `Logger`, `LogEvent`, and
   `JsonFormatter` public contracts unchanged;
10. add one semantically coherent suite-native logging test file covering the
    contract;
11. add source-local public API documentation using the established mechanism;
12. update Standard Library navigation/manifest surfaces only where current HEAD
    conventions require the new public module;
13. do not add Java/runtime code, a new sink, environment/TTY access, or a theme
    system; and
14. after successful E1 validation/publication, reconcile closure of LIB015/#432.

Focused E1 tests must cover at least:

- frozen formatter/public surface;
- invalid `ColorMode` and invalid `autoEnabled`;
- all AUTO/ALWAYS/NEVER resolution combinations;
- disabled exact parity with `TextFormatter` over representative timestamps,
  escaping, nested data, Map ordering, Float edge cases, and Error marker;
- exact TRACE/DEBUG/INFO/WARN/ERROR palette;
- level-token-only styling;
- INFO default producing no generated ANSI;
- enabled textual parity after removing only renderer-generated ANSI;
- escaped input ESC remaining log data rather than terminal styling;
- propagation of existing TextFormatter failure cases;
- composition with existing `TextSink`; and
- preservation of Logger sink-error containment.

## Publication validation

The prepared durable record plus work-index insertion were checked with
`git diff --check` before publication.

~~~text
DOCS_GIT_DIFF_CHECK=PASS
VALIDATION_PROVENANCE=AGENT_EXECUTED
~~~

## Machine-readable closure

~~~text
LIB015_E0_STATUS=RATIFIED
LIB015_E0_RESEARCH_COMPLETE=YES
DECISION_APPROVAL_PROVENANCE=PASS
APPROVAL_ISSUE_COMMENT=6034169830
HEAD_RECONCILIATION=PASS

PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
CURRENT_LIB020_FOUNDATION_AVAILABLE=YES

RECOMMENDED_PUBLIC_MODULE=std:logging/ColoredFormatter
RECOMMENDED_API=ColoredFormatter(mode,autoEnabled)->FROZEN_FORMATTER;FORMAT(event)->String
RECOMMENDED_FORMAT_RESULT=STRING
RECOMMENDED_STYLED_SCOPE=LEVEL_TOKEN_ONLY

TRACE_STYLE=blue
DEBUG_STYLE=cyan
INFO_STYLE=default
WARN_STYLE=yellow
ERROR_STYLE=red

THEME_CONFIGURABLE_INITIAL=NO

COLORMODE_OWNER=std:text/ColorMode
AUTO_ENABLED_OWNER=EXPLICIT_CALLER_TERMINAL_BOUNDARY
AUTO_ENABLED_LIFETIME=RESOLVED_ONCE_AT_FORMATTER_CONSTRUCTION

TEXTFORMATTER_PLAIN_BASELINE_PRESERVED=YES
TEXTFORMATTER_REMAINS_CANONICAL_HUMAN_SERIALIZER=YES
PLAIN_PARITY_REQUIRED=YES
ENABLED_TEXT_PARITY_REQUIRED=YES

TEXTSINK_REUSED_UNCHANGED=YES
TEXTFORMATTER_PUBLIC_CONTRACT_CHANGED=NO
TEXTSINK_PUBLIC_CONTRACT_CHANGED=NO
LOGEVENT_CHANGED=NO
JSONFORMATTER_CHANGED=NO
LOGGER_CHANGED=NO

AMBIENT_ENV_ACCESS=NO
AMBIENT_TTY_ACCESS=NO
NEW_JAVA_RUNTIME_REQUIRED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO

REQUIRED_DURABLE_PUBLICATION=PASS
PARENT_ISSUE_CLOSED=NO

NEXT_SLICE=LIB015-E1
NEXT_SLICE_NAME=ColoredFormatter over reusable terminal styling
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
PARENT_CLOSABLE_AFTER_E1_SUCCESS=YES
~~~
