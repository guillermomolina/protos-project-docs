# PLAT043 — Standard Boolean-control interpreter ownership boundary

Status: **RATIFIED**

Selected architecture: **Candidate B — execute the complete prepared standard Boolean-control capability in the tagged semantic Bytecode interpreter; retain the untagged structured/C-prime owner for every other structured prepared family**.

Approval: explicit project-owner approval on 2026-10-01:

~~~text
ok acepto B
~~~

Decision Issue: `guillermomolina/protos#763`

Parent workstream: `PERF025 / guillermomolina/protos#758`

Product baseline:

~~~text
PROTOS_REVISION=0a5115caddba8ebb7bb4275ce32441ab90938d3d
PROTOS_VERSION=0.3.133-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative JVM/Truffle implementation-architecture decision.
Observable Protos semantics remain owned by the normative specification.

## Decision

PLAT043 selects Candidate B for the residual PERF025-C2 structured-recursion
stack amplification.

The tagged semantic Bytecode interpreter owns the already-prepared standard
Boolean-control orchestration for exactly these existing `PreparedBooleanCall`
kinds:

~~~text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
~~~

Every other structured prepared family continues to cross the PLAT042 helper
boundary into the untagged structured/C-prime interpreter.

Conceptually:

~~~text
tagged semantic Bytecode interpreter
    ordinary source execution
    ordinary source Closure invocation

    prepared standard Boolean control
        IF_TRUE
        IF_FALSE
        IF_TRUE_IF_FALSE
        AND
        OR

untagged structured/C-prime interpreter
    while
    each
    ensure
    Error.handle
    match / case
    structured Map / collection operations
    import
    I/O C-prime
    all other prepared structured families
~~~

This is a physical ownership change only. It does not create a second Boolean
semantics, a new callable kind, a new continuation kind, or a selector-level
intrinsic that bypasses ordinary lookup.

## Trigger

PLAT042 Candidate B′ was implemented at the product baseline above and removed
the old universal source-wrapper/helper double-CallTarget path.

Published C1c evidence established:

~~~text
UNIVERSAL_SEMANTIC_WRAPPER=REMOVED
ORDINARY_SOURCE_CLOSURE_CALLTARGET_COUNT=1

STRUCTURED_CPRIME_OWNER=UNTAGGED
STRUCTURED_ONLY_HELPER_BOUNDARY=YES

GUEST_CALL_STACK_SIZE_BYTES=64 MiB
DEDICATED_GUEST_CARRIER=STILL_PRESENT
~~~

The retained 10,000-deep recursive benchmark shape performs a standard
structured `ifTrue` at each positive recursion level.

PERF025-C2A reconstructed the live physical descent as approximately:

~~~text
R_i = tagged semantic source root executing repeat(count_i, ...)
S_i = untagged structured/C-prime root orchestrating Boolean control
B_i = tagged semantic source root for the selected callback

R_i -> S_i -> B_i -> R_i+1 -> S_i+1 -> B_i+1 -> ...
~~~

PLAT042 already de-structures the callback itself when that child invocation does
not require structured dispatch. The residual avoidable owner is therefore
`S_i`.

Under PLAT043 the intended shape becomes:

~~~text
R_i -> B_i -> R_i+1 -> B_i+1 -> ...
~~~

One helper guest CallTarget is removed per reached Boolean callback level.

## Scope of the Boolean capability

PLAT043 moves the existing finite `PreparedBooleanCall` orchestration unit, not
five unrelated selector special cases.

The standard semantic rules remain unchanged:

- `ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, and `or` remain ordinary
  messages;
- ordinary lookup/selection occurs before backend specialization;
- only the selected callback is callability-validated/invoked where the standard
  protocol requires path-sensitive validation;
- `ifTrueIfFalse` keeps ordinary eager evaluation of both callback-producing
  argument expressions before invocation;
- `and` and `or` still require a canonical Boolean callback result after a
  reached callback returns normally;
- a callback remains any ordinarily invokable object, not a Closure-identity
  shortcut; and
- Error, non-local return, cancellation, suspension and Future behavior are
  unchanged.

`not()` is not a `PreparedBooleanCall` structured callback operation and is
not added to this ownership rule.

No analogous rule is created for `ifNull`, `ifNotNull`, `while`, or any
other protocol by similarity alone.

## Ordinary selection remains authoritative

PLAT040 and ordinary D013 lookup remain authoritative.

The permitted order is:

~~~text
ordinary lookup / selection
    -> selected behavior + receiver + methodHome
    -> existing guarded/provenance classification
    -> prepared standard Boolean capability
    -> semantic-interpreter Boolean orchestration
~~~

PLAT043 does **not** permit:

~~~text
selector spelling == "ifTrue"
    -> bypass lookup
    -> privileged branch
~~~

Overrides, shadowing, copied behavior, aliases and custom same-name methods retain
their existing observable semantics.

## PLAT032 non-canonical wrapper compatibility

The generic prepared-call path may classify implementation provenance for
copied, aliased or otherwise non-canonical standard wrappers.

PLAT043 changes only the physical owner of the prepared Boolean orchestration.
It does not canonicalize the logical selected method.

For every such admitted invocation:

~~~text
SELECTED_METHOD_IDENTITY=PRESERVED
RECEIVER=PRESERVED
METHOD_HOME=PRESERVED
ACTIVATION=PRESERVED
TOOLING_IDENTITY=PRESERVED
NONCANONICAL_DOES_NOT_BECOME_CANONICAL=YES
~~~

PLAT032 therefore remains authoritative with no semantic or logical-wrapper
delta.

## PLAT014 / PLAT028 continuation compatibility

The tagged semantic interpreter already participates in the same Bytecode DSL
resumable machinery used by source execution.

PLAT043 does not add a host continuation ABI, replay protocol, hidden Task,
scheduler handoff, or generic guest-CallTarget trampoline.

The Boolean owner must compose with the existing continuation state exactly as
ordinary semantic Bytecode execution does:

~~~text
CONTINUATION_RESULT_COMPOSITION=PRESERVED
YIELD_RESUME=PRESERVED
SAME_TASK=PRESERVED
STRUCTURED_CHILD_OWNERSHIP=PRESERVED
NO_REPLAY=PRESERVED
NEW_CONTINUATION_KIND=NO
~~~

PLAT028's durable invariant remains: surviving post-callback sequencing state is
guest/C-prime resumable state, never a retained Java/native continuation frame.
PLAT043 only clarifies that the tagged semantic Bytecode interpreter may own the
bounded Boolean sequencing state.

## PLAT021 Error/control compatibility

The Boolean ownership move introduces no handler, cleanup scope, return home,
cancellation boundary, Task, Future or scheduler boundary.

The implementation must preserve:

~~~text
ERROR_PROPAGATION=PRESERVED
NON_LOCAL_RETURN=PRESERVED
RETURN_HOME=PRESERVED
ENSURE_UNWIND=PRESERVED
CANCELLATION_UNWIND=PRESERVED
NO_NEW_CANCELLATION_CHECKPOINT=YES
~~~

A physical Bytecode suspension/resume must not be observable as semantic scope
exit and must not duplicate callback effects.

## RootTag and lexical-state authority

PLAT026 remains authoritative.

The standard Boolean orchestration executes as implementation control inside the
already-truthful tagged semantic source root. No helper root is promoted to a
semantic root and no manual RootTag emulation is introduced.

PLAT036 / PERF013 remain authoritative for lexical-state representation and
captured-local fast paths. Boolean orchestration becomes no new lexical owner or
captured-state authority.

## PLAT042 amendment

PLAT042 remains ratified and authoritative for the general source/structured
interpreter split.

PLAT043 introduces exactly one narrow later authority:

~~~text
PLAT042_GENERAL_RULE=
  structured prepared invocation
  -> untagged structured/C-prime owner

PLAT043_NARROW_EXCEPTION=
  prepared standard Boolean-control capability
  -> tagged semantic Bytecode interpreter

ALL_OTHER_STRUCTURED_PREPARED_INVOCATIONS=
  PLAT042 untagged structured/C-prime owner
~~~

No other PLAT042 family is reopened.

## GITHUB021 invariant/delta result

~~~text
PLAT017_DELTA=NONE
PLAT021_DELTA=NONE
PLAT026_DELTA=NONE
PLAT028_CONTINUATION_INVARIANT_DELTA=NONE
PLAT032_DELTA=NONE
PLAT036_DELTA=NONE
PLAT040_DELTA=NONE
PLAT041_DELTA=NONE

PLAT042_DELTA=
  NARROW_PHYSICAL_OWNERSHIP_EXCEPTION_FOR_PREPARED_STANDARD_BOOLEAN_CONTROL

NEW_PROTOS_SEMANTICS=NO
SPECIFICATION_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

The project owner approved Candidate B after this delta was disclosed.

## Rejected alternatives

### Candidate A — only IF_TRUE / IF_FALSE local

Rejected.

It directly targets the retained benchmark but splits physical ownership inside
one already-unified `PreparedBooleanCall` capability and leaves future Boolean
kinds to reopen the same boundary.

### Candidate C — local Boolean only for exact canonical selections

Rejected.

It can preserve semantics but creates two physical orchestration machines for
the same standard Boolean implementation based on provenance. PLAT032 already
provides the logical-identity constraints required to preserve non-canonical
wrappers without retaining the helper root.

### Candidate D — generic guest-CallTarget trampoline/state handoff

Rejected.

No supported Truffle primitive was established that transparently replaces a
current guest CallTarget activation with a child and later resumes the parent.
A new generic handoff protocol would touch Error, ensure, cancellation,
ReturnHome, nested structured dispatch, tooling and continuation ownership far
beyond the present need.

### Candidate E — retain PLAT042 unchanged

Rejected as the selected PERF025-C2 target.

It is operationally safe but leaves the currently established residual
structured-recursion amplification and the dedicated large-stack carrier
unchanged.

## Carrier consequence

PLAT043 does **not** authorize carrier retirement or a smaller fixed stack.

After Candidate B, recursive callback structure still contains approximately:

~~~text
R_i -> B_i -> R_i+1
~~~

Therefore post-implementation measurement is mandatory before claiming that the
retained 10,000-deep workload is safe on the ordinary host stack.

The existing approximately-32-MiB observation belongs to the old
`R -> S -> B` topology and cannot establish a safe new stack size.

The current guest carrier also provides session/Process serialization in
addition to stack provisioning. If large-stack provisioning becomes unnecessary,
that serialization responsibility still requires an explicit implementation
shape.

~~~text
CARRIER_RETIREMENT_AUTHORIZED=NO
CARRIER_STACK_REDUCTION_AUTHORIZED=NO
POST_IMPLEMENTATION_STACK_EVIDENCE_REQUIRED=YES
SESSION_SERIALIZATION_MUST_REMAIN=YES
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

## Performance and implementation acceptance

The implementation slice released by this decision must keep the retained
benchmark shape/count unchanged and establish at least:

1. focused semantic correctness for all five prepared Boolean kinds;
2. override/custom-selector and copied/aliased/non-canonical wrapper coverage;
3. arbitrary ordinarily-invokable callback behavior and path-sensitive
   callability;
4. Error, non-local return, suspension/resume and cancellation compatibility;
5. RootTag/helper-root topology evidence;
6. multi-Context correctness;
7. Native Image generated-structure/forced-JIT compatibility where affected;
8. the complete required integrated test gate under current implementation
   policy; and
9. unchanged-workload stack measurement for the retained 10,000-deep recursive
   driver.

Only that last evidence may determine the next carrier step.

## Strongest argument against the selected candidate

Candidate B weakens PLAT042's clean physical statement that every structured
prepared invocation crosses the untagged owner, while it removes only one of
the remaining guest roots per recursive level.

It is therefore possible that implementation will materially reduce stack
pressure but still not permit ordinary-stack execution of the retained
10,000-deep workload.

That risk is accepted because Candidate B is bounded to one existing finite
capability, needs no new semantic or continuation mechanism, is reversible, and
directly removes the currently established residual helper activation.

## Implementation authority

After durable ratification and live coordination closure, PERF025 may implement
Candidate B in `guillermomolina/protos`.

The implementation must assume the current product HEAD and preserve every
invariant above.

It must not combine this work with carrier removal. Carrier evidence is a
post-implementation gate.

~~~text
NEXT_SLICE=PERF025-C2B
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

## References

- `guillermomolina/protos#763` — PLAT043 decision Issue.
- `guillermomolina/protos#758` — PERF025 parent/consumer.
- `guillermomolina/protos#760` — PLAT042 prior general ownership boundary.
- `guillermomolina/protos#681` — historical BUG008; remains closed.
- `docs/project/evidence/PLAT043/PLAT043_STANDARD_BOOLEAN_CONTROL_OWNERSHIP_INVESTIGATION.md`.
- `docs/project/decisions/platform/PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md`.
- `docs/project/evidence/PERF025/PERF025_C1C_PLAT042_B_PRIME_CUTOVER.md`.


