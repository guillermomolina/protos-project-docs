# LM011-B2 / TOOL010 — canonical formatter publication

## Scope

This record preserves the published LM011-B2 implementation of TOOL010, the
canonical whole-document Protos formatter released by D183 Candidate B and
PLAT050 Candidate F.

The owning live work item is `guillermomolina/protos#670` (**LM011 — Canonical
source formatter and editor formatting integration**). D183 / #791 and PLAT050
/ #792 remain the ratified policy and architecture authorities.

This record is non-normative implementation/publication evidence. It does not
change Protos language semantics or specification authority.

## Published authority

```text
SLICE=LM011-B2
TOOL_ID=TOOL010
TOOL_NAME=Canonical Source Formatter
FORMAL_WORK_ITEM=LM011/#670
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=5813f154e862e0479d5ebd6c9426eb3c3873156d
PARENT_REVISION=2da6622eb4d6a570ab3e579183ae3b840af1f7b7
COMMIT_SUBJECT=LM011-B2: add canonical source formatter
PROTOS_VERSION=0.3.197-SNAPSHOT

D183_STATUS=RATIFIED
D183_SELECTED_CANDIDATE=B_FIXED_STRUCTURAL_STYLE_PLUS_CONSERVATIVE_LEXICAL_PRESERVATION
PLAT050_STATUS=RATIFIED
PLAT050_SELECTED_CANDIDATE=F_ON_DEMAND_HYBRID_PLUS_BUNDLED_PROTOS_POLICY

OBSERVABLE_LANGUAGE_SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
CLI_CHANGE=NO
LSP_CHANGE=NO
VSCODE_CHANGE=NO

MAINTAINER_REPORTED_INCREMENTAL_GATES=PASS
MAINTAINER_REPORTED_MAKE_TEST=PASS
MAINTAINER_REPORTED_TEST_TOTAL=1328
MAINTAINER_REPORTED_TEST_FAILURES=0
MAINTAINER_REPORTED_TEST_TIME=45s
```

## Changed paths

The publication changes exactly these product-repository paths:

```text
CHANGELOG.md
pom.xml
protos/tools/formatter/Main.protos
protos/tools/formatter/Structural.protos
src/main/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutView.java
src/main/java/com/guillermomolina/protos/execution/ProtosFormatterToolBootstrap.java
src/main/java/com/guillermomolina/protos/execution/ProtosSourceLayoutToolBridge.java
src/main/java/com/guillermomolina/protos/execution/ProtosWholeDocumentFormatter.java
src/test/java/com/guillermomolina/protos/analysis/ProtosSourceLayoutViewTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosFormatterToolBootstrapTest.java
```

## TOOL010 bundled formatter authority

TOOL010 is an exact bundled Protos tool under:

```text
protos/tools/formatter/Main.protos
protos/tools/formatter/Structural.protos
```

The host bootstrap validates the exact TOOL010 identity and boots the bundled
formatter independently of the user's project/package graph. The user's module
is never executed as part of formatting.

D183 layout policy is owned primarily by bundled Protos code rather than by a
Java/native `FormatterPolicy` institution.

The implemented canonical policy includes:

```text
structural indentation = 4 spaces
continuation indentation = 4 spaces
generated structural tabs = none
soft width = 100
canonical operator / delimiter spacing
canonical brace placement
canonical Array / Map / Closure layout
trailing Closure preservation
semicolon versus logical-newline preservation
ordinary blank-line collapse
structural output EOL = LF
final formatter-owned LF = exactly one
```

## B1 source-layout bridge

LM011-B2 reuses the on-demand B1 source-layout mechanism rather than introducing
a second parser, grammar, CST, GreenNode/RedNode hierarchy or source
reconstruction authority.

The bridge exposes only inert mechanical source facts needed by the bundled
formatter, including:

```text
exact immutable source
Surface structural projection
raw token spelling
raw comment spelling
comment placement and attachment
sequence-separator kind and blank-line facts
explicit grouping
Array / Map surface forms
Closure source forms
trailing-Closure relationships
```

Raw spelling authority remains the immutable source snapshot.

## Comments and lexical payload

The formatter preserves comment raw text, order and attachment classes:

```text
OWN_LINE
END_OF_LINE
EMBEDDED_BETWEEN_TOKENS
```

Grammar-significant sequence and trailing-Closure boundaries remain preserved.

String, numeric and valid Unicode spellings are kept exact. Triple-double
String payload keeps its physical LF/CR/CRLF, spaces, tabs, blank whitespace,
delimiter layout and escapes unchanged.

## Whole-document API and fail-closed behavior

