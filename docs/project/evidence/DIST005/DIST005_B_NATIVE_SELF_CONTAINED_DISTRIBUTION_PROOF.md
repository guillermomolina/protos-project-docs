# DIST005-B — Native self-contained distribution proof

Status: PASS
Date: 2026-09-29
Owning Issue: `guillermomolina/protos#549`
Published Protos revision:
`a6aa7f1177f99c77e5148b8363a77d7250184793`

## Scope

DIST005-B implemented and validated the bounded development-only Native Image
self-contained distribution proof selected by DIST005-A.

It does not select the final DIST005 product artifact model, create a public
release, create a tag, replace DIST007, or change observable Protos semantics.

## Publication result

The implementation was published to `guillermomolina/protos` as:

```text
a6aa7f1177f99c77e5148b8363a77d7250184793
DIST005-B: prove self-contained Native distribution
```

Published implementation version:

```text
0.3.110-SNAPSHOT
```

Changed product surfaces:

```text
CHANGELOG.md
bin/protos
build/native/generate-init-args.sh
pom.xml
dist/build_native.py
dist/test_native_launcher.py
dist/validate_native.py
src/main/resources/META-INF/native-image/com.guillermomolina/protos/reachability-metadata.json
```

No normative specification file was changed.

## Implemented artifact model

The development proof archive has one relocatable distribution root containing:

```text
bin/protos
libexec/protos-native
protos/lib/...
protos/tools/...
protos/tests/...
LICENSE.TXT
README.md
SOURCE.txt
RUNTIME.txt
DEPENDENCIES.txt
SHA256SUMS
```

The Native distribution contains no Protos JVM application JAR and no external
Graal/Truffle runtime JAR plane.

The shared `bin/protos` launcher now selects the packaged
`libexec/protos-native` payload before any Java selection or probing, exports
the distribution root as `PROTOS_HOME`, and preserves the existing JVM
checkout/portable-distribution path when no Native payload is present.

## Native build composition

The Maven build was made idempotent across repeated package invocations:

- ordinary JVM packaging retains Shade;
- `maven-jar-plugin` recreates the thin application JAR before post-processing;
- the `native` profile skips Shade;
- Native Image consumes the thin Protos JAR plus the resolved runtime dependency
  graph directly rather than consuming an already shaded JAR and the same
  dependencies again.

The final Native build reported:

```text
NATIVE_BUILD_STATUS=0
NATIVE_SHADE_SKIPPED=PASS
NATIVE_DUPLICATE_INPUT_WARNING=PASS
NATIVE_PRESERVE_WARNING_CLEAN=PASS
NATIVE_TRUFFLE_ACCESS_WARNING_CLEAN=PASS
```

The Native Image build retained Truffle runtime compilation and reported an
x86-64-v3 target machine. The development archive itself deliberately records
`cpu_isa_assumption=unresolved`; no broader CPU/platform support claim is made
by this proof.

## Closed-world reachability repair

LSP4J remained the material closed-world risk. The implementation uses:

```text
-H:+UnlockExperimentalVMOptions
-H:Preserve=package=org.eclipse.lsp4j.*
-H:-UnlockExperimentalVMOptions
--enable-native-access=org.graalvm.truffle
```

and adds explicit dynamic-proxy reachability for:

```text
org.eclipse.lsp4j.services.LanguageClient
org.eclipse.lsp4j.jsonrpc.Endpoint
```

The resulting Native LSP protocol gate passed end-to-end.

This is proof machinery for the current Native runtime surface, not a new Protos
semantic contract.

## Exact final development artifact evidence

The final pre-commit development proof archive was built from the exact
implementation working tree that was subsequently published as
`a6aa7f1177f99c77e5148b8363a77d7250184793`.

Because the archive was intentionally built before the commit, its embedded
source metadata records the then-current baseline HEAD and dirty state:

```text
NATIVE_DIST_SOURCE_REVISION=754de7a2a2d73dd4b39109bb842522ed9cc8153a
NATIVE_DIST_SOURCE_DIRTY=true
```

