# PLAT044 — Semantic Closure activation versus physical RootTag boundary

Status: **RATIFIED**

Selected architecture: **Candidate B′ — guarded Smalltalk-style parser
open-coding for eligible immediate literal standard-control callbacks, preserving
a fresh semantic Closure activation in Bytecode locals while removing the
distinct callback RootCallTarget/FrameInstance for that eligible path**.

Approval: explicit project-owner approval on 2026-10-02:

~~~text
ok apruebo b'
~~~

Decision Issue: `guillermomolina/protos#766`

Parent discovery work: `PERF026 / guillermomolina/protos#765`

Product baseline:

~~~text
PROTOS_REVISION=8ea87fb1794599247f0e95fd5570d7dd7e08a52f
PROTOS_VERSION=0.3.134-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative JVM/Truffle implementation-architecture decision.
Observable Protos language semantics remain owned by the normative specification.

## Decision

PLAT044 selects a narrow physical representation exception for eligible
standard-control callback sites.

The permitted order is:

~~~text
ordinary lookup / selection
    -> selected behavior + receiver + methodHome
    -> prove exact standard-control implementation/provenance
    -> prove eligible immediate literal Closure callback
    -> preserve fresh semantic Closure activation
    -> execute callback body as parser-inlined lexical/resumable region
    -> expose truthful custom inline RootTag + scope/root-instance projection
~~~

Every non-eligible, dynamic, copied/custom, invalidated, tooling-incompatible or
otherwise unsupported callback remains an ordinary Closure invocation through
its physical semantic RootCallTarget.

This is not general Closure inlining authority.

## Selected B′ representation

For the eligible path:

~~~text
SEMANTIC_CLOSURE_VALUE=UNCHANGED
FRESH_CLOSURE_ACTIVATION=YES
ACTIVATION_STORAGE=BYTECODE_LOCALS_OR_EQUIVALENT_APPROVED_LOCAL_STATE
CALLBACK_BODY=INLINE_LEXICAL_RESUMABLE_REGION
INLINE_ROOT_TAG=YES
SEPARATE_CALLBACK_ROOTCALLTARGET=NO
SEPARATE_CALLBACK_FRAMEINSTANCE=NO
~~~

The implementation must preserve:

- ordinary D013 lookup/selection authority;
- callback expression evaluation order;
- callback callability semantics and generic fallback;
- Closure identity whenever observable;
- fresh activation/context semantics;
- capture by reference;
- `this`, `context`, `methodHome`, supplied argument/default/rest semantics;
- ReturnHome and non-local return;
- Error/dynamic-control behavior;
- suspension/resumption and continuation composition;
- cancellation and Task/Actor/Process domain rules;
- source location, breakpoints, stepping and callback scope projection;
- multi-Context isolation; and
- Native Image/runtime-compilation compatibility.

If an implementation shape cannot prove those invariants, it falls back to the
ordinary physical Closure path.

## PLAT026 explicit delta

PLAT044 explicitly reopens and narrows PLAT026's former one-to-one mapping:

~~~text
OLD:
  semantic Closure activation
  -> distinct semantic Bytecode root
  -> automatic RootTag
  -> distinct RootCallTarget / FrameInstance

B_PRIME_ELIGIBLE_STANDARD_CONTROL_PATH:
  semantic Closure activation
  -> preserved semantic activation
  -> parser-inlined callback region in containing semantic root
  -> truthful custom RootTag
  -> no distinct callback RootCallTarget / FrameInstance
~~~

For ordinary/general Closure calls, the old PLAT026 mapping remains unchanged.

The approved tooling consequence is:

~~~text
BREAKPOINT_SOURCE_STEPPING=PRESERVED
CURRENT_CALLBACK_SCOPE=PRESERVED
CURRENT_CALLBACK_ROOT_INSTANCE_NAME=PRESERVED

DISTINCT_CALLBACK_DEBUGSTACKFRAME=NO
DISTINCT_CALLBACK_TRUFFLE_STACKTRACE_ELEMENT=NO
ENCLOSING_CALLER_PLUS_CALLBACK_AS_TWO_PHYSICAL_FRAMES=NO
~~~

