# LIB030-S1 — Protos-first SQLite R1 extension completion

**Date:** 2026-11-11  
**Formal owner:** [LIB030 #1119](https://github.com/guillermomolina/protos/issues/1119)  
**Architecture:** [D256-C1](../../decisions/language/D256_C1_PROTOS_FIRST_SQLITE_ARCHITECTURE_APPROVED_2026_11_11.md)  
**R1 predecessor:** [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052)  
**PROTOS_REVISION:** `5e89b6df54ecdbfb71c96f0269d11fc5037ce940`  
**Product commit:** https://github.com/guillermomolina/protos/commit/5e89b6df54ecdbfb71c96f0269d11fc5037ce940  
**Implementation version:** `0.3.463-SNAPSHOT` (development source, not a tagged release)  
**Validation evidence:** [LIB030-S1 publication and reported validation](../../evidence/LIB030/LIB030_S1_SQLITE_EXTENSION_PUBLICATION_VALIDATION_2026_11_11.md)

## Delivered product behavior

LIB030-S1 extends the LIB029 source-backed `std:sqlite` API without adding a public `std:ffi` descriptor, a SQLite-specific Java database service, or a new language/Core feature.

- `Statement.bind(index, value)` accepts strict UTF-8 String/TEXT and binary Bytes/BLOB alongside signed i64 Integer, Float and `null`. Embedded NULs, empty TEXT, empty BLOB and SQL NULL retain distinct meanings. Input values are independently copied; the library enforces a 16 MiB per-value input limit.
- The private generic FFM provider supports the necessary length-carrying text/octets, computed length, transient copy destructor and in-call SQL tail measurement. Bytes traverse the existing D188 scalar foreign boundary through an explicitly private lossless Latin-1 mapping; the provider converts scalars back to octets. The native `SQLITE_TRANSIENT` sentinel is never Protos-visible.
- `Connection.prepare(sql)` uses SQLite's own parser to admit exactly one complete SQL statement with ordinary `;` terminators, quoted delimiters, trailing whitespace/comments and SQL-internal separators. Remaining input is re-parsed while preventing silent acceptance of a second statement. `pzTail` does not escape its native call lifetime.
- `Statement.reset()` explicitly restarts a statement after SQLite confirms success without clearing bindings. `Statement.clearBindings()` clears bound values to SQL NULL without restarting execution. DONE does not implicitly re-execute SQL; failed/uncertain steps are not retried implicitly.
- Actor-local custody, copied output rows, synchronous native calls, failure classification, resource retirement and fresh terminal-Future close semantics remain in the library.
- New real-provider cases cover text/blob input, SQL admission, statement reuse and input limits; deterministic model and FFM/C fixture tests cover failure and memory-contract paths. Examples, tutorials, provider and SQLite guides were updated.

## Scope and boundaries

This is the independently closable in-memory LIB030 portion of D256-C1. It does **not** enable file-backed durable SQLite acquisition, WAL/SHM confinement, a host database authority, transaction contracts, background cancellation, `std:db`, JDBC, ORM, pooling, or RealWorld persistence. [D256 #1118](https://github.com/guillermomolina/protos/issues/1118) remains open for durable-authority and Actor blocking/cancellation decisions; [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120) remains blocked by those gates, notwithstanding completion of its LIB030 prerequisite.

The 16 MiB input bound is a `std:sqlite` policy, not a generic FFM limit; the provider only enforces its ABI-safe integer range. No `spec/` file was changed by the published revision. Native in-process C remains trusted, without a new sandbox or implied Filesystem authority.

## Publication and validation

GitHub confirms product commit `5e89b6df54ecdbfb71c96f0269d11fc5037ce940`, **27 changed paths**, including the version update to `0.3.463-SNAPSHOT`. The maintainer reports a clean post-push Git diff check and all local tests passing. The product changelog additionally reports focused Java tests, external foreign-provider integration, `make check` and `make test` as PASS. These are maintainer-reported results; this coordination action did not independently run those tests or receive a case-by-case console transcript.

Remaining caveats: native allocation-failure coverage is modeled rather than induced in production SQLite; a provider failure after native return may retain an adopted Statement until session shutdown under the existing R1 custody contract. No exact SQLite native version, shared-library hash or independent CI result is claimed.

```text
FORMAL_IDENTIFIER=LIB030
SLICE=LIB030-S1
PROTOS_REVISION=5e89b6df54ecdbfb71c96f0269d11fc5037ce940
PRODUCT_CHANGED_PATHS=27
IMPLEMENTATION_VERSION=0.3.463-SNAPSHOT
PRODUCT_PUBLICATION=GITHUB_VERIFIED
GIT_DIFF_CHECK=CLEAN_MAINTAINER_REPORTED
LOCAL_TEST_SUITE=PASS_MAINTAINER_REPORTED
FOCUSED_JAVA=PASS_MAINTAINER_CHANGELOG
EXTERNAL_FOREIGN_PROVIDER=PASS_MAINTAINER_CHANGELOG
MAKE_CHECK=PASS_MAINTAINER_CHANGELOG
MAKE_TEST=PASS_MAINTAINER_CHANGELOG
INDEPENDENT_TEST_EXECUTION=NOT_CLAIMED
NATIVE_SQLITE_VERSION=NOT_RECORDED
D256=OPEN
LIB031=BLOCKED
```
