# I098 — D201 published Closure loop result semantics (0.3.346)

**Recorded:** 2026-10-13.
**Owner:** [I098/#886](https://github.com/guillermomolina/protos/issues/886).
**Governing ratification:** [D201/#882](https://github.com/guillermomolina/protos/issues/882), approved Candidate D; [durable decision](../../decisions/language/D201_CLOSURE_WHILE_RESULT_SYMMETRY.md) and [comparative/falsification evidence](../D201/D201_LOOP_RESULT_SYMMETRY_EVIDENCE_2026_10_13.md).
**PROTOS_REVISION:** [`d7e4d4baedf984c3ee316df8bf73cac3c057eae9`](https://github.com/guillermomolina/protos/commit/d7e4d4baedf984c3ee316df8bf73cac3c057eae9).
**Published implementation version:** `0.3.346-SNAPSHOT` from `pom.xml` and `CHANGELOG.md`.
**Published specification revision:** `0.1.459` from `spec/PROTOS_SPEC_CHANGELOG.md`.

## Evidence/provenance boundary

The human executor reported: “I098: implement D201 whileTrue last-body result and standard while (0.3.346) pushed, el git diff check esta limpio. Todos los tests han pasado en local.”

The recording agent independently re-read published GitHub `main`, its exact commit title, changed-path list, `pom.xml`, specification revision and selected Java runtime paths. The agent **did not** run or repeat builds, tests, Native Image, benchmarks, or project programs. The human's green local tests and clean `git diff --check` are valid **human-reported** evidence, not independently obtained CI transcripts. No exact test count or execution command is claimed.

The published product `main` commit is titled exactly:

> I098: implement D201 whileTrue last-body result and standard while (0.3.346)

Its published diff changes **23 paths**.

## Implemented normative behavior

Two distinct **ordinary standard Closure** messages, requiring a semantic Closure receiver and semantic Closure body, keep the exact existing pre-test algorithm and strict canonical Boolean condition.

| Standard message | Zero body executions | At least one normal body execution | Abnormal exit |
| --- | --- | --- | --- |
| `condition.whileTrue(body)` | canonical `null` | last body's exact normally returned value and identity | propagates without normal result |
| `condition.while(body)` | canonical `false` | canonical `false` from final predicate | propagates without normal result |

The final predicate still executes after the last body; it may modify the returned object or fail. If the body produces a Future, `whileTrue` returns that exact Future **without loop-specific await/adoption**; any adoption by an *enclosing asynchronous Future* follows the pre-existing outer-resolution rules. For `while`, the body result is not retained for its normal return.

D201 explicitly supersedes **only** D044's constant-`null` nonzero-loop result and D180's absence of a standard `while` selector. Existing validation order, zero-argument Closure activations, Error/NLR/InvalidReturn, cancellation, suspension exact-once, `ensure`, Future ownership, ordinary lookup/extraction/shadowing and D200 Boolean protocols are preserved. No standard `whileFalse`, new parser keyword, grammar, scheduler or hidden task is added.

## Verified published source and change-set

The 23 paths inspected in the GitHub published commit are grouped below:

| Domain | Committed files and observed purpose |
| --- | --- |
| Direct protocol and publication | `src/main/java/com/guillermomolina/protos/execution/ProtosStandardObjectProtocol.java`: distinct `STANDARD_WHILE_TRUE` and `STANDARD_WHILE`, a common `preTestLoop`, return-policy classifier by selected native body and canonical root guard. `ProtosCoreBootstrap.java`: required Object `while` slot. |
| Prepared loop and Bytecode DSL | `ProtosBytecodeRootNode.java`: `PreparedWhileCall`, result policy, `StructuredWhileRetainBodyResult`, `StructuredWhileResult`, and canonical guard recognizing both. `ProtosSemanticBytecodeRootNode.java`: corresponding Bytecode DSL operations. |
| Structured and literal-inline loop | `ProtosStructuredDispatchLowerer.java`, `CanonicalToBytecodeLowerer.java`: loop result local initializes to `null`; normal body-result retention is conditioned on selected standard policy, then `StructuredWhileResult` chooses the normal return; existing continuation/admission/fallback paths retained. |
| Native conformance | Added `protos/tests/conformance/control/while-loop-results.protos` and `protos/tests/conformance/manifest.tsv` registration; changed `while-basic-and-validation.protos`, `while-future-ownership.protos`. |
| Java conformance and PE architecture | `src/test/java/com/guillermomolina/protos/execution/ProtosCoreNativeBoundaryArchitectureTest.java`, `ProtosPerf006B4FStructuredWhileTest.java`, `ProtosPerf026C1WhileInlineCallbackTest.java`, `tools/java_generated_bytecode_bci_pe_baseline.json`. |
| Normative owners | `spec/semantics/EXECUTION_AND_CONTROL.md`, `spec/semantics/CALLABLES.md`, `spec/PROTOS_GRAMMAR.md`, `spec/concurrency/FUTURES_AND_TASKS.md`, `spec/PROTOS_SPEC_CHANGELOG.md`. |
| Guide and publication metadata | `docs/guide/04-control-flow-through-protocols.md`, `docs/guide/README.md`, `pom.xml`, `CHANGELOG.md`. |

A pre-test static review identified **and fixed** avoidable lifetime extension of the last body object by a dedicated result local in `while`. In the published version, `PreparedWhileCall.retainedBodyResult` keeps the value exactly for `whileTrue` but yields canonical `null` for `while` before the next predicate. Both Bytecode DSL roots expose `StructuredWhileRetainBodyResult`, and the structured and inline lowerers call it after normally completed body activations. This source-level correction does not establish numeric performance gains or a blanket absence of retained scratch references.

The same pre-test inspection reported approximately 50 library/tool loop occurrences traced through their surrounding callback and caller results with no observed dependence on the former constant-`null` return. This was a static heuristic and not a proof. The product commit did not change library/tool source paths; the human-reported green tests provide executable regression evidence, without proving untested third-party usage.

A pre-existing physical-body fallback scratch-local retention possibility in the inline condition path remains outside the narrowly approved D201 correctness implementation. No separate measured memory or CPU claim is made.

## Acceptance gates and publication statement

| Gate | Outcome and provenance |
| --- | --- |
| Owner selection D201 Candidate D | PASS — exact selection and exceptions previously ratified in durable D201 record |
| Product commit/push | VERIFIED — `d7e4d4baedf984c3ee316df8bf73cac3c057eae9` in published `main` |
| Human local tests | PASS — reported explicitly by owner, no transcript/count provided |
| Human product `git diff --check` | PASS — reported explicitly by owner |
| Published `pom.xml` and `CHANGELOG.md` | VERIFIED — `0.3.346-SNAPSHOT` |
| Published normative changelog | VERIFIED — `0.1.459` |
| Separate Native Image, `make check`, perf measurement | NOT REPORTED, NOT IMPLIED |
| Tests by documentation agent | NOT RUN |

**Closure scope:** I098 implements the full D201 approved paired-selector behavior with spec, root Object publication, direct/structured/inline paths, conformance, and documentation in one product commit. The existing D201 decision remains closed and authoritative. Independent I097/D200 Boolean `if`/`else` work is not reopened. No additional implementation slice is known to be required for I098 based on the published evidence and owner report; a future retention/performance comparison would be a different task.

```text
I098_IMPLEMENTATION=PUBLISHED
OWNER_DECISION=D201_CANDIDATE_D
PROTOS_REVISION=d7e4d4baedf984c3ee316df8bf73cac3c057eae9
PRODUCT_VERSION=0.3.346-SNAPSHOT
SPEC_REVISION=0.1.459
HUMAN_LOCAL_TESTS=PASS
HUMAN_GIT_DIFF_CHECK=PASS
AGENT_TESTS_EXECUTED=NO
CODE_PATHS=DIRECT_STRUCTURED_INLINE
RETENTION_POLICY=SOURCE_VERIFIED
MEASURED_PERFORMANCE=NOT_REPORTED
CLOSURE_READY=YES
```
