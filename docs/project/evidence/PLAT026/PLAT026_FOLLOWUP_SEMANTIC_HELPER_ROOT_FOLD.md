# PLAT026 follow-up — semantic/helper Bytecode root fold evidence

Date: 2026-10-01

## Identity

```text
WORK_ITEM=PLAT026-FOLLOWUP/#690
TRIGGERED_CONSUMER=PERF025-C/#758
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
SEMANTIC_CHANGE=NO
NEW_PLATFORM_DECISION=NO
```

This record retains the bounded static closure of the PLAT026 follow-up opened
for the semantic/helper two-Bytecode-root dispatch boundary. It is non-normative
project evidence. The already-ratified PLAT026 decision remains the durable
architecture authority.

No product code, tests, benchmark corpus, specification text, version, or
CHANGELOG was changed by this investigation.

## Result

```text
PLAT026_FOLLOWUP=LOCAL_OPTIMIZATION_POSSIBLE
NESTED_DISPATCH_ESTABLISHED=YES
SECOND_DISPATCH_COST=ESTABLISHED
ROOTTAG_COMPATIBILITY=PRESERVABLE
CONTINUATION_COMPATIBILITY=PRESERVABLE
SOURCE_DEBUGGER_COMPATIBILITY=PRESERVABLE
PERF010_UNBLOCKED=YES
PERF025_C_IMPLEMENTATION_READY=YES
CARRIER_REQUIREMENT=REMOVABLE_AFTER_FIX
```

`SECOND_DISPATCH_COST=ESTABLISHED` is scoped to the duplicated dispatch/host-stack
cost relevant to BUG008 and PERF025-C. Current source explicitly records that a
Protos-level Closure call crosses two nested Bytecode `CallTarget` invocations
and therefore consumes more host stack per guest recursion level. This record
does not claim a new quantitative wall-time attribution for the second dispatch.

## Current execution topology

For an ordinary source-backed Closure, current HEAD establishes:

```text
caller Bytecode root
  -> EnterClosureCall
  -> PreparedClosureCall.bodyTarget()
  -> CallTarget A: ProtosSemanticBytecodeRootNode
       -> InvokeSemanticHelper
       -> CallTarget B: ProtosBytecodeRootNode
            -> parameter binding
            -> canonical Closure body
            -> StatementTag / ExpressionTag / CallTag regions
            -> Bytecode DSL yield / ContinuationResult
            -> control/error/NLR machinery
```

`ProtosBytecodeClosureExecutionPlan` creates the helper activation root with
`CanonicalToBytecodeLowerer.lowerClosureActivationRoot(...)`, then wraps that
helper target with `ProtosSemanticBytecodeRootNode.wrap(...)`.

The semantic wrapper itself owns the automatic semantic `RootTag` identity and
source span, then delegates execution to the helper target and composes the
helper `ContinuationResult`. It does not define a second Protos execution
semantics.

## Local fold boundary

The bounded implementation seam is:

1. lower a genuine source-backed Closure activation directly as a
   `RootTag`-enabled semantic Bytecode DSL root containing the existing
   parameter/body lowering;
2. make the Closure execution plan use that semantic root's `CallTarget`
   directly;
3. remove the ordinary Closure
   `semantic-root -> InvokeSemanticHelper -> helper-root` nested target;
4. keep implementation-only roots, including Object-construction/body helper
   roots, on an untagged helper configuration; and
5. preserve the existing continuation, invocation, control-transfer, source,
   scope, Context ownership and lexical-authority mechanisms.

The local fold changes internal generated-root topology only. It does not
promote a helper root to guest semantic identity and does not change observable
Protos semantics.

## PLAT026 invariant/delta consistency check

The follow-up was checked against the explicit owner-approved PLAT026 invariants.

| Ratified PLAT026 invariant | Follow-up result |
| --- | --- |
| Top-level/module/Closure activations are truthful semantic roots | PRESERVED — the Closure activation remains the one semantic guest root |
| Semantic roots use automatic `RootTag` | PRESERVED |
| Object-construction/body and future implementation-only roots remain untagged | PRESERVED |
| Durable authority is semantic-root vs helper-root truthfulness, not Java class/root/CallTarget count | PRESERVED — this is the key authority permitting the fold |
| `RootBodyTag` remains deferred/unprovided | PRESERVED |
| StatementTag/CallTag semantics remain unchanged | PRESERVED / implementation requirement |
| Root tagging does not alter lookup, parameter/default evaluation, scheduling, suspension, unwind or failures | PRESERVED / implementation requirement |
| Source ownership remains PLAT004-authoritative | PRESERVED |
| Continuation ownership remains PLAT014-authoritative | PRESERVED |
| Helper roots do not leak as debugger roots/stops | PRESERVED / implementation requirement |
| No global root registry or debugger lock is introduced | PRESERVED |
| Exact generated Java class count and operation-sharing mechanism are implementation details | PRESERVED |

