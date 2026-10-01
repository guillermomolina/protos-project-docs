# UPSTREAM004 — Native Bytecode DSL FrameWithoutBoxing report-preparation checkpoint

Status: **READY — EPHEMERAL REPRODUCER CAPTURE REQUIRED BEFORE EXTERNAL SUBMISSION**

This durable, non-normative checkpoint supports
`guillermomolina/protos#753`.

It records the facts already established before the temporary container
reproducer is copied into durable evidence. It is intentionally not an
`oracle/graal` issue draft yet.

## Protos coordination identity

```text
WORK_ITEM=UPSTREAM004
GITHUB_ISSUE=guillermomolina/protos#753
SOURCE_DEFECT=BUG013 / guillermomolina/protos#749
BLOCKED_RELEASE=DIST009 / guillermomolina/protos#743
HARDENED_NATIVE_GATE_REVISION=7770aa135bb65c7219de5dbe9e2b3ac22cdaf31a
HARDENED_NATIVE_GATE_PATH=build/native/test-native.sh
```

BUG013-C durable isolation evidence is:

```text
docs/project/evidence/BUG013/BUG013_C_UPSTREAM_NATIVE_RUNTIME_COMPILATION_ISOLATION.md
```

## External report target

The prospective external target is `oracle/graal`.

Current upstream contribution guidance was re-read during allocation of
UPSTREAM004:

- `oracle/graal/CONTRIBUTING.md` states that reproducible bugs should be filed
  as GitHub issues and asks for the affected GraalVM version, platform details,
  reproduction steps, and a minimal example.
- `oracle/graal/.github/ISSUE_TEMPLATE/2_issues_truffle.md` asks for exact
  GraalVM/JDK/OS/architecture identity, `java -Xinternalversion`, confirmation
  against the latest snapshot, a reproducer, build/run steps, expected behavior,
  and logs/stack traces.
- `oracle/graal/.github/ISSUE_TEMPLATE/1_1_native_image_run_time_bug_report.yml`
  is the Native Image runtime alternative and additionally asks for the exact run
  command and a Native Image bundle/reproduction material.
- `oracle/graal/CODING_ASSISTANTS.md`, blob
  `c4f374a790f636dbe6d801e03ae9c5ad2983fd77`, explicitly applies to issues as
  well as code contributions. AI assistance is permitted; the human contributor
  remains responsible for understanding and defending the submission; explicit
  attribution to a model/tool is optional.

No external issue has been opened.

## Established minimum reproducer behavior

The standalone reproducer is independent from Protos.

The smallest observed root is generated with:

```text
enableYield=false
no custom operations
no guest locals
no guest loops
no Protos classes
result = constant 1
```

The generated root still routes its frame through the Bytecode DSL generated
interpreter boundary:

```java
state = bc.continueAt(this, frame, state);
```

### Same-snapshot JVM control

With the Graal/Truffle `25.5.5-SNAPSHOT` plane:

```text
JVM_PLAIN_STATUS=0
MiniPlainRootGen ... |Tier 2| ... opt done
MiniPlainRootGen ... |Tier 2| ... opt done
REPRO_RESULT=1
```

### Same-snapshot Native result

With the same `25.5.5-SNAPSHOT` API/runtime/processor plane inside Native
Image:

```text
STATUS=0
MiniPlainRootGen ... |Tier 2| ... opt failed
FrameWithoutBoxing should not be materialized
REPRO_RESULT=1
```

The status is zero because `CompilationFailureAction=Print` reports the
compiler bailout and allows interpreted execution to continue.

The earlier `enableYield=true` constant root fails the same way. Native
`local`, `loop`, and `yield` controls also fail, so none of those features
is required to reproduce the defect.

## Current-upstream snapshot already tested

