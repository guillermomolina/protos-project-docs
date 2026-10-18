# PLAT066-C — Source comparison, physical ownership and exact owner approval

**Date:** 2026-10-18. **Issue:** [#930](https://github.com/guillermomolina/protos/issues/930). **Product source revision:** [`fcc5c405413d70df546c5a9e5fa2c57b78c8ec91`](https://github.com/guillermomolina/protos/commit/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91) (`0.3.380-SNAPSHOT`). **GraalPy comparator:** [`graal-25.4.4`](https://github.com/oracle/graalpython/tree/graal-25.4.4). **Approval/decision:** [PLAT066-C durable decision](../../decisions/platform/PLAT066_C_GRAALPY_SHAPED_TERMINAL_CLOSURE_CALL_CONVERGENCE.md). **Implementation:** [I115/#931](https://github.com/guillermomolina/protos/issues/931).

## Comparator correctness

GraalPy `RootNodeCompiler.beginCallNAry(1)` emits `CallUnaryMethod` for the ordinary one-argument positional form. [`CallUnaryMethodNode`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallUnaryMethodNode.java) has a `callPFunction` direct identity specialization with `isSingleContext`, `hasSimpleSignature(cachedFunc,1)`, `func == cachedFunc`, code-stable assumption and `DirectCallNode`. **This instance guard is not the whole GraalPy reference:** its generic fallback uses `CallNode.functionCall` and [`FunctionCachedCallNode.callFunctionCachedCt`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java), whose cache guards by `callee.getCode().getRootNode() == rootNode`, builds arguments via `CreateArgumentsNode`, then `FunctionDirectInvokeNode` → `SimpleDirectInvokeNode` → `DirectCallNode.call`.

GraalPy [`ExecutionContext.CallContext`](https://github.com/oracle/graalpython/blob/graal-25.4.4/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/ExecutionContext.java) conditionally materializes caller state according to target flags. GraalPy `PArguments` has six header slots plus a supplied argument, *not* a smaller ABI than Protos B5's five slots.

Protos:
- [`InvokeClosureOneTail`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosBytecodeRootNode.java#L9724-L10006) source specialization admission; `enterDirect`, cold defer, rejected/generic.
- [`ProtosStandardObjectProtocol`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/protocols/values/ProtosStandardObjectProtocol.java#L73-L129) root `call` certificate and per-hit boolean status.
- [`ProtosObjectValue`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java#L297-L313) local `call` presence via lexical authority.
- [`ProtosClosureCallPreparation`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosClosureCallPreparation.java) admission and entered-Context target resolution.
- [`ProtosClosureValue`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/runtime/lexical/ProtosClosureValue.java) final definition/capture/home, deferred execution plan, projection.
- [`ProtosLanguageContext`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/host/embedding/ProtosLanguageContext.java#L358-L411) Context-owned reusable projection by template.
- [`CanonicalNonSuspensionAnalysis`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalNonSuspensionAnalysis.java#L68-L92) EFFECTIVE_ROOT vs COMPLETE_INVOCATION subject.
- [`CanonicalToBytecodeLowerer.emitTerminalClosureCall`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/bytecode/lowering/CanonicalToBytecodeLowerer.java#L3992-L4078) one DSL terminal op + tags/return.

## B5 concrete conflict, not merely a speculative change

[`ProtosFrameArguments.java`](https://github.com/guillermomolina/protos/blob/fcc5c405413d70df546c5a9e5fa2c57b78c8ec91/src/main/java/com/guillermomolina/protos/execution/bytecode/activation/ProtosFrameArguments.java) defines B5:
```text
[0] invoked Closure | published Activation
[1] receiver | DIRECT marker
[2] methodHome | explicit Task/null
[3] exact ProtosActivation caller (current unpublished contract)
[4] exact ReturnHome / unobservable marker
[5...] supplied positional values
```
The direct terminal constructor `compactDirectClosureCallOneNoHome(closure, exactCaller, supplied0)` demands a rich caller before invoking the callee (lines 251–265). Even an unobserved caller causes `PrepareSendArguments.exactCaller` to import conditional `materializeCompactActivation` into the direct source graph.

Exact dependent current code, requiring a change under B5+:
- `ProtosFrameArguments.materializeCompactActivation` lines 403–481 **casts** `arguments[CALLER_INDEX]` to `ProtosActivation`; collects prelude, actor/module/domain and Task/dynamic-control provenance.
- `compactCaller` lines 772–778 only accepts `ProtosActivation`.
- `hasCompactHeader` lines 824–830 validates `CALLER_INDEX instanceof ProtosActivation`.
- `unmaterializedInheritedProvenanceCaller` and `unmaterializedFullHeaderCaller` (~694–726) cast slot 3 directly.
- `compactTask` (~784–794) accesses the caller Task.
- `ProtosInlineCallbackFrameBindings` uses `compactCaller` and inherited provenance.
- `ProtosLexicalEnvironment.deferredCompactOwner(arguments, resolver, outer)` is a **precedent** for retaining a B5 arguments array and publishing exact Activation only on later demand, not permission to bypass semantic identity.

**Proposed B5+ mechanism:** in the direct admitted terminal lane only, slot3 may be `ProtosActivation` or a deferred reference to the *current caller's B5 argument array*. A centralized `resolveExactCaller` resolves the reference to `ProtosFrameArguments.activation(callerArguments)`; the latter publishes the exact Activation in its argument 0, preserving single identity. Consumers needing only inherited immutable provenance may read it from the compact authority without materializing, *provided* exact Task/Prelude/module/domain and dynamic control semantics are proven. Consumers needing caller guest identity or escaped scope materialize once. Published rich B5 entries (caller slot null) are unchanged. Reject malformed cyclic/self references and avoid introducing a second authoritative identity or copying mutable execution-context state. Preserve same header/offset and same NORMAL/DEBUG semantics.

**Risk/acceptance:** lazy resolution can recursively resolve a caller chain; prove finite valid ancestry and exact identity, avoid eager wrapper or retention overhead for the trivial case, test with caller escaping after return, re-entrant/nonlocal return, Context projected call, active Task vs no Task, DEBUG inspection, native/generic fallback, and published/unpublished transitions.

## Existing Tier-1 physical evidence

[Original I114 BGV investigation](../I114/I114_380_TIER1_BGV_PATH_ATTRIBUTION_2026_10_18.md): user-supplied SHA256 `53979e1181d69adce89c28526c252153ae5a948f78ed8cadc203f03e1d768187`; TOP_LEVEL `After TruffleTier` graph 3, 1,080 nodes, 429 general source-ancestry `InvokeClosureOneTail` nodes. BGV invocation sites `ProtosLexicalBindingAuthorityCalls.contains` #2200, `rebuildCanonicalRootCallCertificate` #2254, `materializeCompactActivation` #2733, cold `ProtosTailCallContinuation.deferred` #2837, direct `OptimizedCallTarget.callBoundary` #3528. Generic fallback methods absent in this direct graph. Inclusive ancestry counts overlap and must not be summed into alleged causal savings. No matched GraalPy BGV exists in this record.

## Approval sequence, scope and nonclaims

1. Owner required comparison of **the same actual DSL invocation behavior** and source-level recursive implementation, not isolated Protos-only PE attribution.
2. Owner explicitly demanded changes **outside the operation**, without adding/altering other DSL operations, and instructed that D/PLAT design decisions must not block improvement merely because previously ratified.
3. Owner said **"aprobados los cambios"** and authorized live Issue coordination and agent-direct `guillermomolina/protos-project-docs` commit/push.
4. On B5/D013 clarification, owner explicitly selected: **"D013, lo dejamos para mas adelante por si hace falta, y el resto ajustar lo que haga falta."** Thus D013 semantics stay untouched; the narrow B5+ physical carrier/reader contract and compatible platform helper adjustments are authorized.

**GITHUB021:** no new guest-visible semantic kind, caller identity, Context presence, Task/error behavior, or second ABI; B5 slot3 representation relaxes behind a verified lazy-to-exact resolver, with same offsets and profiles. Compilation fact proofs and Context protection remain mandatory. This is an approved source-backed architecture proposal, **not** executed validation, a claim of changed product code, or PE improvement.
