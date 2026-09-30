# PERF010-A — Post-PERF010-B residual causal re-evaluation

Date: 2026-09-30

## Scope

This record retains the investigation-only result for PERF010-A / guillermomolina/protos#691 after PERF010-B / #722 reached its declared Step-3 stop gate.

No commands, builds, tests, benchmarks, Docker runs, repository programs, product edits, patches, or Protos repository publication were performed by this investigation.

This record does not reopen PERF010-B, does not authorize PERF010-B Step 4, does not reinterpret PERF014/PERF015/PERF016 historical observations, and does not change observable Protos semantics.

## Evidence identity

```text
WORK_ITEM=PERF010-A/#691
PARENT=PERF010/#680
SLICE=POST_PERF010_B_RESIDUAL_CAUSAL_REEVALUATION
TYPE=INVESTIGATION

PROTOS_HEAD_AT_REEVALUATION=b72778ca446b602f33af5027a0ed28ab788b39ce
PROTOS_VERSION_AT_REEVALUATION=0.3.119-SNAPSHOT
PROTOS_ALLOCATION_HEAD=7665ac7951200b68e672e6a63fbd201a5d4f6415
PROTOS_BENCHMARKS_HEAD=ac59110d23cb4724e4aa438a2a5781aaf1b31a77
PROJECT_DOCS_BASE_REVISION=4340a8cc7e75740703d6d9f596d75c5eb625df01
TOOLCHAIN=25.4.4.1.1
```

The two Protos commits after the allocation-time HEAD modify CI/toolchain-verification surfaces only:

- `.github/workflows/ci-image.yml`
- `.github/workflows/tests.yml`
- `tools/test_verify_toolchain.py`
- `tools/verify_toolchain.py`

No runtime/product implementation file changed in that interval.

## Evidence reconciled

The investigation reconciled the current live Issues and durable records, especially:

- `docs/project/evidence/PERF010-A/PERF010-A_POST_I072_FPRIME_CAUSAL_RESULT.md`
- `docs/project/evidence/PERF010-A/PERF010-A_POST_I072_GUEST_CALL_CROSS_RUNTIME_FINAL.md`
- `docs/project/evidence/PERF010-A/PERF010-A_POST_I072_RECOMMENDED_ACTION_PLAN.md`
- `docs/project/evidence/PERF010-B/PERF010-B_STEP0_COMPILER_CONFIRMATION_GATE.md`
- `docs/project/evidence/PERF010-B/PERF010-B_STEP0_FAILURE_DISCRIMINATION_AND_REMEDIATION_ROUTING.md`
- PERF012 and PERF013 compiler-remediation checkpoints
- `docs/project/evidence/PERF014/PERF014_CONTROLLED_TIMING_RESULT.md`
- PERF015 canonical Boolean represented-selection closure evidence
- `docs/project/evidence/PERF016/PERF016_POST_STEP3_CONTROLLED_TIMING_RESULT.md`
- `docs/project/evidence/PERF010-B/PERF010-B_FINAL_STEP3_ROUTING.md`
- `docs/project/evidence/PERF011/PERF011_RUNTIME_REPRESENTATION_FIT_AUDIT.md`
- `docs/project/evidence/PERF011/PERF011_GUARDED_CALL_DISCRIMINATION_CHECKPOINT.md`
- `docs/project/evidence/PERF020/PERF020_REUSABLE_COMPARATOR_INVESTIGATION.md`

## A. Reconciliation

```text
POST_STEP3_REEVALUATION=ESTABLISHED

OLD_GUEST_CALL_MODEL=WEAKENED

WHAT_PERF010_B_FALSIFIED=
  The concrete prediction that stable direct Closure-call selection plus
  canonical Boolean represented selection plus Integer represented selection
  accounted for enough of the common execution-path cost to produce a material
  timing-class improvement.

WHAT_PERF010_B_DID_NOT_FALSIFY=
  That guest-call-bearing execution still carries the common residual cost;
  that activation/context/capture/control machinery may be material;
  that required helper roots may still fail, invalidate, recompile, or compile
  into an expensive physical form;
  or that a different implementation-only mechanism within the guest-call path
  remains causally material.
```

The old broad `DOMINANT_COST_UNIT=GUEST_CALL_PATH` model is therefore not preserved as an established dominant-cause result. It remains a useful high-level association because Step 0 showed that the trivial control benefited materially from compilation while the guest-call-bearing workload was `JIT ~= INTERPRETER`. The mechanism-level model is weakened because the dependency-ordered fast-path interventions did not produce the predicted class change.

## B. Candidate elimination

