# D183 — Candidate B ratification evidence

Evidence date: **2026-10-04**

Owning decision: `D183 / guillermomolina/protos#791`

Parent workstream: `LM011 / guillermomolina/protos#670`

Released architecture checkpoint: `PLAT050 / guillermomolina/protos#792`

Nature: immutable project evidence for owner selection, moving-HEAD review and
durable publication of the D183 formatter-policy decision.

## Explicit project-owner selection

The project owner explicitly selected:

~~~text
apruebo Candidate B.
~~~

Selected candidate:

~~~text
B — fixed structural style + conservative lexical preservation
~~~

The approval applies to the exact Candidate B contract presented in the D183
decision packet: one fixed Protos-owned structural style with conservative
preservation of comments, literal spellings and selected source forms; no
editor/project style configuration; fail-closed invalid-source behavior;
deterministic and idempotent output; and deferred range/check/recovery/refactoring
capabilities.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=B
~~~

## Product authority at ratification

LM011-A investigated:

~~~text
LM011_A_PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
LM011_A_VSCODE_EXTENSION_REVISION=6332f34f4a581d91771bae1a6b173b6e796ddffe
LM011_A_PROJECT_RECORD_REVISION=36b8d9f6be059d95096ec497f848174ee71eb038
~~~

Immediately before durable D183 ratification:

~~~text
PROTOS_REVISION=37bab1b034e87b79ff4475a3e4013cfb5ef3fd73
~~~

The product repository was two commits ahead of the LM011-A source audit.

The moving-HEAD comparison changed:

~~~text
CHANGELOG.md
pom.xml
protos/tools/package/ExecutionPlan.protos
src/main/java/com/guillermomolina/protos/execution/**
src/test/java/com/guillermomolina/protos/execution/**
tools/java_local_range_pe_reachability_baseline.json
~~~

It did **not** change D183's dependency surfaces:

~~~text
spec/PROTOS_GRAMMAR.md
spec/PROTOS_LANGUAGE_SPEC.md
src/main/java/com/guillermomolina/protos/lexer/TokenOccurrence.java
src/main/java/com/guillermomolina/protos/source/SourceSpan.java
src/main/java/com/guillermomolina/protos/parser/**
src/main/java/com/guillermomolina/protos/parser/ast/Surface*.java
src/main/java/com/guillermomolina/protos/analysis/ProtosDocumentSnapshot.java
src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java
docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md
~~~

Therefore the owner-selected candidate does not conflict with intervening current
HEAD state.

~~~text
MOVING_HEAD_REVIEW=PASS
LM011_A_REAUDIT_REQUIRED=NO
~~~

## Owner-approved invariants preserved

Candidate B preserves the fixed invariants established by D183:

~~~text
SPEC_AUTHORITY=spec/
PROTOS_SEMANTICS=UNCHANGED
TOKEN_SEPARATION=KEEP
DELIMITER_PRECEDENCE_ASSOCIATIVITY=KEEP
REQUIRED_GROUPING=KEEP
NEWLINE_SEMICOLON_LEGALITY=KEEP
CONTINUATION_BEHAVIOR=KEEP
COMMENT_LEXICAL_SEMANTICS=KEEP
STRING_VALUES=KEEP
NUMERIC_VALUES=KEEP
UNICODE_IDENTIFIER_VALIDITY=KEEP
TRIPLE_DOUBLE_STRING_VALUE_AND_INDENT_RULES=KEEP
USER_MODULE_EXECUTION_DURING_FORMATTING=NO
ONE_CANONICAL_FORMATTER_AUTHORITY=KEEP
PLAT024_BOUNDARY=KEEP
INDEPENDENT_TYPESCRIPT_FORMATTER=NO
LM012_FORMATTING_LINT_SEPARATION=KEEP
~~~

Candidate B additionally selects conservative source preservation rather than
semantic-source respelling:

~~~text
COMMENT_TEXT_REFLOW=NO
STRING_LITERAL_RESPELLING=NO
NUMERIC_LITERAL_RESPELLING=NO
EXPLICIT_PARENTHESIS_REMOVAL=NO
SEMICOLON_NEWLINE_NORMALIZATION=NO
CLOSURE_SOURCE_FORM_NORMALIZATION=NO
ARRAY_MAP_DESUGARING_DURING_FORMATTING=NO
~~~

~~~text
DECISION_INVARIANT_CONSISTENCY=PASS
OWNER_INVARIANT_REOPEN_REQUIRED=NO
~~~

## Ratified policy summary

~~~text
FORMATTER_STYLE=FIXED_PROJECT_OWNED

INDENTATION=4_SPACES
CONTINUATION_INDENT=4_SPACES
COLUMN_ALIGNMENT=NO
LINE_WIDTH=SOFT_100

COMMENTS=RAW_TEXT_PRESERVED
COMMENT_REFLOW=NEVER

STRING_SPELLING=PRESERVE_EXACT
NUMERIC_SPELLING=PRESERVE_EXACT
UNICODE_SPELLING=PRESERVE_EXACT_VALID_SOURCE
EXPLICIT_PARENTHESES=PRESERVE
SEMICOLON_VS_NEWLINE=PRESERVE_SOURCE_KIND
CLOSURE_SOURCE_FORM=PRESERVE
TRAILING_CLOSURE_SOURCE_FORM=PRESERVE
ARRAY_MAP_SURFACE_FORM=PRESERVE

STRUCTURAL_OUTPUT_EOL=LF
PRESERVED_PAYLOAD_EOL=EXACT
FINAL_NEWLINE=EXACTLY_ONE_LF

INVALID_SOURCE=FAIL_CLOSED_NO_EDITS
PARSE_ERROR=FAIL_AND_RETURN_NO_EDITS

DETERMINISM=REQUIRED
IDEMPOTENCE=REQUIRED

LSP_TAB_SIZE=IGNORED_AS_STYLE_AUTHORITY
LSP_INSERT_SPACES=IGNORED_AS_STYLE_AUTHORITY

PROJECT_STYLE_CONFIGURATION=NO
EDITOR_STYLE_CONFIGURATION=NO

RANGE_FORMATTING=DEFERRED
CHECK_MODE=DEFERRED
PARSER_RECOVERY=DEFERRED
COMMENT_REFLOW=DEFERRED
SEMANTIC_REFACTORING=DEFERRED
~~~

## Correctness model

The ratified contract keeps three distinct obligations:

~~~text
1. parse(format(source)) succeeds

2. semanticNormalize(parse(format(source)))
   ==
   semanticNormalize(parse(source))

3. selectedSourcePreservationInvariantsHold(source, format(source))
~~~

The semantic comparison is non-executing and must not be direct equality of
Java AST records containing `SourceSpan`.

## Architecture consequence

D183 does not select implementation architecture.

The selected contract requires:

~~~text
EXACT_RAW_SOURCE=YES
SURFACE_AST=YES
TOKEN_OCCURRENCES_SOURCE_SPANS=YES
TRIVIA_COMMENT_AUTHORITY=YES_IN_SOME_FORM
LOSSLESS_CST=NOT_PROVEN_REQUIRED
RAW_LITERAL_SPELLING=YES
MINIMAL_EDIT_SUPPORT=NO_CURRENT_REQUIREMENT
PARSER_RECOVERY=NO_CURRENT_REQUIREMENT
USER_MODULE_EXECUTION=NO
~~~

The exact architecture question is now owned by:

~~~text
PLAT050
guillermomolina/protos#792
Canonical formatter source/trivia authority and tooling bridge
~~~

PLAT050 remains investigation-only and must stop for explicit owner approval.

## Publication result

Canonical durable decision record:

~~~text
docs/project/decisions/tooling/
D183_CANONICAL_SOURCE_FORMATTING_POLICY_AND_PRESERVATION_CONTRACT.md
~~~

This evidence record:

~~~text
docs/project/evidence/D183/
D183_CANDIDATE_B_RATIFICATION_EVIDENCE.md
~~~

~~~text
D183_STATUS=RATIFIED
D183_SELECTED_CANDIDATE=B
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
SPECIFICATION_CHANGE_REQUIRED=NO
LM011_B_RELEASED=NO
NEXT=PLAT050/#792
IMPLEMENTATION_AUTHORIZED=NO
~~~
