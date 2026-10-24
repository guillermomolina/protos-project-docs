# D214 — Vector and Matrix standard linear-algebra semantics (C1 / S1)

**Decision status:** OWNER APPROVED, 2026-10-24. **Normative publication:** PENDING in `guillermomolina/protos/spec/`. **Implementation:** NOT AUTHORIZED until that publication and its exact revision are recorded. **Decision issue:** [D214/#955](https://github.com/guillermomolina/protos/issues/955). **Approval checkpoint:** [eight owner-approved clauses](https://github.com/guillermomolina/protos/issues/955#issuecomment-6380265698). **Evidence:** [D214-S1 publication-gate checkpoint](../../evidence/D214/D214_S1_OWNER_APPROVAL_AND_PUBLICATION_GATE_2026_10_24.md).

This is a **non-normative decision/rationale record**. Observable Protos behavior becomes authoritative only when the matching text is published under `spec/` in the product repository. In particular, the approval and the maintainer's test report do not establish a D214 normative product commit.

## Selected architecture: C1

`Vector` and `Matrix` are ordinary identity-bearing Protos objects with private numeric indexed payloads. They are standard-module members of `std:math/LinearAlgebra`, not additions to Core's `Number` families, new syntax, or a generalized indexed-Object feature. Their frozen standard prototypes must follow the existing standard-member recognition and Actor/P transfer rules. The initial payload representation is private reference storage (R1); adaptive primitive storage (R2) may replace it later only if invisible to programs.

## Owner-approved S1 — exact eight semantic boundaries

1. **Numeric admission:** All genuine Protos `Number` values are supported: Small/Big `Integer`, `Fraction`, `Float`, `Complex`, including heterogeneous contents. Reject `null`, `Boolean`, `String`, delegation lookalikes, and foreign impostors. Constructors and every private indexed writer validate synchronously without conversion hooks or guest callbacks. `atPut` is an admission boundary too.
2. **Shape and construction:** Dense fixed-shape `Vector` and rectangular `Matrix`; dimensions and indices are exact nonnegative `Integer`. Vector has no row/column orientation. Preserve distinct `0×n` and `n×0` Matrix dimensions. `LA.Vector(value, ...)` takes the supplied values literally, so `LA.Vector(3)` is length one. `LA.Matrix(rows, columns, ...flatValues)` requires exactly `rows * columns` entries in row-major order, allowing explicit spread. Do not add filling, inferred shapes, or implicit zero elements.
3. **Indexed API and mutation:** `v.at(i)`, `v.atPut(i,value)`, and `v[i]`; `m.at(row,column)` and `m.atPut(row,column,value)`; `size` for Vector, `rows`/`columns` for Matrix. No `m[row,col]` syntax or ambiguous `m[row]` copies/views. `atPut` returns the assigned value on success. Check frozen state, then bounds, then genuine Number eligibility before mutation; never grow.
4. **Algebra API:** Vector: `add`, `scale`, `dot`, `copy`. Matrix: `add`, `scale`, `matvec`, `matmul`, `transpose`, `copy`. Enforce exact dimensional compatibility; produce fresh open results with no observable payload aliasing, no broadcasting, and no implicit promotion. Numeric kernels use canonical scalar arithmetic without guest-dispatched callbacks; publicly visible sends remain ordinary Protos dispatch.
5. **Strict Float reduction F1:** Compute products and sums in specified ascending inner-index order, without result-changing reassociation, FMA contraction, approximate backend substitution, or nondeterministic parallel reduction. Start a nonempty reduction with its first product (do not seed with `0`, which would change signed-zero behavior); an empty contraction is the exact `Integer 0`. Independent output cells may be computed in parallel if each retains its own ordered reduction.
6. **Complex rules:** Vector `dot` uses the Hermitian inner product and conjugates the **left/first** operand. Matrix `matvec` and `matmul` are bilinear, with no conjugation. Complex products follow the existing scalar Complex coordinate-arithmetic contract, including its finite-coordinate rounding requirements; sums follow F1 order. Do not silently change scalar IEEE Float/Complex semantics.
7. **Physical representation:** R1 is private references (e.g., `Object[]`), not public Array indexed state or a `dtype` protocol. No implicit numeric-family widening or coercion on writes. R2 primitive/adaptive storage is a subsequent optional optimization only, without observable semantic differences.
8. **Identity, mutability and Actor/P:** Default `===`, `==`, and `hash` obey ordinary object identity behavior unless independently overridden. `close` prevents structural growth but does not prevent existing indexed writes; `freeze` prevents them. `copy` and `transpose` create fresh independently mutable results. Actor/P may share frozen standard prototypes, but numeric indexed payloads and ordinary slots must follow graph-aware transfer: separate mutable copies without alias leakage while retaining graph identity/cycles. Private copy-on-write is permitted only if unobservable.

## Source-first contract and examples

```protos
LA: import("std:math/LinearAlgebra")
v: LA.Vector(1, 2.5, 3/4, Complex(1, 2))
m: LA.Matrix(2, 3, 1, 2, 3, 4, 5, 6)
data: [1, 2, 3, 4]
m2: LA.Matrix(2, 2, ...data)
```

`Vector(3)` is a one-element Vector, not a capacity constructor. `Matrix(2, 3, ...)` takes exactly six positional values. Storage and fast-path optimization are implementation details; user-observable operation order and numeric families are not.

## Normative and implementation publication gates

1. **Normative-publication slice:** Human executor edits `spec/semantics/VALUES_AND_COLLECTIONS.md` and adds a matching new entry to the current `spec/PROTOS_SPEC_CHANGELOG.md` in one spec-only product commit. No runtime implementation, `pom.xml`, root `CHANGELOG.md`, or unapproved language sugar. The normative change must explicitly preserve the fact that **Core** does not define Vector/Matrix while documenting the new **Standard Library** module.
2. **Evidence reconciliation:** After the human publishes that product commit, record its exact SHA and new spec revision in D214/#955 and in the durable D214 evidence; replace the pending normative marker. Never infer a SHA from a previous unrelated maintainer test report.
3. **Grouped implementation:** Only after step 2, allocate an appropriately classified single grouped Standard Library implementation issue via the current `AGENTS.md` rules, with focused slices for object/prototype/transfer foundations, Vector arithmetic and strict reduction, and Matrix arithmetic/integration. Product edits and tests remain human-executed.
4. **Closure:** Do not close D214 until publication, evidence, and release to implementation have been confirmed.

Related normative baselines: `spec/semantics/VALUES_AND_COLLECTIONS.md` Number families and Array indexed/mutation laws; `spec/semantics/MODULES.md` standard module framework; prior numeric decisions D196/D197/D198. No invented extra numbered D214 requirements.
