# D235 — Core/Prelude Range and InvertedRange, B1-F selected

**State: RATIFIED — OWNER-APPROVED D235-B1-F (2026-11-01).** This is a durable, non-normative record of the selected design. The normative Protos specification in `guillermomolina/protos/spec/` remains the authority for language behavior; this record authorizes a future single implementation slice, not a claim that product code, specification, tests, or Native have already changed.

- Design authority: [D235 #1019](https://github.com/guillermomolina/protos/issues/1019).
- Initial exhaustive investigation: [D235 research](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554452648).
- Focused exhaustive B1-F packet: [D235-B1-F research](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554664153).
- Approval provenance: owner, current conversation, 2026-11-01: **“apruebo B1-F”**, followed by explicit instructions to update issues and publish project-docs evidence. The owner previously corrected the requirement with **“necesito Range, de la misma manera que necesit Map”**, citing a future `for(...) { ... }` sugar and the `Map` / `%{}` precedent. These statements approve B1-F **as described in the focused packet**, not D231 grammar.
- Earlier selected semantics: [D232 #1014](https://github.com/guillermomolina/protos/issues/1014) and [durable D232 ratification](D232_DIRECTIONAL_RANGE_DUAL_INVERSION_SELECTED.md); implementation [I136 #1015](https://github.com/guillermomolina/protos/issues/1015).
- Source revision inspected for selection: [`guillermomolina/protos@c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5`](https://github.com/guillermomolina/protos/commit/c405bb1d2ff582deb94cb003d98b2f01a5bbb3e5). Implementation must re-read its own HEAD.
- Research/approval evidence: [D235-B1-F approval and invariant reconciliation](../../evidence/D235/D235_B1F_OWNER_APPROVAL_AND_COMPATIBILITY_2026_11_01.md).

## Owner-selected result

**Promote both `Range` and `InvertedRange` to canonical, physically shared, FROZEN Core/Prelude ordinary prototype objects, retaining their behavior in distributable Protos source; keep `std:collections/Range` as an ordinary Actor-local module whose two own public members initially reference those exact Core prototypes. No new host-backed Range value family or intrinsic.**

1. **Availability:** ordinary unqualified `Range` and `InvertedRange` are readable via the existing standard prelude with no `import`. They are ordinary nonreserved names. A nearer lexical `Range: custom` shadows the prelude; bare `Range = custom` where only the frozen prelude binding exists fails under existing assignment rules.
2. **Canonical Core identity:** each named standard prototype is an ordinary object delegating directly to `Object`, shared through the Core prelude, frozen (including the reachable standard Closure/captured-context graph) before any guest observes it. No actor-global lexical root, per-Actor clone/proxy, special hidden parent, additional subtype family, custom `call` or runtime-backed range storage is introduced.
3. **Protos-first:** preserve the five own methods `init`, `each`, `eachStep`, `invert`, `reverse` as ordinary Protos Closures, with inherited `Object.call`. Place their canonical implementation under `protos/lib/core/Range.protos` (the same Core source may own the paired `InvertedRange`); the host only loads/publishes/freezes using existing mechanisms.
4. **Standard Library compatibility bridge:** `import("std:collections/Range")` remains an ordinary source-backed Actor-local cached module object, with exactly two **own** public slots `Range` and `InvertedRange`, initially containing the exact corresponding frozen Core identities. Preserve module-instance caching, source loading, cycle/failure/eviction and import-authority semantics. A missing/unavailable module still fails on direct `import`; Core name availability is not a hidden import. Do not redefine duplicate prototypes inside the library module. The already-existing eager standard-module-member installation facility may be used to seed those two own slots before source execution, but its timing and cycle observability must be tested.
5. **Identity under normal state:** in a module/activation without a nearer shadow of `Range`, after ordinary import:
   ```protos
   Ranges: import("std:collections/Range")
   Range === Ranges.Range                  // true
   InvertedRange === Ranges.InvertedRange  // true
   ```
   This is an initial identity guarantee, **not** a permanent equality invariant after arbitrary user mutations: the Actor-local module object retains ordinary mutable slots, so a later `Ranges.Range = other` can replace that export without modifying Core/prelude. Repeated imports in one Actor return that Actor's same module instance under the ordinary cache rules; distinct Actors do not share a mutable module context.
6. **Frozen canonical prototypes:** attempts to replace/create/remove slots on Core `Range` or `InvertedRange` fail through ordinary frozen-object rules. This explicitly changes the formerly mutable module-defined canonical prototype objects. Other ordinary derived objects and locally shadowed bindings follow existing object rules; there is no universal prohibition on user-defined range-like objects.
7. **No incidental semantics changes:** preserve every D232 observable range-instance/algorithm rule below. No generic Iterator, `size`, indexing, `contains`, structural equality, range operator, range literal, new `for` grammar, or optimized Java range kernel is introduced by D235.
8. **Isolation and transfer:** canonical prototype and its Closure graph obey the existing frozen-shared-prelude invariant, Actor pass-by-value delegation-parent traversal, and P projection semantics. No ad-hoc Range-specific transfer exception. Because old module prototypes captured Actor-local module contexts, promotion can change ordinary range-instance transfer outcomes; this is a disclosed consequence of the selected identity/ownership change and is to be tested and recorded, never assumed to work without execution evidence.

## D232 invariant preservation

The change is **placement and ownership**, not a redesign of directed range semantics:

- `Range(start,stop)`: exactly two genuine unbounded Integers; `start <= stop`; ascending `start, start+1, ..., stop-1`; wrong bounds `Error`, equality empty.
- `InvertedRange(start,stop)`: exactly two genuine unbounded Integers; `start >= stop`; descending `start, start-1, ..., stop+1`; wrong bounds `Error`, equality empty.
- Inherited ordinary `Object.call` creates a fresh child whose immediate delegation parent is the corresponding canonical prototype. The `init` guard still requires **exactly** that immediate canonical parent, not an arbitrary descendant. Each successful instance owns exactly `start`/`stop` with original supplied values and is frozen; identity equality/hashing remain ordinary.
- `each(block)` is unit-step; `eachStep(step,block)` requires strictly positive/negative genuine Integer step according to direction, even on empty instances. Invoke callback normally with one Integer per visited member; ignore result; return `null` on normal completion; preserve `Error`, lexical `^`, `ensure`, suspension/cancellation and no implicit Future await; no proportional allocation or host-width truncation.
- `invert` swaps bounds and direction, possibly changing the members; `reverse` shifts opposite-direction bounds by ±1 to preserve the unit-step members in reverse. Both create new frozen instances. No extra methods on the canonical prototypes.
- **Do not assert** that `Ranges.Range(0,10)` and separately extracted `Range(0,10)` are equivalent; D232 records the ordinary extracted-member receiver-binding distinction.

## Explicit GITHUB021 reopened and preserved invariants

| Former authority | Selected delta / decision |
| --- | --- |
| D232: `std:collections/Range`-only public availability | **Explicitly reopened:** additionally publish the two ordinary names in the frozen Core prelude |
| D232: prototypes ordinary and not frozen by contract, currently open/mutable | **Explicitly reopened:** canonical public prototypes must be FROZEN; mutation of them now fails |
| D232: Actor-local module-owned prototypes and captured module context | **Explicitly reopened:** canonical identities/behavior become physically shared Core-owned; module objects remain Actor-local and re-export exact identities |
| D232: two own local module exports, module ordinary and cached | **Preserved via bridge:** exactly two local exports, normal import/cache, different initialization visibility must be tested and specified |
| D232: constructors, strict Integer bounds, half-open traversal, five methods, identity, `invert`/`reverse` | **Preserved without change** |
| Core frozen-shared-prelude invariant, lexical lookup, ordinary object model | **Preserved:** use existing mechanics, no mutable object shared between Actors |
| D231: dual contextual `for(item in expr)` and `for(index,start,stop[,step])` are under investigation | **Unchanged/unselected:** availability may aid eventual lowering, but neither syntax nor choice between canonical-name dispatch and shadowing is approved |
| D136: `%{}` Map literal checks canonical factory and defines initial Map associations | **Unchanged:** Map is a precedent for availability, not an automatic copy of its literal/constructor checks |

**Decision approval:** exact owner phrase “apruebo B1-F” expressly selects all of B1-F as disclosed in [focused packet](https://github.com/guillermomolina/protos/issues/1019#issuecomment-6554664153), including re-opened freeze/module-identity/transfer consequences. **GITHUB021 consistency:** PASS for this bounded selection. New semantic choices discovered during implementation still require a design checkpoint rather than silent inference.

## Implementation and acceptance boundary

Implement B1-F as **one coherent Ixxx slice in `guillermomolina/protos`**, Protos-first. Main expected surfaces: `protos/lib/core/Range.protos`; `protos/lib/core/prelude.protos`; compatible source-backed `protos/lib/collections/Range.protos`; `src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosCoreBootstrap.java` Core load order and existing standard module-member bridge; governing `spec/semantics/VALUES_AND_COLLECTIONS.md` and, if necessary to record changed observable module initialization/prelude behavior, `MODULES.md`; global `spec/PROTOS_SPEC_CHANGELOG.md`; exact bootstrap and Range/module conformance tests plus Actor/P/Native/CLI checks and docs/examples only as directly required.

Do **not** add a new parser construct, Java Range intrinsic, generic iterator or hidden lexical namespace. Preserve D231 as a separate open decision. The implementation agent edits only; the human executor runs targeted checks, full suites when required, version bump/changelog, commit and push under repository `AGENTS.md`.

**Approval does not mean implementation or tests have passed.** This record is not a claim of published changes in `guillermomolina/protos`.
