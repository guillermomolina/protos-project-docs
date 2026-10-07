# LM011-D2 — installed VSIX standard-LSP formatting acceptance

Evidence date: **2026-10-04**

Owning workstream: `LM011 / guillermomolina/protos#670`

Governing formatter policy: `D183 / guillermomolina/protos#791`

Governing formatter architecture: `PLAT050 / guillermomolina/protos#792`

Governing language-service hosting architecture: `PLAT024 / guillermomolina/protos#342`

Nature: immutable implementation/publication evidence for LM011-D2, proving the
standalone **installed production VSIX** follows the standard
`vscode-languageclient` document-formatting path without adding an independent
VS Code-side formatter.

## Exact publication

~~~text
SLICE=LM011-D2
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-vscode-extension

EXTENSION_REVISION=07ff2ce1bd284cc4ccd5dcfdd85ba242c2e3ee30
COMMIT_SUBJECT=LM011-D2: validate VS Code formatter plumbing
EXTENSION_VERSION=0.2.1

MAINTAINER_REPORTED_MAKE_TEST=PASS

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
PROTOS_SPECIFICATION_CHANGE=NO
PRODUCT_EXTENSION_JS_CHANGE=NO
PRODUCT_PACKAGE_JSON_CHANGE=NO
PACKAGE_VERSION_CHANGE=NO
PROTOS_SOURCE_LOCK_CHANGE=NO
~~~

The production extension version remains `0.2.1`. D2 changes acceptance/test
infrastructure only and does not publish a new extension product release.

## Published files

The exact LM011-D2 commit changes:

~~~text
scripts/test_acceptance.js
test/formatting_fake_server.py
test/formatting_harness_extension.js
test/formatting_harness_make_vsix.py
test/formatting_harness_package.json
test/formatting_harness_validate.py
~~~

No production `extension.js`, production `package.json`, runtime lock, grammar,
parser, Protos formatter-policy source, language specification, README or
release metadata changes are part of D2.

## Installed-VSIX boundary proven

D2 extends the existing canonical installed-VSIX acceptance with an additional
scenario:

~~~text
REAL FORMAT DOCUMENT / FORMAT ON SAVE
~~~

The tested path is:

~~~text
installed canonical Protos VSIX
        |
        | vscode-languageclient
        v
server advertises documentFormattingProvider=true
        |
        v
standard textDocument/formatting
        |
        v
VS Code applies returned TextEdit
~~~

The production extension is installed from the canonical
`protos-vscode.vsix`. No `--extensionDevelopmentPath` is used.

~~~text
INSTALLED_REAL_VSIX=YES
EXTENSION_DEVELOPMENT_PATH_USED=NO
STANDARD_VSCODE_LANGUAGECLIENT_PATH=YES
~~~

## Test-only LSP runtime

Because the production extension's committed canonical runtime lock still
targets Protos `v0.3.139`, which predates LM011-D1, D2 uses a deterministic
scenario-local fake executable only for the new formatting scenario.

The executable accepts the exact production launch shape:

~~~text
<runtime> language-server
shell=false
~~~

It implements only the protocol surface needed by the acceptance scenario:

~~~text
initialize
initialized
textDocument/didOpen
textDocument/didChange
textDocument/formatting
shutdown
exit
~~~

Its initialize response advertises:

~~~text
textDocumentSync.openClose=true
textDocumentSync.change=Full
documentFormattingProvider=true
~~~

It does not advertise range or on-type formatting.

The fake server is copied into the temporary scenario root. It is not shipped
in the production VSIX and is not a Protos formatter implementation or semantic
authority.

~~~text
FAKE_LSP_SERVER=TEST_ONLY
FAKE_SERVER_SHIPPED_IN_PRODUCTION_VSIX=NO
TEST_SERVER_RUNTIME=YES
REAL_TOOL010_RUNTIME=NO
~~~

## Format Document acceptance

The installed extension is activated against the scenario-local test server.
A Protos document is opened and then changed in memory to:

~~~text
value:1
~~~

The harness executes the standard VS Code command:

~~~text
editor.action.formatDocument
~~~

The fake LSP server observes a real:

~~~text
textDocument/formatting
~~~

request carrying the current unsaved buffer state.

VS Code applies the returned whole-document edit, producing exactly:

~~~text
value: 1\n
~~~

Result:

~~~text
LM011_D2_INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
LM011_D2_UNSAVED_BUFFER_FORMATTING=PASS
STANDARD_LSP_FORMATTING_REQUEST_OBSERVED=YES
~~~

This proves that the extension does not need to read the source file or own
formatter semantics to support Format Document.

## format-on-save acceptance

The acceptance workspace selects the installed Protos extension as the default
formatter for Protos and enables standard VS Code `editor.formatOnSave`
behavior.

The harness changes the document to non-canonical:

~~~text
value:1
~~~

and saves it.

