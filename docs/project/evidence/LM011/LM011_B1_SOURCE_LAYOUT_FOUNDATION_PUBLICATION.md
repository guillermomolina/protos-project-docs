# LM011-B1 — on-demand source-layout foundation publication

## Scope

This record preserves the published LM011-B1 implementation of the
source-layout and D183 preservation foundation released by D183 Candidate B and
PLAT050 Candidate F.

The owning live work item is `guillermomolina/protos#670` (**LM011 — Canonical
source formatter and editor formatting integration**). D183 / #791 and PLAT050
/ #792 remain the ratified policy and architecture authorities.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority, and it does not
claim that a canonical formatter is implemented yet.

## Published authority

```text
SLICE=LM011-B1
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=d49361af6fcda2815b89e9ae9ff22bc766ecf45d
COMMIT_SUBJECT=LM011-B1: add on-demand source layout preservation
PROTOS_VERSION=0.3.190-SNAPSHOT

D183_STATUS=RATIFIED
D183_SELECTED_CANDIDATE=B_FIXED_STRUCTURAL_STYLE_PLUS_CONSERVATIVE_LEXICAL_PRESERVATION
PLAT050_STATUS=RATIFIED
PLAT050_SELECTED_CANDIDATE=F_ON_DEMAND_HYBRID_PLUS_BUNDLED_PROTOS_POLICY

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
FORMATTER_TRANSFORMATION_IMPLEMENTED=NO
CLI_CHANGE=NO
LSP_CHANGE=NO
VSCODE_CHANGE=NO

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
MAINTAINER_REPORTED_MAKE_TEST=PASS
```

## Changed paths

The publication changes exactly these product-repository paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java
src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisCore.java
src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java
src/main/java/com/guillermomolina/protos/parser/ProtosParser.java
src/main/java/com/guillermomolina/protos/parser/ProtosParserSourceFacts.java
src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java
src/test/java/com/guillermomolina/protos/lexer/ProtosLexerSourceSpanTest.java
src/test/java/com/guillermomolina/protos/parser/ProtosParserSourceFactsTest.java
```

## On-demand trivia authority

`ProtosLexer` now exposes tooling-only trivia occurrences for:

```text
HORIZONTAL_WHITESPACE
LINE_COMMENT
BLOCK_COMMENT
```

Each trivia occurrence carries only its kind and `SourceSpan`; raw text remains
owned by the immutable source snapshot and is not copied into trivia metadata.

The ordinary tokenization path remains unchanged in authority and does not
retain these side facts. Line comments still leave their terminating newline as
a parser `NEWLINE` token. A block comment remains one trivia occurrence, and
newlines inside it are not emitted as parser newlines. The legacy line-comment
observer remains available for existing documentation extraction.

## Parser source facts

Tooling demand can now request parser-authored source facts without changing the
semantic Surface tree:

```text
sequence contexts:
  PROGRAM
  CLOSURE_BODY
  OBJECT_BODY
  MAP_CONSTRUCTION

sequence separators:
  SEMICOLON
  LOGICAL_NEWLINE

single-parameter Closure forms:
  BARE
  PARENTHESIZED

trailing Closure origin:
  SurfaceCall -> SurfaceClosure
