# PLAT038 — Native Image bootstrap/runtime/release investigation evidence

Status: retained decision evidence

Decision: PLAT038 / guillermomolina/protos#710

Selected candidate after explicit project-owner approval: **Candidate B — dual-runtime build with native consumption and JVM development authority**.

Product baseline:

~~~text
PROTOS_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
JDK_VERSION=25.0.4.1
MAVEN_VERSION=3.9.9
~~~

## Repository evidence inspected

~~~text
guillermomolina/protos:
  AGENTS.md
  AGENTS.work/DESIGN.md
  AGENTS.work/REFERENCE.md
  AGENTS.work/RELEASE.md
  Makefile
  pom.xml
  toolchain.json
  bin/protos
  .devcontainer/Dockerfile
  .github/workflows/tests.yml
  dist/build_portable.py
  dist/validate_portable.sh
  src/main/java/com/guillermomolina/protos/cli/ProtosCli.java
  src/main/java/com/guillermomolina/protos/execution/ProtosLanguage.java
  production reflection/dynamic-loading search
  PLAT038 / #710

guillermomolina/protos-project-docs:
  PLAT033 optimizing runtime packaging authority
  DIST001 release policy
  DIST002 toolchain alignment
~~~

Key repository facts:

1. make test runs Java/JUnit and then Protos conformance through the checkout JVM launcher.
2. bin/protos selects checkout target/runtime jars or the portable JVM runtime plane and exports PROTOS_HOME.
3. ProtosCli resolves Core physically under PROTOS_HOME/protos/lib/core.
4. the portable distribution includes the Protos source/tool tree independently from lib/protos.jar and the Graal/Truffle runtime closure.
5. PLAT033 requires one exact canonical optimizing Graal/Truffle authority and explicitly preserved a future Native Image path.
6. DIST001 separates implementation snapshots from releases and has a proven detached-worktree release-candidate mechanism that leaves active main unchanged.
7. DIST002 aligns the canonical current toolchain at GraalVM Community 25.3.4.1 / JDK 25.0.4.1 / Graal-Truffle 25.3.4.1 / Maven 3.9.9.

## GraalVM / Truffle evidence

### SimpleLanguage

Current SimpleLanguage provides both a JVM standalone launcher and a native
standalone executable. Its repository builds the native form with a dedicated
native profile and CI invokes both forms.

Relevant upstream sources:

- https://github.com/graalvm/simplelanguage
- https://github.com/graalvm/simplelanguage/blob/master/standalone/README.md
- https://github.com/graalvm/simplelanguage/blob/master/standalone/pom.xml

This establishes Native Image as an upstream-supported Truffle language product
form rather than a Protos-specific deployment model.

### Runtime compilation inside Native Image

GraalVM Native Image build output distinguishes runtime-compiled methods and
documents Truffle languages as a case where the runtime compiler remains inside
the image.

Truffle engine options continue to expose guest compilation, background
compilation, multi-tier compilation, thresholds, OSR and compilation tracing.

Relevant upstream documentation:

- https://www.graalvm.org/dev/reference-manual/native-image/overview/BuildOutput/
- https://www.graalvm.org/jdk25/graalvm-as-a-platform/language-implementation-framework/Options/

Conclusion:

~~~text
HOST_JAVA_AOT=YES
GUEST_TRUFFLE_RUNTIME_COMPILATION=SUPPORTED
NATIVE_EQUALS_NO_GUEST_WARMUP=NO
INTERPRETER_ONLY_NATIVE_REQUIRED=NO
~~~

### GraalJS / TruffleRuby / GraalPy

Maintained Truffle implementations preserve meaningful JVM/native duality.
Native execution is especially useful for startup/latency and self-contained
consumption while JVM execution remains valuable for development, diagnostics
and long-running peak-throughput behavior.

GraalPy additionally demonstrates that single-file-like resource virtualization
is possible, but only with explicit filesystem/resource machinery. Protos has no
current need to pay that complexity merely to obtain a native launcher.

