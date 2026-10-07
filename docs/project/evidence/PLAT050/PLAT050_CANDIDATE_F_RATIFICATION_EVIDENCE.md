# PLAT050 — Candidate F ratification evidence

Evidence date: **2026-10-04**

Decision Issue: `guillermomolina/protos#792`

Parent workstream: `LM011 / guillermomolina/protos#670`

Governing public decision: `D183 / guillermomolina/protos#791`

Maintained decision record:

`docs/project/decisions/platform/PLAT050_CANONICAL_FORMATTER_SOURCE_TRIVIA_AUTHORITY.md`

## Explicit project-owner selection

The project owner explicitly selected the completed PLAT050 recommendation:

~~~text
Apruebo candidate F.
~~~

The selected candidate is exactly:

~~~text
Candidate F
=
on-demand hybrid source-layout view
+
exact bundled Protos formatter policy
+
tool-neutral host source mechanism
~~~

The approval applies to the decision packet presented immediately before this selection. It does not authorize an unpresented lossless CST, host-owned formatter policy, TypeScript formatter, range formatting, check mode, parser recovery, style configuration or user-module execution.

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=F
~~~

## Product and project authority

Final product authority reviewed before publication:

~~~text
PROTOS_REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
PROTOS_VSCODE_EXTENSION_REVISION=6332f34f4a581d91771bae1a6b173b6e796ddffe
PROJECT_DOCS_BASE_REVISION=9f6e73056359eb65de75bad4b1a3da0e05626fca
D183_PROJECT_RECORD_REVISION=e6f56757e9191286b5a4fd6f1980cd9e43d995eb
~~~

PLAT050 was initially opened against Protos `37bab1b034e87b79ff4475a3e4013cfb5ef3fd73`.

Before ratification, Protos moved to `a38470bc6e2f68e770ddc8054053995bb2477b19`.

The intervening product change is TOOL001-F2E5 package-run composition. It changes Package Tool/external-package execution surfaces, implementation version and associated tests/metadata. It does not change the PLAT050 dependency surfaces revalidated before publication:

~~~text
spec/PROTOS_GRAMMAR.md
docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md
src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java
src/main/java/com/guillermomolina/protos/lexer/TokenOccurrence.java
src/main/java/com/guillermomolina/protos/source/SourceSpan.java
src/main/java/com/guillermomolina/protos/parser/ProtosParser.java
src/main/java/com/guillermomolina/protos/parser/ast/SurfaceSequence.java
src/main/java/com/guillermomolina/protos/parser/ast/SurfaceClosure.java
src/main/java/com/guillermomolina/protos/parser/ast/SurfaceCall.java
src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java
src/main/java/com/guillermomolina/protos/analysis/ProtosDocumentSnapshot.java
src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java
src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java
src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java
src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java
~~~

The D183 maintained decision and ratification-evidence blobs also remained unchanged while project docs advanced for unrelated TOOL001 evidence.

~~~text
MOVING_HEAD_REVIEW=PASS
PLAT050_REINVESTIGATION_REQUIRED=NO
~~~

## Investigation result retained

The current source pipeline already provides:

~~~text
exact immutable source
+
TokenOccurrence / SourceSpan
+
Surface AST
~~~

The demonstrated gaps are narrow:

~~~text
block comments and horizontal trivia are not retained
line comments are observer-only
comment attachment is not retained
SurfaceSequence erases semicolon-vs-newline separator origin
SurfaceCall does not retain trailing-Closure source origin
selected Closure parameter spelling origin is not retained
~~~

The investigation found no current requirement forcing a second lossless syntax tree.

## Selected boundary

Candidate F selects:

~~~text
EXACT_RAW_SOURCE_AUTHORITY=IMMUTABLE_INPUT_SOURCE

SURFACE_AST_ROLE=VALID_SOURCE_STRUCTURAL_PARSE_AUTHORITY
TOKEN_OCCURRENCE_ROLE=LEXICAL_KIND_AND_SOURCE_EXTENT

TRIVIA_AUTHORITY=
    ON_DEMAND_TOOLING_ONLY_SPAN_BASED_OCCURRENCES

PARSER_SOURCE_FACTS=
    SEPARATOR_KIND
    + SINGLE_PARAMETER_CLOSURE_FORM
    + TRAILING_CLOSURE_ORIGIN

