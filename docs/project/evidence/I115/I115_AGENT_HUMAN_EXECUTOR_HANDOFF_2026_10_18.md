# I115 — Agent and Human Executor: implementation handoff

**Type:** IMPLEMENTATION. **Implement in:** `guillermomolina/protos`. **Issue:** [I115/#931](https://github.com/guillermomolina/protos/issues/931). **Approval record:** [PLAT066-C](../../decisions/platform/PLAT066_C_GRAALPY_SHAPED_TERMINAL_CLOSURE_CALL_CONVERGENCE.md). **Source baseline for explanations only:** `fcc5c405413d70df546c5a9e5fa2c57b78c8ec91` / `0.3.380-SNAPSHOT`. **The local product checkout is already at its real HEAD**: do not reset/fetch/check out the historical baseline. Preserve any concurrent edits, particularly I113 work.

## Agent / Human Executor contract

**Agent owns** source investigation against local HEAD, designing and editing exactly this approved implementation, creating or extending tests, inspecting diffs, and producing complete human commands. **Human owns** Maven/Make builds, tests, benchmarks, `git diff --check`, all Git state mutations and publishing the product repo. No agent-generated product commit, push or test execution. For `guillermomolina/protos-project-docs` only, the agent has express permission to edit and commit/push durable evidence; do not confuse it with product publication.

Do not run `git pull`, `git fetch`, `git checkout`, builds, program execution, test commands, package installation, or validation scripts as agent. No `set`, `pipefail`, `exit`, or `rg`; use `grep`. Edit with Python scripts/in-place file writes, not diff/patch files. Every command block has human-readable context before/after. Never touch `pom.xml` or `CHANGELOG.md` until the **complete modified source/test candidate passes human-run tests** and just before the human commit/push. Do **not** rerun tests after version/changelog edits. Keep one clean English one-line commit subject `I115: ...`.

### Governing code and specification, all available in local product checkout

First inspect `AGENTS.md`, `AGENTS.work/IMPLEMENTATION.md`, `AGENTS.work/REFERENCE.md`, `src/AGENTS.md`, scoped instructions, `spec/semantics/CALLABLES.md`, `OBJECT_MODEL.md`, `EXECUTION_AND_CONTROL.md`, `ERRORS.md`, `spec/concurrency/FUTURES_AND_TASKS.md`, and relevant tests. No need to browse/fetch other repositories or research GraalPy; all external mechanisms and links are summarized in this handoff. D013 remains fully unchanged. Check the implementation-blocker ledger only if already accessible locally; this task provides the necessary semantic gates below rather than requiring access to another repository.

## Exact target and reference (no additional research)

At `oracle/graalpython@graal-25.4.4`:
- `RootNodeCompiler.beginCallNAry(1)` emits the `CallUnaryMethod` unary operation for ordinary one-argument call.
- `CallUnaryMethodNode.callPFunction`: guards `isSingleContext()`, `hasSimpleSignature(cachedFunc,1)`, `func==cachedFunc`; assumption `cachedFunc.getCodeStableAssumption()`; `DirectCallNode` plus `FunctionDirectInvokeNode`.
- Generic `CallUnaryMethodNode.call` delegates to `CallNode.functionCall`; secondary `CallDispatchers.FunctionCachedCallNode.callFunctionCachedCt` caches executable code root rather than instance identity (`callee.getCode().getRootNode()==rootNode`), runs `CreateArgumentsNode`, `FunctionDirectInvokeNode`, and `SimpleDirectInvokeNode`.
- `SimpleDirectInvokeNode` uses `ExecutionContext.CallContext.prepareCall` / callee flags to supply caller data conditionally before `DirectCallNode.call`; `PArguments` is six header slots plus positional args, while Protos B5 remains FIVE + args.
- Reference paths (not needed to visit): `graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java`, `nodes/call/CallDispatchers.java`, `runtime/ExecutionContext.java`, `nodes/bytecode_dsl/PBytecodeDSLRootNode.java`, `compiler/bytecode_dsl/RootNodeCompiler.java`.

**Selected Protos algorithm:** `InvokeClosureOneTail` alone selects stable effective target, constructs the same fixed B5 header, invokes `DirectCallNode.call`, performs only necessary terminal completion. **No new DSL `@Operation`, no edits to other DSL operation bodies, no moved call selection into another operation, no new mandatory guest root or large new bytecode sequence.** Plain Java helpers and runtime/compiler producers/consumers can change. No Python guest semantics.

### Slice A: source/runtime admission, target and proof foundations — IMPLEMENT FIRST

Work in one cohesive patch focused on:
1. `ProtosStandardObjectProtocol` and `ProtosValueLookup`: lazily expose protected root `call` certificate with `Assumption` at PIC installation instead of boolean-certificate rebuild in each compiled hit. Honor canonical exact owner/home/value and all root mutation invalidation/restoration paths; do not eagerly rebuild at bootstrap.
2. `ProtosObjectValue` and its lexical binding authority: efficient own-`call` present/absent guard, avoiding the nonempty-authority `contains` host invoke for unrelated slots; correct after create/assign/remove/compose/replacement/bind/copy, no duplicate semantic store.
3. `ProtosClosureCallPreparation`: direct source-admission overload that does not query caller Prelude (unused for this canonical selection), while existing generic/cross-profile lookup remains authoritative.
4. `ProtosClosureValue`, `ProtosClosureExecutionPlanCell`, `ProtosClosureExecutionPlan`, `ProtosLanguageContext` if necessary: expose stable definition/Context-owned target, frozen plan, ReturnHome nonobservability and version/projection guard with no per-hit reprojection; rematerialize deferred plan only on actual miss. Keep the instance's captures/receiver fresh.
5. Add focused regressions in existing relevant tests for local root `call` mutation and restoration, unrelated slots, fresh Closure definition reused with different captures, foreign Context, deferred plan and observable ReturnHome. Avoid changing other DSL op bodies or ABI yet.

**Slice A terminal output:** edited-file/guard-by-guard summary, Git diff and exact compile/focal-test commands for human. No commands executed by agent. Human tests, then agent finalizes `pom.xml` and `CHANGELOG.md` once after green tests, human commits/pushes. Do not request new tests after those metadata edits.

### Slice B: B5+ deferred caller + same existing DSL operation — only after Slice A publication

Inspect current HEAD, then implement and test together:
1. **Approved PLAT060-B5+ amendment.** Current `ProtosFrameArguments` [0..4] are unchanged; the direct terminal entry may store **either exact caller `ProtosActivation` or the current caller B5 `Object[]` as a deferred reference** in slot3. Use a single typed Java resolver `resolveExactCaller(Object callerRef)` (or equivalent) that returns same already-published Activation or materializes/publishes via `ProtosFrameArguments.activation(callerArgs)`. Avoid wrapper/Task/guest Context creation in trivial admitted lane. Do not permit arbitrary arrays or cycles, and do not use a second authority. Generic/host/DEBUG entries keep their current legal B5 form. Preserve same ABI in both profiles. If `Object[]` reference conflicts with other frame-argument identities, use an equivalent zero/low-cost internal typed handle, but never a guest-observable new identity.
2. **Audit all readers, not just construction:** `ProtosFrameArguments.materializeCompactActivation`, `hasCompactHeader`, `compactCaller`, `unmaterializedInheritedProvenanceCaller`, `unmaterializedFullHeaderCaller`, `compactTask`, `ProtosInlineCallbackFrameBindings`, capture/deferred-owner resolution, Error/Task/debug/tooling consumers. A raw `(ProtosActivation)args[3]` cannot survive on the lane that now accepts `Object[]`. For cheap immutable inherited provenance, use compact-header facts only when provably exact. If the caller or current activation becomes observable/escapes/suspends, resolve exactly once.
3. **Root fact:** `EnterRoot` currently publishes Activation only when it installs frame-backed lexical authority. Record the root guarantee only where true; no eager root materialization merely to make a guard constant. Implemented in root metadata and `CanonicalToBytecodeLowerer`, not in bodies of other DSL operations.
4. **Selected callee non-suspension:** use the `EFFECTIVE_ROOT` (not CANONICAL_BODY/COMPLETE_INVOCATION) proof of the target actually cached, under its Context/projection. Within `InvokeClosureOneTail`, select direct proven-synchronous versus UNKNOWN path at PIC installation. Synchronous bypasses `isSuspended`/C′ transport but still preserves errors and control transfer. UNKNOWN retains suspension and exactly-once delegated completion.
5. **Only modify `ProtosBytecodeRootNode.InvokeClosureOneTail` DSL operation body:** admitted instance current, PIC key of reusable Context-owned executable target, root-Call assumption, no callerPrelude, compact B5+ caller when safely admitted, `DirectCallNode.call`, correct guest-value projection. Keep `guardedSource`, `rejectedSource`/generic and uncached semantic fallback correct. Keep `prepareClosureCallOneFallback` absent from direct compiled ancestry.
6. **Validation matrix:** fresh code-identical Closure with different captures; own-`call` shadow inserted after argument side effect (including non-Closure), root `Object.call` mutation/restore, 0/1/N other DSL operation conformance unchanged, published vs compact caller identity, escaped `context`, reentrant chain, Task/inherited dynamic control, true C′ suspension/cancellation/error/ensure, ReturnHome and nonlocal return, foreign Context, NORMAL/DEBUG tagging/reparse. Require unit/guest regressions then human-run full `make test`, then matched same-program TOP_LEVEL BGV. Compare real whole-root physical work; do not infer percentages or claim unverified GraalPy graph parity.

**Existing historical BGV:** original sha256 `53979e1181d69adce89c28526c252153ae5a948f78ed8cadc203f03e1d768187`, `After TruffleTier` TOP_LEVEL=1,080 nodes, 429 source-ancestry of `InvokeClosureOneTail`; 16 FULL instructions in two source roots. The BGV is **not** in the product repo; no agent-side benchmark execution.

## Exact first pbatch — Human Executor read-only preflight before Slice A edits

In the existing local checkout, run these *read-only* commands to show HEAD/concurrent changes and concrete edit sites:

```bash
git status --short
git rev-parse HEAD
git diff --check
grep -n "CanonicalRootCallCertificate\|isCanonicalRootCallSelectionForRuntime" src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardObjectProtocol.java
grep -n "hasLocalCallSlotForRuntime\|createLocalSlot\|assignLocalSlot\|removeLocalSlot\|composeLocalSlotsFrom" src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java
grep -n "directClosureCallSelectionForPreludeOrNull\|taskOwnedBytecodePlan" src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosClosureCallPreparation.java
```

After preflight the **agent makes all Slice A source/test edits** via Python in the working checkout. It then gives the human the narrowest compile/test gates; human executes them. Any Git stage commands must enumerate exactly modified paths, never `git add .` or `git add -A`. No `set`/`pipefail`/`exit`, no `rg`, no patch files. **Do not** attempt source editing blindly with a script authored against a historical HEAD.

At each validated/published slice, the agent captures exact `PROTOS_REVISION` and then publishes independent evidence to the approved docs repository. I115 closes only after both slices and source-linked final acceptance.
