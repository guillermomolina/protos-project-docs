# LM011-E — released-runtime formatting end-to-end closure

Evidence date: **2026-10-06**

Owning workstream: `LM011 / guillermomolina/protos#670`

Governing formatter policy: `D183 / guillermomolina/protos#791`

Governing formatter architecture: `PLAT050 / guillermomolina/protos#792`

Governing language-service hosting architecture: `PLAT024 / guillermomolina/protos#342`

Nature: immutable implementation/publication evidence for the final LM011 closure slice.
This record proves that the standalone installed production VSIX reaches the exact
published Protos runtime containing LM011-D1 and exercises the canonical TOOL010
formatter through the standard LSP document-formatting path, including
format-on-save.

## Exact publications

```text
SLICE=LM011-E
WORK_TYPE=IMPLEMENTATION

IMPLEMENTATION_REPOSITORY=guillermomolina/protos-vscode-extension
EXTENSION_REVISION=c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a
COMMIT_SUBJECT=LM011-E: validate released-runtime formatting end to end
EXTENSION_VERSION=0.2.2

PROTOS_REVISION=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
PROTOS_RELEASE_VERSION=0.3.237
PROTOS_RELEASE_TAG=v0.3.237
PROTOS_RELEASE_ASSET=protos-0.3.237-native-linux-x86_64.zip
PROTOS_RELEASE_ASSET_SHA256=510537c7c47a323ed05a5ea8e4e46bc3cd63c761a58e68cb0639548feb8cc46c
PROTOS_GRAALVM_RELEASE=25.4.4.1.1
PROTOS_RUNTIME_KIND=graalvm-native-image-truffle
PROTOS_TARGET_OS=linux
PROTOS_TARGET_ARCH=x86_64
PROTOS_LIBC_FAMILY=glibc
PROTOS_LIBC_ABI_MIN=2.39
```

The extension source lock now points to the exact publicly released Protos runtime.
No snapshot, local build, CI artifact, or unpublished candidate is accepted by the
LM011-E authority.

## Released TOOL010 corpus authority

LM011-E validates the canonical released `protos format` surface against a bounded
representative corpus before editor acceptance.

```text
RELEASED_TOOL010_CORPUS_CASES=7
RELEASED_TOOL010_CORPUS=PASS
RELEASED_TOOL010_IDEMPOTENCE=PASS
RELEASED_TOOL010_REPARSE=PASS
RELEASED_TOOL010_PRESERVATION=PASS
RELEASED_TOOL010_INVALID_FAIL_CLOSED=PASS
RELEASED_TOOL010_FILE_MODE_NON_MUTATING=PASS
```

The corpus covers the D183/PLAT050 preservation and structural-formatting surfaces,
including structural whitespace, comments, raw literal spelling, Unicode,
explicit grouping, closure source form, trailing closure, semicolons, and logical
newlines.

Invalid or incomplete source remains fail-closed: the formatter returns the exact
original source on stdout, emits the formatter diagnostic, and exits with the
approved failure status. File-operand formatting remains non-mutating.

## Deterministic D2 protocol boundary retained

The earlier LM011-D2 fake-server acceptance remains useful only as a deterministic
proof of the installed-VSIX standard-LSP boundary.

Its final authority is intentionally bounded to:

```text
LM011_D2_INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
LM011_D2_UNSAVED_BUFFER_FORMATTING=PASS
LM011_D2_SCOPE=FORMAT_DOCUMENT_PROTOCOL_ONLY
LM011_D2_FORMAT_ON_SAVE_AUTHORITY=REAL_RUNTIME_E2E
STANDARD_LSP_FORMATTING_REQUEST_OBSERVED=YES
TEST_SERVER_RUNTIME=YES
REAL_TOOL010_RUNTIME=NO
```

D2 no longer duplicates the format-on-save closure claim against the fake LSP
server. During LM011-E execution, that duplicate gate proved dependent on VS Code
workbench save-participant startup ordering and was therefore not a reliable
closure authority.

This does not weaken product coverage: format-on-save is instead owned by the
stronger final released-runtime end-to-end scenario below.

## Final installed-VSIX released-runtime path

The final LM011-E acceptance path is:

```text
installed packaged Protos VSIX
        |
        | vscode-languageclient
        v
exact locked Protos v0.3.237 native runtime
        |
        | protos language-server
        v
standard textDocument/formatting
        |
        v
ProtosWholeDocumentFormatter / TOOL010
        |
        v
TextEdit applied by VS Code
```

The acceptance uses the packaged production extension rather than an extension
development path.

```text
LM011_E_INSTALLED_PACKAGED_EXTENSION=PASS
EXTENSION_DEVELOPMENT_PATH_USED=NO
TEST_SERVER_RUNTIME_FOR_FINAL_E2E=NO
REAL_TOOL010_RUNTIME=YES
PRODUCT_TYPESCRIPT_FORMATTER=NO
```

No independent TypeScript formatter, editor-side parser, second grammar, or
VS Code-specific formatting semantics were introduced.

