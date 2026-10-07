# PERF026-C1 — standard whileTrue literal-pair local-loop checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-C/#768
SLICE=PERF026-C1
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=92e48a0676da0b35886a186d56f0966dd5865fa3
PROTOS_REVISION=d70d4438170493b61c9da70781130e2a724ce4a1
PROTOS_VERSION=0.3.139-SNAPSHOT
COMMIT_SUBJECT=PERF026-C1: open-code standard whileTrue literal pair

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

C1 is exactly one commit ahead of the product base above. The base includes the
independent TEST006-D publication that landed after PERF026-B3; this record
therefore binds the while checkpoint to the actual product history.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026C1WhileInlineCallbackTest.java
~~~

The new Java regression carries the project APL-1.0 Part 5 notice. Modified Java
files retain their existing notices.

## Published C1 representation

C1 introduces a guarded local-loop alternative only for a send whose receiver
and sole argument are both the already-supported immediate zero-parameter B-prime
literal shape.

The compile-time pair candidate does not consult selector spelling. The local
loop is entered only after ordinary lookup/selection has prepared a standard
structured while call whose receiver and supplied body values are exactly the
staged literals.

Whole-pair admission is deliberately activation-free: it reads the already
prepared call and staged values but does not prepare either condition or body
activation. Therefore the existing validation and activation timing remains
authoritative.

Conceptually:

~~~text
ordinary receiver evaluation
  -> ordinary body-argument evaluation
  -> ordinary lookup / selection / prepared call
  -> exact standard while + staged literal pair admission
  -> PrepareStructuredWhileCall validation

  -> local Bytecode while in semantic source root
       prepare fresh condition child
       exact B-prime child proof
       inline condition region when admitted, physical child fallback otherwise
       strict canonical Boolean result authority

       if true:
         prepare fresh body child
         exact B-prime child proof
         inline body region when admitted, physical child fallback otherwise

  -> canonical null result

otherwise
  -> unchanged structured-dispatch while path
~~~

The semantic condition and body activations remain fresh and real even when
their physical callback RootCallTargets are removed on the admitted path.

## Structural evidence in the exact revision

The focused C1 regression contains explicit assertions/markers for:

~~~text
PERF026_C1_WHILE_HELPER_ROOT_REMOVED=YES
PERF026_C1_WHILE_CALLBACK_ROOTS_REMOVED=YES
PERF026_C1_WHILE_INLINE_ROOTTAGS=YES
PERF026_C1_WHILE_NORMAL_RESULT_NULL=YES
~~~

Thus the eligible literal pair executes the loop in the containing semantic
source root rather than entering the structured-dispatch helper root, and the
eligible condition/body callbacks no longer enter distinct callback roots.

## Semantic preservation evidence

The exact C1 regression also records:

~~~text
PERF026_C1_NLR_PRESERVED=YES
PERF026_C1_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_C1_FRESH_ACTIVATION_PER_ITERATION=YES

PERF026_C1_STRICT_BOOLEAN_CONDITION_PRESERVED=YES
PERF026_C1_BODY_ERROR_STOPS_LOOP=YES

PERF026_C1_CONDITION_SUSPENSION_PRESERVED=YES
PERF026_C1_BODY_SUSPENSION_PRESERVED=YES

PERF026_C1_WHILE_INLINE_SCOPE_PROJECTION=YES
~~~

The existing PreparedWhileCall remains the authority for fresh condition/body
preparation, strict canonical true/false validation, body-result disregard and
normal null completion.

## Fallback evidence

The focused regression records:

~~~text
PERF026_C1_DYNAMIC_CONDITION_FALLBACK=YES
PERF026_C1_DYNAMIC_BODY_FALLBACK=YES
PERF026_C1_NESTED_CLOSURE_LITERAL_FALLBACK=YES
PERF026_C1_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_C1_NON_WHILE_SELECTION_ORDINARY=YES
~~~

Every other standard while and every non-standard selection therefore retains
the existing structured-dispatch/physical-callback path.

## Human-reported validation

The human executor reported for this exact published slice:

~~~text
PERF026-C1: open-code standard whileTrue literal pair pushed
ALL_TESTS=GREEN
TEST_FAILURES=NONE_REPORTED
~~~

Accordingly:

~~~text
PRODUCT_PUBLICATION=YES
HUMAN_REPORTED_ALL_REQUESTED_TESTS=PASS
~~~

No narrower command transcript is fabricated beyond that report.

## PERF026-C completion consequence

PERF026-C owns the current Core standard loop family:

~~~text
Closure.whileTrue(body)
~~~

PERF026-A found no standard whileFalse protocol in current authority.

C1 implements the issue's selected eligible common representation, preserves
generic fallback and the required semantics/tooling boundary, and the human
executor reports all tests green.

Therefore:

~~~text
PERF026_C_IMPLEMENTATION=COMPLETE
PERF026_C_VALIDATION=PASS_HUMAN_REPORTED
PERF026_C_SEMANTIC_CHANGE=NO
PERF026_C_SPECIFICATION_CHANGE=NO
~~~

No additional C2 slice is required by the current issue scope.

## Next PERF026 family

PERF026-D / #769 is READY and owns synchronous standard each specialization:

~~~text
Array.each
Bytes.each
Environment.each
IdentityMap.each
Map.each
~~~

The natural first implementation slice is the shared one-argument indexed
snapshot family, Array.each + Bytes.each. Their existing structured
implementations have the same physical loop shape: validate callback, establish
an indexed snapshot, invoke one callback with one element/octet argument in
ascending order, advance, and return the receiver.

Unlike the completed zero-argument Boolean/while consumers, this slice must
extend the common B-prime inline callback machinery to one simple supplied
parameter while preserving ordinary prepared-call argument binding and exact
fallback.

## Cross references

- guillermomolina/protos#768 — PERF026-C.
- guillermomolina/protos#769 — PERF026-D.
- guillermomolina/protos#767 — PERF026-B completed.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/evidence/PERF026/PERF026_B3_IFTRUEIFFALSE_INLINE_CALLBACK.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