Relevant documentation:

- https://www.graalvm.org/jdk25.1/reference-manual/js/FAQ/
- https://www.graalvm.org/latest/reference-manual/ruby/ReportingPerformanceProblems/
- https://www.graalvm.org/latest/reference-manual/python/Python-Runtime/

### Espresso and Sulong

Espresso and Sulong demonstrate that native-hosted language runtimes can still
load/execute guest code dynamically. Sulong explicitly combines a native-hosted
runtime with later runtime compilation of hot guest code.

Relevant documentation:

- https://www.graalvm.org/jdk25.3/reference-manual/llvm/
- https://www.graalvm.org/latest/reference-manual/java-on-truffle/

This reinforces the host-AOT / guest-dynamic-optimization distinction used by Candidate B.

## Native Image reachability evidence

Native Image uses closed-world reachability. Reflection, resources, JNI,
proxies and other dynamic facilities can require reachability metadata.

Current Protos production-source search found no broad direct use of
java.lang.reflect, Class.forName, getDeclaredMethod or getDeclaredField in
src/main. Java tests do use reflection substantially, but those tests remain a
JVM surface under Candidate B.

Third-party runtime/tool dependencies still require end-to-end native validation.

Relevant documentation:

- https://www.graalvm.org/latest/reference-manual/native-image/metadata/
- https://www.graalvm.org/dev/reference-manual/native-image/guides/use-reachability-metadata-repository-maven/

Conclusion:

~~~text
PROTOS_PRODUCTION_REFLECTION_RISK=BOUNDED_BY_SOURCE_INSPECTION
THIRD_PARTY_REACHABILITY_RISK=REAL
FAIL_CLOSED_NATIVE_METADATA_VALIDATION=REQUIRED
TRACING_AGENT_AS_PERMANENT_AUTHORITY=NO
~~~

## Native Build Tools / Maven evidence

GraalVM provides official Native Build Tools integration for Maven, including
Native Image build/test integration and Reachability Metadata Repository use.

Protos is already Maven-based. A second independent build system is therefore
unnecessary.

Exact plugin version, Maven profile/goal and Makefile aliases are deliberately
left to I069 implementation.

## Apple Pkl comparison

Apple Pkl publishes Java and native executable flavors from the same codebase
and documents the native flavor as avoiding the Java runtime/startup requirement.
Its repository also contains a substantial explicit Native Image path.

Pkl is useful evidence that JVM/native product forms can coexist, but its
single-binary/resource choices are not authority for Protos.

Relevant upstream sources:

- https://github.com/apple/pkl
- https://pkl-lang.org/main/current/pkl-cli/index.html

Conclusion:

~~~text
JVM_NATIVE_DUALITY=STRONG_PRECEDENT
SINGLE_FILE_RESOURCE_EMBEDDING=OPTIONAL_NOT_REQUIRED
~~~

## Bootstrap/toolchain precedent

Rust and GCC use explicit compiler stages because the compiler toolchain is
self-hosting or bootstrap-sensitive.

Protos is currently implemented in Java and built by Java/Maven/GraalVM.
Therefore an installed previous Protos executable is not required to compile the
next Protos implementation.

The useful precedent is separation of identities, not artificial self-hosting:

~~~text
STABLE_INSTALLED_PROTOS=STAGE0_OPERATIONAL
CURRENT_JVM_BUILD=STAGE1_JVM
CURRENT_NATIVE_BUILD=STAGE1_NATIVE
STAGE0_REQUIRED_TO_BUILD_STAGE1=NO
~~~

Relevant references:

- https://rustc-dev-guide.rust-lang.org/building/bootstrapping/what-bootstrapping-does.html
- https://gcc.gnu.org/install/build.html

## Release isolation

LLVM uses release branches because its release process needs stabilization and
parallel maintenance. That is valid precedent for a project with sustained
release branches, but it does not establish that Protos needs one now.

