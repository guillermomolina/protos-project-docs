# PERF025 — Conditional Closure frame materialization

Status: **PUBLISHED / MAINTAINER-REPORTED VALIDATION PASS**

Date: 2026-10-02

Formal owner: `PERF025 / guillermomolina/protos#758`

This durable, non-normative record retains the bounded PERF025 product slice
published as:

`PERF025: conditional frame materialization for Closure roots`

The slice consumes the post-F1 audit finding that any declared ROOT/CLOSURE
binding forced unconditional Truffle frame materialization. It does not change
observable Protos semantics, does not change the specification, does not reopen
PLAT040, and does not claim a measured performance magnitude.

## Publication identity

```text
PROTOS_REPOSITORY=guillermomolina/protos

BASE_REVISION=fe078828412aabfc48dc1835bf34dce784307b3e
BASE_VERSION=0.3.150-SNAPSHOT
BASE_SUBJECT=PERF025: compact deferred invocation for direct source Closure calls

PRODUCT_REVISION=245b0001e4cb7c7ae3192f35d262a8dcf34fdcd6
PRODUCT_VERSION=0.3.151-SNAPSHOT
COMMIT_SUBJECT=PERF025: conditional frame materialization for Closure roots

BASE_IS_EXACT_PARENT=YES
```

The exact product commit changes nine paths and advances the implementation
version by one snapshot revision.

At the time this durable record was prepared, `protos/main` had already
advanced one direct child further to:

```text
CURRENT_PRODUCT_HEAD=9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618
CURRENT_PRODUCT_SUBJECT=PERF025: lazy shared lexical environment for Closure capture
CURRENT_HEAD_PARENT=245b0001e4cb7c7ae3192f35d262a8dcf34fdcd6
```

That later slice is not part of this record. The frame-materialization evidence
below remains revision-bound to `245b0001...`.

## Trigger finding

The retained PERF025 post-F1 pay-as-you-grow audit established the previous
physical admission rule:

```text
root declares at least one frame-backed binding
    -> InstallFrameLexicalAuthority
    -> VirtualFrame.materialize()
    -> MaterializedFrame
    -> ProtosFrameLexicalBindingAuthority
```

The admission did not require a capture, context observation, reflection,
debugger access, or frame escape. A parameter alone was sufficient.

Consequently an ordinary Closure such as:

```protos
identity: (value) => value
```

could materialize its frame on every invocation solely because `value` was a
declared binding.

The optimization target was therefore representation-only:

```text
HAS_LOCAL_OR_PARAMETER != MUST_MATERIALIZE_FRAME
```

while retaining the established single-authority, capture-by-reference,
PRESENT/ABSENT and BUG013 lifetime invariants.

## Published lowering policy

`CanonicalToBytecodeLowerer` no longer emits
`InstallFrameLexicalAuthority` merely because `frameLocals` is non-empty.

Instead it applies a conservative
`requiresPersistentFrameAuthority(...)` decision.

Only Closure roots are eligible to remain frame-native. Persistent authority is
retained conservatively for cases that may require current bindings outside the
live-frame-only representation, including:

```text
top-level/module roots
nested Closures
inline literal callbacks
Closure defaults containing a nested Closure
Object construction
context intrinsic
compose
Candidate/Dynamic or otherwise non-Resolved access to a declared current name
unrecognized canonical forms
```

Thus the published rule is structurally:

```text
Closure root proven live-frame-confined
    -> no entry InstallFrameLexicalAuthority
    -> current static bindings stay in ordinary BytecodeLocals

otherwise
    -> existing BUG013-safe MaterializedFrame authority path
```

The predicate is intentionally conservative. It does not reduce the general
runtime path merely because one static capture class happens to be absent.

## Frame-native binding establishment

Removing the eager authority alone would have been insufficient because
parameter/current-binding establishment previously flowed through activation
named-slot authority APIs and could materialize the general representation on
the first write.

The published slice therefore adds frame-native establishment operations for
statically proven current-root bindings:

```text
BindClosureFrameParameter
BindClosureFrameRest
CreateCurrentFrameLocal
MultipleCreateFrameLocals
```

These operations establish supplied/defaulted/rest parameters and statically
proven current creations directly through the current root's frame-local
layout.

The existing fast paths remain authoritative for ordinary same-scope
`Resolved` access:

```text
read  -> ReadFrameLocal / constant LocalAccessor
write -> PERF028-A current resolved write path / constant LocalAccessor
```

The slice therefore removes the need for a materialized lexical authority as
the ordinary storage mechanism for an admitted Closure root.