The server observes a second real `textDocument/formatting` request. VS Code
applies the edit before the save completes, and both the open document and disk
content become exactly:

~~~text
value: 1\n
~~~

Result:

~~~text
LM011_D2_FORMAT_ON_SAVE=PASS
FORMAT_REQUEST_COUNT=2
~~~

No `onWillSaveTextDocument` formatting implementation or independent editor
formatter is introduced.

## Thin-client authority preserved

Repository validation confirmed there is no product-side registration of:

~~~text
registerDocumentFormattingEditProvider
~~~

D2 therefore preserves the PLAT050 thin-client architecture:

~~~text
PRODUCT_TYPESCRIPT_FORMATTER=NO
PRODUCT_PARSER=NO
PRODUCT_FORMATTING_POLICY=NO
INDEPENDENT_TYPESCRIPT_FORMATTER=NO
~~~

Formatting semantics remain owned by the configured Protos language server.
D2 proves only the standard client/editor plumbing.

## Existing acceptance invariants preserved

The existing scenarios continue to use the published runtime locked by
`protos-source.lock.json`:

~~~text
LOCKED_PROTOS_RELEASE=v0.3.139
LOCKED_PROTOS_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
PROTOS_SOURCE_LOCK_CHANGED=NO
~~~

The final `make test` retained the prior installed-VSIX gates:

~~~text
CLEAN_INSTALL=PASS
REAL_RUN=PASS
REAL_DEBUG=PASS
~~~

The new formatting scenario is additive and uses its test server only through
per-scenario runtime selection.

## Full repository validation

Before publication the maintainer ran the repository's canonical authority:

~~~text
make test
~~~

The repository baseline passed, including build, grammar, contract, packaging,
release-version, Run, Debug and language-client tests.

The installed-VSIX acceptance then passed all four scenarios.

Relevant retained output:

~~~text
LM009_I3A_CLEAN_INSTALL=PASS
LM009_I3B_REAL_RUN=PASS
LM009_I3C_REAL_DEBUG=PASS

LM011_D2_INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
LM011_D2_UNSAVED_BUFFER_FORMATTING=PASS
LM011_D2_FORMAT_ON_SAVE=PASS
STANDARD_LSP_FORMATTING_REQUEST_OBSERVED=YES
FORMAT_REQUEST_COUNT=2
INDEPENDENT_TYPESCRIPT_FORMATTER=NO
EXTENSION_DEVELOPMENT_PATH_USED=NO
TEST_SERVER_RUNTIME=YES
REAL_TOOL010_RUNTIME=NO

PROTOS_VSCODE_ACCEPTANCE_TEST=PASS
MAKE_TEST=PASS
~~~

No executable/test bytes were changed after this successful validation before
publication.

## What D2 deliberately does not prove

D2 does **not** prove:

~~~text
REAL_INSTALLED_VSIX_TOOL010_D1_END_TO_END=PASS
~~~

The locked production acceptance runtime remains Protos `v0.3.139`, with
revision:

~~~text
3895206897ddac795dfebd49709ca97f8d0908b1
~~~

That runtime predates the LM011-D1 server publication:

~~~text
LM011_D1_PROTOS_REVISION=75cfed853c7eef8c9d3dfb0c6f2507fdf8864ecb
~~~

The correct composed evidence remains:

~~~text
D1:
  real Protos LSP -> real TOOL010 = PASS

D2:
  installed real VSIX -> standard LSP formatting plumbing = PASS
~~~

The real installed-VSIX -> published Protos D1 -> TOOL010 composition remains a
final LM011-E/released-runtime gate.

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
LM011_D2=COMPLETE
LM011_D=COMPLETE

STANDARD_LSP_DOCUMENT_FORMATTING=AVAILABLE_IN_D1
INSTALLED_VSIX_FORMAT_DOCUMENT_PLUMBING=PASS
INSTALLED_VSIX_FORMAT_ON_SAVE_PLUMBING=PASS

LM011_COMPLETE=NO
~~~

The remaining closure work is LM011-E. It must include the final real composed
installed-VSIX end-to-end gate once a published Protos runtime containing D1 is
available; D2 does not authorize changing the extension runtime lock to a
snapshot or inventing a release.

~~~text
NEXT_SLICE=LM011-E
NEXT_SLICE_STATUS=BLOCKED_BY_PUBLISHED_PROTOS_RUNTIME_CONTAINING_LM011_D1
REAL_TOOL010_INSTALLED_VSIX_END_TO_END=REQUIRED_FOR_CLOSURE
PROTOS_SOURCE_LOCK_UPDATE_REQUIRES_REAL_RELEASE=YES
NEW_FORMATTER_POLICY_DECISION_REQUIRED=NO
~~~

AI assistance: this durable publication record was drafted with ChatGPT from
the exact published LM011-D2 extension commit, existing LM011/D183/PLAT050
records, and maintainer-reported final validation output.