```

Continuation newlines are not sequence separators.

The ordinary parser path does not retain formatter source facts. In particular,
sequence-newline parsing falls back to the existing newline consumption path
when no source-facts builder is present, avoiding formatter-only separator-span
allocation on ordinary parsing.

## Source-layout view

`ProtosStaticAnalysisCore.sourceLayout(...)` constructs an immutable,
editor-neutral `ProtosSourceLayoutView` only on explicit demand.

The view groups:

```text
exact immutable ProtosDocumentSnapshot
canonical TokenOccurrence list
canonical SurfaceSequence / Surface AST
opt-in trivia
opt-in parser source facts
comment attachments
source-position-independent structural paths
D183 preservation projection
```

It is not a CST, does not implement formatting policy, does not execute a user
module, and introduces no global source-layout cache.

## Structural paths and comment attachment

The source-layout view indexes existing Surface nodes with structural paths
rooted at `program`, using structural roles and list indices rather than source
offsets as identity.

Comment attachment classifies comments as:

```text
OWN_LINE
END_OF_LINE
EMBEDDED_BETWEEN_TOKENS
```

Attachments preserve nearest structural relationships and, where applicable,
explicit boundary authority for:

```text
SEQUENCE_SEPARATOR
TRAILING_CLOSURE
```

Sequence-boundary attachments retain `SEMICOLON` versus `LOGICAL_NEWLINE`.
Trailing-Closure attachment is derived from parser-authored trailing-Closure
origin rather than reconstructed as formatter policy.

## D183 preservation projection

`SourcePreservationProjection` exposes preservation evidence without making
absolute source offsets part of structural identity.

It carries immutable ordered projections for:

```text
raw token spelling
raw comment spelling
comment class and structural attachment
explicit grouping
Array construction form
Map construction form
Closure expression-body versus block-body form
semicolon versus logical-newline separators
bare versus parenthesized single-parameter Closure form
trailing-Closure relationships
```

Raw spellings are recovered from the exact immutable snapshot through existing
source spans. Structural relationships use Surface-tree paths.

The published tests cover raw String and numeric spellings, Unicode identifier
spelling, explicit `SurfaceGroup` reuse, Closure body reuse, Array/Map forms,
triple-double String whitespace/newlines, comment classes, block-comment
internal newlines, trailing-Closure attachment, separator kind, continuation
newline exclusion, and source-position-independent relationships.

## Cost and compatibility boundary

LM011-B1 preserves PLAT050's pay-only-when-used requirement:

```text
ORDINARY_LEXER_TRIVIA_RETENTION=NONE
ORDINARY_PARSER_SOURCE_FACT_RETENTION=NONE
ORDINARY_STATIC_PARSE_SOURCE_LAYOUT_CONSTRUCTION=NONE
SOURCE_LAYOUT_CONSTRUCTION=ON_DEMAND_ONLY
GLOBAL_FORMATTER_METADATA_CACHE=NONE
```

No second parser, second grammar, full CST, GreenNode/RedNode hierarchy,
`FormatterPolicy`, CLI formatter surface, LSP formatting handler, VS Code
formatter, style configuration, parser recovery, range formatting, on-type
formatting, or check mode is introduced by this slice.

## Validation

The maintainer reported the incremental focal tests PASS during implementation,
including lexer/source-span, legacy documentation extraction, parser source
facts, source-layout, and ordinary parser coverage.

The newly added source-layout test class remained a sub-second focal test in the
reported run; no new test with a cost of 10 seconds or more was introduced.

After rebasing the closing metadata onto product HEAD
`7ae6827219bfdeb55c4a365cce046ade1222f08d`, the maintainer reported the final
canonical repository validation:

```text
make test = PASS
```

The published commit is the single successor:

```text
7ae6827219bfdeb55c4a365cce046ade1222f08d
  -> d49361af6fcda2815b89e9ae9ff22bc766ecf45d
```

## Versioning and changelog

The published product version is:

```text
0.3.190-SNAPSHOT
```

The implementation changelog records LM011-B1 as the on-demand source-layout
and source-preservation foundation, with no formatter transformation, CLI/LSP
integration, CST, language semantic change, or specification change.

## Decision consistency

The implementation discovered no contradiction requiring D183 or PLAT050 to be
reopened.

```text
D183_DELTA=NONE
PLAT050_DELTA=NONE
PLAT024_DELTA=NONE
FULL_LOSSLESS_CST_REQUIRED=NO
HOST_FORMATTER_POLICY_INTRODUCED=NO
USER_MODULE_EXECUTION_REQUIRED=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

The implementation therefore validates the mechanical B1 portion of PLAT050
Candidate F without expanding the ratified boundaries.

## Coordination

```text
LM011_A=COMPLETE
LM011_B1=COMPLETE
D183=RATIFIED
PLAT050=RATIFIED

CURRENT_FORMATTER=NONE
LM011_COMPLETE=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=LM011-B2
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=EXACT_BUNDLED_PROTOS_D183_FORMATTER_POLICY_AND_WHOLE_DOCUMENT_AUTHORITY

LM011_C=FUTURE_CLI_SURFACE
LM011_D=FUTURE_STANDARD_LSP_AND_VSCODE_INTEGRATION
LM011_E=FUTURE_CORPUS_IDEMPOTENCE_END_TO_END_CLOSURE
```

LM011 remains open.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published LM011-B1 commit, the ratified D183 and PLAT050 records, the live
LM011/PLAT050 GitHub issue state, and maintainer-reported local validation
results.
