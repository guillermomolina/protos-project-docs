# LIB029-S1 — Protos-first SQLite source library completion

**Date:** 2026-11-11  
**Formal owner:** [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)  
**Ratified authority:** [D238-B1-SQL-R1](../../decisions/language/D238_PROTOS_FIRST_NATIVE_FFI_B1_SQL_R1_RATIFIED.md)  
**Prerequisite issues:** [I144 #1050](https://github.com/guillermomolina/protos/issues/1050) and [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051)  
**PROTOS_REVISION:** `90fd4ece800f6d0e007aa9a2c735ee2f7659919a`  
**Product commit:** https://github.com/guillermomolina/protos/commit/90fd4ece800f6d0e007aa9a2c735ee2f7659919a  
**Implementation version:** `0.3.462-SNAPSHOT` (development source; no new release implied)  
**Normative status:** D238 and published `spec/semantics/FFI.md` were reused, with no normative source path changed by this commit.  
**Validation evidence:** [LIB029-S1 publication and validation](../../evidence/LIB029/LIB029_S1_SQLITE_PUBLICATION_VALIDATION_2026_11_11.md)

## Outcome and ownership

LIB029 delivers a source-backed `std:sqlite` module implementing SQLite-specific policy in ordinary Protos. The provider remains the generic optional Java 25 FFM adapter from I144; no SQLite-specific C symbol is implemented as a Java method.

- `protos/lib/sqlite/module.protos` publishes `SQLite.openMemory()` and the ordinary `SQLiteError` surface.
- `protos/lib/sqlite/abi.protos` declares the SQLite C ABI through existing public `std:ffi`; `internal/Borrowed.protos` privately adapts the provider's established bounded borrowed-memory operations, without publishing new public FFI descriptors.
- `internal/Resources.protos` manages connection and statement custody, conservative SQL admission, numeric/NULL parameter binding, `step`, current-row materialization, ordinary callback iteration, and permanent close cutover.
- Two narrowly scoped Java files, `ProtosHttpAsyncFacility.java` and `ProtosCoreBootstrap.java`, expose/reuse a generic terminal-Future construction facility; they add no SQLite host API, eager native provider loading or native authorization.
- Seven Protos foreign-provider test programs, one focused Java lifecycle test, an assertion-based public example and a tutorial accompany the library; the existing external-provider integration runner registers the LIB029 cases.

## Observable supported subset

An application may open a private `:memory:` connection after the host independently authorizes `ffi:sqlite3`, prepare a single semicolon- and NUL-free statement, bind exact signed i64 Integer, binary64 Float or SQL NULL inputs by one-based index, advance a statement synchronously and read copied rows.

SQLite INTEGER/FLOAT/TEXT/BLOB/NULL outputs become ordinary Protos Integer/Float/strict String/fresh Bytes/`null`. Borrowed TEXT/BLOB contents are copied within the provider's 16 MiB limit before invalidating calls and before ordinary Protos callbacks. A null prepared handle on successful preparation means empty/comment-only SQL rather than an invented Statement. Returned native resources are retired once, statements before their connection, preserving the first cleanup Error and the standard fresh-terminal-Future close observations.

Failed native acquisition and cleanup-error paths are covered with deterministic Protos ABI doubles in addition to real SQLite happy-path integration. The doubles are model evidence, **not** a claim that public `:memory:` calls deterministically induce native failures.

## Boundaries preserved

Only Linux x86-64 LP64, portable JVM/JDK 25, the optional provider and host-provisioned `libsqlite3.so.0` have been positively exercised in the reported integrated result. No claim is made for other systems, Native Image loading, arbitrary file-backed SQLite, SQL scripting, TEXT/BLOB **input** binding, callbacks to C, pointers-as-numbers, a complete SQLite API, or a sandbox for trusted in-process C. No implicit Filesystem grant or Actor/P transfer of native handles is introduced.

## Publication and handoff

GitHub confirms the exact product commit and **26 changed paths**, including `pom.xml` and `CHANGELOG.md` at `0.3.462-SNAPSHOT`. The maintainer reported `git diff --check HEAD` clean, `make check` PASS, `make test` PASS, focused Java/compilation gates PASS, and the external-provider harness **34/34 PASS**, including all seven LIB029 result keys.

Tests were **human-executed**, not re-run through this GitHub coordination action. No independent CI result, exact `sqlite3_libversion_number` or shared-library file hash was supplied; these are not fabricated.

Separate public learning material follow-up: [DOC024 #1116](https://github.com/guillermomolina/protos/issues/1116). DOC024 is independent and does not retroactively block LIB029 implementation closure.

```text
FORMAL_IDENTIFIER=LIB029
SLICE=LIB029-S1
PROTOS_REVISION=90fd4ece800f6d0e007aa9a2c735ee2f7659919a
IMPLEMENTATION_VERSION=0.3.462-SNAPSHOT
PRODUCT_SOURCE_PUBLICATION=PASS_GITHUB_VERIFIED
PRODUCT_CHANGED_PATHS=26
GIT_DIFF_CHECK=CLEAN_MAINTAINER_REPORTED
FOCUSED_JAVA_TESTS=PASS_MAINTAINER_REPORTED
EXTERNAL_PROVIDER_CASES=34_OF_34_PASS_CONSOLE_SHARED
MAKE_CHECK=PASS_MAINTAINER_REPORTED
MAKE_TEST=PASS_MAINTAINER_REPORTED
INDEPENDENT_CI=NOT_CLAIMED
SQLITE_LIBRARY_VERSION=NOT_RECORDED
DOC024=INDEPENDENT_OPEN
```
