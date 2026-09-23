# PERF011 — Runtime-representation fit audit checkpoint

Status: INCONCLUSIVE; BOUNDED CAUSAL EXPERIMENT REQUIRED

This durable, non-normative record retains the read-only PERF011 / #693 audit checkpoint performed after the PERF010-A guarded monomorphic call-path architecture investigation. The purpose was to determine whether current Protos semantic invariants are represented to Truffle/Graal strongly enough for effective specialization, and whether that broader question should change the next PERF010 step.

No Protos source, benchmark, test, version, changelog, or normative specification was modified by this audit.

## Evidence identity

```text
PROTOS_REVISION=2b3a88389da7228caed231a90b14091cf2841115
PROTOS_VERSION=0.3.77-SNAPSHOT
PREVIOUS_PERF010A_RECORD=docs/project/evidence/PERF010-A/PERF010-A_GUARDED_MONOMORPHIC_CALL_PATH_ARCHITECTURE.md
```

The revision above is the current Protos `main` inspected for this checkpoint. The intervening head change is governance-only (`AGENTS.work/IMPLEMENTATION.md`); the runtime representation evidence inspected remains the current source state.

## Result

```text
PERF011_RUNTIME_FIT=INCONCLUSIVE
PERF010_CONTINUATION=BOUNDED_GUARDED_CALL_EXPERIMENT_REQUIRED

CALL_SITE_SPECIALIZATION=STRONG_REPRESENTATION_MISMATCH_CANDIDATE
OBJECT_REPRESENTATION=PLAUSIBLE_MISMATCH_MATERIALITY_UNMEASURED
LOOKUP_INVALIDATION=NO_GENERAL_SHAPE_VERSION_INVALIDATION
CLOSURE_REPRESENTATION=STABLE_IMPLEMENTATION_FACTS_PRESENT
ACTIVATION_REPRESENTATION=MIXED_SEMANTIC_AND_REPRESENTATION_COST
COMPILER_VISIBILITY=SOURCE_EVIDENCE_INSUFFICIENT

NEW_DECISION_REQUIRED=NO
PERF011_CAN_CLOSE=NO
PERF010_CAN_CONTINUE_UNCHANGED=NO
```

The audit found concrete source-level representation mismatches and missed specialization opportunities, but source inspection alone cannot establish that they survive partial evaluation or account for a material fraction of the measured common-path overhead. The correct next step is therefore a bounded compiler-visible causal experiment, not a broad runtime representation migration.

## Representation matrix

| Surface | Semantic invariant | Current representation | Compiler-facing assessment | PERF010 intersection |
| --- | --- | --- | --- | --- |
| Ordinary object slots | exact D013 lookup, shadowing, delegation and mutation visibility | `LinkedHashMap<String,Object>` local slots | no layout/shape/version fact is represented explicitly | direct/indirect |
| Delegation parent | parent identity is immutable | final `Object parent` | individual parent identity is stable; chain/layout invalidation is not represented | indirect |
| OPEN/CLOSED/FROZEN | FROZEN is stronger than CLOSED; CLOSED still permits replacement | mutable enum plus the same generic slot map | stronger frozen stability exists semantically than the generic representation expresses | indirect |
| Selected Closure | selected implementation identity is stable for the selected Closure | `ProtosClosureValue` with final implementation-related fields plus separate execution-plan/projection state | guardable stable facts exist, but invocation still enters generic preparation/classification | direct |
| Receiver / methodHome | authoritative lookup determines the effective receiver/home relation | runtime object values transported into fresh activation | safe post-lookup specialization seam is already established | direct |
| Activation | fresh invocation/context and control state remain semantically live | heap `ProtosActivation` plus generic runtime structures | mixed: some allocation/state is semantic, some representation cost remains unclassified | direct |
| Call target | monomorphic site may stabilize effective Bytecode target | generic preparation reaches a late target guard/direct call | whether target becomes PE-constant early enough requires compiler evidence | direct |

## Object representation

`ProtosObjectValue` currently stores ordinary local slots as:

```text
LinkedHashMap<String,Object>
```

with a final delegation parent and mutable `MutationState`.

This representation correctly implements the semantic model, but it does not expose a Shape-style layout/version identity. The language already supplies useful stability:

- delegation parent identity is immutable;
- FROZEN objects reject creation, replacement, and removal;
- CLOSED objects reject structural creation/removal but still allow replacement;
- selector names at many call sites are compile-time constants.

