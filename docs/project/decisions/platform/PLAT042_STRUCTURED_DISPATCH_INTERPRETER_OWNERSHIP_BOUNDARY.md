# PLAT042 — Structured-dispatch interpreter ownership boundary

Status: **RATIFIED**

Selected architecture: **Candidate B′ — tagged semantic source interpreter + one untagged structured-dispatch/C-prime owner + structured-only helper CallTarget boundary**.

Approval: explicit project-owner approval on 2026-10-01:

~~~text
Apruebo B′ para PLAT042
~~~

Decision Issue: `guillermomolina/protos#760`

Parent workstream: `PERF025 / guillermomolina/protos#758`

Product baseline:

~~~text
PROTOS_REVISION=595d547b2e9714a185a3cfceadf74565229f43e7
PROTOS_VERSION=0.3.132-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative JVM/Truffle implementation-architecture decision.
Observable Protos semantics remain owned by the normative specification.

## Decision

Protos adopts Candidate B′ for the remaining PLAT041/PERF025-C1c topology.

The durable production architecture is:

~~~text
tagged semantic source Bytecode interpreter
    top-level/public source semantic roots
    module semantic roots
    Closure semantic roots
    Object-construction bodies inline as non-root regions

    ordinary source execution
    ordinary prepared Closure invocation

    structured prepared invocation
        -> one intentional helper CallTarget boundary
        -> untagged structured-dispatch/C-prime owner

untagged structured-dispatch/C-prime Bytecode interpreter
    standard structured protocols
        while
        each
        ensure
        Error.handle
        match / case
        import
        structured collection callbacks
        other prepared structured forms

    Task C-prime entry
    TextReader/TextWriter C-prime roots
    BufferedByteReader/BufferedByteWriter C-prime roots
    I/O release C-prime roots
    nested structured dispatch
~~~

No third generated interpreter is selected.

The current `ProtosSemanticBytecodeRootNode` role SHALL evolve from a tiny
semantic RootTag wrapper into the real tagged semantic source interpreter.

The current `ProtosBytecodeRootNode` role MAY remain the untagged
structured/C-prime substrate. Exact class names are implementation-local.

## Trigger and correction to PLAT041

PLAT041 selected Candidate C′ and correctly established:

- one current-activation lowering authority;
- inline Object-construction body execution;
- Object-body helper root removal;
- truthful semantic/helper RootTag separation;
- preservation of PERF013 same-generation captured-local access; and
- no semantic change.

PERF025-C1a+C1b implemented those parts and were published at
`595d547b2e9714a185a3cfceadf74565229f43e7`.

C1c then correctly stopped before implementation because PLAT041's phrase
"compact untagged infrastructure interpreter" rested on an incomplete operation
inventory.

Revision-bound inspection established:

~~~text
CURRENT_SOURCE_LOWERER_OPERATION_NAMES=203
SIX_CPRIME_DIRECT_OPERATION_NAMES=53
PREPARED_STRUCTURED_TRANSITIVE_OPERATION_NAMES=144
ACTUAL_HELPER_UNION_OPERATION_NAMES=183
HELPER_UNION_VS_SOURCE_LOWERER=90.1%
PROTOS_BYTECODE_ROOT_NODE_OPERATION_ANNOTATIONS=223
~~~

The six C-prime builder files are not independent small helpers:
`ProtosTaskCPrimeEntryExecution` invokes
`CanonicalToBytecodeLowerer.emitPreparedInvocationForRuntime(...)`, which
transitively owns the large structured-dispatch family.

Therefore PLAT042 explicitly amends only the PLAT041 C1c helper-ownership clause.

~~~text
PLAT041_C1A=PRESERVED
PLAT041_C1B=PRESERVED

OLD_PLAT041_C1C=
  semantic source interpreter
  + compact untagged infrastructure interpreter

PLAT042_B_PRIME_C1C=
  tagged semantic source interpreter
  + single untagged structured-dispatch/C-prime owner
  + structured-only helper CallTarget boundary
~~~

PLAT041 remains ratified historical authority for the preserved invariants.
PLAT042 is the later authority for C1c structured-dispatch ownership.

## Why B′ is materially different from a near-full duplicate

The large structured-dispatch lowering body uses 137 unique operation names.

When that body is outlined while the semantic source interpreter retains
ordinary/scoped prepared-call composition, the measured operation-name surfaces
at the ratification baseline are approximately:

~~~text
SEMANTIC_SOURCE_OPERATION_NAMES_AFTER_OUTLINING=74
HELPER_UNION_OPERATION_NAMES=183
OVERLAP_BETWEEN_SURFACES=18
COMBINED_UNIQUE_OPERATION_NAMES=239
GENERATED_OPERATION_INCLUSIONS_IF_EXACT_PARTITION=257
~~~

