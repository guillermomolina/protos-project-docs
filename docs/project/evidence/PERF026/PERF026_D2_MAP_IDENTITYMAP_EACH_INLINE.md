# PERF026-D2 — Map.each / IdentityMap.each association-callback checkpoint

Status: **PUBLISHED IMPLEMENTATION EVIDENCE**

Date: 2026-10-02

## Identity

~~~text
WORK_ITEM=PERF026-D/#769
SLICE=PERF026-D2
PRODUCT_REPOSITORY=guillermomolina/protos

BASE_REVISION=f366ae819291a3ad9a6590c8cc4a8cd2b74d1e93
PROTOS_REVISION=14558fd9ea6758698ae4a8d4d38a273438212176
PROTOS_VERSION=0.3.141-SNAPSHOT
COMMIT_SUBJECT=PERF026-D2: inline Map and IdentityMap each literal callbacks

PLAT044=#766 RATIFIED_B_PRIME
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

D2 is exactly one commit ahead of the base revision above. The base is the
independent DIST010-B publication that landed after PERF026-D1, so this record
binds D2 to the actual product history rather than pretending direct ancestry
from D1.

## Changed product paths

~~~text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf026D2AssociationEachInlineCallbackTest.java
~~~

## Published D2 architecture

D2 extends the PLAT044 B-prime standard-each consumer only to an immediate
Closure literal with exactly two ordinary required positional parameters:

~~~text
(key, value) => ...
~~~

with:

~~~text
no default parameters
no rest parameter
no nested Closure literal in the callback body
existing B-prime ReturnHome/context/plan conditions satisfied
~~~

This two-parameter association candidate remains separate from:

~~~text
zero-parameter Boolean/while candidates
one-parameter Array/Bytes indexed-each candidate
~~~

Actual optimized execution is authorized only after ordinary lookup/selection
prepares exact standard Map.each or IdentityMap.each behavior with exactly the
staged callback value.

## Local each convergence

D2 generalizes the D1 local-loop machinery over a common prepared-each view
rather than duplicating the inline callback region.

The common local-each state preserves:

~~~text
validated receiver and callback
one collection-specific snapshot
fresh prepared callback child per snapshot position
exact supplied arguments in the child activation
advance only after normal callback completion
callback result ignored
original receiver returned
~~~

The collection-specific prepared calls retain sole authority over their
receiver validation and snapshots.

For Map/IdentityMap the snapshot contract remains:

~~~text
shallow logical association snapshot
insertion order
representative stored key object
exact value object captured at snapshot time
block(key, value)
~~~

No hash, equality, identity lookup or re-search is introduced merely to iterate.

## Structural evidence

The exact D2 focused regression records:

~~~text
PERF026_D2_MAP_EACH_LOCAL_LOOP=YES
PERF026_D2_MAP_HELPER_ROOT_REMOVED=YES
PERF026_D2_MAP_CALLBACK_ROOT_REMOVED=YES
PERF026_D2_MAP_EXACT_KEY_VALUE_ARGUMENTS=YES

PERF026_D2_IDENTITY_MAP_EACH_LOCAL_LOOP=YES
PERF026_D2_IDENTITY_MAP_HELPER_ROOT_REMOVED=YES
PERF026_D2_IDENTITY_MAP_CALLBACK_ROOT_REMOVED=YES
PERF026_D2_IDENTITY_MAP_EXACT_KEY_VALUE_ARGUMENTS=YES
PERF026_D2_IDENTITY_MAP_IDENTITY_RESEARCH_INTRODUCED=NO
~~~

## Semantic preservation evidence

Map:

~~~text
PERF026_D2_MAP_ASSOCIATION_SNAPSHOT_PRESERVED=YES
PERF026_D2_MAP_INSERTION_ORDER_PRESERVED=YES
PERF026_D2_MAP_CALLBACK_RESULT_IGNORED=YES
PERF026_D2_MAP_RECEIVER_RESULT_IDENTITY=YES
~~~

IdentityMap:

~~~text
PERF026_D2_IDENTITY_MAP_ASSOCIATION_SNAPSHOT_PRESERVED=YES
PERF026_D2_IDENTITY_MAP_INSERTION_ORDER_PRESERVED=YES
PERF026_D2_IDENTITY_MAP_CALLBACK_RESULT_IGNORED=YES
PERF026_D2_IDENTITY_MAP_RECEIVER_RESULT_IDENTITY=YES
~~~

Shared callback semantics:

~~~text
PERF026_D2_CAPTURE_BY_REFERENCE_PRESERVED=YES
PERF026_D2_FRESH_ACTIVATION_PER_CALLBACK=YES
PERF026_D2_NLR_PRESERVED=YES
PERF026_D2_ERROR_PROPAGATION_PRESERVED=YES
PERF026_D2_SUSPENSION_RESUMPTION_PRESERVED=YES
PERF026_D2_COMPLETED_PREFIX_REPLAY=NO
PERF026_D2_INLINE_ROOTTAG=YES
PERF026_D2_INLINE_SCOPE_PROJECTION=YES
~~~

Both callback formals bind through the ordinary Closure parameter-binding
machinery from the fresh prepared child activation.

## Fallback / arity evidence

The exact regression records:

~~~text
PERF026_D2_DYNAMIC_CALLBACK_FALLBACK=YES
PERF026_D2_NESTED_CLOSURE_LITERAL_FALLBACK=YES
PERF026_D2_DEFAULT_REST_PARAMETER_FALLBACK=YES
PERF026_D2_UNSUPPORTED_ARITY_FALLBACK=YES
PERF026_D2_ARITY_PREVALIDATION_INTRODUCED=NO
PERF026_D2_OWNED_RETURN_HOME_FALLBACK=YES
PERF026_D2_NONCLOSURE_INVOKABLE_FALLBACK=YES
PERF026_D2_CUSTOM_EACH_FALLBACK=YES
PERF026_D2_INDEXED_EACH_D1_PRESERVED=YES
~~~

Therefore D2 does not turn the optimization's two-parameter source shape into a
new semantic callback-arity prevalidation rule.

## Human-reported validation

The human executor reported for this exact published slice:

~~~text
PERF026-D2: inline Map and IdentityMap each literal callbacks pushed
TEST=PASSED
TEST_FAILURES=NONE_REPORTED
~~~

Accordingly:

~~~text
PRODUCT_PUBLICATION=YES
HUMAN_REPORTED_TESTS=PASS
~~~

The report does not identify a more specific command set, so this evidence does
not fabricate focal/full-suite detail or claim a broader command transcript.

## PERF026-D progress

Completed owned families:

~~~text
Array.each=COMPLETE_FOR_PERF026_D
Bytes.each=COMPLETE_FOR_PERF026_D
Map.each=COMPLETE_FOR_PERF026_D
IdentityMap.each=COMPLETE_FOR_PERF026_D
~~~

Remaining owned family:

~~~text
Environment.each
~~~

Environment.each uses the same two-argument literal shape already proven by D2,
but has an additional semantic cutover that must remain exact:

~~~text
callback callability validated first
complete portable (String, String) representation validation second
complete portable snapshot sorted into canonical Environment name order
only then may callback #1 be prepared
~~~

The existing PreparedEnvironmentEachCall already owns that entire cutover via
ProtosStandardEnvironmentProtocol.portableEntriesForStructured(...).

The next natural slice is therefore PERF026-D3: reuse D2's proven
two-parameter B-prime callback path and local-each machinery for exact standard
Environment.each while retaining PreparedEnvironmentEachCall as the sole
prevalidation/snapshot/order authority.

If D3 publishes with required validation and no new blocker is discovered, all
standard each families owned by PERF026-D will be implemented.

## Cross references

- guillermomolina/protos#769 — PERF026-D.
- guillermomolina/protos#768 — PERF026-C completed.
- guillermomolina/protos#767 — PERF026-B completed.
- guillermomolina/protos#766 — PLAT044.
- guillermomolina/protos#681 — BUG008 remains closed.
- docs/project/evidence/PERF026/PERF026_D1_ARRAY_BYTES_EACH_INLINE.md.
- docs/project/decisions/platform/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_BOUNDARY.md.
