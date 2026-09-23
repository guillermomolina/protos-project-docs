# PERF011 guarded-call discrimination checkpoint

## Evidence identity

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
BENCHMARK_EVIDENCE_REVISION=1611f9b19005a13723cc9347addcbb74e926290d
PROJECT_CAUSAL_RECORD_REVISION=2d08823efe25a890ef4a07faf2fc266d080f9e14
PROJECT_CAUSAL_RECORD=docs/project/evidence/PERF010-A/PERF010-A_GUARDED_CALL_CAUSAL_RESULT.md
```

This checkpoint follows the source-level PERF011 runtime-representation fit audit with the completed guarded-call experiment shared with PERF010-A.

## Result

```text
PERF011_RUNTIME_FIT=INCONCLUSIVE
CALL_SITE_SPECIALIZATION_EXPERIMENT=VALID
CALL_SITE_TIMING_RESULT=NEGATIVE
COMPILER_VISIBILITY_CHANGED=INCONCLUSIVE
MATERIAL_RUNTIME_FIT_MISMATCH=NOT_ESTABLISHED
PRODUCTION_REPRESENTATION_CHANGE_JUSTIFIED=NO
PERF011_BLOCKS_PERF010=NO
PERF011_CAN_CLOSE=NO
```

The guarded post-D013 intervention was semantically valid and successfully bypassed generic preparation and structured-dispatch machinery while retaining authoritative lookup and exact generic fallback. On the two workloads that exercise the patched call site, however, the guarded image was slower: approximately 9.3% for `micro/method-call` and 7.7% for `runtime/monomorphic-dispatch`. The two indirect workloads moved only about 1.5–2.2% in the opposite direction.

This result does not establish that the source-level representation mismatch candidate is immaterial in general. The intervention itself adds guards and changes specialization structure, and the compiler-visible consequence was not established.

BGV dumps exist for baseline and guarded images, but their graph content was not extracted and compared. Near-equal file sizes are only a weak proxy and must not be promoted to structural evidence. Target visibility, direct invocation, survival of generic classification/structured dispatch, and any inlining change therefore remain `INCONCLUSIVE`.

The negative timing result is sufficient to avoid selecting this guarded bypass as a production optimization. It is not necessary to complete the compiler-graph comparison before PERF010-A continues to its next causal candidate. PERF011 remains an independent open investigation; a future bounded graph comparison is useful only if the runtime-fit question is resumed and the result would affect a concrete next decision.

No Shape/DynamicObject/Frame migration, Closure representation redesign, lookup invalidation redesign, or other broad runtime change is justified by this checkpoint.

## Pause and reactivation contract

PERF011 is deliberately paused while PERF010-A continues dominant common-overhead attribution. This is a scheduling relationship, not a native blocker relationship: PERF011 does not block PERF010/#680 or PERF010-A/#691, and PERF010 does not need PERF011 to complete before continuing.

```text
PERF011_STATUS=PAUSED
PERF011_PAUSE_REASON=NO_MATERIAL_RUNTIME_FIT_MISMATCH_ESTABLISHED; CONTINUE_PERF010_DOMINANT_OVERHEAD_ATTRIBUTION_FIRST
PERF011_BLOCKS_PERF010=NO
PERF010_BLOCKS_PERF011=NO

PERF011_REACTIVATION_TRIGGER=
  PERF010_EVIDENCE_IDENTIFIES_MATERIAL_COMMON_PATH_COST_PLAUSIBLY_CAUSED_BY_RUNTIME_REPRESENTATION_OR_COMPILER_VISIBILITY
  OR
  CONCRETE_OPTIMIZATION_DECISION_REQUIRES_RESOLVING_REMAINING_COMPILER_VISIBILITY_QUESTION
```

Absent either trigger, do not resume broad PERF011 runtime-fit work merely to eliminate the current `INCONCLUSIVE` classification. In particular, the pending BGV graph comparison is not independently sufficient reason to reactivate PERF011; it should be performed when its result can discriminate a concrete PERF010-derived candidate or optimization decision.
