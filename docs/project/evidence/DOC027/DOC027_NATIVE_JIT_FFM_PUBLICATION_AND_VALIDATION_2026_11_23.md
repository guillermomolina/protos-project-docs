# DOC027 — Revision-bound documentation publication and validation evidence

**Date:** 2026-11-23  
**Issue:** [DOC027 #1157](https://github.com/guillermomolina/protos/issues/1157)  
**PROTOS_REVISION:** `89028494587d4d22309a5c048e37da3518170b18`  
**Product commit:** https://github.com/guillermomolina/protos/commit/89028494587d4d22309a5c048e37da3518170b18  
**Owning durable work record:** [DOC027 documentation completion](../../work/DOC027/DOC027_NATIVE_JIT_INTEGRATED_FFM_DOCUMENTATION_COMPLETION_2026_11_23.md)

## Publication inspection

The published product commit `89028494587d4d22309a5c048e37da3518170b18`, compared with `9c56aba2c8b84a9c44440cb361db85b53fcf30b4`, changes exactly one successor commit and the following **14 paths**:

```text
README.md
docs/assets/branding/protos-logo.png
docs/guide/00-try-protos.md
docs/guide/interop/README.md
docs/guide/interop/ffm-provider.md
docs/guide/library/README.md
docs/guide/library/ffi.md
docs/guide/library/interop.md
docs/guide/library/regex.md
docs/guide/library/sqlite.md
docs/guide/tools/cli.md
protos/examples/ffi/README.md
protos/tutorials/README.md
providers/ffm/README.md
```

The 13 Markdown files reconcile (1) guest Tier-2 JIT requirements in installed Native versus portable JVM; (2) external portable-JVM FFM JAR loading versus integrated Native FFM; (3) default-off `--allow-library` authorization, lazy import, trusted native execution and the finite Native signature set; and (4) platform-specific SQLite/FFM example and tutorial invocations. No performance equivalence, unrestricted C ABI, dynamic Java plugin loading in Native, or Protos guest-capability sandboxing is claimed.

The single binary addition restores the root README's linked `docs/assets/branding/protos-logo.png`. GitHub lists its blob SHA as `48862ed6638ec851349211b28b25aec4bb45f0a6`, matching the original referenced in [the DOC027 issue comment](https://github.com/guillermomolina/protos/issues/1157#issuecomment-7097702045). The README's `<img src="docs/assets/branding/protos-logo.png">` link points to that exact tracked asset at the published revision. No direct browser rendering or pixel-level visual check is claimed.

## Source-backed contract and acceptance

- [`scripts/native-test.sh`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/scripts/native-test.sh) rejects fallback markers and requires successful Tier-2 compilation from `ProtosBytecodeRootNodeGen` and `ProtosBuiltinCallRootNode`.
- [`dist/validate_native.py`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/dist/validate_native.py) independently rejects fallback execution or missing successful Tier-2 coverage for either root family.
- [`docs/guide/interop/ffm-provider.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/docs/guide/interop/ffm-provider.md#native-image-integrated-provider) documents the integrated Native provider, default-off explicit authorization, lazy OS library loading, no JAR or dynamic Java plugins, and the finite signature set.
- [`spec/semantics/FFI.md`](https://github.com/guillermomolina/protos/blob/89028494587d4d22309a5c048e37da3518170b18/spec/semantics/FFI.md) remains the unmodified normative public FFI authority, including the `TRUSTED_IN_PROCESS` boundary.

## Validation provenance

| Gate | Result and source |
| --- | --- |
| Protos SHA and changed-file inventory | **PASS**, independently retrieved from published GitHub commit/compare |
| README logo path and original blob identity | **PASS**, independently retrieved from GitHub contents and published README source |
| Targeted published-Markdown stale-assertion scan | **PASS**, inspected 13 current published Markdown contents; no checked obsolete assertions found |
| `git diff --check` | **CLEAN, owner-reported** after publication |
| All local tests | **PASS, owner-reported**; detailed command inventory not provided |
| Full Markdown parser / anchor and external link crawl | **NOT CLAIMED** |
| Direct browser image-render verification | **NOT CLAIMED**; tracked file and README target independently verified |
| CI, Native rebuild, independent tests | **NOT CLAIMED** |

This is a documentation-and-asset-only correction. It neither reopens implementation decisions nor changes `spec/`, Java implementation, executable Protos source, release/version metadata, or website-owned files.

```text
FORMAL_IDENTIFIER=DOC027
PROTOS_REVISION=89028494587d4d22309a5c048e37da3518170b18
PRODUCT_COMMIT=PASS_GITHUB_VERIFIED
PRODUCT_PATHS_CHANGED=14
PRODUCT_MARKDOWN_PATHS_CHANGED=13
ASSET_RESTORED=docs/assets/branding/protos-logo.png
ASSET_BLOB_SHA=48862ed6638ec851349211b28b25aec4bb45f0a6
GIT_DIFF_CHECK=CLEAN_OWNER_REPORTED
ALL_LOCAL_TESTS=PASS_OWNER_REPORTED
DOC_CONSISTENCY_CHECK=TARGETED_PUBLISHED_SOURCE_INSPECTION
FULL_LINK_CHECK=NOT_CLAIMED
BROWSER_RENDER_CHECK=NOT_CLAIMED
```
