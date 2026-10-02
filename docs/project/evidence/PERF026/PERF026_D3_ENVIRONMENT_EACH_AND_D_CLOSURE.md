# PERF026-D3 — Environment.each inline callback and PERF026-D closure checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE / PERF026-D COMPLETE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-D/#769
SLICE=PERF026-D3
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=14558fd9ea6758698ae4a8d4d38a273438212176
PROTOS_REVISION=db4b221d0b4ddd52ae37d57b03ce36cdfc3e25a8
PROTOS_VERSION=0.3.142-SNAPSHOT
COMMIT_SUBJECT=PERF026-D3: inline Environment each literal callback

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

D3 is exactly one commit ahead of D2.

## Changed product paths

~~~text
CHANGELOG.md
Makefile
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardEnvironmentProtocol.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026D3EnvironmentEachInlineCallbackTest.java
~~~

The Makefile change preserves live phase output while retaining the phase exit
status without relying on shell pipefail. It does not change Protos language
semantics.

## Published D3 architecture

D3 reuses the D2 two-required-parameter PLAT044 B-prime literal shape unchanged:

~~~text
(name, value) => ...
~~~

Eligibility remains bounded to an immediate Closure literal with exactly two
ordinary required positional parameters, no default, no rest and no nested
Closure literal in the callback body, together with the existing B-prime
ReturnHome/context/plan conditions.

The send-site source candidate is shared with Map/IdentityMap. Runtime capability
authority distinguishes Environment.each from the association-each families.

After ordinary lookup/selection prepares exact standard Environment.each with
exactly the staged callback, the loop is sequenced in the containing semantic
source root. PreparedEnvironmentEachCall joins the common local-each view
directly rather than being represented as a Map-like association snapshot.

## Environment semantic cutover preserved

PreparedEnvironmentEachCall remains the semantic authority for:

~~~text
receiver validation
callback callability validation
complete portable (String, String) conversion/validation
canonical Environment name ordering
fresh per-entry callback preparation
advance only after normal callback completion
ignored callback result
original Environment receiver result
~~~

The required observable order remains:

~~~text
callback callability validation
  -> complete portable validation of every entry
  -> complete canonical ordering
  -> callback #1 may be prepared
~~~

No callback prefix can run before a later invalid portable entry is discovered.

The exact callback child still carries:

~~~text
supplied[0] = portable name String
supplied[1] = portable value String
~~~

and the inline region binds both formals through ordinary Closure parameter
binding from the fresh prepared child activation.

## Canonical-home guard

The generic structured preparation path retains Environment standard-each
implementation identity but not the selected Closure object. D3 therefore adds
an explicit home proof:

~~~text
ProtosStandardEnvironmentProtocol.isCanonicalStandardEachHome(...)
~~~

This preserves the ordinary path for a canonical-looking standard each body
copied to another home. Implementation identity alone is not treated as
selection authority.

## Structural evidence

The exact D3 regression records:

~~~text
PERF026_D3_ENVIRONMENT_EACH_LOCAL_LOOP=YES
PERF026_D3_ENVIRONMENT_HELPER_ROOT_REMOVED=YES
PERF026_D3_ENVIRONMENT_CALLBACK_ROOT_REMOVED=YES
PERF026_D3_EXACT_NAME_VALUE_ARGUMENTS=YES
PERF026_D3_TWO_PARAMETER_BINDING_FROM_PREPARED_ACTIVATION=YES
PERF026_D3_INLINE_ROOTTAG=YES
PERF026_D3_INLINE_SCOPE_PROJECTION=YES
~~~

## Environment semantic evidence

~~~text
PERF026_D3_CANONICAL_NAME_ORDER_PRESERVED=YES
PERF026_D3_CALLBACK_RESULT_IGNORED=YES
PERF026_D3_RECEIVER_RESULT_IDENTITY=YES

PERF026_D3_WHOLE_PORTABLE_PREVALIDATION_PRESERVED=YES
PERF026_D3_CALLBACK_BEFORE_VALIDATION=NO
~~~

## Callback/control preservation

~~~text
PERF026_D3_FRESH_ACTIVATION_PER_CALLBACK=YES
PERF026_D3_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_D3_NLR_PRESERVED=YES
PERF026_D3_ERROR_PROPAGATION_PRESERVED=YES
PERF026_D3_SUSPENSION_RESUMPTION_PRESERVED=YES
PERF026_D3_COMPLETED_PREFIX_REPLAY=NO
~~~

## Fallback preservation

