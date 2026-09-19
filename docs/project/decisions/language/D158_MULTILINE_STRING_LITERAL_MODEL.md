# D158 — Multiline String literal model

Status: **RATIFIED — Candidate A (retain the current triple-double multiline model)**

Approval date: **2026-09-19**
Decision issue: `guillermomolina/protos#619`
Trigger: AUD009-B8 / `guillermomolina/protos#617`
Protos evidence revision: `ecf563ed01275929d5b85330e8e6259cc85d73d8`
Project-record base: `531088f4d33aa4818e9ce759d5d34448c3a249d5`

This is a durable non-normative decision record. Observable Protos semantics
remain authoritative only through the applicable ratified material under
`guillermomolina/protos:spec/**`.

## Decision

D158 selects **Candidate A**.

Protos retains the existing triple-double-quoted multiline String literal and its
current structural indentation-normalization contract unchanged.

The retained model includes:

```text
triple-double-quoted multiline String literal     KEEP
opening-newline handling                          KEEP
closing-line/trailing-newline handling            KEEP
closing-delimiter structural indentation prefix   KEEP
exact SPACE/TAB prefix matching                    KEEP
blank-line indentation handling                    KEEP
CR/LF/CRLF structural/newline semantics            KEEP
indentation processing before escape processing   KEEP
lexical error on invalid structural indentation   KEEP
current lexer/parser/conformance machinery         KEEP
```

D158 does not add a raw String form, heredoc, new dedent syntax, interpolation,
or a Standard Library `dedent` API.

## Approval provenance

The full D158 comparison was presented to the project owner, including the
current repository cost, prior art, candidate set, scoring, failure modes,
incremental-design analysis, compatibility/reversibility consequences, and the
argument that removal would not produce a meaningful runtime or parser-speed
benefit.

During review, the project owner explicitly challenged removal on the ground that
the feature is already implemented and asked whether deleting it provides a real
benefit beyond simplification. The resulting candidate refinement established the
material invariant that absence of current production use alone is insufficient
to justify deleting an already-bounded, stable, low-cost language capability.

The exact retained outcome was then presented as Candidate A, with all current
multiline mechanisms classified `KEEP`, and the project owner explicitly
approved it in the active interaction on 2026-09-19:

```text
aprobada
```

