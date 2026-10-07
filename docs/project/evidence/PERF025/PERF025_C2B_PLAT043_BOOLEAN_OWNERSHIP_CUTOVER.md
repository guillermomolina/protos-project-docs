# PERF025-C2B — PLAT043 standard Boolean semantic-interpreter ownership cutover

Date: 2026-10-01

## Publication identity

~~~text
WORK_ITEM=PERF025/#758
DECISION=PLAT043/#763
PRODUCT_REPOSITORY=guillermomolina/protos

PLAT043_DECISION_BASELINE=0a5115caddba8ebb7bb4275ce32441ab90938d3d
SLICE_BASE_REVISION=d8dcc95d34088e942e737b98c7ad42a81a977293
PROTOS_REVISION=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
PROTOS_VERSION=0.3.134-SNAPSHOT
COMMIT_SUBJECT=PERF025-C2B: PLAT043 standard Boolean semantic-interpreter ownership

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The slice base differs from the original PLAT043 product baseline because one
intervening published product commit existed before C2B. This evidence attributes
only the paths changed by the exact C2B commit and does not fold unrelated
earlier tooling changes into C2B.

## Exact published delta

The C2B commit changes exactly these product paths:

~~~text
CHANGELOG.md
pom.xml
protos/tests/conformance/boolean/callback-dispatch-and-control.protos
protos/tests/conformance/manifest.tsv
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025C1cSemanticSourceTopologyTest.java
~~~

Version authority advances to:

~~~text
0.3.134-SNAPSHOT
~~~

## Implemented ownership

The published lowerer now classifies the already-prepared invocation in this
order:

~~~text
IsStructuredBooleanCall
    -> local semantic-root Boolean orchestration

otherwise RequiresStructuredDispatch
    -> PLAT042 untagged structured/C-prime root

otherwise
    -> ordinary prepared invocation
~~~

This preserves the PLAT043 rule that selector spelling is not dispatch
authority. The local Boolean path consumes the already-selected prepared
capability.

The local family is exactly:

~~~text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
~~~

No non-Boolean structured family is moved.

## Single Boolean semantic authority

`ProtosSemanticBytecodeRootNode` adds the operation declarations needed by the
semantic interpreter, but those operations delegate to the existing
`ProtosBytecodeRootNode` implementations, including
`PreparedBooleanCall`.

Therefore:

~~~text
SECOND_BOOLEAN_SEMANTICS=NO
PREPARED_BOOLEAN_STATE_MACHINE_REUSED=YES
~~~

The lowerer preserves outer prepared-call completion through Bytecode
`TryFinally`, and the selected callback preserves the scoped child behavior:

~~~text
child.requiresStructuredDispatch()
    -> nested PLAT042 helper

ordinary child
    -> ordinary prepared invocation in semantic path
~~~

Continuation composition remains the existing PLAT014 mechanism.

## Topology evidence in source

The updated topology test covers all five Boolean kinds and records:

~~~text
PERF025_C2B_BOOLEAN_HELPER_CALLTARGET_REMOVED=YES
~~~

For a reached standard Boolean callback, the captured guest stack contains two
semantic roots and no intervening `ProtosBytecodeRootNode` structured helper.

The same test separately exercises `Closure.while` and retains the PLAT042
topology:

~~~text
NON_BOOLEAN_STRUCTURED_HELPER=YES
STRUCTURED_HELPER_ROOT_TAG=NO
~~~

Thus C2B narrows PLAT042 only where PLAT043 authorized.

## Dispatch/control conformance coverage

The new conformance source:

`protos/tests/conformance/boolean/callback-dispatch-and-control.protos`

covers the C2B-sensitive boundaries:

- selected callback may be an ordinarily invokable non-Closure object;
- selected non-invokable `call` behavior signals Error;
- non-local return propagates through all five Boolean callback families;
- Error from a selected callback propagates;
- a custom same-name `ifTrue` / `and` remains ordinary dispatch; and
- a copied standard `ifTrue` keeps its non-Boolean receiver behavior.

The manifest includes this conformance source in the native suite.

## Changelog authority

The 0.3.134 entry states that:

- standard Boolean calls are classified after ordinary lookup from the already
  selected behavior;
- reached Boolean callbacks no longer add the untagged helper CallTarget;
- the semantic operations delegate to the existing `PreparedBooleanCall`
  state machine;
- the outer call is completed exactly once through Bytecode `TryFinally`;
- every other structured family retains the untagged helper; and
- the guest carrier and fixed stack are unchanged.

## Validation evidence boundary

The publication itself is confirmed by the exact public commit.

At this checkpoint there are no GitHub commit statuses or workflow runs attached
to `57d8cf4e...`, and the active interaction did not provide explicit
human-reported focused/full/native validation results.

Accordingly this durable record does **not** infer test success merely from the
published code.

~~~text
IMPLEMENTATION_PUBLICATION=CONFIRMED
FOCUSED_VALIDATION=NOT_RECORDED_HERE
INTEGRATED_VALIDATION=NOT_RECORDED_HERE
NATIVE_VALIDATION=NOT_RECORDED_HERE
~~~

## Mandatory post-C2B stack gate

PLAT043 explicitly required an unchanged-workload measurement after the Boolean
owner cutover before any carrier decision.

The retained target is the existing 10,000-deep recursive shape. Before C2B its
physical topology was approximately:

~~~text
R_i -> S_i -> B_i -> R_i+1
~~~

where `S_i` was the untagged Boolean helper.

C2B statically establishes the intended new topology:

~~~text
R_i -> B_i -> R_i+1
~~~

but the product publication does not retain the dynamic stack-depth result for
that unchanged workload.

The old approximately-32-MiB observation belongs to the pre-C2B
`R -> S -> B` topology and cannot be reused as the post-C2B result.

Therefore:

~~~text
PERF025_C2B_CODE=COMPLETE
PERF025_C2B_PUBLICATION=COMPLETE
PERF025_C2B_STACK_GATE=PENDING

RETAINED_RECURSION_DEPTH=10000
POST_C2B_STACK_RESULT=NOT_RECORDED

CARRIER_RETIREMENT_AUTHORIZED=NO
CARRIER_STACK_REDUCTION_AUTHORIZED=NO
CARRIER_CHANGED=NO

BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

The next project action must obtain this missing stack evidence before assigning
a carrier-removal implementation slice.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#763` — PLAT043.
- `guillermomolina/protos#760` — PLAT042.
- `guillermomolina/protos#681` — historical BUG008.
- `docs/project/decisions/platform/PLAT043_STANDARD_BOOLEAN_CONTROL_INTERPRETER_OWNERSHIP_BOUNDARY.md`.
- `docs/project/evidence/PLAT043/PLAT043_STANDARD_BOOLEAN_CONTROL_OWNERSHIP_INVESTIGATION.md`.
- `docs/project/evidence/PERF025/PERF025_C1C_PLAT042_B_PRIME_CUTOVER.md`.