## Published implementation checkpoint

PERF025-C2B was published in `guillermomolina/protos` at:

~~~text
SLICE_BASE_REVISION=d8dcc95d34088e942e737b98c7ad42a81a977293
PROTOS_REVISION=57d8cf4ec195aca3cb5b7c33d37755d965e9f3df
PROTOS_VERSION=0.3.134-SNAPSHOT
COMMIT_SUBJECT=PERF025-C2B: PLAT043 standard Boolean semantic-interpreter ownership
~~~

The published product implements the PLAT043 physical-owner exception:

~~~text
STANDARD_BOOLEAN_PREPARED_OWNER=TAGGED_SEMANTIC_BYTECODE_INTERPRETER

BOOLEAN_KINDS=
  IF_TRUE
  IF_FALSE
  IF_TRUE_IF_FALSE
  AND
  OR

OTHER_STRUCTURED_PREPARED_OWNER=
  UNTAGGED_STRUCTURED_CPRIME_INTERPRETER

BOOLEAN_HELPER_CALLTARGET_REMOVED=YES
NON_BOOLEAN_PLAT042_HELPER_PRESERVED=YES

SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO
CARRIER_CHANGED=NO
~~~

`CanonicalToBytecodeLowerer` now detects the already-prepared standard Boolean
capability before the general structured-dispatch test and lowers that bounded
state machine inside the semantic source root. The semantic interpreter exposes
operation wrappers that delegate to the existing
`ProtosBytecodeRootNode.PreparedBooleanCall` implementation authority rather
than defining a second Boolean semantics.

