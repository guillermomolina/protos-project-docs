# I136 — Autonomous agent prompt: directional Range and InvertedRange (ONE implementation slice)

**Paste all of this prompt into an editing agent operating ONLY in an existing local `guillermomolina/protos` checkout.** The checkout **is already at HEAD**. Do not checkout, pull, fetch, clone, navigate to any other repository, browse the web, read an external file, or ask another agent to investigate. Everything external required for this implementation is reproduced below; authoritative current source and normative constraints are in the local HEAD.

## Your task and execution boundary

Implement `I136` (not a D design investigation): replace the old Range module iteration helpers with ordinary first-class `Range` and `InvertedRange` prototypes; add `each`, `eachStep`, `invert`, `reverse`; migrate every caller/test, update normative spec/docs/changelogs and version as appropriate. **ONE complete cohesive implementation slice**, one final product publication, no I136-A/B or intermediate feature publication. The user explicitly rejects leaving compatibility wrappers/aliases.

You edit source, specs, tests and docs only. **Human executor runs everything**: project programs, unit/integration/Native tests, Maven, Make, checking, validation scripts, environment-modifying commands, staging, commit and push. Do not run these, and do not run Git fetch/pull/checkout/merge/rebase or create remote branches. Read-only inspection and local file edits by the agent are allowed. Preserve unrelated changes; use repo-owned guidance even if this prompt seems to permit more. Human validation failures are iterated within the SAME slice.

**RATIFIED PRECONDITION SATISFIED.** D232's complete exact public contract was explicitly approved in the active 2026-11-01 dialogue: first **“ok apruebo esto último”** (dual inversion); then **“Si lo apruebo”** (constructor/exposure/ordered-bound/arity/reflection/frozen-instance contract). The full durable decision was published in `guillermomolina/protos-project-docs/docs/project/decisions/language/D232_DIRECTIONAL_RANGE_DUAL_INVERSION_SELECTED.md` at commit `064485be40fde8eef0db93f6ce6b8116b360b9a7`. **I136 is READY for one complete implementation slice.** Do not fetch that repository: the exact semantic contract appears below. If *new* materially observable ambiguities appear outside this ratified contract, report that specific new issue instead of inventing semantics.

### Approved constructor/export contract (included in full; no external read)

```protos
Ranges: import("std:collections/Range")
Range: Ranges.Range
InvertedRange: Ranges.InvertedRange

r: Range(0, 10)
ir: InvertedRange(3, 0)
```

The **module remains the ordinary imported module instance** with members `Range` and `InvertedRange` that hold two ordinary prototype objects. Normal Protos direct invocation of the extracted/bound prototypes uses inherited `Object.call`, which allocates an object with the invoked prototype as its immediate parent and calls `init(args)`. `r.parent() === Range`; `ir.parent() === InvertedRange`. A member expression such as `Ranges.Range(0,10)` is not guaranteed to act as an ordinary callable prototype under existing method-send rules: bind/extract the prototype as shown. There is no special grammar, Java factory, implicit Prelude addition or module-level `call` to introduce.

**Strict constructors**: each must receive *exactly two* genuine semantic unbounded Integer positional arguments `start`, `stop`. `Range(start,stop)` requires `start <= stop`; `InvertedRange(start,stop)` requires `start >= stop`; incorrect order and non-Integer inputs signal ordinary `Error`; equality produces an empty range. Zero/one/three-plus arguments are errors, including otherwise valid first two. **Protos evaluates receiver/explicit argument expressions once left-to-right before validation**; do not impose new evaluation behavior. Normal Closure binding ignores surplus arguments, so the actual `init` behavior must inspect a rest parameter (or an equally existing ordinary mechanism) and reject extras explicitly; missing required Integer arguments bind `null` and must fail validation. `Object.call` returns the initialized instance; `init` return value is ignored. For multiple simultaneous invalid inputs, do not add an unapproved public Error subclass or extra validation-precedence promise.

Each successfully constructed instance owns exactly `start` and `stop` as directly readable, reflectable **local slots containing the exact two supplied Integers** (in traversal order). After successful initialization, each instance is **structurally FROZEN** via ordinary `Object.freeze()`, so it cannot modify its own two slots, add new slots, or remove slots. The two prototypes are ordinary Protos objects, not new Core semantic/value-identity categories; instance `parent()` must report the appropriate prototype. `invert()` and `reverse()` produce fresh frozen instances without modifying their source; object identity `===` after double inversion is not guaranteed. Do not silently install a new `==` or `hash` behavior or new recognizer; standard object identity/reflection/delegation semantics apply. The particular internal `init` validation check order beyond ordinary evaluated argument effects and required generic Error is an implementation choice, not a new normative ordering guarantee.

## Exact owner-selected contract (2026-11-01)