These are operation-name counts, not byte-size claims.

They establish that Protos does not need a 203-operation tagged source
interpreter plus a 183-operation untagged near-duplicate.

## Migration authority

C1c does not need to perform an exact operation partition in one patch.

The approved low-risk migration is:

1. evolve the semantic-root interpreter into the real tagged semantic source
   interpreter containing the source operations required after structured
   dispatch is outlined;
2. retain the current untagged interpreter as the structured/C-prime substrate;
3. initially leave helper-unused/source-only operations in the untagged
   interpreter if removing them would unnecessarily enlarge the migration;
4. remove the old semantic-shell-to-full-helper universal source wrapper;
5. route only prepared invocations classified as structured to the untagged
   structured owner; and
6. treat later exact removal of unused untagged operations as cleanup that does
   not require reopening PLAT042.

Approximate initial generated operation inclusion may therefore be:

~~~text
CURRENT_UNTAGGED_OPERATION_ANNOTATIONS~=223
SEMANTIC_SOURCE_LOWERING_SURFACE~=74
INITIAL_GENERATED_OPERATION_INCLUSIONS~=297
~~~

This is acceptable as a migration envelope and is not a requirement to keep all
223 untagged operations forever.

## Structured-call boundary

The helper CallTarget boundary is intentional and applies only when a prepared
invocation requires structured dispatch.

The semantic interpreter SHALL distinguish:

~~~text
ordinary prepared call
    -> execute through ordinary semantic/Closure composition

structured prepared call
    -> enter Context-owned untagged structured dispatcher
~~~

The current code already contains the relevant continuation shape through
`emitScopedPreparedInvocation(...)` and `EnterNestedStructuredDispatch`.

This existing protocol transports:

- `ContinuationResult`;
- yield/resume values;
- Protos control transfers;
- Truffle guest exceptions;
- module-initialization failures; and
- runtime-failure mapping.

PLAT042 changes the ownership/frequency of that existing boundary. It does not
define a new continuation protocol.

## Boundary frequency

For a standard structured `while`, the helper CallTarget is entered once per
structured `while` invocation.

The Bytecode `While` loop and its dispatcher state remain inside the
structured-dispatch root.

Condition/body Closures still execute as ordinary Closure calls. They enter the
structured dispatcher recursively only if the child invocation itself requires
structured dispatch.

The same ownership principle applies to `each`, matching, `ensure`, Error
handling and the remaining structured protocols.

Therefore the approved architecture does **not** require one helper-root crossing
per loop iteration merely because a loop is structured.

## Stable-call optimization

The Task C-prime plan is Context-local and lazily cached by
`ProtosLanguageContext`.

The structured-dispatch entry SHOULD use a stable-target call specialization
analogous to ordinary Closure dispatch:

~~~text
stable target
    -> cached DirectCallNode

polymorphic / cache-exceeded target
    -> IndirectCallNode fallback
~~~

This preserves multi-Context correctness while making the stable helper
boundary visible to Truffle as an optimizable call site.

The exact cache limit and class/method factoring are implementation details.

A raw `CallTarget.call(...)` is not selected as a required architecture when a
stable DirectCallNode specialization can express the ownership more truthfully.

## RootTag/tooling authority

PLAT026 remains authoritative.

The semantic source interpreter SHALL use automatic `StandardTags.RootTag`
for truthful semantic top-level/module/Closure roots.

The structured/C-prime interpreter SHALL remain untagged.

Structured helper roots do not become semantic function roots merely because
source-level structured protocols enter them.

The following remain unchanged:

- Object bodies remain non-roots after PERF025-C1b;
- Object bodies remain non-lexical;
- `RootBodyTag` remains deferred;
- StatementTag, ExpressionTag and CallTag retain their approved semantics;
- no manual RootTag emulation is introduced;
- no global mutable root-classification registry is introduced; and
- helper roots remain absent from debugger semantic-root identity.

## PLAT014 continuation/control compatibility

PLAT014 remains authoritative.

B′ preserves the existing C-prime continuation composition model.

~~~text
CONTINUATION_RESULT_COMPOSITION=PRESERVED
YIELD_RESUME=PRESERVED
CONTROL_TRANSFER_BRIDGE=PRESERVED
NO_REPLAY=PRESERVED
NEW_CONTINUATION_KIND=NO
~~~

The structured helper boundary is a physical ownership change only.

Error selection, ensure unwinding, cancellation, non-local return and module
initialization behavior must remain semantically identical.

## PERF013 / PLAT036 compatibility

