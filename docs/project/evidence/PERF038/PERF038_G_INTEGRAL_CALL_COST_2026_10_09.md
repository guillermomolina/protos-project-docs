# PERF038-G — Integral fixed-call-cost investigation and single-slice handoff

Date: 2026-10-09
Issue: https://github.com/guillermomolina/protos/issues/852
Product HEAD inspected: `89e1b038c2fd510f47ddb991496882548d1bbe44`
Status: investigation checkpoint; no implementation authorized by this record; one integrated implementation slice proposed.

## Revision-bound evidence

Published PERF038-F graph (product `89e1b038`, GraalVM `25.4.4.1.1`, After TruffleTier/Tier 2, correctness PASS, STABLE [16000,64000]): 132 final nodes, 4 allocations, 3 control splits, 15 guards/deopts, 9 loads, 0 invokes. Expansion attribution: PrepareSendArguments 41, CachedBytecodeNode 23, LoadFrameClosureArgument 6. Reference GraalJS historical 34 nodes; net difference 98 is **not** an estimate of removable nodes. The F capture is Protos-only, harness_dirty=true with source hashes. Older post-E 152, post-D 196, combined C/PERF037 261, post-B 956, original 1789 are different revision-bound states. No stable timing gain is inferred.

## Source inspection

Protos: `src/main/java/com/guillermomolina/protos/execution/{ProtosFrameArguments,ProtosBytecodeRootNode,ProtosSemanticBytecodeRootNode,CanonicalToBytecodeLowerer,ProtosStructuredDispatchLowerer}.java`. Contracts: `AGENTS.md`, `AGENTS.work/{PERFORMANCE,COORDINATION}.md`, `spec/semantics/{CALLABLES,EXECUTION_AND_CONTROL}.md`, issue #852 comments.

GraalJS public source: `graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/nodes/function/{JSFunctionCallNode,FunctionRootNode}.java`, `graal-js/src/com.oracle.truffle.js/src/com/oracle/truffle/js/runtime/JSArguments.java`. Branch source not yet tied to the exact GraalVM binary revision. GraalJS structurally selects Call0/Call1/CallN and Invoke0/Invoke1/InvokeN, and ordinary JSArguments carries two internal slots (this,function) before user args; constructor/resume are separate formats. No claim that GraalJS eliminates all arrays.

## Mechanism-level findings

- Protos immediate method call still constructs the internal `[closure,receiver,methodHome,caller,returnHome,...user]` header, unlike minimal direct zero-argument closure headers. Whether header/array creates final residual nodes requires graph edge tracing.
- `PrepareSendArguments` holds target selection, receiver/methodHome, selection assumption, provenance, return-home policies and admission. PERF038-D/E/F already split compact/inherited/materializing paths; do not repeat them.
- `OrdinarySourceCall` separately transports target, frame args and completion/lifecycle protocol. Carrier allocation survival is not demonstrated.
- `CanonicalToBytecodeLowerer.emitOrdinaryPreparedInvocation` emits EnterClosureCall, `emitContinuationComposition`, FinishClosureCall; composition emits a while(IsContinuation) Yield/Resume loop. Whether this survives the F graph must be established with edges; presence in source alone is insufficient.
- `ProtosSemanticBytecodeRootNode.LoadFrameClosureArgument` specializes compact positional access versus materialized activation, already improved by previous slices.
- Hidden cross-cutting questions: lexical capture and escape, context observation and mutable local slots, prelude and entered-context authority, Task/Process, ReturnHome, receiver/methodHome, nonlocal transfer, exception and debugger instrumentation, conversions, frame states and bytecode-generated temporaries.

## Attribution matrix

