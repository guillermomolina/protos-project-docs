# LM013 / 0.3.389 — Real VS Code loose-file Native LSP acceptance checkpoint

**Recorded:** 2026-10-20.
**Formal owner:** [LM013 / #861](https://github.com/guillermomolina/protos/issues/861).
**Ratified contract:** [D209-C](../../decisions/tooling/D209_UNBOUND_DOCUMENT_LOCAL_LSP_INTELLIGENCE_AUTHORITY.md).
**Product source publication:** [`cb027faa4205ada04ffe9f875e5242f19067e5e8`](https://github.com/guillermomolina/protos/commit/cb027faa4205ada04ffe9f875e5242f19067e5e8), `0.3.389-SNAPSHOT`.
**Earlier publication evidence:** [LM013 product audit](LM013_389_DOCUMENT_LOCAL_LSP_PRODUCT_PUBLICATION_2026_10_20.md).
**Evidence class:** real VS Code UI checks **reported by the human maintainer** plus the maintainer's copied process/version command output; these checks were not run by the coordinating agent. **Partial real-editor acceptance; canonical bound-project positive-navigation checks still pending.**

## Runtime selection: first failed attempt vs validated Native instance

The real editor was attached to a Protos Dev Container (`protos-devcontainer`, user `vscode`). The devcontainer's published configuration pins extension `guillermomolina.protos@0.2.2` and a separate earlier released Protos Native runtime. This is the **repository pin**, not an independently captured installed VS Code extension identity.

The maintainer built/copied a local unpublished Native development distribution to:

```text
/home/vscode/.local/opt/protos-lm013/protos-0.3.389-SNAPSHOT-native-linux-x86_64/bin/protos
```

The exact executable returned `Protos 0.3.389-SNAPSHOT` on `--version`. The initial UI checks failed for Hover and subsequent capabilities, but process inspection showed that VS Code was still running **the old runtime**, `/opt/protos/0.3.312/libexec/protos-native language-server`. The maintainer then corrected invalid syntax in the remote VS Code Machine `settings.json` for `protos.runtime.executable` and reloaded the editor.

After reloading, the maintainer's `/proc/[0-9]*/cmdline` inspection explicitly showed:

```text
PID=6494 CMD=/home/vscode/.local/opt/protos-lm013/protos-0.3.389-SNAPSHOT-native-linux-x86_64/libexec/protos-native language-server
```

This confirms the tested VS Code session **was using the new 0.3.389-SNAPSHOT Native LSP**, rather than the released 0.3.312 server. It does **not** independently prove the binary's source-revision `SOURCE.txt`, checksum, or the exact installed extension build. Those identifiers can be added when observed.

## Human-observed acceptance matrix: loose/unbound Protos document

After confirmation of the 0.3.389 Native LSP process, the maintainer responded `todas las comprobaciones OK` to the four explicit manual real-editor checks. The checks were:

| VS Code check | Human result | Scope |
|---|---|---|
| Hover on the reference `value` in a Closure body | **PASS reported** | Displays `Proven binding` from the open document |
| Completion after typing `val` in the Closure | **PASS reported** | Suggests `value` with `Proven binding: Closure parameter` detail |
| Signature Help on a directly literal Closure call | **PASS reported** | Shows the literal Closure signature `(first, second)` |
| Go to Definition in a loose/unbound `.protos` document | **PASS reported** | No project-authorized navigation is returned |

These observations establish a bounded **real-editor loose-file positive/negative comparison against the new Native runtime**. They do not themselves establish a VS Code window with `workspaceFolders == null`; this extra condition is covered by the separately published JSON-RPC integration test, not independently claimed for the UI. They do not assert a separately inspected native References command, aliased URI, or unsaved-buffer behavior beyond the reported interactive completion.

The extension-side runtime authority is `protos.runtime.executable`; its startup command is `<configured executable> language-server`. No extension release, Protos GitHub Release, public Native distribution, or code change was needed for this local acceptance.

## Remaining gate before closing LM013

**Do not close LM013 yet.** In an actual canonical bound project with exact `protos.toml`, `protos.lock` and `protos.project` metadata, check that a proven Closure parameter reference still supports positive Go to Definition and Find References against the correctly bound source. Confirm real-editor local Hover/Completion/Signature Help still work there if not already reported. Record maintainer result before closing, preserving the D209-C separation between document-local services and ProjectBinding-gated navigation.

```text
LM013_PRODUCT_COMMIT=cb027faa4205ada04ffe9f875e5242f19067e5e8
LM013_IMPLEMENTATION_VERSION=0.3.389-SNAPSHOT
NATIVE_CLI_VERSION=OBSERVED_BY_MAINTAINER
NATIVE_LSP_PROCESS=VERIFIED_FROM_MAINTAINER_PROC_OUTPUT_PID_6494
RELEASE_PUBLISHED=NO
LOOSE_VSCODE_HOVER=HUMAN_PASS
LOOSE_VSCODE_COMPLETION=HUMAN_PASS
LOOSE_VSCODE_LITERAL_SIGNATURE_HELP=HUMAN_PASS
LOOSE_VSCODE_DEFINITION_NEGATIVE=HUMAN_PASS
CANONICAL_BOUND_PROJECT_NAVIGATION=NOT_YET_REPORTED
ACTUAL_INSTALLED_EXTENSION_VERSION=NOT_INDEPENDENTLY_CAPTURED
NATIVE_SOURCE_METADATA_SHA=NOT_INDEPENDENTLY_CAPTURED
LM013_ISSUE=OPEN_REVIEW
```
