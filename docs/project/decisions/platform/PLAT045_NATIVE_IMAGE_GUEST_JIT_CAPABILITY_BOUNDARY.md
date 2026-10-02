# PLAT045 — Native Image guest-JIT capability boundary under upstream Bytecode DSL limitation

Status: **RATIFIED**

Selected architecture: **Candidate B — Native Image supported, guest JIT unavailable while the upstream Bytecode DSL Native runtime-compilation limitation remains active; restore optimizing Native guest JIT only after the objective upstream/revalidation gate passes**.

Approval: explicit project-owner approval on 2026-10-02:

~~~text
apruebo la "Native Image supported, guest JIT unavailable" con vuelta atras cuando se soluucione upstream.
~~~

Decision Issue: `guillermomolina/protos#772`

Triggering defect: `BUG013 / guillermomolina/protos#749`

Upstream authority: `oracle/graal#14579`

Product baseline at ratification:

~~~text
PROTOS_REVISION=b91d6066eb41b420db4a2884ac4719b2468767fa
PROTOS_VERSION=0.3.135-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative runtime/build capability decision. Observable
Protos language and Standard Library semantics remain unchanged.

## Decision

PLAT045 explicitly narrows the Native guest-runtime portion of PLAT038 while the
upstream generated-Bytecode-DSL Native Tier-2 defect remains active.

The selected support matrix is:

~~~text
JVM_RUNTIME=OPTIMIZING_TRUFFLE_RUNTIME
JVM_GUEST_JIT=SUPPORTED_AND_REQUIRED

NATIVE_IMAGE_SUPPORT=YES
NATIVE_HOST=AOT_GRAALVM_NATIVE_IMAGE
NATIVE_RUNTIME=INTERPRETER_ONLY_FALLBACK
NATIVE_GUEST_JIT=UNAVAILABLE
NATIVE_GUEST_JIT_REASON=UPSTREAM_ORACLE_GRAAL_14579

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~

The Native executable remains a real GraalVM Native Image AOT executable. The
selected limitation concerns Truffle guest runtime compilation inside that
executable, not host AOT compilation.

PLAT045 therefore requires user-facing and admission surfaces to distinguish:

~~~text
NATIVE_IMAGE_AOT_HOST=SUPPORTED
PROTOS_GUEST_EXECUTION=SUPPORTED
PROTOS_GUEST_RUNTIME_COMPILATION=UNAVAILABLE
~~~

No Native path may report failed or disabled guest compilation as successful
guest JIT.

## Upstream boundary

`oracle/graal#14579` establishes a minimal generated Truffle Bytecode DSL root
that:

- uses `enableYield=false`;
- declares no custom operations;
- has no locals;
- has no loops;
- contains no Protos code;
- contains no `boxingEliminationTypes` configuration; and
- simply returns constant `1`.

The current upstream evidence is:

~~~text
UPSTREAM_MIN_REPRO_JVM_TIER2=PASS
UPSTREAM_MIN_REPRO_NATIVE_TIER2=FAIL_FRAME_WITHOUT_BOXING

BOXING_ELIMINATION_REQUIRED_FOR_FAILURE=NO
PROTOS_LOCALS_REQUIRED_FOR_FAILURE=NO
PROTOS_CONTINUATIONS_REQUIRED_FOR_FAILURE=NO
PROTOS_SEMANTICS_REQUIRED_FOR_FAILURE=NO
~~~

The issue was also reproduced on the recorded GraalVM developer snapshot:

~~~text
GRAALVM_CE=25.5.5-dev+1.1
TRUFFLE_GRAAL_MAVEN=25.5.5-SNAPSHOT
DEVELOPER_BUILD=25.5.5-dev-20260930_0129
ORACLE_GRAAL_REVISION=11b21fb2e5691e49d46525dac7b87ef33aeafe82
~~~

At ratification, the upstream Issue remains open and no upstream fix is treated
as available authority.

## Supported fallback runtime

GraalVM documents the fallback Truffle runtime as an interpreter-only runtime and
documents `-Dtruffle.UseFallbackRuntime=true` as a supported runtime selection,
including Native Image construction.

