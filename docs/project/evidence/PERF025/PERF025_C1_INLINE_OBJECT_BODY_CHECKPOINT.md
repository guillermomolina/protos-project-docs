# PERF025-C1 — current-activation seam and inline Object-body checkpoint

Date: 2026-10-01

## Publication identity

~~~text
WORK_ITEM=PERF025/#758
PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=595d547b2e9714a185a3cfceadf74565229f43e7
PROTOS_VERSION=0.3.132-SNAPSHOT
COMMIT_SUBJECT=PERF025-C1 checkpoint: inline Object-body execution (C1a+C1b)

PERF025_C1A=COMPLETE
PERF025_C1B=COMPLETE
PERF025_C1C=BLOCKED
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

The product revision is published on `guillermomolina/protos/main`.

## C1a result

C1a established one compile-time lowering authority for the current
`ProtosActivation` in `CanonicalToBytecodeLowerer`.

Normal root lowering continues to source the activation from frame argument 0.
The seam exists so an inline Object-construction region can select its
construction activation without scattering special cases across canonical
lowering.

C1a did not change RootTag topology or execution semantics.

## C1b result

C1b removed the Object-construction body as a physical Bytecode helper root.

The published implementation:

- lowers Object construction bodies inline in the enclosing Bytecode root;
- keeps the construction `ProtosActivation` in resumable Bytecode-local state;
- executes body operations through the C1a current-activation authority;
- preserves `ProtosActivation.forObjectConstruction(...)` as the construction
  activation authority;
- keeps Object bodies non-lexical;
- preserves nested Closure capture of the enclosing genuine lexical chain;
- removes `ProtosObjectBodyTargetCell` and the old Object-body
  Prepare/Enter/Resume/Finish helper-root machinery;
- updates PERF013 grouping tests for the approved C′ physical topology; and
- adds focused Object-body control/suspension/capture coverage plus the
  `construction-body-control.protos` conformance case.

The implementation handoff reported the focused C1b gate PASS before publication.

## Published product files

The product commit changes:

~~~text
CHANGELOG.md
build/native/generate-init-args.sh
pom.xml
protos/tests/conformance/manifest.tsv
protos/tests/conformance/object/construction-body-control.protos
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosObjectBodyTargetCell.java (removed)
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceA3ObjectBodyHelperSharedGroupingTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceASharedLexicalRootGroupingTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf025C1ObjectBodyInlineControlTest.java
~~~

## PLAT041 invariant status after C1b

~~~text
OBJECT_BODY_HELPER_ROOT=REMOVED
OBJECT_BODY_INLINE=YES
OBJECT_BODY_LEXICAL_SCOPE=NO
CONSTRUCTION_ACTIVATION=PRESERVED
PERF013_OWNER_CLOSURE_SAME_GENERATION=PRESERVED
PLAT014_SUSPENSION_MODEL=PRESERVED
SEMANTIC_WRAPPER=STILL_PRESENT
SOURCE_AUTOMATIC_ROOT_TAG_CUTOVER=NOT_DONE
BUG008_CARRIER=STILL_PRESENT
~~~

Thus C1a+C1b are a coherent published partial implementation of PLAT041 C′.
They do not claim C1c or carrier retirement.

## C1c stop gate

C1c stopped before editing after checking the actual operation closure required
by the proposed untagged infrastructure interpreter.

The direct six-builder inventory materially understated the requirement because
`ProtosTaskCPrimeEntryExecution.createPlan(...)` invokes:

~~~text
CanonicalToBytecodeLowerer.emitPreparedInvocationForRuntime(...)
~~~

That entry expands through the prepared/structured dispatch lowering family.
Nested structured dispatch also re-enters the same Task C-prime plan through
`EnterNestedStructuredDispatch`.

Independent static counting at exactly
`595d547b2e9714a185a3cfceadf74565229f43e7` establishes:

| Inventory | Unique builder operation names |
| --- | ---: |
| Canonical source lowerer | 203 |
| Six C-prime builders, direct only | 53 |
| Prepared/structured dispatch transitive family | 144 |
| Actual helper union including transitive family | 183 |
| `@Operation` annotations in `ProtosBytecodeRootNode` | 223 |

The helper union is 90.1% of the source-lowerer operation-name surface.

~~~text
PLAT041_COMPACT_HELPER_PREMISE=FALSIFIED
STOP_GATE_10=PASS
PRODUCT_C1C_CHANGES=NONE
~~~

## Additional partition evidence

The large structured-dispatch body itself emits 137 unique operation names.

If that large body is outlined while the source interpreter retains the
existing scoped/ordinary prepared-call composition, static lowering requires:

~~~text
CURRENT_SOURCE_LOWERER_OPS=203
LARGE_STRUCTURED_DISPATCH_OPS=137
SOURCE_OPS_AFTER_OUTLINING_LARGE_DISPATCH=74
STRUCTURED_ONLY_OPS=129
OVERLAP_OPS=8
~~~

This is important for PLAT042: the choice is not necessarily between one 203-op
interpreter and two near-203-op interpreters. A semantic source interpreter can
potentially retain a much smaller operation surface if structured dispatch has
one untagged owner.

## Existing structured-dispatch composition evidence

Current `emitScopedPreparedInvocation(...)` already implements the relevant
cross-root continuation pattern:

~~~text
prepared.requiresStructuredDispatch()
    -> EnterNestedStructuredDispatch
    -> Context-local ProtosTaskCPrimeEntryExecution target
    -> if ContinuationResult: Yield / ResumeContinuation
otherwise
    -> ordinary prepared invocation
~~~

The structured dispatcher itself owns loops such as standard `while`,
`each`, matching, `ensure`, Error handling and import.

Therefore an out-of-line structured `while` boundary would occur once when
entering the structured `while` invocation. The loop iterations execute
inside that dispatcher root; ordinary condition/body Closure calls do not
re-enter the structured dispatcher unless they themselves require structured
dispatch.

This behavior is already present for nested structured calls; PLAT042 decides
whether source-level structured calls should use the same ownership boundary.

## Decision gate

The remaining topology is not implementation-local because it trades generated
interpreter footprint against a structured-call CallTarget boundary.

~~~text
NEXT_DECISION=PLAT042/#760
PERF025_C=BLOCKED_PENDING_PLAT042
PERF025_C1C=BLOCKED_PENDING_PLAT042
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

PLAT042 must not rewrite C1a+C1b history. It decides only the remaining
structured-dispatch/interpreter ownership boundary and the consequent route to
the C1c RootTag cutover.

## Cross references

- PERF025: `guillermomolina/protos#758`
- PLAT041: `guillermomolina/protos#759`
- PLAT042: `guillermomolina/protos#760`
- BUG008: `guillermomolina/protos#681`
- PLAT041 durable decision:
  `docs/project/decisions/platform/PLAT041_BYTECODE_ROOT_TAG_MATERIALIZED_LOCAL_GROUPING_BOUNDARY.md`