```text
GRAALVM_CE=25.5.5-dev+1.1
JDK=25.0.4.1.1
TRUFFLE_MAVEN_PLANE=25.5.5-SNAPSHOT
ORACLE_GRAAL_REVISION=11b21fb2e5691e49d46525dac7b87ef33aeafe82
DEVELOPER_BUILD_RELEASE=25.5.5-dev-20260930_0129
```

The Native Image build contained:

```text
TruffleBaseFeature
TruffleFeature: Provides internal support for Truffle runtime compilation
TruffleAPIFeature
DynamicObjectFeature
HomeFinderFeature
```

so the reproducer was not merely missing Native Truffle runtime-compilation
support.

## Released-line controls

Independent Native testing also retained:

```text
GRAALVM_TRUFFLE_25.4.4.1.1=FAIL_FRAMEWITHOUTBOXING
GRAALVM_TRUFFLE_25.3.4.1=FAIL_FRAMEWITHOUTBOXING
```

The upstream report should lead with the latest-snapshot result; older release
controls are supporting context rather than the central reproducer.

## GR79495 control

Two later upstream fixes associated with GR79495 were tested as a local
backport on the adopted `25.4.4.1.1` plane:

```text
9f2d0364e8c1fe83d4be302c48808869fd864bda
  Fix: explode copyTo in compiled code

4473f2aba25ec57995abcbd260ce4645989a67ad
  Fix: clear return value from frame before returning
```

The patched artifacts and generated result-slot clear were verified active.
Native runtime compilation still produced the same FrameWithoutBoxing bailout.

Do not claim that GR79495 causes this defect; only that these known fixes do not
resolve the reproducer.

## Upstream-facing claim boundary

The strongest currently justified statement is:

> A minimal Truffle Bytecode DSL root that compiles successfully at Tier 2 on
> the JVM fails Tier-2 runtime compilation inside a Native Image on the same
> current Graal/Truffle snapshot plane because a FrameWithoutBoxing is forced to
> materialize across the generated interpreter boundary.

Do not claim the exact compiler implementation fix or the exact internal root
cause unless upstream analysis establishes it.

## Evidence still trapped in the development container

The following must be captured before the temporary workspace is discarded:

```text
/tmp/bug013-min-repro/pom.xml
/tmp/bug013-min-repro/src/main/java/repro/MiniLanguage.java
/tmp/bug013-min-repro/src/main/java/repro/MiniRoot.java
/tmp/bug013-min-repro/src/main/java/repro/MiniPlainRoot.java
/tmp/bug013-min-repro/src/main/java/repro/Main.java

/tmp/bug013-repro-constant.log
/tmp/bug013-repro-local.log
/tmp/bug013-repro-loop.log
/tmp/bug013-repro-yield.log
/tmp/bug013-repro-jvm-plain.log
/tmp/bug013-repro-253-plain.log
```

Also capture any still-present generated
`target/generated-sources/annotations/repro/*Gen.java`, exact environment
identity, Maven dependency-plane identity, complete Native `plain` compiler
stack trace, and file SHA-256 values.

The capture must preserve the exact files that produced the already-observed
results rather than regenerating them from memory.

## Required next checkpoint

Do not draft the final external issue until the capture is durable.

```text
EPHEMERAL_REPRODUCER_PRESERVED=NO
REPRODUCER_FILE_HASHES_RETAINED=NO
ENVIRONMENT_IDENTITY_RETAINED=PARTIAL
LATEST_SNAPSHOT_NATIVE_FAILURE_RETAINED=YES
SAME_SNAPSHOT_JVM_CONTROL_RETAINED=YES
MINIMAL_PLAIN_ROOT_REPRODUCES=YES
FULL_REPRO_COMMANDS_VERIFIED=PARTIAL
UPSTREAM_TEMPLATE_SELECTED=NO
DRAFT_REVIEWED_BY_HUMAN=NO
EXTERNAL_ISSUE_OPENED=NO
```

The next slice is evidence preservation only. It should not modify Protos,
change GraalVM versions, attempt another product repair, or open the upstream
issue.
