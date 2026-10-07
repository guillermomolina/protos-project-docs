# PLAT043 — standard Boolean-control ownership investigation evidence

Date: 2026-10-01

## Publication identity

~~~text
WORK_ITEM=PLAT043/#763
PARENT=PERF025/#758
TRIGGER=PERF025-C2A

PRODUCT_REPOSITORY=guillermomolina/protos
PROTOS_REVISION=0a5115caddba8ebb7bb4275ce32441ab90938d3d
PROTOS_VERSION=0.3.133-SNAPSHOT

PRIOR_PLATFORM_DECISION=PLAT042/#760
BUG008=#681 CLOSED_DO_NOT_REOPEN

RESEARCH_TYPE=INVESTIGATION_ONLY
PRODUCT_CHANGES=NONE
SPECIFICATION_CHANGE=NO
~~~

## Question investigated

After PLAT042 B′ removed the universal semantic-wrapper CallTarget, why does the
retained 10,000-deep recursive driver still require a large guest stack, and
what is the smallest durable architecture change that can remove the residual
avoidable structured-control activation without changing Protos semantics?

## Current product topology

Revision-bound source inspection established the retained recursion shape:

~~~text
repeat(count, operation)
    -> (count > 0).ifTrue(callback)
    -> recursive repeat(count - 1, operation)
~~~

At each positive level:

~~~text
R_i = tagged semantic source root executing repeat
S_i = untagged structured/C-prime root handling prepared Boolean control
B_i = tagged semantic source root executing selected callback

R_i -> S_i -> B_i -> R_i+1 -> S_i+1 -> B_i+1 -> ...
~~~

The comparison and operation that produce the Boolean receiver complete before
the `S_i` activation. The ordinary `operation()` call in the callback
completes before recursive descent. Those do not accumulate across levels.

The accumulated live owners are the parent semantic invocation, the structured
Boolean helper and the selected callback.

## Callback de-structuring already exists

`ProtosStructuredDispatchLowerer.emitScopedPreparedInvocation(...)` checks the
child `PreparedClosureCall.requiresStructuredDispatch()`.

A selected Boolean callback that is an ordinary source call therefore follows
ordinary prepared invocation rather than entering a second structured dispatcher.

~~~text
CALLBACK_SPECIFIC_DESTRUCTURING=ALREADY_IMPLEMENTED
RESIDUAL_AVOIDABLE_OWNER=S_i
~~~

Candidate work must not claim a second win for machinery already provided by
PLAT042.

## Standard Boolean capability

Current code represents all five structured Boolean callback forms through one
finite prepared capability:

~~~text
PreparedBooleanCall

StructuredCallbackKind:
    IF_TRUE
    IF_FALSE
    IF_TRUE_IF_FALSE
    AND
    OR
~~~

The common state machine is conceptually:

~~~text
prepare standard Boolean call
    -> decide whether callback is selected

no selected callback
    -> return immediate standard result

selected callback
    -> prepare ordinary polymorphic invocation
    -> execute callback
    -> finish Boolean operation
        IF_TRUE / IF_FALSE / IF_TRUE_IF_FALSE
            return exact callback result
        AND / OR
            require exact canonical Boolean result
            return it or signal Error
~~~

This fact materially changes the anti-overengineering comparison: moving the
complete capability does not create five unrelated optimization mechanisms.

## Normative audit

The affected normative owners were reviewed in full or to the complete relevant
domain boundary:

- `spec/semantics/VALUES_AND_COLLECTIONS.md`
- `spec/semantics/CALLABLES.md`
- `spec/semantics/EXECUTION_AND_CONTROL.md`
- `spec/semantics/ERRORS.md`
- `spec/concurrency/FUTURES_AND_TASKS.md`

Constraints derived from them:

### Ordinary messages

`ifTrue`, `ifFalse`, `ifTrueIfFalse`, `and`, and `or` are ordinary
messages.

A valid optimization must occur after ordinary lookup/selection. Selector
spelling alone is insufficient authority.

### Callback invocation

A standard Boolean callback is not Closure-only. It must be ordinarily invokable
through the structural `candidate.call` protocol.

Callability remains path-sensitive: an unselected callback is not validated or
invoked, while argument-producing expressions have already followed ordinary
eager evaluation.

