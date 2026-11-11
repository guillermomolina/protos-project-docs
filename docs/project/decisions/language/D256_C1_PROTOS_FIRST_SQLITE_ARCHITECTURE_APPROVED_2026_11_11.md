# D256-C1 — Protos-first SQLite library with explicit durable authority

**Status: OWNER-APPROVED ARCHITECTURAL DIRECTION, 2026-11-11.** The exact durable authority mechanism and blocking/cancellation guarantees remain **UNRESOLVED VIABILITY GATES**. This is a non-normative architectural decision record, **not** an assertion that every proposed public signature, native provider change, security guarantee, or implementation has been approved or verified.

**Owner approval:** In the active D256 conversation, after the detailed GITHUB010/GITHUB021 decision packet recommending C1, the owner wrote: **“apruebo D256-C1”** and explicitly authorized GitHub Issue coordination and commit/push in `guillermomolina/protos-project-docs`. This selects the C1 direction without converting the packet's explicitly provisional examples or remaining gates into approved normative specifications.

**Canonical live decision:** [D256 #1118](https://github.com/guillermomolina/protos/issues/1118). **Library baseline:** [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052), completed at product commit [`90fd4ece800f6d0e007aa9a2c735ee2f7659919a`](https://github.com/guillermomolina/protos/commit/90fd4ece800f6d0e007aa9a2c735ee2f7659919a). **Latest inspected product tip during coordination:** [`7ae162258f67553a9bbfc02bb859d3ae328c0c0d`](https://github.com/guillermomolina/protos/commit/7ae162258f67553a9bbfc02bb859d3ae328c0c0d); no product source is changed by this decision.

Companion [research, scored matrix and invariant evidence](../../evidence/D256/D256_C1_GITHUB010_GITHUB021_APPROVAL_EVIDENCE_2026_11_11.md) and [implementation handoff/remaining gates](../../work/D256/D256_C1_IMPLEMENTATION_BOUNDARIES_2026_11_11.md).

## 1. Selected invariants

1. The public module remains **SQLite-specific `std:sqlite`**, not JDBC, a generic `std:db`, `std:storage`, a mandatory ORM, nor a C ABI mirror.
2. Continue **Protos-first**: ordinary Protos owns Connection/Statement state, transactions, error protocols and high-level helpers; `std:ffi` and the optional trusted provider supply only required native machinery. No new language syntax, classes, global privilege, or SQLite-specific Java/JNI bypass.
3. **Preserve LIB029 R1**: `SQLite.openMemory()`, `prepare`, numeric/null `bind`, `step`, copied rows, Actor-local resources, synchronous native entry and explicit idempotent terminal-Future `close` retain existing observable behavior.
4. Extend usable SQLite with **durable acquisition**, safe parameterized TEXT/BLOB input, complete single-statement admission, explicit reset and clear-bindings, transaction control and real SQLite transaction-state observation, bounded locking policy, durable configuration and safe lifecycle/failure semantics.
5. SQLite owns atomic transaction and recovery protocols. Protos does **not** implement a duplicate application journal, new database engine or purported SQLite-compatible WAL.
6. An import or ordinary filename does **not** confer native filesystem authority. A Protos `Filesystem` grant **does not confine trusted in-process C**. Any advertised durable confinement must be grounded in an **effective enforced native/host boundary**, including main DB, WAL, SHM, journals, temp files, symlinks and external-file SQL operations.
7. Actor-local Connection and Statement resources are not transferred across Actors/Processes/P; copied values use existing Protos transfer rules. No hidden suspension or automatic task creation.
8. Only verified Linux x86-64/JDK25 portable JVM/authorized provider properties may be claimed initially. Native Image and other platforms require independent evidence.

## 2. Selected layers and excluded features

- **Core SQLite module:** memory acquisition, independently authorized durable acquisition, prepared statements, parameters `Integer(i64)/Float/String/Bytes/null`, copied rows, reset/clear-bindings, explicit transaction begin/commit/rollback, underlying autocommit status, busy/locked reporting, bounded resource limits and explicit close.
- **Optional ordinary Protos modules:** scoped transactional Closures, savepoints, schema migration utilities, application repositories. Their exact APIs must be justified independently and must not force overhead on the core.
- **Deferred:** pooling, online backup, incremental BLOB, native SQL callbacks, user functions/collations, virtual tables, extension loading, general script executor, elaborate ORM/query builder and universal database API.
- **Explicit rejection now:** open arbitrary unrestricted file paths as though confined, implicitly loading native libraries, silent SQL interpolation for user data, implicit multi-statement prefix execution and Java-first special-case SQLite runtime.

The example syntax `SQLite.open(databaseCapability, options)`, `db.begin(mode)`, `db.inTransaction()`, etc. is **provisional API illustration**, not a settled method-signature ratification. Precise value protocols, error ancestry, options, typed capability representation, and state transitions must be reconciled with normative owners before product/spec publication.

## 3. Unresolved hard viability gates (not authorized by approval)

**AUTHORITY GATE:** Select and demonstrate an actual acquisition and I/O confinement mechanism. Compare a host-provisioned database resource plus genuinely enforced VFS/OS boundary with an explicitly labelled trusted-in-process deployment mode. A validated pathname, string token or SQLite SQL authorizer alone is not a filesystem sandbox. Determine how SQL ATTACH/VACUUM INTO, URI modes, symlink races, WAL/SHM/temp files, locking and revoked grants behave. If a safe confined provider requires a new independent PLAT decision, return to the owner. Do **not** silently claim confinement or create new Core Storage.

**EXECUTION/CANCELLATION GATE:** Specify what synchronous native blocking (including busy timeout, disk sync and long-running queries) means for an Actor, whether cancellation can affect work already entered, and the limits of safe cleanup/commit certainty. Do not promise safe interruption or automatic Future suspension without a separate verified design.

These gates are part of the approved direction, **not completed choices**. D256 must remain open as a design gate until the exact authority and lifecycle contract is separately ratified. Independent additive implementation slices not depending on the unresolved gates may be tracked, but are not proof of durable readiness.

## 4. Transaction and data constraints

SQLite autocommit is the transaction state authority; a Protos-only Boolean is insufficient because errors may trigger an automatic rollback. A failed COMMIT, including SQLITE_BUSY, does not necessarily end a transaction. Helpers must preserve the primary error and distinguish failed cleanup, avoid unapproved implicit retries and never report success for uncertain commits. Durable acknowledgment must depend on tested SQLite journal/WAL and synchronous settings, VFS guarantees and the underlying storage, rather than a universal promise of power-loss safety.

TEXT and BLOB input require an explicit native lifetime/copy policy, proper lengths (embedded NUL), distinct SQL NULL and empty values, strict UTF-8 admission, independent captured Bytes and bounded allocation. The provider must not surface raw native pointers or an invented `SQLITE_TRANSIENT` C sentinel without an approved representation.

SQL admission must accept a **complete** single statement, including a valid trailing semicolon, without silently compiling only a prefix or admitting arbitrary multi-statement scripts. R1 reset/completion semantics must not be changed by implicit re-execution.

## 5. Previous decisions preserved (GITHUB021)

- [D238 #1047](https://github.com/guillermomolina/protos/issues/1047), [LIB028 #1051](https://github.com/guillermomolina/protos/issues/1051) and [I144 #1050](https://github.com/guillermomolina/protos/issues/1050): optional typed FFI, host-granted trusted provider, no automatic native authority.
- [LIB029 #1052](https://github.com/guillermomolina/protos/issues/1052): independently completed, unchanged R1 acceptance.
- Normative `spec/semantics/FFI.md`, `spec/io/FILESYSTEM.md`, `spec/io/IO_CORE.md`, Actor/Future/Process isolation owners: C1 creates no new ambient rights or transfer rule.
- [D255 #1107](https://github.com/guillermomolina/protos/issues/1107) and [I153 #1068](https://github.com/guillermomolina/protos/issues/1068): RealWorld must eventually use genuine durable SQLite. The previous D255-H1 journal approval and its published evidence remain **historically true**, but its implementation handoff is suspended for the corrected RealWorld objective. C1 is a reusable library decision, **not** an approval of a RealWorld schema or migration.

## 6. Approval/acceptance boundary

`D256_C1_ARCHITECTURAL_DIRECTION_APPROVED=YES`
`D256_EXACT_DURABLE_AUTHORITY_SELECTED=NO`
`D256_SYNC_CANCEL_CONTRACT_FINAL=NO`
`D256_FULL_PUBLIC_SIGNATURES_NORMATIVE=NO`
`LIB029_R1_COMPLETED_UNCHANGED=YES`
`PRODUCT_SOURCE_CHANGED_BY_THIS_RECORD=NO`
`IMPLEMENTATION_OR_TEST_PASS_CLAIMED=NO`
`REALWORLD_SQLITE_DURABILITY_ACCEPTED=NO`

Owner approval is a **decision milestone**, not permission to close D256, implement unresolved host mechanisms, mark D255 complete, or declare RealWorld durable.