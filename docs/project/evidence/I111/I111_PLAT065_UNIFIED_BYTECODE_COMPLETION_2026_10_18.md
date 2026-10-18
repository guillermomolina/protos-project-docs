# I111 — PLAT065-A unified Bytecode DSL implementation completion

**Date:** 2026-10-18
**Formal implementation:** [I111/#919](https://github.com/guillermomolina/protos/issues/919)
**Ratified owner decision:** [PLAT065/#918](https://github.com/guillermomolina/protos/issues/918), Candidate A
**Product repository:** `guillermomolina/protos`
**Durable record repository:** `guillermomolina/protos-project-docs`

## Exact product publications

| Slice | Product SHA | Published version | Scope |
| --- | --- | --- | --- |
| I111-A | [`63a178e3ea19af1caf3e2e43e947130c5bcfc792`](https://github.com/guillermomolina/protos/commit/63a178e3ea19af1caf3e2e43e947130c5bcfc792) | `0.3.375-SNAPSHOT` | Disable generator-wide automatic RootTag, explicitly tag genuine semantic roots; preserve lazy diagnostics, DEBUG tooling, locals and nested inline callback root tags. |
| I111-B + planned I111-C retirement/conformance work | [`edcb758fadb0e749cd45711cd664044d0b6db6bd`](https://github.com/guillermomolina/protos/commit/edcb758fadb0e749cd45711cd664044d0b6db6bd) | `0.3.377-SNAPSHOT` | Merge the two generated interpreters; consolidate operation owners and C-prime consumers; retire the old generator and its obsolete semantics-by-class distinction; update structural, native, PE and test expectations. |

The second commit message is **`I111: implement PLAT065-A unified Bytecode interpreter (0.3.377)`**. The published `pom.xml` and `CHANGELOG.md` identify version `0.3.377-SNAPSHOT`.

## Effective implementation at the published product revision

1. One Truffle Bytecode DSL interpreter class: `src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java`. It has a single `@GenerateBytecode`, `enableRootTagging=false`, `enableTagInstrumentation=true` and `enableRootBodyTagging=false`. The previous `ProtosSemanticBytecodeRootNode.java` is absent in the published product.
2. Semantic top-level, module and Closure roots use the physical-root field `semanticActivationRoot` and `isSemanticActivationRoot()` to preserve truthful source-root identity. Structured dispatch, Task and I/O C-prime roots are unmarked and cannot masquerade as semantic activations. `CanonicalToBytecodeLowerer` explicitly emits source `RootTag` only for genuine semantic roots.
3. The previous `ProtosBytecodeRootNodeGen` class name is reused by the **new unified generated interpreter**. Legacy PE/BCI baseline items with that spelling are not independent evidence of old/new graph equivalence; the project owner reports applicable local checks green after performing the necessary reconciliation.
4. The unified root retains `@Prolog` and `@EpilogReturn` and `InvokeClosureOneTail`, `InvokeClosureZeroTail`, `InvokeClosureArgumentsTail`; these were introduced by I112 before I111 completion and remain part of the acceptance contract. C-prime-specific completion and distinct primitive return projection remain separate.
5. The former parallel operation implementation hierarchy is no longer a separate generated interpreter. Shared prepared-call, lexical, callback, structured-control/map/each and send behavior resides in dedicated domain-owner classes. Physical C-prime roots, CallTargets and continuations remain real, independent roots under one generator. No change to normative Protos specification was requested or reported.
6. The published `ProtosI112TerminalCallBudgetTest` still uses `TARGET_IDENTITY_FULL = 16` and tests the full, unfiltered `root.getBytecodeNode().getInstructionsAsList()` over exactly two semantic source roots in NORMAL, with a separate absolute ceiling of 17. **Owner-reported green tests** support `<=16`; no exact count from runtime output is reproduced here.

## Validation provenance and bounded interpretation

**Human Executor report, 2026-10-18:** `git diff --check` clean and all locally required tests passed before publication of `edcb758fadb0e749cd45711cd664044d0b6db6bd`. An earlier focused run in the same handoff reported **907 tests, 0 failures, 0 errors, 0 skipped**. The owner subsequently reported completion of all local tests, including the applicable final gates. This documentation agent did not execute builds, tests, `make check`, compiler graphs, benchmarks, native-image compilation, or Git product publication; it does not claim new independent CI results.

**C closure accounting:** The original I111-C plan called for retiring redundant generator references and operations, tightening source/helper tag truthfulness, DEBUG and NORMAL acceptance, source and lexical fidelity, C-prime continuations and independent Builder-oracle regression. Those code and regression changes were grouped into the published B release and confirmed by the owner's successful local suite. A separate I111-C code publication is therefore **not required**. Further cross-project infrastructure-comparison research is tracked independently in PLAT064; it is not an I111 closure prerequisite.

## Legacy decisions / exact invariant reconciliation

- PLAT065-A superseded PLAT026's narrow no-manual-tag selection and PLAT042's two-generator topology with explicit owner approval; does not reopen other semantics.
- PLAT014 C-prime yield/resume/ensure/Error/ReturnHome, PLAT041 grouped lexical Object-body lowering, PLAT044/CLI008 failure-only lazy traces, PLAT058 Context-owned NORMAL/DEBUG physical profile, I107/I108, and I112 terminal-call budgets stay protected.
- No numerical performance improvement, Native Tier-2 success or library/guest-semantics redesign was demanded or demonstrated. Such claims require separate evidence.
- The older PLAT065 original source-baseline statements about **no** `@Prolog` and **two** interpreter classes are historical descriptions of the 2026-10-17 allocation snapshot, **not** descriptions of product `edcb758f`.

## Closure fields

`I111_A_PUBLICATION=63a178e3ea19af1caf3e2e43e947130c5bcfc792`
`PROTOS_REVISION=edcb758fadb0e749cd45711cd664044d0b6db6bd`
`PROJECT_VERSION=0.3.377-SNAPSHOT`
`I111_C_ACCEPTANCE=ABSORBED_IN_I111_B_PUBLICATION`
`HUMAN_REPORTED_DIFF_CHECK=PASS`
`HUMAN_REPORTED_LOCAL_TESTS=PASS`
`SPEC_CHANGE=NO`
`I112_FULL_BUDGET_TARGET=LE16`
`I111_CLOSE_READY=YES`

