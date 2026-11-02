# LM009 — VS Code extension 0.2.3: TextMate alignment and Marketplace report

**Evidence date:** 2026-11-02  
**Classification:** immutable, non-normative editor/extension distribution evidence, linked to existing LM009 and AUD022 formal authority.  
**Semantics:** no Protos language, runtime, Standard Library, LSP or DAP semantic change is claimed.

## Ownership and exact product identity

| Field | Evidence |
| --- | --- |
| Protos audit work | [AUD022 / guillermomolina/protos#1035](https://github.com/guillermomolina/protos/issues/1035) |
| Editor maturity/distribution work | [LM009 / #288](https://github.com/guillermomolina/protos/issues/288), [LM009-I / #474](https://github.com/guillermomolina/protos/issues/474) |
| Shared TextMate authority | [D175 / #523](https://github.com/guillermomolina/protos/issues/523) |
| Extension fix | [BUG002 / guillermomolina/protos-vscode-extension#4](https://github.com/guillermomolina/protos-vscode-extension/issues/4), closed completed |
| Extension repository | `guillermomolina/protos-vscode-extension` |
| Published extension source revision | [`82ceb83872b3ef25362b4b817352b088d21cee59`](https://github.com/guillermomolina/protos-vscode-extension/commit/82ceb83872b3ef25362b4b817352b088d21cee59) |
| Extension package version | `0.2.3` |
| Marketplace extension identity | `guillermomolina.protos` |
| Website grammar consumption | Requires a separate exact-revision pin update; not completed by this work |

The GitHub `main` reference for the extension was checked to point to the exact published commit above. The commit changed 13 files, including `syntaxes/protos.tmLanguage.json`, representative lexical fixtures and guards, packaged/installed VSIX checks, VSIX checksum exclusion, and coordinated version metadata.

## BUG002 correction and architectural boundary

- Removed the false `args` intrinsic scope. The canonical reserved inventory is exactly `this`, `context`, `super`, `true`, `false` and `null`; `args` remains an ordinary identifier.
- Replaced invalid-operator examples from the valid-source fixture and added representative ordinary/contextual names, Unicode identifiers and member names, contextual `for` and `return`, `Range`/`InvertedRange`, `each`/`eachStep`, and bitwise/shift syntax.
- Added regression checks for exact reserved scopes, fixture markers, broad symbolic coloring, byte parity of the TextMate source and packaged grammar, and inspection of the installed grammar contribution.
- Reconciled `.vscodeignore` and packaging validation so `*.vsix.sha256` files are excluded from VSIX payloads.
- Deliberately **retained** broad non-normative TextMate symbolic-operator highlighting. A colored invalid run is not a parser acceptance claim. No second parser or source-semantic authority was introduced.

The website must continue to consume the shared grammar from an **exact extension commit**, as required by [D175](https://github.com/guillermomolina/protos/issues/523). This record does not assert that the website pin has been updated.

## Reported validation and limitations

The human executor supplied results from the extension repository:

```text
npm run test:grammar=PASS
npm run test:packaging=PASS
VSIX_PACKAGE_VALIDATION=PASS
TEXTMATE_GRAMMAR_BYTE_PARITY=PASS
VSIX_CHECKSUM_EXCLUDED=PASS
git diff --check=PASS
PROTOS_VSCODE_ACCEPTANCE_TEST=PASS
```

Installed-VSIX acceptance under VS Code `1.140.0` included clean-install/grammar checks and real released-runtime formatting-on-save. The detailed acceptance transcript belongs to the **pre-version-bump `0.2.2` test candidate**, followed by a coordinated `0.2.3` metadata update and extension publication at the exact commit above. It must not be presented as a captured post-bump canonical `0.2.3` VSIX acceptance transcript.

After reporting Marketplace upload, the owner additionally stated that local `make test`, `make check`, and `git diff --check` pass. That is an **owner attestation**, not independently retrieved CI logs; no independently verified tested source revision was supplied for that later statement.

## Distribution state at evidence capture

**User-reported:** extension `0.2.3` was uploaded to Visual Studio Marketplace. The Marketplace listing/version and uploaded bytes were not independently verified by this record. No Marketplace package digest was supplied.

**GitHub-verified:** as of the 2026-11-02 check, `guillermomolina/protos-vscode-extension` had no `refs/tags/v0.2.3` and no GitHub Release at `v0.2.3` (both tag and release lookups returned 404).

This is a **distribution evidence gap**, not a failure of the completed TextMate/VSIX implementation. The [ratified LM009-I4 release policy](https://github.com/guillermomolina/protos-project-docs/blob/76ee0603d09110430908dc5d229bf8dab836fc52/docs/project/work/LM009/LM009_I4_RELEASE_DISTRIBUTION_POLICY.md) requires an explicit tagged canonical GitHub Release carrying the canonical VSIX and SHA-256 before Marketplace is described as a byte-identical mirror.

No `v0.2.3` tag, GitHub Release, canonical VSIX SHA-256, Marketplace-artifact equality, or release-policy completion is claimed. Creating a public GitHub Release requires a separate explicit release action.

## Cross-repository disposition

- **Extension BUG002:** completed and closed at the exact revision above.
- **Protos AUD022:** existing audit remains completed; this is a downstream editor evidence addition, not a reopening of the core language audit.
- **D175:** existing ratified authority unchanged; website exact-revision refresh remains independently owned.
- **LM009/LM009-I:** previously completed work remains closed; new distribution evidence and the uncompleted canonical `0.2.3` release contract are recorded without rewriting historical releases or inventing a new work identifier.

This record is immutable evidence for the 2026-11-02 state. Future completion of the canonical release and website pin should be recorded with their own exact revision/artifact references rather than retroactively treating this snapshot as proof.
