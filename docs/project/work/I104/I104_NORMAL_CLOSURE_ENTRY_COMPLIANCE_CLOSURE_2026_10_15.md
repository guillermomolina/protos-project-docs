# I104 — NORMAL-first Closure entry: compliance closure evidence (2026-10-15)

**Formal owner:** [I104/#899](https://github.com/guillermomolina/protos/issues/899), implementation of owner-ratified [D202/#898](https://github.com/guillermomolina/protos/issues/898).
**Product revision (published):** [`2263d4f66294817524052b9782ffee1562848a50`](https://github.com/guillermomolina/protos/commit/2263d4f66294817524052b9782ffee1562848a50).
**Implementation version:** `0.3.365-SNAPSHOT`.
**Approval/validation date:** 2026-10-15.
**Record role:** durable, non-normative implementation and human-reported conformance evidence. The live GitHub Issue owns completion status; this document does not amend the normative `spec/` contracts.

## Owner's final scope clarification

After publishing the implementation, the owner explicitly clarified: **“no hace falta medir, esta issue no era por performance, era por compliance”** (2026-10-15). Thus **I104 acceptance is contract/compliance, not a performance claim**. The previously outlined I104-D matched Tier-2/GraalPy benchmarking step is not a blocking acceptance requirement for I104 closure. No speedup, Tier-2 steady-state result, or cross-language performance parity is asserted. This is a subsequent owner-directed acceptance-scope clarification, not a rewrite of the historical D202 investigation or a semantic/ABI design change.

The implementation must still preserve D003/CALLABLES, D179, PLAT058-D immutable NORMAL/DEBUG emission, PLAT060-B5 five-slot Closure-call ABI, Error/control authority, context presence and captures, caller/receiver/home provenance, and original default/rest observable order.

## Published implementation

The exact product commit contains 13 paths: two production files, six test files (including the new `ProtosPerf042IdentityBuilderOracleTest`), four local-range PE analyzer/baseline/test files, and the root `pom.xml` and `CHANGELOG.md`.

- **A — unary parameter establishment.** `BindUnaryFrameParameter` admits the proved compact NORMAL unary binding path while keeping richer fallback and DEBUG behavior. Independent handwritten Builder-oracle and materialization/indexed-binding tests were adapted to actual emitted structure.
- **B — supplied required-prefix transfer.** The attempted `CopyRequiredParameterPrefix` bulk operation with a runtime loop index failed the strict `LocalRangeAccessor` PE guard. The final implementation replaced it with one emitted `BindRequiredFrameParameter` operation per required parameter. Each `ordinal` is an `@ConstantOperand` of the Bytecode DSL operation; the source lowerer loops at emission time, not as a guest execution loop.
- **C — integration.** Required-prefix binding precedes the remaining normative parameter-binding sequence; defaults and rest retain their invocation-time semantics and the existing authoritative paths. DEBUG negative controls, required-prefix structural tests, and the manually authored Builder oracle are included in the final test set.
- **Guard soundness fix.** `tools/java_local_range_pe_guard.py` now recognizes the generated edges from `@Specialization(guards = "...")` to exact local helper calls and follows constant-operand provenance. A positive and negative synthetic test rejects the false inference of a constant ordinal when `@ConstantOperand` is removed. Two local-index risks were explicitly baselined as `METHOD_PARAMETER` syntax only after cross-checking the reachability proof; four reachability sinks were added, and one older `BindClosureFrameParameter` drift was reconciled. No reachable dynamic PE index risk was admitted.

## Human-executed validation

The owner/human executor reported, before final version materialization:

- Focused Java validation: **48 tests run, 0 failures, 0 errors, 0 skipped**.
- `make check-local-range-index-pe`: **PASS**; `TOTAL_LOCAL_RANGE_SINKS=33`, `NEW_UNBASELINED_RISKS=0`, `PE_REACHABLE_RISK=0`, `PE_REACHABILITY_UNKNOWN=0`, `CALL_GRAPH_AMBIGUITIES=0`, `UNPARSED_FILES=0`, `NEW_UNBASELINED_REACHABILITY=0`, `REACHABILITY_DRIFT=0`, and `STALE_REACHABILITY_ENTRIES=0`. The 13 pre-existing/explicitly reviewed baseline entries are **not** claimed to be syntactically proven safe; no such entry was classified as an actual reachable PE risk.
- Full integrated `make test`: **PASS**, as expressly reported by the human executor.
- Finalization `git diff --check`: clean, as reported by the human executor. The version and changelog were promoted **only after** green tests, immediately before human commit/push; no post-version-bump test execution is claimed.
- Human publication: `main` advanced `d12d890d..2263d4f6`, final product SHA verified by GitHub as above. No agent-run compilation, test, or product repository write is claimed.

## Scope and closure decision

**COMPLIANCE_ACCEPTED** for the published I104 A/B/C implementation under the owner's explicit I104 acceptance-scope clarification. The D202 ratified semantic invariants remain in force. The previously proposed Tier-2/performance benchmark is **NOT_REQUIRED_FOR_I104_CLOSURE** and **NOT_MEASURED**; further performance work, if ever requested, is independently scoped and must not be represented as having passed.

```text
FORMAL_ITEM=I104/#899
PROTOS_REVISION=2263d4f66294817524052b9782ffee1562848a50
PROTOS_IMPLEMENTATION_VERSION=0.3.365-SNAPSHOT
FOCUSED_TESTS=48/0/0/0
MAKE_TEST=PASS_HUMAN_REPORTED
LOCAL_RANGE_PE_GUARD=PASS_HUMAN_REPORTED
PE_REACHABLE_RISK=0
PE_REACHABILITY_UNKNOWN=0
DIFF_CHECK=PASS_HUMAN_REPORTED
PERFORMANCE_TIER2=NOT_MEASURED_NOT_REQUIRED
NORMATIVE_SPEC_DELTA=NONE
PLAT060_B5_ABI_DELTA=NONE
I104_ACCEPTANCE_SCOPE=COMPLIANCE
```