This tooling-frame delta is part of Candidate B′ and was disclosed before owner
approval.

No custom DAP, virtual debugger stack, synthetic generic Truffle FrameInstance,
global activation registry, or new Protos-visible stack-frame semantics are
selected.

## PLAT040 / PLAT043 composition

PLAT040/I072 remains authoritative:

~~~text
ordinary selection first
-> guarded stable standard behavior
-> exact generic fallback
~~~

No selector spelling alone may trigger B′.

PLAT043 remains authoritative for complete prepared standard Boolean-control
orchestration in the tagged semantic Bytecode interpreter. PLAT044 changes only
the physical representation of an eligible reached literal callback.

Conceptually, for recursive standard Boolean control:

~~~text
pre-PLAT043:
  R_i -> S_i -> B_i -> R_i+1

post-PLAT043:
  R_i -> B_i -> R_i+1

PLAT044 B_PRIME eligible target:
  R_i -> [inline B_i semantic region] -> R_i+1
~~~

The callback remains a semantic Closure activation even though it no longer
creates a distinct physical callback root in the eligible path.

## PLAT041 / lexical-state composition

PLAT041's inline Object-body architecture is the closest internal implementation
precedent: a semantically distinct activation can be active in a Bytecode local
while its body executes as a resumable region of the containing root.

PLAT044 does not make Object bodies equivalent to Closures. It reuses only the
physical active-activation pattern.

Nested escaping Closure capture from an inline callback must preserve current
same-generation lexical authority. If that cannot be done within the already
approved machinery, the shape falls back instead of inventing a new capture
architecture.

## PLAT014 continuation composition

An inline callback may suspend only through existing Bytecode DSL/C-prime
continuation machinery.

~~~text
NO_REPLAY=YES
NEW_CONTINUATION_KIND=NO
NEW_TASK_OR_SCHEDULER_BOUNDARY=NO
YIELD_RESUME_VALUE=PRESERVED
RETURN_HOME_NLR=PRESERVED
ERROR_CANCELLATION_UNWIND=PRESERVED
~~~

The active inline callback activation must remain live and correct across
suspension/resumption.

## Comparative rationale

The broad Truffle default for ordinary functions/blocks is:

~~~text
RootCallTarget
+ stable DirectCallNode
+ compiler/partial-evaluation inlining
~~~

Current GraalPy, GraalJS ordinary function calls, TruffleRuby blocks,
TruffleSqueak ordinary BlockClosure dispatch, Apple Pkl functions, Espresso,
Sulong and GraalWasm all support that general pattern.

PLAT044 does not reject it for ordinary Closure calls.

The relevant exception is the Smalltalk control family. GNU Smalltalk/Squeak
compile eligible standard Boolean/while/control messages with literal blocks
into local branch/loop bytecodes rather than retaining ordinary send/block
activation overhead.

Current Truffle Bytecode DSL also explicitly supports parser-inlined calls with
custom root-tag regions for tooling.

Protos expresses standard control through ordinary messages and Closures, so
requiring a physical Closure root solely because of the source model would
conflate semantic representation with implementation representation.

## Rejected alternatives

### A — distinct physical Closure root always mandatory

Rejected for the eligible standard-control common path.

It preserves current debugger-frame cardinality but leaves the exact
interpreter/root amplification that PERF025/PERF026 are trying to remove.

### C — tooling-sensitive dual topology

Rejected.

Changing physical execution topology merely because tooling is attached adds a
second operational mode and difficult transition semantics without establishing
a language requirement for old frame cardinality.

### D — physical root plus compiled-only inlining

Retained as the architecture for ordinary/general Closure calls, but rejected as
the complete answer for eligible standard control.

It can remove effective call overhead in compiled code but leaves the
interpreter/deep-recursion `R -> B -> R` topology.

### E — custom virtual debugger-frame architecture

Rejected as overengineering.

PLAT044 does not require reconstructing a second generic Truffle frame after the
physical callback root has deliberately been removed.

## GITHUB021 result

