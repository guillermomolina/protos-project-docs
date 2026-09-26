# I072 Phase E slice 2 — remaining structured-control convergence evidence

FORMAL_IDENTIFIER=I072
PHASE=Phase E — structured-control convergence and closure
SLICE=2
GITHUB_ISSUE=https://github.com/guillermomolina/protos/issues/719
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
BASE_PROTOS_REVISION=e11ca30fd3345da8813114bcffbb76ba0c0f234d
PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
PROTOS_VERSION=0.3.96-SNAPSHOT
COMMIT_MESSAGE=I072 Phase E (slice 2): converge remaining structured-control families
PLAT040_AUTHORITY=docs/project/decisions/platform/PLAT040_TRUFFLE_HOT_PATH_INVOCATION_ARCHITECTURE.md@9d972a84f17cbf652cb8497d76617e76a06ac39c
PHASE_E_SLICE_1_EVIDENCE=docs/project/evidence/I072/I072_PHASE_E_SLICE_1_GUARDED_STRUCTURED_SEND_CONVERGENCE.md@e115a292dee3f05e11d5055a6597dbb6594ceb9e
PHASE_E_IMPLEMENTATION_SCOPE=COMPLETE
I072_IMPLEMENTATION_SCOPE=COMPLETE
I072_CLOSURE_STATE=OPEN_PENDING_FINAL_VALIDATION

## Result

This product revision completes the structured-control families identified by I072 Phase E.

The guarded structured-send specialization now recognizes stable canonical selections for:

```text
Object.ensure
Error.handle
Object.while
Boolean callbacks

Array.each
Bytes.each
ProcessArguments.each
Environment.each
IdentityMap.each
Map.each

Map.at / containsKey / atIfAbsent
Map.atPut
Map.remove

Object.caseOf
Array.match
Map.match
IdentityMap.atIfAbsent
```

For the ordinary-send path, establishment still begins with the Phase A authoritative guarded D013 selection:

```text
ProtosValueLookup.lookupGuarded(...)
```

and only admits an exact canonical selection under the corresponding behavior/home/provenance contract and selector-specific stability Assumption.

A valid guarded hit therefore reuses the already-selected canonical operation instead of repeating the general D013 lookup and generic implementation-family classification.

## Collection and map convergence

The existing `GuardedStructuredKind` descriptor was extended for:

```text
ARRAY_EACH
BYTES_EACH
PROCESS_ARGUMENTS_EACH
ENVIRONMENT_EACH
IDENTITY_MAP_EACH
MAP_EACH
MAP_READ_LOOKUP
MAP_AT_PUT
MAP_REMOVE
```

The cached descriptor now also retains the exact Map structured read-lookup kind for the `at` / `containsKey` / `atIfAbsent` family.

Existing canonical-selection helpers are used directly for Array, Bytes, ProcessArguments, Environment, IdentityMap, and Map operations.

## Final four structured paths

The product also converges four structured paths that were previously recognized later from native-body identity:

```text
Object.caseOf
Array.match
Map.match
IdentityMap.atIfAbsent
```

New canonical-selection helpers were added for:

```text
ProtosStandardObjectProtocol.isCanonicalStandardCaseOfSelection
ProtosStandardArrayProtocol.isCanonicalStandardMatchSelection
ProtosStandardMapProtocol.isCanonicalStandardMatchSelection
```

`IdentityMap.atIfAbsent` reuses the already-existing:

```text
ProtosStandardIdentityMapProtocol.isCanonicalStandardAtIfAbsentSelection
```

The guarded ordinary-send admission therefore requires exact canonical behavior/home provenance instead of selector spelling or native-body identity alone.

The downstream `NativeCall` structured accessors remain unchanged for these four operations. Their native-body-based recognition is retained as compatibility machinery for direct Closure execution and other paths that do not possess reusable D013 ordinary-send provenance.

## Generic/direct compatibility

The product deliberately retains the generic `finishPreparingComposedCallByImplementation` path and related implementation classifiers.

