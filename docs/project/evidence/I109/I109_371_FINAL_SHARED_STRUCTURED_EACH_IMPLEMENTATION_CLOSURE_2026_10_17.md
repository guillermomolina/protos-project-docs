# I109 — shared structured standard `each`: final implementation acceptance and publication (2026-10-17)

**Formal work:** [I109/#910](https://github.com/guillermomolina/protos/issues/910) (single grouped implementation slice I109-A).
**Architectural authority:** [PLAT063-C/#908](https://github.com/guillermomolina/protos/issues/908), exact owner-approved candidate C; [ratified decision](../../decisions/platform/PLAT063_SHARED_STRUCTURED_STANDARD_EACH_LOWERING.md) and [four-family decision evidence](../PLAT063/PLAT063_C_FOUR_FAMILY_AUDIT_AND_APPROVAL_2026_10_17.md).
**Final product revision:** [`e07340c2ea6ac26278a04495e145a08e3bb2cd4d`](https://github.com/guillermomolina/protos/commit/e07340c2ea6ac26278a04495e145a08e3bb2cd4d) on `main` — implementation version **`0.3.371-SNAPSHOT`**.
**Functional product revision:** [`eed3098b59ee86af4639e2710ce38999a4707d39`](https://github.com/guillermomolina/protos/commit/eed3098b59ee86af4639e2710ce38999a4707d39), exactly one commit after prior [`c00b36c0edaf6678b7d8101de3a405124f179644`](https://github.com/guillermomolina/protos/commit/c00b36c0edaf6678b7d8101de3a405124f179644) / `0.3.370-SNAPSHOT`.
**Historical checkpoint:** [I109-A code publication and temporary metadata gap](I109_A_SHARED_STRUCTURED_EACH_IMPLEMENTATION_CHECKPOINT_2026_10_17.md); retain as historically accurate evidence rather than rewriting it.

## Verified implementation

Product source changes at `eed3098b` affect exactly **six files**:

1. [`ProtosBytecodeRootNode.java`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java): six shared C-prime adapter operations reuse the existing `PreparedLocalEachCall` state across Array, Bytes, Map, IdentityMap and Environment.
2. [`ProtosStructuredDispatchLowerer.java`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/ProtosStructuredDispatchLowerer.java): one actual emitted Bytecode DSL `While`/`TryFinally` replaces five duplicated structured-control loops. Scoped callback invocation, native suspension/continuation and collection-specific semantics remain bounded by the original prepared state and existing structure.
3. [`ProtosI109ASharedLocalEachLoweringTest.java`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/src/test/java/com/guillermomolina/protos/execution/ProtosI109ASharedLocalEachLoweringTest.java): structural source-level regression.
4. [`ProtosI109ABytecodeMaterializationTest.java`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/src/test/java/com/guillermomolina/protos/execution/ProtosI109ABytecodeMaterializationTest.java): materialized operation topology and exclusion of five former per-family emitted-loop structures.
5. [`ProtosI109ADynamicLocalEachTest.java`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/src/test/java/com/guillermomolina/protos/execution/ProtosI109ADynamicLocalEachTest.java): dynamic callback/snapshot/identity, suspension/reentry, Error/NLR, cancellation/ensure and NORMAL/DEBUG reparse identity regression cases for five collection families.
6. [`tools/java_generated_bytecode_bci_pe_baseline.json`](https://github.com/guillermomolina/protos/blob/eed3098b59ee86af4639e2710ce38999a4707d39/tools/java_generated_bytecode_bci_pe_baseline.json): reviewed exact generated BCI topology change: **12 new** `PROVEN_HANDLER_RETURN` sites across two cached dispatchers; **6 stale** Cached-only Map sites withdrawn while their TailCall counterparts remain; `RISK=0`, `UNKNOWN=0`. The guard was not loosened.

Canonical standard collection `each` retains dynamic selected-standard admission, collection-specific snapshots and argument ordering, original receiver identity, scoped callbacks with normal-advance-only, suspension/resumption, `Error`, non-local returns, `ensure`, cancellation, source/DEBUG instrumentation and Root/CallTarget reparse identity. Foreign `each`, already-specialized literal callback lowering, ordinary Closure invocation and normative semantics were not redesigned.

## Verified two-commit publication / resolved version correction

- **Code:** [`eed3098b59ee86af4639e2710ce38999a4707d39`](https://github.com/guillermomolina/protos/commit/eed3098b59ee86af4639e2710ce38999a4707d39) contains the six functional/test/guard files.
- **Metadata correction:** [`e07340c2ea6ac26278a04495e145a08e3bb2cd4d`](https://github.com/guillermomolina/protos/commit/e07340c2ea6ac26278a04495e145a08e3bb2cd4d) has parent `eed3098b` and modifies **only** `pom.xml` and `CHANGELOG.md`. The exact final `pom.xml` has `0.3.371-SNAPSHOT`; the first/root changelog entry is `## 0.3.371-SNAPSHOT` and attributes I109/#910. The version correction follows the code commit without rewriting published history. This resolves the temporary metadata omission preserved in the previous checkpoint.
- **Revision provenance:** GitHub commit and compare endpoints independently verified both changesets, their order, and final file contents. No additional code or test modifications occurred in the corrective commit.

## Maintainer-reported acceptance evidence

- `make test` → **PASS** on the functional candidate, prior to the metadata-only correction.
- `make check` → **PASS** after BCI topology reconciliation, with `BCI_TRANSITIONS=1108`, `PROVEN=1104`, `DYNAMIC=4`, `RISK=0`, `UNKNOWN=0` reported at the initial baseline-mismatch diagnostic.
- `git diff --check` → clean; targeted `git diff --cached --check` and final publication commands completed without reported failure.
- Focal test evolution: the six JUnit methods in `ProtosI109ADynamicLocalEachTest` ultimately passed; five were green in the six-test run, and the remaining reparse case passed after tightening the test's approved unreachable local-cleanup epilogue rule. Structural and materialization focal tests were previously reported green.
- **Structural comparison:** in `ProtosStructuredDispatchLowerer.emitPreparedInvocation`, emitted `While` 15→11 (−4), `TryFinally` 29→25 (−4), `IfThenElse` 22→18 (−4), `createLocal` 42→34 (−8), five canonical collection branches → one. The maintainer reported `I109_SHARED_STRUCTURED_EACH=PASS`.
- Following the metadata-only commit, no behavior tests were repeated; that is deliberately consistent with the project human-executor validation/finalization policy. Tests are **maintainer reported**, not rerun or independently witnessed by the documentation agent.

## Scope, non-claims and final disposition

The ratified PLAT063-C structural consolidation has been implemented and published. No new `RootCallTarget` per element, interpreter, optimizer, public iteration protocol, semantic change or new `Task`/Actor/Process overhead is claimed or authorized. Structural Bytecode reductions are **not** proof of improved Graal IR, CPU latency, throughput, compilation stability, allocation, Native Image support or Tier-2 performance.

[PERF045/#889](https://github.com/guillermomolina/protos/issues/889) remains the independently scoped measured `each` performance/inlining investigation. [I108/#909](https://github.com/guillermomolina/protos/issues/909) independently owns ordinary Closure-invocation operation consolidation.

**Disposition:** I109 implementation, full human-reported gates, structural metric, reviewed BCI guard, published implementation and required `0.3.371-SNAPSHOT` metadata are **COMPLETE** at the exact final product SHA. GitHub I109/#910 remains the live closure authority; this evidence supports `completed` closure without claiming unperformed measurements.
