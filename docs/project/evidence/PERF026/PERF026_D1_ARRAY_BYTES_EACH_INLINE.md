# PERF026-D1 — Array.each / Bytes.each one-parameter inline-callback checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-D/#769
SLICE=PERF026-D1
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=d70d4438170493b61c9da70781130e2a724ce4a1
PROTOS_REVISION=0f87a7b2528d73f66a7ae1d3af90fdc3ffad1089
PROTOS_VERSION=0.3.140-SNAPSHOT
COMMIT_SUBJECT=PERF026-D1: inline Array and Bytes each literal callbacks

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

D1 is exactly one commit ahead of the C1 base revision above.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026D1IndexedEachInlineCallbackTest.java
~~~

## Published D1 architecture

D1 extends the proven PLAT044 B-prime literal callback representation only to
the smallest parameterized shape required by standard indexed each:

~~~text
immediate Closure literal
exactly one ordinary required positional parameter
no default
not rest
no nested Closure literal in the body
existing B-prime activation / ReturnHome / plan conditions satisfied
~~~

The one-parameter candidate path is deliberately separate from the existing
zero-parameter Boolean/while candidate predicate, so Boolean and while sites do
not gain parameterized inline regions.

After ordinary evaluation and lookup/selection establish exact standard
Array.each or Bytes.each behavior with exactly the staged literal callback, the
iteration is sequenced locally in the containing semantic source root.

The existing prepared each objects remain semantic authority for validation and
snapshot construction.

Conceptually:

~~~text
ordinary receiver / callback evaluation
  -> ordinary lookup / exact standard each selection
  -> whole-call literal/value admission
  -> PrepareStructuredIndexedEachCall
       validates receiver
       validates callback callability
       establishes one shallow ascending indexed snapshot

  -> local Bytecode iteration
       PrepareStructuredIndexedEachElementCall
         -> exact snapshot element/octet supplied
         -> fresh semantic callback activation

       exact B-prime child proof
         -> inline callback body + RootTag + projected scope
         OR
         -> exact physical child invocation fallback

       AdvanceStructuredIndexedEach only after normal child completion

  -> FinishStructuredIndexedEach
       -> exact original receiver
~~~

Ordinary parameter binding remains authoritative: the inline callback formal is
bound from the prepared child activation through the existing Closure parameter
binding machinery. D1 does not substitute snapshot values directly into body
locals.

## Common indexed-each representation

D1 introduces a common prepared view for the isomorphic standard Array.each and
Bytes.each loops while retaining each collection's own receiver validation and
snapshot authority.

The shared observable contract is:

~~~text
snapshot established once
ascending index order
one supplied callback argument
fresh callback activation per visit
advance only after callback normal completion
callback result ignored
receiver identity returned
~~~

## Structural evidence in the exact revision

The focused D1 test records:

~~~text
PERF026_D1_ARRAY_EACH_LOCAL_LOOP=YES
PERF026_D1_ARRAY_HELPER_ROOT_REMOVED=YES
PERF026_D1_ARRAY_CALLBACK_ROOT_REMOVED=YES
PERF026_D1_ARRAY_EXACT_ELEMENT_ARGUMENT=YES

PERF026_D1_BYTES_EACH_LOCAL_LOOP=YES
PERF026_D1_BYTES_HELPER_ROOT_REMOVED=YES
PERF026_D1_BYTES_CALLBACK_ROOT_REMOVED=YES
PERF026_D1_BYTES_SEMANTIC_INTEGER_OCTET=YES
~~~

## Collection semantic preservation

Array:

~~~text
PERF026_D1_ARRAY_SNAPSHOT_PRESERVED=YES
PERF026_D1_ARRAY_ASCENDING_ORDER_PRESERVED=YES
PERF026_D1_ARRAY_CALLBACK_RESULT_IGNORED=YES
PERF026_D1_ARRAY_RECEIVER_RESULT_IDENTITY=YES
~~~

Bytes:

~~~text
PERF026_D1_BYTES_SNAPSHOT_PRESERVED=YES
PERF026_D1_BYTES_ASCENDING_ORDER_PRESERVED=YES
PERF026_D1_BYTES_CALLBACK_RESULT_IGNORED=YES
PERF026_D1_BYTES_RECEIVER_RESULT_IDENTITY=YES
~~~

Common callback semantics:

~~~text
PERF026_D1_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_D1_FRESH_ACTIVATION_PER_CALLBACK=YES
PERF026_D1_NLR_PRESERVED=YES
PERF026_D1_ERROR_PROPAGATION_PRESERVED=YES
PERF026_D1_SUSPENSION_RESUMPTION_PRESERVED=YES
PERF026_D1_COMPLETED_PREFIX_REPLAY=NO
PERF026_D1_INLINE_SCOPE_PROJECTION=YES
~~~

The published changelog also records that the inline region binds the formal
through ordinary Closure parameter binding from the prepared activation.

## Fallback preservation

The focused D1 regression records:

~~~text
PERF026_D1_DYNAMIC_CALLBACK_FALLBACK=YES
PERF026_D1_NESTED_CLOSURE_LITERAL_FALLBACK=YES
PERF026_D1_DEFAULT_REST_PARAMETER_FALLBACK=YES
PERF026_D1_UNSUPPORTED_ARITY_ORDINARY_ERROR=YES
PERF026_D1_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_D1_NONCLOSURE_INVOKABLE_FALLBACK=YES
PERF026_D1_CUSTOM_EACH_FALLBACK=YES
~~~

Thus unsupported callback shapes retain ordinary callability/arity behavior and
all non-admitted each implementations retain the existing structured/physical
path.

## Human-reported validation

The human executor reported for this exact published slice:

~~~text
PERF026-D1: inline Array and Bytes each literal callbacks pushed
ALL_TESTS=PASSED
TEST_FAILURES=NONE_REPORTED
~~~

Accordingly:

~~~text
PRODUCT_PUBLICATION=YES
HUMAN_REPORTED_ALL_REQUESTED_TESTS=PASS
~~~

No command-level transcript beyond the human report is invented.

## PERF026-D progress

The D issue owns:

~~~text
Array.each
Bytes.each
Environment.each
IdentityMap.each
Map.each
~~~

D1 completes the one-argument indexed snapshot family:

~~~text
Array.each=COMPLETE_FOR_PERF026_D
Bytes.each=COMPLETE_FOR_PERF026_D
~~~

The remaining owned families are:

~~~text
Map.each
IdentityMap.each
Environment.each
~~~

Map.each and IdentityMap.each form the next natural shared slice because both:

~~~text
validate callback before snapshot
establish a shallow association snapshot once
preserve insertion order
supply exactly two positional arguments: key, value
advance only after callback normal completion
return the original receiver
~~~

They therefore require the next bounded B-prime extension: exactly two ordinary
required positional parameters.

Environment.each is intentionally later because, despite also supplying two
arguments, it additionally owns complete portable String representability
prevalidation and deterministic portable ordering before callback #1.

## Cross references

- guillermomolina/protos#769 — PERF026-D.
- guillermomolina/protos#768 — PERF026-C completed.
- guillermomolina/protos#767 — PERF026-B completed.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/evidence/PERF026/PERF026_C1_WHILETRUE_LITERAL_PAIR_INLINE.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
