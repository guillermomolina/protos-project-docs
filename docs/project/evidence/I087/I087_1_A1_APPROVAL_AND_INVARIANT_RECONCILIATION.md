# I087-1 — Owner-approved A1 Java application modules / GITHUB010 / GITHUB021

**Date:** 2026-10-08.  
**Authority:** exact owner response **“apruebo A1”** to the I087-1 full candidate A1 packet in the active conversation. No broader delegation.  
**Design issue:** [PLAT055/#844](https://github.com/guillermomolina/protos/issues/844).  
**Implementation:** [I087/#841](https://github.com/guillermomolina/protos/issues/841).  
**Product baseline:** [`6dab6ecc08c9a2102a388e00908c710a15cf2a4c`](https://github.com/guillermomolina/protos/commit/6dab6ecc08c9a2102a388e00908c710a15cf2a4c); `0.3.298-SNAPSHOT`, normative spec `0.1.451`.  
**Decision:** [PLAT055 canonical record](../../decisions/platform/PLAT055_JAVA_APPLICATION_MODULE_CATALOG.md).

## Investigation and evidence classification

I087-1 was **INVESTIGATION**. No commands, tests, builds or repository mutations by that phase. Public GitHub read-only inspection checked that product HEAD matched the stated baseline (compare `status=identical`, ahead=0, behind=0). The then-current standard embedding bootstrap selected only `ProtosStandardLibraryModuleResolver`, so host-supplied app modules were unavailable by default.

Authoritative sources read: `AGENTS.md`, `AGENTS.work/DESIGN.md`, `AGENTS.work/REFERENCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/COORDINATION.md`, `spec/semantics/MODULES.md`, `spec/concurrency/ACTORS.md`, `spec/io/PROCESS_IO.md`, `spec/PROTOS_SPEC_CHANGELOG.md`; [I087/#841](https://github.com/guillermomolina/protos/issues/841) and [I086/#840](https://github.com/guillermomolina/protos/issues/840).

Runtime proof by source-level call chain:
`ProtosEmbeddedProcess.bootstrap` → `ProtosCoreBootstrap.bootstrap` → one `ProtosModuleRuntime(resolver)`.
`ProtosStandardImportProtocol` uses that runtime for `import`; `ProtosStandardActorProtocol.spawn` uses `resolveModuleKey` before Actor creation and `ProtosActorBootstrap.initialize` uses `loadCanonicalModule` within destination Actor. The destination reads only its own local bootstrap slot. The identical source resolver composition is sufficient for both operations; no second Actor loader is justified. `ProtosModuleSource.fromCharacters` supports pathless host text. `ProtosLanguageContext` already has an entered-Context reference, and bound Process resolver has a single-binding lifecycle (do not overwrite it during `install`).

GraalVM 25.x public documentation supports `Context.initialize(String)`, `Context.enter/leave`, and internal language-side `ContextReference`; it does **not** provide public `Context.setModuleCatalog`, nor guarantee that shared Polyglot bindings are private. A Protos-specific Java bridge is therefore necessary. Static feasibility is supported; actual concurrency/API compatibility remains test-gated.

## GITHUB010 — alternatives and 12 dimensions

1–5 = least to most favorable. H/M/L = high/medium/low confidence. The table records architectural judgment, **not measured timings**.

| Dimension | A1 — in-memory Context catalog | B — authorized physical root |
|---|---|---|
| Correctness and invariants | 5/H; exact identity and no ambient paths | 3/M; confinement must be proven |
| Protos philosophy alignment | 5/H; one host resolver mechanism | 4/M; physical resolver is established |
| Present-need proportionality | 5/M; only explicit modules cost | 3/M; adds physical policy |
| Incremental growth | 5/M; add later backends via resolver | 4/M; can extend physical model |
| Future-option resilience | 5/M; format/OS neutral | 3/M; physical coupling |
| Scalability | 4/M; immutable Context snapshots | 4/M; filesystem caching scales |
| Conceptual simplicity | 5/M; exact specifier → characters | 3/M; canonicalization and symlinks |
| Portability/freedom | 5/H; strings and language Context | 3/M; host filesystem differences |
| Runtime/resources | 4/M; map lookup, lazy compilation | 3/L; varying I/O overhead |
| Failure/operability | 5/M; bounded config failure | 3/M; provider/race/path failures |
| Deferral/reversibility | 5/M; B is additive | 3/M; public path model is sticky |
| Evidence/risk | 4/M; documented Polyglot primitives, bridge untested | 4/M; mature file providers, security unproved |

Other alternatives considered: C custom Truffle FileSystem — viable but disproportionate today; D Polyglot bindings as private registry — not private; E Engine/global mutable registry — rejected for Context isolation; F do nothing — cannot satisfy I087. A1 costs only users who install a catalog; extending later to B/C is a bounded new resolver, not a new semantic universe.

Comparative precedent survey considered GraalJS, GraalPy, TruffleRuby, Espresso, Sulong/LLVM, SimpleLanguage and Apple Pkl, as well as non-Truffle host file/resource approaches. Do not infer tested equivalence or broad behavioral parity from that survey. Evidence confidence is lower for untested performance, portability and individual runtime internals.

## GITHUB021 — exact approval/invariant preservation

The owner's approval is precisely **A1**, including Java `ProtosEmbeddedModules.install(Context, Map<String,String>)`, exact `app:` namespace, immutable snapshot, single pre-bootstrap installation, Java config errors, no additional guest authority, std/foreign key separation, normal canonical import and Actor.spawn, same-Context compatibility and PAY AS YOU GROW. No separate `protos.AppRoot` or hot registry was approved.

Approved prior invariants: PLAT054 lazy same-Context Process, ordinary read-only host bindings and Core precedence; MODULES resolver-supplied canonical ModuleKey and Actor-local lifecycle; ACTORS §8 creator resolution and distinct destination module instance with own bootstrap binding; PROCESS_IO effective host authority only; foreign key exclusivity; no unnecessary universal concurrency machinery. Each is **PRESERVED** by A1. The new Java API, `app:` naming, single installation and its failure/timing policy are **new consequences explicitly surfaced and approved**, not implicit changes. No recorded owner-approved invariant is contradicted. **GITHUB021: PASS within exact A1 scope.** A further change must be surfaced for owner approval.

## Pending acceptance (not claimed)

Only after decision publication: one coherent product I087-2 implementation in `guillermomolina/protos`, human-executed tests and Git publication. Must prove Java installation/Context lifecycle, race freedom, bad/late installation, ordinary import cycles/cache, canonical Actor destination bootstrap and independent moduleContext, `std:` isolation, two Contexts on shared Engine, denied guest file/socket authority, standalone host-eval independence, no-catalog PAY AS YOU GROW, portable-JAR integration and Native Image implications. I086 is closed independently; reopening it is not a prerequisite.

```text
I087_1=INVESTIGATION_COMPLETE
OWNER_EXACT_A1_APPROVAL=YES_2026_10_08
PLAT055=RATIFIED_SUBJECT_TO_DURABLE_RECHECK
PRODUCT_REVISION=6dab6ecc08c9a2102a388e00908c710a15cf2a4c
NORMATIVE_SPEC=0.1.451
GITHUB021=PASS_A1_SCOPE
CODE_CHANGE=NO
TESTS=NOT_RUN
NEXT=I087-2_IMPLEMENTATION_HUMAN_EXECUTOR
```
