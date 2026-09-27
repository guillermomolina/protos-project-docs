# PERF010-B — Post-PERF014 causal reconciliation

Date: 2026-09-27

## Scope

This immutable record retains the narrow causal reconciliation required by
PERF010-B / guillermomolina/protos#722 after PERF014 reached its intended
stable direct Closure-call structure but falsified its predeclared timing
prediction.

The checkpoint answers only whether the accumulated PERF012, PERF013 and PERF014
evidence changes the dependency order before Step 3 of the retained post-I072
plan. It does not reopen the general historical ~500x investigation, perform a
new cross-Truffle survey, authorize Step 4, or change observable Protos semantics.

## Exact inspection identity

```text
PRODUCT_REPOSITORY=guillermomolina/protos
PRODUCT_HEAD_REVISION=343e74eca5870d6d119612be79d968a9a8cce4c9

PARENT_WORK_ITEM=PERF010-B/#722
STEP_2_WORK_ITEM=PERF014/#725
STEP_3A_WORK_ITEM=PERF015/#726
STEP_3B_WORK_ITEM=PERF016/#727

PERF014_INTERVENTION_PRODUCT_REVISION=bcf9eda164d840b0a0b4201753fe5289347afa8a
PERF014_INTERVENTION_PRODUCT_VERSION=0.3.105-SNAPSHOT

PERF014_BENCHMARK_HARNESS_REVISION=92f5dbcf7d54ac048763f85946c973a25fbde241
PERF014_BENCHMARK_EVIDENCE_REVISION=2c722043c9638adf837ca152f682b63b2853b3aa
```

The product HEAD is later than PERF014 because unrelated work has advanced
`main`. The represented-selection questions below were re-audited against that
current HEAD rather than inferred from the PERF014 intervention revision.

## Retained plan boundary

The durable post-I072 ordering remains:

```text
compilability
  -> stable direct Closure-call selection
  -> represented primitive-family selection
  -> residual escape/capture architecture
```

The checkpoint was required by PERF014's stop gate:

```text
If the compiler shape improves but the roughly microsecond-scale per-call tax
remains nearly intact, stop and re-evaluate before Step 3.
```

The same plan also makes Step 3 independently required because fundamental
represented values remain outside I072-A's ordinary-object guarded lookup.

## Evidence consumed

Published causal evidence was limited to the already-retained campaign results:

- PERF012 / #723: frame lexical-layout construction moved off the PE-visible
  per-invocation path; the targeted too-deep-inlining mechanism was removed for
  that path.
- PERF013 / #724: captured read/write lowering uses
  `MaterializedLocalAccessor`; the retained B1 -> B2 timing result materially
  improved method-call and monomorphic-dispatch while leaving closure-call
  essentially unchanged.
- PERF014 / #725: stable direct Closure-call selection reached the intended
  Context-owned target + `DirectCallNode` architecture, including the
  definition-keyed second tier, but the retained timing discriminator was
  falsified.

PERF014 retained:

```text
DIRECT_CLOSURE_STRUCTURAL_RESULT=PASS
PERF014_PARTIAL_POSITIVE_IMPROVEMENT=OBSERVED

CLOSURE_CALL_PAIRED_CONTROL_MEDIAN=+6.0407%
GUEST_CALL_INCREMENT_RATIO_OF_MEDIANS=1.3073241187322053
CLOSURE_CALL_ORDER_EFFECT=DETECTED

PERF014_CLEARLY_MULTIPLICATIVE=NO
PERF014_TIMING_PREDICTION=FALSIFIED
PERF014_TIMING_RESULT=STRUCTURAL_SUCCESS_TIMING_PREDICTION_FALSIFIED
```

This record does not reinterpret that falsification as an implementation
failure. It treats it as the required reason to execute this checkpoint.

## Q1 — canonical Boolean represented-selection gap

Current HEAD still defines:

```text
ProtosBooleanValue implements ProtosRepresentedValue
ProtosBooleanValue.TRUE
ProtosBooleanValue.FALSE
```

Relevant implementation:

- `src/main/java/com/guillermomolina/protos/runtime/ProtosBooleanValue.java`
- `src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`

`ProtosValueLookup.lookup(..., stability)` only tracks a valid guarded lookup
dependency while the current lookup object is a `ProtosObjectValue`. When the
current object is represented and a stability assumption is present, it
invalidates that assumption before continuing through the represented
delegation parent.

Therefore a guarded lookup beginning at canonical `true` or `false` cannot
produce the stable ordinary-object guarded selection required by
`PrepareSendArguments.createGuardedStructuredSend`. The fallback remains the
generic `PrepareSendArguments.perform -> prepareSend ->
performOrdinarySendLookup -> ProtosValueLookup.lookup` route.

```text
BOOLEAN_GENERIC_REPRESENTED_SELECTION_GAP=CONFIRMED
```

## Q2 — I072-E Boolean reuse target

