# D183 — Canonical source formatting policy and preservation contract

Status: **RATIFIED — Candidate B selected**

Explicit project-owner approval: **2026-10-04**

Decision issue: `guillermomolina/protos#791`

Parent work item: `LM011 / guillermomolina/protos#670`

Released architecture decision: `PLAT050 / guillermomolina/protos#792`

Product baseline used for ratification:

~~~text
PROTOS_REVISION=37bab1b034e87b79ff4475a3e4013cfb5ef3fd73
LM011_A_AUDIT_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
LM011_A_PROJECT_RECORD_REVISION=36b8d9f6be059d95096ec497f848174ee71eb038
~~~

Nature: durable non-normative implementation-independent tooling decision.

Observable Protos language effect: **none**.

Specification change required: **none**.

## Decision

D183 selects **Candidate B — fixed structural style + conservative lexical preservation**.

The canonical Protos formatter owns one deterministic project style for valid
ordinary source. It canonicalizes structural layout while conservatively
preserving lexical/source choices that are not required to change for valid
formatting.

The selected contract intentionally distinguishes:

~~~text
canonical source formatting
    !=
canonical semantic source serialization
~~~

For one exact valid input source and one formatter-policy/tool version, the
formatter produces exactly one output. Semantically equivalent but differently
spelled valid source may remain textually distinct where D183 selects
preservation.

The formatter never executes the user's module.

## Established source model

LM011-A remains authoritative:

~~~text
CURRENT_FORMATTER=NONE

CURRENT_SOURCE_MODEL=
  exact immutable source text
  + token occurrences / SourceSpan
  + Surface AST

SURFACE_AST_ALONE_SUFFICIENT=NO
TOKEN_STREAM_ALONE_SUFFICIENT=NO
FULL_LOSSLESS_CST_REQUIRED=NOT_PROVEN
CANONICAL_AST_AS_FORMATTER_SOURCE=REJECTED
~~~

D183 adds one architectural requirement without selecting its representation:
the implementation must have a faithful editor-neutral comment/trivia authority
sufficient to preserve source order, attachment class and grammar-significant
separator/newline boundaries.

Whether that authority is an extension of the current hybrid, a lossless syntax
layer, a CST, token trivia, source gaps or another representation is deferred to
PLAT050.

## Canonical structural style

### Indentation

~~~text
INDENTATION_WIDTH=4
INDENTATION_CHARACTER=SPACE
GENERATED_TAB_INDENTATION=NO
~~~

Formatter-generated structural indentation is four SPACE characters per level.

TAB characters inside preserved lexical payloads, including String/comment
content, are not rewritten merely to satisfy indentation style.

### Continuation indentation

Continuation uses one additional structural indentation level:

~~~text
CONTINUATION_INDENT=4_SPACES
COLUMN_ALIGNMENT=NO
~~~

Representative form:

~~~protos
result: first +
    second

value =
    computeSomething()

items
    .filter(predicate)
    .map(transform)
~~~

Indentation is presentation only and never changes the existing grammar's
continuation semantics.

### Braces

Opening braces remain on the owning construct's source line when the construct
is rendered on one logical line before the body.

Representative forms:

~~~protos
dog: animal {
    name: "Rex"
}

x => {
    x * 2
}

foo() {
    body()
}
~~~

A multiline closing brace is aligned with its owner.

A formatter must never introduce a separating logical `NEWLINE` between a
completed call and an attached trailing Closure.

### Spacing

Canonical ordinary horizontal spacing:

~~~text
binary operators      one SPACE on each side
:                     no SPACE before, one SPACE after
=                     one SPACE on each side
=>                    one SPACE on each side
comma                 no SPACE before, one SPACE after on same line
semicolon             no SPACE before, one SPACE after on same line
call '('              no SPACE before
index '['             no SPACE before
member '.'            no SPACE around
unary ! / -           no SPACE after
^                     no SPACE after
...                   no SPACE after
inside (), []         no padding spaces
~~~