## Presence and creation semantics

Physical allocation of a `BytecodeLocal` still does not mean the semantic
binding is PRESENT.

The frame-native establishment path preserves the existing
presence-by-not-cleared contract and the exact creation semantics, including:

```text
STATIC_LOCAL_EXISTS != SEMANTIC_BINDING_PRESENT

sequential parameter establishment
earlier parameter visibility in later defaults
current/later parameter absence during default evaluation
duplicate creation failure
rest binding semantics
target-less single creation
D143 multiple creation
D179 PRESENT/ABSENT remove/recreate behavior
existing RHS/effect ordering
```

No second binding store was introduced.

## Observation transition while the frame is live

An admitted root can still encounter an activation whose execution Context has
already become observable.

The published implementation handles that case in-frame: on the first
frame-native establishment that discovers an already observed/materialized
activation, it installs the existing materialized-frame authority while the
current `VirtualFrame` is still live and adopts the bindings already present in
the frame.

Tooling scope projection follows the same lifetime-safe principle: when the
unobserved admitted root is queried through the debugger/tooling path, the
authority is installed from the live queried frame before projection.

Therefore the optimization does not depend on retaining a raw frame for later
use.

## BUG013 lifetime invariant

BUG013 remains authoritative.

The implementation does **not** restore the rejected historical shape:

```text
retain raw VirtualFrame
    -> owner root returns
    -> materialize it later
```

Persistent lexical authority continues to retain `MaterializedFrame`, not a
raw `VirtualFrame`.

The focal regression explicitly rejects fields that retain a non-materialized
`VirtualFrame` across the relevant activation/authority/tooling objects.

```text
RAW_VIRTUALFRAME_RETAINED_ACROSS_ROOT_LIFETIME=NO
BUG013_LIFETIME_INVARIANT=PRESERVED
```

## Captures and general fallback

A Closure root that contains a real nested Closure keeps the persistent
authority at this revision.

This preserves genuine lexical capture-by-reference and escaped-lifetime
semantics without trying to solve non-escaping capture in the same slice.

Likewise, explicit `context` observation and non-`Resolved` access to an own
name retain the general authority route.

The slice does not reinterpret `Candidate` or `Dynamic` as static
`Resolved` access.

## Exact product delta

Published changed paths:

```text
M CHANGELOG.md
M pom.xml
M src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java
M src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
M src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
M src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
A src/test/java/com/guillermomolina/protos/execution/ProtosPerf025FrameMaterializationSliceTest.java
```

Exact commit statistics:

```text
FILES_CHANGED=9
ADDITIONS=1194
DELETIONS=84
```

Version publication:

```text
IMPLEMENTATION_VERSION=0.3.151-SNAPSHOT
ROOT_CHANGELOG_ENTRY=YES
SPECIFICATION_CHANGE=NO
```

## Focal regression evidence

The new `ProtosPerf025FrameMaterializationSliceTest` publishes structural and
semantic coverage for the bounded change.

Its focal cases include:

1. parameter-only Closure execution without materialized frame authority;
2. same-root local-only execution remaining in ordinary frame locals;
3. sequential default parameters and supplied/defaulted behavior;
4. duplicate frame-native creation preserving the ordinary creation error;
5. an observed activation transitioning to materialized authority while the
   frame is live and preserving existing bindings;
6. a captured escaping Closure retaining persistent authority and
   capture-by-reference mutation;
7. explicit `context` observation retaining persistent authority;
8. non-`Resolved` access to an own name retaining the general fallback;
9. structural rejection of raw `VirtualFrame` retention.

The test source emits acceptance markers including:

```text
PARAMETER_ONLY_REQUIRES_MATERIALIZED_FRAME=NO
LOCAL_ONLY_REQUIRES_MATERIALIZED_FRAME=NO
SEQUENTIAL_PARAMETERS_DEFAULTS=PASS
DUPLICATE_CREATE_ERROR=PASS
OBSERVED_ACTIVATION_IN_FRAME_TRANSITION=PASS
CAPTURED_ESCAPE_SEMANTICS=PASS
CONTEXT_OBSERVATION=PASS
CANDIDATE_DYNAMIC_FALLBACK=PASS
RAW_VIRTUALFRAME_RETAINED_ACROSS_ROOT_LIFETIME=NO
```

These are published regression assertions and structural markers. This durable
record does not claim an independent re-execution by the coordinating agent.

## Validation provenance

The maintainer reported in the active interaction:

```text
PERF025: conditional frame materialization for Closure roots pushed tests passed
```

