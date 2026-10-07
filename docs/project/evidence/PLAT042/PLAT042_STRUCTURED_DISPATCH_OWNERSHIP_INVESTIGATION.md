# PLAT042 — structured-dispatch interpreter ownership investigation

Status: **RATIFIED — Candidate B′ selected**

Date: 2026-10-01

## Identity

~~~text
WORK_ITEM=PLAT042/#760
PARENT_WORKSTREAM=PERF025/#758
TRIGGER=PERF025-C1c stop gate 10
PRODUCT_REVISION=595d547b2e9714a185a3cfceadf74565229f43e7
PRODUCT_VERSION=0.3.132-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
SEMANTIC_CHANGE=NO
~~~

PLAT041 remains authoritative for its already-implemented C1a+C1b invariants.
PLAT042 decides only the remaining structured-dispatch ownership/topology needed
for C1c.

## Reproduced product facts

Revision-bound static inspection gives:

~~~text
CURRENT_SOURCE_LOWERER_OPERATION_NAMES=203
SIX_CPRIME_DIRECT_OPERATION_NAMES=53
PREPARED_STRUCTURED_TRANSITIVE_OPERATION_NAMES=144
ACTUAL_HELPER_UNION_OPERATION_NAMES=183
PROTOS_BYTECODE_ROOT_NODE_OPERATION_ANNOTATIONS=223
~~~

The 183-operation helper union is 90.1% of the source lowerer's current
operation-name surface.

The large structured-dispatch lowering body itself emits 137 unique operation
names. If that body is outlined while retaining the existing scoped/ordinary
prepared-call composition, the semantic source lowering surface becomes:

~~~text
SEMANTIC_SOURCE_OPERATION_NAMES_AFTER_OUTLINING=74
HELPER_UNION_OPERATION_NAMES=183
OVERLAP_BETWEEN_THOSE_SURFACES=18
COMBINED_UNIQUE_OPERATION_NAMES=239
GENERATED_OPERATION_INCLUSIONS_IF_EXACTLY_PARTITIONED=257
~~~

These are operation-name counts, not byte-size predictions. Specialization size
and generated-interpreter machinery are not uniform.

## Why the transitive requirement exists

`ProtosTaskCPrimeEntryExecution.createPlan(...)` builds its root with
`ProtosBytecodeRootNodeGen` and invokes
`CanonicalToBytecodeLowerer.emitPreparedInvocationForRuntime(...)`.

That path owns the structured protocols for:

- Object.call composition;
- import;
- ensure;
- Error.handle;
- while;
- Boolean callbacks;
- Array/Bytes/Environment/IdentityMap/Map iteration;
- Array/Map matching and case-of;
- Map lookup/update/remove structured callback paths; and
- nested continuation-aware prepared invocation.

`EnterNestedStructuredDispatch` resolves the Context-local Task C-prime target
and enters it when a prepared child itself requires structured dispatch.

The Task C-prime plan is lazily cached in `ProtosLanguageContext` and is stable
for that Context after creation.

## Existing cross-root continuation path

`emitScopedPreparedInvocation(...)` already does:

~~~text
if prepared.requiresStructuredDispatch():
    child = EnterNestedStructuredDispatch(prepared)
    while child is ContinuationResult:
        resumeValue = yield child
        child = ResumeContinuation(prepared, child, resumeValue)
    result = child
else:
    ordinary prepared invocation
~~~

This means the semantic-to-structured boundary proposed below does not invent a
new continuation architecture. It reuses a boundary already exercised for
nested structured calls.

For a standard structured `while`, the helper boundary is entered once for the
`while` invocation. Its Bytecode `While` loop remains inside the structured
dispatcher. Ordinary condition/body Closures execute through ordinary Closure
calls and do not re-enter the structured dispatcher unless those child calls are
themselves structured.

The same ownership property applies to `each`, matching, ensure and the other
structured protocols.

## Current Truffle framework evidence

Current GraalVM/Truffle API documents that each `@GenerateBytecode` root
generates a complete bytecode encoding, optimizing interpreter and Builder.
`OperationProxy` shares/reuses operation *definitions* as an organization and
migration mechanism; it does not merge separately generated interpreters.

Official API:

- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecode.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/OperationProxy.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/DirectCallNode.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/IndirectCallNode.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/CallTarget.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/BytecodeRootNodes.html

Truffle's DSL guidelines explicitly advise minimizing PE/interpreter code
duplication because it affects host inlining, Native Image AOT code and retained
Graal IR used for runtime compilation.

Reference:

- https://www.graalvm.org/dev/graalvm-as-a-platform/language-implementation-framework/DSLGuidelines/