### Results

`ifTrue`, `ifFalse`, and `ifTrueIfFalse` return the selected callback's
exact normal result.

`and` and `or` validate the reached callback's normal result as exactly
canonical `true` or `false`.

There is no truthiness, implicit await, Future adoption or hidden conversion.

### Control/suspension

Boolean control introduces no Task, Future, handler, cleanup scope, return home,
lock, scheduler boundary, cancellation mask, checkpoint or hidden suspension
point.

A reached callback may encounter an explicit existing suspension point. Physical
suspension/resume must continue at the same semantic point without replay or
duplicate effects.

Error, non-local return, InvalidReturn and cancellation propagate under their
ordinary rules.

### Structured ownership

Ordinary synchronous callback activation remains inside the same task-scoped
structured execution scope. Merely moving Boolean sequencing between generated
Bytecode interpreter roles cannot create a new structured owner.

## Protos implementation audit

The following product surfaces were materially inspected at the baseline:

- `src/main/java/com/guillermomolina/protos/execution/CanonicalToBytecodeLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStructuredDispatchLowerer.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosTaskCPrimeEntryExecution.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosStandaloneHostedExecution.java`
- `src/main/java/com/guillermomolina/protos/execution/ProtosGuestCarrier.java`
- standard Boolean protocol implementation/classification surfaces used by the
  prepared-call path.

Key implementation facts:

1. the tagged semantic source interpreter is already Bytecode-DSL resumable;
2. structured prepared invocation currently crosses
   `EnterNestedStructuredDispatch`;
3. the untagged task C-prime entry owns the structured lowerer;
4. the Boolean callback child already avoids nested structured dispatch when it
   is ordinary;
5. prepared-call completion retains exact control/return-home behavior;
6. the dedicated guest carrier provisions 64 MiB and also serializes session
   guest operations.

## Truffle 25.4 mechanism audit

### Tail-call handlers

`GenerateBytecode.enableTailCallHandlers` implements one-compilation-per-
bytecode-handler tail threading between interpreter handler stubs.

It is not guest-language CallTarget tail-call optimization and does not remove
the `R_i`, `S_i`, or `B_i` guest activations.

Evidence:

- GraalVM `GenerateBytecode` API;
- Oracle Graal `OneCompilationPerBytecodeHandler.md`.

### DirectCallNode

`DirectCallNode` provides a stable target call site and can aid partial
evaluation/inlining, but compilation/inlining is not an interpreter-mode stack
safety contract.

A correct PERF025-C2 architecture cannot depend on the optimizer deciding to
inline away the helper.

### Continuations

Bytecode DSL yield/continuation support can explicitly preserve interpreter
state across suspension.

No supported public Truffle primitive was established that means "replace this
current guest CallTarget activation with a child and later resume the current
guest activation" as transparent guest tail-call elimination.

A generic trampoline would therefore be a new Protos runtime protocol.

## Cross-runtime evidence

The comparison was used as implementation evidence, not semantic authority.

### Smalltalk / Pharo

Pharo's compiler recognizes the standard Boolean-control message family
(`ifTrue:`, `ifFalse:`, two-branch forms, `and:`, `or:`) as optimized
control and lowers it to branch-style control where valid.

This is the closest precedent for separating ordinary message-level semantics
from physical conditional control.

Protos cannot copy selector/syntax recognition directly because Protos must
retain current override/copy/alias/`methodHome` behavior. The transferable
lesson is only:

~~~text
semantic dispatch authority first
physical branch/control representation second
~~~

Repository authority observed during investigation:

~~~text
pharo-project/pharo
revision=edd429b0508f80b841eff3d1feb83b96ef6ac746
~~~

### TruffleSqueak

TruffleSqueak's bytecode interpreter has direct conditional-jump handlers and
falls back to Smalltalk Boolean protocol/error behavior when the condition is
not an accepted Boolean.

This supports local physical conditional control inside the semantic interpreter
for a message-oriented language.

~~~text
hpi-swa/trufflesqueak
revision=4ff18896b3a243c9aabde6217b0d9b5e57e91876
~~~

### GraalPy

Current GraalPy Bytecode DSL source uses `enableYield=true` and bytecode-level
short-circuit control support. Resumability and local short-circuit control
coexist in one generated semantic interpreter.