```text
SELECTED_CANDIDATE=A
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

## GITHUB021 invariant consistency

Candidate A preserves every retained D158 String/literal invariant:

```text
STRING_EXACT_IMMUTABLE_UNICODE_SCALAR_SEQUENCE   PRESERVED
NO_IMPLICIT_UNICODE_NORMALIZATION                PRESERVED
SINGLE_QUOTED_STRING_LITERAL                     PRESERVED
DOUBLE_QUOTED_STRING_LITERAL                     PRESERVED
RETAINED_ESCAPE_SEMANTICS                        PRESERVED
NO_CHARACTER_LITERAL                             PRESERVED
NO_INTERPOLATION                                 PRESERVED
D157_STRING_INDEXING_DECISION                    UNAFFECTED
```

The current multiline contract is also preserved without semantic delta:

```text
TRIPLE_DOUBLE_MULTILINE_LITERAL                  PRESERVED
STRUCTURAL_INDENTATION_NORMALIZATION             PRESERVED
OPENING_AND_CLOSING_NEWLINE_RULES                PRESERVED
SPACE_TAB_PREFIX_RULES                           PRESERVED
BLANK_LINE_RULES                                 PRESERVED
CR_LF_CRLF_RULES                                 PRESERVED
ESCAPE_INTERACTION                               PRESERVED
INVALID_INDENTATION_FAILURE                      PRESERVED
```

No previously approved invariant is reopened and no hidden semantic or
architectural consequence is introduced.

## Repository evidence

At the evidence revision, the multiline institution is explicitly owned across
`spec/PROTOS_GRAMMAR.md`, `spec/semantics/VALUES_AND_COLLECTIONS.md`, and
`spec/runtime/ABSTRACT_RUNTIME.md`.

The lexer has dedicated triple-double recognition, decoding, and structural
indentation normalization, and the conformance/lexer tests exercise quote-run
boundaries, newline handling, exact structural indentation, and resulting String
values.

AUD009-B8 found no production guest `.protos` consumer that presently depends on
the multiline literal. That remains relevant evidence about current demand, but
D158 concludes that it is not sufficient evidence for removal because the
feature is already bounded and does not impose a meaningful runtime cost on code
that does not use it.

Multiline TOML handling in the repository is independent parser semantics for
TOML and is not evidence either for or against Protos' own multiline literal.

## Comparative evidence

The investigation compared materially different approaches:

- Python demonstrates multiline triple-quoted text without mandatory structural
  dedent.
- Java text blocks demonstrate an integrated multiline model with incidental
  whitespace processing.
- JavaScript template literals combine multiline source text with interpolation,
  which remains outside D158.
- Kotlin separates multiline/raw literal capability from explicit `trimIndent`
  and `trimMargin` operations.
- Swift uses triple-quoted multiline strings with indentation tied to the closing
  delimiter.
- Rust separates ordinary string literals from raw-string concerns and does not
  require one combined multiline/raw/dedent institution.
- Ruby heredocs expose several explicit indentation variants, demonstrating the
  additional syntax surface of a more general heredoc family.
- Elixir heredocs/sigils show another closing-delimiter/indentation-oriented
  multiline model.
- Io provides prototype-language evidence that simple multiline literal support
  is compatible with a small prototype-based language model.

The comparison shows that several coherent alternatives exist. It does not show
that replacing the current Protos model now would solve a demonstrated problem.

## Why Candidate A was selected

The decisive finding is that the relevant cost is not historical implementation
effort but **ongoing cost and constraint**.

The current multiline feature is already isolated, specified, implemented, and
tested. Programs that do not use it pay effectively no runtime cost, and the
extra lexical recognition does not provide a meaningful parser-performance
opportunity if removed.

Removing it would instead require specification, lexer, and conformance changes,
would intentionally break source that uses the syntax, and could force a later
redesign/reintroduction if multiline source becomes common.

No current architectural decision, runtime model, String representation,
Unicode invariant, concurrency model, or library design is blocked by retaining
it.

Therefore the B8 `REMOVE_NOW_RECONSIDER_LATER` proposal is falsified at the
purpose-built D158 decision stage. The current institution remains `KEEP`.

## Rejected candidates

### Candidate B — remove multiline syntax for now

Rejected because the primary benefit is repository/specification simplification,
not an observable runtime, parser-performance, architectural, or semantic gain.
The current feature is bounded and cheap when unused, while removal creates
compatibility churn and implementation work.

### Candidate C — simpler triple-quoted multiline syntax without automatic dedent

Rejected because it changes already-defined semantics without evidence that the
current indentation model causes a real problem. It would create migration work
merely to substitute one plausible multiline policy for another.

### Candidate D — raw/heredoc/dedent-style alternatives

Rejected for present adoption because these mechanisms add or replace public
surface without demonstrated need. They remain useful prior art if a future use
case exposes shortcomings in the retained model.

## Strongest argument against Candidate A

The strongest objection is permanent semantic surface: retaining the feature
means Protos continues to own detailed rules for opening/trailing newlines,
SPACE/TAB structural prefixes, blank lines, failure behavior, CR/LF/CRLF, and
ordering relative to escape processing even though production guest source does
not currently require the capability.

That objection is accepted as a real maintenance cost, but it is not large enough
to justify compatibility churn and deletion work in the absence of a concrete
conflict or measurable benefit.

## Future-scenario stress result

A future text-heavy Protos ecosystem may reveal that the current model is too
strict, that exact-preserving multiline text is preferable, that raw strings are
needed, or that dedent should be explicit rather than lexical.

Candidate A does not prevent reopening that question. If actual source exposes a
specific deficiency, a later decision can compare the current behavior against
that demonstrated requirement and make a targeted compatibility choice.

Conversely, if multiline strings remain rarely used, their bounded lexer/spec
surface can continue to remain dormant without charging ordinary runtime use.

## Incremental-design result

```text
PAY_FOR_WHAT_YOU_NEED
    ordinary programs pay essentially no runtime cost for retained multiline syntax

GROW_AS_YOU_NEED
    raw strings, heredocs, interpolation, and explicit dedent remain separable later

COST_OF_DEFERRAL
    low for those additional capabilities; none must be preimplemented now

SMALLEST_SUFFICIENT_CHANGE
    no change to the current language
```

This decision distinguishes future compatibility from future preimplementation:
retaining the current multiline capability does not authorize adjacent String
syntax or text-processing features.

## Normative and implementation routing

Candidate A makes **no normative semantic change** and requires **no
implementation reconciliation**. Existing specification, lexer/parser behavior,
and tests remain authoritative as they stand.

No `Ixxx` implementation issue is required by D158 closure.

```text
D158_STATUS=RATIFIED
SELECTED_CANDIDATE=A
MULTILINE_LITERAL=KEEP
STRUCTURAL_INDENTATION_NORMALIZATION=KEEP
NORMATIVE_RECONCILIATION_REQUIRED=NO
IMPLEMENTATION_RECONCILIATION_REQUIRED=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
