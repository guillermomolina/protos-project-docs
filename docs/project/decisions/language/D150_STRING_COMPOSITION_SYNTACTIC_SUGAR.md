# D150 — String composition syntactic sugar

Status: **RATIFIED — Candidate S0 (no additional sugar)**

Approval date: **2026-09-18**
Decision issue: `guillermomolina/protos#588`
Semantic prerequisite: D141 / `guillermomolina/protos#569`
D141 normative Protos revision: `b504b486a81ab880a51924a6eb736cba7ca2d51b`
D141 specification revision: `0.1.424`

This is a durable non-normative decision record. D150 changes no normative
Protos syntax or semantics.

## Decision

The project selects **S0** for the current specification: add no new String
composition syntactic sugar.

Ordinary source remains the intended spelling for direct composition:

```protos
"hola " + como + " estas"
```

D141's separately ratified aggregate mechanism also remains available:

```protos
"hola ".concat(como, " estas")
```

No `c"...{expression}..."`, interpolation form, template String, prefixed String,
alternate delimiter, or other composition-specific source syntax is added.

## Approval provenance

After reviewing the complete D150 research packet and the proposed
`c"...{expression}..."` candidate, the project owner explicitly rejected that
new source form and selected the existing `+` spelling for now, while leaving
other sugar ideas for possible later reconsideration outside the current spec.

```text
SELECTED_CANDIDATE=D150-S0
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```

The “for now” boundary is deliberate: it permits future re-evaluation but does
not pre-approve any syntax, delimiter, interpolation model, conversion rule, or
lowering.

## Preserved language contract

D150 preserves all applicable D141 and String invariants:

```text
D141_COMPOSITION_BEFORE_SUGAR=PRESERVED
D141_STRICT_CONCAT=PRESERVED
D141_CONCAT_NOT_REPEATED_PLUS=PRESERVED
STANDARD_STRING_PLUS=PRESERVED
TEXTUAL_CONVERSION_SEPARATE=PRESERVED
ORDINARY_STRING_LITERAL_SEMANTICS=PRESERVED
LITERAL_DOLLAR_BRACE_TEXT=PRESERVED
BACKSLASH_ESCAPE_ROLE=PRESERVED
```

In particular:

- standard String `+` remains strict binary ordinary message behavior;
- standard `String.concat` remains strict receiver-oriented aggregate
  composition;
- `+` and `concat` remain independent selectors;
- no implicit textual conversion is introduced;
- `${...}` inside ordinary Strings remains literal text;
- no adjacent String-literal concatenation is introduced;
- no new String literal/source category is introduced.

## Why S0

The research established genuine readability friction in String-heavy rendering
code, but it also established that no architectural urgency requires syntax now.

Existing `+` is already concise and directly understandable:

```protos
"[" + phaseName + "] " + countText(completed) + "/" + countText(total)
```

`concat` supplies the aggregate semantic operation where that shape is useful.

Adding a dedicated composition expression would permanently charge the language
and tooling with another lexer/parser mode, delimiter/escape rules, highlighting,
formatting, source mapping, diagnostics, documentation, and compatibility
surface.

The cost of deferral is low: future sugar can be added additively without
rewriting String identity, `+`, `concat`, persisted data, or runtime architecture.

Therefore S0 is not underengineering. It is the smallest current design.

## Rejected current proposal

The packet's recommended experimental candidate was:

```protos
c"hello {name}"
```

with mandatory lowering to one ordinary `concat` call.

The owner did not approve that source category. It is not reserved and has no
special meaning in the current language.

This rejection does not prove that every future composition syntax is wrong. It
only records that D150 selects no new sugar now.

## Future reconsideration boundary

A future decision may reconsider interpolation or another composition notation
if real use justifies the permanent grammar/tooling cost.

Such work must start as a new explicit decision and must independently resolve:

- exact syntax and compatibility;
- evaluation/failure semantics;
- whether lowering targets `+`, `concat`, or another approved mechanism;
- textual-conversion boundaries;
- multiline/raw/formatting interactions;
- tooling and source-generation costs.

No candidate discussed by D150 receives a syntax reservation or compatibility
promise.

```text
FUTURE_RECONSIDERATION_ALLOWED=YES
FUTURE_RECONSIDERATION_PREAPPROVED=NO
```

## Normative publication

None is required because S0 changes no observable Protos behavior.

```text
SPECIFICATION_CHANGED=NO
CURRENT_SPEC_SYNTAX_DELTA=NONE
CURRENT_SPEC_SEMANTIC_DELTA=NONE
PROTOS_REVISION=NOT_APPLICABLE_FOR_D150
```

The D141 specification revision `0.1.424` remains the relevant normative String
composition baseline.

## Closure state

D150 is complete as a decision. There is no D150 implementation owner because
the selected result introduces no feature to implement.

I047 remains independently responsible only for implementation of the already
ratified D141 `String.concat` semantics.

```text
D150_STATUS=RATIFIED
SELECTED_CANDIDATE=S0
IMPLEMENTATION_REQUIRED=NO
D150_IMPLEMENTATION_OWNER=NOT_APPLICABLE
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
```
