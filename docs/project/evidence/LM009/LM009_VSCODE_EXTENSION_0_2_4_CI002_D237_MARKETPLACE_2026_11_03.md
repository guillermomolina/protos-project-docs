# LM009 — VS Code extension 0.2.4: CI002 released-runtime acceptance and Marketplace report

**Evidence date:** 2026-11-03  
**Classification:** immutable, non-normative cross-repository editor/VSIX acceptance and distribution snapshot.  
**Primary extension owner:** [CI002 / guillermomolina/protos-vscode-extension#5](https://github.com/guillermomolina/protos-vscode-extension/issues/5).  
**Protos coordination context:** [LM009 #288](https://github.com/guillermomolina/protos/issues/288), [AUD022 #1035](https://github.com/guillermomolina/protos/issues/1035), [I143 #1039](https://github.com/guillermomolina/protos/issues/1039), [BUG033 #1041](https://github.com/guillermomolina/protos/issues/1041), [BUG034 #1042](https://github.com/guillermomolina/protos/issues/1042) and [DIST021 #1044](https://github.com/guillermomolina/protos/issues/1044).  
**Authority:** the Protos specification and implementation remain owned by `guillermomolina/protos`; the extension owns its thin VSIX integration and non-normative TextMate grammar.

## Exact source and official runtime identity

| Coordinate | Evidence |
| --- | --- |
| Extension repository | `guillermomolina/protos-vscode-extension` |
| Extension published source revision | [`8c0a6ba375646569e5d204d72ec6f5b2b4e5a32b`](https://github.com/guillermomolina/protos-vscode-extension/commit/8c0a6ba375646569e5d204d72ec6f5b2b4e5a32b) |
| Previous committed extension baseline | [`7eb4744bde1a213adc1b99ce02622ad170b0fd5a`](https://github.com/guillermomolina/protos-vscode-extension/commit/7eb4744bde1a213adc1b99ce02622ad170b0fd5a) |
| GitHub-verified extension package / publisher | `0.2.4` / `guillermomolina.protos` |
| VS Code acceptance version | `1.140.0` |
| Official Protos runtime release / pinned source | `v0.3.442` / `b20271369d00f8937890a4a94048bb1754b688b8` |
| Native runtime asset | `protos-0.3.442-native-linux-x86_64.zip` |
| Native runtime archive SHA-256 | `e3aea0bc715a0351cf09d5fe3a52471d444ab17b80924cc7630bf96f2a289380` |
| GraalVM release | `25.4.4.1.1` |
| Protos upstream fixes consumed | D237 / I143, TOOL010 BUG033 and LSP Signature Help BUG034 |

GitHub source inspection of the exact extension revision confirms `package.json` version `0.2.4`, matching README identity, and the official immutable Protos runtime lock. The one-commit comparison from `7eb4744b` to `8c0a6ba3` contains 27 changed paths: real installed-VSIX CI002 expansion, D237 cases and validation, time-zone-independent canonicalization, Podman devcontainer shared-memory configuration, and final version metadata. The runtime is downloaded as an external published artifact; it is not bundled in the VSIX.

## Human-executed installed-VSIX acceptance

The maintainer supplied complete execution transcripts for the **pre-version-bump working tree**, originally packaged as extension `0.2.3`. The acceptance was run both on the EndeavourOS host (KDE Wayland with `xvfb-run`) and, after recreating the Podman devcontainer with `/dev/shm=1.0G`, inside the repository's designated devcontainer (Node.js `v22.23.2`). The successful devcontainer run checked out committed baseline `7eb4744b` with the candidate changes in its working tree, later published under `8c0a6ba3` with version-only finalization.

The complete devcontainer acceptance reported:

```text
VS_CODE_TEST_VERSION=1.140.0
CI002_LOCKED_RELEASE=v0.3.442
LOCKED_PROTOS_RELEASE=PASS
RELEASED_TOOL010_CORPUS_CASES=7
RELEASED_TOOL010_CORPUS=PASS
CI002_INSTALLED_VSIX_DIAGNOSTICS=PASS
CI002_KIND=baseline
CI002_CASE_COUNT=1
LM009_I3A_CLEAN_INSTALL=PASS
LM009_I3B_REAL_RUN=PASS
LM009_I3C_REAL_DEBUG=PASS
LM011_D2_INSTALLED_VSIX_FORMAT_DOCUMENT=PASS
LM011_E_FORMAT_DOCUMENT=PASS
LM011_E_FORMAT_ON_SAVE=PASS
CI002_KIND=features
CI002_CASE_COUNT=7
CI002_INSTALLED_VSIX_RUN=PASS
CI002_RUNTIME_AND_SOURCE_MATCH=PASS
CI002_KIND=named-arguments
CI002_CASE_COUNT=2
CI002_D237_INSTALLED_VSIX_SIGNATURE_HELP=PASS
CI002_D237_SIGNATURE_CASE_COUNT=5
CI002_D237_INSTALLED_VSIX_FORMAT=PASS
CI002_D237_FORMAT_DOCUMENT=PASS
CI002_D237_FORMAT_ON_SAVE=PASS
CI002_D237_FORMAT_IDEMPOTENCE=PASS
CI002_D237_NAMED_LABEL_PRESERVATION=PASS
CI002_INSTALLED_VSIX_RUN=PASS
CI002_RUNTIME_AND_SOURCE_MATCH=PASS
LM009_I3C_REAL_DEBUG=PASS
CI002_FEATURE_ACCEPTANCE=PASS
CI002_D237_ACCEPTANCE=PASS
PROTOS_VSCODE_ACCEPTANCE_TEST=PASS
DEVCONTAINER_CI002_ACCEPTANCE_STATUS=0
```

The 15 installed-extension scenarios include baseline real LSP diagnostics, clean installation, real Run/Debug, deterministic formatter-protocol checking, official-runtime Format Document and format-on-save, contextual-for/Range/bitwise/return diagnostics and execution/debug integration, and five D237-specific installed-VSIX scenarios (diagnostics, Signature Help, formatting, Run and Debug). The D237 Signature Help set contains five cases, including UTF-16/CRLF boundary coverage. The extension delegates actual parsing, formatting, analysis, execution and DAP behavior to the official Protos runtime rather than implementing a second language front end.

After the successful devcontainer acceptance, the maintainer also ran the pre-bump repository focal commands: `test:build`, `test:grammar`, `test:contract`, `test:packaging`, `test:run`, `test:debug`, `test:lsp` and four `node --test` CI002 modules. Reported result: **16/16 Node tests PASS**, `CI002_PREBUMP_TEST_STATUS=0` and `DIFF_CHECK_STATUS=0`. The separate full `make test` aggregate was **not run** for this candidate: its `test:release-version` guard was intentionally excluded while the published `0.2.3` identity still owned the original distribution inputs. The maintainer's workflow prohibits tests after the `0.2.4` bump. Accordingly these are pre-bump acceptance and focal-test attestations, **not a post-bump `0.2.4` installed-VSIX test transcript** or an attestation of `make test=PASS`.

## Reproducibility and devcontainer repair

A reproducibility discrepancy was reproduced between the host's `Europe/Madrid` and the container's `UTC` environment: identical ZIP member names and uncompressed bytes (10 entries, zero differing members) yielded different canonical ZIP SHA-256 values due to time-zone-dependent DOS timestamps emitted through `yazl`. The extension canonicalizer was changed to use `UTC` internally, and `test/validate_reproducibility.py` was extended to compare UTC, Europe/Madrid and America/Los_Angeles.

The pre-bump `0.2.3` candidate's canonical ZIP SHA-256 after the UTC fix was:

```text
881378d351e4d9a5ab9f8705a13721d8fe8182b798375baa7fe83aac43886afc
```

`VSIX_CANONICAL_TIMEZONE_PARITY=PASS` and `VSIX_ARTIFACT_REPRODUCIBLE=YES` were reported both on the host focal check and during the later devcontainer acceptance. **This digest belongs to the tested pre-bump `0.2.3` artifact, NOT to the final `0.2.4` Marketplace VSIX.** No final `0.2.4` VSIX SHA-256 was supplied, so no byte-equivalence assertion is possible.

The original devcontainer had approximately 64 MiB of `/dev/shm` and an intermittent VS Code/Electron Workbench `net::ERR_FAILED` failure. The Podman rootless `--userns=keep-id:uid=1000,gid=1000` setting was retained and `--shm-size=1g` added; a rebuilt container then completed all 15 scenarios. The recovery is directly observed, but the evidence does not independently isolate shared-memory size from other effects of a container rebuild.

## Distribution report and remaining release-policy gap

**Maintainer-reported (2026-11-03):** the `0.2.4` VSIX was uploaded to the Visual Studio Marketplace. The Marketplace listing/version, actual uploaded VSIX digest and publication metadata were not independently fetched; the upload is recorded as a user attestation, not a verified byte identity.

**GitHub-verified:** the `0.2.4` source revision `8c0a6ba3` and runtime lock are published. This record does **not** establish that an explicit `v0.2.4` Git tag and GitHub Release exist or that a canonical `0.2.4` VSIX + SHA-256 sidecar are attached there. No such release/tag was created as part of this evidence publication.

Under the [ratified LM009-I4 distribution policy](https://github.com/guillermomolina/protos-project-docs/blob/76ee0603d09110430908dc5d229bf8dab836fc52/docs/project/work/LM009/LM009_I4_RELEASE_DISTRIBUTION_POLICY.md), an explicit GitHub Release with the canonical VSIX and checksum is the **authoritative public release**, and Marketplace is only an optional mirror. Therefore **Marketplace upload alone does not prove canonical release-policy completion**. The exact `0.2.4` canonical artifact digest, Git tag/Release and registry byte-parity remain unverified. Do not infer a distribution release gate from the confirmed installed-VSIX functional results.

## Coordination and non-effects

- [CI002 / extension #5](https://github.com/guillermomolina/protos-vscode-extension/issues/5) is the owning open Issue for the installed-VSIX work and any follow-up release-artifact reconciliation; this snapshot is not a substitute for its live lifecycle state.
- [AUD022 / Protos #1035](https://github.com/guillermomolina/protos/issues/1035) stays closed; the new end-to-end installed-editor evidence is downstream of the historical source audit rather than a reopened audit.
- [I143 / Protos #1039](https://github.com/guillermomolina/protos/issues/1039), [BUG033 #1041](https://github.com/guillermomolina/protos/issues/1041) and [BUG034 #1042](https://github.com/guillermomolina/protos/issues/1042) remain completed implementation owners; this record adds independently executed installed-VSIX integration evidence.
- [DIST021 / Protos #1044](https://github.com/guillermomolina/protos/issues/1044) owns the already released `v0.3.442` Protos runtime; this extension record consumes that official artifact but neither changes its release identity nor its specification.
- No Protos language/specification/runtime changes and no new design approval are claimed here. No GitHub Issue is closed or reopened solely by publishing this snapshot.
