# PERF010-A — Post-I072 recommended action plan

Status: **RECOMMENDED PERFORMANCE CAMPAIGN; DOES NOT ITSELF AUTHORIZE PRODUCT CHANGES**

This durable, non-normative record retains the recommended action plan produced
after the final post-I072 guest-call cross-runtime investigation.

It is dependency-ordered and deliberately uses stop gates. A slice that changes
the predicted compiler structure but does not produce its predeclared effect
must stop the campaign for re-evaluation rather than automatically advancing to
the next optimization.

## Context

The current fixed point is:

```text
PROTOS_REVISION=cf9b39b25dc9a3c4cd1c538749c3a363760ae45b
PROTOS_VERSION=0.3.96-SNAPSHOT

POST_I072_FPRIME_BIG_COST=NO
PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
```

The investigation indicates that the shared `repeat` driver pays the dominant
common **guest-call tax**, while the measured leaf operations are secondary.

Seven comparison implementations converge on the same mature-Truffle steady
shape: constant/stable call selection, assumptions, `DirectCallNode`, compact
arguments and PE-visible captured-state representation.

## Principles

1. Order by dependency: **compilability -> stable call selection -> represented
   primitive-family selection -> residual escape/capture architecture**.
2. One mechanism per implementation slice.
3. Keep the exact generic path as fallback.
4. Declare the expected observable/compiler effect before each slice.
5. If the expected effect is absent, stop and re-evaluate.
6. Do not reopen ratified language semantics merely for benchmark speed.
7. Any change that reopens PLAT036, PLAT026, PLAT040 or observable
   Closure/Context semantics must go through the design process and explicit
   owner approval.
8. Human-executor mode remains authoritative for `guillermomolina/protos`
   product publication.

## Step 0 — confirmation gate

Repository:

```text
guillermomolina/protos-benchmarks
```

Product changes:

```text
NONE
```

At exact current product revision, run the existing PERF010-A timing/lifecycle
machinery on `closure-call` and its control under the same one-CPU and large
stack conditions used by the retained harness.

Capture:

```text
A. TraceCompilation + CompilationFailureAction=Print
B. Compilation=false
C. normal timing
```

The trace must identify the exact common `repeat`/callback roots, not merely
report aggregate failures.

Classify:

```text
HOT_ROOT_COMPILATION=OPT_DONE|OPT_FAILED
FAILURE_CAUSE=<exact stack>|NONE
JIT_VS_INTERPRETER=<measured relation>
```

Routing:

```text
frame/VirtualFrame/materialization-specific failure -> Step 1
opt done and JIT ~= interpreter                  -> skip Step 1; Step 2
JIT materially better but still expensive       -> skip Step 1; Step 2
different failure                               -> stop and re-evaluate exact cause
```

Step 0 is mandatory. Static source prediction is not sufficient evidence to
perform Step 1.

## Step 1 — restore hot-root compilability

Repository:

`guillermomolina/protos`

Applicability:

**Only if Step 0 demonstrates the exact frame/escape/materialization failure this
slice is designed to fix.**

Mechanism:

Ensure that a lexical authority which genuinely escapes with a frame retains an
escape-safe materialized frame in the same semantic cases where current
semantics require the authority to outlive the virtual invocation frame.

Do not change the compact/deferred path merely to make the benchmark pass.

Success criterion:

```text
BEFORE: target hot root opt failed for the identified frame mechanism
AFTER:  same root opt done or the exact failure is removed
```

Timing is secondary at this step. No predicted percentage improvement is an
acceptance requirement.

If compilation does not change as predicted, stop.

## Step 2 — stable direct Closure-call selection

Repository:

`guillermomolina/protos`

Scope:

Direct Closure invocation paths used by `operation()` and recursive
`repeat(...)`.

Target steady form:

```text
stable Closure definition / entered Context
  -> canonical call-selection assumption
  -> cached Context-owned target
  -> DirectCallNode
  -> compact frame ABI / lazy callee-side activation where semantically valid
  -> exact generic fallback
```

Required semantic guards include ordinary local `call` shadowing and
invalidation of the canonical `Object.call` selection.

The intended implementation pattern is the same architectural class already
used by mature Truffle runtimes and by I072's admitted guarded source-call path,
but direct Closure invocation must preserve its exact current semantics.

Success criterion:

```text
closure-call - control guest-call increment
  -> falls by a clearly multiplicative factor

compiler evidence
  -> target/callee is stable
  -> admitted call reaches DirectCallNode without generic selection surviving
     on the valid hot hit
```

If the compiler shape improves but the roughly microsecond-scale per-call tax
remains nearly intact, stop and re-evaluate before Step 3.

