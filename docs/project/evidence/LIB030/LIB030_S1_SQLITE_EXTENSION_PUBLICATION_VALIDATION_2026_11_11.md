# LIB030-S1 — Revision-bound SQLite extension publication and validation

**Date:** 2026-11-11  
**Owning issue:** [LIB030 #1119](https://github.com/guillermomolina/protos/issues/1119)  
**PROTOS_REVISION:** `5e89b6df54ecdbfb71c96f0269d11fc5037ce940`  
**Product commit:** https://github.com/guillermomolina/protos/commit/5e89b6df54ecdbfb71c96f0269d11fc5037ce940  
**Maintained work record:** [LIB030 completion](../../work/LIB030/LIB030_S1_PROTOS_FIRST_SQLITE_EXTENSION_COMPLETION_2026_11_11.md)

## Independently verified publication

The GitHub commit exists on `guillermomolina/protos` with **27 changed paths**, including:

```text
CHANGELOG.md
dist/test_external_foreign_providers.py
docs/guide/interop/ffm-provider.md
docs/guide/library/sqlite.md
pom.xml
protos/examples/ffi/README.md
protos/examples/ffi/sqlite-complete-public.protos
protos/examples/ffi/sqlite-prepared-numeric.protos
protos/lib/sqlite/abi.protos
protos/lib/sqlite/internal/Resources.protos
protos/lib/sqlite/module.protos
protos/tests/foreign-provider/ffm-inputs.protos
protos/tests/foreign-provider/native/ffi_test.c
protos/tests/foreign-provider/sqlite-bind-text-blob.protos
protos/tests/foreign-provider/sqlite-borrow-model.protos
protos/tests/foreign-provider/sqlite-failure-model.protos
protos/tests/foreign-provider/sqlite-input-limits.protos
protos/tests/foreign-provider/sqlite-query.protos
protos/tests/foreign-provider/sqlite-sql-admission.protos
protos/tests/foreign-provider/sqlite-statement-reuse.protos
protos/tutorials/12-system-resources/05-sqlite-protos-first.protos
protos/tutorials/12-system-resources/06-sqlite-query-lifecycle.protos
providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmBinding.java
providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmTypes.java
providers/ffm/src/main/java/com/guillermomolina/protos/ffm/FfmValueOperations.java
src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosI144FfmFixtureAbiTest.java
src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosLib029SqliteLifecycleTest.java
```

The published `pom.xml` identifies version `0.3.463-SNAPSHOT`. The commit touches no normative `spec/` paths and does not include a new public `std:ffi` API source or platform authority. This is a product source publication, not a tagged release.

## Validation evidence and provenance

| Gate | Result | Source / limitation |
| --- | --- | --- |
| Published product commit | VERIFIED | GitHub commit `5e89b6df54ecdbfb71c96f0269d11fc5037ce940`, 27 files |
| Published version | `0.3.463-SNAPSHOT` | GitHub `pom.xml` inspected at exact revision |
| `git diff --check` | CLEAN | Maintainer's explicit post-push report |
| Local test suite | PASS | Maintainer's explicit report: “todos los tests pasan en local” |
| Focused Java tests | PASS, maintainer-reported | Published CHANGELOG notes focused Java PASS |
| External foreign-provider integration | PASS, maintainer-reported | Published CHANGELOG notes full foreign-provider integration PASS |
| `make check` | PASS, maintainer-reported | Published CHANGELOG; not independently rerun |
| `make test` | PASS, maintainer-reported | Published CHANGELOG; not independently rerun |
| Individual LIB030 integration case counts | NOT RECORDED | No case-by-case transcript supplied in this handoff |
| Independent CI / this coordination action tests | NOT CLAIMED | Source publication inspected, not executed |
| Native SQLite version / binary hash | NOT RECORDED | No version/hash measured in this coordination action |

The integration harness registers `LIB030_BIND_TEXT_BLOB`, `LIB030_STATEMENT_REUSE`, `LIB030_SQL_ADMISSION`, and `LIB030_INPUT_LIMITS`. Registration is independently visible in the published sources; individual case result lines were not supplied and are not independently asserted.

## Semantic and safety scope

The public extension supports TEXT/UTF-8 String and BLOB/Bytes input, 16 MiB `std:sqlite` input bounds, native-syntax-checked single-statement preparation, explicit `reset()` and `clearBindings()`, and continued LIB029 R1 resource and copied-output contracts. The provider-private descriptors carry input bytes and tail information without exposing native pointers or `SQLITE_TRANSIENT` to Protos. The Latin-1 mapping used to convey Bytes across D188 is a private reversible transport rather than a public text coercion.

Deterministic ABI doubles and C fixtures are coverage for error/ownership ordering; they do not constitute proof of induced real SQLite allocation failure or universal safety across unsupported platforms. A provider post-call failure can leave an adopted Statement in session custody until session close, consistent with the recorded R1 limitation. No durable file database, WAL/SHM confinement, cancellation of native SQLite calls or transaction abstraction is delivered.

## Coordination boundaries

- [LIB030 #1119](https://github.com/guillermomolina/protos/issues/1119) is independently complete at this revision and can close.
- [D256 #1118](https://github.com/guillermomolina/protos/issues/1118) remains OPEN pending exact durable authority and native blocking/cancellation decisions.
- [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120) remains BLOCKED by D256 despite LIB030 completion.
- [D255 #1107](https://github.com/guillermomolina/protos/issues/1107) and [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) remain separately owned RealWorld work.

```text
FORMAL_IDENTIFIER=LIB030
PROTOS_REVISION=5e89b6df54ecdbfb71c96f0269d11fc5037ce940
PRODUCT_CHANGED_PATHS=27
IMPLEMENTATION_VERSION=0.3.463-SNAPSHOT
GITHUB_PRODUCT_COMMIT=VERIFIED
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
LOCAL_TESTS=PASS_OWNER_REPORTED
EXTERNAL_PROVIDER=PASS_CHANGELOG_OWNER_REPORTED
MAKE_CHECK=PASS_CHANGELOG_OWNER_REPORTED
MAKE_TEST=PASS_CHANGELOG_OWNER_REPORTED
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
LIB030_CASE_COUNTS=NOT_SUPPLIED
NATIVE_SQLITE_VERSION=UNKNOWN
```
