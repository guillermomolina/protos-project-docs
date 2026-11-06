# D242 — Public MODULE/SOURCE separation, explicit ordinary exports and `package.toml`

**Status:** OWNER-APPROVED / DURABLY RATIFIED on 2026-11-06. **Implementation not yet published.**
**Governing Issue:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060).
**Upstream architecture:** [D240 C incremental from B](D240_PUBLIC_LIBRARY_MODULE_TOPOLOGY_C_INCREMENTAL_B_RATIFIED.md), ratified 2026-11-05; its **staged B-first/no-breaking-migration route is superseded by this separately approved later decision**. D240's historical record is not rewritten.
**Product source baseline inspected:** [guillermomolina/protos@380548e10cec893581ee25ebf00da9c5c6bd7399](https://github.com/guillermomolina/protos/commit/380548e10cec893581ee25ebf00da9c5c6bd7399) (still implements earlier source-module behavior; no D242 implementation asserted).
**Comparative research and GITHUB021 audit:** [D242 2026-11-06 evidence](../../evidence/D242/D242_MODULE_SOURCE_EXPORTS_BREAKING_REPLACEMENT_REVIEW_2026_11_06.md).
**Implementation ownership:** [I151 #1066 — MODULE/SOURCE runtime, resolvers and std library](https://github.com/guillermomolina/protos/issues/1066); [I152 #1067 — package manifest rename, tooling and identity](https://github.com/guillermomolina/protos/issues/1067).
**Superseded plan:** [I146 #1061](https://github.com/guillermomolina/protos/issues/1061) was an additive old-style `datetime.protos` aggregate and is **not** the selected route.

## 1. Owner selection and supersession scope

The owner explicitly approved the recommended MODULE/SOURCE public-interface architecture and the use of mandatory `module.protos`. The owner separately approved renaming `protos.toml` to `package.toml` for naming consistency, then explicitly rejected progressive migration, compatibility shims and fallbacks because there are no published packages to preserve.

This approval is a **new, specific design selection**, not an inference from D240's earlier limited B-first ratification. D242 governs the incompatible replacement on overlapping details, while D240's enduring principles (explicit public modules, package routes separate from files/objects, bounded imports, Actor-local identity, no foreign auto-discovery) remain respected.

This document is non-normative decision authority. The corresponding observable Protos rules are implemented only when the relevant `spec/` changes and code are published in `guillermomolina/protos`.

## 2. Fundamental ontology

Three distinct concepts:

1. **Package:** distribution/identity/dependency boundary; declarative manifest `package.toml`, publishing **module paths** through existing `[exports]`.
2. **MODULE:** public importable Protos logical module identified by a directory with **mandatory** `module.protos`. `import` returns its ordinary Actor-local public object with **only explicitly published own slots**.
3. **SOURCE:** authorized individual `.protos` file. `import` returns the ordinary complete source `moduleContext` containing **every top-level binding of that source**, not a filtered public module interface.

One `import(String)` operation handles both. Target classification is determined by the resolver's **authorized resolved kind** (MODULE or SOURCE), not by inventing a semantic difference between `self:`, `std:` and `dep:` themselves. Prefixes select resolution domains and authority.

A SOURCE file does **not** automatically create a public logical MODULE. A directory without `module.protos` is **not** a Protos logical MODULE. Foreign provider modules are admitted through their provider/catalog contracts; they are not required to contain a Protos physical file.

## 3. Canonical example

```text
math/
├── package.toml
└── range/
    ├── module.protos
    └── Range.protos
```

```toml
# package.toml
manifest-version = 1
[package]
id = "math"
version = "1.0.0"
[exports]
range = "range"
```

```protos
// range/Range.protos — SOURCE
validateBounds: (start, stop) => { start <= stop }
Range: {
    init: (start, stop) => {
        if(!validateBounds(start, stop)) {
            Error().signal()
        }
        this.start: start
        this.stop: stop
    }
}
```

```protos
// range/module.protos — MODULE initializer
implementation: import("./Range.protos")
this.Range: implementation.slotValue("Range")
```

```protos
// Consumer imports a public MODULE
range: import("dep:math/range")
Range: range.Range
item: Range(1, 10)
```

Illustrative observable contract **for two imports made within one Actor in an authorized context**:

```protos
module: import("self:range")
source: import("./range/Range.protos")

module !== source
module.Range === source.slotValue("Range")
module.hasSlot("Range")              // true
module.hasSlot("validateBounds")     // false
source.hasSlot("Range")              // true
source.hasSlot("validateBounds")     // true
```

The package-aware relative `./` SOURCE resolver and MODULE/SOURCE classification are part of I151; do **not** claim these new illustrations already execute in the current product. `module.Range(...)` does **not** automatically invoke a prototype-valued slot through the regular member-call protocol; use `Range: module.Range; Range(...)`.

## 4. MODULE execution contract

For an authorized import resolving to MODULE at logical name `range` and physical `range/module.protos`:

1. Resolve exactly one canonical MODULE `ModuleKey` for the correct package/toolchain origin and logical directory; it must not collide with SOURCE key identities.
2. Allocate the **ordinary public module object** and insert it in the Actor-local module cache in state `INITIALIZING` **before** evaluating the initializer.
3. Allocate a separate **private execution context**; execute `module.protos` with `context` referring to this private context and `this` referring to the public module object.
4. Lexical lookup for this initializer follows **private context → public module context → frozen Prelude**. Private bare bindings are stored in the private context; `this.name: value` creates an ordinary public slot on the public object at the expression's execution point.
5. Each public slot is observable immediately. Ordinary assignment or removal follows the normal Protos slot rules; no implicit public interface freeze or special live binding is introduced. Exporting a value is a reference assignment, not cloning.
6. On successful completion, transition the existing public cached object to `READY` and return it. Ignore the initializer's ordinary final-expression result: **do not** substitute that result as the module instance.
7. On unhandled initialization failure, evict the failed active cache entry and allow retry; do not undo side effects or revoke escaped references to partially initialized objects.
8. Recursive import of MODULE during `INITIALIZING` returns **the same exact public instance immediately**; only exports installed so far exist. No implicit await, TDZ, hoisting, lazy getters, or deferred atomic export table.

**Observable property:** a normal bare initializer binding `implementation: ...` is not a public slot; `this.Range: ...` is public. Private lexical state may remain captured by exported Closures, and remains accessible through such Closures only as permitted by ordinary semantics.

## 5. SOURCE and existing Core rules

SOURCE import retains the existing module runtime model: a cache-before-execute source `moduleContext` is **both** the top-level lexical context and the imported object; all its own top-level slots are observable. SOURCE also participates in canonical identity, cycles, failure/eviction and Actor-local lifetime. The same file must not be evaluated as a second independent SOURCE alias of its own MODULE initializer.

Object identity, delegation, `Object.call`/`construct`, ordinary member sends, `this` of methods, `methodHome`, `super` and extracted Closure rebinding remain governed by existing object/Closure specs. An object-valued export preserves its exact identity. A Closure-valued export is stored in an ordinary public slot and on member read obeys ordinary binding **to the public receiver**, not a magical binding back to its source module.

Public API isolation is **shallow**: an exported object or SOURCE context can expose its own slots by design; this is an interface boundary, not a sandbox or deep object privacy guarantee. No extra `Module` class or syntax `export`/`module.toml` is introduced.

## 6. Resolution, packages, and no compatibility layer

- **`std:range`** resolves to module directory `protos/lib/range/module.protos` (MODULE); SOURCE imports are separately authorized by exact path and caller provenance.
- **`self:range`** resolves to the package's logical MODULE; **`dep:math/range`** follows the dependency's declared `[exports]` entry and exposes the MODULE, not internal SOURCEs.
- An authorized relative import `./Range.protos` resolves a SOURCE relative to its importer in the bounded source tree. Do not grant `dep:` direct access to private SOURCE paths.
- The namespace must reject ambiguity between same-spelling SOURCE and MODULE, case-fold/alias collisions and direct SOURCE imports of `module.protos` that could duplicate initializer identity. No implicit parent-module execution on import of a child SOURCE.
- New logical public module names normally use lowercase path segments; exported Protos prototypes and values retain their ordinary names. No automatic whole-library/root eager load.
- **`package.toml` is the sole package manifest** for the root and workspace members, retaining TOML 1.0 + strict Protos package schema. `[exports]` still maps public MODULE names to internal logical module directory names. A directory has no MODULE without `module.protos`.
- **`protos.toml` is removed from active readers, writers, discovery, package validity and fixtures**: old-only and both-old-and-new fail. No redirect, dual reader, alias, opt-in compatibility mode, or progressive release.
- **`protos.lock` remains the lock filename**. Package metadata, staging paths, lock freshness and Canonical ContentIdentity must use new manifest paths.
- No published packages need retained digest identity. The old SHA-256 vectors do exist in historical repository evidence; do not mistake them for immutable published-package requirements. Replace/recalculate **current** canonical-format vectors and authoring rules coherently, including appropriate method-token/format version discipline. Keep past evidence unchanged as historical facts; do not silently claim an old-format digest authenticates a new-format tree.

## 7. Impact on previously approved and current surfaces

This supersedes the **staged, opt-in B implementation** permitted by D240 and the specific `I146` proposal (`datetime.protos` ordinary aggregator), without retroactively falsifying D240's historical audit. I151 must reorganize the Standard Library, tools, package resolver, Actor bootstraps, documentation and tests into the new model, replacing old public file-backed module specifiers instead of retaining them as aliases.

Where previous source module instances were also factory/delegation prototypes (`Date`, `Duration`, `Set`), changes in parent, recognizer, hashing/equality or Actor message routing require *consistent explicit new semantics and matching conformance tests*. They are not automatically preserved by file moves; nor is a no-published-package assertion evidence that changing them is semantically trivial. Frozen Core `Range` identities may not be cloned or altered merely to implement exports. Preserve foreign admission/capability limits; do not automatically export Java classpath or native library symbols.

Package Tool's current `protos-package-tree-v1` checks include the literal root `protos.toml` and identity-bearing path bytes; I152 must re-baseline them as part of the selected breaking change. This is **not** license for undocumented hash aliasing or treating a prechange SHA as if it were new-format bytes.

## 8. Implementation and acceptance ownership

**I151:** normative modules/control/Closure deltas; MODULE/SOURCE resolution and cache keying; private/public activation (including initial modules), Std Library and tool reorganization; actor bootstrap callers and tests; exact public slot sets, partial cycles, failures, closures, Core identity, source-authority rejection, docs/LSP and hot-path performance checks.

**I152:** `package.toml` manifest resolution, Package Tool/TOML, CLI/workspaces/staging, all fixtures/guides, canonical package tree SHA vectors and resolution-input/`protos.lock` verification; no legacy fallbacks.

Issues may proceed in independently validated coherent slices. The requested end state is **direct replacement**, not long-running dual behavior. Agent publication in `guillermomolina/protos` requires separate human-executed validation and commit under `AGENTS.md`. The owner permitted direct documentation publication only in `guillermomolina/protos-project-docs`.

**Minimum conformance before implementation closure:** all source/module isolation cases, identity and failure/partial cycles, Actor and foreign boundary, package route confinement, exact filename selection, digest and lock consistency, complete repository migration of old import strings and manifest names, docs/navigation, plus owner-reported `make test` and `make check`.

## 9. Ratification and limitations

The owner approved (1) the MODULE/SOURCE design and `this.` exports, (2) the mandatory `module.protos`, (3) the direct `protos.toml` → `package.toml` replacement, and (4) no gradual migration/compatibility because no published package identities need support. These approvals are **design/implementation authority** only; they do **not** assert tests passed or source changes were published.

**GITHUB010 comparative packet, all 12 criteria, and GITHUB021 explicit invariant/delta reconciliation:** [supporting evidence](../../evidence/D242/D242_MODULE_SOURCE_EXPORTS_BREAKING_REPLACEMENT_REVIEW_2026_11_06.md).

**D242 completion criterion:** this durable record published and live Issue linked/reconciled; dependent I work remains open until independently implemented, validated and published.

```text
FORMAL_IDENTIFIER=D242
FAMILY=D
STATUS=OWNER_APPROVED_DURABLY_RATIFIED
OWNER_APPROVAL=2026-11-06_EXPLICIT
DESIGN_SELECTED=C_PUBLIC_MODULE_PRIVATE_SOURCE_CONTEXT
MANIFEST_FILENAME=package.toml
MODULE_ENTRY_FILENAME=module.protos
MIGRATION_POLICY=BREAKING_NO_FALLBACK_NO_PROGRESSIVE_COMPAT
GITHUB010=PASS_RECORDED
GITHUB021=PASS_EXPLICIT_DELTA
PROTOS_REVISION=380548e10cec893581ee25ebf00da9c5c6bd7399
IMPLEMENTATION_OWNERS=I151,I152
IMPLEMENTATION_PUBLISHED=NO
```
