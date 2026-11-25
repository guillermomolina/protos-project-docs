# DOC029 — Contextual match learning materials and first-party adoption

Date: 2026-11-25

- Owning issue: [DOC029 #1179](https://github.com/guillermomolina/protos/issues/1179).
- PROTOS_REVISION: [d2bd4bed4527ef4856841c95b49f20b068d62d55](https://github.com/guillermomolina/protos/commit/d2bd4bed4527ef4856841c95b49f20b068d62d55).
- Implementation prerequisite: [I176 #1173](https://github.com/guillermomolina/protos/issues/1173), [product revision 381bcf7cea43f36d49cb7a112bc822d5b5968450](https://github.com/guillermomolina/protos/commit/381bcf7cea43f36d49cb7a112bc822d5b5968450).
- D263-C design authority: [ratified decision](https://github.com/guillermomolina/protos-project-docs/blob/dce1659fef6e0795f1fbc9fb2a5ef38feae688db/docs/project/decisions/language/D263_CONTEXTUAL_MATCH_IMPLICIT_LAZY_ARM_CLOSURES_C_RATIFIED.md).
- Implementation version unchanged by DOC029: 0.4.223-SNAPSHOT.

## Published scope

Verified exact product commit contains six guide/navigation changes and four new standalone Protos programs:

- docs/guide/04-control-flow-through-protocols.md
- docs/guide/12-matching-expressions.md
- docs/guide/README.md
- docs/guide/SOURCE_STYLE.md
- protos/examples/README.md
- protos/tutorials/README.md
- protos/tutorials/04-control-flow/05-contextual-match.protos
- protos/examples/control-flow/status-dispatch.protos
- protos/examples/control-flow/structural-dispatch.protos
- protos/examples/control-flow/custom-case-selection.protos

The guide teaches ordinary caseOf equivalence, lazy actions, eager keys, captures, Closure-valued expressions, ordinary Errors, nesting, postfix composition, contextual recognition and custom dispatch. No Java, normative specification, existing tests, runtime or version metadata changed in the commit. No performance claim; PERF044 remains independent.

## Adoption inventory

Maintainer-supplied agent report: literal caseOf occurrences before DOC029: 12 (11 in protos/tests/conformance/matching/case-of.protos, one formatter comment in protos/tools/formatter/Structural.protos). No occurrences in protos/lib, tutorials or examples before this change. The tests retain six executable regression/protocol occurrences and five non-executable references. The formatter comment is non-executable. New deliberate teaching occurrences: one canonical-equivalence send in the new tutorial; five in custom-case-selection (two existing Map sends, one existing callable send, one comment, one custom protocol definition). Total 18; no production migration candidate identified. Inventory is agent-reported local inspection, not a fresh exhaustive remote scan.

## Validation and publication

- Maintainer transcript shows git diff --check clean and four lint invocations without diagnostics.
- All four new programs were executed locally; observed expected outputs include sabcB, 42 and 30, successful structural Map matching (42, 73, pong), canonical equality and Closure identity checks.
- Focal Maven ProtosCliLearningMaterialsTest ran locally; maintainer subsequently reported all local tests pass. Aggregate full-suite count/run identity not supplied.
- The explicitly supplied publication_validation.py attempt returned FAIL_CLOSED because six tracked documentation files differed from candidate HEAD before commit. This is a worktree/candidate mismatch, not a test failure. No successful rerun of that exact wrapper is claimed.
- Maintainer subsequently confirmed push, clean git diff --check and all local tests passing. Exact remote product commit and changed-path inventory independently verified. Local full-suite result is maintainer-reported, not a CI or agent-executed result.

## Editor boundary

I176 already owns core parser/static/LSP support; its published evidence records formatter, static and LSP regressions. DOC029 changes no editor/runtime implementation. The VS Code extension's current TextMate policy highlights reserved tokens only, leaving contextual identifiers ordinary; match is not a reserved keyword. Its lexical fixture and validator presently lack contextual match coverage. A future extension acceptance update can add representative contextual and ordinary match forms and test the configured released runtime when available; no new parser or production LSP change is established as necessary by DOC029.

No production adoption issue is required based on the supplied inventory.
