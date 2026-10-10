# I092 — Published compact canonical Boolean implementation (0.3.326-SNAPSHOT)

**Work item:** [I092 / guillermomolina/protos#870](https://github.com/guillermomolina/protos/issues/870).  
**Causal investigation:** [PERF041 / #863](https://github.com/guillermomolina/protos/issues/863).  
**Exact product revision:** [`7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`](https://github.com/guillermomolina/protos/commit/7b609c4ad0e146d6f02a7a3d036c6ff938ccad08).  
**Implementation version:** `0.3.326-SNAPSHOT`.  
**Status:** Product implementation published and functional validation reported PASS by human executor; post-implementation graph comparison **PENDING**.

## Published implementation

The product commit covers the three independent source-level preparation layers diagnosed in PERF041-A2:

1. **Closure literal capture:** deferred compact lexical capture without unconditional rich outer-activation construction, with materialization on observable-demand paths.
2. **Canonical Boolean send:** guarded `ifTrue` selection and a specific `CompactCanonicalBooleanCall` representation, retaining an ordinary generic fallback on noncanonical/missed selection.
3. **Selected callback:** compact prepared `ifTrue` callback selection feeding the approved PLAT044 B-prime inline region, rather than obligatorily entering generic child preparation. The Boolean fast path and generic fallback converge on a **single** inline region.

Production files: `CanonicalToBytecodeLowerer.java`, `ProtosBytecodeRootNode.java`, `ProtosFrameArguments.java`, `ProtosSemanticBytecodeRootNode.java`, and `ProtosLexicalEnvironment.java`. The same commit includes `pom.xml`, `CHANGELOG.md`, and focused regressions in `ProtosI092CompactBooleanTest.java`, `ProtosI072PhaseDPreparedCallSeparationTest.java`, and `ProtosPerf025LazyLexicalCaptureTest.java`. All source and test files are under `src/{main,test}/java/com/guillermomolina/protos/`.

This is an implementation mechanism under existing PLAT040, PLAT043 and PLAT044; no new language semantics, architecture ratification or standard Boolean generalization is claimed. `primitive-if-false` is a diagnostic contrast. `whileTrue` is outside I092.

## Validation provenance

The human executor reported:

- Relevant new I092 and PERF025/PERF026 focused regression commands: **PASS**.
- The inline instrumentation regression (`ProtosPerf026B1BooleanInlineCallbackTest.eligibleIfTrueLiteralCallbackRunsInlineInItsSourceRoot`): **PASS** after convergence to one B-prime region.
- `ProtosI072PhaseDPreparedCallSeparationTest` plus `ProtosPerf025LazyLexicalCaptureTest`: **18 tests, 0 failures, 0 errors, 0 skipped; BUILD SUCCESS** after reconciling the new concrete-call specialization and the valid absence of a rich activation.
- Final integrated `make test`: **PASS**, human-reported after these fixes.
- `git diff --check`: **clean**, human-reported.
- `main` push confirmed: `76de4365..7b609c4a`.

These are human-reported local checks. This record does not fabricate CI results, immutable raw test logs, quantitative performance gains or complete independent semantic auditing.

## Post-publication graph measurement — pending

Use the existing `guillermomolina/protos-benchmarks` cross-Truffle graph harness and policy without altering harness sources, scripts, workload definitions, warmup/steady budgets or product code. Capture `primitive-if-true`, `primitive-if-false` and `primitive-return-literal` at exact product SHA `7b609c4ad0e146d6f02a7a3d036c6ff938ccad08`, with correctness and stability gates.

Compare the selected `StructuredGraph / After TruffleTier` phase with the pinned [PERF041-A2 baseline](../PERF041/PERF041_A2_PRIMITIVE_IF_TRUE_BGV_TOPOLOGY.md), product `0db24f00ff2d92d642351d7f7535517fe01a55ce`: nodes **5193 / 892 / 39**, respectively. Record edges, blocks, `IfNode`, `FrameState`, `InvokeWithExceptionNode`, allocation-related nodes, and especially the presence or absence of normal-path B0 `ProtosFrameArguments.materializeCompactActivation`, outer generic Boolean preparation and selected inline callback preparation. Separate hot/normal paths from cold fallback, deoptimization state and PE-eliminable infrastructure. Require a stable selected graph; stop and report `NOT_STABLE` otherwise.

No post-I092 benchmark measurement, speedup or compiled-node reduction is asserted by this implementation record. Attach the exact later benchmark revision/artifact identities and derived comparison to I092 before its final completion decision.
