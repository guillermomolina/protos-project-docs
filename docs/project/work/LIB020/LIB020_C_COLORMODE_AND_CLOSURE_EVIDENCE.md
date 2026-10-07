# LIB020-C — ColorMode resolution and LIB020 closure evidence

Status: **COMPLETE — LIB020 REUSABLE FOUNDATION CLOSED**

Owning work item: GitHub Issue `#823` — `LIB020 — Reusable terminal styling and color policy`

Ratified design authorities:

- D190 / #826 — reusable styled-text and explicit color-policy architecture.
- D191 / #827 — exact Standard Library module and public API surface.

Published product revision:

```text
PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
PROTOS_COMMIT=LIB020-C: add pure std:text/ColorMode resolution policy
IMPLEMENTATION_VERSION=0.3.269-SNAPSHOT
```

Validation provenance is maintainer-reported:

```text
PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED
```

## LIB020-C published surface

LIB020-C publishes:

```text
std:text/ColorMode
```

with exactly the reusable policy surface:

```text
ColorMode.AUTO
ColorMode.ALWAYS
ColorMode.NEVER
ColorMode.recognizes(value)
ColorMode.resolve(mode, autoEnabled)
```

The three modes are canonical ordinary Strings:

```text
ColorMode.AUTO   === "AUTO"
ColorMode.ALWAYS === "ALWAYS"
ColorMode.NEVER  === "NEVER"
```

No wrapper object, host enum, registry, or runtime family is introduced.

## Recognition contract

`ColorMode.recognizes(value)` answers true exactly for the three canonical String modes and false for every other value.

Recognition uses exact String identity checks and invokes no behavior on candidate values.

There is no parsing, coercion, case folding, alternate spelling, or duck typing.

## Resolution contract

`ColorMode.resolve(mode, autoEnabled)` implements:

```text
AUTO   -> autoEnabled
ALWAYS -> true
NEVER  -> false
```

Both arguments are validated on every call.

`mode` must be one of the three canonical modes.

`autoEnabled` must be exactly canonical Boolean `true` or `false`, including for `ALWAYS` and `NEVER`; the implementation does not skip validation merely because the final result is otherwise known.

Invalid arguments signal ordinary Error.

## Pure policy / authority boundary

ColorMode performs no:

```text
environment access
Process access
stdout/stderr discovery
TTY probing
terminal-capability detection
global configuration
```

In particular, `AUTO` does not read `NO_COLOR`, `FORCE_COLOR`, `CLICOLOR`, `TERM`, CI state, or any equivalent ambient source.

The caller owns the authority needed to compute `autoEnabled`.

The intended composition is therefore:

```text
authorized caller/platform boundary
    -> computes autoEnabled
    -> ColorMode.resolve(mode, autoEnabled)
    -> ANSI.render(styledText, resolvedBoolean)
```

No ColorMode dependency is added inside `std:text/ANSI`; the two public APIs remain independently composable.

## Implementation shape

The published product commit adds:

```text
protos/lib/text/ColorMode.protos
protos/tests/conformance/library/text/color-mode.protos
```

and makes bounded supporting changes to:

```text
protos/tests/conformance/manifest.tsv
pom.xml
CHANGELOG.md
```

No Java, runtime, bootstrap, PLAT051, TextWriter, Style, StyledText, or ANSI implementation change is part of LIB020-C.

## Evidence coverage

The conformance coverage fixes:

- exact module public surface;
- canonical String identity of AUTO, ALWAYS, and NEVER;
- exact recognition success/failure cases;
- proof that hostile candidate behavior is not invoked;
- all six valid resolution combinations;
- rejection of invalid modes;
- unconditional validation of invalid `autoEnabled` values for AUTO, ALWAYS, and NEVER;
- exact arity; and
- composition with `std:text/ANSI` without coupling ANSI to ColorMode.

## Complete LIB020 reusable foundation

The owner-ratified A/B/C sequence is now fully published:

### LIB020-A

Product revision:

`c495b31dc21f71929ae460addad60613acb3d695`

Published:

```text
std:text/Style
std:text/StyledText
```

with immutable semantic styling values, opaque flat logical runs, safe exact-family recognition, Style semantic equality/hash, and explicit non-portability of the sealed families.

### LIB020-B

Product revision:

`b10569680b1532b277ba3ce49eced2e818833d93`

Published:

```text
std:text/ANSI
ANSI.render(styledText, stylingEnabled)
```

with pure deterministic plain/ANSI String rendering, reset-correct basic foreground SGR output, no ambient authority, and arbitrary input-text pass-through.

### LIB020-C

Product revision:

`246a24994d74ca084dc43ceddd4c400c2af94b15`

Published:

```text
std:text/ColorMode
AUTO / ALWAYS / NEVER
recognizes
resolve
```

with pure caller-supplied AUTO resolution and no environment/TTY discovery.

Together these slices satisfy the reusable foundation ratified by D190/D191.

## Deferred features remain deferred

Closure of LIB020 does not add or authorize:

- emphasis;
- background colors;
- 256-color or truecolor;
- hyperlinks;
- cursor/screen control;
- terminal size/watch APIs;
- TTY/environment discovery inside the pure library;
- writer wrappers or TerminalWriter;
- global style configuration;
- logging-specific themes; or
- consumer-specific adoption work.

Those are outside the closed baseline unless separately owned and authorized later.

## LIB015 unblock

LIB015 / #432 previously remained blocked because its ratified colored-human-output adapter required this separately owned reusable capability.

That blocker is now satisfied.

The reusable API available to the future logging adapter is:

```text
Style(foreground)
StyledText(text [, style])
StyledText.concat(...)
ANSI.render(styledText, stylingEnabled)
ColorMode.AUTO / ALWAYS / NEVER
ColorMode.resolve(mode, autoEnabled)
```

The logging adapter must continue to preserve the LIB015 boundaries:

- no ANSI in LogEvent;
- no ANSI in JSON;
- no logging-private styling framework;
- no environment/TTY discovery in the pure logging core;
- caller/authorized boundary computes AUTO input explicitly;
- existing plain logging semantics remain the baseline.

LIB020 closure therefore unblocks, but does not itself implement, the remaining LIB015 colored-human-output adapter.

## Closure state

```text
LIB020_A_STATUS=COMPLETE
LIB020_B_STATUS=COMPLETE
LIB020_C_STATUS=COMPLETE

PROTOS_REVISION=246a24994d74ca084dc43ceddd4c400c2af94b15
IMPLEMENTATION_VERSION=0.3.269-SNAPSHOT

D190_STATUS=RATIFIED
D191_STATUS=RATIFIED
D190_REOPENED=NO
D191_REOPENED=NO

STYLE_IMPLEMENTED=YES
STYLEDTEXT_IMPLEMENTED=YES
ANSI_IMPLEMENTED=YES
COLORMODE_IMPLEMENTED=YES

TEXTWRITER_CHANGED=NO
TTY_ACCESS_IN_PURE_LIBRARY=NO
ENVIRONMENT_ACCESS_IN_PURE_LIBRARY=NO
PLAT051_TRANSFER_CHANGED_BY_C=NO

NEW_Dxxx_REQUIRED=NO
NEW_PLATxxx_REQUIRED=NO
SPECIFICATION_CHANGE=NO

PROTOS_GIT_DIFF_CHECK=PASS
PROTOS_ALL_LOCAL_TESTS=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED

LIB020_PARENT_CLOSURE_READY=YES
LIB020_NEXT_SLICE=NONE
LIB015_COLOR_ADAPTER_BLOCKER_RESOLVED=YES
```
