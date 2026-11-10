# D255-H1 — Approved decision and bounded RealWorld implementation handoff

**Owner:** [D255 #1107](https://github.com/guillermomolina/protos/issues/1107), design selection **H1 E+A approved 2026-11-10**. The [ratified design](../../decisions/language/D255_PROTOS_FIRST_DURABLE_PERSISTENCE_H1_RATIFIED_2026_11_10.md) and [comparative evidence](../../evidence/D255/D255_H1_GITHUB010_GITHUB021_APPROVAL_AND_COMPARATIVE_EVIDENCE_2026_11_10.md) are the complete governing non-normative decision packet. This file is an implementation *handoff*, not product code or a normative specification.

**Existing execution owner:** [I153 #1068](https://github.com/guillermomolina/protos/issues/1068), `guillermomolina/protos-realworld-example-app`. At inspection [RealWorld revision `56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c`](https://github.com/guillermomolina/protos-realworld-example-app/commit/56b4cb202bfd66a2c24ce327ab6eb89347a2bd3c), [Protos revision `c9da7ff3e6c95f7a88948e875f61d67007fdeaa5`](https://github.com/guillermomolina/protos/commit/c9da7ff3e6c95f7a88948e875f61d67007fdeaa5). No source edits or tests in this decision publication.

## Accepted architecture (do not broaden)

RealWorld-only ordinary Protos modules for `Journal`, versioned domain codec, and RootActor commit/recovery orchestration over *explicit* `filesystem` authority. Preprovisioned durable file; one effective cooperative writer with an exclusive `createNew` lock and manual stale-lock recovery; bounded record sizes/quotas; full framed domain transactions; `File.sync()` before announcing durable success; reject uncertain/failed durable writes and require safe recovery. No default SQLite, no generic Core/Standard Library `Storage`, no ORM, no multiprocess ACID promise, no ambient FFI or filesystem grant.

## Initial implementation slices inside I153

1. **Codec and recovery:** explicit schema version and deterministic encoding of complete changes. Validate record framing, magic/version/length/checksum, truncation, corruption and replay limits. Use stable Integer/String/Bytes/Boolean/null domain values only; never try to persist Protos object graphs. Write isolated deterministic codec fixtures and read/replay cases.
2. **Journal and confinement:** RootActor-owned `Filesystem` and File. Validate concrete `Syncable` and other needed protocols, implement single-writer cooperative exclusion and fail-closed startup. Restrict all paths to already-host-authorized directory and preprovision data file. Serialize commits across suspension points, return a successful receipt only after durable synchronization, and expose uncertainty safely on failure/cancellation/termination.
3. **RealWorld integration:** route domain mutations through prepare → durable record commit → publish; recover before TCP listener creation. Persist account identifiers, Argon2id verifier, subjects and generation, username/email uniqueness, article/comment IDs, content, follows and favorites. Preserve existing Worker/RootActor ownership and no secret/capability transfer.
4. **Adversarial acceptance:** inject SIGKILL and torn/truncated/corrupt tail, disk full/quota, wrong schema, invalid path, stale lock, two-Worker interleaving, second Process, cancellation during commit and close, restart with token revocation, stable IDs and complete relationships. Run both portable and native checks only on platforms genuinely supported. Capture exact test/tool/runtime revisions.
5. **I153 final acceptance:** keep unmodified official Hurl 12/13 vs documented adjusted 13/13 (157 requests) separate until explicitly reconciled; repeat official conformance and clean-checkout acceptance after persistence. D255 resolution alone is not I153 completion.

### Acceptance fences

- An acknowledged transaction must survive crash/restart under the selected supported `Syncable` backend contract.
- No receipt/successful HTTP reply may precede the successful durability frontier.
- Crash-time ambiguous unacknowledged transactions cannot be declared certainly rolled back; recovery and domain transaction IDs must be idempotent.
- A corrupt middle record cannot be silently skipped; an incomplete tail cannot cause an arbitrary earlier confirmed transaction to disappear.
- A second cooperative writer cannot concurrently append; stale lock removal needs explicit operator action.
- No transitive external filesystem authority, privilege expansion, Actor/P resource transfer or automatic `File.sync()` inference.
- If the journal cannot meet the durability claim, **do not downgrade the claim**: return to the owner with the exact falsification evidence, possibly selecting a separately authorized SQLite path.

## Formal-work granularity decision (GITHUB006/GITHUB009/GITHUB015)

The selected architecture is **application-owned** and the immediately required codec, journal and RealWorld migration are not yet independently scheduled deliverables. They can be kept as coherent I153 slices. **No additional formal Ixxx is allocated by this ratification.** If a reusable Protos persistence facility, separate implementation workstream, independent blocker, new native dependency or >=3 distinct publication slices of independent work actually emerges, allocate a new I/LIB issue under AGENTS.work/COORDINATION.md and verify its native parent/dependency relationships before claiming publication. Do not invent a D255-to-I153 parent hierarchy: D255 was explicitly created as a standalone design issue.

I153's existing native `blocked_by` D255 edge can remain as historical design dependency after D255 closes; its remaining work remains `status:ready` until actual implementation begins. I144, LIB028 and LIB029 retain their **independent** FFI/SQLite scope and are not fabricated as I153 blockers.

## Current milestone

```text
D255_H1_OWNER_APPROVAL=PASS
DECISION_DOCUMENT=RATIFIED
COMPARATIVE_RESEARCH=DOCUMENTED
GITHUB021_INVARIANTS=PRESERVED
I153_IMPLEMENTATION_STATUS=OPEN_READY
CODE_CHANGED_BY_D255=NO
TESTS_RUN_BY_D255=NO
REALWORLD_DURABILITY_PROVEN=NO
```

No version bump, source modification or runtime success is implied.