For an ordinary non-empty construct kept on one line, braces use one interior
SPACE when that spacing is structural rather than lexical payload:

~~~protos
{ x: 1 }
%{ key: value }
foo() { body() }
~~~

Empty structural forms are compact:

~~~protos
{}
%{}
[]
()
() => {}
~~~

### Line width

~~~text
LINE_WIDTH_POLICY=SOFT_100_COLUMNS
~~~

100 columns is a canonical layout target, not a correctness limit.

A line may exceed 100 when shortening it would require changing a selected
source-preservation invariant, splitting an opaque literal, rewriting preserved
comment text, violating grammar continuation/attachment rules or producing a
less stable structural form.

### Wrapping

Deterministic structural wrapping follows these preferences:

1. use the canonical flat form when it fits the soft width and no preservation
   boundary forces a break;
2. otherwise break only at syntactically valid structural points;
3. recursively format nested constructs;
4. never introduce a break that changes parse, trailing-Closure attachment,
   source-preservation invariants or literal value.

Preferred structural break locations are:

~~~text
comma-separated lists -> one item per continuation line
binary chains         -> break after the binary operator
: / = / =>            -> break after the operator
member chains         -> break before leading '.'
delimited constructs  -> opening delimiter, indented content, closing delimiter
~~~

Protos trailing commas remain invalid. A vertical comma-separated list therefore
keeps commas after each non-final item only.

### Blank lines

For ordinary source layout outside preserved lexical payloads:

~~~text
0 blank lines  -> 0
1 blank line   -> 1
2+ blank lines -> 1
~~~

Leading ordinary blank lines at the beginning of the file are removed.

Trailing ordinary blank lines are removed before the canonical final newline.

Inside structural delimiters, gratuitous leading/trailing blank runs are removed
unless a preserved comment relationship requires the separation.

Blank-line normalization never rewrites String/comment payload.

## Source-form preservation

Formatting is not desugaring or semantic source canonicalization.

The formatter preserves, when present in valid source:

~~~text
explicit parenthesized grouping
semicolon-vs-logical-newline separator kind
Array construction syntax
Map construction syntax
expression-bodied Closure vs braced Closure
single-parameter Closure source spelling
trailing-Closure source form
String literal spelling
numeric literal spelling
Unicode source spelling
comment text/order/attachment class
~~~

### Semicolon vs newline

A `;` and a separating logical `NEWLINE` are not normalized into each other
merely for style.

Thus:

~~~protos
a(); b()
~~~

remains semicolon-separated, while:

~~~protos
a()
b()
~~~

remains newline-separated.

The same preservation rule applies to object-body and Map sequence positions
where the grammar admits those distinct source mechanisms.

### Parentheses

All explicit valid parenthesized grouping is preserved, including redundant but
semantically harmless grouping.

The formatter does not remove parentheses merely because parsing proves them
unnecessary.

Parentheses may be added only when necessary to preserve the exact parse during
an otherwise-authorized formatting transformation.

## Comments

### Raw text

Comment text is not reflowed.

For each comment occurrence preserve:

~~~text
delimiter spelling
comment body characters
SPACE/TAB inside the body
internal block-comment physical newlines
source ordering
~~~

The formatter may canonicalize ordinary whitespace around a comment only when
doing so preserves the selected attachment and grammar behavior.

### Attachment

The architecture must be able to preserve at least:

~~~text
OWN_LINE
END_OF_LINE
EMBEDDED_BETWEEN_TOKENS
~~~

and must preserve the comment's nearest relevant structural relationship plus
any grammar-significant separator boundary.

In particular:

~~~protos
a() // comment
b()
~~~

keeps the line comment attached to `a()` and retains the separating logical
newline.

For a block comment between a completed call and its trailing Closure, the
formatter must not manufacture a logical newline after the block comment that
would detach the Closure.

## Literals and Unicode

### Ordinary Strings

The exact raw literal spelling is preserved.

Examples that remain distinct:

~~~protos
'a'
"a"
"\n"
"\u{0A}"
~~~

No quote normalization, escape normalization or equivalent-value respelling is
part of baseline canonical formatting.

### Triple-double Strings

A triple-double String is a locked lexical payload.

The formatter must not alter any source character that can affect its value or
structural indentation contract, including:

~~~text
LF vs CR vs CRLF retained in content
structural SPACE/TAB prefix
additional indentation
blank-line whitespace
same-line vs multiline delimiter placement
escape spelling
~~~

### Numerics

Preserve exact valid source spelling, including radix, prefix case, digit
separators, leading zeroes and exponent spelling.

Examples may remain distinct:

~~~protos
1000
1_000
0xff
0xFF
2e3
2E3
~~~

### Unicode

Preserve exact valid source code points.

The formatter never silently Unicode-normalizes or repairs an identifier.
Current Protos validity already requires the normative NFC rule; invalid source
is handled by the fail-closed formatting policy.

String/comment Unicode content is likewise not normalized.

## Physical line endings and final newline

Formatter-owned structural/output line endings are:

~~~text
LF
~~~

Physical newlines inside preserved lexical payloads remain exact, especially
inside triple-double String content and raw-preserved multiline comments.

The complete formatted file ends with exactly one formatter-owned `LF`
outside lexical payload.

Trailing ordinary horizontal whitespace is removed.

A formatted file may therefore still contain non-LF characters inside preserved
payloads; this is intentional and required by source preservation.

## Invalid and incomplete source

Baseline whole-document formatting is fail closed.

If lexing, parsing or required static syntax validation fails:

~~~text
FORMATTED_EDITS=NONE
ORIGINAL_SOURCE=UNCHANGED
FORMAT_RESULT=FAILURE
~~~

D183 does not select:

- parser recovery;
- partial formatting of apparently valid regions;
- error-tolerant formatting;
- guessed syntactic repair.

An editor may therefore leave a momentarily incomplete document unformatted.

The exact LSP/UI presentation of that failure is integration policy, not a
license to modify invalid source.

## Determinism

For a fixed formatter-policy/tool version:

~~~text
formatV(source) = exactly one deterministic output
~~~

Output does not depend on:

~~~text
OS
locale
editor
workspace preference
LSP tabSize
LSP insertSpaces
host line-separator convention
current time
filesystem traversal order
runtime scheduling
~~~

## Idempotence

Exact idempotence is required:

~~~text
formatV(formatV(source)) == formatV(source)
~~~

The equality includes exact source characters for:

~~~text
structural line endings
final newline
comments
String spelling
numeric spelling
Unicode spelling
parentheses
selected separator kind
selected source forms
~~~

## Correctness relation

Formatter correctness has three independent layers.

### Parse validity

~~~text
parse(format(source)) succeeds
~~~

Necessary but insufficient.

### Semantic equivalence

Abstractly:

~~~text
semanticNormalize(parse(format(source)))
==
semanticNormalize(parse(source))
~~~

The implementation's equivalence relation must be non-executing and compare
language meaning after mandatory syntactic desugaring while erasing source
position metadata.

It must preserve at least:

~~~text
evaluation order
receiver/member-call distinctions
slot creation vs assignment
control-flow distinctions
literal values under exact Protos semantics
~~~

Direct equality of current Java AST records containing `SourceSpan` is not the
definition of semantic equivalence.

The canonical semantic AST may be used as evidence where appropriate, but it is
not source-reconstruction authority.

### Source-preservation invariants

Additionally:

~~~text
selectedSourcePreservationInvariantsHold(source, format(source))
~~~

Candidate B requires this relation to cover at least:

~~~text
raw String spelling
raw numeric spelling
identifier code points
comment text/order/attachment
explicit grouping
semicolon-vs-newline separator kind
Closure body source form
trailing-Closure form
Array/Map construction source forms
~~~

## LSP formatting options

Standard LSP `FormattingOptions` values are protocol inputs, not Protos style
authority.

