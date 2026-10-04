# PLAT050 — Canonical formatter source/trivia authority and tooling bridge

Status: **RATIFIED**

Selected architecture: **Candidate F — on-demand hybrid source-layout view + exact bundled Protos formatter policy over a tool-neutral host source mechanism**.

Approval: explicit project-owner approval on 2026-10-04:

~~~text
Apruebo candidate F.
~~~

Decision Issue: `guillermomolina/protos#792`

Parent workstream: `LM011 / guillermomolina/protos#670`

Governing public policy: `D183 / guillermomolina/protos#791`, Candidate B — fixed structural style + conservative lexical preservation.

Product authority at final invariant/delta review:

~~~text
PROTOS_REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
PROTOS_VSCODE_EXTENSION_REVISION=6332f34f4a581d91771bae1a6b173b6e796ddffe
D183_PROJECT_RECORD_REVISION=e6f56757e9191286b5a4fd6f1980cd9e43d995eb
~~~

Nature: durable non-normative tooling/platform architecture. Observable Protos language semantics remain unchanged.

## Decision

The canonical formatter uses the existing source authorities as its base:

~~~text
exact immutable source
+
TokenOccurrence / SourceSpan
+
Surface AST
~~~

and constructs additional formatter-only metadata **on demand**.

No dedicated lossless syntax layer or full lossless CST is required for the current D183 baseline.

The selected architecture is:

~~~text
exact immutable source snapshot
        |
        v
canonical lexer
        |
        +---- TokenOccurrence[] -------------------+
        |                                         |
        +---- opt-in trivia occurrences --------+ |
                                                | |
canonical parser                                | |
        |                                       | |
        +---- Surface AST                       | |
        |                                       | |
        +---- opt-in parser source facts -------+ |
                                                  |
                                                  v
                              immutable on-demand source-layout view
                                                  |
                                  +---------------+----------------+
                                  |                                |
                                  v                                v
                         bundled Protos                    correctness
                       formatter policy                     projections
                                  |
                                  v
                           formatted source
                                  |
                      postcondition validation
                                  |
                    success OR fail closed/no edits
~~~

The hybrid representation and bundled-tool ownership are one architecture because they resolve two orthogonal requirements:

- the smallest source/trivia representation that preserves D183 source invariants; and
- the already-ratified Toolchain Tool Architecture boundary in which higher-level formatter policy is not a permanent Java/native `FormatterPolicy` institution.

## Exact raw-source authority

The exact input source remains the only raw spelling authority.

~~~text
EXACT_RAW_SOURCE_AUTHORITY=IMMUTABLE_INPUT_SOURCE
~~~

`Token.lexeme()` and semantic AST values are not raw-source authorities.

Raw String and numeric spelling is recovered from the exact source using the existing source span.

This is especially required for triple-double Strings, whose lexer token value is semantically normalized while the exact source span still identifies the original quotes, escapes, indentation, SPACE/TAB, blank-line whitespace and physical line endings.

## Surface AST role

The existing Surface AST remains the canonical valid-source structural parse authority.

It carries, among other facts:

- precedence and expression structure;
- explicit grouping through `SurfaceGroup`;
- slot creation versus assignment;
- Array versus Map construction;
- object and Closure structure;
- Closure expression-body versus braced-body form;
- source spans.

It is deliberately not a lossless source tree.

No formatter may reconstruct source from the canonical semantic AST because mandatory lowering erases source distinctions such as explicit grouping and surface sugar.

## TokenOccurrence role

Existing token occurrences remain the canonical lexical-token/source-extent authority.

They provide:

- token kinds;
- delimiter and punctuation topology;
- logical `NEWLINE` occurrences;
- exact raw-source spans for identifiers and literals.

Raw spelling is always read from the source snapshot by span when D183 requires exact preservation.

## On-demand trivia authority

The lexer tooling path may expose formatter-only occurrences for source material currently discarded from the normal token stream.

The minimum retained categories are:

~~~text
HORIZONTAL_WHITESPACE
LINE_COMMENT
BLOCK_COMMENT
~~~

Each occurrence needs only its kind and exact `SourceSpan`; the input snapshot owns the characters.

Parser-visible logical newlines continue to be represented by ordinary `TokenOccurrence(NEWLINE)`, not duplicated as trivia.

A block-comment occurrence contains its own physical newlines inside its span. Those physical newlines do not become parser `NEWLINE` tokens, preserving the current lexical contract.

The ordinary execution path must not allocate or retain these formatter-only occurrences.

## Parser source facts

Some source distinctions are known exactly by the parser but disappear from the Surface AST.

The tooling parse path therefore records only the smallest source facts that the formatter cannot recover faithfully from the Surface AST alone:

~~~text
sequence separator source kind:
    SEMICOLON
    LOGICAL_NEWLINE