PLAT045 adopts that upstream-supported mode for the Native artifact while the
re-enable gate below remains closed.

The already-reported Protos feasibility candidate established:

~~~text
NATIVE_BUILD=PASS
NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_GUEST_SMOKE=PASS
NATIVE_TEST_TOOL_SMOKE=PASS
NATIVE_DAP_STACKTRACE_REGRESSION=PASS

NATIVE_INTERPRETER_ONLY_STATUS=0
NATIVE_FALLBACK_RUNTIME_MARKERS=1

OPT_DONE=0
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0

NATIVE_INTERPRETER_ONLY=PASS
NATIVE_GUEST_JIT=UNSUPPORTED_UPSTREAM_ORACLE_GRAAL_14579
NATIVE_REGRESSION_SUITE=PASS
~~~

This feasibility evidence did not itself authorize the policy. The owner approval
above is the ratification authority.

## PLAT038 explicit delta

PLAT038 remains historical authority for its original selection. PLAT045 does not
rewrite that record.

Exactly these Native guest-runtime invariants are amended while the re-enable
gate is closed:

~~~text
PLAT038:
  TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
PLAT045:
  TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=UNAVAILABLE_NOT_REQUIRED_WHILE_GATE_CLOSED

PLAT038:
  GUEST_JIT_BEHAVIOR=REQUIRED_AND_VERIFIED
PLAT045:
  NATIVE_GUEST_JIT=UNAVAILABLE_UPSTREAM_ORACLE_GRAAL_14579

PLAT038:
  INTERPRETER_ONLY_NATIVE_ACCEPTABLE=NO
PLAT045:
  INTERPRETER_ONLY_NATIVE_ACCEPTABLE=YES_WHILE_GATE_CLOSED

PLAT038:
  GUEST_TRUFFLE_JIT_REMAINS_REQUIRED_IN_NATIVE=YES
PLAT045:
  GUEST_TRUFFLE_JIT_REMAINS_REQUIRED_IN_NATIVE=NO_WHILE_GATE_CLOSED
~~~

This is an explicit owner-approved replacement consequence, not a reinterpretation
of the old wording.

## PLAT038 invariants preserved

The following PLAT038 architecture remains unchanged:

~~~text
DUAL_JVM_NATIVE_MODEL=PRESERVED
JVM_DEVELOPMENT_AUTHORITY=PRESERVED
JVM_OPTIMIZING_RUNTIME=PRESERVED
JVM_GUEST_TIER2_ADMISSION=PRESERVED

BYTECODE_DSL_PRODUCTION_ARCHITECTURE=PRESERVED
I075_BOXING_ELIMINATION=PRESERVED

NATIVE_IS_REAL_AOT_EXECUTABLE=PRESERVED
CANONICAL_GRAAL_TRUFFLE_AUTHORITY=PRESERVED
PRIVATE_PATCHED_GRAAL_TOOLCHAIN=NOT_SELECTED

NATIVE_BUILD_IS_NOT_DISTRIBUTION=PRESERVED
NATIVE_BUILD_IS_NOT_RELEASE=PRESERVED
CHANGELOG_IMPLIES_RELEASE=NO
EXACT_CHECKOUT_VALIDATION=PRESERVED
STAGE0_STAGE1_SEPARATION=PRESERVED
DIST001_RELEASE_ISOLATION=PRESERVED

OBSERVABLE_PROTOS_SEMANTICS=PRESERVED
~~~

No Protos semantic, Standard Library, Context, Task, Actor, Process, continuation,
or debugger semantic change is authorized by PLAT045.

## Native admission while the gate is closed

The maintained Native admission must test the selected capability rather than
pretend to test optimizing execution that the selected runtime deliberately does
not provide.

Required Native properties include:

~~~text
NATIVE_IMAGE_BUILD=PASS
NATIVE_RUNTIME_MODE=INTERPRETER_ONLY_FALLBACK
NATIVE_FALLBACK_RUNTIME_MARKERS=EXPECTED
NATIVE_GUEST_JIT_CAPABILITY=UNAVAILABLE

NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_GUEST_SMOKE=PASS
NATIVE_CONFORMANCE=PASS
NATIVE_TEST_TOOL=PASS
REQUIRED_NATIVE_DAP_TOOLING_REGRESSIONS=PASS

OPT_DONE=0_EXPECTED
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
~~~

An unexpected attempt to perform guest runtime compilation is not success under
this contract. Unknown compiler/internal failures remain fail-closed.

The JVM lane continues to prove optimizing guest compilation independently.

## Objective re-enable gate

Optimizing Native guest JIT is restored only when the exact candidate runtime
plane intended for Protos satisfies all applicable gates below:

~~~text
PUBLIC_UPSTREAM_FIX_OR_EQUIVALENT_SUPPORTED_FIX_AVAILABLE
AND
UPSTREAM_MIN_REPRO_JVM_TIER2=PASS
AND
UPSTREAM_MIN_REPRO_NATIVE_TIER2=PASS
AND
PROTOS_NATIVE_HELPER_BYTECODE_ROOT_TIER2=PASS
AND
PROTOS_NATIVE_SEMANTIC_BYTECODE_ROOT_TIER2=PASS
AND
OPT_FAILED=0
AND
FRAME_WITHOUT_BOXING_FAILURES=0
AND
COMPILATION_FAILURES=0
AND
REQUIRED_NATIVE_CONFORMANCE=PASS
AND
NATIVE_TEST_TOOL=PASS
AND
REQUIRED_NATIVE_DAP_TOOLING_REGRESSIONS=PASS
~~~

When available in the adopted Graal/Polyglot baseline, the Native runtime
capability probe should additionally confirm that the intended optimizing
configuration reports guest compilation support rather than fallback-only
execution.

Closing `oracle/graal#14579` by itself is not sufficient. The objective
reproducer and maintained Protos gates are the admission authority.

Once this gate passes, Protos returns to the optimizing PLAT038 Native capability
without changing Protos semantics or the fundamental JVM/native build model.

## Candidate disposition

### Candidate A — retain PLAT038 unchanged

Rejected for the current upstream interval. It would block all supported Native
consumption despite an upstream-supported interpreter-only runtime and a clean
local Protos feasibility result.

### Candidate B — temporarily support Native interpreter-only

**Selected.**

It is the smallest architecture that preserves Native AOT consumption and
correct Protos execution without pretending guest JIT exists, weakening JVM
optimization, changing Protos semantics, or taking ownership of Graal/SVM
internals.

### Candidate C — dual Native supported/blocked capability artifacts

Rejected as unnecessary duplication. The optimizing variant cannot currently
pass admission, so maintaining a separate product/build/support surface adds cost
without providing a working additional capability.

### Candidate D — Protos-controlled patched Graal/SVM toolchain

Rejected for current need. Owning runtime-compilation internals would add version
skew, security-update, reproducibility, CI, portability and distribution burden
without a demonstrated present Native-only throughput requirement that justifies
that ownership.

### Candidate E — alter Protos/Bytecode DSL features

Rejected by the minimum upstream reproducer. Removing I075 boxing elimination,
locals, yield/continuations or Protos-specific semantics does not remove the
minimal failure.

### Candidate F

No materially distinct evidence-backed supported architecture was established.

## Performance and support consequence

PLAT045 does not claim equal performance between the JVM optimizing runtime and
Native fallback runtime.

~~~text
JVM_STEADY_STATE_OPTIMIZATION=PRESERVED
NATIVE_STARTUP_HOST_AOT_BENEFIT=PRESERVED_IN_ARCHITECTURE
NATIVE_GUEST_STEADY_STATE_THROUGHPUT=EXPECTED_TO_BE_LOWER_THAN_OPTIMIZING_NATIVE
EXACT_PROTOS_PERFORMANCE_DELTA=NOT_ESTABLISHED_BY_PLAT045
~~~

A future concrete requirement for Native-only, long-lived, throughput-sensitive
production execution that cannot use the JVM is a valid trigger to reconsider
the selected temporary boundary before upstream repairs the defect.

## GITHUB021 invariant consistency