~~~text
oracle/graalpython
revision=ee204f260d0295b7478ce006afd955e313265a70
~~~

### GraalJS / TruffleRuby / Pkl / Espresso / Sulong

Representative inspected runtimes keep ordinary condition/branch sequencing in
the current semantic frame/root when their language semantics permit it.

This does not imply Protos can bypass its message lookup. It establishes that an
extra helper CallTarget is not a Truffle requirement merely because control is
conditional or resumable.

Observed repository revisions:

~~~text
oracle/graaljs=4c2b8dd51b23919c1d4ba72aa07ad45fbfaac2f7
truffleruby/truffleruby=c734f26543003fefd4519a29adbca62d0c711a2d
apple/pkl=a5bcacf91d04847283de67be7178ca89e448297e
oracle/graal=0a75c47f934728d20de1bfceae868500d2ab76e2
~~~

## Candidate set

### Candidate A — local IF_TRUE / IF_FALSE only

Intended topology for the retained benchmark:

~~~text
R_i -> B_i -> R_i+1
~~~

Advantage: smallest direct patch.

Weakness: splits physical ownership inside one existing Boolean prepared
capability and makes later Boolean kinds reopen the same boundary.

### Candidate B — complete PreparedBooleanCall local

Move:

~~~text
IF_TRUE
IF_FALSE
IF_TRUE_IF_FALSE
AND
OR
~~~

to the semantic interpreter and leave every non-Boolean structured family in the
PLAT042 untagged owner.

Advantage: bounded to one already-existing finite capability; no new semantic or
continuation mechanism; removes the established helper root from all standard
Boolean structured control.

Risk: still leaves source/callback recursion and therefore may not suffice to
retire the carrier.

### Candidate C — canonical-only Boolean local

Canonical exact selections execute locally; copied/aliased/non-canonical
standard wrappers keep the helper.

Advantage: conservative physical reading of PLAT032.

Weakness: creates two physical machines for the same implementation provenance
without a semantic requirement. PLAT032 identity/provenance can be preserved
without retaining the helper.

### Candidate D — generic trampoline/state handoff

Could physically unwind the structured parent before executing its callback and
resume the parent state later.

Advantage: potentially broad stack flattening.

Weakness: new internal control protocol touching Error, ensure, cancellation,
ReturnHome, continuation ownership, tooling and nested structured calls; no
supported Truffle primitive establishes it automatically.

### Candidate E — status quo

Keep PLAT042 B′ and the 64 MiB dedicated carrier.

Advantage: lowest immediate technical risk.

Weakness: leaves the current PERF025-C2 target unresolved.

## GITHUB010 scorecard

Score is 1–5, higher is better. Confidence is H/M/L.

| Criterion | A | B | C | D | E |
| --- | --- | --- | --- | --- | --- |
| Correctness / invariants | 4/H | 4/M | 5/M | 2/L | 5/H |
| Protos alignment | 3/H | 5/H | 4/M | 2/M | 3/H |
| Pay for present need | 5/H | 5/H | 4/M | 1/H | 1/H |
| Grow as needed | 3/M | 5/H | 4/M | 3/M | 2/M |
| Future-option resilience | 4/M | 5/M | 5/M | 4/M | 3/M |
| Scalability | 4/M | 4/M | 4/M | 5/L | 2/H |
| Conceptual simplicity | 3/H | 4/H | 3/M | 1/H | 5/H |
| Portability / freedom | 4/M | 5/M | 4/M | 3/L | 3/M |
| Runtime/resource cost | 5/M | 5/M | 4/M | 3/L | 1/H |
| Failure / operability | 4/M | 4/M | 4/M | 1/L | 5/H |
| Reversibility / migration | 5/H | 5/H | 4/H | 2/M | 3/M |
| Evidence maturity / risk | 4/M | 4/M | 4/M | 1/H | 5/H |

Totals:

~~~text
A=48
B=55
C=49
D=28
E=38
~~~

## Anti-overengineering gate

~~~text
A=PASS_WITH_RESERVATION
  minimal, but splits an existing capability by kind

B=PASS
  moves one finite existing capability; creates no new institution

