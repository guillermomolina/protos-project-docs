# D190 — Reusable styled-text and explicit color policy contract

Status: **RATIFIED**

Decision authority: GitHub Issue `#826` — `D190 — Reusable styled-text and explicit color policy contract`

Triggered by: GitHub Issue `#823` — `LIB020 — Reusable terminal styling and color policy`

Downstream consumer blocked on implementation: GitHub Issue `#432` — `LIB015 — Structured logging events, formatting and sink boundary`

Nature: durable non-normative design record. Observable Standard Library semantics remain authoritative through the applicable specification and published implementation.

## Approval provenance

LIB020-0 completed the comparative terminal-styling architecture audit and recommended one exact candidate.

The project owner explicitly approved that recommendation in the active interaction on 2026-10-07:

> ok apruebo la recomendación

The owner approval covers the exact candidate recorded here. Allocation of D190 is a governance correction required by the current `AGENTS.work/DESIGN.md`: the selected model creates observable Standard Library API and a durable authority architecture, so it must be carried through Dxxx even though the originating work item is LIB020.

D190 does not introduce a new candidate or reopen the approved one.

## Revision reconciliation

The LIB020-0 audit was completed against Protos product revision:

```text
915fa3739e3a0a6e7f0934e3975e79d3bb24eba8
LIB015-D1: add timestamped LogEvent and explicit Logger time source
```

Before ratification, current Protos `main` was rechecked at:

```text
92db72eb8b1e5d66f316a7e9f72a8c7023a28278
TEST009-AD: keep unsupported lookup failure out of PE
0.3.261-SNAPSHOT
```

The intervening product commit changes only:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java
```

It does not touch the text-I/O, TextWriter, Process I/O, logging, Test Tool, Package Tool, CLI terminal integration, or prospective styled-text surfaces that govern this decision.

Therefore the LIB020-0 evidence remains applicable without reopening the candidate.

## Problem

Protos needs reusable colored/styled human output for multiple consumers, beginning with the already-ratified LIB015 logging color requirement and also applicable to Test Tool, Package Tool, and CLI presentation.

The capability must not make ANSI escape sequences part of logging events, JSON, Errors, or other domain values, and must not move terminal/environment authority into pure formatting code.

The design must separate:

```text
content/style intent
        !=
ANSI representation
        !=
