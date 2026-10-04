# TEST009 — systematic Truffle compilerability procedure investigation

Evidence date: **2026-10-04**

Owning work item: `TEST009 / guillermomolina/protos#795`

Trigger: `PERF030 / guillermomolina/protos#784`

Investigation type: **read-only methodology investigation**. No Protos product,
test, build, runtime, version, changelog or specification changes are part of
this record.

## Question

After TEST009-E demonstrated that several cold/generic host paths had expanded
deeply into guest partial evaluation, determine how mature Truffle languages
systematically prevent and diagnose compilerability regressions.

The specific question was whether mature Truffle implementations rely mainly on
manual `@TruffleBoundary` trial-and-error, or whether Truffle/Graal provides a
repeatable compilerability/testing procedure.

## Conclusion

The mature-language procedure is systematic.

There is no upstream tool that automatically decides the correct
`@TruffleBoundary` location for arbitrary Java methods. That choice remains a
semantic/architectural classification.

However, mature Truffle projects do not use method-by-method boundary sweeps as
their primary compilerability authority. They force real Truffle compilation
during tests, make compilation failures fatal, often treat compiler performance
warnings as errors, and only then use Truffle/Graal expansion/inlining
diagnostics to explain a failure.

The reusable model is:

~~~text
functional/corpus execution
        |
        v
forced Truffle compilation
        |
        +-- compilation failure ----------> FAIL
        |
        +-- performance warning ----------> FAIL or explicit regression
        |
        +-- semantic result mismatch ------> FAIL
        |
        +-- clean compilation ------------> PASS

On failure only:
  TraceCompilation
  TracePerformanceWarnings
  TraceMethodExpansion / MethodExpansionStatistics
  TraceNodeExpansion / NodeExpansionStatistics
  TraceInlining
  compiler diagnostics / IGV when textual evidence is insufficient
~~~

This is materially different from an automatic loop that tries
`@TruffleBoundary` on candidate methods and retains benchmark winners.

## TruffleRuby precedent

Exact inspected revision:

~~~text
repository=truffleruby/truffleruby
revision=c734f26543003fefd4519a29adbca62d0c711a2d
~~~

Relevant paths:

~~~text
tool/jt.rb
test/truffle/compiler/compile-immediately.sh
~~~

`tool/jt.rb` provides a `--check-compilation` mode that adds:

~~~text
--engine.CompilationFailureAction=ExitVM
--compiler.TreatPerformanceWarningsAsErrors=all
~~~

Its `--stress` mode adds:

~~~text
--engine.CompileImmediately
--engine.BackgroundCompilation=false
--check-compilation
~~~

Therefore its stress compilation mode composes:

~~~text
CompileImmediately
BackgroundCompilation=false
CompilationFailureAction=ExitVM
TreatPerformanceWarningsAsErrors=all
~~~

The dedicated `compile-immediately.sh` test deliberately runs both:

~~~text
CompileImmediately + BackgroundCompilation=false
CompileImmediately + background compilation enabled
~~~

Its source comment states that the two modes catch different issues.

Classification:

~~~text
TRUFFLERUBY_REAL_COMPILATION_GATE=YES
COMPILATION_FAILURES_FATAL=YES
PERFORMANCE_WARNINGS_FATAL=YES
IMMEDIATE_COMPILATION=YES
SYNC_AND_BACKGROUND_VARIANTS=YES
AUTOMATIC_BOUNDARY_SEARCH=NO
~~~

## GraalJS precedent

Exact inspected revision:

~~~text
repository=oracle/graaljs
revision=1d88c09515b265d25085859434222e75da8fc33d
~~~

Relevant paths include:

~~~text
graal-js/src/com.oracle.truffle.js.test.external/src/com/oracle/truffle/js/test/external/suite/SuiteConfig.java

graal-js/test/regression/perf_warn_array_shift.test
graal-js/test/regression/perf_warn_dynamic_import.test
graal-js/test/regression/perf_warn_interop_typed_array.test
~~~

The external-suite compile mode configures:

~~~text
engine.CompileImmediately=true
engine.BackgroundCompilation=false
engine.Mode=latency
~~~

Performance-warning regression tests run with the equivalent of:

