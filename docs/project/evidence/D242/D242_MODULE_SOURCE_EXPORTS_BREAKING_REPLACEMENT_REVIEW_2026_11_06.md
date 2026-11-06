# D242 — Adversarial evidence for MODULE/SOURCE split and package manifest replacement

**Evidence class:** GITHUB010 comparative design + GITHUB021 bounded invariant/delta audit; **no executable implementation or test evidence**.
**Owner decision date:** 2026-11-06, explicit approval in active design discussion followed by explicit instruction to remove legacy implementations and **not** perform a progressive migration.
**Product revision inspected:** [guillermomolina/protos@380548e10cec893581ee25ebf00da9c5c6bd7399](https://github.com/guillermomolina/protos/commit/380548e10cec893581ee25ebf00da9c5c6bd7399).
**Governing issue:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060); prior design [D240 #1057](https://github.com/guillermomolina/protos/issues/1057); implementation [I151 #1066](https://github.com/guillermomolina/protos/issues/1066) and [I152 #1067](https://github.com/guillermomolina/protos/issues/1067).
**Ratified decision:** [D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md](../../decisions/language/D242_MODULE_SOURCE_PUBLIC_EXPORTS_AND_PACKAGE_MANIFEST_RATIFIED.md).

## 1. Grounded baseline, not an implementation claim

- [MODULES.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/semantics/MODULES.md): current source-backed `module context is **both** executing context and imported object; top-level slot publication is implicit; canonical ModuleKey, Actor cache-before-execute, cycles, failure eviction are normative. D242 **deliberately changes the former relation for logical MODULE**, not for SOURCE.
- [EXECUTION_AND_CONTROL.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/semantics/EXECUTION_AND_CONTROL.md): current lexical parent is frozen Prelude; D242 introduces private context → public module context → Prelude **for MODULE only**, with `this` as public receiver and `context` as private context.
- [CALLABLES.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/semantics/CALLABLES.md): method reads bind receiver and methodHome. Exporting a Closure as an ordinary slot can rebind its receiver; exporting an original prototype object avoids unintended reparenting.
- [OBJECT_MODEL.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/semantics/OBJECT_MODEL.md) and [PROTOS_GRAMMAR.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/PROTOS_GRAMMAR.md): `:` is ordinary slot creation and `this.Range: value` is already valid. `slotValue` returns own raw slot value; `slotNames` reflects own slots. No new export keyword necessary.
- [ProtosModuleRuntime.java](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/src/main/java/com/guillermomolina/protos/execution/modules/ProtosModuleRuntime.java): currently creates and caches one execution context and passes that same object to `newModuleActivation`; `finish(bodyResult)` discards body result. New module-kind activation is a real runtime change, not merely a resolver alias.
- [ProtosPrelude.java](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/src/main/java/com/guillermomolina/protos/runtime/ProtosPrelude.java): current module activation passes execution context as both lexical context and receiver; must be split in new MODULE kind only.
- [ProtosStandardLibraryModuleResolver.java](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/src/main/java/com/guillermomolina/protos/execution/modules/ProtosStandardLibraryModuleResolver.java) and [ProtosWorkspacePackageSourceLookup.java](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageSourceLookup.java): map logical name directly to `<name>.protos`; source file currently implies importable module. Need explicit target kinds, no suffix guessing.
- [ProtosPackageExecutionPlanV2ModuleResolver.java](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/src/main/java/com/guillermomolina/protos/execution/packages/ProtosPackageExecutionPlanV2ModuleResolver.java) and [PACKAGE_MANIFEST_SCHEMA_V1.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/docs/design/PACKAGE_MANIFEST_SCHEMA_V1.md): package `[exports]` already maps public routes to internal module logical names; must now require directory with `module.protos`; no two export manifests.
- [Date.protos](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/protos/lib/datetime/Date.protos) uses its source module itself as an immediate Date prototype; [collections/Range.protos](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/protos/lib/collections/Range.protos) publishes frozen Core Range identities. They require deliberate source-family adaptation, not blind file moves.
- [ACTORS.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/spec/concurrency/ACTORS.md) requires destination module's exact own bootstrap slot; moving `std:collections/Set` cannot silently preserve historical actor bootstraps.
- [PACKAGE_CONTENT_IDENTITY.md](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/docs/design/PACKAGE_CONTENT_IDENTITY.md), [fixed vectors](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/docs/design/PACKAGE_CONTENT_IDENTITY_VECTORS.md), [ProjectMetadata.protos](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/protos/tools/package/ProjectMetadata.protos), [ContentIdentity.protos](https://github.com/guillermomolina/protos/blob/380548e10cec893581ee25ebf00da9c5c6bd7399/protos/tools/package/ContentIdentity.protos) hardcode root `protos.toml` as a path included in canonical package bytes. Filename rename changes digest; no public distribution SHA is reported as needing migration.

## 2. Systems comparison (>=5 maintained systems, >=3 families)

| System | Relevant actual contract | Lesson / rejection |
| --- | --- | --- |
| [GraalJS JSModuleNamespace](https://github.com/oracle/graaljs/blob/master/graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/builtins/JSModuleNamespace.java) | Separate module namespace with special export/property resolution | Public/private split is real, but special namespace and live bindings would be excess machinery for Protos. |
| [GraalPy PythonModule](https://github.com/oracle/graalpython/blob/main/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/builtins/objects/module/PythonModule.java) | Module object and module dictionary are tied to Python module behavior | Reuse ordinary object identity; do not assume Python `__init__.py` or module dictionary yields explicit exports. |
| [Python import system](https://docs.python.org/3/reference/import.html) | Directory `__init__.py` with parent-package initialization; namespace packages can omit it | Adopt mandatory directory initializer **but reject** optional namespace packages and automatic parent initialization. |
| [Rust modules/visibility](https://doc.rust-lang.org/reference/items/modules.html) | Modules, `mod.rs` and explicit exports/visibility | Filename entry convention plus export policy is coherent; no Rust visibility keyword required. |
| [Go spec](https://go.dev/ref/spec#Packages) | Multi-file package shared scope | Credible alternative; not needed because Protos SOURCE contexts already have distinct lexical capture semantics. |
| [Zig module imports](https://ziglang.org/documentation/master/#import) | `@import` handles modules and files through distinguishable targets | One import operation does not imply one import target semantic kind. |
| [ECMAScript modules](https://tc39.es/ecma262/#sec-modules) | Namespace exports separate from lexical declarations, cycle linking and live binding rules | Need early public instance and observable cycles; **reject** hoisting/TDZ/live binding machinery. |
| [Node package exports](https://nodejs.org/api/packages.html#exports) | Package-level authorized public paths independent of source layout | Confirms that package exports must not expose arbitrary internal files. |
| [OCaml modules and signatures](https://ocaml.org/manual/5.3/modules.html) | Signature filters public declarations while implementation has more | Need explicit surface; no separate `module.toml` signature required. |
| [Lua require](https://www.lua.org/manual/5.4/manual.html#pdf-require) | Module loader can return a module value distinct from local implementation | Ordinary object API is attractive; replacing cache object at end is not compatible with Protos cycles. |

These are precedents, not semantic authority for Protos.

## 3. Candidates and GITHUB010 scoring

Candidates:
- **A — Status quo:** each source file is automatically an importable module, all top-level bindings public.
- **B — Ordinary aggregator:** no private/public distinction; explicit aggregator slots but source implementation remains directly importable.
- **C — Selected:** MODULE has public object cached early + separate private context, explicit ordinary `this.Name:` publication; SOURCE retains full source context.
- **D — Return final exports:** `module.protos` returns a table/namespace which replaces or populates public object only on completion.
- **E — New syntactic export/live binding mechanism:** grammar plus separate export storage.

Scale 1 (poor) to 5 (strong). Confidence H=high, M=medium, L=low. Reasons are included per cell.

| Required criterion | A | B | C — selected | D | E |
| --- | --- | --- | --- | --- | --- |
| Correctness/invariants | 2H: no filtering | 3M: manual filtering | 5M: stable identity | 2H: cycles late | 4M: complex |
| Protos alignment | 3H: slots only | 4H: ordinary | 5M: ordinary contexts | 3M: replacement | 2M: new syntax |
| Present-need proportion | 2H: fails goal | 3M: inadequate | 5M: exact goal | 3M: fewer parts | 2M: overbuild |
| Incremental growth | 2M: surface lock-in | 3M: can migrate | 5M: modular | 3M: late exports | 4M: extensible |
| Future-option resilience | 3M: paths exposed | 3M: manual API | 5M: separation | 3M: lifecycle risk | 4M: static semantics |
| Scalability | 4H: established | 4M: eager refs | 4M: context per MODULE | 4L: final copy | 4L: runtime storage |
| Conceptual simplicity | 5H: old simple | 4H: partial | 4M: one new distinction | 3M: commit stage | 2M: syntax+engine |
| Portability | 5H: portable | 5M: portable | 5M: source-independent | 5M: portable | 5M: portable |
| Runtime/resource cost | 5H: existing | 5M: no new context | 4M: one context per MODULE | 4M: commit copying | 3L: specialized rules |
| Failure/operability | 5H: known | 4M: partial | 5M: early observable slots | 2H: cycle hole | 4M: requires linking |
| Deferral/reversibility | 3M: later break | 3M: later break | 4M: clean target now | 2M: awkward rewrite | 2M: syntax entrenched |
| Evidence maturity/risk | 5H: implemented | 4H: source examples | 3M: needs tests | 3M: Lua precedent | 3M: ES precedent |

**Non-compensating gate:** A/B do not meet the required difference between SOURCE-complete and MODULE-export-only; D violates the same-real-module partial-cycle intent; E adds syntax/runtime complexity absent a motivating need. C carries a real new semantic cost (private context identity) but the owner expressly approved that change. No weighted sum can excuse a lost invariant.

## 4. GITHUB021 invariant/delta audit against D240

| D240 invariant or boundary | D242 result |
| --- | --- |
| `import` returns exact Actor-local module object, not arbitrary constructor | **Preserved** for MODULE; SOURCE returns its execution context as before. |
| Canonical ModuleKey, cache-before-execute, partial cycles, eviction/retry | **Preserved**; MODULE public object exists before private body executes. |
| Multiple public modules, no mandatory whole-library import | **Preserved**; one `module.protos` per public directory, imported on demand. |
| Exact frozen Core Range references and ordinary objects/slots | **Preserved as a required invariant**; no cloning or automatic prototype reparenting. |
| Foreign admission/capability restrictions | **Preserved**; Java/native providers have their own authorized facades and never import arbitrary classes/symbols. |
| Actor bootstrap exact own-slot requirement | **Preserved as a rule**; old module-name bootstrap call sites must be rewritten. |
| D240 prior staged B-first rollout and no loader-semantic change in first slice | **Expressly superseded** by owner's new 2026-11-06 approval: direct C and intentional MODULE/SOURCE semantic delta. |
| Previously discussed legacy `std:datetime/Date` etc compatibility | **Expressly declined**, not an implicit backward-compatibility promise; migrate source families and Actor callers rather than publish aliases. |
| Same source `moduleContext` as public object for every import | **Expressly replaced for MODULE only**; SOURCE retains previous contract. |
| Package manifest `protos.toml` and content-tree SHA vectors | **Expressly replaced** with `package.toml`; baseline algorithms/vectors must be consistently reauthored for the new source format, no claim published package digests exist. |

**GITHUB021 status:** PASS as a **bounded change explicitly re-approved by the owner**, not a claim that old D240 staging text was already equivalent. Prior D240 record stays intact as history; this D242 record is the later controlling selection on conflicting facets.

## 5. Adversarial acceptance and failure matrix

1. **No `module.protos`:** folder cannot be imported as a logical MODULE; SOURCE files do not implicitly create public MODULE routes.
2. **SOURCE vs MODULE:** `module.slotNames()` has exactly explicitly published own slots (apart from deliberately specified standard slots); `source.slotNames()` has all top-level source bindings; private body binding absent from public module.
3. **Source object identity:** `module.Range === source.slotValue("Range")` for object-valued export; do not clone frozen Core Range.
4. **Receiver/Closure:** `this` in initializer is public object, `context` is private execution context; exported methods follow ordinary fresh binding, `methodHome`, `super`; no automatic rebind to source receiver.
5. **Cycles:** MODULE cache inserted before private body runs; cyclic reader sees same real public module with only already-published slots; no deadlock/TDZ/hoisting.
6. **Failure:** initialization failure evicts cache; later import retries; escaped partial refs/effects are not rolled back.
7. **Collision:** MODULE key never equals SOURCE key even for neighboring or identical paths; direct `module.protos` SOURCE route cannot create a second initializer.
8. **Authorization:** `dep:` is limited to declared module export routes; no raw file reach-through; same-package SOURCE imports must remain confined by strict path/case/symlink and package custody rules.
9. **Actors/foreign:** Actor-local distinct module instances; frozen Core allowed shared; original destination module bootstrap must be rewritten to the new module's own bootstrap binding; no foreign capability amplification.
10. **Cost:** MODULE pays one private context and explicit imported source contexts only when requested; SOURCE stays ordinary; benchmark closure/dispatch hot paths, ensure no export-check fallback.
11. **Package manifest:** only root/member `package.toml` accepted; `protos.toml` absent/legacy/both rejected; `protos.lock` unchanged; independent new ContentIdentity vector oracle and lock resolution-input check.
12. **Toolchain/documentation:** CLI, bundled tools, all std imports, tests/examples/tutorials, LSP navigation and Standard Library source extraction agree on public module names and private source provenance.

## 6. Anti-overengineering, deferred choices and strongest criticism

**Minimum sufficient design:** two target kinds at resolution; MODULE public object and private initializer; ordinary slots from `this.` as exports; existing SOURCE lifecycle unchanged; one `import`; existing package export table; single `package.toml`. No `module.toml`, parser export construct, synthetic runtime type, live getters or compatibility shims.

**Pay for what you use:** existing file-only applications do not pay the extra MODULE lexical scope; importing one module need not load unrelated modules in its distribution. The only new cost is the logical module's own private context and explicitly imported sources.

**Grow as needed:** preserve future explicit module visibility policies, static export diagnostics, optional signatures and foreign module grouping without implementing them. If true shared-lexical-context multifile units become necessary, evaluate separately: this decision intentionally preserves independent SOURCE contexts.

**Concrete cost of deferral:** without formal split, all public module clients depend on implicitly exported source-local slots and the one-context identity. Adopting it after published packages would force larger compatibility changes, hence the owner's choice to break now while no published packages exist.

**Strongest criticism:** C changes the existing invariant `activation.context === cachedModuleInstance` for MODULE, potentially interacting with Bytecode DSL, closures, debugger, Actor bootstrap, source identity and continuation unwinds. Static review and precedents do **not** prove correctness. The implementation must use the exact normative changes plus conformance and performance tests. No Java/source patch has been made by this decision publication.

**Explicitly deferred:** byte representation of MODULE/SOURCE keys, implementation layering between resolver/activation and compiler, exact lock hash method-token disposition, and any future export-immutability/visibility enforcement beyond ordinary public own slots. If any deferred implementation choice changes observable language semantics, it requires separate owner approval; no silent fallback is authorized.

## 7. Publication evidence scope

- D242 owner's explicit approval covers the proposed C implementation contract, the mandatory `module.protos`, the mandatory `package.toml`, no backwards compatibility/progressive migration, and separate I-family implementation ownership.
- Historical D240 and earlier SHA fixtures remain historical provenance; modifying the *current* implementation/conformance rules is authorized, but do not rewrite historical evidence to imply the old implementation already used `package.toml`.
- **Validation:** static specifications, real source, GitHub issue and source precedents reviewed; **tests/build/benchmarks/product edits: NOT RUN / NOT PUBLISHED**.
- **Next:** I151/I152 implement; only then can normative spec/runtime/Package Tool changes be claimed complete.
