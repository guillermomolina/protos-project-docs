# PERF011 — AUD016 EXPERIMENT_FIRST 25.4 rebaseline

Status: **COMPLETE — IMPLEMENTATION SEQUENCE ALLOCATED**

This durable, non-normative evidence record retains the read-only 25.4 rebaseline
of the three AUD016 `EXPERIMENT_FIRST` Bytecode DSL candidates and the resulting
live implementation routing under PERF011 / guillermomolina/protos#693.

It does not define Protos semantics and does not claim a performance improvement.

## Evidence identity

```text
DATE=2026-09-30

PROTOS_REVISION=738e2b9f5d8101f4229542afbaf8f8689785f680
PROTOS_VERSION=0.3.122-SNAPSHOT
PROTOS_GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
PROTOS_JDK=25.0.4.1.1

PROJECT_DOCS_BASELINE_REVISION=c9955e5539fe12a5cd696ecf47bf52a5c7b0ad7d

PERF011_ISSUE=guillermomolina/protos#693
AUD016_ISSUE=guillermomolina/protos#701

I073_ISSUE=guillermomolina/protos#744
I074_ISSUE=guillermomolina/protos#745
I075_ISSUE=guillermomolina/protos#746
```

The product HEAD above was re-read from the live `main` branch before the
investigation result was finalized. The product POM at that revision pins
`graalvm.version=25.4.4.1.1`.

## Governing project state

The preceding PERF011 scheduling checkpoint retained:

```text
PERF010_STATUS=PAUSED
PERF010A_STATUS=PAUSED
PERF020_STATUS=PAUSED
PERF011_STATUS=READY

PERF020_REQUIRED_FIRST=NO
PROTOS_BENCHMARKS_REQUIRED_FIRST=NO
```

AUD016 / #701 remains complete. Its foundational Bytecode DSL adoption work has
already been substantially consumed by later Protos implementation, including:

- frame-backed lexical representation;
- Bytecode DSL locals;
- materialized captured-local access;
- compact invocation state;
- guarded call selection;
- Boolean and Integer represented selection;
- lexical-layout precomputation.

Those completed surfaces are not reopened by this rebaseline.

## Revalidated 25.4 facilities

All three original AUD016 `EXPERIMENT_FIRST` facilities remain available in
Truffle 25.4.4.1.1:

```text
C1=enableTailCallHandlers
C2=boxingEliminationTypes
C3=enableUncachedInterpreter
```

Current Protos does not enable any of the three in
`ProtosBytecodeRootNode`.

## Cross-language / upstream precedent correction

The three candidates do **not** have the same precedent strength.

### C1 — bytecode-handler tail-call compilation

`enableTailCallHandlers` is a real Truffle Bytecode DSL facility introduced in
the 25.x line and retained in 25.4. It is exercised by upstream Bytecode DSL
tests and benchmark variants.

No use was found in the mature Truffle runtimes checked during this rebaseline,
and the current SimpleLanguage Bytecode DSL root does not enable it.

Therefore C1 is best classified as an early-adopter use of a real upstream
facility, not as copying a mature-language production convention.

```text
C1_CROSS_TRUFFLE_PRECEDENT=WEAK
C1_UPSTREAM_IMPLEMENTATION_EVIDENCE=STRONG
```

### C2 — boxing elimination

The current SimpleLanguage Bytecode DSL root enables:

```text
boxingEliminationTypes = {long.class, boolean.class}
```

Upstream also has extensive tests and benchmark coverage for boxing elimination.

However, current Protos guest Float, Boolean and Integer values normally travel
through semantic Protos value objects rather than a broad primitive-carrier
pipeline between Bytecode DSL operations. Merely adding primitive types to
`boxingEliminationTypes` would not by itself create such a pipeline.

Therefore useful Protos adoption requires first selecting a semantics-invisible
primitive carrier and its materialization boundaries.

```text
C2_CROSS_TRUFFLE_PRECEDENT=MODERATE
C2_IMPLEMENTATION_REQUIRES_CARRIER_SELECTION=YES
```

Float/`double` is the leading first candidate because it avoids the
unbounded-Integer overflow/promotion problem, but this is a hypothesis for the
investigation slice, not a pre-ratified representation decision.

### C3 — uncached interpreter