~~~text
LSP_TAB_SIZE_POLICY=ACCEPT_BUT_IGNORE_AS_CANONICAL_STYLE_AUTHORITY
LSP_INSERT_SPACES_POLICY=ACCEPT_BUT_IGNORE_AS_CANONICAL_STYLE_AUTHORITY
~~~

A client asking for tab indentation or a different tab size does not change
canonical Protos output.

The adapter must not silently translate editor preferences into an unapproved
Protos formatter configuration.

Other optional LSP whitespace/final-newline preferences likewise cannot override
the canonical project policy.

## Formatter configuration model

Baseline D183 formatting is fixed-style.

~~~text
FORMATTER_CONFIGURATION_MODEL=FIXED_PROJECT_OWNED_STYLE
PROJECT_STYLE_CONFIG=NO
EDITOR_STYLE_CONFIG=NO
STYLE_PRESETS=NO
~~~

If a future requirement justifies configuration, that changes the public
formatter contract from conceptually:

~~~text
source + formatter-policy-version -> output
~~~

to:

~~~text
source + formatter-policy-version + configuration -> output
~~~

That later change requires an explicit decision; no speculative configuration
machinery is installed by D183.

## Style evolution

D183 distinguishes:

~~~text
1. Protos source semantics
2. canonical formatting policy
3. formatter implementation/tool version
4. exact formatted bytes
~~~

A formatter-style change does not by itself change Protos language semantics.

D183 does not promise byte-identical output across every future formatter
release forever.

Intentional style evolution must be explicit and documented. A consumer that
needs historical byte reproducibility pins the applicable Protos tool version.

A separate formatter-style-edition/version protocol is deferred until a real
compatibility requirement justifies concurrent historical/current style
policies.

## Explicitly deferred capability

~~~text
RANGE_FORMATTING=DEFERRED
CHECK_MODE=DEFERRED
ON_TYPE_FORMATTING=DEFERRED
FORMATTER_DISABLE_DIRECTIVES=DEFERRED
PROJECT_STYLE_CONFIGURATION=DEFERRED
ERROR_TOLERANT_FORMATTING=DEFERRED
PARTIAL_VALID_REGION_FORMATTING=DEFERRED
PARSER_RECOVERY=DEFERRED
COMMENT_REFLOW=DEFERRED
LITERAL_CANONICALIZATION=DEFERRED
REDUNDANT_PARENTHESIS_REMOVAL=DEFERRED
SEMANTIC_REFACTORING=DEFERRED
MINIMAL_EDIT_GUARANTEE=DEFERRED
FORMATTER_STYLE_EDITION_PROTOCOL=DEFERRED
EXACT_CLI_SPELLING=DEFERRED
~~~

## Compatibility boundaries

### PLAT024

~~~text
PLAT024_COMPATIBILITY=PASS
~~~

VS Code and other editors consume the formatter through the existing thin
standard-LSP boundary. No independent TypeScript Protos formatter is authorized.

### LM012

~~~text
LM012_SEPARATION=PASS
~~~

Formatting remains distinct from lint/style diagnostics. Formatter output does
not automatically define a lint violation or warning policy.

### Toolchain Tool Architecture

~~~text
BUNDLED_TOOL_ARCHITECTURE_COMPATIBILITY=PASS_PARTIAL
~~~

The common architecture remains authoritative:

- one public `protos` driver;
- small host/bootstrap;
- higher-level official tool policy primarily as bundled Protos tooling where
  naturally expressible;
- avoid a Java/native `FormatterPolicy` institution merely for convenience.

D183 intentionally does not decide the exact formatter host/bundled bridge.

## Architecture consequences

The selected contract classifies implementation needs as follows:

