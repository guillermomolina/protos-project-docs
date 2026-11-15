# PLAT075 — Protos-first module import policy and Actor-local lifecycle

**Status:** RATIFIED — owner-approved Protos-first, 2026-11-15.
**Decision owner and approval:** the Protos owner explicitly stated **"apruebo protos-first"** on 2026-11-15, after the corrected adversarial tests and the reported six-class 82-test regression run.
**Live issue:** [PLAT075 #1124](https://github.com/guillermomolina/protos/issues/1124).
**Evidence:** [PLAT075 local feasibility and owner approval](../../evidence/PLAT075/PLAT075_PROTOS_FIRST_FEASIBILITY_AND_OWNER_APPROVAL_2026_11_15.md).
**Normative owner:** [spec/semantics/MODULES.md](https://github.com/guillermomolina/protos/blob/main/spec/semantics/MODULES.md). This decision changes **implementation architecture only**; it does not redefine observable module semantics.

**Public product inspection baseline:** [guillermomolina/protos@`5b5580ff49e2e3a379c8eb0a63d2906d98a43800`](https://github.com/guillermomolina/protos/commit/5b5580ff49e2e3a379c8eb0a63d2906d98a43800). The source-backed prototype and successful tests are **owner-reported results from an uncommitted local working tree**; they are not claimed to exist at this revision.

## Approved architecture

The standard public `import.call` must be **source-backed Protos**, using `protos/lib/core/import.protos`, rather than keeping the Java `StandardImportBody` as its production policy owner.

Its policy sequence is: admit an exact semantic String; request strict canonical `ModuleKey` resolution from the private bridge; perform an authorized Actor-cache lookup; on READY or INITIALIZING hit return the identical instance; on miss `begin` and then `execute` the exact private attempt through the existing nested C-prime module-initialization path; return the module instance rather than the body's result.

Java/Truffle retains **mechanisms**, not the guest-facing policy: host-defined canonical resolution, validation and authority before observing cache state, private per-Actor `ModuleRecord` storage, D242 MODULE/SOURCE contexts/public objects, FOREIGN provider acquisition and Actor-local facade, compilation and nested guest-root execution, exact `finish`/`fail`/`removeIfSame` finalization and host-error conversion. No guest-writable global registry or authority is introduced.

The private bootstrap bridge has `resolve`, `lookup`, `begin` and `execute` operations. Its opaque, frozen tokens are scoped to the runtime, Actor state, execution domain and Task, and are single-use. **`begin` creates no cached INITIALIZING record**. On the exact selected structured `execute` implementation, the token is claimed; cache/authority is rechecked; only then the existing shared preparation inserts INITIALIZING **before** source-body execution or FOREIGN acquisition. This prevents an unclaimed/cancelled reservation from poisoning the cache.

The experimental `ProtosPrelude` has a private, per-instance, one-time-bound `ProtosModuleRuntime` association. The production implementation may retain that internal association for PLAT051's `loadCanonicalModuleThroughPreludeForRuntime` without adding a public prelude slot or inspecting source Closure capture state.

## Existing authority, invariant reconciliation, and narrow supersession

[PLAT025 Candidate A′](PLAT025_BYTECODE_MODULE_INITIALIZATION_COMPOSITION_BOUNDARY.md) continues to own the **Bytecode module-child and C-prime lifecycle**: recognize a canonical exact implementation **after** ordinary lookup and callable selection; use the existing `ModuleInitializationCall`/`PreparedModuleInitialization`, `RunModuleInitialization` and `FailModuleInitializationUnlessCompleted`; preserve `ContinuationResult` as the only continuation authority; never add a generic native-to-guest request ABI, child Task, blocking carrier or AST/replay fallback.

**Precisely superseded PLAT025 physical choice:** its selected canonical standard `import` wrapper was physically native and its Candidate G guest policy was deferred. PLAT075 replaces **only the public import policy's physical owner** with source-backed `import.call`; the post-lookup intrinsic recognizes the exact private **bridge `execute` body identity**, not a selector name or an arbitrary source-backed Closure. Module-child execution, cancellation and exact record cleanup remain PLAT025 mechanisms. Do not keep a native policy fallback after production cutover.

| Previously approved invariant | Resolution |
| --- | --- |
| D242 MODULE private initializer/public object vs SOURCE context | Preserved, shared allocation path |
| Canonical `ModuleKey`, exact String and resolver contract | Preserved; guest policy requests private resolver |
| Host authority before any cache observation | Preserved in bridge, including hits |
| Actor-local cache, READY/INITIALIZING cycle identity | Preserved |
| Cache-before-execute, FOREIGN façade, exact failure eviction/retry | Preserved; insertion moves from `begin` to claimed `execute`, before execution |
| PLAT014/025 single C-prime continuation and one child root | Preserved |
| PLAT017 specialized implementation only after selection | Preserved; exact bridge `execute` recognition |
| PLAT025 native public import policy | **Prospectively superseded by explicit owner-approved Protos-first** |
| PLAT051 private Prelude runtime lookup | Preserved by private Prelude-runtime binding |
| RootActor initial module semantics | Preserved; may use the same shared internal initialization machinery |
| No shared mutable cross-Actor module state or public cache registry | Preserved |

There is no approved observable semantic change. New conflicting semantics must return to their normative decision owner.

## Considered design space and precedents

This is the previously deferred PLAT025 **Candidate G** refined into the validated **source-backed policy plus private, structured C-prime execution**. Alternatives remain: (A) status quo Java native policy, (B) Protos-first policy with narrow private bridge (**selected**), (C) generic native→guest request protocol, (D) child Task/await, (E) blocked carrier, (F) AST/replay fallback, and (G) a public guest-writable module registry. The rejected options introduce competing continuation authority, scheduling/authority surface, or stale parallel policy, while the status quo fails the owner's Protos-first objective.

The architectural evidence includes GraalPy `importlib` cache-before-execute and failure eviction, GraalJS `CyclicModuleRecord` lifecycle ownership, TruffleRuby loader-specific child execution, Apple Pkl's module cache/initialization split, and Espresso's separation of guest continuation from host frames. These precedents are **mechanism evidence**, not permission to copy other languages' public semantics, locks or async evaluation graphs.

**GITHUB010 qualitative comparison** (5 favorable, 1 unfavorable; H = high and M = medium confidence; no unmeasured runtime result treated as fact):

| Dimension | Native Java policy | Protos-first selected |
| --- | --- | --- |
| Correctness/invariant preservation | 5/H, established | 5/M, 82-test local proof |
| Protos alignment | 2/H, policy in Java | 5/H, guest policy |
| Present-need proportionality | 4/H, existing | 4/M, small bridge reused |
| Incremental growth | 3/M, native changes | 5/M, guest policy evolves |
| Future-option resilience | 3/M, native coupling | 4/M, mechanism split |
| Scalability | 4/M, Actor-local | 4/M, same Actor-local state |
| Conceptual simplicity | 3/M, wrapper mixes concerns | 4/M, named boundaries |
| Portability | 3/M, host policy | 4/M, host primitives only |
| Runtime/resource cost | 5/M, likely fewer calls | 3/LOW, no benchmark |
| Failure/operability | 5/H, previous coverage | 4/M, adversarial proof |
| Deferral/migration reversibility | 2/M, delayed guest extraction | 4/M, prototype ready |
| Evidence maturity/implementation risk | 5/H, shipped path | 4/M, functional proof only |

The owner expressly accepted that Protos-backed policy may be slower and **does not require benchmarking** as a selection condition. Performance claims, Native Image, partial evaluation, startup and production adoption are **not established** by the POC. Not requiring speed parity does not claim Native Image correctness or waive separately applicable distribution gates.

## Implementation handoff

**Use the existing tested local prototype as the implementation base. Do not rewrite it.** Allocate independent I-family work for production default, removing the `sourceBackedPolicy`/test-only bootstrap switch and now-redundant `StandardImportBody` policy, while preserving shared private bridge, Actor caches, PLAT051 association and PLAT025 C-prime lifecycle. Existing root-Actor initial-module support is retained unless an independently required migration is proven. Keep tests, remove only obsolete duplicate policy and verify no native fallback.

The owner executes product builds/tests/Git; this ratification makes **no** product commit and does not claim the experiment was already published or that Native Image passed.