```text
CANDIDATE=broad guest-call-bearing execution association
STATUS=SUPPORTED
EVIDENCE=
  PERF010-B Step 0 localized the lost JIT benefit to the guest-call-bearing
  workload rather than to the shared trivial driver alone.

CANDIDATE=direct Closure-call selection as dominant component
STATUS=FALSIFIED
EVIDENCE=
  PERF014 achieved the required stable-selection / Context-owned-target /
  DirectCallNode structural shape but falsified its predeclared clearly
  multiplicative timing prediction.

CANDIDATE=canonical Boolean represented selection as dominant component
STATUS=WEAKENED
EVIDENCE=
  The structural gap is closed, but no isolated dominant attributable fraction
  was established and the combined Step-3 result remained essentially unchanged.

CANDIDATE=Integer represented selection as dominant component
STATUS=WEAKENED
EVIDENCE=
  The structural gap is closed, but the combined Step-3 result remained
  essentially unchanged.

CANDIDATE=combined direct-Closure + Boolean + Integer fast-path model
STATUS=FALSIFIED
EVIDENCE=
  STEP_3_TIMING_CLASS=ESSENTIALLY_UNCHANGED with mixed canonical movement and
  no material consistent paired-control improvement.

CANDIDATE=PERF012 frame-layout-construction compiler failure
STATUS=FALSIFIED
EVIDENCE=
  The identified per-invocation frame-layout construction path was removed and
  its exact failure class was reported absent at PERF012 closure.

CANDIDATE=PERF013 captured-local PE-constant failure
STATUS=FALSIFIED
EVIDENCE=
  CAPTURED_READ_PE_FAILURE=ABSENT and CAPTURED_WRITE_PE_FAILURE=ABSENT after
  the MaterializedLocalAccessor cutover.

CANDIDATE=semantic/helper double Bytecode dispatch as dominant component
STATUS=WEAKENED
EVIDENCE=
  The physical semantic-wrapper -> helperTarget.call(...) structure remains at
  current HEAD, but prior causal work bounded wrapper/helper removal to a small
  percentage-scale contribution rather than a dominant recovery.

CANDIDATE=activation/context/capture physical machinery
STATUS=NOT_ESTABLISHED
EVIDENCE=
  Current source still constructs fresh invocation state, but source shape does
  not establish what remains physically allocated or compiler-visible after PE.

CANDIDATE=residual compiler bailout or incomplete mature compiled form
STATUS=SUPPORTED
EVIDENCE=
  Independent Too-deep-inlining failures remained after the exact PERF013
  captured-local failure was removed. No equivalent post-PERF014/015/016
  lifecycle discriminator exists on the current 25.4 baseline.

CANDIDATE=additional generic lookup/classification surviving hot hits
STATUS=WEAKENED
EVIDENCE=
  Several major generic-selection/classification paths were already removed by
  PERF014/015/016 without the expected timing collapse. Another source-visible
  generic branch is not selectable without compiler evidence that it survives.
```

## C. Strongest residual boundary

```text
STRONGEST_RESIDUAL_CANDIDATE=
  CURRENT_25_4_COMPILED_FORM_OF_GUEST_CALL_BEARING_HOT_ROOTS

CAUSAL_BASIS=
  The last strong lifecycle evidence established partial/asymmetric compilation:
  semantic roots reached optimized compilation while required helper Bytecode
  roots failed for concrete reasons. PERF012/PERF013 removed two exact causes,
  but independent Too-deep-inlining failures remained. Subsequent product
  fast-path changes and the 25.4 toolchain were measured for timing without a
  new source-correlated lifecycle/compiled-shape discriminator.

SEMANTICALLY_REQUIRED_WORK=
  fresh invocation semantics;
  receiver and methodHome;
  arguments and observable results;
  capture-by-reference;
  return-home/non-local-return behavior;
  Error/control propagation;
  Task/Actor/dynamic-control state;
  Context isolation;
  represented-value semantics.

IMPLEMENTATION_ONLY_WORK=
  physical allocation/materialization choices for that state;
  host collection/classification machinery surviving PE;
  cache and carrier representation;
  helper/call transport that current compiler evidence proves removable without
  altering the required semantics.

CURRENT_COMPILER_STATE=
  Historical retained state:
    semantic roots -> optimized compilation observed;
    required helper roots -> partial/asymmetric failures observed;
    PERF012 exact frame-layout failure -> removed;
    PERF013 captured-local PE-constant failures -> removed;
    independent Too-deep-inlining failures -> still observed after PERF013 B2.

  Current post-PERF014/015/016 25.4 state:
    UNKNOWN_FROM_RETAINED_EVIDENCE.
```

## Current 25.4 external implication

External GraalVM documentation is used only to interpret why the old lifecycle result cannot be promoted to a current-head fact.

