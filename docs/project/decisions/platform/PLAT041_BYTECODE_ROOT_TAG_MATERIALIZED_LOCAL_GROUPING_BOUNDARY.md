# PLAT041 — Bytecode RootTag and materialized-local grouping boundary

Status: **RATIFIED**

Selected architecture: **Candidate C′ — semantic-only source-root universe + inline Object-construction body + compact untagged infrastructure interpreter**.

Approval: explicit project-owner approval on 2026-10-01:

~~~text
Apruebo C′ para PLAT041
~~~

Decision Issue: `guillermomolina/protos#759`

Parent workstream: `PERF025 / guillermomolina/protos#758`

Product baseline:

~~~text
PROTOS_REVISION=6811d0cef3735d39ffd3801b3bae6ef48318bb66
PROTOS_VERSION=0.3.131-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.4.4.1.1
~~~

Nature: durable non-normative JVM/Truffle implementation-architecture decision.
Observable Protos semantics remain owned by the normative specification.

## Decision

Protos adopts Candidate C′.

The production Bytecode topology SHALL converge to:

~~~text
full semantic source interpreter
    top-level/public source root      -> automatic StandardTags.RootTag
    module-body root                  -> automatic StandardTags.RootTag
    Closure activation root           -> automatic StandardTags.RootTag

    Object construction body
        -> NOT a physical Bytecode root
        -> inline structured/resumable region
        -> owns a construction ProtosActivation as private execution state
        -> remains NOT a lexical execution context
        -> remains NOT a debugger/function root

compact infrastructure interpreter
    Task C-prime entry and bounded C-prime I/O orchestration roots
        -> automatic RootTag disabled
        -> implementation/helper roots only
~~~

The current `ProtosSemanticBytecodeRootNode` wrapper is transitional machinery,
not durable architecture. Once the source interpreter itself is a truthful
semantic-root-only generated interpreter, ordinary source-backed Closure
activation does not require a second semantic wrapper `CallTarget`.

## Trigger

PERF025-C attempted to remove the ordinary source-backed Closure path:

~~~text
caller source Bytecode root
    -> semantic RootTag wrapper CallTarget
        -> untagged source/helper CallTarget
~~~

The first implementation attempt correctly stopped before modifying product
source because current Truffle automatic root tagging applies to every root
created by one generated interpreter, while PERF013 deliberately grouped
semantic Closure roots and Object-body helper roots in one
`BytecodeRootNodes` generation to permit `MaterializedLocalAccessor` captured
access.

The previously closed PLAT026 follow-up had not reached that implementation
constraint. PLAT041 resolves it without weakening either the semantic tooling
boundary or the captured-local optimization.

## Semantic/tooling root authority

PLAT026 remains authoritative and is not reopened.

A truthful semantic source root is a Protos top-level/public-source unit,
module-body activation, or Closure activation. Those roots use the Bytecode
DSL's automatic `RootTag`.

Object construction/body evaluation remains implementation machinery. It does
not become a function, Closure, lexical scope, semantic stack frame, or
debugger-visible root merely because the current implementation used a
`RootCallTarget`.

Therefore Candidate C′ removes the physical Object-body helper root instead of
promoting it.

The following remain unchanged:

- `RootBodyTag` stays deferred/unprovided;
- StatementTag, ExpressionTag and CallTag membership retain their current
  approved meanings;
- source ownership remains PLAT004-authoritative;
- tooling correspondence remains PLAT005/PLAT026/PLAT034-authoritative;
- debugger value/scope projection remains PLAT013/PLAT015-authoritative; and
- no global mutable root registry or debugger lock is introduced.

## Inline Object-construction boundary

An Object literal still creates exactly the same conceptual construction
activation.

Candidate C′ changes only where its body instructions are physically hosted.

The containing semantic Bytecode root SHALL keep the construction activation and
constructed Object in resumable root-local state while emitting the Object body
as an inline structured region.

Guest operations inside that region observe the construction activation as the
current activation, preserving:

- `this` / receiver identity;
- construction `context` identity;
- Object local slot creation/assignment;
- enclosing lexical lookup/capture;
- captured receiver and method home;
- return home;
- Core prelude;
- Task identity or direct/deferred dynamic-control identity;
- Error/handler/ensure behavior;
- non-local return behavior;
- Actor/module/execution-domain ownership; and
- suspension/resumption.