This is therefore development-proof evidence, not clean-candidate/release
evidence.

Final artifact identity:

```text
implementation_version=0.3.110-SNAPSHOT
target_os=linux
target_arch=x86_64
linkage=dynamic
libc_family=glibc

native_binary_sha256=c46aa09210641e81e0f357f9c25f188acd1dfa669750c185f0b028266dca516e
archive=protos-0.3.110-SNAPSHOT-native-linux-x86_64.zip
archive_sha256=3117fb827149a422b7dd9305f3334193a305d7503408dcef43d06e74d05e03a6
```

The external checksum recorded the portable archive basename and the same
SHA-256.

## Final validation

Repository-wide Maven validation completed successfully:

```text
MAVEN_VERIFY=0
JAR_LICENSE=PASS
```

The final Native archive passed the complete composed validation with no external
Java or Maven visible on the proof PATH.

Public-surface admission:

```text
NATIVE_DIST_RELOCATED_ROOT_CHECK=PASS
NATIVE_DIST_NO_JAVA_PATH_CHECK=PASS
NATIVE_DIST_NO_MAVEN_PATH_CHECK=PASS

NATIVE_DIST_VERSION_CHECK=PASS
NATIVE_DIST_HELP_CHECK=PASS
NATIVE_DIST_EVAL_CHECK=PASS
NATIVE_DIST_EXTERNAL_SOURCE_CHECK=PASS
NATIVE_DIST_UNRELATED_CWD_CHECK=PASS
NATIVE_DIST_PROTOS_HOME_DERIVATION_CHECK=PASS

NATIVE_DIST_PACKAGE_TOOL_ADMISSION_CHECK=PASS
NATIVE_DIST_WORKSPACE_RUN_ADMISSION_CHECK=PASS
NATIVE_DIST_TEST_TOOL_FOCAL_ADMISSION_CHECK=PASS

NATIVE_DIST_REPL_PTY_ADMISSION_CHECK=PASS
NATIVE_DIST_LSP_STDIO_ADMISSION_CHECK=PASS
NATIVE_DIST_DAP_ADMISSION_CHECK=PASS
```

The DAP proof exercises the existing public debug readiness/TCP protocol path
through initialize, initialized, launch, configurationDone, routed stdout/stderr,
termination, and process exit. It does not create a new debugger protocol
contract.

Forced guest runtime-compilation evidence against the exact extracted Native
payload:

```text
NATIVE_DIST_FORCED_JIT_OPT_DONE=2
NATIVE_DIST_FORCED_JIT_OPT_FAILED=0
NATIVE_DIST_FORCED_JIT_FRAME_FAILURES=0
NATIVE_DIST_FORCED_JIT_COMPILATION_FAILURES=0
NATIVE_DIST_FORCED_JIT_HELPER_TIER2=1
NATIVE_DIST_FORCED_JIT_SEMANTIC_TIER2=1

NATIVE_DIST_FORCED_GUEST_JIT_CHECK=PASS
```

Definitive complete Native Test Tool run:

```text
NATIVE_DIST_FULL_TEST_TOOL_JOBS=16
NATIVE_DIST_FULL_TEST_TOOL_STATUS=0
NATIVE_DIST_FULL_TEST_TOOL_SECONDS=92.35
NATIVE_DIST_FULL_TEST_TOOL_CONTEXT_TEARDOWN_FAILURES=0
NATIVE_DIST_FULL_TEST_TOOL_REFLECTION_FAILURES=0
NATIVE_DIST_FULL_TEST_TOOL_PASSED=1263
NATIVE_DIST_FULL_TEST_TOOL_FAILED=0
DIST005_NATIVE_FULL_TEST_TOOL_ADMISSION=PASS
```

Archive immutability and composed closure:

```text
NATIVE_DIST_OUTER_CHECKSUM_CHECK=PASS
NATIVE_DIST_ARCHIVE_UNCHANGED_CHECK=PASS
DIST005_NATIVE_A_G_ADMISSION=PASS
DIST005_NATIVE_EXTRACTED_JIT_ADMISSION=PASS
DIST005_NATIVE_COMPLETE_ADMISSION=PASS
```

