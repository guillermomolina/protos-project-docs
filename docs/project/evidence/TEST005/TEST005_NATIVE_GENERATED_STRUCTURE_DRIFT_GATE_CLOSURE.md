# TEST005 — Native generated-structure drift gate closure evidence

Date: 2026-10-03
Status: **CLOSED**

## Work identity

```text
WORK_ITEM=TEST005
PROTOS_ISSUE=guillermomolina/protos#747
TRIGGER=DIST009/guillermomolina/protos#743
ESCAPED_SHAPE_CHANGE=I074/guillermomolina/protos#745
PRODUCT_REPOSITORY=guillermomolina/protos
PROJECT_RECORD_REPOSITORY=guillermomolina/protos-project-docs
IMPLEMENTATION_REVISION=4b354722906055b969c67bf02a91a6d9ccd5764e
IMPLEMENTATION_PARENT=898eb8b2bafe4be99a33032ab0cf6436ce6f3e72
CLOSURE_INSPECTION_REVISION=1e8fbb27ee3966ccc48a57e04308c17a58995bdc
```

This record is durable non-normative validation evidence. It does not replace
Native Image release admission, the normative Protos specification, or the live
GitHub Issue state.

## Trigger and required gap closure

DIST009 exposed a repository-owned validation gap after I074 enabled the
Bytecode DSL uncached interpreter: the ordinary JVM/Test Tool path could remain
green while `build/native/generate-init-args.sh` still encoded an older generated
Bytecode shape. The escaped generated structures included:

```text
ProtosBytecodeRootNodeGen$UncachedBytecodeNode
ProtosBytecodeRootNodeGen$UncachedBytecodeNodeTailCall
ProtosBytecodeRootNodeGen$TagNode
ProtosBytecodeRootNodeGen$VirtualState
```

TEST005 owns only that cheap structural drift class. It does not claim that a
JVM-only check can prove that a complete Native Image will build.

## Published implementation

The bounded implementation was published in:

```text
4b354722906055b969c67bf02a91a6d9ccd5764e
TEST005: detect Native generated-structure policy drift
```

That publication changed exactly the two TEST005-owned surfaces:

- `build/native/generate-init-args.sh` now derives the applicable initial
  Bytecode tier from generated classes under `target/classes`, conditionally
  includes the generated uncached tail-call, `TagNode`, and `VirtualState`
  structures, and fails on missing or ambiguous generated structure;
- `src/test/java/com/guillermomolina/protos/execution/ProtosNativeGeneratedStructurePolicyTest.java`
  runs the generator against compiled output, inventories sensitive direct
  generated roles, and verifies that the emitted Native build-time
  initialization policy covers the generated structural set.

The generator continues to discover generated operation-node classes directly
from `target/classes`; TEST005 does not add a Native Image compilation to this
path.

## Ordinary approval-path ownership

At closure inspection revision
`1e8fbb27ee3966ccc48a57e04308c17a58995bdc`, repository `Makefile` authority
still defines:

```text
make test
    -> test-java
    -> test-protos
```

`ProtosNativeGeneratedStructurePolicyTest` is an ordinary JUnit test and is not
in the serial-lane exclusion set, so it participates in the ordinary Java test
approval path after Maven/annotation processing has produced `target/classes`.

The regression invokes only:

```text
bash build/native/generate-init-args.sh <classes> <output>
```

The generator inspects class files and writes Native Image arguments. Neither
the regression nor the generator invokes `native-image`.

Therefore:

```text
ORDINARY_APPROVAL_REQUIRES_NATIVE_IMAGE_BUILD=NO
FULL_NATIVE_BUILD_AUTHORITY_PRESERVED=YES
```

## Drift signal and I074 regression

The retained regression makes generated-shape drift fail closed in two layers:

1. the sensitive direct-role inventory must match the currently maintained
   Bytecode DSL structural shape; and
2. the generated initialization arguments must contain every required
   repository-owned structural class.

The required set contains the I074 escape class explicitly, including
`UncachedBytecodeNode`, `UncachedBytecodeNodeTailCall`, `TagNode`, and
`VirtualState`.

Failure output is structural rather than generic. The generator reports cases
such as missing/ambiguous initial tiers or missing generated structural classes,
and the JUnit coverage assertion reports the exact missing Native generated
structures.

Subsequent platform work changed the semantic Bytecode root to the same uncached
interpreter structural shape. At the closure inspection revision, TEST005 has
been maintained accordingly: both generated root families are checked for the
same sensitive uncached/tail-call/tag/virtual-state role set. This confirms that
the guard is a live repository-owned drift check rather than a one-time I074
snapshot left behind after later generator evolution.

## Exact-revision validation

GitHub Actions CI ran against the exact implementation revision.

```text
CI_RUN=36743305422
CI_EVENT=push
CI_CONCLUSION=success
CI_JOB=test
CI_STEP=Run repository tests
CI_STEP_CONCLUSION=success
```

The exact-revision `.github/workflows/tests.yml` defines that step as:

```text
make test
echo "CI_REPOSITORY_TESTS: PASS"
```

A later CI run for the same product SHA (`36748979867`) also completed
successfully.

The product implementation therefore has an exact-revision green ordinary
repository gate that includes the TEST005 JUnit regression without compiling a
Native Image.

## Closure matrix

```text
ORDINARY_APPROVAL_REQUIRES_NATIVE_IMAGE_BUILD=NO
GENERATED_BYTECODE_STRUCTURE_DISCOVERY=PASS
NATIVE_INIT_POLICY_STRUCTURAL_COVERAGE=PASS
I074_ESCAPED_DRIFT_REGRESSION=PASS
UNCOVERED_CLASS_DIAGNOSTIC=ACTIONABLE
FULL_NATIVE_BUILD_AUTHORITY_PRESERVED=PASS
OBSERVABLE_PROTOS_SEMANTICS_CHANGE=NONE
REQUIRED_VALIDATION=PASS
PUBLICATION=PASS

TEST005_PRODUCT_IMPLEMENTATION=COMPLETE
TEST005_TECHNICAL_SLICE_PENDING=NO
TEST005_STATUS=CLOSED_COMPLETED
```

No additional TEST005 implementation or investigation slice is identified by
this closure. Full Native Image build/release admission remains independently
authoritative for failure classes that cannot be proved by this JVM-only
structural gate.
