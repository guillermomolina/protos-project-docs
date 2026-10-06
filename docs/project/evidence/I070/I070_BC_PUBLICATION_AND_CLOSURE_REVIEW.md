# I070-B/C publication and I070 closure review evidence

This record binds the combined I070-B / #714 and I070-C / #715 publication to
the exact product revision and records the remaining top-level I070 / #712
closure evidence gap without inventing validation results.

## Exact product publication

```text
PROTOS_REVISION=46e4b426a4eef04dbb68c57efe36459a5547f6d5
PROTOS_VERSION=0.3.223-SNAPSHOT
COMMIT=I070: complete test and generated warning reconciliation
PARENT=#712
TEST_CHILD=#714
GENERATED_CHILD=#715
```

The native GitHub hierarchy has been re-read: #714 and #715 are native children
of #712, alongside already-completed #713.

## Compiler baseline and final ownership result

The exact product changelog at the published revision records the combined
clean-Maven baseline:

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

The remaining raw warnings are explicitly dispositioned rather than hidden:

```text
RETAINED_NON_PROTOS_WARNINGS=18
TRUFFLE_GENERATOR_THREADDEATH_WARNINGS=16
JAVAC_MAIN_PROCESSING_WARNINGS=2
GLOBAL_LINT_DISABLED=NO
```

The 16 removal/deprecation diagnostics come from the Truffle Bytecode DSL
generator's exception-handling template around `java.lang.ThreadDeath`
(`resolveThrowable` / `handleException`) for generated
`@GenerateBytecode` roots. They are generator-owned rather than handwritten
Protos warnings.

The two main-compilation `processing` diagnostics are javac's
`No processor claimed any of these annotations` warnings over runtime/nested
DSL annotations. They are retained explicitly; `-Xlint:all` remains enabled
and no global lint category is disabled.

These retained diagnostics therefore do not represent actionable Protos-owned
debt under the I070 closure contract.

## I070-B / #714 product result

The publication records the following test-source remediation:

- 33 deliberately lifetime-only try-with-resources uses receive narrow
  method-level `@SuppressWarnings("try")`;
- five fixtures narrow `close()` from `throws Exception` to
  `throws IOException` where IOException is the actual possible checked
  failure;
- the Actor `CarrierPool` fixture retains real `InterruptedException`
  propagation while using a narrow `try` suppression;
- two redundant `@SafeVarargs` annotations on reifiable `Future<?>` varargs
  are removed;
- test compilation disables annotation processing with `<proc>none</proc>`
  because test sources declare no Truffle DSL elements and the processor
  generated nothing for them.

The product record reports zero actionable Protos-owned warnings after the
combined reconciliation, and all 18 retained diagnostics are generated/main
processing diagnostics. Therefore the handwritten Java test warning objective is
technically satisfied.

However, the current publication handoff did not explicitly report the
#714-required focused/local test result, and GitHub exposes no exact-SHA status
checks or pull-request workflow runs. This record does not invent that
validation result.

## I070-C / #715 product result

The locally actionable generated warning family was corrected from handwritten
source rather than by editing generated output:

- `ComposeLocalSlots` now carries immutable non-generic
  `ProtosBytecodeRootNode.ComposeReservedNames` instead of
  `List<String>`, eliminating eight DSL-generated unchecked casts.
- test annotation processing is disabled only for `default-testCompile`, where
  no Truffle DSL elements exist and the processor generated no test output.
- generated sources were not edited directly.
- no Java, GraalVM, Truffle, Maven, or foundational dependency upgrade was used
  as incidental cleanup.
- the remaining 18 diagnostics have exact non-Protos ownership and bounded
  disposition.

No independent upstream blocker is required for I070 closure because the parent
contract explicitly permits evidence-backed retained platform/generated
warnings. The retained diagnostics are documented facts, not a local blocked
implementation surface.

Accordingly:

```text
I070_C_TECHNICAL_ACCEPTANCE=PASS
READY_TO_CLOSE_715=YES
UPSTREAM_BLOCKER_FOR_I070=NO
```

## Top-level validation review

The current user handoff establishes:

```text
PRODUCT_PUSH=PASS
```

but does not explicitly state:

```text
GIT_DIFF_CHECK=PASS
FOCUSED_OR_REQUIRED_TESTS=PASS
MAKE_TEST=PASS
```

The exact product commit has no GitHub combined statuses and no pull-request
workflow runs.

Therefore those gates remain unproven in this durable record. This is a
validation-provenance gap only; it is not a compiler-warning or design blocker.

Current closure state:

```text
I070_A=#713 CLOSED_COMPLETE
I070_B=#714 TECHNICALLY_COMPLETE_VALIDATION_EVIDENCE_PENDING
I070_C=#715 READY_TO_CLOSE_COMPLETE
I070_PARENT=#712 TECHNICALLY_COMPLETE_FINAL_VALIDATION_EVIDENCE_PENDING

ACTIONABLE_PROTOS_OWNED_WARNINGS=0
LANGUAGE_SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

PENDING_FOR_714=explicit required test PASS evidence
PENDING_FOR_712=explicit make test PASS and publication-validation evidence
```

This is non-normative project evidence. Live lifecycle state remains owned by
GitHub Issues.
