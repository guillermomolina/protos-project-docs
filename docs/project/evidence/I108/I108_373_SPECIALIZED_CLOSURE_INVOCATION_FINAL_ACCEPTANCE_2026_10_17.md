# I108 — PLAT062-B specialized Closure invocation: final implementation and acceptance (2026-10-17)

**Formal work:** [I108/#909](https://github.com/guillermomolina/protos/issues/909), implementation of owner-ratified [PLAT062-B/#907](https://github.com/guillermomolina/protos/issues/907).
**Design authority:** [PLAT062-B decision](../../decisions/platform/PLAT062_SPECIALIZED_CLOSURE_INVOCATION_OPERATION_BOUNDARY.md) and [approval/research evidence](../PLAT062/PLAT062_B_ARCHITECTURE_AND_OWNER_APPROVAL_2026_10_16.md). This record documents implementation, not a new semantic or architectural decision.
**Exact product revision:** [`62ca4eb602432f3457c080954b716f8466b76983`](https://github.com/guillermomolina/protos/commit/62ca4eb602432f3457c080954b716f8466b76983), published to `guillermomolina/protos` `main`, implementing version **`0.3.373-SNAPSHOT`**. Previous product baseline: [`30b5c617fd135f7cc0759a34bb726fa30238f274`](https://github.com/guillermomolina/protos/commit/30b5c617fd135f7cc0759a34bb726fa30238f274) / `0.3.372-SNAPSHOT`.

## Published implementation boundary

One published I108 commit contains **12 changed files** as classified by GitHub (the previous `ProtosI101CGuardedBuilderOracleTest.java` is identified as a rename to `ProtosI101CIndependentBuilderReferenceTest.java`; earlier local status counted that rename as a deletion plus an untracked file).

- `ProtosSemanticBytecodeRootNode.java`: Truffle Bytecode DSL semantic invocation operations for ordinary Closure Call0, Call1 and CallN/spread, including frame-native inline callback variants; guarded definition-keyed PIC (bound 3) and exact ordinary generic fallback. Keep the original Bytecode DSL yield/resume structure wherever a call can suspend.
- `CanonicalToBytecodeLowerer.java`: emit arity-appropriate semantic operations rather than decomposing the equivalent ordinary dispatch into a caller-side chain; preserve evaluated-argument order and required structured continuation/finalization.
- `ProtosBytecodeRootNode.java`: narrowly coordinated preparation/selection integration without introducing a new universal Closure ABI.
- Regression/oracle changes: `ProtosI096InlineClosureLiteralCallTest.java`, renamed `ProtosI101CIndependentBuilderReferenceTest.java`, `ProtosI108BCallNDispatchTest.java`, `ProtosPerf042DefinitionKeyedDirectCallTest.java`, and updated `ProtosI101ProfileSpecializedLoweringTest.java` / `ProtosPerf025CallbackConsumerSpecializationTest.java`.
- `tools/java_generated_bytecode_bci_pe_baseline.json`: reconcile newly generated, statically classified cached-dispatch BCI transition topology.
- `pom.xml` and root `CHANGELOG.md`: exact `0.3.373-SNAPSHOT` metadata and I108 implementation entry committed together with the code; no follow-up version-only commit.

**Semantic invariants targeted and validated by the maintainer's integrated suite:** D013 `Object.call` selection happens after argument evaluation, including dynamic override; retain `methodHome`, receiver identity, generic selection and guest Error outcomes; D203 default/rest/spread semantics; stable B5 Closure-call ABI; ReturnHome/nonlocal-return, captured context/lexical authority, NORMAL/DEBUG, and resumable suspension without replay of evaluated arguments or duplicate completion. The existing `MaterializeClosure` remains the creation boundary; there is no new Closure identity model.

## Investigated failure and resolution

The integration initially exposed two obsolete test expectations, not demonstrated guest-semantic regressions:

1. `ProtosPerf025CallbackConsumerSpecializationTest#directSourceClosureCallStaysLazy` still expected `PrepareInlineClosureCall` where the physical operation is now `InvokeInlineClosureOne`. The check was updated; an excessive failure diagnostic that printed an entire bytecode instruction list was replaced by a concise missing-operation message.
2. `ProtosI101ProfileSpecializedLoweringTest#i101CD203ExcludedCallsKeepOrdinaryTopology` still expected the old `PrepareClosureCall*` decomposition. It now checks that the excluded D203 cases use ordinary Call0 and CallN/spread invocations without accidentally admitting inline-literal calls; the failure output was made concise.

During source-level integration, generated Java inspection caught repeated D013 lookups caused by separately generated `rejected` specializations and guard bindings. The source-backed correction removes those specializations, initializes the PIC target from its source Closure without D013, and retains a single D013 selection in the chosen operation path. The three subsequent targeted operation suites passed.

## Generated-code and BCI PE checks

A **read-only generated-Java structural audit** following the final source correction reported, for each `InvokeInlineClosureZero`, `InvokeInlineClosureOne`, `InvokeInlineClosureArguments`:

| Audit | Call0 | Call1 | CallN/spread |
| --- | ---: | ---: | ---: |
| Generated `selectCallOrNull` references | 3 | 3 | 3 |
| PIC target initializers from source | 1 | 1 | 1 |
| Generated `rejected` references | 0 | 0 | 0 |
| `UNCACHED` path lookup references | 1 | 1 | 1 |
| Structural verdict | PASS | PASS | PASS |

This is **structural evidence**, not a runtime count or benchmark. It confirms the intended generated dispatch shape; it does not measure latency, allocations or post-PE graph performance.

The TEST009-D generated BCI PE guard initially rejected the intentional new instruction topology. The reviewed candidate replaced exactly **2 obsolete** `TRY_DIRECT_CALL_*` cases with **10 new** `PROVEN_HANDLER_RETURN` cases for semantic Call0/Call1/CallN, inline equivalents and argument collection/snapshot. All **4 `DYNAMIC`** transitions were unchanged. The guard reported **1250 transitions**, **1246 `PROVEN`**, **4 `DYNAMIC`**, **0 `RISK`**, **0 `UNKNOWN`**. The strict baseline was reconciled, not loosened, and the subsequently rerun full check passed.

## Human-executor validation and publication

The maintainer reported:

- Six I108 focal tests **PASS** after the D013 source correction, including generic-first/PIC saturation, resume/no replay, selected methodHome and Error.
- Both corrected legacy structural tests **PASS** in their targeted two-test run.
- Final integrated `make test` **PASS**.
- Final integrated `make check` **PASS** after reviewed BCI baseline reconciliation.
- `git diff --check` clean. Metadata `pom.xml` / `CHANGELOG.md` finalized **after** functional checks, with **no additional builds/tests** after that change.
- Human-executed product commit and push succeeded: [`62ca4eb602432f3457c080954b716f8466b76983`](https://github.com/guillermomolina/protos/commit/62ca4eb602432f3457c080954b716f8466b76983). This SHA, `pom.xml` version, `CHANGELOG.md` entry and 12 changed paths have been independently read back using the GitHub connector.

These are **maintainer-reported test results**; the documentation agent has not rerun `make test`, `make check`, or guest programs.

## Disposition and deliberately bounded claims

The ratified PLAT062-B semantic-operation architecture is implemented for the ordinary Call0, Call1 and CallN/spread families in I108, including the structurally necessary Bytecode DSL continuation escape. No new Protos language semantics, ABI or speculative optimizer were authorized or introduced by this work.

**No measured speedup, reduced allocations, Tier-2 compiled graph reduction, GraalVM JIT stability gain, or Native Image improvement is claimed.** Any quantitative cross-language/runtime performance comparison belongs to [PERF042/#871](https://github.com/guillermomolina/protos/issues/871), not this implementation closure.

**Closure evidence:** `PROTOS_REVISION=62ca4eb602432f3457c080954b716f8466b76983`; implementation version `0.3.373-SNAPSHOT`; required durable evidence published under genuine owner `I108`. The live completion state remains governed by [I108/#909](https://github.com/guillermomolina/protos/issues/909), not this snapshot.
