# LM010-D — real VS Code acceptance and formal LM010 closure evidence

**Date:** 2026-10-09. **Work item:** [guillermomolina/protos#493](https://github.com/guillermomolina/protos/issues/493). **Role:** revision-specific, non-normative evidence of integrated LSP publication and manually observed real-editor behavior. This record is not new language semantics, an editor extension release, or a statement that an automated VS Code acceptance suite was executed.

```text
PRODUCT_REPOSITORY=guillermomolina/protos
LM010_D_PRODUCT_REVISION=5ade2e2e5950f45cb8ca8a7f40682c73fc72acc2
LM010_D_PRODUCT_COMMIT=LM010-D: integrated LSP transport acceptance test (#493)
RELEASE_TAG=v0.3.312
RELEASE_PRODUCT_SOURCE_REVISION=f8ff34498f3a193c9bfe15f215a181c23504a6ad
RELEASE_NATIVE_ASSET=protos-0.3.312-native-linux-x86_64.zip
RELEASE_NATIVE_SHA256=70f893d225419713b312054f3558e2e4b3f4169e8e7bd37e015aff92157d067b
DEVCONTAINER_REPOSITORY=guillermomolina/protos-devcontainer
DEVCONTAINER_REVISION=5f5993c1b9eecb74a7b9fe5a71d842bc3e4a057f
DEVCONTAINER_PINNED_PROTOS=0.3.312
DEVCONTAINER_PINNED_VSCODE_EXTENSION=guillermomolina.protos@0.2.2
MAINTAINER_REAL_EDITOR_ACCEPTANCE=PASS_REPORTED_FOR_EXPLICIT_CASES
MAINTAINER_FOCAL_AND_MAKE_TEST=PASS_REPORTED_FOR_LM010_D
COORDINATOR_RAN_TESTS=NO
COORDINATOR_OBSERVED_VSCODE_UI=NO
NORMATIVE_SPEC_CHANGED_BY_LM010_D=NO
```

## Published implementation and runtime identities

The precise LM010-D [Protos source commit](https://github.com/guillermomolina/protos/commit/5ade2e2e5950f45cb8ca8a7f40682c73fc72acc2) adds `src/test/java/com/guillermomolina/protos/lsp/ProtosLanguageServerIntegratedEditorTest.java` and updates `pom.xml` and `CHANGELOG.md` to `0.3.309-SNAPSHOT`. The integrated Java acceptance drives Hover, Completion and literal-Closure Signature Help through LSP4J JSON-RPC/stdio serialization along with initialization, document synchronization, authority, diagnostics, definition and lifecycle. This is protocol acceptance, not a native executable subprocess or a VS Code UI test. The implementing agent reported focal tests and `make test` PASS; that validation was not re-executed or independently instrumented by the coordinating agent.

Protos [v0.3.312](https://github.com/guillermomolina/protos/releases/tag/v0.3.312) is the published Native release used for real-editor acceptance, with exact release source `f8ff34498f3a193c9bfe15f215a181c23504a6ad`. The [devcontainer published revision](https://github.com/guillermomolina/protos-devcontainer/commit/5f5993c1b9eecb74a7b9fe5a71d842bc3e4a057f) pins Native `0.3.312` and extension `0.2.2`, including the release asset's checksum. The maintainer independently showed `protos --version` -> `Protos 0.3.312` from inside the running container, and confirmed the installed VS Code Server extension directory `/home/vscode/.vscode-server/extensions/guillermomolina.protos-0.2.2`. The extension is the pre-existing standard LSP client; no extension implementation changes are represented by LM010-D.

## Human-observed VS Code editor acceptance

The maintainer used the real VS Code UI attached to the running Protos Dev Container with a canonical project fixture containing exact `protos.toml`, `protos.lock`, `protos.project` metadata and an open `Main.protos` source. This requirement matters: loose `.protos` files without canonical project authority intentionally return empty navigation/intelligence answers.

The following are **maintainer-reported manual observations from the conversation**, not screenshots, automated VS Code tests, or coordinator-executed commands:

| Editor check | Observed result | Scope and expected behavior |
| --- | --- | --- |
| Hover | PASS reported | Pointer over a known Closure parameter reference displays static information |
| Signature Help | PASS reported | Direct literal Closure call shows parameters on `(` or comma |
| Completion | PASS reported | `Ctrl+Space` after `fir` in a Closure suggests proven parameter `first` |
| Unsaved editor buffer | PASS reported | Rename parameter to `primary`, type `pri`, and see `primary` completion before save |
| Conservative negative | PASS reported | Named call `f(1, 2)` where `f` holds a Closure does **not** invent a statically proven signature |
| Diagnostics coexistence | PASS reported | Deliberately malformed `f: (` produces a visible editor syntax diagnostic |
| Formatting coexistence | PASS reported | VS Code Format Document works on restored valid source |

The maintainer answered affirmatively to the Hover/Signature Help checks, Completion, unsaved-buffer behavior, and finally the three conservative-negative/diagnostics/formatting checks. The report is accepted as human-observed proof of those exact cases. The conversation did **not** separately document manual UI inspection of every conceivable URI/UTF-16/non-BMP/CRLF, all LSP lifecycle paths, document symbol/navigation interactions, editor restart, or a performance benchmark; those cannot be claimed as independently observed UI results. Scope and regression boundaries outside the manually exercised cases remain covered by the published automated protocol and feature tests, to the extent of their explicit assertions.

## Closure assessment

- LM010-A Hover, LM010-B Completion and LM010-C Signature Help are independently published, and their prior evidence is retained in this directory.
- LM010-D integrated protocol test is published at an exact Protos revision; maintainer-reported local focal tests and `make test` PASS.
- Real VS Code/Native editor integration for the three LM010 features and their key positive/negative/unsaved/regression cases has now been maintainer-observed, using a released and checksum-pinned Protos runtime.
- No TypeScript-side semantic duplication, extension release, new Dxxx/PLATxxx decision, or normative language change is needed for this bounded workstream.
- The known broader editor and tooling ambitions (e.g. dynamic call-site inference, rename/refactoring, richer completion, and separate LM009 extension distribution) remain intentionally outside LM010's narrow accepted generation-one contract.

**Decision:** LM010/#493 is eligible for formal closure as **completed within its bounded published scope** once this record is committed, read back, and linked in a final GitHub Issue comment. The live Issue state, rather than this historical evidence file, records whether the formal closing transaction has completed.

```text
LM010_A=IMPLEMENTED_PUBLISHED
LM010_B=IMPLEMENTED_PUBLISHED
LM010_C=IMPLEMENTED_PUBLISHED
LM010_D=INTEGRATED_TRANSPORT_PUBLISHED
REAL_VSCODE_ACCEPTANCE=PASS_MAINTAINER_REPORTED_BOUNDED_MATRIX
DURABLE_RECORD_DECISION=REQUIRED
REQUIRED_DURABLE_PUBLICATION=SATISFIED_AFTER_VERIFIED_COMMIT
ISSUE_CLOSURE_COMMENT=REQUIRED_BEFORE_CLOSE
```
