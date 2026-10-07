# PLAT039 — Truffle runtime-compilation boundary decision evidence

Status: FINAL DECISION EVIDENCE

Decision: PLAT039 / guillermomolina/protos#716

Ratified candidate: **C — evidence-gated PE-visible guest kernel with narrow host gateways**

Approval provenance:

~~~text
DATE=2026-09-25
OWNER=guillermomolina
APPROVAL="Approve Candidate C"
~~~

Evidence identities:

~~~text
PROTOS_REVISION=d4ac1c7be600c00c785dc9742dc7aeaeab17eaec
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
GRAALVM_SOURCE_TAG=vm-25.3.4.1
GRAALVM_SOURCE_REVISION=7b025988a922a73286d1326e1eddc1ca39d3f569
PRELIMINARY_EVIDENCE=
  docs/project/evidence/PLAT039/PLAT039_TRUFFLE_RUNTIME_COMPILATION_BOUNDARY_PRELIMINARY_EVIDENCE.md
~~~

Current `guillermomolina/protos` `main` was identical to the pinned baseline
during the final research pass.

## Exact GraalVM 25.3.4.1 findings

In
`substratevm/src/com.oracle.svm.truffle/src/com/oracle/svm/truffle/TruffleFeature.java`
at `vm-25.3.4.1`:

- `allowRuntimeCompilation()` rejects `runtimeCompilationForbidden()` methods
  and blocklisted methods;
- `runtimeCompilationForbidden()` returns true for `@TruffleBoundary`;
- the blocklist includes `BigInteger`, `BigDecimal`, `Collection`, `List`,
  `Map`, `HashMap`, `ConcurrentHashMap`, `IdentityHashMap`, `Iterable`,
  `Iterator` and generic functional interfaces including `BiFunction`,
  `Function`, `Predicate` and `Supplier`;
- `BigInteger.signum()` is explicitly removed from the blocklist;
- temporary target allowlisting includes selected `Function`, `Predicate`,
  `List`, `Iterator` and `Iterable` operations; and
- a reported violation is constructed only for a blocklisted implementation
  method that is an actual runtime-compilation candidate whose target is not
  temporarily allowlisted.

This establishes the required distinction:

~~~text
BLOCKLISTED
!=
REACHABLE_FROM_RELEVANT_GUEST_ROOT
!=
SURVIVES_PE_IN_HOT_COMPILED_GRAPH
~~~

No candidate may skip those distinctions.

## Normative constraints

The final decision was checked against the primary normative owners.

Required semantic invariants include:

- execution contexts remain objects;
- lexical lookup and ordinary object lookup remain distinct;
- object slots are receiver state, not lexical variables;
- parameter/default/rest binding remains ordinary Closure invocation semantics;
- receiver and `methodHome` behavior remain exact;
- guest execution and guest callbacks remain ordinary Protos execution;
- Future/Task/Actor/C-prime control, cancellation and lifecycle semantics remain
  observable according to their normative owners;
- Integer remains unbounded; and
- physical host representation is non-normative when observable behavior is
  preserved.

These constraints reject broad boundaries around parameter binding, ordinary
send/lookup, continuation resumption or guest callback execution.

## Final gateway inventory

