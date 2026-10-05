# I058 — final closure after BUG018 P-path revalidation

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=I058
PROTOS_ISSUE=guillermomolina/protos#661
DECISION_OWNER=D160
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative closure evidence. It reconciles the
previous I058 implementation, performance, CI, and P-path correctness/liveness
checkpoints.

## Implemented D160 result

I058's product implementation was published earlier at:

~~~text
I058_IMPLEMENTATION_REVISION=1ad6c5b5566a7e2c639a27ae95a8a545b03fd402
IMPLEMENTATION_VERSION=0.3.178-SNAPSHOT
SPECIFICATION_REVISION=0.1.440
~~~

The five privileged Core Array parallel selectors were removed and the
capability was relocated to source-backed functions in
`std:collections/Array`. The retained `Closure.parallel`, Future/Future.all
and isolated P substrate remained.

No privileged replacement worker/executor/task API, `Array.parallelEach`, or
writable collection-partition authority was introduced.

## Previously closed gates

### Performance

The canonical migrated `parallel-array-map` evidence was recorded at:

~~~text
BENCHMARK_REVISION=564dc97aacb593826011a8876554d69dd6529faa
SPEEDUP_1=1.000
SPEEDUP_2=1.715
SPEEDUP_4=2.544
SPEEDUP_8=4.473
MATERIAL_PERFORMANCE_REGRESSION=NO
I058_PERFORMANCE_CLOSURE_GATE=PASS
~~~

The absolute timings were not claimed comparable to PERF001-F; only the
methodologically qualified scaling shape was used for the I058 gate.

### General P/Future liveness containment

BUG017 repaired the independent carrier host-failure settlement defect at:

~~~text
BUG017_REVISION=a85d9ca846d7408f869916a59b72f02ee12b9222
BUG017_RESULT=PASS
~~~

A host failure can no longer leave the P Completion/Future pending indefinitely.

### Captured-materialized correctness blocker

BUG018 subsequently identified and repaired the concrete
`FrameSlotTypeException` that had triggered the observed P failure.

~~~text
BUG018_REPAIR_REVISION=234314b1791e5dc1c05ee2341ae46a56f8670512
BUG018_REPAIR_VERSION=0.3.215-SNAPSHOT
BUG018_REPAIR_LOCAL_VALIDATION=PASS
BUG018_REPAIR_EXACT_SHA_CI=SUCCESS
BUG018_REPAIR_CI_RUN=37325362553
~~~

BUG018-D then revalidated the canonical real P workload at:

~~~text
VALIDATED_HEAD=9c28dee1199840e6fcaa4fc914d115c6c90a37bd
VALIDATED_HEAD_SUBJECT=TEST009-M: amortized post-K compiler-expansion cleanup batch
VALIDATED_HEAD_CI_RUN=37339480515
VALIDATED_HEAD_CI_NUMBER=2162
VALIDATED_HEAD_CI_RESULT=SUCCESS
AFFINITY=0-1
P_SMOKE=PASS
FRESH_PROCESS_RUNS=10/10 PASS
TIMEOUTS=0
NONZERO_EXITS=0
FRAMESLOTTYPEEXCEPTION_OCCURRENCES=0
HOST_EXECUTION_FAILURE_OCCURRENCES=0
GIT_DIFF_CHECK=PASS
TRACKED_WORKTREE_CLEAN=YES
~~~

Therefore the final I058 correctness/liveness blocker is cleared.

## Final closure reconciliation

The closure requirements in #661 are all satisfied:

- the five parallel algorithms are available from `std:collections/Array`;
- the five former Core selectors are absent;
- retained Closure.parallel/Future/Future.all/P behavior is green;
- documentation, examples, benchmark source, tests and normative ownership were
  reconciled by the implementation;
- required benchmark/performance evidence is PASS;
- BUG017 liveness containment is repaired;
- BUG018 retained-frame correctness is repaired;
- the canonical P workload passes under the historical two-CPU failure
  affinity;
- the current validated HEAD has green required CI.

No additional I058 product change is required.

## Final verdict

~~~text
I058_PRODUCT_COMPLETE=YES
I058_PERFORMANCE_EVIDENCE=PASS
I058_PERFORMANCE_CLOSURE_GATE=PASS
BUG017_LIVENESS_CONTAINMENT=PASS
BUG018_REPAIR=PASS
BUG018_P_WORKLOAD_GATE=PASS
I058_CORRECTNESS_LIVENESS_GATE=PASS
CURRENT_HEAD_CI=PASS
I058_READY_TO_CLOSE=YES
NEXT_TECHNICAL_SLICE=NONE
~~~

I058 is complete.