Closure parameter source form:
    BARE_SINGLE_PARAMETER
    PARENTHESIZED_PARAMETER_LIST

trailing-Closure source origin
~~~

These are compact side facts, not nodes in a new syntax tree.

The formatter must not independently reimplement parser continuation/separator rules to guess whether a physical/logical newline was a sequence separator.

## Comment attachment

Comment attachment is derived deterministically when the on-demand source-layout view is built and then stored in that immutable view.

The minimum D183 attachment classes are:

~~~text
OWN_LINE
END_OF_LINE
EMBEDDED_BETWEEN_TOKENS
~~~

Attachment also carries the nearest relevant structural relationship and any grammar-significant separator boundary.

Structural identity for before/after preservation checks must be independent of source offsets, for example a stable structural path through the parsed source shape.

This is required because formatting changes offsets.

## Critical block-comment / trailing-Closure invariant

For source such as:

~~~protos
foo() /*
    comment
*/ {
    body()
}
~~~

the block comment consumes its internal physical newlines. There is no parser `NEWLINE` token between the call close and the Closure open.

The source-layout view therefore preserves both:

~~~text
COMMENT_ATTACHMENT=EMBEDDED_BETWEEN_TOKENS
TRAILING_CLOSURE_ORIGIN=YES
~~~

The formatter policy must not manufacture a logical separating newline that detaches the trailing Closure.

## Lossless-tree decision

~~~text
DEDICATED_LOSSLESS_SYNTAX_LAYER_REQUIRED=NO
FULL_LOSSLESS_CST_REQUIRED=NO
~~~

Roslyn, SwiftSyntax and rust-analyzer/rowan demonstrate the value of full-fidelity/lossless trees for broad source tooling, recovery, incremental editing and refactoring.

Prettier demonstrates that original source + AST/source locations + explicit comment attachment can support a formatter without requiring a CST.

clang-format demonstrates that one shared formatter authority can operate over lexer/token/source machinery and be reused by CLI and editors.

For current Protos requirements, parser recovery, partial invalid-region formatting, range formatting, general refactoring and incremental syntax identity are explicitly deferred. Building their substrate now would violate present-need proportionality.

If future source-tooling work causes the compact side-fact set to grow into a grammar-parallel representation, that is the concrete trigger to reopen the representation question and reconsider a dedicated lossless syntax layer.

## Host mechanism boundary

The host/runtime side owns only tool-neutral source mechanics:

~~~text
exact immutable source
canonical lexing
canonical parsing
token/source spans
on-demand trivia occurrences
on-demand parser source facts
immutable source-layout-view construction
source-position-free semantic-normalization mechanics
~~~

It does **not** own:

~~~text
indentation policy
spacing policy
wrapping policy
blank-line policy
comment layout policy
line-width policy
final-newline policy
FormatterPolicy
~~~

Those are D183 formatter policy.

## Formatter policy owner

~~~text
FORMATTER_POLICY_OWNER=EXACT_TOOLCHAIN_BUNDLED_PROTOS_FORMATTER
~~~

The existing Toolchain Tool Architecture already provides exact bundled-tool acquisition independent of the user's project package graph.

Therefore the formatter tool does not need project package resolution to bootstrap and does not execute the user's module.

~~~text
PROJECT_PACKAGE_RESOLUTION_REQUIRED_FOR_FORMATTER_BOOTSTRAP=NO
USER_MODULE_EXECUTION_REQUIRED=NO
~~~

The user's source is input data to source tooling.

## Editor-neutral whole-document boundary

The canonical formatter authority is whole-document and editor-neutral.

Conceptually:

~~~text
formatWholeDocument(exactSourceSnapshot)
    ->
    Success(formattedText)
    |
    InvalidSource(noEdits, diagnostic)
    |
    InvariantFailure(noEdits, diagnostic)
~~~

No editor style configuration participates in the D183 result.

Standard LSP formatting options such as `tabSize` and `insertSpaces` may be accepted by the protocol adapter but are ignored as canonical style authority.

## PLAT024 bridge

PLAT024 remains unchanged.

~~~text
VS Code / another standard LSP client
        |
        | textDocument/formatting
        v
thin LSP adapter
        |
        v
same editor-neutral formatWholeDocument authority
~~~

No independent TypeScript formatter is authorized.

The standalone VS Code extension remains a thin `vscode-languageclient` consumer of the external `protos` language server.

## Correctness architecture

Formatter output is accepted only after all applicable postconditions succeed.

### Parse validity

~~~text
parse(format(source)) succeeds
~~~

Failure produces no edits.

### Semantic equivalence

Both original and formatted source are:

1. parsed through the canonical parser;
2. lowered through mandatory semantic canonicalization/desugaring;
3. projected into a source-position-free semantic form;
4. normalized for literal semantic values;
5. structurally compared.