C=PASS_PARTIAL
  conservative but adds a provenance-dependent physical split not demanded by semantics

D=FAIL
  creates a general control institution before present evidence requires it

E=PASS_TECHNICALLY
  no overengineering, but does not solve PERF025-C2
~~~

The Smalltalk precedent does not justify moving `while` or the entire
structured dispatcher.

For Protos `while`, one helper entry owns many loop iterations. In the retained
recursive `ifTrue` shape, a new helper activation accumulates at every recursion
level. The performance/stack motivation is therefore materially different.

## GITHUB021 invariant/delta review

~~~text
PLAT017=
  PASS
  lookup/selection remains first; no canonical eligibility widening

PLAT021=
  PASS
  no second unwind machine; Error/NLR/ensure/cancellation unchanged

PLAT026=
  PASS
  truthful semantic RootTag remains; no helper promotion/manual tag

PLAT028=
  PASS
  surviving sequencing remains guest Bytecode resumable state, not Java/native continuation state

PLAT032=
  PASS
  selected wrapper identity, receiver, methodHome, activation and tooling identity preserved

PLAT036_PERF013=
  PASS
  no lexical ownership or captured-state authority change

PLAT040=
  PASS
  guarded specialization remains post-selection

PLAT041=
  PASS
  Object-body/lexical/root-group invariants unchanged

PLAT042=
  EXPLICIT_DELTA_REQUIRED
  prepared standard Boolean control becomes the sole narrow physical-owner exception

NEW_PROTOS_SEMANTICS=NO
SPECIFICATION_CHANGE=NO
DECISION_INVARIANT_CONSISTENCY=PASS
~~~

## Selected result

The project owner approved Candidate B after the complete decision packet:

~~~text
APPROVED_CANDIDATE=B

STANDARD_BOOLEAN_PREPARED_ORCHESTRATION_OWNER=
  TAGGED_SEMANTIC_BYTECODE_INTERPRETER

OTHER_STRUCTURED_PREPARED_OWNER=
  UNTAGGED_STRUCTURED_CPRIME_INTERPRETER

NEW_CONTINUATION_KIND=NO
NEW_SEMANTICS=NO
SPECIFICATION_CHANGE=NO
~~~

## Strongest counterargument

After Candidate B:

~~~text
R_i -> B_i -> R_i+1
~~~

still remains.

Therefore the decision can improve stack topology while failing to eliminate the
need for a large stack at depth 10,000.

The old approximately-32-MiB threshold cannot answer the new topology's stack
requirement.

This is why carrier removal and stack-size reduction remain explicitly outside
PLAT043 and must follow implementation measurement.

## Next work

~~~text
NEXT_SLICE=PERF025-C2B
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos

IMPLEMENTATION_ASSUMPTION=
  repo already at current HEAD

CARRIER_CHANGE_IN_THIS_SLICE=NO
BENCHMARK_SHAPE_CHANGE=NO
BUG008_REOPEN=NO
~~~

## External references

- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/GenerateBytecode.html
- https://github.com/oracle/graal/blob/master/truffle/docs/OneCompilationPerBytecodeHandler.md
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/nodes/DirectCallNode.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ContinuationResult.html
- https://www.graalvm.org/truffle/javadoc/com/oracle/truffle/api/bytecode/ContinuationRootNode.html
- https://github.com/pharo-project/pharo
- https://github.com/hpi-swa/trufflesqueak
- https://github.com/oracle/graalpython
- https://github.com/oracle/graaljs
- https://github.com/truffleruby/truffleruby
- https://github.com/apple/pkl

## Cross references

- `guillermomolina/protos#763` — PLAT043.
- `guillermomolina/protos#758` — PERF025.
- `guillermomolina/protos#760` — PLAT042.
- `guillermomolina/protos#681` — historical BUG008.
- `docs/project/decisions/platform/PLAT043_STANDARD_BOOLEAN_CONTROL_INTERPRETER_OWNERSHIP_BOUNDARY.md`.
- `docs/project/decisions/platform/PLAT042_STRUCTURED_DISPATCH_INTERPRETER_OWNERSHIP_BOUNDARY.md`.
- `docs/project/evidence/PERF025/PERF025_C1C_PLAT042_B_PRIME_CUTOVER.md`.
