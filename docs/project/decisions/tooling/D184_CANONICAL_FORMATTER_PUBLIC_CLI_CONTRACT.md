# D184 — Canonical formatter public CLI contract

Status: **RATIFIED — Candidate B selected**

Explicit project-owner approval: **2026-10-04**

Decision issue: `guillermomolina/protos#796`

Parent work item: `LM011 / guillermomolina/protos#670`

Released implementation slice: `LM011-C`

Decision-packet product baseline:

~~~text
PROTOS_REVISION=5813f154e862e0479d5ebd6c9426eb3c3873156d
COMMIT_SUBJECT=LM011-B2: add canonical source formatter
TOOL010_STATUS=IMPLEMENTED
~~~

Moving-HEAD compatibility review at ratification:

~~~text
CURRENT_PROTOS_REVISION=38abecd1697993ddba5c0bc549aff662c98af535
INTERVENING_COMMIT=TEST009-B: guard LocalAccessor/MaterializedLocalAccessor PE operands
D184_DEPENDENCY_SURFACE_DELTA=NONE
MOVING_HEAD_REVIEW=PASS
~~~

Nature: durable non-normative implementation-independent tooling/CLI decision.

Observable Protos language effect: **none**.

Specification change required: **none**.

## Decision

D184 selects **Candidate B — stdout-default single-source formatting**.

The public baseline command is:

~~~text
protos format [<file>]
~~~

The command exposes the already-published editor-neutral TOOL010 whole-document
formatter authority through the public `protos` driver. The CLI owns only command
selection, source acquisition, terminal stream behavior and exit-status
translation. It does not own formatting policy and must not duplicate D183 or
TOOL010.

The baseline is deliberately non-mutating.

~~~text
DEFAULT_MUTATION=NONE
FILESYSTEM_WRITE_CONTRACT=NONE
~~~

An explicit write mode may be considered later without changing the baseline
contract selected here.

## Fixed authorities preserved

D184 does not reopen or modify:

- D183/#791 Candidate B — fixed structural style plus conservative lexical
  preservation;
- PLAT050/#792 Candidate F — on-demand hybrid source-layout view plus exact
  bundled Protos formatter policy over tool-neutral host source mechanics;
- TOOL010 as the single canonical formatter-policy authority;
- `ProtosWholeDocumentFormatter` as the editor-neutral whole-document result
  authority;
- PLAT024's thin standard-LSP boundary;
- fail-closed invalid/incomplete-source behavior;
- no execution of the user's module;
- no project package-graph dependency for formatting;
- no TypeScript formatter;
- no second parser, grammar or formatter;
- no host-owned `FormatterPolicy`.

~~~text
D183_DELTA=NONE
PLAT050_DELTA=NONE
PLAT024_DELTA=NONE
TOOL010_POLICY_DELTA=NONE
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Public command spelling

The selected spelling is:

~~~text
protos format
protos format <file>
~~~

`format` is a dedicated top-level command of the one public `protos` driver.

D184 does not expose TOOL010 through a generic public `tool` namespace and does
not reserve or require a `fmt` alias.

Internal TOOL ownership remains independent of public command spelling, as
required by the selected Toolchain Tool Architecture.

## Input model

The baseline accepts exactly one source document.

### Zero operands

~~~text
protos format
~~~

reads the complete source document from standard input.

~~~text
ZERO_OPERANDS=STDIN
STDIN_ENCODING=UTF-8
STDIN_LOGICAL_DIAGNOSTIC_NAME=<stdin>
~~~

No stdin filename/path option is part of the baseline because D183 style,
parsing and TOOL010 selection do not depend on a filename, project configuration
or package graph.

### One operand

~~~text
protos format file.protos
~~~

reads exactly that explicit file as UTF-8 source.

~~~text
ONE_OPERAND=ONE_EXPLICIT_FILE
~~~

### More than one operand

~~~text
protos format a.protos b.protos
~~~

is a CLI usage error.

~~~text
MULTI_FILE_BASELINE=NO
N_OPERANDS_GREATER_THAN_1=USAGE_ERROR
EXIT_STATUS=2
~~~

D184 does not define multi-file output framing, aggregate failures, partial
success or traversal order.

## Output and mutation model

