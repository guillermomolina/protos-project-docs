# D208 — Runtime metadata without executionMode: Candidate A

**Status:** RATIFIED — 2026-10-19 owner answer **“ratifico ambas decisiones”**, expressly referring to presented **PLAT067-B and D208-A**.
**Decision Issue:** [D208/#934](https://github.com/guillermomolina/protos/issues/934).
**Architectural companion:** [PLAT067-B](../platform/PLAT067_UNIFIED_PROOF_OPTIMIZED_BYTECODE_DSL_WITH_LAZY_TOOLING.md), [PLAT067/#933](https://github.com/guillermomolina/protos/issues/933).
**Implementation:** [I116/#935](https://github.com/guillermomolina/protos/issues/935) (product changes not yet implemented/tested/published).
**Product baseline for research:** [guillermomolina/protos@27cb31d20e517d8377e58963ea595e5ec16bfd15](https://github.com/guillermomolina/protos/commit/27cb31d20e517d8377e58963ea595e5ec16bfd15), read-only evidence, not a checkout instruction.
**Prior authority:** [D205/#922](https://github.com/guillermomolina/protos/issues/922), [D205 B2+ original ratification](D205_LIB022_RUNTIME_INTROSPECTION_B2_PLUS_PUBLIC_CONTRACT.md); normative `spec/semantics/MODULES.md` §73 in the product repository **still contains the pre-D208 five-slot contract until I116 publishes the amendment**.
**Research/evidence:** [PLAT067 original source and peer research](../../evidence/PLAT067/PLAT067_ONE_STREAM_RESEARCH_2026_10_19.md).
**Role:** durable non-normative public-contract design record. This record **does not directly change normative `spec/`** and is not evidence that the product already conforms.

## Exact selection and rationale

D205 B2+ ratified `std:runtime.info()` with exactly **five** String own slots: `implementation`, `implementationVersion`, `engineVersion`, `platform`, and `executionMode` (`NORMAL`/`DEBUG`). Its `protos.Debug` language option selected an immutable Context compilation mode. PLAT067-B has now separately removed the **underlying conceptual distinction** from the selected architecture.

The owner was explicitly offered:

- **A — RECOMMENDED and NOW RATIFIED:** delete `executionMode` from the public `std:runtime.info()` result and retire `protos.Debug` language execution-profile option; keep DAP as an independent host tooling facility; do **not** fabricate a constant `NORMAL`.
- B — compatibility string or changed/constant mode value, misleading under one-stream semantics.
- C — a new host/instrumentation state field, which changes schema and creates conceptual coupling.
- D — retain the old mode distinction, conflicting with approved PLAT067 goal.

After both decisions were presented, the owner answered exactly **“ratifico ambas decisiones”** (2026-10-19). This selects **D208 Candidate A**. It does not select B/C/D, any additional metadata property, an unrelated guest host capability, or a new authority.

## Ratified guest-visible contract (new normative target)

`import("std:runtime").info()` / `std:runtime.info/0` remains a **zero-argument** operation and returns a **fresh, frozen, ordinary Protos Object** delegating to `Object`. The returned object has exactly **four own slots**, all semantic `String` values:

| Own slot | Unchanged value contract |
|---|---|
| `implementation` | Exactly `"Protos"` |
| `implementationVersion` | Distribution/package version if available by the existing precedence, otherwise `"development"`; same identity as CLI `--version` |
| `engineVersion` | Nonempty, opaque version of the **entered** Polyglot Engine; do not assume SemVer |
| `platform` | Exactly `"JVM"` or `"NATIVE_IMAGE"`, based on the actual runtime and not JIT state |

**There is no `executionMode` member.** Do not return host-null, a fake `"NORMAL"`, or a `"DEBUG"` alias. Do not add a `debuggerAvailable` or `instrumentationState` field. The object is still exactly four-slot, ordinary, fresh and frozen. The module retains existing context-entry observation timing, detached guest String values, authority checks, errors and separation between Engine, Context, Process, Actor and Task. Values are read on demand from the currently entered Context; nothing is eagerly observed on import.

**Retain every other D205 B2+ contract unchanged**: exactly four public `std:inspect` operations, implementation-specific read-only snapshots, `protos.Inspect` explicit opt-in host authority default false, independent of debugger, no implicit source/heap/compiler/foreign authority, fresh/detached diagnostic results, no debug-mode prerequisites. This is a **narrow prospective amendment** of the single `executionMode` member only.

## External host/tool compatibility selection

- Retire the **experimental** language option `protos.Debug` used to select `ProtosExecutionMode` or physical compilation stream; there is no semantic DEBUG Context. Document the retired option explicitly and ensure it does not silently select another code path.
- Preserve `ProtosPolyglotRuntimeHost.openDebug(OutputStream)` and CLI/VS Code debugger launch/readiness. Graal DAP remains a real host **Engine tool** with proper loopback endpoint, suspend/wait and `PROTOS_DEBUG_READY` protocol, independent of guest compilation mode.
- If the public Java method `ProtosPolyglotRuntimeHost.isDebugMode()` is retained for compatibility, its implementation must truthfully report **whether the host DAP service is configured** (or be deprecated with a clear truthful replacement), **not** an immutable guest mode. Do not expose new guest-visible observer state. This choice is a bounded API adaptation of the now-selected “host DAP, not guest DEBUG” distinction, not a mandate to introduce a second host abstraction.
- Keep `protos.Inspect` independent and fixed per Context; DAP must not automatically authorize `std:inspect`.
- Compatibility impact: code explicitly reading `executionMode` or passing `protos.Debug` must be migrated; this is an **intentional breaking schema/option change**, not silently backward-compatible behavior. A user opting into DAP retains host DAP service, subject to PLAT067 optimized-site stepping limitations.

## GITHUB021 — superseded vs retained D205 invariants

| D205 component | D208-A result |
|---|---|
| `std:runtime.info/0` public arity, fresh ordinary frozen result | **Unchanged** |
| `implementation`, `implementationVersion`, `engineVersion`, `platform` | **Unchanged** |
| Exactly five own String slots | **Superseded:** exactly four own String slots |
| `executionMode = NORMAL|DEBUG` from entered Context | **Superseded:** removed, no fake status replacement |
| `protos.Debug` guest physical-profile selector | **Retired** as architectural + external experimental option; DAP host service preserved |
| `std:inspect` public operations and `protos.Inspect` authority | **Unchanged**, no implicit DAP/DEBUG coupling |
| Cross-Context isolation, lazy metadata lookup and lack of privileged guest reflection | **Unchanged** |

D205's ratified original record and earlier `spec/` revisions remain historical evidence and must not be mechanically overwritten as if the old release always returned four slots. The new prospective normative source revision must explicitly record the D208 change.

## Exact I116 migration acceptance

Affected code/contract owners in `guillermomolina/protos`:

- `spec/semantics/MODULES.md` §73: remove one row and five-slot wording; precisely document **four** own slots, zero-argument `info`, fresh frozen ordinary Object and unchanged authority.
- `spec/PROTOS_SPEC_CHANGELOG.md`: add an accurate, new-version normative change entry; never alter historic changelog entries as if D205 had not existed.
- `protos/lib/runtime.protos` and `docs/guide/library/runtime.md`: update API narrative/examples without inventing another field.
- `src/main/java/com/guillermomolina/protos/execution/facilities/ProtosRuntimeMetadataFacility.java`: remove mode observation, any validation dependent on it and own-slot creation; preserve other metadata and failures.
- `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosLanguage.java`: remove the experimental `@Option(name="Debug")` and all mode consumption; preserve `Inspect` and `CoreRoot`.
- `ProtosPolyglotRuntimeHost` and host session/CLI: keep actual DAP service but no `protos.Debug` Engine option.
- `ProtosRuntimeMetadataEmbeddingTest`, library/embedding/CLI/DAP tests: assert exactly **four** fields, no `executionMode`, distinct Engine versions/Contexts, JDK/native platform value, `std:inspect` authority isolation, valid DAP behavior. No removal of tests without equivalent acceptance.
- Check downstream docs/spec references to literal `executionMode` on actual local HEAD; update maintained current-contract references, **not historical immutable records**.

**Implementation workflow:** Agent edits product source/tests; human runs validation and handles version/changelog and Git publication after tests pass, per product `AGENTS.md` and I116. The product repository's normative specification is **not changed by this project-docs ratification commit**. Stop rather than selecting a new semantic option if an unapproved API requirement appears.

**Decision status:** `OWNER_RATIFICATION=PASS`; `D208_A=RATIFIED`; `PLAT067_B=RATIFIED`; `PUBLIC_BREAKING_CHANGE=INTENTIONAL`; `PRODUCT_SPEC_CHANGE=PENDING_I116`; `PRODUCT_CODE_CHANGE=PENDING_I116`; `TESTS=NOT_RUN`; `RELEASED=NO`.
