# PERF025 — Lazy shared lexical environment for Closure capture

## Status

Published product implementation evidence for PERF025 / guillermomolina/protos#758.

This record is non-normative. It retains the exact product publication identity,
the structural implementation result, and the maintainer-reported validation
outcome for the lazy Closure lexical-capture slice.

## Exact product publication

PROTOS_REVISION=9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618
PROTOS_PARENT_REVISION=245b0001e4cb7c7ae3192f35d262a8dcf34fdcd6
PROTOS_VERSION=0.3.152-SNAPSHOT
COMMIT_SUBJECT=PERF025: lazy shared lexical environment for Closure capture
OWNING_ISSUE=guillermomolina/protos#758

The product commit is exactly one commit after the preceding PERF025 conditional
frame-materialization slice.

## Published structural change

Before this slice, materializing a Closure literal could eagerly project the
creating activation into a complete List of guest execution Context objects.
The MaterializeClosure path called lexicalContextsForClosureCapture(), which
could call context(), materialize the outer semantic execution Context, build a
capture chain, and then copy that chain into the Closure and later activation
representations.

The published implementation replaces that hot physical shape with a shared
ProtosLexicalEnvironment.

Each activation creates at most one lexical-environment node. Closures created
from that activation capture the same node by reference through
ProtosActivation.lexicalEnvironmentForClosureCapture(). MaterializeClosure no
longer needs to call context() or eagerly build/copy the complete guest Context
chain merely because a Closure value exists.

Captured reads, writes and lexical fallback walk the lexical-environment chain
directly. Deferred environment nodes answer membership and frame-backed
authority from the owning activation's existing single lexical store.

Guest-visible execution Context materialization remains available through the
activation's context() path when semantic observation actually requires it.
That preserves one semantic Context identity for observers while keeping the
ordinary unobserved Closure-capture path lazy.

The legacy list projections capturedLexicalContexts() and
lexicalContextsForClosureCapture() remain available as cold materializing
projections for consumers that genuinely require guest Context objects.

## Preserved semantic and runtime boundaries

The product changelog and implementation retain the following boundaries:

CAPTURE_BY_REFERENCE=PRESERVED
SHARED_CONTEXT_IDENTITY=PRESERVED
D179_C0_NEARER_PRESENCE_RETARGETING=PRESERVED
LATE_CREATION_AND_REMOVAL=PRESERVED
OBJECT_BODY_CAPTURE_BOUNDARY=PRESERVED
BIND_METHOD_LEXICAL_PROVENANCE=PRESERVED
PARALLEL_PROJECTION_BOUNDARY=PRESERVED
RECEIVER_AND_METHOD_HOME=PRESERVED
RETURN_HOME_AND_NLR=PRESERVED
FRAME_BACKED_SINGLE_AUTHORITY=PRESERVED
PERF013_MATERIALIZED_LOCAL_ACCESSOR_PATH=PRESERVED
SEMANTIC_CHANGE=NO
SPECIFICATION_CHANGE=NO

In particular, this is not a new non-capturing Closure category and it does not
change capture-by-reference into capture-by-value. The optimization changes the
physical representation and materialization timing only.

## Exact product delta

The published commit changes:

- CHANGELOG.md
- pom.xml
- src/main/java/com/guillermomolina/protos/execution/CanonicalLexicalScope.java
- src/main/java/com/guillermomolina/protos/execution/ProtosBytecodeRootNode.java
- src/main/java/com/guillermomolina/protos/runtime/ProtosActivation.java
- src/main/java/com/guillermomolina/protos/runtime/ProtosClosureValue.java
- src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalFallback.java
- src/test/java/com/guillermomolina/protos/execution/ProtosPerf025LazyLexicalCaptureTest.java

and adds:

- src/main/java/com/guillermomolina/protos/runtime/ProtosLexicalEnvironment.java

The product CHANGELOG records the implementation under 0.3.152-SNAPSHOT.

## Focused regression coverage

The product publication adds
ProtosPerf025LazyLexicalCaptureTest as the dedicated regression surface for the
new lazy shared lexical-environment representation.

The maintained product changelog additionally records preservation of captured
read/write behavior, debugger/reflection observation through one Context
identity, D179 C0 retargeting, object-body capture boundaries, bindMethod,
parallel projection, receiver/methodHome, return-home and NLR behavior.

## Validation status

The maintainer reported after publication:

PERF025: lazy shared lexical environment for Closure capture, todos los test pass

This durable record therefore retains:

MAINTAINER_REPORTED_VALIDATION=PASS
PUBLICATION=PASS
PROTOS_REVISION=9ecb5c7d0a7ccc47f3b7e9d0f16e2ecc84e95618

No independent test rerun is claimed by this documentation publication.

## PERF025 consequence

This product revision resolves the post-F1 structural finding previously
retained as UNIVERSAL_CLOSURE_CONTEXT_CAPTURE: Closure creation no longer has to
materialize the outer guest execution Context or copy the complete lexical
Context chain merely because Protos Closures have lexical-capture semantics.

UNIVERSAL_CLOSURE_CONTEXT_CAPTURE=RESOLVED_BY_PRODUCT_REVISION
EAGER_CONTEXT_MATERIALIZATION_ON_EVERY_CLOSURE=NO
EAGER_FULL_CAPTURE_CHAIN_BUILD_ON_EVERY_CLOSURE=NO
SHARED_LAZY_LEXICAL_ENVIRONMENT=YES

PERF025 itself remains open. This record does not claim a measured speedup,
benchmark magnitude, or closure of the broader PERF025 workstream.

BENCHMARK_RUN_FOR_THIS_SLICE=NO
PERFORMANCE_MAGNITUDE_CLAIMED=NO
PERF025_STATUS=OPEN
