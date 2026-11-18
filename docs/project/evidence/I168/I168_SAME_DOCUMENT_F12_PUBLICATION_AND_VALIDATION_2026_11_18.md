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
---

## Addendum — lexical callback navigation, object construction and installed-editor acceptance (2026-11-18)

**Scope of this addendum:** later product publication and owner-observed functional acceptance. The preceding `0.4.167-SNAPSHOT` checkpoint is historical evidence and remains intact.

**Exact later product revision:** [`guillermomolina/protos@01c8566e6481501b051ee67d475e1d67accadb4a`](https://github.com/guillermomolina/protos/commit/01c8566e6481501b051ee67d475e1d67accadb4a).
**Commit subject:** `I168: Navigate Definition to lexical declarations in Closures and object construction contexts`.
**Published product implementation version:** `0.4.170-SNAPSHOT`, confirmed in the exact commit's `pom.xml` and matching root `CHANGELOG.md`.
**Files:** 15 modified tracked paths including `pom.xml`, `CHANGELOG.md`, maintained language-server guide, analysis/LSP production code, and tests. No new production Java class or runtime/JIT semantics change is claimed.

### Behavior and proof boundaries

- `textDocument/definition` now navigates within the exact open URI and snapshot from bare reads inside ordinary, immediate, stored and deferred Closures, including callbacks to methods of arbitrary names. Source-backed lexical-context identity is constructed from the existing Protos AST in a two-pass, query-local scope walk. The implementation does not need to infer the implementation or invocation time of `Range.each` to navigate the lexical read `finalDoors`.
- During object construction, direct reads can navigate to earlier own-slot declarations of the object being built. Closures and nested objects constructed inside that body skip the constructed object when selecting a lexical parent, preserving Protos's genuine lexical-context capture rule.
- Parameter navigation targets the identifier alone, excluding default expressions and the rest prefix; F12 also accepts a cursor immediately after the reference span, while keeping the parser's half-open spans.
- The existing exact-origin proof remains a separate query used by References and Hover; Completion remains limited to its exact proven-parameter policy. References and Workspace Symbols still require ProjectBinding. Neither cross-file navigation nor unbound package/source authority has been added.
- **Contract distinction:** in `ProtosStaticAnalysisCore.definition()`, an exact proven origin is preferred, but when unavailable a bare-name read may navigate to AST-derived lexical declaration candidate(s). These are *not* a complete proven set of dynamically selected slot origins. In particular, unknown calls, mutable aliases, `removeSlot`, recreated slots and delayed callback invocation can invalidate the exact-origin proof without changing the source location of the lexical declaration.

### Reported validation

The maintainer supplied the focal Maven output for substantive code at `0.4.169-SNAPSHOT`, before final commit/version finalization:

| Test class | Tests | Failures | Errors | Skipped |
| --- | ---: | ---: | ---: | ---: |
| `ProtosStaticDefinitionsTest` | 110 | 0 | 0 | 0 |
| `ProtosStaticReferencesTest` | 24 | 0 | 0 | 0 |
| `ProtosStaticHoverTest` | 17 | 0 | 0 | 0 |
| `ProtosStaticCompletionTest` | 18 | 0 | 0 | 0 |
| `ProtosLanguageServerDefinitionTest` | 17 | 0 | 0 | 0 |
| `ProtosLanguageServerIntegratedEditorTest` | 17 | 0 | 0 | 0 |
| `ProtosLm009FLanguageServerCliTest` | 4 | 0 | 0 | 0 |
| **Total** | **207** | **0** | **0** | **0** |

Focal command: `mvn test -Dtest='ProtosStaticDefinitionsTest,ProtosStaticHoverTest,ProtosStaticReferencesTest,ProtosStaticCompletionTest,ProtosLanguageServerDefinitionTest,ProtosLanguageServerIntegratedEditorTest,ProtosLm009FLanguageServerCliTest'`. The log ends `BUILD SUCCESS` on 2026-11-18 at 11:38:23Z; `git diff --check` was clean. The earlier intermediate focal checkpoints were 79/79, 55/55, 158 with one parameter-span assertion failure, 163/163 and 196/196; the final 207/207 focal supersedes the failed intermediate check.

The maintainer **later explicitly reported** that the implementation was pushed, the Git diff check was clean, and **all local tests passed**. This is owner-attested final/integrated validation: no final post-commit full-suite transcript or report count was supplied and none is invented here.

**Installed-editor acceptance — owner reported PASS:** after installing a locally built Protos SNAPSHOT and testing the extension in VS Code, the maintainer wrote **"ya esta, funciona perfecto"**, confirming the earlier F12 acceptance problem is resolved in the actual editor. The earlier manual tests had established caret-inside navigation for `x` in `slots.protos` and `doorsAfterPasses` in `100-doors.protos`, with the selected-identifier boundary and captured `finalDoors` as the missing cases. The new integrated JSON-RPC test tests the *real complete* `protos/examples/algorithms/100-doors.protos` and asserts that F12 on `finalDoors` inside the `each` callback returns the `finalDoors:` creation for cursor start/interior/end, including snapshot change and document close. Exact installed Native artifact digest, binary/source cleanliness marker and installed-extension revision were not provided; do not equate that local binary with a separately verified post-publication `0.4.170-SNAPSHOT` artifact.

### Design governance still requiring reconciliation

[D110 Candidate B′](../../decisions/tooling/D110_STATIC_GO_TO_DEFINITION_PROOF_BOUNDARY_IDENTITY_AND_AMBIGUITY_CONTRACT.md) requires standard `textDocument/definition` to return only a **complete finite static proof set of exact runtime-selectable binding origins**; if that proof is incomplete, it must return no definition. [D258 Candidate B](../../decisions/tooling/D258_UNBOUND_DOCUMENT_LOCAL_GO_TO_DEFINITION_AUTHORITY.md) amended *only* the same-open-snapshot source authority, not the D110 semantic completeness guarantee.

The `0.4.170-SNAPSHOT` lexical-navigation fallback thus materially widens Definition behavior beyond the ratified D110/D258 contracts. It is useful, backed by Truffle lexical-scope precedents, and owner-tested in VS Code, but **successful implementation and functional acceptance are not, by themselves, ratification of a replacement D110 policy**. The exact approval/invariant reconciliation must be recorded through the canonical Protos design process before claiming formal contract closure. Do not retroactively rewrite the historical D110 or D258 decision records, and do not label this as an already-approved amendment.

**Coordination recommendation:** the original real-editor acceptance gate is satisfied; I168 can be recorded as *implemented, published, locally tested, editor-accepted*, while formal closure remains in **Review** pending the explicit D110/D258 policy reconciliation. No public release, new I-family issue, guest runtime change or blanket project-authority expansion is claimed.

### Revision-bound markers

```text
WORK_ITEM=I168
EARLIER_PRODUCT_COMMIT=e59693eafa634f81eb5708a20925c3847ee41e52
LATEST_I168_PRODUCT_COMMIT=01c8566e6481501b051ee67d475e1d67accadb4a
LATEST_I168_PRODUCT_VERSION=0.4.170-SNAPSHOT
LATEST_I168_PRODUCT_COMMIT_VERIFIED=PASS
FOCAL_PRE_FINALIZATION_JUNIT=207_207_PASS
FOCAL_FAILURES=0
FOCAL_ERRORS=0
FOCAL_SKIPPED=0
FINAL_LOCAL_TESTS=PASS_OWNER_REPORTED
FINAL_LOCAL_GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
REAL_VSCODE_F12=PASS_OWNER_REPORTED
EXACT_POST_COMMIT_FULL_SUITE_TRANSCRIPT=NOT_PROVIDED
INSTALLED_NATIVE_ARTIFACT_SHA256=NOT_PROVIDED
D110_LEXICAL_NAVIGATION_POLICY_RECONCILIATION=PENDING_EXPLICIT_APPROVAL
RUNTIME_AND_JIT_SEMANTICS=UNCHANGED
PUBLIC_RELEASE=NOT_CLAIMED
ISSUE_CLOSURE=REVIEW_PENDING_DESIGN_RECONCILIATION
```
