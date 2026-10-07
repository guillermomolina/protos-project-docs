# LM011-C — canonical formatter CLI publication

Evidence date: **2026-10-04**

Owning workstream: LM011 / guillermomolina/protos#670

Governing CLI decision: D184 / guillermomolina/protos#796

Governing formatter policy: D183 / guillermomolina/protos#791

Governing formatter architecture: PLAT050 / guillermomolina/protos#792

Nature: immutable implementation/publication evidence for the LM011-C public
canonical formatter CLI adapter.

## Exact publication

~~~text
SLICE=LM011-C
WORK_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

PROTOS_REVISION=8f19524fd78f638ed6d4ec6dc1deaf31413a3eb6
COMMIT_SUBJECT=LM011-C: expose canonical formatter CLI
PROTOS_VERSION=0.3.199-SNAPSHOT

D184_STATUS=RATIFIED
D184_SELECTED_CANDIDATE=B_STDOUT_DEFAULT_SINGLE_SOURCE

MAINTAINER_REPORTED_LOCAL_TESTS=PASS
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

LM011-C was published on top of the concurrently published TEST009-C revision:

~~~text
IMMEDIATE_PRECEDING_REVISION=bdeed4534beeba74949c2c65d74d7eb89d95b589
IMMEDIATE_PRECEDING_SUBJECT=TEST009-C: guard Bytecode API PE arguments
~~~

The final LM011-C commit therefore uses the live repository-global
0.3.199-SNAPSHOT version rather than the version visible when the slice started.

## Published files

The exact LM011-C commit changes:

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/test/java/com/guillermomolina/protos/cli/ProtosCliTest.java
~~~

No bundled formatter-policy source, parser, grammar, specification, LSP server
or VS Code extension source changed in this slice.

## Public command

The existing one-public-driver surface now includes:

~~~text
protos format
protos format <file>
~~~

Help advertises:

~~~text
protos format [<file>]
~~~

No fmt alias, generic public Tool namespace, write mode, check mode or
multi-file form is added.

## Input contract implemented

With zero operands, protos format reads one complete UTF-8 source document from
stdin. The decoder is fail-closed for malformed and unmappable input.

~~~text
ZERO_OPERANDS=STDIN
STDIN_DOCUMENT_ID=<stdin>
STDIN_UTF8_MALFORMED_ACTION=REPORT
~~~

With one operand, protos format file.protos resolves that explicit path,
requires a regular file and reads it as UTF-8. A symlink resolving to a regular
file is accepted for reading through ordinary Files.isRegularFile behavior.

The source file is never executed and no user-project import/package graph is
resolved.

More than one source operand is rejected through the CLI usage boundary:

~~~text
MULTI_FILE=NO
MULTIPLE_OPERANDS_EXIT=2
~~~

## Canonical formatter authority

The CLI adapter invokes the already-published editor-neutral
ProtosWholeDocumentFormatter.

The command does not implement formatting rules in ProtosCli and does not call
the bundled Structural policy directly.

~~~text
CLI adapter
    |
    v
ProtosDocumentSnapshot
    |
    v
ProtosWholeDocumentFormatter
    |
    v
TOOL010 exact bundled formatter policy
~~~

This preserves the single formatter authority selected by D183 and PLAT050.

## Success behavior

On a valid document:

~~~text
stdout = exact source returned by ProtosWholeDocumentFormatter
stderr = empty
exit   = 0
filesystem mutation = none
~~~

The CLI emits the returned characters verbatim rather than adding an additional
newline beyond TOOL010's canonical result.

An already-canonical source remains a normal successful formatting operation.

## Invalid/incomplete source

When the whole-document authority returns FAILURE:

~~~text
stdout = exact original source returned by the formatter result
stderr = protos format diagnostic with inert reason
exit   = 1
filesystem mutation = none
~~~

The failure is not translated into partial formatting and is not presented as
an internal error.

The existing B2 behavior remains authoritative: parse failure occurs before
TOOL010 policy execution and returns the original snapshot unchanged.

## Input acquisition failures

For missing, unreadable, non-regular or malformed-UTF-8 input:

~~~text
stdout = empty
stderr = protos format: cannot read ... diagnostic
exit   = 1
~~~

Directory operands are rejected as non-regular input.

Malformed UTF-8 is not silently replaced.

## Genuine internal failures

The adapter deliberately does not collapse formatter/bootstrap IOException or
unexpected host failures into ordinary input failure.