| Gateway | Role | Class | Required consequence |
|---|---|---|---|
| Bytecode ordinary send/call/lookup | guest dispatch | KEEP_PE | never broad-boundary |
| `bindParameters` | callable semantics | KEEP_PE | parameter/default/rest work stays visible |
| frame-backed lexical access | lexical semantics | KEEP_PE | preserve I068 authority |
| ordinary `ProtosValueLookup.lookup()` | object semantics | KEEP_PE | do not hide lookup |
| member-read Closure materialization | post-lookup materialization | BOUNDARY | existing narrow seam remains valid |
| module cache hit | resolved guest path | KEEP_PE | execute cached target visibly |
| module cache miss | source/runtime service | SPLIT | isolate miss path |
| source load/parse/canonicalize/lower | cold compiler | BOUNDARY | return target before guest execution |
| Integer operation selection/checks | guest arithmetic semantics | KEEP_PE | explicit operation path |
| individual `BigInteger` operation | Java numeric leaf | BOUNDARY | narrow leaf helper |
| `ProtosBinary64Rounding` BigInteger-heavy calculation | Java numeric leaf | BOUNDARY | isolate opaque numeric algorithm |
| call argument preparation | guest call semantics | KEEP_PE | no broad boundary |
| hot `List`/`ArrayList` carrier | implementation representation | EVIDENCE-GATED SPLIT | replace only if it survives PE/problem trace |
| ordinary slot backing map | implementation representation | EVIDENCE-GATED SPLIT | no automatic rewrite |
| native suspension + `Supplier` | mixed continuation/host carrier | SPLIT | explicit state + narrow host leaf |
| I/O operation/release suspension | mixed lifecycle/host carrier | SPLIT | guest outcome outside boundary |
| physical NIO/socket/stream operation | host effect | BOUNDARY | narrow backend leaf |
| Process environment/host acquisition | host input | BOUNDARY/SPLIT | snapshot host work; preserve guest Process state |
| Actor worker enqueue/scheduling | host execution service | BOUNDARY/SPLIT | guest handler outside opaque leaf |

## Required falsification results

### Could Candidate B be sufficient?

Partly.

Candidate B is sufficient for pure host/cold leaves such as parsing, physical I/O
and individual `BigInteger` operations.

It is not sufficient as the durable rule for methods that already mix guest
semantics and blocklisted generic carriers. Such methods require SPLIT.

### Are hot collection replacements premature?

Yes unless compiler/reachability evidence shows the representation survives PE or
prevents Native Image runtime compilation.

This objection changed the preliminary Candidate C and became part of the
ratified evidence gate.

### Can temporary allowlisting make a refactor unnecessary?

For a concrete method at a concrete GraalVM version, yes.

It cannot be Protos architecture authority because the source explicitly treats
this area as temporary migration support.

### Can PE already eliminate the problematic structure?

Yes. That is why current graph/reachability evidence is required before changing
an ordinary hot representation.

### Is a broad boundary harmless?

Not accepted. It can hide exactly the guest work Truffle must specialize:
dispatch, parameter binding, continuation decisions and guest callbacks.

### Can refactor cost exceed the value?

Yes for speculative representation rewrites. Candidate C therefore gates those
changes on current evidence and does not pre-authorize a repository-wide rewrite.

### Does Candidate C tie Protos semantics to GraalVM?

No observable semantic rule changes. KEEP_PE/BOUNDARY/SPLIT is implementation
architecture; a future backend may map the same guest-kernel/host-service
separation to different mechanisms.

## Comparative evidence

The survey covered the relevant Truffle implementation space required by
GITHUB010, including SimpleLanguage, GraalJS, GraalPy, TruffleRuby, Espresso,
Sulong/LLVM, TruffleSqueak, GraalWasm and FastR, plus Apple Pkl where comparable.

The strongest exact-version precedent is SimpleLanguage's pattern of retaining
guest numeric specialization while wrapping individual `BigInteger` operations
in narrow `@TruffleBoundary(allowInlining = true)` helpers.

Across the wider Truffle implementations, the recurring architecture is to keep
guest dispatch/execution in the Truffle optimization surface while isolating
parsing/loading, native/host calls, physical I/O and other runtime services.

Apple Pkl is materially comparable for module/source loading boundaries but is
not treated as authority for Protos continuation architecture.

Non-Truffle comparison included tracing/JIT and optimized-runtime precedents such
as PyPy and V8. They reinforce the general distinction between optimizer-visible
guest kernels and explicit residual/slow/runtime services; the principle is not
unique to Truffle.

## GITHUB010 scoring

Arithmetic totals were not used to select the candidate.