~~~text
EXACT_RAW_SOURCE=REQUIRED
SURFACE_AST=REQUIRED
TOKEN_OCCURRENCES_SOURCE_SPANS=REQUIRED
TRIVIA_COMMENT_REPRESENTATION=REQUIRED_IN_SOME_FORM
FULL_LOSSLESS_CST=NOT_PROVEN_REQUIRED
LITERAL_RAW_SPELLING=REQUIRED
MINIMAL_EDIT_SUPPORT=NOT_REQUIRED_BY_PUBLIC_CONTRACT
PARSER_RECOVERY=NOT_REQUIRED
USER_MODULE_EXECUTION=FORBIDDEN
~~~

These are requirements on PLAT050's architecture investigation, not selections
of concrete representations.

## Strongest counterargument

Candidate B deliberately permits semantically equivalent Protos programs to
remain textually distinct after canonical formatting.

For example, equivalent String values may preserve different quote/escape
spellings, and semantically equivalent sequences may preserve `;` versus
logical-newline source choice.

If Protos wanted a canonical textual serialization of program semantics,
Candidate A would be more convergent.

That is not the current requirement. LM011 requires a source formatter for
human/editor/tool reuse. Destroying lexical information now would be
irreversible, while stronger canonicalization can be introduced later if a
real semantic-serialization/content-addressing requirement appears.

## Invariant and moving-HEAD review

LM011-A audited:

~~~text
PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
~~~

Immediately before ratification, current Protos HEAD was:

~~~text
PROTOS_REVISION=37bab1b034e87b79ff4475a3e4013cfb5ef3fd73
~~~

The two intervening commits changed package execution/preflight and
PERF030 execution machinery plus tests/metadata. They did not change the
surfaces on which D183 depends:

~~~text
spec/PROTOS_GRAMMAR.md=UNCHANGED
spec/PROTOS_LANGUAGE_SPEC.md=UNCHANGED
lexer/token occurrence/source span authorities=UNCHANGED
Surface AST=UNCHANGED
parser=UNCHANGED
document snapshot=UNCHANGED
language-server formatting capability=UNCHANGED
VS Code thin-client boundary=UNCHANGED
TOOLCHAIN_TOOL_ARCHITECTURE.md=UNCHANGED
~~~

Therefore:

~~~text
LM011_A_REAUDIT_REQUIRED=NO
D183_CANDIDATE_B_DELTA=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
OWNER_INVARIANT_REOPEN_REQUIRED=NO
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
SPECIFICATION_CHANGE_REQUIRED=NO
~~~

## Implementation routing

D183 does **not** release LM011-B directly.

It releases the architecture checkpoint:

~~~text
NEXT=PLAT050
ISSUE=guillermomolina/protos#792
TITLE=Canonical formatter source/trivia authority and tooling bridge
TYPE=INVESTIGATION
IMPLEMENTATION_AUTHORIZED=NO
LM011_B_RELEASED=NO
~~~

PLAT050 must decide the smallest editor-neutral representation and
host/bundled-tool/LSP/CLI authority bridge capable of implementing this exact
D183 contract.

Only after PLAT050 is explicitly approved and durably ratified may LM011-B
become mechanical implementation work.

## Ratified result

~~~text
D183_STATUS=RATIFIED

CURRENT_FORMATTER=NONE
CURRENT_SOURCE_MODEL=EXACT_RAW_SOURCE + TOKEN_OCCURRENCES/SOURCE_SPANS + SURFACE_AST
MECHANICALLY_REQUIRED_FORMATTING=TOKEN_SEPARATION + DELIMITER_VALIDITY + PRECEDENCE/GROUPING + SIGNIFICANT_NEWLINE/SEMICOLON_RULES + CONTINUATION_RULES + COMMENT_LEXICAL_RULES + EXACT_STRING_VALUES + NUMERIC_VALUES + UNICODE_IDENTIFIER_VALIDITY
PUBLIC_POLICY_SURFACE=FIXED_STRUCTURAL_STYLE + CONSERVATIVE_LEXICAL_PRESERVATION + FAIL_CLOSED_INVALID_SOURCE + DETERMINISM + IDEMPOTENCE + FIXED_LSP_INDEPENDENT_STYLE