This is a plausible representation mismatch, especially for frozen or otherwise guarded paths. It is **not yet a measured material mismatch**. A DynamicObject/Shape migration is therefore not authorized by this checkpoint.

## Lookup and invalidation

The prior PERF010-A audit established that current ordinary lookup has no general slot-shape/version invalidation facility based on `Assumption`, `CyclicAssumption`, `@CompilationFinal`, or an equivalent ordinary-object mechanism.

Consequently:

```text
ARBITRARY_MUTABLE_LOOKUP_CAN_BE_SKIPPED=NO
GUARDED_BOUNDED_LOOKUP_SPECIALIZATION=YES
FULLY_FROZEN_RELEVANT_PATH_CAN_BE_STABLE=YES
```

This is an implementation-representation limitation rather than a language-semantic requirement. Exact D013 mutation visibility still prevents pretending mutable lookup is stable.

## Call-site specialization

This is the strongest direct PERF010/PERF011 intersection.

PERF010-A already established the safe specialization seam immediately after authoritative D013 lookup has selected:

```text
selected ProtosClosureValue
selected methodHome
```

The generic path then performs repeated implementation/category preparation and a structured-dispatch predicate universe before the final ordinary invocation arm. The retained architecture audit counted 16 preparation classifiers and 19 structured-call predicates.

For a guarded monomorphic ordinary composed call, those classifications are not semantically required on a cache hit when the selected Closure and provenance guards remain valid. Exact current generic behavior remains available on guard failure.

This is strong source-level evidence of an implementation representation/specialization opportunity. What remains unknown is whether Graal already removes enough of this machinery in the baseline compiled graph that the runtime-level redundancy is cheap.

## Closure representation

`ProtosClosureValue` contains multiple stable facts in final fields, including definition/capture information, receiver/home relationships, prelude/native implementation references, while execution-plan/context-local projection state is carried separately.

The important distinction is:

```text
fresh Closure identity != unstable implementation category
```

Protos semantics may require fresh extracted Closure identity, but that does not imply that every invocation must rediscover whether the selected implementation is an ordinary composed Closure, a native structured implementation, or another implementation category.

This supports call-site guarding without weakening Closure freshness semantics.

## Activation representation

The audit does not establish activation construction as removable overhead.

Fresh activation/context, receiver and `methodHome`, arguments, captured lexical relations, return-home state, Actor/module/execution-domain state, and Task/dynamic-control propagation remain semantically relevant under the current specification and implementation architecture.

Therefore activation is classified as a mixed surface:

```text
SEMANTIC_ACTIVATION_COST=YES
ADDITIONAL_REPRESENTATION_COST=POSSIBLE_NOT_ATTRIBUTED
```

A Frame/indexed-location/scalar-replacement investigation remains a possible later PERF011 surface, not the first causal experiment.

## Compiler visibility limitation

This checkpoint deliberately does **not** claim that source-level generic structures survive in the optimized graph.

Source inspection cannot establish:

- which `Map`, `String`, `Optional`, collection-copy or generic helper operations survive partial evaluation;
- whether the effective call target becomes compiler-constant at the relevant point;
- whether generic structured/native classification remains in the hot compiled arm;
- how much timing is recoverable from changing the representation.

Therefore:

```text
MATERIAL_RUNTIME_FIT_MISMATCH=NOT_ESTABLISHED
COMPILER_DIAGNOSTIC_REQUIRED=YES
```

This is why PERF011 cannot close from this audit alone.

## Relationship to previous PERF010-A ablations

The earlier semantic/helper and other bounded ablations remain valid causal evidence for the exact mechanisms removed by those experiments.

This checkpoint cannot yet distinguish whether their small measured contributions are:

1. independent local overheads; or
2. downstream symptoms of a broader failure to expose stable runtime facts to Graal.

They must not be reinterpreted as proof of a representation mismatch without the bounded compiler-visible intervention below.

## First causal experiment

The first discriminating experiment should reuse the already-established PERF010-A guarded call seam rather than create a separate broad PERF011 migration.

```text
one constant-selector local-method monomorphic composed send

baseline:
  authoritative D013 lookup
  -> generic preparation/classification
  -> structured-dispatch universe
  -> ordinary invocation

diagnostic hit:
  authoritative D013 result / bounded valid guards
  -> selected Closure + methodHome
  -> preserve exact fresh activation/control semantics
  -> bypass generic preparation/classification
  -> direct ordinary prepared-call path

miss:
  -> exact current generic path
```