GraalVM 25.4.4.1.1 release notes state that guest-language inlining now uses Graal IR call-site frequencies rather than runtime direct-call counters:

https://www.graalvm.org/release-notes/25.4/

Current Truffle options still expose compilation tracing, inlining budgets and recursive-inlining depth, including `engine.TraceCompilation`, `InliningExpansionBudget`, `InliningInliningBudget`, and `InliningRecursionDepth`:

https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Options/

Therefore the old Too-deep-inlining observation remains a technically plausible candidate class, but it must be re-observed on current 25.4 before it can select a production intervention.

## D. Next action

```text
NEXT_STATE=MEASUREMENT_REQUIRED

NEXT_CAUSAL_TARGET=
  exact current-25.4 compiler lifecycle and surviving compiled shape of the
  guest-call-bearing helper/semantic roots used by the common driver

NEXT_SLICE=
  POST_PERF010_B_25_4_HOT_ROOT_COMPILER_LIFECYCLE_DISCRIMINATOR

NEXT_SLICE_KIND=INVESTIGATION

NEXT_IMPLEMENTATION_REPOSITORY=NONE

CONTROL_REVISION=NOT_YET_SELECTABLE
CONTROL_VERSION=NOT_YET_SELECTABLE

CONTROL_SELECTION_REASON=
  No product intervention is selected yet. The current Protos HEAD is the exact
  next measurement baseline, but the future before/after CONTROL is pinned only
  after the discriminator identifies a concrete product intervention.
```

The smallest discriminating measurement must correlate the exact current source roots for:

- shared repeat/control driver;
- `micro/closure-call`;
- `micro/method-call`;
- `runtime/monomorphic-dispatch`.

For each relevant semantic/helper root it must establish:

```text
OPT_DONE vs OPT_FAILED
tier reached
permanent bailout reason/stack
recompilation/invalidation lifecycle
exact source/root identity
```

The same bounded experiment must retain the existing normal-compilation versus `Compilation=false` discriminator where useful to preserve the causal relation between compiler state and workload cost.

Routing is predeclared:

```text
IF required current 25.4 roots still OPT_FAIL / permanently bail out /
repeatedly invalidate:
  exact current failure becomes the next causal boundary.

ELSE IF roots stably OPT_DONE but compiler diagnostics show substantial
activation/context/helper/generic machinery survives:
  that surviving machinery becomes eligible for one bounded causal ablation.

ELSE IF roots stably OPT_DONE and the suspect machinery is eliminated:
  the old PE-failure guest-call model is further weakened and no implementation
  may be authorized from it.
```

Timing is not the first admission criterion for this slice. The first question is what 25.4 actually compiles.

Existing `guillermomolina/protos-benchmarks` infrastructure is sufficient as the starting point: PERF010-A source-identity diagnostics, PERF010-B Step-0 compiler tracing, and PERF013 compiler-gate machinery already provide the needed lineage. PERF020 does not need to be implemented to perform this bounded discriminator.

## E. PERF011 routing

```text
PERF011_REACTIVATE=NO

PERF011_RATIONALE=
  The residual has not yet been attributed to a representation/compiler-
  visibility surface owned by PERF011. Reactivating the broad audit now would
  choose candidates from static attractiveness. Reactivate only if the current
  25.4 discriminator establishes a material representation/compiler-visibility
  mechanism or a concrete optimization decision needs one of PERF011's pending
  compiler-visibility questions.
```

## F. PERF020 routing

```text
PERF020_IMPLEMENTATION_DEFERRED=YES

PERF020_TRIGGER_STATE=NOT_READY

PERF020_IMPLEMENTATION_REPOSITORY=
  guillermomolina/protos-benchmarks
```

There is no selected and published product intervention endpoint yet, so the demand-driven PERF020 implementation trigger has not fired.

Intended ordering remains:

```text
25.4 causal discriminator
-> select exact bounded intervention, if any
-> pin CONTROL
-> implement + validate + publish INTERVENTION
-> implement PERF020 comparator
-> QUICK
-> REFERENCE if warranted
```

## G. Existing campaign boundaries

```text
PERF010_B_REOPENED=NO
PERF010_B_STEP4_AUTHORIZED=NO
HISTORICAL_EVIDENCE_MUTATION=NO
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

## Final recommendation

```text
FINAL_RECOMMENDATION=
  Do not implement another Protos optimization yet.

  Perform one source-correlated current-25.4 compiler-lifecycle/compiled-shape
  discriminator against the current 0.3.119-SNAPSHOT product baseline.

  Its purpose is to decide whether the remaining guest-call cost is currently:
    - a compiler failure;
    - unstable compilation;
    - expensive implementation machinery surviving successful compilation; or
    - already-mature compiled semantic work.

  Only that result may select the next production intervention.
```