The construction activation remains explicitly non-lexical. A Closure
materialized inside an Object body captures the enclosing genuine lexical
contexts, not the constructed Object as a lexical scope.

## Continuation compatibility

PLAT014 remains authoritative.

An Object-body suspension no longer needs a private child-root continuation
solely because the body was physically outlined as a root. Instead, the
containing semantic Bytecode root's existing Bytecode DSL continuation retains
the construction activation, object and intermediate values in normal resumable
locals.

This introduces no new public continuation kind, no replay, no AST fallback,
no Task bridge, and no host-thread continuation semantics.

Nested ordinary Closure calls remain ordinary PLAT014 composed calls.

## PERF013 invariant and explicit physical delta

PERF013 established two distinct facts:

1. same-generation lexical owner + nested Closure + matching retained frame may
   use `MaterializedLocalAccessor`; and
2. cross-generation access must use the existing runtime lexical-authority
   fallback because frame/descriptor identity cannot be reconstructed safely.

Candidate C′ preserves both.

The explicit physical A3 delta is:

~~~text
BEFORE:
  lexical owner root
      + Object-body helper root
      + method Closure root
  share one BytecodeRootNodes group

C_PRIME:
  lexical owner root
      + method Closure root
  share one BytecodeRootNodes group

  Object body
      = inline region, no root
~~~

The Object-body helper root was never a genuine lexical execution context.
Removing that physical root therefore does not remove a lexical owner.

The owner-approved PERF013 outcome that matters to execution remains:

~~~text
same-generation owner BytecodeLocal
+ nested Closure
+ matching owner MaterializedFrame
    -> MaterializedLocalAccessor fast path

cross-generation / incompatible frame identity
    -> runtime lexical-authority fallback
~~~

No captured value is copied into a second authority and capture-by-reference
semantics remain unchanged.

## Current-activation lowering authority

Implementation SHALL avoid scattering special Object-body semantics across every
operation that currently emits frame argument 0.

The selected migration shape is one lowering authority equivalent in role to:

~~~text
emitCurrentActivation(builder)
~~~

For an ordinary semantic root it reads the root activation from frame argument
0. Inside an inline Object-construction region it reads the construction
activation from the region's selected local.

Nested Object bodies select their own construction activation. Nested Closures
remain separate semantic roots and therefore begin with their own ordinary root
activation.

The exact Java helper/class name is implementation-local.

## Infrastructure interpreter

Once Object bodies cease to be physical helper roots, the canonical/source
interpreter's generated-root population can contain only semantic source roots
and can truthfully enable automatic root tagging.

Current product inspection found six bounded production builders outside the
canonical lowerer that still create untagged C-prime infrastructure roots:

- `ProtosTaskCPrimeEntryExecution`
- `ProtosTextWriterCPrimeExecution`
- `ProtosTextReaderCPrimeExecution`
- `ProtosIoReleaseCPrimeExecution`
- `ProtosBufferedByteReaderCPrimeExecution`
- `ProtosBufferedByteWriterCPrimeExecution`

These SHALL move to a compact untagged infrastructure generated interpreter
rather than forcing a second complete source-language interpreter.

Supported Bytecode DSL mechanisms such as `OperationProxy` or shared
specialization helpers MAY be used to avoid duplicating operation semantics.
The exact sharing mechanism, generated class names and internal builder names
remain implementation details.

## Rejected/currently unavailable alternatives

### A — two full tagged/untagged source interpreters

Not selected.

It preserves PLAT026 but partitions common Object/method lexical shapes across
generated groups, broadly degrading PERF013 same-generation materialized-local
access and generating two substantial source interpreters.

### B — retain the current semantic wrapper indefinitely

Not selected as the target architecture.

It remains a safe fallback during migration, but keeps the nested source
`CallTarget` topology that causes the BUG008 host-stack amplification PERF025-C
was created to remove.

### D — first-class per-root automatic RootTag classification

Preferred as a possible future simplification, but unavailable in the current
Truffle 25.4 API.