Required evidence:

1. semantic mutation/shadowing/receiver/`methodHome` regressions;
2. non-local-return and applicable suspension/control regressions;
3. compiler diagnostics showing whether the generic machinery is absent from the specialized hot arm and whether target visibility improves;
4. paired timing on at least `micro/method-call` and `runtime/monomorphic-dispatch`;
5. exact Protos and benchmark/harness revisions.

Interpretation:

- compiler-graph simplification plus material controlled timing reduction: evidence that implementation representation/specialization is materially contributing to PERF010;
- compiler-graph simplification without material timing reduction: real mismatch, but not a Pareto-leading PERF010 cause;
- baseline compiler already removes the machinery: evidence against this representation hypothesis as the dominant explanation.

## Decision boundary

No new Dxxx/PLATxxx decision is required to run the bounded diagnostic experiment because it preserves current observable semantics and retains the exact generic fallback.

A future wholesale object representation, Shape model, activation/frame architecture, or durable runtime protocol migration may require a separate architecture decision if it changes durable implementation architecture. This checkpoint does not select or authorize such a migration.

## Reconciled state

```text
PERF011_RUNTIME_FIT=INCONCLUSIVE
MATERIAL_RUNTIME_FIT_MISMATCH=NOT_ESTABLISHED
STRONGEST_CANDIDATE=post-D013 monomorphic Closure call representation/specialization
SECONDARY_CANDIDATE=ordinary slot/layout stability and invalidation representation
TERTIARY_CANDIDATE=activation/lexical-state representation

FIRST_CAUSAL_EXPERIMENT=guarded constant-selector local-method monomorphic composed-send bypass plus compiler diagnostics and paired timing
DIAGNOSTIC_CODE_CHANGE_REQUIRED=YES
NEW_DECISION_REQUIRED=NO

PERF011_CAN_CLOSE=NO
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
```

The next bounded experiment should be owned by PERF010-A / #691 because it is already that Issue's selected causal intervention. PERF011 consumes its compiler-visible result to determine whether a broader representation-fit workstream is justified; it should not duplicate the experiment independently.

## Materially inspected sources

- `guillermomolina/protos:AGENTS.md`
- `guillermomolina/protos:AGENTS.work/PERFORMANCE.md`
- `guillermomolina/protos:AGENTS.work/COORDINATION.md`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/runtime/ProtosObjectValue.java`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/runtime/ProtosValueLookup.java`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java`
- `guillermomolina/protos:src/main/java/com/guillermomolina/protos/execution/ProtosClosureInvoker.java`
- `guillermomolina/protos-project-docs:docs/project/evidence/PERF010-A/PERF010-A_GUARDED_MONOMORPHIC_CALL_PATH_ARCHITECTURE.md`
- live GitHub Issues #680, #691 and #693.

All repository coordinates above intentionally use `guillermomolina` with the canonical double-`mo` spelling.


## Semantic-to-compiler polymorphism fidelity scope expansion — 2026-09-23

PERF011 now explicitly incorporates a cross-language diagnostic lens derived
from the current PERF010-A evidence and Smalltalk-on-Truffle precedent.

This does **not** allocate a new PERF item. The question remains the one already
owned by PERF011:

> Does the Protos runtime representation expose stable semantic facts to
> Truffle/Graal in a form that permits effective specialization?

The additional discriminator is:

> Does the polymorphism and dynamism observed by Truffle/Graal faithfully
> reflect the semantic polymorphism and dynamism of the Protos program, or does
> the runtime representation manufacture accidental polymorphism/megamorphism?

### First established Protos mismatch

PERF010-A has established the first concrete case.

At:

```text
PROTOS_REVISION=3a4afc27a96ae4efd6f3f0c71bc0990d5d102e30
```

the relevant `micro/method-call` send is semantically monomorphic, while
`PrepareSendArguments.fastOrdinarySend` consumes distinct DSL cache entries
because fresh `Closure` and `methodHome` object identities participate in
the specialization identity.

```text
SEMANTIC_SELECTOR_COUNT=1
SEMANTIC_EXECUTABLE_BEHAVIOR=STABLE
SEMANTIC_CALL_SITE=MONOMORPHIC

CLOSURE_OBJECT_IDENTITY=CHANGES
METHOD_HOME_OBJECT_IDENTITY=CHANGES

