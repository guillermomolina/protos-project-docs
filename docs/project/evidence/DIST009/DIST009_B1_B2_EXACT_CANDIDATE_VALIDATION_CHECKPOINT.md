# DIST009-B1/B2 — exact candidate validation checkpoint

Status: **B1 PASS / B2 PASS / DIST009-C PASS / DIST009 COMPLETE**

This durable, non-normative record captures the exact DIST009 candidate identity
established by B1 and the completed B2 packaged VS Code Run/Debug validation
observed on 2026-10-02.

## Ownership

```text
DATE=2026-10-02
WORK_ITEM=DIST009
GITHUB_ISSUE=guillermomolina/protos#743
CONSUMER=DIST006-B2/guillermomolina/protos#734
B1_REPOSITORY=guillermomolina/protos
B2_REPOSITORY=guillermomolina/protos-vscode-extension
```

DIST009 owns publication of the frozen JVM_PLUS_NATIVE prerelease candidate.
DIST006-B2 remains the later consumer that updates the public extension runtime
lock and performs final installed/public Run/Debug acceptance after publication.

## B1 exact candidate

The human executor reported DIST009-B1 PASS and retained this exact release
identity:

```text
DIST009_B1_STATUS=PASS

BASELINE_REVISION=d70d4438170493b61c9da70781130e2a724ce4a1
BASELINE_VERSION=0.3.139-SNAPSHOT
RELEASE_VERSION=0.3.139
RELEASE_TAG=v0.3.139
SPECIFICATION_REVISION=0.1.436

BUG012_REPAIR_IN_ANCESTRY=YES

CANDIDATE_SOURCE_REVISION=3895206897ddac795dfebd49709ca97f8d0908b1
CANDIDATE_PARENT=d70d4438170493b61c9da70781130e2a724ce4a1
CANDIDATE_DETACHED=YES
```

The exact Native candidate consumed by B2 is:

```text
NATIVE_ASSET=protos-0.3.139-native-linux-x86_64.zip
NATIVE_SHA256=8e87dfc410c6a49f194786f205ba434f0b5b60865873e99e18ded8aba66c46da
RUNTIME_VERSION=Protos 0.3.139
```

B1 did not publish a tag, GitHub Release, or assets.

## B2 packaged extension authority

B2 retained one canonical packaged VSIX for Run and Debug:

```text
EXTENSION_REVISION=bef23f2b204dc784aaddf1ee327a6f36e52bebc7
PACKAGED_VSIX_SHA256=d60a43d17d32020c791e6bec4ce4917dadf3efea49b5e4ddfd6063976b152cf7
PROTOS_SOURCE_LOCK_SHA256=0c8814f6b969be8ccd4f535efe2d05ecdb3e5ff41f1d4316dd4158d5b8dcc939
B2E_HARNESS_IMPLEMENTATION_REVISION=4b0659b4061a5d0e9a7de46cf30920e7b2004af9
```

The package-preparation gates had already passed:

```text
NPM_CI=PASS
PACKAGE_TEST_SUITE=PASS
RAW_VSIX_PACKAGING=PASS
VSIX_VALIDATION=PASS
VSIX_CANONICALIZATION=PASS
VSIX_REPRODUCIBILITY=PASS
```

No product source change, Protos rebuild, public lock update, tag, Release, or
asset publication occurred during B2.

## B2E packaged VS Code environment

The earlier desktop-in-container path was not retained. Running a second full
Electron/VS Code Desktop instance inside the devcontainer required extra desktop
packages/Xvfb and encountered Chromium namespace/sandbox constraints.

The maintained B2E route instead used the ordinary VS Code Dev Containers
architecture already active for development:

```text
HOST_VSCODE_DESKTOP
  -> DEV_CONTAINER_VSCODE_SERVER
  -> REMOTE_EXTENSION_HOST
  -> PACKAGED_EXTENSION_VSIX
```

The active Remote Extension Host was addressed through the VS Code Server
remote CLI and its live `/tmp/vscode-ipc-*.sock` IPC endpoint. No
`--extensionDevelopmentPath` route was used.

The acceptance harness changes were published in
`guillermomolina/protos-vscode-extension` at exact revision
`4b0659b4061a5d0e9a7de46cf30920e7b2004af9`.

The acceptance harnesses were made safe for this interactive route:

- descriptor-file configuration is supported when CI environment variables are
  absent;
- interactive descriptor mode explicitly sets `quitWhenFinished=false`, so the
  harness cannot execute `workbench.action.quit`;
- the Debug harness treats a `vscode-remote://...` frame source as the same
  workspace-host path when the URI path equals the exact fixture path;
- the line-2 source breakpoint is removed before `next`;
- the harness observes and waits for the DAP acknowledgement of
  `setBreakpoints([])` before sending `next`;
- the harness then waits for a subsequent DAP `stopped` event before reading the
  post-step stack frame.

That breakpoint-removal ordering matches the already retained prepublication
acceptance finding in `DIST009_PREPUBLICATION_VSCODE_ACCEPTANCE.md`: leaving
the line-2 breakpoint installed makes `next` resume and immediately stop on
the same breakpoint.

A direct DAP discriminator against the exact 0.3.139 Native candidate reproduced
that expected harness effect while the breakpoint remained installed:

```text
FIRST_STOP_REASON=breakpoint
FIRST_STOP_LINE=2
SECOND_STOP_REASON=breakpoint
SECOND_STOP_LINE=2
DIRECT_DAP_NEXT_2_TO_3=FAIL
```

This was not classified as a Protos product defect because it exactly matches
the pre-existing debugger/harness interaction already documented before
DIST009. Removing the breakpoint before `next` restores the intended step
surface.

## B2 exact-candidate Real Run

The exact packaged VSIX and exact Native 0.3.139 candidate passed real packaged
Run through the Dev Containers Remote Extension Host:

```text
REAL_RUN=PASS
PACKAGED_EXTENSION_FOUND=YES
PACKAGED_EXTENSION_ACTIVATED=YES
PACKAGED_RUN_COMMAND=PASS
REAL_PROTOS_RUNTIME_INVOKED=YES
RUN_TASK_EXIT_CODE=0
EXTENSION_DEVELOPMENT_PATH_USED=NO
```

The Run fixture was:

```protos
print("VS_CODE_RUN")
```

The product VSIX remained byte-identical at
`d60a43d17d32020c791e6bec4ce4917dadf3efea49b5e4ddfd6063976b152cf7`.

## B2 exact-candidate Real Debug

The final exact-candidate packaged Real Debug result was:

```text
LM009_I3C_REAL_DEBUG=PASS
PACKAGED_EXTENSION_FOUND=YES
PACKAGED_EXTENSION_ACTIVATED=YES
SOURCE_BREAKPOINT=PASS
STOP_LOCATION=PASS
STACK_FRAMES=PASS
ACTIVATION_LOCAL_SCOPE=PASS
REPRESENTATIVE_SCALAR_VALUE=PASS
STEP_NEXT=PASS
CONTINUE=PASS
CLEAN_TERMINATION=PASS
EXTENSION_DEVELOPMENT_PATH_USED=NO
REAL_DEBUG_VALIDATE_RC=0
```

Observed debugger state:

```text
STATUS=pass
BREAKPOINT_LINE=2
STACK_FRAMES_OBSERVED=1
OBSERVED_TOP_FRAME_LINE=2
SCOPES_OBSERVED=1
LOCALS_OBSERVED=YES
X_VALUE=41
OBSERVED_NEXT_FRAME_LINE=3
NEXT_COMPLETED=YES
CONTINUE_COMPLETED=YES
TERMINATED_CLEANLY=YES
REMOTE_SESSION_SOCKET=AVAILABLE
```

The Debug fixture was:

```protos
x: 41
print(x)
print("done")
```

The same canonical product VSIX remained unchanged after Debug:

```text
PACKAGED_VSIX_SHA256=d60a43d17d32020c791e6bec4ce4917dadf3efea49b5e4ddfd6063976b152cf7
RUNTIME_VERSION=Protos 0.3.139
```

## Final B2 classification

```text
DIST009_B2_STATUS=PASS
REAL_RUN=PASS
REAL_DEBUG=PASS

SOURCE_BREAKPOINT=PASS
STOP_LOCATION=PASS
STACK_FRAMES=PASS
ACTIVATION_LOCAL_SCOPE=PASS
REPRESENTATIVE_SCALAR_VALUE=PASS
STEP_NEXT=PASS
CONTINUE=PASS
CLEAN_TERMINATION=PASS

EXTENSION_DEVELOPMENT_PATH_USED=NO
VSIX_UNCHANGED=YES
PUBLIC_LOCK_UNCHANGED=YES
PROTOS_REBUILT=NO
PUBLICATION_PERFORMED=NO
TAG_CREATED=NO
```

The earlier `PACKAGED_VSCODE_DESKTOP_VALIDATION_ENVIRONMENT` blocker is
resolved. No Protos or packaged-extension functional failure remains in the B2
gate.

## Release routing

DIST009-C has published and independently verified the frozen candidate:

```text
DIST009_B1_STATUS=PASS
DIST009_B2_STATUS=PASS
DIST009_C_STATUS=PASS
DIST009_STATUS=COMPLETE

RELEASE_TAG=v0.3.139
PUBLIC_TAG_TARGET=3895206897ddac795dfebd49709ca97f8d0908b1
RELEASE_MANIFEST_SHA256=aeee10ec0abb8d9064068d67a1a6c000f03b7bce47ef04bd0d9e6e94c0d5fdc6

PUBLIC_TAG_IDENTITY=PASS
GITHUB_PRERELEASE_METADATA=PASS
PUBLISHED_ASSET_SET=PASS
PUBLISHED_ASSET_DIGESTS=PASS
POST_PUBLICATION_VERIFICATION=PASS

DIST006_B2_PUBLISHED_NATIVE_PREREQUISITE=READY
```

Durable publication/closure evidence:

```text
DIST009_C_RECORD=docs/project/evidence/DIST009/DIST009_C_RELEASE_PUBLICATION_AND_CLOSURE.md
DIST009_C_RECORD_INITIAL_REVISION=de79ab3f683fb5f64316a44b7a8f12f82ab7ee18
```

The next executable unit is DIST006-B2 in
`guillermomolina/protos-vscode-extension`: update the exact public runtime lock
to v0.3.139 and perform final installed/public Run/Debug acceptance.