`DirectCallNode` exists specifically for constant-target calls and supports
call-site inlining and call-site-sensitive target duplication. A raw
`CallTarget.call(...)` may also inline when the target receiver is a
partial-evaluation constant, but `DirectCallNode` supplies the explicit
stable-call abstraction.

GraalVM 25.4 changed guest-language inlining to use Graal IR call-site
frequencies rather than runtime direct-call counters. This reinforces the need
for truthful branch/call frequency evidence but does not make a stable direct
structured-dispatch boundary inherently non-inlineable.

## Production-runtime comparison

Current Oracle SimpleLanguage Bytecode DSL uses one semantic Bytecode
interpreter and makes extensive use of `OperationProxy` to organize operation
semantics.

Current GraalPy's `PBytecodeDSLRootNode` likewise has a large production
Bytecode interpreter with many operation proxies rather than relying on a
framework facility that merges independently generated interpreters.

Neither supplies a current supported Bytecode DSL mechanism for one generated
root group to have per-root automatic RootTag enablement.

Relevant current upstream source:

- `oracle/graal`:
  `truffle/src/com.oracle.truffle.sl/src/com/oracle/truffle/sl/bytecode/SLBytecodeRootNode.java`
- `oracle/graalpython`:
  `graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode_dsl/PBytecodeDSLRootNode.java`

The portable lesson is not to duplicate a nearly complete generated interpreter
merely to change root classification when the execution domain can instead be
partitioned at a real semantic/helper call boundary.

## Candidate A — full semantic interpreter + near-full untagged duplicate

Shape:

~~~text
tagged semantic source interpreter
    current ~203-op source surface

untagged helper interpreter
    ~183-op helper/structured surface
~~~

Pros:

- conceptually straightforward;
- preserves current inlined structured dispatch in semantic source roots;
- PLAT026/PLAT014/PERF013 can be preserved.

Cons:

- duplicates approximately 183 operation names across generated interpreters;
- source+helper generated operation inclusion is approximately 406 before
  considering generated built-ins/configuration machinery;
- contrary to Truffle footprint guidance;
- requires a large proxy/share conversion but `OperationProxy` does not
  deduplicate generated interpreter code;
- Native Image size/build-time risk is high.

~~~text
CANDIDATE_A=SUPPORTED_BUT_REJECTED_AS_DEFAULT
~~~

## Candidate B′ — semantic source interpreter + single untagged structured/C-prime owner

Recommended.

Shape:

~~~text
tagged semantic source Bytecode interpreter
    top/module/Closure roots
    Object body inline
    ordinary source execution
    ordinary prepared Closure invocation
    ~74 lowering operation names after structured body outlining

    structured prepared call
        -> stable Context-owned structured-dispatch CallTarget
        -> untagged structured/C-prime interpreter

untagged structured/C-prime Bytecode interpreter
    full structured dispatch
    Task C-prime entry
    existing Text/Buffered/I/O C-prime roots
    nested structured dispatch
~~~

The current generated-interpreter count can remain **two**:

1. the current semantic wrapper role evolves into the real semantic source
   interpreter; and
2. the current untagged `ProtosBytecodeRootNode` remains the structured/C-prime
   substrate.

The implementation does not need to create a third generated interpreter.

Two implementation levels are compatible with this architecture:

### B′-1 — low-risk additive migration

Keep the current untagged interpreter's full operation set initially, even if
some source-only operations become unused there, and add only the operation
surface required by the semantic source interpreter.

Approximate operation-name inclusions:

~~~text
CURRENT_UNTAGGED_ROOT_ANNOTATIONS=223
SEMANTIC_SOURCE_LOWERING_SURFACE~=74
TOTAL_INCLUSIONS_PROXY~=297
~~~

This is far below A's near-full 223+183 duplication and minimizes simultaneous
operation migration.

### B′-2 — exact partition cleanup

After correctness/topology is stable, remove helper-unused source operations
from the untagged interpreter so the approximate surfaces become 183 + 74 with
18 shared names.

That cleanup does not change the architecture and need not be a ratification
gate.

### Structured-call optimization

The current helper target is stable per `ProtosLanguageContext`.

The structured-entry operation can therefore use the same general specialization
shape already used by ordinary Closure entry:

~~~text
stable target -> cached DirectCallNode
polymorphic/excess contexts -> IndirectCallNode fallback
~~~

This preserves multi-Context correctness while giving Truffle an explicit
inlinable stable call site.

The exact cache limit is implementation-local.

### Why the extra call is bounded

The call boundary is paid per structured protocol invocation.

It is not paid for:

- ordinary source Closure calls;
- each Bytecode loop iteration of one structured `while`;
- each element merely because an `each` dispatcher is looping.

Child callbacks still make their own ordinary Closure calls as they do today.
Only a child that itself requires structured dispatch crosses the structured
boundary recursively.