If a future supported Bytecode DSL API can classify semantic roots individually
inside one generated group while preserving automatic pre-prolog tagging,
Protos may replace internal C′ factoring without reopening PLAT041, provided all
tool-visible and lexical invariants remain identical.

### E — promote Object-body helper roots to semantic roots

Rejected.

That would contradict PLAT026 and expose an implementation frame that is not an
ordinary Protos function/Closure/call frame.

## Required invariants

1. Top-level/public-source, module and Closure activations remain truthful
   semantic roots.
2. Semantic source roots use automatic `StandardTags.RootTag`.
3. Object construction/body evaluation receives no `RootTag` and no
   guest-function/frame identity.
4. `RootBodyTag` remains deferred.
5. StatementTag, ExpressionTag and CallTag semantics remain unchanged.
6. Object construction remains non-lexical.
7. Closures created in Object bodies capture the enclosing genuine lexical
   context(s), never the Object construction activation as a lexical scope.
8. Same-generation lexical owner/nested Closure
   `MaterializedLocalAccessor` fast paths remain available.
9. Cross-generation accessor/frame mixing remains forbidden.
10. Runtime lexical-authority fallback remains semantically exact.
11. PLAT014 continuation, Error, ensure, cancellation and NLR behavior remains
    unchanged.
12. No AST/replay fallback is reintroduced.
13. No global mutable root classification, lexical-state or continuation
    registry is introduced.
14. Helper/infrastructure roots remain untagged by default.
15. Native Image generation and runtime compilation remain supported.
16. Ordinary execution without attached tooling pays no new global
    instrumentation coordination cost.
17. BUG008 remains historical closed work; PERF025 owns removal of its workaround
    cost.
18. Carrier retirement is not authorized until the double source-Closure target
    is actually gone and deep-recursion evidence proves the enlarged stack is no
    longer required.

## GITHUB021 invariant/delta result

~~~text
PLAT026_INVARIANT_DELTA=NONE
PLAT014_INVARIANT_DELTA=NONE

PERF013_PHYSICAL_A3_DELTA=OBJECT_HELPER_ROOT_REMOVED
PERF013_MATERIALIZED_LOCAL_INVARIANT=PRESERVED
PERF013_CAPTURE_SEMANTICS=PRESERVED

DECISION_INVARIANT_CONSISTENCY=PASS
NEW_PROTOS_SEMANTICS=NO
~~~

The disclosed PERF013 delta changes physical backend topology, not the accepted
captured-state authority or optimization invariant.

## Implementation release

Ratification releases PERF025-C implementation in bounded slices.

### C1a — current-activation lowering seam

Introduce one backend-private current-activation emission authority and route the
canonical lowerer through it while retaining the current physical topology.

No RootTag topology or Object-body root removal belongs in C1a.

### C1b — inline Object-body execution

Remove the Object-body helper `RootCallTarget` and emit the construction body
as a structured/resumable region using the construction activation selected by
the C1a seam.

Retain the semantic wrapper during this slice if that keeps the patch bounded.

### C1c — semantic source-root tagging cutover

Make the canonical source interpreter semantic-root-only and enable automatic
RootTag. Remove the now-redundant semantic wrapper/double source-Closure
`CallTarget`. Move the bounded C-prime infrastructure builders to the compact
untagged interpreter and update DAP/native generated-structure evidence.

### C2 — BUG008 carrier retirement

Only after C1c proves deep recursion no longer has the two-target stack
amplification may PERF025 remove/reduce the BUG008-specific 64 MiB carrier
requirement while preserving reusable-session serialization and then perform
the required before/after measurement.

## Scalability and future endurance

C′ keeps runtime state local to the executing semantic root/continuation. It
does not scale physical helper roots, registries or locks with Object literals,
Tasks, Actors, Processes or Contexts.

The source interpreter remains one full generated semantic interpreter rather
than two full source interpreters. Infrastructure helpers remain a bounded
separate untagged substrate.

The decision also reduces accidental physical stack depth: an Object body no
longer adds a helper-root call solely for backend organization, and C1c removes
the semantic wrapper around ordinary source roots.

## Strongest argument against C′

