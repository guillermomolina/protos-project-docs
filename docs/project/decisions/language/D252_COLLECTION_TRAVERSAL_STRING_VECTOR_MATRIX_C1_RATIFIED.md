# D252 — Standard `each` for String, Vector and Matrix (C1)

**Decision:** **D252-C1 explicitly approved by owner 2026-11-09; durably ratified as a design choice, implementation pending.**
**Governing issue:** [D252 #1098](https://github.com/guillermomolina/protos/issues/1098).
**Exact owner approval:** [D252 approval-provenance comment](https://github.com/guillermomolina/protos/issues/1098#issuecomment-6731616772); owner's exact statement: **“perfecto me gusta C1, apruebo”**.
**Motivation:** [AUD023 #1087, historical B → preferred C correction](https://github.com/guillermomolina/protos/issues/1087#issuecomment-6731237139).
**Approval and invariant evidence:** [D252 C1 evidence](../../evidence/D252/D252_C1_OWNER_APPROVAL_AND_INVARIANT_RECONCILIATION_2026_11_09.md).
**Product specification/source baseline:** [guillermomolina/protos@52b6da5da98e7eedb525ab4eb5111258c136193e](https://github.com/guillermomolina/protos/commit/52b6da5da98e7eedb525ab4eb5111258c136193e). At this baseline, String, Vector and Matrix have **no standard `each`**. This record changes no product `spec/`, sources, tests or version.

## 1. Selected public behavior

Add an **ordinary** `each(block)` message to each of these existing receiver domains:

| Receiver | Publishing owner | Element supplied to callback | Order | Successful normal result |
| --- | --- | --- | --- | --- |
| Semantic `String` | Core standard String prototype | Semantic `String` containing exactly one Unicode scalar | Ascending scalar position, consistent with `String.at(i)` | Exact original String receiver |
| `Vector` | `std:math/linear_algebra` Vector prototype | The genuine Number stored in that logical position, unchanged | Ascending index `0 .. size-1` | Exact original Vector receiver |
| `Matrix` | `std:math/linear_algebra` Matrix prototype | The genuine Number stored in that logical cell, unchanged | Row-major: row ascending, then column ascending | Exact original Matrix receiver |

There is exactly **one positional callback argument per visited element**. Values and callback results are never coerced. Callback return values are ignored for traversal. No hidden shape conversion or numeric reboxing rule is authorized.

For String, "Unicode character" means **Unicode scalar**, *not* a byte, UTF-16 unit, grapheme cluster, or numeric codepoint. Exactly the elements returned successively by `String.at(i)` are visited; `"e\u{301}"` has two visits and the family emoji sequence has seven. The String receiver is immutable; implementers may traverse a cursor without making a copy.

Vector and Matrix establish **one shallow logical indexed snapshot before invoking any callback**. For a Vector, snapshot order is ascending index. For Matrix, snapshot order is row-major. Later `atPut` mutations, including reentrant or other Actor-local mutations during explicit suspension, do not change pending callback values, count or order. A snapshot captures the scalar Number values/references, not an immutable copy of the enclosing object; it does not lock the receiver, close/freeze it, or forbid otherwise legal mutation. Storage optimizations (e.g. Matrix R1 vs R2-Float) must be observationally invisible.

Matrix empty shapes remain distinct: `Matrix(0,n)` and `Matrix(n,0)` both visit zero values; they preserve their exact original rows/columns, and both return their own receiver on normal completion. No row objects, views or coordinate pairs are supplied by `each`.

## 2. Validation, control transfer and object model

After **ordinary expression evaluation and standard message selection**, the standard method:
1. Requires a receiver belonging to that method's actual semantic family; merely delegating to the standard prototype does not fabricate String/Vector/Matrix indexed state.
2. Requires exactly one supplied argument and validates its ordinary **polymorphic invokability**, not Closure-only status, **before iteration, including empty inputs**.
3. For Vector/Matrix, establishes the logical shallow snapshot only after successful receiver, arity and callback validation.
4. Invokes `block(element)` once for each position in order, ignoring normal callback results. On normal exhaustion returns the **exact receiver**.

Invalid receiver, argument count, or callback signals ordinary `Error` without invoking a callback. The validation order is receiver → arity → invokability → snapshot → callbacks. Internal resource or physical-representation failures must not leak JVM/host exceptions as an alternate guest contract.

A reached callback's `Error`, valid non-local return, `ensure` cleanup, explicit suspension/reentrancy or cancellation follow **existing** ordinary invocation/control rules. No callbacks after a nonlocal transfer, and already-completed callback effects are not rolled back. Suspension must retain cursor and snapshot; resumption must not restart a callback or silently await a Future returned by an ordinary callback. Neither `each` creates a Task/Actor/P computation nor mandates parallel work.

Object lookup/delegation/shadowing remains ordinary, including receiver-specific custom `each` slots. Open/closed/frozen object semantics are unchanged. Reading a closed/frozen Vector or Matrix is allowed; mutating one is permitted or rejected by its existing `atPut` state contract, independently of iteration. String remains intrinsically immutable. Actor and isolated P transfers preserve their existing copy/isolation rules; an iteration snapshot is **not** an Actor/P transfer snapshot.

## 3. `for` and related operations

Generic `for(item in receiver) { body }` retains D231's exact lowering to `receiver.each(item => { body })`: this decision adds eligible receiver behaviors, **not grammar, special dispatch syntax, or a common abstract iterator**. Standard Map's two-argument callback, Range's lazily computed traversal/`null` result, foreign pull iteration, and other family-specific `each` contracts remain different and unchanged.

`Vector.add` and `Matrix.add` remain mathematical addition. D214 Vector length and Matrix shape are fixed, `atPut` only replaces existing elements; Matrix still uses `at(row,column)`, not multidimensional bracket syntax. D213 Core Arrays remain fixed-length and Array library copy/edit functions remain nonmutating. Set/IdentitySet backing and identity/equality semantics remain untouched.

No `Iterable`, `Iterator`, `Collection` or `Stream` hierarchy; no public row/column view, `eachRow`, `eachCell`, index+value callback, `map`, `filter`, grapheme traversal, automatic vectorization or parallel iteration. These are separable future decisions and may be added later without changing C1.

## 4. Illustrative post-implementation examples

These examples **do not run under the referenced pre-implementation baseline**:

```protos
for(c in "A😀B") {
    print(c)
}

LA: import("std:math/linear_algebra")
Vector: LA.Vector
Matrix: LA.Matrix

for(n in Vector(10, 20, 30)) {
    print(n)
}

for(n in Matrix(2, 2, 1, 2, 3, 4)) {
    print(n)
}
```

The expected visitation sequences are the scalar Strings `"A"`, `"😀"`, `"B"`; Numbers `10,20,30`; and Numbers `1,2,3,4`, respectively. Return values of each underlying `each` invocation are its receiver; the contextual `for` lowering remains unchanged.

## 5. Architecture/implementation freedom and risks

This is an **observable** contract, not an instruction to implement loops in a particular Java node. Existing Array/Bytes `each` have structured execution paths capable of preserving suspension; new traversal must be equally correct for the entire bytecode/task call path, including arbitrary ordinary callable values. A simple non-resumable host loop is insufficient if it cannot preserve guest suspension.

Avoid quadratic String iteration by repeatedly calling the current index-from-start `scalarAtForRuntime(i)`; use a sequential scalar cursor or equivalent O(n) traversal. Vector/Matrix snapshots have O(n) logical element storage/time implications and are invoked only when needed. A future optimized representation must retain the original normal order, exact Numbers and error/control behavior; no eager snapshot of the full Matrix for a zero-volume shape with huge nonzero dimension. Detect callback reentrancy and alternate receiver shadowing.

Large matrices may favor future separate no-copy/read-only or axis-specific operations, but those must have **distinct selectors/contracts** and must not weaken C1's snapshot promise. A future sparse Matrix would still visit all **logical** elements in row-major order, including implicit zeroes, if it implements this exact `Matrix.each` contract.

## 6. Authority, boundaries and follow-up

- **D157** String scalar indexing and Unicode domain: preserved.
- **D213** fixed-length Arrays and nonmutating copy/edit helpers: preserved.
- **D214** fixed-shape Vector/Matrix, standard algebra, strict admission, identity, transfer, row-major constructor and R1/R2 transparency: preserved.
- **D231** contextual `for` ordinary-message lowering: preserved.
- **AUD023** B approval and completed audit are retained as dated history; owner later changed the **future direction** to C. D252-C1 selects that new capability precisely without relabeling historical evidence.
- **DOC021 #1089** remains documentation-only, independently owned. Its pre-D252 text describing absent methods is historical baseline, not a prohibition on a subsequently owner-approved Ixxx. Public documentation must describe **what is actually shipped** until implementation publishes the normative `spec/` and code.

**Implementation belongs to a new Ixxx issue**, not D252 or the completed audit. No product code/spec edits, tests, benchmarks, or successful feature runtime execution are asserted by ratifying this decision. The authoritative observable implementation contract will be published through the canonical `spec/` owners in the Protos product repository during Ixxx.