### Continuation/control compatibility

This boundary already exists for nested structured calls.

The existing code already transports:

- `ContinuationResult`;
- yield/resume;
- Protos control transfers;
- AbstractTruffleException;
- module initialization failure; and
- runtime failure mapping

across `EnterNestedStructuredDispatch`.

Thus B′ changes ownership/frequency of an existing PLAT014 composition boundary,
not its semantic protocol.

~~~text
CANDIDATE_B_PRIME=RECOMMENDED
~~~

## Candidate C — current-framework shared generated dispatcher without a root boundary

Investigated mechanisms include operation proxies, generated-root groups and
current Bytecode DSL configuration.

No supported current mechanism was found that simultaneously provides:

- one shared generated instruction implementation across two differently
  RootTag-configured root populations;
- one shared Builder/root group;
- per-root automatic RootTag classification; and
- no CallTarget boundary.

`OperationProxy` shares operation definitions, not the generated interpreter.
`BytecodeRootNodes` groups roots from one parse of one generated interpreter.

A future Bytecode DSL per-root automatic classification facility remains a valid
escape hatch.

~~~text
CANDIDATE_C=CURRENT_API_NOT_AVAILABLE
~~~

## Candidate D — keep the semantic wrapper

Shape:

~~~text
tagged semantic shell
    -> untagged full source/structured interpreter
~~~

Pros:

- smallest implementation risk;
- minimum new generated code;
- already validated tooling/correctness.

Cons:

- ordinary source Closures retain the extra CallTarget;
- BUG008's universal source-call stack amplification remains;
- PERF025-C's primary objective is not achieved.

D remains the fallback if B′ implementation evidence reveals an unacceptable
structured-dispatch regression.

~~~text
CANDIDATE_D=SAFE_FALLBACK_NOT_PREFERRED
~~~

## GITHUB010 scorecard

Scores 1-5. These compare current implementable choices, not hypothetical future
Bytecode DSL support.

| Criterion | A near-full duplicate | B′ structured owner | C current API | D wrapper |
| --- | ---: | ---: | ---: | ---: |
| Correctness / invariant preservation | 4 | 5 | 5 concept | 5 |
| Protos architecture alignment | 3 | 5 | 5 concept | 4 |
| Future-option resilience | 3 | 5 | 5 concept | 3 |
| Scalability | 2 | 5 | 5 concept | 3 |
| Conceptual simplicity | 3 | 4 | 5 concept | 5 |
| Portability / implementation freedom | 3 | 5 | 5 concept | 4 |
| Runtime/resource cost | 2 | 4 | 5 concept | 3 |
| Failure / operability | 3 | 4 | 5 concept | 5 |
| Reversibility | 3 | 5 | 5 concept | 5 |
| Evidence maturity | 4 | 4 | 1 current | 5 |
| **Total / 50** | **30** | **46** | **46 concept / unavailable** | **42** |

### Owner-focus view

| Candidate | Future endurance | Scalability | Protos philosophy | Mean |
| --- | ---: | ---: | ---: | ---: |
| A | 6.0 | 4.0 | 6.0 | 5.33 |
| **B′** | **9.5** | **9.0** | **9.5** | **9.33** |
| C future API | 10.0 | 10.0 | 10.0 | 10.0 unavailable |
| D | 7.0 | 6.5 | 7.5 | 7.00 |

## GITHUB021 invariant/delta review

### PLAT041

Preserved from the approved C′ architecture:

~~~text
CURRENT_ACTIVATION_LOWERING_AUTHORITY=PRESERVED
OBJECT_BODY_INLINE=PRESERVED
OBJECT_BODY_SEMANTIC_ROOT=NO
OBJECT_BODY_LEXICAL_CONTEXT=NO
SEMANTIC_SOURCE_ROOT_AUTOMATIC_ROOT_TAG=PRESERVED_AS_TARGET
~~~

Explicit delta requiring project-owner approval:

~~~text
OLD_C1C=compact untagged infrastructure interpreter
B_PRIME_C1C=single untagged structured-dispatch/C-prime owner
              + structured prepared calls cross one helper CallTarget
PLAT041_DELTA=EXPLICIT
~~~

### PLAT026

~~~text
TRUTHFUL_AUTOMATIC_ROOT_TAG=PRESERVED
HELPER_ROOT_TAG=NO
ROOT_BODY_TAG_DEFERRED=PRESERVED
NO_MANUAL_ROOT_TAG=PRESERVED
PLAT026_INVARIANT_DELTA=NONE
~~~

### PLAT014