Direct equality of current Java canonical records is not sufficient because they retain `SourceSpan`, and numeric canonical literals currently preserve spelling before runtime materialization.

No user module is executed.

### Source-preservation invariants

A source-position-independent preservation projection compares at least:

~~~text
raw String spelling
raw numeric spelling
identifier source code points
comment raw text/order/attachment
explicit grouping
semicolon-vs-newline separator kind
Closure expression/braced form
single-parameter Closure source form
trailing-Closure relationship
Array construction form
Map construction form
~~~

### Idempotence

~~~text
format(format(source)) == format(source)
~~~

must hold exactly.

### Fail closed

Any lex, parse, required static syntax, output-parse, semantic-equivalence or source-preservation failure returns no formatted edits and leaves the original source unchanged.

## Pay only for what is used

Ordinary Protos execution must retain:

~~~text
FORMATTER_TRIVIA_ALLOCATION=0
FORMATTER_ATTACHMENT_CONSTRUCTION=0
FORMATTER_SOURCE_FACT_RETENTION=0
~~~

when formatting is not requested.

The current lexer already provides precedent for opt-in tooling metadata through its nullable line-comment observer.

A formatting invocation pays O(n) source-view construction and validation cost.

A language server may cache formatter metadata only for the current immutable document version and may discard it on replacement or close.

The distributed JVM/Native toolchain may incur some additional bundled-code/image size; that cost is implementation evidence to measure, not a reason to impose runtime metadata on ordinary programs.

## JVM, Native Image and alternative runtimes

The architecture is compatible with JVM and Native Image because it requires no dynamic plugin system and no live parser/tree graph retention after formatting.

The exact implementation and Native Image reachable-size delta must be measured during implementation.

The bundled formatter policy remains Protos code. A future runtime/backend may replace only the source-mechanics producer while preserving the same editor-neutral formatter contract.

## Candidate disposition

### Candidate A — current hybrid + source gaps/trivia

Viable and selected as the representation half of Candidate F.

Alone it does not decide bundled-tool/host policy ownership.

### Candidate B — dedicated lossless syntax layer

Viable but not selected.

It offers a cleaner foundation for future recovery, refactoring, incremental syntax identity and range transformations, but those are not current requirements.

### Candidate C — full lossless CST

Rejected for the present baseline.

It pre-pays for broad source-tooling capability not required by D183.

### Candidate D — host-owned formatter engine

Rejected as formatter-policy ownership.

The irreducible host source mechanism is retained, but D183 policy must not become a Java/native `FormatterPolicy` institution.

### Candidate E — bundled Protos formatter over host source mechanism

Viable and selected as the ownership/bridge half of Candidate F.

Alone it does not define the source/trivia representation.

### Candidate F — real hybrid

Selected.

~~~text
A'S_MINIMAL_ON_DEMAND_SOURCE_REPRESENTATION
+
E'S_EXACT_BUNDLED_PROTOS_POLICY_OWNERSHIP
~~~

The combination removes a demonstrated trade-off rather than accumulating speculative mechanisms.

### Candidate G — defer

Rejected.

A smallest-sufficient architecture now exists; continued deferral would keep LM011-B blocked without removing a demonstrated risk.

## Strongest argument against Candidate F

Candidate F maintains coordinated but separate views:

~~~text
Surface AST
TokenOccurrence list
trivia occurrences
small parser source-fact table
comment-attachment relationships
~~~

A dedicated lossless syntax layer would unify those into one topology.

If future syntax growth causes many source-form side facts to accumulate, the hybrid could become an implicit CST with worse maintenance properties.

This risk is accepted because the currently required facts are small and enumerated. Crossing that boundary is the explicit future trigger to reconsider Candidate B rather than endlessly extending the side tables.

## GITHUB021 invariant/delta result

The final approved Candidate F preserves every applicable previously approved invariant.

~~~text
D183_DELTA=NONE
PLAT024_DELTA=NONE
TOOLCHAIN_TOOL_ARCHITECTURE_DELTA=NONE
LM012_FORMATTING_LINT_SEPARATION_DELTA=NONE

D183_FIXED_STYLE=UNCHANGED
D183_CONSERVATIVE_LEXICAL_PRESERVATION=UNCHANGED
D183_INVALID_SOURCE_FAIL_CLOSED=UNCHANGED
D183_DETERMINISM_IDEMPOTENCE=UNCHANGED
D183_RANGE_FORMATTING_DEFERRED=UNCHANGED
D183_CHECK_MODE_DEFERRED=UNCHANGED

VSCODE_THIN_CLIENT=UNCHANGED
INDEPENDENT_TYPESCRIPT_FORMATTER=NO
JAVA_NATIVE_FORMATTER_POLICY_INSTITUTION=NO
USER_MODULE_EXECUTION_DURING_FORMATTING=NO

