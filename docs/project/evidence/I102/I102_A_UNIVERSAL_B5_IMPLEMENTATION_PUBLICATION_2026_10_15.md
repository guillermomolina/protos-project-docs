# I102-A — Published universal B5 ordinary Closure-call ABI: product and validation checkpoint

**Evidence date:** 2026-10-15.
**Formal owner:** [I102 / guillermomolina/protos#896](https://github.com/guillermomolina/protos/issues/896).
**Architectural authority:** [ratified PLAT060-B5](https://github.com/guillermomolina/protos-project-docs/blob/5c63fa39aaec4e088e2a2c5029ef63703aa9c100/docs/project/decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md) and [GITHUB021 approval evidence](https://github.com/guillermomolina/protos-project-docs/blob/5c63fa39aaec4e088e2a2c5029ef63703aa9c100/docs/project/evidence/PLAT060/PLAT060_B5_OWNER_APPROVAL_INVARIANT_RECONCILIATION_2026_10_15.md).
**Implementation commit / product revision:** [`1f23f43aaf82a95306731180c82823ae046d560e`](https://github.com/guillermomolina/protos/commit/1f23f43aaf82a95306731180c82823ae046d560e), parent `c7004f71cb05621dd5a7f89e46459542bd7d9c87`; commit message `102: universal B5 Closure-call ABI`.
**Published product version:** `0.3.359-SNAPSHOT` (verified in the product commit's `pom.xml` and `CHANGELOG.md` changes).
**Record classification:** implementation publication evidence for slice I102-A, **not** a new design decision, not I102-B acceptance, not final-tier performance evidence.

## Product delta verified read-only from the published commit

The published product revision changes **18 paths**: eight production Java files, eight existing test Java files, `pom.xml`, and `CHANGELOG.md`. GitHub reports **486 additions / 735 deletions**; these are source-diff statistics, not code size or performance measurements.

- `ProtosFrameArguments.java`: fixed ordinary five-position header and positional user values from index 5, including Call0/Call1 without an intermediate supplied-argument vector. Removes `MINIMAL_A/B/C/D` and the old layout-by-length dispatch. Adds `publishedInvocation(ProtosActivation)` for rich entry through B5; retains lazy publication in slot 0 and specialized activation access.
- `ProtosBytecodeRootNode.java` and `ProtosBytecodeClosureExecutionPlan.java`: source-backed ordinary rich invocation enters with a B5 `Object[]` instead of the old sole `[activation]` argument; preselected arity rejection remains a separate responsibility.
- `ProtosHostExecutableClosure.java` and `CanonicalToBytecodeLowerer.java`: remove the NORMAL-only physical `headerless` entry and its eligibility recording, while retaining normal admission/selection and profile-specialized emission.
- `ProtosSemanticBytecodeRootNode.java`: remove specializations and tests needed solely by short-header layouts; retain compact/full-header and materialized state handling.
- `ProtosBytecodeTagTreeNodeExports.java` and `ProtosExecutionMode.java`: separate immutable NORMAL/DEBUG lowering/tooling policy from physical frame-argument length; retain DEBUG scope/late-attach obligations. NORMAL may expose already-available scope information without acquiring a universal DEBUG instrumentation guarantee.

**Physical contract implemented for ordinary compact entry:**

```text
0    Closure, replaced only on publication by exact ProtosActivation
1    method receiver or direct-call marker
2    methodHome or explicit Task/null
3    caller activation
4    exact per-invocation ReturnHome or approved marker
5... supplied user arguments
```

Sources: [exact product commit](https://github.com/guillermomolina/protos/commit/1f23f43aaf82a95306731180c82823ae046d560e), [frame arguments at published revision](https://github.com/guillermomolina/protos/blob/1f23f43aaf82a95306731180c82823ae046d560e/src/main/java/com/guillermomolina/protos/execution/ProtosFrameArguments.java), [ordinary carrier](https://github.com/guillermomolina/protos/blob/1f23f43aaf82a95306731180c82823ae046d560e/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java), [scope export](https://github.com/guillermomolina/protos/blob/1f23f43aaf82a95306731180c82823ae046d560e/src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java).

### Test changes retained in product

The eight test paths are `ProtosI101BLiteralOneTest`, `ProtosI101ProfileSpecializedLoweringTest`, `ProtosPerf025H1LazyRootActivationTest`, `ProtosPerf032ExecutionModeTest`, `ProtosPerf032G7MinimalDirectHeaderTest`, `ProtosPerf033CanonicalCallableInteropTest`, `ProtosPerf038EResidualCallGraphTest`, and `ProtosPerf038FResidualPrimitiveCallTest` (all under `src/test/java/com/guillermomolina/protos/execution/`).

The commit's test diff replaces prior zero-/short-header assertions with B5 assertions and covers NORMAL/DEBUG host entry, debugger-scoped/reprepared calls, unmaterialized and published activation, exact ReturnHome, explicit Task, immediate method, supplied arguments, and interop. Historical class names retaining `MinimalDirectHeader` do not themselves establish remaining short-header behavior.

## Validation provenance (human executor, not independently reproduced)

The project owner reported in the current handoff:

- `git diff --check`: **CLEAN**.
- **All tests run locally:** **PASS**.

No exact command list, test count, run manifest, test timestamps, or local test execution SHA was supplied. These reports are recorded **as maintainer-reported validation**, not as agent-executed tests or independently archived integration evidence. The product commit and its published SHA are independently verifiable, but the reports must not be expanded into claims about test order, Native Image, compilation tier, or benchmarking.

The docs repository is published through the explicitly owner-authorized direct GitHub-content workflow. Its text was inspected for trailing whitespace before publication; a local `git diff --check` against this docs commit was **not** executed or claimed.

## I102-B review gates — still open

**Source-backed approval-consistency question:** the new `publishedInvocation(ProtosActivation)` creates a B5 array, publishes the exact activation in slot 0, and populates slots 1/2 and 4 plus supplied arguments, but **does not assign the caller slot 3**, leaving it `null` for a rich-entry-from-start. Its code comments state that the activation already owns provenance and that published-state readers must not consult slot 3. This may be semantically sound and preserves the *length* of B5, but the [ratified B5 positional contract](https://github.com/guillermomolina/protos-project-docs/blob/5c63fa39aaec4e088e2a2c5029ef63703aa9c100/docs/project/decisions/platform/PLAT060_B5_UNIVERSAL_ORDINARY_CLOSURE_CALL_ABI.md) names slot 3 as **caller activation**. The I102-B review must prove that this state-dependent nullable slot is consistent with the *exact* approved B5 invariant and with all published-state consumers; if it is a material exception to that invariant, it requires explicit owner reconciliation rather than silently treating green tests as design approval. **No new semantic exception is approved by this evidence note.**

Other I102-B closure work:
1. Complete the exact post-migration ordinary CallTarget producer/consumer census, including interop, rich entry, module/continuation classification, FrameNative, debugger scopes, and the published-activation state transition.
2. Verify the whole agreed correctness matrix: NORMAL/DEBUG, Call0/1/N, default/rest/error order, fresh/captured Context/D179, dynamic `Object.call` overrides, extracted methods/home, NLR/ReturnHome, cancellation/suspendible cleanup/Task, Actor/Process, cross-Context and host-interop paths.
3. Preserve the independent Bytecode DSL Builder oracle and source/physical-root observability gates. Compare retained pre/post PE and final-tier graphs, escape/allocations and performance **only when actual compatible, stabilized evidence is available**; record `NOT_MEASURED` otherwise. No extra benchmark policy or test rerun is authorized by this record.
4. Reconcile the remaining I102 acceptance checklist in the live Issue. Keep **I102 OPEN** until the implementation's whole acceptance has been substantiated; retain **PLAT060 CLOSED/RATIFIED**.

## Publication state

- `I102_A_PRODUCT_PUBLISHED=YES` at `PROTOS_REVISION=1f23f43aaf82a95306731180c82823ae046d560e`.
- `MAINTAINER_REPORTED_GIT_DIFF_CHECK=CLEAN`.
- `MAINTAINER_REPORTED_LOCAL_TESTS=PASS`.
- `I102_B_ACCEPTANCE=NOT_YET_RECONCILED`.
- `UNIVERSAL_B5_GRAAL_PERFORMANCE_GAIN=NOT_MEASURED`.
- `PLAT060_DESIGN=RATIFIED`; no ratification change.
- `I102_FORMAL_CLOSURE=NOT_YET_CLAIMED`.
