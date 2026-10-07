# LIB015-E1 — Colored human logging formatter evidence and LIB015 closure

Status: **COMPLETE**

Owning work item: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Slice: `LIB015-E1 — ColoredFormatter over reusable terminal styling`

Nature: implementation evidence and parent-closure record; **non-normative**

Ratified contract record:
`docs/project/work/LIB015/LIB015_E0_COLORED_HUMAN_LOGGING_ADAPTER_CONTRACT_DECISION.md`

Published Protos revision:
`d80536af4db3632d7268d370d216418c1c4fa533`

Published implementation version:
`0.3.271-SNAPSHOT`

## Publication evidence

The maintainer reported on 2026-10-07:

> LIB015-E1: add colored human logging formatter pushed.
> el git diff check esta limpio.
> Todos los tests han pasado en local

The published `protos/main` revision was re-read and verified as:

~~~text
d80536af4db3632d7268d370d216418c1c4fa533
LIB015-E1: add colored human logging formatter
~~~

Its changed paths are exactly:

- `protos/lib/logging/ColoredFormatter.protos`
- `protos/tests/library/logging/colored-formatter.protos`
- `pom.xml`
- `CHANGELOG.md`

The published version is `0.3.271-SNAPSHOT`.

## Implemented contract

The new public module is:

~~~text
std:logging/ColoredFormatter
~~~

Construction remains:

~~~protos
formatter: ColoredFormatter(mode, autoEnabled)
~~~

The constructor resolves `std:text/ColorMode.resolve(mode, autoEnabled)` once and
returns a fresh frozen formatter whose only public slot is `format`.

`format(event)` calls `std:logging/TextFormatter.format(event)` exactly once.
That existing formatter remains the sole owner of the canonical human logging
grammar.

When styling is disabled, the result is exactly the canonical plain line.
When styling is enabled, only the already-serialized LEVEL token is decorated
through the reusable `std:text/Style`, `std:text/StyledText`, and
`std:text/ANSI` facilities.

The fixed palette is:

~~~text
TRACE -> blue
DEBUG -> cyan
INFO  -> default
WARN  -> yellow
ERROR -> red
~~~

INFO therefore produces no ANSI even when styling is enabled.

The implementation does not consult environment, TTY, Process, stdout/stderr, or
other ambient terminal authority.

Existing `TextSink(formatter, writer)` is reused unchanged. No ColoredSink,
StyledTextSink, theme registry, Java/runtime support, new Dxxx, or new PLATxxx was
introduced.

## Published test evidence

The new suite-native
`protos/tests/library/logging/colored-formatter.protos` covers the E0 contract,
including:

- fresh/frozen formatter and minimal public surface;
- canonical and invalid ColorMode inputs;
- AUTO/ALWAYS/NEVER resolution matrix;
- exact TRACE/DEBUG/INFO/WARN/ERROR palette behavior;
- LEVEL-token-only styling;
- exact disabled parity with `TextFormatter`;
- enabled textual parity after removing only generated ANSI;
- INFO no-ANSI behavior;
- hostile ESC/control text remaining escaped data;
- propagation of existing TextFormatter failures;
- `TextSink` framing/Future/borrowed-writer behavior; and
- Logger containment of sink failures.

Maintainer-reported validation:

~~~text
PROTOS_GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
~~~

No test was reported after the final version/CHANGELOG publication boundary.

## LIB015 closure

LIB015 has now completed its ratified sequence:

~~~text
LIB015-0  logging architecture decision
LIB015-A  structured event / Logger core
LIB015-B0 plain formatter and TextWriter-sink contract
LIB015-B1 plain formatter / TextSink / MemorySink implementation
LIB015-C0 JSON formatter contract
LIB015-C1 JsonFormatter implementation
LIB015-D0 timestamp / explicit time-authority contract
LIB015-D1 timestamped event / Logger time-source integration
LIB015-E0 colored human logging adapter contract
LIB015-E1 ColoredFormatter implementation
~~~

The last remaining dependency from LIB015-0 — reusable terminal styling — was
provided by LIB020 before E1.

No remaining approved LIB015 implementation slice is known.

Deferred items such as asynchronous/buffered sinks, network exporters, global
logging configuration, theme registries, richer terminal styles, TTY/environment
detection, and alternate non-ANSI styled renderers remain future independently
justified work and do not block LIB015 closure.

## Closure machine record

~~~text
LIB015_E1_STATUS=COMPLETE
LIB015_PARENT_STATUS=COMPLETE

PROTOS_REVISION=d80536af4db3632d7268d370d216418c1c4fa533
IMPLEMENTATION_VERSION=0.3.271-SNAPSHOT

PUBLIC_MODULE=std:logging/ColoredFormatter
FORMAT_RESULT=String
COLOR_POLICY_RESOLVED_AT_CONSTRUCTION=YES

TRACE_STYLE=blue
DEBUG_STYLE=cyan
INFO_STYLE=default
WARN_STYLE=yellow
ERROR_STYLE=red
STYLED_SCOPE=LEVEL_TOKEN_ONLY

TEXTFORMATTER_IS_SINGLE_HUMAN_GRAMMAR_OWNER=YES
TEXTSINK_REUSED_UNCHANGED=YES
LOGGER_PUBLIC_CONTRACT_CHANGED=NO
LOGEVENT_CHANGED=NO
JSONFORMATTER_CHANGED=NO

AMBIENT_ENV_ACCESS=NO
AMBIENT_TTY_ACCESS=NO
THEME_CONFIGURABLE_INITIAL=NO

NEW_JAVA_RUNTIME=NO
NEW_Dxxx=NO
NEW_PLATxxx=NO

PROTOS_GIT_DIFF_CHECK=PASS
ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

NEXT_LIB015_SLICE=NONE
PARENT_CLOSURE_READY=YES
~~~
