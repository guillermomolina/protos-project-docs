# D256-C1 — Comparative decision evidence, invariant reconciliation and owner approval

**Date:** 2026-11-11. **Owner's exact approval:** “apruebo D256-C1”; the same message requested coordination of `guillermomolina/protos` issues and explicitly authorized direct commits/pushes to `guillermomolina/protos-project-docs`. Approval was given after the C1 design packet; **the packet reserved two hard feasibility gates** (effective durable authority and synchronous execution/cancellation). Ratification records the architecture, **not a verified host enforcement method**.

[Live D256 #1118](https://github.com/guillermomolina/protos/issues/1118) · [Architecture record](../../decisions/language/D256_C1_PROTOS_FIRST_SQLITE_ARCHITECTURE_APPROVED_2026_11_11.md) · [Handoff](../../work/D256/D256_C1_IMPLEMENTATION_BOUNDARIES_2026_11_11.md).

## Inspected evidence and limitations

Current product tip observed for coordination: [`7ae162258f67553a9bbfc02bb859d3ae328c0c0d`](https://github.com/guillermomolina/protos/commit/7ae162258f67553a9bbfc02bb859d3ae328c0c0d). LIB029 acceptance revision [`90fd4ece800f6d0e007aa9a2c735ee2f7659919a`](https://github.com/guillermomolina/protos/commit/90fd4ece800f6d0e007aa9a2c735ee2f7659919a), which reports 34/34 maintainer-run foreign-provider cases, `make check` and `make test` PASS **for R1 only**. Those tests were not rerun in this decision.

Repository instructions inspected: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/COORDINATION.md`, `AGENTS.work/LIBRARY.md`, `protos/AGENTS.md`. Implemented/contract sources inspected: `protos/lib/sqlite/module.protos`, `protos/lib/sqlite/abi.protos`, `protos/lib/sqlite/internal/Resources.protos`, `protos/lib/sqlite/internal/Borrowed.protos`, `docs/guide/library/sqlite.md`, `spec/semantics/FFI.md`, `spec/io/IO_CORE.md` and the controlling `spec/io/FILESYSTEM.md` authority issue text. The complete post-selection normative audit across Actors/Tasks/P/Process/Errors is still a required handoff gate, not falsely claimed as complete.

R1 observed facts: `SQLite.openMemory()`; source-authored Connection/Statement state; semicolon/NUL-free single SQL string; i64/Float/null bind only; copied TEXT/BLOB output (16 MiB); native synchronous stepping; Actor-local nontransferable resources; explicit idempotent Future close; no durable acquisition. `abi.protos` already calls `sqlite3_open_v2` with `SQLITE_OPEN_MEMORY`. Borrowed column readers are private; public `std:ffi` exposes no arbitrary borrowed pointer memory access.

## GITHUB010 primary-source survey (7 comparators, 4 approaches)

| Comparator and primary source | Approach | Finding relevant to C1 |
|---|---|---|
| [SQLite C binding API](https://www.sqlite.org/c3ref/bind_blob.html) | Native SQLite | Explicit byte lengths and `SQLITE_STATIC`/`SQLITE_TRANSIENT` destructor semantics; C NULL/empty BLOB and embedded NUL require special handling. |
| [SQLite transactions](https://www.sqlite.org/lang_transaction.html), [autocommit](https://www.sqlite.org/c3ref/get_autocommit.html), [WAL](https://www.sqlite.org/wal.html) | Native transactional engine | SQLite owns locks, transaction state and crash recovery; SQLITE_BUSY commit can retain a transaction; WAL/PRAGMA modes affect durability and process coordination. |
| [Python sqlite3](https://docs.python.org/3/library/sqlite3.html) | Language wrapper / common DB-API | Distinct legacy/PEP249/autocommit models demonstrate hidden transaction policy cost; no reason to import Python defaults into Protos. |
| [Ruby sqlite3](https://github.com/sparklemotion/sqlite3-ruby) | Engine-specific binding | Shows useful SQLite-only API without general JDBC obligation. |
| [Node node:sqlite](https://nodejs.org/api/sqlite.html) | Engine-specific synchronous facade | `DatabaseSync`, `StatementSync` and native parameters demonstrate a focused API; synchronous blocking must remain explicit. |
| [better-sqlite3](https://github.com/WiseLibs/better-sqlite3/blob/master/docs/api.md) | Engine-specific ergonomic layer | Scoped transactional helpers can be optional; avoid mixing automatic and manual transaction management without a custody rule. |
| [Rust rusqlite](https://docs.rs/rusqlite/latest/rusqlite/struct.Connection.html) | Typed engine-specific API | Connection ownership, open flags, transaction helpers and statement lifetime directly inform Protos resource protocol. |
| [Rust SQLx sqlite options](https://docs.rs/sqlx/latest/sqlx/sqlite/struct.SqliteConnectOptions.html) | Cross-backend layer with SQLite-specific options | Journal/synchronous settings remain backend-specific even with a common interface. |
| [Go database/sql](https://pkg.go.dev/database/sql) | Common pool/driver abstraction | Cross-backend pooling, transaction context and driver responsibilities add semantics with no present second Protos engine. |

These are API/protocol precedents, **not performance benchmarks**. SQLite is the authority for C behavior; host confinement of C execution is **not** inferred from its VFS interface. No completed VFS security implementation is claimed.

## Candidate alternatives, pay-as-you-grow and disqualifiers

- **A:** direct incremental `openFile(String)` plus existing FFI; cheap and reusable SQL functions, but cannot honestly claim confined host authority. Unsuitable as the general secure durable contract; possible only as separately labelled trusted-in-process deployment.
- **B:** host-provisioned native database capability with an effective containment boundary; security-focused but adds native authority complexity and does not alone supply the layered guest API.
- **C1 (approved):** preserve `std:sqlite` R1, add needed SQLite-only mechanisms, optional ordinary Protos helpers, explicit host-provisioned durable acquisition with verified enforcement; keep exact mechanism conditional.
- **D:** universal `std:db`/JDBC abstraction, including drivers/pooling; unjustified cross-engine semantics and current cost. Preserve the *option* of extracting shared helpers later.
- **E:** defer expansion; minimizes immediate change but does not satisfy general durable SQLite/RealWorld.

A's underengineering red flag: publishing a path API now could require a later public incompatibility or permanent insecure bypass. D's overengineering red flag: API abstraction, driver compatibility and pooling before a second consumer. C1's cost and counterargument: effective native confinement might entail a VFS or isolation service, with portability/complexity that could invalidate the current implementation plan.

## 12-factor scored matrix

Scale 1 (weak) to 5 (strong). Confidence H = high, M = medium, L = low. Scores represent reasoned *design assessment*, not measurements.

| Dimension | A | B | C1 | D | E |
|---|---:|---:|---:|---:|---:|
| Correctness/invariants | 2H | 4M | 5M | 3M | 4H |
| Protos alignment | 4H | 3M | 5M | 2H | 3H |
| Present proportionality | 5H | 3M | 4M | 1H | 2H |
| Incremental growth | 3M | 4M | 5M | 3M | 2H |
| Future-option resilience | 3M | 4M | 5M | 4M | 4H |
| Scalability | 2M | 4M | 4M | 4M | 1H |
| Conceptual simplicity | 4H | 3M | 4M | 2H | 5H |
| Portability/freedom | 2H | 3L | 3M | 3M | 5H |
| Runtime/resource cost | 5M | 3L | 4M | 2M | 5H |
| Failure/operability | 2H | 4M | 4M | 3M | 2H |
| Deferral/reversibility | 2M | 3M | 4M | 2M | 2H |
| Evidence/implementation risk | 4H | 2L | 3M | 4M | 5H |
| **Indicative total (not decision authority)** | **38** | **40** | **50** | **33** | **40** |

Dimension-specific rationale, in table order: A fails confinement; C1 requires proof of enforcement. C1 best retains ordinary Protos and source-language mechanisms. A is initially cheaper; D charges current users for speculative backends. C1 can evolve without replacing R1 types. C1 preserves later backend alternatives without building them now. Concurrency can use SQLite locking under separate connections, though scheduling costs remain. E adds no concepts but fails present durable need. C1's portability depends on unverified host adapters. A adds little runtime cost; C1 must make the secure provider opt-in. B/C1 clarify native error boundaries rather than silently accepting arbitrary paths. C1 avoids a public incompatible unrestricted path. SQLite API precedent is mature, but C1 native confined acquisition remains unproven.

The score is **not** a security proof: any inability to enforce promised authority would fail the hard gate regardless of C1's total.

## GITHUB021 owner-invariant reconciliation

| Previously approved invariant | C1 delta and disposition |
|---|---|
| D238 optional trusted FFI, explicit aliases, no implicit native privilege | Preserve; SQLite needs bounded generic provider additions only when justified |
| LIB028 typed declarations, opaque handles and release retirement | Preserve; no public raw pointer arithmetic or new callable category |
| LIB029 R1 semantics and acceptance | Preserve; additive binding/reset/durable operations, no reopening of R1 PASS |
| Filesystem confined paths/Files | Preserve; **do not pretend** it confines SQLite C accesses |
| IO/Future operations and cancellation | Preserve existing explicit Future rules; SQLite synchronous calls remain synchronous pending separate decision |
| Actor/Process/P isolation and resource lifetime | Preserve; local handles only; copied rows carry ordinary values |
| D255 corrected owner objective | Preserve: SQLite-backed RealWorld, not replacement custom journal; H1 evidence remains historical |
| Anti-JDBC preference stated in active discussion | C1 avoids prebuilt common abstraction; does not prohibit future independently justified `std:db` |

**Formal approval scope:** the owner explicitly selected D256-C1 in this conversation. It did **not** approve a specific VFS, provider implementation, exact `open` signature, new Core capability, fully asynchronous SQL, Native Image support, RealWorld migration, or completion of D256's remaining authority/cancellation gates.

## Adversarial acceptance outline

1. Separate Protos processes A/B reopen the same database, verify exact i64/Float/TEXT/BLOB/SQL NULL, embedded NUL and distinct empty BLOB.
2. Commit/rollback, schema changes, restart and SIGKILL during commit; conservative reporting of uncertain outcomes.
3. WAL/SHM/journal/temp/ATTACH/VACUUM INTO/URI/symlink escapes fail closed under the **promised** confinement profile; cannot pass by relying only on path validation.
4. No provider / denied alias / wrong Actor / handle retired / destructor error / double close / malformed UTF-8 / oversized data / resource exhaustion.
5. Concurrent reader and writer, busy/locked/deadlock, disk full, read-only and corrupted database; autocommit reflects SQLite state after rollback-worthy errors.
6. Existing LIB029 R1 suites must remain green; RealWorld restart/auth/content integration is **additional**, never the SQLite core acceptance substitute.
7. Record exact SQLite library/version, provider, OS/ABI/VM, product revision and test output; explicitly mark unsupported platforms, do not infer Native Image compatibility.

## Audit limits

No code, specification, runtime, benchmarks, crash tests, or native VFS prototype were executed/changed in this decision. Future normative edits require full owner-domain review (`spec/semantics/FFI.md`, `spec/io/FILESYSTEM.md`, `spec/io/IO_CORE.md`, Actors/Tasks/P/Process/Errors/Modules). No claims about the installed SQLite build/version or its exact sync guarantees are made without per-platform testing.