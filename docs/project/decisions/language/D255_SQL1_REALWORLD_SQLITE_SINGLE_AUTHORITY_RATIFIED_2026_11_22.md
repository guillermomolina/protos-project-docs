# D255-SQL1 — RealWorld persistent SQLite as the single source of truth

**Status: OWNER-RATIFIED DECISION (2026-11-22). Implementation and application acceptance remain outstanding.**

- **Live owner:** [D255 #1107](https://github.com/guillermomolina/protos/issues/1107).
- **Implementation owner:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), in the separate [RealWorld application repository](https://github.com/guillermomolina/protos-realworld-example-app).
- **Explicit approval:** The user was asked, **“¿Apruebas D255-SQL1 con estas decisiones?”**, after a full proposal describing the persistent SQLite architecture, the comparison, the migration and failure boundaries. The user's immediate response on **2026-11-22** was **“si la apruebo”**. This is exact approval of the presented **D255-SQL1** candidate, not generic permission to change unrelated Protos semantics.
- **Approval comparison and GITHUB021:** [D255-SQL1 evidence](../../evidence/D255/D255_SQL1_OWNER_APPROVAL_GITHUB010_GITHUB021_2026_11_22.md).
- **Superseded only for RealWorld:** [D255-H1 historical journal ratification](D255_PROTOS_FIRST_DURABLE_PERSISTENCE_H1_RATIFIED_2026_11_10.md). H1's 2026-11-10 owner approval remains an authentic historical event, but does **not** authorize the I153 persistence implementation after the explicit corrected objective and SQL1 selection.
- **Current Protos product authority:** [`c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5`](https://github.com/guillermomolina/protos/commit/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5), global specification revision `0.1.507` ([SQLite owner](https://github.com/guillermomolina/protos/blob/c2bfd9b3fd4d3873a7530d1b0549cc6e4e9936e5/spec/semantics/SQLITE.md)). [D256 #1118](https://github.com/guillermomolina/protos/issues/1118), [LIB030 #1119](https://github.com/guillermomolina/protos/issues/1119) and [LIB031 #1120](https://github.com/guillermomolina/protos/issues/1120) are already completed.
- **Application inspection baseline:** [RealWorld `56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c). This baseline has memory-only application state; it does **not** contain the SQL1 implementation.

## 1. Selected model and authority boundary

**SQLite is the only authoritative durable and current application data store.** The production RealWorld service must not use `MemoryStore` as a parallel authoritative cache plus asynchronously mirrored SQLite. Do not introduce dual-write reconciliation, a second journal/database engine, a generic `std:storage`, an ORM, a Core storage primitive, application-specific Java, JDBC or a new Truffle/FFI bypass.

RealWorld's existing HTTP, authentication and domain policy remains ordinary Protos. Implement private application repositories using the already published ordinary Protos `std:sqlite` API. Preserve existing public HTTP behavior and the 19 implemented routes. The old `MemoryStore` may remain for standalone legacy/fixture tests, but **the production root service uses SQLite for authoritative reads and writes**; copied result values or request-local snapshots are not a second authority.

The **RootActor** exclusively owns the database Connection, its Statements, credentials and mutable application transaction boundary. The two HTTP Workers retain their existing bounded transferable request/response snapshots; they never receive the Connection, Statement, signing key, verifier collection, Clock/Entropy grants or mutable database authority. Native calls are synchronous and Actor-local, not asynchronous queries or cancellation-safe implicit Futures. RootActor message turns may suspend for unrelated cryptographic operations: always recheck admission immediately before a transaction, and perform no Future wait inside the synchronous SQL mutation/commit interval.

## 2. Approved application data and invariants

Use a **minimal explicitly versioned relational application schema** for:

- `users`: stable numeric account identity, exact unique username/email, public immutable JWT `subject`, retained Argon2id password verifier string, and credential `generation` used for JWT revocation; nullable bio/image as appropriate. A separate private credentials table is an implementation detail only if it preserves transactional one-account invariants and never creates a second authority.
- `articles` and `article_tags`: stable slug ownership by account ID, title/description/body, tags, creation/update Instants represented by an explicitly validated persistent encoding; title/slug uniqueness and existing deterministic slug behavior.
- `comments`: non-reused stable numeric public comment IDs, author and article foreign keys, body and timestamps.
- `follows`: a unique directed pair of stable user IDs; no self-follows.
- `favorites`: a unique user/article pair; repeat favorite/unfavorite remains idempotent.

Persist and check stable subjects, verifiers and generations atomically with the related account state. Retain security after restart: changed-password tokens whose generation is obsolete remain revoked, and existing account/token identity must never accidentally become a different user. Do not persist plaintext passwords, raw signing keys in SQLite, live Protos objects, native pointers, ActorRefs, Closures, Futures or Java objects. RealWorld's external JWT signing key remains supplied by the host independently and must remain stable across restarts when the deployment wants previously issued tokens to remain verifiable.

Avoid redundant independently mutable derived state: favorite counts are obtained or maintained consistently under the same SQL transaction, never updated as an asynchronous side effect. Keep profile updates and stable author/follow/favorite relationships correct across username changes. Schema constraints must enforce uniqueness and referential integrity independently of application-level prechecks.

**No exact SQL DDL, specific environment-variable name, generic repository protocol or schema migration script is ratified by this design record.** Those are bounded implementation choices inside I153, subject to the approved invariants. No automatic import of an H1 application journal is authorized.

## 3. Transaction and failure discipline

- Each admitted logical domain write has one SQLite transaction, with dependent record changes committed or rolled back together. Do not acknowledge successful registration, account/password updates, article/comment changes or relationship changes before the SQL COMMIT has succeeded.
- Password hashing and entropy/JWT issuance may return Futures. Perform such operations **before** the SQL mutation; after any suspension re-read identity, uniqueness, credential generation and author/ownership state before entering the SQL transaction. Never assume a snapshot obtained before a suspension remains valid. Successful token issuance is not a commitment of its identity record.
- Parameterize SQL through `std:sqlite` typed binds; never interpolate request-controlled strings into SQL.
- The SQLite Connection is Actor-local; use `ensure` for all prepared Statement and Connection lifetimes; finalize child Statements before their Connection. Neither close nor a failed COMMIT is proof of durable success.
- A COMMIT error is **not an implicit ROLLBACK**. `inTransaction()` observes the native transaction state. Report no HTTP mutation success on an unconfirmed commit, prevent subsequent unsafe writes until the state has been reconciled, and do not silently retry a commit that might already have taken effect. Any recovery procedure must avoid duplicate writes or guessed rollback.
- Use the selected SQLite engine for locking, atomicity and crash recovery. No secondary Protos journal or application WAL is permitted.
- Explicitly configure and check SQLite database-wide `PRAGMA journal_mode=WAL` and the writer Connection's `PRAGMA synchronous=FULL` (effective numeric value `2`); do not assume new connections inherit the writer's synchronization policy. Native WAL effects include database sidecar files. These settings do not establish a universal physical-power-loss guarantee.
- Set and verify `PRAGMA foreign_keys=ON` on every active Connection **outside any transaction**, before accepting dependent writes. Reject startup or a connection when required settings cannot be established.
- A host explicitly supplies the SQLite filename and external FFM provider and authorizes `sqlite3` under D259. Import itself grants no authority. Trusted native SQLite runs with the OS process's permissions: guest Filesystem grants **do not confine** SQLite native I/O. No Landlock/native sandbox or Native Image availability is claimed.

## 4. Startup, schema versions and migration

A durable production service starts by obtaining the configured DB path, trusted FFM provider, allowed native SQLite library and stable JWT signing-key configuration. The RootActor obtains the Connection, establishes/verifies connection policies, validates the database and schema version, performs an allowed atomic initialization/migration only when explicitly supported, and restores consistency **before creating/advertising a listening HTTP service**. Reopening a known schema does not silently replace or clear existing data.

Use SQLite `PRAGMA user_version` for the **application-owned** schema version. Initialize an empty, explicitly targeted database with a known initial version; reject unknown future versions, invalid relational constraints, corrupt state, unsafe/partial migration or unavailable native authority rather than serving from empty memory. Concrete future migration steps are added only when a new schema actually needs them; the initial design does not prebuild a migration framework.

RootActor controls shutdown and releases Statements before Connection and native session retirement, after it stops accepting and drains admitted requests. A physical SIGKILL is not graceful shutdown: SQLite recovery and post-crash restart, rather than application cleanup callbacks, must be checked.

An in-memory RealWorld deployment has no previously durable data to migrate. Historical work on a branch using an H1 app-owned journal must be **preserved as historical work and tests**, but is not imported, used as a backend or discarded by this ratification. Any future export/import of actual H1 journal data requires a separately justified explicit one-time migration, not an automatic second storage path.

## 5. I153 implementation handoff and acceptance

Work belongs to existing [I153 #1068](https://github.com/guillermomolina/protos/issues/1068) and its separate RealWorld repository. Do not allocate new formal issues or split this decision into invented phases without a governance trigger. A coherent application change should integrate storage, auth, domain methods, startup/shutdown, deployment instructions and tests rather than deliver a new Java wrapper.

Acceptance must demonstrate on real authorized SQLite:

1. The same database reopened by a **new Protos OS process** restores account IDs/subjects, Argon2id verifiers and generations, profiles, articles/tags, comments, follows and favorites, with relationships intact.
2. Real authentication after restart, stable signing key policy and previously revoked JWT invalidation (including a password-change-then-crash/restart case).
3. Atomic account registration/update and domain mutations: rejected/failed operations do not leak partial persistent state; foreign keys, uniqueness and idempotent favorite/follow operations remain correct under two Workers.
4. Startup fail-closed on missing/unauthorized provider, absent or invalid DB path, unknown schema, corrupt DB, unsupported migration, locks or incomplete recovery. **No HTTP readiness marker before successful DB validation**.
5. Real SQLite contention/lock timeout, failures around COMMIT, explicit ROLLBACK, SIGKILL and restart; distinguish process-crash recovery from physical power-loss guarantees. Recovery after uncertain commits must never invent a success or silently duplicate effects.
6. Graceful drain/retire without leaking Statement/Connection ownership or exposing errors/keys/verifiers to Worker logs.
7. The **unmodified official RealWorld Hurl** compatibility gap is tracked independently. Earlier reported 12/13 unmodified versus 13/13 adjusted (157 requests) remain historical, not SQL1 acceptance. Final conformance requires separately pinned official test revision and truthful results; no SQL1 tests have been run as part of this ratification.

No modifications to `guillermomolina/protos`, to the Standard Library or to the RealWorld application are performed by this design publication. No Java is introduced. If a real public SQLite limitation is discovered during implementation, report the exact gap and return to the appropriate decision owner rather than invent a workaround that bypasses the approved generic FFM architecture.

## 6. Supersession and independent status

The 2026-11-10 **D255-H1** ratification remains historical evidence of a different initially selected path. The owner later clarified that I153 had been waiting for *SQLite*, not a self-built journal. The 2026-11-22 explicit approval of **D255-SQL1** replaces H1 **for RealWorld application persistence** and preserves these still-applicable constraints: ordinary Protos policy, RootActor ownership, bounded Worker snapshots, no ambient authority, stable identity and credential revocation, honest durability, and no generic Core/ORM. It deliberately replaces H1's underlying journal/File.sync/lock/replay engine with the already approved `std:sqlite` + trusted D259 FFM/SQLite model. This is an **explicitly reselected owner-approved design**, not an unacknowledged violation of H1.

D255 can close `completed` after this exact durable record is committed and verified. **I153 remains OPEN** until implemented and separately validated. No new specification revision, Maven version increment or product `CHANGELOG.md` edit follows from this non-normative, application-specific decision.

**Publication evidence:** The product authority and RealWorld baselines above are pinned to exact Git revisions. GitHub returns the exact `PROJECT_RECORD_REVISION` after this record is published; it must be re-read at that revision before issue closure.