For successful formatting, stdout contains the canonical source and nothing
else.

~~~text
SUCCESS_STDOUT=CANONICAL_SOURCE_ONLY
SUCCESS_STDERR=EMPTY
SUCCESS_EXIT=0
MUTATION=NONE
~~~

The CLI must not add banners, filenames, progress messages or diagnostics to
stdout.

A source that is already canonical is still a normal success:

~~~text
ALREADY_FORMATTED=EXIT_0
CHANGED_UNCHANGED_CLASSIFICATION=NOT_PART_OF_BASELINE
~~~

That distinction belongs to any future check/no-write contract, not to D184's
baseline format operation.

## Invalid or incomplete source

D183 and TOOL010 already require invalid/incomplete source to fail closed before
formatter execution.

At the CLI boundary D184 translates that result as:

~~~text
FORMAT_RESULT=FAILURE
TOOL010_PROCESS_STARTED=NO
STDOUT=EXACT_ORIGINAL_SOURCE
STDERR=COMMAND_DIAGNOSTIC_WITH_INERT_REASON
EXIT_STATUS=1
INPUT_FILE_MUTATED=NO
~~~

The exact original source on stdout is deliberately not claimed to be canonical.
Automation must inspect the non-zero exit status.

This preserves source continuity for filter-style use while preventing partial,
guessed or repaired formatting.

## I/O and usage failures

If the source cannot be acquired completely, no source is emitted.

~~~text
MISSING_PATH:
  stdout=EMPTY
  stderr=DIAGNOSTIC
  exit=1

READ_FAILURE:
  stdout=EMPTY
  stderr=DIAGNOSTIC
  exit=1

UNSUPPORTED_PATH_KIND:
  stdout=EMPTY
  stderr=DIAGNOSTIC
  exit=1

USAGE_ERROR:
  stdout=EMPTY
  stderr=USAGE_DIAGNOSTIC
  exit=2
~~~

Ordinary user/input/I/O failures reuse the existing Protos CLI exit class 1.
D184 does not invent new numeric status classes solely for the formatter.

## Genuine TOOL010/bootstrap/internal failure

A genuine formatter bootstrap, bundled-tool identity or other internal failure
must not be misclassified as invalid source.

~~~text
TOOL010_BOOTSTRAP_OR_INTERNAL_FAILURE:
  stdout=EMPTY
  stderr=INTERNAL_DIAGNOSTIC
  exit=70
~~~

This reuses the current `ProtosCli` internal-error class.

The shell launcher behavior that occurs before `ProtosCli` starts remains
outside D184.

## stdout / stderr discipline

The selected terminal contract is:

| Outcome | stdout | stderr | exit |
|---|---|---|---:|
| format success | canonical source only | empty | 0 |
| invalid/incomplete fail-closed | exact original source only | diagnostic | 1 |
| missing/read/unsupported input | empty | diagnostic | 1 |
| usage error | empty | usage diagnostic | 2 |
| TOOL010/bootstrap/internal failure | empty | internal diagnostic | 70 |

Source and diagnostics never share stdout.

This permits ordinary filter composition such as:

~~~text
cat file.protos | protos format > formatted.protos
protos format file.protos > formatted.protos
~~~

A caller must never overwrite the same input path using shell redirection such
as `protos format file.protos > file.protos`; shell truncation can occur before
Protos opens the source. A future explicit write mode is the proper place to
provide safe same-path mutation.

## File/path boundary

The one-file baseline intentionally minimizes filesystem semantics.

~~~text
REGULAR_FILE=ACCEPT
SYMLINK_RESOLVING_TO_REGULAR_FILE=ACCEPT_FOR_READ
BROKEN_SYMLINK=REJECT
DIRECTORY=REJECT
FIFO_DEVICE_SOCKET_NON_REGULAR=REJECT
READ_FAILURE=REJECT
WRITE_FAILURE=NOT_APPLICABLE
ATOMIC_REPLACEMENT=NOT_APPLICABLE
PERMISSION_METADATA_PRESERVATION=NOT_APPLICABLE
~~~

Acceptance of a symlink for read does not preselect any future in-place-write
semantics for symlinks.

Streaming through a named FIFO/device is unnecessary because zero operands
already provide the explicit stdin streaming boundary.