~~~text
PLAT038_NATIVE_GUEST_JIT_INVARIANTS=EXPLICITLY_REOPENED
OWNER_APPROVED_REPLACEMENT_CONSEQUENCES=YES

ALL_OTHER_PLAT038_INVARIANTS=PRESERVED
NEW_PROTOS_LANGUAGE_SEMANTICS=NO
SPECIFICATION_CHANGE=NO

DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Implementation authority

The immediate next slice remains inside the existing technical defect/admission
owners:

~~~text
NEXT_SLICE=BUG013-F
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

BUG013=#749
TEST006=#755
DIST009=#743
~~~

BUG013-F may implement the ratified fallback runtime selection and synchronize
the maintained Native admission with PLAT045. TEST006 remains the admission-policy
authority for the resulting Native regression gate. DIST009 must remain blocked
until the implementation and required Native validation are green.

No private Graal patch, dependency upgrade, Protos semantic change, release,
tag, asset publication, or unrelated Native redesign is authorized by this
decision.

## Evidence and references

- `guillermomolina/protos#772` — PLAT045 decision Issue.
- `guillermomolina/protos#749` — BUG013 technical defect.
- `guillermomolina/protos#755` — TEST006 Native admission policy.
- `guillermomolina/protos#743` — DIST009 release owner.
- `oracle/graal#14579` — minimal upstream Native Tier-2 failure.
- `docs/project/decisions/platform/PLAT038_NATIVE_IMAGE_BOOTSTRAP_RUNTIME_RELEASE_BOUNDARY.md`.
- `docs/project/evidence/PLAT045/PLAT045_NATIVE_IMAGE_GUEST_JIT_CAPABILITY_DECISION_EVIDENCE.md`.

## Published implementation checkpoint — BUG013-F

The first product implementation of this ratified capability boundary was published
in `guillermomolina/protos` at:

~~~text
PROTOS_REVISION=7c16cec611c3cf5e504d9271c6964a32656ba32f
PROTOS_VERSION=0.3.137-SNAPSHOT
COMMIT_SUBJECT=BUG013-F: Native fallback runtime per PLAT045
~~~

The published slice:

~~~text
NATIVE_PROFILE_USE_FALLBACK_RUNTIME=YES
NATIVE_BUILD_ARG=-Dtruffle.UseFallbackRuntime=true

JVM_OPTIMIZING_RUNTIME_POLICY=PRESERVED
JVM_LAUNCHER_FALLBACK_RUNTIME=NO

NATIVE_FORCED_GUEST_JIT_SUCCESS_CLAIM=REMOVED
NATIVE_INTERPRETER_ONLY_POLICY=EXPLICIT
NATIVE_GUEST_JIT=UNSUPPORTED_UPSTREAM_ORACLE_GRAAL_14579

UNKNOWN_OPT_FAILURES_FAIL_CLOSED=YES
FRAME_WITHOUT_BOXING_FAILURES_FAIL_CLOSED=YES
COMPILATION_FAILURES_FAIL_CLOSED=YES

NATIVE_TEST_TOOL_GATE=PRESERVED
NATIVE_DAP_GATE=PRESERVED
~~~

Current-facing README/Native Makefile text was updated to reflect the selected
capability matrix. Historical evidence was not broadly rewritten.

At publication time no workflow run, commit-status context, or live-Issue
human-reported exact-revision validation was observed for this SHA. Therefore the
implementation checkpoint is revision-bound but **not yet admission-complete**:

~~~text
BUG013_F_CODE_PUBLICATION=COMPLETE
BUG013_F_EXACT_REVISION_VALIDATION=PENDING

BUG013=#749 REMAINS_OPEN
TEST006=#755 REMAINS_OPEN
DIST009=#743 REMAINS_BLOCKED

NEXT_SLICE=TEST006-C
NEXT_SLICE_TYPE=IMPLEMENTATION_VALIDATION
NEXT_SLICE_REPOSITORY=guillermomolina/protos
~~~

Detailed evidence:
`docs/project/evidence/BUG013/BUG013_F_PLAT045_NATIVE_FALLBACK_RUNTIME_IMPLEMENTATION.md`.

