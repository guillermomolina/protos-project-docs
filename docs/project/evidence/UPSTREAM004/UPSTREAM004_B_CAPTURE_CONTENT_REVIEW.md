# UPSTREAM004-B — Captured reproducer content review

Status: **ARCHIVE VERIFIED; TWO EVIDENCE GAPS IDENTIFIED**

This durable, non-normative review supports
`guillermomolina/protos#753`.

It reviews the externally retained archive uploaded after UPSTREAM004-A:

```text
ARCHIVE=upstream004-container-evidence.tar.gz
ARCHIVE_SHA256=ef610d44838774a54a2dc8ba9b2a379a9427cafd7640e0e0ad103e7194cce6b0
ARCHIVE_SIZE_BYTES=12216
```

## Integrity verification

The uploaded archive SHA-256 exactly matches the value produced in the
development container and retained by UPSTREAM004-A.

All 20 entries covered by the internal `MANIFEST.sha256` were recomputed from
the uploaded tarball and matched their retained SHA-256 values.

```text
ARCHIVE_SHA256_MATCH=YES
MANIFEST_ENTRY_COUNT=20
MANIFEST_ALL_MATCH=YES
```

The uploaded copy is therefore byte-identical to the captured container
archive.

## Exact standalone sources retained

The archive contains the exact reproducer sources:

```text
reproducer/pom.xml
reproducer/src/main/java/repro/Main.java
reproducer/src/main/java/repro/MiniLanguage.java
reproducer/src/main/java/repro/MiniPlainRoot.java
reproducer/src/main/java/repro/MiniRoot.java
```

The minimum plain root is:

```java
@GenerateBytecode(
        languageClass = MiniLanguage.class,
        enableYield = false)
public abstract class MiniPlainRoot extends RootNode implements BytecodeRootNode {
    protected MiniPlainRoot(
            MiniLanguage language,
            FrameDescriptor frameDescriptor) {
        super(language, frameDescriptor);
    }
}
```

The `plain` builder declares no custom operation, local, loop or yield and
emits only:

```java
b.beginRoot();
b.beginReturn();
b.emitLoadConstant(1L);
b.endReturn();
b.endRoot();
```

The root is called 64 times after:

```java
plainRoot.getBytecodeNode().setUncachedThreshold(0);
```

This is independent of Protos.

## Same-snapshot JVM control retained

`logs/bug013-repro-jvm-plain.log` contains:

```text
MiniPlainRootGen ... |Tier 2| ... opt done
MiniPlainRootGen ... |Tier 2| ... opt done
REPRO_RESULT=1
```

No `FrameWithoutBoxing` bailout is present in that log.

## Same-snapshot Native failure retained

`logs/bug013-repro-plain.log` contains:

```text
MiniPlainRootGen ... |Tier 2| ... opt failed
Object of type Lcom/oracle/truffle/api/impl/FrameWithoutBoxing;
should not be materialized
REPRO_RESULT=1
```

The retained compiler stack reaches:

```text
EnsureVirtualizedNode.ensureVirtualFailure
CommitAllocationNode.lower
...
TruffleCompilerImpl.compilePEGraph
TruffleCompilerImpl.compileAST
...
org.graalvm.truffle.runtime.svm/
  com.oracle.svm.truffle.isolated.IsolateAwareTruffleCompiler.doCompile0
```

This supports the current boundary:

```text
SAME_SNAPSHOT_JVM_TIER2=PASS
SAME_SNAPSHOT_NATIVE_TIER2=FAIL_FRAMEWITHOUTBOXING
GUEST_RESULT_AFTER_NATIVE_BAILOUT=1
```

## Broader retained controls

The uploaded archive also retains Native failures for:

```text
constant
local
loop
yield
```

All reach the same `FrameWithoutBoxing should not be materialized` bailout.

The `enableYield=false` `plain` case remains the smallest and should be the
primary upstream reproducer.

The archive also retains a GraalVM 25.3.4.1 Native `plain` failure with the
same bailout. That older release result is supporting context only; latest
snapshot evidence remains primary.

## Environment identity retained

The current-upstream control environment is captured as:

```text
OS=Oracle Linux Server 10.2
ARCH=x86_64
CPU=AMD Ryzen 7 3700X 8-Core Processor
AVAILABLE_CPUS=16

GRAALVM=GraalVM CE 25.5.5-dev+1.1
JAVA=25.0.4.1.1
JVMCI=25.4-b23
GRAALVM_VERSION_FIELD=25.5.5-dev
ORACLE_GRAAL_COMPILER_REVISION=11b21fb2e5691e49d46525dac7b87ef33aeafe82
ORACLE_GRAAL_TRUFFLE_REVISION=11b21fb2e5691e49d46525dac7b87ef33aeafe82
DEVELOPER_BUILD_RELEASE=25.5.5-dev-20260930_0129
MAVEN=3.9.9
```

The captured GraalVM release metadata itself names
`oracle/graal@11b21fb2e5691e49d46525dac7b87ef33aeafe82`.

## Maven snapshot-plane evidence retained

The archive contains SHA-256 identities for the local
`25.5.5-SNAPSHOT` artifacts, including:

```text
truffle-api
truffle-runtime
truffle-compiler
truffle-dsl-processor
polyglot
graal-sdk
```

The timestamped Truffle artifacts are from the same
`20260930.005636/005637` snapshot publication set.

## Evidence gap 1 — dependency tree capture invalid

The captured:

```text
maven-plane/dependency-tree.txt
```

does **not** contain a dependency tree.

It records:

```text
BUILD FAILURE
Goal requires a project to execute but there is no POM in this directory (/tmp).
```

This happened because the capture command invoked Maven from `/tmp` instead of
`/tmp/bug013-min-repro`.

No dependency-tree claim should be made from this file.

A supplemental capture should run the dependency tree from the reproducer
directory using the same developer-build Java, settings and
`25.5.5-SNAPSHOT` version.

## Evidence gap 2 — generated sources absent

The archive does not contain:

```text
target/generated-sources/annotations/repro/MiniRootGen.java
target/generated-sources/annotations/repro/MiniPlainRootGen.java
```

Those generated sources were inspected during BUG013-C but were not copied into
UPSTREAM004-A.

They are useful supporting evidence for the generated interpreter boundary, but
they are not needed to establish the defect if the standalone source and
JVM/Native result pair remain sufficient.

If retained, they must be regenerated deterministically from the exact captured
sources and the exact `25.5.5-SNAPSHOT` processor plane, then clearly labelled
as a **supplemental regeneration**, not as bytes captured from the original
run.

## Review result

```text
ARCHIVE_EXTERNAL_RETENTION=PASS
ARCHIVE_INTEGRITY=PASS
SOURCE_CONTENT_REVIEW=PASS
MINIMAL_PLAIN_ROOT_CONFIRMED=YES
LATEST_SNAPSHOT_JVM_CONTROL=PASS
LATEST_SNAPSHOT_NATIVE_FAILURE=PASS
COMPILER_STACK_RETAINED=YES
ENVIRONMENT_IDENTITY=PASS
MAVEN_ARTIFACT_IDENTITIES=PASS

DEPENDENCY_TREE_CAPTURE=INVALID
GENERATED_SOURCE_CAPTURE=ABSENT

EXTERNAL_ISSUE_READY=NOT_YET
```

Before the final upstream issue draft, capture the corrected dependency tree and
optionally regenerate/capture the two generated root sources under the exact
current snapshot plane. Do not rerun Protos or alter product code for this
supplement.