The current SimpleLanguage Bytecode DSL root enables
`enableUncachedInterpreter=true`.

Upstream Bytecode DSL documentation recommends uncached execution for reducing
cold-code allocation/footprint and improving startup. The facility transitions
roots to cached execution as they become hot. Operations must support uncached
execution or intentionally force cached execution.

No use was found in the mature pre-Bytecode-DSL runtimes checked, but that
absence is not a strong negative precedent because those runtimes do not all use
the same generated-interpreter architecture.

```text
C3_CROSS_TRUFFLE_PRECEDENT=MODERATE
C3_UPSTREAM_RECOMMENDATION=STRONG
```

## Adoption conclusion

The project should retain all three facilities as legitimate adoption targets
subject to exact Protos correctness and regression gates.

This does **not** mean blindly enabling every available Truffle option.

The acceptance rule is:

```text
adopt when:
  observable Protos semantics remain unchanged
  AND no incompatible architecture is introduced
  AND no clear material regression outweighs the intended value

do not:
  distort Protos representation merely to use a facility
  weaken semantic invariants
  create benchmark infrastructure as a prerequisite
```

The three facilities have different implementation shapes:

```text
C1
  direct implementation
  generator-owned change
  no pre-research required

C3
  implementation with compatibility discovery
  adapt operations only where bounded and appropriate
  stop if a substantive architecture decision appears

C2
  investigation first
  then conditional implementation
  first select a semantics-invisible primitive carrier and boundaries
```

## Allocated child work

GitHub native Parent/Sub-issue hierarchy is authoritative.

At publication preparation time the following children were created under
PERF011 / #693:

### I073 / #744

`I073 — Adopt Bytecode DSL bytecode-handler tail-call compilation`

```text
TYPE=IMPLEMENTATION
STATUS=READY
ORDER=1
PRE_RESEARCH_REQUIRED=NO
TARGET=enableTailCallHandlers
```

### I074 / #745

`I074 — Adopt Bytecode DSL uncached interpreter execution`

```text
TYPE=IMPLEMENTATION
STATUS=PAUSED
ORDER=2
DISCOVERY_DURING_IMPLEMENTATION=YES
TARGET=enableUncachedInterpreter
```

I074 is scheduled after I073. This is execution ordering, not a fabricated
semantic or technical blocker.

### I075 / #746

`I075 — Introduce the first primitive carrier and Bytecode DSL boxing elimination`

```text
TYPE=TWO_PHASE_IMPLEMENTATION_ITEM
STATUS=PAUSED
ORDER=3

I075-A=INVESTIGATION_ONLY
I075-B=CONDITIONAL_IMPLEMENTATION

TARGET=primitive carrier + boxingEliminationTypes
LIKELY_FIRST_CANDIDATE=Float/double
```

I075 is scheduled after I074. This is execution ordering, not a fabricated
native dependency.

## Native hierarchy verification

The three Issues were re-read through GitHub's native parent endpoint.

```text
I073/#744 PARENT=PERF011/#693 PASS
I074/#745 PARENT=PERF011/#693 PASS
I075/#746 PARENT=PERF011/#693 PASS

PERF011_SUB_ISSUES_TOTAL=3
```

Formal identifier post-create searches also returned exactly one Issue for each
of `I073`, `I074`, and `I075`.

## Current execution sequence

```text
PERF011/#693
  -> I073/#744 READY
  -> I074/#745 PAUSED
  -> I075/#746 PAUSED
```

No separate Dxxx/PLATxxx is allocated now.

A new design/platform decision is created only if implementation exposes a real
substantive choice that cannot be resolved as semantics-neutral implementation
machinery.

## Next action

```text
NEXT_SLICE=I073
NEXT_SLICE_TYPE=IMPLEMENTATION
NEXT_REPOSITORY=guillermomolina/protos

PERF020_REQUIRED_FIRST=NO
PROTOS_BENCHMARKS_REQUIRED_FIRST=NO
```

I073 should assume current local `guillermomolina/protos` is already at HEAD,
read the repository's current instructions and authority, enable
`enableTailCallHandlers` in the existing Bytecode DSL root, make only bounded
compatibility changes if the 25.4.4.1.1 generator requires them, and hand all
build/test/publication commands to the human executor.
