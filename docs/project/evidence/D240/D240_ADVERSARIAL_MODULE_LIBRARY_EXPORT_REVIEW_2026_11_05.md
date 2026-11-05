# D240 — Adversarial evaluation of public library/module topology

**Evidence role:** GITHUB010 comparative/falsification packet and GITHUB021 invariant review for the bounded D240 architecture selection. This is non-normative decision evidence, **not** implementation or test evidence.
**Product revision inspected:** [guillermomolina/protos@384966eec789831bdc954f44d3d17e891fd1cb19](https://github.com/guillermomolina/protos/commit/384966eec789831bdc954f44d3d17e891fd1cb19), 2026-11-05, `0.3.446-SNAPSHOT / specification 0.1.493.
**Issue:** [D240 #1057](https://github.com/guillermomolina/protos/issues/1057).
**Incubator provenance:** [INC004 #1010](https://github.com/guillermomolina/protos/issues/1010), “Module access declarations”.
**Separate exact-contract design:** [D242 #1060](https://github.com/guillermomolina/protos/issues/1060).
**First optional additive implementation owner:** [I146 #1061](https://github.com/guillermomolina/protos/issues/1061).
**Decision record:** [D240 C staged from B](../../decisions/language/D240_PUBLIC_LIBRARY_MODULE_TOPOLOGY_C_INCREMENTAL_B_RATIFIED.md).
**Verification:** Static source/spec/issue and selected external precedent review only. NO builds, Protos execution, performance measurements, formatter, tests or `git diff --check` were run for this design work.

## 1. What was actually found

The initial hypothesis that `import()` has inconsistent return *kinds* is false. `MODULES.md` guarantees the module instance: source modules return their ordinary Actor-local `moduleContext`; foreign providers return an Actor-local Protos facade over a foreign handle. A foreign class object is **not** the module identity itself. The same canonical `ModuleKey` within an Actor returns the same active cached instance; Actors have distinct module instances, initialization is cache-before-execute, cycles may observe partial real state, and failed loads are evicted.

The genuine gap is the lack of a reliable distinction between (1) a logical library/distribution, (2) an importable module with a canonical `ModuleKey`, (3) named public values or prototypes on that module, and (4) source files and foreign runtime objects backing that interface.

Protos already has **package-level module exports**: `[exports]` maps package-public logical module paths to package-internal logical modules, and `dep:<alias>/<public-export>` routes through that map. This is NOT named slot-level visibility or private lexical state. In contrast, the `std:` resolver maps an exact logical name to `protos/lib/<name>.protos` without an allowlist; for instance `std:datetime` currently fails because `protos/lib/datetime.protos` does not exist, while `std:datetime/Date` resolves to a module that is itself the Date prototype.

Representative baselines:
- [`MODULES.md`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/spec/semantics/MODULES.md): identity, source/foreign module placement, partially initialized modules, failure/retry.
- [`ProtosStandardLibraryModuleResolver.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/execution/modules/ProtosStandardLibraryModuleResolver.java): exact file resolution; no public-path allowlist.
- [`PACKAGE_MANIFEST_SCHEMA_V1.md`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/docs/design/PACKAGE_MANIFEST_SCHEMA_V1.md) and [`ProtosWorkspacePackageModuleResolver.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/execution/workspace/ProtosWorkspacePackageModuleResolver.java): path-level package exports; not slot-level exports.
- [`Date.protos`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/protos/lib/datetime/Date.protos): values have imported module instance as immutable immediate `parent()` and are recognized by that family.
- [`Range.protos` (Core)](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/protos/lib/core/Range.protos) and [`std:collections/Range` compatibility](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/protos/lib/collections/Range.protos): canonical frozen Core/Prelude Range identities are published on an Actor-local module unchanged.
- [`ProtosHostJavaCatalogue.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/execution/foreign/host/ProtosHostJavaCatalogue.java) and [`ProtosForeignModuleFacadeValue.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/execution/foreign/api/ProtosForeignModuleFacadeValue.java): an allowlisted Java class may back an importable Protos facade, with no classpath scan or ambient authority.
- [`ProtosActorValueTransfer.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/runtime/ProtosActorValueTransfer.java): execution-context parents are nontransferable; an ordinary Date instance cannot be assumed transferable simply because it is frozen.
- [`ACTORS.md`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/spec/concurrency/ACTORS.md), `8: `Actor.spawn` selects a canonical destination module and its own-slot callable bootstrap binding; a new aggregator does not automatically replace historical entry points.
- [`ProtosStandardLibraryDocumentationExtractor.java`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/src/main/java/com/guillermomolina/protos/documentation/ProtosStandardLibraryDocumentationExtractor.java): current API extraction derives `std:` identities and mechanically visible own slots from physical sources, plus runtime-provided member markers.
- [`CALLABLES.md`](https://github.com/guillermomolina/protos/blob/384966eec789831bdc954f44d3d17e891fd1cb19/spec/semantics/CALLABLES.md): `module.Date(...)` is a method send requiring a Closure; a slot containing a prototype is not implicitly transformed into a method; `Date: module.Date; Date(...)` works by normal invocation of the extracted prototype.
- [D238 #1047](https://github.com/guillermomolina/protos/issues/1047): C ABI `std:ffi` / explicit authorized `ffi:sqlite3` design, not an already-implemented generic `c:sqlite3` module import.

## 2. Counterexamples intended to break the proposal

| Attempted falsification | Actual failure / constraint | Implication |
|---|---|---|
| Require one importable module for each complete library | Eager imports of numerous components change startup, side effects and failure order; multiply work per Actor | **REJECT** mandatory one-library/one-module |
| Treat a Java `java.base` or package import as automatic export of every class | Host-Java catalog explicitly authorizes individual classes and members; classpath scanning would bypass restricted admission | **REJECT** automatic foreign discovery and authority |
| Treat a native C shared library as a namespace of Protos-typed functions | Exported symbols omit argument/lifetime semantics; ABI descriptions and host authority are indispensable | **REJECT** untyped automatic C symbol export |
| Export a newly constructed `Date` object instead of original date module | Existing date `parent()`/recognizer and identity change | Preserve old object; future migration requires owner-approved observable delta |
| Copy Range prototypes into new collections module | D235 freezes canonical Core/Prelude identities | Reexport exact references, never clone |
| Replace `Actor.spawn("std:collections/Set","construct",...)` with a new aggregate entry | Bootstrap selects an exact own Closure-valued slot of a canonical module | Preserve old canonical entry or explicitly revise Actor bootstrap semantics |
| Publish namespace after completed initialization | Cyclic import must see *same real module instance* during INITIALIZING | Reject completion-time replacement namespace/copy |
| Assume reexported Closure keeps original receiver | Ordinary lookup binds receiver and methodHome; copying Closure slot may alter method behavior | Export original object or prove forwarding contract; no silent method copying |
| Assume `std:collections/internal/ArrayBulk.protos` is private by folder name | Exact `std:` resolver can locate the existing file; no allowlist checks | An aggregate is not an access-control boundary |
| Assume foreign module members already form a finite enumerable own-slot table | Facade member fallback is projected from provider; own-slot reflection is not a complete export catalog | Separate public API declaration from provider reflection |
| Add live/promise/lazy exports without rule changes | Ordinary object slots are not module live-binding cells/getters; error/cycle/suspension semantics absent | Defer and design under D242 |
| Treat frozen Date/Duration as Actor-transferable | Immutable immediate delegation parent is Actor-local module execution context | A separate family-transfer design would be needed |

## 3. Comparative precedents (9; >=3 design families)

1. **Prototype languages — Self**: object graph/traits and module organization can be orthogonal; not every namespace is an instance prototype. [Self documentation](https://bibliography.selflanguage.org/).
2. **Prototype languages — Io**: ordinary objects/slots and cloning suit export composition, but do not establish privacy by themselves. [Io](https://iolanguage.org/).
3. **Multi-file packages — Go**: package names normally lowercase; separately declare exported identifiers while grouping source files. [Go specification](https://go.dev/ref/spec#Exported_identifiers).
4. **Module visibility — Rust**: distinct crates/modules/files and explicit public reexports; export surface need not mirror physical structure. [Rust reference](https://doc.rust-lang.org/reference/visibility-and-privacy.html).
5. **ES modules / Node.js**: module namespace and named bindings differ from package path `exports`; live bindings and cycles require their own semantics. [ECMAScript import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import), [Node package exports](https://nodejs.org/api/packages.html#exports).
6. **Java / JPMS**: modules contain exported packages and types; a Java package is not one class, and exported packages need not provide a complete static class enumeration. [JPMS Module](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/lang/Module.html).
7. **Python / GraalPy**: package/submodule names are not in general identical to one well-defined exported-object kind, motivating Protos' stricter contract. [Python import](https://docs.python.org/3/reference/import.html).
8. **Zig**: file/namespace-centric imports are straightforward but undesirably tighten the public file-to-module relation if copied wholesale. [Zig documentation](https://ziglang.org/documentation/master/#Importing-files).
9. **Lua**: module loader can return an arbitrary value; deliberately NOT Protos' invariant. [Lua `require`](https://www.lua.org/manual/5.4/manual.html#pdf-require).

These precedents inform trade-offs; they do not override Protos' ratified module, Actor or foreign admission semantics. No claimed direct transplant of GraalPy/GraalJS runtime internals.

## 4. Complete candidate set and scored GITHUB010 matrix

**A** status quo plus documentation only. **B** additive optional ordinary `std:datetime`/other aggregators using existing imports, without real export enforcement. **C** logical libraries publish one or more public modules, each with explicitly defined named public API; keep imported module identity and separate files/foreign objects; begin incrementally with B. **D** mandatory exactly one entire-library namespace and forced source/foreign remapping; rejected. **E** dynamic module return value as in Lua; rejected outright because it violates the invariant that `import` returns the Actor-local Protos module instance.

Scores: 1=poor, 5=strong. H=high, M=medium, L=low confidence. The numbers are comparison aids, not a substitute for noncompensating invariant gates.

| Criterion | A | B | C | D |
|---|---|---|---|---|
| 1. Correctness/invariants | 5 H: no change | 4 H: additive identities | 4 M: exact export semantics pending | 1 H: breaks parent/Actor/import |
| 2. Protos alignment | 3 H: roles conflated | 5 H: ordinary composition | 5 M: explicit ordinary public boundary | 2 H: forced special cases |
| 3. Present proportionality | 5 H: zero cost | 5 H: optional | 3 M: new public rule is work | 1 H: broad compulsory migration |
| 4. Incremental growth | 3 M: accumulates debt | 5 H: grows additively | 5 M: extend export boundaries | 1 H: all-or-nothing |
| 5. Future options | 2 M: remains ambiguous | 4 M: later private exports possible | 5 M: paths/foreign remain independent | 2 M: locks module granularity |
| 6. Scalability | 3 M: fine-grained irregular | 4 M: opt-in aggregation | 5 M: one or more public units | 1 M: eager library explosion |
| 7. Simplicity | 3 H: fewer rules, confusing roles | 4 H: trivial guest layer | 4 M: two explicit visibility layers | 2 H: numerous workarounds |
| 8. Portability | 5 H: current | 5 H: ordinary source | 4 M: provider projections needed | 2 M: native-language mismatch |
| 9. Runtime/resources | 5 H: current | 4 M: opt-in import costs | 4 L: exports implementation not yet measured | 1 M: aggregate eager costs |
| 10. Failure/operability | 3 M: accidental surfaces | 3 M: aggregate init failures | 4 M: declared failures can be verified | 1 M: cross-library failures |
| 11. Deferral/migration | 2 M: future rewrite pressure | 5 H: reversible | 4 M: staged compatibility | 1 H: observable identity breaks |
| 12. Evidence/risk | 5 H: implemented | 5 H: existing mechanisms | 3 M: semantic details deferred | 2 M: analogy is insufficient |

**Noncompensating gates:** D is disqualified by eager loading, prior identity and permission contracts. E is disqualified by module return semantics. A alone fails the approved public-API clarity goal. C survives as a *direction*, not as approval to implement unspecified export syntax/visibility. B is the smallest presently implementable, reversible unit.

## 5. GITHUB021 previously approved invariant/delta audit

| Owner-approved source/contract | C staged via B disposition |
|---|---|
| `MODULES.md` and D137: exact resolver-to-ModuleKey; same Actor same module; cache-before-execute; partial cycles; failed retry | **PRESERVED**. No new namespace replacing cached module. `./file.protos` remains a module import. |
| `OBJECT_MODEL.md`: ordinary slots, frozen/closed behavior, observable immutable `parent()` | **PRESERVED in B**; any true private/export model that changes observable lookup requires separate D242 approval. |
| `CALLABLES.md`, D239-C1: late `call` selection, standard `construct` bridge, receiver/methodHome | **PRESERVED**. No `module.Prototype(args)` magic; extract prototype first. |
| D235/D236: shared frozen Core Range/InvertedRange, cross-direction construction | **PRESERVED**. Export exact Core references. |
| Date/Duration source families: module is immediate parent, family recognition, semantic equality/hash | **PRESERVED by aliases**. Any parent migration is a separately exposed breaking design choice. |
| `ACTORS.md`: Actor-local module ownership and `Actor.spawn` own-slot bootstrap | **PRESERVED**. Old bootstrap specifiers/bindings not silently renamed. |
| D188/PLAT052/PLAT053: foreign module facade, per-provider admission, no ambient host authority, Actor/provider isolation | **PRESERVED**. New foreign library grouping is not authorized pending D242. |
| D238: `std:ffi` / authorized `ffi:sqlite3` selection | **PRESERVED**. Never infer `import("c:sqlite3")` as already implemented/approved. |
| Package-tool `[exports]`: package-public MODULE path mapping, not slot visibility | **PRESERVED**. No conflation; slot exports exact rule deferred. |
| D061/D064/D067: documentation of mechanically observable current module surfaces | **PRESERVED**. A future export policy must explicitly reconcile extractor and symbol identity. |

There is **no unapproved reopened invariant** in the bounded D240 choice because it commits only to future architectural organization and an additive ordinary-module first step. The exact new binding/visibility contract is deliberately a separate, unselected D242 decision. Publishing this evidence does not itself alter normative `spec/`.

## 6. Deferred decisions, anti-overengineering and stress

**Approved smallest step:** an opt-in `std:datetime` ordinary aggregator exposing existing objects by identity; the existing `std:datetime/Date` etc. remain. Only consumers explicitly importing the aggregator pay its eager dependency cost.

**Not approved now:** a new `Library` value/type; mandatory one module per library; directory implicit import; default/implicit exports; strong privacy; export getter/live cells; generic laziness; JPMS/JAR scanning; raw C symbols as public methods; Core prototype cloning; breaking public `std:` renames.

**Current architecture regret if overbuilt:** one-library/one-module would make each Actor initialize unrelated modules; visible parent identity and Actor bootstrap break; foreign provider authority could be broadened; documentation and LSP would need a second universe.

**Concrete cost of deferral:** creating additive aggregators now is cheap and reversible; delaying the exact public module/slot boundary allows more implicit std/foreign surfaces and later migration work. D242 therefore explicitly tracks that semantic obligation, rather than leaving it in INC004 or silently implementing it within I146.

**Future stress cases / escape path:** thousands of Actors, large foreign catalogs, partial cycles, dynamic host providers, distributed execution, foreign resources and alternative VM backends all favor preserving granular module identity. If future demand requires lazy exports, live bindings or dynamic catalogs, obtain a bounded new decision and adapt providers/module presentation; do not reinterpret `import` return kind or admit ambient symbols by convention.

## 7. Investigation completion, approval provenance and limitations

The owner examined the proposal, asked to seek falsifying evidence under `AGENTS.md`, received the comparative C-vs-B-vs-D packet, then explicitly stated on **2026-11-05**: **“ok apruebo esta recomendación”** and ordered issue updates and durable evidence publication. The approved referent is **C as architectural direction, implemented incrementally through B**, including the disclosed deferrals. It does **not** constitute approval of any unpresented exact new syntax/visibility/lifetime/foreign import semantics.

**Evidence status:** static investigation complete for this bounded architecture choice; exact slot-level export semantics intentionally open in D242. No runtime execution, CI, benchmarks, or tests performed. This record is non-normative and revision-bound.
