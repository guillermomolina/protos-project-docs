# I070 final closure evidence

This record binds the completed I070 warning-reconciliation work to the exact
published product revision, the subsequently reconciled current product head,
and the maintainer-reported final validation.

## Published I070 revision

```text
I070_REVISION=46e4b426a4eef04dbb68c57efe36459a5547f6d5
I070_VERSION=0.3.223-SNAPSHOT
COMMIT=I070: complete test and generated warning reconciliation
PARENT=#712
I070_A=#713
I070_B=#714
I070_C=#715
```

I070-A had already completed at:

```text
I070_A_REVISION=7f398f623a210b2bc3837d37dd3e5980dd726e75
I070_A_VERSION=0.3.222-SNAPSHOT
HANDWRITTEN_MAIN_WARNINGS_AFTER=0
```

## Compiler baseline and final ownership result

The exact I070 product changelog records:

```text
COMMAND=mvn clean package -DskipTests
BASELINE_TOTAL_WARNINGS=89
BASELINE_MAIN_WARNINGS=0
BASELINE_TEST_WARNINGS=62
BASELINE_GENERATED_WARNINGS=24
BASELINE_PROCESSING_WARNINGS=3

FINAL_ACTIONABLE_PROTOS_OWNED_WARNINGS=0
FINAL_HANDWRITTEN_MAIN_WARNINGS=0
```

The remaining raw diagnostics were explicitly dispositioned rather than hidden:

```text
RETAINED_NON_PROTOS_WARNINGS=18
TRUFFLE_GENERATOR_THREADDEATH_WARNINGS=16
JAVAC_MAIN_PROCESSING_WARNINGS=2
GLOBAL_LINT_DISABLED=NO
```

The 16 removal/deprecation diagnostics come from the Truffle Bytecode DSL
generator's exception-handling template around `java.lang.ThreadDeath`
(`resolveThrowable` / `handleException`) for generated
`@GenerateBytecode` roots.

The two main-compilation `processing` diagnostics are javac's
`No processor claimed any of these annotations` warnings over runtime/nested
DSL annotations.

These 18 retained diagnostics are therefore not actionable Protos-owned warning
debt under I070's closure contract.

## I070-B / #714

The combined publication eliminated the handwritten Java-test warning debt:

- 33 deliberately lifetime-only try-with-resources uses receive narrow
  method-level `@SuppressWarnings("try")`;
- five fixtures narrow `close()` from `throws Exception` to
  `throws IOException` where IOException is the actual possible checked
  failure;
- the Actor `CarrierPool` fixture preserves real `InterruptedException`
  propagation under a narrow `try` suppression;
- two redundant `@SafeVarargs` annotations on reifiable `Future<?>` varargs
  are removed;
- test compilation uses `<proc>none</proc>` because test sources declare no
  Truffle DSL elements and annotation processing generated no test output.

The maintainer subsequently confirmed:

```text
GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
```

Accordingly:

```text
I070_B_ACCEPTANCE=PASS
READY_TO_CLOSE_714=YES
```

## I070-C / #715

The locally actionable generated warning family was fixed from handwritten
source rather than by editing generated output:

- `ComposeLocalSlots` uses immutable non-generic
  `ProtosBytecodeRootNode.ComposeReservedNames` instead of
  `List<String>`, eliminating eight DSL-generated unchecked casts;
- test annotation processing is disabled only for `default-testCompile`;
- generated files were not edited directly;
- no Java, GraalVM, Truffle, Maven, or foundational dependency upgrade was used
  as incidental cleanup;
- all retained raw diagnostics have explicit non-Protos ownership.

```text
I070_C_ACCEPTANCE=PASS
UPSTREAM_BLOCKER_FOR_I070=NO
READY_TO_CLOSE_715=YES
```

#715 was closed complete before the final parent closure.

## Concurrent-head reconciliation

After the I070 publication, `main` advanced by one direct descendant commit:

```text
CURRENT_RECONCILED_HEAD=206bbde5593e4eea414b695047d07b894e201fbc
CURRENT_VERSION=0.3.224-SNAPSHOT
INTERVENING_COMMIT=TEST009-Q: revert global native-body PE specialization
ANCESTRY=I070_REVISION_IS_DIRECT_PARENT
```

The intervening TEST009-Q change touches:

```text
CHANGELOG.md
pom.xml
src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
src/main/java/com/guillermomolina/protos/execution/ProtosSemanticBytecodeRootNode.java
src/test/java/com/guillermomolina/protos/execution/ProtosI072PhaseDPreparedCallSeparationTest.java
```

Its substantive Java changes remove the TEST009-M native-body specialization and
its associated helper/import/test expectations. It does not alter I070's Maven
lint configuration, test-compile `proc=none` disposition, serialization
treatment, canonical Bind cleanup, generated ComposeReservedNames carrier, or
the retained-warning ownership classification.

The maintainer's final handoff reports a clean diff check and all local tests
passing after publication. No I070 regression or new warning-remediation slice
is identified.

## Final validation and closure

Maintainer-reported final validation:

```text
PRODUCT_PUSH=PASS
GIT_DIFF_CHECK=PASS
LOCAL_ALL_TESTS=PASS
```

GitHub exposes no exact-SHA combined status or pull-request workflow run for the
I070 publication, so no additional CI result is claimed.

Final state:

```text
I070_A=#713 CLOSED_COMPLETE
I070_B=#714 READY_TO_CLOSE_COMPLETE
I070_C=#715 CLOSED_COMPLETE

ACTIONABLE_PROTOS_OWNED_WARNINGS=0
RETAINED_NON_PROTOS_WARNINGS=18
LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

I070_CLOSURE_CONTRACT=PASS
NEXT_I070_SLICE=NONE
READY_TO_CLOSE_712=YES
```

This is non-normative project evidence. Live lifecycle state remains owned by
GitHub Issues.