Product initially had `std:collections/Range` with module-local `each(start,stop,block)` and `reverseEach(start,stop,block)` helpers, both on ascending `[start,stop)` with unit step. **The owner explicitly chose to REMOVE both public module helpers**, not retain wrappers; existing external consumers may break. The prior LIB017 record is historical, do not rewrite it.

Both new values are ordinary first-class prototype-based Protos objects and can be directly constructed (not Java special classes, primitive numeric values, special AST or language syntax). For mathematical `a<=b` in true unbounded semantic Integer:

```text
Range(a,b).each(block)          a,a+1,...,b-1
InvertedRange(b,a).each(block)  b,b-1,...,a+1
Range(0,3)                     0,1,2
InvertedRange(3,0)             3,2,1
Range(0,0)                     empty
InvertedRange(0,0)             empty

Range(a,b).invert()           -> InvertedRange(b,a)
InvertedRange(b,a).invert()   -> Range(a,b)
Range(0,3).invert()            InvertedRange(3,0): 3,2,1
InvertedRange(3,0).invert()    Range(0,3): 0,1,2

Range(a,b).reverse()          -> InvertedRange(b-1,a-1)
InvertedRange(b,a).reverse()  -> Range(a+1,b+1)
Range(0,3).reverse()           InvertedRange(2,-1): 2,1,0
InvertedRange(3,0).reverse()   Range(1,4): 1,2,3
Range(0,0).reverse()           InvertedRange(-1,-1): empty
Range(0,0).reverse().reverse() Range(0,0): empty

Range(0,11).eachStep(3,block)                  0,3,6,9
Range(0,11).invert().eachStep(-3,block)        11,8,5,2
Range(0,11).reverse().eachStep(-3,block)       10,7,4,1
Range(0,11).eachStep(-3,block)                 Error
Range(0,11).invert().eachStep(3,block)         Error
Range(0,0).eachStep(0,block)                   Error
```

**Distinguish the transforms**: `invert()` swaps endpoints and direction, generally changes the visited member set; `reverse()` changes direction but retains EXACTLY the original unit-step members. The two transformations are involutions on logical descriptor, but a new object does not have to be `===` to the original. Reverse then step is not reversal of previously selected stride values (`0,3,6,9` reversed is `9,6,3,0`; `Range(0,11).reverse().eachStep(-3)` is `10,7,4,1`). Each traverses from the included initial endpoint toward the excluded second endpoint, not from a numeric min/max inferred from `step`.

`Range.eachStep` only accepts genuine Integer step `>0`; `InvertedRange.eachStep` only accepts genuine Integer step `<0`. Zero, contradictory sign and invalid Float/Fraction/Complex, delegates-to-Integer impostors signal ordinary Error even for empty sequences. Inclusive initial endpoint, exclusive final endpoint in both orientations; no overflows, host-width limits or implicit coercions. No proportional Array materialization. `each` is semantically unit-step with sign chosen by object, not positive for both.

Callbacks keep the ratified LIB017 contract: one ordinary polymorphic `block(integer)` call per iteration; block return ignored; normal range iteration result is null; receiver/call argument evaluations occur exactly once left to right; empty iteration does not probe whether callback is invokable; reached callback Error identity and lexical `^` nonlocal-return plus `ensure` unwind propagate; no forced Future awaiting or Task orchestration, no callback retry. Transforms have pure result semantics and do not mutate their source descriptors.

No additional `contains/size/at/array/iterator`, lazy views, `for` grammar, float ranges, inclusive constructors, generic comparable ranges, Java/Truffle fast paths, concurrency primitives, nor syntactic sugar.

## Local source audit and files to edit

At start inspect `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/LIBRARY.md`, `AGENTS.work/REFERENCE.md`, `protos/AGENTS.md`, `spec/AGENTS.md`, relevant blocker ledger, current `spec/PROTOS_LANGUAGE_SPEC.md`, `spec/semantics/{VALUES_AND_COLLECTIONS,OBJECT_MODEL,CALLABLES,MODULES,EXECUTION_AND_CONTROL,ERRORS}.md`. Read current files instead of assuming their line numbers, spec revision or library code from 2026-11-01.

Product baseline at handoff `fba47c40c4b79b6e580df1c1f78a3fcecbd067e2`, version `0.3.429-SNAPSHOT`, but **do not checkout that SHA**, use provided local HEAD. Baseline `protos/lib/collections/Range.protos` contained simple integer-validated loops with module top-level `each`/ `reverseEach`. Baseline tests:
- `protos/tests/conformance/library/collections/range-core.protos`: invalid Fraction/Float/delegation bound rejection, evaluate endpoints once, unit order and `null`, BigInteger endpoints, polymorphic callback `call`.
- `protos/tests/conformance/library/collections/range-callbacks-and-control.protos`: exactly one supplied callback argument, error identity, noninvokable callback on empty, `^` nonlocal return.
- Example consumers `protos/examples/algorithms/fizzbuzz.protos`, `100-doors.protos`, `protos/examples/closures/value-capture.protos`, `first-class-functions.protos`.
Search current local HEAD for every remaining import/helper use and update source/docs/tests coherently. Do not change an algorithm's mutable bound semantics by naively snapshotting an expression.

