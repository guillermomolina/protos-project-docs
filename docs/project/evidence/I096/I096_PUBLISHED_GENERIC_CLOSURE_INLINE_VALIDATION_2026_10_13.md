# I096 — Published generic ordinary Closure inlining: validation checkpoint

> **Coordination correction, 2026-10-13:** This record's initial publication mentioned I099 as the separate future Boolean-control implementation. The maintainer corrected that issue boundary: [I099/#887](https://github.com/guillermomolina/protos/issues/887) was closed as duplicate; all proposed four-selector work stays within [I095/#878](https://github.com/guillermomolina/protos/issues/878). Published source/test facts for I096 are unchanged.

**Evidence date:** 2026-10-13  
**Product implementation owner:** [I096 / #884](https://github.com/guillermomolina/protos/issues/884)  
**Architecture authority:** [PLAT057 / #881](https://github.com/guillermomolina/protos/issues/881), ratified conservative Candidate B  
**Product repository:** `guillermomolina/protos`  
**PROTOS_REVISION:** `ff57d22b6fa1abdc252200c3c114d015bb80a315`  
**Product commit:** [`ff57d22b`](https://github.com/guillermomolina/protos/commit/ff57d22b6fa1abdc252200c3c114d015bb80a315)  
**Published version:** `0.3.347-SNAPSHOT`, verified in `pom.xml` and `CHANGELOG.md` at the published revision  
**Scope:** I096-A and I096-B1, *not* general C2/N, dynamically polymorphic calls or all possible Closure/Context shapes  
**Validation evidence type:** **project-owner / human-executor report**, not independently executed tests

## Published implementation verified from exact product revision

The published product commit contains changes to 12 tracked paths:

- `CanonicalToBytecodeLowerer.java` — guarded ordinary Closure-literal and same-root binding-call candidate admission, normal-mode inline body, independent generic fallback.
- `ProtosBytecodeRootNode.java` — `prepareInlineClosureLiteralCall`, using ordinary `call` lookup, canonical standard selection and the existing `PreparedInlineLiteralCall` carrier.
- `ProtosSemanticBytecodeRootNode.java` — Bytecode DSL operations for admission/selection and existing callback semantics.
- `ProtosI096InlineClosureLiteralCallTest.java` — focused tests including C0/C1, own lexical bindings, captured writes, D179 presence/remove/recreate, selected assignment targets, fresh observed Context, reentrancy, Error/non-local return, suspension, `call` overrides, debug topology and binding-call invalidations.
- Four PERF026 callback regression tests, `ProtosPerf042DefinitionKeyedDirectCallTest.java`, `tools/java_generated_bytecode_bci_pe_baseline.json` — directly affected existing acceptance/PE inventories.
- `pom.xml` and `CHANGELOG.md` — already-published `0.3.347-SNAPSHOT` metadata.

The exact changelog states: a direct C0/C1 call whose receiver is an immediate Closure literal (I096-A), or a binding holding an eligible definition in the same root (I096-B1), can run the eligible body in the containing physical root in NORMAL, after proving the ordinary selected `Object.call` behavior and its target. Reassigned, removed, shadowed, substituted, aliased or overridden behavior continues through guarded physical fallback. DEBUG retains the ordinary physical callee topology. Module-level candidates, defaults/rest, spread, C2+, bodies constructing nested Closures and non-direct calls are not covered by this checkpoint. These are **source/publication facts**, not evidence that generated machine code has no surviving overhead.

## Human-executor validation reported after the earlier pending-test checkpoint

In the current interaction, after the implementation had been published but previously reported as not yet tested, the maintainer explicitly stated:

- `git diff --check` is clean.
- **All tests have passed locally.**

The human did not supply the names of individual test commands, their log excerpts, test counts, native-image gate results or timestamps beyond the interaction date. Accordingly, this record reports **human-reported all-local-tests PASS** and **human-reported diff-check clean** only. It does not manufacture a `make check`, `make -C build/native test`, CI, benchmark, or independent agent-run result. No product code change or new product commit was performed by the recording agent. Published metadata and exact commit identity were read from GitHub.

## Semantics protected by the implementation contract

PLAT057 Candidate B is an implementation architecture, not permission to change the Protos language:

- Ordinary `Object.call` selector selection is overridable and guarded; body inlining is never a bypass of a selected different behavior.
- A fresh semantic call has independent lexical binding/Context identity. D179 PRESENT/ABSENT, removals/recreation, captured by-reference writes, and pre-RHS assignment target selection remain exact.
- Non-local return, `ensure`, error precedence, suspension/cancellation/continuation, receiver/`this`/`methodHome` and ordinary argument rules remain semantic obligations; unsupported physical forms fall back.
- NORMAL/DEBUG RootTag, SourceSection, tooling/scope and frame-topology restrictions of PLAT044 and PLAT057 are not globally relaxed.
- The simple synchronous path must not acquire unconditional RootTask/RootActor/RootProcess, eager Context/arguments Array or a global rematerialization registry.

The existence of tests covering these cases and the maintainer's reported suite success are evidence of reported validation; neither is represented as an independent proof of every possible program.

## Residual acceptance / independent follow-up

**I096/#884 remains OPEN.** Even with a published commit and human-reported green local tests, its independent final-tier graph/code-size/compiled-call acceptance has not been supplied. [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) owns ordinary Closure-call measurement; [PERF041/#863](https://github.com/guillermomolina/protos/issues/863) owns Boolean-control graphs, and historical pre-I095 5,193/892/39-node values must not be attributed to this commit.

[I095/#878](https://github.com/guillermomolina/protos/issues/878) remains OPEN and, following the owner's coordination correction, **also owns the next coherent four-selector Boolean-control inlining slice** (`ifTrue`, `ifFalse`, `if`, `else`). [I099/#887](https://github.com/guillermomolina/protos/issues/887) was briefly allocated separately in error, then **CLOSED AS DUPLICATE** without any product source changes, tests, publication or independent implementation. The identifier is not reused. This later correction does not modify the published I096-A/B1 validation evidence above.

**Evidence boundaries:** a source-code/commit inspection, owner-reported local tests/diff-check, and the cited ratified decision. **Not performed by the recording agent:** product tests, compilation, execution, Native Image, graph BGV capture, benchmark timing, CodeSize analysis. **No normative spec changed in this documentation checkpoint.**

```text
FORMAL_IDENTIFIER=I096
PROTOS_REVISION=ff57d22b6fa1abdc252200c3c114d015bb80a315
PRODUCT_VERSION=0.3.347-SNAPSHOT
HUMAN_REPORTED_ALL_LOCAL_TESTS=PASS
HUMAN_REPORTED_GIT_DIFF_CHECK=CLEAN
AGENT_EXECUTED_TESTS=NO
POST_I096_MATCHED_FINAL_TIER_GRAPHS=PENDING
I096_FUNCTIONAL_PUBLICATION=YES
I096_FULL_PERFORMANCE_ACCEPTANCE=PENDING
NEW_PRODUCT_CHANGE_IN_THIS_RECORD=NONE
I099_IMPLEMENTATION=CLOSED_DUPLICATE_NO_PRODUCT_WORK
I095_FOUR_SELECTOR_EXTENSION=NEXT_IMPLEMENTATION_SLICE_NOT_YET_EXECUTED
```
