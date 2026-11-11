# LIB029-S1 — Revision-bound SQLite publication and validation

**Date:** 2026-11-11  
**Owning Issue:** [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)  
**PROTOS_REVISION:** `90fd4ece800f6d0e007aa9a2c735ee2f7659919a`  
**Product commit:** https://github.com/guillermomolina/protos/commit/90fd4ece800f6d0e007aa9a2c735ee2f7659919a  
**Maintained work record:** [LIB029 completion](../../work/LIB029/LIB029_S1_PROTOS_FIRST_SQLITE_COMPLETION_2026_11_11.md)

## Independently inspected product publication

The live GitHub commit is present with **26 changed paths**, namely:

```text
CHANGELOG.md
dist/test_external_foreign_providers.py
docs/guide/interop/README.md
docs/guide/library/README.md
docs/guide/library/interop.md
docs/guide/library/sqlite.md
pom.xml
protos/examples/ffi/README.md
protos/examples/ffi/sqlite-library.protos
protos/lib/sqlite/abi.protos
protos/lib/sqlite/internal/Borrowed.protos
protos/lib/sqlite/internal/Resources.protos
protos/lib/sqlite/module.protos
protos/tests/foreign-provider/sqlite-borrow-model.protos
protos/tests/foreign-provider/sqlite-denied.protos
protos/tests/foreign-provider/sqlite-failure-model.protos
protos/tests/foreign-provider/sqlite-lifecycle.protos
protos/tests/foreign-provider/sqlite-missing.protos
protos/tests/foreign-provider/sqlite-query.protos
protos/tests/foreign-provider/sqlite-rows.protos
protos/tutorials/12-system-resources/05-sqlite-protos-first.protos
protos/tutorials/README.md
providers/ffm/README.md
src/main/java/com/guillermomolina/protos/execution/facilities/ProtosHttpAsyncFacility.java
src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java
src/test/java/com/guillermomolina/protos/execution/foreign/api/ProtosLib029SqliteLifecycleTest.java
```

The published `pom.xml` declares `0.3.462-SNAPSHOT`, `maven.compiler.release=25` and GraalVM `25.5.5.0.3`. The source changes no `spec/` path, and no SQLite-specific Java foreign provider implementation was introduced. These observations concern published source, not independent test execution.

## Human-executed validation evidence

| Gate | Result | Provenance |
| --- | --- | --- |
| `git diff --check HEAD` | CLEAN | Maintainer reported no output and staged 25 product paths before the final `pom.xml` metadata addition |
| Final Git staged change set | 26 paths published | GitHub commit inspected |
| `mvn -q -DskipTests test-compile` | PASS | Maintainer supplied terminal transcript, zero visible errors |
| `mvn -q -Dtest=ProtosLib029SqliteLifecycleTest,ProtosI144FfmFixtureAbiTest,ProtosI144ForeignExecutionAdmissionTest,ProtosExternalForeignProviderTest test` | PASS | Maintainer supplied terminal transcript, zero visible errors |
| `mvn -q '-Dtest=ProtosLib029SqliteLifecycleTest#acquisitionAndReleaseFailuresPreserveCustodyAndFirstError' test` | PASS | Maintainer supplied terminal transcript, zero visible errors |
| `make test-protos-foreign` | **34/34 PASS** | Complete maintainer-supplied external-provider transcript, no result marked UNSUPPORTED |
| `make check` | PASS | Maintainer explicit report |
| `make test` | PASS | Maintainer explicit report |
| Final diff check after metadata | CLEAN | Maintainer explicit post-push report |
| Independently rerun product tests or CI | NOT CLAIMED | No tests were run by this coordination action |
| SQLite upstream version / native binary hash | NOT REPORTED | Do not infer from the provider's PASS |

The submitted foreign-provider console also reported:

```text
DIST_CLASSFILE_MAJOR: 69
DIST_RUNTIME_PROJECTION_CHECK: PASS
DIST_ARCHIVE_CRC_CHECK: PASS
DIST_LAYOUT_CHECK: PASS
DIST_SOURCE_IDENTITY_CHECK: PASS
DIST_RUNTIME_METADATA_CHECK: PASS
DIST_LAUNCHER_MODE_CHECK: PASS
DIST_BUILD: PASS
[foreign-provider] 34/34 passed
LIB029_MISSING=PASS
LIB029_QUERY=PASS
LIB029_ROWS=PASS
LIB029_LIFECYCLE=PASS
LIB029_DENIED=PASS
LIB029_EXAMPLE=PASS
LIB029_TUTORIAL=PASS
```

The remaining 27 reported passing keys cover the existing I085/I144/DOC022/DOC023/provider/distribution cases. The transcript shows actual optional native provider execution on the tested host, not a promise for unsupported platforms. The deterministic failure-model and borrowed-model `.protos` programs are exercised through the focused Java lifecycle test, not asserted as additional standalone foreign-provider result keys.

## Scope and cautions

The native real-SQLite cases demonstrate normal open/query/row and close operations. Deterministic test doubles cover error-path contracts including non-null output on failed open, cleanup error precedence, borrowed byte ordering, NULL and invalid encodings; this does not constitute a real SQLite induced-native-failure proof.

Host authorization for `ffi:sqlite3` remains independent of merely importing `std:ffi` or `std:sqlite`. `sqlite-denied.protos` confirms rejection of an **alternative** unauthorized alias while legitimate `ffi:sqlite3` remains available; it is not a test of denial of the legitimately authorized alias. The implementation is trusted in-process and supplies neither sandboxing nor automatic Filesystem confinement.

This is a development implementation publication, not a tagged release. DOC024 #1116 owns additional pedagogical work, without altering the completed LIB029 contract.

```text
FORMAL_IDENTIFIER=LIB029
PROTOS_REVISION=90fd4ece800f6d0e007aa9a2c735ee2f7659919a
PRODUCT_CHANGED_PATHS=26
IMPLEMENTATION_VERSION=0.3.462-SNAPSHOT
JAVA_CLASSFILE_MAJOR=69
JAVA_FOCAL_TESTS=PASS_HUMAN_EXECUTED
LIB029_NATIVE_CASES=7_PASS_OF_7
TOTAL_FOREIGN_PROVIDER_RESULTS=34_PASS_OF_34
MAKE_CHECK=PASS_OWNER_REPORTED
MAKE_TEST=PASS_OWNER_REPORTED
POST_PUSH_DIFF_CHECK=CLEAN_OWNER_REPORTED
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
NATIVE_SQLITE_VERSION=UNKNOWN
```
