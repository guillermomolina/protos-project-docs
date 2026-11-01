# D232 — Directed Range / InvertedRange research and owner-invariant evidence (2026-11-01)

**Authority:** non-normative comparative/design evidence for [D232 #1014](https://github.com/guillermomolina/protos/issues/1014). Owner's selected invariants are recorded in the active conversation and the D232 Issue; this file does not by itself authorize implementation. **Product baseline:** [guillermomolina/protos@fba47c40c4b79b6e580df1c1f78a3fcecbd067e2](https://github.com/guillermomolina/protos/commit/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2). No code/tests/benchmarks executed for this evidence publication.

## Evidence and compatibility baseline

The current source-backed `protos/lib/collections/Range.protos` exposes only `each(start,stop,block)` and `reverseEach(start,stop,block)`; neither stores a first-class range. LIB017 G-prime's [ratified decision](https://github.com/guillermomolina/protos-project-docs/blob/main/docs/project/work/LIB017/LIB017_RANGE_API_DECISION.md) explicitly deferred first-class values and steps. `spec/semantics/CALLABLES.md` defines flexible Closure argument binding, extra supplied arguments evaluated and ignored, and normal `Object.call`/instance `init` construction. Hence overloading the old helper by arity would silently change an already valid call. `spec/semantics/OBJECT_MODEL.md` mandates ordinary single-parent delegation and preserves identity/reflection; `spec/semantics/VALUES_AND_COLLECTIONS.md` defines arbitrarily precise semantic Integer membership and ordinary object identity. `MODULES.md` describes import as returning the ordinary source-backed module context, not an invented static class.

Actual checked-out product source references:
- [Range implementation](https://github.com/guillermomolina/protos/blob/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2/protos/lib/collections/Range.protos)
- [Range core conformance](https://github.com/guillermomolina/protos/blob/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2/protos/tests/conformance/library/collections/range-core.protos)
- [Range callback/control conformance](https://github.com/guillermomolina/protos/blob/fba47c40c4b79b6e580df1c1f78a3fcecbd067e2/protos/tests/conformance/library/collections/range-callbacks-and-control.protos)
- Known examples requiring migration: `protos/examples/algorithms/fizzbuzz.protos`, `100-doors.protos`, `protos/examples/closures/value-capture.protos`, `first-class-functions.protos`. This is a source search sample, not a proof about external users.

## External primary-source design families

| Language / reference | Relevant semantic model | Contrast |
| --- | --- | --- |
| [Python range](https://docs.python.org/3/library/stdtypes.html#range) | Directed lazy bounded integer sequence with signed step, excludes stop | Strong simple progression; direction from step, not separate prototype |
| [Swift stride](https://developer.apple.com/documentation/swift/stride(from:to:by:)) | Exclusive `to`, signed `by`; `through` is separate inclusive bound | Supports explicit endpoint policy; not Protos's object/prototype choice |
| [Kotlin progressions](https://kotlinlang.org/docs/ranges.html) | `downTo`, positive-magnitude `step`, `reversed()` | Separates direction from stride, but does *not* demand signed negative step in descending case |
| [Rust step_by](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.step_by) / [rev](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.rev) | Independent iterator adapters; order of composition matters | Opposite of conflating range reversal with reversal of sampled progression |
| [Ruby Range](https://docs.ruby-lang.org/en/master/Range.html) | Range object and interval iteration; `step` | Precedent for an interval object with messages |
| [Java IntStream.range](https://docs.oracle.com/en/java/javase/25/docs/api/java.base/java/util/stream/IntStream.html#range(int,int)) | Half-open counted ascending interval | Good indexing default but cannot directly model both orientations |
| [C++ ranges reverse / stride](https://en.cppreference.com/w/cpp/ranges/reverse_view.html) | View adapters that compose lazily | Powerful; an unnecessary generalized lazy-view framework for Protos today |

Three distinct families: directed signed progression (Python/Swift), oriented progression (Kotlin), and interval/value/iterator adapter composition (Ruby/Rust/C++/Java).

## Precise GITHUB021 mathematical counterexamples

For strict unit-step directed intervals, where `a <= b`:
- `Range(a,b)` visits all integers `a <= x < b` in ascending order.
- `InvertedRange(b,a)` visits all integers `a < x <= b` in descending *coordinate order* (`b,b-1,...,a+1`); the written tuple is a directed pair, not conventional ascending real-interval notation.
- `invert` swaps the two constructor arguments and flips orientation, so it does **not** promise the same set: `Range(0,3)` visits `0,1,2`; `invert` visits `3,2,1`. `InvertedRange(3,0).invert()` visits `0,1,2`.
- `reverse` flips orientation and *preserves the set*: `Range(a,b).reverse()=InvertedRange(b-1,a-1)`; `InvertedRange(b,a).reverse()=Range(a+1,b+1)`. Thus `Range(0,3).reverse()` visits `2,1,0` and `InvertedRange(3,0).reverse()` visits `1,2,3`.
- Both transforms are involutive on oriented limits, not necessarily referential object identity: `r.invert().invert()` and `r.reverse().reverse()` agree in limits/orientation with `r`. The shifts by 1 use unbounded Integer arithmetic, including for negative and equal bounds.
- For `Range(0,11)`: `eachStep(3)` -> `0,3,6,9`; `invert().eachStep(-3)` -> `11,8,5,2`; `reverse().eachStep(-3)` -> `10,7,4,1`. Neither reversed traversal is the exact reversal `9,6,3,0` of the *sampled* positive-stride progression; `reverse` operates on the unit interval before sampling.
- Wrong-signed steps and step=0 -> generic Error, even when there are no elements; strict genuine semantic Integer steps only. Each `each` is unit step respecting object orientation.

## Deliberately reopened LIB017 invariants vs preserved ones

**Owner-approved replacement**: retire the two public module helpers instead of keeping wrappers; introduce two directly constructible ordinary directional range value/prototype categories. This is deliberately incompatible with the ratified LIB017 G-prime public surface, and external consumers cannot be proved migrated.

**Preserved semantics**: half-open boundary meaning per directed traversal, exact arbitrary-precision Integer arithmetic, callback single-argument ordinary invocation, null normal result, callback result ignored, Error identity/nonlocal-return/ensure propagation, no implicit Future await and no array/progression materialization proportional to cardinality. Reversed constructor bounds historically meant *empty* in old helpers; rejecting such constructors is a separate observable compatibility choice, not silently assumed settled.

**Not yet separately ratified by the owner's exact two-transform approval**: names and accessibility of exported prototype values vs module `call`; wrong-ordered endpoint rejection and exact empty interval normalization; externally observable parent/delegation topology and the structuring/freezing of stored bounds; constructor/selector arity/error precedence; mutation invariants for ordinary objects (ordinary `freeze()` is shallow); conflict between normal module-context parents and standalone prototype parents. No agent may turn these questions into new normative commitments through a casual code-lowering choice.

## GITHUB010 decision comparison (qualitative scores, 1=weak 5=strong)

These are *reasoned estimates*, not runtime measurements. Confidence H=high, M=medium. Alternatives: A old helpers only; B named step helpers preserving old API; C one descriptor carrying direction; D distinct first-class Range and InvertedRange with both transformations (owner-selected).

| Criterion | A | B | C | D |
| --- | --- | --- | --- | --- |
| Correctness/invariants | 5H unchanged | 5H exact | 5H exact | 5H exact |
| Protos alignment | 5H ordinary | 5H ordinary | 5H ordinary | 4H prototypes ordinary |
| Present proportionality | 2H no stride | 5H local | 3M descriptors | 3M extra visible distinction |
| Incremental growth | 4H possible | 5H additive | 5M extensible | 5M reusable |
| Future options | 5H open | 5H open | 5M variants | 4M two public families |
| Scalability | 5H compact | 5H compact | 4M per descriptor | 4M per descriptor |
| Conceptual simplicity | 5H narrow | 4H two helpers | 4M one type | 4H two clear orientations |
| Portability | 5H Protos | 5H Protos | 5H Protos | 5H Protos |
| Runtime resources | 5H no descriptors | 5M constant state | 3M object cost | 3M object cost |
| Failures/operability | 3H manual stride | 5H explicit sign | 5M sign validation | 5M sign validation |
| Migration/reversibility | 5H no break | 5H additive | 3M new API | 2H breaks helpers |
| Evidence/risk | 5H implemented | 5H conventional | 4M feasible | 4M feasible |

**Non-compensating red flags**: D introduces a compatibility break and two first-class object forms where B needs only new helpers; the selected semantic separation of practical inversion vs element-preserving reversal and user request to eliminate redundant helpers are the concrete present demands paying that cost. No benchmark supports a performance gain. Strongest counterargument is B (only helpers): much less migration cost, ordinary Protos implementation, and no object/factory topology. The owner nevertheless chose D deliberately for first-class composition and intuitive construction. This is **selection evidence**, not a general license for more range operations.

**Future stress/falsification**: huge BigInteger endpoints/steps, empty/one-value ranges, wrong-sign/zero step, non-Integer and delegating lookalikes, nested callbacks, exact exception propagation, `^` lexical home returns, Task/Future callbacks, independent Actor imports, repeated invert/reverse and algebraic identity, frozen vs mutable bounds, equality/hash/reflection, constructor overload ambiguity and external consumers. No synthetic `Iterator`, grammar, generic `Collection` hierarchy, Java/Truffle primitive or C++ views are justified by the selection.

## Status / provenance

Owner's **“ok apruebo esto último”** selected both `invert()` and `reverse()` after their exact illustrated outputs; earlier conversation established direct `InvertedRange` construction and signed step semantics. [Live D232](https://github.com/guillermomolina/protos/issues/1014) owns any remaining contract questions. Normative spec and implementation remain unchanged until separately authorized. No test run, version bump or product publication is claimed.
