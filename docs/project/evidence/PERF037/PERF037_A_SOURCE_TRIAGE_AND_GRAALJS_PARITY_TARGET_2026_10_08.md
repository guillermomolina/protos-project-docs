# PERF037-A — Source triage and GraalJS parity target (2026-10-08)

**Role:** revision-coupled, non-normative investigation evidence; no implementation approval, test result, or benchmark result is asserted here.

- **Owning issue:** [PERF037 / protos#851](https://github.com/guillermomolina/protos/issues/851).
- **Product HEAD inspected (read-only):** [`guillermomolina/protos@b750036119ca3af14732c658819b62b49039b350`](https://github.com/guillermomolina/protos/commit/b750036119ca3af14732c658819b62b49039b350).
- **Benchmark repository:** [`guillermomolina/protos-benchmarks`](https://github.com/guillermomolina/protos-benchmarks); **benchmark HEAD, exact workload source, raw BGVs, and comparable run identities have not been established in this checkpoint**.
- **Named rung in the existing benchmark policy:** `truffle/measure/graphs.json` -> `primitive-object-slot-read`, described as “monomorphic read of an existing object slot/property/attribute.”
- **Execution:** no program, test, build, profiler, graph capture, command, or product Git state-changing operation was executed for this record.

## Owner-directed objective

For exactly the same prepared `primitive-object-slot-read` case, seek the minimum Protos compiled graph relative to GraalJS. Every **Protos-owned** surviving operation that is not present in the corresponding GraalJS execution path carries a burden of proof: either it is mandatory for an observable Protos semantic in this concrete case, or it is optimization debt. Reject generalized machinery paid by a simple data-member read when its corresponding feature is not used (**pay as you grow**). A different semantic requirement may justify an operation, but must be proved specifically rather than claimed abstractly.

Graph-size parity is an investigative target, not an unconditional authorization to change normative behavior; retained calls, guards and allocations must also be judged by measured runtime cost. Preserve the exact selected operand/result, equivalence of prepared execution surfaces, closure extraction semantics, structural invalidation, and error behavior.

## Direct source findings at exact product revision

1. [`ProtosBytecodeRootNode.ReadMember`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java) and [`ProtosSemanticBytecodeRootNode.ReadMember`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java) have shared-inherited and exact-receiver PIC paths, with selector identity/name guards and `Assumption`-protected slot-owner selection; fallback does authoritative generic lookup. The exact-receiver specialization knows the chosen **owner**, not a stable physical **location** for the current value.
2. [`ProtosValueLookup.GuardedSlotSelection`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java) caches `home` and stability but explicitly does not cache the value. Its `materializeGuardedMemberRead` performs `home.readLocalSlot(name).orElseThrow()` on each valid hit, then creates a newly receiver/home-bound Closure when the slot value is a Closure; plain data values are returned directly.
3. [`ProtosObjectValue.readLocalSlot`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java) routes nonempty authorities to [`ProtosLexicalBindingAuthorityCalls.read`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalBindingAuthorityCalls.java), an explicit `@TruffleBoundary`. [`ProtosMapBackedLexicalBindingAuthority.readBinding`](https://github.com/guillermomolina/protos/blob/b750036119ca3af14732c658819b62b49039b350/src/main/java/com/guillermomolina/protos/runtime/ProtosMapBackedLexicalBindingAuthority.java) probes a `LinkedHashMap<String,Object>` and returns `Optional<Object>`. This is a source-proven String-keyed value read after owner selection; whether it is in the **selected compiled unit** as a surviving call or an expanded graph fragment is not yet established.
4. The existing slot-selection assumptions are distinct from value-sensitive dependencies: an ordinary assignment changes the value without necessarily changing selected home, while structural shadowing, slot creation/removal, and inherited lookup can change selection. Any direct-location cache must preserve precisely these invalidation semantics, including value changes after warming.
5. The inherited/shared-parent specialization is a different case from an exact own-data-slot read; never remove the local-shadowing proof for sibling receivers without an equivalent correctness invariant. A data-member read must not pay for bound-Closure extraction when no Closure is present, but Closure reads still have to return a fresh appropriate bound value.
6. [PERF035's historical Tier-2 slot-write evidence](../PERF035/PERF035_PUBLISHED_SLOT_WRITE_TIER2_STABILIZATION_2026_10_08.md) covers `primitive-object-slot-write` at product `203c0f9...`; its `1122` graph nodes are **not** a read-rung baseline, JS comparison, or present-product result.

## Hypothesis requiring BGV and timing attribution

**H1 (unproven):** for a warmed monomorphic existing *data* slot, the remaining `readLocalSlot(name)` → bounded lexical authority → String-keyed map query is an avoidable per-hit operation after the PIC already knows `home`. A stable, structurally invalidated *slot location* with a fresh value load may remove this operation without re-reading by key on each hit.

This is not an instruction to change `LinkedHashMap`, remove all `@TruffleBoundary` seams, cache the old slot **value**, pre-allocate shape metadata for every object, or add benchmark-specific shortcuts. A direct-location design is acceptable only after demonstrating both that its required state is lazy and that all source-defined mutation/alias/composition/inheritance semantics remain exact. The possible gain is currently **qualitative**, not a verified graph-node delta or speedup.

## Read-only next slice: PERF037-A

1. Verify the exact authoritative `primitive-object-slot-read` source and result in the benchmark repository, plus counterpart JS/Py source and prepared `Value.execute()` surface. Do not run the workloads.
2. Locate retained valid `After TruffleTier` graph evidence and matching runtime measurements, if already available. Record exact product/harness revisions, graph identity, selected units, phase, compiler tier, correctness, warmup/admission and peer comparability. If absent or stale, mark **PENDING** and supply the minimum human-executor capture/analyze/verify commands **as a future execution gate**, not as executed steps.
3. For every Protos-only surviving node or edge compared with GraalJS, classify: required in this precise semantic case; residual of an unused general feature; redundant guard/lookup/carrier; framework/unattributed. Quantify loads, `Pi`, branches, frame states, virtual arrays, calls and `@TruffleBoundary` effects by selected BGV when available. No invented graph counts.
4. Trace the first source-owned divergence through `ReadMember`, `ProtosValueLookup`, `ProtosObjectValue`, authority storage, lowerer and root/frame machinery before deciding whether H1 is the first fix. Check that the gap is not attributable to the host `Value.execute()` scaffold or a non-equivalent peer workload.
5. Define one **grouped**, semantics-preserving implementation candidate, with impacted paths, exact guards/invalidation rules, tests to add or reuse, predicted removed hot operations, residual risks, and A/B plan. STOP before any implementation or execution; owner/human authorizes the implementation slice separately.

## Boundaries and status

- **SOURCE_INSPECTION:** PASS for the six named product files at product `b750036119ca3af14732c658819b62b49039b350`.
- **NORMATIVE AUTHORITY:** `spec/semantics/OBJECT_MODEL.md`, `EXECUTION_AND_CONTROL.md`, and `CALLABLES.md`; their full case-level implications are a follow-up investigation task, not a proved conformance verdict here.
- **IDENTICAL_WORKLOAD_CONFIRMED:** PENDING.
- **GRAALJS_GRAPH_BASELINE:** PENDING.
- **PROTOS_READ_GRAPH_BASELINE:** PENDING.
- **STABLE_TIMING_AND_AB:** PENDING.
- **DESIGN_RATIFICATION:** NOT REQUESTED.
- **PRODUCT_MODIFICATIONS / TESTS / BUILDS:** NONE.
- **ISSUE CLOSURE:** NOT CLAIMED.

The canonical live work status and next slice remain in [PERF037/#851](https://github.com/guillermomolina/protos/issues/851). This project record preserves **source-derived evidence only**, not raw measurements or a completed performance comparison.