~~~text
PLAT014_DELTA=NONE_IF_IMPLEMENTATION_PROVES_EQUIVALENCE
PLAT015_SCOPE_SEMANTICS_DELTA=NONE
PLAT026_DELTA=EXPLICIT_AND_APPROVED
PLAT040_DELTA=NONE
PLAT041_DELTA=NONE
PLAT043_DELTA=NONE_TO_BOOLEAN_SEMANTICS

NEW_PROTOS_LANGUAGE_SEMANTICS=NO
SPECIFICATION_CHANGE=NO
DECISION_APPROVAL_PROVENANCE=PASS
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Implementation authority

The first released implementation work is:

~~~text
NEXT_WORK=PERF026-B/#767
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
~~~

PERF026-B should establish the common B′ mechanism through the smallest standard
Boolean surface first. It must not simultaneously implement `whileTrue`,
collection `each`, carrier retirement, or a general Closure-inlining framework.

PERF026-C and PERF026-D remain later consumers of the proven mechanism.

PERF025's carrier decision remains blocked until the relevant Boolean/common
callback-root topology has been implemented and measured.

## Evidence and references

- `guillermomolina/protos#766` — PLAT044 decision Issue.
- `guillermomolina/protos#767` — PERF026-B first implementation consumer.
- `guillermomolina/protos#768` — PERF026-C later while consumer.
- `guillermomolina/protos#769` — PERF026-D later each consumer.
- `guillermomolina/protos#758` — PERF025 carrier consumer.
- `guillermomolina/protos#375` — PLAT026 amended physical-root/tooling boundary.
- `guillermomolina/protos#718` — PLAT040 selected-send specialization authority.
- `guillermomolina/protos#759` — PLAT041 inline Object-body precedent.
- `guillermomolina/protos#763` — PLAT043 Boolean interpreter ownership.
- `docs/project/evidence/PLAT044/PLAT044_SEMANTIC_CLOSURE_PHYSICAL_ROOT_DECISION_EVIDENCE.md`.


## Published implementation checkpoint — PERF026-B1

The first PLAT044 B-prime implementation slice was published in
guillermomolina/protos at:

~~~text
PROTOS_REVISION=5b5dedd7a36b4aba0684a072a4f1a86a52ec9923
PROTOS_VERSION=0.3.135-SNAPSHOT
COMMIT_SUBJECT=PERF026-B1: inline standard ifTrue literal callback
~~~

B1 consumes B-prime only for an already-selected standard IF_TRUE whose reached
callback is an eligible immediate zero-argument Closure literal.

~~~text
ELIGIBLE_IF_TRUE_LITERAL_CALLBACK=
  fresh semantic callback activation preserved
  + callback body inline in containing semantic source root
  + custom inline RootTag
  + callback scope projection
  - distinct callback RootCallTarget / FrameInstance

DYNAMIC_OR_NON_ELIGIBLE_CALLBACK=
  ordinary physical Closure invocation preserved
~~~

Fallback remains for module-top-level literals owning their return home,
literal callbacks with nested Closure literals, literal callbacks declaring
parameters, dynamic callbacks, non-Closure invokables, custom same-name
selectors, and every other unsupported shape.

The guest carrier and its fixed 64 MiB stack remain unchanged.

Human-reported validation of the exact published candidate:

~~~text
FOCAL_SIX_TEST_CLASSES=PASS
B1_MARKERS=PASS
MAKE_TEST=PASS
GIT_DIFF_CHECK=CLEAN
SEPARATE_COMPILE_ONLY_STATIC_CHECK=NOT_RUN
COMPILE_COVERED_BY_TEST_GATES=YES
~~~

Known follow-up observations are retained as evidence:

- inline-body lexical lookups currently use the general runtime path instead of
  the captured-local fast path;
- the compile-time candidate detector emits an inline copy for every single
  zero-parameter literal send argument with no nested Closure even when the
  selected operation cannot consume B-prime; and
- the performance effect of both points is unmeasured.

Detailed evidence:
docs/project/evidence/PERF026/PERF026_B1_PLAT044_IFTRUE_INLINE_CALLBACK.md.
