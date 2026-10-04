# LM011-A — Formatter authority and policy audit

Status: **COMPLETED INVESTIGATION — D183 REQUIRED BEFORE IMPLEMENTATION**

Evidence date: **2026-10-04**

Owning work item:

- LM011 / `guillermomolina/protos#670`
- D183 / `guillermomolina/protos#791`

Audited revisions:

```text
PROTOS_REVISION=ee066755219dfbe6ac7dd69a68b3ac76492bf7ee
PROTOS_VSCODE_EXTENSION_REVISION=6332f34f4a581d91771bae1a6b173b6e796ddffe
PROJECT_DOCS_BASE_REVISION=dc751e4c10dd1bc52b7961558044729cc0b3910a
```

Nature: non-normative investigation evidence. This record does not select formatting policy, ratify D183, authorize formatter implementation, or change Protos language semantics.

## Purpose

LM011-A was required to audit the current Protos source, parser, toolchain and editor architecture before any formatter implementation.

The audit had to answer:

1. whether Protos already had a formatter authority;
2. which existing source representations preserve enough information for faithful formatting;
3. which formatter behaviors are mechanically determined by the language versus public formatting policy;
4. whether PLAT024 and the existing VS Code language-client boundary can be reused;
5. whether the common bundled-tool architecture already constrains formatter ownership; and
6. what decision gate is required before LM011-B.

## Material authorities inspected

The audit materially inspected current repository state including:

```text
guillermomolina/protos
  AGENTS.md
  AGENTS.work/IMPLEMENTATION.md
  AGENTS.work/COORDINATION.md
  AGENTS.work/DESIGN.md
  AGENTS.work/REFERENCE.md
  src/AGENTS.md
  spec/AGENTS.md
  docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md
  spec/PROTOS_GRAMMAR.md
  spec/PROTOS_LANGUAGE_SPEC.md
  src/main/java/com/guillermomolina/protos/lexer/Token.java
  src/main/java/com/guillermomolina/protos/lexer/TokenType.java
  src/main/java/com/guillermomolina/protos/lexer/TokenOccurrence.java
  src/main/java/com/guillermomolina/protos/lexer/ProtosLexer.java
  src/main/java/com/guillermomolina/protos/source/SourceSpan.java
  src/main/java/com/guillermomolina/protos/parser/ProtosParser.java
  src/main/java/com/guillermomolina/protos/parser/TokenCursor.java
  src/main/java/com/guillermomolina/protos/parser/ast/Surface*.java
  src/main/java/com/guillermomolina/protos/semantic/Canonicalizer.java
  src/main/java/com/guillermomolina/protos/analysis/ProtosDocumentSnapshot.java
  src/main/java/com/guillermomolina/protos/analysis/ProtosStaticParseResult.java
  src/main/java/com/guillermomolina/protos/analysis/ProtosStaticAnalysisSession.java
  src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java
  pom.xml
  LM009-F / #340
  LM009-G / #360
  PLAT024 / #342

guillermomolina/protos-project-docs
  AGENTS.md
  docs/project/README.md
  docs/project/decisions/platform/PLAT024_STATIC_LANGUAGE_SERVICE_HOSTING_PROTOCOL_BOUNDARY.md
  docs/project/registries/IMPLEMENTATION_BLOCKERS.md

guillermomolina/protos-vscode-extension
  AGENTS.md
  package.json
  extension.js
  README.md
```

The audit also verified the current standalone editor-product authority after the LM009 repository split rather than assuming the historical `editors/vscode/` location under `guillermomolina/protos`.

## Finding 1 — no canonical formatter exists

Current HEAD contains no canonical Protos source formatter.

In particular:

- no formatter library or bundled formatter tool exists;
- the language server does not advertise document formatting;
- the text-document service has no document-formatting handler;
- no formatter CLI surface exists; and
- the VS Code extension has no independent formatter implementation.

Therefore:

```text
CURRENT_FORMATTER=NONE
```

## Finding 2 — the Surface AST is structurally useful but not lossless

The current source pipeline preserves information across three different authorities:

```text
exact document source
    +
token occurrences / source spans
    +
Surface AST
```

### Exact document source

`ProtosDocumentSnapshot` retains the complete current source characters.

This is the only existing representation that necessarily retains every source spelling distinction.

### Tokens and spans

`TokenOccurrence` couples emitted tokens to exact `SourceSpan` ranges.

This allows tooling to recover raw token spelling from the immutable source snapshot even when `Token.lexeme()` itself is normalized or semanticized.

### Surface AST

The `Surface*` model preserves important syntax distinctions such as:

- explicit parenthesized grouping;
- slot creation versus assignment;
- Array and Map construction forms;
- object/Closure structure;
- source spans;
- expression-bodied versus braced Closure source.

It does not retain all source presentation information.

## Finding 3 — information not independently represented by the Surface AST

The current lexer/parser intentionally discards or normalizes source details irrelevant to execution.

Examples include:

- horizontal SPACE/TAB runs;
- comment attachment;
- block comments in the parser token stream;
- blank-line multiplicity;
- physical `LF` versus `CR` versus `CRLF` spelling;
- `;` versus separating logical newline after parsing into sequence structure;
- comma and delimiter positions as first-class AST objects;
- exact String quote form and escape spelling;
- some trailing-Closure source distinctions;
- general trivia ownership.

The raw immutable source still contains these characters, and existing spans provide structural anchors.

The audit therefore does not justify introducing a full lossless CST yet.

The narrowest demonstrated source model is:

```text
SOURCE_MODEL=HYBRID_SOURCE_AST_TOKEN_MODEL_REQUIRED
FULL_LOSSLESS_CST_REQUIRED=NOT_PROVEN
SURFACE_AST_ALONE_SUFFICIENT=NO
TOKEN_STREAM_ALONE_SUFFICIENT=NO
```

## Finding 4 — canonical semantic AST must not be formatter authority

`Canonicalizer` deliberately erases or lowers syntax distinctions.

Examples include:

- `SurfaceGroup` disappearing;
- Array construction lowering toward an ordinary call;
- member calls/indexing lowering into canonical sends/calls;
- expression-bodied and braced Closures converging;
- binary syntax lowering into semantic send forms.

Canonical AST is therefore suitable as possible semantic-equivalence evidence, but not as a source reconstruction authority.

```text
CANONICAL_AST_AS_FORMATTER_SOURCE=REJECTED
```

## Finding 5 — language mechanics versus formatting policy

The grammar mechanically constrains a formatter to preserve:

- valid token separation;
- required delimiters;
- precedence and associativity;
- grouping where removing it changes parse or meaning;
- newline/semicolon legality;
- continuation behavior;
- comment lexical semantics;
- String values;
- numeric values;
- Unicode identifier validity; and
- triple-double-quoted String newline and indentation semantics.

The grammar does not choose a canonical style for:

- indentation width;
- tabs versus spaces;
- continuation indentation;
- brace placement;
- whitespace around operators, `:`, `=`, commas, calls, indexing or member access;
- line width or wrapping;
- blank-line normalization;
- redundant-parentheses policy;
- comment reflow/preservation policy;
- String or numeric source-spelling normalization;
- final newline;
- physical line-ending normalization outside semantic literal content;
- invalid-source behavior;
- idempotence;
- formatter configurability; or
- whether LSP `tabSize` / `insertSpaces` influence canonical output.

Those are observable formatting-tool behavior and require implementation-independent project-owner selection before implementation.

```text
PUBLIC_FORMAT_POLICY_DECISION=REQUIRED
```

## Finding 6 — PLAT024 is reusable

PLAT024 Candidate A-prime remains directly applicable.

The existing architecture already provides:

```text
VS Code / another LSP editor
        |
        | standard LSP over stdio
        v
thin protocol adapter / server host
        |
        v
editor-neutral Protos static-analysis core
        |
        +-- real Protos source/parser authorities
```

The standalone VS Code extension currently remains a thin `vscode-languageclient` consumer that starts the external `protos language-server` through `protos.runtime.executable`.

A future document-formatting request can therefore use the same client/server boundary.

