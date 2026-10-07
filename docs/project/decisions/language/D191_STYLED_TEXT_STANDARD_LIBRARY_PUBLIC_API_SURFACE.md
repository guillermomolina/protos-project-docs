# D191 — Styled-text Standard Library module and public API surface

Status: **RATIFIED**

Decision authority: GitHub Issue `#827` — `D191 — Styled-text Standard Library module and public API surface`

Triggered by: `D190/#826` and `LIB020/#823`

Nature: durable non-normative decision record. Observable Standard Library behavior remains authoritative through the published implementation and its public module documentation.

## Approval provenance

D191 completed the required comparative investigation and recommended the exact public API recorded below.

The project owner explicitly approved that recommendation in the active interaction on 2026-10-07:

> ok aprobada la recomendación

The approval is for the exact candidate below. It does not reopen D190.

## Revision baseline and closure reconciliation

The final current-Protos audit and owner approval were completed against:

```text
D191_AUDIT_BASELINE=e681dc3a09165e53e2a977afd9ab6b89b80b9583
D191_AUDIT_BASELINE_MESSAGE=PLAT051-A2: two-stage Standard Library semantic-value transfer/materialization
```

The D191 investigation rechecked the active Standard Library conventions at that revision, including logging, datetime, regex, JSON, collections, text encoding modules, import/module conventions, recognition/freeze patterns, and equality/hash tests.

Before closure, Protos `main` advanced by one commit to:

```text
PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
PROTOS_REVISION_MESSAGE=TEST009-AF: keep context projection failure out of PE
```

