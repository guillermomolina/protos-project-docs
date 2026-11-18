# LM009 / external CI002 — VS Code extension acceptance against Protos v0.4.158 (2026-11-18)

**Evidence date:** 2026-11-18. **Core cross-reference:** [LM009 #288](https://github.com/guillermomolina/protos/issues/288), previously completed; the independently owning extension work item is [CI002 #5](https://github.com/guillermomolina/protos-vscode-extension/issues/5). **Related core work:** [I151 #1066](https://github.com/guillermomolina/protos/issues/1066) (D242 module/SOURCE migration) and [DIST024 #1129](https://github.com/guillermomolina/protos/issues/1129) (published release). This is **non-normative, revision-bound acceptance evidence**, not a new language decision or an alteration to closed work.

## Exact published identities

| Coordinate | Value / provenance |
| --- | --- |
| Core repository | `guillermomolina/protos` |
| Core public release | [`v0.4.158`](https://github.com/guillermomolina/protos/releases/tag/v0.4.158), published 2026-11-18T04:32:02Z, prerelease |
| Core release-only source revision (`PROTOS_REVISION`) | [`6b66acf256157dcaec4810c5ab5ec83a5026b939`](https://github.com/guillermomolina/protos/commit/6b66acf256157dcaec4810c5ab5ec83a5026b939) |
| Published Native archive | `protos-0.4.158-native-linux-x86_64.zip` |
| Native ZIP SHA-256 | `08ba2986c5efa82d1a4f670a9fb7c2e10bc67609e7e81836e94920a7704d5233` |
| GraalVM release | `25.5.5.0.3` |
| Extension repository | `guillermomolina/protos-vscode-extension` |
| Published extension source revision (`EXTENSION_REVISION`) | [`9216e0b06f19497de7c5030f4f36de500a020f64`](https://github.com/guillermomolina/protos-vscode-extension/commit/9216e0b06f19497de7c5030f4f36de500a020f64) |
| Extension package version | `0.2.4` (unreleased candidate at acceptance; no Marketplace release is implied) |
| Test editor version | VS Code `1.140.0`, Linux x86_64 development container |
| Local pre-commit canonical VSIX SHA-256 | `35171b8f1766a343460c6ac8e61dcf435dabf1f83a32c2c9ff7d5fb59b44f141` |

The `VSIX_SHA256` immediately above belongs to the **tested pre-commit working-tree package**, not to a separately published extension release or to a post-commit rebuild. The acceptance script reported `CI002_VSIX_SOURCE_REVISION=8c0a6ba375646569e5d204d72ec6f5b2b4e5a32b`, its prior committed HEAD, while four uncommitted changes were already present. The subsequent extension commit `9216e0b...` publishes those exact four changed paths. The maintainer reported no post-push rebuild or test rerun, and no published Marketplace/VSIX asset is claimed by this record.

## Migration from previous locked runtime

The current extension initially passed the full installed-VSIX acceptance against official Protos `v0.3.442` (extension `0.2.4`, baseline `8c0a6ba...`). It was then repinned to official `v0.4.158` by updating its lock and corresponding README identity.

The first `v0.4.158` full `make test` run passed repository gates, binary ZIP/SOURCE/RUNTIME identity, installed-VSIX baseline Run, Debug, formatter and LSP diagnostics, and feature diagnostics/formatting, but failed at `ci002-run-features`: the VS Code task's executable, one source argument, and working directory matched the intended runtime invocation; Protos returned process exit code 1. The maintainer independently reproduced the failure by running the extracted official Native binary on the original fixture: an `Error: Object {}` arose from `protos/lib/core/import.protos:33:14`, reached at fixture line 4, with `EXIT_CODE=1`.

The specific cause was **outdated test source imports** introduced by D242's intentionally breaking Standard Library module layout, not a verified defect in `extension.js`: both assertion fixtures still used `import("std:test/Assertions")`, the old source path present in `v0.3.442`. Protos `v0.4.158` instead exports the canonical logical module at `std:test/assertions`, implemented in `protos/lib/test/assertions/module.protos`. No compatibility alias was added to Protos.

Both fixture imports were changed to `import("std:test/assertions")`; the maintainer ran the two complete fixture programs directly with official `v0.4.158`, receiving `FEATURE_EXIT=0` and `NAMED_EXIT=0`, respectively, and both expected final assertion-completion markers.

## Exact extension publication delta

Independently verified GitHub commit [`9216e0b...`](https://github.com/guillermomolina/protos-vscode-extension/commit/9216e0b06f19497de7c5030f4f36de500a020f64) has exactly four changed files (**13 insertions, 13 deletions**, as reported by the maintainer):

1. `protos-source.lock.json` — release tag, candidate SHA, official Native ZIP identity and SHA-256, GraalVM version.
2. `README.md` — matching published runtime identity.
3. `test/fixtures/ci002_run_features.protos` — canonical D242 assertion module import.
4. `test/fixtures/ci002_named_run_features.protos` — the same import correction for named-argument execution.

No changes were made to the extension's production JavaScript, LSP client, DAP adapter, canonical Protos runtime, or normative specification.

## Maintainer-reported installed-VSIX acceptance after correction

The maintainer executed `make test` in `/workspaces/protos-vscode-extension` after correcting the fixture imports, and reported **all local tests passing** and `git diff --check` clean; the provided terminal transcript includes the following:

| Gate | Outcome |
| --- | --- |
| `npm test` build, lexical grammar, extension contract, packaging, version guard, Run, Debug and LSP integration | PASS |
| CI002 structural acceptance-plan/release-gate Node tests | **16/16 PASS** |
| Official extracted Native runtime source/release/checksum/GraalVM identity | `LOCKED_PROTOS_RELEASE=PASS` |
| Actual `bin/protos --version` | `Protos 0.4.158` |
| Canonical VSIX content, deterministic packaging and reproducibility | PASS |
| Installed VSIX clean install, real Run, real DAP breakpoint/stack/locals/step/continue/termination | PASS |
| Real released-runtime Format Document, unsaved buffer format, Format on Save | PASS |
| CI002 released feature LSP diagnostics, formatting, Run and bitwise Debug | PASS |
| D237 named-argument LSP diagnostics, Signature Help, formatting, Run and Debug | PASS |
| Final markers | `CI002_FEATURE_ACCEPTANCE=PASS`; `CI002_D237_ACCEPTANCE=PASS`; `PROTOS_VSCODE_ACCEPTANCE_TEST=PASS` |
| Local `git diff --check` | CLEAN, maintainer-reported |
| Extension source commit/push to `main` | VERIFIED by GitHub at `9216e0b...` |

The generated acceptance JSON is named `/tmp/protos-vscode-acceptance/ci002-evidence.json` in the user's container; **the JSON file is not included in this repository record**. Its contents and any post-commit VSIX digest were not independently downloaded. This record is grounded in the maintainer's logs and statements plus read-only verification of the official core tag and extension source commit. It does not claim independent execution of the tests, additional untested feature compatibility, or a published `0.2.4` extension release.

## Ownership and closure interpretation

- **CI002 #5**, in the extension repository, owns the VSIX compatibility gate and its issue closure.
- **LM009 #288**, **I151 #1066** and **DIST024 #1129** in core Protos were already closed before this follow-up; this new result does not reopen or replace their historical acceptance and publication records.
- The successful tests demonstrate compatibility of this exact extension source and released Native Protos runtime for the **existing CI002 test matrix**, not a general promise of cross-version compatibility. D242's case-sensitive module path migration remains normative under core `spec/semantics/MODULES.md`.

```text
EVIDENCE=LM009_EXTERNAL_CI002_VSCODE_0_4_158
PROTOS_REVISION=6b66acf256157dcaec4810c5ab5ec83a5026b939
EXTENSION_REVISION=9216e0b06f19497de7c5030f4f36de500a020f64
EXTENSION_VERSION=0.2.4
OFFICIAL_RUNTIME_RELEASE=v0.4.158
OFFICIAL_NATIVE_ZIP_SHA256=08ba2986c5efa82d1a4f670a9fb7c2e10bc67609e7e81836e94920a7704d5233
VSCODE_VERSION=1.140.0
CI002_NODE_TESTS=16/16_PASS
CI002_FEATURE_ACCEPTANCE=PASS_REPORTED
CI002_D237_ACCEPTANCE=PASS_REPORTED
PROTOS_VSCODE_ACCEPTANCE_TEST=PASS_REPORTED
GIT_DIFF_CHECK=CLEAN_REPORTED
EXTENSION_SOURCE_PUBLICATION=PASS_GITHUB_VERIFIED
POST_COMMIT_TEST_EXECUTION=NOT_REPORTED
VSIX_MARKETPLACE_RELEASE=NOT_CLAIMED
PROJECT_RECORD_REVISION=SEE_PUBLISHING_COMMIT_AND_LIVE_ISSUE
```
