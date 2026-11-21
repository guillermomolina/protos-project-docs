# D256-C1R — Owner approval, trusted-native SQLite and minimal implementation slicing

**Date:** 2026-11-21  
**Live design:** [D256 #1118](https://github.com/guillermomolina/protos/issues/1118)  
**Delivery owner:** [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120)  
**Prior design:** [D256-C1 architecture](../../decisions/language/D256_C1_PROTOS_FIRST_SQLITE_ARCHITECTURE_APPROVED_2026_11_11.md)  
**Reconciliation:** [PLAT074 withdrawal after D259](../PLAT074/PLAT074_D259_TRUSTED_SQLITE_CONFINEMENT_WITHDRAWAL_2026_11_21.md)

## Exact approval provenance

After a D256 design investigation, the owner was given this short explicit recommendation:

> Recomiendo D256-C1R: SQLite Protos-first, aprovechando todo lo que ya tenemos implementado. SQLite nativo se encarga de los archivos, transacciones, WAL, locking y recuperación; Protos implementa la API mediante objetos y Closures ordinarios; FFM existente conecta Protos con SQLite; D259 autoriza la biblioteca nativa y el host acepta todos sus riesgos, sin Landlock ni confinamiento; las conexiones siguen siendo Actor-local y las operaciones síncronas sin suspensión implícita. Siguiente paso: aprobar D256-C1R, fijar el contrato en la especificación y desbloquear LIB031.

**Owner's direct response:** “ok apruebo eso, lo que no apruebo son tantos slices, quiero que se haga en la menor cantidad de slices posible. Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs.”

Thus **D256-C1R is explicitly owner-approved as the selected design direction**, and the owner separately **rejects the proposed multi-slice decomposition**. Approval does not falsely certify that new normative specification text, executable SQLite durability or its tests have already been published.

## Selected Protos-first architecture and observable boundaries

1. Expand the existing ordinary-Protos `std:sqlite` MODULE, `protos/lib/sqlite/abi.protos`, and `protos/lib/sqlite/internal/Resources.protos`. Use Connection/Statement objects and ordinary Closures, existing Errors and lifecycle protocols. No new Core type, reserved syntax, privileged Database capability, generic `std:db`, JDBC, ORM or Java-specific SQLite subsystem.
2. Keep native SQLite responsible for file-backed open, lock handling, WAL/journal, commit/rollback and recovery. Reuse existing generic `std:ffi` and optional FFM provider; authorize `sqlite3` explicitly through D259's `--allow-library sqlite3=libsqlite3.so.0` or the host embedding equivalent. No implicit authorization on `import("std:sqlite")`.
3. The host accepts the trusted C library's OS-permitted side effects, including file access beyond Protos guest `Filesystem` grants. There is no promise of SQLite OS sandboxing, no Landlock worker, custom VFS, isolated subprocess or IPC requirement. Preserve normal Protos guest capability guarantees *for guest API operations*.
4. Keep a synchronous in-Actor execution segment for SQLite operations. Native blocking occupies the invoking Actor. No implicit Future, suspension, Actor reentrancy, automatic native cancellation, hidden executor or timeout guarantee for arbitrary queries. SQLite's busy timeout, if exposed, only bounds qualifying SQLite lock waits.
5. Preserve Actor-local Connection/Statement and FFM session custody, non-transferability, explicit terminal-Future close and once-only resource retirement. No silent retry of a native destructor; preserve the primary error across cleanup.
6. Expose file-backed SQLite and explicit transaction control through ordinary `std:sqlite` methods. The illustrative names `openFile`, `begin`, `commit`, `rollback`, `inTransaction` and mode spellings from the design packet are **not a separately ratified complete exact signature/default/error catalogue**. Their precise normative shape must be reconciled with existing Protos conventions as part of the same bounded D256 specification handoff; genuinely new observable decisions must return for owner approval rather than be hidden in implementation.
7. SQLite is the transaction-state authority. Never mirror it with a Protos-only Boolean. After failed commit or native errors, query actual SQLite transaction state when still observable; do not promise rollback, non-effect or power-loss guarantees that SQLite/the underlying storage do not provide.
8. Preserve completed LIB029/LIB030: `openMemory`, input TEXT/BLOB/null, full single-statement admission, copied rows, explicit reset/clearBindings, exact numeric domains, existing R1 close semantics and supported Linux x86-64 / JDK25 JVM profile. Native Image/other platforms require independent tests.

## GITHUB021 approved-invariant / delta table

| Earlier invariant or unresolved requirement | D256-C1R disposition |
| --- | --- |
| D256-C1 Protos-first and no JDBC/Java-first special case | PRESERVED |
| LIB029/LIB030 existing observable behavior | PRESERVED; regressions mandatory |
| D238 FFI host authorization and no implicit library grant | PRESERVED |
| D259-B2+A host-trusted in-process native execution | PRESERVED |
| Earlier D256-C1 effective native filesystem confinement gate | SUPERSEDED explicitly by owner-selected trusted-native boundary; [PLAT074 #1122](https://github.com/guillermomolina/protos/issues/1122) `not_planned` |
| Actor-local resources and synchronous FFM semantics (ACTORS §24J/§24K.1) | PRESERVED |
| SQLite owns WAL, locking, transactions and recovery | PRESERVED |
| Explicit transaction truth; failed COMMIT is not automatic rollback | PRESERVED |
| Unresolved exact method signatures, defaults and SQLiteError mappings | REMAIN TO BE NORMATIVELY RECONCILED; no invented approval claimed |
| Former implementation prerequisite I169 #1140 | SATISFIED: [published product commit `c808de27b35137c064b8166ed7cca20e34a6db59`](https://github.com/guillermomolina/protos/commit/c808de27b35137c064b8166ed7cca20e34a6db59), maintainer-reported local tests PASS |
| Provisional four-slice LIB031 roadmap | REJECTED by owner; target ONE integrated implementation/publication slice |

## Minimum-slice handoff

**One coherent LIB031 implementation slice is the target, not four staged slices or new child issues.** Complete the necessary normative/public method contract, source changes, tests, examples/guides, version and CHANGELOG for the bounded deliverable together; preserve existing validations. The design/spec publication prerequisite may require a distinct approval/publishing checkpoint under AGENTS policy, but must **not** be multiplied into artificial implementation slices. Only an independently proven technical blocker, a genuinely new owner decision or an unavoidable separately publishable change justifies revising this one-slice plan. Do not split by open/transactions/recovery/docs as a default.

Integrated acceptance within that slice must cover: create and reopen file DB across two OS processes; read-only/read-write/create modes; parameterized exact i64/Float/String/Bytes/null round-trip; explicit begin/commit/rollback and SQLite-derived transaction state; busy/locked behavior and failed commit; WAL/journal and concurrency under supported settings; terminal close, error cleanup, session/Actor nontransferability; crash/restart and I/O failure cases; full LIB029/LIB030 regression; real FFM/distribution path when available. Distinguish platform UNSUPPORTED from PASS. No claims of completed tests or product changes are made by this record.

**Inspected product anchor:** [I169 executable baseline `c808de27b35137c064b8166ed7cca20e34a6db59`](https://github.com/guillermomolina/protos/commit/c808de27b35137c064b8166ed7cca20e34a6db59), including `protos/lib/sqlite/module.protos`, `abi.protos`, `internal/Resources.protos`, and `spec/semantics/FFI.md`. **Normative D259 anchor:** [`ee8be92c482cc2451e888d444d74b382b319f4f1`](https://github.com/guillermomolina/protos/commit/ee8be92c482cc2451e888d444d74b382b319f4f1), spec `0.1.506`. No new product commit, local validation command, or test run is claimed here.

`D256_C1R_OWNER_DIRECTION_APPROVED=YES`  
`D256_ONE_INTEGRATED_LIB031_SLICE_SELECTED=YES`  
`D256_NORMATIVE_EXACT_API_PUBLISHED=NO`  
`LIB031_IMPLEMENTED_OR_TESTED=NO`
