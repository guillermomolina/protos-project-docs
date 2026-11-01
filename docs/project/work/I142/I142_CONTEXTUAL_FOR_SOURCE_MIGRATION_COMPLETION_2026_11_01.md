# I142 — Protos-first contextual `for` migration: completed

**Owner:** [I142 #1033](https://github.com/guillermomolina/protos/issues/1033). **Date:** 2026-11-01. **Classification:** implementation/source-style migration, not new language design. **Exact published source:** [`guillermomolina/protos@7879eeedd6dc07d87bd9c459d2fd762d0524eb3d`](https://github.com/guillermomolina/protos/commit/7879eeedd6dc07d87bd9c459d2fd762d0524eb3d), version `0.3.436-SNAPSHOT`; parent `fadb184409fc9b985e19db4a75ca76461592ff6b`.

## Published result

One coherent product commit migrated 529 ordinary single-parameter braced `.each` callbacks into `for(binder in receiver) { ... }` across 112 Protos source files: 159 rewrites / 34 Library files, 217 / 35 Tool files and 153 / 43 Library algorithmic test files. `pom.xml` and `CHANGELOG.md` were updated in the **same** 114-file commit (GitHub verified 1,088 additions and 1,078 deletions). The spec remains `0.1.490`.

The migration retains ordinary protocol dispatch, original receiver evaluation, Closure binding/result and Error/control semantics by using the already-ratified D231-F2-PURE desugaring. It changes neither algorithms nor Java/parser/Range implementations. Two-argument Map callbacks, stored callbacks, Core Range methods, conformance tests, fixed benchmarks, unrelated fixtures and DOC016-owned guides/tutorials/examples deliberately retain their prior forms. No numeric-form migration in I142-owned sources was necessary.

## Owner validation and closure

The maintainer reported the exact product commit **pushed**, with local `git diff --check` **clean**, `make test` **PASS** and `make check` **PASS**. The coordinator verified the published GitHub commit, parent, 114-file scope and both version metadata files. The coordinator did not independently run tests, source-AST recounters, formatter idempotence, Native Image or benchmark checks.

The language feature was already implemented by [I141 #1030](https://github.com/guillermomolina/protos/issues/1030); the educational materials were independently completed by [DOC016 #1032](https://github.com/guillermomolina/protos/issues/1032). I142 closes the owned source-migration follow-up without reopening [D231 #1013](https://github.com/guillermomolina/protos/issues/1013).

See the [revision-bound publication and validation evidence](../../evidence/I142/I142_0_3_436_CONTEXTUAL_FOR_SOURCE_MIGRATION_PUBLICATION_AND_VALIDATION_2026_11_01.md). Live GitHub issue closure should reference the exact project-docs commit after publication and readback.