COMMENT_ATTACHMENT=
    DETERMINISTIC_DERIVATION
    + IMMUTABLE_SOURCE_VIEW_RETENTION

LOSSLESS_SYNTAX_LAYER_REQUIRED=NO
FULL_LOSSLESS_CST_REQUIRED=NO

FORMATTER_POLICY_OWNER=
    EXACT_TOOLCHAIN_BUNDLED_PROTOS_FORMATTER

HOST_POLICY_OWNER=NO
USER_MODULE_EXECUTION_REQUIRED=NO
~~~

## GITHUB021 invariant/delta consistency

No previously approved D183 or PLAT024 invariant is changed by Candidate F.

~~~text
D183_FIXED_PROJECT_STYLE=PRESERVED
D183_CONSERVATIVE_LEXICAL_PRESERVATION=PRESERVED
D183_COMMENT_ATTACHMENT_REQUIREMENT=PRESERVED
D183_LITERAL_RAW_SPELLING=PRESERVED
D183_SEPARATOR_KIND_PRESERVATION=PRESERVED
D183_TRAILING_CLOSURE_PRESERVATION=PRESERVED
D183_FAIL_CLOSED_INVALID_SOURCE=PRESERVED
D183_DETERMINISM=PRESERVED
D183_IDEMPOTENCE=PRESERVED
D183_RANGE_FORMATTING_DEFERRED=PRESERVED
D183_CHECK_MODE_DEFERRED=PRESERVED

PLAT024_THIN_LSP_BOUNDARY=PRESERVED
VSCODE_SIDE_FORMATTER=NO
TOOLCHAIN_TOOL_ARCHITECTURE=PRESERVED
JAVA_NATIVE_FORMATTER_POLICY_INSTITUTION=NO
PROJECT_PACKAGE_RESOLUTION_FOR_FORMATTER_BOOTSTRAP=NO

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

No invariant was reopened.

## Correctness obligations retained

Candidate F preserves the three independent D183 correctness obligations:

~~~text
parse(format(source)) succeeds

semanticNormalize(parse(format(source)))
==
semanticNormalize(parse(source))

selectedSourcePreservationInvariantsHold(source, format(source))
~~~

and exact idempotence:

~~~text
format(format(source)) == format(source)
~~~

The semantic comparison is non-executing and source-position-free.

The source-preservation projection retains raw literals, identifier code points, comments/order/attachment, explicit grouping, separator kind, Closure source forms, trailing-Closure relationships and Array/Map surface forms.

Any failed correctness postcondition is fail closed with no edits.

## Pay-for-what-you-need result

The selected source-layout metadata is on demand.

~~~text
ORDINARY_PROGRAM_FORMATTER_TRIVIA_ALLOCATION=0
ORDINARY_PROGRAM_FORMATTER_ATTACHMENT_CONSTRUCTION=0
ORDINARY_PROGRAM_FORMATTER_SOURCE_FACT_RETENTION=0
~~~

A format request pays for source-view construction, formatting and postcondition validation.

JVM/Native distributions may pay some additional code/image size for the bundled formatter/source mechanism. Exact magnitude remains implementation evidence, not a semantic/runtime-state requirement.

## Strongest retained counterargument

The hybrid maintains several coordinated views rather than one lossless syntax tree.

If the parser-source-fact catalogue grows materially with future syntax/tooling requirements, Candidate F can become an implicit CST with poor maintenance properties.

That is the ratified migration trigger: stop extending side facts and reconsider a dedicated lossless syntax layer when the current small fact set is no longer conceptually smaller.

## Implementation release

Ratification releases:

~~~text
SLICE=LM011-B1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
LM011_B_RELEASED=YES
~~~

LM011-B1 is the on-demand source-layout-view and source-preservation-authority foundation.

It does not select CLI spelling or any deferred D183 capability.

## Result

~~~text
PLAT050_STATUS=RATIFIED
SELECTED_CANDIDATE=F_ON_DEMAND_HYBRID_PLUS_BUNDLED_PROTOS_POLICY
GITHUB021_INVARIANT_DELTA_CHECK=PASS
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
LM011_B_RELEASED=YES
NEXT=LM011-B1
~~~