The C1a+C1b result remains authoritative:

~~~text
OBJECT_BODY_INLINE=YES
OBJECT_BODY_LEXICAL_CONTEXT=NO

LEXICAL_OWNER_AND_NESTED_CLOSURE_SAME_GENERATION=PRESERVED
MATERIALIZED_LOCAL_FAST_PATH=PRESERVED
CROSS_GENERATION_RUNTIME_FALLBACK=PRESERVED
CAPTURE_BY_REFERENCE=PRESERVED
~~~

The structured-dispatch interpreter is not a lexical owner and must not become
a second captured-state authority.

## Operation sharing

`OperationProxy` and shared specialization helpers MAY be used to avoid
duplicating Java operation semantics.

They do not merge independently generated Bytecode interpreters and are not an
excuse to recreate Candidate A's near-full duplicate.

`GenerateBytecodeTestVariants` remains test-only and is not approved as
production architecture.

## Rejected alternatives

### Candidate A — near-full second untagged interpreter

Rejected as the default architecture.

It can preserve semantics but would carry a near-full second generated
interpreter merely to solve root classification. The measured helper union is
183 operation names versus 203 in the current source lowerer.

### Candidate C — framework-level shared generated dispatcher with no root boundary

Not currently available.

No supported Truffle 25.4 Bytecode DSL mechanism was found that simultaneously
provides:

- one shared generated instruction machine across differently RootTag-configured
  root populations;
- one shared root group;
- automatic per-root RootTag classification; and
- no CallTarget boundary.

A future supported per-root automatic classification mechanism remains a valid
simplification path.

### Candidate D — retain the current semantic wrapper

Retained as an implementation fallback only if B′ produces a material
unacceptable regression that cannot be corrected within the ratified
architecture.

It is not the selected target because it leaves the universal extra source
CallTarget and therefore does not complete PERF025-C1c's primary objective.

Falling back to D after material contrary evidence requires recording that
evidence before abandoning B′. It is not an implementation-local silent choice.

## GITHUB021 invariant/delta result

~~~text
PLAT041_DELTA=
  COMPACT_HELPER_CLAUSE_REPLACED_BY_STRUCTURED_DISPATCH_OWNER_BOUNDARY

PLAT026_INVARIANT_DELTA=NONE
PLAT014_INVARIANT_DELTA=NONE
PERF013_INVARIANT_DELTA=NONE

NEW_PROTOS_SEMANTICS=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

The project owner explicitly approved the PLAT041 delta as part of selecting B′.

## PERF025 and BUG008 consequence

B′ releases PERF025-C1c.

C1c must remove the universal semantic-wrapper/helper double CallTarget from
ordinary source Closure invocation.

Structured standard protocols intentionally retain one helper boundary.

Therefore carrier retirement is **not** automatically authorized by PLAT042.

After C1c, PERF025 must validate at least:

- deep ordinary source Closure recursion; and
- a representative deep/recursive structured-control shape

before changing the BUG008-specific 64 MiB carrier.

~~~text
PERF025_C1C=IMPLEMENTATION_AUTHORIZED
PERF025_C2=STACK_EVIDENCE_GATED
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

## Performance acceptance

The structured helper boundary is new for top-level/source-level structured
invocations that previously executed the structured dispatcher inline in the
full source interpreter.

C1c implementation acceptance therefore requires a **bounded** representative
before/after check for structured paths.

This is not an open-ended benchmark campaign.

At minimum it should distinguish:

- ordinary non-structured Closure invocation, where the universal wrapper is
  removed; and
- representative structured protocols such as `while` and/or `each`, where
  the intentional helper boundary is introduced.

A material structured-path regression is evidence requiring correction or a
return to PLAT042; it is not permission to alter observable semantics.

## Scalability and future endurance

B′ scales root identity according to real semantic ownership:

- semantic source roots remain one tagged source interpreter family;
- structured orchestration remains one untagged implementation family;
- no registry grows with source roots, Objects, Tasks, Actors, Processes or
  Contexts;
- each Context owns one cached structured-entry plan rather than one generated
  interpreter per call;
- structured recursion uses ordinary Truffle call/continuation composition.

Future Truffle per-root classification can collapse physical interpreter
factoring without changing these semantic ownership rules.

## Strongest argument against B′

B′ introduces an intentional helper CallTarget on source-level structured
protocol invocations that currently execute their structured dispatcher inline.

A cached DirectCallNode makes the stable boundary optimizable but does not
guarantee inlining. Interpreter-mode structured calls pay the boundary.

This cost is accepted because the boundary corresponds to a real
structured-control ownership domain, is already used for nested structured
dispatch, and avoids making a near-full second generated interpreter permanent
architecture.

