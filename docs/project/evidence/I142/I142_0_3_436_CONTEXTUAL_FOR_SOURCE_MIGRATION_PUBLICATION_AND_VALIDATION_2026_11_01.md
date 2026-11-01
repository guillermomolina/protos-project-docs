# I142 — Contextual `for` source migration, publication and validation (0.3.436)

**Date:** 2026-11-01. **Formal owner:** [I142 #1033](https://github.com/guillermomolina/protos/issues/1033), implementation/source migration. **Language authority:** previously ratified [D231-F2-PURE #1013](https://github.com/guillermomolina/protos/issues/1013), normative `spec/PROTOS_GRAMMAR.md` §36. **Record role:** revision-bound, non-normative product-publication evidence.

## Exact published artifact

- **PRODUCT_REPOSITORY:** `guillermomolina/protos`
- **PRODUCT_REVISION:** [`7879eeedd6dc07d87bd9c459d2fd762d0524eb3d`](https://github.com/guillermomolina/protos/commit/7879eeedd6dc07d87bd9c459d2fd762d0524eb3d)
- **PARENT_REVISION:** `fadb184409fc9b985e19db4a75ca76461592ff6b` (previously completed [DOC016 #1032](https://github.com/guillermomolina/protos/issues/1032))
- **COMMIT_TITLE:** `I142: migrate eligible Library, Tool and test traversals to contextual for (0.3.436)`
- **PRODUCT_VERSION:** `0.3.436-SNAPSHOT`, verified in the committed `pom.xml` and `CHANGELOG.md`
- **NORMATIVE_SPEC_REVISION:** `0.1.490`, unchanged by this migration; the last language change was I141
- **GITHUB_COMMIT_DELTA:** 114 changed files, 1,088 additions, 1,078 deletions; independently read from GitHub's published commit
- **FILE_GROUPS:** 34 `protos/lib/**`, 35 `protos/tools/**`, 43 `protos/tests/library/**`, plus `pom.xml` and `CHANGELOG.md`
- **MIGRATION_COUNT:** 529 eligible generic `for` rewrites, specifically 159 Library + 217 Tool + 153 Library-test sites. These figures are in the published changelog and match the editing-agent's dry-run/applied-edit reports; they were not separately re-counted by the coordinator with a source AST tool.

## Scope and Protos-first semantic boundary

I142 replaces ordinary single-plain-parameter fresh braced Closure calls such as `items.each((item) => { ... })` with `for(item in items) { ... }`. The new contextual spelling is an expression and canonically lowers to the same ordinary receiver-selected `each` message and Closure. No collection protocol, runtime dispatch, parser, Range behavior, algorithm or evaluation policy changes in this commit.

The migration is intentionally **generic-only**: no numeric `Range(start,stop).each/eachStep` source rewrite was required in its owned files. Existing explicit two-argument Map/IdentityMap callbacks, `Set`/ `IdentitySet` key-marker traversal, stored callbacks, the Core `Range` implementation, protocol/conformance tests, fixed benchmarks, generated/embedded fixtures, and unrelated source remain deliberately canonical. No `spec/**`, Java parser/runtime, `docs/guide/**`, `protos/tutorials/**` or `protos/examples/**` files are changed by I142. Thus DOC016's educational content is not overwritten. Existing APL notices are retained; no new executable source file is introduced.

The previously shipped [I141 #1030](https://github.com/guillermomolina/protos/issues/1030) provides three exact contextual spellings, `for(item in iterable)`, `for(i, start, stop)`, and `for(i, start, stop, step)`, all lowered to ordinary `each` or `eachStep` sends. I142 is **adoption**, not language implementation.

## Validation and provenance

**Maintainer explicitly attested, after pushing the I142 product commit:**
- `git diff --check`: **CLEAN**, local, reported.
- `make test`: **PASS**, local, reported.
- `make check`: **PASS**, local, reported.

**Coordinator independently checked via GitHub:** live `main` pointed to the product commit above at documentation preparation; exact commit title, parent, counts and changed-path category breakdown; next-patch Maven version and matching changelog text. Coordinator did not execute the developer's local commands or independently validate all 529 transformed syntax trees, formatter round-trips, Native Image or benchmark/performance outcomes. No separately scoped focal tests or `git diff --cached --check` run are claimed.

## Related published work and editorial boundary

- [I136 #1015](https://github.com/guillermomolina/protos/issues/1015), product `dd4db2e9722df293aac0f3674aa2d88b46841aff` / 0.3.430: first-class ordinary directed `Range`/`InvertedRange` prototypes, `eachStep`, `invert` and `reverse`.
- [I139 #1022](https://github.com/guillermomolina/protos/issues/1022), product `a7b1a66dc6bb73c97ec0d52bb603bd9836600737` / 0.3.433: canonical frozen Prelude Range prototypes and identity-preserving standard module bridge.
- [I140 #1029](https://github.com/guillermomolina/protos/issues/1029), product `2108e5ab2eacd1ebd0f64041b49d7245a414481e` / 0.3.434: direction-selecting ordinary constructor `call`.
- [I141 #1030](https://github.com/guillermomolina/protos/issues/1030), product `1c774003a2c697b7c2adae9967b562872b8d652a` / 0.3.435: all three contextual `for` forms, grammar/tooling/tests.
- [DOC016 #1032](https://github.com/guillermomolina/protos/issues/1032), product `fadb184409fc9b985e19db4a75ca76461592ff6b`: independent tutorial, example and guide material.

See [I142 completed work record](../../work/I142/I142_CONTEXTUAL_FOR_SOURCE_MIGRATION_COMPLETION_2026_11_01.md). Formal issue closure requires this record's committed Git SHA and re-read live GitHub status. Priority, assignee, milestone and native hierarchy are not inferred.

```text
WORK_ITEM=I142
PRODUCT_PUBLICATION=VERIFIED
PROTOS_REVISION=7879eeedd6dc07d87bd9c459d2fd762d0524eb3d
PARENT_REVISION=fadb184409fc9b985e19db4a75ca76461592ff6b
PRODUCT_VERSION=0.3.436-SNAPSHOT
SPEC_REVISION=0.1.490_UNCHANGED
CHANGED_PATHS=114
PROTOS_SOURCE_PATHS=112
MIGRATION_FORMS=529_REPORTED_AND_CHANGELOG_RECORDED
HUMAN_GIT_DIFF_CHECK=CLEAN_REPORTED
HUMAN_MAKE_TEST=PASS_REPORTED
HUMAN_MAKE_CHECK=PASS_REPORTED
NATIVE_CHECK=NOT_ATTESTED
```