The exact delta from the audit baseline to the closure revision changes only:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosAPlusExecutionProjectionTest.java
```

That delta is a TEST009 partial-evaluation repair and version/changelog update. It does not touch `protos/lib/text/**`, logging, datetime, regex, JSON, collections, Standard Library import/module conventions, recognition/equality APIs, TextWriter, or the prospective styled-text implementation surface. Therefore the D191 evidence and approved public API remain applicable without reopening the decision.

## D190 invariant/delta consistency check

D191 refines API spelling and value-shape details only. It preserves every applicable owner-approved D190 invariant.

| D190 invariant | D191 result |
|---|---|
| Style is an immutable semantic value | Preserved |
| StyledText is immutable flat logical runs | Preserved |
| No nested public span tree | Preserved |
| No ANSI in Style or StyledText | Preserved |
| No public stateful renderer required | Preserved |
| Render is pure to String | Preserved |
| No TextWriter change | Preserved |
| No TerminalWriter | Preserved |
| No initial public terminal-capabilities API | Preserved |
| Initial palette is default + 8 basic foreground colors | Preserved |
| No initial emphasis/background/256/truecolor/hyperlinks | Preserved |
| ColorMode is AUTO / ALWAYS / NEVER | Preserved |
| AUTO consumes explicit caller-supplied autoEnabled | Preserved |
| No ambient environment access | Preserved |
| No ambient TTY probing | Preserved |
| No process-global style configuration | Preserved |

No D190 invariant is narrowed, contradicted, or reopened. No materially new authority or platform consequence is introduced.

```text
DECISION_INVARIANT_CONSISTENCY=PASS
D190_REOPENED=NO
```

## Selected namespace

The selected candidate is the refined neutral text namespace:

```text
std:text/Style
std:text/StyledText
std:text/ANSI
std:text/ColorMode
```

The reason to use the existing `std:text` namespace is that Style and StyledText are semantic text values, while ANSI is a pure text representation adapter and ColorMode is a pure explicit policy value. Creating `std:terminal` now would install a terminal institution before Protos has any public terminal capability object.

The existing `std:text` encoding modules provide a precedent for keeping pure representation/transformation adapters in the neutral text namespace without transferring terminal authority there.

## Style public contract

Import:

```protos
Style: import("std:text/Style")
```

Construction:

```protos
Style("default")
Style("black")
Style("red")
Style("green")
Style("yellow")
Style("blue")
Style("magenta")
Style("cyan")
Style("white")
```

Contract:

- `Style(foreground)` accepts exactly one canonical foreground String.
- The initial vocabulary is exactly `default` plus the eight basic foreground colors above.
- Invalid input signals ordinary `Error`; there is no coercion, case folding, parser, null shorthand, or fallback.
- `Style.recognizes(value)` is a safe exact-family recognition operation and returns false for malformed values or lookalikes without invoking candidate behavior.
- Each successful construction returns a fresh frozen Style value.
- Initial Style state is opaque: no public `.foreground`, properties map, effects collection, builder, or parser is added.
- Independent Style values with the same complete semantic style compare equal under `==`.
- Style has a coherent semantic `hash` compatible with `==`.
- `===` remains ordinary identity, so independently constructed equal styles are not identical.
- The nine foreground choices are constructed semantic values, not required canonical singleton Style constants.

The opaque initial state deliberately avoids freezing a one-property physical representation into the public API and leaves later emphasis/truecolor extension additive.

## StyledText public contract

Import:

```protos
StyledText: import("std:text/StyledText")
```

Construction:

```protos
StyledText("plain")
StyledText("FAIL", Style("red"))
```

Composition:

```protos
StyledText.concat(
    "[phase] ",
    StyledText("FAIL", Style("red")),
    " path/to/test"
)
```

Contract:

- `StyledText(text)` accepts one String and creates plain/default-style logical text.
- `StyledText(text, style)` accepts a String and a recognized Style.
- Invalid arguments signal ordinary `Error`; there is no implicit String coercion or null shorthand.
- `StyledText.concat(...fragments)` accepts only String or recognized StyledText fragments.
- Strings passed to concat are treated as plain/default-style fragments.
- concat always returns StyledText, including zero-argument and one-argument calls.
- `StyledText("")`, styled empty input, and `StyledText.concat()` all represent logically empty text.
- Empty styled fragments carry no latent observable style.
- Logical runs remain opaque. No public `runs`, `spans`, fragments collection, builder, append API, join API, or `+` operator is added initially.
- Implementations may remove empty runs and coalesce adjacent semantically equal styles without changing public behavior.
- Physical storage may later change from an eager array to chunks, a rope, persistent structure, or another representation while preserving the flat logical model.
- `StyledText.recognizes(value)` is a safe exact-family recognition operation.
- StyledText initially keeps ordinary identity equality only; D191 does not define a semantic `==` or semantic hash for StyledText.

Semantic StyledText equality is intentionally deferred because no current consumer needs it and standardizing it now would make normalization/segmentation observability unnecessarily difficult to evolve.

## ANSI renderer public contract

Import:

```protos
ANSI: import("std:text/ANSI")
```

Operation:

```protos
ANSI.render(styledText, stylingEnabled)
```

Contract:

- `styledText` must be a recognized StyledText.
- `stylingEnabled` must be the exact canonical Boolean `true` or `false`.
- Invalid input signals ordinary `Error`.
- The module is pure and stateless.
- It performs no environment lookup, TTY probing, stdout/stderr discovery, Process access, global configuration, writer ownership, flush, close, or background work.
- `false` concatenates text content directly and generates no ESC bytes. It must not render ANSI and strip it afterward.
- `true` emits ANSI SGR only as needed for the approved palette.
- Completely plain StyledText generates no ANSI even when styling is enabled.
- Existing ESC/control characters, newlines, Unicode, combining characters, and bidi controls already present in input Strings are passed through unchanged and are not interpreted as renderer state.

Initial foreground mapping:

```text
black   -> ESC[30m
red     -> ESC[31m
green   -> ESC[32m
yellow  -> ESC[33m
blue    -> ESC[34m
magenta -> ESC[35m
cyan    -> ESC[36m
white   -> ESC[37m
default -> no generated SGR while already default
```

Initial transition contract:

```text
default -> default       : emit nothing
default -> color         : emit target foreground SGR
same color -> same color : emit nothing
color -> default         : emit ESC[0m
color A -> color B       : emit ESC[0m, then target foreground SGR
end while color active   : emit ESC[0m
```

The renderer tracks only SGR state that it generated itself. It does not promise to preserve or reconstruct arbitrary external terminal state embedded in input text.

## ColorMode public contract

Import:

```protos
ColorMode: import("std:text/ColorMode")
```

Public values follow the existing canonical-String Standard Library idiom:

```protos
ColorMode.AUTO   === "AUTO"
ColorMode.ALWAYS === "ALWAYS"
ColorMode.NEVER  === "NEVER"
```

Operations:

```protos
ColorMode.recognizes(value)
ColorMode.resolve(mode, autoEnabled)
```

Contract:

- recognizes returns true only for the three canonical mode Strings.
- resolve validates both arguments on every call.
- `autoEnabled` must be exact canonical Boolean.
- invalid mode or Boolean input signals ordinary `Error`.
- resolution is exactly:

```text
AUTO   -> autoEnabled
ALWAYS -> true
NEVER  -> false
```

There is no zero/one-argument ambient resolver, TTY detection, environment access, or process-global state.

## Invalid-input policy

Across the surface:

- `recognizes(x)` is safe and returns false for invalid, malformed, or lookalike values.
- constructors and operational APIs reject invalid input with ordinary `Error`.
- no API performs implicit String coercion, case folding, markup parsing, null fallback, environment discovery, or duck-typed family acceptance.

## Candidate comparison

D191 compared four required families plus the refined existing-`std:text` form selected here:

- A: put the whole capability under `std:terminal`;
- B: neutral styled-text namespace;
- C: semantic text under `std:text`, ANSI/policy under `std:terminal`;
- D: one façade module exporting constructors/render/policy;
- refined B: reuse existing `std:text` with separate small modules.

The selected refined B has the best proportionality and Protos alignment because it does not create a terminal capability namespace before one exists, keeps each semantic family independently importable, and leaves a future real `std:terminal` namespace available for actual terminal authority.

The strongest counterargument is that ANSI and ColorMode are terminal-presentation concepts and a split namespace is conceptually cleaner. That cost is accepted because the split would create `std:terminal` today without any public terminal capability, while `std:text` already hosts pure representation adapters.

## Pay-as-you-grow and future escape

D191 adds no builder, theme registry, markup language, terminal object, TTY detector, width/cursor/screen API, 256-color model, truecolor model, background model, emphasis model, hyperlink model, or writer wrapper.

Future large styled-text workloads may replace the internal physical representation while runs remain opaque.

Future emphasis/256/truecolor support may extend Style and ANSI rendering additively.

Future non-ANSI renderers may be introduced as separate modules without changing StyledText.

If Protos later gains genuine public terminal capabilities, those capabilities may live under a future `std:terminal` namespace while the D191 text values continue unchanged.

## Implementation decomposition

Owner approval authorizes implementation of the already-planned LIB020 slices in `guillermomolina/protos`:

```text
LIB020-A=Style + StyledText public semantic-value surface
LIB020-B=pure std:text/ANSI renderer
LIB020-C=ColorMode canonical values and pure resolution
```

LIB020-A is next.

No new Dxxx or PLATxxx is required by D191. No separate language-specification edit is required merely to ratify this Standard Library surface; the implementation must publish the corresponding public module documentation and tests with the code.

## Machine-readable ratification

```text
D191_STATUS=RATIFIED
D190_REOPENED=NO
D190_INVARIANTS_PRESERVED=YES
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
RECOMMENDED_CANDIDATE=B_REFINED_EXISTING_STD_TEXT_NAMESPACE
PUBLIC_NAMESPACE=std:text
STYLE_IMPORT_PATH=std:text/Style
STYLE_MODEL=FRESH_FROZEN_EXTENSIBLE_SEMANTIC_VALUE
STYLE_CONSTRUCTION_API=Style(foreground)
STYLE_RECOGNITION=Style.recognizes(value)
STYLE_PUBLIC_STATE=OPAQUE
STYLE_EQUALITY=SEMANTIC
STYLE_HASH=COHERENT_SEMANTIC_HASH
STYLEDTEXT_IMPORT_PATH=std:text/StyledText
STYLEDTEXT_MODEL=IMMUTABLE_FLAT_LOGICAL_RUNS
STYLEDTEXT_PLAIN_CONSTRUCTION=StyledText(text)
STYLEDTEXT_STYLED_CONSTRUCTION=StyledText(text,style)
STYLEDTEXT_COMPOSITION=StyledText.concat(...String_or_StyledText)
STYLEDTEXT_RUN_VISIBILITY=OPAQUE
STYLEDTEXT_RECOGNITION=StyledText.recognizes(value)
STYLEDTEXT_EQUALITY=ORDINARY_IDENTITY_ONLY
STYLEDTEXT_HASH=NO_SEMANTIC_HASH
RENDER_IMPORT_PATH=std:text/ANSI
RENDER_OPERATION=ANSI.render(styledText,stylingEnabled)
RENDER_PURE=YES
COLORMODE_IMPORT_PATH=std:text/ColorMode
COLORMODE_VALUES=AUTO_ALWAYS_NEVER
COLORMODE_REPRESENTATION=CANONICAL_STRINGS
COLORMODE_RECOGNITION=ColorMode.recognizes(value)
COLORMODE_RESOLUTION=ColorMode.resolve(mode,autoEnabled)
AMBIENT_ENVIRONMENT_ACCESS=NO
AMBIENT_TTY_ACCESS=NO
TEXTWRITER_CHANGE=NO
TERMINAL_WRITER=NO
NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
SPECIFICATION_CHANGES_REQUIRED=NO
D191_AUDIT_BASELINE=e681dc3a09165e53e2a977afd9ab6b89b80b9583
PROTOS_REVISION=6a807bb64d44e86a232803d66da9fb4e46953219
CROSS_REFERENCES=PASS
NEXT_WORK_ITEM=LIB020/#823
NEXT_SLICE=LIB020-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES_FOR_LIB020_A
```