Implementation measurement remains mandatory.

## Approval record

The project owner explicitly approved Candidate B′ on 2026-10-01 after the
PLAT042 decision packet disclosed the PLAT041 C1c delta and compared Candidates
A, B′, C and D.

Exact approval:

~~~text
Apruebo B′ para PLAT042
~~~

No language/specification change is authorized.

## References

- `guillermomolina/protos#760` — PLAT042 decision Issue.
- `guillermomolina/protos#758` — PERF025 parent/consumer.
- `guillermomolina/protos#759` — PLAT041, preserved except for the amended C1c
  compact-helper clause.
- `guillermomolina/protos#681` — historical BUG008; remains closed.
- `docs/project/evidence/PLAT042/PLAT042_STRUCTURED_DISPATCH_OWNERSHIP_INVESTIGATION.md`.
- `docs/project/evidence/PERF025/PERF025_C1_INLINE_OBJECT_BODY_CHECKPOINT.md`.

## Published implementation evidence

PLAT042 Candidate B′ was implemented and published in
`guillermomolina/protos` at:

~~~text
PROTOS_REVISION=0a5115caddba8ebb7bb4275ce32441ab90938d3d
PROTOS_VERSION=0.3.133-SNAPSHOT
COMMIT_SUBJECT=PERF025-C: PLAT042 B′ semantic/structured interpreter cutover
~~~

The published implementation establishes:

~~~text
SEMANTIC_SOURCE_INTERPRETER=YES
SEMANTIC_AUTOMATIC_ROOT_TAG=YES

UNIVERSAL_SEMANTIC_WRAPPER=REMOVED
ORDINARY_SOURCE_CLOSURE_CALLTARGET_COUNT=1

STRUCTURED_CPRIME_OWNER=UNTAGGED
STRUCTURED_ONLY_HELPER_BOUNDARY=YES
STRUCTURED_DIRECT_CALL_SPECIALIZATION=YES

OBJECT_BODY_INLINE=YES
OBJECT_BODY_SEMANTIC_ROOT=NO
OBJECT_BODY_LEXICAL_CONTEXT=NO

PERF013_MATERIALIZED_FAST_PATH=PRESERVED
PLAT014_CONTINUATION_MODEL=PRESERVED
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The source/structured split was implemented by evolving
`ProtosSemanticBytecodeRootNode` into the real semantic source interpreter and
moving the large structured dispatcher into
`ProtosStructuredDispatchLowerer`, which continues to target the untagged
`ProtosBytecodeRootNode` interpreter.

The old semantic wrapper topology is gone for ordinary source roots.

The implementation also retains source-root operation semantics by delegating
the semantic interpreter's source-surface operation wrappers to the existing
single implementations in `ProtosBytecodeRootNode`, rather than duplicating
semantic logic.

Bounded JVM before/after evidence recorded in the product CHANGELOG against
`595d547b2e9714a185a3cfceadf74565229f43e7`:

~~~text
ORDINARY_WORKLOAD_DELTA=-46%
STRUCTURED_BOUNDARY_WORKLOAD_DELTA=-36%
RETAINED_CLOSURE_CALL_WORKLOAD_DELTA=-43%
~~~

The maintainer-reported Protos test-tool corpus at publication was:

~~~text
1271 passed, 0 failed
Protos tests total time: 34 s
~~~

This outcome confirms that PLAT042 B′ was not merely a tooling/topology
correction; it removed material runtime overhead.

### Residual BUG008 carrier evidence

Carrier retirement did not accompany this implementation.

The published product still has:

~~~text
GUEST_CALL_STACK_SIZE_BYTES=64 MiB
DEDICATED_GUEST_CARRIER=STILL_PRESENT
~~~

The first post-C1c stack gate established a different remaining source of stack
pressure: the retained 10,000-deep benchmark recursion shape invokes a
structured `ifTrue` at every recursion level. Under B′ each such structured
invocation owns the intentional untagged dispatcher boundary plus its callback
call.

Observed evidence recorded in the 0.3.133 CHANGELOG states that those retained
drivers still require a large fixed stack, approximately 32 MiB in interpreter
execution.

This does **not** invalidate PLAT042. It means the old universal source-wrapper
stack amplification was removed, while one structured-recursion amplification
remains intentionally visible for PERF025-C2 classification.

~~~text
PLAT042_IMPLEMENTATION=COMPLETE
PERF025_C1C=COMPLETE
PERF025_C2=CARRIER_RETIREMENT_PENDING
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

Detailed implementation evidence is retained under
`docs/project/evidence/PERF025/PERF025_C1C_PLAT042_B_PRIME_CUTOVER.md`.