`ProtosWholeDocumentFormatter` is the editor-neutral whole-document boundary.

Its minimum result contract is:

```text
SUCCESS(formatted exact source)
FAILURE(original exact source unchanged + inert reason)
```

Invalid or incomplete source fails before TOOL010 bootstrap:

```text
FORMATTED_EDITS=NONE
ORIGINAL_SOURCE=UNCHANGED
FORMAT_RESULT=FAILURE
TOOL010_PROCESS_STARTED=NO
```

A genuine bundled formatter/bootstrap failure remains a host/tool failure and is
not mislabeled as invalid source.

No LSP `TextEdit` shape or CLI exit-code/spelling policy is introduced by B2.

## Correctness gates

LM011-B2 closed the formatter correctness requirements independently.

### Reparse, determinism and exact idempotence

```text
REPARSE=PASS
DETERMINISM=PASS
IDEMPOTENCE=PASS
```

Formatting the same valid source is deterministic; formatted output reparses;
and exact whole-document formatting is idempotent.

### D183 source-preservation invariants

```text
SOURCE_PRESERVATION_GATE=PASS
```

The gate compares D183-preserved facts rather than policy-owned structural
newlines or blank-line normalization. It preserves token spellings excluding
formatter-owned structural NEWLINE tokens, comment raw text/placement and
structural attachment, structural source forms, separator context/kind and
neighbors, Closure forms, and trailing-Closure relationships.

### Non-executing semantic equivalence

```text
SEMANTIC_EQUIVALENCE_GATE=PASS
USER_MODULE_EXECUTION_REQUIRED=NO
```

Original and formatted sources are compared through the canonical semantic AST
without executing source. Source spans are deliberately excluded from semantic
identity; all non-span canonical record fields, list order and semantic scalar
values remain part of the projection.

### Invalid/incomplete fail-closed

```text
FAIL_CLOSED_GATE=PASS
```

Both incomplete and syntactically invalid inputs preserve their exact original
source and do not start TOOL010.

## Validation

During implementation the maintainer reported all focal formatter and
source-layout gates PASS.

Before the version/changelog closeout, the maintainer reported the full
repository validation:

```text
make test
1328 passed, 0 failed
Protos tests total time: 45 s
```

After `pom.xml` and `CHANGELOG.md` were advanced to
`0.3.197-SNAPSHOT`, no further test was used as publication evidence, in
accordance with the repository closeout rule.

A later host-side attempt to invoke `bin/protos test` did not start Protos
because the host shell could not find `java`; it is not counted as a test run
or as publication validation.

The published commit is the single successor:

```text
2da6622eb4d6a570ab3e579183ae3b840af1f7b7
  -> 5813f154e862e0479d5ebd6c9426eb3c3873156d
```

## Decision consistency

The implementation exposed no contradiction requiring D183 or PLAT050 to be
reopened.

```text
D183_DELTA=NONE
PLAT050_DELTA=NONE
PLAT024_DELTA=NONE
FULL_LOSSLESS_CST_REQUIRED=NO
SECOND_PARSER_OR_GRAMMAR_INTRODUCED=NO
HOST_FORMATTER_POLICY_INTRODUCED=NO
USER_MODULE_EXECUTION_REQUIRED=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

## Deferred scope

LM011-B2 does not implement:

```text
public formatter CLI surface
LSP textDocument/formatting
VS Code Format Document / format-on-save
range formatting
check mode
on-type formatting
formatter-disable directives
project/editor style configuration
parser recovery
partial valid-region formatting
comment reflow
literal canonicalization
redundant-parenthesis removal
semantic refactoring
minimal-edit guarantee
```

## Coordination

```text
LM011_A=COMPLETE
LM011_B1=COMPLETE
LM011_B2=COMPLETE
LM011_B=COMPLETE
TOOL010=IMPLEMENTED
D183=RATIFIED
PLAT050=RATIFIED

CURRENT_FORMATTER=CANONICAL_WHOLE_DOCUMENT_TOOL010
LM011_COMPLETE=NO
NEW_FORMAL_ISSUE_REQUIRED=NO

NEXT_SLICE=LM011-C
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=TOOLCHAIN_CLI_FORMATTING_SURFACE

LM011_D=FUTURE_STANDARD_LSP_AND_VSCODE_INTEGRATION
LM011_E=FUTURE_CORPUS_IDEMPOTENCE_END_TO_END_CLOSURE
```

LM011 remains open.

AI assistance: this durable publication record was drafted with ChatGPT from
the exact published LM011-B2 commit, the ratified D183 and PLAT050 records, the
TOOL010 allocation evidence, the live LM011 GitHub issue state, and
maintainer-reported validation results.
