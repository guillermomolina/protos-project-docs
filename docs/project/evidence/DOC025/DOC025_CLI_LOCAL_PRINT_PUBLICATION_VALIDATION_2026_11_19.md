# DOC025 — Revision-bound publication and validation evidence

**Date:** 2026-11-19  
**Owning issue:** [DOC025 #1123](https://github.com/guillermomolina/protos/issues/1123)  
**PROTOS_REVISION:** `e834a05a25844e006c77c3be06fa53474a5142a2`  
**Exact published commit:** https://github.com/guillermomolina/protos/commit/e834a05a25844e006c77c3be06fa53474a5142a2  
**Maintained DOC025 record:** [Work completion and acceptance boundary](../../work/DOC025/DOC025_CLI_LOCAL_PRINT_AND_EXPLICIT_OUTPUT_COMPLETION_2026_11_19.md)

## Independently inspected GitHub publication

The commit subject is `DOC025: Clarify CLI-local print and explicit output injection`. At `0.4.177-SNAPSHOT` it changed exactly ten product paths:

```text
README.md
docs/guide/00-try-protos.md
docs/guide/01-bindings-contexts-and-state.md
docs/guide/06-modules-and-imports.md
docs/guide/tools/cli.md
protos/examples/README.md
protos/examples/modules/direct-file-import/helper.protos
protos/examples/modules/direct-file-import/main.protos
spec/runtime/ABSTRACT_RUNTIME.md
spec/semantics/MODULES.md
```

The public explanation states that CLI-local `print` belongs to the initial entry context, not Core/Prelude and not an imported SOURCE/MODULE initializer. The informative runtime's incorrect `prelude.print` example is replaced by `Object` lookup/shadowing. An existing two-file direct-import example is extended with explicit `Helper.greet(print)` injection and linked from the examples index. The SOURCE/MODULE explanation is clarified without changing runtime production Java or introducing implicit output authorization.

## Validation provenance

| Gate | Result and source |
| --- | --- |
| Exact product commit and ten changed paths | **VERIFIED** from GitHub commit |
| CLI/import and binding-guide corrections | **VERIFIED** in published diff |
| Informative `prelude.print` error removed | **VERIFIED** in published diff |
| Runnable two-file source and index | **VERIFIED AS PUBLISHED SOURCE**; not separately executed by this agent |
| `git diff --check` in the Protos checkout | **CLEAN**, explicitly reported by maintainer |
| All local Protos tests | **PASS**, explicitly reported by maintainer; separate commands/counts not given |
| Example produces three lines in a maintainer-executed CLI run | **NOT REPORTED** as a distinct run |
| Original case fails at imported SOURCE's bare `print` lookup | **DOCUMENTED**, but fresh maintainer-executed reproduction **NOT REPORTED** |
| CI, rebuilt Native, installed Native, independent rerun | **NOT CLAIMED** |
| `git diff --check` on the documentation repository | **NOT RUN AS GIT COMMAND** through GitHub connector publication; added-line whitespace can be checked against the published commit |

The documented expected example output is:

```text
Hello
Hello from a local module
Main, Protos!
```

These are documented expectations, **not** an invented observation from a supplied execution transcript. The maintainer's all-tests-PASS attestation is recorded as such; the separately worded manual example and original negative-case acceptance remains open until explicitly evidenced.

```text
FORMAL_IDENTIFIER=DOC025
PROTOS_REVISION=e834a05a25844e006c77c3be06fa53474a5142a2
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
PRODUCT_PATHS_CHANGED=10
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
EXAMPLE_SOURCE_AND_INDEX=PASS_GITHUB_VERIFIED
THREE_LINE_CLI_EXECUTION=NOT_REPORTED
ORIGINAL_NEGATIVE_CASE_REEXECUTION=NOT_REPORTED
INDEPENDENT_CI_OR_BINARY_TESTS=NOT_CLAIMED
```
