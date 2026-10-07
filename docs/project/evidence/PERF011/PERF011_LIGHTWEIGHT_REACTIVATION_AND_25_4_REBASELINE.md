# PERF011 — Lightweight runtime-fit reactivation and performance-workflow reset

Status: **OWNER-APPROVED SCHEDULING / EXECUTION-POLICY CHECKPOINT**

This durable, non-normative record retains the project-owner decision made after
the current PERF010-A 25.4 hot-root diagnostic harness proved operationally
expensive and still failed to produce usable measurement evidence.

The decision does not reinterpret historical retained benchmark evidence. It
changes the scheduling and evidence burden for future performance work.

## Evidence identity

```text
DATE=2026-09-30

PROTOS_REVISION=738e2b9f5d8101f4229542afbaf8f8689785f680
PROTOS_BENCHMARKS_REVISION=90efd2ef7ca713547f50911aca27dda74bc4c41c
PROJECT_DOCS_BASELINE_REVISION=2c93f0aab16532dcd02c4c67ddf4a6715665b537

PERF010_ISSUE=guillermomolina/protos#680
PERF010A_ISSUE=guillermomolina/protos#691
PERF011_ISSUE=guillermomolina/protos#693
PERF020_ISSUE=guillermomolina/protos#741
AUD016_ISSUE=guillermomolina/protos#701
```

The benchmark revision above intentionally retains the current hot-root harness
as non-working infrastructure:

```text
PERF010-A: retain non-working hot-root diagnostic harness
```

It is historical/engineering evidence, not a working measurement authority.

## Owner scheduling decision

```text
PERF010_STATUS=PAUSED
PERF010A_STATUS=PAUSED
PERF020_STATUS=PAUSED
PERF011_STATUS=READY

PERF010A_25_4_DISCRIMINATOR_REQUIRED_BEFORE_NEXT_OPTIMIZATION=NO
PERF020_COMPARATOR_REQUIRED_BEFORE_NEXT_OPTIMIZATION=NO

PROTOS_BENCHMARKS_REFACTOR=DEFERRED_SEPARATE_WORK
HISTORICAL_PERFORMANCE_EVIDENCE=RETAINED
```

PERF010 and PERF010-A are not rejected. Their retained causal and compiler
evidence remains useful, but the project will not continue paying the current
cost of precise attribution before every next implementation candidate.

PERF020 remains valid as a prospective comparator/refactor direction, but its
previous demand-driven trigger is superseded: the next ordinary performance
intervention does not have to implement PERF020 first.

PERF011 is reactivated as the broad runtime-representation / Truffle-fit owner.

## Lightweight performance workflow

The ordinary development loop becomes:

```text
current implementation / cross-runtime evidence
  -> identify one concrete representation or Truffle-idiom mismatch
  -> bounded semantics-preserving implementation
  -> required correctness validation
  -> lightweight make test-protos before/after sanity comparison
  -> retain the implementation if independently architecturally justified
  -> continue to the next candidate
```

The lightweight timing check is intentionally non-authoritative:

- prefer a small repeated before/after sample, e.g. three runs per side and the
  median, rather than treating one noisy full-suite run as exact evidence;
- a clear large movement is useful directional evidence;
- a small difference is classified as no demonstrated performance change;
- no whole-language performance claim is made from `make test-protos`;
- a semantics-preserving architecture improvement may remain even when no clear
  timing gain is demonstrated.

Deep retained benchmark work is reserved for cases where it is independently
needed, for example:

- a surprising or material regression;
- a large effect whose cause must be understood;
- a release/platform decision that genuinely depends on precise comparison;
- a concrete architecture choice whose acceptance requires compiler-visible
  evidence rather than source-level fit.

This policy deliberately removes the benchmark harness from the critical path of
ordinary runtime-improvement work.

## Relationship to AUD016

AUD016 / #701 remains the completed Bytecode-DSL capability inventory. Much of
its foundational adoption backlog has already been consumed by later platform
and implementation work, including frame-backed lexical representation,
materialized captured locals, compact invocation state and guarded selection.

The still-interesting AUD016 `EXPERIMENT_FIRST` backlog is:

```text
C1 bytecode-handler tail-call compilation
C2 boxing elimination
C3 uncached interpreter/startup mode
```

AUD016 originally evaluated these against Graal/Truffle 25.3.4.1. Protos now
uses GraalVM/Graal/Truffle 25.4.4.1.1, and the product implementation has changed
substantially since that audit. Therefore none of C1/C2/C3 is automatically
selected for implementation from the old audit alone.

## PERF011 next slice

The next slice is investigation only:

```text
SLICE=PERF011_AUD016_EXPERIMENT_FIRST_25_4_REBASELINE
TYPE=INVESTIGATION
REPOSITORY_MUTATION=NO
COMMAND_EXECUTION=NO

CANDIDATES=
  C1 bytecode-handler tail-call compilation
  C2 boxing elimination
  C3 uncached interpreter/startup mode

REQUIRED_RESULT=
  select exactly one smallest coherent next implementation target
  OR establish that none should currently be implemented
```

The investigation must consume:

- current `guillermomolina/protos` HEAD;
- current Graal/Truffle 25.4.4.1.1 API/source/documentation;
- the retained AUD016 conclusions;
- mature Truffle implementations where they provide transferable implementation
  precedent.

It must not run builds, tests, benchmarks, profilers, generated-code probes or
repository commands.

For each candidate classify:

```text
CURRENT_25_4_APPLICABILITY=
CURRENT_PROTOS_FIT=
SEMANTIC_RISK=
IMPLEMENTATION_COMPLEXITY=
EXPECTED_PRIMARY_VALUE=
CROSS_TRUFFLE_PRECEDENT=
ARCHITECTURE_DECISION_REQUIRED=
IMPLEMENTATION_READY=
```

The final result must contain:

```text
NEXT_IMPLEMENTATION_TARGET=
  BYTECODE_HANDLER_TAIL_CALLS |
  BOXING_ELIMINATION |
  UNCACHED_INTERPRETER |
  NONE

NEXT_IMPLEMENTATION_SCOPE=<smallest coherent bounded scope or NONE>
NEXT_IMPLEMENTATION_REPOSITORY=guillermomolina/protos | NONE
NEW_DECISION_REQUIRED=YES | NO
PERF020_REQUIRED_FIRST=NO
PROTOS_BENCHMARKS_REQUIRED_FIRST=NO
```

## Boundaries

- no observable Protos semantic change;
- no broad runtime rewrite;
- do not reopen completed PERF012-PERF016 merely to obtain another timing claim;
- do not treat old 25.3 API details as current 25.4 authority without
  revalidation;
- do not create a benchmark harness as part of this investigation;
- do not mutate historical evidence;
- all repository coordinates use the canonical `guillermomolina` spelling.