The local Boolean lowering retains the outer prepared-call completion in a
Bytecode `TryFinally` and preserves child continuation composition. A child
callback that itself requires structured dispatch still enters the PLAT042
untagged root; an ordinary callback is entered directly from the semantic root.

Published topology regression evidence changes the prior C1c expectation:

~~~text
STANDARD_BOOLEAN_CALLBACK_STACK=
  semantic callback root
  -> semantic caller root

UNTAGGED_BOOLEAN_HELPER_BETWEEN_THEM=NO
~~~

and separately preserves:

~~~text
NON_BOOLEAN_STRUCTURED_EXAMPLE=Closure.while
UNTAGGED_PLAT042_HELPER=YES
HELPER_ROOT_TAG=NO
~~~

The product also adds Boolean callback dispatch/control conformance coverage for
ordinary non-Closure invokable callbacks, reached invalid-call behavior,
non-local return, Error propagation, custom same-name selectors, and copied
standard `ifTrue` behavior.

### Carrier gate remains open

The published C2B commit intentionally leaves:

~~~text
GUEST_CALL_STACK_SIZE_BYTES=64 MiB
DEDICATED_GUEST_CARRIER=STILL_PRESENT
~~~

PLAT043 required an unchanged-workload post-implementation measurement of the
retained 10,000-deep recursive driver before any carrier or fixed-stack
decision.

No such post-C2B measurement is retained in the published product commit,
CHANGELOG, current PERF025/PLAT043 Issue evidence, or commit CI/status metadata
available at this checkpoint.

Therefore durable project state is:

~~~text
PERF025_C2B_CODE=COMPLETE
PERF025_C2B_PUBLICATION=COMPLETE
PERF025_C2B_STACK_GATE=PENDING

CARRIER_RETIREMENT_AUTHORIZED=NO
CARRIER_STACK_REDUCTION_AUTHORIZED=NO

BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

This checkpoint does not infer unreported validation or benchmark success from
the existence of the commit.

Detailed implementation evidence is retained under
`docs/project/evidence/PERF025/PERF025_C2B_PLAT043_BOOLEAN_OWNERSHIP_CUTOVER.md`.