environment/output authority
```

## Current Protos constraints

The audit established these relevant existing boundaries:

- `TextWriter` already owns text encoding, ordered logical writes, Future completion, line framing, backpressure/failure propagation and lifecycle.
- Process stdout/stderr are byte capabilities; they may target terminals, files, pipes or other destinations.
- `TextSink` borrows an explicitly supplied TextWriter and does not discover process streams, inspect environment, own the writer, flush it implicitly or close it.
- `TextFormatter` and `JsonFormatter` are deterministic plain/structured formatters with no terminal styling authority.
- `LogEvent` is structured domain data and has no output-format or terminal authority.
- Test Tool and Package Tool already have independent presentation flows through TextWriter; they must not be forced through logging merely to gain color.
- JLine exists internally in the CLI but is too broad and too host-specific to become public Protos styling semantics.

## Comparative evidence

LIB020-0 compared materially different prior art.

### Rust — anstyle / anstream / owo-colors

Useful separation exists between lightweight semantic style values and output/capability handling. The important lesson is not API spelling but the ability to keep style representation small while resolving output policy separately.

### Node — Chalk / supports-color

Chalk demonstrates ergonomic mixed-style composition, while supports-color keeps capability/environment detection distinct. Ecosystem-specific numeric `FORCE_COLOR` levels are not adopted as Protos semantics.

### Python — Rich / Colorama

Rich demonstrates capable terminal detection, `NO_COLOR`, `FORCE_COLOR`, and `TERM=dumb` handling, but its console/rendering institution is much broader than the Protos requirement. Colorama demonstrates legacy Windows adaptation, which is intentionally not part of the first Protos public contract.

### JVM — JLine / picocli

JLine is a full terminal abstraction with writer, size, signals and capabilities. That is excessive for LIB020. Picocli provides evidence for a simple explicit auto/on/off style policy without making a complete terminal framework mandatory.

### Go

Go color libraries show the usefulness of TTY/environment heuristics but also the downside of process-global ambient switches such as a global `NoColor`. Lip Gloss evolution toward plain style values and output-boundary capability handling reinforces the selected separation.

### .NET — Spectre.Console

Spectre.Console shows the scalability of a rich terminal institution, but also how much surface and runtime machinery that choice brings. It is evidence for deferring full terminal abstraction until Protos actually needs it.

### NO_COLOR and FORCE_COLOR conventions

`NO_COLOR` is useful as consumer-side default policy. It should influence AUTO behavior rather than become hidden state read by a pure renderer. Explicit user policy may override it. FORCE_COLOR-style conventions are ecosystem policy, not a required public Protos semantic in the initial capability.

## Candidate set

The audit evaluated:

### A — Style + String renderer

Small public surface, but awkward for mixed-style lines because consumers must manually compose independently rendered fragments.

### B — StyledText / spans

Good semantic separation and mixed-style composition, but a general public span hierarchy is more structure than current needs require.

### C — builder / operation stream

Potentially allocation-efficient at very high run counts, but prematurely installs stateful machinery and sequencing concerns.

### D — Style directly applied to String

Minimal syntax but mixes semantic intent with representation too early and scales poorly to mixed regions.

### E — TerminalWriter

Centralizes policy but incorrectly couples styling to output authority, risks duplicating TextWriter responsibilities, and grows toward a terminal framework.

### F — flat StyledText runs + pure renderer

Selected.

It preserves semantic style values, supports mixed-style text without a tree hierarchy, keeps rendering pure, reuses ordinary String + TextWriter output, and leaves richer streaming/capability layers addable later.

## Comparative scorecard

Scores are 1–5 over the 12 common DESIGN dimensions.

| Candidate | Total |
|---|---:|
| A — Style + String renderer | 49/60 |
| B — StyledText / spans | 54/60 |
| C — builder / operation stream | 44/60 |
| D — Style directly applied to String | 37/60 |
| E — TerminalWriter | 42/60 |
| **F — flat StyledText runs + pure renderer** | **58/60** |

Dimensions evaluated:

1. correctness / invariant preservation;
2. Protos alignment;
3. present-need proportionality;
4. incremental growth;
5. future-option resilience;
6. scalability;
7. conceptual simplicity;
8. portability / implementation freedom;
9. runtime / resource cost;
10. failure / operability;
11. deferral / reversibility / migration cost;
12. evidence maturity / implementation risk.

The arithmetic total supports the qualitative conclusion but does not replace it. F wins because it satisfies the current use cases while preserving growth paths without installing speculative terminal institutions.

## Ratified public model

The public semantic model is:

```text
Style
StyledText
render(styledText, stylingEnabled: Boolean) -> String
ColorMode.AUTO / ALWAYS / NEVER
```

### Style

`Style` is an immutable semantic value.

Initial foreground vocabulary:

```text
default
black
red
green
yellow
blue
magenta
cyan
white
```

Not included initially:

```text
bright colors
bold
dim
italic
underline
background colors
256-color palette
truecolor
hyperlinks
cursor movement
screen clearing
alternate screen
terminal size/watch
TUI behavior
```

Those features may be added later only when concrete requirements justify them.

### StyledText

`StyledText` is an immutable flat sequence of logical runs:

```text
(text: String, style: Style)
```

The public model does not require:

- a nested tree;
- a public Span hierarchy;
- embedded ANSI;
- a stateful renderer object;
- terminal ownership.

Adjacent equal-style runs may be coalesced without changing semantics.

### Pure rendering

The required operation is conceptually:

```text
render(styledText, stylingEnabled: Boolean) -> String
```

When `stylingEnabled == false`:

- result is deterministic plain text;
- no library-generated ESC/ANSI bytes appear;
- the implementation must not generate ANSI and then strip it.

When `stylingEnabled == true`:

- the result uses ANSI SGR sufficient for the approved basic foreground palette;
- style transitions must not leak generated style into following output;
- a simple full-reset transition strategy is acceptable initially;
- if generated styling is active at the end, the result ends reset;
- rendering remains a pure transformation to ordinary String.

All-plain StyledText must not acquire unnecessary escape sequences.

The renderer does not preserve arbitrary pre-existing external ANSI terminal state. Correct composition is:

```text
compose StyledText
then render once
```

not independent ANSI fragment concatenation.

## ColorMode

The reusable policy type is:

```text
AUTO
ALWAYS
NEVER
```

Its semantic resolution is:

```text
NEVER  -> false
ALWAYS -> true
AUTO   -> caller-supplied autoEnabled
```

The renderer receives the resolved Boolean, not ColorMode plus ambient process state.

`ALWAYS` intentionally means explicit emission even when automatic capability detection would have declined color. If a caller directs such output to a file or incapable destination, that is the caller's explicit policy.

## TTY and environment authority

TTY detection and environment policy are not owned by the pure styling renderer.

They belong at an authorized consumer/platform boundary that already has access to the relevant process/output capability.

The initial public styling contract therefore does not add:

- implicit `isatty` calls;
- implicit stdout/stderr discovery;
- hidden Process access;
- hidden environment reads;
- process-global styling configuration;
- mandatory background Tasks/threads;
- a public TerminalCapabilities institution.

### NO_COLOR

Under AUTO, a consumer may apply this policy:

```text
NO_COLOR is non-empty -> autoEnabled = false
```

Explicit `ALWAYS` may override this default policy because it is an explicit user/caller selection.

`NO_COLOR` is a color convention, not proof that every possible future non-color emphasis attribute must be disabled.

### FORCE_COLOR / CLICOLOR / CI heuristics

These are deferred as public Protos semantics.

A consumer may later map environment conventions to Protos-native policy, for example FORCE_COLOR to ALWAYS, without changing the pure rendering contract.

Chalk-specific numeric FORCE_COLOR levels are not adopted.

### TERM=dumb

This may be used by an authorized AUTO-detection boundary as a hint. It is not renderer policy.

## TextWriter integration

D190 does not change TextWriter.

The selected flow is:

```text
StyledText
    -> pure render(...)
    -> String
    -> existing TextWriter.writeText/writeLine
