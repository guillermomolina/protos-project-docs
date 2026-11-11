# D256-C1 — Bounded implementation handoff and unresolved design gates

**Owner-approved architecture:** [D256-C1](../../decisions/language/D256_C1_PROTOS_FIRST_SQLITE_ARCHITECTURE_APPROVED_2026_11_11.md), explicitly approved in the active conversation on 2026-11-11. Research/approval evidence: [GITHUB010/GITHUB021](../../evidence/D256/D256_C1_GITHUB010_GITHUB021_APPROVAL_EVIDENCE_2026_11_11.md). Live design issue: [D256 #1118](https://github.com/guillermomolina/protos/issues/1118).

**Important:** An architectural direction has been selected. The exact durable-provider confinement and Actor blocking/cancellation policies were expressly reserved in the decision packet and need concrete design resolution. Implementation ownership is only allocated where independently executable without making either assumption.

## Workstreams

### Library-level SQLite surface (independent of disk confinement)

- Preserve R1 `openMemory()`, `prepare`, numeric/NULL `bind`, `step`, `column`, `row`, `each`, `close` semantics.
- Add byte-length-aware copied input TEXT/String and BLOB/Bytes; handle embedded NUL, SQL NULL, empty TEXT and empty BLOB distinctly; correct ABI lifetime and error paths. The generic FFI/provider must provide the minimal reusable safe mechanism. Arbitrary guest-visible native pointers are excluded.
- Add a rigorously complete **single-statement** SQL boundary: conventional trailing semicolon allowed, multiple executable statements never silently dropped, `pzTail` inspected only within valid native-call memory lifetime.
- Add explicitly controlled Statement reset and clearBindings (SQLite reset does not clear old bindings). Keep R1 completed-state behavior unchanged until reset is called.
- Add the connection/statement primitives needed for explicit transactions and introspection (get_autocommit/txn state), metadata, busy status and bounded configuration, but do not invent durable authority semantics.
- Gate with deterministic mock-provider unit cases and real FFM Linux/JVM integration for memory-only operations.

### Host/native database authority (hard design gate — not preselected)

- Compare bounded host-provisioned DB identity + SQLite-aware VFS, OS-enforced data scope, and explicitly labelled trusted-in-process mode; document effective filesystem authority of C code and limitations.
- Threat model main DB, journals, WAL, SHM, temporary files, external-file SQL (ATTACH, VACUUM INTO), extension loading, URI modes, symlinks, races, lock files, revocation and errors.
- Do not implement an unsandboxed `openFile(path)` that is described as confined. A Protos `Filesystem` capability or checked path prefix is insufficient.
- If this requires a new independently reusable host/capability design, present a separate PLAT/D design approval gate and allocate implementation in the corresponding family only after approval.

### Blocking, transaction/cancellation (hard design gate)

- Keep existing synchronous calls explicit until any alternative has been evaluated. Evaluate bounded `busy_timeout`, potential CPU/disk/lock blocking in the Actor, native interrupt safety, pending work at close and commit uncertainty.
- Derive state from real SQLite autocommit; a failed COMMIT may leave an active transaction, and errors may auto-rollback.
- Define exact failures, acknowledgment and cleanup: an unobserved result or termination cannot convert an entered native operation into guaranteed non-effect.
- Explicit BEGIN/COMMIT/ROLLBACK, savepoints only as justified, optional Protos callback helpers after manual operations are sound; no implicit suspension or unapproved task machinery.

### After library and provider viability

- Consumer-independent SQLite acceptance: two independent Protos processes, same DB, reopen across restart, exact copied parameterized TEXT/BLOB/NULL/i64, commit/rollback, contention, SIGKILL recovery, WAL sidecars, denial and malicious path/SQL operations; all existing LIB029 tests preserved.
- [D255 #1107](https://github.com/guillermomolina/protos/issues/1107) must separately select RealWorld's SQLite persistence architecture: schema/versioning, RootActor custody, data identity and JWT revocation, recovery before HTTP listen and disposition of journal branch. D256 cannot approve that consumer contract.
- [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) remains blocked for SQLite durability until the necessary LIB/provider work is tested and D255's application migration is approved. The journal tests may be retained as historical evidence but never count as SQLite proof.

## Public-surface triage

| Surface | Ownership | Now or later |
|---|---|---|
| Memory open, copied rows, close | `std:sqlite` | Already R1; preserve |
| Input String/Bytes and bind lifetime | `std:sqlite` + generic FFI/provider mechanism | Now |
| Complete single-statement admission | `std:sqlite` + bounded tail-copy mechanism if needed | Now |
| Reset, clearBindings, bound parameter indices | `std:sqlite` | Now |
| Durable open, file identity/modes and locking | `std:sqlite` + host-approved authority mechanism | Now, only after hard gate |
| BEGIN/COMMIT/ROLLBACK/autocommit | `std:sqlite` | Now |
| Journal/synchronous config, busy timeout, resource limits | `std:sqlite` | Now, bounded/explicit |
| Transaction Closure, migrations, savepoint convenience | Optional ordinary Protos modules | Later |
| Pool, ORM, query builder, backups, incremental blob, callbacks | Separate future work | Defer |
| Generic `std:db` or Core Storage | No current owner | Reject in D256-C1 |

## Next exact design checkpoint

Before closing D256, submit to owner:
1. Concrete durable host grant and enforcement mechanism, including threat model and adversarial evidence, plus the exact application-visible capability shape.
2. Exact synchronous blocking/cancellation/close/transaction state and uncertain-commit contract, reconciled with Actors, Futures and Core I/O.
3. Fully specified public method signatures, error categories, option defaults and compatibility delta.
4. Exact supported platforms and explicit unsupported scope.
5. GITHUB021 invariant-delta table and independently executable acceptance tests.

Only then can D256 be considered fully ratified/closed and durable implementation unblocked. No source, tests, product version or RealWorld code is changed by this record.

`D256_C1_DIRECTION=APPROVED`
`D256_AUTHORITY_AND_CANCELLATION_GATES=OPEN`
`D256_CODE_OR_TEST_ACCEPTANCE=NOT_CLAIMED`
`D255_REALWORLD_SQLITE_HANDOFF=NOT_YET_APPROVED`
