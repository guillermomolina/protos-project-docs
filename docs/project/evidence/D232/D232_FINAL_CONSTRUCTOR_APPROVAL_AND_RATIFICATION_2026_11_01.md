# D232 — Final constructor approval, GITHUB021 reconciliation and ratification evidence

**Date:** 2026-11-01. **Formal owner:** [D232 #1014](https://github.com/guillermomolina/protos/issues/1014) in `guillermomolina/protos`. **Implementation successor:** [I136 #1015](https://github.com/guillermomolina/protos/issues/1015), one atomic slice; `guillermomolina/protos` only.

## Exact conversation approval sequence

1. The owner first corrected the meaning of inversion: direct `InvertedRange(3,0)` must mean `3,2,1`, and reversing an existing range while preserving its original member set is a DIFFERENT operation than the practical swap of directional bounds.
2. The owner expressly said **“ok apruebo esto último”** after the concrete dual-operator proposal:
   - `Range(0,3).invert() -> InvertedRange(3,0)`, visits `3,2,1`, swaps limits and direction, changes member set;
   - `Range(0,3).reverse() -> InvertedRange(2,-1)`, visits `2,1,0`, preserves original set;
   - both transforms are involutions of oriented limits; signed `eachStep` is direction-restricted; two ordinary first-class prototypes; migrate off old module helpers.
3. The owner asked to update issues, publish project-docs evidence and prepare one self-contained I implementation slice, expressly authorizing project-docs commit and push despite the normally human-executed repository-publication restriction.
4. The assistant presented exactly THREE final constructor constraints, and the owner explicitly answered **“Si lo apruebo”**:
   - `Ranges: import("std:collections/Range")`, `Range: Ranges.Range`, `InvertedRange: Ranges.InvertedRange`, then ordinary `Range(0,10)` and `InvertedRange(3,0)`.
   - Strict `Range(a,b)` only for `a<=b`, strict `InvertedRange(a,b)` only for `a>=b`; invalid order -> `Error`, equality -> empty.
   - Exactly two genuine semantic Integer inputs, ordinary evaluated-once left-to-right effects; each instance frozen after construction, with its `start` and `stop` locally visible/reflected.
5. No additional user assent is assumed for extra operators, generic Iterator, syntax or host-level fast paths. Implementation must enforce full exact arity independently of the flexible ordinary Closure binder and use standard Object.call/init semantics.

## Ratified construction examples (exact owner-approved public surface)

```protos
Ranges: import("std:collections/Range")
Range: Ranges.Range
InvertedRange: Ranges.InvertedRange

r: Range(0, 3)
ir: InvertedRange(3, 0)

r.parent() === Range                // true
ir.parent() === InvertedRange       // true
r.start                             // 0
r.stop                              // 3
ir.start                            // 3
ir.stop                             // 0

r.each(i => print(i))               // 0,1,2
ir.each(i => print(i))              // 3,2,1
r.invert().each(i => print(i))      // 3,2,1
r.reverse().each(i => print(i))     // 2,1,0
```

The snippets are **future-contract illustrations**, not commands already run or functions already implemented. Reflection must observe `start`/`stop` as the constructed instance's own slots; frozen structural state forbids later slot reassignment/creation/deletion. Exactly two input arguments are accepted. Calls to prototype extracted from module use ordinary Protos `Object.call` / `init`; the module is not given special constructor syntax.

## Owner-invariant consistency and deliberate previous-decision override

| Governing invariant | Final outcome |
| --- | --- |
| LIB017 two module-local helpers published, first-class Range deferred | **EXPRESSLY REOPENED**: remove both old helpers, replace by two oriented first-class prototypes, no alias |
| LIB017 half-open directed counting | **PRESERVED** under first-inclusive/last-exclusive in each direction |
| LIB017 `start>=stop` old helper emptiness | **REOPENED** for new constructors: wrong-order -> `Error`; equality -> empty |
| LIB017 exact unbounded Integer arithmetic, callback contract, no eager allocation | **PRESERVED** |
| Owner: `invert` vs `reverse` differences | **PRESERVED** exact swap vs set-preserving ±1 mathematical transform |
| Owner: `eachStep` uses positive on ascender, negative on descender | **PRESERVED**, 0 and wrong sign -> `Error` even on empty |
| Owner: ordinary first-class prototype-backed objects, no special grammar | **PRESERVED** by explicit module exports and ordinary `Object.call/init` |
| Owner: exact constructor arity/endpoints/frozen reflected state | **APPROVED in final explicit “Si lo apruebo”** |
| GITHUB010/021 no invented unresolved semantic changes | **PRESERVED**; implementation may choose representation/validation algorithm only when not observable beyond contract |

**DECISION_APPROVAL_PROVENANCE=PASS** for the exact two owner approvals. **DECISION_INVARIANT_CONSISTENCY=PASS** after explicitly reopening the LIB017 selector and wrong-order compatibility changes. **FINAL_PUBLIC_CONTRACT=SELECTED**. **REQUIRED_DURABLE_PUBLICATION=PASS** for the ratified [D232 language decision at immutable commit 064485be](https://github.com/guillermomolina/protos-project-docs/blob/064485be40fde8eef0db93f6ce6b8116b360b9a7/docs/project/decisions/language/D232_DIRECTIONAL_RANGE_DUAL_INVERSION_SELECTED.md).

## Execution boundary and immutable handoff

The one-slice implementation handoff is in:
- [Agent prompt, ratified constructor contract](https://github.com/guillermomolina/protos-project-docs/blob/b5fa6e2c7d732942df3c8450911682b00832c8f5/docs/project/work/I136/I136_AGENT_ONE_SLICE_PROMPT.md)
- [Human executor prompt, ratified constructor contract](https://github.com/guillermomolina/protos-project-docs/blob/625f9d51efddd20590bcdee1e564ec66851c84d9/docs/project/work/I136/I136_HUMAN_EXECUTOR_ONE_SLICE.md).

The agent operates only in the already-HEAD local `guillermomolina/protos` checkout, with no web or other repository access, and **does not execute** builds/tests/validation, state-changing Git or publication. The human executor runs the exact current Makefile tests/checks, reviews diffs and performs one final product commit/push. Neither approval nor this documentation is a claim of product code changes, conformance PASS, Native PASS, speedup, version bump or published implementation.

**Product inspection SHA:** `fba47c40c4b79b6e580df1c1f78a3fcecbd067e2` (I135 baseline); the future agent must re-read its provided *local* HEAD, never checkout this historical SHA.