The coordinating publication review independently verified:

```text
REMOTE_PUBLICATION=PASS
PRODUCT_REVISION=245b0001e4cb7c7ae3192f35d262a8dcf34fdcd6
COMMIT_SUBJECT_MATCH=PASS
EXACT_PARENT=fe078828412aabfc48dc1835bf34dce784307b3e
PRODUCT_VERSION=0.3.151-SNAPSHOT
EXPECTED_CHANGED_PATHS=PASS
FOCAL_TEST_PUBLISHED=PASS
```

Therefore:

```text
MAINTAINER_REPORTED_VALIDATION=PASS
INDEPENDENT_TEST_REEXECUTION_BY_COORDINATING_AGENT=NO
```

No command-by-command validation result beyond the maintainer's report is
invented.

## Specification and architecture impact

```text
OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
NEW_PLAT_DECISION=NO

PLAT040_F_PRIME_PAY_AS_YOU_GROW_DIRECTION=PRESERVED
PLAT036_SINGLE_BINDING_AUTHORITY=PRESERVED
I068_CAPTURE_BY_REFERENCE=PRESERVED
D179_PRESENT_ABSENT_SEMANTICS=PRESERVED
PERF028_RESOLVED_CURRENT_WRITE_FAST_PATH=PRESERVED
BUG013_FRAME_LIFETIME_RULE=PRESERVED
```

No specification path is changed by the exact product commit.

## Audit-candidate consequence

Relative to the retained PERF025 post-F1 audit:

```text
P0 EAGER_FRAME_MATERIALIZATION_FOR_ANY_LOCAL
    -> CONSUMED BY THIS SLICE

P1 PARAMETER_BINDING_THROUGH_DYNAMIC_SLOT_AUTHORITY
    -> CONSUMED FOR ADMITTED STATIC CURRENT-ROOT BINDINGS
    -> GENERAL/DYNAMIC FALLBACK REMAINS

P1 INLINE_CALLBACK_LOSES_FRAME_NATIVE_LEXICAL_PATH
    -> NOT ADDRESSED

P0 UNIVERSAL_CLOSURE_CONTEXT_CAPTURE
    -> NOT ADDRESSED BY THIS EXACT REVISION

P1 RETURN_HOME_PHYSICALLY_CREATED_WHEN_UNOBSERVED
    -> NOT ADDRESSED
```

The subsequent direct child `9ecb5c7d...` begins a separate Closure-capture
slice. Its implementation and validation are intentionally not folded into this
frame-materialization record.

## Performance-claim boundary

No benchmark or profiling measurement is part of this slice.

The record establishes a structural physical-cost removal only:

```text
EAGER_MATERIALIZE_FOR_ANY_DECLARED_BINDING=NO
PARAMETER_ONLY_REQUIRES_MATERIALIZED_FRAME=NO

ATTRIBUTABLE_NS_PER_CALL=NOT_MEASURED
SURVIVING_HEAP_REDUCTION=NOT_MEASURED
END_TO_END_SPEEDUP_PERCENT=NOT_MEASURED
```

No Fibonacci speedup is claimed. At the exact revision retained here,
Fibonacci-style nested literal callbacks remain a real capture case and
conservatively retain persistent frame authority.

## PERF025 status

This slice does not close PERF025.

```text
PERF025_FRAME_MATERIALIZATION_SLICE=COMPLETE
PERF025_STATUS=OPEN

EAGER_FRAME_MATERIALIZATION_FOR_ANY_LOCAL=CONSUMED
STATIC_CURRENT_BINDING_ESTABLISHMENT_FRAME_NATIVE=YES
RAW_VIRTUALFRAME_RETENTION=NO

OBSERVABLE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BENCHMARK_RESULT_CLAIMED=NO
```

The next work item is not selected by this record.

## Materially inspected publication evidence

The coordinating publication review inspected:

- `guillermomolina/protos@245b0001e4cb7c7ae3192f35d262a8dcf34fdcd6`;
- its exact parent
  `fe078828412aabfc48dc1835bf34dce784307b3e`;
- the exact commit path/statistics payload;
- `CHANGELOG.md` content contained in the product commit;
- the published
  `ProtosPerf025FrameMaterializationSliceTest.java` regression source;
- the retained PERF025 post-F1 pay-as-you-grow audit;
- the preceding direct source Closure compact/deferred evidence;
- the repository PERF, implementation, coordination and durable-record
  instructions;
- the current product head relation showing
  `9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618` as a direct child of the
  retained product revision.

This record is evidence only and does not replace live GitHub Issue
coordination.
