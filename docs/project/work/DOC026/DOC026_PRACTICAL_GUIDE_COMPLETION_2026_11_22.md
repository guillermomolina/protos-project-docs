# DOC026 — Practical programming guide completion

**Date:** 2026-11-22  
**Formal owner:** [DOC026 #1144](https://github.com/guillermomolina/protos/issues/1144)  
**PROTOS_REVISION:** `b0bbcae2e69b888cbb3d64c471791a0251ef3b4b`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/b0bbcae2e69b888cbb3d64c471791a0251ef3b4b  
**Revision-bound evidence:** [DOC026 publication and validation](../../evidence/DOC026/DOC026_PRACTICAL_PROGRAMMING_GUIDE_PUBLICATION_2026_11_22.md)

## Purpose and result

DOC026 turns the existing programming guide into a practical learning path without replacing the normative language specification. A new [first-program walkthrough](https://github.com/guillermomolina/protos/blob/b0bbcae2e69b888cbb3d64c471791a0251ef3b4b/docs/guide/first-program.md) introduces five successive runnable steps with expected output and exercises. The [guide index](https://github.com/guillermomolina/protos/blob/b0bbcae2e69b888cbb3d64c471791a0251ef3b4b/docs/guide/README.md) organizes navigation around goals and common tasks. All twelve existing language chapters gain practical starting points and return links.

The revision removes work-item closure inventories, historical conformance reports and administrative status prose from learner-facing explanations, retaining useful runtime/availability limitations and specification references. Existing chapter paths remain intact. The root README and changelog point readers to the revised learning path. The SQLite guide retains its pending-ratification boundary; the change does not approve new semantics.

## Published scope

The verified Protos publication changes **21 documentation files**: `README.md`, `CHANGELOG.md`, the twelve `docs/guide/01-...` through `12-...` chapters, and seven additional guide files: `docs/guide/README.md`, `docs/guide/SOURCE_STYLE.md`, `docs/guide/first-program.md`, `docs/guide/interop/README.md`, `docs/guide/library/sqlite.md`, `docs/guide/tools/README.md`, and `docs/guide/tools/test-tool.md`.

No production Java, executable source, normative specification or product version change is included.

## Validation provenance and limits

The maintainer reported a clean `git diff --check` and all local tests passing. Earlier DOC026 issue evidence records maintainer-executed `bin/protos -e` checks of all five new first-program steps: exit code zero and stdout exactly matching the documented expectations. The maintainer also reported existing local link targets present, a working-copy source-style check passing, and the post-publication commit-to-commit source-style guard passing with **20 scoped changed files, one existing exception and zero allowed increases**.

These are owner-reported execution results, not independent agent runs. GitHub independently identifies the exact published product commit and its changed paths. The existing local-target check did not include anchor or remote-link verification; no CI, Native build or independent full-suite rerun is claimed.

**Disposition:** Product documentation published and the DOC026 durable record supplied. The live Issue is the authority for final completed status; this record is non-normative.

```text
FORMAL_IDENTIFIER=DOC026
PROTOS_REVISION=b0bbcae2e69b888cbb3d64c471791a0251ef3b4b
PRODUCT_PATHS_CHANGED=21
PRODUCT_PUBLICATION=PASS_GITHUB_VERIFIED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
FIRST_PROGRAM_STEPS=5_EXACT_STDOUT_OWNER_REPORTED
SOURCE_STYLE_PUBLICATION_GUARD=PASS_OWNER_REPORTED
REMOTE_AND_ANCHOR_LINK_VALIDATION=NOT_CLAIMED
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
```
