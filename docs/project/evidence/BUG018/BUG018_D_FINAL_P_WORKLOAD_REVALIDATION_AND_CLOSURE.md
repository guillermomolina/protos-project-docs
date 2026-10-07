# BUG018-D — final P workload revalidation and closure

Date: 2026-10-05

## Work identity

~~~text
WORK_ITEM=BUG018
VALIDATION_SLICE=BUG018-D
PROTOS_ISSUE=guillermomolina/protos#801
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
~~~

This record is durable non-normative closure evidence. It does not replace the
live GitHub Issue state or the normative Protos specification.

## Validated product state

~~~text
VALIDATED_HEAD=9c28dee1199840e6fcaa4fc914d115c6c90a37bd
VALIDATED_HEAD_SUBJECT=TEST009-M: amortized post-K compiler-expansion cleanup batch
BUG018_C_REVISION=234314b1791e5dc1c05ee2341ae46a56f8670512
BUG018_C_PRESENT_IN_VALIDATED_HEAD=YES
BUG018_C_VERSION=0.3.215-SNAPSHOT
VALIDATED_HEAD_CI_RUN=37339480515
VALIDATED_HEAD_CI_NUMBER=2162
VALIDATED_HEAD_CI_RESULT=SUCCESS
~~~

BUG018-C is an ancestor of the validated HEAD. The validation therefore covers
the published retained-frame repair plus the subsequent TEST009-M compiler
cleanup state.

## Validation performed

The maintainer rebuilt the checkout executable without running tests:

~~~text
COMMAND=mvn -q package -DskipTests
RESULT=PASS
TESTS_EXECUTED=NO
~~~

The final acceptance workload was the canonical source-backed P workload:

~~~text
WORKLOAD=protos/benchmarks/concurrency/parallel-array-map.protos
AFFINITY=taskset -c 0-1
AVAILABLE_CPUS=0-15
WATCHDOG=180 seconds per process
~~~

CPUs 0-1 were permitted and therefore the validation reused exactly the CPU pair
that exposed the historical failure.

### Smoke

~~~text
P_SMOKE=PASS
EXIT_STATUS=0
TIMEOUT=NO
STDOUT=EMPTY
STDERR=EMPTY
~~~

### Fresh-process campaign

Ten fresh sequential processes were executed under the same two-CPU affinity.

~~~text
CAMPAIGN_RUNS_PASS=10/10
TIMEOUTS=0
NONZERO_EXITS=0
OBSERVED_RUNTIME_RANGE=2-6 seconds
STDERR_TOTAL_BYTES=0
FRAMESLOTTYPEEXCEPTION_OCCURRENCES=0
HOST_EXECUTION_FAILURE_OCCURRENCES=0
~~~

The signature search returned no matches. Its grep status 1 therefore means
"no matching failure signature", not workload failure.

No sleep, retry-until-pass behavior, concurrent campaign execution, or benchmark
timing claim was used.

## Repository state

~~~text
GIT_DIFF_CHECK=PASS
TRACKED_WORKTREE_CLEAN=YES
VALIDATION_OUTPUT_LOCATION=target/bug018-d/
TRACKED_FILES_CHANGED=NO
~~~

The validation artifacts are untracked build/output material under `target/`.
No product, specification, version, or changelog content was modified by this
slice.

## Closure synthesis

BUG018-A proved the cause: a retained frame could keep a local physically
PRESENT while another activation's uncached-to-cached transition published a
cached BytecodeNode whose local-kind metadata for that local remained ILLEGAL.

BUG018-B selected the smallest safe dual read mechanism.

BUG018-C implemented it:

- compile-time-proven captured reads select the owner/presence separately and
  read with Bytecode DSL `LoadLocalMaterialized`;
- inline captured reads use the same safe split without forcing activation
  materialization;
- retained `ProtosFrameLexicalBindingAuthority` value reads use
  `BytecodeLocation.update()` plus public `BytecodeNode.getLocalValue` and
  public local offsets behind the existing boundary;
- writes remain unchanged;
- deterministic tier-transition regressions cover both retained-read surfaces.

BUG018-C had maintainer-reported full local validation PASS and exact-SHA CI
#2161 / run `37325362553` SUCCESS. BUG018-D now closes the only remaining
acceptance gate by exercising the canonical real P workload under the same
two-CPU affinity that historically exposed the failure.

## Final verdict

~~~text
BUG018_D_VERDICT=PASS
BUG018_C_PRESENT_IN_VALIDATED_HEAD=YES
AFFINITY=0-1
P_SMOKE=PASS
CAMPAIGN_RUNS_PASS=10/10
TIMEOUTS=0
NONZERO_EXITS=0
FRAMESLOTTYPEEXCEPTION_OCCURRENCES=0
HOST_EXECUTION_FAILURE_OCCURRENCES=0
GIT_DIFF_CHECK=PASS
TRACKED_WORKTREE_CLEAN=YES
BUG018_ACCEPTANCE_COMPLETE=YES
BUG018_READY_TO_CLOSE=YES
I058_BUG018_BLOCKER=CLEARED
SEMANTIC_CHANGE=NO
D_OR_PLAT_DECISION_REQUIRED=NO
NEXT_SLICE=NONE
NEXT_SLICE_TYPE=NONE
~~~

BUG018 is complete.
