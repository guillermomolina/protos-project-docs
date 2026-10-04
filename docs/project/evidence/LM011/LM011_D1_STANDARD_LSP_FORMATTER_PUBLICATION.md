# LM011-D1 — canonical formatter standard-LSP publication

Evidence date: **2026-10-04**

Owning workstream: `LM011 / guillermomolina/protos#670`

Governing formatter policy: `D183 / guillermomolina/protos#791`

Governing formatter architecture: `PLAT050 / guillermomolina/protos#792`

Governing language-service hosting architecture: `PLAT024 / guillermomolina/protos#342`

Nature: immutable implementation/publication evidence for LM011-D1, which exposes
the already-published canonical whole-document formatter through standard LSP
`textDocument/formatting`.

## Exact publication

~~~text
SLICE=LM011-D1
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=75cfed853c7eef8c9d3dfb0c6f2507fdf8864ecb
COMMIT_SUBJECT=LM011-D1: expose canonical formatter over LSP
PROTOS_VERSION=0.3.200-SNAPSHOT

MAINTAINER_REPORTED_LOCAL_TESTS=PASS

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

LM011-D1 was finalized on top of concurrent TEST009-C completion:

~~~text
IMMEDIATE_PRECEDING_REVISION=bb3a15b2c529368879eeed527356bac0bb9e7d6e
IMMEDIATE_PRECEDING_SUBJECT=TEST009-C: complete Bytecode local API PE coverage
~~~

The final release metadata correctly uses the then-current
`0.3.200-SNAPSHOT` implementation version.

## Published files

The exact commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/execution/ProtosToolchainRoots.java
src/main/java/com/guillermomolina/protos/lsp/ProtosLanguageServer.java
src/main/java/com/guillermomolina/protos/lsp/ProtosLspDocumentFormatter.java
src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java
src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerFormattingTest.java
~~~

No parser, grammar, specification, bundled formatter-policy source or VS Code
extension source changes are part of D1.

## Standard-LSP capability

The language server now advertises:

~~~text
documentFormattingProvider=true
~~~

Existing definition, references and workspace-symbol capabilities remain
advertised.

The server does not advertise:

~~~text
documentRangeFormattingProvider
documentOnTypeFormattingProvider
~~~

Those capabilities remain deferred.

## Formatter authority

The LSP edge delegates to the same editor-neutral formatter used by the public
CLI:

~~~text
LSP textDocument/formatting
        |
        v
ProtosLspDocumentFormatter
        |
        v
ProtosWholeDocumentFormatter
        |
        v
TOOL010 exact bundled Protos formatter policy
~~~

`ProtosLspDocumentFormatter` is a package-private request adapter/test seam.
It owns no formatting policy.

There is no TypeScript formatter, second parser, second grammar or host-owned
`FormatterPolicy`.

## Toolchain-root reuse

LM011-D1 factors the pre-existing `PROTOS_HOME`/working-directory root lookup
into:

~~~text
src/main/java/com/guillermomolina/protos/execution/
ProtosToolchainRoots.java
~~~

The CLI and LSP now reuse that mechanical root lookup.

The helper owns no formatter policy and does not create a second toolchain
layout.

## Open-document source authority

Formatting uses the immutable source snapshot already held by the LSP open
documents domain:

~~~text
ProtosStaticAnalysisSession
    OPEN_DOCUMENTS_DOMAIN
        |
        v
ProtosDocumentSnapshot
~~~

Consequences:

~~~text
UNSAVED_EDITOR_BUFFER_SUPPORTED=YES
DISK_FALLBACK=NO
USER_MODULE_EXECUTION=NO
PROJECT_PACKAGE_RESOLUTION=NO
~~~

An unopened document URI returns no edits; the server does not attempt to read
the source from disk.

## FormattingOptions boundary

LSP `FormattingOptions` are accepted as protocol input but do not select
canonical style.

~~~text
tabSize=IGNORED_AS_STYLE_AUTHORITY
insertSpaces=IGNORED_AS_STYLE_AUTHORITY
editor_newline_preferences=IGNORED_AS_STYLE_AUTHORITY
~~~

D183 remains the sole canonical formatting-policy authority.

## Success edit model

For a changed valid document:

~~~text
EDIT_COUNT=1
EDIT_RANGE=WHOLE_ORIGINAL_DOCUMENT
EDIT_START=(0,0)
EDIT_END=UTF16_LSP_POSITION_AT_ORIGINAL_SOURCE_LENGTH
EDIT_NEW_TEXT=CANONICAL_TOOL010_SOURCE
~~~

The end position is computed through the existing
`ProtosLspSourcePositions.position` UTF-16 LSP mapping.

For an already-canonical document:

~~~text
EDIT_COUNT=0
~~~

No minimal-edit algorithm or no-op edit is introduced.

## Invalid/incomplete source