## Step 3 — represented receiver guarded selection

Repository:

`guillermomolina/protos`

This is required work, not merely a benchmark-dependent optional optimization,
because fundamental represented values currently fall outside I072-A's
ordinary-object guarded lookup.

### Step 3A — Boolean

Initial mandatory family:

```text
canonical true
canonical false
```

Goal:

Allow stable canonical Boolean sends such as:

```text
ifTrue
ifFalse
ifTrueIfFalse
and
or
```

to reach the already-existing I072-E structured Boolean machinery through a
representation-aware guard/assumption scheme, while retaining exact ordinary
override/fallback behavior.

This does **not** reopen D051 or introduce primitive `if` semantics.

Success criterion:

```text
canonical Boolean control sites no longer use the generic represented-value
selection path on stable valid hits
```

A performance gain is expected but is not the sole justification: this closes a
known implementation-completeness gap in the normal Protos conditional path.

### Step 3B — Integer

Initial family:

`Integer`

Targets in the shared driver:

```text
count > 0
count - 1
```

Use a proven representation-family invariant plus entered Context and delegation
assumptions. Do **not** generalize blindly by Java class to every
`ProtosRepresentedValue`; each admitted family must guarantee the required
delegation/own-slot invariants.

Success criterion:

```text
stable Integer driver sends no longer use generic represented-value selection
on the admitted hit
```

Performance discriminator for Step 3 as a whole:

```text
shared control/driver time falls by a large factor
```

If selection becomes stable but the control cost remains in the same class,
stop and re-evaluate.

## Step 4 — residual architecture checkpoint

Step 4 is conditional on the residual after Steps 1–3.

If the guest-call tax is already in a normal Truffle cost class, Step 4 is not a
PERF010 blocker.

Otherwise route the remaining architecture through the required PLAT/D approval
gate.

### 4A — captured-variable/environment representation

Question:

Keep the PLAT036 frame-backed authority for captured execution, or move the
Bytecode backend toward cell/copied-value/fixed-depth representation with lazy
guest Context reification while preserving capture-by-reference and D179.

Relevant comparison patterns include GraalPy PCell and the lazy/context
representations in Squeak/SOM/Ruby/JS/Pkl.

### 4B — literal Boolean-control Closure

Question:

For a canonical stable Boolean selection with a literal trailing Closure, can
the compiled implementation avoid physical per-iteration Closure
materialization while preserving:

- ordinary `ifTrue` message semantics;
- override/shadowing behavior;
- Closure identity when observable;
- capture by reference;
- reflection/tooling;
- non-local return;
- suspension/control behavior?

D051 remains authoritative. I072-E remains the existing structured Boolean
execution machinery. Step 4B concerns only the physical optimized
representation/lowering when all guards permit it.

## Explicitly not recommended before the gates

Do not resume percentage-scale micro-ablations as the primary search:

- duplicate Optional projections;
- duplicate map probes;
- isolated `List.copyOf`;
- another F′ refinement;
- semantic-shell removal by itself;
- unconditional rewrite of the structured cascade.

These may be revisited after stable compilation/selection if residual profiles
show that they remain material.

## Verification per implementation step

Correctness must be proportional to the touched surface and include applicable
focal Java tests plus relevant conformance around:

- D013 lookup and shadowing;
- Closure invocation and local `call` override;
- execution-context behavior;
- D179 binding/remove/recreate semantics where touched;
- Boolean exact-domain and callback behavior;
- suspension/control/non-local-return paths when touched.

At formal slice closure, follow the repository's implementation/versioning and
changelog rules and run the required integrated gate.

Performance uses the same fixed harness and exact before/after revisions.

## Campaign state fields

```text
STEP_0=REQUIRED
STEP_1=CONDITIONAL_ON_EXACT_COMPILER_FAILURE
STEP_2=REQUIRED
STEP_3A_BOOLEAN=REQUIRED
STEP_3B_INTEGER=REQUIRED
STEP_4=CONDITIONAL_ON_RESIDUAL

BOOLEAN_SEMANTICS_REOPENED=NO
I072_E_STRUCTURED_BOOLEAN_RETAINED=YES
REPRESENTED_BOOLEAN_FAST_PATH_COMPLETE=NO

PERF010A_DOMINANT_CAUSE=NOT_ESTABLISHED
ATTRIBUTABLE_FRACTION=NOT_ESTABLISHED
PRODUCT_IMPLEMENTATION_AUTHORIZED_BY_THIS_RECORD=NO
```

The live Issue/Project layer remains authoritative for whether this plan is
adopted as formal PERF010 follow-up work.