~~~text
engine.CompilationFailureAction=ExitVM
engine.TraceCompilation=true
engine.CompileImmediately=true
engine.BackgroundCompilation=false
compiler.TracePerformanceWarnings=all
~~~

and assert that no performance warning is emitted while compilation completes.

Classification:

~~~text
GRAALJS_FORCED_COMPILATION_TEST_MODE=YES
COMPILATION_FAILURES_FATAL=YES
PERFORMANCE_WARNING_REGRESSIONS=YES
AUTOMATIC_BOUNDARY_SEARCH=NO
~~~

## GraalPython precedent

Exact inspected revision:

~~~text
repository=oracle/graalpython
revision=178ea261764d6478b09d7cdc2dee1060e139e91e
~~~

Relevant paths:

~~~text
mx.graalpython/mx_graalpython.py
graalpython/com.oracle.graal.python.shell/src/com/oracle/graal/python/shell/GraalPythonMain.java
~~~

The test/launcher infrastructure uses:

~~~text
-Dpolyglot.engine.CompilationFailureAction=ExitVM
~~~

when compiler support is present, making genuine Truffle compilation failures
fatal to those runs.

The performance-debug launcher exposes
`TracePerformanceWarnings=all` among its diagnostic options.

Classification:

~~~text
GRAALPYTHON_COMPILATION_FAILURES_FATAL_IN_TEST_INFRA=YES
PERFORMANCE_WARNING_DIAGNOSTIC_SURFACE=YES
AUTOMATIC_BOUNDARY_SEARCH=NO
~~~

## Truffle/Graal framework procedure

Exact inspected Graal revision:

~~~text
repository=oracle/graal
revision=384c27b420f8e425fa7eeecd4466767d50f4746f
~~~

Relevant framework references:

~~~text
truffle/docs/Optimizing.md
truffle/docs/PartialEvaluation.md
truffle/docs/Options.md
truffle/CHANGELOG.md

compiler/src/jdk.graal.compiler/src/jdk/graal/compiler/truffle/
TruffleCompilerOptions.java

compiler/src/jdk.graal.compiler.test/src/jdk/graal/compiler/truffle/test/
PerformanceWarningTest.java
~~~

The framework provides dedicated observability for:

~~~text
TraceCompilation

TracePerformanceWarnings=
  call
  instanceof
  store
  frame_merge
  trivial
  all

TreatPerformanceWarningsAsErrors

TraceMethodExpansion
MethodExpansionStatistics

TraceNodeExpansion
NodeExpansionStatistics

TraceInlining
TraceInliningDetails

TraceTransferToInterpreter
TraceAssumptions

compiler graph dumps / IGV
~~~

The framework documentation describes the characteristic symptoms of bad
partial evaluation as including:

~~~text
performance warnings
repeated deoptimization / invalidation
graph-size bailouts
unexpected Java methods in method expansion
unexpected Truffle nodes in node expansion
~~~

The Truffle changelog explicitly encourages language implementations to run the
method/node expansion diagnostics and investigate unexpected results.

The Graal compiler's own `PerformanceWarningTest` constructs a context with:

~~~text
compiler.TracePerformanceWarnings=all
compiler.TreatPerformanceWarningsAsErrors=all
engine.CompilationFailureAction=ExitVM
~~~

which confirms that performance-warning-as-error is a framework-supported
testing pattern, not a TruffleRuby-specific convention.

## Systematic classification of remedies

The framework diagnostics identify a failing property; they do not prescribe one
universal source edit.

A failure must be classified before choosing a remedy:

~~~text
cold/generic host work expanded into PE
  -> candidate @TruffleBoundary

PE-relevant path has unresolved polymorphism
  -> specialization/cache/guard/library design

PE-relevant method is valid but excessively large
  -> inspect inlining structure; possible InliningCutoff or structural change

required structural operand is not PE-constant
  -> constant operand / cached invariant / existing specialized static guard

unstable state repeatedly invalidates compiled code
  -> assumption/state-lifetime investigation

framework/native PE block-list violation
  -> framework-supported representation or explicit boundary

unknown graph owner
  -> TraceMethodExpansion / TraceNodeExpansion before source changes
~~~

Therefore:

~~~text
DIAGNOSTIC_FAILURE != AUTOMATIC_BOUNDARY_PRESCRIPTION
~~~

