# UPSTREAM004-F — Oracle response and Protos Native optimizing-runtime checkpoint (2026-11-13)

**Status:** investigative checkpoint only; Protos optimized Native Image and guest Tier-2 **not validated**. Further work is owner-deferred.

**Live work owner:** [UPSTREAM004 / protos#753](https://github.com/guillermomolina/protos/issues/753). This is non-normative evidence, not approval, implementation completion, or a change to the ratified PLAT045 Native capability matrix.

## Sources and revision boundary

- [Oracle/Graal #14579](https://github.com/oracle/graal/issues/14579), closed following Oracle's explanation, without a demonstrated change to the GraalVM compiler in this checkpoint.
- Read-only public Protos reference: [main at 5e89b6df54ecdbfb71c96f0269d11fc5037ce940](https://github.com/guillermomolina/protos/commit/5e89b6df54ecdbfb71c96f0269d11fc5037ce940). The experiments below used additional **local, uncommitted** edits in the user's container; this commit is *not* the experiment's product revision.
- Maintainer-supplied Maven/Native Image output dated 2026-11-13, for Protos `0.3.463-SNAPSHOT`, GraalVM CE `25.5.5.0.3+1.1` / Java 25, native-maven-plugin `1.1.14`.
- Previously published [UPSTREAM004-A–E evidence](./UPSTREAM004_E_ORACLE_GRAAL_ISSUE_PUBLICATION_AND_CLOSURE.md) is historical and remains unchanged.

## What Oracle clarified

Oracle identified the minimum reproducer's generated `MiniPlainRootGen` being initialized **at run time** as the reason `MiniPlainRootGen#execute` was excluded from Native runtime compilation. The opaque call lets the `FrameWithoutBoxing` escape during partial evaluation. The advised configuration for that standalone reproducer is `--initialize-at-build-time=repro`; Oracle said it would investigate an earlier Native Image build-time diagnostic in future releases.

This explanation concerns the reproducer's package. It is **not** a guarantee that the full `com.guillermomolina.protos.execution` package can safely be build-time initialized or that Protos has no other blockers.

The maintainer independently re-downloaded the original 20-member reproducer archive. Its SHA-256 matches the GitHub attachment:

```text
d02b26d8c90a095d4e0d41718889d7e7f654493862c57369adcb38b177289e5e
UPSTREAM004_REPRODUCER_SHA256=PASS
UPSTREAM004_ARCHIVE_INSPECTION=PASS
```

No additional build of the reproducer with Oracle's option was run in this checkpoint. The owner judged such a control redundant for the Protos-specific problem.

## Local experimental variants (not committed or published)

The local Native profile temporarily accepted an **explicit** optimizing-runtime override (`-Dprotos.native.fallbackRuntime=false`) and two generated-initialization scopes:

- `selected`: preserve the existing explicit generated-class initialization list (and preserve fallback-runtime `true` by default).
- `execution-package`: add `com.guillermomolina.protos.execution` as a package scope. A later diagnostic-only iteration also added explicit `ProtosSemanticTransferFamily` and `ProtosModuleKey` entries because the Regex transfer family inherits from the former.

Observed local file set: `pom.xml`, `scripts/native-generate-init-args.sh`, new `scripts/upstream004-native-tier2-probe.sh`, `ProtosNativeGeneratedStructurePolicyTest.java`, and `ProtosPerf006C1OptimizingRuntimeClosureTest.java`. These changes are **uncommitted and not an accepted Protos product fix**; the user's checkout must be inspected afresh before future work.

Focal validation reported by the maintainer: initial **6/6 PASS** (four optimizing-runtime policy and two generated-structure policy tests). A subsequent generated-structure test failed because the experiment's shell path interpolation emitted literal backslashes; the path conversion was corrected and the failing focal class subsequently returned **2/2 PASS**, `BUILD SUCCESS`. Latest reported `git diff --check` was clean. No full Protos test suite was run, no product version change or changelog change was made for this experiment, and no optimizing Native executable was produced.

## Native Image build failures

| Variant / stage | Observable failure | Significance |
| --- | --- | --- |
| `selected`, optimizing runtime enabled | `HostedError: Blocklisted methods are reachable for runtime compilation`; **one** violation: `java.math.BigInteger.valueOf(long)` | Independent Native Image build-time gate, **not** evidence of Tier-2 `FrameWithoutBoxing` behavior |
| `execution-package`, first build | Image-heap object of `com.guillermomolina.protos.execution.ProtosRegexSemanticTransferFamily` not permitted while initialized at run time; image root at `ProtosCoreBootstrap.standardSemanticTransferFamilies` | Package-wide initialization exposed a separate class/heap consistency problem |
| `execution-package`, after narrow superclass/module-key additions | Image-heap object of `com.guillermomolina.protos.runtime.ProtosClosureValue` marked for run-time initialization; build aborted | Broad package initialization continues to pull Protos runtime state into image build, and was not pursued further |

The `selected` blocklist trace runs through:

```text
java.math.BigInteger.valueOf(long)
  com.oracle.truffle.api.interop.InteropLibraryGen.genericDispatch
  com.oracle.truffle.api.library.ReflectionLibraryDefault$Send.doSendCached
  com.oracle.truffle.api.interop.InteropLibraryGen$Proxy.getArraySize
  com.oracle.truffle.polyglot.PolyglotListFactory$CacheFactory$SizeNodeGen.executeImpl
  com.oracle.truffle.polyglot.HostToGuestRootNode.execute
  com.oracle.truffle.runtime.OptimizedCallTarget.executeRootNode
  com.oracle.truffle.runtime.OptimizedCallTarget.callInlined
```

The blocklist violation is **reported by Native Image's Truffle feature**, not directly by a Protos numeric operation. Its precise root cause and smallest supported repair are *not established*. Do not remove `BigInteger` functionality or bypass the compiler's blocklist based on this trace.

In the later `execution-package` iteration, the maintainer verified the actual generated image arguments included the package-level build-time entry and the specific Bytecode DSL root entry. The following heap failures therefore cannot be attributed simply to that package option being omitted.

No successful optimized Native Image build was obtained, so **no runtime Tier-2 probe ran**. The Oracle `FrameWithoutBoxing` explanation has not been verified against Protos' optimized Native artifact.

## Exact current disposition

```text
ORACLE_GRAAL_14579=UPSTREAM_CLOSED_WITH_CONFIGURATION_EXPLANATION
UPSTREAM_REPRODUCER_SHA256=PASS
PROTOS_SELECTED_OPTIMIZING_NATIVE_BUILD=FAIL_BLOCKLIST_BIGINTEGER_VALUEOF
PROTOS_EXECUTION_PACKAGE_INITIAL_BUILD=FAIL_IMAGE_HEAP_REGEX_FAMILY
PROTOS_EXECUTION_PACKAGE_LATER_BUILD=FAIL_IMAGE_HEAP_CLOSURE
PROTOS_OPTIMIZING_NATIVE_BUILD=NOT_AVAILABLE
PROTOS_NATIVE_GUEST_TIER2=NOT_TESTED
FRAMEWITHOUTBOXING_REPAIRED_IN_PROTOS=NOT_DEMONSTRATED
PROTOS_PRODUCTION_NATIVE=PLAT045_INTERPRETER_ONLY_UNCHANGED
PRODUCT_COMMIT_OR_PUSH=NO
FULL_SUITE=NOT_RUN
FOLLOW_UP=OWNER_DEFERRED
```

## Resume boundary (only on explicit owner direction)

1. Inspect the then-current Protos checkout and reconcile/revert the local **package-wide experimental expansion**, without disturbing independent work or weakening the ordinary `-Dtruffle.UseFallbackRuntime=true` default.
2. Research the **`BigInteger.valueOf(long)` runtime-compilation blocklist** against the exact adopted GraalVM sources and comparable Truffle language/runtime code (including GraalPy), using the actual reachability trace. Avoid constructing a separate reproducer for each failure unless isolation becomes necessary for an upstream report.
3. Propose one narrow, justified remediation and focal tests, without changing Protos observable semantics, deleting numeric support, disabling blocklist checks, or widening build-time initialization indiscriminately.
4. Only after an optimized Native Image **builds successfully**, run a real guest Tier-2 workload and determine whether the Oracle root-initialization explanation resolves `FrameWithoutBoxing` in Protos. Do not equate build success or guest result success with successful Tier-2 compilation.
5. Preserve PLAT045's supported interpreter-only Native mode and the separate JVM optimizing-runtime admission. Any change to ratified platform policy needs its own owner-approved decision.

This record is a pause checkpoint. It does not close UPSTREAM004, reopen BUG013, claim a compiler fix, authorize a release, or schedule further tests.
