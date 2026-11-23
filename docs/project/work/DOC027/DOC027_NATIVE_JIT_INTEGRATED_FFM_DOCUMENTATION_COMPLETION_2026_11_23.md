# DOC027 — Native guest JIT and integrated FFM documentation reconciliation

**Date:** 2026-11-23  
**Formal owner:** [DOC027 #1157](https://github.com/guillermomolina/protos/issues/1157)  
**PROTOS_REVISION:** `89028494587d4d22309a5c048e37da3518170b18`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18  
**Revision-bound evidence:** [DOC027 publication and validation](../../evidence/DOC027/DOC027_NATIVE_JIT_FFM_PUBLICATION_AND_VALIDATION_2026_11_23.md)

## Purpose and result

DOC027 reconciles public documentation with the already-published Native runtime acceptance and native-library interop implementation. The former interpreter-only/fallback description was historical, not a current limitation. The published Native acceptance gate requires successful guest Truffle Tier-2 compilation for the Protos Bytecode DSL root and builtin-call root families. This document does not establish a new JIT or performance guarantee.

The installed standalone Native executable already integrates the generic `ffi:` provider, without an external Java provider JAR, whereas the portable JVM continues to load the optional external provider JAR using `--foreign-provider-path`. Native host library authorization is explicit and default-off (`--allow-library alias=target`), acquisition is lazy on `ffi:<alias>` import, and only the finite registered physical C signature set is supported. The authorized C library runs trusted in process, without sandbox confinement by Protos guest capabilities. Public application code continues to use the Protos-first `std:ffi` and `std:sqlite` modules.

## Published scope

Exact GitHub commit https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18 changes **14 paths**: **13 Markdown documentation files** across the root README, installation, CLI, Standard Library, interop, provider, example and tutorial guides, plus **one restored README logo asset**.

The image `docs/assets/branding/protos-logo.png` was restored at the referenced path using the original blob `48862ed6638ec851349211b28b25aec4bb45f0a6`. The missing-image regression had been included in [the DOC027 issue comments](https://github.com/guillermomolina/protos/issues/1157#issuecomment-7097702045); this is a source-repository README asset, separate from website work.

No product Java implementation, executable Protos source, normative specification, Maven version, runtime machinery, or website files were changed in this revision.

## Published documentation anchors

- [Native acceptance gate in the README](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/README.md) and [Native installation limitations](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/00-try-protos.md).
- [CLI Native/JVM comparison](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/tools/cli.md) and [Standard Library availability](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/library/README.md).
- [Foreign interoperability availability](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/interop/README.md), [public FFI](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/library/ffi.md), and [canonical Native FFM provider section](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/interop/ffm-provider.md#native-image-integrated-provider).
- [SQLite usage](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/library/sqlite.md), [FFM examples](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/protos/examples/ffi/README.md), and [tutorial invocation examples](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/protos/tutorials/README.md).
- [Executable Native guest Tier-2 gate](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/scripts/native-test.sh) and [distribution JIT acceptance validator](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/dist/validate_native.py), unchanged by DOC027.

## Validation provenance and limitations

The maintainer reports `git diff --check` clean and all local tests passing after publication. Those are **owner-reported local outcomes**, not agent-executed tests or an independently verified per-command test transcript.

GitHub inspection independently confirmed the published SHA and exact 14-file changed-path inventory. A targeted review of the 13 published Markdown contents found no remaining occurrences of the specific obsolete interpreter-only/Native-unavailable assertions checked; both FFM workflows and Native ABI limitations are represented in the revised guides. GitHub also confirmed that the README references an existing logo file with its exact original blob identity. This proves the source image link target exists, not an independent visual browser-render test.

A comprehensive Markdown parser pass, full link/anchor crawl, CI execution, rebuilt Native binary and independent test rerun are **not claimed**. Historical CHANGELOG entries and the separately owned WEB010 website were untouched. The live Issue remains the authority for closure.

```text
FORMAL_IDENTIFIER=DOC027
PROTOS_REVISION=89028494587d4d22309a5c048e37da3518170b18
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
PRODUCT_PATHS_CHANGED=14
DOCUMENTATION_MARKDOWN_PATHS_CHANGED=13
README_LOGO_BLOB=48862ed6638ec851349211b28b25aec4bb45f0a6
README_IMAGE_SOURCE_TARGET=PASS_GITHUB_VERIFIED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
PUBLISHED_DOC_STALE_ASSERTION_REVIEW=PASS_TARGETED
FULL_LINK_AND_ANCHOR_CRAWL=NOT_CLAIMED
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
SPEC_OR_IMPLEMENTATION_CHANGE=NONE
```
