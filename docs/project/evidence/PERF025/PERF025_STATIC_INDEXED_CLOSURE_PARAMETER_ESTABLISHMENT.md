# PERF025 — Static indexed Closure parameter establishment

## Status

Published product implementation evidence for PERF025 / `guillermomolina/protos#758`.

This record is non-normative. It retains the exact product publication identity,
the structural implementation result, and the maintainer-reported validation
outcome for the static indexed Closure-parameter establishment slice.

## Exact product publication

```text
PROTOS_REVISION=bf08d8b4e106e299d3250ac16079767feb3e48a3
PROTOS_PARENT_REVISION=10b5a6b29c81d142c3348484d0b3dd8132f78d0d
PROTOS_VERSION=0.3.156-SNAPSHOT
COMMIT_SUBJECT=PERF025-H1: establish static Closure parameters by ordinal
OWNING_ISSUE=guillermomolina/protos#758
BASE_IS_EXACT_PARENT=YES
```

The exact comparison is one commit ahead of the preceding PERF025 virtual
return-home publication.

## Historical label note

The PERF025 stream had already used the textual slice label `PERF025-H1` for
the earlier lazy-root-Activation publication at
`a7e26228543f1af65a75e2e95bd6e384b67cc7d3`.

The later product commit at `bf08d8b4...` also uses `PERF025-H1` in its
published commit subject and changelog. This record does not rewrite or
reinterpret the earlier historical record. Exact product revision and the
descriptive evidence name are the disambiguating authorities.

## Bounded objective

The residual case addressed here was parameter establishment for Closure roots
that install a persistent frame-backed lexical authority.

Before this slice, a statically admitted parameter could still fall through the
generic named creation machinery:

```text
known static parameter identity
  -> createCurrentLocalSlotForRuntime(name, value)
  -> containsBinding(name)
  -> layout.offsetOf(name)
  -> putBinding(name, value)
  -> layout.offsetOf(name)
```

The implementation preserves the statically proven root/layout/ordinal through
lowering and establishes that binding directly at its known frame-backed
ordinal.

```text
STATIC_BINDING_IDENTITY=RETAINED
STATIC_BINDING_PRESENCE=DYNAMIC
NAME_TO_ORDINAL_REDISCOVERY_FOR_ELIGIBLE_PARAMETER=REMOVED
```

## Published implementation shape

`CanonicalToBytecodeLowerer` now tracks the persistent root layout and emits
new indexed parameter operations for statically proven root-owned parameter
bindings:

```text
BindClosureIndexedParameter
BindClosureIndexedRest
```

The lowerer carries the exact `ProtosFrameLexicalLayout`, parameter ordinal,
and parameter name. The name remains metadata/fallback input; physical frame
slot selection on the indexed path uses the proven ordinal.

`ProtosFrameLexicalBindingAuthority.createFrameBackedBindingAt(int, Object)`
then:

- derives the name from the authority's own layout at that ordinal;
- checks frame-local presence with `isCleared`;
- rejects duplicate creation, including PRESENT(null);
- records establishment order exactly as before; and
- writes the authoritative frame local directly at the ordinal.

```text
STATIC_PARAMETER_USES_INDEXED_AUTHORITY_ESTABLISHMENT=YES
GENERIC_NAME_LOOKUP_FOR_STATIC_PARAMETER=NO
FRAME_BACKED_SINGLE_AUTHORITY=PRESERVED
```

## Admission and generic fallback

The indexed path is admitted only while the activation's current authority:

- is the installed `ProtosFrameLexicalBindingAuthority`;
- stores the exact layout carried by the lowered operation; and
- is currently allowed to create a local binding.

`ProtosActivation.currentAuthorityAdmittingLocalCreationForRuntime()` returns
that authority for an unmaterialized execution Context, or for a materialized
OPEN execution Context still using the same authority.

CLOSED/FROZEN contexts, another authority, or an unproven binding retain the
existing named creation path.

```text
OPEN_CREATION_INDEXED_PATH=YES
CLOSED_FROZEN_GENERIC_RULE=PRESERVED
OTHER_AUTHORITY_GENERIC_RULE=PRESERVED
UNPROVEN_PARAMETER_GENERIC_RULE=PRESERVED
GENERIC_DYNAMIC_PARAMETER_FALLBACK_PRESERVED=YES
```

## Static identity does not imply static presence

The implementation preserves the D179 C0 distinction between stable identity
and dynamic presence.

```text
STATIC_BINDING_IDENTITY=YES
STATIC_BINDING_PRESENCE=NO
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
D179_C0_REMOVE_RECREATE=PASS
```

A parameter binding already PRESENT is still duplicate creation. A parameter
binding removed before its formal binding point can be re-established at the
same static ordinal when the execution Context remains semantically eligible.

No separate bitmap, side table, binding-cell authority, or duplicated parameter
value store was introduced.

## Parameter semantics retained

The product changelog explicitly records that defaults, rest Array creation,
argument transport, and the pre-existing frame-native
`BindClosureFrameParameter` path are unchanged.

The dedicated regression covers:

```text
LEFT_TO_RIGHT_PARAMETER_BINDING=PASS
DEFAULT_SEMANTICS=PASS
DUPLICATE_CREATION_ERROR=PASS
OPEN_CLOSED_FROZEN_RULES=PASS
REST_SEMANTICS=PASS
ESTABLISHMENT_ORDER_SEMANTICS=PASS
CONTEXT_REFLECTION=PASS
CAPTURE_BY_REFERENCE=PASS
FRAME_NATIVE_PARAMETER_PATH_PRESERVED=YES
GENERIC_DYNAMIC_PARAMETER_FALLBACK_PRESERVED=YES
```