DECISION_INVARIANT_CONSISTENCY=PASS
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The move from the investigation revision `37bab1b034e87b79ff4475a3e4013cfb5ef3fd73` to final product authority `a38470bc6e2f68e770ddc8054053995bb2477b19` changes Package Tool external-execution composition, implementation version and associated tests/metadata. It does not change the lexer/source/parser/Surface-AST/static-analysis/LSP/grammar/toolchain surfaces on which PLAT050 depends.

## Released implementation

Ratification releases the first mechanical LM011-B implementation slice:

~~~text
SLICE=LM011-B1
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
GOAL=on-demand source-layout view and D183 preservation authorities
~~~

LM011-B1 may implement:

1. tooling-only lexer trivia occurrences for horizontal whitespace, line comments and block comments while preserving the ordinary zero-metadata path;
2. opt-in parser source facts for sequence separator kind, single-parameter Closure source form and trailing-Closure origin;
3. an immutable editor-neutral source-layout view over exact source, tokens, Surface AST, trivia and parser facts;
4. deterministic comment attachment with D183 attachment classes and grammar-significant relationships;
5. source-position-independent preservation projections and focal tests for the source-model invariants.

LM011-B1 must not yet:

- select a CLI spelling;
- implement range/check/on-type formatting;
- add parser recovery or partial invalid-source formatting;
- add style/project/editor configuration;
- add a TypeScript formatter;
- execute user modules;
- introduce a lossless CST;
- turn Java/native into formatter-policy authority.

After B1, later LM011-B slices may implement the bundled Protos D183 formatting policy, whole-document authority and the ratified correctness checks without another architecture decision unless implementation reveals a concrete contradiction with this decision.

## Ratified result

~~~text
PLAT050_STATUS=RATIFIED

CURRENT_PRODUCT_REVISION=a38470bc6e2f68e770ddc8054053995bb2477b19
CURRENT_SOURCE_MODEL=EXACT_SOURCE + TOKEN_OCCURRENCES/SOURCE_SPANS + SURFACE_AST
SELECTED_CANDIDATE=F_ON_DEMAND_HYBRID_SOURCE_LAYOUT_VIEW_PLUS_EXACT_BUNDLED_PROTOS_FORMATTER_POLICY

EXACT_RAW_SOURCE_AUTHORITY=IMMUTABLE_INPUT_SOURCE
TRIVIA_COMMENT_AUTHORITY=ON_DEMAND_SPAN_BASED_TRIVIA_PLUS_DERIVED_ATTACHMENT
PARSER_SOURCE_FACTS=ON_DEMAND_SMALL_SIDE_FACT_SET
LOSSLESS_SYNTAX_LAYER_REQUIRED=NO
FULL_LOSSLESS_CST_REQUIRED=NO

FORMATTER_POLICY_OWNER=EXACT_TOOLCHAIN_BUNDLED_PROTOS_FORMATTER
HOST_MECHANISM_BOUNDARY=TOOL_NEUTRAL_SOURCE_LEX_PARSE_PROJECTION_MECHANICS
PROJECT_PACKAGE_RESOLUTION_REQUIRED_FOR_FORMATTER_BOOTSTRAP=NO
PLAT024_LSP_BRIDGE=UNCHANGED_THIN_STANDARD_LSP
CLI_CI_REUSE_BOUNDARY=SAME_EDITOR_NEUTRAL_WHOLE_DOCUMENT_AUTHORITY

SEMANTIC_EQUIVALENCE=NON_EXECUTING_SOURCE_POSITION_FREE_SEMANTIC_NORMALIZATION
SOURCE_PRESERVATION=SOURCE_POSITION_INDEPENDENT_D183_PRESERVATION_PROJECTION
USER_MODULE_EXECUTION_REQUIRED=NO

ORDINARY_PROGRAM_FORMATTER_METADATA_COST=ZERO
GITHUB021_INVARIANT_DELTA_CHECK=PASS
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

LM011_B_RELEASED=YES
IMPLEMENTATION_AUTHORIZED=YES
IMPLEMENTATION_SLICE=LM011-B1
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

## References

- `guillermomolina/protos#792` — PLAT050 decision Issue.
- `guillermomolina/protos#670` — LM011 formatter workstream.
- `guillermomolina/protos#791` — D183 formatter policy decision.
- `docs/project/decisions/tooling/D183_CANONICAL_SOURCE_FORMATTING_POLICY_AND_PRESERVATION_CONTRACT.md`.
- `docs/project/evidence/PLAT050/PLAT050_CANDIDATE_F_RATIFICATION_EVIDENCE.md`.
- `docs/project/evidence/LM011/LM011_PLAT050_RELEASE.md`.
- `docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md` in `guillermomolina/protos`.
