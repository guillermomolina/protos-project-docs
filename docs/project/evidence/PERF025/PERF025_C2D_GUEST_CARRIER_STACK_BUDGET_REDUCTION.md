# PERF025-C2D — guest-carrier stack budget reduction

Date: 2026-10-02

## Publication identity

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-C2D
TYPE=IMPLEMENTATION
PRODUCT_REPOSITORY=guillermomolina/protos

SLICE_BASE_REVISION=db4b221d0b4ddd52ae37d57b03ce36cdfc3e25a8
PROTOS_REVISION=19d7426a5b8f0e3b93d36f56aee33377a4ee9985
PROTOS_VERSION=0.3.143-SNAPSHOT
COMMIT_SUBJECT=PERF025-C2D: reduce guest carrier stack budget

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

## Published delta

The exact C2D commit changes:

~~~text
CHANGELOG.md
pom.xml
protos/benchmarks/README.md
src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java
~~~

The shared runtime stack authority changes from:

~~~text
64 MiB
~~~

to:

~~~text
16 MiB
~~~

through:

~~~java
public static final long GUEST_CALL_STACK_SIZE_BYTES = 16L * 1024 * 1024;
~~~

No second stack-size authority is introduced.

## Retained carrier architecture

C2D deliberately preserves the architecture selected at the C2C gate:

~~~text
DEDICATED_GUEST_CARRIER=RETAINED
TEST_TOOL_PER_CASE_CARRIER_USE=RETAINED
CARRIER_SERIALIZATION=RETAINED
AMBIENT_CALLER_STACK_DEPENDENCE=NOT_INTRODUCED
~~~

The budget remains an explicit fixed part of the reference runtime run identity.
C2D does not claim that 16 MiB is the minimum possible stack and does not retire
the carrier.

The product Javadoc records that PERF025/PLAT043 and PERF026/PLAT044 removed the
known artificial per-level root multipliers for the retained recursive workload.
The benchmark README now records the 16 MiB fixed carrier stack while preserving
the rule that the canonical recursive workload shape is not rewritten to fit host
defaults.

## Retained regression workload

The authoritative retained stack-capacity regression remains:

~~~text
protos/tests/conformance/regression/deep-recursive-closure-call-stack-capacity.protos
RECURSION_DEPTH=10000
~~~

C2D does not modify that file or its recursion count.

## Version and changelog

The implementation version advances:

~~~text
0.3.142-SNAPSHOT -> 0.3.143-SNAPSHOT
~~~

The matching 0.3.143 changelog entry states that:

- the shared explicit JVM guest-carrier budget is reduced from 64 MiB to 16 MiB;
- the dedicated guest carrier and Test Tool per-Case carrier use remain;
- serialization/placement behavior remains unchanged;
- the 10,000-deep regression workload remains unchanged;
- observable Protos semantics and the specification remain unchanged; and
- 16 MiB is a conservative fixed budget, not a claimed minimum.

## Validation evidence

The maintainer reports that the requested C2D validation completed successfully
before push:

~~~text
FOCUSED_VALIDATION=PASS
FULL_VALIDATION=PASS
ALL_REQUESTED_TESTS=PASS
~~~

This record relies on the human-reported validation result and does not infer
additional unpublished timing or benchmark measurements from the commit.

## PERF025 acceptance-state consequence

C2D satisfies the carrier-tax acceptance branch by reducing the historical
BUG008 fixed-stack requirement without changing Protos semantics:

~~~text
PERF025_C2D=COMPLETE
CURRENT_GUEST_CARRIER_STACK=16_MIB
BUG008_REOPENED=NO
~~~

PERF025/#758 is not closed by C2D alone.

The Issue acceptance contract still requires explicit repeated-call benchmark
evidence for the A/B reusable-call improvements using the established PERF024
primitive harness. Existing implementation publication and C2D correctness
validation do not substitute for that measurement evidence.

Therefore:

~~~text
PERF025_STATUS=OPEN
NEXT_SLICE=PERF025-D
NEXT_SLICE_TYPE=INVESTIGATION_MEASUREMENT
PRODUCT_CHANGE=NONE
HARNESS_CHANGE_EXPECTED=NO
~~~

PERF025-D should reconcile the current PERF024 harness authority, exact baseline
and final Protos revisions, and produce the smallest human-execution measurement
packet needed to satisfy the remaining before/after acceptance criterion. It must
not start a new open-ended profiling campaign.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#681` — historical BUG008, closed.
- `docs/project/evidence/PERF025/PERF025_A_DIRECT_ROOT_TASK_DISPATCH.md`.
- `docs/project/evidence/PERF025/PERF025_B_PREPARED_TOP_LEVEL_CALLABLE.md`.
- `docs/project/evidence/PERF025/PERF025_C2C_POST_B_PRIME_STACK_GATE_CLOSURE.md`.
- `guillermomolina/protos-benchmarks@5486c515369c8a958c8baa6949b818dbfaf7d0f8` — current PERF024 primitive JVM harness authority observed at C2D closure.
