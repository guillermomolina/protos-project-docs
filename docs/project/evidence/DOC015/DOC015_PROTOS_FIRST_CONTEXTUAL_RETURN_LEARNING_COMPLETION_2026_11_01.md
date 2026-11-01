# DOC015 — Protos-first contextual return tutorials, example and guides: completion evidence

**Date:** 2026-11-01. **Owning formal work:** [DOC015 #1025](https://github.com/guillermomolina/protos/issues/1025). **Category:** completed documentation/tutorial/example publication, not a normative language revision or new runtime implementation.

This record distinguishes **GitHub-fetched publication facts**, **inspection of published source and README links**, and **the human maintainer's reported validations**. The assistant did not execute the Protos CLI, Maven tests, `make test`, `make check`, or a local Git diff.

## Exact identity and publication lineage

- **Product repository:** `guillermomolina/protos` (two occurrences of `mo`).
- **Published product revision:** [`PROTOS_REVISION=8af1a70a56ca113a94eea1a5935a74a80fecdfd2`](https://github.com/guillermomolina/protos/commit/8af1a70a56ca113a94eea1a5935a74a80fecdfd2).
- **Commit subject:** `DOC015: add Protos-first contextual return tutorial, example and guide links`.
- **Published date:** 2026-11-01, as recorded in the GitHub commit.
- **Commit parent:** `a7b1a66dc6bb73c97ec0d52bb603bd9836600737` (prior I139 product publication). The head `pom.xml` contains `0.3.433-SNAPSHOT`; DOC015 does **not** change this version.
- **Commit delta:** exactly 7 product paths (2 new Protos source programs and 5 maintained Markdown files), 216 additions, zero deletions.
- **Normative basis:** previously ratified [D234-B2-S1](../../decisions/language/D234_RETURN_CONTEXTUAL_SUGAR_B2_S1.md), [I138 #1021](https://github.com/guillermomolina/protos/issues/1021) published at [`ab637aa264ddbb2b99fdb03cb19207cc19651f45`](https://github.com/guillermomolina/protos/commit/ab637aa264ddbb2b99fdb03cb19207cc19651f45), which introduced `return expression` as exact `^expression` alias under specification revision `0.1.487`. The I138 [revision-bound evidence](../I138/I138_0_3_432_CONTEXTUAL_RETURN_PUBLICATION_AND_VALIDATION_2026_11_01.md) is unchanged.

No `spec/`, `src/`, `protos/lib/`, `pom.xml`, `CHANGELOG.md`, test harness or runtime file is part of the DOC015 commit.

## Verified published deliverables

| Role | Exact file | Published behavior / change |
| --- | --- | --- |
| Progressive tutorial | [protos/tutorials/03-closures/03-return-home.protos](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/protos/tutorials/03-closures/03-return-home.protos) | New CLI-runnable source contrasts final-expression results, `^item`, `return item`, early exit, observed visit counter, fallthrough; APL Part 5 notice is present; eight `print` calls |
| Task-oriented example | [protos/examples/closures/non-local-return.protos](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/protos/examples/closures/non-local-return.protos) | New CLI-runnable first-match/early-exit example, old caret equivalent, `(return)(21)` ordinary binding escape, no-match case; APL Part 5 notice is present; five `print` calls |
| Tutorial navigation | [protos/tutorials/README.md](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/protos/tutorials/README.md) | Links new third Closure lesson in existing sequence |
| Cookbook navigation | [protos/examples/README.md](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/protos/examples/README.md) | Adds return-from-Array-callback task and distinguishes existing `std:collections/Array.findIndex` |
| Callable guide | [docs/guide/03-closures-methods-and-receivers.md](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/docs/guide/03-closures-methods-and-receivers.md) | Adds explicit normal completion versus early return, return-home lifetime, `InvalidReturn`, grouping, lexical boundaries and runnable links |
| Control-flow guide | [docs/guide/04-control-flow-through-protocols.md](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/docs/guide/04-control-flow-through-protocols.md) | Shows `each` + contextual `if` + early `return`, JavaScript/GraalJS callback distinction and `ensure` semantics, with runnable links |
| Guide navigation | [docs/guide/README.md](https://github.com/guillermomolina/protos/blob/8af1a70a56ca113a94eea1a5935a74a80fecdfd2/docs/guide/README.md) | Adds contextual-return route beside existing tutorial/example guidance |

Published source files and Markdown link targets were fetched at the exact commit, including the unchanged APL Part 5 notices. The product code uses the existing Protos Closure, Array `each`, contextual `if`, captured return-home and ordinary `print` facilities; no new Java or external language library is introduced.

## Validation evidence, exact attribution

The maintainer reported after publication (2026-11-01):

> `DOC015: add Protos-first contextual return tutorial, example and guide links pushed.`  
> `el git diff check esta limpio.`  
> `make test y make check pasan en local.`

| Gate | Result and attribution |
| --- | --- |
| Product commit/push, exact product SHA, 7-file inventory | **VERIFIED** by GitHub repository/commit read |
| Published tutorial and cookbook source, APL notices and guide links | **VERIFIED** by immutable source reads |
| `git diff --check` | **PASS — maintainer reported** |
| `make test` | **PASS — maintainer reported** |
| `make check` | **PASS — maintainer reported** |
| Standalone CLI discovery for print-dependent learning material | **Source verified:** `ProtosCliLearningMaterialsTest` scans `protos/tutorials` and `protos/examples`; these new files contain `print(`. Integrated test PASS is maintainer-reported |
| Exact stdout comparison to the teaching prompt | **Not separately attested.** Existing CLI test requires exit 0, nonempty stdout and blank stderr, but does not assert exact lines; no independent CLI execution is claimed |
| Native Image validation | **Not separately attested** |

The original teaching prompt specified expected standalone output, not an independently observed result:

- Tutorial: `42`, `3`, `3`, `2`, `2`, `false`, `false`, `true`.
- Example: `3`, `2`, `3`, `true`, `42`.

These expectations are retained for reproducible follow-up checks, not misrepresented as logged execution output.

## Closure boundary and scope

- Product publication and reported integrated validation satisfy the core documentation and executable-program delivery evidence. The Source, tutorials, guide text and closure semantics were not rewritten as a new language specification.
- The independently observed GitHub product artifact provides durable identity; this separate `protos-project-docs` record fulfills DOC015's explicit revision-bound evidence commitment. The exact `PROJECT_RECORD_REVISION` is recorded in DOC015's live GitHub closure note after this evidence is published and reread.
- DOC015 does not claim completion of [TEST011 #1023](https://github.com/guillermomolina/protos/issues/1023) or [BUG032 #1024](https://github.com/guillermomolina/protos/issues/1024), which concern independently tracked pre-existing test/formatter issues. No new feature, normative authority or further product work is implied.
