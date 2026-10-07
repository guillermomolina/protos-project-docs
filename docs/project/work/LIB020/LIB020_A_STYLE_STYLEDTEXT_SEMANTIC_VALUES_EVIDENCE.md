# LIB020-A — Style and StyledText semantic-value evidence

Status: **COMPLETE**

Work authority: `guillermomolina/protos#823` — `LIB020 — Reusable terminal styling and color policy`

Ratified design authorities:

- D190 / #826 — reusable styled-text and explicit color-policy architecture.
- D191 / #827 — exact Standard Library module and public API surface.

Published product revision:

```text
PROTOS_REVISION=c495b31dc21f71929ae460addad60613acb3d695
PROTOS_COMMIT=LIB020-A: add std:text Style and StyledText semantic values
IMPLEMENTATION_VERSION=0.3.267-SNAPSHOT
```

Validation provenance is maintainer-reported:

```text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

## Published surface

LIB020-A publishes exactly the first D191 implementation slice:

```text
std:text/Style
std:text/StyledText
```

It does not publish the later LIB020-B/C surfaces:

```text
std:text/ANSI       NOT_IMPLEMENTED
std:text/ColorMode  NOT_IMPLEMENTED
```

No TextWriter, terminal-detection, environment, TTY, theme, markup, background-color, emphasis, 256-color, truecolor, hyperlink, or terminal-capability API is added by this slice.

## Style

`std:text/Style` implements the D191 value contract:

- `Style(foreground)` accepts exactly `default` plus the eight basic foreground colors;
- every successful construction returns a fresh frozen value;
- state is opaque and is not exposed through guest-visible state slots;
- `Style.recognizes(value)` performs exact-family recognition without invoking candidate behavior;
- lookalikes, malformed values, unfrozen forgeries, and children of genuine values are rejected;
- equality is semantic for equal foreground values;
- `hash` is coherent with semantic equality;
- `===` remains ordinary object identity.

The public instance surface is limited to the equality/hash behavior required by the semantic-value contract. The module does not expose foreground inspection, constants, builders, parsers, properties, or effects.

## StyledText

`std:text/StyledText` implements:

```text
StyledText(text)
StyledText(text, style)
StyledText.concat(...String_or_StyledText)
```

Published behavior includes:

- fresh frozen StyledText values;
- flat logical runs with runtime-private state;
- Strings treated as plain/default-style fragments in concat;
- concat always returns StyledText;
- empty runs are discarded;
- adjacent semantically equal styles may be coalesced;
- empty styled fragments retain no observable latent style;
- runs/spans/fragments remain opaque;
- StyledText retains ordinary identity equality and hash rather than defining semantic equality.

## Safe exact-family recognition and opaque state

The implementation adds the internal runtime types:

```text
ProtosSealedValue
ProtosSealedFamilyFacility
```

Each public module receives its own frozen sealing facility during bootstrap. The module captures that facility lexically and removes its bootstrap slot before exposing the public module surface.

The facility owns only the mechanisms Protos source cannot guarantee itself:

- minting an unforgeable family member;
- recognizing family membership from host representation without sending messages to the candidate;
- retaining family-private state outside Protos-visible slots.

Family policy remains in the Protos modules.

This is internal runtime support, not public JVM semantics or an external dependency.

## Transfer boundary

LIB020-A deliberately does **not** opt Style or StyledText into PLAT051 semantic transfer.

A sealed value crossing an Actor boundary is rejected as:

```text
NonTransferableValue
```

and isolated-parallel capture is rejected as:

```text
NonParallelValue
```

This prevents a sealed semantic value from being silently copied into an ordinary object that would lose exact-family identity.

The focal Java tests cover Style and StyledText across both boundaries, values nested inside ordinary graphs, and an ordinary frozen-object control proving that the boundary is not simply rejecting all frozen objects.

## Implementation files

The product commit adds:

```text
protos/lib/text/Style.protos
protos/lib/text/StyledText.protos
src/main/java/com/guillermomolina/protos/execution/ProtosSealedFamilyFacility.java
src/main/java/com/guillermomolina/protos/runtime/ProtosSealedValue.java
protos/tests/conformance/library/text/style.protos
protos/tests/conformance/library/text/styled-text.protos
src/test/java/com/guillermomolina/protos/execution/ProtosSealedFamilyFacilityTest.java
```

and makes bounded supporting changes to:

```text
protos/tests/conformance/manifest.tsv
src/main/java/com/guillermomolina/protos/execution/ProtosCoreBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosParallelRuntime.java
src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java
src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java
pom.xml
CHANGELOG.md
```

The `ProtosCoreBootstrap` module-member map changes from `Map.of` to `Map.ofEntries` because the additional module facilities take the table beyond the `Map.of` arity limit; the existing entries retain their meaning.

## Evidence coverage

The conformance corpus fixes:

- all nine valid Style foregrounds;
- rejection of invalid Style inputs;
- fresh identity and freezing;
- Style semantic equality and coherent hash, including Map-key behavior;
- exact-family recognition;
- rejection of lookalikes, malformed/unfrozen forgeries and children;
- proof that hostile candidate `parent`, `slotNames`, `hasSlot`, `slotValue`, equality and hash behavior is not invoked by recognition;
- StyledText plain/styled construction;
- invalid construction and concat inputs;
- concat result type, composition and empty behavior;
- ordinary identity equality;
- absence of public run/state surface.

The Java focal coverage fixes the runtime-private logical run content, exact module surfaces, sealed-family ownership, Actor/parallel non-portability, nested transfer rejection, and ordinary-object transfer control.

## Scope reconciliation

LIB020-A introduces no new public design choice beyond D190/D191.

```text
D190_REOPENED=NO
D191_REOPENED=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
SPECIFICATION_CHANGE=NO
STYLE_IMPLEMENTED=YES
STYLEDTEXT_IMPLEMENTED=YES
ANSI_IMPLEMENTED=NO
COLORMODE_IMPLEMENTED=NO
TEXTWRITER_CHANGED=NO
```

## Next slice

The ratified LIB020 sequence continues with:

```text
NEXT_SLICE=LIB020-B
NEXT_SLICE_NAME=Pure deterministic ANSI renderer
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```

LIB020 remains open after A because B and C are still pending.
