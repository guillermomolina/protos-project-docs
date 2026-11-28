# BUG044 — Partial interpreter call stack overhead reduction

Date: 2026-11-28. Non-normative, revision-bound investigation and publication evidence.

- Owning issue: [BUG044 #1225](https://github.com/guillermomolina/protos/issues/1225), **OPEN — partial improvement, not a correctness fix**.
- Related evaluation: [PERF049 #1230](https://github.com/guillermomolina/protos/issues/1230), [UPSTREAM007 #1229](https://github.com/guillermomolina/protos/issues/1229).
- Investigation baseline: [348a2bdd55b7e4c82b4a7572914030f0c8104d4b](https://github.com/guillermomolina/protos/commit/348a2bdd55b7e4c82b4a7572914030f0c8104d4b).
- Published product revision: [0d108637668d7e8e4284970c996f98ec412dabe8](https://github.com/guillermomolina/protos/commit/0d108637668d7e8e4284970c996f98ec412dabe8), `BUG044: Reduce interpreter call stack overhead via typed direct calls`, implementation version 0.4.258-SNAPSHOT (verified from published POM/changelog delta).
- Runtime: GraalVM CE 25.6.5.1+2.1, JDK 25.0.5.1, Truffle 25.6.5.1; requested 16 MiB guest-carrier stack. No budget or workload changes.

## Published delta

The exact product diff contains the following retained changes (experiment identifiers are evidence labels, not production identifiers):

- B1: typed `DirectCallNode.call(ProtosBytecodeRootNodeGen.class, -1, arguments)` for the admitted unobservable-home guest-frame route; existing untyped call retained otherwise.
- F1/F1-A: existing Task preparation encapsulated in `PrepareInvocationTaskNode` with `@GenerateInline`, `@GenerateUncached`, `@GenerateCached(false)` and one specialization; five existing profiles moved into the preparation node; direct test entry points adapted.
- F10: 17 redundant Assumption specialization parameters removed, preserving the original assumption expressions and invalidation.
- F11: six redundant root bindings moved into existing cache initializers.
- F13/F13-E: replace the constant-returning TextReader initial-state operation with `emitLoadConstant(Advance.needInput())`; reconcile the exact generated BCI topology baseline (no weakening of guard).
- F18: two redundant binary identity-PIC target caches removed; use the existing DirectCallNode target to derive the cached callee root; retain nullable admission, selection and stability checks. No independent physical-frame measurement for F18.
- The five fixed-arity call operation classes, shared dispatcher, root, TextReader execution, deferred-caller tests and generated BCI baseline are the affected paths. POM and changelog are publication metadata. No normative specification delta; no launcher/default option change. Published Java diff does not modify existing APL notices; no new Java source file.

Retained semantics: ordinary message selection and redefinition, specialized internal hot paths, caller and Task provenance, fresh activation, exact ReturnHome ownership, suspension and error propagation. No new trampoline, alternative activation, or obsolete call machinery.

## Upstream references

Pinned development GraalPy revision [a0c7f5ca28ae9773781a5d4291e91d33591d3a9b](https://github.com/oracle/graalpython/tree/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b):

- [CallDispatchers.SimpleDirectInvokeNode.doDirect](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java#L108-L144): inline DSL node, caller preparation, typed guest-frame call and ordinary frameless call. B1 adopts the typed call form, not a replacement dispatcher.
- [ExecutionContext.CallContext](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/runtime/ExecutionContext.java#L166-L197): inline uncached-capable preparation node; Protos retains its own existing Task behavior.
- [CallBinaryMethodNode.callPFunction](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/special/CallBinaryMethodNode.java#L137-L149): identity PIC, direct assumption expression, cached direct call node; [CallDispatchers](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/call/CallDispatchers.java#L499-L529) also derives information from the cached call node target.
- [ImportFromNode.doImport](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/nodes/bytecode/ImportFromNode.java): existing-node initializer pattern using `$node` (syntactic precedent, not a claim of identical target-selection semantics).
- [RootNodeCompiler.createConstant](https://github.com/oracle/graalpython/blob/a0c7f5ca28ae9773781a5d4291e91d33591d3a9b/graalpython/com.oracle.graal.python/src/com/oracle/graal/python/compiler/bytecode_dsl/RootNodeCompiler.java#L2838-L2844): stable singleton values emitted with LoadConstant.
- Truffle [ac4eb8eadd38720cce32c9cdd7e337939515b5d0](https://github.com/oracle/graal/commit/ac4eb8eadd38720cce32c9cdd7e337939515b5d0): root-aware calls. Released API and development source are distinct; no adoption of unverified new proxy bindings.

## Physical-stack diagnostics

Maintainer-executed x86-64 HotSpot C1 diagnostics, guest compilation disabled, using compiled method prologues, OopMaps and inlining maps, not logical Java trace-frame counts. These are intermediate local candidates, not independently measured on the final published SHA.

| Candidate | Bytes per recurring level |
| --- | ---: |
| B1 | 3120 |
| B1 + F1 | 2976 |
| F10 | 2928 |
| F11 | 2896 |
| F13/F14A (before F18) | 2624 |
| Comparable GraalPy probe | 2800 |

F13 reduces 496 B/level (15.9%) relative to B1 and is 176 B/level (6.3%) below the measured GraalPy probe. F13 removes an unrelated TextReader handler that expanded the generated interpreter partition frame from 96 to 368 bytes; the resulting partition frame is 96 bytes. The arithmetic 16 MiB / 10,000 is about 1678 B/level before other overhead, not a measured actual stack limit. C1-only 10,000-call capacity was not established for either language and is not an added acceptance requirement. No final F18-specific frame size, steady-state speedup, guest graph reduction or Native-specific stack measurement is claimed.

The comparable GraalPy probe used matching Truffle runtime/API/compiler artifacts, a platform thread requesting 16 MiB, guest compilation disabled, a recursion limit of 20,000, and analogous recursive callback/identity work. Asynchronous host and C1 runs could fail in both languages; synchronous host compilation passed in sampled runs. This supports a host compilation timing/stack-cost hypothesis, not proof of an upstream compiler defect.

Historical archive experiments found five normal and five guest-interpreter successes both immediately before and at the GraalVM 25.5 adoption (250b89bf2faf3e580980a67c2d69aa94d10e4df3 and a7aa54bc0eceabcc2258ebf0c5b52c1b6d6ea71b). The later pre-refactor revision failed. Historical bisection was inconclusive because intermediate commits did not build or timed out; no first-bad commit is claimed.

## Negative experiments and residual defect

| Intermediate candidate | Normal successes | Guest-interpreter successes |
| --- | ---: | ---: |
| F11 | 5/5 | 3/5 |
| F13 | 4/10 | 5/10 |
| F14B | 7/20 | 8/20 |
| F16 | 9/20 | 6/20 |
| F18 | 8/20 | 7/20 |
| F19 revised | 2/5 | 2/5 |

F9 (three-specialization dispatcher) increased the reconstructed C1 frame total to 3168 B and was reverted. F14B cutoff experiment, F16 uncached-threshold experiment and F19 revised shared single-specialization dispatcher were also reverted. F19 passed 272 focal tests but failed the capacity gate. F18 is retained as a structural simplification, not a demonstrated standalone capacity fix. No alternate CALL2 bypass was adopted.

## Validation and temporary workaround

Maintainer reported final consolidated local validation: all tests pass, ordinary configuration; git diff check clean, product pushed. The requested final gates were make check and make test; individual final gate logs and CI results were not supplied in the completion message. Earlier focal suites passed (F18 156/156; F13 209/209; generated BCI static/dynamic checks passed after exact reconciliation). F19 tests are rejected-candidate evidence, not final candidate validation.

On the consolidated pre-publication build, five independent unchanged standalone benchmark processes with `JAVA_TOOL_OPTIONS="-Xbatch"` exited zero (5/5); the original Test Tool regression passed 1/1 with the same option. Commands:

```sh
JAVA_TOOL_OPTIONS="-Xbatch" bin/protos protos/benchmarks/micro/closure-call.protos
JAVA_TOOL_OPTIONS="-Xbatch" bin/protos test --jobs 1 --file protos/tests/conformance/regression/deep-recursive-closure-call-stack-capacity.protos
```

This is an opt-in HotSpot/JVM workaround using synchronous host compilation (not guest BackgroundCompilation). It can increase startup/warmup time and does not apply to a Native executable. No permanent launcher/CI option, increased stack, weakened test, retry masking, or closure claim. Finite passing samples are not a reliability guarantee.

## Remaining acceptance

BUG044 remains open: stable success of the unchanged 10,000-call benchmark and regression in normal supported JVM execution and reproduced guest-interpreter diagnostic configuration without workaround, with appropriate validation and Native/hosted assessment. Preserve PLAT046 (#778) and the historical BUG037 (#1131) closure. PERF049 may reuse the physical diagnostics and published source-call typed-call adoption, but its broader causal candidate comparisons, guest-tier and Native performance evidence remain outstanding. UPSTREAM007's remaining adoption evaluation is independent.

Evidence provenance: maintainer-supplied local experiment logs and completion report in the 2026-11-28 investigation; published product diff independently reconciled through GitHub. Raw local diagnostics were not uploaded into this repository; no raw artifact or CI run is fabricated. This snapshot records results and limitations, not a normative decision or final closure.