Current HEAD retains exactly one structured Boolean machinery:

- `ProtosStandardBooleanProtocol.StructuredCallbackKind`
- `structuredCallbackKindForCanonicalSelection`
- `PrepareSendArguments.guardedStructuredSend`
- `PreparedBooleanCall`
- the lowerer path through `IsStructuredBooleanCall`,
  `PrepareStructuredBooleanCall`, callback/immediate-result handling, and
  `FinishStructuredBooleanCallback`.

The five required canonical kinds remain:

```text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
```

`PreparedBooleanCall` already owns the exact callback selection, short-circuit,
immediate-result and Boolean-result validation behavior. No second Boolean
control implementation is required or justified.

```text
I072_E_BOOLEAN_MACHINERY=STILL_CORRECT_REUSE_TARGET
```

## Q3 — Integer represented-selection gap

Current HEAD still defines:

```text
ProtosIntegerValue implements ProtosRepresentedValue
representedDelegationParent(...) -> prelude.integerPrototype()
```

The same `lookupGuarded` admission restriction therefore applies to Integer
receivers.

The shared-driver selectors also retain the expected standard implementations:

- `>` is installed on `Number` by
  `ProtosStandardNumberOrderingProtocol` as a native Closure.
- `-` is installed on `Integer` by
  `ProtosStandardIntegerProtocol.installBinary(..., SUBTRACT)` as a native
  Closure.

In addition, `PrepareSendArguments.ordinarySendClosureOrNull` deliberately
admits only non-native ordinary Closures; native Closures remain on the exact
generic `perform` path.

Thus `count > 0` and `count - 1` still have the represented-receiver
selection gap allocated to PERF016.

```text
INTEGER_GENERIC_REPRESENTED_SELECTION_GAP=CONFIRMED
```

## Q4 — pre-Step-3 blocker check

No evidence from PERF012, PERF013, PERF014 or current HEAD establishes a new
mechanism that must precede Step 3.

Specifically:

- Step 3 has not disappeared; both represented-selection gaps remain present.
- I072-E remains available as the Boolean execution destination.
- PERF014 reached its intended structural shape; its failed prediction concerns
  timing magnitude, not a missing structural prerequisite for represented
  selection.
- No current semantic/specification contradiction prevents Step 3A.
- Step 4 remains conditional on the combined post-Step-3 residual.

```text
NEW_PRE_STEP3_BLOCKER=NONE
```

## Reconciliation matrix

| Question | Classification | Decisive current evidence |
|---|---|---|
| Q1 Boolean represented selection | `CONFIRMED` | `ProtosBooleanValue` is represented and `lookupGuarded` invalidates represented receivers |
| Q2 I072-E reuse | `STILL_CORRECT_REUSE_TARGET` | canonical Boolean kinds and `PreparedBooleanCall` remain intact |
| Q3 Integer represented selection | `CONFIRMED` | Integer is represented; `>` and `-` remain generic native sends for this admission boundary |
| Q4 new prior dependency | `NONE` | no new structural, semantic or dependency blocker is present |

## Final routing

```text
POST_PERF014_RECONCILIATION=PASS

BOOLEAN_GAP=CONFIRMED
INTEGER_GAP=CONFIRMED
NEW_PRE_STEP3_BLOCKER=NONE

PLAN_ORDER_REMAINS_VALID=YES
NEXT_STEP=PERF015/#726

PERF015_RECOMMENDATION=ACTIVATE
PERF016_REMAINS_BLOCKED_BY_PERF015=YES
STEP4_REMAINS_CONDITIONAL=YES
```

The PERF014 stop gate has therefore been honored rather than bypassed: the
campaign stopped, returned to PERF010-B, and re-audited the next dependency.
The timing prediction's falsification reduces confidence that Step 2 explains a
large share of the residual tax, but it does not remove the separately proven
Step-3 represented-selection gaps.

## PERF015 implementation boundary

PERF015 should remain bounded to Step 3A:

1. admit canonical `ProtosBooleanValue.TRUE` / `FALSE` through an exact,
   representation-aware guarded-selection mechanism;
2. keep D013 ordinary selection as authority rather than privileging selector
   spelling;
3. cover `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and` and `or`;
4. protect valid hits with exact provenance/delegation/invalidation facts;
5. avoid generic represented-value selection on an admitted valid hit;
6. reuse `structuredCallbackKindForCanonicalSelection` and the existing I072-E
   structured Boolean machinery;
7. preserve override, shadowing, delegation, non-local return, suspension and
   control behavior; and
8. retain the exact generic represented-value path as fallback.

PERF016 remains the following Step 3B work item and owns the combined post-Step-3
timing checkpoint.

## Method boundary

This checkpoint was a read-only causal reconciliation. No builds, tests,
benchmarks, Maven commands, scripts, programs, Git commands, Docker operations
or profiling were run as part of the investigation, and no new general
cross-Truffle hypothesis search was performed.