The root changelog states that those paths remain required by:

```text
cache miss / megamorphic fallback
direct Closure invocation
Object.call
standard import
other paths without reusable guarded D013 provenance
```

No dead compatibility machinery was identified by this slice.

This is consistent with the Phase E rule: remove compatibility machinery only when all real consumers are gone.

## Focused product tests

`ProtosI072PhaseEStructuredSendConvergenceTest` now retains focused warmed-site coverage for:

```text
canonical ensure + nearer override invalidation
canonical Error.handle + fresh Error identity across repeated hits
canonical Array.each + nearer override invalidation
canonical Map.atPut + nearer override invalidation
```

The product therefore contains direct regression coverage for canonical guarded execution, stale-hit invalidation, and fallback to newly selected ordinary behavior.

This durable record does not manufacture execution results that were not independently supplied.

## Product delta

The Phase E slice-2 delta from `e11ca30f...` to `cf9b39b...` changes:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardArrayProtocol.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardMapProtocol.java
src/main/java/com/guillermomolina/protos/execution/ProtosStandardObjectProtocol.java
src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseEStructuredSendConvergenceTest.java
```

## Phase E implementation status

The product changelog explicitly states:

```text
This completes the structured families identified for I072 Phase E.
```

Accordingly, the implementation-scope status is:

```text
ALL_IDENTIFIED_PHASE_E_STRUCTURED_FAMILIES_MIGRATED=YES
PHASE_E_IMPLEMENTATION_SCOPE=COMPLETE

GUARDED_STRUCTURED_ENSURE=YES
GUARDED_STRUCTURED_ERROR_HANDLE=YES
GUARDED_STRUCTURED_WHILE=YES
GUARDED_STRUCTURED_BOOLEAN=YES

GUARDED_STRUCTURED_ARRAY_EACH=YES
GUARDED_STRUCTURED_BYTES_EACH=YES
GUARDED_STRUCTURED_PROCESS_ARGUMENTS_EACH=YES
GUARDED_STRUCTURED_ENVIRONMENT_EACH=YES
GUARDED_STRUCTURED_IDENTITY_MAP_EACH=YES
GUARDED_STRUCTURED_MAP_EACH=YES

GUARDED_STRUCTURED_MAP_READ_LOOKUP=YES
GUARDED_STRUCTURED_MAP_AT_PUT=YES
GUARDED_STRUCTURED_MAP_REMOVE=YES

GUARDED_STRUCTURED_OBJECT_CASE_OF=YES
GUARDED_STRUCTURED_ARRAY_MATCH=YES
GUARDED_STRUCTURED_MAP_MATCH=YES
GUARDED_STRUCTURED_IDENTITY_MAP_AT_IF_ABSENT=YES

SELECTOR_NAME_ALONE_AUTHORIZES_SPECIALIZATION=NO
CANONICAL_HOME_PROVENANCE_REQUIRED=YES
GENERIC_FALLBACK_PRESERVED=YES
DIRECT_CLOSURE_COMPATIBILITY_PRESERVED=YES
DEAD_COMPATIBILITY_MACHINERY_REMOVED=NONE_REQUIRED
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

## Final closure state

I072's implementation scope is now present in the product at `cf9b39b25dc9a3c4cd1c538749c3a363760ae45b`.

However, the formal Issue closure contract also requires final integrated validation and structural confirmation across all phases A-E.

At the time this record was prepared, the GitHub Actions `test` check for the product revision was still running.

Therefore this record does not infer a final remote-CI PASS or close I072 administratively:

```text
I072_IMPLEMENTATION_SCOPE=COMPLETE
I072_CLOSURE_STATE=OPEN_PENDING_FINAL_VALIDATION
REMOTE_CI_AT_CAPTURE=IN_PROGRESS
```

PERF010-A remains the owner of attributable runtime measurement after the completed I072 product revision is admitted as the final validated F′ baseline.