```

Existing TextWriter semantics remain unchanged:

- encoding;
- ordered logical writes;
- Future completion;
- line framing;
- backpressure;
- failure propagation;
- ownership;
- flush/close lifecycle.

No `TerminalWriter` is introduced.

## Arbitrary String and control-character contract

The styling capability is not a markup parser or terminal sanitizer.

- Input text is treated as ordinary String data.
- Existing ESC/control characters already present in input are passed through unchanged.
- The renderer does not interpret embedded markup.
- Newlines are preserved; TextWriter owns line framing when `writeLine` is used.
- Unicode scalar content, combining characters and bidi controls are preserved unchanged.
- Grapheme layout, terminal width and bidi safety are outside D190.
- Terminal-output sanitization, when needed by a consumer, is a separate responsibility.

This distinction prevents styling from silently mutating ordinary String content.

## Portability

The public model depends only on semantic values and String production.

It does not expose JLine, JVM Console, POSIX `isatty`, Win32 console APIs or another host mechanism as Protos semantics.

Modern Windows VT/SGR support can be used when a consumer determines that ANSI rendering is appropriate. Legacy Win32 console emulation is not part of the public baseline.

A later platform decision may add richer capability discovery if Protos needs 256-color, truecolor, terminal width or non-ANSI output adaptation. That future adapter can still feed the current Boolean render boundary.

## Pay-as-you-grow contract

Programs that do not import/use styling must not pay for:

- terminal probing;
- environment discovery;
- background workers;
- writer wrappers;
- global registries;
- rich terminal capability objects.

Disabled styling performs plain run concatenation rather than generate-and-strip ANSI.

If future syntax highlighting or diagnostics produce very large run counts, implementation may add:

- a builder;
- streaming rendering;
- `renderTo` over an explicit destination;
- more compact internal storage;

provided the public Style/StyledText logical contract remains preserved.

## Future regret and escape path

The strongest current argument against F is allocation overhead: eight initial colors could appear simple enough for `Style.red.apply(text)`.

That smaller-looking model becomes expensive as soon as one line contains independently styled and plain regions, because every consumer must invent its own fragment composition and reset discipline.

The main future regret scenario for F is thousands of runs from syntax highlighting, diagnostics, truecolor or hyperlinks.

Escape path:

```text
same Style semantics
same flat-run logical StyledText model
+ optional builder / streaming renderer / renderTo implementation
```

No foundational authority or semantic rewrite is required.

## Anti-overengineering gate

Capabilities whose deferral would be expensive or create incompatible consumer-private machinery are included now:

- semantic Style value;
- flat mixed-style StyledText;
- ANSI-free domain representation;
- explicit render enablement;
- reusable AUTO/ALWAYS/NEVER policy.

Capabilities whose deferral is cheap are intentionally excluded:

- emphasis attributes;
- bright/background/256/truecolor;
- hyperlinks;
- public terminal-capability levels;
- CI-specific detection;
- legacy Windows emulation;
- streaming builders;
- cursor/screen/TUI behavior.

The selected architecture is future-compatible, not future-preimplemented.

## LIB015 unblock contract

LIB015 may proceed to its colored human formatter only after LIB020 publishes the reusable foundation required by this decision.

Minimum contract:

1. `Style` can express basic foreground colors plus default.
2. `StyledText` can combine plain/styled fragments with no ANSI stored in its semantic values.
3. disabled rendering produces a plain String with no generated ESC bytes.
4. enabled rendering produces reset-correct ANSI String for the basic palette.
5. `ColorMode` resolves AUTO/ALWAYS/NEVER to an explicit Boolean without hidden environment/TTY access.

Then the logging flow is:

```text
LogEvent
    -> logging-specific layout/theme
    -> StyledText
    -> LIB020 renderer
    -> String
    -> existing TextSink
