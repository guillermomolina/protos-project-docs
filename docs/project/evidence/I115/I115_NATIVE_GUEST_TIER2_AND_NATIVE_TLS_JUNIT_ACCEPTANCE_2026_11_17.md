# I115 — Native guest Tier-2 JIT and Native TLS JUnit image acceptance (2026-11-17)

**Non-normative, maintainer-supplied validation evidence.** This records the bounded native behavior observed on 2026-11-17 and the subsequent published product change. It is not a new language/runtime decision, a general GraalVM fix, or an assertion that the full validation transcript was captured from the post-commit revision.

## Exact publication and provenance

- Product owner: [I115 / protos#931](https://github.com/guillermomolina/protos/issues/931).
- Published source and test commit: [`guillermomolina/protos@50a7d3b6924c8a2a87e0488f909d8fa98316dc66`](https://github.com/guillermomolina/protos/commit/50a7d3b6924c8a2a87e0488f909d8fa98316dc66), subject `I115: Require Native guest Tier-2 JIT and initialize the shared test Polyglot context at run time in the Native TLS JUnit image`.
- Published version: **0.4.155-SNAPSHOT**; the commit changes `pom.xml`, `CHANGELOG.md`, `scripts/native-test.sh`, and `src/test/java/com/guillermomolina/protos/execution/compiler/closures/ProtosPerf006C1OptimizingRuntimeClosureTest.java`.
- The complete supplied `make test-native` transcript identifies the locally built executable as **0.4.154-SNAPSHOT** on GraalVM Community Java 25, **GraalVM/Truffle 25.5.5.0.3**; it predates the product version bump in the published commit. **No checkout SHA occurs in that transcript.** The maintainer later reports that all local tests pass and `git diff --check` is clean after pushing the named commit. These are distinct provenance statements, not a fabricated exact-revision test run.

## Maintained positive Native JIT admission

The new `scripts/native-test.sh` replaces the historical PLAT045 interpreter-only check with a **fail-closed positive Tier-2 compilation check** on the actual Native executable.

- The file-backed guest probe defines `x: 1`, `f: () => { x }`, and `g: () => { (() => false).whileTrue(() => { null }) }`; it executes 500 `f()`/`g()` pairs and ends with `print(f())`.
- Engine options: `AllowExperimentalOptions=true`, `TraceCompilation=true`, `CompileImmediately=true`, `BackgroundCompilation=false`, and `CompilationFailureAction=Print`.
- It requires successful Tier-2 traces for **both** `ProtosBytecodeRootNodeGen` and `ProtosBuiltinCallRootNode` (distinct current I115 execution-root types), guest output `1`, process exit 0, and zero compilation, `FrameWithoutBoxing`, and fallback-runtime failures. Correct guest execution alone is not sufficient.

Maintainer-supplied results from `scripts/native-test.sh` and the subsequent full `make test-native` run:

```text
NATIVE_GUEST_JIT_STATUS=0
NATIVE_GUEST_JIT_RESULT_OK=1
NATIVE_FALLBACK_RUNTIME_MARKERS=0
OPT_DONE=4
BYTECODE_ROOT_TIER2_COMPILATIONS=2
BUILTIN_CALL_ROOT_TIER2_COMPILATIONS=2
OPT_FAILED=0
FRAME_WITHOUT_BOXING_FAILURES=0
COMPILATION_FAILURES=0
NATIVE_GUEST_JIT=PASS
FRAME_WITHOUT_BOXING_REGRESSION=PASS
NATIVE_REGRESSION_SUITE=PASS
```

The CLI Native regression also reported PASS for numeric/TLS conformance, Test Tool, DAP breakpoint/stack trace, Inspect, filesystem I155, LSP, regex, HMAC-SHA256, and I115 caller recovery. The suite explicitly leaves I156 bind-mount and some symlink scenarios **not verified here**; do not promote those to PASS.

## Native TLS JUnit image — image-heap repair and execution

Before the final product change, the separate `native-tls-tests` JUnit image failed during image analysis with a builder-thread image-heap reachability chain through `Engine.ENGINES`, `PolyglotEngineImpl`, `PolyglotContextImpl.threads`, and `NativeImageForkJoinWorkerThread`.

The published `pom.xml` change adds **only in the `native-tls-tests` profile**:

```text
--initialize-at-run-time=com.guillermomolina.protos.execution.ProtosTestExecutionSupport
```

This test-only class owns a shared entered Polyglot context. Deferring its static initialization prevents the JUnit image from storing a builder-created Engine/Context/thread. The production `scripts/native-generate-init-args.sh` initialization policy was not changed.

The complete local `make test-native` run reached **BUILD SUCCESS** for both the Protos CLI Native Image and the separate `target/native-tests` JUnit executable, and ended with:

```text
38 tests found
38 tests started
38 tests successful
0 tests failed
BUILD SUCCESS
NATIVE_TLS_PEER_INTEGRATION=PASS
NATIVE_TLS_TRUST_PROVENANCE=PASS
```

The two selected classes are `ProtosTlsClientIntegrationTest` and `ProtosTlsTrustTest`; the maintainer also ran focal static policy tests including `ProtosPerf006C1OptimizingRuntimeClosureTest` and `ProtosNativeGeneratedStructurePolicyTest`. Their invocation passed without failure.

## Scope and issue links

- [UPSTREAM004 / #753](https://github.com/guillermomolina/protos/issues/753) tracks the earlier independently reproducible `FrameWithoutBoxing` case and Oracle's [#14579](https://github.com/oracle/graal/issues/14579). The successful Protos configuration and workloads **do not establish an upstream compiler patch or universal elimination of that failure**.
- [PLAT045 / #772](https://github.com/guillermomolina/protos/issues/772) is a **historical closed decision** that admitted an interpreter-only Native mode when optimizing Native could not pass. The I115 publication supersedes that *operational probe*; it does not rewrite the original decision's historical facts or claim a separate formal ratification.
- [BUG013 / #749](https://github.com/guillermomolina/protos/issues/749), [TEST006 / #755](https://github.com/guillermomolina/protos/issues/755), and [TEST007 / #757](https://github.com/guillermomolina/protos/issues/757) preserve their historical investigations and original generated-root coverage requirements. After I115 the current positive probe covers `ProtosBytecodeRootNodeGen` and `ProtosBuiltinCallRootNode`; do **not** mislabel the latter `ProtosSemanticBytecodeRootNodeGen`.
- This evidence is not a new release/tag/asset publication and does not close the still-open parent [I115 / #931](https://github.com/guillermomolina/protos/issues/931).

**Validation provenance:** terminal output supplied by the maintainer, plus read-only inspection of the published product commit; no tests or Native Image builds were executed while writing this record.
