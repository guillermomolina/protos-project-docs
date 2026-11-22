# DOC026 — Revision-bound publication and validation evidence

**Date:** 2026-11-22  
**Issue:** [DOC026 #1144](https://github.com/guillermomolina/protos/issues/1144)  
**PROTOS_REVISION:** `b0bbcae2e69b888cbb3d64c471791a0251ef3b4b`  
**Product commit:** https://github.com/guillermomolina/protos/commit/b0bbcae2e69b888cbb3d64c471791a0251ef3b4b  
**Durable work record:** [DOC026 practical guide completion](../../work/DOC026/DOC026_PRACTICAL_GUIDE_COMPLETION_2026_11_22.md)

## Independent publication inspection

GitHub commit `b0bbcae2e69b888cbb3d64c471791a0251ef3b4b` is titled `DOC026: Rework the programming guide around practical learning`. The changed-file inventory has 21 entries:

```text
CHANGELOG.md
README.md
docs/guide/01-bindings-contexts-and-state.md
docs/guide/02-objects-delegation-and-composition.md
docs/guide/03-closures-methods-and-receivers.md
docs/guide/04-control-flow-through-protocols.md
docs/guide/05-values-identity-equality-and-collections.md
docs/guide/06-modules-and-imports.md
docs/guide/07-errors-handlers-ensure-and-resource-lifetime.md
docs/guide/08-futures-and-structured-concurrency.md
docs/guide/09-isolated-parallel-execution.md
docs/guide/10-actors-actorrefs-and-groups.md
docs/guide/11-process-io-filesystems-and-authority.md
docs/guide/12-matching-expressions.md
docs/guide/README.md
docs/guide/SOURCE_STYLE.md
docs/guide/first-program.md
docs/guide/interop/README.md
docs/guide/library/sqlite.md
docs/guide/tools/README.md
docs/guide/tools/test-tool.md
```

The published delta introduces the first-program lesson and practical entry/navigation additions, while reducing work-tracking prose in the learner-facing chapters. The first-program material is educational documentation, not an additional normative authority.

## Human-reported verification

| Gate | Provenance / result |
| --- | --- |
| Product commit and 21 changed paths | GitHub **verified** |
| `git diff --check` | Maintainer reports **clean** |
| All local tests | Maintainer reports **PASS**; command-level inventory not supplied |
| Five first-program snippets using `bin/protos -e` | Maintainer reports **PASS**, exit 0, exact expected stdout for each |
| New local Markdown link targets | Maintainer reports **PASS** for path existence; not an anchor/remote-link check |
| Working-copy source-style occurrence comparison | Maintainer reports **PASS** using existing `candidate_counts` guard |
| Commit-to-commit source-style publication guard | Maintainer reports **PASS**: 20 scoped files; one existing exception; zero allowed increases |
| Independent rerun, CI, Native rebuild | **Not claimed** |
| Anchor and external link verification | **Not claimed** |

The maintainer's DOC026 Issue comments record the staged validation, publication, and the final revision SHA. Those reports are not reinterpreted as executions performed by this agent.

```text
FORMAL_IDENTIFIER=DOC026
PROTOS_REVISION=b0bbcae2e69b888cbb3d64c471791a0251ef3b4b
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
PRODUCT_PATHS_CHANGED=21
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
FIRST_PROGRAM_EXECUTION=5_OF_5_PASS_OWNER_REPORTED
LOCAL_LINK_TARGETS=PASS_OWNER_REPORTED
SOURCE_STYLE_COMMIT_GUARD=PASS_OWNER_REPORTED
PROJECT_RECORD_PUBLISHED=THIS_RECORD
```