~~~text
PERF026_D3_DYNAMIC_CALLBACK_FALLBACK=YES
PERF026_D3_NESTED_CLOSURE_FALLBACK=YES
PERF026_D3_DEFAULT_REST_FALLBACK=YES
PERF026_D3_UNSUPPORTED_ARITY_FALLBACK=YES
PERF026_D3_ARITY_PREVALIDATION_INTRODUCED=NO
PERF026_D3_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_D3_NONCLOSURE_INVOKABLE_FALLBACK=YES
PERF026_D3_CUSTOM_EACH_FALLBACK=YES
PERF026_D3_COPIED_STANDARD_HOME_FALLBACK=YES
~~~

## Prior D-family paths preserved

The D3 focused regression also records:

~~~text
PERF026_D3_ARRAY_BYTES_D1_PRESERVED=YES
PERF026_D3_MAP_IDENTITYMAP_D2_PRESERVED=YES
~~~

Therefore D3 completes the Environment consumer without replacing the bounded
one-parameter indexed-each path or the two-parameter Map/IdentityMap path.

## Human-reported validation

The human executor reported for this exact published slice:

~~~text
PERF026-D3 pushed
TODO_PASS=YES
TEST_FAILURES=NONE_REPORTED
~~~

Interpreted as:

~~~text
PRODUCT_PUBLICATION=YES
HUMAN_REPORTED_REQUIRED_VALIDATION=PASS
~~~

The exact command transcript is not reconstructed or invented.

## PERF026-D completion

PERF026-D owns these standard synchronous each families:

~~~text
Array.each
Bytes.each
Environment.each
IdentityMap.each
Map.each
~~~

Published completion map:

~~~text
Array.each=COMPLETE
Bytes.each=COMPLETE
Map.each=COMPLETE
IdentityMap.each=COMPLETE
Environment.each=COMPLETE
~~~

All five use the ratified PLAT044 B-prime representation only after ordinary
selection/capability proof and preserve collection-specific
snapshot/prevalidation/order authority plus exact generic fallback.

Accordingly:

~~~text
PERF026_D_IMPLEMENTATION=COMPLETE
PERF026_D_VALIDATION=PASS_HUMAN_REPORTED
PERF026_D_SEMANTIC_CHANGE=NO
PERF026_D_SPECIFICATION_CHANGE=NO
PERF026_D_NEXT_IMPLEMENTATION_SLICE=NONE
~~~

## PERF026 family status

PERF026-A/#765 completed the discovery/classification audit and is already
closed. PLAT044/#766 is ratified and closed. The implementation children are:

~~~text
PERF026-B/#767=COMPLETED
PERF026-C/#768=COMPLETED
PERF026-D/#769=COMPLETED_BY_D3
~~~

The original PERF026 audit deliberately left several families as DEFER/REJECT
rather than unfinished implementation:

~~~text
Object.ifNull / ifNotNull=DEFER
Map/IdentityMap.atIfAbsent=DEFER
Array.match / Map.match / Object.caseOf=DEFER
ensure / Error.handle=DEFER
async continuation local open-coding=REJECT
Map hash/equality as PERF026 family=REJECT
stdlib HOF wrappers=TRANSITIVE_DEFER
~~~

Those dispositions do not create another mandatory PERF026 implementation
slice.

## Related next performance gate

PERF025/#758 remains open. Its latest retained coordination says that after the
first B-prime Boolean publication the post-B-prime unchanged-workload stack
measurement became actionable before any carrier-size or carrier-retirement
decision.

D3 does not itself change or authorize that carrier decision.

~~~text
PERF025_STATUS=READY
NEXT_RELEVANT_PERFORMANCE_GATE=POST_B_PRIME_STACK_EVIDENCE
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

## Cross references

- guillermomolina/protos#765 — PERF026 audit, closed.
- guillermomolina/protos#766 — PLAT044, closed/ratified.
- guillermomolina/protos#767 — PERF026-B, closed/completed.
- guillermomolina/protos#768 — PERF026-C, closed/completed.
- guillermomolina/protos#769 — PERF026-D, completed by this checkpoint.
- guillermomolina/protos#758 — PERF025, open; post-B-prime stack evidence remains relevant.
- guillermomolina/protos#681 — BUG008 remains historically closed.
- docs/project/evidence/PERF026/PERF026_D1_ARRAY_BYTES_EACH_INLINE.md.
- docs/project/evidence/PERF026/PERF026_D2_MAP_IDENTITYMAP_EACH_INLINE.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
