# LIB020-B — Pure deterministic ANSI renderer evidence

Status: **COMPLETE**

Work authority: `guillermomolina/protos#823` — `LIB020 — Reusable terminal styling and color policy`

Ratified design authorities:

- D190 / #826 — reusable styled-text and explicit color-policy architecture.
- D191 / #827 — exact Standard Library module and public API surface.

Published product revision:

```text
PROTOS_REVISION=b10569680b1532b277ba3ce49eced2e818833d93
PROTOS_COMMIT=LIB020-B: add pure deterministic std:text/ANSI renderer
IMPLEMENTATION_VERSION=0.3.268-SNAPSHOT
```

Validation provenance is maintainer-reported:

```text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

## Published surface

LIB020-B publishes exactly:

```text
std:text/ANSI
```

with one public operation:

```text
ANSI.render(styledText, stylingEnabled)
```

The module exposes no terminal capability, writer, environment detection, global configuration, color-policy detection, theme, markup, or stateful renderer object.

`std:text/ColorMode` remains deferred to LIB020-C.

## Pure rendering contract

`ANSI.render` accepts:

- one exact-family `std:text/StyledText` value; and
- one exact canonical Boolean `true` or `false`.

Invalid inputs signal ordinary Error. StyledText validation continues to use the runtime-safe exact-family recognition published by LIB020-A; no candidate reflection or behavior is invoked to decide family membership.

Rendering is a pure transformation to ordinary String and performs no:

```text
environment lookup
TTY probing
stdout/stderr discovery
Process access
TextWriter access
global style configuration
background work
```

## Disabled rendering

When `stylingEnabled == false`, the implementation follows a distinct plain path that concatenates run text only.

It does not generate ANSI and strip it afterward.

Therefore:

```text
DISABLED_GENERATES_ANSI=NO
DISABLED_GENERATE_THEN_STRIP=NO
```

## Enabled rendering

The initial foreground mapping is:

```text
black   -> ESC[30m
red     -> ESC[31m
green   -> ESC[32m
yellow  -> ESC[33m
blue    -> ESC[34m
magenta -> ESC[35m
cyan    -> ESC[36m
white   -> ESC[37m
default -> no generated foreground SGR
```

The exact transition algorithm is:

```text
default -> default       : emit nothing
default -> color         : emit target foreground SGR
same color -> same color : emit nothing
color -> default         : emit ESC[0m
color A -> color B       : emit ESC[0m then target foreground SGR
end while color active   : emit ESC[0m
```

The implementation does not use `39m` and does not transition directly from one foreground SGR to another without the ratified reset.

Completely plain/default StyledText generates no ANSI even when styling is enabled, and logically empty StyledText renders as the empty String in both modes.

## Arbitrary input text

Run text is copied unchanged.

The renderer is neither a markup parser nor a terminal sanitizer. Existing content passes through without reinterpretation, including:

- ESC/control characters;
- newlines and carriage returns;
- tabs;
- Unicode;
- combining characters; and
- bidi controls.

The renderer tracks only SGR state that it generated itself and does not promise to reconstruct arbitrary external terminal state embedded in input text.

## Private access to Style and StyledText state

LIB020-A kept Style foreground state and StyledText runs outside guest-visible slots.

LIB020-B preserves that opacity.

During bootstrap, the Style and StyledText sealing facilities are each created once and the same exact instances are supplied to:

```text
std:text/Style
std:text/StyledText
std:text/ANSI
```

ANSI receives the two facilities as bootstrap-private module members, captures them lexically, and removes those slots before exposing its module surface.

No new Style or StyledText family is created.

The imported ANSI module exposes exactly:

```text
render
```

and does not expose the private facilities or family state.

## Transfer and authority invariants

LIB020-B does not change PLAT051 or the LIB020-A transfer model.

Style and StyledText remain non-portable by default:

```text
Actor transfer            -> NonTransferableValue
isolated-parallel capture -> NonParallelValue
```

No semantic-transfer family registration is added.

LIB020-B also makes no change to TextWriter or any output ownership/lifecycle contract.

## Implementation files

The published commit adds:

```text
protos/lib/text/ANSI.protos
protos/tests/conformance/library/text/ansi.protos
```

and makes bounded supporting changes to:

```text
protos/tests/conformance/manifest.tsv
src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosSealedFamilyFacility.java
src/test/java/com/guillermomolina/protos/execution/ProtosSealedFamilyFacilityTest.java
pom.xml
CHANGELOG.md
```

## Evidence coverage

The conformance coverage fixes:

- ANSI module public surface;
- exact StyledText/Boolean validation;
- rejection of lookalikes, children, malformed values and invalid flags;
- proof that hostile candidate behavior is not invoked by validation;
- disabled rendering with no generated ESC;
- all eight basic foreground SGR codes;
- default/plain behavior;
- every ratified transition form;
- final reset behavior;
- empty rendering; and
- pass-through of newline/CR, controls, Unicode, combining characters, bidi controls and existing ESC sequences.

The Java focal coverage proves that ANSI receives the same Style and StyledText facility instances used by their owning modules, that those bootstrap-private facilities do not remain on the public ANSI surface, and that an end-to-end styled render produces the expected reset-correct ANSI String.

## Scope reconciliation

LIB020-B introduces no new public decision beyond D190/D191.

```text
D190_REOPENED=NO
D191_REOPENED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
SPECIFICATION_CHANGE=NO
ANSI_IMPLEMENTED=YES
COLORMODE_IMPLEMENTED=NO
TEXTWRITER_CHANGED=NO
TTY_ACCESS=NO
ENVIRONMENT_ACCESS=NO
PLAT051_TRANSFER_CHANGED=NO
```

## Next slice

The ratified LIB020 sequence continues with:

```text
NEXT_SLICE=LIB020-C
NEXT_SLICE_NAME=ColorMode AUTO / ALWAYS / NEVER resolution
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

LIB020 remains open after B because C is still pending.