No independent TypeScript formatter is required.

```text
PLAT024_REUSED=YES
VSCODE_THIN_CLIENT_PRESERVED=YES
VSCODE_SIDE_FORMATTING_SEMANTICS=NO
```

PLAT024 deliberately did not select formatter/refactoring ownership, so reusing PLAT024 does not settle the formatter implementation architecture.

## Finding 7 — bundled-tool architecture constrains later formatter ownership

`docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md` already establishes that future formatter transformation policy should be evaluated under the common official tool architecture rather than growing into a host/runtime institution.

The selected direction favors:

- one public `protos` toolchain driver;
- small host/bootstrap machinery;
- higher-level official tool policy primarily as exact toolchain-bundled Protos programs where naturally expressible;
- `TOOLxxx` ownership for concrete promoted official bundled tools; and
- no Java/native `FormatterPolicy` institution merely because the host can implement it.

However, current parser/source authority is JVM-hosted while bundled tool policy normally lives in Protos code.

The durable bridge among:

```text
lossless source/parser mechanism
bundled formatter policy
language-server formatting request
CLI / CI reuse
```

is not currently selected.

Therefore:

```text
BUNDLED_TOOL_ARCHITECTURE_REUSED=PARTIAL
FORMATTER_ARCHITECTURE_DECISION=REQUIRED_AFTER_PUBLIC_POLICY
```

The architecture decision must follow the public-policy decision so the chosen representation and execution topology satisfy, rather than silently define, the formatter contract.

## Correctness model identified by LM011-A

The audit distinguishes three independent levels of formatter correctness:

```text
1. parse(format(source)) succeeds
2. semanticNormalize(parse(format(source)))
   == semanticNormalize(parse(source))
3. selected source-preservation invariants hold
```

A future policy may additionally require byte-for-byte idempotence:

```text
format(format(source)) == format(source)
```

The first two properties cannot by themselves decide comment, literal-spelling or other source-preservation policy.

## Decision routing

LM011-A exposes two substantive gates:

```text
GATE 1
  public canonical formatting/source-preservation contract
  -> Dxxx

GATE 2
  formatter authority + lossless-source/tool/LSP architecture
  -> PLATxxx
```

Gate 1 must precede Gate 2.

The collision-safe allocation audit found D182 as the greatest allocated top-level D identifier and no existing D183 in current GitHub work or durable-project commit evidence.

D183 was therefore allocated as:

```text
D183 — Canonical source formatting policy and preservation contract
GITHUB_ISSUE=guillermomolina/protos#791
STATE=NEEDS_DECISION
PRIORITY=P1
IMPLEMENTATION_AUTHORIZED=NO
```

Native Parent/Sub-issue linkage from D183/#791 to LM011/#670 remains pending because the available GitHub connector exposes no native sub-issue mutation operation. The textual parent declaration in D183 is bootstrap evidence only and is not claimed as equivalent.

## LM011 state after audit

LM011-A itself is complete.

LM011-B is not yet released because the public formatter policy is unresolved.

The resulting work state is:

```text
LM011_A_AUDIT=PASS
CURRENT_FORMATTER=NONE
SOURCE_MODEL=HYBRID_SOURCE_AST_TOKEN_MODEL_REQUIRED
PUBLIC_FORMAT_POLICY_DECISION=D183/#791
FORMATTER_ARCHITECTURE_DECISION=REQUIRED_AFTER_D183
PLAT024_REUSED=YES
BUNDLED_TOOL_ARCHITECTURE_REUSED=PARTIAL
VSCODE_THIN_CLIENT_PRESERVED=YES
IMPLEMENTATION_AUTHORIZED=NO
NEXT=D183_FORMATTING_POLICY_INVESTIGATION
```

## Next step

The next executable unit is not implementation.

It is the D183 investigation packet required by `AGENTS.work/DESIGN.md` / GITHUB010.

That investigation must compare materially different formatter policy families, define the candidate set and scorecard, recommend one contract, and stop for explicit project-owner approval.

Only after D183 is explicitly approved and durably ratified may the project allocate and investigate the subsequent formatter architecture decision.