The rest path still constructs the required fresh frozen standard guest Array;
only establishment of the statically proven lexical binding uses the indexed
authority seam.

## Existing frame-native path preserved

Closure roots that do not require persistent frame authority continue to use
the earlier frame-native parameter path.

```text
BindClosureFrameParameter
```

The new indexed operations are therefore a persistent-authority counterpart,
not a replacement for the compact frame-native path.

## Argument transport deliberately unchanged

This slice does not change supplied-argument transport or the compact frame ABI.

In particular, it does not claim removal of the remaining List-oriented
argument-access machinery.

```text
ARGUMENT_TRANSPORT_CHANGED=NO
LIST_SIZE_GET_OPTIMIZED=NO
COMPACT_FRAME_ARGUMENT_ABI_CHANGED=NO
```

That boundary remains a separate PERF025 follow-up.

## Dedicated structural regression

The product publication adds:

```text
src/test/java/com/guillermomolina/protos/execution/
  ProtosPerf025H1IndexedParameterEstablishmentTest.java
```

Its focused methods cover:

- persistent-authority parameters established by ordinal;
- sequential default visibility/absence;
- static identity versus dynamic presence;
- D179 remove/recreate and PRESENT(null);
- CLOSED/FROZEN fallback behavior;
- rest freshness/frozen/exact suffix semantics;
- Context reflection;
- capture by reference; and
- preservation of the pre-existing frame-native parameter root.

The test publishes the structural markers listed above rather than relying only
on a value-equivalence assertion.

## Exact product delta

Exact comparison:

```text
BASE=10b5a6b29c81d142c3348484d0b3dd8132f78d0d
HEAD=bf08d8b4e106e299d3250ac16079767feb3e48a3
COMMITS=1
FILES_CHANGED=8
ADDITIONS=662
DELETIONS=21
```

Changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025H1IndexedParameterEstablishmentTest.java
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.156-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

The newly added Protos-owned test source carries the repository APL-1.0 Part 5
notice.

## Validation provenance

The maintainer reported in the active interaction after product publication:

```text
PERF025-H1: establish static Closure parameters by ordinal pushed, test passed
```

The coordinating publication review independently verified:

```text
REMOTE_PUBLICATION=PASS
PRODUCT_REVISION=bf08d8b4e106e299d3250ac16079767feb3e48a3
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT=10b5a6b29c81d142c3348484d0b3dd8132f78d0d
BASE_IS_EXACT_PARENT=YES
PRODUCT_VERSION=0.3.156-SNAPSHOT
FILES_CHANGED=8
ADDITIONS=662
DELETIONS=21
FOCAL_TEST_PUBLISHED=PASS
CHANGELOG_RECORD=PASS
LICENSE_NOTICE_NEW_SOURCE=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command-by-command test result beyond the maintainer's report is invented.

## PERF025 audit consequence

This slice consumes the residual persistent-authority form of static parameter
binding through dynamic name lookup:

```text
PERSISTENT_AUTHORITY_STATIC_PARAMETER_NAME_TO_ORDINAL_LOOKUP
    -> CONSUMED for statically proven root-owned parameter bindings

STATIC_BINDING_PRESENCE_CHECK
    -> PRESERVED

GENERIC_DYNAMIC_BINDING_FALLBACK
    -> PRESERVED

FRAME_NATIVE_PARAMETER_BINDING
    -> PRESERVED

ARGUMENT_LIST_SIZE_GET_OVERHEAD
    -> NOT ADDRESSED

ESTABLISHMENT_ORDER_REPRESENTATION
    -> NOT REDESIGNED
```

## Architecture and semantic boundary

This implementation is an implementation-local continuation of the established
frame-backed lexical authority and compact-source-call direction.

```text
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLATFORM_DECISION_REQUIRED=NO
NO_DUAL_BINDING_AUTHORITY=PASS
```

## Performance-claim boundary

No benchmark, timing campaign, JFR profile, allocation profile, or attributable
per-call magnitude is part of this publication.

```text
BENCHMARK_RUN_FOR_THIS_SLICE=NO
ATTRIBUTABLE_NS_PER_CALL=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
PERFORMANCE_MAGNITUDE_CLAIMED=NO
```

The durable claim is structural only: eligible persistent-authority Closure
parameters no longer re-resolve a statically known frame-backed binding by name
to rediscover its ordinal.

## PERF025 status

This bounded implementation slice is complete. PERF025 itself remains open.

```text
PERF025_STATIC_INDEXED_PARAMETER_ESTABLISHMENT=COMPLETE
PERF025_STATUS=OPEN

STATIC_PARAMETER_ORDINAL_PATH=YES
PRESENCE_REMAINS_DYNAMIC=YES
D179_C0_PRESERVED=YES
ESTABLISHMENT_ORDER_SEMANTICS=PRESERVED
GENERIC_FALLBACK=PRESERVED
FRAME_NATIVE_PATH_PRESERVED=YES

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@bf08d8b4e106e299d3250ac16079767feb3e48a3`;
- the exact comparison against
  `10b5a6b29c81d142c3348484d0b3dd8132f78d0d`;
- the `0.3.156-SNAPSHOT` root CHANGELOG entry;
- `CanonicalToBytecodeLowerer.java`;
- `ProtosBytecodeRootNode.java`;
- `ProtosFrameLexicalBindingAuthority.java`;
- `ProtosSemanticBytecodeRootNode.java`;
- `ProtosActivation.java`;
- `ProtosPerf025H1IndexedParameterEstablishmentTest.java`;
- the immediately preceding PERF025 durable evidence; and
- the repository coordination and durable-record instructions.

This record is evidence only and does not replace live GitHub Issue
coordination.