~~~text
CONTINUATION_RESULT_COMPOSITION=PRESERVED
YIELD_RESUME=PRESERVED
CONTROL_TRANSFER_BRIDGE=PRESERVED
NO_REPLAY=PRESERVED
NEW_CONTINUATION_KIND=NO
PLAT014_INVARIANT_DELTA=NONE
~~~

Physical ownership changes because source-level structured calls use the
already-existing cross-root composition path.

### PERF013 / PLAT036

~~~text
LEXICAL_OWNER_AND_NESTED_CLOSURE_SAME_GENERATION=PRESERVED
MATERIALIZED_LOCAL_FAST_PATH=PRESERVED
CROSS_GENERATION_RUNTIME_FALLBACK=PRESERVED
OBJECT_BODY_REMAINS_NON_LEXICAL=PRESERVED
PERF013_INVARIANT_DELTA=NONE
~~~

## BUG008 / PERF025 consequence

B′ removes the semantic-wrapper/helper double CallTarget from **ordinary source
Closure invocation**.

Structured standard operations retain an intentional helper boundary.

Therefore carrier retirement is not an automatic consequence of PLAT042.
PERF025 must still run the already-planned deep-recursion/stack validation
against the final topology, including a structured-control stress shape, before
changing the BUG008-specific 64 MiB carrier.

~~~text
PERF025_C1C=UNBLOCKED_IF_B_PRIME_RATIFIED
PERF025_C2=EVIDENCE_GATED
BUG008=#681 REMAINS_CLOSED
~~~

## Recommendation

~~~text
PLAT042_STATUS=PROPOSED
CURRENT_CONFLICT=ESTABLISHED

CANDIDATE_A=SUPPORTED_BUT_HIGH_GENERATED_DUPLICATION
CANDIDATE_B_PRIME=SEMANTIC_SOURCE_PLUS_SINGLE_UNTAGGED_STRUCTURED_CPRIME_OWNER
CANDIDATE_C=CURRENT_API_NOT_AVAILABLE
CANDIDATE_D=SAFE_WRAPPER_FALLBACK

RECOMMENDED_CANDIDATE=B_PRIME

PLAT041_DELTA=REPLACE_COMPACT_HELPER_CLAUSE_WITH_STRUCTURED_DISPATCH_OWNER_BOUNDARY
PLAT026_INVARIANT_DELTA=NONE
PLAT014_INVARIANT_DELTA=NONE
PERF013_INVARIANT_DELTA=NONE

GENERATED_CODE_CONSEQUENCE=MODERATE_PARTITIONED_DUPLICATION_NOT_NEAR_FULL_DUPLICATION
HOT_PATH_CONSEQUENCE=ONE_EXTRA_HELPER_BOUNDARY_ONLY_FOR_STRUCTURED_PROTOCOL_INVOCATIONS; DIRECT_CALL_SPECIALIZATION_AVAILABLE
NATIVE_IMAGE_CONSEQUENCE=EXPECTED_MATERIALLY_LOWER_THAN_CANDIDATE_A; MUST_BE_MEASURED_AFTER_IMPLEMENTATION
PERF025_C_CONSEQUENCE=C1C_UNBLOCKED; C2_REMAINS_STACK_EVIDENCE_GATED

FUTURE_ENDURANCE=HIGH
SCALABILITY=HIGH
PROTOS_PHILOSOPHY=HIGH
NEW_SEMANTICS=NO
~~~

## Strongest argument against B′

B′ deliberately introduces a physical helper CallTarget for source-level
structured standard protocols that currently execute inline in the source
interpreter.

Even with a stable `DirectCallNode`, Truffle inlining remains heuristic and a
large structured dispatcher may not always inline. Interpreter-mode structured
calls also pay the boundary unconditionally.

This is accepted in the recommendation because the boundary corresponds to a
real structured-control ownership domain, already exists for nested structured
calls, and avoids carrying ~183 structured/helper operations in a second
semantic interpreter solely to preserve inline ownership.

The implementation must measure representative structured workloads before
claiming a performance win for those operations. A material regression is an
implementation acceptance failure, not permission to silently change semantics.

## Decision status

No candidate is ratified by this record.

Exact project-owner approval of B′ (or another exact candidate) is required
before C1c implementation may resume.

## Ratification

The project owner explicitly approved the exact recommended candidate on
2026-10-01:

~~~text
Apruebo B′ para PLAT042
~~~

~~~text
DECISION_APPROVAL_PROVENANCE=PASS
APPROVED_CANDIDATE=B_PRIME
DECISION_INVARIANT_CONSISTENCY=PASS
PERF025_C1C=IMPLEMENTATION_AUTHORIZED
~~~

The durable normative-for-project architecture statement is:

`docs/project/decisions/platform/PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md`

The investigation's numerical inventory remains revision-bound evidence at
`595d547b2e9714a185a3cfceadf74565229f43e7`.
