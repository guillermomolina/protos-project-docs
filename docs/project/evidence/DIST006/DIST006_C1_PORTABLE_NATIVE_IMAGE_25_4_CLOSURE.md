# DIST006-C1 — portable distribution and Native Image 25.4 closure

Status: PUBLISHED
Owning Issue: `guillermomolina/protos#733` (`DIST006`)
Slice: `DIST006-C1 — distribution / Native Image / full closure`

## Publication identity

```text
PROTOS_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
PROTOS_PARENT_REVISION=7aaaec6923265c99723ce5bca064e5b3ab52b8c4
PROTOS_VERSION=0.3.108-SNAPSHOT
COMMIT_MESSAGE=DIST006-C1: reconcile Native Image 25.4 closure
GRAALVM_GRAAL_TRUFFLE=25.4.4.1.1
JDK_VERSION=25.0.4.1.1
EXPECTED_RUNTIME_CLASS=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
NATIVE_IMAGE_CONTAINER=ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10
```

The published product commit changes exactly eight files:

```text
CHANGELOG.md
build/native/Dockerfile
build/native/generate-init-args.sh
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeTagTreeNodeExports.java
src/main/java/com/guillermomolina/protos/execution/ProtosDebuggerScope.java
tools/test_verify_toolchain.py
tools/verify_toolchain.py
```

No Protos language/specification semantic change or performance optimization is
part of this slice.

## Canonical Native Image 25.4 binding

The stale Native Image 25.3 container binding was replaced with the exact
canonical 25.4 family/tag:

```text
OLD=ghcr.io/graalvm/native-image-community:25i3-25.0.4.1-ol10-20260825
NEW=ghcr.io/graalvm/native-image-community:25i4-25.0.4.1.1-ol10
```

The repository toolchain verifier now derives the expected Native Image
container from the canonical `toolchain.json` Graal container channel and JDK
version. All-surface verification includes `native.image`; development scope
intentionally excludes that binding.

Owner-executed focal validation reported:

```text
TOOLCHAIN_VERIFIER_TESTS=PASS
TOOLCHAIN_BINDING: native.image status=PASS
TOOLCHAIN_DRIFT_COUNT=0
TOOLCHAIN_BINDINGS=PASS
NATIVE_IMAGE_25_4_BINDING=PASS
CANONICAL_RUNTIME_ZERO_DRIFT=PASS
```

## Portable distribution closure

Pre-publication portable-distribution validation was executed from the C1
working state based on product parent revision
`7aaaec6923265c99723ce5bca064e5b3ab52b8c4`, using the explicit dirty-source
validation mode required before commit.

The build and extracted-distribution gates established:

```text
DIST_RUNTIME_PROJECTION_CHECK=PASS
DIST_ARCHIVE_CRC_CHECK=PASS
DIST_LAYOUT_CHECK=PASS
DIST_SOURCE_IDENTITY_CHECK=PASS
DIST_RUNTIME_METADATA_CHECK=PASS
DIST_LAUNCHER_MODE_CHECK=PASS
DIST_BUILD=PASS

DIST_OUTSIDE_CHECKOUT_CHECK=PASS
DIST_RUNTIME_ISOLATION_CHECK=PASS
DIST_PACKAGE_TOOL_CWD_CHECK=PASS
DIST_BUNDLED_TEST_TOOL_CHECK=PASS
TEST001_H_PORTABLE_REPOSITORY_SUITE_CHECK=PASS passed=1263 failed=0
DISTRIBUTION_TEST_TOOL_VALIDATED=YES

DIST_SELECTED_JDK_CHECK=PASS java.version=25.0.4.1.1
DIST_SELECTED_RUNTIME_GATE_CHECK=PASS
DIST_OPTIMIZER_JAR_INTACT_CHECK=PASS version=25.4.4.1.1
DIST_TRUFFLE_COMPILER_CHECK=PASS version=25.4.4.1.1
DIST_DAP_RUNTIME_CLOSURE_CHECK=PASS version=25.4.4.1.1
DIST_OPTIMIZING_RUNTIME_CHECK=PASS class=com.oracle.truffle.runtime.hotspot.HotSpotTruffleRuntime
DIST001_B5_CROSS_SLICE=PASS
PORTABLE_DISTRIBUTION_25_4_CLOSURE=PASS
```

This portable gate establishes the exact 25.4 runtime/compiler/DAP closure,
execution outside the source checkout, runtime isolation, bundled Test Tool
execution, and canonical HotSpot Truffle runtime identity. The later C1 changes
do not alter the portable runtime dependency authority or distribution builder;
the final product version is `0.3.108-SNAPSHOT`.