## Format Document and unsaved-buffer proof

The installed extension exercises standard VS Code Format Document against an
unsaved Protos buffer. The formatting request reaches the exact released Protos
language server and is answered by TOOL010.

```text
LM011_E_FORMAT_DOCUMENT=PASS
LM011_E_UNSAVED_BUFFER_FORMATTING=PASS
```

The disk remains unchanged during the unsaved-buffer formatting step; the edit is
applied to the open editor buffer through the standard LSP path.

## Real format-on-save and disk persistence proof

The final authority for format-on-save uses the exact released Protos runtime, not
the deterministic fake server.

The acceptance workspace selects the installed Protos extension as the default
formatter for Protos and enables standard VS Code format-on-save behavior before
the scenario starts.

Result:

```text
LM011_E_FORMAT_ON_SAVE=PASS
LM011_E_DISK_CONTENT_CANONICAL_AFTER_SAVE=PASS
TEST_SERVER_RUNTIME_FOR_FINAL_E2E=NO
REAL_TOOL010_RUNTIME=YES
```

Therefore the final product path proves both editor-buffer formatting and
canonical persistence to disk through the released LSP/TOOL010 authority.

## Extension release identity

LM011-E changes the extension candidate version from `0.2.1` to `0.2.2` because
the committed published-runtime lock and acceptance authority changed.

```text
EXTENSION_VERSION=0.2.2
EXTENSION_RELEASE_TAG=v0.2.2
EXTENSION_RELEASE_TAG_EXISTS=NO
EXTENSION_VERSION_STATE=UNRELEASED_CANDIDATE
REL001_RELEASE_VERSION_GUARD=PASS
```

LM011-E does not publish a Marketplace release, Git tag, GitHub release, or other
extension distribution. It only establishes the correct unreleased candidate
identity.

## Final repository validation

After all LM011-E changes were complete, including the deterministic D2 authority
narrowing, the maintainer ran the repository's canonical authority:

```text
make test
```

Reported final result:

```text
REL001_RELEASE_VERSION_GUARD=PASS

RELEASED_TOOL010_CORPUS_CASES=7
RELEASED_TOOL010_CORPUS=PASS
RELEASED_TOOL010_IDEMPOTENCE=PASS
RELEASED_TOOL010_REPARSE=PASS
RELEASED_TOOL010_PRESERVATION=PASS
RELEASED_TOOL010_INVALID_FAIL_CLOSED=PASS
RELEASED_TOOL010_FILE_MODE_NON_MUTATING=PASS

LM011_D2_INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
LM011_D2_UNSAVED_BUFFER_FORMATTING=PASS
LM011_D2_SCOPE=FORMAT_DOCUMENT_PROTOCOL_ONLY
LM011_D2_FORMAT_ON_SAVE_AUTHORITY=REAL_RUNTIME_E2E

LM011_E_INSTALLED_PACKAGED_EXTENSION=PASS
LM011_E_FORMAT_DOCUMENT=PASS
LM011_E_UNSAVED_BUFFER_FORMATTING=PASS
LM011_E_FORMAT_ON_SAVE=PASS
LM011_E_DISK_CONTENT_CANONICAL_AFTER_SAVE=PASS

PROTOS_VSCODE_ACCEPTANCE_TEST=PASS
FINAL_TEST_EXIT_CODE=0
GIT_DIFF_CHECK=PASS
```

The extension commit was then published to `main` as
`c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a`, and the maintainer reported a clean
post-push worktree.

No executable or test changes were made after the successful final repository
authority before publication.

## Decision and architecture consistency

```text
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

OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
PROTOS_SPECIFICATION_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
```

The final extension remains a thin standard-LSP client. Formatting semantics stay
owned by TOOL010 in Protos.

## LM011 closure

All planned LM011 slices are complete.

```text
LM011_A=COMPLETE
LM011_B=COMPLETE
LM011_C=COMPLETE
LM011_D=COMPLETE
LM011_E=COMPLETE

CANONICAL_FORMATTER=TOOL010
CANONICAL_CLI_FORMATTING_SURFACE=AVAILABLE
STANDARD_LSP_DOCUMENT_FORMATTING=AVAILABLE
INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
INSTALLED_VSIX_UNSAVED_BUFFER_FORMATTING=PASS
INSTALLED_VSIX_FORMAT_ON_SAVE=PASS
RELEASED_RUNTIME_TOOL010_END_TO_END=PASS

LM011_COMPLETE=YES
GUILLERMOMOLINA_PROTOS_ISSUE_670_READY_TO_CLOSE=YES
FOLLOWUP_LM011_SLICE_REQUIRED=NO
```

There is no remaining LM011 implementation, cleanup, documentation, or evidence
slice required for closure.

AI assistance: this durable publication record was drafted with ChatGPT from the
exact published LM011-E extension commit, the committed runtime lock, existing
LM011/D183/D184/PLAT050/PLAT024 durable records, live GitHub coordination, and
maintainer-reported final validation output.
