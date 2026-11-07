# AUD023 — Cross-family collection protocol coherence audit

- **Issue:** [AUD023 / guillermomolina/protos#1087](https://github.com/guillermomolina/protos/issues/1087).
- **Product baseline:** [`guillermomolina/protos@08c27760eb3f85105277e5b1b1b159abbca67ebe`](https://github.com/guillermomolina/protos/commit/08c27760eb3f85105277e5b1b1b159abbca67ebe), inspected 2026-11-07.
- **Evidence:** [AUD023 exact source and test map](../../evidence/AUD023/AUD023_REVISION_BOUND_COLLECTION_PROTOCOL_EVIDENCE_2026_11_07.md).
- **Status:** research assessment available for owner review; **no new semantics selected, no normative edit or implementation, no tests executed as part of this audit**.
- **Authority:** normative product specifications, not this non-normative work record. The previously ratified D213, D214, D231, D232/D235/D236, D188/D192 and AUD009-B9/E1 outcomes are preserved.

## Executive finding

The collection APIs are not a single interface and should not be made one. Existing **syntax** is already coherent: indexing lowers uniformly to ordinary `at` / `atPut` sends, and contextual generic `for` to `each`; the **receiver** chooses whether and how those messages apply. This intentionally allows a dense fixed-length mutable `Array`, growable `Bytes`, keyed `Map`, mutable-role Sets, frozen computational Ranges, fixed-shape numeric Vector/Matrix, immutable Unicode String, read-only Process Environment and provider-specific foreign collections.

There is **no confirmed cross-family contradiction in the normative contracts examined**. There are three real *discoverability/ergonomic boundaries* worth explicit documentation and, only if concrete users require it, a later owner decision: (1) `for` only works on values with `each`, so standard `Vector`/`String` are excluded; (2) `each` is a dispatch point, **not** a promise of a common result, callback arity or snapshot strategy; (3) index grammar handles one expression, so `Matrix.at(row, column)` has no multi-index bracket spelling. A further naming hazard is `add`: it cannot mean generic collection insertion because `Vector.add` and `Matrix.add` already mean algebraic addition.

**Recommended disposition:** **KEEP** existing public semantics; **DOCUMENT/CLARIFY** the optional capability matrix, especially `for`/`each` and the separate meaning of `add`; **DEFER** any new Core collection hierarchy, iterator/stream, universal mutator, syntax extension or default retroactive `each`. No existing Dxxx is reopened by this recommendation.

## Boundary and definitions

This audit distinguishes:
1. **Storage-family**: identity-bearing indexed or keyed state with a Core/standard contract (`Array`, `Bytes`, `Map`, `IdentityMap`, `Vector`, `Matrix`).
2. **Library role or algorithm module**: behavior layered over an existing family (`Set` over Map, `IdentitySet` over IdentityMap, `std:collections/array`, `std:collections/range`).
3. **Computed or immutable sequence**: `Range`/`InvertedRange`, `String`.
4. **Read-only keyed view**: Process `Environment`, not a Map.
5. **Provider projection**: foreign `at`/`atPut`/`each` where supported, not Core family membership.
6. **Composite domain object**: `Path` (immutable component sequence, no general collection API), `StyledText` (opaque frozen runs); URI, JSON, TOML, HTTP, filesystem and logging build *with* collections but do not standardize independent general collection families. Rest/Process argument values are ordinary, possibly frozen Arrays.

A protocol is **a selected message plus its owning contract**, not a class or an assumption that every object must implement it. Similar names across unrelated contracts are not automatically a shared protocol law.

## Matrix A — complete public-family coverage

| Family / role | Index read | Write / structural growth | Traversal | Identity / equality |
|---|---|---|---|---|
| Core `Array` | `at(i)` / `a[i]`, `size()` | `atPut(i,v)` existing slot only; no in-place resize | `each(element)` shallow ascending snapshot, returns receiver | identity by default; length fixed |
| Core `Bytes` | `at(i)` / `b[i]`, `size()` | `atPut` existing octet; `add(value)` +1; `removeAt(i)` -1; octets exact Integer 0–255 | `each(octet)` ascending snapshot, returns receiver | identity by default; length growable |
| Core `Map` | `at(key)` / `m[key]`, `size()` | `atPut(key,v)` insert or replace; `remove(key)` | `each(key,value)` insertion-order snapshot, returns receiver | identity by default; keys by `hash` + ordinary directed `==` |
| Core `IdentityMap` | same keyed selector surface | same structural capabilities | `each(key,value)` insertion-order snapshot, returns receiver | identity by default; keys by `identityHashOf` + `===` |
| `std:collections/set` | role-specific `Set.contains(set,element)`; underlying Map index remains exposed | `Set.add/remove(set,element)` mutate backing Map, return it | `Set.each(set,block)` one member, returns backing Map | Map family, membership by hash/==; `sameMembers` for content |
| `std:collections/identity_set` | analogous role | analogous mutation of backing IdentityMap | one member, returns backing IdentityMap | IdentityMap family, membership by identity |
| Core `Range` | no `at`/`size` | frozen, derived transform creates new Range | `each(integer)` / `eachStep` compute on demand, return `null` | ordinary object identity |
| Core `InvertedRange` | no `at`/`size` | same | descending `each`/`eachStep`, return `null` | ordinary object identity |
| `Vector` in `std:math/linear_algebra` | `at(i)` / `v[i]`, `size()` | `atPut` replaces numeric element; fixed shape; `add(other)` = mathematical sum, **not insertion** | no standard `each` | identity; fresh numeric results |
| `Matrix` in `std:math/linear_algebra` | `at(row,column)`, `rows()`, `columns()`; no 2-D bracket syntax | `atPut(row,column,v)`, fixed shape; `add(other)` mathematical sum | no standard `each` | identity; fresh numeric results |
| Core `String` | `at(i)` / `s[i]`, `size()` in Unicode scalar values | immutable; no standard `atPut`; string concat returns new value | no standard `each` | semantic Unicode value identity/equality |
| Process `Environment` | `get(name)`, `contains(name)`, no `at` | read-only snapshot; no `atPut` | `each(name,value)` canonical Unicode-name order; returns receiver | native name identity; not Map subtype |
| Foreign projected collections | optionally `at`, `atPut` | host/provider-defined | optional pull-based `each(item)`; returns facade/receiver | not Core Array/Map; no Core snapshot promise |

Set/IdentitySet are **module interfaces**, not independent Set/IdentitySet instance families. Their actual values are Maps/IdentityMaps with the invariant `member -> true`. They retain the Maps' own public indexing: direct writes to a member's marker may violate the library-role invariant. This is documented and ratified, not evidence of a hidden Set runtime defect.

## Matrix B — operations, results and semantics

| Operation / family | Effect and successful return | Coherence classification |
|---|---|---|
| `Array.atPut(i,v)`, `Bytes.atPut(i,v)`, `Vector.atPut(i,v)` | replace one existing value; return exact `v` | shared broad replacement convention, with family-specific domains |
| `Map.atPut(k,v)` / `IdentityMap.atPut(k,v)` | insert or replace association; return exact `v` | **intentional**: index assignment never implies only replacement |
| `Matrix.atPut(r,c,v)` | replace numeric cell, return exact `v` | arity differs; no bracket sugar, intentional D214 |
| `Bytes.add(v)` | grow receiver by octet, return exact `v` | Bytes-only structural insertion |
| `Vector.add(other)` / `Matrix.add(other)` | algebraic sum, fresh numeric object | independent algebraic selector: never standardize `add` as universal insertion |
| `Sets.add(set,v)` / `IdentitySets.add(set,v)` | mutate backing keyed collection; return that collection | module functions, not `set.add` |
| `Bytes.removeAt(i)` | mutate receiver and return removed octet | distinct from Arrays.removeAt |
| `Arrays.removeAt(array,i)` | leave source unchanged, return **new** Array | D213 copy/edit protocol |
| `Map.remove(k)` | remove entry and return previous stored value; absent -> Error | keyed domain |
| `Set.remove(set,v)` | delegate to Map.remove; return set; absent -> Error | separate module function |
| `Arrays.append(array,v)` / `Array(...array,v)` | new open Array, shallow elements; original length unchanged | deliberate solution for fixed-size Array |
| `String.concat` vs `Arrays.concat` | new String semantic value vs fresh Array; module/receiver dispatch differs | domain-specific composition, not shared in-place growth |

`[]` and indexed assignment each accept **one index expression**, and syntax lowers into ordinary `Send(receiver, "at", [index])` / `Send(receiver, "atPut", [index,value])`. The syntax imposes no resizing, key existence, result identity or return-value policy. Standard indexed *assignment expression* returns the assigned right-hand value, independent of any custom `atPut` return convention.

## Matrix C — mutation and lifecycle

| Kind | Open | Closed | Frozen / intrinsic |
|---|---|---|---|
| `Array` / `Vector` / `Matrix` | existing element replacement | existing element replacement | replacement forbidden |
| `Bytes` | existing replacement + append/remove | existing replacement only | all mutation forbidden |
| `Map` / `IdentityMap` | insert/replace/remove | existing-value replacement only | all keyed mutation forbidden |
| `Set` / `IdentitySet` | backed by Map/IdentityMap mutability | no membership change; marker replacement may remain allowed under Map rules | no membership change |
| `String` | immutable scalar sequence | not applicable as mutable indexed state | no indexed mutation |
| `Range` / `InvertedRange` | constructed frozen | constructed frozen | never mutate bounds |
| `Environment` | stable read-only Process snapshot | not a mutable Map | no native reload/mutation |
| Projected foreign objects | provider-defined | provider-defined | not inferred from Core family rules |

`close`/`freeze` are shallow object-state controls, not a universal "collection mutability" enum. An immutable String is conceptually distinct from a frozen ordinary Array; a closed Array is **not** immutable. A failed mutation must satisfy the owning family validation/commit rule; Map key hashing/equality may execute user code, so do not promise universal callback-effect rollback.

## Traversal semantics: **syntax uniform, iteration contracts not interchangeable**

- `for(x in e) { body }` is exactly an ordinary `e.each(x => { body })`. It neither discovers a generic iterator nor imposes count, snapshot, callback arity or result type. Numeric `for` uses `Range(...).each` / `eachStep`.
- Core Array/Bytes snapshot existing indexed state; Map/IdentityMap snapshot associations in insertion order; Set/IdentitySet wrap underlying Map iteration and expose *member* as the logical iteration value.
- Standard Map `each` passes **two** callback arguments; a contextual `for(key in map)` body binds the first (key). To access both, call `map.each((key, value) => {...})` directly.
- Standard Range and InvertedRange visit computed Integer values without proportional storage; return `null`, including successful empty traversal. An empty range can ignore an invalid callback when never reached, per its own ratified contract.
- Process Environment prevalidates callback callability and all native name/value representability before invoking callbacks; it enumerates canonically ordered Unicode names, and returns the receiver.
- Foreign projected `each` can be a *live pull* with provider-defined effects; no automatic Array/Map snapshot equivalence. It returns its exact Protos-facing receiver on success (D192).
- Standard `Vector`, `Matrix` and `String` do **not** currently publish `each`; they are index-readable (Matrix by two arguments) but **not** generic-`for` targets. This is a genuine affordance boundary, not proof of incorrect semantics.

Important consequence: it would be a **breaking change** to require that all `each` methods return the receiver, eagerly snapshot, validate callbacks before empty traversal, accept one argument, or be implemented by every index-readable value. Generic custom `each` is deliberately open. The smallest coherent law is the ordinary-call lowering, plus the specific owning family's published contract.

## Equality, hashing, aliasing and transfer

- Default equality/hash of mutable identity-bearing Arrays, Bytes, Maps, Sets' backing Maps and numeric Vector/Matrix remain identity-based; no implicit deep structural comparison.
- String equality observes its exact Unicode scalar sequence; Range instances use ordinary identity even with equal endpoints.
- `Map` key matching is ordinary hash+equality with representative-key retention and insertion order; `IdentityMap` is strict semantic identity; `Set.sameMembers` and `IdentitySet.sameMembers` are explicit content-comparison operations, not overrides of `==`.
- `Arrays.distinct` deliberately uses ordinary Set (`hash + ==`), **not** IdentitySet; the pinned `ArrayBulk.protos` and conformance tests agree. This was checked against a potential mistaken retrospective note.
- Fresh Arrays/Bytes/Vector/Matrix result contracts protect identity and independent indexed storage where specified; shallow copying preserves element references. Frozen rest-parameter and Process-arguments Arrays are ordinary Arrays with restrictive state, not new types.
- Actor/P crossings have separate family-specific transfer rules. This audit does **not** infer that an arbitrary foreign object, Process Environment, opaque StyledText or live resource has the same transferability as an Array. Per-family authority and isolation remain normative.

## Exact inconsistencies vs deliberate differences

| Finding | Observation | Disposition | Confidence |
|---|---|---|---|
| F1 | Indexed syntax lowers via `at`/`atPut` while Map inserts and Array replaces | **KEEP**: correct receiver-owned policy | High |
| F2 | Fixed Array vs resizable Bytes/Maps | **KEEP**: repeated owner-approved constraint | High |
| F3 | `add` has mutation, algebraic and module-role meanings | **KEEP + DOCUMENT**: do not claim one generic add law | High |
| F4 | Generic `for` unavailable on String/Vector despite indexed readability | **DOCUMENT**, future optional domain-specific `each` candidate **only after user need** | High for current absence; medium for ergonomic impact |
| F5 | Matrix `at(row,column)` cannot use `matrix[row,column]` or `matrix[row][column]` | **KEEP** D214; future syntax is separate approval-gated option | High |
| F6 | `each` callback arity, result and snapshot differ, especially Range/Map/foreign | **DOCUMENT**; **reject** retroactive universal result/snapshot law | High |
| F7 | Set role leaves raw Map operations exposed | **KEEP + DOCUMENT** invariant; avoid pretending runtime Set type | High |
| F8 | Process Environment uses `get`/`contains` rather than Map selectors | **KEEP**: native-name identity and absence/representation errors differ | High |
| F9 | `Array` algorithms are module functions while Bytes/Vector expose own methods | **KEEP** current D213/D214 topology; no implicit receiver aliases | High |
| F10 | `String` immutable but `Bytes` mutable; String has scalar indexing and no `each` | **KEEP**, document distinct text/byte boundary | High |
| F11 | Computed Ranges have neither size nor indexed access | **KEEP**: arbitrary precision/no proportional materialization; no demanded pseudo-collection type | High |
| F12 | Foreign projections share spellings but not Core membership/snapshot guarantees | **KEEP** with explicit provider documentation | High |
| F13 | `Path`/`StyledText` have ordered internal content yet intentionally opaque public APIs | **KEEP / OUT OF GENERIC COLLECTION SCOPE** | High |

**Confirmed source-vs-normative contradictions: none found in the audited surfaces.** This is not a statement that full runtime conformance or exhaustive dynamic testing passed; this research did not execute anything.

## Alternatives and comparative evaluation

**A — Preserve semantics, no new reference documentation.** Lowest edit cost, but the same naming and `for` surprises recur. **Not recommended as the final communication outcome.**

**B — Preserve semantics; publish a common *capability/contract map* and focused examples, without code changes.** Minimal, reversible, Protos-first, no runtime overhead or new public protocol institution. **Recommended immediate direction; documentation work only after normal scoped authorization.**

**C — Add narrowly justified `each` to String and/or Vector, or explicit Matrix traversal APIs, independently.** Ergonomics improve in demonstrated cases; however callback domain, result, snapshot, error priority, iteration order and Actor/P visibility require their own **Dxxx owner decision**, not silently automatic adoption. **Defer pending concrete consumer evidence; do not implement under AUD023.**

**D — Generic Collection/Iterable/Iterator/MutableSequence/Stream hierarchy with universal mutators and equality.** Potential generic algorithms at high semantic/institutional cost. Conflicts with historical explicit **RETAIN ABSENCE** decisions; risks `Range` memory behavior, foreign lifetime, different error timing, `add` algebra, fixed-shape numeric API and Set representation. **Reject as premature, not a newly ratified permanent prohibition.**

| Axis | A | B | C | D |
|---|---|---|---|---|
| Preservation of ratified semantics | Excellent | Excellent | Requires fresh approval | Conflicts / major redesign |
| Current-user cognitive clarity | Weak | Strong | Stronger in scoped examples | Ambiguous/complex |
| Protos ordinary-message fit | Strong | Strong | Possible | Weak unless proven |
| Runtime allocation/performance overhead | None | None | May add callbacks/snapshots | High potential |
| Implementation/maintenance cost | Lowest | Low | Per-family medium | High |
| Compatibility/migration exposure | None | None | New API, possible shadowing | Large |
| Callback/error/suspension correctness | Unchanged | Unchanged | Contract to research | Multiple new contracts |
| Equality and identity preservation | Strong | Strong | Can preserve | Risks implicit structural laws |
| Actor/P transfer resilience | Unchanged | Unchanged | Requires verification | High additional boundary cost |
| Grow-as-you-need and reversibility | Poor clarity | Excellent | Good when justified | Poor |
| Future extensibility | No added guidance | Strong optionality | Scoped | Hardcoded cross-family premise |
| Evidence of current need | Existing | Repeated confusion + issue | Insufficient for exact API | None |

**Strongest argument against B:** documentation cannot make `for` work on an indexed String/Vector; repeated manual `Range` loops may be awkward. **Response:** record concrete use cases and quantify their recurrence before adding domain-specific ordinary `each`; the latter can be designed without changing Array, introducing generic iterators, or requiring universal mutation. Another risk is that public docs themselves accidentally promise a stronger uniform `each` law than exists; the contract map must explicitly state where laws stop.

## External precedents (comparison, not authority)

- [Pharo by Example, collections](https://files.pharo.org/books-pdfs/updated-pharo-by-example/2016-10-27-UpdatedPharoByExample.pdf): distinct fixed-sized Array and growable OrderedCollection, despite both being indexable; supports the mutability-vs-resizability distinction.
- [Ruby Enumerable](https://ruby-doc.org/3.4.1/Enumerable.html): algorithms supplied on top of an `each` contract; useful analogy, but Protos does not inherit Ruby's mixin or implicit iteration requirements.
- [Python collections.abc](https://docs.python.org/3/library/collections.abc.html): Iterable, Sized, Sequence, MutableSequence, Mapping, MutableMapping are separate roles; demonstrates expressive protocol separation but not the need to copy an ABC hierarchy.
- [Swift RangeReplaceableCollection](https://developer.apple.com/documentation/Swift/RangeReplaceableCollection): structural insertion/deletion capabilities distinguished from basic Collection; a precedent for separating replacing existing positions from changing length.
- [Rust IntoIterator](https://doc.rust-lang.org/std/iter/trait.IntoIterator.html): `for` needs an explicitly satisfied traversal contract, with separate iteration ownership; Protos chooses direct `each` dispatch instead of copying iterator state.
- [Java SE 25 Collection](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/Collection.html): universal interfaces can contain optional mutators that fail at runtime; an alternative with more common surface but increased runtime surprise. Do not infer it is better for Protos.

## Explicitly preserve prior ratifications

- **AUD009-B9 / #620:** Core Array's dense fixed length, simple `size/at/atPut/each`, Map and IdentityMap semantics and snapshots were classified KEEP; growth and universal collection structures were deliberately absent.
- **AUD009-E1 / #646:** small source-backed Array, Set, IdentitySet and Range libraries kept; no generic Iterator/Sequence/Stream, backend views, Set runtime type, growable Array.
- **D213 / #949 and LIB023 / #953:** nonmutating fresh-copy Array algorithms, no new Core growth or hierarchy.
- **D214 / #955:** fixed-shape Vector/Matrix; `add` numerical, Matrix no bracket index; numeric element admission strict.
- **D231/D232/D235/D236:** `for` is ordinary `each` dispatch; canonical frozen directional Core Ranges and their exact traversal/result semantics.
- **D188/D192:** foreign projections do not inherit standard Array/Map semantics; projected `each` returns the Protos-facing receiver.
- **D168 / Process Environment:** distinct, stable, read-only Process snapshot with native name identity and representability validation.

## Smallest practical follow-up and non-goals

1. Owner reviews findings and classifications; **do not treat this audit as approval**.
2. If desired, scope one **documentation-only** child/slice to add a brief matrix in maintained guide/library docs, explicitly stating that generic `for` is just `each` and that `add` has no universal meaning. This itself creates no new API.
3. Independently collect real use cases in which `for` on `Vector` or `String` would materially simplify ordinary Protos without surprising Unicode iteration or numeric performance. A **new Dxxx** is required *if and only if* a concrete public protocol is proposed later; AUD023 does not implement one.
4. No mutable `Array` growth, no generic interface hierarchy, no semantic rewrite of `Set`, no new indexed syntax, no implicit conversion, no hidden iterator, no renaming or deprecating `add`, no modifying or removing prior conformance tests.

## Reproducibility and audit boundaries

The evidence record lists the exact audited source/spec/test paths and links. Source files were inspected read-only at the product commit above, with historical decision records in `guillermomolina/protos-project-docs`. Published test files demonstrate intended assertions; **none were executed here**. No performance numbers were measured, and none are claimed. No normative product source changed; no Protos version bump or changelog update is applicable.

**Proposed work disposition:** `AUDIT_CLASSIFICATIONS=PROPOSED_FOR_OWNER_REVIEW`; `PROTOS_SEMANTIC_CHANGE=NONE`; `MUTABILITY_REFORM=NOT_PROPOSED`; `DXXX_RATIFICATION=NONE`.