Such failures propagate to the pre-existing dispatchCommand internal-error
boundary:

~~~text
stderr = Internal error: ...
exit   = 70
~~~

LM011-C introduces no new formatter-specific numeric exit class.

## Exit-status matrix

~~~text
FORMAT_SUCCESS=0
INVALID_OR_INCOMPLETE_SOURCE=1
INPUT_IO_OR_DECODE_FAILURE=1
USAGE_ERROR=2
GENUINE_FORMATTER_BOOTSTRAP_OR_INTERNAL_FAILURE=70
~~~

The existing Test Tool-specific infrastructure-abort exit 3 remains unrelated
and unchanged.

## Filesystem and mutation boundary

~~~text
DEFAULT_MUTATION=NONE
FILESYSTEM_WRITE_CONTRACT=NONE
ATOMIC_REPLACEMENT=NOT_APPLICABLE
BACKUP=NONE
TEMP_RENAME=NONE
DIRECTORY_RECURSION=NONE
WORKSPACE_DISCOVERY=NONE
PACKAGE_DISCOVERY=NONE
~~~

The source file is read-only from the formatter command's perspective.

## Focused regression coverage

ProtosCliTest was extended to cover:

~~~text
help advertises protos format [<file>]

file input:
  non-canonical valid source -> canonical stdout
  stderr empty
  exit 0
  original file bytes unchanged

stdin input:
  non-canonical valid source -> canonical stdout
  stderr empty
  exit 0

already canonical source:
  exact source retained
  exit 0

symlink -> regular file:
  accepted for reading
  target remains unchanged

invalid file source:
  exit 1
  exact original source on stdout
  protos format diagnostic on stderr
  file unchanged

invalid stdin source:
  exit 1
  exact original source on stdout
  diagnostic on stderr

too many operands:
  exit 2
  stdout empty

missing path:
  exit 1
  stdout empty

directory operand:
  exit 1
  stdout empty

malformed UTF-8 file:
  exit 1
  stdout empty
  original bytes unchanged

malformed UTF-8 stdin:
  exit 1
  stdout empty
~~~

The symlink case is skipped only when the executing platform does not provide
the required symbolic-link facility.

## Maintainer-reported validation

The maintainer reported after publication:

~~~text
Todos los tests han pasado en local
~~~

Retained evidence classification:

~~~text
MAINTAINER_REPORTED_LOCAL_TESTS=PASS
EXACT_TEST_COMMANDS_NOT_REPORTED_IN_HANDOFF=YES
~~~

No specific test count, elapsed time or remote-CI result is inferred from that
summary.

## Decision consistency

LM011-C implements D184 without reopening formatter policy or architecture.

~~~text
D184_DELTA=NONE
D183_DELTA=NONE
PLAT050_DELTA=NONE
PLAT024_DELTA=NONE
TOOL010_POLICY_DELTA=NONE

SECOND_FORMATTER_AUTHORITY=NO
HOST_FORMATTER_POLICY=NO
SECOND_PARSER_OR_GRAMMAR=NO
USER_MODULE_EXECUTION=NO
PROJECT_PACKAGE_RESOLUTION=NO

DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Explicitly still deferred

LM011-C does not implement:

~~~text
--write / in-place mutation
--check
multi-file formatting
directory or recursive formatting
workspace/project formatting
package discovery
range formatting
on-type formatting
formatter directives
style configuration
LSP textDocument/formatting
VS Code Format Document integration
VS Code format-on-save integration
~~~

Those LSP/editor surfaces remain LM011-D.

## Resulting LM011 state

~~~text
LM011_A=COMPLETE
LM011_B=COMPLETE
LM011_C=COMPLETE

CURRENT_FORMATTER=CANONICAL_WHOLE_DOCUMENT_TOOL010
PUBLIC_CLI_FORMATTER=AVAILABLE

LM011_COMPLETE=NO

NEXT_SLICE=LM011-D
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_SCOPE=STANDARD_LSP_TEXTDOCUMENT_FORMATTING_AND_VSCODE_FORMAT_DOCUMENT_FORMAT_ON_SAVE
~~~

PLAT050 already selected the required LM011-D architecture:

~~~text
VS Code / another standard LSP client
        |
        | textDocument/formatting
        v
thin LSP adapter
        |
        v
same editor-neutral ProtosWholeDocumentFormatter authority
~~~

No independent TypeScript formatter is authorized.
