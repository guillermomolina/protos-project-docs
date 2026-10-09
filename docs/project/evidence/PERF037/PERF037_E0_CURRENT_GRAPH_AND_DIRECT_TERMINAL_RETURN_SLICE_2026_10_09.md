# PERF037-E0 — Current object-slot-read graph and direct terminal return slice (2026-10-09)

**Live owner:** [PERF037 / guillermomolina/protos#851](https://github.com/guillermomolina/protos/issues/851). **Workload:** `primitive-object-slot-read`. **Status:** investigation complete for a bounded next implementation proposal; **PERF037-E NOT YET IMPLEMENTED OR VALIDATED**.

## Provenance and human-reported gate

- **Product HEAD examined:** [`guillermomolina/protos@89e1b038c2fd510f47ddb991496882548d1bbe44`](https://github.com/guillermomolina/protos/commit/89e1b038c2fd510f47ddb991496882548d1bbe44). It already includes PERF037-D and concurrent PERF038-F. The owner explicitly prefers working against the **latest HEAD**, not replaying an old commit for historical attribution.
- **Full stable compiler-IR evidence:** [`guillermomolina/protos-benchmarks@dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs`](https://github.com/guillermomolina/protos-benchmarks/tree/dd8b6518057de62d940a10e5b3db1de7ea97929b/results/perf037-d-89e1b038-graphs); source producer harness `a599949830cc7a240ffd347de615006c112c94d6`. `unit.json`: `After TruffleTier`, `STABLE`, `evidence_valid=true`, **64 nodes**.
- **GraalJS comparison:** [stable `primitive-object-slot-read/js/unit.json`](https://github.com/guillermomolina/protos-benchmarks/blob/623e494493f0d3f91136d10048f365c24407c78c/results/global-20261008-graphs/primitive-object-slot-read/js/unit.json): **36 nodes**. The JS peer uses `executable-value` surface, Protos uses `canonical` and the peer was recorded under a different harness revision. Structural comparison is informative, not a guaranteed runtime speedup or direct equal-surface timing equivalence.
- **Human-submitted graph extract:** the owner supplied `perf037-nodes-protos64-vs-js36.json` containing both selected `StructuredGraph` `After TruffleTier` node/property/connection dictionaries. This submitted conversation attachment is not misrepresented as a committed artifact. The raw graph/filtered exports are in the public benchmark commit above.
- **Existing product validation:** the owner explicitly reported `git diff --check` clean and **all local tests PASS**. This statement applies to **already-published/current product source**, **not** the next PERF037-E source changes, which have not yet been made. Never claim new tests ran.

## Exact residual graph evidence

The current Protos 64-node graph contains:

| IR class and instances | Count | Interpretation |
| --- | ---: | --- |
| `VirtualInstanceNode #85` | 1 | Virtual `FrameWithoutBoxing` of the invocation root |
| `VirtualArrayNode #89,#90,#91` | 3 | `Object[6]`, `long[6]`, `byte[6]` backing that virtual frame |
| `VirtualArrayNode #967` | 1 | `Object[2]` runtime-entry arguments (not a guest `arguments` Array) |
| `VirtualObjectState #968–#972` | 5 | Deoptimization-state descriptions of those virtual objects |
| `TrufflePreserveFrameStateNode #139` | 1 | Emitted in generated `CachedBytecodeNode.continueAt` |
| `FrameState` | 6 | `#3,#123,#124,#140,#141,#142`; `#140` is root generated-interpreter frame-preserve site |
| `NarrowNode #542` | 1 | Physical byte-tag read through `FrameWithoutBoxing.unsafeGetIndexedTag` in `LoadLocalMaterialized`; associated `RawLoad #540` and `FixedGuard #545` |
| `ConstantNode` | 20 | Constants associated with runtime root, access and deoptimization structures |

GraalJS's corresponding stable graph has **no virtual objects or virtual object states, one `FrameState`, no `TrufflePreserveFrameStateNode`, no `NarrowNode`**. It **does** have tag/representation checks for captured locals in its own node implementation; the absence of `NarrowNode` is not proof that GraalJS ignores local-kind correctness.

The observed **frame object bundle** is the virtual current execution frame and its deoptimization encoding, not five proven heap allocations. The existence of the virtual frame is *not* on its own evidence that removing a sequence-local will eliminate all virtual nodes.

## Specific Protos-owned lowering opportunity (PERF037-E)

Source location: `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`, `emitRootBody` near lines 810–845 in product `89e1b038`.

Current behavior for non-empty sequence:

```java
BytecodeLocal result = builder.createLocal("sequenceResult", null);
...
if (last instanceof CanonicalLookup lookup && isScalarLocalRead(lookup)) {
    emitStatementsToLocal(builder, prefix, result);
    beginStatement(builder, lookup);
    emitScalarLocalRead(builder, lookup, null); // already directly returns
    endStatement(builder);
} else {
    emitStatementsToLocal(builder, sequence, result);
    builder.beginReturn();
    builder.emitLoadLocal(result);
    builder.endReturn();
}
```

The timed workload source is `holder: { value: 1 }; run: () => { holder.value }`. Its last body expression is a `CanonicalMember` whose receiver is a non-composed lookup; `requiresComposedInvocation(CanonicalMember)` delegates to its receiver and does not demand invocation staging in this case. Nevertheless the final value is routed through `sequenceResult` rather than returned directly.

**Recommended single implementation slice:** `PERF037-E — Direct return for eligible value-shaped final Sequence expressions`. Within `CanonicalToBytecodeLowerer.emitRootBody`, emit the **last expression directly into a Bytecode DSL `Return`** for value-shaped, non-composed final expressions. Do not allocate `sequenceResult` for a single-expression admitted root. Preserve earlier-expression evaluation and all source sections, statement/expression tags, diagnostic stack/scope behavior and cleanup/return semantics. Reuse the existing `emitExpression` implementation, existing `beginStatement`/`endStatement` and existing scalar-direct-return precedent. Keep original `emitStatementsToLocal` + staging for composed invocations/sends, return/control transfer, assignment/create, object construction, suspensions/defaults where the generic composed lowering is necessary. This is a general source-shape rule; **do not special-case benchmark name, `holder`, `value`, receiver identity or concrete literal**.

This is a **source-proven redundant temporary candidate**, not an isolated measured `sequenceResult` graph-node count. No reduction number, timing improvement, or virtual-frame elimination is asserted before measurement.

## Coordination boundaries / deferred lines

- **PERF037 owns:** this source-level last-expression return, `primitive-object-slot-read` captured-member lowering, `NarrowNode`/physical frame-tag follow-up, and the identified graph residues.
- **PERF038 owns:** alternative root/interpreter architecture, zero/one/many argument entry, call-site dispatch, activation/return-home lifecycle, and invocation ABI. No `@GenerateBytecode` flag changes or alternative root under PERF037.
- **Deferred:** `SlotCell` non-Closure continuity assumption remains a recorded candidate, **not** part of PERF037-E.
- **Next human validation:** agent edits product source/tests, human runs targeted correctness/static checks, then impact-aware required integrated validation. Version and `CHANGELOG.md` are finalized **only after tests green**, immediately before human commit/push, with no automatic rerun of tests solely for these metadata edits.
- **Next performance measurement:** only after a clean product publication, capture this exact workload once, Protos-only, under the pinned harness; compare against the 64-node revision-bound baseline and reuse the 36-node GraalJS reference. `NOT_STABLE` and invalid comparisons block any positive performance conclusion.

## Normative guardrails and record authority

`spec/semantics/EXECUTION_AND_CONTROL.md` §8.2 explicitly defines Sequence result as the exact final-expression value; an empty sequence returns canonical null, and control-transfer does not turn into null. `spec/semantics/CALLABLES.md` defines braced Closure bodies using this Sequence normal result. `spec/semantics/OBJECT_MODEL.md` still owns member lookup identity/failure; no semantic changes are authorized.

The next implementation agent must inspect and obey `AGENTS.md`, `AGENTS.work/PERFORMANCE.md`, `AGENTS.work/IMPLEMENTATION.md`, `src/AGENTS.md`, current source/tests and the relevant normative sections. The maintainer runs all builds/tests/git operations for the product; docs may be published directly in `guillermomolina/protos-project-docs` under the explicit owner authorization.
