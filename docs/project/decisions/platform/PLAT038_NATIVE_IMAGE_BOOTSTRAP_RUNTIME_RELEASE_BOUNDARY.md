# PLAT038 — Native Image bootstrap, runtime and release boundary

Status: RATIFIED

Selected architecture: **Candidate B — dual-runtime build with native consumption and JVM development authority**.

Approval: explicit project-owner approval on 2026-09-25:

~~~text
aprobado candidate b.
~~~

Decision Issue: guillermomolina/protos#710

Implementation consumer: I069 / guillermomolina/protos#711

Product baseline:

~~~text
PROTOS_REVISION=f1cee2d85858804ad3775adf43a9fab97664da2a
PROTOS_VERSION=0.3.87-SNAPSHOT
GRAALVM_TRUFFLE_VERSION=25.3.4.1
JDK_VERSION=25.0.4.1
MAVEN_VERSION=3.9.9
~~~

Primary prior authorities:

- PLAT033 — one canonical optimizing Graal/Truffle runtime authority;
- DIST001 — implementation snapshots are not releases and exact public candidates use detached release-only worktrees;
- DIST002 — development, CI and live distribution runtime/toolchain alignment.

Nature: durable non-normative runtime/build/bootstrap architecture decision.
Observable Protos semantics remain owned by the normative specification.

## Decision

Protos adopts a dual-runtime build architecture.

The current source checkout may produce two implementation forms:

~~~text
stage1-JVM
  current checkout
  JVM/Truffle execution
  primary implementation-development and compiler-diagnostic surface

stage1-native
  current checkout
  GraalVM Native Image execution
  first-class runtime/user artifact
  same canonical Graal/Truffle authority
~~~

A separately installed stable released Protos toolchain may be called stage0
operationally:

~~~text
stage0
  stable installed released Protos toolchain
  intended for consuming/using Protos
  NOT required to build stage1
~~~

Protos is not currently self-hosting. Java/Maven/GraalVM remain the actual
bootstrap toolchain for the Protos implementation. PLAT038 therefore does not
manufacture a GCC/Rust-style stage0 -> stage1 compiler dependency that does not
exist.

## Native runtime contract

Native Image is a first-class build artifact, not a release action.

The native executable must retain the optimizing Truffle guest-runtime behavior
required by Protos. Host Java code is AOT-compiled by Native Image, but guest
Protos execution remains eligible for Truffle profiling, specialization and
runtime compilation.

~~~text
TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
GUEST_JIT_BEHAVIOR=REQUIRED_AND_VERIFIED
INTERPRETER_ONLY_NATIVE_ACCEPTABLE=NO
~~~

PLAT038 does not claim that Native Image removes all Protos warm-up. It removes
or substantially reduces JVM/host startup and host-JIT work; guest Truffle
warm-up and runtime compilation remain part of optimized Protos execution.

## Build, distribution and release are independent operations

The following concepts remain distinct:

~~~text
JVM BUILD
!=
NATIVE BUILD
!=
DISTRIBUTION ASSEMBLY
!=
PUBLIC RELEASE
~~~

In particular:

~~~text
CHANGELOG_IMPLIES_DISTRIBUTION=NO
CHANGELOG_IMPLIES_RELEASE=NO
SNAPSHOT_VERSION_IMPLIES_RELEASE=NO
NATIVE_BUILD_IMPLIES_DISTRIBUTION=NO
NATIVE_BUILD_IMPLIES_RELEASE=NO
~~~

An implementation revision may build and validate JVM and native artifacts
without creating a public distribution, tag or GitHub Release.

## Release isolation

PLAT038 reuses the already-proven DIST001 exact-candidate model rather than
introducing a new release-branch policy.

A selected V-SNAPSHOT baseline may be materialized in an external detached
worktree, transitioned to public version V in a release-only candidate commit,
fully validated and only later tagged/published after explicit authorization.

The active main branch remains free to continue development independently.

~~~text
EXACT_REVISION_RELEASE_ISOLATION=
  EXISTING_DIST001_DETACHED_WORKTREE_CANDIDATE_MODEL

MAIN_CAN_CONTINUE_DURING_RELEASE=YES
~~~

A future release branch may be investigated only if Protos later acquires a
real need for long-lived stabilization, backports or parallel maintenance
series. PLAT038 does not create that machinery pre-emptively.

## Conformance and validation boundary

The installed stable stage0 executable must never substitute for validation of
the current source revision.

Required authority:

~~~text
JAVA_TESTS=
  JVM

PROTOS_CONFORMANCE_STAGE1_JVM=
  REQUIRED

PROTOS_CONFORMANCE_STAGE1_NATIVE=
  REQUIRED_FOR_NATIVE_READINESS_AND_ANY_NATIVE_DISTRIBUTION_OR_RELEASE

