# AUD021-A — Protos-first Path / Filesystem: ownership audit checkpoint and owner-selected research hypothesis

**Date:** 2026-10-11  
**Formal owner:** [AUD021 / guillermomolina/protos#874](https://github.com/guillermomolina/protos/issues/874)  
**Type:** non-normative, immutable investigation evidence / owner-direction checkpoint  
**Product revision inspected:** [guillermomolina/protos@b47be4b698ca27b537898abee89dd6f4c8c49fc3](https://github.com/guillermomolina/protos/commit/b47be4b698ca27b537898abee89dd6f4c8c49fc3)  
**Documentation baseline before this record:** [guillermomolina/protos-project-docs@a610f43930f7c1d7e2491ca3f9edc6455326c8db](https://github.com/guillermomolina/protos-project-docs/commit/a610f43930f7c1d7e2491ca3f9edc6455326c8db)  
**Authorization:** the owner approved **Candidate B only as the preferred hypothesis for the next research slice**, not a definitive architecture or authorization to implement, modify D169/PLAT051, close AUD021, or alter normative semantics. The owner explicitly rejected treating additional allocations or a slower Path constructor as a general veto against Protos-first architecture.  
**Next slice:** `AUD021-B — Falsify and precisely specify the Protos-first Path representation, trusted recognition, transfer and host boundary`; **TIPO=INVESTIGACIÓN**, read-only, no commands/tests/code/publication during the investigation itself.

## 1. Observed state and ownership friction

At the product SHA above:

- [`protos/lib/core/Path.protos`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/protos/lib/core/Path.protos) declares only `Path: {}`; [`ProtosStandardPathProtocol.java`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardPathProtocol.java) installs Java native closures for `relative`, `child`, `==`, `hash`; Java also recognizes the canonical receiver via `instanceof ProtosPathValue`.
- [`ProtosPathValue.java`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/runtime/ProtosPathValue.java) stores a guest prototype and a defensively copied immutable Java `List<String>`; `child` clones the list, equality uses list equality and hash uses list hash. A constructor alone does not validate that each component is a portable normal component; trusted producers and backend checks currently supply different parts of that proof.
- Filesystem Java flow/backend contracts accept `ProtosPathValue` directly: [`ProtosStandardFilesystemProtocol`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/execution/ProtosStandardFilesystemProtocol.java), [`ProtosFilesystemOpenFlow`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/runtime/ProtosFilesystemOpenFlow.java), `ProtosFilesystemNamespaceMutationFlow`, `ProtosFilesystemTreeObservationFlow`. Host backends call `path.components()`.
- Backends: `ProtosEmbeddedFilesystemCustody` resolves through the Context's `TruffleFile` provider and refuses separator ambiguities; `ProtosNioConfinedFilesystemBackend`, `ProtosNioReadOnlyFilesystemBackend` and `ProtosNioReadOnlyTreeFilesystemBackend` rely on authorized-root relative names and `SecureDirectoryStream` / `NOFOLLOW_LINKS`; `ProtosNioCapturedTreeFilesystemBackend` resolves against a captured immutable logical tree. These are distinct host duties and must not be moved into guest Path protocols.
- `ProtosFilesystemOpenOptions` snapshots **local** option slots once and validates canonical Booleans before effects. The open, mutation and entries flows own Future result/error channels, commitment, cancellation, late completion and resource custody. These obligations are not ordinary Path semantics.
- [`ProtosActorValueTransfer`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java) has an explicit `ProtosPathValue` copy/rematerialization branch, and `ProtosParallelRuntime` independently copies Path into P. These branches execute **on transfer**, not when a one-Actor program merely creates or compares a Path.
- `ProtosCoreBootstrap` eagerly loads `Path.protos` and installs the four Java closures whether or not the program later creates a Path. This is a distinct bootstrap cost; do not confuse it with the optional transfer cost.
- `Filesystem.entries(path)` constructs an Array of fresh frozen ordinary `{name, kind}` descriptors from inert host entries. Those descriptors are **not** Path or live File authority. `protos/lib/io/Files.protos` already owns whole-file convenience operations in guest source.

**Observed architecture concern:** guest semantic policy (construct/validate/equal/hash Path) is split across native Java methods while host backend APIs retain a guest-specific carrier. Reducing this friction is a first-class architectural objective **independent of I090/I091 numeric optimizations**.

## 2. Normative baseline and non-negotiable invariants

Relevant current owners: [`spec/io/FILESYSTEM.md`](https://github.com/guillermomolina/protos/blob/b47be4b698ca27b537898abee89dd6f4c8c49fc3/spec/io/FILESYSTEM.md) §§18,20,20.1–20.4; `spec/io/IO_CORE.md`, `BYTE_IO.md`, `PROCESS_IO.md`; `spec/semantics/OBJECT_MODEL.md`, `VALUES_AND_COLLECTIONS.md` §21; `spec/concurrency/ACTORS.md`, `PARALLEL_EXECUTION.md`; ratified [D169/#640](https://github.com/guillermomolina/protos/issues/640), [D170/#641](https://github.com/guillermomolina/protos/issues/641), [PLAT051/#813](https://github.com/guillermomolina/protos/issues/813). Follow `AGENTS.md`, `AGENTS.work/AUDIT.md`, `DESIGN.md` GITHUB010, `COORDINATION.md`, `REFERENCE.md` and path-scoped agent instructions at future actual HEAD.

- Path is **authority-free**, distinct from String and host `java.nio.file.Path`; Filesystem is explicitly provisioned authority; File is an already acquired resource. No ambient authority, upward/rooted path, implicit String coercion or file-URL conversion.
- `Path.relative()` is the empty component sequence; `child(name)` appends one semantic String other than empty, dot or dot-dot. A component containing slash, backslash, colon, drive spelling or other host syntax remains **one semantic component**, not guest pathname syntax; a backend unable to represent it exactly must fail rather than split or normalize it.
- Equality/hash are structural and backend-independent. A distinct but equal Path must satisfy `a == b` and `a !== b`: **Path is not in the closed value-identity set**, which is Number, String, Boolean and null. An internal representation change may not silently grant `===` value identity.
- Path must be immutable and Actor-portable, with correct destination owning prelude/context and no capability transfer. Ordinary user closures must not become transferable to make this work.
- Invalid invocation/options must not enter the backend. No guest-overridable selector/getter or callback may run on the I/O poller/host acquisition worker. Host confinement, symlink/link and concurrent namespace changes must be enforced at actual resource selection/acquisition without TOCTOU or silent fallbacks.
- Exact Future failure channel, effect commitment, file identity, cancellation, late-result release and C-prime execution-domain rules remain unchanged.
- `Path.parent()` is Object delegation reflection, not filesystem parent traversal. Existing conformance covers invalid child names, forged/delegated receiver, equality vs identity and separators-as-data.

## 3. Alternatives and preliminary review

| Candidate | Representation / ownership | Benefit | Adversarial concern |
|---|---|---|---|
| **A — current** | Dedicated Java `ProtosPathValue`, native protocol, typed host APIs | Existing behavior/recognition proven by current source and tests | Semantic logic remains in Java; physical guest carrier leaks into host backends |
| **B — preferred *research hypothesis*** | Source-owned canonical Protos Path object/protocol, minimal trusted no-callback recognition/extraction, inert immutable `PathComponents` request descriptor into host | Maximizes Protos semantic ownership and removes backend dependency on guest prototype/carrier | Must prove unforgeable membership, immutable content, exact identity/reflection, correct cross-domain copy, no new mandatory infrastructure |
| **C — hybrid** | Keep compact private carrier, source-back semantic methods, extract inert boundary descriptor | Lower-risk migration and compact representation while shrinking native semantics | Java remains part of guest representation; actual benefit vs B depends on invariants |
| **D — boundary-only** | Keep native Path and protocols; change host interfaces to inert descriptors | Fastest way to decouple NIO/Truffle backend | Leaves significant Java/Protos semantic ownership friction |

The prior discussion included a **tentative** 12-axis GITHUB010 1–5 comparative matrix, **not** a benchmark, feasibility proof or binding score. Recalculate on exact B/C designs after studying authenticity, closure/prototype transfer and measurable costs. No score total or speculative allocation penalty decides the owner’s weighting.

### Owner correction: preference and pay-as-you-grow

The owner explicitly rejected this proposed *general rule*: “moving Path from Java to Protos is not an improvement if it causes more allocations or permanent infrastructure to get the same behavior.” **Do not reinstate it as an architectural gate.** A greater cost of constructing a Path can be acceptable in this case in exchange for less Java and stronger Protos-first semantic ownership, at the owner's choice after a concrete pro/con comparison.

Investigate separately:

1. **No Path used:** bootstrap/loading/closure/registry overhead. Avoid new mandatory registries, workers, Tasks or path-transfer state.
2. **One Actor using Path:** cost and semantics of `relative`, `child`, equality and hash. This is local Path work, not cross-Actor overhead.
3. **Filesystem invocation:** one trusted no-callback extraction, validation and snapshot only on actual operation; host security/commit remain unchanged.
4. **Actual Actor/P/Process transfer:** optional rematerialization/copy cost only at the boundary. Do not charge a single-Actor program for nonexistent transfers.

Any allocation, startup or PE penalty must be *estimated/verified and presented to the owner*, not treated as an automatic rejection of B. No performance numbers have been collected in this investigation.

## 4. Most important unsolved proof obligations for AUD021-B

1. **Canonical authenticity / no pets:** How is a Path recognized without admitting `Fake: Path {}`, arbitrary forged slot shapes, delegated methods or guest mutation? Compare ordinary deeply frozen source objects with exact structural validation, a minimal unforgeable private tag, and existing trusted family mechanisms. Do not create an unnecessary new public value family.
2. **Source semantics and reflection:** Which exact local slots, prototype, freeze state, `parent()`, slot enumeration, `===`, `==`, `hash`, Map key and error behaviors are preserved? Does exposing a `components` slot change observable semantics? If yes, either avoid it or route an explicit D decision; never silently amend the spec.
3. **Actor/P/Context/Process:** Can an ordinary frozen object with methods only on the shared canonical prototype use the normal object copier without transferring Closures or identity/authority? Explicitly compare existing Path branches and PLAT051. PLAT051 is currently limited to authorized `std:` families; it is not a free Core-Path transfer registry. Distinguish same-Actor, Actor, P, Context and true Process transport, and mark unsupported functionality without guessing.
4. **Inert host boundary:** Define internal `PathComponents` (name, ownership, invariant, immutability, validity and construction contract), boundary exactness, time of extraction, invalid-error Future result, atomic options/path snapshot and capture of two paths for replace, with zero callbacks in host worker. Backends must see no `ProtosPathValue`, `ProtosObjectValue`, guest prototype, prelude or Actor.
5. **Host authority and lifecycle:** Preserve the distinct secure `TruffleFile` and `SecureDirectoryStream`/captured-tree backends; prove no lexical-check-then-insecure-open window; maintain exclusive create and no-follow entry semantics, errors, Future lifecycle and late-result custody.
6. **Pay-as-you-grow/performance:** Show path-absent bootstrap, simple single-Actor Path, actual Filesystem invocation and actual transfer separately. Inventory expected allocations, native-closure count/PE risks and a *future human-run* benchmark plan. Extra local Path allocations are **not a veto**.
7. **Adversarial fallback:** If B cannot preserve all observable properties without a hidden semantic category, explain the minimal necessary trusted mechanism and compare with C. Clearly distinguish “requires a tiny host kernel” from “requires keeping the current Java implementation”.

The strongest challenge to B is **not presumed runtime speed**, but preserving authenticity, exact reflection, and transfer without creating another privileged object universe. This is not yet settled.

## 5. Preliminary retrospective classification (NOT owner-ratified)

| Current mechanism | Preliminary direction |
|---|---|
| Path as distinct non-authoritative value; relative/downward semantics; structural equality/hash | **KEEP** |
| Explicit Filesystem capability and confined host resource acquisition | **KEEP** |
| Snapshot of options, Future/commit/cancel/resource custody | **KEEP** |
| Source-expressible native Path `relative`/`child`/`==`/`hash` | **KEEP BEHAVIOR / MOVE RESPONSIBILITY** (candidate, not yet authorized) |
| Backends accepting guest `ProtosPathValue` rather than inert components | **REMOVE_NOW / RECONSIDER_LATER** (proposed coupling removal, pending architecture gate) |
| Entire `ProtosPathValue` specialized carrier | **KEEP PROVISIONALLY** until B proof; then compare removal against C |
| Extra public pathname syntax, rooted/upward traversal, URL conversion | **ABSENT / RETAIN ABSENCE** under D169 |
| Unconditional global new Core semantic-transfer registry | **NOT JUSTIFIED** by current evidence |

Do not interpret any proposed classification as permission to remove an implemented mechanism or close AUD021.

## 6. Bounded secondary-library triage

- `Encoding` / `ProtosEncodingValue`: native charset/streaming codecs and transaction semantics have genuine host obligations; source wrapper policy merits independent evidence before any move. Existing D167/I060/PLAT031 owners remain.
- `ProtosTextReader`, `ProtosTextWriter`, `ProtosBufferedByteIo`: queues, C-prime, Future/commit/cancel/state machine are not merely Java implementations of optional guest conveniences; blanket porting is unjustified. Existing `std:io` wrappers should be recognized.
- `ProtosBytesValue`: `List<Object>` octets is a possible physical representation efficiency topic, not proof the semantics belong in Java or a reason to widen AUD021.
- String/Array/Map: identify specific duplicated source-expressible protocols case by case; retaining kernel storage and lookup/hash invariants may be required. No new issues justified by file size alone.
- Actor/P: Path-specific rematerialization is a real coupling seam; PLAT051 is relevant but does not authorize arbitrary closures or Core-family transfer.
- Already Protos-owned JSON/TOML/CSV/datetime/URI/SemVer/logging/etc.: no wholesale re-audit without concrete new evidence.

No secondary issue has been allocated, and none blocks the primary B feasibility slice on available evidence.

## 7. Proposed next work, human-executor boundary and gate

**AUD021-B** continues **inside existing AUD021/#874**; do not create a micro-issue. It is an exhaustive **investigation only**; agent reads actual current GitHub HEAD and governing AGENTS/specs/decisions, analyzes and reports, and may coordinate the live Issue under repository policy. **No local command execution, build, test, benchmark, code/spec edits, commits or pushes during the research slice.** The human executor has **no commands** for this slice.

Deliver one closed-form *decision packet*: exact B object representation and minimal trusted machinery; pseudocode/contract (not patch); no-callback recognition proof; immutable components/identity/reflection; Actor/P/Context/Process transfer proof; exact inert descriptor contract through every flow/backend; alternative C comparison; 12-axis GITHUB010 scoring with justifications/confidence; red flags/pay-as-you-grow separate scenarios; falsifying examples; unchanged normative invariants or explicitly scoped D/PLAT amendment; grouped implementation plan and exact suggested tests for a **later authorized Ixxx**, but do not create Ixxx or treat it as authorized now.

**Approval checkpoint:** present the concrete recommendation to the owner. Approval of B *as a research hypothesis* is not approval of B as a final architecture, any normative change, removal, a platform decision, implementation or AUD021 closure. The publication of this document records the earlier owner direction and source checkpoint only; it neither ratifies nor closes work.

## 8. Provenance and validation status

- GitHub source/spec/issue inspection only; no local project execution or tests/benchmarks during this audit checkpoint.
- Source citations above are pinned to the inspected product commit; decisions and next-step authority are in the live issues.
- Evidence is deliberately a **checkpoint**, not a claim to have conclusively proved candidate B or completed all AUD021 closure conditions.
- Documentation-only publication is owner-authorized directly for `guillermomolina/protos-project-docs`. The published commit is recorded externally in the live AUD021 issue after GitHub confirms it.
