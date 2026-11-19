# DOC025 — CLI-local `print` and explicit output injection

**Date:** 2026-11-19  
**Formal owner:** [DOC025 #1123](https://github.com/guillermomolina/protos/issues/1123)  
**PROTOS_REVISION:** `e834a05a25844e006c77c3be06fa53474a5142a2`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/e834a05a25844e006c77c3be06fa53474a5142a2  
**Product version:** `0.4.177-SNAPSHOT` (no new release)  
**Revision-bound evidence:** [DOC025 publication and validation](../../evidence/DOC025/DOC025_CLI_LOCAL_PRINT_PUBLICATION_VALIDATION_2026_11_19.md)

## Scope and documented contract

The standalone CLI supplies `print(value)` in the **initial CLI entry context**. It is a host-provided convenience, not a Core/global or standard Prelude binding. A Closure created in the initial entry may capture that slot through ordinary lexical lookup. An imported SOURCE creates its own lexical context; a MODULE initializer similarly does not automatically receive the caller's `print`. Output behavior must be passed explicitly where needed. The DOC025 publication clarifies existing behavior, not a new host grant or changed runtime.

The previous explanatory `prelude.print` example in the informative abstract-runtime document was incorrect. It now uses the genuine `Object` Prelude binding for frozen-slot lookup, write refusal, and explicit shadowing; the nearby `printVersion` illustration accepts an `output` argument. The normative MODULES document now states the distinct context/authority rule.

## Source-backed deliverables

- [README Prelude discussion](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/README.md) and [try Protos](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/docs/guide/00-try-protos.md): initial-entry limit and no implicit availability in imports.
- [Bindings and contexts](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/docs/guide/01-bindings-contexts-and-state.md), [Modules and Imports](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/docs/guide/06-modules-and-imports.md) and [CLI guide](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/docs/guide/tools/cli.md): capture vs independent imported contexts, explicit injection, and failure interpretation.
- [Two-file direct SOURCE entry](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/protos/examples/modules/direct-file-import/main.protos) plus [helper](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/protos/examples/modules/direct-file-import/helper.protos): `Helper.greet(print)`; then `print(Helper.greeting)`; then `print("Main, Protos!")`. Listed in the [examples index](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/protos/examples/README.md).
- [Informative abstract runtime](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/spec/runtime/ABSTRACT_RUNTIME.md) and [normative MODULES clarification](https://github.com/guillermomolina/protos/blob/e834a05a25844e006c77c3be06fa53474a5142a2/spec/semantics/MODULES.md): no false claim that Prelude grants `print`.

The documented expected lines for the example are `Hello`, `Hello from a local module`, and `Main, Protos!`, in that order. Reading `Helper.greeting` by itself does not print; bare `print` inside the helper's initializer without a supplied binding fails name lookup, not relative import resolution.

## Verification and acceptance boundary

GitHub publication and exact ten-path changed-file scope were independently checked. The maintainer explicitly reports **`git diff --check` clean** and **all local tests PASS**, without individual commands or counts. This report is accepted as owner-supplied local-suite evidence and is not mislabeled as an independent rerun.

The issue separately requests a maintainer-executed two-file CLI demonstration with the three observed output lines **and** verification of the original failing initializer. Neither command transcript nor an explicit statement of those two manual results was supplied in the completion report. The source and expected output are published, but executable-example acceptance remains **not independently established**. Do not turn a general tests-PASS report into an invented manual transcript. No CI, Native rebuild, installed-binary execution, or full per-command test inventory is claimed.

**Disposition:** documentation published; live [DOC025 #1123](https://github.com/guillermomolina/protos/issues/1123) remains in review pending its explicit example/reproducer acceptance checkpoint. No changes to runtime Java, import semantics or new implicit output authority.

```text
FORMAL_IDENTIFIER=DOC025
PROTOS_REVISION=e834a05a25844e006c77c3be06fa53474a5142a2
PRODUCT_VERSION=0.4.177-SNAPSHOT
PRODUCT_PATHS_CHANGED=10
PRODUCT_PUBLICATION=PASS_GITHUB_VERIFIED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
EXAMPLE_EXPECTED_OUTPUT=DOCUMENTED_NOT_OBSERVED_IN_SUPPLIED_TRANSCRIPT
ORIGINAL_REPRODUCER_EXECUTION=NOT_REPORTED
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
```
