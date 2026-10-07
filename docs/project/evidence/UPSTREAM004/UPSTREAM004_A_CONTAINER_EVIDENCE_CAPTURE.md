# UPSTREAM004-A — Ephemeral container evidence capture

Status: **CAPTURE COMPLETED; ARCHIVE CONTENT PENDING EXTERNAL RETENTION**

This durable, non-normative record supports
`guillermomolina/protos#753`.

It records the exact evidence-capture result produced in the development
container before the temporary BUG013 minimal reproducer can disappear.

## Capture identity

```text
WORK_ITEM=UPSTREAM004-A
PARENT=UPSTREAM004 / guillermomolina/protos#753
CAPTURE_ROOT=/tmp/upstream004-capture
ARCHIVE=/tmp/upstream004-container-evidence.tar.gz
ARCHIVE_SIZE=12K
ARCHIVE_SHA256=ef610d44838774a54a2dc8ba9b2a379a9427cafd7640e0e0ad103e7194cce6b0
```

The archive was created directly from the live container state after the
independent minimal JVM/Native reproducer had already been exercised.

No Protos repository mutation, rebuild, or Native Image recompilation was
performed during this capture.

## Captured files

```text
CAPTURE_SUMMARY.txt
environment/graal-25.3.4.1.txt
environment/graal-25.5.5-dev.txt
environment/graal-dev-release.json
environment/host.txt
logs/bug013-repro-253-plain.log
logs/bug013-repro-constant.log
logs/bug013-repro-jvm-plain.log
logs/bug013-repro-local.log
logs/bug013-repro-loop.log
logs/bug013-repro-plain.log
logs/bug013-repro-yield.log
MANIFEST.sha256
maven-plane/25.5.5-snapshot-artifacts.txt
maven-plane/dependency-tree.txt
maven-plane/settings.txt
reproducer/pom.xml
reproducer/src/main/java/repro/Main.java
reproducer/src/main/java/repro/MiniLanguage.java
reproducer/src/main/java/repro/MiniPlainRoot.java
reproducer/src/main/java/repro/MiniRoot.java
```

## File manifest

```text
eb06a3928a26cd9d651ec1d870ee4de115d3d3cc3550c6b2ddd11b4e6049042d  CAPTURE_SUMMARY.txt
8a65b3331f569a96e1c7cd318b3c7f9b28706b6705012e290e985ae56f2b2379  environment/graal-25.3.4.1.txt
1c422c07118a3e9082195a7e1a9e74f3a9c64b375d70f5895d155159817d5682  environment/graal-25.5.5-dev.txt
c21e9f5df457a2b26a5a94963c0e2c5ed973675f1c2784881c9fcd23a0c566a1  environment/graal-dev-release.json
33daf20464d5bf8fd38b4c967cebf1c5bb44e2c41dbe095d76c7c5ecd3d7b13a  environment/host.txt
093bf8cb8fae7b956ba9778cc0db08894e8e46d329277e8345d2e60758e2969b  logs/bug013-repro-253-plain.log
de9685d9023cabfe17dc501876c88e0542dc2b9cacefb0253bf7c285adae7d78  logs/bug013-repro-constant.log
a2b7026b0a89623b3e84719ed2df5a4b34984ddf21a06da4a766183baf8c0ce3  logs/bug013-repro-jvm-plain.log
ae996f74bf0ca53646eb2a2adf0d338b070f8659642e35b535c54b9ceb7f13d7  logs/bug013-repro-local.log
180c80197c594d8e24dbb155ca7480a06b7413a2eb4c037eb67853d24835a35d  logs/bug013-repro-loop.log
8116e3cab72de4130b89b8ed70bb03e70bd825021c9c1f54339a221919c5a72a  logs/bug013-repro-plain.log
e8ff32f9d0bbab2987e22f93db243fefe0c55a3595ec531e2000fa2b904b76cf  logs/bug013-repro-yield.log
e377950fb5386d822d8fbb72bb61e720d36c5c992a1771fd0085a6fb54ded707  maven-plane/25.5.5-snapshot-artifacts.txt
4aae8712d59dc6a3188b4174b460aba85a4f7dd89a59977f8c7bedfbb1d0fe57  maven-plane/dependency-tree.txt
41d992ee55a832298fc09584c942f6628f05d0799f33e58b936c94eb861f010b  maven-plane/settings.txt
fb43982b0ae8cb817d5ba5b1504a808708ea62cd1ea368bf40f1d25bfce1f391  reproducer/pom.xml
8796bf44b7083f3e6bac8090d6e97a6150233099aefb17fa9157abf4c236b4fc  reproducer/src/main/java/repro/Main.java
a89c2b17bc6ee7d5331186c641de476cd531653ebd0642acbd877cd676ae198e  reproducer/src/main/java/repro/MiniLanguage.java
ad81a2d7b29b6fe87d32375efd9e8add4d83ea99b9ab0a536d6ed70c89813422  reproducer/src/main/java/repro/MiniPlainRoot.java
b911f3f0879a17bd18bc2b89a90e3d0b227ee8cb45e8c1cb30c00787bd6a0386  reproducer/src/main/java/repro/MiniRoot.java
```

The paths above are written relative to the capture root. The original
`MANIFEST.sha256` contained their absolute `/tmp/upstream004-capture/...`
paths.

## Important retained evidence

The capture contains both halves of the minimum same-snapshot control:

```text
logs/bug013-repro-jvm-plain.log
  JVM 25.5.5-SNAPSHOT control
  MiniPlainRootGen Tier 2 opt done
  REPRO_RESULT=1

logs/bug013-repro-plain.log
  Native Image 25.5.5-SNAPSHOT control
  MiniPlainRootGen Tier 2 opt failed
  FrameWithoutBoxing should not be materialized
  REPRO_RESULT=1
```

It also retains the broader Native controls:

```text
constant
local
loop
yield
```

and the independent GraalVM 25.3.4.1 Native `plain` failure.

## Environment and dependency-plane capture

The capture additionally retains:

- host OS, architecture and CPU identity;
- GraalVM CE 25.5.5-dev Java, `java -Xinternalversion`,
  `native-image --version`, release file and Maven identity;
- GraalVM 25.3.4.1 identity;
- the developer-build release metadata;
- exact `25.5.5-SNAPSHOT` Maven artifact paths and SHA-256 values;
- Maven dependency tree;
- the temporary settings file identity and non-secret repository structure.

This is sufficient to reconstruct the exact runtime/dependency plane without
depending on conversation history.

## Current retention boundary

The tarball itself still exists only in the user's development container unless
and until it is uploaded or otherwise stored durably.

Therefore:

```text
EPHEMERAL_REPRODUCER_CAPTURE=COMPLETE
ARCHIVE_SHA256_RETAINED=YES
FILE_MANIFEST_RETAINED=YES
ARCHIVE_EXTERNAL_RETENTION=PENDING
SOURCE_CONTENT_REVIEW=PENDING
LOG_CONTENT_REVIEW=PENDING
UPSTREAM_DRAFT=PENDING
EXTERNAL_ISSUE_OPENED=NO
```

Do not delete the current container archive until an externally retained copy
has been verified against:

```text
ef610d44838774a54a2dc8ba9b2a379a9427cafd7640e0e0ad103e7194cce6b0
```

## Next slice

The next UPSTREAM004 slice is review of the archived source/log contents and
construction of a compact upstream-facing reproducer package and issue draft.

That slice must use the captured files as authority. It must not reconstruct
source, commands, or logs from memory when an exact captured artifact exists.
