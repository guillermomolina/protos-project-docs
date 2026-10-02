# DIST009-B1/B2 — exact candidate validation checkpoint

Status: **B1 PASS / B2 BLOCKED ON PACKAGED VS CODE VALIDATION ENVIRONMENT**

This durable, non-normative record captures the exact DIST009 candidate identity
established by B1 and the bounded B2 packaged VS Code validation result observed
on 2026-10-02.

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

## B2 packaged extension preparation

B2 operated from the current packaged extension repository state and used one
canonical VSIX for all attempted acceptance work:

```text
EXTENSION_REVISION=bef23f2b204dc784aaddf1ee327a6f36e52bebc7
PACKAGED_VSIX_SHA256=d60a43d17d32020c791e6bec4ce4917dadf3efea49b5e4ddfd6063976b152cf7
PROTOS_SOURCE_LOCK_SHA256=0c8814f6b969be8ccd4f535efe2d05ecdb3e5ff41f1d4316dd4158d5b8dcc939
```

The human executor reported all requested package-preparation gates PASS,
including dependency installation, the repository test gate, raw VSIX
packaging, VSIX validation, canonicalization, and reproducibility verification.

The same packaged VSIX remained byte-identical across the real-Run diagnostic
attempts, and the public Protos source lock remained byte-identical.

## B2 environment diagnosis

The first real desktop launch could not run because the development container
did not initially contain the desktop VS Code/Xvfb prerequisites. Those tools
were then installed only in the live container for diagnosis; this did not
change repository content.

Once desktop VS Code was available, Chromium initially aborted before the
workbench because the container did not permit the required namespace creation:

```text
Failed to move to new namespace: PID namespaces supported, Network namespace supported, but failed: errno = Operation not permitted
FATAL:content/browser/zygote_host/zygote_host_impl_linux.cc:207
```

A diagnostic launch with the Chromium sandbox disabled progressed far enough to
start the VS Code workbench and Extension Host. The harness package and packaged
Protos extension were installed, the candidate runtime remained accessible, and
no `--extensionDevelopmentPath` route was used.

However, the real-Run harness never produced its required `result.json`.
Additional ad-hoc launch variants did not establish a reliable canonical
packaged desktop acceptance path. Real Debug was therefore not run.

The resulting B2 classification is:

```text
DIST009_B2_STATUS=BLOCKED
BLOCKER=PACKAGED_VSCODE_DESKTOP_VALIDATION_ENVIRONMENT
PRODUCT_FAILURE_ESTABLISHED=NO

REAL_RUN=NOT_ADMITTED
REAL_RUN_RESULT=NOT_PRODUCED
REAL_DEBUG=NOT_EXECUTED
REAL_DEBUG=NOT_ADMITTED

EXTENSION_DEVELOPMENT_PATH_USED=NO
VSIX_UNCHANGED=YES
PUBLIC_LOCK_UNCHANGED=YES
WORKTREE_CLEAN=YES

PROTOS_REBUILT=NO
PUBLICATION_PERFORMED=NO
TAG_CREATED=NO
```

This checkpoint does not convert the missing B2 acceptance into a product
failure. The exact candidate and exact VSIX remain eligible for a later B2 retry
after the packaged desktop acceptance environment is made reproducible.

## Release routing

DIST009-C is not authorized while B2 remains unadmitted:

```text
DIST009_STATUS=BLOCKED
DIST009_C_READY=NO
DIST006_B2_PUBLISHED_NATIVE_PREREQUISITE=NOT_READY
```

The next bounded slice remains inside DIST009-B2 and owns only the validation
environment:

```text
NEXT_SLICE=DIST009-B2E
TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos-vscode-extension

GOAL=make the packaged VS Code desktop Run/Debug acceptance path reproducible
REUSE_EXACT_CANDIDATE=YES
REUSE_EXACT_VSIX=YES
REBUILD_PROTOS=NO
PUBLISH_RELEASE=NO
UPDATE_PUBLIC_EXTENSION_LOCK=NO
```

B2E must first establish one maintained, deterministic desktop-launch route for
the existing packaged harnesses. It may change only extension-repository
validation/environment machinery required for that route. It must not weaken
the Run/Debug assertions, substitute Extension Development Host execution, or
change Protos runtime semantics.

After B2E validation is green, DIST009-B2 reruns only the exact-candidate real
Run and real Debug gates needed to admit the frozen candidate. DIST009-C remains
a separate later publication slice.
