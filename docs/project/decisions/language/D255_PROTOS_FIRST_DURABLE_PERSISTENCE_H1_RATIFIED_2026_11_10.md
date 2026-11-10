# D255-H1 — Protos-first durable persistence by composition

**Status:** OWNER-RATIFIED DESIGN (2026-11-10); **implementation, normative product publication and crash-test acceptance NOT YET DONE**. This is a durable non-normative design record, not an installed Core or Standard Library API.

- Authority: [D255 #1107](https://github.com/guillermomolina/protos/issues/1107), explicitly selected by the owner on **2026-11-10**: **“Apruebo D255-H1”**, after the full H1 decision packet (including the exact protocol, adverse cases, exclusions and escape path).
- Downstream existing implementation owner: [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), RealWorld. Native GitHub edge: I153 blocked by D255 for *this design checkpoint*; closing D255 does **not** complete I153's implementation/acceptance.
- Fixed inspected baselines: [PROTOS_REVISION=`c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5); [RealWorld `56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c). These are **inspection baselines, not an H1 implementation**.
- Supporting [GITHUB010/GITHUB021 and approval evidence](../../evidence/D255/D255_H1_GITHUB010_GITHUB021_APPROVAL_AND_COMPARATIVE_EVIDENCE_2026_11_10.md) and [bounded handoff](../../work/D255/D255_H1_IMPLEMENTATION_HANDOFF_2026_11_10.md).

## 1. Ratified architectural choice and scope

**D255-H1 = E+A**: application-owned, ordinary Protos persistence modules composed over **explicitly host-provisioned confined Filesystem** authority. The initial consumer is the existing RealWorld application's single state-owning RootActor. Persist **versioned domain records** as an append-only journal; require a preprovisioned file capable of the needed standard read/write, seek, size, truncate and durability protocols; append whole application transactions and acknowledge a durable commit only after successful `File.sync()`; recover conservatively and deterministically.

The chosen initial profile is **one effective writer per store, one RootActor custodian, bounded local data, no third-party service**, and no claim of a multi-process database. SQLite, LMDB, PostgreSQL and other backends remain possible later; H1 does not make any of them a dependency and does not repeal the separately ratified optional D238 `std:ffi` architecture.

No `std:storage` general module, universal ORM, Core `Storage`/database primitive, new global store, ambient disk authority, automatic object-graph persistence or magic transaction machinery is approved. A private application interface is not a standardized public protocol merely because it is described here.

## 2. Initial private guest-side protocol (contract selected for the bounded consumer)

Conceptual names from the owner-approved H1 packet (not claims that these functions exist today):

```text
Journal.open(filesystem, journalPath, lockPath, limits) -> Future<Journal>
journal.recover() -> Future<RecoveredState>
journal.appendCommit(transactionId, payloadBytes) -> Future<CommitReceipt>
journal.close() -> Future<Journal>
```

- **Authority/admission:** The caller supplies an already-held `Filesystem`; no paths are resolved outside it and no disk authority is minted by `import`. Open validates the concrete capabilities it needs and fails closed if they are absent. Initial data file and its durable namespace binding are provisioned by the host/deployer; no assumed portable directory `sync`. No hidden creation, arbitrary path discovery or ambient FFI.
- **Ownership and locking:** Journal and open File remain with the RootActor. An exclusive, confined `createNew` lock name prevents a *second cooperative participant* from acquiring the same logical journal simultaneously. A stale lock after crash requires **explicit administrative resolution**, not timestamp/PID-based automatic stealing. This is *not* distributed fencing, a lease protocol, mandatory cross-process OS locking or a promise to exclude uncooperative writers.
- **Codec:** Write complete application transactions in a bounded versioned record encoding with magic, format version, sequence, transaction ID, length, payload and checksum. Validate finite lengths/allocations and strict types before application. Persist stable IDs and explicit domain relationships, never `===` identity, Closures, Contexts, ActorRefs, Futures, Files, native handles, cycles or arbitrary object delegation.
- **Commit:** Prepare/validate an immutable logical transaction first; capture input Bytes independently; serialize; write one complete framed record (possibly multiple physical writes); invoke `sync()`; publish the matching domain-state transition in RootActor and only then send successful HTTP response. The successful receipt denotes a durable record at that receiver's supported `sync` boundary, not durable namespace replacement or general cross-file atomicity.
- **Exclusive sequencing:** Application writes through one journal-owned queue. A `.value()` wait may suspend the RootActor and allow unrelated messages: no other mutation may validate or publish on intermediate state during that pending transaction. This is a **store-level** ordering policy; it adds no global Filesystem lock or same-Actor implicit FIFO.
- **Recovery:** Read and validate records in sequence and apply **only complete confirmed logical records**. Reject interior corruption, invalid schema, unknown versions, quota breaches and inconsistent relationships. Discard a trailing incomplete fragment only when correctness can be proven by the format and recovery protocol; never infer that arbitrary corrupt bytes are safely ignorable. Persistence, lock and replay invariants must be proved by fault injection before acceptance.
- **Uncertainty:** Before any write, cancellation can have no effect. After any physical write starts and before durable confirmation, failure/cancellation/termination may leave an uncertain, partially written journal; do not promise rollback, automatic retry safety or exactly-once HTTP delivery. Failed `sync()` may have partially persisted its frontier; stop new writes until a conservative recovery. A successfully confirmed `sync()` cannot be undone by cancellation of an observer. `close()` does not silently act as `sync()`.
- **Lifecycle:** `close` rejects new work, settles or retires prior admitted operations under the selected commitment rules and releases resources; ordinary Future/Actor/Process termination and resource revocation constraints remain controlling. RootActor retains the File/custody; Workers receive only request/response snapshots.
- **Errors:** Application-private distinctions may include `InvalidRecord`, `SchemaMismatch`, `StoreBusy`, `CapacityExceeded`, `CorruptStore`, `CommitUncertain`, `StorageUnavailable` and `ClosedStore`; these are **not** approved new Core Error prototypes. Operational errors must not be silently converted to `null`/absence or HTTP success.

**Safety qualification:** The storage implementation must not claim to distinguish committed from uncommitted *solely by seeing a checksum-valid complete record* if a crash could have persisted the record before its `sync()` result. Recovery and acknowledgement must be coherent about this unavoidable uncertainty; no promise of zero effects for a failed/aborted partially written append. Define a conservative replay decision for complete tail records and test idempotent domain transaction IDs. Do not make claims beyond the underlying `Syncable` durability boundary or the tested host/platform.

## 3. RealWorld integration and acceptance

RealWorld is already divided into `MemoryStore.protos`, `AuthStore.protos`, `IdentityStore.protos`, `ContentStore.protos`, `FollowStore.protos` and `FavoriteStore.protos`. Adapt the domain mutation boundary to prepare a transaction, durably commit its record, then publish the state. Restore *together* account IDs, username/email uniqueness, Argon2id verifier, JWT subject and revocation generation, article/comment ID counters, articles, authors, follows, favorites and relationship invariants. The RootActor must finish recovery **before opening TCP listening**. A persistent JWT signing key must remain provisioned explicitly; persisted verification generations are necessary for revocation consistency.

The implementation requires reproducible tests: kill mid-write, before/after sync, torn/truncated/corrupt records, schema mismatch, quota/disk full, two Worker interleavings, concurrent Process attempted access, path traversal/symlinks, cancellation and close during commit, revoked Filesystem, restore after process restart, password change and old-token rejection. JVM/Native and OS claims require independent evidence; Linux confinement success is not proof of macOS/Windows support.

I153 remains open until implementation, restart/recovery and conformance requirements are independently met. Its **unmodified official RealWorld Hurl remains 12/13** versus **adapted 13/13, 157 requests** in the existing maintainer-reported evidence: H1 does not close or erase this gap.

## 4. Preserved prior owner-approved invariants (GITHUB021)

| Approved/current invariant | H1 consequence | Result |
| --- | --- | --- |
| D248 / I155 / I156: explicit confined Filesystem; no ambient filesystem or external symlink/mount escape | Open only through supplied authority; preprovisioned namespace binding | PRESERVED |
| FILESYSTEM: separately opened Files are not transactional; `seekToEnd` + write is not atomic cross-File append | One effective cooperative writer and application queue, no cross-file transaction | PRESERVED |
| BYTE_IO / IO_CORE: `sync` only covers receiver durability frontier; `close` does not imply `sync`; irreversible writes and cancellation rules | Sync is an explicit commit barrier; failures retain uncertainty | PRESERVED |
| ACTORS / FUTURES / D251: Actor-local mutable custody, suspension allows interleaving; no implied rollback or ownership transfer | RootActor-owned journal, sequencing across suspension, snapshot-only Workers | PRESERVED |
| PARALLEL_EXECUTION: no transfer of open capabilities to P | P receives no Filesystem/File/Journal resource | PRESERVED |
| MODULES/OBJECT_MODEL: ordinary modules/objects and no implicit global authority | Private Protos module, explicit values/codec | PRESERVED |
| D238-B1-SQL-R1: optional FFI, trusted-in-process, native handles nontransferable, no implicit confinement | H1 imposes no SQLite/FFI prerequisite or revised FFI contract | PRESERVED |
| D255 owner-approved exact H1: E+A, restricted writer, sync, no standard store/ORM, migration path | Contract above matches the approved scope | PRESERVED |

**Delta check:** H1 introduces *only* an application-owned persisted-data protocol and a narrow commitment/locking/recovery discipline. It does not redefine Core `File` or establish new normative Standard Library semantics. No earlier owner-approved invariant has been identified that must be reopened. **DECISION_APPROVAL_PROVENANCE=PASS; DECISION_INVARIANT_CONSISTENCY=PASS.** Exact naming and additional implementation details must not expand semantic promises silently.

## 5. Non-selected work, reversibility and strongest objection

- Not selected: general `std:storage` drivers, ORM, SQL-in-Core, cross-file transactions, multiprocess ACID, automatic lock stealing, networked database prerequisite, native `sqlite3` access under a false Filesystem confinement claim, arbitrary graph/Closure serialization, automatic snapshot/compaction promises.
- **Pay for what is used:** no runtime or conceptual storage substrate for non-storage programs. RealWorld bears journal, codec, fsync and bounded replay costs.
- **Grow as needed:** keep HTTP/domain interfaces independent of byte offsets; version records and provide a future export/migration path. Later SQLite/remote backends require a backend and data migration, not changes to HTTP semantics or domain stable identities. A safely crash-durable compaction protocol or stronger namespace-durability facility requires its own justified work/approval.
- **Strongest objection:** writing safe journal recovery, locking and transaction semantics in Protos repeats difficult solved SQLite engineering. H1 is justified only for a **small, bounded, single-writer** application. If its crash invariants cannot be demonstrated, reject its implementation rather than quietly weakening durability and select an explicitly authorized SQLite alternative through the proper decision process.
- **Tests/build/benchmarks from this ratification: NOT RUN.** No Protos product sources, specs, RealWorld files, release or runtime were modified or validated by the decision itself.

Implementation handoff remains with the existing I153 issue as *application-owned slices*. A separate formal Ixxx is required only if new work acquires independent closure, blockage, scheduling, dependency or multi-publication scope per AGENTS.work/COORDINATION.md; this ratification itself does not automatically create a generalized storage implementation.
