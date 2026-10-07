# DIST013-A2 — Reselection after release-tooling repair

Issue: `guillermomolina/protos#805`  
Blocking: LM011 / `guillermomolina/protos#670`

## Decision

The project owner explicitly approved replacing the original DIST013 release selection with the following exact baseline:

```text
RELEASE_BASELINE_REVISION=72c0515a707b876697c5a3afb96b0672264eac61
RELEASE_BASELINE_VERSION=0.3.237-SNAPSHOT
RELEASE_VERSION=0.3.237
RELEASE_TAG=v0.3.237
SPECIFICATION_REVISION=0.1.444
```

This approval authorizes candidate selection and DIST013-B preparation only. It does **not** authorize creating or pushing the tag, creating a GitHub Release, or publishing assets.

## Why the previous selection was superseded

The original approved selection was:

```text
22e3aa9466d172ac816b3a74b4e3ca9db155a92e
0.3.236-SNAPSHOT -> 0.3.236 / v0.3.236
```

DIST013-B successfully built the Native Image from that candidate but failed during Native archive construction with:

```text
native distribution build failed: internal checksum coverage does not match archive files
```

The root cause was a release-tooling defect: both Native and portable distribution builders excluded every file whose basename was `SHA256SUMS` from the distribution-root checksum manifest. The retained official TOML corpus legitimately contains its own nested `SHA256SUMS`, so that file was archived but not covered by the root manifest.

The repair was validated and published as:

```text
72c0515a707b876697c5a3afb96b0672264eac61
DIST013-B: fix nested distribution checksum coverage
```

Before that repair landed, concurrent commit:

```text
21437984ceb311ff89d19563ea97da342e37f8da
TEST009-V2: stop single-root diagnostics at BGV capture
```

advanced the Maven development line to `0.3.237-SNAPSHOT`. Therefore the repaired exact baseline cannot truthfully produce a `0.3.236` release-only candidate.

## Verified preconditions

At reselection time:

- LM011-D1 revision `75cfed853c7eef8c9d3dfb0c6f2507fdf8864ecb` is an ancestor of `72c0515a707b876697c5a3afb96b0672264eac61`;
- the Native builder contains the nested-`SHA256SUMS` coverage fix;
- the portable builder contains the same fix;
- the Maven development version is `0.3.237-SNAPSHOT`;
- the specification revision remains `0.1.444`;
- the checksum repair passed its focal regression and the integrated `make test` gate before publication to main.

## Execution boundary

DIST013-B may now materialize an exact detached `0.3.237` candidate from the selected baseline and run the maintained Native/JVM release admissions under the previously approved detached-candidate exception.

The following remain false until a later explicit DIST013-C authorization bound to the exact candidate and release-manifest digest:

```text
RELEASE_PUBLICATION_AUTHORIZED=NO
TAG_CREATED=NO
GITHUB_RELEASE_CREATED=NO
ASSETS_PUBLISHED=NO
```