C′ is a multi-slice migration rather than a small local patch. It requires a
current-activation lowering seam, inline resumable Object construction, an
infrastructure-interpreter split, tooling/native generated-structure updates and
focused control/capture regression evidence.

That cost is accepted because it removes accidental root machinery while
preserving both already-ratified semantic tooling truthfulness and PERF013's
same-generation captured-local optimization.

## Approval record

The project owner explicitly approved Candidate C′ on 2026-10-01 after the
PLAT041 decision packet compared Candidates A, B, C′, D and E and disclosed the
PERF013 physical A3 delta.

The exact approval was:

~~~text
Apruebo C′ para PLAT041
~~~

No language/specification change is authorized by this decision.

## References

- `guillermomolina/protos#759` — PLAT041 decision Issue.
- `guillermomolina/protos#758` — PERF025 parent/consumer.
- `guillermomolina/protos#690` — historical PLAT026 follow-up, whose local-
  optimization readiness conclusion is superseded as implementation authority.
- `guillermomolina/protos#724` — PERF013.
- `guillermomolina/protos#681` — historical BUG008; remains closed.
- PLAT004, PLAT005, PLAT008, PLAT014, PLAT026, PLAT034, PLAT036 and PLAT040.
- `docs/project/evidence/PLAT041/PLAT041_ROOT_TAG_MATERIALIZED_LOCAL_DECISION_EVIDENCE.md`.

## Post-ratification implementation discovery — PLAT042 gate

PERF025-C1a+C1b were published on 2026-10-01 at:

~~~text
PROTOS_REVISION=595d547b2e9714a185a3cfceadf74565229f43e7
PROTOS_VERSION=0.3.132-SNAPSHOT
~~~

Those slices implement and preserve the approved C′ facts for current-activation
lowering and inline Object-body execution.

C1c then stopped before implementation because the ratification's
"compact untagged infrastructure interpreter" estimate counted only operations
emitted directly by the six C-prime builder classes. The Task C-prime entry also
calls `CanonicalToBytecodeLowerer.emitPreparedInvocationForRuntime(...)`, whose
transitive structured-dispatch lowering materially changes the footprint.

Revision-bound static inspection at the published product checkpoint gives:

~~~text
SOURCE_LOWERER_BUILDER_OPERATION_NAMES=203
SIX_CPRIME_DIRECT_BUILDER_OPERATION_NAMES=53
PREPARED_STRUCTURED_DISPATCH_TRANSITIVE_OPERATION_NAMES=144
ACTUAL_HELPER_UNION_OPERATION_NAMES=183
HELPER_UNION_VS_SOURCE_LOWERER=90.1%
PROTOS_BYTECODE_ROOT_NODE_OPERATION_ANNOTATIONS=223
~~~

Therefore the PLAT041 C1c phrase "compact untagged infrastructure interpreter"
is **superseded as implementation authorization**. It is not evidence that C′'s
already-implemented Object-body or lexical/tooling invariants were wrong.

The remaining structured-dispatch ownership question is promoted to:

~~~text
PLAT042=guillermomolina/protos#760
PERF025_C1C=BLOCKED_BY_PLAT042
BUG008=#681 CLOSED_DO_NOT_REOPEN
~~~

Until PLAT042 is explicitly selected and durably ratified, PLAT041 does not
authorize choosing between a near-full duplicate interpreter, an out-of-line
structured dispatcher, or another topology for C1c.

## PLAT042 ratified amendment to the C1c helper-ownership clause

PLAT042 / `guillermomolina/protos#760` was explicitly approved on 2026-10-01
after implementation falsified this decision's original assumption that the
remaining C-prime interpreter could be a compact bounded helper.

PLAT041 remains authoritative for C1a/C1b and the semantic invariants selected
here. The following C1c implementation phrase is superseded:

~~~text
OLD:
  semantic source interpreter
  + compact untagged infrastructure interpreter
~~~

The later authority is PLAT042 Candidate B′:

~~~text
CURRENT:
  tagged semantic source interpreter
  + one untagged structured-dispatch/C-prime owner
  + structured-only helper CallTarget boundary
~~~

This amendment changes no PLAT026, PLAT014 or PERF013 invariant and introduces
no Protos semantic/specification change.

See
`docs/project/decisions/platform/PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md`.

