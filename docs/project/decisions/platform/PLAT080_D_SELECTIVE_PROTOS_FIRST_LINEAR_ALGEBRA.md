# PLAT080-D — Selective Protos-first linear algebra direction

**Status:** OWNER APPROVED — final retention disposition (see below).
**Date:** 2026-11-27.  
**Issue:** [PLAT080 #1212](https://github.com/guillermomolina/protos/issues/1212), child of [AUD025 #1209](https://github.com/guillermomolina/protos/issues/1209).  
**Evidence:** [Research and owner approval](../../evidence/PLAT080/PLAT080_D_RESEARCH_AND_OWNER_APPROVAL_2026_11_27.md), published at [14e8cf274b7d64df05e0c1aff203fa77eb0d18f9](https://github.com/guillermomolina/protos-project-docs/commit/14e8cf274b7d64df05e0c1aff203fa77eb0d18f9).  
**Inspected Protos revision:** [509221a9d28b79e9da9accf721fad2358d81ec49](https://github.com/guillermomolina/protos/tree/509221a9d28b79e9da9accf721fad2358d81ec49).

## Final owner-approved disposition (2026-11-27)

The prior conditional migration direction below is historical. The owner clarified that Vector and Matrix must retain their current native representation, with no Array intermediates and no replacement for replacement's sake, and explicitly approved the final retention proposal ("si apruebo."). **Retain native fundamental linear-algebra operations; prefer Protos for naturally composable higher-level algorithms.** No new I, Java bridge, wrapper, storage change, normative change, or hardware work. D214, D252, I124, I172, PLAT068-R3, D242 and I115 invariants remain unchanged. The feasibility gate is resolved by justified retention, not by claiming migration impossible.

[Final investigation and owner approval](../../evidence/PLAT080/PLAT080_NATIVE_LINEAR_ALGEBRA_RETENTION_2026_11_27.md) records the source-backed reasons and GITHUB010/GITHUB021 review at [5f2b40910f09f163e2c2987a4731c0d0d499b373](https://github.com/guillermomolina/protos/tree/5f2b40910f09f163e2c2987a4731c0d0d499b373). AUD025 phase B disposition is complete; live issue lifecycle remains in GitHub. No product changes or tests.

## Historical conditional checkpoint

## Selected direction

Attempt selective migration to ordinary Protos of Vector add, scale and dot, and Matrix add, scale, transpose, matvec and generic matmul, only where established canonical access, arithmetic, construction and specialized execution mechanisms preserve the existing contract. Retain the current implementation for an operation if migration would require unjustified Java infrastructure, dynamic messages in the hot path, avoidable representation regressions or semantic changes. The owner explicitly approved alternative D in the active interaction: “ok apruebo la recomendacion.”

Retain private Vector/Matrix runtime representation and admission, indexed access and mutation, construction, physical copy initially, graph-aware Actor/P transfer, structured snapshot each, canonical scalar primitives and the existing packed Float matmul path. This is a boundary policy, not approval of any new Java mechanism or public interface.

## Invariants

D214-C1/S1/F1, I124, I172 and PLAT068-R3 remain unchanged: genuine Number admission, exact dimensions and zero shapes, canonical arithmetic without user callbacks, Hermitian dot, bilinear matrix operations, ascending contraction order and first-product seed, exact Integer zero for empty reductions, coupled Complex rounding, ordinary public dispatch, private packed storage, fresh independent results, state, transfer and isolation. No new dtype, syntax, storage representation or hardware acceleration. Preserve existing Truffle specialization, suspension and Native Image compatibility; Java production changes require exact verified method/annotation/behavior precedent in another Truffle language, preferably GraalPy. Protos primitives remain long/double/boolean.

## Remaining gate

No exact reusable guest-to-canonical-payload execution mechanism has yet been demonstrated across all affected operations. The preceding research explicitly did not complete whole-document normative secondary reads. The owner selection does not discharge those conditions, authorize a speculative implementation issue, or ratify a specific runtime bridge. Keep PLAT080 open until a viable bounded implementation candidate or concrete retention disposition is established. If implementation is warranted, allocate one I issue under the coordination rules; do not allocate one per operation. AUD025 phase B remains incomplete.

No product edits, version/spec changes, tests or benchmarks were made. No extra success metrics or deletion targets. This record is non-normative.
