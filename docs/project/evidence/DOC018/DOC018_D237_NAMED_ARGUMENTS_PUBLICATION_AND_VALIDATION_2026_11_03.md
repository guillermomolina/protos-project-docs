# DOC018 — D237 named arguments: immutable publication and validation evidence

**Date:** 2026-11-03. **Formal owner:** [DOC018 #1040](https://github.com/guillermomolina/protos/issues/1040). **Evidence kind:** exact product publication identity, published change scope, and attributed maintainer validation (not an additional normative design decision).

## Published product identity

- **PROTOS_REVISION:** [`2c89adf6062f212c4d9e20f68b919e843a05b523`](https://github.com/guillermomolina/protos/commit/2c89adf6062f212c4d9e20f68b919e843a05b523).
- **PARENT_REVISION:** `c2a2b8ccbf51874b89d3c5a3b714d104c08ee414` (I143 implementation prerequisite).
- **COMMIT_SUBJECT:** `DOC018: teach D237 named arguments in guides and learning materials`.
- **GITHUB_DIFF:** exactly six published files, 189 additions, 0 deletions, two newly created `.protos` programs and four amended Markdown documents.
- **PRODUCT_VERSION:** `0.3.440-SNAPSHOT` (unchanged); **SPEC_REVISION:** `0.1.492` (unchanged). No `pom.xml`, `CHANGELOG.md`, `spec/**`, Java implementation, or product test infrastructure appears in this DOC-only commit.

| File | Changes |
| --- | --- |
| [Guide chapter 03](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/docs/guide/03-closures-methods-and-receivers.md) | +74 lines explaining D237 syntax, MIX-C binding, grouping, defaults, rest/spread, name-association errors, forwarding and interop limits |
| [Guide index](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/docs/guide/README.md) | +10 lines linking the guide, tutorial and cookbook |
| [Tutorial index](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/tutorials/README.md) | +9 lines documenting the `07-call-arguments` progression |
| [Cookbook index](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/examples/README.md) | +1 navigation row |
| [New named-arguments tutorial](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/tutorials/07-call-arguments/02-named-arguments.protos) | +54 lines, APL-licensed, standalone CLI, eleven Boolean prints |
| [New incident-triage cookbook](https://github.com/guillermomolina/protos/blob/2c89adf6062f212c4d9e20f68b919e843a05b523/protos/examples/closures/named-arguments.protos) | +41 lines, APL-licensed, standalone CLI, six Boolean prints |

## Evidence chronology and its limits

1. **Batch 1:** The maintainer created only the two new Protos educational sources. Reported command output showed successful standalone execution of both using `bin/protos` with **eleven** `true` lines for the tutorial and **six** `true` lines for the incident-triage example, with no reported errors; both paths initially remained untracked as expected.
2. **Batch 2:** The maintainer ran the supplied Python-only Markdown patch from a ZIP on the host, with preflight and application output confirming the four intended files and the exact per-file added-line counts. The maintainer supplied the actual `git diff` for all four Markdown changes and a clean `git diff --check`. Review found no unexpected deletion or unrelated edit.
3. **Final integrated suite:** After the substantive six-file candidate was complete, the maintainer reported **`make test` PASS** and **`make check` PASS**. These tests were not rerun by the publishing coordinator. No version/changelog finalization was necessary for this documentation-only change.
4. **Publication:** The maintainer ran `git commit -m "DOC018: teach D237 named arguments in guides and learning materials"` and `git push origin main`. The supplied terminal output confirmed the successful push from `c2a2b8cc` to `2c89adf6`, printed the full new commit SHA, and returned a clean `git status --short`.
5. **Independent GitHub readback:** The coordinator subsequently confirmed the exact published commit, its parent, its six changed paths and aggregate `+189/-0` statistics, and confirmed `main` points to that commit at evidence-creation time.

**Not separately attested:** exact output-byte snapshot beyond the 17 reported `true` values; the proposed extra static changed-path/link/header gate (no execution transcript supplied); a Native Image-specific test; independent replay of the local full suite; GitHub Project-field synchronization. These unobserved checks are not described as PASS.

## Authority and closure provenance

- [D237 #1038](https://github.com/guillermomolina/protos/issues/1038) and the immutable [ratified design record](../../decisions/language/D237_NAMED_ARGUMENTS_CONTEXTUAL_MIX_C_R3_RATIFIED.md) define the approved intent.
- [I143 #1039](https://github.com/guillermomolina/protos/issues/1039) delivered the source grammar/runtime and normative specification prior to DOC018.
- [DOC018 maintained work record](../../work/DOC018/DOC018_D237_NAMED_ARGUMENTS_LEARNING_COMPLETION_2026_11_03.md) describes the published instructional deliverable; [DOC018 #1040](https://github.com/guillermomolina/protos/issues/1040) is authoritative for live completion.
- A final Issue closure comment must reference **the exact docs-publication revision** after the record and navigation links are published and reread. An Issue comment alone is not a substitute for this record.

```text
WORK_ITEM=DOC018
PROTOS_REVISION=2c89adf6062f212c4d9e20f68b919e843a05b523
PARENT_REVISION=c2a2b8ccbf51874b89d3c5a3b714d104c08ee414
PRODUCT_PATHS=6
ADDITIONS=189
DELETIONS=0
HUMAN_STANDALONE_CLI_CHECKS=17_TRUE_REPORTED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
PRODUCT_PUBLICATION=VERIFIED
PRODUCT_VERSION=0.3.440-SNAPSHOT_UNCHANGED
NORMATIVE_SPEC=0.1.492_UNCHANGED
GITHUB_PROJECT_ROUTING=NOT_INDEPENDENTLY_VERIFIED
```
