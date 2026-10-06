# AUD005 / LIB010-E2-D — final corrective closure evidence

## Status

```text
WORK_ITEM=AUD005
ISSUE=guillermomolina/protos#451
TARGET=LIB010
TARGET_ISSUE=guillermomolina/protos#418
SLICE=LIB010-E2-D
SLICE_TYPE=IMPLEMENTATION
STATUS=COMPLETE
PROTOS_REVISION=6a9ca47f6305f77a01632aca84828a1574d63731
PROTOS_REVISION_SUBJECT=LIB010-E2-D: close AUD005 corrective debt
IMPLEMENTATION_VERSION=0.3.234-SNAPSHOT
LOCAL_GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
FINAL_INTEGRATED_MAKE_TEST=PASS
VALIDATION_PROVENANCE=MAINTAINER_REPORTED_PLUS_COMMITTED_CHANGELOG
PUBLIC_API_CHANGED=NO
TOML_SEMANTICS_CHANGED=NO
SPECIFICATION_CHANGED=NO
D104_CHANGED=NO
D109_CHANGED=NO
D087_CHANGED=NO
TOOL001_CHANGED=NO
TOOL002_CHANGED=NO
F1=RESOLVED
F2=RESOLVED
F3=RESOLVED
F4=EXPLICITLY_DEFERRED_NO_BEHAVIOR_CHANGE
F5=RESOLVED
F6=RESOLVED
F7=READY_FOR_LIVE_GITHUB_RECONCILIATION
NO_UNRESOLVED_CORRECTIVE_DEBT=YES
AUD005_CLOSURE_READY=YES
LIB010_CLOSURE_READY=YES
NEXT_SLICE=NONE
```

This is the final revision-bound durable implementation/validation record that
releases the live GitHub closure transaction for AUD005 and LIB010. It does not
create or alter normative Protos semantics.

## Exact published product

```text
6a9ca47f6305f77a01632aca84828a1574d63731
LIB010-E2-D: close AUD005 corrective debt
0.3.234-SNAPSHOT
```

The exact revision is published on `guillermomolina/protos` `main`. The commit
changes only:

```text
CHANGELOG.md
pom.xml
protos/lib/toml/TOML.protos
scripts/test_validation_impact.py
src/test/java/com/guillermomolina/protos/documentation/ProtosStandardLibraryDocumentationExtractorTest.java
```

The publication itself records that focal gates and the integrated `make test`
closure gate passed before `pom.xml` / `CHANGELOG.md` metadata finalization.
The maintainer additionally reports a clean `git diff --check` and all local
tests passing after publication.

## Final finding disposition

### F1 — forbidden control characters in comments

```text
F1=RESOLVED
```

Closed by LIB010-E1 and retained by its conformance regressions.

### F2 — quote-run host recursion

```text
F2=RESOLVED
```

Closed by LIB010-E1; input-proportional quote-run host recursion was removed.

### F3 — official TOML 1.1 conformance evidence

```text
F3=RESOLVED
```

LIB010-E2-C retains the pinned official `toml-test` v2.2.0 TOML 1.1 corpus and
its deterministic suite-native projection. The retained inventory is 895
logical cases: 884 executable and 11 explicitly outside the public String-input
byte domain.

Durable E2-C evidence remains:

```text
docs/project/work/AUD005/AUD005_E2_C_OFFICIAL_TOML_11_CONFORMANCE_EVIDENCE.md
```

### F4 — composite constructor validation depth

```text
F4=EXPLICITLY_DEFERRED_NO_BEHAVIOR_CHANGE
DEEP_CONSTRUCTOR_VALIDATION_CHANGE=NO
NEW_DXXX=NO
```

The project-owner-directed final closure explicitly defers any strengthening of
`TOML.array` / `TOML.table` to recursively validate arbitrarily forged
descendants.

E2-D changes no constructor behavior. It adds authored public documentation that
states the current immediate-envelope contract without promising compatibility
for invalid forged trees. `TOML.encode` continues to perform the complete
encodability validation required by encoding.

A future independently approved decision may strengthen constructor validation.
This closure therefore does not freeze accidental acceptance of invalid forged
trees as a public compatibility promise.

The two documentation gaps routed from DOC008 are closed. The retained Standard
Library documentation test now requires:

```text
missingModuleDocumentation().isEmpty()
missingSymbolDocumentation().isEmpty()
documentedModuleCount() == moduleCount()
documentedSymbolCount() == symbolCount()
```

### F5 — LIB010-D V9 reusable-evidence path gate

```text
F5=RESOLVED
V9_HISTORICAL_GATE_PROOF=INVALID_AND_RECONCILED
V9_HISTORICAL_INTERVAL=BENIGN
OLD_STDIN_HEREDOC_MECHANISM=RETIRED
CURRENT_GATE=BASE_HEAD_DIRECT_GIT
STDIN_DEPENDENCY_REGRESSION=RETAINED
```

The historical V9 proof attempted to send both the Python program and changed
paths through stdin by combining `python3 -` with a heredoc/pipeline. That
specific proof was invalid. The previously completed retrospective interval
review remains benign; LIB010 publication was not corrupted.

The canonical current selector does not reuse that mechanism.
`scripts/validation_impact.py` obtains its definitive delta directly from Git
using explicit `--base` / `--head` revisions and `git diff --name-status -z`.

E2-D retains `test_cli_derives_delta_from_git_base_head_not_stdin`: the CLI is
fed irrelevant non-empty stdin containing paths that would otherwise force FULL,
while the actual Git delta remains Test Tool-local. The expected
`TOOL_LOCAL:TEST` / `SKIP_ALLOWED` result proves delta acquisition is independent
of stdin.

### F6 — temporal fraction scaling

```text
F6=RESOLVED
```

Closed by LIB010-E2-A with linear emitted-length construction and retained
400-digit/source-structure evidence.

### F7 — GitHub/durable closure mismatch

The product corrective work and final integrated validation are now complete.
This publication deliberately precedes the live Issue state transition so that
the closure transaction is fail-closed and revision-bound.

The maintained current pointers published alongside this file supersede the
transitional AUD005 status preamble embedded in the earlier
`LIB010_TOML_DESIGN.md` publication. That large design record is preserved as
ratified design plus historical lifecycle evidence rather than mechanically
rewritten.

After this durable publication succeeds, the live transaction is authorized to:

```text
close guillermomolina/protos#451 as completed
close guillermomolina/protos#418 as completed
```

A post-closure reconciliation record will bind those live states back to this
exact durable publication and mark F7 resolved.

## Corrective slice chain

```text
LIB010-E1   F1 + F2                           RESOLVED
LIB010-E2-A F6                                RESOLVED
LIB010-E2-B F3 research                       COMPLETE
LIB010-E2-C F3 implementation/conformance     RESOLVED
LIB010-E2-D F4 + F5 + final validation        COMPLETE
```

No additional implementation/research slice is required.

## Final closure invariant

```text
NO_UNRESOLVED_CORRECTIVE_DEBT=YES
AUD005_CLOSURE_READY=YES
LIB010_CLOSURE_READY=YES
NEXT_SLICE=NONE
```