SELECTED_CANDIDATE=B_FIXED_STRUCTURAL_STYLE_PLUS_CONSERVATIVE_LEXICAL_PRESERVATION

INDENTATION_POLICY=4_SPACES_PER_STRUCTURAL_LEVEL
TABS_SPACES_POLICY=GENERATE_SPACES_ONLY_EXCEPT_PRESERVED_PAYLOAD
CONTINUATION_INDENT_POLICY=ONE_ADDITIONAL_4_SPACE_LEVEL
BRACE_POLICY=OPEN_ON_OWNER_LINE_AND_PRESERVE_TRAILING_CLOSURE_ATTACHMENT
SPACING_POLICY=RATIFIED_AS_ABOVE
LINE_WIDTH_POLICY=SOFT_100
WRAPPING_POLICY=DETERMINISTIC_STRUCTURAL_FIT_WITH_PRESERVATION_OVERRIDE
BLANK_LINE_POLICY=PRESERVE_ZERO_OR_ONE_AND_COLLAPSE_GREATER_RUNS
PARENTHESIS_POLICY=PRESERVE_ALL_EXPLICIT_GROUPING

COMMENT_PRESERVATION_POLICY=PRESERVE_RAW_TEXT_ORDER_AND_ATTACHMENT_CLASS
COMMENT_REFLOW_POLICY=NEVER
STRING_SPELLING_POLICY=PRESERVE_EXACT_RAW_LITERAL
NUMERIC_SPELLING_POLICY=PRESERVE_EXACT_RAW_LITERAL
LINE_ENDING_POLICY=FORMATTER_STRUCTURAL_LF_WITH_PRESERVED_LITERAL_COMMENT_PAYLOAD_EOL
FINAL_NEWLINE_POLICY=EXACTLY_ONE_LF
UNICODE_SOURCE_POLICY=PRESERVE_EXACT_VALID_CODEPOINTS

INVALID_SOURCE_POLICY=FAIL_CLOSED_NO_EDITS
PARSE_ERROR_POLICY=FAIL_AND_LEAVE_ORIGINAL_SOURCE_UNCHANGED

DETERMINISM_REQUIRED=YES
IDEMPOTENCE_REQUIRED=YES
SEMANTIC_EQUIVALENCE_CONTRACT=NON_EXECUTING_SOURCE_SPAN_ERASED_SEMANTIC_NORMALIZATION_AFTER_MANDATORY_DESUGARING
SOURCE_PRESERVATION_CONTRACT=COMMENTS + LITERALS + IDENTIFIERS + GROUPING + SEPARATOR_KIND + CLOSURE_FORMS + ARRAY_MAP_FORMS

LSP_TAB_SIZE_POLICY=IGNORE_AS_STYLE_AUTHORITY
LSP_INSERT_SPACES_POLICY=IGNORE_AS_STYLE_AUTHORITY
FORMATTER_CONFIGURATION_MODEL=FIXED_PROJECT_OWNED_STYLE

RANGE_FORMATTING=DEFERRED
CHECK_MODE=DEFERRED

PLAT024_COMPATIBILITY=PASS
LM012_SEPARATION=PASS
BUNDLED_TOOL_ARCHITECTURE_COMPATIBILITY=PASS_PARTIAL

POST_D183_PLAT_DECISION_REQUIRED=YES
POST_D183_PLAT_DECISION=PLAT050/#792

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
IMPLEMENTATION_AUTHORIZED=NO
~~~

## References

- `guillermomolina/protos#791` — D183 decision issue.
- `guillermomolina/protos#670` — LM011 formatter workstream.
- `guillermomolina/protos#792` — PLAT050 architecture decision.
- `docs/project/evidence/LM011/LM011_A_FORMATTER_AUTHORITY_POLICY_AUDIT.md`.
- `docs/project/evidence/D183/D183_CANDIDATE_B_RATIFICATION_EVIDENCE.md`.
- `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md` in `guillermomolina/protos`.