| Family | Protos origin | GraalJS analogue | Required by unchanged M1? | F-node attribution | Candidate architecture | Semantic risk |
|---|---|---|---|---|---|---|
| Call preparation | PrepareSendArguments | Invoke0/1/N | Some yes | expansion 41, nonexclusive | plan-aware minimal topology | high |
| Internal args | ProtosFrameArguments | JSArguments | receiver/methodHome as required | unproven | arity and capability-aware header | high |
| Parameter reads | LoadFrameClosureArgument | getUserArgument | only if used | expansion 6, nonexclusive | positional-only access without collection projection | medium |
| Argument collection | published activation/context | observable JS arguments | only on actual observation | unproven | defer projection | high |
| Carrier | OrdinarySourceCall | call/cache node | transport yes | unproven | scalarize/avoid redundant state | high |
| PIC and method binding | GuardedSendTarget | JS call cache | yes | unproven | cache immutable selection + guard correctness | critical |
| ReturnHome/nonlocal | invocationHome, FinishClosureCall | ordinary/special control | depends on callee | unproven | capability-indexed completion | critical |
| Continuations | emitContinuationComposition | async/generator pathways | only if suspensible | unproven | proven non-suspending lowering only | critical |
| Lexical context/captures | frame owner, activation | enclosing frame | depends | unproven | escape-aware minimal state | critical |
| Provenance/prelude/domain | inherited caller guards | realm/context | isolation is required | unproven | immutable guarded invariant | critical |
| Task/Process | dynamic authority | no direct equivalent | depends | unproven | non-task path by construction | critical |
| Exceptions/FrameState | control transfer, bytecode | JS exception state | failure semantics required | unproven | bounded cold failure path | high |
| Generated bytecode | CachedBytecodeNode | JS body | execution required | expansion 23, nonexclusive | remove redundant emit temporaries where proven | high |
| Interop/instrumentation | foreign entry, tag tree | foreign/tag nodes | not ordinarily | unproven | dormant feature routes with deopt correctness | critical |
| Value conversion | primitive representation | JS internal types | depends | unproven | remove only proven redundant conversions | high |

No node count per family may be inferred from Truffle expansion attribution. Shared dependent guards/Pi/frame states must not be counted twice.

## One integrated implementation slice only: PERF038-H

Scope (single coherent change, one review, one validation campaign): introduce minimal ordinary-call plan based on independently established properties (arity/collection use, suspension, capture/observation, return-home and method binding), use it to omit *provably unnecessary* preparation/transport/continuation/finish and bytecode temporaries for C0/C1/C2/M1 while retaining safe fully general fallback and sound invalidation. Do not implement work whose causality cannot be established by existing captured BGV evidence or a human-provided edge mapping. No benchmark-only shortcuts.

All source changes in `guillermomolina/protos`, primarily the five classes above and execution plan classes where needed; add focal semantics/conformance tests. Keep unchanged `primitive-method-call` M1 and introduce separate C0/C1/C2 controls without silently changing benchmark harness. Verify fresh bound-Closure identities, inheritance/super/receiver/methodHome, arity, lexical context escape and structural mutation, return homes, nonlocal transfer, suspension and cancellation, Task/Process, interop, errors, debugger/instrumentation, PIC invalidation, mono/polymorphic fallbacks.

## Evidence/acceptance gate

Before claiming architectural removal, inspect comparable F BGV edges for VirtualObject/VirtualArray, If, Pi, Narrow, loads/stores, FrameState, guards, exceptions and bytecode operations; map each live component to actual source. If unavailable, report negative/unknown rather than declare comprehensive causal proof. Human executor alone runs tests and captures new Protos/JS graphs on identical GraalVM/toolchain, policy, surface, workload and tier, correctness PASS and STABLE, preserves exact product/harness SHA and raw graphs. Do not claim timing improvement from node reduction. `pom.xml` and `CHANGELOG.md` update **after green tests and immediately before human commit/push**, and no tests after touching those two files.

Current checkpoint is a bounded source-grounded research result, **not proof that all 98 net nodes are attributable or removable**. Issue #852 stays open. No product changes/tests/builds performed in PERF038-G.
