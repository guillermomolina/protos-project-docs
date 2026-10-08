# PERF032 — Performance workstream reconciliation (2026-10-08)

**Kind:** durable coordination and evidence-routing checkpoint; not a benchmark, a new implementation result, or a normative language decision.

**Authority:** explicit project-owner acceptance on 2026-10-08 of the proposal to retain PERF032 as the only actively pursued PERF workstream, preserve PERF031 as independent backlog, and retire PERF024/PERF030 without declaring their original acceptance criteria completed.

**Repositories:** [`guillermomolina/protos`](https://github.com/guillermomolina/protos) owns live Issues; [`guillermomolina/protos-project-docs`](https://github.com/guillermomolina/protos-project-docs) retains this non-normative historical checkpoint.

## Revision and evidence anchors

- Product repository HEAD inspected during routing: [`2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3`](https://github.com/guillermomolina/protos/commit/2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3). This is an identity checkpoint, **not** a new performance measurement.
- TEST009 complete at product `65a23bc25bd30dfd65843468dba3812a78121c06`, per [PERF030's TEST009 closure handoff](https://github.com/guillermomolina/protos/issues/784#issuecomment-6035023452). Its closure does **not** discharge all of PERF030's own historical acceptance criteria.
- PERF032-F's retained [cross-Truffle graph parity matrix](PERF032_F_CROSS_TRUFFLE_GRAPH_PARITY_MATRIX.md) uses `protos@c0ac98971df115d64b7bc9f146e8b02e11da30e6` and `protos-benchmarks@e221bb056a208693c9102891df2e13d65e61eefb`: After-TruffleTier nodes Protos 282, GraalJS 13, GraalPy 49 in the primitive return-literal case. These numbers are historical baseline evidence, not a fresh current-HEAD conclusion.
- [PERF032-G2 symmetric audit](PERF032_G2_SYMMETRIC_AUDIT_PARTIAL.md) classified `HARNESS_VERDICT=PARTIALLY_VALID` and `COMPARISON_EQUIVALENCE=UNRESOLVED`; the differences have **not** been causally attributed to proven removable Protos machinery.
- [I086-3C standard embedding publication](https://github.com/guillermomolina/protos/issues/831#issuecomment-6053343257) reached product `2bc07f1b99d6fbb9bf8d53c46c64112fd0ffd7e3`. It removes the optional Core-root configuration prerequisite for standard embedding, but establishes neither comparable cross-language graph roots nor speedup.

## Project-owner-approved routing

| Work item | Decision | Meaning and residuals |
| --- | --- | --- |
| [PERF024 / #756](https://github.com/guillermomolina/protos/issues/756) | Close **not planned / superseded scope** | Retire the broad primitive/structured cross-Truffle decomposition campaign. Preserve its completed harness, recorded evidence and history. Its originally unchecked acceptance criteria are **not** marked completed. Its return-literal question is owned independently by PERF032, not implicitly fulfilled by this closure. |
| [PERF030 / #784](https://github.com/guillermomolina/protos/issues/784) | Close **not planned / discontinued** | Retire the old `primitive-closure-call` PE-bailout campaign. Published local repairs and TEST009 completion survive. PERF030-specific final IGV/JFR and original acceptance criteria are **not** declared passing. Preserve the native sub-issue relationship to PERF024 and TEST009's native child relationship to PERF030. Do not restart warning-count cleanup from this issue. |
| [PERF031 / #787](https://github.com/guillermomolina/protos/issues/787) | Keep **open / ready** | Distinct Java integration-test workload proportionality/duplication investigation, triggered by TEST008. No current execution is claimed; keep unscheduled backlog, with no dependency invented toward PERF032. |
| [PERF032 / #831](https://github.com/guillermomolina/protos/issues/831) | Keep **open / in progress** | Sole actively pursued PERF research/optimization stream: `primitive-return-literal`, profiling-first, same-workload causal attribution. No global zero-warning requirement and no arbitrary graph-node target. Remaining platform embedding conformance is owned separately by PLAT054/I086. |

## Dependency and boundary resolution

1. Historical lineage is PERF023 (closed) -> PERF024 (retired), with PERF030 its native child and TEST009 (completed) a native child of PERF030. Closure does not erase those historical relationships.
2. PERF032 arose from PERF024's primitive decomposition but owns only the return-literal question, not the unfinished multi-workload campaign. PERF033 was independently completed for canonical callable interop; its closure must not be mistaken for current graph/timing parity.
3. PERF031 addresses Java test-suite costs. Its remaining work neither blocks PERF032 nor licenses any change to ordinary testing budgets or semantic coverage.
4. The PLAT054/I086 standard `Context.eval` + `getBindings` + `Value.execute` path removes an embedding preparation limitation. New parity or measured speedup requires later equivalent source/root/inlining/return and same-runtime evidence under PERF032; do not infer it from correct API implementation.
5. No speculative optimization, return-literal graph-node deletion, global Truffle warning cleanup, workload redesign, or repository code change is authorized by this reconciliation.

## Closure classification and validation

- PERF024: `state=closed`, `state_reason=not_planned`, explanation **superseded scope**, not acceptance complete.
- PERF030: `state=closed`, `state_reason=not_planned`, explanation **discontinued historical work**, not acceptance complete.
- PERF031: open/ready; PERF032: open/in-progress, as approved routing.
- This documentation-only coordination task performs no build, functional test, performance run, source edit, specification edit, version increment, or changelog change. Prior human-reported tests remain prior evidence only, not new validation.
- GitHub Issue comments and transitions must point to this exact published record revision and be re-read afterward; publication of this record alone must not be reported as Issue closure.

## Reopening discipline

An independently demonstrated new performance problem may justify a new bounded work item or an explicit reopening decision. Neither an open diagnostic trace nor historical residual acceptance criteria automatically reactivates PERF024/PERF030. Maintain all earlier evidence without rewriting historical benchmark results.
