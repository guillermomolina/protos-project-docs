# PERF012 — frame lexical layout implementation and closure evidence

Date: 2026-09-27

## Scope

This record retains the publication and closure evidence for:

`guillermomolina/protos#723 — PERF012 — Precompute frame lexical layout for PE-safe authority installation`.

PERF012 is a child of PERF010-B / #722 and addresses only the independently
identified frame-authority construction bailout. Captured-local
`MaterializedLocalAccessor` remediation remains owned by PERF013 / #724.

## Baseline and publication

```text
PARENT=PERF010-B/#722

BASELINE_PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
BASELINE_PROTOS_VERSION=0.3.96-SNAPSHOT

PERF012_PRODUCT_REVISION=a85e66292c4f0314b91c07c1fec3cce058be752c
PERF012_PRODUCT_VERSION=0.3.97-SNAPSHOT
COMMIT_MESSAGE=PERF012: precompute frame lexical layout for PE-safe authority installation
```

The publication commit is exactly one commit after the PERF010-B investigation
baseline.

Changed product paths:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalBindingAuthority.java
src/main/java/com/guillermomolina/protos/execution/ProtosFrameLexicalLayout.java
src/test/java/com/guillermomolina/protos/execution/ProtosPerf012FrameLexicalLayoutTest.java
```

No normative specification path changed.

## Published implementation

The product commit introduces backend-private immutable
`ProtosFrameLexicalLayout`.

`CanonicalToBytecodeLowerer` now constructs that descriptor once while lowering
a genuine ROOT/CLOSURE helper root:

```text
frameLocalNames
  -> ProtosFrameLexicalLayout.of(frameLocalNames)
  -> InstallFrameLexicalAuthority ConstantOperand
```

`InstallFrameLexicalAuthority` passes the already-built descriptor into each
runtime authority instance.

Each `ProtosFrameLexicalBindingAuthority` therefore retains:

```text
shared immutable layout
+ LocalRangeAccessor
+ current BytecodeNode
+ current frame
+ per-invocation dynamic overflow / establishment state
```

instead of rebuilding its own `ArrayList` and
`LinkedHashMap<String,Integer>` from the static names on every invocation.

The descriptor contains layout metadata only:

- declaration-order binding names;
- name-to-ordinal mapping.

It contains no binding value, presence state, frame, dynamic overflow, or
semantic execution-context state.

## Causal result

The implementation removes the exact avoidable host-construction prefix identified
by the PERF010-B failure-discrimination investigation:

```text
ProtosFrameLexicalBindingAuthority.<init>
  -> per-invocation LinkedHashMap.put
  -> HashMap/Class generic machinery
  -> Too deep inlining
```

from the per-invocation authority constructor.

The corresponding PERF012 structural compiler gate and requested validation were
reported by the maintainer after publication as:

```text
MAINTAINER_REPORT=todo verde
PERF012_COMPILER_GATE=PASS_REPORTED_BY_MAINTAINER
FRAME_AUTHORITY_TOO_DEEP_INLINING=ABSENT_REPORTED_BY_MAINTAINER
FINAL_REQUIRED_VALIDATION=PASS_REPORTED_BY_MAINTAINER
```

No separate post-PERF012 compiler trace/log artifact was published to
`guillermomolina/protos-benchmarks` at this checkpoint, so this record does not
invent root IDs, trace lines, or a benchmark revision for that rerun.

The existing retained Step-0 benchmark evidence remains:

```text
STEP0_BENCHMARK_REVISION=aeffc2014f5202141feb4a5f2b0680cdc09869fc
STEP0_BENCHMARK_COMMIT=perf010b: capture Step 0 compiler gate evidence
```

That revision predates the PERF012 product publication and remains baseline
evidence only.

## Semantic/runtime invariants

Source inspection of the published delta plus the maintainer-reported green
validation establish the intended bounded result:

```text
STATIC_LEXICAL_LAYOUT_PRECOMPUTED=YES
PER_INVOCATION_LAYOUT_MAP_REBUILD=NO

FRAME_BACKED_SINGLE_AUTHORITY=PASS
D179_C0=PASS
I071_FRAME_NATIVE_PRESENCE=PASS
PRESENT_NULL_DISTINCT_FROM_ABSENT=PASS
DYNAMIC_OVERFLOW=PASS
CAPTURE_BY_REFERENCE=PASS
ESTABLISHMENT_ORDER=PASS
CONTEXT_REFLECTION_PROJECTION=PASS

SEMANTIC_CHANGE=NO
SPEC_CHANGE=NO
```

The implementation does not alter captured-local representation; the
`LocalRangeAccessor.isCleared` PE-constant failure identified by PERF010-B is
still independently owned by PERF013.

## Validation provenance

The maintainer explicitly reported after publication:

```text
subido:
PERF012: precompute frame lexical layout for PE-safe authority installation

todo verde.
```

This record therefore retains the requested local/focal/integrated/compiler-gate
validation as maintainer-reported PASS.

GitHub commit status/workflow APIs exposed no associated remote CI result for
`a85e66292c4f0314b91c07c1fec3cce058be752c` at record time.

```text
REMOTE_CI_PASS=NOT_CLAIMED
```

## Coordination

Native GitHub hierarchy is now confirmed:

```text
PERF012=#723
NATIVE_PARENT=#722
NATIVE_PARENT=PASS

PERF013=#724
PERF013_NATIVE_PARENT=#722
PERF013_NATIVE_PARENT=PASS
```

PERF012 has independent closure and is complete.

```text
PERF012_STATUS=COMPLETE
NEXT_WORK_ITEM=PERF013/#724
NEXT_WORK_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos
```

PERF013 must begin from current HEAD, not from its historical opening baseline.

Current product baseline entering PERF013:

```text
PROTOS_REVISION=a85e66292c4f0314b91c07c1fec3cce058be752c
PROTOS_VERSION=0.3.97-SNAPSHOT
```

After PERF013 is published and its captured-local PE failure is removed,
PERF010-B must rerun the combined compiler/timing gate before Step 2 is allowed
to proceed.