When `ProtosWholeDocumentFormatter` returns its fail-closed `FAILURE`
result:

~~~text
EDIT_COUNT=0
SOURCE_MUTATION=NONE
~~~

The formatting request does not duplicate normal parser diagnostic publication
and does not manufacture partial or recovery formatting.

## Stale snapshot protection

LM011-D1 captures one immutable snapshot and checks that the same snapshot is
still current before returning edits.

If a document changes while formatting is in flight:

~~~text
CAPTURED_SOURCE_VERSION=N
CURRENT_SOURCE_VERSION=N+1
RESULT=ZERO_EDITS
~~~

Formatting computed from the stale source is never returned for application to
the replacement source.

The numeric version remains opaque metadata; D1 introduces no new version-order
semantics.

## Genuine internal failures

A genuine formatter/bootstrap/runtime failure does not collapse into the
invalid-source empty-edit outcome.

The request completes exceptionally:

~~~text
INTERNAL_FORMATTER_FAILURE=SURFACES_AS_LSP_REQUEST_FAILURE
SILENT_ZERO_EDITS=NO
~~~

No private error-code taxonomy is introduced.

## Runtime lifecycle

The production formatting operation creates a request-scoped
`ProtosPolyglotRuntimeHost` and closes it after the single formatting
operation.

~~~text
PERSISTENT_FORMATTER_RUNTIME_HOST=NO
STATIC_ANALYSIS_SESSION_OWNS_RUNTIME=NO
BACKGROUND_FORMATTER_WORKER=NO
FORMATTER_COST_ON_NON_FORMATTING_REQUESTS=NONE
~~~

This preserves PLAT024's independently disposable static-service state and
PLAT050's pay-only-on-formatting requirement.

## Focused regression coverage

`ProtosLanguageServerFormattingTest` covers:

~~~text
initialize advertises documentFormattingProvider
range formatting remains unadvertised
on-type formatting remains unadvertised

changed open source:
  exactly one full-document edit
  canonical source from formatter authority

multiline/UTF-16 source:
  exact full-document end position

already canonical source:
  zero edits

invalid/incomplete source:
  zero edits

unopened URI:
  zero edits
  formatter not invoked

stale snapshot:
  zero edits
  latest snapshot retained

internal formatter failure:
  request completes exceptionally

real TOOL010 integration:
  current open snapshot -> real ProtosWholeDocumentFormatter -> TOOL010
  editor tabSize/insertSpaces do not change canonical style
  canonical document -> zero edits
  invalid source -> zero edits
~~~

The real-integration case uses the production
`ProtosLspDocumentFormatter.toolchain()` path rather than only a fake test
operation.

## Maintainer-reported validation

The maintainer reported after publication:

~~~text
Todos los tests han pasado en local
~~~

Retained evidence:

~~~text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
EXACT_TEST_COMMANDS_NOT_REPORTED_IN_HANDOFF=YES
~~~

No test count, duration or remote-CI state is inferred.

## Decision consistency

~~~text
D183_DELTA=NONE
D184_DELTA=NONE
PLAT050_DELTA=NONE
PLAT024_DELTA=NONE
TOOL010_POLICY_DELTA=NONE

SECOND_FORMATTER_AUTHORITY=NO
TYPESCRIPT_FORMATTER=NO
SECOND_PARSER_OR_GRAMMAR=NO
USER_MODULE_EXECUTION=NO
PROJECT_PACKAGE_RESOLUTION=NO

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Resulting LM011 state

~~~text
LM011_A=COMPLETE
LM011_B=COMPLETE
LM011_C=COMPLETE
LM011_D1=COMPLETE

STANDARD_LSP_DOCUMENT_FORMATTING=AVAILABLE

LM011_COMPLETE=NO
~~~

The next editor-facing work is LM011-D2: prove the standalone installed VS Code
extension's standard-LSP Format Document / format-on-save plumbing without
introducing extension-side formatting semantics.

At this publication point the standalone extension's committed canonical
acceptance runtime is still Protos `v0.3.139`, which predates LM011-D1.
Therefore D2 must not falsely claim actual TOOL010/D1 end-to-end evidence from
that locked runtime.

The D2 extension test may prove the thin standard-LSP editor plumbing with a
deterministic formatting-capable test server. Final real installed-VSIX
TOOL010/D1 end-to-end evidence remains LM011-E or a later exact released-runtime
gate once an appropriate published Protos runtime contains D1.

~~~text
NEXT_SLICE=LM011-D2
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos-vscode-extension
D2_PRODUCT_SEMANTICS=NONE
D2_EXTENSION_SIDE_FORMATTER=NO
D2_REAL_D1_RELEASE_RUNTIME_REQUIRED=NO_FOR_THIN_CLIENT_PLUMBING
FINAL_REAL_TOOL010_VSCODE_END_TO_END=DEFER_TO_LM011-E_OR_RELEASED_RUNTIME_GATE
~~~