## Implication for existing TEST009 work

Existing TEST009-A/B/C/D guards remain valid because they prove narrow,
mechanically defined PE contracts more cheaply than a whole-runtime compilation
test.

TEST009-D already contains part of the mature-language procedure:

~~~text
CompileImmediately=true
BackgroundCompilation=false
CompilationFailureAction=Print
TraceCompilation=true
~~~

It remains family-specific because it deliberately treats unrelated permanent
failures as reported-but-not-owned.

TEST009-E product repairs also remain valid and published. They moved host-side
paths already demonstrated by real compilation evidence out of PE while
preserving hot fast paths.

No existing A-E work needs to be reverted merely because the methodology is
being improved.

What changes is the authority for the next dynamic family:

~~~text
OLD_DYNAMIC_METHOD:
  try boundaries on many methods
  compile/measure
  retain winners

NEW_DYNAMIC_METHOD:
  run strict corpus compilation gate
  fail closed on compiler failure/warnings
  diagnose the exact failing expansion
  classify the remedy
  implement one causally justified repair
~~~

## Protos implementation direction

The next slice should build one general compilerability gate by reusing the
packaged-JVM launch and log parsing concepts already present in:

~~~text
tools/java_generated_bytecode_bci_compilation_check.py
~~~

The new gate must not replace the D family-specific check. It should provide a
general repository-level authority with two explicit modes:

~~~text
SYNC:
  CompileImmediately=true
  BackgroundCompilation=false

BACKGROUND:
  CompileImmediately=true
  background compilation enabled
~~~

Both must fail closed on genuine compilation failure.

The implementation must first verify the exact GraalVM 25.4.4.1 option spelling
accepted by the repository's current toolchain before making performance warnings
fatal.

The target architecture is:

~~~text
make check-truffle-compilation
  -> package once
  -> run representative Test Tool/corpus under SYNC strict compilation
  -> run representative Test Tool/corpus under BACKGROUND strict compilation
  -> require semantic success
  -> require actual successful Truffle compilation
  -> require zero fatal compilation failures
  -> require zero forbidden performance warnings

make diagnose-truffle-compilation
  -> explicit/manual diagnostic target only
  -> TraceCompilation
  -> TracePerformanceWarnings
  -> TraceMethodExpansion / statistics
  -> TraceNodeExpansion / statistics
  -> TraceInlining
  -> retain artifacts under target/
  -> no automatic production mutation
~~~

IGV remains an escalation tool when compact textual diagnostics are
insufficient; it should not be required for an ordinary compilerability PASS.

## Aggregate-policy boundary

Do not automatically add the new general gate to ordinary `make check` before
measuring its runtime and determinism.

Existing TEST009 policy remains:

~~~text
individual target = always available
cheap/deterministic target = candidate for make check
expensive diagnostic target = explicit/periodic
~~~

The implementation slice must report its runtime before deciding aggregate
placement.

## Result

~~~text
SYSTEMATIC_TRUFFLE_COMPILERABILITY_PROCEDURE=ESTABLISHED

AUTOMATIC_BOUNDARY_SEARCH_AS_DESIGN_AUTHORITY=REJECTED
EXISTING_TEST009_A_TO_E_WORK=KEEP
EXISTING_TEST009_E_PRODUCT_REPAIRS=KEEP
EXISTING_FAMILY_SPECIFIC_STATIC_GUARDS=KEEP
EXISTING_TEST009_D_DYNAMIC_CHECK=KEEP

NEXT_SLICE=TEST009-F
NEXT_SLICE_TYPE=IMPLEMENTATION
IMPLEMENTATION_REPOSITORY=guillermomolina/protos
NEXT_SCOPE=GENERAL_STRICT_TRUFFLE_COMPILATION_GATE
PRODUCT_SEMANTIC_CHANGE=NO
PRODUCT_RUNTIME_CHANGE_EXPECTED=NO
NEW_D_OR_PLAT_GATE_REQUIRED=NO
NEW_FORMAL_ISSUE_REQUIRED=NO
~~~

AI assistance: this record was drafted with ChatGPT from the current Protos
TEST009 infrastructure and exact upstream TruffleRuby, GraalJS, GraalPython and
Graal/Truffle source/documentation revisions listed above.