## Native Image 25.4 compatibility repair

With the canonical 25.4 Native Image container active, the initial build exposed
a real compatibility regression in the existing bootstrap assumptions.

The first failures were hosted compilation entry points that became reachable
without having been seen during Native Image bytecode parsing:

```text
ProtosBytecodeTagTreeNodeExports.hasScope%%D(...)
ProtosDebuggerScope.<init>%%D(ProtosActivation)
```

After exposing the external-receiver `NodeLibrary` export owner to the existing
build-time initialization generator, Native Image 25.4 reported 87 runtime
compilation blocklist violations.

The diagnostic traces showed that these were not 87 independent Protos defects.
They converged through the debugger/tooling projection, principally:

```text
ProtosDebuggerScope.getMembers(...)
ProtosDebuggerScope.visibleNamesSnapshot(...)
ProtosDebuggerScope.appendLocalNames(...)
```

The repair therefore does not maintain a method-by-method allow/block list.
Instead it preserves frame/activation extraction on the PE-visible side and
places debugger-scope construction plus debugger-only member projection behind
narrow `TruffleBoundary` edges. The special external-receiver export owner is
retained in the Native Image initialization input so its generated
`NodeLibrary` entry points are visible during hosted parsing.

This preserves the PLAT038/I069 requirement that guest runtime compilation
remain enabled and does not use fallback compilation or disable the 25.4
blocklist gate.

## Native Image validation

After the structural debugger/tooling boundary repair, the owner executed the
full native regression gate:

```text
NATIVE_BUILD=PASS

NATIVE_VERSION_STATUS=0
NATIVE_HELP_STATUS=0
NATIVE_GUEST_SMOKE_STATUS=0
NATIVE_FORCED_JIT_STATUS=0

OPT_DONE=2
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0

HELPER_BYTECODE_ROOT_TIER2=1
SEMANTIC_BYTECODE_ROOT_TIER2=1

NATIVE_VERSION_SMOKE=PASS
NATIVE_HELP_SMOKE=PASS
NATIVE_GUEST_SMOKE=PASS
NATIVE_FORCED_GUEST_JIT=PASS
HELPER_BYTECODE_ROOT_TIER2=PASS
SEMANTIC_BYTECODE_ROOT_TIER2=PASS
FRAME_WITHOUT_BOXING_REGRESSION=PASS
NATIVE_REGRESSION_SUITE=PASS
```

Thus both generated Bytecode DSL roots still reach Truffle Tier 2 in the Native
Image runtime and the retained forced-JIT invariants remain satisfied.

## Version and final integrated validation

Because the compatibility repair changes executable implementation source under
`src/main/java/**`, C1 consumes the next implementation version:

```text
PROTOS_VERSION=0.3.108-SNAPSHOT
```

The root changelog records the 25.4 Native Image compatibility repair and
canonical binding reconciliation.

After all C1 source/build/version/changelog changes, the owner executed the
canonical integrated repository gate:

```text
make test=PASS
FULL_REQUIRED_VALIDATION=PASS
```

Pre-publication review also established:

```text
git diff --check=PASS
HEAD_BEFORE_PUBLICATION=7aaaec6923265c99723ce5bca064e5b3ab52b8c4
origin/main=7aaaec6923265c99723ce5bca064e5b3ab52b8c4
```

The product commit was then published directly on top of that synchronized
baseline as
`d18822e968a1ee6986d832731054b9b90b332b1d`.

## Scope conclusion

```text
DIST006_C1_STATUS=PUBLISHED
PROTOS_REVISION=d18822e968a1ee6986d832731054b9b90b332b1d
PORTABLE_DISTRIBUTION_25_4_CLOSURE=PASS
NATIVE_IMAGE_25_4_BINDING=PASS
NATIVE_BUILD=PASS
NATIVE_EXECUTABLE_SMOKE=PASS
NATIVE_FORCED_GUEST_JIT=PASS
CANONICAL_RUNTIME_ZERO_DRIFT=PASS
FULL_REQUIRED_VALIDATION=PASS
SEMANTIC_CHANGE=NO
PERFORMANCE_OPTIMIZATION_MIXED_IN=NO
HISTORICAL_25_3_EVIDENCE_REWRITTEN=NO
```

DIST006-C1 completes the portable-distribution and Native Image 25.4 closure
obligations owned by C. Parent DIST006 remains open because the independently
owned DIST006-B2 packaged VS Code editor acceptance is still incomplete.
