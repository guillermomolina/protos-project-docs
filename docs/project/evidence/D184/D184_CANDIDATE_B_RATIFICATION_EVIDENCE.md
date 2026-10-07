# D184 — Candidate B ratification evidence

Evidence date: **2026-10-04**

Owning decision: `D184 / guillermomolina/protos#796`

Parent workstream: `LM011 / guillermomolina/protos#670`

Nature: immutable project evidence for project-owner selection, moving-HEAD
consistency review, durable D184 publication and release of LM011-C.

## Explicit project-owner selection

The project owner explicitly approved:

~~~text
Apruebo Candidate B
~~~

Selected candidate:

~~~text
Candidate B — stdout-default single-source formatting
~~~

The approval applies to the exact Candidate B contract presented in the D184
decision packet, including:

~~~text
PUBLIC_COMMAND_SPELLING=protos format [<file>]

0 operands = stdin
1 operand  = one explicit file
>1         = usage error / exit 2

success:
  stdout = canonical source only
  stderr = empty
  exit   = 0

invalid/incomplete fail-closed:
  stdout = exact original source only
  stderr = diagnostic
  exit   = 1
  mutation = none

input/read/unsupported-path failure:
  stdout = empty
  stderr = diagnostic
  exit   = 1

usage failure:
  stdout = empty
  stderr = diagnostic
  exit   = 2

genuine TOOL010/bootstrap/internal failure:
  stdout = empty
  stderr = diagnostic
  exit   = 70

default mutation = none
multi-file = deferred
directories/recursion = none
workspace/package discovery = none
formatter options = none
check mode = deferred
range formatting = deferred
style configuration = none by D183
~~~

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
SELECTED_CANDIDATE=B
D184_STATUS=RATIFIED
~~~

## Product baseline used by the decision packet

The D184 investigation and recommendation were completed against:

~~~text
PROTOS_REVISION=5813f154e862e0479d5ebd6c9426eb3c3873156d
COMMIT_SUBJECT=LM011-B2: add canonical source formatter
PROTOS_VERSION=0.3.197-SNAPSHOT
~~~

At that revision:

~~~text
D183_STATUS=RATIFIED
PLAT050_STATUS=RATIFIED
TOOL010_STATUS=IMPLEMENTED
LM011_B=COMPLETE
LM011_C_STATUS=BLOCKED_BY_D184
~~~

## Moving-HEAD review before durable ratification

Immediately before durable publication, `guillermomolina/protos` had advanced
by exactly one commit:

~~~text
CURRENT_PROTOS_REVISION=38abecd1697993ddba5c0bc549aff662c98af535
INTERVENING_COMMIT=TEST009-B: guard LocalAccessor/MaterializedLocalAccessor PE operands
~~~

The comparison from the decision-packet baseline to current HEAD changed only:

~~~text
Makefile
tools/java_local_accessor_pe_baseline.json
tools/java_local_accessor_pe_guard.py
tools/test_java_local_accessor_pe_guard.py
~~~

It did not change:

~~~text
src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
src/main/java/com/guillermomolina/protos/execution/ProtosWholeDocumentFormatter.java
src/main/java/com/guillermomolina/protos/execution/ProtosFormatterToolBootstrap.java
protos/tools/formatter/Main.protos
protos/tools/formatter/Structural.protos
AGENTS.work/TOOL.md
AGENTS.work/DESIGN.md
docs/design/TOOLCHAIN_TOOL_ARCHITECTURE.md
D183 ratified contract
PLAT050 ratified architecture
~~~

Therefore the intervening TEST009-B publication introduces no D184-relevant
semantic, architectural, CLI or formatter-policy delta.

~~~text
MOVING_HEAD_REVIEW=PASS
D184_RESEARCH_REOPEN_REQUIRED=NO
OWNER_REAPPROVAL_REQUIRED_FOR_HEAD_DRIFT=NO
~~~

## Current CLI compatibility

The current public driver already establishes these exit classes:

~~~text
0  success
1  ordinary user/tool/input/I/O failure
2  CLI usage error
70 internal error
~~~

The Test Tool additionally owns its independent infrastructure-abort exit 3,
which D184 does not reuse.

Candidate B therefore introduces no formatter-only numeric exit taxonomy.

## D183 / PLAT050 / TOOL010 consistency

The selected Candidate B preserves every applicable fixed authority:

~~~text
D183_FIXED_STYLE=KEEP
D183_CONSERVATIVE_SOURCE_PRESERVATION=KEEP
D183_INVALID_SOURCE_FAIL_CLOSED=KEEP
D183_DETERMINISM=KEEP
D183_IDEMPOTENCE=KEEP
D183_CHECK_MODE_DEFERRED=KEEP
D183_RANGE_FORMATTING_DEFERRED=KEEP

PLAT050_ON_DEMAND_HYBRID_SOURCE_LAYOUT=KEEP
PLAT050_EXACT_BUNDLED_PROTOS_POLICY=KEEP
PLAT050_TOOL_NEUTRAL_HOST_MECHANISM=KEEP
PLAT050_NO_PROJECT_PACKAGE_GRAPH=KEEP
PLAT050_NO_USER_MODULE_EXECUTION=KEEP