## What DIST005-B proves

```text
NATIVE_RELOCATABLE_ARCHIVE_PROOF=PASS
NATIVE_EXTERNAL_JAVA_REQUIRED=NO
NATIVE_EXTERNAL_MAVEN_REQUIRED=NO
NATIVE_EXTERNAL_PROTOS_CHECKOUT_REQUIRED=NO
EXTERNAL_PROTOS_RESOURCE_TREE_PRESERVED=YES

NATIVE_RUN_SURFACE=PASS
NATIVE_PACKAGE_SURFACE=PASS
NATIVE_TEST_SURFACE=PASS
NATIVE_REPL_SURFACE=PASS
NATIVE_LSP_SURFACE=PASS
NATIVE_DAP_SURFACE=PASS
NATIVE_OPTIMIZING_TRUFFLE_SURFACE=PASS

CLOSED_WORLD_BLOCKER_PRESENT=NO_FOR_PROVEN_SURFACES
OBSERVABLE_PROTOS_SEMANTIC_CHANGE=NO
```

The result closes the technical conditions that DIST005-A left open for one
canonical Linux/x86_64/glibc development proof.

## Remaining DIST005 boundaries

DIST005 itself remains open.

DIST005-B does not decide:

```text
FINAL_ARTIFACT_MODEL=UNSELECTED
JVM_ONLY vs JVM_PLUS_NATIVE vs NATIVE_ONLY=UNSELECTED
RECOMMENDED_FIRST_RUN_ARTIFACT=UNSELECTED
PUBLIC_NATIVE_SUPPORT_MATRIX=UNSELECTED
PUBLIC_RELEASE_CREATED=NO
DIST007_BLOCKED=NO
```

The current proof is one Linux/x86_64/glibc dynamically linked target. It is not
evidence for a multi-platform Native support matrix.

The release metadata/envelope machinery is still JVM-artifact-centric and would
need a bounded extension if the selected product model publishes Native alongside
the portable JVM artifact.

Final public-release third-party notice/compliance review remains a release
boundary rather than part of this development proof.

## Next bounded work

The next slice is an investigation/decision packet:

```text
DIST005-C — final self-contained artifact-model selection packet
WORK_TYPE=INVESTIGATION
PRODUCT_REPOSITORY=guillermomolina/protos
COMMAND_EXECUTION=NONE
```

It should re-evaluate the three product models now that the Native proof is no
longer conditional on the previously open tooling/closed-world gates:

```text
JVM_ONLY
JVM_PLUS_NATIVE
NATIVE_ONLY
```

The packet must separate technical feasibility from the product choice, compare
first-run behavior, breadth of platform support, retained JVM value, Native
target specificity, release-envelope impact, maintenance/security/update costs,
artifact size/startup considerations, and the relationship to DIST007.

It must stop with an exact owner-selection packet and must not choose the final
artifact model on the owner's behalf.

No new Dxxx/PLATxxx should be allocated unless the current evidence exposes a
durable platform/semantic choice outside the already-ratified PLAT038/PLAT039
boundaries.

## Conclusion

```text
DIST005_B_STATUS=PASS
PROTOS_REVISION=a6aa7f1177f99c77e5148b8363a77d7250184793
IMPLEMENTATION_VERSION=0.3.110-SNAPSHOT

NATIVE_ARCHIVE_PROOF=PASS
NO_EXTERNAL_JAVA=PASS
FULL_PUBLIC_TOOLING_SURFACE=PASS
GUEST_TIER2_RUNTIME_COMPILATION=PASS
FULL_NATIVE_TEST_TOOL=1263/1263_PASS

FINAL_DIST005_ARTIFACT_MODEL_SELECTED=NO
DIST005_PARENT_STATUS=OPEN
NEXT_SLICE=DIST005-C
NEXT_SLICE_WORK_TYPE=INVESTIGATION
DIST007_BLOCKED=NO
```
