# DIST014 — Protos devcontainer 0.3.237 / VS Code extension 0.2.2 closure evidence

Date: 2026-10-06

This snapshot records implementation, real Dev Container acceptance, and the
project owner's explicit closure direction for `guillermomolina/protos#810`
(DIST014). It is durable non-normative project evidence.

## Stable identities

```text
FORMAL_WORK_ITEM=DIST014
GITHUB_ISSUE=guillermomolina/protos#810
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-devcontainer
DEVCONTAINER_REVISION=35458e11e08d8f67a2f34eb1adb966588bd214b4
DEVCONTAINER_COMMIT=Update protos and vscode extension

PROTOS_VERSION=0.3.237
PROTOS_RELEASE_TAG=v0.3.237
PROTOS_REVISION=f627734c8feaf3c26c4a32903b6ea0e25f4a9956
PROTOS_ASSET=protos-0.3.237-native-linux-x86_64.zip
PROTOS_ASSET_SHA256=510537c7c47a323ed05a5ea8e4e46bc3cd63c761a58e68cb0639548feb8cc46c

VSCODE_EXTENSION_ID=guillermomolina.protos
VSCODE_EXTENSION_VERSION=0.2.2
VSCODE_EXTENSION_REVISION=c6f8d8cd2979d355adbd3cf7b64bf3475fbf6f9a
```

The published devcontainer commit changes exactly:

```text
.devcontainer/devcontainer.json
README.md
versions.json
```

`versions.json` pins the exact Protos release tag, Native asset name and
SHA-256, and the official extension version. The Dev Container configuration
pins `guillermomolina.protos@0.2.2`.

## Real Dev Container acceptance

The maintainer rebuilt and reopened the actual VS Code Dev Container. Observed
runtime/extension identity included:

```text
LAUNCHER=/usr/local/bin/protos
NATIVE_RUNTIME=/opt/protos/0.3.237/bin/protos
NATIVE_RUNTIME_SELECTED=PASS
VSCODE_EXTENSION_INSTALL=guillermomolina.protos-0.2.2
EXTENSION_PINNED=PASS
MARKETPLACE_EXTENSION_INSTALL=PASS
```

Direct runtime smoke completed and printed:

```text
DIST014_RUN_OK
```

The Truffle interpreter-only warning observed during Native execution is the
expected PLAT045 fallback-runtime behavior and was not classified as a DIST014
failure.

Real editor acceptance then passed:

```text
RUN_CURRENT_FILE=PASS
RUN_OUTPUT=41,DIST014_RUN_OK
LANGUAGE_SERVER=PASS
LANGUAGE_SERVER_EXECUTABLE=/opt/protos/0.3.237/libexec/protos-native
REAL_DEBUG=PASS
FORMAT_DOCUMENT=PASS
FORMAT_ON_SAVE=PASS
```

The debugger proof included breakpoint stop, visible stack/local value
`x = 41`, Step Over to the next line after disabling the breakpoint, Continue,
and clean termination. Format Document and format-on-save both produced the
expected canonical source through the installed Marketplace extension 0.2.2.

No Protos runtime was built from source inside the devcontainer.

## Known version-output residual and owner closure direction

The immutable Protos 0.3.237 Native artifact reports:

```text
protos --version
Protos development
```

rather than `Protos 0.3.237`. The product identity itself was independently
proven by the exact pinned release tag, asset name, SHA-256, installed path and
successful runtime/editor acceptance.

This defect became BUG019/#811 and is independent of the devcontainer refresh.
The project owner explicitly directed that it **does not block DIST014** and
subsequently directed DIST014 closure.

The published devcontainer README still contains the historical expectation:

```text
Protos 0.3.237
```

for that command. Because the 0.3.237 release is immutable, this snapshot does
not misreport that example as currently true. It is retained here as a known
non-blocking documentation erratum under the owner's closure direction.

BUG019 was later technically repaired on Protos main at
`7b9e629bddce9d501a2773a9f3a3973c8de37885` for future Native distributions;
that later repair does not mutate the already-published 0.3.237 asset consumed
by DIST014.

## Closure result

```text
DEVCONTAINER_PUBLICATION=PASS
CONTAINER_BUILD=PASS
PROTOS_ASSET_PIN=PASS
PROTOS_ASSET_SHA256_PIN=PASS
NATIVE_RUNTIME_SELECTED=PASS
NO_RUNTIME_BUILD_FROM_SOURCE=YES
EXTENSION_PINNED=PASS
MARKETPLACE_EXTENSION_INSTALL=PASS
RUN_CURRENT_FILE=PASS
LANGUAGE_SERVER=PASS
REAL_DEBUG=PASS
FORMAT_DOCUMENT=PASS
FORMAT_ON_SAVE=PASS

PROTOS_VERSION_OUTPUT=KNOWN_UPSTREAM_DEFECT_NONBLOCKING
README_VERSION_OUTPUT_EXAMPLE=KNOWN_ERRATUM_NONBLOCKING
OWNER_CLOSURE_DIRECTION=YES
NEXT_DIST014_TECHNICAL_SLICE=NONE
DIST014_CLOSE_READY=YES
```

AI assistance: this evidence record was drafted with ChatGPT from the live
DIST014 Issue, the maintainer's real Dev Container acceptance reports, exact
inspection of `guillermomolina/protos-devcontainer` revision
`35458e11e08d8f67a2f34eb1adb966588bd214b4`, and the linked BUG019 evidence.