## Encoding and newline boundary

CLI source acquisition is UTF-8, consistent with the current driver boundary.

D184 introduces no formatter-owned newline rules. The CLI passes source to the
existing formatter authority and emits the returned source without performing an
independent newline normalization.

The already-ratified D183 contract remains:

~~~text
STRUCTURAL_OUTPUT_EOL=LF
PRESERVED_LEXICAL_PAYLOAD_PHYSICAL_NEWLINE=EXACT
FINAL_FORMATTER_OWNED_NEWLINE=EXACTLY_ONE_LF
~~~

On fail-closed invalid/incomplete source, the exact original source is returned;
the CLI does not repair or normalize it.

## Workspace and package graph

~~~text
WORKSPACE_PACKAGE_DISCOVERY=NONE
PACKAGE_GRAPH_DISCOVERY=NONE
DIRECTORY_RECURSION=NONE
~~~

TOOL010 already bootstraps independently from the user's package graph. D184
keeps the public baseline file/stdin-oriented and does not introduce project or
workspace discovery.

## Formatter options

The baseline has no formatter-style options.

~~~text
FORMATTER_OPTIONS=NONE
STYLE_CONFIGURATION=NONE_BY_D183
LSP_TAB_SIZE_STYLE_AUTHORITY=NO
LSP_INSERT_SPACES_STYLE_AUTHORITY=NO
~~~

CLI mechanics must not become an alternate style-policy surface.

## Explicitly deferred capabilities

The following are not part of Candidate B:

~~~text
--write / in-place mutation
multi-file formatting
directory traversal
recursive formatting
workspace/project discovery
package discovery
check/no-write status mode
range formatting
on-type formatting
formatter directives
project style configuration
editor style configuration
generic public bundled-tool namespace
fmt alias
~~~

Deferral is intentional. Each capability can be added later without changing
the selected meaning of `protos format` and `protos format <file>`.

## Candidate comparison outcome

The investigated candidate set was:

- **A** — `protos format <file>` writes in place by default;
- **B** — stdout-default, one-source baseline with stdin when no operand is
  supplied;
- **C** — Candidate B plus an explicit `--write` mode;
- **D** — stdin-first formatter with paths secondary;
- **E** — generic bundled-tool namespace;
- **F** — defer the public CLI.

Candidate B was recommended and explicitly approved because it is the smallest
surface that exposes TOOL010 while preserving shell composition, avoiding
accidental mutation and leaving filesystem-write policy unselected until a real
write requirement exists.

Candidate C remains the strongest future extension. Its explicit write mode can
be added later, but doing so must then define safe filesystem mutation rather
than treating `--write` as a policy-free flag.

## Incremental-design result

~~~text
SMALLEST_SUFFICIENT_SOLUTION=
  ONE_COMMAND
  + ONE_SOURCE
  + ONE_FORMATTER_AUTHORITY
  + ONE_SOURCE_OUTPUT_STREAM
  + ZERO_WRITE_MACHINERY

PAY_FOR_WHAT_YOU_NEED=PASS
GROW_AS_YOU_NEED=PASS
REVERSIBILITY=PASS
AUTOMATION_COMPOSABILITY=PASS
ACCIDENTAL_MUTATION_RISK=MINIMIZED
~~~

No formatter, parser, source-layout, package-resolution or LSP authority must be
rewritten to add an explicit write/check/multi-file capability later.

## Editor and CI reuse

D184 does not make the LSP adapter invoke the CLI process.

The shared shape remains:

~~~text
                 ProtosWholeDocumentFormatter
                    /                  \
             CLI adapter            LSP adapter
~~~

Both consumers reuse one editor-neutral formatter authority.

The CLI's stdout behavior is therefore an external terminal contract, not an
intermediate formatter API.

## Implementation consequence

Ratification releases exactly:

~~~text
NEXT_SLICE=LM011-C
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
IMPLEMENTATION_AUTHORIZED=YES
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

LM011-C may add the minimal `format` dispatch, one-source input acquisition,
whole-document formatter invocation, stdout/stderr/exit translation, help text
and contract tests.

It does not authorize any deferred capability above.

~~~text
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
SPECIFICATION_CHANGE_REQUIRED=NO
~~~