For Protos ordinary prototype construction, `spec/semantics/CALLABLES.md` establishes `Object.call` as default object allocation with parent equal to the constructor receiver, then an ordinary `init(...args)` message; `Object.init` with inherited zero arguments is not a two-arg constructor. The module instance is an ordinary source-backed module context per `MODULES.md`. The existing `protos/lib/math/LinearAlgebra.protos` models library exported `Vector` and `Matrix` prototypes and documents that `LA.Vector(1)` calls a member directly, which is NOT the same as `Vector: LA.Vector; Vector(1)`. Check current HEAD for ordinary frozen prototype export/bootstrap patterns and the exact chosen D232 gate, then implement in Protos source without privileged host changes.

`Integer.recognizes(value)` returns true only for genuine semantic SmallInteger/BigInteger (not an object that merely delegates). `spec/semantics/VALUES_AND_COLLECTIONS.md` owns normal Integer semantics; `OBJECT_MODEL.md` owns delegation, object-state/reflection/identity, and `CALLABLES.md` owns flexible argument handling, standard Object.call/init, dispatch, callback, `this` and `^`. `MODULES.md` owns import and module instance. No new parser or Java implementation is required.

Implement all needed source/library/prototype changes in `protos/lib/collections/Range.protos` and tightly necessary adjacent paths; migrate all Range examples and tests; add cohesive tests for constructors, parent/reflection (as selected), each, both transforms, signed stride, non-integer, empty, BigInteger and callback/control. Use repository's current test-corpus registration; source tests as semantically coherent groups. Update canonical normative Standard Library specification owner and required `spec/PROTOS_SPEC_CHANGELOG.md`, docs/user guide and source doc comments. Later after human validation passes, bump current Protos SNAPSHOT and CHANGELOG exactly once according to actual HEAD. Keep all changes in ONE final product commit.

## Human executor handshake / validation

Give human bounded commands to run specific current test-tool selectors/files first; human then runs `make test`, `make check`, `git diff --check` and required Native smoke (only where impact policy calls for it). Human reports PASS/failure. You modify source to fix failed cases within I136 only. Human reviews exact file list and executes explicit-path `git add`, single `git commit`, `git push origin main`; agent never executes these. Do not claim tests, commits, publication or performance without human evidence. Final report must include changed files, exact contract check, any unstated blocker, actual reported PASS and immutable product commit after publication.

## External source packet: no research or browsing required

- Python `range(start,stop,step)`: sign of step selects direction; stop exclusive. D232 intentionally differs by fixing orientation on the object. https://docs.python.org/3/library/stdtypes.html#range
- Swift `stride(from:to:by:)`: signed step and exclusive stop vs `through` inclusive. D232 keeps traversal first-inclusive, last-exclusive. https://developer.apple.com/documentation/swift/stride(from:to:by:)
- Kotlin `downTo`, `step`, `reversed`: oriented progression; uses POSITIVE magnitude for `step` in either direction, unlike D232's signed-step restriction. https://kotlinlang.org/docs/ranges.html
- Rust `Iterator::step_by` and `rev`: distinct adapters; reversing the unit interval before sampling is different from reversing an already sampled set. https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.rev
- Ruby `Range`: first-class interval with iteration messages. https://docs.ruby-lang.org/en/master/Range.html
- GraalPy (Truffle Python implementation): language-level range values can be backed by host objects/iterators, but its representation and signed Python range behavior are **not** Protos normative authority. https://www.graalvm.org/reference-manual/graalpy/
- TruffleRuby: Truffle Ruby implementation with Ruby Range behavior; not evidence Protos must add RangeNodes or Java fast paths. https://github.com/truffleruby/truffleruby
- GraalJS/Truffle DSL: optimization may use `@Specialization` with guarded selector provenance, only if a later separately approved performance task requires it; DSL rules caution against duplicated specialized code and incorrect cache/uncached divergences. https://www.graalvm.org/dev/graalvm-as-a-platform/language-implementation-framework/DSLGuidelines/

**Do not open these links.** These details are all external comparison needed. The authoritative implementation mechanism is ordinary Protos current-HEAD modules/objects/closures, not foreign Truffle internals. No cross-repository access, additional research, or split implementation is required: the three exact D232 construction gates are now owner-approved and included above.