Protos DIST001 already proved a smaller mechanism:

~~~text
selected exact V-SNAPSHOT main SHA
  -> detached external worktree
  -> release-only V transition/commit
  -> validation
  -> explicit publication authorization
  -> tag / GitHub Release
~~~

Active main remains independent.

Relevant external reference:

- https://llvm.org/docs/HowToReleaseLLVM.html

Current Protos authority:

- docs/project/work/DIST001/DIST001_RELEASE_POLICY.md

Conclusion:

~~~text
NEW_RELEASE_BRANCH_REQUIRED=NO
DIST001_DETACHED_CANDIDATE_REUSE=YES
MAIN_CONTINUES_DURING_RELEASE=YES
~~~

## Resource-layout discriminator

Current Protos runtime lookup uses PROTOS_HOME and a physical
protos/lib/core tree. Package Tool, Test Tool, Standard Library and related
bundled Protos sources are also represented as filesystem trees in the current
distribution.

Therefore Native Image can remove the JVM/Graal runtime-jar requirement without
requiring an immediate one-file complete toolchain:

~~~text
native executable
+
existing Protos resource/tool tree
~~~

Embedding/virtualizing that tree would create new package/module/resource
resolution and tooling questions. It is unnecessary to establish Native Image
as a first-class runtime and is deferred.

## Candidate discrimination

### Candidate A — JVM-only / defer

Advantages: no new build surface; current behavior already proven.

Failure: does not solve stable no-JVM consumption/startup need.

### Candidate B — dual runtime

Advantages:

- preserves JVM development and diagnostics;
- makes native a first-class user/runtime artifact;
- reuses PLAT033 authority and DIST001 release isolation;
- keeps current Protos resource tree;
- preserves later single-file and multi-platform options;
- imposes native build cost only when native validation is requested.

Risks:

- Native Image reachability metadata and platform build prerequisites;
- separate native validation lane;
- native distribution matrix still requires later DIST work.

Disposition: **SELECTED**.

### Candidate C — immediate complete single-file native toolchain

Failure: requires resource embedding/virtual filesystem decisions not needed by
the motivating requirement and unnecessarily couples module/tool resource
semantics to Native Image.

Disposition: rejected as premature complexity.

### Candidate D — native-first development

Failure: forces AOT build constraints into ordinary implementation cycles and
weakens the JVM/Graal diagnostic path without demonstrated compensating value.

Disposition: rejected.

## Decision evidence summary

~~~text
NATIVE_IMAGE_ARCHITECTURALLY_VALID=YES
TRUFFLE_RUNTIME_COMPILER_CAN_EXIST_IN_NATIVE=YES
GUEST_JIT_REQUIRED_BY_PLAT038=YES
NATIVE_ZERO_GUEST_WARMUP_CLAIM=REJECTED

STAGE0_SELF_HOSTING_REQUIRED=NO
STAGE0_OPERATIONAL_STABLE_TOOLCHAIN=YES

JVM_DEVELOPMENT_AUTHORITY=RETAIN
NATIVE_USER_RUNTIME_ARTIFACT=ADOPT

JVM_CONFORMANCE=REQUIRED
NATIVE_CONFORMANCE=REQUIRED_FOR_NATIVE_READINESS
INSTALLED_STAGE0_AS_STAGE1_TEST_AUTHORITY=FORBIDDEN

CURRENT_EXTERNAL_PROTOS_RESOURCE_TREE=RETAIN
SINGLE_FILE_COMPLETE_TOOLCHAIN=DEFER

BUILD_NATIVE_DIST_RELEASE_SEPARATION=REQUIRED
DIST001_DETACHED_RELEASE_CANDIDATE=REUSE
CHANGELOG_TRIGGERS_DIST_OR_RELEASE=NO

I069_IMPLEMENTATION_READY=YES
FOLLOWUP_DIST_ALLOCATION=DEFER_UNTIL_I069_EVIDENCE
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
~~~
