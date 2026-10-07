# PERF013 Slice B2 — Materialized captured-write implementation

Date: 2026-09-27

## Published product checkpoint

```text
WORK_ITEM=PERF013/#724
PARENT=PERF010-B/#722
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a
PRODUCT_VERSION=0.3.102-SNAPSHOT

PERF013_SLICE_B2=IMPLEMENTED
SEMANTIC_CHANGE=NO
NEW_PLAT_DECISION=NO
```

Published commit:

```text
0af8960363a557dad1b87968cf8a632e4716ee8a
PERF013 Slice B2: use MaterializedLocalAccessor for captured writes
```

## Exact changed paths

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI068Slice5CapturedMaterializedLexicalLoweringTest.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf013SliceB2MaterializedCapturedWriteTest.java
```

## Implementation result

For a statically proven `CapturedResolved` write whose owner is available in
the same physical `BytecodeRootNodes` group, lowering now uses:

```text
ResolveCapturedMaterializedWritableLexicalTarget
AssignCapturedMaterializedLocal
```

with a Bytecode-DSL-generated `MaterializedLocalAccessor` derived from the
compile-time-proven owner `BytecodeLocal`.

The previous runtime-authority path remains only as the safe fallback when the
owner local is not available in the current physical root group:

```text
ResolveCapturedWritableLexicalTarget
AssignCapturedFrameLocal
```

Slice B1 captured reads remain unchanged.

## Destination-before-RHS invariant

B2 preserves:

```text
resolve exact destination
 -> retain destination
evaluate RHS
 -> may mutate lexical topology
revalidate exact retained destination
 -> do not re-resolve
write exact retained destination
```

For a selected materialized owner, mutation rechecks presence through the same
accessor and retained owner frame. If that exact selected binding is no longer
present, the existing mutation error is produced; assignment does not retarget
to another lexical candidate.

## Absent static owner

A `CapturedResolved` identity can remain statically known while its runtime
frame slot is currently cleared under D179 C0:

```text
STATIC_OWNER_IDENTITY=KNOWN
RUNTIME_OWNER_PRESENCE=ABSENT
```

If the static owner's slot is absent when a write destination is resolved, the
resolver must select the farther/generic writable fallback present at that
moment. If the RHS later recreates the static owner's slot, destination-before-
RHS semantics require the write to remain locked to the previously selected
fallback.

The B2 checklist requested a dedicated standalone regression for this exact
write-side scenario. The published B2 focal class does not contain that exact
standalone case:

```text
OWNER_ABSENT_BEFORE_RESOLUTION_DIRECT_B2_TEST=NOT_PUBLISHED
```

This is recorded as a focal-coverage gap, not an observed semantic failure.
Adjacent C0 presence/fallback and no-post-RHS-retarget regressions are green.

## Focal coverage

The new B2 test class contains 13 focused tests covering ordinary same-group
materialized writes, capture by reference, escaped and multi-depth writes,
destination-before-RHS, removal/recreation effects, CLOSED/FROZEN mutation,
nearer binding precedence, default-parameter assignment, object-body capture,
and isolated-rematerialization fallback.

## Maintainer-reported validation

```text
mvn -q -o clean test-compile=PASS
PERF013_B2_FOCAL_SUITE=PASS
PERF013_B1_REGRESSIONS=PASS
PERF013_A_A2_A3_REGRESSIONS=PASS
I068_I071_D179_RELATED_REGRESSIONS=PASS
ALL_com.guillermomolina.protos.execution.*Test=PASS
make test=PASS
FINAL_REQUIRED_VALIDATION=PASS
REMOTE_CI_PASS=NOT_CLAIMED
```

The focal suite initially exposed one stale instruction-identity assertion. It
was updated for the newly migrated same-group write path, after which the focal
and broader execution tests passed.

The maintainer also reported `git diff --check` clean during implementation.

### Non-benchmark timing observation

The maintainer additionally observed the integrated Protos test phase decreasing
from roughly 132 seconds to roughly 119 seconds in the current environment.

This is retained only as an encouraging local observation:

```text
MAKE_TEST_PROTOS_PREVIOUS_OBSERVED_SECONDS=132
MAKE_TEST_PROTOS_CURRENT_OBSERVED_SECONDS=119
FORMAL_PERFORMANCE_CLAIM=NO
```

No causal or reproducible performance conclusion is drawn from this single
integrated-suite observation.

## Immediate compiler causal gate

B2 publication and semantic validation do not yet establish the compiler
acceptance discriminator.

The next checkpoint must rerun the same bounded compiler lifecycle gate against:

```text
TARGET_PROTOS_REVISION=0af8960363a557dad1b87968cf8a632e4716ee8a

EXPECTED_CAPTURED_READ_PE_FAILURE=ABSENT
EXPECTED_CAPTURED_WRITE_PE_FAILURE=ABSENT
```

The B2-specific failure expected to disappear is:

```text
CompilerAsserts.partialEvaluationConstant
 -> LocalRangeAccessor.isCleared
 -> ProtosFrameLexicalBindingAuthority.hasFrameBackedBindingAt
 -> ResolveCapturedWritableLexicalTarget.perform
```

Previously observed independent `Too deep inlining` failures on other roots are
not this gate's B2 discriminator.

## Routing

```text
PERF013_STATUS=IN_PROGRESS
PERF013_B2_IMPLEMENTATION=PUBLISHED
PERF013_B2_LOCAL_VALIDATION=PASS
PERF013_B2_COMPILER_CAUSAL_GATE=PENDING

NEXT_TASK=PERF013_B2_POST_PUBLICATION_COMPILER_CAUSAL_GATE
NEXT_TASK_TYPE=VALIDATION_EXPERIMENT
NEXT_REPOSITORY=guillermomolina/protos-benchmarks

PERF013_SLICE_C=AFTER_B2_GATE_IF_STILL_REQUIRED
```
