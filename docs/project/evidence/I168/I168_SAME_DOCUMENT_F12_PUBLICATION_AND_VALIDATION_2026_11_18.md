# I168 — Document-local Go to Definition implementation publication and validation (2026-11-18)

**Type:** revision-bound implementation evidence and validation checkpoint; not a final claim of installed VS Code F12 acceptance.
**Owning work item:** [I168 / protos#1135](https://github.com/guillermomolina/protos/issues/1135).
**Approved source-authority decision:** [D258-B / protos#1134](https://github.com/guillermomolina/protos/issues/1134), [ratified record](../../decisions/tooling/D258_UNBOUND_DOCUMENT_LOCAL_GO_TO_DEFINITION_AUTHORITY.md).
**Exact published Protos implementation commit:** [`guillermomolina/protos@e59693eafa634f81eb5708a20925c3847ee41e52`](https://github.com/guillermomolina/protos/commit/e59693eafa634f81eb5708a20925c3847ee41e52).
**Commit subject:** `I168: Prove local slot and own-member definitions with conservative context-write invalidation`.
**Version recorded in that exact commit:** `0.4.167-SNAPSHOT` (`pom.xml`; `CHANGELOG.md` has matching I168 entry).
**Other concurrent changes:** `main` advanced subsequently to `0.4.168-SNAPSHOT` under a separate I115 commit; the later version does not change the identity of the published I168 implementation.

## Publication audit

The published I168 commit changes **11 tracked files**; no new Java production class, alternative resolver, Truffle runtime node, Native Image profile, workspace ProjectBinding provider, or VS Code extension source is introduced.

- Implementation: `src/main/java/com/guillermomolina/protos/analysis/ProtosStaticDefinitions.java` and `src/main/java/com/guillermomolina/protos/lsp/ProtosTextDocumentService.java`.
- Regression tests: `src/test/java/com/guillermomolina/protos/analysis/ProtosStaticCompletionTest.java`, `ProtosStaticDefinitionsTest.java`, `ProtosStaticHoverTest.java`, `ProtosStaticReferencesTest.java`; `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerDefinitionTest.java` and `ProtosLanguageServerIntegratedEditorTest.java`.
- Maintained user-facing guide: `docs/guide/tools/language-server.md`.
- Finalization metadata in the same product commit: `pom.xml`, `CHANGELOG.md`.

The local-source authority gate in `ProtosTextDocumentService.definition()` is removed only for **complete proven origins within the same exact open URI and immutable snapshot**; it retains currentness and target equality checks, without changing the canonical gate for References or Workspace Symbols.

The existing parser-backed D110 `ProtosStaticDefinitions` now tracks same-activation local slot creations, stored Closures, provable assignment destinations and own members of statically proven bare object literals. It observes ordered evaluation, excludes unknown inherited and dynamic member lookup, invalidates facts conservatively after opaque effects and context/receiver writes, and does not promote arbitrary same-name matches to definitions. Parameter-only Completion and truthful Hover descriptions are protected by tests. D124 References continues to invert only proven, complete definitions for authorized project sources; selector collection has its own current limits.

**No guest execution semantics changed.** The decision source span, snapshot and proof machinery remains in static analysis; the implementation does not introduce any runtime Java infrastructure.

## Maintainer-provided focal validation (observed in chat attachments)

Exact command supplied in the local log:

```text
mvn test -Dtest='ProtosStaticDefinitionsTest,ProtosStaticReferencesTest,ProtosLanguageServerIntegratedEditorTest,ProtosLanguageServerDefinitionTest,ProtosStaticCompletionTest,ProtosStaticHoverTest'
```

The attachment reports **`BUILD SUCCESS`**, completed **2026-11-18T07:04:34Z** in `24.888 s`, on the locally built **`0.4.166-SNAPSHOT`** substantive pre-finalization code:

| Suite | Tests | Failures | Errors | Skipped |
| --- | ---: | ---: | ---: | ---: |
| `ProtosStaticCompletionTest` | 13 | 0 | 0 | 0 |
| `ProtosStaticDefinitionsTest` | 48 | 0 | 0 | 0 |
| `ProtosStaticHoverTest` | 13 | 0 | 0 | 0 |
| `ProtosStaticReferencesTest` | 19 | 0 | 0 | 0 |
| `ProtosLanguageServerDefinitionTest` | 8 | 0 | 0 | 0 |
| `ProtosLanguageServerIntegratedEditorTest` | 10 | 0 | 0 | 0 |
| **Total** | **111** | **0** | **0** | **0** |

A previous focal execution after the initial breadth expansion also reported 98/98 PASS on the pre-finalization code, before additional context-write and D124 regression changes. The final 111/111 execution is the stronger relevant focal evidence.

The owner then stated after **publishing the exact I168 commit**:

- `git diff --check` clean.
- **All local tests pass.**
- The implementation was pushed to `guillermomolina/protos`.

The published commit and its changed paths, version/changelog and exact SHA were independently verified through read-only GitHub inspection. The owner's statement confirms local full-test success, **but no exact post-finalization full-suite command/output, exit status, test counts or report artifacts were supplied**. Therefore the 111/111 focal run must not be represented as an exact-byte post-commit test of `0.4.167-SNAPSHOT`, and the reported full local validation must remain owner-attested rather than independently reconstructed.

## Acceptance and remaining explicit gate

**Proven here:**

1. Published, identifiable product commit with finalization metadata.
2. Positive focused static-analysis and real JSON-RPC LSP test results, including same-URI loose-file Definition, slot/Closure/assignment/member proof, aliases, unsaved change/close, UTF-16 and context mutation regressions.
3. Maintainer attestation of clean whitespace check and all local tests passing after push.
4. Original D258-B canonical ProjectBinding boundaries preserved for References/Workspace Symbols.

**Not yet proven here:** manual **installed VS Code extension F12 acceptance against an installed Native SNAPSHOT that contains I168**, including `saludar()`, bare slot reads, parameter and a provable `obj.metodo()` in an ordinary open Protos file with no `package.toml`, `protos.lock` or `protos.project`. The only earlier full installed-VSIX CI002 evidence used published `v0.4.158` and did not exercise F12. No Native ZIP build/install/smoke, new native artifact SHA, installed `protos --version`, extension runtime selection, or manual F12 result is claimed by this record.

The current stable Native installer documentation permits installing a locally built `protos-<version>-SNAPSHOT-native-linux-x86_64.zip` and its adjacent `.zip.sha256` offline via `sh dist/install.sh --inspect --local-native <absolute-zip-path>` and `sh dist/install.sh --local-native <absolute-zip-path>`. Source/build/installation identity must be recorded from the real produced artifact, not inferred from the source commit. A subsequent `0.4.168-SNAPSHOT` Native build includes other concurrent work and must have its own exact source identity.

**Coordination outcome:** preserve I168 as **open / review pending real VS Code F12 acceptance**; do not claim I168 fully closed or a public release. The existing D258 design issue remains closed with its independent ratification intact. No additional implementation issue is required for the same-file F12 goal.

## Compact evidence markers

```text
WORK_ITEM=I168
APPROVED_DECISION=D258_B
PROTOS_COMMIT=e59693eafa634f81eb5708a20925c3847ee41e52
PROTOS_COMMIT_VERIFIED=PASS
PROTOS_COMMIT_VERSION=0.4.167-SNAPSHOT
PROTOS_CHANGELOG_AND_POM_IN_SAME_COMMIT=PASS
FOCAL_PRE_FINALIZATION_JUNIT=111_111_PASS
FOCAL_FAILURES=0
FOCAL_ERRORS=0
FOCAL_SKIPPED=0
FOCAL_FINISHED_UTC=2026-11-18T07:04:34Z
FINAL_LOCAL_TESTS=PASS_OWNER_REPORTED
FINAL_LOCAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
EXACT_POST_COMMIT_FULL_SUITE_TRANSCRIPT=NOT_PROVIDED
NEW_PRODUCTION_JAVA_CLASSES=NONE
RUNTIME_AND_JIT_SEMANTICS=UNCHANGED
REAL_VSCODE_F12_WITH_I168=NOT_YET_PROVIDED
NATIVE_SNAPSHOT_INSTALLED=NOT_YET_VERIFIED
ISSUE_CLOSURE=WAITING_REAL_EDITOR_ACCEPTANCE
```
