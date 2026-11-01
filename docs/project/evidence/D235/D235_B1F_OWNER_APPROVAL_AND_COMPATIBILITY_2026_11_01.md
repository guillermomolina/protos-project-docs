# D235-B1-F — Approval, research evidence and compatibility reconciliation

**Date:** 2026-11-01. **Owner:** [D235 #1019](https://github.com/guillermomolina/protos/issues/1019), repository `guillermomolina/protos`. **Status:** owner explicitly selected **B1-F**, with implementation **NOT** executed as part of this record. **Normative authority:** product `spec/`; this is archival decision evidence only.

## Exact approval provenance

1. Owner originally commissioned [D235 #1019](https://github.com/guillermomolina/protos/issues/1019) to compare Standard Library, Core/Prelude and runtime-intrinsic placements for ordinary first-class `Range`/`InvertedRange`.
2. [Initial GITHUB010 packet](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554452648) recommended explicit library alias while keeping the current D232 module-only semantics; no selection.
3. Owner correction in the active 2026-11-01 discussion: **“no, esa no es la pregunta, necesito Range , de la misma manera que necesit Map”**. The desired `for(...) { ... }` contextual sugar requires baseline first-class `Range` name availability, not merely a user-inserted import. The question is availability and ownership, **not authorization of a particular `for` grammar**.
4. [B1 focused research packet](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554664153) specified **B1-F**: exactly two canonical frozen Core/prelude ordinary prototype objects, both Protos-first, with old importable module retained as ordinary Actor-local compatibility bridge re-exporting those exact identities. This packet explicitly disclosed changes to D232 prototype mutability, module-only placement and per-Actor prototype ownership/transfer.
5. Owner expressly replied **“apruebo  B1-F”**, followed by **“Actualiza issues en guillermomolina/protos y agrega evidencias en guillermomolina/protos-project-docs”** and explicit approval to commit/push documentation in that repository despite AGENTS, 2026-11-01. The owner wants the implementation in a single slice with separately described agent-editor and human-executor responsibilities.
6. **Scope limit:** this approves B1-F exactly as researched. It does **not** approve D231 syntax, import bypasses, Java intrinsics, new Array/Iterator protocols, host-specific performance shortcuts, or new source-level aliases.

## Inspected exact source evidence

Product HEAD inspected: [`c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`](https://github.com/guillermomolina/protos/commit/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5) (2026-11-01). D232 implementation baseline: [`dd4db2e9722df293aac0f3674aa2d88b46841aff`](https://github.com/guillermomolina/protos/commit/dd4db2e9722df293aac0f3674aa2d88b46841aff). No evidence of a changed Range/bootstrap surface in intervening I137; subsequent implementers must verify their own HEAD.

- `protos/lib/collections/Range.protos`: two source-defined ordinary prototypes with five own selectors each, D232 constructor/step/transform contracts; currently these are created by Actor-local module execution.
- `protos/lib/core/Map.protos`: ordinary `Map: {}`. `protos/lib/core/prelude.protos` publishes `Map`/other standard objects, while `ProtosCoreBootstrap.java` source-loads them and freezes shared graph including Closure lexical captures. **Therefore** Core name availability does not imply a new intrinsic Java range family.
- `spec/semantics/VALUES_AND_COLLECTIONS.md` owns D232 ranges and `%{}` Map construction. The Map literal resolves ordinary `Map` then checks canonical factory behavior. Its *construction-time* Map association semantics are distinct from `atPut` and must not be analogized into a new Range control primitive.
- `spec/semantics/MODULES.md`, `EXECUTION_AND_CONTROL.md`: frozen prelude is moduleContext lexical parent; no extra global namespace, and physically shared Protos objects must be frozen/semantically immutable. A nearer `Range:` shadows; bare `=` cannot mutate frozen prelude.
- `spec/semantics/OBJECT_MODEL.md` and `CALLABLES.md`: one immutable delegation parent, direct Object parent unless otherwise specified, ordinary invokable constructor, direct parent guard, reflected slots, Closure captures, receiver/return semantics.
- `spec/concurrency/ACTORS.md`: ordinary object transfer graph includes delegation-parent edge and local slot edges; Closure captures are Actor-local and non-transferable. An old Range instance probably cannot cross Actors (inference only; no Range-specific execution run); the canonical frozen Core parent could make new instances transferable under general rules. No Range-specific transfer exception is selected.
- `spec/concurrency/PARALLEL_EXECUTION.md`: projected P-local Closure uses the standard frozen prelude as lexical parent; canonical `Range` should be resolvable without Actor-local import/capture.
- `src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java` has eager `standardModuleMembers` with frozen values; `ProtosModuleRuntime.java` creates an Actor-local module instance, preinstalls members and caches the module before executing its source. Reuse that general seam to export canonical Core objects; do not add a special loader exception or duplicate prototype.
- `protos/tests/conformance/library/collections/{range-core,module-surface-and-imports,range-callbacks-and-control}.protos`; `src/test/java/com/guillermomolina/protos/execution/integration/modules/ProtosCoreBootstrapTest.java` are the direct existing conformance/bootstrap witnesses. The last test asserts an exact prelude name set that must be deliberately updated.

Additional inspected guidance: `AGENTS.md`, `AGENTS.work/{DESIGN,REFERENCE,COORDINATION,IMPLEMENTATION}.md`, `spec/AGENTS.md`, `protos/AGENTS.md`, docs repo `AGENTS.md`, and [`docs/project/architecture/CORE_BOOTSTRAP_ARCHITECTURE.md`](../../architecture/CORE_BOOTSTRAP_ARCHITECTURE.md).

## GITHUB021 invariant/delta approval checkpoint

| Requirement or former invariant | Result and evidence |
| --- | --- |
| User: Range available like Map, without import | **SELECTED**: two ordinary nonreserved Core prelude names |
| D232 ordinary prototype-backed values and exact five selectors | **PRESERVED**: methods remain Protos Closures, inherited Object.call, immediate parent Object |
| D232 only Standard Library export | **EXPLICITLY REOPENED**: add Core Prelude names while preserving old importable module |
| D232 prototype structure not frozen by contract | **EXPLICITLY REOPENED**: canonical prototypes now FROZEN, standard mutating extensions fail |
| D232 per-Actor module-local prototype identities | **EXPLICITLY REOPENED**: canonical prototypes shared through Core, module remains Actor-local but exports same identities by default |
| D232 frozen instances with own start/stop and identity equality | **PRESERVED** |
| D232 Integer-only, half-open directions, sign-specific eachStep, empty/wrong-order cases | **PRESERVED** |
| D232 ordinary callback, no implicit await, error/ensure/lexical non-local return | **PRESERVED** |
| D232 distinct `invert` and `reverse` | **PRESERVED** |
| Normal module importing, caching, cycles, error/eviction | **PRESERVE**, but module-member visibility is earlier than prior source declarations and must be covered; do not claim no observable change |
| Shared-prelude immutable closure graph; Actor/P isolation | **PRESERVE** by freeze and ordinary transfer/projection, no host custom bypass |
| D231 two complementary for forms | **UNSELECTED**, separate design owner; no parser change is authorized |
| D136 Map literal initial-association semantics | **UNTOUCHED** |

**Decision provenance PASS:** owner explicitly approved fully disclosed B1-F in the active conversation. **Decision invariant consistency PASS for this exact selected contract:** all preserved/reopened items have been surfaced, with no unapproved extra semantics. A later discovery of an unaccounted public incompatibility must return to owner rather than being silently resolved inside Ixxx.

## Adversarial acceptance evidence to obtain in Ixxx, not claimed here

1. Initial `Range === import("std:collections/Range").Range` and similarly InvertedRange; two module own slots; normal cache; module rebind diverges normally from the prelude; no accidental source-level self-shadowing.
2. `Range(0,10)` and `InvertedRange(10,0)` without import; instances' immediate parent and own start/stop; frozen prototypes and instances; prototype user extension attempts fail; local name shadowing and forbidden bare assignment behave ordinarily.
3. Strict two-Integer args, wrong bounds vs equal empty, zero/negative/positive signed step even when empty, huge unbounded arithmetic, invert vs reverse and callback/ensure/non-local ^ behavior. Full D232 suite remains meaningful.
4. Standard module-source availability, startup failure and cache eviction; cyclic import preinitialization frontier; Actor A/B independent moduleContext, shared immutable prototype; P-local lexical visibility; Actor send/request transfer result under existing value-graph rules (document actual outcomes).
5. Both JVM and Native bootstrap/package/REPL/CLI, exact prelude binding-list tests, whitespace/style/static/focal/full tests. PERF045 owns any subsequent measured optimization; no free performance claim from the move.

## Comparative precedents (primary specifications and documentation)

- Python: built-in [range](https://docs.python.org/3/library/stdtypes.html#ranges) provides name availability plus immutable range semantics.
- Ruby: [Range](https://docs.ruby-lang.org/en/master/Range.html) is a Core range concept with separate literal syntax and ordinary block traversal.
- Rust: [core::ops::Range](https://doc.rust-lang.org/core/ops/struct.Range.html) is a half-open Core value, with static-type and ownership semantics not transferable to Protos.
- Kotlin: [ranges and progressions](https://kotlinlang.org/docs/ranges.html) combine named progression values, `downTo`, `step`, and syntax.
- Swift: [Range](https://developer.apple.com/documentation/swift/Range) is a standard half-open abstraction.
- JavaScript: [iterators and generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_generators) show that a language loop can consume ordinary public protocols rather than a special host numeric range object.
- Go: [for statements](https://go.dev/ref/spec#For_statements) show numeric loop syntax without a required first-class range object; it does not meet this owner's additional explicit first-class Range requirement.

For all 12 GITHUB010 scores, confidence levels, alternative B2/B3/C tradeoffs, disqualifying red flags, change costs, examples and exact file evidence see the [focused B1-F research](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554664153) and [original full candidate comparison](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554452648). This record does not replace those primary decision packets.

**No tests, builds, compilation, benchmarks, Native checks, or changes in `guillermomolina/protos` were executed by publication of this evidence.**
