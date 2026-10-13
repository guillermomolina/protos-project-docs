# I095 — Published canonical Boolean inline checkpoint and validation

**Evidence date:** 2026-10-13
**Product owner:** [I095 / #878](https://github.com/guillermomolina/protos/issues/878)
**Performance investigation:** [PERF041 / #863](https://github.com/guillermomolina/protos/issues/863)
**New broader architectural investigation:** [PLAT057 / #881](https://github.com/guillermomolina/protos/issues/881)
**PROTOS_REVISION:** `57a43b6d94697f9a6256e36c7a775dfa4a1218d7`
**Published version:** `0.3.343-SNAPSHOT`
**Previous published main:** `55be9274bf90677a2ac243c77450476150b1c8c6`
**Commit:** [I095 squash integration into main](https://github.com/guillermomolina/protos/commit/57a43b6d94697f9a6256e36c7a775dfa4a1218d7)
**Evidence level:** human-reported local command results, independently verified published commit/POM/changelog through GitHub; no independent compilation, native build, benchmark or BGV execution by the recording agent.

## Publication identity and content

The maintainer merged the experimental `experiment/perf041-iftrue-abcd-e1-20261012` checkpoint into `main` using `git merge --squash` and committed/pushed the single publication `57a43b6d`. The source integration covers the preceding experimental branch history plus its locally validated final B6a/B6b and debugger fixes, not merely the final 12-file local staging diff.

The published commit changes **16 files** (the maintainer's final Git output: **2,544 insertions / 196 deletions**):

- **Compiler/runtime:** `CanonicalBindingAnalysis.java`, `CanonicalToBytecodeLowerer.java`, `ProtosBytecodeRootNode.java`, `ProtosSemanticBytecodeRootNode.java`, new `ProtosCanonicalBooleanOneNode.java` and `ProtosNativeIfTrueCallbackNode.java`.
- **Debugger/scope:** `ProtosBytecodeTagTreeNodeExports.java`, `ProtosDebuggerScope.java`.
- **Java regressions:** `ProtosI092CompactBooleanTest.java`, `ProtosPerf032ExecutionModeTest.java`, `ProtosI026EScopeTest.java`, `ProtosLm009DPublicDebugCliTest.java`.
- **Static PE-safety inventories:** `tools/java_bytecode_api_pe_baseline.json`, `tools/java_generated_bytecode_bci_pe_baseline.json`.
- **Publication metadata:** `pom.xml` and `CHANGELOG.md`, both updated to `0.3.343-SNAPSHOT` in the same product commit.

The resulting implementation introduces compiler-side guarded/canonical Boolean control lowering, literal-callback body emission, NORMAL-mode inlining of admitted callback-owned bindings and direct-parent captures, and DEBUG tooling scope projection. Ineligible cases preserve the B′/ordinary-message path. This is a **bounded implemented checkpoint**, not a claim that the general aggressive Closure inliner is already designed or that a speedup has been causally demonstrated.

## Validation sequence and exact limitations

Human executor reported the following gates for the staged implementation **before** final publication metadata edits:

1. The original `make test` found one scope-projection regression: `ProtosPerf026B1BooleanInlineCallbackTest.debuggerScopeInsideInlineCallbackProjectsTheCallbackActivation` could not see `innerMarker` in the debugger's callback scope. An instrumentation-only virtual local-binding projection was added. The focused rerun reported `PERF026_B1_IFTRUE_INLINE_SCOPE_PROJECTION=YES`.
2. A subsequent full `make test` reported **3,466 tests, four failures, zero errors, one skipped**: two obsolete B′ topology assertions in `ProtosI092CompactBooleanTest`, a `ProtosI026EScopeTest` single-field implementation-shape assertion, and a `ProtosLm009DPublicDebugCliTest` CLI exit-code failure. The human applied the bounded regression/diagnostic correction. **No failed suite is represented as PASS.**
3. The human then explicitly reported final **`make test -> PASS`**.
4. The initial `make check` rejected two newly introduced Bytecode API sites in `normalInlineCallbackBindings` as `NEW_UNBASELINED_SITE`, though both had `BOUNDARY_CUT`, zero PE-reachable risks, and zero analysis ambiguities. TEST009-C's checked-in baseline was updated to account for exactly those two tooling-only sites.
5. A later `make check` rejected changes to the generated semantic interpreter's dispatch BCI topology (`NEW_TOPOLOGY`/`STALE_BASELINE`). The human updated TEST009-D's reviewed generated-dispatch topology baseline, then reported final **`make check -> PASS`**. The generated-dispatch guard's dynamic compilation check is part of `make check`.
6. The human executed **`make -C build/native test`** and reported **PASS**. This target builds the native executable and runs the native regression gate (CLI, guest smoke, Test Tool, numeric conformance, DAP stacktrace and interpreter-only behavior under PLAT045). No guest JIT support in Native Image is inferred.

After these three reported PASS gates, `pom.xml` and `CHANGELOG.md` were updated. No further functional test runs were reported or requested after those versioning-only edits. Final `git diff --cached --check` was clean and the squash was pushed to `main`. Published `pom.xml` and `CHANGELOG.md` were independently read at the public revision.

**Evidence boundary:** the recording agent did not independently run the tests or the native executable and has not inspected complete final transcripts. This checkpoint has **no accepted matched before/after BGV/code-size allocation/timing proof** for the newly published architecture. Historic node counts in PERF041 must not be relabelled as measurements of `57a43b6d`.

## Design authority and follow-up

The project owner directed that **I095 remain open but paused** after publication, rather than be closed or expanded into a collection of `ifTrue`-specific micro-optimizations. The central discovered question is *general* Closure/method inlining: local block bindings and captured scopes should not by themselves bar a semantics-preserving physical inline expansion. The conceptual precedents from Self and Smalltalk (including Dolphin's optimized lexical scope relocation) motivate the research, not a selected Protos implementation.

[PLAT057/#881](https://github.com/guillermomolina/protos/issues/881) now owns that unresolved runtime/compiler architecture investigation. It must distinguish source-to-bytecode lowering from Truffle/Graal CallTarget inlining, virtual lexical scope/activation rematerialization and all semantic/tooling/deopt obligations. Comparative research, falsification, 12-axis scoring and **explicit project-owner approval** are required before any durable new architecture is ratified.

PERF041 remains the specialized current/revised `primitive-if-true` graph attribution owner; [PERF042/#871](https://github.com/guillermomolina/protos/issues/871) separately owns direct ordinary Closure-call cost. These performance investigations supply evidence to PLAT057; their historical benchmark captures do not prove a general architecture or its speedup.

**No normative specification change and no architectural ratification are recorded by this evidence document.** GitHub issues remain the live status authority.