| Dimension | B — leaf boundaries only | C — evidence-gated PE kernel |
|---|---|---|
| correctness / invariant preservation | 3/5 MEDIUM | 5/5 HIGH |
| Protos alignment | 4/5 HIGH | 5/5 HIGH |
| present-need proportionality | 5/5 HIGH | 4/5 MEDIUM |
| incremental growth | 3/5 MEDIUM | 5/5 HIGH |
| future-option resilience | 2/5 MEDIUM | 5/5 HIGH |
| scalability | 3/5 MEDIUM | 4/5 MEDIUM |
| conceptual simplicity | 5/5 HIGH | 4/5 HIGH |
| portability / implementation freedom | 3/5 MEDIUM | 4/5 MEDIUM |
| runtime / resource cost | 4/5 MEDIUM | 4/5 MEDIUM |
| failure / operability | 3/5 MEDIUM | 4/5 HIGH |
| deferral / reversibility / migration | 3/5 MEDIUM | 4/5 MEDIUM |
| evidence maturity / implementation risk | 3/5 MEDIUM | 4/5 MEDIUM |

Anti-overengineering result:

~~~text
REPOSITORY_WIDE_COLLECTION_PURGE=REJECTED
ORDINARY_SLOT_STORAGE_REDESIGN_WITHOUT_EVIDENCE=REJECTED
SMALL_INT_REDESIGN_REQUIRED_NOW=NO
PAY_FOR_WHAT_YOU_NEED=PASS
GROW_AS_YOU_NEED=PASS
FUTURE_COMPATIBLE=YES
FUTURE_PREIMPLEMENTED=NO
SMALLEST_SUFFICIENT_SOLUTION=PASS
~~~

Underengineering result:

~~~text
B_ONLY=INSUFFICIENT_AS_DURABLE_MIXED_GATEWAY_RULE
~~~

## Future stress result

Candidate C was checked against many Contexts, many Tasks/Actors/Processes,
multicore execution, Native Image, JVM execution, future Truffle upgrades,
alternative backends, debugger/instrumentation, large argument counts, large
object slot sets, large Integers, heavy I/O, module-heavy programs and
cancellation/unwind/failure.

No case requires a new Protos semantic category.

The main deliberately deferred implementation choices are concrete argument
carrier layout and ordinary object-slot backing representation when current
compiler evidence does not require changing them.

## I068 and PERF010 interaction

~~~text
I068_INTERACTION=
  PRESERVED AND COMPLEMENTARY;
  FRAME-BACKED LEXICAL AUTHORITY REMAINS CORRECT;
  NO REOPEN OR REWORK REQUIRED

PERF010_INTERACTION=
  ARCHITECTURAL TARGETS IDENTIFIED;
  DOMINANT CAUSE NOT ESTABLISHED;
  ATTRIBUTABLE FRACTION NOT ESTABLISHED;
  CAUSAL GRAPH/LIFECYCLE + TIMING MEASUREMENT REQUIRED
~~~

## Final decision packet

~~~text
PLAT039_STATUS=RESEARCH_COMPLETE

TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED

GUEST_HOT_PATH_BOUNDARIES_IDENTIFIED=YES
HOST_RUNTIME_BOUNDARIES_IDENTIFIED=YES

PE_GATEWAYS=
  ORDINARY BYTECODE SEND/CALL/LOOKUP;
  PARAMETER/DEFAULT/REST BINDING;
  FRAME-BACKED LEXICAL ACCESS;
  ORDINARY OBJECT LOOKUP SEMANTICS;
  MODULE CACHE HIT;
  RESULTING/CACHED GUEST CALLTARGET EXECUTION;
  FUTURE/TASK/ACTOR/C-PRIME SEMANTIC STATE;
  GUEST CONTINUATION/RESUMPTION DECISION;
  GUEST CALLBACK/CALLTARGET EXECUTION

BOUNDARY_GATEWAYS=
  SOURCE LOAD/PARSE/CANONICALIZE/LOWER;
  PHYSICAL FILESYSTEM/NETWORK/STREAM;
  HOST ENVIRONMENT/PROCESS ACQUISITION;
  HOST WAIT/SCHEDULING/RESOURCE LEAVES;
  INDIVIDUAL BIGINTEGER OPERATIONS;
  BIGINTEGER-HEAVY BINARY64 ROUNDING;
  EXPLICIT MEMBER-READ CLOSURE MATERIALIZATION