```

D190 does not change:

- `LogEvent`;
- `JsonFormatter`;
- existing `TextFormatter`;
- timestamp semantics;
- TextSink authority/lifecycle.

A later styled human logging formatter must preserve the existing canonical plain `TextFormatter` bytes when styling is disabled.

## Test Tool and Package Tool reuse

The reusable path for tool presentation is independent of logging:

```text
tool presentation data
    -> StyledText
    -> LIB020 renderer
    -> existing TextWriter
```

Tool result models do not become LogEvents merely to gain color.

## Implementation decomposition

After D190 ratification, LIB020/#823 may proceed in these bounded slices:

### LIB020-A — semantic styled-text core

Implement:

- Style;
- basic foreground/default vocabulary;
- flat immutable StyledText runs;
- construction/composition;
- valid empty/plain/styled states;
- adjacent equal-style coalescing where useful.

Do not implement ANSI, TTY/env detection, ColorMode or writer changes in A.

### LIB020-B — deterministic renderers

Implement:

- disabled/plain rendering;
- enabled ANSI SGR rendering;
- reset/no-leak behavior;
- no unnecessary ANSI for plain content;
- Unicode/control pass-through;
- no TextWriter change.

### LIB020-C — ColorMode policy

Implement:

- AUTO / ALWAYS / NEVER values;
- pure resolution from `mode + autoEnabled` to Boolean;
- no environment or TTY discovery inside the policy value/renderer.

Consumer-specific detection and mapping are later work.

## Decision-family classification

This is a Dxxx because it defines observable reusable Standard Library API/behavior and a durable authority architecture independent of one host runtime.

It is not a PLATxxx:

- no JVM/Truffle/JLine implementation architecture is selected;
- no OS-specific mechanism becomes public semantics;
- conforming implementations may realize the same contract differently.

## Machine-readable ratification

```text
D190_STATUS=RATIFIED
OWNER_APPROVAL_DATE=2026-10-07
OWNER_APPROVAL_PROVENANCE=ACTIVE_INTERACTION_EXPLICIT_APPROVAL
TRIGGER=LIB020/#823
DOWNSTREAM_BLOCKED_CONSUMER=LIB015/#432

