# DOC024-S1 — Revision-bound SQLite learning-path publication and validation

**Date:** 2026-11-11  
**Owning Issue:** [DOC024 #1116](https://github.com/guillermomolina/protos/issues/1116)  
**PROTOS_REVISION:** `7ae162258f67553a9bbfc02bb859d3ae328c0c0d`  
**Published product commit:** https://github.com/guillermomolina/protos/commit/7ae162258f67553a9bbfc02bb859d3ae328c0c0d  
**Work record:** [DOC024 completion](../../work/DOC024/DOC024_S1_SQLITE_LEARNING_PATH_COMPLETION_2026_11_11.md)  
**Related published implementation:** [LIB029 `90fd4ece800f6d0e007aa9a2c735ee2f7659919a`](https://github.com/guillermomolina/protos/commit/90fd4ece800f6d0e007aa9a2c735ee2f7659919a)

## Product publication independently verified

The exact GitHub commit contains 16 changed paths:

```text
dist/test_external_foreign_providers.py
docs/guide/README.md
docs/guide/interop/README.md
docs/guide/interop/ffm-provider.md
docs/guide/library/README.md
docs/guide/library/ffi.md
docs/guide/library/sqlite.md
protos/examples/README.md
protos/examples/ffi/README.md
protos/examples/ffi/sqlite-copied-values.protos
protos/examples/ffi/sqlite-memory-query.protos
protos/examples/ffi/sqlite-prepared-numeric.protos
protos/tutorials/08-language-interactions/07-ffi-provider-basics.protos
protos/tutorials/12-system-resources/04-ffi-ownership-and-borrows.protos
protos/tutorials/12-system-resources/06-sqlite-query-lifecycle.protos
protos/tutorials/README.md
```

Three new programs were added; the other 13 paths were updated. Published `pom.xml` remains `0.3.462-SNAPSHOT`, Java classfile baseline 25. No `src/`, `protos/lib/`, `spec/` or `pom.xml` edits occurred in DOC024. This is source publication, not a GitHub Release.

## Human-run validation: exact observed versus reported gates

| Gate | Outcome | Evidence provenance |
| --- | --- | --- |
| Development distribution built from pre-commit changes | PASS | Maintainer console transcript: `DIST_BUILD: PASS`, `DIST_ARTIFACT_KIND: development-distribution` |
| Distribution runtime projection | PASS | Maintainer console: 12 projected JARs, `DIST_RUNTIME_PROJECTION_CHECK: PASS` |
| Classfile major | PASS | Maintainer console: `DIST_CLASSFILE_MAJOR: 69`, `PROTOS_CLASSFILE_BASELINE=PASS` |
| ZIP CRC/layout/source identity/runtime metadata/launcher | PASS | Maintainer console: each gate PASS |
| External native provider + existing I144/DOC022/DOC023/LIB029 learning integration | PASS | Maintainer console: all former 34 result keys PASS |
| New `DOC024_TUTORIAL_SQLITE_LIFECYCLE` | PASS | Maintainer console transcript |
| New `DOC024_EXAMPLE_NUMERIC` | PASS | Maintainer console transcript |
| New `DOC024_EXAMPLE_COPIED_VALUES` | PASS | Maintainer console transcript |
| `make test-protos-foreign` aggregate | **37/37 PASS** | Maintainer console transcript |
| `make check` | PASS | Maintainer final explicit statement: all local tests PASS |
| `make test` | PASS | Maintainer final explicit statement: all local tests PASS |
| `git diff --check` | CLEAN | Maintainer explicit statement after product push |
| Independent test/CI re-execution | NOT CLAIMED | Coordination agent did not run product tests |

The maintainer first attempted `make dist` with a dirty working tree and obtained the expected refusal; then attempted `make test-protos-foreign` without the shaded project JAR and obtained the expected missing-artifact refusal. Neither is counted as a source/test failure. The maintainer next ran `mvn -q -DskipTests package` and `make test-protos-foreign`, which produced the successful development distribution and full 37-case report.

### Relevant transcript extract

```text
DIST_CLASSFILE_MAJOR: 69
DIST_RUNTIME_PROJECTION_CHECK: PASS
DIST_RUNTIME_PROJECTION_JARS: 12
DIST_ARCHIVE_CRC_CHECK: PASS
DIST_LAYOUT_CHECK: PASS
DIST_SOURCE_IDENTITY_CHECK: PASS
DIST_RUNTIME_METADATA_CHECK: PASS
DIST_LAUNCHER_MODE_CHECK: PASS
DIST_ARTIFACT_KIND: development-distribution
DIST_BUILD: PASS
[foreign-provider] 37/37 passed
DOC024_TUTORIAL_SQLITE_LIFECYCLE=PASS
DOC024_EXAMPLE_NUMERIC=PASS
DOC024_EXAMPLE_COPIED_VALUES=PASS
```

The harness used a real external native FFM provider and the SQLite library for supported-platform cases. This confirms successful execution of these educational programs on the tested host, rather than an independently observed claim for every OS or Native Image.

## Preservation and technical boundary

DOC024 did not change the implemented C ABI, Java provider, `std:ffi` API or `std:sqlite` implementation. Its programs rely on one Statement per execution without reset, in-memory SQL only, Integer/Float/NULL input parameters, independently copied strict-UTF-8 TEXT/Bytes BLOB outputs and explicit `ensure` cleanup. No print-based provider-free tutorial was introduced. Previously registered provider-free denial and no-implicit-authorization cases remain intact.

```text
FORMAL_IDENTIFIER=DOC024
PROTOS_REVISION=7ae162258f67553a9bbfc02bb859d3ae328c0c0d
PUBLISHED_CHANGED_PATHS=16
NEW_TEACHING_PROGRAMS=3
FFM_HARNESS_RESULT_KEYS=37
FFM_RESULT=PASS_37_OF_37_MAINTAINER_TRANSCRIPT
DOC024_RESULT=PASS_3_OF_3_MAINTAINER_TRANSCRIPT
MAKE_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
DIFF_CHECK=CLEAN_MAINTAINER_REPORTED
CI_INDEPENDENT_VALIDATION=NOT_CLAIMED
```