TRUFFLE_FAST_SPECIALIZATION_INSTANCES=1 -> 2 -> 3
CACHE_LIMIT=3
NEXT_STATE=GENERIC_REPLACING_SPECIALIZATION

POLYMORPHISM_FIDELITY=MISMATCH
ACCIDENTAL_MEGAMORPHISM=ESTABLISHED_FOR_THIS_CALL_SITE
```

The causal record is:

`docs/project/evidence/PERF010-A/PERF010-A_STABLE_SPECIALIZATION_IDENTITY_CHURN_ROOT_CAUSE.md`

This establishes implementation-induced polymorphism without requiring any
change to Protos semantics.

### Smalltalk/Truffle precedent

The audit now retains the following architectural and published precedent:

- current TruffleSOM and TruffleSqueak ordinary dispatch caches classify by
  stable receiver class/object-layout and resolved method/target state rather
  than each fresh receiver object's identity;
- *Cross-Language Compiler Benchmarking: Are We Fast Yet?* reports a
  TruffleSOM performance outlier involving a hot access becoming megamorphic
  because of runtime/object representation;
- *Optimizing Communicating Event-Loop Languages with Truffle* describes
  avoiding an effectively megamorphic shared dispatch funnel by retaining
  send-site-specific PIC context.

References:

- https://stefan-marr.de/papers/dls-marr-et-al-cross-language-compiler-benchmarking-are-we-fast-yet/
- https://stefan-marr.de/papers/agere-marr-moessenboeck-optimizing-communicating-event-loop-languages-with-truffle/

These results do not imply that Protos should adopt Smalltalk class semantics.
They justify making accidental compiler-visible polymorphism a first-class
PERF011 audit dimension.

### Expanded audit matrix

PERF011 should apply this matrix to at least:

1. ordinary method send;
2. Closure call;
3. ordinary slot read;
4. ordinary slot write;
5. delegated lookup where distinct;
6. object construction;
7. integer/arithmetic primitive dispatch;
8. Array/indexed access;
9. activation/lexical access where hot.

For each applicable surface classify:

```text
ABSTRACTION=...

SEMANTIC_POLYMORPHISM=...
SEMANTIC_STABILITY_FACTS=...

TRUFFLE_SPECIALIZATION_POLYMORPHISM=...
CACHE_KEY=...
CACHE_LIMIT=...
INVALIDATION_MODEL=...
GENERIC_FALLBACK_TRIGGER=...

HOST_IMPLEMENTATION_VISIBLE_AFTER_PE=
  YES | NO | INCONCLUSIVE

POLYMORPHISM_FIDELITY=
  MATCH | MISMATCH | INCONCLUSIVE

ACCIDENTAL_MEGAMORPHISM=
  ESTABLISHED | NOT_ESTABLISHED | INCONCLUSIVE

SEMANTIC_CHANGE_REQUIRED_TO_FIX=
  YES | NO | INCONCLUSIVE

NEXT_BOUNDED_EXPERIMENT=
  ... | NONE
```

Source-level apparent stability is insufficient when a conclusion depends on
compiler behavior. Where material, dynamic evidence is required: generated DSL
specialization state, compiler lifecycle/invalidation, compiler graph/trace, or
an equivalent direct compiler-visible discriminator.

### Steady-state validity

PERF011 also distinguishes timing stability from compiler-state stability:

```text
TIMING_STABLE=YES | NO | INCONCLUSIVE
COMPILER_LIFECYCLE_STABLE=YES | NO | INCONCLUSIVE
STEADY_STATE_VALID=...
```

The PERF010-A lifecycle demonstrates why: a helper can compile successfully,
later consume additional PIC entries, invalidate, activate a replacing generic
specialization, recompile, and finally receive a permanent bailout.

### Work-item boundary

```text
PERF010-A / #691
  -> fix and causally measure the concrete prepared-target/PIC pathology

PERF011 / #693
  -> generalize the diagnostic method
  -> audit remaining runtime abstractions
  -> identify independent representation mismatches
  -> define bounded experiments

future PERFxxx
  -> independently valuable interventions discovered by PERF011 when they
     require their own closure/blockage/scheduling identity
```

No broad Shape/DynamicObject/Frame migration is authorized by this scope
expansion. PERF011 must not fork a competing implementation experiment while
#691 owns the current method-send discriminator.