NATIVE_RUNTIME_VALIDATION=
  CLI/tooling smoke
  same Protos conformance authority
  optimizing-runtime / guest-compilation proof
~~~

The native lane must invoke the exact stage1-native artifact derived from the
checkout under test, not whichever protos happens to be first on PATH.

Native JUnit for the entire Java suite is not required by this architecture.
Focused Native Image Java tests remain available when a specific AOT/reachability
regression warrants them.

## Resource and toolchain boundary

The initial native architecture does not require a single-file Protos toolchain.

Current Protos Core, Standard Library, Package Tool, Test Tool and other bundled
Protos sources are filesystem resources under PROTOS_HOME. I069 must preserve
that existing external resource-tree model unless concrete implementation
evidence exposes a new design requirement.

Therefore:

~~~text
NATIVE_IMPLEMENTATION_EXECUTABLE=SINGLE_NATIVE_EXECUTABLE_ALLOWED
COMPLETE_PROTOS_TOOLCHAIN_SINGLE_FILE=NOT_REQUIRED
EXTERNAL_PROTOS_RESOURCE_TREE=PRESERVED
VIRTUAL_FILESYSTEM_OR_EMBEDDED_STDLIB=DEFERRED
~~~

This keeps I069 bounded. Embedding or virtualizing the Protos tree would be a
separate architecture choice because it changes packaging, module/resource
resolution and tooling assumptions.

## CLI and tooling parity

The target architecture is one Protos public CLI surface across JVM and native
forms, including the existing file/eval/run/package/test/REPL/debug/language
server commands where technically supported.

PLAT038 does not pre-authorize a permanent JVM-only public command. I069 should
first attempt native parity and return to design governance only if concrete
evidence establishes a fundamental incompatibility or a materially better
architectural boundary.

The JVM remains the primary implementation/compiler-diagnostic surface even
when end-user native execution is available.

## Reachability and Native Image metadata

Native Image closed-world reachability is an implementation constraint, not a
reason to change Protos semantics.

I069 must use a reproducible, fail-closed reachability/resource configuration
for production dependencies and tooling surfaces. Discovery aids such as the
Native Image tracing agent may be used to investigate missing metadata but are
not by themselves durable authority.

Current source inspection found no broad production use of Java reflection or
Class.forName in the Protos implementation; third-party libraries and tooling
still require end-to-end native validation.

## Toolchain authority

PLAT033 remains authoritative.

The native build must derive from the same canonical repository-owned
Graal/Truffle coordinates used by JVM development/testing. It must not introduce
an independently versioned native runtime authority.

~~~text
CANONICAL_GRAAL_TRUFFLE_AUTHORITY=ONE
NATIVE_COORDINATE_DRIFT=FORBIDDEN
NATIVE_BUILD_SYSTEM=MAVEN_INTEGRATED
SECOND_INDEPENDENT_BUILD_SYSTEM=NO
~~~

Exact Maven plugin names/goals and Makefile target names are I069 implementation
details as long as this authority remains intact.

## Platform/portability boundary

I069 proves the architecture first on the canonical current development host
and toolchain.

A user-facing native distribution later needs an explicit OS/architecture/libc
support matrix, checksums, notices and release metadata. Linux x86_64,
Linux aarch64, glibc/musl, macOS and Windows are distribution choices and are
not silently promised merely because Native Image can target them.

~~~text
I069_PORTABILITY_SCOPE=CANONICAL_DEVELOPMENT_HOST_FIRST
PUBLIC_NATIVE_PLATFORM_MATRIX=DEFERRED_TO_DIST_WORK
~~~

## Candidate comparison

PLAT038 considered:

- Candidate A — JVM-only / defer Native Image;
- Candidate B — dual-runtime build, JVM development authority and native
  consumption/runtime artifact;
- Candidate C — Candidate B plus immediate single-file embedding/virtualization
  of the Protos resource tree;
- Candidate D — native-first development/runtime architecture.

Candidate B is selected because it is the smallest architecture that solves the
actual consumption/startup/bootstrap need while preserving JVM diagnostics,
PLAT033 single runtime authority, DIST001 release isolation, and future
single-file/native-platform options.

A leaves the motivating need unresolved. C adds virtual-filesystem/resource
machinery without a current requirement. D makes Native Image build constraints
part of every ordinary implementation-development cycle without demonstrated
benefit.

## Owner-approved invariant/delta consistency check

The exact selected Candidate B preserves every owner-established PLAT038
constraint recorded before approval:

~~~text
NATIVE_BUILD_IS_NOT_DISTRIBUTION=PASS
NATIVE_BUILD_IS_NOT_RELEASE=PASS
CHANGELOG_DOES_NOT_FORCE_DISTRIBUTION=PASS
CHANGELOG_DOES_NOT_FORCE_RELEASE=PASS
EXACT_REVISION_CAN_BE_FROZEN_WHILE_MAIN_CONTINUES=PASS
STABLE_INSTALLED_TOOLCHAIN_CAN_COEXIST_WITH_CHECKOUT=PASS
CURRENT_CHECKOUT_TESTS_VALIDATE_CURRENT_CHECKOUT=PASS
JVM_DEVELOPMENT_PATH_RETAINED=PASS
~~~

No owner-approved invariant is reopened or contradicted by Candidate B.

Material candidate refinement introduced during investigation is also preserved
explicitly:

~~~text
STAGE0_IS_OPERATIONAL_NOT_SELF_HOSTING_BOOTSTRAP=YES
GUEST_TRUFFLE_JIT_REMAINS_REQUIRED_IN_NATIVE=YES
NATIVE_DOES_NOT_MEAN_ZERO_GUEST_WARMUP=YES
SINGLE_FILE_COMPLETE_TOOLCHAIN_NOT_REQUIRED=YES
DIST001_DETACHED_RELEASE_CANDIDATE_REUSED=YES
PUBLIC_NATIVE_PLATFORM_MATRIX_DEFERRED=YES
~~~

## Ratification summary

~~~text
NATIVE_IMAGE_FIRST_CLASS_ARTIFACT=YES

STAGE0_MODEL=
  STABLE INSTALLED RELEASED PROTOS TOOLCHAIN;
  OPERATIONAL CONSUMPTION ROLE ONLY;
  NOT A BUILD DEPENDENCY FOR STAGE1

STAGE1_JVM_MODEL=
  CURRENT CHECKOUT JVM IMPLEMENTATION;
  PRIMARY LANGUAGE-IMPLEMENTATION DEVELOPMENT/DIAGNOSTIC SURFACE

STAGE1_NATIVE_MODEL=
  EXPLICIT NATIVE IMAGE DERIVED FROM THE SAME CURRENT CHECKOUT
  AND THE SAME CANONICAL GRAAL/TRUFFLE AUTHORITY

TRUFFLE_RUNTIME_COMPILER_IN_NATIVE=REQUIRED
GUEST_JIT_BEHAVIOR=REQUIRED_AND_VERIFIED
INTERPRETER_ONLY_NATIVE_ACCEPTABLE=NO

JVM_CONFORMANCE_REQUIRED=YES
NATIVE_CONFORMANCE_REQUIRED=
  YES FOR NATIVE READINESS AND ANY NATIVE DISTRIBUTION/RELEASE

NATIVE_TEST_SCOPE=
  SAME PROTOS CONFORMANCE AUTHORITY
  + NATIVE CLI/TOOLING SMOKES
  + OPTIMIZING-RUNTIME/GUEST-COMPILATION PROOF

BUILD_NATIVE_DIST_RELEASE_SEPARATION=REQUIRED
CHANGELOG_IMPLIES_DISTRIBUTION=NO
CHANGELOG_IMPLIES_RELEASE=NO

EXACT_REVISION_RELEASE_ISOLATION=
  EXISTING DIST001 DETACHED-WORKTREE CANDIDATE MODEL

MAIN_CAN_CONTINUE_DURING_RELEASE=YES

SUPPORTED_HOST_PORTABILITY=
  I069 PROVES CANONICAL DEVELOPMENT HOST FIRST;
  USER-FACING OS/ARCH/LIBC MATRIX DEFERRED TO DIST

DEBUG_PROFILING_BOUNDARY=
  JVM REMAINS PRIMARY IMPLEMENTATION/GRAAL DIAGNOSTIC SURFACE;
  NATIVE REMAINS DIAGNOSABLE WITH NATIVE/TRUFFLE TOOLS

RESOURCE_REFLECTION_REQUIREMENTS=
  EXTERNAL PROTOS RESOURCE TREE PRESERVED;
  FAIL-CLOSED NATIVE REACHABILITY VALIDATION;
  NO SINGLE-FILE/VFS REQUIREMENT IN I069

PLAT038_SELECTED_CANDIDATE=B
PLAT038_SELECTED_CANDIDATE_NAME=
  DUAL_RUNTIME_BUILD_NATIVE_CONSUMPTION_JVM_DEVELOPMENT_AUTHORITY

OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
IMPLEMENTATION_READY=YES
FOLLOWUP_I_REQUIRED=YES — I069/#711
FOLLOWUP_DIST_REQUIRED=
  YES AFTER I069 PROVES THE NATIVE BUILD/VALIDATION CONTRACT;
  DO NOT ALLOCATE YET

NEW_DECISION_REQUIRED_BEYOND_THIS_PLAT=
  NO FOR CURRENT I069 PATH
~~~