TOOL010_ONE_FORMATTER_AUTHORITY=KEEP
PROTOS_WHOLE_DOCUMENT_FORMATTER_AUTHORITY=KEEP
PLAT024_THIN_LSP_BOUNDARY=KEEP
HOST_OWNED_FORMATTER_POLICY=NO
SECOND_PARSER_OR_GRAMMAR=NO
TYPESCRIPT_FORMATTER=NO
~~~

~~~text
DECISION_INVARIANT_CONSISTENCY=PASS
D183_DELTA=NONE
PLAT050_DELTA=NONE
TOOL010_DELTA=NONE
PLAT024_DELTA=NONE
~~~

## Comparative evidence retained by the decision

The investigation compared the required formatter families:

~~~text
gofmt
rustfmt
Black
clang-format
Prettier
~~~

The comparison covered command spelling, mutation defaults, stdin/stdout,
multi-file and recursive behavior, invalid syntax, exit status, diagnostics,
configuration, check modes, editor reuse, CI ergonomics and safe-write
implications.

The evidence showed no universal convention that formatting a path must overwrite
that path. In particular, stdout-default plus explicit future write mode is a
mature model, while in-place defaults imply additional filesystem policy that is
not required by LM011-C.

The strongest alternative remained Candidate C:

~~~text
protos format <file>          -> stdout
protos format --write <file>  -> explicit mutation
~~~

Candidate C was not selected because `--write` would require D184 to define
safe replacement, failure atomicity, symlink behavior and metadata/permission
handling before any current LM011-C requirement demonstrates that cost.

## Smallest-sufficient-solution result

Candidate B selects:

~~~text
ONE_PUBLIC_COMMAND
ONE_SOURCE
ONE_EXISTING_FORMATTER_AUTHORITY
ONE_SOURCE_OUTPUT_CHANNEL
ZERO_FILESYSTEM_WRITE_MACHINERY
~~~

It intentionally defers:

~~~text
in-place write
multi-file aggregation
directory traversal
workspace/package discovery
check/no-write status semantics
range formatting
on-type formatting
formatter directives
style configuration
generic public Tool namespace
fmt alias
~~~

Each deferred capability can be added later without changing the selected
meaning of the two baseline invocations:

~~~text
protos format
protos format file.protos
~~~

~~~text
PAY_FOR_WHAT_YOU_NEED=PASS
GROW_AS_YOU_NEED=PASS
REVERSIBILITY=PASS
COST_OF_DEFERRAL=LOW_FOR_WRITE_AND_CHECK
SPECULATION_BURDEN=MINIMIZED
AUTOMATION_COMPOSABILITY=PASS
ACCIDENTAL_MUTATION_RISK=MINIMIZED
~~~

## Maintainer-reported validation context

The project owner reports:

~~~text
LOCAL_TESTS=PASS
~~~

D184 ratification itself changes no product code and does not rely on a new test
run for semantic authority. The reported local-test state is retained as
coordination context only.

## Durable publication

Canonical durable decision record:

~~~text
docs/project/decisions/tooling/
D184_CANONICAL_FORMATTER_PUBLIC_CLI_CONTRACT.md
~~~

Decision-record publication commit:

~~~text
d6226007a99e1463d83d1fccde014afb14901c50
~~~

This evidence record:

~~~text
docs/project/evidence/D184/
D184_CANDIDATE_B_RATIFICATION_EVIDENCE.md
~~~

~~~text
EVIDENCE_PUBLICATION_REVISION=SAME_COMMIT
~~~

## LM011-C release

D184 Candidate B makes LM011-C mechanical.

~~~text
D184_STATUS=RATIFIED
LM011_C_RELEASED=YES
LM011_C_STATUS=READY_FOR_IMPLEMENTATION
IMPLEMENTATION_AUTHORIZED=YES
NEXT_SLICE=LM011-C
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

Authorized LM011-C scope is limited to:

1. add public `format` dispatch to the existing `protos` driver;
2. accept zero operands from stdin or one explicit regular-file source;
3. reject multiple operands and unsupported file kinds according to D184;
4. construct/use the existing editor-neutral whole-document formatter input;
5. invoke `ProtosWholeDocumentFormatter`, without reimplementing formatting;
6. translate formatter result into the selected stdout/stderr/exit contract;
7. preserve genuine TOOL010/bootstrap/internal failures as internal failures;
8. update public CLI help;
9. add focused CLI contract tests.

Not authorized by D184:

~~~text
--write
--check
multi-file
directory recursion
workspace formatting
package discovery
range formatting
on-type formatting
style configuration
new formatter policy
new parser/grammar
LSP formatting
VS Code integration
LM011-D
LM011-E
~~~

~~~text
OBSERVABLE_PROTOS_LANGUAGE_SEMANTIC_CHANGE_REQUIRED=NO
SPECIFICATION_CHANGE_REQUIRED=NO
~~~
