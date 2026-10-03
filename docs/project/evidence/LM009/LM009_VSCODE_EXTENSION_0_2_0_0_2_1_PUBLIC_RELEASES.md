# LM009 — VS Code extension 0.2.0 / 0.2.1 public-release evidence

Status: **RETAINED POST-CLOSURE EVIDENCE**

Evidence date: **2026-10-03**

Related Protos work:

- LM009 / `guillermomolina/protos#288`
- LM009-I / `guillermomolina/protos#474`
- D128 / `guillermomolina/protos#500`
- DIST006-B / `guillermomolina/protos#734`

Extension repository:

`guillermomolina/protos-vscode-extension`

Nature: non-normative editor packaging, release, distribution and external-runtime
consumer evidence.

## Purpose

Retain the first explicit post-LM009 public release chain for the standalone Protos
VS Code extension after its repository split.

The evidence confirms that the extension's independent SemVer authority is being
used as ratified by LM009-I4, that canonical GitHub Release artifacts remain the
release authority, and that Marketplace publication consumes the exact accepted
canonical VSIX rather than rebuilding it.

No Protos language, Standard Library, runtime, DAP or LSP semantic contract is
changed by this record.

## Release 0.2.0

The standalone extension first published its materially current editor product as
a distinct `0.2.0` release instead of continuing to reuse the historical
`0.1.0` identity.

```text
EXTENSION_VERSION=0.2.0
EXTENSION_COMMIT=f6aa6178c61ee13eb8bad2b53c5a0c4f2986ab70
EXTENSION_TAG=v0.2.0
GITHUB_RELEASE_ID=402305543
GITHUB_RELEASE_STATUS=PUBLISHED
VSIX_ASSET=protos-vscode.vsix
VSIX_SHA256=6ed56f73a5bfed9029c924dc5714fe8dfc69f7e8cdf32c520622a0f1cc17f23e
MARKETPLACE_PUBLICATION=PUBLISHED
PACKAGE_TOOL_VSCE=3.9.2
```

The accepted product surface included the standalone extension's current:

- Run Current File integration;
- real DAP/debugger integration;
- static language-server client integration;
- VS Code Remote workspace-extension behavior;
- external Protos launcher/runtime boundary;
- deterministic canonical VSIX packaging;
- installed-VSIX clean-install, real Run and real Debug acceptance;
- release-version guard preventing a materially changed distributed artifact from
  silently retaining an already-published extension version.

The final post-commit repository authority passed:

```text
MAKE_TEST=PASS
CLEAN_INSTALL=PASS
REAL_RUN=PASS
REAL_DEBUG=PASS
EXTENSION_DEVELOPMENT_PATH_USED=NO
```

The GitHub Release and Marketplace publication used the same canonical VSIX
identity above.

## Release 0.2.1

The immediate maintenance release upgraded the official VS Code packaging
toolchain while preserving the established extension/runtime boundary and
installed-VSIX behavior.

```text
EXTENSION_VERSION=0.2.1
EXTENSION_COMMIT=6332f34f4a581d91771bae1a6b173b6e796ddffe
EXTENSION_TAG=v0.2.1
GITHUB_RELEASE_ID=402328600
GITHUB_RELEASE_STATUS=PUBLISHED
VSIX_ASSET=protos-vscode.vsix
VSIX_SHA256=ae87201434897d85a689bd4301202fa695a900c81c8b65f7036a6008d9d12f77
MARKETPLACE_PUBLICATION=PUBLISHED

NODE_VERSION=22.23.2
NPM_VERSION=10.9.8
PACKAGE_TOOL_VSCE=4.0.0
```

The toolchain transition was:

```text
EXTENSION_VERSION: 0.2.0 -> 0.2.1
@vscode/vsce:      3.9.2 -> 4.0.0
NODE_BASELINE:     >=22 SATISFIED
NPM_CI:            PASS
```

The package-lock transition reduced the resolved package-entry count from 354 to
200 while the repository and installed-VSIX authority remained green.

Final post-commit acceptance:

```text
MAKE_TEST=PASS
CLEAN_INSTALL=PASS
REAL_RUN=PASS
REAL_DEBUG=PASS
EXTENSION_DEVELOPMENT_PATH_USED=NO
```

Again, the GitHub Release and Marketplace mirror consumed the same canonical VSIX
that passed the post-commit acceptance; there was no post-acceptance rebuild.

## External Protos runtime authority

Both extension releases retained the published Protos Native runtime selected by
DIST006-B2 / DIST009 as their installed acceptance baseline:

```text
PROTOS_RELEASE_TAG=v0.3.139
PROTOS_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
PROTOS_NATIVE_ASSET=protos-0.3.139-native-linux-x86_64.zip
PROTOS_NATIVE_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
```

This preserves the ratified thin-client separation:

```text
VS Code extension version
    !=
Protos runtime version
```

The extension does not bundle a private Protos runtime. Run, Debug and
language-server startup remain delegated through the external
`protos.runtime.executable` authority.

## Distribution-policy confirmation

The observed public release sequence matches LM009-I4:

```text
accepted source commit
        |
        v
canonical VSIX + SHA-256
        |
        v
explicit extension tag
        |
        v
GitHub Release
        |
        v
Marketplace mirror of the same VSIX
```

Therefore:

```text
PUBLIC_RELEASE_CANONICAL_PATH=GITHUB_RELEASE
EXTENSION_VERSION_POLICY=INDEPENDENT_SEMVER
RUNTIME_VERSION_POLICY=SEPARATE
MARKETPLACE_ROLE=DOWNSTREAM_MIRROR
MARKETPLACE_REBUILD=NO
CANONICAL_VSIX_ATTACHED_TO_RELEASE=YES
GENERATED_VSIX_COMMITTED=NO
PROTOS_RUNTIME_IN_VSIX=NO
```

## Conclusions

The standalone VS Code extension has now exercised the LM009 packaging and
distribution contracts through two immutable public release identities.

The evidence strengthens, but does not reinterpret, the already-closed LM009,
LM009-I, D128 or DIST006-B work:

- D128's canonical-artifact model remains effective across a packaging-tool
  upgrade;
- LM009-I4's independent extension SemVer and GitHub-Release-first distribution
  model is exercised in practice;
- Marketplace publication remains a mirror of the accepted canonical artifact;
- DIST006-B's published Protos v0.3.139 Native runtime remains a working external
  consumer boundary for installed extension Run and Debug acceptance;
- no Protos semantic authority migrated into the extension.
