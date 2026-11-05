# D240 — Explicit logical library interfaces, C staged incrementally from B

**Status:** OWNER-APPROVED ARCHITECTURAL DIRECTION, durably recorded 2026-11-05; **no new Core or Standard Library semantics have been implemented by this record**.
**Governing issue:** [D240 #1057](https://github.com/guillermomolina/protos/issues/1057).
**Exact product baseline reviewed:** [`guillermomolina/protos@384966eec789831bdc954f44d3d17e891fd1cb19`](https://github.com/guillermomolina/protos/commit/384966eec789831bdc954f44d3d17e891fd1cb19), 2026-11-05, `0.3.446-SNAPSHOT`, specification revision `0.1.493`; D239-C1/I145-A already incorporated.
**Selection provenance:** After the exhaustive D240 topology and INC004 export investigation, the maintainer rejected ambiguity between libraries, prototypes, modules and files. The agent then presented an adversarial GITHUB010 comparison concluding: **“adopt C, built incrementally over B”**, with explicit limitations on mandatory whole-library imports, hidden/private exports, foreign authority, live bindings and changing prototype parents. On 2026-11-05 the maintainer answered **“ok apruebo esta recomendación”** and requested coordinated issues and durable project evidence. This is approval of the **disclosed bounded architecture**; not implicit approval of unexamined exact export syntax/semantics, breaking migrations or implementation.
**Supporting evidence:** [D240 falsification and GITHUB010/GITHUB021 comparison](../../evidence/D240/D240_ADVERSARIAL_MODULE_LIBRARY_EXPORT_REVIEW_2026_11_05.md).
**Related Incubator:** [INC004 #1010](https://github.com/guillermomolina/protos/issues/1010), “Module access declarations”.
**Separate unresolved decision:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060), exact named module-export and public-route semantics, unselected.
**First implementation owner:** [I146 #1061](https://github.com/guillermomolina/protos/issues/1061), opt-in additive `std:datetime` aggregator only.
**Authority:** Durable non-normative decision record. Observable language/stdlib behavior remains owned by the exact `guillermomolina/protos/spec/` content until separately approved and published implementation/specification changes. GitHub Issues remain live coordination.

## 1. Why the decision changed

The original D240 compared two Standard Library topologies:
- **module-as-prototype** (`import("std:datetime/Date")` returns a module instance that also is the Date constructor/prototype; Date objects immediately delegate to it);
- **module-exported-prototypes** (`import("std:collections/Range")` has named `Range` and `InvertedRange` bindings referencing the canonical frozen Core/Prelude prototypes).

The conversation exposed a more fundamental problem: a consumer should not need to guess whether an import points to one class, one family, many prototypes or an arbitrary foreign value. A module, library and physical source file are different concepts.

Research **falsified one premise**: `import()` already always returns a Protos module instance under `MODULES.md`. In source-backed cases the instance IS the executing `moduleContext`; for foreign imports it is an Actor-local Protos facade over the provider target. No change to the return kind is needed or selected.

Research also found that package `[exports]` already maps public logical module paths to private package module paths. This is **not** a mechanism for exporting named values from within one module and does not apply to current `std:` path resolution. The proposal must not duplicate package tooling or falsely claim slot-level privacy.

## 2. Selected architecture (C) and mandatory constraints

> A logical library may publish **one or several** publicly importable Protos modules. Each public module should have a deliberate, named public API of Protos objects, prototypes, Closures and/or explicitly admitted foreign values. Source files, package layout and foreign implementation units are *not* automatically public library interfaces. `import(specifier)` always returns the requested Actor-local Protos module instance, never an arbitrarily selected class or module result.

This is an architectural policy for future library design and approved **direction** of staged migration, not a novel Core `Library` value kind or an immediate change in existing resolution behavior.

**Exact selected boundaries:**

1. **Module identity:** preserve canonical `ModuleKey`, Actor-local instance identity, eager module imports and same-Actor cache identity, initialization-before-ready cycle behavior, failed-init eviction/retry. Never replace the cached module with a completed, separate namespace after execution.
2. **Multiple public modules allowed:** e.g. a public `std:datetime` aggregator is useful, but a large library may also provide explicitly public granular submodules. Do **not** require one root module to load everything in every program.
3. **Source file is not semantic identity:** a library public module may be backed by one file importing many source modules; package `[exports]` already separates public module paths from internal module names. An import of `./helper.protos` still names an ordinary module, not a special raw-file return kind.
4. **Named export intent:** public names should be explicit; neither the filename nor every top-level internal implementation slot should implicitly become a promised public library API. **How** that is represented/enforced is unselected and belongs to D242.
5. **Naming:** prefer lowercase logical library names for *newly introduced* APIs (e.g. `std:datetime`); preserve normal published capitalization of prototype/value names (`Date`, `Time`, `Range`) and the historical existing `std:datetime/Date` entry points unless a separately authorized migration says otherwise.
6. **Interop follows the same Protos module result:** a Java or C provider may supply named public values only through an explicitly admitted authority-bound surface. Never auto-discover everything in a Java module/JAR/package, native library or C symbol table. `import("java:java.base/java.time")` and `import("c:sqlite3")` are *examples of a possible future logical API*, NOT working or approved scheme targets.
7. **No second object model:** ordinary objects, slots, delegation, receiver binding, `call`/`construct`, reflection and frozen Core/Prelude identities remain authoritative. No new Core `Library` class, implicit default export or module-wide callable magic.
8. **Preserve currently observable prototype families:** avoid silently changing `Date.parent()`, `Date.recognizes`, equality/hash, Range identities, Actor transfer/ownership or `Actor.spawn` bootstrap selectors.

## 3. Incremental implementation strategy: B first, C only after exact decision

The first authorized design implementation direction uses **ordinary composition**, with no module semantic change:

```protos
datetime: import("std:datetime")
Date: datetime.Date
Time: datetime.Time
Duration: datetime.Duration
today: Date(2026, 11, 5)
```

The spelling `import("std:datetime")` is a **future API** and is not present at the exact investigated product revision. In B it would be implemented by a new opt-in `protos/lib/datetime.protos` module with ordinary bindings to the **same objects** already imported as `std:datetime/Date`, `std:datetime/Time` etc. Merely using the old granular imports must not initialize the new aggregate.

**Deliberate limitation:** `datetime.Date(2026, 11, 5)` is not promised. In Protos, `obj.member(...)` is a method send whose selected member must be a Closure; the `Date` export here is an ordinary prototype-valued slot. Use `Date: datetime.Date; Date(...)` or independently design Closure-valued constructors if truly needed. No implicit conversion from exported prototypes into callable member methods.

The additive module does not provide slot-level privacy and may eagerly initialize its referenced submodules when explicitly imported. The optional nature of the import is the bounded pay-as-you-grow answer for now.

I146 owns only this `std:datetime` additive step. Exact `std:collections` publication needs special care: `Array` in Core versus `std:collections/Array` algorithms; shared `Range` identities, ordinary `Set` factory semantics, and method/prototype naming cannot be blindly collapsed. No mass rename or breaking migration is approved.

## 4. Exact D242 deferred decision boundary

An **independent, still OPEN/UNSELECTED** GITHUB010/GITHUB021 decision must specify the smallest useful semantic contract for:
- a module's explicit named **value exports** distinct from package `[exports]` module-path exports;
- whether the module's execution context is itself the public surface or private lexical state is projected through another *ordinary* object; the implications for `parent()`, `this`, `super`, `methodHome`, lookup, assignment, `slotNames`, `hasSlot`, `slotValue` and freezing;
- whether exports are snapshots, binding cells, aliases or forwarding Closures, including mutability, initialization cycles, failure and error precedence; no live-binding/getter/lazy-export feature is approved by D240;
- public vs internal standard-library module paths and compatibility with package and direct-file resolvers;
- strict foreign-provider catalogs, authority boundaries and provenance rather than automatic host type/symbol enumeration;
- Actor bootstrap own-slot selection, interoperability with foreign facade identities, documentation extraction and source navigation.

D242 must make and obtain a **new exact owner-approved semantic choice** before implementing any such change. The D240 approval does not select syntax (`export` or otherwise), strong privacy, source-level module visibility rules or new foreign-import schemes. The original INC004 remains an incubator catalogue for other capabilities and is not magically closed by this decision.

## 5. GITHUB021 invariant reconciliation for selected scope

| Ratified earlier rule | D240-C/B treatment |
|---|---|
| `MODULES.md`: Protos source module == own `moduleContext`; foreign facade == Protos module instance | **Preserved in B**; D242 must explicitly revisit before any change of observable result |
| Canonical `ModuleKey`; cache-before-execute; cycles; retry; Actor locality | **Preserved**; aggregator has an additional, ordinary distinct `ModuleKey` |
| D137 direct-file `./file.protos` module identity | **Preserved**, no new raw-file return path |
| Package `[exports]` maps public module names to internal module paths | **Preserved**, no claim of existing slot-level exports |
| D188 and PLAT052/PLAT053: foreign facades, authorization, per-Actor provider/session lifecycle | **Preserved**, no automatic classes/JAR/C symbols and no new provider |
| D235/D236: frozen Core/Prelude Range/InvertedRange canonical identities | **Preserved**, only exact-reference publication permitted |
| D239-C1/I145-A: ordinary `Object.call` → `construct` protocol and fresh method-extraction semantics | **Preserved**, no implicit `module.Prototype(...)` |
| Date/Time/Duration: immediate parent is source module and family recognizer observes it | **Preserved in B**, not silently migrated |
| `ACTORS.md`: module+own bootstrap binding and pass-by-value | **Preserved**, no forced bootstrap renames |
| D061/D064/D067: mechanically observable API/documentation surface | **Preserved** until explicit D242-compatible tooling contract |
| D238: explicitly approved `std:ffi` and trusted `ffi:sqlite3` direction | **Preserved**, `import("c:sqlite3")` is not approved |

No already-approved invariant is silently reopened by this **bounded** architectural choice; all potential observable changes have been separated into D242 or future approved implementation decisions. This GITHUB021 check does **not** grant D242 authorization.

## 6. Alternatives, strongest objection, and why selection is limited

- **A status quo + docs only:** smallest zero-code choice, but leaves the public-API ambiguity unaddressed.
- **B additive aggregate modules:** smallest implementable current step, low compatibility risk, no enforced export visibility and potential opt-in eager cost.
- **C selected target / staged from B:** separates logical library, importable module, exported object and source unit, preserving Protos ordinary objects and offering gradual migration; exact slot export semantics remain undecided.
- **D mandatory entire-library root, forced reorganization:** REJECTED; high eager-startup cost, Core/Actor parent/call/identity breaks, auto-discovery pressure.
- **E arbitrary module return values:** REJECTED because it would undermine `import()`'s predictable module-instance identity.

**Strongest argument against C:** it risks imposing an export/visibility framework, duplicating package `[exports]`, inventing getter/proxy/lazy-binding semantics, or new semantics of `parent()`/receiver binding merely for a nicer `std:datetime` call site. The answer is NOT to prebuild the framework: stage B with existing ordinary objects, and ask D242 to falsify the minimum exact export contract before implementation.

**Costs of deferral:** deferring B leaves today’s import ergonomics; low future cost. Deferring exact named public export semantics entirely would invite additional implicitly public std/foreign APIs and later migration cost; it is explicitly tracked in D242 instead of implemented speculatively.

## 7. Status and evidence

This is **ratification of a narrow design architecture**, not a declaration that every detail of library/module/export behavior is now specified. Documentation and future implementation must cite the exact Protos revision when asserting implemented behavior. No Protos source, spec, tests, CLI resolution, foreign provider, changelog or version has been changed by recording D240.

[Adversarial evidence](../../evidence/D240/D240_ADVERSARIAL_MODULE_LIBRARY_EXPORT_REVIEW_2026_11_05.md) includes the complete 12-criterion scoring matrix, nine cross-system comparators, concrete counterexamples, noncompensating red flags, GITHUB021 preservation audit, scale/Actor stress and unapproved-change ledger.

```text
FORMAL_IDENTIFIER=D240
FAMILY=D
DECISION=C_ARCHITECTURAL_DIRECTION_INCREMENTAL_FROM_B
OWNER_APPROVAL=2026-11-05_EXPLICIT
GITHUB010=EVIDENCE_RECORDED
GITHUB021=PASS_FOR_BOUNDED_SELECTION
PROTOS_REVISION=384966eec789831bdc954f44d3d17e891fd1cb19
NEXT_EXACT_DESIGN=D242_UNSELECTED
NEXT_ADDITIVE_IMPLEMENTATION=I146
IMPLEMENTATION_PERFORMED_BY_THIS_RECORD=NO
NORMATIVE_SPEC_MODIFIED=NO
TESTS_PERFORMED=NO
```
