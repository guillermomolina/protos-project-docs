# PERF025-C2C — post-B-prime guest-stack gate closure

Date: 2026-10-02

## Identity and scope

~~~text
WORK_ITEM=PERF025/#758
SLICE=PERF025-C2C
TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos

PRODUCT_REVISION=db4b221d0b4ddd52ae37d57b03ce36cdfc3e25a8
PRODUCT_VERSION=0.3.142-SNAPSHOT

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
PRODUCT_CHANGE_DURING_C2C=NONE
CARRIER_CHANGE_DURING_C2C=NONE
~~~

PERF025-C2C closes the post-PLAT044 B-prime stack-evidence gate required before
changing the historical BUG008 64 MiB guest-carrier budget. It does not attempt
to prove the smallest host stack on which every current or future Protos program
can execute.

The retained regression workload is:

~~~text
protos/tests/conformance/regression/deep-recursive-closure-call-stack-capacity.protos
RETAINED_RECURSION_DEPTH=10000
~~~

The source workload was not rewritten or reduced for this investigation.

## Post-B-prime topology

At the product revision above, the relevant retained recursive shape is:

~~~text
R_i = repeat semantic Closure root

R_i
  -> [eligible standard ifTrue literal callback runs inline in R_i]
  -> R_i+1
~~~

The completed PERF025/PLAT043 and PERF026/PLAT044 work has removed the earlier
persistent Boolean structured-helper and eligible ifTrue callback CallTargets
from each accumulated recursion level.

For this exact retained source, the persistent accumulated topology is therefore:

~~~text
ROOT_CALLTARGETS_PER_RECURSION_LEVEL=
  1 repeat semantic Closure CallTarget
  + 0 standard-Boolean structured-helper CallTargets
  + 0 eligible ifTrue callback CallTargets
~~~

The ordinary operation and identity calls inside one iteration complete before
the next recursive repeat descent and do not add another persistent root per
accumulated recursion level.

## Human-executed stack observations

The human executor reported the following completed matrix results using one
built product artifact and the unchanged retained workload:

| Requested carrier stack | Completed runs | Reported result | StackOverflowError |
| --- | ---: | --- | --- |
| 64 MiB | 3/3 | PASS, exit 0 | no |
| 32 MiB | 3/3 | PASS, exit 0 | no |
| 16 MiB | 3/3 | PASS, exit 0 | no |
| 8 MiB | 3/3 | PASS, exit 0 | no |

The lower requested sizes are **not** accepted as stack-failure measurements:

~~~text
4_MIB_RUN_1=EXTERNALLY_TERMINATED_EXIT_143
4_MIB_RUN_2=EXTERNALLY_INTERRUPTED_EXIT_130
4_MIB_RUN_3=EXTERNALLY_INTERRUPTED_EXIT_130

2_MIB_RUNS=EXTERNALLY_INTERRUPTED_EXIT_130
1_MIB_RUNS=EXTERNALLY_INTERRUPTED_EXIT_130

VALID_STACK_FAILURE_BOUNDARY_FROM_4_2_1_MIB=NO
~~~

During the first 4 MiB attempt, a captured thread dump showed the exact Test Tool
carrier waiting in Process termination cleanup rather than executing the
recursive guest call at the observation instant. Because that process was later
terminated externally, C2C does not classify the point as either a stable pass
or a stack-overflow failure.

A later fresh 4 MiB diagnostic did not execute the workload at all. It exited
with code 70 while constructing the Polyglot context because:

~~~text
Option 'engine.Compilation' is experimental and must be enabled with
allowExperimentalOptions(boolean) in Context.Builder or Engine.Builder.
~~~

Accordingly that diagnostic supplies no stack-capacity result.

## Runtime-mode evidence boundary

C2C originally attempted to make JVM interpreter-only execution the primary
measurement mode by supplying:

~~~text
-Dpolyglot.engine.Compilation=false
~~~

The fresh diagnostic establishes that this mechanism is not admissible through
the current Protos Polyglot builder without additionally enabling experimental
options.

Therefore this closure deliberately does **not** claim:

~~~text
INTERPRETER_ONLY_BOUNDARY_ESTABLISHED=NO
EXACT_MINIMUM_STACK_ESTABLISHED=NO
4_MIB_STACK_FAILURE_ESTABLISHED=NO
2_MIB_STACK_FAILURE_ESTABLISHED=NO
1_MIB_STACK_FAILURE_ESTABLISHED=NO
~~~

The completed 64/32/16/8 MiB runs remain useful product observations, but C2C
does not relabel them as an interpreter-only minimum study.

## Why the exact minimum is not pursued

The product question is whether the historical 64 MiB workaround remains a
reasonable fixed budget after removal of the known per-level implementation
multipliers. It is not necessary to find the smallest stack request that happens
to survive one JDK, architecture, Truffle revision and host configuration.

The stable 8 MiB observations already establish that 64 MiB is substantially
larger than needed by the retained post-B-prime workload in the measured
configuration. Continuing through arbitrary 4/2/1 MiB boundaries would produce a
fragile host/runtime-specific number and is not required to choose a conservative
production budget.

C2C therefore terminates the minimum-stack search intentionally.

## Cross-Truffle context

Current GraalJS source provides useful scale context, not a Protos compatibility
requirement.

At exact GraalJS revision:

~~~text
oracle/graaljs@4c2b8dd51b23919c1d4ba72aa07ad45fbfaac2f7
graal-js/mx.graal-js/mx_graal_js.py
~~~

the mx runner defines:

~~~text
_default_stacksize():
  aarch64 -> 24m
  otherwise -> 16m
~~~

and adds the corresponding `-Xss` argument when the caller has not already
provided one.

This is development/runtime-launch infrastructure in GraalJS, not proof that all
Truffle languages have an identical stack model. It does establish that an
explicit host-stack budget in the 16-24 MiB range is not intrinsically anomalous
for a mature Truffle-language implementation.

## C2C conclusion

The owner selected a conservative fixed budget rather than continuing minimum
boundary measurement:

~~~text
PERF025_C2C_STATUS=COMPLETE

CURRENT_PRODUCT_STACK_BUDGET=64_MIB
LOWEST_STABLE_COMPLETED_OBSERVATION=8_MIB_3_OF_3_PASS
SELECTED_NEXT_PRODUCT_STACK_BUDGET=16_MIB

EXACT_MINIMUM_REQUIRED=NO
CONTINUE_4_2_1_MIB_SEARCH=NO

DEDICATED_GUEST_CARRIER=RETAIN
CARRIER_SERIALIZATION=RETAIN
CARRIER_RETIREMENT_AUTHORIZED=NO

BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

The 16 MiB selection is intentionally conservative: it is twice the lowest
stable completed observation retained here and remains an explicit fixed runtime
identity rather than depending on an ambient caller-thread stack default.

This is an implementation-performance policy only. It does not alter observable
Protos semantics.

## Next slice

~~~text
NEXT_SLICE=PERF025-C2D
TYPE=IMPLEMENTATION
REPOSITORY=guillermomolina/protos

GOAL=
  reduce the shared explicit guest-carrier stack budget from 64 MiB to
  16 MiB while retaining the existing dedicated-carrier architecture,
  serialization/placement behavior, and unchanged recursive workload

CARRIER_RETIREMENT=OUT_OF_SCOPE
AMBIENT_CALLER_STACK_DEPENDENCE=OUT_OF_SCOPE
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
~~~

PERF025/#758 remains open for that implementation and its validation/publication.

## Cross references

- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#681` — historical BUG008, closed and not reopened.
- `guillermomolina/protos#760` — PLAT042.
- `guillermomolina/protos#763` — PLAT043.
- `guillermomolina/protos#766` — PLAT044.
- `guillermomolina/protos#767` — PERF026-B.
- `docs/project/evidence/PERF025/PERF025_C2B_PLAT043_BOOLEAN_OWNERSHIP_CUTOVER.md`.
- `docs/project/evidence/PERF026/PERF026_B1_PLAT044_IFTRUE_INLINE_CALLBACK.md`.
