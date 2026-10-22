# DOC011 — Main-HEAD learning-material reconciliation publication

Date: 2026-10-22  
Live issue: https://github.com/guillermomolina/protos/issues/952  
Release parent: https://github.com/guillermomolina/protos/issues/951

## Exact product publication

- Product repository: `guillermomolina/protos`
- Base revision: `7af62b8394b04790b678029501031a12d9d13c4d`
- **Published product revision:** `1111ccaf83ced008a273918af94cd11627d44f6d`
- Commit: https://github.com/guillermomolina/protos/commit/1111ccaf83ced008a273918af94cd11627d44f6d
- The publication is exactly one main-branch commit containing ten files (449 insertions, 52 deletions); the maintainer reported a clean post-push worktree. The remote `main` ref and ten-path diff were independently verified through GitHub.
- The native GitHub parent relation `DOC011 #952 -> DIST016 #951` was independently verified through the GitHub issue parent endpoint.

## Published slices

1. **Control flow and runnable lessons.** Updated `docs/guide/04-control-flow-through-protocols.md` and `docs/guide/README.md`; added `protos/tutorials/04-control-flow/02-pre-test-loops.protos` and `03-post-test-loops.protos`. Covers contextual if/else, the specified contextual while and suffix until surfaces, and explicit pre-/post-test loop protocols without inventing new syntax.
2. **D203 Closure arguments.** Updated `docs/guide/03-closures-methods-and-receivers.md` for missing versus explicitly null arguments, defaults, extra arguments, frozen rest Array and exact-arity API boundaries; links existing accurate lessons.
3. **Numeric model.** Updated `docs/guide/05-values-identity-equality-and-collections.md`, library navigation and the examples index; added `protos/examples/numbers/exact-and-complex.protos` covering Integer, Float, Fraction, Complex and `std:math/Math` as currently implemented.
4. **Runtime, inspection and navigation.** Corrected the `std:runtime.info()` index description to its four fields; added both existing runtime-introspection programs to the cookbook; fixed the `--inspect` invocation comment. Preserved the existing accurate standalone runtime and inspect guides.

Materially published files: 
`docs/guide/{03-closures-methods-and-receivers,04-control-flow-through-protocols,05-values-identity-equality-and-collections,README}.md`,
`docs/guide/library/README.md`,
`protos/examples/{README.md,numbers/exact-and-complex.protos,runtime-introspection/inspect.protos}`, and
`protos/tutorials/04-control-flow/{02-pre-test-loops,03-post-test-loops}.protos`.

No normative specification, compiler/runtime, POM, changelog, release tag, GitHub Release, website lock or binary artifact was changed by this commit. DIST016 still owns final candidate freeze, source/release locks, release-specific validation and explicit approval to publish.

## Human-executed validation evidence

- Batch 1: both new control-flow tutorials printed the expected four lines and returned `0`; `ProtosCliLearningMaterialsTest` and `ProtosD212StandardLoopPolaritiesTest` returned `0`.
- Batch 2: current D203 tutorial and the rest-arguments example returned `0`; `ProtosPerf006B2C3ARestBindingTest`, `ProtosPerf006B2C3B2SimpleDefaultBindingTest`, `ProtosPerf006B2C3B3CallSendDefaultBindingTest` returned `0`.
- Batch 3: new numeric example printed eight `true` lines and returned `0`; `ProtosFractionSemanticsTest`, `ProtosComplexArithmeticTest`, `ProtosMathModuleTest` and `ProtosMathIntegerModuleTest` returned `0`.
- Batch 4: runtime-introspection `info.protos` and `--inspect .../inspect.protos` returned `0`; six runtime/inspect/CLI/learning-material JUnit classes returned `0`.
- Maintainer reported `git diff --cached --check` clean and the final `make test` completed with **`DOC011_FULL_SUITE_RC=0`**. This is human-reported evidence, not a separately rerun suite.
- The separately suggested Markdown local-link checker and `scripts/source_style_guard.py` have **no reported outcomes in this conversation**. Do not represent those specific static gates as PASS until their output is provided or separately evidenced.

## Closure / DIST016 handoff

The bounded DOC011 source publication and its integrated test gate are complete. Document the separately unreported static link/style checks before marking every issue acceptance checkbox complete. This publication **is not** a DIST016 frozen release candidate and does not authorize a version, tag, website update, or public release. DIST016 must resolve its candidate SHA from the then-current main HEAD under its existing release workflow.