SPLIT_GATEWAYS=
  MODULE HIT/MISS;
  PREPARED CLOSURE ENTER-NATIVE;
  CONTINUATION/NATIVE SUSPENSION;
  GENERIC SUPPLIER GUEST-HOT CARRIERS;
  ARGUMENT-VECTOR REPRESENTATION WHERE CURRENT PE EVIDENCE REQUIRES;
  FUTURE/IO/RELEASE SUSPENSION;
  ACTOR HOST DISPATCH VS GUEST HANDLER;
  PORTABLE FILE/OPEN STATE VS PHYSICAL NIO;
  ORDINARY SLOT REPRESENTATION ONLY IF CURRENT EVIDENCE REQUIRES

BROAD_ENTER_NATIVE_BOUNDARY_ACCEPTABLE=NO
BROAD_RESUME_BOUNDARY_ACCEPTABLE=NO
BIND_PARAMETERS_MUST_REMAIN_PE=YES

SOURCE_COMPILATION_RUNTIME_BOUNDARY=
  CACHE HIT + RESULTING GUEST TARGET KEEP_PE;
  CACHE MISS COMPILER WORK BOUNDARY;
  GUEST EXECUTION OUTSIDE BOUNDARY

IO_ENVIRONMENT_RUNTIME_BOUNDARY=
  PHYSICAL HOST SERVICE BOUNDARY;
  PROTOS LIFECYCLE/CONTINUATION/GUEST REENTRY PE-VISIBLE

INTEGER_BIGINT_BOUNDARY=
  OPERATION-SPECIFIC GUEST PATH;
  REMOVE GENERIC BIFUNCTION FROM PROVEN HOT PATH;
  NARROW BIGINTEGER LEAF;
  UNBOUNDED INTEGER SEMANTICS PRESERVED

CALL_ARGUMENT_REPRESENTATION=
  PE-VISIBLE;
  OBJECT[] PREFERRED SEALED-HOT DIRECTION;
  REPLACE LIST/ARRAYLIST ONLY ON CURRENT EVIDENCE

ORDINARY_OBJECT_SLOT_REPRESENTATION=
  LOOKUP PE-VISIBLE;
  NO AUTOMATIC MAP REPLACEMENT;
  REPLACE ONLY ON CURRENT PE/REACHABILITY EVIDENCE

CONTINUATION_FUNCTIONAL_CARRIER_BOUNDARY=
  HOST WAIT/RESOURCE CALLBACK BOUNDARY;
  GENERIC SUPPLIER REMOVED FROM GUEST-HOT SEAM;
  EXPLICIT STATE + PE-VISIBLE GUEST RESUMPTION/EXECUTION

OBSERVABLE_PROTOS_SEMANTIC_CHANGE_REQUIRED=NO

I068_INTERACTION=PRESERVED_NO_REOPEN
PERF010_INTERACTION=CAUSAL_MEASUREMENT_REQUIRED

SURVIVING_CANDIDATES=B,C
ELIMINATED_CANDIDATES=A,D,E,DEFER

RECOMMENDED_CANDIDATE=C
RECOMMENDED_CANDIDATE_NAME=
  EVIDENCE_GATED_PE_VISIBLE_GUEST_KERNEL_WITH_NARROW_HOST_GATEWAYS

RECOMMENDATION_STATUS=OWNER_APPROVED

IMPLEMENTATION_SCOPE_IF_APPROVED=
  BOUNDED I069 GATEWAY SLICES WITH EVIDENCE-GATED REPRESENTATION CHANGES

IMPLEMENTATION_REPOSITORY=guillermomolina/protos
EXISTING_I069_CAN_OWN_IMPLEMENTATION=YES
NEW_IMPLEMENTATION_ISSUE_REQUIRED=NO

IMPLEMENTATION_READY=YES
~~~