RESEARCH_BASELINE_PROTOS_REVISION=915fa3739e3a0a6e7f0934e3975e79d3bb24eba8
RATIFICATION_RECONCILIATION_PROTOS_REVISION=92db72eb8b1e5d66f316a7e9f72a8c7023a28278
HEAD_MOVEMENT_RELEVANT_TO_DECISION=NO

SELECTED_CANDIDATE=F
PUBLIC_MODEL=IMMUTABLE_STYLE_PLUS_FLAT_STYLEDTEXT_RUNS
STYLE_VALUE_REQUIRED=YES
STYLEDTEXT_VALUE_REQUIRED=YES
PUBLIC_SPAN_TREE_REQUIRED=NO
PUBLIC_STATEFUL_RENDERER_REQUIRED=NO
TERMINAL_WRITER_REQUIRED=NO
TEXTWRITER_CHANGE_REQUIRED=NO
PUBLIC_TERMINAL_CAPABILITIES_INITIAL=NO

INITIAL_FOREGROUND_COLOR_MODEL=ANSI_BASIC_8_PLUS_DEFAULT
INITIAL_EMPHASIS_MODEL=NONE_DEFERRED
BACKGROUND_COLOR_INITIAL=NO
ANSI_256_INITIAL=NO
TRUECOLOR_INITIAL=NO
HYPERLINKS_INITIAL=NO

RENDERING_OWNER=PURE_LIB020_TRANSFORMATION
RENDER_RESULT=ORDINARY_STRING
PLAIN_RENDER_NO_GENERATED_ESC=YES
ANSI_RENDER_SELF_CONTAINED_RESET=YES

COLOR_MODE_MODEL=AUTO_ALWAYS_NEVER
AUTO_POLICY=CALLER_SUPPLIED_AUTO_ENABLED
ALWAYS_POLICY=RENDER_STYLING_TRUE
NEVER_POLICY=RENDER_STYLING_FALSE

TTY_DETECTION_OWNER=AUTHORIZED_CONSUMER_OR_PLATFORM_BOUNDARY
ENVIRONMENT_POLICY_OWNER=AUTHORIZED_CONSUMER_BOUNDARY
NO_COLOR_INITIAL_POLICY=AUTO_CONSUMER_POLICY_EXPLICIT_ALWAYS_MAY_OVERRIDE
FORCE_COLOR_PUBLIC_CONTRACT_INITIAL=NO
CLICOLOR_PUBLIC_CONTRACT_INITIAL=NO
CI_HEURISTICS_INITIAL=NO

AMBIENT_ENVIRONMENT_ACCESS_IN_RENDERER=NO
AMBIENT_TTY_ACCESS_IN_RENDERER=NO
IMPLICIT_STDOUT_STDERR_DISCOVERY=NO
PROCESS_GLOBAL_STYLE_CONFIG=NO
MANDATORY_BACKGROUND_TASK=NO

ARBITRARY_STRING_SANITIZATION_BY_STYLE_RENDERER=NO
UNICODE_CONTENT_PRESERVED=YES
CONTROL_CONTENT_PASSED_THROUGH=YES
LINE_FRAMING_OWNER=TEXTWRITER

LIB015_REUSE_FEASIBLE=YES
TEST_TOOL_REUSE_FEASIBLE=YES
PACKAGE_TOOL_REUSE_FEASIBLE=YES

PLATxxx_REQUIRED=NO
NEXT_WORK_ITEM=LIB020/#823
NEXT_SLICE=LIB020-A
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
```