```text
DECISION_INVARIANT_CONSISTENCY=PASS
REOPENED_PLAT026_INVARIANT=NONE
NEW_OWNER_APPROVAL_REQUIRED=NO
```

The result therefore consumes the already-ratified PLAT026 architecture; it
does not amend or supersede it.

## PLAT014 / continuation compatibility

The real C-prime continuation structure already lives in the canonical helper
lowering:

```text
EnterClosureCall
-> child result
-> while IsContinuation
   -> Yield
   -> ResumeContinuation
-> FinishClosureCall
```

Moving that existing lowering into the semantic Closure root does not require
changing `ContinuationResult`, Task suspension publication, cancellation,
ensure/error handling, non-local return, or resumed scheduler participation.

The implementation must not change those mechanisms merely as part of the root
fold.

## Source and debugger compatibility

The semantic Closure root can retain:

```text
automatic RootTag = enabled
RootBodyTag = disabled
semantic source span = unchanged
StatementTag membership = unchanged
ExpressionTag membership = unchanged
CallTag membership = unchanged
debugger scope/value authority = unchanged
```

Implementation-only roots remain untagged. Therefore the fold can reduce
physical `CallTarget` cardinality without changing tool-visible Protos root
cardinality.

## Current lexical-state constraint

Current HEAD also contains PERF013/I068 captured-lexical optimization:

- lexical owner roots and lexically nested Closures are grouped in a shared
  `BytecodeRootNodes` group where possible;
- same-group proven captures may use `MaterializedLocalAccessor`; and
- an existing runtime-authority fallback remains available when owner and child
  do not share the required generated group.

PERF025-C must preserve same-group semantic Closure nesting where practical and
must preserve the existing fallback at any semantic/helper partition boundary.
That is an implementation constraint, not a new platform decision.

## BUG008 carrier consequence

Current source records the dedicated 64 MiB guest carrier as a workaround for
host-stack amplification from two nested Bytecode `CallTarget` invocations per
Protos-level Closure recursion.

Therefore:

```text
CARRIER_REQUIREMENT=REMOVABLE_AFTER_FIX
```

means the special enlarged-stack requirement may be removed once the ordinary
Closure double-target path is actually eliminated and regression evidence
confirms the original recursion corpus no longer needs it.

It does **not** authorize dropping the reusable-session serialization contract.
If the dedicated carrier thread/queue is removed, the session must still
serialize guest entry by a cheaper implementation-local mechanism.

## PERF025-C routing

PERF025-A and PERF025-B are already published. This closure releases the
remaining PERF025-C slice:

```text
NEXT_SLICE=PERF025-C
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEW_JFR_REQUIRED=NO
NEW_BENCHMARK_CAMPAIGN_REQUIRED=NO
NEW_PLAT_DECISION_REQUIRED=NO
BUG008=#681 CLOSED_DO_NOT_REOPEN
```

PERF025-C remains responsible for implementation, focused/full correctness,
versioning/CHANGELOG, and the before/after measurement required by PERF025.
The PLAT026 follow-up itself makes no implementation claim.

## Materially inspected product/authority surfaces

```text
AGENTS.md
AGENTS.work/DESIGN.md
AGENTS.work/COORDINATION.md
AGENTS.work/PERFORMANCE.md
AGENTS.work/REFERENCE.md
src/AGENTS.md
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeClosureExecutionPlan.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTaskExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosClosureInvoker.java
src/main/java/com/guillermomolina/protos/execution/ProtosSourceCompiler.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java
src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedSession.java
```

Also reconciled against the ratified durable records for PLAT005, PLAT008,
PLAT014, PLAT026 and PLAT034, plus live issues #690 and #758.

## References

- `guillermomolina/protos#690` — PLAT026 follow-up
- `guillermomolina/protos#758` — PERF025
- `guillermomolina/protos#681` — historical BUG008, remains closed
- Protos revision `6811d0cef3735d39ffd3801b3bae6ef48318bb66`
- `docs/project/decisions/platform/PLAT026_BYTECODE_PRODUCTION_ROOT_TAG_COMPATIBILITY_BOUNDARY.md`
- `docs/project/evidence/PERF025/PERF025_A_DIRECT_ROOT_TASK_DISPATCH.md`
- `docs/project/evidence/PERF025/PERF025_B_PREPARED_TOP_LEVEL_CALLABLE.md`
